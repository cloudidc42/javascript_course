# Part 35: Callbacks และ Event Loop (Steps 671-690)

## บทนำ

JavaScript เป็นภาษาที่ทำงานแบบ Single-threaded แต่สามารถจัดการ asynchronous operations ได้ผ่าน Event Loop ซึ่งเป็นกลไกสำคัญที่ทำให้ JavaScript สามารถทำงานได้อย่างมีประสิทธิภาพ การเข้าใจ Event Loop จะช่วยให้เขียนโค้ด async ได้ถูกต้องและหลีกเลี่ยง bugs ที่ซับซ้อน

---

## Step 671: JavaScript Runtime Environment

JavaScript ทำงานอยู่ใน Runtime Environment ที่มีส่วนประกอบหลายอย่าง:

```javascript
// ภาพรวมของ JavaScript Runtime
// ┌─────────────────────────────────────────────┐
// │              JavaScript Engine               │
// │  ┌──────────────┐  ┌───────────────────────┐│
// │  │  Call Stack  │  │   Memory Heap         ││
// │  │              │  │ (Object allocation)   ││
// │  │ main()       │  │                       ││
// │  │ foo()        │  │                       ││
// │  │ bar()        │  │                       ││
// │  └──────────────┘  └───────────────────────┘│
// └─────────────────────────────────────────────┘
//
// ┌─────────────────────────────────────────────┐
// │              Web APIs / Node.js APIs         │
// │  setTimeout │ fetch │ DOM events │ fs.read   │
// └─────────────────────────────────────────────┘
//
// ┌─────────────────────────────────────────────┐
// │              Event Loop                      │
// │                                              │
// │  Microtask Queue (Promises, queueMicrotask)  │
// │  Macrotask Queue (setTimeout, setInterval)   │
// └─────────────────────────────────────────────┘

// Memory Heap: เก็บ objects
const obj = { name: 'อลิส' }; // เก็บใน heap

// Call Stack: เก็บ function calls
function greet(name) {
  const message = `สวัสดี ${name}`; // local variable ใน stack frame
  return message;
}

console.log(greet('บ็อบ')); // สวัสดี บ็อบ
```

```javascript
// Call Stack ทำงานอย่างไร
function third() {
  console.log('Third function');
  // Stack: [main, second, third] <- top
}

function second() {
  console.log('Before third');
  third();
  // Stack เมื่อ third return: [main, second] <- top
  console.log('After third');
}

function first() {
  console.log('Before second');
  second();
  console.log('After second');
}

first();
// Stack progression:
// 1. [main]
// 2. [main, first]
// 3. [main, first, second]
// 4. [main, first, second, third]
// 5. [main, first, second] <- third returned
// 6. [main, first] <- second returned
// 7. [main] <- first returned
```

---

## Step 672: Call Stack

```javascript
// Stack Overflow - เมื่อ stack เต็ม
function infinite() {
  return infinite(); // เรียกตัวเองไม่สิ้นสุด
}

try {
  infinite();
} catch (e) {
  console.log(e instanceof RangeError); // true
  console.log(e.message); // Maximum call stack size exceeded
}

// Tail Call Optimization (TCO)
// ES6 มี TCO แต่ browser support ยังจำกัด
function factorialTCO(n, accumulator = 1) {
  if (n <= 1) return accumulator;
  return factorialTCO(n - 1, n * accumulator); // tail position
}

// iterative version (ปลอดภัยกว่า)
function factorial(n) {
  let result = 1;
  for (let i = 2; i <= n; i++) {
    result *= i;
  }
  return result;
}

console.log(factorial(10)); // 3628800
```

```javascript
// Call Stack Visualization สำหรับ async code
console.log('1 - Sync start');

setTimeout(() => {
  console.log('3 - Timeout callback');
}, 0);

Promise.resolve().then(() => {
  console.log('2.5 - Microtask');
});

console.log('2 - Sync end');

// Output:
// 1 - Sync start
// 2 - Sync end
// 2.5 - Microtask  <- microtask queue (ก่อน macrotask!)
// 3 - Timeout callback  <- macrotask queue

// เหตุผล: Microtasks (Promises) มีความสำคัญสูงกว่า Macrotasks (setTimeout)
```

---

## Step 673: Web APIs / Node.js APIs

```javascript
// Web APIs ทำงาน "นอก" JavaScript engine
// พวกมัน asynchronous โดยธรรมชาติ

// 1. Timer APIs
console.log('Start');

const timerId = setTimeout(() => {
  console.log('Timeout 1s');
}, 1000);

const intervalId = setInterval(() => {
  console.log('Interval every 500ms');
}, 500);

// ยกเลิก timer
setTimeout(() => {
  clearInterval(intervalId);
  console.log('Cleared interval');
}, 2200);

// 2. Network APIs (fetch)
// fetch('https://api.example.com/data')
//   .then(res => res.json())
//   .then(data => console.log(data));

// 3. DOM Events (browser only)
// document.addEventListener('click', (event) => {
//   console.log('Clicked at:', event.clientX, event.clientY);
// });

// 4. File System (Node.js)
// const fs = require('fs');
// fs.readFile('file.txt', 'utf8', (err, data) => {
//   console.log(data);
// });
```

```javascript
// API Calls ทำงานนอก JS Thread
function demonstrateAsync() {
  console.log('1. เริ่มต้น');

  // setTimeout ส่งให้ Web API รับผิดชอบ
  // JavaScript ทำงานต่อได้ทันที
  setTimeout(() => {
    console.log('4. setTimeout callback (หลังสุด)');
  }, 0);

  // fetch ส่งให้ Web API (network) ดำเนินการ
  // JavaScript ทำงานต่อได้ทันที
  fetch('https://jsonplaceholder.typicode.com/posts/1')
    .then(res => res.json())
    .then(data => console.log('3. Fetch result:', data.title));

  console.log('2. ทำงานต่อ (synchronous)');
}

// demonstrateAsync();
// Output order:
// 1. เริ่มต้น
// 2. ทำงานต่อ (synchronous)
// 3. Fetch result: ... (เมื่อ network ตอบกลับ)
// 4. setTimeout callback (หลังสุด)
```

---

## Step 674: Callback Queue (Macrotask Queue)

```javascript
// Macrotask Queue เก็บ callbacks จาก:
// - setTimeout
// - setInterval
// - setImmediate (Node.js)
// - I/O events
// - UI rendering events

// ลำดับ Macrotask execution
console.log('Script start');

setTimeout(() => console.log('setTimeout 1'), 0);
setTimeout(() => console.log('setTimeout 2'), 0);
setTimeout(() => console.log('setTimeout 3'), 100);

// Promise microtasks มาก่อน macrotasks
Promise.resolve().then(() => console.log('Promise 1'));

console.log('Script end');

// Output:
// Script start
// Script end
// Promise 1        <- microtask (ก่อน macrotask)
// setTimeout 1     <- macrotask
// setTimeout 2     <- macrotask
// (100ms later)
// setTimeout 3     <- macrotask
```

```javascript
// Timer accuracy - setTimeout ไม่รับประกันเวลาที่แน่นอน
function measureDelay(delay) {
  const start = Date.now();
  setTimeout(() => {
    const actual = Date.now() - start;
    console.log(`ตั้ง: ${delay}ms, จริง: ${actual}ms`);
  }, delay);
}

measureDelay(0);    // อาจจะได้ 1-5ms
measureDelay(100);  // อาจจะได้ 100-105ms

// เหตุที่ไม่แม่นยำ:
// 1. Call Stack ยังไม่ว่าง
// 2. Browser throttle tabs ที่ inactive
// 3. System timer resolution (~4ms minimum)
// 4. Long running tasks บล็อก event loop

// ตัวอย่าง setTimeout ถูกชะลอเพราะ long task
setTimeout(() => console.log('Timeout (อาจช้า)'), 0);

// Long synchronous task - บล็อก event loop!
const start = Date.now();
while (Date.now() - start < 100) { /* busy wait */ }

// setTimeout จะรันหลังจาก loop เสร็จ (>100ms จริงๆ)
```

---

## Step 675: Microtask Queue

```javascript
// Microtask Queue มีความสำคัญสูงกว่า Macrotask
// Microtasks มาจาก:
// - Promise.then/catch/finally
// - queueMicrotask()
// - MutationObserver (browser)

// Event Loop ลำดับ:
// 1. รัน synchronous code ใน Call Stack จนหมด
// 2. รัน ALL microtasks จนหมด (รวม microtasks ที่สร้างใหม่)
// 3. รัน ONE macrotask
// 4. กลับไป step 2

console.log('1: Sync');

setTimeout(() => {
  console.log('5: Macrotask 1');
  Promise.resolve().then(() => console.log('6: Microtask from Macrotask'));
}, 0);

setTimeout(() => console.log('7: Macrotask 2'), 0);

Promise.resolve()
  .then(() => {
    console.log('3: Microtask 1');
    return Promise.resolve();
  })
  .then(() => console.log('4: Microtask 2'));

console.log('2: Sync end');

// Output:
// 1: Sync
// 2: Sync end
// 3: Microtask 1
// 4: Microtask 2
// 5: Macrotask 1
// 6: Microtask from Macrotask  <- ทำงานก่อน Macrotask 2!
// 7: Macrotask 2
```

```javascript
// Microtask flooding - ระวัง infinite microtasks!
let count = 0;

function potentiallyProblematic() {
  if (count++ < 5) {
    Promise.resolve().then(potentiallyProblematic);
    console.log(`Microtask ${count}`);
  }
}

potentiallyProblematic();
setTimeout(() => console.log('Macrotask'), 0);

// Output:
// Microtask 1
// Microtask 2
// Microtask 3
// Microtask 4
// Microtask 5
// Macrotask  <- รอจนกว่า microtasks หมด
```

---

## Step 676: Event Loop Mechanism

```javascript
// Event Loop Algorithm (simplified):
/*
while (true) {
  // 1. รัน synchronous code จน Call Stack ว่าง
  runCurrentTask();
  
  // 2. รัน microtasks ทั้งหมด
  while (microtaskQueue.length > 0) {
    const task = microtaskQueue.shift();
    runTask(task);
    // Microtasks ที่สร้างใหม่จะถูก run ทันทีในรอบนี้ด้วย
  }
  
  // 3. (Browser only) ถ้าถึงเวลา render
  if (shouldRender()) {
    render();
  }
  
  // 4. รัน macrotask ถัดไป (ถ้ามี)
  if (macrotaskQueue.length > 0) {
    const task = macrotaskQueue.shift();
    runTask(task);
  }
}
*/

// ตัวอย่าง event loop behavior
async function demonstrate() {
  console.log('A: sync start');

  await Promise.resolve('immediate');
  console.log('C: after await'); // microtask

  setTimeout(() => console.log('E: setTimeout'), 0); // macrotask

  await new Promise(resolve => setTimeout(resolve, 0));
  console.log('F: after awaited timeout'); // หลัง macrotask

  console.log('G: sync end');
}

console.log('B: before async');
demonstrate();
console.log('D: after async call');

// Output:
// A: sync start (synchronous ใน demonstrate)
// B: before async
// D: after async call
// C: after await       <- microtask
// E: setTimeout        <- macrotask 
// F: after awaited timeout <- หลัง macrotask
// G: sync end
```

---

## Step 677: setTimeout และ setInterval

```javascript
// setTimeout - ทำงานครั้งเดียวหลัง delay
function countdown(from, onComplete) {
  let count = from;

  function tick() {
    console.log(count--);
    if (count >= 0) {
      setTimeout(tick, 1000);
    } else {
      onComplete();
    }
  }

  tick();
}

// countdown(5, () => console.log('🚀 Launch!'));
// 5, 4, 3, 2, 1, 0, Launch!

// setInterval - ทำงานซ้ำทุก interval
function createClock() {
  let id;

  function start() {
    id = setInterval(() => {
      const now = new Date();
      console.log(now.toLocaleTimeString('th-TH'));
    }, 1000);
    return this;
  }

  function stop() {
    clearInterval(id);
    id = null;
    return this;
  }

  return { start, stop };
}

const clock = createClock();
// clock.start();
// setTimeout(() => clock.stop(), 5000);
```

```javascript
// setTimeout กับ delay = 0 (ไม่ได้ทำงานทันที)
console.log('1');
setTimeout(() => console.log('3'), 0); // หลัง synchronous code ทั้งหมด
console.log('2');
// Output: 1, 2, 3

// ใช้ setTimeout(fn, 0) สำหรับ:
// 1. ทำงาน "after current call stack"
function updateUI(data) {
  // ทำงาน sync ก่อน
  const result = processData(data);

  // update UI ใน next tick
  setTimeout(() => {
    renderResult(result);
    console.log('UI updated');
  }, 0);

  // return ทันที ไม่รอ UI update
  return result;
}

function processData(d) { return d; }
function renderResult(r) { console.log('Rendering:', r); }

// 2. ทำลาย "maximum call stack" ใน recursion
function processBatch(items, batchSize = 100) {
  function processNext(index) {
    const end = Math.min(index + batchSize, items.length);
    for (let i = index; i < end; i++) {
      // process items[i]
    }

    if (end < items.length) {
      setTimeout(() => processNext(end), 0); // ให้ event loop "หายใจ"
    } else {
      console.log('Done!');
    }
  }

  processNext(0);
}

processBatch(new Array(1000).fill('item'));
```

---

## Step 678: clearTimeout และ clearInterval

```javascript
// clearTimeout - ยกเลิก timeout ที่ยังไม่ทำงาน
function withTimeout(promise, ms) {
  let timeoutId;

  const timeout = new Promise((_, reject) => {
    timeoutId = setTimeout(() => {
      reject(new Error(`Timeout หลัง ${ms}ms`));
    }, ms);
  });

  return Promise.race([promise, timeout])
    .finally(() => clearTimeout(timeoutId)); // cleanup
}

// ใช้งาน
async function fetchWithTimeout(url, ms = 5000) {
  const fetchPromise = fetch(url).then(r => r.json());
  return withTimeout(fetchPromise, ms);
}

// clearInterval - ยกเลิก interval
function createAutoSave(getData, interval = 5000) {
  let id = null;
  let lastSaved = null;

  function save() {
    const data = getData();
    if (JSON.stringify(data) !== JSON.stringify(lastSaved)) {
      lastSaved = data;
      console.log('Auto-saved:', data);
    }
  }

  return {
    start() {
      if (id) return; // ไม่ start ซ้ำ
      id = setInterval(save, interval);
      console.log('Auto-save started');
    },
    stop() {
      clearInterval(id);
      id = null;
      console.log('Auto-save stopped');
    },
    saveNow: save
  };
}

let documentContent = 'Initial content';
const autoSave = createAutoSave(() => documentContent);
autoSave.start();
// documentContent = 'Modified content'; // triggers save
// setTimeout(() => autoSave.stop(), 15000);
```

---

## Step 679: Promise Microtasks

```javascript
// Promises สร้าง microtasks
const p1 = Promise.resolve(1);
const p2 = Promise.resolve(2);

p1.then(v => {
  console.log('P1 first then:', v);
  return v + 10;
}).then(v => {
  console.log('P1 second then:', v);
});

p2.then(v => {
  console.log('P2 then:', v);
});

console.log('Sync code');

// Output:
// Sync code
// P1 first then: 1   <- microtask queue order
// P2 then: 2         <- microtask queue order
// P1 second then: 11 <- microtask สร้างใหม่จาก first then
```

```javascript
// Promise chain เป็น microtasks ที่ต่อเนื่องกัน
function delay(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}

async function demonstrateChain() {
  console.log('Chain start');

  const result = await Promise.resolve('value')
    .then(v => {
      console.log('Step 1:', v);
      return v + '->2';
    })
    .then(v => {
      console.log('Step 2:', v);
      return v + '->3';
    })
    .then(v => {
      console.log('Step 3:', v);
      return v;
    });

  console.log('Chain done:', result);
}

demonstrateChain();
console.log('After chain call');
// Chain start
// After chain call
// Step 1: value
// Step 2: value->2
// Step 3: value->2->3
// Chain done: value->2->3
```

---

## Step 680: queueMicrotask()

```javascript
// queueMicrotask ส่งงานเข้า microtask queue โดยตรง
console.log('Sync 1');

queueMicrotask(() => {
  console.log('Microtask 1');
});

Promise.resolve().then(() => {
  console.log('Promise 1');
});

queueMicrotask(() => {
  console.log('Microtask 2');
});

console.log('Sync 2');

// Output:
// Sync 1
// Sync 2
// Microtask 1  <- queueMicrotask ก่อน Promise (order of scheduling)
// Promise 1
// Microtask 2
```

```javascript
// ประโยชน์ของ queueMicrotask
// 1. ใช้ batch updates เพื่อ performance
class BatchUpdater {
  #pending = new Set();
  #scheduled = false;

  schedule(item) {
    this.#pending.add(item);

    if (!this.#scheduled) {
      this.#scheduled = true;
      queueMicrotask(() => this.#flush());
    }
  }

  #flush() {
    const toProcess = [...this.#pending];
    this.#pending.clear();
    this.#scheduled = false;

    console.log('Processing batch:', toProcess);
    toProcess.forEach(item => this.#process(item));
  }

  #process(item) {
    console.log('Processed:', item);
  }
}

const updater = new BatchUpdater();
updater.schedule('item1'); // ไม่ process ทันที
updater.schedule('item2'); // ไม่ process ทันที
updater.schedule('item3'); // ไม่ process ทันที
// ทั้งหมด process พร้อมกันใน microtask เดียว!
```

---

## Step 681: requestAnimationFrame

```javascript
// requestAnimationFrame (Browser only) - sync กับ browser refresh
// ใช้สำหรับ animations

function animate(element, from, to, duration) {
  const start = performance.now();

  function update(currentTime) {
    const elapsed = currentTime - start;
    const progress = Math.min(elapsed / duration, 1);
    const eased = easeInOut(progress);
    const value = from + (to - from) * eased;

    // update element (จำลอง)
    console.log(`Position: ${value.toFixed(2)}`);

    if (progress < 1) {
      requestAnimationFrame(update);
    } else {
      console.log('Animation complete!');
    }
  }

  requestAnimationFrame(update);
}

function easeInOut(t) {
  return t < 0.5 ? 2 * t * t : -1 + (4 - 2 * t) * t;
}

// Game loop
function createGameLoop(update, render) {
  let lastTime;
  let running = false;
  let rafId;

  function loop(currentTime) {
    if (!lastTime) lastTime = currentTime;
    const deltaTime = currentTime - lastTime;
    lastTime = currentTime;

    update(deltaTime);
    render();

    if (running) {
      rafId = requestAnimationFrame(loop);
    }
  }

  return {
    start() {
      running = true;
      lastTime = undefined;
      rafId = requestAnimationFrame(loop);
    },
    stop() {
      running = false;
      cancelAnimationFrame(rafId);
    }
  };
}
```

---

## Step 682: Callback Patterns

```javascript
// Error-first callbacks (Node.js style)
function readFileSimulated(filename, callback) {
  setTimeout(() => {
    if (!filename) {
      callback(new Error('ไม่ได้ระบุชื่อไฟล์'));
      return;
    }
    callback(null, `เนื้อหาของ ${filename}`);
  }, 100);
}

// ใช้งาน
readFileSimulated('data.txt', (err, data) => {
  if (err) {
    console.error('ผิดพลาด:', err.message);
    return;
  }
  console.log('ข้อมูล:', data);
});

// Promisify: แปลง callback เป็น Promise
function promisify(fn) {
  return function(...args) {
    return new Promise((resolve, reject) => {
      fn(...args, (err, ...results) => {
        if (err) {
          reject(err);
        } else {
          resolve(results.length === 1 ? results[0] : results);
        }
      });
    });
  };
}

const readFile = promisify(readFileSimulated);

readFile('config.json')
  .then(data => console.log('Promise data:', data))
  .catch(err => console.error('Promise error:', err.message));
```

```javascript
// Callback composition
function waterfall(tasks, callback) {
  function runNext(index, prevResult) {
    if (index >= tasks.length) {
      callback(null, prevResult);
      return;
    }

    tasks[index](prevResult, (err, result) => {
      if (err) {
        callback(err);
        return;
      }
      runNext(index + 1, result);
    });
  }

  runNext(0, undefined);
}

// ตัวอย่าง
const tasks = [
  (_, cb) => {
    setTimeout(() => {
      console.log('Task 1: Fetch user');
      cb(null, { id: 1, name: 'อลิส' });
    }, 50);
  },
  (user, cb) => {
    setTimeout(() => {
      console.log('Task 2: Fetch orders for', user.name);
      cb(null, { user, orders: ['order1', 'order2'] });
    }, 50);
  },
  (data, cb) => {
    setTimeout(() => {
      console.log('Task 3: Process orders');
      cb(null, { ...data, processed: true });
    }, 50);
  }
];

waterfall(tasks, (err, result) => {
  if (err) return console.error(err);
  console.log('Final result:', result);
});
```

---

## Step 683: Callback Hell และ Solutions

```javascript
// Callback Hell (Pyramid of doom)
function callbackHell() {
  login('user@example.com', 'password', (err, user) => {
    if (err) { console.error(err); return; }

    fetchProfile(user.id, (err, profile) => {
      if (err) { console.error(err); return; }

      fetchOrders(profile.id, (err, orders) => {
        if (err) { console.error(err); return; }

        fetchOrderDetails(orders[0].id, (err, details) => {
          if (err) { console.error(err); return; }

          calculateTotal(details, (err, total) => {
            if (err) { console.error(err); return; }

            console.log('Total:', total);
            // ยิ่งลึกยิ่งอ่านยาก!
          });
        });
      });
    });
  });
}
```

```javascript
// วิธีแก้ 1: Named functions
function handleError(err) {
  if (err) { console.error(err); return true; }
  return false;
}

function onLoginComplete(err, user) {
  if (handleError(err)) return;
  fetchProfile(user.id, onProfileComplete);
}

function onProfileComplete(err, profile) {
  if (handleError(err)) return;
  fetchOrders(profile.id, onOrdersComplete);
}

// วิธีแก้ 2: Promises
function withPromises() {
  login('user@example.com', 'password')
    .then(user => fetchProfile(user.id))
    .then(profile => fetchOrders(profile.id))
    .then(orders => fetchOrderDetails(orders[0].id))
    .then(details => calculateTotal(details))
    .then(total => console.log('Total:', total))
    .catch(err => console.error(err));
}

// วิธีแก้ 3: async/await (ดีที่สุด)
async function withAsync() {
  try {
    const user    = await login('user@example.com', 'password');
    const profile = await fetchProfile(user.id);
    const orders  = await fetchOrders(profile.id);
    const details = await fetchOrderDetails(orders[0].id);
    const total   = await calculateTotal(details);
    console.log('Total:', total);
  } catch (err) {
    console.error(err);
  }
}
```

---

## Step 684: Node.js Event Loop Phases

```javascript
// Node.js Event Loop มี 6 phases:
// 1. timers: setTimeout, setInterval callbacks
// 2. pending callbacks: I/O callbacks ที่รอจาก previous iteration
// 3. idle, prepare: ใช้ internally
// 4. poll: รอ I/O events, รัน I/O callbacks
// 5. check: setImmediate callbacks
// 6. close callbacks: socket.on('close')
// ระหว่างแต่ละ phase: microtasks queue ถูกล้าง

// ตัวอย่าง Node.js event loop
if (typeof process !== 'undefined') { // ตรวจว่าเป็น Node.js
  const fs = require('fs');

  console.log('1: Start');

  setTimeout(() => console.log('7: setTimeout'), 0);

  setImmediate(() => console.log('6: setImmediate'));

  process.nextTick(() => console.log('3: nextTick 1'));

  Promise.resolve().then(() => console.log('4: Promise 1'));

  fs.readFile(__filename, () => {
    console.log('8: I/O callback');
    setTimeout(() => console.log('10: setTimeout in I/O'), 0);
    setImmediate(() => console.log('9: setImmediate in I/O'));
    process.nextTick(() => console.log('Inside I/O nextTick'));
  });

  process.nextTick(() => console.log('5: nextTick 2'));

  console.log('2: End');

  // Output (Node.js):
  // 1: Start
  // 2: End
  // 3: nextTick 1        <- process.nextTick (highest priority microtask)
  // 5: nextTick 2        <- process.nextTick
  // 4: Promise 1         <- Promise microtask
  // 7: setTimeout        <- timers phase
  // 6: setImmediate      <- check phase
  // 8: I/O callback      <- poll phase
  // Inside I/O nextTick  <- nextTick (ก่อน setImmediate)
  // 9: setImmediate in I/O <- check phase
  // 10: setTimeout in I/O  <- timers phase (next iteration)
}
```

---

## Step 685: setImmediate (Node.js)

```javascript
// setImmediate vs setTimeout(fn, 0)
// setImmediate: ทำงานใน check phase (หลัง poll phase)
// setTimeout(fn, 0): ทำงานใน timers phase

if (typeof setImmediate !== 'undefined') { // Node.js only
  // ใน I/O callback: setImmediate มาก่อน setTimeout
  const fs = require('fs');
  fs.readFile('/some/file', () => {
    setImmediate(() => console.log('setImmediate in I/O'));
    setTimeout(() => console.log('setTimeout in I/O'), 0);
    // Output: setImmediate ก่อน setTimeout
  });

  // นอก I/O callback: ไม่ได้ predict ได้
  setTimeout(() => console.log('setTimeout'), 0);
  setImmediate(() => console.log('setImmediate'));
  // Order ไม่แน่นอน ขึ้นกับ performance ของ system

  // ใช้ setImmediate สำหรับ:
  function processLargeArray(arr, callback) {
    let index = 0;

    function processChunk() {
      const end = Math.min(index + 100, arr.length);

      while (index < end) {
        // process arr[index]
        index++;
      }

      if (index < arr.length) {
        setImmediate(processChunk); // ไม่บล็อก event loop
      } else {
        callback(null, 'Done');
      }
    }

    processChunk();
  }
}
```

---

## Step 686: process.nextTick (Node.js)

```javascript
// process.nextTick: runs BEFORE other microtasks!
// มีความสำคัญสูงสุด (higher than Promises)

if (typeof process !== 'undefined') {
  console.log('sync 1');

  process.nextTick(() => {
    console.log('nextTick 1');
    process.nextTick(() => console.log('nested nextTick'));
  });

  Promise.resolve().then(() => console.log('Promise 1'));

  process.nextTick(() => console.log('nextTick 2'));

  console.log('sync 2');

  // Output:
  // sync 1
  // sync 2
  // nextTick 1       <- nextTick ก่อน Promise
  // nested nextTick  <- nested nextTick ยังอยู่ใน nextTick queue
  // nextTick 2
  // Promise 1

  // ใช้ process.nextTick สำหรับ:
  // 1. Error handling ให้ consistent
  class EventEmitter {
    on(event, listener) { /* ... */ }

    emit(event, ...args) {
      // emit ใน nextTick เพื่อให้ caller สามารถ add more listeners
      process.nextTick(() => {
        // ... call listeners
      });
    }
  }

  // 2. ให้ user code ทำงานก่อน internal logic
  function AsyncTask() {
    process.nextTick(() => {
      this.emit('ready'); // emit หลังจาก constructor ทำงานเสร็จ
    });
  }
}
```

---

## Step 687: Performance Implications

```javascript
// Long-running tasks บล็อก Event Loop
// ตัวอย่างที่ไม่ดี
function blockingOperation() {
  const start = Date.now();
  while (Date.now() - start < 1000) {
    // บล็อก event loop 1 วินาที!
    // ระหว่างนี้ ไม่มี event ไหนทำงานได้
  }
}

// ตัวอย่างที่ดีกว่า: แบ่งงานออกเป็นชิ้นเล็กๆ
async function nonBlockingOperation(items) {
  const CHUNK_SIZE = 100;

  for (let i = 0; i < items.length; i += CHUNK_SIZE) {
    const chunk = items.slice(i, i + CHUNK_SIZE);

    // process chunk
    chunk.forEach(item => {
      // process item
    });

    // ให้ event loop ทำงานได้ระหว่างชิ้น
    await new Promise(resolve => setTimeout(resolve, 0));
  }
}

// Worker Threads (Node.js) สำหรับ CPU-intensive tasks
/*
const { Worker, isMainThread, parentPort } = require('worker_threads');

if (isMainThread) {
  const worker = new Worker(__filename);
  worker.on('message', result => console.log('Result:', result));
  worker.postMessage({ data: [1, 2, 3, 4, 5] });
} else {
  parentPort.on('message', ({ data }) => {
    // Heavy computation ใน worker thread
    const result = data.reduce((a, b) => a + b, 0);
    parentPort.postMessage(result);
  });
}
*/
```

```javascript
// Measuring Event Loop lag
function measureEventLoopLag() {
  const start = process.hrtime.bigint();
  setImmediate(() => {
    const lag = Number(process.hrtime.bigint() - start) / 1e6; // ms
    console.log(`Event Loop Lag: ${lag.toFixed(2)}ms`);
  });
}

// ถ้า lag สูง (>10ms) แสดงว่ามีงานหนักบล็อก event loop

// ตัวอย่าง: Monitoring event loop
function createEventLoopMonitor(intervalMs = 1000) {
  let lastCheck = Date.now();

  const id = setInterval(() => {
    const now = Date.now();
    const lag = now - lastCheck - intervalMs;
    if (lag > 10) {
      console.warn(`High Event Loop Lag: ${lag}ms`);
    }
    lastCheck = now;
  }, intervalMs);

  return () => clearInterval(id);
}
```

---

## Step 688: Async Patterns และ Best Practices

```javascript
// Pattern 1: Parallel execution
async function parallelFetch() {
  const [users, posts, comments] = await Promise.all([
    fetch('/api/users').then(r => r.json()),
    fetch('/api/posts').then(r => r.json()),
    fetch('/api/comments').then(r => r.json())
  ]);

  return { users, posts, comments };
}

// Pattern 2: Sequential vs Parallel
async function sequential(ids) {
  const results = [];
  for (const id of ids) {
    const result = await fetch(`/api/item/${id}`).then(r => r.json());
    results.push(result);
  }
  return results;
}

async function parallel(ids) {
  return Promise.all(ids.map(id =>
    fetch(`/api/item/${id}`).then(r => r.json())
  ));
}

// Pattern 3: Rate-limited parallel
async function rateLimitedParallel(items, limit, fn) {
  const results = [];
  let active = 0;
  let index = 0;

  return new Promise((resolve, reject) => {
    function next() {
      while (active < limit && index < items.length) {
        const i = index++;
        active++;

        fn(items[i]).then(result => {
          results[i] = result;
          active--;
          if (index >= items.length && active === 0) {
            resolve(results);
          } else {
            next();
          }
        }).catch(reject);
      }
    }

    next();
  });
}

// ตัวอย่าง
async function fetchWithRateLimit(urls) {
  return rateLimitedParallel(urls, 3, url =>
    fetch(url).then(r => r.json())
  );
}
```

---

## Step 689: Real-World Async Examples

```javascript
// Retry with exponential backoff
async function withRetry(fn, options = {}) {
  const {
    maxRetries = 3,
    baseDelay = 1000,
    maxDelay = 30000,
    factor = 2,
    onRetry = () => {}
  } = options;

  let lastError;

  for (let attempt = 0; attempt <= maxRetries; attempt++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error;

      if (attempt === maxRetries) break;

      const delay = Math.min(
        baseDelay * factor ** attempt + Math.random() * 1000,
        maxDelay
      );

      console.log(`Attempt ${attempt + 1} failed. Retrying in ${delay.toFixed(0)}ms...`);
      onRetry(error, attempt + 1, delay);

      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }

  throw lastError;
}

// ใช้งาน
async function fetchData(url) {
  return withRetry(
    () => fetch(url).then(r => {
      if (!r.ok) throw new Error(`HTTP ${r.status}`);
      return r.json();
    }),
    {
      maxRetries: 3,
      baseDelay: 1000,
      onRetry: (err, attempt) => console.log(`Retry ${attempt}: ${err.message}`)
    }
  );
}
```

```javascript
// Event-driven architecture
class EventQueue {
  #queue = [];
  #handlers = new Map();
  #processing = false;

  emit(eventType, payload) {
    this.#queue.push({ eventType, payload, timestamp: Date.now() });
    this.#processQueue();
  }

  on(eventType, handler) {
    if (!this.#handlers.has(eventType)) {
      this.#handlers.set(eventType, []);
    }
    this.#handlers.get(eventType).push(handler);
    return this;
  }

  async #processQueue() {
    if (this.#processing) return;
    this.#processing = true;

    while (this.#queue.length > 0) {
      const event = this.#queue.shift();
      await this.#handle(event);
    }

    this.#processing = false;
  }

  async #handle({ eventType, payload }) {
    const handlers = this.#handlers.get(eventType) || [];

    for (const handler of handlers) {
      try {
        await handler(payload);
      } catch (err) {
        console.error(`Handler error for ${eventType}:`, err);
      }
    }
  }
}

const eventQueue = new EventQueue();

eventQueue
  .on('order:created', async order => {
    console.log('Processing order:', order.id);
    await new Promise(r => setTimeout(r, 100)); // จำลอง async work
    console.log('Order processed:', order.id);
  })
  .on('order:created', async order => {
    console.log('Sending confirmation email for order:', order.id);
  })
  .on('order:paid', async order => {
    console.log('Fulfilling order:', order.id);
  });

eventQueue.emit('order:created', { id: 'ORD-001', total: 1500 });
eventQueue.emit('order:paid', { id: 'ORD-001' });
```

---

## Step 690: สรุปและ Advanced Concepts

```javascript
// Promise.race และ Promise.any
async function demonstratePromiseVariants() {
  const fast = new Promise(resolve => setTimeout(() => resolve('fast'), 100));
  const slow = new Promise(resolve => setTimeout(() => resolve('slow'), 500));
  const fail = new Promise((_, reject) => setTimeout(() => reject(new Error('fail')), 200));

  // race: win แรกที่ settle (resolve หรือ reject)
  const raceResult = await Promise.race([slow, fast, fail]);
  console.log('Race:', raceResult); // 'fast'

  // any: win แรกที่ resolve (ไม่สนใจ reject)
  const anyResult = await Promise.any([fail, slow, fast]);
  console.log('Any:', anyResult); // 'fast'

  // all: รอทุกตัว resolve (ล้มเหลวถ้ามีตัวเดียว reject)
  try {
    await Promise.all([fast, fail, slow]);
  } catch (e) {
    console.log('All failed:', e.message); // 'fail'
  }

  // allSettled: รอทุกตัว settle (ไม่ throw)
  const settled = await Promise.allSettled([fast, fail, slow]);
  settled.forEach(s => {
    if (s.status === 'fulfilled') console.log('✓', s.value);
    else console.log('✗', s.reason.message);
  });
}

demonstratePromiseVariants();
```

```javascript
// AbortController - cancel async operations
function fetchWithAbort(url, signal) {
  return fetch(url, { signal })
    .then(r => r.json())
    .catch(err => {
      if (err.name === 'AbortError') {
        console.log('Fetch cancelled');
        return null;
      }
      throw err;
    });
}

const controller = new AbortController();

// ตัวอย่าง: cancel หลัง 2 วินาที
setTimeout(() => controller.abort(), 2000);

fetchWithAbort('/api/slow-endpoint', controller.signal)
  .then(data => {
    if (data) console.log('Got data:', data);
  });

// AbortController ใน React-like pattern
function useSearch(query) {
  let controller;

  async function search(q) {
    if (controller) controller.abort(); // cancel previous
    controller = new AbortController();

    try {
      const results = await fetchWithAbort(
        `/api/search?q=${encodeURIComponent(q)}`,
        controller.signal
      );
      return results;
    } catch (err) {
      if (err.name !== 'AbortError') throw err;
    }
  }

  return { search, abort: () => controller?.abort() };
}
```

```javascript
// Async Generator - สร้าง async stream
async function* paginate(url) {
  let page = 1;
  let hasMore = true;

  while (hasMore) {
    const response = await fetch(`${url}?page=${page}&limit=10`)
      .then(r => r.json());

    yield response.data;

    hasMore = response.hasNextPage;
    page++;
  }
}

// ใช้งาน
async function processAllPages() {
  const stream = paginate('/api/users');

  for await (const batch of stream) {
    console.log(`Processing ${batch.length} users...`);
    batch.forEach(user => console.log(' -', user.name));
  }
}

// Async iteration ด้วย for await...of
async function* numberStream(n) {
  for (let i = 0; i < n; i++) {
    await new Promise(r => setTimeout(r, 100));
    yield i;
  }
}

async function main() {
  for await (const num of numberStream(5)) {
    console.log('Received:', num);
  }
  console.log('Stream complete');
}

// main();
```

---

## แบบฝึกหัด (Exercises)

### แบบฝึกหัดที่ 1: Event Loop Quiz
```javascript
// ทำนาย output ก่อนรัน:

console.log('A');

setTimeout(() => console.log('B'), 0);
setTimeout(() => console.log('C'), 100);

Promise.resolve()
  .then(() => {
    console.log('D');
    return Promise.resolve();
  })
  .then(() => console.log('E'));

queueMicrotask(() => console.log('F'));

console.log('G');

// คำตอบ: A, G, D, F, E, B, C
```

### แบบฝึกหัดที่ 2: Async Task Manager
```javascript
// สร้าง TaskManager ที่:
// 1. รัน tasks ด้วย concurrency limit
// 2. มี priority queue
// 3. retry on failure
// 4. timeout per task
// 5. progress reporting
// 6. cancel individual tasks หรือ all tasks

class TaskManager {
  constructor({ concurrency = 3, defaultTimeout = 30000 } = {}) {
    // TODO
  }

  add(task, { priority = 0, timeout, retries = 0 } = {}) {
    // TODO
    // return Promise ที่ resolve เมื่อ task เสร็จ
  }

  cancel(taskId) { /* TODO */ }
  cancelAll() { /* TODO */ }
  
  get stats() {
    // return { pending, running, completed, failed }
  }
}
```

### แบบฝึกหัดที่ 3: Simple Scheduler
```javascript
// สร้าง scheduler ที่:
// 1. schedule tasks ที่เวลาเฉพาะ
// 2. รอง recurring tasks (cron-like)
// 3. cancel scheduled tasks
// 4. ทำงาน even หลัง missed schedules

class Scheduler {
  // TODO
  schedule(fn, when) { /* date or cron */ }
  every(fn, interval) { /* recurring */ }
  cancel(id) { /* TODO */ }
}
```

### แบบฝึกหัดที่ 4: Observable Stream
```javascript
// สร้าง Observable ที่:
// 1. emit values over time
// 2. operator: map, filter, take, skip, debounce, throttle
// 3. combine: merge, zip, switchMap
// 4. error handling: catch, retry
// 5. subscribe/unsubscribe

class Observable {
  constructor(subscriber) {
    this._subscriber = subscriber;
  }

  static interval(ms) { /* TODO */ }
  static fromEvent(element, event) { /* TODO */ }
  static fromPromise(promise) { /* TODO */ }

  subscribe(onNext, onError, onComplete) { /* TODO */ }

  map(fn) { /* TODO */ }
  filter(pred) { /* TODO */ }
  take(n) { /* TODO */ }
  // ... more operators
}
```

---

## สรุป

ใน Part 35 เราได้เรียนรู้:

1. **JavaScript Runtime**: Call Stack, Heap, Web APIs
2. **Call Stack**: LIFO structure สำหรับ function calls
3. **Web APIs**: timer, network, DOM events
4. **Macrotask Queue**: setTimeout, setInterval, I/O
5. **Microtask Queue**: Promises, queueMicrotask (สำคัญกว่า macrotask)
6. **Event Loop**: กลไกที่ประสาน synchronous และ asynchronous
7. **setTimeout/setInterval**: timer APIs
8. **clearTimeout/clearInterval**: cleanup
9. **Promise Microtasks**: ทำงานก่อน macrotasks
10. **queueMicrotask**: เพิ่ม microtask โดยตรง
11. **requestAnimationFrame**: สำหรับ animations
12. **Callback Patterns**: error-first, promisify
13. **Callback Hell**: ปัญหาและวิธีแก้
14. **Node.js Event Loop**: 6 phases
15. **setImmediate**: check phase ใน Node.js
16. **process.nextTick**: highest priority microtask
17. **Performance**: หลีกเลี่ยง blocking event loop
18. **Async Patterns**: retry, parallel, rate-limit
19. **AbortController**: cancel async operations
20. **Async Generators**: async streams

ยินดีด้วย! คุณได้เรียนจบ Part 35 แล้ว ต่อไปคือ Part 36: Promises!
