# ตอนที่ 7: Objects (ออบเจกต์) ใน JavaScript

## บทนำ

Object คือโครงสร้างข้อมูลที่สำคัญที่สุดใน JavaScript เกือบทุกอย่างใน JavaScript ล้วนเป็น Object ไม่ว่าจะเป็น Arrays, Functions, หรือแม้แต่ Date

Object ใช้เก็บข้อมูลในรูปแบบ key-value pairs เหมาะสำหรับการแทนค่าสิ่งของในโลกความเป็นจริง เช่น คน, สินค้า, หรือการตั้งค่า

ในบทนี้เราจะครอบคลุม Steps 111-130

---

## Step 111: Object Literals

```javascript
// Object literal คือการสร้าง object ด้วย {}
const person = {
  name: 'สมชาย',
  age: 25,
  city: 'กรุงเทพ'
};

console.log(person);  // { name: 'สมชาย', age: 25, city: 'กรุงเทพ' }

// Object ว่าง
const empty = {};

// Object ที่มีข้อมูลหลากหลายชนิด
const product = {
  id: 1,
  name: 'เสื้อยืด',
  price: 299,
  inStock: true,
  tags: ['เสื้อผ้า', 'ลำลอง'],
  dimensions: { width: 50, height: 70 },
  getLabel: function() {
    return `${this.name} - ${this.price} บาท`;
  }
};
```

```javascript
// ตัวอย่าง object ต่างๆ
const student = {
  id: 'S001',
  name: 'มานะ รักเรียน',
  subjects: ['คณิต', 'วิทย์', 'ภาษาไทย'],
  gpa: 3.75,
  active: true
};

const config = {
  theme: 'dark',
  language: 'th',
  fontSize: 16,
  notifications: {
    email: true,
    sms: false,
    push: true
  }
};

const coordinates = {
  lat: 13.7563,
  lng: 100.5018,
  name: 'กรุงเทพมหานคร'
};
```

---

## Step 112: การเข้าถึง Properties

### Dot Notation

```javascript
const person = {
  name: 'สมชาย',
  age: 25,
  address: {
    street: 'ถนนสุขุมวิท',
    city: 'กรุงเทพ'
  }
};

// เข้าถึงด้วย dot
console.log(person.name);              // 'สมชาย'
console.log(person.age);               // 25
console.log(person.address.city);      // 'กรุงเทพ'

// property ที่ไม่มี
console.log(person.email);             // undefined
console.log(person.address.country);   // undefined
```

### Bracket Notation

```javascript
const person = {
  name: 'สมชาย',
  'first name': 'สมชาย',  // key ที่มีช่องว่าง
  age: 25
};

// เข้าถึงด้วย bracket
console.log(person['name']);       // 'สมชาย'
console.log(person['first name']); // 'สมชาย' (ต้องใช้ bracket!)
console.log(person['age']);        // 25

// dynamic key (key เป็นตัวแปร)
const key = 'name';
console.log(person[key]);  // 'สมชาย'

// ใช้ใน loop
const keys = ['name', 'age'];
keys.forEach(k => console.log(`${k}: ${person[k]}`));
```

```javascript
// เปรียบเทียบ dot vs bracket
const obj = { a: 1, b: 2, c: 3 };

// dot notation (ใช้บ่อยกว่า แต่ต้องรู้ key ล่วงหน้า)
console.log(obj.a);

// bracket notation (ยืดหยุ่นกว่า สำหรับ dynamic key)
function getProperty(obj, propName) {
  return obj[propName];  // ต้องใช้ bracket ไม่สามารถใช้ dot ได้
}
console.log(getProperty(obj, 'b'));  // 2
```

---

## Step 113: เพิ่ม แก้ไข ลบ Properties

### เพิ่ม Property

```javascript
const person = { name: 'สมชาย' };

// เพิ่มด้วย dot
person.age = 25;
person.email = 'somchai@test.com';

// เพิ่มด้วย bracket
person['phone'] = '081-234-5678';
person['home address'] = 'กรุงเทพ';

console.log(person);
```

### แก้ไข Property

```javascript
const product = {
  name: 'เสื้อ',
  price: 299,
  stock: 10
};

product.price = 349;     // ปรับราคา
product.stock -= 3;      // ลดสต็อก
product.name = 'เสื้อยืด Premium';  // เปลี่ยนชื่อ

console.log(product);
```

### ลบ Property

```javascript
const user = {
  name: 'สมชาย',
  password: 'secret123',
  email: 'somchai@test.com',
  age: 25
};

// ลบด้วย delete
delete user.password;
delete user['age'];

console.log(user);  // { name: 'สมชาย', email: 'somchai@test.com' }
console.log(user.password);  // undefined

// ตรวจสอบว่า property มีอยู่
console.log('password' in user);  // false
console.log('name' in user);      // true
```

```javascript
// ตรวจสอบ property ด้วยวิธีต่างๆ
const obj = { a: 1, b: undefined, c: null };

console.log('a' in obj);             // true
console.log('b' in obj);             // true (มี property แต่ค่าเป็น undefined)
console.log('d' in obj);             // false

console.log(obj.b !== undefined);    // false (แต่ property มีอยู่!)
console.log(obj.hasOwnProperty('a')); // true
console.log(obj.hasOwnProperty('d')); // false
```

---

## Step 114: Object Methods

```javascript
// Method คือ function ที่เป็น property ของ object
const calculator = {
  value: 0,
  
  add: function(n) {
    this.value += n;
    return this;  // คืน this สำหรับ chaining
  },
  
  subtract: function(n) {
    this.value -= n;
    return this;
  },
  
  multiply: function(n) {
    this.value *= n;
    return this;
  },
  
  getResult: function() {
    return this.value;
  },
  
  reset: function() {
    this.value = 0;
    return this;
  }
};

// Method chaining
const result = calculator
  .reset()
  .add(10)
  .multiply(3)
  .subtract(5)
  .getResult();

console.log(result);  // 25
```

```javascript
// Shorthand method (ES6)
const greetings = {
  language: 'Thai',
  
  // รูปแบบเก่า
  hello: function() {
    return 'สวัสดี';
  },
  
  // รูปแบบใหม่ (shorthand)
  bye() {
    return 'ลาก่อน';
  },
  
  welcome(name) {
    return `ยินดีต้อนรับ ${name}`;
  }
};

console.log(greetings.hello());          // สวัสดี
console.log(greetings.bye());            // ลาก่อน
console.log(greetings.welcome('สมชาย')); // ยินดีต้อนรับ สมชาย
```

---

## Step 115: this Keyword

`this` ใน method ชี้ไปที่ object ที่ method นั้นสังกัดอยู่

```javascript
const person = {
  firstName: 'สมชาย',
  lastName: 'ใจดี',
  age: 25,
  
  getFullName() {
    return `${this.firstName} ${this.lastName}`;
  },
  
  greet() {
    return `สวัสดี! ฉันชื่อ ${this.getFullName()} อายุ ${this.age} ปี`;
  },
  
  birthday() {
    this.age += 1;
    return `สุขสันต์วันเกิด! ตอนนี้อายุ ${this.age} ปีแล้ว`;
  }
};

console.log(person.getFullName());  // สมชาย ใจดี
console.log(person.greet());
console.log(person.birthday());
console.log(person.age);  // 26
```

```javascript
// this ในบริบทต่างๆ
const obj = {
  name: 'MyObject',
  
  regularFunction: function() {
    console.log('regular:', this.name);  // 'MyObject'
  },
  
  arrowFunction: () => {
    // Arrow function ไม่มี this ของตัวเอง!
    console.log('arrow:', this);  // undefined หรือ global
  },
  
  withTimeout() {
    // ปัญหา: this ใน callback
    setTimeout(function() {
      console.log(this.name);  // undefined! (this เปลี่ยน)
    }, 100);
    
    // แก้ด้วย arrow function
    setTimeout(() => {
      console.log(this.name);  // 'MyObject' (this ถูก)
    }, 100);
  }
};

obj.regularFunction();
```

```javascript
// bind, call, apply
function introduce(greeting, punctuation) {
  return `${greeting} ฉันชื่อ ${this.name}${punctuation}`;
}

const person1 = { name: 'สมชาย' };
const person2 = { name: 'สมหญิง' };

// call - เรียกทันที ส่ง this และ arguments ทีละตัว
console.log(introduce.call(person1, 'สวัสดี', '!'));
// 'สวัสดี ฉันชื่อ สมชาย!'

// apply - เรียกทันที ส่ง arguments เป็น array
console.log(introduce.apply(person2, ['ยินดีที่รู้จัก', '.']));
// 'ยินดีที่รู้จัก ฉันชื่อ สมหญิง.'

// bind - สร้าง function ใหม่ที่ผูก this
const introduceSomchai = introduce.bind(person1);
console.log(introduceSomchai('สวัสดีครับ', '~'));
// 'สวัสดีครับ ฉันชื่อ สมชาย~'
```

---

## Step 116: Object Destructuring

```javascript
// รูปแบบพื้นฐาน
const person = {
  name: 'สมชาย',
  age: 25,
  city: 'กรุงเทพ'
};

const { name, age, city } = person;
console.log(name, age, city);  // สมชาย 25 กรุงเทพ

// เปลี่ยนชื่อตัวแปร
const { name: personName, age: personAge } = person;
console.log(personName, personAge);  // สมชาย 25

// ค่า default
const { name: n, email = 'ไม่มี', phone = '000' } = person;
console.log(n, email, phone);  // สมชาย ไม่มี 000
```

```javascript
// Nested destructuring
const user = {
  id: 1,
  name: 'สมชาย',
  address: {
    street: 'สุขุมวิท 1',
    city: 'กรุงเทพ',
    zip: '10110'
  },
  contact: {
    email: 'somchai@test.com',
    phone: '081-234-5678'
  }
};

const {
  name: userName,
  address: { city, zip },
  contact: { email }
} = user;

console.log(userName, city, zip, email);
// สมชาย กรุงเทพ 10110 somchai@test.com
```

```javascript
// rest ใน object destructuring
const { name: nm, age: ag, ...rest } = person;
console.log(nm, ag);    // สมชาย 25
console.log(rest);      // { city: 'กรุงเทพ' }

// ใน function parameters
function displayUser({ name, age, email = 'N/A' }) {
  console.log(`ชื่อ: ${name}, อายุ: ${age}, อีเมล: ${email}`);
}

displayUser({ name: 'สมชาย', age: 25 });
displayUser({ name: 'สมหญิง', age: 30, email: 'somying@test.com' });
```

```javascript
// Destructuring ใน loop
const users = [
  { id: 1, name: 'สมชาย', role: 'admin' },
  { id: 2, name: 'สมหญิง', role: 'user' },
  { id: 3, name: 'สมศักดิ์', role: 'user' }
];

for (const { id, name, role } of users) {
  console.log(`[${id}] ${name} (${role})`);
}

// ใช้กับ Object.entries
const scores = { math: 90, science: 85, thai: 78 };
for (const [subject, score] of Object.entries(scores)) {
  console.log(`${subject}: ${score}`);
}
```

---

## Step 117: Shorthand Properties และ Methods

```javascript
// Shorthand property (ES6)
const name = 'สมชาย';
const age = 25;
const city = 'กรุงเทพ';

// แบบเก่า
const personOld = { name: name, age: age, city: city };

// แบบใหม่ (shorthand)
const person = { name, age, city };

console.log(person);  // { name: 'สมชาย', age: 25, city: 'กรุงเทพ' }

// ใช้ประโยชน์จาก function
function createUser(name, age, email) {
  return { name, age, email };  // compact!
}

const user = createUser('สมหญิง', 30, 'somying@test.com');
console.log(user);
```

```javascript
// Shorthand method
const mathUtils = {
  // แบบเก่า
  addOld: function(a, b) { return a + b; },
  
  // แบบใหม่ (shorthand method)
  add(a, b) { return a + b; },
  subtract(a, b) { return a - b; },
  multiply(a, b) { return a * b; },
  divide(a, b) {
    if (b === 0) throw new Error('หารด้วยศูนย์ไม่ได้');
    return a / b;
  }
};

console.log(mathUtils.add(5, 3));       // 8
console.log(mathUtils.multiply(4, 7));  // 28
```

```javascript
// Getter และ Setter
const temperature = {
  _celsius: 0,
  
  get celsius() {
    return this._celsius;
  },
  
  set celsius(value) {
    if (value < -273.15) {
      throw new Error('อุณหภูมิต่ำกว่า absolute zero!');
    }
    this._celsius = value;
  },
  
  get fahrenheit() {
    return this._celsius * 9/5 + 32;
  },
  
  set fahrenheit(value) {
    this._celsius = (value - 32) * 5/9;
  },
  
  get kelvin() {
    return this._celsius + 273.15;
  }
};

temperature.celsius = 100;
console.log(temperature.fahrenheit);  // 212
console.log(temperature.kelvin);      // 373.15

temperature.fahrenheit = 32;
console.log(temperature.celsius);     // 0
```

---

## Step 118: Computed Property Names

```javascript
// Computed property names ใช้ [] ใน object literal
const propName = 'name';
const obj = {
  [propName]: 'สมชาย',  // เหมือน obj.name = 'สมชาย'
  [`${propName}Length`]: 4  // nameLengeth: 4
};

console.log(obj.name);        // สมชาย
console.log(obj.nameLength);  // 4
```

```javascript
// ใช้กับ dynamic keys
function createWithKey(key, value) {
  return { [key]: value };
}

console.log(createWithKey('color', 'red'));   // { color: 'red' }
console.log(createWithKey('size', 'large'));  // { size: 'large' }

// สร้าง object จาก array
const keys = ['a', 'b', 'c'];
const values = [1, 2, 3];

const obj2 = keys.reduce((acc, key, i) => ({
  ...acc,
  [key]: values[i]
}), {});

console.log(obj2);  // { a: 1, b: 2, c: 3 }
```

```javascript
// ตัวอย่างจริง: สร้าง validation errors object
function validateForm(data) {
  const errors = {};
  
  if (!data.name) {
    errors['name'] = 'กรุณาระบุชื่อ';
  }
  
  if (!data.email) {
    errors['email'] = 'กรุณาระบุอีเมล';
  } else if (!data.email.includes('@')) {
    errors['email'] = 'อีเมลไม่ถูกต้อง';
  }
  
  if (!data.age || data.age < 0) {
    errors['age'] = 'กรุณาระบุอายุที่ถูกต้อง';
  }
  
  return errors;
}

const formErrors = validateForm({ name: '', email: 'test', age: -1 });
console.log(formErrors);
/*
{
  name: 'กรุณาระบุชื่อ',
  email: 'อีเมลไม่ถูกต้อง',
  age: 'กรุณาระบุอายุที่ถูกต้อง'
}
*/
```

```javascript
// Symbol เป็น computed property
const id = Symbol('id');
const type = Symbol('type');

const user = {
  [id]: 12345,
  [type]: 'admin',
  name: 'สมชาย'
};

console.log(user[id]);    // 12345
console.log(user[type]);  // admin
console.log(user.name);   // สมชาย

// Symbol ไม่ปรากฏใน Object.keys หรือ for...in
console.log(Object.keys(user));  // ['name']
```

---

## Step 119: Object.keys(), Object.values(), Object.entries()

```javascript
const scores = {
  math: 90,
  science: 85,
  thai: 78,
  english: 92
};

// Object.keys() - คืน array ของ keys
const keys = Object.keys(scores);
console.log(keys);  // ['math', 'science', 'thai', 'english']

// Object.values() - คืน array ของ values
const values = Object.values(scores);
console.log(values);  // [90, 85, 78, 92]

// Object.entries() - คืน array ของ [key, value] pairs
const entries = Object.entries(scores);
console.log(entries);
/*
[
  ['math', 90],
  ['science', 85],
  ['thai', 78],
  ['english', 92]
]
*/
```

```javascript
// การใช้งานจริง
const product = {
  name: 'เสื้อยืด',
  price: 299,
  color: 'ขาว',
  size: 'M'
};

// วนซ้ำด้วย for...in
for (const key in product) {
  console.log(`${key}: ${product[key]}`);
}

// หา property จำนวนทั้งหมด
console.log(Object.keys(product).length);  // 4

// คำนวณด้วย values
const prices = { shirt: 299, pants: 499, shoes: 799 };
const total = Object.values(prices).reduce((sum, p) => sum + p, 0);
console.log(`รวม: ${total} บาท`);  // รวม: 1597 บาท

// สร้าง object ใหม่จาก entries
const doubled = Object.fromEntries(
  Object.entries(prices).map(([key, val]) => [key, val * 2])
);
console.log(doubled);  // { shirt: 598, pants: 998, shoes: 1598 }
```

```javascript
// กรอง properties
function filterByValue(obj, predicate) {
  return Object.fromEntries(
    Object.entries(obj).filter(([_, value]) => predicate(value))
  );
}

const inventory = {
  shirt: 5,
  pants: 0,
  shoes: 3,
  hat: 0,
  bag: 8
};

const inStock = filterByValue(inventory, qty => qty > 0);
console.log(inStock);  // { shirt: 5, shoes: 3, bag: 8 }

// แปลง keys
function transformKeys(obj, transform) {
  return Object.fromEntries(
    Object.entries(obj).map(([key, value]) => [transform(key), value])
  );
}

const camelToUpper = transformKeys(
  { firstName: 'สมชาย', lastName: 'ใจดี' },
  key => key.toUpperCase()
);
console.log(camelToUpper);  // { FIRSTNAME: 'สมชาย', LASTNAME: 'ใจดี' }
```

---

## Step 120: Object.assign()

```javascript
// Object.assign(target, ...sources)
// copy properties จาก sources ไป target

const defaults = {
  theme: 'light',
  language: 'th',
  fontSize: 14,
  notifications: true
};

const userSettings = {
  theme: 'dark',
  fontSize: 16
};

// รวม settings (userSettings ทับ defaults)
const settings = Object.assign({}, defaults, userSettings);
console.log(settings);
/*
{
  theme: 'dark',        (ทับ)
  language: 'th',       (จาก defaults)
  fontSize: 16,         (ทับ)
  notifications: true   (จาก defaults)
}
*/

// ต้นฉบับไม่เปลี่ยน
console.log(defaults.theme);  // 'light' ยังเดิม
```

```javascript
// copy object (shallow copy)
const original = { a: 1, b: { c: 2 } };
const copy = Object.assign({}, original);

copy.a = 99;
copy.b.c = 99;  // ระวัง! nested object ยังชี้ที่เดิม

console.log(original.a);    // 1 (ไม่เปลี่ยน)
console.log(original.b.c);  // 99 (เปลี่ยน! เพราะ shallow copy)

// deep copy ด้วย JSON
const deepCopy = JSON.parse(JSON.stringify(original));
```

```javascript
// เพิ่ม properties ลงใน existing object
const user = { name: 'สมชาย', age: 25 };
Object.assign(user, {
  email: 'somchai@test.com',
  phone: '081-234-5678'
});
console.log(user);
// { name: 'สมชาย', age: 25, email: '...', phone: '...' }

// ใช้ใน mixin pattern
const Serializable = {
  serialize() {
    return JSON.stringify(this);
  },
  
  deserialize(json) {
    return JSON.parse(json);
  }
};

const Printable = {
  print() {
    console.log(JSON.stringify(this, null, 2));
  }
};

class Person {
  constructor(name, age) {
    this.name = name;
    this.age = age;
    Object.assign(this, Serializable, Printable);
  }
}

const p = new Person('สมชาย', 25);
p.print();
```

---

## Step 121: Object.freeze() และ Object.seal()

### Object.freeze()

```javascript
// freeze: ไม่สามารถแก้ไข เพิ่ม หรือลบ property
const config = Object.freeze({
  API_URL: 'https://api.example.com',
  VERSION: '1.0.0',
  MAX_RETRY: 3
});

config.API_URL = 'https://evil.com';  // ไม่มีผล (หรือ error ใน strict mode)
config.newProp = 'test';              // ไม่มีผล
delete config.VERSION;                // ไม่มีผล

console.log(config.API_URL);  // ยังคง 'https://api.example.com'

// ตรวจสอบ
console.log(Object.isFrozen(config));  // true
```

```javascript
// freeze เป็น shallow เหมือนกัน!
const obj = Object.freeze({
  a: 1,
  nested: { b: 2 }
});

obj.a = 99;         // ไม่มีผล
obj.nested.b = 99;  // มีผล! (nested ไม่ถูก freeze)

console.log(obj.a);         // 1
console.log(obj.nested.b);  // 99

// deep freeze
function deepFreeze(obj) {
  Object.keys(obj).forEach(key => {
    if (typeof obj[key] === 'object' && obj[key] !== null) {
      deepFreeze(obj[key]);
    }
  });
  return Object.freeze(obj);
}

const frozen = deepFreeze({ a: 1, nested: { b: 2 } });
frozen.nested.b = 99;  // ไม่มีผล
console.log(frozen.nested.b);  // 2
```

### Object.seal()

```javascript
// seal: สามารถแก้ไข properties ที่มีอยู่แล้วได้ แต่ไม่สามารถเพิ่มหรือลบ
const settings = Object.seal({
  theme: 'light',
  language: 'th'
});

settings.theme = 'dark';      // ได้! (แก้ไขได้)
settings.fontSize = 16;       // ไม่มีผล (ไม่สามารถเพิ่ม)
delete settings.theme;        // ไม่มีผล (ไม่สามารถลบ)

console.log(settings.theme);    // 'dark' (เปลี่ยนได้)
console.log(settings.fontSize); // undefined

console.log(Object.isSealed(settings));  // true
```

```javascript
// เปรียบเทียบ freeze vs seal
const frozenObj = Object.freeze({ a: 1 });
const sealedObj = Object.seal({ a: 1 });

// ทั้งคู่: ไม่สามารถเพิ่ม property
frozenObj.b = 2;  // ไม่มีผล
sealedObj.b = 2;  // ไม่มีผล

// ทั้งคู่: ไม่สามารถลบ property
delete frozenObj.a;  // ไม่มีผล
delete sealedObj.a;  // ไม่มีผล

// ต่างกัน: แก้ไข property
frozenObj.a = 99;  // ไม่มีผล (frozen)
sealedObj.a = 99;  // มีผล! (sealed ยังแก้ได้)
```

---

## Step 122: Spread Operator กับ Objects

```javascript
// Spread operator {...obj}
const person = { name: 'สมชาย', age: 25 };

// copy object
const copy = { ...person };
copy.age = 30;
console.log(person.age);  // 25 (ไม่เปลี่ยน)

// รวม objects
const address = { city: 'กรุงเทพ', zip: '10110' };
const fullPerson = { ...person, ...address };
console.log(fullPerson);
// { name: 'สมชาย', age: 25, city: 'กรุงเทพ', zip: '10110' }
```

```javascript
// ทับ property ด้วย spread
const defaults = {
  color: 'white',
  size: 'M',
  material: 'cotton'
};

const custom = {
  color: 'black',
  size: 'L'
};

const product = { ...defaults, ...custom };
console.log(product);
// { color: 'black', size: 'L', material: 'cotton' }

// เพิ่ม/แก้ไข บาง properties
const updatedProduct = { ...product, price: 299, inStock: true };
console.log(updatedProduct);
```

```javascript
// ตัวอย่างจริง: update object โดยไม่แก้ต้นฉบับ (immutable pattern)
const state = {
  user: { name: 'สมชาย', age: 25 },
  cart: [],
  isLoggedIn: false
};

// เปลี่ยน isLoggedIn
const newState1 = { ...state, isLoggedIn: true };

// เปลี่ยน user.age (nested update)
const newState2 = {
  ...state,
  user: { ...state.user, age: 26 }
};

console.log(state.isLoggedIn);  // false (ไม่เปลี่ยน)
console.log(newState1.isLoggedIn);  // true
console.log(newState2.user.age);    // 26
```

```javascript
// ลบ property ด้วย spread + rest
const { password, ...safeUser } = {
  name: 'สมชาย',
  email: 'somchai@test.com',
  password: 'secret123'
};

console.log(safeUser);  // { name: 'สมชาย', email: '...' }

// ฟังก์ชัน sanitize
function sanitize(user, ...removeKeys) {
  const result = { ...user };
  removeKeys.forEach(key => delete result[key]);
  return result;
}

const userData = { name: 'A', password: '123', token: 'abc', age: 25 };
const clean = sanitize(userData, 'password', 'token');
console.log(clean);  // { name: 'A', age: 25 }
```

---

## Step 123: Nested Objects

```javascript
// Nested objects
const company = {
  name: 'Tech Corp',
  founded: 2010,
  address: {
    street: 'ถนนสุขุมวิท',
    city: 'กรุงเทพ',
    country: 'ไทย',
    coordinates: {
      lat: 13.7563,
      lng: 100.5018
    }
  },
  departments: {
    engineering: {
      head: 'สมชาย',
      employees: 50
    },
    marketing: {
      head: 'สมหญิง',
      employees: 20
    }
  }
};

console.log(company.address.city);                           // 'กรุงเทพ'
console.log(company.address.coordinates.lat);                // 13.7563
console.log(company.departments.engineering.head);           // 'สมชาย'
```

```javascript
// Optional chaining (?.) - ป้องกัน error จาก nested access
const user = {
  name: 'สมชาย',
  address: {
    city: 'กรุงเทพ'
  }
};

// ถ้าไม่มี ?. จะ error
// console.log(user.contact.email);  // TypeError!

// ใช้ ?. ป้องกัน
console.log(user.contact?.email);        // undefined (ไม่ error)
console.log(user.address?.city);         // 'กรุงเทพ'
console.log(user.address?.zip?.code);    // undefined (ไม่ error)

// Nullish coalescing (??) กับ optional chaining
const email = user.contact?.email ?? 'ไม่มีอีเมล';
console.log(email);  // 'ไม่มีอีเมล'
```

```javascript
// Deep clone
function deepClone(obj) {
  if (obj === null || typeof obj !== 'object') return obj;
  if (Array.isArray(obj)) return obj.map(deepClone);
  
  return Object.keys(obj).reduce((clone, key) => {
    clone[key] = deepClone(obj[key]);
    return clone;
  }, {});
}

const original = {
  a: 1,
  b: { c: 2, d: [3, 4] },
  e: { f: { g: 5 } }
};

const clone = deepClone(original);
clone.b.c = 99;
clone.b.d.push(5);

console.log(original.b.c);        // 2 (ไม่เปลี่ยน)
console.log(original.b.d.length); // 2 (ไม่เปลี่ยน)
```

```javascript
// Access nested property ด้วย path string
function getNestedValue(obj, path) {
  return path.split('.').reduce((current, key) => {
    return current !== undefined && current !== null ? current[key] : undefined;
  }, obj);
}

const data = {
  user: {
    profile: {
      name: 'สมชาย',
      contact: {
        email: 'somchai@test.com'
      }
    }
  }
};

console.log(getNestedValue(data, 'user.profile.name'));          // 'สมชาย'
console.log(getNestedValue(data, 'user.profile.contact.email')); // 'somchai@test.com'
console.log(getNestedValue(data, 'user.settings.theme'));        // undefined
```

---

## Step 124: Object.create() และ Prototype

```javascript
// Object.create() สร้าง object ใหม่ที่มี prototype ที่กำหนด
const animalProto = {
  speak() {
    return `${this.name} กล่าวว่า ${this.sound}`;
  },
  
  describe() {
    return `${this.name} เป็น ${this.type}`;
  }
};

const dog = Object.create(animalProto);
dog.name = 'บั๊กส์';
dog.sound = 'โฮ่ง';
dog.type = 'สุนัข';

console.log(dog.speak());     // 'บั๊กส์ กล่าวว่า โฮ่ง'
console.log(dog.describe());  // 'บั๊กส์ เป็น สุนัข'
console.log(Object.getPrototypeOf(dog) === animalProto);  // true
```

---

## Step 125: Object.fromEntries()

```javascript
// Object.fromEntries() แปลง entries กลับเป็น object
const entries = [['name', 'สมชาย'], ['age', 25], ['city', 'กรุงเทพ']];
const obj = Object.fromEntries(entries);
console.log(obj);  // { name: 'สมชาย', age: 25, city: 'กรุงเทพ' }

// แปลง Map เป็น object
const map = new Map([['a', 1], ['b', 2], ['c', 3]]);
const fromMap = Object.fromEntries(map);
console.log(fromMap);  // { a: 1, b: 2, c: 3 }

// ใช้กับ Object.entries เพื่อ transform
const prices = { apple: 30, banana: 15, orange: 25 };

const discounted = Object.fromEntries(
  Object.entries(prices).map(([fruit, price]) => [fruit, price * 0.9])
);
console.log(discounted);
// { apple: 27, banana: 13.5, orange: 22.5 }
```

```javascript
// แปลง query string เป็น object
function parseQueryString(queryString) {
  const params = new URLSearchParams(queryString);
  return Object.fromEntries(params);
}

const query = 'name=สมชาย&age=25&city=กรุงเทพ';
const parsed = parseQueryString(query);
console.log(parsed);
// { name: 'สมชาย', age: '25', city: 'กรุงเทพ' }
```

---

## Step 126: Property Descriptors

```javascript
// defineProperty ควบคุม property อย่างละเอียด
const person = {};

Object.defineProperty(person, 'name', {
  value: 'สมชาย',
  writable: false,    // ห้ามแก้ไข
  enumerable: true,   // แสดงใน for...in
  configurable: false // ห้ามลบหรือเปลี่ยน descriptor
});

person.name = 'อื่น';  // ไม่มีผล (ใน strict mode จะ error)
console.log(person.name);  // 'สมชาย'

// ดู descriptor
const descriptor = Object.getOwnPropertyDescriptor(person, 'name');
console.log(descriptor);
/*
{
  value: 'สมชาย',
  writable: false,
  enumerable: true,
  configurable: false
}
*/
```

```javascript
// defineProperties (หลาย properties)
const circle = {};

Object.defineProperties(circle, {
  radius: {
    value: 5,
    writable: true,
    enumerable: true,
    configurable: true
  },
  diameter: {
    get() { return this.radius * 2; },
    enumerable: true
  },
  area: {
    get() { return Math.PI * this.radius ** 2; },
    enumerable: true
  },
  circumference: {
    get() { return 2 * Math.PI * this.radius; },
    enumerable: true
  }
});

console.log(circle.diameter);       // 10
console.log(circle.area.toFixed(2)); // 78.54
circle.radius = 10;
console.log(circle.diameter);       // 20
```

---

## Step 127: Proxy

```javascript
// Proxy ดักจับการใช้งาน object
const handler = {
  get(target, key) {
    console.log(`อ่าน property: ${key}`);
    return key in target ? target[key] : `ไม่มี property ${key}`;
  },
  
  set(target, key, value) {
    console.log(`เขียน property: ${key} = ${value}`);
    if (key === 'age' && typeof value !== 'number') {
      throw new TypeError('age ต้องเป็นตัวเลข');
    }
    target[key] = value;
    return true;
  },
  
  deleteProperty(target, key) {
    console.log(`ลบ property: ${key}`);
    return delete target[key];
  }
};

const person = new Proxy({ name: 'สมชาย', age: 25 }, handler);

console.log(person.name);      // อ่าน property: name → สมชาย
console.log(person.email);     // อ่าน property: email → ไม่มี property email
person.age = 30;               // เขียน property: age = 30
// person.age = 'หลายปี';     // TypeError!
```

---

## Step 128: Prototype Chain และ Inheritance

```javascript
// Prototype chain
const animal = {
  type: 'สัตว์',
  breathe() {
    return `${this.name} หายใจ`;
  }
};

const mammal = Object.create(animal);
mammal.feed = 'นมแม่';
mammal.warmBlooded = true;

const dog = Object.create(mammal);
dog.name = 'บั๊กส์';
dog.bark = function() {
  return `${this.name} เห่า: โฮ่ง!`;
};

console.log(dog.breathe());     // 'บั๊กส์ หายใจ' (จาก animal)
console.log(dog.warmBlooded);   // true (จาก mammal)
console.log(dog.bark());        // 'บั๊กส์ เห่า: โฮ่ง!'

// instanceof และ isPrototypeOf
console.log(mammal.isPrototypeOf(dog));  // true
console.log(animal.isPrototypeOf(dog));  // true
```

---

## Step 129: ตัวอย่างขั้นสูง - Observer Pattern

```javascript
// Observer Pattern ด้วย Object
function createObservable(target) {
  const handlers = {};
  
  const observable = new Proxy(target, {
    set(obj, prop, value) {
      const oldValue = obj[prop];
      obj[prop] = value;
      
      if (handlers[prop]) {
        handlers[prop].forEach(handler => handler(value, oldValue));
      }
      if (handlers['*']) {
        handlers['*'].forEach(handler => handler({ prop, value, oldValue }));
      }
      
      return true;
    }
  });
  
  observable.watch = function(prop, handler) {
    if (!handlers[prop]) handlers[prop] = [];
    handlers[prop].push(handler);
  };
  
  return observable;
}

const state = createObservable({ count: 0, name: 'test' });

state.watch('count', (newVal, oldVal) => {
  console.log(`count เปลี่ยนจาก ${oldVal} เป็น ${newVal}`);
});

state.watch('*', ({ prop, value }) => {
  console.log(`[any] ${prop} = ${value}`);
});

state.count = 5;   // count เปลี่ยนจาก 0 เป็น 5
state.name = 'new'; // [any] name = new
```

---

## Step 130: ตัวอย่างโปรเจกต์จริง - Config Manager

```javascript
class ConfigManager {
  #config = {};
  #defaults = {};
  #validators = {};
  #listeners = {};
  
  constructor(defaults = {}) {
    this.#defaults = deepFreeze(defaults);
    this.#config = { ...defaults };
  }
  
  set(key, value) {
    if (this.#validators[key]) {
      const error = this.#validators[key](value);
      if (error) throw new Error(error);
    }
    
    const oldValue = this.#config[key];
    this.#config[key] = value;
    
    if (this.#listeners[key]) {
      this.#listeners[key].forEach(cb => cb(value, oldValue));
    }
    
    return this;
  }
  
  get(key) {
    return key ? this.#config[key] : { ...this.#config };
  }
  
  reset(key) {
    if (key) {
      this.#config[key] = this.#defaults[key];
    } else {
      this.#config = { ...this.#defaults };
    }
    return this;
  }
  
  addValidator(key, validator) {
    this.#validators[key] = validator;
    return this;
  }
  
  onChange(key, callback) {
    if (!this.#listeners[key]) this.#listeners[key] = [];
    this.#listeners[key].push(callback);
    return this;
  }
  
  toJSON() {
    return JSON.stringify(this.#config, null, 2);
  }
}

function deepFreeze(obj) {
  Object.keys(obj).forEach(key => {
    if (typeof obj[key] === 'object' && obj[key] !== null) {
      deepFreeze(obj[key]);
    }
  });
  return Object.freeze(obj);
}

// ใช้งาน
const config = new ConfigManager({
  theme: 'light',
  fontSize: 14,
  language: 'th'
});

config
  .addValidator('fontSize', size => {
    if (size < 10 || size > 24) return 'Font size ต้องอยู่ระหว่าง 10-24';
  })
  .onChange('theme', (newTheme) => {
    console.log(`เปลี่ยน theme เป็น ${newTheme}`);
  });

config.set('theme', 'dark');    // เปลี่ยน theme เป็น dark
config.set('fontSize', 16);

try {
  config.set('fontSize', 50);   // Error: Font size ต้องอยู่ระหว่าง 10-24
} catch (e) {
  console.error(e.message);
}

console.log(config.get('theme'));    // 'dark'
console.log(config.get('fontSize')); // 16
config.reset('theme');
console.log(config.get('theme'));    // 'light' (reset แล้ว)
```

---

## แบบฝึกหัด (Exercises)

### ระดับง่าย

**แบบฝึกหัด 1:** สร้าง object `bankAccount` ที่มี properties: `owner`, `balance` และ methods: `deposit(amount)`, `withdraw(amount)`, `getBalance()`

**แบบฝึกหัด 2:** เขียน function `mergeObjects(...objects)` ที่รวม objects หลายอันเข้าด้วยกัน

**แบบฝึกหัด 3:** เขียน function `pick(obj, keys)` ที่ดึงเฉพาะ properties ที่ต้องการออกมา

**แบบฝึกหัด 4:** เขียน function `omit(obj, keys)` ที่ลบ properties ที่ไม่ต้องการออก

**แบบฝึกหัด 5:** เขียน function `invert(obj)` ที่สลับ keys และ values

### ระดับกลาง

**แบบฝึกหัด 6:** สร้าง `EventEmitter` class ที่รองรับ `on()`, `off()`, `emit()`

**แบบฝึกหัด 7:** เขียน function `diff(obj1, obj2)` ที่หาความแตกต่างระหว่าง 2 objects

**แบบฝึกหัด 8:** เขียน function `flattenObject(obj, separator='.')` ที่แปลง nested object เป็น flat object

**แบบฝึกหัด 9:** สร้าง `LinkedList` ด้วย objects

### ระดับยาก

**แบบฝึกหัด 10:** สร้าง `Store` ที่ทำงานแบบ Redux (state management)

### เฉลยบางส่วน

```javascript
// แบบฝึกหัด 1
const bankAccount = {
  owner: 'สมชาย',
  balance: 0,
  
  deposit(amount) {
    if (amount <= 0) throw new Error('จำนวนต้องมากกว่า 0');
    this.balance += amount;
    console.log(`ฝากเงิน ${amount} บาท, ยอดคงเหลือ ${this.balance} บาท`);
    return this;
  },
  
  withdraw(amount) {
    if (amount > this.balance) throw new Error('ยอดเงินไม่พอ');
    this.balance -= amount;
    console.log(`ถอนเงิน ${amount} บาท, ยอดคงเหลือ ${this.balance} บาท`);
    return this;
  },
  
  getBalance() { return this.balance; }
};

// แบบฝึกหัด 3
function pick(obj, keys) {
  return keys.reduce((result, key) => {
    if (key in obj) result[key] = obj[key];
    return result;
  }, {});
}

// แบบฝึกหัด 4
function omit(obj, keys) {
  return Object.fromEntries(
    Object.entries(obj).filter(([key]) => !keys.includes(key))
  );
}

// แบบฝึกหัด 5
function invert(obj) {
  return Object.fromEntries(
    Object.entries(obj).map(([key, value]) => [value, key])
  );
}

// แบบฝึกหัด 8
function flattenObject(obj, separator = '.', prefix = '') {
  return Object.keys(obj).reduce((flat, key) => {
    const fullKey = prefix ? `${prefix}${separator}${key}` : key;
    
    if (typeof obj[key] === 'object' && obj[key] !== null && !Array.isArray(obj[key])) {
      Object.assign(flat, flattenObject(obj[key], separator, fullKey));
    } else {
      flat[fullKey] = obj[key];
    }
    
    return flat;
  }, {});
}

const nested = { a: { b: { c: 1 } }, d: { e: 2 }, f: 3 };
console.log(flattenObject(nested));
// { 'a.b.c': 1, 'd.e': 2, f: 3 }
```

---

## สรุป

ในบทนี้เราได้เรียนรู้:
- การสร้าง Object literals
- การเข้าถึง properties ด้วย dot และ bracket notation
- การเพิ่ม แก้ไข และลบ properties
- Object methods และ this keyword
- Object destructuring
- Shorthand properties และ methods
- Computed property names
- Object.keys(), Object.values(), Object.entries()
- Object.assign(), Object.freeze(), Object.seal()
- Spread operator กับ objects
- Nested objects และ optional chaining
- Property descriptors
- Proxy

Objects เป็นรากฐานสำคัญของ JavaScript การเข้าใจ objects อย่างลึกซึ้งจะช่วยให้เขียนโค้ดที่มีคุณภาพและแก้ปัญหาได้อย่างมีประสิทธิภาพ
