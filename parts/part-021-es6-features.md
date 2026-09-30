# Part 21: ES6+ Features พื้นฐาน - let, const, Arrow Functions (Steps 391-410)

## บทนำ

ECMAScript 6 (ES6) หรือที่รู้จักกันในชื่อ ES2015 ถือเป็นการเปลี่ยนแปลงครั้งใหญ่ที่สุดของ JavaScript นับตั้งแต่เริ่มต้น ภาษานี้ได้รับการปรับปรุงครั้งใหญ่เพื่อรองรับการพัฒนาแอปพลิเคชันขนาดใหญ่ และทำให้โค้ดอ่านง่ายและบำรุงรักษาได้ดีขึ้น

ในบทนี้เราจะเรียนรู้ฟีเจอร์สำคัญของ ES6 ได้แก่ `let`, `const`, Arrow Functions และ Default Parameters พร้อมทั้งทำความเข้าใจความแตกต่างจาก JavaScript แบบเดิม

---

## Step 391: ประวัติของ ES6 และความสำคัญ

### ประวัติ ECMAScript

JavaScript ถูกสร้างขึ้นในปี 1995 โดย Brendan Eich ที่ Netscape ชื่อ ECMAScript มาจาก ECMA International ซึ่งเป็นองค์กรที่ทำหน้าที่กำหนดมาตรฐาน

**ไทม์ไลน์สำคัญ:**

| เวอร์ชัน | ปี | ความสำคัญ |
|---------|-----|-----------|
| ES1 | 1997 | มาตรฐานแรก |
| ES2 | 1998 | แก้ไขเล็กน้อย |
| ES3 | 1999 | RegExp, try/catch |
| ES4 | ยกเลิก | ซับซ้อนเกินไป |
| ES5 | 2009 | strict mode, JSON, Array methods |
| **ES6/ES2015** | **2015** | **การปฏิวัติครั้งใหญ่** |
| ES2016 | 2016 | Array.includes, ** operator |
| ES2017 | 2017 | async/await, Object.entries |
| ES2018 | 2018 | Rest/Spread for objects |
| ES2019 | 2019 | Array.flat, Object.fromEntries |
| ES2020 | 2020 | Optional chaining, Nullish coalescing |
| ES2021 | 2021 | String.replaceAll, Logical assignment |
| ES2022 | 2022 | Top-level await, Array.at |
| ES2023 | 2023 | Array.findLast, Hashbang |

```javascript
// ตัวอย่าง: วิวัฒนาการของ JavaScript
// แบบเก่า (ES5)
var greeting = function(name) {
  return 'Hello, ' + name + '!';
};

// แบบใหม่ (ES6+)
const greeting = (name) => `Hello, ${name}!`;

console.log(greeting('World')); // Hello, World!
```

---

## Step 392: var vs let vs const - การเปรียบเทียบโดยละเอียด

### ปัญหาของ var

`var` มีพฤติกรรมที่อาจทำให้เกิดบั๊กได้ง่าย:

```javascript
// ปัญหา 1: var ถูก redeclare ได้
var x = 1;
var x = 2; // ไม่เกิด error!
console.log(x); // 2

// ปัญหา 2: var มี function scope ไม่ใช่ block scope
if (true) {
  var blockVar = 'ฉันอยู่ใน if block';
}
console.log(blockVar); // 'ฉันอยู่ใน if block' - รั่วออกมาข้างนอก!

// ปัญหา 3: var ถูก hoist ขึ้นไปด้านบน
console.log(hoisted); // undefined (ไม่ใช่ error!)
var hoisted = 'ค่า';
```

### let - ตัวแปรที่เปลี่ยนค่าได้

```javascript
// let มี block scope
if (true) {
  let blockLet = 'ฉันอยู่ใน block';
  console.log(blockLet); // 'ฉันอยู่ใน block'
}
// console.log(blockLet); // ReferenceError: blockLet is not defined

// let ไม่สามารถ redeclare ในขอบเขตเดียวกัน
let y = 1;
// let y = 2; // SyntaxError: Identifier 'y' has already been declared

// แต่ reassign ได้
y = 2;
console.log(y); // 2

// let ใน loop - แต่ละรอบมีขอบเขตของตัวเอง
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100);
}
// 0, 1, 2 (ถูกต้อง!)

// เปรียบเทียบกับ var ใน loop
for (var j = 0; j < 3; j++) {
  setTimeout(() => console.log(j), 100);
}
// 3, 3, 3 (ผิด! เพราะ var ไม่มี block scope)
```

### const - ค่าคงที่

```javascript
// const ต้องกำหนดค่าทันที
const PI = 3.14159;
// const EMPTY; // SyntaxError: Missing initializer in const declaration

// const ไม่สามารถ reassign
// PI = 3; // TypeError: Assignment to constant variable

// แต่สำหรับ object และ array - สามารถเปลี่ยน property ได้
const person = { name: 'สมชาย', age: 25 };
person.name = 'สมหญิง'; // ทำได้!
person.age = 26; // ทำได้!
console.log(person); // { name: 'สมหญิง', age: 26 }

// แต่ไม่สามารถ reassign ตัวแปรเอง
// person = { name: 'อื่น' }; // TypeError!

const numbers = [1, 2, 3];
numbers.push(4); // ทำได้!
console.log(numbers); // [1, 2, 3, 4]

// numbers = []; // TypeError!
```

### ตารางเปรียบเทียบ

```javascript
// สรุปความแตกต่าง
/*
| คุณสมบัติ          | var       | let       | const     |
|-------------------|-----------|-----------|-----------|
| Scope             | Function  | Block     | Block     |
| Hoisting          | Yes (undef)| Yes (TDZ) | Yes (TDZ) |
| Redeclaration     | Yes       | No        | No        |
| Reassignment      | Yes       | Yes       | No        |
| Global property   | Yes       | No        | No        |
*/

// ตัวอย่างแสดง Global property
var globalVar = 'ฉันเป็น global';
console.log(window.globalVar); // 'ฉันเป็น global' (ใน browser)

let globalLet = 'ฉันเป็น let global';
console.log(window.globalLet); // undefined (ไม่กลายเป็น property)
```

---

## Step 393: Block Scope vs Function Scope vs Global Scope

```javascript
// Global Scope
const globalConst = 'ฉันอยู่ทุกที่';

function outerFunction() {
  // Function Scope
  const functionVar = 'ฉันอยู่ใน function นี้';
  
  if (true) {
    // Block Scope
    const blockVar = 'ฉันอยู่ใน block นี้';
    console.log(globalConst);  // ✓ เข้าถึงได้
    console.log(functionVar);  // ✓ เข้าถึงได้
    console.log(blockVar);     // ✓ เข้าถึงได้
  }
  
  console.log(globalConst);  // ✓ เข้าถึงได้
  console.log(functionVar);  // ✓ เข้าถึงได้
  // console.log(blockVar);  // ✗ ReferenceError!
}

outerFunction();
console.log(globalConst);  // ✓ เข้าถึงได้
// console.log(functionVar); // ✗ ReferenceError!
```

```javascript
// Scope Chain - การค้นหาตัวแปรจากใกล้ไปไกล
const x = 'global';

function outer() {
  const x = 'outer';
  
  function inner() {
    const x = 'inner';
    console.log(x); // 'inner' - ใช้ค่าใกล้ที่สุด
  }
  
  function innerNoX() {
    console.log(x); // 'outer' - ค้นหาขึ้นไปชั้นนอก
  }
  
  inner();      // 'inner'
  innerNoX();   // 'outer'
}

outer();
console.log(x); // 'global'
```

```javascript
// Block scope ใน switch
switch (true) {
  case true: {
    // ใช้ {} เพื่อสร้าง block scope
    const switchVar = 'อยู่ใน case block';
    console.log(switchVar);
    break;
  }
  case false: {
    const switchVar = 'อยู่ใน case อื่น'; // ไม่ conflict!
    console.log(switchVar);
    break;
  }
}
```

---

## Step 394: Temporal Dead Zone (TDZ)

TDZ คือช่วงเวลาตั้งแต่เริ่ม block จนถึงบรรทัดที่ประกาศตัวแปร ในช่วงนี้ตัวแปรยังไม่สามารถใช้งานได้

```javascript
// TDZ กับ let
{
  // เริ่ม TDZ สำหรับ myLet
  console.log(typeof myLet); // ReferenceError! (ใน TDZ)
  // ↑ ต่างจาก var ที่จะได้ undefined
  
  let myLet = 'ค่าของฉัน'; // TDZ สิ้นสุดที่นี่
  console.log(myLet); // 'ค่าของฉัน'
}

// TDZ กับ const
{
  // console.log(myConst); // ReferenceError!
  const myConst = 42;
  console.log(myConst); // 42
}
```

```javascript
// ตัวอย่างที่น่าสนใจ: TDZ กับ typeof
console.log(typeof undeclaredVar); // 'undefined' (ปลอดภัย)
// console.log(typeof myLet);       // ReferenceError! (ถ้า myLet อยู่ใน TDZ)

// TDZ ใน class
class MyClass {
  constructor() {
    // this.value = this.method(); // อาจเกิดปัญหา
  }
  
  method() {
    return 42;
  }
}
```

```javascript
// TDZ กับ function parameters
function example(a = b, b = 1) {
  // a = b: ตอนประเมิน b ยังอยู่ใน TDZ!
  return a + b;
}

// example(); // ReferenceError!

function correct(a = 1, b = a) {
  return a + b;
}

console.log(correct()); // 2 (a=1, b=1)
console.log(correct(5)); // 10 (a=5, b=5)
```

---

## Step 395: Hoisting - พฤติกรรมการยกตัวแปร

```javascript
// var hoisting - ยกตัวแปรขึ้นไปด้านบน พร้อมค่า undefined
console.log(varHoisted); // undefined (ไม่ใช่ error)
var varHoisted = 'ค่า';
console.log(varHoisted); // 'ค่า'

// JavaScript แปล var เป็น:
var varHoisted; // ถูก hoist ขึ้นมา
console.log(varHoisted); // undefined
varHoisted = 'ค่า'; // ค่าที่แท้จริงอยู่ที่นี่
```

```javascript
// let และ const hoisting - ยกขึ้นไปแต่ไม่กำหนดค่า (TDZ)
// console.log(letHoisted); // ReferenceError: Cannot access before init
let letHoisted = 'ค่า';

// function declaration hoisting - ยกทั้ง function ขึ้นไป
console.log(hoistedFunc()); // 'ฉัน hoist ได้!'
function hoistedFunc() {
  return 'ฉัน hoist ได้!';
}

// function expression - ไม่ hoist เหมือน function declaration
// console.log(notHoisted()); // TypeError: notHoisted is not a function
var notHoisted = function() {
  return 'ฉัน hoist ไม่ได้';
};
```

```javascript
// class hoisting - คล้ายกับ let
// const obj = new MyClass(); // ReferenceError: Cannot access before init
class MyClass {
  constructor() {
    this.value = 42;
  }
}
const obj = new MyClass();
console.log(obj.value); // 42
```

---

## Step 396: Arrow Functions - รูปแบบต่างๆ

### รูปแบบ Arrow Function

```javascript
// 1. ไม่มี parameter
const greet = () => 'สวัสดี!';
console.log(greet()); // 'สวัสดี!'

// 2. มี parameter เดียว (ไม่ต้องใช้วงเล็บ)
const double = x => x * 2;
console.log(double(5)); // 10

// หรือใช้วงเล็บก็ได้
const triple = (x) => x * 3;
console.log(triple(5)); // 15

// 3. หลาย parameter (ต้องใช้วงเล็บ)
const add = (a, b) => a + b;
console.log(add(3, 4)); // 7

// 4. Expression body (return โดยนัย)
const square = x => x * x;
console.log(square(4)); // 16

// 5. Block body (ต้องมี return ชัดเจน)
const complexCalc = (a, b) => {
  const sum = a + b;
  const product = a * b;
  return { sum, product };
};
console.log(complexCalc(3, 4)); // { sum: 7, product: 12 }
```

```javascript
// 6. Return object literal (ต้องใส่ () รอบ object)
const makeObject = (name, age) => ({ name, age });
console.log(makeObject('สมชาย', 25)); // { name: 'สมชาย', age: 25 }

// ถ้าไม่ใส่ () - JavaScript จะคิดว่า {} เป็น block
const wrongWay = (name) => { name }; // ไม่ return อะไร!
console.log(wrongWay('test')); // undefined

// 7. Default parameters
const greetUser = (name = 'ผู้ใช้') => `สวัสดี, ${name}!`;
console.log(greetUser());        // 'สวัสดี, ผู้ใช้!'
console.log(greetUser('สมชาย')); // 'สวัสดี, สมชาย!'

// 8. Rest parameters
const sum = (...nums) => nums.reduce((acc, n) => acc + n, 0);
console.log(sum(1, 2, 3, 4, 5)); // 15

// 9. Destructuring parameter
const getFullName = ({ firstName, lastName }) => `${firstName} ${lastName}`;
console.log(getFullName({ firstName: 'สมชาย', lastName: 'ใจดี' }));
// 'สมชาย ใจดี'
```

---

## Step 397: Arrow Functions vs Regular Functions - ความแตกต่างของ this

นี่คือความแตกต่างที่สำคัญที่สุดระหว่าง Arrow Functions และ Regular Functions

```javascript
// Regular function - this ขึ้นอยู่กับว่าถูกเรียกอย่างไร
const person1 = {
  name: 'สมชาย',
  greet: function() {
    console.log(`สวัสดี ฉันชื่อ ${this.name}`);
  }
};

person1.greet(); // 'สวัสดี ฉันชื่อ สมชาย'

// Arrow function - this ถูกกำหนดจาก lexical scope (ที่ประกาศ)
const person2 = {
  name: 'สมหญิง',
  greet: () => {
    console.log(`สวัสดี ฉันชื่อ ${this.name}`);
    // this ที่นี่ไม่ใช่ person2!
    // this คือ outer scope (global หรือ undefined ใน strict mode)
  }
};

person2.greet(); // 'สวัสดี ฉันชื่อ undefined'
```

```javascript
// ปัญหาใน setTimeout กับ Regular Function
const timer1 = {
  name: 'Timer1',
  start: function() {
    setTimeout(function() {
      console.log(`${this.name} เสร็จแล้ว`); // this ไม่ใช่ timer1!
    }, 1000);
  }
};

// วิธีแก้แบบเก่า: เก็บ this ไว้
const timer2 = {
  name: 'Timer2',
  start: function() {
    const self = this; // เก็บ reference
    setTimeout(function() {
      console.log(`${self.name} เสร็จแล้ว`); // ใช้ self แทน this
    }, 1000);
  }
};

// วิธีแก้แบบใหม่: ใช้ Arrow Function
const timer3 = {
  name: 'Timer3',
  start: function() {
    setTimeout(() => {
      console.log(`${this.name} เสร็จแล้ว`); // this คือ timer3!
    }, 1000);
  }
};

timer3.start(); // 'Timer3 เสร็จแล้ว'
```

```javascript
// this ใน class methods
class Counter {
  constructor() {
    this.count = 0;
  }
  
  // Regular method - this ถูกต้องเมื่อเรียกผ่าน object
  incrementRegular() {
    this.count++;
    return this.count;
  }
  
  // Arrow property - this ถูกผูกตอน constructor
  incrementArrow = () => {
    this.count++;
    return this.count;
  }
  
  startAutoCount() {
    // ปัญหา: regular function ใน callback
    // setInterval(function() { this.count++; }, 1000); // this ผิด!
    
    // วิธีแก้: ใช้ arrow function
    setInterval(() => {
      this.count++; // this ถูกต้อง!
    }, 1000);
  }
}

const counter = new Counter();
console.log(counter.incrementRegular()); // 1

// ปัญหาเมื่อ destructure
const { incrementRegular, incrementArrow } = counter;
// incrementRegular(); // TypeError: Cannot read 'count' of undefined
console.log(incrementArrow()); // 2 (ทำงานได้!)
```

---

## Step 398: Arrow Functions กับ arguments Object

```javascript
// Regular function มี arguments object
function regularSum() {
  let total = 0;
  for (let i = 0; i < arguments.length; i++) {
    total += arguments[i];
  }
  return total;
}
console.log(regularSum(1, 2, 3, 4, 5)); // 15

// Arrow function ไม่มี arguments object!
const arrowSum = () => {
  // console.log(arguments); // ReferenceError ใน strict mode
  // หรือ arguments จาก outer scope (ใน non-strict)
};

// วิธีที่ถูกต้องสำหรับ arrow function: ใช้ rest parameters
const arrowSumCorrect = (...nums) => {
  return nums.reduce((acc, n) => acc + n, 0);
};
console.log(arrowSumCorrect(1, 2, 3, 4, 5)); // 15
```

```javascript
// arguments ใน nested functions
function outer() {
  console.log(arguments[0]); // 'outer arg'
  
  // Arrow function ใน regular function - ใช้ outer's arguments
  const inner = () => {
    console.log(arguments[0]); // 'outer arg' (จาก outer!)
  };
  inner();
  
  // Regular function - มี arguments ของตัวเอง
  function innerRegular() {
    console.log(arguments[0]); // 'inner arg'
  }
  innerRegular('inner arg');
}

outer('outer arg');
```

---

## Step 399: Arrow Functions เป็น Callbacks

```javascript
// Arrow functions ทำให้โค้ดสั้นลงมากเมื่อใช้เป็น callbacks

const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

// แบบเก่า
const evensOld = numbers.filter(function(n) {
  return n % 2 === 0;
});

// แบบใหม่
const evens = numbers.filter(n => n % 2 === 0);
console.log(evens); // [2, 4, 6, 8, 10]

// map
const doubled = numbers.map(n => n * 2);
console.log(doubled); // [2, 4, 6, 8, 10, 12, 14, 16, 18, 20]

// reduce
const sum = numbers.reduce((acc, n) => acc + n, 0);
console.log(sum); // 55

// chaining
const result = numbers
  .filter(n => n % 2 === 0)
  .map(n => n * n)
  .reduce((acc, n) => acc + n, 0);
console.log(result); // 220 (4+16+36+64+100)
```

```javascript
// Arrow functions กับ promise chains
const fetchData = () =>
  Promise.resolve({ data: [1, 2, 3] })
    .then(response => response.data)
    .then(data => data.map(n => n * 2))
    .then(doubled => doubled.filter(n => n > 2))
    .catch(error => console.error(error));

fetchData().then(console.log); // [4, 6]

// เปรียบเทียบกับ async/await
const fetchDataAsync = async () => {
  const response = await Promise.resolve({ data: [1, 2, 3] });
  const doubled = response.data.map(n => n * 2);
  return doubled.filter(n => n > 2);
};

fetchDataAsync().then(console.log); // [4, 6]
```

```javascript
// Arrow functions กับ event handlers (ข้อควรระวัง)
class Button {
  constructor(label) {
    this.label = label;
    this.clickCount = 0;
  }
  
  // ดี: arrow function method
  handleClick = () => {
    this.clickCount++;
    console.log(`${this.label} ถูกคลิก ${this.clickCount} ครั้ง`);
  }
  
  // ไม่แนะนำสำหรับ DOM event: สร้างฟังก์ชันใหม่ทุกครั้ง
  // addEventListener('click', () => this.handleClick());
}

const btn = new Button('ปุ่มส่ง');
btn.handleClick(); // 'ปุ่มส่ง ถูกคลิก 1 ครั้ง'
btn.handleClick(); // 'ปุ่มส่ง ถูกคลิก 2 ครั้ง'
```

---

## Step 400: เมื่อไหร่ควรใช้ Arrow Function และเมื่อไหร่ไม่ควร

```javascript
// ✓ ควรใช้: callbacks ใน array methods
const prices = [100, 200, 300, 400, 500];
const discounted = prices.map(price => price * 0.9);

// ✓ ควรใช้: callbacks ที่ต้องการ lexical this
class Timer {
  constructor() {
    this.seconds = 0;
  }
  start() {
    setInterval(() => this.seconds++, 1000); // ✓
  }
}

// ✗ ไม่ควรใช้: methods ใน object literal (ถ้าต้องการ this)
const obj = {
  value: 42,
  getValue: () => this.value, // this ไม่ใช่ obj!
};
console.log(obj.getValue()); // undefined

// ✓ ควรใช้ regular function สำหรับ object method
const obj2 = {
  value: 42,
  getValue() { return this.value; }, // ES6 shorthand
};
console.log(obj2.getValue()); // 42

// ✗ ไม่ควรใช้: constructor function
// const MyClass = () => {}; // Arrow function ไม่สามารถเป็น constructor!
// new MyClass(); // TypeError: MyClass is not a constructor

// ✗ ไม่ควรใช้: generator function
// const gen = *() => {}; // SyntaxError!
// ใช้ regular function แทน:
function* generator() { yield 1; yield 2; }
```

---

## Step 401: Default Parameter Values

```javascript
// รูปแบบพื้นฐาน
function greet(name = 'ผู้ใช้', greeting = 'สวัสดี') {
  return `${greeting}, ${name}!`;
}

console.log(greet());                    // 'สวัสดี, ผู้ใช้!'
console.log(greet('สมชาย'));             // 'สวัสดี, สมชาย!'
console.log(greet('สมชาย', 'ดีครับ'));  // 'ดีครับ, สมชาย!'
console.log(greet(undefined, 'หวัดดี')); // 'หวัดดี, ผู้ใช้!' (undefined ใช้ default)
console.log(greet(null, 'หวัดดี'));      // 'หวัดดี, null!' (null ไม่ใช้ default)
```

```javascript
// Default parameters กับ undefined vs null
function test(value = 'default') {
  return value;
}

console.log(test());          // 'default'
console.log(test(undefined)); // 'default' (undefined triggers default)
console.log(test(null));      // null (null ไม่ใช่ undefined)
console.log(test(0));         // 0
console.log(test(''));        // ''
console.log(test(false));     // false
```

---

## Step 402: Default Parameters with Expressions

```javascript
// Default เป็น expression - ถูกประเมินทุกครั้งที่เรียกใช้
function getTimestamp(date = new Date()) {
  return date.toISOString();
}

console.log(getTimestamp()); // เวลาปัจจุบัน
// รอ 1 วินาที
console.log(getTimestamp()); // เวลาใหม่ (ไม่ใช่เวลาเดิม!)

// Default เป็น function call
function generateId() {
  return Math.random().toString(36).substr(2, 9);
}

function createUser(name, id = generateId()) {
  return { id, name };
}

console.log(createUser('สมชาย')); // { id: 'abc123xyz', name: 'สมชาย' }
console.log(createUser('สมหญิง')); // { id: 'def456uvw', name: 'สมหญิง' }
```

```javascript
// Default expressions ถูกประเมิน lazily (เฉพาะเมื่อต้องการ)
let callCount = 0;
function expensive() {
  callCount++;
  return 'expensive result';
}

function example(param = expensive()) {
  return param;
}

console.log(example('provided')); // 'provided' - expensive() ไม่ถูกเรียก!
console.log(callCount); // 0

console.log(example()); // 'expensive result' - expensive() ถูกเรียก
console.log(callCount); // 1
```

---

## Step 403: Default Parameters ใช้ Parameter อื่น

```javascript
// Parameter สามารถใช้ parameter ก่อนหน้าเป็น default ได้
function createRect(width, height = width) {
  return { width, height, area: width * height };
}

console.log(createRect(5));    // { width: 5, height: 5, area: 25 }
console.log(createRect(5, 3)); // { width: 5, height: 3, area: 15 }

// ตัวอย่างที่ซับซ้อนขึ้น
function range(start, end = start + 10, step = 1) {
  const result = [];
  for (let i = start; i < end; i += step) {
    result.push(i);
  }
  return result;
}

console.log(range(0));        // [0, 1, 2, ..., 9]
console.log(range(0, 5));     // [0, 1, 2, 3, 4]
console.log(range(0, 10, 2)); // [0, 2, 4, 6, 8]
```

```javascript
// ใช้ destructuring กับ default parameters
function setupConfig({
  host = 'localhost',
  port = 3000,
  protocol = 'http',
  debug = false
} = {}) {
  return `${protocol}://${host}:${port} (debug: ${debug})`;
}

console.log(setupConfig());
// 'http://localhost:3000 (debug: false)'

console.log(setupConfig({ port: 8080, debug: true }));
// 'http://localhost:8080 (debug: true)'

console.log(setupConfig({ host: 'example.com', protocol: 'https' }));
// 'https://example.com:3000 (debug: false)'
```

---

## Step 404: Short-circuit Evaluation กับ Default Values

```javascript
// Short-circuit ด้วย || (OR)
function greetOld(name) {
  name = name || 'ผู้ใช้'; // ถ้า name เป็น falsy ใช้ 'ผู้ใช้'
  return `สวัสดี, ${name}!`;
}

console.log(greetOld('สมชาย')); // 'สวัสดี, สมชาย!'
console.log(greetOld(''));      // 'สวัสดี, ผู้ใช้!' ('' เป็น falsy!)
console.log(greetOld(0));       // 'สวัสดี, ผู้ใช้!' (0 เป็น falsy!)

// ES2020: Nullish Coalescing (??) - ดีกว่า
function greetNew(name) {
  name = name ?? 'ผู้ใช้'; // เฉพาะ null หรือ undefined
  return `สวัสดี, ${name}!`;
}

console.log(greetNew(''));  // 'สวัสดี, !' ('' ไม่ถูกแทนที่!)
console.log(greetNew(0));   // 'สวัสดี, 0!' (0 ไม่ถูกแทนที่!)
console.log(greetNew(null)); // 'สวัสดี, ผู้ใช้!'
console.log(greetNew(undefined)); // 'สวัสดี, ผู้ใช้!'
```

```javascript
// Short-circuit กับ && (AND)
const user = { name: 'สมชาย', profile: { age: 25 } };

// แบบเก่า
const age = user && user.profile && user.profile.age;

// ES2020: Optional Chaining (?.)
const ageSafe = user?.profile?.age;
console.log(ageSafe); // 25

const noUser = null;
const safeAge = noUser?.profile?.age; // ไม่ throw error
console.log(safeAge); // undefined
```

---

## Step 405: ES2016 Features

```javascript
// ES2016 (ES7) - มีแค่ 2 features หลัก

// 1. Array.prototype.includes
const fruits = ['แอปเปิล', 'กล้วย', 'ส้ม'];

// แบบเก่า
console.log(fruits.indexOf('กล้วย') !== -1); // true

// แบบใหม่
console.log(fruits.includes('กล้วย')); // true
console.log(fruits.includes('มะม่วง')); // false

// includes กับ NaN
const withNaN = [1, 2, NaN, 4];
console.log(withNaN.includes(NaN));     // true
console.log(withNaN.indexOf(NaN) > -1); // false (indexOf ไม่รู้จัก NaN)

// 2. Exponentiation Operator (**)
console.log(2 ** 10); // 1024
console.log(Math.pow(2, 10)); // 1024 (แบบเก่า)

console.log(3 ** 3); // 27
console.log((-2) ** 3); // -8

// Exponentiation assignment
let base = 2;
base **= 8;
console.log(base); // 256
```

---

## Step 406: ES2017 Features

```javascript
// 1. async/await (จะเรียนละเอียดในบทหลัง)
async function fetchUser(id) {
  const response = await fetch(`/api/users/${id}`);
  const user = await response.json();
  return user;
}

// 2. Object.entries() - ได้ [key, value] pairs
const person = { name: 'สมชาย', age: 25, city: 'กรุงเทพ' };

const entries = Object.entries(person);
console.log(entries);
// [['name', 'สมชาย'], ['age', 25], ['city', 'กรุงเทพ']]

for (const [key, value] of Object.entries(person)) {
  console.log(`${key}: ${value}`);
}

// 3. Object.values()
const values = Object.values(person);
console.log(values); // ['สมชาย', 25, 'กรุงเทพ']

// 4. String padding
console.log('5'.padStart(3, '0'));  // '005'
console.log('5'.padEnd(3, '0'));    // '500'
console.log('42'.padStart(5));      // '   42' (spaces)

// ใช้งานจริง: format เวลา
const hours = 9;
const minutes = 5;
const time = `${String(hours).padStart(2, '0')}:${String(minutes).padStart(2, '0')}`;
console.log(time); // '09:05'

// 5. Trailing commas in function parameters
function trailingComma(
  param1,
  param2,
  param3, // trailing comma ได้แล้วใน ES2017
) {
  return param1 + param2 + param3;
}

// 6. Object.getOwnPropertyDescriptors()
const desc = Object.getOwnPropertyDescriptors(person);
console.log(desc.name); // { value: 'สมชาย', writable: true, enumerable: true, configurable: true }
```

---

## Step 407: ES2018 และ ES2019 Features

```javascript
// ES2018

// 1. Rest/Spread for Objects (จะเรียนละเอียดในบท Spread/Rest)
const { name, ...rest } = { name: 'สมชาย', age: 25, city: 'กรุงเทพ' };
console.log(name); // 'สมชาย'
console.log(rest); // { age: 25, city: 'กรุงเทพ' }

// 2. Async iteration
async function* asyncGenerator() {
  yield 1;
  yield 2;
  yield 3;
}

(async () => {
  for await (const value of asyncGenerator()) {
    console.log(value); // 1, 2, 3
  }
})();

// 3. Promise.finally()
fetch('/api/data')
  .then(data => processData(data))
  .catch(err => handleError(err))
  .finally(() => hideLoadingSpinner()); // ทำงานเสมอ

// ES2019

// 1. Array.flat()
const nested = [1, [2, 3], [4, [5, 6]]];
console.log(nested.flat());    // [1, 2, 3, 4, [5, 6]]
console.log(nested.flat(2));   // [1, 2, 3, 4, 5, 6]
console.log(nested.flat(Infinity)); // [1, 2, 3, 4, 5, 6]

// 2. Array.flatMap()
const sentences = ['สวัสดี โลก', 'Hello World'];
const words = sentences.flatMap(s => s.split(' '));
console.log(words); // ['สวัสดี', 'โลก', 'Hello', 'World']

// 3. Object.fromEntries()
const entries = [['a', 1], ['b', 2], ['c', 3]];
const obj = Object.fromEntries(entries);
console.log(obj); // { a: 1, b: 2, c: 3 }

// ใช้ร่วมกับ map ใน Map object
const map = new Map([['x', 10], ['y', 20]]);
const fromMap = Object.fromEntries(map);
console.log(fromMap); // { x: 10, y: 20 }

// 4. String.trimStart() / String.trimEnd()
const str = '   hello   ';
console.log(str.trimStart()); // 'hello   '
console.log(str.trimEnd());   // '   hello'
console.log(str.trim());      // 'hello'

// 5. Optional catch binding
try {
  JSON.parse('invalid json');
} catch { // ไม่ต้องมี (error) ถ้าไม่ใช้
  console.log('Parse failed');
}
```

---

## Step 408: ES2020 Features

```javascript
// 1. Optional Chaining (?.)
const user = {
  name: 'สมชาย',
  address: {
    street: '123 ถนนสุขุมวิท',
    city: 'กรุงเทพ'
  }
};

console.log(user?.address?.city);     // 'กรุงเทพ'
console.log(user?.phone?.number);     // undefined (ไม่ throw error)
console.log(user?.greet?.());         // undefined (method ไม่มี)

// กับ array
const arr = [1, 2, 3];
console.log(arr?.[0]);  // 1
const nullArr = null;
console.log(nullArr?.[0]); // undefined

// 2. Nullish Coalescing (??)
const value1 = null ?? 'default';    // 'default'
const value2 = undefined ?? 'default'; // 'default'
const value3 = 0 ?? 'default';       // 0 (0 ไม่ใช่ null/undefined)
const value4 = '' ?? 'default';      // '' ('' ไม่ใช่ null/undefined)
const value5 = false ?? 'default';   // false

console.log(value1, value2, value3, value4, value5);

// 3. BigInt
const bigNumber = 9007199254740991n; // Number.MAX_SAFE_INTEGER
console.log(bigNumber + 1n); // 9007199254740992n
console.log(typeof bigNumber); // 'bigint'

// 4. Promise.allSettled()
const promises = [
  Promise.resolve('success1'),
  Promise.reject('error1'),
  Promise.resolve('success2'),
];

Promise.allSettled(promises).then(results => {
  results.forEach(result => {
    if (result.status === 'fulfilled') {
      console.log('สำเร็จ:', result.value);
    } else {
      console.log('ล้มเหลว:', result.reason);
    }
  });
});

// 5. String.matchAll()
const text = 'cat bat sat';
const regex = /[a-z]at/g;
const matches = [...text.matchAll(regex)];
console.log(matches.length); // 3
```

---

## Step 409: ES2021, ES2022, ES2023 Features

```javascript
// ES2021

// 1. String.replaceAll()
const str = 'สวัสดี โลก โลก โลก';
console.log(str.replace('โลก', 'World')); // replaces only first
console.log(str.replaceAll('โลก', 'World')); // replaces all

// 2. Logical Assignment Operators
let a = null;
a ??= 'default'; // a = a ?? 'default'
console.log(a); // 'default'

let b = 0;
b ||= 42; // b = b || 42
console.log(b); // 42

let c = 5;
c &&= c * 2; // c = c && c * 2
console.log(c); // 10

// 3. Numeric separators
const million = 1_000_000;
const hex = 0xFF_EC_D0_12;
const bytes = 0b1010_0001;
console.log(million); // 1000000

// 4. Promise.any()
const p1 = Promise.reject('error 1');
const p2 = Promise.resolve('success');
const p3 = Promise.resolve('also success');

Promise.any([p1, p2, p3]).then(value => {
  console.log(value); // 'success' (first fulfilled)
});

// ES2022

// 1. Array.at() - รองรับ negative index
const arr = [1, 2, 3, 4, 5];
console.log(arr.at(0));   // 1
console.log(arr.at(-1));  // 5 (ตัวสุดท้าย)
console.log(arr.at(-2));  // 4

// 2. Object.hasOwn() - แทน hasOwnProperty
const obj = { name: 'test' };
console.log(Object.hasOwn(obj, 'name'));    // true
console.log(Object.hasOwn(obj, 'toString')); // false

// 3. Class Fields (Public and Private)
class Person {
  // Public field
  name;
  age = 0;
  
  // Private field
  #id;
  #secret = 'ความลับ';
  
  constructor(name, age) {
    this.name = name;
    this.age = age;
    this.#id = Math.random();
  }
  
  getId() {
    return this.#id;
  }
}

const p = new Person('สมชาย', 25);
console.log(p.name); // 'สมชาย'
// console.log(p.#id); // SyntaxError: private field

// 4. Top-level await (ใน modules เท่านั้น)
// const data = await fetchSomething(); // ทำได้ใน ES2022 modules

// ES2023

// 1. Array.findLast() / Array.findLastIndex()
const numbers = [1, 2, 3, 4, 5];
console.log(numbers.findLast(n => n % 2 === 0));      // 4
console.log(numbers.findLastIndex(n => n % 2 === 0)); // 3

// 2. Array.toSorted() / toReversed() / toSpliced() - immutable versions
const arr2 = [3, 1, 4, 1, 5, 9];
const sorted = arr2.toSorted();    // ไม่เปลี่ยน arr2
const reversed = arr2.toReversed(); // ไม่เปลี่ยน arr2
console.log(arr2);   // [3, 1, 4, 1, 5, 9] ยังเหมือนเดิม
console.log(sorted); // [1, 1, 3, 4, 5, 9]
```

---

## Step 410: สรุปและ Best Practices

```javascript
// Best Practices สรุป

// 1. ใช้ const เป็น default, ใช้ let เมื่อจำเป็น, หลีกเลี่ยง var
const MAX_SIZE = 100; // ค่าคงที่
let currentSize = 0;  // ค่าที่เปลี่ยนได้

// 2. ใช้ Arrow Functions สำหรับ callbacks และ methods ที่ต้องการ lexical this
class MyComponent {
  data = [];
  
  // ✓ Arrow function method - this ถูกต้องเสมอ
  fetchData = async () => {
    const result = await fetch('/api/data');
    this.data = await result.json(); // this ถูกต้อง
  };
  
  // ✓ ใช้ arrow ใน callback
  processData() {
    return this.data.map(item => item.value * 2);
  }
}

// 3. ใช้ default parameters แทน || สำหรับ numeric/boolean values
function createEl(tag = 'div', className = '', text = '') {
  return { tag, className, text };
}

// 4. ใช้ Optional Chaining และ Nullish Coalescing
function getUserCity(user) {
  return user?.address?.city ?? 'ไม่ระบุ';
}

console.log(getUserCity(null));                               // 'ไม่ระบุ'
console.log(getUserCity({ address: { city: 'กรุงเทพ' } })); // 'กรุงเทพ'
console.log(getUserCity({ name: 'สมชาย' }));                 // 'ไม่ระบุ'
```

---

## แบบฝึกหัด

### ระดับ 1: พื้นฐาน

**แบบฝึกหัดที่ 1:** แปลงโค้ดต่อไปนี้จาก ES5 เป็น ES6+

```javascript
// ES5 code ที่ต้องแปลง
var calculateArea = function(width, height) {
  height = height || width;
  return width * height;
};

var numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
var evenNumbers = numbers.filter(function(n) {
  return n % 2 === 0;
});

var User = function(name, age) {
  var self = this;
  this.name = name;
  this.age = age;
  setTimeout(function() {
    console.log(self.name + ' is ' + self.age);
  }, 1000);
};
```

**เฉลย:**

```javascript
// ES6+ version
const calculateArea = (width, height = width) => width * height;

const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
const evenNumbers = numbers.filter(n => n % 2 === 0);

class User {
  constructor(name, age) {
    this.name = name;
    this.age = age;
    setTimeout(() => {
      console.log(`${this.name} is ${this.age}`);
    }, 1000);
  }
}
```

### ระดับ 2: กลาง

**แบบฝึกหัดที่ 2:** สร้าง function ที่ใช้ default parameters ให้ครบถ้วน

```javascript
// สร้าง createBankAccount function ที่:
// - รับ owner (required)
// - รับ balance เริ่มต้น (default: 0)
// - รับ currency (default: 'THB')
// - รับ interestRate (default: 0.5)
// - return object ที่มี methods: deposit, withdraw, getBalance

function createBankAccount(owner, balance = 0, currency = 'THB', interestRate = 0.5) {
  return {
    owner,
    currency,
    interestRate,
    _balance: balance,
    
    deposit(amount) {
      this._balance += amount;
      return this;
    },
    
    withdraw(amount) {
      if (amount > this._balance) throw new Error('ยอดเงินไม่พอ');
      this._balance -= amount;
      return this;
    },
    
    getBalance() {
      return `${this._balance} ${this.currency}`;
    },
    
    addInterest() {
      this._balance *= (1 + this.interestRate / 100);
      return this;
    }
  };
}

const account = createBankAccount('สมชาย', 10000);
account.deposit(5000).addInterest();
console.log(account.getBalance()); // '15075 THB'
```

### ระดับ 3: ยาก

**แบบฝึกหัดที่ 3:** สร้าง EventEmitter ด้วย ES6+

```javascript
class EventEmitter {
  #events = new Map();
  
  on(event, callback) {
    if (!this.#events.has(event)) {
      this.#events.set(event, []);
    }
    this.#events.get(event).push(callback);
    return this; // สำหรับ chaining
  }
  
  off(event, callback) {
    if (this.#events.has(event)) {
      const filtered = this.#events.get(event).filter(cb => cb !== callback);
      this.#events.set(event, filtered);
    }
    return this;
  }
  
  emit(event, ...args) {
    if (this.#events.has(event)) {
      this.#events.get(event).forEach(cb => cb(...args));
    }
    return this;
  }
  
  once(event, callback) {
    const wrapper = (...args) => {
      callback(...args);
      this.off(event, wrapper);
    };
    return this.on(event, wrapper);
  }
}

// ทดสอบ
const emitter = new EventEmitter();
const handleLogin = (user) => console.log(`${user} เข้าสู่ระบบแล้ว`);

emitter.on('login', handleLogin);
emitter.once('firstVisit', () => console.log('ยินดีต้อนรับครั้งแรก!'));

emitter.emit('login', 'สมชาย');    // 'สมชาย เข้าสู่ระบบแล้ว'
emitter.emit('firstVisit');         // 'ยินดีต้อนรับครั้งแรก!'
emitter.emit('firstVisit');         // (ไม่มีผลลัพธ์ - once ใช้ครั้งเดียว)
emitter.emit('login', 'สมหญิง');  // 'สมหญิง เข้าสู่ระบบแล้ว'
```

---

## สรุปบทนี้

ในบทนี้เราได้เรียนรู้:

1. **ประวัติ ES6** - การปฏิวัติ JavaScript ในปี 2015
2. **var/let/const** - ความแตกต่างด้าน scope, hoisting, และการใช้งาน
3. **TDZ** - Temporal Dead Zone และผลกระทบต่อการเขียนโค้ด
4. **Arrow Functions** - รูปแบบต่างๆ และความแตกต่างกับ regular functions
5. **this binding** - Arrow functions ใช้ lexical this
6. **Default Parameters** - การกำหนดค่าเริ่มต้นให้ parameters
7. **ES2016-ES2023** - ฟีเจอร์ใหม่ที่เพิ่มขึ้นในแต่ละปี

**หลักการสำคัญ:**
- ใช้ `const` เป็น default, `let` เมื่อต้องการ reassign, หลีกเลี่ยง `var`
- Arrow functions เหมาะสำหรับ callbacks ที่ต้องการ lexical `this`
- Regular functions เหมาะสำหรับ object methods และ constructors
- Default parameters ทำให้ API ใช้งานง่ายขึ้น
