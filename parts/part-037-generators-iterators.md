# Part 37: Generators และ Iterators (Steps 711-730)

## บทนำ

**Iterator** และ **Generator** เป็น pattern ที่ทรงพลังใน JavaScript สำหรับการจัดการกับ sequences ของข้อมูล ไม่ว่าจะเป็นข้อมูลจำนวนจำกัดหรืออนันต์

- **Iterator**: protocol ที่กำหนดว่า object สามารถ "iterate" ได้อย่างไร
- **Generator**: ฟังก์ชันพิเศษที่หยุดการทำงาน (pause) และกลับมาทำงานต่อได้ (resume)

---

## Step 711: Iterable Protocol

object ที่ "iterable" ต้องมี method `[Symbol.iterator]()` ที่คืน Iterator

```javascript
// Built-in iterables
const arr = [1, 2, 3];
const str = "hello";
const map = new Map([["a", 1], ["b", 2]]);
const set = new Set([1, 2, 3]);

// ทุก iterable สามารถใช้กับ for-of ได้
for (const x of arr) console.log(x);  // 1, 2, 3
for (const c of str) console.log(c);  // h, e, l, l, o

// ตรวจสอบว่า iterable หรือไม่
function isIterable(obj) {
  return obj != null && typeof obj[Symbol.iterator] === "function";
}

console.log(isIterable([1, 2, 3]));   // true
console.log(isIterable("hello"));      // true
console.log(isIterable(new Map()));    // true
console.log(isIterable(new Set()));    // true
console.log(isIterable(42));           // false
console.log(isIterable({ a: 1 }));    // false
```

```javascript
// Iterables สนับสนุน spread, destructuring, Array.from
const [first, second, ...rest] = "hello";
console.log(first);  // 'h'
console.log(second); // 'e'
console.log(rest);   // ['l', 'l', 'o']

const chars = [...new Set("mississippi")];
console.log(chars); // ['m', 'i', 's', 'p']

const arr = Array.from(new Map([["a", 1], ["b", 2]]));
console.log(arr); // [['a', 1], ['b', 2]]
```

---

## Step 712: Iterator Protocol

**Iterator** ต้องมี method `next()` ที่คืน object `{ value, done }`

```javascript
// สร้าง iterator ด้วยมือ
function createRangeIterator(start, end, step = 1) {
  let current = start;
  
  return {
    next() {
      if (current <= end) {
        const value = current;
        current += step;
        return { value, done: false };
      }
      return { value: undefined, done: true };
    }
  };
}

const iter = createRangeIterator(1, 5);
console.log(iter.next()); // { value: 1, done: false }
console.log(iter.next()); // { value: 2, done: false }
console.log(iter.next()); // { value: 3, done: false }
console.log(iter.next()); // { value: 4, done: false }
console.log(iter.next()); // { value: 5, done: false }
console.log(iter.next()); // { value: undefined, done: true }
console.log(iter.next()); // { value: undefined, done: true }
```

```javascript
// Iterator กับ Array (built-in iterator)
const arr = ["a", "b", "c"];
const iterator = arr[Symbol.iterator]();

console.log(iterator.next()); // { value: 'a', done: false }
console.log(iterator.next()); // { value: 'b', done: false }
console.log(iterator.next()); // { value: 'c', done: false }
console.log(iterator.next()); // { value: undefined, done: true }
```

---

## Step 713: Built-in Iterables

```javascript
// 1. Array iterator
const arr = [10, 20, 30];
for (const val of arr) console.log(val); // 10, 20, 30

// 2. String iterator (Unicode-aware)
const emoji = "😀😂🎉";
console.log(emoji.length); // 6 (byte length)
console.log([...emoji].length); // 3 (character count)

for (const char of emoji) {
  console.log(char); // 😀, 😂, 🎉 (ถูกต้อง)
}

// 3. Map iterator
const map = new Map([["a", 1], ["b", 2]]);
for (const [key, val] of map) {
  console.log(`${key}: ${val}`);
}

// 4. Set iterator
const set = new Set([1, 2, 3]);
for (const val of set) console.log(val);

// 5. arguments object
function example() {
  for (const arg of arguments) {
    console.log(arg);
  }
}
example(1, 2, 3); // 1, 2, 3

// 6. NodeList (in browser)
// document.querySelectorAll('div') is iterable
```

```javascript
// String iterator รองรับ surrogate pairs
const str = "Hello 👋 World";
const chars = [...str];
console.log(chars.length); // 13 (ถูกต้อง รวม emoji เป็น 1)

// แบบเก่า (ผิด)
console.log(str.split("").length); // 15 (emoji ถูกแยกเป็น 2 chars)
```

---

## Step 714: Custom Iterables ด้วย Symbol.iterator

```javascript
// สร้าง iterable object
const range = {
  from: 1,
  to: 5,
  
  [Symbol.iterator]() {
    let current = this.from;
    const last = this.to;
    
    return {
      next() {
        if (current <= last) {
          return { value: current++, done: false };
        }
        return { value: undefined, done: true };
      }
    };
  }
};

for (const num of range) {
  console.log(num); // 1, 2, 3, 4, 5
}

console.log([...range]); // [1, 2, 3, 4, 5]
const [a, b, c] = range;
console.log(a, b, c); // 1 2 3
```

```javascript
// Self-iterable (iterator คือ iterable ของตัวเอง)
function createCounter(max) {
  let count = 0;
  
  const iterator = {
    next() {
      if (count < max) {
        return { value: count++, done: false };
      }
      return { value: undefined, done: true };
    },
    
    [Symbol.iterator]() {
      return this;
    }
  };
  
  return iterator;
}

const counter = createCounter(3);
const [x, y, z] = counter; // ใช้ destructuring ได้
console.log(x, y, z); // 0 1 2
```

```javascript
// Iterable class
class LinkedList {
  #head = null;
  #size = 0;
  
  push(value) {
    this.#head = { value, next: this.#head };
    this.#size++;
    return this;
  }
  
  get size() {
    return this.#size;
  }
  
  [Symbol.iterator]() {
    let current = this.#head;
    const list = [];
    
    // เก็บ values ทั้งหมดก่อน (เพราะ linked list เก็บ reverse)
    while (current) {
      list.unshift(current.value);
      current = current.next;
    }
    
    let index = 0;
    return {
      next() {
        if (index < list.length) {
          return { value: list[index++], done: false };
        }
        return { value: undefined, done: true };
      }
    };
  }
}

const list = new LinkedList();
list.push(1).push(2).push(3);

for (const val of list) {
  console.log(val); // 1, 2, 3
}

console.log([...list]); // [1, 2, 3]
```

---

## Step 715: Generators - function* Syntax

```javascript
// Generator function ใช้ function* syntax
function* simpleGenerator() {
  console.log("Start");
  yield 1;
  console.log("After first yield");
  yield 2;
  console.log("After second yield");
  yield 3;
  console.log("End");
}

const gen = simpleGenerator();
console.log(gen.next()); // Start -> { value: 1, done: false }
console.log(gen.next()); // After first yield -> { value: 2, done: false }
console.log(gen.next()); // After second yield -> { value: 3, done: false }
console.log(gen.next()); // End -> { value: undefined, done: true }
```

```javascript
// Generator เป็นทั้ง Iterator และ Iterable
function* counter() {
  yield 1;
  yield 2;
  yield 3;
}

const gen = counter();

// ใช้ as Iterator
console.log(gen.next()); // { value: 1, done: false }

// ใช้ as Iterable (for-of)
for (const val of counter()) {
  console.log(val); // 1, 2, 3
}

// spread
console.log([...counter()]); // [1, 2, 3]

// destructuring
const [a, b, c] = counter();
console.log(a, b, c); // 1 2 3
```

```javascript
// Generator method ใน class
class NumberRange {
  constructor(start, end) {
    this.start = start;
    this.end = end;
  }
  
  *[Symbol.iterator]() {
    for (let i = this.start; i <= this.end; i++) {
      yield i;
    }
  }
  
  *evens() {
    for (const num of this) {
      if (num % 2 === 0) yield num;
    }
  }
  
  *odds() {
    for (const num of this) {
      if (num % 2 !== 0) yield num;
    }
  }
}

const range = new NumberRange(1, 10);
console.log([...range]);          // [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
console.log([...range.evens()]);  // [2, 4, 6, 8, 10]
console.log([...range.odds()]);   // [1, 3, 5, 7, 9]
```

---

## Step 716: yield Keyword

```javascript
// yield หยุดการทำงานและส่งค่าออก
function* fibonacci() {
  let [prev, curr] = [0, 1];
  
  while (true) {
    yield curr;
    [prev, curr] = [curr, prev + curr];
  }
}

const fib = fibonacci();
for (let i = 0; i < 8; i++) {
  process.stdout.write(fib.next().value + " ");
}
// 1 1 2 3 5 8 13 21

// ใช้ helper function เพื่อดึงค่า n ตัวแรก
function take(iterable, n) {
  const result = [];
  for (const item of iterable) {
    result.push(item);
    if (result.length >= n) break;
  }
  return result;
}

console.log(take(fibonacci(), 10)); // [1, 1, 2, 3, 5, 8, 13, 21, 34, 55]
```

```javascript
// yield expressions
function* loggedGenerator() {
  const x = yield "What is x?";
  console.log(`x = ${x}`);
  
  const y = yield "What is y?";
  console.log(`y = ${y}`);
  
  yield `Sum is ${x + y}`;
}

const gen = loggedGenerator();
console.log(gen.next());        // { value: 'What is x?', done: false }
console.log(gen.next(10));      // x = 10, { value: 'What is y?', done: false }
console.log(gen.next(20));      // y = 20, { value: 'Sum is 30', done: false }
console.log(gen.next());        // { value: undefined, done: true }
```

---

## Step 717: Generator Return Value

```javascript
// Generator return
function* generatorWithReturn() {
  yield 1;
  yield 2;
  return "done!"; // return value
  yield 3;        // ไม่ถูก execute
}

const gen = generatorWithReturn();
console.log(gen.next()); // { value: 1, done: false }
console.log(gen.next()); // { value: 2, done: false }
console.log(gen.next()); // { value: 'done!', done: true }
console.log(gen.next()); // { value: undefined, done: true }

// for-of ไม่รับ return value (done: true ถือว่าจบ)
for (const val of generatorWithReturn()) {
  console.log(val); // 1, 2 (ไม่มี 'done!')
}
```

```javascript
// gen.return() - บังคับจบ generator
function* counter() {
  let i = 0;
  while (true) {
    yield i++;
  }
}

const gen = counter();
console.log(gen.next());        // { value: 0, done: false }
console.log(gen.next());        // { value: 1, done: false }
console.log(gen.return("bye")); // { value: 'bye', done: true }
console.log(gen.next());        // { value: undefined, done: true }
```

```javascript
// finally ใน generator
function* withCleanup() {
  try {
    yield 1;
    yield 2;
    yield 3;
  } finally {
    console.log("Cleanup!"); // จะถูก run เสมอ
  }
}

const gen = withCleanup();
console.log(gen.next()); // { value: 1, done: false }
gen.return("early end"); // "Cleanup!" ถูก run ก่อน return
```

---

## Step 718: yield* Delegation

```javascript
// yield* delegate ไปยัง iterable อื่น
function* inner() {
  yield "a";
  yield "b";
  yield "c";
}

function* outer() {
  yield 1;
  yield* inner(); // delegate ไปยัง inner
  yield 2;
}

console.log([...outer()]); // [1, 'a', 'b', 'c', 2]

// yield* กับ array
function* combined() {
  yield* [1, 2, 3];
  yield* "hello";
  yield* new Set([4, 5, 6]);
}

console.log([...combined()]); // [1, 2, 3, 'h', 'e', 'l', 'l', 'o', 4, 5, 6]
```

```javascript
// yield* คืน return value ของ sub-generator
function* subGen() {
  yield 1;
  yield 2;
  return "sub-done"; // return value
}

function* mainGen() {
  const result = yield* subGen(); // result = "sub-done"
  console.log(`Sub-generator returned: ${result}`);
  yield 3;
}

const gen = mainGen();
console.log(gen.next()); // { value: 1, done: false }
console.log(gen.next()); // { value: 2, done: false }
console.log(gen.next()); // "Sub-generator returned: sub-done" -> { value: 3, done: false }
console.log(gen.next()); // { value: undefined, done: true }
```

```javascript
// ตัวอย่าง: flatten nested iterables
function* flatten(iterable, depth = Infinity) {
  for (const item of iterable) {
    if (depth > 0 && typeof item[Symbol.iterator] === "function" && typeof item !== "string") {
      yield* flatten(item, depth - 1);
    } else {
      yield item;
    }
  }
}

const nested = [1, [2, 3], [4, [5, 6]], [[7, [8, 9]]]];
console.log([...flatten(nested)]);      // [1, 2, 3, 4, 5, 6, 7, 8, 9]
console.log([...flatten(nested, 1)]);   // [1, 2, 3, 4, [5, 6], [7, [8, 9]]]
```

---

## Step 719: Two-Way Communication (Passing Values to yield)

```javascript
// ส่งค่าเข้าไปใน generator ผ่าน next(value)
function* calculator() {
  let result = 0;
  
  while (true) {
    const input = yield result;
    
    if (input === null) break;
    
    if (typeof input === "object") {
      const { op, num } = input;
      if (op === "+") result += num;
      else if (op === "-") result -= num;
      else if (op === "*") result *= num;
      else if (op === "/") result /= num;
    }
  }
  
  return result;
}

const calc = calculator();
calc.next();                          // เริ่ม generator, result = 0
console.log(calc.next({ op: "+", num: 10 }).value); // 10
console.log(calc.next({ op: "*", num: 3 }).value);  // 30
console.log(calc.next({ op: "-", num: 5 }).value);  // 25
console.log(calc.next({ op: "/", num: 5 }).value);  // 5
console.log(calc.next(null));                        // { value: 5, done: true }
```

```javascript
// ตัวอย่าง: state machine
function* trafficLight() {
  while (true) {
    console.log("🔴 Red");
    yield "red";
    
    console.log("🟡 Yellow");
    yield "yellow";
    
    console.log("🟢 Green");
    yield "green";
  }
}

const light = trafficLight();
console.log(light.next().value); // 🔴 Red -> "red"
console.log(light.next().value); // 🟡 Yellow -> "yellow"
console.log(light.next().value); // 🟢 Green -> "green"
console.log(light.next().value); // 🔴 Red -> "red" (วนซ้ำ)
```

---

## Step 720: Generator throw() Method

```javascript
// gen.throw() - ส่ง error เข้าไปใน generator
function* safeGenerator() {
  try {
    yield 1;
    yield 2;
    yield 3;
  } catch (error) {
    console.log(`Caught: ${error.message}`);
    yield "recovered";
  }
}

const gen = safeGenerator();
console.log(gen.next());             // { value: 1, done: false }
console.log(gen.throw(new Error("Oops!"))); // "Caught: Oops!" -> { value: 'recovered', done: false }
console.log(gen.next());             // { value: undefined, done: true }
```

```javascript
// ตัวอย่าง: Validation generator
function* validateData() {
  while (true) {
    const data = yield;
    
    try {
      if (typeof data !== "object") {
        throw new TypeError("Expected object");
      }
      if (!data.name || typeof data.name !== "string") {
        throw new Error("Invalid name");
      }
      if (!data.age || data.age < 0 || data.age > 150) {
        throw new Error("Invalid age");
      }
      
      yield { valid: true, data };
    } catch (error) {
      yield { valid: false, error: error.message };
    }
  }
}

const validator = validateData();
validator.next(); // เริ่ม

validator.next({ name: "สมชาย", age: 30 });
console.log(validator.next().value); // { valid: true, data: { name: 'สมชาย', age: 30 } }

validator.next({ name: "", age: 25 });
console.log(validator.next().value); // { valid: false, error: 'Invalid name' }
```

---

## Step 721: Infinite Generators

```javascript
// Infinite counter
function* infiniteCounter(start = 0, step = 1) {
  let current = start;
  while (true) {
    yield current;
    current += step;
  }
}

// ดึงค่า 5 ตัวแรก
const counter = infiniteCounter(0, 2);
for (let i = 0; i < 5; i++) {
  console.log(counter.next().value);
}
// 0, 2, 4, 6, 8

// Infinite prime numbers
function* primes() {
  const sieve = new Map();
  let num = 2;
  
  while (true) {
    if (!sieve.has(num)) {
      yield num;
      sieve.set(num * num, [num]);
    } else {
      const primeFactors = sieve.get(num);
      primeFactors.forEach((prime) => {
        const next = num + prime;
        if (sieve.has(next)) {
          sieve.get(next).push(prime);
        } else {
          sieve.set(next, [prime]);
        }
      });
      sieve.delete(num);
    }
    num++;
  }
}

const primeGen = primes();
const first10Primes = Array.from({ length: 10 }, () => primeGen.next().value);
console.log(first10Primes); // [2, 3, 5, 7, 11, 13, 17, 19, 23, 29]
```

```javascript
// Infinite sequence combinators
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

function* take(iterable, n) {
  let count = 0;
  for (const item of iterable) {
    if (count++ >= n) break;
    yield item;
  }
}

function* zip(...iterables) {
  const iters = iterables.map((it) => it[Symbol.iterator]());
  
  while (true) {
    const results = iters.map((it) => it.next());
    if (results.some((r) => r.done)) break;
    yield results.map((r) => r.value);
  }
}

// ใช้งาน
const evenSquares = filter(
  map(infiniteCounter(), (n) => n * n),
  (n) => n % 2 === 0
);

console.log([...take(evenSquares, 5)]); // [0, 4, 16, 36, 64]

// zip สอง sequences
const counts = infiniteCounter(1);
const letters = function* () { yield* "abcdefghij"; }();
const zipped = [...take(zip(counts, letters), 5)];
console.log(zipped); // [[1, 'a'], [2, 'b'], [3, 'c'], [4, 'd'], [5, 'e']]
```

---

## Step 722: Lazy Evaluation กับ Generators

```javascript
// Lazy evaluation: คำนวณเมื่อต้องการเท่านั้น

// Eager (คำนวณทั้งหมดก่อน)
function getEagerPipeline(data) {
  return data
    .filter((x) => x > 0)       // สร้าง array ใหม่
    .map((x) => x * x)           // สร้าง array ใหม่
    .filter((x) => x < 100);    // สร้าง array ใหม่
}

// Lazy (คำนวณเมื่อดึงค่า)
function* lazyFilter(iterable, predicate) {
  for (const item of iterable) {
    if (predicate(item)) yield item;
  }
}

function* lazyMap(iterable, fn) {
  for (const item of iterable) {
    yield fn(item);
  }
}

function getLazyPipeline(data) {
  return lazyFilter(
    lazyMap(
      lazyFilter(data, (x) => x > 0),
      (x) => x * x
    ),
    (x) => x < 100
  );
}

const data = [-1, 2, -3, 4, 5, 6, -7, 8, 9, 10];

// Lazy: ดึงแค่ตัวแรก ไม่ต้องคำนวณทั้งหมด
const lazyResult = getLazyPipeline(data);
console.log(lazyResult.next().value); // 4 (ดึงแค่ตัวแรก)
```

```javascript
// ตัวอย่าง: Lazy file reader (แนวคิด)
function* readLines(text) {
  let start = 0;
  
  while (start < text.length) {
    const end = text.indexOf("\n", start);
    if (end === -1) {
      yield text.slice(start);
      break;
    }
    yield text.slice(start, end);
    start = end + 1;
  }
}

const csv = `name,age,city
สมชาย,30,กรุงเทพ
สมศรี,25,เชียงใหม่
วิภา,28,ภูเก็ต`;

const lines = readLines(csv);
const header = lines.next().value.split(",");

for (const line of lines) {
  const values = line.split(",");
  const record = Object.fromEntries(header.map((h, i) => [h, values[i]]));
  console.log(record);
}
```

```javascript
// ตัวอย่าง: Pagination generator
function* paginate(fetchFn, pageSize = 10) {
  let page = 1;
  let hasMore = true;
  
  while (hasMore) {
    const items = fetchFn(page, pageSize);
    
    if (items.length === 0) {
      hasMore = false;
      break;
    }
    
    for (const item of items) {
      yield item;
    }
    
    hasMore = items.length === pageSize;
    page++;
  }
}

// Mock data
const allData = Array.from({ length: 25 }, (_, i) => ({ id: i + 1, name: `Item ${i + 1}` }));

function mockFetch(page, size) {
  const start = (page - 1) * size;
  return allData.slice(start, start + size);
}

let count = 0;
for (const item of paginate(mockFetch, 10)) {
  count++;
  if (count <= 3 || count > 23) console.log(item);
}
console.log(`Total: ${count} items`); // Total: 25 items
```

---

## Step 723: Async Generators (async function*)

```javascript
// Async generator รวม async/await กับ generator
async function* asyncCounter(max, delay = 100) {
  for (let i = 0; i < max; i++) {
    // จำลอง async operation
    await new Promise((resolve) => setTimeout(resolve, delay));
    yield i;
  }
}

// ใช้กับ for-await-of
async function main() {
  for await (const num of asyncCounter(5, 100)) {
    console.log(`Received: ${num}`);
  }
  console.log("Done!");
}

main();
// Received: 0
// Received: 1
// Received: 2
// Received: 3
// Received: 4
// Done!
```

```javascript
// Async generator สำหรับ API pagination
async function* fetchPages(baseUrl, pageSize = 10) {
  let page = 1;
  
  while (true) {
    const response = await fetch(`${baseUrl}?page=${page}&size=${pageSize}`);
    
    if (!response.ok) throw new Error(`HTTP error: ${response.status}`);
    
    const data = await response.json();
    
    if (data.items.length === 0) break;
    
    for (const item of data.items) {
      yield item;
    }
    
    if (!data.hasMore) break;
    page++;
  }
}

// ใช้งาน (สมมติ)
async function processAllUsers() {
  let count = 0;
  
  for await (const user of fetchPages("/api/users")) {
    console.log(`Processing user: ${user.name}`);
    count++;
  }
  
  console.log(`Processed ${count} users`);
}
```

```javascript
// Async generator สำหรับ data stream
async function* splitByLine(asyncIterable) {
  let buffer = "";
  
  for await (const chunk of asyncIterable) {
    buffer += chunk;
    
    let newlineIndex;
    while ((newlineIndex = buffer.indexOf("\n")) >= 0) {
      yield buffer.slice(0, newlineIndex);
      buffer = buffer.slice(newlineIndex + 1);
    }
  }
  
  if (buffer.length > 0) yield buffer;
}

// Mock async stream
async function* mockStream() {
  yield "Hello, ";
  yield "World!\n";
  yield "Second ";
  yield "line\n";
  yield "Third line";
}

async function processStream() {
  for await (const line of splitByLine(mockStream())) {
    console.log(`Line: "${line}"`);
  }
}

processStream();
// Line: "Hello, World!"
// Line: "Second line"
// Line: "Third line"
```

---

## Step 724: for-await-of กับ Async Generators

```javascript
// for-await-of ทำงานกับ async iterables
async function* asyncRange(start, end, delay = 0) {
  for (let i = start; i <= end; i++) {
    if (delay > 0) {
      await new Promise((r) => setTimeout(r, delay));
    }
    yield i;
  }
}

// Basic usage
async function example1() {
  for await (const num of asyncRange(1, 5)) {
    console.log(num);
  }
}

// with break
async function example2() {
  for await (const num of asyncRange(1, 100)) {
    if (num > 3) break; // หยุดได้
    console.log(num);
  }
}
```

```javascript
// ตัวอย่าง: อ่านไฟล์ขนาดใหญ่ทีละ chunk
async function* readFileChunks(filePath, chunkSize = 1024) {
  // ใน Node.js จริงจะใช้ fs.createReadStream
  // นี่เป็นตัวอย่างแนวคิด
  const { createReadStream } = require("fs");
  const stream = createReadStream(filePath, { highWaterMark: chunkSize });
  
  for await (const chunk of stream) {
    yield chunk;
  }
}

// ตัวอย่าง: นับคำในไฟล์ขนาดใหญ่โดยไม่โหลดทั้งหมดเข้า memory
async function countWordsInFile(filePath) {
  let count = 0;
  let remainder = "";
  
  for await (const chunk of readFileChunks(filePath)) {
    const text = remainder + chunk.toString();
    const words = text.split(/\s+/);
    remainder = words.pop(); // เก็บคำที่ยังไม่สมบูรณ์
    count += words.filter(Boolean).length;
  }
  
  if (remainder.trim()) count++;
  return count;
}
```

---

## Step 725: Practical Use Case - Data Pipeline

```javascript
// สร้าง pipeline system ด้วย generators
function pipeline(...fns) {
  return function*(source) {
    let current = source;
    for (const fn of fns) {
      current = fn(current);
    }
    yield* current;
  };
}

// Utility generators
function* mapGen(iterable, fn) {
  for (const item of iterable) yield fn(item);
}

function* filterGen(iterable, fn) {
  for (const item of iterable) {
    if (fn(item)) yield item;
  }
}

function* takeGen(iterable, n) {
  let count = 0;
  for (const item of iterable) {
    if (count++ >= n) break;
    yield item;
  }
}

function* flatMapGen(iterable, fn) {
  for (const item of iterable) {
    yield* fn(item);
  }
}

// สร้าง data pipeline
const numbers = function*() {
  let i = 1;
  while (true) yield i++;
}();

const process = pipeline(
  (src) => filterGen(src, (n) => n % 2 === 0),     // เอาเลขคู่
  (src) => mapGen(src, (n) => n * n),                // ยกกำลัง 2
  (src) => filterGen(src, (n) => n.toString().includes("6")), // มีเลข 6
  (src) => takeGen(src, 5)                           // เอา 5 ตัวแรก
);

console.log([...process(numbers)]);
// [16, 36, 64, 196, 256, ...] (เลขคู่ยกกำลัง 2 ที่มีเลข 6)
```

---

## Step 726: Generator สำหรับ Coroutines

```javascript
// Coroutines: หลาย generator ทำงานร่วมกัน
class Scheduler {
  #tasks = [];
  
  addTask(generator) {
    this.#tasks.push(generator);
    return this;
  }
  
  run() {
    while (this.#tasks.length > 0) {
      const nextTasks = [];
      
      for (const task of this.#tasks) {
        const { done } = task.next();
        if (!done) nextTasks.push(task);
      }
      
      this.#tasks = nextTasks;
    }
  }
}

function* task1() {
  console.log("Task 1: Step 1");
  yield;
  console.log("Task 1: Step 2");
  yield;
  console.log("Task 1: Step 3");
}

function* task2() {
  console.log("Task 2: Step A");
  yield;
  console.log("Task 2: Step B");
}

function* task3() {
  console.log("Task 3: I");
  yield;
  console.log("Task 3: II");
  yield;
  console.log("Task 3: III");
  yield;
  console.log("Task 3: IV");
}

const scheduler = new Scheduler();
scheduler.addTask(task1()).addTask(task2()).addTask(task3()).run();
// Task 1: Step 1
// Task 2: Step A
// Task 3: I
// Task 1: Step 2
// Task 2: Step B
// Task 3: II
// Task 1: Step 3
// Task 3: III
// Task 3: IV
```

---

## Step 727: Generator สำหรับ Stateful Iterations

```javascript
// Stateful cursor
class DataCursor {
  #data;
  #generator;
  #current = undefined;
  #done = false;
  
  constructor(data) {
    this.#data = data;
    this.#generator = this.#createGenerator();
    this.advance(); // go to first item
  }
  
  *#createGenerator() {
    for (const item of this.#data) {
      yield item;
    }
  }
  
  advance() {
    const { value, done } = this.#generator.next();
    this.#current = value;
    this.#done = done;
    return !done;
  }
  
  get current() {
    return this.#current;
  }
  
  get isDone() {
    return this.#done;
  }
  
  *[Symbol.iterator]() {
    while (!this.#done) {
      yield this.#current;
      this.advance();
    }
  }
}

const cursor = new DataCursor([1, 2, 3, 4, 5]);
console.log(cursor.current); // 1
cursor.advance();
console.log(cursor.current); // 2

// หรือ iterate ทั้งหมด
const cursor2 = new DataCursor(["a", "b", "c"]);
for (const item of cursor2) {
  console.log(item); // a, b, c
}
```

---

## Step 728: Error Handling ใน Generators

```javascript
// Error handling pattern
function* robustGenerator(data) {
  for (const item of data) {
    try {
      if (typeof item !== "number") {
        throw new TypeError(`Expected number, got ${typeof item}`);
      }
      yield item * 2;
    } catch (error) {
      console.error(`Skipping "${item}": ${error.message}`);
      yield null; // yield null แทน skip
    }
  }
}

const data = [1, 2, "three", 4, null, 5];
const gen = robustGenerator(data);

for (const result of gen) {
  if (result !== null) {
    console.log(result);
  }
}
// 2, 4, Skipping "three": ..., 8, Skipping "null": ..., 10
```

```javascript
// Generator wrapper สำหรับ retry logic
async function* withRetry(asyncGen, maxRetries = 3) {
  for await (const item of asyncGen) {
    let lastError;
    
    for (let attempt = 1; attempt <= maxRetries; attempt++) {
      try {
        // สมมติ process item
        const result = await processItem(item);
        yield result;
        break;
      } catch (error) {
        lastError = error;
        console.log(`Retry ${attempt}/${maxRetries} for item ${item.id}`);
        await new Promise((r) => setTimeout(r, 1000 * attempt));
      }
    }
  }
}
```

---

## Step 729: Combining Generators กับ Promises

```javascript
// ใช้ Generator สำหรับ async flow control
function run(generatorFn) {
  const gen = generatorFn();
  
  function handle(result) {
    if (result.done) return Promise.resolve(result.value);
    
    return Promise.resolve(result.value)
      .then(
        (value) => handle(gen.next(value)),
        (error) => handle(gen.throw(error))
      );
  }
  
  return handle(gen.next());
}

// ใช้งาน (ก่อน async/await)
function* fetchUserData(userId) {
  const user = yield fetch(`/api/users/${userId}`).then((r) => r.json());
  const posts = yield fetch(`/api/posts?userId=${userId}`).then((r) => r.json());
  
  return { user, posts };
}

run(fetchUserData.bind(null, 1))
  .then((data) => console.log(data))
  .catch(console.error);
```

```javascript
// ตัวอย่าง: Observable-like stream ด้วย generator
function* interval(ms) {
  let tick = 0;
  while (true) {
    yield new Promise((resolve) => {
      setTimeout(() => resolve(tick++), ms);
    });
  }
}

async function listenToInterval(gen, count) {
  let received = 0;
  for await (const tick of gen) {
    console.log(`Tick: ${tick}`);
    if (++received >= count) break;
  }
}

// listenToInterval(interval(500), 5);
```

---

## Step 730: Real-world Examples

### Example 1: Parser Generator

```javascript
function* tokenize(input) {
  const patterns = [
    { type: "NUMBER", regex: /^\d+(\.\d+)?/ },
    { type: "STRING", regex: /^"[^"]*"/ },
    { type: "IDENTIFIER", regex: /^[a-zA-Z_]\w*/ },
    { type: "OPERATOR", regex: /^[+\-*/=<>!&|]/ },
    { type: "PUNCTUATION", regex: /^[(){}[\],;]/ },
    { type: "WHITESPACE", regex: /^\s+/ },
  ];
  
  let position = 0;
  
  while (position < input.length) {
    let matched = false;
    
    for (const { type, regex } of patterns) {
      const remaining = input.slice(position);
      const match = remaining.match(regex);
      
      if (match) {
        if (type !== "WHITESPACE") {
          yield { type, value: match[0], position };
        }
        position += match[0].length;
        matched = true;
        break;
      }
    }
    
    if (!matched) {
      throw new SyntaxError(`Unexpected character: ${input[position]}`);
    }
  }
}

const code = 'let x = 42 + "hello"';
for (const token of tokenize(code)) {
  console.log(token);
}
// { type: 'IDENTIFIER', value: 'let', position: 0 }
// { type: 'IDENTIFIER', value: 'x', position: 4 }
// { type: 'OPERATOR', value: '=', position: 6 }
// { type: 'NUMBER', value: '42', position: 8 }
// { type: 'OPERATOR', value: '+', position: 11 }
// { type: 'STRING', value: '"hello"', position: 13 }
```

### Example 2: Tree Traversal

```javascript
// Binary tree node
class TreeNode {
  constructor(value, left = null, right = null) {
    this.value = value;
    this.left = left;
    this.right = right;
  }
}

// สร้าง tree
const tree = new TreeNode(
  1,
  new TreeNode(2, new TreeNode(4), new TreeNode(5)),
  new TreeNode(3, new TreeNode(6), new TreeNode(7))
);

// In-order traversal (left, root, right)
function* inOrder(node) {
  if (!node) return;
  yield* inOrder(node.left);
  yield node.value;
  yield* inOrder(node.right);
}

// Pre-order traversal (root, left, right)
function* preOrder(node) {
  if (!node) return;
  yield node.value;
  yield* preOrder(node.left);
  yield* preOrder(node.right);
}

// Level-order traversal (BFS)
function* levelOrder(root) {
  if (!root) return;
  const queue = [root];
  
  while (queue.length > 0) {
    const node = queue.shift();
    yield node.value;
    
    if (node.left) queue.push(node.left);
    if (node.right) queue.push(node.right);
  }
}

console.log([...inOrder(tree)]);   // [4, 2, 5, 1, 6, 3, 7]
console.log([...preOrder(tree)]);  // [1, 2, 4, 5, 3, 6, 7]
console.log([...levelOrder(tree)]); // [1, 2, 3, 4, 5, 6, 7]
```

---

## แบบฝึกหัด

### Easy
1. สร้าง generator `range(start, end, step)` ที่ทำงานเหมือน Python's range
2. สร้าง generator ที่คืน fibonacci sequence ทีละตัว
3. Implement `zip(iter1, iter2)` generator

### Medium
4. สร้าง `roundRobin(...iterables)` generator ที่วน yield จาก iterables ทีละตัว
5. Implement async generator สำหรับ rate-limited API calls
6. สร้าง generator-based state machine

### Hard
7. Implement `Observable` class โดยใช้ generators
8. สร้าง async generator pipeline สำหรับ ETL process
9. Implement coroutine scheduler ที่รองรับ priority

### Solution ตัวอย่าง

```javascript
// 1. range generator
function* range(start, end, step = 1) {
  if (end === undefined) {
    end = start;
    start = 0;
  }
  
  if (step > 0) {
    for (let i = start; i < end; i += step) yield i;
  } else {
    for (let i = start; i > end; i += step) yield i;
  }
}

console.log([...range(5)]);       // [0, 1, 2, 3, 4]
console.log([...range(2, 8)]);    // [2, 3, 4, 5, 6, 7]
console.log([...range(0, 10, 2)]); // [0, 2, 4, 6, 8]
console.log([...range(5, 0, -1)]); // [5, 4, 3, 2, 1]

// 4. roundRobin
function* roundRobin(...iterables) {
  const iters = iterables.map((it) => it[Symbol.iterator]());
  
  while (iters.length > 0) {
    let i = 0;
    while (i < iters.length) {
      const { value, done } = iters[i].next();
      if (done) {
        iters.splice(i, 1);
      } else {
        yield value;
        i++;
      }
    }
  }
}

console.log([...roundRobin([1, 2, 3], ["a", "b"], [true])]);
// [1, 'a', true, 2, 'b', 3]
```

---

## สรุป

| Concept | Description |
|---------|-------------|
| Iterable | Object ที่มี `[Symbol.iterator]()` |
| Iterator | Object ที่มี `next()` คืน `{value, done}` |
| Generator | ฟังก์ชันที่ใช้ `function*` และ `yield` |
| `yield` | หยุดการทำงานและส่งค่าออก |
| `yield*` | delegate ไปยัง iterable อื่น |
| Async generator | `async function*` กับ `for-await-of` |
| Lazy evaluation | คำนวณเมื่อต้องการเท่านั้น |

**เมื่อไหร่ควรใช้ Generators:**
- ต้องการ lazy evaluation
- สร้าง infinite sequences
- ต้องการ pausable computation
- สร้าง data pipelines
- Async data streams
