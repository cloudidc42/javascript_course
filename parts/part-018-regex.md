# Part 18: Regular Expressions พื้นฐาน
## Steps 331-350

---

## บทนำ

Regular Expressions (Regex หรือ RegExp) คือรูปแบบ (pattern) สำหรับค้นหาและจับคู่ข้อความ เป็นเครื่องมือที่ทรงพลังสำหรับการ validate input, ค้นหาและแทนที่ข้อความ, แยกข้อมูลจาก strings และอีกมากมาย

---

## Step 331: Regular Expressions คืออะไร

```javascript
// Regular Expression คือ pattern สำหรับ match ข้อความ

// ตัวอย่างง่ายๆ - หาคำว่า "hello"
const pattern = /hello/;
const text = "say hello to the world";

console.log(pattern.test(text));   // true
console.log(/world/.test(text));   // true
console.log(/goodbye/.test(text)); // false

// Regex มีประโยชน์ใน:
// 1. Validation (email, phone, password)
const email = "somchai@example.com";
const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
console.log(emailRegex.test(email)); // true

// 2. Search and replace
const sentence = "กล้วย มะม่วง กล้วย ส้ม";
console.log(sentence.replace(/กล้วย/g, "ฝรั่ง"));
// "ฝรั่ง มะม่วง ฝรั่ง ส้ม"

// 3. Extracting data
const dateStr = "วันที่ 15 มกราคม 2024";
const numbers = dateStr.match(/\d+/g);
console.log(numbers); // ["15", "2024"]

// 4. Splitting text
const csv = "สมชาย,25,กรุงเทพ,ไทย";
const parts = csv.split(/,/);
console.log(parts); // ["สมชาย", "25", "กรุงเทพ", "ไทย"]
```

---

## Step 332: การสร้าง Regular Expression

### วิธีที่ 1: Literal Syntax

```javascript
// /pattern/flags
const regex1 = /hello/;          // ธรรมดา
const regex2 = /hello/i;         // case-insensitive
const regex3 = /hello/g;         // global (หาทั้งหมด)
const regex4 = /hello/gi;        // global + case-insensitive

// ใช้งาน
console.log(/thai/i.test("Thai Food")); // true
console.log(/THAI/i.test("thai"));      // true

// Literal regex ถูก compile ณ เวลา load
// ใช้เมื่อ pattern รู้แน่นอนแล้ว
```

### วิธีที่ 2: RegExp Constructor

```javascript
// new RegExp(pattern, flags)
const pattern = "hello";
const regex = new RegExp(pattern, "gi");
console.log(regex.test("Hello World")); // true

// ประโยชน์: สร้าง dynamic patterns
function createSearchRegex(query) {
  // Escape special regex characters
  const escaped = query.replace(/[.*+?^${}()|[\]\\]/g, '\\$&');
  return new RegExp(escaped, 'gi');
}

const searchRegex = createSearchRegex("c++");
console.log(searchRegex.test("learning c++")); // true

// ใช้ตัวแปรใน pattern
const word = "javascript";
const wordRegex = new RegExp(`\\b${word}\\b`, 'i');
console.log(wordRegex.test("I love JavaScript")); // true

// เปรียบเทียบ
const r1 = /hello/gi;
const r2 = new RegExp("hello", "gi");
// ทำงานเหมือนกัน แต่ r2 ยืดหยุ่นกว่า
```

---

## Step 333: test() Method

```javascript
// test() คืน true/false - ใช้เมื่อต้องการแค่ตรวจว่า match หรือไม่

const hasNumber = /\d/.test("abc123");  // true
const allDigits = /^\d+$/.test("12345"); // true
const noDigits = /^\d+$/.test("abc");   // false

// ตัวอย่างการ validate
function validateEmail(email) {
  return /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/.test(email);
}

console.log(validateEmail("test@example.com"));    // true
console.log(validateEmail("invalid-email"));       // false
console.log(validateEmail("test@.com"));           // false

function validateThaiPhone(phone) {
  return /^0[6-9]\d{8}$/.test(phone.replace(/[-\s]/g, ''));
}

console.log(validateThaiPhone("0812345678"));   // true
console.log(validateThaiPhone("081-234-5678")); // true (after strip)
console.log(validateThaiPhone("1234567890"));   // false

// Regex เก็บ state ด้วย global flag - ระวัง!
const globalRegex = /hello/g;
const text = "hello world hello";

console.log(globalRegex.test(text)); // true (lastIndex: 5)
console.log(globalRegex.test(text)); // true (lastIndex: 17)
console.log(globalRegex.test(text)); // false (lastIndex reset to 0)
console.log(globalRegex.test(text)); // true (เริ่มใหม่)

// หลีกเลี่ยงโดย:
// 1. สร้าง regex ใหม่ทุกครั้ง
// 2. Reset lastIndex ก่อนใช้
globalRegex.lastIndex = 0;

// หรือ ไม่ใช้ global flag กับ test()
```

---

## Step 334: match() และ matchAll()

```javascript
// match() - หาผลลัพธ์ที่ match

const str = "Hello World Hello";

// ไม่มี g flag - คืน first match พร้อม details
const match = str.match(/hello/i);
console.log(match[0]);     // "Hello" (full match)
console.log(match.index);  // 0 (position)
console.log(match.input);  // "Hello World Hello"

// มี g flag - คืน array ของ matches ทั้งหมด
const matches = str.match(/hello/gi);
console.log(matches); // ["Hello", "Hello"]
console.log(matches.length); // 2

// ไม่ match คืน null
const noMatch = str.match(/xyz/);
console.log(noMatch); // null

// ตัวอย่างดึงข้อมูล
const html = '<a href="https://example.com">Link 1</a> <a href="https://test.com">Link 2</a>';
const links = html.match(/href="([^"]+)"/g);
console.log(links); // ['href="https://example.com"', 'href="https://test.com"']

// ดึงตัวเลขทั้งหมด
const text = "ราคา 500 บาท ลด 20 บาท เหลือ 480 บาท";
const nums = text.match(/\d+/g);
console.log(nums); // ["500", "20", "480"]
const total = nums.reduce((sum, n) => sum + parseInt(n), 0);

// matchAll() - คืน iterator ที่มีรายละเอียดทุก match
const str2 = "2024-01-15 and 2024-02-20";
const dateRegex = /(\d{4})-(\d{2})-(\d{2})/g;

for (const match of str2.matchAll(dateRegex)) {
  console.log(`Full: ${match[0]}`);    // "2024-01-15"
  console.log(`Year: ${match[1]}`);    // "2024"
  console.log(`Month: ${match[2]}`);   // "01"
  console.log(`Day: ${match[3]}`);     // "15"
  console.log(`At index: ${match.index}`);
}

// แปลงเป็น array
const allMatches = [...str2.matchAll(dateRegex)];
console.log(allMatches.length); // 2
```

---

## Step 335: search() Method

```javascript
// search() คืน index ของ match แรก หรือ -1 ถ้าไม่เจอ

const text = "สวัสดี Hello World";

console.log(text.search(/Hello/));  // 8 (index ที่เจอ)
console.log(text.search(/hello/i)); // 8 (case insensitive)
console.log(text.search(/xyz/));    // -1 (ไม่เจอ)

// เปรียบเทียบกับ indexOf
console.log(text.indexOf("Hello")); // 8
// search รับ regex, indexOf รับ string

// ประโยชน์ของ search: ใช้ pattern ซับซ้อนได้
const hasNumber = text.search(/\d/) !== -1;
const hasUpperCase = text.search(/[A-Z]/) !== -1;
const startsWithThai = text.search(/^[฀-๿]/) === 0;

console.log(hasNumber);     // false
console.log(hasUpperCase);  // true
console.log(startsWithThai); // true

// ตัวอย่างการใช้
function findFirstUrl(text) {
  const pos = text.search(/https?:\/\/\S+/);
  if (pos === -1) return null;
  return { position: pos, found: true };
}

const article = "อ่านเพิ่มเติมที่ https://example.com นะครับ";
console.log(findFirstUrl(article));
// { position: 18, found: true }
```

---

## Step 336: replace() และ replaceAll()

```javascript
// replace() - แทนที่ match แรก (หรือทั้งหมดถ้าใช้ g flag)

const str = "foo bar foo baz foo";

// แทนที่ครั้งแรก
console.log(str.replace("foo", "qux"));      // "qux bar foo baz foo"
console.log(str.replace(/foo/, "qux"));      // "qux bar foo baz foo"

// แทนที่ทั้งหมด (g flag)
console.log(str.replace(/foo/g, "qux"));     // "qux bar qux baz qux"

// replaceAll() - แทนที่ทั้งหมด (ES2021+)
console.log(str.replaceAll("foo", "qux"));   // "qux bar qux baz qux"

// replace กับ function
const result = "hello world".replace(/\b\w/g, char => char.toUpperCase());
console.log(result); // "Hello World"

// Capture groups ใน replacement
const name = "สมชาย ใจดี";
// แปลง "ชื่อ นามสกุล" เป็น "นามสกุล, ชื่อ"
const swapped = name.replace(/^(\S+)\s+(.+)$/, '$2, $1');
console.log(swapped); // "ใจดี, สมชาย"

// Named groups ใน replacement
const dateStr = "2024-01-15";
const formatted = dateStr.replace(
  /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/,
  '$<day>/$<month>/$<year>'
);
console.log(formatted); // "15/01/2024"

// Replace กับ function ที่ซับซ้อน
function highlight(text, keyword) {
  const regex = new RegExp(`(${keyword})`, 'gi');
  return text.replace(regex, '<mark>$1</mark>');
}

console.log(highlight("I love JavaScript and Java", "java"));
// "I love <mark>Java</mark>Script and <mark>Java</mark>"

// Template literals replace
const template = "สวัสดี {name}! คุณมีอายุ {age} ปี";
const data = { name: "สมชาย", age: 25 };

const filled = template.replace(/\{(\w+)\}/g, (_, key) => data[key] || '');
console.log(filled); // "สวัสดี สมชาย! คุณมีอายุ 25 ปี"
```

---

## Step 337: split() กับ Regex

```javascript
// split() กับ regex ทำให้ยืดหยุ่นกว่า string

// แยกด้วย whitespace หลายชนิด
const text = "hello\tworld\nfoo  bar";
console.log(text.split(/\s+/));
// ["hello", "world", "foo", "bar"]

// แยกด้วยหลาย separator
const csv = "สมชาย, 25 , กรุงเทพ , ไทย";
const parts = csv.split(/\s*,\s*/);
console.log(parts); // ["สมชาย", "25", "กรุงเทพ", "ไทย"]

// แยกประโยค
const paragraph = "This is sentence one. This is two! Is this three?";
const sentences = paragraph.split(/[.!?]+\s*/);
console.log(sentences.filter(s => s.trim()));
// ["This is sentence one", "This is two", "Is this three", ""]

// Split พร้อม capture group - เก็บ separator ด้วย
const str = "one1two2three3four";
console.log(str.split(/(\d)/));
// ["one", "1", "two", "2", "three", "3", "four"]

// Tokenizer ง่ายๆ
function tokenize(code) {
  return code.split(/(\s+|[(){}\[\];,]|"[^"]*"|'[^']*'|\d+\.\d+|\d+|\w+|[^\s\w])/)
    .filter(token => token.trim());
}

const code = 'let x = 42;';
console.log(tokenize(code));
// ["let", "x", "=", "42", ";"]
```

---

## Step 338: Character Classes

```javascript
// [] - character class - match any one character

// Basic character class
console.log(/[abc]/.test("apple")); // true (has 'a')
console.log(/[abc]/.test("dog"));   // false

// Range
console.log(/[a-z]/.test("Hello")); // true (has lowercase)
console.log(/[A-Z]/.test("hello")); // false (no uppercase)
console.log(/[0-9]/.test("abc42")); // true

// Multiple ranges
console.log(/[a-zA-Z]/.test("Hello123")); // true
console.log(/[a-zA-Z0-9]/.test("!@#"));   // false

// Negated class [^...]
console.log(/[^0-9]/.test("abc123")); // true (has non-digits)
console.log(/[^0-9]/.test("12345"));  // false (all digits)

// Predefined classes
console.log(/\d/.test("abc5def")); // true - digit [0-9]
console.log(/\D/.test("12345"));   // false - non-digit [^0-9]
console.log(/\w/.test("hello_1")); // true - word char [a-zA-Z0-9_]
console.log(/\W/.test("hello"));   // false - non-word
console.log(/\s/.test("a b"));     // true - whitespace [ \t\n\r\f\v]
console.log(/\S/.test("   "));     // false - non-whitespace

// Thai characters
const thaiRange = /[฀-๿]/;
console.log(thaiRange.test("สวัสดี"));  // true
console.log(thaiRange.test("hello"));   // false

// Mixed Thai and English
const thaiOrEng = /[a-zA-Z฀-๿]/;
console.log(thaiOrEng.test("สวัสดี hello")); // true

// ตัวอย่างการใช้
function countVowels(str) {
  const matches = str.match(/[aeiouAEIOU]/g);
  return matches ? matches.length : 0;
}
console.log(countVowels("Hello World")); // 3

function removeSpecialChars(str) {
  return str.replace(/[^a-zA-Z0-9฀-๿\s]/g, '');
}
console.log(removeSpecialChars("Hello! สวัสดี @#$")); // "Hello สวัสดี "

// Hex colors
function isHexColor(str) {
  return /^#[0-9A-Fa-f]{3}([0-9A-Fa-f]{3})?$/.test(str);
}
console.log(isHexColor("#fff"));     // true
console.log(isHexColor("#FF5733")); // true
console.log(isHexColor("#gg1234")); // false
```

---

## Step 339: Anchors

```javascript
// Anchors - ตำแหน่งใน string ไม่ใช่ characters

// ^ - เริ่มต้น string (หรือบรรทัด ถ้าใช้ m flag)
console.log(/^hello/.test("hello world")); // true
console.log(/^hello/.test("say hello"));  // false
console.log(/^hello/.test("Hello World")); // false

// $ - สิ้นสุด string (หรือบรรทัด)
console.log(/world$/.test("hello world")); // true
console.log(/world$/.test("world peace")); // false

// ใช้ ^ และ $ พร้อมกัน - match ทั้ง string
console.log(/^\d+$/.test("12345"));     // true (all digits)
console.log(/^\d+$/.test("123abc"));    // false
console.log(/^[a-z]{3,}$/.test("abc")); // true

// \b - word boundary (ระหว่าง \w และ \W)
console.log(/\bcat\b/.test("the cat sat")); // true
console.log(/\bcat\b/.test("concatenate")); // false
console.log(/\bcat\b/.test("cat-nap"));     // true

// \B - NOT word boundary
console.log(/\Bcat\B/.test("concatenate")); // true
console.log(/\Bcat\B/.test("the cat sat")); // false

// ตัวอย่าง
function startsWithUpperCase(str) {
  return /^[A-Z]/.test(str);
}

function endsWithPunctuation(str) {
  return /[.!?]$/.test(str);
}

function isWholeWord(text, word) {
  return new RegExp(`\\b${word}\\b`, 'i').test(text);
}

console.log(startsWithUpperCase("Hello"));     // true
console.log(startsWithUpperCase("hello"));     // false
console.log(endsWithPunctuation("Hello!"));    // true
console.log(isWholeWord("I eat cats", "cat")); // true
console.log(isWholeWord("I concatenate", "cat")); // false

// Multiline mode
const multiline = `line one
line two
line three`;

console.log(multiline.match(/^line/gm));
// ["line", "line", "line"] (with m flag)
console.log(multiline.match(/^line/g));
// ["line"] (without m flag - only first line)
```

---

## Step 340: Quantifiers

```javascript
// Quantifiers - จำนวนครั้งที่ match

// * - 0 หรือมากกว่า
console.log(/go*d/.test("gd"));     // true (0 'o')
console.log(/go*d/.test("good"));   // true (2 'o')
console.log(/go*d/.test("gooood")); // true (4 'o')

// + - 1 หรือมากกว่า
console.log(/go+d/.test("gd"));     // false (need at least 1 'o')
console.log(/go+d/.test("god"));    // true
console.log(/go+d/.test("gooood")); // true

// ? - 0 หรือ 1
console.log(/colou?r/.test("color"));  // true (0 'u')
console.log(/colou?r/.test("colour")); // true (1 'u')

// {n} - ตรงๆ n ครั้ง
console.log(/\d{4}/.test("1234"));   // true (exactly 4 digits)
console.log(/\d{4}/.test("123"));    // false
console.log(/\d{4}/.test("12345")); // true (has 4+ digits)

// {n,} - อย่างน้อย n ครั้ง
console.log(/\d{3,}/.test("12"));   // false
console.log(/\d{3,}/.test("123"));  // true
console.log(/\d{3,}/.test("12345")); // true

// {n,m} - ระหว่าง n ถึง m ครั้ง
console.log(/\d{2,4}/.test("1"));     // false
console.log(/\d{2,4}/.test("12"));    // true
console.log(/\d{2,4}/.test("1234"));  // true
console.log(/\d{2,4}/.test("12345")); // true (matches first 4)

// ตัวอย่าง password validation
function validatePassword(pwd) {
  if (pwd.length < 8) return "รหัสผ่านต้องมีอย่างน้อย 8 ตัวอักษร";
  if (!/[A-Z]/.test(pwd)) return "ต้องมีตัวพิมพ์ใหญ่อย่างน้อย 1 ตัว";
  if (!/[a-z]/.test(pwd)) return "ต้องมีตัวพิมพ์เล็กอย่างน้อย 1 ตัว";
  if (!/\d/.test(pwd)) return "ต้องมีตัวเลขอย่างน้อย 1 ตัว";
  if (!/[!@#$%^&*]/.test(pwd)) return "ต้องมีอักขระพิเศษอย่างน้อย 1 ตัว";
  return null; // valid
}

console.log(validatePassword("short"));       // "รหัสผ่านต้องมีอย่างน้อย..."
console.log(validatePassword("Password1!"));  // null (valid)
console.log(validatePassword("password1!"));  // "ต้องมีตัวพิมพ์ใหญ่..."
```

---

## Step 341: Greedy vs Lazy Matching

```javascript
// Quantifiers เป็น Greedy โดยปริยาย - match มากที่สุดเท่าที่เป็นไปได้

const html = "<b>bold</b> and <i>italic</i>";

// Greedy - match มากที่สุด
const greedy = html.match(/<.+>/);
console.log(greedy[0]); // "<b>bold</b> and <i>italic</i>" (whole thing!)

// Lazy (?) - match น้อยที่สุด
const lazy = html.match(/<.+?>/);
console.log(lazy[0]); // "<b>" (just first tag)

// เปรียบเทียบ
const text = "start MATCH1 end start MATCH2 end";

console.log(text.match(/start .+ end/)[0]);
// "start MATCH1 end start MATCH2 end" (greedy)

console.log(text.match(/start .+? end/)[0]);
// "start MATCH1 end" (lazy)

// ตัวอย่างในทางปฏิบัติ
const json = '{"key1":"value1","key2":"value2"}';

// Greedy - ผิด
const greedyMatch = json.match(/"[^"]*":"[^"]*"/g);
// ถูกต้องกว่าด้วย specific pattern:
const allPairs = json.match(/"([^"]+)":"([^"]+)"/g);
console.log(allPairs);
// ['"key1":"value1"', '"key2":"value2"']

// HTML stripping
function stripHtml(html) {
  return html.replace(/<[^>]+>/g, ''); // [^>]+ - non-greedy alternative
}
console.log(stripHtml("<p>Hello <b>World</b>!</p>")); // "Hello World!"

// Quote matching
const quotes = '"Hello" and "World"';
const greedyQuote = quotes.match(/".*"/)[0];    // '"Hello" and "World"'
const lazyQuote = quotes.match(/".*?"/)[0];     // '"Hello"'
const allQuotes = quotes.match(/"[^"]*"/g);     // ['"Hello"', '"World"']
console.log(allQuotes);
```

---

## Step 342: Groups และ Capturing

```javascript
// () - Capturing group - เก็บ match ไว้ใช้ทีหลัง

// Basic capturing
const dateStr = "2024-01-15";
const dateMatch = dateStr.match(/(\d{4})-(\d{2})-(\d{2})/);
console.log(dateMatch[0]); // "2024-01-15" (full match)
console.log(dateMatch[1]); // "2024" (group 1)
console.log(dateMatch[2]); // "01" (group 2)
console.log(dateMatch[3]); // "15" (group 3)

// Non-capturing group (?:)
const noCapture = "2024-01-15".match(/(?:\d{4})-(\d{2})-(\d{2})/);
console.log(noCapture[1]); // "01" (group 1, not year!)
// (?:) ไม่นับเป็น capture group

// Named capturing groups (?<name>)
const namedMatch = dateStr.match(/(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/);
console.log(namedMatch.groups.year);  // "2024"
console.log(namedMatch.groups.month); // "01"
console.log(namedMatch.groups.day);   // "15"

// Backreference - อ้างถึง group ที่ผ่านมา
// \1 อ้างถึง group 1
const repeated = /(\w+)\s+\1/;
console.log(repeated.test("the the"));    // true (same word twice)
console.log(repeated.test("hello hello")); // true
console.log(repeated.test("the a"));      // false

// Named backreference \k<name>
const htmlTag = /<(?<tag>\w+)>[^<]*<\/\k<tag>>/;
console.log(htmlTag.test("<b>bold</b>")); // true
console.log(htmlTag.test("<b>text</i>")); // false (mismatched tags)

// Groups ใน replace
const name = "John Smith";
console.log(name.replace(/(\w+)\s+(\w+)/, "$2, $1")); // "Smith, John"

// Named groups ใน replace
const date = "2024-01-15";
console.log(date.replace(
  /(?<y>\d{4})-(?<m>\d{2})-(?<d>\d{2})/,
  "$<d>/$<m>/$<y>"
)); // "15/01/2024"
```

---

## Step 343: Alternation

```javascript
// | - OR สำหรับ regex patterns

// Match หลายค่า
const colorRegex = /red|green|blue/i;
console.log(colorRegex.test("I like red color"));   // true
console.log(colorRegex.test("I like blue color"));  // true
console.log(colorRegex.test("I like pink color"));  // false

// Alternation กับ groups
const fruitRegex = /^(apple|banana|orange)$/i;
console.log(fruitRegex.test("apple"));   // true
console.log(fruitRegex.test("banana"));  // true
console.log(fruitRegex.test("mango"));   // false

// Complex alternation
const unitRegex = /\d+\s*(kg|g|lb|oz|L|ml)/i;
console.log(unitRegex.test("500 g"));    // true
console.log(unitRegex.test("2.5 kg"));   // true
console.log(unitRegex.test("3 L"));      // true
console.log(unitRegex.test("100 km"));   // false

// File extension validation
function isImageFile(filename) {
  return /\.(jpg|jpeg|png|gif|webp|svg|bmp)$/i.test(filename);
}
console.log(isImageFile("photo.jpg"));    // true
console.log(isImageFile("image.PNG"));    // true (case insensitive)
console.log(isImageFile("document.pdf")); // false

// URL protocol matching
const urlRegex = /^(https?|ftp):\/\/.+/;
console.log(urlRegex.test("https://example.com")); // true
console.log(urlRegex.test("ftp://files.example.com")); // true
console.log(urlRegex.test("http://test.org"));     // true
console.log(urlRegex.test("file:///local"));       // false

// Multiple word patterns
const greetings = "สวัสดี Hello Hola Bonjour";
const matches = greetings.match(/สวัสดี|Hello|Hola|Bonjour/g);
console.log(matches.length); // 4
```

---

## Step 344: Lookahead และ Lookbehind

```javascript
// Lookahead - ดูข้างหน้าโดยไม่ consume characters

// Positive Lookahead (?=...)
// Match ถ้ามี pattern ตามหลัง
const prices = "100 THB 200 USD 300 EUR";
const thbAmounts = prices.match(/\d+(?=\s*THB)/g);
console.log(thbAmounts); // ["100"]

// Negative Lookahead (?!...)
// Match ถ้าไม่มี pattern ตามหลัง
const nonThb = prices.match(/\d+(?!\s*THB)\b/g);
// นับ non-THB amounts (tricky - ต้องระวัง partial matches)

// Lookbehind (?<=...) - ดูข้างหลัง
// Positive Lookbehind (?<=...)
const str = "100USD 200EUR 300THB";
const afterUSD = str.match(/(?<=USD)\d+/g);
// Note: lookbehind requires numbers after

// ตัวอย่างที่ดีกว่า
const prices2 = "USD100 EUR200 THB300";
const usdAmount = prices2.match(/(?<=USD)\d+/g);
console.log(usdAmount); // ["100"]

// Negative Lookbehind (?<!...)
const text = "100px 200em 300px 400rem";
const notPx = text.match(/\d+(?!px)\b/g);
console.log(notPx); // ["200", "400"] (ไม่มี px ตาม)

// Password validation ด้วย lookahead
function checkPasswordStrength(password) {
  const checks = [
    { regex: /(?=.*[A-Z])/, message: "มีตัวพิมพ์ใหญ่" },
    { regex: /(?=.*[a-z])/, message: "มีตัวพิมพ์เล็ก" },
    { regex: /(?=.*\d)/, message: "มีตัวเลข" },
    { regex: /(?=.*[!@#$%^&*])/, message: "มีอักขระพิเศษ" },
    { regex: /.{8,}/, message: "ยาวอย่างน้อย 8 ตัว" }
  ];
  
  return checks.map(check => ({
    passed: check.regex.test(password),
    message: check.message
  }));
}

const result = checkPasswordStrength("MyPass1!");
result.forEach(({ passed, message }) => {
  console.log(`${passed ? '✓' : '✗'} ${message}`);
});
```

---

## Step 345: Flags

```javascript
// i - case insensitive
console.log(/hello/i.test("HELLO")); // true
console.log(/hello/i.test("Hello")); // true
console.log(/hello/i.test("hElLo")); // true

// g - global (หาทุก match)
const str = "cat bat sat";
console.log(str.match(/[bcs]at/));  // ["cat"] (first only)
console.log(str.match(/[bcs]at/g)); // ["cat", "bat", "sat"]

// m - multiline (^ and $ match start/end of each line)
const multiStr = `line1
line2
line3`;
console.log(multiStr.match(/^\w+/g));  // ["line1"] (without m)
console.log(multiStr.match(/^\w+/gm)); // ["line1", "line2", "line3"]

// s - dotAll (. matches \n too)
const text = "line1\nline2";
console.log(/line1.line2/.test(text));  // false (. ไม่ match \n)
console.log(/line1.line2/s.test(text)); // true (s flag)

// u - unicode
const emoji = "Hello 😀 World";
console.log(emoji.match(/./gu).length);  // ถูกต้องกว่า (ใช้ u)
console.log(/\u{1F600}/u.test(emoji));   // true (emoji unicode point)

// Thai with unicode flag
console.log(/[฀-๿]+/u.test("สวัสดี")); // true

// y - sticky (match from lastIndex position)
const stickyRegex = /\d+/y;
const numStr = "12 34 56";
stickyRegex.lastIndex = 0;
console.log(stickyRegex.exec(numStr)?.[0]); // "12"
stickyRegex.lastIndex = 3;
console.log(stickyRegex.exec(numStr)?.[0]); // "34"
stickyRegex.lastIndex = 2;
console.log(stickyRegex.exec(numStr)?.[0]); // null (no match at index 2)

// Combining flags
const text2 = "Hello\nHELLO\nhello";
console.log(text2.match(/^hello$/gim));
// ["Hello", "HELLO", "hello"]
```

---

## Step 346: Regex Patterns ทั่วไป

```javascript
// Email
const emailRegex = /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/;
function validateEmail(email) {
  return emailRegex.test(email);
}
console.log(validateEmail("test@example.com"));   // true
console.log(validateEmail("user.name+tag@gmail.com")); // true
console.log(validateEmail("invalid@"));           // false
console.log(validateEmail("@invalid.com"));       // false

// Thai Phone Numbers
function validateThaiPhone(phone) {
  const cleaned = phone.replace(/[\s\-()]/g, '');
  // 0xx-xxx-xxxx format (mobile)
  return /^0[6-9]\d{8}$/.test(cleaned);
}
console.log(validateThaiPhone("0812345678"));    // true
console.log(validateThaiPhone("081-234-5678")); // true
console.log(validateThaiPhone("02-123-4567"));  // false (landline - different)

// Thai Landline
function validateThaiLandline(phone) {
  const cleaned = phone.replace(/[\s\-()]/g, '');
  return /^0[2-9]\d{7}$/.test(cleaned) || /^0[2-9]\d{8}$/.test(cleaned);
}

// URL
const urlRegex = /^(https?|ftp):\/\/[^\s/$.?#].[^\s]*$/i;
function validateURL(url) {
  return urlRegex.test(url);
}
console.log(validateURL("https://www.example.com"));         // true
console.log(validateURL("http://example.com/path?q=1"));    // true
console.log(validateURL("ftp://files.server.com"));          // true
console.log(validateURL("not-a-url"));                       // false

// Password strength
const passwordRegex = /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[@$!%*?&])[A-Za-z\d@$!%*?&]{8,}$/;
function validateStrongPassword(pwd) {
  return passwordRegex.test(pwd);
}
console.log(validateStrongPassword("MyPass1!"));  // true
console.log(validateStrongPassword("weak"));      // false

// Thai ID Card
function validateThaiID(id) {
  const cleaned = id.replace(/[\s\-]/g, '');
  if (!/^\d{13}$/.test(cleaned)) return false;
  
  // Checksum validation
  let sum = 0;
  for (let i = 0; i < 12; i++) {
    sum += parseInt(cleaned[i]) * (13 - i);
  }
  const check = (11 - (sum % 11)) % 10;
  return check === parseInt(cleaned[12]);
}

// Date formats
const datePatterns = {
  iso: /^\d{4}-\d{2}-\d{2}$/,
  thai: /^\d{1,2}\/\d{1,2}\/\d{4}$/,
  full: /^\d{1,2}\s+\w+\s+\d{4}$/
};

console.log(datePatterns.iso.test("2024-01-15"));   // true
console.log(datePatterns.thai.test("15/01/2024")); // true

// Postal Code (Thailand)
function validatePostalCode(code) {
  return /^[1-9]\d{4}$/.test(code);
}
console.log(validatePostalCode("10110")); // true
console.log(validatePostalCode("00000")); // false

// Credit Card (basic)
function isCreditCard(num) {
  const cleaned = num.replace(/\s/g, '');
  return /^\d{13,19}$/.test(cleaned);
}

// IPv4
function isIPv4(ip) {
  return /^(\d{1,3}\.){3}\d{1,3}$/.test(ip) &&
    ip.split('.').every(n => parseInt(n) >= 0 && parseInt(n) <= 255);
}
console.log(isIPv4("192.168.1.1"));  // true
console.log(isIPv4("256.1.1.1"));   // false
```

---

## Step 347: Advanced Regex Techniques

```javascript
// Regex ที่ซับซ้อนกว่า

// 1. Extracting all URLs from text
function extractURLs(text) {
  const urlRegex = /https?:\/\/[^\s<>"{}|\\^`[\]]+/g;
  return text.match(urlRegex) || [];
}

const article = "Visit https://example.com or https://test.org/page?id=1 for more info";
console.log(extractURLs(article));
// ["https://example.com", "https://test.org/page?id=1"]

// 2. Parse query string
function parseQueryString(qs) {
  const params = {};
  const cleaned = qs.startsWith('?') ? qs.slice(1) : qs;
  const matches = cleaned.matchAll(/([^&=]+)=([^&]*)/g);
  
  for (const match of matches) {
    params[decodeURIComponent(match[1])] = decodeURIComponent(match[2]);
  }
  
  return params;
}

console.log(parseQueryString("?name=สมชาย&age=25&city=กรุงเทพ"));
// { name: "สมชาย", age: "25", city: "กรุงเทพ" }

// 3. Slug generator
function toSlug(text) {
  return text
    .toLowerCase()
    .replace(/\s+/g, '-')        // spaces to hyphens
    .replace(/[^\w\-]+/g, '')    // remove special chars
    .replace(/\-\-+/g, '-')      // collapse multiple hyphens
    .replace(/^-+|-+$/g, '');    // trim hyphens
}

console.log(toSlug("Hello World! This is a Test"));
// "hello-world-this-is-a-test"

// 4. Highlight search terms
function highlightTerms(text, terms) {
  const escapedTerms = terms.map(t => t.replace(/[.*+?^${}()|[\]\\]/g, '\\$&'));
  const pattern = new RegExp(`(${escapedTerms.join('|')})`, 'gi');
  return text.replace(pattern, '<mark>$1</mark>');
}

console.log(highlightTerms("JavaScript is a programming language", ["java", "language"]));
// "<mark>Java</mark>Script is a programming <mark>language</mark>"

// 5. Truncate text at word boundary
function truncateAtWord(text, maxLength) {
  if (text.length <= maxLength) return text;
  return text.slice(0, maxLength).replace(/\s+\S*$/, '') + '...';
}

console.log(truncateAtWord("This is a long sentence that needs truncating", 20));
// "This is a long..."

// 6. Extract hashtags
function extractHashtags(text) {
  return (text.match(/#[a-zA-Z฀-๿]\w*/g) || [])
    .map(tag => tag.slice(1));
}

const tweet = "ชอบ #javascript และ #react มาก #programming #webdev";
console.log(extractHashtags(tweet));
// ["javascript", "react", "programming", "webdev"]

// 7. Mask sensitive data
function maskEmail(email) {
  return email.replace(/^(.{2})[^@]+(@.+)$/, '$1***$2');
}

function maskPhone(phone) {
  return phone.replace(/(\d{3})\d{4}(\d{4})/, '$1****$2');
}

function maskCreditCard(card) {
  return card.replace(/\d(?=\d{4})/g, '*');
}

console.log(maskEmail("somchai@example.com"));    // "so***@example.com"
console.log(maskPhone("0812345678"));             // "081****5678"
console.log(maskCreditCard("1234567890123456"));  // "************3456"
```

---

## Step 348: Regex Performance

```javascript
// Regex performance tips

// 1. ใช้ specific patterns แทน generic
// ช้า:
const slowRegex = /.*foo.*/;
// เร็วกว่า:
const fastRegex = /foo/;

// 2. Anchor เมื่อรู้ตำแหน่ง
// ช้า:
const noAnchor = /^\d+/; // ถ้าตรวจสอบ from start - ดีแล้ว
// แต่:
const withAnchor = /^\d{5}$/; // เร็วกว่า /\d{5}/ เพราะหยุดทันที

// 3. หลีกเลี่ยง catastrophic backtracking
// อันตราย (ReDoS):
// /^(a+)+$/ - exponential backtracking
// ปลอดภัยกว่า:
// /^a+$/ - linear

// 4. Compile regex นอก loop
// ช้า:
function slowSearch(texts, pattern) {
  return texts.filter(t => t.match(new RegExp(pattern))); // สร้าง regex ทุก iteration
}

// เร็วกว่า:
function fastSearch(texts, pattern) {
  const regex = new RegExp(pattern); // สร้างครั้งเดียว
  return texts.filter(t => regex.test(t));
}

// 5. ใช้ indexOf สำหรับ simple string search
const text = "Hello World";
// เร็วกว่า:
const hasHello = text.includes("Hello");
// ช้ากว่า (ถ้าไม่ต้องการ regex features):
const hasHelloRegex = /Hello/.test(text);

// 6. Non-capturing groups เมื่อไม่ต้องการ capture
// ช้ากว่าเล็กน้อย:
/(https?):\/\//;
// เร็วกว่า (ถ้าไม่ใช้ capture):
/(?:https?):\/\//;

// Testing performance
function measureRegexTime(regex, text, iterations = 100000) {
  const start = performance.now();
  for (let i = 0; i < iterations; i++) {
    regex.test(text);
  }
  return performance.now() - start;
}
```

---

## Step 349: Regex Tester Utility

```javascript
// Regex utility ที่ใช้งานได้จริง

const RegexUtils = {
  // Test pattern
  test(pattern, flags, text) {
    try {
      const regex = new RegExp(pattern, flags);
      return {
        isValid: true,
        matches: regex.test(text),
        error: null
      };
    } catch (e) {
      return { isValid: false, matches: false, error: e.message };
    }
  },
  
  // Find all matches with details
  findAll(pattern, flags, text) {
    try {
      const regex = new RegExp(pattern, `${flags}g`);
      const matches = [];
      
      for (const match of text.matchAll(regex)) {
        matches.push({
          value: match[0],
          index: match.index,
          groups: match.groups || {},
          captures: match.slice(1)
        });
      }
      
      return { isValid: true, matches, error: null };
    } catch (e) {
      return { isValid: false, matches: [], error: e.message };
    }
  },
  
  // Replace with details
  replace(pattern, flags, text, replacement) {
    try {
      const regex = new RegExp(pattern, flags);
      return {
        isValid: true,
        result: text.replace(regex, replacement),
        error: null
      };
    } catch (e) {
      return { isValid: false, result: text, error: e.message };
    }
  },
  
  // Explain regex (simplified)
  explain(pattern) {
    const explanations = {
      '\\d': 'digit (0-9)',
      '\\D': 'non-digit',
      '\\w': 'word character (a-z, A-Z, 0-9, _)',
      '\\W': 'non-word character',
      '\\s': 'whitespace',
      '\\S': 'non-whitespace',
      '.': 'any character except newline',
      '^': 'start of string',
      '$': 'end of string',
      '*': 'zero or more',
      '+': 'one or more',
      '?': 'zero or one',
    };
    
    return Object.entries(explanations)
      .filter(([token]) => pattern.includes(token))
      .map(([token, desc]) => `${token} = ${desc}`);
  },
  
  // Common validators
  validators: {
    email: /^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$/,
    phone: /^0[6-9]\d{8}$/,
    url: /^https?:\/\/[^\s]+$/,
    ipv4: /^(\d{1,3}\.){3}\d{1,3}$/,
    hex: /^#[0-9A-Fa-f]{6}$/,
    date: /^\d{4}-\d{2}-\d{2}$/
  },
  
  validate(type, value) {
    const regex = this.validators[type];
    if (!regex) return { valid: false, error: `Unknown type: ${type}` };
    return { valid: regex.test(value), error: null };
  }
};

// ทดสอบ
console.log(RegexUtils.test('\\d+', '', 'abc123'));
// { isValid: true, matches: true, error: null }

console.log(RegexUtils.findAll('(\\d+)', '', 'a1b2c3'));
// matches with captures

console.log(RegexUtils.validate('email', 'test@example.com'));
// { valid: true, error: null }

console.log(RegexUtils.validate('email', 'invalid'));
// { valid: false, error: null }
```

---

## Step 350: โปรเจค - Form Validator ด้วย Regex

```javascript
class FormValidator {
  constructor() {
    this.rules = {};
  }
  
  addRule(fieldName, rules) {
    this.rules[fieldName] = rules;
    return this;
  }
  
  validate(data) {
    const errors = {};
    
    for (const [field, rules] of Object.entries(this.rules)) {
      const value = data[field];
      const fieldErrors = [];
      
      for (const rule of rules) {
        const error = this.checkRule(value, rule, data);
        if (error) fieldErrors.push(error);
      }
      
      if (fieldErrors.length > 0) {
        errors[field] = fieldErrors;
      }
    }
    
    return {
      isValid: Object.keys(errors).length === 0,
      errors
    };
  }
  
  checkRule(value, rule, allData) {
    switch (rule.type) {
      case 'required':
        if (!value || String(value).trim() === '') {
          return rule.message || 'Field is required';
        }
        break;
        
      case 'minLength':
        if (value && value.length < rule.min) {
          return rule.message || `Minimum length is ${rule.min}`;
        }
        break;
        
      case 'maxLength':
        if (value && value.length > rule.max) {
          return rule.message || `Maximum length is ${rule.max}`;
        }
        break;
        
      case 'pattern':
        if (value && !rule.regex.test(value)) {
          return rule.message || 'Invalid format';
        }
        break;
        
      case 'email':
        if (value && !/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value)) {
          return rule.message || 'Invalid email address';
        }
        break;
        
      case 'phone':
        if (value && !/^0[6-9]\d{8}$/.test(value.replace(/[\s\-]/g, ''))) {
          return rule.message || 'Invalid phone number';
        }
        break;
        
      case 'match':
        if (value !== allData[rule.field]) {
          return rule.message || `Must match ${rule.field}`;
        }
        break;
        
      case 'numeric':
        if (value && !/^\d+(\.\d+)?$/.test(String(value))) {
          return rule.message || 'Must be a number';
        }
        break;
    }
    return null;
  }
}

// ใช้งาน
const validator = new FormValidator();

validator
  .addRule('name', [
    { type: 'required', message: 'กรุณาใส่ชื่อ' },
    { type: 'minLength', min: 2, message: 'ชื่อต้องมีอย่างน้อย 2 ตัวอักษร' }
  ])
  .addRule('email', [
    { type: 'required', message: 'กรุณาใส่อีเมล' },
    { type: 'email', message: 'รูปแบบอีเมลไม่ถูกต้อง' }
  ])
  .addRule('phone', [
    { type: 'phone', message: 'เบอร์โทรศัพท์ไม่ถูกต้อง' }
  ])
  .addRule('password', [
    { type: 'required', message: 'กรุณาใส่รหัสผ่าน' },
    { type: 'minLength', min: 8, message: 'รหัสผ่านต้องมีอย่างน้อย 8 ตัวอักษร' },
    { type: 'pattern', regex: /[A-Z]/, message: 'ต้องมีตัวพิมพ์ใหญ่อย่างน้อย 1 ตัว' },
    { type: 'pattern', regex: /\d/, message: 'ต้องมีตัวเลขอย่างน้อย 1 ตัว' }
  ])
  .addRule('confirmPassword', [
    { type: 'required', message: 'กรุณายืนยันรหัสผ่าน' },
    { type: 'match', field: 'password', message: 'รหัสผ่านไม่ตรงกัน' }
  ]);

// ทดสอบ
const testData = {
  name: "ก",
  email: "invalid-email",
  phone: "0999999999",
  password: "weak",
  confirmPassword: "different"
};

const result = validator.validate(testData);
console.log('Valid:', result.isValid);
console.log('Errors:', JSON.stringify(result.errors, null, 2));

const validData = {
  name: "สมชาย",
  email: "somchai@example.com",
  phone: "0812345678",
  password: "MyPass1!",
  confirmPassword: "MyPass1!"
};

const validResult = validator.validate(validData);
console.log('Valid:', validResult.isValid); // true
```

---

## แบบฝึกหัด (Exercises)

### แบบฝึกหัดที่ 1: Password Validator
สร้าง `validatePassword(pwd)` ที่แสดงว่าผ่านกฎแต่ละข้อหรือไม่ (strength meter)

### แบบฝึกหัดที่ 2: CSV Parser
สร้าง `parseCSV(text)` ที่รองรับ quoted fields และ newlines ภายใน quotes

### แบบฝึกหัดที่ 3: Log Analyzer
Parse nginx access log format ดึง IP, method, path, status code, bytes

### แบบฝึกหัดที่ 4: Template Engine
สร้าง template engine ง่ายๆ ที่รองรับ `{{variable}}` และ `{{#if condition}}...{{/if}}`

### แบบฝึกหัดที่ 5: Code Syntax Highlighter
Highlight JavaScript keywords, strings, comments ด้วย regex และ HTML spans

---

## สรุป Part 18

ในบทนี้เรียนรู้:
- สร้าง Regex ด้วย literal syntax และ RegExp constructor
- Methods: test(), match(), matchAll(), search(), replace(), split()
- Character classes: [abc], [^abc], [a-z], \d, \w, \s
- Anchors: ^, $, \b, \B
- Quantifiers: *, +, ?, {n}, {n,}, {n,m}
- Greedy vs Lazy matching
- Groups: capturing, non-capturing, named
- Alternation, Lookahead, Lookbehind
- Flags: i, g, m, s, u, y
- Common patterns สำหรับ validation
- Form Validator project
