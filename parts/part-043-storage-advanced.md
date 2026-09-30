# Part 43: Web Storage ขั้นสูง (Steps 831-850)

## บทนำ

Web Storage APIs ให้วิธีการจัดเก็บข้อมูลบน client-side ที่หลากหลาย ตั้งแต่ localStorage ที่เรียบง่าย ไปจนถึง IndexedDB ที่ทรงพลัง และ Cache API สำหรับ offline applications

ในบทนี้เราจะเรียนรู้:
- localStorage/sessionStorage patterns ขั้นสูง
- StorageManager API
- IndexedDB สำหรับข้อมูลซับซ้อน
- Cache API สำหรับ Service Workers
- Caching Strategies

---

## Step 831: localStorage Advanced Patterns

```javascript
// localStorage พื้นฐาน
localStorage.setItem('key', 'value');
const value = localStorage.getItem('key');
localStorage.removeItem('key');
localStorage.clear();
console.log(localStorage.length); // จำนวน items

// ปัญหา: localStorage เก็บได้แค่ string
// ต้องแปลงด้วย JSON
localStorage.setItem('user', JSON.stringify({ name: 'สมชาย', age: 30 }));
const user = JSON.parse(localStorage.getItem('user'));

// localStorage Wrapper class พื้นฐาน
class LocalStorage {
  static set(key, value) {
    try {
      localStorage.setItem(key, JSON.stringify(value));
      return true;
    } catch (error) {
      // QuotaExceededError
      console.error('localStorage เต็ม:', error.message);
      return false;
    }
  }
  
  static get(key, defaultValue = null) {
    try {
      const item = localStorage.getItem(key);
      if (item === null) return defaultValue;
      return JSON.parse(item);
    } catch (error) {
      console.error('Parse error:', error.message);
      return defaultValue;
    }
  }
  
  static remove(key) {
    localStorage.removeItem(key);
  }
  
  static clear() {
    localStorage.clear();
  }
  
  static has(key) {
    return localStorage.getItem(key) !== null;
  }
  
  static keys() {
    return Object.keys(localStorage);
  }
  
  static getAll() {
    const result = {};
    for (let i = 0; i < localStorage.length; i++) {
      const key = localStorage.key(i);
      result[key] = LocalStorage.get(key);
    }
    return result;
  }
  
  static size() {
    return localStorage.length;
  }
}

// ใช้งาน
LocalStorage.set('settings', { theme: 'dark', language: 'th' });
const settings = LocalStorage.get('settings', { theme: 'light', language: 'en' });
console.log(settings); // { theme: 'dark', language: 'th' }
```

---

## Step 832: localStorage Wrapper พร้อม Expiry

```javascript
// localStorage พร้อมการหมดอายุ
class ExpiringStorage {
  constructor(prefix = 'app_', defaultTTL = 3600000) { // 1 hour default
    this.prefix = prefix;
    this.defaultTTL = defaultTTL;
  }
  
  #key(key) {
    return `${this.prefix}${key}`;
  }
  
  set(key, value, ttl = this.defaultTTL) {
    const item = {
      value,
      expires: ttl ? Date.now() + ttl : null,
      created: Date.now()
    };
    
    try {
      localStorage.setItem(this.#key(key), JSON.stringify(item));
      return true;
    } catch (error) {
      if (error.name === 'QuotaExceededError') {
        this.#evictExpired();
        try {
          localStorage.setItem(this.#key(key), JSON.stringify(item));
          return true;
        } catch {
          console.error('Storage เต็ม ไม่สามารถบันทึกได้');
          return false;
        }
      }
      return false;
    }
  }
  
  get(key, defaultValue = null) {
    try {
      const raw = localStorage.getItem(this.#key(key));
      if (!raw) return defaultValue;
      
      const item = JSON.parse(raw);
      
      // ตรวจสอบการหมดอายุ
      if (item.expires && Date.now() > item.expires) {
        this.remove(key);
        return defaultValue;
      }
      
      return item.value;
    } catch (error) {
      return defaultValue;
    }
  }
  
  // ดึงพร้อมข้อมูล metadata
  getWithMeta(key) {
    try {
      const raw = localStorage.getItem(this.#key(key));
      if (!raw) return null;
      
      const item = JSON.parse(raw);
      
      if (item.expires && Date.now() > item.expires) {
        this.remove(key);
        return null;
      }
      
      return {
        value: item.value,
        expires: item.expires ? new Date(item.expires) : null,
        created: new Date(item.created),
        ttl: item.expires ? item.expires - Date.now() : null,
        isExpiringSoon: item.expires ? (item.expires - Date.now()) < 60000 : false
      };
    } catch {
      return null;
    }
  }
  
  // ต่ออายุ
  refresh(key, newTTL = this.defaultTTL) {
    const item = this.getWithMeta(key);
    if (!item) return false;
    return this.set(key, item.value, newTTL);
  }
  
  remove(key) {
    localStorage.removeItem(this.#key(key));
  }
  
  has(key) {
    return this.get(key) !== null;
  }
  
  // ลบ items ที่หมดอายุ
  #evictExpired() {
    const now = Date.now();
    const keysToRemove = [];
    
    for (let i = 0; i < localStorage.length; i++) {
      const storageKey = localStorage.key(i);
      if (!storageKey.startsWith(this.prefix)) continue;
      
      try {
        const item = JSON.parse(localStorage.getItem(storageKey));
        if (item.expires && now > item.expires) {
          keysToRemove.push(storageKey);
        }
      } catch {}
    }
    
    keysToRemove.forEach(key => localStorage.removeItem(key));
    console.log(`ลบ ${keysToRemove.length} expired items`);
  }
  
  // ล้าง expired items ทั้งหมด
  cleanup() {
    this.#evictExpired();
  }
  
  // ดู keys ทั้งหมดที่ยังไม่หมดอายุ
  keys() {
    const keys = [];
    for (let i = 0; i < localStorage.length; i++) {
      const storageKey = localStorage.key(i);
      if (!storageKey.startsWith(this.prefix)) continue;
      
      try {
        const item = JSON.parse(localStorage.getItem(storageKey));
        if (!item.expires || Date.now() <= item.expires) {
          keys.push(storageKey.slice(this.prefix.length));
        }
      } catch {}
    }
    return keys;
  }
}

// ตัวอย่างการใช้งาน
const cache = new ExpiringStorage('cache_', 5 * 60 * 1000); // 5 minutes

// บันทึก API response
async function fetchWithCache(url) {
  const cacheKey = btoa(url);
  
  // ตรวจสอบ cache ก่อน
  const cached = cache.get(cacheKey);
  if (cached) {
    console.log('ใช้ข้อมูล cache');
    return cached;
  }
  
  // ดึงข้อมูลใหม่
  const response = await fetch(url);
  const data = await response.json();
  
  // บันทึกลง cache
  cache.set(cacheKey, data);
  console.log('บันทึกลง cache');
  
  return data;
}
```

---

## Step 833: localStorage กับ Versioning

```javascript
// localStorage พร้อม versioning - รองรับการ migrate ข้อมูล
class VersionedStorage {
  constructor(namespace, version, migrations = {}) {
    this.namespace = namespace;
    this.version = version;
    this.migrations = migrations;
    
    this.#initialize();
  }
  
  #initialize() {
    const currentVersion = this.#getVersion();
    
    if (currentVersion === null) {
      // Fresh install
      this.#setVersion(this.version);
    } else if (currentVersion !== this.version) {
      // Version changed - migrate
      this.#migrate(currentVersion, this.version);
    }
  }
  
  #getVersion() {
    const raw = localStorage.getItem(`${this.namespace}_version`);
    return raw ? parseInt(raw) : null;
  }
  
  #setVersion(version) {
    localStorage.setItem(`${this.namespace}_version`, version.toString());
  }
  
  #migrate(fromVersion, toVersion) {
    console.log(`Migration: v${fromVersion} -> v${toVersion}`);
    
    // Run migrations ตาม order
    let currentVersion = fromVersion;
    
    while (currentVersion < toVersion) {
      const nextVersion = currentVersion + 1;
      const migrationFn = this.migrations[nextVersion];
      
      if (migrationFn) {
        try {
          const allData = this.#getAllRaw();
          const migratedData = migrationFn(allData);
          
          // บันทึก migrated data
          Object.entries(migratedData).forEach(([key, value]) => {
            this.set(key, value);
          });
          
          console.log(`Migration v${currentVersion} -> v${nextVersion} สำเร็จ`);
        } catch (error) {
          console.error(`Migration v${nextVersion} ล้มเหลว:`, error.message);
          // Rollback
          break;
        }
      }
      
      currentVersion = nextVersion;
    }
    
    this.#setVersion(toVersion);
  }
  
  #getAllRaw() {
    const data = {};
    const prefix = `${this.namespace}_data_`;
    
    for (let i = 0; i < localStorage.length; i++) {
      const key = localStorage.key(i);
      if (key.startsWith(prefix)) {
        const shortKey = key.slice(prefix.length);
        try {
          data[shortKey] = JSON.parse(localStorage.getItem(key));
        } catch {}
      }
    }
    
    return data;
  }
  
  set(key, value) {
    const storageKey = `${this.namespace}_data_${key}`;
    try {
      localStorage.setItem(storageKey, JSON.stringify(value));
      return true;
    } catch {
      return false;
    }
  }
  
  get(key, defaultValue = null) {
    const storageKey = `${this.namespace}_data_${key}`;
    try {
      const raw = localStorage.getItem(storageKey);
      return raw !== null ? JSON.parse(raw) : defaultValue;
    } catch {
      return defaultValue;
    }
  }
  
  remove(key) {
    localStorage.removeItem(`${this.namespace}_data_${key}`);
  }
  
  getVersion() {
    return this.#getVersion();
  }
}

// ตัวอย่าง
const storage = new VersionedStorage('myapp', 3, {
  // Migration v1 -> v2: เปลี่ยน 'name' เป็น 'fullName'
  2: (data) => {
    if (data.settings?.name) {
      return {
        ...data,
        settings: {
          ...data.settings,
          fullName: data.settings.name,
          name: undefined
        }
      };
    }
    return data;
  },
  // Migration v2 -> v3: เพิ่ม 'preferences' object
  3: (data) => ({
    ...data,
    preferences: {
      theme: data.settings?.theme || 'light',
      language: data.settings?.language || 'th',
      notifications: true
    }
  })
});

storage.set('settings', { theme: 'dark', language: 'th' });
console.log('Version:', storage.getVersion()); // 3
```

---

## Step 834: sessionStorage

```javascript
// sessionStorage - เก็บข้อมูลตลอด session (ปิด tab = ข้อมูลหาย)

// Session-scoped storage
class SessionStorage {
  constructor(prefix = '') {
    this.prefix = prefix;
  }
  
  #key(key) { return this.prefix ? `${this.prefix}_${key}` : key; }
  
  set(key, value) {
    try {
      sessionStorage.setItem(this.#key(key), JSON.stringify(value));
      return true;
    } catch (e) {
      return false;
    }
  }
  
  get(key, defaultValue = null) {
    try {
      const item = sessionStorage.getItem(this.#key(key));
      return item !== null ? JSON.parse(item) : defaultValue;
    } catch {
      return defaultValue;
    }
  }
  
  remove(key) { sessionStorage.removeItem(this.#key(key)); }
  clear() { sessionStorage.clear(); }
  has(key) { return sessionStorage.getItem(this.#key(key)) !== null; }
}

// ใช้ sessionStorage สำหรับ multi-step form
class MultiStepFormStorage {
  constructor(formId) {
    this.storage = new SessionStorage(`form_${formId}`);
    this.formId = formId;
  }
  
  saveStep(stepNumber, data) {
    this.storage.set(`step_${stepNumber}`, data);
    this.storage.set('lastStep', stepNumber);
  }
  
  getStep(stepNumber) {
    return this.storage.get(`step_${stepNumber}`, {});
  }
  
  getLastStep() {
    return this.storage.get('lastStep', 1);
  }
  
  getAllSteps() {
    const result = {};
    let step = 1;
    while (this.storage.has(`step_${step}`)) {
      result[step] = this.getStep(step);
      step++;
    }
    return result;
  }
  
  clear() {
    this.storage.clear();
  }
  
  // รวมข้อมูลทุก step
  compile() {
    return Object.values(this.getAllSteps()).reduce(
      (acc, stepData) => ({ ...acc, ...stepData }),
      {}
    );
  }
}

// ใช้งาน Multi-step Form
const registrationForm = new MultiStepFormStorage('registration');

// Step 1: ข้อมูลส่วนตัว
function savePersonalInfo(data) {
  registrationForm.saveStep(1, {
    firstName: data.firstName,
    lastName: data.lastName,
    email: data.email
  });
}

// Step 2: ที่อยู่
function saveAddress(data) {
  registrationForm.saveStep(2, {
    address: data.address,
    city: data.city,
    province: data.province
  });
}

// Step 3: บัญชีผู้ใช้
function saveAccount(data) {
  registrationForm.saveStep(3, {
    username: data.username,
    password: data.password  // ควร hash ก่อน
  });
}

// Submit ทั้งหมด
async function submitRegistration() {
  const formData = registrationForm.compile();
  console.log('Form data:', formData);
  
  try {
    // await api.registerUser(formData);
    registrationForm.clear();
    console.log('ลงทะเบียนสำเร็จ');
  } catch (error) {
    console.error('ลงทะเบียนไม่สำเร็จ:', error.message);
  }
}

// ตรวจสอบ ณ load เพื่อ restore form state
const lastStep = registrationForm.getLastStep();
if (lastStep > 1) {
  console.log(`กลับมาที่ step ${lastStep}`);
  const lastData = registrationForm.getStep(lastStep);
  console.log('ข้อมูลที่บันทึกไว้:', lastData);
}
```

---

## Step 835: Storage Quota

```javascript
// ตรวจสอบ storage quota

async function checkStorageQuota() {
  if ('storage' in navigator && 'estimate' in navigator.storage) {
    const estimate = await navigator.storage.estimate();
    
    const usage = estimate.usage || 0;
    const quota = estimate.quota || 0;
    const available = quota - usage;
    const percentUsed = quota > 0 ? (usage / quota * 100).toFixed(1) : 0;
    
    return {
      usage: formatBytes(usage),
      quota: formatBytes(quota),
      available: formatBytes(available),
      percentUsed: `${percentUsed}%`,
      raw: { usage, quota, available }
    };
  }
  
  return { error: 'Storage Manager API ไม่รองรับ' };
}

function formatBytes(bytes) {
  if (bytes === 0) return '0 Bytes';
  const k = 1024;
  const sizes = ['Bytes', 'KB', 'MB', 'GB'];
  const i = Math.floor(Math.log(bytes) / Math.log(k));
  return parseFloat((bytes / Math.pow(k, i)).toFixed(2)) + ' ' + sizes[i];
}

// ใช้งาน
checkStorageQuota().then(quota => {
  console.log('Storage Usage:');
  console.log('  ใช้ไป:', quota.usage);
  console.log('  ทั้งหมด:', quota.quota);
  console.log('  เหลือ:', quota.available);
  console.log('  เปอร์เซ็นต์:', quota.percentUsed);
});

// ตรวจสอบก่อนบันทึกข้อมูลขนาดใหญ่
async function canStoreData(dataSize) {
  const quota = await checkStorageQuota();
  return quota.raw.available > dataSize;
}

// localStorage size estimation
function getLocalStorageSize() {
  let total = 0;
  for (const key in localStorage) {
    if (localStorage.hasOwnProperty(key)) {
      total += (localStorage[key].length + key.length) * 2; // UTF-16
    }
  }
  return {
    bytes: total,
    formatted: formatBytes(total),
    approximate: true
  };
}

console.log('localStorage size:', getLocalStorageSize());
```

---

## Step 836: StorageManager API

```javascript
// StorageManager API - Persistence

class StoragePersistenceManager {
  async isPersistent() {
    if (!('storage' in navigator)) return false;
    return await navigator.storage.persisted();
  }
  
  async requestPersistence() {
    if (!('storage' in navigator)) return false;
    
    const isPersisted = await navigator.storage.persisted();
    
    if (isPersisted) {
      console.log('Storage ถาวรอยู่แล้ว');
      return true;
    }
    
    const granted = await navigator.storage.persist();
    
    if (granted) {
      console.log('ได้รับสิทธิ์ Storage ถาวรแล้ว');
    } else {
      console.log('ไม่ได้รับสิทธิ์ Storage ถาวร');
    }
    
    return granted;
  }
  
  async getDetails() {
    const [estimate, persisted] = await Promise.all([
      navigator.storage.estimate(),
      this.isPersistent()
    ]);
    
    const usageDetails = estimate.usageDetails || {};
    
    return {
      persisted,
      total: formatBytes(estimate.quota || 0),
      used: formatBytes(estimate.usage || 0),
      available: formatBytes((estimate.quota || 0) - (estimate.usage || 0)),
      percentUsed: estimate.quota 
        ? ((estimate.usage / estimate.quota) * 100).toFixed(1) + '%' 
        : '0%',
      breakdown: {
        indexedDB: formatBytes(usageDetails.indexedDB || 0),
        serviceWorkerRegistrations: formatBytes(usageDetails.serviceWorkerRegistrations || 0),
        cacheStorage: formatBytes(usageDetails.caches || 0)
      }
    };
  }
}

// ใช้ persistent storage สำหรับข้อมูลสำคัญ
async function ensurePersistentStorage() {
  const manager = new StoragePersistenceManager();
  const details = await manager.getDetails();
  
  console.log('Storage Details:', details);
  
  if (!details.persisted) {
    console.log('กำลังขอ persistent storage...');
    const granted = await manager.requestPersistence();
    
    if (!granted) {
      console.warn('⚠️ Storage อาจถูกลบเมื่อ device มี storage น้อย');
    }
  }
}

ensurePersistentStorage();
```

---

## Step 837: IndexedDB - Overview

IndexedDB คือ database เต็มรูปแบบในบราวเซอร์ รองรับข้อมูลที่ซับซ้อนและขนาดใหญ่

```javascript
// IndexedDB คืออะไร:
// - NoSQL database แบบ key-value store
// // - รองรับ indexes สำหรับ query
// - Transaction-based
// - Asynchronous
// - รองรับ objects, arrays, Blobs, Files
// - Storage ไม่มีขีดจำกัดที่ชัดเจน (ขึ้นกับ device)

// ตรวจสอบการรองรับ
function isIndexedDBSupported() {
  return 'indexedDB' in window;
}

// เปิด database
function openDatabase(name, version, upgradeCallback) {
  return new Promise((resolve, reject) => {
    if (!isIndexedDBSupported()) {
      reject(new Error('IndexedDB ไม่รองรับ'));
      return;
    }
    
    const request = indexedDB.open(name, version);
    
    // เรียกเมื่อ version เพิ่มขึ้น (สร้างหรืออัพเกรด)
    request.onupgradeneeded = (event) => {
      const db = event.target.result;
      const oldVersion = event.oldVersion;
      const newVersion = event.newVersion;
      
      console.log(`Database upgrade: v${oldVersion} -> v${newVersion}`);
      upgradeCallback?.(db, oldVersion, newVersion);
    };
    
    request.onsuccess = (event) => {
      resolve(event.target.result);
    };
    
    request.onerror = (event) => {
      reject(new Error(`IndexedDB error: ${event.target.error}`));
    };
    
    request.onblocked = () => {
      console.warn('Database blocked - กรุณาปิด tabs อื่นที่ใช้ database นี้อยู่');
    };
  });
}
```

---

## Step 838: IndexedDB - Object Stores

```javascript
// สร้าง Object Store (เหมือน Table ใน SQL)

async function setupDatabase() {
  const db = await openDatabase('ShopDB', 1, (db) => {
    // สร้าง products store
    if (!db.objectStoreNames.contains('products')) {
      const productStore = db.createObjectStore('products', {
        keyPath: 'id',        // primary key
        autoIncrement: false  // เราจะกำหนด id เอง
      });
      
      // สร้าง indexes
      productStore.createIndex('name', 'name', { unique: false });
      productStore.createIndex('category', 'category', { unique: false });
      productStore.createIndex('price', 'price', { unique: false });
      productStore.createIndex('name_category', ['name', 'category'], { unique: false });
    }
    
    // สร้าง customers store
    if (!db.objectStoreNames.contains('customers')) {
      const customerStore = db.createObjectStore('customers', {
        keyPath: 'id',
        autoIncrement: true  // auto-increment key
      });
      
      customerStore.createIndex('email', 'email', { unique: true });
      customerStore.createIndex('phone', 'phone', { unique: false });
    }
    
    // สร้าง orders store
    if (!db.objectStoreNames.contains('orders')) {
      const orderStore = db.createObjectStore('orders', {
        keyPath: 'orderId',
        autoIncrement: true
      });
      
      orderStore.createIndex('customerId', 'customerId', { unique: false });
      orderStore.createIndex('status', 'status', { unique: false });
      orderStore.createIndex('createdAt', 'createdAt', { unique: false });
    }
  });
  
  return db;
}
```

---

## Step 839: IndexedDB - CRUD Operations

```javascript
// CRUD operations helper
class IndexedDBHelper {
  constructor(db) {
    this.db = db;
  }
  
  // สร้าง transaction
  #transaction(storeNames, mode = 'readonly') {
    return this.db.transaction(storeNames, mode);
  }
  
  // Add (ไม่อนุญาตให้ key ซ้ำ)
  add(storeName, data) {
    return new Promise((resolve, reject) => {
      const tx = this.#transaction(storeName, 'readwrite');
      const store = tx.objectStore(storeName);
      const request = store.add(data);
      
      request.onsuccess = () => resolve(request.result); // return key
      request.onerror = () => reject(request.error);
    });
  }
  
  // Put (add หรือ update ถ้า key มีอยู่แล้ว)
  put(storeName, data) {
    return new Promise((resolve, reject) => {
      const tx = this.#transaction(storeName, 'readwrite');
      const store = tx.objectStore(storeName);
      const request = store.put(data);
      
      request.onsuccess = () => resolve(request.result);
      request.onerror = () => reject(request.error);
    });
  }
  
  // Get by key
  get(storeName, key) {
    return new Promise((resolve, reject) => {
      const tx = this.#transaction(storeName);
      const store = tx.objectStore(storeName);
      const request = store.get(key);
      
      request.onsuccess = () => resolve(request.result || null);
      request.onerror = () => reject(request.error);
    });
  }
  
  // Get all
  getAll(storeName, query = null) {
    return new Promise((resolve, reject) => {
      const tx = this.#transaction(storeName);
      const store = tx.objectStore(storeName);
      const request = store.getAll(query);
      
      request.onsuccess = () => resolve(request.result);
      request.onerror = () => reject(request.error);
    });
  }
  
  // Get by index
  getByIndex(storeName, indexName, value) {
    return new Promise((resolve, reject) => {
      const tx = this.#transaction(storeName);
      const store = tx.objectStore(storeName);
      const index = store.index(indexName);
      const request = index.get(value);
      
      request.onsuccess = () => resolve(request.result || null);
      request.onerror = () => reject(request.error);
    });
  }
  
  // Get all by index
  getAllByIndex(storeName, indexName, value) {
    return new Promise((resolve, reject) => {
      const tx = this.#transaction(storeName);
      const store = tx.objectStore(storeName);
      const index = store.index(indexName);
      const request = index.getAll(value);
      
      request.onsuccess = () => resolve(request.result);
      request.onerror = () => reject(request.error);
    });
  }
  
  // Delete
  delete(storeName, key) {
    return new Promise((resolve, reject) => {
      const tx = this.#transaction(storeName, 'readwrite');
      const store = tx.objectStore(storeName);
      const request = store.delete(key);
      
      request.onsuccess = () => resolve(true);
      request.onerror = () => reject(request.error);
    });
  }
  
  // Count
  count(storeName, query = null) {
    return new Promise((resolve, reject) => {
      const tx = this.#transaction(storeName);
      const store = tx.objectStore(storeName);
      const request = store.count(query);
      
      request.onsuccess = () => resolve(request.result);
      request.onerror = () => reject(request.error);
    });
  }
  
  // Clear store
  clear(storeName) {
    return new Promise((resolve, reject) => {
      const tx = this.#transaction(storeName, 'readwrite');
      const store = tx.objectStore(storeName);
      const request = store.clear();
      
      request.onsuccess = () => resolve(true);
      request.onerror = () => reject(request.error);
    });
  }
}

// ใช้งาน
async function demoIndexedDB() {
  const db = await setupDatabase();
  const helper = new IndexedDBHelper(db);
  
  // Add products
  await helper.add('products', {
    id: 'P001',
    name: 'เสื้อยืด',
    category: 'เสื้อผ้า',
    price: 299,
    stock: 50
  });
  
  await helper.add('products', {
    id: 'P002',
    name: 'กางเกงยีนส์',
    category: 'เสื้อผ้า',
    price: 599,
    stock: 30
  });
  
  // Get product
  const product = await helper.get('products', 'P001');
  console.log('Product:', product);
  
  // Update product
  await helper.put('products', { ...product, price: 349 });
  
  // Get all by category
  const clothes = await helper.getAllByIndex('products', 'category', 'เสื้อผ้า');
  console.log('เสื้อผ้าทั้งหมด:', clothes);
  
  // Count
  const total = await helper.count('products');
  console.log('จำนวน products:', total);
}
```

---

## Step 840: IndexedDB - Transactions

```javascript
// Transactions ขั้นสูง

class TransactionManager {
  constructor(db) {
    this.db = db;
  }
  
  // Multiple operations ใน transaction เดียว
  async runTransaction(storeNames, operations) {
    return new Promise((resolve, reject) => {
      const tx = this.db.transaction(
        Array.isArray(storeNames) ? storeNames : [storeNames],
        'readwrite'
      );
      
      const results = [];
      
      tx.oncomplete = () => resolve(results);
      tx.onerror = () => reject(tx.error);
      tx.onabort = () => reject(new Error('Transaction aborted'));
      
      // Execute operations
      operations.forEach((op, index) => {
        const store = tx.objectStore(op.store);
        let request;
        
        switch (op.type) {
          case 'add': request = store.add(op.data); break;
          case 'put': request = store.put(op.data); break;
          case 'delete': request = store.delete(op.key); break;
          case 'get': request = store.get(op.key); break;
          default: throw new Error(`Unknown operation: ${op.type}`);
        }
        
        request.onsuccess = () => {
          results[index] = request.result;
        };
        
        request.onerror = () => {
          tx.abort();
          reject(request.error);
        };
      });
    });
  }
  
  // Transfer ระหว่าง stores
  async transferStock(fromProductId, toProductId, qty) {
    return new Promise((resolve, reject) => {
      const tx = this.db.transaction('products', 'readwrite');
      const store = tx.objectStore('products');
      
      let fromProduct, toProduct;
      
      // Get from product
      const getFrom = store.get(fromProductId);
      getFrom.onsuccess = () => {
        fromProduct = getFrom.result;
        
        if (!fromProduct || fromProduct.stock < qty) {
          tx.abort();
          reject(new Error('สินค้าไม่เพียงพอ'));
          return;
        }
        
        // Get to product
        const getTo = store.get(toProductId);
        getTo.onsuccess = () => {
          toProduct = getTo.result;
          
          if (!toProduct) {
            tx.abort();
            reject(new Error('ไม่พบ product ปลายทาง'));
            return;
          }
          
          // Update both products
          store.put({ ...fromProduct, stock: fromProduct.stock - qty });
          store.put({ ...toProduct, stock: toProduct.stock + qty });
        };
      };
      
      tx.oncomplete = () => resolve({ success: true, qty });
      tx.onerror = () => reject(tx.error);
      tx.onabort = () => reject(new Error('Transfer ถูกยกเลิก'));
    });
  }
}
```

---

## Step 841: IndexedDB - Cursor Iteration

```javascript
// Cursor สำหรับ iterate ข้อมูลจำนวนมาก

class CursorIterator {
  constructor(db) {
    this.db = db;
  }
  
  // Iterate ทั้งหมด
  iterateAll(storeName, callback) {
    return new Promise((resolve, reject) => {
      const tx = this.db.transaction(storeName, 'readonly');
      const store = tx.objectStore(storeName);
      const request = store.openCursor();
      
      request.onsuccess = (event) => {
        const cursor = event.target.result;
        
        if (cursor) {
          callback(cursor.value, cursor.key);
          cursor.continue();
        } else {
          resolve(); // ไม่มีข้อมูลเหลือแล้ว
        }
      };
      
      request.onerror = () => reject(request.error);
    });
  }
  
  // Iterate พร้อม filter
  iterateWithFilter(storeName, filterFn) {
    return new Promise((resolve, reject) => {
      const results = [];
      const tx = this.db.transaction(storeName, 'readonly');
      const store = tx.objectStore(storeName);
      const request = store.openCursor();
      
      request.onsuccess = (event) => {
        const cursor = event.target.result;
        
        if (cursor) {
          if (filterFn(cursor.value)) {
            results.push(cursor.value);
          }
          cursor.continue();
        } else {
          resolve(results);
        }
      };
      
      request.onerror = () => reject(request.error);
    });
  }
  
  // Iterate กับ range
  iterateRange(storeName, indexName, range) {
    return new Promise((resolve, reject) => {
      const results = [];
      const tx = this.db.transaction(storeName, 'readonly');
      const store = tx.objectStore(storeName);
      const index = store.index(indexName);
      const request = index.openCursor(range);
      
      request.onsuccess = (event) => {
        const cursor = event.target.result;
        
        if (cursor) {
          results.push(cursor.value);
          cursor.continue();
        } else {
          resolve(results);
        }
      };
      
      request.onerror = () => reject(request.error);
    });
  }
  
  // Update ขณะ iterate
  updateWhileIterating(storeName, updateFn) {
    return new Promise((resolve, reject) => {
      let updateCount = 0;
      const tx = this.db.transaction(storeName, 'readwrite');
      const store = tx.objectStore(storeName);
      const request = store.openCursor();
      
      request.onsuccess = (event) => {
        const cursor = event.target.result;
        
        if (cursor) {
          const newValue = updateFn(cursor.value);
          if (newValue !== null) {
            cursor.update(newValue);
            updateCount++;
          }
          cursor.continue();
        } else {
          resolve(updateCount);
        }
      };
      
      request.onerror = () => reject(request.error);
    });
  }
  
  // Delete ขณะ iterate
  deleteWhileIterating(storeName, shouldDelete) {
    return new Promise((resolve, reject) => {
      let deleteCount = 0;
      const tx = this.db.transaction(storeName, 'readwrite');
      const store = tx.objectStore(storeName);
      const request = store.openCursor();
      
      request.onsuccess = (event) => {
        const cursor = event.target.result;
        
        if (cursor) {
          if (shouldDelete(cursor.value)) {
            cursor.delete();
            deleteCount++;
          }
          cursor.continue();
        } else {
          resolve(deleteCount);
        }
      };
      
      request.onerror = () => reject(request.error);
    });
  }
  
  // Pagination ด้วย cursor
  async getPage(storeName, page = 1, pageSize = 10) {
    return new Promise((resolve, reject) => {
      const results = [];
      const skip = (page - 1) * pageSize;
      let skipped = 0;
      let count = 0;
      
      const tx = this.db.transaction(storeName, 'readonly');
      const store = tx.objectStore(storeName);
      const request = store.openCursor();
      
      request.onsuccess = (event) => {
        const cursor = event.target.result;
        
        if (!cursor) {
          resolve(results);
          return;
        }
        
        if (skipped < skip) {
          skipped++;
          cursor.continue();
          return;
        }
        
        if (count < pageSize) {
          results.push(cursor.value);
          count++;
          cursor.continue();
        } else {
          resolve(results);
        }
      };
      
      request.onerror = () => reject(request.error);
    });
  }
}

// IDBKeyRange - สำหรับ range queries
function demoKeyRange() {
  // ค่าเดียว
  const only = IDBKeyRange.only(5);
  
  // Lower bound (>= 5)
  const lowerBound = IDBKeyRange.lowerBound(5);
  // Lower bound (> 5) - open = exclude boundary
  const lowerBoundExclusive = IDBKeyRange.lowerBound(5, true);
  
  // Upper bound (<= 100)
  const upperBound = IDBKeyRange.upperBound(100);
  
  // Bound (5 <= x <= 100)
  const bound = IDBKeyRange.bound(5, 100);
  // (5 < x < 100)
  const boundExclusive = IDBKeyRange.bound(5, 100, true, true);
  
  return { only, lowerBound, upperBound, bound };
}
```

---

## Step 842: IndexedDB กับ Promises (Promisify)

```javascript
// IndexedDB Database Manager ที่ใช้ Promises อย่างเต็มรูปแบบ

class IDBDatabase {
  constructor(name, version, schema) {
    this.name = name;
    this.version = version;
    this.schema = schema;
    this.db = null;
  }
  
  async open() {
    if (this.db) return this.db;
    
    this.db = await new Promise((resolve, reject) => {
      const request = indexedDB.open(this.name, this.version);
      
      request.onupgradeneeded = (event) => {
        const db = event.target.result;
        this.#applySchema(db, event.oldVersion);
      };
      
      request.onsuccess = (event) => resolve(event.target.result);
      request.onerror = (event) => reject(event.target.error);
      request.onblocked = () => reject(new Error('Database blocked'));
    });
    
    // Handle version change from another tab
    this.db.onversionchange = () => {
      this.db.close();
      this.db = null;
      console.warn('Database version changed, connection closed');
    };
    
    return this.db;
  }
  
  #applySchema(db, oldVersion) {
    for (const [storeName, storeConfig] of Object.entries(this.schema)) {
      if (!db.objectStoreNames.contains(storeName)) {
        const store = db.createObjectStore(storeName, {
          keyPath: storeConfig.keyPath,
          autoIncrement: storeConfig.autoIncrement || false
        });
        
        (storeConfig.indexes || []).forEach(index => {
          store.createIndex(
            index.name,
            index.keyPath,
            { unique: index.unique || false }
          );
        });
      }
    }
  }
  
  async close() {
    if (this.db) {
      this.db.close();
      this.db = null;
    }
  }
  
  async delete() {
    await this.close();
    
    return new Promise((resolve, reject) => {
      const request = indexedDB.deleteDatabase(this.name);
      request.onsuccess = () => resolve();
      request.onerror = () => reject(request.error);
    });
  }
  
  store(name) {
    return {
      db: this,
      storeName: name,
      
      async add(data) {
        const db = await this.db.open();
        return new Promise((resolve, reject) => {
          const tx = db.transaction(this.storeName, 'readwrite');
          const store = tx.objectStore(this.storeName);
          const req = store.add(data);
          req.onsuccess = () => resolve(req.result);
          req.onerror = () => reject(req.error);
        });
      },
      
      async put(data) {
        const db = await this.db.open();
        return new Promise((resolve, reject) => {
          const tx = db.transaction(this.storeName, 'readwrite');
          const store = tx.objectStore(this.storeName);
          const req = store.put(data);
          req.onsuccess = () => resolve(req.result);
          req.onerror = () => reject(req.error);
        });
      },
      
      async get(key) {
        const db = await this.db.open();
        return new Promise((resolve, reject) => {
          const tx = db.transaction(this.storeName, 'readonly');
          const store = tx.objectStore(this.storeName);
          const req = store.get(key);
          req.onsuccess = () => resolve(req.result || null);
          req.onerror = () => reject(req.error);
        });
      },
      
      async getAll() {
        const db = await this.db.open();
        return new Promise((resolve, reject) => {
          const tx = db.transaction(this.storeName, 'readonly');
          const store = tx.objectStore(this.storeName);
          const req = store.getAll();
          req.onsuccess = () => resolve(req.result);
          req.onerror = () => reject(req.error);
        });
      },
      
      async delete(key) {
        const db = await this.db.open();
        return new Promise((resolve, reject) => {
          const tx = db.transaction(this.storeName, 'readwrite');
          const store = tx.objectStore(this.storeName);
          const req = store.delete(key);
          req.onsuccess = () => resolve(true);
          req.onerror = () => reject(req.error);
        });
      }
    };
  }
}

// ใช้งาน
const appDB = new IDBDatabase('AppDB', 1, {
  users: {
    keyPath: 'id',
    indexes: [
      { name: 'email', keyPath: 'email', unique: true }
    ]
  },
  posts: {
    keyPath: 'id',
    autoIncrement: true,
    indexes: [
      { name: 'userId', keyPath: 'userId' },
      { name: 'createdAt', keyPath: 'createdAt' }
    ]
  }
});

async function dbDemo() {
  const users = appDB.store('users');
  
  await users.put({ id: 'u1', name: 'สมชาย', email: 'somchai@example.com' });
  
  const user = await users.get('u1');
  console.log('User:', user);
  
  const allUsers = await users.getAll();
  console.log('All users:', allUsers);
}
```

---

## Step 843: Cache API

```javascript
// Cache API - สำหรับ cache network requests

class CacheManager {
  constructor(cacheName, version = 1) {
    this.cacheName = `${cacheName}-v${version}`;
  }
  
  async open() {
    return await caches.open(this.cacheName);
  }
  
  // Cache specific URL
  async add(url) {
    const cache = await this.open();
    await cache.add(url);
    console.log(`Cached: ${url}`);
  }
  
  // Cache multiple URLs
  async addAll(urls) {
    const cache = await this.open();
    await cache.addAll(urls);
    console.log(`Cached ${urls.length} URLs`);
  }
  
  // Cache a Response manually
  async put(url, response) {
    const cache = await this.open();
    await cache.put(url, response);
  }
  
  // Get cached response
  async match(url) {
    const cache = await this.open();
    return await cache.match(url);
  }
  
  // Get from any cache
  async matchAll(url) {
    return await caches.match(url);
  }
  
  // Delete cached item
  async delete(url) {
    const cache = await this.open();
    return await cache.delete(url);
  }
  
  // Get all cached keys
  async keys() {
    const cache = await this.open();
    return await cache.keys();
  }
  
  // List all caches
  static async listAll() {
    const cacheNames = await caches.keys();
    console.log('All caches:', cacheNames);
    return cacheNames;
  }
  
  // Delete old caches (ใน Service Worker)
  static async cleanup(keepCaches = []) {
    const cacheNames = await caches.keys();
    
    await Promise.all(
      cacheNames
        .filter(name => !keepCaches.includes(name))
        .map(name => {
          console.log(`Deleting old cache: ${name}`);
          return caches.delete(name);
        })
    );
  }
}
```

---

## Step 844-845: Caching Strategies

```javascript
// Caching Strategies

// Strategy 1: Cache First (Static assets)
async function cacheFirst(request) {
  const cached = await caches.match(request);
  
  if (cached) {
    return cached;
  }
  
  // ถ้าไม่มีใน cache ดึงจาก network และ cache ไว้
  const response = await fetch(request);
  
  if (response.ok) {
    const cache = await caches.open('static-v1');
    cache.put(request, response.clone());
  }
  
  return response;
}

// Strategy 2: Network First (Dynamic content)
async function networkFirst(request, cacheName = 'dynamic-v1') {
  try {
    const response = await fetch(request);
    
    if (response.ok) {
      const cache = await caches.open(cacheName);
      cache.put(request, response.clone());
    }
    
    return response;
    
  } catch (error) {
    // Network failed - ใช้ cache แทน
    const cached = await caches.match(request);
    
    if (cached) {
      console.log('Network failed, using cache');
      return cached;
    }
    
    throw error;
  }
}

// Strategy 3: Stale While Revalidate (APIs, feeds)
async function staleWhileRevalidate(request, cacheName = 'api-v1') {
  const cache = await caches.open(cacheName);
  const cached = await cache.match(request);
  
  // ดึง network ใน background (ไม่ await)
  const networkPromise = fetch(request).then(response => {
    if (response.ok) {
      cache.put(request, response.clone());
      console.log('Cache updated in background');
    }
    return response;
  });
  
  // Return cached immediately ถ้ามี, ไม่งั้น รอ network
  return cached || networkPromise;
}

// Strategy 4: Cache Only (เมื่อ offline)
async function cacheOnly(request) {
  const cached = await caches.match(request);
  
  if (!cached) {
    throw new Error('ไม่พบใน cache และไม่มีการเชื่อมต่อ');
  }
  
  return cached;
}

// Strategy 5: Network Only (ไม่ cache เลย)
async function networkOnly(request) {
  return fetch(request);
}

// Smart Router - เลือก strategy ตาม URL
class CacheRouter {
  constructor() {
    this.routes = [];
  }
  
  addRoute(pattern, strategy, options = {}) {
    this.routes.push({ pattern, strategy, options });
    return this;
  }
  
  async handle(request) {
    const url = new URL(request.url);
    
    for (const route of this.routes) {
      if (this.#matches(url, route.pattern)) {
        return route.strategy(request, route.options);
      }
    }
    
    // Default: network first
    return networkFirst(request);
  }
  
  #matches(url, pattern) {
    if (pattern instanceof RegExp) {
      return pattern.test(url.pathname);
    }
    if (typeof pattern === 'function') {
      return pattern(url);
    }
    return url.pathname.startsWith(pattern);
  }
}

// ใน Service Worker
const cacheRouter = new CacheRouter()
  // Static assets - cache first
  .addRoute(/\.(js|css|png|jpg|svg|woff2?)$/, cacheFirst)
  // HTML pages - network first  
  .addRoute(/\.html$/, networkFirst)
  // API calls - stale while revalidate
  .addRoute('/api/', staleWhileRevalidate)
  // Images - cache first
  .addRoute('/images/', cacheFirst);

// Service Worker fetch handler
// self.addEventListener('fetch', event => {
//   event.respondWith(cacheRouter.handle(event.request));
// });
```

---

## Step 846-850: Advanced Storage Patterns

```javascript
// Offline Queue - Queue operations ขณะ offline

class OfflineQueue {
  constructor() {
    this.storage = new ExpiringStorage('offline_queue_', null); // ไม่หมดอายุ
    this.queueKey = 'pending_operations';
  }
  
  enqueue(operation) {
    const queue = this.getQueue();
    const item = {
      id: Date.now() + '_' + Math.random().toString(36).substr(2, 9),
      ...operation,
      timestamp: new Date().toISOString(),
      retries: 0
    };
    
    queue.push(item);
    this.storage.set(this.queueKey, queue);
    
    console.log(`Operation queued: ${operation.type}`);
    return item.id;
  }
  
  dequeue(id) {
    const queue = this.getQueue().filter(op => op.id !== id);
    this.storage.set(this.queueKey, queue);
  }
  
  getQueue() {
    return this.storage.get(this.queueKey, []);
  }
  
  size() {
    return this.getQueue().length;
  }
  
  async processQueue(processor) {
    const queue = this.getQueue();
    
    if (queue.length === 0) return;
    
    console.log(`Processing ${queue.length} queued operations...`);
    
    for (const operation of queue) {
      try {
        await processor(operation);
        this.dequeue(operation.id);
        console.log(`✓ Operation ${operation.id} processed`);
      } catch (error) {
        operation.retries++;
        
        if (operation.retries >= 3) {
          console.error(`✗ Operation ${operation.id} failed after 3 retries`);
          this.dequeue(operation.id);
        } else {
          console.warn(`Retry ${operation.retries}/3 for operation ${operation.id}`);
          // Update retry count
          const queue = this.getQueue();
          const idx = queue.findIndex(op => op.id === operation.id);
          if (idx >= 0) {
            queue[idx] = operation;
            this.storage.set(this.queueKey, queue);
          }
        }
      }
    }
  }
}

// Sync เมื่อกลับมา online
const offlineQueue = new OfflineQueue();

window.addEventListener('online', async () => {
  console.log('กลับมาออนไลน์ กำลัง sync...');
  
  await offlineQueue.processQueue(async (operation) => {
    switch (operation.type) {
      case 'CREATE':
        await fetch('/api/' + operation.resource, {
          method: 'POST',
          body: JSON.stringify(operation.data)
        });
        break;
        
      case 'UPDATE':
        await fetch(`/api/${operation.resource}/${operation.id}`, {
          method: 'PUT',
          body: JSON.stringify(operation.data)
        });
        break;
        
      case 'DELETE':
        await fetch(`/api/${operation.resource}/${operation.id}`, {
          method: 'DELETE'
        });
        break;
    }
  });
  
  console.log('Sync เสร็จสิ้น');
});

// ตัวอย่างการใช้งาน
async function createPost(data) {
  if (!navigator.onLine) {
    // Queue สำหรับส่งทีหลัง
    offlineQueue.enqueue({
      type: 'CREATE',
      resource: 'posts',
      data
    });
    
    // Save locally
    const localPosts = LocalStorage.get('local_posts', []);
    localPosts.push({ ...data, _local: true, _id: Date.now() });
    LocalStorage.set('local_posts', localPosts);
    
    return { success: true, offline: true, message: 'จะส่งเมื่อออนไลน์' };
  }
  
  const response = await fetch('/api/posts', {
    method: 'POST',
    body: JSON.stringify(data)
  });
  
  return response.json();
}
```

---

## แบบฝึกหัด (Exercises)

### แบบฝึกหัดที่ 1: Shopping Cart ด้วย IndexedDB

```javascript
// TODO: สร้าง ShoppingCart class ที่ใช้ IndexedDB:
// 1. addItem(product, qty)
// 2. removeItem(productId)
// 3. updateQty(productId, qty)
// 4. getItems() - รวม product details
// 5. getTotal()
// 6. clear()
// 7. sync กับ server เมื่อออนไลน์

class ShoppingCart {
  constructor() {
    // TODO: setup IndexedDB
  }
  
  async addItem(product, qty = 1) {
    // TODO
  }
  
  async removeItem(productId) {
    // TODO
  }
  
  async getTotal() {
    // TODO
  }
}
```

### แบบฝึกหัดที่ 2: Media Cache Manager

```javascript
// TODO: สร้าง MediaCacheManager ที่:
// 1. Cache images, videos, audio files
// 2. Track cache size
// 3. Evict ตาม LRU policy
// 4. Support partial caching (byte ranges)
// 5. แสดง download progress

class MediaCacheManager {
  // TODO
}
```

### แบบฝึกหัดที่ 3: Storage Analytics

```javascript
// TODO: สร้าง StorageAnalytics ที่:
// 1. Track ขนาดของแต่ละ storage type
// 2. แสดง breakdown ตาม prefix/namespace
// 3. Identify items ที่ใหญ่ที่สุด
// 4. Generate report

class StorageAnalytics {
  // TODO
}
```

---

## สรุป

Web Storage APIs ให้เครื่องมือที่ครบครันสำหรับการจัดเก็บข้อมูล:

- **localStorage**: ข้อมูลถาวร, ง่ายแต่จำกัด 5-10MB
- **sessionStorage**: ข้อมูลชั่วคราวต่อ session
- **IndexedDB**: Database เต็มรูปแบบ, transactions, indexes
- **Cache API**: Cache network requests สำหรับ offline
- **StorageManager**: จัดการ quota และ persistence

Key patterns ที่สำคัญ:
1. **Expiry**: localStorage ไม่มี built-in expiry ต้องจัดการเอง
2. **Offline Queue**: Queue operations ขณะ offline แล้ว sync ทีหลัง
3. **Caching Strategies**: Cache First / Network First / SWR ตามประเภทข้อมูล

---

*ต่อไป: Part 44 - Cookies*
