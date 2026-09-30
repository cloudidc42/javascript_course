# Part 61: Authentication และ Authorization (Steps 1191-1210)

## บทนำ

Authentication และ Authorization เป็นส่วนสำคัญที่สุดของการพัฒนาเว็บแอปพลิเคชันที่ต้องจัดการกับข้อมูลผู้ใช้ บทนี้จะสอนวิธีการสร้างระบบความปลอดภัยที่มั่นคงตั้งแต่การจัดเก็บรหัสผ่านไปจนถึงการจัดการสิทธิ์การเข้าถึง

---

## Step 1191: Authentication vs Authorization

**Authentication (การตรวจสอบตัวตน)** คือกระบวนการยืนยันว่า "คุณเป็นใคร"
**Authorization (การอนุญาต)** คือกระบวนการตรวจสอบว่า "คุณมีสิทธิ์ทำอะไรได้บ้าง"

```javascript
// ตัวอย่างแนวคิด
// Authentication: ตรวจสอบว่าผู้ใช้คือใคร
function authenticate(username, password) {
  // ตรวจสอบว่า username/password ถูกต้อง
  const user = findUserByUsername(username);
  if (!user) return null;
  const isValid = bcrypt.compareSync(password, user.passwordHash);
  return isValid ? user : null;
}

// Authorization: ตรวจสอบว่าผู้ใช้มีสิทธิ์ทำสิ่งที่ต้องการหรือไม่
function authorize(user, resource, action) {
  // ตรวจสอบว่า user มีสิทธิ์ทำ action กับ resource หรือไม่
  return user.permissions.includes(`${resource}:${action}`);
}

// การใช้งาน
const user = authenticate('john@example.com', 'password123');
if (user) {
  console.log('ยืนยันตัวตนสำเร็จ:', user.name);
  
  if (authorize(user, 'posts', 'delete')) {
    console.log('มีสิทธิ์ลบโพสต์');
  } else {
    console.log('ไม่มีสิทธิ์ลบโพสต์');
  }
}
```

### ความแตกต่างที่สำคัญ

```javascript
// Authentication Flow
// 1. รับ credentials จากผู้ใช้
// 2. ตรวจสอบกับฐานข้อมูล
// 3. สร้าง session/token
// 4. ส่งกลับไปให้ผู้ใช้

// Authorization Flow
// 1. รับ token/session จากผู้ใช้
// 2. ตรวจสอบว่า token ถูกต้อง
// 3. ดึงข้อมูลสิทธิ์ของผู้ใช้
// 4. ตรวจสอบว่ามีสิทธิ์ทำสิ่งที่ขอหรือไม่

// ตัวอย่างกับ Express
const express = require('express');
const app = express();

// Authentication Middleware
function requireAuth(req, res, next) {
  const token = req.headers.authorization?.split(' ')[1];
  if (!token) {
    return res.status(401).json({ error: 'กรุณาเข้าสู่ระบบ' });
  }
  
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    req.user = decoded;
    next();
  } catch (err) {
    return res.status(401).json({ error: 'Token ไม่ถูกต้อง' });
  }
}

// Authorization Middleware
function requireRole(role) {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({ error: 'กรุณาเข้าสู่ระบบ' });
    }
    
    if (req.user.role !== role) {
      return res.status(403).json({ error: 'ไม่มีสิทธิ์เข้าถึง' });
    }
    
    next();
  };
}

// การใช้งาน
app.get('/admin', requireAuth, requireRole('admin'), (req, res) => {
  res.json({ message: 'ยินดีต้อนรับสู่หน้า Admin' });
});
```

---

## Step 1192: Password Hashing กับ Bcrypt

**ห้ามเก็บรหัสผ่านเป็น plaintext** ควรใช้ bcrypt ในการ hash รหัสผ่าน

```javascript
// ติดตั้ง: npm install bcryptjs
const bcrypt = require('bcryptjs');

// การ hash รหัสผ่าน
async function hashPassword(password) {
  // saltRounds คือจำนวนรอบในการ hash (ยิ่งมาก ยิ่งปลอดภัย แต่ยิ่งช้า)
  const saltRounds = 12;
  const hashedPassword = await bcrypt.hash(password, saltRounds);
  return hashedPassword;
}

// การตรวจสอบรหัสผ่าน
async function verifyPassword(plainPassword, hashedPassword) {
  const isMatch = await bcrypt.compare(plainPassword, hashedPassword);
  return isMatch;
}

// ตัวอย่างการใช้งาน
async function registerUser(email, password) {
  // ตรวจสอบว่ารหัสผ่านแข็งแรงพอ
  if (password.length < 8) {
    throw new Error('รหัสผ่านต้องมีอย่างน้อย 8 ตัวอักษร');
  }
  
  const hashedPassword = await hashPassword(password);
  
  // บันทึกลงฐานข้อมูล
  const user = await User.create({
    email,
    password: hashedPassword
  });
  
  return user;
}

async function loginUser(email, password) {
  const user = await User.findOne({ email });
  if (!user) {
    throw new Error('ไม่พบผู้ใช้');
  }
  
  const isValid = await verifyPassword(password, user.password);
  if (!isValid) {
    throw new Error('รหัสผ่านไม่ถูกต้อง');
  }
  
  return user;
}
```

### Salt คืออะไร?

```javascript
// Salt เป็น random data ที่เพิ่มเข้าไปก่อน hash
// เพื่อป้องกัน rainbow table attacks

const bcrypt = require('bcryptjs');

// แสดงให้เห็นว่า salt ทำงานอย่างไร
async function demonstrateSalt() {
  const password = 'mypassword123';
  
  // Hash เดียวกันแต่ได้ผลต่างกัน เพราะ salt ต่างกัน
  const hash1 = await bcrypt.hash(password, 10);
  const hash2 = await bcrypt.hash(password, 10);
  
  console.log('Hash 1:', hash1);
  console.log('Hash 2:', hash2);
  console.log('เหมือนกันไหม:', hash1 === hash2); // false
  
  // แต่ตรวจสอบได้ถูกต้องทั้งคู่
  const valid1 = await bcrypt.compare(password, hash1);
  const valid2 = await bcrypt.compare(password, hash2);
  console.log('Hash1 ถูกต้อง:', valid1); // true
  console.log('Hash2 ถูกต้อง:', valid2); // true
}

demonstrateSalt();
```

### การเลือก saltRounds ที่เหมาะสม

```javascript
const bcrypt = require('bcryptjs');

async function benchmarkRounds() {
  const password = 'testpassword';
  
  for (let rounds = 8; rounds <= 14; rounds++) {
    const start = Date.now();
    await bcrypt.hash(password, rounds);
    const end = Date.now();
    console.log(`Rounds ${rounds}: ${end - start}ms`);
  }
}

// ผลลัพธ์โดยประมาณ:
// Rounds 8: 50ms
// Rounds 10: 100ms
// Rounds 12: 400ms
// Rounds 14: 1600ms

// แนะนำ: 12 สำหรับ production, 10 สำหรับ development
```

---

## Step 1193: JWT (JSON Web Tokens) - โครงสร้าง

JWT ประกอบด้วย 3 ส่วน แยกด้วย `.`

```javascript
// โครงสร้างของ JWT
// header.payload.signature

// ตัวอย่าง JWT
// eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiOiIxMjMiLCJlbWFpbCI6InRlc3RAZXhhbXBsZS5jb20iLCJpYXQiOjE2MDAwMDAwMDB9.signature

// Header
const header = {
  alg: 'HS256', // Algorithm ที่ใช้ในการ sign
  typ: 'JWT'    // Token type
};

// Payload (Claims)
const payload = {
  // Registered Claims (มาตรฐาน)
  sub: '1234567890',      // Subject (ผู้ใช้)
  iss: 'myapp.com',       // Issuer (ผู้สร้าง token)
  aud: 'myapp.com',       // Audience (ผู้รับ)
  exp: 1700000000,        // Expiration time (Unix timestamp)
  iat: 1699996400,        // Issued at (เวลาสร้าง)
  nbf: 1699996400,        // Not before (ใช้ได้ตั้งแต่เมื่อไร)
  jti: 'unique-id-123',   // JWT ID (รหัสเฉพาะ)
  
  // Custom Claims (กำหนดเอง)
  userId: '507f1f77bcf86cd799439011',
  email: 'user@example.com',
  role: 'admin'
};

// Signature
// HMACSHA256(base64UrlEncode(header) + "." + base64UrlEncode(payload), secret)
```

### การ Decode JWT (ไม่ verify)

```javascript
// Decode โดยไม่ verify (อย่าใช้ใน production โดยไม่ verify!)
function decodeJWT(token) {
  const parts = token.split('.');
  if (parts.length !== 3) {
    throw new Error('JWT ไม่ถูกต้อง');
  }
  
  const header = JSON.parse(Buffer.from(parts[0], 'base64url').toString());
  const payload = JSON.parse(Buffer.from(parts[1], 'base64url').toString());
  
  return { header, payload };
}

const token = 'eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiOiIxMjMifQ.sig';
const decoded = decodeJWT(token);
console.log('Header:', decoded.header);
console.log('Payload:', decoded.payload);
```

---

## Step 1194: JWT - การสร้างและตรวจสอบ Token

```javascript
// ติดตั้ง: npm install jsonwebtoken
const jwt = require('jsonwebtoken');

const JWT_SECRET = process.env.JWT_SECRET || 'your-secret-key-keep-it-safe';

// การสร้าง Token
function createToken(user) {
  const payload = {
    userId: user._id,
    email: user.email,
    role: user.role
  };
  
  const options = {
    expiresIn: '24h',         // หมดอายุใน 24 ชั่วโมง
    issuer: 'myapp.com',      // ผู้สร้าง
    audience: 'myapp-users',  // ผู้รับ
  };
  
  return jwt.sign(payload, JWT_SECRET, options);
}

// การตรวจสอบ Token
function verifyToken(token) {
  try {
    const decoded = jwt.verify(token, JWT_SECRET, {
      issuer: 'myapp.com',
      audience: 'myapp-users'
    });
    return decoded;
  } catch (err) {
    if (err.name === 'TokenExpiredError') {
      throw new Error('Token หมดอายุ กรุณาเข้าสู่ระบบใหม่');
    }
    if (err.name === 'JsonWebTokenError') {
      throw new Error('Token ไม่ถูกต้อง');
    }
    throw err;
  }
}

// ตัวอย่างการใช้งาน
const user = {
  _id: '507f1f77bcf86cd799439011',
  email: 'john@example.com',
  role: 'user'
};

const token = createToken(user);
console.log('Token:', token);

try {
  const decoded = verifyToken(token);
  console.log('Decoded:', decoded);
} catch (err) {
  console.error('Error:', err.message);
}
```

### JWT กับ Algorithms ต่างๆ

```javascript
const jwt = require('jsonwebtoken');
const fs = require('fs');

// HS256 (HMAC SHA-256) - ใช้ symmetric key
const hs256Token = jwt.sign({ userId: '123' }, 'secret-key', {
  algorithm: 'HS256',
  expiresIn: '1h'
});

// RS256 (RSA SHA-256) - ใช้ asymmetric key (private/public key)
// สร้าง key pair: openssl genrsa -out private.pem 2048
// openssl rsa -in private.pem -pubout -out public.pem

// const privateKey = fs.readFileSync('private.pem');
// const publicKey = fs.readFileSync('public.pem');

// const rs256Token = jwt.sign({ userId: '123' }, privateKey, {
//   algorithm: 'RS256',
//   expiresIn: '1h'
// });

// const decoded = jwt.verify(rs256Token, publicKey);

// ES256 (ECDSA SHA-256) - ใช้ elliptic curve
// มีขนาดเล็กกว่า RS256 แต่ปลอดภัยเท่ากัน
```

---

## Step 1195: Token Expiry และ Refresh Token Pattern

```javascript
const jwt = require('jsonwebtoken');

const ACCESS_TOKEN_SECRET = process.env.ACCESS_TOKEN_SECRET || 'access-secret';
const REFRESH_TOKEN_SECRET = process.env.REFRESH_TOKEN_SECRET || 'refresh-secret';

// สร้าง Access Token (อายุสั้น)
function createAccessToken(user) {
  return jwt.sign(
    {
      userId: user._id,
      email: user.email,
      role: user.role,
      type: 'access'
    },
    ACCESS_TOKEN_SECRET,
    { expiresIn: '15m' } // หมดอายุใน 15 นาที
  );
}

// สร้าง Refresh Token (อายุยาว)
function createRefreshToken(user) {
  return jwt.sign(
    {
      userId: user._id,
      type: 'refresh'
    },
    REFRESH_TOKEN_SECRET,
    { expiresIn: '7d' } // หมดอายุใน 7 วัน
  );
}

// เก็บ Refresh Token ในฐานข้อมูล
const refreshTokenStore = new Map(); // ในจริงควรเก็บใน database

async function login(email, password) {
  // ตรวจสอบ credentials
  const user = await User.findOne({ email });
  if (!user || !await bcrypt.compare(password, user.password)) {
    throw new Error('email หรือ password ไม่ถูกต้อง');
  }
  
  // สร้าง tokens
  const accessToken = createAccessToken(user);
  const refreshToken = createRefreshToken(user);
  
  // บันทึก refresh token
  refreshTokenStore.set(user._id.toString(), refreshToken);
  
  return { accessToken, refreshToken };
}

async function refreshAccessToken(refreshToken) {
  try {
    // ตรวจสอบ refresh token
    const decoded = jwt.verify(refreshToken, REFRESH_TOKEN_SECRET);
    
    if (decoded.type !== 'refresh') {
      throw new Error('Token ชนิดไม่ถูกต้อง');
    }
    
    // ตรวจสอบว่า refresh token อยู่ในฐานข้อมูล
    const storedToken = refreshTokenStore.get(decoded.userId);
    if (storedToken !== refreshToken) {
      throw new Error('Refresh token ไม่ถูกต้อง');
    }
    
    // ดึงข้อมูล user
    const user = await User.findById(decoded.userId);
    if (!user) {
      throw new Error('ไม่พบผู้ใช้');
    }
    
    // สร้าง access token ใหม่
    const newAccessToken = createAccessToken(user);
    
    return { accessToken: newAccessToken };
  } catch (err) {
    throw new Error('Refresh token ไม่ถูกต้องหรือหมดอายุ');
  }
}

async function logout(userId) {
  // ลบ refresh token
  refreshTokenStore.delete(userId.toString());
}
```

---

## Step 1196: Session-based Auth vs Token-based Auth

```javascript
// Session-based Authentication
const express = require('express');
const session = require('express-session');
const MongoStore = require('connect-mongo');

const app = express();

// ตั้งค่า Session
app.use(session({
  secret: process.env.SESSION_SECRET,
  resave: false,
  saveUninitialized: false,
  store: MongoStore.create({
    mongoUrl: process.env.MONGODB_URI
  }),
  cookie: {
    secure: process.env.NODE_ENV === 'production', // HTTPS only
    httpOnly: true, // ป้องกัน XSS
    maxAge: 24 * 60 * 60 * 1000 // 24 ชั่วโมง
  }
}));

// Login ด้วย Session
app.post('/login', async (req, res) => {
  const { email, password } = req.body;
  
  try {
    const user = await User.findOne({ email });
    if (!user || !await bcrypt.compare(password, user.password)) {
      return res.status(401).json({ error: 'credentials ไม่ถูกต้อง' });
    }
    
    // บันทึกข้อมูลใน session
    req.session.userId = user._id;
    req.session.role = user.role;
    
    res.json({ message: 'เข้าสู่ระบบสำเร็จ' });
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

// Middleware ตรวจสอบ Session
function requireSession(req, res, next) {
  if (!req.session.userId) {
    return res.status(401).json({ error: 'กรุณาเข้าสู่ระบบ' });
  }
  next();
}

// Logout ด้วย Session
app.post('/logout', (req, res) => {
  req.session.destroy((err) => {
    if (err) {
      return res.status(500).json({ error: 'Logout ไม่สำเร็จ' });
    }
    res.clearCookie('connect.sid');
    res.json({ message: 'ออกจากระบบสำเร็จ' });
  });
});
```

### เปรียบเทียบ Session vs JWT

```javascript
// Session-based
// ข้อดี:
// - ยกเลิกได้ทันที
// - ข้อมูล user อยู่ server-side
// - ปลอดภัยกว่าในบางกรณี
// ข้อเสีย:
// - ต้องเก็บ session ใน server/database
// - ไม่เหมาะกับ microservices
// - Scale ยาก

// Token-based (JWT)
// ข้อดี:
// - Stateless ไม่ต้องเก็บ state ใน server
// - เหมาะกับ microservices
// - Scale ง่าย
// - ทำงานได้ข้าม domain
// ข้อเสีย:
// - ยกเลิกยาก (ต้องใช้ blacklist)
// - ข้อมูลใน payload มองเห็นได้
// - Token ขนาดใหญ่กว่า session cookie

// การเลือกใช้
// - Web app แบบ traditional: Session
// - SPA, Mobile app, API: JWT
// - Microservices: JWT
```

---

## Step 1197: OAuth 2.0 Overview

```javascript
// OAuth 2.0 Flow ทั่วไป
// 1. User คลิก "Login with Google"
// 2. App redirect ไปยัง Google Authorization Server
// 3. User grant permission
// 4. Google redirect กลับมาพร้อม Authorization Code
// 5. App แลก Authorization Code เอา Access Token
// 6. App ใช้ Access Token เข้าถึง Google API

// Grant Types ของ OAuth 2.0
// 1. Authorization Code Flow (แนะนำสำหรับ web app)
// 2. Implicit Flow (deprecated)
// 3. Client Credentials Flow (สำหรับ machine-to-machine)
// 4. Resource Owner Password Credentials (ไม่แนะนำ)
// 5. Device Code Flow (สำหรับ IoT)

// ตัวอย่าง Authorization Code URL
function getGoogleAuthUrl() {
  const params = new URLSearchParams({
    client_id: process.env.GOOGLE_CLIENT_ID,
    redirect_uri: 'http://localhost:3000/auth/google/callback',
    response_type: 'code',
    scope: 'openid email profile',
    state: generateRandomState(), // ป้องกัน CSRF
    access_type: 'offline', // รับ refresh token
    prompt: 'consent'
  });
  
  return `https://accounts.google.com/o/oauth2/v2/auth?${params}`;
}

function generateRandomState() {
  return require('crypto').randomBytes(32).toString('hex');
}
```

---

## Step 1198: Google OAuth Flow

```javascript
// ติดตั้ง: npm install googleapis
const { google } = require('googleapis');
const express = require('express');
const app = express();

const oauth2Client = new google.auth.OAuth2(
  process.env.GOOGLE_CLIENT_ID,
  process.env.GOOGLE_CLIENT_SECRET,
  'http://localhost:3000/auth/google/callback'
);

// เก็บ state สำหรับป้องกัน CSRF
const stateStore = new Map();

// Step 1: Redirect ไป Google
app.get('/auth/google', (req, res) => {
  const state = require('crypto').randomBytes(32).toString('hex');
  stateStore.set(state, { createdAt: Date.now() });
  
  const authUrl = oauth2Client.generateAuthUrl({
    access_type: 'offline',
    scope: ['openid', 'email', 'profile'],
    state
  });
  
  res.redirect(authUrl);
});

// Step 2: รับ Callback จาก Google
app.get('/auth/google/callback', async (req, res) => {
  const { code, state, error } = req.query;
  
  if (error) {
    return res.redirect('/login?error=google_denied');
  }
  
  // ตรวจสอบ state
  if (!stateStore.has(state)) {
    return res.redirect('/login?error=invalid_state');
  }
  stateStore.delete(state);
  
  try {
    // แลก code เอา tokens
    const { tokens } = await oauth2Client.getToken(code);
    oauth2Client.setCredentials(tokens);
    
    // ดึงข้อมูล user
    const oauth2 = google.oauth2({ version: 'v2', auth: oauth2Client });
    const { data } = await oauth2.userinfo.get();
    
    // หา user ในฐานข้อมูล หรือสร้างใหม่
    let user = await User.findOne({ googleId: data.id });
    if (!user) {
      user = await User.create({
        googleId: data.id,
        email: data.email,
        name: data.name,
        picture: data.picture,
        isEmailVerified: true
      });
    }
    
    // สร้าง JWT
    const token = createAccessToken(user);
    
    // Redirect ไปหน้า frontend พร้อม token
    res.redirect(`http://localhost:3000/auth/success?token=${token}`);
  } catch (err) {
    console.error('Google auth error:', err);
    res.redirect('/login?error=auth_failed');
  }
});
```

---

## Step 1199: Passport.js Overview

```javascript
// ติดตั้ง: npm install passport passport-local passport-jwt

const passport = require('passport');
const LocalStrategy = require('passport-local').Strategy;
const JwtStrategy = require('passport-jwt').Strategy;
const ExtractJwt = require('passport-jwt').ExtractJwt;

// Local Strategy (username/password)
passport.use(new LocalStrategy(
  {
    usernameField: 'email',
    passwordField: 'password'
  },
  async (email, password, done) => {
    try {
      const user = await User.findOne({ email });
      
      if (!user) {
        return done(null, false, { message: 'ไม่พบผู้ใช้' });
      }
      
      const isValid = await bcrypt.compare(password, user.password);
      if (!isValid) {
        return done(null, false, { message: 'รหัสผ่านไม่ถูกต้อง' });
      }
      
      return done(null, user);
    } catch (err) {
      return done(err);
    }
  }
));

// JWT Strategy
passport.use(new JwtStrategy(
  {
    jwtFromRequest: ExtractJwt.fromAuthHeaderAsBearerToken(),
    secretOrKey: process.env.JWT_SECRET
  },
  async (payload, done) => {
    try {
      const user = await User.findById(payload.userId);
      
      if (!user) {
        return done(null, false);
      }
      
      return done(null, user);
    } catch (err) {
      return done(err);
    }
  }
));

// Serialize/Deserialize สำหรับ Session
passport.serializeUser((user, done) => {
  done(null, user._id);
});

passport.deserializeUser(async (id, done) => {
  try {
    const user = await User.findById(id);
    done(null, user);
  } catch (err) {
    done(err);
  }
});

// การใช้งานใน Express
const express = require('express');
const app = express();

app.use(passport.initialize());

// Login
app.post('/login',
  passport.authenticate('local', { session: false }),
  (req, res) => {
    const token = createAccessToken(req.user);
    res.json({ token });
  }
);

// Protected Route
app.get('/protected',
  passport.authenticate('jwt', { session: false }),
  (req, res) => {
    res.json({ user: req.user });
  }
);
```

---

## Step 1200: Role-based Access Control (RBAC)

```javascript
// RBAC Model
// User -> Role -> Permissions

const ROLES = {
  ADMIN: 'admin',
  MODERATOR: 'moderator',
  USER: 'user',
  GUEST: 'guest'
};

const PERMISSIONS = {
  // User permissions
  READ_USER: 'read:user',
  UPDATE_USER: 'update:user',
  DELETE_USER: 'delete:user',
  
  // Post permissions
  CREATE_POST: 'create:post',
  READ_POST: 'read:post',
  UPDATE_POST: 'update:post',
  DELETE_POST: 'delete:post',
  
  // Admin permissions
  MANAGE_USERS: 'manage:users',
  MANAGE_SYSTEM: 'manage:system'
};

const ROLE_PERMISSIONS = {
  [ROLES.ADMIN]: [
    PERMISSIONS.READ_USER,
    PERMISSIONS.UPDATE_USER,
    PERMISSIONS.DELETE_USER,
    PERMISSIONS.CREATE_POST,
    PERMISSIONS.READ_POST,
    PERMISSIONS.UPDATE_POST,
    PERMISSIONS.DELETE_POST,
    PERMISSIONS.MANAGE_USERS,
    PERMISSIONS.MANAGE_SYSTEM
  ],
  [ROLES.MODERATOR]: [
    PERMISSIONS.READ_USER,
    PERMISSIONS.READ_POST,
    PERMISSIONS.UPDATE_POST,
    PERMISSIONS.DELETE_POST
  ],
  [ROLES.USER]: [
    PERMISSIONS.READ_USER,
    PERMISSIONS.CREATE_POST,
    PERMISSIONS.READ_POST,
    PERMISSIONS.UPDATE_POST // แก้ไขเฉพาะโพสต์ตัวเอง
  ],
  [ROLES.GUEST]: [
    PERMISSIONS.READ_POST
  ]
};

// ตรวจสอบสิทธิ์
function hasPermission(userRole, permission) {
  const permissions = ROLE_PERMISSIONS[userRole] || [];
  return permissions.includes(permission);
}

// Middleware สำหรับตรวจสอบสิทธิ์
function requirePermission(permission) {
  return (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({ error: 'กรุณาเข้าสู่ระบบ' });
    }
    
    if (!hasPermission(req.user.role, permission)) {
      return res.status(403).json({
        error: 'ไม่มีสิทธิ์เข้าถึง',
        required: permission
      });
    }
    
    next();
  };
}

// การใช้งาน
app.delete('/posts/:id',
  requireAuth,
  requirePermission(PERMISSIONS.DELETE_POST),
  async (req, res) => {
    // ลบโพสต์
    await Post.findByIdAndDelete(req.params.id);
    res.json({ message: 'ลบโพสต์สำเร็จ' });
  }
);
```

---

## Step 1201: Permission-based Access Control

```javascript
// Permission-based ยืดหยุ่นกว่า RBAC
// กำหนดสิทธิ์ให้ user แต่ละคนโดยตรง

const mongoose = require('mongoose');

const userSchema = new mongoose.Schema({
  email: String,
  password: String,
  role: {
    type: String,
    enum: ['admin', 'moderator', 'user'],
    default: 'user'
  },
  // สิทธิ์เพิ่มเติมนอกเหนือจาก role
  additionalPermissions: [String],
  // สิทธิ์ที่ถูกถอนจาก role
  revokedPermissions: [String]
});

// Method สำหรับตรวจสอบสิทธิ์
userSchema.methods.hasPermission = function(permission) {
  // ตรวจสอบสิทธิ์ที่ถูกถอน
  if (this.revokedPermissions.includes(permission)) {
    return false;
  }
  
  // ตรวจสอบสิทธิ์เพิ่มเติม
  if (this.additionalPermissions.includes(permission)) {
    return true;
  }
  
  // ตรวจสอบสิทธิ์จาก role
  return hasPermission(this.role, permission);
};

const User = mongoose.model('User', userSchema);

// ตัวอย่างการใช้งาน
async function checkAndGrantPermission() {
  const user = await User.findById('userId');
  
  // ให้สิทธิ์เพิ่มเติม
  await User.findByIdAndUpdate('userId', {
    $addToSet: { additionalPermissions: 'delete:post' }
  });
  
  // ถอนสิทธิ์
  await User.findByIdAndUpdate('userId', {
    $addToSet: { revokedPermissions: 'update:user' }
  });
  
  // ตรวจสอบสิทธิ์
  console.log('มีสิทธิ์ลบโพสต์:', user.hasPermission('delete:post'));
}
```

---

## Step 1202: API Key Authentication

```javascript
const crypto = require('crypto');
const mongoose = require('mongoose');

// Schema สำหรับ API Key
const apiKeySchema = new mongoose.Schema({
  key: {
    type: String,
    required: true,
    unique: true
  },
  keyHash: {
    type: String,
    required: true
  },
  userId: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',
    required: true
  },
  name: String,
  permissions: [String],
  lastUsed: Date,
  expiresAt: Date,
  isActive: {
    type: Boolean,
    default: true
  }
});

const ApiKey = mongoose.model('ApiKey', apiKeySchema);

// สร้าง API Key ใหม่
async function generateApiKey(userId, name, permissions = []) {
  // สร้าง key แบบสุ่ม
  const rawKey = `ak_${crypto.randomBytes(32).toString('hex')}`;
  
  // Hash key สำหรับเก็บในฐานข้อมูล
  const keyHash = crypto
    .createHash('sha256')
    .update(rawKey)
    .digest('hex');
  
  const apiKey = await ApiKey.create({
    key: rawKey.substring(0, 8) + '...', // เก็บแค่ prefix
    keyHash,
    userId,
    name,
    permissions
  });
  
  // ส่ง raw key กลับไปให้ user (จะไม่แสดงอีก!)
  return {
    id: apiKey._id,
    key: rawKey, // แสดงครั้งเดียว!
    name
  };
}

// ตรวจสอบ API Key
async function validateApiKey(rawKey) {
  const keyHash = crypto
    .createHash('sha256')
    .update(rawKey)
    .digest('hex');
  
  const apiKey = await ApiKey.findOne({
    keyHash,
    isActive: true
  }).populate('userId');
  
  if (!apiKey) {
    throw new Error('API key ไม่ถูกต้อง');
  }
  
  // ตรวจสอบวันหมดอายุ
  if (apiKey.expiresAt && apiKey.expiresAt < new Date()) {
    throw new Error('API key หมดอายุ');
  }
  
  // อัพเดต lastUsed
  await ApiKey.findByIdAndUpdate(apiKey._id, { lastUsed: new Date() });
  
  return apiKey;
}

// Middleware สำหรับ API Key
async function apiKeyAuth(req, res, next) {
  const apiKey = req.headers['x-api-key'];
  
  if (!apiKey) {
    return res.status(401).json({ error: 'กรุณาใส่ API key' });
  }
  
  try {
    const keyDoc = await validateApiKey(apiKey);
    req.user = keyDoc.userId;
    req.apiKey = keyDoc;
    next();
  } catch (err) {
    res.status(401).json({ error: err.message });
  }
}
```

---

## Step 1203: Two-Factor Authentication (2FA)

```javascript
// ติดตั้ง: npm install speakeasy qrcode
const speakeasy = require('speakeasy');
const QRCode = require('qrcode');

// Step 1: สร้าง Secret สำหรับ 2FA
async function setup2FA(userId) {
  const secret = speakeasy.generateSecret({
    name: 'MyApp',
    issuer: 'MyApp Inc.'
  });
  
  // บันทึก secret ลงฐานข้อมูล (ยังไม่ enable)
  await User.findByIdAndUpdate(userId, {
    twoFactorSecret: secret.base32,
    twoFactorEnabled: false
  });
  
  // สร้าง QR Code
  const qrCodeUrl = await QRCode.toDataURL(secret.otpauth_url);
  
  return {
    secret: secret.base32,
    qrCode: qrCodeUrl
  };
}

// Step 2: ยืนยัน 2FA Setup
async function verify2FASetup(userId, token) {
  const user = await User.findById(userId);
  
  const isValid = speakeasy.totp.verify({
    secret: user.twoFactorSecret,
    encoding: 'base32',
    token,
    window: 2 // อนุญาต +-2 time steps
  });
  
  if (!isValid) {
    throw new Error('รหัส 2FA ไม่ถูกต้อง');
  }
  
  // Enable 2FA
  await User.findByIdAndUpdate(userId, { twoFactorEnabled: true });
  
  // สร้าง backup codes
  const backupCodes = generateBackupCodes();
  await User.findByIdAndUpdate(userId, { backupCodes });
  
  return { backupCodes };
}

function generateBackupCodes(count = 8) {
  return Array.from({ length: count }, () => 
    require('crypto').randomBytes(4).toString('hex')
  );
}

// Step 3: ตรวจสอบ 2FA ตอน Login
async function verifyLogin2FA(userId, token) {
  const user = await User.findById(userId);
  
  if (!user.twoFactorEnabled) {
    return true; // ไม่ได้เปิดใช้ 2FA
  }
  
  // ตรวจสอบ TOTP
  const isValidTotp = speakeasy.totp.verify({
    secret: user.twoFactorSecret,
    encoding: 'base32',
    token,
    window: 2
  });
  
  if (isValidTotp) {
    return true;
  }
  
  // ตรวจสอบ backup code
  if (user.backupCodes.includes(token)) {
    // ลบ backup code ที่ใช้แล้ว
    await User.findByIdAndUpdate(userId, {
      $pull: { backupCodes: token }
    });
    return true;
  }
  
  return false;
}
```

---

## Step 1204: Secure Password Storage

```javascript
// แนวทางการเก็บรหัสผ่านอย่างปลอดภัย

const bcrypt = require('bcryptjs');
const zxcvbn = require('zxcvbn'); // ตรวจสอบความแข็งแรงของรหัสผ่าน

// 1. ตรวจสอบความแข็งแรง
function validatePasswordStrength(password) {
  const result = zxcvbn(password);
  
  // score: 0 (weak) - 4 (strong)
  if (result.score < 3) {
    const suggestions = result.feedback.suggestions.join(' ');
    throw new Error(`รหัสผ่านไม่แข็งแรงพอ: ${suggestions}`);
  }
  
  return true;
}

// 2. ตรวจสอบ requirements พื้นฐาน
function validatePasswordRequirements(password) {
  const errors = [];
  
  if (password.length < 8) {
    errors.push('ต้องมีอย่างน้อย 8 ตัวอักษร');
  }
  if (!/[A-Z]/.test(password)) {
    errors.push('ต้องมีตัวพิมพ์ใหญ่อย่างน้อย 1 ตัว');
  }
  if (!/[a-z]/.test(password)) {
    errors.push('ต้องมีตัวพิมพ์เล็กอย่างน้อย 1 ตัว');
  }
  if (!/\d/.test(password)) {
    errors.push('ต้องมีตัวเลขอย่างน้อย 1 ตัว');
  }
  if (!/[!@#$%^&*]/.test(password)) {
    errors.push('ต้องมีอักขระพิเศษอย่างน้อย 1 ตัว');
  }
  
  if (errors.length > 0) {
    throw new Error(errors.join(', '));
  }
  
  return true;
}

// 3. Hash รหัสผ่าน
async function hashPassword(password) {
  validatePasswordRequirements(password);
  
  const saltRounds = 12;
  return bcrypt.hash(password, saltRounds);
}

// 4. Password History (ป้องกันใช้รหัสผ่านเดิมซ้ำ)
async function updatePassword(userId, newPassword, oldPasswordCount = 5) {
  const user = await User.findById(userId).select('+password +passwordHistory');
  
  // ตรวจสอบว่าไม่ใช่รหัสผ่านเดิม
  for (const oldHash of user.passwordHistory || []) {
    const isReused = await bcrypt.compare(newPassword, oldHash);
    if (isReused) {
      throw new Error(`ไม่สามารถใช้รหัสผ่าน ${oldPasswordCount} รหัสล่าสุดซ้ำได้`);
    }
  }
  
  const newHash = await hashPassword(newPassword);
  
  // อัพเดต password history
  const history = user.passwordHistory || [];
  history.unshift(user.password);
  if (history.length > oldPasswordCount) {
    history.splice(oldPasswordCount);
  }
  
  await User.findByIdAndUpdate(userId, {
    password: newHash,
    passwordHistory: history,
    passwordChangedAt: new Date()
  });
}
```

---

## Step 1205: Auth Middleware Implementation

```javascript
const jwt = require('jsonwebtoken');

// Middleware หลายแบบ
// 1. Required Auth - ต้องเข้าสู่ระบบ
const requireAuth = async (req, res, next) => {
  try {
    const token = extractToken(req);
    
    if (!token) {
      return res.status(401).json({ error: 'กรุณาเข้าสู่ระบบ' });
    }
    
    const decoded = jwt.verify(token, process.env.JWT_SECRET);
    
    // ดึงข้อมูล user จากฐานข้อมูล
    const user = await User.findById(decoded.userId)
      .select('-password');
    
    if (!user) {
      return res.status(401).json({ error: 'ไม่พบผู้ใช้' });
    }
    
    // ตรวจสอบว่ารหัสผ่านไม่ถูกเปลี่ยนหลัง token ถูกสร้าง
    if (user.passwordChangedAt) {
      const changedAt = Math.floor(user.passwordChangedAt.getTime() / 1000);
      if (decoded.iat < changedAt) {
        return res.status(401).json({ error: 'รหัสผ่านถูกเปลี่ยน กรุณาเข้าสู่ระบบใหม่' });
      }
    }
    
    req.user = user;
    next();
  } catch (err) {
    if (err.name === 'TokenExpiredError') {
      return res.status(401).json({ error: 'Session หมดอายุ กรุณาเข้าสู่ระบบใหม่', code: 'TOKEN_EXPIRED' });
    }
    return res.status(401).json({ error: 'Token ไม่ถูกต้อง' });
  }
};

// 2. Optional Auth - ไม่ต้องเข้าสู่ระบบก็ได้
const optionalAuth = async (req, res, next) => {
  try {
    const token = extractToken(req);
    
    if (token) {
      const decoded = jwt.verify(token, process.env.JWT_SECRET);
      req.user = await User.findById(decoded.userId).select('-password');
    }
    
    next();
  } catch (err) {
    // ไม่หยุด request แม้ token ผิด
    next();
  }
};

// Helper function ดึง token จาก request
function extractToken(req) {
  // ดูจาก Authorization header
  if (req.headers.authorization?.startsWith('Bearer ')) {
    return req.headers.authorization.split(' ')[1];
  }
  
  // ดูจาก cookie
  if (req.cookies?.token) {
    return req.cookies.token;
  }
  
  // ดูจาก query parameter (ไม่แนะนำ แต่บางกรณีใช้ได้)
  if (req.query.token) {
    return req.query.token;
  }
  
  return null;
}

// 3. Rate Limiting สำหรับ Auth
const rateLimit = require('express-rate-limit');

const loginLimiter = rateLimit({
  windowMs: 15 * 60 * 1000, // 15 นาที
  max: 5, // 5 ครั้งต่อ window
  message: {
    error: 'ลองเข้าสู่ระบบบ่อยเกินไป กรุณารออีก 15 นาที'
  },
  standardHeaders: true,
  legacyHeaders: false
});
```

---

## Step 1206: Complete Auth System - Register

```javascript
// ระบบ Auth ครบวงจร
const express = require('express');
const bcrypt = require('bcryptjs');
const jwt = require('jsonwebtoken');
const mongoose = require('mongoose');
const crypto = require('crypto');

const router = express.Router();

// User Schema
const userSchema = new mongoose.Schema({
  name: {
    type: String,
    required: [true, 'กรุณาใส่ชื่อ'],
    trim: true,
    maxlength: [100, 'ชื่อยาวเกินไป']
  },
  email: {
    type: String,
    required: [true, 'กรุณาใส่ email'],
    unique: true,
    lowercase: true,
    match: [/^\w+([.-]?\w+)*@\w+([.-]?\w+)*(\.\w{2,3})+$/, 'Email ไม่ถูกต้อง']
  },
  password: {
    type: String,
    required: [true, 'กรุณาใส่รหัสผ่าน'],
    minlength: [8, 'รหัสผ่านต้องมีอย่างน้อย 8 ตัวอักษร'],
    select: false
  },
  role: {
    type: String,
    enum: ['user', 'moderator', 'admin'],
    default: 'user'
  },
  isEmailVerified: {
    type: Boolean,
    default: false
  },
  emailVerificationToken: String,
  emailVerificationExpires: Date,
  passwordResetToken: String,
  passwordResetExpires: Date,
  refreshTokens: [String],
  isActive: {
    type: Boolean,
    default: true
  },
  createdAt: {
    type: Date,
    default: Date.now
  }
});

const User = mongoose.model('User', userSchema);

// Register
router.post('/register', async (req, res) => {
  try {
    const { name, email, password, confirmPassword } = req.body;
    
    // Validation
    if (!name || !email || !password) {
      return res.status(400).json({ error: 'กรุณากรอกข้อมูลให้ครบ' });
    }
    
    if (password !== confirmPassword) {
      return res.status(400).json({ error: 'รหัสผ่านไม่ตรงกัน' });
    }
    
    // ตรวจสอบ email ซ้ำ
    const existingUser = await User.findOne({ email });
    if (existingUser) {
      return res.status(400).json({ error: 'Email นี้ถูกใช้แล้ว' });
    }
    
    // Hash รหัสผ่าน
    const hashedPassword = await bcrypt.hash(password, 12);
    
    // สร้าง email verification token
    const verificationToken = crypto.randomBytes(32).toString('hex');
    const verificationExpires = new Date(Date.now() + 24 * 60 * 60 * 1000);
    
    // สร้าง user
    const user = await User.create({
      name,
      email,
      password: hashedPassword,
      emailVerificationToken: verificationToken,
      emailVerificationExpires: verificationExpires
    });
    
    // ส่ง verification email
    // await sendVerificationEmail(user.email, verificationToken);
    
    // ส่ง response (ไม่รวม password)
    const userResponse = user.toObject();
    delete userResponse.password;
    
    res.status(201).json({
      message: 'สมัครสมาชิกสำเร็จ กรุณายืนยัน email',
      user: {
        id: user._id,
        name: user.name,
        email: user.email
      }
    });
  } catch (err) {
    console.error('Register error:', err);
    res.status(500).json({ error: 'เกิดข้อผิดพลาด กรุณาลองใหม่' });
  }
});
```

---

## Step 1207: Complete Auth System - Login

```javascript
// Login
router.post('/login', loginLimiter, async (req, res) => {
  try {
    const { email, password } = req.body;
    
    if (!email || !password) {
      return res.status(400).json({ error: 'กรุณากรอก email และ password' });
    }
    
    // หา user พร้อม password
    const user = await User.findOne({ email }).select('+password');
    
    // ตรวจสอบ user และ password (ใช้เวลาเท่ากันเสมอ เพื่อป้องกัน timing attack)
    if (!user || !await bcrypt.compare(password, user.password)) {
      return res.status(401).json({ error: 'Email หรือรหัสผ่านไม่ถูกต้อง' });
    }
    
    // ตรวจสอบว่า account ยัง active
    if (!user.isActive) {
      return res.status(401).json({ error: 'บัญชีถูกระงับ กรุณาติดต่อ admin' });
    }
    
    // สร้าง tokens
    const accessToken = jwt.sign(
      { userId: user._id, role: user.role },
      process.env.JWT_SECRET,
      { expiresIn: '15m' }
    );
    
    const refreshToken = jwt.sign(
      { userId: user._id, type: 'refresh' },
      process.env.JWT_REFRESH_SECRET,
      { expiresIn: '7d' }
    );
    
    // บันทึก refresh token
    user.refreshTokens.push(refreshToken);
    await user.save();
    
    // ตั้ง cookie สำหรับ refresh token
    res.cookie('refreshToken', refreshToken, {
      httpOnly: true,
      secure: process.env.NODE_ENV === 'production',
      sameSite: 'strict',
      maxAge: 7 * 24 * 60 * 60 * 1000 // 7 วัน
    });
    
    res.json({
      message: 'เข้าสู่ระบบสำเร็จ',
      accessToken,
      user: {
        id: user._id,
        name: user.name,
        email: user.email,
        role: user.role
      }
    });
  } catch (err) {
    console.error('Login error:', err);
    res.status(500).json({ error: 'เกิดข้อผิดพลาด' });
  }
});

// Refresh Token
router.post('/refresh', async (req, res) => {
  const refreshToken = req.cookies.refreshToken;
  
  if (!refreshToken) {
    return res.status(401).json({ error: 'ไม่มี refresh token' });
  }
  
  try {
    const decoded = jwt.verify(refreshToken, process.env.JWT_REFRESH_SECRET);
    
    const user = await User.findById(decoded.userId);
    if (!user || !user.refreshTokens.includes(refreshToken)) {
      return res.status(401).json({ error: 'Refresh token ไม่ถูกต้อง' });
    }
    
    // สร้าง access token ใหม่
    const newAccessToken = jwt.sign(
      { userId: user._id, role: user.role },
      process.env.JWT_SECRET,
      { expiresIn: '15m' }
    );
    
    res.json({ accessToken: newAccessToken });
  } catch (err) {
    res.status(401).json({ error: 'Refresh token ไม่ถูกต้องหรือหมดอายุ' });
  }
});
```

---

## Step 1208: Complete Auth System - Protected Routes

```javascript
// Protected Routes
router.get('/me', requireAuth, async (req, res) => {
  res.json({
    user: {
      id: req.user._id,
      name: req.user.name,
      email: req.user.email,
      role: req.user.role,
      isEmailVerified: req.user.isEmailVerified,
      createdAt: req.user.createdAt
    }
  });
});

// Change Password
router.put('/change-password', requireAuth, async (req, res) => {
  try {
    const { currentPassword, newPassword, confirmNewPassword } = req.body;
    
    if (newPassword !== confirmNewPassword) {
      return res.status(400).json({ error: 'รหัสผ่านใหม่ไม่ตรงกัน' });
    }
    
    const user = await User.findById(req.user._id).select('+password');
    
    // ตรวจสอบรหัสผ่านปัจจุบัน
    const isValid = await bcrypt.compare(currentPassword, user.password);
    if (!isValid) {
      return res.status(401).json({ error: 'รหัสผ่านปัจจุบันไม่ถูกต้อง' });
    }
    
    // Hash รหัสผ่านใหม่
    const hashedPassword = await bcrypt.hash(newPassword, 12);
    
    // อัพเดต
    user.password = hashedPassword;
    user.refreshTokens = []; // Invalidate ทุก session
    await user.save();
    
    res.json({ message: 'เปลี่ยนรหัสผ่านสำเร็จ กรุณาเข้าสู่ระบบใหม่' });
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

// Forgot Password
router.post('/forgot-password', async (req, res) => {
  try {
    const { email } = req.body;
    
    const user = await User.findOne({ email });
    
    // ส่ง response เหมือนกันไม่ว่าจะพบ user หรือไม่ (ป้องกัน enumeration attack)
    if (!user) {
      return res.json({ message: 'ถ้า email นี้มีในระบบ จะได้รับ email สำหรับ reset รหัสผ่าน' });
    }
    
    // สร้าง reset token
    const resetToken = crypto.randomBytes(32).toString('hex');
    const hashedToken = crypto.createHash('sha256').update(resetToken).digest('hex');
    
    user.passwordResetToken = hashedToken;
    user.passwordResetExpires = new Date(Date.now() + 10 * 60 * 1000); // 10 นาที
    await user.save();
    
    // ส่ง email
    // await sendPasswordResetEmail(user.email, resetToken);
    
    res.json({ message: 'ถ้า email นี้มีในระบบ จะได้รับ email สำหรับ reset รหัสผ่าน' });
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

// Reset Password
router.post('/reset-password/:token', async (req, res) => {
  try {
    const { password, confirmPassword } = req.body;
    
    if (password !== confirmPassword) {
      return res.status(400).json({ error: 'รหัสผ่านไม่ตรงกัน' });
    }
    
    // Hash token ที่ได้รับ
    const hashedToken = crypto
      .createHash('sha256')
      .update(req.params.token)
      .digest('hex');
    
    // หา user ที่มี reset token ที่ยังไม่หมดอายุ
    const user = await User.findOne({
      passwordResetToken: hashedToken,
      passwordResetExpires: { $gt: new Date() }
    });
    
    if (!user) {
      return res.status(400).json({ error: 'Token ไม่ถูกต้องหรือหมดอายุ' });
    }
    
    // อัพเดตรหัสผ่าน
    user.password = await bcrypt.hash(password, 12);
    user.passwordResetToken = undefined;
    user.passwordResetExpires = undefined;
    user.refreshTokens = []; // Invalidate ทุก session
    await user.save();
    
    res.json({ message: 'เปลี่ยนรหัสผ่านสำเร็จ กรุณาเข้าสู่ระบบ' });
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});
```

---

## Step 1209: Email Verification

```javascript
// Verify Email
router.get('/verify-email/:token', async (req, res) => {
  try {
    const user = await User.findOne({
      emailVerificationToken: req.params.token,
      emailVerificationExpires: { $gt: new Date() }
    });
    
    if (!user) {
      return res.status(400).json({ error: 'Token ไม่ถูกต้องหรือหมดอายุ' });
    }
    
    user.isEmailVerified = true;
    user.emailVerificationToken = undefined;
    user.emailVerificationExpires = undefined;
    await user.save();
    
    res.json({ message: 'ยืนยัน email สำเร็จ' });
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

// Resend Verification Email
router.post('/resend-verification', requireAuth, async (req, res) => {
  try {
    if (req.user.isEmailVerified) {
      return res.status(400).json({ error: 'Email ยืนยันแล้ว' });
    }
    
    const verificationToken = crypto.randomBytes(32).toString('hex');
    const verificationExpires = new Date(Date.now() + 24 * 60 * 60 * 1000);
    
    await User.findByIdAndUpdate(req.user._id, {
      emailVerificationToken: verificationToken,
      emailVerificationExpires: verificationExpires
    });
    
    // await sendVerificationEmail(req.user.email, verificationToken);
    
    res.json({ message: 'ส่ง verification email ใหม่แล้ว' });
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

// Logout
router.post('/logout', requireAuth, async (req, res) => {
  try {
    const refreshToken = req.cookies.refreshToken;
    
    if (refreshToken) {
      // ลบ refresh token ออกจาก database
      await User.findByIdAndUpdate(req.user._id, {
        $pull: { refreshTokens: refreshToken }
      });
    }
    
    // Clear cookie
    res.clearCookie('refreshToken');
    
    res.json({ message: 'ออกจากระบบสำเร็จ' });
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

// Logout ทุก device
router.post('/logout-all', requireAuth, async (req, res) => {
  try {
    await User.findByIdAndUpdate(req.user._id, {
      refreshTokens: []
    });
    
    res.clearCookie('refreshToken');
    res.json({ message: 'ออกจากระบบทุก device แล้ว' });
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});
```

---

## Step 1210: Security Best Practices

```javascript
// สรุป Security Best Practices

// 1. HTTPS only
// - ใช้ SSL/TLS certificate
// - Redirect HTTP -> HTTPS
// - ใช้ HSTS header

// 2. Input Validation
const { body, validationResult } = require('express-validator');

const registerValidation = [
  body('email').isEmail().normalizeEmail(),
  body('password')
    .isLength({ min: 8 })
    .matches(/^(?=.*[A-Z])(?=.*[a-z])(?=.*\d)(?=.*[@$!%*?&])/),
  body('name').trim().notEmpty().isLength({ max: 100 })
];

// 3. Security Headers
const helmet = require('helmet');
app.use(helmet());

// 4. CORS
const cors = require('cors');
app.use(cors({
  origin: process.env.FRONTEND_URL,
  credentials: true
}));

// 5. XSS Protection
// Helmet ช่วยได้, และ sanitize input

// 6. SQL/NoSQL Injection Prevention
// ใช้ parameterized queries / Mongoose sanitization
const mongoSanitize = require('express-mongo-sanitize');
app.use(mongoSanitize());

// 7. Rate Limiting
const rateLimit = require('express-rate-limit');
app.use('/api/', rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 100
}));

// 8. Secure Cookie
res.cookie('refreshToken', token, {
  httpOnly: true,
  secure: true,
  sameSite: 'strict',
  maxAge: 7 * 24 * 60 * 60 * 1000
});

// 9. Password Storage
// - ใช้ bcrypt กับ saltRounds >= 12
// - ห้าม MD5, SHA1
// - ไม่เก็บ plaintext

// 10. Logging
// - Log auth attempts (สำเร็จ/ล้มเหลว)
// - ไม่ log passwords หรือ tokens
// - ส่ง alerts เมื่อมีพฤติกรรมน่าสงสัย

// ตัวอย่าง complete app setup
function setupSecurity(app) {
  app.use(helmet());
  app.use(cors({ origin: process.env.FRONTEND_URL, credentials: true }));
  app.use(mongoSanitize());
  app.use(express.json({ limit: '10kb' })); // จำกัด request body size
  
  // Global rate limiter
  app.use('/api/', rateLimit({
    windowMs: 15 * 60 * 1000,
    max: 100,
    message: { error: 'Request มากเกินไป กรุณารอสักครู่' }
  }));
}
```

---

## แบบฝึกหัด

### ระดับเริ่มต้น
1. สร้างฟังก์ชัน `hashAndVerifyPassword` ที่ hash รหัสผ่านและตรวจสอบ
2. สร้าง JWT token ที่มีข้อมูล userId, email, role และหมดอายุใน 1 ชั่วโมง
3. เขียน middleware `requireAuth` สำหรับ Express

### ระดับกลาง
4. สร้างระบบ Register/Login/Logout ครบวงจรพร้อม JWT
5. implement Refresh Token pattern
6. เพิ่ม Rate limiting ใน login route

### ระดับสูง
7. สร้างระบบ RBAC ที่รองรับ role และ permission
8. implement Google OAuth โดยใช้ passport-google-oauth20
9. สร้างระบบ 2FA ด้วย TOTP
10. เขียน unit tests สำหรับ auth middleware

---

*Part 61 จบแล้ว ต่อไปเป็น Part 62: MongoDB กับ JavaScript*
