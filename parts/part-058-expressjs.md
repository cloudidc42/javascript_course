# Part 58: Express.js (Steps 1131-1150)

## บทนำ

Express.js เป็น web framework ยอดนิยมสำหรับ Node.js ที่ช่วยให้สร้าง web servers และ APIs ได้อย่างรวดเร็วและมีประสิทธิภาพ Express มีขนาดเล็ก (minimal) แต่ยืดหยุ่นสูง (flexible) เป็น "unopinionated" framework ที่ไม่บังคับโครงสร้าง

---

## Step 1131: Express.js คืออะไร

```javascript
// ทำไมต้องใช้ Express แทน http module โดยตรง?

// ===== ด้วย http module เพียงอย่างเดียว =====
const http = require('http');

const server = http.createServer((req, res) => {
  if (req.url === '/users' && req.method === 'GET') {
    res.setHeader('Content-Type', 'application/json');
    res.statusCode = 200;
    res.end(JSON.stringify([{ id: 1, name: 'Alice' }]));
  } else if (req.url.match(/^\/users\/\d+$/) && req.method === 'GET') {
    const id = parseInt(req.url.split('/')[2]);
    res.setHeader('Content-Type', 'application/json');
    res.statusCode = 200;
    res.end(JSON.stringify({ id, name: 'Alice' }));
  } else if (req.url === '/users' && req.method === 'POST') {
    let body = '';
    req.on('data', chunk => body += chunk);
    req.on('end', () => {
      const data = JSON.parse(body);
      res.setHeader('Content-Type', 'application/json');
      res.statusCode = 201;
      res.end(JSON.stringify({ ...data, id: 2 }));
    });
  } else {
    res.statusCode = 404;
    res.end(JSON.stringify({ error: 'Not Found' }));
  }
});

// ===== ด้วย Express =====
const express = require('express');
const app = express();

app.use(express.json());

app.get('/users', (req, res) => {
  res.json([{ id: 1, name: 'Alice' }]);
});

app.get('/users/:id', (req, res) => {
  res.json({ id: req.params.id, name: 'Alice' });
});

app.post('/users', (req, res) => {
  res.status(201).json({ ...req.body, id: 2 });
});

// Express ดีกว่าเพราะ:
// 1. Routing ง่ายกว่า
// 2. Middleware ecosystem
// 3. req.params, req.query, req.body ทำงานอัตโนมัติ
// 4. Response helpers (res.json, res.status)
// 5. Error handling ดีกว่า
```

---

## Step 1132: Installing และ Setting Up Express

```bash
# สร้างโปรเจค
mkdir my-express-app
cd my-express-app
npm init -y

# ติดตั้ง Express
npm install express

# ติดตั้ง development tools
npm install --save-dev nodemon

# เพิ่ม scripts ใน package.json
```

```json
{
  "scripts": {
    "start": "node src/index.js",
    "dev": "nodemon src/index.js",
    "test": "jest"
  }
}
```

```javascript
// src/index.js - โครงสร้างพื้นฐาน
const express = require('express');

// สร้าง Express application
const app = express();

// กำหนด port
const PORT = process.env.PORT || 3000;

// Middleware
app.use(express.json());
app.use(express.urlencoded({ extended: true }));

// Routes
app.get('/', (req, res) => {
  res.json({ message: 'Welcome to Express!' });
});

// Start server
app.listen(PORT, () => {
  console.log(`Server is running on http://localhost:${PORT}`);
});

module.exports = app; // สำหรับ testing
```

```bash
# รัน server
npm run dev

# ทดสอบ
curl http://localhost:3000
# {"message":"Welcome to Express!"}
```

---

## Step 1133: Basic Routes

```javascript
const express = require('express');
const app = express();

app.use(express.json());

// ===== HTTP Methods =====

// GET - ดึงข้อมูล
app.get('/hello', (req, res) => {
  res.send('Hello, World!');
});

// POST - สร้างข้อมูล
app.post('/users', (req, res) => {
  const { name, email } = req.body;
  // สร้าง user
  res.status(201).json({ id: 1, name, email });
});

// PUT - อัปเดตข้อมูลทั้งหมด (replace)
app.put('/users/:id', (req, res) => {
  const { id } = req.params;
  const data = req.body;
  res.json({ id: parseInt(id), ...data });
});

// PATCH - อัปเดตบางส่วน
app.patch('/users/:id', (req, res) => {
  const { id } = req.params;
  const updates = req.body;
  res.json({ id: parseInt(id), ...updates });
});

// DELETE - ลบข้อมูล
app.delete('/users/:id', (req, res) => {
  const { id } = req.params;
  res.status(204).send(); // No Content
});

// ===== Multiple HTTP methods =====
app.route('/products')
  .get((req, res) => {
    res.json({ products: [] });
  })
  .post((req, res) => {
    res.status(201).json({ ...req.body, id: 1 });
  });

// ===== All methods =====
app.all('/api/*', (req, res, next) => {
  console.log(`${req.method} ${req.path}`);
  next();
});

// ===== Route dengan pattern =====
// ? = optional character
app.get('/colo(u?)r', (req, res) => {
  res.send('color or colour');
});

// * = wildcard
app.get('/search/*', (req, res) => {
  res.send(`Searching for: ${req.params[0]}`);
});

// RegEx
app.get(/^\/api\/v\d+\/.*$/, (req, res) => {
  res.send('API route matched');
});

app.listen(3000);
```

---

## Step 1134: Route Parameters

```javascript
const express = require('express');
const app = express();

app.use(express.json());

// ===== req.params =====
// URL parameters (dynamic segments)

// Single param
app.get('/users/:id', (req, res) => {
  const { id } = req.params;
  console.log('User ID:', id); // เป็น string เสมอ
  
  // แปลงเป็น number ถ้าต้องการ
  const userId = parseInt(id);
  
  if (isNaN(userId)) {
    return res.status(400).json({ error: 'Invalid user ID' });
  }
  
  res.json({ id: userId, name: 'Alice' });
});

// Multiple params
app.get('/users/:userId/posts/:postId', (req, res) => {
  const { userId, postId } = req.params;
  res.json({ userId, postId });
});

// Optional params (ใช้ ?)
app.get('/users/:id/profile/:section?', (req, res) => {
  const { id, section = 'basic' } = req.params;
  res.json({ id, section });
});

// Wildcard
app.get('/files/*', (req, res) => {
  const filePath = req.params[0]; // path หลัง /files/
  res.json({ path: filePath });
});

// ===== ตัวอย่างจริง: CRUD API =====
let users = [
  { id: 1, name: 'Alice', email: 'alice@example.com' },
  { id: 2, name: 'Bob', email: 'bob@example.com' }
];

// GET /users - ดึงทุก users
app.get('/users', (req, res) => {
  res.json(users);
});

// GET /users/:id - ดึง user เดียว
app.get('/users/:id', (req, res) => {
  const id = parseInt(req.params.id);
  const user = users.find(u => u.id === id);
  
  if (!user) {
    return res.status(404).json({ error: 'User not found' });
  }
  
  res.json(user);
});

// POST /users - สร้าง user ใหม่
app.post('/users', (req, res) => {
  const { name, email } = req.body;
  
  if (!name || !email) {
    return res.status(400).json({ error: 'name and email are required' });
  }
  
  const newUser = {
    id: Math.max(...users.map(u => u.id)) + 1,
    name,
    email
  };
  
  users.push(newUser);
  res.status(201).json(newUser);
});

// PUT /users/:id - อัปเดต user ทั้งหมด
app.put('/users/:id', (req, res) => {
  const id = parseInt(req.params.id);
  const index = users.findIndex(u => u.id === id);
  
  if (index === -1) {
    return res.status(404).json({ error: 'User not found' });
  }
  
  const { name, email } = req.body;
  users[index] = { id, name, email };
  
  res.json(users[index]);
});

// PATCH /users/:id - อัปเดตบางส่วน
app.patch('/users/:id', (req, res) => {
  const id = parseInt(req.params.id);
  const index = users.findIndex(u => u.id === id);
  
  if (index === -1) {
    return res.status(404).json({ error: 'User not found' });
  }
  
  users[index] = { ...users[index], ...req.body };
  res.json(users[index]);
});

// DELETE /users/:id
app.delete('/users/:id', (req, res) => {
  const id = parseInt(req.params.id);
  const index = users.findIndex(u => u.id === id);
  
  if (index === -1) {
    return res.status(404).json({ error: 'User not found' });
  }
  
  users.splice(index, 1);
  res.status(204).send();
});

app.listen(3000);
```

---

## Step 1135: Query Strings

```javascript
const express = require('express');
const app = express();

// ===== req.query =====
// URL: /users?page=2&limit=10&sort=name&order=asc&search=alice

app.get('/users', (req, res) => {
  const {
    page = 1,
    limit = 10,
    sort = 'id',
    order = 'asc',
    search = ''
  } = req.query;
  
  // แปลงประเภท
  const pageNum = parseInt(page);
  const limitNum = parseInt(limit);
  
  if (isNaN(pageNum) || pageNum < 1) {
    return res.status(400).json({ error: 'Invalid page' });
  }
  
  console.log({ page: pageNum, limit: limitNum, sort, order, search });
  
  // จำลองการ filter/sort/paginate
  let result = [...users];
  
  if (search) {
    result = result.filter(u => 
      u.name.toLowerCase().includes(search.toLowerCase())
    );
  }
  
  result.sort((a, b) => {
    const comparison = a[sort] > b[sort] ? 1 : -1;
    return order === 'desc' ? -comparison : comparison;
  });
  
  const total = result.length;
  const offset = (pageNum - 1) * limitNum;
  const data = result.slice(offset, offset + limitNum);
  
  res.json({
    data,
    pagination: {
      page: pageNum,
      limit: limitNum,
      total,
      totalPages: Math.ceil(total / limitNum)
    }
  });
});

// ===== Array query params =====
// URL: /filter?tags=nodejs&tags=express&tags=api
app.get('/filter', (req, res) => {
  const tags = [].concat(req.query.tags || []); // รับเป็น array เสมอ
  res.json({ tags });
});

// ===== Multiple values =====
// URL: /search?min=100&max=500&category=electronics
app.get('/search', (req, res) => {
  const { min, max, category } = req.query;
  
  const filters = {};
  if (min) filters.minPrice = parseFloat(min);
  if (max) filters.maxPrice = parseFloat(max);
  if (category) filters.category = category;
  
  res.json({ filters });
});

app.listen(3000);
```

---

## Step 1136: Request Body

```javascript
const express = require('express');
const app = express();

// ===== Built-in body parsers =====

// JSON body parser
app.use(express.json({ limit: '10mb' }));  // จำกัด request size

// URL-encoded body parser (form data)
app.use(express.urlencoded({
  extended: true,   // ใช้ qs library (รองรับ nested objects)
  limit: '10mb'
}));

// Raw text parser (for webhooks, etc.)
app.use(express.text());

// Raw Buffer
app.use(express.raw({ type: 'application/octet-stream' }));

// ===== req.body =====
app.post('/users', (req, res) => {
  console.log('Content-Type:', req.headers['content-type']);
  console.log('Body:', req.body);
  
  // req.body คือ parsed body
  const { name, email, password } = req.body;
  
  res.status(201).json({ name, email });
});

// ===== Nested objects =====
// POST /orders
// Body: { "user": { "name": "Alice" }, "items": [{ "id": 1, "qty": 2 }] }
app.post('/orders', (req, res) => {
  const { user, items, shipping } = req.body;
  
  if (!user || !items || items.length === 0) {
    return res.status(400).json({ error: 'user and items are required' });
  }
  
  const order = {
    id: Date.now(),
    user,
    items,
    shipping,
    total: items.reduce((sum, item) => sum + item.price * item.qty, 0),
    createdAt: new Date()
  };
  
  res.status(201).json(order);
});

// ===== Form data =====
// Content-Type: application/x-www-form-urlencoded
app.post('/form', (req, res) => {
  const { username, password } = req.body;
  res.json({ received: { username } });
});

// ===== Raw body สำหรับ webhook =====
app.post('/webhook', express.raw({ type: '*/*' }), (req, res) => {
  const signature = req.headers['x-signature'];
  const body = req.body; // Buffer
  
  // ตรวจสอบ signature
  const crypto = require('crypto');
  const expected = crypto
    .createHmac('sha256', process.env.WEBHOOK_SECRET)
    .update(body)
    .digest('hex');
  
  if (signature !== `sha256=${expected}`) {
    return res.status(401).json({ error: 'Invalid signature' });
  }
  
  const event = JSON.parse(body.toString());
  console.log('Webhook event:', event);
  
  res.json({ received: true });
});

app.listen(3000);
```

---

## Step 1137: Middleware Concept

```javascript
const express = require('express');
const app = express();

// ===== Middleware คืออะไร? =====
// Middleware คือ function ที่มีการเข้าถึง req, res, next
// สามารถ:
// 1. Execute any code
// 2. Modify req/res
// 3. End the request-response cycle
// 4. Call next middleware

// Middleware signature:
// (req, res, next) => { ... }

// ===== Application-level middleware =====
// รันทุก request
app.use((req, res, next) => {
  console.log(`[${new Date().toISOString()}] ${req.method} ${req.path}`);
  next(); // ต้องเรียก next() เสมอ ไม่งั้น request จะ hang
});

// ===== Middleware order สำคัญมาก! =====
app.use(express.json()); // ต้องอยู่ก่อน routes ที่ใช้ req.body

// ===== Path-specific middleware =====
app.use('/api', (req, res, next) => {
  console.log('API request');
  next();
});

// ===== Route-specific middleware =====
function authenticate(req, res, next) {
  const token = req.headers.authorization?.replace('Bearer ', '');
  
  if (!token) {
    return res.status(401).json({ error: 'No token provided' });
  }
  
  try {
    // ตรวจสอบ token
    const user = verifyToken(token);
    req.user = user; // เพิ่ม user เข้า req
    next();
  } catch (err) {
    res.status(401).json({ error: 'Invalid token' });
  }
}

app.get('/profile', authenticate, (req, res) => {
  res.json({ user: req.user });
});

// ===== Multiple middleware =====
app.get(
  '/admin/dashboard',
  authenticate,
  requireRole('admin'),
  (req, res) => {
    res.json({ dashboard: 'Admin data' });
  }
);

function requireRole(role) {
  return (req, res, next) => {
    if (req.user?.role !== role) {
      return res.status(403).json({ error: 'Forbidden' });
    }
    next();
  };
}

// ===== Middleware array =====
const authMiddleware = [authenticate, requireRole('admin')];
app.get('/admin/users', authMiddleware, (req, res) => {
  res.json({ users: [] });
});

app.listen(3000);
```

---

## Step 1138: Custom Middleware

```javascript
const express = require('express');
const app = express();

app.use(express.json());

// ===== Logging Middleware =====
function logger(options = {}) {
  const { format = 'combined' } = options;
  
  return (req, res, next) => {
    const start = Date.now();
    
    // Override res.end เพื่อ capture status code
    const originalEnd = res.end;
    res.end = function(...args) {
      const duration = Date.now() - start;
      const log = `${new Date().toISOString()} - ${req.method} ${req.originalUrl} - ${res.statusCode} - ${duration}ms`;
      console.log(log);
      originalEnd.apply(res, args);
    };
    
    next();
  };
}

app.use(logger());

// ===== Request ID Middleware =====
const { randomUUID } = require('crypto');

function requestId(req, res, next) {
  const id = req.headers['x-request-id'] || randomUUID();
  req.id = id;
  res.setHeader('X-Request-ID', id);
  next();
}

app.use(requestId);

// ===== Rate Limiting (simple) =====
function rateLimiter(options = {}) {
  const { windowMs = 60000, max = 100 } = options;
  const requests = new Map();
  
  return (req, res, next) => {
    const ip = req.ip;
    const now = Date.now();
    const windowStart = now - windowMs;
    
    // ล้าง requests เก่า
    if (requests.has(ip)) {
      const timestamps = requests.get(ip).filter(t => t > windowStart);
      requests.set(ip, timestamps);
    }
    
    const timestamps = requests.get(ip) || [];
    
    if (timestamps.length >= max) {
      return res.status(429).json({
        error: 'Too Many Requests',
        retryAfter: Math.ceil((timestamps[0] + windowMs - now) / 1000)
      });
    }
    
    timestamps.push(now);
    requests.set(ip, timestamps);
    
    res.setHeader('X-RateLimit-Limit', max);
    res.setHeader('X-RateLimit-Remaining', max - timestamps.length);
    
    next();
  };
}

app.use('/api/', rateLimiter({ windowMs: 60000, max: 60 }));

// ===== CORS Middleware (manual) =====
function cors(options = {}) {
  const {
    origin = '*',
    methods = 'GET,HEAD,PUT,PATCH,POST,DELETE',
    credentials = false
  } = options;
  
  return (req, res, next) => {
    const requestOrigin = req.headers.origin;
    
    if (origin === '*') {
      res.setHeader('Access-Control-Allow-Origin', '*');
    } else if (Array.isArray(origin)) {
      if (origin.includes(requestOrigin)) {
        res.setHeader('Access-Control-Allow-Origin', requestOrigin);
      }
    } else if (typeof origin === 'function') {
      const allowed = origin(requestOrigin);
      if (allowed) {
        res.setHeader('Access-Control-Allow-Origin', requestOrigin);
      }
    } else {
      res.setHeader('Access-Control-Allow-Origin', origin);
    }
    
    res.setHeader('Access-Control-Allow-Methods', methods);
    res.setHeader('Access-Control-Allow-Headers', 'Content-Type, Authorization');
    
    if (credentials) {
      res.setHeader('Access-Control-Allow-Credentials', 'true');
    }
    
    // Handle preflight
    if (req.method === 'OPTIONS') {
      return res.status(204).send();
    }
    
    next();
  };
}

app.use(cors({ origin: ['http://localhost:3000', 'https://myapp.com'] }));

// ===== Auth Middleware =====
const jwt = require('jsonwebtoken'); // npm install jsonwebtoken

function authenticate(req, res, next) {
  const authHeader = req.headers.authorization;
  
  if (!authHeader || !authHeader.startsWith('Bearer ')) {
    return res.status(401).json({ error: 'Unauthorized' });
  }
  
  const token = authHeader.slice(7);
  
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = decoded;
    next();
  } catch (err) {
    if (err.name === 'TokenExpiredError') {
      return res.status(401).json({ error: 'Token expired' });
    }
    res.status(401).json({ error: 'Invalid token' });
  }
}

// ===== Request Validation Middleware =====
function validate(schema) {
  return (req, res, next) => {
    const { error, value } = schema.validate(req.body, { abortEarly: false });
    
    if (error) {
      const errors = error.details.map(detail => ({
        field: detail.path.join('.'),
        message: detail.message
      }));
      
      return res.status(422).json({ errors });
    }
    
    req.body = value; // use validated/sanitized value
    next();
  };
}

app.listen(3000);
```

---

## Step 1139: Error Handling Middleware

```javascript
const express = require('express');
const app = express();

app.use(express.json());

// ===== Error-handling middleware =====
// มี 4 parameters: (err, req, res, next)
// ต้องอยู่หลัง routes ทั้งหมด

// Custom error class
class AppError extends Error {
  constructor(message, statusCode = 500, code = 'INTERNAL_ERROR') {
    super(message);
    this.statusCode = statusCode;
    this.code = code;
    this.isOperational = true;
  }
}

// Routes (throw errors)
app.get('/users/:id', (req, res, next) => {
  try {
    const id = parseInt(req.params.id);
    
    if (isNaN(id)) {
      throw new AppError('Invalid user ID', 400, 'INVALID_ID');
    }
    
    // จำลอง: user ไม่พบ
    if (id > 100) {
      throw new AppError(`User ${id} not found`, 404, 'USER_NOT_FOUND');
    }
    
    res.json({ id, name: 'Alice' });
  } catch (err) {
    next(err); // ส่ง error ไปยัง error handler
  }
});

// Async error handling
app.get('/async-example', async (req, res, next) => {
  try {
    // async operations
    const data = await fetchSomeData();
    res.json(data);
  } catch (err) {
    next(err);
  }
});

// wrapper สำหรับ async routes
const asyncHandler = (fn) => (req, res, next) => {
  Promise.resolve(fn(req, res, next)).catch(next);
};

app.get('/users', asyncHandler(async (req, res) => {
  const users = await User.findAll(); // ถ้า throw ก็จะไปที่ next(err) อัตโนมัติ
  res.json(users);
}));

// ===== 404 Handler =====
// อยู่หลัง routes ทั้งหมด
app.use((req, res, next) => {
  next(new AppError(`Route ${req.method} ${req.originalUrl} not found`, 404, 'ROUTE_NOT_FOUND'));
});

// ===== Global Error Handler =====
// ต้องอยู่ท้ายสุด
app.use((err, req, res, next) => {
  // Log error
  if (process.env.NODE_ENV !== 'test') {
    console.error(`[ERROR] ${err.message}`, {
      stack: err.stack,
      requestId: req.id,
      method: req.method,
      url: req.originalUrl
    });
  }
  
  // กำหนด status code
  const statusCode = err.statusCode || err.status || 500;
  
  // กำหนด response
  const response = {
    error: {
      message: err.message,
      code: err.code || 'INTERNAL_ERROR',
      requestId: req.id
    }
  };
  
  // เพิ่ม stack trace ใน development
  if (process.env.NODE_ENV === 'development') {
    response.error.stack = err.stack;
  }
  
  res.status(statusCode).json(response);
});

// ===== Error types =====
app.get('/examples/errors', (req, res, next) => {
  const { type } = req.query;
  
  switch (type) {
    case 'validation':
      throw new AppError('Name is required', 422, 'VALIDATION_ERROR');
    case 'auth':
      throw new AppError('Unauthorized', 401, 'UNAUTHORIZED');
    case 'forbidden':
      throw new AppError('Forbidden', 403, 'FORBIDDEN');
    case 'not-found':
      throw new AppError('Resource not found', 404, 'NOT_FOUND');
    case 'conflict':
      throw new AppError('Email already exists', 409, 'CONFLICT');
    default:
      throw new Error('Unexpected error');
  }
});

app.listen(3000);
```

---

## Step 1140: Router (express.Router)

```javascript
const express = require('express');

// ===== Express Router =====
// แบ่ง routes ออกเป็น modules

// routes/users.js
const router = express.Router();

// Middleware เฉพาะ router นี้
router.use((req, res, next) => {
  console.log('Users router middleware');
  next();
});

// Routes
router.get('/', (req, res) => {
  res.json({ users: [] });
});

router.post('/', (req, res) => {
  res.status(201).json({ ...req.body, id: 1 });
});

router.get('/:id', (req, res) => {
  res.json({ id: req.params.id });
});

router.put('/:id', (req, res) => {
  res.json({ id: req.params.id, ...req.body });
});

router.delete('/:id', (req, res) => {
  res.status(204).send();
});

module.exports = router;

// routes/posts.js
const postsRouter = express.Router({ mergeParams: true }); // inherit params จาก parent

postsRouter.get('/', (req, res) => {
  const { userId } = req.params; // จาก parent route
  res.json({ userId, posts: [] });
});

postsRouter.post('/', (req, res) => {
  const { userId } = req.params;
  res.status(201).json({ userId, ...req.body });
});

module.exports = postsRouter;

// app.js
const express = require('express');
const app = express();

app.use(express.json());

const usersRouter = require('./routes/users');
const postsRouter = require('./routes/posts');

// Mount routers
app.use('/api/v1/users', usersRouter);
app.use('/api/v1/users/:userId/posts', postsRouter);

// ===== โครงสร้างโปรเจค =====
/*
src/
  routes/
    index.js          - รวม routes ทั้งหมด
    users.js          - user routes
    posts.js          - post routes
    auth.js           - auth routes
  controllers/
    userController.js
    postController.js
    authController.js
  middleware/
    auth.js
    validate.js
    logger.js
  models/
    User.js
    Post.js
  app.js
  server.js
*/

// routes/index.js
const express = require('express');
const router = express.Router();

router.use('/auth', require('./auth'));
router.use('/users', require('./users'));
router.use('/posts', require('./posts'));

module.exports = router;

// app.js
const express = require('express');
const app = express();

app.use(express.json());
app.use('/api/v1', require('./routes'));

module.exports = app;

// server.js
const app = require('./app');

app.listen(3000, () => {
  console.log('Server started on port 3000');
});

app.listen(3000);
```

---

## Step 1141: Serving Static Files

```javascript
const express = require('express');
const path = require('path');
const app = express();

// ===== express.static() =====
// Serve static files from 'public' directory

app.use(express.static(path.join(__dirname, 'public')));
// เข้าถึงได้: http://localhost:3000/images/photo.jpg
// จากไฟล์: public/images/photo.jpg

// Virtual path prefix
app.use('/static', express.static(path.join(__dirname, 'public')));
// เข้าถึงได้: http://localhost:3000/static/css/style.css

// Options
app.use(express.static('public', {
  maxAge: '1d',           // cache 1 วัน
  etag: true,             // ETag headers
  lastModified: true,     // Last-Modified headers
  index: 'index.html',    // default file
  dotfiles: 'ignore',     // ซ่อน dotfiles
  extensions: ['html', 'htm'], // ลอง extensions ต่างๆ
  fallthrough: true       // ถ้าไม่พบ ให้ต่อไปยัง next middleware
}));

// Multiple static directories
app.use(express.static('public'));
app.use(express.static('uploads'));
app.use(express.static('assets'));

// ===== SPA (Single Page Application) =====
// Serve React/Vue/Angular app

app.use(express.static(path.join(__dirname, 'client/build')));

// Handle React Router (client-side routing)
app.get('*', (req, res) => {
  // เฉพาะ request ที่ไม่ใช่ API
  if (!req.path.startsWith('/api')) {
    res.sendFile(path.join(__dirname, 'client/build', 'index.html'));
  }
});

// ===== ตัวอย่าง: File download =====
app.get('/download/:filename', (req, res) => {
  const { filename } = req.params;
  const filePath = path.join(__dirname, 'downloads', filename);
  
  // ตรวจสอบ path traversal
  const normalizedPath = path.normalize(filePath);
  if (!normalizedPath.startsWith(path.join(__dirname, 'downloads'))) {
    return res.status(400).json({ error: 'Invalid file path' });
  }
  
  res.download(filePath, filename, (err) => {
    if (err) {
      if (!res.headersSent) {
        res.status(404).json({ error: 'File not found' });
      }
    }
  });
});

app.listen(3000);
```

---

## Step 1142: Template Engines

```javascript
const express = require('express');
const path = require('path');
const app = express();

// ===== EJS (Embedded JavaScript) =====
// npm install ejs

app.set('view engine', 'ejs');
app.set('views', path.join(__dirname, 'views'));

// views/index.ejs:
/*
<!DOCTYPE html>
<html>
<head><title><%= title %></title></head>
<body>
  <h1><%= title %></h1>
  <% if (user) { %>
    <p>Welcome, <%= user.name %>!</p>
  <% } %>
  <ul>
    <% items.forEach(item => { %>
      <li><%= item.name %> - <%- item.description %></li>
    <% }); %>
  </ul>
  <%- include('partials/footer') %>
</body>
</html>
*/

app.get('/', (req, res) => {
  res.render('index', {
    title: 'My Express App',
    user: { name: 'Alice' },
    items: [
      { name: 'Item 1', description: '<b>Bold item</b>' },
      { name: 'Item 2', description: 'Regular item' }
    ]
  });
});

// EJS tags:
// <%= value %>   - output (HTML escaped)
// <%- value %>   - output (unescaped, HTML safe)
// <% code %>     - JavaScript code (no output)
// <%# comment %> - comment

// ===== Handlebars =====
// npm install express-handlebars

const { engine } = require('express-handlebars');

app.engine('handlebars', engine({
  defaultLayout: 'main',
  helpers: {
    formatDate: (date) => new Date(date).toLocaleDateString('th-TH'),
    uppercase: (str) => str.toUpperCase()
  }
}));
app.set('view engine', 'handlebars');

// views/home.handlebars:
/*
<h1>{{title}}</h1>
{{#if user}}
  <p>Hello, {{user.name}}</p>
{{/if}}
{{#each items}}
  <li>{{this.name}}: {{formatDate this.date}}</li>
{{/each}}
*/

app.get('/home', (req, res) => {
  res.render('home', {
    title: 'Home Page',
    user: { name: 'Alice' },
    items: [{ name: 'Post 1', date: new Date() }]
  });
});

app.listen(3000);
```

---

## Step 1143: Response Methods

```javascript
const express = require('express');
const path = require('path');
const app = express();

app.use(express.json());

// ===== Response methods =====

// res.send() - ส่ง response
app.get('/send-text', (req, res) => {
  res.send('Hello, World!'); // text/html
});

app.get('/send-html', (req, res) => {
  res.send('<h1>Hello, World!</h1>');
});

app.get('/send-buffer', (req, res) => {
  res.send(Buffer.from('Hello'));
});

// res.json() - ส่ง JSON
app.get('/json', (req, res) => {
  res.json({
    status: 'success',
    data: { id: 1, name: 'Alice' }
  });
});

// res.status() - กำหนด HTTP status
app.post('/users', (req, res) => {
  res.status(201).json({ id: 1, ...req.body });
});

app.delete('/users/:id', (req, res) => {
  res.status(204).send(); // No Content
});

// res.redirect() - redirect
app.get('/old-path', (req, res) => {
  res.redirect('/new-path');           // 302 by default
  // res.redirect(301, '/new-path');   // 301 Permanent
  // res.redirect('back');             // redirect ไปหน้าก่อน
});

// res.render() - render template
app.get('/page', (req, res) => {
  res.render('index', { title: 'My Page' });
});

// res.download() - download file
app.get('/download', (req, res) => {
  const file = path.join(__dirname, 'files', 'document.pdf');
  res.download(file, 'my-document.pdf'); // filename ที่ client จะเห็น
});

// res.sendFile() - ส่งไฟล์
app.get('/file', (req, res) => {
  const options = {
    root: path.join(__dirname, 'public'),
    dotfiles: 'deny',
    headers: {
      'x-timestamp': Date.now(),
      'x-sent': true
    }
  };
  
  res.sendFile('image.jpg', options, (err) => {
    if (err) next(err);
  });
});

// res.set() / res.setHeader() - กำหนด headers
app.get('/headers', (req, res) => {
  res.set({
    'Content-Type': 'application/json',
    'X-Custom-Header': 'value',
    'Cache-Control': 'no-cache'
  });
  res.json({ data: 'with custom headers' });
});

// res.cookie() - กำหนด cookie
app.get('/set-cookie', (req, res) => {
  res.cookie('session', 'abc123', {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    sameSite: 'strict',
    maxAge: 24 * 60 * 60 * 1000 // 1 day
  });
  res.json({ message: 'Cookie set' });
});

// res.clearCookie() - ลบ cookie
app.get('/clear-cookie', (req, res) => {
  res.clearCookie('session');
  res.json({ message: 'Cookie cleared' });
});

// res.locals - data สำหรับ template
app.use((req, res, next) => {
  res.locals.user = req.user; // เข้าถึงได้ในทุก templates
  res.locals.siteName = 'My App';
  next();
});

// res.type() - set Content-Type
app.get('/xml', (req, res) => {
  res.type('xml');
  res.send('<response><message>Hello</message></response>');
});

// Chaining
app.get('/chained', (req, res) => {
  res
    .status(200)
    .set('X-Custom', 'value')
    .json({ success: true });
});

app.listen(3000);
```

---

## Step 1144: CORS

```javascript
const express = require('express');
const cors = require('cors'); // npm install cors
const app = express();

app.use(express.json());

// ===== cors package =====

// Simple: อนุญาตทุก origins
app.use(cors());

// Options
app.use(cors({
  origin: 'http://localhost:3000',     // อนุญาต origin เดียว
  methods: ['GET', 'POST', 'PUT', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
  credentials: true,                    // อนุญาต cookies
  maxAge: 86400                         // preflight cache 1 วัน (วินาที)
}));

// Multiple origins
const allowedOrigins = [
  'http://localhost:3000',
  'https://myapp.com',
  'https://www.myapp.com'
];

app.use(cors({
  origin: (origin, callback) => {
    // อนุญาต request ที่ไม่มี origin (curl, Postman, same-origin)
    if (!origin) return callback(null, true);
    
    if (allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      callback(new Error(`CORS: Origin ${origin} not allowed`));
    }
  },
  credentials: true
}));

// CORS เฉพาะบาง routes
app.get('/public', cors(), (req, res) => {
  res.json({ message: 'Public endpoint, any origin allowed' });
});

app.get('/private', (req, res) => {
  res.json({ message: 'No CORS headers' });
});

// Preflight OPTIONS
app.options('/api/*', cors()); // ต้องมีสำหรับ preflight

// ===== CORS ทำงานอย่างไร =====
/*
1. Browser ส่ง preflight OPTIONS request:
   OPTIONS /api/users HTTP/1.1
   Origin: http://localhost:3000
   Access-Control-Request-Method: POST
   Access-Control-Request-Headers: Content-Type

2. Server ตอบ:
   Access-Control-Allow-Origin: http://localhost:3000
   Access-Control-Allow-Methods: POST
   Access-Control-Allow-Headers: Content-Type
   
3. ถ้า OK, browser ส่ง actual request
*/

app.listen(3000);
```

---

## Step 1145: Helmet (Security Headers)

```javascript
const express = require('express');
const helmet = require('helmet'); // npm install helmet
const app = express();

// ===== Helmet =====
// เพิ่ม security HTTP headers

// ใช้ default settings (แนะนำ)
app.use(helmet());

// หรือ configure แต่ละส่วน
app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'", "'unsafe-inline'", 'https://cdn.jsdelivr.net'],
      styleSrc: ["'self'", "'unsafe-inline'", 'https://fonts.googleapis.com'],
      imgSrc: ["'self'", 'data:', 'https:'],
      connectSrc: ["'self'", 'https://api.example.com'],
      fontSrc: ["'self'", 'https://fonts.gstatic.com'],
      objectSrc: ["'none'"],
      mediaSrc: ["'self'"],
      frameSrc: ["'none'"],
    },
  },
  hsts: {
    maxAge: 31536000,      // 1 ปี
    includeSubDomains: true,
    preload: true
  },
  referrerPolicy: { policy: 'strict-origin-when-cross-origin' },
  noSniff: true,           // X-Content-Type-Options: nosniff
  frameguard: { action: 'sameorigin' }, // X-Frame-Options
  xssFilter: true          // X-XSS-Protection
}));

// Headers ที่ Helmet เพิ่ม:
// Content-Security-Policy
// X-DNS-Prefetch-Control
// X-Frame-Options
// X-Download-Options
// X-Content-Type-Options
// Permissions-Policy
// Referrer-Policy
// Strict-Transport-Security (HSTS)
// X-XSS-Protection

app.listen(3000);
```

---

## Step 1146: Rate Limiting

```javascript
const express = require('express');
const rateLimit = require('express-rate-limit'); // npm install express-rate-limit
const app = express();

app.use(express.json());

// ===== Basic rate limiter =====
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000,  // 15 minutes
  max: 100,                    // จำกัด 100 requests ต่อ window
  standardHeaders: true,       // Return rate limit info in `RateLimit-*` headers
  legacyHeaders: false,        // Disable `X-RateLimit-*` headers
  
  // Custom error message
  message: {
    error: 'Too many requests, please try again later.',
    retryAfter: '15 minutes'
  },
  
  // Custom handler
  handler: (req, res, next, options) => {
    res.status(options.statusCode).json({
      error: 'Rate limit exceeded',
      limit: options.max,
      windowMs: options.windowMs,
      retryAfter: new Date(Date.now() + options.windowMs).toISOString()
    });
  }
});

// Apply to all requests
app.use(limiter);

// ===== Different limits for different routes =====

// Strict limit for auth
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5, // 5 login attempts per 15 minutes
  skipSuccessfulRequests: true // ไม่นับ successful requests
});

app.use('/api/auth/login', authLimiter);
app.use('/api/auth/register', authLimiter);

// API limiter
const apiLimiter = rateLimit({
  windowMs: 60 * 1000, // 1 minute
  max: 30,
  message: 'API rate limit exceeded'
});

app.use('/api/', apiLimiter);

// ===== Per-user rate limiting =====
const userLimiter = rateLimit({
  windowMs: 60 * 1000,
  max: 60,
  keyGenerator: (req) => {
    // ใช้ user ID ถ้า authenticated
    return req.user?.id || req.ip;
  }
});

// ===== Redis store (สำหรับ distributed systems) =====
// npm install rate-limit-redis @upstash/redis
const RedisStore = require('rate-limit-redis');

const distributedLimiter = rateLimit({
  windowMs: 60 * 1000,
  max: 100,
  store: new RedisStore({
    client: redisClient,
    prefix: 'rate_limit:'
  })
});

app.listen(3000);
```

---

## Step 1147: File Uploads กับ Multer

```javascript
const express = require('express');
const multer = require('multer'); // npm install multer
const path = require('path');
const app = express();

// ===== Multer configuration =====

// Memory storage (file อยู่ใน memory ชั่วคราว)
const memoryStorage = multer.memoryStorage();

// Disk storage
const diskStorage = multer.diskStorage({
  destination: (req, file, cb) => {
    cb(null, 'uploads/');  // directory สำหรับเก็บไฟล์
  },
  filename: (req, file, cb) => {
    // ตั้งชื่อไฟล์: timestamp + original name
    const uniqueName = Date.now() + '-' + Math.round(Math.random() * 1E9);
    const ext = path.extname(file.originalname);
    cb(null, `${uniqueName}${ext}`);
  }
});

// File filter
const fileFilter = (req, file, cb) => {
  const allowedTypes = ['image/jpeg', 'image/png', 'image/gif', 'image/webp'];
  
  if (allowedTypes.includes(file.mimetype)) {
    cb(null, true);   // accept file
  } else {
    cb(new Error('Only image files are allowed'), false); // reject file
  }
};

const upload = multer({
  storage: diskStorage,
  limits: {
    fileSize: 5 * 1024 * 1024,  // 5MB
    files: 5                      // สูงสุด 5 ไฟล์
  },
  fileFilter
});

// ===== Single file upload =====
app.post('/upload/single', upload.single('avatar'), (req, res) => {
  if (!req.file) {
    return res.status(400).json({ error: 'No file uploaded' });
  }
  
  console.log('File:', req.file);
  // req.file = {
  //   fieldname: 'avatar',
  //   originalname: 'photo.jpg',
  //   encoding: '7bit',
  //   mimetype: 'image/jpeg',
  //   destination: 'uploads/',
  //   filename: '1234567890-photo.jpg',
  //   path: 'uploads/1234567890-photo.jpg',
  //   size: 102400
  // }
  
  res.json({
    message: 'File uploaded successfully',
    file: {
      name: req.file.filename,
      size: req.file.size,
      url: `/uploads/${req.file.filename}`
    }
  });
});

// ===== Multiple files =====
app.post('/upload/multiple', upload.array('photos', 5), (req, res) => {
  if (!req.files || req.files.length === 0) {
    return res.status(400).json({ error: 'No files uploaded' });
  }
  
  const files = req.files.map(file => ({
    name: file.filename,
    size: file.size,
    url: `/uploads/${file.filename}`
  }));
  
  res.json({ files });
});

// ===== Different fields =====
app.post('/upload/profile', upload.fields([
  { name: 'avatar', maxCount: 1 },
  { name: 'cover', maxCount: 1 }
]), (req, res) => {
  const avatar = req.files?.avatar?.[0];
  const cover = req.files?.cover?.[0];
  
  res.json({
    avatar: avatar ? `/uploads/${avatar.filename}` : null,
    cover: cover ? `/uploads/${cover.filename}` : null
  });
});

// ===== Error handling =====
app.use((err, req, res, next) => {
  if (err instanceof multer.MulterError) {
    switch (err.code) {
      case 'LIMIT_FILE_SIZE':
        return res.status(400).json({ error: 'File too large (max 5MB)' });
      case 'LIMIT_FILE_COUNT':
        return res.status(400).json({ error: 'Too many files (max 5)' });
      case 'LIMIT_UNEXPECTED_FILE':
        return res.status(400).json({ error: `Unexpected field: ${err.field}` });
      default:
        return res.status(400).json({ error: err.message });
    }
  }
  
  if (err.message === 'Only image files are allowed') {
    return res.status(400).json({ error: err.message });
  }
  
  next(err);
});

// Serve uploaded files
app.use('/uploads', express.static('uploads'));

app.listen(3000);
```

---

## Step 1148: Complete Express Application

```javascript
// ===== ตัวอย่าง Express App สมบูรณ์ =====

// src/app.js
const express = require('express');
const cors = require('cors');
const helmet = require('helmet');
const rateLimit = require('express-rate-limit');
const path = require('path');

const app = express();

// ===== Security Middleware =====
app.use(helmet());

app.use(cors({
  origin: process.env.ALLOWED_ORIGINS?.split(',') || '*',
  credentials: true
}));

// ===== Rate Limiting =====
app.use('/api/', rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 100
}));

// ===== Body Parsing =====
app.use(express.json({ limit: '10mb' }));
app.use(express.urlencoded({ extended: true, limit: '10mb' }));

// ===== Request Logging =====
app.use((req, res, next) => {
  const start = Date.now();
  res.on('finish', () => {
    const duration = Date.now() - start;
    console.log(`${req.method} ${req.originalUrl} ${res.statusCode} ${duration}ms`);
  });
  next();
});

// ===== Static Files =====
app.use(express.static(path.join(__dirname, '../public')));

// ===== API Routes =====
app.use('/api/v1', require('./routes'));

// ===== Health Check =====
app.get('/health', (req, res) => {
  res.json({
    status: 'ok',
    timestamp: new Date().toISOString(),
    uptime: process.uptime(),
    memory: process.memoryUsage()
  });
});

// ===== 404 Handler =====
app.use((req, res, next) => {
  res.status(404).json({
    error: {
      message: `Route ${req.method} ${req.originalUrl} not found`,
      code: 'ROUTE_NOT_FOUND'
    }
  });
});

// ===== Error Handler =====
app.use((err, req, res, next) => {
  const status = err.statusCode || err.status || 500;
  
  if (status >= 500) {
    console.error('Server Error:', err);
  }
  
  res.status(status).json({
    error: {
      message: err.message || 'Internal Server Error',
      code: err.code || 'INTERNAL_ERROR',
      ...(process.env.NODE_ENV === 'development' && { stack: err.stack })
    }
  });
});

module.exports = app;

// src/server.js
require('dotenv').config();
const app = require('./app');

const PORT = parseInt(process.env.PORT, 10) || 3000;

const server = app.listen(PORT, () => {
  console.log(`Server running at http://localhost:${PORT}`);
  console.log(`Environment: ${process.env.NODE_ENV || 'development'}`);
});

// Graceful shutdown
const shutdown = (signal) => {
  console.log(`\n${signal} received. Shutting down gracefully...`);
  server.close(() => {
    console.log('HTTP server closed.');
    process.exit(0);
  });
  
  setTimeout(() => {
    console.error('Force closing server');
    process.exit(1);
  }, 10000);
};

process.on('SIGTERM', () => shutdown('SIGTERM'));
process.on('SIGINT', () => shutdown('SIGINT'));
process.on('unhandledRejection', (reason) => {
  console.error('Unhandled Promise Rejection:', reason);
  shutdown('unhandledRejection');
});
```

---

## Step 1149: Express Best Practices

```javascript
// ===== 1. Separate concerns =====
// controllers/userController.js
const User = require('../models/User');

exports.getAllUsers = async (req, res, next) => {
  try {
    const users = await User.findAll({
      page: parseInt(req.query.page) || 1,
      limit: parseInt(req.query.limit) || 10
    });
    res.json({ success: true, data: users });
  } catch (err) {
    next(err);
  }
};

exports.getUserById = async (req, res, next) => {
  try {
    const user = await User.findById(req.params.id);
    if (!user) {
      return res.status(404).json({ error: 'User not found' });
    }
    res.json({ success: true, data: user });
  } catch (err) {
    next(err);
  }
};

// ===== 2. Input validation =====
const Joi = require('joi'); // npm install joi

const createUserSchema = Joi.object({
  name: Joi.string().min(2).max(100).required(),
  email: Joi.string().email().required(),
  password: Joi.string().min(8).pattern(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)/).required(),
  age: Joi.number().integer().min(0).max(150)
});

const validateBody = (schema) => (req, res, next) => {
  const { error, value } = schema.validate(req.body, { abortEarly: false });
  if (error) {
    return res.status(422).json({
      error: 'Validation failed',
      details: error.details.map(d => ({ field: d.path[0], message: d.message }))
    });
  }
  req.body = value;
  next();
};

// ===== 3. Async wrapper =====
const asyncHandler = fn => (req, res, next) =>
  Promise.resolve(fn(req, res, next)).catch(next);

// ===== 4. Environment config =====
const config = {
  port: parseInt(process.env.PORT, 10) || 3000,
  nodeEnv: process.env.NODE_ENV || 'development',
  jwtSecret: process.env.JWT_SECRET,
  dbUrl: process.env.DATABASE_URL
};

// ===== 5. API versioning =====
// /api/v1/users
// /api/v2/users (backward compatible)
app.use('/api/v1', require('./routes/v1'));
app.use('/api/v2', require('./routes/v2'));
```

---

## Step 1150: Testing Express Apps

```javascript
const request = require('supertest'); // npm install --save-dev supertest jest
const app = require('../src/app');

// ===== Integration Tests =====

describe('Users API', () => {
  let userId;
  
  describe('GET /api/v1/users', () => {
    it('should return list of users', async () => {
      const response = await request(app)
        .get('/api/v1/users')
        .expect(200);
      
      expect(response.body).toHaveProperty('data');
      expect(Array.isArray(response.body.data)).toBe(true);
    });
    
    it('should support pagination', async () => {
      const response = await request(app)
        .get('/api/v1/users?page=1&limit=5')
        .expect(200);
      
      expect(response.body.data).toHaveLength(5);
    });
  });
  
  describe('POST /api/v1/users', () => {
    it('should create a new user', async () => {
      const userData = {
        name: 'Test User',
        email: 'test@example.com',
        password: 'Password123!'
      };
      
      const response = await request(app)
        .post('/api/v1/users')
        .send(userData)
        .expect(201);
      
      expect(response.body.data).toMatchObject({
        name: userData.name,
        email: userData.email
      });
      expect(response.body.data).not.toHaveProperty('password');
      
      userId = response.body.data.id;
    });
    
    it('should return 422 for invalid data', async () => {
      const response = await request(app)
        .post('/api/v1/users')
        .send({ email: 'not-valid-email' })
        .expect(422);
      
      expect(response.body).toHaveProperty('error');
      expect(response.body).toHaveProperty('details');
    });
  });
  
  describe('GET /api/v1/users/:id', () => {
    it('should return a user by id', async () => {
      const response = await request(app)
        .get(`/api/v1/users/${userId}`)
        .expect(200);
      
      expect(response.body.data.id).toBe(userId);
    });
    
    it('should return 404 for non-existent user', async () => {
      await request(app)
        .get('/api/v1/users/999999')
        .expect(404);
    });
  });
  
  describe('Authentication', () => {
    let authToken;
    
    it('should login with valid credentials', async () => {
      const response = await request(app)
        .post('/api/v1/auth/login')
        .send({ email: 'test@example.com', password: 'Password123!' })
        .expect(200);
      
      expect(response.body).toHaveProperty('token');
      authToken = response.body.token;
    });
    
    it('should access protected route with token', async () => {
      await request(app)
        .get('/api/v1/profile')
        .set('Authorization', `Bearer ${authToken}`)
        .expect(200);
    });
    
    it('should reject request without token', async () => {
      await request(app)
        .get('/api/v1/profile')
        .expect(401);
    });
  });
});
```

---

## สรุป Steps 1131-1150

| Step | หัวข้อ |
|------|--------|
| 1131 | Express.js คืออะไร |
| 1132 | Installation และ setup |
| 1133 | Basic routes: GET, POST, PUT, PATCH, DELETE |
| 1134 | Route parameters: req.params |
| 1135 | Query strings: req.query |
| 1136 | Request body: req.body |
| 1137 | Middleware concept |
| 1138 | Custom middleware |
| 1139 | Error handling middleware |
| 1140 | Router (express.Router) |
| 1141 | Static files |
| 1142 | Template engines |
| 1143 | Response methods |
| 1144 | CORS |
| 1145 | Helmet security headers |
| 1146 | Rate limiting |
| 1147 | File uploads (multer) |
| 1148 | Complete Express app |
| 1149 | Best practices |
| 1150 | Testing Express apps |

---

## แบบฝึกหัดท้ายบท

### แบบฝึกหัดที่ 1: Blog API
สร้าง REST API สำหรับ blog ที่มี:
1. Users: register, login, profile
2. Posts: CRUD, filter by author/tag, pagination
3. Comments: add, delete
4. Authentication ด้วย JWT
5. Image upload สำหรับ post thumbnail

### แบบฝึกหัดที่ 2: File Manager API
สร้าง API สำหรับจัดการไฟล์:
1. Upload ไฟล์ (รูปภาพ, PDF, docs)
2. List ไฟล์พร้อม pagination
3. Download ไฟล์
4. Delete ไฟล์
5. Resize รูปภาพ (ใช้ sharp package)

### แบบฝึกหัดที่ 3: Middleware Chain
สร้าง middleware ที่:
1. Log request/response ลงไฟล์
2. ตรวจสอบ API key
3. Validate request body ตาม schema
4. Cache response (in-memory)
5. Compress response ด้วย gzip

### แบบฝึกหัดที่ 4: E-commerce API
สร้าง API สำหรับ e-commerce:
1. Products: CRUD, search, filter, sort
2. Cart: add, remove, update quantity
3. Orders: place, status update
4. Categories: tree structure
5. Authentication และ Authorization

---

*จบ Part 58: Express.js - ในบทต่อไปเราจะเรียนการออกแบบ RESTful API*
