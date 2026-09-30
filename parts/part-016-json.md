# Part 16: JSON (JavaScript Object Notation)
## Steps 291-310

---

## บทนำ

JSON (JavaScript Object Notation) คือรูปแบบการแลกเปลี่ยนข้อมูลที่เบาและอ่านง่าย ถูกคิดค้นโดย Douglas Crockford และกลายเป็นมาตรฐานสำคัญในการส่งข้อมูลระหว่างเซิร์ฟเวอร์และเว็บแอพพลิเคชัน แม้ชื่อจะมี "JavaScript" แต่ JSON เป็นรูปแบบข้อความที่เป็นอิสระจากภาษาโปรแกรมมิ่ง และถูกรองรับโดยภาษาโปรแกรมมิ่งส่วนใหญ่

---

## Step 291: JSON คืออะไร

JSON ย่อมาจาก JavaScript Object Notation เป็นรูปแบบข้อมูลแบบข้อความ (text-based format) ที่ใช้ในการแสดงข้อมูลที่มีโครงสร้าง

### ลักษณะของ JSON

- อ่านง่ายสำหรับมนุษย์
- เบา (lightweight)
- อิสระจากภาษา (language-independent)
- เป็น subset ของ JavaScript

### ตัวอย่าง JSON พื้นฐาน

```json
{
  "name": "สมชาย ใจดี",
  "age": 25,
  "isStudent": true,
  "hobbies": ["อ่านหนังสือ", "ดูหนัง", "เล่นกีฬา"],
  "address": {
    "city": "กรุงเทพ",
    "country": "ไทย"
  },
  "phone": null
}
```

### การใช้งาน JSON ใน JavaScript

```javascript
// JSON เป็น string
const jsonString = '{"name": "สมชาย", "age": 25}';
console.log(typeof jsonString); // "string"

// แปลง JSON string เป็น JavaScript object
const obj = JSON.parse(jsonString);
console.log(typeof obj); // "object"
console.log(obj.name); // "สมชาย"

// แปลง JavaScript object เป็น JSON string
const person = { name: "สมหญิง", age: 30 };
const json = JSON.stringify(person);
console.log(typeof json); // "string"
console.log(json); // '{"name":"สมหญิง","age":30}'
```

---

## Step 292: กฎไวยากรณ์ของ JSON

JSON มีกฎไวยากรณ์ที่เข้มงวดกว่า JavaScript Object

### กฎสำคัญของ JSON

1. **Key ต้องอยู่ใน double quotes เสมอ**
2. **String ต้องใช้ double quotes เท่านั้น** (ไม่ใช้ single quotes)
3. **ไม่มี comment**
4. **ค่าสุดท้ายใน array หรือ object ต้องไม่มี trailing comma**
5. **ไม่รองรับ undefined, function, Symbol**

```javascript
// ถูกต้อง - Valid JSON
const validJSON = `{
  "name": "สมชาย",
  "age": 25,
  "active": true
}`;

// ผิด - Invalid JSON (key ไม่มี quotes)
// { name: "สมชาย" }

// ผิด - Invalid JSON (single quotes)
// { "name": 'สมชาย' }

// ผิด - Invalid JSON (trailing comma)
// { "name": "สมชาย", }

// ผิด - Invalid JSON (comment)
// { "name": "สมชาย" // ชื่อ }

// ตรวจสอบว่า JSON ถูกต้องหรือไม่
function isValidJSON(str) {
  try {
    JSON.parse(str);
    return true;
  } catch (e) {
    return false;
  }
}

console.log(isValidJSON('{"name": "test"}')); // true
console.log(isValidJSON('{name: "test"}')); // false
console.log(isValidJSON("{'name': 'test'}")); // false
```

### เปรียบเทียบ JavaScript Object vs JSON

```javascript
// JavaScript Object - ยืดหยุ่นกว่า
const jsObj = {
  name: 'สมชาย',        // single quotes ได้
  'last-name': 'ใจดี',  // key มี special chars
  greet() {             // method ได้
    return 'สวัสดี';
  },
  created: undefined,   // undefined ได้
  id: Symbol('id')      // Symbol ได้
};

// JSON String - เข้มงวดกว่า
const jsonStr = `{
  "name": "สมชาย",
  "last-name": "ใจดี"
}`;
// method, undefined, Symbol ไม่รองรับใน JSON
```

---

## Step 293: ประเภทข้อมูลใน JSON

JSON รองรับประเภทข้อมูล 6 ประเภท

### 1. String

```javascript
const jsonStr = `{
  "firstName": "สมชาย",
  "lastName": "ใจดี",
  "email": "somchai@example.com",
  "message": "สวัสดี\\nยินดีต้อนรับ"
}`;

const data = JSON.parse(jsonStr);
console.log(data.firstName); // "สมชาย"
console.log(data.message);   // "สวัสดี\nยินดีต้อนรับ"

// Escape characters ใน JSON
const escaped = {
  quote: 'เขาพูดว่า \\"สวัสดี\\"',
  backslash: 'C:\\\\Users\\\\somchai',
  newline: 'บรรทัดที่ 1\\nบรรทัดที่ 2',
  tab: 'คอลัมน์1\\tคอลัมน์2',
  unicode: '\\u0E2A\\u0E27\\u0E31\\u0E2A\\u0E14\\u0E35'
};
```

### 2. Number

```javascript
const numbers = JSON.parse(`{
  "integer": 42,
  "negative": -17,
  "float": 3.14,
  "scientific": 1.5e10,
  "zero": 0
}`);

console.log(numbers.integer);    // 42
console.log(numbers.float);      // 3.14
console.log(numbers.scientific); // 15000000000

// JSON ไม่รองรับ Infinity, NaN
console.log(JSON.stringify({ val: Infinity })); // '{"val":null}'
console.log(JSON.stringify({ val: NaN }));      // '{"val":null}'
```

### 3. Boolean

```javascript
const boolData = JSON.parse(`{
  "isActive": true,
  "isDeleted": false,
  "hasPermission": true
}`);

console.log(boolData.isActive);     // true
console.log(typeof boolData.isActive); // "boolean"
```

### 4. Array

```javascript
const arrayData = JSON.parse(`{
  "colors": ["แดง", "เขียว", "น้ำเงิน"],
  "numbers": [1, 2, 3, 4, 5],
  "mixed": [1, "สอง", true, null],
  "nested": [[1, 2], [3, 4], [5, 6]]
}`);

console.log(arrayData.colors[0]);      // "แดง"
console.log(arrayData.mixed[2]);       // true
console.log(arrayData.nested[1][0]);   // 3
```

### 5. Object

```javascript
const objectData = JSON.parse(`{
  "person": {
    "name": "สมชาย",
    "address": {
      "street": "ถนนสุขุมวิท",
      "city": "กรุงเทพ",
      "zipcode": "10110"
    }
  }
}`);

console.log(objectData.person.name);              // "สมชาย"
console.log(objectData.person.address.city);      // "กรุงเทพ"
```

### 6. Null

```javascript
const nullData = JSON.parse(`{
  "name": "สมชาย",
  "middleName": null,
  "deletedAt": null
}`);

console.log(nullData.middleName);  // null
console.log(nullData.middleName === null); // true

// ความแตกต่างระหว่าง null และ undefined
const obj = JSON.parse('{"a": null}');
console.log(obj.a);   // null (มี property แต่เป็น null)
console.log(obj.b);   // undefined (ไม่มี property)
```

---

## Step 294: JSON.stringify() พื้นฐาน

`JSON.stringify()` แปลง JavaScript value เป็น JSON string

```javascript
// แปลง object
const person = {
  name: "สมชาย",
  age: 25,
  city: "กรุงเทพ"
};
console.log(JSON.stringify(person));
// '{"name":"สมชาย","age":25,"city":"กรุงเทพ"}'

// แปลง array
const fruits = ["มะม่วง", "กล้วย", "ส้ม"];
console.log(JSON.stringify(fruits));
// '["มะม่วง","กล้วย","ส้ม"]'

// แปลงค่าพื้นฐาน
console.log(JSON.stringify(42));       // '42'
console.log(JSON.stringify("hello"));  // '"hello"'
console.log(JSON.stringify(true));     // 'true'
console.log(JSON.stringify(null));     // 'null'

// ค่าที่จะถูกแปลงเป็น null
console.log(JSON.stringify(undefined)); // undefined (ไม่ใช่ string)
console.log(JSON.stringify(Infinity));  // 'null'
console.log(JSON.stringify(NaN));       // 'null'

// Function และ Symbol จะถูกละ
const obj = {
  name: "test",
  func: function() {},
  sym: Symbol('id'),
  und: undefined
};
console.log(JSON.stringify(obj));
// '{"name":"test"}'
```

### ข้อควรระวัง

```javascript
// Circular reference จะทำให้เกิด error
const circObj = {};
circObj.self = circObj;
try {
  JSON.stringify(circObj);
} catch (e) {
  console.log(e.message); // Converting circular structure to JSON
}

// Date objects
const now = new Date();
const dateObj = { created: now };
const jsonDate = JSON.stringify(dateObj);
console.log(jsonDate); // '{"created":"2024-01-15T10:30:00.000Z"}'
// Date ถูกแปลงเป็น ISO string อัตโนมัติ

// BigInt จะเกิด error
try {
  JSON.stringify({ val: 9007199254740991n });
} catch (e) {
  console.log(e.message); // Do not know how to serialize a BigInt
}
```

---

## Step 295: JSON.parse() พื้นฐาน

`JSON.parse()` แปลง JSON string เป็น JavaScript value

```javascript
// แปลง JSON object string
const personJSON = '{"name":"สมชาย","age":25,"city":"กรุงเทพ"}';
const person = JSON.parse(personJSON);
console.log(person.name); // "สมชาย"
console.log(person.age);  // 25

// แปลง JSON array string
const fruitsJSON = '["มะม่วง","กล้วย","ส้ม"]';
const fruits = JSON.parse(fruitsJSON);
console.log(fruits[0]); // "มะม่วง"
console.log(fruits.length); // 3

// แปลงค่าพื้นฐาน
console.log(JSON.parse('42'));       // 42 (number)
console.log(JSON.parse('"hello"'));  // "hello" (string)
console.log(JSON.parse('true'));     // true (boolean)
console.log(JSON.parse('null'));     // null

// JSON ที่มีหลาย level
const complexJSON = `{
  "user": {
    "id": 1,
    "profile": {
      "firstName": "สมชาย",
      "lastName": "ใจดี",
      "contacts": [
        {"type": "email", "value": "somchai@example.com"},
        {"type": "phone", "value": "081-234-5678"}
      ]
    }
  }
}`;

const data = JSON.parse(complexJSON);
console.log(data.user.profile.firstName); // "สมชาย"
console.log(data.user.profile.contacts[0].value); // "somchai@example.com"
```

### การจัดการ Error

```javascript
function safeParse(jsonString) {
  try {
    return { success: true, data: JSON.parse(jsonString) };
  } catch (error) {
    return { success: false, error: error.message };
  }
}

const result1 = safeParse('{"name": "สมชาย"}');
console.log(result1); // { success: true, data: { name: "สมชาย" } }

const result2 = safeParse('{invalid json}');
console.log(result2); // { success: false, error: "..." }

const result3 = safeParse(null);
console.log(result3); // { success: false, error: "..." }
```

---

## Step 296: JSON.stringify() กับ Replacer

Replacer parameter ให้ควบคุมว่าจะ stringify อะไรบ้าง

### Replacer แบบ Array

```javascript
const person = {
  name: "สมชาย",
  age: 25,
  password: "secret123",
  email: "somchai@example.com",
  ssn: "1234567890123"
};

// เลือก properties ที่ต้องการ
const publicData = JSON.stringify(person, ["name", "age", "email"]);
console.log(publicData);
// '{"name":"สมชาย","age":25,"email":"somchai@example.com"}'

// แสดงเฉพาะ name
const nameOnly = JSON.stringify(person, ["name"]);
console.log(nameOnly); // '{"name":"สมชาย"}'
```

### Replacer แบบ Function

```javascript
const data = {
  username: "somchai",
  password: "mypassword",
  token: "abc123token",
  email: "somchai@example.com",
  age: 25,
  salary: 50000
};

// ซ่อน sensitive fields
const safeJSON = JSON.stringify(data, (key, value) => {
  const sensitiveFields = ["password", "token", "ssn"];
  if (sensitiveFields.includes(key)) {
    return undefined; // ไม่รวม field นี้
  }
  return value;
});
console.log(safeJSON);
// '{"username":"somchai","email":"somchai@example.com","age":25,"salary":50000}'

// แปลงค่าระหว่าง stringify
const product = {
  name: "สินค้า A",
  price: 100.5,
  discount: 0.1,
  stock: 50
};

const productJSON = JSON.stringify(product, (key, value) => {
  if (key === "price") return Math.round(value * 100) / 100; // round to 2 decimal
  if (key === "discount") return `${value * 100}%`; // แปลงเป็น percentage
  return value;
});
console.log(productJSON);
// '{"name":"สินค้า A","price":100.5,"discount":"10%","stock":50}'
```

### Space Parameter

```javascript
const obj = { name: "สมชาย", age: 25, hobbies: ["อ่านหนังสือ", "ดูหนัง"] };

// ไม่มี formatting
console.log(JSON.stringify(obj));
// '{"name":"สมชาย","age":25,"hobbies":["อ่านหนังสือ","ดูหนัง"]}'

// indent ด้วยตัวเลข (จำนวน spaces)
console.log(JSON.stringify(obj, null, 2));
/*
{
  "name": "สมชาย",
  "age": 25,
  "hobbies": [
    "อ่านหนังสือ",
    "ดูหนัง"
  ]
}
*/

// indent ด้วย tab
console.log(JSON.stringify(obj, null, '\t'));
/*
{
	"name": "สมชาย",
	"age": 25,
	"hobbies": [
		"อ่านหนังสือ",
		"ดูหนัง"
	]
}
*/

// indent ด้วย custom string
console.log(JSON.stringify(obj, null, '---'));
/*
{
---"name": "สมชาย",
---"age": 25,
---"hobbies": [
------"อ่านหนังสือ",
------"ดูหนัง"
---]
}
*/
```

---

## Step 297: JSON.parse() กับ Reviver

Reviver function ให้แปลงค่าระหว่าง parse

```javascript
// แปลง string กลับเป็น Date
const dateJSON = '{"name":"Event","startDate":"2024-06-15T10:00:00.000Z","endDate":"2024-06-16T18:00:00.000Z"}';

const event = JSON.parse(dateJSON, (key, value) => {
  if (key === "startDate" || key === "endDate") {
    return new Date(value);
  }
  return value;
});

console.log(event.startDate instanceof Date); // true
console.log(event.startDate.getFullYear());   // 2024

// แปลงชนิดข้อมูล
const dataJSON = '{"price":"100.50","qty":"5","active":"true","code":"ABC123"}';

const processed = JSON.parse(dataJSON, (key, value) => {
  if (key === "price") return parseFloat(value);
  if (key === "qty") return parseInt(value);
  if (key === "active") return value === "true";
  return value;
});

console.log(processed.price);  // 100.5 (number)
console.log(processed.qty);    // 5 (number)
console.log(processed.active); // true (boolean)
console.log(processed.code);   // "ABC123" (string)

// การ filter ค่า
const users = `[
  {"name": "สมชาย", "age": 25, "active": true},
  {"name": "สมหญิง", "age": 17, "active": false},
  {"name": "สมศักดิ์", "age": 30, "active": true}
]`;

const parsedUsers = JSON.parse(users);
// Reviver ไม่เหมาะกับการ filter array items
// ใช้ filter หลัง parse แทน
const activeAdults = parsedUsers.filter(u => u.active && u.age >= 18);
console.log(activeAdults.length); // 2
```

---

## Step 298: การทำงานกับ Nested JSON

```javascript
// โครงสร้าง JSON ที่ซับซ้อน
const company = {
  name: "บริษัท เทคโนโลยี จำกัด",
  founded: 2010,
  departments: [
    {
      id: "dept-01",
      name: "วิศวกรรมซอฟต์แวร์",
      manager: {
        name: "นายสมชาย ใจดี",
        email: "somchai@tech.co.th"
      },
      employees: [
        { id: "emp-001", name: "สมศักดิ์ มีสุข", role: "Senior Developer" },
        { id: "emp-002", name: "สมปอง ทำดี", role: "Junior Developer" }
      ]
    },
    {
      id: "dept-02",
      name: "การตลาด",
      manager: {
        name: "นางสมหญิง สวยงาม",
        email: "somying@tech.co.th"
      },
      employees: [
        { id: "emp-003", name: "วิชัย ดีมาก", role: "Marketing Manager" }
      ]
    }
  ]
};

// แปลงเป็น JSON string
const companyJSON = JSON.stringify(company, null, 2);

// แปลงกลับและเข้าถึงข้อมูล
const restored = JSON.parse(companyJSON);

// เข้าถึงข้อมูลลึก
console.log(restored.departments[0].manager.name);
// "นายสมชาย ใจดี"

console.log(restored.departments[0].employees[1].name);
// "สมปอง ทำดี"

// ค้นหาพนักงาน
const allEmployees = restored.departments.flatMap(dept => 
  dept.employees.map(emp => ({ ...emp, department: dept.name }))
);
console.log(allEmployees);

// ค้นหา department ที่มีพนักงานเยอะที่สุด
const largestDept = restored.departments.reduce((max, dept) => 
  dept.employees.length > max.employees.length ? dept : max
);
console.log(largestDept.name); // "วิศวกรรมซอฟต์แวร์"
```

### การแก้ไข Nested JSON

```javascript
// Deep update ใน nested structure
function updateNested(obj, path, value) {
  const keys = path.split('.');
  const result = JSON.parse(JSON.stringify(obj)); // deep clone
  
  let current = result;
  for (let i = 0; i < keys.length - 1; i++) {
    current = current[keys[i]];
  }
  current[keys[keys.length - 1]] = value;
  
  return result;
}

const original = {
  user: {
    profile: {
      name: "สมชาย",
      age: 25
    }
  }
};

const updated = updateNested(original, "user.profile.name", "สมหญิง");
console.log(updated.user.profile.name);   // "สมหญิง"
console.log(original.user.profile.name);  // "สมชาย" (unchanged)
```

---

## Step 299: JSON Arrays

```javascript
// Array ของ objects
const products = [
  { id: 1, name: "แล็ปท็อป", price: 25000, category: "electronics" },
  { id: 2, name: "เมาส์", price: 500, category: "electronics" },
  { id: 3, name: "โต๊ะ", price: 5000, category: "furniture" },
  { id: 4, name: "เก้าอี้", price: 3000, category: "furniture" }
];

const productsJSON = JSON.stringify(products, null, 2);
const parsedProducts = JSON.parse(productsJSON);

// Filter
const electronics = parsedProducts.filter(p => p.category === "electronics");
console.log(electronics.length); // 2

// Sort
const byPrice = [...parsedProducts].sort((a, b) => a.price - b.price);
console.log(byPrice[0].name); // "เมาส์"

// Map
const names = parsedProducts.map(p => p.name);
console.log(names); // ["แล็ปท็อป", "เมาส์", "โต๊ะ", "เก้าอี้"]

// Reduce
const totalValue = parsedProducts.reduce((sum, p) => sum + p.price, 0);
console.log(totalValue); // 33500

// Find
const laptop = parsedProducts.find(p => p.name === "แล็ปท็อป");
console.log(laptop.price); // 25000

// Group by category
const grouped = parsedProducts.reduce((acc, product) => {
  const cat = product.category;
  if (!acc[cat]) acc[cat] = [];
  acc[cat].push(product);
  return acc;
}, {});
console.log(grouped.electronics.length); // 2
console.log(grouped.furniture.length);   // 2
```

### Nested Arrays

```javascript
const matrix = [[1, 2, 3], [4, 5, 6], [7, 8, 9]];
const matrixJSON = JSON.stringify(matrix);
const restored = JSON.parse(matrixJSON);

console.log(restored[1][1]); // 5

// Flatten nested arrays (หลัง parse)
const flat = restored.flat();
console.log(flat); // [1, 2, 3, 4, 5, 6, 7, 8, 9]
```

---

## Step 300: Use Cases ทั่วไปของ JSON

### 1. API Response Handling

```javascript
// จำลอง API response
const apiResponse = `{
  "status": "success",
  "code": 200,
  "data": {
    "users": [
      {"id": 1, "name": "สมชาย", "role": "admin"},
      {"id": 2, "name": "สมหญิง", "role": "user"}
    ],
    "pagination": {
      "page": 1,
      "perPage": 10,
      "total": 2
    }
  }
}`;

const response = JSON.parse(apiResponse);

if (response.status === "success") {
  const users = response.data.users;
  console.log(`พบผู้ใช้ทั้งหมด ${users.length} คน`);
  
  const admins = users.filter(u => u.role === "admin");
  console.log(`Admin: ${admins.map(a => a.name).join(', ')}`);
}
```

### 2. Configuration Files

```javascript
// config.json content
const configJSON = `{
  "app": {
    "name": "MyApp",
    "version": "1.0.0",
    "debug": false
  },
  "database": {
    "host": "localhost",
    "port": 5432,
    "name": "mydb"
  },
  "features": {
    "newUI": true,
    "darkMode": false,
    "notifications": true
  }
}`;

const config = JSON.parse(configJSON);

function getConfig(path) {
  return path.split('.').reduce((obj, key) => obj?.[key], config);
}

console.log(getConfig('app.name'));           // "MyApp"
console.log(getConfig('database.port'));      // 5432
console.log(getConfig('features.newUI'));     // true
```

### 3. Data Exchange

```javascript
// ส่งข้อมูลระหว่าง functions
function createOrder(items, customer) {
  const order = {
    id: Date.now(),
    customer,
    items,
    total: items.reduce((sum, item) => sum + (item.price * item.qty), 0),
    status: "pending",
    createdAt: new Date().toISOString()
  };
  return JSON.stringify(order);
}

function processOrder(orderJSON) {
  const order = JSON.parse(orderJSON);
  order.status = "processing";
  order.processedAt = new Date().toISOString();
  return order;
}

const orderJSON = createOrder(
  [
    { id: 1, name: "สินค้า A", price: 100, qty: 2 },
    { id: 2, name: "สินค้า B", price: 50, qty: 3 }
  ],
  { name: "สมชาย", email: "somchai@example.com" }
);

const processed = processOrder(orderJSON);
console.log(processed.status); // "processing"
console.log(processed.total);  // 350
```

---

## Step 301: การจัดการ Error ใน JSON

```javascript
// Error ที่พบบ่อย
const errorExamples = [
  '{"name": "test",}',          // trailing comma
  "{'name': 'test'}",           // single quotes
  '{name: "test"}',             // unquoted key
  '{"name": undefined}',        // undefined value
  '{"val": Infinity}',          // Infinity
  '',                           // empty string
  null,                         // null input
  undefined                     // undefined input
];

errorExamples.forEach((input, i) => {
  try {
    const result = JSON.parse(input);
    console.log(`[${i}] สำเร็จ:`, result);
  } catch (e) {
    console.log(`[${i}] Error: ${e.message}`);
  }
});

// Robust JSON parser
function robustParse(input, defaultValue = null) {
  if (input === null || input === undefined) {
    return defaultValue;
  }
  
  if (typeof input !== 'string') {
    try {
      input = String(input);
    } catch {
      return defaultValue;
    }
  }
  
  const trimmed = input.trim();
  if (!trimmed) return defaultValue;
  
  try {
    return JSON.parse(trimmed);
  } catch (e) {
    console.error('JSON Parse Error:', e.message);
    return defaultValue;
  }
}

console.log(robustParse('{"a":1}'));     // { a: 1 }
console.log(robustParse(null));          // null
console.log(robustParse('{bad}'));       // null
console.log(robustParse('{bad}', {}));   // {}
```

### การจัดการ JSON จาก APIs

```javascript
async function fetchJSON(url) {
  const response = await fetch(url);
  
  // ตรวจสอบ HTTP status
  if (!response.ok) {
    throw new Error(`HTTP error! status: ${response.status}`);
  }
  
  // ตรวจสอบ content type
  const contentType = response.headers.get('content-type');
  if (!contentType || !contentType.includes('application/json')) {
    throw new Error('Response is not JSON');
  }
  
  try {
    return await response.json();
  } catch (e) {
    throw new Error(`Failed to parse JSON: ${e.message}`);
  }
}
```

---

## Step 302: Pretty Printing JSON

```javascript
// Pretty print function
function prettyPrint(data, indent = 2) {
  return JSON.stringify(data, null, indent);
}

const config = {
  server: {
    host: "localhost",
    port: 3000,
    ssl: false
  },
  database: {
    url: "mongodb://localhost:27017",
    name: "myapp",
    options: {
      useNewUrlParser: true,
      poolSize: 5
    }
  },
  features: ["auth", "logging", "cache"]
};

console.log(prettyPrint(config));
/*
{
  "server": {
    "host": "localhost",
    "port": 3000,
    "ssl": false
  },
  "database": {
    "url": "mongodb://localhost:27017",
    "name": "myapp",
    "options": {
      "useNewUrlParser": true,
      "poolSize": 5
    }
  },
  "features": [
    "auth",
    "logging",
    "cache"
  ]
}
*/

// Color output (Node.js)
function colorPrettyPrint(data) {
  const str = JSON.stringify(data, null, 2);
  
  // Replace strings, numbers, booleans, null with colors
  return str
    .replace(/"(.*?)":/g, '\x1b[34m"$1"\x1b[0m:')  // keys - blue
    .replace(/: "(.*?)"/g, ': \x1b[32m"$1"\x1b[0m') // string values - green
    .replace(/: (\d+\.?\d*)/g, ': \x1b[33m$1\x1b[0m') // numbers - yellow
    .replace(/: (true|false)/g, ': \x1b[35m$1\x1b[0m') // booleans - magenta
    .replace(/: null/g, ': \x1b[31mnull\x1b[0m');   // null - red
}
```

---

## Step 303: Deep Cloning กับ JSON

```javascript
// ปัญหาของ shallow copy
const original = {
  name: "สมชาย",
  address: {
    city: "กรุงเทพ",
    country: "ไทย"
  },
  hobbies: ["อ่านหนังสือ", "ดูหนัง"]
};

// Shallow copy - แชร์ reference
const shallow = { ...original };
shallow.address.city = "เชียงใหม่"; // เปลี่ยน original ด้วย!
console.log(original.address.city); // "เชียงใหม่" (เปลี่ยนไป!)

// Deep clone ด้วย JSON
const deepClone = JSON.parse(JSON.stringify(original));
deepClone.address.city = "ภูเก็ต";
console.log(original.address.city); // "เชียงใหม่" (ไม่เปลี่ยน)

// ฟังก์ชัน deep clone
function deepCopy(obj) {
  return JSON.parse(JSON.stringify(obj));
}

// ข้อจำกัดของ JSON deep clone
const complex = {
  date: new Date(),           // จะกลายเป็น string
  func: () => "hello",        // จะหายไป
  undef: undefined,           // จะหายไป
  regex: /pattern/gi,         // จะกลายเป็น {}
  set: new Set([1, 2, 3]),    // จะกลายเป็น {}
  map: new Map([['a', 1]])    // จะกลายเป็น {}
};

const cloned = deepCopy(complex);
console.log(cloned.date);   // string (ไม่ใช่ Date object)
console.log(cloned.func);   // undefined
console.log(cloned.regex);  // {}

// Alternative: structuredClone (modern browsers/Node.js)
const betterClone = structuredClone(complex);
console.log(betterClone.date instanceof Date); // true
console.log(betterClone.set instanceof Set);   // true
```

---

## Step 304: JSON กับ Date Objects

```javascript
// JSON ไม่มี Date type - จะถูกแปลงเป็น ISO string
const eventDate = new Date('2024-06-15T10:00:00');
const event = {
  name: "JavaScript Workshop",
  date: eventDate
};

const json = JSON.stringify(event);
console.log(json);
// '{"name":"JavaScript Workshop","date":"2024-06-15T03:00:00.000Z"}'

const restored = JSON.parse(json);
console.log(typeof restored.date);       // "string"
console.log(restored.date instanceof Date); // false

// วิธีที่ 1: แปลงกลับด้วย reviver
const restoredWithDate = JSON.parse(json, (key, value) => {
  if (key === 'date') return new Date(value);
  return value;
});
console.log(restoredWithDate.date instanceof Date); // true

// วิธีที่ 2: Pattern ทั่วไป - detect date string
function dateReviver(key, value) {
  const datePattern = /^\d{4}-\d{2}-\d{2}T\d{2}:\d{2}:\d{2}/;
  if (typeof value === 'string' && datePattern.test(value)) {
    const date = new Date(value);
    if (!isNaN(date.getTime())) return date;
  }
  return value;
}

const data = `{
  "name": "Meeting",
  "start": "2024-06-15T10:00:00.000Z",
  "end": "2024-06-15T12:00:00.000Z",
  "title": "Team Sync"
}`;

const parsed = JSON.parse(data, dateReviver);
console.log(parsed.start instanceof Date); // true
console.log(parsed.title instanceof Date); // false

// วิธีที่ 3: เก็บ timestamp แทน
const event2 = {
  name: "ประชุม",
  timestamp: Date.now() // เก็บเป็น number
};
const json2 = JSON.stringify(event2);
const restored2 = JSON.parse(json2);
const date2 = new Date(restored2.timestamp);
console.log(date2 instanceof Date); // true
```

---

## Step 305: การดึงและประมวลผล JSON จาก API

```javascript
// ใช้ fetch API (Browser/Node.js 18+)
async function getUsers() {
  try {
    const response = await fetch('https://jsonplaceholder.typicode.com/users');
    
    if (!response.ok) {
      throw new Error(`HTTP Error: ${response.status}`);
    }
    
    const users = await response.json();
    return users;
  } catch (error) {
    console.error('Error fetching users:', error);
    throw error;
  }
}

// ประมวลผล JSON data
async function processUsers() {
  const users = await getUsers();
  
  // สร้าง summary
  const summary = {
    total: users.length,
    byCity: users.reduce((acc, user) => {
      const city = user.address.city;
      acc[city] = (acc[city] || 0) + 1;
      return acc;
    }, {}),
    emails: users.map(u => u.email),
    names: users.map(u => u.name)
  };
  
  return summary;
}

// POST JSON data
async function createUser(userData) {
  const response = await fetch('https://jsonplaceholder.typicode.com/users', {
    method: 'POST',
    headers: {
      'Content-Type': 'application/json',
      'Accept': 'application/json'
    },
    body: JSON.stringify(userData)
  });
  
  if (!response.ok) {
    throw new Error(`Failed to create user: ${response.status}`);
  }
  
  return response.json();
}

// PUT/UPDATE JSON data
async function updateUser(id, updates) {
  const response = await fetch(`https://jsonplaceholder.typicode.com/users/${id}`, {
    method: 'PUT',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify(updates)
  });
  return response.json();
}

// สาธิตการใช้งาน (จะทำงานเมื่อมี network)
// getUsers().then(users => console.log(`Found ${users.length} users`));
```

---

## Step 306: LocalStorage กับ JSON

```javascript
// LocalStorage เก็บได้เฉพาะ string
// ต้องใช้ JSON.stringify/parse

// เก็บ object
function saveToStorage(key, value) {
  try {
    localStorage.setItem(key, JSON.stringify(value));
    return true;
  } catch (e) {
    console.error('Storage Error:', e);
    return false;
  }
}

// อ่าน object
function getFromStorage(key, defaultValue = null) {
  try {
    const item = localStorage.getItem(key);
    return item ? JSON.parse(item) : defaultValue;
  } catch (e) {
    console.error('Parse Error:', e);
    return defaultValue;
  }
}

// ตัวอย่างการใช้งาน
const userSettings = {
  theme: "dark",
  language: "th",
  fontSize: 16,
  notifications: true,
  savedAt: new Date().toISOString()
};

saveToStorage('userSettings', userSettings);
const loaded = getFromStorage('userSettings');
console.log(loaded.theme); // "dark"

// เก็บ Cart
class ShoppingCart {
  constructor() {
    this.items = getFromStorage('cart', []);
  }
  
  addItem(product) {
    const existing = this.items.find(i => i.id === product.id);
    if (existing) {
      existing.qty++;
    } else {
      this.items.push({ ...product, qty: 1 });
    }
    this.save();
  }
  
  removeItem(productId) {
    this.items = this.items.filter(i => i.id !== productId);
    this.save();
  }
  
  getTotal() {
    return this.items.reduce((sum, item) => sum + (item.price * item.qty), 0);
  }
  
  save() {
    saveToStorage('cart', this.items);
  }
  
  clear() {
    this.items = [];
    localStorage.removeItem('cart');
  }
}
```

---

## Step 307: JSON Validation

```javascript
// Schema validation
function validateUserJSON(data) {
  const errors = [];
  
  if (typeof data !== 'object' || data === null) {
    return ['ข้อมูลต้องเป็น object'];
  }
  
  // Required fields
  const required = ['name', 'email', 'age'];
  required.forEach(field => {
    if (!(field in data)) {
      errors.push(`Field '${field}' is required`);
    }
  });
  
  // Type validation
  if (data.name !== undefined && typeof data.name !== 'string') {
    errors.push("'name' must be a string");
  }
  if (data.email !== undefined && typeof data.email !== 'string') {
    errors.push("'email' must be a string");
  }
  if (data.age !== undefined && (typeof data.age !== 'number' || data.age < 0)) {
    errors.push("'age' must be a positive number");
  }
  
  // Email format
  if (data.email && !/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(data.email)) {
    errors.push("'email' format is invalid");
  }
  
  return errors;
}

// Test
const validUser = { name: "สมชาย", email: "somchai@example.com", age: 25 };
const invalidUser = { name: 123, email: "not-an-email" };

console.log(validateUserJSON(validUser));   // []
console.log(validateUserJSON(invalidUser)); // array of errors

// JSON Schema ง่ายๆ
function validateSchema(data, schema) {
  const errors = [];
  
  Object.entries(schema).forEach(([key, rules]) => {
    const value = data[key];
    
    if (rules.required && (value === undefined || value === null)) {
      errors.push(`'${key}' is required`);
      return;
    }
    
    if (value !== undefined && rules.type && typeof value !== rules.type) {
      errors.push(`'${key}' should be type '${rules.type}'`);
    }
    
    if (rules.minLength && typeof value === 'string' && value.length < rules.minLength) {
      errors.push(`'${key}' must be at least ${rules.minLength} characters`);
    }
    
    if (rules.min !== undefined && typeof value === 'number' && value < rules.min) {
      errors.push(`'${key}' must be at least ${rules.min}`);
    }
  });
  
  return errors;
}

const userSchema = {
  name: { required: true, type: 'string', minLength: 2 },
  age: { required: true, type: 'number', min: 0 },
  email: { required: true, type: 'string' }
};

const result = validateSchema({ name: "ก", age: -1, email: "test" }, userSchema);
console.log(result);
// ["'name' must be at least 2 characters", "'age' must be at least 0"]
```

---

## Step 308: JSON Merge และ Transformation

```javascript
// Merge JSON objects
function mergeJSON(...objects) {
  return objects.reduce((result, obj) => {
    return deepMerge(result, obj);
  }, {});
}

function deepMerge(target, source) {
  const result = { ...target };
  
  Object.keys(source).forEach(key => {
    if (source[key] instanceof Object && !Array.isArray(source[key])) {
      result[key] = deepMerge(target[key] || {}, source[key]);
    } else {
      result[key] = source[key];
    }
  });
  
  return result;
}

const defaultConfig = {
  theme: "light",
  lang: "en",
  notifications: { email: true, sms: false },
  display: { sidebar: true, compact: false }
};

const userConfig = {
  theme: "dark",
  notifications: { sms: true }
};

const merged = mergeJSON(defaultConfig, userConfig);
console.log(merged.theme);                    // "dark" (overridden)
console.log(merged.notifications.email);      // true (kept from default)
console.log(merged.notifications.sms);        // true (overridden)
console.log(merged.display.sidebar);          // true (kept from default)

// JSON Transformation
function transformKeys(obj, transformer) {
  if (Array.isArray(obj)) {
    return obj.map(item => transformKeys(item, transformer));
  }
  if (obj && typeof obj === 'object') {
    return Object.fromEntries(
      Object.entries(obj).map(([key, val]) => [
        transformer(key),
        transformKeys(val, transformer)
      ])
    );
  }
  return obj;
}

// snake_case เป็น camelCase
const snakeToCamel = str => 
  str.replace(/_([a-z])/g, (_, letter) => letter.toUpperCase());

const apiData = {
  user_name: "สมชาย",
  first_name: "สมชาย",
  last_name: "ใจดี",
  email_address: "somchai@example.com",
  created_at: "2024-01-01"
};

const camelData = transformKeys(apiData, snakeToCamel);
console.log(camelData.userName);      // "สมชาย"
console.log(camelData.emailAddress);  // "somchai@example.com"
```

---

## Step 309: JSON Stream Processing

```javascript
// Processing large JSON line by line (NDJSON - Newline Delimited JSON)
const ndjson = `{"id":1,"name":"สมชาย","age":25}
{"id":2,"name":"สมหญิง","age":30}
{"id":3,"name":"สมศักดิ์","age":28}`;

function parseNDJSON(text) {
  return text
    .split('\n')
    .filter(line => line.trim())
    .map(line => JSON.parse(line));
}

const users = parseNDJSON(ndjson);
console.log(users.length); // 3
console.log(users[0].name); // "สมชาย"

// Generator สำหรับ process ทีละ record
function* streamNDJSON(text) {
  const lines = text.split('\n');
  for (const line of lines) {
    const trimmed = line.trim();
    if (trimmed) {
      try {
        yield JSON.parse(trimmed);
      } catch (e) {
        console.error('Parse error on line:', trimmed);
      }
    }
  }
}

for (const user of streamNDJSON(ndjson)) {
  console.log(`Processing: ${user.name}`);
}

// Batch processing
function processBatch(items, batchSize = 10) {
  const batches = [];
  for (let i = 0; i < items.length; i += batchSize) {
    batches.push(items.slice(i, i + batchSize));
  }
  return batches;
}

const largeArray = Array.from({ length: 100 }, (_, i) => ({ id: i, value: i * 2 }));
const jsonString = JSON.stringify(largeArray);
const parsed = JSON.parse(jsonString);
const batches = processBatch(parsed, 10);
console.log(`Processed ${batches.length} batches`); // 10
```

---

## Step 310: โปรเจคจบ - JSON Utility Library

```javascript
// JSON Utility Library ที่ครอบคลุม

const JSONUtils = {
  
  // Safe parse ที่มี default value
  safeParse(str, defaultValue = null) {
    try {
      if (str === null || str === undefined) return defaultValue;
      return JSON.parse(str);
    } catch {
      return defaultValue;
    }
  },
  
  // Safe stringify ที่จัดการ circular refs
  safeStringify(obj, indent = 0) {
    const seen = new WeakSet();
    return JSON.stringify(obj, (key, value) => {
      if (typeof value === 'object' && value !== null) {
        if (seen.has(value)) return '[Circular]';
        seen.add(value);
      }
      return value;
    }, indent);
  },
  
  // Deep clone
  deepClone(obj) {
    return JSON.parse(JSON.stringify(obj));
  },
  
  // Flatten nested object
  flatten(obj, prefix = '', separator = '.') {
    return Object.entries(obj).reduce((acc, [key, val]) => {
      const newKey = prefix ? `${prefix}${separator}${key}` : key;
      if (val && typeof val === 'object' && !Array.isArray(val)) {
        Object.assign(acc, this.flatten(val, newKey, separator));
      } else {
        acc[newKey] = val;
      }
      return acc;
    }, {});
  },
  
  // Unflatten object
  unflatten(obj, separator = '.') {
    return Object.entries(obj).reduce((acc, [key, val]) => {
      const keys = key.split(separator);
      let current = acc;
      for (let i = 0; i < keys.length - 1; i++) {
        if (!current[keys[i]]) current[keys[i]] = {};
        current = current[keys[i]];
      }
      current[keys[keys.length - 1]] = val;
      return acc;
    }, {});
  },
  
  // Get value by path
  get(obj, path, defaultValue) {
    const result = path
      .split('.')
      .reduce((curr, key) => curr?.[key], obj);
    return result !== undefined ? result : defaultValue;
  },
  
  // Set value by path
  set(obj, path, value) {
    const cloned = this.deepClone(obj);
    const keys = path.split('.');
    let current = cloned;
    for (let i = 0; i < keys.length - 1; i++) {
      if (!current[keys[i]]) current[keys[i]] = {};
      current = current[keys[i]];
    }
    current[keys[keys.length - 1]] = value;
    return cloned;
  },
  
  // Remove null/undefined values
  compact(obj) {
    if (Array.isArray(obj)) {
      return obj
        .filter(v => v !== null && v !== undefined)
        .map(v => typeof v === 'object' ? this.compact(v) : v);
    }
    return Object.fromEntries(
      Object.entries(obj)
        .filter(([, v]) => v !== null && v !== undefined)
        .map(([k, v]) => [k, typeof v === 'object' ? this.compact(v) : v])
    );
  },
  
  // Pretty print with colors (browser console)
  prettyPrint(obj) {
    console.log(JSON.stringify(obj, null, 2));
  },
  
  // Compare two objects
  equals(a, b) {
    return JSON.stringify(a) === JSON.stringify(b);
  },
  
  // Size in bytes
  size(obj) {
    return new Blob([JSON.stringify(obj)]).size;
  }
};

// ทดสอบ
const testObj = {
  user: {
    name: "สมชาย",
    address: { city: "กรุงเทพ", zip: null },
    scores: [90, 85, null, 95]
  }
};

// Flatten
const flat = JSONUtils.flatten(testObj);
console.log(flat['user.name']);         // "สมชาย"
console.log(flat['user.address.city']); // "กรุงเทพ"

// Get by path
console.log(JSONUtils.get(testObj, 'user.address.city'));   // "กรุงเทพ"
console.log(JSONUtils.get(testObj, 'user.phone', 'N/A'));   // "N/A"

// Set by path
const updated = JSONUtils.set(testObj, 'user.address.city', 'เชียงใหม่');
console.log(updated.user.address.city);   // "เชียงใหม่"
console.log(testObj.user.address.city);   // "กรุงเทพ" (original unchanged)

// Compact (remove nulls)
const compacted = JSONUtils.compact(testObj);
console.log(compacted.user.address.zip);  // undefined (null removed)
```

---

## แบบฝึกหัด (Exercises)

### แบบฝึกหัดที่ 1: JSON Serializer
สร้างฟังก์ชัน `serializeUser(user)` ที่:
- แปลง user object เป็น JSON string
- ซ่อน password, ssn fields
- แปลง Date เป็น ISO string

```javascript
// ตัวอย่าง input:
const user = {
  name: "สมชาย",
  email: "test@example.com",
  password: "secret",
  ssn: "1234567890",
  birthDate: new Date("1990-01-01"),
  age: 34
};

// Expected output (JSON string):
// {"name":"สมชาย","email":"test@example.com","birthDate":"1990-01-01T00:00:00.000Z","age":34}
```

### แบบฝึกหัดที่ 2: Config Manager
สร้าง class `ConfigManager` ที่:
- โหลด/บันทึก config จาก localStorage
- รองรับ default values
- รองรับ nested paths (config.theme.primary)
- มี method: get, set, reset, export

### แบบฝึกหัดที่ 3: JSON Diff
สร้างฟังก์ชัน `jsonDiff(original, updated)` ที่แสดงความแตกต่างระหว่าง objects สองตัว

```javascript
const a = { name: "สมชาย", age: 25, city: "กรุงเทพ" };
const b = { name: "สมหญิง", age: 25, country: "ไทย" };

// Expected output:
// {
//   changed: { name: ["สมชาย", "สมหญิง"] },
//   removed: ["city"],
//   added: ["country"]
// }
```

### แบบฝึกหัดที่ 4: Data Transformer
สร้างฟังก์ชัน `transformAPIData(data)` ที่:
- แปลง snake_case keys เป็น camelCase
- แปลง ISO date strings เป็น Date objects
- กรอง null values ออก
- Flatten nested objects ที่ลึกเกิน 2 levels

### แบบฝึกหัดที่ 5: JSON Cache System
สร้าง `JSONCache` class ที่:
- เก็บ data ใน localStorage
- มี TTL (Time-To-Live) สำหรับแต่ละ entry
- มี method: set, get, has, delete, clear
- ลบ entries ที่หมดอายุอัตโนมัติ

---

## สรุป Part 16

ในบทนี้เราได้เรียนรู้:
- JSON คืออะไรและกฎไวยากรณ์
- ประเภทข้อมูลใน JSON (string, number, boolean, array, object, null)
- `JSON.stringify()` กับ replacer และ space
- `JSON.parse()` กับ reviver
- การทำงานกับ Nested JSON และ Arrays
- Use cases ทั่วไป (API, config, storage)
- การจัดการ Error
- Deep cloning ด้วย JSON
- การทำงานกับ Date objects
- LocalStorage integration

JSON เป็นพื้นฐานสำคัญในการพัฒนา web applications สมัยใหม่ที่นักพัฒนาทุกคนต้องเข้าใจ
