# Part 47: Service Workers เบื้องต้น

## Service Worker คืออะไร?

Service Worker เป็น JavaScript script ที่รันอยู่ใน background thread แยกจาก main thread ทำหน้าที่เป็น "proxy" ระหว่าง browser กับ network ช่วยให้ web app ทำงานแบบ offline ได้ รับ push notifications และทำ background sync

**Steps 911-930**

---

## Step 911: ทำความเข้าใจ Service Worker

```javascript
// Service Worker มีความสามารถพิเศษ:
// 1. Intercept network requests
// 2. Cache assets for offline use
// 3. Receive push notifications
// 4. Background sync
// 5. Handle periodic background sync

// ข้อจำกัด:
// - ต้องใช้ HTTPS (หรือ localhost)
// - ไม่สามารถเข้าถึง DOM โดยตรง
// - รันแยกจาก main thread

// วงจรชีวิต Service Worker:
// 1. Register (จาก main thread)
// 2. Download
// 3. Install
// 4. Activate
// 5. Idle
// 6. Fetch/Message/Push events

// Scope ของ Service Worker
// /sw.js ที่ root จะ control ทุก URL ใน origin
// /admin/sw.js จะ control เฉพาะ /admin/*
```

---

## Step 912: ลงทะเบียน Service Worker

```javascript
// main.js หรือ index.js

// ตรวจสอบว่า browser รองรับ Service Worker
if ("serviceWorker" in navigator) {
  // ลงทะเบียนเมื่อหน้าโหลดเสร็จ
  window.addEventListener("load", async () => {
    try {
      const registration = await navigator.serviceWorker.register("/sw.js", {
        scope: "/"  // Optional: กำหนด scope
      });
      
      console.log("Service Worker ลงทะเบียนสำเร็จ!");
      console.log("Scope:", registration.scope);
      
      // ตรวจสอบ state
      if (registration.installing) {
        console.log("Service Worker กำลัง install");
      } else if (registration.waiting) {
        console.log("Service Worker ติดตั้งแล้ว รอ activate");
      } else if (registration.active) {
        console.log("Service Worker กำลัง active");
      }
      
    } catch (error) {
      console.error("Service Worker ลงทะเบียนไม่สำเร็จ:", error);
    }
  });
}
```

```javascript
// ตรวจสอบและ update Service Worker
async function registerServiceWorker() {
  if (!("serviceWorker" in navigator)) {
    console.log("Browser ไม่รองรับ Service Worker");
    return;
  }
  
  try {
    const registration = await navigator.serviceWorker.register("/sw.js");
    
    // ฟัง update event
    registration.addEventListener("updatefound", () => {
      const newWorker = registration.installing;
      console.log("Service Worker ใหม่กำลัง install");
      
      newWorker.addEventListener("statechange", () => {
        console.log("State เปลี่ยนเป็น:", newWorker.state);
        
        if (newWorker.state === "installed" && navigator.serviceWorker.controller) {
          // มี Service Worker ใหม่รอ activate
          showUpdateNotification();
        }
      });
    });
    
    // ฟัง controller change
    navigator.serviceWorker.addEventListener("controllerchange", () => {
      console.log("Service Worker controller เปลี่ยนแล้ว");
      // Reload เพื่อใช้ Service Worker ใหม่
      window.location.reload();
    });
    
    return registration;
    
  } catch (error) {
    console.error("Registration failed:", error);
    throw error;
  }
}

function showUpdateNotification() {
  const notification = document.createElement("div");
  notification.className = "update-notification";
  notification.innerHTML = `
    <p>มีเวอร์ชันใหม่ของแอป</p>
    <button onclick="updateApp()">อัพเดทเดี๋ยวนี้</button>
  `;
  document.body.appendChild(notification);
}

async function updateApp() {
  const registration = await navigator.serviceWorker.getRegistration();
  if (registration?.waiting) {
    registration.waiting.postMessage({ type: "SKIP_WAITING" });
  }
}

registerServiceWorker();
```

---

## Step 913: Install Event - Caching Static Assets

```javascript
// sw.js - Install event

const CACHE_NAME = "my-app-v1";
const ASSETS_TO_CACHE = [
  "/",
  "/index.html",
  "/styles/main.css",
  "/scripts/app.js",
  "/images/logo.png",
  "/fonts/roboto.woff2",
  "/offline.html"
];

// Install event - จะเกิดขึ้นเมื่อ Service Worker ถูก install ครั้งแรก
self.addEventListener("install", (event) => {
  console.log("Service Worker: กำลัง install...");
  
  // waitUntil() ทำให้ install event รอจนกว่า cache จะเสร็จ
  event.waitUntil(
    caches.open(CACHE_NAME)
      .then((cache) => {
        console.log("กำลัง cache static assets...");
        return cache.addAll(ASSETS_TO_CACHE);
      })
      .then(() => {
        console.log("Cache เสร็จแล้ว");
        // skipWaiting() ทำให้ SW ใหม่ activate ทันทีไม่ต้องรอ
        return self.skipWaiting();
      })
      .catch((error) => {
        console.error("Cache ไม่สำเร็จ:", error);
      })
  );
});
```

```javascript
// Advanced install with progress tracking
self.addEventListener("install", (event) => {
  event.waitUntil(installServiceWorker());
});

async function installServiceWorker() {
  const cache = await caches.open(CACHE_NAME);
  
  // Cache แต่ละไฟล์แยกกัน เพื่อ handle error แต่ละไฟล์
  const cachePromises = ASSETS_TO_CACHE.map(async (url) => {
    try {
      await cache.add(url);
      console.log(`Cached: ${url}`);
    } catch (error) {
      console.warn(`ไม่สามารถ cache: ${url}`, error.message);
      // ไม่ throw error เพื่อให้ cache ไฟล์อื่นต่อได้
    }
  });
  
  await Promise.allSettled(cachePromises);
  
  // Cache API responses
  const apiCache = await caches.open("api-cache-v1");
  try {
    const response = await fetch("/api/initial-data");
    if (response.ok) {
      await apiCache.put("/api/initial-data", response);
    }
  } catch (e) {
    console.warn("ไม่สามารถ cache API data:", e.message);
  }
  
  return self.skipWaiting();
}
```

---

## Step 914: Activate Event - Cleanup Old Caches

```javascript
// sw.js - Activate event

const CACHE_VERSION = "v2";
const CURRENT_CACHES = {
  static: `static-cache-${CACHE_VERSION}`,
  dynamic: `dynamic-cache-${CACHE_VERSION}`,
  api: `api-cache-${CACHE_VERSION}`
};

self.addEventListener("activate", (event) => {
  console.log("Service Worker: กำลัง activate...");
  
  event.waitUntil(
    (async () => {
      // ลบ caches เก่าที่ไม่ใช้แล้ว
      const cacheKeys = await caches.keys();
      
      const deletePromises = cacheKeys
        .filter(key => !Object.values(CURRENT_CACHES).includes(key))
        .map(key => {
          console.log(`ลบ cache เก่า: ${key}`);
          return caches.delete(key);
        });
      
      await Promise.all(deletePromises);
      
      // ควบคุม clients ทั้งหมดทันที ไม่ต้องรือ reload
      await self.clients.claim();
      
      console.log("Service Worker: activate เสร็จแล้ว");
    })()
  );
});
```

```javascript
// การจัดการ cache version อย่างละเอียด
const CACHE_MANIFEST = {
  version: "2.0.0",
  staticCache: "static-v2.0.0",
  dynamicCache: "dynamic-v2.0.0"
};

self.addEventListener("activate", (event) => {
  event.waitUntil(cleanupOldCaches());
});

async function cleanupOldCaches() {
  const allCaches = await caches.keys();
  
  // Cache patterns ที่ควรเก็บ
  const validCachePatterns = [
    /^static-v\d+\.\d+\.\d+$/,
    /^dynamic-v\d+\.\d+\.\d+$/
  ];
  
  for (const cacheName of allCaches) {
    // ตรวจสอบว่า cache ยังใช้งานได้
    const isValidPattern = validCachePatterns.some(pattern => pattern.test(cacheName));
    const isCurrentCache = Object.values(CACHE_MANIFEST).includes(cacheName);
    
    if (isValidPattern && !isCurrentCache) {
      console.log(`กำลังลบ cache เก่า: ${cacheName}`);
      await caches.delete(cacheName);
    }
  }
  
  // Migrate data from old cache to new
  const oldStaticCache = await caches.open("static-v1.0.0");
  const newStaticCache = await caches.open(CACHE_MANIFEST.staticCache);
  
  const oldRequests = await oldStaticCache.keys();
  for (const request of oldRequests) {
    const response = await oldStaticCache.match(request);
    if (response) {
      await newStaticCache.put(request, response.clone());
    }
  }
  
  return self.clients.claim();
}
```

---

## Step 915: Fetch Event - Intercepting Requests

```javascript
// sw.js - Fetch event

self.addEventListener("fetch", (event) => {
  const { request } = event;
  const url = new URL(request.url);
  
  // ข้าม cross-origin requests
  if (url.origin !== location.origin) {
    return;
  }
  
  // ข้าม non-GET requests สำหรับ basic caching
  if (request.method !== "GET") {
    return;
  }
  
  event.respondWith(handleFetch(request));
});

async function handleFetch(request) {
  const url = new URL(request.url);
  
  // เลือก strategy ตามประเภทของ request
  if (url.pathname.startsWith("/api/")) {
    return networkFirstStrategy(request);
  } else if (url.pathname.match(/\.(jpg|jpeg|png|gif|svg|webp)$/)) {
    return cacheFirstStrategy(request, "images-cache");
  } else if (url.pathname.match(/\.(css|js|woff2?)$/)) {
    return staleWhileRevalidateStrategy(request);
  } else {
    return networkFirstStrategy(request);
  }
}
```

---

## Step 916: Cache First Strategy

```javascript
// Cache First - เหมาะสำหรับ static assets ที่ไม่ค่อยเปลี่ยน
// 1. ตรวจสอบ cache ก่อน
// 2. ถ้าไม่มี ดึงจาก network และ cache ไว้

async function cacheFirstStrategy(request, cacheName = "static-cache-v1") {
  // ตรวจสอบ cache ก่อน
  const cachedResponse = await caches.match(request);
  
  if (cachedResponse) {
    return cachedResponse;
  }
  
  // ไม่มีใน cache - ดึงจาก network
  try {
    const networkResponse = await fetch(request);
    
    if (networkResponse.ok) {
      // บันทึกลง cache สำหรับครั้งต่อไป
      const cache = await caches.open(cacheName);
      cache.put(request, networkResponse.clone());
    }
    
    return networkResponse;
    
  } catch (error) {
    // ไม่มี network และไม่มี cache
    console.error("Cache First ล้มเหลว:", error);
    
    // ส่ง offline page
    const offlinePage = await caches.match("/offline.html");
    return offlinePage || new Response("Offline", {
      status: 503,
      statusText: "Service Unavailable"
    });
  }
}
```

---

## Step 917: Network First Strategy

```javascript
// Network First - เหมาะสำหรับ dynamic content ที่ต้องการข้อมูลล่าสุด
// 1. ดึงจาก network ก่อน
// 2. ถ้า network ล้มเหลว ดึงจาก cache

async function networkFirstStrategy(request, cacheName = "dynamic-cache-v1") {
  try {
    const networkResponse = await fetch(request);
    
    if (networkResponse.ok) {
      // บันทึกลง cache
      const cache = await caches.open(cacheName);
      cache.put(request, networkResponse.clone());
    }
    
    return networkResponse;
    
  } catch (error) {
    // Network ล้มเหลว - ดึงจาก cache
    const cachedResponse = await caches.match(request);
    
    if (cachedResponse) {
      console.log("ใช้ cached version:", request.url);
      return cachedResponse;
    }
    
    // ไม่มีทั้ง network และ cache
    if (request.destination === "document") {
      return caches.match("/offline.html");
    }
    
    return new Response(JSON.stringify({ error: "Offline", offline: true }), {
      status: 503,
      headers: { "Content-Type": "application/json" }
    });
  }
}

// Network First with timeout
async function networkFirstWithTimeout(request, timeoutMs = 3000) {
  const cache = await caches.open("dynamic-cache-v1");
  
  const networkPromise = fetch(request).then(response => {
    if (response.ok) {
      cache.put(request, response.clone());
    }
    return response;
  });
  
  const timeoutPromise = new Promise((_, reject) =>
    setTimeout(() => reject(new Error("Network timeout")), timeoutMs)
  );
  
  try {
    return await Promise.race([networkPromise, timeoutPromise]);
  } catch (error) {
    const cached = await cache.match(request);
    return cached || new Response("Offline", { status: 503 });
  }
}
```

---

## Step 918: Stale-While-Revalidate Strategy

```javascript
// Stale-While-Revalidate - ตอบสนองด้วย cache ทันที แต่ update ใน background
// เหมาะสำหรับ content ที่โอเคถ้าข้อมูลเก่าเล็กน้อย

async function staleWhileRevalidateStrategy(request, cacheName = "swr-cache-v1") {
  const cache = await caches.open(cacheName);
  const cachedResponse = await cache.match(request);
  
  // ส่ง fetch request ใน background เสมอ
  const fetchPromise = fetch(request)
    .then(networkResponse => {
      if (networkResponse.ok) {
        cache.put(request, networkResponse.clone());
      }
      return networkResponse;
    })
    .catch(error => {
      console.warn("Background fetch ล้มเหลว:", error.message);
    });
  
  // ถ้ามี cache ส่งทันที (stale), network update ใน background
  if (cachedResponse) {
    return cachedResponse;
  }
  
  // ไม่มี cache รอ network
  return fetchPromise;
}

// ตัวอย่าง complete sw.js ด้วย multiple strategies
const STRATEGIES = {
  // Static assets - Cache First
  STATIC: ["/styles/", "/scripts/", "/fonts/", "/images/"],
  // API calls - Network First  
  API: ["/api/"],
  // HTML pages - Stale-While-Revalidate
  PAGES: ["/"],
};

self.addEventListener("fetch", (event) => {
  const url = new URL(event.request.url);
  
  if (url.origin !== location.origin) return;
  
  const path = url.pathname;
  
  if (STRATEGIES.STATIC.some(p => path.startsWith(p)) || 
      path.match(/\.(css|js|png|jpg|svg|woff2)$/)) {
    event.respondWith(cacheFirstStrategy(event.request));
  } else if (STRATEGIES.API.some(p => path.startsWith(p))) {
    event.respondWith(networkFirstStrategy(event.request));
  } else {
    event.respondWith(staleWhileRevalidateStrategy(event.request));
  }
});
```

---

## Step 919: Cache Only และ Network Only Strategies

```javascript
// Cache Only - ใช้เฉพาะ cache ไม่ต่อ network เลย
// เหมาะสำหรับ assets ที่ pre-cached ใน install event

async function cacheOnlyStrategy(request) {
  const cachedResponse = await caches.match(request);
  
  if (cachedResponse) {
    return cachedResponse;
  }
  
  return new Response("ไม่พบใน cache", { status: 404 });
}

// Network Only - ใช้ network เสมอ ไม่ cache
// เหมาะสำหรับ analytics, non-GET requests

async function networkOnlyStrategy(request) {
  try {
    return await fetch(request);
  } catch (error) {
    return new Response("Network Error", { status: 503 });
  }
}

// ตัวอย่าง complete strategy implementation
class CacheStrategy {
  static async cacheFirst(request, options = {}) {
    const { cacheName = "default", maxAge = Infinity } = options;
    
    const cache = await caches.open(cacheName);
    const cached = await cache.match(request);
    
    if (cached) {
      const cachedTime = cached.headers.get("sw-cached-time");
      if (cachedTime && Date.now() - parseInt(cachedTime) < maxAge) {
        return cached;
      }
    }
    
    try {
      const response = await fetch(request);
      if (response.ok) {
        const responseToCache = new Response(response.body, {
          status: response.status,
          statusText: response.statusText,
          headers: {
            ...Object.fromEntries(response.headers.entries()),
            "sw-cached-time": Date.now().toString()
          }
        });
        await cache.put(request, responseToCache);
      }
      return response;
    } catch {
      return cached || Response.error();
    }
  }
  
  static async networkFirst(request, options = {}) {
    const { cacheName = "default", timeout = 5000 } = options;
    const cache = await caches.open(cacheName);
    
    try {
      const controller = new AbortController();
      const timeoutId = setTimeout(() => controller.abort(), timeout);
      
      const response = await fetch(request, { signal: controller.signal });
      clearTimeout(timeoutId);
      
      if (response.ok) {
        await cache.put(request, response.clone());
      }
      return response;
    } catch {
      const cached = await cache.match(request);
      return cached || Response.error();
    }
  }
}
```

---

## Step 920: Offline Functionality

```javascript
// sw.js - Offline support

const OFFLINE_PAGE = "/offline.html";
const OFFLINE_FALLBACKS = {
  document: OFFLINE_PAGE,
  image: "/images/offline-placeholder.svg",
  font: "/fonts/fallback.woff2"
};

// Pre-cache offline fallbacks
self.addEventListener("install", (event) => {
  event.waitUntil(
    caches.open("offline-fallbacks").then(cache =>
      cache.addAll(Object.values(OFFLINE_FALLBACKS))
    )
  );
});

self.addEventListener("fetch", (event) => {
  event.respondWith(
    fetch(event.request).catch(async () => {
      // Network failed - serve offline fallback
      const destination = event.request.destination;
      const fallbackUrl = OFFLINE_FALLBACKS[destination];
      
      if (fallbackUrl) {
        const fallback = await caches.match(fallbackUrl);
        if (fallback) return fallback;
      }
      
      // Generic offline response
      return new Response(
        JSON.stringify({ error: "You are offline", offline: true }),
        {
          status: 503,
          headers: { "Content-Type": "application/json" }
        }
      );
    })
  );
});
```

```html
<!-- offline.html -->
<!DOCTYPE html>
<html lang="th">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>ไม่มีการเชื่อมต่ออินเทอร์เน็ต</title>
  <style>
    body {
      font-family: 'Sarabun', sans-serif;
      display: flex;
      justify-content: center;
      align-items: center;
      min-height: 100vh;
      margin: 0;
      background: #f5f5f5;
    }
    .container {
      text-align: center;
      padding: 2rem;
      background: white;
      border-radius: 12px;
      box-shadow: 0 4px 20px rgba(0,0,0,0.1);
      max-width: 400px;
    }
    .icon { font-size: 4rem; margin-bottom: 1rem; }
    h1 { color: #333; }
    p { color: #666; }
    button {
      background: #007bff;
      color: white;
      border: none;
      padding: 0.75rem 1.5rem;
      border-radius: 8px;
      cursor: pointer;
      font-size: 1rem;
    }
  </style>
</head>
<body>
  <div class="container">
    <div class="icon">📡</div>
    <h1>ไม่มีการเชื่อมต่ออินเทอร์เน็ต</h1>
    <p>กรุณาตรวจสอบการเชื่อมต่อของคุณ แล้วลองใหม่อีกครั้ง</p>
    <button onclick="window.location.reload()">ลองใหม่</button>
  </div>
</body>
</html>
```

---

## Step 921: Background Sync

```javascript
// sw.js - Background Sync

self.addEventListener("sync", (event) => {
  console.log("Background Sync event:", event.tag);
  
  if (event.tag === "sync-messages") {
    event.waitUntil(syncMessages());
  } else if (event.tag === "sync-form-data") {
    event.waitUntil(syncFormData());
  }
});

async function syncMessages() {
  try {
    // ดึง messages ที่รอส่งจาก IndexedDB
    const pendingMessages = await getPendingMessages();
    
    for (const message of pendingMessages) {
      try {
        const response = await fetch("/api/messages", {
          method: "POST",
          headers: { "Content-Type": "application/json" },
          body: JSON.stringify(message)
        });
        
        if (response.ok) {
          await deletePendingMessage(message.id);
          console.log(`Message ${message.id} ส่งสำเร็จ`);
        }
      } catch (error) {
        console.error(`ส่ง message ${message.id} ไม่สำเร็จ:`, error);
        throw error; // re-throw เพื่อให้ sync ลองใหม่
      }
    }
    
    // แจ้ง clients ว่า sync เสร็จแล้ว
    const clients = await self.clients.matchAll();
    clients.forEach(client => {
      client.postMessage({ type: "SYNC_COMPLETE", tag: "sync-messages" });
    });
    
  } catch (error) {
    console.error("Background Sync ล้มเหลว:", error);
    throw error;
  }
}

// ฟังก์ชัน helper สำหรับ IndexedDB
function openDB() {
  return new Promise((resolve, reject) => {
    const request = indexedDB.open("offline-db", 1);
    request.onsuccess = (e) => resolve(e.target.result);
    request.onerror = (e) => reject(e.target.error);
    request.onupgradeneeded = (e) => {
      const db = e.target.result;
      if (!db.objectStoreNames.contains("pendingMessages")) {
        db.createObjectStore("pendingMessages", { keyPath: "id", autoIncrement: true });
      }
    };
  });
}

async function getPendingMessages() {
  const db = await openDB();
  return new Promise((resolve, reject) => {
    const tx = db.transaction("pendingMessages", "readonly");
    const request = tx.objectStore("pendingMessages").getAll();
    request.onsuccess = (e) => resolve(e.target.result);
    request.onerror = (e) => reject(e.target.error);
  });
}

async function deletePendingMessage(id) {
  const db = await openDB();
  return new Promise((resolve, reject) => {
    const tx = db.transaction("pendingMessages", "readwrite");
    const request = tx.objectStore("pendingMessages").delete(id);
    request.onsuccess = () => resolve();
    request.onerror = (e) => reject(e.target.error);
  });
}
```

```javascript
// main.js - ส่ง message แบบ offline-first
async function sendMessage(messageData) {
  if (navigator.onLine) {
    try {
      const response = await fetch("/api/messages", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(messageData)
      });
      return response.json();
    } catch (error) {
      await saveForLater(messageData);
      await scheduleSync();
    }
  } else {
    await saveForLater(messageData);
    await scheduleSync();
  }
}

async function saveForLater(data) {
  const db = await openDB();
  return new Promise((resolve, reject) => {
    const tx = db.transaction("pendingMessages", "readwrite");
    const request = tx.objectStore("pendingMessages").add({
      ...data,
      timestamp: Date.now()
    });
    request.onsuccess = () => resolve(request.result);
    request.onerror = (e) => reject(e.target.error);
  });
}

async function scheduleSync() {
  const registration = await navigator.serviceWorker.ready;
  
  if ("sync" in registration) {
    await registration.sync.register("sync-messages");
    console.log("Background Sync ลงทะเบียนแล้ว");
  }
}
```

---

## Step 922: Push Notifications

```javascript
// main.js - ขอ permission และ subscribe

async function subscribeToPushNotifications() {
  // ขอ permission
  const permission = await Notification.requestPermission();
  
  if (permission !== "granted") {
    console.log("ผู้ใช้ไม่อนุญาต push notifications");
    return;
  }
  
  // รับ push subscription
  const registration = await navigator.serviceWorker.ready;
  
  // Public VAPID key จาก server
  const publicVapidKey = "YOUR_PUBLIC_VAPID_KEY";
  
  const subscription = await registration.pushManager.subscribe({
    userVisibleOnly: true,
    applicationServerKey: urlBase64ToUint8Array(publicVapidKey)
  });
  
  console.log("Push Subscription:", subscription);
  
  // ส่ง subscription ไปยัง server
  await fetch("/api/push-subscribe", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(subscription)
  });
  
  return subscription;
}

// Helper: แปลง base64 เป็น Uint8Array
function urlBase64ToUint8Array(base64String) {
  const padding = "=".repeat((4 - base64String.length % 4) % 4);
  const base64 = (base64String + padding)
    .replace(/-/g, "+")
    .replace(/_/g, "/");
  
  const rawData = window.atob(base64);
  const outputArray = new Uint8Array(rawData.length);
  
  for (let i = 0; i < rawData.length; ++i) {
    outputArray[i] = rawData.charCodeAt(i);
  }
  return outputArray;
}
```

```javascript
// sw.js - รับ Push notifications

self.addEventListener("push", (event) => {
  console.log("ได้รับ Push notification");
  
  let data = {};
  
  if (event.data) {
    try {
      data = event.data.json();
    } catch {
      data = { title: "Notification", body: event.data.text() };
    }
  }
  
  const options = {
    body: data.body || "คุณมีการแจ้งเตือนใหม่",
    icon: data.icon || "/images/notification-icon.png",
    badge: "/images/badge.png",
    image: data.image,
    vibrate: [100, 50, 100],
    data: {
      url: data.url || "/",
      timestamp: Date.now()
    },
    actions: [
      { action: "view", title: "ดู", icon: "/images/view.png" },
      { action: "dismiss", title: "ปิด", icon: "/images/dismiss.png" }
    ],
    tag: data.tag || "default",
    requireInteraction: data.requireInteraction || false
  };
  
  event.waitUntil(
    self.registration.showNotification(data.title || "แจ้งเตือน", options)
  );
});

// จัดการเมื่อผู้ใช้คลิก notification
self.addEventListener("notificationclick", (event) => {
  event.notification.close();
  
  const action = event.action;
  const url = event.notification.data?.url || "/";
  
  if (action === "dismiss") {
    return;
  }
  
  event.waitUntil(
    clients.matchAll({ type: "window", includeUncontrolled: true })
      .then((clientList) => {
        // เปิดแท็บที่มีอยู่แล้ว
        for (const client of clientList) {
          if (client.url === url && "focus" in client) {
            return client.focus();
          }
        }
        // เปิดแท็บใหม่
        if (clients.openWindow) {
          return clients.openWindow(url);
        }
      })
  );
});
```

---

## Step 923: Service Worker Communication

```javascript
// การสื่อสารระหว่าง main thread และ Service Worker

// main.js - ส่งข้อความไปยัง Service Worker
async function sendMessageToSW(message) {
  const registration = await navigator.serviceWorker.ready;
  
  if (registration.active) {
    registration.active.postMessage(message);
  }
}

// รับข้อความจาก Service Worker
navigator.serviceWorker.addEventListener("message", (event) => {
  console.log("ได้รับจาก SW:", event.data);
  
  const { type, payload } = event.data;
  
  switch (type) {
    case "CACHE_UPDATED":
      showNotification("Content updated! Refresh to see changes.");
      break;
    case "SYNC_COMPLETE":
      updateSyncStatus("เสร็จแล้ว");
      break;
    case "OFFLINE":
      showOfflineBanner();
      break;
  }
});

// ส่งข้อความพร้อม reply
function sendAndWait(message) {
  return new Promise((resolve, reject) => {
    const messageChannel = new MessageChannel();
    
    messageChannel.port1.onmessage = (event) => {
      if (event.data.error) {
        reject(event.data.error);
      } else {
        resolve(event.data.result);
      }
    };
    
    navigator.serviceWorker.controller?.postMessage(message, [messageChannel.port2]);
    
    setTimeout(() => reject(new Error("Timeout")), 5000);
  });
}

// การใช้งาน
sendAndWait({ type: "GET_CACHE_SIZE" })
  .then(size => console.log("Cache size:", size))
  .catch(err => console.error(err));
```

```javascript
// sw.js - รับและตอบกลับข้อความ

self.addEventListener("message", (event) => {
  const { type, payload } = event.data;
  const port = event.ports[0];
  
  switch (type) {
    case "SKIP_WAITING":
      self.skipWaiting();
      break;
      
    case "GET_CACHE_SIZE":
      getCacheSize().then(size => {
        if (port) port.postMessage({ result: size });
      });
      break;
      
    case "CLEAR_CACHE":
      clearAllCaches().then(() => {
        if (port) port.postMessage({ result: "cleared" });
        broadcastToClients({ type: "CACHE_CLEARED" });
      });
      break;
      
    case "GET_VERSION":
      if (port) port.postMessage({ result: CACHE_VERSION });
      break;
  }
});

async function getCacheSize() {
  const keys = await caches.keys();
  let totalSize = 0;
  
  for (const key of keys) {
    const cache = await caches.open(key);
    const requests = await cache.keys();
    
    for (const request of requests) {
      const response = await cache.match(request);
      if (response) {
        const blob = await response.blob();
        totalSize += blob.size;
      }
    }
  }
  
  return {
    bytes: totalSize,
    formatted: formatBytes(totalSize)
  };
}

async function broadcastToClients(message) {
  const clients = await self.clients.matchAll();
  clients.forEach(client => client.postMessage(message));
}

function formatBytes(bytes) {
  if (bytes < 1024) return `${bytes} B`;
  if (bytes < 1024 * 1024) return `${(bytes / 1024).toFixed(1)} KB`;
  return `${(bytes / (1024 * 1024)).toFixed(1)} MB`;
}
```

---

## Step 924: Workbox Library Overview

```javascript
// workbox.js - ใช้ Google Workbox สำหรับ Service Worker ง่ายขึ้น

// import Workbox modules
importScripts("https://storage.googleapis.com/workbox-cdn/releases/6.5.4/workbox-sw.js");

const { precacheAndRoute } = workbox.precaching;
const { registerRoute } = workbox.routing;
const { CacheFirst, NetworkFirst, StaleWhileRevalidate } = workbox.strategies;
const { ExpirationPlugin } = workbox.expiration;
const { BackgroundSyncPlugin } = workbox.backgroundSync;

// Pre-cache static files
precacheAndRoute([
  { url: "/index.html", revision: "abc123" },
  { url: "/styles/main.css", revision: "def456" },
  { url: "/scripts/app.js", revision: "ghi789" }
]);

// Cache images with Cache First + expiration
registerRoute(
  ({ request }) => request.destination === "image",
  new CacheFirst({
    cacheName: "images",
    plugins: [
      new ExpirationPlugin({
        maxEntries: 50,
        maxAgeSeconds: 30 * 24 * 60 * 60, // 30 days
      })
    ]
  })
);

// Cache API with Network First + background sync
const bgSyncPlugin = new BackgroundSyncPlugin("api-queue", {
  maxRetentionTime: 24 * 60 // 24 hours
});

registerRoute(
  ({ url }) => url.pathname.startsWith("/api/"),
  new NetworkFirst({
    cacheName: "api-cache",
    networkTimeoutSeconds: 3,
    plugins: [bgSyncPlugin]
  }),
  "POST"
);

// Cache fonts with Stale-While-Revalidate
registerRoute(
  ({ request }) => request.destination === "font",
  new StaleWhileRevalidate({
    cacheName: "fonts",
    plugins: [
      new ExpirationPlugin({ maxAgeSeconds: 60 * 60 * 24 * 365 })
    ]
  })
);
```

---

## Step 925: Debugging Service Workers

```javascript
// sw-debug.js - เพิ่ม debug logging

const DEBUG = true;

function log(...args) {
  if (DEBUG) console.log("[SW]", ...args);
}

self.addEventListener("install", (event) => {
  log("Installing...");
  event.waitUntil(
    install().then(() => log("Install complete"))
         .catch(err => log("Install failed:", err))
  );
});

self.addEventListener("activate", (event) => {
  log("Activating...");
  event.waitUntil(
    activate().then(() => log("Activate complete"))
  );
});

self.addEventListener("fetch", (event) => {
  const url = new URL(event.request.url);
  log(`Fetch: ${event.request.method} ${url.pathname}`);
  
  event.respondWith(
    handleFetch(event.request).then(response => {
      log(`Response: ${response.status} for ${url.pathname}`);
      return response;
    })
  );
});

// Tools สำหรับ debug ใน browser:
// 1. Chrome DevTools > Application > Service Workers
// 2. about:serviceworker (Firefox)
// 3. chrome://serviceworker-internals/

// การ test offline mode:
// DevTools > Network > Offline checkbox
// หรือ DevTools > Application > Service Workers > Offline checkbox
```

```javascript
// main.js - Debug helpers

class ServiceWorkerDebug {
  static async getRegistrations() {
    const registrations = await navigator.serviceWorker.getRegistrations();
    return registrations.map(reg => ({
      scope: reg.scope,
      state: reg.active?.state || "unknown",
      scriptURL: reg.active?.scriptURL
    }));
  }
  
  static async unregisterAll() {
    const registrations = await navigator.serviceWorker.getRegistrations();
    await Promise.all(registrations.map(reg => reg.unregister()));
    console.log(`Unregistered ${registrations.length} Service Workers`);
  }
  
  static async clearAllCaches() {
    const keys = await caches.keys();
    await Promise.all(keys.map(key => caches.delete(key)));
    console.log(`Cleared ${keys.length} caches`);
  }
  
  static async getCacheInfo() {
    const keys = await caches.keys();
    const info = {};
    
    for (const key of keys) {
      const cache = await caches.open(key);
      const requests = await cache.keys();
      info[key] = {
        count: requests.length,
        urls: requests.map(r => r.url)
      };
    }
    
    return info;
  }
  
  static logCacheStatus() {
    // เพิ่ม debug panel ใน page
    const panel = document.createElement("div");
    panel.id = "sw-debug";
    panel.style.cssText = `
      position: fixed; bottom: 0; right: 0;
      background: rgba(0,0,0,0.8); color: #0f0;
      font-family: monospace; font-size: 12px;
      padding: 10px; z-index: 9999;
      max-width: 300px; max-height: 200px;
      overflow: auto;
    `;
    
    const update = async () => {
      const regs = await ServiceWorkerDebug.getRegistrations();
      const cacheInfo = await ServiceWorkerDebug.getCacheInfo();
      
      panel.innerHTML = `
        <strong>SW Debug</strong><br>
        SW: ${regs.length > 0 ? "Active" : "None"}<br>
        Caches: ${Object.keys(cacheInfo).length}<br>
        ${Object.entries(cacheInfo).map(([k, v]) => `${k}: ${v.count} items`).join("<br>")}
      `;
    };
    
    document.body.appendChild(panel);
    update();
    setInterval(update, 5000);
  }
}
```

---

## Step 926: Complete PWA Service Worker

```javascript
// sw.js - Complete PWA implementation

const APP_VERSION = "1.2.0";
const STATIC_CACHE = `static-${APP_VERSION}`;
const DYNAMIC_CACHE = `dynamic-${APP_VERSION}`;
const API_CACHE = `api-${APP_VERSION}`;

const STATIC_ASSETS = [
  "/",
  "/index.html",
  "/manifest.json",
  "/styles/main.css",
  "/scripts/app.js",
  "/offline.html",
  "/images/icon-192.png",
  "/images/icon-512.png"
];

// Install
self.addEventListener("install", event => {
  event.waitUntil(
    Promise.all([
      caches.open(STATIC_CACHE).then(cache => cache.addAll(STATIC_ASSETS)),
      self.skipWaiting()
    ])
  );
});

// Activate
self.addEventListener("activate", event => {
  event.waitUntil(
    Promise.all([
      // ลบ caches เก่า
      caches.keys().then(keys =>
        Promise.all(
          keys
            .filter(key => ![STATIC_CACHE, DYNAMIC_CACHE, API_CACHE].includes(key))
            .map(key => caches.delete(key))
        )
      ),
      self.clients.claim()
    ])
  );
});

// Fetch
self.addEventListener("fetch", event => {
  const { request } = event;
  const url = new URL(request.url);
  
  // ข้าม non-same-origin requests
  if (url.origin !== location.origin && !url.hostname.includes("api.myapp.com")) {
    return;
  }
  
  // API routes
  if (url.pathname.startsWith("/api/")) {
    event.respondWith(handleAPIRequest(request));
    return;
  }
  
  // Static assets
  if (STATIC_ASSETS.some(asset => url.pathname === asset || url.pathname.endsWith(asset))) {
    event.respondWith(cacheFirst(request, STATIC_CACHE));
    return;
  }
  
  // Dynamic content
  event.respondWith(staleWhileRevalidate(request));
});

async function handleAPIRequest(request) {
  if (request.method !== "GET") {
    // POST/PUT/DELETE - try network, queue if offline
    try {
      return await fetch(request);
    } catch {
      // Queue for background sync
      await queueRequest(request);
      return new Response(JSON.stringify({ queued: true }), {
        headers: { "Content-Type": "application/json" }
      });
    }
  }
  
  return networkFirstWithTimeout(request, 3000);
}

async function cacheFirst(request, cacheName) {
  const cached = await caches.match(request);
  if (cached) return cached;
  
  const response = await fetch(request);
  const cache = await caches.open(cacheName);
  cache.put(request, response.clone());
  return response;
}

async function networkFirstWithTimeout(request, ms) {
  const cache = await caches.open(API_CACHE);
  
  try {
    const controller = new AbortController();
    setTimeout(() => controller.abort(), ms);
    
    const response = await fetch(request, { signal: controller.signal });
    if (response.ok) cache.put(request, response.clone());
    return response;
  } catch {
    return await cache.match(request) || 
           new Response(JSON.stringify({ offline: true }), { 
             status: 503,
             headers: { "Content-Type": "application/json" }
           });
  }
}

async function staleWhileRevalidate(request) {
  const cache = await caches.open(DYNAMIC_CACHE);
  const cached = await cache.match(request);
  
  const fetchPromise = fetch(request).then(response => {
    if (response.ok) cache.put(request, response.clone());
    return response;
  });
  
  return cached || await fetchPromise;
}

async function queueRequest(request) {
  // บันทึก request ลง IndexedDB
  const db = await openOfflineDB();
  const requestData = {
    url: request.url,
    method: request.method,
    headers: Object.fromEntries(request.headers.entries()),
    body: await request.text(),
    timestamp: Date.now()
  };
  
  const tx = db.transaction("pendingRequests", "readwrite");
  tx.objectStore("pendingRequests").add(requestData);
}
```

---

## Step 927: Web App Manifest

```json
// manifest.json - สำหรับ PWA
{
  "name": "My JavaScript App",
  "short_name": "JSApp",
  "description": "แอปพลิเคชัน JavaScript ที่รองรับ offline",
  "start_url": "/",
  "display": "standalone",
  "background_color": "#ffffff",
  "theme_color": "#007bff",
  "orientation": "portrait",
  "lang": "th",
  "icons": [
    {
      "src": "/images/icon-72.png",
      "sizes": "72x72",
      "type": "image/png",
      "purpose": "maskable any"
    },
    {
      "src": "/images/icon-192.png",
      "sizes": "192x192",
      "type": "image/png",
      "purpose": "maskable any"
    },
    {
      "src": "/images/icon-512.png",
      "sizes": "512x512",
      "type": "image/png",
      "purpose": "maskable any"
    }
  ],
  "screenshots": [
    {
      "src": "/screenshots/desktop.png",
      "sizes": "1280x720",
      "type": "image/png",
      "form_factor": "wide"
    }
  ],
  "shortcuts": [
    {
      "name": "เพิ่มรายการใหม่",
      "url": "/new",
      "icons": [{ "src": "/images/new-icon.png", "sizes": "96x96" }]
    }
  ]
}
```

---

## Step 928: Periodic Background Sync

```javascript
// sw.js - Periodic Background Sync
self.addEventListener("periodicsync", (event) => {
  console.log("Periodic Sync:", event.tag);
  
  if (event.tag === "update-news") {
    event.waitUntil(updateNewsCache());
  } else if (event.tag === "sync-data") {
    event.waitUntil(syncAllData());
  }
});

async function updateNewsCache() {
  try {
    const response = await fetch("/api/latest-news");
    if (!response.ok) throw new Error("Fetch failed");
    
    const cache = await caches.open("news-cache");
    await cache.put("/api/latest-news", response);
    
    // แจ้ง clients
    const clients = await self.clients.matchAll();
    clients.forEach(client => 
      client.postMessage({ type: "NEWS_UPDATED" })
    );
    
  } catch (error) {
    console.error("Periodic sync failed:", error);
  }
}

// main.js - ลงทะเบียน Periodic Sync
async function registerPeriodicSync() {
  const registration = await navigator.serviceWorker.ready;
  
  if ("periodicSync" in registration) {
    const status = await navigator.permissions.query({ name: "periodic-background-sync" });
    
    if (status.state === "granted") {
      await registration.periodicSync.register("update-news", {
        minInterval: 24 * 60 * 60 * 1000 // อย่างน้อยทุก 24 ชั่วโมง
      });
      console.log("Periodic sync ลงทะเบียนสำเร็จ");
    }
  }
}
```

---

## Step 929: Service Worker ในการทำ Prefetch

```javascript
// sw.js - Predictive Prefetch

self.addEventListener("message", (event) => {
  if (event.data.type === "PREFETCH") {
    prefetchResources(event.data.urls);
  }
});

async function prefetchResources(urls) {
  const cache = await caches.open(DYNAMIC_CACHE);
  
  const prefetchPromises = urls.map(async (url) => {
    // ตรวจว่ามีใน cache แล้วหรือยัง
    const cached = await cache.match(url);
    if (cached) return;
    
    try {
      const response = await fetch(url);
      if (response.ok) {
        await cache.put(url, response);
        console.log(`Prefetched: ${url}`);
      }
    } catch (error) {
      console.warn(`Prefetch ล้มเหลว: ${url}`);
    }
  });
  
  await Promise.allSettled(prefetchPromises);
}

// main.js - ส่ง URLs ให้ prefetch
function prefetchNextPage(links) {
  if ("serviceWorker" in navigator && navigator.serviceWorker.controller) {
    navigator.serviceWorker.controller.postMessage({
      type: "PREFETCH",
      urls: links.map(link => link.href)
    });
  }
}

// Prefetch เมื่อ hover บน link
document.querySelectorAll("a[data-prefetch]").forEach(link => {
  link.addEventListener("mouseenter", () => {
    prefetchNextPage([link]);
  }, { once: true });
});
```

---

## Step 930: Best Practices และ Advanced Patterns

```javascript
// best-practices-sw.js

// 1. ใช้ versioning สำหรับ cache names
const VERSION = "v1.0.0";
const CACHES = {
  static: `app-static-${VERSION}`,
  dynamic: `app-dynamic-${VERSION}`,
  api: `app-api-${VERSION}`
};

// 2. Handle navigation requests อย่างถูกต้อง
self.addEventListener("fetch", event => {
  if (event.request.mode === "navigate") {
    event.respondWith(handleNavigation(event.request));
  }
});

async function handleNavigation(request) {
  try {
    const networkResponse = await fetch(request);
    
    // Cache HTML หน้าใหม่
    if (networkResponse.ok) {
      const cache = await caches.open(CACHES.dynamic);
      cache.put(request, networkResponse.clone());
    }
    
    return networkResponse;
  } catch {
    // Offline - ส่ง cached version หรือ offline page
    const cached = await caches.match(request);
    return cached || caches.match("/offline.html");
  }
}

// 3. Efficient cache management
async function pruneCache(cacheName, maxItems = 30) {
  const cache = await caches.open(cacheName);
  const requests = await cache.keys();
  
  if (requests.length > maxItems) {
    // ลบรายการเก่าที่สุด
    const itemsToDelete = requests.slice(0, requests.length - maxItems);
    await Promise.all(itemsToDelete.map(req => cache.delete(req)));
  }
}

// 4. Cache with metadata
async function cacheWithMetadata(cacheName, request, response) {
  const cache = await caches.open(cacheName);
  
  // เพิ่ม metadata headers
  const headers = new Headers(response.headers);
  headers.set("sw-cache-date", new Date().toISOString());
  headers.set("sw-cache-version", VERSION);
  
  const responseWithMeta = new Response(response.body, {
    status: response.status,
    statusText: response.statusText,
    headers
  });
  
  await cache.put(request, responseWithMeta);
}

// 5. Progressive loading
self.addEventListener("fetch", event => {
  if (event.request.destination === "image") {
    event.respondWith(
      fetchWithFallback(
        event.request,
        "/images/placeholder.svg"
      )
    );
  }
});

async function fetchWithFallback(request, fallbackUrl) {
  try {
    const cached = await caches.match(request);
    if (cached) return cached;
    
    const response = await fetch(request);
    if (response.ok) {
      const cache = await caches.open(CACHES.dynamic);
      cache.put(request, response.clone());
    }
    return response;
  } catch {
    return caches.match(fallbackUrl);
  }
}
```

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: สร้าง PWA เต็มรูปแบบ
สร้างเว็บแอป Todo List ที่ทำงานได้แบบ offline พร้อม Service Worker

```javascript
// Todo App Service Worker
const CACHE_NAME = "todo-app-v1";
const OFFLINE_URLS = ["/", "/index.html", "/app.css", "/app.js"];

self.addEventListener("install", event => {
  event.waitUntil(
    caches.open(CACHE_NAME).then(cache => cache.addAll(OFFLINE_URLS))
  );
});

self.addEventListener("activate", event => {
  event.waitUntil(
    caches.keys().then(keys =>
      Promise.all(keys.filter(k => k !== CACHE_NAME).map(k => caches.delete(k)))
    )
  );
  return self.clients.claim();
});

self.addEventListener("fetch", event => {
  event.respondWith(
    caches.match(event.request).then(cached => cached || fetch(event.request))
  );
});
```

### แบบฝึกหัดที่ 2: Cache Strategy ที่ชาญฉลาด
สร้าง Service Worker ที่:
- Cache รูปภาพแบบ Cache First
- Cache API แบบ Network First + fallback
- Auto-cleanup cache เมื่อเกิน 50 items

### แบบฝึกหัดที่ 3: Push Notification System
สร้างระบบ Push Notification พร้อม
- Subscribe/Unsubscribe
- Handle notification click
- Update badge count

---

## สรุป Part 47

Service Workers เป็นหัวใจสำคัญของ Progressive Web Apps:

1. **Lifecycle**: register → install → activate → fetch/push/sync
2. **Cache Strategies**: เลือกให้เหมาะกับประเภทข้อมูล
3. **Offline Support**: ผู้ใช้ยังใช้งานได้แม้ไม่มีอินเทอร์เน็ต
4. **Background Sync**: ส่งข้อมูลเมื่อออนไลน์อีกครั้ง
5. **Push Notifications**: แจ้งเตือนผู้ใช้แม้ปิดเว็บแล้ว

**สิ่งสำคัญ**:
- ต้องใช้ HTTPS เสมอ (ยกเว้น localhost)
- Test ด้วย DevTools > Application > Service Workers
- ใช้ Workbox สำหรับ production
