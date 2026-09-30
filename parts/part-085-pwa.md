# Part 85: Progressive Web Apps (PWA)
## Steps 1671-1690

Progressive Web Apps (PWA) คือ web applications ที่ใช้ web technologies สมัยใหม่เพื่อมอบประสบการณ์
ที่ใกล้เคียงกับ native apps รองรับการทำงาน offline, push notifications, และ install ได้บน device

---

## Step 1671: What is PWA

```
Progressive Web Apps (PWA) คืออะไร?

PWA คือ web app ที่ตอบสนองต่อ progressive enhancement:
- ทำงานได้ดีบน browser ทุกประเภท
- Responsive - ทำงานได้บนทุก device
- Connectivity independent - ทำงาน offline ได้
- App-like - รู้สึกเหมือน native app
- Fresh - อัปเดตอัตโนมัติผ่าน Service Worker
- Safe - ต้องเป็น HTTPS
- Discoverable - ค้นพบได้ผ่าน search engines
- Re-engageable - Push notifications
- Installable - เพิ่มลงหน้าจอหลักได้
- Linkable - แชร์ URL ได้

ข้อดีของ PWA เทียบกับ Native Apps:
✓ ไม่ต้องดาวน์โหลดจาก App Store
✓ ขนาดเล็กกว่ามาก
✓ อัปเดตอัตโนมัติ
✓ Cross-platform เดียวกัน
✓ SEO-friendly
✓ ราคาพัฒนาต่ำกว่า

ข้อจำกัด:
✗ iOS รองรับน้อยกว่า Android
✗ ไม่สามารถเข้าถึง hardware APIs บางอย่าง
✗ Push notification บน iOS ต้องการ iOS 16.4+
✗ Background processing จำกัด
```

---

## Step 1672: PWA Criteria

```javascript
// PWA Checklist

// 1. HTTPS
// เว็บต้องทำงานบน HTTPS เท่านั้น (localhost ยกเว้น)

// 2. Web App Manifest
// ต้องมี manifest.json ที่สมบูรณ์

// 3. Service Worker
// ต้องลงทะเบียน Service Worker

// 4. Installability criteria:
// - มี manifest.json ที่ valid
// - มี Service Worker ที่ลงทะเบียนแล้ว
// - มี icon ที่ใช้ได้
// - มี start_url ที่ทำงานได้ offline
// - display: "standalone" หรือ "fullscreen"

// ตรวจสอบว่า PWA installable ไหม
window.addEventListener("beforeinstallprompt", (e) => {
  console.log("สามารถติดตั้ง PWA ได้!");
});

// ตรวจสอบว่าทำงานในโหมด standalone (installed) หรือเปล่า
function isRunningAsPWA() {
  return window.matchMedia("(display-mode: standalone)").matches ||
         window.navigator.standalone === true; // iOS
}

if (isRunningAsPWA()) {
  console.log("กำลังทำงานในโหมด PWA");
  document.body.classList.add("pwa-mode");
}
```

---

## Step 1673: Web App Manifest

```json
// manifest.json - ไฟล์หลักของ PWA

{
  "name": "แอปพลิเคชั่นของฉัน",
  "short_name": "MyApp",
  "description": "แอปพลิเคชั่นที่ยอดเยี่ยมสำหรับคุณ",
  "start_url": "/",
  "scope": "/",
  "display": "standalone",
  "orientation": "portrait-primary",
  "background_color": "#ffffff",
  "theme_color": "#3367D6",
  "lang": "th",
  "dir": "ltr",
  
  "icons": [
    {
      "src": "/icons/icon-72.png",
      "sizes": "72x72",
      "type": "image/png",
      "purpose": "maskable any"
    },
    {
      "src": "/icons/icon-96.png",
      "sizes": "96x96",
      "type": "image/png",
      "purpose": "maskable any"
    },
    {
      "src": "/icons/icon-128.png",
      "sizes": "128x128",
      "type": "image/png",
      "purpose": "maskable any"
    },
    {
      "src": "/icons/icon-144.png",
      "sizes": "144x144",
      "type": "image/png",
      "purpose": "maskable any"
    },
    {
      "src": "/icons/icon-152.png",
      "sizes": "152x152",
      "type": "image/png",
      "purpose": "maskable any"
    },
    {
      "src": "/icons/icon-192.png",
      "sizes": "192x192",
      "type": "image/png",
      "purpose": "maskable any"
    },
    {
      "src": "/icons/icon-384.png",
      "sizes": "384x384",
      "type": "image/png",
      "purpose": "maskable any"
    },
    {
      "src": "/icons/icon-512.png",
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
      "form_factor": "wide",
      "label": "Desktop view"
    },
    {
      "src": "/screenshots/mobile.png",
      "sizes": "390x844",
      "type": "image/png",
      "form_factor": "narrow",
      "label": "Mobile view"
    }
  ],
  
  "shortcuts": [
    {
      "name": "สร้างรายการใหม่",
      "short_name": "สร้างใหม่",
      "description": "สร้างรายการใหม่อย่างรวดเร็ว",
      "url": "/create",
      "icons": [{ "src": "/icons/add.png", "sizes": "192x192" }]
    },
    {
      "name": "โปรไฟล์ของฉัน",
      "url": "/profile",
      "icons": [{ "src": "/icons/profile.png", "sizes": "192x192" }]
    }
  ],
  
  "categories": ["productivity", "utilities"],
  
  "prefer_related_applications": false,
  
  "protocol_handlers": [
    {
      "protocol": "web+app",
      "url": "/handle?url=%s"
    }
  ],
  
  "share_target": {
    "action": "/share",
    "method": "POST",
    "enctype": "multipart/form-data",
    "params": {
      "title": "title",
      "text": "text",
      "url": "url",
      "files": [
        {
          "name": "media",
          "accept": ["image/*", "video/*"]
        }
      ]
    }
  }
}
```

```html
<!-- เพิ่มใน <head> ของ HTML -->
<link rel="manifest" href="/manifest.json">
<meta name="theme-color" content="#3367D6">
<meta name="apple-mobile-web-app-capable" content="yes">
<meta name="apple-mobile-web-app-status-bar-style" content="black">
<meta name="apple-mobile-web-app-title" content="MyApp">
<link rel="apple-touch-icon" href="/icons/icon-192.png">

<!-- Splash screen สำหรับ iOS -->
<link rel="apple-touch-startup-image" href="/splash/splash-640x1136.png"
  media="(device-width: 320px) and (device-height: 568px) and (-webkit-device-pixel-ratio: 2)">
<link rel="apple-touch-startup-image" href="/splash/splash-750x1334.png"
  media="(device-width: 375px) and (device-height: 667px) and (-webkit-device-pixel-ratio: 2)">
```

---

## Step 1674: Service Worker สำหรับ PWA

```javascript
// service-worker.js - หัวใจของ PWA

const CACHE_NAME = "myapp-v1.0.0";
const STATIC_CACHE = "static-v1.0.0";
const DYNAMIC_CACHE = "dynamic-v1.0.0";

// ไฟล์ที่ต้อง cache ตั้งแต่แรก (App Shell)
const APP_SHELL = [
  "/",
  "/index.html",
  "/offline.html",
  "/css/main.css",
  "/js/app.js",
  "/icons/icon-192.png",
  "/icons/icon-512.png",
  "/manifest.json"
];

// Install event - cache app shell
self.addEventListener("install", (event) => {
  console.log("Service Worker: Installing...");
  
  event.waitUntil(
    Promise.all([
      caches.open(STATIC_CACHE).then(cache => {
        console.log("Caching App Shell...");
        return cache.addAll(APP_SHELL);
      }),
      // Skip waiting เพื่อ activate ทันที
      self.skipWaiting()
    ])
  );
});

// Activate event - ลบ cache เก่า
self.addEventListener("activate", (event) => {
  console.log("Service Worker: Activating...");
  
  event.waitUntil(
    Promise.all([
      // ลบ caches เก่า
      caches.keys().then(cacheNames => {
        return Promise.all(
          cacheNames
            .filter(name => name !== STATIC_CACHE && name !== DYNAMIC_CACHE)
            .map(name => {
              console.log(`Deleting old cache: ${name}`);
              return caches.delete(name);
            })
        );
      }),
      // Claim clients ทันที
      self.clients.claim()
    ])
  );
});

// Fetch event - intercept network requests
self.addEventListener("fetch", (event) => {
  const { request } = event;
  const url = new URL(request.url);
  
  // ไม่ cache requests จาก other origins
  if (url.origin !== location.origin) {
    return;
  }
  
  // Strategy based on request type
  if (request.destination === "document") {
    // HTML: Network First with offline fallback
    event.respondWith(networkFirstWithFallback(request));
  } else if (request.destination === "image") {
    // Images: Cache First
    event.respondWith(cacheFirst(request));
  } else if (url.pathname.startsWith("/api/")) {
    // API calls: Network First with cache fallback
    event.respondWith(networkFirst(request));
  } else {
    // Assets (CSS, JS): Stale While Revalidate
    event.respondWith(staleWhileRevalidate(request));
  }
});

// Cache Strategies

// 1. Cache First - เร็วที่สุด, ดีสำหรับ static assets
async function cacheFirst(request) {
  const cached = await caches.match(request);
  
  if (cached) {
    return cached;
  }
  
  try {
    const response = await fetch(request);
    if (response.ok) {
      const cache = await caches.open(DYNAMIC_CACHE);
      cache.put(request, response.clone());
    }
    return response;
  } catch {
    return new Response("Resource not available offline", { status: 503 });
  }
}

// 2. Network First - ข้อมูลล่าสุดเสมอ, fallback ไปยัง cache
async function networkFirst(request) {
  try {
    const response = await fetch(request);
    
    if (response.ok) {
      const cache = await caches.open(DYNAMIC_CACHE);
      cache.put(request, response.clone());
    }
    
    return response;
  } catch {
    const cached = await caches.match(request);
    return cached || new Response(
      JSON.stringify({ error: "Offline", message: "ไม่มีการเชื่อมต่ออินเทอร์เน็ต" }),
      {
        status: 503,
        headers: { "Content-Type": "application/json" }
      }
    );
  }
}

// 3. Network First with HTML fallback
async function networkFirstWithFallback(request) {
  try {
    const response = await fetch(request);
    
    if (response.ok) {
      const cache = await caches.open(DYNAMIC_CACHE);
      cache.put(request, response.clone());
    }
    
    return response;
  } catch {
    const cached = await caches.match(request);
    if (cached) return cached;
    
    // Fallback ไปยัง offline page
    return caches.match("/offline.html");
  }
}

// 4. Stale While Revalidate - ตอบกลับทันทีจาก cache, อัปเดตใน background
async function staleWhileRevalidate(request) {
  const cached = await caches.match(request);
  
  const networkFetch = fetch(request).then(response => {
    if (response.ok) {
      const cache = caches.open(DYNAMIC_CACHE);
      cache.then(c => c.put(request, response.clone()));
    }
    return response;
  });
  
  return cached || networkFetch;
}
```

---

## Step 1675: การลงทะเบียน Service Worker

```javascript
// main.js - ลงทะเบียน Service Worker

async function registerServiceWorker() {
  if (!("serviceWorker" in navigator)) {
    console.log("Browser ไม่รองรับ Service Worker");
    return null;
  }
  
  try {
    const registration = await navigator.serviceWorker.register("/service-worker.js", {
      scope: "/"
    });
    
    console.log("Service Worker ลงทะเบียนสำเร็จ:", registration.scope);
    
    // ตรวจสอบอัปเดต
    registration.addEventListener("updatefound", () => {
      const newWorker = registration.installing;
      console.log("Service Worker ใหม่กำลัง install...");
      
      newWorker.addEventListener("statechange", () => {
        if (
          newWorker.state === "installed" &&
          navigator.serviceWorker.controller
        ) {
          // Service Worker ใหม่พร้อมใช้งาน แต่ยังไม่ active
          showUpdateNotification();
        }
      });
    });
    
    return registration;
  } catch (error) {
    console.error("Service Worker registration ล้มเหลว:", error);
    return null;
  }
}

// แสดง notification เมื่อมีอัปเดต
function showUpdateNotification() {
  const notification = document.createElement("div");
  notification.className = "update-notification";
  notification.innerHTML = `
    <p>🎉 แอปมีการอัปเดตใหม่!</p>
    <button id="updateBtn">อัปเดตตอนนี้</button>
    <button id="dismissBtn">ทีหลัง</button>
  `;
  document.body.appendChild(notification);
  
  document.getElementById("updateBtn").onclick = () => {
    // บอก Service Worker ใหม่ให้ skipWaiting
    navigator.serviceWorker.getRegistration().then(registration => {
      registration.waiting?.postMessage({ type: "SKIP_WAITING" });
    });
    
    // Reload page
    window.location.reload();
  };
  
  document.getElementById("dismissBtn").onclick = () => {
    notification.remove();
  };
}

// ใน Service Worker: รับ message จาก page
self.addEventListener("message", (event) => {
  if (event.data?.type === "SKIP_WAITING") {
    self.skipWaiting();
  }
});

// รัน
registerServiceWorker();
```

---

## Step 1676: Install Prompt (beforeinstallprompt)

```javascript
// จัดการ install prompt

class PWAInstallManager {
  #deferredPrompt = null;
  #installButton = null;
  #isInstalled = false;
  
  constructor() {
    this.#checkIfInstalled();
    this.#setupEventListeners();
    this.#installButton = document.getElementById("installPWA");
  }
  
  #checkIfInstalled() {
    // ตรวจสอบว่า installed แล้วหรือเปล่า
    this.#isInstalled = 
      window.matchMedia("(display-mode: standalone)").matches ||
      window.navigator.standalone === true;
    
    if (this.#isInstalled) {
      console.log("PWA ติดตั้งแล้ว");
    }
    
    // ฟัง display mode changes
    window.matchMedia("(display-mode: standalone)").addEventListener("change", (e) => {
      this.#isInstalled = e.matches;
      this.#updateUI();
    });
  }
  
  #setupEventListeners() {
    // ดัก beforeinstallprompt event
    window.addEventListener("beforeinstallprompt", (e) => {
      e.preventDefault(); // ป้องกัน auto-prompt
      this.#deferredPrompt = e;
      console.log("Install prompt พร้อมแล้ว");
      this.#updateUI();
    });
    
    // เมื่อ install สำเร็จ
    window.addEventListener("appinstalled", () => {
      this.#deferredPrompt = null;
      this.#isInstalled = true;
      this.#updateUI();
      console.log("PWA ติดตั้งสำเร็จ!");
      
      // Track install event
      this.#trackInstall();
    });
  }
  
  #updateUI() {
    if (!this.#installButton) return;
    
    if (this.#isInstalled) {
      this.#installButton.style.display = "none";
    } else if (this.#deferredPrompt) {
      this.#installButton.style.display = "block";
      this.#installButton.onclick = () => this.promptInstall();
    }
  }
  
  async promptInstall() {
    if (!this.#deferredPrompt) {
      console.log("ไม่มี install prompt");
      return;
    }
    
    // แสดง install prompt
    this.#deferredPrompt.prompt();
    
    // รอผลลัพธ์
    const { outcome } = await this.#deferredPrompt.userChoice;
    
    console.log(`ผู้ใช้ตอบ: ${outcome}`); // "accepted" หรือ "dismissed"
    
    if (outcome === "accepted") {
      console.log("ผู้ใช้ยอมรับการติดตั้ง");
    } else {
      console.log("ผู้ใช้ปฏิเสธการติดตั้ง");
    }
    
    this.#deferredPrompt = null;
  }
  
  #trackInstall() {
    // ส่ง analytics event
    if (window.gtag) {
      window.gtag("event", "pwa_install", {
        event_category: "PWA",
        event_label: "Install Success"
      });
    }
  }
  
  get canInstall() {
    return !!this.#deferredPrompt;
  }
  
  get installed() {
    return this.#isInstalled;
  }
}

// iOS installation instructions (เนื่องจาก iOS ไม่รองรับ beforeinstallprompt)
function showIOSInstallInstructions() {
  const isIOS = /iPad|iPhone|iPod/.test(navigator.userAgent);
  const isInStandaloneMode = window.navigator.standalone;
  
  if (isIOS && !isInStandaloneMode) {
    // แสดง instructions สำหรับ iOS
    showModal({
      title: "ติดตั้งแอปบน iPhone",
      content: `
        <ol>
          <li>กด <strong>Share</strong> (ไอคอน square with arrow)</li>
          <li>เลื่อนลงและกด <strong>"Add to Home Screen"</strong></li>
          <li>กด <strong>"Add"</strong></li>
        </ol>
      `
    });
  }
}

// ใช้งาน
const installManager = new PWAInstallManager();

// แสดปุ่มติดตั้งเมื่อพร้อม
document.getElementById("installBtn").onclick = () => {
  installManager.promptInstall();
};
```

---

## Step 1677: Offline Functionality

```javascript
// Offline support ที่ครอบคลุม

// offline.html - หน้าสำหรับ offline
// ควรเป็น standalone page ที่ทำงานได้โดยไม่ต้องมี network

// ตรวจสอบสถานะ online/offline
class NetworkStatus {
  #listeners = new Set();
  
  constructor() {
    window.addEventListener("online", () => this.#notify(true));
    window.addEventListener("offline", () => this.#notify(false));
  }
  
  get isOnline() {
    return navigator.onLine;
  }
  
  #notify(isOnline) {
    this.#listeners.forEach(listener => listener(isOnline));
    
    if (!isOnline) {
      this.#showOfflineBanner();
    } else {
      this.#hideOfflineBanner();
      this.#syncPendingData();
    }
  }
  
  onChange(listener) {
    this.#listeners.add(listener);
    return () => this.#listeners.delete(listener);
  }
  
  #showOfflineBanner() {
    const banner = document.getElementById("offlineBanner") 
      || this.#createBanner();
    banner.style.display = "block";
  }
  
  #hideOfflineBanner() {
    const banner = document.getElementById("offlineBanner");
    if (banner) banner.style.display = "none";
  }
  
  #createBanner() {
    const banner = document.createElement("div");
    banner.id = "offlineBanner";
    banner.className = "offline-banner";
    banner.textContent = "⚠️ คุณกำลัง offline - ข้อมูลบางส่วนอาจไม่เป็นปัจจุบัน";
    document.body.prepend(banner);
    return banner;
  }
  
  async #syncPendingData() {
    // Sync ข้อมูลที่ค้างอยู่
    const pending = await getPendingSync();
    
    for (const item of pending) {
      try {
        await syncItem(item);
        await removePendingSync(item.id);
      } catch (error) {
        console.error("Sync ล้มเหลว:", error);
      }
    }
  }
}

// IndexedDB สำหรับ offline storage
class OfflineStorage {
  #db = null;
  #dbName = "pwa-offline-db";
  #version = 1;
  
  async init() {
    return new Promise((resolve, reject) => {
      const request = indexedDB.open(this.#dbName, this.#version);
      
      request.onupgradeneeded = (event) => {
        const db = event.target.result;
        
        // Store สำหรับ cached data
        if (!db.objectStoreNames.contains("data")) {
          const dataStore = db.createObjectStore("data", { keyPath: "id" });
          dataStore.createIndex("timestamp", "timestamp");
          dataStore.createIndex("type", "type");
        }
        
        // Store สำหรับ pending sync
        if (!db.objectStoreNames.contains("pendingSync")) {
          const syncStore = db.createObjectStore("pendingSync", { 
            keyPath: "id", 
            autoIncrement: true 
          });
          syncStore.createIndex("createdAt", "createdAt");
          syncStore.createIndex("type", "type");
        }
      };
      
      request.onsuccess = (event) => {
        this.#db = event.target.result;
        resolve();
      };
      
      request.onerror = () => reject(request.error);
    });
  }
  
  async saveData(key, data) {
    return new Promise((resolve, reject) => {
      const tx = this.#db.transaction("data", "readwrite");
      const store = tx.objectStore("data");
      
      const record = {
        id: key,
        data,
        timestamp: Date.now()
      };
      
      const request = store.put(record);
      request.onsuccess = () => resolve();
      request.onerror = () => reject(request.error);
    });
  }
  
  async getData(key) {
    return new Promise((resolve, reject) => {
      const tx = this.#db.transaction("data", "readonly");
      const store = tx.objectStore("data");
      const request = store.get(key);
      
      request.onsuccess = () => resolve(request.result?.data);
      request.onerror = () => reject(request.error);
    });
  }
  
  async addPendingSync(type, payload) {
    return new Promise((resolve, reject) => {
      const tx = this.#db.transaction("pendingSync", "readwrite");
      const store = tx.objectStore("pendingSync");
      
      const record = {
        type,
        payload,
        createdAt: Date.now(),
        attempts: 0
      };
      
      const request = store.add(record);
      request.onsuccess = () => resolve(request.result);
      request.onerror = () => reject(request.error);
    });
  }
  
  async getPendingSync() {
    return new Promise((resolve, reject) => {
      const tx = this.#db.transaction("pendingSync", "readonly");
      const store = tx.objectStore("pendingSync");
      const request = store.getAll();
      
      request.onsuccess = () => resolve(request.result);
      request.onerror = () => reject(request.error);
    });
  }
  
  async removePendingSync(id) {
    return new Promise((resolve, reject) => {
      const tx = this.#db.transaction("pendingSync", "readwrite");
      const store = tx.objectStore("pendingSync");
      const request = store.delete(id);
      
      request.onsuccess = () => resolve();
      request.onerror = () => reject(request.error);
    });
  }
}

// การใช้งาน
const storage = new OfflineStorage();
await storage.init();

// บันทึก API response สำหรับ offline use
async function fetchWithOfflineSupport(url) {
  try {
    const response = await fetch(url);
    const data = await response.json();
    
    // บันทึกไว้สำหรับ offline
    await storage.saveData(url, data);
    return data;
  } catch {
    // คืนค่าจาก cache ถ้า offline
    const cached = await storage.getData(url);
    if (cached) {
      console.log("ใช้ข้อมูลจาก cache (offline)");
      return cached;
    }
    throw new Error("ไม่มีข้อมูล offline");
  }
}
```

---

## Step 1678: Background Sync

```javascript
// Background Sync - sync ข้อมูลเมื่อ online กลับมา

// ใน Service Worker:
self.addEventListener("sync", async (event) => {
  console.log("Background Sync:", event.tag);
  
  if (event.tag === "sync-messages") {
    event.waitUntil(syncMessages());
  } else if (event.tag === "sync-form-data") {
    event.waitUntil(syncFormData());
  }
});

async function syncMessages() {
  const pendingMessages = await getPendingMessages();
  
  const results = await Promise.allSettled(
    pendingMessages.map(async (message) => {
      const response = await fetch("/api/messages", {
        method: "POST",
        headers: { "Content-Type": "application/json" },
        body: JSON.stringify(message)
      });
      
      if (!response.ok) throw new Error(`HTTP ${response.status}`);
      
      await removePendingMessage(message.id);
      return response.json();
    })
  );
  
  const failed = results.filter(r => r.status === "rejected");
  if (failed.length > 0) {
    throw new Error(`${failed.length} messages ส่งไม่สำเร็จ`);
  }
}

// Main thread - ลงทะเบียน background sync
async function scheduleSync(tag) {
  const registration = await navigator.serviceWorker.ready;
  
  try {
    await registration.sync.register(tag);
    console.log(`Background sync ลงทะเบียนแล้ว: ${tag}`);
  } catch (error) {
    // Browser ไม่รองรับ background sync
    console.error("Background sync ไม่รองรับ:", error);
    // Fallback: sync ทันที
    await syncData(tag);
  }
}

// Periodic Background Sync (สำหรับ news/weather apps)
async function schedulePeriodicSync() {
  const registration = await navigator.serviceWorker.ready;
  
  const status = await navigator.permissions.query({
    name: "periodic-background-sync"
  });
  
  if (status.state === "granted") {
    try {
      await registration.periodicSync.register("update-content", {
        minInterval: 24 * 60 * 60 * 1000 // ทุก 24 ชั่วโมง
      });
      console.log("Periodic sync ลงทะเบียนแล้ว");
    } catch (error) {
      console.error("Periodic sync ล้มเหลว:", error);
    }
  }
}

// ใน Service Worker: จัดการ periodic sync
self.addEventListener("periodicsync", (event) => {
  if (event.tag === "update-content") {
    event.waitUntil(updateContent());
  }
});

async function updateContent() {
  // ดึงข้อมูลใหม่และ cache ไว้
  const responses = await Promise.all([
    fetch("/api/news"),
    fetch("/api/weather")
  ]);
  
  const cache = await caches.open(DYNAMIC_CACHE);
  
  for (let i = 0; i < responses.length; i++) {
    if (responses[i].ok) {
      const urls = ["/api/news", "/api/weather"];
      cache.put(urls[i], responses[i]);
    }
  }
}

// ตัวอย่างการใช้งาน - Form submission แบบ offline-capable
class OfflineForm {
  #storage;
  
  constructor(storage) {
    this.#storage = storage;
  }
  
  async submitForm(formData) {
    if (navigator.onLine) {
      // ส่งทันทีถ้า online
      return this.#sendToServer(formData);
    } else {
      // บันทึกไว้และ schedule sync
      const id = await this.#storage.addPendingSync("form-submit", formData);
      await scheduleSync("sync-form-data");
      
      return { success: true, pending: true, message: "จะส่งเมื่อเชื่อมต่ออินเทอร์เน็ต" };
    }
  }
  
  async #sendToServer(formData) {
    const response = await fetch("/api/submit", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(formData)
    });
    
    if (!response.ok) throw new Error("ส่งข้อมูลไม่สำเร็จ");
    return response.json();
  }
}
```

---

## Step 1679: Push Notifications - Permission และ Setup

```javascript
// Push Notifications

// ขอ permission
async function requestNotificationPermission() {
  const permission = await Notification.requestPermission();
  
  switch (permission) {
    case "granted":
      console.log("ได้รับสิทธิ์ notifications แล้ว");
      return true;
    case "denied":
      console.log("ผู้ใช้ปฏิเสธ notifications");
      return false;
    case "default":
      console.log("ผู้ใช้ยังไม่ได้ตัดสินใจ");
      return false;
  }
}

// แสดง local notification (ไม่ต้องการ server)
function showLocalNotification(title, options = {}) {
  if (Notification.permission !== "granted") {
    console.error("ไม่มีสิทธิ์แสดง notifications");
    return;
  }
  
  const notification = new Notification(title, {
    body: "เนื้อหาของ notification",
    icon: "/icons/icon-192.png",
    badge: "/icons/badge-72.png",
    image: "/images/notification-banner.jpg",
    tag: "my-notification",     // ID ที่ unique (ป้องกัน duplicate)
    renotify: false,             // ไม่แจ้งซ้ำถ้า tag เดิมอยู่แล้ว
    requireInteraction: false,  // ปิดอัตโนมัติ
    silent: false,
    timestamp: Date.now(),
    data: { url: "/some-page" }, // ข้อมูลเพิ่มเติม
    actions: [
      { action: "open", title: "เปิด" },
      { action: "dismiss", title: "ปิด" }
    ],
    ...options
  });
  
  notification.onclick = () => {
    window.focus();
    notification.close();
  };
  
  return notification;
}

// ผ่าน Service Worker (รองรับ push notifications)
async function showServiceWorkerNotification(title, options) {
  const registration = await navigator.serviceWorker.ready;
  
  await registration.showNotification(title, {
    body: options.body,
    icon: "/icons/icon-192.png",
    badge: "/icons/badge-72.png",
    vibrate: [100, 50, 100],    // Vibration pattern
    tag: options.tag || "default",
    data: options.data,
    actions: options.actions
  });
}
```

---

## Step 1680: Push Subscription

```javascript
// Web Push subscription

// VAPID keys - สร้างด้วย web-push library
// npx web-push generate-vapid-keys
const VAPID_PUBLIC_KEY = "BEl62iUYgUivxIkv69yViEuiBIa-Ib9-SkvMeAtA3LFgDzkrxZJjSgSnfckjBJuBkr3qBUYIHBQFLXYp5Nksh8U";

// Subscribe to push
async function subscribeToPush() {
  const permission = await requestNotificationPermission();
  if (!permission) return null;
  
  const registration = await navigator.serviceWorker.ready;
  
  try {
    // ตรวจสอบว่า subscribe แล้วหรือยัง
    const existingSubscription = await registration.pushManager.getSubscription();
    if (existingSubscription) {
      return existingSubscription;
    }
    
    // Subscribe ใหม่
    const subscription = await registration.pushManager.subscribe({
      userVisibleOnly: true, // ต้องเป็น true เสมอ
      applicationServerKey: urlBase64ToUint8Array(VAPID_PUBLIC_KEY)
    });
    
    console.log("Push subscription:", subscription.endpoint);
    
    // ส่ง subscription ไปยัง server
    await sendSubscriptionToServer(subscription);
    
    return subscription;
  } catch (error) {
    console.error("Push subscription ล้มเหลว:", error);
    return null;
  }
}

// Helper: แปลง VAPID key
function urlBase64ToUint8Array(base64String) {
  const padding = "=".repeat((4 - (base64String.length % 4)) % 4);
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

// ส่ง subscription ไปยัง server
async function sendSubscriptionToServer(subscription) {
  const response = await fetch("/api/push/subscribe", {
    method: "POST",
    headers: { "Content-Type": "application/json" },
    body: JSON.stringify(subscription)
  });
  
  if (!response.ok) throw new Error("บันทึก subscription ไม่สำเร็จ");
}

// Unsubscribe
async function unsubscribeFromPush() {
  const registration = await navigator.serviceWorker.ready;
  const subscription = await registration.pushManager.getSubscription();
  
  if (subscription) {
    await subscription.unsubscribe();
    
    // แจ้ง server
    await fetch("/api/push/unsubscribe", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ endpoint: subscription.endpoint })
    });
  }
}
```

---

## Step 1681: Web Push Server

```javascript
// server/push.js - Web Push server ด้วย Node.js

const webpush = require("web-push");
const express = require("express");
const router = express.Router();

// ตั้งค่า VAPID keys
webpush.setVapidDetails(
  "mailto:admin@example.com",
  process.env.VAPID_PUBLIC_KEY,
  process.env.VAPID_PRIVATE_KEY
);

// Database ของ subscriptions (ใช้ real DB ใน production)
const subscriptions = new Map();

// บันทึก subscription
router.post("/subscribe", async (req, res) => {
  const subscription = req.body;
  
  subscriptions.set(subscription.endpoint, subscription);
  
  res.json({ success: true });
});

// ลบ subscription
router.post("/unsubscribe", async (req, res) => {
  const { endpoint } = req.body;
  subscriptions.delete(endpoint);
  res.json({ success: true });
});

// ส่ง notification ไปยัง user คนเดียว
router.post("/send/:userId", async (req, res) => {
  const { title, body, data } = req.body;
  const subscription = subscriptions.get(req.params.userId);
  
  if (!subscription) {
    return res.status(404).json({ error: "ไม่พบ subscription" });
  }
  
  const payload = JSON.stringify({
    title,
    body,
    icon: "/icons/icon-192.png",
    badge: "/icons/badge-72.png",
    data: {
      url: data?.url || "/",
      ...data
    }
  });
  
  try {
    await webpush.sendNotification(subscription, payload);
    res.json({ success: true });
  } catch (error) {
    if (error.statusCode === 410) {
      // Subscription หมดอายุแล้ว
      subscriptions.delete(subscription.endpoint);
      res.status(410).json({ error: "Subscription หมดอายุ" });
    } else {
      res.status(500).json({ error: error.message });
    }
  }
});

// Broadcast ไปยัง users ทั้งหมด
router.post("/broadcast", async (req, res) => {
  const { title, body, data } = req.body;
  
  const payload = JSON.stringify({ title, body, data });
  
  const results = await Promise.allSettled(
    Array.from(subscriptions.values()).map(subscription =>
      webpush.sendNotification(subscription, payload)
    )
  );
  
  const failed = results.filter(r => r.status === "rejected").length;
  const succeeded = results.length - failed;
  
  res.json({ 
    success: true, 
    sent: succeeded, 
    failed 
  });
});

module.exports = router;
```

---

## Step 1682: Service Worker Push Handler

```javascript
// ใน service-worker.js: จัดการ push events

self.addEventListener("push", (event) => {
  console.log("Push event received");
  
  let notificationData;
  
  try {
    notificationData = event.data?.json() || {};
  } catch {
    notificationData = { title: "การแจ้งเตือนใหม่" };
  }
  
  const {
    title = "การแจ้งเตือน",
    body = "",
    icon = "/icons/icon-192.png",
    badge = "/icons/badge-72.png",
    image,
    tag = "default",
    data = {}
  } = notificationData;
  
  const options = {
    body,
    icon,
    badge,
    image,
    tag,
    data,
    vibrate: [100, 50, 100, 50, 100],
    actions: [
      { action: "open", title: "เปิด", icon: "/icons/open.png" },
      { action: "dismiss", title: "ปิด", icon: "/icons/close.png" }
    ],
    requireInteraction: notificationData.requireInteraction || false,
    timestamp: Date.now()
  };
  
  event.waitUntil(
    self.registration.showNotification(title, options)
  );
});

// จัดการ click บน notification
self.addEventListener("notificationclick", (event) => {
  event.notification.close();
  
  const { action } = event;
  const { url } = event.notification.data;
  
  if (action === "dismiss") {
    return; // ปิดเฉยๆ
  }
  
  // เปิด URL เมื่อกด notification
  event.waitUntil(
    clients.matchAll({ type: "window" }).then(windowClients => {
      // ตรวจว่ามี tab เปิดอยู่แล้วไหม
      for (const client of windowClients) {
        if (client.url === url && "focus" in client) {
          return client.focus();
        }
      }
      
      // เปิด tab ใหม่
      if (clients.openWindow) {
        return clients.openWindow(url || "/");
      }
    })
  );
});

// Notification close event
self.addEventListener("notificationclose", (event) => {
  console.log("Notification ถูกปิด:", event.notification.tag);
  
  // Track ว่า notification ถูกปิดโดยไม่ได้กด
  // (analytics)
});
```

---

## Step 1683: App Shell Architecture

```javascript
// App Shell Architecture
// โหลด shell ก่อน แล้ว populate ด้วยข้อมูล

// service-worker.js

// App Shell files - โหลดทันที, ไม่เคยเปลี่ยน
const APP_SHELL = [
  "/",
  "/index.html",
  "/app.js",
  "/app.css",
  "/fonts/thai-font.woff2",
  "/icons/icon-192.png"
];

// Cache App Shell ตอน install
self.addEventListener("install", event => {
  event.waitUntil(
    caches.open("app-shell-v1").then(cache => cache.addAll(APP_SHELL))
  );
});

// Serve App Shell จาก cache เสมอ (cache-first สำหรับ shell)
self.addEventListener("fetch", event => {
  const url = new URL(event.request.url);
  
  if (APP_SHELL.includes(url.pathname)) {
    event.respondWith(
      caches.match(event.request).then(cached => cached || fetch(event.request))
    );
    return;
  }
  
  // Content: network first
  event.respondWith(networkFirst(event.request));
});

// index.html - App Shell
// <html>
// <head>
//   <link rel="stylesheet" href="/app.css">
//   <script defer src="/app.js"></script>
// </head>
// <body>
//   <!-- App Shell: navigation, header, layout -->
//   <header>...</header>
//   <nav>...</nav>
//   
//   <!-- Content placeholder -->
//   <main id="content">
//     <!-- Loading skeleton -->
//     <div class="skeleton">Loading...</div>
//   </main>
//   
//   <footer>...</footer>
// </body>
// </html>
```

---

## Step 1684: Cache Strategies

```javascript
// Cache strategies สำหรับ PWA

// 1. Precaching - cache ล่วงหน้าตอน install
const PRECACHE_URLS = [
  "/",
  "/about",
  "/contact",
  "/css/main.css",
  "/js/app.js",
  "/js/vendor.js"
];

// 2. Runtime caching strategies
const strategies = {
  // Static assets (CSS, JS, fonts)
  staticAssets: {
    strategy: "cacheFirst",
    cacheName: "static-v1",
    expiration: {
      maxEntries: 50,
      maxAgeSeconds: 30 * 24 * 60 * 60 // 30 วัน
    }
  },
  
  // API responses
  apiData: {
    strategy: "networkFirst",
    cacheName: "api-v1",
    networkTimeoutSeconds: 3,
    expiration: {
      maxEntries: 200,
      maxAgeSeconds: 24 * 60 * 60 // 1 วัน
    }
  },
  
  // Images
  images: {
    strategy: "cacheFirst",
    cacheName: "images-v1",
    expiration: {
      maxEntries: 100,
      maxAgeSeconds: 7 * 24 * 60 * 60 // 7 วัน
    }
  },
  
  // HTML pages
  pages: {
    strategy: "networkFirst",
    cacheName: "pages-v1",
    expiration: {
      maxEntries: 50,
      maxAgeSeconds: 24 * 60 * 60 // 1 วัน
    }
  }
};

// Cache with expiration
async function cacheWithExpiration(request, response, cacheName, maxAgeSeconds) {
  const cache = await caches.open(cacheName);
  
  // เพิ่ม timestamp header
  const headers = new Headers(response.headers);
  headers.append("sw-fetched-on", new Date().toISOString());
  
  const cachedResponse = new Response(response.body, {
    status: response.status,
    statusText: response.statusText,
    headers
  });
  
  cache.put(request, cachedResponse);
}

// ตรวจสอบว่า cache หมดอายุหรือยัง
async function getCachedWithExpiry(request, maxAgeSeconds) {
  const cached = await caches.match(request);
  if (!cached) return null;
  
  const fetchedOn = cached.headers.get("sw-fetched-on");
  if (!fetchedOn) return cached;
  
  const age = (Date.now() - new Date(fetchedOn).getTime()) / 1000;
  
  if (age > maxAgeSeconds) {
    // Cache หมดอายุ
    const cache = await caches.open("dynamic-v1");
    await cache.delete(request);
    return null;
  }
  
  return cached;
}

// Workbox-like cache size management
async function limitCacheSize(cacheName, maxEntries) {
  const cache = await caches.open(cacheName);
  const keys = await cache.keys();
  
  if (keys.length > maxEntries) {
    // ลบ entries เก่าที่สุด
    const toDelete = keys.slice(0, keys.length - maxEntries);
    await Promise.all(toDelete.map(key => cache.delete(key)));
  }
}
```

---

## Step 1685: PWA Auditing with Lighthouse

```javascript
// การตรวจสอบ PWA ด้วย Lighthouse

// Lighthouse scores ที่ควรทำให้ได้:
// Performance: 90+
// Accessibility: 90+
// Best Practices: 90+
// SEO: 90+
// PWA: ✓ (ผ่านทุก criteria)

// Common PWA issues และวิธีแก้:

// 1. manifest.json ไม่ครบ
// แก้: ตรวจสอบ required fields

// 2. Service Worker ไม่ register
// แก้: ลงทะเบียนใน main.js

// 3. ไม่ทำงาน offline
// แก้: cache pages ที่สำคัญ

// 4. icons ขนาดไม่ครบ
// แก้: สร้างทุกขนาดที่ระบุใน manifest

// 5. HTTPS ไม่ใช้
// แก้: configure server ให้ใช้ HTTPS

// script สำหรับ generate icons ทุกขนาด
// scripts/generate-icons.js

const sharp = require("sharp");
const path = require("path");
const fs = require("fs");

const sizes = [72, 96, 128, 144, 152, 192, 384, 512];
const inputIcon = "src/icon-source.png";
const outputDir = "public/icons";

fs.mkdirSync(outputDir, { recursive: true });

Promise.all(
  sizes.map(size =>
    sharp(inputIcon)
      .resize(size, size)
      .png()
      .toFile(path.join(outputDir, `icon-${size}.png`))
      .then(() => console.log(`สร้าง icon-${size}.png สำเร็จ`))
  )
).then(() => {
  console.log("สร้าง icons ทุกขนาดสำเร็จ!");
});

// Lighthouse CI configuration
// lighthouserc.json
const lighthouseConfig = {
  ci: {
    collect: {
      url: ["http://localhost:3000"],
      numberOfRuns: 3
    },
    assert: {
      assertions: {
        "categories:performance": ["warn", { minScore: 0.9 }],
        "categories:accessibility": ["error", { minScore: 0.9 }],
        "categories:best-practices": ["warn", { minScore: 0.9 }],
        "categories:seo": ["warn", { minScore: 0.9 }],
        "categories:pwa": ["warn", { minScore: 0.9 }]
      }
    },
    upload: {
      target: "temporary-public-storage"
    }
  }
};
```

---

## Step 1686: Update Flow สำหรับ PWA

```javascript
// PWA Update Flow

// service-worker.js

const CACHE_VERSION = "v1.2.3"; // เปลี่ยนเมื่อ update
const STATIC_CACHE = `static-${CACHE_VERSION}`;
const DYNAMIC_CACHE = `dynamic-${CACHE_VERSION}`;

self.addEventListener("activate", event => {
  event.waitUntil(
    caches.keys().then(cacheNames => {
      return Promise.all(
        cacheNames
          .filter(name => 
            !name.includes(CACHE_VERSION) && 
            (name.startsWith("static-") || name.startsWith("dynamic-"))
          )
          .map(name => {
            console.log(`ลบ cache เก่า: ${name}`);
            return caches.delete(name);
          })
      );
    }).then(() => self.clients.claim())
  );
});

// รับ message จาก main thread
self.addEventListener("message", event => {
  if (event.data?.type === "GET_VERSION") {
    event.ports[0].postMessage({ version: CACHE_VERSION });
  }
  
  if (event.data?.type === "SKIP_WAITING") {
    self.skipWaiting();
  }
});

// main.js - จัดการ update

class PWAUpdater {
  #registration = null;
  
  async setup() {
    this.#registration = await navigator.serviceWorker.getRegistration();
    
    if (!this.#registration) return;
    
    // ตรวจสอบอัปเดตทุก 1 ชั่วโมง
    setInterval(() => {
      this.#registration.update();
    }, 60 * 60 * 1000);
    
    // จัดการ update found
    this.#registration.addEventListener("updatefound", () => {
      const newWorker = this.#registration.installing;
      
      newWorker.addEventListener("statechange", () => {
        if (
          newWorker.state === "installed" && 
          navigator.serviceWorker.controller
        ) {
          this.#onUpdateReady(newWorker);
        }
      });
    });
    
    // ตรวจสอบ version
    const version = await this.#getVersion();
    console.log("SW Version:", version);
  }
  
  #onUpdateReady(newWorker) {
    // แสดง toast notification
    const toast = document.createElement("div");
    toast.className = "update-toast";
    toast.innerHTML = `
      <div class="toast-content">
        <span>🎉 มีการอัปเดตใหม่!</span>
        <button class="toast-btn primary" id="reloadBtn">อัปเดตตอนนี้</button>
        <button class="toast-btn" id="laterBtn">ทีหลัง</button>
      </div>
    `;
    document.body.appendChild(toast);
    
    document.getElementById("reloadBtn").onclick = () => {
      newWorker.postMessage({ type: "SKIP_WAITING" });
      window.location.reload();
    };
    
    document.getElementById("laterBtn").onclick = () => {
      toast.remove();
    };
  }
  
  async #getVersion() {
    if (!navigator.serviceWorker.controller) return null;
    
    return new Promise(resolve => {
      const channel = new MessageChannel();
      channel.port1.onmessage = e => resolve(e.data.version);
      navigator.serviceWorker.controller.postMessage(
        { type: "GET_VERSION" },
        [channel.port2]
      );
    });
  }
}

const updater = new PWAUpdater();
updater.setup();
```

---

## Step 1687: iOS PWA Limitations

```javascript
// iOS PWA ข้อจำกัดที่ต้องรู้

/*
iOS PWA Limitations (ณ iOS 17):

1. Push Notifications:
   - รองรับตั้งแต่ iOS 16.4+ เท่านั้น
   - ต้องได้รับสิทธิ์จากผู้ใช้

2. Storage:
   - IndexedDB: จำกัด 50MB ต่อ origin
   - เคลียร์ได้เมื่อ storage pressure สูง

3. Background Processing:
   - Background Sync: ไม่รองรับ
   - Periodic Background Sync: ไม่รองรับ
   - Background Fetch: ไม่รองรับ

4. System Integration:
   - ไม่มี "Share Sheet" integration
   - Bluetooth: จำกัด
   - NFC: ไม่รองรับ

5. Web APIs:
   - Web Bluetooth: ไม่รองรับ
   - Web Serial: ไม่รองรับ
   - Contacts API: ไม่รองรับ

6. PWA Installation:
   - ต้องใช้ Safari เท่านั้น (Chrome/Firefox บน iOS ไม่รองรับ)
   - ไม่มี beforeinstallprompt event
   - ต้อง add manually จาก Share Sheet
*/

// Detect iOS และแสดง instructions
function handleIOSPWA() {
  const isIOS = /iPad|iPhone|iPod/.test(navigator.userAgent);
  const isSafari = /^((?!chrome|android).)*safari/i.test(navigator.userAgent);
  const isInStandaloneMode = window.navigator.standalone;
  
  if (isIOS && !isInStandaloneMode) {
    if (!isSafari) {
      // ใช้ Chrome/Firefox บน iOS
      showMessage("กรุณาเปิดใน Safari เพื่อติดตั้ง PWA");
    } else {
      showIOSInstallGuide();
    }
  }
}

function showIOSInstallGuide() {
  const guide = document.createElement("div");
  guide.className = "ios-install-guide";
  guide.innerHTML = `
    <div class="guide-content">
      <h3>ติดตั้งแอปบน iPhone ของคุณ</h3>
      <ol>
        <li>กดปุ่ม <strong>แชร์</strong> <img src="/icons/share.svg" alt="share"> ที่ด้านล่าง</li>
        <li>เลื่อนลงและกด <strong>"เพิ่มที่หน้าจอโฮม"</strong></li>
        <li>กด <strong>"เพิ่ม"</strong> ที่มุมบนขวา</li>
      </ol>
      <button onclick="this.parentElement.parentElement.remove()">ปิด</button>
    </div>
  `;
  document.body.appendChild(guide);
}

// iOS Safari-specific CSS
// สำหรับ notch/safe areas
const iosSafeAreaCSS = `
  /* iOS safe areas */
  body {
    padding-top: env(safe-area-inset-top);
    padding-right: env(safe-area-inset-right);
    padding-bottom: env(safe-area-inset-bottom);
    padding-left: env(safe-area-inset-left);
  }
  
  /* Standalone mode header */
  @media (display-mode: standalone) {
    .app-header {
      padding-top: calc(20px + env(safe-area-inset-top));
    }
    
    .app-footer {
      padding-bottom: calc(10px + env(safe-area-inset-bottom));
    }
  }
  
  /* iOS rubber band scrolling */
  .scroll-container {
    -webkit-overflow-scrolling: touch;
    overflow-y: scroll;
  }
`;
```

---

## Step 1688-1690: Complete PWA Example

```javascript
// Complete PWA Application

// วิธีใช้งาน - main.js
class PWAApp {
  #installManager;
  #networkStatus;
  #pushManager;
  #updater;
  #storage;
  
  async init() {
    // Initialize core modules
    this.#storage = new OfflineStorage();
    await this.#storage.init();
    
    this.#networkStatus = new NetworkStatus();
    this.#installManager = new PWAInstallManager();
    this.#updater = new PWAUpdater();
    
    // Register Service Worker
    await this.#registerSW();
    
    // Setup push notifications
    await this.#setupPush();
    
    // Load initial data
    await this.#loadData();
    
    // iOS handling
    handleIOSPWA();
    
    console.log("PWA initialized สำเร็จ");
  }
  
  async #registerSW() {
    if (!("serviceWorker" in navigator)) return;
    
    try {
      const reg = await navigator.serviceWorker.register("/sw.js");
      
      reg.addEventListener("updatefound", () => {
        this.#updater.setup();
      });
      
      console.log("Service Worker registered");
    } catch (err) {
      console.error("SW registration failed:", err);
    }
  }
  
  async #setupPush() {
    const hasPermission = Notification.permission === "granted";
    
    if (hasPermission) {
      await subscribeToPush();
    }
  }
  
  async #loadData() {
    const users = await fetchWithOfflineSupport("/api/users");
    this.#renderUsers(users);
  }
  
  #renderUsers(users) {
    const container = document.getElementById("users");
    container.innerHTML = users
      .map(u => `<div class="user-card">${u.name}</div>`)
      .join("");
  }
  
  async askForNotificationPermission() {
    const granted = await requestNotificationPermission();
    
    if (granted) {
      const subscription = await subscribeToPush();
      if (subscription) {
        showLocalNotification("เปิดใช้งานการแจ้งเตือนแล้ว!", {
          body: "คุณจะได้รับการแจ้งเตือนใหม่"
        });
      }
    }
  }
}

// เริ่มต้น app
const app = new PWAApp();
app.init().catch(console.error);

// manifest.json (ตัวอย่างสมบูรณ์ดูใน Step 1673)
// service-worker.js (ตัวอย่างสมบูรณ์ดูใน Steps 1674-1686)
```

```html
<!-- index.html - สมบูรณ์ -->
<!DOCTYPE html>
<html lang="th">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>My PWA App</title>
  
  <!-- PWA Meta Tags -->
  <link rel="manifest" href="/manifest.json">
  <meta name="theme-color" content="#3367D6">
  <meta name="apple-mobile-web-app-capable" content="yes">
  <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
  <meta name="apple-mobile-web-app-title" content="MyApp">
  <link rel="apple-touch-icon" href="/icons/icon-192.png">
  
  <!-- Preload critical assets -->
  <link rel="preload" href="/fonts/thai-font.woff2" as="font" crossorigin>
  <link rel="preload" href="/css/critical.css" as="style">
  
  <link rel="stylesheet" href="/css/app.css">
</head>
<body>
  <header class="app-header">
    <h1>My PWA App</h1>
    <button id="installPWA" style="display:none">📱 ติดตั้งแอป</button>
  </header>
  
  <!-- Offline banner -->
  <div id="offlineBanner" class="offline-banner" style="display:none">
    ⚠️ คุณกำลัง offline
  </div>
  
  <main id="content">
    <div class="skeleton-loader">กำลังโหลด...</div>
  </main>
  
  <footer class="app-footer">
    <nav>
      <a href="/">หน้าหลัก</a>
      <a href="/about">เกี่ยวกับ</a>
    </nav>
  </footer>
  
  <script type="module" src="/js/app.js"></script>
</body>
</html>
```

---

## แบบฝึกหัด (Exercises)

### แบบฝึกหัดที่ 1: Todo App แบบ Offline-First
สร้าง Todo app ที่:
- ทำงานได้แม้ offline
- Sync ไปยัง server เมื่อ online กลับมา
- แสดง sync status อย่างชัดเจน
- รองรับ conflict resolution

### แบบฝึกหัดที่ 2: News PWA
สร้าง news app ที่:
- Cache เนื้อหาสำหรับ offline
- Push notifications เมื่อมีข่าวด่วน
- Periodic background sync ทุก 6 ชั่วโมง
- Installable บน Android และ iOS

### แบบฝึกหัดที่ 3: Camera PWA
สร้าง app ถ่ายภาพที่:
- ใช้ Camera API
- บันทึกรูปใน IndexedDB
- Upload ไปยัง server เมื่อ online
- แสดง gallery ออฟไลน์

### แบบฝึกหัดที่ 4: PWA Performance Optimization
Optimize PWA ที่มีอยู่:
- ปรับปรุง Lighthouse score เป็น 90+
- ลด Time to First Byte
- ปรับ cache strategies
- เพิ่ม precaching

### แบบฝึกหัดที่ 5: Share Target
สร้าง PWA ที่:
- รับ shared content (text, URL, files)
- ประมวลผล shared files
- แสดงใน app
- Share ต่อไปยัง contacts

---

## สรุป (Summary)

ใน Part 85 เราได้เรียนรู้ Progressive Web Apps:

1. **PWA คืออะไร** - ความหมาย, ข้อดี, ข้อจำกัด
2. **PWA Criteria** - เงื่อนไขที่ต้องผ่านเพื่อเป็น PWA
3. **Web App Manifest** - การตั้งค่า icons, display, shortcuts
4. **Service Worker** - การ install, activate, fetch events
5. **Cache Strategies** - Cache First, Network First, SWR
6. **Install Prompt** - beforeinstallprompt, iOS instructions
7. **Offline Functionality** - IndexedDB, offline detection
8. **Background Sync** - Sync เมื่อ online กลับมา
9. **Push Notifications** - Permission, subscription, handling
10. **Web Push Server** - Node.js + web-push library
11. **App Shell Architecture** - โหลด shell ก่อน content
12. **Update Flow** - จัดการ SW updates อย่าง smooth
13. **iOS Limitations** - ข้อจำกัดบน iOS Safari
14. **Complete Example** - PWA app สมบูรณ์
15. **Lighthouse Auditing** - ตรวจสอบ PWA quality

PWA เป็นอนาคตของ web development - มอบประสบการณ์ native-like
พร้อม web distribution model ที่ง่ายและประหยัดกว่า native apps
