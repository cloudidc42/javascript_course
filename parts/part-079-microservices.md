# Part 79: Microservices กับ JavaScript (Steps 1551-1570)

## บทนำ: Microservices Architecture คืออะไร?

**Microservices** คือรูปแบบสถาปัตยกรรม Software ที่แบ่ง application ออกเป็น services เล็กๆ ที่ทำงานแยกกัน แต่ละ service มีหน้าที่เฉพาะ, ใช้ database แยก, และสื่อสารผ่าน network

---

## Step 1551: Monolith vs Microservices

### Monolithic Architecture

```
┌─────────────────────────────────────────────────────┐
│                  Monolith App                        │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐            │
│  │   User   │ │ Product  │ │  Order   │            │
│  │ Module   │ │ Module   │ │ Module   │            │
│  └──────────┘ └──────────┘ └──────────┘            │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐            │
│  │ Payment  │ │Inventory │ │Shipping  │            │
│  │ Module   │ │ Module   │ │ Module   │            │
│  └──────────┘ └──────────┘ └──────────┘            │
│                                                      │
│  ┌────────────────────────────────────────────────┐ │
│  │           Single Database                       │ │
│  └────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────┘
```

**ข้อดี Monolith:**
- ง่ายในการพัฒนาเริ่มต้น
- ง่ายใน debug (ทุกอย่างอยู่ที่เดียว)
- Transaction ง่าย
- Deploy ง่าย (ไฟล์เดียว)

**ข้อเสีย Monolith:**
- Scale ทั้งหมดหรือไม่ก็ไม่ scale
- Deploy ทั้งหมดแม้แก้นิดเดียว
- Technology locked (ใช้ได้แค่ภาษาเดียว)
- Teams ต้อง coordinate มาก
- ยิ่งใหญ่ยิ่ง slow

### Microservices Architecture

```
                   API Gateway
                       │
          ┌────────────┼────────────┐
          │            │            │
   ┌──────▼─────┐ ┌───▼────┐ ┌────▼───────┐
   │   User     │ │Product │ │  Order     │
   │  Service   │ │Service │ │  Service   │
   │  :3001     │ │ :3002  │ │   :3003    │
   └──────┬─────┘ └───┬────┘ └────┬───────┘
          │           │           │
   ┌──────▼─┐    ┌────▼──┐   ┌───▼────┐
   │User DB │    │Prod DB│   │Order DB│
   └────────┘    └───────┘   └────────┘
```

**ข้อดี Microservices:**
- Scale แต่ละ service แยกกัน
- Deploy แยกกัน
- Technology ต่างกันได้
- Teams ทำงาน independent
- Fault isolation (service เดียว fail ไม่พังทั้งหมด)

**ข้อเสีย Microservices:**
- ซับซ้อนกว่ามาก
- Network latency
- Distributed transactions ยาก
- Testing ยาก
- Operational overhead สูง

---

## Step 1552: เมื่อไหรควรใช้ Microservices?

```
ควรใช้ Microservices เมื่อ:
✓ ทีมใหญ่ (50+ developers)
✓ System ซับซ้อน, แต่ละส่วน scale requirement ต่างกัน
✓ ต้องการ release frequency สูง
✓ มี budget สำหรับ DevOps infrastructure
✓ Organization structure แยก teams ตาม domains

ไม่ควรใช้ Microservices เมื่อ:
✗ ทีมเล็ก (< 10 คน)
✗ Product ยังไม่ชัดเจน (MVP stage)
✗ ไม่มี DevOps expertise
✗ Business domain ยังไม่ชัดเจน
✗ Startup ที่ต้องการ speed

คำแนะนำ: เริ่มด้วย Monolith แล้วค่อยแยกเป็น Services
เมื่อ "Seams" ชัดเจนแล้ว
```

---

## Step 1553: สร้าง Microservices ด้วย Node.js/Express

### โครงสร้างโปรเจค

```
microservices-demo/
├── api-gateway/          ← API Gateway
│   ├── src/
│   └── package.json
├── user-service/         ← User management
│   ├── src/
│   └── package.json
├── product-service/      ← Product catalog
│   ├── src/
│   └── package.json
├── order-service/        ← Order management
│   ├── src/
│   └── package.json
├── notification-service/ ← Email/SMS
│   ├── src/
│   └── package.json
└── docker-compose.yml
```

### User Service

```javascript
// user-service/src/index.js
const express = require('express')
const { Pool } = require('pg')
const bcrypt = require('bcrypt')
const jwt = require('jsonwebtoken')

const app = express()
app.use(express.json())

const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
})

// Health check endpoint (ทุก service ต้องมี)
app.get('/health', (req, res) => {
  res.json({
    status: 'healthy',
    service: 'user-service',
    timestamp: new Date().toISOString(),
  })
})

// Register user
app.post('/users/register', async (req, res) => {
  try {
    const { name, email, password } = req.body
    
    // Validate input
    if (!name || !email || !password) {
      return res.status(400).json({ error: 'All fields required' })
    }
    
    // Check existing user
    const existing = await pool.query(
      'SELECT id FROM users WHERE email = $1',
      [email]
    )
    if (existing.rows.length > 0) {
      return res.status(409).json({ error: 'Email already exists' })
    }
    
    // Hash password
    const hashedPassword = await bcrypt.hash(password, 12)
    
    // Create user
    const result = await pool.query(
      'INSERT INTO users (name, email, password) VALUES ($1, $2, $3) RETURNING id, name, email',
      [name, email, hashedPassword]
    )
    
    const user = result.rows[0]
    
    // Generate token
    const token = jwt.sign(
      { userId: user.id, email: user.email },
      process.env.JWT_SECRET,
      { expiresIn: '7d' }
    )
    
    res.status(201).json({ user, token })
  } catch (error) {
    console.error('Register error:', error)
    res.status(500).json({ error: 'Internal server error' })
  }
})

// Login
app.post('/users/login', async (req, res) => {
  try {
    const { email, password } = req.body
    
    const result = await pool.query(
      'SELECT * FROM users WHERE email = $1',
      [email]
    )
    
    const user = result.rows[0]
    if (!user) {
      return res.status(401).json({ error: 'Invalid credentials' })
    }
    
    const isValid = await bcrypt.compare(password, user.password)
    if (!isValid) {
      return res.status(401).json({ error: 'Invalid credentials' })
    }
    
    const token = jwt.sign(
      { userId: user.id, email: user.email },
      process.env.JWT_SECRET,
      { expiresIn: '7d' }
    )
    
    res.json({
      user: { id: user.id, name: user.name, email: user.email },
      token,
    })
  } catch (error) {
    res.status(500).json({ error: 'Internal server error' })
  }
})

// Get user by ID (internal endpoint สำหรับ services อื่น)
app.get('/users/:id', async (req, res) => {
  try {
    const result = await pool.query(
      'SELECT id, name, email, created_at FROM users WHERE id = $1',
      [req.params.id]
    )
    
    if (result.rows.length === 0) {
      return res.status(404).json({ error: 'User not found' })
    }
    
    res.json(result.rows[0])
  } catch (error) {
    res.status(500).json({ error: 'Internal server error' })
  }
})

const PORT = process.env.PORT || 3001
app.listen(PORT, () => console.log(`User service running on port ${PORT}`))
```

### Product Service

```javascript
// product-service/src/index.js
const express = require('express')
const mongoose = require('mongoose')

const app = express()
app.use(express.json())

mongoose.connect(process.env.MONGODB_URL)

// Product Schema
const productSchema = new mongoose.Schema({
  name: { type: String, required: true },
  description: String,
  price: { type: Number, required: true },
  stock: { type: Number, default: 0 },
  category: String,
  imageUrl: String,
}, { timestamps: true })

const Product = mongoose.model('Product', productSchema)

app.get('/health', (req, res) => res.json({ status: 'healthy', service: 'product-service' }))

// Get all products
app.get('/products', async (req, res) => {
  try {
    const { category, minPrice, maxPrice, page = 1, limit = 10 } = req.query
    
    const query = {}
    if (category) query.category = category
    if (minPrice || maxPrice) {
      query.price = {}
      if (minPrice) query.price.$gte = Number(minPrice)
      if (maxPrice) query.price.$lte = Number(maxPrice)
    }
    
    const products = await Product.find(query)
      .skip((page - 1) * limit)
      .limit(Number(limit))
    
    const total = await Product.countDocuments(query)
    
    res.json({
      products,
      pagination: {
        page: Number(page),
        limit: Number(limit),
        total,
        pages: Math.ceil(total / limit),
      },
    })
  } catch (error) {
    res.status(500).json({ error: 'Internal server error' })
  }
})

// Get product by ID
app.get('/products/:id', async (req, res) => {
  try {
    const product = await Product.findById(req.params.id)
    if (!product) return res.status(404).json({ error: 'Product not found' })
    res.json(product)
  } catch (error) {
    res.status(500).json({ error: 'Internal server error' })
  }
})

// Update stock (internal endpoint)
app.patch('/products/:id/stock', async (req, res) => {
  try {
    const { quantity, operation } = req.body
    // operation: 'decrease' หรือ 'increase'
    
    const product = await Product.findByIdAndUpdate(
      req.params.id,
      {
        $inc: { stock: operation === 'decrease' ? -quantity : quantity }
      },
      { new: true }
    )
    
    if (!product) return res.status(404).json({ error: 'Product not found' })
    
    res.json(product)
  } catch (error) {
    res.status(500).json({ error: 'Internal server error' })
  }
})

const PORT = process.env.PORT || 3002
app.listen(PORT, () => console.log(`Product service running on port ${PORT}`))
```

---

## Step 1554: API Gateway Pattern

**API Gateway** เป็น single entry point สำหรับ clients ทั้งหมด

```
Client → API Gateway → Services
                    ├── User Service
                    ├── Product Service
                    └── Order Service

Gateway ทำหน้าที่:
- Authentication/Authorization
- Rate limiting
- Load balancing
- Request/Response transformation
- Logging/Monitoring
- SSL termination
```

```javascript
// api-gateway/src/index.js
const express = require('express')
const httpProxy = require('http-proxy-middleware')
const rateLimit = require('express-rate-limit')
const jwt = require('jsonwebtoken')

const app = express()
app.use(express.json())

// Rate Limiting
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000,  // 15 minutes
  max: 100,
  message: { error: 'Too many requests' },
})
app.use('/api', limiter)

// Authentication middleware
function authMiddleware(req, res, next) {
  const token = req.headers.authorization?.split(' ')[1]
  
  if (!token) {
    return res.status(401).json({ error: 'No token provided' })
  }
  
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET)
    req.user = decoded
    next()
  } catch (error) {
    res.status(401).json({ error: 'Invalid token' })
  }
}

// Proxy configuration
const createProxy = (target) => httpProxy.createProxyMiddleware({
  target,
  changeOrigin: true,
  pathRewrite: { '^/api/users': '' },  // /api/users/1 → /users/1
  on: {
    error: (err, req, res) => {
      console.error('Proxy error:', err)
      res.status(503).json({ error: 'Service unavailable' })
    },
  },
})

// Public routes (ไม่ต้อง auth)
app.use('/api/users/register', createProxy(process.env.USER_SERVICE_URL))
app.use('/api/users/login', createProxy(process.env.USER_SERVICE_URL))
app.use('/api/products', createProxy(process.env.PRODUCT_SERVICE_URL))

// Protected routes (ต้อง auth)
app.use('/api/orders', authMiddleware, createProxy(process.env.ORDER_SERVICE_URL))
app.use('/api/users', authMiddleware, createProxy(process.env.USER_SERVICE_URL))

// Health check
app.get('/health', (req, res) => res.json({ status: 'healthy', service: 'api-gateway' }))

const PORT = process.env.PORT || 3000
app.listen(PORT, () => console.log(`API Gateway running on port ${PORT}`))
```

---

## Step 1555: Service-to-Service Communication (REST)

```javascript
// order-service/src/services/userService.js
const axios = require('axios')

class UserServiceClient {
  constructor() {
    this.baseUrl = process.env.USER_SERVICE_URL || 'http://user-service:3001'
    this.client = axios.create({
      baseURL: this.baseUrl,
      timeout: 5000,
    })
  }
  
  async getUser(userId) {
    try {
      const response = await this.client.get(`/users/${userId}`)
      return response.data
    } catch (error) {
      if (error.response?.status === 404) {
        return null
      }
      throw new Error(`User service error: ${error.message}`)
    }
  }
  
  async validateToken(token) {
    try {
      const response = await this.client.post('/users/validate', { token })
      return response.data
    } catch (error) {
      return null
    }
  }
}

module.exports = new UserServiceClient()
```

```javascript
// order-service/src/services/productService.js
const axios = require('axios')

class ProductServiceClient {
  constructor() {
    this.baseUrl = process.env.PRODUCT_SERVICE_URL || 'http://product-service:3002'
    this.client = axios.create({
      baseURL: this.baseUrl,
      timeout: 5000,
    })
  }
  
  async getProduct(productId) {
    const response = await this.client.get(`/products/${productId}`)
    return response.data
  }
  
  async decreaseStock(productId, quantity) {
    await this.client.patch(`/products/${productId}/stock`, {
      quantity,
      operation: 'decrease',
    })
  }
  
  async increaseStock(productId, quantity) {
    await this.client.patch(`/products/${productId}/stock`, {
      quantity,
      operation: 'increase',
    })
  }
}

module.exports = new ProductServiceClient()
```

```javascript
// order-service/src/index.js - ใช้ clients
const userServiceClient = require('./services/userService')
const productServiceClient = require('./services/productService')

app.post('/orders', async (req, res) => {
  const { userId, items } = req.body
  
  // Validate user
  const user = await userServiceClient.getUser(userId)
  if (!user) {
    return res.status(400).json({ error: 'User not found' })
  }
  
  // Validate and calculate products
  let totalAmount = 0
  for (const item of items) {
    const product = await productServiceClient.getProduct(item.productId)
    if (!product) {
      return res.status(400).json({ error: `Product ${item.productId} not found` })
    }
    if (product.stock < item.quantity) {
      return res.status(400).json({ error: `Insufficient stock for ${product.name}` })
    }
    totalAmount += product.price * item.quantity
  }
  
  // Create order in DB
  const order = await createOrder({ userId, items, totalAmount })
  
  // Update stock for each product
  for (const item of items) {
    await productServiceClient.decreaseStock(item.productId, item.quantity)
  }
  
  res.status(201).json(order)
})
```

---

## Step 1556: Message Queues - RabbitMQ

**Message Queue** ช่วยให้ services สื่อสารแบบ asynchronous (ไม่ต้องรอ response)

```
กรณีสั่งออเดอร์:
ก่อน (Synchronous):
  Order Service → ส่ง email → รอ... → ตอบ user
  (user ต้องรอ email send เสร็จ!)

หลัง (Asynchronous ด้วย Message Queue):
  Order Service → ส่ง message ลง queue → ตอบ user ทันที
  Email Service → รับ message จาก queue → ส่ง email (background)
```

```bash
# ติดตั้ง amqplib
npm install amqplib
```

```javascript
// shared/messageQueue.js
const amqp = require('amqplib')

class MessageQueue {
  constructor() {
    this.connection = null
    this.channel = null
    this.url = process.env.RABBITMQ_URL || 'amqp://rabbitmq:5672'
  }
  
  async connect() {
    this.connection = await amqp.connect(this.url)
    this.channel = await this.connection.createChannel()
    
    this.connection.on('error', (err) => {
      console.error('RabbitMQ connection error:', err)
      setTimeout(() => this.connect(), 5000)  // reconnect
    })
    
    console.log('Connected to RabbitMQ')
  }
  
  async publish(queue, message) {
    if (!this.channel) await this.connect()
    
    await this.channel.assertQueue(queue, { durable: true })
    
    this.channel.sendToQueue(
      queue,
      Buffer.from(JSON.stringify(message)),
      { persistent: true }  // message จะไม่หายเมื่อ RabbitMQ restart
    )
  }
  
  async subscribe(queue, handler) {
    if (!this.channel) await this.connect()
    
    await this.channel.assertQueue(queue, { durable: true })
    
    // prefetch 1: process ทีละ 1 message
    this.channel.prefetch(1)
    
    this.channel.consume(queue, async (msg) => {
      if (msg === null) return
      
      try {
        const message = JSON.parse(msg.content.toString())
        await handler(message)
        this.channel.ack(msg)  // acknowledge สำเร็จ
      } catch (error) {
        console.error('Error processing message:', error)
        this.channel.nack(msg, false, true)  // requeue message
      }
    })
  }
}

module.exports = new MessageQueue()
```

```javascript
// order-service/src/index.js - publish events
const mq = require('../../shared/messageQueue')

app.post('/orders', async (req, res) => {
  // ... create order ...
  
  // Publish events (async - ไม่ต้องรอ)
  await mq.publish('order.created', {
    orderId: order.id,
    userId: order.userId,
    items: order.items,
    totalAmount: order.totalAmount,
    createdAt: order.createdAt,
  })
  
  res.status(201).json(order)  // ตอบ user ทันที
})
```

```javascript
// notification-service/src/index.js - subscribe to events
const mq = require('../../shared/messageQueue')
const emailService = require('./emailService')

async function start() {
  await mq.subscribe('order.created', async (event) => {
    console.log('Processing order.created event:', event.orderId)
    
    // ส่ง email แบบ async
    await emailService.sendOrderConfirmation({
      userId: event.userId,
      orderId: event.orderId,
      items: event.items,
      totalAmount: event.totalAmount,
    })
    
    console.log(`Order confirmation email sent for order ${event.orderId}`)
  })
  
  console.log('Notification service listening for events...')
}

start().catch(console.error)
```

---

## Step 1557: Event-Driven Architecture

```javascript
// Event Bus ด้วย Redis Pub/Sub
const Redis = require('ioredis')

class EventBus {
  constructor() {
    this.publisher = new Redis(process.env.REDIS_URL)
    this.subscriber = new Redis(process.env.REDIS_URL)
    this.handlers = {}
    
    this.subscriber.on('message', (channel, message) => {
      const handlers = this.handlers[channel] || []
      const data = JSON.parse(message)
      handlers.forEach(handler => handler(data))
    })
  }
  
  async publish(event, data) {
    await this.publisher.publish(event, JSON.stringify(data))
  }
  
  subscribe(event, handler) {
    if (!this.handlers[event]) {
      this.handlers[event] = []
      this.subscriber.subscribe(event)
    }
    this.handlers[event].push(handler)
  }
}

const eventBus = new EventBus()

// User Service - publish user events
app.post('/users/register', async (req, res) => {
  const user = await createUser(req.body)
  
  // Publish event
  await eventBus.publish('user.registered', {
    userId: user.id,
    email: user.email,
    name: user.name,
  })
  
  res.status(201).json(user)
})

// Notification Service - subscribe to user events
eventBus.subscribe('user.registered', async (event) => {
  await sendWelcomeEmail(event.email, event.name)
})

// Analytics Service - subscribe to user events
eventBus.subscribe('user.registered', async (event) => {
  await trackNewUser(event.userId)
})
```

---

## Step 1558: CQRS Pattern

**CQRS** = Command Query Responsibility Segregation แยก read และ write operations

```
แบบเดิม (CRUD):
  Read และ Write ใช้ same model/database

CQRS:
  Commands (Write) → Command Handler → Write Database
  Queries (Read) → Query Handler → Read Database (optimized)
```

```javascript
// cqrs/userService.js

// ════════════════════════════
// Write Side (Commands)
// ════════════════════════════

class UserCommandHandler {
  constructor(writeDb, eventBus) {
    this.writeDb = writeDb
    this.eventBus = eventBus
  }
  
  async registerUser(command) {
    const { name, email, password } = command
    
    // Business logic validation
    const existing = await this.writeDb.query(
      'SELECT id FROM users WHERE email = $1', [email]
    )
    if (existing.rows.length > 0) {
      throw new Error('Email already exists')
    }
    
    // Write to DB
    const hashedPassword = await bcrypt.hash(password, 12)
    const result = await this.writeDb.query(
      'INSERT INTO users (name, email, password) VALUES ($1, $2, $3) RETURNING id',
      [name, email, hashedPassword]
    )
    
    const userId = result.rows[0].id
    
    // Publish domain event
    await this.eventBus.publish('UserRegistered', {
      userId,
      name,
      email,
      registeredAt: new Date(),
    })
    
    return userId
  }
  
  async updateProfile(command) {
    const { userId, name, phone } = command
    
    await this.writeDb.query(
      'UPDATE users SET name = $1, phone = $2 WHERE id = $3',
      [name, phone, userId]
    )
    
    await this.eventBus.publish('UserProfileUpdated', {
      userId, name, phone, updatedAt: new Date(),
    })
  }
}

// ════════════════════════════
// Read Side (Queries)
// ════════════════════════════

class UserQueryHandler {
  constructor(readDb) {
    this.readDb = readDb  // อาจเป็น read replica หรือ denormalized view
  }
  
  async getUserById(userId) {
    const result = await this.readDb.query(
      `SELECT 
        u.id, u.name, u.email, u.phone,
        COUNT(o.id) as total_orders,
        SUM(o.total_amount) as total_spent
       FROM users u
       LEFT JOIN orders o ON o.user_id = u.id
       WHERE u.id = $1
       GROUP BY u.id`,
      [userId]
    )
    return result.rows[0]
  }
  
  async searchUsers(query) {
    const result = await this.readDb.query(
      `SELECT id, name, email
       FROM users_search_view
       WHERE to_tsvector(name || ' ' || email) @@ to_tsquery($1)`,
      [query]
    )
    return result.rows
  }
}
```

---

## Step 1559: Saga Pattern - Distributed Transactions

**Saga Pattern** จัดการ distributed transactions ที่ครอบคลุมหลาย services

```
ปัญหา: สั่งออเดอร์ต้องทำหลาย operations:
1. Reserve stock (Product Service)
2. Charge payment (Payment Service)
3. Create order (Order Service)
4. Send notification (Notification Service)

ถ้า step 3 fail → ต้อง rollback step 1 และ 2
แต่เป็นคนละ database!
```

### Choreography Saga

```javascript
// choreography-saga.js
// ไม่มี central coordinator - แต่ละ service react ต่อ events

// Order Service - เริ่มต้น saga
app.post('/orders', async (req, res) => {
  const order = await createOrder({ status: 'PENDING', ...req.body })
  
  // ยิง event เพื่อเริ่ม saga
  await eventBus.publish('OrderCreated', {
    orderId: order.id,
    userId: order.userId,
    items: order.items,
    totalAmount: order.totalAmount,
  })
  
  res.status(202).json({ orderId: order.id, status: 'PENDING' })
})

// Product Service - step 1: reserve stock
eventBus.subscribe('OrderCreated', async (event) => {
  try {
    for (const item of event.items) {
      await reserveStock(item.productId, item.quantity)
    }
    
    await eventBus.publish('StockReserved', event)
  } catch (error) {
    // Stock reserve failed - compensate
    await eventBus.publish('StockReserveFailed', {
      ...event,
      reason: error.message,
    })
  }
})

// Payment Service - step 2: charge payment
eventBus.subscribe('StockReserved', async (event) => {
  try {
    const payment = await chargePayment(event.userId, event.totalAmount)
    
    await eventBus.publish('PaymentProcessed', {
      ...event,
      paymentId: payment.id,
    })
  } catch (error) {
    await eventBus.publish('PaymentFailed', {
      ...event,
      reason: error.message,
    })
  }
})

// Order Service - complete order
eventBus.subscribe('PaymentProcessed', async (event) => {
  await updateOrderStatus(event.orderId, 'COMPLETED')
  await eventBus.publish('OrderCompleted', event)
})

// ════ Compensating Transactions ════

// Product Service - release reserved stock เมื่อ payment fail
eventBus.subscribe('PaymentFailed', async (event) => {
  for (const item of event.items) {
    await releaseReservedStock(item.productId, item.quantity)
  }
  await eventBus.publish('StockReleased', event)
})

// Order Service - mark order as failed
eventBus.subscribe('StockReserveFailed', async (event) => {
  await updateOrderStatus(event.orderId, 'FAILED')
})

eventBus.subscribe('PaymentFailed', async (event) => {
  await updateOrderStatus(event.orderId, 'PAYMENT_FAILED')
})
```

---

## Step 1560: Circuit Breaker

**Circuit Breaker** ป้องกัน cascade failure เมื่อ service ใด service หนึ่งล้มเหลว

```
CLOSED (ปกติ):
  requests ผ่านปกติ
  นับ failures

OPEN (Circuit เปิด):
  service มีปัญหา
  reject requests ทันทีโดยไม่ส่งไปยัง service
  รอ timeout แล้วลอง HALF-OPEN

HALF-OPEN (ทดสอบ):
  ลอง request บางส่วน
  ถ้า succeed → กลับ CLOSED
  ถ้า fail → กลับ OPEN
```

```javascript
// circuitBreaker.js
class CircuitBreaker {
  constructor(options = {}) {
    this.failureThreshold = options.failureThreshold || 5
    this.recoveryTimeout = options.recoveryTimeout || 30000
    this.state = 'CLOSED'
    this.failureCount = 0
    this.lastFailureTime = null
    this.nextAttempt = Date.now()
  }
  
  async execute(fn) {
    if (this.state === 'OPEN') {
      if (Date.now() < this.nextAttempt) {
        throw new Error('Circuit breaker is OPEN - service unavailable')
      }
      // Try half-open
      this.state = 'HALF-OPEN'
    }
    
    try {
      const result = await fn()
      this.onSuccess()
      return result
    } catch (error) {
      this.onFailure()
      throw error
    }
  }
  
  onSuccess() {
    if (this.state === 'HALF-OPEN') {
      console.log('Circuit breaker: CLOSED')
    }
    this.failureCount = 0
    this.state = 'CLOSED'
  }
  
  onFailure() {
    this.failureCount++
    this.lastFailureTime = Date.now()
    
    if (this.failureCount >= this.failureThreshold || this.state === 'HALF-OPEN') {
      this.state = 'OPEN'
      this.nextAttempt = Date.now() + this.recoveryTimeout
      console.warn(`Circuit breaker: OPEN (will retry after ${this.recoveryTimeout}ms)`)
    }
  }
  
  getState() {
    return {
      state: this.state,
      failureCount: this.failureCount,
      nextAttempt: this.nextAttempt,
    }
  }
}

// ใช้งาน
const userServiceBreaker = new CircuitBreaker({
  failureThreshold: 3,
  recoveryTimeout: 30000,
})

async function getUser(userId) {
  return userServiceBreaker.execute(async () => {
    const response = await fetch(`http://user-service/users/${userId}`)
    if (!response.ok) throw new Error(`HTTP ${response.status}`)
    return response.json()
  })
}
```

---

## Step 1561: Service Discovery

**Service Discovery** ช่วยให้ services ค้นหากันเองโดยไม่ต้อง hardcode IPs

```javascript
// consul-service-discovery.js
const consul = require('consul')

class ServiceRegistry {
  constructor() {
    this.consul = new consul({
      host: process.env.CONSUL_HOST || 'consul',
      port: 8500,
    })
  }
  
  async register(service) {
    const { name, port, id } = service
    
    await this.consul.agent.service.register({
      id: id || `${name}-${port}`,
      name,
      port,
      address: process.env.SERVICE_HOST || 'localhost',
      check: {
        http: `http://localhost:${port}/health`,
        interval: '10s',
        timeout: '5s',
      },
      tags: ['nodejs'],
    })
    
    console.log(`Registered service: ${name} on port ${port}`)
  }
  
  async deregister(serviceId) {
    await this.consul.agent.service.deregister(serviceId)
  }
  
  async getService(name) {
    const services = await this.consul.health.service({
      service: name,
      passing: true,  // เฉพาะ healthy services
    })
    
    if (services.length === 0) {
      throw new Error(`No healthy instances of ${name}`)
    }
    
    // Load balancing: เลือกแบบ round-robin
    const instance = services[Math.floor(Math.random() * services.length)]
    const { Service } = instance
    
    return `http://${Service.Address}:${Service.Port}`
  }
}

const registry = new ServiceRegistry()

// Register เมื่อ start
async function start() {
  await registry.register({
    name: 'user-service',
    port: 3001,
  })
  
  // Deregister เมื่อ shutdown
  process.on('SIGTERM', async () => {
    await registry.deregister('user-service-3001')
    process.exit(0)
  })
}

// ใช้ service discovery
async function callUserService(userId) {
  const serviceUrl = await registry.getService('user-service')
  const response = await fetch(`${serviceUrl}/users/${userId}`)
  return response.json()
}
```

---

## Step 1562: Logging ใน Microservices

**Centralized Logging** รวม logs จากทุก services

```javascript
// shared/logger.js
const { createLogger, format, transports } = require('winston')

const logger = createLogger({
  format: format.combine(
    format.timestamp(),
    format.errors({ stack: true }),
    format.json(),
  ),
  
  defaultMeta: {
    service: process.env.SERVICE_NAME || 'unknown',
    version: process.env.SERVICE_VERSION || '1.0.0',
  },
  
  transports: [
    // Console (dev)
    new transports.Console({
      format: format.combine(
        format.colorize(),
        format.simple(),
      ),
    }),
    
    // File
    new transports.File({
      filename: 'logs/error.log',
      level: 'error',
    }),
    
    // ElasticSearch (production)
    // new transports.Elasticsearch({ ... })
  ],
})

module.exports = logger
```

```javascript
// Request logging middleware
const { v4: uuidv4 } = require('uuid')

function requestLogger(req, res, next) {
  const requestId = req.headers['x-request-id'] || uuidv4()
  req.requestId = requestId
  
  // ส่ง requestId ต่อไปยัง downstream services
  req.headers['x-request-id'] = requestId
  
  const start = Date.now()
  
  res.on('finish', () => {
    logger.info('HTTP Request', {
      requestId,
      method: req.method,
      url: req.url,
      statusCode: res.statusCode,
      duration: Date.now() - start,
      userAgent: req.get('user-agent'),
      ip: req.ip,
    })
  })
  
  next()
}
```

---

## Step 1563: Health Checks

```javascript
// shared/healthcheck.js
const { Pool } = require('pg')
const Redis = require('ioredis')

class HealthCheck {
  constructor() {
    this.checks = {}
  }
  
  addCheck(name, checkFn) {
    this.checks[name] = checkFn
  }
  
  async runAll() {
    const results = {}
    let isHealthy = true
    
    await Promise.all(
      Object.entries(this.checks).map(async ([name, fn]) => {
        const start = Date.now()
        try {
          await fn()
          results[name] = {
            status: 'healthy',
            responseTime: Date.now() - start,
          }
        } catch (error) {
          results[name] = {
            status: 'unhealthy',
            error: error.message,
            responseTime: Date.now() - start,
          }
          isHealthy = false
        }
      })
    )
    
    return { isHealthy, checks: results }
  }
}

// ใช้งานใน Express
function setupHealthChecks(app) {
  const health = new HealthCheck()
  
  // ตรวจสอบ PostgreSQL
  health.addCheck('database', async () => {
    const result = await pool.query('SELECT 1')
    if (!result) throw new Error('Database not responding')
  })
  
  // ตรวจสอบ Redis
  health.addCheck('redis', async () => {
    const result = await redis.ping()
    if (result !== 'PONG') throw new Error('Redis not responding')
  })
  
  // ตรวจสอบ external services
  health.addCheck('product-service', async () => {
    const response = await fetch(`${process.env.PRODUCT_SERVICE_URL}/health`)
    if (!response.ok) throw new Error('Product service unhealthy')
  })
  
  // Liveness probe: service ยัง run อยู่ไหม?
  app.get('/health/live', (req, res) => {
    res.json({ status: 'alive' })
  })
  
  // Readiness probe: service พร้อม serve requests ไหม?
  app.get('/health/ready', async (req, res) => {
    const result = await health.runAll()
    const status = result.isHealthy ? 200 : 503
    res.status(status).json(result)
  })
}
```

---

## Step 1564: Distributed Tracing

```javascript
// tracing.js - ด้วย OpenTelemetry
const { NodeSDK } = require('@opentelemetry/sdk-node')
const { JaegerExporter } = require('@opentelemetry/exporter-jaeger')
const { getNodeAutoInstrumentations } = require('@opentelemetry/auto-instrumentations-node')

const sdk = new NodeSDK({
  traceExporter: new JaegerExporter({
    endpoint: process.env.JAEGER_ENDPOINT || 'http://jaeger:14268/api/traces',
  }),
  
  instrumentations: [
    getNodeAutoInstrumentations({
      '@opentelemetry/instrumentation-express': { enabled: true },
      '@opentelemetry/instrumentation-http': { enabled: true },
      '@opentelemetry/instrumentation-pg': { enabled: true },
    }),
  ],
  
  serviceName: process.env.SERVICE_NAME,
})

sdk.start()

// Manual tracing
const { trace, context, propagation } = require('@opentelemetry/api')

async function processOrder(orderId) {
  const tracer = trace.getTracer('order-service')
  
  return tracer.startActiveSpan('processOrder', async (span) => {
    span.setAttribute('order.id', orderId)
    
    try {
      const order = await tracer.startActiveSpan('db.getOrder', async (dbSpan) => {
        const result = await db.query('SELECT * FROM orders WHERE id = $1', [orderId])
        dbSpan.end()
        return result.rows[0]
      })
      
      // Propagate trace context ไปยัง user service
      const headers = {}
      propagation.inject(context.active(), headers)
      
      const user = await fetch(`${USER_SERVICE_URL}/users/${order.userId}`, {
        headers,  // ← ส่ง trace context ไปด้วย
      })
      
      span.setStatus({ code: SpanStatusCode.OK })
      return { order, user }
    } catch (error) {
      span.recordException(error)
      span.setStatus({ code: SpanStatusCode.ERROR })
      throw error
    } finally {
      span.end()
    }
  })
}
```

---

## Step 1565: Microservices ด้วย Docker Compose

```yaml
# docker-compose.yml
version: '3.8'

services:
  # ─────────────────────────
  # API Gateway
  # ─────────────────────────
  api-gateway:
    build: ./api-gateway
    ports:
      - "3000:3000"
    environment:
      USER_SERVICE_URL: http://user-service:3001
      PRODUCT_SERVICE_URL: http://product-service:3002
      ORDER_SERVICE_URL: http://order-service:3003
      JWT_SECRET: ${JWT_SECRET}
    depends_on:
      - user-service
      - product-service
      - order-service
    networks:
      - microservices-net
  
  # ─────────────────────────
  # User Service
  # ─────────────────────────
  user-service:
    build: ./user-service
    environment:
      DATABASE_URL: postgresql://postgres:password@user-db:5432/users
      RABBITMQ_URL: amqp://rabbitmq:5672
      JWT_SECRET: ${JWT_SECRET}
      SERVICE_NAME: user-service
    depends_on:
      user-db:
        condition: service_healthy
      rabbitmq:
        condition: service_healthy
    networks:
      - microservices-net
  
  user-db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: users
      POSTGRES_PASSWORD: password
    volumes:
      - user_db_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres -d users"]
      interval: 5s
      retries: 5
    networks:
      - microservices-net
  
  # ─────────────────────────
  # Product Service
  # ─────────────────────────
  product-service:
    build: ./product-service
    environment:
      MONGODB_URL: mongodb://mongo:27017/products
      RABBITMQ_URL: amqp://rabbitmq:5672
      SERVICE_NAME: product-service
    depends_on:
      - mongo
      - rabbitmq
    networks:
      - microservices-net
  
  mongo:
    image: mongo:7
    volumes:
      - mongo_data:/data/db
    networks:
      - microservices-net
  
  # ─────────────────────────
  # Order Service
  # ─────────────────────────
  order-service:
    build: ./order-service
    environment:
      DATABASE_URL: postgresql://postgres:password@order-db:5432/orders
      RABBITMQ_URL: amqp://rabbitmq:5672
      USER_SERVICE_URL: http://user-service:3001
      PRODUCT_SERVICE_URL: http://product-service:3002
      SERVICE_NAME: order-service
    depends_on:
      order-db:
        condition: service_healthy
      rabbitmq:
        condition: service_healthy
    networks:
      - microservices-net
  
  order-db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: orders
      POSTGRES_PASSWORD: password
    volumes:
      - order_db_data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres -d orders"]
      interval: 5s
      retries: 5
    networks:
      - microservices-net
  
  # ─────────────────────────
  # Notification Service
  # ─────────────────────────
  notification-service:
    build: ./notification-service
    environment:
      RABBITMQ_URL: amqp://rabbitmq:5672
      SMTP_HOST: mailhog
      SMTP_PORT: 1025
      SERVICE_NAME: notification-service
    depends_on:
      - rabbitmq
    networks:
      - microservices-net
  
  # ─────────────────────────
  # Message Broker
  # ─────────────────────────
  rabbitmq:
    image: rabbitmq:3-management-alpine
    ports:
      - "5672:5672"
      - "15672:15672"  # Management UI
    environment:
      RABBITMQ_DEFAULT_USER: admin
      RABBITMQ_DEFAULT_PASS: password
    volumes:
      - rabbitmq_data:/var/lib/rabbitmq
    healthcheck:
      test: rabbitmq-diagnostics -q ping
      interval: 10s
      timeout: 5s
      retries: 5
    networks:
      - microservices-net
  
  # ─────────────────────────
  # Monitoring Stack
  # ─────────────────────────
  jaeger:
    image: jaegertracing/all-in-one:latest
    ports:
      - "16686:16686"  # Jaeger UI
      - "14268:14268"  # HTTP collector
    networks:
      - microservices-net
  
  prometheus:
    image: prom/prometheus:latest
    volumes:
      - ./docker/prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"
    networks:
      - microservices-net
  
  grafana:
    image: grafana/grafana:latest
    ports:
      - "3003:3000"
    volumes:
      - grafana_data:/var/lib/grafana
    networks:
      - microservices-net
  
  # ─────────────────────────
  # Dev Tools
  # ─────────────────────────
  mailhog:
    image: mailhog/mailhog
    ports:
      - "1025:1025"
      - "8025:8025"
    networks:
      - microservices-net

networks:
  microservices-net:
    driver: bridge

volumes:
  user_db_data:
  order_db_data:
  mongo_data:
  rabbitmq_data:
  grafana_data:
```

---

## Step 1566-1570: Testing Microservices

```javascript
// order-service/tests/integration/order.test.js
const request = require('supertest')
const app = require('../../src/app')
const nock = require('nock')  // Mock HTTP calls

describe('Order Service Integration Tests', () => {
  beforeEach(() => {
    // Mock User Service
    nock(process.env.USER_SERVICE_URL)
      .get('/users/user-1')
      .reply(200, {
        id: 'user-1',
        name: 'Test User',
        email: 'test@example.com',
      })
    
    // Mock Product Service
    nock(process.env.PRODUCT_SERVICE_URL)
      .get('/products/prod-1')
      .reply(200, {
        id: 'prod-1',
        name: 'Test Product',
        price: 100,
        stock: 10,
      })
      
      .patch('/products/prod-1/stock')
      .reply(200, { stock: 9 })
  })
  
  afterEach(() => {
    nock.cleanAll()
  })
  
  test('should create order successfully', async () => {
    const response = await request(app)
      .post('/orders')
      .set('Authorization', 'Bearer test-token')
      .send({
        userId: 'user-1',
        items: [{ productId: 'prod-1', quantity: 1 }],
      })
    
    expect(response.status).toBe(201)
    expect(response.body).toMatchObject({
      userId: 'user-1',
      status: 'PENDING',
      totalAmount: 100,
    })
  })
  
  test('should fail if product out of stock', async () => {
    nock.cleanAll()
    
    nock(process.env.PRODUCT_SERVICE_URL)
      .get('/products/prod-2')
      .reply(200, {
        id: 'prod-2',
        name: 'Out of Stock',
        price: 100,
        stock: 0,  // ← ไม่มี stock
      })
    
    const response = await request(app)
      .post('/orders')
      .send({
        userId: 'user-1',
        items: [{ productId: 'prod-2', quantity: 1 }],
      })
    
    expect(response.status).toBe(400)
    expect(response.body.error).toContain('Insufficient stock')
  })
})
```

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: สร้าง 2 Microservices

1. **User Service**: register, login, get user
2. **Post Service**: CRUD posts พร้อม ownership check

Services ต้องสื่อสารกันผ่าน REST

### แบบฝึกหัดที่ 2: เพิ่ม Message Queue

1. เพิ่ม RabbitMQ
2. เมื่อ user สร้าง post → publish event
3. สร้าง notification service ที่ subscribe event นี้

### แบบฝึกหัดที่ 3: Circuit Breaker

ใช้ Circuit Breaker pattern เมื่อ call ระหว่าง services

---

## สรุป Part 79

1. **Microservices** แยก app เป็น services เล็กๆ ที่ทำงานอิสระ
2. **API Gateway**: single entry point, routing, auth
3. **REST communication**: HTTP calls ระหว่าง services
4. **Message Queues**: async communication ด้วย RabbitMQ
5. **CQRS**: แยก read/write concerns
6. **Saga Pattern**: distributed transactions
7. **Circuit Breaker**: ป้องกัน cascade failures
8. **Service Discovery**: ค้นหา services แบบ dynamic
9. **Distributed Tracing**: trace requests ข้ามหลาย services
