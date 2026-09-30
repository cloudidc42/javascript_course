# Part 56: Node.js พื้นฐาน (Steps 1091-1110)

## บทนำ

Node.js คือ runtime environment สำหรับรัน JavaScript นอก browser โดยสร้างขึ้นบน V8 JavaScript engine ของ Google Chrome Node.js ทำให้เราสามารถใช้ JavaScript ในฝั่ง server ได้ ซึ่งเปิดโอกาสให้นักพัฒนา JavaScript สามารถเขียนทั้ง frontend และ backend ด้วยภาษาเดียวกัน

---

## Step 1091: Node.js คืออะไร และทำไมต้องใช้

Node.js ถูกสร้างโดย Ryan Dahl ในปี 2009 โดยมีจุดเด่นหลักคือ:

1. **Non-blocking I/O**: ไม่หยุดรอการทำงานที่ใช้เวลานาน
2. **Event-driven**: ทำงานด้วย event loop
3. **Single-threaded**: ใช้ thread เดียวแต่จัดการ concurrent requests ได้มาก
4. **npm ecosystem**: มี package มากกว่า 2 ล้าน package

### ทำไมต้องใช้ Node.js?

```javascript
// ตัวอย่าง: server แบบ traditional (blocking)
// ทุก request ต้องรอให้ request ก่อนหน้าเสร็จก่อน

// Node.js (non-blocking):
const fs = require('fs');

// อ่านไฟล์แบบ async - ไม่หยุดรอ
fs.readFile('data.txt', 'utf8', (err, data) => {
  if (err) {
    console.error('Error reading file:', err);
    return;
  }
  console.log('File content:', data);
});

console.log('This runs BEFORE the file is read!');
// Output:
// This runs BEFORE the file is read!
// File content: [content of data.txt]
```

### Use cases ที่เหมาะกับ Node.js

```javascript
// 1. REST APIs และ Web Services
// 2. Real-time applications (chat, gaming)
// 3. Streaming applications
// 4. CLI tools
// 5. Microservices

// ตัวอย่าง performance comparison concept
const scenarios = {
  traditional: {
    description: 'PHP/Ruby/Python แบบ synchronous',
    requestPerSecond: 1000,
    notes: 'แต่ละ request ใช้ thread ของตัวเอง'
  },
  nodejs: {
    description: 'Node.js แบบ async non-blocking',
    requestPerSecond: 10000,
    notes: 'thread เดียวจัดการทุก request ด้วย event loop'
  }
};

console.log('Node.js เหมาะกับ I/O intensive applications');
console.log('ไม่เหมาะกับ CPU intensive (ควรใช้ Worker Threads)');
```

---

## Step 1092: Node.js vs Browser JavaScript

แม้ทั้งคู่จะใช้ JavaScript แต่มีความแตกต่างสำคัญ:

```javascript
// ===== สิ่งที่มีใน Browser แต่ไม่มีใน Node.js =====
// - window object
// - document object
// - DOM APIs
// - fetch API (ต้อง polyfill หรือใช้ node-fetch ใน Node.js เก่า)
// - localStorage, sessionStorage
// - alert, confirm, prompt

// ===== สิ่งที่มีใน Node.js แต่ไม่มีใน Browser =====
// - process object
// - __dirname, __filename
// - require() function
// - module, exports
// - Built-in modules (fs, path, os, etc.)
// - Buffer class
// - global (แทน window)

// ===== เช็คว่าอยู่ใน Node.js หรือ Browser =====
const isNode = typeof process !== 'undefined' && 
               process.versions != null && 
               process.versions.node != null;

if (isNode) {
  console.log('Running in Node.js version:', process.versions.node);
} else {
  console.log('Running in browser');
}

// ===== Global objects =====
// Browser: window.setTimeout, window.setInterval
// Node.js: global.setTimeout, global.setInterval

// ใน Node.js สามารถใช้ได้โดยตรง:
setTimeout(() => {
  console.log('Both browser and Node.js have this!');
}, 1000);

// ===== Module System =====
// Browser (ES Modules): import/export
// Node.js (CommonJS): require/module.exports
// Node.js ก็รองรับ ES Modules ได้แล้วด้วย (.mjs หรือ "type": "module" ใน package.json)
```

```javascript
// ===== ความแตกต่างของ this =====

// Browser: this ใน global scope = window
// Node.js: this ใน global scope = module.exports (empty object)

console.log(this); // {} ใน Node.js, Window ใน browser

function showThis() {
  console.log(this); // undefined ใน strict mode, global ใน non-strict
}

// Arrow function
const arrowThis = () => {
  console.log(this); // inherits from enclosing scope
};

// ===== Error handling =====
// Node.js มี uncaughtException และ unhandledRejection

process.on('uncaughtException', (err) => {
  console.error('Uncaught Exception:', err.message);
  process.exit(1);
});

process.on('unhandledRejection', (reason, promise) => {
  console.error('Unhandled Rejection at:', promise, 'reason:', reason);
});
```

---

## Step 1093: การติดตั้ง Node.js และ npm

### วิธีการติดตั้ง

```bash
# วิธีที่ 1: ดาวน์โหลดจาก nodejs.org
# ไปที่ https://nodejs.org/en/download/
# เลือก LTS (Long Term Support) version

# วิธีที่ 2: ใช้ nvm (Node Version Manager) - แนะนำ!
# สำหรับ macOS/Linux:
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash

# หลังติดตั้ง nvm:
nvm install --lts          # ติดตั้ง LTS version ล่าสุด
nvm install 20.0.0         # ติดตั้ง version ที่ต้องการ
nvm use 20.0.0             # เปลี่ยนไปใช้ version นั้น
nvm ls                     # ดู version ที่ติดตั้ง
nvm current                # ดู version ที่ใช้อยู่

# วิธีที่ 3: ใช้ Homebrew (macOS)
brew install node

# สำหรับ Windows:
# ใช้ nvm-windows: https://github.com/coreybutler/nvm-windows
```

```bash
# ตรวจสอบการติดตั้ง
node --version    # หรือ node -v
npm --version     # หรือ npm -v

# Output ตัวอย่าง:
# v20.10.0
# 10.2.3
```

### npm (Node Package Manager)

```bash
# npm คือ package manager ที่มาพร้อมกับ Node.js
# ใช้สำหรับ:
# - ติดตั้ง packages
# - จัดการ dependencies
# - รัน scripts

# ดูข้อมูล npm
npm --version
npm help

# ค้นหา package
npm search lodash

# ดูข้อมูล package
npm info express
```

---

## Step 1094: Node.js REPL (Read-Eval-Print Loop)

REPL คือ interactive shell สำหรับทดสอบ JavaScript:

```bash
# เปิด REPL
node

# หรือ
node --experimental-repl-await  # รองรับ await ใน top level
```

```javascript
// ภายใน REPL:
> 1 + 1
2

> const name = 'Node.js'
undefined

> `Hello, ${name}!`
'Hello, Node.js!'

> [1, 2, 3].map(x => x * 2)
[ 2, 4, 6 ]

> // ใช้ _ สำหรับผลลัพธ์ล่าสุด
> 5 + 3
8
> _ * 2
16

> // คำสั่งพิเศษใน REPL
> .help        // ดูคำสั่งทั้งหมด
> .exit        // ออกจาก REPL
> .clear       // ล้าง context
> .save file   // บันทึก session ลงไฟล์
> .load file   // โหลดไฟล์เข้า REPL
> .editor      // เข้า editor mode (Ctrl+D เพื่อ execute)

> // Tab completion ใช้งานได้
> process.  // กด Tab สองครั้งเพื่อดู properties ทั้งหมด

> // Multi-line input
> function add(a, b) {
... return a + b;
... }
undefined
> add(5, 3)
8
```

```javascript
// เปิด REPL แบบ custom ด้วย code
const repl = require('repl');

const replServer = repl.start({
  prompt: 'myapp> ',
  useColors: true
});

// เพิ่ม variables เข้า REPL context
replServer.context.message = 'Hello from custom REPL!';
replServer.context.add = (a, b) => a + b;
```

---

## Step 1095: การรันไฟล์ JavaScript ด้วย node

```javascript
// สร้างไฟล์ hello.js
// hello.js
console.log('Hello, Node.js!');
console.log('Version:', process.version);
console.log('Platform:', process.platform);
```

```bash
# รันไฟล์
node hello.js

# รันพร้อม arguments
node hello.js arg1 arg2

# รัน script จาก package.json
npm start
npm test
npm run build
```

```javascript
// การรับ command line arguments
// args.js
const args = process.argv;
console.log('All arguments:', args);
// args[0] = path to node
// args[1] = path to script
// args[2+] = user arguments

const userArgs = process.argv.slice(2);
console.log('User arguments:', userArgs);

// รัน: node args.js hello world
// Output:
// All arguments: ['/usr/local/bin/node', '/path/to/args.js', 'hello', 'world']
// User arguments: ['hello', 'world']
```

```javascript
// ตัวอย่าง CLI tool
// calculator.js
const [,, operation, a, b] = process.argv;

const num1 = parseFloat(a);
const num2 = parseFloat(b);

if (isNaN(num1) || isNaN(num2)) {
  console.error('Please provide valid numbers');
  process.exit(1);
}

const operations = {
  add: (x, y) => x + y,
  sub: (x, y) => x - y,
  mul: (x, y) => x * y,
  div: (x, y) => {
    if (y === 0) throw new Error('Division by zero');
    return x / y;
  }
};

if (!operations[operation]) {
  console.error('Unknown operation. Use: add, sub, mul, div');
  process.exit(1);
}

const result = operations[operation](num1, num2);
console.log(`${num1} ${operation} ${num2} = ${result}`);

// รัน: node calculator.js add 5 3
// Output: 5 add 3 = 8
```

---

## Step 1096: Global Objects ใน Node.js

### __dirname และ __filename

```javascript
// globals.js
console.log('__dirname:', __dirname);
// Output: /home/user/myproject

console.log('__filename:', __filename);
// Output: /home/user/myproject/globals.js

// ใช้งานจริง: สร้าง path แบบ absolute
const path = require('path');
const dataFile = path.join(__dirname, 'data', 'users.json');
console.log('Data file path:', dataFile);

// ทำไม __dirname ดีกว่า relative path?
// เพราะ __dirname คือ directory ของไฟล์นั้น
// ไม่ใช่ directory ที่รัน node command

// ถ้าใช้ './data/users.json' จะอิงกับ current working directory
// แต่ path.join(__dirname, 'data/users.json') จะอิงกับ ที่อยู่ของไฟล์เสมอ
```

### global object

```javascript
// global คือ global scope ของ Node.js (เหมือน window ใน browser)

// ดู global properties
console.log(Object.keys(global));

// เพิ่ม global variable (ไม่แนะนำ!)
global.myGlobal = 'I am global';
console.log(myGlobal); // 'I am global'

// ตัวอย่างที่ควรหลีกเลี่ยง:
global.db = require('./database'); // ไม่ดี!
// ควรใช้ require/import แทน

// global functions ที่ใช้บ่อย:
setTimeout(() => console.log('Timeout!'), 1000);
setInterval(() => console.log('Interval!'), 2000);
clearTimeout(timerId);
clearInterval(intervalId);
setImmediate(() => console.log('Immediate!'));

// console เป็น global เช่นกัน
console.log('log');
console.error('error');
console.warn('warn');
console.info('info');
console.table([{name: 'Alice', age: 30}]);
console.time('timer');
console.timeEnd('timer');
console.group('Group');
console.groupEnd();
```

---

## Step 1097: process object

process object มีข้อมูลสำคัญเกี่ยวกับ Node.js process ปัจจุบัน:

```javascript
// process.argv - command line arguments
console.log(process.argv);
// ['/path/to/node', '/path/to/script.js', ...args]

// process.env - environment variables
console.log(process.env.NODE_ENV);    // 'development', 'production', etc.
console.log(process.env.HOME);        // home directory
console.log(process.env.PATH);        // system PATH

// กำหนด env var เมื่อรัน
// NODE_ENV=production node app.js

// process.exit() - จบการทำงาน
process.exit(0);    // success
process.exit(1);    // failure

// process.cwd() - current working directory
console.log(process.cwd());
// เปรียบเทียบกับ __dirname:
// process.cwd() = directory ที่รัน node command
// __dirname = directory ของไฟล์ปัจจุบัน

// process.platform
console.log(process.platform);
// 'linux', 'darwin', 'win32'

// ตรวจสอบ OS
if (process.platform === 'win32') {
  console.log('Windows');
} else if (process.platform === 'darwin') {
  console.log('macOS');
} else {
  console.log('Linux/Unix');
}

// process.version และ process.versions
console.log(process.version);          // Node.js version: 'v20.10.0'
console.log(process.versions);         // { node: '20.10.0', v8: '11.3.244.8', ... }
console.log(process.versions.node);    // '20.10.0'
console.log(process.versions.v8);      // V8 engine version

// process.pid - Process ID
console.log('PID:', process.pid);

// process.ppid - Parent Process ID
console.log('Parent PID:', process.ppid);

// process.memoryUsage()
const mem = process.memoryUsage();
console.log('Memory Usage:');
console.log('  rss:', Math.round(mem.rss / 1024 / 1024), 'MB');
console.log('  heapTotal:', Math.round(mem.heapTotal / 1024 / 1024), 'MB');
console.log('  heapUsed:', Math.round(mem.heapUsed / 1024 / 1024), 'MB');
console.log('  external:', Math.round(mem.external / 1024 / 1024), 'MB');

// process.uptime() - เวลาที่รันมา (วินาที)
console.log('Uptime:', process.uptime(), 'seconds');

// process.hrtime() - high resolution time
const start = process.hrtime();
// ... do something ...
const end = process.hrtime(start);
console.log('Elapsed:', end[0] * 1e9 + end[1], 'nanoseconds');

// process.hrtime.bigint() - แบบใหม่
const startBig = process.hrtime.bigint();
// ... do something ...
const endBig = process.hrtime.bigint();
console.log('Elapsed:', endBig - startBig, 'ns');
```

```javascript
// process.stdin, process.stdout, process.stderr
// รับ input จาก keyboard
process.stdout.write('Enter your name: ');

process.stdin.setEncoding('utf8');
process.stdin.on('data', (input) => {
  const name = input.trim();
  process.stdout.write(`Hello, ${name}!\n`);
  process.stdin.pause();
});

// process events
process.on('exit', (code) => {
  console.log(`About to exit with code: ${code}`);
  // ทำงานได้เฉพาะ synchronous code
});

process.on('beforeExit', (code) => {
  console.log(`Process beforeExit event with code: ${code}`);
  // สามารถเพิ่ม async work ได้
});

process.on('SIGINT', () => {
  console.log('\nReceived SIGINT (Ctrl+C)');
  // cleanup code
  process.exit(0);
});

process.on('SIGTERM', () => {
  console.log('Received SIGTERM');
  // graceful shutdown
  process.exit(0);
});
```

---

## Step 1098: Module System ใน Node.js (CommonJS)

### require และ module.exports

```javascript
// ===== การสร้าง module =====
// math.js
function add(a, b) {
  return a + b;
}

function subtract(a, b) {
  return a - b;
}

function multiply(a, b) {
  return a * b;
}

// export แบบต่างๆ

// วิธี 1: export ทีละ function
module.exports.add = add;
module.exports.subtract = subtract;
module.exports.multiply = multiply;

// วิธี 2: export เป็น object
module.exports = {
  add,
  subtract,
  multiply
};

// วิธี 3: สร้าง object แล้ว export
const math = {};
math.add = add;
math.subtract = subtract;
module.exports = math;
```

```javascript
// ===== การใช้ module =====
// main.js
const math = require('./math');      // ใส่ .js หรือไม่ก็ได้
const { add, subtract } = require('./math');  // destructuring

console.log(math.add(5, 3));        // 8
console.log(subtract(10, 4));       // 6

// require built-in modules (ไม่ต้องใส่ path)
const fs = require('fs');
const path = require('path');

// require npm packages (ต้องติดตั้งก่อน)
const express = require('express');
const lodash = require('lodash');
```

```javascript
// ===== Module caching =====
// Node.js cache modules หลังจาก require ครั้งแรก
// ครั้งต่อไปจะได้ object เดิม

// counter.js
let count = 0;
module.exports = {
  increment: () => ++count,
  getCount: () => count
};

// main.js
const counter1 = require('./counter');
const counter2 = require('./counter');

counter1.increment();
counter1.increment();

console.log(counter1.getCount()); // 2
console.log(counter2.getCount()); // 2 (same object!)
console.log(counter1 === counter2); // true (same reference)

// ล้าง cache (ไม่แนะนำใน production)
delete require.cache[require.resolve('./counter')];
```

```javascript
// ===== exports shortcut =====
// exports เป็น shortcut ของ module.exports

// แบบนี้ใช้ได้:
exports.hello = () => 'Hello!';
exports.world = () => 'World!';

// แบบนี้ใช้ไม่ได้! (เพราะ reassign exports):
exports = { hello: () => 'Hello!' }; // ผิด!
// ต้องใช้ module.exports แทน:
module.exports = { hello: () => 'Hello!' }; // ถูก!

// เหตุผล: exports เป็น reference ไปยัง module.exports
// ถ้า reassign exports จะทำให้ lost reference
```

```javascript
// ===== Circular dependencies =====
// a.js
const b = require('./b');
console.log('In a, b.done =', b.done);
exports.done = false;
b.getA();
exports.done = true;

// b.js
const a = require('./a');
exports.done = false;
exports.getA = () => {
  console.log('In b, a.done =', a.done);
};
exports.done = true;

// Node.js จัดการ circular dependencies ได้
// แต่อาจได้รับ incomplete exports object
// ควรหลีกเลี่ยง circular dependencies
```

```javascript
// ===== ES Modules ใน Node.js =====
// ใช้ .mjs extension หรือ "type": "module" ใน package.json

// math.mjs
export function add(a, b) {
  return a + b;
}

export function subtract(a, b) {
  return a - b;
}

export default {
  add,
  subtract
};

// main.mjs
import math, { add, subtract } from './math.mjs';
import { readFile } from 'fs/promises';

console.log(add(5, 3));

// ใน ES Modules ไม่มี __dirname และ __filename
// ต้องใช้ import.meta.url แทน
import { fileURLToPath } from 'url';
import { dirname } from 'path';

const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);
```

---

## Step 1099: Built-in Module - fs (File System)

### Asynchronous (Callback-based)

```javascript
const fs = require('fs');

// ===== readFile =====
fs.readFile('./data.txt', 'utf8', (err, data) => {
  if (err) {
    console.error('Error reading file:', err.message);
    return;
  }
  console.log('Content:', data);
});

// อ่านเป็น Buffer (ไม่ระบุ encoding)
fs.readFile('./image.png', (err, buffer) => {
  if (err) throw err;
  console.log('Buffer length:', buffer.length);
});

// ===== writeFile =====
const content = 'Hello, Node.js!\nThis is line 2';
fs.writeFile('./output.txt', content, 'utf8', (err) => {
  if (err) {
    console.error('Error writing file:', err.message);
    return;
  }
  console.log('File written successfully');
});

// เขียนทับไฟล์เดิม
fs.writeFile('./config.json', JSON.stringify({name: 'app', version: '1.0'}, null, 2), (err) => {
  if (err) throw err;
  console.log('Config saved');
});

// ===== appendFile =====
fs.appendFile('./log.txt', `\n${new Date().toISOString()} - New log entry`, (err) => {
  if (err) throw err;
  console.log('Log appended');
});

// ===== readdir =====
fs.readdir('./', (err, files) => {
  if (err) throw err;
  console.log('Files:', files);
  
  // กรองเฉพาะไฟล์ .js
  const jsFiles = files.filter(f => f.endsWith('.js'));
  console.log('JS files:', jsFiles);
});

// ===== mkdir =====
fs.mkdir('./newFolder', { recursive: true }, (err) => {
  if (err && err.code !== 'EEXIST') {
    throw err;
  }
  console.log('Directory created');
});

// สร้าง nested directories
fs.mkdir('./a/b/c', { recursive: true }, (err) => {
  if (err) throw err;
  console.log('Nested directories created');
});

// ===== unlink (delete file) =====
fs.unlink('./temp.txt', (err) => {
  if (err) {
    if (err.code === 'ENOENT') {
      console.log('File not found');
    } else {
      throw err;
    }
    return;
  }
  console.log('File deleted');
});

// ===== rename (move/rename file) =====
fs.rename('./old.txt', './new.txt', (err) => {
  if (err) throw err;
  console.log('File renamed');
});

// ===== stat (file information) =====
fs.stat('./data.txt', (err, stats) => {
  if (err) throw err;
  console.log('Is file:', stats.isFile());
  console.log('Is directory:', stats.isDirectory());
  console.log('Size:', stats.size, 'bytes');
  console.log('Created:', stats.birthtime);
  console.log('Modified:', stats.mtime);
});

// ===== exists (ตรวจสอบว่าไฟล์มีอยู่) =====
// ไม่แนะนำ fs.exists() เพราะ deprecated
// ใช้ fs.access แทน
fs.access('./data.txt', fs.constants.F_OK, (err) => {
  if (err) {
    console.log('File does not exist');
  } else {
    console.log('File exists');
  }
});

// ตรวจสอบ read/write permission
fs.access('./data.txt', fs.constants.R_OK | fs.constants.W_OK, (err) => {
  if (err) {
    console.log('No read/write permission');
  } else {
    console.log('Can read and write');
  }
});
```

### Synchronous (Blocking)

```javascript
const fs = require('fs');

// ===== Sync versions =====
// ใช้ได้แต่ block event loop - ใช้เฉพาะตอน startup

try {
  // readFileSync
  const data = fs.readFileSync('./data.txt', 'utf8');
  console.log('Content:', data);

  // writeFileSync
  fs.writeFileSync('./output.txt', 'Hello World');
  
  // appendFileSync
  fs.appendFileSync('./log.txt', '\nNew entry');
  
  // readdirSync
  const files = fs.readdirSync('./');
  console.log('Files:', files);
  
  // mkdirSync
  fs.mkdirSync('./newDir', { recursive: true });
  
  // unlinkSync
  fs.unlinkSync('./temp.txt');
  
  // statSync
  const stats = fs.statSync('./data.txt');
  console.log('File size:', stats.size);
  
  // existsSync
  const exists = fs.existsSync('./data.txt');
  console.log('File exists:', exists);
  
} catch (err) {
  console.error('Error:', err.message);
}
```

### Promises API (แนะนำ!)

```javascript
const fs = require('fs').promises;
// หรือ
const { readFile, writeFile, readdir } = require('fs/promises');

// ใช้ async/await
async function readConfig() {
  try {
    const data = await readFile('./config.json', 'utf8');
    const config = JSON.parse(data);
    return config;
  } catch (err) {
    if (err.code === 'ENOENT') {
      return {}; // return default config
    }
    throw err;
  }
}

async function processFiles() {
  const files = await readdir('./data');
  
  const contents = await Promise.all(
    files
      .filter(f => f.endsWith('.txt'))
      .map(f => readFile(`./data/${f}`, 'utf8'))
  );
  
  return contents;
}

// ตัวอย่างจริง: copy file
async function copyFile(src, dest) {
  const content = await readFile(src);
  await writeFile(dest, content);
  console.log(`Copied ${src} to ${dest}`);
}

copyFile('./source.txt', './destination.txt');

// ===== fs.watch - ดู file changes =====
const watcher = fs.watch('./src', { recursive: true }, (eventType, filename) => {
  console.log(`${eventType}: ${filename}`);
});

setTimeout(() => watcher.close(), 10000); // หยุดดูหลัง 10 วินาที
```

---

## Step 1100: Built-in Module - path

```javascript
const path = require('path');

// ===== path.join =====
// รวม path segments โดยใช้ OS separator ที่ถูกต้อง
const filePath = path.join('/home', 'user', 'documents', 'file.txt');
console.log(filePath);
// Unix: /home/user/documents/file.txt
// Windows: \home\user\documents\file.txt

// รับมือกับ ..
const cleanPath = path.join('/home/user', '../other', 'file.txt');
console.log(cleanPath); // /home/other/file.txt

// ===== path.resolve =====
// สร้าง absolute path จาก right to left
const abs1 = path.resolve('file.txt');
console.log(abs1); // /current/working/directory/file.txt

const abs2 = path.resolve('/home', 'user', 'file.txt');
console.log(abs2); // /home/user/file.txt

const abs3 = path.resolve('/home/user', '../other/file.txt');
console.log(abs3); // /home/other/file.txt

// ===== path.basename =====
// ชื่อไฟล์ส่วนท้ายสุด
console.log(path.basename('/home/user/file.txt'));        // file.txt
console.log(path.basename('/home/user/file.txt', '.txt')); // file (ไม่มี extension)
console.log(path.basename('/home/user/'));                // user

// ===== path.dirname =====
// directory ของ path
console.log(path.dirname('/home/user/file.txt'));   // /home/user
console.log(path.dirname('/home/user/'));           // /home

// ===== path.extname =====
// นามสกุลไฟล์
console.log(path.extname('file.txt'));          // .txt
console.log(path.extname('archive.tar.gz'));    // .gz
console.log(path.extname('file'));              // ''
console.log(path.extname('.hidden'));           // ''

// ===== path.parse =====
// แยก path เป็น components
const parsed = path.parse('/home/user/documents/file.txt');
console.log(parsed);
// {
//   root: '/',
//   dir: '/home/user/documents',
//   base: 'file.txt',
//   ext: '.txt',
//   name: 'file'
// }

// ===== path.format =====
// รวม components เป็น path
const formatted = path.format({
  dir: '/home/user/documents',
  name: 'file',
  ext: '.txt'
});
console.log(formatted); // /home/user/documents/file.txt

// ===== path.isAbsolute =====
console.log(path.isAbsolute('/home/user'));     // true
console.log(path.isAbsolute('./relative'));     // false
console.log(path.isAbsolute('C:\\Windows'));    // true (Windows)

// ===== path.relative =====
// relative path จาก from ไป to
const rel = path.relative('/home/user/docs', '/home/user/pics/photo.jpg');
console.log(rel); // ../pics/photo.jpg

// ===== path.sep =====
console.log(path.sep);     // '/' บน Unix, '\' บน Windows

// ===== path.delimiter =====
console.log(path.delimiter); // ':' บน Unix, ';' บน Windows
// ใช้แยก PATH environment variable

// ===== ตัวอย่างการใช้งานจริง =====
function getProjectPaths() {
  const root = path.resolve(__dirname, '..');
  return {
    root,
    src: path.join(root, 'src'),
    dist: path.join(root, 'dist'),
    config: path.join(root, 'config'),
    public: path.join(root, 'public'),
    logs: path.join(root, 'logs')
  };
}

function changeExtension(filePath, newExt) {
  const { dir, name } = path.parse(filePath);
  return path.format({ dir, name, ext: newExt });
}

console.log(changeExtension('/docs/readme.md', '.html'));
// /docs/readme.html
```

---

## Step 1101: Built-in Module - os

```javascript
const os = require('os');

// ===== os.platform() =====
console.log(os.platform());
// 'linux', 'darwin', 'win32', 'freebsd', etc.

// ===== os.arch() =====
console.log(os.arch());
// 'x64', 'arm64', 'ia32', etc.

// ===== os.cpus() =====
const cpus = os.cpus();
console.log('CPU count:', cpus.length);
console.log('First CPU:', cpus[0].model);
cpus.forEach((cpu, i) => {
  console.log(`CPU ${i + 1}: ${cpu.model} @ ${cpu.speed}MHz`);
});

// ===== Memory =====
const totalMem = os.totalmem();
const freeMem = os.freemem();
const usedMem = totalMem - freeMem;

console.log('Total Memory:', (totalMem / 1024 / 1024 / 1024).toFixed(2), 'GB');
console.log('Free Memory:', (freeMem / 1024 / 1024 / 1024).toFixed(2), 'GB');
console.log('Used Memory:', (usedMem / 1024 / 1024 / 1024).toFixed(2), 'GB');
console.log('Memory Usage:', ((usedMem / totalMem) * 100).toFixed(1) + '%');

// ===== os.homedir() =====
console.log('Home directory:', os.homedir());
// '/home/username' (Unix) or 'C:\Users\username' (Windows)

// ===== os.hostname() =====
console.log('Hostname:', os.hostname());

// ===== os.tmpdir() =====
console.log('Temp directory:', os.tmpdir());
// '/tmp' หรือ '/var/folders/...' บน macOS

// ===== os.uptime() =====
const uptime = os.uptime();
const hours = Math.floor(uptime / 3600);
const minutes = Math.floor((uptime % 3600) / 60);
const seconds = Math.floor(uptime % 60);
console.log(`System uptime: ${hours}h ${minutes}m ${seconds}s`);

// ===== os.networkInterfaces() =====
const nets = os.networkInterfaces();
for (const [name, interfaces] of Object.entries(nets)) {
  for (const iface of interfaces) {
    if (iface.family === 'IPv4' && !iface.internal) {
      console.log(`${name}: ${iface.address}`);
    }
  }
}

// ===== os.loadavg() =====
// Unix only: load average ช่วง 1, 5, 15 นาที
const [load1, load5, load15] = os.loadavg();
console.log(`Load Average: ${load1.toFixed(2)}, ${load5.toFixed(2)}, ${load15.toFixed(2)}`);

// ===== os constants =====
console.log(os.constants.signals.SIGKILL);  // 9
console.log(os.constants.errno.ENOENT);     // 2

// ===== ตัวอย่างการใช้งาน: System info report =====
function getSystemInfo() {
  return {
    platform: os.platform(),
    arch: os.arch(),
    hostname: os.hostname(),
    cpuCount: os.cpus().length,
    cpuModel: os.cpus()[0].model,
    totalMemory: `${(os.totalmem() / 1024 ** 3).toFixed(2)} GB`,
    freeMemory: `${(os.freemem() / 1024 ** 3).toFixed(2)} GB`,
    homeDir: os.homedir(),
    nodeVersion: process.version,
    uptime: `${Math.floor(os.uptime() / 3600)}h ${Math.floor((os.uptime() % 3600) / 60)}m`
  };
}

console.log(JSON.stringify(getSystemInfo(), null, 2));
```

---

## Step 1102: Built-in Module - events (EventEmitter)

```javascript
const EventEmitter = require('events');

// ===== สร้าง EventEmitter =====
const emitter = new EventEmitter();

// ===== on() - ฟัง event =====
emitter.on('data', (value) => {
  console.log('Received data:', value);
});

emitter.on('data', (value) => {
  console.log('Second listener:', value * 2);
});

// ===== emit() - ส่ง event =====
emitter.emit('data', 42);
// Output:
// Received data: 42
// Second listener: 84

// ===== once() - ฟัง event ครั้งเดียว =====
emitter.once('connect', () => {
  console.log('Connected! (only fires once)');
});

emitter.emit('connect'); // fires
emitter.emit('connect'); // ไม่ fires

// ===== removeListener() =====
const handler = (data) => console.log('Handler:', data);
emitter.on('message', handler);
emitter.emit('message', 'Hello');
emitter.removeListener('message', handler);
emitter.emit('message', 'World'); // handler ไม่ทำงานแล้ว

// ===== off() - alias ของ removeListener =====
emitter.off('message', handler);

// ===== removeAllListeners() =====
emitter.removeAllListeners('message');  // ลบ listeners ทั้งหมดของ event นี้
emitter.removeAllListeners();            // ลบทุก listeners

// ===== listenerCount() =====
emitter.on('test', () => {});
emitter.on('test', () => {});
console.log(emitter.listenerCount('test')); // 2

// ===== eventNames() =====
console.log(emitter.eventNames()); // ['data', 'test']

// ===== setMaxListeners() =====
// Default max listeners = 10 (warning ถ้าเกิน)
emitter.setMaxListeners(20);

// ===== error event =====
// ถ้า emit 'error' โดยไม่มี listener จะ throw exception
emitter.on('error', (err) => {
  console.error('Error:', err.message);
});

emitter.emit('error', new Error('Something went wrong'));
```

```javascript
// ===== Custom EventEmitter class =====
const EventEmitter = require('events');

class MyDatabase extends EventEmitter {
  constructor() {
    super();
    this.connected = false;
    this.data = new Map();
  }

  connect(host) {
    // จำลองการ connect
    setTimeout(() => {
      this.connected = true;
      this.emit('connect', { host, timestamp: new Date() });
    }, 100);
  }

  insert(key, value) {
    if (!this.connected) {
      this.emit('error', new Error('Not connected'));
      return;
    }
    this.data.set(key, value);
    this.emit('insert', { key, value });
  }

  find(key) {
    if (!this.connected) {
      this.emit('error', new Error('Not connected'));
      return null;
    }
    const value = this.data.get(key);
    this.emit('find', { key, value, found: value !== undefined });
    return value;
  }

  disconnect() {
    this.connected = false;
    this.emit('disconnect', { timestamp: new Date() });
  }
}

// การใช้งาน
const db = new MyDatabase();

db.on('connect', ({ host }) => {
  console.log(`Connected to ${host}`);
  db.insert('user:1', { name: 'Alice', age: 30 });
  db.insert('user:2', { name: 'Bob', age: 25 });
  
  const user = db.find('user:1');
  console.log('Found:', user);
  
  db.disconnect();
});

db.on('insert', ({ key, value }) => {
  console.log(`Inserted ${key}:`, value);
});

db.on('find', ({ key, found }) => {
  console.log(`Find ${key}: ${found ? 'found' : 'not found'}`);
});

db.on('disconnect', () => {
  console.log('Disconnected');
});

db.on('error', (err) => {
  console.error('DB Error:', err.message);
});

db.connect('localhost:5432');
```

---

## Step 1103: Built-in Module - http

```javascript
const http = require('http');

// ===== สร้าง HTTP server =====
const server = http.createServer((req, res) => {
  // req = IncomingMessage (request)
  // res = ServerResponse (response)
  
  console.log(`${req.method} ${req.url}`);
  
  // กำหนด response headers
  res.setHeader('Content-Type', 'application/json');
  res.setHeader('X-Powered-By', 'Node.js');
  
  // routing แบบง่าย
  if (req.url === '/' && req.method === 'GET') {
    res.statusCode = 200;
    res.end(JSON.stringify({ message: 'Hello, World!' }));
  } else if (req.url === '/health' && req.method === 'GET') {
    res.statusCode = 200;
    res.end(JSON.stringify({ status: 'ok', uptime: process.uptime() }));
  } else {
    res.statusCode = 404;
    res.end(JSON.stringify({ error: 'Not Found' }));
  }
});

server.listen(3000, () => {
  console.log('Server running at http://localhost:3000/');
});

// ===== รับ request body =====
const server2 = http.createServer((req, res) => {
  if (req.method === 'POST' && req.url === '/data') {
    let body = '';
    
    req.on('data', chunk => {
      body += chunk.toString();
    });
    
    req.on('end', () => {
      try {
        const data = JSON.parse(body);
        console.log('Received:', data);
        
        res.setHeader('Content-Type', 'application/json');
        res.statusCode = 201;
        res.end(JSON.stringify({ received: data }));
      } catch (err) {
        res.statusCode = 400;
        res.end(JSON.stringify({ error: 'Invalid JSON' }));
      }
    });
  }
});

// ===== HTTP client =====
// ทำ HTTP request
const options = {
  hostname: 'jsonplaceholder.typicode.com',
  path: '/todos/1',
  method: 'GET',
  headers: {
    'Accept': 'application/json'
  }
};

const req = http.request(options, (res) => {
  let data = '';
  
  res.on('data', (chunk) => {
    data += chunk;
  });
  
  res.on('end', () => {
    console.log('Response:', JSON.parse(data));
  });
});

req.on('error', (err) => {
  console.error('Request error:', err.message);
});

req.end();

// ===== https (HTTPS) =====
const https = require('https');

https.get('https://jsonplaceholder.typicode.com/users/1', (res) => {
  let data = '';
  res.on('data', chunk => data += chunk);
  res.on('end', () => console.log(JSON.parse(data)));
}).on('error', err => console.error(err));
```

---

## Step 1104: Built-in Module - url

```javascript
const { URL, URLSearchParams } = require('url');

// ===== URL class =====
const myURL = new URL('https://example.com:8080/path/to/page?name=Alice&age=30#section');

console.log(myURL.protocol);   // 'https:'
console.log(myURL.host);       // 'example.com:8080'
console.log(myURL.hostname);   // 'example.com'
console.log(myURL.port);       // '8080'
console.log(myURL.pathname);   // '/path/to/page'
console.log(myURL.search);     // '?name=Alice&age=30'
console.log(myURL.hash);       // '#section'
console.log(myURL.origin);     // 'https://example.com:8080'
console.log(myURL.href);       // Full URL string

// แก้ไข URL
myURL.pathname = '/new/path';
myURL.searchParams.append('city', 'Bangkok');
console.log(myURL.href);

// ===== URLSearchParams =====
const params = new URLSearchParams('name=Alice&age=30&hobby=coding&hobby=reading');

console.log(params.get('name'));        // 'Alice'
console.log(params.get('hobby'));       // 'coding' (first value)
console.log(params.getAll('hobby'));    // ['coding', 'reading']
console.log(params.has('age'));         // true
console.log(params.has('email'));       // false

// วนลูป
for (const [key, value] of params) {
  console.log(`${key}: ${value}`);
}

// แก้ไข
params.set('name', 'Bob');           // แทนที่ค่าเดิม
params.append('hobby', 'gaming');    // เพิ่มค่าใหม่
params.delete('age');                // ลบ key
console.log(params.toString());

// ===== สร้าง URL จาก base และ relative =====
const base = new URL('https://example.com/path/');
const relative = new URL('../other', base);
console.log(relative.href); // https://example.com/other

// ===== parse URL ใน Express-style =====
function parseRequestURL(req) {
  const url = new URL(req.url, `http://${req.headers.host}`);
  return {
    path: url.pathname,
    query: Object.fromEntries(url.searchParams)
  };
}
```

---

## Step 1105: Built-in Module - util

```javascript
const util = require('util');
const fs = require('fs');

// ===== util.promisify =====
// แปลง callback-based functions เป็น Promise-based

const readFile = util.promisify(fs.readFile);
const writeFile = util.promisify(fs.writeFile);

async function processFile() {
  try {
    const data = await readFile('./data.txt', 'utf8');
    const processed = data.toUpperCase();
    await writeFile('./output.txt', processed);
    console.log('Done!');
  } catch (err) {
    console.error('Error:', err.message);
  }
}

processFile();

// Custom function ที่ใช้ pattern (err, result)
function delay(ms, callback) {
  setTimeout(() => callback(null, `Waited ${ms}ms`), ms);
}

const delayAsync = util.promisify(delay);

delayAsync(1000).then(console.log); // 'Waited 1000ms'

// ===== util.callbackify =====
// แปลง async function เป็น callback-based
async function fetchUser(id) {
  return { id, name: 'Alice' };
}

const fetchUserCb = util.callbackify(fetchUser);

fetchUserCb(1, (err, user) => {
  if (err) throw err;
  console.log('User:', user);
});

// ===== util.inspect =====
// แสดงค่า object แบบละเอียด

const obj = {
  name: 'Alice',
  hobbies: ['coding', 'reading'],
  address: { city: 'Bangkok', country: 'Thailand' }
};

console.log(util.inspect(obj));
// { name: 'Alice', hobbies: [ 'coding', 'reading' ], address: { city: 'Bangkok', ... } }

console.log(util.inspect(obj, {
  depth: null,        // แสดง deep objects ทั้งหมด
  colors: true,       // สี (ใน terminal)
  compact: false,     // format แบบ pretty
  showHidden: false   // ไม่แสดง hidden properties
}));

// ===== util.format =====
// คล้าย printf ใน C

console.log(util.format('Hello, %s! You are %d years old.', 'Alice', 30));
// Hello, Alice! You are 30 years old.

console.log(util.format('Object: %o', { a: 1, b: 2 }));
console.log(util.format('JSON: %j', { a: 1, b: 2 }));

// ===== util.isDeepStrictEqual =====
const a = { x: 1, y: { z: 2 } };
const b = { x: 1, y: { z: 2 } };
const c = { x: 1, y: { z: 3 } };

console.log(util.isDeepStrictEqual(a, b)); // true
console.log(util.isDeepStrictEqual(a, c)); // false

// ===== util.deprecate =====
const oldFunction = util.deprecate(
  (x) => x * 2,
  'oldFunction is deprecated. Use newFunction instead.',
  'DEP001'
);

oldFunction(5); // WarningMessage + return 10

// ===== util.types =====
console.log(util.types.isAsyncFunction(async () => {})); // true
console.log(util.types.isGeneratorFunction(function* () {})); // true
console.log(util.types.isPromise(Promise.resolve())); // true
console.log(util.types.isDate(new Date())); // true
console.log(util.types.isRegExp(/hello/)); // true
console.log(util.types.isMap(new Map())); // true
```

---

## Step 1106: Built-in Module - crypto

```javascript
const crypto = require('crypto');

// ===== randomBytes =====
// สร้าง random bytes (ใช้สำหรับ tokens, passwords, etc.)
const token = crypto.randomBytes(32).toString('hex');
console.log('Token:', token);
// Output: 64 character hex string

const tokenBase64 = crypto.randomBytes(32).toString('base64');
console.log('Base64 token:', tokenBase64);
// URL-safe token
const urlSafeToken = crypto.randomBytes(32).toString('base64url');

// ===== randomUUID =====
const uuid = crypto.randomUUID();
console.log('UUID:', uuid);
// Output: 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'

// ===== createHash =====
// สร้าง hash (MD5, SHA256, etc.)

// SHA256 (แนะนำ)
const hash256 = crypto.createHash('sha256')
  .update('Hello, World!')
  .digest('hex');
console.log('SHA256:', hash256);

// SHA512
const hash512 = crypto.createHash('sha512')
  .update('password123')
  .digest('hex');
console.log('SHA512:', hash512);

// MD5 (ไม่ปลอดภัย แต่ใช้สำหรับ checksum)
const md5 = crypto.createHash('md5')
  .update('Hello')
  .digest('hex');
console.log('MD5:', md5);

// ===== HMAC =====
// Hash-based Message Authentication Code
const hmac = crypto.createHmac('sha256', 'secret-key')
  .update('data to authenticate')
  .digest('hex');
console.log('HMAC:', hmac);

// ===== Hash password (ใช้ bcrypt ใน production!) =====
function hashPassword(password, salt) {
  return crypto
    .createHash('sha256')
    .update(password + salt)
    .digest('hex');
}

const salt = crypto.randomBytes(16).toString('hex');
const hashed = hashPassword('myPassword123', salt);
console.log('Salt:', salt);
console.log('Hashed:', hashed);

// ===== Symmetric Encryption =====
// AES-256-GCM (แนะนำ)
function encrypt(text, key) {
  const iv = crypto.randomBytes(16);
  const cipher = crypto.createCipheriv('aes-256-gcm', key, iv);
  
  let encrypted = cipher.update(text, 'utf8', 'hex');
  encrypted += cipher.final('hex');
  
  const authTag = cipher.getAuthTag().toString('hex');
  
  return {
    encrypted,
    iv: iv.toString('hex'),
    authTag
  };
}

function decrypt(encryptedData, key) {
  const { encrypted, iv, authTag } = encryptedData;
  const decipher = crypto.createDecipheriv(
    'aes-256-gcm',
    key,
    Buffer.from(iv, 'hex')
  );
  
  decipher.setAuthTag(Buffer.from(authTag, 'hex'));
  
  let decrypted = decipher.update(encrypted, 'hex', 'utf8');
  decrypted += decipher.final('utf8');
  
  return decrypted;
}

const key = crypto.randomBytes(32); // 256-bit key
const message = 'Secret message in Thai: สวัสดีครับ';

const encryptedData = encrypt(message, key);
console.log('Encrypted:', encryptedData);

const decrypted = decrypt(encryptedData, key);
console.log('Decrypted:', decrypted);

// ===== scrypt สำหรับ password hashing =====
const { promisify } = require('util');
const scrypt = promisify(crypto.scrypt);

async function hashPasswordSecure(password) {
  const salt = crypto.randomBytes(16).toString('hex');
  const derivedKey = await scrypt(password, salt, 64);
  return `${salt}:${derivedKey.toString('hex')}`;
}

async function verifyPassword(password, hash) {
  const [salt, storedKey] = hash.split(':');
  const derivedKey = await scrypt(password, salt, 64);
  return storedKey === derivedKey.toString('hex');
}

async function demo() {
  const hash = await hashPasswordSecure('myPassword');
  console.log('Hash:', hash);
  
  const isValid = await verifyPassword('myPassword', hash);
  console.log('Valid:', isValid); // true
  
  const isInvalid = await verifyPassword('wrongPassword', hash);
  console.log('Invalid:', isInvalid); // false
}

demo();
```

---

## Step 1107: npm Basics

### npm init

```bash
# สร้าง package.json ใหม่
npm init

# สร้างแบบ skip questions (ใช้ defaults)
npm init -y
# หรือ
npm init --yes
```

```json
// package.json ตัวอย่าง
{
  "name": "my-awesome-app",
  "version": "1.0.0",
  "description": "My Node.js application",
  "main": "index.js",
  "scripts": {
    "start": "node index.js",
    "dev": "nodemon index.js",
    "test": "jest",
    "build": "tsc",
    "lint": "eslint src/",
    "format": "prettier --write src/"
  },
  "keywords": ["nodejs", "javascript"],
  "author": "Your Name <email@example.com>",
  "license": "MIT",
  "dependencies": {
    "express": "^4.18.2",
    "axios": "^1.6.0"
  },
  "devDependencies": {
    "nodemon": "^3.0.2",
    "jest": "^29.7.0",
    "eslint": "^8.54.0"
  },
  "engines": {
    "node": ">=18.0.0"
  }
}
```

### npm install

```bash
# ติดตั้ง package ใน dependencies
npm install express
npm i express           # shorthand

# ติดตั้ง package ใน devDependencies
npm install --save-dev nodemon
npm i -D nodemon        # shorthand

# ติดตั้ง package แบบ global
npm install -g typescript
npm i -g nodemon

# ติดตั้ง specific version
npm install express@4.18.2
npm install express@latest
npm install express@">=4.0.0 <5.0.0"

# ติดตั้งทุก dependencies จาก package.json
npm install
npm ci      # install แบบ clean (ใช้ package-lock.json)

# ติดตั้งเฉพาะ production dependencies
npm install --production
npm install --omit=dev
```

### npm uninstall, update, list

```bash
# ลบ package
npm uninstall express
npm remove express      # same thing
npm rm express          # shorthand

# ลบ global package
npm uninstall -g typescript

# อัปเดต packages
npm update              # อัปเดตทุก packages
npm update express      # อัปเดต specific package
npm outdated            # ดู packages ที่มี version ใหม่

# ดูรายการ packages ที่ติดตั้ง
npm list                # tree format
npm ls                  # same
npm list --depth=0      # แสดงระดับเดียว
npm list -g --depth=0   # global packages

# ตรวจสอบ security vulnerabilities
npm audit
npm audit fix

# รัน scripts
npm run start
npm run dev
npm test            # shorthand สำหรับ npm run test
npm start           # shorthand สำหรับ npm run start

# ดูข้อมูล package
npm info express
npm info express versions    # ดูทุก versions
```

---

## Step 1108: package.json และ package-lock.json

```json
// package.json - อธิบาย version format
{
  "dependencies": {
    "express": "4.18.2",        // Exact version
    "express": "^4.18.2",       // Compatible: >=4.18.2 <5.0.0
    "express": "~4.18.2",       // Patch updates: >=4.18.2 <4.19.0
    "express": "*",             // Any version (อันตราย!)
    "express": ">=4.0.0",       // Any >= 4.0.0
    "express": "latest"         // Latest published
  }
}
```

```bash
# package-lock.json
# - บันทึก exact versions ของ packages ทั้งหมด (รวม transitive dependencies)
# - ควร commit เข้า git เสมอ
# - ทำให้ install ซ้ำได้ผลเหมือนกันทุกครั้ง (reproducible builds)

# .npmrc - npm configuration
# สร้างไฟล์ .npmrc ในโปรเจค
# save-exact=true    # บันทึก exact version
# engine-strict=true # error ถ้า Node.js version ไม่ match
```

### npm scripts ขั้นสูง

```json
{
  "scripts": {
    "start": "node dist/index.js",
    "dev": "nodemon src/index.js",
    "build": "tsc",
    "clean": "rm -rf dist/",
    "prebuild": "npm run clean",      // รันก่อน build
    "postbuild": "echo Build done!",  // รันหลัง build
    "test": "jest --coverage",
    "test:watch": "jest --watch",
    "lint": "eslint src/ --ext .ts,.js",
    "lint:fix": "eslint src/ --ext .ts,.js --fix",
    "format": "prettier --write 'src/**/*.{ts,js,json}'",
    "check": "npm run lint && npm run test",
    "prepare": "husky install",
    "release": "npm version patch && git push && git push --tags"
  }
}
```

```bash
# npm lifecycle scripts
# prepare: รันก่อน pack/publish และหลัง install
# prepublishOnly: รันก่อน publish เท่านั้น
# postinstall: รันหลัง install
# pretest: รันก่อน test
# posttest: รันหลัง test
```

---

## Step 1109: node_modules และ .gitignore

```bash
# node_modules directory:
# - เก็บ packages ที่ติดตั้ง
# - อาจมีขนาดใหญ่มาก (GB)
# - ต้อง add ใน .gitignore เสมอ!
# - สร้างใหม่ได้ด้วย npm install จาก package.json

# .gitignore สำหรับ Node.js
```

```
# .gitignore

# Dependencies
node_modules/
npm-debug.log*
yarn-debug.log*
yarn-error.log*

# Environment variables (สำคัญมาก!)
.env
.env.local
.env.*.local

# Build outputs
dist/
build/
out/

# OS files
.DS_Store
.DS_Store?
._*
Thumbs.db

# IDE files
.idea/
.vscode/
*.swp
*.swo

# Logs
logs/
*.log

# Test coverage
coverage/

# Cache
.npm
.eslintcache
.cache/
```

```javascript
// ===== npm workspaces (Monorepo) =====
// package.json ใน root
{
  "name": "my-monorepo",
  "private": true,
  "workspaces": [
    "packages/*",
    "apps/*"
  ]
}

// structure:
// my-monorepo/
//   package.json
//   packages/
//     utils/
//       package.json
//     ui-components/
//       package.json
//   apps/
//     web/
//       package.json
//     api/
//       package.json

// ติดตั้งทุก workspaces
// npm install

// รัน script ใน workspace ที่ต้องการ
// npm run build -w packages/utils
// npm run build --workspace=packages/utils
```

---

## Step 1110: npx และ tools เพิ่มเติม

```bash
# npx - รัน npm packages โดยไม่ต้องติดตั้ง global
npx create-react-app my-app
npx typescript
npx prettier --write .
npx eslint src/

# npx รัน local scripts ก็ได้
npx jest

# ===== yarn (alternative package manager) =====
# ติดตั้ง
npm install -g yarn

# การใช้งาน
yarn init           # npm init
yarn add express    # npm install express
yarn add -D jest    # npm install --save-dev jest
yarn remove express # npm uninstall express
yarn                # npm install
yarn run start      # npm run start
yarn test           # npm test

# ===== pnpm (performant npm) =====
# ติดตั้ง
npm install -g pnpm

# การใช้งาน (คล้าย npm)
pnpm init
pnpm add express
pnpm install
pnpm run start

# ===== package.json engines field =====
{
  "engines": {
    "node": ">=18.0.0 <21.0.0",
    "npm": ">=9.0.0"
  }
}
```

```javascript
// ===== ตัวอย่างโปรเจคสมบูรณ์ =====
// โครงสร้างไฟล์:
// my-project/
//   src/
//     index.js
//     utils/
//       logger.js
//       helpers.js
//   tests/
//     utils.test.js
//   package.json
//   .gitignore
//   .env
//   .env.example

// src/utils/logger.js
const path = require('path');
const fs = require('fs');

class Logger {
  constructor(options = {}) {
    this.level = options.level || 'info';
    this.logFile = options.logFile || null;
    this.levels = { debug: 0, info: 1, warn: 2, error: 3 };
  }

  _log(level, ...args) {
    if (this.levels[level] < this.levels[this.level]) return;
    
    const timestamp = new Date().toISOString();
    const message = args.map(a => 
      typeof a === 'object' ? JSON.stringify(a) : String(a)
    ).join(' ');
    
    const formatted = `[${timestamp}] [${level.toUpperCase()}] ${message}`;
    
    if (level === 'error') {
      process.stderr.write(formatted + '\n');
    } else {
      process.stdout.write(formatted + '\n');
    }
    
    if (this.logFile) {
      fs.appendFileSync(this.logFile, formatted + '\n');
    }
  }

  debug(...args) { this._log('debug', ...args); }
  info(...args) { this._log('info', ...args); }
  warn(...args) { this._log('warn', ...args); }
  error(...args) { this._log('error', ...args); }
}

module.exports = new Logger({ level: 'info' });

// src/index.js
const http = require('http');
const path = require('path');
const logger = require('./utils/logger');

const PORT = process.env.PORT || 3000;

const server = http.createServer((req, res) => {
  logger.info(`${req.method} ${req.url}`);
  
  res.setHeader('Content-Type', 'application/json');
  
  const routes = {
    'GET /': () => ({ message: 'Welcome to My API', version: '1.0.0' }),
    'GET /health': () => ({ status: 'healthy', uptime: process.uptime() }),
  };
  
  const route = `${req.method} ${req.url}`;
  const handler = routes[route];
  
  if (handler) {
    res.statusCode = 200;
    res.end(JSON.stringify(handler()));
  } else {
    res.statusCode = 404;
    res.end(JSON.stringify({ error: 'Not Found', path: req.url }));
  }
});

server.listen(PORT, () => {
  logger.info(`Server started on port ${PORT}`);
  logger.info(`Platform: ${process.platform}, Node.js ${process.version}`);
});

server.on('error', (err) => {
  logger.error('Server error:', err.message);
  process.exit(1);
});

// graceful shutdown
process.on('SIGTERM', () => {
  logger.info('SIGTERM received, shutting down gracefully');
  server.close(() => {
    logger.info('Server closed');
    process.exit(0);
  });
});
```

---

## สรุป Steps 1091-1110

| Step | หัวข้อ |
|------|--------|
| 1091 | Node.js คืออะไรและทำไมต้องใช้ |
| 1092 | Node.js vs Browser JavaScript |
| 1093 | การติดตั้ง Node.js และ npm |
| 1094 | Node.js REPL |
| 1095 | การรันไฟล์ด้วย node |
| 1096 | Global objects: __dirname, __filename, global |
| 1097 | process object |
| 1098 | CommonJS Module System |
| 1099 | Built-in fs module |
| 1100 | Built-in path module |
| 1101 | Built-in os module |
| 1102 | Built-in events (EventEmitter) |
| 1103 | Built-in http module |
| 1104 | Built-in url module |
| 1105 | Built-in util module |
| 1106 | Built-in crypto module |
| 1107 | npm basics |
| 1108 | package.json และ package-lock.json |
| 1109 | node_modules และ .gitignore |
| 1110 | npx และ tools เพิ่มเติม |

---

## แบบฝึกหัดท้ายบท

### แบบฝึกหัดที่ 1: File Organizer
สร้างโปรแกรมที่:
1. รับ directory path จาก command line argument
2. อ่านรายการไฟล์ใน directory
3. จัดกลุ่มไฟล์ตาม extension (.jpg, .pdf, .txt, etc.)
4. สร้าง subdirectory สำหรับแต่ละประเภท
5. ย้ายไฟล์ไปยัง subdirectory ที่ถูกต้อง

```javascript
// starter code
const fs = require('fs').promises;
const path = require('path');

async function organizeFiles(dirPath) {
  // TODO: Implement
}

const targetDir = process.argv[2] || '.';
organizeFiles(targetDir).catch(console.error);
```

### แบบฝึกหัดที่ 2: Custom Logger
สร้าง Logger class ที่:
1. รองรับ levels: debug, info, warn, error
2. บันทึกลงไฟล์ตามวันที่ (logs/2024-01-15.log)
3. Rotate logs ที่เก่ากว่า 7 วัน
4. รองรับ formatting แบบ JSON และ text

### แบบฝึกหัดที่ 3: CLI Todo App
สร้าง command-line todo app ที่:
1. เก็บข้อมูลในไฟล์ JSON
2. รองรับ commands: add, list, complete, delete
3. แสดง colored output ด้วย ANSI escape codes

```bash
# ตัวอย่างการใช้งาน
node todo.js add "เรียน Node.js"
node todo.js list
node todo.js complete 1
node todo.js delete 1
```

### แบบฝึกหัดที่ 4: File Watcher
สร้างโปรแกรมที่:
1. ดูการเปลี่ยนแปลงในไฟล์/directory
2. เมื่อไฟล์ .js เปลี่ยน ให้แสดง diff
3. บันทึก events ลงใน log file

### แบบฝึกหัดที่ 5: HTTP File Server
สร้าง HTTP server ที่:
1. Serve ไฟล์จาก directory
2. แสดง directory listing (เหมือน `ls`)
3. รองรับ file download
4. แสดง error page สำหรับ 404

```javascript
// hint: ใช้ http, fs, path modules
const http = require('http');
const fs = require('fs');
const path = require('path');

// TODO: implement file server
```

---

*จบ Part 56: Node.js พื้นฐาน - ในบทต่อไปเราจะเรียน Node.js ขั้นสูง ได้แก่ Streams, Child Processes, Worker Threads และอื่นๆ*
