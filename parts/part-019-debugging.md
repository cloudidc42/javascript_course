# Part 19: Debugging เบื้องต้น
## Steps 351-370

---

## บทนำ

Debugging คือกระบวนการค้นหาและแก้ไขข้อผิดพลาด (bugs) ในโค้ด ทักษะการ debug ที่ดีเป็นสิ่งจำเป็นสำหรับนักพัฒนาทุกคน ในบทนี้เราจะเรียนรู้เครื่องมือและเทคนิคต่างๆ สำหรับการ debug JavaScript

---

## Step 351: ประเภทของ Bugs และ Errors

```javascript
// 1. Syntax Error - เกิดระหว่าง parsing
// โค้ดจะไม่ทำงานเลย
// function broken( {  // SyntaxError: Unexpected token
//   console.log("never runs");
// }

// 2. Runtime Error - เกิดขึ้นระหว่าง execution
function runtimeError() {
  const obj = null;
  return obj.property; // TypeError: Cannot read properties of null
}

try {
  runtimeError();
} catch (e) {
  console.log(e instanceof TypeError); // true
  console.log(e.message); // "Cannot read properties of null"
}

// 3. Logic Error - โค้ดทำงานแต่ผลลัพธ์ผิด
function add(a, b) {
  return a - b; // Logic error: should be a + b
}
console.log(add(5, 3)); // 2 (ผิด! ควรเป็น 8)

// 4. Type Errors ที่พบบ่อย
// Cannot read properties of undefined/null
// is not a function
// is not defined
// Unexpected token

// ตัวอย่าง errors ที่พบบ่อย
const errors = {
  typeError: () => {
    null.toString();    // TypeError
  },
  referenceError: () => {
    console.log(x);    // ReferenceError: x is not defined
  },
  rangeError: () => {
    new Array(-1);     // RangeError: Invalid array length
  },
  syntaxError: () => {
    eval('invalid syntax {{{'); // SyntaxError
  }
};

// Error object properties
try {
  undefined.property;
} catch (error) {
  console.log(error.name);    // "TypeError"
  console.log(error.message); // "Cannot read properties..."
  console.log(error.stack);   // Stack trace
}

// Custom errors
class ValidationError extends Error {
  constructor(message, field) {
    super(message);
    this.name = 'ValidationError';
    this.field = field;
  }
}

try {
  throw new ValidationError('Invalid email format', 'email');
} catch (e) {
  if (e instanceof ValidationError) {
    console.log(`Validation failed on field: ${e.field}`);
    console.log(`Message: ${e.message}`);
  }
}
```

---

## Step 352: Browser Developer Tools Overview

```javascript
// DevTools เปิดด้วย:
// - F12
// - Ctrl+Shift+I (Windows/Linux)
// - Cmd+Option+I (Mac)
// - คลิกขวา > "Inspect"

// แต่ละ Panel ใน DevTools:

// 1. Elements - ดู/แก้ HTML และ CSS
// 2. Console - รัน JavaScript, ดู logs
// 3. Sources - ดู/แก้ JS code, set breakpoints
// 4. Network - ดู HTTP requests
// 5. Performance - profile performance
// 6. Memory - analyze memory usage
// 7. Application - Storage, Cookies, etc.
// 8. Security - SSL, content security
// 9. Lighthouse - audit performance/accessibility

// ในโค้ด - ตรวจสอบว่าอยู่ใน browser หรือ Node.js
const isBrowser = typeof window !== 'undefined';
const isNode = typeof process !== 'undefined' && process.versions.node;

console.log('Running in:', isBrowser ? 'Browser' : isNode ? 'Node.js' : 'Unknown');
```

---

## Step 353: Console Methods

```javascript
// console.log - แสดงข้อมูลทั่วไป
console.log("Hello, World!");
console.log(42);
console.log({ name: "สมชาย", age: 25 });
console.log([1, 2, 3]);
console.log("User:", { id: 1, name: "test" }); // หลาย arguments

// console.warn - คำเตือน (สีเหลือง)
console.warn("คำเตือน: ฟีเจอร์นี้จะถูกลบในเวอร์ชันถัดไป");
console.warn("Deprecated:", "oldFunction()");

// console.error - แสดง error (สีแดง)
console.error("เกิดข้อผิดพลาด!");
console.error(new Error("Custom error message"));

// console.info - ข้อมูล (สีน้ำเงิน บาง browsers)
console.info("ข้อมูล: แอพเริ่มต้นแล้ว");

// console.debug - debug info
console.debug("Debug info:", { state: "loading", progress: 50 });
```

### console.table

```javascript
// console.table - แสดงข้อมูลแบบตาราง
const users = [
  { id: 1, name: "สมชาย", age: 25, role: "admin" },
  { id: 2, name: "สมหญิง", age: 30, role: "user" },
  { id: 3, name: "สมศักดิ์", age: 28, role: "user" }
];

console.table(users);
// แสดงตาราง: index | id | name | age | role

// ระบุ columns ที่ต้องการ
console.table(users, ["name", "role"]);
// แสดงเฉพาะ name และ role

// Objects
const stats = {
  visitors: 1250,
  pageViews: 5000,
  bounceRate: "40%",
  avgSession: "2:30"
};
console.table(stats);
```

### console.group

```javascript
// console.group/groupEnd - จัดกลุ่ม logs
console.group("User Registration");
  console.log("Validating input...");
  console.log("Creating account...");
  console.group("Email Settings");
    console.log("Sending welcome email...");
    console.log("Email sent successfully");
  console.groupEnd();
  console.log("Registration complete!");
console.groupEnd();

// console.groupCollapsed - collapsed by default
console.groupCollapsed("Debug Info (collapsed)");
  console.log("Internal state data");
  console.log("Performance metrics");
console.groupEnd();

// ใช้ใน functions
function debugFunction(name, args, result) {
  console.group(`Function: ${name}`);
    console.log("Arguments:", args);
    console.log("Result:", result);
    console.log("Time:", new Date().toISOString());
  console.groupEnd();
}
```

### console.time

```javascript
// console.time/timeEnd - วัดเวลา
console.time("fetchData");
// ... some operation
setTimeout(() => {
  console.timeEnd("fetchData"); // "fetchData: 1002ms"
}, 1000);

// console.timeLog - log intermediate times
console.time("process");
// step 1
console.timeLog("process", "After step 1");
// step 2
console.timeLog("process", "After step 2");
// done
console.timeEnd("process");

// วัดเวลา function
function measureTime(fn, label = 'execution') {
  console.time(label);
  const result = fn();
  console.timeEnd(label);
  return result;
}

const sorted = measureTime(
  () => [3, 1, 4, 1, 5, 9].sort((a, b) => a - b),
  'sort'
);
```

### console.count และ console.assert

```javascript
// console.count - นับจำนวนครั้งที่เรียก
function processItem(type) {
  console.count(type);
  // process...
}

processItem("error");   // error: 1
processItem("warning"); // warning: 1
processItem("error");   // error: 2
processItem("error");   // error: 3
console.countReset("error"); // reset counter
processItem("error");   // error: 1

// console.assert - แสดง error ถ้า condition เป็น false
console.assert(1 === 1, "1 should equal 1");       // ไม่แสดงอะไร
console.assert(1 === 2, "1 should not equal 2");   // แสดง error!
console.assert(Array.isArray([]), "Should be array"); // ไม่แสดง
console.assert(Array.isArray({}), "Should be array", {}); // แสดง error + data

// ใช้ใน testing
function divide(a, b) {
  console.assert(b !== 0, "Divisor cannot be zero", { a, b });
  return b !== 0 ? a / b : null;
}

divide(10, 2);  // ปกติ
divide(10, 0);  // แสดง assertion error

// console.trace - แสดง call stack
function outer() {
  inner();
}

function inner() {
  console.trace("Trace from inner");
}

outer();
// แสดง: Trace from inner
// inner @ script.js:2
// outer @ script.js:5
// ... 
```

---

## Step 354: ใช้ Breakpoints ใน DevTools

```javascript
// Breakpoints ทำให้หยุด execution ณ จุดที่ต้องการ

// 1. Line breakpoint - คลิกที่เลขบรรทัดใน Sources panel
function calculateTotal(items) {
  let total = 0;           // ← set breakpoint ที่บรรทัดนี้
  for (const item of items) {
    total += item.price;   // ← หรือบรรทัดนี้
  }
  return total;
}

// 2. debugger statement - เหมือน breakpoint แต่ใส่ใน code
function problematicFunction(data) {
  debugger; // หยุดที่นี่เมื่อ DevTools เปิดอยู่
  const result = data.map(x => x * 2);
  return result;
}

// 3. Conditional breakpoint
// คลิกขวาที่เลขบรรทัด > "Add conditional breakpoint"
// ใส่ condition: item.price > 1000

// 4. Logpoint - แทน console.log
// คลิกขวาที่เลขบรรทัด > "Add logpoint"
// ใส่ message: "item = {item.name}, price = {item.price}"

// ตัวอย่างการใช้ debugger ในสถานการณ์จริง
function processOrders(orders) {
  const results = [];
  
  for (const order of orders) {
    if (order.total > 10000) {
      debugger; // หยุดเมื่อมี order ขนาดใหญ่
    }
    results.push({
      id: order.id,
      processed: true,
      discounted: order.total * 0.9
    });
  }
  
  return results;
}

// หมายเหตุ: ลบ debugger ออกก่อน deploy production!
// ใช้ eslint rule no-debugger เพื่อป้องกัน
```

---

## Step 355: Step Controls

```javascript
// เมื่อ pause ที่ breakpoint มี controls:

// 1. Continue (F8 / Ctrl+/) - ทำงานต่อจนถึง breakpoint ถัดไป

// 2. Step Over (F10) - ทำงาน statement ปัจจุบัน, ไม่เข้า function
function stepOverExample() {
  const a = 5;
  const b = getValue(); // F10: ทำงาน getValue() ทั้งหมดแล้วข้ามไป
  const c = a + b;
}

function getValue() {
  return 10; // ไม่เข้ามาถ้า Step Over
}

// 3. Step Into (F11) - เข้าไปใน function ที่กำลัง call
function stepIntoExample() {
  const result = complexCalc(); // F11: เข้าไปใน complexCalc
}

function complexCalc() {
  // cursor มาที่นี่เมื่อ Step Into
  let x = 10;
  return x * 2;
}

// 4. Step Out (Shift+F11) - ทำงานจนออกจาก function ปัจจุบัน
function currentFunction() {
  for (let i = 0; i < 100; i++) {
    // Shift+F11: ออกจาก loop/function ทั้งหมด
    process(i);
  }
}

// Best practices:
// - ใช้ Step Over เมื่อแน่ใจว่า function ที่เรียกทำงานถูกต้อง
// - ใช้ Step Into เมื่อต้องการตรวจสอบภายใน function
// - ใช้ Step Out เมื่อเข้ามาผิด function
// - ใช้ Continue เมื่อต้องการข้ามไปยังจุดที่สนใจ
```

---

## Step 356: Watch Expressions และ Scope Panel

```javascript
// Watch Expressions - ติดตามค่าของ expression ตลอดเวลา

// ใน Sources panel > Watch panel
// คลิก + แล้วพิมพ์ expression ที่ต้องการ watch เช่น:
// - user.email
// - items.length
// - total * 1.07
// - typeof data
// - JSON.stringify(state)

// ตัวอย่างที่ควร watch:
function shoppingCart() {
  let items = [];
  let total = 0;  // watch: total
  
  function addItem(item) {
    items.push(item);
    total += item.price; // watch: items.length, total
    // จาก Watch panel จะเห็นค่า update ทุก step
  }
  
  return { addItem, getTotal: () => total };
}

// Scope Panel - ดู variables ใน scope ปัจจุบัน
// แสดง:
// - Local: variables ใน function ปัจจุบัน
// - Closure: variables จาก outer scope
// - Global: window object

function outer() {
  const outerVar = "I'm outer"; // จะเห็นใน Scope > Closure
  
  function inner() {
    const innerVar = "I'm inner"; // จะเห็นใน Scope > Local
    // outerVar จะเห็นใน Scope > Closure
    return innerVar + outerVar;
  }
  
  return inner;
}

// ตัวอย่างการดู closure
function makeCounter(initial = 0) {
  let count = initial; // จะเห็นใน Scope > Closure เมื่อ debug increment
  
  return {
    increment() {
      count++;           // debug ที่นี่
      return count;
    },
    decrement() {
      count--;
      return count;
    },
    value() {
      return count;
    }
  };
}
```

---

## Step 357: Call Stack Panel

```javascript
// Call Stack แสดงลำดับการเรียก functions

function a() {
  b();  // Call Stack จะแสดง: a > b > c เมื่อ debug ใน c
}

function b() {
  c();
}

function c() {
  debugger; // Call Stack: c -> b -> a -> anonymous
  console.log("Inside c");
}

a();

// Call Stack ช่วยเข้าใจว่า:
// - function ไหนเรียก function ปัจจุบัน
// - ลำดับการเรียก functions
// - State ของ variables ในแต่ละ frame

// ตัวอย่างจริง - recursive function
function factorial(n) {
  if (n <= 1) {
    debugger; // ดู call stack ที่ deepest point
    return 1;
  }
  return n * factorial(n - 1);
}

factorial(5);
// Call Stack เมื่อ pause:
// factorial(1) <- current
// factorial(2)
// factorial(3)
// factorial(4)
// factorial(5)
// anonymous

// Reading error stack traces
function outer() {
  inner();
}

function inner() {
  throw new Error("Something went wrong");
}

try {
  outer();
} catch (e) {
  // e.stack จะแสดง:
  // Error: Something went wrong
  //   at inner (script.js:2:9)
  //   at outer (script.js:6:3)
  //   at script.js:9:1
  
  console.log(e.stack);
  
  // Parse stack trace
  const frames = e.stack.split('\n')
    .slice(1)
    .map(line => line.trim())
    .filter(line => line.startsWith('at '));
  
  console.log('Call chain:');
  frames.forEach(frame => console.log(' -', frame));
}
```

---

## Step 358: Network Tab สำหรับ Debugging

```javascript
// Network tab ช่วย debug HTTP requests/responses

// ข้อมูลที่เห็นใน Network tab:
// - Name: URL/filename
// - Status: HTTP status code
// - Type: response type (json, html, css, js...)
// - Initiator: ส่วนไหนที่ request
// - Size: ขนาด response
// - Time: เวลาที่ใช้

// ตัวอย่าง fetch ที่อาจมีปัญหา
async function fetchUserData(userId) {
  try {
    const response = await fetch(`/api/users/${userId}`);
    
    // Network tab จะแสดง:
    // - URL: /api/users/123
    // - Method: GET
    // - Status: 200 (สีเขียว) หรือ 404/500 (สีแดง)
    
    if (!response.ok) {
      throw new Error(`HTTP Error: ${response.status} ${response.statusText}`);
    }
    
    const data = await response.json();
    return data;
    
  } catch (error) {
    console.error('Fetch failed:', error);
    
    // ดูใน Network tab:
    // - Headers tab: ดู request/response headers
    // - Response tab: ดู response body
    // - Preview tab: ดู JSON preview
    // - Timing tab: ดูเวลาแต่ละส่วน (DNS, TCP, etc.)
    throw error;
  }
}

// การ filter Network requests
// - Filter by type: XHR/Fetch, JS, CSS, Img, etc.
// - Search by URL
// - Show only slow requests (> 1s)
// - Show only failed requests (สีแดง)

// Debugging CORS errors
async function corsExample() {
  try {
    const response = await fetch('https://different-origin.com/api/data');
    // ถ้าเกิด CORS error จะเห็น:
    // Status: CORS error (สีแดง)
    // Console: Access to fetch... has been blocked by CORS policy
  } catch (e) {
    console.error('CORS Error:', e.message);
  }
}

// Simulating Network Conditions
// DevTools > Network > เลือก "Throttling preset"
// - Fast 3G, Slow 3G, Offline
// ทดสอบว่าแอพทำงานได้บน network ช้าๆ

// ดู Request Headers ที่ส่งไป
async function debugHeaders() {
  const response = await fetch('/api/data', {
    headers: {
      'Authorization': 'Bearer token123',
      'Content-Type': 'application/json',
      'X-Custom-Header': 'value'
    }
  });
  // ดู Headers ใน Network tab > Headers > Request Headers
}
```

---

## Step 359: Performance Profiling

```javascript
// Performance tab - ดูว่าโค้ดใช้เวลาตรงไหนมากที่สุด

// 1. Recording performance
// - กด Record button
// - ทำ action ที่ต้องการ profile
// - กด Stop
// - ดูผลใน Flame chart

// 2. ใช้ Performance API ใน code
// Navigation Timing
const navEntry = performance.getEntriesByType('navigation')[0];
if (navEntry) {
  console.log('Page Load Time:', navEntry.loadEventEnd - navEntry.startTime, 'ms');
  console.log('DOM Interactive:', navEntry.domInteractive - navEntry.startTime, 'ms');
  console.log('First Byte:', navEntry.responseStart - navEntry.requestStart, 'ms');
}

// PerformanceObserver
const observer = new PerformanceObserver((list) => {
  for (const entry of list.getEntries()) {
    if (entry.entryType === 'largest-contentful-paint') {
      console.log('LCP:', entry.startTime, 'ms');
    }
    if (entry.entryType === 'first-input') {
      console.log('FID:', entry.processingStart - entry.startTime, 'ms');
    }
  }
});

observer.observe({ entryTypes: ['largest-contentful-paint', 'first-input'] });

// 3. User Timing API - วัดเวลา custom
performance.mark('processStart');
// ... expensive operation
performance.mark('processEnd');
performance.measure('processTime', 'processStart', 'processEnd');

const measures = performance.getEntriesByName('processTime');
console.log('Process time:', measures[0].duration, 'ms');

// 4. ตัวอย่าง profiling code
function profiledFunction(data) {
  performance.mark('sort-start');
  const sorted = [...data].sort((a, b) => a - b);
  performance.mark('sort-end');
  
  performance.mark('filter-start');
  const filtered = sorted.filter(x => x > 50);
  performance.mark('filter-end');
  
  performance.measure('sort-time', 'sort-start', 'sort-end');
  performance.measure('filter-time', 'filter-start', 'filter-end');
  
  const sortTime = performance.getEntriesByName('sort-time')[0].duration;
  const filterTime = performance.getEntriesByName('filter-time')[0].duration;
  
  console.table({ sortTime, filterTime });
  
  return filtered;
}

// ทดสอบกับข้อมูลใหญ่
const largeArray = Array.from({ length: 100000 }, () => Math.random() * 100);
profiledFunction(largeArray);
```

---

## Step 360: Memory Debugging

```javascript
// Memory leaks ทำให้แอพช้าลงและ crash

// สาเหตุของ Memory Leaks ที่พบบ่อย:

// 1. Event listeners ที่ไม่ถูก remove
class LeakyComponent {
  mount() {
    // Memory Leak! ถ้าไม่ remove listener
    window.addEventListener('scroll', this.handleScroll);
    document.addEventListener('click', this.handleClick);
  }
  
  handleScroll() { /* ... */ }
  handleClick() { /* ... */ }
  
  // ✗ ลืม unmount!
}

class GoodComponent {
  mount() {
    this.scrollHandler = this.handleScroll.bind(this);
    this.clickHandler = this.handleClick.bind(this);
    
    window.addEventListener('scroll', this.scrollHandler);
    document.addEventListener('click', this.clickHandler);
  }
  
  unmount() {
    // ✓ Remove listeners เมื่อไม่ใช้แล้ว
    window.removeEventListener('scroll', this.scrollHandler);
    document.removeEventListener('click', this.clickHandler);
  }
  
  handleScroll() { /* ... */ }
  handleClick() { /* ... */ }
}

// 2. Closures ที่ hold references
function createBigDataProcessor() {
  const bigData = new Array(1000000).fill('data'); // 1M items
  
  return {
    // ✗ closure holds reference to bigData
    process() {
      return bigData.length; 
    }
  };
}

function betterDataProcessor() {
  const bigData = new Array(1000000).fill('data');
  const result = bigData.length; // extract only what we need
  
  // bigData can be GC'd after this
  return {
    process() {
      return result; // ✓ only holds the number, not the array
    }
  };
}

// 3. Growing data structures
const cache = new Map();

function getCachedData(key) {
  if (!cache.has(key)) {
    cache.set(key, fetchData(key)); // ✗ cache ไม่มี limit
  }
  return cache.get(key);
}

// ✓ LRU Cache แทน
class LRUCache {
  constructor(maxSize = 100) {
    this.maxSize = maxSize;
    this.cache = new Map();
  }
  
  get(key) {
    if (this.cache.has(key)) {
      const value = this.cache.get(key);
      // Move to end (most recently used)
      this.cache.delete(key);
      this.cache.set(key, value);
      return value;
    }
    return undefined;
  }
  
  set(key, value) {
    if (this.cache.has(key)) {
      this.cache.delete(key);
    } else if (this.cache.size >= this.maxSize) {
      // Delete oldest (first item)
      this.cache.delete(this.cache.keys().next().value);
    }
    this.cache.set(key, value);
  }
}

// 4. Detached DOM nodes
function detachedNodeLeak() {
  const div = document.createElement('div');
  document.body.appendChild(div);
  
  let children = [];
  children.push(div); // ✗ keeps reference even after removal
  
  document.body.removeChild(div);
  // div is removed from DOM but still in children array!
}

// ✓ ใช้ WeakRef หรือ WeakMap
const elementRefs = new WeakMap();
function trackElement(element) {
  elementRefs.set(element, { created: Date.now() });
  // ถ้า element ถูก GC, WeakMap entry จะถูกลบออกด้วย
}
```

---

## Step 361: debugger Statement

```javascript
// debugger statement - หยุด execution เมื่อ DevTools เปิดอยู่
// ถ้า DevTools ปิด - ไม่มีผลอะไร

// ใช้ใน function ที่ต้องการ debug
function suspiciousFunction(input) {
  debugger; // execution หยุดที่นี่
  
  const result = input * 2;
  
  if (result > 100) {
    debugger; // หยุดอีกครั้งถ้า result > 100
  }
  
  return result;
}

// Conditional debugger
function debugIf(condition) {
  if (condition) debugger;
  return condition;
}

// ใน production - ต้องแน่ใจว่าลบออก
// ใช้ eslint rule: "no-debugger": "error"

// Alternative: debug flag
const DEBUG = process.env.NODE_ENV === 'development';

function debugStep(label, data) {
  if (DEBUG) {
    console.log(`[DEBUG] ${label}:`, data);
    // debugger; // uncomment เมื่อต้องการ
  }
}

// ตัวอย่างการใช้งาน
function processPayment(amount, currency) {
  debugStep("Input", { amount, currency });
  
  const converted = amount * getRate(currency);
  debugStep("After conversion", { converted });
  
  const fee = calculateFee(converted);
  debugStep("Fee", { fee });
  
  return { amount: converted, fee, total: converted + fee };
}

function getRate(currency) {
  const rates = { USD: 35, EUR: 38, GBP: 44 };
  return rates[currency] || 1;
}

function calculateFee(amount) {
  return amount * 0.015; // 1.5%
}
```

---

## Step 362: Source Maps

```javascript
// Source Maps ทำให้ debug minified/transpiled code ได้

// ปกติ minified code อ่านยาก:
// function a(b,c){return b+c}var x=a(1,2);
// 
// Source Map เชื่อม minified code กับ original code
// เพื่อให้ DevTools แสดง original code เมื่อ debug

// การสร้าง Source Map:

// Webpack config:
// devtool: 'source-map'  // ดีที่สุดสำหรับ development
// devtool: 'eval-source-map'  // เร็วกว่า ใช้ใน dev
// devtool: 'hidden-source-map'  // ไม่ expose ใน browser

// TypeScript:
// "sourceMap": true ใน tsconfig.json

// Source Map comment ใน file:
// //# sourceMappingURL=bundle.js.map

// Source Map format:
const sourceMapExample = {
  version: 3,
  sources: ["src/app.js", "src/utils.js"],
  names: ["processData", "formatData", "result"],
  mappings: "AAAA,SAAS...",  // encoded mappings
  sourceRoot: ""
};

// อ่าน Source Map ใน DevTools:
// Sources panel > เปิด minified file
// คลิก {} (Format) เพื่อ pretty print
// หรือดู original source ถ้ามี source map

// inline source maps (สำหรับ debugging เท่านั้น)
// //# sourceMappingURL=data:application/json;base64,...

// ตรวจสอบว่า file มี source map หรือไม่
function hasSourceMap(jsUrl) {
  // ตรวจสอบ comment ท้าย file
  return fetch(jsUrl)
    .then(r => r.text())
    .then(code => {
      const lastLine = code.split('\n').slice(-2).join('\n');
      return /sourceMappingURL/.test(lastLine);
    });
}
```

---

## Step 363: Debugging Asynchronous Code

```javascript
// Async code ยาก debug กว่า sync code

// 1. Async/Await ง่ายกว่า Promise chains
// ยาก debug:
fetch('/api/user')
  .then(r => r.json())
  .then(user => fetch(`/api/posts?userId=${user.id}`))
  .then(r => r.json())
  .then(posts => console.log(posts))
  .catch(e => console.error(e));

// ง่ายกว่า:
async function getUserPosts() {
  try {
    const userResponse = await fetch('/api/user');
    const user = await userResponse.json();       // ← breakpoint ที่นี่
    
    const postsResponse = await fetch(`/api/posts?userId=${user.id}`);
    const posts = await postsResponse.json();     // ← หรือที่นี่
    
    return posts;
  } catch (error) {
    console.error('Error:', error);
    throw error;
  }
}

// 2. async_hooks (Node.js) สำหรับ track async operations
// 3. Async stack traces ใน DevTools
// Enable: Settings > Experiments > Capture async stack traces

// 4. Promise debugging
function debugPromise(promise, name) {
  return promise
    .then(value => {
      console.log(`[Promise:${name}] Resolved:`, value);
      return value;
    })
    .catch(error => {
      console.error(`[Promise:${name}] Rejected:`, error);
      throw error;
    });
}

const userPromise = debugPromise(fetch('/api/user').then(r => r.json()), 'getUser');

// 5. Track pending promises
const pendingPromises = new Set();

function trackPromise(promise, name) {
  const tracked = { name, start: Date.now() };
  pendingPromises.add(tracked);
  
  return promise.finally(() => {
    pendingPromises.delete(tracked);
    console.log(`Promise "${name}" resolved in ${Date.now() - tracked.start}ms`);
  });
}

// 6. Timeout สำหรับ stuck promises
function withTimeout(promise, ms, message = 'Promise timed out') {
  const timeout = new Promise((_, reject) => {
    setTimeout(() => reject(new Error(message)), ms);
  });
  return Promise.race([promise, timeout]);
}

// ใช้งาน
async function loadDataWithTimeout() {
  try {
    const data = await withTimeout(
      fetch('/api/slow-endpoint').then(r => r.json()),
      5000,
      'API request timed out after 5 seconds'
    );
    return data;
  } catch (e) {
    console.error('Failed:', e.message);
    return null;
  }
}
```

---

## Step 364: Debugging Strategies

```javascript
// กลยุทธ์สำหรับการ debug อย่างมีประสิทธิภาพ

// 1. Reproduce the bug consistently
// หา minimal test case:
function findMinimalBug() {
  // ลบส่วนที่ไม่เกี่ยวข้องออกจนเหลือแค่ส่วนที่ reproduce bug ได้
}

// 2. Binary Search debugging
function binarySearchDebug(items) {
  // แทนที่จะ debug ทีละ statement:
  // - ตรวจสอบ halfway point ก่อน
  // - ถ้า OK แสดงว่า bug อยู่ครึ่งหลัง
  // - ถ้าไม่ OK bug อยู่ครึ่งแรก
  // - ทำซ้ำจนเจอ
  
  const midpoint = Math.floor(items.length / 2);
  const firstHalf = items.slice(0, midpoint);
  console.log("First half result:", processItems(firstHalf));
  // ถ้าผลผิด: bug อยู่ใน processItems หรือ first half data
  // ถ้าผลถูก: bug อยู่ใน second half
}

// 3. Print state at key points
function debugState(label, state) {
  console.group(`State: ${label}`);
  console.log('Timestamp:', new Date().toISOString());
  console.log('State:', JSON.stringify(state, null, 2));
  console.groupEnd();
}

// 4. Rubber Duck Debugging
// อธิบายโค้ดทีละบรรทัดให้ตัวเองฟัง
// มักจะเจอ bug เอง

// 5. Version control bisect
// git bisect start
// git bisect bad  (current commit has bug)
// git bisect good <commit>  (known good commit)
// git ทำ binary search หา commit ที่ทำให้เกิด bug

// 6. การใช้ console.log อย่างมีประสิทธิภาพ
// ✗ ไม่ดี
console.log("here");
console.log("value");
console.log(x);

// ✓ ดีกว่า
console.log("[auth.login] Starting login process for:", username);
console.log("[auth.login] Response status:", response.status);
console.log("[auth.login] Parsed user:", { id: user.id, role: user.role });

// 7. สร้าง debug helper
class Logger {
  constructor(namespace) {
    this.namespace = namespace;
    this.enabled = localStorage.getItem(`debug:${namespace}`) === 'true';
  }
  
  enable() {
    localStorage.setItem(`debug:${this.namespace}`, 'true');
    this.enabled = true;
  }
  
  disable() {
    localStorage.removeItem(`debug:${this.namespace}`);
    this.enabled = false;
  }
  
  log(...args) {
    if (this.enabled) console.log(`[${this.namespace}]`, ...args);
  }
  
  warn(...args) {
    if (this.enabled) console.warn(`[${this.namespace}]`, ...args);
  }
  
  error(...args) {
    // errors always log regardless of enabled state
    console.error(`[${this.namespace}]`, ...args);
  }
  
  time(label) {
    if (this.enabled) console.time(`[${this.namespace}] ${label}`);
  }
  
  timeEnd(label) {
    if (this.enabled) console.timeEnd(`[${this.namespace}] ${label}`);
  }
}

// ใช้งาน
const authLogger = new Logger('auth');
// authLogger.enable(); // เปิดใน console เพื่อ debug

authLogger.log('Login attempt:', username);
authLogger.time('login-request');
// ... login logic
authLogger.timeEnd('login-request');
```

---

## Step 365: Error Boundaries และ Global Error Handling

```javascript
// 1. try-catch ทุกที่ที่เป็นไปได้
function safeOperation(fn, ...args) {
  try {
    return { success: true, data: fn(...args) };
  } catch (error) {
    return { success: false, error };
  }
}

// 2. Global error handlers
// Uncaught synchronous errors
window.addEventListener('error', function(event) {
  console.error('Uncaught Error:', {
    message: event.message,
    filename: event.filename,
    line: event.lineno,
    column: event.colno,
    error: event.error
  });
  
  // ส่งไปยัง error tracking service
  reportError(event.error);
  
  // return true เพื่อป้องกัน default behavior (console.error)
  // return true;
});

// Uncaught Promise rejections
window.addEventListener('unhandledrejection', function(event) {
  console.error('Unhandled Promise Rejection:', {
    reason: event.reason,
    promise: event.promise
  });
  
  reportError(event.reason);
  
  // ป้องกัน browser default
  // event.preventDefault();
});

// Node.js equivalents
// process.on('uncaughtException', (err) => { ... });
// process.on('unhandledRejection', (reason, promise) => { ... });

// 3. Error tracking (simplified Sentry-like)
class ErrorTracker {
  constructor(options = {}) {
    this.errors = [];
    this.maxErrors = options.maxErrors || 100;
    this.onError = options.onError || null;
    
    this.setupGlobalHandlers();
  }
  
  setupGlobalHandlers() {
    window.addEventListener('error', (e) => {
      this.capture(e.error, {
        type: 'uncaught',
        url: e.filename,
        line: e.lineno,
        column: e.colno
      });
    });
    
    window.addEventListener('unhandledrejection', (e) => {
      this.capture(e.reason, { type: 'promise' });
    });
  }
  
  capture(error, context = {}) {
    const entry = {
      id: Date.now(),
      timestamp: new Date().toISOString(),
      name: error?.name || 'Unknown',
      message: error?.message || String(error),
      stack: error?.stack,
      userAgent: navigator.userAgent,
      url: window.location.href,
      context
    };
    
    this.errors.unshift(entry);
    
    // Keep only recent errors
    if (this.errors.length > this.maxErrors) {
      this.errors = this.errors.slice(0, this.maxErrors);
    }
    
    console.error('[ErrorTracker]', entry);
    
    if (this.onError) this.onError(entry);
  }
  
  getErrors() {
    return [...this.errors];
  }
  
  clear() {
    this.errors = [];
  }
}

function reportError(error) {
  // ส่งไปยัง server หรือ service เช่น Sentry
  console.log('Reporting error to server...');
}

const tracker = new ErrorTracker({ onError: reportError });
```

---

## Step 366: การ Debug Common Issues

```javascript
// Bug ที่พบบ่อยและวิธีแก้

// 1. "this" context problems
class EventHandler {
  constructor() {
    this.count = 0;
    
    // ✗ Problem: this ใน callback ไม่ใช่ class instance
    // document.querySelector('button').addEventListener('click', this.handleClick);
    
    // ✓ Solution 1: bind
    document.querySelector('button')?.addEventListener('click', this.handleClick.bind(this));
    
    // ✓ Solution 2: arrow function
    document.querySelector('button')?.addEventListener('click', () => this.handleClick());
    
    // ✓ Solution 3: arrow function method (class field)
    // handleClick = () => { this.count++; }
  }
  
  handleClick() {
    this.count++; // this จะถูกต้องถ้าใช้ bind หรือ arrow
  }
}

// 2. Async/await ใน forEach (ไม่รอ)
const items = [1, 2, 3];

// ✗ Problem: forEach ไม่รอ async operations
async function badForEach() {
  items.forEach(async (item) => {
    await processItem(item); // ไม่รอ!
  });
  console.log("Done?"); // print ก่อน items ถูก process
}

// ✓ Solution: for...of
async function goodForOf() {
  for (const item of items) {
    await processItem(item); // รอจริงๆ
  }
  console.log("Done!"); // print หลัง items ทั้งหมดถูก process
}

// ✓ หรือ Promise.all สำหรับ parallel
async function parallelProcess() {
  await Promise.all(items.map(item => processItem(item)));
  console.log("All done!");
}

// 3. Off-by-one errors
function getLastNItems(arr, n) {
  // ✗ Bug: slice(-n) เมื่อ n > arr.length คืน [] ไม่ใช่ arr
  // return arr.slice(-n);
  
  // ✓ Fix:
  return arr.slice(Math.max(0, arr.length - n));
}

// 4. Mutation vs immutability
const original = [1, 2, 3];

// ✗ Problem: sort() mutates original
const sorted = original.sort((a, b) => b - a); // changes original!
console.log(original); // [3, 2, 1] ← changed!

// ✓ Fix: copy before sorting
const sortedSafe = [...original].sort((a, b) => b - a);
console.log(original); // [1, 2, 3] ← unchanged

// 5. Floating point precision
console.log(0.1 + 0.2 === 0.3); // false!
console.log(0.1 + 0.2);         // 0.30000000000000004

// ✓ Fix:
const epsilon = Number.EPSILON;
function floatEquals(a, b) {
  return Math.abs(a - b) < epsilon;
}
console.log(floatEquals(0.1 + 0.2, 0.3)); // true

// หรือใช้ toFixed สำหรับ display
console.log((0.1 + 0.2).toFixed(2)); // "0.30"

async function processItem(item) {
  return new Promise(resolve => setTimeout(() => resolve(item * 2), 100));
}
```

---

## Step 367: ESLint Concepts

```javascript
// ESLint ช่วยหา bugs และ enforce code style
// ติดตั้ง: npm install -D eslint
// สร้าง config: npx eslint --init

// ตัวอย่าง .eslintrc.json
const eslintConfig = {
  "env": {
    "browser": true,
    "es2021": true,
    "node": true
  },
  "extends": [
    "eslint:recommended"
  ],
  "parserOptions": {
    "ecmaVersion": "latest",
    "sourceType": "module"
  },
  "rules": {
    // Error level: 0=off, 1=warn, 2=error
    "no-console": "warn",           // warn เมื่อใช้ console
    "no-debugger": "error",         // error ถ้าลืมลบ debugger
    "no-unused-vars": "error",      // error ถ้ามี unused variables
    "no-undef": "error",            // error ถ้าใช้ undefined variable
    "eqeqeq": "error",              // ต้องใช้ === ไม่ใช่ ==
    "no-var": "error",              // ต้องใช้ let/const ไม่ใช่ var
    "prefer-const": "warn",         // แนะนำใช้ const เมื่อไม่ reassign
    "semi": ["error", "always"],    // ต้องมี semicolon
    "quotes": ["error", "single"],  // single quotes
    "no-trailing-spaces": "error",
    "no-duplicate-case": "error",
    "no-empty": "warn",
    "no-extra-boolean-cast": "warn",
    "no-unreachable": "error"
  }
};

// Rules ที่ช่วย catch bugs:
// - no-unused-vars: ตัวแปรที่สร้างแต่ไม่ใช้ (อาจเป็น typo)
// - no-undef: ใช้ตัวแปรที่ไม่ได้ declare
// - eqeqeq: ป้องกัน type coercion bugs
// - no-debugger: ป้องกันลืม remove debugger
// - no-unreachable: โค้ดหลัง return/throw ที่ไม่มีทางทำงานได้

// ตัวอย่าง code ที่ ESLint จะ flag:
function lintExamples() {
  var x = 5; // no-var: ควรใช้ let/const
  let unused = 10; // no-unused-vars: ไม่ได้ใช้ unused
  
  if (x == "5") { // eqeqeq: ควรใช้ ===
    console.log("equal"); // no-console: ไม่ควรมี console.log ใน production
  }
  
  return x;
  
  console.log("after return"); // no-unreachable: โค้ดนี้ไม่มีทางทำงาน
}

// ESLint disable comments (ใช้เมื่อจำเป็นจริงๆ)
/* eslint-disable no-console */
console.log("this is intentional"); // OK
/* eslint-enable no-console */

const value = "test"; // eslint-disable-line no-unused-vars
```

---

## Step 368: Debugging Tools และ Extensions

```javascript
// 1. React DevTools
// - ดู component tree
// - ดู/แก้ props และ state
// - profile performance

// 2. Redux DevTools
// - ดู action history
// - Time-travel debugging (ย้อนกลับ state)
// - Export/import state

// 3. Vue DevTools
// - เหมือน React DevTools แต่สำหรับ Vue

// 4. Node.js debugging
// รัน: node --inspect app.js
// รัน: node --inspect-brk app.js (หยุดที่บรรทัดแรก)
// เปิด: chrome://inspect ใน Chrome

// 5. VS Code debugging
// .vscode/launch.json
const vscodeConfig = {
  "version": "0.2.0",
  "configurations": [
    {
      "type": "node",
      "request": "launch",
      "name": "Launch Node.js",
      "program": "${workspaceFolder}/app.js"
    },
    {
      "type": "chrome",
      "request": "launch",
      "name": "Launch Chrome",
      "url": "http://localhost:3000",
      "webRoot": "${workspaceFolder}/src"
    }
  ]
};

// 6. Debugging Environment Variables
const config = {
  isDev: process.env.NODE_ENV === 'development',
  isDebug: process.env.DEBUG === 'true',
  logLevel: process.env.LOG_LEVEL || 'info'
};

// Debug logging กับ levels
const LOG_LEVELS = { debug: 0, info: 1, warn: 2, error: 3 };

class DebugLogger {
  constructor(level = 'info') {
    this.level = LOG_LEVELS[level] || LOG_LEVELS.info;
  }
  
  debug(...args) {
    if (this.level <= 0) console.debug('[DEBUG]', ...args);
  }
  
  info(...args) {
    if (this.level <= 1) console.info('[INFO]', ...args);
  }
  
  warn(...args) {
    if (this.level <= 2) console.warn('[WARN]', ...args);
  }
  
  error(...args) {
    if (this.level <= 3) console.error('[ERROR]', ...args);
  }
}

const logger = new DebugLogger(config.isDev ? 'debug' : 'warn');
logger.debug('Starting up...'); // แสดงแค่ใน development
logger.error('Critical error!'); // แสดงทุก environment
```

---

## Step 369: Testing ในฐานะ Debugging Tool

```javascript
// Unit tests ช่วย prevent และ find bugs

// Test ง่ายๆ ด้วย assert
function assert(condition, message) {
  if (!condition) {
    throw new Error(`Assertion failed: ${message}`);
  }
  return true;
}

function assertEqual(actual, expected, message) {
  const pass = JSON.stringify(actual) === JSON.stringify(expected);
  if (!pass) {
    throw new Error(`${message}\n  Expected: ${JSON.stringify(expected)}\n  Actual: ${JSON.stringify(actual)}`);
  }
  return true;
}

// ฟังก์ชันที่จะทดสอบ
function calculateDiscount(price, percentage) {
  if (price < 0) throw new Error('Price cannot be negative');
  if (percentage < 0 || percentage > 100) throw new Error('Percentage must be 0-100');
  return price * (1 - percentage / 100);
}

// Tests
function runTests() {
  const results = { passed: 0, failed: 0, errors: [] };
  
  const tests = [
    // [description, fn]
    ['10% discount on 100', () => {
      assertEqual(calculateDiscount(100, 10), 90, '10% off 100 should be 90');
    }],
    ['50% discount on 200', () => {
      assertEqual(calculateDiscount(200, 50), 100, '50% off 200 should be 100');
    }],
    ['0% discount unchanged', () => {
      assertEqual(calculateDiscount(100, 0), 100, '0% off should be unchanged');
    }],
    ['100% discount is free', () => {
      assertEqual(calculateDiscount(100, 100), 0, '100% off should be 0');
    }],
    ['Negative price throws', () => {
      let threw = false;
      try { calculateDiscount(-10, 10); } catch { threw = true; }
      assert(threw, 'Should throw on negative price');
    }],
    ['Invalid percentage throws', () => {
      let threw = false;
      try { calculateDiscount(100, 110); } catch { threw = true; }
      assert(threw, 'Should throw on percentage > 100');
    }]
  ];
  
  tests.forEach(([desc, fn]) => {
    try {
      fn();
      results.passed++;
      console.log(`✓ ${desc}`);
    } catch (e) {
      results.failed++;
      results.errors.push({ test: desc, error: e.message });
      console.error(`✗ ${desc}: ${e.message}`);
    }
  });
  
  console.log(`\n${results.passed}/${tests.length} tests passed`);
  return results;
}

runTests();
```

---

## Step 370: Complete Debugging Workflow

```javascript
// ตัวอย่าง workflow การ debug ปัญหาจริง

// สถานการณ์: ฟังก์ชัน calculateMonthlyPayment คืนค่าผิด

// ขั้นที่ 1: ทำซ้ำปัญหา
function calculateMonthlyPayment(principal, annualRate, months) {
  const monthlyRate = annualRate / 1200; // annual % to monthly decimal
  
  if (monthlyRate === 0) {
    return principal / months;
  }
  
  const payment = principal * monthlyRate * 
    Math.pow(1 + monthlyRate, months) / 
    (Math.pow(1 + monthlyRate, months) - 1);
  
  return Math.round(payment * 100) / 100;
}

// ขั้นที่ 2: เพิ่ม debug logging
function calculateMonthlyPaymentDebug(principal, annualRate, months) {
  console.group('calculateMonthlyPayment Debug');
  console.log('Input:', { principal, annualRate, months });
  
  const monthlyRate = annualRate / 1200;
  console.log('Monthly rate:', monthlyRate);
  
  if (monthlyRate === 0) {
    const result = principal / months;
    console.log('Zero rate result:', result);
    console.groupEnd();
    return result;
  }
  
  const powerFactor = Math.pow(1 + monthlyRate, months);
  console.log('Power factor:', powerFactor);
  
  const payment = principal * monthlyRate * powerFactor / (powerFactor - 1);
  console.log('Raw payment:', payment);
  
  const rounded = Math.round(payment * 100) / 100;
  console.log('Final payment:', rounded);
  console.groupEnd();
  
  return rounded;
}

// ขั้นที่ 3: Unit tests
function testCalculator() {
  const tests = [
    // principal, rate, months, expected
    [100000, 5, 12, 8560.75],
    [200000, 6, 24, 8861.36],
    [500000, 0, 12, 41666.67]
  ];
  
  tests.forEach(([principal, rate, months, expected]) => {
    const result = calculateMonthlyPayment(principal, rate, months);
    const diff = Math.abs(result - expected);
    
    if (diff > 0.01) {
      console.error(`FAIL: principal=${principal}, rate=${rate}%, months=${months}`);
      console.error(`  Expected: ${expected}, Got: ${result}, Diff: ${diff}`);
    } else {
      console.log(`PASS: ${principal} at ${rate}% for ${months} months = ${result}`);
    }
  });
}

testCalculator();

// ขั้นที่ 4: Performance profiling
function profileCalculations() {
  const iterations = 10000;
  
  console.time('calculations');
  
  for (let i = 0; i < iterations; i++) {
    calculateMonthlyPayment(
      Math.random() * 1000000,
      Math.random() * 15,
      Math.floor(Math.random() * 360) + 12
    );
  }
  
  console.timeEnd('calculations');
  console.log(`Completed ${iterations} calculations`);
}

profileCalculations();

// ขั้นที่ 5: Error handling ที่ดี
function safeCalculatePayment(principal, annualRate, months) {
  if (typeof principal !== 'number' || principal <= 0) {
    throw new TypeError('Principal must be a positive number');
  }
  if (typeof annualRate !== 'number' || annualRate < 0) {
    throw new TypeError('Annual rate must be a non-negative number');
  }
  if (!Number.isInteger(months) || months <= 0) {
    throw new TypeError('Months must be a positive integer');
  }
  
  return calculateMonthlyPayment(principal, annualRate, months);
}
```

---

## แบบฝึกหัด (Exercises)

### แบบฝึกหัดที่ 1: Console Master
สร้าง debug utility ที่:
- มี log levels (debug, info, warn, error)
- แสดง timestamp
- รองรับ groups และ timers
- enable/disable ผ่าน localStorage

### แบบฝึกหัดที่ 2: Error Boundary
สร้าง function wrapper ที่:
- catch errors และ log รายละเอียด
- retry async operations ถ้า fail
- report errors ไปยัง console ด้วย format ที่ดี

### แบบฝึกหัดที่ 3: Memory Leak Finder
สร้าง script ที่ detect event listener leaks โดย:
- track ว่าเพิ่ม listeners อะไรบ้าง
- warning เมื่อมี listeners เยอะเกินไป
- list listeners ที่ยังคงอยู่

### แบบฝึกหัดที่ 4: Performance Monitor
สร้าง class ที่ measure และ report:
- Function execution times
- Memory usage ก่อน/หลัง
- Call frequency

### แบบฝึกหัดที่ 5: Async Debugger
Debug async function chain ที่ผิดและอธิบายว่า:
- ผิดตรงไหน
- ทำไมถึงผิด
- แก้ไขอย่างไร

---

## สรุป Part 19

ในบทนี้เรียนรู้:
- ประเภทของ bugs: Syntax, Runtime, Logic
- Browser DevTools: Elements, Console, Sources, Network, Performance
- Console methods: log, warn, error, table, group, time, count, assert, trace
- Breakpoints: line, conditional, logpoint
- Step controls: Continue, Step Over, Into, Out
- Watch Expressions และ Scope Panel
- Call Stack
- Network tab debugging
- Performance profiling
- Memory debugging และ memory leaks
- debugger statement
- Source maps
- Async debugging
- Common bugs และ strategies
- ESLint
- Error tracking
