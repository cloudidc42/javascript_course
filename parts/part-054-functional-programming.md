# Part 54: Functional Programming (Steps 1051-1070)

## การเขียนโปรแกรมแบบ Functional

**คำอธิบาย:** Functional Programming (FP) เป็นแนวคิดการเขียนโปรแกรมที่ใช้ฟังก์ชันเป็นหน่วยพื้นฐาน หลีกเลี่ยง side effects และ mutation ทำให้โค้ดอ่านง่าย ทดสอบง่าย และ predictable มากขึ้น

---

## Step 1051: Functional Programming คืออะไร?

```javascript
// ========================
// FP vs OOP Comparison
// ========================

// OOP approach
class UserAccount {
  constructor(balance) {
    this.balance = balance; // mutable state
  }
  
  deposit(amount) {
    this.balance += amount; // mutation!
    return this;
  }
  
  withdraw(amount) {
    if (amount > this.balance) throw new Error('Insufficient funds');
    this.balance -= amount; // mutation!
    return this;
  }
}

const account = new UserAccount(1000);
account.deposit(500);
account.withdraw(200);
console.log(account.balance); // 1300

// FP approach - ไม่ mutate, สร้างสิ่งใหม่
const createAccount = (balance) => ({ balance });

const deposit = (account, amount) => ({
  ...account,
  balance: account.balance + amount
});

const withdraw = (account, amount) => {
  if (amount > account.balance) throw new Error('Insufficient funds');
  return { ...account, balance: account.balance - amount };
};

// Immutable - ทุก operation สร้าง object ใหม่
const acc1 = createAccount(1000);
const acc2 = deposit(acc1, 500);
const acc3 = withdraw(acc2, 200);

console.log(acc1.balance); // 1000 - ไม่เปลี่ยน!
console.log(acc2.balance); // 1500
console.log(acc3.balance); // 1300

// หลักการ FP
const fpPrinciples = {
  pureFunctions: 'same input → same output, no side effects',
  immutability: 'ไม่แก้ไข data, สร้างใหม่เสมอ',
  firstClassFunctions: 'function เป็น value เหมือน int, string',
  higherOrderFunctions: 'function ที่รับ/return function อื่น',
  functionComposition: 'รวม functions เล็กๆ เป็น functions ใหญ่ขึ้น',
  avoidSharedState: 'ไม่ใช้ global state',
  lazyEvaluation: 'คำนวณเมื่อจำเป็น'
};
```

---

## Step 1052: Pure Functions

**Pure Functions** มีคุณสมบัติ: (1) same input → same output และ (2) ไม่มี side effects

```javascript
// ========================
// Pure vs Impure Functions
// ========================

// Impure - มี side effect
let total = 0;
function addToTotal(n) {
  total += n; // side effect! แก้ไข external state
  return total;
}

// Impure - ผลลัพธ์ขึ้นกับ external state
function getCurrentYear() {
  return new Date().getFullYear(); // ผลลัพธ์เปลี่ยนตามเวลา
}

// Impure - แก้ไข input
function sortInPlace(arr) {
  return arr.sort(); // mutation!
}

// Pure equivalents
function pureAdd(a, b) {
  return a + b; // same input → same output, no side effects
}

function pureSort(arr) {
  return [...arr].sort(); // สร้าง copy ใหม่
}

function addToList(list, item) {
  return [...list, item]; // immutable
}

function removeFromList(list, item) {
  return list.filter(x => x !== item);
}

function updateItem(list, index, newValue) {
  return list.map((item, i) => i === index ? newValue : item);
}

// ทดสอบ purity
const list = [3, 1, 4, 1, 5, 9];
const sorted = pureSort(list);
console.log('Original:', list); // [3, 1, 4, 1, 5, 9] - ไม่เปลี่ยน
console.log('Sorted:', sorted); // [1, 1, 3, 4, 5, 9]

// Pure functions ทดสอบง่ายมาก
console.log(pureAdd(2, 3) === 5); // true เสมอ
console.log(pureAdd(2, 3) === pureAdd(2, 3)); // true เสมอ

// ========================
// ทำให้ Impure ฟังก์ชันเป็น Pure
// ========================

// Impure: Logger ที่ side effect
// แก้ด้วย dependency injection
function processOrder(order, logger) {
  // ผล order validation
  const errors = validateOrder(order);
  
  if (errors.length > 0) {
    logger(`Order ${order.id} failed: ${errors.join(', ')}`);
    return { success: false, errors };
  }
  
  logger(`Order ${order.id} processed`);
  return { success: true, order };
}

function validateOrder(order) {
  const errors = [];
  if (!order.id) errors.push('ID required');
  if (!order.items || order.items.length === 0) errors.push('No items');
  if (!order.total || order.total <= 0) errors.push('Invalid total');
  return errors;
}

// ใช้ Pure: ไม่ต้องกังวล side effect
const order = { id: 1, items: ['A'], total: 100 };
const result = processOrder(order, console.log);
console.log(result);

// ========================
// Benefits ของ Pure Functions
// ========================

// 1. Memoization - cache ผลลัพธ์ได้เลย
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

// fibonacci ที่ slow
function fib(n) {
  if (n <= 1) return n;
  return fib(n - 1) + fib(n - 2);
}

// fibonacci ที่ memoized
const memoFib = memoize(function fib(n) {
  if (n <= 1) return n;
  return memoFib(n - 1) + memoFib(n - 2);
});

console.time('slow fib');
console.log(fib(35)); // ช้ามาก
console.timeEnd('slow fib');

console.time('memo fib');
console.log(memoFib(35)); // เร็วมาก
console.timeEnd('memo fib');

// 2. Parallelization - pure functions run ใน parallel ได้
const data = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
const results = data.map(x => x * x); // safe to parallelize
console.log(results);
```

---

## Step 1053: Immutability

```javascript
// ========================
// Immutability Techniques
// ========================

// Object.freeze - shallow immutability
const config = Object.freeze({
  apiUrl: 'https://api.example.com',
  timeout: 5000,
  features: { auth: true } // nested object ยังแก้ได้!
});

config.apiUrl = 'hacked'; // silently fails (or throws in strict mode)
console.log(config.apiUrl); // 'https://api.example.com' - ไม่เปลี่ยน

// Deep freeze
function deepFreeze(obj) {
  if (obj === null || typeof obj !== 'object') return obj;
  
  Object.getOwnPropertyNames(obj).forEach(name => {
    deepFreeze(obj[name]);
  });
  
  return Object.freeze(obj);
}

const deepConfig = deepFreeze({
  server: { host: 'localhost', port: 3000 },
  db: { host: 'db.example.com', maxConn: 10 }
});

// ========================
// Immutable Update Patterns
// ========================

// สำหรับ Objects
const original = { a: 1, b: 2, c: { d: 3, e: 4 } };

// เพิ่ม property
const withF = { ...original, f: 5 };

// แก้ property
const withNewB = { ...original, b: 99 };

// ลบ property
const { a, ...withoutA } = original;

// แก้ nested property
const withNewD = {
  ...original,
  c: { ...original.c, d: 99 }
};

console.log(original); // ไม่เปลี่ยน!
console.log(withF);
console.log(withNewD);

// สำหรับ Arrays
const arr = [1, 2, 3, 4, 5];

// เพิ่ม
const withItem = [...arr, 6];
const withItemFront = [0, ...arr];
const withItemMiddle = [...arr.slice(0, 2), 99, ...arr.slice(2)];

// ลบ
const withoutFirst = arr.slice(1);
const withoutIndex2 = arr.filter((_, i) => i !== 2);

// แก้ไข
const withUpdatedIndex = arr.map((item, i) => i === 2 ? 99 : item);

console.log(arr); // [1, 2, 3, 4, 5] - ไม่เปลี่ยน!

// ========================
// Immutable Data Structures
// ========================

class ImmutableStack {
  constructor(head = null, tail = null) {
    this._head = head;
    this._tail = tail;
    this._size = tail ? tail._size + 1 : head !== null ? 1 : 0;
    Object.freeze(this);
  }
  
  push(value) {
    return new ImmutableStack(value, this);
  }
  
  pop() {
    if (this.isEmpty()) throw new Error('Stack is empty');
    return this._tail;
  }
  
  peek() {
    if (this.isEmpty()) throw new Error('Stack is empty');
    return this._head;
  }
  
  isEmpty() {
    return this._size === 0;
  }
  
  size() {
    return this._size;
  }
  
  toArray() {
    const result = [];
    let current = this;
    while (!current.isEmpty()) {
      result.push(current._head);
      current = current._tail;
    }
    return result;
  }
}

let stack = new ImmutableStack();
const stack1 = stack.push(1);
const stack2 = stack1.push(2);
const stack3 = stack2.push(3);

console.log(stack.size()); // 0
console.log(stack1.size()); // 1
console.log(stack3.toArray()); // [3, 2, 1]
console.log(stack3.pop().toArray()); // [2, 1] - stack3 ยังคงเดิม
```

---

## Step 1054: Higher-Order Functions เชิงลึก

```javascript
// ========================
// Higher-Order Functions
// ========================

// Function ที่รับ function เป็น argument
function pipe(...fns) {
  return function(x) {
    return fns.reduce((acc, fn) => fn(acc), x);
  };
}

function compose(...fns) {
  return function(x) {
    return fns.reduceRight((acc, fn) => fn(acc), x);
  };
}

// Function ที่ return function
function multiplier(factor) {
  return (n) => n * factor;
}

const double = multiplier(2);
const triple = multiplier(3);
const times10 = multiplier(10);

console.log(double(5));   // 10
console.log(triple(4));   // 12
console.log(times10(7)); // 70

// ========================
// Custom map, filter, reduce
// ========================

// Custom map
const myMap = (fn) => (arr) => arr.reduce(
  (acc, item) => [...acc, fn(item)],
  []
);

// Custom filter
const myFilter = (predicate) => (arr) => arr.reduce(
  (acc, item) => predicate(item) ? [...acc, item] : acc,
  []
);

// Custom reduce
const myReduce = (fn, initial) => (arr) => arr.reduce(fn, initial);

// ใช้งาน
const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

const squareAll = myMap(x => x * x);
const keepEvens = myFilter(x => x % 2 === 0);
const sumAll = myReduce((acc, x) => acc + x, 0);

console.log(squareAll(numbers));
console.log(keepEvens(numbers));
console.log(sumAll(numbers));

// รวมกัน ด้วย pipe
const sumOfSquareEvens = pipe(
  keepEvens,
  squareAll,
  sumAll
);

console.log(sumOfSquareEvens(numbers)); // sum of squares of even numbers

// ========================
// Transducer-like composition
// ========================

// Standard: ทำ 3 passes
const result1 = numbers
  .filter(x => x % 2 === 0) // pass 1
  .map(x => x * x)          // pass 2
  .reduce((a, b) => a + b, 0); // pass 3

// Transducer: ทำ 1 pass
function mapping(fn) {
  return (reducer) => (acc, item) => reducer(acc, fn(item));
}

function filtering(predicate) {
  return (reducer) => (acc, item) => predicate(item) ? reducer(acc, item) : acc;
}

const xf = compose(
  filtering(x => x % 2 === 0),
  mapping(x => x * x)
);

const sumReducer = (acc, item) => acc + item;
const transducedResult = numbers.reduce(xf(sumReducer), 0);

console.log(result1 === transducedResult); // true - same result, 1 pass!

// ========================
// Custom Array operations
// ========================

function flatMap(arr, fn) {
  return arr.reduce((acc, item) => [...acc, ...fn(item)], []);
}

function groupBy(arr, keyFn) {
  return arr.reduce((groups, item) => {
    const key = keyFn(item);
    return {
      ...groups,
      [key]: [...(groups[key] || []), item]
    };
  }, {});
}

function indexBy(arr, keyFn) {
  return arr.reduce((index, item) => ({
    ...index,
    [keyFn(item)]: item
  }), {});
}

function countBy(arr, keyFn) {
  return arr.reduce((counts, item) => {
    const key = keyFn(item);
    return { ...counts, [key]: (counts[key] || 0) + 1 };
  }, {});
}

function partition(arr, predicate) {
  return arr.reduce(
    ([truthy, falsy], item) => predicate(item)
      ? [[...truthy, item], falsy]
      : [truthy, [...falsy, item]],
    [[], []]
  );
}

// ใช้งาน
const orders = [
  { id: 1, user: 'สมชาย', status: 'pending', amount: 500 },
  { id: 2, user: 'สมหญิง', status: 'completed', amount: 300 },
  { id: 3, user: 'สมชาย', status: 'completed', amount: 750 },
  { id: 4, user: 'วิชัย', status: 'pending', amount: 200 },
  { id: 5, user: 'สมหญิง', status: 'cancelled', amount: 100 }
];

console.log('By user:', groupBy(orders, o => o.user));
console.log('Indexed:', indexBy(orders, o => o.id)[2]);
console.log('Count by status:', countBy(orders, o => o.status));

const [pending, notPending] = partition(orders, o => o.status === 'pending');
console.log('Pending:', pending.map(o => o.id));
console.log('Not pending:', notPending.map(o => o.id));

const userItems = flatMap(
  [{ user: 'สมชาย', items: ['A', 'B'] }, { user: 'สมหญิง', items: ['C'] }],
  u => u.items.map(item => ({ user: u.user, item }))
);
console.log('Flat mapped:', userItems);
```

---

## Step 1055: Currying และ Partial Application

```javascript
// ========================
// Currying
// ========================

// Manual curry
function add(a, b, c) {
  return a + b + c;
}

// Curried version
const curriedAdd = a => b => c => a + b + c;

console.log(curriedAdd(1)(2)(3)); // 6
console.log(curriedAdd(1)(2));    // returns function
console.log(curriedAdd(1));       // returns function

// Auto curry
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) {
      return fn.apply(this, args);
    }
    return function(...args2) {
      return curried.apply(this, args.concat(args2));
    };
  };
}

const curriedAdd3 = curry(add);
console.log(curriedAdd3(1, 2, 3)); // 6
console.log(curriedAdd3(1)(2)(3)); // 6
console.log(curriedAdd3(1, 2)(3)); // 6
console.log(curriedAdd3(1)(2, 3)); // 6

// ========================
// Practical Curry Examples
// ========================

// curry ช่วยสร้าง specialized functions
const multiply = curry((a, b) => a * b);
const double = multiply(2);
const triple = multiply(3);
const times100 = multiply(100);

console.log([1, 2, 3, 4, 5].map(double));   // [2, 4, 6, 8, 10]
console.log([1, 2, 3].map(triple));           // [3, 6, 9]

// Curried utility functions
const getProperty = curry((key, obj) => obj[key]);
const getName = getProperty('name');
const getAge = getProperty('age');

const users = [
  { name: 'สมชาย', age: 25, email: 'a@test.com' },
  { name: 'สมหญิง', age: 30, email: 'b@test.com' },
  { name: 'วิชัย', age: 28, email: 'c@test.com' }
];

console.log(users.map(getName)); // ['สมชาย', 'สมหญิง', 'วิชัย']
console.log(users.map(getAge));  // [25, 30, 28]

// Curried comparison
const isGreaterThan = curry((min, value) => value > min);
const isPositive = isGreaterThan(0);
const isAdult = isGreaterThan(17);

console.log([-1, 0, 1, 2].filter(isPositive)); // [1, 2]
console.log(users.filter(u => isAdult(u.age))); // users 18+

// Curried string operations
const split = curry((separator, str) => str.split(separator));
const join = curry((separator, arr) => arr.join(separator));
const replace = curry((pattern, replacement, str) => str.replace(pattern, replacement));

const splitBySpace = split(' ');
const joinWithDash = join('-');
const removeVowels = replace(/[aeiou]/gi, '*');

const kebabCase = pipe(
  splitBySpace,
  joinWithDash,
  str => str.toLowerCase()
);

console.log(kebabCase('Hello World JavaScript')); // 'hello-world-javascript'
console.log(removeVowels('Hello World')); // 'H*ll* W*rld'

// ========================
// Partial Application
// ========================

function partial(fn, ...presetArgs) {
  return function(...laterArgs) {
    return fn(...presetArgs, ...laterArgs);
  };
}

function partialRight(fn, ...presetArgs) {
  return function(...laterArgs) {
    return fn(...laterArgs, ...presetArgs);
  };
}

function greet(greeting, name, punctuation) {
  return `${greeting}, ${name}${punctuation}`;
}

const sayHello = partial(greet, 'สวัสดี');
const sayHelloExcited = partial(greet, 'สวัสดี', undefined);

console.log(sayHello('สมชาย', '!'));
console.log(sayHello('วิชัย', '.'));

// Partial ด้วย bind
function power(base, exponent) {
  return Math.pow(base, exponent);
}

const square = partial(power, undefined); // cannot do this way
const square2 = (n) => power(n, 2);      // simpler
const square3 = partialRight(power, 2);  // use partialRight

console.log([2, 3, 4, 5].map(square3)); // [4, 9, 16, 25]
```

---

## Step 1056: Function Composition เชิงลึก

```javascript
// ========================
// Composition และ Pipe
// ========================

// compose: right to left
const compose = (...fns) => x => fns.reduceRight((acc, fn) => fn(acc), x);

// pipe: left to right (more readable)
const pipe = (...fns) => x => fns.reduce((acc, fn) => fn(acc), x);

// ตัวอย่าง
const double = x => x * 2;
const addTen = x => x + 10;
const square = x => x * x;
const negate = x => -x;

const transform1 = compose(negate, square, addTen, double); // double → addTen → square → negate
const transform2 = pipe(double, addTen, square, negate);     // same thing, readable order

console.log(transform1(3)); // -(((3*2)+10)^2) = -(16^2) = -256
console.log(transform2(3)); // same: -256

// ========================
// Async Composition
// ========================

const pipeAsync = (...fns) => async (x) => {
  let result = x;
  for (const fn of fns) {
    result = await fn(result);
  }
  return result;
};

const composeAsync = (...fns) => pipeAsync(...fns.reverse());

// ตัวอย่าง async pipeline
const fetchUser = async (id) => {
  await new Promise(r => setTimeout(r, 10)); // จำลอง API call
  return { id, name: 'สมชาย', email: 'somchai@example.com', roleId: 2 };
};

const enrichWithRole = async (user) => {
  const roles = { 1: 'user', 2: 'admin', 3: 'super_admin' };
  return { ...user, role: roles[user.roleId] || 'unknown' };
};

const addPermissions = async (user) => {
  const permissions = {
    user: ['read'],
    admin: ['read', 'write', 'delete'],
    super_admin: ['read', 'write', 'delete', 'manage']
  };
  return { ...user, permissions: permissions[user.role] || [] };
};

const formatForDisplay = (user) => ({
  id: user.id,
  displayName: user.name,
  email: user.email,
  role: user.role,
  canWrite: user.permissions.includes('write')
});

const getUserWithDetails = pipeAsync(
  fetchUser,
  enrichWithRole,
  addPermissions,
  formatForDisplay
);

getUserWithDetails(1).then(console.log);

// ========================
// Point-free Style
// ========================

// Point-free: ไม่ระบุ arguments โดยตรง

// Normal style (with points)
const isEven = x => x % 2 === 0;
const isPositive = x => x > 0;
const doubleIt = x => x * 2;

// Point-free equivalents using compose/curry
const mod = curry((n, x) => x % n);
const equals = curry((a, b) => a === b);
const gt = curry((min, x) => x > min);
const times = curry((factor, x) => x * factor);

const isEvenPF = compose(equals(0), mod(2));
const isPositivePF = gt(0);
const doublePF = times(2);

// ใช้งาน
const numbers = [-3, -1, 0, 1, 2, 3, 4, 5, 6];

console.log(numbers.filter(isEvenPF));
console.log(numbers.filter(isPositivePF));
console.log(numbers.map(doublePF));

// Point-free data pipeline
const getUserEmails = pipe(
  users => users.filter(u => u.age >= 18), // filter adults
  users => users.map(u => u.email),          // extract emails
  emails => emails.filter(Boolean)           // remove nulls
);

const exampleUsers = [
  { name: 'อ', age: 15, email: 'a@test.com' },
  { name: 'ข', age: 25, email: 'b@test.com' },
  { name: 'ค', age: 30, email: null },
  { name: 'ง', age: 20, email: 'd@test.com' }
];

console.log(getUserEmails(exampleUsers)); // ['b@test.com', 'd@test.com']
```

---

## Step 1057: Functor และ Monad

```javascript
// ========================
// Functor - Mappable Container
// ========================

// Functor คือ container ที่มีเมธอด map
// ตัวอย่างที่รู้จักกันดี: Array
[1, 2, 3].map(x => x * 2); // Array เป็น Functor!

// Custom Functor - Box
class Box {
  constructor(value) {
    this._value = value;
  }
  
  static of(value) {
    return new Box(value);
  }
  
  // Functor law: ต้องมี map
  map(fn) {
    return Box.of(fn(this._value));
  }
  
  // ดูค่าภายใน
  fold(fn) {
    return fn(this._value);
  }
  
  toString() {
    return `Box(${JSON.stringify(this._value)})`;
  }
}

const result = Box.of(5)
  .map(x => x * 2)      // Box(10)
  .map(x => x + 1)      // Box(11)
  .map(x => x.toString()) // Box("11")
  .fold(x => x);        // "11"

console.log(result); // "11"

// ========================
// Maybe Monad - จัดการ null/undefined
// ========================

class Maybe {
  constructor(value) {
    this._value = value;
  }
  
  static of(value) {
    return new Maybe(value);
  }
  
  static empty() {
    return new Maybe(null);
  }
  
  isNothing() {
    return this._value === null || this._value === undefined;
  }
  
  // map: ถ้า nothing ข้ามไป
  map(fn) {
    if (this.isNothing()) return Maybe.empty();
    return Maybe.of(fn(this._value));
  }
  
  // chain/flatMap: สำหรับ functions ที่ return Maybe
  chain(fn) {
    if (this.isNothing()) return Maybe.empty();
    return fn(this._value);
  }
  
  // getOrElse: ดึงค่า หรือใช้ default
  getOrElse(defaultValue) {
    return this.isNothing() ? defaultValue : this._value;
  }
  
  // filter: กลายเป็น nothing ถ้าไม่ผ่าน
  filter(predicate) {
    if (this.isNothing()) return this;
    return predicate(this._value) ? this : Maybe.empty();
  }
  
  toString() {
    return this.isNothing() ? 'Nothing' : `Just(${JSON.stringify(this._value)})`;
  }
}

// ปัญหา: null pointer exceptions
const users2 = {
  1: { name: 'สมชาย', address: { city: 'Bangkok', zip: '10110' } },
  2: { name: 'สมหญิง', address: null },
  3: null
};

// แบบเดิม - เต็มไปด้วย null checks
function getCity(userId) {
  const user = users2[userId];
  if (!user) return 'Unknown';
  if (!user.address) return 'No address';
  if (!user.address.city) return 'No city';
  return user.address.city;
}

// แบบ Maybe
function getCityMaybe(userId) {
  return Maybe.of(users2[userId])
    .chain(user => Maybe.of(user.address))
    .chain(address => Maybe.of(address.city))
    .getOrElse('Unknown');
}

console.log(getCityMaybe(1)); // 'Bangkok'
console.log(getCityMaybe(2)); // 'Unknown' (address is null)
console.log(getCityMaybe(3)); // 'Unknown' (user is null)
console.log(getCityMaybe(4)); // 'Unknown' (no such user)

// Practical example: form validation
function validateEmail(email) {
  return Maybe.of(email)
    .filter(e => typeof e === 'string')
    .filter(e => e.length > 0)
    .filter(e => e.includes('@'))
    .filter(e => e.includes('.'))
    .map(e => e.toLowerCase().trim());
}

console.log(validateEmail('Test@Example.com').toString()); // Just("test@example.com")
console.log(validateEmail('invalid').toString()); // Nothing
console.log(validateEmail(null).toString()); // Nothing
console.log(validateEmail('').toString()); // Nothing

// ========================
// Result/Either Monad - จัดการ errors
// ========================

class Result {
  constructor(value, error = null) {
    this._value = value;
    this._error = error;
    this._isOk = error === null;
  }
  
  static ok(value) {
    return new Result(value);
  }
  
  static err(error) {
    return new Result(null, error);
  }
  
  isOk() { return this._isOk; }
  isErr() { return !this._isOk; }
  
  map(fn) {
    if (this.isErr()) return this;
    try {
      return Result.ok(fn(this._value));
    } catch (e) {
      return Result.err(e.message);
    }
  }
  
  mapErr(fn) {
    if (this.isOk()) return this;
    return Result.err(fn(this._error));
  }
  
  chain(fn) {
    if (this.isErr()) return this;
    try {
      return fn(this._value);
    } catch (e) {
      return Result.err(e.message);
    }
  }
  
  getOrElse(defaultValue) {
    return this.isOk() ? this._value : defaultValue;
  }
  
  getOrThrow() {
    if (this.isErr()) throw new Error(this._error);
    return this._value;
  }
  
  match({ ok, err }) {
    return this.isOk() ? ok(this._value) : err(this._error);
  }
  
  toString() {
    return this.isOk() 
      ? `Ok(${JSON.stringify(this._value)})` 
      : `Err(${this._error})`;
  }
}

// ใช้งาน
function parseJSON(str) {
  try {
    return Result.ok(JSON.parse(str));
  } catch (e) {
    return Result.err(`Invalid JSON: ${e.message}`);
  }
}

function validateUser(user) {
  if (!user.name) return Result.err('Name is required');
  if (!user.email) return Result.err('Email is required');
  if (!user.email.includes('@')) return Result.err('Invalid email format');
  return Result.ok(user);
}

function saveUser(user) {
  // จำลองการบันทึก
  return Result.ok({ ...user, id: Date.now(), createdAt: new Date().toISOString() });
}

// Pipeline ด้วย Result
function processUserRegistration(jsonString) {
  return parseJSON(jsonString)
    .chain(validateUser)
    .chain(saveUser)
    .match({
      ok: (user) => `สมัครสมาชิกสำเร็จ! ID: ${user.id}`,
      err: (error) => `เกิดข้อผิดพลาด: ${error}`
    });
}

console.log(processUserRegistration('{"name":"สมชาย","email":"test@example.com"}'));
console.log(processUserRegistration('{"name":"","email":"test@example.com"}'));
console.log(processUserRegistration('{invalid json}'));
```

---

## Step 1058: Recursion แทน Loops

```javascript
// ========================
// Recursion in FP
// ========================

// Loop approach (imperative)
function factorialLoop(n) {
  let result = 1;
  for (let i = 2; i <= n; i++) {
    result *= i;
  }
  return result;
}

// Recursive approach (functional)
function factorial(n) {
  if (n <= 1) return 1;
  return n * factorial(n - 1);
}

// Tail-recursive (TCO)
function factorialTail(n, acc = 1) {
  if (n <= 1) return acc;
  return factorialTail(n - 1, n * acc); // tail call
}

console.log(factorial(10));     // 3628800
console.log(factorialTail(10)); // 3628800

// ========================
// Tree Operations ด้วย Recursion
// ========================

const fileTree = {
  name: 'root',
  type: 'dir',
  children: [
    {
      name: 'src',
      type: 'dir',
      children: [
        { name: 'index.js', type: 'file', size: 1500 },
        { name: 'utils.js', type: 'file', size: 800 },
        {
          name: 'components',
          type: 'dir',
          children: [
            { name: 'Button.jsx', type: 'file', size: 600 },
            { name: 'Input.jsx', type: 'file', size: 450 }
          ]
        }
      ]
    },
    { name: 'README.md', type: 'file', size: 2000 }
  ]
};

// คำนวณขนาดทั้งหมด
function totalSize(node) {
  if (node.type === 'file') return node.size;
  return node.children.reduce((sum, child) => sum + totalSize(child), 0);
}

// ค้นหาไฟล์
function findFiles(node, predicate, path = '') {
  const currentPath = path ? `${path}/${node.name}` : node.name;
  
  if (node.type === 'file') {
    return predicate(node) ? [{ ...node, path: currentPath }] : [];
  }
  
  return node.children.flatMap(child => findFiles(child, predicate, currentPath));
}

// Transform tree
function mapTree(node, fn) {
  const transformed = fn(node);
  
  if (node.type === 'dir') {
    return {
      ...transformed,
      children: node.children.map(child => mapTree(child, fn))
    };
  }
  
  return transformed;
}

console.log('Total size:', totalSize(fileTree));

const jsFiles = findFiles(fileTree, f => f.name.endsWith('.js') || f.name.endsWith('.jsx'));
console.log('JS files:', jsFiles.map(f => f.path));

// เพิ่ม size label
const withSizeLabel = mapTree(fileTree, node => ({
  ...node,
  sizeLabel: node.type === 'file' 
    ? `${node.size} bytes` 
    : `${totalSize(node)} bytes total`
}));

// ========================
// Flatten ด้วย Recursion
// ========================

function flatten(arr) {
  return arr.reduce((flat, item) => 
    Array.isArray(item) 
      ? [...flat, ...flatten(item)] 
      : [...flat, item],
    []
  );
}

function flattenDepth(arr, depth = 1) {
  if (depth === 0) return arr;
  return arr.reduce((flat, item) => 
    Array.isArray(item) 
      ? [...flat, ...flattenDepth(item, depth - 1)] 
      : [...flat, item],
    []
  );
}

const nested = [1, [2, [3, [4, [5]]]]];
console.log(flatten(nested)); // [1, 2, 3, 4, 5]
console.log(flattenDepth(nested, 2)); // [1, 2, 3, [4, [5]]]

// ========================
// Trampoline สำหรับ Deep Recursion
// ========================

function trampoline(fn) {
  return function(...args) {
    let result = fn(...args);
    while (typeof result === 'function') {
      result = result();
    }
    return result;
  };
}

// Factorial ที่ไม่ stack overflow ด้วย trampoline
function factorialTrampoline(n, acc = 1) {
  if (n <= 1) return acc;
  return () => factorialTrampoline(n - 1, n * acc); // return function แทน recursive call
}

const safFactorial = trampoline(factorialTrampoline);
console.log(safFactorial(100000)); // ไม่ stack overflow!
```

---

## Step 1059: Lazy Evaluation

```javascript
// ========================
// Lazy Evaluation / Lazy Sequences
// ========================

// Eager (evaluated immediately)
function eagerRange(start, end) {
  const result = [];
  for (let i = start; i <= end; i++) {
    result.push(i);
  }
  return result; // สร้าง array ทั้งหมดทันที
}

// Lazy (evaluated when needed)
function* lazyRange(start, end = Infinity) {
  for (let i = start; i <= end; i++) {
    yield i;
  }
}

// Lazy operations
function* lazyMap(iterable, fn) {
  for (const item of iterable) {
    yield fn(item);
  }
}

function* lazyFilter(iterable, predicate) {
  for (const item of iterable) {
    if (predicate(item)) yield item;
  }
}

function* lazyTake(iterable, n) {
  let count = 0;
  for (const item of iterable) {
    if (count >= n) return;
    yield item;
    count++;
  }
}

function lazyToArray(iterable) {
  return [...iterable];
}

// ใช้งาน - ทำงานกับ infinite sequence!
const infiniteNumbers = lazyRange(1);
const evenNumbers = lazyFilter(infiniteNumbers, x => x % 2 === 0);
const squaredEvens = lazyMap(evenNumbers, x => x * x);
const first10 = lazyTake(squaredEvens, 10);

console.log(lazyToArray(first10)); // [4, 16, 36, 64, 100, 144, 196, 256, 324, 400]

// ========================
// Lazy Class
// ========================

class LazySequence {
  constructor(source) {
    this._source = source;
    this._operations = [];
  }
  
  static from(source) {
    return new LazySequence(source);
  }
  
  static range(start, end = Infinity) {
    return LazySequence.from(lazyRange(start, end));
  }
  
  map(fn) {
    const newSeq = new LazySequence(this._source);
    newSeq._operations = [...this._operations, { type: 'map', fn }];
    return newSeq;
  }
  
  filter(predicate) {
    const newSeq = new LazySequence(this._source);
    newSeq._operations = [...this._operations, { type: 'filter', predicate }];
    return newSeq;
  }
  
  take(n) {
    const newSeq = new LazySequence(this._source);
    newSeq._operations = [...this._operations, { type: 'take', n }];
    return newSeq;
  }
  
  *[Symbol.iterator]() {
    let source = this._source;
    
    for (const op of this._operations) {
      if (op.type === 'map') {
        source = lazyMap(source, op.fn);
      } else if (op.type === 'filter') {
        source = lazyFilter(source, op.predicate);
      } else if (op.type === 'take') {
        source = lazyTake(source, op.n);
      }
    }
    
    yield* source;
  }
  
  toArray() {
    return [...this];
  }
  
  first() {
    for (const item of this) return item;
    return undefined;
  }
  
  reduce(fn, initial) {
    let acc = initial;
    for (const item of this) {
      acc = fn(acc, item);
    }
    return acc;
  }
}

// ใช้งาน
const result2 = LazySequence.range(1)
  .filter(x => x % 3 === 0)    // หาร 3 ลงตัว
  .map(x => x * x)               // ยกกำลัง 2
  .take(10)                       // เอา 10 ตัวแรก
  .toArray();

console.log(result2); // [9, 36, 81, 144, 225, 324, 441, 576, 729, 900]

// Fibonacci lazy
function* lazyFibonacci() {
  let [a, b] = [0, 1];
  while (true) {
    yield a;
    [a, b] = [b, a + b];
  }
}

const fibSeq = LazySequence.from(lazyFibonacci())
  .filter(x => x % 2 === 0) // even fibonacci
  .take(8)
  .toArray();

console.log('Even Fibonacci:', fibSeq);
```

---

## Step 1060: FP Utilities Library

```javascript
// ========================
// Mini FP Library
// ========================

const FP = {
  // Composition
  pipe: (...fns) => x => fns.reduce((acc, fn) => fn(acc), x),
  compose: (...fns) => x => fns.reduceRight((acc, fn) => fn(acc), x),
  
  // Currying
  curry(fn) {
    return function curried(...args) {
      return args.length >= fn.length
        ? fn.apply(this, args)
        : (...more) => curried(...args, ...more);
    };
  },
  
  partial(fn, ...args) {
    return (...moreArgs) => fn(...args, ...moreArgs);
  },
  
  // Array operations
  map: FP => FP.curry((fn, arr) => arr.map(fn)),
  filter: FP => FP.curry((pred, arr) => arr.filter(pred)),
  reduce: FP => FP.curry((fn, init, arr) => arr.reduce(fn, init)),
  
  // Object operations
  prop: key => obj => obj[key],
  path: (...keys) => obj => keys.reduce((acc, key) => acc?.[key], obj),
  assoc: FP => FP.curry((key, value, obj) => ({ ...obj, [key]: value })),
  dissoc: key => obj => { const { [key]: _, ...rest } = obj; return rest; },
  
  // Logical
  and: (...fns) => x => fns.every(fn => fn(x)),
  or: (...fns) => x => fns.some(fn => fn(x)),
  not: fn => x => !fn(x),
  
  // Math
  add: FP => FP.curry((a, b) => a + b),
  subtract: FP => FP.curry((a, b) => a - b),
  multiply: FP => FP.curry((a, b) => a * b),
  divide: FP => FP.curry((a, b) => a / b),
  
  // Predicates
  equals: FP => FP.curry((a, b) => a === b),
  gt: FP => FP.curry((min, x) => x > min),
  lt: FP => FP.curry((max, x) => x < max),
  
  // String
  split: FP => FP.curry((sep, str) => str.split(sep)),
  join: FP => FP.curry((sep, arr) => arr.join(sep)),
  trim: str => str.trim(),
  toLower: str => str.toLowerCase(),
  toUpper: str => str.toUpperCase(),
  
  // Maybe
  maybe: (defaultVal) => (fn) => (value) => 
    value == null ? defaultVal : fn(value),
  
  // Identity and constant
  identity: x => x,
  constant: x => () => x,
  
  // Tap (for debugging, impure but useful)
  tap: fn => x => { fn(x); return x; }
};

// ทำให้ methods ที่ต้องการ FP ทำงานได้
Object.keys(FP).forEach(key => {
  if (typeof FP[key] === 'function' && FP[key].length > 0) {
    try {
      const result = FP[key](FP);
      if (typeof result === 'function') {
        FP[key] = result;
      }
    } catch {}
  }
});

// ใช้งาน
const { pipe, compose, curry, prop, filter, map, reduce } = FP;

const products = [
  { id: 1, name: 'Laptop', price: 45000, category: 'electronics', inStock: true },
  { id: 2, name: 'Mouse', price: 1500, category: 'electronics', inStock: true },
  { id: 3, name: 'Shirt', price: 800, category: 'clothing', inStock: false },
  { id: 4, name: 'Headphones', price: 3500, category: 'electronics', inStock: true },
  { id: 5, name: 'Book', price: 350, category: 'education', inStock: true }
];

// FP data processing
const getElectronics = filter(p => p.category === 'electronics');
const getInStock = filter(p => p.inStock);
const getPrice = prop('price');
const sumPrices = reduce((sum, p) => sum + p.price, 0);

const totalElectronicsInStock = pipe(
  getElectronics,
  getInStock,
  sumPrices
);

console.log('Total in-stock electronics:', totalElectronicsInStock(products));

// ========================
// Ramda-like Utilities
// ========================

// evolve: transform object properties with functions
function evolve(transformations, obj) {
  return Object.entries(transformations).reduce((acc, [key, fn]) => ({
    ...acc,
    [key]: fn(obj[key])
  }), { ...obj });
}

// juxtapose: apply multiple functions to same input
function juxt(...fns) {
  return (x) => fns.map(fn => fn(x));
}

// converge: apply functions to input and combine
function converge(combineFn, fns) {
  return (x) => combineFn(...fns.map(fn => fn(x)));
}

// ตัวอย่าง
const user3 = { name: ' สมชาย ', age: 17, email: 'SOMCHAI@EXAMPLE.COM' };

const normalizeUser = obj => evolve({
  name: str => str.trim(),
  age: n => n + 1,
  email: str => str.toLowerCase()
}, obj);

console.log(normalizeUser(user3));

const stats = juxt(
  arr => arr.length,
  arr => arr.reduce((a, b) => a + b, 0),
  arr => Math.max(...arr),
  arr => Math.min(...arr)
);

const [count, sum, max, min] = stats([3, 1, 4, 1, 5, 9, 2, 6]);
console.log({ count, sum, max, min });

const average = converge(
  (sum, count) => sum / count,
  [
    arr => arr.reduce((a, b) => a + b, 0),
    arr => arr.length
  ]
);

console.log('Average:', average([1, 2, 3, 4, 5])); // 3
```

---

## Step 1061-1070: สรุปและ Exercises

```javascript
// ========================
// FP Principles Summary
// ========================

// 1. Pure Functions - Referential Transparency
const add = (a, b) => a + b;
// สามารถแทนที่ add(2, 3) ด้วย 5 ได้ทุกที่

// 2. Immutability
const updateUser = (user, updates) => ({ ...user, ...updates });
// ไม่แก้ไข user เดิม

// 3. Function Composition
const processInput = pipe(
  str => str.trim(),
  str => str.toLowerCase(),
  str => str.replace(/\s+/g, '-')
);

// 4. Currying/Partial Application
const greet = curry((greeting, name) => `${greeting}, ${name}!`);
const sayHi = greet('สวัสดี');
console.log(sayHi('สมชาย'));

// 5. Higher-Order Functions
const withRetry = (fn, times = 3) => async (...args) => {
  let lastError;
  for (let i = 0; i < times; i++) {
    try {
      return await fn(...args);
    } catch (e) {
      lastError = e;
    }
  }
  throw lastError;
};

// 6. Maybe/Result for safe operations
const safeGet = (obj, key) => Maybe.of(obj).map(o => o[key]);
const safeDivide = (a, b) => b === 0 
  ? Result.err('Division by zero') 
  : Result.ok(a / b);

// 7. Recursion
const sum = (arr) => arr.length === 0 ? 0 : arr[0] + sum(arr.slice(1));
const deepMap = (fn) => (tree) => 
  Array.isArray(tree) 
    ? tree.map(deepMap(fn)) 
    : fn(tree);

// ========================
// Real-world FP Pipeline
// ========================

// ประมวลผลข้อมูลนักเรียน
const students = [
  { id: 1, name: 'สมชาย', grades: [85, 90, 78, 92, 88], class: 'A' },
  { id: 2, name: 'สมหญิง', grades: [70, 75, 80, 85, 90], class: 'B' },
  { id: 3, name: 'วิชัย', grades: [60, 55, 65, 70, 58], class: 'A' },
  { id: 4, name: 'วิไล', grades: [95, 98, 92, 96, 94], class: 'B' },
  { id: 5, name: 'มานะ', grades: [40, 45, 38, 50, 42], class: 'A' }
];

// Pure helper functions
const average = (nums) => nums.reduce((a, b) => a + b, 0) / nums.length;
const grade = (avg) => avg >= 80 ? 'A' : avg >= 70 ? 'B' : avg >= 60 ? 'C' : avg >= 50 ? 'D' : 'F';

// FP data pipeline
const processStudents = pipe(
  students => students.map(s => ({
    ...s,
    average: average(s.grades),
  })),
  students => students.map(s => ({
    ...s,
    letterGrade: grade(s.average)
  })),
  students => students.sort((a, b) => b.average - a.average),
  students => students.map((s, i) => ({
    ...s,
    rank: i + 1
  }))
);

const results = processStudents(students);

console.log('Student Rankings:');
results.forEach(s => {
  console.log(`${s.rank}. ${s.name}: ${s.average.toFixed(1)} (${s.letterGrade})`);
});

const classSummary = pipe(
  students => students.filter(s => s.class === 'A'),
  students => students.map(s => s.average),
  avgs => ({
    classAverage: average(avgs),
    highest: Math.max(...avgs),
    lowest: Math.min(...avgs)
  })
)(results);

console.log('Class A Summary:', classSummary);
```

---

## แบบฝึกหัด (Exercises)

**Exercise 1:** Pure Functions
```javascript
// แปลงฟังก์ชันต่อไปนี้ให้เป็น pure:

// Impure 1
let counter = 0;
function increment() {
  return ++counter; // แก้ให้ pure
}

// Impure 2
const userDB = [];
function addUser(name) {
  userDB.push({ name, id: userDB.length + 1 }); // แก้ให้ pure
}

// Impure 3
function formatDate() {
  return new Date().toLocaleDateString('th-TH'); // แก้ให้ pure (accept date as param)
}
```

**Exercise 2:** Composition
```javascript
// สร้าง text processing pipeline ที่:
// 1. trim ช่องว่าง
// 2. แทนที่หลายช่องว่างด้วยช่องเดียว
// 3. ทำให้เป็น Title Case
// 4. เพิ่ม prefix 'Draft: '
// 5. limit ความยาวไม่เกิน 50 chars

const processTitle = pipe(
  // TODO: implement steps
);

console.log(processTitle('  hello   world  this is a very long title here  '));
// 'Draft: Hello World This Is A Very Long...'
```

**Exercise 3:** Maybe Monad
```javascript
// ใช้ Maybe monad ดึงข้อมูลจาก nested object:
const database = {
  users: {
    1: { 
      profile: { 
        social: { twitter: '@somchai' } 
      } 
    },
    2: { profile: {} },
    3: null
  }
};

function getTwitterHandle(userId) {
  // TODO: use Maybe monad
  // database.users[userId]?.profile?.social?.twitter
  // แต่ใช้ Maybe chain แทน optional chaining
}

console.log(getTwitterHandle(1)); // '@somchai'
console.log(getTwitterHandle(2)); // null/undefined/default
console.log(getTwitterHandle(3)); // null/undefined/default
```

**Exercise 4:** Recursion
```javascript
// Implement ด้วย recursion (ห้ามใช้ loop):
// 1. flatten(arr) - flatten nested arrays
// 2. deepEqual(a, b) - เปรียบเทียบ objects/arrays แบบ deep
// 3. countOccurrences(tree, value) - นับจำนวนใน tree
// 4. generatePermutations(arr) - permutations ของ array

function flatten(arr) { /* TODO */ }
function deepEqual(a, b) { /* TODO */ }
```

---

## สรุป Functional Programming

| หลักการ | ทำ | หลีกเลี่ยง |
|---------|-----|---------|
| Pure Functions | same input → same output | global state, I/O |
| Immutability | spread, map, filter | push, splice, assignment |
| Composition | pipe, compose | if-else chains |
| Currying | specialized functions | over-parameterization |
| Maybe/Result | safe null handling | try/catch everywhere |
| Recursion | tree traversal | deep loops |

**ขั้นตอนต่อไป:** ใน Part 55 เราจะเรียนรู้ **Reactive Programming** และ RxJS
