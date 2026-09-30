# ส่วนที่ 1: บทนำสู่ JavaScript

## คำอธิบาย
ยินดีต้อนรับสู่คอร์ส JavaScript ภาษาไทย! ในส่วนนี้คุณจะได้เรียนรู้พื้นฐานของ JavaScript ตั้งแต่ประวัติความเป็นมา การติดตั้งเครื่องมือ ไปจนถึงการเขียนโปรแกรมแรก เนื้อหาครอบคลุม Step 1-10 ของหลักสูตร

---

## Step 1: JavaScript คืออะไร?

JavaScript เป็นภาษาโปรแกรมมิ่งที่ถูกออกแบบมาเพื่อทำให้หน้าเว็บมีความสามารถในการโต้ตอบกับผู้ใช้ได้ ปัจจุบัน JavaScript เป็นหนึ่งในภาษาโปรแกรมมิ่งที่ได้รับความนิยมมากที่สุดในโลก

### ความสามารถหลักของ JavaScript:
- **Frontend**: ทำให้หน้าเว็บมีความสามารถโต้ตอบกับผู้ใช้
- **Backend**: รันบน server ด้วย Node.js
- **Mobile**: พัฒนาแอปมือถือด้วย React Native
- **Desktop**: พัฒนาแอปเดสก์ท็อปด้วย Electron
- **Game**: พัฒนาเกมด้วย Phaser, Three.js
- **AI/ML**: ใช้กับ TensorFlow.js

```javascript
// ตัวอย่างที่ 1: JavaScript ทำอะไรได้บ้าง?
// 1. เปลี่ยน HTML content
document.getElementById('title').textContent = 'สวัสดี JavaScript!';

// 2. เปลี่ยน CSS style
document.body.style.backgroundColor = '#f0f0f0';

// 3. ตอบสนองต่อ event ของผู้ใช้
document.getElementById('btn').addEventListener('click', function() {
    alert('คุณกดปุ่มแล้ว!');
});

// 4. ส่ง HTTP request
fetch('https://api.example.com/data')
    .then(response => response.json())
    .then(data => console.log(data));

// 5. จัดการข้อมูล
const numbers = [1, 2, 3, 4, 5];
const doubled = numbers.map(n => n * 2);
console.log(doubled); // [2, 4, 6, 8, 10]
```

---

## Step 2: ประวัติความเป็นมาของ JavaScript

### Timeline สำคัญ:

| ปี | เหตุการณ์ |
|-----|-----------|
| 1995 | Brendan Eich สร้าง JavaScript ใน 10 วัน ที่ Netscape |
| 1996 | Microsoft สร้าง JScript (เวอร์ชันของตัวเอง) |
| 1997 | ECMAScript มาตรฐานแรก (ES1) |
| 1999 | ES3 - เพิ่ม regex, try/catch |
| 2009 | ES5 - strict mode, JSON, Array methods |
| 2015 | ES6/ES2015 - let, const, arrow functions, class |
| 2016+ | ES2016, ES2017... อัพเดตรายปี |

```javascript
// ตัวอย่างที่ 2: วิวัฒนาการของ JavaScript
// ES5 (2009) - แบบเก่า
var name = 'สมชาย';
function greet(person) {
    return 'สวัสดี ' + person;
}

// ES6+ (2015+) - แบบใหม่
const greeting = (person) => `สวัสดี ${person}`;
console.log(greeting('สมชาย')); // สวัสดี สมชาย

// ES2022 - ฟีเจอร์ล่าสุด
const arr = [1, 2, 3, 4, 5];
console.log(arr.at(-1)); // 5 (นับจากท้าย)
```

---

## Step 3: การติดตั้งเครื่องมือ

### 3.1 ติดตั้ง Node.js

Node.js คือ JavaScript runtime ที่ทำให้เราสามารถรัน JavaScript นอก browser ได้

**ขั้นตอนการติดตั้ง:**
1. ไปที่ https://nodejs.org/
2. ดาวน์โหลด LTS version (แนะนำ)
3. ติดตั้งตามขั้นตอน
4. ตรวจสอบการติดตั้ง:

```bash
# ใน Terminal / Command Prompt
node --version   # v18.x.x หรือสูงกว่า
npm --version    # 9.x.x หรือสูงกว่า
```

### 3.2 ติดตั้ง VS Code

VS Code เป็น Code Editor ที่ดีที่สุดสำหรับ JavaScript

**Extensions ที่แนะนำ:**
- **ESLint** - ตรวจสอบ error ในโค้ด
- **Prettier** - จัดรูปแบบโค้ดอัตโนมัติ
- **JavaScript (ES6) code snippets** - code shortcuts
- **Live Server** - รัน HTML ใน browser แบบ real-time
- **GitLens** - เครื่องมือ Git ขั้นสูง

```javascript
// ตัวอย่างที่ 3: ทดสอบ Node.js
// สร้างไฟล์ test.js แล้วรันด้วย: node test.js
console.log('Node.js ทำงานได้!');
console.log('Node version:', process.version);
console.log('Platform:', process.platform);
```

### 3.3 Browser Developer Tools

```javascript
// ตัวอย่างที่ 4: เปิด Browser Console
// Windows/Linux: F12 หรือ Ctrl+Shift+I
// Mac: Cmd+Option+I

// แล้วพิมพ์ใน Console tab:
1 + 1                    // 2
"สวัสดี" + " " + "โลก"   // "สวัสดี โลก"
Math.random()            // ตัวเลขสุ่ม 0-1
new Date()               // วันเวลาปัจจุบัน
```

---

## Step 4: โปรแกรมแรกของคุณ

```javascript
// ตัวอย่างที่ 5: Hello World - โปรแกรมแรก
console.log('สวัสดีโลก!');
console.log('Hello, World!');

// ตัวอย่างที่ 6: การแสดงผลหลายวิธี
// วิธีที่ 1: console.log
console.log('วิธีที่ 1: console.log');

// วิธีที่ 2: alert (ใน browser เท่านั้น)
// alert('วิธีที่ 2: alert dialog');

// วิธีที่ 3: document.write (ใน browser เท่านั้น)
// document.write('<h1>สวัสดีโลก</h1>');

// วิธีที่ 4: innerHTML (ใน browser เท่านั้น)
// document.getElementById('output').innerHTML = 'สวัสดีโลก';
```

```javascript
// ตัวอย่างที่ 7: โปรแกรมที่ซับซ้อนขึ้นนิดหน่อย
// คำนวณอายุจากปีเกิด
const birthYear = 1995;
const currentYear = new Date().getFullYear();
const age = currentYear - birthYear;

console.log(`คุณเกิดปี: ${birthYear}`);
console.log(`ปีปัจจุบัน: ${currentYear}`);
console.log(`อายุประมาณ: ${age} ปี`);
```

---

## Step 5: ที่ที่เขียน JavaScript ได้

### 5.1 Inline JavaScript (ใน attribute)

```html
<!-- ตัวอย่างที่ 8: Inline JavaScript -->
<!DOCTYPE html>
<html>
<body>
    <!-- onclick ใน attribute -->
    <button onclick="alert('คุณกดปุ่มแล้ว!')">กดฉัน</button>
    
    <!-- onmouseover -->
    <p onmouseover="this.style.color='red'">วางเมาส์ที่นี่</p>
    
    <!-- ไม่แนะนำ: ยากต่อการ maintain -->
</body>
</html>
```

### 5.2 Script Tag (ใน HTML)

```html
<!-- ตัวอย่างที่ 9: Internal Script -->
<!DOCTYPE html>
<html>
<head>
    <title>JavaScript ใน Script Tag</title>
</head>
<body>
    <h1 id="greeting">กำลังโหลด...</h1>
    
    <!-- ใส่ก่อน </body> เพื่อให้ HTML โหลดก่อน -->
    <script>
        document.getElementById('greeting').textContent = 'สวัสดีจาก Script Tag!';
        console.log('Script ทำงานแล้ว');
    </script>
</body>
</html>
```

### 5.3 External JavaScript File (แนะนำที่สุด)

```html
<!-- ตัวอย่างที่ 10: External Script -->
<!DOCTYPE html>
<html>
<head>
    <title>External JavaScript</title>
</head>
<body>
    <h1 id="title">หน้าเว็บ</h1>
    
    <!-- โหลด script ภายนอก -->
    <script src="main.js"></script>
    
    <!-- หรือใช้ defer เพื่อโหลดหลัง HTML -->
    <script src="app.js" defer></script>
    
    <!-- หรือใช้ async สำหรับ script ที่ไม่ขึ้นกัน HTML -->
    <script src="analytics.js" async></script>
</body>
</html>
```

```javascript
// ตัวอย่างที่ 11: ไฟล์ main.js
// ไฟล์ JavaScript แยกต่างหาก - ดีที่สุดสำหรับโปรเจ็กต์จริง

function updateTitle(text) {
    document.getElementById('title').textContent = text;
}

// เรียกใช้ function
updateTitle('หน้าเว็บของฉัน');
```

### ความแตกต่างระหว่าง defer และ async:

```html
<!-- ตัวอย่างที่ 12: defer vs async -->

<!-- ปกติ: หยุด parse HTML รอ script โหลดและรัน -->
<script src="blocking.js"></script>

<!-- defer: โหลด script พร้อมกับ HTML, รันหลัง HTML parse เสร็จ -->
<!-- รักษาลำดับ script -->
<script src="first.js" defer></script>
<script src="second.js" defer></script>  <!-- รันหลัง first.js -->

<!-- async: โหลดและรัน script ทันทีที่โหลดเสร็จ -->
<!-- ไม่รักษาลำดับ - ใช้กับ script ที่ไม่ขึ้นกัน DOM -->
<script src="analytics.js" async></script>
```

---

## Step 6: Console Methods

Console เป็นเครื่องมือสำคัญสำหรับ debugging และดูผลลัพธ์

```javascript
// ตัวอย่างที่ 13: console.log - แสดงข้อมูลทั่วไป
console.log('ข้อความปกติ');
console.log(42);
console.log(true);
console.log([1, 2, 3]);
console.log({ name: 'สมชาย', age: 25 });

// แสดงหลายค่าในบรรทัดเดียว
console.log('ชื่อ:', 'สมชาย', 'อายุ:', 25);

// ตัวอย่างที่ 14: console.warn - แสดง warning (สีเหลือง)
console.warn('คำเตือน: ฟังก์ชันนี้จะถูกยกเลิกในเวอร์ชันถัดไป');
console.warn('ระดับน้ำมันต่ำ!');

// ตัวอย่างที่ 15: console.error - แสดง error (สีแดง)
console.error('เกิดข้อผิดพลาด!');
console.error('ไม่พบไฟล์:', 'data.json');

// ตัวอย่างที่ 16: console.info - ข้อมูล (สีน้ำเงิน)
console.info('แอปเวอร์ชัน 1.0.0');
console.info('กำลังเชื่อมต่อ database...');
```

```javascript
// ตัวอย่างที่ 17: console.table - แสดงข้อมูลเป็นตาราง
const students = [
    { name: 'สมชาย', grade: 'A', score: 95 },
    { name: 'สมหญิง', grade: 'B', score: 82 },
    { name: 'สมศักดิ์', grade: 'C', score: 70 }
];

console.table(students);
// แสดงเป็นตารางสวยงาม!

// แสดงเฉพาะบางคอลัมน์
console.table(students, ['name', 'grade']);
```

```javascript
// ตัวอย่างที่ 18: console.time และ console.timeEnd - วัดเวลา
console.time('การประมวลผล');

// โค้ดที่ต้องการวัดเวลา
let sum = 0;
for (let i = 0; i < 1000000; i++) {
    sum += i;
}

console.timeEnd('การประมวลผล');
// แสดง: การประมวลผล: 5.123ms

// ตัวอย่างที่ 19: console.count - นับจำนวนครั้งที่เรียก
function processItem(item) {
    console.count('processItem เรียกแล้ว');
    return item * 2;
}

processItem(1);  // processItem เรียกแล้ว: 1
processItem(2);  // processItem เรียกแล้ว: 2
processItem(3);  // processItem เรียกแล้ว: 3

// reset counter
console.countReset('processItem เรียกแล้ว');
```

```javascript
// ตัวอย่างที่ 20: console.group - จัดกลุ่ม log
console.group('ข้อมูลผู้ใช้');
console.log('ชื่อ: สมชาย');
console.log('อายุ: 25');
console.log('จังหวัด: กรุงเทพ');
console.groupEnd();

// Nested groups
console.group('ออเดอร์ #001');
console.group('รายการสินค้า');
console.log('- กาแฟ x2: 120 บาท');
console.log('- เค้ก x1: 80 บาท');
console.groupEnd();
console.log('รวม: 200 บาท');
console.groupEnd();
```

```javascript
// ตัวอย่างที่ 21: console.assert - แสดง error ถ้า condition เป็น false
const age = 15;
console.assert(age >= 18, 'ผู้ใช้ต้องมีอายุ 18 ปีขึ้นไป');
// แสดง error: Assertion failed: ผู้ใช้ต้องมีอายุ 18 ปีขึ้นไป

const x = 10;
console.assert(x > 5, 'x ต้องมากกว่า 5');
// ไม่แสดงอะไร เพราะ 10 > 5 เป็น true

// ตัวอย่างที่ 22: console.dir - แสดง object structure
const button = document.createElement('button');
console.dir(button);
// แสดง properties ทั้งหมดของ DOM element

// ตัวอย่างที่ 23: console.clear - ล้าง console
// console.clear(); // ล้าง console ทั้งหมด
```

```javascript
// ตัวอย่างที่ 24: Styling console output
console.log('%c JavaScript Course %c เริ่มแล้ว!', 
    'background: #222; color: #bada55; font-size: 20px; padding: 5px',
    'color: blue; font-size: 16px'
);

console.log('%cข้อความสีแดง', 'color: red; font-weight: bold;');
console.log('%cข้อความขนาดใหญ่', 'font-size: 24px; color: green;');
```

---

## Step 7: Comments (คอมเมนต์)

คอมเมนต์คือข้อความที่ JavaScript ไม่สนใจ ใช้สำหรับอธิบายโค้ด

```javascript
// ตัวอย่างที่ 25: Single-line comment (คอมเมนต์บรรทัดเดียว)
// นี่คือคอมเมนต์บรรทัดเดียว
// JavaScript จะไม่รันบรรทัดที่มี // นำหน้า

let x = 10; // คอมเมนต์ท้ายบรรทัดก็ได้
let y = 20; // y คือตัวแปรที่สอง

// ตัวอย่างที่ 26: Multi-line comment (คอมเมนต์หลายบรรทัด)
/*
    นี่คือคอมเมนต์หลายบรรทัด
    สามารถเขียนได้หลายบรรทัด
    มักใช้สำหรับอธิบาย function หรือ algorithm
*/

/*
 * รูปแบบที่นิยม
 * ใช้ * นำหน้าทุกบรรทัด
 * เพื่อให้อ่านง่ายขึ้น
 */
```

```javascript
// ตัวอย่างที่ 27: JSDoc comment - มาตรฐานการเขียน comment สำหรับ function
/**
 * คำนวณพื้นที่สี่เหลี่ยมผืนผ้า
 * 
 * @param {number} width - ความกว้าง (เมตร)
 * @param {number} height - ความสูง (เมตร)
 * @returns {number} พื้นที่ (ตารางเมตร)
 * @example
 * const area = calculateRectangle(5, 3);
 * console.log(area); // 15
 */
function calculateRectangle(width, height) {
    return width * height;
}

/**
 * แปลงอุณหภูมิจาก Celsius เป็น Fahrenheit
 * 
 * @param {number} celsius - อุณหภูมิในหน่วย Celsius
 * @returns {number} อุณหภูมิในหน่วย Fahrenheit
 */
function celsiusToFahrenheit(celsius) {
    return (celsius * 9/5) + 32;
}
```

```javascript
// ตัวอย่างที่ 28: Comment เพื่อ disable โค้ดชั่วคราว
let result = 10 + 20;
// let result = 10 * 20;  // ปิดชั่วคราว - กำลังทดสอบการบวก

// TODO: เพิ่มการตรวจสอบ input ภายหลัง
function divide(a, b) {
    // TODO: ตรวจสอบว่า b ไม่เท่ากับ 0
    return a / b;
}

// FIXME: bug ที่ต้องแก้ไข
function getUser(id) {
    // FIXME: ยังไม่ handle กรณี id เป็น null
    return users[id];
}

// HACK: วิธีแก้ปัญหาชั่วคราว
// HACK: ต้อง refactor เมื่อ API ใหม่พร้อม
const data = JSON.parse(JSON.stringify(originalData));
```

---

## Step 8: Strict Mode

Strict Mode ทำให้ JavaScript เข้มงวดมากขึ้น ช่วยป้องกัน bug ที่พบบ่อย

```javascript
// ตัวอย่างที่ 29: Strict Mode สำหรับไฟล์ทั้งหมด
'use strict';

// ตัวอย่างที่ 30: สิ่งที่ Strict Mode ป้องกัน
'use strict';

// 1. ห้ามใช้ตัวแปรที่ไม่ได้ประกาศ
// x = 10; // Error: x is not defined
let x = 10; // ต้องประกาศด้วย let, const หรือ var

// 2. ห้ามลบ property ที่ลบไม่ได้
// delete Object.prototype; // Error

// 3. ห้ามใช้ชื่อที่ reserved ไว้
// let let = 5;     // Error
// let static = 5;  // Error

// 4. ห้ามใช้ duplicate parameter
// function f(a, a) {} // Error

// 5. this ใน function ปกติเป็น undefined แทน global
function showThis() {
    console.log(this); // undefined (ใน strict mode)
}
showThis();
```

```javascript
// ตัวอย่างที่ 31: Strict Mode ใน function เดียว
function strictFunction() {
    'use strict';
    // โค้ดในนี้ใช้ strict mode
    let y = 20;
    console.log(y);
}

function normalFunction() {
    // โค้ดนี้ไม่ใช้ strict mode
    z = 30; // อนุญาตใน non-strict mode (ไม่แนะนำ!)
}

// ตัวอย่างที่ 32: ES Modules ใช้ strict mode อัตโนมัติ
// ไฟล์ที่ใช้ import/export จะเป็น strict mode เสมอ
// import { something } from './module.js'; // strict โดยอัตโนมัติ
```

---

## Step 9: ภาพรวมของตัวแปร

```javascript
// ตัวอย่างที่ 33: สามวิธีในการประกาศตัวแปร
var oldWay = 'แบบเก่า - ไม่แนะนำ';      // ES5
let newWay = 'แบบใหม่ - แนะนำ';          // ES6+
const constant = 'ค่าคงที่ - ไม่เปลี่ยน'; // ES6+

// ตัวอย่างที่ 34: ความแตกต่างพื้นฐาน
// let - เปลี่ยนค่าได้
let count = 0;
count = 1;      // OK
count = 2;      // OK

// const - เปลี่ยนค่าไม่ได้
const PI = 3.14159;
// PI = 3;      // Error! Assignment to constant variable

// ตัวอย่างที่ 35: การตั้งชื่อตัวแปร
// ถูกต้อง - camelCase (แนะนำ)
let firstName = 'สมชาย';
let lastName = 'ใจดี';
let totalPrice = 1500;
let isLoggedIn = true;

// ถูกต้อง - UPPER_CASE สำหรับ constants
const MAX_SIZE = 100;
const API_URL = 'https://api.example.com';

// ถูกต้อง - _underscore นำหน้า (private convention)
let _privateVar = 'internal value';

// ผิด
// let 123abc = 'ไม่ถูกต้อง'; // ขึ้นต้นด้วยตัวเลขไม่ได้
// let my-var = 'ไม่ถูกต้อง';  // มี - ไม่ได้
// let let = 'ไม่ถูกต้อง';     // keyword ไม่ได้
```

```javascript
// ตัวอย่างที่ 36: ประเภทข้อมูลเบื้องต้น
// Primitive types
let text = 'สวัสดี';         // String
let num = 42;                  // Number
let decimal = 3.14;           // Number (ทศนิยมก็เป็น Number)
let isTrue = true;            // Boolean
let nothing = null;           // Null
let notDefined;               // Undefined
let big = 9007199254740991n;  // BigInt
let sym = Symbol('unique');   // Symbol

// Object types
let arr = [1, 2, 3];          // Array
let obj = { name: 'สมชาย' }; // Object
let fn = function() {};       // Function

console.log(typeof text);     // "string"
console.log(typeof num);      // "number"
console.log(typeof isTrue);   // "boolean"
console.log(typeof nothing);  // "object" (quirk ของ JS!)
console.log(typeof notDefined); // "undefined"
console.log(typeof big);      // "bigint"
console.log(typeof sym);      // "symbol"
console.log(typeof arr);      // "object"
console.log(typeof obj);      // "object"
console.log(typeof fn);       // "function"
```

---

## Step 10: โปรแกรมตัวอย่างที่สมบูรณ์

```javascript
// ตัวอย่างที่ 37: โปรแกรมคำนวณเกรดนักเรียน
'use strict';

/**
 * คำนวณเกรดจากคะแนน
 * @param {number} score - คะแนน (0-100)
 * @returns {string} เกรด (A, B, C, D, F)
 */
function calculateGrade(score) {
    if (score >= 90) return 'A';
    if (score >= 80) return 'B';
    if (score >= 70) return 'C';
    if (score >= 60) return 'D';
    return 'F';
}

// ข้อมูลนักเรียน
const students = [
    { name: 'สมชาย ใจดี', score: 92 },
    { name: 'สมหญิง สวย', score: 78 },
    { name: 'สมศักดิ์ เก่ง', score: 65 },
    { name: 'สมปอง รวย', score: 55 }
];

// แสดงผลลัพธ์
console.log('=== ผลการเรียน ===');
console.table(students.map(s => ({
    ชื่อ: s.name,
    คะแนน: s.score,
    เกรด: calculateGrade(s.score)
})));

// หาคะแนนเฉลี่ย
const average = students.reduce((sum, s) => sum + s.score, 0) / students.length;
console.log(`\nคะแนนเฉลี่ย: ${average.toFixed(2)}`);
```

```javascript
// ตัวอย่างที่ 38: โปรแกรมแปลงสกุลเงิน
'use strict';

const EXCHANGE_RATES = {
    USD: 35.5,   // 1 USD = 35.5 THB
    EUR: 38.2,   // 1 EUR = 38.2 THB
    JPY: 0.24,   // 1 JPY = 0.24 THB
    GBP: 44.8    // 1 GBP = 44.8 THB
};

/**
 * แปลงเงินต่างประเทศเป็นบาทไทย
 * @param {number} amount - จำนวนเงิน
 * @param {string} currency - สกุลเงิน (USD, EUR, JPY, GBP)
 * @returns {number} จำนวนเงินบาทไทย
 */
function convertToTHB(amount, currency) {
    const rate = EXCHANGE_RATES[currency];
    if (!rate) {
        console.error(`ไม่รู้จักสกุลเงิน: ${currency}`);
        return null;
    }
    return amount * rate;
}

// ทดสอบ
console.log('=== แปลงสกุลเงิน ===');
console.log(`100 USD = ${convertToTHB(100, 'USD').toFixed(2)} บาท`);
console.log(`50 EUR = ${convertToTHB(50, 'EUR').toFixed(2)} บาท`);
console.log(`1000 JPY = ${convertToTHB(1000, 'JPY').toFixed(2)} บาท`);
console.log(`20 GBP = ${convertToTHB(20, 'GBP').toFixed(2)} บาท`);
```

```javascript
// ตัวอย่างที่ 39: โปรแกรม FizzBuzz (ปัญหา classic)
'use strict';

console.log('=== FizzBuzz 1-30 ===');
for (let i = 1; i <= 30; i++) {
    if (i % 15 === 0) {
        console.log(`${i}: FizzBuzz`);
    } else if (i % 3 === 0) {
        console.log(`${i}: Fizz`);
    } else if (i % 5 === 0) {
        console.log(`${i}: Buzz`);
    } else {
        console.log(i);
    }
}
```

```javascript
// ตัวอย่างที่ 40: โปรแกรมนับกลับ
'use strict';

function countdown(from) {
    console.log(`=== นับถอยหลังจาก ${from} ===`);
    
    for (let i = from; i >= 0; i--) {
        if (i === 0) {
            console.log('🚀 ปล่อยจรวด!');
        } else {
            console.log(i);
        }
    }
}

countdown(10);
```

```javascript
// ตัวอย่างที่ 41: โปรแกรมตรวจสอบเลขเฉพาะ
'use strict';

function isPrime(n) {
    if (n < 2) return false;
    if (n === 2) return true;
    if (n % 2 === 0) return false;
    
    for (let i = 3; i <= Math.sqrt(n); i += 2) {
        if (n % i === 0) return false;
    }
    return true;
}

console.log('=== เลขเฉพาะ 1-50 ===');
const primes = [];
for (let i = 2; i <= 50; i++) {
    if (isPrime(i)) primes.push(i);
}
console.log(primes.join(', '));
```

```javascript
// ตัวอย่างที่ 42: โปรแกรมแสดงปฏิทิน
'use strict';

function getMonthName(month) {
    const months = [
        'มกราคม', 'กุมภาพันธ์', 'มีนาคม', 'เมษายน',
        'พฤษภาคม', 'มิถุนายน', 'กรกฎาคม', 'สิงหาคม',
        'กันยายน', 'ตุลาคม', 'พฤศจิกายน', 'ธันวาคม'
    ];
    return months[month];
}

function getDaysInMonth(year, month) {
    return new Date(year, month + 1, 0).getDate();
}

const today = new Date();
const year = today.getFullYear();
const month = today.getMonth();

console.log(`=== ${getMonthName(month)} ${year} ===`);
console.log('อา จ  อ  พ  พฤ ศ  ส');

const firstDay = new Date(year, month, 1).getDay();
const daysInMonth = getDaysInMonth(year, month);

let calendar = '   '.repeat(firstDay);
for (let day = 1; day <= daysInMonth; day++) {
    calendar += String(day).padStart(2, ' ') + ' ';
    if ((firstDay + day) % 7 === 0) calendar += '\n';
}
console.log(calendar);
```

```javascript
// ตัวอย่างที่ 43: โปรแกรมเกม ทายตัวเลข (logic)
'use strict';

function numberGuessingGame() {
    const secret = Math.floor(Math.random() * 100) + 1;
    let attempts = 0;
    let found = false;
    
    // จำลองการเล่น (ในโปรแกรมจริงจะรับ input จากผู้ใช้)
    const guesses = [50, 75, 62, 56, 53, secret];
    
    console.log('=== เกมทายตัวเลข ===');
    console.log(`(เฉลย: ${secret})`);
    
    for (const guess of guesses) {
        attempts++;
        console.log(`\nทาย: ${guess}`);
        
        if (guess < secret) {
            console.log('น้อยไป! ลองใหม่');
        } else if (guess > secret) {
            console.log('มากไป! ลองใหม่');
        } else {
            console.log(`ถูกต้อง! ใช้ ${attempts} ครั้ง`);
            found = true;
            break;
        }
    }
    
    if (!found) console.log('หมดสิทธิ์แล้ว!');
}

numberGuessingGame();
```

```javascript
// ตัวอย่างที่ 44: โปรแกรมแสดงรูปดาว
'use strict';

function printTriangle(rows) {
    console.log('=== สามเหลี่ยม ===');
    for (let i = 1; i <= rows; i++) {
        console.log('*'.repeat(i));
    }
}

function printPyramid(rows) {
    console.log('\n=== ปิรามิด ===');
    for (let i = 1; i <= rows; i++) {
        const spaces = ' '.repeat(rows - i);
        const stars = '*'.repeat(2 * i - 1);
        console.log(spaces + stars);
    }
}

function printDiamond(rows) {
    console.log('\n=== เพชร ===');
    // ครึ่งบน
    for (let i = 1; i <= rows; i++) {
        const spaces = ' '.repeat(rows - i);
        const stars = '*'.repeat(2 * i - 1);
        console.log(spaces + stars);
    }
    // ครึ่งล่าง
    for (let i = rows - 1; i >= 1; i--) {
        const spaces = ' '.repeat(rows - i);
        const stars = '*'.repeat(2 * i - 1);
        console.log(spaces + stars);
    }
}

printTriangle(5);
printPyramid(5);
printDiamond(5);
```

```javascript
// ตัวอย่างที่ 45: โปรแกรมตรวจสอบ palindrome
'use strict';

function isPalindrome(str) {
    // ลบช่องว่างและแปลงเป็นตัวพิมพ์เล็ก
    const cleaned = str.toLowerCase().replace(/[^a-z0-9ก-๙]/g, '');
    const reversed = cleaned.split('').reverse().join('');
    return cleaned === reversed;
}

const words = ['racecar', 'hello', 'level', 'world', 'madam', 'javascript', 'ทอง', 'สวัสดี'];

console.log('=== ตรวจสอบ Palindrome ===');
words.forEach(word => {
    const result = isPalindrome(word);
    console.log(`"${word}": ${result ? 'ใช่ palindrome' : 'ไม่ใช่ palindrome'}`);
});
```

```javascript
// ตัวอย่างที่ 46: โปรแกรมแปลง Roman Numerals
'use strict';

function toRoman(num) {
    const values = [1000, 900, 500, 400, 100, 90, 50, 40, 10, 9, 5, 4, 1];
    const symbols = ['M', 'CM', 'D', 'CD', 'C', 'XC', 'L', 'XL', 'X', 'IX', 'V', 'IV', 'I'];
    
    let result = '';
    for (let i = 0; i < values.length; i++) {
        while (num >= values[i]) {
            result += symbols[i];
            num -= values[i];
        }
    }
    return result;
}

console.log('=== เลขโรมัน ===');
[1, 4, 9, 14, 40, 90, 400, 1994, 2024].forEach(n => {
    console.log(`${n} = ${toRoman(n)}`);
});
```

```javascript
// ตัวอย่างที่ 47: โปรแกรมคำนวณ Fibonacci
'use strict';

function fibonacci(n) {
    if (n <= 0) return [];
    if (n === 1) return [0];
    if (n === 2) return [0, 1];
    
    const seq = [0, 1];
    for (let i = 2; i < n; i++) {
        seq.push(seq[i-1] + seq[i-2]);
    }
    return seq;
}

console.log('=== ลำดับ Fibonacci 15 ตัวแรก ===');
console.log(fibonacci(15).join(', '));
// 0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144, 233, 377
```

```javascript
// ตัวอย่างที่ 48: โปรแกรม Temperature Converter
'use strict';

const temperatureConverter = {
    celsiusToFahrenheit(c) { return (c * 9/5) + 32; },
    celsiusToKelvin(c) { return c + 273.15; },
    fahrenheitToCelsius(f) { return (f - 32) * 5/9; },
    fahrenheitToKelvin(f) { return (f - 32) * 5/9 + 273.15; },
    kelvinToCelsius(k) { return k - 273.15; },
    kelvinToFahrenheit(k) { return (k - 273.15) * 9/5 + 32; }
};

const temps = [0, 100, -40, 37, 20];
console.log('=== แปลงอุณหภูมิ ===');
console.log('Celsius | Fahrenheit | Kelvin');
console.log('-'.repeat(35));
temps.forEach(c => {
    const f = temperatureConverter.celsiusToFahrenheit(c).toFixed(1);
    const k = temperatureConverter.celsiusToKelvin(c).toFixed(2);
    console.log(`${String(c).padStart(7)}°C | ${String(f).padStart(10)}°F | ${k}K`);
});
```

```javascript
// ตัวอย่างที่ 49: โปรแกรมจัดการ Todo List (logic เท่านั้น)
'use strict';

class TodoList {
    constructor() {
        this.todos = [];
        this.nextId = 1;
    }
    
    add(text) {
        this.todos.push({
            id: this.nextId++,
            text,
            done: false,
            createdAt: new Date().toLocaleString('th-TH')
        });
        console.log(`เพิ่ม: "${text}"`);
    }
    
    complete(id) {
        const todo = this.todos.find(t => t.id === id);
        if (todo) {
            todo.done = true;
            console.log(`เสร็จแล้ว: "${todo.text}"`);
        }
    }
    
    delete(id) {
        const index = this.todos.findIndex(t => t.id === id);
        if (index !== -1) {
            const removed = this.todos.splice(index, 1)[0];
            console.log(`ลบแล้ว: "${removed.text}"`);
        }
    }
    
    show() {
        console.log('\n=== รายการ Todo ===');
        if (this.todos.length === 0) {
            console.log('ไม่มีรายการ');
            return;
        }
        this.todos.forEach(t => {
            const status = t.done ? '✓' : '○';
            console.log(`[${status}] ${t.id}. ${t.text}`);
        });
    }
}

const todo = new TodoList();
todo.add('เรียน JavaScript');
todo.add('ทำโปรเจ็กต์');
todo.add('ออกกำลังกาย');
todo.complete(1);
todo.show();
todo.delete(2);
todo.show();
```

```javascript
// ตัวอย่างที่ 50: โปรแกรมสรุปสถิติ
'use strict';

function statistics(numbers) {
    if (numbers.length === 0) return null;
    
    const sorted = [...numbers].sort((a, b) => a - b);
    const sum = numbers.reduce((acc, n) => acc + n, 0);
    const mean = sum / numbers.length;
    
    // Median
    const mid = Math.floor(sorted.length / 2);
    const median = sorted.length % 2 !== 0
        ? sorted[mid]
        : (sorted[mid - 1] + sorted[mid]) / 2;
    
    // Mode
    const freq = {};
    numbers.forEach(n => freq[n] = (freq[n] || 0) + 1);
    const maxFreq = Math.max(...Object.values(freq));
    const mode = Object.keys(freq).filter(k => freq[k] === maxFreq).map(Number);
    
    // Variance และ Standard Deviation
    const variance = numbers.reduce((acc, n) => acc + Math.pow(n - mean, 2), 0) / numbers.length;
    const stdDev = Math.sqrt(variance);
    
    return { sum, mean, median, mode, min: sorted[0], max: sorted[sorted.length-1], variance, stdDev };
}

const scores = [85, 92, 78, 65, 92, 88, 74, 96, 82, 79];
const stats = statistics(scores);

console.log('=== สถิติคะแนน ===');
console.log('ข้อมูล:', scores.join(', '));
console.log(`ผลรวม: ${stats.sum}`);
console.log(`ค่าเฉลี่ย: ${stats.mean.toFixed(2)}`);
console.log(`ค่ามัธยฐาน: ${stats.median}`);
console.log(`ฐานนิยม: ${stats.mode.join(', ')}`);
console.log(`ต่ำสุด: ${stats.min}, สูงสุด: ${stats.max}`);
console.log(`ส่วนเบี่ยงเบนมาตรฐาน: ${stats.stdDev.toFixed(2)}`);
```

---

## สรุป Step 1-10

ในส่วนนี้คุณได้เรียนรู้:
1. JavaScript คืออะไรและทำอะไรได้บ้าง
2. ประวัติและวิวัฒนาการของ JavaScript
3. การติดตั้ง Node.js และ VS Code
4. การเขียนโปรแกรมแรก Hello World
5. วิธีต่างๆ ในการเขียน JavaScript (inline, script tag, external file)
6. Console methods ต่างๆ (log, warn, error, table, time, group, assert)
7. การเขียน Comments (single-line, multi-line, JSDoc)
8. Strict Mode และประโยชน์ของมัน
9. ภาพรวมของตัวแปรใน JavaScript
10. โปรแกรมตัวอย่างที่หลากหลาย

---

## แบบฝึกหัด

### ระดับพื้นฐาน

**แบบฝึกหัดที่ 1**: Hello World
สร้างไฟล์ JavaScript และแสดงข้อความ "สวัสดีจาก JavaScript!" ใน console

```javascript
// โค้ดเริ่มต้น
// TODO: เขียนโค้ดของคุณที่นี่
```

**แบบฝึกหัดที่ 2**: ข้อมูลส่วนตัว
สร้างตัวแปรเก็บข้อมูลส่วนตัว (ชื่อ, อายุ, จังหวัด) แล้วแสดงผลด้วย console.log

```javascript
// โค้ดเริ่มต้น
// TODO: สร้างตัวแปร name, age, province
// TODO: แสดงผลด้วย console.log
```

**แบบฝึกหัดที่ 3**: Console Methods
ทดลองใช้ console methods ทั้งหมดที่เรียนมา (log, warn, error, table, time)

### ระดับกลาง

**แบบฝึกหัดที่ 4**: เครื่องคิดเลขพื้นฐาน
สร้างโปรแกรมที่รับตัวเลข 2 ตัวและแสดงผลการบวก ลบ คูณ หาร

```javascript
'use strict';

const num1 = 10;
const num2 = 3;

// TODO: แสดงผล:
// 10 + 3 = 13
// 10 - 3 = 7
// 10 * 3 = 30
// 10 / 3 = 3.33...
// 10 % 3 = 1 (เศษจากการหาร)
// 10 ** 3 = 1000 (ยกกำลัง)
```

**แบบฝึกหัดที่ 5**: ตาราง 10
แสดงตารางสูตรคูณตั้งแต่ 1x1 ถึง 10x10

```javascript
'use strict';
// TODO: วนลูปแสดง:
// 1x1=1  1x2=2  ... 1x10=10
// 2x1=2  2x2=4  ... 2x10=20
// ...
// 10x1=10 ... 10x10=100
```

**แบบฝึกหัดที่ 6**: ตรวจสอบ palindrome
สร้างฟังก์ชันที่ตรวจสอบว่าคำที่รับมาเป็น palindrome หรือไม่

### ระดับสูง

**แบบฝึกหัดที่ 7**: สร้าง Timer
สร้างฟังก์ชันที่วัดเวลาการทำงานของฟังก์ชันอื่น (Function Timer)

```javascript
'use strict';

function timer(fn, ...args) {
    // TODO: วัดเวลาก่อนและหลังรัน fn
    // TODO: return { result, duration }
}

// ทดสอบ:
function slowCalc(n) {
    let sum = 0;
    for (let i = 0; i < n; i++) sum += i;
    return sum;
}

const { result, duration } = timer(slowCalc, 1000000);
console.log(`ผลลัพธ์: ${result}, ใช้เวลา: ${duration}ms`);
```

**แบบฝึกหัดที่ 8**: Debug โค้ดนี้
หาและแก้ไข bug ในโค้ดต่อไปนี้:

```javascript
// โค้ดที่มี bug - ลองหาและแก้ไข
function calculateArea(shape, ...dimensions) {
    if (shape = 'circle') {
        return Math.PI * dimensions[0] * dimensions[0];
    } else if (shape == 'rectangle') {
        return dimensions[0] + dimensions[1];  // bug!
    } else if (shape === 'triangle') {
        return 0.5 * dimension[0] * dimensions[1];  // bug!
    }
}

console.log(calculateArea('circle', 5));      // ควรได้ ~78.54
console.log(calculateArea('rectangle', 4, 6)); // ควรได้ 24
console.log(calculateArea('triangle', 3, 8)); // ควรได้ 12
```

---

## เฉลยแบบฝึกหัด

**เฉลยที่ 4**: เครื่องคิดเลข
```javascript
'use strict';

const num1 = 10;
const num2 = 3;

console.log(`${num1} + ${num2} = ${num1 + num2}`);
console.log(`${num1} - ${num2} = ${num1 - num2}`);
console.log(`${num1} * ${num2} = ${num1 * num2}`);
console.log(`${num1} / ${num2} = ${(num1 / num2).toFixed(2)}`);
console.log(`${num1} % ${num2} = ${num1 % num2}`);
console.log(`${num1} ** ${num2} = ${num1 ** num2}`);
```

**เฉลยที่ 8**: Debug
```javascript
'use strict';

function calculateArea(shape, ...dimensions) {
    if (shape === 'circle') {           // แก้: = เป็น ===
        return Math.PI * dimensions[0] * dimensions[0];
    } else if (shape === 'rectangle') { // แก้: == เป็น ===
        return dimensions[0] * dimensions[1]; // แก้: + เป็น *
    } else if (shape === 'triangle') {
        return 0.5 * dimensions[0] * dimensions[1]; // แก้: dimension เป็น dimensions
    }
}

console.log(calculateArea('circle', 5).toFixed(2));    // 78.54
console.log(calculateArea('rectangle', 4, 6));          // 24
console.log(calculateArea('triangle', 3, 8));           // 12
```

---

## แหล่งข้อมูลเพิ่มเติม

- **MDN Web Docs**: https://developer.mozilla.org/th/docs/Web/JavaScript
- **javascript.info**: https://javascript.info (มีภาษาไทยบางส่วน)
- **Node.js Documentation**: https://nodejs.org/docs/
- **VS Code Documentation**: https://code.visualstudio.com/docs

---

*ส่วนถัดไป: Part 2 - ตัวแปรและชนิดข้อมูล (Variables & Data Types)*
