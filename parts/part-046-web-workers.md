# Part 46: Web Workers

## ทำไมต้องใช้ Web Workers? JavaScript แบบ Single-Threaded

JavaScript ทำงานแบบ single-threaded หมายความว่าโค้ดทั้งหมดรันบน thread เดียวกัน หากมีการคำนวณหนักๆ เช่น การประมวลผลข้อมูลขนาดใหญ่ หน้าเว็บจะหยุดตอบสนองต่อผู้ใช้

**Steps 891-910**

---

## Step 891: ทำความเข้าใจ JavaScript Single-Threaded

```javascript
// ปัญหาของ single-threaded JavaScript
console.log("เริ่มต้น");

// การคำนวณที่ใช้เวลานาน - จะบล็อก UI
function heavyCalculation() {
  let result = 0;
  for (let i = 0; i < 1000000000; i++) {
    result += i;
  }
  return result;
}

// ขณะที่ฟังก์ชันนี้ทำงาน ผู้ใช้ไม่สามารถคลิกปุ่มหรือ scroll ได้
const result = heavyCalculation();
console.log("ผลลัพธ์:", result);
console.log("เสร็จสิ้น");

// Event Loop ทำงานอย่างไร
// 1. Call Stack - เก็บ function calls
// 2. Web APIs - setTimeout, fetch, DOM events
// 3. Callback Queue - เก็บ callbacks ที่รอ
// 4. Microtask Queue - เก็บ Promise callbacks
```

```javascript
// ตัวอย่าง: UI ที่หยุดตอบสนอง
document.getElementById("btn").addEventListener("click", () => {
  console.log("ปุ่มถูกกด");
  
  // จำลองการทำงานหนัก
  let sum = 0;
  for (let i = 0; i < 5000000000; i++) {
    sum += Math.sqrt(i);
  }
  
  // ขณะรันลูปนี้ ผู้ใช้ไม่สามารถทำอะไรกับหน้าเว็บได้เลย
  document.getElementById("result").textContent = sum;
});
```

---

## Step 892: Web Workers คืออะไร?

```javascript
// Web Workers ช่วยให้รันโค้ดบน background thread แยกต่างหาก
// ทำให้ main thread ยังตอบสนองต่อผู้ใช้ได้

// ข้อจำกัดของ Web Workers:
// 1. ไม่สามารถเข้าถึง DOM ได้โดยตรง
// 2. ไม่สามารถเข้าถึง window object บางส่วนได้
// 3. สื่อสารกับ main thread ผ่าน postMessage() เท่านั้น

// สิ่งที่ Web Workers เข้าถึงได้:
// - navigator object
// - location object (read-only)
// - XMLHttpRequest
// - fetch API
// - setTimeout/setInterval
// - importScripts()
// - WebSockets
// - IndexedDB

// โครงสร้างการสื่อสาร:
// Main Thread <--postMessage()--> Worker Thread
//             <--onmessage-----
```

---

## Step 893: สร้าง Web Worker แรก

```javascript
// main.js - ไฟล์หลัก

// สร้าง Worker จากไฟล์ worker
const worker = new Worker("worker.js");

// ส่งข้อความไปยัง Worker
worker.postMessage("สวัสดี Worker!");

// รับข้อความจาก Worker
worker.onmessage = function (event) {
  console.log("ได้รับจาก Worker:", event.data);
};

// จัดการ error
worker.onerror = function (error) {
  console.error("Worker error:", error.message);
};
```

```javascript
// worker.js - ไฟล์ Worker

// รับข้อความจาก main thread
self.onmessage = function (event) {
  console.log("Worker ได้รับ:", event.data);
  
  // ประมวลผลข้อมูล
  const result = "Worker ตอบกลับ: " + event.data;
  
  // ส่งข้อความกลับไปยัง main thread
  self.postMessage(result);
};
```

---

## Step 894: postMessage() และ onmessage Event

```javascript
// main.js

const worker = new Worker("calculator-worker.js");

// ส่งข้อมูลหลายประเภทได้
worker.postMessage(42);                          // ตัวเลข
worker.postMessage("Hello");                     // string
worker.postMessage([1, 2, 3]);                   // array
worker.postMessage({ x: 10, y: 20 });           // object
worker.postMessage(true);                        // boolean

// รับผลลัพธ์
worker.onmessage = (event) => {
  console.log("ผลลัพธ์:", event.data);
  console.log("ประเภทข้อมูล:", typeof event.data);
};

// ใช้ addEventListener แทน onmessage ก็ได้
worker.addEventListener("message", (event) => {
  console.log("ข้อความ:", event.data);
});
```

```javascript
// calculator-worker.js

// ใช้ addEventListener หรือ onmessage ได้ทั้งคู่
self.addEventListener("message", (event) => {
  const data = event.data;
  
  // ตรวจสอบประเภทข้อมูลและประมวลผล
  if (typeof data === "number") {
    // คำนวณ factorial
    const factorial = calculateFactorial(data);
    self.postMessage({ type: "factorial", result: factorial });
  } else if (Array.isArray(data)) {
    // เรียงลำดับ array
    const sorted = [...data].sort((a, b) => a - b);
    self.postMessage({ type: "sorted", result: sorted });
  } else if (typeof data === "object") {
    // คำนวณระยะทาง
    const distance = Math.sqrt(data.x ** 2 + data.y ** 2);
    self.postMessage({ type: "distance", result: distance });
  }
});

function calculateFactorial(n) {
  if (n <= 1) return 1;
  return n * calculateFactorial(n - 1);
}
```

---

## Step 895: ส่งข้อมูลซับซ้อนผ่าน postMessage

```javascript
// การส่งข้อมูลผ่าน postMessage ใช้การ "clone" (structured clone)
// ซึ่งต่างจาก reference

// main.js
const worker = new Worker("data-worker.js");

// ส่ง object ซับซ้อน
const largeData = {
  id: 1,
  name: "ข้อมูลตัวอย่าง",
  numbers: new Array(100000).fill(0).map((_, i) => i),
  metadata: {
    created: new Date(),
    version: "1.0"
  }
};

console.time("postMessage");
worker.postMessage(largeData);  // ข้อมูลถูก clone ไม่ใช่ reference
console.timeEnd("postMessage");

// ข้อมูลต้นฉบับยังคงอยู่ใน main thread
console.log("ข้อมูลเดิม:", largeData.id);
```

```javascript
// data-worker.js
self.onmessage = (event) => {
  const data = event.data;
  
  // ประมวลผลข้อมูล
  const sum = data.numbers.reduce((acc, n) => acc + n, 0);
  const avg = sum / data.numbers.length;
  
  self.postMessage({
    id: data.id,
    sum: sum,
    average: avg,
    count: data.numbers.length
  });
};
```

---

## Step 896: Transferable Objects

```javascript
// Transferable Objects - ส่งข้อมูลโดยไม่ต้อง clone (เร็วกว่ามาก)
// ข้อมูลจะถูก "ย้าย" ไปยัง Worker แทนที่จะ copy
// ArrayBuffer, MessagePort, ImageBitmap, OffscreenCanvas

// main.js
const worker = new Worker("buffer-worker.js");

// สร้าง ArrayBuffer
const buffer = new ArrayBuffer(1024 * 1024 * 10); // 10MB
const view = new Uint8Array(buffer);

// เติมข้อมูล
for (let i = 0; i < view.length; i++) {
  view[i] = i % 256;
}

console.log("ก่อนส่ง - buffer.byteLength:", buffer.byteLength); // 10485760

// ส่งเป็น transferable (ไม่ clone แต่ย้าย ownership)
worker.postMessage(buffer, [buffer]);

// หลังส่ง - buffer ไม่สามารถใช้งานได้อีกใน main thread
console.log("หลังส่ง - buffer.byteLength:", buffer.byteLength); // 0

worker.onmessage = (event) => {
  const processedBuffer = event.data;
  console.log("ได้รับ buffer กลับมา:", processedBuffer.byteLength);
};
```

```javascript
// buffer-worker.js
self.onmessage = (event) => {
  const buffer = event.data;
  const view = new Uint8Array(buffer);
  
  // ประมวลผล buffer
  for (let i = 0; i < view.length; i++) {
    view[i] = (view[i] * 2) % 256;
  }
  
  // ส่งกลับเป็น transferable
  self.postMessage(buffer, [buffer]);
};
```

```javascript
// ตัวอย่างใช้กับ ImageData
// main.js
const canvas = document.createElement("canvas");
const ctx = canvas.getContext("2d");
canvas.width = 1920;
canvas.height = 1080;

// วาดภาพ
ctx.fillStyle = "red";
ctx.fillRect(0, 0, 100, 100);

const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
const buffer = imageData.data.buffer;

const worker = new Worker("image-worker.js");

// ส่ง buffer เป็น transferable
worker.postMessage(
  { buffer, width: canvas.width, height: canvas.height },
  [buffer]
);

worker.onmessage = (event) => {
  const processedBuffer = event.data;
  const processedImageData = new ImageData(
    new Uint8ClampedArray(processedBuffer),
    canvas.width,
    canvas.height
  );
  ctx.putImageData(processedImageData, 0, 0);
};
```

---

## Step 897: Worker Scope - ไม่สามารถเข้าถึง DOM

```javascript
// worker.js - สิ่งที่ทำได้และทำไม่ได้

// ไม่สามารถทำได้:
// document.getElementById("btn")  // Error!
// window.alert("hello")           // Error!
// document.createElement("div")   // Error!

// สิ่งที่ทำได้:
console.log(self === globalThis);  // true ใน Worker

// เข้าถึง navigator
console.log(navigator.userAgent);
console.log(navigator.onLine);

// เข้าถึง location
console.log(location.href);

// ใช้ fetch
fetch("/api/data")
  .then(res => res.json())
  .then(data => self.postMessage(data));

// ใช้ setTimeout
setTimeout(() => {
  self.postMessage("3 วินาทีผ่านไป");
}, 3000);

// ใช้ IndexedDB
const request = indexedDB.open("WorkerDB", 1);

// ใช้ WebSocket
const ws = new WebSocket("wss://example.com");
ws.onmessage = (e) => self.postMessage(e.data);
```

```javascript
// ตัวอย่าง: Worker ที่ทำ API calls
// api-worker.js

self.onmessage = async (event) => {
  const { url, options } = event.data;
  
  try {
    const response = await fetch(url, options);
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    const data = await response.json();
    self.postMessage({ success: true, data });
  } catch (error) {
    self.postMessage({ success: false, error: error.message });
  }
};
```

---

## Step 898: Importing Scripts ใน Worker

```javascript
// worker.js

// วิธีที่ 1: importScripts() - แบบ synchronous
importScripts("utils.js", "math-lib.js");

// ตอนนี้ใช้ฟังก์ชันจาก utils.js และ math-lib.js ได้
self.onmessage = (event) => {
  const result = mathLibrary.calculate(event.data);
  self.postMessage(result);
};
```

```javascript
// utils.js (ไฟล์ที่ import)
function formatNumber(n) {
  return n.toLocaleString("th-TH");
}

function debounce(fn, delay) {
  let timer;
  return function (...args) {
    clearTimeout(timer);
    timer = setTimeout(() => fn.apply(this, args), delay);
  };
}
```

```javascript
// worker-es-module.js - ใช้ ES Modules ใน Worker
// (ต้องระบุ type: 'module' ตอนสร้าง Worker)

import { calculate } from "./math-utils.js";
import { formatData } from "./data-utils.js";

self.onmessage = (event) => {
  const processed = calculate(event.data);
  const formatted = formatData(processed);
  self.postMessage(formatted);
};
```

```javascript
// main.js - สร้าง Worker แบบ ES Module
const worker = new Worker("worker-es-module.js", {
  type: "module"  // ระบุว่าเป็น ES Module
});
```

---

## Step 899: Terminating Workers

```javascript
// main.js

const worker = new Worker("long-task-worker.js");

worker.postMessage({ task: "startHeavyComputation" });

// หยุด Worker จาก main thread
setTimeout(() => {
  worker.terminate();  // หยุด Worker ทันที
  console.log("Worker ถูกหยุดแล้ว");
}, 5000);

// หยุด Worker จากภายใน Worker
// worker.js
self.onmessage = (event) => {
  if (event.data.command === "stop") {
    self.close();  // หยุด Worker จากข้างใน
    console.log("Worker ปิดตัวเอง");
  }
};
```

```javascript
// ตัวอย่าง: Worker ที่มีระบบยกเลิก
// main.js

class CancellableTask {
  constructor(workerUrl) {
    this.worker = new Worker(workerUrl);
    this.isRunning = false;
  }
  
  start(data) {
    return new Promise((resolve, reject) => {
      this.isRunning = true;
      
      this.worker.postMessage({ command: "start", data });
      
      this.worker.onmessage = (event) => {
        if (event.data.type === "result") {
          this.isRunning = false;
          resolve(event.data.value);
        } else if (event.data.type === "progress") {
          console.log(`ความคืบหน้า: ${event.data.percent}%`);
        }
      };
      
      this.worker.onerror = (error) => {
        this.isRunning = false;
        reject(error);
      };
    });
  }
  
  cancel() {
    if (this.isRunning) {
      this.worker.postMessage({ command: "cancel" });
      this.isRunning = false;
    }
  }
  
  destroy() {
    this.worker.terminate();
  }
}

// การใช้งาน
const task = new CancellableTask("computation-worker.js");

task.start({ numbers: new Array(10000000).fill(Math.random()) })
  .then(result => console.log("ผลลัพธ์:", result))
  .catch(err => console.error("Error:", err));

// ยกเลิกหลัง 2 วินาที
setTimeout(() => {
  task.cancel();
  console.log("ยกเลิกงานแล้ว");
}, 2000);
```

---

## Step 900: Error Handling ใน Workers

```javascript
// main.js

const worker = new Worker("error-worker.js");

// จัดการ error จาก Worker
worker.onerror = (errorEvent) => {
  console.error("Worker Error:", {
    message: errorEvent.message,
    filename: errorEvent.filename,
    lineno: errorEvent.lineno,
    colno: errorEvent.colno
  });
  
  // ตัดสินใจว่าจะ terminate หรือ restart
  errorEvent.preventDefault();  // ป้องกัน error propagate ขึ้นไป
};

// จัดการ messageerror (เมื่อไม่สามารถ deserialize message ได้)
worker.addEventListener("messageerror", (event) => {
  console.error("ไม่สามารถ deserialize ข้อความ:", event);
});

worker.postMessage({ action: "doSomething" });
```

```javascript
// error-worker.js

self.onmessage = (event) => {
  try {
    const result = riskyOperation(event.data);
    self.postMessage({ success: true, result });
  } catch (error) {
    // ส่ง error กลับไปยัง main thread
    self.postMessage({
      success: false,
      error: {
        message: error.message,
        stack: error.stack,
        type: error.constructor.name
      }
    });
  }
};

function riskyOperation(data) {
  if (!data.action) {
    throw new TypeError("ต้องระบุ action");
  }
  
  if (data.action === "divide") {
    if (data.divisor === 0) {
      throw new RangeError("ไม่สามารถหารด้วยศูนย์ได้");
    }
    return data.dividend / data.divisor;
  }
  
  throw new Error(`ไม่รู้จัก action: ${data.action}`);
}

// จัดการ unhandled errors ภายใน Worker
self.addEventListener("error", (event) => {
  console.error("Unhandled error in Worker:", event.message);
  // ส่ง error ไปยัง main thread
  self.postMessage({
    type: "UNHANDLED_ERROR",
    message: event.message,
    filename: event.filename,
    line: event.lineno
  });
});
```

---

## Step 901: Dedicated Workers vs Shared Workers vs Service Workers

```javascript
// 1. Dedicated Worker - ใช้กับหน้าเดียว
// main.js
const dedicatedWorker = new Worker("dedicated.js");
dedicatedWorker.postMessage("เฉพาะแท็บนี้");

// 2. Shared Worker - ใช้ร่วมกันได้หลายแท็บ/หน้า
// main.js
const sharedWorker = new SharedWorker("shared.js");
sharedWorker.port.start();
sharedWorker.port.postMessage("ข้อความจากแท็บ 1");
sharedWorker.port.onmessage = (event) => {
  console.log("ได้รับ:", event.data);
};

// 3. Service Worker - ทำงานเป็น proxy ระหว่าง browser และ network
// main.js
if ("serviceWorker" in navigator) {
  navigator.serviceWorker.register("/sw.js")
    .then(registration => {
      console.log("Service Worker ลงทะเบียนสำเร็จ:", registration.scope);
    });
}
```

---

## Step 902: Shared Worker - Multiple Tabs

```javascript
// shared-worker.js

// เก็บ list ของ ports ที่เชื่อมต่อ
const ports = [];

self.onconnect = (event) => {
  const port = event.ports[0];
  ports.push(port);
  
  port.start();
  
  console.log(`มีการเชื่อมต่อใหม่ รวม: ${ports.length} connections`);
  
  port.onmessage = (messageEvent) => {
    const message = messageEvent.data;
    
    // broadcast ไปยังทุก port
    if (message.type === "broadcast") {
      ports.forEach(p => {
        if (p !== port) {  // ไม่ส่งกลับไปยังผู้ส่ง
          p.postMessage({
            type: "broadcast",
            from: message.from,
            content: message.content
          });
        }
      });
    }
    
    // ส่งกลับเฉพาะผู้ส่ง
    if (message.type === "echo") {
      port.postMessage({ type: "echo", content: message.content });
    }
    
    // sync state ระหว่างแท็บ
    if (message.type === "updateState") {
      sharedState = { ...sharedState, ...message.data };
      // แจ้งทุกแท็บ
      ports.forEach(p => {
        p.postMessage({ type: "stateUpdate", state: sharedState });
      });
    }
  };
  
  // แจ้งเมื่อ port ปิด
  port.addEventListener("close", () => {
    const index = ports.indexOf(port);
    if (index > -1) {
      ports.splice(index, 1);
    }
    console.log(`Port ปิด เหลือ: ${ports.length} connections`);
  });
  
  // ส่ง state ปัจจุบันให้แท็บใหม่
  port.postMessage({ type: "init", state: sharedState });
};

let sharedState = {
  counter: 0,
  users: [],
  lastUpdate: Date.now()
};
```

```javascript
// tab1.js - ใช้ Shared Worker
const worker = new SharedWorker("shared-worker.js");
worker.port.start();

// ส่ง broadcast ไปยังทุกแท็บ
function sendBroadcast(message) {
  worker.port.postMessage({
    type: "broadcast",
    from: "Tab 1",
    content: message
  });
}

// รับข้อความ
worker.port.onmessage = (event) => {
  if (event.data.type === "broadcast") {
    console.log(`ข้อความจาก ${event.data.from}: ${event.data.content}`);
  } else if (event.data.type === "stateUpdate") {
    updateUI(event.data.state);
  }
};

function updateUI(state) {
  document.getElementById("counter").textContent = state.counter;
}
```

---

## Step 903: SharedArrayBuffer และ Atomics

```javascript
// SharedArrayBuffer ช่วยให้หลาย Worker ใช้ memory ร่วมกันได้
// (ต้องการ Cross-Origin Isolation headers)

// main.js
// ต้องตั้งค่า headers:
// Cross-Origin-Opener-Policy: same-origin
// Cross-Origin-Embedder-Policy: require-corp

const sharedBuffer = new SharedArrayBuffer(4 * 4); // 4 integers
const sharedArray = new Int32Array(sharedBuffer);

const worker1 = new Worker("atomic-worker-1.js");
const worker2 = new Worker("atomic-worker-2.js");

// ส่ง sharedBuffer ไปยังทั้งสอง Worker
worker1.postMessage({ buffer: sharedBuffer });
worker2.postMessage({ buffer: sharedBuffer });

// ตรวจสอบค่าหลังจาก Workers ทำงาน
setTimeout(() => {
  console.log("ค่าใน SharedArrayBuffer:", Array.from(sharedArray));
}, 2000);
```

```javascript
// atomic-worker-1.js
self.onmessage = (event) => {
  const array = new Int32Array(event.data.buffer);
  
  // Atomics.add() - atomic add (thread-safe)
  for (let i = 0; i < 1000; i++) {
    Atomics.add(array, 0, 1);  // index 0 เพิ่ม 1
  }
  
  // Atomics.store() - atomic write
  Atomics.store(array, 1, 42);
  
  // Atomics.load() - atomic read
  const value = Atomics.load(array, 0);
  console.log("Worker 1 อ่านค่า index 0:", value);
  
  // Atomics.compareExchange() - compare and swap
  const oldValue = Atomics.compareExchange(array, 2, 0, 100);
  console.log("Old value:", oldValue);
  
  // Atomics.wait() - รอจนกว่า value เปลี่ยน (ใน Worker เท่านั้น)
  // Atomics.wait(array, 3, 0);  // รอถ้า index 3 เป็น 0
  
  self.postMessage("Worker 1 เสร็จแล้ว");
};
```

```javascript
// atomic-worker-2.js
self.onmessage = (event) => {
  const array = new Int32Array(event.data.buffer);
  
  // เพิ่มค่าพร้อมกับ Worker 1 (thread-safe)
  for (let i = 0; i < 1000; i++) {
    Atomics.add(array, 0, 1);
  }
  
  // Atomics.notify() - ปลุก Workers ที่กำลัง wait
  Atomics.notify(array, 3, 1);  // ปลุก 1 Worker ที่ wait อยู่ที่ index 3
  
  self.postMessage("Worker 2 เสร็จแล้ว");
};
```

---

## Step 904: Worker Performance Patterns

```javascript
// Pattern 1: Worker Pool
class WorkerPool {
  constructor(workerUrl, poolSize = navigator.hardwareConcurrency || 4) {
    this.workers = [];
    this.queue = [];
    this.poolSize = poolSize;
    
    for (let i = 0; i < poolSize; i++) {
      const worker = new Worker(workerUrl);
      this.workers.push({ worker, busy: false });
      
      worker.onmessage = (event) => {
        this.handleWorkerResponse(i, event);
      };
      
      worker.onerror = (error) => {
        this.handleWorkerError(i, error);
      };
    }
  }
  
  handleWorkerResponse(workerIndex, event) {
    const workerInfo = this.workers[workerIndex];
    workerInfo.busy = false;
    
    if (workerInfo.resolve) {
      workerInfo.resolve(event.data);
      workerInfo.resolve = null;
      workerInfo.reject = null;
    }
    
    // ประมวลผล task ถัดไปถ้ามี
    this.processQueue();
  }
  
  handleWorkerError(workerIndex, error) {
    const workerInfo = this.workers[workerIndex];
    workerInfo.busy = false;
    
    if (workerInfo.reject) {
      workerInfo.reject(error);
      workerInfo.resolve = null;
      workerInfo.reject = null;
    }
    
    this.processQueue();
  }
  
  processQueue() {
    if (this.queue.length === 0) return;
    
    const freeWorker = this.workers.find(w => !w.busy);
    if (!freeWorker) return;
    
    const { data, resolve, reject } = this.queue.shift();
    freeWorker.busy = true;
    freeWorker.resolve = resolve;
    freeWorker.reject = reject;
    freeWorker.worker.postMessage(data);
  }
  
  run(data) {
    return new Promise((resolve, reject) => {
      const freeWorker = this.workers.find(w => !w.busy);
      
      if (freeWorker) {
        freeWorker.busy = true;
        freeWorker.resolve = resolve;
        freeWorker.reject = reject;
        freeWorker.worker.postMessage(data);
      } else {
        this.queue.push({ data, resolve, reject });
      }
    });
  }
  
  terminate() {
    this.workers.forEach(({ worker }) => worker.terminate());
    this.workers = [];
    this.queue = [];
  }
}

// การใช้งาน Worker Pool
const pool = new WorkerPool("computation-worker.js", 4);

// ส่งงาน 100 tasks ไปพร้อมกัน
const tasks = Array.from({ length: 100 }, (_, i) => ({ number: i * 1000 }));

Promise.all(tasks.map(task => pool.run(task)))
  .then(results => {
    console.log("ผลลัพธ์ทั้งหมด:", results.length);
    pool.terminate();
  });
```

---

## Step 905: Use Cases - Heavy Computation

```javascript
// ตัวอย่าง 1: Image Processing Worker
// image-processor.js

self.onmessage = (event) => {
  const { imageData, filter } = event.data;
  const data = imageData.data;
  
  switch (filter) {
    case "grayscale":
      for (let i = 0; i < data.length; i += 4) {
        const avg = (data[i] + data[i + 1] + data[i + 2]) / 3;
        data[i] = avg;      // R
        data[i + 1] = avg;  // G
        data[i + 2] = avg;  // B
        // data[i + 3] Alpha ไม่เปลี่ยน
      }
      break;
      
    case "invert":
      for (let i = 0; i < data.length; i += 4) {
        data[i] = 255 - data[i];
        data[i + 1] = 255 - data[i + 1];
        data[i + 2] = 255 - data[i + 2];
      }
      break;
      
    case "blur":
      // Gaussian blur (simplified)
      const kernel = [1/16, 2/16, 1/16, 2/16, 4/16, 2/16, 1/16, 2/16, 1/16];
      const width = imageData.width;
      const height = imageData.height;
      const tempData = new Uint8ClampedArray(data);
      
      for (let y = 1; y < height - 1; y++) {
        for (let x = 1; x < width - 1; x++) {
          let r = 0, g = 0, b = 0;
          let ki = 0;
          
          for (let ky = -1; ky <= 1; ky++) {
            for (let kx = -1; kx <= 1; kx++) {
              const idx = ((y + ky) * width + (x + kx)) * 4;
              r += tempData[idx] * kernel[ki];
              g += tempData[idx + 1] * kernel[ki];
              b += tempData[idx + 2] * kernel[ki];
              ki++;
            }
          }
          
          const idx = (y * width + x) * 4;
          data[idx] = r;
          data[idx + 1] = g;
          data[idx + 2] = b;
        }
      }
      break;
  }
  
  self.postMessage({ imageData }, [imageData.data.buffer]);
};
```

```javascript
// main.js - ใช้ Image Processing Worker
const canvas = document.getElementById("canvas");
const ctx = canvas.getContext("2d");
const img = new Image();

img.onload = () => {
  canvas.width = img.width;
  canvas.height = img.height;
  ctx.drawImage(img, 0, 0);
  
  const worker = new Worker("image-processor.js");
  
  // นำ ImageData ออกจาก canvas
  const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
  
  // ส่งไปประมวลผลใน Worker
  worker.postMessage({ imageData, filter: "grayscale" }, [imageData.data.buffer]);
  
  worker.onmessage = (event) => {
    ctx.putImageData(event.data.imageData, 0, 0);
    worker.terminate();
  };
};

img.src = "photo.jpg";
```

---

## Step 906: Use Cases - Data Processing

```javascript
// data-processing-worker.js

self.onmessage = (event) => {
  const { command, data } = event.data;
  
  switch (command) {
    case "sort":
      const sorted = quickSort(data);
      self.postMessage({ command: "sortResult", data: sorted });
      break;
      
    case "filter":
      const filtered = data.filter(item => item.value > event.data.threshold);
      self.postMessage({ command: "filterResult", data: filtered });
      break;
      
    case "aggregate":
      const stats = computeStatistics(data);
      self.postMessage({ command: "aggregateResult", data: stats });
      break;
      
    case "search":
      const results = fuzzySearch(data, event.data.query);
      self.postMessage({ command: "searchResult", data: results });
      break;
  }
};

function quickSort(arr) {
  if (arr.length <= 1) return arr;
  
  const pivot = arr[Math.floor(arr.length / 2)];
  const left = arr.filter(x => x < pivot);
  const middle = arr.filter(x => x === pivot);
  const right = arr.filter(x => x > pivot);
  
  return [...quickSort(left), ...middle, ...quickSort(right)];
}

function computeStatistics(data) {
  const n = data.length;
  const sum = data.reduce((a, b) => a + b, 0);
  const mean = sum / n;
  
  const sorted = [...data].sort((a, b) => a - b);
  const median = n % 2 === 0
    ? (sorted[n/2 - 1] + sorted[n/2]) / 2
    : sorted[Math.floor(n/2)];
  
  const variance = data.reduce((acc, x) => acc + (x - mean) ** 2, 0) / n;
  const stdDev = Math.sqrt(variance);
  
  return { sum, mean, median, variance, stdDev, min: sorted[0], max: sorted[n-1] };
}

function fuzzySearch(items, query) {
  const queryLower = query.toLowerCase();
  return items.filter(item => {
    const text = item.toString().toLowerCase();
    let queryIndex = 0;
    
    for (let i = 0; i < text.length; i++) {
      if (text[i] === queryLower[queryIndex]) {
        queryIndex++;
        if (queryIndex === queryLower.length) return true;
      }
    }
    return false;
  });
}
```

---

## Step 907: Comlink Library Concept

```javascript
// Comlink ทำให้การใช้ Worker ง่ายขึ้น เหมือนเรียกฟังก์ชันธรรมดา
// แนวคิดหลักของ Comlink

// worker.js (ถ้าใช้ Comlink จริง)
// import * as Comlink from "comlink";
// Comlink.expose({ add, multiply, processData });

// การ implement แนวคิดเดียวกันเอง:

// rpc-worker.js
const handlers = {};

function expose(obj) {
  for (const [key, value] of Object.entries(obj)) {
    if (typeof value === "function") {
      handlers[key] = value;
    }
  }
}

self.onmessage = async (event) => {
  const { id, method, args } = event.data;
  
  try {
    if (!handlers[method]) {
      throw new Error(`Method "${method}" not found`);
    }
    
    const result = await handlers[method](...args);
    self.postMessage({ id, result });
  } catch (error) {
    self.postMessage({ id, error: error.message });
  }
};

// Expose functions
expose({
  add: (a, b) => a + b,
  multiply: (a, b) => a * b,
  factorial: (n) => {
    if (n <= 1) return 1;
    return n * handlers.factorial(n - 1);
  },
  processLargeArray: async (arr) => {
    // จำลองงานที่ใช้เวลา
    return arr.map(x => x * 2).filter(x => x > 10).sort((a, b) => a - b);
  }
});
```

```javascript
// main.js - proxy สำหรับเรียกใช้ Worker เหมือนฟังก์ชันปกติ

function createWorkerProxy(workerUrl) {
  const worker = new Worker(workerUrl);
  let callId = 0;
  const pendingCalls = new Map();
  
  worker.onmessage = (event) => {
    const { id, result, error } = event.data;
    const { resolve, reject } = pendingCalls.get(id);
    pendingCalls.delete(id);
    
    if (error) {
      reject(new Error(error));
    } else {
      resolve(result);
    }
  };
  
  return new Proxy({}, {
    get: (_, method) => (...args) => {
      const id = callId++;
      return new Promise((resolve, reject) => {
        pendingCalls.set(id, { resolve, reject });
        worker.postMessage({ id, method, args });
      });
    }
  });
}

// การใช้งาน
const workerAPI = createWorkerProxy("rpc-worker.js");

async function main() {
  const sum = await workerAPI.add(5, 3);
  console.log("5 + 3 =", sum);  // 8
  
  const product = await workerAPI.multiply(4, 7);
  console.log("4 * 7 =", product);  // 28
  
  const fact = await workerAPI.factorial(10);
  console.log("10! =", fact);  // 3628800
  
  const processed = await workerAPI.processLargeArray([1, 15, 3, 8, 20, 5]);
  console.log("Processed:", processed);
}

main();
```

---

## Step 908: WorkerPool Pattern ขั้นสูง

```javascript
// advanced-worker-pool.js

class AdvancedWorkerPool {
  constructor(workerUrl, options = {}) {
    this.workerUrl = workerUrl;
    this.minWorkers = options.minWorkers || 2;
    this.maxWorkers = options.maxWorkers || navigator.hardwareConcurrency || 4;
    this.idleTimeout = options.idleTimeout || 60000; // 1 นาที
    
    this.workers = new Map();
    this.taskQueue = [];
    this.taskIdCounter = 0;
    
    // เริ่มต้น workers ขั้นต่ำ
    for (let i = 0; i < this.minWorkers; i++) {
      this.createWorker();
    }
  }
  
  createWorker() {
    const id = Date.now() + Math.random();
    const worker = new Worker(this.workerUrl);
    
    const workerInfo = {
      id,
      worker,
      busy: false,
      taskCount: 0,
      createdAt: Date.now(),
      lastIdleAt: Date.now(),
      currentTask: null
    };
    
    worker.onmessage = (event) => this.onWorkerMessage(id, event);
    worker.onerror = (error) => this.onWorkerError(id, error);
    
    this.workers.set(id, workerInfo);
    return workerInfo;
  }
  
  onWorkerMessage(workerId, event) {
    const workerInfo = this.workers.get(workerId);
    if (!workerInfo) return;
    
    const task = workerInfo.currentTask;
    if (task) {
      task.resolve(event.data);
      workerInfo.currentTask = null;
    }
    
    workerInfo.busy = false;
    workerInfo.lastIdleAt = Date.now();
    workerInfo.taskCount++;
    
    this.processQueue();
    this.scaleDown();
  }
  
  onWorkerError(workerId, error) {
    const workerInfo = this.workers.get(workerId);
    if (!workerInfo) return;
    
    const task = workerInfo.currentTask;
    if (task) {
      task.reject(error);
      workerInfo.currentTask = null;
    }
    
    // Restart worker
    workerInfo.worker.terminate();
    this.workers.delete(workerId);
    
    if (this.workers.size < this.minWorkers) {
      this.createWorker();
    }
    
    this.processQueue();
  }
  
  processQueue() {
    if (this.taskQueue.length === 0) return;
    
    let freeWorker = null;
    for (const [, workerInfo] of this.workers) {
      if (!workerInfo.busy) {
        freeWorker = workerInfo;
        break;
      }
    }
    
    if (!freeWorker && this.workers.size < this.maxWorkers) {
      freeWorker = this.createWorker();
    }
    
    if (!freeWorker) return;
    
    const task = this.taskQueue.shift();
    freeWorker.busy = true;
    freeWorker.currentTask = task;
    freeWorker.worker.postMessage(task.data);
  }
  
  scaleDown() {
    if (this.workers.size <= this.minWorkers) return;
    
    const now = Date.now();
    for (const [id, workerInfo] of this.workers) {
      if (!workerInfo.busy && 
          now - workerInfo.lastIdleAt > this.idleTimeout) {
        workerInfo.worker.terminate();
        this.workers.delete(id);
        
        if (this.workers.size <= this.minWorkers) break;
      }
    }
  }
  
  run(data) {
    return new Promise((resolve, reject) => {
      const task = {
        id: this.taskIdCounter++,
        data,
        resolve,
        reject,
        queuedAt: Date.now()
      };
      
      this.taskQueue.push(task);
      this.processQueue();
    });
  }
  
  getStats() {
    const workerCount = this.workers.size;
    const busyWorkers = [...this.workers.values()].filter(w => w.busy).length;
    const queueLength = this.taskQueue.length;
    const totalTasksCompleted = [...this.workers.values()]
      .reduce((sum, w) => sum + w.taskCount, 0);
    
    return { workerCount, busyWorkers, idleWorkers: workerCount - busyWorkers, queueLength, totalTasksCompleted };
  }
  
  terminate() {
    this.workers.forEach(({ worker }) => worker.terminate());
    this.workers.clear();
    
    this.taskQueue.forEach(task => task.reject(new Error("Pool terminated")));
    this.taskQueue = [];
  }
}

// การใช้งาน
const pool = new AdvancedWorkerPool("computation-worker.js", {
  minWorkers: 2,
  maxWorkers: 8,
  idleTimeout: 30000
});

async function runBenchmark() {
  const tasks = Array.from({ length: 50 }, (_, i) => ({ n: i * 100 }));
  
  console.log("เริ่มประมวลผล 50 tasks...");
  console.time("pool-benchmark");
  
  const results = await Promise.all(tasks.map(task => pool.run(task)));
  
  console.timeEnd("pool-benchmark");
  console.log("Stats:", pool.getStats());
  
  pool.terminate();
}

runBenchmark();
```

---

## Step 909: Inline Workers (Blob URL)

```javascript
// สร้าง Worker จาก string โดยตรง ไม่ต้องมีไฟล์แยก

function createInlineWorker(fn) {
  const code = fn.toString();
  // ดึง function body
  const body = code.slice(code.indexOf("{") + 1, code.lastIndexOf("}"));
  
  const blob = new Blob([body], { type: "application/javascript" });
  const url = URL.createObjectURL(blob);
  const worker = new Worker(url);
  
  // cleanup URL หลังสร้าง Worker
  URL.revokeObjectURL(url);
  
  return worker;
}

// สร้าง Worker จาก function
const worker = createInlineWorker(function () {
  self.onmessage = function (event) {
    const { numbers } = event.data;
    
    // คำนวณ prime numbers
    const primes = [];
    for (let n = 2; n <= Math.max(...numbers); n++) {
      if (isPrime(n) && numbers.includes(n)) {
        primes.push(n);
      }
    }
    
    self.postMessage({ primes });
  };
  
  function isPrime(n) {
    if (n < 2) return false;
    for (let i = 2; i <= Math.sqrt(n); i++) {
      if (n % i === 0) return false;
    }
    return true;
  }
});

worker.postMessage({ numbers: [2, 3, 4, 5, 6, 7, 8, 9, 10, 11, 13, 17, 19] });

worker.onmessage = (event) => {
  console.log("Prime numbers:", event.data.primes);
};
```

```javascript
// Template literal approach
function createWorkerFromString(workerCode) {
  const blob = new Blob([workerCode], { type: "text/javascript" });
  return new Worker(URL.createObjectURL(blob));
}

const workerCode = `
  importScripts("https://cdnjs.cloudflare.com/ajax/libs/lodash.js/4.17.21/lodash.min.js");
  
  self.onmessage = function(event) {
    const data = event.data;
    
    // ใช้ lodash ประมวลผล
    const result = _.chain(data)
      .groupBy("category")
      .mapValues(items => _.sumBy(items, "value"))
      .value();
    
    self.postMessage(result);
  };
`;

const worker = createWorkerFromString(workerCode);
```

---

## Step 910: Real-World Example - Background File Processing

```javascript
// file-processor-worker.js

self.onmessage = async (event) => {
  const { file, operation } = event.data;
  
  try {
    let result;
    
    switch (operation) {
      case "parseCSV":
        result = await parseCSV(file);
        break;
      case "parseJSON":
        result = await parseJSON(file);
        break;
      case "compress":
        result = await compressData(file);
        break;
      default:
        throw new Error(`Unknown operation: ${operation}`);
    }
    
    self.postMessage({ success: true, result, operation });
  } catch (error) {
    self.postMessage({ success: false, error: error.message, operation });
  }
};

async function parseCSV(arrayBuffer) {
  const text = new TextDecoder().decode(arrayBuffer);
  const lines = text.split("\n").filter(Boolean);
  const headers = lines[0].split(",").map(h => h.trim());
  
  const data = lines.slice(1).map(line => {
    const values = line.split(",");
    return headers.reduce((obj, header, i) => {
      obj[header] = values[i]?.trim() ?? "";
      return obj;
    }, {});
  });
  
  // ส่ง progress
  self.postMessage({ type: "progress", percent: 50 });
  
  return data;
}

async function parseJSON(arrayBuffer) {
  const text = new TextDecoder().decode(arrayBuffer);
  return JSON.parse(text);
}

async function compressData(arrayBuffer) {
  const stream = new CompressionStream("gzip");
  const writer = stream.writable.getWriter();
  const reader = stream.readable.getReader();
  
  writer.write(arrayBuffer);
  writer.close();
  
  const chunks = [];
  let done = false;
  
  while (!done) {
    const { value, done: isDone } = await reader.read();
    if (isDone) {
      done = true;
    } else {
      chunks.push(value);
    }
  }
  
  const totalLength = chunks.reduce((sum, chunk) => sum + chunk.length, 0);
  const compressed = new Uint8Array(totalLength);
  let offset = 0;
  for (const chunk of chunks) {
    compressed.set(chunk, offset);
    offset += chunk.length;
  }
  
  return compressed.buffer;
}
```

```javascript
// main.js - ใช้ File Processing Worker
const worker = new Worker("file-processor-worker.js");

document.getElementById("fileInput").addEventListener("change", async (event) => {
  const file = event.target.files[0];
  if (!file) return;
  
  const progressBar = document.getElementById("progress");
  const statusEl = document.getElementById("status");
  
  statusEl.textContent = "กำลังประมวลผลไฟล์...";
  
  // อ่านไฟล์เป็น ArrayBuffer
  const arrayBuffer = await file.arrayBuffer();
  
  // ส่งไปประมวลผลใน Worker
  const extension = file.name.split(".").pop().toLowerCase();
  const operation = extension === "csv" ? "parseCSV" : "parseJSON";
  
  worker.postMessage({ file: arrayBuffer, operation }, [arrayBuffer]);
  
  worker.onmessage = (event) => {
    if (event.data.type === "progress") {
      progressBar.value = event.data.percent;
      return;
    }
    
    if (event.data.success) {
      progressBar.value = 100;
      statusEl.textContent = `ประมวลผลสำเร็จ! พบ ${event.data.result.length} รายการ`;
      console.log("ผลลัพธ์:", event.data.result);
    } else {
      statusEl.textContent = `เกิดข้อผิดพลาด: ${event.data.error}`;
    }
  };
});
```

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: เปรียบเทียบประสิทธิภาพ
สร้างโปรแกรมที่คำนวณ Fibonacci(40) ทั้งแบบใช้ Worker และไม่ใช้ แล้ววัดเวลา

```javascript
// ตัวอย่างเฉลย

// fib-worker.js
self.onmessage = (event) => {
  const n = event.data;
  
  function fib(n) {
    if (n <= 1) return n;
    return fib(n - 1) + fib(n - 2);
  }
  
  const start = performance.now();
  const result = fib(n);
  const time = performance.now() - start;
  
  self.postMessage({ result, time });
};

// main.js
function fibWithoutWorker(n) {
  if (n <= 1) return n;
  return fibWithoutWorker(n - 1) + fibWithoutWorker(n - 2);
}

// แบบไม่ใช้ Worker (blocking)
console.time("no-worker");
const result1 = fibWithoutWorker(40);
console.timeEnd("no-worker");
console.log("Fib(40) =", result1, "UI blocked during calculation");

// แบบใช้ Worker (non-blocking)
const worker = new Worker("fib-worker.js");
const start = performance.now();

worker.postMessage(40);
console.log("Worker started... UI สามารถโต้ตอบได้ระหว่างนี้");

worker.onmessage = (event) => {
  const end = performance.now();
  console.log(`Worker Fib(40) = ${event.data.result}`);
  console.log(`Worker time: ${event.data.time.toFixed(2)}ms`);
  console.log(`Total time including overhead: ${(end - start).toFixed(2)}ms`);
};
```

### แบบฝึกหัดที่ 2: Worker Pool สำหรับประมวลผลรูปภาพ
สร้าง Worker Pool ที่รับรูปภาพหลายภาพและทำ grayscale filter พร้อมกัน

### แบบฝึกหัดที่ 3: Real-time Data Processing
สร้างระบบที่รับ data stream จาก WebSocket และประมวลผลใน Worker

```javascript
// websocket-worker.js
let ws = null;

self.onmessage = (event) => {
  if (event.data.command === "connect") {
    ws = new WebSocket(event.data.url);
    
    ws.onmessage = (wsEvent) => {
      try {
        const data = JSON.parse(wsEvent.data);
        const processed = processData(data);
        self.postMessage({ type: "data", payload: processed });
      } catch (e) {
        self.postMessage({ type: "error", message: e.message });
      }
    };
    
    ws.onopen = () => self.postMessage({ type: "connected" });
    ws.onclose = () => self.postMessage({ type: "disconnected" });
    ws.onerror = (e) => self.postMessage({ type: "error", message: "WebSocket error" });
    
  } else if (event.data.command === "disconnect") {
    ws?.close();
  } else if (event.data.command === "send") {
    ws?.send(JSON.stringify(event.data.payload));
  }
};

function processData(data) {
  // ทำ complex data transformation
  return {
    ...data,
    processed: true,
    timestamp: Date.now(),
    summary: computeSummary(data)
  };
}

function computeSummary(data) {
  if (Array.isArray(data.values)) {
    const sum = data.values.reduce((a, b) => a + b, 0);
    return { sum, avg: sum / data.values.length, count: data.values.length };
  }
  return {};
}
```

---

## สรุป Part 46

Web Workers เป็นเครื่องมือสำคัญสำหรับการทำให้เว็บแอปพลิเคชันทำงานได้อย่างมีประสิทธิภาพ:

1. **Dedicated Workers** - ใช้งานง่ายที่สุด เหมาะสำหรับงานหนักที่ต้องการ background processing
2. **Shared Workers** - ใช้ร่วมกันได้หลาย tabs เหมาะสำหรับ shared state
3. **Service Workers** - ทำงานเป็น network proxy เหมาะสำหรับ offline support
4. **WorkerPool Pattern** - จัดการ Workers หลายตัวอย่างมีประสิทธิภาพ
5. **Transferable Objects** - ส่งข้อมูลขนาดใหญ่โดยไม่ต้อง copy

**ควรใช้ Web Worker เมื่อ:**
- การคำนวณใช้เวลา > 50ms
- ประมวลผลข้อมูลขนาดใหญ่
- การประมวลผลรูปภาพหรือวิดีโอ
- Machine Learning inference
- การเข้ารหัสหรือ compression
