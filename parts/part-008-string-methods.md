# ตอนที่ 8: String Methods (เมธอดของ String) ใน JavaScript

## บทนำ

String คือชนิดข้อมูลที่ใช้บ่อยที่สุดในการเขียนโปรแกรม JavaScript มี built-in methods มากมายสำหรับจัดการข้อความ ไม่ว่าจะเป็นการค้นหา, แก้ไข, จัดรูปแบบ หรือแปลงข้อมูล

ในบทนี้เราจะครอบคลุม Steps 131-150

---

## Step 131: การสร้าง String และ Properties

### การสร้าง String

```javascript
// String literal (แนะนำ)
const str1 = 'Hello World';
const str2 = "สวัสดีโลก";
const str3 = `Template literal`;

// String constructor (ไม่แนะนำ)
const str4 = new String('hello');  // String object ไม่ใช่ primitive
console.log(typeof str1);  // 'string'
console.log(typeof str4);  // 'object'

// แปลงเป็น string
const num = 42;
const bool = true;
console.log(String(num));   // '42'
console.log(String(bool));  // 'true'
console.log(String(null));  // 'null'
console.log(num.toString()); // '42'
console.log(num.toString(2));  // '101010' (binary)
console.log(num.toString(16)); // '2a' (hex)
```

### Length Property

```javascript
const text = 'Hello World';
console.log(text.length);  // 11

const thai = 'สวัสดี';
console.log(thai.length);  // 6

const empty = '';
console.log(empty.length);  // 0

// ระวัง! emoji อาจมี length > 1
const emoji = '😀';
console.log(emoji.length);  // 2 (เป็น surrogate pair)

// วนซ้ำตัวอักษร
for (let i = 0; i < text.length; i++) {
  process.stdout.write(text[i]);
}
console.log();  // Hello World
```

### การเข้าถึงตัวอักษร

```javascript
const str = 'JavaScript';

// ด้วย index
console.log(str[0]);    // 'J'
console.log(str[4]);    // 'S'
console.log(str[str.length - 1]);  // 't'

// ด้วย charAt()
console.log(str.charAt(0));   // 'J'
console.log(str.charAt(4));   // 'S'
console.log(str.charAt(99));  // '' (ว่าง ไม่ใช่ undefined)

// charCodeAt() และ codePointAt()
console.log(str.charCodeAt(0));    // 74 (ASCII code ของ 'J')
console.log(str.codePointAt(0));   // 74

// String.fromCharCode()
console.log(String.fromCharCode(74, 83));  // 'JS'
console.log(String.fromCharCode(65, 66, 67));  // 'ABC'
```

---

## Step 132: at() Method (ES2022)

```javascript
const str = 'Hello World';

console.log(str.at(0));    // 'H'
console.log(str.at(-1));   // 'd' (ตัวสุดท้าย)
console.log(str.at(-2));   // 'l'
console.log(str.at(100));  // undefined

// เปรียบเทียบกับ []
console.log(str[str.length - 1]);  // 'd' (วิธีเก่า)
console.log(str.at(-1));           // 'd' (วิธีใหม่ สั้นกว่า)

// ใช้กับ array ด้วย
const arr = [1, 2, 3, 4, 5];
console.log(arr.at(-1));   // 5
console.log(arr.at(-2));   // 4
```

---

## Step 133: Case Methods

### toUpperCase() และ toLowerCase()

```javascript
const text = 'Hello World JavaScript';

console.log(text.toUpperCase());  // 'HELLO WORLD JAVASCRIPT'
console.log(text.toLowerCase());  // 'hello world javascript'

// ต้นฉบับไม่เปลี่ยน
console.log(text);  // 'Hello World JavaScript'
```

```javascript
// ใช้งานจริง: normalize การเปรียบเทียบ
function equalsIgnoreCase(str1, str2) {
  return str1.toLowerCase() === str2.toLowerCase();
}

console.log(equalsIgnoreCase('Hello', 'hello'));   // true
console.log(equalsIgnoreCase('WORLD', 'World'));   // true
console.log(equalsIgnoreCase('JavaScript', 'java')); // false
```

```javascript
// Title Case
function toTitleCase(str) {
  return str
    .toLowerCase()
    .split(' ')
    .map(word => word.charAt(0).toUpperCase() + word.slice(1))
    .join(' ');
}

console.log(toTitleCase('hello world'));          // 'Hello World'
console.log(toTitleCase('the quick brown fox')); // 'The Quick Brown Fox'
```

```javascript
// Capitalize first letter เท่านั้น
function capitalize(str) {
  if (!str) return str;
  return str.charAt(0).toUpperCase() + str.slice(1).toLowerCase();
}

console.log(capitalize('hello'));           // 'Hello'
console.log(capitalize('HELLO WORLD'));     // 'Hello world'
```

```javascript
// camelCase, snake_case, kebab-case conversions
function toCamelCase(str) {
  return str
    .toLowerCase()
    .replace(/[^a-z0-9]+(.)/g, (_, char) => char.toUpperCase());
}

function toSnakeCase(str) {
  return str
    .replace(/([A-Z])/g, '_$1')
    .toLowerCase()
    .replace(/^_/, '');
}

function toKebabCase(str) {
  return str
    .replace(/([A-Z])/g, '-$1')
    .toLowerCase()
    .replace(/^-/, '');
}

console.log(toCamelCase('hello world'));       // 'helloWorld'
console.log(toCamelCase('the quick-brown_fox')); // 'theQuickBrownFox'
console.log(toSnakeCase('helloWorld'));         // 'hello_world'
console.log(toKebabCase('helloWorld'));         // 'hello-world'
```

---

## Step 134: Trim Methods

### trim(), trimStart(), trimEnd()

```javascript
const messy = '   Hello World   ';

console.log(messy.trim());      // 'Hello World'
console.log(messy.trimStart()); // 'Hello World   ' (ลบซ้าย)
console.log(messy.trimEnd());   // '   Hello World' (ลบขวา)

// aliases
console.log(messy.trimLeft());  // เหมือน trimStart
console.log(messy.trimRight()); // เหมือน trimEnd
```

```javascript
// ใช้งานจริง: clean user input
function cleanInput(input) {
  return input.trim().replace(/\s+/g, ' ');
}

console.log(cleanInput('  Hello   World  '));  // 'Hello World'

// validation หลัง trim
function validateName(name) {
  const trimmed = name.trim();
  if (!trimmed) return 'กรุณาระบุชื่อ';
  if (trimmed.length < 2) return 'ชื่อต้องมีอย่างน้อย 2 ตัวอักษร';
  return null;
}

console.log(validateName('   '));      // 'กรุณาระบุชื่อ'
console.log(validateName('  A  '));    // 'ชื่อต้องมีอย่างน้อย 2 ตัวอักษร'
console.log(validateName('  สมชาย  ')); // null (ผ่าน)
```

```javascript
// trim ตัวอักษรอื่นๆ (ไม่ใช่ whitespace)
function trimChar(str, char) {
  const escaped = char.replace(/[.*+?^${}()|[\]\\]/g, '\\$&');
  const regex = new RegExp(`^${escaped}+|${escaped}+$`, 'g');
  return str.replace(regex, '');
}

console.log(trimChar('***hello***', '*'));    // 'hello'
console.log(trimChar('---hello---', '-'));    // 'hello'
console.log(trimChar('...hello...', '.'));    // 'hello'
```

---

## Step 135: Search Methods

### indexOf() และ lastIndexOf()

```javascript
const text = 'Hello World, Hello JavaScript';

// indexOf คืน index แรกที่พบ (-1 ถ้าไม่พบ)
console.log(text.indexOf('Hello'));      // 0
console.log(text.indexOf('o'));          // 4
console.log(text.lastIndexOf('Hello'));  // 13 (ตำแหน่งสุดท้าย)
console.log(text.indexOf('Python'));     // -1

// ระบุจุดเริ่มต้นค้นหา
console.log(text.indexOf('Hello', 5));  // 13 (เริ่มหาจาก index 5)

// ตรวจสอบว่ามีอยู่
const hasHello = text.indexOf('Hello') !== -1;
console.log(hasHello);  // true
```

```javascript
// นับจำนวนครั้งที่ปรากฏ
function countOccurrences(str, substr) {
  let count = 0;
  let index = str.indexOf(substr);
  
  while (index !== -1) {
    count++;
    index = str.indexOf(substr, index + 1);
  }
  
  return count;
}

const sentence = 'the cat sat on the mat by the door';
console.log(countOccurrences(sentence, 'the'));  // 3
```

### includes()

```javascript
const text = 'สวัสดี โลก JavaScript';

console.log(text.includes('สวัสดี'));     // true
console.log(text.includes('JavaScript')); // true
console.log(text.includes('Python'));     // false

// case sensitive!
console.log(text.includes('javascript')); // false

// ระบุจุดเริ่มต้น
console.log(text.includes('สวัสดี', 1)); // false (เริ่มหาจาก index 1)
```

### startsWith() และ endsWith()

```javascript
const filename = 'document.pdf';
const url = 'https://example.com/api/users';

console.log(filename.endsWith('.pdf'));     // true
console.log(filename.endsWith('.jpg'));     // false
console.log(url.startsWith('https://'));   // true
console.log(url.startsWith('http://'));    // false

// ระบุ length
console.log('Hello World'.startsWith('Hello', 0));  // true
console.log('Hello World'.endsWith('World', 11));    // true

// ตัวอย่าง: ตรวจสอบไฟล์
function getFileType(filename) {
  const lower = filename.toLowerCase();
  if (lower.endsWith('.jpg') || lower.endsWith('.png') || lower.endsWith('.gif')) {
    return 'image';
  }
  if (lower.endsWith('.pdf')) return 'pdf';
  if (lower.endsWith('.doc') || lower.endsWith('.docx')) return 'document';
  return 'unknown';
}

console.log(getFileType('photo.JPG'));    // 'image'
console.log(getFileType('report.pdf'));   // 'pdf'
console.log(getFileType('script.js'));    // 'unknown'
```

### search()

```javascript
// search() ใช้ regex
const text = 'Hello World 123';

console.log(text.search(/\d+/));   // 12 (ตำแหน่งแรกของตัวเลข)
console.log(text.search(/xyz/));   // -1 (ไม่พบ)
console.log(text.search(/world/i)); // 6 (case insensitive)

// เปรียบเทียบกับ indexOf
// indexOf ใช้ string ธรรมดา
// search ใช้ regex (ยืดหยุ่นกว่า)
```

---

## Step 136: Extract Methods

### slice()

```javascript
const str = 'Hello World JavaScript';

// slice(start, end) - ไม่รวม end
console.log(str.slice(0, 5));    // 'Hello'
console.log(str.slice(6, 11));   // 'World'
console.log(str.slice(12));      // 'JavaScript'
console.log(str.slice(-10));     // 'JavaScript'
console.log(str.slice(-10, -5)); // 'JavaS'
console.log(str.slice(0, -11));  // 'Hello World'

// ต้นฉบับไม่เปลี่ยน
console.log(str);
```

```javascript
// ใช้งานจริง
function truncate(str, maxLength, suffix = '...') {
  if (str.length <= maxLength) return str;
  return str.slice(0, maxLength - suffix.length) + suffix;
}

console.log(truncate('Hello World JavaScript', 15));  // 'Hello World ...'
console.log(truncate('Hi', 15));                       // 'Hi'
console.log(truncate('Hello World!', 5, '…'));         // 'Hell…'
```

### substring()

```javascript
const str = 'Hello World';

// substring(start, end) - คล้าย slice แต่ไม่รับ negative
console.log(str.substring(0, 5));   // 'Hello'
console.log(str.substring(6, 11));  // 'World'
console.log(str.substring(6));      // 'World'

// ต่างจาก slice: ถ้า start > end จะสลับ
console.log(str.substring(5, 0));   // 'Hello' (สลับเป็น 0, 5)
console.log(str.slice(5, 0));       // '' (ว่าง ไม่สลับ)

// negative ถูกแปลงเป็น 0
console.log(str.substring(-3));     // 'Hello World' (เริ่มจาก 0)
console.log(str.slice(-3));         // 'rld'
```

### substr() (deprecated แต่ยังใช้ได้)

```javascript
const str = 'Hello World';

// substr(start, length) - ระบุจำนวนตัวอักษร
console.log(str.substr(0, 5));   // 'Hello' (5 ตัวจาก index 0)
console.log(str.substr(6, 5));   // 'World' (5 ตัวจาก index 6)
console.log(str.substr(-5));     // 'World' (5 ตัวท้าย)
console.log(str.substr(-5, 3));  // 'Wor' (3 ตัวจากท้าย 5)

// ควรใช้ slice แทน
```

---

## Step 137: Replace Methods

### replace()

```javascript
const text = 'Hello World Hello';

// แทนที่ครั้งแรก
console.log(text.replace('Hello', 'Hi'));  // 'Hi World Hello'

// แทนที่ด้วย regex
console.log(text.replace(/Hello/g, 'Hi'));  // 'Hi World Hi' (global flag)
console.log(text.replace(/hello/gi, 'Hi')); // 'Hi World Hi' (case insensitive)
```

```javascript
// Replacement function
const result = 'hello world'.replace(/\b\w/g, char => char.toUpperCase());
console.log(result);  // 'Hello World'

// Template replacement
const template = 'สวัสดี, {{name}}! คุณอายุ {{age}} ปี';
const data = { name: 'สมชาย', age: 25 };

const rendered = template.replace(/\{\{(\w+)\}\}/g, (_, key) => data[key] || '');
console.log(rendered);  // 'สวัสดี, สมชาย! คุณอายุ 25 ปี'
```

```javascript
// การใช้ $1, $2 (capturing groups)
const name = 'สมชาย ใจดี';
const swapped = name.replace(/(\S+)\s+(\S+)/, '$2 $1');
console.log(swapped);  // 'ใจดี สมชาย'

// format phone number
const phone = '0812345678';
const formatted = phone.replace(/(\d{3})(\d{3})(\d{4})/, '$1-$2-$3');
console.log(formatted);  // '081-234-5678'
```

### replaceAll()

```javascript
const text = 'apple banana apple orange apple';

// replaceAll ใน ES2021
console.log(text.replaceAll('apple', 'mango'));
// 'mango banana mango orange mango'

// เปรียบเทียบกับ replace + /g
console.log(text.replace(/apple/g, 'mango'));  // เหมือนกัน

// replaceAll ใช้ string ธรรมดา (ไม่ต้องใช้ regex)
const path = 'C:\\Users\\somchai\\Documents';
console.log(path.replaceAll('\\', '/'));
// 'C:/Users/somchai/Documents'
```

```javascript
// highlight text
function highlight(text, searchTerm) {
  const regex = new RegExp(`(${searchTerm})`, 'gi');
  return text.replace(regex, '<mark>$1</mark>');
}

const article = 'JavaScript is great. I love JavaScript.';
console.log(highlight(article, 'javascript'));
// 'JavaScript is great. I love JavaScript.'
// (พร้อม <mark> tags)
```

---

## Step 138: split() และ join()

### split()

```javascript
const csv = 'apple,banana,orange,mango';

// แบ่งด้วย separator
const fruits = csv.split(',');
console.log(fruits);  // ['apple', 'banana', 'orange', 'mango']

// แบ่งเป็นตัวอักษร
const chars = 'Hello'.split('');
console.log(chars);  // ['H', 'e', 'l', 'l', 'o']

// แบ่งด้วย regex
const words = 'one1two2three3four'.split(/\d/);
console.log(words);  // ['one', 'two', 'three', 'four']

// จำกัดจำนวน
const limited = 'a,b,c,d,e'.split(',', 3);
console.log(limited);  // ['a', 'b', 'c']
```

```javascript
// ใช้งานจริง: parse data
const logLine = '2024-01-15 10:30:45 ERROR Database connection failed';
const [date, time, level, ...messageParts] = logLine.split(' ');
const message = messageParts.join(' ');

console.log({ date, time, level, message });

// split แล้ว process แล้ว join กลับ
function reverseWords(sentence) {
  return sentence.split(' ').reverse().join(' ');
}

console.log(reverseWords('Hello World JavaScript'));
// 'JavaScript World Hello'
```

```javascript
// แบ่งหลาย separator
const text = 'apple;banana|orange,mango grape';
const items = text.split(/[;|, ]+/);
console.log(items);  // ['apple', 'banana', 'orange', 'mango', 'grape']

// split แล้วแปลง
const nums = '1,2,3,4,5'.split(',').map(Number);
console.log(nums);  // [1, 2, 3, 4, 5]
console.log(nums[0] + nums[1]);  // 3
```

---

## Step 139: padStart() และ padEnd()

```javascript
// padStart(targetLength, padString)
const num = '5';
console.log(num.padStart(3, '0'));   // '005'
console.log(num.padStart(5, '0'));   // '00005'
console.log(num.padStart(3, ' '));   // '  5'
console.log(num.padStart(3));        // '  5' (ค่าเริ่มต้นคือ space)

// padEnd(targetLength, padString)
const text = 'Hello';
console.log(text.padEnd(10));       // 'Hello     '
console.log(text.padEnd(10, '.'));  // 'Hello.....'
console.log(text.padEnd(10, '!-')); // 'Hello!-!-!'
```

```javascript
// ใช้งานจริง: format ตัวเลข
function formatNumber(n, digits = 3) {
  return String(n).padStart(digits, '0');
}

console.log(formatNumber(1));    // '001'
console.log(formatNumber(42));   // '042'
console.log(formatNumber(123));  // '123'
console.log(formatNumber(1234)); // '1234' (ไม่ตัด)

// สร้างตาราง
function formatTable(data) {
  const maxNameLen = Math.max(...data.map(d => d.name.length));
  return data.map(d => {
    const name = d.name.padEnd(maxNameLen + 2);
    const score = String(d.score).padStart(5);
    return `${name}${score}`;
  }).join('\n');
}

const students = [
  { name: 'สมชาย', score: 85 },
  { name: 'สมหญิง', score: 92 },
  { name: 'สมศักดิ์ ใจกล้า', score: 78 }
];

console.log(formatTable(students));
```

```javascript
// ซ่อนข้อมูลบัตรเครดิต
function maskCard(cardNumber) {
  const last4 = cardNumber.slice(-4);
  return last4.padStart(cardNumber.length, '*');
}

console.log(maskCard('1234567890123456'));  // '************3456'

// format เวลา
function formatTime(h, m, s) {
  return [h, m, s].map(n => String(n).padStart(2, '0')).join(':');
}

console.log(formatTime(9, 5, 3));    // '09:05:03'
console.log(formatTime(14, 30, 0));  // '14:30:00'
```

---

## Step 140: repeat()

```javascript
console.log('ab'.repeat(3));    // 'ababab'
console.log('Ha'.repeat(5));    // 'HaHaHaHaHa'
console.log('-'.repeat(20));    // '--------------------'
console.log('*'.repeat(0));     // '' (ว่าง)

// ใช้งานจริง
function createDivider(char = '-', length = 40) {
  return char.repeat(length);
}

function printHeader(title) {
  const divider = createDivider('=', title.length + 4);
  console.log(divider);
  console.log(`= ${title} =`);
  console.log(divider);
}

printHeader('รายงานประจำเดือน');
```

```javascript
// สร้าง progress bar
function progressBar(percent, width = 20) {
  const filled = Math.round((percent / 100) * width);
  const empty = width - filled;
  return `[${'█'.repeat(filled)}${'░'.repeat(empty)}] ${percent}%`;
}

console.log(progressBar(0));    // [░░░░░░░░░░░░░░░░░░░░] 0%
console.log(progressBar(50));   // [██████████░░░░░░░░░░] 50%
console.log(progressBar(100));  // [████████████████████] 100%
```

---

## Step 141: Template Literals ขั้นสูง

### Multi-line strings

```javascript
// ก่อน ES6 ต้องใช้ \n
const oldStyle = 'บรรทัดที่ 1\n' +
                 'บรรทัดที่ 2\n' +
                 'บรรทัดที่ 3';

// Template literal สะดวกกว่า
const newStyle = `บรรทัดที่ 1
บรรทัดที่ 2
บรรทัดที่ 3`;

console.log(newStyle);
```

```javascript
// Expression ใน template literal
const a = 10, b = 20;
console.log(`${a} + ${b} = ${a + b}`);  // '10 + 20 = 30'
console.log(`${a > b ? 'a มากกว่า' : 'b มากกว่าหรือเท่ากัน'}`);

// Function call
function formatCurrency(amount) {
  return amount.toLocaleString('th-TH', { style: 'currency', currency: 'THB' });
}

const price = 1299.50;
console.log(`ราคา: ${formatCurrency(price)}`);
```

```javascript
// Tagged Template Literals
function highlight(strings, ...values) {
  return strings.reduce((result, str, i) => {
    const value = values[i - 1];
    return result + (value ? `<strong>${value}</strong>` : '') + str;
  });
}

const name = 'สมชาย';
const age = 25;
const message = highlight`ชื่อ: ${name}, อายุ: ${age} ปี`;
console.log(message);
// 'ชื่อ: <strong>สมชาย</strong>, อายุ: <strong>25</strong> ปี'
```

```javascript
// SQL builder (ตัวอย่าง)
function sql(strings, ...values) {
  const params = [];
  const query = strings.reduce((result, str, i) => {
    if (i > 0) {
      params.push(values[i - 1]);
      return result + `$${i}`;
    }
    return result + str;
  });
  
  return { query: query + strings[strings.length - 1], params };
}

const userId = 42;
const name2 = 'สมชาย';
const { query, params } = sql`SELECT * FROM users WHERE id = ${userId} AND name = ${name2}`;
console.log(query);   // 'SELECT * FROM users WHERE id = $1 AND name = $2'
console.log(params);  // [42, 'สมชาย']
```

---

## Step 142: String Comparison

```javascript
// เปรียบเทียบแบบ lexicographic
console.log('apple' < 'banana');  // true
console.log('z' > 'a');           // true
console.log('ABC' < 'abc');       // true (uppercase มี code น้อยกว่า)

// localeCompare สำหรับภาษาต่างๆ
const fruits = ['มะม่วง', 'แอปเปิ้ล', 'กล้วย', 'ส้ม'];
fruits.sort((a, b) => a.localeCompare(b, 'th'));
console.log(fruits);  // เรียงตามลำดับภาษาไทย

// localeCompare ด้วย options
const names = ['café', 'cafe', 'Café'];
names.sort((a, b) => a.localeCompare(b, 'fr', { sensitivity: 'base' }));
console.log(names);  // เรียงโดยไม่สนใจ accent
```

```javascript
// เปรียบเทียบ version strings
function compareVersions(v1, v2) {
  const parts1 = v1.split('.').map(Number);
  const parts2 = v2.split('.').map(Number);
  
  for (let i = 0; i < Math.max(parts1.length, parts2.length); i++) {
    const p1 = parts1[i] || 0;
    const p2 = parts2[i] || 0;
    if (p1 !== p2) return p1 > p2 ? 1 : -1;
  }
  return 0;
}

console.log(compareVersions('1.0.0', '1.0.1'));   // -1
console.log(compareVersions('2.0.0', '1.9.9'));   // 1
console.log(compareVersions('1.0.0', '1.0.0'));   // 0
```

---

## Step 143: Unicode และ Escape Sequences

```javascript
// Escape sequences
console.log('Hello\nWorld');    // ขึ้นบรรทัดใหม่
console.log('Tab\there');       // tab
console.log('Quote: \'hello\'');// single quote
console.log('Backslash: \\');   // backslash
console.log('\u0041');          // 'A' (Unicode)
console.log('\u0E2A');          // 'ส' (ภาษาไทย)
console.log('\u{1F600}');       // '😀' (emoji ด้วย ES6 syntax)
```

```javascript
// Unicode methods
const str = '😀 Hello สวัสดี';

// codePointAt สำหรับ surrogate pairs
console.log(str.codePointAt(0));  // 128512 (emoji code point)
console.log(str.charCodeAt(0));   // 55357 (high surrogate)
console.log(str.charCodeAt(1));   // 56832 (low surrogate)

// String.fromCodePoint
console.log(String.fromCodePoint(128512));  // '😀'
console.log(String.fromCodePoint(0x0E2A));  // 'ส'

// normalize (สำหรับ accented characters)
const cafe1 = 'caf\u00E9';       // café (single character é)
const cafe2 = 'cafe\u0301';      // café (e + combining accent)
console.log(cafe1 === cafe2);    // false
console.log(cafe1.normalize() === cafe2.normalize());  // true
```

```javascript
// นับตัวอักษรที่แท้จริง (รวม emoji)
function countChars(str) {
  return [...str].length;
}

const text = '😀 Hello 😊';
console.log(text.length);       // 12 (นับ code units)
console.log(countChars(text));  // 10 (นับตัวอักษรจริง)

// วนซ้ำตัวอักษรรวม emoji
for (const char of text) {
  process.stdout.write(`[${char}]`);
}
```

---

## Step 144: Regular Expressions กับ String

```javascript
// match() - หาทุกตัวที่ตรงกับ pattern
const text = 'Today is 2024-01-15 and tomorrow is 2024-01-16';

const dates = text.match(/\d{4}-\d{2}-\d{2}/g);
console.log(dates);  // ['2024-01-15', '2024-01-16']

const emails = 'contact: a@test.com, b@test.com'.match(/[\w.]+@[\w.]+/g);
console.log(emails);  // ['a@test.com', 'b@test.com']
```

```javascript
// matchAll() - คืน iterator ของ matches (ES2020)
const str = 'JavaScript Python Ruby JavaScript';
const matches = [...str.matchAll(/JavaScript/g)];

matches.forEach(match => {
  console.log(`พบ "${match[0]}" ที่ index ${match.index}`);
});
// พบ "JavaScript" ที่ index 0
// พบ "JavaScript" ที่ index 21
```

```javascript
// test() ด้วย RegExp object
const emailRegex = /^[\w.-]+@[\w.-]+\.\w+$/;
const phoneRegex = /^0\d{9}$/;
const thaiRegex = /^[\u0E00-\u0E7F\s]+$/;

function validateEmail(email) {
  return emailRegex.test(email);
}

function validatePhone(phone) {
  return phoneRegex.test(phone.replace(/[-\s]/g, ''));
}

function isThai(text) {
  return thaiRegex.test(text);
}

console.log(validateEmail('test@example.com'));  // true
console.log(validateEmail('invalid-email'));      // false
console.log(validatePhone('081-234-5678'));       // true
console.log(isThai('สวัสดี'));                    // true
console.log(isThai('Hello'));                     // false
```

---

## Step 145: String Utilities ที่ใช้บ่อย

```javascript
// word count
function wordCount(text) {
  return text.trim().split(/\s+/).filter(Boolean).length;
}

console.log(wordCount('Hello World'));          // 2
console.log(wordCount('  Hello   World  '));    // 2
console.log(wordCount(''));                     // 0

// อ่านเป็นตัวเลข
function parseThaiNumber(str) {
  const thaiDigits = '๐๑๒๓๔๕๖๗๘๙';
  return Number(
    str.replace(/[๐-๙]/g, d => thaiDigits.indexOf(d))
  );
}

console.log(parseThaiNumber('๑๒๓'));  // 123
```

```javascript
// slug generator
function createSlug(str) {
  return str
    .toLowerCase()
    .replace(/\s+/g, '-')
    .replace(/[^\w-]/g, '')
    .replace(/-+/g, '-')
    .replace(/^-|-$/g, '');
}

console.log(createSlug('Hello World!'));      // 'hello-world'
console.log(createSlug('  My First Post  ')); // 'my-first-post'

// email masking
function maskEmail(email) {
  const [local, domain] = email.split('@');
  const masked = local[0] + '***' + local[local.length - 1];
  return `${masked}@${domain}`;
}

console.log(maskEmail('somchai@example.com'));  // 's***i@example.com'
```

```javascript
// string interpolation ขั้นสูง
function interpolate(template, data) {
  return template.replace(/\${(\w+)}/g, (_, key) => {
    return data.hasOwnProperty(key) ? data[key] : '';
  });
}

const tmpl = 'สวัสดี ${name}! คุณอายุ ${age} ปี อาศัยอยู่ที่ ${city}';
const info = { name: 'สมชาย', age: 25, city: 'กรุงเทพ' };
console.log(interpolate(tmpl, info));
// 'สวัสดี สมชาย! คุณอายุ 25 ปี อาศัยอยู่ที่ กรุงเทพ'
```

---

## Step 146: String Formatting

```javascript
// Number formatting
const price = 1234567.89;
console.log(price.toLocaleString('th-TH'));  // '1,234,567.89'
console.log(price.toLocaleString('th-TH', {
  style: 'currency',
  currency: 'THB'
}));  // '฿1,234,567.89'

// Date formatting (ดูรายละเอียดในบท Date)
const date = new Date();
console.log(date.toLocaleDateString('th-TH'));  // '15/1/2567'
```

```javascript
// String.raw สำหรับ raw string (ไม่ process escape)
const rawPath = String.raw`C:\Users\somchai\Documents`;
console.log(rawPath);  // 'C:\Users\somchai\Documents'

// เปรียบเทียบ
console.log(`C:\Users\somchai`);    // 'C:UsersSomchai' (escape processed)
console.log(String.raw`C:\Users`);  // 'C:\Users' (raw)
```

```javascript
// format ตัวเลขด้วย padding
function formatTable(headers, rows) {
  const widths = headers.map((h, i) => {
    const colValues = rows.map(r => String(r[i]).length);
    return Math.max(h.length, ...colValues);
  });
  
  const formatRow = row => row.map((cell, i) => 
    String(cell).padEnd(widths[i])
  ).join(' | ');
  
  const separator = widths.map(w => '-'.repeat(w)).join('-+-');
  
  return [
    formatRow(headers),
    separator,
    ...rows.map(formatRow)
  ].join('\n');
}

const headers = ['ชื่อ', 'คะแนน', 'เกรด'];
const rows = [
  ['สมชาย', 85, 'B'],
  ['สมหญิง', 92, 'A'],
  ['สมศักดิ์', 78, 'C+']
];

console.log(formatTable(headers, rows));
```

---

## Step 147: String Parsing

```javascript
// JSON parse/stringify
const obj = { name: 'สมชาย', age: 25, skills: ['JS', 'Python'] };
const jsonStr = JSON.stringify(obj);
console.log(jsonStr);  // '{"name":"สมชาย","age":25,...}'

const parsed = JSON.parse(jsonStr);
console.log(parsed.name);  // 'สมชาย'

// Pretty print
const pretty = JSON.stringify(obj, null, 2);
console.log(pretty);
```

```javascript
// URL parsing
const url = 'https://example.com:8080/path/to/page?query=hello&page=2#section';

// ใช้ URL API (ในบราวเซอร์หรือ Node.js)
const urlObj = new URL(url);
console.log(urlObj.protocol);  // 'https:'
console.log(urlObj.hostname);  // 'example.com'
console.log(urlObj.port);      // '8080'
console.log(urlObj.pathname);  // '/path/to/page'
console.log(urlObj.search);    // '?query=hello&page=2'
console.log(urlObj.hash);      // '#section'

// query parameters
console.log(urlObj.searchParams.get('query'));  // 'hello'
console.log(urlObj.searchParams.get('page'));   // '2'
```

---

## Step 148: String Validation ขั้นสูง

```javascript
// Luhn algorithm สำหรับตรวจบัตรเครดิต
function isValidCreditCard(cardNumber) {
  const digits = cardNumber.replace(/\D/g, '');
  let sum = 0;
  let isEven = false;
  
  for (let i = digits.length - 1; i >= 0; i--) {
    let digit = parseInt(digits[i]);
    
    if (isEven) {
      digit *= 2;
      if (digit > 9) digit -= 9;
    }
    
    sum += digit;
    isEven = !isEven;
  }
  
  return sum % 10 === 0;
}

console.log(isValidCreditCard('4111111111111111'));  // true (Visa test)
console.log(isValidCreditCard('1234567890123456'));  // false
```

```javascript
// Password strength checker
function checkPasswordStrength(password) {
  const checks = {
    length: password.length >= 8,
    uppercase: /[A-Z]/.test(password),
    lowercase: /[a-z]/.test(password),
    numbers: /\d/.test(password),
    special: /[!@#$%^&*]/.test(password)
  };
  
  const score = Object.values(checks).filter(Boolean).length;
  const passed = Object.entries(checks)
    .filter(([, v]) => !v)
    .map(([k]) => k);
  
  const strength = ['Very Weak', 'Weak', 'Fair', 'Good', 'Strong', 'Very Strong'][score];
  
  return { score, strength, passed, checks };
}

const result = checkPasswordStrength('MyP@ss123');
console.log(result.strength);  // 'Very Strong'
```

---

## Step 149: Performance Tips

```javascript
// String concatenation ใน loop
// วิธีช้า:
let slow = '';
for (let i = 0; i < 10000; i++) {
  slow += 'x';  // สร้าง string ใหม่ทุกครั้ง!
}

// วิธีเร็ว:
const parts = [];
for (let i = 0; i < 10000; i++) {
  parts.push('x');
}
const fast = parts.join('');

// หรือใช้ repeat:
const fastest = 'x'.repeat(10000);
```

```javascript
// ใช้ template literal แทน concatenation
const name = 'สมชาย';
const age = 25;

// ช้ากว่า (ในบางกรณี):
const str1 = 'ชื่อ: ' + name + ', อายุ: ' + age;

// อ่านง่ายกว่า:
const str2 = `ชื่อ: ${name}, อายุ: ${age}`;
```

---

## Step 150: String ในชีวิตจริง

```javascript
// Parser สำหรับ CSV
function parseCSV(csv, delimiter = ',') {
  const lines = csv.trim().split('\n');
  const headers = lines[0].split(delimiter).map(h => h.trim());
  
  return lines.slice(1).map(line => {
    const values = line.split(delimiter).map(v => v.trim());
    return headers.reduce((obj, header, i) => {
      obj[header] = values[i];
      return obj;
    }, {});
  });
}

const csvData = `name,age,city
สมชาย,25,กรุงเทพ
สมหญิง,30,เชียงใหม่
สมศักดิ์,28,ภูเก็ต`;

const parsed = parseCSV(csvData);
console.log(parsed);
```

```javascript
// HTML escaping
function escapeHtml(str) {
  const entities = {
    '&': '&amp;',
    '<': '&lt;',
    '>': '&gt;',
    '"': '&quot;',
    "'": '&#39;'
  };
  return str.replace(/[&<>"']/g, char => entities[char]);
}

function unescapeHtml(str) {
  return str
    .replace(/&amp;/g, '&')
    .replace(/&lt;/g, '<')
    .replace(/&gt;/g, '>')
    .replace(/&quot;/g, '"')
    .replace(/&#39;/g, "'");
}

console.log(escapeHtml('<script>alert("XSS")</script>'));
// '&lt;script&gt;alert(&quot;XSS&quot;)&lt;/script&gt;'
```

```javascript
// Text highlighter
function highlightKeywords(text, keywords) {
  const pattern = new RegExp(`(${keywords.join('|')})`, 'gi');
  return text.replace(pattern, '<span class="highlight">$1</span>');
}

const article = 'JavaScript is a programming language. JavaScript is used for web development.';
const result = highlightKeywords(article, ['JavaScript', 'programming']);
console.log(result);

// String Builder (สำหรับสร้าง string ขนาดใหญ่)
class StringBuilder {
  #parts = [];
  
  append(str) {
    this.#parts.push(str);
    return this;
  }
  
  appendLine(str) {
    this.#parts.push(str + '\n');
    return this;
  }
  
  prepend(str) {
    this.#parts.unshift(str);
    return this;
  }
  
  toString() {
    return this.#parts.join('');
  }
  
  get length() {
    return this.toString().length;
  }
  
  clear() {
    this.#parts = [];
    return this;
  }
}

const sb = new StringBuilder();
sb.appendLine('บรรทัดที่ 1')
  .appendLine('บรรทัดที่ 2')
  .append('บรรทัดที่ 3');

console.log(sb.toString());
console.log(`ความยาว: ${sb.length}`);
```

---

## แบบฝึกหัด (Exercises)

### ระดับง่าย

**แบบฝึกหัด 1:** เขียน function `isPalindrome(str)` ที่ตรวจสอบว่า string เป็น palindrome หรือไม่ (โดยไม่สนใจตัวพิมพ์เล็ก/ใหญ่และช่องว่าง)

**แบบฝึกหัด 2:** เขียน function `countVowels(str)` ที่นับสระ (a, e, i, o, u) ใน string

**แบบฝึกหัด 3:** เขียน function `reverseWords(str)` ที่กลับลำดับคำใน string

**แบบฝึกหัด 4:** เขียน function `compress(str)` ที่บีบอัด string เช่น "aaabbbcc" → "a3b3c2"

**แบบฝึกหัด 5:** เขียน function `truncateWords(str, n)` ที่ตัดคำให้เหลือ n คำแรกแล้วใส่ "..."

### ระดับกลาง

**แบบฝึกหัด 6:** เขียน function `parseQueryString(url)` ที่แปลง query string เป็น object

**แบบฝึกหัด 7:** เขียน function `formatCurrency(amount, currency, locale)` ที่ format ตัวเลขเป็นสกุลเงิน

**แบบฝึกหัด 8:** เขียน function `validatePassword(password)` ที่ตรวจสอบความปลอดภัยของรหัสผ่าน

**แบบฝึกหัด 9:** เขียน Markdown parser อย่างง่าย (แค่ bold, italic, code)

### ระดับยาก

**แบบฝึกหัด 10:** เขียน function `diff(str1, str2)` ที่แสดงความแตกต่างระหว่างสอง strings

**แบบฝึกหัด 11:** เขียน wildcard matcher `matches(pattern, str)` ที่รองรับ `*` (หลายตัว) และ `?` (หนึ่งตัว)

### เฉลยบางส่วน

```javascript
// แบบฝึกหัด 1
function isPalindrome(str) {
  const cleaned = str.toLowerCase().replace(/[^a-z0-9]/g, '');
  return cleaned === cleaned.split('').reverse().join('');
}

// แบบฝึกหัด 2
function countVowels(str) {
  return (str.match(/[aeiouAEIOU]/g) || []).length;
}

// แบบฝึกหัด 4
function compress(str) {
  return str.replace(/(.)\1+/g, (match, char) => `${char}${match.length}`);
}

// แบบฝึกหัด 5
function truncateWords(str, n) {
  const words = str.trim().split(/\s+/);
  if (words.length <= n) return str;
  return words.slice(0, n).join(' ') + '...';
}

// แบบฝึกหัด 6
function parseQueryString(queryString) {
  return queryString
    .replace(/^\?/, '')
    .split('&')
    .reduce((obj, pair) => {
      const [key, value] = pair.split('=').map(decodeURIComponent);
      obj[key] = value;
      return obj;
    }, {});
}

console.log(parseQueryString('?name=สมชาย&age=25&city=กรุงเทพ'));
// { name: 'สมชาย', age: '25', city: 'กรุงเทพ' }
```

---

## สรุป

ในบทนี้เราได้เรียนรู้ string methods ที่สำคัญทั้งหมด:
- Case methods: toUpperCase, toLowerCase
- Trim methods: trim, trimStart, trimEnd
- Search methods: indexOf, lastIndexOf, includes, startsWith, endsWith, search
- Extract methods: slice, substring, substr
- Replace methods: replace, replaceAll
- split และ join
- Padding: padStart, padEnd
- repeat และ at()
- Template literals ขั้นสูง
- Unicode และ escape sequences
- Regular expressions กับ String

String manipulation เป็นทักษะที่จำเป็นมากสำหรับการพัฒนาโปรแกรม ตั้งแต่การ validate input ไปจนถึงการ parse data และสร้าง output
