# Part 59: RESTful API Design (Steps 1151-1170)

## บทนำ

REST (Representational State Transfer) คือ architectural style สำหรับการออกแบบ networked applications โดย Roy Fielding ในปี 2000 การออกแบบ RESTful API ที่ดีทำให้ API ใช้งานง่าย สม่ำเสมอ และบำรุงรักษาได้

---

## Step 1151: REST Principles

```javascript
// ===== 6 REST Constraints =====

/*
1. CLIENT-SERVER
   - แยก client และ server ออกจากกัน
   - Client ไม่รู้เรื่อง data storage
   - Server ไม่รู้เรื่อง UI
*/

/*
2. STATELESS
   - แต่ละ request ต้องมีข้อมูลครบถ้วนในตัวเอง
   - Server ไม่เก็บ session state ของ client
   - ข้อมูล authentication ต้องส่งมาทุก request (ใช้ token)
*/

// GOOD: Stateless
// Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

// BAD: Stateful
// ไม่ควรพึ่ง session ID ที่ server เก็บไว้

/*
3. CACHEABLE
   - Response ต้องบอก client ว่า cache ได้หรือไม่
   - ใช้ Cache-Control, ETag, Last-Modified headers
*/

// Express example:
app.get('/api/products', (req, res) => {
  res.set('Cache-Control', 'public, max-age=300'); // cache 5 นาที
  res.json({ products: [] });
});

/*
4. UNIFORM INTERFACE
   - Identification of resources (URL)
   - Manipulation through representations
   - Self-descriptive messages
   - HATEOAS
*/

/*
5. LAYERED SYSTEM
   - Client ไม่รู้ว่า connect กับ server โดยตรงหรือผ่าน proxy/load balancer
*/

/*
6. CODE ON DEMAND (Optional)
   - Server สามารถส่ง executable code ให้ client ได้
   - เช่น JavaScript
*/
```

---

## Step 1152: HTTP Methods และ Semantics

```javascript
const express = require('express');
const app = express();
app.use(express.json());

/*
HTTP Method Semantics:

GET    - ดึงข้อมูล (safe, idempotent)
POST   - สร้างข้อมูลใหม่ (not safe, not idempotent)
PUT    - แทนที่ข้อมูลทั้งหมด (not safe, idempotent)
PATCH  - แก้ไขบางส่วน (not safe, not necessarily idempotent)
DELETE - ลบข้อมูล (not safe, idempotent)
HEAD   - เหมือน GET แต่ไม่มี body (safe, idempotent)
OPTIONS - ดู methods ที่รองรับ (safe, idempotent)

Idempotent = ทำซ้ำกี่ครั้งก็ได้ผลเหมือนกัน
Safe = ไม่เปลี่ยนแปลง server state
*/

// ===== GET =====
// ดึงข้อมูล - ต้องไม่มี side effects
app.get('/api/users', (req, res) => {
  res.json({ users: [] });
});

// ===== POST =====
// สร้างข้อมูลใหม่ - response 201 Created + Location header
app.post('/api/users', (req, res) => {
  const user = createUser(req.body);
  res.status(201)
     .location(`/api/users/${user.id}`)
     .json(user);
});

// ===== PUT =====
// แทนที่ทั้งหมด - ถ้าไม่มีบางฟิลด์ก็ลบออก
app.put('/api/users/:id', (req, res) => {
  const user = replaceUser(req.params.id, req.body);
  res.json(user);
});

// ===== PATCH =====
// แก้ไขบางส่วน - ส่งเฉพาะฟิลด์ที่ต้องการแก้
app.patch('/api/users/:id', (req, res) => {
  const user = updateUser(req.params.id, req.body);
  res.json(user);
});

// ===== DELETE =====
// ลบข้อมูล - response 204 No Content (ไม่มี body)
app.delete('/api/users/:id', (req, res) => {
  deleteUser(req.params.id);
  res.status(204).send();
});

// ===== HEAD =====
// ตรวจสอบว่า resource มีอยู่ + ดู headers โดยไม่ดาวน์โหลด content
app.head('/api/files/:filename', (req, res) => {
  const stats = getFileStats(req.params.filename);
  if (!stats) return res.status(404).send();
  
  res.set({
    'Content-Type': stats.mimeType,
    'Content-Length': stats.size,
    'Last-Modified': stats.mtime.toUTCString()
  });
  res.send();
});

// ===== OPTIONS =====
// แสดง methods ที่รองรับ
app.options('/api/users', (req, res) => {
  res.set('Allow', 'GET, POST, HEAD, OPTIONS');
  res.send();
});
```

---

## Step 1153: HTTP Status Codes

```javascript
const express = require('express');
const app = express();
app.use(express.json());

// ===== 2xx Success =====

// 200 OK - request สำเร็จ
app.get('/api/users/:id', (req, res) => {
  const user = findUser(req.params.id);
  res.status(200).json(user);
  // หรือ res.json(user) (200 เป็น default)
});

// 201 Created - resource ถูกสร้าง
app.post('/api/users', (req, res) => {
  const user = createUser(req.body);
  res.status(201)
     .header('Location', `/api/users/${user.id}`)
     .json(user);
});

// 202 Accepted - request รับไว้แต่ยังไม่ประมวลผล (async)
app.post('/api/reports/generate', (req, res) => {
  const jobId = startAsyncJob(req.body);
  res.status(202).json({
    message: 'Report generation started',
    jobId,
    statusUrl: `/api/jobs/${jobId}`
  });
});

// 204 No Content - สำเร็จแต่ไม่มีข้อมูล return
app.delete('/api/users/:id', (req, res) => {
  deleteUser(req.params.id);
  res.status(204).send();
});

// ===== 3xx Redirection =====

// 301 Moved Permanently - URL เปลี่ยนถาวร
app.get('/api/v1/users', (req, res) => {
  res.redirect(301, '/api/v2/users');
});

// 302 Found - Redirect ชั่วคราว
app.get('/api/login', (req, res) => {
  res.redirect(302, '/auth/login');
});

// 304 Not Modified - resource ไม่เปลี่ยน
app.get('/api/config', (req, res) => {
  const etag = generateETag(config);
  
  if (req.headers['if-none-match'] === etag) {
    return res.status(304).send();
  }
  
  res.setHeader('ETag', etag);
  res.json(config);
});

// ===== 4xx Client Errors =====

// 400 Bad Request - request ไม่ถูกต้อง
app.post('/api/users', (req, res) => {
  const { name, email } = req.body;
  
  if (!name || !email) {
    return res.status(400).json({
      error: {
        code: 'MISSING_REQUIRED_FIELDS',
        message: 'name and email are required',
        fields: ['name', 'email'].filter(f => !req.body[f])
      }
    });
  }
  
  res.status(201).json({ name, email });
});

// 401 Unauthorized - ไม่ได้ authenticate
app.get('/api/profile', (req, res) => {
  if (!req.headers.authorization) {
    return res.status(401)
      .header('WWW-Authenticate', 'Bearer realm="API"')
      .json({ error: 'Authentication required' });
  }
  
  res.json({ profile: {} });
});

// 403 Forbidden - authenticate แล้วแต่ไม่มีสิทธิ์
app.delete('/api/users/:id', (req, res) => {
  if (req.user.role !== 'admin') {
    return res.status(403).json({
      error: 'Insufficient permissions to delete users'
    });
  }
  
  deleteUser(req.params.id);
  res.status(204).send();
});

// 404 Not Found - ไม่พบ resource
app.get('/api/users/:id', (req, res) => {
  const user = findUser(req.params.id);
  
  if (!user) {
    return res.status(404).json({
      error: {
        code: 'USER_NOT_FOUND',
        message: `User with id ${req.params.id} not found`
      }
    });
  }
  
  res.json(user);
});

// 405 Method Not Allowed
app.all('/api/users/:id', (req, res, next) => {
  const allowed = ['GET', 'PUT', 'PATCH', 'DELETE'];
  
  if (!allowed.includes(req.method)) {
    return res.status(405)
      .header('Allow', allowed.join(', '))
      .json({ error: `Method ${req.method} not allowed` });
  }
  
  next();
});

// 409 Conflict - resource มีอยู่แล้ว หรือ conflict
app.post('/api/users', async (req, res) => {
  const existingUser = await findUserByEmail(req.body.email);
  
  if (existingUser) {
    return res.status(409).json({
      error: {
        code: 'EMAIL_ALREADY_EXISTS',
        message: 'A user with this email already exists'
      }
    });
  }
  
  res.status(201).json(createUser(req.body));
});

// 422 Unprocessable Entity - validation failed
app.post('/api/users', (req, res) => {
  const errors = validateUser(req.body);
  
  if (errors.length > 0) {
    return res.status(422).json({
      error: {
        code: 'VALIDATION_ERROR',
        message: 'Request data failed validation',
        details: errors
      }
    });
  }
  
  res.status(201).json(createUser(req.body));
});

// 429 Too Many Requests
// (จัดการโดย rate limiter middleware)

// ===== 5xx Server Errors =====

// 500 Internal Server Error
app.get('/api/users', (req, res, next) => {
  try {
    const users = getUsers();
    res.json(users);
  } catch (err) {
    next(err); // Error handler จะ return 500
  }
});

// 502 Bad Gateway
// 503 Service Unavailable (maintenance mode)
// 504 Gateway Timeout
```

---

## Step 1154: RESTful URL Design

```javascript
// ===== Resource Naming Rules =====

// Rule 1: ใช้ nouns (ไม่ใช่ verbs) สำหรับ resources
// GOOD:
// GET /api/users
// POST /api/users
// GET /api/users/123

// BAD:
// GET /api/getUsers
// POST /api/createUser
// GET /api/getUserById/123

// Rule 2: ใช้ plural nouns
// GOOD: /api/users, /api/products, /api/orders
// BAD: /api/user, /api/product, /api/order

// Rule 3: ใช้ lowercase และ hyphens (ไม่ใช่ camelCase)
// GOOD: /api/user-profiles, /api/blog-posts
// BAD: /api/userProfiles, /api/blogPosts, /api/UserProfiles

// Rule 4: Nested resources สำหรับ relationships
// GOOD:
app.get('/api/users/:userId/orders', ...);          // orders ของ user
app.get('/api/users/:userId/orders/:orderId', ...); // specific order ของ user
app.get('/api/posts/:postId/comments', ...);        // comments ของ post
app.post('/api/posts/:postId/comments', ...);       // เพิ่ม comment

// อย่า nest ลึกเกิน 2-3 ระดับ
// AVOID: /api/users/1/posts/2/comments/3/replies/4/likes/5

// Rule 5: ใช้ query parameters สำหรับ filtering, sorting, pagination
app.get('/api/users?role=admin&status=active', ...);    // filter
app.get('/api/users?sort=name&order=asc', ...);          // sort
app.get('/api/users?page=2&limit=10', ...);              // paginate
app.get('/api/products?category=electronics&minPrice=100&maxPrice=500', ...);

// Rule 6: Actions ที่ไม่ใช่ CRUD (ใช้ verbs ได้บ้าง)
// สำหรับ actions เฉพาะที่ไม่ map ไปยัง CRUD
app.post('/api/users/:id/activate', ...);     // activate user
app.post('/api/users/:id/deactivate', ...);   // deactivate user
app.post('/api/orders/:id/cancel', ...);      // cancel order
app.post('/api/posts/:id/publish', ...);      // publish post
app.post('/api/emails/:id/send', ...);        // send email
app.post('/api/passwords/reset', ...);        // reset password

// Rule 7: API versioning ใน URL
app.use('/api/v1', v1Router);
app.use('/api/v2', v2Router);

// ===== ตัวอย่าง URL structure =====
/*
Blog API:
GET    /api/v1/posts                    - list all posts
POST   /api/v1/posts                    - create post
GET    /api/v1/posts/:id                - get post
PUT    /api/v1/posts/:id                - replace post
PATCH  /api/v1/posts/:id                - update post
DELETE /api/v1/posts/:id                - delete post
GET    /api/v1/posts/:id/comments       - list comments
POST   /api/v1/posts/:id/comments       - add comment
DELETE /api/v1/posts/:postId/comments/:commentId - delete comment
POST   /api/v1/posts/:id/publish        - publish post
POST   /api/v1/posts/:id/unpublish      - unpublish post
GET    /api/v1/posts?tag=nodejs&author=alice&page=1&limit=10
*/
```

---

## Step 1155: API Versioning

```javascript
const express = require('express');
const app = express();
app.use(express.json());

// ===== Strategy 1: URL Path (แนะนำ) =====
app.use('/api/v1', require('./routes/v1'));
app.use('/api/v2', require('./routes/v2'));

// ===== Strategy 2: Request Header =====
app.use((req, res, next) => {
  const version = req.headers['api-version'] || '1';
  req.apiVersion = parseInt(version);
  next();
});

app.get('/api/users', (req, res) => {
  if (req.apiVersion >= 2) {
    return res.json({ data: users, meta: { total: users.length } }); // v2 format
  }
  res.json(users); // v1 format
});

// ===== Strategy 3: Accept Header =====
app.get('/api/users', (req, res) => {
  const accepts = req.headers.accept || '';
  
  if (accepts.includes('application/vnd.myapi.v2+json')) {
    return res.json({ data: users, meta: {} }); // v2
  }
  
  res.json(users); // v1 (default)
});

// ===== Strategy 4: Query Parameter =====
app.get('/api/users', (req, res) => {
  const version = req.query.v || '1';
  
  if (version === '2') {
    return res.json({ data: users, version: '2' });
  }
  
  res.json(users);
});

// ===== Versioning best practices =====
// routes/v1/index.js
const express = require('express');
const router = express.Router();

router.use('/users', require('./users'));
router.use('/posts', require('./posts'));

module.exports = router;

// routes/v2/index.js  
const express = require('express');
const router = express.Router();

// v2 มี additional features, different response format
router.use('/users', require('./users')); // updated for v2
router.use('/posts', require('./posts'));
router.use('/analytics', require('./analytics')); // new in v2

module.exports = router;

// Deprecation notice
app.use('/api/v1', (req, res, next) => {
  res.set('Deprecation', 'true');
  res.set('Sunset', 'Sat, 31 Dec 2025 23:59:59 GMT');
  res.set('Link', '</api/v2/users>; rel="successor-version"');
  next();
}, require('./routes/v1'));
```

---

## Step 1156: Input Validation กับ express-validator

```javascript
const express = require('express');
const { body, param, query, validationResult } = require('express-validator');
// npm install express-validator

const app = express();
app.use(express.json());

// ===== Validation middleware helper =====
const validate = (req, res, next) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(422).json({
      error: {
        code: 'VALIDATION_ERROR',
        message: 'Request validation failed',
        details: errors.array().map(err => ({
          field: err.path,
          message: err.msg,
          value: err.value
        }))
      }
    });
  }
  next();
};

// ===== User validation rules =====
const userValidation = {
  create: [
    body('name')
      .trim()
      .notEmpty().withMessage('Name is required')
      .isLength({ min: 2, max: 100 }).withMessage('Name must be 2-100 characters')
      .matches(/^[a-zA-Z\s฀-๿]+$/).withMessage('Name contains invalid characters'),
    
    body('email')
      .trim()
      .notEmpty().withMessage('Email is required')
      .isEmail().withMessage('Must be a valid email')
      .normalizeEmail(),
    
    body('password')
      .notEmpty().withMessage('Password is required')
      .isLength({ min: 8 }).withMessage('Password must be at least 8 characters')
      .matches(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])/)
      .withMessage('Password must contain uppercase, lowercase, number, and special character'),
    
    body('age')
      .optional()
      .isInt({ min: 0, max: 150 }).withMessage('Age must be between 0 and 150'),
    
    body('role')
      .optional()
      .isIn(['user', 'admin', 'moderator']).withMessage('Invalid role'),
    
    body('website')
      .optional()
      .isURL({ protocols: ['http', 'https'] }).withMessage('Must be a valid URL'),
    
    validate
  ],
  
  update: [
    param('id').isInt({ min: 1 }).withMessage('Invalid user ID'),
    
    body('name')
      .optional()
      .trim()
      .isLength({ min: 2, max: 100 }).withMessage('Name must be 2-100 characters'),
    
    body('email')
      .optional()
      .isEmail().withMessage('Must be a valid email')
      .normalizeEmail(),
    
    validate
  ]
};

// ===== Routes with validation =====
app.post('/api/users', userValidation.create, (req, res) => {
  // req.body ผ่าน validation แล้ว
  const { name, email, password } = req.body;
  res.status(201).json({ id: 1, name, email });
});

app.patch('/api/users/:id', userValidation.update, (req, res) => {
  res.json({ id: req.params.id, ...req.body });
});

// ===== Complex validations =====
app.post('/api/products', [
  body('name').trim().notEmpty(),
  body('price').isFloat({ min: 0 }).withMessage('Price must be non-negative'),
  body('stock').isInt({ min: 0 }),
  body('category').notEmpty(),
  
  body('images').optional().isArray(),
  body('images.*').optional().isURL(),
  
  body('variants').optional().isArray(),
  body('variants.*.name').if(body('variants').exists()).notEmpty(),
  body('variants.*.price').if(body('variants').exists()).isFloat({ min: 0 }),
  
  body('discount.type')
    .optional()
    .isIn(['percentage', 'fixed']),
  body('discount.value')
    .if(body('discount').exists())
    .isFloat({ min: 0 }),
  
  validate
], (req, res) => {
  res.status(201).json(req.body);
});

// ===== Query validation =====
app.get('/api/products', [
  query('page').optional().isInt({ min: 1 }).toInt(),
  query('limit').optional().isInt({ min: 1, max: 100 }).toInt(),
  query('sort').optional().isIn(['name', 'price', 'created_at']),
  query('order').optional().isIn(['asc', 'desc']),
  query('minPrice').optional().isFloat({ min: 0 }).toFloat(),
  query('maxPrice').optional().isFloat({ min: 0 }).toFloat(),
  
  // Custom validation: minPrice <= maxPrice
  query('maxPrice').custom((maxPrice, { req }) => {
    const minPrice = req.query.minPrice;
    if (minPrice && maxPrice && parseFloat(maxPrice) < parseFloat(minPrice)) {
      throw new Error('maxPrice must be greater than minPrice');
    }
    return true;
  }),
  
  validate
], (req, res) => {
  const { page = 1, limit = 10, sort = 'created_at', order = 'desc' } = req.query;
  res.json({ products: [], page, limit });
});
```

---

## Step 1157: Input Validation กับ Zod

```javascript
const express = require('express');
const { z } = require('zod'); // npm install zod
const app = express();
app.use(express.json());

// ===== Zod schemas =====
const createUserSchema = z.object({
  name: z.string()
    .min(2, 'Name must be at least 2 characters')
    .max(100, 'Name must not exceed 100 characters')
    .regex(/^[a-zA-Z\s฀-๿]+$/, 'Name contains invalid characters'),
  
  email: z.string()
    .email('Invalid email format')
    .toLowerCase(),
  
  password: z.string()
    .min(8, 'Password must be at least 8 characters')
    .regex(
      /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])/,
      'Password must include uppercase, lowercase, number, and special character'
    ),
  
  age: z.number().int().min(0).max(150).optional(),
  
  role: z.enum(['user', 'admin', 'moderator']).default('user'),
  
  address: z.object({
    street: z.string(),
    city: z.string(),
    country: z.string(),
    zipCode: z.string().regex(/^\d{5}$/, 'Invalid zip code')
  }).optional()
});

const updateUserSchema = createUserSchema.partial(); // ทุกฟิลด์เป็น optional

// ===== Validation middleware =====
function validateBody(schema) {
  return (req, res, next) => {
    const result = schema.safeParse(req.body);
    
    if (!result.success) {
      const errors = result.error.errors.map(err => ({
        field: err.path.join('.'),
        message: err.message,
        code: err.code
      }));
      
      return res.status(422).json({
        error: {
          code: 'VALIDATION_ERROR',
          message: 'Validation failed',
          details: errors
        }
      });
    }
    
    req.body = result.data; // use parsed/sanitized data
    next();
  };
}

// ===== Routes =====
app.post('/api/users', validateBody(createUserSchema), (req, res) => {
  // req.body ผ่าน validation + transformation แล้ว
  res.status(201).json({ id: 1, ...req.body });
});

app.patch('/api/users/:id', validateBody(updateUserSchema), (req, res) => {
  res.json({ id: parseInt(req.params.id), ...req.body });
});

// ===== Complex schema =====
const orderSchema = z.object({
  items: z.array(z.object({
    productId: z.number().int().positive(),
    quantity: z.number().int().positive().max(100),
    price: z.number().positive()
  })).min(1, 'At least one item required'),
  
  shipping: z.object({
    address: z.string().min(10),
    city: z.string(),
    country: z.string().length(2),
    method: z.enum(['standard', 'express', 'overnight'])
  }),
  
  payment: z.object({
    method: z.enum(['credit_card', 'promptpay', 'bank_transfer']),
    amount: z.number().positive()
  }),
  
  couponCode: z.string().optional(),
  
  // computed fields
  subtotal: z.number().optional(),
  total: z.number().optional()
}).refine(data => {
  // custom validation: payment amount = sum of items
  const subtotal = data.items.reduce((sum, item) => sum + item.price * item.quantity, 0);
  return data.payment.amount >= subtotal; // allow for tips/extra
}, {
  message: 'Payment amount is insufficient',
  path: ['payment', 'amount']
});

app.post('/api/orders', validateBody(orderSchema), (req, res) => {
  const { items, shipping, payment } = req.body;
  const subtotal = items.reduce((sum, item) => sum + item.price * item.quantity, 0);
  
  const order = {
    id: Date.now(),
    ...req.body,
    subtotal,
    tax: subtotal * 0.07,
    total: subtotal * 1.07,
    status: 'pending',
    createdAt: new Date()
  };
  
  res.status(201).json(order);
});
```

---

## Step 1158: JWT Authentication

```javascript
const express = require('express');
const jwt = require('jsonwebtoken'); // npm install jsonwebtoken
const bcrypt = require('bcrypt'); // npm install bcrypt

const app = express();
app.use(express.json());

// ===== JWT Structure =====
/*
JWT = Header.Payload.Signature

Header: { "alg": "HS256", "typ": "JWT" }
Payload: { "sub": "user_id", "email": "user@example.com", "role": "user", "iat": 1516239022, "exp": 1516325422 }
Signature: HMACSHA256(base64(header) + "." + base64(payload), secret)

ข้อมูลใน Payload ถูก decode ได้โดยทุกคน แต่ Signature ยืนยันว่ามาจาก server
ห้ามเก็บ sensitive data (password, credit card) ใน JWT!
*/

const JWT_SECRET = process.env.JWT_SECRET || 'your-super-secret-key';
const JWT_EXPIRES_IN = '24h';
const JWT_REFRESH_EXPIRES_IN = '7d';

// Mock database
const users = new Map([
  [1, {
    id: 1,
    email: 'alice@example.com',
    // hash ของ 'password123'
    passwordHash: '$2b$10$N9qo8uLOickgx2ZMRZoMyeIjZAgcfl7p92ldGxad68LJZdL17lhWy',
    role: 'user',
    name: 'Alice'
  }]
]);

// ===== Create token =====
function createAccessToken(user) {
  return jwt.sign(
    {
      sub: user.id,
      email: user.email,
      role: user.role,
      name: user.name
    },
    JWT_SECRET,
    {
      expiresIn: JWT_EXPIRES_IN,
      issuer: 'myapp.com',
      audience: 'myapp-users'
    }
  );
}

function createRefreshToken(userId) {
  return jwt.sign(
    { sub: userId, type: 'refresh' },
    JWT_SECRET + '-refresh',
    { expiresIn: JWT_REFRESH_EXPIRES_IN }
  );
}

// ===== Auth Routes =====
app.post('/api/auth/register', async (req, res) => {
  const { name, email, password } = req.body;
  
  // ตรวจสอบ email ซ้ำ
  const existingUser = [...users.values()].find(u => u.email === email);
  if (existingUser) {
    return res.status(409).json({ error: 'Email already registered' });
  }
  
  // Hash password
  const saltRounds = 12;
  const passwordHash = await bcrypt.hash(password, saltRounds);
  
  // สร้าง user
  const id = users.size + 1;
  const newUser = { id, name, email, passwordHash, role: 'user' };
  users.set(id, newUser);
  
  // สร้าง tokens
  const accessToken = createAccessToken(newUser);
  const refreshToken = createRefreshToken(id);
  
  res.status(201).json({
    user: { id, name, email, role: newUser.role },
    accessToken,
    refreshToken,
    expiresIn: JWT_EXPIRES_IN
  });
});

app.post('/api/auth/login', async (req, res) => {
  const { email, password } = req.body;
  
  if (!email || !password) {
    return res.status(400).json({ error: 'Email and password are required' });
  }
  
  // หา user
  const user = [...users.values()].find(u => u.email === email);
  
  if (!user) {
    // ใช้เวลาเท่ากันเพื่อป้องกัน timing attacks
    await bcrypt.compare(password, '$2b$10$invalid-hash-to-waste-time');
    return res.status(401).json({ error: 'Invalid credentials' });
  }
  
  // ตรวจสอบ password
  const isValid = await bcrypt.compare(password, user.passwordHash);
  
  if (!isValid) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }
  
  const accessToken = createAccessToken(user);
  const refreshToken = createRefreshToken(user.id);
  
  res.json({
    user: { id: user.id, name: user.name, email: user.email, role: user.role },
    accessToken,
    refreshToken,
    expiresIn: JWT_EXPIRES_IN
  });
});

app.post('/api/auth/refresh', (req, res) => {
  const { refreshToken } = req.body;
  
  if (!refreshToken) {
    return res.status(401).json({ error: 'Refresh token required' });
  }
  
  try {
    const decoded = jwt.verify(refreshToken, JWT_SECRET + '-refresh');
    
    if (decoded.type !== 'refresh') {
      throw new Error('Invalid token type');
    }
    
    const user = users.get(decoded.sub);
    if (!user) {
      throw new Error('User not found');
    }
    
    const accessToken = createAccessToken(user);
    
    res.json({ accessToken, expiresIn: JWT_EXPIRES_IN });
  } catch (err) {
    res.status(401).json({ error: 'Invalid refresh token' });
  }
});

// ===== Auth Middleware =====
function authenticate(req, res, next) {
  const authHeader = req.headers.authorization;
  
  if (!authHeader?.startsWith('Bearer ')) {
    return res.status(401).json({ error: 'Bearer token required' });
  }
  
  const token = authHeader.slice(7);
  
  try {
    const decoded = jwt.verify(token, JWT_SECRET, {
      issuer: 'myapp.com',
      audience: 'myapp-users'
    });
    
    req.user = decoded;
    next();
  } catch (err) {
    if (err.name === 'TokenExpiredError') {
      return res.status(401).json({
        error: 'Token expired',
        code: 'TOKEN_EXPIRED'
      });
    }
    
    if (err.name === 'JsonWebTokenError') {
      return res.status(401).json({ error: 'Invalid token' });
    }
    
    next(err);
  }
}

function authorize(...roles) {
  return (req, res, next) => {
    if (!roles.includes(req.user.role)) {
      return res.status(403).json({
        error: `Required role: ${roles.join(' or ')}, current role: ${req.user.role}`
      });
    }
    next();
  };
}

// ===== Protected Routes =====
app.get('/api/profile', authenticate, (req, res) => {
  const user = users.get(req.user.sub);
  const { passwordHash, ...safeUser } = user;
  res.json(safeUser);
});

app.get('/api/admin/users', authenticate, authorize('admin'), (req, res) => {
  const allUsers = [...users.values()].map(({ passwordHash, ...u }) => u);
  res.json({ users: allUsers });
});

app.listen(3000);
```

---

## Step 1159: API Documentation กับ Swagger/OpenAPI

```javascript
const express = require('express');
const swaggerUi = require('swagger-ui-express'); // npm install swagger-ui-express
const YAML = require('yamljs'); // npm install yamljs
// หรือ
const swaggerJsdoc = require('swagger-jsdoc'); // npm install swagger-jsdoc

const app = express();
app.use(express.json());

// ===== OpenAPI Specification (swagger.yaml) =====
/*
openapi: 3.0.0
info:
  title: My API
  version: 1.0.0
  description: My awesome API documentation
  contact:
    email: api@example.com
  license:
    name: MIT

servers:
  - url: http://localhost:3000/api/v1
    description: Development server
  - url: https://api.example.com/v1
    description: Production server

paths:
  /users:
    get:
      summary: Get all users
      tags: [Users]
      parameters:
        - in: query
          name: page
          schema:
            type: integer
            default: 1
        - in: query
          name: limit
          schema:
            type: integer
            default: 10
      responses:
        '200':
          description: List of users
          content:
            application/json:
              schema:
                $ref: '#/components/schemas/UserList'

components:
  schemas:
    User:
      type: object
      properties:
        id:
          type: integer
        name:
          type: string
        email:
          type: string
          format: email
    UserList:
      type: object
      properties:
        data:
          type: array
          items:
            $ref: '#/components/schemas/User'
        pagination:
          $ref: '#/components/schemas/Pagination'
*/

// ===== swagger-jsdoc (JSDoc approach) =====
const swaggerOptions = {
  definition: {
    openapi: '3.0.0',
    info: {
      title: 'My API',
      version: '1.0.0',
      description: 'API Documentation'
    },
    servers: [
      { url: 'http://localhost:3000/api/v1', description: 'Development' }
    ],
    components: {
      securitySchemes: {
        bearerAuth: {
          type: 'http',
          scheme: 'bearer',
          bearerFormat: 'JWT'
        }
      },
      schemas: {
        User: {
          type: 'object',
          required: ['name', 'email'],
          properties: {
            id: { type: 'integer', readOnly: true },
            name: { type: 'string', minLength: 2, maxLength: 100 },
            email: { type: 'string', format: 'email' },
            role: { type: 'string', enum: ['user', 'admin'] },
            createdAt: { type: 'string', format: 'date-time', readOnly: true }
          }
        },
        Error: {
          type: 'object',
          properties: {
            error: {
              type: 'object',
              properties: {
                code: { type: 'string' },
                message: { type: 'string' }
              }
            }
          }
        }
      }
    }
  },
  apis: ['./src/routes/*.js'] // path ไปยังไฟล์ที่มี JSDoc comments
};

const specs = swaggerJsdoc(swaggerOptions);

// Serve Swagger UI
app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(specs, {
  explorer: true,
  customCss: '.swagger-ui .topbar { display: none }',
  customSiteTitle: 'My API Docs'
}));

// Serve JSON spec
app.get('/api-docs.json', (req, res) => {
  res.setHeader('Content-Type', 'application/json');
  res.send(specs);
});

// ===== JSDoc annotations ใน routes =====

/**
 * @swagger
 * /users:
 *   get:
 *     summary: Get all users
 *     tags: [Users]
 *     security:
 *       - bearerAuth: []
 *     parameters:
 *       - in: query
 *         name: page
 *         schema:
 *           type: integer
 *           default: 1
 *         description: Page number
 *       - in: query
 *         name: limit
 *         schema:
 *           type: integer
 *           default: 10
 *           maximum: 100
 *         description: Items per page
 *     responses:
 *       200:
 *         description: Success
 *         content:
 *           application/json:
 *             schema:
 *               type: object
 *               properties:
 *                 data:
 *                   type: array
 *                   items:
 *                     $ref: '#/components/schemas/User'
 *       401:
 *         description: Unauthorized
 *         content:
 *           application/json:
 *             schema:
 *               $ref: '#/components/schemas/Error'
 */
app.get('/api/v1/users', (req, res) => {
  res.json({ data: [], pagination: {} });
});

/**
 * @swagger
 * /users:
 *   post:
 *     summary: Create a new user
 *     tags: [Users]
 *     requestBody:
 *       required: true
 *       content:
 *         application/json:
 *           schema:
 *             $ref: '#/components/schemas/User'
 *           example:
 *             name: Alice
 *             email: alice@example.com
 *             password: Password123!
 *     responses:
 *       201:
 *         description: User created
 *         content:
 *           application/json:
 *             schema:
 *               $ref: '#/components/schemas/User'
 *       409:
 *         description: Email already exists
 */
app.post('/api/v1/users', (req, res) => {
  res.status(201).json({ id: 1, ...req.body });
});

app.listen(3000, () => {
  console.log('API docs available at http://localhost:3000/api-docs');
});
```

---

## Step 1160: Pagination Patterns

```javascript
const express = require('express');
const app = express();
app.use(express.json());

// ===== Pattern 1: Offset Pagination =====
// เหมาะกับ: UI ที่มีหน้าแบบ traditional

app.get('/api/users', async (req, res) => {
  const page = Math.max(1, parseInt(req.query.page) || 1);
  const limit = Math.min(100, Math.max(1, parseInt(req.query.limit) || 10));
  const offset = (page - 1) * limit;
  
  const total = await User.count();
  const users = await User.findAll({ offset, limit });
  
  const totalPages = Math.ceil(total / limit);
  
  res.json({
    data: users,
    pagination: {
      page,
      limit,
      total,
      totalPages,
      hasNext: page < totalPages,
      hasPrev: page > 1,
      nextPage: page < totalPages ? page + 1 : null,
      prevPage: page > 1 ? page - 1 : null
    },
    links: {
      self: `/api/users?page=${page}&limit=${limit}`,
      first: `/api/users?page=1&limit=${limit}`,
      last: `/api/users?page=${totalPages}&limit=${limit}`,
      next: page < totalPages ? `/api/users?page=${page + 1}&limit=${limit}` : null,
      prev: page > 1 ? `/api/users?page=${page - 1}&limit=${limit}` : null
    }
  });
});

// ===== Pattern 2: Cursor Pagination =====
// เหมาะกับ: infinite scroll, real-time data

app.get('/api/posts', async (req, res) => {
  const limit = Math.min(100, parseInt(req.query.limit) || 20);
  const cursor = req.query.cursor; // base64 encoded cursor
  
  let afterId = null;
  if (cursor) {
    try {
      afterId = parseInt(Buffer.from(cursor, 'base64').toString());
    } catch {
      return res.status(400).json({ error: 'Invalid cursor' });
    }
  }
  
  const posts = await Post.findAll({
    where: afterId ? { id: { $gt: afterId } } : {},
    limit: limit + 1, // fetch one extra to check if has next
    orderBy: [['id', 'ASC']]
  });
  
  const hasNext = posts.length > limit;
  if (hasNext) posts.pop(); // remove extra item
  
  const nextCursor = hasNext
    ? Buffer.from(String(posts[posts.length - 1].id)).toString('base64')
    : null;
  
  res.json({
    data: posts,
    pagination: {
      limit,
      hasNext,
      cursor: nextCursor,
      nextUrl: nextCursor ? `/api/posts?cursor=${nextCursor}&limit=${limit}` : null
    }
  });
});

// ===== Pattern 3: Time-based cursor =====
app.get('/api/events', async (req, res) => {
  const limit = parseInt(req.query.limit) || 20;
  const before = req.query.before ? new Date(req.query.before) : new Date();
  
  const events = await Event.findAll({
    where: { createdAt: { $lt: before } },
    limit: limit + 1,
    orderBy: [['createdAt', 'DESC']]
  });
  
  const hasMore = events.length > limit;
  if (hasMore) events.pop();
  
  const oldestEvent = events[events.length - 1];
  
  res.json({
    data: events,
    pagination: {
      hasMore,
      before: oldestEvent ? oldestEvent.createdAt : null
    }
  });
});

// ===== Pagination helper =====
function createPagination(page, limit, total) {
  const totalPages = Math.ceil(total / limit);
  return {
    page,
    limit,
    total,
    totalPages,
    hasNext: page < totalPages,
    hasPrev: page > 1
  };
}
```

---

## Step 1161: Filtering และ Sorting

```javascript
const express = require('express');
const app = express();
app.use(express.json());

// ===== Filtering =====
app.get('/api/products', async (req, res) => {
  const {
    category,
    minPrice,
    maxPrice,
    brand,
    inStock,
    tags,
    search,
    page = 1,
    limit = 20,
    sort = 'created_at',
    order = 'desc'
  } = req.query;
  
  // สร้าง filter object
  const filters = {};
  
  if (category) filters.category = category;
  if (brand) filters.brand = brand;
  
  if (minPrice || maxPrice) {
    filters.price = {};
    if (minPrice) filters.price.$gte = parseFloat(minPrice);
    if (maxPrice) filters.price.$lte = parseFloat(maxPrice);
  }
  
  if (inStock !== undefined) {
    filters.stock = inStock === 'true' ? { $gt: 0 } : 0;
  }
  
  if (tags) {
    filters.tags = { $in: [].concat(tags) }; // handle single or array
  }
  
  if (search) {
    filters.$text = { $search: search }; // full-text search
  }
  
  // Sort validation
  const allowedSortFields = ['name', 'price', 'created_at', 'updated_at', 'stock'];
  const allowedOrders = ['asc', 'desc'];
  
  const sortField = allowedSortFields.includes(sort) ? sort : 'created_at';
  const sortOrder = allowedOrders.includes(order) ? order : 'desc';
  
  const products = await Product.findAll({
    where: filters,
    orderBy: [[sortField, sortOrder]],
    limit: parseInt(limit),
    offset: (parseInt(page) - 1) * parseInt(limit)
  });
  
  const total = await Product.count({ where: filters });
  
  res.json({
    data: products,
    filters: { category, minPrice, maxPrice, brand, inStock, search },
    pagination: createPagination(parseInt(page), parseInt(limit), total),
    sort: { field: sortField, order: sortOrder }
  });
});

// ===== Advanced filtering with filter syntax =====
// GET /api/users?filter[age][gte]=18&filter[role]=admin
// GET /api/products?filter[price][between]=100,500

app.get('/api/items', (req, res) => {
  const filterParams = req.query.filter || {};
  
  // Parse filter[field][operator]=value
  const filters = {};
  
  for (const [field, condition] of Object.entries(filterParams)) {
    if (typeof condition === 'string') {
      // filter[field]=value (exact match)
      filters[field] = condition;
    } else if (typeof condition === 'object') {
      // filter[field][operator]=value
      filters[field] = {};
      
      for (const [operator, value] of Object.entries(condition)) {
        switch (operator) {
          case 'eq': filters[field] = value; break;
          case 'ne': filters[field].$ne = value; break;
          case 'gt': filters[field].$gt = parseFloat(value); break;
          case 'gte': filters[field].$gte = parseFloat(value); break;
          case 'lt': filters[field].$lt = parseFloat(value); break;
          case 'lte': filters[field].$lte = parseFloat(value); break;
          case 'in': filters[field].$in = value.split(','); break;
          case 'nin': filters[field].$nin = value.split(','); break;
          case 'like': filters[field].$like = `%${value}%`; break;
          case 'between':
            const [min, max] = value.split(',').map(Number);
            filters[field].$between = [min, max];
            break;
        }
      }
    }
  }
  
  console.log('Filters:', JSON.stringify(filters, null, 2));
  res.json({ data: [], filters });
});
```

---

## Step 1162: Error Response Format Standards

```javascript
const express = require('express');
const app = express();
app.use(express.json());

// ===== Standard Error Format =====
// RFC 7807 - Problem Details for HTTP APIs

class ApiError extends Error {
  constructor({
    status = 500,
    code,
    message,
    detail,
    instance,
    errors
  } = {}) {
    super(message);
    this.status = status;
    this.code = code;
    this.detail = detail;
    this.instance = instance;
    this.errors = errors;
  }

  toJSON() {
    return {
      status: this.status,
      code: this.code,
      message: this.message,
      ...(this.detail && { detail: this.detail }),
      ...(this.instance && { instance: this.instance }),
      ...(this.errors && { errors: this.errors })
    };
  }
}

// Error types
const Errors = {
  NotFound: (resource, id) => new ApiError({
    status: 404,
    code: 'NOT_FOUND',
    message: `${resource} not found`,
    detail: id ? `No ${resource.toLowerCase()} found with id ${id}` : undefined
  }),
  
  Unauthorized: (message = 'Authentication required') => new ApiError({
    status: 401,
    code: 'UNAUTHORIZED',
    message
  }),
  
  Forbidden: (message = 'Access denied') => new ApiError({
    status: 403,
    code: 'FORBIDDEN',
    message
  }),
  
  Validation: (errors) => new ApiError({
    status: 422,
    code: 'VALIDATION_ERROR',
    message: 'Request validation failed',
    errors: errors.map(e => ({
      field: e.field,
      message: e.message,
      code: e.code || 'INVALID_VALUE',
      value: e.value
    }))
  }),
  
  Conflict: (message) => new ApiError({
    status: 409,
    code: 'CONFLICT',
    message
  }),
  
  RateLimit: (retryAfter) => new ApiError({
    status: 429,
    code: 'RATE_LIMIT_EXCEEDED',
    message: 'Too many requests',
    detail: `Try again after ${retryAfter} seconds`
  }),
  
  Internal: (message = 'Internal server error') => new ApiError({
    status: 500,
    code: 'INTERNAL_ERROR',
    message
  })
};

// Error handler middleware
app.use((err, req, res, next) => {
  let apiError;
  
  if (err instanceof ApiError) {
    apiError = err;
  } else if (err.name === 'ValidationError') {
    // Mongoose validation error
    apiError = Errors.Validation(
      Object.values(err.errors).map(e => ({
        field: e.path,
        message: e.message
      }))
    );
  } else if (err.code === '23505') {
    // PostgreSQL unique violation
    apiError = Errors.Conflict('Resource already exists');
  } else {
    // Unknown error
    if (process.env.NODE_ENV !== 'production') {
      console.error(err.stack);
    }
    apiError = Errors.Internal();
  }
  
  // เพิ่ม request ID สำหรับ debugging
  const body = {
    error: apiError.toJSON(),
    requestId: req.id,
    timestamp: new Date().toISOString()
  };
  
  res.status(apiError.status).json(body);
});

// ===== ตัวอย่างการใช้งาน =====
app.get('/api/users/:id', async (req, res, next) => {
  try {
    const user = await User.findById(req.params.id);
    if (!user) throw Errors.NotFound('User', req.params.id);
    res.json({ data: user });
  } catch (err) {
    next(err);
  }
});

app.post('/api/users', async (req, res, next) => {
  try {
    const existing = await User.findByEmail(req.body.email);
    if (existing) throw Errors.Conflict(`Email ${req.body.email} is already registered`);
    
    const user = await User.create(req.body);
    res.status(201).json({ data: user });
  } catch (err) {
    next(err);
  }
});
```

---

## Step 1163: Building Complete CRUD API

```javascript
const express = require('express');
const router = express.Router();
const { body, param, query, validationResult } = require('express-validator');

// ===== In-memory database =====
let products = [
  { id: 1, name: 'MacBook Pro', price: 59900, category: 'computers', stock: 50, createdAt: new Date('2024-01-01') },
  { id: 2, name: 'iPhone 15', price: 32900, category: 'phones', stock: 100, createdAt: new Date('2024-01-02') },
  { id: 3, name: 'AirPods Pro', price: 9490, category: 'accessories', stock: 200, createdAt: new Date('2024-01-03') }
];
let nextId = 4;

// ===== Validation schemas =====
const productValidation = {
  create: [
    body('name').trim().notEmpty().withMessage('Name is required')
      .isLength({ min: 2, max: 200 }),
    body('price').isFloat({ min: 0 }).withMessage('Price must be non-negative'),
    body('category').notEmpty().withMessage('Category is required'),
    body('stock').isInt({ min: 0 }).withMessage('Stock must be non-negative'),
    body('description').optional().isLength({ max: 2000 })
  ],
  update: [
    param('id').isInt({ min: 1 }),
    body('name').optional().trim().isLength({ min: 2, max: 200 }),
    body('price').optional().isFloat({ min: 0 }),
    body('stock').optional().isInt({ min: 0 })
  ]
};

const validate = (req, res, next) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(422).json({
      error: { code: 'VALIDATION_ERROR', message: 'Validation failed',
        details: errors.array().map(e => ({ field: e.path, message: e.msg })) }
    });
  }
  next();
};

// ===== GET /products =====
router.get('/', [
  query('page').optional().isInt({ min: 1 }).toInt(),
  query('limit').optional().isInt({ min: 1, max: 100 }).toInt(),
  query('sort').optional().isIn(['name', 'price', 'createdAt', 'stock']),
  query('order').optional().isIn(['asc', 'desc']),
  query('category').optional().trim(),
  query('search').optional().trim(),
  validate
], (req, res) => {
  const { page = 1, limit = 10, sort = 'createdAt', order = 'desc', category, search } = req.query;
  
  let result = [...products];
  
  // Filter
  if (category) {
    result = result.filter(p => p.category.toLowerCase() === category.toLowerCase());
  }
  
  if (search) {
    result = result.filter(p =>
      p.name.toLowerCase().includes(search.toLowerCase())
    );
  }
  
  // Sort
  result.sort((a, b) => {
    if (order === 'asc') {
      return a[sort] > b[sort] ? 1 : -1;
    }
    return a[sort] < b[sort] ? 1 : -1;
  });
  
  const total = result.length;
  const offset = (page - 1) * limit;
  const data = result.slice(offset, offset + limit);
  
  res.json({
    data,
    pagination: {
      page, limit, total,
      totalPages: Math.ceil(total / limit),
      hasNext: page * limit < total,
      hasPrev: page > 1
    }
  });
});

// ===== GET /products/:id =====
router.get('/:id', [
  param('id').isInt({ min: 1 }).toInt(), validate
], (req, res) => {
  const product = products.find(p => p.id === req.params.id);
  
  if (!product) {
    return res.status(404).json({
      error: { code: 'NOT_FOUND', message: `Product ${req.params.id} not found` }
    });
  }
  
  res.json({ data: product });
});

// ===== POST /products =====
router.post('/', [...productValidation.create, validate], (req, res) => {
  const { name, price, category, stock, description } = req.body;
  
  const product = {
    id: nextId++,
    name,
    price,
    category,
    stock,
    description: description || null,
    createdAt: new Date(),
    updatedAt: new Date()
  };
  
  products.push(product);
  
  res.status(201)
    .location(`/api/products/${product.id}`)
    .json({ data: product });
});

// ===== PUT /products/:id =====
router.put('/:id', [
  param('id').isInt({ min: 1 }).toInt(),
  ...productValidation.create,
  validate
], (req, res) => {
  const index = products.findIndex(p => p.id === req.params.id);
  
  if (index === -1) {
    return res.status(404).json({
      error: { code: 'NOT_FOUND', message: `Product ${req.params.id} not found` }
    });
  }
  
  const { name, price, category, stock, description } = req.body;
  
  products[index] = {
    id: req.params.id,
    name,
    price,
    category,
    stock,
    description: description || null,
    createdAt: products[index].createdAt,
    updatedAt: new Date()
  };
  
  res.json({ data: products[index] });
});

// ===== PATCH /products/:id =====
router.patch('/:id', [...productValidation.update, validate], (req, res) => {
  const index = products.findIndex(p => p.id === req.params.id);
  
  if (index === -1) {
    return res.status(404).json({
      error: { code: 'NOT_FOUND', message: `Product ${req.params.id} not found` }
    });
  }
  
  // อัปเดตเฉพาะฟิลด์ที่ส่งมา
  const allowedFields = ['name', 'price', 'category', 'stock', 'description'];
  const updates = {};
  
  for (const field of allowedFields) {
    if (req.body[field] !== undefined) {
      updates[field] = req.body[field];
    }
  }
  
  products[index] = {
    ...products[index],
    ...updates,
    updatedAt: new Date()
  };
  
  res.json({ data: products[index] });
});

// ===== DELETE /products/:id =====
router.delete('/:id', [
  param('id').isInt({ min: 1 }).toInt(), validate
], (req, res) => {
  const index = products.findIndex(p => p.id === req.params.id);
  
  if (index === -1) {
    return res.status(404).json({
      error: { code: 'NOT_FOUND', message: `Product ${req.params.id} not found` }
    });
  }
  
  products.splice(index, 1);
  res.status(204).send();
});

module.exports = router;
```

---

## Step 1164: HATEOAS Concept

```javascript
// HATEOAS = Hypermedia As The Engine Of Application State
// ทำให้ API self-describing โดยมี links ใน response

// ===== HATEOAS Response Example =====
app.get('/api/v1/orders/:id', authenticate, (req, res) => {
  const order = getOrder(req.params.id);
  
  if (!order) {
    return res.status(404).json({ error: 'Order not found' });
  }
  
  // เพิ่ม links ตาม state ของ order
  const links = {
    self: { href: `/api/v1/orders/${order.id}`, method: 'GET' },
    user: { href: `/api/v1/users/${order.userId}`, method: 'GET' }
  };
  
  // ใส่ links ตาม business logic
  if (order.status === 'pending') {
    links.confirm = { href: `/api/v1/orders/${order.id}/confirm`, method: 'POST' };
    links.cancel = { href: `/api/v1/orders/${order.id}/cancel`, method: 'POST' };
  }
  
  if (order.status === 'confirmed') {
    links.ship = { href: `/api/v1/orders/${order.id}/ship`, method: 'POST' };
    links.cancel = { href: `/api/v1/orders/${order.id}/cancel`, method: 'POST' };
  }
  
  if (order.status === 'shipped') {
    links.track = { href: `/api/v1/orders/${order.id}/tracking`, method: 'GET' };
  }
  
  if (['pending', 'confirmed'].includes(order.status)) {
    links.items = { href: `/api/v1/orders/${order.id}/items`, method: 'GET' };
  }
  
  res.json({
    data: order,
    _links: links
  });
});

// ===== Helper สำหรับสร้าง links =====
function createOrderLinks(order, baseUrl = '') {
  const base = `${baseUrl}/api/v1/orders/${order.id}`;
  
  const links = {
    self: { href: base, method: 'GET' },
    update: { href: base, method: 'PATCH' }
  };
  
  const transitions = {
    pending: ['confirm', 'cancel'],
    confirmed: ['ship', 'cancel'],
    shipped: ['deliver'],
    cancelled: []
  };
  
  const available = transitions[order.status] || [];
  
  for (const action of available) {
    links[action] = { href: `${base}/${action}`, method: 'POST' };
  }
  
  return links;
}
```

---

## Step 1165: Rate Limiting Strategies

```javascript
const express = require('express');
const rateLimit = require('express-rate-limit');
const RedisStore = require('rate-limit-redis');

const app = express();

// ===== Strategy 1: Global limit =====
app.use(rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 1000,
  message: { error: 'Too many requests' }
}));

// ===== Strategy 2: Per-endpoint limits =====
// Login - strict
const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5,
  skipSuccessfulRequests: true,  // ไม่นับ successful logins
  message: {
    error: 'Too many login attempts',
    retryAfter: '15 minutes'
  }
});

// API - moderate
const apiLimiter = rateLimit({
  windowMs: 60 * 1000,
  max: 60
});

// Public endpoints - loose
const publicLimiter = rateLimit({
  windowMs: 60 * 1000,
  max: 100
});

app.use('/api/auth/login', loginLimiter);
app.use('/api/', apiLimiter);
app.use('/public/', publicLimiter);

// ===== Strategy 3: Per-user limits =====
const userLimiter = rateLimit({
  windowMs: 60 * 1000,
  max: (req) => {
    // Premium users get higher limits
    if (req.user?.tier === 'premium') return 120;
    if (req.user?.tier === 'business') return 300;
    return 60;
  },
  keyGenerator: (req) => req.user?.id || req.ip
});

// ===== Strategy 4: Sliding window with Redis =====
const slidingLimiter = rateLimit({
  windowMs: 60 * 1000,
  max: 100,
  store: new RedisStore({
    client: redisClient,
    prefix: 'rl:',
    sendCommand: (...args) => redisClient.sendCommand(args)
  }),
  standardHeaders: 'draft-7',
  legacyHeaders: false
});

// ===== Custom rate limit response =====
const customLimiter = rateLimit({
  windowMs: 60 * 1000,
  max: 30,
  handler: (req, res) => {
    const resetTime = new Date(req.rateLimit.resetTime);
    
    res.status(429).json({
      error: {
        code: 'RATE_LIMIT_EXCEEDED',
        message: 'Too many requests',
        limit: req.rateLimit.limit,
        remaining: 0,
        reset: resetTime.toISOString(),
        retryAfterSeconds: Math.ceil((resetTime - Date.now()) / 1000)
      }
    });
  }
});
```

---

## Step 1166: Complete REST API Example

```javascript
// ===== Complete Blog API =====

const express = require('express');
const { body, param, query, validationResult } = require('express-validator');

const app = express();
app.use(express.json());

// Mock data
let posts = [
  {
    id: 1, title: 'Getting Started with Node.js',
    content: 'Node.js is...', slug: 'getting-started-nodejs',
    authorId: 1, status: 'published', tags: ['nodejs', 'javascript'],
    views: 1250, createdAt: new Date('2024-01-01'), updatedAt: new Date('2024-01-01')
  }
];
let comments = [
  { id: 1, postId: 1, authorId: 2, content: 'Great post!', createdAt: new Date() }
];

// ===== Posts Router =====
const postsRouter = express.Router();

postsRouter.get('/', [
  query('page').optional().isInt({ min: 1 }).toInt(),
  query('limit').optional().isInt({ min: 1, max: 50 }).toInt(),
  query('status').optional().isIn(['draft', 'published', 'archived']),
  query('tag').optional().trim(),
  query('author').optional().isInt().toInt(),
  query('sort').optional().isIn(['createdAt', 'updatedAt', 'views', 'title']),
  query('order').optional().isIn(['asc', 'desc'])
], (req, res) => {
  const {
    page = 1, limit = 10, status, tag, author,
    sort = 'createdAt', order = 'desc', search
  } = req.query;
  
  let result = [...posts];
  
  if (status) result = result.filter(p => p.status === status);
  if (tag) result = result.filter(p => p.tags.includes(tag));
  if (author) result = result.filter(p => p.authorId === author);
  if (search) result = result.filter(p =>
    p.title.toLowerCase().includes(search.toLowerCase())
  );
  
  result.sort((a, b) => {
    const cmp = a[sort] > b[sort] ? 1 : -1;
    return order === 'asc' ? cmp : -cmp;
  });
  
  const total = result.length;
  const data = result.slice((page - 1) * limit, page * limit);
  
  res.json({
    data: data.map(p => ({ ...p, commentCount: comments.filter(c => c.postId === p.id).length })),
    pagination: { page, limit, total, totalPages: Math.ceil(total / limit) }
  });
});

postsRouter.get('/:slug', (req, res) => {
  const post = posts.find(p => p.slug === req.params.slug);
  if (!post) return res.status(404).json({ error: { code: 'NOT_FOUND', message: 'Post not found' } });
  
  post.views++;
  
  const postComments = comments.filter(c => c.postId === post.id);
  res.json({ data: { ...post, comments: postComments } });
});

postsRouter.post('/', [
  body('title').trim().notEmpty().isLength({ max: 200 }),
  body('content').trim().notEmpty(),
  body('tags').optional().isArray(),
  body('status').optional().isIn(['draft', 'published'])
], (req, res) => {
  const errors = validationResult(req);
  if (!errors.isEmpty()) {
    return res.status(422).json({ error: { code: 'VALIDATION_ERROR', details: errors.array() } });
  }
  
  const { title, content, tags = [], status = 'draft' } = req.body;
  const slug = title.toLowerCase().replace(/[^a-z0-9]+/g, '-').replace(/^-|-$/g, '');
  
  if (posts.find(p => p.slug === slug)) {
    return res.status(409).json({ error: { code: 'CONFLICT', message: 'Slug already exists' } });
  }
  
  const post = {
    id: posts.length + 1, title, content, slug,
    authorId: req.user?.id || 1, status, tags, views: 0,
    createdAt: new Date(), updatedAt: new Date()
  };
  
  posts.push(post);
  res.status(201).location(`/api/v1/posts/${slug}`).json({ data: post });
});

app.use('/api/v1/posts', postsRouter);

app.listen(3000, () => console.log('Blog API running on port 3000'));
```

---

## Step 1167-1170: API Testing

```javascript
// ===== Testing with Jest + Supertest =====
const request = require('supertest');
const app = require('../src/app');

describe('Products API', () => {
  let authToken;
  let createdProductId;
  
  beforeAll(async () => {
    const loginRes = await request(app)
      .post('/api/auth/login')
      .send({ email: 'admin@example.com', password: 'Admin123!' });
    authToken = loginRes.body.token;
  });
  
  describe('GET /api/products', () => {
    it('returns paginated list', async () => {
      const res = await request(app)
        .get('/api/products?page=1&limit=5')
        .expect(200);
      
      expect(res.body).toMatchObject({
        data: expect.any(Array),
        pagination: {
          page: 1,
          limit: 5,
          total: expect.any(Number)
        }
      });
    });
    
    it('filters by category', async () => {
      const res = await request(app)
        .get('/api/products?category=computers')
        .expect(200);
      
      res.body.data.forEach(product => {
        expect(product.category).toBe('computers');
      });
    });
    
    it('sorts by price descending', async () => {
      const res = await request(app)
        .get('/api/products?sort=price&order=desc')
        .expect(200);
      
      const prices = res.body.data.map(p => p.price);
      const sorted = [...prices].sort((a, b) => b - a);
      expect(prices).toEqual(sorted);
    });
  });
  
  describe('POST /api/products', () => {
    it('creates product with valid data', async () => {
      const product = {
        name: 'Test Product',
        price: 999,
        category: 'test',
        stock: 100
      };
      
      const res = await request(app)
        .post('/api/products')
        .set('Authorization', `Bearer ${authToken}`)
        .send(product)
        .expect(201);
      
      expect(res.headers.location).toMatch(/\/api\/products\/\d+/);
      expect(res.body.data).toMatchObject(product);
      expect(res.body.data.id).toBeDefined();
      
      createdProductId = res.body.data.id;
    });
    
    it('validates required fields', async () => {
      const res = await request(app)
        .post('/api/products')
        .set('Authorization', `Bearer ${authToken}`)
        .send({ price: -100 })
        .expect(422);
      
      expect(res.body.error.code).toBe('VALIDATION_ERROR');
      expect(res.body.error.details.length).toBeGreaterThan(0);
    });
    
    it('requires authentication', async () => {
      await request(app)
        .post('/api/products')
        .send({ name: 'Test', price: 100, category: 'test', stock: 1 })
        .expect(401);
    });
  });
  
  describe('PATCH /api/products/:id', () => {
    it('updates specific fields', async () => {
      const res = await request(app)
        .patch(`/api/products/${createdProductId}`)
        .set('Authorization', `Bearer ${authToken}`)
        .send({ price: 1999, stock: 50 })
        .expect(200);
      
      expect(res.body.data.price).toBe(1999);
      expect(res.body.data.stock).toBe(50);
    });
  });
  
  describe('DELETE /api/products/:id', () => {
    it('deletes product', async () => {
      await request(app)
        .delete(`/api/products/${createdProductId}`)
        .set('Authorization', `Bearer ${authToken}`)
        .expect(204);
      
      await request(app)
        .get(`/api/products/${createdProductId}`)
        .expect(404);
    });
  });
});
```

---

## สรุป Steps 1151-1170

| Step | หัวข้อ |
|------|--------|
| 1151 | REST Principles |
| 1152 | HTTP Methods และ Semantics |
| 1153 | HTTP Status Codes |
| 1154 | RESTful URL Design |
| 1155 | API Versioning |
| 1156 | Input Validation (express-validator) |
| 1157 | Input Validation (Zod) |
| 1158 | JWT Authentication |
| 1159 | API Documentation (Swagger) |
| 1160 | Pagination Patterns |
| 1161 | Filtering and Sorting |
| 1162 | Error Response Standards |
| 1163 | Complete CRUD API |
| 1164 | HATEOAS |
| 1165 | Rate Limiting Strategies |
| 1166 | Complete REST API Example |
| 1167-1170 | API Testing |

---

## แบบฝึกหัดท้ายบท

### แบบฝึกหัดที่ 1: Library API
ออกแบบและสร้าง REST API สำหรับห้องสมุดที่มี:
1. Books: CRUD, search, filter by genre/author/year
2. Authors: CRUD, books by author
3. Members: register, borrow history
4. Borrowings: borrow book, return book, overdue check
5. Authentication และ Authorization (admin/librarian/member)

### แบบฝึกหัดที่ 2: Social Media API
สร้าง API สำหรับ social media:
1. Users: profile, follow/unfollow
2. Posts: text, images, hashtags
3. Comments: nested comments
4. Likes: like posts/comments
5. Feed: timeline, trending, pagination (cursor-based)

### แบบฝึกหัดที่ 3: E-Commerce API
สร้าง API สำหรับ e-commerce:
1. Products + inventory management
2. Shopping cart
3. Orders + order lifecycle
4. Payment simulation
5. Admin dashboard endpoints

---

*จบ Part 59: RESTful API Design - ในบทต่อไปเราจะเรียน GraphQL*
