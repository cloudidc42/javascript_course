# Part 93: Performance at Scale (Steps 1831-1850)

## ประสิทธิภาพในระดับ Production - V8 Internals และ Profiling

ในส่วนนี้เราจะเรียนรู้เกี่ยวกับ V8 engine internals, การ profiling, และเทคนิคการ optimize JavaScript ในระดับ production สำหรับระบบที่มีผู้ใช้งานจำนวนมาก

---

## Step 1831: V8 Engine Internals Overview

V8 คือ JavaScript engine ที่ Google พัฒนา ใช้ใน Chrome และ Node.js มันทำให้ JavaScript ทำงานได้เร็วมากด้วย JIT compilation

```javascript
// V8 Pipeline:
// Source Code
//   ↓ Parser
// AST (Abstract Syntax Tree)
//   ↓ Ignition (Interpreter)
// Bytecode
//   ↓ Sparkplug (Baseline Compiler)
// Unoptimized Machine Code
//   ↓ Turbofan (Optimizing Compiler) [เมื่อ code "hot"]
// Optimized Machine Code

// ดูว่า V8 ทำอะไรกับ code ของเรา
node --print-bytecode --print-bytecode-filter=myFunction app.js

// ดู optimized code
node --print-opt-code --print-opt-code-filter=myFunction app.js

// ตรวจสอบ deoptimizations
node --trace-deopt app.js

// ตรวจสอบ inline caches
node --trace-ic app.js
```

```javascript
// ทำความเข้าใจ V8 optimization lifecycle

// 1. Code เริ่มต้นทำงานใน Ignition (interpreter)
// 2. V8 track "hot" functions (เรียกบ่อย)
// 3. Turbofan optimize hot functions
// 4. ถ้า assumptions ผิด -> deoptimize -> กลับไป interpreter

function add(a, b) {
  return a + b;
}

// ครั้งแรก: V8 ไม่รู้ type ของ a และ b
add(1, 2);    // V8 สร้าง feedback: "integers"
add(3, 4);    // ยืนยัน: "integers"
// ... หลายครั้ง -> Turbofan optimize สำหรับ integer addition

add(1.5, 2.5); // ยังโอเค: floating point
add("a", "b"); // DEOPTIMIZE! Type เปลี่ยน -> กลับ interpreter

// หลังจาก deopt, Turbofan ต้องรอให้ hot อีกครั้ง
// เพื่อ compile ใหม่สำหรับ wider types
```

---

## Step 1832: Hidden Classes and Inline Caches

```javascript
// Hidden Classes (Shapes) - V8 track object structure

// V8 สร้าง Hidden Class ตาม property ที่เพิ่ม

// ❌ ผิด - ทำให้ V8 สร้าง hidden classes หลายตัว
function createPoint(x, y) {
  const point = {};
  point.x = x;  // Hidden Class 0 -> HC1 (has x)
  point.y = y;  // HC1 -> HC2 (has x, y)
  return point;
}

// บาง objects เพิ่ม property ต่างลำดับ
function createPointBad(x, y, shouldAddZ) {
  const point = {};
  point.x = x;
  if (shouldAddZ) {
    point.z = 0;  // บาง objects มี z บาง objects ไม่มี -> หลาย hidden classes
  }
  point.y = y;
  return point;
}

// ✅ ถูก - สร้าง hidden class เดียวกัน
function createPoint2D(x, y) {
  return { x, y };  // V8 สร้าง HC เดียวสำหรับทุก {x, y} object
}

// ตัวอย่าง Inline Caches (ICs)
class Product {
  constructor(name, price) {
    this.name = name;
    this.price = price;
  }
}

function getPrice(product) {
  return product.price;  // IC: cache location of 'price' property
}

const p1 = new Product('A', 100);
const p2 = new Product('B', 200);

// ทั้ง p1 และ p2 มี hidden class เดียวกัน
// IC สำหรับ .price ทำงานได้เร็วมาก (monomorphic)
getPrice(p1); // cache miss (ครั้งแรก)
getPrice(p1); // cache hit (fast!)
getPrice(p2); // cache hit (same shape!)

// Polymorphic IC (ช้าลงหน่อย)
function getAnyPrice(obj) {
  return obj.price;
}

getAnyPrice({ price: 100 });      // shape 1
getAnyPrice({ name: 'A', price: 200 }); // shape 2 - polymorphic
getAnyPrice(new Product('B', 300));     // shape 3 - megamorphic!
// Megamorphic = V8 ยอมแพ้ cache ทำให้ช้ามาก
```

---

## Step 1833: Deoptimization

```javascript
// สาเหตุของ Deoptimization และวิธีหลีกเลี่ยง

// 1. Type Changes
function badAdd(a, b) {
  return a + b;
}
// call ด้วย numbers หลายครั้ง -> optimize สำหรับ numbers
// จากนั้น call ด้วย string -> DEOPT

// Fix: ใช้ type เดิมเสมอ
function safeAdd(a, b) {
  // Type guard
  if (typeof a !== 'number' || typeof b !== 'number') {
    return Number(a) + Number(b);
  }
  return a + b;
}

// 2. Arguments Object
function badFunc() {
  const args = arguments;  // ทำให้ deopt
  return Array.from(args);
}

// Fix: ใช้ rest parameters
function goodFunc(...args) {
  return args;
}

// 3. try-catch ใน hot paths
// ❌ V8 ไม่ optimize functions ที่มี try-catch (เดิม, ปัจจุบันดีขึ้นแล้ว)
function hotPath(data) {
  try {
    return processData(data);
  } catch (e) {
    return null;
  }
}

// Fix: แยก try-catch ออกจาก hot path
function hotPathFixed(data) {
  return processData(data);  // แยก error handling ออก
}

function withErrorHandling(data) {
  try {
    return hotPathFixed(data);
  } catch (e) {
    return null;
  }
}

// 4. delete operator
const obj = { a: 1, b: 2 };
delete obj.a;  // เปลี่ยน hidden class -> deopt

// Fix: ตั้งเป็น undefined/null แทน delete
obj.a = undefined;  // ยัง keep shape เดิม
// หรือสร้าง object ใหม่
const newObj = Object.fromEntries(
  Object.entries(obj).filter(([k]) => k !== 'a')
);

// 5. Polymorphic calls
class Dog {
  speak() { return 'Woof'; }
}
class Cat {
  speak() { return 'Meow'; }
}
class Cow {
  speak() { return 'Moo'; }
}

function makeNoise(animal) {
  return animal.speak();  // polymorphic - หลาย implementations
}

// Fix: ถ้า performance critical ให้ใช้ type check
// หรือ monomorphic design
```

---

## Step 1834: Memory Model and Garbage Collection

```javascript
// V8 Memory Model

// Heap ถูกแบ่งเป็น:
// - New Space (Young Generation): objects ใหม่ เล็ก เร็ว
//   - Semi-space 1 และ 2
//   - Minor GC (Scavenger): เร็วมาก
// - Old Space (Old Generation): objects ที่ผ่าน 2 minor GCs
//   - Major GC (Mark-Compact): ช้ากว่า

// ดู V8 heap statistics
const v8 = require('v8');
const heapStats = v8.getHeapStatistics();
console.log('Heap Statistics:');
console.log(`  Total heap size: ${Math.round(heapStats.total_heap_size / 1024 / 1024)} MB`);
console.log(`  Used heap size: ${Math.round(heapStats.used_heap_size / 1024 / 1024)} MB`);
console.log(`  Heap size limit: ${Math.round(heapStats.heap_size_limit / 1024 / 1024)} MB`);
console.log(`  External memory: ${Math.round(heapStats.external_memory / 1024 / 1024)} MB`);

// ดู heap space details
const heapSpaces = v8.getHeapSpaceStatistics();
heapSpaces.forEach(space => {
  console.log(`\n  ${space.space_name}:`);
  console.log(`    Used: ${Math.round(space.space_used_size / 1024)} KB`);
  console.log(`    Available: ${Math.round(space.space_available_size / 1024)} KB`);
});

// Memory leaks - สาเหตุทั่วไป

// 1. Global variables
// ❌ ผิด
function processData() {
  globalData = [];  // ไม่มี let/const/var -> global!
  for (let i = 0; i < 1000000; i++) {
    globalData.push(i);
  }
}

// 2. Event listeners ที่ไม่ถูก remove
class EventEmitterLeak {
  constructor(emitter) {
    // ❌ ผิด - ไม่ remove listener
    emitter.on('data', this.handleData.bind(this));
  }
  handleData(data) { /* ... */ }
}

// ✅ ถูก
class EventEmitterFixed {
  constructor(emitter) {
    this.emitter = emitter;
    this.boundHandler = this.handleData.bind(this);
    emitter.on('data', this.boundHandler);
  }
  handleData(data) { /* ... */ }
  destroy() {
    this.emitter.off('data', this.boundHandler);
  }
}

// 3. Closures ที่ hold references
function createLeak() {
  const hugeData = new Array(1000000).fill(0);
  
  return function() {
    // อ้างถึง hugeData ทำให้มันไม่ถูก GC
    return hugeData[0];
  };
}

// 4. Timers
function setupInterval() {
  const data = new Array(1000).fill(0);
  
  // ❌ ผิด - interval ยัง hold reference ถึง data
  setInterval(() => {
    processData(data);
  }, 1000);
}

// ✅ ถูก
function setupIntervalFixed() {
  const data = new Array(1000).fill(0);
  const intervalId = setInterval(() => {
    processData(data);
  }, 1000);
  
  // cleanup when done
  return () => clearInterval(intervalId);
}
```

---

## Step 1835: Performance-Critical Code Patterns

```javascript
// Pattern 1: Object Pooling - ลดการสร้าง objects
class Vector2D {
  constructor(x = 0, y = 0) {
    this.x = x;
    this.y = y;
  }
  set(x, y) {
    this.x = x;
    this.y = y;
    return this;
  }
}

class VectorPool {
  constructor(initialSize = 100) {
    this._pool = Array.from({ length: initialSize }, () => new Vector2D());
    this._available = initialSize;
  }

  acquire() {
    if (this._available === 0) {
      return new Vector2D(); // fallback
    }
    return this._pool[--this._available];
  }

  release(vector) {
    if (this._available < this._pool.length) {
      this._pool[this._available++] = vector;
    }
  }
}

const pool = new VectorPool(1000);

// ❌ ผิด - สร้าง objects ใหม่ทุกครั้ง
function badCalculation(points) {
  return points.map(p => {
    const v = new Vector2D(p.x * 2, p.y * 2);  // GC pressure
    return v.x + v.y;
  });
}

// ✅ ถูก - ใช้ object pool
function goodCalculation(points) {
  return points.map(p => {
    const v = pool.acquire().set(p.x * 2, p.y * 2);
    const result = v.x + v.y;
    pool.release(v);
    return result;
  });
}

// Pattern 2: Typed Arrays
// ❌ ผิด - regular array สำหรับ numeric data
const regularArray = new Array(1000000).fill(0);

// ✅ ถูก - typed array เร็วกว่ามาก
const typedArray = new Float64Array(1000000);
const intArray = new Int32Array(1000000);
const uint8Array = new Uint8Array(1000000);

// Benchmark
console.time('regular sum');
let sum1 = 0;
for (let i = 0; i < regularArray.length; i++) sum1 += regularArray[i];
console.timeEnd('regular sum');

console.time('typed sum');
let sum2 = 0;
for (let i = 0; i < typedArray.length; i++) sum2 += typedArray[i];
console.timeEnd('typed sum');

// Pattern 3: Avoid property lookup in loops
const items = [{ value: 1 }, { value: 2 }, { value: 3 }];

// ❌ ผิด - property lookup ทุก iteration
function badLoop(arr) {
  let sum = 0;
  for (let i = 0; i < arr.length; i++) {
    sum += arr[i].value;
  }
  return sum;
}

// ✅ ถูก - cache length
function goodLoop(arr) {
  let sum = 0;
  const len = arr.length;  // cache length
  for (let i = 0; i < len; i++) {
    sum += arr[i].value;
  }
  return sum;
}

// Pattern 4: Short-circuit evaluation
function processUser(user) {
  // ถ้า user เป็น null, ข้ามได้เลย
  if (!user || !user.isActive || !user.permissions) return null;
  
  // ใช้ short-circuit ลด branches
  const name = user.displayName || user.username || user.email || 'Unknown';
  
  return { name };
}
```

---

## Step 1836: Avoiding Performance Pitfalls

```javascript
// Pitfall 1: String concatenation ใน loop
// ❌ ผิด - O(n²)
function buildStringBad(items) {
  let result = '';
  for (const item of items) {
    result += item.name + ', ';  // สร้าง string ใหม่ทุกครั้ง
  }
  return result.slice(0, -2);
}

// ✅ ถูก - O(n)
function buildStringGood(items) {
  return items.map(item => item.name).join(', ');
}

// ✅ ดีกว่า - ใช้ Array
function buildStringBetter(items) {
  const parts = [];
  for (const item of items) {
    parts.push(item.name);
  }
  return parts.join(', ');
}

// Pitfall 2: Excessive DOM manipulation
// ❌ ผิด - แต่ละ append ทำให้เกิด reflow
function badDOMUpdate(container, items) {
  items.forEach(item => {
    const el = document.createElement('div');
    el.textContent = item.name;
    container.appendChild(el);  // reflow ทุกครั้ง!
  });
}

// ✅ ถูก - ใช้ DocumentFragment
function goodDOMUpdate(container, items) {
  const fragment = document.createDocumentFragment();
  items.forEach(item => {
    const el = document.createElement('div');
    el.textContent = item.name;
    fragment.appendChild(el);
  });
  container.appendChild(fragment);  // reflow ครั้งเดียว
}

// Pitfall 3: Synchronous operations ที่บล็อค event loop
// ❌ ผิด - บล็อค event loop
function readFileSyncBad(path) {
  const fs = require('fs');
  return fs.readFileSync(path, 'utf8');  // บล็อคทุก request!
}

// ✅ ถูก - async
async function readFileGood(path) {
  const fs = require('fs').promises;
  return await fs.readFile(path, 'utf8');
}

// Pitfall 4: N+1 Query Problem
// ❌ ผิด - N+1 queries
async function getUsersWithOrdersBad(userIds) {
  const users = await User.findAll({ where: { id: userIds } }); // 1 query
  
  for (const user of users) {
    user.orders = await Order.findAll({ where: { userId: user.id } }); // N queries!
  }
  
  return users;
}

// ✅ ถูก - single query
async function getUsersWithOrdersGood(userIds) {
  const users = await User.findAll({
    where: { id: userIds },
    include: [{ model: Order }],  // JOIN ใน single query
  });
  return users;
}

// หรือ batch queries
async function getUsersWithOrdersBatch(userIds) {
  const [users, allOrders] = await Promise.all([
    User.findAll({ where: { id: userIds } }),
    Order.findAll({ where: { userId: userIds } }),  // 2 queries แทน N+1
  ]);
  
  const ordersByUser = groupBy(allOrders, o => o.userId);
  return users.map(u => ({ ...u, orders: ordersByUser[u.id] || [] }));
}

// Pitfall 5: JSON.parse/stringify ที่ใหญ่มาก
// ❌ ผิด - parse ทั้งก้อนสำหรับ 100MB JSON
const hugeParsed = JSON.parse(hugeJsonString);

// ✅ ถูก - streaming JSON parser
const { Parser } = require('@streamparser/json');

async function streamParseJSON(stream) {
  const parser = new Parser();
  const results = [];
  
  parser.onValue = ({ value, key, parent }) => {
    if (key === 'item') {
      results.push(value);
    }
  };
  
  for await (const chunk of stream) {
    parser.write(chunk);
  }
  
  return results;
}
```

---

## Step 1837: Profiling Node.js Apps in Production

```javascript
// Profiling tools

// 1. Node.js built-in profiler
// node --prof app.js
// หลังจากรัน
// node --prof-process isolate-*.log > profile.txt

// 2. V8 inspector ผ่าน Chrome DevTools
const { Session } = require('inspector');
const fs = require('fs');

async function collectCPUProfile(duration = 10000) {
  const session = new Session();
  session.connect();
  
  return new Promise((resolve, reject) => {
    session.post('Profiler.enable', () => {
      session.post('Profiler.start', () => {
        setTimeout(() => {
          session.post('Profiler.stop', (err, { profile }) => {
            if (err) return reject(err);
            
            fs.writeFileSync('profile.cpuprofile', JSON.stringify(profile));
            console.log('Profile saved to profile.cpuprofile');
            console.log('เปิดใน Chrome DevTools > Performance > Load profile');
            
            session.disconnect();
            resolve(profile);
          });
        }, duration);
      });
    });
  });
}

// ใช้งาน
// collectCPUProfile(10000).then(() => process.exit(0));

// 3. 0x - Flame graph profiler
// npm install -g 0x
// 0x app.js
// เปิด flame graph ใน browser

// 4. Clinic.js - All-in-one profiling
// npm install -g clinic
// clinic doctor -- node app.js
// clinic flame -- node app.js
// clinic bubbleprof -- node app.js
```

```javascript
// Custom performance monitoring

const { performance, PerformanceObserver } = require('perf_hooks');

// ตรวจสอบ performance marks
function measureOperation(name, fn) {
  const startMark = `${name}-start`;
  const endMark = `${name}-end`;
  
  performance.mark(startMark);
  const result = fn();
  performance.mark(endMark);
  
  performance.measure(name, startMark, endMark);
  
  const [measure] = performance.getEntriesByName(name);
  console.log(`${name}: ${measure.duration.toFixed(2)}ms`);
  
  return result;
}

// ตรวจสอบ async operations
async function measureAsync(name, fn) {
  const start = performance.now();
  try {
    const result = await fn();
    const duration = performance.now() - start;
    console.log(`${name}: ${duration.toFixed(2)}ms`);
    return result;
  } catch (error) {
    const duration = performance.now() - start;
    console.log(`${name} FAILED after ${duration.toFixed(2)}ms`);
    throw error;
  }
}

// Observer สำหรับ GC events
const obs = new PerformanceObserver((list) => {
  list.getEntries().forEach((entry) => {
    if (entry.entryType === 'gc') {
      console.log(`GC: ${entry.detail.kind} took ${entry.duration.toFixed(2)}ms`);
    }
  });
});
obs.observe({ entryTypes: ['gc'], buffered: true });

// Production metrics collector
class MetricsCollector {
  constructor() {
    this.metrics = new Map();
    this.startTime = Date.now();
  }

  record(name, value) {
    if (!this.metrics.has(name)) {
      this.metrics.set(name, { count: 0, total: 0, min: Infinity, max: -Infinity });
    }
    const m = this.metrics.get(name);
    m.count++;
    m.total += value;
    m.min = Math.min(m.min, value);
    m.max = Math.max(m.max, value);
  }

  getStats(name) {
    const m = this.metrics.get(name);
    if (!m) return null;
    return {
      count: m.count,
      avg: m.total / m.count,
      min: m.min,
      max: m.max,
      total: m.total,
    };
  }

  reportAll() {
    console.log('\n=== Performance Report ===');
    for (const [name, _] of this.metrics) {
      const stats = this.getStats(name);
      console.log(`${name}:`);
      console.log(`  Count: ${stats.count}`);
      console.log(`  Avg: ${stats.avg.toFixed(2)}ms`);
      console.log(`  Min: ${stats.min.toFixed(2)}ms`);
      console.log(`  Max: ${stats.max.toFixed(2)}ms`);
    }
  }
}

const metrics = new MetricsCollector();

// Express middleware สำหรับ request timing
function timingMiddleware(req, res, next) {
  const start = performance.now();
  
  res.on('finish', () => {
    const duration = performance.now() - start;
    const route = req.route?.path || req.path;
    metrics.record(`${req.method} ${route}`, duration);
  });
  
  next();
}
```

---

## Step 1838: Clinic.js for Node.js Profiling

```bash
# Clinic.js - comprehensive profiling toolkit
# npm install -g clinic

# 1. Clinic Doctor - ตรวจสอบ health
clinic doctor -- node app.js
# วิเคราะห์: CPU usage, event loop delay, memory, handles

# 2. Clinic Flame - CPU flame graph
clinic flame -- node app.js
# แสดง where CPU time is spent

# 3. Clinic BubbleProf - async profiling
clinic bubbleprof -- node app.js
# แสดง async operations และ bottlenecks

# 4. Clinic HeapProfiler - memory profiling
clinic heapprofiler -- node app.js
# แสดง memory allocations

# Load testing พร้อม profiling
# npm install -g autocannon
autocannon -c 100 -d 30 http://localhost:3000/api/users

# ตัวอย่างการรัน
# Terminal 1: รัน app
clinic flame -- node server.js

# Terminal 2: load test
autocannon -c 50 -d 10 http://localhost:3000/api/products

# ดู flame graph ใน browser
```

```javascript
// Node.js Event Loop Monitoring
const { monitorEventLoopDelay } = require('perf_hooks');

// ตรวจสอบ event loop lag
const histogram = monitorEventLoopDelay({ resolution: 20 });
histogram.enable();

setInterval(() => {
  const lag = histogram.mean / 1e6; // nanoseconds to milliseconds
  
  if (lag > 100) {
    console.warn(`HIGH EVENT LOOP LAG: ${lag.toFixed(2)}ms`);
  }
  
  console.log(`Event Loop Delay (P99): ${(histogram.percentile(99) / 1e6).toFixed(2)}ms`);
  histogram.reset();
}, 5000);

// ตรวจสอบ active handles/requests
function getActiveHandles() {
  return {
    handles: process._getActiveHandles().length,
    requests: process._getActiveRequests().length,
  };
}

// Heap snapshot
const v8 = require('v8');
const fs = require('fs');

function takeHeapSnapshot(filename = 'heap.heapsnapshot') {
  const snapshotStream = v8.writeHeapSnapshot();
  fs.copyFileSync(snapshotStream, filename);
  console.log(`Heap snapshot saved to ${filename}`);
  console.log('เปิดใน Chrome DevTools > Memory > Load Profile');
}

// รัน heap snapshot ตาม signal
process.on('SIGUSR2', () => {
  takeHeapSnapshot(`heap-${Date.now()}.heapsnapshot`);
});

// ส่ง signal
// kill -USR2 $(pgrep -f 'node server.js')
```

---

## Step 1839: Micro-benchmarking with Benchmark.js

```javascript
// npm install benchmark
const Benchmark = require('benchmark');

// สร้าง benchmark suite
const suite = new Benchmark.Suite('Array Operations');

const data = Array.from({ length: 10000 }, (_, i) => i);

suite
  .add('for loop', () => {
    let sum = 0;
    for (let i = 0; i < data.length; i++) {
      sum += data[i];
    }
    return sum;
  })
  .add('for...of', () => {
    let sum = 0;
    for (const item of data) {
      sum += item;
    }
    return sum;
  })
  .add('Array.reduce', () => {
    return data.reduce((sum, x) => sum + x, 0);
  })
  .add('forEach', () => {
    let sum = 0;
    data.forEach(x => { sum += x; });
    return sum;
  })
  .on('cycle', (event) => {
    console.log(String(event.target));
  })
  .on('complete', function() {
    console.log(`Fastest: ${this.filter('fastest').map('name')}`);
  })
  .run({ async: true });

// ตัวอย่างผล:
// for loop x 10,234,567 ops/sec ±0.54% (94 runs sampled)
// for...of x 9,876,543 ops/sec ±0.62% (92 runs sampled)
// Array.reduce x 7,654,321 ops/sec ±0.71% (91 runs sampled)
// forEach x 8,901,234 ops/sec ±0.58% (93 runs sampled)
// Fastest: for loop

// String operations benchmark
const stringSuite = new Benchmark.Suite('String Operations');

const strings = Array.from({ length: 1000 }, (_, i) => `item_${i}`);

stringSuite
  .add('string concatenation', () => {
    let result = '';
    for (const s of strings) result += s;
    return result;
  })
  .add('array join', () => {
    return strings.join('');
  })
  .add('template literal', () => {
    let result = '';
    for (const s of strings) result = `${result}${s}`;
    return result;
  })
  .add('Array push + join', () => {
    const parts = [];
    for (const s of strings) parts.push(s);
    return parts.join('');
  })
  .on('cycle', e => console.log(String(e.target)))
  .on('complete', function() {
    console.log(`Fastest: ${this.filter('fastest').map('name')}`);
  })
  .run({ async: true });
```

---

## Step 1840: Big O Complexity in Real JavaScript Code

```javascript
// ทำความเข้าใจ Big O ในโค้ดจริง

// O(1) - Constant time
function getFirst(arr) {
  return arr[0];  // เข้าถึงโดยตรง
}

function getByKey(map, key) {
  return map.get(key);  // Map lookup O(1)
}

// O(n) - Linear time
function findUser(users, id) {
  return users.find(u => u.id === id);  // worst case: scan all
}

function sumArray(arr) {
  return arr.reduce((s, x) => s + x, 0);  // visit each element once
}

// O(n log n) - Linearithmic
function sortData(arr) {
  return arr.sort((a, b) => a - b);  // V8 uses TimSort: O(n log n)
}

// O(n²) - Quadratic (ระวัง!)
function findDuplicatesBad(arr) {
  const duplicates = [];
  for (let i = 0; i < arr.length; i++) {
    for (let j = i + 1; j < arr.length; j++) {
      if (arr[i] === arr[j]) {
        duplicates.push(arr[i]);
      }
    }
  }
  return duplicates;
}

// O(n) - ใช้ Set แทน
function findDuplicatesGood(arr) {
  const seen = new Set();
  const duplicates = new Set();
  for (const item of arr) {
    if (seen.has(item)) {
      duplicates.add(item);
    } else {
      seen.add(item);
    }
  }
  return Array.from(duplicates);
}

// Real-world example: เปรียบเทียบ algorithm complexities
async function analyzeComplexity() {
  const sizes = [100, 1000, 10000, 100000];
  
  for (const n of sizes) {
    const data = Array.from({ length: n }, (_, i) => i);
    
    // O(n) - linear search
    const t1 = performance.now();
    data.find(x => x === n - 1);
    const linearTime = performance.now() - t1;
    
    // O(1) - Set lookup
    const set = new Set(data);
    const t2 = performance.now();
    set.has(n - 1);
    const setTime = performance.now() - t2;
    
    console.log(`n=${n}: Linear=${linearTime.toFixed(3)}ms, Set=${setTime.toFixed(3)}ms`);
  }
}

// ผล:
// n=100:    Linear=0.001ms, Set=0.000ms
// n=1000:   Linear=0.010ms, Set=0.000ms
// n=10000:  Linear=0.100ms, Set=0.000ms
// n=100000: Linear=1.000ms, Set=0.000ms
```

---

## Step 1841: Optimizing Array Operations

```javascript
// ลำดับ array methods ที่เร็วที่สุด
// for loop > for...of > forEach > map/filter/reduce

// 1. ใช้ early return
function findFirstPositive(arr) {
  // ❌ map ทั้ง array แล้วค่อย filter
  // return arr.map(x => x * 2).filter(x => x > 0)[0];
  
  // ✅ หยุดเมื่อเจอตัวแรก
  for (let i = 0; i < arr.length; i++) {
    if (arr[i] > 0) return arr[i] * 2;
  }
  return null;
}

// 2. Avoid creating intermediate arrays
const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

// ❌ ผิด - สร้าง intermediate array
const result1 = numbers
  .filter(x => x % 2 === 0)  // new array
  .map(x => x * x)           // another new array
  .filter(x => x > 10);      // yet another

// ✅ ถูก - single pass
const result2 = [];
for (const x of numbers) {
  if (x % 2 === 0) {
    const squared = x * x;
    if (squared > 10) result2.push(squared);
  }
}

// ✅ หรือใช้ flatMap สำหรับบางกรณี
const result3 = numbers.flatMap(x => {
  if (x % 2 !== 0) return [];
  const squared = x * x;
  return squared > 10 ? [squared] : [];
});

// 3. Pre-allocate array size
// ❌ ผิด - array grow dynamically
function buildArrayBad(n) {
  const arr = [];
  for (let i = 0; i < n; i++) {
    arr.push(i * 2);
  }
  return arr;
}

// ✅ ถูก - pre-allocate
function buildArrayGood(n) {
  const arr = new Array(n);
  for (let i = 0; i < n; i++) {
    arr[i] = i * 2;
  }
  return arr;
}

// ✅ หรือ typed array สำหรับ numbers
function buildTypedArray(n) {
  const arr = new Int32Array(n);
  for (let i = 0; i < n; i++) {
    arr[i] = i * 2;
  }
  return arr;
}

// 4. Sort optimization
// ❌ ผิด - sort string comparison (ช้า)
data.sort();

// ✅ ถูก - explicit comparison (เร็วกว่า)
data.sort((a, b) => a - b);

// สำหรับ complex objects
users.sort((a, b) => {
  // เปรียบเทียบ primary key ก่อน
  if (a.lastName !== b.lastName) {
    return a.lastName.localeCompare(b.lastName);
  }
  return a.firstName.localeCompare(b.firstName);
});

// Schwartzian Transform - precompute sort keys
const sorted = users
  .map(u => ({ user: u, key: `${u.lastName},${u.firstName}` }))  // precompute
  .sort((a, b) => a.key.localeCompare(b.key))  // sort by precomputed key
  .map(({ user }) => user);  // extract
```

---

## Step 1842: Optimizing String Operations

```javascript
// String optimization techniques

// 1. Template literals vs concatenation (modern V8 optimize ทั้งคู่)
// แต่ใน heavy loops, ใช้ array join

// 2. String search
// ❌ ผิด - Regular expression สำหรับ simple search
if (/hello/.test(str)) { /* ... */ }

// ✅ ถูก - includes() เร็วกว่า
if (str.includes('hello')) { /* ... */ }

// ✅ ดียิ่งขึ้น - indexOf() เร็วที่สุด
if (str.indexOf('hello') !== -1) { /* ... */ }

// 3. Compile RegExp ครั้งเดียว
// ❌ ผิด - compile ทุกครั้ง
function validateEmailBad(emails) {
  return emails.filter(email => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email));
}

// ✅ ถูก - compile ครั้งเดียว
const EMAIL_REGEX = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
function validateEmailGood(emails) {
  return emails.filter(email => EMAIL_REGEX.test(email));
}

// 4. String splitting
const csv = 'name,age,city,country';

// ✅ ธรรมดา
const parts = csv.split(',');

// สำหรับ large string, streaming approach
function* splitIterator(str, separator) {
  let start = 0;
  let index;
  
  while ((index = str.indexOf(separator, start)) !== -1) {
    yield str.slice(start, index);
    start = index + separator.length;
  }
  
  yield str.slice(start);
}

// ใช้ generator แทน array เพื่อประหยัด memory
for (const part of splitIterator(hugeCSV, '\n')) {
  processLine(part);
}

// 5. String building ใน Node.js streams
const { Readable } = require('stream');

function generateCSV(data) {
  const stream = new Readable({ read() {} });
  
  // Header
  stream.push('name,age,city\n');
  
  // Data rows
  for (const row of data) {
    stream.push(`${row.name},${row.age},${row.city}\n`);
  }
  
  stream.push(null); // end
  return stream;
}

// ส่งไปยัง response โดยตรง
app.get('/export', (req, res) => {
  res.setHeader('Content-Type', 'text/csv');
  res.setHeader('Content-Disposition', 'attachment; filename=data.csv');
  generateCSV(largeDataset).pipe(res);  // streaming ไม่ใช้ memory มาก
});
```

---

## Step 1843: Caching Computed Values

```javascript
// Caching strategies

// 1. Memoization
function memoize(fn) {
  const cache = new Map();
  
  return function(...args) {
    const key = JSON.stringify(args);
    
    if (cache.has(key)) {
      return cache.get(key);
    }
    
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}

// Fibonacci ด้วย memoization
const fib = memoize(function(n) {
  if (n <= 1) return n;
  return fib(n - 1) + fib(n - 2);
});

console.log(fib(50)); // เร็วมาก!

// 2. LRU Cache
class LRUCache {
  constructor(capacity) {
    this.capacity = capacity;
    this.cache = new Map();
  }

  get(key) {
    if (!this.cache.has(key)) return -1;
    
    // Move to end (most recently used)
    const value = this.cache.get(key);
    this.cache.delete(key);
    this.cache.set(key, value);
    return value;
  }

  put(key, value) {
    if (this.cache.has(key)) {
      this.cache.delete(key);
    } else if (this.cache.size >= this.capacity) {
      // Remove least recently used (first item)
      this.cache.delete(this.cache.keys().next().value);
    }
    this.cache.set(key, value);
  }

  get size() { return this.cache.size; }
}

// 3. Application-level caching
class CacheService {
  constructor({ ttl = 300, maxSize = 1000 } = {}) {
    this.cache = new LRUCache(maxSize);
    this.ttl = ttl * 1000; // convert to ms
    this.timestamps = new Map();
  }

  get(key) {
    const timestamp = this.timestamps.get(key);
    if (!timestamp || Date.now() - timestamp > this.ttl) {
      this.cache.cache.delete(key);
      this.timestamps.delete(key);
      return null;
    }
    return this.cache.get(key);
  }

  set(key, value) {
    this.cache.put(key, value);
    this.timestamps.set(key, Date.now());
  }

  invalidate(key) {
    this.cache.cache.delete(key);
    this.timestamps.delete(key);
  }

  invalidatePattern(pattern) {
    for (const key of this.cache.cache.keys()) {
      if (key.match(pattern)) {
        this.invalidate(key);
      }
    }
  }
}

const cache = new CacheService({ ttl: 300, maxSize: 1000 });

// ใช้กับ Express
async function getProductWithCache(productId) {
  const cacheKey = `product:${productId}`;
  const cached = cache.get(cacheKey);
  
  if (cached) {
    return { ...cached, fromCache: true };
  }
  
  const product = await productRepository.findById(productId);
  if (product) {
    cache.set(cacheKey, product);
  }
  
  return product;
}

// 4. Redis caching
const redis = require('redis');
const client = redis.createClient();

async function getWithRedisCache(key, fetchFn, ttl = 300) {
  // ลอง get จาก cache
  const cached = await client.get(key);
  if (cached) {
    return JSON.parse(cached);
  }
  
  // Fetch fresh data
  const data = await fetchFn();
  
  // Cache it
  await client.setex(key, ttl, JSON.stringify(data));
  
  return data;
}

// ใช้งาน
async function getPopularProducts() {
  return getWithRedisCache(
    'popular:products',
    () => productRepository.findPopular(),
    600 // 10 minutes
  );
}
```

---

## Step 1844: Lazy Initialization

```javascript
// Lazy initialization patterns

// 1. Lazy property
class HeavyCalculation {
  constructor(data) {
    this.data = data;
    // ไม่คำนวณ expensive ใน constructor
  }

  // Compute on first access
  get statistics() {
    if (!this._statistics) {
      this._statistics = this._calculateStatistics();
    }
    return this._statistics;
  }

  get histogram() {
    if (!this._histogram) {
      this._histogram = this._buildHistogram();
    }
    return this._histogram;
  }

  _calculateStatistics() {
    // expensive computation
    const values = this.data.map(d => d.value);
    return {
      mean: values.reduce((s, x) => s + x, 0) / values.length,
      min: Math.min(...values),
      max: Math.max(...values),
    };
  }

  _buildHistogram() {
    // expensive computation
    const buckets = new Array(10).fill(0);
    for (const item of this.data) {
      const bucket = Math.min(Math.floor(item.value / 10), 9);
      buckets[bucket]++;
    }
    return buckets;
  }
}

// 2. Lazy module loading
let _heavyModule = null;

function getHeavyModule() {
  if (!_heavyModule) {
    _heavyModule = require('./heavy-module');  // load เมื่อต้องการ
  }
  return _heavyModule;
}

// 3. Lazy database connection
class Database {
  constructor(connectionString) {
    this.connectionString = connectionString;
    this._connection = null;
  }

  async getConnection() {
    if (!this._connection) {
      this._connection = await createConnection(this.connectionString);
    }
    return this._connection;
  }

  async query(sql, params) {
    const conn = await this.getConnection();
    return conn.execute(sql, params);
  }
}

// 4. WeakMap for lazy computed properties
const computedCache = new WeakMap();

function getExpensiveData(obj) {
  if (computedCache.has(obj)) {
    return computedCache.get(obj);
  }
  
  const result = expensiveComputation(obj);
  computedCache.set(obj, result);
  return result;
}
// WeakMap ทำให้ cache ถูก GC เมื่อ obj ถูก GC
```

---

## Step 1845: Connection Pooling

```javascript
// Database Connection Pooling

// PostgreSQL Pool ด้วย pg
const { Pool } = require('pg');

const pgPool = new Pool({
  connectionString: process.env.DATABASE_URL,
  max: 20,           // maximum connections
  min: 5,            // minimum connections (keep alive)
  idleTimeoutMillis: 30000,  // close idle connections after 30s
  connectionTimeoutMillis: 2000,  // fail if can't connect within 2s
  maxUses: 7500,     // close connection after 7500 uses (prevent memory leaks)
});

// ตรวจสอบ pool health
pgPool.on('connect', (client) => {
  console.log('New client connected to PostgreSQL');
});

pgPool.on('error', (err, client) => {
  console.error('Unexpected error on idle PostgreSQL client', err);
});

// ใช้งาน
async function query(sql, params) {
  const client = await pgPool.connect();
  try {
    const result = await client.query(sql, params);
    return result.rows;
  } finally {
    client.release();  // IMPORTANT: always release!
  }
}

// Transaction
async function withTransaction(fn) {
  const client = await pgPool.connect();
  try {
    await client.query('BEGIN');
    const result = await fn(client);
    await client.query('COMMIT');
    return result;
  } catch (error) {
    await client.query('ROLLBACK');
    throw error;
  } finally {
    client.release();
  }
}

// MongoDB connection pooling
const mongoose = require('mongoose');

mongoose.connect(process.env.MONGODB_URL, {
  maxPoolSize: 10,      // maximum connections
  minPoolSize: 5,       // minimum connections
  serverSelectionTimeoutMS: 5000,
  socketTimeoutMS: 45000,
  heartbeatFrequencyMS: 10000,
});

// Redis connection pooling
const Redis = require('ioredis');

const redis = new Redis.Cluster(
  [
    { host: '127.0.0.1', port: 7000 },
    { host: '127.0.0.1', port: 7001 },
  ],
  {
    redisOptions: {
      password: process.env.REDIS_PASSWORD,
      maxRetriesPerRequest: 3,
    },
    enableReadyCheck: true,
    scaleReads: 'slave',  // read จาก replicas
  }
);

// HTTP connection pooling
const http = require('http');
const https = require('https');

const httpAgent = new http.Agent({
  keepAlive: true,
  maxSockets: 100,
  maxFreeSockets: 10,
  timeout: 30000,
});

const fetch = require('node-fetch');
const response = await fetch(url, {
  agent: url.startsWith('https') ? new https.Agent({ keepAlive: true }) : httpAgent,
});
```

---

## Step 1846: Node.js Clustering in Production

```javascript
// Node.js Clustering
// ใช้ประโยชน์จาก multiple CPU cores

// cluster.js
const cluster = require('cluster');
const os = require('os');
const http = require('http');

const numCPUs = os.cpus().length;

if (cluster.isPrimary) {
  console.log(`Primary ${process.pid} is running`);
  console.log(`Starting ${numCPUs} workers...`);
  
  // Fork workers
  for (let i = 0; i < numCPUs; i++) {
    const worker = cluster.fork();
    
    worker.on('message', (msg) => {
      if (msg.type === 'metrics') {
        collectMetrics(worker.id, msg.data);
      }
    });
  }
  
  // Replace crashed workers
  cluster.on('exit', (worker, code, signal) => {
    console.log(`Worker ${worker.process.pid} died (code: ${code}, signal: ${signal})`);
    console.log('Starting new worker...');
    cluster.fork();
  });
  
  // Graceful restart
  process.on('SIGUSR2', () => {
    const workers = Object.values(cluster.workers);
    let i = 0;
    
    const restartNext = () => {
      if (i >= workers.length) {
        console.log('All workers restarted');
        return;
      }
      
      const worker = workers[i++];
      console.log(`Stopping worker ${worker.process.pid}...`);
      
      worker.send('shutdown');
      worker.disconnect();
      
      cluster.fork().on('listening', restartNext);
    };
    
    restartNext();
  });
  
} else {
  // Worker code
  const app = require('./app');
  const server = http.createServer(app);
  
  server.listen(process.env.PORT || 3000, () => {
    console.log(`Worker ${process.pid} started`);
  });
  
  // Send metrics to primary
  setInterval(() => {
    process.send({
      type: 'metrics',
      data: {
        pid: process.pid,
        memory: process.memoryUsage(),
        uptime: process.uptime(),
      },
    });
  }, 5000);
  
  // Graceful shutdown
  process.on('message', (msg) => {
    if (msg === 'shutdown') {
      console.log(`Worker ${process.pid} shutting down gracefully...`);
      
      server.close(() => {
        process.exit(0);
      });
      
      // Force exit after 30s
      setTimeout(() => process.exit(0), 30000);
    }
  });
}
```

---

## Step 1847: PM2 for Process Management

```bash
# PM2 - Production Process Manager
npm install -g pm2

# รัน application
pm2 start app.js
pm2 start app.js --name "my-api"

# Cluster mode (ใช้ all CPUs)
pm2 start app.js -i max --name "my-api"
# หรือระบุ number of instances
pm2 start app.js -i 4 --name "my-api"

# ดู status
pm2 status
pm2 list

# ดู logs
pm2 logs
pm2 logs my-api
pm2 logs --lines 100

# ดู metrics
pm2 monit

# Restart
pm2 restart my-api
pm2 reload my-api  # zero-downtime reload

# Stop/Delete
pm2 stop my-api
pm2 delete my-api

# Startup script (auto-start on reboot)
pm2 startup
pm2 save

# Update (zero-downtime)
pm2 reload my-api

# Scale
pm2 scale my-api 8   # scale to 8 instances
pm2 scale my-api +2  # add 2 more instances
pm2 scale my-api -1  # remove 1 instance
```

```javascript
// ecosystem.config.js - PM2 configuration
module.exports = {
  apps: [
    {
      name: 'my-api',
      script: './src/index.js',
      
      // Cluster mode
      instances: 'max',
      exec_mode: 'cluster',
      
      // Environment
      env: {
        NODE_ENV: 'development',
        PORT: 3000,
      },
      env_production: {
        NODE_ENV: 'production',
        PORT: 8080,
      },
      
      // Memory limit & restart
      max_memory_restart: '500M',
      
      // Logs
      log_file: './logs/combined.log',
      out_file: './logs/out.log',
      error_file: './logs/error.log',
      log_date_format: 'YYYY-MM-DD HH:mm:ss Z',
      
      // Restart behavior
      exp_backoff_restart_delay: 100,
      max_restarts: 10,
      
      // Watch (dev only)
      watch: false,
      ignore_watch: ['node_modules', 'logs'],
      
      // Graceful shutdown
      kill_timeout: 10000,
      wait_ready: true,
      listen_timeout: 10000,
    },
  ],
};

// รัน production
// pm2 start ecosystem.config.js --env production
```

---

## Step 1848: Advanced Performance Patterns

```javascript
// Worker Threads สำหรับ CPU-intensive tasks

const { Worker, isMainThread, parentPort, workerData } = require('worker_threads');

// worker.js
if (!isMainThread) {
  // CPU-intensive work ใน separate thread
  const { data, operation } = workerData;
  
  let result;
  if (operation === 'sort') {
    result = [...data].sort((a, b) => a - b);
  } else if (operation === 'hash') {
    const crypto = require('crypto');
    result = data.map(item => 
      crypto.createHash('sha256').update(String(item)).digest('hex')
    );
  }
  
  parentPort.postMessage(result);
}

// main.js
function runInWorker(data, operation) {
  return new Promise((resolve, reject) => {
    const worker = new Worker(__filename, {
      workerData: { data, operation },
    });
    
    worker.on('message', resolve);
    worker.on('error', reject);
    worker.on('exit', (code) => {
      if (code !== 0) {
        reject(new Error(`Worker stopped with exit code ${code}`));
      }
    });
  });
}

// Worker Pool
class WorkerPool {
  constructor(size, workerPath) {
    this.size = size;
    this.workers = [];
    this.queue = [];
    this.available = [];
    
    for (let i = 0; i < size; i++) {
      this._createWorker(workerPath);
    }
  }
  
  _createWorker(workerPath) {
    const worker = new Worker(workerPath);
    
    worker.on('message', (result) => {
      const { resolve } = worker._currentTask;
      worker._currentTask = null;
      this.available.push(worker);
      this._processQueue();
      resolve(result);
    });
    
    worker.on('error', (error) => {
      const { reject } = worker._currentTask;
      worker._currentTask = null;
      this.available.push(worker);
      this._processQueue();
      reject(error);
    });
    
    this.workers.push(worker);
    this.available.push(worker);
  }
  
  run(data) {
    return new Promise((resolve, reject) => {
      this.queue.push({ data, resolve, reject });
      this._processQueue();
    });
  }
  
  _processQueue() {
    if (this.queue.length === 0 || this.available.length === 0) return;
    
    const worker = this.available.pop();
    const task = this.queue.shift();
    
    worker._currentTask = task;
    worker.postMessage(task.data);
  }
  
  async destroy() {
    await Promise.all(this.workers.map(w => w.terminate()));
  }
}

const pool = new WorkerPool(4, './worker.js');

// ใช้งาน
async function processLargeDataset(dataset) {
  const chunks = chunk(dataset, Math.ceil(dataset.length / 4));
  const results = await Promise.all(chunks.map(c => pool.run(c)));
  return results.flat();
}
```

---

## Step 1849: Memory Optimization Techniques

```javascript
// Memory optimization

// 1. Stream large files instead of loading into memory
const fs = require('fs');
const { pipeline } = require('stream/promises');
const { Transform } = require('stream');

async function processLargeFile(inputPath, outputPath) {
  const input = fs.createReadStream(inputPath);
  const output = fs.createWriteStream(outputPath);
  
  const transform = new Transform({
    transform(chunk, encoding, callback) {
      const processed = processChunk(chunk.toString());
      callback(null, processed);
    },
  });
  
  await pipeline(input, transform, output);
}

// 2. Use generators for large sequences
function* range(start, end, step = 1) {
  for (let i = start; i < end; i += step) {
    yield i;
  }
}

function* map(iterable, fn) {
  for (const item of iterable) {
    yield fn(item);
  }
}

function* filter(iterable, predicate) {
  for (const item of iterable) {
    if (predicate(item)) yield item;
  }
}

// ใช้งาน - ไม่สร้าง intermediate arrays
for (const value of filter(map(range(0, 1000000), x => x * 2), x => x % 3 === 0)) {
  process(value);
}

// 3. Reuse buffers
const BUFFER_SIZE = 64 * 1024; // 64KB
const sharedBuffer = Buffer.allocUnsafe(BUFFER_SIZE);

async function readFileChunks(filePath, callback) {
  const fd = await fs.promises.open(filePath, 'r');
  
  try {
    let bytesRead;
    let position = 0;
    
    do {
      ({ bytesRead } = await fd.read(sharedBuffer, 0, BUFFER_SIZE, position));
      if (bytesRead > 0) {
        await callback(sharedBuffer.slice(0, bytesRead));
        position += bytesRead;
      }
    } while (bytesRead === BUFFER_SIZE);
    
  } finally {
    await fd.close();
  }
}

// 4. WeakRef สำหรับ optional caching
class SoftCache {
  constructor() {
    this.cache = new Map();
  }
  
  set(key, value) {
    this.cache.set(key, new WeakRef(value));
  }
  
  get(key) {
    const ref = this.cache.get(key);
    if (!ref) return undefined;
    
    const value = ref.deref();
    if (!value) {
      this.cache.delete(key);  // clean up
      return undefined;
    }
    
    return value;
  }
}
```

---

## Step 1850: Production Monitoring and Alerting

```javascript
// Production monitoring setup

// Memory monitoring
function setupMemoryMonitoring() {
  const MEMORY_LIMIT = 400 * 1024 * 1024; // 400MB
  
  setInterval(() => {
    const { heapUsed, heapTotal, external, rss } = process.memoryUsage();
    
    const stats = {
      heapUsed: Math.round(heapUsed / 1024 / 1024),
      heapTotal: Math.round(heapTotal / 1024 / 1024),
      external: Math.round(external / 1024 / 1024),
      rss: Math.round(rss / 1024 / 1024),
    };
    
    // Log to metrics service
    metrics.gauge('memory.heap_used', stats.heapUsed);
    metrics.gauge('memory.rss', stats.rss);
    
    if (heapUsed > MEMORY_LIMIT) {
      console.error(`HIGH MEMORY USAGE: ${stats.heapUsed}MB`);
      alerting.send('MEMORY_ALERT', stats);
      
      // Force GC if available
      if (global.gc) global.gc();
    }
  }, 30000);
}

// CPU monitoring
function setupCPUMonitoring() {
  let previousCpuUsage = process.cpuUsage();
  
  setInterval(() => {
    const currentUsage = process.cpuUsage(previousCpuUsage);
    previousCpuUsage = process.cpuUsage();
    
    // เปลี่ยนเป็น percentage
    const totalMicroseconds = 1000 * 1000; // 1 second
    const userPercent = (currentUsage.user / totalMicroseconds) * 100;
    const systemPercent = (currentUsage.system / totalMicroseconds) * 100;
    
    metrics.gauge('cpu.user', userPercent);
    metrics.gauge('cpu.system', systemPercent);
    
    if (userPercent + systemPercent > 80) {
      console.warn(`HIGH CPU USAGE: ${(userPercent + systemPercent).toFixed(1)}%`);
    }
  }, 1000);
}

// Request performance monitoring
function performanceMiddleware(req, res, next) {
  const start = process.hrtime.bigint();
  
  res.on('finish', () => {
    const duration = Number(process.hrtime.bigint() - start) / 1e6; // to ms
    
    metrics.histogram('http.request.duration', duration, {
      method: req.method,
      route: req.route?.path || 'unknown',
      status: res.statusCode,
    });
    
    if (duration > 1000) {
      console.warn(`SLOW REQUEST: ${req.method} ${req.path} took ${duration.toFixed(2)}ms`);
    }
  });
  
  next();
}

// ตัวอย่าง Performance Summary Report
function generatePerformanceReport() {
  const report = {
    timestamp: new Date().toISOString(),
    uptime: process.uptime(),
    memory: process.memoryUsage(),
    cpu: process.cpuUsage(),
    requests: {
      total: requestCount,
      perSecond: requestCount / process.uptime(),
      avgDuration: totalDuration / requestCount,
    },
    database: {
      activeConnections: pgPool.totalCount - pgPool.idleCount,
      idleConnections: pgPool.idleCount,
      waitingClients: pgPool.waitingCount,
    },
  };
  
  return report;
}

app.get('/metrics', (req, res) => {
  res.json(generatePerformanceReport());
});
```

---

## แบบฝึกหัด (Exercises)

### Exercise 1: Benchmark Array Methods
สร้าง benchmark เปรียบเทียบ:
- for loop vs for...of vs forEach vs reduce
- ขนาด array: 100, 1000, 10000, 100000
- Operations: sum, filter, map, find
- สรุปผลและอธิบายว่าทำไม

### Exercise 2: Fix Memory Leak
แก้ไข memory leak ในโค้ดต่อไปนี้:
```javascript
class EventManager {
  constructor() {
    this.handlers = [];
  }
  
  on(event, handler) {
    process.on(event, handler);
    this.handlers.push(handler);
  }
  
  // Missing cleanup!
}
```

### Exercise 3: Implement LRU Cache with TTL
สร้าง LRU Cache ที่:
- มี capacity limit
- มี TTL (time to live) ต่อ entry
- Thread-safe (ใช้กับ async code)
- มี hit/miss statistics
- เขียน tests ครบถ้วน

### Exercise 4: Optimize Slow Function
Profile และ optimize function ต่อไปนี้:
```javascript
function processOrders(orders) {
  return orders
    .filter(o => o.status === 'active')
    .map(o => ({ ...o, total: o.items.reduce((s, i) => s + i.price * i.qty, 0) }))
    .filter(o => o.total > 1000)
    .sort((a, b) => b.total - a.total)
    .slice(0, 100);
}
```

### Exercise 5: Production Monitoring Setup
ตั้งค่า monitoring สำหรับ Express app:
- Request timing middleware
- Memory usage alerts
- Database query time tracking
- Error rate tracking
- Dashboard endpoint ที่แสดง metrics

---

*จบ Part 93: Performance at Scale*
*ต่อไป Part 94: Advanced Security*
