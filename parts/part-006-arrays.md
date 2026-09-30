# ตอนที่ 6: Arrays (อาร์เรย์) ใน JavaScript

## บทนำ

Array คือโครงสร้างข้อมูลพื้นฐานที่สำคัญที่สุดอย่างหนึ่งใน JavaScript ใช้สำหรับเก็บข้อมูลหลายรายการในตัวแปรเดียว เช่น รายชื่อสินค้า, คะแนนนักเรียน, หรือรายการงานที่ต้องทำ

ในบทนี้เราจะครอบคลุม Steps 91-110 ซึ่งเป็นส่วนที่สำคัญมากในการเขียน JavaScript

---

## Step 91: การสร้าง Array

### 1. Array Literal (วิธีที่ใช้บ่อยที่สุด)

```javascript
// สร้าง array เปล่า
const emptyArray = [];

// สร้าง array ที่มีสมาชิก
const fruits = ['แอปเปิ้ล', 'กล้วย', 'ส้ม'];
const numbers = [1, 2, 3, 4, 5];
const mixed = [1, 'hello', true, null, undefined];

console.log(fruits);   // ['แอปเปิ้ล', 'กล้วย', 'ส้ม']
console.log(numbers);  // [1, 2, 3, 4, 5]
console.log(mixed);    // [1, 'hello', true, null, undefined]
```

### 2. Array Constructor

```javascript
// สร้าง array ด้วย new Array()
const arr1 = new Array(3);           // array ว่าง ขนาด 3
const arr2 = new Array(1, 2, 3);     // [1, 2, 3]
const arr3 = new Array('a', 'b');    // ['a', 'b']

console.log(arr1);  // [empty × 3]
console.log(arr2);  // [1, 2, 3]
console.log(arr3);  // ['a', 'b']

// ระวัง! new Array(3) ไม่เหมือน new Array(1,2,3)
const confusing = new Array(5);
console.log(confusing.length);  // 5
console.log(confusing[0]);      // undefined
```

### 3. Array.from()

```javascript
// จาก string
const chars = Array.from('hello');
console.log(chars);  // ['h', 'e', 'l', 'l', 'o']

// จาก Set
const uniqueNums = Array.from(new Set([1, 2, 2, 3, 3]));
console.log(uniqueNums);  // [1, 2, 3]

// จาก Map
const map = new Map([['a', 1], ['b', 2]]);
const fromMap = Array.from(map);
console.log(fromMap);  // [['a', 1], ['b', 2]]

// ใช้ mapping function
const doubled = Array.from([1, 2, 3], x => x * 2);
console.log(doubled);  // [2, 4, 6]

// สร้าง array ด้วย range
const range = Array.from({ length: 5 }, (_, i) => i + 1);
console.log(range);  // [1, 2, 3, 4, 5]

// สร้าง array ตัวเลขคู่
const evens = Array.from({ length: 5 }, (_, i) => (i + 1) * 2);
console.log(evens);  // [2, 4, 6, 8, 10]

// แปลง NodeList เป็น Array (ในเบราว์เซอร์)
// const divs = Array.from(document.querySelectorAll('div'));
```

### 4. Array.of()

```javascript
// Array.of() แก้ปัญหาของ new Array()
const arr1 = Array.of(3);        // [3] ไม่ใช่ array ว่างขนาด 3!
const arr2 = Array.of(1, 2, 3); // [1, 2, 3]
const arr3 = Array.of('a');     // ['a']

console.log(arr1);  // [3]
console.log(arr2);  // [1, 2, 3]

// เปรียบเทียบ
console.log(new Array(3));   // [empty × 3]
console.log(Array.of(3));    // [3]
```

---

## Step 92: การเข้าถึงสมาชิกใน Array

### การใช้ Index

```javascript
const colors = ['แดง', 'เขียว', 'น้ำเงิน', 'เหลือง'];

// index เริ่มที่ 0
console.log(colors[0]);   // 'แดง'
console.log(colors[1]);   // 'เขียว'
console.log(colors[2]);   // 'น้ำเงิน'
console.log(colors[3]);   // 'เหลือง'
console.log(colors[4]);   // undefined (ไม่มี index 4)

// เข้าถึงจากท้าย
const last = colors[colors.length - 1];
console.log(last);  // 'เหลือง'

// ใช้ at() method (ES2022)
console.log(colors.at(0));   // 'แดง'
console.log(colors.at(-1));  // 'เหลือง' (นับจากท้าย)
console.log(colors.at(-2));  // 'น้ำเงิน'
```

### Length Property

```javascript
const nums = [10, 20, 30, 40, 50];

console.log(nums.length);  // 5

// เปลี่ยน length ได้!
nums.length = 3;
console.log(nums);  // [10, 20, 30]

// เพิ่ม length
nums.length = 6;
console.log(nums);  // [10, 20, 30, empty × 3]

// เคลียร์ array
const arr = [1, 2, 3, 4, 5];
arr.length = 0;
console.log(arr);  // []
```

### การแก้ไขค่าใน Array

```javascript
const scores = [85, 90, 75, 88];

// แก้ไขค่า
scores[0] = 95;
console.log(scores);  // [95, 90, 75, 88]

// เพิ่มค่าที่ index ใหม่
scores[4] = 100;
console.log(scores);  // [95, 90, 75, 88, 100]

// เพิ่มที่ index ไกล (จะมี empty slots)
scores[10] = 50;
console.log(scores.length);  // 11
console.log(scores[7]);      // undefined
```

---

## Step 93: Mutating Methods - push และ pop

### push() - เพิ่มสมาชิกท้าย array

```javascript
const stack = [];

// push คืนค่า length ใหม่
let newLength = stack.push('ชั้น 1');
console.log(stack);      // ['ชั้น 1']
console.log(newLength);  // 1

stack.push('ชั้น 2');
stack.push('ชั้น 3');
console.log(stack);  // ['ชั้น 1', 'ชั้น 2', 'ชั้น 3']

// push หลายค่าพร้อมกัน
stack.push('ชั้น 4', 'ชั้น 5');
console.log(stack);  // ['ชั้น 1', 'ชั้น 2', 'ชั้น 3', 'ชั้น 4', 'ชั้น 5']
```

### pop() - ลบสมาชิกท้าย array

```javascript
const stack = ['ชั้น 1', 'ชั้น 2', 'ชั้น 3'];

// pop คืนค่าที่ถูกลบ
const removed = stack.pop();
console.log(removed);  // 'ชั้น 3'
console.log(stack);    // ['ชั้น 1', 'ชั้น 2']

stack.pop();
console.log(stack);  // ['ชั้น 1']

stack.pop();
console.log(stack);  // []

// pop จาก array ว่าง
const result = stack.pop();
console.log(result);  // undefined
console.log(stack);   // []
```

### ตัวอย่าง Stack (LIFO)

```javascript
// Stack คือโครงสร้างข้อมูลแบบ Last In, First Out
const undoStack = [];

function doAction(action) {
  undoStack.push(action);
  console.log(`ทำ: ${action}`);
}

function undo() {
  if (undoStack.length === 0) {
    console.log('ไม่มีอะไรให้ยกเลิก');
    return;
  }
  const action = undoStack.pop();
  console.log(`ยกเลิก: ${action}`);
}

doAction('พิมพ์ข้อความ A');
doAction('พิมพ์ข้อความ B');
doAction('ลบข้อความ C');
undo();  // ยกเลิก: ลบข้อความ C
undo();  // ยกเลิก: พิมพ์ข้อความ B
```

---

## Step 94: Mutating Methods - shift และ unshift

### shift() - ลบสมาชิกแรก

```javascript
const queue = ['คนที่ 1', 'คนที่ 2', 'คนที่ 3'];

const first = queue.shift();
console.log(first);  // 'คนที่ 1'
console.log(queue);  // ['คนที่ 2', 'คนที่ 3']
```

### unshift() - เพิ่มสมาชิกที่ต้น

```javascript
const queue = ['คนที่ 2', 'คนที่ 3'];

queue.unshift('คนที่ 1');
console.log(queue);  // ['คนที่ 1', 'คนที่ 2', 'คนที่ 3']

// เพิ่มหลายค่า
queue.unshift('คนที่ 0', 'คนที่ -1');
console.log(queue);  // ['คนที่ 0', 'คนที่ -1', 'คนที่ 1', 'คนที่ 2', 'คนที่ 3']

// unshift คืนค่า length ใหม่
const newLen = queue.unshift('VIP');
console.log(newLen);  // 6
```

### ตัวอย่าง Queue (FIFO)

```javascript
// Queue คือโครงสร้างข้อมูลแบบ First In, First Out
const printQueue = [];

function addToQueue(job) {
  printQueue.push(job);  // เพิ่มท้าย
  console.log(`เพิ่มงาน: ${job}`);
}

function processNext() {
  if (printQueue.length === 0) {
    console.log('ไม่มีงานในคิว');
    return;
  }
  const job = printQueue.shift();  // ดึงจากต้น
  console.log(`ประมวลผล: ${job}`);
}

addToQueue('เอกสาร A');
addToQueue('เอกสาร B');
addToQueue('เอกสาร C');
processNext();  // ประมวลผล: เอกสาร A
processNext();  // ประมวลผล: เอกสาร B
```

---

## Step 95: Mutating Methods - splice

splice() ใช้สำหรับเพิ่ม ลบ หรือแทนที่สมาชิกใน array

```javascript
// splice(startIndex, deleteCount, ...itemsToAdd)
const arr = ['a', 'b', 'c', 'd', 'e'];

// ลบ 2 ตัวเริ่มจาก index 1
const removed = arr.splice(1, 2);
console.log(removed);  // ['b', 'c']
console.log(arr);      // ['a', 'd', 'e']
```

```javascript
// เพิ่มโดยไม่ลบ
const fruits = ['แอปเปิ้ล', 'ส้ม', 'มะม่วง'];
fruits.splice(1, 0, 'กล้วย', 'สับปะรด');
console.log(fruits);  // ['แอปเปิ้ล', 'กล้วย', 'สับปะรด', 'ส้ม', 'มะม่วง']
```

```javascript
// แทนที่ค่า
const colors = ['แดง', 'เขียว', 'น้ำเงิน'];
colors.splice(1, 1, 'เหลือง');
console.log(colors);  // ['แดง', 'เหลือง', 'น้ำเงิน']
```

```javascript
// ลบจากท้าย (ใช้ index ลบ)
const nums = [1, 2, 3, 4, 5];
nums.splice(-2);  // ลบ 2 ตัวท้าย
console.log(nums);  // [1, 2, 3]
```

```javascript
// ตัวอย่างการใช้งานจริง: ลบสินค้าออกจากตะกร้า
const cart = [
  { id: 1, name: 'เสื้อ', price: 299 },
  { id: 2, name: 'กางเกง', price: 499 },
  { id: 3, name: 'รองเท้า', price: 799 }
];

function removeFromCart(id) {
  const index = cart.findIndex(item => item.id === id);
  if (index !== -1) {
    const [removed] = cart.splice(index, 1);
    console.log(`ลบ ${removed.name} ออกจากตะกร้า`);
  }
}

removeFromCart(2);
console.log(cart);
// [{ id: 1, name: 'เสื้อ', price: 299 }, { id: 3, name: 'รองเท้า', price: 799 }]
```

---

## Step 96: Mutating Methods - sort

```javascript
// sort ปกติ (เรียงเป็น string)
const fruits = ['มะม่วง', 'แอปเปิ้ล', 'กล้วย', 'ส้ม'];
fruits.sort();
console.log(fruits);  // ['กล้วย', 'ซ้ม', 'มะม่วง', 'แอปเปิ้ล'] (เรียงตาม Unicode)
```

```javascript
// sort ตัวเลข (ต้องใช้ comparator)
const numbers = [10, 3, 25, 1, 8];

// ผิด! sort โดยไม่มี comparator
numbers.sort();
console.log(numbers);  // [1, 10, 25, 3, 8] ผิด!

// ถูก! ใช้ comparator
numbers.sort((a, b) => a - b);  // เรียงจากน้อยไปมาก
console.log(numbers);  // [1, 3, 8, 10, 25]

numbers.sort((a, b) => b - a);  // เรียงจากมากไปน้อย
console.log(numbers);  // [25, 10, 8, 3, 1]
```

```javascript
// sort object array
const students = [
  { name: 'สมชาย', score: 85 },
  { name: 'สมหญิง', score: 92 },
  { name: 'สมศักดิ์', score: 78 },
  { name: 'สมใจ', score: 95 }
];

// เรียงตามคะแนน (มากไปน้อย)
students.sort((a, b) => b.score - a.score);
console.log(students);
/*
[
  { name: 'สมใจ', score: 95 },
  { name: 'สมหญิง', score: 92 },
  { name: 'สมชาย', score: 85 },
  { name: 'สมศักดิ์', score: 78 }
]
*/

// เรียงตามชื่อ
students.sort((a, b) => a.name.localeCompare(b.name, 'th'));
```

```javascript
// Stable Sort (ES2019+)
// sort ใน JavaScript เป็น stable sort แล้ว
const items = [
  { name: 'A', priority: 1 },
  { name: 'B', priority: 2 },
  { name: 'C', priority: 1 },
  { name: 'D', priority: 2 }
];

items.sort((a, b) => a.priority - b.priority);
// A และ C ยังคงอยู่ตามลำดับเดิมของกัน
console.log(items.map(i => i.name));  // ['A', 'C', 'B', 'D']
```

---

## Step 97: Mutating Methods - reverse และ fill

### reverse()

```javascript
const arr = [1, 2, 3, 4, 5];
arr.reverse();
console.log(arr);  // [5, 4, 3, 2, 1]

// reverse string
function reverseString(str) {
  return str.split('').reverse().join('');
}
console.log(reverseString('hello'));  // 'olleh'
console.log(reverseString('JavaScript'));  // 'tpircSavaJ'
```

```javascript
// ตรวจสอบ palindrome
function isPalindrome(str) {
  const cleaned = str.toLowerCase().replace(/[^a-z0-9]/g, '');
  const reversed = cleaned.split('').reverse().join('');
  return cleaned === reversed;
}

console.log(isPalindrome('racecar'));  // true
console.log(isPalindrome('hello'));    // false
console.log(isPalindrome('A man a plan a canal Panama'));  // true
```

### fill()

```javascript
// fill(value, start, end)
const arr = [1, 2, 3, 4, 5];

arr.fill(0);  // เติม 0 ทั้งหมด
console.log(arr);  // [0, 0, 0, 0, 0]

const arr2 = [1, 2, 3, 4, 5];
arr2.fill(9, 2, 4);  // เติม 9 จาก index 2 ถึง 4 (ไม่รวม 4)
console.log(arr2);  // [1, 2, 9, 9, 5]

// สร้าง array ที่เต็มด้วยค่า
const zeros = new Array(5).fill(0);
console.log(zeros);  // [0, 0, 0, 0, 0]

const board = new Array(3).fill(null).map(() => new Array(3).fill(0));
console.log(board);  // [[0,0,0],[0,0,0],[0,0,0]]
```

---

## Step 98: Non-Mutating Methods - slice

slice() สร้าง array ใหม่โดยไม่เปลี่ยนต้นฉบับ

```javascript
// slice(start, end) - ไม่รวม end
const fruits = ['แอปเปิ้ล', 'กล้วย', 'ส้ม', 'มะม่วง', 'สับปะรด'];

console.log(fruits.slice(1, 3));   // ['กล้วย', 'ส้ม']
console.log(fruits.slice(2));      // ['ส้ม', 'มะม่วง', 'สับปะรด']
console.log(fruits.slice(0, -1));  // ['แอปเปิ้ล', 'กล้วย', 'ส้ม', 'มะม่วง']
console.log(fruits.slice(-2));     // ['มะม่วง', 'สับปะรด']
console.log(fruits.slice());       // copy ทั้งหมด

// ต้นฉบับไม่เปลี่ยน
console.log(fruits);  // ยังคงเดิม
```

```javascript
// ใช้ slice สำหรับ copy array
const original = [1, 2, 3, 4, 5];
const copy = original.slice();
copy.push(6);
console.log(original);  // [1, 2, 3, 4, 5]
console.log(copy);      // [1, 2, 3, 4, 5, 6]
```

```javascript
// แบ่ง array เป็น chunks
function chunkArray(arr, size) {
  const chunks = [];
  for (let i = 0; i < arr.length; i += size) {
    chunks.push(arr.slice(i, i + size));
  }
  return chunks;
}

const data = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
console.log(chunkArray(data, 3));
// [[1,2,3], [4,5,6], [7,8,9], [10]]
```

---

## Step 99: Non-Mutating Methods - concat, join, indexOf

### concat()

```javascript
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];
const arr3 = [7, 8, 9];

// รวม arrays
const combined = arr1.concat(arr2);
console.log(combined);  // [1, 2, 3, 4, 5, 6]

// รวมหลาย array
const all = arr1.concat(arr2, arr3);
console.log(all);  // [1, 2, 3, 4, 5, 6, 7, 8, 9]

// รวมกับค่าเดี่ยว
const withExtra = arr1.concat(10, 11, arr2);
console.log(withExtra);  // [1, 2, 3, 10, 11, 4, 5, 6]

// concat ไม่เปลี่ยนต้นฉบับ
console.log(arr1);  // [1, 2, 3]
```

### join()

```javascript
const words = ['Hello', 'World', 'JavaScript'];

console.log(words.join());        // 'Hello,World,JavaScript'
console.log(words.join(' '));     // 'Hello World JavaScript'
console.log(words.join('-'));     // 'Hello-World-JavaScript'
console.log(words.join(', '));    // 'Hello, World, JavaScript'
console.log(words.join(''));      // 'HelloWorldJavaScript'

// ใช้กับตัวเลข
const nums = [1, 2, 3, 4, 5];
console.log(nums.join(' + ') + ' = ' + nums.reduce((a, b) => a + b));
// '1 + 2 + 3 + 4 + 5 = 15'
```

### indexOf() และ lastIndexOf()

```javascript
const arr = [10, 20, 30, 20, 40, 20];

console.log(arr.indexOf(20));      // 1 (ตำแหน่งแรก)
console.log(arr.lastIndexOf(20));  // 5 (ตำแหน่งสุดท้าย)
console.log(arr.indexOf(99));      // -1 (ไม่พบ)

// หาจาก index ที่กำหนด
console.log(arr.indexOf(20, 2));   // 3 (หาจาก index 2 เป็นต้นไป)

// ตรวจสอบว่ามีค่าหรือไม่
function contains(arr, value) {
  return arr.indexOf(value) !== -1;
}
console.log(contains([1, 2, 3], 2));  // true
console.log(contains([1, 2, 3], 5));  // false
```

---

## Step 100: Non-Mutating Methods - includes

```javascript
const fruits = ['แอปเปิ้ล', 'กล้วย', 'ส้ม'];

console.log(fruits.includes('กล้วย'));    // true
console.log(fruits.includes('มะม่วง')); // false

// includes กับ NaN (ดีกว่า indexOf)
const arr = [1, NaN, 3];
console.log(arr.indexOf(NaN));   // -1 (ไม่สามารถหา NaN ด้วย indexOf)
console.log(arr.includes(NaN));  // true (includes ทำงานกับ NaN ได้)

// หาจาก index ที่กำหนด
const nums = [1, 2, 3, 4, 5];
console.log(nums.includes(3, 3));  // false (หาจาก index 3 เป็นต้นไป)
console.log(nums.includes(3, 2));  // true (3 อยู่ที่ index 2)

// ใช้งานจริง: ตรวจสอบ permission
const userPermissions = ['read', 'write', 'delete'];
const hasDeletePermission = userPermissions.includes('delete');
console.log(hasDeletePermission);  // true
```

---

## Step 101: Iteration Methods - forEach

forEach() วนซ้ำทุกสมาชิก แต่ไม่คืนค่า

```javascript
const numbers = [1, 2, 3, 4, 5];

// รูปแบบพื้นฐาน
numbers.forEach(function(num) {
  console.log(num * 2);
});
// 2, 4, 6, 8, 10

// ใช้ arrow function
numbers.forEach(num => console.log(num));

// รับ index ด้วย
numbers.forEach((num, index) => {
  console.log(`index ${index}: ${num}`);
});

// รับ array ต้นฉบับด้วย
numbers.forEach((num, index, arr) => {
  console.log(`${num} / ${arr.length}`);
});
```

```javascript
// ตัวอย่างจริง: แสดงรายการสินค้า
const products = [
  { name: 'เสื้อ', price: 299, stock: 10 },
  { name: 'กางเกง', price: 499, stock: 5 },
  { name: 'รองเท้า', price: 799, stock: 8 }
];

let total = 0;
products.forEach(product => {
  console.log(`${product.name}: ราคา ${product.price} บาท`);
  total += product.price * product.stock;
});
console.log(`มูลค่าสินค้าทั้งหมด: ${total} บาท`);
```

```javascript
// forEach ไม่สามารถ break ได้!
// ถ้าต้องการ break ให้ใช้ for...of แทน
const nums = [1, 2, 3, 4, 5];

// อันนี้จะวนครบทุกตัว แม้จะ return
nums.forEach(num => {
  if (num === 3) return;  // แค่ skip ไม่ใช่ break
  console.log(num);  // 1, 2, 4, 5
});

// ใช้ for...of ถ้าต้องการ break
for (const num of nums) {
  if (num === 3) break;
  console.log(num);  // 1, 2
}
```

---

## Step 102: Iteration Methods - map

map() สร้าง array ใหม่โดย transform ทุกสมาชิก

```javascript
const numbers = [1, 2, 3, 4, 5];

// คูณ 2 ทุกตัว
const doubled = numbers.map(num => num * 2);
console.log(doubled);   // [2, 4, 6, 8, 10]
console.log(numbers);   // [1, 2, 3, 4, 5] ไม่เปลี่ยน

// ยกกำลังสอง
const squared = numbers.map(num => num ** 2);
console.log(squared);  // [1, 4, 9, 16, 25]
```

```javascript
// แปลง object array
const users = [
  { firstName: 'สมชาย', lastName: 'ใจดี', age: 25 },
  { firstName: 'สมหญิง', lastName: 'ใจงาม', age: 30 },
  { firstName: 'สมศักดิ์', lastName: 'ใจกล้า', age: 28 }
];

// ดึงแค่ชื่อ
const names = users.map(user => `${user.firstName} ${user.lastName}`);
console.log(names);
// ['สมชาย ใจดี', 'สมหญิง ใจงาม', 'สมศักดิ์ ใจกล้า']

// เพิ่ม property ใหม่
const usersWithFullName = users.map(user => ({
  ...user,
  fullName: `${user.firstName} ${user.lastName}`
}));
```

```javascript
// map ใช้กับ API responses
const apiResponse = [
  { id: 1, product_name: 'shirt', product_price: 299 },
  { id: 2, product_name: 'pants', product_price: 499 }
];

const products = apiResponse.map(item => ({
  id: item.id,
  name: item.product_name,
  price: item.product_price,
  priceWithVat: item.product_price * 1.07
}));

console.log(products);
```

```javascript
// chain methods
const result = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
  .map(n => n * 2)        // [2, 4, 6, 8, 10, 12, 14, 16, 18, 20]
  .filter(n => n > 10)    // [12, 14, 16, 18, 20]
  .map(n => `${n} บาท`);  // ['12 บาท', '14 บาท', ...]

console.log(result);
```

---

## Step 103: Iteration Methods - filter

filter() สร้าง array ใหม่ที่มีเฉพาะสมาชิกที่ผ่านเงื่อนไข

```javascript
const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

// กรองเลขคู่
const evens = numbers.filter(n => n % 2 === 0);
console.log(evens);  // [2, 4, 6, 8, 10]

// กรองเลขที่มากกว่า 5
const bigNums = numbers.filter(n => n > 5);
console.log(bigNums);  // [6, 7, 8, 9, 10]
```

```javascript
// กรอง object array
const products = [
  { name: 'เสื้อ', price: 299, inStock: true },
  { name: 'กางเกง', price: 499, inStock: false },
  { name: 'รองเท้า', price: 799, inStock: true },
  { name: 'หมวก', price: 199, inStock: true },
  { name: 'กระเป๋า', price: 1299, inStock: false }
];

// กรองเฉพาะสินค้าที่มีในคลัง
const available = products.filter(p => p.inStock);
console.log(available.length);  // 3

// กรองสินค้าราคาต่ำกว่า 500
const affordable = products.filter(p => p.price < 500 && p.inStock);
console.log(affordable.map(p => p.name));  // ['เสื้อ', 'หมวก']
```

```javascript
// กรองค่าที่ไม่ต้องการออก
const data = [0, 1, null, 2, undefined, 3, '', 4, false, 5];

// กรองเฉพาะค่า truthy
const truthy = data.filter(Boolean);
console.log(truthy);  // [1, 2, 3, 4, 5]

// กรอง null และ undefined
const withoutNullish = data.filter(x => x !== null && x !== undefined);
console.log(withoutNullish);  // [0, 1, 2, 3, '', 4, false, 5]
```

```javascript
// ลบค่าซ้ำ (วิธีง่าย)
const nums = [1, 2, 2, 3, 3, 3, 4];
const unique = nums.filter((value, index, arr) => arr.indexOf(value) === index);
console.log(unique);  // [1, 2, 3, 4]

// วิธีที่ดีกว่าด้วย Set
const uniqueWithSet = [...new Set(nums)];
console.log(uniqueWithSet);  // [1, 2, 3, 4]
```

---

## Step 104: Iteration Methods - reduce

reduce() รวม array เป็นค่าเดียว (powerful มาก!)

```javascript
// reduce(callback, initialValue)
// callback รับ (accumulator, currentValue, index, array)
const numbers = [1, 2, 3, 4, 5];

// บวกทุกตัว
const sum = numbers.reduce((acc, curr) => acc + curr, 0);
console.log(sum);  // 15

// คูณทุกตัว
const product = numbers.reduce((acc, curr) => acc * curr, 1);
console.log(product);  // 120

// หาค่าสูงสุด
const max = numbers.reduce((acc, curr) => curr > acc ? curr : acc, -Infinity);
console.log(max);  // 5
```

```javascript
// นับจำนวนของแต่ละค่า
const votes = ['A', 'B', 'A', 'C', 'B', 'A', 'B'];
const counts = votes.reduce((acc, vote) => {
  acc[vote] = (acc[vote] || 0) + 1;
  return acc;
}, {});
console.log(counts);  // { A: 3, B: 3, C: 1 }
```

```javascript
// จัดกลุ่มข้อมูล (groupBy)
const students = [
  { name: 'สมชาย', grade: 'A' },
  { name: 'สมหญิง', grade: 'B' },
  { name: 'สมศักดิ์', grade: 'A' },
  { name: 'สมใจ', grade: 'C' },
  { name: 'สมบัติ', grade: 'B' }
];

const grouped = students.reduce((acc, student) => {
  const grade = student.grade;
  if (!acc[grade]) acc[grade] = [];
  acc[grade].push(student.name);
  return acc;
}, {});

console.log(grouped);
/*
{
  A: ['สมชาย', 'สมศักดิ์'],
  B: ['สมหญิง', 'สมบัติ'],
  C: ['สมใจ']
}
*/
```

```javascript
// สร้าง array จาก array (flatten)
const nested = [[1, 2], [3, 4], [5, 6]];
const flat = nested.reduce((acc, arr) => acc.concat(arr), []);
console.log(flat);  // [1, 2, 3, 4, 5, 6]
```

```javascript
// คำนวณยอดขาย
const sales = [
  { product: 'เสื้อ', qty: 10, price: 299 },
  { product: 'กางเกง', qty: 5, price: 499 },
  { product: 'รองเท้า', qty: 8, price: 799 }
];

const report = sales.reduce((acc, sale) => {
  const revenue = sale.qty * sale.price;
  return {
    totalRevenue: acc.totalRevenue + revenue,
    totalItems: acc.totalItems + sale.qty,
    details: [...acc.details, { ...sale, revenue }]
  };
}, { totalRevenue: 0, totalItems: 0, details: [] });

console.log(`รายได้รวม: ${report.totalRevenue} บาท`);
console.log(`จำนวนชิ้นทั้งหมด: ${report.totalItems} ชิ้น`);
```

---

## Step 105: reduceRight และ Search Methods

### reduceRight()

```javascript
// reduceRight วนจากขวาไปซ้าย
const arr = [[1, 2], [3, 4], [5, 6]];
const flatRight = arr.reduceRight((acc, val) => acc.concat(val), []);
console.log(flatRight);  // [5, 6, 3, 4, 1, 2]

// ใช้กับ function composition
const compose = (...fns) => x => fns.reduceRight((acc, fn) => fn(acc), x);

const double = x => x * 2;
const addOne = x => x + 1;
const square = x => x * x;

const transform = compose(square, addOne, double);
console.log(transform(3));  // square(addOne(double(3))) = square(addOne(6)) = square(7) = 49
```

### find() และ findIndex()

```javascript
const users = [
  { id: 1, name: 'สมชาย', role: 'admin' },
  { id: 2, name: 'สมหญิง', role: 'user' },
  { id: 3, name: 'สมศักดิ์', role: 'user' },
  { id: 4, name: 'สมใจ', role: 'admin' }
];

// find คืนค่าแรกที่พบ
const admin = users.find(u => u.role === 'admin');
console.log(admin);  // { id: 1, name: 'สมชาย', role: 'admin' }

const notFound = users.find(u => u.role === 'superadmin');
console.log(notFound);  // undefined

// findIndex คืน index
const adminIndex = users.findIndex(u => u.role === 'admin');
console.log(adminIndex);  // 0

const notFoundIndex = users.findIndex(u => u.id === 99);
console.log(notFoundIndex);  // -1
```

---

## Step 106: findLast, findLastIndex, flat, flatMap

### findLast() และ findLastIndex() (ES2023)

```javascript
const nums = [1, 2, 3, 4, 5, 4, 3, 2, 1];

// findLast หาตัวสุดท้ายที่ตรงเงื่อนไข
const lastEven = nums.findLast(n => n % 2 === 0);
console.log(lastEven);  // 2

// findLastIndex หา index ตัวสุดท้าย
const lastEvenIndex = nums.findLastIndex(n => n % 2 === 0);
console.log(lastEvenIndex);  // 7
```

### flat()

```javascript
// flat() ทำให้ array ลดระดับความซ้อน
const nested = [1, [2, 3], [4, [5, 6]]];

console.log(nested.flat());     // [1, 2, 3, 4, [5, 6]] (ระดับเดียว)
console.log(nested.flat(2));    // [1, 2, 3, 4, 5, 6] (2 ระดับ)
console.log(nested.flat(Infinity));  // flat ทั้งหมด

const deepNested = [1, [2, [3, [4, [5]]]]];
console.log(deepNested.flat(Infinity));  // [1, 2, 3, 4, 5]
```

### flatMap()

```javascript
// flatMap = map + flat(1)
const sentences = ['Hello World', 'JavaScript is Fun'];

// แยกคำทุกประโยค
const words = sentences.flatMap(s => s.split(' '));
console.log(words);  // ['Hello', 'World', 'JavaScript', 'is', 'Fun']

// เทียบกับ map แล้ว flat
const wordsManual = sentences.map(s => s.split(' ')).flat();
console.log(wordsManual);  // เหมือนกัน

// flatMap กับ object
const orders = [
  { customer: 'สมชาย', items: ['เสื้อ', 'กางเกง'] },
  { customer: 'สมหญิง', items: ['รองเท้า'] },
  { customer: 'สมศักดิ์', items: ['หมวก', 'กระเป๋า', 'เสื้อ'] }
];

const allItems = orders.flatMap(order => order.items);
console.log(allItems);  // ['เสื้อ', 'กางเกง', 'รองเท้า', 'หมวก', 'กระเป๋า', 'เสื้อ']
```

---

## Step 107: every และ some

### every() - ตรวจว่าทุกตัวผ่านเงื่อนไข

```javascript
const nums = [2, 4, 6, 8, 10];

console.log(nums.every(n => n % 2 === 0));  // true (ทุกตัวเป็นเลขคู่)
console.log(nums.every(n => n > 5));         // false (ไม่ใช่ทุกตัว)

// ตรวจสอบความถูกต้องของข้อมูล
const formData = {
  name: 'สมชาย',
  email: 'somchai@test.com',
  age: 25
};

const requiredFields = ['name', 'email', 'age'];
const isComplete = requiredFields.every(field => 
  formData[field] !== undefined && formData[field] !== ''
);
console.log(isComplete);  // true
```

### some() - ตรวจว่ามีอย่างน้อยหนึ่งตัวผ่านเงื่อนไข

```javascript
const products = [
  { name: 'A', inStock: false },
  { name: 'B', inStock: true },
  { name: 'C', inStock: false }
];

console.log(products.some(p => p.inStock));   // true (B มีในคลัง)
console.log(products.every(p => p.inStock));  // false

// ตรวจสอบ permission
const userRoles = ['viewer', 'editor'];
const canEdit = userRoles.some(role => ['editor', 'admin'].includes(role));
console.log(canEdit);  // true

// ตรวจสอบว่ามีสมาชิกที่อายุต่ำกว่า 18
const ages = [20, 25, 17, 30];
const hasMinor = ages.some(age => age < 18);
console.log(hasMinor);  // true
```

---

## Step 108: Array Destructuring

```javascript
// รูปแบบพื้นฐาน
const [a, b, c] = [1, 2, 3];
console.log(a, b, c);  // 1 2 3

// ข้ามค่า
const [first, , third] = [10, 20, 30];
console.log(first, third);  // 10 30

// ค่า default
const [x = 0, y = 0, z = 0] = [1, 2];
console.log(x, y, z);  // 1 2 0

// rest operator
const [head, ...tail] = [1, 2, 3, 4, 5];
console.log(head);  // 1
console.log(tail);  // [2, 3, 4, 5]
```

```javascript
// สลับค่าตัวแปร
let p = 1, q = 2;
[p, q] = [q, p];
console.log(p, q);  // 2 1
```

```javascript
// จาก function ที่คืน array
function getMinMax(arr) {
  return [Math.min(...arr), Math.max(...arr)];
}

const [min, max] = getMinMax([5, 3, 8, 1, 9, 2]);
console.log(min, max);  // 1 9
```

```javascript
// Nested destructuring
const matrix = [[1, 2], [3, 4]];
const [[a, b], [c, d]] = matrix;
console.log(a, b, c, d);  // 1 2 3 4

// จาก API response
const response = {
  data: {
    users: [
      { id: 1, name: 'สมชาย' },
      { id: 2, name: 'สมหญิง' }
    ]
  }
};

const { data: { users: [firstUser] } } = response;
console.log(firstUser);  // { id: 1, name: 'สมชาย' }
```

```javascript
// ใช้ใน function parameters
function processCoordinates([x, y, z = 0]) {
  console.log(`X: ${x}, Y: ${y}, Z: ${z}`);
}

processCoordinates([10, 20]);      // X: 10, Y: 20, Z: 0
processCoordinates([10, 20, 30]); // X: 10, Y: 20, Z: 30

// ใช้กับ for...of
const points = [[1, 2], [3, 4], [5, 6]];
for (const [x, y] of points) {
  console.log(`(${x}, ${y})`);
}
```

---

## Step 109: Multi-dimensional Arrays

```javascript
// สร้าง 2D array
const matrix = [
  [1, 2, 3],
  [4, 5, 6],
  [7, 8, 9]
];

console.log(matrix[1][2]);  // 6 (แถวที่ 2, คอลัมน์ที่ 3)
console.log(matrix[0][0]);  // 1

// วนซ้ำ 2D array
for (let i = 0; i < matrix.length; i++) {
  for (let j = 0; j < matrix[i].length; j++) {
    process.stdout.write(matrix[i][j] + ' ');
  }
  console.log();
}
```

```javascript
// สร้าง grid
function createGrid(rows, cols, defaultValue = 0) {
  return Array.from({ length: rows }, () => 
    Array.from({ length: cols }, () => defaultValue)
  );
}

const grid = createGrid(3, 4, 0);
console.log(grid);
/*
[
  [0, 0, 0, 0],
  [0, 0, 0, 0],
  [0, 0, 0, 0]
]
*/
```

```javascript
// ตาราง multiplication
const multiTable = Array.from({ length: 10 }, (_, i) =>
  Array.from({ length: 10 }, (_, j) => (i + 1) * (j + 1))
);

console.log(multiTable[2][3]);  // 12 (3 × 4)
console.log(multiTable[6][7]);  // 56 (7 × 8)
```

```javascript
// เกม Tic-Tac-Toe
class TicTacToe {
  constructor() {
    this.board = Array.from({ length: 3 }, () => Array(3).fill(null));
    this.currentPlayer = 'X';
  }

  move(row, col) {
    if (this.board[row][col] !== null) {
      return false;  // ตำแหน่งนั้นถูกใช้แล้ว
    }
    this.board[row][col] = this.currentPlayer;
    this.currentPlayer = this.currentPlayer === 'X' ? 'O' : 'X';
    return true;
  }

  checkWinner() {
    const lines = [
      // แถวนอน
      [[0,0],[0,1],[0,2]],
      [[1,0],[1,1],[1,2]],
      [[2,0],[2,1],[2,2]],
      // แถวตั้ง
      [[0,0],[1,0],[2,0]],
      [[0,1],[1,1],[2,1]],
      [[0,2],[1,2],[2,2]],
      // แนวทะแยง
      [[0,0],[1,1],[2,2]],
      [[0,2],[1,1],[2,0]]
    ];

    for (const line of lines) {
      const [a, b, c] = line.map(([r, col]) => this.board[r][col]);
      if (a && a === b && b === c) return a;
    }
    return null;
  }

  display() {
    return this.board.map(row => 
      row.map(cell => cell || '.').join(' | ')
    ).join('\n---------\n');
  }
}

const game = new TicTacToe();
game.move(0, 0);  // X
game.move(1, 1);  // O
game.move(0, 1);  // X
game.move(2, 2);  // O
game.move(0, 2);  // X wins!
console.log(game.display());
console.log('ผู้ชนะ:', game.checkWinner());
```

---

## Step 110: Advanced Array Techniques

### Spread Operator กับ Array

```javascript
// copy array
const original = [1, 2, 3];
const copy = [...original];
copy.push(4);
console.log(original);  // [1, 2, 3]
console.log(copy);      // [1, 2, 3, 4]

// รวม arrays
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];
const combined = [...arr1, ...arr2];
console.log(combined);  // [1, 2, 3, 4, 5, 6]

// แทรกตรงกลาง
const withMiddle = [...arr1, 10, 11, ...arr2];
console.log(withMiddle);  // [1, 2, 3, 10, 11, 4, 5, 6]
```

```javascript
// ใช้ spread กับ Math functions
const scores = [85, 92, 78, 95, 88];
console.log(Math.max(...scores));  // 95
console.log(Math.min(...scores));  // 78

// ส่ง array เป็น arguments
function sum(a, b, c) {
  return a + b + c;
}
const nums = [1, 2, 3];
console.log(sum(...nums));  // 6
```

```javascript
// Array utility functions
const arr = [3, 1, 4, 1, 5, 9, 2, 6, 5, 3, 5];

// ลบค่าซ้ำ
const unique = [...new Set(arr)];
console.log(unique);  // [3, 1, 4, 5, 9, 2, 6]

// หาค่าซ้ำ
const duplicates = arr.filter((item, index) => arr.indexOf(item) !== index);
const uniqueDuplicates = [...new Set(duplicates)];
console.log(uniqueDuplicates);  // [1, 5, 3]

// แบ่ง array ออกเป็น 2 กลุ่ม
function partition(arr, predicate) {
  return arr.reduce(
    ([pass, fail], item) => 
      predicate(item) ? [[...pass, item], fail] : [pass, [...fail, item]],
    [[], []]
  );
}

const [evens, odds] = partition([1, 2, 3, 4, 5, 6], n => n % 2 === 0);
console.log(evens);  // [2, 4, 6]
console.log(odds);   // [1, 3, 5]
```

```javascript
// zip: รวม arrays คู่กัน
function zip(...arrays) {
  const minLength = Math.min(...arrays.map(a => a.length));
  return Array.from({ length: minLength }, (_, i) => arrays.map(a => a[i]));
}

const names = ['Alice', 'Bob', 'Charlie'];
const ages = [25, 30, 35];
const cities = ['กรุงเทพ', 'เชียงใหม่', 'ภูเก็ต'];

const zipped = zip(names, ages, cities);
console.log(zipped);
/*
[
  ['Alice', 25, 'กรุงเทพ'],
  ['Bob', 30, 'เชียงใหม่'],
  ['Charlie', 35, 'ภูเก็ต']
]
*/
```

```javascript
// intersection (ค่าที่มีทั้งสอง array)
function intersection(a, b) {
  return a.filter(item => b.includes(item));
}

const setA = [1, 2, 3, 4, 5];
const setB = [3, 4, 5, 6, 7];
console.log(intersection(setA, setB));  // [3, 4, 5]

// difference (ค่าใน a ที่ไม่มีใน b)
function difference(a, b) {
  return a.filter(item => !b.includes(item));
}
console.log(difference(setA, setB));  // [1, 2]

// union (ค่าทั้งหมดไม่ซ้ำ)
function union(a, b) {
  return [...new Set([...a, ...b])];
}
console.log(union(setA, setB));  // [1, 2, 3, 4, 5, 6, 7]
```

---

## ตัวอย่างโปรเจกต์จริง: Shopping Cart

```javascript
class ShoppingCart {
  constructor() {
    this.items = [];
  }

  addItem(product, quantity = 1) {
    const existingIndex = this.items.findIndex(item => item.id === product.id);
    
    if (existingIndex !== -1) {
      this.items[existingIndex].quantity += quantity;
    } else {
      this.items.push({ ...product, quantity });
    }
  }

  removeItem(productId) {
    this.items = this.items.filter(item => item.id !== productId);
  }

  updateQuantity(productId, quantity) {
    const item = this.items.find(item => item.id === productId);
    if (item) {
      if (quantity <= 0) {
        this.removeItem(productId);
      } else {
        item.quantity = quantity;
      }
    }
  }

  getTotal() {
    return this.items.reduce((sum, item) => sum + (item.price * item.quantity), 0);
  }

  getItemCount() {
    return this.items.reduce((sum, item) => sum + item.quantity, 0);
  }

  getSummary() {
    return this.items.map(item => ({
      name: item.name,
      quantity: item.quantity,
      unitPrice: item.price,
      subtotal: item.price * item.quantity
    }));
  }

  applyDiscount(percent) {
    const total = this.getTotal();
    const discount = total * (percent / 100);
    return total - discount;
  }

  checkout() {
    if (this.items.length === 0) {
      return { success: false, message: 'ตะกร้าว่าง' };
    }
    
    const summary = this.getSummary();
    const total = this.getTotal();
    
    // ล้างตะกร้า
    this.items = [];
    
    return {
      success: true,
      items: summary,
      total,
      message: 'สั่งซื้อสำเร็จ!'
    };
  }
}

// ใช้งาน
const cart = new ShoppingCart();

cart.addItem({ id: 1, name: 'เสื้อ', price: 299 }, 2);
cart.addItem({ id: 2, name: 'กางเกง', price: 499 });
cart.addItem({ id: 1, name: 'เสื้อ', price: 299 });  // เพิ่มอีก 1 ตัว

console.log('สินค้าในตะกร้า:');
cart.getSummary().forEach(item => {
  console.log(`  ${item.name} × ${item.quantity} = ${item.subtotal} บาท`);
});
console.log(`รวม: ${cart.getTotal()} บาท`);
console.log(`จำนวนสินค้า: ${cart.getItemCount()} ชิ้น`);

const result = cart.checkout();
console.log(result.message);
```

---

## แบบฝึกหัด (Exercises)

### ระดับง่าย

**แบบฝึกหัด 1:** สร้าง function `sumArray(arr)` ที่บวกตัวเลขทุกตัวใน array และคืนผลลัพธ์

**แบบฝึกหัด 2:** สร้าง function `reverseArray(arr)` ที่คืน array ที่กลับด้านโดยไม่แก้ไข array ต้นฉบับ

**แบบฝึกหัด 3:** สร้าง function `removeDuplicates(arr)` ที่ลบค่าซ้ำออกจาก array

**แบบฝึกหัด 4:** สร้าง function `countOccurrences(arr, value)` ที่นับจำนวนครั้งที่ value ปรากฏใน arr

**แบบฝึกหัด 5:** สร้าง function `flatten(arr)` ที่ flat array ทั้งหมดไม่ว่าจะซ้อนกันกี่ชั้น

### ระดับกลาง

**แบบฝึกหัด 6:** สร้าง function `groupBy(arr, key)` ที่จัดกลุ่ม array of objects ตาม key ที่กำหนด

```javascript
// Expected output:
groupBy([
  { name: 'สมชาย', dept: 'IT' },
  { name: 'สมหญิง', dept: 'HR' },
  { name: 'สมศักดิ์', dept: 'IT' }
], 'dept')
// { IT: [{...}, {...}], HR: [{...}] }
```

**แบบฝึกหัด 7:** สร้าง function `paginate(arr, page, pageSize)` ที่แบ่งข้อมูลเป็นหน้า

**แบบฝึกหัด 8:** สร้าง function `sortByMultipleKeys(arr, keys)` ที่เรียงข้อมูลตามหลาย key

**แบบฝึกหัด 9:** สร้าง function `deepEqual(arr1, arr2)` ที่เปรียบเทียบ array สองอัน (รวมถึง nested)

### ระดับยาก

**แบบฝึกหัด 10:** สร้าง `LazyArray` class ที่รองรับ map, filter, และ reduce แบบ lazy (ไม่ประมวลผลจนกว่าจะเรียก `toArray()`)

**แบบฝึกหัด 11:** สร้าง function `permutations(arr)` ที่หา permutation ทั้งหมดของ array

**แบบฝึกหัด 12:** สร้าง function `longestIncreasingSubsequence(arr)` ที่หา subsequence ที่ยาวที่สุดที่เรียงจากน้อยไปมาก

### เฉลยบางส่วน

```javascript
// แบบฝึกหัด 1
function sumArray(arr) {
  return arr.reduce((sum, n) => sum + n, 0);
}
console.log(sumArray([1, 2, 3, 4, 5]));  // 15

// แบบฝึกหัด 2
function reverseArray(arr) {
  return [...arr].reverse();
}

// แบบฝึกหัด 3
function removeDuplicates(arr) {
  return [...new Set(arr)];
}

// แบบฝึกหัด 4
function countOccurrences(arr, value) {
  return arr.filter(item => item === value).length;
}

// แบบฝึกหัด 5
function flatten(arr) {
  return arr.flat(Infinity);
}

// แบบฝึกหัด 6
function groupBy(arr, key) {
  return arr.reduce((groups, item) => {
    const group = item[key];
    if (!groups[group]) groups[group] = [];
    groups[group].push(item);
    return groups;
  }, {});
}

// แบบฝึกหัด 7
function paginate(arr, page, pageSize) {
  const start = (page - 1) * pageSize;
  const end = start + pageSize;
  return {
    data: arr.slice(start, end),
    page,
    pageSize,
    total: arr.length,
    totalPages: Math.ceil(arr.length / pageSize)
  };
}
```

---

## สรุป

ในบทนี้เราได้เรียนรู้:
- การสร้าง Array ด้วยวิธีต่างๆ
- การเข้าถึงและแก้ไขสมาชิก
- Mutating methods: push, pop, shift, unshift, splice, sort, reverse, fill
- Non-mutating methods: slice, concat, join, indexOf, includes
- Iteration methods: forEach, map, filter, reduce
- Search methods: find, findIndex, findLast, findLastIndex
- flat, flatMap, every, some
- Array destructuring
- Multi-dimensional arrays

Array เป็นโครงสร้างข้อมูลที่ใช้บ่อยที่สุดใน JavaScript การเข้าใจ methods ต่างๆ จะช่วยให้เขียนโค้ดได้สั้น กระชับ และอ่านง่ายขึ้นมาก
