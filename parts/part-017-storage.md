# Part 17: Local Storage และ Session Storage
## Steps 311-330

---

## บทนำ

Web Storage API ให้กลไกสำหรับเว็บแอพพลิเคชันในการเก็บข้อมูลใน browser ได้สะดวกกว่าการใช้ cookies โดย Web Storage ประกอบด้วย `localStorage` และ `sessionStorage` ซึ่งเก็บข้อมูลในรูป key-value pairs บน client-side

---

## Step 311: Web Storage Overview

### ข้อดีของ Web Storage เหนือ Cookies

| ลักษณะ | localStorage | sessionStorage | Cookie |
|--------|-------------|----------------|--------|
| ความจุ | ~5-10MB | ~5MB | ~4KB |
| อายุข้อมูล | ถาวร | ตาม session | กำหนดได้ |
| ส่งไปกับ Request | ไม่ | ไม่ | ใช่ |
| เข้าถึงจาก JS | ใช่ | ใช่ | ใช่ |
| Scope | Origin | Origin + Tab | Domain |

```javascript
// ตรวจสอบว่า browser รองรับ Web Storage
function isStorageAvailable(type) {
  let storage;
  try {
    storage = window[type];
    const testKey = '__storage_test__';
    storage.setItem(testKey, testKey);
    storage.removeItem(testKey);
    return true;
  } catch (e) {
    return e instanceof DOMException && (
      // Firefox
      e.code === 22 ||
      e.code === 1014 ||
      e.name === 'QuotaExceededError' ||
      e.name === 'NS_ERROR_DOM_QUOTA_REACHED'
    ) && (storage && storage.length !== 0);
  }
}

console.log(isStorageAvailable('localStorage'));   // true/false
console.log(isStorageAvailable('sessionStorage')); // true/false

// วิธีง่ายกว่า
const hasLocalStorage = () => {
  try {
    localStorage.setItem('test', 'test');
    localStorage.removeItem('test');
    return true;
  } catch {
    return false;
  }
};
```

---

## Step 312: localStorage vs sessionStorage

### localStorage

```javascript
// localStorage - ข้อมูลจะคงอยู่แม้ปิด browser
localStorage.setItem('username', 'สมชาย');
localStorage.setItem('theme', 'dark');

// ข้อมูลยังคงอยู่หลัง refresh หรือปิด/เปิด browser
console.log(localStorage.getItem('username')); // "สมชาย"

// เข้าถึงได้จากทุก tab ใน origin เดียวกัน
// Tab 1: localStorage.setItem('shared', 'value')
// Tab 2: localStorage.getItem('shared') // 'value'
```

### sessionStorage

```javascript
// sessionStorage - ข้อมูลจะหายเมื่อปิด tab
sessionStorage.setItem('currentPage', '3');
sessionStorage.setItem('searchQuery', 'javascript tutorial');

// ข้อมูลหายเมื่อปิด tab หรือ browser
console.log(sessionStorage.getItem('currentPage')); // "3"

// แต่ละ tab มี sessionStorage แยกกัน
// Tab 1: sessionStorage.setItem('data', 'tab1')
// Tab 2: sessionStorage.getItem('data') // null (different tab)
```

### เปรียบเทียบการใช้งาน

```javascript
// localStorage ใช้สำหรับ:
// - User preferences (theme, language)
// - Saved login sessions
// - Application state ที่ต้องการระหว่าง sessions

// sessionStorage ใช้สำหรับ:
// - Shopping cart ที่ต้องการเฉพาะใน session นั้น
// - Form data ระหว่างหน้าต่างๆ
// - One-time data ที่ไม่ต้องการเก็บถาวร

class StorageManager {
  constructor(storage) {
    this.storage = storage;
  }
  
  set(key, value) {
    this.storage.setItem(key, JSON.stringify(value));
  }
  
  get(key, defaultValue = null) {
    const item = this.storage.getItem(key);
    return item ? JSON.parse(item) : defaultValue;
  }
  
  remove(key) {
    this.storage.removeItem(key);
  }
  
  clear() {
    this.storage.clear();
  }
}

const local = new StorageManager(localStorage);
const session = new StorageManager(sessionStorage);

local.set('user', { name: 'สมชาย', role: 'admin' });
session.set('cart', [{ id: 1, qty: 2 }]);
```

---

## Step 313: Storage Methods - setItem และ getItem

```javascript
// setItem(key, value) - เก็บข้อมูล
localStorage.setItem('name', 'สมชาย');
localStorage.setItem('age', '25');  // value ต้องเป็น string
localStorage.setItem('active', 'true');

// getItem(key) - อ่านข้อมูล
const name = localStorage.getItem('name');
console.log(name); // "สมชาย"
console.log(typeof name); // "string"

// getItem คืน null ถ้าไม่มี key
const missing = localStorage.getItem('nonexistent');
console.log(missing); // null
console.log(missing === null); // true

// การเข้าถึงแบบ property (ไม่แนะนำ)
localStorage.name = 'สมหญิง'; // ไม่แนะนำ
console.log(localStorage.name); // "สมหญิง"

// ดีกว่า: ใช้ setItem/getItem
localStorage.setItem('name', 'สมหญิง');
console.log(localStorage.getItem('name')); // "สมหญิง"

// Wrapper functions ที่ดีกว่า
const storage = {
  set: (key, value) => {
    try {
      localStorage.setItem(key, JSON.stringify(value));
      return true;
    } catch (e) {
      if (e.name === 'QuotaExceededError') {
        console.error('Storage quota exceeded');
      }
      return false;
    }
  },
  
  get: (key, defaultValue = null) => {
    try {
      const item = localStorage.getItem(key);
      if (item === null) return defaultValue;
      return JSON.parse(item);
    } catch (e) {
      return defaultValue;
    }
  }
};

storage.set('count', 42);
console.log(storage.get('count')); // 42 (number, not string)

storage.set('user', { name: 'สมชาย', age: 25 });
console.log(storage.get('user').name); // "สมชาย"
```

---

## Step 314: Storage Methods - removeItem และ clear

```javascript
// removeItem(key) - ลบข้อมูลตาม key
localStorage.setItem('temp1', 'value1');
localStorage.setItem('temp2', 'value2');
localStorage.setItem('keep', 'keepValue');

localStorage.removeItem('temp1');
console.log(localStorage.getItem('temp1')); // null
console.log(localStorage.getItem('temp2')); // "value2" (ยังอยู่)
console.log(localStorage.getItem('keep'));  // "keepValue" (ยังอยู่)

// clear() - ลบทั้งหมด
localStorage.setItem('a', '1');
localStorage.setItem('b', '2');
localStorage.setItem('c', '3');

console.log(localStorage.length); // 3 (+ keep จากด้านบน)
localStorage.clear();
console.log(localStorage.length); // 0

// Selective clear ตาม prefix
function clearByPrefix(storage, prefix) {
  const keysToRemove = [];
  for (let i = 0; i < storage.length; i++) {
    const key = storage.key(i);
    if (key && key.startsWith(prefix)) {
      keysToRemove.push(key);
    }
  }
  keysToRemove.forEach(key => storage.removeItem(key));
  return keysToRemove.length;
}

localStorage.setItem('cache_user1', 'data1');
localStorage.setItem('cache_user2', 'data2');
localStorage.setItem('settings_theme', 'dark');

const removed = clearByPrefix(localStorage, 'cache_');
console.log(`Removed ${removed} cache entries`); // "Removed 2 cache entries"
console.log(localStorage.getItem('settings_theme')); // "dark" (ยังอยู่)
```

---

## Step 315: Storage length และ key()

```javascript
// length - จำนวน items ที่เก็บอยู่
localStorage.clear();
localStorage.setItem('a', '1');
localStorage.setItem('b', '2');
localStorage.setItem('c', '3');

console.log(localStorage.length); // 3

// key(index) - ได้ key ตาม index
console.log(localStorage.key(0)); // อาจเป็น 'a', 'b', หรือ 'c'
console.log(localStorage.key(1)); // ไม่ guarantee ลำดับ
console.log(localStorage.key(10)); // null (out of range)

// วน loop ดู keys ทั้งหมด
function getAllKeys(storage) {
  const keys = [];
  for (let i = 0; i < storage.length; i++) {
    keys.push(storage.key(i));
  }
  return keys;
}

console.log(getAllKeys(localStorage)); // ['a', 'b', 'c'] (ลำดับไม่ guarantee)

// วน loop ดู key-value ทั้งหมด
function getAllEntries(storage) {
  const entries = {};
  for (let i = 0; i < storage.length; i++) {
    const key = storage.key(i);
    entries[key] = storage.getItem(key);
  }
  return entries;
}

console.log(getAllEntries(localStorage));
// { a: '1', b: '2', c: '3' }

// การใช้ Object.keys (ไม่แนะนำ - อาจรวม built-in methods)
// Object.keys(localStorage).forEach(key => ...)

// ดีกว่า: ใช้ loop แบบข้างต้น

// Iterate ด้วย entries
function* iterateStorage(storage) {
  for (let i = 0; i < storage.length; i++) {
    const key = storage.key(i);
    yield [key, storage.getItem(key)];
  }
}

for (const [key, value] of iterateStorage(localStorage)) {
  console.log(`${key}: ${value}`);
}
```

---

## Step 316: การเก็บ Strings

```javascript
// ทุกค่าใน Web Storage ต้องเป็น string
localStorage.setItem('greeting', 'สวัสดีครับ');
localStorage.setItem('number', '42');        // ต้องแปลงเป็น string
localStorage.setItem('bool', 'true');        // ต้องแปลงเป็น string

// อ่านค่า
const greeting = localStorage.getItem('greeting');
const num = parseInt(localStorage.getItem('number')); // ต้องแปลงกลับ
const bool = localStorage.getItem('bool') === 'true'; // ต้องแปลงกลับ

console.log(typeof greeting); // "string"
console.log(typeof num);      // "number"
console.log(typeof bool);     // "boolean"

// Multiline strings
const multiline = `บรรทัดที่ 1
บรรทัดที่ 2
บรรทัดที่ 3`;
localStorage.setItem('text', multiline);
console.log(localStorage.getItem('text').split('\n').length); // 3

// Special characters
localStorage.setItem('special', '< > & " \' / \\ ');
const special = localStorage.getItem('special');
console.log(special); // "< > & " ' / \ "

// Unicode (Thai)
localStorage.setItem('thai', 'สวัสดีครับ ยินดีต้อนรับ');
console.log(localStorage.getItem('thai')); // "สวัสดีครับ ยินดีต้อนรับ"

// Long strings
const longText = 'x'.repeat(100000);
localStorage.setItem('long', longText);
console.log(localStorage.getItem('long').length); // 100000
```

---

## Step 317: การเก็บ Objects และ Arrays ด้วย JSON

```javascript
// Objects ต้องแปลงเป็น JSON ก่อน
const user = {
  id: 1,
  name: "สมชาย ใจดี",
  email: "somchai@example.com",
  preferences: {
    theme: "dark",
    language: "th",
    fontSize: 16
  }
};

// เก็บ object
localStorage.setItem('currentUser', JSON.stringify(user));

// อ่าน object กลับ
const storedUser = JSON.parse(localStorage.getItem('currentUser'));
console.log(storedUser.name); // "สมชาย ใจดี"
console.log(storedUser.preferences.theme); // "dark"

// Arrays
const recentSearches = ["javascript", "react", "nodejs", "python"];
localStorage.setItem('searches', JSON.stringify(recentSearches));

const searches = JSON.parse(localStorage.getItem('searches'));
console.log(searches.length); // 4
console.log(searches[0]);     // "javascript"

// Helper class
class TypedStorage {
  constructor(storage = localStorage) {
    this.storage = storage;
  }
  
  setString(key, value) {
    this.storage.setItem(key, value);
  }
  
  getString(key, defaultVal = '') {
    return this.storage.getItem(key) || defaultVal;
  }
  
  setNumber(key, value) {
    this.storage.setItem(key, String(value));
  }
  
  getNumber(key, defaultVal = 0) {
    const val = this.storage.getItem(key);
    return val !== null ? Number(val) : defaultVal;
  }
  
  setBoolean(key, value) {
    this.storage.setItem(key, value ? '1' : '0');
  }
  
  getBoolean(key, defaultVal = false) {
    const val = this.storage.getItem(key);
    if (val === null) return defaultVal;
    return val === '1';
  }
  
  setObject(key, obj) {
    this.storage.setItem(key, JSON.stringify(obj));
  }
  
  getObject(key, defaultVal = null) {
    try {
      const val = this.storage.getItem(key);
      return val ? JSON.parse(val) : defaultVal;
    } catch {
      return defaultVal;
    }
  }
  
  remove(key) {
    this.storage.removeItem(key);
  }
}

const ts = new TypedStorage();
ts.setNumber('visitCount', 42);
ts.setBoolean('isDarkMode', true);
ts.setObject('profile', { name: 'สมชาย', level: 5 });

console.log(ts.getNumber('visitCount'));   // 42 (number)
console.log(ts.getBoolean('isDarkMode'));  // true (boolean)
console.log(ts.getObject('profile').name); // "สมชาย"
```

---

## Step 318: Storage Events

```javascript
// Storage event เกิดขึ้นเมื่อ localStorage เปลี่ยนใน tab อื่น
// (ไม่ trigger ใน tab เดียวกับที่ทำการเปลี่ยน)

window.addEventListener('storage', function(event) {
  console.log('Storage changed!');
  console.log('Key:', event.key);        // key ที่เปลี่ยน (null ถ้า clear)
  console.log('Old Value:', event.oldValue); // ค่าเก่า
  console.log('New Value:', event.newValue); // ค่าใหม่
  console.log('URL:', event.url);        // URL ของ tab ที่เปลี่ยน
  console.log('Storage Area:', event.storageArea); // localStorage object
});

// ตัวอย่าง: Sync ระหว่าง tabs
class CrossTabSync {
  constructor(channelName) {
    this.channel = channelName;
    this.listeners = new Map();
    
    window.addEventListener('storage', this.handleStorage.bind(this));
  }
  
  handleStorage(event) {
    if (!event.key || !event.key.startsWith(this.channel)) return;
    
    const messageType = event.key.slice(this.channel.length + 1);
    const data = event.newValue ? JSON.parse(event.newValue) : null;
    
    const listener = this.listeners.get(messageType);
    if (listener) listener(data);
  }
  
  on(eventType, callback) {
    this.listeners.set(eventType, callback);
  }
  
  emit(eventType, data) {
    const key = `${this.channel}_${eventType}`;
    const value = JSON.stringify({ data, timestamp: Date.now() });
    localStorage.setItem(key, value);
    // ลบทันทีเพื่อให้ event trigger ได้ครั้งต่อไป
    setTimeout(() => localStorage.removeItem(key), 100);
  }
}

// Tab 1
const sync = new CrossTabSync('myapp');
sync.on('userLogin', ({ data }) => {
  console.log('User logged in:', data.username);
});

// Tab 2
const sync2 = new CrossTabSync('myapp');
sync2.emit('userLogin', { username: 'สมชาย', role: 'admin' });
// Tab 1 จะได้รับ event นี้
```

---

## Step 319: Storage Limits และ Error Handling

```javascript
// ขนาด localStorage ประมาณ 5-10MB ขึ้นอยู่กับ browser
// แต่ละ origin มี quota ของตัวเอง

// ตรวจสอบ storage ที่ใช้อยู่ (ไม่มี API ตรงๆ แต่ประมาณได้)
function estimateStorageUsed(storage = localStorage) {
  let total = 0;
  for (let i = 0; i < storage.length; i++) {
    const key = storage.key(i);
    const value = storage.getItem(key);
    total += key.length + value.length;
  }
  return total * 2; // UTF-16 = 2 bytes per char
}

// ขนาดที่เหลือโดยประมาณ (ใช้ใน production จริงต้องระวัง)
function estimateStorageRemaining(storage = localStorage) {
  const used = estimateStorageUsed(storage);
  const total = 5 * 1024 * 1024; // 5MB estimate
  return total - used;
}

console.log(`Used: ~${estimateStorageUsed() / 1024} KB`);

// จัดการ QuotaExceededError
function safeSetItem(key, value) {
  try {
    localStorage.setItem(key, value);
    return { success: true };
  } catch (e) {
    if (e.name === 'QuotaExceededError' || e.name === 'NS_ERROR_DOM_QUOTA_REACHED') {
      return { 
        success: false, 
        error: 'storage_full',
        message: 'Storage is full. Please clear some data.' 
      };
    }
    return { success: false, error: 'unknown', message: e.message };
  }
}

// LRU Cache ที่จัดการ storage overflow
class LRUStorage {
  constructor(maxItems = 50) {
    this.maxItems = maxItems;
    this.prefix = 'lru_';
    this.indexKey = 'lru_index';
  }
  
  getIndex() {
    const stored = localStorage.getItem(this.indexKey);
    return stored ? JSON.parse(stored) : [];
  }
  
  saveIndex(index) {
    localStorage.setItem(this.indexKey, JSON.stringify(index));
  }
  
  set(key, value) {
    const fullKey = this.prefix + key;
    let index = this.getIndex();
    
    // ลบออกถ้ามีอยู่แล้ว (จะเพิ่มท้ายสุด)
    index = index.filter(k => k !== key);
    index.push(key);
    
    // ถ้าเกิน limit ลบอันเก่าที่สุด
    while (index.length > this.maxItems) {
      const oldest = index.shift();
      localStorage.removeItem(this.prefix + oldest);
    }
    
    localStorage.setItem(fullKey, JSON.stringify(value));
    this.saveIndex(index);
  }
  
  get(key) {
    const item = localStorage.getItem(this.prefix + key);
    if (!item) return null;
    
    // Move to end (most recently used)
    let index = this.getIndex();
    index = index.filter(k => k !== key);
    index.push(key);
    this.saveIndex(index);
    
    return JSON.parse(item);
  }
}
```

---

## Step 320: Cookies - บทนำและการเปรียบเทียบ

```javascript
// Cookies ทำงานต่างจาก Web Storage
// - ส่งไปกับทุก HTTP request
// - มี expiry date
// - สามารถ access จาก server
// - ขนาดจำกัด ~4KB

// การใช้ Cookie พื้นฐาน
document.cookie = "username=สมชาย";
document.cookie = "theme=dark; expires=Thu, 01 Jan 2025 00:00:00 GMT";
document.cookie = "lang=th; path=/; SameSite=Strict";

// อ่าน Cookie
console.log(document.cookie);
// "username=สมชาย; theme=dark; lang=th"
// (string เดียวที่รวมทุก cookies)

// Cookie Helper
const CookieManager = {
  set(name, value, days = 7, path = '/', sameSite = 'Strict') {
    const expires = new Date(Date.now() + days * 864e5).toUTCString();
    document.cookie = `${encodeURIComponent(name)}=${encodeURIComponent(value)}; expires=${expires}; path=${path}; SameSite=${sameSite}`;
  },
  
  get(name) {
    const cookies = document.cookie.split(';');
    for (const cookie of cookies) {
      const [key, val] = cookie.trim().split('=');
      if (decodeURIComponent(key) === name) {
        return decodeURIComponent(val);
      }
    }
    return null;
  },
  
  remove(name, path = '/') {
    document.cookie = `${encodeURIComponent(name)}=; expires=Thu, 01 Jan 1970 00:00:00 GMT; path=${path}`;
  },
  
  getAll() {
    return document.cookie.split(';').reduce((acc, cookie) => {
      const [key, val] = cookie.trim().split('=');
      if (key) {
        acc[decodeURIComponent(key)] = decodeURIComponent(val || '');
      }
      return acc;
    }, {});
  }
};

// เปรียบเทียบการใช้งาน
// Cookies - ใช้เมื่อต้องการ:
// - Authentication tokens ที่ server ต้องอ่าน
// - Third-party tracking
// - Session management บน server

// localStorage - ใช้เมื่อต้องการ:
// - User preferences
// - Offline data
// - Large data ที่ไม่ต้องส่ง server

// sessionStorage - ใช้เมื่อต้องการ:
// - Per-tab data
// - Temporary form data
```

---

## Step 321: IndexedDB บทนำ

```javascript
// IndexedDB - Database บน browser ที่รองรับ:
// - ข้อมูลขนาดใหญ่ (ร้อย MB ถึง GB)
// - Structured data
// - Indexes
// - Transactions
// - Asynchronous API

// เปิด/สร้าง database
function openDB(name, version, onUpgrade) {
  return new Promise((resolve, reject) => {
    const request = indexedDB.open(name, version);
    
    request.onerror = () => reject(request.error);
    request.onsuccess = () => resolve(request.result);
    
    request.onupgradeneeded = (event) => {
      const db = event.target.result;
      onUpgrade(db, event.oldVersion, event.newVersion);
    };
  });
}

// สร้าง object store
async function initDB() {
  const db = await openDB('myApp', 1, (db) => {
    // สร้าง users store
    if (!db.objectStoreNames.contains('users')) {
      const userStore = db.createObjectStore('users', { keyPath: 'id', autoIncrement: true });
      userStore.createIndex('email', 'email', { unique: true });
      userStore.createIndex('name', 'name', { unique: false });
    }
    
    // สร้าง products store
    if (!db.objectStoreNames.contains('products')) {
      const productStore = db.createObjectStore('products', { keyPath: 'id' });
      productStore.createIndex('category', 'category');
    }
  });
  
  return db;
}

// CRUD operations
class IndexedDBService {
  constructor(db, storeName) {
    this.db = db;
    this.storeName = storeName;
  }
  
  async add(item) {
    return new Promise((resolve, reject) => {
      const tx = this.db.transaction(this.storeName, 'readwrite');
      const store = tx.objectStore(this.storeName);
      const request = store.add(item);
      request.onsuccess = () => resolve(request.result);
      request.onerror = () => reject(request.error);
    });
  }
  
  async get(id) {
    return new Promise((resolve, reject) => {
      const tx = this.db.transaction(this.storeName, 'readonly');
      const store = tx.objectStore(this.storeName);
      const request = store.get(id);
      request.onsuccess = () => resolve(request.result);
      request.onerror = () => reject(request.error);
    });
  }
  
  async getAll() {
    return new Promise((resolve, reject) => {
      const tx = this.db.transaction(this.storeName, 'readonly');
      const store = tx.objectStore(this.storeName);
      const request = store.getAll();
      request.onsuccess = () => resolve(request.result);
      request.onerror = () => reject(request.error);
    });
  }
  
  async update(item) {
    return new Promise((resolve, reject) => {
      const tx = this.db.transaction(this.storeName, 'readwrite');
      const store = tx.objectStore(this.storeName);
      const request = store.put(item);
      request.onsuccess = () => resolve(request.result);
      request.onerror = () => reject(request.error);
    });
  }
  
  async delete(id) {
    return new Promise((resolve, reject) => {
      const tx = this.db.transaction(this.storeName, 'readwrite');
      const store = tx.objectStore(this.storeName);
      const request = store.delete(id);
      request.onsuccess = () => resolve();
      request.onerror = () => reject(request.error);
    });
  }
}
```

---

## Step 322: Cache API บทนำ

```javascript
// Cache API - สำหรับ Service Workers และ Progressive Web Apps
// เก็บ HTTP requests และ responses

// เปิด cache
async function openCache(cacheName) {
  return await caches.open(cacheName);
}

// Cache a response
async function cacheResponse(url) {
  const cache = await caches.open('my-cache-v1');
  try {
    await cache.add(url); // fetch และ cache ในครั้งเดียว
    console.log(`Cached: ${url}`);
  } catch (error) {
    console.error(`Failed to cache ${url}:`, error);
  }
}

// Cache multiple resources
async function cacheResources(urls) {
  const cache = await caches.open('static-v1');
  await cache.addAll(urls);
}

// Get from cache or fetch
async function fetchWithCache(url) {
  const cachedResponse = await caches.match(url);
  
  if (cachedResponse) {
    console.log('Served from cache:', url);
    return cachedResponse;
  }
  
  console.log('Fetching from network:', url);
  const response = await fetch(url);
  
  if (response.ok) {
    const cache = await caches.open('dynamic-v1');
    cache.put(url, response.clone());
  }
  
  return response;
}

// Service Worker ตัวอย่าง
// (ต้องอยู่ใน service worker file)
/*
self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open('static-v1').then((cache) => {
      return cache.addAll([
        '/',
        '/index.html',
        '/styles.css',
        '/app.js'
      ]);
    })
  );
});

self.addEventListener('fetch', (event) => {
  event.respondWith(
    caches.match(event.request).then((response) => {
      return response || fetch(event.request);
    })
  );
});
*/
```

---

## Step 323: Shopping Cart ด้วย localStorage

```javascript
class ShoppingCart {
  constructor() {
    this.storageKey = 'shopping_cart';
    this.items = this.load();
  }
  
  load() {
    try {
      const saved = localStorage.getItem(this.storageKey);
      return saved ? JSON.parse(saved) : [];
    } catch {
      return [];
    }
  }
  
  save() {
    localStorage.setItem(this.storageKey, JSON.stringify(this.items));
  }
  
  addItem(product) {
    const existing = this.items.find(item => item.id === product.id);
    
    if (existing) {
      existing.quantity += product.quantity || 1;
    } else {
      this.items.push({
        id: product.id,
        name: product.name,
        price: product.price,
        image: product.image || '',
        quantity: product.quantity || 1
      });
    }
    
    this.save();
    return this;
  }
  
  removeItem(productId) {
    this.items = this.items.filter(item => item.id !== productId);
    this.save();
    return this;
  }
  
  updateQuantity(productId, quantity) {
    const item = this.items.find(i => i.id === productId);
    if (item) {
      if (quantity <= 0) {
        return this.removeItem(productId);
      }
      item.quantity = quantity;
      this.save();
    }
    return this;
  }
  
  getTotal() {
    return this.items.reduce((total, item) => {
      return total + (item.price * item.quantity);
    }, 0);
  }
  
  getItemCount() {
    return this.items.reduce((count, item) => count + item.quantity, 0);
  }
  
  clear() {
    this.items = [];
    localStorage.removeItem(this.storageKey);
    return this;
  }
  
  toSummary() {
    return {
      items: this.items,
      itemCount: this.getItemCount(),
      total: this.getTotal(),
      updatedAt: new Date().toISOString()
    };
  }
}

// ใช้งาน
const cart = new ShoppingCart();

cart.addItem({ id: 'p1', name: 'แล็ปท็อป', price: 25000 });
cart.addItem({ id: 'p2', name: 'เมาส์', price: 500, quantity: 2 });
cart.addItem({ id: 'p1', name: 'แล็ปท็อป', price: 25000 }); // เพิ่ม qty

console.log(cart.getItemCount()); // 4
console.log(cart.getTotal());     // 26000 (25000 + 500*2)
console.log(cart.toSummary());
```

---

## Step 324: User Preferences Manager

```javascript
class UserPreferences {
  constructor(userId = 'default') {
    this.key = `prefs_${userId}`;
    this.defaults = {
      theme: 'light',
      language: 'th',
      fontSize: 16,
      notifications: {
        email: true,
        push: false,
        sms: false
      },
      layout: {
        sidebar: true,
        compact: false,
        density: 'comfortable'
      },
      accessibility: {
        highContrast: false,
        reduceMotion: false,
        screenReader: false
      }
    };
    this.prefs = this.load();
  }
  
  load() {
    try {
      const saved = localStorage.getItem(this.key);
      if (!saved) return { ...this.defaults };
      
      // Deep merge with defaults (in case new defaults are added)
      return this.deepMerge(this.defaults, JSON.parse(saved));
    } catch {
      return { ...this.defaults };
    }
  }
  
  deepMerge(target, source) {
    const result = { ...target };
    for (const key in source) {
      if (source[key] && typeof source[key] === 'object' && !Array.isArray(source[key])) {
        result[key] = this.deepMerge(target[key] || {}, source[key]);
      } else {
        result[key] = source[key];
      }
    }
    return result;
  }
  
  get(path, defaultValue) {
    const value = path.split('.').reduce((obj, key) => obj?.[key], this.prefs);
    return value !== undefined ? value : defaultValue;
  }
  
  set(path, value) {
    const keys = path.split('.');
    let current = this.prefs;
    
    for (let i = 0; i < keys.length - 1; i++) {
      if (!current[keys[i]]) current[keys[i]] = {};
      current = current[keys[i]];
    }
    
    current[keys[keys.length - 1]] = value;
    this.save();
    this.applyPreferences();
    return this;
  }
  
  save() {
    localStorage.setItem(this.key, JSON.stringify(this.prefs));
  }
  
  reset(path) {
    if (path) {
      const defaultValue = path.split('.').reduce((obj, key) => obj?.[key], this.defaults);
      this.set(path, defaultValue);
    } else {
      this.prefs = { ...this.defaults };
      this.save();
      this.applyPreferences();
    }
  }
  
  applyPreferences() {
    // Apply to DOM
    document.documentElement.setAttribute('data-theme', this.get('theme'));
    document.documentElement.setAttribute('data-lang', this.get('language'));
    document.documentElement.style.fontSize = `${this.get('fontSize')}px`;
    
    if (this.get('accessibility.highContrast')) {
      document.body.classList.add('high-contrast');
    } else {
      document.body.classList.remove('high-contrast');
    }
    
    if (this.get('accessibility.reduceMotion')) {
      document.body.classList.add('reduce-motion');
    } else {
      document.body.classList.remove('reduce-motion');
    }
  }
  
  export() {
    return JSON.stringify(this.prefs, null, 2);
  }
  
  import(jsonString) {
    try {
      const imported = JSON.parse(jsonString);
      this.prefs = this.deepMerge(this.defaults, imported);
      this.save();
      this.applyPreferences();
      return true;
    } catch {
      return false;
    }
  }
}

// ใช้งาน
const prefs = new UserPreferences('user_123');
prefs.set('theme', 'dark');
prefs.set('notifications.email', false);
prefs.set('layout.compact', true);

console.log(prefs.get('theme'));              // "dark"
console.log(prefs.get('notifications.email')); // false
console.log(prefs.get('language'));           // "th" (from defaults)
```

---

## Step 325: Form Data Persistence

```javascript
class FormPersist {
  constructor(formId, options = {}) {
    this.formId = formId;
    this.key = `form_${formId}`;
    this.debounceTime = options.debounceTime || 500;
    this.excludeFields = options.excludeFields || ['password', 'confirm-password'];
    this.storage = options.storage || sessionStorage;
    
    this.debounceTimer = null;
    this.form = document.getElementById(formId);
    
    if (this.form) {
      this.init();
    }
  }
  
  init() {
    // โหลดข้อมูลเก่า
    this.restore();
    
    // ฟัง input events
    this.form.addEventListener('input', this.handleInput.bind(this));
    this.form.addEventListener('change', this.handleInput.bind(this));
    
    // ลบข้อมูลเมื่อ submit
    this.form.addEventListener('submit', this.clear.bind(this));
  }
  
  handleInput() {
    clearTimeout(this.debounceTimer);
    this.debounceTimer = setTimeout(() => this.save(), this.debounceTime);
  }
  
  getData() {
    const data = {};
    const inputs = this.form.querySelectorAll('input, textarea, select');
    
    inputs.forEach(input => {
      const name = input.name || input.id;
      if (!name || this.excludeFields.includes(name)) return;
      
      if (input.type === 'checkbox') {
        data[name] = input.checked;
      } else if (input.type === 'radio') {
        if (input.checked) data[name] = input.value;
      } else {
        data[name] = input.value;
      }
    });
    
    return data;
  }
  
  save() {
    const data = this.getData();
    this.storage.setItem(this.key, JSON.stringify({
      data,
      savedAt: Date.now()
    }));
  }
  
  restore() {
    try {
      const saved = this.storage.getItem(this.key);
      if (!saved) return;
      
      const { data, savedAt } = JSON.parse(saved);
      
      // ข้ามถ้าข้อมูลเก่าเกิน 24 ชั่วโมง (สำหรับ sessionStorage)
      const ageHours = (Date.now() - savedAt) / (1000 * 60 * 60);
      if (ageHours > 24) {
        this.clear();
        return;
      }
      
      Object.entries(data).forEach(([name, value]) => {
        const input = this.form.querySelector(`[name="${name}"], #${name}`);
        if (!input) return;
        
        if (input.type === 'checkbox') {
          input.checked = value;
        } else if (input.type === 'radio') {
          const radio = this.form.querySelector(`[name="${name}"][value="${value}"]`);
          if (radio) radio.checked = true;
        } else {
          input.value = value;
        }
      });
      
      console.log('Form data restored from', ageHours.toFixed(1), 'hours ago');
    } catch (e) {
      console.error('Failed to restore form:', e);
    }
  }
  
  clear() {
    this.storage.removeItem(this.key);
  }
  
  hasSavedData() {
    return this.storage.getItem(this.key) !== null;
  }
}

// ใช้งาน
// const persist = new FormPersist('registration-form', {
//   excludeFields: ['password', 'credit-card'],
//   debounceTime: 1000
// });
```

---

## Step 326: Building a Notes App ด้วย localStorage

```html
<!DOCTYPE html>
<html lang="th">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>โน้ตของฉัน</title>
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; }
    body { font-family: 'Sarabun', Arial, sans-serif; background: #f5f5f5; }
    .container { max-width: 1200px; margin: 0 auto; padding: 20px; }
    header { background: #2196F3; color: white; padding: 20px; margin-bottom: 20px; border-radius: 8px; }
    .toolbar { display: flex; gap: 10px; margin-bottom: 20px; }
    .toolbar input { flex: 1; padding: 10px; border: 1px solid #ddd; border-radius: 4px; font-size: 16px; }
    .btn { padding: 10px 20px; border: none; border-radius: 4px; cursor: pointer; font-size: 14px; }
    .btn-primary { background: #2196F3; color: white; }
    .btn-danger { background: #f44336; color: white; }
    .notes-grid { display: grid; grid-template-columns: repeat(auto-fill, minmax(250px, 1fr)); gap: 16px; }
    .note-card { background: white; border-radius: 8px; padding: 16px; box-shadow: 0 2px 4px rgba(0,0,0,0.1); }
    .note-card h3 { margin-bottom: 8px; color: #333; }
    .note-card p { color: #666; font-size: 14px; line-height: 1.5; max-height: 100px; overflow: hidden; }
    .note-card .meta { font-size: 12px; color: #999; margin-top: 8px; }
    .note-card .actions { display: flex; gap: 8px; margin-top: 12px; }
    .modal { display: none; position: fixed; inset: 0; background: rgba(0,0,0,0.5); z-index: 1000; }
    .modal.show { display: flex; align-items: center; justify-content: center; }
    .modal-content { background: white; padding: 24px; border-radius: 8px; width: 90%; max-width: 500px; }
    .modal-content input, .modal-content textarea { 
      width: 100%; padding: 10px; margin-bottom: 12px; 
      border: 1px solid #ddd; border-radius: 4px; font-size: 16px; 
    }
    .modal-content textarea { height: 150px; resize: vertical; }
    .modal-actions { display: flex; gap: 8px; justify-content: flex-end; }
    .empty-state { text-align: center; padding: 60px; color: #999; }
  </style>
</head>
<body>
  <div class="container">
    <header>
      <h1>📝 โน้ตของฉัน</h1>
      <p id="noteCount">0 โน้ต</p>
    </header>
    
    <div class="toolbar">
      <input type="text" id="searchInput" placeholder="ค้นหาโน้ต..." />
      <button class="btn btn-primary" onclick="openNewNoteModal()">+ โน้ตใหม่</button>
      <button class="btn btn-danger" onclick="clearAllNotes()">ลบทั้งหมด</button>
    </div>
    
    <div class="notes-grid" id="notesGrid"></div>
  </div>
  
  <!-- Modal -->
  <div class="modal" id="noteModal">
    <div class="modal-content">
      <h2 id="modalTitle">โน้ตใหม่</h2>
      <input type="text" id="noteTitle" placeholder="หัวข้อโน้ต" />
      <textarea id="noteContent" placeholder="เนื้อหาโน้ต..."></textarea>
      <div class="modal-actions">
        <button class="btn" onclick="closeModal()">ยกเลิก</button>
        <button class="btn btn-primary" onclick="saveNote()">บันทึก</button>
      </div>
    </div>
  </div>

  <script>
    class NotesApp {
      constructor() {
        this.key = 'my_notes';
        this.notes = this.loadNotes();
        this.editingId = null;
        this.searchQuery = '';
        
        document.getElementById('searchInput').addEventListener('input', (e) => {
          this.searchQuery = e.target.value.toLowerCase();
          this.render();
        });
        
        this.render();
      }
      
      loadNotes() {
        try {
          const stored = localStorage.getItem(this.key);
          return stored ? JSON.parse(stored) : [];
        } catch {
          return [];
        }
      }
      
      saveNotes() {
        localStorage.setItem(this.key, JSON.stringify(this.notes));
      }
      
      addNote(title, content) {
        const note = {
          id: Date.now().toString(),
          title: title.trim() || 'ไม่มีหัวข้อ',
          content: content.trim(),
          createdAt: new Date().toISOString(),
          updatedAt: new Date().toISOString()
        };
        this.notes.unshift(note);
        this.saveNotes();
        this.render();
        return note;
      }
      
      updateNote(id, title, content) {
        const note = this.notes.find(n => n.id === id);
        if (note) {
          note.title = title.trim() || 'ไม่มีหัวข้อ';
          note.content = content.trim();
          note.updatedAt = new Date().toISOString();
          this.saveNotes();
          this.render();
        }
      }
      
      deleteNote(id) {
        if (confirm('ต้องการลบโน้ตนี้หรือไม่?')) {
          this.notes = this.notes.filter(n => n.id !== id);
          this.saveNotes();
          this.render();
        }
      }
      
      clearAll() {
        if (confirm('ต้องการลบโน้ตทั้งหมดหรือไม่?')) {
          this.notes = [];
          localStorage.removeItem(this.key);
          this.render();
        }
      }
      
      getFilteredNotes() {
        if (!this.searchQuery) return this.notes;
        return this.notes.filter(n => 
          n.title.toLowerCase().includes(this.searchQuery) ||
          n.content.toLowerCase().includes(this.searchQuery)
        );
      }
      
      formatDate(isoString) {
        const date = new Date(isoString);
        return date.toLocaleDateString('th-TH', {
          year: 'numeric', month: 'short', day: 'numeric',
          hour: '2-digit', minute: '2-digit'
        });
      }
      
      render() {
        const grid = document.getElementById('notesGrid');
        const filtered = this.getFilteredNotes();
        
        document.getElementById('noteCount').textContent = `${this.notes.length} โน้ต`;
        
        if (filtered.length === 0) {
          grid.innerHTML = `
            <div class="empty-state" style="grid-column: 1/-1">
              ${this.searchQuery ? 'ไม่พบโน้ตที่ค้นหา' : 'ยังไม่มีโน้ต คลิก "โน้ตใหม่" เพื่อเริ่มต้น'}
            </div>`;
          return;
        }
        
        grid.innerHTML = filtered.map(note => `
          <div class="note-card">
            <h3>${this.escapeHtml(note.title)}</h3>
            <p>${this.escapeHtml(note.content) || '<em>ไม่มีเนื้อหา</em>'}</p>
            <div class="meta">
              แก้ไขล่าสุด: ${this.formatDate(note.updatedAt)}
            </div>
            <div class="actions">
              <button class="btn btn-primary" onclick="app.editNote('${note.id}')">แก้ไข</button>
              <button class="btn btn-danger" onclick="app.deleteNote('${note.id}')">ลบ</button>
            </div>
          </div>
        `).join('');
      }
      
      escapeHtml(text) {
        const div = document.createElement('div');
        div.appendChild(document.createTextNode(text));
        return div.innerHTML;
      }
      
      openEditModal(id) {
        const note = this.notes.find(n => n.id === id);
        if (!note) return;
        
        this.editingId = id;
        document.getElementById('modalTitle').textContent = 'แก้ไขโน้ต';
        document.getElementById('noteTitle').value = note.title;
        document.getElementById('noteContent').value = note.content;
        document.getElementById('noteModal').classList.add('show');
      }
    }
    
    const app = new NotesApp();
    
    function openNewNoteModal() {
      app.editingId = null;
      document.getElementById('modalTitle').textContent = 'โน้ตใหม่';
      document.getElementById('noteTitle').value = '';
      document.getElementById('noteContent').value = '';
      document.getElementById('noteModal').classList.add('show');
      document.getElementById('noteTitle').focus();
    }
    
    function closeModal() {
      document.getElementById('noteModal').classList.remove('show');
      app.editingId = null;
    }
    
    function saveNote() {
      const title = document.getElementById('noteTitle').value;
      const content = document.getElementById('noteContent').value;
      
      if (!title.trim() && !content.trim()) {
        alert('กรุณาใส่หัวข้อหรือเนื้อหา');
        return;
      }
      
      if (app.editingId) {
        app.updateNote(app.editingId, title, content);
      } else {
        app.addNote(title, content);
      }
      
      closeModal();
    }
    
    function clearAllNotes() {
      app.clearAll();
    }
    
    // Close modal on backdrop click
    document.getElementById('noteModal').addEventListener('click', function(e) {
      if (e.target === this) closeModal();
    });
    
    // Keyboard shortcuts
    document.addEventListener('keydown', (e) => {
      if (e.key === 'Escape') closeModal();
      if (e.ctrlKey && e.key === 'n') {
        e.preventDefault();
        openNewNoteModal();
      }
    });
  </script>
</body>
</html>
```

---

## Step 327: Token และ Auth State Management

```javascript
class AuthStorage {
  constructor() {
    this.TOKEN_KEY = 'auth_token';
    this.USER_KEY = 'auth_user';
    this.EXPIRY_KEY = 'auth_expiry';
  }
  
  setAuth(token, user, expiryMinutes = 60) {
    const expiry = new Date(Date.now() + expiryMinutes * 60 * 1000).toISOString();
    
    localStorage.setItem(this.TOKEN_KEY, token);
    localStorage.setItem(this.USER_KEY, JSON.stringify(user));
    localStorage.setItem(this.EXPIRY_KEY, expiry);
  }
  
  getToken() {
    if (!this.isValid()) return null;
    return localStorage.getItem(this.TOKEN_KEY);
  }
  
  getUser() {
    if (!this.isValid()) return null;
    try {
      return JSON.parse(localStorage.getItem(this.USER_KEY));
    } catch {
      return null;
    }
  }
  
  isValid() {
    const token = localStorage.getItem(this.TOKEN_KEY);
    const expiry = localStorage.getItem(this.EXPIRY_KEY);
    
    if (!token || !expiry) return false;
    
    const expiryDate = new Date(expiry);
    return expiryDate > new Date();
  }
  
  isLoggedIn() {
    return this.isValid();
  }
  
  clearAuth() {
    localStorage.removeItem(this.TOKEN_KEY);
    localStorage.removeItem(this.USER_KEY);
    localStorage.removeItem(this.EXPIRY_KEY);
  }
  
  getTimeUntilExpiry() {
    const expiry = localStorage.getItem(this.EXPIRY_KEY);
    if (!expiry) return 0;
    return Math.max(0, new Date(expiry) - new Date());
  }
  
  extendSession(minutes = 30) {
    const newExpiry = new Date(Date.now() + minutes * 60 * 1000).toISOString();
    localStorage.setItem(this.EXPIRY_KEY, newExpiry);
  }
}

// Session timeout warning
class SessionManager {
  constructor(authStorage, warningMinutes = 5) {
    this.auth = authStorage;
    this.warningMinutes = warningMinutes;
    this.warningTimer = null;
    this.expiryTimer = null;
  }
  
  start() {
    this.scheduleTimers();
  }
  
  scheduleTimers() {
    this.clearTimers();
    
    const timeLeft = this.auth.getTimeUntilExpiry();
    const warningTime = timeLeft - (this.warningMinutes * 60 * 1000);
    
    if (warningTime > 0) {
      this.warningTimer = setTimeout(() => {
        this.onWarning();
      }, warningTime);
    }
    
    if (timeLeft > 0) {
      this.expiryTimer = setTimeout(() => {
        this.onExpiry();
      }, timeLeft);
    }
  }
  
  onWarning() {
    const minutes = this.auth.getTimeUntilExpiry() / 60000;
    console.log(`Session จะหมดอายุใน ${minutes.toFixed(0)} นาที`);
    // Show dialog to extend session
  }
  
  onExpiry() {
    console.log('Session หมดอายุแล้ว');
    this.auth.clearAuth();
    window.location.href = '/login';
  }
  
  extendSession() {
    this.auth.extendSession(30);
    this.scheduleTimers();
  }
  
  clearTimers() {
    clearTimeout(this.warningTimer);
    clearTimeout(this.expiryTimer);
  }
}
```

---

## Step 328-330: Storage Utilities และ Best Practices

```javascript
// Versioned Storage - จัดการ migration เมื่อ data format เปลี่ยน
class VersionedStorage {
  constructor(key, currentVersion, migrations = {}) {
    this.key = key;
    this.currentVersion = currentVersion;
    this.migrations = migrations;
  }
  
  load(defaultData) {
    try {
      const stored = localStorage.getItem(this.key);
      if (!stored) return { version: this.currentVersion, data: defaultData };
      
      const parsed = JSON.parse(stored);
      return this.migrate(parsed);
    } catch {
      return { version: this.currentVersion, data: defaultData };
    }
  }
  
  migrate(stored) {
    let { version, data } = stored;
    
    while (version < this.currentVersion) {
      const migration = this.migrations[version];
      if (migration) {
        data = migration(data);
        console.log(`Migrated from v${version} to v${version + 1}`);
      }
      version++;
    }
    
    return { version: this.currentVersion, data };
  }
  
  save(data) {
    localStorage.setItem(this.key, JSON.stringify({
      version: this.currentVersion,
      data,
      savedAt: Date.now()
    }));
  }
}

// ตัวอย่าง migration
const appStorage = new VersionedStorage('app_data', 3, {
  1: (data) => ({ ...data, displayName: data.name }), // v1 -> v2: add displayName
  2: (data) => ({ ...data, preferences: { theme: 'light' } }) // v2 -> v3: add preferences
});

// Storage Quota Monitor
class StorageMonitor {
  constructor() {
    this.warningThreshold = 0.8; // 80%
  }
  
  async checkQuota() {
    if (navigator.storage && navigator.storage.estimate) {
      const estimate = await navigator.storage.estimate();
      const usage = estimate.usage;
      const quota = estimate.quota;
      const percentage = usage / quota;
      
      console.log(`Storage: ${(usage / 1024 / 1024).toFixed(2)} MB / ${(quota / 1024 / 1024).toFixed(2)} MB (${(percentage * 100).toFixed(1)}%)`);
      
      if (percentage >= this.warningThreshold) {
        this.onWarning(percentage);
      }
      
      return { usage, quota, percentage };
    }
    
    // Fallback estimate
    return this.estimateLocalStorage();
  }
  
  estimateLocalStorage() {
    let total = 0;
    for (const key of Object.keys(localStorage)) {
      total += localStorage.getItem(key).length * 2;
    }
    return {
      usage: total,
      quota: 5 * 1024 * 1024,
      percentage: total / (5 * 1024 * 1024)
    };
  }
  
  onWarning(percentage) {
    console.warn(`Storage usage is at ${(percentage * 100).toFixed(1)}% - consider cleaning up`);
  }
}

// Cleanup strategies
class StorageCleaner {
  // ลบข้อมูลที่หมดอายุ
  cleanExpired(storage = localStorage) {
    const toRemove = [];
    const now = Date.now();
    
    for (let i = 0; i < storage.length; i++) {
      const key = storage.key(i);
      try {
        const item = JSON.parse(storage.getItem(key));
        if (item && item.expiresAt && item.expiresAt < now) {
          toRemove.push(key);
        }
      } catch {}
    }
    
    toRemove.forEach(key => storage.removeItem(key));
    return toRemove.length;
  }
  
  // ลบข้อมูลเก่ากว่า N วัน
  cleanOlderThan(days, storage = localStorage) {
    const cutoff = Date.now() - (days * 24 * 60 * 60 * 1000);
    const toRemove = [];
    
    for (let i = 0; i < storage.length; i++) {
      const key = storage.key(i);
      try {
        const item = JSON.parse(storage.getItem(key));
        if (item && item.savedAt && item.savedAt < cutoff) {
          toRemove.push(key);
        }
      } catch {}
    }
    
    toRemove.forEach(key => storage.removeItem(key));
    return toRemove.length;
  }
}
```

---

## แบบฝึกหัด (Exercises)

### แบบฝึกหัดที่ 1: Persistent Counter
สร้างแอพ counter ที่จำค่าไว้แม้ refresh โดยใช้ localStorage มี +, -, reset และแสดงประวัติการกด

### แบบฝึกหัดที่ 2: Theme Switcher
สร้างระบบ theme switcher (light/dark/auto) ที่บันทึกการตั้งค่าลง localStorage และ apply CSS variables

### แบบฝึกหัดที่ 3: Multi-step Form
สร้าง form หลายหน้าที่บันทึก progress ลง sessionStorage เพื่อ resume ได้เมื่อ navigate กลับ

### แบบฝึกหัดที่ 4: Read Later List
สร้าง "อ่านทีหลัง" list ที่บันทึก URL, title, และ timestamp ลง localStorage พร้อม search และ filter

### แบบฝึกหัดที่ 5: Offline Score Board
สร้าง leaderboard ที่เก็บ scores ของเกมง่ายๆ ลง localStorage ด้วย top 10 entries

---

## สรุป Part 17

ในบทนี้เราเรียนรู้:
- ความแตกต่างระหว่าง localStorage, sessionStorage และ Cookies
- CRUD operations กับ Web Storage (setItem, getItem, removeItem, clear)
- การเก็บ objects และ arrays ด้วย JSON
- Storage events สำหรับ cross-tab communication
- การจัดการ quota errors
- IndexedDB และ Cache API เบื้องต้น
- การสร้าง Shopping Cart, User Preferences, Form Persistence
- Notes App ที่สมบูรณ์ด้วย localStorage
- Best practices และ security considerations
