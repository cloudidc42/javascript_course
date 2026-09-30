# Part 33: Closures (Steps 631-650)

## บทนำ

Closure เป็นหนึ่งในคุณสมบัติที่ทรงพลังที่สุดของ JavaScript ซึ่งทำให้ function สามารถ "จำ" และเข้าถึงตัวแปรจาก scope ที่มันถูกสร้างขึ้น แม้ว่า scope นั้นจะ "หมดอายุ" ไปแล้วก็ตาม Closures ใช้ในการสร้าง private state, module pattern, และ functional programming patterns ต่างๆ

---

## Step 631: What is a Closure?

```javascript
// Closure พื้นฐาน
function outer() {
  const message = 'สวัสดี'; // variable ใน outer scope

  function inner() {
    // inner function สามารถเข้าถึง message ได้
    // แม้ว่าจะ return จาก outer ไปแล้ว
    console.log(message);
  }

  return inner; // return function (ไม่ใช่ execution)
}

const greet = outer(); // outer ทำงานและ return inner
// ตอนนี้ outer scope "หมดแล้ว" แต่...
greet(); // สวัสดี ← inner ยังเข้าถึง message ได้!

// นี่คือ Closure:
// inner function "ปิด" (closes over) ตัวแปรจาก outer scope
```

```javascript
// Closure จำค่าของตัวแปร (ไม่ใช่ copy)
function makeCounter() {
  let count = 0; // ตัวแปรที่ closure จะ "close over"

  return {
    increment() { count++; },
    decrement() { count--; },
    getCount() { return count; }
  };
}

const counter = makeCounter();
counter.increment();
counter.increment();
counter.increment();
counter.decrement();
console.log(counter.getCount()); // 2

// แต่ละ counter มี count เป็นของตัวเอง
const counter2 = makeCounter();
counter2.increment();
console.log(counter2.getCount()); // 1
console.log(counter.getCount());  // 2 - ไม่ได้รับผลกระทบ!
```

---

## Step 632: Lexical Scoping

Lexical Scoping หมายความว่า scope ถูกกำหนดโดยที่ function ถูกเขียน ไม่ใช่ที่ถูกเรียก

```javascript
// Lexical scope ถูกกำหนดตอน write-time ไม่ใช่ run-time
const x = 'global';

function outer() {
  const x = 'outer';

  function inner() {
    const x = 'inner';
    console.log(x); // 'inner' - ใกล้ที่สุด
  }

  function middle() {
    console.log(x); // 'outer' - จาก lexical scope ของ middle
  }

  inner();
  middle();
}

outer();
// 'inner'
// 'outer'

// Dynamic scope (ไม่ใช่ JS) vs Lexical scope (JS)
function printX() {
  console.log(x); // ใน JS จะอ่าน x จาก lexical scope (global)
}

function callPrintX() {
  const x = 'local'; // นี่ไม่มีผลต่อ printX
  printX();
}

callPrintX(); // 'global' ← lexical scope ของ printX คือ global
```

```javascript
// Closure เก็บ reference ไปยัง scope ทั้งหมด
function createMultiplier(factor) {
  // factor อยู่ใน scope ของ createMultiplier
  return function(number) {
    // inner function ใช้ factor จาก outer scope
    return number * factor;
  };
}

const double = createMultiplier(2);
const triple = createMultiplier(3);
const tenTimes = createMultiplier(10);

console.log(double(5));   // 10
console.log(triple(5));   // 15
console.log(tenTimes(5)); // 50

// แต่ละ function จำ factor ของตัวเอง
console.log(double(100));  // 200
console.log(triple(100));  // 300
```

---

## Step 633: How Closures Work

```javascript
// Closure สร้างอย่างไร - ดูจาก JavaScript engine perspective

function bankAccount(initialBalance) {
  // Environment ที่สร้างขึ้นเมื่อ bankAccount ถูกเรียก
  // บันทึก: initialBalance = 1000

  let balance = initialBalance;
  // บันทึก: balance = 1000

  // แต่ละ method ที่ return กลับไปจะ "capture" environment นี้
  return {
    deposit(amount) {
      balance += amount; // อ้างอิง balance จาก environment
      return balance;
    },
    withdraw(amount) {
      if (amount > balance) throw new Error('ยอดไม่พอ');
      balance -= amount; // อ้างอิง balance จาก environment เดียวกัน
      return balance;
    },
    getBalance() {
      return balance; // อ้างอิง balance
    }
  };
  // bankAccount scope จะถูกลบถ้าไม่มี closure
  // แต่เนื่องจาก returned methods ยัง reference balance
  // JavaScript engine จะเก็บ environment ไว้ใน memory
}

const acct = bankAccount(1000);
console.log(acct.getBalance()); // 1000
acct.deposit(500);
console.log(acct.getBalance()); // 1500
acct.withdraw(200);
console.log(acct.getBalance()); // 1300
```

```javascript
// Closure เก็บ live reference (ไม่ใช่ snapshot)
function makeSharedCounter() {
  let count = 0;

  const increment = () => ++count;
  const decrement = () => --count;
  const getValue  = () => count;

  // ทั้งสามฟังก์ชันใช้ count เดียวกัน!
  return { increment, decrement, getValue };
}

const { increment, decrement, getValue } = makeSharedCounter();

increment(); // count = 1
increment(); // count = 2
increment(); // count = 3
decrement(); // count = 2
console.log(getValue()); // 2 - ทุก function เห็น count เดียวกัน
```

---

## Step 634: Practical Closure Examples

```javascript
// 1. Greeting factory
function greetingFactory(greeting) {
  return function(name) {
    return `${greeting}, ${name}!`;
  };
}

const sayHello   = greetingFactory('สวัสดี');
const sayGoodbye = greetingFactory('ลาก่อน');
const sayHowdy   = greetingFactory('หวัดดี');

console.log(sayHello('อลิส'));   // สวัสดี, อลิส!
console.log(sayGoodbye('บ็อบ')); // ลาก่อน, บ็อบ!
console.log(sayHowdy('ชาร์ลี')); // หวัดดี, ชาร์ลี!
```

```javascript
// 2. Configuration with closure
function createHttpClient(baseUrl, defaultOptions = {}) {
  const base = baseUrl.replace(/\/$/, '');
  const options = { ...defaultOptions };

  async function request(path, method, body, extraOptions = {}) {
    const url = `${base}${path}`;
    const config = {
      method,
      headers: { 'Content-Type': 'application/json', ...options.headers, ...extraOptions.headers },
      ...options,
      ...extraOptions
    };

    if (body) config.body = JSON.stringify(body);

    const response = await fetch(url, config);
    if (!response.ok) throw new Error(`HTTP ${response.status}`);
    return response.json();
  }

  return {
    get: (path, opts) => request(path, 'GET', null, opts),
    post: (path, body, opts) => request(path, 'POST', body, opts),
    put: (path, body, opts) => request(path, 'PUT', body, opts),
    delete: (path, opts) => request(path, 'DELETE', null, opts),
    setHeader(key, value) {
      options.headers = { ...options.headers, [key]: value };
    }
  };
}

const api = createHttpClient('https://api.example.com', {
  headers: { 'Authorization': 'Bearer token123' }
});

// api.get('/users')
// api.post('/users', { name: 'อลิส' })
```

```javascript
// 3. Event handler with context
function createButton(label, handler) {
  const clickCount = { value: 0 };

  return {
    click() {
      clickCount.value++;
      console.log(`"${label}" ถูกคลิก ${clickCount.value} ครั้ง`);
      handler(label, clickCount.value);
    },
    getClickCount() {
      return clickCount.value;
    }
  };
}

const btn = createButton('Submit', (label, count) => {
  console.log(`Handler: ${label} clicked ${count} times`);
});

btn.click(); // "Submit" ถูกคลิก 1 ครั้ง + Handler: Submit clicked 1 times
btn.click(); // "Submit" ถูกคลิก 2 ครั้ง + ...
```

---

## Step 635: Counter using Closure

```javascript
// Counter แบบต่างๆ
function makeCounter(start = 0, step = 1) {
  let current = start;

  return {
    next() {
      const value = current;
      current += step;
      return value;
    },
    prev() {
      current -= step;
      return current;
    },
    reset() {
      current = start;
      return this;
    },
    peek() {
      return current;
    },
    // Iterator protocol
    [Symbol.iterator]() {
      return {
        next: () => ({ value: this.next(), done: false })
      };
    }
  };
}

const counter = makeCounter(0, 2); // เริ่มที่ 0 เพิ่มทีละ 2
console.log(counter.next()); // 0
console.log(counter.next()); // 2
console.log(counter.next()); // 4
console.log(counter.peek()); // 6 (ยังไม่ได้เพิ่ม)
counter.reset();
console.log(counter.peek()); // 0
```

```javascript
// ID Generator
function createIdGenerator(prefix = '') {
  let lastId = 0;
  const generated = new Set();

  return {
    next() {
      const id = `${prefix}${++lastId}`;
      generated.add(id);
      return id;
    },
    peek() {
      return `${prefix}${lastId + 1}`;
    },
    isValid(id) {
      return generated.has(id);
    },
    get count() {
      return lastId;
    }
  };
}

const userId = createIdGenerator('USER-');
const orderId = createIdGenerator('ORD-');

console.log(userId.next());  // USER-1
console.log(userId.next());  // USER-2
console.log(orderId.next()); // ORD-1
console.log(userId.next());  // USER-3

console.log(userId.isValid('USER-2')); // true
console.log(userId.isValid('USER-5')); // false
console.log(userId.count); // 3
```

---

## Step 636: Private State with Closures

```javascript
// Closure ใช้แทน private fields
function createPerson(name, age) {
  // ตัวแปรเหล่านี้ "private" - ไม่สามารถเข้าถึงจากภายนอกได้
  let _name = name;
  let _age = age;
  const _history = [];

  function validateAge(value) {
    if (typeof value !== 'number' || value < 0 || value > 150) {
      throw new Error('อายุไม่ถูกต้อง');
    }
  }

  // ส่วน public interface
  return {
    get name() { return _name; },
    set name(value) {
      if (!value.trim()) throw new Error('ชื่อต้องไม่ว่าง');
      _name = value.trim();
    },

    get age() { return _age; },
    set age(value) {
      validateAge(value);
      _history.push({ field: 'age', old: _age, new: value });
      _age = value;
    },

    haveBirthday() {
      this.age = _age + 1;
      return this;
    },

    getHistory() {
      return [..._history];
    },

    toString() {
      return `${_name} (${_age})`;
    }
  };
}

const person = createPerson('อลิส', 25);
console.log(person.name); // อลิส
person.age = 26;
person.haveBirthday();
console.log(`${person}`); // อลิส (27)
console.log(person.getHistory()); // [{field: 'age', old: 25, new: 26}, ...]

// ไม่สามารถเข้าถึง _name, _age, _history โดยตรง
// person._name // undefined (ไม่มี property นี้)
```

```javascript
// Secure vault
function createVault(masterPassword) {
  const secrets = new Map();
  let attempts = 0;
  const MAX_ATTEMPTS = 3;
  let locked = false;

  function authenticate(password) {
    if (locked) throw new Error('Vault ถูกล็อค');
    if (password !== masterPassword) {
      attempts++;
      if (attempts >= MAX_ATTEMPTS) {
        locked = true;
        throw new Error('Vault ถูกล็อคเนื่องจากพยายามหลายครั้ง');
      }
      throw new Error(`รหัสผ่านผิด (${MAX_ATTEMPTS - attempts} ครั้งที่เหลือ)`);
    }
    attempts = 0; // reset on success
  }

  return {
    store(key, value, password) {
      authenticate(password);
      secrets.set(key, value);
    },
    retrieve(key, password) {
      authenticate(password);
      return secrets.get(key);
    },
    isLocked() {
      return locked;
    }
  };
}

const vault = createVault('secret123');
vault.store('apiKey', 'abc-123-xyz', 'secret123');
console.log(vault.retrieve('apiKey', 'secret123')); // abc-123-xyz

try {
  vault.retrieve('apiKey', 'wrong'); // ผิด!
} catch (e) {
  console.log(e.message); // รหัสผ่านผิด (2 ครั้งที่เหลือ)
}
```

---

## Step 637: Module Pattern using Closures

```javascript
// Module Pattern - IIFE สร้าง private scope
const CounterModule = (function() {
  // Private state
  let count = 0;
  const history = [];

  // Private functions
  function saveHistory(action, value) {
    history.push({ action, value, timestamp: Date.now() });
  }

  // Public API
  return {
    increment(amount = 1) {
      count += amount;
      saveHistory('increment', amount);
      return this;
    },
    decrement(amount = 1) {
      count -= amount;
      saveHistory('decrement', amount);
      return this;
    },
    reset() {
      saveHistory('reset', count);
      count = 0;
      return this;
    },
    get value() { return count; },
    getHistory() { return [...history]; }
  };
})();

CounterModule.increment(5).increment(3).decrement(2);
console.log(CounterModule.value);      // 6
console.log(CounterModule.getHistory()); // [{action: 'increment', value: 5, ...}, ...]

// ไม่มีทางเข้าถึง count หรือ history โดยตรง
```

```javascript
// Revealing Module Pattern
const ShoppingModule = (function() {
  const items = [];
  let discount = 0;

  function addItem(name, price, qty = 1) {
    const existing = items.find(i => i.name === name);
    if (existing) {
      existing.qty += qty;
    } else {
      items.push({ name, price, qty });
    }
  }

  function removeItem(name) {
    const index = items.findIndex(i => i.name === name);
    if (index > -1) items.splice(index, 1);
  }

  function setDiscount(percent) {
    if (percent < 0 || percent > 100) throw new Error('Invalid discount');
    discount = percent;
  }

  function getTotal() {
    const subtotal = items.reduce((sum, item) => sum + item.price * item.qty, 0);
    return subtotal * (1 - discount / 100);
  }

  function getItems() {
    return items.map(i => ({ ...i })); // copy เพื่อป้องกัน mutation
  }

  // Reveal เฉพาะ public API
  return { addItem, removeItem, setDiscount, getTotal, getItems };
})();

ShoppingModule.addItem('แอปเปิล', 30, 5);
ShoppingModule.addItem('กล้วย', 10, 3);
ShoppingModule.setDiscount(10);
console.log(ShoppingModule.getTotal()); // (150 + 30) * 0.9 = 162
```

---

## Step 638: Factory Functions with Closures

```javascript
// Factory function ที่ใช้ closure สร้าง objects ที่มี state
function createAnimal(name, sound, legs) {
  // private state
  let energy = 100;
  let age = 0;

  // private methods
  function consume(amount) {
    energy -= amount;
    if (energy < 0) energy = 0;
  }

  return {
    get name() { return name; },
    get energy() { return energy; },
    get age() { return age; },

    eat(food) {
      const gain = food.length * 5; // แค่ example
      energy = Math.min(100, energy + gain);
      console.log(`${name} กิน ${food} ได้พลังงาน +${gain} (รวม: ${energy})`);
      return this;
    },

    sleep(hours) {
      const gain = hours * 10;
      energy = Math.min(100, energy + gain);
      console.log(`${name} นอน ${hours} ชั่วโมง ได้พลังงาน +${gain}`);
      return this;
    },

    move(distance) {
      const cost = distance * 2;
      consume(cost);
      console.log(`${name} เดิน ${distance} ม. ใช้พลังงาน -${cost} (เหลือ: ${energy})`);
      return this;
    },

    makeSound() {
      console.log(`${name}: ${sound}!`);
      return this;
    },

    birthday() {
      age++;
      console.log(`สุขสันต์วันเกิด ${name}! อายุ ${age} ปี`);
      return this;
    }
  };
}

// สร้าง animals ด้วย factory
const cat = createAnimal('วิสกี้', 'เมี้ยว', 4);
const dog = createAnimal('บัดดี้', 'โฮ่ง', 4);
const bird = createAnimal('ทวีตตี้', 'จิ้บ', 2);

cat.eat('ปลา').sleep(8).makeSound();
dog.move(100).eat('กระดูก').makeSound();

console.log(`${cat.name} มีพลังงาน: ${cat.energy}`);
```

```javascript
// Function factory พร้อม configuration
function createValidator(rules) {
  return function validate(data) {
    const errors = {};

    for (const [field, rule] of Object.entries(rules)) {
      const value = data[field];
      const fieldErrors = [];

      if (rule.required && (value === undefined || value === null || value === '')) {
        fieldErrors.push(`${field} เป็นค่าที่จำเป็น`);
      }

      if (value !== undefined && value !== null) {
        if (rule.type && typeof value !== rule.type) {
          fieldErrors.push(`${field} ต้องเป็น ${rule.type}`);
        }
        if (rule.min !== undefined && value < rule.min) {
          fieldErrors.push(`${field} ต้องมากกว่าหรือเท่ากับ ${rule.min}`);
        }
        if (rule.max !== undefined && value > rule.max) {
          fieldErrors.push(`${field} ต้องน้อยกว่าหรือเท่ากับ ${rule.max}`);
        }
        if (rule.minLength !== undefined && value.length < rule.minLength) {
          fieldErrors.push(`${field} ต้องมีอย่างน้อย ${rule.minLength} ตัว`);
        }
        if (rule.pattern && !rule.pattern.test(value)) {
          fieldErrors.push(`${field} รูปแบบไม่ถูกต้อง`);
        }
        if (rule.custom) {
          const err = rule.custom(value, data);
          if (err) fieldErrors.push(err);
        }
      }

      if (fieldErrors.length > 0) {
        errors[field] = fieldErrors;
      }
    }

    return {
      valid: Object.keys(errors).length === 0,
      errors
    };
  };
}

// สร้าง validator สำหรับ user registration
const validateUser = createValidator({
  name: { required: true, type: 'string', minLength: 2 },
  age: { required: true, type: 'number', min: 0, max: 150 },
  email: {
    required: true,
    pattern: /^[^\s@]+@[^\s@]+\.[^\s@]+$/,
    custom: (email, data) => {
      if (email.includes('spam')) return 'ไม่รับ spam email';
    }
  }
});

console.log(validateUser({ name: 'อ', age: 25, email: 'test@test.com' }));
// { valid: false, errors: { name: ['name ต้องมีอย่างน้อย 2 ตัว'] } }

console.log(validateUser({ name: 'สมชาย', age: 25, email: 'test@test.com' }));
// { valid: true, errors: {} }
```

---

## Step 639: Memoization using Closures

```javascript
// Memoization: cache ผลลัพธ์ของ function เพื่อ performance
function memoize(fn) {
  const cache = new Map();

  return function(...args) {
    const key = JSON.stringify(args);

    if (cache.has(key)) {
      console.log(`[Cache Hit] args: ${key}`);
      return cache.get(key);
    }

    console.log(`[Computing] args: ${key}`);
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}

// ตัวอย่าง: Fibonacci ที่ไม่มี memoization (ช้ามาก)
function fibSlow(n) {
  if (n <= 1) return n;
  return fibSlow(n - 1) + fibSlow(n - 2);
}

// ด้วย memoization
const fib = memoize(function fibonacci(n) {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2); // ต้องเรียกผ่าน fibonacci ที่ memoized
});

// Recursive memoization ที่ถูกต้อง
function createMemoFib() {
  const memo = {};

  return function fib(n) {
    if (n in memo) return memo[n];
    if (n <= 1) return n;
    return (memo[n] = fib(n - 1) + fib(n - 2));
  };
}

const memoFib = createMemoFib();
console.log(memoFib(40)); // 102334155 (เร็วมาก!)
console.log(memoFib(40)); // 102334155 (จาก cache)
```

```javascript
// Memoize แบบ advanced พร้อม TTL (Time To Live)
function memoizeWithTTL(fn, ttlMs = 5000) {
  const cache = new Map();

  return function(...args) {
    const key = JSON.stringify(args);
    const now = Date.now();

    if (cache.has(key)) {
      const { value, timestamp } = cache.get(key);
      if (now - timestamp < ttlMs) {
        return value;
      }
      // expired - ลบออก
      cache.delete(key);
    }

    const result = fn.apply(this, args);
    cache.set(key, { value: result, timestamp: now });
    return result;
  };
}

// Memoize สำหรับ async function
function memoizeAsync(fn) {
  const cache = new Map();
  const pending = new Map();

  return async function(...args) {
    const key = JSON.stringify(args);

    if (cache.has(key)) return cache.get(key);

    // ถ้ากำลัง pending ให้รอ promise เดิม (deduplication)
    if (pending.has(key)) return pending.get(key);

    const promise = fn.apply(this, args).then(result => {
      cache.set(key, result);
      pending.delete(key);
      return result;
    }).catch(err => {
      pending.delete(key);
      throw err;
    });

    pending.set(key, promise);
    return promise;
  };
}
```

---

## Step 640: Partial Application using Closures

```javascript
// Partial Application: กำหนด arguments บางส่วนไว้ก่อน
function partial(fn, ...presetArgs) {
  return function(...laterArgs) {
    return fn(...presetArgs, ...laterArgs);
  };
}

// ตัวอย่าง
function add(a, b, c) {
  return a + b + c;
}

const add10 = partial(add, 10);
const add10and20 = partial(add, 10, 20);

console.log(add10(5, 3));    // 18 (10 + 5 + 3)
console.log(add10and20(7));  // 37 (10 + 20 + 7)

// Real-world example
function logMessage(level, timestamp, message) {
  console.log(`[${level}] ${timestamp}: ${message}`);
}

const logNow = partial(logMessage, 'INFO', new Date().toISOString());
logNow('เริ่มต้นระบบ');
logNow('โหลดข้อมูลแล้ว');
```

```javascript
// Partial Application กับ object methods
function partialMethod(obj, methodName, ...presetArgs) {
  return function(...laterArgs) {
    return obj[methodName](...presetArgs, ...laterArgs);
  };
}

const numbers = [3, 1, 4, 1, 5, 9, 2, 6, 5];
const multiply = (factor, num) => num * factor;

const double = numbers.map.bind(numbers, x => x * 2);
// หรือ
const doubled = numbers.map(n => n * 2);

// ใช้ partial ใน real scenario
function apiRequest(baseUrl, endpoint, method, body) {
  return fetch(`${baseUrl}${endpoint}`, {
    method,
    body: JSON.stringify(body),
    headers: { 'Content-Type': 'application/json' }
  });
}

const myApi = partial(apiRequest, 'https://api.example.com');
const postToMyApi = partial(myApi, '/data', 'POST');

// postToMyApi({ name: 'test' }) = apiRequest('https://api.example.com', '/data', 'POST', { name: 'test' })
```

---

## Step 641: Currying using Closures

```javascript
// Currying: แปลง function ที่รับหลาย args เป็น chain ของ functions ที่รับ arg ทีละตัว
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) {
      // มี args ครบแล้ว - เรียก function จริง
      return fn.apply(this, args);
    }
    // ยังไม่ครบ - return function ที่รอ args เพิ่ม
    return function(...moreArgs) {
      return curried.apply(this, args.concat(moreArgs));
    };
  };
}

// ตัวอย่าง
const curriedAdd = curry((a, b, c) => a + b + c);

// วิธีใช้หลายรูปแบบ
console.log(curriedAdd(1)(2)(3));    // 6
console.log(curriedAdd(1, 2)(3));    // 6
console.log(curriedAdd(1)(2, 3));    // 6
console.log(curriedAdd(1, 2, 3));    // 6

// สร้าง specialized functions
const add1 = curriedAdd(1);
const add1and2 = curriedAdd(1, 2);

console.log(add1(5)(10));  // 16
console.log(add1and2(20)); // 23
```

```javascript
// Currying ใน real-world
const curry2 = fn => a => b => fn(a, b);
const curry3 = fn => a => b => c => fn(a, b, c);

// Filter ที่ curried
const filter = curry2((pred, arr) => arr.filter(pred));
const map    = curry2((fn, arr) => arr.map(fn));
const reduce = curry3((fn, init, arr) => arr.reduce(fn, init));

const isEven = n => n % 2 === 0;
const double = n => n * 2;
const sum    = (acc, n) => acc + n;

const filterEven  = filter(isEven);
const doubleAll   = map(double);
const sumAll      = reduce(sum, 0);

const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

const result = sumAll(doubleAll(filterEven(numbers)));
console.log(result); // (2+4+6+8+10) * 2 = 60

// ใช้ร่วมกับ compose
const compose = (...fns) => x => fns.reduceRight((v, f) => f(v), x);
const pipeline = compose(sumAll, doubleAll, filterEven);
console.log(pipeline(numbers)); // 60
```

---

## Step 642: Closures in Loops (Common Pitfall)

```javascript
// ปัญหาคลาสสิก: closures ใน loop กับ var
console.log('=== ปัญหา ===');
for (var i = 0; i < 3; i++) {
  setTimeout(function() {
    console.log(i); // จะ print 3, 3, 3 ไม่ใช่ 0, 1, 2!
  }, 100 * i);
}
// เพราะ var ไม่มี block scope - ทุก callback share ตัวแปร i เดียวกัน
// เมื่อ callback รัน, i ได้เพิ่มเป็น 3 แล้ว

// วิธีแก้ 1: ใช้ let (block scope)
console.log('=== แก้ด้วย let ===');
for (let i = 0; i < 3; i++) {
  setTimeout(function() {
    console.log(i); // 0, 1, 2 ✓
  }, 100 * i);
}

// วิธีแก้ 2: IIFE เพื่อสร้าง scope ใหม่
console.log('=== แก้ด้วย IIFE ===');
for (var i = 0; i < 3; i++) {
  (function(index) {
    setTimeout(function() {
      console.log(index); // 0, 1, 2 ✓
    }, 100 * index);
  })(i);
}

// วิธีแก้ 3: bind
console.log('=== แก้ด้วย bind ===');
for (var i = 0; i < 3; i++) {
  setTimeout(console.log.bind(null, i), 100 * i); // 0, 1, 2 ✓
}

// วิธีแก้ 4: arrow function พร้อม parameter
console.log('=== แก้ด้วย forEach ===');
[0, 1, 2].forEach(i => {
  setTimeout(() => console.log(i), 100 * i); // 0, 1, 2 ✓
});
```

```javascript
// ตัวอย่างที่ซับซ้อนกว่า
function createButtons() {
  const buttons = [];

  // ปัญหา: ทุก button จะ log 5 (ค่าสุดท้ายของ i)
  for (var i = 0; i < 5; i++) {
    buttons.push({
      label: `Button ${i}`,
      click: function() { console.log(`Clicked button ${i}`); }
    });
  }
  return buttons;
}

// วิธีที่ถูก
function createButtonsFixed() {
  const buttons = [];

  for (let i = 0; i < 5; i++) {
    buttons.push({
      label: `Button ${i}`,
      click: function() { console.log(`Clicked button ${i}`); }
    });
  }
  return buttons;
}

const btns = createButtonsFixed();
btns[2].click(); // Clicked button 2 ✓
btns[4].click(); // Clicked button 4 ✓
```

---

## Step 643: Closures และ Garbage Collection

```javascript
// Closure ป้องกัน garbage collection ของ variables ที่ยัง reference อยู่

// Memory leak ที่อาจเกิด
function potentialLeak() {
  const hugeArray = new Array(1000000).fill('data'); // ข้อมูลใหญ่

  return function() {
    // ถ้า closure นี้ยังมีชีวิต, hugeArray จะไม่ถูก GC
    console.log(hugeArray.length);
  };
}

const leaky = potentialLeak(); // hugeArray ยังอยู่ใน memory!
// leaky = null; // เมื่อ null ออก, hugeArray จะถูก GC

// วิธีหลีกเลี่ยง
function memoryFriendly() {
  const hugeArray = new Array(1000000).fill('data');
  const length = hugeArray.length; // เก็บแค่ที่ต้องการ

  // ไม่ต้อง reference hugeArray ใน closure
  return function() {
    console.log(length); // เก็บแค่ตัวเลข ไม่ใช่ array ทั้งก้อน
  };
}
// hugeArray สามารถถูก GC ได้หลังจาก memoryFriendly() return

// Example: EventEmitter และ cleanup
function createTimer(interval) {
  const data = new Array(100000).fill('timer data'); // ข้อมูลสำคัญ
  let count = 0;

  const id = setInterval(() => {
    count++;
    console.log(`Tick ${count}: data length = ${data.length}`);
  }, interval);

  // ส่งคืน cleanup function
  return {
    stop() {
      clearInterval(id);
      // data จะถูก GC เมื่อ closure ไม่มี reference แล้ว
    },
    getCount() { return count; }
  };
}

const timer = createTimer(100);
// timer.stop(); // cleanup!
```

---

## Step 644: Closures vs Classes for Encapsulation

```javascript
// เปรียบเทียบ Closure approach vs Class approach

// === Closure approach ===
function createStack() {
  const items = []; // truly private

  return {
    push(item) { items.push(item); return this; },
    pop() {
      if (items.length === 0) throw new Error('Stack empty');
      return items.pop();
    },
    peek() { return items[items.length - 1]; },
    get size() { return items.length; },
    isEmpty() { return items.length === 0; }
  };
}

// === Class approach ===
class Stack {
  #items = []; // private field (ES2022)

  push(item) { this.#items.push(item); return this; }
  pop() {
    if (this.#items.length === 0) throw new Error('Stack empty');
    return this.#items.pop();
  }
  peek() { return this.#items[this.#items.length - 1]; }
  get size() { return this.#items.length; }
  isEmpty() { return this.#items.length === 0; }
}

// ข้อแตกต่าง:
const cs = createStack();
const classStack = new Stack();

// 1. Memory: Closure แต่ละ instance มี methods ของตัวเอง
//    Class แชร์ methods ผ่าน prototype
console.log(cs.push === createStack().push); // false (different instances)
console.log(classStack.push === new Stack().push); // true (shared prototype)

// 2. Inheritance: Class ง่ายกว่า
//    Closure ต้อง manual composition

// 3. instanceof: Class รองรับ, Closure ไม่รองรับ
console.log(classStack instanceof Stack); // true
// cs instanceof ??? // ไม่รองรับ

// 4. Privacy: ทั้งคู่ปกป้องได้
// items ใน closure: ไม่มีทางเข้าถึงได้เลย
// #items ใน class: เข้าถึงได้บางส่วนผ่าน WeakRef tricks
```

---

## Step 645: Once() - Run Function Only Once

```javascript
// once: สร้าง function ที่ทำงานได้แค่ครั้งเดียว
function once(fn) {
  let called = false;
  let result;

  return function(...args) {
    if (!called) {
      called = true;
      result = fn.apply(this, args);
    }
    return result;
  };
}

const initializeApp = once(function() {
  console.log('กำลัง initialize แอป...');
  return { initialized: true, timestamp: Date.now() };
});

console.log(initializeApp()); // กำลัง initialize... { initialized: true, ... }
console.log(initializeApp()); // { initialized: true, ... } (ไม่ print ซ้ำ)
console.log(initializeApp()); // { initialized: true, ... }
```

```javascript
// once พร้อม callback สำหรับ subsequent calls
function onceWithFallback(fn, fallback = () => {}) {
  let called = false;
  let result;

  return function(...args) {
    if (!called) {
      called = true;
      result = fn.apply(this, args);
      return result;
    }
    return fallback(result, ...args);
  };
}

const setupDatabase = onceWithFallback(
  () => {
    console.log('Setting up database...');
    return { db: 'connected' };
  },
  (existingResult) => {
    console.log('Database already set up, returning existing connection');
    return existingResult;
  }
);

setupDatabase(); // Setting up database...
setupDatabase(); // Database already set up, returning existing connection
```

---

## Step 646: Debounce Implementation

```javascript
// Debounce: รอให้หยุดพิมพ์ก่อนค่อยทำงาน
function debounce(fn, delay) {
  let timerId;

  return function(...args) {
    clearTimeout(timerId);
    timerId = setTimeout(() => {
      fn.apply(this, args);
    }, delay);
  };
}

// ตัวอย่าง: Search input
const searchInput = debounce(function(query) {
  console.log(`ค้นหา: "${query}"`);
  // fetch('/api/search?q=' + query)
}, 300);

// จำลองการพิมพ์
searchInput('h');     // ยกเลิก
searchInput('he');    // ยกเลิก
searchInput('hel');   // ยกเลิก
searchInput('hell');  // ยกเลิก
searchInput('hello'); // รอ 300ms แล้วจึงทำงาน -> ค้นหา: "hello"
```

```javascript
// Debounce แบบ advanced
function debounceAdvanced(fn, delay, { leading = false, trailing = true } = {}) {
  let timerId;
  let lastCallTime;

  return function(...args) {
    const now = Date.now();
    const shouldLead = leading && (lastCallTime === undefined || now - lastCallTime >= delay);

    clearTimeout(timerId);
    lastCallTime = now;

    if (shouldLead) {
      fn.apply(this, args); // เรียกทันทีครั้งแรก
    }

    if (trailing) {
      timerId = setTimeout(() => {
        if (trailing && !shouldLead) {
          fn.apply(this, args);
        }
        lastCallTime = undefined;
      }, delay);
    }
  };
}

// cancel และ flush
function debounceWithControl(fn, delay) {
  let timerId;
  let pendingArgs;

  const debounced = function(...args) {
    pendingArgs = args;
    clearTimeout(timerId);
    timerId = setTimeout(() => {
      fn.apply(this, pendingArgs);
      pendingArgs = undefined;
    }, delay);
  };

  debounced.cancel = function() {
    clearTimeout(timerId);
    pendingArgs = undefined;
  };

  debounced.flush = function() {
    if (pendingArgs) {
      clearTimeout(timerId);
      fn.apply(this, pendingArgs);
      pendingArgs = undefined;
    }
  };

  return debounced;
}
```

---

## Step 647: Throttle Implementation

```javascript
// Throttle: จำกัดความถี่การทำงาน
function throttle(fn, interval) {
  let lastTime = 0;

  return function(...args) {
    const now = Date.now();

    if (now - lastTime >= interval) {
      lastTime = now;
      return fn.apply(this, args);
    }
  };
}

// ตัวอย่าง: scroll event handler
const handleScroll = throttle(function() {
  console.log('กำลัง scroll, scrollY:', window.scrollY || 0);
}, 100);

// จำลอง scroll events ที่มาถี่มาก
const scrollTimes = [0, 20, 50, 80, 100, 150, 200, 250, 300];
scrollTimes.forEach(time => {
  setTimeout(handleScroll, time);
});
// จะทำงานที่ 0, 100, 200, 300 ไม่ใช่ทุก event
```

```javascript
// Throttle แบบ advanced พร้อม trailing
function throttleAdvanced(fn, interval) {
  let lastTime = 0;
  let trailingTimer;

  return function(...args) {
    const now = Date.now();
    const remaining = interval - (now - lastTime);

    if (remaining <= 0) {
      clearTimeout(trailingTimer);
      lastTime = now;
      fn.apply(this, args);
    } else {
      // Schedule trailing call
      clearTimeout(trailingTimer);
      trailingTimer = setTimeout(() => {
        lastTime = Date.now();
        fn.apply(this, args);
      }, remaining);
    }
  };
}

// rate limiter สำหรับ API calls
function createRateLimiter(maxCalls, windowMs) {
  const calls = [];

  return function canCall() {
    const now = Date.now();
    const windowStart = now - windowMs;

    // ลบ calls เก่า
    while (calls.length > 0 && calls[0] < windowStart) {
      calls.shift();
    }

    if (calls.length < maxCalls) {
      calls.push(now);
      return true;
    }
    return false;
  };
}

const limiter = createRateLimiter(3, 1000); // 3 calls per second

function makeApiCall(id) {
  if (limiter()) {
    console.log(`API call ${id} ผ่าน`);
  } else {
    console.log(`API call ${id} ถูก rate limit`);
  }
}

makeApiCall(1); // ผ่าน
makeApiCall(2); // ผ่าน
makeApiCall(3); // ผ่าน
makeApiCall(4); // ถูก rate limit
```

---

## Step 648: Real-World Applications

```javascript
// 1. Middleware pattern (คล้าย Express.js)
function createMiddlewareChain() {
  const middlewares = [];

  function use(fn) {
    middlewares.push(fn);
    return { use }; // chaining
  }

  function execute(context) {
    let index = 0;

    function next(err) {
      if (err) {
        console.error('Error in middleware:', err.message);
        return;
      }

      if (index >= middlewares.length) return;

      const middleware = middlewares[index++];
      try {
        middleware(context, next);
      } catch (e) {
        next(e);
      }
    }

    next();
    return context;
  }

  return { use, execute };
}

const pipeline = createMiddlewareChain();

pipeline
  .use((ctx, next) => {
    console.log('Middleware 1: เริ่มต้น');
    ctx.start = Date.now();
    next();
  })
  .use((ctx, next) => {
    console.log('Middleware 2: ตรวจสอบ authentication');
    ctx.user = { id: 1, name: 'อลิส' };
    next();
  })
  .use((ctx, next) => {
    console.log('Middleware 3: ดำเนินการ');
    ctx.result = 'success';
    next();
  })
  .use((ctx, next) => {
    const elapsed = Date.now() - ctx.start;
    console.log(`Middleware 4: เสร็จสิ้น ใช้เวลา ${elapsed}ms`);
  });

const result = pipeline.execute({ method: 'GET', path: '/api/data' });
console.log('Final context:', result);
```

```javascript
// 2. State machine with closure
function createStateMachine(initialState, transitions) {
  let currentState = initialState;
  const listeners = new Map();
  const history = [];

  function emit(event, data) {
    const handlers = listeners.get(event) || [];
    handlers.forEach(handler => handler(data));
  }

  return {
    get state() { return currentState; },
    get history() { return [...history]; },

    transition(event) {
      const stateTransitions = transitions[currentState];
      if (!stateTransitions) {
        throw new Error(`ไม่มี transitions สำหรับ state: ${currentState}`);
      }

      const nextState = stateTransitions[event];
      if (!nextState) {
        throw new Error(`ไม่มี transition "${event}" จาก state "${currentState}"`);
      }

      history.push({ from: currentState, event, to: nextState, timestamp: Date.now() });
      const prevState = currentState;
      currentState = nextState;

      emit('transition', { from: prevState, to: nextState, event });
      return this;
    },

    on(event, handler) {
      if (!listeners.has(event)) listeners.set(event, []);
      listeners.get(event).push(handler);
      return this;
    },

    can(event) {
      return !!(transitions[currentState]?.[event]);
    }
  };
}

// Traffic light state machine
const trafficLight = createStateMachine('red', {
  red: { go: 'green' },
  green: { slow: 'yellow' },
  yellow: { stop: 'red' }
});

trafficLight.on('transition', ({ from, to, event }) => {
  console.log(`ไฟจราจร: ${from} --[${event}]--> ${to}`);
});

trafficLight.transition('go');   // ไฟจราจร: red --[go]--> green
trafficLight.transition('slow'); // ไฟจราจร: green --[slow]--> yellow
trafficLight.transition('stop'); // ไฟจราจร: yellow --[stop]--> red

console.log('State ปัจจุบัน:', trafficLight.state); // red
console.log('สามารถ go ได้:', trafficLight.can('go')); // true
```

---

## Step 649: Function Composition with Closures

```javascript
// Compose และ Pipe
const compose = (...fns) => x => fns.reduceRight((v, f) => f(v), x);
const pipe    = (...fns) => x => fns.reduce((v, f) => f(v), x);

// ตัวอย่าง
const trim     = s => s.trim();
const lower    = s => s.toLowerCase();
const removeSpaces = s => s.replace(/\s+/g, '-');
const addPrefix = s => `blog-${s}`;

const slugify = pipe(trim, lower, removeSpaces, addPrefix);

console.log(slugify('  Hello World  ')); // blog-hello-world
console.log(slugify('JavaScript คืออะไร')); // blog-javascript-คืออะไร
```

```javascript
// Transducer concept
const transduce = (xf, reducer, init, coll) => {
  return coll.reduce(xf(reducer), init);
};

const mapping = fn => reducer => (acc, val) => reducer(acc, fn(val));
const filtering = pred => reducer => (acc, val) => pred(val) ? reducer(acc, val) : acc;
const appending = (acc, val) => [...acc, val];

const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

// Transducer: filter even -> double -> collect
const xf = compose(
  filtering(x => x % 2 === 0),
  mapping(x => x * 2)
);

const result = transduce(xf, appending, [], numbers);
console.log(result); // [4, 8, 12, 16, 20]
```

---

## Step 650: สรุปและ Advanced Patterns

```javascript
// Closure ใน Async Context
function createAsyncQueue() {
  const queue = [];
  let processing = false;

  async function processNext() {
    if (processing || queue.length === 0) return;

    processing = true;
    const { task, resolve, reject } = queue.shift();

    try {
      const result = await task();
      resolve(result);
    } catch (err) {
      reject(err);
    } finally {
      processing = false;
      processNext(); // process next item
    }
  }

  return {
    add(task) {
      return new Promise((resolve, reject) => {
        queue.push({ task, resolve, reject });
        processNext();
      });
    },
    get size() { return queue.length; },
    get isProcessing() { return processing; }
  };
}

const queue = createAsyncQueue();

// เพิ่ม tasks
queue.add(async () => {
  await new Promise(r => setTimeout(r, 100));
  console.log('Task 1 เสร็จ');
  return 'Result 1';
}).then(r => console.log('Got:', r));

queue.add(async () => {
  await new Promise(r => setTimeout(r, 50));
  console.log('Task 2 เสร็จ');
  return 'Result 2';
}).then(r => console.log('Got:', r));
```

---

## แบบฝึกหัด (Exercises)

### แบบฝึกหัดที่ 1: Cache with Expiry
```javascript
// สร้าง createCache ที่:
// 1. เก็บ key-value pairs
// 2. มี TTL (time to live) per entry
// 3. มี max size และ eviction policy (LRU)
// 4. มี stats (hits, misses, evictions)

function createCache(maxSize = 100, defaultTTL = 5000) {
  // TODO: implement
  // hint: ใช้ Map สำหรับ storage + doubly linked list สำหรับ LRU
}
```

### แบบฝึกหัดที่ 2: Promise-based Queue
```javascript
// สร้าง PromiseQueue ที่:
// 1. รัน tasks พร้อมกันได้ตามจำนวน concurrency ที่กำหนด
// 2. retry ได้เมื่อ task ล้มเหลว
// 3. มี timeout per task
// 4. emit events (task:start, task:end, task:error)

function createPromiseQueue({ concurrency = 1, retries = 0, timeout = 30000 } = {}) {
  // TODO: implement
}
```

### แบบฝึกหัดที่ 3: Event Bus
```javascript
// สร้าง global event bus ที่:
// 1. subscribe/unsubscribe ด้วย patterns (wildcards)
// 2. emit ด้วย async support
// 3. ป้องกัน memory leaks ด้วย WeakRef

function createEventBus() {
  // TODO: implement
}
```

### แบบฝึกหัดที่ 4: Functional State Management
```javascript
// สร้าง store คล้าย Redux:
// 1. dispatch(action) เพื่อเปลี่ยน state
// 2. subscribe(listener) เพื่อฟังการเปลี่ยนแปลง
// 3. getState() อ่าน current state
// 4. รองรับ middleware

function createStore(reducer, initialState, ...middlewares) {
  // TODO: implement
}
```

---

## สรุป

ใน Part 33 เราได้เรียนรู้:

1. **Closure คืออะไร**: function ที่จำ scope ที่สร้างขึ้น
2. **Lexical Scoping**: scope ถูกกำหนดตอน write-time
3. **How Closures Work**: closure capture environment
4. **Practical Examples**: greeting factory, HTTP client
5. **Counter**: การใช้ closure สร้าง stateful counter
6. **Private State**: ซ่อน implementation detail ด้วย closure
7. **Module Pattern**: IIFE + closure = module
8. **Factory Functions**: สร้าง objects ด้วย closure
9. **Memoization**: cache ผลลัพธ์ด้วย closure
10. **Partial Application**: preset args บางส่วน
11. **Currying**: แปลง multi-arg function เป็น chain
12. **Loop Pitfall**: ปัญหา closure ใน loops
13. **Garbage Collection**: memory implications
14. **Once, Debounce, Throttle**: patterns ที่ใช้งานจริง

ใน Part 34 เราจะเรียนรู้เรื่อง Higher-Order Functions!
