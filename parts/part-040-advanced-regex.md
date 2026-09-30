# Part 40: Regular Expressions ขั้นสูง (Steps 771-790)

## บทนำ

**Regular Expressions (Regex)** เป็นเครื่องมือที่ทรงพลังสำหรับการค้นหา ตรวจสอบ และแปลง text ใน ES2018+ JavaScript ได้เพิ่มความสามารถใหม่หลายอย่างที่ทำให้ regex ยิ่งทรงพลังขึ้น

---

## Step 771: ทบทวน Regex พื้นฐาน

```javascript
// สร้าง regex
const pattern1 = /hello/;          // literal syntax
const pattern2 = new RegExp("hello"); // constructor syntax

// Flags
const caseInsensitive = /hello/i;   // i = case insensitive
const global = /hello/g;            // g = global (find all)
const multiline = /^hello/m;        // m = multiline
const dotAll = /./s;                // s = dotAll (. matches newline)
const unicode = /\u{1F600}/u;       // u = unicode
const sticky = /hello/y;            // y = sticky
const hasIndices = /hello/d;        // d = hasIndices (ES2022)

// Methods
const str = "Hello World";
console.log(/hello/i.test(str));           // true
console.log(/hello/i.exec(str));           // ['Hello', index: 0, ...]
console.log(str.match(/[A-Z]/g));          // ['H', 'W']
console.log(str.replace(/world/i, "JS")); // "Hello JS"
console.log(str.search(/world/i));         // 6
console.log(str.split(/\s+/));            // ['Hello', 'World']
```

```javascript
// Character classes
const regex1 = /[abc]/;           // a, b, หรือ c
const regex2 = /[^abc]/;          // ไม่ใช่ a, b, c
const regex3 = /[a-z]/;           // lowercase a ถึง z
const regex4 = /[A-Z0-9]/;        // uppercase หรือตัวเลข
const regex5 = /\d/;              // digit [0-9]
const regex6 = /\w/;              // word char [a-zA-Z0-9_]
const regex7 = /\s/;              // whitespace
const regex8 = /\D/;              // non-digit
const regex9 = /\W/;              // non-word
const regex10 = /\S/;             // non-whitespace

// Quantifiers
const q1 = /a*/;    // 0 หรือมากกว่า
const q2 = /a+/;    // 1 หรือมากกว่า
const q3 = /a?/;    // 0 หรือ 1
const q4 = /a{3}/;  // exactly 3
const q5 = /a{2,4}/; // 2-4
const q6 = /a{2,}/;  // 2 หรือมากกว่า

// Greedy vs Lazy
const greedy = /a.+b/;   // greedy (ยาวที่สุด)
const lazy = /a.+?b/;    // lazy (สั้นที่สุด)
```

---

## Step 772: Lookahead - (?=...) Positive

```javascript
// Positive Lookahead (?=...): ตรวจสอบว่ามีตามหลัง แต่ไม่ consume

// ตัวอย่าง: หาราคาที่มี $ ตามหลัง
const str = "I have $100 and €200";
const matches = str.match(/\d+(?=\$|\€)/g);
// ❌ ผิด - lookahead อยู่หน้า, แต่ $ มีหลัง

// ถูก: หาเลขที่มี บาท ตามหลัง
const prices = "ราคา 100 บาท, 200 ดอลลาร์";
const bahtPrices = prices.match(/\d+(?= บาท)/g);
console.log(bahtPrices); // ['100']
```

```javascript
// ตัวอย่างใช้งานจริง

// 1. Password strength (ต้องมี uppercase, lowercase, digit)
function isStrongPassword(password) {
  const hasUpper = /(?=.*[A-Z])/.test(password);
  const hasLower = /(?=.*[a-z])/.test(password);
  const hasDigit = /(?=.*\d)/.test(password);
  const hasSpecial = /(?=.*[!@#$%^&*])/.test(password);
  const minLength = password.length >= 8;
  
  return {
    valid: hasUpper && hasLower && hasDigit && hasSpecial && minLength,
    strength: [hasUpper, hasLower, hasDigit, hasSpecial, minLength].filter(Boolean).length,
    details: { hasUpper, hasLower, hasDigit, hasSpecial, minLength }
  };
}

console.log(isStrongPassword("Abc123!@"));
// { valid: true, strength: 5, details: {...} }

console.log(isStrongPassword("password"));
// { valid: false, strength: 2, details: {...} }
```

```javascript
// 2. Number formatting (comma every 3 digits)
function formatNumber(num) {
  return String(num).replace(/\d(?=(\d{3})+(?!\d))/g, "$&,");
}

console.log(formatNumber(1234567));    // "1,234,567"
console.log(formatNumber(9876543210)); // "9,876,543,210"
console.log(formatNumber(123));        // "123"
```

```javascript
// 3. camelCase to snake_case
function camelToSnake(str) {
  return str.replace(/([A-Z])(?=[a-z]|$)/g, "_$1").toLowerCase();
}

console.log(camelToSnake("camelCaseString")); // "camel_case_string"
console.log(camelToSnake("XMLParser"));        // "x_m_l_parser"

// 4. PascalCase ตรวจจับ
function isPascalCase(str) {
  return /^[A-Z][a-z]+(?:[A-Z][a-z]+)*$/.test(str);
}

console.log(isPascalCase("PascalCase")); // true
console.log(isPascalCase("camelCase")); // false
```

---

## Step 773: Lookahead - (?!...) Negative

```javascript
// Negative Lookahead (?!...): ตรวจสอบว่าไม่มีตามหลัง

// หาคำที่ไม่ตามด้วย "ing"
const sentence = "I am running and jogging but not walking slowly";
const notIng = sentence.match(/\w+(?!ing\b)\b/g);
// (complex - ตัวอย่างนี้เป็นแค่ concept)

// ตัวอย่างใช้งานจริง

// 1. หาเลขที่ไม่ใช่ทศนิยม
const numbers = "10, 3.14, 42, 2.718, 100";
const integers = numbers.match(/\d+(?!\.\d)/g);
console.log(integers); // ['10', '42', '100'] (ไม่รวมทศนิยม)
```

```javascript
// 2. URL ที่ไม่ใช่ HTTPS
function findInsecureUrls(text) {
  return text.match(/https?:\/\/(?!www\.|secure\.)\S+/g);
}

const html = `
  <a href="https://example.com">safe</a>
  <a href="http://insecure.com">insecure</a>
  <a href="https://www.example.com">safe www</a>
`;

console.log(findInsecureUrls(html));
// ['http://insecure.com']
```

```javascript
// 3. หา if statement ที่ไม่มี else
const code = `
if (a) doSomething();
if (b) doThis();
else doThat();
if (c) doAnother();
`;

// หา if ที่ไม่มี else ตามหลัง (simplified)
const ifWithoutElse = code.match(/if\s*\([^)]+\)[^;]+;(?!\s*else)/g);
console.log(ifWithoutElse);
```

```javascript
// 4. Validate ว่าไม่มี reserved words
const RESERVED = ["class", "function", "return", "var", "let", "const"];
const reservedPattern = new RegExp(
  `(?!\\b(?:${RESERVED.join("|")})\\b)[a-zA-Z_]\\w*`,
  "g"
);

const identifiers = "let myVar = function() { return class; }";
const nonReserved = identifiers.match(reservedPattern);
console.log(nonReserved); // ['myVar']
```

---

## Step 774: Lookbehind - (?<=...) Positive

```javascript
// Positive Lookbehind (?<=...): ตรวจสอบว่ามีอยู่ก่อนหน้า (ES2018)

// หาตัวเลขที่มี $ นำหน้า
const prices = "Apple: $25, Banana: €10, Cherry: $50";
const dollarPrices = prices.match(/(?<=\$)\d+/g);
console.log(dollarPrices); // ['25', '50']

// หาตัวเลขที่มี € นำหน้า
const euroPrices = prices.match(/(?<=€)\d+/g);
console.log(euroPrices); // ['10']
```

```javascript
// ตัวอย่างใช้งานจริง

// 1. Extract domain จาก email
function extractDomain(email) {
  const match = email.match(/(?<=@)[^.]+\.[^.]+$/);
  return match ? match[0] : null;
}

console.log(extractDomain("user@gmail.com"));      // "gmail.com"
console.log(extractDomain("admin@company.co.th")); // "co.th" -- ต้องปรับ pattern
```

```javascript
// 2. หาค่า property ใน CSS
function extractCSSValues(css, property) {
  const pattern = new RegExp(`(?<=${property}:\\s*)[^;]+`, "g");
  return css.match(pattern)?.map((v) => v.trim()) || [];
}

const css = `
  .container {
    color: red;
    background-color: blue;
    font-size: 16px;
  }
  .text {
    color: green;
    font-size: 14px;
  }
`;

console.log(extractCSSValues(css, "color"));     // ['red', 'green']
console.log(extractCSSValues(css, "font-size")); // ['16px', '14px']
```

```javascript
// 3. Extract quoted strings
const text = `He said "Hello World" and then "Goodbye!"`;
const quoted = text.match(/(?<=")\w[\w\s]*(?=")/g);
console.log(quoted); // ['Hello World', 'Goodbye!']
```

---

## Step 775: Lookbehind - (?<!...) Negative

```javascript
// Negative Lookbehind (?<!...): ตรวจสอบว่าไม่มีอยู่ก่อนหน้า

// หาตัวเลขที่ไม่มี $ นำหน้า
const prices = "Discount 10%, Price $50, Quantity 3, Tax $5";
const nonDollarNumbers = prices.match(/(?<!\$)\b\d+/g);
console.log(nonDollarNumbers); // ['10', '3'] (ไม่รวม 50 และ 5)
```

```javascript
// ตัวอย่างใช้งานจริง

// 1. หา URL ที่ไม่ใช่ image
function findNonImageUrls(urls) {
  return urls.filter((url) => !url.match(/(?<=\.)(jpg|jpeg|png|gif|webp)$/i));
}

const urls = [
  "https://example.com/page",
  "https://cdn.example.com/image.jpg",
  "https://api.example.com/data.json",
  "https://example.com/photo.png",
];

console.log(findNonImageUrls(urls));
// ['https://example.com/page', 'https://api.example.com/data.json']
```

```javascript
// 2. ค้นหา variable ที่ไม่ถูก declare ด้วย let/const/var
const code = `
  let declared = 1;
  const alsoDeclarad = 2;
  undeclared = 3;
  var another = 4;
`;

// simplified pattern
const undeclared = code.match(/(?<!(?:let|const|var)\s)\b([a-zA-Z_]\w*)\s*=/g);
console.log(undeclared);
```

---

## Step 776: Named Capturing Groups - (?<name>...)

```javascript
// Named groups ทำให้เข้าถึง captured group ด้วยชื่อ

// ISO date parsing
const dateRegex = /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/;
const dateStr = "2024-03-15";

const match = dateStr.match(dateRegex);
const { year, month, day } = match.groups;
console.log(`Year: ${year}, Month: ${month}, Day: ${day}`);
// Year: 2024, Month: 03, Day: 15
```

```javascript
// ตัวอย่าง: Parse URL
const urlRegex = /(?<protocol>https?):\/\/(?<domain>[^/]+)(?<path>\/[^?#]*)?(?:\?(?<query>[^#]*))?(?:#(?<fragment>.*))?/;

function parseURL(url) {
  const match = url.match(urlRegex);
  if (!match) return null;
  
  const { protocol, domain, path, query, fragment } = match.groups;
  
  const queryParams = {};
  if (query) {
    query.split("&").forEach((pair) => {
      const [key, value] = pair.split("=");
      queryParams[decodeURIComponent(key)] = decodeURIComponent(value || "");
    });
  }
  
  return { protocol, domain, path: path || "/", query: queryParams, fragment: fragment || "" };
}

const url = "https://example.com/search?q=javascript&lang=th#results";
console.log(parseURL(url));
// {
//   protocol: 'https',
//   domain: 'example.com',
//   path: '/search',
//   query: { q: 'javascript', lang: 'th' },
//   fragment: 'results'
// }
```

```javascript
// Named groups ใน replace
const dateStr2 = "2024-03-15";
const formatted = dateStr2.replace(
  /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/,
  "$<day>/$<month>/$<year>"
);
console.log(formatted); // "15/03/2024"
```

```javascript
// ตัวอย่าง: Parse log entries
const logRegex = /\[(?<timestamp>\d{4}-\d{2}-\d{2} \d{2}:\d{2}:\d{2})\] (?<level>ERROR|WARN|INFO|DEBUG) (?<message>.+)/;

const logs = [
  "[2024-03-15 14:30:00] ERROR Database connection failed",
  "[2024-03-15 14:30:01] WARN Retry attempt 1",
  "[2024-03-15 14:30:02] INFO Connected successfully",
];

logs.forEach((log) => {
  const match = log.match(logRegex);
  if (match) {
    const { timestamp, level, message } = match.groups;
    console.log({ timestamp, level, message });
  }
});
```

---

## Step 777: Backreferences - \1 และ \k<name>

```javascript
// Backreferences: อ้างอิงกลับไป captured group

// \1 อ้างอิง group ที่ 1
const doubled = /(\w+)\s+\1/;
console.log(doubled.test("hello hello")); // true
console.log(doubled.test("hello world")); // false

// หาคำซ้ำในประโยค
const sentence = "the the quick brown fox fox jumps";
const duplicates = sentence.match(/\b(\w+)\s+\1\b/g);
console.log(duplicates); // ['the the', 'fox fox']
```

```javascript
// Named backreferences: \k<name>
const htmlTag = /<(?<tag>[a-z]+)>.*?<\/\k<tag>>/s;
const html1 = "<div>content</div>";
const html2 = "<div>content</span>"; // mismatched tags

console.log(htmlTag.test(html1)); // true
console.log(htmlTag.test(html2)); // false
```

```javascript
// ตัวอย่าง: ตรวจสอบ palindrome
function isPalindrome(str) {
  const clean = str.toLowerCase().replace(/[^a-z0-9]/g, "");
  const regex = new RegExp(`^(.)(.)(.)\\3\\2\\1$|^(.)(.)\\.\\5\\4$|^(.)\\6$|^(.)$|^$`);
  // วิธีง่ายกว่า:
  return clean === clean.split("").reverse().join("");
}

// Backreference กับ string quotation
const quotedString = /(['"`])(?:(?!\1).)*\1/;
console.log(quotedString.test('"hello world"'));  // true
console.log(quotedString.test("'single quoted'")); // true
console.log(quotedString.test('"mixed quotes\''));  // false
```

```javascript
// ตัวอย่าง: Match XML-like balanced tags
function extractTagContent(html, tagName) {
  const pattern = new RegExp(
    `<${tagName}(?:\\s[^>]*)?>([\\s\\S]*?)<\\/${tagName}>`,
    "g"
  );
  
  const results = [];
  let match;
  
  while ((match = pattern.exec(html)) !== null) {
    results.push(match[1].trim());
  }
  
  return results;
}

const html = `
  <div>First div</div>
  <p>A paragraph</p>
  <div>Second div</div>
  <span>A span</span>
`;

console.log(extractTagContent(html, "div")); // ['First div', 'Second div']
console.log(extractTagContent(html, "p"));   // ['A paragraph']
```

---

## Step 778: Unicode Property Escapes - \p{...}

```javascript
// Unicode property escapes (ต้องใช้ u flag)

// \p{Letter} หรือ \p{L} - ตัวอักษรทุกภาษา
const unicode = /\p{L}+/gu;
const text = "Hello สวัสดี مرحبا 你好";
console.log(text.match(unicode)); // ['Hello', 'สวัสดี', 'مرحبا', '你好']

// \p{Number} หรือ \p{N} - ตัวเลขทุกภาษา
const numbers = /\p{N}+/gu;
const mixedNums = "เลขไทย ๑๒๓ and Arabic 123";
console.log(mixedNums.match(numbers)); // ['๑๒๓', '123']

// \p{Uppercase_Letter}
const upperCase = /\p{Lu}/gu;
"Hello World".match(upperCase); // ['H', 'W']

// \p{Lowercase_Letter}
const lowerCase = /\p{Ll}/gu;
"Hello World".match(lowerCase); // ['e', 'l', 'l', 'o', 'o', 'r', 'l', 'd']

// \p{Script=Thai}
const thaiScript = /\p{Script=Thai}+/gu;
const mixed = "English ภาษาไทย more English";
console.log(mixed.match(thaiScript)); // ['ภาษาไทย']
```

```javascript
// ตัวอย่างการใช้งาน

// 1. ตรวจสอบว่าเป็นตัวอักษรหรือเลขเท่านั้น (ทุกภาษา)
function isAlphanumeric(str) {
  return /^[\p{L}\p{N}]+$/u.test(str);
}

console.log(isAlphanumeric("Hello"));      // true
console.log(isAlphanumeric("สวัสดี"));    // true
console.log(isAlphanumeric("Hello123"));   // true
console.log(isAlphanumeric("Hello!"));     // false (มี !)

// 2. ลบ punctuation ทุกภาษา
function removePunctuation(str) {
  return str.replace(/\p{P}/gu, "");
}

console.log(removePunctuation("Hello, World! How are you?")); // "Hello World How are you"
```

```javascript
// \p{Emoji}
const emoji = /\p{Emoji_Presentation}+/gu;
const emojiText = "Hello 😀 World 🌍 Test 🎉";
console.log(emojiText.match(emoji)); // ['😀', '🌍', '🎉']

// ลบ emoji จาก text
function removeEmoji(str) {
  return str.replace(/\p{Emoji_Presentation}/gu, "").trim();
}

console.log(removeEmoji("Hello 😀 World! 🌍")); // "Hello  World!"
```

---

## Step 779: Unicode Flag (u) และผลกระทบ

```javascript
// Unicode flag (u) เปลี่ยนพฤติกรรมหลายอย่าง

// 1. Surrogate pairs (emoji, rare CJK)
const emoji = "😀";
console.log(emoji.length);         // 2 (สองหน่วย UTF-16)
console.log(/^.$/.test(emoji));    // false (. ไม่ match surrogate pair)
console.log(/^.$/u.test(emoji));   // true (u flag ทำให้ . match emoji ได้)

// 2. Unicode escape sequences
console.log(/\u{1F600}/u.test("😀")); // true
// console.log(/\u{1F600}/.test("😀")); // ❌ error หรือ false ถ้าไม่มี u

// 3. Strict mode: quantifiers บน surrogate pairs
const str = "😀😂🎉";
console.log(str.match(/😀/g));  // ['😀'] (surrogate pairs)
console.log(str.match(/./gu));             // ['😀', '😂', '🎉'] (characters)
```

```javascript
// ตัวอย่าง: นับตัวอักษรจริง (ไม่ใช่ code units)
function countCharacters(str) {
  return [...str].length; // หรือ str.length กับ u flag

  // หรือ
  // return Array.from(str).length;
}

const text = "Hello 😀 World";
console.log(text.length);            // 13 (code units)
console.log(countCharacters(text)); // 12 (characters)
```

---

## Step 780: Sticky Flag (y)

```javascript
// Sticky flag (y): match เฉพาะจากตำแหน่ง lastIndex เท่านั้น

const stickyRegex = /\d+/y;
const str = "abc 123 def 456";

stickyRegex.lastIndex = 4; // ตั้งตำแหน่งเริ่ม
console.log(stickyRegex.exec(str)); // ['123'] (ตำแหน่ง 4)
console.log(stickyRegex.lastIndex); // 7

stickyRegex.lastIndex = 7; // ตำแหน่ง " def..."
console.log(stickyRegex.exec(str)); // null (ไม่มีเลขที่ตำแหน่ง 7)
```

```javascript
// ความแตกต่างระหว่าง g กับ y
const str2 = "foo bar baz";
const global = /\w+/g;
const sticky = /\w+/y;

// g flag: ค้นหา next match ที่ไหนก็ได้
global.lastIndex = 0;
console.log(global.exec(str2));  // ['foo'] lastIndex=3
console.log(global.exec(str2));  // ['bar'] lastIndex=7
console.log(global.exec(str2));  // ['baz'] lastIndex=11

// y flag: ต้อง match ที่ lastIndex เท่านั้น
sticky.lastIndex = 0;
console.log(sticky.exec(str2)); // ['foo'] lastIndex=3
sticky.lastIndex = 4;
console.log(sticky.exec(str2)); // ['bar'] lastIndex=7
sticky.lastIndex = 5;           // ตำแหน่งกลางคำ
console.log(sticky.exec(str2)); // null (ไม่มี match ที่ตำแหน่ง 5)
```

```javascript
// Use case: Tokenizer ที่มีประสิทธิภาพ
function* tokenize(input) {
  const patterns = [
    { type: "NUMBER", regex: /\d+(?:\.\d+)?/y },
    { type: "STRING", regex: /"[^"]*"/y },
    { type: "IDENTIFIER", regex: /[a-zA-Z_]\w*/y },
    { type: "OPERATOR", regex: /[+\-*\/=<>!&|]+/y },
    { type: "WHITESPACE", regex: /\s+/y },
    { type: "PUNCTUATION", regex: /[(){}[\],;]/y },
  ];
  
  let pos = 0;
  
  while (pos < input.length) {
    let matched = false;
    
    for (const { type, regex } of patterns) {
      regex.lastIndex = pos;
      const match = regex.exec(input);
      
      if (match) {
        pos = regex.lastIndex;
        matched = true;
        
        if (type !== "WHITESPACE") {
          yield { type, value: match[0], pos: match.index };
        }
        break;
      }
    }
    
    if (!matched) {
      throw new SyntaxError(`Unexpected character: '${input[pos]}' at position ${pos}`);
    }
  }
}

for (const token of tokenize('let x = 42 + 3.14')) {
  console.log(token);
}
```

---

## Step 781: dotAll Flag (s)

```javascript
// s flag: . matches ทุก character รวมถึง newline

// ไม่มี s flag
const noS = /start.+end/;
console.log(noS.test("start middle end"));      // true
console.log(noS.test("start\nmiddle\nend"));    // false (. ไม่ match \n)

// มี s flag
const withS = /start.+end/s;
console.log(withS.test("start middle end"));     // true
console.log(withS.test("start\nmiddle\nend"));   // true
```

```javascript
// ตัวอย่าง: Parse multi-line comments
function extractComments(code) {
  const comments = [];
  const pattern = /\/\*(.*?)\*\//gs;
  
  let match;
  while ((match = pattern.exec(code)) !== null) {
    comments.push(match[1].trim());
  }
  
  return comments;
}

const code = `
/* Single line comment */
function hello() {
  /* Multi-line
     comment
     here */
  return "world";
}
/* Another comment */
`;

console.log(extractComments(code));
// ['Single line comment', 'Multi-line\n     comment\n     here', 'Another comment']
```

```javascript
// ตัวอย่าง: Extract HTML block content
function extractBlockContent(html, tagName) {
  const pattern = new RegExp(`<${tagName}[^>]*>([\\s\\S]*?)<\\/${tagName}>`, "gs");
  const results = [];
  
  let match;
  while ((match = pattern.exec(html)) !== null) {
    results.push(match[1].trim());
  }
  
  return results;
}

const html = `
<div>
  First
  block
</div>
<p>Paragraph</p>
<div>
  Second
  block
</div>
`;

console.log(extractBlockContent(html, "div"));
// ['First\n  block', 'Second\n  block']
```

---

## Step 782: hasIndices Flag (d) - ES2022

```javascript
// d flag: เพิ่ม indices property ให้ match result
const regex = /(\d+)/d;
const match = "test 123 end".match(regex);

console.log(match[0]);        // "123"
console.log(match.indices[0]); // [5, 8] (start, end)
console.log(match.indices[1]); // [5, 8] (capture group 1)
```

```javascript
// Named groups กับ indices
const namedRegex = /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/d;
const dateMatch = "Date: 2024-03-15".match(namedRegex);

console.log(dateMatch.groups);
// { year: '2024', month: '03', day: '15' }

console.log(dateMatch.indices.groups);
// { year: [6, 10], month: [11, 13], day: [14, 16] }
```

```javascript
// ตัวอย่าง: ใช้ indices สำหรับ syntax highlighting
function highlightPattern(text, pattern) {
  const regex = new RegExp(pattern, "dg");
  const highlights = [];
  
  let match;
  while ((match = regex.exec(text)) !== null) {
    highlights.push({
      text: match[0],
      start: match.indices[0][0],
      end: match.indices[0][1],
    });
  }
  
  return highlights;
}

const code = "const x = 42; let y = 100;";
const numberHighlights = highlightPattern(code, "\\d+");

console.log(numberHighlights);
// [{ text: '42', start: 10, end: 12 }, { text: '100', start: 22, end: 25 }]
```

---

## Step 783: String.prototype.matchAll()

```javascript
// matchAll() คืน iterator ของ ALL matches (ต้องใช้ g flag)

const text = "cat bat sat fat";
const regex = /[bcfs]at/g;

// เก่า: ต้องใช้ loop กับ exec
const oldMatches = [];
let m;
while ((m = regex.exec(text)) !== null) {
  oldMatches.push(m);
}

// ใหม่: matchAll
const matches = [...text.matchAll(/[bcfs]at/g)];
console.log(matches.length); // 4
console.log(matches[0][0]);  // "cat"
console.log(matches[1][0]);  // "bat"
```

```javascript
// matchAll กับ capturing groups
const html = '<a href="url1">Link 1</a> <a href="url2">Link 2</a>';
const linkRegex = /<a href="([^"]+)">([^<]+)<\/a>/g;

for (const match of html.matchAll(linkRegex)) {
  const [fullMatch, url, text] = match;
  console.log(`URL: ${url}, Text: ${text}`);
}
// URL: url1, Text: Link 1
// URL: url2, Text: Link 2
```

```javascript
// matchAll กับ named groups
const data = "2024-01-15, 2024-02-20, 2024-03-25";
const dateRegex = /(?<year>\d{4})-(?<month>\d{2})-(?<day>\d{2})/g;

const dates = [...data.matchAll(dateRegex)].map((m) => ({
  year: parseInt(m.groups.year),
  month: parseInt(m.groups.month),
  day: parseInt(m.groups.day),
}));

console.log(dates);
// [
//   { year: 2024, month: 1, day: 15 },
//   { year: 2024, month: 2, day: 20 },
//   { year: 2024, month: 3, day: 25 }
// ]
```

---

## Step 784: Replace with Function

```javascript
// replace() กับ function callback

// ตัวอย่าง 1: Title Case
function toTitleCase(str) {
  return str.replace(/\w\S*/g, (word) =>
    word.charAt(0).toUpperCase() + word.slice(1).toLowerCase()
  );
}

console.log(toTitleCase("hello world javascript")); // "Hello World Javascript"
```

```javascript
// ตัวอย่าง 2: Interpolate template
function interpolate(template, data) {
  return template.replace(/\{\{(\w+)\}\}/g, (match, key) => {
    return key in data ? data[key] : match;
  });
}

const template = "Hello, {{name}}! You have {{count}} messages.";
const data = { name: "สมชาย", count: 5 };

console.log(interpolate(template, data));
// "Hello, สมชาย! You have 5 messages."
```

```javascript
// ตัวอย่าง 3: Math expressions
function evaluateMath(str) {
  return str.replace(/(\d+)\s*([+\-*/])\s*(\d+)/g, (_, a, op, b) => {
    switch (op) {
      case "+": return Number(a) + Number(b);
      case "-": return Number(a) - Number(b);
      case "*": return Number(a) * Number(b);
      case "/": return Number(a) / Number(b);
    }
  });
}

console.log(evaluateMath("3 + 4 equals 3 + 4")); // "7 equals 7"
console.log(evaluateMath("10 * 5"));              // "50"
```

```javascript
// ตัวอย่าง 4: Caesar cipher
function caesarCipher(text, shift) {
  return text.replace(/[a-zA-Z]/g, (char) => {
    const base = char <= "Z" ? 65 : 97; // A=65, a=97
    return String.fromCharCode(((char.charCodeAt(0) - base + shift) % 26) + base);
  });
}

console.log(caesarCipher("Hello World", 3));  // "Khoor Zruog"
console.log(caesarCipher("Khoor Zruog", -3)); // "Hello World"
```

```javascript
// ตัวอย่าง 5: Markdown bold/italic
function parseMarkdown(text) {
  return text
    .replace(/\*\*(.+?)\*\*/g, "<strong>$1</strong>")
    .replace(/\*(.+?)\*/g, "<em>$1</em>")
    .replace(/`(.+?)`/g, "<code>$1</code>")
    .replace(/\[([^\]]+)\]\(([^)]+)\)/g, '<a href="$2">$1</a>');
}

const md = "Hello **World** and *JavaScript*. See `console.log()` at [MDN](https://mdn.io)";
console.log(parseMarkdown(md));
// "Hello <strong>World</strong> and <em>JavaScript</em>. See <code>console.log()</code> at <a href="https://mdn.io">MDN</a>"
```

---

## Step 785: Complex Real-world Pattern - URL Parsing

```javascript
// Full URL parser
const URL_REGEX = /^(?:(?<protocol>[a-z][a-z0-9+\-.]*):\/\/)?(?:(?<userinfo>[^@\s]+)@)?(?<host>(?:[a-zA-Z0-9](?:[a-zA-Z0-9-]*[a-zA-Z0-9])?\.)+[a-zA-Z]{2,}|localhost|\d{1,3}(?:\.\d{1,3}){3})(?::(?<port>\d+))?(?<path>\/[^\s?#]*)?(?:\?(?<query>[^\s#]*))?(?:#(?<fragment>[^\s]*))?$/i;

function parseURL(url) {
  const match = url.match(URL_REGEX);
  if (!match) return null;
  
  const { protocol, userinfo, host, port, path, query, fragment } = match.groups;
  
  const queryParams = {};
  if (query) {
    new URLSearchParams(query).forEach((value, key) => {
      queryParams[key] = value;
    });
  }
  
  return {
    protocol: protocol || "https",
    userinfo: userinfo || null,
    host,
    port: port ? parseInt(port) : null,
    path: path || "/",
    query: queryParams,
    fragment: fragment || null,
    toString() {
      let str = `${this.protocol}://`;
      if (this.userinfo) str += `${this.userinfo}@`;
      str += this.host;
      if (this.port) str += `:${this.port}`;
      str += this.path;
      const qs = new URLSearchParams(this.query).toString();
      if (qs) str += `?${qs}`;
      if (this.fragment) str += `#${this.fragment}`;
      return str;
    }
  };
}

const parsed = parseURL("https://user:pass@example.com:8080/path/to/page?key=value&other=123#section");
console.log(parsed.host);       // "example.com"
console.log(parsed.port);       // 8080
console.log(parsed.query);      // { key: 'value', other: '123' }
console.log(parsed.fragment);   // "section"
```

---

## Step 786: Complex Pattern - Markdown to HTML

```javascript
// Partial Markdown parser
class MarkdownParser {
  #rules = [
    // Headers
    { regex: /^#{6}\s+(.+)$/gm, replacement: "<h6>$1</h6>" },
    { regex: /^#{5}\s+(.+)$/gm, replacement: "<h5>$1</h5>" },
    { regex: /^#{4}\s+(.+)$/gm, replacement: "<h4>$1</h4>" },
    { regex: /^###\s+(.+)$/gm, replacement: "<h3>$1</h3>" },
    { regex: /^##\s+(.+)$/gm, replacement: "<h2>$1</h2>" },
    { regex: /^#\s+(.+)$/gm, replacement: "<h1>$1</h1>" },
    
    // Code blocks (ต้องทำก่อน inline)
    { regex: /```(\w+)?\n([\s\S]*?)```/g, replacement: (_, lang, code) =>
      `<pre><code class="${lang || ""}">${escapeHtml(code.trim())}</code></pre>` },
    
    // Inline code
    { regex: /`([^`]+)`/g, replacement: "<code>$1</code>" },
    
    // Bold and italic
    { regex: /\*\*\*(.+?)\*\*\*/g, replacement: "<strong><em>$1</em></strong>" },
    { regex: /\*\*(.+?)\*\*/g, replacement: "<strong>$1</strong>" },
    { regex: /\*(.+?)\*/g, replacement: "<em>$1</em>" },
    
    // Links and images
    { regex: /!\[([^\]]*)\]\(([^)]+)\)/g, replacement: '<img alt="$1" src="$2">' },
    { regex: /\[([^\]]+)\]\(([^)]+)\)/g, replacement: '<a href="$2">$1</a>' },
    
    // Blockquotes
    { regex: /^>\s+(.+)$/gm, replacement: "<blockquote>$1</blockquote>" },
    
    // Horizontal rule
    { regex: /^[-*_]{3,}$/gm, replacement: "<hr>" },
    
    // Unordered lists
    { regex: /^[-*+]\s+(.+)$/gm, replacement: "<li>$1</li>" },
    
    // Ordered lists
    { regex: /^\d+\.\s+(.+)$/gm, replacement: "<li>$1</li>" },
    
    // Paragraphs (double newline)
    { regex: /\n\n(?!<[h1-6]|<ul|<ol|<li|<pre|<blockquote|<hr)/g,
      replacement: "</p>\n<p>" },
  ];
  
  parse(markdown) {
    let html = markdown.trim();
    
    for (const { regex, replacement } of this.#rules) {
      html = html.replace(regex, replacement);
    }
    
    // Wrap in paragraph if not already wrapped
    if (!html.startsWith("<")) {
      html = `<p>${html}</p>`;
    }
    
    return html;
  }
}

function escapeHtml(str) {
  return str
    .replace(/&/g, "&amp;")
    .replace(/</g, "&lt;")
    .replace(/>/g, "&gt;")
    .replace(/"/g, "&quot;");
}

const parser = new MarkdownParser();
const md = `# Hello World

This is **bold** and *italic* text.

## Code Example

\`\`\`javascript
const x = 42;
console.log(x);
\`\`\`

Visit [MDN](https://developer.mozilla.org) for more.`;

console.log(parser.parse(md));
```

---

## Step 787: Complex Pattern - CSV Parsing

```javascript
// CSV Parser ที่รองรับ quoted fields และ escaped commas

function parseCSV(csv, options = {}) {
  const {
    delimiter = ",",
    quote = '"',
    hasHeader = true,
    skipEmpty = true,
  } = options;
  
  // Regex สำหรับ CSV field (รองรับ quoted strings)
  const fieldRegex = new RegExp(
    `(?:^|${delimiter})(?:"((?:[^"]*(?:""[^"]*)*)*)"|([^"${delimiter}\r\n]*))`,
    "g"
  );
  
  function parseLine(line) {
    const fields = [];
    let match;
    
    while ((match = fieldRegex.exec(line)) !== null) {
      if (match[1] !== undefined) {
        // Quoted field: unescape ""
        fields.push(match[1].replace(/""/g, '"'));
      } else {
        fields.push(match[2]);
      }
    }
    
    fieldRegex.lastIndex = 0;
    return fields;
  }
  
  const lines = csv.split(/\r?\n/).filter((line) => !skipEmpty || line.trim());
  
  if (lines.length === 0) return [];
  
  if (hasHeader) {
    const headers = parseLine(lines[0]);
    return lines.slice(1).map((line) => {
      const values = parseLine(line);
      return Object.fromEntries(headers.map((h, i) => [h, values[i] ?? ""]));
    });
  }
  
  return lines.map(parseLine);
}

// ทดสอบ
const csv = `name,age,city,quote
สมชาย,30,กรุงเทพ,"Hello, World"
สมศรี,25,เชียงใหม่,"She said ""Hi"""
วิภา,28,ภูเก็ต,No quotes`;

const records = parseCSV(csv);
console.log(records);
// [
//   { name: 'สมชาย', age: '30', city: 'กรุงเทพ', quote: 'Hello, World' },
//   { name: 'สมศรี', age: '25', city: 'เชียงใหม่', quote: 'She said "Hi"' },
//   { name: 'วิภา', age: '28', city: 'ภูเก็ต', quote: 'No quotes' },
// ]
```

---

## Step 788: Complex Pattern - Template Engine

```javascript
// Simple Template Engine

class TemplateEngine {
  #helpers = new Map();
  
  registerHelper(name, fn) {
    this.#helpers.set(name, fn);
    return this;
  }
  
  compile(template) {
    return (data) => this.render(template, data);
  }
  
  render(template, data) {
    let result = template;
    
    // 1. {{#each items}}...{{/each}}
    result = result.replace(
      /\{\{#each\s+(\w+)\}\}([\s\S]*?)\{\{\/each\}\}/g,
      (_, key, body) => {
        const items = this.#get(data, key);
        if (!Array.isArray(items)) return "";
        
        return items.map((item, index) =>
          this.render(body, { ...data, this: item, "@index": index, "@first": index === 0, "@last": index === items.length - 1 })
        ).join("");
      }
    );
    
    // 2. {{#if condition}}...{{else}}...{{/if}}
    result = result.replace(
      /\{\{#if\s+(.+?)\}\}([\s\S]*?)(?:\{\{else\}\}([\s\S]*?))?\{\{\/if\}\}/g,
      (_, condition, truePart, falsePart = "") => {
        const value = this.#evaluate(condition, data);
        return this.render(value ? truePart : falsePart, data);
      }
    );
    
    // 3. {{#unless condition}}...{{/unless}}
    result = result.replace(
      /\{\{#unless\s+(.+?)\}\}([\s\S]*?)\{\{\/unless\}\}/g,
      (_, condition, body) => {
        const value = this.#evaluate(condition, data);
        return this.render(value ? "" : body, data);
      }
    );
    
    // 4. {{helper arg1 arg2}}
    result = result.replace(
      /\{\{(\w+)\s+(.+?)\}\}/g,
      (_, helperName, argsStr) => {
        const helper = this.#helpers.get(helperName);
        if (!helper) return `{{${helperName} ${argsStr}}}`;
        
        const args = argsStr.split(/\s+/).map((arg) => {
          if (arg.startsWith('"') || arg.startsWith("'")) {
            return arg.slice(1, -1);
          }
          return this.#get(data, arg);
        });
        
        return helper(...args);
      }
    );
    
    // 5. {{variable}} and {{variable.nested}}
    result = result.replace(/\{\{([\w.@]+)\}\}/g, (_, key) => {
      const value = this.#get(data, key);
      return value !== undefined && value !== null ? escapeHtml(String(value)) : "";
    });
    
    return result;
  }
  
  #get(data, path) {
    return path.split(".").reduce((obj, key) => obj?.[key], data);
  }
  
  #evaluate(condition, data) {
    const value = this.#get(data, condition.trim());
    return Boolean(value);
  }
}

function escapeHtml(str) {
  return str.replace(/[&<>"']/g, (char) => ({
    "&": "&amp;", "<": "&lt;", ">": "&gt;", '"': "&quot;", "'": "&#39;"
  }[char]));
}

// ตัวอย่างการใช้งาน
const engine = new TemplateEngine();

engine.registerHelper("uppercase", (str) => str.toUpperCase());
engine.registerHelper("format", (num) => Number(num).toLocaleString());

const template = `
<div class="user-card">
  <h2>{{user.name}}</h2>
  <p>Age: {{user.age}}</p>
  {{#if user.premium}}
    <span class="badge">Premium</span>
  {{else}}
    <span class="badge free">Free</span>
  {{/if}}
  
  <ul>
    {{#each items}}
      <li class="{{#if @first}}first{{/if}} {{#if @last}}last{{/if}}">
        {{@index}}. {{this.name}} - {{this.price}} บาท
      </li>
    {{/each}}
  </ul>
</div>`;

const data = {
  user: { name: "สมชาย", age: 30, premium: true },
  items: [
    { name: "Apple", price: 20 },
    { name: "Banana", price: 15 },
    { name: "Cherry", price: 50 },
  ]
};

console.log(engine.render(template, data));
```

---

## Step 789: Complex Pattern - Code Syntax Highlighting

```javascript
// Simple JavaScript syntax highlighter

const JS_PATTERNS = [
  {
    name: "comment-block",
    pattern: /\/\*[\s\S]*?\*\//g,
    class: "comment"
  },
  {
    name: "comment-line",
    pattern: /\/\/[^\n]*/g,
    class: "comment"
  },
  {
    name: "string-template",
    pattern: /`(?:[^`\\]|\\.)*`/g,
    class: "string"
  },
  {
    name: "string-double",
    pattern: /"(?:[^"\\]|\\.)*"/g,
    class: "string"
  },
  {
    name: "string-single",
    pattern: /'(?:[^'\\]|\\.)*'/g,
    class: "string"
  },
  {
    name: "keyword",
    pattern: /\b(?:async|await|break|case|catch|class|const|continue|debugger|default|delete|do|else|export|extends|false|finally|for|from|function|if|import|in|instanceof|let|new|null|of|return|static|super|switch|this|throw|true|try|typeof|undefined|var|void|while|with|yield)\b/g,
    class: "keyword"
  },
  {
    name: "number",
    pattern: /\b(?:0x[\da-f]+|0o[0-7]+|0b[01]+|\d+(?:\.\d+)?(?:e[+-]?\d+)?n?)\b/gi,
    class: "number"
  },
  {
    name: "function",
    pattern: /\b(\w+)(?=\s*\()/g,
    class: "function"
  },
  {
    name: "property",
    pattern: /(?<=\.)\w+/g,
    class: "property"
  },
];

function highlightJS(code) {
  // แทนที่ด้วย placeholder เพื่อป้องกัน double-replace
  const placeholders = new Map();
  let counter = 0;
  let result = code;
  
  for (const { name, pattern, class: cls } of JS_PATTERNS) {
    result = result.replace(pattern, (match) => {
      const id = `__PLACEHOLDER_${counter++}__`;
      placeholders.set(id, `<span class="${cls}">${escapeHtml(match)}</span>`);
      return id;
    });
  }
  
  // Restore placeholders
  for (const [id, html] of placeholders) {
    result = result.replace(id, html);
  }
  
  return `<pre><code class="language-javascript">${result}</code></pre>`;
}

function escapeHtml(str) {
  return str.replace(/[&<>"']/g, (c) => ({ "&": "&amp;", "<": "&lt;", ">": "&gt;", '"': "&quot;", "'": "&#39;" }[c]));
}

const code = `
// Fibonacci sequence
function fibonacci(n) {
  if (n <= 1) return n;
  const result = fibonacci(n - 1) + fibonacci(n - 2);
  return result;
}

console.log(fibonacci(10)); // 55
`;

console.log(highlightJS(code));
```

---

## Step 790: ตัวอย่างสมบูรณ์ - Email Validation System

```javascript
// Email validation ที่ครอบคลุม

const EMAIL_RULES = {
  // RFC 5321 compliant (simplified)
  pattern: /^(?:(?:[\w!#$%&'*+\-/=?^`{|}~]+(?:\.[\w!#$%&'*+\-/=?^`{|}~]+)*)|(?:"(?:[^"\\]|\\.)*"))@(?:(?:[a-zA-Z0-9](?:[a-zA-Z0-9-]*[a-zA-Z0-9])?\.)+[a-zA-Z]{2,}|(?:\[(?:(?:25[0-5]|2[0-4]\d|[01]?\d\d?)\.){3}(?:25[0-5]|2[0-4]\d|[01]?\d\d?)\]))$/,
  
  maxLength: 254,
  localMaxLength: 64,
  
  blockedDomains: new Set([
    "tempmail.com",
    "throwaway.email",
    "mailinator.com"
  ]),
  
  validate(email) {
    const errors = [];
    
    if (!email || typeof email !== "string") {
      return { valid: false, errors: ["Email is required"] };
    }
    
    const normalized = email.trim().toLowerCase();
    
    // Length checks
    if (normalized.length > this.maxLength) {
      errors.push(`Email too long (max ${this.maxLength} characters)`);
    }
    
    // Format check
    if (!this.pattern.test(normalized)) {
      errors.push("Invalid email format");
      return { valid: false, errors };
    }
    
    const [local, domain] = normalized.split("@");
    
    // Local part length
    if (local.length > this.localMaxLength) {
      errors.push(`Local part too long (max ${this.localMaxLength} characters)`);
    }
    
    // Blocked domains
    if (this.blockedDomains.has(domain)) {
      errors.push(`Domain "${domain}" is not allowed`);
    }
    
    // Double dots
    if (/\.\./.test(normalized)) {
      errors.push("Email cannot contain consecutive dots");
    }
    
    // Starting/ending with dot
    if (local.startsWith(".") || local.endsWith(".")) {
      errors.push("Local part cannot start or end with a dot");
    }
    
    return {
      valid: errors.length === 0,
      normalized,
      local,
      domain,
      errors
    };
  }
};

// ทดสอบ
const testEmails = [
  "valid@example.com",
  "user.name@domain.co.th",
  "invalid-email",
  "missing@",
  "@nodomain.com",
  "spaces in@email.com",
  "test@tempmail.com",
  "a".repeat(65) + "@example.com",
  "valid+tag@gmail.com",
  "test..double@dots.com",
];

testEmails.forEach((email) => {
  const result = EMAIL_RULES.validate(email);
  console.log(`${email}: ${result.valid ? "✓ Valid" : "✗ Invalid"}`);
  if (!result.valid) {
    result.errors.forEach((e) => console.log(`  - ${e}`));
  }
});
```

---

## แบบฝึกหัด

### Easy
1. เขียน regex ตรวจสอบ phone number ไทย (0XX-XXXXXXX หรือ 0XXXXXXXXX)
2. หาคำที่ขึ้นต้นด้วย capital letter ในประโยค
3. เขียน regex แทน profanity ด้วย ***

### Medium
4. สร้าง URL shortener validator ที่ใช้ lookahead/lookbehind
5. Parse markdown table เป็น 2D array
6. สร้าง pattern ตรวจสอบ Thai national ID card (13 digits + checksum)

### Hard
7. สร้าง mini regex engine ที่รองรับ `.`, `*`, `+`, `?`, `[]`
8. Implement full CSV parser ที่รองรับ all edge cases
9. สร้าง Syntax Highlighter ที่รองรับ multiple languages

### Solution ตัวอย่าง

```javascript
// 1. Thai phone number
function validateThaiPhone(phone) {
  const patterns = [
    /^0[689]\d{8}$/,          // mobile: 06X, 08X, 09X
    /^0[2-9]\d{7}$/,           // landline: 02-09
    /^0[689]\d{1}-\d{3}-\d{4}$/, // formatted mobile
    /^0[2-9]-\d{3}-\d{4}$/,   // formatted landline
  ];
  
  const cleaned = phone.replace(/[-\s]/g, "");
  return patterns.some((p) => p.test(cleaned));
}

console.log(validateThaiPhone("0812345678"));  // true
console.log(validateThaiPhone("081-234-5678")); // true
console.log(validateThaiPhone("021234567"));    // true
console.log(validateThaiPhone("1234567890"));   // false

// 6. Thai National ID validation
function validateThaiID(id) {
  // ต้องเป็นเลข 13 หลัก
  if (!/^\d{13}$/.test(id)) return false;
  
  // Checksum algorithm
  const digits = id.split("").map(Number);
  const weights = [13, 12, 11, 10, 9, 8, 7, 6, 5, 4, 3, 2];
  
  const sum = weights.reduce((acc, w, i) => acc + w * digits[i], 0);
  const checkDigit = (11 - (sum % 11)) % 10;
  
  return checkDigit === digits[12];
}

// ตัวอย่าง ID (สมมติ)
console.log(validateThaiID("1234567890121")); // ขึ้นอยู่กับ checksum

// 5. Parse Markdown table
function parseMarkdownTable(table) {
  const lines = table.trim().split("\n");
  
  if (lines.length < 3) return null;
  
  const parseRow = (line) =>
    line.split("|").slice(1, -1).map((cell) => cell.trim());
  
  const headers = parseRow(lines[0]);
  // lines[1] คือ separator row (|---|---|...)
  
  const rows = lines.slice(2).map((line) => {
    const cells = parseRow(line);
    return Object.fromEntries(headers.map((h, i) => [h, cells[i] ?? ""]));
  });
  
  return { headers, rows };
}

const mdTable = `
| Name | Age | City |
|------|-----|------|
| สมชาย | 30 | กรุงเทพ |
| สมศรี | 25 | เชียงใหม่ |
`.trim();

console.log(parseMarkdownTable(mdTable));
// {
//   headers: ['Name', 'Age', 'City'],
//   rows: [
//     { Name: 'สมชาย', Age: '30', City: 'กรุงเทพ' },
//     { Name: 'สมศรี', Age: '25', City: 'เชียงใหม่' }
//   ]
// }
```

---

## สรุป - Regex Features ES2015+

| Feature | Syntax | เวอร์ชัน |
|---------|--------|---------|
| Named groups | `(?<name>...)` | ES2018 |
| Named backreference | `\k<name>` | ES2018 |
| Positive lookbehind | `(?<=...)` | ES2018 |
| Negative lookbehind | `(?<!...)` | ES2018 |
| Unicode property escapes | `\p{...}` | ES2018 |
| dotAll flag | `s` | ES2018 |
| `String.matchAll()` | - | ES2020 |
| hasIndices flag | `d` | ES2022 |

**Best Practices:**
1. ใช้ named groups แทน numbered groups สำหรับ pattern ที่ซับซ้อน
2. ใช้ `u` flag เสมอเมื่อ deal กับ Unicode
3. ใช้ `s` flag เมื่อต้องการ match multi-line text กับ `.`
4. ใช้ `matchAll()` แทน `exec()` loop
5. ทดสอบ regex บน regex101.com ก่อน deploy
6. หลีกเลี่ยง catastrophic backtracking ด้วยการ limit backtracking
