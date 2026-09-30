# Part 42: Browser APIs (Steps 811-830)

## บทนำ

Browser APIs คือชุดของ interfaces ที่ browser ให้มาเพื่อให้ JavaScript สามารถโต้ตอบกับ browser และอุปกรณ์ได้ เช่น ตำแหน่งที่ตั้ง, clipboard, notifications, speech, และอื่นๆ อีกมากมาย

ในบทนี้เราจะเรียนรู้:
- Navigator API และ feature detection
- Geolocation, Clipboard, Notifications
- Speech, Storage APIs
- History และ URL APIs
- Broadcast Channel, Page Visibility
- Fullscreen, Pointer Lock

---

## Step 811: Navigator API

Navigator object มีข้อมูลเกี่ยวกับ browser และอุปกรณ์

```javascript
// ข้อมูลพื้นฐานจาก Navigator
console.log(navigator.userAgent);
// "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36..."

console.log(navigator.language);    // "th" หรือ "en-US"
console.log(navigator.languages);   // ["th", "en-US", "en"]
console.log(navigator.platform);    // "Win32", "MacIntel", "Linux x86_64"
console.log(navigator.cookieEnabled); // true หรือ false
console.log(navigator.onLine);      // true ถ้าออนไลน์

// Device memory (GB)
console.log(navigator.deviceMemory); // 2, 4, 8 เป็นต้น

// Hardware concurrency (จำนวน CPU cores)
console.log(navigator.hardwareConcurrency); // 4, 8, เป็นต้น

// Vendor
console.log(navigator.vendor); // "Google Inc."

// Feature Detection
function detectFeatures() {
  const features = {
    geolocation: 'geolocation' in navigator,
    notifications: 'Notification' in window,
    serviceWorker: 'serviceWorker' in navigator,
    bluetooth: 'bluetooth' in navigator,
    usb: 'usb' in navigator,
    credentials: 'credentials' in navigator,
    storage: 'storage' in navigator,
    vibrate: 'vibrate' in navigator,
    share: 'share' in navigator,
    clipboard: 'clipboard' in navigator,
    mediaDevices: 'mediaDevices' in navigator,
    permissions: 'permissions' in navigator
  };
  
  return features;
}

console.log('Browser Features:', detectFeatures());

// Online/Offline events
window.addEventListener('online', () => {
  console.log('กลับมาออนไลน์แล้ว');
  // แจ้งผู้ใช้
  showNotification('เชื่อมต่ออินเทอร์เน็ตแล้ว');
});

window.addEventListener('offline', () => {
  console.log('ออฟไลน์แล้ว');
  // แจ้งผู้ใช้
  showNotification('ไม่มีการเชื่อมต่ออินเทอร์เน็ต', 'warning');
});

function showNotification(message, type = 'info') {
  console.log(`[${type.toUpperCase()}] ${message}`);
}

// User Agent Parser (basic)
function parseUserAgent() {
  const ua = navigator.userAgent;
  
  return {
    isMobile: /Mobile|Android|iPhone|iPad/i.test(ua),
    isTablet: /iPad|Tablet/i.test(ua),
    isDesktop: !/Mobile|Android|iPhone|iPad|Tablet/i.test(ua),
    
    browser: {
      isChrome: /Chrome/.test(ua) && !/Edge/.test(ua),
      isFirefox: /Firefox/.test(ua),
      isSafari: /Safari/.test(ua) && !/Chrome/.test(ua),
      isEdge: /Edg/.test(ua),
      isOpera: /OPR|Opera/.test(ua)
    },
    
    os: {
      isWindows: /Windows/.test(ua),
      isMac: /Macintosh|Mac OS X/.test(ua),
      isLinux: /Linux/.test(ua) && !/Android/.test(ua),
      isAndroid: /Android/.test(ua),
      isIOS: /iPhone|iPad|iPod/.test(ua)
    }
  };
}

const deviceInfo = parseUserAgent();
console.log('Device Info:', deviceInfo);
```

---

## Step 812: Geolocation API

```javascript
// ขอตำแหน่งปัจจุบัน
function getCurrentLocation() {
  return new Promise((resolve, reject) => {
    if (!navigator.geolocation) {
      reject(new Error('Geolocation ไม่รองรับบน browser นี้'));
      return;
    }
    
    const options = {
      enableHighAccuracy: true,  // ใช้ GPS ถ้าเป็นไปได้
      timeout: 10000,            // timeout 10 วินาที
      maximumAge: 0              // ไม่ใช้ cached position
    };
    
    navigator.geolocation.getCurrentPosition(
      (position) => {
        resolve({
          latitude: position.coords.latitude,
          longitude: position.coords.longitude,
          accuracy: position.coords.accuracy,      // ความแม่นยำ (เมตร)
          altitude: position.coords.altitude,      // ความสูง (เมตร)
          altitudeAccuracy: position.coords.altitudeAccuracy,
          heading: position.coords.heading,        // ทิศทาง (องศา)
          speed: position.coords.speed,            // ความเร็ว (m/s)
          timestamp: position.timestamp
        });
      },
      (error) => {
        switch (error.code) {
          case error.PERMISSION_DENIED:
            reject(new Error('ผู้ใช้ปฏิเสธการขออนุญาต'));
            break;
          case error.POSITION_UNAVAILABLE:
            reject(new Error('ไม่สามารถหาตำแหน่งได้'));
            break;
          case error.TIMEOUT:
            reject(new Error('หมดเวลาในการค้นหาตำแหน่ง'));
            break;
          default:
            reject(new Error(`เกิดข้อผิดพลาด: ${error.message}`));
        }
      },
      options
    );
  });
}

// ใช้งาน
async function showUserLocation() {
  try {
    const location = await getCurrentLocation();
    console.log('ตำแหน่งของคุณ:');
    console.log(`  Latitude: ${location.latitude}`);
    console.log(`  Longitude: ${location.longitude}`);
    console.log(`  ความแม่นยำ: ${location.accuracy} เมตร`);
    
    // สร้าง Google Maps link
    const mapsUrl = `https://maps.google.com/?q=${location.latitude},${location.longitude}`;
    console.log(`  Google Maps: ${mapsUrl}`);
    
    return location;
  } catch (error) {
    console.error('ไม่สามารถรับตำแหน่งได้:', error.message);
  }
}

// watchPosition - ติดตามตำแหน่งแบบ real-time
class LocationTracker {
  constructor(options = {}) {
    this.watchId = null;
    this.positions = [];
    this.onUpdate = options.onUpdate || (() => {});
    this.onError = options.onError || console.error;
    this.options = {
      enableHighAccuracy: options.highAccuracy || false,
      timeout: options.timeout || 10000,
      maximumAge: options.maxAge || 5000
    };
  }
  
  start() {
    if (!navigator.geolocation) {
      throw new Error('Geolocation ไม่รองรับ');
    }
    
    if (this.watchId !== null) {
      console.warn('LocationTracker กำลังทำงานอยู่แล้ว');
      return;
    }
    
    this.watchId = navigator.geolocation.watchPosition(
      (position) => {
        const loc = {
          lat: position.coords.latitude,
          lng: position.coords.longitude,
          accuracy: position.coords.accuracy,
          timestamp: new Date(position.timestamp)
        };
        
        this.positions.push(loc);
        this.onUpdate(loc);
      },
      (error) => {
        this.onError(error);
      },
      this.options
    );
    
    console.log('เริ่มติดตามตำแหน่ง...');
    return this;
  }
  
  stop() {
    if (this.watchId !== null) {
      navigator.geolocation.clearWatch(this.watchId);
      this.watchId = null;
      console.log('หยุดติดตามตำแหน่ง');
    }
  }
  
  getLastPosition() {
    return this.positions[this.positions.length - 1] || null;
  }
  
  getHistory() {
    return [...this.positions];
  }
  
  calculateDistance() {
    if (this.positions.length < 2) return 0;
    
    let total = 0;
    for (let i = 1; i < this.positions.length; i++) {
      total += this.#haversine(this.positions[i-1], this.positions[i]);
    }
    return total;
  }
  
  // Haversine formula - คำนวณระยะทางระหว่างสองจุดบนโลก
  #haversine(pos1, pos2) {
    const R = 6371000; // รัศมีโลก (เมตร)
    const φ1 = pos1.lat * Math.PI / 180;
    const φ2 = pos2.lat * Math.PI / 180;
    const Δφ = (pos2.lat - pos1.lat) * Math.PI / 180;
    const Δλ = (pos2.lng - pos1.lng) * Math.PI / 180;
    
    const a = Math.sin(Δφ/2) * Math.sin(Δφ/2) +
              Math.cos(φ1) * Math.cos(φ2) *
              Math.sin(Δλ/2) * Math.sin(Δλ/2);
    const c = 2 * Math.atan2(Math.sqrt(a), Math.sqrt(1-a));
    
    return R * c; // ระยะทางเป็นเมตร
  }
}

// ใช้งาน LocationTracker
const tracker = new LocationTracker({
  highAccuracy: true,
  onUpdate: (position) => {
    console.log(`ตำแหน่งใหม่: ${position.lat}, ${position.lng}`);
    console.log(`ระยะทางรวม: ${tracker.calculateDistance().toFixed(2)} เมตร`);
  }
});
```

---

## Step 813: Clipboard API

```javascript
// Clipboard API (Modern)

// เขียนข้อความไปยัง clipboard
async function copyToClipboard(text) {
  try {
    await navigator.clipboard.writeText(text);
    console.log('คัดลอกสำเร็จ:', text);
    return true;
  } catch (error) {
    console.error('ไม่สามารถคัดลอกได้:', error.message);
    
    // Fallback สำหรับ browser เก่า
    return fallbackCopy(text);
  }
}

// อ่านข้อความจาก clipboard
async function readFromClipboard() {
  try {
    const text = await navigator.clipboard.readText();
    console.log('ข้อความใน clipboard:', text);
    return text;
  } catch (error) {
    console.error('ไม่สามารถอ่าน clipboard ได้:', error.message);
    return null;
  }
}

// Fallback สำหรับ browser เก่า
function fallbackCopy(text) {
  const textArea = document.createElement('textarea');
  textArea.value = text;
  textArea.style.position = 'fixed';
  textArea.style.opacity = '0';
  document.body.appendChild(textArea);
  textArea.focus();
  textArea.select();
  
  try {
    const success = document.execCommand('copy');
    console.log(success ? 'Fallback copy สำเร็จ' : 'Fallback copy ล้มเหลว');
    return success;
  } catch (error) {
    console.error('Fallback copy error:', error);
    return false;
  } finally {
    document.body.removeChild(textArea);
  }
}

// อ่าน/เขียน รูปภาพและข้อมูลอื่น (Clipboard Items API)
async function copyImageToClipboard(imageBlob) {
  try {
    const item = new ClipboardItem({ 'image/png': imageBlob });
    await navigator.clipboard.write([item]);
    console.log('คัดลอกรูปภาพสำเร็จ');
    return true;
  } catch (error) {
    console.error('ไม่สามารถคัดลอกรูปภาพ:', error.message);
    return false;
  }
}

async function readImageFromClipboard() {
  try {
    const items = await navigator.clipboard.read();
    
    for (const item of items) {
      for (const type of item.types) {
        if (type.startsWith('image/')) {
          const blob = await item.getType(type);
          const url = URL.createObjectURL(blob);
          console.log('พบรูปภาพใน clipboard:', url);
          return { blob, url, type };
        }
      }
    }
    
    console.log('ไม่พบรูปภาพใน clipboard');
    return null;
  } catch (error) {
    console.error('ไม่สามารถอ่าน clipboard:', error.message);
    return null;
  }
}

// Copy Button Component
function createCopyButton(element, textToopy) {
  const button = document.createElement('button');
  button.textContent = '📋 คัดลอก';
  button.className = 'copy-button';
  
  let timeout;
  
  button.addEventListener('click', async () => {
    const text = textToopy || element.textContent;
    const success = await copyToClipboard(text);
    
    if (success) {
      button.textContent = '✅ คัดลอกแล้ว';
      clearTimeout(timeout);
      timeout = setTimeout(() => {
        button.textContent = '📋 คัดลอก';
      }, 2000);
    } else {
      button.textContent = '❌ ไม่สำเร็จ';
      setTimeout(() => {
        button.textContent = '📋 คัดลอก';
      }, 2000);
    }
  });
  
  return button;
}

// ตัวอย่างการใช้งาน clipboard
async function demoClipboard() {
  // คัดลอกข้อความ
  await copyToClipboard('สวัสดีโลก!');
  
  // อ่านกลับ
  const text = await readFromClipboard();
  console.log('อ่านได้:', text); // 'สวัสดีโลก!'
  
  // คัดลอก URL ปัจจุบัน
  await copyToClipboard(window.location.href);
}
```

---

## Step 814: Notification API

```javascript
// Notification API

class NotificationManager {
  constructor() {
    this.permission = Notification.permission;
    this.defaultOptions = {
      icon: '/favicon.ico',
      badge: '/badge.png',
      requireInteraction: false,
      silent: false
    };
  }
  
  // ขอสิทธิ์
  async requestPermission() {
    if (!('Notification' in window)) {
      console.error('Browser ไม่รองรับ Notifications');
      return false;
    }
    
    if (Notification.permission === 'granted') {
      this.permission = 'granted';
      return true;
    }
    
    if (Notification.permission !== 'denied') {
      const permission = await Notification.requestPermission();
      this.permission = permission;
      return permission === 'granted';
    }
    
    console.warn('สิทธิ์ Notification ถูกปฏิเสธ');
    return false;
  }
  
  // แสดง notification
  async show(title, options = {}) {
    // ขอสิทธิ์ถ้ายังไม่ได้
    if (this.permission !== 'granted') {
      const granted = await this.requestPermission();
      if (!granted) return null;
    }
    
    const notificationOptions = {
      ...this.defaultOptions,
      ...options,
      // timestamp ปัจจุบัน
      timestamp: options.timestamp || Date.now()
    };
    
    const notification = new Notification(title, notificationOptions);
    
    // Event handlers
    notification.addEventListener('click', () => {
      console.log('Notification clicked');
      window.focus();
      notification.close();
      options.onClick?.();
    });
    
    notification.addEventListener('close', () => {
      console.log('Notification closed');
      options.onClose?.();
    });
    
    notification.addEventListener('error', (event) => {
      console.error('Notification error:', event);
      options.onError?.(event);
    });
    
    // Auto-close ถ้ากำหนด duration
    if (options.duration) {
      setTimeout(() => notification.close(), options.duration);
    }
    
    return notification;
  }
  
  // แสดง notification แบบ simple
  async notify(title, body, options = {}) {
    return this.show(title, { body, ...options });
  }
  
  // แสดง success notification
  async success(title, body) {
    return this.notify(title, body, {
      icon: '/icons/success.png',
      tag: 'success'
    });
  }
  
  // แสดง error notification  
  async error(title, body) {
    return this.notify(title, body, {
      icon: '/icons/error.png',
      tag: 'error',
      requireInteraction: true
    });
  }
  
  // แสดง notification พร้อม actions
  async showWithActions(title, body, actions = []) {
    // Actions ใช้งานได้เฉพาะผ่าน Service Worker
    return this.show(title, {
      body,
      actions: actions.map(a => ({
        action: a.id,
        title: a.label,
        icon: a.icon
      }))
    });
  }
}

// ใช้งาน
const notificationManager = new NotificationManager();

// ขอสิทธิ์เมื่อผู้ใช้คลิก
document.getElementById('enable-notifications')?.addEventListener('click', async () => {
  const granted = await notificationManager.requestPermission();
  if (granted) {
    await notificationManager.success('สมัครรับการแจ้งเตือนสำเร็จ', 'คุณจะได้รับแจ้งเตือนจากเราแล้ว');
  }
});

// แสดง notification เมื่อมีข้อความใหม่
async function showNewMessageNotification(sender, message) {
  await notificationManager.notify(
    `ข้อความจาก ${sender}`,
    message.substring(0, 100),
    {
      tag: 'new-message',  // ใช้ tag เดิมจะ replace notification เก่า
      renotify: true,
      duration: 5000,
      onClick: () => {
        // นำทางไปยังหน้าข้อความ
        window.location.href = '/messages';
      }
    }
  );
}
```

---

## Step 815: Vibration API

```javascript
// Vibration API (ส่วนใหญ่ใช้บน mobile)

// ตรวจสอบว่ารองรับหรือไม่
function isVibrationSupported() {
  return 'vibrate' in navigator;
}

// สั่นครั้งเดียว (ms)
function vibrate(duration = 100) {
  if (!isVibrationSupported()) {
    console.log('Vibration API ไม่รองรับ');
    return false;
  }
  return navigator.vibrate(duration);
}

// pattern: [สั่น, หยุด, สั่น, ...]
function vibratePattern(pattern) {
  if (!isVibrationSupported()) return false;
  return navigator.vibrate(pattern);
}

// หยุดสั่น
function stopVibration() {
  if (!isVibrationSupported()) return false;
  return navigator.vibrate(0); // หรือ navigator.vibrate([])
}

// Vibration Patterns
const VibrationPatterns = {
  // การแจ้งเตือนทั่วไป
  notification: [100],
  
  // เสร็จสิ้น / สำเร็จ
  success: [50, 50, 150],
  
  // ข้อผิดพลาด
  error: [100, 50, 100, 50, 100],
  
  // SOS pattern
  sos: [100, 30, 100, 30, 100, 200, 200, 30, 200, 30, 200, 200, 100, 30, 100, 30, 100],
  
  // Double tap
  doubleTap: [50, 100, 50],
  
  // Long press feedback
  longPress: [20, 10, 20, 10, 60]
};

// Haptic feedback manager
const haptic = {
  light: () => vibrate(10),
  medium: () => vibrate(20),
  heavy: () => vibrate(50),
  success: () => vibratePattern(VibrationPatterns.success),
  error: () => vibratePattern(VibrationPatterns.error),
  notification: () => vibratePattern(VibrationPatterns.notification)
};

// ตัวอย่างการใช้งาน
document.querySelectorAll('button').forEach(button => {
  button.addEventListener('click', () => haptic.light());
});

// Form submission haptic feedback
document.getElementById('submit-form')?.addEventListener('click', async () => {
  try {
    haptic.medium();
    // await submitForm();
    haptic.success(); // แจ้งว่าสำเร็จ
  } catch (error) {
    haptic.error(); // แจ้งว่าผิดพลาด
  }
});
```

---

## Step 816: Battery API

```javascript
// Battery Status API

class BatteryMonitor {
  constructor() {
    this.battery = null;
    this.listeners = {};
  }
  
  async init() {
    if (!('getBattery' in navigator)) {
      console.log('Battery API ไม่รองรับบน browser นี้');
      return false;
    }
    
    try {
      this.battery = await navigator.getBattery();
      this.#setupListeners();
      return true;
    } catch (error) {
      console.error('ไม่สามารถเข้าถึง Battery API:', error);
      return false;
    }
  }
  
  #setupListeners() {
    if (!this.battery) return;
    
    this.battery.addEventListener('levelchange', () => {
      console.log(`Battery level: ${this.getLevel()}%`);
      this.#notify('levelchange', this.getInfo());
      
      // แจ้งเตือนเมื่อแบตน้อย
      if (this.battery.level <= 0.2 && !this.battery.charging) {
        this.#notify('lowbattery', this.getInfo());
      }
    });
    
    this.battery.addEventListener('chargingchange', () => {
      const status = this.battery.charging ? 'กำลังชาร์จ' : 'ไม่ได้ชาร์จ';
      console.log(`Battery charging: ${status}`);
      this.#notify('chargingchange', this.getInfo());
    });
    
    this.battery.addEventListener('chargingtimechange', () => {
      const time = this.getChargingTime();
      console.log(`เวลาชาร์จเต็ม: ${time}`);
      this.#notify('chargingtimechange', this.getInfo());
    });
    
    this.battery.addEventListener('dischargingtimechange', () => {
      const time = this.getDischargingTime();
      console.log(`เวลาแบตหมด: ${time}`);
      this.#notify('dischargingtimechange', this.getInfo());
    });
  }
  
  #notify(event, data) {
    (this.listeners[event] || []).forEach(fn => fn(data));
  }
  
  on(event, fn) {
    if (!this.listeners[event]) {
      this.listeners[event] = [];
    }
    this.listeners[event].push(fn);
    return () => this.off(event, fn); // return unsubscribe function
  }
  
  off(event, fn) {
    if (this.listeners[event]) {
      this.listeners[event] = this.listeners[event].filter(f => f !== fn);
    }
  }
  
  getLevel() {
    return this.battery ? Math.floor(this.battery.level * 100) : null;
  }
  
  isCharging() {
    return this.battery?.charging ?? null;
  }
  
  getChargingTime() {
    if (!this.battery) return null;
    const seconds = this.battery.chargingTime;
    if (seconds === Infinity) return 'ไม่ทราบ';
    return this.#formatTime(seconds);
  }
  
  getDischargingTime() {
    if (!this.battery) return null;
    const seconds = this.battery.dischargingTime;
    if (seconds === Infinity) return 'ไม่ทราบ';
    return this.#formatTime(seconds);
  }
  
  #formatTime(seconds) {
    const hours = Math.floor(seconds / 3600);
    const minutes = Math.floor((seconds % 3600) / 60);
    return `${hours} ชั่วโมง ${minutes} นาที`;
  }
  
  getInfo() {
    return {
      level: this.getLevel(),
      charging: this.isCharging(),
      chargingTime: this.getChargingTime(),
      dischargingTime: this.getDischargingTime()
    };
  }
}

// ใช้งาน
const batteryMonitor = new BatteryMonitor();

async function initBatteryMonitor() {
  const supported = await batteryMonitor.init();
  
  if (supported) {
    console.log('Battery Info:', batteryMonitor.getInfo());
    
    // แจ้งเตือนเมื่อแบตน้อย
    batteryMonitor.on('lowbattery', (info) => {
      console.warn(`⚠️ แบตเตอรีเหลือน้อย: ${info.level}%`);
      // showLowBatteryWarning(info.level);
    });
    
    // เมื่อชาร์จ - อาจเพิ่ม sync เพราะมีไฟ
    batteryMonitor.on('chargingchange', (info) => {
      if (info.charging) {
        console.log('กำลังชาร์จ - เริ่ม background sync');
        // startBackgroundSync();
      } else {
        console.log('ถอดชาร์จ - หยุด heavy processes');
        // stopBackgroundSync();
      }
    });
  }
}
```

---

## Step 817: Network Information API

```javascript
// Network Information API

class NetworkMonitor {
  constructor() {
    this.connection = navigator.connection || 
                     navigator.mozConnection || 
                     navigator.webkitConnection;
  }
  
  isSupported() {
    return !!this.connection;
  }
  
  getInfo() {
    if (!this.connection) {
      return {
        supported: false,
        online: navigator.onLine
      };
    }
    
    return {
      supported: true,
      online: navigator.onLine,
      effectiveType: this.connection.effectiveType, // 'slow-2g', '2g', '3g', '4g'
      type: this.connection.type,                   // 'wifi', 'cellular', 'ethernet', 'none'
      downlink: this.connection.downlink,           // Mbps (estimate)
      downlinkMax: this.connection.downlinkMax,     // max Mbps
      rtt: this.connection.rtt,                     // Round Trip Time (ms)
      saveData: this.connection.saveData             // Data saver enabled
    };
  }
  
  isSlowConnection() {
    const info = this.getInfo();
    if (!info.supported) return false;
    return ['slow-2g', '2g'].includes(info.effectiveType);
  }
  
  isFastConnection() {
    const info = this.getInfo();
    if (!info.supported) return navigator.onLine; // assume fast if not supported
    return info.effectiveType === '4g';
  }
  
  shouldSaveData() {
    const info = this.getInfo();
    return info.saveData || this.isSlowConnection();
  }
  
  onConnectionChange(callback) {
    if (!this.connection) {
      window.addEventListener('online', () => callback(this.getInfo()));
      window.addEventListener('offline', () => callback(this.getInfo()));
      return;
    }
    
    this.connection.addEventListener('change', () => {
      callback(this.getInfo());
    });
  }
}

// Adaptive Loading ตาม network
const networkMonitor = new NetworkMonitor();

function getImageQuality() {
  const info = networkMonitor.getInfo();
  
  if (!info.online) return null;
  
  switch (info.effectiveType) {
    case 'slow-2g':
    case '2g':
      return 'low';    // 320px, quality 40
    case '3g':
      return 'medium'; // 640px, quality 60
    case '4g':
    default:
      return 'high';   // 1280px, quality 80
  }
}

function getVideoQuality() {
  if (networkMonitor.shouldSaveData()) return '360p';
  
  const info = networkMonitor.getInfo();
  switch (info.effectiveType) {
    case 'slow-2g': return '144p';
    case '2g': return '240p';
    case '3g': return '480p';
    case '4g': return '1080p';
    default: return '720p';
  }
}

// ตัวอย่าง adaptive image loading
function loadAdaptiveImage(container, imageUrls) {
  const quality = getImageQuality();
  
  if (!quality) {
    container.innerHTML = '<p>ไม่มีการเชื่อมต่ออินเทอร์เน็ต</p>';
    return;
  }
  
  const imgUrl = imageUrls[quality] || imageUrls.medium || imageUrls.high;
  const img = document.createElement('img');
  img.src = imgUrl;
  img.alt = 'Adaptive image';
  container.appendChild(img);
  
  console.log(`Loading ${quality} quality image (network: ${networkMonitor.getInfo().effectiveType})`);
}

// Monitor network changes
networkMonitor.onConnectionChange((info) => {
  console.log('Network changed:', info);
  
  if (!info.online) {
    showOfflineBanner();
  } else if (networkMonitor.isSlowConnection()) {
    showSlowConnectionWarning();
  }
});

function showOfflineBanner() {
  console.log('📵 คุณออฟไลน์อยู่');
}

function showSlowConnectionWarning() {
  console.log('🐢 การเชื่อมต่อช้า อาจมีผลต่อประสบการณ์การใช้งาน');
}
```

---

## Step 818: History API

```javascript
// History API สำหรับ Single Page Applications

class Router {
  constructor(routes = {}) {
    this.routes = routes;
    this.currentPath = window.location.pathname;
    this.params = {};
    
    // ฟัง popstate event (เมื่อกด back/forward)
    window.addEventListener('popstate', (event) => {
      this.#handleRoute(window.location.pathname, event.state);
    });
  }
  
  // เพิ่ม route
  addRoute(path, handler) {
    this.routes[path] = handler;
    return this;
  }
  
  // navigate ไปยัง path ใหม่
  navigate(path, state = null, title = '') {
    history.pushState(state, title, path);
    this.#handleRoute(path, state);
  }
  
  // replace current history entry
  replace(path, state = null, title = '') {
    history.replaceState(state, title, path);
    this.#handleRoute(path, state);
  }
  
  // กลับหน้าก่อนหน้า
  back() { history.back(); }
  
  // ไปหน้าถัดไป
  forward() { history.forward(); }
  
  // ไปยัง index ที่กำหนด
  go(delta) { history.go(delta); }
  
  // จำนวน entries ใน history
  get length() { return history.length; }
  
  // state ปัจจุบัน
  get state() { return history.state; }
  
  #handleRoute(path, state) {
    this.currentPath = path;
    
    // ค้นหา route ที่ตรงกัน
    const matched = this.#matchRoute(path);
    
    if (matched) {
      const { handler, params } = matched;
      this.params = params;
      handler({ path, params, state });
    } else {
      // 404
      const notFoundHandler = this.routes['*'] || this.routes['/404'];
      notFoundHandler?.({ path, params: {}, state });
    }
  }
  
  #matchRoute(path) {
    for (const [routePath, handler] of Object.entries(this.routes)) {
      const result = this.#matchPath(routePath, path);
      if (result) {
        return { handler, params: result };
      }
    }
    return null;
  }
  
  #matchPath(routePath, path) {
    // แปลง route pattern เป็น regex
    const paramNames = [];
    const regexStr = routePath
      .replace(/:([^/]+)/g, (_, name) => {
        paramNames.push(name);
        return '([^/]+)';
      })
      .replace(/\*/g, '.*');
    
    const regex = new RegExp(`^${regexStr}$`);
    const match = path.match(regex);
    
    if (!match) return null;
    
    const params = {};
    paramNames.forEach((name, i) => {
      params[name] = match[i + 1];
    });
    
    return params;
  }
  
  // Start routing
  start() {
    this.#handleRoute(window.location.pathname, history.state);
  }
}

// ใช้งาน Router
const router = new Router();

router
  .addRoute('/', ({ path }) => {
    console.log('Home page');
    renderHome();
  })
  .addRoute('/users', ({ path }) => {
    console.log('Users list');
    renderUserList();
  })
  .addRoute('/users/:id', ({ path, params }) => {
    console.log('User profile:', params.id);
    renderUserProfile(params.id);
  })
  .addRoute('/users/:id/posts/:postId', ({ params }) => {
    console.log(`User ${params.id} post ${params.postId}`);
    renderPost(params.id, params.postId);
  })
  .addRoute('*', ({ path }) => {
    console.log('404 Not Found:', path);
    render404();
  });

router.start();

// Navigation links
document.querySelectorAll('[data-link]').forEach(link => {
  link.addEventListener('click', (e) => {
    e.preventDefault();
    const path = link.getAttribute('data-link');
    router.navigate(path);
  });
});

function renderHome() { console.log('Rendering home...'); }
function renderUserList() { console.log('Rendering user list...'); }
function renderUserProfile(id) { console.log(`Rendering user ${id}...`); }
function renderPost(userId, postId) { console.log(`Rendering post ${postId} by ${userId}...`); }
function render404() { console.log('Rendering 404...'); }
```

---

## Step 819: URL API

```javascript
// URL API - จัดการ URLs อย่างมีประสิทธิภาพ

// สร้าง URL object
const url = new URL('https://example.com:8080/path/to/page?key=value&foo=bar#section');

console.log(url.protocol); // 'https:'
console.log(url.hostname); // 'example.com'
console.log(url.port);     // '8080'
console.log(url.host);     // 'example.com:8080'
console.log(url.pathname); // '/path/to/page'
console.log(url.search);   // '?key=value&foo=bar'
console.log(url.hash);     // '#section'
console.log(url.origin);   // 'https://example.com:8080'
console.log(url.href);     // full URL

// แก้ไข URL
url.pathname = '/new/path';
url.searchParams.set('lang', 'th');
url.hash = 'new-section';
console.log(url.href);
// 'https://example.com:8080/new/path?key=value&foo=bar&lang=th#new-section'

// URLSearchParams - จัดการ query parameters
const params = new URLSearchParams('key=value&foo=bar&arr=1&arr=2');

console.log(params.get('key'));     // 'value'
console.log(params.get('foo'));     // 'bar'
console.log(params.getAll('arr')); // ['1', '2']
console.log(params.has('key'));    // true
console.log(params.has('xyz'));    // false

// เพิ่ม/แก้ไข/ลบ params
params.set('key', 'newvalue');     // เปลี่ยนค่า
params.append('arr', '3');         // เพิ่มค่า (ไม่แทนที่)
params.delete('foo');              // ลบ
params.sort();                     // เรียงลำดับ alphabetically

// Iterate
for (const [key, value] of params) {
  console.log(`${key}: ${value}`);
}

// แปลงเป็น object
function paramsToObject(params) {
  const obj = {};
  for (const [key, value] of params) {
    if (key in obj) {
      // Multiple values - make array
      if (!Array.isArray(obj[key])) {
        obj[key] = [obj[key]];
      }
      obj[key].push(value);
    } else {
      obj[key] = value;
    }
  }
  return obj;
}

// URL Builder utility
class UrlBuilder {
  constructor(baseUrl) {
    this.url = new URL(baseUrl);
  }
  
  path(pathname) {
    this.url.pathname = pathname;
    return this;
  }
  
  addPath(segment) {
    this.url.pathname = this.url.pathname.replace(/\/$/, '') + '/' + segment.replace(/^\//, '');
    return this;
  }
  
  param(key, value) {
    if (value !== null && value !== undefined && value !== '') {
      this.url.searchParams.set(key, value);
    }
    return this;
  }
  
  params(obj) {
    Object.entries(obj).forEach(([key, value]) => {
      this.param(key, value);
    });
    return this;
  }
  
  removeParam(key) {
    this.url.searchParams.delete(key);
    return this;
  }
  
  hash(hash) {
    this.url.hash = hash;
    return this;
  }
  
  build() {
    return this.url.href;
  }
  
  toString() {
    return this.build();
  }
}

// ตัวอย่างการใช้งาน
const apiUrl = new UrlBuilder('https://api.example.com')
  .path('/v1/users')
  .params({
    page: 1,
    limit: 20,
    sort: 'createdAt',
    order: 'desc',
    search: 'สมชาย'
  })
  .build();

console.log(apiUrl);
// 'https://api.example.com/v1/users?page=1&limit=20&sort=createdAt&order=desc&search=...'

// URL validation
function isValidUrl(string) {
  try {
    new URL(string);
    return true;
  } catch {
    return false;
  }
}

function isAbsoluteUrl(string) {
  return /^https?:\/\//i.test(string);
}

function isRelativeUrl(string) {
  return !isAbsoluteUrl(string) && !string.startsWith('//');
}

// Resolve relative URL
function resolveUrl(base, relative) {
  return new URL(relative, base).href;
}

console.log(resolveUrl('https://example.com/path/', '../other')); 
// 'https://example.com/other'
```

---

## Step 820: Broadcast Channel API

```javascript
// Broadcast Channel - สื่อสารระหว่าง tabs/windows ที่มี origin เดียวกัน

class TabCommunicator {
  constructor(channelName) {
    this.channel = new BroadcastChannel(channelName);
    this.id = `tab_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`;
    this.handlers = {};
    
    this.channel.addEventListener('message', (event) => {
      const { type, data, senderId } = event.data;
      
      // ไม่ process ข้อความที่ส่งเอง
      if (senderId === this.id) return;
      
      const handler = this.handlers[type];
      if (handler) {
        handler(data, { senderId, type });
      }
    });
    
    console.log(`Tab ${this.id} registered on channel ${channelName}`);
  }
  
  // ส่งข้อความไปยัง tabs อื่น
  broadcast(type, data) {
    this.channel.postMessage({
      type,
      data,
      senderId: this.id,
      timestamp: Date.now()
    });
  }
  
  // ฟัง event type
  on(type, handler) {
    this.handlers[type] = handler;
    return () => { delete this.handlers[type]; };
  }
  
  close() {
    this.channel.close();
  }
}

// ตัวอย่างการใช้งาน

// Tab 1: ส่ง logout event
const tabComm = new TabCommunicator('app-channel');

// เมื่อผู้ใช้ logout
function handleLogout() {
  // broadcast ไปยัง tabs อื่น
  tabComm.broadcast('USER_LOGOUT', { userId: 'user123' });
  
  // logout ตัวเอง
  clearSession();
  window.location.href = '/login';
}

// Tab 2: รับ logout event
tabComm.on('USER_LOGOUT', ({ userId }) => {
  console.log(`ผู้ใช้ ${userId} logout แล้ว กำลัง redirect...`);
  clearSession();
  window.location.href = '/login';
});

// Sync cart ระหว่าง tabs
tabComm.on('CART_UPDATED', (cart) => {
  console.log('Cart updated in another tab:', cart);
  updateCartUI(cart);
});

function updateCart(newItem) {
  const cart = [...currentCart, newItem];
  updateCartUI(cart);
  tabComm.broadcast('CART_UPDATED', cart);
}

// Sync theme
tabComm.on('THEME_CHANGED', ({ theme }) => {
  applyTheme(theme);
});

function changeTheme(theme) {
  applyTheme(theme);
  localStorage.setItem('theme', theme);
  tabComm.broadcast('THEME_CHANGED', { theme });
}

function clearSession() { console.log('Session cleared'); }
function updateCartUI(cart) { console.log('Cart UI updated:', cart); }
function applyTheme(theme) { 
  document.documentElement.setAttribute('data-theme', theme);
  console.log('Theme applied:', theme); 
}
const currentCart = [];
```

---

## Step 821: Page Visibility API

```javascript
// Page Visibility API - ตรวจสอบว่า tab ถูก focus หรือไม่

class PageVisibilityManager {
  constructor() {
    this.isVisible = !document.hidden;
    this.visibilityListeners = [];
    this.hiddenTime = null;
    this.totalHiddenTime = 0;
    
    document.addEventListener('visibilitychange', () => {
      this.isVisible = !document.hidden;
      
      if (document.hidden) {
        this.hiddenTime = Date.now();
        this.#notify('hidden');
      } else {
        if (this.hiddenTime) {
          this.totalHiddenTime += Date.now() - this.hiddenTime;
          this.hiddenTime = null;
        }
        this.#notify('visible');
      }
    });
  }
  
  #notify(state) {
    this.visibilityListeners.forEach(fn => fn(state));
  }
  
  onChange(fn) {
    this.visibilityListeners.push(fn);
    return () => {
      this.visibilityListeners = this.visibilityListeners.filter(f => f !== fn);
    };
  }
  
  getStats() {
    return {
      isVisible: this.isVisible,
      totalHiddenTime: this.totalHiddenTime,
      currentlyHidden: !!this.hiddenTime,
      hiddenSince: this.hiddenTime ? new Date(this.hiddenTime) : null
    };
  }
}

const visibilityManager = new PageVisibilityManager();

// หยุดอัพเดตเมื่อ tab ไม่ถูก focus
let updateInterval = null;

function startUpdates() {
  if (updateInterval) return;
  updateInterval = setInterval(updateDashboard, 5000);
  console.log('เริ่ม updates');
}

function stopUpdates() {
  if (updateInterval) {
    clearInterval(updateInterval);
    updateInterval = null;
    console.log('หยุด updates');
  }
}

visibilityManager.onChange((state) => {
  if (state === 'visible') {
    startUpdates();
    // อัพเดตทันทีเมื่อกลับมา
    updateDashboard();
  } else {
    stopUpdates();
  }
});

// เริ่มถ้า visible
if (visibilityManager.isVisible) {
  startUpdates();
}

function updateDashboard() {
  console.log('Updating dashboard...', new Date().toLocaleTimeString());
}

// Video auto-pause
const videoElement = document.querySelector('video');

if (videoElement) {
  visibilityManager.onChange((state) => {
    if (state === 'hidden') {
      videoElement.pause();
    } else if (!videoElement.paused) {
      videoElement.play();
    }
  });
}

// Analytics - ส่ง engagement time
window.addEventListener('beforeunload', () => {
  const stats = visibilityManager.getStats();
  const sessionTime = Date.now() - window.performance.timing.navigationStart;
  const activeTime = sessionTime - stats.totalHiddenTime;
  
  console.log(`Session time: ${sessionTime}ms, Active time: ${activeTime}ms`);
  // ส่งข้อมูลไปยัง analytics
});
```

---

## Step 822: Fullscreen API

```javascript
// Fullscreen API

class FullscreenManager {
  constructor() {
    this.element = null;
    this.listeners = [];
    
    // ฟัง fullscreen change events
    const events = [
      'fullscreenchange',
      'webkitfullscreenchange',
      'mozfullscreenchange',
      'MSFullscreenChange'
    ];
    
    events.forEach(event => {
      document.addEventListener(event, () => {
        this.#notify(this.isFullscreen());
      });
    });
    
    // ฟัง fullscreen error
    const errors = [
      'fullscreenerror',
      'webkitfullscreenerror'
    ];
    
    errors.forEach(event => {
      document.addEventListener(event, (e) => {
        console.error('Fullscreen error:', e);
      });
    });
  }
  
  isFullscreen() {
    return !!(
      document.fullscreenElement ||
      document.webkitFullscreenElement ||
      document.mozFullScreenElement ||
      document.msFullscreenElement
    );
  }
  
  isSupported() {
    return !!(
      document.fullscreenEnabled ||
      document.webkitFullscreenEnabled ||
      document.mozFullScreenEnabled ||
      document.msFullscreenEnabled
    );
  }
  
  async enter(element = document.documentElement) {
    if (!this.isSupported()) {
      throw new Error('Fullscreen ไม่รองรับ');
    }
    
    this.element = element;
    
    const requestFn = element.requestFullscreen ||
                     element.webkitRequestFullscreen ||
                     element.mozRequestFullScreen ||
                     element.msRequestFullscreen;
    
    if (!requestFn) {
      throw new Error('Element ไม่รองรับ fullscreen');
    }
    
    await requestFn.call(element, { navigationUI: 'hide' });
  }
  
  async exit() {
    const exitFn = document.exitFullscreen ||
                  document.webkitExitFullscreen ||
                  document.mozCancelFullScreen ||
                  document.msExitFullscreen;
    
    if (exitFn) {
      await exitFn.call(document);
    }
  }
  
  async toggle(element) {
    if (this.isFullscreen()) {
      await this.exit();
    } else {
      await this.enter(element);
    }
  }
  
  onChange(fn) {
    this.listeners.push(fn);
    return () => {
      this.listeners = this.listeners.filter(f => f !== fn);
    };
  }
  
  #notify(isFullscreen) {
    this.listeners.forEach(fn => fn(isFullscreen));
  }
}

const fullscreen = new FullscreenManager();

// ตัวอย่าง video player
const videoContainer = document.getElementById('video-container');
const fullscreenBtn = document.getElementById('fullscreen-btn');

fullscreenBtn?.addEventListener('click', async () => {
  try {
    await fullscreen.toggle(videoContainer);
  } catch (error) {
    console.error('ไม่สามารถเข้า fullscreen:', error.message);
  }
});

fullscreen.onChange((isFullscreen) => {
  if (fullscreenBtn) {
    fullscreenBtn.textContent = isFullscreen ? '⊡ ออกจาก Fullscreen' : '⊞ เต็มหน้าจอ';
  }
  console.log('Fullscreen:', isFullscreen);
});

// Keyboard shortcut F
document.addEventListener('keydown', async (e) => {
  if (e.key === 'f' || e.key === 'F') {
    if (!e.ctrlKey && !e.altKey && !e.metaKey) {
      await fullscreen.toggle(videoContainer);
    }
  }
  
  if (e.key === 'Escape' && fullscreen.isFullscreen()) {
    await fullscreen.exit();
  }
});
```

---

## Step 823: Speech Recognition API

```javascript
// Speech Recognition API (Web Speech API)

class SpeechRecognizer {
  constructor(options = {}) {
    const SpeechRecognition = window.SpeechRecognition || window.webkitSpeechRecognition;
    
    if (!SpeechRecognition) {
      throw new Error('Speech Recognition ไม่รองรับบน browser นี้');
    }
    
    this.recognition = new SpeechRecognition();
    this.isListening = false;
    this.transcript = '';
    this.interimTranscript = '';
    
    // Configuration
    this.recognition.lang = options.lang || 'th-TH';      // ภาษาไทย
    this.recognition.continuous = options.continuous ?? true; // ฟังต่อเนื่อง
    this.recognition.interimResults = options.interim ?? true; // แสดงผลชั่วคราว
    this.recognition.maxAlternatives = options.maxAlt || 3;   // จำนวนตัวเลือก
    
    this.#setupListeners(options);
  }
  
  #setupListeners(options) {
    this.recognition.onstart = () => {
      this.isListening = true;
      console.log('🎤 เริ่มรับเสียง...');
      options.onStart?.();
    };
    
    this.recognition.onend = () => {
      this.isListening = false;
      console.log('🔇 หยุดรับเสียง');
      options.onEnd?.();
    };
    
    this.recognition.onresult = (event) => {
      let interim = '';
      let final = '';
      
      for (let i = event.resultIndex; i < event.results.length; i++) {
        const result = event.results[i];
        const text = result[0].transcript;
        
        if (result.isFinal) {
          final += text;
        } else {
          interim += text;
        }
      }
      
      if (final) {
        this.transcript += final;
        options.onFinalResult?.(final, this.transcript);
      }
      
      this.interimTranscript = interim;
      options.onResult?.(interim, final, this.transcript);
    };
    
    this.recognition.onerror = (event) => {
      const errors = {
        'no-speech': 'ไม่พบเสียงพูด',
        'aborted': 'การรับเสียงถูกยกเลิก',
        'audio-capture': 'ไม่สามารถรับสัญญาณเสียงได้',
        'network': 'ปัญหาเครือข่าย',
        'not-allowed': 'ไม่ได้รับสิทธิ์ใช้ไมโครโฟน',
        'service-not-allowed': 'ไม่อนุญาตให้ใช้บริการ',
        'bad-grammar': 'รูปแบบ grammar ไม่ถูกต้อง',
        'language-not-supported': 'ภาษาไม่รองรับ'
      };
      
      const message = errors[event.error] || `ข้อผิดพลาด: ${event.error}`;
      console.error('Speech error:', message);
      options.onError?.(event.error, message);
    };
    
    this.recognition.onnomatch = () => {
      console.log('ไม่สามารถจดจำเสียงได้');
      options.onNoMatch?.();
    };
  }
  
  start() {
    if (this.isListening) return;
    this.recognition.start();
  }
  
  stop() {
    if (!this.isListening) return;
    this.recognition.stop();
  }
  
  abort() {
    this.recognition.abort();
  }
  
  clearTranscript() {
    this.transcript = '';
    this.interimTranscript = '';
  }
  
  getTranscript() {
    return this.transcript;
  }
}

// Voice Command system
class VoiceCommandManager {
  constructor(lang = 'th-TH') {
    this.commands = new Map();
    this.recognizer = null;
    this.lang = lang;
  }
  
  addCommand(phrase, handler, options = {}) {
    this.commands.set(phrase.toLowerCase(), { handler, options });
    return this;
  }
  
  removeCommand(phrase) {
    this.commands.delete(phrase.toLowerCase());
  }
  
  async start() {
    this.recognizer = new SpeechRecognizer({
      lang: this.lang,
      continuous: true,
      onFinalResult: (text) => {
        this.#processCommand(text.toLowerCase().trim());
      },
      onError: (error) => {
        if (error !== 'no-speech') {
          console.error('Voice command error:', error);
        }
      }
    });
    
    this.recognizer.start();
  }
  
  stop() {
    this.recognizer?.stop();
  }
  
  #processCommand(text) {
    for (const [phrase, { handler }] of this.commands) {
      if (text.includes(phrase)) {
        console.log(`Command recognized: "${phrase}"`);
        handler(text);
        return;
      }
    }
    
    console.log(`ไม่พบคำสั่ง: "${text}"`);
  }
}

// ใช้งาน Voice Commands
const voiceCommands = new VoiceCommandManager('th-TH');

voiceCommands
  .addCommand('เปิดเมนู', () => {
    console.log('เปิดเมนู');
    document.getElementById('menu')?.classList.add('open');
  })
  .addCommand('ปิดเมนู', () => {
    console.log('ปิดเมนู');
    document.getElementById('menu')?.classList.remove('open');
  })
  .addCommand('ไปหน้าแรก', () => {
    window.location.href = '/';
  })
  .addCommand('ค้นหา', (text) => {
    const query = text.replace('ค้นหา', '').trim();
    console.log('ค้นหา:', query);
  });
```

---

## Step 824: Speech Synthesis API

```javascript
// Speech Synthesis API - Text-to-Speech

class TextToSpeech {
  constructor() {
    this.synth = window.speechSynthesis;
    this.voices = [];
    this.currentUtterance = null;
    
    // Load voices
    this.#loadVoices();
    
    // Some browsers load voices asynchronously
    if (speechSynthesis.onvoiceschanged !== undefined) {
      speechSynthesis.addEventListener('voiceschanged', () => {
        this.#loadVoices();
      });
    }
  }
  
  #loadVoices() {
    this.voices = this.synth.getVoices();
  }
  
  isSupported() {
    return 'speechSynthesis' in window;
  }
  
  getVoices(lang = null) {
    if (lang) {
      return this.voices.filter(v => v.lang.startsWith(lang));
    }
    return this.voices;
  }
  
  getThaiVoices() {
    return this.getVoices('th');
  }
  
  speak(text, options = {}) {
    return new Promise((resolve, reject) => {
      if (!this.isSupported()) {
        reject(new Error('Speech Synthesis ไม่รองรับ'));
        return;
      }
      
      // หยุด speech ที่กำลังพูดอยู่
      this.synth.cancel();
      
      const utterance = new SpeechSynthesisUtterance(text);
      this.currentUtterance = utterance;
      
      // ตั้งค่าเสียง
      if (options.voice) {
        utterance.voice = options.voice;
      } else if (options.lang) {
        const voices = this.getVoices(options.lang);
        if (voices.length > 0) utterance.voice = voices[0];
      }
      
      utterance.lang = options.lang || 'th-TH';
      utterance.rate = options.rate || 1;        // 0.1 - 10
      utterance.pitch = options.pitch || 1;      // 0 - 2
      utterance.volume = options.volume || 1;    // 0 - 1
      
      utterance.onstart = () => {
        console.log('🔊 เริ่มพูด:', text.substring(0, 50));
        options.onStart?.();
      };
      
      utterance.onend = () => {
        console.log('✓ พูดเสร็จแล้ว');
        this.currentUtterance = null;
        resolve();
        options.onEnd?.();
      };
      
      utterance.onerror = (event) => {
        console.error('Speech error:', event.error);
        this.currentUtterance = null;
        reject(new Error(event.error));
        options.onError?.(event.error);
      };
      
      utterance.onpause = () => options.onPause?.();
      utterance.onresume = () => options.onResume?.();
      
      utterance.onboundary = (event) => {
        // เรียกเมื่อถึง word หรือ sentence boundary
        options.onBoundary?.(event.name, event.charIndex);
      };
      
      this.synth.speak(utterance);
    });
  }
  
  pause() {
    this.synth.pause();
  }
  
  resume() {
    this.synth.resume();
  }
  
  stop() {
    this.synth.cancel();
    this.currentUtterance = null;
  }
  
  isSpeaking() {
    return this.synth.speaking;
  }
  
  isPaused() {
    return this.synth.paused;
  }
  
  // พูดข้อความยาว (split เป็น chunks)
  async speakLong(text, options = {}) {
    const sentences = text.match(/[^.!?]+[.!?]+/g) || [text];
    
    for (const sentence of sentences) {
      if (sentence.trim()) {
        await this.speak(sentence.trim(), options);
        // หยุดสั้นๆ ระหว่าง sentences
        await new Promise(r => setTimeout(r, 100));
      }
    }
  }
}

// ใช้งาน
const tts = new TextToSpeech();

// แสดงรายชื่อ voices ที่ใช้ได้
console.log('Thai voices:', tts.getThaiVoices().map(v => v.name));

// พูดข้อความ
async function speakText() {
  await tts.speak('สวัสดีครับ ยินดีต้อนรับสู่เว็บไซต์ของเรา', {
    lang: 'th-TH',
    rate: 0.9,
    pitch: 1.1,
    onStart: () => console.log('กำลังพูด...'),
    onEnd: () => console.log('พูดเสร็จแล้ว')
  });
}

// Screen reader utility
function announceMessage(message) {
  if (tts.isSupported()) {
    tts.speak(message, { lang: 'th-TH', volume: 0.8 });
  }
}
```

---

## Step 825: Share API

```javascript
// Web Share API

class WebShareManager {
  isSupported() {
    return 'share' in navigator;
  }
  
  canShare(data) {
    if (!this.isSupported()) return false;
    return navigator.canShare?.(data) ?? true;
  }
  
  async share(data) {
    if (!this.isSupported()) {
      // Fallback to clipboard
      const text = [data.title, data.text, data.url].filter(Boolean).join('\n');
      await navigator.clipboard?.writeText(text);
      throw new Error('Web Share ไม่รองรับ - คัดลอกไปยัง clipboard แล้ว');
    }
    
    try {
      await navigator.share({
        title: data.title,
        text: data.text,
        url: data.url,
        files: data.files
      });
      
      console.log('แชร์สำเร็จ');
      return true;
    } catch (error) {
      if (error.name === 'AbortError') {
        console.log('ผู้ใช้ยกเลิกการแชร์');
        return false;
      }
      throw error;
    }
  }
  
  async shareCurrentPage(extra = {}) {
    return this.share({
      title: document.title,
      url: window.location.href,
      ...extra
    });
  }
  
  async shareImage(imageBlob, title = '', text = '') {
    const file = new File([imageBlob], 'image.png', { type: 'image/png' });
    
    if (!this.canShare({ files: [file] })) {
      throw new Error('ไม่สามารถแชร์ไฟล์ได้');
    }
    
    return this.share({ title, text, files: [file] });
  }
}

const webShare = new WebShareManager();

// Share button
document.getElementById('share-btn')?.addEventListener('click', async () => {
  try {
    const shared = await webShare.share({
      title: 'บทความน่าอ่าน',
      text: 'มาอ่านบทความนี้ด้วยกัน!',
      url: window.location.href
    });
    
    if (shared) {
      console.log('แชร์แล้ว!');
    }
  } catch (error) {
    console.error('แชร์ไม่สำเร็จ:', error.message);
  }
});
```

---

## Step 826-830: Permissions API และ Feature Detection

```javascript
// Permissions API

class PermissionManager {
  async check(permission) {
    if (!('permissions' in navigator)) {
      console.warn('Permissions API ไม่รองรับ');
      return 'unknown';
    }
    
    try {
      const result = await navigator.permissions.query({ name: permission });
      return result.state; // 'granted', 'denied', 'prompt'
    } catch (error) {
      console.error(`ไม่สามารถตรวจสอบ permission ${permission}:`, error.message);
      return 'unknown';
    }
  }
  
  async checkAll(permissions) {
    const results = {};
    for (const permission of permissions) {
      results[permission] = await this.check(permission);
    }
    return results;
  }
  
  async watchPermission(permission, callback) {
    if (!('permissions' in navigator)) return null;
    
    try {
      const result = await navigator.permissions.query({ name: permission });
      
      result.addEventListener('change', () => {
        callback(result.state, permission);
      });
      
      return result;
    } catch (error) {
      console.error(`ไม่สามารถติดตาม permission ${permission}:`, error.message);
      return null;
    }
  }
}

const permissions = new PermissionManager();

// ตรวจสอบ permissions ทั้งหมด
async function checkAppPermissions() {
  const permissionList = [
    'geolocation',
    'notifications',
    'camera',
    'microphone',
    'clipboard-read',
    'clipboard-write'
  ];
  
  const results = await permissions.checkAll(permissionList);
  console.log('App permissions:', results);
  return results;
}

// Screen API
function getScreenInfo() {
  return {
    width: screen.width,
    height: screen.height,
    availWidth: screen.availWidth,   // ไม่รวม taskbar
    availHeight: screen.availHeight,
    colorDepth: screen.colorDepth,   // bits per pixel
    pixelDepth: screen.pixelDepth,
    orientation: screen.orientation?.type // 'landscape-primary', 'portrait-primary'
  };
}

// Window size
function getWindowSize() {
  return {
    innerWidth: window.innerWidth,
    innerHeight: window.innerHeight,
    outerWidth: window.outerWidth,
    outerHeight: window.outerHeight,
    devicePixelRatio: window.devicePixelRatio // 1, 1.5, 2 สำหรับ Retina
  };
}

// Responsive utilities
function getBreakpoint() {
  const width = window.innerWidth;
  if (width < 576) return 'xs';
  if (width < 768) return 'sm';
  if (width < 992) return 'md';
  if (width < 1200) return 'lg';
  return 'xl';
}

// Screen orientation
screen.orientation?.addEventListener('change', () => {
  const { type, angle } = screen.orientation;
  console.log(`หน้าจอหมุน: ${type} (${angle}°)`);
});

// Media query listener
const mobileQuery = window.matchMedia('(max-width: 768px)');

function handleMobileChange(e) {
  if (e.matches) {
    console.log('Mobile view');
    document.body.classList.add('mobile');
  } else {
    console.log('Desktop view');
    document.body.classList.remove('mobile');
  }
}

mobileQuery.addEventListener('change', handleMobileChange);
handleMobileChange(mobileQuery); // ตรวจสอบค่าเริ่มต้น

// Dark mode detection
const darkModeQuery = window.matchMedia('(prefers-color-scheme: dark)');

function handleDarkMode(e) {
  if (e.matches) {
    console.log('Dark mode');
    document.documentElement.setAttribute('data-theme', 'dark');
  } else {
    console.log('Light mode');
    document.documentElement.setAttribute('data-theme', 'light');
  }
}

darkModeQuery.addEventListener('change', handleDarkMode);
handleDarkMode(darkModeQuery);
```

---

## แบบฝึกหัด (Exercises)

### แบบฝึกหัดที่ 1: Location-Based Service

```javascript
// TODO: สร้าง class LocationBasedService ที่:
// 1. ขอตำแหน่ง user
// 2. หาสถานที่ใกล้เคียงจาก API
// 3. แสดง notifications เมื่อเข้าใกล้สถานที่น่าสนใจ
// 4. บันทึกประวัติการเดินทาง
// 5. คำนวณระยะทางที่เดินทาง

class LocationBasedService {
  // TODO: implement
}
```

### แบบฝึกหัดที่ 2: Smart Notification System

```javascript
// TODO: สร้างระบบ notifications ที่:
// 1. Queue notifications และแสดงทีละอัน
// 2. Group notifications ประเภทเดียวกัน
// 3. ส่ง notification ตามเวลาที่กำหนด
// 4. Support notification templates
// 5. Track notification interactions

class SmartNotificationSystem {
  // TODO: implement
}
```

### แบบฝึกหัดที่ 3: Offline-First App

```javascript
// TODO: สร้าง Offline-First architecture ที่:
// 1. ตรวจสอบ network status
// 2. Queue operations ขณะ offline
// 3. Sync เมื่อกลับมา online
// 4. แสดง UI state ที่เหมาะสม

class OfflineManager {
  // TODO: implement
}
```

### แบบฝึกหัดที่ 4: Voice Assistant

```javascript
// TODO: สร้าง Voice Assistant ที่:
// 1. รับคำสั่งเสียง
// 2. ประมวลผลคำสั่ง
// 3. ตอบกลับด้วยเสียง
// 4. Support follow-up questions

class VoiceAssistant {
  // TODO: implement
}
```

---

## สรุป

Browser APIs เปิดโอกาสให้ JavaScript โต้ตอบกับ browser และอุปกรณ์ได้อย่างลึกซึ้ง:

- **Navigator**: ข้อมูล browser, online status, feature detection
- **Geolocation**: ตำแหน่ง GPS แบบ real-time
- **Clipboard**: อ่าน/เขียน clipboard
- **Notifications**: แจ้งเตือนผู้ใช้
- **Speech APIs**: รับเสียง/อ่านออกเสียง
- **History & URL**: จัดการ navigation
- **Broadcast Channel**: สื่อสารระหว่าง tabs
- **Visibility**: รู้ว่า tab ถูก focus หรือไม่
- **Fullscreen**: เต็มหน้าจอ

---

*ต่อไป: Part 43 - Web Storage ขั้นสูง*
