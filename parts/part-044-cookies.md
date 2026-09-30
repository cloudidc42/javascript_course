# Part 44: Cookies (Steps 851-870)

## บทนำ

Cookies เป็นวิธีการจัดเก็บข้อมูลขนาดเล็กบน client ที่ส่งไปกับทุก HTTP request โดยอัตโนมัติ ต่างจาก localStorage ที่ JavaScript เข้าถึงได้ฝั่งเดียว cookies สามารถอ่าน/เขียนได้ทั้ง client และ server

ในบทนี้เราจะเรียนรู้:
- การทำงานของ cookies
- การตั้งค่า attributes ต่างๆ
- Utility functions
- Security considerations
- Cookie consent

---

## Step 851: Cookies คืออะไร

```javascript
// Cookies คือ string ที่ browser เก็บไว้และส่งไปกับ HTTP requests
// Format: name=value; attribute1; attribute2; ...

// ดู cookies ปัจจุบัน
console.log(document.cookie);
// Output: "username=john; theme=dark; sessionId=abc123"

// Cookies properties:
// - name=value: ชื่อและค่า
// - expires: วันหมดอายุ
// - max-age: อายุเป็นวินาที
// - path: path ที่ cookie ใช้ได้
// - domain: domain ที่ cookie ใช้ได้
// - secure: ส่งเฉพาะ HTTPS
// - httponly: JavaScript เข้าไม่ได้ (server only)
// - samesite: ควบคุม cross-site requests

// ข้อจำกัดของ Cookies:
// - ขนาดสูงสุด: ~4KB ต่อ cookie
// - จำนวนสูงสุด: ~20 cookies ต่อ domain (ขึ้นกับ browser)
// - ส่งไปกับทุก request ไปยัง domain

// Cookies vs localStorage:
// | Feature        | Cookies          | localStorage    |
// |----------------|------------------|-----------------|
// | Size           | ~4KB             | ~5-10MB         |
// | Expiry         | ตั้งได้           | ไม่มี           |
// | Server access  | ✓               | ✗               |
// | JavaScript     | ✓               | ✓               |
// | Sent with req  | ✓               | ✗               |
// | HTTP only      | ✓               | ✗               |

// ตัวอย่างดู cookies ทั้งหมด
function getAllCookies() {
  if (!document.cookie) return {};
  
  return document.cookie
    .split(';')
    .reduce((acc, cookie) => {
      const [name, ...valueParts] = cookie.trim().split('=');
      acc[decodeURIComponent(name)] = decodeURIComponent(valueParts.join('='));
      return acc;
    }, {});
}

console.log('All cookies:', getAllCookies());
```

---

## Step 852: การตั้งค่า Cookie

```javascript
// ตั้งค่า cookie ง่ายๆ
document.cookie = 'username=สมชาย';

// Cookie จะถูก URL encode โดยอัตโนมัติบางกรณี
// แต่ควร encode เองเพื่อความปลอดภัย
document.cookie = `name=${encodeURIComponent('สมชาย มีดี')}`;

// ตั้งค่าพร้อม attributes
document.cookie = 'theme=dark; path=/; max-age=2592000'; // 30 วัน

// หมายเหตุ: document.cookie = ... ไม่ได้ replace ทุก cookies
// แต่เพิ่ม/แก้ไข cookie ที่ชื่อตรงกัน

// ตัวอย่างการตั้งค่า cookies ต่างๆ
function setCookieExamples() {
  // 1. Simple cookie (session cookie - หายเมื่อปิด browser)
  document.cookie = 'visitCount=1';
  
  // 2. Cookie ที่หมดอายุตามวันที่
  const expires = new Date('2025-12-31').toUTCString();
  document.cookie = `promotion=summer; expires=${expires}`;
  
  // 3. Cookie ที่หมดอายุตาม max-age (วินาที)
  document.cookie = 'lastVisit=' + Date.now() + '; max-age=86400'; // 1 วัน
  
  // 4. Cookie เฉพาะ path
  document.cookie = 'adminToken=xyz; path=/admin';
  
  // 5. Cookie สำหรับทุก subdomain
  document.cookie = 'userId=123; domain=.example.com; path=/';
  
  // 6. Secure cookie (HTTPS only)
  document.cookie = 'secureData=abc; secure; path=/';
  
  // 7. SameSite cookie
  document.cookie = 'csrfToken=xyz; samesite=strict; path=/; secure';
  
  // 8. Cookie ที่มีค่าเป็น JSON (ต้อง encode)
  const userData = { id: 1, role: 'admin' };
  document.cookie = `user=${encodeURIComponent(JSON.stringify(userData))}; path=/`;
}
```

---

## Step 853: Cookie Utility Functions

```javascript
// Cookie utility functions ครบครัน

const Cookies = {
  // ตั้งค่า cookie
  set(name, value, options = {}) {
    let cookie = `${encodeURIComponent(name)}=${encodeURIComponent(value)}`;
    
    // Expiry
    if (options.expires) {
      if (options.expires instanceof Date) {
        cookie += `; expires=${options.expires.toUTCString()}`;
      } else if (typeof options.expires === 'number') {
        // days
        const date = new Date();
        date.setTime(date.getTime() + (options.expires * 24 * 60 * 60 * 1000));
        cookie += `; expires=${date.toUTCString()}`;
      }
    }
    
    // Max-Age (วินาที)
    if (options.maxAge !== undefined) {
      cookie += `; max-age=${options.maxAge}`;
    }
    
    // Path
    cookie += `; path=${options.path || '/'}`;
    
    // Domain
    if (options.domain) {
      cookie += `; domain=${options.domain}`;
    }
    
    // Secure
    if (options.secure) {
      cookie += '; secure';
    }
    
    // SameSite
    if (options.sameSite) {
      cookie += `; samesite=${options.sameSite}`;
    }
    
    document.cookie = cookie;
    return true;
  },
  
  // อ่าน cookie
  get(name) {
    const nameStr = `${encodeURIComponent(name)}=`;
    const cookies = document.cookie.split(';');
    
    for (const cookie of cookies) {
      const trimmed = cookie.trim();
      if (trimmed.startsWith(nameStr)) {
        return decodeURIComponent(trimmed.substring(nameStr.length));
      }
    }
    
    return null;
  },
  
  // ตรวจสอบว่า cookie มีอยู่หรือไม่
  has(name) {
    return this.get(name) !== null;
  },
  
  // ดึงค่าเป็น JSON
  getJSON(name) {
    const value = this.get(name);
    if (!value) return null;
    
    try {
      return JSON.parse(value);
    } catch {
      return null;
    }
  },
  
  // บันทึก JSON
  setJSON(name, value, options = {}) {
    return this.set(name, JSON.stringify(value), options);
  },
  
  // ลบ cookie
  remove(name, options = {}) {
    this.set(name, '', {
      ...options,
      maxAge: -1,
      expires: new Date(0)
    });
  },
  
  // ดู cookies ทั้งหมด
  getAll() {
    if (!document.cookie) return {};
    
    return document.cookie
      .split(';')
      .reduce((acc, cookie) => {
        const eqIndex = cookie.indexOf('=');
        if (eqIndex === -1) return acc;
        
        const name = decodeURIComponent(cookie.slice(0, eqIndex).trim());
        const value = decodeURIComponent(cookie.slice(eqIndex + 1).trim());
        acc[name] = value;
        return acc;
      }, {});
  },
  
  // ลบ cookies ทั้งหมด (เฉพาะที่เข้าถึงได้)
  clearAll() {
    const allCookies = this.getAll();
    Object.keys(allCookies).forEach(name => this.remove(name));
  }
};

// ตัวอย่างการใช้งาน
Cookies.set('theme', 'dark', { expires: 30, path: '/' }); // 30 วัน
Cookies.set('lang', 'th', { maxAge: 86400 }); // 1 วัน (วินาที)

const theme = Cookies.get('theme');
console.log('Theme:', theme); // 'dark'

Cookies.setJSON('userPrefs', { 
  fontSize: 16, 
  notifications: true 
}, { expires: 365 }); // 1 ปี

const prefs = Cookies.getJSON('userPrefs');
console.log('Prefs:', prefs);

Cookies.remove('theme');
console.log('After remove:', Cookies.get('theme')); // null

console.log('All cookies:', Cookies.getAll());
```

---

## Step 854: Cookie Attributes รายละเอียด

```javascript
// expires
function setExpiringCookie() {
  // วันที่แน่นอน
  const expiryDate = new Date();
  expiryDate.setFullYear(expiryDate.getFullYear() + 1); // 1 ปี
  
  document.cookie = `annual_cookie=value; expires=${expiryDate.toUTCString()}; path=/`;
  
  // หรือใช้ helper
  Cookies.set('annualCookie', 'value', { expires: 365 });
}

// max-age vs expires
function maxAgeVsExpires() {
  // max-age เป็น relative time (วินาที จาก ตอนนี้)
  // expires เป็น absolute datetime
  
  // ดีกว่าใช้ max-age เพราะไม่ขึ้นกับ clock ของ client
  
  // 1 ชั่วโมง
  document.cookie = 'shortLived=value; max-age=3600; path=/';
  
  // 1 วัน
  document.cookie = 'oneDay=value; max-age=86400; path=/';
  
  // 1 สัปดาห์
  document.cookie = 'oneWeek=value; max-age=604800; path=/';
  
  // 1 เดือน
  document.cookie = 'oneMonth=value; max-age=2592000; path=/';
  
  // 1 ปี
  document.cookie = 'oneYear=value; max-age=31536000; path=/';
}

// path attribute
function pathCookies() {
  // Cookie เฉพาะ root (ทุก path)
  document.cookie = 'global=value; path=/';
  
  // Cookie เฉพาะ /admin
  document.cookie = 'adminOnly=value; path=/admin';
  // เข้าถึงได้ที่ /admin, /admin/users, /admin/settings
  // ไม่สามารถเข้าถึงได้ที่ /, /profile
  
  // Cookie เฉพาะ /shop/cart
  document.cookie = 'cartData=value; path=/shop/cart';
}

// domain attribute
function domainCookies() {
  // เฉพาะ domain ปัจจุบัน
  document.cookie = 'strictDomain=value; path=/';
  // ใช้ได้ที่ www.example.com เท่านั้น
  
  // ทุก subdomain
  document.cookie = 'sharedCookie=value; domain=.example.com; path=/';
  // ใช้ได้ที่ www.example.com, api.example.com, shop.example.com
  
  // หมายเหตุ: ไม่สามารถตั้ง domain เป็น domain อื่น (security)
}
```

---

## Step 855: Session Cookies vs Persistent Cookies

```javascript
// Session Cookies - หายเมื่อปิด browser
function setSessionCookie(name, value) {
  // ไม่ระบุ expires หรือ max-age = session cookie
  document.cookie = `${name}=${encodeURIComponent(value)}; path=/`;
}

// Persistent Cookies - อยู่จนหมดอายุ
function setPersistentCookie(name, value, days) {
  const expires = new Date();
  expires.setDate(expires.getDate() + days);
  document.cookie = `${name}=${encodeURIComponent(value)}; expires=${expires.toUTCString()}; path=/`;
}

// Use cases ของแต่ละแบบ:

// Session Cookies ใช้สำหรับ:
// - Shopping cart ชั่วคราว
// - Form state ระหว่าง pages
// - Authentication tokens (ใน memory)
// - Temporary user preferences

// Persistent Cookies ใช้สำหรับ:
// - Remember me (login)
// - User preferences (ภาษา, theme)
// - Analytics identifiers
// - Affiliate tracking

// ตัวอย่าง Remember Me
class RememberMeManager {
  constructor() {
    this.cookieName = 'rememberToken';
    this.duration = 30; // 30 วัน
  }
  
  setRememberMe(token) {
    Cookies.set(this.cookieName, token, {
      expires: this.duration,
      secure: true,
      sameSite: 'strict',
      path: '/'
    });
  }
  
  getToken() {
    return Cookies.get(this.cookieName);
  }
  
  clearRememberMe() {
    Cookies.remove(this.cookieName);
  }
  
  isRemembered() {
    return this.getToken() !== null;
  }
}

const rememberMe = new RememberMeManager();

// Login form
async function handleLogin(username, password, remember) {
  try {
    // const { token } = await authAPI.login(username, password);
    const token = 'mock_token_' + Date.now();
    
    if (remember) {
      rememberMe.setRememberMe(token);
    } else {
      // Session only
      setSessionCookie('authToken', token);
    }
    
    console.log('เข้าสู่ระบบสำเร็จ');
  } catch (error) {
    console.error('เข้าสู่ระบบไม่สำเร็จ:', error.message);
  }
}

// เช็คตอน load
function checkRememberedLogin() {
  const token = rememberMe.getToken() || Cookies.get('authToken');
  
  if (token) {
    console.log('พบ saved login token, auto-login...');
    return token;
  }
  
  return null;
}
```

---

## Step 856: SameSite Attribute

```javascript
// SameSite Attribute - ควบคุม cross-site requests

// SameSite=Strict
// - ส่ง cookie เฉพาะ same-site requests เท่านั้น
// - ปลอดภัยที่สุด แต่อาจทำให้ user experience แย่ลง
// - กด link จากเว็บอื่นมา จะไม่ส่ง cookie

function setStrictCookie(name, value) {
  Cookies.set(name, value, {
    sameSite: 'strict',
    secure: true,
    path: '/'
  });
}
// ใช้สำหรับ: banking operations, admin panels

// SameSite=Lax (Default ใน modern browsers)
// - ส่ง cookie เมื่อ navigate ด้วย GET top-level navigation
// - ไม่ส่งสำหรับ cross-site POST, iframes, images, AJAX

function setLaxCookie(name, value) {
  Cookies.set(name, value, {
    sameSite: 'lax',
    path: '/'
  });
}
// ใช้สำหรับ: session cookies, user preferences

// SameSite=None
// - ส่ง cookie ใน cross-site requests
// - ต้องใช้ร่วมกับ Secure
// - ใช้สำหรับ third-party integrations

function setCrossSiteCookie(name, value) {
  Cookies.set(name, value, {
    sameSite: 'none',
    secure: true,  // จำเป็นต้องมี
    path: '/'
  });
}
// ใช้สำหรับ: analytics, payment gateways, embedded widgets

// CSRF Protection ด้วย SameSite
function setCSRFToken(token) {
  // CSRF token ควรเป็น SameSite=Strict
  Cookies.set('csrfToken', token, {
    sameSite: 'strict',
    secure: true,
    path: '/'
  });
}

// Middleware สำหรับ verify CSRF
function verifyCsrfToken(req) {
  const cookieToken = req.cookies.csrfToken;
  const headerToken = req.headers['x-csrf-token'];
  
  if (!cookieToken || !headerToken || cookieToken !== headerToken) {
    throw new Error('CSRF token ไม่ถูกต้อง');
  }
}
```

---

## Step 857: Secure Cookies

```javascript
// Secure Cookie - ส่งเฉพาะ HTTPS

function setSecureCookie(name, value, options = {}) {
  const isHttps = window.location.protocol === 'https:';
  
  if (!isHttps) {
    console.warn(`⚠️ ไม่ควรตั้ง cookie "${name}" บน HTTP`);
  }
  
  Cookies.set(name, value, {
    ...options,
    secure: true,
    path: '/'
  });
}

// Cookie Security Best Practices

class SecureCookieManager {
  static set(name, value, options = {}) {
    const isProduction = window.location.hostname !== 'localhost';
    
    const secureOptions = {
      path: '/',
      sameSite: 'lax',       // Default safe
      secure: isProduction,   // HTTPS in production
      ...options
    };
    
    Cookies.set(name, value, secureOptions);
  }
  
  // Sensitive cookie (auth tokens, etc.)
  static setSecure(name, value, options = {}) {
    SecureCookieManager.set(name, value, {
      sameSite: 'strict',
      secure: true,
      ...options
    });
  }
  
  // Cookie สำหรับ preferences (ที่ไม่ sensitive)
  static setPreference(name, value, options = {}) {
    SecureCookieManager.set(name, value, {
      expires: 365, // 1 ปี
      sameSite: 'lax',
      ...options
    });
  }
}

// ตัวอย่างการใช้งาน
SecureCookieManager.setSecure('authToken', 'abc123xyz', { 
  maxAge: 3600 // 1 ชั่วโมง
});

SecureCookieManager.setPreference('language', 'th');
SecureCookieManager.setPreference('theme', 'dark');

// Cookie Encryption (สำหรับ sensitive data)
class EncryptedCookies {
  constructor(secretKey) {
    this.key = secretKey;
  }
  
  // Simple XOR encryption (ไม่ใช้ใน production จริง)
  #xorEncrypt(text, key) {
    return btoa(
      text.split('').map((char, i) => 
        String.fromCharCode(char.charCodeAt(0) ^ key.charCodeAt(i % key.length))
      ).join('')
    );
  }
  
  #xorDecrypt(encoded, key) {
    const text = atob(encoded);
    return text.split('').map((char, i) =>
      String.fromCharCode(char.charCodeAt(0) ^ key.charCodeAt(i % key.length))
    ).join('');
  }
  
  set(name, value, options = {}) {
    const encrypted = this.#xorEncrypt(JSON.stringify(value), this.key);
    Cookies.set(name, encrypted, { secure: true, sameSite: 'strict', ...options });
  }
  
  get(name) {
    const encrypted = Cookies.get(name);
    if (!encrypted) return null;
    
    try {
      return JSON.parse(this.#xorDecrypt(encrypted, this.key));
    } catch {
      return null;
    }
  }
}

// การใช้งาน Encrypted Cookies
const encryptedCookies = new EncryptedCookies('my-secret-key-2024');
encryptedCookies.set('userData', { userId: 123, role: 'admin' }, { maxAge: 3600 });
console.log('Decrypted:', encryptedCookies.get('userData'));
```

---

## Step 858: Cookie vs localStorage

```javascript
// เปรียบเทียบ Cookie vs localStorage

// เมื่อใดควรใช้ Cookie:
// 1. ข้อมูลที่ server ต้องอ่านด้วย (session ID, auth token)
// 2. ต้องการควบคุม expiry อย่างละเอียด
// 3. ต้องการ cross-subdomain sharing
// 4. ต้องการ HttpOnly (JS เข้าไม่ได้)
// 5. Cross-site requests ที่ต้องการ state

// เมื่อใดควรใช้ localStorage:
// 1. ข้อมูลขนาดใหญ่ (> 4KB)
// 2. Client-only data
// 3. ไม่ต้องการส่งกับทุก request
// 4. Offline-first applications

// ตัวอย่าง: Authentication State Management

class AuthStorage {
  // Token ที่ server ต้องอ่าน -> Cookie
  setServerToken(token) {
    Cookies.set('sessionToken', token, {
      httpOnly: true,  // ใน browser JS ทำไม่ได้จริง แต่ concept
      secure: true,
      sameSite: 'lax',
      maxAge: 3600
    });
  }
  
  // User data ที่ client ต้องการ -> localStorage
  setUserData(userData) {
    localStorage.setItem('userData', JSON.stringify(userData));
  }
  
  // Preferences -> localStorage หรือ Cookie (ขึ้นกับว่า server ต้องอ่านไหม)
  setTheme(theme) {
    // ถ้า server-side render ต้องใช้ cookie
    Cookies.set('theme', theme, { expires: 365 });
    // ถ้า client-side only ใช้ localStorage
    localStorage.setItem('theme', theme);
  }
  
  getUserData() {
    try {
      return JSON.parse(localStorage.getItem('userData'));
    } catch {
      return null;
    }
  }
  
  clearAll() {
    Cookies.remove('sessionToken');
    localStorage.removeItem('userData');
    localStorage.removeItem('theme');
  }
}

// Decision Matrix
function chooseStorage(requirements) {
  const { 
    serverNeedsAccess,  // Server ต้องอ่านไหม?
    sizeKB,            // ขนาดข้อมูล
    needsExpiry,       // ต้องการ expiry?
    sensitive,         // ข้อมูล sensitive?
    crossSubdomain     // ต้องการ cross-subdomain?
  } = requirements;
  
  if (serverNeedsAccess) return 'cookie';
  if (sizeKB > 4) return 'localStorage';
  if (crossSubdomain) return 'cookie';
  if (sensitive && needsExpiry) return 'cookie'; // HttpOnly + Expiry
  
  return needsExpiry ? 'cookie' : 'localStorage';
}

// ตัวอย่าง
console.log(chooseStorage({ serverNeedsAccess: true }));     // 'cookie'
console.log(chooseStorage({ sizeKB: 100 }));                 // 'localStorage'
console.log(chooseStorage({ needsExpiry: false }));          // 'localStorage'
console.log(chooseStorage({ crossSubdomain: true }));        // 'cookie'
```

---

## Step 859: Third-Party Cookies

```javascript
// Third-Party Cookies - cookies จาก domain อื่น
// ถูกใช้โดย: analytics, advertising, social buttons

// ตัวอย่าง third-party cookie patterns
// (JavaScript ที่ embed จาก domain อื่น)

// First-party vs Third-party:
// First-party: cookie จาก domain ที่ user กำลัง visit
// Third-party: cookie จาก domain อื่น (embedded content)

// ปัญหา:
// - Privacy concerns
// - Browser blocking (Safari ITP, Firefox, Chrome planned)
// - GDPR compliance

// Alternative approaches:
// 1. First-party cookies + server-to-server communication
// 2. Privacy-preserving alternatives (Topics API, FLEDGE)
// 3. Local processing ด้วย JavaScript

// ตรวจสอบว่า third-party cookies ถูก block หรือไม่
function checkThirdPartyCookies() {
  return new Promise((resolve) => {
    const testCookieName = 'tpc_test';
    
    // พยายามตั้ง cookie
    document.cookie = `${testCookieName}=1; SameSite=None; Secure`;
    
    // ตรวจสอบ
    const blocked = !document.cookie.includes(testCookieName);
    
    // ลบ test cookie
    document.cookie = `${testCookieName}=; max-age=-1`;
    
    resolve(!blocked);
  });
}

// Alternative: ใช้ postMessage แทน third-party cookies
// ใน parent page:
window.addEventListener('message', (event) => {
  if (event.origin !== 'https://trusted-third-party.com') return;
  
  const { action, data } = event.data;
  if (action === 'storeData') {
    localStorage.setItem('thirdPartyData', JSON.stringify(data));
  }
});

// ใน iframe:
// window.parent.postMessage({
//   action: 'storeData',
//   data: { userId: '123', tracking: 'abc' }
// }, 'https://your-site.com');
```

---

## Step 860: Cookie Security Considerations

```javascript
// Security Considerations

// 1. XSS (Cross-Site Scripting) Protection
// HttpOnly cookie ป้องกัน JS access
// แต่ document.cookie เป็น frontend API จึงทำ HttpOnly จาก JS ไม่ได้จริง
// ต้องทำจาก server: Set-Cookie: name=value; HttpOnly

// ตรวจสอบ XSS vulnerability
function sanitizeForCookie(value) {
  // ลบ characters ที่อันตราย
  return value
    .replace(/[;\s]/g, '')  // ลบ semicolon และ whitespace
    .replace(/['"]/g, '')    // ลบ quotes
    .substring(0, 4000);     // จำกัดขนาด
}

// 2. CSRF (Cross-Site Request Forgery) Protection

class CSRFProtection {
  static generateToken() {
    // ใช้ crypto.getRandomValues สำหรับ secure random
    const array = new Uint8Array(32);
    crypto.getRandomValues(array);
    return Array.from(array, byte => byte.toString(16).padStart(2, '0')).join('');
  }
  
  static setToken() {
    const token = this.generateToken();
    
    // Cookie: accessible by JS (ไม่ HttpOnly)
    Cookies.set('csrfToken', token, {
      sameSite: 'strict',
      secure: true,
      path: '/'
    });
    
    return token;
  }
  
  static getToken() {
    return Cookies.get('csrfToken');
  }
  
  // เพิ่ม CSRF token ใน headers
  static addToHeaders(headers = {}) {
    const token = this.getToken();
    if (token) {
      headers['X-CSRF-Token'] = token;
    }
    return headers;
  }
  
  // Wrapper สำหรับ fetch
  static async secureFetch(url, options = {}) {
    const token = this.getToken() || this.setToken();
    
    return fetch(url, {
      ...options,
      headers: {
        ...options.headers,
        'X-CSRF-Token': token,
        'Content-Type': 'application/json'
      }
    });
  }
}

// ใช้งาน
const token = CSRFProtection.setToken();
console.log('CSRF Token:', token);

// Secure API call
CSRFProtection.secureFetch('/api/user/update', {
  method: 'POST',
  body: JSON.stringify({ name: 'สมชาย' })
}).then(response => response.json())
  .then(data => console.log('Response:', data));

// 3. Cookie Theft Prevention
// - ใช้ HttpOnly สำหรับ auth cookies (server-side)
// - ใช้ Secure (HTTPS only)
// - ตั้ง SameSite ให้เหมาะสม
// - Monitor suspicious activity

// 4. Cookie Poisoning Detection
function validateCookieIntegrity(name, expectedFormat) {
  const value = Cookies.get(name);
  
  if (!value) return null;
  
  // Validate format
  if (expectedFormat instanceof RegExp && !expectedFormat.test(value)) {
    console.warn(`Cookie "${name}" มีรูปแบบไม่ถูกต้อง`);
    Cookies.remove(name);
    return null;
  }
  
  return value;
}

// ตัวอย่าง
const userId = validateCookieIntegrity('userId', /^\d+$/);
console.log('Valid userId:', userId);
```

---

## Step 861: Cookie Consent (GDPR)

```javascript
// Cookie Consent Management

class CookieConsentManager {
  constructor(options = {}) {
    this.consentCookieName = options.cookieName || 'cookieConsent';
    this.consentExpiry = options.expiry || 365; // 1 ปี
    
    this.categories = {
      necessary: {
        label: 'จำเป็น',
        description: 'Cookies ที่จำเป็นสำหรับการทำงานของเว็บไซต์',
        required: true  // บังคับ ผู้ใช้ไม่สามารถปฏิเสธได้
      },
      analytics: {
        label: 'วิเคราะห์',
        description: 'ช่วยให้เราเข้าใจการใช้งานเว็บไซต์',
        required: false
      },
      marketing: {
        label: 'การตลาด',
        description: 'ใช้สำหรับโฆษณาที่ตรงกับความสนใจ',
        required: false
      },
      preferences: {
        label: 'การตั้งค่า',
        description: 'จดจำการตั้งค่าของท่าน',
        required: false
      }
    };
  }
  
  // ดึงค่า consent ปัจจุบัน
  getConsent() {
    const consent = Cookies.getJSON(this.consentCookieName);
    
    if (!consent) return null;
    
    return consent;
  }
  
  // ตรวจสอบว่า category ได้รับ consent หรือไม่
  hasConsent(category) {
    const consent = this.getConsent();
    
    // Necessary cookies ไม่ต้อง consent
    if (this.categories[category]?.required) return true;
    
    if (!consent) return false;
    
    return consent[category] === true;
  }
  
  // บันทึก consent
  saveConsent(choices) {
    const consent = {
      necessary: true, // บังคับ
      analytics: choices.analytics || false,
      marketing: choices.marketing || false,
      preferences: choices.preferences || false,
      timestamp: new Date().toISOString(),
      version: '1.0'
    };
    
    Cookies.set(this.consentCookieName, JSON.stringify(consent), {
      expires: this.consentExpiry,
      sameSite: 'lax',
      path: '/'
    });
    
    // Apply consent
    this.#applyConsent(consent);
    
    console.log('บันทึก Cookie Consent:', consent);
    return consent;
  }
  
  // ยอมรับทั้งหมด
  acceptAll() {
    return this.saveConsent({
      analytics: true,
      marketing: true,
      preferences: true
    });
  }
  
  // ปฏิเสธทั้งหมด (เก็บเฉพาะ necessary)
  rejectAll() {
    return this.saveConsent({
      analytics: false,
      marketing: false,
      preferences: false
    });
  }
  
  // ถอน consent
  withdraw() {
    Cookies.remove(this.consentCookieName);
    console.log('ถอน Cookie Consent แล้ว');
    
    // ลบ non-necessary cookies
    this.#removeNonNecessaryCookies();
  }
  
  // ต้องการ consent dialog หรือไม่
  needsConsent() {
    const consent = this.getConsent();
    return !consent; // ถ้าไม่มี consent ต้องแสดง dialog
  }
  
  #applyConsent(consent) {
    // Analytics
    if (consent.analytics) {
      this.#initAnalytics();
    } else {
      this.#removeAnalytics();
    }
    
    // Marketing
    if (consent.marketing) {
      this.#initMarketing();
    }
    
    // Preferences
    if (consent.preferences) {
      this.#initPreferences();
    }
  }
  
  #initAnalytics() {
    console.log('Initializing analytics...');
    // Google Analytics, Mixpanel, etc.
  }
  
  #removeAnalytics() {
    // ลบ analytics cookies
    const analyticsCookies = ['_ga', '_gid', '_gat', '__utma', '__utmz'];
    analyticsCookies.forEach(name => Cookies.remove(name));
    console.log('Removed analytics cookies');
  }
  
  #initMarketing() {
    console.log('Initializing marketing...');
    // Facebook Pixel, Google Ads, etc.
  }
  
  #initPreferences() {
    console.log('Initializing preferences...');
    // Restore user preferences
  }
  
  #removeNonNecessaryCookies() {
    const allCookies = Cookies.getAll();
    const necessaryCookies = ['session', 'csrfToken', this.consentCookieName];
    
    Object.keys(allCookies).forEach(name => {
      if (!necessaryCookies.some(n => name.includes(n))) {
        Cookies.remove(name);
      }
    });
    
    console.log('ลบ non-necessary cookies แล้ว');
  }
}

// ใช้งาน
const cookieConsent = new CookieConsentManager();

// เช็คตอน load
if (cookieConsent.needsConsent()) {
  console.log('แสดง Cookie Consent Banner');
  showConsentBanner();
} else {
  const consent = cookieConsent.getConsent();
  console.log('มี consent อยู่แล้ว:', consent);
}

function showConsentBanner() {
  // สร้าง UI สำหรับ consent
  console.log('Showing consent banner...');
  
  // Simulate user accepting all
  cookieConsent.acceptAll();
}

// ตรวจสอบก่อนใช้งาน cookie ใดๆ
function trackUserAction(action) {
  if (!cookieConsent.hasConsent('analytics')) {
    console.log('Analytics disabled by user');
    return;
  }
  
  // บันทึก analytics
  console.log('Tracking:', action);
}
```

---

## Step 862: Cookie Store API (Modern)

```javascript
// Cookie Store API - async API ที่ทันสมัยกว่า

// ตรวจสอบการรองรับ
const isCookieStoreSupported = 'cookieStore' in window;

// Modern async cookie operations
if (isCookieStoreSupported) {
  // อ่าน cookie
  async function modernGetCookie(name) {
    const cookie = await cookieStore.get(name);
    return cookie?.value || null;
  }
  
  // อ่านทุก cookies
  async function modernGetAllCookies() {
    const cookies = await cookieStore.getAll();
    return cookies.reduce((acc, cookie) => {
      acc[cookie.name] = cookie.value;
      return acc;
    }, {});
  }
  
  // ตั้งค่า cookie
  async function modernSetCookie(name, value, options = {}) {
    await cookieStore.set({
      name,
      value,
      expires: options.expires,
      path: options.path || '/',
      domain: options.domain,
      secure: options.secure,
      sameSite: options.sameSite || 'lax'
    });
  }
  
  // ลบ cookie
  async function modernDeleteCookie(name) {
    await cookieStore.delete(name);
  }
  
  // ฟัง cookie changes
  async function watchCookieChanges() {
    cookieStore.addEventListener('change', (event) => {
      event.changed.forEach(cookie => {
        console.log(`Cookie changed: ${cookie.name} = ${cookie.value}`);
      });
      
      event.deleted.forEach(cookie => {
        console.log(`Cookie deleted: ${cookie.name}`);
      });
    });
  }
  
  // Subscribe to specific cookie changes
  async function watchSpecificCookie(name) {
    await cookieStore.observe({
      name: [name]
    });
    
    cookieStore.addEventListener('change', (event) => {
      for (const cookie of event.changed) {
        if (cookie.name === name) {
          console.log(`${name} changed to: ${cookie.value}`);
        }
      }
    });
  }
  
  // ตัวอย่าง
  (async () => {
    await modernSetCookie('testCookie', 'hello', {
      expires: Date.now() + 3600000, // 1 hour
      secure: true,
      sameSite: 'lax'
    });
    
    const value = await modernGetCookie('testCookie');
    console.log('Cookie value:', value); // 'hello'
    
    await modernDeleteCookie('testCookie');
    console.log('After delete:', await modernGetCookie('testCookie')); // null
  })();
}

// Polyfill สำหรับ browser ที่ไม่รองรับ
const cookieAPI = {
  async get(name) {
    if (isCookieStoreSupported) {
      const cookie = await cookieStore.get(name);
      return cookie?.value || null;
    }
    return Cookies.get(name);
  },
  
  async set(name, value, options) {
    if (isCookieStoreSupported) {
      await cookieStore.set({ name, value, ...options });
    } else {
      Cookies.set(name, value, options);
    }
  },
  
  async delete(name) {
    if (isCookieStoreSupported) {
      await cookieStore.delete(name);
    } else {
      Cookies.remove(name);
    }
  }
};
```

---

## Step 863-870: Practical Examples

```javascript
// Shopping Cart Cookie Manager

class CartCookieManager {
  constructor() {
    this.cookieName = 'cart';
    this.maxItems = 20;
    this.expireDays = 7; // 7 วัน
  }
  
  getCart() {
    return Cookies.getJSON(this.cookieName) || { items: [], updatedAt: null };
  }
  
  saveCart(cart) {
    cart.updatedAt = new Date().toISOString();
    Cookies.setJSON(this.cookieName, cart, {
      expires: this.expireDays,
      path: '/',
      sameSite: 'lax'
    });
  }
  
  addItem(product, qty = 1) {
    const cart = this.getCart();
    const existingIndex = cart.items.findIndex(item => item.id === product.id);
    
    if (existingIndex >= 0) {
      cart.items[existingIndex].qty += qty;
    } else {
      if (cart.items.length >= this.maxItems) {
        throw new Error(`Cart เต็มแล้ว (สูงสุด ${this.maxItems} items)`);
      }
      cart.items.push({ 
        id: product.id, 
        name: product.name, 
        price: product.price, 
        qty 
      });
    }
    
    this.saveCart(cart);
    return cart;
  }
  
  removeItem(productId) {
    const cart = this.getCart();
    cart.items = cart.items.filter(item => item.id !== productId);
    this.saveCart(cart);
    return cart;
  }
  
  updateQty(productId, qty) {
    const cart = this.getCart();
    const item = cart.items.find(i => i.id === productId);
    
    if (!item) throw new Error('ไม่พบ item ใน cart');
    
    if (qty <= 0) {
      return this.removeItem(productId);
    }
    
    item.qty = qty;
    this.saveCart(cart);
    return cart;
  }
  
  getTotal() {
    const cart = this.getCart();
    return cart.items.reduce((total, item) => total + (item.price * item.qty), 0);
  }
  
  getItemCount() {
    const cart = this.getCart();
    return cart.items.reduce((count, item) => count + item.qty, 0);
  }
  
  clear() {
    Cookies.remove(this.cookieName);
  }
  
  isEmpty() {
    return this.getCart().items.length === 0;
  }
}

// ใช้งาน Shopping Cart
const cart = new CartCookieManager();

cart.addItem({ id: 'P1', name: 'เสื้อยืด', price: 299 }, 2);
cart.addItem({ id: 'P2', name: 'กางเกง', price: 599 }, 1);
cart.addItem({ id: 'P1', name: 'เสื้อยืด', price: 299 }, 1); // เพิ่ม qty

console.log('Cart items:', cart.getCart().items);
console.log('Total:', cart.getTotal().toLocaleString('th-TH', { style: 'currency', currency: 'THB' }));
console.log('Item count:', cart.getItemCount());

// User Tracking (with consent)
class UserTracker {
  constructor(consentManager) {
    this.consent = consentManager;
    this.sessionId = null;
    this.userId = null;
  }
  
  init() {
    if (this.consent.hasConsent('analytics')) {
      this.#initSession();
      this.#initUserId();
    }
  }
  
  #initSession() {
    let sessionId = Cookies.get('sessionId');
    
    if (!sessionId) {
      sessionId = 'session_' + Date.now() + '_' + Math.random().toString(36).substr(2);
      Cookies.set('sessionId', sessionId, {
        path: '/',
        sameSite: 'lax'
        // No expiry = session cookie
      });
    }
    
    this.sessionId = sessionId;
  }
  
  #initUserId() {
    let userId = Cookies.get('userId');
    
    if (!userId) {
      userId = 'user_' + Math.random().toString(36).substr(2, 12);
      Cookies.set('userId', userId, {
        expires: 365,  // 1 ปี
        path: '/',
        sameSite: 'lax'
      });
    }
    
    this.userId = userId;
  }
  
  track(event, properties = {}) {
    if (!this.consent.hasConsent('analytics')) return;
    
    const eventData = {
      event,
      properties,
      sessionId: this.sessionId,
      userId: this.userId,
      timestamp: new Date().toISOString(),
      url: window.location.href
    };
    
    console.log('Tracking:', eventData);
    // ส่งไปยัง analytics server
  }
}

// Theme Preference Cookie
class ThemeCookieManager {
  constructor() {
    this.cookieName = 'theme';
    this.validThemes = ['light', 'dark', 'system'];
  }
  
  getTheme() {
    const saved = Cookies.get(this.cookieName);
    
    if (saved && this.validThemes.includes(saved)) {
      return saved;
    }
    
    // Default: system preference
    return 'system';
  }
  
  setTheme(theme) {
    if (!this.validThemes.includes(theme)) {
      throw new Error(`Theme ไม่ถูกต้อง: ${theme}`);
    }
    
    Cookies.set(this.cookieName, theme, {
      expires: 365,
      path: '/',
      sameSite: 'lax'
    });
    
    this.#applyTheme(theme);
  }
  
  #applyTheme(theme) {
    let resolvedTheme = theme;
    
    if (theme === 'system') {
      resolvedTheme = window.matchMedia('(prefers-color-scheme: dark)').matches 
        ? 'dark' 
        : 'light';
    }
    
    document.documentElement.setAttribute('data-theme', resolvedTheme);
    console.log('Applied theme:', resolvedTheme);
  }
  
  init() {
    const theme = this.getTheme();
    this.#applyTheme(theme);
    return theme;
  }
}

// ใช้งาน Theme Manager
const themeManager = new ThemeCookieManager();
const currentTheme = themeManager.init();
console.log('Current theme:', currentTheme);
```

---

## แบบฝึกหัด (Exercises)

### แบบฝึกหัดที่ 1: Cookie-Based Authentication

```javascript
// TODO: สร้าง Cookie Authentication system ที่:
// 1. Login: ส่ง credentials ไปยัง server, รับ token, save ใน cookie
// 2. Logout: ลบ auth cookies และ redirect
// 3. Auto-refresh token ก่อนหมดอายุ
// 4. Handle "Remember Me" (30 วัน vs session)
// 5. Detect token tampering

class CookieAuthManager {
  async login(username, password, rememberMe) {
    // TODO
  }
  
  async logout() {
    // TODO
  }
  
  async refreshToken() {
    // TODO
  }
  
  isLoggedIn() {
    // TODO
  }
}
```

### แบบฝึกหัดที่ 2: Cookie Consent Banner

```javascript
// TODO: สร้าง Cookie Consent Banner ที่:
// 1. แสดงเมื่อ user ยังไม่ได้ให้ consent
// 2. มี options: Accept All, Reject All, Customize
// 3. Customize panel แสดง categories แต่ละอัน
// 4. บันทึก consent อย่างถูกต้อง
// 5. รองรับ GDPR requirements

function createConsentBanner() {
  // TODO: สร้าง HTML element
  // TODO: handle user interactions
  // TODO: save consent
}
```

### แบบฝึกหัดที่ 3: Multi-Language Cookie Manager

```javascript
// TODO: สร้าง Language preference manager ที่:
// 1. บันทึกภาษาใน cookie
// 2. Apply language เมื่อ page load
// 3. Sync กับ Accept-Language header (server-side)
// 4. รองรับ RTL languages

class LanguageManager {
  // TODO
}
```

---

## สรุป

Cookies ยังคงเป็นเครื่องมือสำคัญสำหรับ web development:

1. **การตั้งค่า**: ใช้ attributes ให้ครบ (path, expires, secure, sameSite)
2. **Security**: Secure + HttpOnly + SameSite=Strict สำหรับ sensitive data
3. **Privacy**: ขอ consent ก่อนใช้ non-essential cookies (GDPR)
4. **Alternatives**: ใช้ localStorage เมื่อไม่ต้องการ server access
5. **Modern API**: Cookie Store API ให้ async interface ที่ดีกว่า

---

*ต่อไป: Part 45 - Canvas API*
