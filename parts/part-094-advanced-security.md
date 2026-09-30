# Part 94: Advanced Security (Steps 1851-1870)

## ความปลอดภัยขั้นสูง - Threat Modeling และ Cryptography ใน JavaScript

ในส่วนนี้เราจะเรียนรู้เกี่ยวกับการรักษาความปลอดภัยขั้นสูงสำหรับ web applications ตั้งแต่ threat modeling, การโจมตีแบบต่างๆ ไปจนถึงการใช้ cryptography อย่างถูกต้อง

---

## Step 1851: Threat Modeling for Web Apps

Threat Modeling คือกระบวนการวิเคราะห์ security threats อย่างเป็นระบบ เพื่อหาจุดอ่อนก่อนที่ผู้โจมตีจะหาเจอ

```javascript
// STRIDE Threat Model Framework
// S - Spoofing (การปลอมตัวตน)
// T - Tampering (การแก้ไขข้อมูล)
// R - Repudiation (การปฏิเสธการกระทำ)
// I - Information Disclosure (การรั่วไหลข้อมูล)
// D - Denial of Service (การทำให้บริการหยุด)
// E - Elevation of Privilege (การขอสิทธิ์เพิ่ม)

const threatModel = {
  assets: [
    'User credentials',
    'Personal data (PII)',
    'Payment information',
    'API keys',
    'Business logic',
  ],
  
  entryPoints: [
    'HTTP endpoints',
    'WebSocket connections',
    'File uploads',
    'Third-party integrations',
    'Admin interfaces',
  ],
  
  threats: {
    authentication: [
      { type: 'S', name: 'Credential stuffing', mitigation: 'Rate limiting, MFA' },
      { type: 'S', name: 'Session hijacking', mitigation: 'Secure cookies, HTTPS' },
      { type: 'E', name: 'Privilege escalation', mitigation: 'RBAC, authorization checks' },
    ],
    dataProtection: [
      { type: 'I', name: 'SQL injection', mitigation: 'Parameterized queries' },
      { type: 'I', name: 'Path traversal', mitigation: 'Input validation, sandboxing' },
      { type: 'T', name: 'CSRF', mitigation: 'CSRF tokens, SameSite cookies' },
    ],
    availability: [
      { type: 'D', name: 'DDoS', mitigation: 'Rate limiting, CDN' },
      { type: 'D', name: 'ReDoS', mitigation: 'Regex review, timeout' },
    ],
  },
};

// DREAD Risk Scoring
function calculateRisk(threat) {
  // Damage potential (0-10)
  // Reproducibility (0-10)
  // Exploitability (0-10)
  // Affected users (0-10)
  // Discoverability (0-10)
  
  const { damage, reproducibility, exploitability, affectedUsers, discoverability } = threat;
  const score = (damage + reproducibility + exploitability + affectedUsers + discoverability) / 5;
  
  return {
    score,
    severity: score >= 7 ? 'HIGH' : score >= 4 ? 'MEDIUM' : 'LOW',
  };
}
```

---

## Step 1852: Advanced XSS - DOM-based XSS

DOM-based XSS เกิดขึ้นเมื่อ JavaScript บน client-side อ่านข้อมูลจาก attacker-controlled source และ write ลงใน DOM

```javascript
// DOM-based XSS Sinks (แหล่งที่เป็นอันตราย)
// document.write()
// innerHTML
// outerHTML
// insertAdjacentHTML()
// eval()
// setTimeout(string)
// setInterval(string)
// location.href
// element.src
// element.action

// DOM-based XSS Sources (แหล่งข้อมูลอันตราย)
// location.href
// location.search
// location.hash
// document.URL
// document.referrer
// window.name
// postMessage data

// ตัวอย่างการโจมตี
// URL: https://example.com/page?name=<img src=x onerror=alert(1)>
// ❌ โค้ดที่มีช่องโหว่
function vulnerableCode() {
  const name = new URLSearchParams(location.search).get('name');
  document.getElementById('greeting').innerHTML = `Hello, ${name}!`;
  // หรือ
  document.write(`<h1>Welcome ${name}</h1>`);
}

// ✅ แก้ไขด้วย textContent
function safeCode() {
  const name = new URLSearchParams(location.search).get('name');
  const el = document.getElementById('greeting');
  el.textContent = `Hello, ${name}!`; // ไม่ interpret HTML
}

// ✅ ใช้ DOMPurify สำหรับ HTML ที่จำเป็นต้องแสดง
// npm install dompurify
const DOMPurify = require('dompurify');

function safeWithHTML(htmlContent) {
  const clean = DOMPurify.sanitize(htmlContent, {
    ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'p', 'br'],
    ALLOWED_ATTR: [],
  });
  document.getElementById('content').innerHTML = clean;
}

// ตรวจสอบ URL
function safeRedirect(url) {
  // ❌ ผิด - เปิดโอกาส javascript: protocol
  location.href = url;
  
  // ✅ ถูก - validate URL
  try {
    const parsed = new URL(url);
    if (parsed.protocol !== 'https:' && parsed.protocol !== 'http:') {
      throw new Error('Invalid protocol');
    }
    if (!ALLOWED_DOMAINS.includes(parsed.hostname)) {
      throw new Error('Domain not allowed');
    }
    location.href = url;
  } catch {
    console.error('Invalid redirect URL');
  }
}

// Content Security Policy (CSP) เพื่อป้องกัน XSS
const cspMiddleware = (req, res, next) => {
  const nonce = crypto.randomBytes(16).toString('base64');
  res.locals.cspNonce = nonce;
  
  res.setHeader('Content-Security-Policy', [
    `default-src 'self'`,
    `script-src 'self' 'nonce-${nonce}'`,
    `style-src 'self' 'nonce-${nonce}'`,
    `img-src 'self' data: https:`,
    `font-src 'self'`,
    `connect-src 'self' https://api.example.com`,
    `frame-ancestors 'none'`,
    `base-uri 'self'`,
    `form-action 'self'`,
  ].join('; '));
  
  next();
};
```

---

## Step 1853: Prototype Pollution Attacks and Prevention

Prototype Pollution คือการที่ผู้โจมตีสามารถเปลี่ยนแปลง `Object.prototype` ทำให้กระทบทุก object ในระบบ

```javascript
// ตัวอย่างการโจมตี Prototype Pollution

// ❌ โค้ดที่มีช่องโหว่
function merge(target, source) {
  for (const key of Object.keys(source)) {
    if (typeof source[key] === 'object') {
      target[key] = merge(target[key] || {}, source[key]);
    } else {
      target[key] = source[key];
    }
  }
  return target;
}

// ผู้โจมตีส่งข้อมูลนี้
const maliciousPayload = JSON.parse('{"__proto__": {"isAdmin": true}}');
merge({}, maliciousPayload);

// ตอนนี้ทุก object มี isAdmin = true!
const user = {};
console.log(user.isAdmin); // true  <- DANGER!

// หรือ
const maliciousPayload2 = JSON.parse('{"constructor": {"prototype": {"isAdmin": true}}}');

// ✅ วิธีป้องกัน

// 1. ใช้ Object.create(null) สำหรับ plain objects
const safeObj = Object.create(null);
safeObj.name = 'safe';
console.log(safeObj.__proto__); // undefined

// 2. ตรวจสอบ dangerous keys
function safeMerge(target, source) {
  const DANGEROUS_KEYS = ['__proto__', 'constructor', 'prototype'];
  
  for (const key of Object.keys(source)) {
    if (DANGEROUS_KEYS.includes(key)) {
      console.warn(`Potential prototype pollution: ${key}`);
      continue;
    }
    
    if (typeof source[key] === 'object' && source[key] !== null) {
      if (!Object.prototype.hasOwnProperty.call(target, key)) {
        target[key] = {};
      }
      safeMerge(target[key], source[key]);
    } else {
      target[key] = source[key];
    }
  }
  
  return target;
}

// 3. Freeze Object.prototype
// (ใช้ใน development สำหรับตรวจจับ)
Object.freeze(Object.prototype);

// 4. ใช้ JSON schema validation ก่อน process
const Ajv = require('ajv');
const ajv = new Ajv({ strict: true });

const schema = {
  type: 'object',
  properties: {
    name: { type: 'string' },
    settings: { type: 'object' },
  },
  additionalProperties: false,
};

function safeProcess(input) {
  const valid = ajv.validate(schema, input);
  if (!valid) throw new Error('Invalid input: ' + ajv.errorsText());
  // safe to process
}

// 5. npm audit - ตรวจสอบ packages ที่มีช่องโหว่
// npm audit
// npm audit fix

// ใช้ lodash ที่ patch แล้ว
const _ = require('lodash');
const safeResult = _.merge({}, maliciousPayload); // lodash ป้องกัน prototype pollution
```

---

## Step 1854: ReDoS (Regular Expression Denial of Service)

```javascript
// ReDoS เกิดจาก "evil" regex ที่ใช้เวลา exponential

// ❌ Vulnerable regex (catastrophic backtracking)
const EVIL_REGEX = /^(a+)+$/;
// Input ที่ทำให้ hang: 'aaaaaaaaaaaaaaaaaaaaaaaa!'
// เวลาเพิ่มแบบ exponential ตามความยาว!

// ทดสอบ
console.time('test');
EVIL_REGEX.test('aaaaaaaaaaaaaaaaaaaaaaaaaab');
console.timeEnd('test'); // อาจใช้เวลาหลายวินาทีหรือหลายนาที!

// Patterns ที่เป็นอันตราย
const vulnerablePatterns = [
  /^(a+)+$/,           // Grouping with quantifiers
  /^([a-zA-Z]+)*$/,    // Repeated alternation
  /^(\w+\s?)*$/,       // Overlapping possibilities
  /(a|aa)+/,           // Nested quantifiers
];

// ✅ วิธีป้องกัน

// 1. ใช้ regex ที่ simple และ predictable
// ❌ ผิด
const BAD_EMAIL_REGEX = /^([a-zA-Z0-9._%-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,6})*$/;

// ✅ ถูก
const GOOD_EMAIL_REGEX = /^[^\s@]+@[^\s@]+\.[^\s@]{2,}$/;

// 2. ตั้ง timeout สำหรับ regex
function safeTest(pattern, input, timeoutMs = 100) {
  return new Promise((resolve, reject) => {
    const timeout = setTimeout(() => {
      reject(new Error('Regex timeout - possible ReDoS'));
    }, timeoutMs);
    
    try {
      const result = pattern.test(input);
      clearTimeout(timeout);
      resolve(result);
    } catch (error) {
      clearTimeout(timeout);
      reject(error);
    }
  });
}

// 3. ใช้ re2 library (Google's RE2 - linear time)
// npm install re2
const RE2 = require('re2');

const safeRegex = new RE2(/^(a+)+$/);
// RE2 ใช้เวลา O(n) เสมอ ไม่มี catastrophic backtracking

// 4. ตรวจสอบ regex ด้วย safe-regex
// npm install safe-regex
const safeCheck = require('safe-regex');

console.log(safeCheck(/^(a+)+$/));          // false - unsafe!
console.log(safeCheck(/^[a-z]+@[a-z]+\.com$/)); // true - safe

// 5. Input length limits
function validateInput(input) {
  if (typeof input !== 'string') return false;
  if (input.length > 1000) {
    throw new Error('Input too long');
  }
  return SAFE_PATTERN.test(input);
}

// ตัวอย่าง safe regex patterns
const safePatterns = {
  email: /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/,
  url: /^https?:\/\/[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}(\/[^\s]*)?$/,
  phoneNumber: /^[+]?[\d\s\-().]{7,15}$/,
  zipCode: /^\d{5}(-\d{4})?$/,
  uuid: /^[0-9a-f]{8}-[0-9a-f]{4}-4[0-9a-f]{3}-[89ab][0-9a-f]{3}-[0-9a-f]{12}$/i,
};
```

---

## Step 1855: Server-Side Request Forgery (SSRF)

```javascript
// SSRF เกิดเมื่อ server ทำ HTTP request ตาม user input
// ผู้โจมตีสามารถเข้าถึง internal services ได้

// ❌ โค้ดที่มีช่องโหว่
app.post('/fetch-url', async (req, res) => {
  const { url } = req.body;
  const response = await fetch(url);  // SSRF!
  // ผู้โจมตีส่ง: http://169.254.169.254/latest/meta-data/ (AWS metadata)
  // หรือ: http://localhost:6379/ (Redis)
  // หรือ: file:///etc/passwd
  res.json(await response.json());
});

// ✅ วิธีป้องกัน

const dns = require('dns').promises;
const net = require('net');

// ตรวจสอบ IP ranges ที่อันตราย
function isPrivateIP(ip) {
  // Private IPv4 ranges
  const privateRanges = [
    /^127\./,                    // Loopback
    /^10\./,                     // Class A private
    /^172\.(1[6-9]|2\d|3[01])\./, // Class B private
    /^192\.168\./,               // Class C private
    /^169\.254\./,               // Link-local (AWS metadata)
    /^::1$/,                     // IPv6 loopback
    /^fc00:/,                    // IPv6 private
    /^fe80:/,                    // IPv6 link-local
  ];
  return privateRanges.some(range => range.test(ip));
}

async function isSafeUrl(urlString) {
  let parsed;
  try {
    parsed = new URL(urlString);
  } catch {
    return false;
  }
  
  // Allow only HTTP/HTTPS
  if (!['http:', 'https:'].includes(parsed.protocol)) {
    return false;
  }
  
  // Allowlist of allowed hosts
  const ALLOWED_HOSTS = [
    'api.external-service.com',
    'cdn.trusted-provider.com',
  ];
  
  if (!ALLOWED_HOSTS.includes(parsed.hostname)) {
    return false;
  }
  
  // Resolve DNS and check IP
  try {
    const addresses = await dns.lookup(parsed.hostname, { all: true });
    for (const { address } of addresses) {
      if (isPrivateIP(address)) {
        console.warn(`SSRF attempt: ${parsed.hostname} resolved to private IP ${address}`);
        return false;
      }
    }
  } catch {
    return false;
  }
  
  return true;
}

// Safe fetch wrapper
async function safeFetch(url, options = {}) {
  if (!await isSafeUrl(url)) {
    throw new Error('URL not allowed');
  }
  
  // Set timeout
  const controller = new AbortController();
  const timeout = setTimeout(() => controller.abort(), 5000);
  
  try {
    const response = await fetch(url, {
      ...options,
      signal: controller.signal,
      // ไม่ follow redirects โดยอัตโนมัติ
      redirect: 'manual',
    });
    
    // ตรวจสอบ redirect
    if (response.type === 'opaqueredirect') {
      throw new Error('Redirect not allowed');
    }
    
    return response;
  } finally {
    clearTimeout(timeout);
  }
}

// Middleware
app.post('/fetch-url', async (req, res, next) => {
  try {
    const { url } = req.body;
    const response = await safeFetch(url);
    const data = await response.text();
    res.json({ data: data.substring(0, 1000) }); // จำกัดขนาด
  } catch (error) {
    next(new Error('Cannot fetch URL: ' + error.message));
  }
});
```

---

## Step 1856: Mass Assignment Vulnerabilities

```javascript
// Mass Assignment เกิดเมื่อ user input ถูก assign ไปยัง model โดยตรง
// ผู้โจมตีสามารถ set fields ที่ไม่ควรแก้ไขได้

// ❌ โค้ดที่มีช่องโหว่
app.put('/users/:id', async (req, res) => {
  // Mass assignment! ผู้โจมตีส่ง { isAdmin: true, role: 'admin' }
  await User.findByIdAndUpdate(req.params.id, req.body);
  res.json({ success: true });
});

// Model
const UserModel = {
  username: String,
  email: String,
  password: String,
  isAdmin: Boolean,   // should not be user-settable!
  role: String,       // should not be user-settable!
  createdAt: Date,    // should not be user-settable!
};

// ✅ วิธีป้องกัน

// 1. Whitelist approach - ระบุ fields ที่ allowed
function sanitizeUserUpdate(body) {
  const ALLOWED_FIELDS = ['username', 'email', 'bio', 'avatarUrl'];
  
  return Object.fromEntries(
    Object.entries(body)
      .filter(([key]) => ALLOWED_FIELDS.includes(key))
  );
}

app.put('/users/:id', async (req, res) => {
  const sanitized = sanitizeUserUpdate(req.body);
  await User.findByIdAndUpdate(req.params.id, sanitized);
  res.json({ success: true });
});

// 2. DTO (Data Transfer Object) pattern
class UpdateUserDto {
  constructor(data) {
    // เฉพาะ fields ที่อนุญาต
    this.username = data.username;
    this.email = data.email;
    this.bio = data.bio;
    this.avatarUrl = data.avatarUrl;
    // ไม่รับ isAdmin, role, password
  }
  
  validate() {
    if (this.email && !isValidEmail(this.email)) {
      throw new Error('Invalid email');
    }
    if (this.username && this.username.length < 3) {
      throw new Error('Username too short');
    }
    return this;
  }
}

app.put('/users/:id', async (req, res, next) => {
  try {
    const dto = new UpdateUserDto(req.body).validate();
    await userService.updateUser(req.params.id, dto);
    res.json({ success: true });
  } catch (error) {
    next(error);
  }
});

// 3. ใช้ validation library
const { body, validationResult } = require('express-validator');

const updateUserValidation = [
  body('username').optional().isLength({ min: 3, max: 30 }).trim(),
  body('email').optional().isEmail().normalizeEmail(),
  body('bio').optional().isLength({ max: 500 }).trim().escape(),
  // ไม่มี isAdmin หรือ role
];

app.put('/users/:id',
  updateUserValidation,
  async (req, res, next) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
      return res.status(400).json({ errors: errors.array() });
    }
    
    // เฉพาะ validated fields
    const { username, email, bio } = req.body;
    await userService.updateUser(req.params.id, { username, email, bio });
    res.json({ success: true });
  }
);
```

---

## Step 1857: Timing Attacks and Constant-Time Comparison

```javascript
// Timing attacks เกิดเมื่อ attacker วัดเวลาการตอบสนอง
// เพื่อเดา secret values

// ❌ โค้ดที่มีช่องโหว่
function compareSecretsBad(userInput, secret) {
  // หยุดเมื่อพบตัวอักษรไม่ตรง - timing leak!
  if (userInput.length !== secret.length) return false;
  
  for (let i = 0; i < secret.length; i++) {
    if (userInput[i] !== secret[i]) {
      return false;  // เวลาขึ้นอยู่กับว่าตรงกันแค่ไหน
    }
  }
  return true;
}

// ✅ Constant-time comparison
function constantTimeEquals(a, b) {
  // ตรวจสอบทุกตัวอักษรเสมอ ไม่ว่าจะตรงหรือไม่
  if (a.length !== b.length) {
    // ยังต้องวน loop เพื่อ constant time
    // แต่ return false เมื่อ length ต่างกัน
    let diff = 0;
    for (let i = 0; i < b.length; i++) {
      diff |= a.charCodeAt(i % a.length) ^ b.charCodeAt(i);
    }
    return false;
  }
  
  let diff = 0;
  for (let i = 0; i < a.length; i++) {
    diff |= a.charCodeAt(i) ^ b.charCodeAt(i);
  }
  return diff === 0;
}

// ✅ ดีกว่า - ใช้ crypto.timingSafeEqual (Node.js built-in)
const crypto = require('crypto');

function safeCompare(a, b) {
  const bufA = Buffer.from(a);
  const bufB = Buffer.from(b);
  
  if (bufA.length !== bufB.length) {
    // สร้าง buffer เท่ากันเพื่อ constant time
    const dummy = Buffer.alloc(bufA.length);
    crypto.timingSafeEqual(bufA, dummy);  // fake comparison
    return false;
  }
  
  return crypto.timingSafeEqual(bufA, bufB);
}

// ตัวอย่างการใช้งาน
function verifyAPIKey(providedKey, storedKey) {
  return safeCompare(
    crypto.createHash('sha256').update(providedKey).digest('hex'),
    storedKey
  );
}

// HMAC verification - timing safe
function verifyHMAC(message, signature, secret) {
  const expectedSig = crypto
    .createHmac('sha256', secret)
    .update(message)
    .digest('hex');
  
  return safeCompare(signature, expectedSig);
}

// Webhook verification (เช่น GitHub webhooks)
function verifyGitHubWebhook(payload, signature, secret) {
  const hmac = crypto.createHmac('sha256', secret);
  const expectedSignature = 'sha256=' + hmac.update(payload).digest('hex');
  
  return crypto.timingSafeEqual(
    Buffer.from(signature),
    Buffer.from(expectedSignature)
  );
}
```

---

## Step 1858: Cryptography - Web Crypto API

```javascript
// Web Crypto API (Browser & Node.js >= 15)
// ใช้ global crypto object

// 1. Generating random values
const randomBytes = crypto.getRandomValues(new Uint8Array(32));
const randomHex = Array.from(randomBytes)
  .map(b => b.toString(16).padStart(2, '0'))
  .join('');

// Node.js
const { randomBytes: nodeRandomBytes } = require('crypto');
const secureRandom = nodeRandomBytes(32).toString('hex');

// 2. Hashing
async function hashData(data) {
  const encoder = new TextEncoder();
  const encoded = encoder.encode(data);
  const hashBuffer = await crypto.subtle.digest('SHA-256', encoded);
  const hashArray = Array.from(new Uint8Array(hashBuffer));
  return hashArray.map(b => b.toString(16).padStart(2, '0')).join('');
}

// Node.js synchronous
const nodeHash = crypto.createHash('sha256').update('data').digest('hex');

// 3. Password hashing (ต้องใช้ bcrypt, scrypt, argon2 ไม่ใช่ SHA)
const bcrypt = require('bcrypt');
const scrypt = require('scrypt-js');

// bcrypt
async function hashPassword(password) {
  const saltRounds = 12;
  return bcrypt.hash(password, saltRounds);
}

async function verifyPassword(password, hash) {
  return bcrypt.compare(password, hash);
}

// Node.js built-in scrypt
const { scrypt: nodeScrypt, randomBytes: rb } = require('crypto');
const { promisify } = require('util');
const scryptAsync = promisify(nodeScrypt);

async function hashPasswordScrypt(password) {
  const salt = rb(16);
  const key = await scryptAsync(password, salt, 64);
  return `${salt.toString('hex')}:${key.toString('hex')}`;
}

async function verifyPasswordScrypt(password, hash) {
  const [saltHex, keyHex] = hash.split(':');
  const salt = Buffer.from(saltHex, 'hex');
  const storedKey = Buffer.from(keyHex, 'hex');
  const key = await scryptAsync(password, salt, 64);
  return crypto.timingSafeEqual(key, storedKey);
}
```

---

## Step 1859: AES Encryption/Decryption

```javascript
// AES-GCM Encryption (Authenticated Encryption)

// Node.js
const crypto = require('crypto');

class AESEncryption {
  constructor(keyHex) {
    this.key = Buffer.from(keyHex, 'hex');
    if (this.key.length !== 32) {
      throw new Error('Key must be 256-bit (32 bytes)');
    }
  }

  encrypt(plaintext) {
    // AES-GCM ต้องการ unique IV ทุกครั้ง!
    const iv = crypto.randomBytes(12); // 96-bit IV for GCM
    
    const cipher = crypto.createCipheriv('aes-256-gcm', this.key, iv);
    
    const encrypted = Buffer.concat([
      cipher.update(plaintext, 'utf8'),
      cipher.final(),
    ]);
    
    const authTag = cipher.getAuthTag(); // 16 bytes
    
    // รวม iv + authTag + encrypted
    const result = Buffer.concat([iv, authTag, encrypted]);
    return result.toString('base64');
  }

  decrypt(encryptedBase64) {
    const data = Buffer.from(encryptedBase64, 'base64');
    
    // แยกส่วน
    const iv = data.slice(0, 12);
    const authTag = data.slice(12, 28);
    const encrypted = data.slice(28);
    
    const decipher = crypto.createDecipheriv('aes-256-gcm', this.key, iv);
    decipher.setAuthTag(authTag);
    
    try {
      const decrypted = Buffer.concat([
        decipher.update(encrypted),
        decipher.final(),
      ]);
      return decrypted.toString('utf8');
    } catch (error) {
      throw new Error('Decryption failed - data may be tampered');
    }
  }
}

// Key generation
function generateAESKey() {
  return crypto.randomBytes(32).toString('hex');
}

// ใช้งาน
const keyHex = generateAESKey();
const aes = new AESEncryption(keyHex);

const original = 'ข้อมูลลับสุดยอด';
const encrypted = aes.encrypt(original);
const decrypted = aes.decrypt(encrypted);
console.log(decrypted === original); // true

// Web Crypto API (Browser)
async function encryptBrowser(data, password) {
  const encoder = new TextEncoder();
  
  // Derive key from password
  const passwordKey = await crypto.subtle.importKey(
    'raw',
    encoder.encode(password),
    'PBKDF2',
    false,
    ['deriveKey']
  );
  
  const salt = crypto.getRandomValues(new Uint8Array(16));
  
  const key = await crypto.subtle.deriveKey(
    {
      name: 'PBKDF2',
      salt,
      iterations: 100000,
      hash: 'SHA-256',
    },
    passwordKey,
    { name: 'AES-GCM', length: 256 },
    false,
    ['encrypt']
  );
  
  const iv = crypto.getRandomValues(new Uint8Array(12));
  const encrypted = await crypto.subtle.encrypt(
    { name: 'AES-GCM', iv },
    key,
    encoder.encode(data)
  );
  
  // รวมทุกอย่างใน single buffer
  const result = new Uint8Array(salt.length + iv.length + encrypted.byteLength);
  result.set(salt, 0);
  result.set(iv, salt.length);
  result.set(new Uint8Array(encrypted), salt.length + iv.length);
  
  return btoa(String.fromCharCode(...result));
}

async function decryptBrowser(encryptedData, password) {
  const decoder = new TextDecoder();
  const encoder = new TextEncoder();
  
  const data = Uint8Array.from(atob(encryptedData), c => c.charCodeAt(0));
  
  const salt = data.slice(0, 16);
  const iv = data.slice(16, 28);
  const encrypted = data.slice(28);
  
  const passwordKey = await crypto.subtle.importKey(
    'raw',
    encoder.encode(password),
    'PBKDF2',
    false,
    ['deriveKey']
  );
  
  const key = await crypto.subtle.deriveKey(
    {
      name: 'PBKDF2',
      salt,
      iterations: 100000,
      hash: 'SHA-256',
    },
    passwordKey,
    { name: 'AES-GCM', length: 256 },
    false,
    ['decrypt']
  );
  
  const decrypted = await crypto.subtle.decrypt(
    { name: 'AES-GCM', iv },
    key,
    encrypted
  );
  
  return decoder.decode(decrypted);
}
```

---

## Step 1860: RSA Key Pair Generation and Digital Signatures

```javascript
// RSA Key Generation และ Digital Signatures

// Node.js
const { generateKeyPair, createSign, createVerify } = require('crypto');
const { promisify } = require('util');
const generateKeyPairAsync = promisify(generateKeyPair);

async function generateRSAKeyPair() {
  const { publicKey, privateKey } = await generateKeyPairAsync('rsa', {
    modulusLength: 2048,
    publicKeyEncoding: { type: 'spki', format: 'pem' },
    privateKeyEncoding: { type: 'pkcs8', format: 'pem' },
  });
  return { publicKey, privateKey };
}

// Digital Signature
function signData(data, privateKeyPem) {
  const sign = createSign('SHA256');
  sign.update(data);
  sign.end();
  return sign.sign(privateKeyPem, 'base64');
}

function verifySignature(data, signature, publicKeyPem) {
  const verify = createVerify('SHA256');
  verify.update(data);
  verify.end();
  return verify.verify(publicKeyPem, signature, 'base64');
}

// ใช้งาน
async function demonstrateSignature() {
  const { publicKey, privateKey } = await generateRSAKeyPair();
  
  const message = JSON.stringify({ userId: '123', action: 'transfer', amount: 1000 });
  const signature = signData(message, privateKey);
  
  const isValid = verifySignature(message, signature, publicKey);
  console.log('Signature valid:', isValid); // true
  
  // ทดสอบ tampered message
  const tampered = JSON.stringify({ userId: '123', action: 'transfer', amount: 9999 });
  const isValidTampered = verifySignature(tampered, signature, publicKey);
  console.log('Tampered valid:', isValidTampered); // false
}

// Web Crypto API - ECDSA (ดีกว่า RSA สำหรับ modern apps)
async function generateECKeyPair() {
  const keyPair = await crypto.subtle.generateKey(
    {
      name: 'ECDSA',
      namedCurve: 'P-256',
    },
    true,  // extractable
    ['sign', 'verify']
  );
  return keyPair;
}

async function signWithEC(data, privateKey) {
  const encoder = new TextEncoder();
  const signature = await crypto.subtle.sign(
    {
      name: 'ECDSA',
      hash: { name: 'SHA-256' },
    },
    privateKey,
    encoder.encode(data)
  );
  return signature;
}

async function verifyWithEC(data, signature, publicKey) {
  const encoder = new TextEncoder();
  return crypto.subtle.verify(
    {
      name: 'ECDSA',
      hash: { name: 'SHA-256' },
    },
    publicKey,
    signature,
    encoder.encode(data)
  );
}

// JWT signing ด้วย RS256
const jwt = require('jsonwebtoken');

async function createSecureJWT(payload, privateKey) {
  return jwt.sign(payload, privateKey, {
    algorithm: 'RS256',
    expiresIn: '1h',
    issuer: 'https://api.example.com',
    audience: 'https://app.example.com',
  });
}

function verifyJWT(token, publicKey) {
  return jwt.verify(token, publicKey, {
    algorithms: ['RS256'],
    issuer: 'https://api.example.com',
    audience: 'https://app.example.com',
  });
}
```

---

## Step 1861: Subresource Integrity (SRI)

```html
<!-- SRI ป้องกัน CDN tampering -->

<!-- ❌ ไม่มี SRI - ถ้า CDN ถูก hack, malicious code จะถูกโหลด -->
<script src="https://cdn.example.com/jquery.min.js"></script>

<!-- ✅ มี SRI - browser ตรวจสอบ hash -->
<script 
  src="https://cdn.jsdelivr.net/npm/lodash@4.17.21/lodash.min.js"
  integrity="sha256-qXBd/EfAdjOA2FGrGAG+b3YBn2tn5A6bhz+LSgYD96k="
  crossorigin="anonymous">
</script>

<link 
  rel="stylesheet" 
  href="https://cdn.example.com/style.css"
  integrity="sha384-AbCdEf..."
  crossorigin="anonymous">
```

```javascript
// Generate SRI hash
const crypto = require('crypto');
const fs = require('fs');

function generateSRIHash(filePath, algorithm = 'sha384') {
  const fileContent = fs.readFileSync(filePath);
  const hash = crypto.createHash(algorithm).update(fileContent).digest('base64');
  return `${algorithm}-${hash}`;
}

// สำหรับ URL
const https = require('https');

function generateSRIFromURL(url) {
  return new Promise((resolve, reject) => {
    https.get(url, (response) => {
      const chunks = [];
      response.on('data', chunk => chunks.push(chunk));
      response.on('end', () => {
        const content = Buffer.concat(chunks);
        const hash = crypto.createHash('sha384').update(content).digest('base64');
        resolve(`sha384-${hash}`);
      });
    }).on('error', reject);
  });
}

// ใช้งาน
async function checkSRI() {
  const sri = await generateSRIFromURL(
    'https://cdn.jsdelivr.net/npm/lodash@4.17.21/lodash.min.js'
  );
  console.log(`integrity="${sri}"`);
}

// Express middleware ที่เพิ่ม SRI
const sriHashes = {
  '/static/js/app.js': generateSRIHash('./dist/app.js'),
  '/static/css/style.css': generateSRIHash('./dist/style.css'),
};

app.use((req, res, next) => {
  res.locals.sriHash = (path) => sriHashes[path] || '';
  next();
});
```

---

## Step 1862: Trusted Types API

```javascript
// Trusted Types ป้องกัน DOM XSS โดย enforce safe DOM manipulation

// ตรวจสอบว่า browser รองรับ
if (window.trustedTypes && window.trustedTypes.createPolicy) {
  
  // สร้าง policy
  const policy = trustedTypes.createPolicy('default', {
    createHTML: (string) => {
      // sanitize HTML
      return DOMPurify.sanitize(string, { RETURN_TRUSTED_TYPE: true });
    },
    createScriptURL: (url) => {
      // validate URLs
      const allowed = ['https://cdn.example.com', 'https://api.example.com'];
      const parsed = new URL(url, location.origin);
      if (!allowed.some(a => parsed.href.startsWith(a))) {
        throw new Error(`URL not allowed: ${url}`);
      }
      return url;
    },
    createScript: (script) => {
      // ปกติไม่ควร allow
      throw new Error('Dynamic script not allowed');
    },
  });

  // ใช้ policy
  const userInput = '<p>Hello <strong>World</strong><script>alert(1)</script></p>';
  const safeHTML = policy.createHTML(userInput);
  document.getElementById('content').innerHTML = safeHTML;
  // script tag ถูก sanitize ออก

  // Dynamic script URL
  const scriptUrl = policy.createScriptURL('https://cdn.example.com/widget.js');
  const script = document.createElement('script');
  script.src = scriptUrl;
  document.body.appendChild(script);
}

// Content-Security-Policy สำหรับ Trusted Types
// Content-Security-Policy: require-trusted-types-for 'script'; trusted-types default

// Meta tag
// <meta http-equiv="Content-Security-Policy" 
//       content="require-trusted-types-for 'script'; trusted-types default">
```

---

## Step 1863: Advanced Content Security Policy

```javascript
// Advanced CSP configuration

// Express middleware
function advancedCSPMiddleware(req, res, next) {
  const nonce = crypto.randomBytes(16).toString('base64');
  res.locals.cspNonce = nonce;
  
  const policy = [
    // ค่า default ที่เข้มงวด
    "default-src 'none'",
    
    // Scripts: เฉพาะ nonce และ self
    `script-src 'self' 'nonce-${nonce}' https://cdn.jsdelivr.net`,
    
    // Styles: เฉพาะ nonce และ self
    `style-src 'self' 'nonce-${nonce}' https://fonts.googleapis.com`,
    
    // Images
    "img-src 'self' data: https:",
    
    // Fonts
    "font-src 'self' https://fonts.gstatic.com",
    
    // Connections: API endpoints ที่อนุญาต
    "connect-src 'self' https://api.example.com wss://ws.example.com",
    
    // Frames: ป้องกัน clickjacking
    "frame-src 'none'",
    "frame-ancestors 'none'",
    
    // Forms
    "form-action 'self'",
    
    // Base URI
    "base-uri 'self'",
    
    // Objects/embeds
    "object-src 'none'",
    
    // Manifests
    "manifest-src 'self'",
    
    // Workers
    "worker-src 'self'",
    
    // Report violations
    "report-uri /api/csp-report",
    "report-to csp-endpoint",
  ];
  
  res.setHeader('Content-Security-Policy', policy.join('; '));
  
  // Report-To header
  res.setHeader('Report-To', JSON.stringify({
    group: 'csp-endpoint',
    max_age: 10886400,
    endpoints: [{ url: '/api/csp-report' }],
  }));
  
  next();
}

// CSP Report handler
app.post('/api/csp-report', express.json({ type: 'application/csp-report' }), (req, res) => {
  const report = req.body['csp-report'];
  
  console.warn('CSP Violation:', {
    documentUri: report['document-uri'],
    blockedUri: report['blocked-uri'],
    violatedDirective: report['violated-directive'],
    originalPolicy: report['original-policy'],
  });
  
  // ส่งไปยัง monitoring service
  monitoringService.track('csp_violation', report);
  
  res.status(204).end();
});

// Helmet.js สำหรับ security headers
const helmet = require('helmet');

app.use(helmet({
  contentSecurityPolicy: {
    directives: {
      defaultSrc: ["'none'"],
      scriptSrc: ["'self'", (req, res) => `'nonce-${res.locals.cspNonce}'`],
      styleSrc: ["'self'"],
      imgSrc: ["'self'", 'data:', 'https:'],
      connectSrc: ["'self'"],
      frameSrc: ["'none'"],
      objectSrc: ["'none'"],
      baseUri: ["'self'"],
      formAction: ["'self'"],
      frameAncestors: ["'none'"],
    },
  },
  hsts: {
    maxAge: 31536000,
    includeSubDomains: true,
    preload: true,
  },
  referrerPolicy: { policy: 'strict-origin-when-cross-origin' },
  permissionsPolicy: {
    features: {
      camera: [],
      microphone: [],
      geolocation: [],
    },
  },
}));
```

---

## Step 1864: NPM Supply Chain Attacks

```bash
# Supply Chain Attacks ผ่าน npm

# 1. Typosquatting - ชื่อ package คล้ายกับ package จริง
# express vs expres (ไม่มี s)
# lodash vs iodash
# moment vs moment-js

# ตรวจสอบก่อนติดตั้ง
npm info express  # ตรวจสอบ package info
npm info express author  # ตรวจสอบ author

# 2. Dependency confusion - package ชื่อเดียวกันแต่ public vs private
# ถ้า registry public มี package ชื่อเดียวกับ private package
# npm จะดึง public version ที่อาจ malicious

# แก้ไขด้วย .npmrc
# @mycompany:registry=https://registry.mycompany.com
# //registry.mycompany.com/:_authToken=${PRIVATE_NPM_TOKEN}

# 3. npm audit
npm audit
npm audit --audit-level=moderate
npm audit fix
npm audit fix --force  # ระวัง! อาจ breaking changes

# 4. Lock files
# package-lock.json หรือ yarn.lock ต้อง commit เสมอ

# 5. ตรวจสอบ package integrity
npm ci  # ใช้ lock file เสมอ (CI/CD)

# 6. Socket.dev - security scanning
# https://socket.dev
```

```javascript
// Security scanning ใน GitHub Actions
// .github/workflows/security.yml

const securityWorkflow = `
name: Security Scan

on: [push, pull_request]

jobs:
  audit:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: 18
      
      - name: Install dependencies
        run: npm ci
      
      - name: NPM Audit
        run: npm audit --audit-level=high
      
      - name: Snyk Security Scan
        uses: snyk/actions/node@master
        env:
          SNYK_TOKEN: \${{ secrets.SNYK_TOKEN }}
      
      - name: License Check
        run: npx license-checker --failOn 'GPL;LGPL'
      
      - name: Dependency Review
        uses: actions/dependency-review-action@v2
`;

// ใช้ snyk
// npm install -g snyk
// snyk test
// snyk monitor

// ใช้ OWASP Dependency Check
// docker run --rm -v $(pwd):/src owasp/dependency-check:latest \
//   --project "My App" \
//   --scan /src/package.json
```

---

## Step 1865: Secret Scanning and Management

```javascript
// ป้องกัน secrets ใน codebase

// 1. .gitignore
const gitignoreEntries = `
# Secrets
.env
.env.local
.env.*.local
*.pem
*.key
*.cert
*.p12
secrets.json
config/secrets.js

# IDE
.vscode/settings.json  # อาจมี credentials
.idea/
`;

// 2. Pre-commit hook สำหรับตรวจจับ secrets
// .husky/pre-commit
const preCommitHook = `
#!/bin/sh
# ตรวจสอบ secrets ก่อน commit

npx detect-secrets scan --baseline .secrets.baseline
if [ $? -ne 0 ]; then
  echo "Potential secrets detected! Review and update .secrets.baseline"
  exit 1
fi
`;

// 3. detect-secrets
// pip install detect-secrets
// detect-secrets scan > .secrets.baseline
// detect-secrets audit .secrets.baseline

// 4. กำหนด patterns
// .gitignore ไม่เพียงพอ ต้องใช้ tools

// 5. GitHub Secret Scanning (built-in)
// Settings > Security > Secret scanning

// 6. Environment variables management
// ❌ ผิด - hardcode
const apiKey = 'sk-1234567890abcdef';

// ✅ ถูก - environment variables
const apiKeyFromEnv = process.env.API_KEY;

// ✅ ดีกว่า - ใช้ secrets manager
const { SecretsManager } = require('@aws-sdk/client-secrets-manager');

const secretsManager = new SecretsManager({ region: 'ap-southeast-1' });

async function getSecret(secretName) {
  const response = await secretsManager.getSecretValue({ SecretId: secretName });
  return JSON.parse(response.SecretString);
}

// ใช้งาน
const secrets = await getSecret('myapp/production');
const { databasePassword, apiKey: key } = secrets;

// 7. Vault (HashiCorp)
const vault = require('node-vault')();

async function getVaultSecret(path) {
  const result = await vault.read(path);
  return result.data;
}

// 8. dotenv-vault
// npm install dotenv-vault
// npx dotenv-vault new
// npx dotenv-vault push
// npx dotenv-vault pull
```

---

## Step 1866: Security Headers Checklist

```javascript
// Security Headers ครบถ้วน

const securityHeaders = {
  // ป้องกัน XSS
  'X-XSS-Protection': '1; mode=block',
  
  // ป้องกัน MIME sniffing
  'X-Content-Type-Options': 'nosniff',
  
  // ป้องกัน Clickjacking
  'X-Frame-Options': 'DENY',
  
  // HTTPS enforcement
  'Strict-Transport-Security': 'max-age=31536000; includeSubDomains; preload',
  
  // CSP
  'Content-Security-Policy': "default-src 'none'; ...",
  
  // Referrer policy
  'Referrer-Policy': 'strict-origin-when-cross-origin',
  
  // Permissions policy
  'Permissions-Policy': 'camera=(), microphone=(), geolocation=()',
  
  // Clear site data (สำหรับ logout)
  'Clear-Site-Data': '"cache", "cookies", "storage"',
  
  // ไม่ส่ง server version
  // ลบ 'X-Powered-By: Express'
};

function allSecurityHeaders(req, res, next) {
  // ลบข้อมูลที่เปิดเผย server info
  res.removeHeader('X-Powered-By');
  res.removeHeader('Server');
  
  // เพิ่ม security headers
  Object.entries(securityHeaders).forEach(([header, value]) => {
    res.setHeader(header, value);
  });
  
  next();
}

// ตรวจสอบ headers ด้วย SecurityHeaders.com
// https://securityheaders.com/?q=https://example.com

// ตรวจสอบ SSL/TLS
// https://www.ssllabs.com/ssltest/

// OWASP ZAP scan
// docker run -t owasp/zap2docker-stable zap-baseline.py -t https://example.com

// Middleware สำหรับ logout
app.post('/logout', (req, res) => {
  req.session.destroy();
  res.setHeader('Clear-Site-Data', '"cache", "cookies", "storage"');
  res.redirect('/login');
});
```

---

## Step 1867: OWASP ASVS Overview

```javascript
// OWASP Application Security Verification Standard (ASVS)
// https://owasp.org/www-project-application-security-verification-standard/

// ASVS Levels:
// L1 - Opportunistic (พื้นฐาน)
// L2 - Standard (แนะนำสำหรับ most apps)
// L3 - Advanced (สำหรับ critical apps)

// Key requirements checklist

const asvsChecklist = {
  
  // V1 - Architecture, Design and Threat Modeling
  architecture: [
    '✅ มี threat model',
    '✅ Security requirements documented',
    '✅ แยก trusted และ untrusted zones',
  ],
  
  // V2 - Authentication
  authentication: [
    '✅ Password ต้องมีอย่างน้อย 12 characters',
    '✅ ป้องกัน common passwords (top 10000)',
    '✅ ไม่แสดง password ใน logs',
    '✅ MFA สำหรับ sensitive operations',
    '✅ Account lockout หลัง failed attempts',
    '✅ Secure password reset flow',
  ],
  
  // V3 - Session Management
  sessions: [
    '✅ Sessions ถูกสร้างใน server-side',
    '✅ Session ID มี entropy >= 128 bits',
    '✅ Session หมดอายุหลัง inactive',
    '✅ Session invalidated หลัง logout',
    '✅ HttpOnly cookies',
    '✅ Secure cookies (HTTPS only)',
    '✅ SameSite=Strict หรือ Lax',
  ],
  
  // V4 - Access Control
  accessControl: [
    '✅ Deny by default',
    '✅ Authorization checks ทุก endpoint',
    '✅ ไม่ trust client-side authorization',
    '✅ Privilege escalation ถูกป้องกัน',
    '✅ Directory listing disabled',
  ],
  
  // V5 - Validation, Sanitization
  validation: [
    '✅ Validate all inputs',
    '✅ Reject invalid input (fail-safe)',
    '✅ Output encoding ตาม context',
    '✅ ป้องกัน injection attacks',
  ],
  
  // V9 - Communications
  communications: [
    '✅ TLS 1.2+ เท่านั้น',
    '✅ Strong cipher suites',
    '✅ HSTS enabled',
    '✅ Certificate validation',
  ],
  
  // V14 - Configuration
  configuration: [
    '✅ Production mode ไม่มี debug',
    '✅ Error messages ไม่แสดง stack trace',
    '✅ Security headers ครบ',
    '✅ ไม่มี default credentials',
  ],
};

// Implementation examples สำหรับแต่ละ requirement

// Password strength check
function isStrongPassword(password) {
  if (password.length < 12) return false;
  
  const commonPasswords = new Set([
    'password123', '123456789012', 'qwerty123456',
    // ... top common passwords list
  ]);
  
  if (commonPasswords.has(password.toLowerCase())) return false;
  
  // ตรวจสอบ variety (optional แต่แนะนำ)
  const hasLower = /[a-z]/.test(password);
  const hasUpper = /[A-Z]/.test(password);
  const hasDigit = /\d/.test(password);
  const hasSpecial = /[^a-zA-Z0-9]/.test(password);
  
  return hasLower && hasUpper && hasDigit;
}

// Account lockout
const lockoutStore = new Map(); // ควรใช้ Redis ใน production

async function checkLockout(username) {
  const key = `lockout:${username}`;
  const data = lockoutStore.get(key) || { attempts: 0, lockedUntil: null };
  
  if (data.lockedUntil && data.lockedUntil > Date.now()) {
    const remaining = Math.ceil((data.lockedUntil - Date.now()) / 1000);
    throw new Error(`บัญชีถูกล็อค กรุณารอ ${remaining} วินาที`);
  }
  
  return data;
}

async function recordFailedLogin(username) {
  const key = `lockout:${username}`;
  const data = lockoutStore.get(key) || { attempts: 0, lockedUntil: null };
  
  data.attempts++;
  
  if (data.attempts >= 5) {
    // Progressive lockout
    const lockoutMs = Math.pow(2, data.attempts - 5) * 30000; // 30s, 60s, 120s...
    data.lockedUntil = Date.now() + Math.min(lockoutMs, 3600000); // max 1 hour
  }
  
  lockoutStore.set(key, data);
}

async function clearLockout(username) {
  lockoutStore.delete(`lockout:${username}`);
}
```

---

## Step 1868: SQL Injection Prevention

```javascript
// SQL Injection - ยังคงเป็นหนึ่งใน top vulnerabilities

// ❌ โค้ดที่มีช่องโหว่
async function getUserBad(username) {
  // SQL Injection! ส่ง username = "' OR '1'='1"
  const query = `SELECT * FROM users WHERE username = '${username}'`;
  return db.execute(query);
}

// ✅ Parameterized Queries
async function getUserGood(username) {
  const query = 'SELECT * FROM users WHERE username = ?';
  return db.execute(query, [username]);
}

// PostgreSQL
async function getUserPostgres(username) {
  const query = 'SELECT * FROM users WHERE username = $1';
  return pg.query(query, [username]);
}

// ORM (Sequelize)
async function getUserORM(username) {
  return User.findOne({
    where: { username },  // ใช้ ORM = safe by default
  });
}

// MongoDB - NoSQL Injection
// ❌ ผิด
async function loginBad(username, password) {
  // ผู้โจมตีส่ง: { "$gt": "" } สำหรับ password
  return db.users.findOne({ username, password });
}

// ✅ ถูก
async function loginGood(username, password) {
  // ตรวจสอบ type ก่อน
  if (typeof username !== 'string' || typeof password !== 'string') {
    throw new Error('Invalid credentials type');
  }
  
  const user = await db.users.findOne({ username });
  if (!user) return null;
  
  // Compare password hash (bcrypt)
  const isValid = await bcrypt.compare(password, user.passwordHash);
  return isValid ? user : null;
}

// Express-validator สำหรับ sanitization
const { body, query } = require('express-validator');

app.get('/users',
  query('search').trim().escape().isLength({ max: 100 }),
  async (req, res) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) return res.status(400).json({ errors: errors.array() });
    
    const { search } = req.query;
    
    // Safe search with LIKE
    const users = await User.findAll({
      where: {
        name: { [Op.like]: `%${search}%` },  // Sequelize escapes this
      },
    });
    
    res.json(users);
  }
);
```

---

## Step 1869: Security in Dependencies

```javascript
// ตรวจสอบ security ของ dependencies

// 1. Snyk integration
// npm install -g snyk
// snyk auth
// snyk test

// .snyk policy file
const snykPolicy = `
version: v1.25.0
ignore: {}
patch: {}
`;

// 2. npm audit ใน CI/CD
// package.json
{
  "scripts": {
    "audit": "npm audit --audit-level=high",
    "audit:fix": "npm audit fix"
  }
}

// 3. Renovate bot - auto update dependencies
// renovate.json
const renovateConfig = {
  extends: ['config:base'],
  schedule: ['every weekend'],
  assignees: ['maintainer'],
  labels: ['dependencies'],
  automerge: true,
  automergeType: 'pr',
  automergeStrategy: 'squash',
  packageRules: [
    {
      matchUpdateTypes: ['minor', 'patch'],
      automerge: true,
    },
    {
      matchUpdateTypes: ['major'],
      automerge: false,
      labels: ['major-update'],
    },
  ],
};

// 4. ตรวจสอบ license
// npm install -g license-checker
// license-checker --onlyAllow 'MIT;ISC;BSD-3-Clause;Apache-2.0'
// license-checker --failOn 'GPL'

// 5. Package integrity verification
// npm install ด้วย --ignore-scripts ป้องกัน malicious postinstall
// npm install --ignore-scripts

// แต่ต้อง re-enable สำหรับ packages ที่ต้องการ build
// npm rebuild

// 6. Socket.dev
// npm install -g @socket.dev/cli
// socket npm install lodash
// วิเคราะห์ package ก่อนติดตั้ง

// 7. SBOM (Software Bill of Materials)
// npm install -g @cyclonedx/cyclonedx-npm
// cyclonedx-npm --output-format XML --output-file sbom.xml
```

---

## Step 1870: Complete Security Audit Checklist

```javascript
// Security Audit Checklist ครบถ้วน

const securityAudit = {
  
  // Authentication
  auth: {
    items: [
      { check: 'Strong password policy (min 12 chars)', critical: true },
      { check: 'Multi-factor authentication available', critical: true },
      { check: 'Account lockout mechanism', critical: true },
      { check: 'Secure password reset flow', critical: true },
      { check: 'Passwords hashed with bcrypt/scrypt/argon2', critical: true },
      { check: 'Sessions use secure tokens', critical: true },
      { check: 'Sessions invalidated on logout', critical: true },
    ],
  },
  
  // Input/Output
  io: {
    items: [
      { check: 'All inputs validated', critical: true },
      { check: 'Parameterized queries used', critical: true },
      { check: 'Output encoded appropriately', critical: true },
      { check: 'File upload validation', critical: true },
      { check: 'Content-Type validation', critical: false },
    ],
  },
  
  // Configuration
  config: {
    items: [
      { check: 'HTTPS enforced', critical: true },
      { check: 'Security headers configured', critical: true },
      { check: 'Debug mode disabled in production', critical: true },
      { check: 'Error details hidden from users', critical: true },
      { check: 'Secrets in environment variables', critical: true },
    ],
  },
  
  // Dependencies
  dependencies: {
    items: [
      { check: 'npm audit clean', critical: true },
      { check: 'Dependencies up to date', critical: false },
      { check: 'License compliance', critical: false },
    ],
  },
};

// สรุป audit
function runAudit(config) {
  const issues = [];
  
  Object.entries(config).forEach(([category, { items }]) => {
    items.forEach(item => {
      if (!item.passed) {
        issues.push({
          category,
          check: item.check,
          critical: item.critical,
          severity: item.critical ? 'HIGH' : 'MEDIUM',
        });
      }
    });
  });
  
  const critical = issues.filter(i => i.critical).length;
  const medium = issues.filter(i => !i.critical).length;
  
  return {
    issues,
    summary: {
      critical,
      medium,
      passed: critical === 0,
    },
  };
}

// Express security best practices
function securityBestPractices(app) {
  // Disable unnecessary headers
  app.disable('x-powered-by');
  
  // Parse JSON safely with size limit
  app.use(express.json({ limit: '10kb' }));
  app.use(express.urlencoded({ extended: false, limit: '10kb' }));
  
  // Rate limiting
  const rateLimit = require('express-rate-limit');
  app.use('/api/', rateLimit({
    windowMs: 15 * 60 * 1000,
    max: 100,
    message: 'Too many requests',
  }));
  
  // CORS
  const cors = require('cors');
  app.use(cors({
    origin: process.env.ALLOWED_ORIGINS?.split(',') || [],
    credentials: true,
    methods: ['GET', 'POST', 'PUT', 'DELETE', 'PATCH'],
    allowedHeaders: ['Content-Type', 'Authorization'],
  }));
  
  // Helmet
  app.use(require('helmet')());
  
  // CSRF Protection
  const csrf = require('csurf');
  app.use(csrf({ cookie: { httpOnly: true, secure: true, sameSite: 'strict' } }));
}
```

---

## แบบฝึกหัด (Exercises)

### Exercise 1: Threat Modeling
เลือก web application หนึ่งแบบ (เช่น ระบบ login):
1. ระบุ assets สำคัญ
2. ระบุ entry points
3. ใช้ STRIDE จำแนก threats
4. วาง mitigation strategies

### Exercise 2: Fix Vulnerable Code
แก้ไข vulnerabilities ในโค้ดต่อไปนี้:
```javascript
app.post('/login', async (req, res) => {
  const { username, password } = req.body;
  const user = await db.query(
    `SELECT * FROM users WHERE username='${username}' AND password='${password}'`
  );
  if (user.length > 0) {
    req.session.userId = user[0].id;
    res.redirect('/dashboard');
  } else {
    res.send('Invalid credentials');
  }
});
```

### Exercise 3: Implement AES Encryption
สร้าง utility class ที่:
- Encrypt/decrypt ข้อมูลด้วย AES-256-GCM
- ใช้ PBKDF2 สำหรับ key derivation
- รองรับทั้ง Buffer และ string
- มี proper error handling
- เขียน unit tests

### Exercise 4: Security Headers Audit
ตรวจสอบ security headers ของ website จริง:
1. ใช้ https://securityheaders.com
2. บันทึกว่า headers ใดขาดหายไป
3. Implement ที่ขาดหายด้วย Express

### Exercise 5: CSP Implementation
เขียน Express middleware ที่:
- สร้าง nonce ทุก request
- ตั้งค่า CSP ที่เข้มงวด
- Handle CSP violations reports
- ทดสอบว่า inline scripts ถูกบล็อก

---

*จบ Part 94: Advanced Security*
*ต่อไป Part 95: Blockchain กับ JavaScript*
