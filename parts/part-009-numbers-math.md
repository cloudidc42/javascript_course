# ตอนที่ 9: Numbers และ Math Object ใน JavaScript

## บทนำ

JavaScript ใช้ชนิดข้อมูล `number` เพียงชนิดเดียวสำหรับตัวเลขทั้งหมด (ไม่แยก int/float เหมือนภาษาอื่น) โดยใช้ IEEE 754 double precision floating point ซึ่งมีความแม่นยำสูงแต่ก็มีข้อจำกัดบางอย่างที่ควรรู้

นอกจากนี้ ES2020 ยังเพิ่ม `BigInt` สำหรับตัวเลขขนาดใหญ่มากๆ

ในบทนี้เราจะครอบคลุม Steps 151-170

---

## Step 151: Number Literals

### Integer

```javascript
const decimal = 42;          // เลขฐาน 10 (ปกติ)
const thousand = 1_000_000;  // ใช้ _ แยกหลัก (ES2021)
console.log(thousand);       // 1000000

// ฐานต่างๆ
const hex = 0xFF;           // เลขฐาน 16 = 255
const binary = 0b1010;      // เลขฐาน 2 = 10
const octal = 0o17;         // เลขฐาน 8 = 15

console.log(hex);     // 255
console.log(binary);  // 10
console.log(octal);   // 15

// แสดงในฐานต่างๆ
console.log((255).toString(16));  // 'ff'
console.log((10).toString(2));    // '1010'
console.log((15).toString(8));    // '17'
```

### Float

```javascript
const pi = 3.14159265358979;
const small = 0.0001;
const negative = -99.99;

// Scientific notation
const huge = 1e6;    // 1,000,000
const tiny = 1e-6;   // 0.000001

console.log(huge);   // 1000000
console.log(tiny);   // 0.000001

const avogadro = 6.022e23;
console.log(avogadro);  // 6.022e+23
```

```javascript
// Floating point quirk!
console.log(0.1 + 0.2);           // 0.30000000000000004
console.log(0.1 + 0.2 === 0.3);   // false!

// วิธีแก้ไข
function roundToDecimal(num, decimals) {
  return Number(num.toFixed(decimals));
}

console.log(roundToDecimal(0.1 + 0.2, 1));  // 0.3
console.log(roundToDecimal(0.1 + 0.2, 1) === 0.3);  // true

// ใช้ EPSILON
function almostEqual(a, b, epsilon = Number.EPSILON) {
  return Math.abs(a - b) < epsilon;
}

console.log(almostEqual(0.1 + 0.2, 0.3));  // true
```

---

## Step 152: Number Properties

```javascript
// Number.MAX_VALUE - ตัวเลขที่ใหญ่ที่สุด
console.log(Number.MAX_VALUE);    // 1.7976931348623157e+308
console.log(Number.MIN_VALUE);    // 5e-324 (ค่าบวกที่เล็กที่สุด ไม่ใช่ค่าลบ!)

// Infinity
console.log(Number.POSITIVE_INFINITY);  // Infinity
console.log(Number.NEGATIVE_INFINITY);  // -Infinity

console.log(1 / 0);    // Infinity
console.log(-1 / 0);   // -Infinity
console.log(Infinity + 1);  // Infinity
console.log(Infinity - Infinity);  // NaN

// NaN
console.log(Number.NaN);         // NaN
console.log(NaN === NaN);         // false! (NaN ไม่เท่ากับ NaN)
console.log(isNaN(NaN));          // true
console.log(Number.isNaN(NaN));   // true (แม่นยำกว่า)

// Safe Integer range
console.log(Number.MAX_SAFE_INTEGER);  // 9007199254740991 (2^53 - 1)
console.log(Number.MIN_SAFE_INTEGER);  // -9007199254740991

// EPSILON - ค่าความแตกต่างที่เล็กที่สุด
console.log(Number.EPSILON);           // 2.220446049250313e-16
```

---

## Step 153: Number Methods - toString, toFixed, toPrecision

### toString()

```javascript
const num = 255;

console.log(num.toString());     // '255' (ฐาน 10)
console.log(num.toString(2));    // '11111111' (ฐาน 2)
console.log(num.toString(8));    // '377' (ฐาน 8)
console.log(num.toString(16));   // 'ff' (ฐาน 16)

// ใช้กับตัวเลขลบ
console.log((-255).toString(16));  // '-ff'

// แปลง binary string
console.log(parseInt('11111111', 2));  // 255
console.log(parseInt('ff', 16));       // 255
```

### toFixed()

```javascript
const pi = 3.14159265358979;

console.log(pi.toFixed(0));   // '3'
console.log(pi.toFixed(2));   // '3.14'
console.log(pi.toFixed(4));   // '3.1416' (ปัดเศษ!)
console.log(pi.toFixed(10));  // '3.1415926536'

// ระวัง: toFixed คืน string ไม่ใช่ number!
const result = pi.toFixed(2);
console.log(typeof result);  // 'string'
console.log(Number(result));  // 3.14

// ใช้งานจริง: ราคาสินค้า
function formatPrice(price) {
  return `฿${price.toFixed(2)}`;
}

console.log(formatPrice(29.9));    // '฿29.90'
console.log(formatPrice(1234.5));  // '฿1234.50'
```

### toPrecision()

```javascript
const num = 123.456789;

console.log(num.toPrecision(4));  // '123.5' (4 หลักทั้งหมด)
console.log(num.toPrecision(7));  // '123.4568' (7 หลักทั้งหมด)
console.log(num.toPrecision(2));  // '1.2e+2' (scientific)
console.log(num.toPrecision(10)); // '123.4567890'

// เปรียบเทียบ toFixed vs toPrecision
const small = 0.00123456;
console.log(small.toFixed(4));      // '0.0012' (4 ตำแหน่งหลังทศนิยม)
console.log(small.toPrecision(4));  // '0.001235' (4 หลักที่มีนัยสำคัญ)
```

### toLocaleString()

```javascript
const num = 1234567.89;

console.log(num.toLocaleString());         // ตามระบบ
console.log(num.toLocaleString('th-TH'));  // '1,234,567.89'
console.log(num.toLocaleString('en-US'));  // '1,234,567.89'
console.log(num.toLocaleString('de-DE'));  // '1.234.567,89'

// สกุลเงิน
console.log(num.toLocaleString('th-TH', {
  style: 'currency',
  currency: 'THB'
}));  // '฿1,234,567.89'

console.log(num.toLocaleString('en-US', {
  style: 'currency',
  currency: 'USD'
}));  // '$1,234,567.89'

// เปอร์เซ็นต์
console.log((0.75).toLocaleString('th-TH', {
  style: 'percent'
}));  // '75%'
```

---

## Step 154: Number Type Checking Methods

### isNaN() vs Number.isNaN()

```javascript
// isNaN() global - แปลงเป็น number ก่อน แล้วค่อยตรวจ
console.log(isNaN(NaN));          // true
console.log(isNaN('hello'));      // true (แปลง 'hello' เป็น NaN ก่อน)
console.log(isNaN('123'));        // false (แปลงเป็น 123 ได้)
console.log(isNaN(undefined));    // true

// Number.isNaN() - ไม่แปลง ตรวจตรงๆ
console.log(Number.isNaN(NaN));       // true
console.log(Number.isNaN('hello'));   // false! (string ไม่ใช่ NaN)
console.log(Number.isNaN(undefined)); // false

// ใช้ Number.isNaN() จะแม่นยำกว่า
function safeParseFloat(str) {
  const num = parseFloat(str);
  return Number.isNaN(num) ? null : num;
}

console.log(safeParseFloat('3.14'));   // 3.14
console.log(safeParseFloat('abc'));    // null
console.log(safeParseFloat(''));       // null
```

### isFinite() และ Number.isFinite()

```javascript
// isFinite() global
console.log(isFinite(42));        // true
console.log(isFinite(Infinity));  // false
console.log(isFinite(NaN));       // false
console.log(isFinite('42'));      // true (แปลงก่อน)

// Number.isFinite() - ไม่แปลง
console.log(Number.isFinite(42));    // true
console.log(Number.isFinite('42')); // false (string ไม่ใช่ finite number)

// ใช้งานจริง: ตรวจสอบผลลัพธ์การคำนวณ
function safeDivide(a, b) {
  if (b === 0) return null;
  const result = a / b;
  return Number.isFinite(result) ? result : null;
}

console.log(safeDivide(10, 2));    // 5
console.log(safeDivide(10, 0));    // null
console.log(safeDivide(Infinity, 1)); // null (ถ้าต้องการ)
```

### isInteger() และ isSafeInteger()

```javascript
console.log(Number.isInteger(42));      // true
console.log(Number.isInteger(42.0));    // true (42.0 คือ 42)
console.log(Number.isInteger(42.5));    // false
console.log(Number.isInteger('42'));    // false
console.log(Number.isInteger(NaN));     // false

// isSafeInteger: อยู่ใน safe range?
console.log(Number.isSafeInteger(42));            // true
console.log(Number.isSafeInteger(9007199254740992)); // false (เกิน MAX_SAFE_INTEGER)
console.log(Number.isSafeInteger(9007199254740991)); // true

// ตัวอย่าง: ตรวจสอบ ID
function isValidId(id) {
  return Number.isSafeInteger(id) && id > 0;
}

console.log(isValidId(42));                   // true
console.log(isValidId(-1));                   // false
console.log(isValidId(9007199254740992));      // false
```

---

## Step 155: parseInt() และ parseFloat()

```javascript
// parseInt(string, radix)
console.log(parseInt('42'));        // 42
console.log(parseInt('42.7'));      // 42 (ตัดทศนิยม)
console.log(parseInt('42abc'));     // 42 (หยุดที่ตัวอักษรไม่ใช่ตัวเลข)
console.log(parseInt('abc'));       // NaN
console.log(parseInt(''));          // NaN
console.log(parseInt('0x1F', 16)); // 31 (hex)
console.log(parseInt('1010', 2));  // 10 (binary)
console.log(parseInt('17', 8));    // 15 (octal)
```

```javascript
// ระวัง! parseInt ไม่ได้ผลกับ scientific notation
console.log(parseInt('1e3'));   // 1 (ไม่ใช่ 1000!)
console.log(parseFloat('1e3')); // 1000

// parseFloat
console.log(parseFloat('3.14'));      // 3.14
console.log(parseFloat('3.14abc'));   // 3.14
console.log(parseFloat('.5'));        // 0.5
console.log(parseFloat('1e3'));       // 1000
console.log(parseFloat('abc'));       // NaN
```

```javascript
// ใช้งานจริง: parse user input
function parseUserInput(input) {
  const num = parseFloat(input);
  
  if (Number.isNaN(num)) {
    return { error: 'กรุณาระบุตัวเลข', value: null };
  }
  
  return { error: null, value: num };
}

console.log(parseUserInput('42.5'));   // { error: null, value: 42.5 }
console.log(parseUserInput('abc'));    // { error: 'กรุณาระบุตัวเลข', value: null }
console.log(parseUserInput(''));       // { error: 'กรุณาระบุตัวเลข', value: null }
```

```javascript
// แปลงตัวเลขจากหน่วย
function parseWithUnit(str) {
  const match = str.match(/^([\d.]+)\s*(kb|mb|gb|tb)?$/i);
  if (!match) return null;
  
  const num = parseFloat(match[1]);
  const unit = (match[2] || '').toLowerCase();
  
  const multipliers = { kb: 1024, mb: 1024**2, gb: 1024**3, tb: 1024**4 };
  return num * (multipliers[unit] || 1);
}

console.log(parseWithUnit('1.5 GB'));  // 1610612736
console.log(parseWithUnit('512 MB'));  // 536870912
console.log(parseWithUnit('100'));     // 100
```

---

## Step 156: BigInt

```javascript
// BigInt สำหรับตัวเลขใหญ่มากๆ
const big = 9007199254740991n;  // เติม n ที่ท้าย
console.log(big + 1n);  // 9007199254740992n (ถูกต้อง!)

// เปรียบเทียบกับ regular number
const maxSafe = Number.MAX_SAFE_INTEGER;
console.log(maxSafe + 1 === maxSafe + 2);  // true (ผิดพลาด!)
console.log(9007199254740991n + 1n === 9007199254740991n + 2n);  // false (ถูกต้อง!)
```

```javascript
// สร้าง BigInt
const n1 = 42n;
const n2 = BigInt(42);
const n3 = BigInt('9007199254740993');

console.log(n1 === n2);  // true
console.log(n3);  // 9007199254740993n

// operations
console.log(100n + 200n);    // 300n
console.log(100n * 200n);    // 20000n
console.log(100n / 3n);      // 33n (integer division, ไม่มีทศนิยม)
console.log(100n % 3n);      // 1n
console.log(2n ** 100n);     // ค่าใหญ่มาก

// ผสมกับ number ไม่ได้!
// console.log(100n + 200);  // TypeError!
console.log(Number(100n) + 200);  // 300 (แปลงก่อน)
```

```javascript
// ใช้งานจริง: คำนวณ factorial ใหญ่
function factorial(n) {
  let result = 1n;
  for (let i = 2n; i <= BigInt(n); i++) {
    result *= i;
  }
  return result;
}

console.log(factorial(20));   // 2432902008176640000n
console.log(factorial(50));   // ตัวเลขขนาดใหญ่มาก
console.log(factorial(100));  // ใหญ่มากๆๆๆ

// UUID generation (ตัวอย่าง)
function generateId() {
  return (BigInt(Date.now()) * 1000000n + BigInt(Math.floor(Math.random() * 1000000))).toString();
}
```

---

## Step 157: Math Object - Constants

```javascript
console.log(Math.PI);     // 3.141592653589793
console.log(Math.E);      // 2.718281828459045 (Euler's number)
console.log(Math.LN2);    // 0.6931471805599453 (ln(2))
console.log(Math.LN10);   // 2.302585092994046 (ln(10))
console.log(Math.LOG2E);  // 1.4426950408889634
console.log(Math.LOG10E); // 0.4342944819032518
console.log(Math.SQRT2);  // 1.4142135623730951 (√2)
console.log(Math.SQRT1_2); // 0.7071067811865476 (1/√2)

// ใช้ค่าคงที่
const circumference = 2 * Math.PI * 5;  // เส้นรอบวงรัศมี 5
console.log(circumference.toFixed(4));   // '31.4159'

const area = Math.PI * 5 ** 2;          // พื้นที่วงกลมรัศมี 5
console.log(area.toFixed(4));            // '78.5398'
```

---

## Step 158: Math Methods - abs, ceil, floor, round, trunc

### abs()

```javascript
console.log(Math.abs(5));    // 5
console.log(Math.abs(-5));   // 5
console.log(Math.abs(-3.7)); // 3.7
console.log(Math.abs(0));    // 0

// ใช้งาน: คำนวณระยะห่าง
function distance(a, b) {
  return Math.abs(a - b);
}
console.log(distance(3, 7));   // 4
console.log(distance(7, 3));   // 4
console.log(distance(-5, 5));  // 10
```

### ceil(), floor(), round(), trunc()

```javascript
const num = 4.7;
const neg = -4.7;

// ceil: ปัดขึ้น
console.log(Math.ceil(4.1));   // 5
console.log(Math.ceil(4.9));   // 5
console.log(Math.ceil(-4.1));  // -4 (ปัดขึ้น = น้อยลงสำหรับค่าลบ)
console.log(Math.ceil(-4.9));  // -4

// floor: ปัดลง
console.log(Math.floor(4.9));  // 4
console.log(Math.floor(4.1));  // 4
console.log(Math.floor(-4.1)); // -5 (ปัดลง = มากขึ้นสำหรับค่าลบ)
console.log(Math.floor(-4.9)); // -5

// round: ปัดเศษปกติ
console.log(Math.round(4.4));  // 4
console.log(Math.round(4.5));  // 5
console.log(Math.round(-4.5)); // -4 (round towards positive infinity)
console.log(Math.round(4.6));  // 5

// trunc: ตัดทศนิยมออก (ไม่ปัด)
console.log(Math.trunc(4.9));  // 4
console.log(Math.trunc(4.1));  // 4
console.log(Math.trunc(-4.9)); // -4 (ต่างจาก floor!)
console.log(Math.trunc(-4.1)); // -4
```

```javascript
// ปัดเศษหลายตำแหน่ง
function round(num, decimals) {
  const factor = 10 ** decimals;
  return Math.round(num * factor) / factor;
}

console.log(round(1.2345, 2));   // 1.23
console.log(round(1.235, 2));    // 1.24 (แต่อาจมี floating point issue)
console.log(round(1.2355, 3));   // 1.236

// วิธีที่แม่นยำกว่า
function roundPrecise(num, decimals) {
  return Number(Math.round(num + 'e' + decimals) + 'e-' + decimals);
}

console.log(roundPrecise(1.005, 2));  // 1.01 (ถูกต้อง)
```

---

## Step 159: Math.max(), Math.min(), Math.random()

### max() และ min()

```javascript
console.log(Math.max(1, 3, 2, 5, 4));  // 5
console.log(Math.min(1, 3, 2, 5, 4));  // 1

// ไม่มี argument
console.log(Math.max());  // -Infinity
console.log(Math.min());  // Infinity

// ใช้กับ array
const scores = [85, 92, 78, 95, 88];
console.log(Math.max(...scores));  // 95
console.log(Math.min(...scores));  // 78

// ใช้ apply
console.log(Math.max.apply(null, scores));  // 95

// วิธีอื่น
const maxScore = scores.reduce((max, n) => n > max ? n : max, -Infinity);
console.log(maxScore);  // 95
```

```javascript
// clamp: จำกัดค่าให้อยู่ในช่วง
function clamp(value, min, max) {
  return Math.min(Math.max(value, min), max);
}

console.log(clamp(5, 0, 10));   // 5 (อยู่ในช่วง)
console.log(clamp(-5, 0, 10));  // 0 (ต่ำเกิน)
console.log(clamp(15, 0, 10));  // 10 (สูงเกิน)

// ใช้งาน: ปรับ opacity
const opacity = clamp(userInput / 100, 0, 1);
```

### random()

```javascript
// Math.random() คืนค่า [0, 1) 
console.log(Math.random());  // เช่น 0.7234...

// สุ่มในช่วง [min, max)
function random(min, max) {
  return Math.random() * (max - min) + min;
}

// สุ่มจำนวนเต็มในช่วง [min, max]
function randomInt(min, max) {
  return Math.floor(Math.random() * (max - min + 1)) + min;
}

console.log(randomInt(1, 6));   // ลูกเต๋า: 1-6
console.log(randomInt(0, 100)); // 0-100
```

```javascript
// สุ่มจาก array
function randomFrom(arr) {
  return arr[randomInt(0, arr.length - 1)];
}

const fruits = ['แอปเปิ้ล', 'กล้วย', 'ส้ม', 'มะม่วง'];
console.log(randomFrom(fruits));

// สุ่มหลายตัวไม่ซ้ำ (Fisher-Yates shuffle)
function shuffle(arr) {
  const result = [...arr];
  for (let i = result.length - 1; i > 0; i--) {
    const j = randomInt(0, i);
    [result[i], result[j]] = [result[j], result[i]];
  }
  return result;
}

console.log(shuffle([1, 2, 3, 4, 5]));

// สุ่ม n ตัวจาก array
function sample(arr, n) {
  return shuffle(arr).slice(0, n);
}

console.log(sample([1, 2, 3, 4, 5, 6, 7, 8, 9, 10], 3));
```

```javascript
// สุ่มสีแบบ hex
function randomColor() {
  return '#' + Math.floor(Math.random() * 16777215).toString(16).padStart(6, '0');
}

console.log(randomColor());  // เช่น '#a3f5c8'

// สุ่ม UUID v4 (simplified)
function simpleUUID() {
  return 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, c => {
    const r = Math.random() * 16 | 0;
    const v = c === 'x' ? r : (r & 0x3 | 0x8);
    return v.toString(16);
  });
}

console.log(simpleUUID());  // เช่น 'a3b4c5d6-e7f8-4a9b-8c7d-6e5f4a3b2c1d'
```

---

## Step 160: Math.pow(), Math.sqrt(), Math.cbrt()

### pow() - ยกกำลัง

```javascript
console.log(Math.pow(2, 10));  // 1024
console.log(Math.pow(2, -1));  // 0.5
console.log(Math.pow(9, 0.5)); // 3 (ถอดรากที่สอง)

// ES7: ** operator
console.log(2 ** 10);   // 1024
console.log(9 ** 0.5);  // 3

// ใช้งาน
function compound(principal, rate, years) {
  return principal * Math.pow(1 + rate, years);
}

console.log(compound(10000, 0.05, 10).toFixed(2));  // '16288.95'
```

### sqrt() และ cbrt()

```javascript
console.log(Math.sqrt(9));    // 3
console.log(Math.sqrt(2));    // 1.4142135623730951
console.log(Math.sqrt(-1));   // NaN

console.log(Math.cbrt(27));   // 3
console.log(Math.cbrt(8));    // 2
console.log(Math.cbrt(-8));   // -2

// Pythagorean theorem
function hypotenuse(a, b) {
  return Math.sqrt(a ** 2 + b ** 2);
}

console.log(hypotenuse(3, 4));  // 5
console.log(hypotenuse(5, 12)); // 13

// Math.hypot (built-in!)
console.log(Math.hypot(3, 4));  // 5
console.log(Math.hypot(3, 4, 5)); // ระยะใน 3D = 7.0710...

// ระยะระหว่างสองจุด
function distance2D(x1, y1, x2, y2) {
  return Math.hypot(x2 - x1, y2 - y1);
}

console.log(distance2D(0, 0, 3, 4));   // 5
console.log(distance2D(1, 1, 4, 5));   // 5
```

---

## Step 161: Trigonometry

```javascript
// Math ใช้ radian ไม่ใช่ degree!
// แปลง: degrees * Math.PI / 180

function degToRad(deg) {
  return deg * Math.PI / 180;
}

function radToDeg(rad) {
  return rad * 180 / Math.PI;
}

// sin, cos, tan
console.log(Math.sin(0));                    // 0
console.log(Math.sin(Math.PI / 2));          // 1 (sin 90°)
console.log(Math.sin(degToRad(30)));         // 0.5 (sin 30°)
console.log(Math.cos(0));                    // 1
console.log(Math.cos(Math.PI));              // -1 (cos 180°)
console.log(Math.tan(degToRad(45)));         // ~1 (tan 45°)

// inverse trig
console.log(radToDeg(Math.asin(1)));         // 90
console.log(radToDeg(Math.acos(0)));         // 90
console.log(radToDeg(Math.atan(1)));         // 45
console.log(radToDeg(Math.atan2(1, 1)));     // 45 (atan2 มีประโยชน์กว่า)
```

```javascript
// ใช้งานจริง: วงกลม
function circlePoint(cx, cy, radius, angleDeg) {
  const rad = degToRad(angleDeg);
  return {
    x: cx + radius * Math.cos(rad),
    y: cy + radius * Math.sin(rad)
  };
}

// จุดบน clock
for (let hour = 1; hour <= 12; hour++) {
  const angle = hour * 30 - 90;  // -90 เพื่อเริ่มจากบน
  const point = circlePoint(0, 0, 1, angle);
  console.log(`${hour}: (${point.x.toFixed(2)}, ${point.y.toFixed(2)})`);
}
```

```javascript
// คำนวณมุมระหว่างสองจุด
function angleBetweenPoints(x1, y1, x2, y2) {
  return radToDeg(Math.atan2(y2 - y1, x2 - x1));
}

console.log(angleBetweenPoints(0, 0, 1, 0));   // 0
console.log(angleBetweenPoints(0, 0, 0, 1));   // 90
console.log(angleBetweenPoints(0, 0, -1, 0));  // 180
console.log(angleBetweenPoints(0, 0, 1, 1));   // 45
```

---

## Step 162: Math Logarithm Functions

### log(), log2(), log10()

```javascript
// Natural log (ln)
console.log(Math.log(Math.E));  // 1 (ln(e) = 1)
console.log(Math.log(1));       // 0 (ln(1) = 0)
console.log(Math.log(10));      // 2.302585...

// Log base 2
console.log(Math.log2(8));      // 3 (2^3 = 8)
console.log(Math.log2(1024));   // 10 (2^10 = 1024)

// Log base 10
console.log(Math.log10(100));   // 2 (10^2 = 100)
console.log(Math.log10(1000));  // 3

// log ฐานอื่น
function logBase(base, x) {
  return Math.log(x) / Math.log(base);
}

console.log(logBase(3, 27));    // 3 (3^3 = 27)
console.log(logBase(5, 125));   // 3 (5^3 = 125)
```

```javascript
// ใช้งาน log
// จำนวน binary digits ที่ต้องการ
function bitsNeeded(n) {
  return Math.floor(Math.log2(n)) + 1;
}

console.log(bitsNeeded(1));     // 1 bit (แทน 1)
console.log(bitsNeeded(7));     // 3 bits (แทน 0-7)
console.log(bitsNeeded(255));   // 8 bits
console.log(bitsNeeded(1023));  // 10 bits

// Decibels
function linearToDecibels(linear) {
  return 20 * Math.log10(linear);
}

function decibelsToLinear(db) {
  return Math.pow(10, db / 20);
}

console.log(linearToDecibels(1));     // 0 dB
console.log(linearToDecibels(10));    // 20 dB
console.log(linearToDecibels(100));   // 40 dB
```

```javascript
// Math.log1p และ Math.expm1 (precision สำหรับค่าเล็กๆ)
// log1p(x) = log(1 + x) ไม่สูญเสีย precision
console.log(Math.log1p(0.0000001));  // 9.999999950000001e-8 (แม่นยำ)
console.log(Math.log(1 + 0.0000001)); // 9.99999995e-8 (อาจมีข้อผิดพลาด)

// expm1(x) = e^x - 1 ไม่สูญเสีย precision
console.log(Math.expm1(0.0000001));  // 1.00000000500000004e-7
```

---

## Step 163: Math รวมเบ็ดเตล็ด

```javascript
// Math.sign - คืน 1, -1, หรือ 0
console.log(Math.sign(5));    // 1
console.log(Math.sign(-5));   // -1
console.log(Math.sign(0));    // 0
console.log(Math.sign(NaN));  // NaN

// ใช้งาน
function directionFromCenter(x) {
  return Math.sign(x) === 1 ? 'ขวา' : Math.sign(x) === -1 ? 'ซ้าย' : 'กลาง';
}
```

```javascript
// Math.fround - แปลงเป็น float32
console.log(Math.fround(1.337));    // 1.3370000123977661 (precision ต่างกัน)

// Math.clz32 - นับ leading zeros ใน 32-bit
console.log(Math.clz32(1));   // 31
console.log(Math.clz32(4));   // 29

// Math.imul - 32-bit integer multiplication
console.log(Math.imul(3, 4));  // 12

// Bitwise operations
console.log(5 & 3);   // 1 (AND)
console.log(5 | 3);   // 7 (OR)
console.log(5 ^ 3);   // 6 (XOR)
console.log(~5);       // -6 (NOT)
console.log(5 << 1);  // 10 (left shift)
console.log(5 >> 1);  // 2 (right shift)
```

---

## Step 164: ตัวอย่างการคำนวณจริง

### การเงิน

```javascript
// คำนวณดอกเบี้ยทบต้น
function compoundInterest(principal, annualRate, compoundsPerYear, years) {
  const rate = annualRate / compoundsPerYear;
  const periods = compoundsPerYear * years;
  return principal * Math.pow(1 + rate, periods);
}

const p = 10000;  // เงินต้น
const r = 0.05;   // ดอกเบี้ย 5% ต่อปี
console.log(`รายปี:    ${compoundInterest(p, r, 1, 10).toFixed(2)}`);
console.log(`รายเดือน: ${compoundInterest(p, r, 12, 10).toFixed(2)}`);
console.log(`รายวัน:   ${compoundInterest(p, r, 365, 10).toFixed(2)}`);
```

```javascript
// คำนวณผ่อนชำระ (mortgage)
function monthlyPayment(principal, annualRate, months) {
  const r = annualRate / 12;
  if (r === 0) return principal / months;
  return principal * r * Math.pow(1 + r, months) / (Math.pow(1 + r, months) - 1);
}

const loan = 1000000;    // กู้ 1 ล้าน
const rate = 0.05;       // ดอกเบี้ย 5%
const years = 30;        // 30 ปี

const monthly = monthlyPayment(loan, rate, years * 12);
console.log(`ผ่อนต่อเดือน: ${monthly.toFixed(2)} บาท`);
console.log(`รวมทั้งหมด: ${(monthly * years * 12).toFixed(2)} บาท`);
console.log(`ดอกเบี้ยรวม: ${(monthly * years * 12 - loan).toFixed(2)} บาท`);
```

---

## Step 165: Statistics Functions

```javascript
// สถิติพื้นฐาน
class Statistics {
  static mean(data) {
    return data.reduce((sum, n) => sum + n, 0) / data.length;
  }
  
  static median(data) {
    const sorted = [...data].sort((a, b) => a - b);
    const mid = Math.floor(sorted.length / 2);
    return sorted.length % 2 !== 0
      ? sorted[mid]
      : (sorted[mid - 1] + sorted[mid]) / 2;
  }
  
  static mode(data) {
    const freq = {};
    data.forEach(n => freq[n] = (freq[n] || 0) + 1);
    const maxFreq = Math.max(...Object.values(freq));
    return Object.entries(freq)
      .filter(([, count]) => count === maxFreq)
      .map(([num]) => Number(num));
  }
  
  static variance(data) {
    const m = Statistics.mean(data);
    return data.reduce((sum, n) => sum + (n - m) ** 2, 0) / data.length;
  }
  
  static stdDev(data) {
    return Math.sqrt(Statistics.variance(data));
  }
  
  static range(data) {
    return Math.max(...data) - Math.min(...data);
  }
  
  static percentile(data, p) {
    const sorted = [...data].sort((a, b) => a - b);
    const index = (p / 100) * (sorted.length - 1);
    const lower = Math.floor(index);
    const upper = Math.ceil(index);
    return lower === upper
      ? sorted[lower]
      : sorted[lower] + (sorted[upper] - sorted[lower]) * (index - lower);
  }
  
  static summary(data) {
    return {
      count: data.length,
      mean: Statistics.mean(data),
      median: Statistics.median(data),
      mode: Statistics.mode(data),
      stdDev: Statistics.stdDev(data),
      min: Math.min(...data),
      max: Math.max(...data),
      range: Statistics.range(data),
      q1: Statistics.percentile(data, 25),
      q3: Statistics.percentile(data, 75)
    };
  }
}

const scores = [85, 92, 78, 95, 88, 72, 90, 85, 88, 76];
const stats = Statistics.summary(scores);

console.log(`ค่าเฉลี่ย: ${stats.mean.toFixed(2)}`);
console.log(`ค่ากลาง: ${stats.median}`);
console.log(`ฐานนิยม: ${stats.mode}`);
console.log(`ส่วนเบี่ยงเบนมาตรฐาน: ${stats.stdDev.toFixed(2)}`);
console.log(`Percentile 75: ${stats.q3}`);
```

---

## Step 166: ตัวอย่าง - จำนวนเฉพาะ

```javascript
// ตรวจสอบจำนวนเฉพาะ
function isPrime(n) {
  if (n < 2) return false;
  if (n === 2) return true;
  if (n % 2 === 0) return false;
  
  for (let i = 3; i <= Math.sqrt(n); i += 2) {
    if (n % i === 0) return false;
  }
  return true;
}

// Sieve of Eratosthenes
function sieve(limit) {
  const primes = new Array(limit + 1).fill(true);
  primes[0] = primes[1] = false;
  
  for (let i = 2; i <= Math.sqrt(limit); i++) {
    if (primes[i]) {
      for (let j = i * i; j <= limit; j += i) {
        primes[j] = false;
      }
    }
  }
  
  return primes.reduce((arr, isPrime, num) => {
    if (isPrime) arr.push(num);
    return arr;
  }, []);
}

console.log(isPrime(17));    // true
console.log(isPrime(18));    // false
console.log(sieve(50));      // [2, 3, 5, 7, 11, 13, 17, 19, 23, 29, 31, 37, 41, 43, 47]
```

---

## Step 167: Number Formatting ขั้นสูง

```javascript
// Intl.NumberFormat
const formatter = new Intl.NumberFormat('th-TH', {
  style: 'currency',
  currency: 'THB',
  minimumFractionDigits: 2,
  maximumFractionDigits: 2
});

console.log(formatter.format(1234567.89));  // '฿1,234,567.89'

// format หลายค่า
const prices = [299, 1499.5, 9999];
prices.forEach(p => console.log(formatter.format(p)));

// ตัวเลขแบบไทย
const thaiFormatter = new Intl.NumberFormat('th-TH-u-nu-thai');
console.log(thaiFormatter.format(1234));  // '๑,๒๓๔'
```

```javascript
// formatNumber function
function formatNumber(num, options = {}) {
  const {
    locale = 'th-TH',
    style = 'decimal',
    currency = 'THB',
    minimumFractionDigits = 0,
    maximumFractionDigits = 2
  } = options;
  
  return new Intl.NumberFormat(locale, {
    style,
    currency: style === 'currency' ? currency : undefined,
    minimumFractionDigits,
    maximumFractionDigits
  }).format(num);
}

console.log(formatNumber(1234567));                          // '1,234,567'
console.log(formatNumber(1234.5, { minimumFractionDigits: 2 })); // '1,234.50'
console.log(formatNumber(0.75, { style: 'percent' }));       // '75%'
console.log(formatNumber(1234, { style: 'currency' }));      // '฿1,234.00'
```

---

## Step 168: Geometry Calculations

```javascript
// คำนวณรูปทรงต่างๆ
const Geometry = {
  // วงกลม
  circle: {
    area: r => Math.PI * r ** 2,
    circumference: r => 2 * Math.PI * r,
    diameter: r => 2 * r
  },
  
  // สามเหลี่ยม
  triangle: {
    area: (base, height) => 0.5 * base * height,
    areaByHeron: (a, b, c) => {
      const s = (a + b + c) / 2;
      return Math.sqrt(s * (s - a) * (s - b) * (s - c));
    },
    hypotenuse: (a, b) => Math.sqrt(a ** 2 + b ** 2)
  },
  
  // สี่เหลี่ยม
  rectangle: {
    area: (w, h) => w * h,
    perimeter: (w, h) => 2 * (w + h),
    diagonal: (w, h) => Math.sqrt(w ** 2 + h ** 2)
  },
  
  // ทรงกลม
  sphere: {
    volume: r => (4/3) * Math.PI * r ** 3,
    surfaceArea: r => 4 * Math.PI * r ** 2
  },
  
  // ทรงกระบอก
  cylinder: {
    volume: (r, h) => Math.PI * r ** 2 * h,
    surfaceArea: (r, h) => 2 * Math.PI * r * (r + h)
  }
};

const r = 5;
console.log(`พื้นที่วงกลม: ${Geometry.circle.area(r).toFixed(2)}`);
console.log(`ปริมาตรทรงกลม: ${Geometry.sphere.volume(r).toFixed(2)}`);
console.log(`พื้นที่ผิวทรงกลม: ${Geometry.sphere.surfaceArea(r).toFixed(2)}`);
```

---

## Step 169: Numerical Algorithms

```javascript
// Newton's Method หาค่า sqrt
function newtonSqrt(n, tolerance = 1e-10) {
  let x = n / 2;  // initial guess
  
  while (Math.abs(x * x - n) > tolerance) {
    x = (x + n / x) / 2;
  }
  
  return x;
}

console.log(newtonSqrt(2));   // 1.4142135623730951
console.log(Math.sqrt(2));    // 1.4142135623730951

// Binary search สำหรับ sqrt
function binarySqrt(n) {
  let low = 0, high = n;
  
  while (high - low > 1e-10) {
    const mid = (low + high) / 2;
    if (mid * mid < n) low = mid;
    else high = mid;
  }
  
  return (low + high) / 2;
}
```

```javascript
// GCD (Greatest Common Divisor)
function gcd(a, b) {
  a = Math.abs(a);
  b = Math.abs(b);
  while (b !== 0) {
    [a, b] = [b, a % b];
  }
  return a;
}

// LCM (Least Common Multiple)
function lcm(a, b) {
  return Math.abs(a * b) / gcd(a, b);
}

console.log(gcd(48, 18));   // 6
console.log(lcm(4, 6));     // 12
console.log(lcm(12, 18));   // 36

// เศษส่วน
class Fraction {
  constructor(num, den) {
    const g = gcd(Math.abs(num), Math.abs(den));
    this.num = num / g * Math.sign(den);
    this.den = Math.abs(den) / g;
  }
  
  add(other) {
    return new Fraction(
      this.num * other.den + other.num * this.den,
      this.den * other.den
    );
  }
  
  toString() {
    return this.den === 1 ? `${this.num}` : `${this.num}/${this.den}`;
  }
}

const half = new Fraction(1, 2);
const third = new Fraction(1, 3);
console.log(half.add(third).toString());  // '5/6'
```

---

## Step 170: ตัวอย่างโปรเจกต์ - Calculator

```javascript
class ScientificCalculator {
  #history = [];
  #memory = 0;
  
  // Basic operations
  add(a, b) { return this.#record(`${a} + ${b}`, a + b); }
  subtract(a, b) { return this.#record(`${a} - ${b}`, a - b); }
  multiply(a, b) { return this.#record(`${a} × ${b}`, a * b); }
  divide(a, b) {
    if (b === 0) throw new Error('ไม่สามารถหารด้วยศูนย์ได้');
    return this.#record(`${a} ÷ ${b}`, a / b);
  }
  
  // Scientific
  power(base, exp) { return this.#record(`${base}^${exp}`, Math.pow(base, exp)); }
  sqrt(n) { return this.#record(`√${n}`, Math.sqrt(n)); }
  log(n) { return this.#record(`ln(${n})`, Math.log(n)); }
  log10(n) { return this.#record(`log(${n})`, Math.log10(n)); }
  sin(deg) { return this.#record(`sin(${deg}°)`, Math.sin(deg * Math.PI / 180)); }
  cos(deg) { return this.#record(`cos(${deg}°)`, Math.cos(deg * Math.PI / 180)); }
  tan(deg) { return this.#record(`tan(${deg}°)`, Math.tan(deg * Math.PI / 180)); }
  factorial(n) {
    if (!Number.isInteger(n) || n < 0) throw new Error('ต้องเป็นจำนวนเต็มบวก');
    const result = n <= 1 ? 1 : Array.from({length: n}, (_, i) => i + 1).reduce((p, x) => p * x, 1);
    return this.#record(`${n}!`, result);
  }
  
  // Memory
  memStore(value) { this.#memory = value; return value; }
  memRecall() { return this.#memory; }
  memClear() { this.#memory = 0; }
  
  // History
  #record(expression, result) {
    this.#history.push({ expression, result, time: new Date() });
    return result;
  }
  
  getHistory() { return [...this.#history]; }
  clearHistory() { this.#history = []; }
  
  getLastResult() {
    return this.#history.length > 0 ? this.#history[this.#history.length - 1].result : null;
  }
}

const calc = new ScientificCalculator();

console.log(calc.add(10, 5));       // 15
console.log(calc.multiply(3, 4));   // 12
console.log(calc.sqrt(144));         // 12
console.log(calc.sin(30));           // 0.5
console.log(calc.factorial(5));      // 120
console.log(calc.power(2, 10));      // 1024

console.log('\nประวัติการคำนวณ:');
calc.getHistory().forEach(({ expression, result }) => {
  console.log(`  ${expression} = ${result}`);
});
```

---

## แบบฝึกหัด (Exercises)

### ระดับง่าย

**แบบฝึกหัด 1:** เขียน function `roundToNearest(n, nearest)` เช่น roundToNearest(17, 5) = 15

**แบบฝึกหัด 2:** เขียน function `celsiusToFahrenheit(c)` และ `fahrenheitToCelsius(f)`

**แบบฝึกหัด 3:** เขียน function `isPerfectSquare(n)` ตรวจสอบว่าเป็นกำลังสองสมบูรณ์

**แบบฝึกหัด 4:** เขียน function `digitSum(n)` บวกเลขทุกหลักของ n เช่น 123 → 6

**แบบฝึกหัด 5:** เขียน function `fibonacci(n)` คืน fibonacci ตำแหน่งที่ n

### ระดับกลาง

**แบบฝึกหัด 6:** สร้าง `RomanNumeral` converter ที่แปลง integer เป็น Roman numeral และกลับกัน

**แบบฝึกหัด 7:** เขียน function `formatBytes(bytes)` แสดงขนาดไฟล์ในหน่วยที่เหมาะสม (B, KB, MB, GB)

**แบบฝึกหัด 8:** เขียน function `interpolateColor(color1, color2, t)` ที่ interpolate ระหว่างสองสี hex

**แบบฝึกหัด 9:** สร้าง `Matrix` class ที่รองรับ add, multiply, transpose, determinant

### ระดับยาก

**แบบฝึกหัด 10:** เขียน numerical integration (Simpson's rule)

**แบบฝึกหัด 11:** สร้าง `Complex` number class

### เฉลยบางส่วน

```javascript
// แบบฝึกหัด 1
function roundToNearest(n, nearest) {
  return Math.round(n / nearest) * nearest;
}

// แบบฝึกหัด 2
const celsiusToFahrenheit = c => c * 9/5 + 32;
const fahrenheitToCelsius = f => (f - 32) * 5/9;

// แบบฝึกหัด 3
function isPerfectSquare(n) {
  return Number.isInteger(Math.sqrt(n));
}

// แบบฝึกหัด 4
function digitSum(n) {
  return Math.abs(n).toString().split('').reduce((sum, d) => sum + Number(d), 0);
}

// แบบฝึกหัด 5
function fibonacci(n) {
  if (n <= 1) return n;
  let [a, b] = [0, 1];
  for (let i = 2; i <= n; i++) [a, b] = [b, a + b];
  return b;
}

// แบบฝึกหัด 7
function formatBytes(bytes) {
  if (bytes === 0) return '0 B';
  const units = ['B', 'KB', 'MB', 'GB', 'TB'];
  const k = 1024;
  const i = Math.floor(Math.log(bytes) / Math.log(k));
  return `${parseFloat((bytes / k ** i).toFixed(2))} ${units[i]}`;
}

console.log(formatBytes(1024));          // '1 KB'
console.log(formatBytes(1536));          // '1.5 KB'
console.log(formatBytes(1073741824));    // '1 GB'
```

---

## สรุป

ในบทนี้เราได้เรียนรู้:
- Number literals: decimal, hex, binary, octal, scientific notation
- Number properties: MAX_VALUE, MIN_VALUE, POSITIVE_INFINITY, NaN, EPSILON
- Number methods: toString, toFixed, toPrecision, toLocaleString
- Type checking: isNaN, isFinite, isInteger, isSafeInteger
- parseInt และ parseFloat
- BigInt สำหรับตัวเลขขนาดใหญ่
- Math constants: PI, E, SQRT2
- Math methods: abs, ceil, floor, round, trunc
- Math.max, Math.min, Math.random
- Math.pow, Math.sqrt, Math.cbrt
- Trigonometry: sin, cos, tan, atan2
- Math.log, Math.log2, Math.log10
- การประยุกต์ใช้ในการคำนวณจริง

การเข้าใจ numbers และ Math object ช่วยให้สามารถสร้างแอพพลิเคชั่นที่ต้องการการคำนวณได้อย่างถูกต้องและมีประสิทธิภาพ
