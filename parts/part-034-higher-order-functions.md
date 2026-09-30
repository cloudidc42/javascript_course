# Part 34: Higher-Order Functions (Steps 651-670)

## บทนำ

Higher-Order Functions (HOFs) คือฟังก์ชันที่รับฟังก์ชันอื่นเป็น argument หรือ return ฟังก์ชัน (หรือทั้งสอง) เป็นหนึ่งในแนวคิดหลักของ Functional Programming ซึ่งช่วยให้เขียนโค้ดที่ modular, reusable และ composable ได้ดียิ่งขึ้น

---

## Step 651: What Are Higher-Order Functions?

```javascript
// Higher-Order Function รับ function เป็น argument
function applyToAll(array, fn) {
  return array.map(fn);
}

const numbers = [1, 2, 3, 4, 5];
const doubled  = applyToAll(numbers, x => x * 2);
const squared  = applyToAll(numbers, x => x ** 2);
const strings  = applyToAll(numbers, x => `item-${x}`);

console.log(doubled);  // [2, 4, 6, 8, 10]
console.log(squared);  // [1, 4, 9, 16, 25]
console.log(strings);  // ['item-1', 'item-2', ...]
```

```javascript
// Higher-Order Function return function
function multiplier(factor) {
  return function(number) {
    return number * factor;
  };
}

const double  = multiplier(2);
const triple  = multiplier(3);
const tenX    = multiplier(10);

console.log(double(5));  // 10
console.log(triple(5));  // 15
console.log(tenX(5));    // 50

// ทั้ง input และ output เป็น functions
function compose(f, g) {
  return function(x) {
    return f(g(x));
  };
}

const addOne   = x => x + 1;
const double2  = x => x * 2;

const addOneThenDouble = compose(double2, addOne);
const doubleThenAddOne = compose(addOne, double2);

console.log(addOneThenDouble(5)); // (5+1)*2 = 12
console.log(doubleThenAddOne(5)); // (5*2)+1 = 11
```

---

## Step 652: Functions as First-Class Citizens

ใน JavaScript ฟังก์ชันเป็น "first-class citizens" หมายความว่า:

```javascript
// 1. กำหนดให้ตัวแปร
const greet = function(name) {
  return `สวัสดี ${name}`;
};

// 2. เก็บใน array
const operations = [
  x => x + 1,
  x => x * 2,
  x => x ** 2,
  x => Math.sqrt(x)
];

let result = 4;
operations.forEach(op => {
  result = op(result);
  console.log(result);
});
// 5, 10, 100, 10

// 3. เก็บใน object
const mathOps = {
  add: (a, b) => a + b,
  subtract: (a, b) => a - b,
  multiply: (a, b) => a * b,
  divide: (a, b) => {
    if (b === 0) throw new Error('หารด้วยศูนย์ไม่ได้');
    return a / b;
  }
};

console.log(mathOps.add(5, 3));      // 8
console.log(mathOps.multiply(4, 7)); // 28

// 4. ส่งเป็น argument
function calculate(a, b, operation) {
  return operation(a, b);
}

console.log(calculate(10, 3, mathOps.subtract)); // 7
console.log(calculate(4, 5, mathOps.multiply));  // 20

// 5. Return จาก function
function makeAdder(x) {
  return function(y) { return x + y; };
}

const add5 = makeAdder(5);
console.log(add5(3));  // 8
console.log(add5(10)); // 15
```

---

## Step 653: Passing Functions as Arguments (Callbacks)

```javascript
// Callback pattern
function processArray(arr, callback) {
  const results = [];
  for (const item of arr) {
    results.push(callback(item));
  }
  return results;
}

const nums = [1, 2, 3, 4, 5];

// Named function
function square(x) { return x ** 2; }
console.log(processArray(nums, square)); // [1, 4, 9, 16, 25]

// Anonymous function
console.log(processArray(nums, function(x) { return x + 10; }));

// Arrow function
console.log(processArray(nums, x => x % 2 === 0 ? 'even' : 'odd'));
```

```javascript
// Callback สำหรับ async operations (แบบดั้งเดิม)
function fetchUserData(userId, onSuccess, onError) {
  // จำลอง async operation
  setTimeout(() => {
    if (userId > 0) {
      onSuccess({ id: userId, name: 'User ' + userId });
    } else {
      onError(new Error('Invalid user ID'));
    }
  }, 100);
}

fetchUserData(
  1,
  user => console.log('สำเร็จ:', user),
  err  => console.log('ผิดพลาด:', err.message)
);

// Event listeners
const button = {
  listeners: {},
  on(event, callback) {
    if (!this.listeners[event]) this.listeners[event] = [];
    this.listeners[event].push(callback);
  },
  emit(event, data) {
    (this.listeners[event] || []).forEach(cb => cb(data));
  }
};

button.on('click', (data) => console.log('Click:', data));
button.on('click', (data) => console.log('Also Click:', data));
button.emit('click', { x: 100, y: 200 });
```

---

## Step 654: Returning Functions from Functions

```javascript
// Function factories
function createGreeter(language) {
  const greetings = {
    en: 'Hello',
    th: 'สวัสดี',
    jp: 'こんにちは',
    fr: 'Bonjour'
  };

  const greeting = greetings[language] || greetings.en;

  return function(name) {
    return `${greeting}, ${name}!`;
  };
}

const greetEn = createGreeter('en');
const greetTh = createGreeter('th');
const greetJp = createGreeter('jp');

console.log(greetEn('Alice'));   // Hello, Alice!
console.log(greetTh('สมชาย'));  // สวัสดี, สมชาย!
console.log(greetJp('Tanaka')); // こんにちは, Tanaka!
```

```javascript
// Middleware creator
function createRateLimit(limit, windowMs) {
  return function rateLimitMiddleware(req, res, next) {
    const key = req.ip || 'unknown';
    // Logic would use closure to track requests...
    console.log(`Rate limit check for ${key}: ${limit} req/${windowMs}ms`);
    next();
  };
}

function createLogger(format) {
  return function loggerMiddleware(req, res, next) {
    const start = Date.now();
    next();
    console.log(format
      .replace('{method}', req.method || 'GET')
      .replace('{url}', req.url || '/')
      .replace('{time}', Date.now() - start + 'ms')
    );
  };
}

const rateLimit = createRateLimit(100, 60000);
const logger = createLogger('[{method}] {url} - {time}');

// middleware chain (simplified)
function applyMiddleware(req, middlewares) {
  const res = {};
  let i = 0;
  function next() {
    if (i < middlewares.length) {
      middlewares[i++](req, res, next);
    }
  }
  next();
}

applyMiddleware(
  { method: 'GET', url: '/api/users', ip: '127.0.0.1' },
  [rateLimit, logger]
);
```

---

## Step 655: Function Factories

```javascript
// Validator factory
function createValidator(type) {
  const validators = {
    email: value => /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value),
    phone: value => /^[0-9]{9,10}$/.test(value),
    thai_id: value => /^[0-9]{13}$/.test(value),
    url: value => {
      try { new URL(value); return true; }
      catch { return false; }
    },
    positive: value => typeof value === 'number' && value > 0,
    integer: value => Number.isInteger(value)
  };

  return validators[type] || (() => true);
}

const isEmail    = createValidator('email');
const isPhone    = createValidator('phone');
const isPositive = createValidator('positive');

console.log(isEmail('test@example.com'));  // true
console.log(isEmail('not-an-email'));      // false
console.log(isPhone('0812345678'));        // true
console.log(isPositive(42));              // true
console.log(isPositive(-5));              // false
```

```javascript
// Transformer factory
function createTransformer(operations) {
  return function transform(data) {
    return operations.reduce((acc, op) => op(acc), data);
  };
}

const processText = createTransformer([
  s => s.trim(),
  s => s.toLowerCase(),
  s => s.replace(/[^\w\s-]/g, ''),
  s => s.replace(/\s+/g, '-'),
  s => s.replace(/-+/g, '-')
]);

const processNumber = createTransformer([
  n => Math.abs(n),
  n => Math.round(n * 100) / 100,
  n => Math.min(n, 1000),
  n => Math.max(n, 0)
]);

console.log(processText('  Hello, World! How are you?  ')); // hello-world-how-are-you
console.log(processNumber(-1234.5678)); // 1000 (capped)
console.log(processNumber(-3.141592)); // 3.14
```

---

## Step 656: Composition - compose() and pipe()

```javascript
// compose: right-to-left (mathematical notation)
// pipe: left-to-right (more readable)

const compose = (...fns) =>
  fns.reduce((f, g) => (...args) => f(g(...args)));

const pipe = (...fns) =>
  fns.reduce((f, g) => (...args) => g(f(...args)));

// String processing functions
const trim    = s => s.trim();
const lower   = s => s.toLowerCase();
const words   = s => s.split(/\s+/);
const unique  = arr => [...new Set(arr)];
const sort    = arr => [...arr].sort();
const join    = sep => arr => arr.join(sep);

// compose กับ pipe
const normalize = pipe(trim, lower, words, unique, sort, join(' '));

console.log(normalize('  Hello world hello JavaScript World  '));
// hello javascript world

// Compose ใช้สำหรับ data transformation pipeline
const processUser = pipe(
  user => ({ ...user, name: user.name.trim() }),
  user => ({ ...user, email: user.email.toLowerCase() }),
  user => ({ ...user, createdAt: new Date().toISOString() }),
  user => ({ ...user, id: Math.random().toString(36).slice(2) })
);

const user = processUser({ name: '  สมชาย  ', email: 'SOMCHAI@TEST.COM' });
console.log(user);
// { name: 'สมชาย', email: 'somchai@test.com', createdAt: '...', id: '...' }
```

```javascript
// Async pipe
const pipeAsync = (...fns) =>
  async (x) => fns.reduce(async (v, f) => f(await v), Promise.resolve(x));

const fetchUser = async (id) => {
  await new Promise(r => setTimeout(r, 10));
  return { id, name: 'User ' + id, active: true };
};

const checkActive = async (user) => {
  if (!user.active) throw new Error('User inactive');
  return user;
};

const enrichUser = async (user) => ({
  ...user,
  displayName: `Mr./Ms. ${user.name}`,
  lastSeen: new Date().toISOString()
});

const processUserPipeline = pipeAsync(fetchUser, checkActive, enrichUser);

processUserPipeline(42).then(result => {
  console.log(result);
  // { id: 42, name: 'User 42', active: true, displayName: 'Mr./Ms. User 42', ... }
});
```

---

## Step 657: Currying - Detailed Explanation

```javascript
// Currying คืออะไร?
// แปลง f(a, b, c) เป็น f(a)(b)(c)

// Manual currying
const add3Manual = a => b => c => a + b + c;
console.log(add3Manual(1)(2)(3)); // 6

// Automatic currying
function curry(fn) {
  const arity = fn.length;

  return function curried(...args) {
    if (args.length >= arity) {
      return fn(...args);
    }
    return function(...moreArgs) {
      return curried(...args, ...moreArgs);
    };
  };
}

// ตัวอย่าง
const add = curry((a, b) => a + b);
const multiply = curry((a, b) => a * b);
const subtract = curry((a, b) => a - b);

// สร้าง specialized functions
const add10   = add(10);
const double  = multiply(2);
const sub5    = subtract(5);  // จะ: 5 - b ไม่ใช่ b - 5

console.log(add10(5));   // 15
console.log(double(7));  // 14
console.log(sub5(3));    // 2 (5 - 3)
```

```javascript
// Real-world currying examples
const curry2 = fn => a => b => fn(a, b);
const curry3 = fn => a => b => c => fn(a, b, c);

// Database query builder
const buildQuery = curry3((table, conditions, fields) => ({
  table,
  conditions,
  fields,
  toSQL() {
    const fieldStr = fields.join(', ');
    const condStr = conditions.map(([k, v]) => `${k} = '${v}'`).join(' AND ');
    return `SELECT ${fieldStr} FROM ${table}${condStr ? ' WHERE ' + condStr : ''}`;
  }
}));

const fromUsers  = buildQuery('users');
const activeUsers = fromUsers([['active', 'true']]);

const userNames  = activeUsers(['name', 'email']);
const userAll    = activeUsers(['*']);

console.log(userNames.toSQL()); // SELECT name, email FROM users WHERE active = 'true'
console.log(userAll.toSQL());   // SELECT * FROM users WHERE active = 'true'

// String interpolation
const interpolate = curry2((template, data) =>
  template.replace(/\{(\w+)\}/g, (_, key) => data[key] ?? '')
);

const greetTemplate = interpolate('สวัสดี {name} คุณอายุ {age} ปี');
console.log(greetTemplate({ name: 'สมชาย', age: 25 }));
// สวัสดี สมชาย คุณอายุ 25 ปี
```

---

## Step 658: Partial Application

```javascript
// Partial Application: กำหนด arguments บางส่วนล่วงหน้า
function partial(fn, ...presetArgs) {
  return function partiallyApplied(...laterArgs) {
    return fn(...presetArgs, ...laterArgs);
  };
}

// ตัวอย่าง
function log(level, timestamp, message) {
  console.log(`[${level.toUpperCase()}] ${timestamp}: ${message}`);
}

const logNow = partial(log, 'info', new Date().toISOString());
logNow('เริ่มต้นระบบ');  // [INFO] 2024-...: เริ่มต้นระบบ
logNow('โหลด config');   // [INFO] 2024-...: โหลด config

const logError = partial(log, 'error');
logError(new Date().toISOString(), 'ไม่สามารถเชื่อมต่อ database');
```

```javascript
// partialRight: กำหนด args จากด้านขวา
function partialRight(fn, ...presetArgs) {
  return function(...laterArgs) {
    return fn(...laterArgs, ...presetArgs);
  };
}

function divide(a, b) { return a / b; }

const divideBy2 = partialRight(divide, 2);
const divideBy10 = partialRight(divide, 10);

console.log(divideBy2(100));  // 50 (100 / 2)
console.log(divideBy10(500)); // 50 (500 / 10)

// Real-world: API with base configuration
function makeRequest(config, url, method, body) {
  const fullConfig = {
    ...config,
    url,
    method,
    body: body ? JSON.stringify(body) : undefined
  };
  console.log('Making request:', fullConfig);
  return fullConfig;
}

const makeAuthRequest = partial(makeRequest, {
  headers: { 'Authorization': 'Bearer token123' },
  timeout: 5000
});

const postAuthRequest = partial(makeAuthRequest, null, 'POST');
// usage: postAuthRequest('/api/data', { name: 'test' })
```

---

## Step 659: once() - Run Function Only Once

```javascript
// once: ทำงานแค่ครั้งเดียว
function once(fn) {
  let done = false;
  let result;

  return function(...args) {
    if (!done) {
      done = true;
      result = fn.apply(this, args);
    }
    return result;
  };
}

// Initialization
const initDatabase = once(async () => {
  console.log('กำลัง initialize database...');
  await new Promise(r => setTimeout(r, 100));
  return { connection: 'active', host: 'localhost' };
});

// เรียกกี่ครั้งก็ได้ แต่ทำงานแค่ครั้งแรก
initDatabase().then(db => console.log('DB 1:', db.connection));
initDatabase().then(db => console.log('DB 2:', db.connection)); // ได้ cached result
initDatabase().then(db => console.log('DB 3:', db.connection)); // ได้ cached result
```

```javascript
// once พร้อม named wrapper
function namedOnce(name, fn) {
  let called = false;
  let result;

  return function(...args) {
    if (!called) {
      console.log(`[once:${name}] กำลังรัน...`);
      called = true;
      result = fn.apply(this, args);
    } else {
      console.log(`[once:${name}] เรียกซ้ำ - return cached result`);
    }
    return result;
  };
}

const loadConfig = namedOnce('loadConfig', () => ({
  theme: 'dark',
  language: 'th',
  apiUrl: 'https://api.example.com'
}));

const c1 = loadConfig(); // [once:loadConfig] กำลังรัน...
const c2 = loadConfig(); // [once:loadConfig] เรียกซ้ำ - return cached result
console.log(c1 === c2);  // true - same object reference
```

---

## Step 660: memoize() - Caching Function Results

```javascript
// Basic memoize
function memoize(fn) {
  const cache = new Map();

  function memoized(...args) {
    const key = JSON.stringify(args);

    if (cache.has(key)) {
      return cache.get(key);
    }

    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  }

  memoized.cache = cache;
  memoized.clear = () => cache.clear();
  memoized.delete = (...args) => cache.delete(JSON.stringify(args));

  return memoized;
}

// Heavy computation
const expensiveCalc = memoize(function(n) {
  console.log(`Computing for n=${n}...`);
  let result = 0;
  for (let i = 0; i < n * 1000; i++) {
    result += Math.sqrt(i);
  }
  return result;
});

console.time('first');
expensiveCalc(1000); // Computing for n=1000...
console.timeEnd('first');

console.time('second');
expensiveCalc(1000); // ไม่ print (cached)
console.timeEnd('second'); // เร็วกว่ามาก!

// Fibonacci
const fib = memoize(function fibonacci(n) {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
  // Note: fibonacci ต้องอ้างถึง memoized version
});

// ทางที่ถูกต้องสำหรับ recursive memoize
function createMemoFib() {
  const memo = new Map();

  return function fib(n) {
    if (memo.has(n)) return memo.get(n);
    const result = n <= 1 ? n : fib(n - 1) + fib(n - 2);
    memo.set(n, result);
    return result;
  };
}

const memoFib = createMemoFib();
console.log(memoFib(50)); // 12586269025 (เร็วมาก!)
```

---

## Step 661: debounce() - Practical Implementation

```javascript
// Debounce: ทำงานหลังจากหยุด "ส่งสัญญาณ" แล้ว delay ms
function debounce(fn, delay = 300, options = {}) {
  const { leading = false, trailing = true, maxWait } = options;
  let timer;
  let lastCallTime = 0;
  let lastInvokeTime = 0;
  let lastArgs;
  let lastThis;

  function invoke(time) {
    lastInvokeTime = time;
    const args = lastArgs;
    const thisArg = lastThis;
    lastArgs = lastThis = undefined;
    return fn.apply(thisArg, args);
  }

  function shouldInvoke(time) {
    const timeSinceLastCall   = time - lastCallTime;
    const timeSinceLastInvoke = time - lastInvokeTime;

    return (
      lastCallTime === 0 ||
      timeSinceLastCall >= delay ||
      timeSinceLastCall < 0 ||
      (maxWait !== undefined && timeSinceLastInvoke >= maxWait)
    );
  }

  function trailingEdge(time) {
    timer = undefined;
    if (trailing && lastArgs) {
      return invoke(time);
    }
    lastArgs = lastThis = undefined;
  }

  function debounced(...args) {
    const time = Date.now();
    const isInvoking = shouldInvoke(time);

    lastArgs = args;
    lastThis = this;
    lastCallTime = time;

    if (isInvoking) {
      if (timer === undefined) {
        if (leading) return invoke(time);
      } else if (maxWait !== undefined) {
        clearTimeout(timer);
        timer = setTimeout(trailingEdge, delay, Date.now());
        return invoke(time);
      }
    }

    if (timer === undefined) {
      timer = setTimeout(trailingEdge, delay, Date.now());
    }
  }

  debounced.cancel = function() {
    clearTimeout(timer);
    timer = lastArgs = lastThis = undefined;
    lastCallTime = lastInvokeTime = 0;
  };

  debounced.flush = function() {
    return timer === undefined ? undefined : trailingEdge(Date.now());
  };

  return debounced;
}

// Usage
const searchAPI = debounce((query) => {
  console.log(`Searching: "${query}"`);
}, 300);

// Simulated rapid typing
['h', 'he', 'hel', 'hell', 'hello'].forEach((q, i) => {
  setTimeout(() => searchAPI(q), i * 50);
}); // Only 'hello' will be searched (after 300ms silence)
```

---

## Step 662: throttle() - Practical Implementation

```javascript
// Throttle: จำกัด rate การเรียกใช้งาน
function throttle(fn, limit = 100, options = {}) {
  const { leading = true, trailing = true } = options;
  let lastTime = 0;
  let timer;
  let lastArgs;

  return function throttled(...args) {
    const now = Date.now();
    const remaining = limit - (now - lastTime);

    lastArgs = args;

    if (remaining <= 0 || remaining > limit) {
      if (timer) {
        clearTimeout(timer);
        timer = undefined;
      }
      if (leading || lastTime > 0) {
        lastTime = now;
        fn.apply(this, args);
      }
    } else if (!timer && trailing) {
      timer = setTimeout(() => {
        lastTime = leading ? Date.now() : 0;
        timer = undefined;
        fn.apply(this, lastArgs);
      }, remaining);
    }
  };
}

// Usage: resize handler
const handleResize = throttle(() => {
  console.log('Window resized:', Date.now());
}, 200);

// Animation frame throttle
function rafThrottle(fn) {
  let rafId = null;

  return function(...args) {
    if (rafId) return;

    rafId = requestAnimationFrame(() => {
      fn.apply(this, args);
      rafId = null;
    });
  };
}

// Scroll optimization
const optimizedScroll = rafThrottle(() => {
  console.log('Scroll position:', window.scrollY || 0);
});
```

---

## Step 663: tap() for Debugging

```javascript
// tap: ทำงาน side effect แล้ว pass value ต่อ
const tap = fn => x => {
  fn(x);
  return x;
};

// ใช้ใน pipeline สำหรับ debug
const pipeline = [
  x => x * 2,
  tap(x => console.log('after double:', x)),  // debug point
  x => x + 1,
  tap(x => console.log('after add:', x)),      // debug point
  x => x ** 2,
  tap(x => console.log('after square:', x))    // debug point
];

const result = pipeline.reduce((val, fn) => fn(val), 5);
// after double: 10
// after add: 11
// after square: 121
console.log('Final:', result); // 121
```

```javascript
// tap สำหรับ array operations
const tapLog = label => value => {
  console.log(`[${label}]`, JSON.stringify(value));
  return value;
};

const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

const result2 = numbers
  .filter(n => n % 2 === 0)
  // ไม่สามารถ tap ตรงๆ ใน chain ได้, แต่ใช้ trick นี้
  .reduce((acc, n, i, arr) => {
    if (i === 0) console.log('[after filter]', arr);
    acc.push(n * 2);
    return acc;
  }, []);

console.log('Result:', result2); // [4, 8, 12, 16, 20]

// tap ใน promise chain
const tapAsync = fn => async x => {
  await fn(x);
  return x;
};

async function processData(data) {
  return Promise.resolve(data)
    .then(tapAsync(async d => console.log('Start:', d)))
    .then(d => d.filter(n => n > 0))
    .then(tapAsync(async d => console.log('After filter:', d)))
    .then(d => d.map(n => n * 2))
    .then(tapAsync(async d => console.log('After map:', d)));
}

processData([-1, 2, -3, 4, 5]).then(result => {
  console.log('Final:', result);
});
```

---

## Step 664: Built-in HOFs - map()

```javascript
// Array.prototype.map - สร้าง array ใหม่จาก transformation
const numbers = [1, 2, 3, 4, 5];

// Basic usage
console.log(numbers.map(n => n * 2));   // [2, 4, 6, 8, 10]
console.log(numbers.map(n => n ** 2));  // [1, 4, 9, 16, 25]
console.log(numbers.map(String));       // ['1', '2', '3', '4', '5']

// map กับ objects
const users = [
  { id: 1, name: 'อลิส', age: 25 },
  { id: 2, name: 'บ็อบ', age: 30 },
  { id: 3, name: 'ชาร์ลี', age: 35 }
];

const names      = users.map(u => u.name);
const ages       = users.map(u => u.age);
const nameAndAge = users.map(u => `${u.name} (${u.age})`);

console.log(names);      // ['อลิส', 'บ็อบ', 'ชาร์ลี']
console.log(ages);       // [25, 30, 35]
console.log(nameAndAge); // ['อลิส (25)', 'บ็อบ (30)', 'ชาร์ลี (35)']

// Map กับ index
const indexed = numbers.map((n, i) => `${i}: ${n}`);
console.log(indexed); // ['0: 1', '1: 2', ...]

// Flatten nested structure
const matrix = [[1, 2], [3, 4], [5, 6]];
const flat = matrix.map(row => row.map(n => n * 2));
console.log(flat); // [[2, 4], [6, 8], [10, 12]]

// Custom map implementation
Array.prototype.myMap = function(callback) {
  const result = [];
  for (let i = 0; i < this.length; i++) {
    result.push(callback(this[i], i, this));
  }
  return result;
};

console.log([1, 2, 3].myMap(x => x + 10)); // [11, 12, 13]
```

---

## Step 665: Built-in HOFs - filter()

```javascript
// Array.prototype.filter - กรองด้วย predicate
const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

const evens    = numbers.filter(n => n % 2 === 0);
const odds     = numbers.filter(n => n % 2 !== 0);
const bigNums  = numbers.filter(n => n > 5);
const between  = numbers.filter(n => n >= 3 && n <= 7);

console.log(evens);   // [2, 4, 6, 8, 10]
console.log(odds);    // [1, 3, 5, 7, 9]
console.log(bigNums); // [6, 7, 8, 9, 10]
console.log(between); // [3, 4, 5, 6, 7]

// Filter กับ objects
const products = [
  { name: 'แล็ปท็อป', price: 25000, stock: 5 },
  { name: 'เมาส์', price: 800, stock: 0 },
  { name: 'คีย์บอร์ด', price: 1500, stock: 10 },
  { name: 'จอมอนิเตอร์', price: 8000, stock: 3 },
  { name: 'หูฟัง', price: 2000, stock: 0 }
];

const inStock     = products.filter(p => p.stock > 0);
const affordable  = products.filter(p => p.price < 5000);
const available   = products.filter(p => p.stock > 0 && p.price < 3000);

console.log(inStock.map(p => p.name));    // ['แล็ปท็อป', 'คีย์บอร์ด', 'จอมอนิเตอร์']
console.log(affordable.map(p => p.name)); // ['เมาส์', 'คีย์บอร์ด', 'หูฟัง']
console.log(available.map(p => p.name));  // ['คีย์บอร์ด']

// Filter with type checking
const mixed = [1, 'hello', null, undefined, true, [], {}, 0, ''];
const truthy  = mixed.filter(Boolean);
const strings = mixed.filter(x => typeof x === 'string');
const numbers2 = mixed.filter(Number.isFinite);

console.log(truthy);   // [1, 'hello', true, [], {}]
console.log(strings);  // ['hello', '']
console.log(numbers2); // [1, 0]
```

---

## Step 666: Built-in HOFs - reduce()

```javascript
// Array.prototype.reduce - ลดทั้ง array เป็นค่าเดียว
const numbers = [1, 2, 3, 4, 5];

// Sum
const sum = numbers.reduce((acc, n) => acc + n, 0);
console.log(sum); // 15

// Product
const product = numbers.reduce((acc, n) => acc * n, 1);
console.log(product); // 120

// Max / Min
const max = numbers.reduce((max, n) => n > max ? n : max, -Infinity);
const min = numbers.reduce((min, n) => n < min ? n : min, Infinity);
console.log(max, min); // 5 1

// Flatten
const nested = [[1, 2], [3, 4], [5, 6]];
const flat = nested.reduce((acc, arr) => [...acc, ...arr], []);
console.log(flat); // [1, 2, 3, 4, 5, 6]

// Group by
const people = [
  { name: 'อลิส', dept: 'IT' },
  { name: 'บ็อบ', dept: 'HR' },
  { name: 'ชาร์ลี', dept: 'IT' },
  { name: 'เดวิด', dept: 'Finance' },
  { name: 'อีฟ', dept: 'HR' }
];

const grouped = people.reduce((groups, person) => {
  const dept = person.dept;
  groups[dept] = groups[dept] || [];
  groups[dept].push(person.name);
  return groups;
}, {});

console.log(grouped);
// { IT: ['อลิส', 'ชาร์ลี'], HR: ['บ็อบ', 'อีฟ'], Finance: ['เดวิด'] }

// Count occurrences
const fruits = ['แอปเปิล', 'กล้วย', 'แอปเปิล', 'ส้ม', 'กล้วย', 'แอปเปิล'];
const count = fruits.reduce((acc, fruit) => {
  acc[fruit] = (acc[fruit] || 0) + 1;
  return acc;
}, {});

console.log(count); // { แอปเปิล: 3, กล้วย: 2, ส้ม: 1 }
```

```javascript
// Reduce สำหรับ pipeline/compose
const pipeline = [
  n => n * 2,
  n => n + 1,
  n => n ** 2
];

const applyPipeline = (value, fns) => fns.reduce((v, fn) => fn(v), value);

console.log(applyPipeline(5, pipeline)); // ((5*2)+1)^2 = 121

// reduceRight
const composeWith = (...fns) => x => fns.reduceRight((v, f) => f(v), x);

const transform = composeWith(
  n => n ** 2,
  n => n + 1,
  n => n * 2
);

console.log(transform(5)); // เหมือนกัน
```

---

## Step 667: Built-in HOFs - sort(), forEach(), flatMap()

```javascript
// sort() - เรียงลำดับ (modify in place!)
const nums = [3, 1, 4, 1, 5, 9, 2, 6, 5, 3];

// Default sort (lexicographic)
console.log([...nums].sort()); // [1, 1, 2, 3, 3, 4, 5, 5, 6, 9]

// Numeric sort
console.log([...nums].sort((a, b) => a - b)); // ascending
console.log([...nums].sort((a, b) => b - a)); // descending

// Sort objects
const products = [
  { name: 'C', price: 300 },
  { name: 'A', price: 100 },
  { name: 'B', price: 200 }
];

const byPrice = [...products].sort((a, b) => a.price - b.price);
const byName  = [...products].sort((a, b) => a.name.localeCompare(b.name));

// Multi-field sort
const employees = [
  { dept: 'IT', salary: 50000 },
  { dept: 'HR', salary: 40000 },
  { dept: 'IT', salary: 60000 },
  { dept: 'HR', salary: 45000 }
];

const sorted = [...employees].sort((a, b) => {
  const deptCompare = a.dept.localeCompare(b.dept);
  if (deptCompare !== 0) return deptCompare;
  return b.salary - a.salary; // ภายใน dept เรียงตาม salary descending
});

console.log(sorted);
// [{dept: 'HR', salary: 45000}, {dept: 'HR', salary: 40000}, ...]
```

```javascript
// flatMap() - map แล้ว flatten 1 level
const sentences = ['Hello world', 'How are you', 'I am fine'];

const words = sentences.flatMap(s => s.split(' '));
console.log(words); // ['Hello', 'world', 'How', 'are', 'you', 'I', 'am', 'fine']

// flatMap สำหรับ expand items
const orders = [
  { id: 1, items: ['apple', 'banana'] },
  { id: 2, items: ['orange'] },
  { id: 3, items: ['grape', 'melon', 'kiwi'] }
];

const allItems = orders.flatMap(order =>
  order.items.map(item => ({ orderId: order.id, item }))
);
console.log(allItems);
// [{orderId: 1, item: 'apple'}, {orderId: 1, item: 'banana'}, ...]

// forEach - side effects เท่านั้น
const cart = new Map();
['apple', 'banana', 'apple', 'orange'].forEach(item => {
  cart.set(item, (cart.get(item) || 0) + 1);
});
console.log(Object.fromEntries(cart)); // { apple: 2, banana: 1, orange: 1 }
```

---

## Step 668: Creating Custom HOFs

```javascript
// Custom fold (reduce) สำหรับ objects
function objectReduce(obj, fn, initial) {
  return Object.entries(obj).reduce((acc, [key, value]) => {
    return fn(acc, value, key, obj);
  }, initial);
}

const scores = { อลิส: 90, บ็อบ: 75, ชาร์ลี: 85, เดวิด: 95 };

const total   = objectReduce(scores, (acc, score) => acc + score, 0);
const average = total / Object.keys(scores).length;
const highest = objectReduce(scores, (max, score, name) =>
  score > max.score ? { name, score } : max, { score: -Infinity });

console.log('Total:', total);    // 345
console.log('Average:', average); // 86.25
console.log('Highest:', highest); // { name: 'เดวิด', score: 95 }
```

```javascript
// zipWith - combine สอง arrays ด้วย function
function zipWith(fn, arr1, arr2) {
  const len = Math.min(arr1.length, arr2.length);
  return Array.from({ length: len }, (_, i) => fn(arr1[i], arr2[i]));
}

const xs = [1, 2, 3, 4, 5];
const ys = [10, 20, 30, 40, 50];

console.log(zipWith((a, b) => a + b, xs, ys)); // [11, 22, 33, 44, 55]
console.log(zipWith((a, b) => a * b, xs, ys)); // [10, 40, 90, 160, 250]

// scan - like reduce but keeps all intermediate values
function scan(arr, fn, init) {
  const result = [init];
  arr.reduce((acc, val) => {
    const next = fn(acc, val);
    result.push(next);
    return next;
  }, init);
  return result;
}

const cumSum = scan([1, 2, 3, 4, 5], (a, b) => a + b, 0);
console.log(cumSum); // [0, 1, 3, 6, 10, 15]

// unfold - opposite of reduce, generate array
function unfold(fn, seed, maxCount = Infinity) {
  const result = [];
  let value = seed;
  let i = 0;

  while (i++ < maxCount) {
    const next = fn(value);
    if (!next) break;
    const [nextValue, nextSeed] = next;
    result.push(nextValue);
    value = nextSeed;
  }

  return result;
}

// Generate Fibonacci sequence
const fibs = unfold(([a, b]) => [a, [b, a + b]], [0, 1], 10);
console.log(fibs); // [0, 1, 1, 2, 3, 5, 8, 13, 21, 34]
```

---

## Step 669: Functional Programming Patterns

```javascript
// Functor - สิ่งที่มี map method
class Maybe {
  #value;

  constructor(value) {
    this.#value = value;
  }

  static of(value) { return new Maybe(value); }
  static empty() { return new Maybe(null); }

  isNothing() {
    return this.#value === null || this.#value === undefined;
  }

  map(fn) {
    return this.isNothing()
      ? Maybe.empty()
      : Maybe.of(fn(this.#value));
  }

  getOrElse(defaultValue) {
    return this.isNothing() ? defaultValue : this.#value;
  }

  toString() {
    return this.isNothing() ? 'Maybe(Nothing)' : `Maybe(${this.#value})`;
  }
}

// ใช้งาน Maybe เพื่อหลีกเลี่ยง null checks
const getCity = user =>
  Maybe.of(user)
    .map(u => u.address)
    .map(a => a.city)
    .getOrElse('Unknown City');

console.log(getCity({ name: 'อลิส', address: { city: 'กรุงเทพ' } })); // กรุงเทพ
console.log(getCity({ name: 'บ็อบ' }));          // Unknown City
console.log(getCity(null));                        // Unknown City
```

```javascript
// Either - Left (error) หรือ Right (success)
class Either {
  constructor(value) {
    this._value = value;
  }

  static right(value) {
    return new Right(value);
  }

  static left(value) {
    return new Left(value);
  }

  static try(fn) {
    try {
      return Either.right(fn());
    } catch (e) {
      return Either.left(e.message);
    }
  }
}

class Right extends Either {
  isRight() { return true; }
  isLeft()  { return false; }

  map(fn)    { return Either.right(fn(this._value)); }
  chain(fn)  { return fn(this._value); }
  getOrElse(defaultValue) { return this._value; }
  fold(leftFn, rightFn) { return rightFn(this._value); }
}

class Left extends Either {
  isRight() { return false; }
  isLeft()  { return true; }

  map(fn)    { return this; } // ไม่ทำอะไร
  chain(fn)  { return this; }
  getOrElse(defaultValue) { return defaultValue; }
  fold(leftFn, rightFn) { return leftFn(this._value); }
}

// ตัวอย่าง
function parseJSON(str) {
  return Either.try(() => JSON.parse(str));
}

const result = parseJSON('{"name": "อลิส", "age": 25}')
  .map(data => ({ ...data, processed: true }))
  .fold(
    err  => `Error: ${err}`,
    data => `Success: ${data.name}`
  );

console.log(result); // Success: อลิส

const error = parseJSON('invalid json')
  .map(data => data.name)
  .fold(
    err  => `Error: ${err}`,
    data => `Success: ${data}`
  );

console.log(error); // Error: Unexpected token ...
```

---

## Step 670: สรุปและ Real-World Example

```javascript
// Real-world: Data Processing Pipeline
function createDataPipeline(initialData) {
  let data = [...initialData];
  const steps = [];

  const pipeline = {
    filter(predicate) {
      steps.push({ type: 'filter', fn: predicate });
      return this;
    },

    map(transformer) {
      steps.push({ type: 'map', fn: transformer });
      return this;
    },

    reduce(fn, initial) {
      steps.push({ type: 'reduce', fn, initial });
      return this;
    },

    sort(comparator) {
      steps.push({ type: 'sort', fn: comparator });
      return this;
    },

    take(n) {
      steps.push({ type: 'take', n });
      return this;
    },

    execute() {
      return steps.reduce((current, step) => {
        switch (step.type) {
          case 'filter': return current.filter(step.fn);
          case 'map':    return current.map(step.fn);
          case 'reduce': return [current.reduce(step.fn, step.initial)];
          case 'sort':   return [...current].sort(step.fn);
          case 'take':   return current.slice(0, step.n);
          default:       return current;
        }
      }, data);
    }
  };

  return pipeline;
}

// ตัวอย่างการใช้งาน
const sales = [
  { product: 'A', amount: 1200, month: 'Jan', region: 'North' },
  { product: 'B', amount: 800,  month: 'Jan', region: 'South' },
  { product: 'A', amount: 1500, month: 'Feb', region: 'North' },
  { product: 'C', amount: 600,  month: 'Feb', region: 'South' },
  { product: 'B', amount: 900,  month: 'Mar', region: 'North' },
  { product: 'A', amount: 1800, month: 'Mar', region: 'South' }
];

const topNorthSales = createDataPipeline(sales)
  .filter(s => s.region === 'North')
  .map(s => ({ ...s, tax: s.amount * 0.07 }))
  .sort((a, b) => b.amount - a.amount)
  .take(2)
  .execute();

console.log('Top 2 North Sales:');
topNorthSales.forEach(s => {
  console.log(`  ${s.product}: ฿${s.amount} (ภาษี: ฿${s.tax.toFixed(0)})`);
});
```

---

## แบบฝึกหัด (Exercises)

### แบบฝึกหัดที่ 1: Implement Array Methods
```javascript
// สร้าง higher-order functions เหล่านี้จาก scratch:
function myMap(arr, fn) { /* TODO */ }
function myFilter(arr, pred) { /* TODO */ }
function myReduce(arr, fn, init) { /* TODO */ }
function myFlatMap(arr, fn) { /* TODO */ }
function myFind(arr, pred) { /* TODO */ }
function mySome(arr, pred) { /* TODO */ }
function myEvery(arr, pred) { /* TODO */ }
```

### แบบฝึกหัดที่ 2: Functional Utilities
```javascript
// สร้าง utility functions เหล่านี้:
// 1. juxt: รัน array ของ functions บน value เดียว
//    juxt([f, g, h])(x) === [f(x), g(x), h(x)]
// 2. converge: รัน fn บน results ของ array ของ functions
//    converge(fn, [f, g])(x) === fn(f(x), g(x))
// 3. until: รัน fn จนกว่า pred เป็น true
//    until(pred, fn, init) -> value
// 4. evolve: apply fns ให้กับ corresponding keys ของ object
//    evolve({ x: add1, y: double }, { x: 5, y: 10 }) -> { x: 6, y: 20 }
```

### แบบฝึกหัดที่ 3: Lazy Evaluation
```javascript
// สร้าง LazyList ที่:
// 1. ประมวลผลแบบ lazy (ไม่คำนวณจนกว่าจะ materialize)
// 2. รองรับ map, filter, take, first, toArray
// 3. จัดการ infinite sequences ได้

class LazyList {
  // TODO
  static from(iterable) { /* TODO */ }
  static range(start, end, step = 1) { /* TODO */ }
  static repeat(value) { /* TODO */ }
  map(fn) { /* TODO */ }
  filter(pred) { /* TODO */ }
  take(n) { /* TODO */ }
  toArray() { /* TODO */ }
}

// ตัวอย่างการใช้งาน:
// LazyList.range(1, Infinity).filter(n => n % 2 === 0).take(5).toArray()
// -> [2, 4, 6, 8, 10]
```

### แบบฝึกหัดที่ 4: Monad
```javascript
// สร้าง Task monad สำหรับ async operations:
class Task {
  // TODO
  static of(value) { /* resolved task */ }
  static reject(error) { /* rejected task */ }
  static fromPromise(promise) { /* TODO */ }
  map(fn) { /* TODO */ }
  chain(fn) { /* flatMap */ }
  fork(onReject, onResolve) { /* execute */ }
}
```

---

## สรุป

ใน Part 34 เราได้เรียนรู้:

1. **Higher-Order Functions คืออะไร**: รับ/return functions
2. **First-Class Citizens**: functions เป็น values ใน JS
3. **Callbacks**: ส่ง functions เป็น arguments
4. **Returning Functions**: function factories
5. **Composition**: compose() และ pipe()
6. **Currying**: แปลง multi-arg เป็น chain
7. **Partial Application**: preset บาง args
8. **once()**: run function ครั้งเดียว
9. **memoize()**: cache results
10. **debounce()**: delay execution
11. **throttle()**: rate limiting
12. **tap()**: side effects ใน pipeline
13. **Built-in HOFs**: map, filter, reduce, sort
14. **Custom HOFs**: zipWith, scan, unfold
15. **Functional Patterns**: Maybe, Either

ใน Part 35 เราจะเรียนรู้เรื่อง Callbacks และ Event Loop!
