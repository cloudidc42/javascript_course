# Part 22: Destructuring (Steps 411-430)

## บทนำ

Destructuring เป็นฟีเจอร์ใน ES6 ที่ช่วยให้เราแยกค่าจาก arrays หรือ properties จาก objects มาใส่ในตัวแปรต่างๆ ได้อย่างสะดวกในบรรทัดเดียว ทำให้โค้ดกระชับและอ่านง่ายขึ้นมาก

---

## Step 411: Array Destructuring พื้นฐาน

```javascript
// แบบเก่า - ไม่มี destructuring
const colors = ['แดง', 'เขียว', 'น้ำเงิน'];
const red = colors[0];
const green = colors[1];
const blue = colors[2];

// แบบใหม่ - ใช้ destructuring
const [first, second, third] = colors;
console.log(first);  // 'แดง'
console.log(second); // 'เขียว'
console.log(third);  // 'น้ำเงิน'

// ตัวอย่างเพิ่มเติม
const [a, b, c] = [1, 2, 3];
console.log(a, b, c); // 1 2 3

// destructure function return value
function getCoordinates() {
  return [13.7563, 100.5018]; // lat, lng ของกรุงเทพ
}

const [latitude, longitude] = getCoordinates();
console.log(`Lat: ${latitude}, Lng: ${longitude}`);
// 'Lat: 13.7563, Lng: 100.5018'
```

```javascript
// Destructuring กับ string
const [firstChar, secondChar, ...restChars] = 'สวัสดี';
console.log(firstChar);  // 'ส'
console.log(secondChar); // 'ว'
console.log(restChars);  // ['า', 'ส', 'ด', 'ี']

// Destructuring กับ iterable
const [x, y] = new Set([10, 20, 30]);
console.log(x, y); // 10 20

// Destructuring กับ Map
const map = new Map([['a', 1], ['b', 2]]);
const [[key1, val1], [key2, val2]] = map;
console.log(key1, val1); // 'a' 1
console.log(key2, val2); // 'b' 2
```

---

## Step 412: Skipping Elements ในการ Destructure

```javascript
// ข้ามธาตุด้วยเครื่องหมายจุลภาค
const [, second, , fourth] = [1, 2, 3, 4, 5];
console.log(second); // 2
console.log(fourth); // 4

// ข้ามหลายๆ ตัว
const scores = [95, 87, 72, 68, 55];
const [, , third] = scores;
console.log(third); // 72

// ใช้จริงๆ: เอาแค่บางส่วนของ function result
function getMinMax(arr) {
  return [Math.min(...arr), Math.max(...arr), arr.length];
}

const [min, , count] = getMinMax([3, 1, 4, 1, 5, 9, 2, 6]);
console.log(`Min: ${min}, Count: ${count}`); // 'Min: 1, Count: 8'

// ข้ามตัวแรก
const [, ...withoutFirst] = [1, 2, 3, 4, 5];
console.log(withoutFirst); // [2, 3, 4, 5]
```

---

## Step 413: Default Values ใน Destructuring

```javascript
// Default values - ใช้เมื่อค่าเป็น undefined
const [p = 10, q = 20, r = 30] = [1, 2];
console.log(p); // 1 (มีค่า)
console.log(q); // 2 (มีค่า)
console.log(r); // 30 (ใช้ default เพราะ undefined)

// Default values กับ null
const [s = 'default'] = [null];
console.log(s); // null (null ไม่ trigger default)

const [t = 'default'] = [undefined];
console.log(t); // 'default' (undefined trigger default)

// Default values เป็น expression
const [timestamp = Date.now()] = [];
console.log(timestamp); // เวลาปัจจุบัน

// Default values เป็น function call
function getDefault() { return 'ค่าเริ่มต้น'; }
const [value = getDefault()] = [];
console.log(value); // 'ค่าเริ่มต้น'
```

```javascript
// Default values ใน nested destructuring
const matrix = [[1, 2], [3]];
const [[a, b], [c, d = 4]] = matrix;
console.log(a, b, c, d); // 1 2 3 4

// ใช้งานจริง: config options
function parseConfig(config = {}) {
  const [host = 'localhost', port = 3000] = config.server || [];
  return { host, port };
}

console.log(parseConfig({ server: ['example.com'] }));
// { host: 'example.com', port: 3000 }
```

---

## Step 414: Swapping Variables ด้วย Destructuring

```javascript
// วิธีแลกค่าตัวแปรแบบเก่า
let x = 1, y = 2;
let temp = x;
x = y;
y = temp;
console.log(x, y); // 2 1

// วิธีใหม่ด้วย destructuring - ไม่ต้องใช้ temp!
let a = 1, b = 2;
[a, b] = [b, a];
console.log(a, b); // 2 1

// Swap หลายค่าพร้อมกัน
let first = 'A', second = 'B', third = 'C';
[first, second, third] = [third, first, second];
console.log(first, second, third); // C A B

// ใช้ใน sorting algorithm
function bubbleSort(arr) {
  const result = [...arr];
  for (let i = 0; i < result.length; i++) {
    for (let j = 0; j < result.length - i - 1; j++) {
      if (result[j] > result[j + 1]) {
        [result[j], result[j + 1]] = [result[j + 1], result[j]]; // swap!
      }
    }
  }
  return result;
}

console.log(bubbleSort([5, 3, 8, 1, 9, 2])); // [1, 2, 3, 5, 8, 9]
```

---

## Step 415: Rest Pattern ใน Array Destructuring

```javascript
// Rest pattern - เก็บที่เหลือไว้ใน array
const [head, ...tail] = [1, 2, 3, 4, 5];
console.log(head); // 1
console.log(tail); // [2, 3, 4, 5]

// เอาตัวแรกออก
const [removed, ...kept] = ['ลบ', 'เก็บ1', 'เก็บ2', 'เก็บ3'];
console.log(removed); // 'ลบ'
console.log(kept);    // ['เก็บ1', 'เก็บ2', 'เก็บ3']

// Rest ต้องอยู่ท้ายสุดเสมอ
// const [...first, last] = [1, 2, 3]; // SyntaxError!

// ใช้ rest กับ function
function processArgs(required1, required2, ...optional) {
  console.log(required1, required2);
  console.log('Optional args:', optional);
}

processArgs('a', 'b', 'c', 'd', 'e');
// 'a' 'b'
// 'Optional args:' ['c', 'd', 'e']
```

```javascript
// Rest ใช้กับการแยก header/body
function parseCSV(csvString) {
  const lines = csvString.split('\n');
  const [header, ...rows] = lines;
  const columns = header.split(',');
  
  return rows.map(row => {
    const values = row.split(',');
    return Object.fromEntries(
      columns.map((col, i) => [col.trim(), values[i]?.trim()])
    );
  });
}

const csv = `name,age,city
สมชาย,25,กรุงเทพ
สมหญิง,30,เชียงใหม่
สมศรี,28,ภูเก็ต`;

console.log(parseCSV(csv));
// [
//   { name: 'สมชาย', age: '25', city: 'กรุงเทพ' },
//   { name: 'สมหญิง', age: '30', city: 'เชียงใหม่' },
//   { name: 'สมศรี', age: '28', city: 'ภูเก็ต' }
// ]
```

---

## Step 416: Nested Array Destructuring

```javascript
// Matrix (2D array)
const matrix = [
  [1, 2, 3],
  [4, 5, 6],
  [7, 8, 9]
];

const [[a, b], [, e], [, , i]] = matrix;
console.log(a, b, e, i); // 1 2 5 9

// ดึง diagonal
const [[top], [, mid], [, , bottom]] = matrix;
console.log(top, mid, bottom); // 1 5 9

// Nested กับ default values
const nested = [[1], [2, 3]];
const [[x, y = 10], [z, w = 20]] = nested;
console.log(x, y, z, w); // 1 10 2 3
```

```javascript
// ใช้จริง: coordinates
const route = [
  [13.7563, 100.5018], // กรุงเทพ
  [18.7884, 98.9853],  // เชียงใหม่
  [7.8804, 98.3923],   // ภูเก็ต
];

const [[bkkLat, bkkLng], [cnxLat, cnxLng]] = route;
console.log(`กรุงเทพ: ${bkkLat}, ${bkkLng}`);
// 'กรุงเทพ: 13.7563, 100.5018'

// Nested arrays ใน response
const apiResponse = {
  data: [[1, 2], [3, 4], [5, 6]]
};

const { data: [[r1c1, r1c2]] } = apiResponse;
console.log(r1c1, r1c2); // 1 2
```

---

## Step 417: Object Destructuring พื้นฐาน

```javascript
// แบบเก่า
const person = { name: 'สมชาย', age: 25, city: 'กรุงเทพ' };
const name = person.name;
const age = person.age;
const city = person.city;

// แบบใหม่
const { name: personName, age: personAge, city: personCity } = person;
// หรือถ้าชื่อตัวแปรตรงกับ key:
const { name, age, city } = person;

console.log(name); // 'สมชาย'
console.log(age);  // 25
console.log(city); // 'กรุงเทพ'

// Destructure เฉพาะบาง properties
const { name: justName } = { name: 'สมชาย', age: 25, extra: 'data' };
console.log(justName); // 'สมชาย'

// จาก function return
function getPerson() {
  return { firstName: 'สมชาย', lastName: 'ใจดี', age: 25 };
}

const { firstName, lastName } = getPerson();
console.log(`${firstName} ${lastName}`); // 'สมชาย ใจดี'
```

---

## Step 418: Renaming Variables ใน Object Destructuring

```javascript
// เปลี่ยนชื่อตัวแปรขณะ destructure
const user = { firstName: 'สมชาย', lastName: 'ใจดี' };

// { originalKey: newVariableName }
const { firstName: fn, lastName: ln } = user;
console.log(fn); // 'สมชาย'
console.log(ln); // 'ใจดี'

// เหตุผลที่ต้องเปลี่ยนชื่อ: ชื่อซ้ำกัน
const { firstName: userFirst } = { firstName: 'A' };
const { firstName: adminFirst } = { firstName: 'B' };
console.log(userFirst, adminFirst); // 'A' 'B'

// เปลี่ยนชื่อพร้อม default value
const config = { timeout: 5000 };
const { timeout: requestTimeout = 3000, retries: maxRetries = 3 } = config;
console.log(requestTimeout); // 5000 (มีค่า)
console.log(maxRetries);     // 3 (ใช้ default)
```

```javascript
// ใช้งานจริง: API response mapping
const apiData = {
  user_name: 'john_doe',
  user_email: 'john@example.com',
  created_at: '2024-01-01',
};

// แปลง snake_case เป็น camelCase ขณะ destructure
const {
  user_name: username,
  user_email: email,
  created_at: createdAt
} = apiData;

console.log(username);  // 'john_doe'
console.log(email);     // 'john@example.com'
console.log(createdAt); // '2024-01-01'
```

---

## Step 419: Default Values ใน Object Destructuring

```javascript
// Default values สำหรับ properties ที่ไม่มี
const { x = 0, y = 0, z = 0 } = { x: 10, y: 20 };
console.log(x, y, z); // 10 20 0

// Default values กับการ rename
const { width: w = 100, height: h = 100 } = { width: 200 };
console.log(w, h); // 200 100

// Default values เป็น object หรือ array
const { options = {}, tags = [] } = { tags: ['js', 'es6'] };
console.log(options); // {}
console.log(tags);    // ['js', 'es6']

// Default values เป็น function
const { name, validator = (v) => true } = { name: 'test' };
console.log(validator('anything')); // true
```

```javascript
// ใช้งานจริง: function options pattern
function createServer({
  host = 'localhost',
  port = 3000,
  ssl = false,
  timeout = 30000,
  maxConnections = 100,
} = {}) {
  return {
    url: `${ssl ? 'https' : 'http'}://${host}:${port}`,
    timeout,
    maxConnections,
  };
}

console.log(createServer());
// { url: 'http://localhost:3000', timeout: 30000, maxConnections: 100 }

console.log(createServer({ port: 8080, ssl: true }));
// { url: 'https://localhost:8080', timeout: 30000, maxConnections: 100 }
```

---

## Step 420: Nested Object Destructuring

```javascript
// Object ซ้อน object
const company = {
  name: 'TechCo',
  address: {
    street: '123 ถนนสุขุมวิท',
    city: 'กรุงเทพ',
    country: 'Thailand',
  },
  founder: {
    name: 'สมชาย',
    age: 40,
  },
};

const {
  name: companyName,
  address: { city, country },
  founder: { name: founderName }
} = company;

console.log(companyName);  // 'TechCo'
console.log(city);         // 'กรุงเทพ'
console.log(country);      // 'Thailand'
console.log(founderName);  // 'สมชาย'
```

```javascript
// Nested กับ default values
const config = {
  database: {
    host: 'localhost',
  }
};

const {
  database: {
    host = 'localhost',
    port = 5432,
    name: dbName = 'mydb',
  } = {},
  cache: {
    ttl = 300,
  } = {}
} = config;

console.log(host, port, dbName); // 'localhost' 5432 'mydb'
console.log(ttl); // 300

// ข้อควรระวัง: ค่าที่ destructure มา คือค่า ไม่ใช่ reference
const original = { a: { b: 1 } };
const { a } = original;
a.b = 999; // นี่คือ mutation ของ original!
console.log(original.a.b); // 999
```

---

## Step 421: Mixed Nested Destructuring

```javascript
// ผสม array และ object
const data = {
  users: [
    { id: 1, name: 'สมชาย', scores: [85, 90, 78] },
    { id: 2, name: 'สมหญิง', scores: [92, 88, 95] },
  ]
};

const { users: [firstUser, secondUser] } = data;
console.log(firstUser.name);  // 'สมชาย'
console.log(secondUser.name); // 'สมหญิง'

// ลึกกว่านั้น
const { users: [{ name: user1Name, scores: [score1] }] } = data;
console.log(user1Name); // 'สมชาย'
console.log(score1);    // 85
```

```javascript
// ตัวอย่างจริง: Redux state
const state = {
  auth: {
    user: { id: 1, name: 'สมชาย' },
    token: 'abc123',
    permissions: ['read', 'write'],
  },
  ui: {
    theme: 'dark',
    language: 'th',
  }
};

const {
  auth: {
    user: { name: userName },
    token,
    permissions: [firstPermission],
  },
  ui: { theme }
} = state;

console.log(userName);         // 'สมชาย'
console.log(token);            // 'abc123'
console.log(firstPermission);  // 'read'
console.log(theme);            // 'dark'
```

```javascript
// Array of objects
const products = [
  { id: 1, name: 'แล็ปท็อป', price: 35000, specs: { ram: 16, storage: 512 } },
  { id: 2, name: 'สมาร์ทโฟน', price: 18000, specs: { ram: 8, storage: 256 } },
];

const [
  { name: laptop, price: laptopPrice, specs: { ram: laptopRam } },
  { name: phone, specs: { storage: phoneStorage } }
] = products;

console.log(laptop, laptopPrice, laptopRam); // 'แล็ปท็อป' 35000 16
console.log(phone, phoneStorage); // 'สมาร์ทโฟน' 256
```

---

## Step 422: Function Parameter Destructuring

```javascript
// Object destructuring ใน parameters
function displayUser({ name, age, city = 'ไม่ระบุ' }) {
  console.log(`${name}, ${age} ปี, ${city}`);
}

displayUser({ name: 'สมชาย', age: 25, city: 'กรุงเทพ' });
// 'สมชาย, 25 ปี, กรุงเทพ'

displayUser({ name: 'สมหญิง', age: 30 });
// 'สมหญิง, 30 ปี, ไม่ระบุ'

// Array destructuring ใน parameters
function sum([a, b, c = 0]) {
  return a + b + c;
}

console.log(sum([1, 2, 3])); // 6
console.log(sum([1, 2]));    // 3
```

```javascript
// ใช้กับ nested object
function renderCard({
  title,
  description = 'ไม่มีคำอธิบาย',
  image: { url, alt = 'รูปภาพ' } = {},
  tags = [],
}) {
  return `
    <div class="card">
      <img src="${url || '/default.jpg'}" alt="${alt}" />
      <h2>${title}</h2>
      <p>${description}</p>
      <div class="tags">${tags.join(', ')}</div>
    </div>
  `;
}

const card = renderCard({
  title: 'สวัสดีโลก',
  image: { url: '/hello.jpg' },
  tags: ['javascript', 'es6'],
});
console.log(card);
```

```javascript
// Arrow function กับ parameter destructuring
const formatAddress = ({ street, city, country }) =>
  `${street}, ${city}, ${country}`;

const getFullName = ({ firstName, lastName, title = '' }) =>
  `${title} ${firstName} ${lastName}`.trim();

console.log(formatAddress({
  street: '123 สุขุมวิท',
  city: 'กรุงเทพ',
  country: 'Thailand'
}));
// '123 สุขุมวิท, กรุงเทพ, Thailand'

console.log(getFullName({ firstName: 'สมชาย', lastName: 'ใจดี', title: 'นาย' }));
// 'นาย สมชาย ใจดี'
```

---

## Step 423: Destructuring กับ Loops

```javascript
// for...of กับ array destructuring
const points = [[1, 2], [3, 4], [5, 6]];

for (const [x, y] of points) {
  console.log(`(${x}, ${y})`);
}
// (1, 2)
// (3, 4)
// (5, 6)

// for...of กับ Object.entries()
const scores = { สมชาย: 85, สมหญิง: 92, สมศรี: 78 };

for (const [name, score] of Object.entries(scores)) {
  console.log(`${name}: ${score}`);
}
// สมชาย: 85
// สมหญิง: 92
// สมศรี: 78
```

```javascript
// Array.forEach กับ destructuring
const users = [
  { id: 1, name: 'สมชาย', role: 'admin' },
  { id: 2, name: 'สมหญิง', role: 'user' },
  { id: 3, name: 'สมศรี', role: 'moderator' },
];

users.forEach(({ id, name, role }) => {
  console.log(`[${id}] ${name} (${role})`);
});
// [1] สมชาย (admin)
// [2] สมหญิง (user)
// [3] สมศรี (moderator)

// Array.map กับ destructuring
const userNames = users.map(({ name }) => name);
console.log(userNames); // ['สมชาย', 'สมหญิง', 'สมศรี']

// Array.filter กับ destructuring
const admins = users.filter(({ role }) => role === 'admin');
console.log(admins); // [{ id: 1, name: 'สมชาย', role: 'admin' }]
```

```javascript
// entries() กับ destructuring
const fruits = ['แอปเปิล', 'กล้วย', 'ส้ม'];

for (const [index, fruit] of fruits.entries()) {
  console.log(`${index + 1}. ${fruit}`);
}
// 1. แอปเปิล
// 2. กล้วย
// 3. ส้ม

// Map iteration
const inventory = new Map([
  ['แอปเปิล', 50],
  ['กล้วย', 30],
  ['ส้ม', 20],
]);

for (const [item, quantity] of inventory) {
  console.log(`${item}: ${quantity} ชิ้น`);
}
// แอปเปิล: 50 ชิ้น
// กล้วย: 30 ชิ้น
// ส้ม: 20 ชิ้น
```

---

## Step 424: Destructuring กับ Map และ Set

```javascript
// Destructuring จาก Map
const userMap = new Map([
  ['name', 'สมชาย'],
  ['age', 25],
  ['city', 'กรุงเทพ'],
]);

// ใช้ Object.fromEntries แล้ว destructure
const { name, age } = Object.fromEntries(userMap);
console.log(name, age); // 'สมชาย' 25

// Destructure map entries โดยตรง
const [[firstKey, firstValue], [secondKey, secondValue]] = userMap;
console.log(firstKey, firstValue);   // 'name' 'สมชาย'
console.log(secondKey, secondValue); // 'age' 25
```

```javascript
// Destructuring กับ Set
const numberSet = new Set([10, 20, 30, 40, 50]);

// Set เป็น iterable จึง destructure ด้วย array syntax
const [first, second, ...others] = numberSet;
console.log(first, second); // 10 20
console.log(others);        // [30, 40, 50]

// ใช้ Set เพื่อ unique values
const tags = new Set(['js', 'es6', 'js', 'typescript', 'es6']);
const [tag1, tag2, tag3] = tags;
console.log(tag1, tag2, tag3); // 'js' 'es6' 'typescript'
```

---

## Step 425: Dynamic Property Name (Computed Keys) ใน Destructuring

```javascript
// ใช้ [] สำหรับ computed property names
const propName = 'name';
const { [propName]: value } = { name: 'สมชาย', age: 25 };
console.log(value); // 'สมชาย'

// ใช้กับตัวแปร
function getProperty(obj, key) {
  const { [key]: result } = obj;
  return result;
}

const user = { name: 'สมชาย', age: 25, city: 'กรุงเทพ' };
console.log(getProperty(user, 'name')); // 'สมชาย'
console.log(getProperty(user, 'age'));  // 25

// ต้อง rename เสมอเมื่อใช้ computed keys
const key = 'dynamic';
// const { [key] } = obj; // SyntaxError! ต้อง rename
const { [key]: dynamicValue } = { dynamic: 'ค่าไดนามิค' };
console.log(dynamicValue); // 'ค่าไดนามิค'
```

```javascript
// ใช้งานจริง: multilingual config
function translate(key, lang = 'th') {
  const translations = {
    hello: { th: 'สวัสดี', en: 'Hello', ja: 'こんにちは' },
    goodbye: { th: 'ลาก่อน', en: 'Goodbye', ja: 'さようなら' },
  };
  
  const { [lang]: text = key } = translations[key] || {};
  return text;
}

console.log(translate('hello'));       // 'สวัสดี'
console.log(translate('hello', 'en')); // 'Hello'
console.log(translate('hello', 'ja')); // 'こんにちは'
console.log(translate('unknown'));     // 'unknown'
```

---

## Step 426: Rest Pattern ใน Object Destructuring

```javascript
// Rest กับ object destructuring
const { a, b, ...rest } = { a: 1, b: 2, c: 3, d: 4, e: 5 };
console.log(a, b); // 1 2
console.log(rest); // { c: 3, d: 4, e: 5 }

// ใช้เพื่อ exclude properties
function omit(obj, ...keysToOmit) {
  const result = { ...obj };
  keysToOmit.forEach(key => delete result[key]);
  return result;
}

// หรือด้วย destructuring (เฉพาะ static keys)
const { password, token, ...safeUser } = {
  id: 1,
  name: 'สมชาย',
  password: 'secret',
  token: 'abc123',
};
console.log(safeUser); // { id: 1, name: 'สมชาย' }
```

```javascript
// Rest ใน function parameters
function updateUser({ id, ...updates }) {
  console.log(`Updating user ${id} with:`, updates);
  return { id, ...updates };
}

updateUser({ id: 1, name: 'สมชาย', age: 26, city: 'เชียงใหม่' });
// 'Updating user 1 with:' { name: 'สมชาย', age: 26, city: 'เชียงใหม่' }

// สร้าง pick function
function pick(obj, ...keys) {
  return Object.fromEntries(
    keys.map(key => [key, obj[key]]).filter(([, v]) => v !== undefined)
  );
}

const full = { a: 1, b: 2, c: 3, d: 4 };
console.log(pick(full, 'a', 'c')); // { a: 1, c: 3 }
```

---

## Step 427: Real-world Use Cases - API Responses

```javascript
// จัดการ API response
async function fetchUserProfile(userId) {
  // Simulate API response
  const response = {
    data: {
      user: {
        id: userId,
        personal: {
          firstName: 'สมชาย',
          lastName: 'ใจดี',
          birthDate: '1995-05-15',
        },
        contact: {
          email: 'somchai@example.com',
          phone: '0812345678',
          address: {
            street: '123 สุขุมวิท',
            city: 'กรุงเทพ',
            zipCode: '10110',
          }
        },
        preferences: {
          language: 'th',
          notifications: {
            email: true,
            sms: false,
          }
        }
      }
    },
    meta: {
      requestId: 'req-123',
      timestamp: '2024-01-01T00:00:00Z',
    }
  };
  
  const {
    data: {
      user: {
        id,
        personal: { firstName, lastName },
        contact: {
          email,
          address: { city }
        },
        preferences: {
          notifications: { email: emailNotif }
        }
      }
    },
    meta: { requestId }
  } = response;
  
  return {
    id,
    fullName: `${firstName} ${lastName}`,
    email,
    city,
    emailNotif,
    requestId,
  };
}

fetchUserProfile(1).then(console.log);
// {
//   id: 1,
//   fullName: 'สมชาย ใจดี',
//   email: 'somchai@example.com',
//   city: 'กรุงเทพ',
//   emailNotif: true,
//   requestId: 'req-123'
// }
```

---

## Step 428: Real-world Use Cases - React/Component Patterns

```javascript
// Component props destructuring (React-style)
function UserCard({ 
  user: { name, avatar = '/default-avatar.png', role },
  onEdit,
  onDelete,
  isActive = true,
}) {
  const roleLabel = {
    admin: 'ผู้ดูแลระบบ',
    user: 'ผู้ใช้ทั่วไป',
    moderator: 'ผู้ดูแลเนื้อหา',
  }[role] || 'ไม่ระบุ';
  
  return {
    type: 'div',
    props: { className: `card ${isActive ? 'active' : 'inactive'}` },
    children: [
      { type: 'img', props: { src: avatar, alt: name } },
      { type: 'h3', props: { children: name } },
      { type: 'span', props: { children: roleLabel } },
    ],
  };
}

// useState-style hook pattern
function useState(initialValue) {
  let value = initialValue;
  const get = () => value;
  const set = (newValue) => { value = newValue; };
  return [get, set];
}

const [count, setCount] = useState(0);
setCount(5);
console.log(count()); // 5

// Reducer pattern
function userReducer(state, action) {
  const { type, payload } = action;
  
  switch (type) {
    case 'UPDATE_NAME': {
      const { name } = payload;
      return { ...state, name };
    }
    case 'UPDATE_PROFILE': {
      const { city, phone } = payload;
      return { ...state, contact: { ...state.contact, city, phone } };
    }
    default:
      return state;
  }
}

const initialState = { name: 'สมชาย', contact: { city: 'กรุงเทพ' } };
const newState = userReducer(initialState, {
  type: 'UPDATE_NAME',
  payload: { name: 'สมหญิง' }
});
console.log(newState); // { name: 'สมหญิง', contact: { city: 'กรุงเทพ' } }
```

---

## Step 429: Real-world Use Cases - Data Transformation

```javascript
// Transform data with destructuring
const rawData = [
  { user_id: 1, user_name: 'john', user_email: 'john@test.com', is_active: 1 },
  { user_id: 2, user_name: 'jane', user_email: 'jane@test.com', is_active: 0 },
  { user_id: 3, user_name: 'bob', user_email: 'bob@test.com', is_active: 1 },
];

const transformedData = rawData.map(({
  user_id: id,
  user_name: name,
  user_email: email,
  is_active: active,
}) => ({
  id,
  name,
  email,
  isActive: Boolean(active),
}));

console.log(transformedData);
// [
//   { id: 1, name: 'john', email: 'john@test.com', isActive: true },
//   { id: 2, name: 'jane', email: 'jane@test.com', isActive: false },
//   { id: 3, name: 'bob', email: 'bob@test.com', isActive: true },
// ]
```

```javascript
// กลุ่ม data ด้วย reduce + destructuring
const orders = [
  { id: 1, customer: 'สมชาย', product: 'แล็ปท็อป', amount: 35000 },
  { id: 2, customer: 'สมหญิง', product: 'สมาร์ทโฟน', amount: 18000 },
  { id: 3, customer: 'สมชาย', product: 'เมาส์', amount: 500 },
  { id: 4, customer: 'สมศรี', product: 'คีย์บอร์ด', amount: 1500 },
  { id: 5, customer: 'สมหญิง', product: 'หูฟัง', amount: 2500 },
];

const ordersByCustomer = orders.reduce((groups, { customer, ...order }) => {
  if (!groups[customer]) {
    groups[customer] = [];
  }
  groups[customer].push(order);
  return groups;
}, {});

console.log(ordersByCustomer);
// {
//   สมชาย: [{ id:1, product:'แล็ปท็อป', amount:35000 }, ...],
//   สมหญิง: [...],
//   สมศรี: [...]
// }

// สรุปยอดสั่งซื้อต่อลูกค้า
const totals = Object.entries(ordersByCustomer).map(([customer, orders]) => ({
  customer,
  orderCount: orders.length,
  totalAmount: orders.reduce((sum, { amount }) => sum + amount, 0),
}));

console.log(totals);
// [
//   { customer: 'สมชาย', orderCount: 2, totalAmount: 35500 },
//   { customer: 'สมหญิง', orderCount: 2, totalAmount: 20500 },
//   { customer: 'สมศรี', orderCount: 1, totalAmount: 1500 },
// ]
```

---

## Step 430: สรุปและ Best Practices

```javascript
// Do's and Don'ts

// ✓ DO: ใช้ destructuring ใน function parameters
function good({ name, age }) {
  return `${name}: ${age}`;
}

// ✗ DON'T: access properties ซ้ำๆ
function bad(user) {
  return `${user.name}: ${user.age}`;
}

// ✓ DO: ใช้ default values ป้องกัน undefined
const { count = 0, list = [] } = data;

// ✓ DO: rename เพื่อหลีกเลี่ยงการชน
const { id: userId } = user;
const { id: postId } = post;

// ✓ DO: ใช้ rest pattern แทน manual copy
function cleanUser({ password, token, ...safeFields }) {
  return safeFields;
}

// ✗ DON'T: ทำ nested destructuring ลึกเกินไป
// ยากอ่านมาก:
const { a: { b: { c: { d: { e } } } } } = deepObject;

// ✓ ควรแบ่งเป็นหลายขั้น:
const { a } = deepObject;
const { b } = a;
const { c: { d: { e: deepValue } } } = b;
```

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: Array Destructuring

```javascript
// Task: แปลงคะแนนนักเรียนจาก array เป็น object
// Input: [['สมชาย', 85, 90, 78], ['สมหญิง', 92, 88, 95]]
// Output: [{ name: 'สมชาย', math: 85, science: 90, english: 78, avg: 84.33 }, ...]

function transformScores(studentData) {
  return studentData.map(([name, math, science, english]) => ({
    name,
    math,
    science,
    english,
    avg: ((math + science + english) / 3).toFixed(2),
  }));
}

const data = [
  ['สมชาย', 85, 90, 78],
  ['สมหญิง', 92, 88, 95],
  ['สมศรี', 70, 75, 82],
];

console.log(transformScores(data));
```

### แบบฝึกหัดที่ 2: Object Destructuring กับ API

```javascript
// Task: สร้าง function ที่ parse GitHub API response
// และ return ข้อมูลที่จำเป็น

function parseGitHubUser(apiResponse) {
  const {
    login: username,
    name: displayName,
    public_repos: repoCount,
    followers,
    following,
    company,
    location,
    bio = 'ไม่มีคำอธิบาย',
    created_at: joinedAt,
  } = apiResponse;
  
  return {
    username,
    displayName: displayName || username,
    repoCount,
    social: { followers, following },
    info: { company, location, bio },
    joinedAt: new Date(joinedAt).getFullYear(),
  };
}

const mockGitHubUser = {
  login: 'somchai',
  name: 'สมชาย ใจดี',
  public_repos: 42,
  followers: 150,
  following: 30,
  company: 'TechCo',
  location: 'กรุงเทพ',
  bio: null,
  created_at: '2018-03-15T10:00:00Z',
};

console.log(parseGitHubUser(mockGitHubUser));
```

### แบบฝึกหัดที่ 3: สร้าง Utility Functions

```javascript
// สร้าง utility functions ด้วย destructuring

// 1. groupBy
function groupBy(array, key) {
  return array.reduce((groups, item) => {
    const { [key]: groupKey, ...rest } = item;
    if (!groups[groupKey]) groups[groupKey] = [];
    groups[groupKey].push({ ...rest });
    return groups;
  }, {});
}

const people = [
  { name: 'A', department: 'Engineering' },
  { name: 'B', department: 'Design' },
  { name: 'C', department: 'Engineering' },
];

console.log(groupBy(people, 'department'));
// {
//   Engineering: [{ name: 'A' }, { name: 'C' }],
//   Design: [{ name: 'B' }]
// }

// 2. zipArrays
function zipArrays(...arrays) {
  const maxLength = Math.max(...arrays.map(arr => arr.length));
  return Array.from({ length: maxLength }, (_, i) =>
    arrays.map(arr => arr[i])
  );
}

const keys = ['name', 'age', 'city'];
const values = ['สมชาย', 25, 'กรุงเทพ'];
const zipped = zipArrays(keys, values);
console.log(Object.fromEntries(zipped));
// { name: 'สมชาย', age: 25, city: 'กรุงเทพ' }

// 3. pick and omit
const pick = (obj, keys) =>
  Object.fromEntries(keys.map(k => [k, obj[k]]));

const omit = (obj, keys) => {
  const { ...copy } = obj;
  keys.forEach(k => delete copy[k]);
  return copy;
};

const user = { id: 1, name: 'สมชาย', password: 'secret', token: 'xyz' };
console.log(pick(user, ['id', 'name'])); // { id: 1, name: 'สมชาย' }
console.log(omit(user, ['password', 'token'])); // { id: 1, name: 'สมชาย' }
```

---

## สรุปบทนี้

ในบทนี้เราได้เรียนรู้:

1. **Array Destructuring** - แยกค่าจาก array ลงตัวแปร
2. **Skipping Elements** - ข้ามธาตุที่ไม่ต้องการ
3. **Default Values** - กำหนดค่าเมื่อ undefined
4. **Swap Variables** - แลกค่าตัวแปรโดยไม่ต้อง temp
5. **Rest Pattern** - เก็บส่วนที่เหลือ
6. **Nested Destructuring** - destructure ลึกหลายชั้น
7. **Object Destructuring** - แยก properties จาก object
8. **Renaming** - เปลี่ยนชื่อตัวแปรขณะ destructure
9. **Function Parameter Destructuring** - ทำให้ function API ชัดเจน
10. **Loop Destructuring** - ใช้ใน for...of และ forEach

**หลักการสำคัญ:**
- Destructuring ทำให้โค้ดกระชับและอ่านง่ายขึ้น
- ใช้ default values เพื่อป้องกัน undefined errors
- อย่า nested destructuring ลึกเกินไป - แบ่งเป็นหลายขั้นดีกว่า
- Rest pattern มีประโยชน์มากสำหรับ filtering properties
