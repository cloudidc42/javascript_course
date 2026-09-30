# ส่วนที่ 2: ตัวแปรและชนิดข้อมูล (Variables & Data Types)

## คำอธิบาย
ในส่วนนี้คุณจะได้เรียนรู้เกี่ยวกับตัวแปรและชนิดข้อมูลทั้งหมดใน JavaScript อย่างละเอียด ตั้งแต่ความแตกต่างระหว่าง var, let, const ไปจนถึงชนิดข้อมูลทั้ง 8 ชนิด พร้อมการแปลงชนิดข้อมูล เนื้อหาครอบคลุม Step 11-30

---

## Step 11: var - ตัวแปรแบบเก่า

`var` เป็นวิธีการประกาศตัวแปรแบบดั้งเดิมใน JavaScript ก่อน ES6 แต่มีพฤติกรรมที่อาจสร้างความสับสน

```javascript
// ตัวอย่างที่ 1: var พื้นฐาน
var name = 'สมชาย';
var age = 25;
var isStudent = true;

console.log(name);      // สมชาย
console.log(age);       // 25
console.log(isStudent); // true

// ตัวอย่างที่ 2: var ประกาศซ้ำได้
var x = 10;
var x = 20;  // ไม่ error! (อันตราย)
console.log(x); // 20

// ตัวอย่างที่ 3: var มี Function Scope (ไม่ใช่ Block Scope)
function testVar() {
    if (true) {
        var blockVar = 'ใน if block';
    }
    console.log(blockVar); // 'ใน if block' - รั่วออกมานอก block!
}
testVar();
```

```javascript
// ตัวอย่างที่ 4: var Hoisting
console.log(hoisted); // undefined (ไม่ error!)
var hoisted = 'ค่าของฉัน';
console.log(hoisted); // 'ค่าของฉัน'

// JavaScript ทำแบบนี้จริงๆ:
// var hoisted;          // ประกาศถูก hoist ขึ้นไปบน
// console.log(hoisted); // undefined
// hoisted = 'ค่าของฉัน';
// console.log(hoisted); // 'ค่าของฉัน'

// ตัวอย่างที่ 5: var ใน loop - ปัญหาคลาสสิก
for (var i = 0; i < 3; i++) {
    setTimeout(function() {
        console.log(i); // แสดง 3, 3, 3 (ไม่ใช่ 0, 1, 2!)
    }, 100);
}
// เพราะ var ไม่มี block scope ค่า i จะเป็น 3 ทุกครั้ง
```

---

## Step 12: let - ตัวแปรแบบใหม่ (แนะนำ)

`let` เป็นการปรับปรุง var ให้ดีขึ้น มี Block Scope และป้องกัน hoisting problems

```javascript
// ตัวอย่างที่ 6: let พื้นฐาน
let city = 'กรุงเทพ';
let population = 10000000;

// เปลี่ยนค่าได้
city = 'เชียงใหม่';
population = 300000;
console.log(city, population); // เชียงใหม่ 300000

// ตัวอย่างที่ 7: let ประกาศซ้ำไม่ได้
let score = 100;
// let score = 200; // SyntaxError: Identifier 'score' has already been declared

// ตัวอย่างที่ 8: let มี Block Scope
function testLet() {
    if (true) {
        let blockLet = 'ใน if block';
        console.log(blockLet); // OK - อยู่ใน block
    }
    // console.log(blockLet); // ReferenceError: blockLet is not defined
}
testLet();
```

```javascript
// ตัวอย่างที่ 9: let ใน loop - ทำงานถูกต้อง
for (let j = 0; j < 3; j++) {
    setTimeout(function() {
        console.log(j); // แสดง 0, 1, 2 - ถูกต้อง!
    }, 100);
}
// let สร้าง closure ใหม่ในแต่ละ iteration

// ตัวอย่างที่ 10: Temporal Dead Zone (TDZ) - ต้องระวัง
// console.log(tdz); // ReferenceError! (ต่างจาก var ที่เป็น undefined)
let tdz = 'I exist now';
console.log(tdz); // 'I exist now'

// ตัวอย่างที่ 11: let ใน nested scope
let outerValue = 'ภายนอก';

function outer() {
    let outerValue = 'ใน outer function'; // shadowing - ซ่อนค่าภายนอก
    
    function inner() {
        let outerValue = 'ใน inner function'; // shadowing อีกครั้ง
        console.log(outerValue); // 'ใน inner function'
    }
    
    inner();
    console.log(outerValue); // 'ใน outer function'
}

outer();
console.log(outerValue); // 'ภายนอก'
```

---

## Step 13: const - ค่าคงที่

`const` ประกาศค่าที่ไม่ต้องการเปลี่ยนแปลง แต่ระวัง! Object และ Array ยังเปลี่ยนได้

```javascript
// ตัวอย่างที่ 12: const พื้นฐาน
const PI = 3.14159265358979;
const GRAVITY = 9.8; // m/s²
const APP_NAME = 'MyApp';
const MAX_RETRIES = 3;

// PI = 3.14; // TypeError: Assignment to constant variable

// ตัวอย่างที่ 13: const ต้องกำหนดค่าทันที
// const UNDEFINED_CONST; // SyntaxError: Missing initializer in const declaration
const INITIALIZED_CONST = 'ต้องกำหนดค่าทันที';

// ตัวอย่างที่ 14: const กับ Object - Object ยังเปลี่ยนได้!
const person = {
    name: 'สมชาย',
    age: 25
};

person.age = 26;        // OK! เปลี่ยน property ได้
person.job = 'นักพัฒนา'; // OK! เพิ่ม property ได้
console.log(person);    // { name: 'สมชาย', age: 26, job: 'นักพัฒนา' }

// person = {}; // TypeError: ไม่สามารถ reassign ตัวแปรได้
```

```javascript
// ตัวอย่างที่ 15: const กับ Array - Array ยังเปลี่ยนได้!
const fruits = ['แอปเปิ้ล', 'กล้วย', 'ส้ม'];

fruits.push('มะม่วง');    // OK! เพิ่มได้
fruits[0] = 'สับปะรด';   // OK! เปลี่ยนได้
fruits.pop();            // OK! ลบได้
console.log(fruits);     // ['สับปะรด', 'กล้วย', 'ส้ม']

// fruits = []; // TypeError: ไม่สามารถ reassign ได้

// ตัวอย่างที่ 16: Object.freeze() - ทำให้เปลี่ยนไม่ได้จริงๆ
const frozenPerson = Object.freeze({
    name: 'สมหญิง',
    age: 22
});

frozenPerson.age = 23;  // ไม่ error แต่ไม่เปลี่ยน! (silent fail)
frozenPerson.job = 'ครู'; // ไม่เปลี่ยน!
console.log(frozenPerson); // { name: 'สมหญิง', age: 22 }

// ในกรณี strict mode จะ throw error:
// 'use strict';
// frozenPerson.age = 23; // TypeError: Cannot assign to read only property
```

---

## Step 14: เปรียบเทียบ var, let, const

```javascript
// ตัวอย่างที่ 17: ตารางเปรียบเทียบ
/*
feature          | var      | let      | const
-----------------|----------|----------|--------
scope            | function | block    | block
re-declaration   | yes      | no       | no
re-assignment    | yes      | yes      | no
hoisting         | yes(undef)| TDZ     | TDZ
global property  | yes      | no       | no
*/

// ตัวอย่างที่ 18: การเลือกใช้
const API_URL = 'https://api.example.com'; // ค่าไม่เปลี่ยน = const
let currentUser = null;                     // เปลี่ยนได้ = let
let count = 0;                             // เปลี่ยนได้ = let

// Rule of thumb: ใช้ const ก่อนเสมอ ถ้าต้องเปลี่ยนค่าค่อยใช้ let
// ห้ามใช้ var (ยกเว้นมีเหตุผลพิเศษ)

// ตัวอย่างที่ 19: Destructuring กับ const
const { name, age } = { name: 'สมชาย', age: 25 };
console.log(name, age); // สมชาย 25

const [first, second, ...rest] = [1, 2, 3, 4, 5];
console.log(first, second, rest); // 1 2 [3, 4, 5]
```

---

## Step 15: String - ชนิดข้อมูลข้อความ

```javascript
// ตัวอย่างที่ 20: การสร้าง String
let str1 = 'Single quotes';
let str2 = "Double quotes";
let str3 = `Template literal`;

// ทั้งสามเหมือนกัน แต่ template literal มีความสามารถพิเศษ
console.log(typeof str1); // "string"

// ตัวอย่างที่ 21: Escape characters
let newline = 'บรรทัดแรก\nบรรทัดสอง';
let tab = 'คอลัมน์1\tคอลัมน์2';
let quote = 'เขาพูดว่า \'สวัสดี\'';
let backslash = 'C:\\Users\\Desktop';
let unicode = '\u0E2A\u0E27\u0E31\u0E2A\u0E14\u0E35'; // สวัสดี

console.log(newline);
console.log(tab);
console.log(quote);
console.log(unicode);

// ตัวอย่างที่ 22: String concatenation
let firstName = 'สมชาย';
let lastName = 'ใจดี';

// วิธีเก่า
let fullName1 = firstName + ' ' + lastName;

// Template literal (แนะนำ)
let fullName2 = `${firstName} ${lastName}`;

console.log(fullName1); // สมชาย ใจดี
console.log(fullName2); // สมชาย ใจดี
```

```javascript
// ตัวอย่างที่ 23: Template Literals ขั้นสูง
const price = 1500;
const quantity = 3;
const discount = 0.1;

const orderSummary = `
=== ใบสั่งซื้อ ===
ราคาต่อชิ้น: ${price} บาท
จำนวน: ${quantity} ชิ้น
ราคารวม: ${price * quantity} บาท
ส่วนลด ${discount * 100}%: ${price * quantity * discount} บาท
ราคาสุทธิ: ${price * quantity * (1 - discount)} บาท
`;

console.log(orderSummary);

// ตัวอย่างที่ 24: Multiline String
const multiline = `บรรทัดที่ 1
บรรทัดที่ 2
บรรทัดที่ 3`;

console.log(multiline);

// ตัวอย่างที่ 25: Expression ใน Template Literal
const a = 5, b = 3;
console.log(`${a} + ${b} = ${a + b}`);
console.log(`${a} > ${b} ? ${a > b}`);
console.log(`Max: ${Math.max(a, b)}`);
console.log(`${a} is ${a % 2 === 0 ? 'even' : 'odd'}`);
```

```javascript
// ตัวอย่างที่ 26: String Properties และ Methods
const text = 'Hello, สวัสดี World!';

// Properties
console.log(text.length); // 19

// Methods - ไม่เปลี่ยน string ต้นฉบับ (immutable)
console.log(text.toUpperCase());      // HELLO, สวัสดี WORLD!
console.log(text.toLowerCase());      // hello, สวัสดี world!
console.log(text.indexOf('สวัสดี')); // 7
console.log(text.includes('World')); // true
console.log(text.startsWith('Hello')); // true
console.log(text.endsWith('!'));     // true

console.log(text.slice(0, 5));       // Hello
console.log(text.slice(7, 13));      // สวัสดี
console.log(text.slice(-6));         // orld!

console.log(text.replace('Hello', 'สวัสดี'));  // สวัสดี, สวัสดี World!
console.log(text.split(', '));        // ['Hello', 'สวัสดี World!']
console.log('  spaces  '.trim());    // 'spaces'
console.log('  spaces  '.trimStart()); // 'spaces  '
console.log('  spaces  '.trimEnd());   // '  spaces'

console.log('abc'.repeat(3));        // abcabcabc
console.log('5'.padStart(3, '0'));   // 005
console.log('5'.padEnd(3, '0'));     // 500
```

```javascript
// ตัวอย่างที่ 27: String Methods ที่สำคัญเพิ่มเติม
const csv = 'สมชาย,25,กรุงเทพ,นักพัฒนา';
const parts = csv.split(',');
console.log(parts); // ['สมชาย', '25', 'กรุงเทพ', 'นักพัฒนา']
console.log(parts.join(' | ')); // สมชาย | 25 | กรุงเทพ | นักพัฒนา

// charAt vs []
const word = 'JavaScript';
console.log(word.charAt(0));  // J
console.log(word[0]);          // J
console.log(word.charCodeAt(0)); // 74 (รหัส ASCII)
console.log(String.fromCharCode(74)); // J

// search กับ regex
const email = 'user@example.com';
console.log(email.search(/@/)); // 4
console.log(email.match(/@\w+/)); // ['@example']

// replaceAll
const sentence = 'the cat sat on the mat';
console.log(sentence.replaceAll('the', 'a')); // a cat sat on a mat

// at() method (ES2022)
console.log(word.at(0));   // J
console.log(word.at(-1));  // t
```

---

## Step 16: Number - ชนิดข้อมูลตัวเลข

```javascript
// ตัวอย่างที่ 28: Number basics
let integer = 42;
let float = 3.14;
let negative = -10;
let exponent = 2.5e6;  // 2,500,000
let binary = 0b1010;   // 10 (เลขฐาน 2)
let octal = 0o17;      // 15 (เลขฐาน 8)
let hex = 0xFF;        // 255 (เลขฐาน 16)

console.log(exponent); // 2500000
console.log(binary);   // 10
console.log(octal);    // 15
console.log(hex);      // 255

// ตัวอย่างที่ 29: Number Separators (ES2021) - อ่านง่ายขึ้น
const million = 1_000_000;
const bytes = 0xFF_EC_D9_12;
const price = 1_299.99;
console.log(million); // 1000000
console.log(price);   // 1299.99
```

```javascript
// ตัวอย่างที่ 30: Special Values - NaN
console.log(NaN);           // NaN
console.log(typeof NaN);    // "number" (!)
console.log(isNaN(NaN));    // true
console.log(isNaN('hello')); // true
console.log(isNaN(42));      // false
console.log(Number.isNaN(NaN));    // true
console.log(Number.isNaN('hello')); // false (แม่นกว่า isNaN)

// NaN ไม่เท่ากับตัวเอง
console.log(NaN === NaN); // false!
console.log(NaN !== NaN); // true!

// ตัวอย่างที่ 31: Infinity
console.log(Infinity);          // Infinity
console.log(-Infinity);         // -Infinity
console.log(1 / 0);             // Infinity
console.log(-1 / 0);            // -Infinity
console.log(isFinite(Infinity)); // false
console.log(isFinite(42));       // true

// ตัวอย่างที่ 32: Number Limits
console.log(Number.MAX_SAFE_INTEGER); // 9007199254740991 (2^53 - 1)
console.log(Number.MIN_SAFE_INTEGER); // -9007199254740991
console.log(Number.MAX_VALUE);        // 1.7976931348623157e+308
console.log(Number.MIN_VALUE);        // 5e-324
console.log(Number.EPSILON);          // 2.220446049250313e-16
```

```javascript
// ตัวอย่างที่ 33: Floating Point ปัญหาที่ต้องรู้
console.log(0.1 + 0.2);           // 0.30000000000000004 (!!)
console.log(0.1 + 0.2 === 0.3);   // false (!!)

// วิธีแก้: ใช้ toFixed หรือ epsilon comparison
console.log((0.1 + 0.2).toFixed(1));  // "0.3" (string)
console.log(parseFloat((0.1 + 0.2).toFixed(10))); // 0.3

// หรือใช้ Number.EPSILON
function nearlyEqual(a, b) {
    return Math.abs(a - b) < Number.EPSILON;
}
console.log(nearlyEqual(0.1 + 0.2, 0.3)); // true
```

```javascript
// ตัวอย่างที่ 34: Number Methods
const num = 123.456789;

console.log(num.toFixed(2));        // "123.46" (string!)
console.log(num.toPrecision(5));    // "123.46"
console.log(num.toString());        // "123.456789"
console.log(num.toString(2));       // แปลงเป็น binary string
console.log(num.toString(16));      // แปลงเป็น hex string
console.log(num.toExponential(2));  // "1.23e+2"

// Number static methods
console.log(Number.isInteger(42));      // true
console.log(Number.isInteger(42.5));    // false
console.log(Number.isFinite(42));       // true
console.log(Number.isFinite(Infinity)); // false
console.log(Number.isSafeInteger(9007199254740991));  // true
console.log(Number.isSafeInteger(9007199254740992));  // false!
```

---

## Step 17: Boolean - ชนิดข้อมูล true/false

```javascript
// ตัวอย่างที่ 35: Boolean basics
let isActive = true;
let isDeleted = false;
let hasPermission = true;

console.log(typeof isActive); // "boolean"

// ตัวอย่างที่ 36: Truthy values - ค่าที่ถือว่าเป็น true
if ('hello') console.log('string ที่ไม่ว่างเป็น truthy');
if (1) console.log('ตัวเลขที่ไม่ใช่ 0 เป็น truthy');
if (-1) console.log('ตัวเลขลบก็เป็น truthy');
if ([]) console.log('array ว่างเป็น truthy');
if ({}) console.log('object ว่างเป็น truthy');
if (function(){}) console.log('function เป็น truthy');

// ตัวอย่างที่ 37: Falsy values - ค่าที่ถือว่าเป็น false
const falsyValues = [false, 0, -0, 0n, '', "", ``, null, undefined, NaN];

falsyValues.forEach(val => {
    if (!val) console.log(`${String(val)} (${typeof val}) เป็น falsy`);
});
```

```javascript
// ตัวอย่างที่ 38: Boolean conversion
console.log(Boolean(0));          // false
console.log(Boolean(''));         // false
console.log(Boolean(null));       // false
console.log(Boolean(undefined));  // false
console.log(Boolean(NaN));        // false

console.log(Boolean(1));          // true
console.log(Boolean('0'));        // true! ('0' คือ string ที่ไม่ว่าง)
console.log(Boolean([]));         // true! (array ว่างก็ true)
console.log(Boolean({}));         // true! (object ว่างก็ true)

// Double NOT !! - แปลงเป็น boolean
console.log(!!'hello'); // true
console.log(!!0);       // false
console.log(!!null);    // false
console.log(!![]);      // true
```

---

## Step 18: null และ undefined

```javascript
// ตัวอย่างที่ 39: null vs undefined
let notAssigned;           // undefined - ยังไม่กำหนดค่า
let explicitNull = null;   // null - ตั้งใจให้ว่างเปล่า

console.log(notAssigned);   // undefined
console.log(explicitNull);  // null

console.log(typeof undefined); // "undefined"
console.log(typeof null);      // "object" (historical bug ของ JS!)

// ตัวอย่างที่ 40: ความแตกต่าง
console.log(null == undefined);  // true  (loose equality)
console.log(null === undefined); // false (strict equality)
console.log(null == 0);          // false
console.log(undefined == 0);    // false
console.log(null == '');         // false

// ตัวอย่างที่ 41: เมื่อไหร่ที่เจอ undefined
// 1. ตัวแปรที่ไม่ได้กำหนดค่า
let x;
console.log(x); // undefined

// 2. function ที่ไม่ return
function noReturn() {}
console.log(noReturn()); // undefined

// 3. parameter ที่ไม่ได้ส่งไป
function greet(name) {
    console.log(name); // undefined ถ้าไม่ส่ง name
}
greet();

// 4. property ที่ไม่มีใน object
const obj = { a: 1 };
console.log(obj.b); // undefined
```

```javascript
// ตัวอย่างที่ 42: การตรวจสอบ null/undefined
const value = null;

// วิธีที่ 1: strict equality
if (value === null) console.log('เป็น null');
if (value === undefined) console.log('เป็น undefined');

// วิธีที่ 2: loose equality (ตรวจทั้ง null และ undefined)
if (value == null) console.log('เป็น null หรือ undefined');

// วิธีที่ 3: Optional chaining
const user = null;
console.log(user?.name);       // undefined (ไม่ error!)
console.log(user?.address?.city); // undefined

// วิธีที่ 4: Nullish coalescing
const name = null ?? 'ไม่ระบุ';
console.log(name); // ไม่ระบุ
```

---

## Step 19: Symbol - ค่าที่ unique

```javascript
// ตัวอย่างที่ 43: Symbol basics
const sym1 = Symbol('description');
const sym2 = Symbol('description');

console.log(sym1 === sym2);      // false! แต่ละ Symbol unique
console.log(typeof sym1);        // "symbol"
console.log(sym1.toString());    // "Symbol(description)"
console.log(sym1.description);   // "description"

// ตัวอย่างที่ 44: Symbol ใช้เป็น Object key
const ID = Symbol('id');
const user = {
    name: 'สมชาย',
    [ID]: 12345  // Symbol key ไม่แสดงใน for...in
};

console.log(user[ID]);       // 12345
console.log(user.name);      // สมชาย
console.log(Object.keys(user)); // ['name'] - ไม่มี Symbol

// ตัวอย่างที่ 45: Symbol.for - Shared Symbol
const globalSym1 = Symbol.for('app.id');
const globalSym2 = Symbol.for('app.id');
console.log(globalSym1 === globalSym2); // true! (shared)
```

---

## Step 20: BigInt - จำนวนเต็มขนาดใหญ่

```javascript
// ตัวอย่างที่ 46: BigInt basics
const big1 = 9007199254740991n;     // n นำหน้าทำให้เป็น BigInt
const big2 = BigInt(9007199254740991);
const big3 = BigInt('9007199254740992'); // ใหญ่กว่า MAX_SAFE_INTEGER

console.log(typeof big1);  // "bigint"
console.log(big1 + 1n);    // 9007199254740992n

// ตัวอย่างที่ 47: BigInt ไม่ผสมกับ Number ปกติ
// console.log(big1 + 1);  // TypeError!
console.log(big1 + BigInt(1)); // OK

// แปลงระหว่างกัน
console.log(Number(big1));   // 9007199254740991
console.log(BigInt(42));     // 42n

// ตัวอย่างที่ 48: การใช้งาน BigInt
const MAX_SAFE = 9007199254740991n;
console.log(MAX_SAFE + 1n); // 9007199254740992n - ถูกต้อง!
console.log(MAX_SAFE + 2n); // 9007199254740993n - ถูกต้อง!
// ต่างจาก Number ที่จะ round off
```

---

## Step 21: typeof Operator

```javascript
// ตัวอย่างที่ 49: typeof ทุกชนิด
console.log(typeof 'hello');         // "string"
console.log(typeof 42);              // "number"
console.log(typeof 3.14);           // "number"
console.log(typeof true);            // "boolean"
console.log(typeof undefined);       // "undefined"
console.log(typeof null);            // "object" (bug!)
console.log(typeof Symbol());        // "symbol"
console.log(typeof 42n);             // "bigint"
console.log(typeof {});              // "object"
console.log(typeof []);              // "object"
console.log(typeof function(){});    // "function"

// ตัวอย่างที่ 50: ตรวจสอบ null อย่างถูกต้อง
function getType(value) {
    if (value === null) return 'null';
    if (Array.isArray(value)) return 'array';
    return typeof value;
}

console.log(getType(null));    // "null"
console.log(getType([]));      // "array"
console.log(getType({}));      // "object"
console.log(getType('hi'));    // "string"
```

---

## Step 22: Type Conversion (การแปลงชนิดข้อมูล)

### Implicit Conversion (JavaScript แปลงให้อัตโนมัติ)

```javascript
// ตัวอย่างที่ 51: Implicit conversion กับ + operator
console.log(1 + '2');      // "12" (number + string = string)
console.log('3' + 4);      // "34"
console.log(1 + 2 + '3'); // "33" (1+2=3 แล้ว 3+'3'="33")
console.log('1' + 2 + 3); // "123"
console.log(true + 1);    // 2 (true = 1)
console.log(false + 1);   // 1 (false = 0)
console.log(null + 1);    // 1 (null = 0)

// ตัวอย่างที่ 52: Implicit conversion กับ - * / %
console.log('5' - 3);    // 2 (string แปลงเป็น number!)
console.log('5' * 2);    // 10
console.log('10' / 2);   // 5
console.log('5' - '3');  // 2
console.log('abc' - 1);  // NaN

// ตัวอย่างที่ 53: Implicit conversion กับ comparison
console.log('5' > 3);     // true (string แปลงเป็น number)
console.log('10' > '9');  // false! (เปรียบเทียบเป็น string, '1' < '9')
console.log(null > 0);    // false
console.log(null == 0);   // false
console.log(null >= 0);   // true (!!) - null = 0 เฉพาะ >= <=
```

```javascript
// ตัวอย่างที่ 54: == vs === (loose vs strict equality)
// == มีการแปลงชนิดข้อมูล (coercion)
console.log(1 == '1');     // true!
console.log(0 == false);   // true!
console.log(0 == '');      // true!
console.log('' == false);  // true!
console.log(null == undefined); // true!

// === ไม่มีการแปลง - แนะนำให้ใช้เสมอ
console.log(1 === '1');    // false
console.log(0 === false);  // false
console.log(null === undefined); // false
```

---

## Step 23: Explicit Conversion

```javascript
// ตัวอย่างที่ 55: แปลงเป็น Number
console.log(Number('42'));        // 42
console.log(Number('3.14'));      // 3.14
console.log(Number(''));          // 0
console.log(Number('  '));        // 0
console.log(Number('abc'));       // NaN
console.log(Number(true));        // 1
console.log(Number(false));       // 0
console.log(Number(null));        // 0
console.log(Number(undefined));   // NaN
console.log(Number([]));          // 0
console.log(Number([1]));         // 1
console.log(Number([1, 2]));      // NaN

// ตัวอย่างที่ 56: parseInt
console.log(parseInt('42'));         // 42
console.log(parseInt('42.7'));       // 42 (ตัดทศนิยม)
console.log(parseInt('42abc'));      // 42 (หยุดที่ตัวเลข)
console.log(parseInt('abc'));        // NaN
console.log(parseInt('0xFF', 16));   // 255
console.log(parseInt('10', 2));      // 2 (binary)
console.log(parseInt('17', 8));      // 15 (octal)
```

```javascript
// ตัวอย่างที่ 57: parseFloat
console.log(parseFloat('3.14'));     // 3.14
console.log(parseFloat('3.14abc')); // 3.14
console.log(parseFloat('abc'));      // NaN
console.log(parseFloat('1e2'));      // 100

// ตัวอย่างที่ 58: แปลงเป็น String
console.log(String(42));            // "42"
console.log(String(3.14));          // "3.14"
console.log(String(true));          // "true"
console.log(String(false));         // "false"
console.log(String(null));          // "null"
console.log(String(undefined));     // "undefined"

// .toString() method
console.log((42).toString());       // "42"
console.log((255).toString(16));    // "ff" (hex)
console.log((8).toString(2));       // "1000" (binary)

// + '' - แปลงเป็น string แบบสั้น
console.log(42 + '');              // "42"
console.log(true + '');            // "true"
```

```javascript
// ตัวอย่างที่ 59: แปลงเป็น Boolean
console.log(Boolean(0));          // false
console.log(Boolean(''));         // false
console.log(Boolean(null));       // false
console.log(Boolean(undefined));  // false
console.log(Boolean(NaN));        // false

console.log(Boolean(1));          // true
console.log(Boolean('a'));        // true
console.log(Boolean([]));         // true
console.log(Boolean({}));         // true

// !! shortcut
console.log(!!0);    // false
console.log(!!1);    // true
console.log(!!'');   // false
console.log(!!'hi'); // true
```

---

## Step 24: String Methods เพิ่มเติม

```javascript
// ตัวอย่างที่ 60: String Methods ที่ต้องรู้
const sentence = 'The Quick Brown Fox Jumps Over The Lazy Dog';

// Case methods
console.log(sentence.toLowerCase()); // the quick brown fox...
console.log(sentence.toUpperCase()); // THE QUICK BROWN FOX...

// Search
console.log(sentence.indexOf('Fox'));       // 16
console.log(sentence.lastIndexOf('The'));   // 31
console.log(sentence.includes('Quick'));    // true
console.log(sentence.startsWith('The'));    // true
console.log(sentence.endsWith('Dog'));      // true

// Extraction
console.log(sentence.substring(4, 9));  // Quick
console.log(sentence.slice(4, 9));      // Quick
console.log(sentence.slice(-3));        // Dog

// ตัวอย่างที่ 61: Split และ Join
const words = sentence.split(' ');
console.log(words.length); // 9
console.log(words[0]);     // The

// นับคำที่ซ้ำ
const wordCount = {};
words.forEach(word => {
    wordCount[word.toLowerCase()] = (wordCount[word.toLowerCase()] || 0) + 1;
});
console.log(wordCount);
```

```javascript
// ตัวอย่างที่ 62: String Manipulation ที่มีประโยชน์
// ทำ Title Case
function toTitleCase(str) {
    return str.split(' ')
        .map(word => word.charAt(0).toUpperCase() + word.slice(1).toLowerCase())
        .join(' ');
}
console.log(toTitleCase('hello world')); // Hello World

// Truncate ข้อความยาว
function truncate(str, maxLength) {
    return str.length > maxLength ? str.slice(0, maxLength - 3) + '...' : str;
}
console.log(truncate('ข้อความที่ยาวมากๆ', 10)); // ข้อความที่...

// นับจำนวนคำ
function countWords(str) {
    return str.trim().split(/\s+/).length;
}
console.log(countWords('Hello World JavaScript')); // 3

// ลบ HTML tags
function stripHtml(html) {
    return html.replace(/<[^>]*>/g, '');
}
console.log(stripHtml('<h1>สวัสดี</h1><p>โลก</p>')); // สวัสดีโลก
```

```javascript
// ตัวอย่างที่ 63: String padding และ formatting
// padStart - จัดชิดขวา
const numbers = [1, 10, 100, 1000];
numbers.forEach(n => {
    console.log(String(n).padStart(6, ' '));
});

// เลขลำดับ
for (let i = 1; i <= 10; i++) {
    console.log(`${String(i).padStart(3, '0')}. Item ${i}`);
}

// padEnd
const items = ['กาแฟ', 'ชานมไข่มุก', 'น้ำเปล่า'];
items.forEach(item => {
    console.log(`${item.padEnd(15, '.')} 50 บาท`);
});
```

---

## Step 25: Number Methods ขั้นสูง

```javascript
// ตัวอย่างที่ 64: Math Object
console.log(Math.PI);           // 3.141592653589793
console.log(Math.E);            // 2.718281828459045
console.log(Math.sqrt(16));     // 4
console.log(Math.cbrt(27));     // 3 (cube root)
console.log(Math.pow(2, 10));   // 1024
console.log(Math.abs(-5));      // 5
console.log(Math.ceil(4.1));    // 5
console.log(Math.floor(4.9));   // 4
console.log(Math.round(4.5));   // 5
console.log(Math.trunc(4.9));   // 4 (ตัดทศนิยม)
console.log(Math.trunc(-4.9));  // -4

// ตัวอย่างที่ 65: Math.min, max, random
console.log(Math.min(1, 5, 3, 9, 2)); // 1
console.log(Math.max(1, 5, 3, 9, 2)); // 9
console.log(Math.random());    // 0 ถึง 0.999...

// Random ในช่วง [min, max)
function randomBetween(min, max) {
    return Math.floor(Math.random() * (max - min)) + min;
}
console.log(randomBetween(1, 7));    // 1-6 (ลูกเต๋า)
console.log(randomBetween(0, 100)); // 0-99

// Random ในช่วง [min, max] รวม max
function randomInclusive(min, max) {
    return Math.floor(Math.random() * (max - min + 1)) + min;
}
console.log(randomInclusive(1, 6)); // 1-6

// ตัวอย่างที่ 66: Math.log
console.log(Math.log(Math.E)); // 1 (natural log)
console.log(Math.log2(8));     // 3
console.log(Math.log10(1000)); // 3

// ตัวอย่างที่ 67: Trigonometry
console.log(Math.sin(Math.PI / 2)); // 1
console.log(Math.cos(0));           // 1
console.log(Math.tan(Math.PI / 4)); // 1
```

---

## Step 26: Type Checking ขั้นสูง

```javascript
// ตัวอย่างที่ 68: instanceof
const arr = [1, 2, 3];
const obj = { a: 1 };
const date = new Date();
const regex = /abc/;

console.log(arr instanceof Array);    // true
console.log(obj instanceof Object);   // true
console.log(date instanceof Date);    // true
console.log(regex instanceof RegExp); // true
console.log(arr instanceof Object);   // true! Array extends Object

// ตัวอย่างที่ 69: Object.prototype.toString - แม่นกว่า typeof
function getExactType(value) {
    return Object.prototype.toString.call(value);
}

console.log(getExactType(42));         // "[object Number]"
console.log(getExactType('hello'));    // "[object String]"
console.log(getExactType(true));       // "[object Boolean]"
console.log(getExactType(null));       // "[object Null]"
console.log(getExactType(undefined));  // "[object Undefined]"
console.log(getExactType([]));         // "[object Array]"
console.log(getExactType({}));         // "[object Object]"
console.log(getExactType(/abc/));      // "[object RegExp]"
console.log(getExactType(new Date())); // "[object Date]"
```

```javascript
// ตัวอย่างที่ 70: Type Guard Functions
function isString(value) {
    return typeof value === 'string';
}

function isNumber(value) {
    return typeof value === 'number' && !isNaN(value);
}

function isArray(value) {
    return Array.isArray(value);
}

function isObject(value) {
    return value !== null && typeof value === 'object' && !Array.isArray(value);
}

function isFunction(value) {
    return typeof value === 'function';
}

function isNullOrUndefined(value) {
    return value == null;
}

// ทดสอบ
const values = ['hello', 42, true, null, undefined, [], {}, () => {}];
values.forEach(v => {
    console.log(`${JSON.stringify(v)}: string=${isString(v)}, number=${isNumber(v)}, array=${isArray(v)}, object=${isObject(v)}`);
});
```

---

## Step 27: Variable Scope ขั้นสูง

```javascript
// ตัวอย่างที่ 71: Global Scope
var globalVar = 'global var';
let globalLet = 'global let';
const globalConst = 'global const';

// var กลายเป็น property ของ window (browser)
// console.log(window.globalVar);  // 'global var' (browser only)
// console.log(window.globalLet);  // undefined

// ตัวอย่างที่ 72: Function Scope
function testScope() {
    var funcVar = 'function var';
    let funcLet = 'function let';
    
    console.log(funcVar); // OK
    console.log(funcLet); // OK
}

testScope();
// console.log(funcVar); // ReferenceError
// console.log(funcLet); // ReferenceError

// ตัวอย่างที่ 73: Block Scope
{
    var blockVar = 'block var';    // รั่วออกนอก block!
    let blockLet = 'block let';   // อยู่แค่ใน block
    const blockConst = 'block const'; // อยู่แค่ใน block
}

console.log(blockVar); // 'block var' (var รั่วออกมา)
// console.log(blockLet);   // ReferenceError
// console.log(blockConst); // ReferenceError
```

```javascript
// ตัวอย่างที่ 74: Closure
function createCounter() {
    let count = 0; // private variable
    
    return {
        increment() { count++; },
        decrement() { count--; },
        getCount() { return count; }
    };
}

const counter = createCounter();
counter.increment();
counter.increment();
counter.increment();
counter.decrement();
console.log(counter.getCount()); // 2

// count ไม่สามารถเข้าถึงได้จากภายนอก
// console.log(counter.count); // undefined

// ตัวอย่างที่ 75: Closure ใน loop
function createFunctions() {
    const functions = [];
    
    for (let i = 0; i < 5; i++) {
        functions.push(() => console.log(i)); // let มี block scope
    }
    
    return functions;
}

const fns = createFunctions();
fns[0](); // 0
fns[2](); // 2
fns[4](); // 4
```

---

## Step 28: Destructuring

```javascript
// ตัวอย่างที่ 76: Object Destructuring
const student = {
    name: 'สมชาย',
    age: 20,
    grade: 'A',
    address: {
        city: 'กรุงเทพ',
        province: 'กรุงเทพมหานคร'
    }
};

// พื้นฐาน
const { name, age, grade } = student;
console.log(name, age, grade); // สมชาย 20 A

// เปลี่ยนชื่อตัวแปร
const { name: studentName, age: studentAge } = student;
console.log(studentName, studentAge); // สมชาย 20

// ค่า default
const { name: n, score = 0, gpa = 3.5 } = student;
console.log(n, score, gpa); // สมชาย 0 3.5

// Nested destructuring
const { address: { city, province } } = student;
console.log(city, province); // กรุงเทพ กรุงเทพมหานคร
```

```javascript
// ตัวอย่างที่ 77: Array Destructuring
const colors = ['แดง', 'เขียว', 'น้ำเงิน', 'เหลือง', 'ม่วง'];

// พื้นฐาน
const [first, second, third] = colors;
console.log(first, second, third); // แดง เขียว น้ำเงิน

// ข้ามบางตัว
const [, , blue] = colors;
console.log(blue); // น้ำเงิน

// Rest elements
const [primary, ...others] = colors;
console.log(primary);  // แดง
console.log(others);   // ['เขียว', 'น้ำเงิน', 'เหลือง', 'ม่วง']

// Swap variables
let a = 1, b = 2;
[a, b] = [b, a];
console.log(a, b); // 2 1

// Default values
const [x = 0, y = 0, z = 0] = [1, 2];
console.log(x, y, z); // 1 2 0
```

---

## Step 29: Spread Operator

```javascript
// ตัวอย่างที่ 78: Spread กับ Array
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];

// รวม array
const combined = [...arr1, ...arr2];
console.log(combined); // [1, 2, 3, 4, 5, 6]

// copy array
const copy = [...arr1];
copy.push(4);
console.log(arr1);  // [1, 2, 3] - ไม่เปลี่ยน
console.log(copy);  // [1, 2, 3, 4]

// แทรกตรงกลาง
const inserted = [...arr1, 10, 20, ...arr2];
console.log(inserted); // [1, 2, 3, 10, 20, 4, 5, 6]

// ใช้กับ Math
console.log(Math.max(...arr1));  // 3
console.log(Math.min(...arr2));  // 4

// ตัวอย่างที่ 79: Spread กับ Object
const defaults = { theme: 'light', lang: 'th', fontSize: 14 };
const userPrefs = { theme: 'dark', fontSize: 16 };

// merge objects (userPrefs override defaults)
const settings = { ...defaults, ...userPrefs };
console.log(settings);
// { theme: 'dark', lang: 'th', fontSize: 16 }

// copy object
const originalUser = { name: 'สมชาย', age: 25 };
const updatedUser = { ...originalUser, age: 26, job: 'Developer' };
console.log(originalUser); // { name: 'สมชาย', age: 25 } - ไม่เปลี่ยน
console.log(updatedUser);  // { name: 'สมชาย', age: 26, job: 'Developer' }
```

---

## Step 30: โปรแกรมตัวอย่างที่สมบูรณ์

```javascript
// ตัวอย่างที่ 80: ระบบจัดการนักเรียน
'use strict';

class StudentManager {
    #students = []; // Private field
    
    addStudent(name, scores) {
        const student = {
            id: this.#students.length + 1,
            name,
            scores,
            average: scores.reduce((sum, s) => sum + s, 0) / scores.length,
            grade: this.#calculateGrade(scores)
        };
        this.#students.push(student);
        return student;
    }
    
    #calculateGrade(scores) {
        const avg = scores.reduce((sum, s) => sum + s, 0) / scores.length;
        if (avg >= 90) return 'A';
        if (avg >= 80) return 'B';
        if (avg >= 70) return 'C';
        if (avg >= 60) return 'D';
        return 'F';
    }
    
    getTopStudents(n = 3) {
        return [...this.#students]
            .sort((a, b) => b.average - a.average)
            .slice(0, n);
    }
    
    getStatistics() {
        const averages = this.#students.map(s => s.average);
        return {
            count: this.#students.length,
            highest: Math.max(...averages),
            lowest: Math.min(...averages),
            classAverage: averages.reduce((sum, a) => sum + a, 0) / averages.length
        };
    }
    
    displayReport() {
        console.log('\n=== รายงานผลการเรียน ===');
        console.table(this.#students.map(s => ({
            ID: s.id,
            ชื่อ: s.name,
            คะแนนเฉลี่ย: s.average.toFixed(2),
            เกรด: s.grade
        })));
        
        const stats = this.getStatistics();
        console.log(`\nสถิติห้องเรียน:`);
        console.log(`จำนวนนักเรียน: ${stats.count} คน`);
        console.log(`คะแนนเฉลี่ยห้อง: ${stats.classAverage.toFixed(2)}`);
        console.log(`สูงสุด: ${stats.highest.toFixed(2)}, ต่ำสุด: ${stats.lowest.toFixed(2)}`);
        
        console.log('\nTop 3 นักเรียน:');
        this.getTopStudents().forEach((s, i) => {
            console.log(`${i + 1}. ${s.name} (${s.average.toFixed(2)})`);
        });
    }
}

const manager = new StudentManager();
manager.addStudent('สมชาย ใจดี', [85, 92, 78, 88, 95]);
manager.addStudent('สมหญิง สวย', [72, 68, 75, 82, 79]);
manager.addStudent('สมศักดิ์ เก่ง', [95, 98, 92, 96, 100]);
manager.addStudent('สมปอง รวย', [60, 55, 62, 58, 65]);
manager.addStudent('สมใจ รัก', [88, 85, 90, 87, 92]);

manager.displayReport();
```

```javascript
// ตัวอย่างที่ 81: Utility Functions ที่ใช้บ่อย
'use strict';

const utils = {
    // ตรวจสอบ email
    isValidEmail(email) {
        return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
    },
    
    // ตรวจสอบเบอร์โทรไทย
    isThaiPhone(phone) {
        return /^(06|08|09)\d{8}$/.test(phone.replace(/[-\s]/g, ''));
    },
    
    // Format จำนวนเงินไทย
    formatTHB(amount) {
        return new Intl.NumberFormat('th-TH', {
            style: 'currency',
            currency: 'THB'
        }).format(amount);
    },
    
    // Format วันที่ไทย
    formatThaiDate(date) {
        return new Intl.DateTimeFormat('th-TH', {
            year: 'numeric',
            month: 'long',
            day: 'numeric'
        }).format(date);
    },
    
    // Clamp ตัวเลขในช่วง
    clamp(value, min, max) {
        return Math.min(Math.max(value, min), max);
    },
    
    // Random item จาก array
    randomItem(arr) {
        return arr[Math.floor(Math.random() * arr.length)];
    },
    
    // Chunk array
    chunk(arr, size) {
        const chunks = [];
        for (let i = 0; i < arr.length; i += size) {
            chunks.push(arr.slice(i, i + size));
        }
        return chunks;
    }
};

// ทดสอบ
console.log(utils.isValidEmail('test@example.com'));    // true
console.log(utils.isValidEmail('invalid-email'));         // false
console.log(utils.isThaiPhone('0812345678'));             // true
console.log(utils.formatTHB(1299.99));                   // ฿1,299.99
console.log(utils.formatThaiDate(new Date()));           // วันปัจจุบัน
console.log(utils.clamp(15, 0, 10));                     // 10
console.log(utils.clamp(-5, 0, 10));                     // 0
console.log(utils.randomItem(['แดง', 'เขียว', 'น้ำเงิน']));
console.log(utils.chunk([1,2,3,4,5,6,7,8,9], 3));       // [[1,2,3],[4,5,6],[7,8,9]]
```

---

## สรุป Step 11-30

ในส่วนนี้คุณได้เรียนรู้:
- **var, let, const**: ความแตกต่าง scope, hoisting, และการเลือกใช้
- **String**: template literals, methods, manipulation
- **Number**: NaN, Infinity, Math object, floating point issues
- **Boolean**: truthy/falsy values
- **null, undefined**: ความแตกต่างและการตรวจสอบ
- **Symbol, BigInt**: ชนิดข้อมูลพิเศษ
- **typeof**: type checking
- **Type Conversion**: implicit, explicit, parseInt, parseFloat
- **Scope**: global, function, block, closure
- **Destructuring**: object, array
- **Spread Operator**: array, object

---

## แบบฝึกหัด

### ระดับพื้นฐาน

**แบบฝึกหัดที่ 1**: ประกาศตัวแปร
```javascript
// สร้างตัวแปรสำหรับข้อมูลสินค้า:
// - ชื่อสินค้า (ไม่เปลี่ยน)
// - ราคา (เปลี่ยนได้)
// - จำนวนในสต็อก (เปลี่ยนได้)
// - มีส่วนลดไหม (ไม่เปลี่ยน)
// แล้วแสดงผลด้วย console.log

// TODO: เขียนโค้ดของคุณที่นี่
```

**แบบฝึกหัดที่ 2**: Type Conversion
```javascript
// แปลงค่าต่อไปนี้และแสดงผล:
const inputs = ['42', '3.14', 'abc', '', true, false, null, undefined, '0'];
// แปลงแต่ละค่าเป็น Number และบอกว่าเป็น NaN หรือไม่

// TODO: เขียนโค้ดของคุณที่นี่
```

### ระดับกลาง

**แบบฝึกหัดที่ 3**: String Processing
```javascript
'use strict';
// รับข้อความต่อไปนี้:
const text = '   สวัสดี   โลก   JavaScript   ';

// ทำ:
// 1. ลบ whitespace หัวท้าย
// 2. แปลงเป็น array ของคำ
// 3. นับจำนวนคำ
// 4. เรียงตัวอักษร
// 5. รวมกลับด้วย ' | '
// TODO: เขียนโค้ดของคุณที่นี่
```

**แบบฝึกหัดที่ 4**: Destructuring
```javascript
'use strict';
const config = {
    database: {
        host: 'localhost',
        port: 5432,
        name: 'mydb',
        credentials: {
            user: 'admin',
            password: 'secret'
        }
    },
    server: {
        port: 3000,
        ssl: false
    }
};

// ใช้ destructuring เพื่อดึง: host, port(database), user, password, serverPort
// TODO: เขียนโค้ดของคุณที่นี่
```

### ระดับสูง

**แบบฝึกหัดที่ 5**: Type System
```javascript
'use strict';
// สร้าง function checkValue(value) ที่:
// - บอกชนิดข้อมูลที่แท้จริง (null, undefined, array, object, etc.)
// - บอกว่าเป็น truthy หรือ falsy
// - แสดง conversion เป็น String, Number, Boolean

// ทดสอบกับ: 0, '', null, undefined, [], {}, NaN, Infinity, 'hello', 42, true

// TODO: เขียนโค้ดของคุณที่นี่
```

---

## เฉลยแบบฝึกหัด

**เฉลยที่ 3**: String Processing
```javascript
'use strict';
const text = '   สวัสดี   โลก   JavaScript   ';

const trimmed = text.trim();
const words = trimmed.split(/\s+/);
const count = words.length;
const sorted = [...words].sort();
const result = sorted.join(' | ');

console.log('Trimmed:', trimmed);
console.log('Words:', words);
console.log('Count:', count);
console.log('Sorted:', sorted);
console.log('Result:', result);
```

**เฉลยที่ 4**: Destructuring
```javascript
'use strict';
const {
    database: {
        host,
        port: dbPort,
        credentials: { user, password }
    },
    server: { port: serverPort }
} = config;

console.log(host, dbPort, user, password, serverPort);
// localhost 5432 admin secret 3000
```

---

*ส่วนถัดไป: Part 3 - Operators (ตัวดำเนินการ)*
