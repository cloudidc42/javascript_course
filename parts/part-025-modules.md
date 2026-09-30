# Part 25: JavaScript Modules (Steps 471-490)

## บทนำ

JavaScript Modules คือระบบที่ช่วยให้เราแบ่งโค้ดออกเป็นไฟล์ย่อยๆ ที่สามารถ import/export กันได้ ทำให้โค้ดมีโครงสร้างที่ดี บำรุงรักษาง่าย และสามารถ reuse ได้

---

## Step 471: ทำไมต้องมี Modules - ประวัติและปัญหา

### ปัญหาก่อนมี Modules

```javascript
// ปัญหา 1: Global Namespace Pollution
// ไฟล์ A
var utils = { ... };
var helpers = { ... };

// ไฟล์ B - อาจ override ของ A โดยไม่รู้ตัว!
var utils = { different: true };

// ปัญหา 2: ลำดับ script tags สำคัญมาก
// HTML:
// <script src="dep.js"></script>   // ต้องมาก่อน
// <script src="app.js"></script>   // ขึ้นอยู่กับ dep.js

// ปัญหา 3: ไม่มี encapsulation - ทุกอย่างเป็น global
```

### แนวทางแก้ปัญหาแบบเก่า

```javascript
// วิธี 1: IIFE (Immediately Invoked Function Expression)
const MyModule = (function() {
  // private
  let _privateVar = 'ส่วนตัว';
  
  function _privateMethod() {
    return _privateVar;
  }
  
  // public API
  return {
    getVar: _privateMethod,
    setVar: (val) => { _privateVar = val; }
  };
})();

console.log(MyModule.getVar());    // 'ส่วนตัว'
// console.log(MyModule._privateVar); // undefined!

// วิธี 2: Namespace Object
const MyApp = MyApp || {};
MyApp.utils = {
  format: function(val) { return String(val); }
};
MyApp.models = {
  User: function(name) { this.name = name; }
};
```

### CommonJS (Node.js)

```javascript
// math.js - เขียน module
function add(a, b) { return a + b; }
function multiply(a, b) { return a * b; }

module.exports = { add, multiply };
// หรือ
module.exports.add = add;

// app.js - ใช้ module
const math = require('./math');
console.log(math.add(1, 2)); // 3

const { add, multiply } = require('./math');
console.log(multiply(3, 4)); // 12

// require เป็น synchronous - block จนกว่าจะโหลดเสร็จ
// ใช้ในทุก Node.js version (เก่าและใหม่)
```

### AMD (Asynchronous Module Definition)

```javascript
// AMD: ใช้ใน browser ก่อน ES6 (RequireJS)
define(['jquery', 'lodash'], function($, _) {
  // module code
  return {
    init: function() {
      // ...
    }
  };
});

require(['app'], function(app) {
  app.init();
});
// AMD โหลดแบบ asynchronous - ดีสำหรับ browser
```

---

## Step 472: ES Modules (ESM) - มาตรฐานใหม่

```javascript
// ES Modules คือ standard ของ JavaScript (ES6/2015)
// ใช้ import/export keywords

// math.js
export function add(a, b) { return a + b; }
export function multiply(a, b) { return a * b; }

// app.js
import { add, multiply } from './math.js';
console.log(add(1, 2));       // 3
console.log(multiply(3, 4));  // 12
```

```javascript
// ความแตกต่างหลักของ ESM vs CommonJS

// ESM:
// - Static imports (วิเคราะห์ตอน parse ไม่ใช่ runtime)
// - Asynchronous loading
// - Strict mode by default
// - Top-level this = undefined
// - import/export อยู่ระดับ top-level เท่านั้น (ไม่อยู่ใน if, function)

// CommonJS:
// - Dynamic requires (ทำงานตอน runtime)
// - Synchronous loading
// - Non-strict mode by default
// - Top-level this = module.exports
// - require() ได้ทุกที่ใน code
```

---

## Step 473: Named Exports

```javascript
// === string-utils.js ===

// Named export: ประกาศและ export พร้อมกัน
export function capitalize(str) {
  return str.charAt(0).toUpperCase() + str.slice(1).toLowerCase();
}

export function truncate(str, maxLength = 50) {
  if (str.length <= maxLength) return str;
  return str.slice(0, maxLength - 3) + '...';
}

export function countWords(str) {
  return str.trim().split(/\s+/).filter(Boolean).length;
}

// Named export: ประกาศก่อน แล้ว export ทีหลัง
const reverseString = (str) => str.split('').reverse().join('');
const isPalindrome = (str) => {
  const cleaned = str.toLowerCase().replace(/[^a-z0-9]/g, '');
  return cleaned === reverseString(cleaned);
};

export { reverseString, isPalindrome };

// Export constants
export const VERSION = '1.0.0';
export const MAX_LENGTH = 1000;
```

```javascript
// === app.js ===
import { capitalize, truncate, countWords, VERSION } from './string-utils.js';

console.log(capitalize('hello world')); // 'Hello world'
console.log(truncate('นี่คือข้อความที่ยาวมากๆ', 10)); // 'นี่คือข้อ...'
console.log(countWords('สวัสดี โลก JavaScript')); // 3
console.log(VERSION); // '1.0.0'
```

---

## Step 474: Default Exports

```javascript
// === User.js ===
// Default export: แต่ละ module มีได้แค่ 1 default export

class User {
  #id;
  #name;
  #email;
  
  constructor({ id, name, email }) {
    this.#id = id;
    this.#name = name;
    this.#email = email;
  }
  
  get id() { return this.#id; }
  get name() { return this.#name; }
  get email() { return this.#email; }
  
  toJSON() {
    return { id: this.#id, name: this.#name, email: this.#email };
  }
  
  toString() {
    return `User(${this.#name})`;
  }
  
  static create(data) {
    return new User(data);
  }
}

export default User;
```

```javascript
// === app.js ===
// Default import: ตั้งชื่ออะไรก็ได้!
import User from './User.js';
import MyUser from './User.js'; // ชื่อเดียวกับ module ก็ได้

const user = new User({ id: 1, name: 'สมชาย', email: 'somchai@example.com' });
console.log(user.name);    // 'สมชาย'
console.log(user.toJSON()); // { id: 1, name: 'สมชาย', email: '...' }
console.log(String(user));  // 'User(สมชาย)'
```

```javascript
// Default export รูปแบบต่างๆ
// 1. export default function
export default function greet(name) {
  return `สวัสดี, ${name}!`;
}

// 2. export default class
export default class Animal {
  constructor(name) { this.name = name; }
  speak() { return `${this.name} พูด`; }
}

// 3. export default expression
export default {
  host: 'localhost',
  port: 3000,
  debug: true,
};

// 4. ประกาศก่อน แล้ว export default
const config = { host: 'localhost', port: 3000 };
export default config;
```

---

## Step 475: Named + Default Combined Exports

```javascript
// === api-client.js ===

// Default export: main class
export default class ApiClient {
  #baseUrl;
  #headers;
  
  constructor(baseUrl, options = {}) {
    this.#baseUrl = baseUrl;
    this.#headers = {
      'Content-Type': 'application/json',
      ...options.headers,
    };
  }
  
  async get(path) {
    const response = await fetch(`${this.#baseUrl}${path}`, {
      headers: this.#headers,
    });
    return response.json();
  }
  
  async post(path, data) {
    const response = await fetch(`${this.#baseUrl}${path}`, {
      method: 'POST',
      headers: this.#headers,
      body: JSON.stringify(data),
    });
    return response.json();
  }
}

// Named exports: helpers และ constants
export const DEFAULT_TIMEOUT = 30000;
export const BASE_URL = 'https://api.example.com';

export function createAuthHeaders(token) {
  return { Authorization: `Bearer ${token}` };
}

export class ApiError extends Error {
  constructor(message, status) {
    super(message);
    this.name = 'ApiError';
    this.status = status;
  }
}
```

```javascript
// === app.js ===
import ApiClient, { DEFAULT_TIMEOUT, BASE_URL, createAuthHeaders, ApiError } from './api-client.js';

const client = new ApiClient(BASE_URL, {
  headers: createAuthHeaders('my-token'),
});

try {
  const data = await client.get('/users');
  console.log(data);
} catch (err) {
  if (err instanceof ApiError) {
    console.error(`API Error ${err.status}: ${err.message}`);
  }
}
```

---

## Step 476: Import Named Exports

```javascript
// รูปแบบต่างๆ ของ named imports

// === math.js ===
export const PI = 3.14159;
export const E = 2.71828;
export function add(a, b) { return a + b; }
export function subtract(a, b) { return a - b; }
export function multiply(a, b) { return a * b; }
export function divide(a, b) {
  if (b === 0) throw new Error('Division by zero');
  return a / b;
}

// === app.js ===

// 1. Import specific
import { add, multiply } from './math.js';

// 2. Import with rename (aliasing)
import { add as sum, multiply as product } from './math.js';
console.log(sum(1, 2));      // 3
console.log(product(3, 4));  // 12

// 3. Import constants
import { PI, E } from './math.js';
console.log(PI); // 3.14159

// 4. Import ทุกอย่างเป็น namespace
import * as Math from './math.js';
console.log(Math.PI);            // 3.14159
console.log(Math.add(1, 2));     // 3
console.log(Math.subtract(5, 3)); // 2
```

---

## Step 477: Import Default Export

```javascript
// Default import ไม่ต้องใช้ {}
// ตั้งชื่ออะไรก็ได้

// === calculator.js ===
export default class Calculator {
  constructor() { this.history = []; }
  
  add(a, b) {
    const result = a + b;
    this.history.push(`${a} + ${b} = ${result}`);
    return result;
  }
}

// === app.js ===
import Calculator from './calculator.js';
// หรือชื่ออื่น:
import Calc from './calculator.js';
import MyCalc from './calculator.js';

const calc = new Calculator();
calc.add(5, 3);
console.log(calc.history); // ['5 + 3 = 8']
```

```javascript
// Import default และ named พร้อมกัน
// === utils.js ===
export default function mainUtil() { return 'main'; }
export function helper1() { return 'h1'; }
export function helper2() { return 'h2'; }

// === app.js ===
import mainUtil, { helper1, helper2 } from './utils.js';
// หรือ rename default:
import myUtil, { helper1 as h1, helper2 as h2 } from './utils.js';
```

---

## Step 478: Import All as Namespace

```javascript
// === colors.js ===
export const RED = '#FF0000';
export const GREEN = '#00FF00';
export const BLUE = '#0000FF';
export const WHITE = '#FFFFFF';
export const BLACK = '#000000';

export function hexToRgb(hex) {
  const result = /^#?([a-f\d]{2})([a-f\d]{2})([a-f\d]{2})$/i.exec(hex);
  return result ? {
    r: parseInt(result[1], 16),
    g: parseInt(result[2], 16),
    b: parseInt(result[3], 16)
  } : null;
}

export function rgbToHex(r, g, b) {
  return '#' + [r, g, b].map(v => v.toString(16).padStart(2, '0')).join('');
}

// === app.js ===
import * as Colors from './colors.js';

console.log(Colors.RED);          // '#FF0000'
console.log(Colors.hexToRgb(Colors.RED)); // { r: 255, g: 0, b: 0 }

// Namespace import เหมาะเมื่อต้องการทุก export
// หรือเมื่อไม่รู้ว่า module export อะไรบ้าง
// (แต่ทำให้ tree shaking ยากกว่า)
```

---

## Step 479: Renaming Imports และ Exports

```javascript
// === string-utils.js ===

// Rename ตอน export
function internalCapitalize(str) {
  return str.charAt(0).toUpperCase() + str.slice(1);
}

export {
  internalCapitalize as capitalize,  // export ชื่อใหม่
  internalCapitalize as titleCase,    // export ชื่ออื่นของ function เดียวกัน
};

// === app.js ===

// Rename ตอน import
import { capitalize as cap, titleCase as tc } from './string-utils.js';
console.log(cap('hello'));   // 'Hello'
console.log(tc('world'));    // 'World'
```

```javascript
// Rename เพื่อหลีกเลี่ยง conflict
// === module-a.js ===
export function process(data) { return `A: ${data}`; }

// === module-b.js ===
export function process(data) { return `B: ${data}`; }

// === app.js ===
import { process as processA } from './module-a.js';
import { process as processB } from './module-b.js';

console.log(processA('test')); // 'A: test'
console.log(processB('test')); // 'B: test'
```

---

## Step 480: Re-exporting

```javascript
// Re-export: นำ export จาก module อื่นมา export ต่อ
// มีประโยชน์ในการสร้าง "barrel file"

// === utils/string.js ===
export function capitalize(str) { return str.charAt(0).toUpperCase() + str.slice(1); }
export function truncate(str, len) { return str.slice(0, len) + '...'; }

// === utils/array.js ===
export function unique(arr) { return [...new Set(arr)]; }
export function flatten(arr) { return arr.flat(Infinity); }
export function chunk(arr, size) {
  return Array.from({ length: Math.ceil(arr.length / size) }, (_, i) =>
    arr.slice(i * size, (i + 1) * size)
  );
}

// === utils/object.js ===
export function pick(obj, keys) {
  return Object.fromEntries(keys.map(k => [k, obj[k]]));
}
export function omit(obj, keys) {
  const copy = { ...obj };
  keys.forEach(k => delete copy[k]);
  return copy;
}

// === utils/index.js (barrel file) ===
// Re-export ทั้งหมด
export * from './string.js';
export * from './array.js';
export * from './object.js';

// Re-export เฉพาะบางอย่าง
export { capitalize, truncate } from './string.js';

// Re-export พร้อม rename
export { unique as arrayUnique } from './array.js';

// Re-export default
export { default as StringUtils } from './string-module.js';
```

```javascript
// === app.js ===
// Import จาก barrel file - สะดวกมาก!
import { capitalize, unique, pick } from './utils/index.js';
// แทนที่จะเป็น:
// import { capitalize } from './utils/string.js';
// import { unique } from './utils/array.js';
// import { pick } from './utils/object.js';
```

---

## Step 481: Dynamic Imports - import()

```javascript
// Dynamic import - โหลดเฉพาะเมื่อต้องการ
// ส่งคืน Promise

// แบบ static (ต้องรู้ตอน compile time)
import { util } from './module.js'; // โหลดทันที

// แบบ dynamic (รู้ตอน runtime)
async function loadModule() {
  const module = await import('./module.js');
  return module;
}

// ใช้งานจริง: Lazy loading
async function handleUserAction() {
  // โหลด module เฉพาะเมื่อผู้ใช้ทำบางอย่าง
  const { heavyProcess } = await import('./heavy-module.js');
  heavyProcess();
}
```

```javascript
// Dynamic import ตาม condition
async function loadLocale(language) {
  let locale;
  
  switch (language) {
    case 'th':
      locale = await import('./locales/th.js');
      break;
    case 'en':
      locale = await import('./locales/en.js');
      break;
    default:
      locale = await import('./locales/en.js');
  }
  
  return locale.default;
}

// Dynamic import ตาม variable (ต้องระวัง: บาง bundler ไม่รองรับ)
async function loadPlugin(pluginName) {
  try {
    const plugin = await import(`./plugins/${pluginName}.js`);
    return plugin.default;
  } catch (error) {
    console.error(`Plugin "${pluginName}" not found`);
    return null;
  }
}
```

```javascript
// Code splitting กับ dynamic import
// (Pattern ที่ใช้ใน webpack, Vite, etc.)

// Component lazy loading (React-style)
async function lazyLoad(importFn) {
  const module = await importFn();
  return module.default || module;
}

const routes = {
  '/home': () => import('./pages/Home.js'),
  '/about': () => import('./pages/About.js'),
  '/contact': () => import('./pages/Contact.js'),
  '/dashboard': () => import('./pages/Dashboard.js'),
};

async function navigate(path) {
  const importFn = routes[path];
  if (!importFn) {
    console.error('Route not found:', path);
    return;
  }
  
  const page = await lazyLoad(importFn);
  renderPage(page);
}

// ใช้งาน
navigate('/dashboard'); // โหลด Dashboard.js เฉพาะเมื่อเข้า route นี้
```

---

## Step 482: Module Loading Behavior

```javascript
// Modules โหลดแบบ static analysis + async
// JavaScript engine อ่าน import statements ทั้งหมดก่อน execute

// === a.js ===
console.log('A กำลังโหลด');
export const fromA = 'ค่าจาก A';
console.log('A โหลดเสร็จ');

// === b.js ===
import { fromA } from './a.js';
console.log('B กำลังโหลด');
console.log('ได้รับจาก A:', fromA);
export const fromB = 'ค่าจาก B';
console.log('B โหลดเสร็จ');

// === main.js ===
import { fromB } from './b.js';
console.log('Main กำลังทำงาน');
console.log('ได้รับจาก B:', fromB);

// Output order:
// A กำลังโหลด
// A โหลดเสร็จ
// B กำลังโหลด
// ได้รับจาก A: ค่าจาก A
// B โหลดเสร็จ
// Main กำลังทำงาน
// ได้รับจาก B: ค่าจาก B
```

```javascript
// Modules execute เพียงครั้งเดียว (cached)
// === counter.js ===
let count = 0;
export function increment() { count++; }
export function getCount() { return count; }
console.log('counter.js executed');

// === a.js ===
import { increment } from './counter.js'; // counter.js executed
increment();

// === b.js ===
import { getCount } from './counter.js'; // ไม่ execute อีกครั้ง!
console.log(getCount()); // 1 (เห็น state เดียวกัน)

// === main.js ===
import './a.js';
import './b.js';
// Output:
// counter.js executed (ครั้งเดียว!)
// 1
```

---

## Step 483: Circular Dependencies

```javascript
// Circular dependency: A import B, B import A
// ควรหลีกเลี่ยง แต่ต้องเข้าใจพฤติกรรม

// === a.js ===
import { bValue } from './b.js';
export const aValue = 'A';
console.log('In A, bValue =', bValue); // อาจเป็น undefined ตอนนี้!

// === b.js ===
import { aValue } from './a.js';
export const bValue = 'B';
console.log('In B, aValue =', aValue); // อาจเป็น undefined ตอนนี้!

// ปัญหา: ตอน b.js execute, a.js ยังไม่ export ค่าออกมา
```

```javascript
// วิธีแก้ circular dependency

// วิธีที่ 1: ย้าย shared code ไป module กลาง
// === shared.js ===
export const SHARED_VALUE = 'shared';

// === a.js ===
import { SHARED_VALUE } from './shared.js'; // ไม่ circular แล้ว

// === b.js ===
import { SHARED_VALUE } from './shared.js';

// วิธีที่ 2: ใช้ function แทน value (lazy evaluation)
// === a.js ===
import { getB } from './b.js';
export const aValue = 'A';
export function getADependency() {
  return getB(); // เรียกตอน runtime ไม่ใช่ import time
}
```

---

## Step 484: Modules ใน Browser

```html
<!-- ใช้ type="module" ใน script tag -->
<!DOCTYPE html>
<html>
<head>
  <title>ES Modules Demo</title>
</head>
<body>
  <!-- type="module" บอก browser ว่าเป็น ES module -->
  <script type="module" src="./app.js"></script>
  
  <!-- Inline module -->
  <script type="module">
    import { greet } from './utils.js';
    greet('World');
  </script>
  
  <!-- defer by default: module scripts รอ DOM โหลดเสร็จก่อน -->
  <!-- crossorigin: modules ต้องการ CORS -->
  <!-- nomodule: fallback สำหรับ browser เก่า -->
  <script nomodule src="./bundle.js"></script>
</body>
</html>
```

```javascript
// Module scope ใน browser
// === app.js ===

// Variables ใน module ไม่ใช่ global!
const secret = 'ไม่ใช่ global';
window.secret; // undefined (ถ้าเป็น module)

// แต่ถ้าต้องการ global:
window.myApp = {
  version: '1.0.0',
};

// CORS: modules ต้องโหลดจาก server (ไม่ใช่ file://)
// ต้องใช้ local server เช่น Live Server, http-server, Vite
```

```javascript
// Dynamic import ใน browser
document.getElementById('loadBtn').addEventListener('click', async () => {
  // โหลด module เฉพาะเมื่อคลิกปุ่ม
  const { Chart } = await import('./chart.js');
  
  const chart = new Chart({
    container: '#chart-container',
    data: [1, 2, 3, 4, 5],
  });
  
  chart.render();
});

// Import maps - กำหนด aliases สำหรับ module specifiers
// HTML:
// <script type="importmap">
// {
//   "imports": {
//     "lodash": "https://cdn.jsdelivr.net/npm/lodash@4/lodash.min.js",
//     "@utils/": "/src/utils/"
//   }
// }
// </script>

// ใน JS:
// import _ from 'lodash'; // จะ resolve ตาม importmap
```

---

## Step 485: Modules ใน Node.js

```javascript
// Node.js รองรับทั้ง CommonJS และ ESM

// วิธีที่ 1: ใช้ .mjs extension
// utils.mjs - ES Module
export function hello() { return 'Hello from ESM!'; }

// app.mjs
import { hello } from './utils.mjs';
console.log(hello());
```

```json
// วิธีที่ 2: ตั้ง "type": "module" ใน package.json
// package.json
{
  "name": "my-app",
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "start": "node app.js"
  }
}
```

```javascript
// เมื่อใช้ "type": "module" ทุก .js ไฟล์เป็น ESM
// ถ้าต้องการ CommonJS ในบางไฟล์ ใช้ .cjs extension

// utils.js (ESM เพราะ "type": "module")
export const name = 'ESM Utils';

// legacy.cjs (CommonJS เสมอ)
const name = 'CJS Legacy';
module.exports = { name };
```

```javascript
// ESM ใน Node.js - ข้อแตกต่างสำคัญ
// === app.mjs ===

// 1. ต้องระบุ extension เสมอ
import { util } from './utils.js';    // ✓
// import { util } from './utils'; // ✗ Error ใน Node.js ESM

// 2. ไม่มี __dirname หรือ __filename
// แต่ใช้สิ่งนี้แทน:
import { fileURLToPath } from 'url';
import { dirname } from 'path';

const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);

console.log(__filename); // /path/to/app.mjs
console.log(__dirname);  // /path/to

// 3. import.meta
console.log(import.meta.url);  // file:///path/to/app.mjs

// 4. Top-level await (Node.js 14.8+)
const data = await fetch('https://api.example.com/data').then(r => r.json());
console.log(data);
```

---

## Step 486: CommonJS vs ESM เปรียบเทียบ

```javascript
// === CommonJS ===
// math.cjs
const add = (a, b) => a + b;
const multiply = (a, b) => a * b;

module.exports = { add, multiply };
// หรือ:
exports.add = add;
exports.multiply = multiply;

// app.cjs
const { add } = require('./math.cjs');
const math = require('./math.cjs');

// Dynamic require
const moduleName = 'lodash';
const _ = require(moduleName); // ทำได้!

// Conditional require
if (process.env.NODE_ENV === 'development') {
  require('./dev-tools');
}
```

```javascript
// === ES Modules ===
// math.mjs
export const add = (a, b) => a + b;
export const multiply = (a, b) => a * b;

// app.mjs
import { add } from './math.mjs';
import * as math from './math.mjs';

// ✗ ไม่ได้! static imports ต้องอยู่ top-level
// if (condition) {
//   import { util } from './util.js'; // SyntaxError!
// }

// ✓ ใช้ dynamic import แทน
async function conditionalLoad() {
  if (condition) {
    const { util } = await import('./util.js');
  }
}
```

```javascript
// ตาราง CommonJS vs ESM
/*
| Feature              | CommonJS          | ESM               |
|---------------------|-------------------|-------------------|
| Syntax              | require/exports   | import/export     |
| Loading             | Synchronous       | Asynchronous      |
| Analysis            | Runtime           | Static (parse)    |
| Tree shaking        | Hard              | Easy              |
| Top-level await     | No                | Yes (Node 14.8+)  |
| Browser support     | No (needs bundle) | Yes (modern)      |
| Node.js support     | Default           | With .mjs/"type"  |
| Circular deps       | Supported         | Supported (care!) |
| Dynamic imports     | require()         | import()          |
| __dirname           | Yes               | No (use meta)     |
| strict mode         | No (default)      | Always            |
*/
```

---

## Step 487: Barrel Files (index.js Pattern)

```javascript
// โครงสร้างโปรเจค
// src/
//   components/
//     Button.js
//     Input.js
//     Modal.js
//     Dropdown.js
//     index.js  ← barrel file
//   utils/
//     format.js
//     validate.js
//     api.js
//     index.js  ← barrel file
//   models/
//     User.js
//     Post.js
//     Comment.js
//     index.js  ← barrel file

// === components/index.js ===
export { default as Button } from './Button.js';
export { default as Input } from './Input.js';
export { default as Modal } from './Modal.js';
export { default as Dropdown } from './Dropdown.js';

// นอกจากนี้ยัง export named exports ได้
export * from './Button.js'; // ถ้า Button.js มี named exports ด้วย
```

```javascript
// === การใช้งาน ===

// ✗ ต้องหลาย imports
import Button from '../components/Button.js';
import Input from '../components/Input.js';
import Modal from '../components/Modal.js';

// ✓ ใช้ barrel file
import { Button, Input, Modal } from '../components/index.js';
// หรือสั้นกว่า:
import { Button, Input, Modal } from '../components';

// ✓ ดีมากสำหรับ libraries
import { useState, useEffect, useCallback } from 'react';
// react ใช้ barrel pattern เหมือนกัน!
```

```javascript
// Barrel file ที่ดี
// === utils/index.js ===

// Re-export โดยไม่ชนกัน
export { capitalize, truncate, countWords } from './string.js';
export { unique, flatten, chunk, zip } from './array.js';
export { pick, omit, merge, deepClone } from './object.js';
export { formatDate, parseDate, addDays } from './date.js';

// Re-export กับ namespace
import * as StringUtils from './string.js';
import * as ArrayUtils from './array.js';
export { StringUtils, ArrayUtils };

// Default export (ถ้าต้องการ)
export { default } from './main-util.js';
```

---

## Step 488: Tree Shaking

```javascript
// Tree Shaking = การตัด code ที่ไม่ได้ใช้ออก (bundlers เช่น Webpack, Rollup, Vite)

// === large-library.js ===
export function func1() { return 1; }
export function func2() { return 2; }
export function func3() { return 3; }
// ... 100 more functions

// === app.js ===
import { func1 } from './large-library.js';
// ถ้า bundler รองรับ tree shaking:
// func2, func3, และ functions อื่นๆ จะถูกตัดออก!
// Bundle size ลดลงมาก

// ESM รองรับ tree shaking ได้ดีกว่า CommonJS เพราะ:
// - Static analysis: รู้ตอน parse time ว่าใช้อะไรบ้าง
// - Side effect-free modules (ถ้า mark ใน package.json)
```

```json
// package.json - บอก bundler ว่า module ไม่มี side effects
{
  "name": "my-lib",
  "version": "1.0.0",
  "sideEffects": false,
  // หรือระบุเฉพาะไฟล์ที่มี side effects
  "sideEffects": [
    "./src/setup.js",
    "*.css"
  ]
}
```

```javascript
// Side effects คืออะไร
// ✗ มี side effects (ทำงานตอน import)
import './polyfills.js'; // modify global object
import './analytics.js'; // send analytics data
import './styles.css';   // inject CSS

// ✓ ไม่มี side effects (pure module)
import { add } from './math.js'; // แค่ export functions
```

```javascript
// Tips สำหรับ tree-shaking-friendly code

// ✓ Named exports ดีกว่า default object
// ดี (tree-shakeable)
export function add(a, b) { return a + b; }
export function subtract(a, b) { return a - b; }
export function multiply(a, b) { return a * b; }

// ✗ Default object ทำ tree-shaking ยาก
export default {
  add: (a, b) => a + b,
  subtract: (a, b) => a - b,
  multiply: (a, b) => a * b,
};

// ✓ Barrel files ที่ดี - re-export โดยตรง
export { add } from './add.js';      // tree-shakeable
// ✗ Barrel files ที่ไม่ดี
export * from './math.js'; // อาจป้องกัน tree-shaking บาง bundler
```

---

## Step 489: Module Patterns ขั้นสูง

```javascript
// Plugin System ด้วย Modules
// === plugin-system.js ===
class PluginSystem {
  #plugins = new Map();
  #hooks = new Map();
  
  register(name, plugin) {
    this.#plugins.set(name, plugin);
    return this;
  }
  
  async loadPlugin(name) {
    if (this.#plugins.has(name)) {
      return this.#plugins.get(name);
    }
    
    // Dynamic import
    try {
      const module = await import(`./plugins/${name}.js`);
      const plugin = module.default;
      this.register(name, plugin);
      return plugin;
    } catch (err) {
      throw new Error(`Plugin "${name}" not found: ${err.message}`);
    }
  }
  
  addHook(hookName, fn) {
    if (!this.#hooks.has(hookName)) {
      this.#hooks.set(hookName, []);
    }
    this.#hooks.get(hookName).push(fn);
    return this;
  }
  
  async runHook(hookName, ...args) {
    const hooks = this.#hooks.get(hookName) || [];
    let result = args[0];
    for (const hook of hooks) {
      result = await hook(result, ...args.slice(1));
    }
    return result;
  }
}

export default new PluginSystem(); // singleton
```

```javascript
// Module Federation pattern (Micro-frontends)
// === remote-module.js ===
export async function loadRemoteModule(url, moduleName) {
  // โหลด module จาก URL อื่น
  await loadScript(url);
  const container = window[moduleName];
  await container.init(__webpack_share_scopes__.default);
  const factory = await container.get('./Component');
  return factory();
}

async function loadScript(src) {
  return new Promise((resolve, reject) => {
    const script = document.createElement('script');
    script.src = src;
    script.onload = resolve;
    script.onerror = reject;
    document.head.appendChild(script);
  });
}
```

```javascript
// Singleton pattern ด้วย module
// === database.js ===
let connection = null;
let isConnecting = false;

async function connect() {
  if (connection) return connection;
  if (isConnecting) {
    // รอให้ connect เสร็จ
    while (isConnecting) {
      await new Promise(r => setTimeout(r, 10));
    }
    return connection;
  }
  
  isConnecting = true;
  connection = await createConnection();
  isConnecting = false;
  return connection;
}

export { connect };
// Module caching ทำให้ connection เป็น singleton
// ทุก import ของ database.js จะได้ connection เดียวกัน
```

---

## Step 490: สรุปและ Best Practices

```javascript
// === Project Structure Best Practices ===

// โครงสร้างที่แนะนำ:
// src/
//   index.js (entry point)
//   config/
//     index.js
//     database.js
//     auth.js
//   models/
//     index.js
//     User.js
//     Post.js
//   services/
//     index.js
//     UserService.js
//     AuthService.js
//   utils/
//     index.js
//     string.js
//     date.js
//   routes/
//     index.js
//     users.js
//     posts.js
```

```javascript
// Best Practice 1: Explicit exports
// ✓ ดี - รู้ว่า export อะไร
export { UserService } from './UserService.js';
export { AuthService } from './AuthService.js';

// Best Practice 2: Group related exports
// === models/index.js ===
export { default as User } from './User.js';
export { default as Post } from './Post.js';
export { default as Comment } from './Comment.js';

// Best Practice 3: Avoid circular dependencies
// ใช้ dependency injection แทน

// Best Practice 4: Use dynamic imports for large features
const LazyChart = async () => {
  const { Chart } = await import('./Chart.js');
  return Chart;
};

// Best Practice 5: Mark side effects
// package.json: "sideEffects": ["./src/setup.js"]
```

```javascript
// สรุปทุก export/import patterns
// === complete-module.js ===

// Exports:
export const CONSTANT = 42;
export function namedFunction() {}
export class NamedClass {}
export { existing as renamed };
export { a, b, c };
export * from './other.js';
export { x as default };
export default value;

// Imports:
import defaultExport from './module.js';
import * as namespace from './module.js';
import { named } from './module.js';
import { named as alias } from './module.js';
import defaultExport, { named } from './module.js';
import './side-effects-only.js';

// Dynamic:
const module = await import('./dynamic.js');
```

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: สร้าง Math Library

```javascript
// Task: สร้าง math library ที่ใช้ ES Modules

// === math/basic.js ===
export const add = (a, b) => a + b;
export const subtract = (a, b) => a - b;
export const multiply = (a, b) => a * b;
export const divide = (a, b) => {
  if (b === 0) throw new Error('Division by zero');
  return a / b;
};

// === math/advanced.js ===
export const power = (base, exp) => base ** exp;
export const sqrt = (n) => {
  if (n < 0) throw new Error('Cannot sqrt negative number');
  return Math.sqrt(n);
};
export const factorial = (n) => {
  if (n < 0) throw new Error('Cannot factorial negative number');
  if (n === 0 || n === 1) return 1;
  return n * factorial(n - 1);
};

// === math/statistics.js ===
export const sum = (arr) => arr.reduce((a, b) => a + b, 0);
export const mean = (arr) => sum(arr) / arr.length;
export const median = (arr) => {
  const sorted = [...arr].sort((a, b) => a - b);
  const mid = Math.floor(sorted.length / 2);
  return sorted.length % 2
    ? sorted[mid]
    : (sorted[mid - 1] + sorted[mid]) / 2;
};
export const variance = (arr) => {
  const avg = mean(arr);
  return mean(arr.map(x => (x - avg) ** 2));
};
export const stdDev = (arr) => sqrt(variance(arr));

// === math/index.js (barrel) ===
export * from './basic.js';
export * from './advanced.js';
export * from './statistics.js';

// ใช้งาน:
// import { add, factorial, mean, stdDev } from './math/index.js';
const data = [2, 4, 4, 4, 5, 5, 7, 9];
// console.log(mean(data)); // 5
// console.log(stdDev(data)); // 2
```

### แบบฝึกหัดที่ 2: สร้าง Plugin Architecture

```javascript
// Task: สร้าง system ที่รองรับ plugins

class EventSystem {
  #handlers = new Map();
  
  on(event, handler) {
    if (!this.#handlers.has(event)) {
      this.#handlers.set(event, new Set());
    }
    this.#handlers.get(event).add(handler);
    return () => this.off(event, handler); // returns unsubscribe fn
  }
  
  off(event, handler) {
    this.#handlers.get(event)?.delete(handler);
  }
  
  emit(event, data) {
    this.#handlers.get(event)?.forEach(handler => handler(data));
  }
}

class App {
  #plugins = new Map();
  events = new EventSystem();
  
  use(name, plugin) {
    if (this.#plugins.has(name)) {
      console.warn(`Plugin "${name}" already registered`);
      return this;
    }
    
    plugin.install(this);
    this.#plugins.set(name, plugin);
    this.events.emit('plugin:registered', { name, plugin });
    return this;
  }
  
  getPlugin(name) {
    return this.#plugins.get(name);
  }
}

// === plugins/logger.js ===
const LoggerPlugin = {
  name: 'logger',
  
  install(app) {
    app.log = (level, message, data) => {
      const entry = {
        timestamp: new Date().toISOString(),
        level,
        message,
        data,
      };
      console.log(JSON.stringify(entry));
    };
    
    // Hook into events
    app.events.on('plugin:registered', ({ name }) => {
      app.log('info', `Plugin registered: ${name}`);
    });
  }
};

export default LoggerPlugin;

// ใช้งาน:
const app = new App();
// app.use('logger', LoggerPlugin);
// app.log('info', 'Server started', { port: 3000 });
```

### แบบฝึกหัดที่ 3: Lazy Loading System

```javascript
// Task: สร้าง lazy loading system สำหรับ components

class ComponentRegistry {
  #registry = new Map();
  #cache = new Map();
  
  register(name, importFn) {
    this.#registry.set(name, importFn);
    return this;
  }
  
  async load(name) {
    if (this.#cache.has(name)) {
      return this.#cache.get(name);
    }
    
    const importFn = this.#registry.get(name);
    if (!importFn) {
      throw new Error(`Component "${name}" not registered`);
    }
    
    console.log(`Loading component: ${name}`);
    const module = await importFn();
    const component = module.default || module;
    
    this.#cache.set(name, component);
    return component;
  }
  
  async loadAll(...names) {
    return Promise.all(names.map(name => this.load(name)));
  }
  
  preload(name) {
    // โหลดล่วงหน้าโดยไม่รอ
    this.load(name).catch(console.error);
  }
}

const registry = new ComponentRegistry();

// Register lazy components (ยังไม่โหลด)
registry
  .register('Button', () => import('./components/Button.js'))
  .register('Modal', () => import('./components/Modal.js'))
  .register('Chart', () => import('./components/Chart.js'));

// โหลดเมื่อต้องการ
async function renderPage(pageName) {
  if (pageName === 'dashboard') {
    const [Chart, Modal] = await registry.loadAll('Chart', 'Modal');
    // ใช้งาน Chart, Modal
  }
}
```

---

## สรุปบทนี้

ในบทนี้เราได้เรียนรู้:

1. **ประวัติ Modules** - IIFE, CommonJS, AMD, และทำไมถึงต้องมี ESM
2. **Named Exports** - export หลายค่าจาก module เดียว
3. **Default Exports** - export ค่าหลักของ module
4. **Import Syntax** - named, default, namespace, dynamic
5. **Re-exporting** - barrel files pattern
6. **Dynamic Imports** - import() สำหรับ lazy loading
7. **Module Loading** - execute ครั้งเดียว, cached
8. **Circular Dependencies** - ปัญหาและวิธีแก้
9. **Browser vs Node.js** - วิธีใช้ modules ในแต่ละ environment
10. **CommonJS vs ESM** - ข้อแตกต่างและการเลือกใช้
11. **Barrel Files** - organize exports
12. **Tree Shaking** - ลด bundle size

**หลักการสำคัญ:**
- ใช้ Named Exports เป็น default - ทำให้ tree-shaking ดีกว่า
- Barrel files ช่วยให้ import ง่ายขึ้น แต่ระวัง circular deps
- Dynamic imports เหมาะสำหรับ code splitting
- ESM คือ standard ของ JavaScript - ใช้เมื่อเป็นไปได้
- Module execute เพียงครั้งเดียว - เหมาะสำหรับ singleton
