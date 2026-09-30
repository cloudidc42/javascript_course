# Part 36: Map, Set, WeakMap, WeakSet (Steps 691-710)

## บทนำ

ใน JavaScript นอกจาก Object และ Array ที่เราใช้กันเป็นประจำแล้ว ยังมี Data Structure อื่นที่ทรงพลังมากสำหรับงานเฉพาะทาง ได้แก่ **Map**, **Set**, **WeakMap** และ **WeakSet** ซึ่งถูกเพิ่มเข้ามาใน ES6 (2015)

โครงสร้างข้อมูลเหล่านี้แก้ปัญหาที่ Object และ Array ทำได้ไม่สะดวก เช่น:
- ต้องการ key ที่เป็นอะไรก็ได้ (ไม่ใช่แค่ string)
- ต้องการ collection ที่ไม่มีค่าซ้ำ
- ต้องการจัดเก็บ metadata โดยไม่กระทบต่อ garbage collection

---

## Step 691: Map คืออะไรและทำไมต้องใช้แทน Object

### Map vs Object

**Object** มีข้อจำกัดหลายอย่าง:
1. Key ต้องเป็น String หรือ Symbol เท่านั้น
2. มี prototype ที่อาจทับซ้อน key ที่เราตั้ง
3. ไม่มี property `size` โดยตรง
4. ลำดับ key ไม่แน่นอนใน Object เก่า ๆ

**Map** แก้ปัญหาเหล่านี้ทั้งหมด:

```javascript
// Object ธรรมดา - key ต้องเป็น string
const objStore = {};
const key1 = { id: 1 };
const key2 = { id: 2 };

objStore[key1] = "value1";
objStore[key2] = "value2";

console.log(objStore); 
// { '[object Object]': 'value2' } -- ปัญหา! key ถูก convert เป็น string เหมือนกัน

// Map แก้ปัญหาได้
const mapStore = new Map();
mapStore.set(key1, "value1");
mapStore.set(key2, "value2");

console.log(mapStore.get(key1)); // "value1"
console.log(mapStore.get(key2)); // "value2"
console.log(mapStore.size);      // 2
```

```javascript
// ตัวอย่าง: ใช้ Function เป็น key
const map = new Map();

function greet() { return "Hello"; }
function farewell() { return "Goodbye"; }

map.set(greet, "ทักทาย");
map.set(farewell, "ลาจาก");

console.log(map.get(greet));    // "ทักทาย"
console.log(map.get(farewell)); // "ลาจาก"
```

```javascript
// ตัวอย่าง: ใช้ Number เป็น key
const scores = new Map();
scores.set(1, "นักเรียนคนที่ 1");
scores.set(2, "นักเรียนคนที่ 2");
scores.set(42, "คำตอบของจักรวาล");

console.log(scores.get(42)); // "คำตอบของจักรวาล"
```

```javascript
// ตัวอย่าง: Object มี prototype pollution ได้
const obj = {};
console.log(obj.toString);    // function toString() {...} (มาจาก prototype)
console.log("toString" in obj); // true ทั้งที่เราไม่ได้ตั้ง

// Map ไม่มีปัญหานี้
const map = new Map();
console.log(map.get("toString")); // undefined (ปลอดภัย)
```

---

## Step 692: Map Constructor

### สร้าง Map ด้วยวิธีต่าง ๆ

```javascript
// 1. Map ว่าง
const emptyMap = new Map();
console.log(emptyMap.size); // 0

// 2. Map จาก array of entries [key, value]
const fruitsMap = new Map([
  ["apple", "แอปเปิล"],
  ["banana", "กล้วย"],
  ["cherry", "เชอร์รี่"],
]);
console.log(fruitsMap.size); // 3
console.log(fruitsMap.get("apple")); // "แอปเปิล"

// 3. Map จาก iterable ใด ๆ
function* entries() {
  yield ["x", 10];
  yield ["y", 20];
  yield ["z", 30];
}
const coordMap = new Map(entries());
console.log(coordMap.get("y")); // 20

// 4. Map จาก Map อื่น (copy)
const original = new Map([["a", 1], ["b", 2]]);
const copy = new Map(original);
copy.set("c", 3);

console.log(original.size); // 2 (ไม่เปลี่ยน)
console.log(copy.size);     // 3
```

```javascript
// ตัวอย่าง: สร้าง Map จาก Object
const obj = { name: "สมชาย", age: 30, city: "กรุงเทพ" };
const map = new Map(Object.entries(obj));

console.log(map.get("name")); // "สมชาย"
console.log(map.get("age"));  // 30
```

---

## Step 693: Map Methods - set, get, has

```javascript
const studentMap = new Map();

// set(key, value) - เพิ่มหรืออัปเดตคู่ key-value
studentMap.set("std001", { name: "อนันต์", grade: "A" });
studentMap.set("std002", { name: "วิภา", grade: "B+" });
studentMap.set("std003", { name: "ชัยชนะ", grade: "A-" });

// get(key) - ดึงค่าจาก key
console.log(studentMap.get("std001")); // { name: 'อนันต์', grade: 'A' }
console.log(studentMap.get("std999")); // undefined (key ไม่มี)

// has(key) - ตรวจสอบว่า key มีอยู่หรือไม่
console.log(studentMap.has("std002")); // true
console.log(studentMap.has("std999")); // false
```

```javascript
// set() คืนค่า Map เอง ทำให้ chain ได้
const settings = new Map();
settings
  .set("theme", "dark")
  .set("language", "th")
  .set("fontSize", 16)
  .set("notifications", true);

console.log(settings.get("theme")); // "dark"
console.log(settings.size);         // 4
```

```javascript
// การอัปเดตค่า
const counter = new Map();
counter.set("clicks", 0);
counter.set("clicks", counter.get("clicks") + 1);
counter.set("clicks", counter.get("clicks") + 1);
counter.set("clicks", counter.get("clicks") + 1);

console.log(counter.get("clicks")); // 3
```

```javascript
// ตัวอย่าง: Map ใช้ Object reference เป็น key
const user1 = { id: 1, name: "สมศรี" };
const user2 = { id: 2, name: "สมชาย" };

const permissionsMap = new Map();
permissionsMap.set(user1, ["read", "write"]);
permissionsMap.set(user2, ["read"]);

console.log(permissionsMap.get(user1)); // ["read", "write"]
console.log(permissionsMap.get(user2)); // ["read"]

// Object อื่นที่มีค่าเหมือนกัน ≠ key เดียวกัน
const fakeUser1 = { id: 1, name: "สมศรี" };
console.log(permissionsMap.get(fakeUser1)); // undefined (คนละ reference)
```

---

## Step 694: Map Methods - delete, clear, size

```javascript
const inventory = new Map([
  ["apple", 50],
  ["banana", 30],
  ["cherry", 10],
  ["durian", 5],
]);

console.log(inventory.size); // 4

// delete(key) - ลบคู่ key-value
const deleted = inventory.delete("durian");
console.log(deleted);        // true (ลบสำเร็จ)
console.log(inventory.size); // 3

// delete key ที่ไม่มี
const notFound = inventory.delete("mango");
console.log(notFound); // false

// clear() - ลบทั้งหมด
inventory.clear();
console.log(inventory.size); // 0
```

```javascript
// ตัวอย่าง: Cache ที่มีการลบข้อมูลเก่า
const cache = new Map();
const MAX_CACHE_SIZE = 3;

function getCached(key, computeFn) {
  if (cache.has(key)) {
    console.log(`Cache hit: ${key}`);
    return cache.get(key);
  }
  
  const value = computeFn();
  
  // ถ้า cache เต็ม ลบตัวแรก
  if (cache.size >= MAX_CACHE_SIZE) {
    const firstKey = cache.keys().next().value;
    cache.delete(firstKey);
    console.log(`Evicted: ${firstKey}`);
  }
  
  cache.set(key, value);
  return value;
}

getCached("a", () => "result_a"); // เพิ่ม a
getCached("b", () => "result_b"); // เพิ่ม b
getCached("c", () => "result_c"); // เพิ่ม c
getCached("d", () => "result_d"); // ลบ a แล้วเพิ่ม d
getCached("a", () => "result_a"); // ลบ b แล้วเพิ่ม a ใหม่
getCached("b", () => "result_b"); // ลบ c แล้วเพิ่ม b ใหม่
```

---

## Step 695: Iterating Map - forEach

```javascript
const capitals = new Map([
  ["Thailand", "Bangkok"],
  ["Japan", "Tokyo"],
  ["France", "Paris"],
  ["Germany", "Berlin"],
]);

// forEach(callback) - callback ได้รับ (value, key, map)
capitals.forEach((value, key, map) => {
  console.log(`${key}: ${value}`);
});
// Thailand: Bangkok
// Japan: Tokyo
// France: Paris
// Germany: Berlin
```

```javascript
// ตัวอย่าง: นับสถิติจาก Map
const wordCount = new Map();
const text = "the cat sat on the mat the cat";

text.split(" ").forEach((word) => {
  wordCount.set(word, (wordCount.get(word) || 0) + 1);
});

wordCount.forEach((count, word) => {
  console.log(`"${word}": ${count} ครั้ง`);
});
// "the": 3 ครั้ง
// "cat": 2 ครั้ง
// "sat": 1 ครั้ง
// "on": 1 ครั้ง
// "mat": 1 ครั้ง
```

```javascript
// ตัวอย่าง: transform Map เป็น Map ใหม่ด้วย forEach
function mapValues(map, transformFn) {
  const result = new Map();
  map.forEach((value, key) => {
    result.set(key, transformFn(value, key));
  });
  return result;
}

const prices = new Map([
  ["apple", 20],
  ["banana", 15],
  ["cherry", 50],
]);

const discounted = mapValues(prices, (price) => price * 0.9);
discounted.forEach((price, item) => {
  console.log(`${item}: ${price.toFixed(2)} บาท`);
});
```

---

## Step 696: Iterating Map - for-of, keys(), values(), entries()

```javascript
const colorMap = new Map([
  ["red", "#FF0000"],
  ["green", "#00FF00"],
  ["blue", "#0000FF"],
]);

// for-of กับ entries() (default iterator)
for (const [color, hex] of colorMap) {
  console.log(`${color} => ${hex}`);
}

// keys() - iterator ของ keys
for (const color of colorMap.keys()) {
  console.log(color);
}

// values() - iterator ของ values
for (const hex of colorMap.values()) {
  console.log(hex);
}

// entries() - iterator ของ [key, value] pairs
for (const entry of colorMap.entries()) {
  console.log(entry); // ['red', '#FF0000'], ...
}
```

```javascript
// ตัวอย่าง: หาค่าสูงสุดใน Map
const scores = new Map([
  ["อนันต์", 85],
  ["วิภา", 92],
  ["ชัยชนะ", 78],
  ["สุดา", 95],
  ["ประยุทธ์", 88],
]);

let maxScore = -Infinity;
let topStudent = "";

for (const [student, score] of scores) {
  if (score > maxScore) {
    maxScore = score;
    topStudent = student;
  }
}

console.log(`นักเรียนที่ได้คะแนนสูงสุด: ${topStudent} (${maxScore} คะแนน)`);
// นักเรียนที่ได้คะแนนสูงสุด: สุดา (95 คะแนน)
```

```javascript
// ตัวอย่าง: spread Map
const map = new Map([["a", 1], ["b", 2], ["c", 3]]);

const keys = [...map.keys()];
const values = [...map.values()];
const entries = [...map.entries()];

console.log(keys);    // ['a', 'b', 'c']
console.log(values);  // [1, 2, 3]
console.log(entries); // [['a', 1], ['b', 2], ['c', 3]]
```

```javascript
// ตัวอย่าง: filter Map
function filterMap(map, predicate) {
  return new Map([...map].filter(([key, value]) => predicate(key, value)));
}

const inventory = new Map([
  ["apple", 50],
  ["banana", 2],
  ["cherry", 10],
  ["durian", 0],
]);

const lowStock = filterMap(inventory, (_, qty) => qty < 10);
console.log([...lowStock]); // [['banana', 2], ['durian', 0]]
```

---

## Step 697: Converting Map to/from Objects and Arrays

```javascript
// Map to Object
const map = new Map([
  ["name", "สมชาย"],
  ["age", 30],
  ["city", "กรุงเทพ"],
]);

const obj = Object.fromEntries(map);
console.log(obj); // { name: 'สมชาย', age: 30, city: 'กรุงเทพ' }
```

```javascript
// Object to Map
const settings = {
  theme: "dark",
  language: "th",
  fontSize: 16,
};

const settingsMap = new Map(Object.entries(settings));
console.log(settingsMap.get("theme")); // "dark"
```

```javascript
// Map to Array
const scores = new Map([["A", 90], ["B", 80], ["C", 70]]);

const entriesArray = [...scores];           // [['A', 90], ['B', 80], ['C', 70]]
const keysArray = [...scores.keys()];       // ['A', 'B', 'C']
const valuesArray = [...scores.values()];   // [90, 80, 70]

// หรือใช้ Array.from
const entriesArray2 = Array.from(scores);
const keysArray2 = Array.from(scores.keys());
```

```javascript
// Array to Map
const pairs = [
  ["Thailand", 66],
  ["Japan", 81],
  ["USA", 1],
];

const dialCodes = new Map(pairs);
console.log(dialCodes.get("Thailand")); // 66
```

```javascript
// ตัวอย่าง: รวม 2 Map
function mergeMaps(...maps) {
  return new Map(maps.flatMap((m) => [...m]));
}

const map1 = new Map([["a", 1], ["b", 2]]);
const map2 = new Map([["c", 3], ["d", 4]]);
const map3 = new Map([["b", 99], ["e", 5]]); // ทับ b จาก map1

const merged = mergeMaps(map1, map2, map3);
console.log([...merged]);
// [['a', 1], ['b', 99], ['c', 3], ['d', 4], ['e', 5]]
```

```javascript
// ตัวอย่าง: JSON กับ Map
// Map ไม่รองรับ JSON.stringify โดยตรง
const map = new Map([["a", 1], ["b", 2]]);

// วิธี serialize
const json = JSON.stringify([...map]);
console.log(json); // '[["a",1],["b",2]]'

// วิธี deserialize
const restored = new Map(JSON.parse(json));
console.log(restored.get("a")); // 1
```

---

## Step 698: Use Cases for Map

### Use Case 1: Word Frequency Counter

```javascript
function wordFrequency(text) {
  const freq = new Map();
  const words = text.toLowerCase().match(/\b\w+\b/g) || [];
  
  for (const word of words) {
    freq.set(word, (freq.get(word) || 0) + 1);
  }
  
  // เรียงตาม frequency
  return new Map([...freq].sort((a, b) => b[1] - a[1]));
}

const text = "the quick brown fox jumps over the lazy dog the fox";
const freq = wordFrequency(text);

for (const [word, count] of freq) {
  console.log(`${word}: ${count}`);
}
```

### Use Case 2: Bidirectional Map

```javascript
class BidirectionalMap {
  constructor() {
    this.forwardMap = new Map();
    this.backwardMap = new Map();
  }
  
  set(key, value) {
    this.forwardMap.set(key, value);
    this.backwardMap.set(value, key);
    return this;
  }
  
  getByKey(key) {
    return this.forwardMap.get(key);
  }
  
  getByValue(value) {
    return this.backwardMap.get(value);
  }
  
  has(key) {
    return this.forwardMap.has(key);
  }
  
  hasValue(value) {
    return this.backwardMap.has(value);
  }
  
  delete(key) {
    const value = this.forwardMap.get(key);
    this.forwardMap.delete(key);
    this.backwardMap.delete(value);
  }
  
  get size() {
    return this.forwardMap.size;
  }
}

const langMap = new BidirectionalMap();
langMap.set("TH", "Thailand").set("JP", "Japan").set("FR", "France");

console.log(langMap.getByKey("TH"));       // "Thailand"
console.log(langMap.getByValue("Japan")); // "JP"
```

### Use Case 3: Memoization

```javascript
function memoize(fn) {
  const cache = new Map();
  
  return function(...args) {
    const key = JSON.stringify(args);
    
    if (cache.has(key)) {
      console.log("Cache hit!");
      return cache.get(key);
    }
    
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}

function slowFibonacci(n) {
  if (n <= 1) return n;
  return slowFibonacci(n - 1) + slowFibonacci(n - 2);
}

const fastFib = memoize(function fib(n) {
  if (n <= 1) return n;
  return fastFib(n - 1) + fastFib(n - 2);
});

console.time("fib(40)");
console.log(fastFib(40)); // 102334155
console.timeEnd("fib(40)");
```

### Use Case 4: Graph adjacency list

```javascript
class Graph {
  constructor() {
    this.adjacencyList = new Map();
  }
  
  addVertex(vertex) {
    if (!this.adjacencyList.has(vertex)) {
      this.adjacencyList.set(vertex, []);
    }
  }
  
  addEdge(v1, v2) {
    this.adjacencyList.get(v1).push(v2);
    this.adjacencyList.get(v2).push(v1);
  }
  
  getNeighbors(vertex) {
    return this.adjacencyList.get(vertex) || [];
  }
  
  bfs(start) {
    const visited = new Set();
    const queue = [start];
    const result = [];
    
    visited.add(start);
    
    while (queue.length > 0) {
      const vertex = queue.shift();
      result.push(vertex);
      
      for (const neighbor of this.getNeighbors(vertex)) {
        if (!visited.has(neighbor)) {
          visited.add(neighbor);
          queue.push(neighbor);
        }
      }
    }
    
    return result;
  }
}

const graph = new Graph();
["A", "B", "C", "D", "E"].forEach((v) => graph.addVertex(v));
graph.addEdge("A", "B");
graph.addEdge("A", "C");
graph.addEdge("B", "D");
graph.addEdge("C", "D");
graph.addEdge("D", "E");

console.log(graph.bfs("A")); // ['A', 'B', 'C', 'D', 'E']
```

---

## Step 699: Set คืออะไร - Unique Values Collection

**Set** คือ collection ที่เก็บค่าที่ไม่ซ้ำกัน ทุกค่าใน Set จะ unique เสมอ

```javascript
// สร้าง Set ว่าง
const emptySet = new Set();
console.log(emptySet.size); // 0

// สร้าง Set จาก array
const numbers = new Set([1, 2, 3, 2, 1, 4, 3, 5]);
console.log(numbers); // Set(5) { 1, 2, 3, 4, 5 }
console.log(numbers.size); // 5 (ค่าซ้ำถูกลบออก)

// สร้าง Set จาก string
const chars = new Set("javascript");
console.log(chars); // Set(9) { 'j', 'a', 'v', 's', 'c', 'r', 'i', 'p', 't' }

// Set ใช้ SameValueZero algorithm
const set = new Set();
set.add(NaN);
set.add(NaN); // ไม่เพิ่มซ้ำ
console.log(set.size); // 1

set.add(0);
set.add(-0); // 0 กับ -0 ถือว่าเหมือนกันใน Set
console.log(set.size); // 2
```

```javascript
// ตัวอย่าง: Set กับ objects
const objSet = new Set();
const obj1 = { x: 1 };
const obj2 = { x: 1 }; // ค่าเหมือนกัน แต่คนละ reference

objSet.add(obj1);
objSet.add(obj2); // เพิ่มได้ เพราะคนละ reference
objSet.add(obj1); // ไม่เพิ่มซ้ำ

console.log(objSet.size); // 2
```

---

## Step 700: Set Constructor และ Methods - add, has, delete, clear

```javascript
const fruits = new Set();

// add(value) - เพิ่มค่า (คืน Set เอง สำหรับ chaining)
fruits.add("apple");
fruits.add("banana");
fruits.add("cherry");
fruits.add("apple"); // ไม่เพิ่มซ้ำ

console.log(fruits.size); // 3

// has(value) - ตรวจสอบว่ามีค่านี้หรือไม่
console.log(fruits.has("banana")); // true
console.log(fruits.has("mango"));  // false

// delete(value) - ลบค่า
const deleted = fruits.delete("banana");
console.log(deleted);     // true
console.log(fruits.size); // 2

// ลบค่าที่ไม่มี
const notFound = fruits.delete("mango");
console.log(notFound); // false

// clear() - ลบทั้งหมด
fruits.clear();
console.log(fruits.size); // 0
```

```javascript
// Chaining add
const tags = new Set();
tags
  .add("javascript")
  .add("es6")
  .add("programming")
  .add("javascript"); // ซ้ำ ไม่เพิ่ม

console.log(tags.size); // 3
console.log([...tags]); // ['javascript', 'es6', 'programming']
```

```javascript
// ตัวอย่าง: ระบบ permission
class PermissionManager {
  constructor() {
    this.permissions = new Set();
  }
  
  grant(permission) {
    this.permissions.add(permission);
    console.log(`Granted: ${permission}`);
    return this;
  }
  
  revoke(permission) {
    if (this.permissions.delete(permission)) {
      console.log(`Revoked: ${permission}`);
    }
    return this;
  }
  
  can(permission) {
    return this.permissions.has(permission);
  }
  
  list() {
    return [...this.permissions];
  }
}

const admin = new PermissionManager();
admin.grant("read").grant("write").grant("delete");

console.log(admin.can("write"));  // true
console.log(admin.can("admin"));  // false

admin.revoke("delete");
console.log(admin.list()); // ['read', 'write']
```

---

## Step 701: Iterating Set

```javascript
const colors = new Set(["red", "green", "blue", "yellow"]);

// for-of
for (const color of colors) {
  console.log(color);
}

// forEach
colors.forEach((value) => {
  console.log(value.toUpperCase());
});

// values() - iterator (เหมือน default iterator)
for (const color of colors.values()) {
  console.log(color);
}

// keys() - ใน Set, keys() === values() (มีเพื่อความสอดคล้องกับ Map)
for (const color of colors.keys()) {
  console.log(color);
}

// entries() - คืน [value, value] pairs
for (const entry of colors.entries()) {
  console.log(entry); // ['red', 'red'], ['green', 'green'], ...
}
```

```javascript
// spread Set
const set = new Set([1, 2, 3, 4, 5]);
const arr = [...set];
console.log(arr); // [1, 2, 3, 4, 5]

// Array.from
const arr2 = Array.from(set);
console.log(arr2); // [1, 2, 3, 4, 5]

// destructuring
const [first, second, ...rest] = set;
console.log(first);  // 1
console.log(second); // 2
console.log(rest);   // [3, 4, 5]
```

---

## Step 702: Set Operations - Union, Intersection, Difference

```javascript
const A = new Set([1, 2, 3, 4, 5]);
const B = new Set([4, 5, 6, 7, 8]);

// Union (สหภาพ): ค่าทั้งหมดจากทั้งสอง Set
const union = new Set([...A, ...B]);
console.log([...union]); // [1, 2, 3, 4, 5, 6, 7, 8]

// Intersection (ตัดกัน): ค่าที่มีในทั้งสอง Set
const intersection = new Set([...A].filter((x) => B.has(x)));
console.log([...intersection]); // [4, 5]

// Difference (ผลต่าง): ค่าที่มีใน A แต่ไม่มีใน B
const difference = new Set([...A].filter((x) => !B.has(x)));
console.log([...difference]); // [1, 2, 3]

// Symmetric Difference: ค่าที่มีใน A หรือ B แต่ไม่ทั้งสอง
const symDiff = new Set([
  ...[...A].filter((x) => !B.has(x)),
  ...[...B].filter((x) => !A.has(x)),
]);
console.log([...symDiff]); // [1, 2, 3, 6, 7, 8]
```

```javascript
// Helper functions สำหรับ Set operations
function union(setA, setB) {
  return new Set([...setA, ...setB]);
}

function intersection(setA, setB) {
  return new Set([...setA].filter((x) => setB.has(x)));
}

function difference(setA, setB) {
  return new Set([...setA].filter((x) => !setB.has(x)));
}

function isSubset(setA, setB) {
  return [...setA].every((x) => setB.has(x));
}

function isSuperset(setA, setB) {
  return isSubset(setB, setA);
}

// ตัวอย่าง: วิชาที่นักเรียนลงทะเบียน
const alice = new Set(["Math", "Science", "English", "History"]);
const bob = new Set(["Science", "English", "Art", "Music"]);

console.log([...union(alice, bob)]);
// ['Math', 'Science', 'English', 'History', 'Art', 'Music']

console.log([...intersection(alice, bob)]);
// ['Science', 'English']

console.log([...difference(alice, bob)]);
// ['Math', 'History']

const coreSubjects = new Set(["Science", "English"]);
console.log(isSubset(coreSubjects, alice)); // true (alice ลง core ทั้งหมด)
```

```javascript
// ตัวอย่างใช้งานจริง: ค้นหา mutual friends
const friendsOfAlice = new Set(["Bob", "Charlie", "Dave", "Eve"]);
const friendsOfBob = new Set(["Alice", "Charlie", "Frank", "Eve"]);

const mutualFriends = intersection(friendsOfAlice, friendsOfBob);
console.log(`เพื่อนร่วมกัน: ${[...mutualFriends].join(", ")}`);
// เพื่อนร่วมกัน: Charlie, Eve
```

---

## Step 703: Converting Set to/from Arrays และ Removing Duplicates

```javascript
// วิธียอดนิยมในการลบค่าซ้ำจาก Array
const numbers = [1, 2, 3, 2, 1, 4, 3, 5, 4];

// วิธีที่ 1: Set + spread
const unique1 = [...new Set(numbers)];
console.log(unique1); // [1, 2, 3, 4, 5]

// วิธีที่ 2: Array.from
const unique2 = Array.from(new Set(numbers));
console.log(unique2); // [1, 2, 3, 4, 5]
```

```javascript
// ลบ string ซ้ำ
const words = ["hello", "world", "hello", "foo", "world", "bar"];
const uniqueWords = [...new Set(words)];
console.log(uniqueWords); // ['hello', 'world', 'foo', 'bar']

// ลบ string ซ้ำ (case-insensitive)
const tags = ["JavaScript", "javascript", "Python", "PYTHON", "Go"];
const uniqueTags = [...new Set(tags.map((t) => t.toLowerCase()))];
console.log(uniqueTags); // ['javascript', 'python', 'go']
```

```javascript
// ตัวอย่าง: ลบ objects ซ้ำจาก property
function uniqueByProperty(array, property) {
  const seen = new Set();
  return array.filter((item) => {
    const value = item[property];
    if (seen.has(value)) return false;
    seen.add(value);
    return true;
  });
}

const products = [
  { id: 1, name: "Apple" },
  { id: 2, name: "Banana" },
  { id: 1, name: "Apple (duplicate)" },
  { id: 3, name: "Cherry" },
  { id: 2, name: "Banana (duplicate)" },
];

const unique = uniqueByProperty(products, "id");
console.log(unique);
// [{ id: 1, name: 'Apple' }, { id: 2, name: 'Banana' }, { id: 3, name: 'Cherry' }]
```

```javascript
// ตัวอย่าง: ตรวจสอบ anagram โดยใช้ Set
function areAnagrams(str1, str2) {
  if (str1.length !== str2.length) return false;
  
  const charSet = new Set(str1.toLowerCase());
  return [...charSet].every((char) =>
    str2.toLowerCase().split("").filter((c) => c === char).length ===
    str1.toLowerCase().split("").filter((c) => c === char).length
  );
}

console.log(areAnagrams("listen", "silent")); // true
console.log(areAnagrams("hello", "world"));   // false
```

---

## Step 704: WeakMap - Keys Must Be Objects

**WeakMap** คล้าย Map แต่มีความแตกต่างสำคัญ:
1. Key ต้องเป็น Object หรือ non-registered Symbol เท่านั้น
2. ไม่สามารถ enumerate (วน loop) ได้
3. เมื่อ key object ถูก garbage collect, entry นั้นหายไปอัตโนมัติ

```javascript
const weakMap = new WeakMap();

// Key ต้องเป็น object
let obj1 = { name: "สมชาย" };
let obj2 = { name: "สมศรี" };

weakMap.set(obj1, "data1");
weakMap.set(obj2, "data2");

console.log(weakMap.get(obj1)); // "data1"
console.log(weakMap.has(obj2)); // true

// Key ที่เป็น primitive ใช้ไม่ได้
try {
  weakMap.set("string", "value"); // TypeError!
} catch (e) {
  console.log("Error:", e.message);
}

// ลบ entry
weakMap.delete(obj1);
console.log(weakMap.has(obj1)); // false
```

```javascript
// Garbage Collection demonstration (แนวคิด)
let element = { id: "btn-submit", type: "button" };
const weakMap = new WeakMap();
weakMap.set(element, { clickCount: 0, lastClicked: null });

// เมื่อ element ถูก garbage collect
element = null;
// entry ใน weakMap จะหายไปอัตโนมัติ
// (ไม่มี memory leak)
```

---

## Step 705: WeakMap Use Cases - Private Data

### Pattern 1: Private Class Data

```javascript
const _private = new WeakMap();

class BankAccount {
  constructor(owner, balance) {
    _private.set(this, {
      owner,
      balance,
      transactions: [],
    });
  }
  
  get owner() {
    return _private.get(this).owner;
  }
  
  get balance() {
    return _private.get(this).balance;
  }
  
  deposit(amount) {
    if (amount <= 0) throw new Error("จำนวนเงินต้องมากกว่า 0");
    
    const data = _private.get(this);
    data.balance += amount;
    data.transactions.push({ type: "deposit", amount, date: new Date() });
    
    return this;
  }
  
  withdraw(amount) {
    const data = _private.get(this);
    if (amount > data.balance) throw new Error("เงินในบัญชีไม่พอ");
    
    data.balance -= amount;
    data.transactions.push({ type: "withdraw", amount, date: new Date() });
    
    return this;
  }
  
  getTransactionHistory() {
    return [..._private.get(this).transactions];
  }
}

const account = new BankAccount("สมชาย", 1000);
account.deposit(500).deposit(200).withdraw(100);

console.log(account.balance); // 1600
console.log(account.getTransactionHistory().length); // 3

// ไม่สามารถเข้าถึง _private ข้างนอกได้
// (ถ้า _private เป็น module-level variable)
```

### Pattern 2: Memoization กับ WeakMap

```javascript
const memoMap = new WeakMap();

function processExpensiveData(obj) {
  if (memoMap.has(obj)) {
    console.log("Cached result used");
    return memoMap.get(obj);
  }
  
  // คำนวณที่ใช้เวลานาน
  const result = {
    sum: Object.values(obj.data || {}).reduce((a, b) => a + b, 0),
    count: Object.keys(obj.data || {}).length,
    processed: true,
  };
  
  memoMap.set(obj, result);
  return result;
}

const data1 = { data: { a: 1, b: 2, c: 3 } };
console.log(processExpensiveData(data1)); // คำนวณใหม่
console.log(processExpensiveData(data1)); // ใช้ cache
```

### Pattern 3: DOM Element Metadata

```javascript
const elementData = new WeakMap();

function trackElement(element, metadata) {
  elementData.set(element, {
    ...metadata,
    createdAt: Date.now(),
  });
}

function getElementData(element) {
  return elementData.get(element);
}

// ใน browser environment
// const btn = document.getElementById('myButton');
// trackElement(btn, { clicks: 0, label: 'Submit' });
// เมื่อ btn ถูกลบออกจาก DOM และ garbage collected
// ข้อมูลใน elementData ก็จะหายไปอัตโนมัติ
```

---

## Step 706: WeakSet - Set of Objects

```javascript
const weakSet = new WeakSet();

let obj1 = { name: "item1" };
let obj2 = { name: "item2" };

// เพิ่ม object
weakSet.add(obj1);
weakSet.add(obj2);

// ตรวจสอบ
console.log(weakSet.has(obj1)); // true
console.log(weakSet.has(obj2)); // true

// ลบ
weakSet.delete(obj1);
console.log(weakSet.has(obj1)); // false

// ใช้ primitive ไม่ได้
try {
  weakSet.add(42); // TypeError
} catch (e) {
  console.log("Error:", e.message);
}
```

```javascript
// Use Case: ตรวจสอบ circular references
const seen = new WeakSet();

function deepClone(obj) {
  if (typeof obj !== "object" || obj === null) return obj;
  
  if (seen.has(obj)) {
    throw new Error("Circular reference detected");
  }
  
  seen.add(obj);
  
  const clone = Array.isArray(obj) ? [] : {};
  
  for (const [key, value] of Object.entries(obj)) {
    clone[key] = deepClone(value);
  }
  
  seen.delete(obj);
  return clone;
}

const obj = { a: 1, b: { c: 2, d: [3, 4] } };
const cloned = deepClone(obj);
console.log(cloned); // { a: 1, b: { c: 2, d: [3, 4] } }

// ทดสอบ circular
const circular = { a: 1 };
circular.self = circular;
try {
  deepClone(circular);
} catch (e) {
  console.log(e.message); // "Circular reference detected"
}
```

```javascript
// Use Case: ระบุ objects ที่ถูก process แล้ว
const processed = new WeakSet();

class DataProcessor {
  process(data) {
    if (processed.has(data)) {
      console.log("Already processed, skipping...");
      return;
    }
    
    // ทำงานกับข้อมูล
    console.log("Processing:", data);
    processed.add(data);
  }
}

const processor = new DataProcessor();
const record = { id: 1, value: "test" };

processor.process(record); // Processing
processor.process(record); // Already processed, skipping...
```

---

## Step 707: WeakRef - อ้างอิงแบบ Weak

```javascript
// WeakRef ช่วยอ้างอิง object โดยไม่ prevent garbage collection
let bigData = {
  // สมมติว่านี่เป็นข้อมูลขนาดใหญ่มาก
  data: new Array(1000000).fill(0),
};

const weakRef = new WeakRef(bigData);

// เข้าถึงด้วย .deref()
const data = weakRef.deref();
if (data) {
  console.log("Object still alive, data length:", data.data.length);
} else {
  console.log("Object has been garbage collected");
}

// ถ้า bigData = null; และ GC ทำงาน
// weakRef.deref() จะคืน undefined
```

```javascript
// ตัวอย่าง: Cache ที่ไม่ prevent GC
class WeakCache {
  constructor() {
    this.cache = new Map(); // key -> WeakRef<value>
  }
  
  set(key, value) {
    this.cache.set(key, new WeakRef(value));
  }
  
  get(key) {
    const ref = this.cache.get(key);
    if (!ref) return undefined;
    
    const value = ref.deref();
    if (!value) {
      // Object ถูก GC แล้ว ลบออกจาก Map
      this.cache.delete(key);
      return undefined;
    }
    
    return value;
  }
  
  has(key) {
    return this.get(key) !== undefined;
  }
}
```

---

## Step 708: FinalizationRegistry

```javascript
// FinalizationRegistry ให้ callback เมื่อ object ถูก GC
const registry = new FinalizationRegistry((heldValue) => {
  console.log(`Object with id ${heldValue} has been garbage collected`);
});

let myObj = { data: "some data" };
registry.register(myObj, "obj-001"); // register พร้อม held value

// เมื่อ myObj ถูก GC callback จะถูกเรียก
// (ไม่สามารถ predict ว่าเมื่อไหร่)
myObj = null;
```

```javascript
// ตัวอย่าง: Resource cleanup
class ManagedResource {
  #resourceId;
  #registry;
  
  static #cleanupRegistry = new FinalizationRegistry((id) => {
    console.log(`Cleaning up resource: ${id}`);
    // ทำ cleanup จริง ๆ เช่น ปิด connection
  });
  
  constructor(id) {
    this.#resourceId = id;
    ManagedResource.#cleanupRegistry.register(this, id);
    console.log(`Resource ${id} created`);
  }
  
  use() {
    console.log(`Using resource ${this.#resourceId}`);
  }
}

let resource = new ManagedResource("conn-001");
resource.use();
resource = null; // เมื่อ GC ทำงาน callback จะถูกเรียก
```

---

## Step 709: Practical Comparison - Map vs Object vs Array vs Set

```javascript
// เมื่อไหร่ควรใช้อะไร

// Object: เหมาะกับ record ที่มี property ชัดเจน
const user = {
  name: "สมชาย",
  age: 30,
  email: "somchai@example.com",
};

// Map: เหมาะกับ dynamic key-value pairs, key ที่ไม่ใช่ string
const dynamicStore = new Map();
const domElement = document.createElement("div"); // สมมติ
dynamicStore.set(domElement, { clickCount: 0 });

// Array: เหมาะกับ ordered list
const orderedItems = ["first", "second", "third"];

// Set: เหมาะกับ unique values, membership testing
const uniqueVisitors = new Set();
uniqueVisitors.add("user-001");
uniqueVisitors.add("user-002");
uniqueVisitors.add("user-001"); // ไม่เพิ่มซ้ำ
```

```javascript
// Performance Comparison: Map vs Object for key lookup

// สร้างข้อมูลขนาดใหญ่
const SIZE = 100000;

const obj = {};
const map = new Map();

for (let i = 0; i < SIZE; i++) {
  obj[`key${i}`] = i;
  map.set(`key${i}`, i);
}

// Test lookup performance
console.time("Object lookup");
for (let i = 0; i < SIZE; i++) {
  const _ = obj[`key${i}`];
}
console.timeEnd("Object lookup");

console.time("Map lookup");
for (let i = 0; i < SIZE; i++) {
  const _ = map.get(`key${i}`);
}
console.timeEnd("Map lookup");
```

```javascript
// Performance: Set vs Array for membership testing
const SIZE = 10000;
const items = Array.from({ length: SIZE }, (_, i) => `item${i}`);

const array = items.slice();
const set = new Set(items);

const searchItems = Array.from({ length: 1000 }, (_, i) => `item${Math.floor(Math.random() * SIZE * 2)}`);

console.time("Array includes");
searchItems.forEach((item) => array.includes(item));
console.timeEnd("Array includes"); // O(n) ต่อการค้นหา

console.time("Set has");
searchItems.forEach((item) => set.has(item));
console.timeEnd("Set has"); // O(1) ต่อการค้นหา
```

---

## Step 710: ตัวอย่างขั้นสูง - Combining Data Structures

```javascript
// ระบบ Inventory Management ที่ใช้ Map, Set ร่วมกัน

class Inventory {
  #products = new Map();    // productId -> product data
  #categories = new Map();  // category -> Set of productIds
  #tags = new Map();        // tag -> Set of productIds
  
  addProduct(product) {
    const { id, name, price, category, tags = [] } = product;
    
    // เก็บข้อมูลสินค้า
    this.#products.set(id, { id, name, price, category, tags });
    
    // จัดหมวดหมู่
    if (!this.#categories.has(category)) {
      this.#categories.set(category, new Set());
    }
    this.#categories.get(category).add(id);
    
    // จัด tags
    for (const tag of tags) {
      if (!this.#tags.has(tag)) {
        this.#tags.set(tag, new Set());
      }
      this.#tags.get(tag).add(id);
    }
    
    return this;
  }
  
  getProduct(id) {
    return this.#products.get(id);
  }
  
  getByCategory(category) {
    const ids = this.#categories.get(category) || new Set();
    return [...ids].map((id) => this.#products.get(id));
  }
  
  getByTag(tag) {
    const ids = this.#tags.get(tag) || new Set();
    return [...ids].map((id) => this.#products.get(id));
  }
  
  getByTags(tags, mode = "any") {
    const sets = tags.map((tag) => this.#tags.get(tag) || new Set());
    
    if (mode === "any") {
      // Union: มี tag ใด tag หนึ่ง
      const ids = new Set(sets.flatMap((s) => [...s]));
      return [...ids].map((id) => this.#products.get(id));
    } else {
      // Intersection: มีทุก tag
      if (sets.length === 0) return [];
      let ids = sets[0];
      for (let i = 1; i < sets.length; i++) {
        ids = new Set([...ids].filter((x) => sets[i].has(x)));
      }
      return [...ids].map((id) => this.#products.get(id));
    }
  }
  
  get stats() {
    return {
      totalProducts: this.#products.size,
      categories: this.#categories.size,
      tags: this.#tags.size,
    };
  }
}

// ทดสอบ
const store = new Inventory();

store
  .addProduct({ id: "p001", name: "MacBook Pro", price: 59900, category: "laptop", tags: ["apple", "premium", "portable"] })
  .addProduct({ id: "p002", name: "iPhone 15", price: 29900, category: "phone", tags: ["apple", "premium", "5g"] })
  .addProduct({ id: "p003", name: "Dell XPS", price: 45000, category: "laptop", tags: ["windows", "premium", "portable"] })
  .addProduct({ id: "p004", name: "Samsung Galaxy", price: 25000, category: "phone", tags: ["android", "5g"] });

console.log("Laptops:", store.getByCategory("laptop").map((p) => p.name));
// ['MacBook Pro', 'Dell XPS']

console.log("Apple products:", store.getByTag("apple").map((p) => p.name));
// ['MacBook Pro', 'iPhone 15']

console.log("Premium AND portable:", store.getByTags(["premium", "portable"], "all").map((p) => p.name));
// ['MacBook Pro', 'Dell XPS']

console.log("Stats:", store.stats);
// { totalProducts: 4, categories: 2, tags: 7 }
```

---

## แบบฝึกหัด (Exercises)

### Easy
1. สร้าง Map ที่เก็บชื่อนักเรียนและคะแนน แล้วหาค่าเฉลี่ย
2. ใช้ Set เพื่อลบค่าซ้ำจาก array ต่อไปนี้: `[3, 1, 4, 1, 5, 9, 2, 6, 5, 3, 5]`
3. สร้างฟังก์ชัน `groupBy(array, key)` ที่ใช้ Map

### Medium
4. Implement `MultiMap` - Map ที่ key หนึ่งมีได้หลาย values
5. ใช้ WeakMap เพื่อเพิ่ม private data ให้กับ DOM elements
6. สร้าง LRU Cache โดยใช้ Map (LRU = Least Recently Used)

### Hard
7. Implement `ObservableMap` ที่แจ้งเตือนเมื่อมีการ set/delete
8. สร้างระบบ graph ที่ใช้ Map และ Set สำหรับ DFS และ BFS
9. Implement `PersistentSet` ที่ serialize/deserialize ได้

### Solution ตัวอย่าง

```javascript
// 4. MultiMap
class MultiMap {
  #map = new Map();
  
  add(key, value) {
    if (!this.#map.has(key)) {
      this.#map.set(key, new Set());
    }
    this.#map.get(key).add(value);
    return this;
  }
  
  get(key) {
    return this.#map.get(key) || new Set();
  }
  
  has(key, value) {
    if (value === undefined) return this.#map.has(key);
    return this.#map.has(key) && this.#map.get(key).has(value);
  }
  
  delete(key, value) {
    if (value === undefined) {
      return this.#map.delete(key);
    }
    const values = this.#map.get(key);
    if (!values) return false;
    const deleted = values.delete(value);
    if (values.size === 0) this.#map.delete(key);
    return deleted;
  }
  
  get size() {
    return this.#map.size;
  }
  
  *[Symbol.iterator]() {
    for (const [key, values] of this.#map) {
      for (const value of values) {
        yield [key, value];
      }
    }
  }
}

const mm = new MultiMap();
mm.add("fruit", "apple")
  .add("fruit", "banana")
  .add("fruit", "cherry")
  .add("veggie", "carrot");

console.log([...mm.get("fruit")]); // ['apple', 'banana', 'cherry']
console.log(mm.has("fruit", "apple")); // true
console.log(mm.has("fruit", "mango")); // false

// 6. LRU Cache
class LRUCache {
  #capacity;
  #cache; // Map รักษา insertion order
  
  constructor(capacity) {
    this.#capacity = capacity;
    this.#cache = new Map();
  }
  
  get(key) {
    if (!this.#cache.has(key)) return -1;
    
    // ย้ายไปท้ายสุด (most recently used)
    const value = this.#cache.get(key);
    this.#cache.delete(key);
    this.#cache.set(key, value);
    
    return value;
  }
  
  put(key, value) {
    if (this.#cache.has(key)) {
      this.#cache.delete(key);
    } else if (this.#cache.size >= this.#capacity) {
      // ลบตัวแรก (least recently used)
      const firstKey = this.#cache.keys().next().value;
      this.#cache.delete(firstKey);
    }
    
    this.#cache.set(key, value);
  }
  
  get size() {
    return this.#cache.size;
  }
}

const lru = new LRUCache(3);
lru.put("a", 1);
lru.put("b", 2);
lru.put("c", 3);
lru.get("a");     // ทำให้ a เป็น most recent
lru.put("d", 4);  // ลบ b (LRU)

console.log(lru.get("b")); // -1 (ถูกลบ)
console.log(lru.get("a")); // 1
console.log(lru.get("c")); // 3
console.log(lru.get("d")); // 4
```

---

## สรุป

| Feature | Map | Set | WeakMap | WeakSet |
|---------|-----|-----|---------|---------|
| Key type | Any | N/A | Object only | N/A |
| Value type | Any | Any (unique) | Any | Object only |
| Enumerable | Yes | Yes | No | No |
| Size property | Yes | Yes | No | No |
| GC-friendly | No | No | Yes | Yes |
| Use case | Key-value store | Unique values | Private data | Object tracking |

**เมื่อไหร่ควรใช้อะไร:**
- **Map**: เมื่อต้องการ key-value store ที่ key ไม่ใช่ string, หรือต้องการรักษา insertion order
- **Set**: เมื่อต้องการ collection ของค่า unique หรือต้องการ set operations
- **WeakMap**: เมื่อต้องการเก็บ private data ของ object หรือ cache โดยไม่ prevent GC
- **WeakSet**: เมื่อต้องการ track object membership โดยไม่ prevent GC
