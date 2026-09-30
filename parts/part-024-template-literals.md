# Part 24: Template Literals ขั้นสูง (Steps 451-470)

## บทนำ

Template Literals (หรือ Template Strings) เป็นฟีเจอร์ที่ถูกเพิ่มมาใน ES6 โดยใช้เครื่องหมาย backtick (`) แทน quotes ธรรมดา ทำให้การสร้าง string ซับซ้อนๆ ง่ายขึ้นมาก และฟีเจอร์ขั้นสูงอย่าง Tagged Template Literals ช่วยให้เราปรับแต่งการแปลง string ได้อย่างทรงพลัง

---

## Step 451: Template Literal Syntax ทบทวน

```javascript
// แบบเก่า: String concatenation
const name = 'สมชาย';
const age = 25;
const old = 'สวัสดี, ฉันชื่อ ' + name + ' อายุ ' + age + ' ปี';

// แบบใหม่: Template literal
const modern = `สวัสดี, ฉันชื่อ ${name} อายุ ${age} ปี`;

console.log(old);    // 'สวัสดี, ฉันชื่อ สมชาย อายุ 25 ปี'
console.log(modern); // 'สวัสดี, ฉันชื่อ สมชาย อายุ 25 ปี'

// Template literal สามารถ span หลายบรรทัด
const multiline = `บรรทัดที่ 1
บรรทัดที่ 2
บรรทัดที่ 3`;

console.log(multiline);
// บรรทัดที่ 1
// บรรทัดที่ 2
// บรรทัดที่ 3

// Escape backtick ด้วย backslash
const withBacktick = `นี่คือ backtick: \``;
console.log(withBacktick); // 'นี่คือ backtick: `'
```

---

## Step 452: Multi-line Strings

```javascript
// HTML templates
const htmlTemplate = `
<!DOCTYPE html>
<html>
  <head>
    <title>My Page</title>
  </head>
  <body>
    <h1>สวัสดีโลก</h1>
  </body>
</html>
`;

// ข้อควรระวัง: ช่องว่างก็รวมอยู่ด้วย
const indented = `
  Hello
  World
`;
console.log(indented.startsWith('\n')); // true
console.log(indented.endsWith('\n')); // true

// วิธีจัดการ indentation
function dedent(str) {
  const lines = str.split('\n').filter(line => line.trim().length > 0);
  const indent = lines.reduce((min, line) => {
    const match = line.match(/^(\s+)/);
    return match ? Math.min(min, match[1].length) : min;
  }, Infinity);
  
  return lines.map(line => line.slice(indent)).join('\n');
}

const code = dedent(`
  function hello() {
    console.log('Hello!');
  }
`);
console.log(code);
// function hello() {
//   console.log('Hello!');
// }
```

```javascript
// SQL queries (multi-line)
const userId = 123;
const query = `
  SELECT 
    u.id,
    u.name,
    u.email,
    COUNT(o.id) as order_count,
    SUM(o.total) as total_spent
  FROM users u
  LEFT JOIN orders o ON u.id = o.user_id
  WHERE u.id = ${userId}
    AND u.active = true
  GROUP BY u.id
  ORDER BY total_spent DESC
`;

// CSS multi-line
const styles = `
  .card {
    background: white;
    border-radius: 8px;
    padding: 16px;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
  }
  
  .card:hover {
    box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
    transform: translateY(-2px);
  }
`;
```

---

## Step 453: Expression Interpolation

```javascript
// Interpolate any expression
const a = 10, b = 20;

// Arithmetic
console.log(`${a} + ${b} = ${a + b}`); // '10 + 20 = 30'

// Ternary
const isAdult = (age) => `${age >= 18 ? 'ผู้ใหญ่' : 'เด็ก'}`;
console.log(isAdult(20)); // 'ผู้ใหญ่'
console.log(isAdult(15)); // 'เด็ก'

// Function calls
const greet = (name) => `สวัสดี, ${name.toUpperCase()}!`;
console.log(greet('สมชาย')); // 'สวัสดี, สมชาย!'

// Method calls
const arr = [1, 2, 3, 4, 5];
console.log(`Array: ${arr.join(', ')}`); // 'Array: 1, 2, 3, 4, 5'

// Object property access
const user = { name: 'สมชาย', scores: [85, 90, 78] };
console.log(`${user.name}'s average: ${(user.scores.reduce((a, b) => a + b) / user.scores.length).toFixed(1)}`);
// "สมชาย's average: 84.3"

// Array methods
const items = ['แอปเปิล', 'กล้วย', 'ส้ม'];
console.log(`Items: ${items.map(i => `• ${i}`).join('\n')}`);
// Items: • แอปเปิล
//        • กล้วย
//        • ส้ม
```

---

## Step 454: Nested Template Literals

```javascript
// Template ซ้อน template
const users = [
  { name: 'สมชาย', role: 'admin' },
  { name: 'สมหญิง', role: 'user' },
];

const userList = `
<ul>
  ${users.map(u => `<li class="${u.role}">${u.name}</li>`).join('\n  ')}
</ul>
`;
console.log(userList);
// <ul>
//   <li class="admin">สมชาย</li>
//   <li class="user">สมหญิง</li>
// </ul>

// Conditional nested templates
function renderCard({ title, subtitle, badge, content, actions = [] }) {
  return `
<div class="card">
  <div class="card-header">
    <h2>${title}</h2>
    ${subtitle ? `<p class="subtitle">${subtitle}</p>` : ''}
    ${badge ? `<span class="badge badge-${badge.type}">${badge.text}</span>` : ''}
  </div>
  <div class="card-body">
    ${content}
  </div>
  ${actions.length > 0 ? `
  <div class="card-footer">
    ${actions.map(a => `<button class="btn btn-${a.type}" onclick="${a.handler}">${a.label}</button>`).join('\n    ')}
  </div>` : ''}
</div>
  `.trim();
}

console.log(renderCard({
  title: 'สวัสดีโลก',
  subtitle: 'บทนำ JavaScript',
  badge: { type: 'success', text: 'ใหม่' },
  content: 'เนื้อหาบทเรียน...',
  actions: [
    { type: 'primary', label: 'อ่านต่อ', handler: 'readMore()' },
    { type: 'secondary', label: 'บันทึก', handler: 'save()' },
  ],
}));
```

---

## Step 455: Tagged Template Literals - บทนำ

Tagged Template Literals เป็นฟีเจอร์ขั้นสูงที่ให้เรา "tag" template literal ด้วย function เพื่อควบคุมการแปลงค่า

```javascript
// Syntax พื้นฐาน: tagFunction`template string`
// tagFunction ถูกเรียกด้วย:
// - strings: array ของ string segments
// - ...values: ค่าที่ interpolate

function myTag(strings, ...values) {
  console.log('Strings:', strings);
  console.log('Values:', values);
  return 'tagged result';
}

const name = 'สมชาย';
const age = 25;
const result = myTag`สวัสดี ${name} อายุ ${age} ปี`;
// Strings: ['สวัสดี ', ' อายุ ', ' ปี']
// Values: ['สมชาย', 25]

// strings.length === values.length + 1 เสมอ
```

```javascript
// Tag function ที่ทำงานเหมือน template literal ธรรมดา
function identity(strings, ...values) {
  return strings.reduce((result, str, i) => {
    return result + str + (values[i] !== undefined ? values[i] : '');
  }, '');
}

const name = 'สมชาย';
console.log(identity`สวัสดี ${name}!`); // 'สวัสดี สมชาย!'

// Simplify ด้วย zip
function zip(strings, values) {
  return strings.map((str, i) => str + (values[i] ?? '')).join('');
}

function tag(strings, ...values) {
  return zip(strings, values);
}
```

---

## Step 456: Tag Function Parameters โดยละเอียด

```javascript
// สำรวจ strings array
function explore(strings, ...values) {
  console.log('--- Template Analysis ---');
  console.log(`Total segments: ${strings.length}`);
  console.log(`Total values: ${values.length}`);
  
  strings.forEach((str, i) => {
    console.log(`Segment ${i}: "${str}"`);
    if (i < values.length) {
      console.log(`Value ${i}: ${JSON.stringify(values[i])}`);
    }
  });
  
  return 'analyzed';
}

explore`Hello ${'World'} and ${'Universe'}!`;
// --- Template Analysis ---
// Total segments: 3
// Total values: 2
// Segment 0: "Hello "
// Value 0: "World"
// Segment 1: " and "
// Value 1: "Universe"
// Segment 2: "!"

// strings.raw - original string (ไม่ process escape sequences)
function showRaw(strings, ...values) {
  console.log('Cooked:', strings[0]); // 'Line1\nLine2' - \n เป็น newline
  console.log('Raw:', strings.raw[0]); // 'Line1\\nLine2' - \\n เป็น 2 chars
}

showRaw`Line1\nLine2`;
```

---

## Step 457: Building a Tag Function จาก Scratch

```javascript
// สร้าง tag function ที่ format ตัวเลข
function currency(strings, ...values) {
  return strings.reduce((result, str, i) => {
    const value = values[i - 1];
    if (typeof value === 'number') {
      return result + value.toLocaleString('th-TH', {
        style: 'currency',
        currency: 'THB',
      }) + str;
    }
    return result + (value !== undefined ? value : '') + str;
  });
}

const price = 1500.50;
const tax = 105.035;
console.log(currency`ราคา: ${price} ภาษี: ${tax} รวม: ${price + tax}`);
// 'ราคา: ฿1,500.50 ภาษี: ฿105.04 รวม: ฿1,605.54'
```

```javascript
// สร้าง tag function ที่ highlight values
function highlight(strings, ...values) {
  return strings.reduce((result, str, i) => {
    const value = values[i - 1];
    if (value !== undefined) {
      return result + `<mark>${value}</mark>` + str;
    }
    return result + str;
  });
}

const searchTerm = 'JavaScript';
const count = 42;
console.log(highlight`พบ ${count} ผลลัพธ์สำหรับ "${searchTerm}"`);
// 'พบ <mark>42</mark> ผลลัพธ์สำหรับ "<mark>JavaScript</mark>"'
```

```javascript
// สร้าง tag function ที่ handle types ต่างกัน
function smart(strings, ...values) {
  return strings.reduce((result, str, i) => {
    const value = values[i - 1];
    
    if (value === undefined || value === null) {
      return result + str;
    }
    
    let formatted;
    if (typeof value === 'number') {
      formatted = value.toLocaleString('th-TH');
    } else if (value instanceof Date) {
      formatted = value.toLocaleDateString('th-TH', {
        year: 'numeric', month: 'long', day: 'numeric'
      });
    } else if (Array.isArray(value)) {
      formatted = value.join(', ');
    } else if (typeof value === 'boolean') {
      formatted = value ? 'ใช่' : 'ไม่ใช่';
    } else {
      formatted = String(value);
    }
    
    return result + formatted + str;
  });
}

const count = 1500000;
const date = new Date('2024-01-15');
const active = true;
const tags = ['JavaScript', 'ES6', 'Template Literals'];

console.log(smart`จำนวน: ${count}`);
// 'จำนวน: 1,500,000'

console.log(smart`วันที่: ${date}`);
// 'วันที่: 15 มกราคม 2567'

console.log(smart`สถานะ: ${active}`);
// 'สถานะ: ใช่'

console.log(smart`แท็ก: ${tags}`);
// 'แท็ก: JavaScript, ES6, Template Literals'
```

---

## Step 458: html Tag สำหรับ Sanitizing HTML

```javascript
// ปัญหา XSS (Cross-Site Scripting)
const userInput = '<script>alert("XSS!")</script>';
const unsafe = `<p>สวัสดี, ${userInput}</p>`; // อันตราย!

// html tag ที่ escape HTML entities
function escapeHtml(str) {
  return String(str)
    .replace(/&/g, '&amp;')
    .replace(/</g, '&lt;')
    .replace(/>/g, '&gt;')
    .replace(/"/g, '&quot;')
    .replace(/'/g, '&#39;');
}

function html(strings, ...values) {
  return strings.reduce((result, str, i) => {
    const value = values[i - 1];
    if (value === undefined) return result + str;
    
    // ถ้า value เป็น SafeHTML (trust it)
    if (value && value.__isSafeHtml) {
      return result + value.content + str;
    }
    
    return result + escapeHtml(value) + str;
  });
}

// สร้าง safe HTML wrapper
function safeHtml(content) {
  return { __isSafeHtml: true, content };
}

const name = '<script>alert("hack")</script>';
const safe = html`<p>สวัสดี, ${name}</p>`;
console.log(safe);
// '<p>สวัสดี, &lt;script&gt;alert(&quot;hack&quot;)&lt;/script&gt;</p>'

// ใช้ safeHtml สำหรับเนื้อหาที่ trust
const trustedIcon = safeHtml('<span class="icon">✓</span>');
const result = html`<div>${trustedIcon} สำเร็จ</div>`;
console.log(result); // '<div><span class="icon">✓</span> สำเร็จ</div>'
```

```javascript
// html tag ที่สมบูรณ์กว่า
class SafeHTML {
  constructor(content) {
    this.content = content;
  }
  toString() {
    return this.content;
  }
}

function html(strings, ...values) {
  const result = strings.reduce((acc, str, i) => {
    if (i === 0) return str;
    
    const value = values[i - 1];
    
    if (value instanceof SafeHTML) {
      return acc + value.content + str;
    }
    
    if (Array.isArray(value)) {
      // ถ้าเป็น array ของ SafeHTML - join them
      return acc + value.map(v => 
        v instanceof SafeHTML ? v.content : escapeHtml(v)
      ).join('') + str;
    }
    
    return acc + escapeHtml(value ?? '') + str;
  }, '');
  
  return new SafeHTML(result);
}

// ทดสอบ
const items = ['A', 'B', '<script>hack</script>'];
const list = html`
  <ul>
    ${items.map(item => html`<li>${item}</li>`)}
  </ul>
`;
console.log(list.content.trim());
```

---

## Step 459: css Tag สำหรับ CSS-in-JS

```javascript
// CSS-in-JS pattern
function css(strings, ...values) {
  return strings.reduce((result, str, i) => {
    const value = values[i - 1];
    if (value === undefined) return result + str;
    if (typeof value === 'number') {
      return result + value + 'px' + str; // auto-add px
    }
    return result + String(value) + str;
  });
}

const primaryColor = '#007bff';
const fontSize = 16; // number - auto px
const padding = 12;

const buttonStyles = css`
  background-color: ${primaryColor};
  color: white;
  font-size: ${fontSize};
  padding: ${padding} ${padding * 2};
  border-radius: 4;
  border: none;
  cursor: pointer;
`;

console.log(buttonStyles);
// 'background-color: #007bff; color: white; font-size: 16px; padding: 12px 24px; ...'
```

```javascript
// CSS-in-JS ที่สมบูรณ์กว่า
class StyleSheet {
  #styles = new Map();
  
  inject(styles) {
    const id = `style-${Date.now()}-${Math.random().toString(36).slice(2)}`;
    this.#styles.set(id, styles);
    return id;
  }
  
  getStyles() {
    return [...this.#styles.entries()]
      .map(([id, styles]) => `.${id} { ${styles} }`)
      .join('\n');
  }
}

const sheet = new StyleSheet();

function styled(tagName) {
  return function(strings, ...values) {
    const cssText = strings.reduce((acc, str, i) => {
      const value = values[i - 1];
      if (value === undefined) return acc + str;
      return acc + value + str;
    });
    
    const className = sheet.inject(cssText);
    
    return function(content = '') {
      return `<${tagName} class="${className}">${content}</${tagName}>`;
    };
  };
}

const Button = styled('button')`
  background: blue;
  color: white;
  padding: 8px 16px;
  border-radius: 4px;
`;

console.log(Button('คลิกฉัน'));
// '<button class="style-xxx-yyy">คลิกฉัน</button>'
```

---

## Step 460: sql Tag สำหรับ SQL Queries

```javascript
// SQL injection prevention ด้วย tagged template
class SqlQuery {
  constructor(text, values) {
    this.text = text;
    this.values = values;
  }
}

function sql(strings, ...values) {
  let queryText = '';
  const queryValues = [];
  
  strings.forEach((str, i) => {
    queryText += str;
    if (i < values.length) {
      queryValues.push(values[i]);
      queryText += `$${queryValues.length}`; // PostgreSQL style
    }
  });
  
  return new SqlQuery(queryText, queryValues);
}

// ตัวอย่างใช้งาน
const userId = 123;
const status = 'active';

const query = sql`
  SELECT * FROM users 
  WHERE id = ${userId} 
  AND status = ${status}
`;

console.log(query.text);
// 'SELECT * FROM users WHERE id = $1 AND status = $2'
console.log(query.values);
// [123, 'active']

// เปรียบเทียบกับแบบอันตราย
// const unsafe = `SELECT * FROM users WHERE id = ${userId}`;
// userId = "1; DROP TABLE users;" - SQL Injection!
```

```javascript
// SQL tag ที่รองรับ subqueries
function sql(strings, ...values) {
  let text = '';
  const params = [];
  
  strings.forEach((str, i) => {
    text += str;
    if (i < values.length) {
      const value = values[i];
      if (value instanceof SqlQuery) {
        // Merge subquery
        const offset = params.length;
        text += value.text.replace(/\$(\d+)/g, (_, n) => `$${parseInt(n) + offset}`);
        params.push(...value.values);
      } else {
        params.push(value);
        text += `$${params.length}`;
      }
    }
  });
  
  return { text, params };
}

// ใช้ subquery
const subQuery = sql`SELECT id FROM admins WHERE active = ${true}`;
const mainQuery = sql`
  SELECT * FROM users 
  WHERE id IN (${subQuery})
    AND name LIKE ${'%สม%'}
`;

console.log(mainQuery.text);
// SELECT * FROM users WHERE id IN (SELECT id FROM admins WHERE active = $1) AND name LIKE $2
console.log(mainQuery.params); // [true, '%สม%']
```

---

## Step 461: i18n Tag สำหรับ Internationalization

```javascript
// i18n tag function สำหรับการแปลภาษา
const translations = {
  th: {
    'welcome': 'ยินดีต้อนรับ',
    'hello {name}': 'สวัสดี {name}',
    'you have {count} messages': 'คุณมี {count} ข้อความ',
    'price is {amount}': 'ราคา {amount} บาท',
  },
  en: {
    'welcome': 'Welcome',
    'hello {name}': 'Hello {name}',
    'you have {count} messages': 'You have {count} messages',
    'price is {amount}': 'Price is {amount}',
  },
  ja: {
    'hello {name}': 'こんにちは {name}',
    'you have {count} messages': '{count}件のメッセージがあります',
  }
};

let currentLang = 'th';

function i18n(strings, ...values) {
  // สร้าง template key (ใช้ {0}, {1}, ... แทน actual values)
  const template = strings.reduce((acc, str, i) => {
    if (i === 0) return str;
    const index = i - 1;
    return acc + `{${index}}` + str;
  }, '');
  
  const lang = translations[currentLang] || translations['en'];
  
  // หา translation โดย pattern matching
  const translated = findTranslation(lang, template, strings, values);
  
  return translated || template;
}

function findTranslation(lang, template, strings, values) {
  // Simple implementation
  const key = strings.join('{value}');
  const found = lang[key];
  if (!found) return null;
  
  return values.reduce((str, val, i) => str.replace(`{${i}}`, val), found);
}

// Better i18n approach
class I18n {
  #locale = 'th';
  #translations = {};
  
  setLocale(locale) {
    this.#locale = locale;
    return this;
  }
  
  load(locale, messages) {
    this.#translations[locale] = messages;
    return this;
  }
  
  get tag() {
    return (strings, ...values) => {
      const rawKey = strings.join('__VALUE__');
      const translated = this.#translations[this.#locale]?.[rawKey]
        || this.#translations['en']?.[rawKey]
        || strings.reduce((acc, str, i) => acc + (values[i - 1] ?? '') + str);
      
      if (typeof translated === 'function') {
        return translated(...values);
      }
      
      return values.reduce(
        (str, val, i) => str.replace(`{${i}}`, val),
        translated
      );
    };
  }
}

const i18nInstance = new I18n();
i18nInstance
  .load('th', {
    'Hello, __VALUE__!': (name) => `สวัสดี, ${name}!`,
    'You have __VALUE__ items': (count) => `คุณมี ${count} รายการ`,
  })
  .load('en', {
    'Hello, __VALUE__!': (name) => `Hello, ${name}!`,
    'You have __VALUE__ items': (count) => `You have ${count} items`,
  });

const t = i18nInstance.tag;

i18nInstance.setLocale('th');
console.log(t`Hello, ${'สมชาย'}!`); // 'สวัสดี, สมชาย!'
console.log(t`You have ${5} items`); // 'คุณมี 5 รายการ'

i18nInstance.setLocale('en');
console.log(t`Hello, ${'Somchai'}!`); // 'Hello, Somchai!'
```

---

## Step 462: String.raw Tag

```javascript
// String.raw คือ built-in tagged template
// ส่งคืน raw string ที่ไม่ process escape sequences

// ปกติ
console.log(`Hello\nWorld`);
// Hello
// World (newline จริงๆ)

// String.raw
console.log(String.raw`Hello\nWorld`);
// Hello\nWorld (literal backslash-n)

// ใช้งาน: Windows paths
const path = String.raw`C:\Users\สมชาย\Documents\file.txt`;
console.log(path); // C:\Users\สมชาย\Documents\file.txt

// ใช้งาน: RegExp patterns
const regexStr = String.raw`^\d{4}-\d{2}-\d{2}$`;
const dateRegex = new RegExp(regexStr);
console.log(dateRegex.test('2024-01-15')); // true

// ใช้งาน: LaTeX
const latex = String.raw`\frac{-b \pm \sqrt{b^2 - 4ac}}{2a}`;
console.log(latex); // \frac{-b \pm \sqrt{b^2 - 4ac}}{2a}
```

---

## Step 463: Practical Tagged Template - Debug Tag

```javascript
// Debug tag สำหรับ logging
function debug(strings, ...values) {
  const parts = strings.map((str, i) => {
    if (i === 0) return str;
    const value = values[i - 1];
    const type = typeof value;
    const displayValue = JSON.stringify(value);
    return `\x1b[33m[${type}: ${displayValue}]\x1b[0m${str}`;
    // \x1b[33m = yellow color in terminal
  });
  
  return parts.join('');
}

const user = { name: 'สมชาย' };
const count = 42;
const active = true;

console.log(debug`User: ${user}, Count: ${count}, Active: ${active}`);
// User: [object: {"name":"สมชาย"}], Count: [number: 42], Active: [boolean: true]
```

```javascript
// Trace tag - แสดง call stack
function trace(strings, ...values) {
  const message = strings.reduce((acc, str, i) => {
    return acc + (values[i - 1] ?? '') + str;
  });
  
  const stack = new Error().stack.split('\n').slice(2, 5).join('\n');
  
  return `${message}\n  Called from:\n${stack}`;
}

function outerFunction() {
  function innerFunction() {
    return trace`เกิดขึ้นที่นี่`;
  }
  return innerFunction();
}

// console.log(outerFunction()); // แสดง call stack
```

---

## Step 464: Practical Tagged Template - Interpolation Control

```javascript
// Limit string length ใน interpolation
function truncate(maxLength = 20) {
  return function(strings, ...values) {
    return strings.reduce((result, str, i) => {
      const value = values[i - 1];
      if (value === undefined) return result + str;
      
      const strValue = String(value);
      const truncated = strValue.length > maxLength
        ? strValue.slice(0, maxLength) + '...'
        : strValue;
      
      return result + truncated + str;
    });
  };
}

const shortTag = truncate(10);
const longText = 'นี่คือข้อความที่ยาวมากๆ จริงๆ';
console.log(shortTag`ข้อความ: ${longText}`);
// 'ข้อความ: นี่คือข้อความ...'

// Format numbers
function format(options = {}) {
  return function(strings, ...values) {
    return strings.reduce((result, str, i) => {
      const value = values[i - 1];
      if (value === undefined) return result + str;
      
      if (typeof value === 'number') {
        const formatted = value.toLocaleString('th-TH', options);
        return result + formatted + str;
      }
      
      return result + String(value) + str;
    });
  };
}

const money = format({ style: 'currency', currency: 'THB' });
const amount = 1500000.50;
console.log(money`ยอดรวม: ${amount}`);
// 'ยอดรวม: ฿1,500,000.50'
```

---

## Step 465: Template Literals vs String Concatenation

```javascript
// Performance test (conceptual)
const iterations = 1000000;
const name = 'สมชาย';
const age = 25;

// Method 1: Concatenation
console.time('concat');
let result1;
for (let i = 0; i < iterations; i++) {
  result1 = 'สวัสดี, ' + name + ' อายุ ' + age + ' ปี';
}
console.timeEnd('concat');

// Method 2: Template literal
console.time('template');
let result2;
for (let i = 0; i < iterations; i++) {
  result2 = `สวัสดี, ${name} อายุ ${age} ปี`;
}
console.timeEnd('template');

// Method 3: Array join
console.time('join');
let result3;
for (let i = 0; i < iterations; i++) {
  result3 = ['สวัสดี, ', name, ' อายุ ', age, ' ปี'].join('');
}
console.timeEnd('join');
```

```javascript
// ความแตกต่างที่สำคัญ (ไม่ใช่แค่ performance)

// 1. Multi-line: Template literals ชนะ
const multilineTemplate = `
  Line 1
  Line 2
  Line 3
`;

const multilineConcat = '  Line 1\n' +
  '  Line 2\n' +
  '  Line 3\n';

// 2. Complex expressions: Template literals อ่านง่ายกว่า
const scores = [85, 90, 78];
const avg = scores.reduce((a, b) => a + b) / scores.length;

const readable = `เฉลี่ย: ${avg.toFixed(2)}`;
const awkward = 'เฉลี่ย: ' + avg.toFixed(2);

// 3. Nested: Template literals ง่ายกว่า
const html = `
  <ul>
    ${items.map(item => `<li>${item}</li>`).join('')}
  </ul>
`;

// 4. Tagged templates: ทำได้เฉพาะ template literals
const safe = html`<p>${userInput}</p>`; // sanitized
```

---

## Step 466: Advanced Pattern - Template Engine

```javascript
// Mini template engine ด้วย tagged template literals
class TemplateEngine {
  #partials = new Map();
  #helpers = new Map();
  
  registerPartial(name, template) {
    this.#partials.set(name, template);
    return this;
  }
  
  registerHelper(name, fn) {
    this.#helpers.set(name, fn);
    return this;
  }
  
  render(strings, ...values) {
    return strings.reduce((result, str, i) => {
      if (i === 0) return str;
      
      const value = values[i - 1];
      
      // Partial rendering
      if (value && value.__partial) {
        const partial = this.#partials.get(value.__partial);
        return result + (partial?.(value.context) || '') + str;
      }
      
      // Helper function
      if (value && value.__helper) {
        const helper = this.#helpers.get(value.__helper.name);
        return result + (helper?.(value.__helper.args) || '') + str;
      }
      
      return result + (value ?? '') + str;
    });
  }
  
  partial(name, context = {}) {
    return { __partial: name, context };
  }
  
  helper(name, ...args) {
    return { __helper: { name, args } };
  }
}

const engine = new TemplateEngine();

engine.registerPartial('userCard', ({ name, role }) =>
  `<div class="user-card"><span>${name}</span><small>${role}</small></div>`
);

engine.registerHelper('pluralize', ([count, singular, plural]) =>
  `${count} ${count === 1 ? singular : plural}`
);

const t = engine.render.bind(engine);
const users = [
  { name: 'สมชาย', role: 'admin' },
  { name: 'สมหญิง', role: 'user' },
];

const page = t`
  <div>
    <h2>${engine.helper('pluralize', users.length, 'ผู้ใช้', 'ผู้ใช้')}</h2>
    ${users.map(u => engine.partial('userCard', u)).join('')}
  </div>
`;
```

---

## Step 467: Template Literals กับ Regular Expressions

```javascript
// สร้าง regex ที่อ่านง่าย
function regex(strings, ...values) {
  const source = strings.reduce((acc, str, i) => {
    if (i === 0) return str;
    const value = values[i - 1];
    // escape special regex chars ใน values
    const escaped = String(value).replace(/[.*+?^${}()|[\]\\]/g, '\\$&');
    return acc + escaped + str;
  });
  
  return new RegExp(source);
}

const protocol = 'https?';
const domain = 'example.com'; // user input - needs escaping
const urlRegex = regex`^${protocol}://${domain}/`;
console.log(urlRegex);
// /^https?:\/\/example\.com\//

// สร้าง regex patterns ที่ซับซ้อน
function buildRegex({ flags = '' } = {}) {
  return function(strings, ...values) {
    const source = strings.reduce((acc, str, i) => {
      if (i === 0) return str.trim();
      const value = values[i - 1];
      const part = value instanceof RegExp ? value.source : String(value).trim();
      return acc + part + str.trim();
    });
    return new RegExp(source, flags);
  };
}

const re = buildRegex({ flags: 'gi' });
const datePattern = re`
  (\d{4})  # year
  -
  (\d{2})  # month
  -
  (\d{2})  # day
`;
// (จะ error เพราะมี comments - แค่แสดง pattern)
```

---

## Step 468: Template Literals กับ Generators

```javascript
// Lazy template evaluation ด้วย generator
function* lazyTemplate(strings, ...valueGetters) {
  for (let i = 0; i < strings.length; i++) {
    yield strings[i];
    if (i < valueGetters.length) {
      const value = typeof valueGetters[i] === 'function'
        ? valueGetters[i]()
        : valueGetters[i];
      yield String(value);
    }
  }
}

// สร้าง tag function จาก generator
function lazy(strings, ...values) {
  // values สามารถเป็น functions ที่จะถูกเรียกทีหลัง
  return {
    evaluate() {
      return strings.reduce((acc, str, i) => {
        if (i === 0) return str;
        const value = values[i - 1];
        return acc + (typeof value === 'function' ? value() : value) + str;
      });
    },
    
    toString() {
      return this.evaluate();
    }
  };
}

let counter = 0;
const getCount = () => ++counter;

const template = lazy`ค่า: ${getCount}, ค่า: ${getCount}`;
console.log(counter); // 0 - ยังไม่ถูกเรียก!

console.log(template.evaluate()); // 'ค่า: 1, ค่า: 2'
console.log(template.evaluate()); // 'ค่า: 3, ค่า: 4' (เรียกใหม่ทุกครั้ง)
```

---

## Step 469: Template Literals ใน Real-world Applications

```javascript
// GraphQL queries
function gql(strings, ...values) {
  const query = strings.reduce((acc, str, i) => {
    if (i === 0) return str;
    const value = values[i - 1];
    // Handle fragments
    if (value && value.__gqlFragment) {
      return acc + value.fragment + str;
    }
    return acc + String(value) + str;
  });
  
  return {
    query: query.trim(),
    toString() { return this.query; }
  };
}

const UserFields = {
  __gqlFragment: true,
  fragment: `
    fragment UserFields on User {
      id
      name
      email
    }
  `
};

const userId = 123;
const getUserQuery = gql`
  query GetUser {
    user(id: ${userId}) {
      ${UserFields}
      createdAt
      role
    }
  }
`;

console.log(getUserQuery.query);
```

```javascript
// Error messages ที่ rich
function error(strings, ...values) {
  const message = strings.reduce((acc, str, i) => {
    if (i === 0) return str;
    const value = values[i - 1];
    return acc + JSON.stringify(value, null, 2) + str;
  });
  
  const err = new Error(message);
  err.template = strings;
  err.values = values;
  return err;
}

function validateAge(age) {
  if (typeof age !== 'number') {
    throw error`Expected age to be a number, got ${typeof age} (${age})`;
  }
  if (age < 0 || age > 150) {
    throw error`Age ${age} is out of valid range [0, 150]`;
  }
  return age;
}

try {
  validateAge('25');
} catch (e) {
  console.log(e.message);
  // Expected age to be a number, got "string" ("25")
}
```

---

## Step 470: สรุปและ Best Practices

```javascript
// Best Practices

// 1. ใช้ template literals สำหรับ strings ที่มี interpolation
// ✓
const greeting = `สวัสดี, ${name}`;
// ✗ (ไม่จำเป็น)
const staticString = `Hello World`; // ไม่ต้อง template ถ้าไม่มี interpolation

// 2. ใช้ tagged templates สำหรับ security
const safeOutput = html`<p>${userInput}</p>`;
const safeQuery = sql`SELECT * FROM users WHERE id = ${userId}`;

// 3. String.raw สำหรับ patterns
const pattern = String.raw`\d{4}-\d{2}-\d{2}`;
const re = new RegExp(pattern);

// 4. หลีกเลี่ยง logic ที่ซับซ้อนใน interpolation
// ✗ ยากอ่าน
const bad = `${users.filter(u => u.active).map(u => u.name).join(', ')}`;

// ✓ แยกออกมาก่อน
const activeNames = users.filter(u => u.active).map(u => u.name).join(', ');
const good = `Active users: ${activeNames}`;

// 5. สร้าง reusable tag functions
function createFormatter(options) {
  return function(strings, ...values) {
    return strings.reduce((acc, str, i) => {
      const value = values[i - 1];
      if (typeof value === 'number') {
        return acc + value.toLocaleString('th-TH', options) + str;
      }
      return acc + (value ?? '') + str;
    });
  };
}

const money = createFormatter({ style: 'currency', currency: 'THB' });
const percent = createFormatter({ style: 'percent', minimumFractionDigits: 1 });

console.log(money`ราคา: ${1500000}`);
// 'ราคา: ฿1,500,000.00'

console.log(percent`อัตรา: ${0.185}`);
// 'อัตรา: 18.5%'
```

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: สร้าง Template Engine อย่างง่าย

```javascript
// Task: สร้าง tagged template สำหรับ generate HTML table

function table(strings, ...values) {
  const [headers, ...rows] = values;
  
  const headerRow = headers.map(h => `<th>${h}</th>`).join('');
  const bodyRows = rows.map(row =>
    `<tr>${row.map(cell => `<td>${cell}</td>`).join('')}</tr>`
  ).join('\n');
  
  return `
<table>
  <thead><tr>${headerRow}</tr></thead>
  <tbody>
    ${bodyRows}
  </tbody>
</table>`.trim();
}

// แต่ tag functions ทำงานแตกต่างออกไป
// ควรทดสอบ:
const headers = ['ชื่อ', 'อายุ', 'เมือง'];
const rows = [
  ['สมชาย', 25, 'กรุงเทพ'],
  ['สมหญิง', 30, 'เชียงใหม่'],
];

// สร้าง function ที่ทำงานได้
function createTable(headers, rows) {
  const headerHtml = headers.map(h => `<th>${h}</th>`).join('');
  const rowsHtml = rows.map(row =>
    `<tr>${row.map(cell => `<td>${cell}</td>`).join('')}</tr>`
  ).join('\n');
  
  return `<table><thead><tr>${headerHtml}</tr></thead><tbody>${rowsHtml}</tbody></table>`;
}

console.log(createTable(headers, rows));
```

### แบบฝึกหัดที่ 2: สร้าง Validation Tag

```javascript
// Task: สร้าง tag function ที่ validate ค่าขณะ interpolate

function validate(rules) {
  return function(strings, ...values) {
    const errors = [];
    const result = strings.reduce((acc, str, i) => {
      if (i === 0) return str;
      const value = values[i - 1];
      const rule = rules[i - 1];
      
      if (rule) {
        if (rule.required && (value === null || value === undefined || value === '')) {
          errors.push(`ค่าที่ ${i} จำเป็นต้องมี`);
        }
        if (rule.type && typeof value !== rule.type) {
          errors.push(`ค่าที่ ${i} ต้องเป็น ${rule.type}`);
        }
        if (rule.min !== undefined && value < rule.min) {
          errors.push(`ค่าที่ ${i} ต้องมากกว่าหรือเท่ากับ ${rule.min}`);
        }
        if (rule.max !== undefined && value > rule.max) {
          errors.push(`ค่าที่ ${i} ต้องน้อยกว่าหรือเท่ากับ ${rule.max}`);
        }
      }
      
      return acc + String(value ?? '') + str;
    });
    
    return { result, errors, isValid: errors.length === 0 };
  };
}

const v = validate([
  { required: true, type: 'string' },   // name
  { required: true, type: 'number', min: 0, max: 150 }, // age
]);

const { result, errors, isValid } = v`ชื่อ: ${'สมชาย'}, อายุ: ${25}`;
console.log(isValid); // true
console.log(result);  // 'ชื่อ: สมชาย, อายุ: 25'

const bad = v`ชื่อ: ${''}, อายุ: ${200}`;
console.log(bad.isValid); // false
console.log(bad.errors);  // ['ค่าที่ 1 จำเป็นต้องมี', 'ค่าที่ 2 ต้องน้อยกว่าหรือเท่ากับ 150']
```

### แบบฝึกหัดที่ 3: สร้าง Logger ด้วย Tagged Templates

```javascript
// Task: สร้าง structured logger ด้วย tagged template

class Logger {
  #level = 'info';
  #format = 'text';
  
  setLevel(level) { this.#level = level; return this; }
  setFormat(format) { this.#format = format; return this; }
  
  #createTag(level) {
    return (strings, ...values) => {
      const message = strings.reduce((acc, str, i) => {
        return acc + (values[i - 1] ?? '') + str;
      });
      
      const entry = {
        timestamp: new Date().toISOString(),
        level: level.toUpperCase(),
        message,
      };
      
      if (this.#format === 'json') {
        console.log(JSON.stringify(entry));
      } else {
        console.log(`[${entry.timestamp}] [${entry.level}] ${entry.message}`);
      }
    };
  }
  
  get info() { return this.#createTag('info'); }
  get warn() { return this.#createTag('warn'); }
  get error() { return this.#createTag('error'); }
  get debug() { return this.#createTag('debug'); }
}

const logger = new Logger();
logger.setFormat('text');

const user = 'สมชาย';
const count = 42;

logger.info`User ${user} has ${count} items`;
// [2024-01-01T00:00:00.000Z] [INFO] User สมชาย has 42 items

logger.error`Failed to process: ${new Error('Connection refused')}`;
// [2024-01-01T00:00:00.000Z] [ERROR] Failed to process: Error: Connection refused
```

---

## สรุปบทนี้

ในบทนี้เราได้เรียนรู้:

1. **Template Literal พื้นฐาน** - backtick syntax, multi-line strings
2. **Expression Interpolation** - ใส่ expression ใดๆ ใน `${}`
3. **Nested Templates** - template ซ้อน template
4. **Tagged Template Literals** - tag functions ที่ควบคุมการแปลง
5. **Tag Function Parameters** - strings array และ values
6. **Built-in Tags** - String.raw
7. **Practical Tags** - html (XSS prevention), css, sql (injection prevention), i18n
8. **Advanced Patterns** - template engines, lazy evaluation

**หลักการสำคัญ:**
- Tagged templates มีประโยชน์มากสำหรับ security (html, sql)
- String.raw ช่วยกับ regex patterns และ file paths
- สร้าง reusable tag functions แทนที่จะ format ทุกที่
- Tag functions สามารถ return ค่าที่ไม่ใช่ string ก็ได้
