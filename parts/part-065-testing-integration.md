# Part 65: Testing - Integration Tests (Steps 1271-1290)

## บทนำ

Integration Tests ทดสอบการทำงานร่วมกันของหลาย components เช่น API + Database หรือ Service + External API บทนี้จะสอนการเขียน Integration Tests สำหรับ Express API ด้วย Supertest และ real database

---

## Step 1271: Integration Testing คืออะไร?

```javascript
// Unit Test vs Integration Test

// Unit Test:
// - ทดสอบ function/class แยกกัน
// - Mock dependencies ทั้งหมด
// - เร็ว ไม่มี side effects

// Integration Test:
// - ทดสอบหลาย components ทำงานร่วมกัน
// - ใช้ real dependencies (database, etc.)
// - ช้ากว่า แต่ test ได้สิ่งที่ใกล้เคียงจริงมากกว่า

// ตัวอย่าง Integration Test
// ทดสอบ: POST /users -> User Service -> Database

// ลำดับ:
// 1. ส่ง HTTP request
// 2. Middleware จัดการ
// 3. Controller เรียก service
// 4. Service เรียก database
// 5. Database บันทึกข้อมูล
// 6. ส่ง response กลับ

// สิ่งที่ Integration Test ตรวจสอบ:
// - HTTP status codes ถูกต้อง
// - Response body มีข้อมูลที่ถูกต้อง
// - ข้อมูลถูกบันทึกลง database
// - Validation ทำงาน
// - Authentication/Authorization ทำงาน
```

---

## Step 1272: Project Structure สำหรับ Testing

```javascript
// โครงสร้าง project สำหรับ integration tests

/*
project/
├── src/
│   ├── app.js          (Express app)
│   ├── routes/
│   ├── controllers/
│   ├── services/
│   ├── models/
│   └── middleware/
├── tests/
│   ├── unit/
│   │   └── *.test.js
│   ├── integration/
│   │   └── *.test.js
│   └── fixtures/
│       └── *.js
├── jest.config.js
├── jest.setup.js
└── package.json
*/

// package.json
/*
{
  "scripts": {
    "test": "jest",
    "test:unit": "jest tests/unit",
    "test:integration": "jest tests/integration",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage"
  },
  "jest": {
    "testEnvironment": "node",
    "setupFilesAfterFramework": ["./jest.setup.js"],
    "testMatch": [
      "**/__tests__/**/*.js",
      "**/*.test.js"
    ]
  }
}
*/

// jest.setup.js
const mongoose = require('mongoose');

beforeAll(async () => {
  // เชื่อมต่อ test database
  await mongoose.connect(process.env.TEST_DB_URL || 'mongodb://localhost:27017/testdb');
});

afterAll(async () => {
  // ปิดการเชื่อมต่อ
  await mongoose.connection.close();
});

afterEach(async () => {
  // ล้างข้อมูลระหว่าง tests
  const collections = mongoose.connection.collections;
  for (const key in collections) {
    await collections[key].deleteMany({});
  }
});
```

---

## Step 1273: Testing Express API กับ Supertest

```javascript
// ติดตั้ง: npm install --save-dev supertest

// src/app.js - Express app
const express = require('express');
const mongoose = require('mongoose');
const userRouter = require('./routes/users');

const app = express();

app.use(express.json());
app.use('/api/users', userRouter);

// Error handler
app.use((err, req, res, next) => {
  res.status(err.status || 500).json({
    error: err.message
  });
});

module.exports = app;

// tests/integration/users.test.js
const request = require('supertest');
const app = require('../../src/app');
const User = require('../../src/models/User');

describe('Users API Integration Tests', () => {
  
  // เตรียมข้อมูลทดสอบ
  let testUser;
  let authToken;
  
  beforeEach(async () => {
    // สร้าง test user
    testUser = await User.create({
      name: 'Test User',
      email: 'test@example.com',
      password: '$2a$12$hash...' // bcrypt hash ของ 'password123'
    });
    
    // Login เพื่อรับ token
    const loginRes = await request(app)
      .post('/api/auth/login')
      .send({ email: 'test@example.com', password: 'password123' });
    
    authToken = loginRes.body.token;
  });
  
  describe('GET /api/users', () => {
    test('ควรส่ง list of users', async () => {
      const res = await request(app)
        .get('/api/users')
        .set('Authorization', `Bearer ${authToken}`)
        .expect(200);
      
      expect(res.body.users).toBeInstanceOf(Array);
      expect(res.body.users.length).toBeGreaterThan(0);
      expect(res.body.pagination).toBeDefined();
    });
    
    test('ควรส่ง 401 เมื่อไม่มี token', async () => {
      await request(app)
        .get('/api/users')
        .expect(401);
    });
    
    test('ควร paginate ได้', async () => {
      // สร้าง users เพิ่ม
      await User.insertMany([
        { name: 'User 2', email: 'user2@test.com', password: 'hash' },
        { name: 'User 3', email: 'user3@test.com', password: 'hash' },
      ]);
      
      const res = await request(app)
        .get('/api/users?page=1&limit=2')
        .set('Authorization', `Bearer ${authToken}`)
        .expect(200);
      
      expect(res.body.users).toHaveLength(2);
      expect(res.body.pagination.total).toBeGreaterThanOrEqual(3);
    });
  });
  
  describe('GET /api/users/:id', () => {
    test('ควรส่ง user เมื่อพบ', async () => {
      const res = await request(app)
        .get(`/api/users/${testUser._id}`)
        .set('Authorization', `Bearer ${authToken}`)
        .expect(200);
      
      expect(res.body.id).toBe(testUser._id.toString());
      expect(res.body.name).toBe(testUser.name);
      expect(res.body.password).toBeUndefined(); // ไม่ควร return password
    });
    
    test('ควรส่ง 404 เมื่อไม่พบ user', async () => {
      const fakeId = '507f1f77bcf86cd799439011';
      
      await request(app)
        .get(`/api/users/${fakeId}`)
        .set('Authorization', `Bearer ${authToken}`)
        .expect(404);
    });
    
    test('ควรส่ง 400 เมื่อ ID ไม่ถูกต้อง', async () => {
      await request(app)
        .get('/api/users/invalid-id')
        .set('Authorization', `Bearer ${authToken}`)
        .expect(400);
    });
  });
});
```

---

## Step 1274: Testing CRUD Operations

```javascript
// ทดสอบ CRUD ครบวงจร

describe('User CRUD Integration Tests', () => {
  
  describe('POST /api/users (Create)', () => {
    test('ควรสร้าง user ใหม่', async () => {
      const userData = {
        name: 'New User',
        email: 'new@example.com',
        password: 'StrongPass@123'
      };
      
      const res = await request(app)
        .post('/api/users')
        .send(userData)
        .expect(201);
      
      // ตรวจสอบ response
      expect(res.body.id).toBeDefined();
      expect(res.body.name).toBe(userData.name);
      expect(res.body.email).toBe(userData.email);
      expect(res.body.password).toBeUndefined();
      
      // ตรวจสอบว่าถูกบันทึกใน database จริง
      const dbUser = await User.findById(res.body.id);
      expect(dbUser).not.toBeNull();
      expect(dbUser.email).toBe(userData.email);
    });
    
    test('ควรส่ง 400 เมื่อ email ซ้ำ', async () => {
      const userData = {
        name: 'Test',
        email: 'test@example.com', // email นี้มีแล้ว
        password: 'Pass@123'
      };
      
      const res = await request(app)
        .post('/api/users')
        .send(userData)
        .expect(400);
      
      expect(res.body.error).toMatch(/email/i);
    });
    
    test('ควรส่ง 400 เมื่อข้อมูลไม่ครบ', async () => {
      await request(app)
        .post('/api/users')
        .send({ name: 'No Email' })
        .expect(400);
    });
    
    test('ควรส่ง 400 เมื่อรหัสผ่านอ่อนเกินไป', async () => {
      await request(app)
        .post('/api/users')
        .send({
          name: 'Test',
          email: 'test2@example.com',
          password: '123' // อ่อนเกินไป
        })
        .expect(400);
    });
  });
  
  describe('PUT /api/users/:id (Update)', () => {
    test('ควรอัพเดต user', async () => {
      const updateData = { name: 'Updated Name' };
      
      const res = await request(app)
        .put(`/api/users/${testUser._id}`)
        .set('Authorization', `Bearer ${authToken}`)
        .send(updateData)
        .expect(200);
      
      expect(res.body.name).toBe('Updated Name');
      
      // ตรวจสอบใน database
      const dbUser = await User.findById(testUser._id);
      expect(dbUser.name).toBe('Updated Name');
    });
    
    test('ควรส่ง 403 เมื่ออัพเดต user คนอื่น', async () => {
      // สร้าง user อื่น
      const otherUser = await User.create({
        name: 'Other User',
        email: 'other@example.com',
        password: 'hash'
      });
      
      await request(app)
        .put(`/api/users/${otherUser._id}`)
        .set('Authorization', `Bearer ${authToken}`)
        .send({ name: 'Hacked' })
        .expect(403);
    });
  });
  
  describe('DELETE /api/users/:id (Delete)', () => {
    test('ควรลบ user', async () => {
      await request(app)
        .delete(`/api/users/${testUser._id}`)
        .set('Authorization', `Bearer ${authToken}`)
        .expect(200);
      
      // ตรวจสอบว่าถูกลบจาก database
      const dbUser = await User.findById(testUser._id);
      expect(dbUser).toBeNull();
    });
    
    test('ควรส่ง 404 เมื่อลบ user ที่ไม่มี', async () => {
      const fakeId = '507f1f77bcf86cd799439011';
      
      await request(app)
        .delete(`/api/users/${fakeId}`)
        .set('Authorization', `Bearer ${authToken}`)
        .expect(404);
    });
  });
});
```

---

## Step 1275: Testing Authentication Flow

```javascript
// ทดสอบ Authentication ครบวงจร

describe('Authentication Integration Tests', () => {
  
  describe('POST /api/auth/register', () => {
    test('ควร register สำเร็จ', async () => {
      const res = await request(app)
        .post('/api/auth/register')
        .send({
          name: 'New User',
          email: 'newuser@example.com',
          password: 'StrongPass@123',
          confirmPassword: 'StrongPass@123'
        })
        .expect(201);
      
      expect(res.body.message).toBeDefined();
      expect(res.body.user.email).toBe('newuser@example.com');
      
      // รหัสผ่านต้อง hash แล้ว
      const dbUser = await User.findOne({ email: 'newuser@example.com' }).select('+password');
      expect(dbUser.password).not.toBe('StrongPass@123');
      expect(dbUser.password).toMatch(/^\$2[ab]\$/); // bcrypt format
    });
    
    test('ควรส่ง 400 เมื่อ password ไม่ตรงกัน', async () => {
      await request(app)
        .post('/api/auth/register')
        .send({
          name: 'Test',
          email: 'test3@example.com',
          password: 'Pass@123',
          confirmPassword: 'DifferentPass@123'
        })
        .expect(400);
    });
  });
  
  describe('POST /api/auth/login', () => {
    test('ควร login สำเร็จและรับ token', async () => {
      const res = await request(app)
        .post('/api/auth/login')
        .send({
          email: 'test@example.com',
          password: 'password123'
        })
        .expect(200);
      
      expect(res.body.accessToken).toBeDefined();
      expect(res.body.user).toBeDefined();
      expect(res.body.user.email).toBe('test@example.com');
      expect(res.body.user.password).toBeUndefined();
    });
    
    test('ควรส่ง 401 เมื่อรหัสผ่านผิด', async () => {
      const res = await request(app)
        .post('/api/auth/login')
        .send({
          email: 'test@example.com',
          password: 'wrongpassword'
        })
        .expect(401);
      
      expect(res.body.error).toBeDefined();
    });
    
    test('ควรส่ง 401 เมื่อ email ไม่มีในระบบ', async () => {
      await request(app)
        .post('/api/auth/login')
        .send({
          email: 'notexist@example.com',
          password: 'password123'
        })
        .expect(401);
    });
    
    test('ควรมี rate limiting', async () => {
      // ลอง login ผิดหลายครั้ง
      const loginAttempts = Array(6).fill(null);
      
      for (const _ of loginAttempts) {
        await request(app)
          .post('/api/auth/login')
          .send({ email: 'test@example.com', password: 'wrong' });
      }
      
      // ครั้งที่ 6 ควรถูก rate limit
      const res = await request(app)
        .post('/api/auth/login')
        .send({ email: 'test@example.com', password: 'wrong' });
      
      expect(res.status).toBe(429);
    });
  });
  
  describe('Token Refresh', () => {
    test('ควร refresh access token', async () => {
      // Login เพื่อรับ refresh token
      const loginRes = await request(app)
        .post('/api/auth/login')
        .send({ email: 'test@example.com', password: 'password123' });
      
      const refreshToken = loginRes.headers['set-cookie']
        .find(c => c.startsWith('refreshToken='));
      
      const res = await request(app)
        .post('/api/auth/refresh')
        .set('Cookie', refreshToken)
        .expect(200);
      
      expect(res.body.accessToken).toBeDefined();
      expect(res.body.accessToken).not.toBe(loginRes.body.accessToken);
    });
    
    test('ควรส่ง 401 เมื่อไม่มี refresh token', async () => {
      await request(app)
        .post('/api/auth/refresh')
        .expect(401);
    });
  });
  
  describe('Logout', () => {
    test('ควร logout สำเร็จ', async () => {
      await request(app)
        .post('/api/auth/logout')
        .set('Authorization', `Bearer ${authToken}`)
        .expect(200);
      
      // ลอง access protected route ด้วย token เดิม
      const res = await request(app)
        .get('/api/users/me')
        .set('Authorization', `Bearer ${authToken}`);
      
      // Token ควรใช้ไม่ได้แล้ว (ถ้า implement token blacklist)
      // หรือรอ token expire
    });
  });
});
```

---

## Step 1276: Database Cleanup Between Tests

```javascript
// กลยุทธ์การ cleanup database

// 1. deleteMany() หลังแต่ละ test
afterEach(async () => {
  await User.deleteMany({});
  await Post.deleteMany({});
  await Comment.deleteMany({});
});

// 2. Transaction Rollback (PostgreSQL)
describe('with transaction rollback', () => {
  let client;
  
  beforeEach(async () => {
    client = await pool.connect();
    await client.query('BEGIN');
  });
  
  afterEach(async () => {
    await client.query('ROLLBACK');
    client.release();
  });
  
  test('ควรสร้าง user', async () => {
    const result = await client.query(
      'INSERT INTO users (name, email) VALUES ($1, $2) RETURNING *',
      ['Test', 'test@test.com']
    );
    expect(result.rows[0].name).toBe('Test');
    // หลัง test, ROLLBACK จะลบข้อมูลทั้งหมด
  });
});

// 3. MongoDB in-memory server (เร็วที่สุด)
// ติดตั้ง: npm install --save-dev mongodb-memory-server

const { MongoMemoryServer } = require('mongodb-memory-server');
const mongoose = require('mongoose');

let mongoServer;

beforeAll(async () => {
  mongoServer = await MongoMemoryServer.create();
  const mongoUri = mongoServer.getUri();
  await mongoose.connect(mongoUri);
});

afterAll(async () => {
  await mongoose.disconnect();
  await mongoServer.stop();
});

afterEach(async () => {
  const collections = mongoose.connection.collections;
  for (const key in collections) {
    await collections[key].deleteMany({});
  }
});

// 4. Test containers (Docker)
// ใช้ testcontainers-node สำหรับ real database
// const { GenericContainer } = require('testcontainers');
// 
// const container = await new GenericContainer('postgres:14')
//   .withEnvironment({ POSTGRES_PASSWORD: 'test' })
//   .withExposedPorts(5432)
//   .start();
```

---

## Step 1277: Test Fixtures และ Factories

```javascript
// Fixtures - ข้อมูลทดสอบที่กำหนดไว้ล่วงหน้า
// Factories - function ที่สร้างข้อมูลทดสอบ

// tests/fixtures/users.js
const bcrypt = require('bcryptjs');

const fixtures = {
  users: {
    admin: {
      name: 'Admin User',
      email: 'admin@test.com',
      password: 'AdminPass@123',
      role: 'admin',
      isActive: true
    },
    regular: {
      name: 'Regular User',
      email: 'user@test.com',
      password: 'UserPass@123',
      role: 'user',
      isActive: true
    },
    inactive: {
      name: 'Inactive User',
      email: 'inactive@test.com',
      password: 'InactivePass@123',
      role: 'user',
      isActive: false
    }
  }
};

module.exports = fixtures;

// tests/factories/userFactory.js
const bcrypt = require('bcryptjs');
const User = require('../../src/models/User');

let counter = 0;

async function createUser(overrides = {}) {
  counter++;
  
  const defaults = {
    name: `Test User ${counter}`,
    email: `user${counter}@test.com`,
    password: await bcrypt.hash('password123', 10),
    role: 'user',
    isActive: true
  };
  
  return User.create({ ...defaults, ...overrides });
}

async function createAdmin(overrides = {}) {
  return createUser({ role: 'admin', ...overrides });
}

async function createUserWithPosts(postCount = 3, overrides = {}) {
  const user = await createUser(overrides);
  
  const posts = await Promise.all(
    Array.from({ length: postCount }, (_, i) =>
      Post.create({
        title: `Test Post ${i + 1}`,
        content: 'Test content',
        authorId: user._id,
        isPublished: true
      })
    )
  );
  
  return { user, posts };
}

module.exports = { createUser, createAdmin, createUserWithPosts };

// การใช้งาน
const { createUser, createAdmin } = require('../factories/userFactory');

describe('Posts API', () => {
  let admin;
  let regularUser;
  let adminToken;
  let userToken;
  
  beforeEach(async () => {
    admin = await createAdmin({ email: 'admin@test.com' });
    regularUser = await createUser({ email: 'user@test.com' });
    
    // สร้าง tokens
    adminToken = createToken(admin);
    userToken = createToken(regularUser);
  });
  
  test('admin ควรลบโพสต์ได้', async () => {
    const post = await Post.create({ title: 'Test', authorId: regularUser._id });
    
    await request(app)
      .delete(`/api/posts/${post._id}`)
      .set('Authorization', `Bearer ${adminToken}`)
      .expect(200);
  });
  
  test('user ธรรมดาไม่ควรลบโพสต์คนอื่น', async () => {
    const admin2 = await createAdmin({ email: 'admin2@test.com' });
    const post = await Post.create({ title: 'Admin Post', authorId: admin2._id });
    
    await request(app)
      .delete(`/api/posts/${post._id}`)
      .set('Authorization', `Bearer ${userToken}`)
      .expect(403);
  });
});
```

---

## Step 1278: Testing File Uploads

```javascript
// ทดสอบ file upload

// src/routes/uploads.js
const express = require('express');
const multer = require('multer');
const path = require('path');

const router = express.Router();

const storage = multer.diskStorage({
  destination: 'uploads/',
  filename: (req, file, cb) => {
    const uniqueSuffix = Date.now() + '-' + Math.round(Math.random() * 1E9);
    cb(null, uniqueSuffix + path.extname(file.originalname));
  }
});

const upload = multer({
  storage,
  limits: { fileSize: 5 * 1024 * 1024 }, // 5MB
  fileFilter: (req, file, cb) => {
    if (file.mimetype.startsWith('image/')) {
      cb(null, true);
    } else {
      cb(new Error('เฉพาะไฟล์รูปภาพเท่านั้น'));
    }
  }
});

router.post('/upload', upload.single('image'), (req, res) => {
  if (!req.file) {
    return res.status(400).json({ error: 'ไม่พบไฟล์' });
  }
  
  res.json({
    filename: req.file.filename,
    size: req.file.size,
    path: `/uploads/${req.file.filename}`
  });
});

module.exports = router;

// tests/integration/uploads.test.js
const request = require('supertest');
const app = require('../../src/app');
const path = require('path');
const fs = require('fs');

describe('File Upload Tests', () => {
  const uploadDir = path.join(__dirname, '../../uploads');
  const uploadedFiles = [];
  
  afterEach(() => {
    // ลบไฟล์ที่อัพโหลดทดสอบ
    uploadedFiles.forEach(file => {
      const filePath = path.join(uploadDir, file);
      if (fs.existsSync(filePath)) {
        fs.unlinkSync(filePath);
      }
    });
    uploadedFiles.length = 0;
  });
  
  test('ควรอัพโหลดรูปภาพสำเร็จ', async () => {
    const testImagePath = path.join(__dirname, '../fixtures/test-image.jpg');
    
    const res = await request(app)
      .post('/api/upload')
      .set('Authorization', `Bearer ${authToken}`)
      .attach('image', testImagePath)
      .expect(200);
    
    expect(res.body.filename).toBeDefined();
    expect(res.body.size).toBeGreaterThan(0);
    
    uploadedFiles.push(res.body.filename);
    
    // ตรวจสอบว่าไฟล์ถูกบันทึก
    const savedPath = path.join(uploadDir, res.body.filename);
    expect(fs.existsSync(savedPath)).toBe(true);
  });
  
  test('ควรส่ง 400 เมื่อไม่มีไฟล์', async () => {
    await request(app)
      .post('/api/upload')
      .set('Authorization', `Bearer ${authToken}`)
      .expect(400);
  });
  
  test('ควรส่ง 400 เมื่อไฟล์ไม่ใช่รูปภาพ', async () => {
    const testTextPath = path.join(__dirname, '../fixtures/test.txt');
    
    await request(app)
      .post('/api/upload')
      .set('Authorization', `Bearer ${authToken}`)
      .attach('image', testTextPath)
      .expect(400);
  });
  
  test('ควรส่ง 400 เมื่อไฟล์ใหญ่เกิน 5MB', async () => {
    // สร้างไฟล์ขนาดใหญ่
    const largePath = path.join(__dirname, '../fixtures/large-file.jpg');
    const largeBuffer = Buffer.alloc(6 * 1024 * 1024); // 6MB
    fs.writeFileSync(largePath, largeBuffer);
    
    try {
      await request(app)
        .post('/api/upload')
        .set('Authorization', `Bearer ${authToken}`)
        .attach('image', largePath)
        .expect(400);
    } finally {
      fs.unlinkSync(largePath);
    }
  });
});
```

---

## Step 1279: Testing Email Sending

```javascript
// Mock email service ใน integration tests

// tests/integration/auth.test.js
const nodemailer = require('nodemailer');

jest.mock('nodemailer');

describe('Auth Integration Tests with Email', () => {
  let mockSendMail;
  
  beforeAll(() => {
    mockSendMail = jest.fn().mockResolvedValue({ messageId: 'test-id' });
    nodemailer.createTransport.mockReturnValue({
      sendMail: mockSendMail
    });
  });
  
  afterEach(() => {
    mockSendMail.mockClear();
  });
  
  test('register ควรส่ง verification email', async () => {
    await request(app)
      .post('/api/auth/register')
      .send({
        name: 'Test User',
        email: 'newtest@example.com',
        password: 'StrongPass@123',
        confirmPassword: 'StrongPass@123'
      })
      .expect(201);
    
    expect(mockSendMail).toHaveBeenCalledTimes(1);
    expect(mockSendMail).toHaveBeenCalledWith(
      expect.objectContaining({
        to: 'newtest@example.com',
        subject: expect.stringContaining('ยืนยัน')
      })
    );
  });
  
  test('forgot-password ควรส่ง reset email', async () => {
    await request(app)
      .post('/api/auth/forgot-password')
      .send({ email: 'test@example.com' })
      .expect(200);
    
    expect(mockSendMail).toHaveBeenCalledTimes(1);
  });
  
  test('forgot-password ไม่ควรเปิดเผย user ที่ไม่มี', async () => {
    await request(app)
      .post('/api/auth/forgot-password')
      .send({ email: 'notexist@example.com' })
      .expect(200); // ยังส่ง 200 เสมอ
    
    // แต่ไม่ส่ง email
    expect(mockSendMail).not.toHaveBeenCalled();
  });
});
```

---

## Step 1280: Testing กับ Real Database (PostgreSQL + Prisma)

```javascript
// Setup สำหรับ PostgreSQL integration tests

// jest.setup.js สำหรับ Prisma
const { PrismaClient } = require('@prisma/client');
const { execSync } = require('child_process');

const prisma = new PrismaClient({
  datasources: {
    db: { url: process.env.TEST_DATABASE_URL }
  }
});

global.prisma = prisma;

beforeAll(async () => {
  // Reset database
  execSync('npx prisma migrate reset --force --skip-seed', {
    env: {
      ...process.env,
      DATABASE_URL: process.env.TEST_DATABASE_URL
    }
  });
});

afterAll(async () => {
  await prisma.$disconnect();
});

afterEach(async () => {
  // ล้างข้อมูลในลำดับที่ถูกต้อง (foreign key constraints)
  await prisma.$executeRaw`TRUNCATE TABLE "comments" CASCADE`;
  await prisma.$executeRaw`TRUNCATE TABLE "posts" CASCADE`;
  await prisma.$executeRaw`TRUNCATE TABLE "users" CASCADE`;
});

// tests/integration/prisma/users.test.js
const request = require('supertest');
const app = require('../../../src/app');

describe('Users API with Prisma', () => {
  test('ควร paginate users ได้', async () => {
    // สร้าง users จำนวนมาก
    const users = await Promise.all(
      Array.from({ length: 15 }, (_, i) =>
        prisma.user.create({
          data: {
            name: `User ${i}`,
            email: `user${i}@test.com`,
            password: 'hash'
          }
        })
      )
    );
    
    const res = await request(app)
      .get('/api/users?page=1&limit=10')
      .set('Authorization', `Bearer ${adminToken}`)
      .expect(200);
    
    expect(res.body.users).toHaveLength(10);
    expect(res.body.pagination.total).toBe(15);
    expect(res.body.pagination.totalPages).toBe(2);
  });
  
  test('ควร search users ได้', async () => {
    await prisma.user.createMany({
      data: [
        { name: 'Alice Johnson', email: 'alice@test.com', password: 'hash' },
        { name: 'Bob Smith', email: 'bob@test.com', password: 'hash' },
        { name: 'Alice Wong', email: 'alice2@test.com', password: 'hash' }
      ]
    });
    
    const res = await request(app)
      .get('/api/users?search=alice')
      .set('Authorization', `Bearer ${adminToken}`)
      .expect(200);
    
    expect(res.body.users).toHaveLength(2);
    expect(res.body.users.every(u => u.name.toLowerCase().includes('alice'))).toBe(true);
  });
});
```

---

## Step 1281: Test Isolation Strategies

```javascript
// กลยุทธ์การ isolate tests

// 1. Database per test suite
// แต่ละ test file ใช้ database ต่างกัน

// jest.config.js
// testEnvironment: './tests/setup/prismaEnvironment.js'

class PrismaTestEnvironment {
  async setup() {
    const databaseName = `test_${Math.random().toString(36).substring(7)}`;
    this.databaseUrl = `postgresql://test:test@localhost/${databaseName}`;
    
    // สร้าง database ใหม่
    await createDatabase(databaseName);
    
    // Run migrations
    process.env.DATABASE_URL = this.databaseUrl;
    execSync('npx prisma migrate deploy');
  }
  
  async teardown() {
    await dropDatabase(this.databaseUrl);
  }
}

// 2. Cleanup ด้วย transaction
// (เห็นแล้วใน step 1276)

// 3. Unique identifiers
async function createTestData() {
  const uniqueId = `test_${Date.now()}_${Math.random()}`;
  
  const user = await User.create({
    email: `${uniqueId}@example.com`,
    name: 'Test User'
  });
  
  return user;
}

// 4. Jest worker isolation
// jest.config.js
// maxWorkers: 1  // รัน test ทีละ file
// หรือ
// --runInBand  // รัน sequential

// 5. เก็บ references ของ created data
describe('isolated test suite', () => {
  const createdIds = [];
  
  afterEach(async () => {
    // ลบเฉพาะ data ที่สร้างใน suite นี้
    await User.deleteMany({ _id: { $in: createdIds } });
    createdIds.length = 0;
  });
  
  test('create user', async () => {
    const user = await User.create({ name: 'Test', email: 'test@test.com' });
    createdIds.push(user._id);
    
    expect(user.name).toBe('Test');
  });
});
```

---

## Step 1282: Parallel Test Execution

```javascript
// Jest รัน test files แบบ parallel โดย default
// แต่ tests ใน file เดียวกันรัน sequential

// jest.config.js
module.exports = {
  // จำนวน parallel workers
  maxWorkers: '50%', // ใช้ 50% ของ CPU cores
  // หรือ maxWorkers: 4
  
  // สำหรับ integration tests ที่ share database
  // ควรใช้ --runInBand หรือ maxWorkers: 1
};

// ปัญหาที่พบบ่อยกับ parallel tests:
// 1. Race conditions ใน database
// 2. Port conflicts (ถ้า start server)
// 3. File system conflicts

// แก้ปัญหา port conflicts
const getPort = require('get-port');

async function startTestServer() {
  const port = await getPort(); // หา port ที่ว่าง
  const server = app.listen(port);
  return { server, port };
}

// แก้ปัญหา database conflicts
// ใช้ separate database per worker
// หรือ unique data per test

// jest.config.js สำหรับ integration tests
const integrationConfig = {
  testMatch: ['**/integration/**/*.test.js'],
  maxWorkers: 1, // รัน sequential
  testTimeout: 30000
};
```

---

## Step 1283: CI/CD Integration

```javascript
// จัดการ tests ใน CI/CD pipeline

// .github/workflows/test.yml
/*
name: Tests

on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    
    services:
      mongodb:
        image: mongo:6
        ports:
          - 27017:27017
      
      postgres:
        image: postgres:14
        env:
          POSTGRES_PASSWORD: testpass
          POSTGRES_DB: testdb
        ports:
          - 5432:5432
        options: >-
          --health-cmd pg_isready
          --health-interval 10s
          --health-timeout 5s
          --health-retries 5
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run unit tests
        run: npm run test:unit
      
      - name: Run integration tests
        run: npm run test:integration
        env:
          MONGODB_URI: mongodb://localhost:27017/testdb
          DATABASE_URL: postgresql://postgres:testpass@localhost:5432/testdb
      
      - name: Upload coverage
        uses: codecov/codecov-action@v3
*/

// Scripts ใน package.json
/*
{
  "scripts": {
    "test": "jest",
    "test:unit": "jest tests/unit --coverage",
    "test:integration": "jest tests/integration --runInBand",
    "test:ci": "jest --ci --coverage --runInBand"
  }
}
*/

// ตั้งค่าสำหรับ CI environment
if (process.env.CI) {
  jest.setTimeout(30000); // CI ช้ากว่า local
}
```

---

## Step 1284: Testing Middleware

```javascript
// ทดสอบ Custom Middleware

// src/middleware/rateLimiter.js
const rateLimit = require('express-rate-limit');

function createRateLimiter(options = {}) {
  return rateLimit({
    windowMs: options.windowMs || 15 * 60 * 1000,
    max: options.max || 100,
    message: options.message || { error: 'Request มากเกินไป' },
    keyGenerator: (req) => req.ip || 'unknown',
    skip: (req) => process.env.NODE_ENV === 'test' && req.headers['x-skip-rate-limit']
  });
}

module.exports = { createRateLimiter };

// tests/integration/middleware/rateLimiter.test.js
const request = require('supertest');
const express = require('express');
const { createRateLimiter } = require('../../../src/middleware/rateLimiter');

describe('Rate Limiter Middleware', () => {
  let app;
  
  beforeEach(() => {
    app = express();
    
    const limiter = createRateLimiter({ max: 3, windowMs: 1000 });
    app.use(limiter);
    app.get('/test', (req, res) => res.json({ ok: true }));
  });
  
  test('ควร allow requests ภายใน limit', async () => {
    for (let i = 0; i < 3; i++) {
      await request(app)
        .get('/test')
        .expect(200);
    }
  });
  
  test('ควรปฏิเสธ requests ที่เกิน limit', async () => {
    // ส่ง 3 requests ที่อนุญาต
    for (let i = 0; i < 3; i++) {
      await request(app).get('/test');
    }
    
    // ส่งอีกครั้ง - ควร fail
    const res = await request(app)
      .get('/test')
      .expect(429);
    
    expect(res.body.error).toBeDefined();
  });
  
  test('ควรมี RateLimit headers', async () => {
    const res = await request(app)
      .get('/test')
      .expect(200);
    
    expect(res.headers['ratelimit-limit']).toBeDefined();
    expect(res.headers['ratelimit-remaining']).toBeDefined();
  });
});
```

---

## Step 1285: Testing Validation

```javascript
// ทดสอบ Input Validation

// src/middleware/validate.js
const { validationResult } = require('express-validator');

function validate(validations) {
  return async (req, res, next) => {
    await Promise.all(validations.map(v => v.run(req)));
    
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({
        error: 'Validation failed',
        details: errors.array()
      });
    }
    
    next();
  };
}

module.exports = { validate };

// tests/integration/validation/userValidation.test.js
const request = require('supertest');
const app = require('../../../src/app');

describe('User Validation', () => {
  describe('POST /api/users - Validation', () => {
    const validUser = {
      name: 'Alice Johnson',
      email: 'alice@example.com',
      password: 'SecurePass@123',
      confirmPassword: 'SecurePass@123'
    };
    
    test.each([
      ['name ว่าง', { ...validUser, name: '' }, 'name'],
      ['name สั้นเกิน', { ...validUser, name: 'A' }, 'name'],
      ['email ไม่ถูกต้อง', { ...validUser, email: 'notanemail' }, 'email'],
      ['password สั้นเกิน', { ...validUser, password: '123', confirmPassword: '123' }, 'password'],
      ['password ไม่ตรงกัน', { ...validUser, confirmPassword: 'DifferentPass@123' }, 'confirmPassword'],
    ])('ควรส่ง 400 เมื่อ %s', async (_, data, expectedField) => {
      const res = await request(app)
        .post('/api/users')
        .send(data)
        .expect(400);
      
      expect(res.body.details).toBeDefined();
      expect(res.body.details.some(e => e.path === expectedField)).toBe(true);
    });
    
    test('ควรผ่าน validation เมื่อข้อมูลถูกต้อง', async () => {
      await request(app)
        .post('/api/users')
        .send(validUser)
        .expect(201);
    });
  });
});
```

---

## Step 1286: Testing Pagination และ Sorting

```javascript
// ทดสอบ Pagination และ Sorting

describe('Pagination and Sorting', () => {
  beforeEach(async () => {
    // สร้าง users สำหรับทดสอบ
    await User.insertMany([
      { name: 'Charlie', email: 'charlie@test.com', createdAt: new Date('2024-01-01') },
      { name: 'Alice', email: 'alice@test.com', createdAt: new Date('2024-01-02') },
      { name: 'Bob', email: 'bob@test.com', createdAt: new Date('2024-01-03') },
      { name: 'Diana', email: 'diana@test.com', createdAt: new Date('2024-01-04') },
      { name: 'Eve', email: 'eve@test.com', createdAt: new Date('2024-01-05') },
    ]);
  });
  
  test('ควร paginate ได้ถูกต้อง', async () => {
    const page1 = await request(app)
      .get('/api/users?page=1&limit=2')
      .set('Authorization', `Bearer ${authToken}`)
      .expect(200);
    
    expect(page1.body.users).toHaveLength(2);
    expect(page1.body.pagination.page).toBe(1);
    expect(page1.body.pagination.total).toBe(5); // 5 + testUser
    
    const page2 = await request(app)
      .get('/api/users?page=2&limit=2')
      .set('Authorization', `Bearer ${authToken}`)
      .expect(200);
    
    expect(page2.body.users).toHaveLength(2);
    expect(page2.body.pagination.page).toBe(2);
    
    // ตรวจสอบว่าหน้า 1 และ 2 ไม่ซ้ำกัน
    const page1Ids = page1.body.users.map(u => u._id);
    const page2Ids = page2.body.users.map(u => u._id);
    const overlap = page1Ids.filter(id => page2Ids.includes(id));
    expect(overlap).toHaveLength(0);
  });
  
  test('ควร sort ตาม name ได้', async () => {
    const res = await request(app)
      .get('/api/users?sort=name&order=asc')
      .set('Authorization', `Bearer ${authToken}`)
      .expect(200);
    
    const names = res.body.users.map(u => u.name);
    const sortedNames = [...names].sort();
    expect(names).toEqual(sortedNames);
  });
  
  test('ควร sort ตาม createdAt desc', async () => {
    const res = await request(app)
      .get('/api/users?sort=createdAt&order=desc')
      .set('Authorization', `Bearer ${authToken}`)
      .expect(200);
    
    const dates = res.body.users.map(u => new Date(u.createdAt).getTime());
    for (let i = 0; i < dates.length - 1; i++) {
      expect(dates[i]).toBeGreaterThanOrEqual(dates[i + 1]);
    }
  });
  
  test('ควรมี hasMore flag', async () => {
    const res = await request(app)
      .get('/api/users?page=1&limit=3')
      .set('Authorization', `Bearer ${authToken}`)
      .expect(200);
    
    expect(res.body.pagination.hasNextPage).toBe(true);
    
    const lastPage = await request(app)
      .get(`/api/users?page=${res.body.pagination.totalPages}&limit=3`)
      .set('Authorization', `Bearer ${authToken}`)
      .expect(200);
    
    expect(lastPage.body.pagination.hasNextPage).toBe(false);
  });
});
```

---

## Step 1287: Integration Tests กับ WebSockets

```javascript
// ทดสอบ WebSocket connections

// ติดตั้ง: npm install --save-dev socket.io-client

const io = require('socket.io-client');
const http = require('http');
const app = require('../../src/app');
const setupSocket = require('../../src/socket');

describe('WebSocket Integration Tests', () => {
  let server;
  let clientSocket;
  let serverSocket;
  
  beforeAll(done => {
    server = http.createServer(app);
    setupSocket(server);
    server.listen(() => {
      const port = server.address().port;
      clientSocket = io(`http://localhost:${port}`, {
        auth: { token: authToken }
      });
      
      clientSocket.on('connect', done);
    });
  });
  
  afterAll(() => {
    server.close();
    clientSocket.close();
  });
  
  test('ควร connect สำเร็จ', done => {
    expect(clientSocket.connected).toBe(true);
    done();
  });
  
  test('ควรรับ notification', done => {
    clientSocket.on('notification', (data) => {
      expect(data.message).toBeDefined();
      done();
    });
    
    // Trigger notification
    request(app)
      .post('/api/notifications/send')
      .set('Authorization', `Bearer ${adminToken}`)
      .send({ userId: testUser._id, message: 'Test notification' });
  });
  
  test('ควร join room', done => {
    clientSocket.emit('join-room', { roomId: 'test-room' }, (response) => {
      expect(response.success).toBe(true);
      done();
    });
  });
});
```

---

## Step 1288: Performance Testing

```javascript
// ทดสอบ performance เบื้องต้น

describe('Performance Tests', () => {
  test('API ควรตอบสนองภายใน 200ms', async () => {
    const start = Date.now();
    
    await request(app)
      .get('/api/users')
      .set('Authorization', `Bearer ${authToken}`)
      .expect(200);
    
    const duration = Date.now() - start;
    expect(duration).toBeLessThan(200);
  });
  
  test('ควร handle concurrent requests', async () => {
    const concurrentRequests = 10;
    
    const start = Date.now();
    const requests = Array.from({ length: concurrentRequests }, () =>
      request(app)
        .get('/api/users')
        .set('Authorization', `Bearer ${authToken}`)
    );
    
    const responses = await Promise.all(requests);
    const duration = Date.now() - start;
    
    // ทุก requests ควรสำเร็จ
    responses.forEach(res => {
      expect(res.status).toBe(200);
    });
    
    console.log(`${concurrentRequests} concurrent requests ใช้เวลา ${duration}ms`);
    expect(duration).toBeLessThan(1000);
  });
  
  test('ควร paginate ได้เร็วแม้มีข้อมูลมาก', async () => {
    // สร้าง users จำนวนมาก
    await User.insertMany(
      Array.from({ length: 1000 }, (_, i) => ({
        name: `User ${i}`,
        email: `user${i}@perf.test`,
        password: 'hash'
      }))
    );
    
    const start = Date.now();
    
    await request(app)
      .get('/api/users?page=50&limit=20')
      .set('Authorization', `Bearer ${authToken}`)
      .expect(200);
    
    const duration = Date.now() - start;
    expect(duration).toBeLessThan(500);
  });
});
```

---

## Step 1289: End-to-End Test Flow

```javascript
// ทดสอบ complete user journey

describe('Complete User Journey', () => {
  test('Register -> Verify Email -> Login -> Create Post -> Logout', async () => {
    // 1. Register
    const registerRes = await request(app)
      .post('/api/auth/register')
      .send({
        name: 'Journey User',
        email: 'journey@example.com',
        password: 'JourneyPass@123',
        confirmPassword: 'JourneyPass@123'
      })
      .expect(201);
    
    expect(registerRes.body.user.isEmailVerified).toBe(false);
    
    // ดึง verification token จาก database
    const user = await User.findOne({ email: 'journey@example.com' });
    const verifyToken = user.emailVerificationToken;
    
    // 2. Verify Email
    await request(app)
      .get(`/api/auth/verify-email/${verifyToken}`)
      .expect(200);
    
    const verifiedUser = await User.findOne({ email: 'journey@example.com' });
    expect(verifiedUser.isEmailVerified).toBe(true);
    
    // 3. Login
    const loginRes = await request(app)
      .post('/api/auth/login')
      .send({
        email: 'journey@example.com',
        password: 'JourneyPass@123'
      })
      .expect(200);
    
    const token = loginRes.body.accessToken;
    expect(token).toBeDefined();
    
    // 4. Create Post
    const postRes = await request(app)
      .post('/api/posts')
      .set('Authorization', `Bearer ${token}`)
      .send({
        title: 'My First Post',
        content: 'Hello, World!'
      })
      .expect(201);
    
    expect(postRes.body.title).toBe('My First Post');
    
    // ตรวจสอบว่าโพสต์ถูกบันทึก
    const post = await Post.findById(postRes.body._id);
    expect(post.title).toBe('My First Post');
    
    // 5. Logout
    await request(app)
      .post('/api/auth/logout')
      .set('Authorization', `Bearer ${token}`)
      .expect(200);
    
    // 6. ลอง access protected route ด้วย token เดิม (หลัง logout)
    // ถ้า implement token blacklist จะ fail
    // ถ้าไม่มี จะต้องรอ token expire
  });
});
```

---

## Step 1290: Best Practices สรุป

```javascript
// Best Practices สำหรับ Integration Tests

// 1. Isolation - แต่ละ test ต้อง independent
// ✓ Cleanup หลังแต่ละ test
// ✓ ใช้ unique data (timestamps, random values)
// ✓ ไม่ depend on order ของ tests

// 2. Arrange-Act-Assert Pattern
test('ควรสร้าง post', async () => {
  // Arrange
  const user = await createUser();
  const token = createToken(user);
  const postData = { title: 'Test Post', content: 'Test' };
  
  // Act
  const res = await request(app)
    .post('/api/posts')
    .set('Authorization', `Bearer ${token}`)
    .send(postData);
  
  // Assert
  expect(res.status).toBe(201);
  expect(res.body.title).toBe(postData.title);
  
  const dbPost = await Post.findById(res.body._id);
  expect(dbPost).not.toBeNull();
});

// 3. Test ทั้ง happy path และ error cases
// Happy path: สิ่งที่ควรเกิดขึ้นปกติ
// Error cases: validation errors, not found, unauthorized, etc.

// 4. Assert response กับ database state
// ไม่ใช่แค่ HTTP status เพียงอย่างเดียว
test('delete ควรลบออกจาก database', async () => {
  const res = await request(app)
    .delete(`/api/users/${userId}`)
    .set('Authorization', `Bearer ${adminToken}`)
    .expect(200);
  
  // ตรวจสอบ database state ด้วย
  const user = await User.findById(userId);
  expect(user).toBeNull();
});

// 5. Test Authentication และ Authorization แยกกัน
// Authentication: มีหรือไม่มี token
// Authorization: มีสิทธิ์หรือไม่

// 6. Mock ที่จำเป็นเท่านั้น
// ✓ Mock: email sending, external APIs, payments
// ✗ ไม่ควร Mock: database (ใช้ test database จริง)

// 7. ตั้งชื่อ test ให้ชัดเจน
test('DELETE /api/users/:id ควรส่ง 403 เมื่อ user ลบบัญชีคนอื่น', async () => {
  // ...
});

// 8. รัน integration tests แยกจาก unit tests
// package.json
// "test:unit": "jest tests/unit --coverage"
// "test:integration": "jest tests/integration --runInBand"

// 9. ตั้ง timeout ที่เหมาะสม
jest.setTimeout(30000); // 30 วินาที สำหรับ integration tests

// 10. Log ข้อมูลเพิ่มเติมเมื่อ test fail
test('complex test', async () => {
  const res = await request(app).get('/api/complex');
  
  if (res.status !== 200) {
    console.error('Test failed:', {
      status: res.status,
      body: res.body,
      headers: res.headers
    });
  }
  
  expect(res.status).toBe(200);
});
```

---

## แบบฝึกหัด

### ระดับเริ่มต้น
1. สร้าง integration tests สำหรับ GET /api/users ด้วย Supertest
2. ทดสอบ POST /api/auth/register ทั้ง success และ error cases
3. เขียน cleanup code ที่ถูกต้องสำหรับ test database

### ระดับกลาง
4. สร้าง test factory สำหรับ User, Post, Comment
5. ทดสอบ authentication flow ครบวงจร
6. เขียน tests สำหรับ file upload

### ระดับสูง
7. สร้าง integration test suite ครบสำหรับ Blog API (CRUD + Auth + Search)
8. implement database isolation ด้วย MongoDB Memory Server
9. เขียน tests สำหรับ WebSocket connections
10. สร้าง CI/CD pipeline configuration สำหรับ automated testing

---

*Part 65 จบแล้ว - จบหน้าที่ 1271-1290 ของ JavaScript Course*
