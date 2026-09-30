# Part 30: Object Methods ขั้นสูง (Advanced Object Methods)
## ขั้นตอนที่ 571-590

---

## บทนำ

JavaScript objects มี methods และ features ที่ทรงพลังมากมายที่นักพัฒนาส่วนใหญ่ไม่ค่อยได้ใช้ ใน Part นี้เราจะเรียนรู้เกี่ยวกับการควบคุม property descriptors, prototype chain, และ object utilities ต่างๆ อย่างลึกซึ้ง

---

## ขั้นตอนที่ 571: Object.create() และ Prototype-based Inheritance

```javascript
// ตัวอย่างที่ 1: Object.create() พื้นฐาน
const animal = {
  breathe() {
    return `${this.name} กำลังหายใจ`;
  },
  eat(food) {
    return `${this.name} กำลังกิน ${food}`;
  }
};

// สร้าง object ที่มี animal เป็น prototype
const dog = Object.create(animal);
dog.name = "บิ้ง";
dog.bark = function() {
  return `${this.name} เห่า: โฮ่งๆ!`;
};

console.log(dog.breathe()); // "บิ้ง กำลังหายใจ" (สืบทอดจาก animal)
console.log(dog.eat("ข้าว")); // "บิ้ง กำลังกิน ข้าว" (สืบทอดจาก animal)
console.log(dog.bark()); // "บิ้ง เห่า: โฮ่งๆ!" (เมธอดของตัวเอง)

// ตรวจสอบ prototype chain
console.log(Object.getPrototypeOf(dog) === animal); // true
```

```javascript
// ตัวอย่างที่ 2: Object.create() สำหรับ inheritance chain
const vehicle = {
  type: "ยานพาหนะ",
  move() {
    return `${this.name} กำลังเคลื่อนที่`;
  },
  describe() {
    return `${this.name} เป็น ${this.type}`;
  }
};

const car = Object.create(vehicle);
car.type = "รถยนต์";
car.honk = function() {
  return `${this.name}: บีบ!`;
};

const electricCar = Object.create(car);
electricCar.type = "รถไฟฟ้า";
electricCar.charge = function() {
  return `${this.name} กำลังชาร์จไฟ`;
};

const tesla = Object.create(electricCar);
tesla.name = "Tesla Model 3";
tesla.range = 500;

// Prototype chain: tesla -> electricCar -> car -> vehicle -> Object.prototype
console.log(tesla.move());     // สืบทอดจาก vehicle
console.log(tesla.honk());     // สืบทอดจาก car
console.log(tesla.charge());   // สืบทอดจาก electricCar
console.log(tesla.describe()); // "Tesla Model 3 เป็น รถไฟฟ้า"
```

```javascript
// ตัวอย่างที่ 3: Object.create(null) - object ที่ไม่มี prototype
const pureObject = Object.create(null);
pureObject.key = "value";
pureObject.count = 42;

// ไม่มี toString, hasOwnProperty, etc.
console.log(Object.keys(pureObject)); // ["key", "count"]

// ใช้เป็น clean lookup table (ไม่มี prototype pollution)
const lookup = Object.create(null);
lookup["__proto__"] = "ปลอดภัย";  // ไม่ทำให้ prototype chain เสีย
lookup["constructor"] = "ปลอดภัย";
```

---

## ขั้นตอนที่ 572: Object.defineProperty() และ Property Descriptors

```javascript
// ตัวอย่างที่ 4: Property Descriptor
// แต่ละ property มี descriptor ที่กำหนดพฤติกรรม
const obj = {};

Object.defineProperty(obj, "name", {
  value: "สมชาย",    // ค่าของ property
  writable: true,     // แก้ไขได้ไหม
  enumerable: true,   // แสดงใน for...in และ Object.keys() ไหม
  configurable: true  // ลบหรือแก้ descriptor ได้ไหม
});

console.log(obj.name); // "สมชาย"

// ค่าเริ่มต้นสำหรับ defineProperty: writable/enumerable/configurable = false
Object.defineProperty(obj, "id", {
  value: 12345
  // writable: false (ค่าเริ่มต้น)
  // enumerable: false (ค่าเริ่มต้น)
  // configurable: false (ค่าเริ่มต้น)
});

console.log(obj.id); // 12345
obj.id = 99999; // ไม่มีผลเพราะ writable: false
console.log(obj.id); // ยังเป็น 12345
```

```javascript
// ตัวอย่างที่ 5: writable: false - ค่าแก้ไขไม่ได้
const config = {};

Object.defineProperty(config, "API_URL", {
  value: "https://api.example.com",
  writable: false,
  enumerable: true,
  configurable: false
});

config.API_URL = "https://hacker.com"; // ถูกละเว้น (silent fail)
console.log(config.API_URL); // "https://api.example.com"

// ใน strict mode จะ throw TypeError
"use strict";
// config.API_URL = "new"; // TypeError: Cannot assign to read only property
```

```javascript
// ตัวอย่างที่ 6: enumerable: false - ซ่อน property
const user = {
  name: "สมชาย",
  email: "somchai@example.com"
};

// เพิ่ม property ที่ซ่อน
Object.defineProperty(user, "_password", {
  value: "hashed_password_123",
  writable: true,
  enumerable: false,  // ไม่แสดงใน for...in หรือ Object.keys()
  configurable: false
});

console.log("Object.keys:", Object.keys(user));
// ["name", "email"] - _password ไม่แสดง

console.log("for...in:");
for (const key in user) {
  console.log(key); // name, email - _password ไม่แสดง
}

console.log("JSON.stringify:", JSON.stringify(user));
// {"name":"สมชาย","email":"somchai@example.com"}

// แต่ยังเข้าถึงได้โดยตรง
console.log(user._password); // "hashed_password_123"
console.log(Object.getOwnPropertyNames(user)); // รวม _password
```

---

## ขั้นตอนที่ 573: Property Attributes

```javascript
// ตัวอย่างที่ 7: configurable: false - ล็อค property
const immutableObj = {};

Object.defineProperty(immutableObj, "VERSION", {
  value: "1.0.0",
  writable: false,
  enumerable: true,
  configurable: false // ไม่สามารถ redefine หรือ delete ได้
});

// พยายาม delete
delete immutableObj.VERSION; // false (ไม่สำเร็จ)
console.log(immutableObj.VERSION); // ยังอยู่

// พยายาม redefine
try {
  Object.defineProperty(immutableObj, "VERSION", {
    value: "2.0.0" // TypeError!
  });
} catch (e) {
  console.error("ไม่สามารถ redefine:", e.message);
}
```

```javascript
// ตัวอย่างที่ 8: Accessor Properties - getter/setter
const temperature = {};

Object.defineProperty(temperature, "celsius", {
  get() {
    return this._celsius;
  },
  set(value) {
    if (typeof value !== "number") {
      throw new TypeError("อุณหภูมิต้องเป็นตัวเลข");
    }
    if (value < -273.15) {
      throw new RangeError("อุณหภูมิต่ำกว่า absolute zero ไม่ได้");
    }
    this._celsius = value;
  },
  enumerable: true,
  configurable: true
});

Object.defineProperty(temperature, "fahrenheit", {
  get() {
    return this._celsius * 9/5 + 32;
  },
  set(value) {
    this.celsius = (value - 32) * 5/9;
  },
  enumerable: true,
  configurable: true
});

temperature.celsius = 100;
console.log("เซลเซียส:", temperature.celsius);   // 100
console.log("ฟาเรนไฮต์:", temperature.fahrenheit); // 212

temperature.fahrenheit = 32;
console.log("เซลเซียส:", temperature.celsius);   // 0

try {
  temperature.celsius = -300;
} catch (e) {
  console.error(e.message);
}
```

```javascript
// ตัวอย่างที่ 9: getter/setter ใน object literal (ย่อกว่า defineProperty)
const circle = {
  _radius: 5,
  
  get radius() {
    return this._radius;
  },
  
  set radius(value) {
    if (value < 0) throw new RangeError("รัศมีต้องไม่ติดลบ");
    this._radius = value;
  },
  
  get area() {
    return Math.PI * this._radius ** 2;
  },
  
  get circumference() {
    return 2 * Math.PI * this._radius;
  }
};

circle.radius = 10;
console.log(`รัศมี: ${circle.radius}`);
console.log(`พื้นที่: ${circle.area.toFixed(2)}`);
console.log(`เส้นรอบวง: ${circle.circumference.toFixed(2)}`);
```

---

## ขั้นตอนที่ 574: Object.defineProperties()

```javascript
// ตัวอย่างที่ 10: defineProperty หลายๆ properties พร้อมกัน
const Product = function(id, name, price) {
  this._id = id;
  this._name = name;
  this._price = price;
};

Object.defineProperties(Product.prototype, {
  id: {
    get() { return this._id; },
    enumerable: true,
    configurable: false
  },
  name: {
    get() { return this._name; },
    set(value) {
      if (!value || value.trim() === "") {
        throw new Error("ชื่อสินค้าห้ามว่าง");
      }
      this._name = value.trim();
    },
    enumerable: true,
    configurable: false
  },
  price: {
    get() { return this._price; },
    set(value) {
      if (value < 0) throw new RangeError("ราคาต้องไม่ติดลบ");
      this._price = value;
    },
    enumerable: true,
    configurable: false
  },
  formattedPrice: {
    get() { return `฿${this._price.toLocaleString()}`; },
    enumerable: false,
    configurable: false
  },
  priceWithVat: {
    get() { return this._price * 1.07; },
    enumerable: false,
    configurable: false
  }
});

const iphone = new Product(1, "iPhone 15 Pro", 45000);
console.log(`${iphone.name}: ${iphone.formattedPrice}`);
console.log(`ราคารวม VAT: ฿${iphone.priceWithVat.toLocaleString()}`);

iphone.price = 40000;
console.log(`ราคาใหม่: ${iphone.formattedPrice}`);
```

---

## ขั้นตอนที่ 575: Object.getOwnPropertyDescriptor()

```javascript
// ตัวอย่างที่ 11: ตรวจสอบ property descriptor
const person = {
  name: "สมชาย",
  age: 25
};

const nameDescriptor = Object.getOwnPropertyDescriptor(person, "name");
console.log("Descriptor ของ name:");
console.log("  value:", nameDescriptor.value);       // "สมชาย"
console.log("  writable:", nameDescriptor.writable);  // true
console.log("  enumerable:", nameDescriptor.enumerable); // true
console.log("  configurable:", nameDescriptor.configurable); // true
```

```javascript
// ตัวอย่างที่ 12: ดู descriptor ของ built-in properties
const arr = [1, 2, 3];

const lengthDesc = Object.getOwnPropertyDescriptor(arr, "length");
console.log("Array.length descriptor:");
console.log("  value:", lengthDesc.value);       // 3
console.log("  writable:", lengthDesc.writable);  // true (สามารถ truncate ได้)
console.log("  enumerable:", lengthDesc.enumerable); // false
console.log("  configurable:", lengthDesc.configurable); // false
```

```javascript
// ตัวอย่างที่ 13: Object.getOwnPropertyDescriptors() - ดูทั้งหมด
const source = {
  name: "สมชาย",
  get fullName() { return `คุณ${this.name}`; }
};

const descriptors = Object.getOwnPropertyDescriptors(source);
console.log("Descriptors ทั้งหมด:");
Object.entries(descriptors).forEach(([key, desc]) => {
  console.log(`  ${key}:`, desc);
});

// ใช้ clone object พร้อม getters/setters
function deepCloneWithAccessors(obj) {
  return Object.create(
    Object.getPrototypeOf(obj),
    Object.getOwnPropertyDescriptors(obj)
  );
}

const cloned = deepCloneWithAccessors(source);
cloned.name = "ประทีป";
console.log("clone.fullName:", cloned.fullName); // "คุณประทีป"
```

---

## ขั้นตอนที่ 576: Object.getOwnPropertyNames()

```javascript
// ตัวอย่างที่ 14: เปรียบเทียบวิธีดู properties
const obj = {};

Object.defineProperty(obj, "visible", {
  value: 1,
  enumerable: true
});

Object.defineProperty(obj, "hidden", {
  value: 2,
  enumerable: false // ซ่อน
});

// Object.keys() - เฉพาะ enumerable
console.log("Object.keys:", Object.keys(obj));
// ["visible"]

// Object.getOwnPropertyNames() - ทั้ง enumerable และ non-enumerable
console.log("getOwnPropertyNames:", Object.getOwnPropertyNames(obj));
// ["visible", "hidden"]

// for...in - enumerable + inherited
// (ในกรณีนี้ไม่ต่างจาก keys เพราะไม่มี prototype)

// Object.getOwnPropertySymbols() - เฉพาะ Symbol keys
const sym = Symbol("mySymbol");
obj[sym] = "symbol value";
console.log("getOwnPropertySymbols:", Object.getOwnPropertySymbols(obj));
// [Symbol(mySymbol)]

// Reflect.ownKeys() - ทุก keys (string + symbol, enumerable + non-enumerable)
console.log("Reflect.ownKeys:", Reflect.ownKeys(obj));
// ["visible", "hidden", Symbol(mySymbol)]
```

---

## ขั้นตอนที่ 577: Object.getPrototypeOf() และ Object.setPrototypeOf()

```javascript
// ตัวอย่างที่ 15: ตรวจสอบ prototype
const arr = [];
const obj2 = {};
const fn = function() {};

console.log(Object.getPrototypeOf(arr) === Array.prototype); // true
console.log(Object.getPrototypeOf(obj2) === Object.prototype); // true
console.log(Object.getPrototypeOf(fn) === Function.prototype); // true

// Prototype chain
function Person(name) {
  this.name = name;
}

function Employee(name, company) {
  Person.call(this, name);
  this.company = company;
}

Employee.prototype = Object.create(Person.prototype);
Employee.prototype.constructor = Employee;

const emp = new Employee("สมชาย", "บริษัท ABC");

console.log(Object.getPrototypeOf(emp) === Employee.prototype); // true
console.log(Object.getPrototypeOf(Employee.prototype) === Person.prototype); // true
```

```javascript
// ตัวอย่างที่ 16: instanceof vs prototype check
function Animal(name) {
  this.name = name;
}

function Dog(name) {
  Animal.call(this, name);
}
Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog;

const dog2 = new Dog("บิ้ง");

console.log(dog2 instanceof Dog);    // true
console.log(dog2 instanceof Animal); // true (prototype chain)
console.log(dog2 instanceof Object); // true

// isPrototypeOf
console.log(Dog.prototype.isPrototypeOf(dog2));    // true
console.log(Animal.prototype.isPrototypeOf(dog2)); // true
```

```javascript
// ตัวอย่างที่ 17: Object.setPrototypeOf() (หลีกเลี่ยงถ้าทำได้)
// การใช้ setPrototypeOf ช้าและควรใช้ Object.create() แทน
const base = {
  greet() {
    return `สวัสดี, ฉันชื่อ ${this.name}`;
  }
};

const obj3 = { name: "สมชาย" };
Object.setPrototypeOf(obj3, base);

console.log(obj3.greet()); // "สวัสดี, ฉันชื่อ สมชาย"
console.log(Object.getPrototypeOf(obj3) === base); // true
```

---

## ขั้นตอนที่ 578: Object.keys(), values(), entries() - Deep Dive

```javascript
// ตัวอย่างที่ 18: Object.keys() กับ inheritance
function Base() {
  this.own1 = "ของตัวเอง 1";
}
Base.prototype.inherited = "สืบทอดมา";

const instance = new Base();
instance.own2 = "ของตัวเอง 2";

// Object.keys() เฉพาะ own + enumerable
console.log("keys:", Object.keys(instance)); // ["own1", "own2"]

// for...in รวม inherited
console.log("for...in:");
for (const key in instance) {
  console.log(` ${key} (own: ${instance.hasOwnProperty(key)})`);
}
// own1 (own: true)
// own2 (own: true)
// inherited (own: false)
```

```javascript
// ตัวอย่างที่ 19: Object.values() และ Object.entries()
const scores2 = {
  สมชาย: 85,
  สมหญิง: 92,
  ประทีป: 78,
  มาลี: 95
};

// values
const values = Object.values(scores2);
const avg = values.reduce((sum, v) => sum + v, 0) / values.length;
console.log("ค่าเฉลี่ย:", avg.toFixed(2));
console.log("สูงสุด:", Math.max(...values));
console.log("ต่ำสุด:", Math.min(...values));

// entries
const sorted4 = Object.entries(scores2)
  .sort((a, b) => b[1] - a[1])
  .map(([name, score], i) => `${i + 1}. ${name}: ${score}`);

console.log("อันดับ:");
sorted4.forEach(s => console.log("  " + s));
```

```javascript
// ตัวอย่างที่ 20: ใช้ entries กับ Map operations
const config2 = {
  host: "localhost",
  port: 5432,
  database: "mydb",
  username: "admin"
};

// แปลง object เป็น Map
const configMap = new Map(Object.entries(config2));
console.log("host:", configMap.get("host"));

// แปลง Map กลับเป็น object
const backToObj = Object.fromEntries(configMap);
console.log("กลับเป็น object:", backToObj);

// Transform entries
const uppercased = Object.fromEntries(
  Object.entries(config2).map(([key, value]) => [
    key.toUpperCase(),
    value
  ])
);
console.log("uppercase keys:", uppercased);
```

---

## ขั้นตอนที่ 579: Object.fromEntries()

```javascript
// ตัวอย่างที่ 21: Object.fromEntries() พื้นฐาน
// สร้าง object จาก key-value pairs
const entries = [["a", 1], ["b", 2], ["c", 3]];
const obj4 = Object.fromEntries(entries);
console.log(obj4); // { a: 1, b: 2, c: 3 }

// จาก Map
const map2 = new Map([["x", 10], ["y", 20], ["z", 30]]);
const fromMap2 = Object.fromEntries(map2);
console.log(fromMap2); // { x: 10, y: 20, z: 30 }
```

```javascript
// ตัวอย่างที่ 22: fromEntries สำหรับ object transformation
const prices = {
  apple: 50,
  banana: 30,
  orange: 45,
  mango: 80
};

// เพิ่มราคา 10%
const increased = Object.fromEntries(
  Object.entries(prices).map(([fruit, price]) => [fruit, price * 1.1])
);
console.log("ราคาใหม่:", increased);

// กรองเฉพาะราคาต่ำกว่า 50
const cheap = Object.fromEntries(
  Object.entries(prices).filter(([_, price]) => price < 50)
);
console.log("ราคาต่ำกว่า 50:", cheap);
```

```javascript
// ตัวอย่างที่ 23: fromEntries กับ URL params
function parseQueryString(queryString) {
  return Object.fromEntries(new URLSearchParams(queryString));
}

const params = parseQueryString("?name=สมชาย&age=25&city=กรุงเทพ");
console.log("params:", params);

// สร้าง query string จาก object
function buildQueryString(params) {
  return new URLSearchParams(params).toString();
}

const qs = buildQueryString({ search: "iPhone", category: "phone", page: "1" });
console.log("query string:", qs);
```

---

## ขั้นตอนที่ 580: Object.assign() - Deep Dive

```javascript
// ตัวอย่างที่ 24: Object.assign() พื้นฐาน
const target = { a: 1, b: 2 };
const source1 = { b: 3, c: 4 };
const source2 = { c: 5, d: 6 };

// แก้ไข target โดยตรง
const result = Object.assign(target, source1, source2);
console.log("target:", target);  // { a: 1, b: 3, c: 5, d: 6 }
console.log("result === target:", result === target); // true
```

```javascript
// ตัวอย่างที่ 25: clone object ด้วย assign (shallow clone)
const original = {
  name: "สมชาย",
  address: { city: "กรุงเทพ" }
};

const shallow = Object.assign({}, original);
shallow.name = "ประทีป"; // แก้ไม่กระทบ original
shallow.address.city = "เชียงใหม่"; // แก้กระทบ original! (shallow copy)

console.log("original.name:", original.name); // "สมชาย" (ไม่เปลี่ยน)
console.log("original.address.city:", original.address.city); // "เชียงใหม่" (เปลี่ยน!)
```

```javascript
// ตัวอย่างที่ 26: merge objects
const defaults = {
  theme: "light",
  language: "th",
  notifications: true,
  fontSize: 16
};

const userPreferences = {
  theme: "dark",
  fontSize: 18
};

// userPreferences override defaults
const merged = Object.assign({}, defaults, userPreferences);
console.log("merged:", merged);
// { theme: "dark", language: "th", notifications: true, fontSize: 18 }
```

```javascript
// ตัวอย่างที่ 27: assign กับ class instance
class Config {
  constructor(settings) {
    this.debug = false;
    this.timeout = 5000;
    this.retries = 3;
    Object.assign(this, settings); // apply custom settings
  }
}

const config3 = new Config({ debug: true, timeout: 10000 });
console.log("config:", config3);
// Config { debug: true, timeout: 10000, retries: 3 }
```

---

## ขั้นตอนที่ 581: Object.freeze() vs Object.seal() vs Object.preventExtensions()

```javascript
// ตัวอย่างที่ 28: Object.freeze() - ล็อคทุกอย่าง
const frozenConfig = Object.freeze({
  API_URL: "https://api.example.com",
  VERSION: "1.0.0",
  MAX_RETRIES: 3
});

frozenConfig.API_URL = "hacked"; // ไม่มีผล (silent fail)
frozenConfig.newProp = "new";     // ไม่มีผล
delete frozenConfig.VERSION;      // ไม่มีผล

console.log("freeze:", frozenConfig);
// { API_URL: "https://api.example.com", VERSION: "1.0.0", MAX_RETRIES: 3 }

console.log("isFrozen:", Object.isFrozen(frozenConfig)); // true

// freeze เป็น shallow - nested objects ยังแก้ได้
const shallowFrozen = Object.freeze({
  settings: { debug: false }
});
shallowFrozen.settings.debug = true; // ทำได้! settings ไม่ได้ freeze
console.log(shallowFrozen.settings.debug); // true
```

```javascript
// ตัวอย่างที่ 29: Object.seal() - เพิ่ม/ลบ property ไม่ได้ แต่แก้ค่าได้
const sealed = Object.seal({
  name: "สมชาย",
  age: 25
});

sealed.name = "ประทีป"; // ได้ (value แก้ได้)
sealed.email = "test@example.com"; // ไม่ได้ (เพิ่ม property ใหม่ไม่ได้)
delete sealed.age; // ไม่ได้ (ลบไม่ได้)

console.log("seal:", sealed); // { name: "ประทีป", age: 25 }
console.log("isSealed:", Object.isSealed(sealed)); // true
```

```javascript
// ตัวอย่างที่ 30: Object.preventExtensions() - เพิ่ม property ใหม่ไม่ได้
const limited = Object.preventExtensions({
  x: 1,
  y: 2
});

limited.x = 10;  // ได้ (แก้ค่าได้)
limited.z = 30;  // ไม่ได้ (เพิ่มไม่ได้)
delete limited.y; // ได้ (ลบได้)

console.log("preventExtensions:", limited); // { x: 10 }
console.log("isExtensible:", Object.isExtensible(limited)); // false
```

```javascript
// ตัวอย่างที่ 31: Deep freeze
function deepFreeze(obj) {
  // freeze ทุก property ที่เป็น object ก่อน
  Object.getOwnPropertyNames(obj).forEach(name => {
    const value = obj[name];
    if (value && typeof value === "object") {
      deepFreeze(value);
    }
  });
  
  return Object.freeze(obj);
}

const config4 = deepFreeze({
  server: {
    host: "localhost",
    port: 3000
  },
  database: {
    host: "db.example.com",
    port: 5432,
    credentials: {
      username: "admin",
      password: "secret"
    }
  }
});

config4.server.host = "changed"; // ไม่มีผล
config4.database.credentials.password = "hacked"; // ไม่มีผล

console.log("deep frozen host:", config4.server.host); // "localhost"
console.log("deep frozen pass:", config4.database.credentials.password); // "secret"
```

---

## ขั้นตอนที่ 582: Object.is() Comparison

```javascript
// ตัวอย่างที่ 32: Object.is() vs === 
// Object.is() เหมือน === แต่จัดการ NaN และ +0/-0 ต่างกัน

// NaN
console.log(NaN === NaN);           // false (ทำให้สับสน)
console.log(Object.is(NaN, NaN));   // true (ถูกต้อง!)

// +0 และ -0
console.log(+0 === -0);             // true (ทำให้สับสน)
console.log(Object.is(+0, -0));     // false (ถูกต้อง!)
console.log(Object.is(+0, +0));     // true

// ค่าทั่วไป
console.log(Object.is(1, 1));       // true
console.log(Object.is("a", "a"));   // true
console.log(Object.is(null, null)); // true
console.log(Object.is(undefined, undefined)); // true
console.log(Object.is(1, 2));       // false
```

```javascript
// ตัวอย่างที่ 33: ประโยชน์ของ Object.is()
function areEqual(a, b) {
  return Object.is(a, b);
}

// ใช้ในการเปรียบเทียบผลลัพธ์ที่อาจเป็น NaN
function safeDivide(a, b) {
  const result = a / b;
  return Object.is(result, NaN) ? null : result;
}

console.log(safeDivide(10, 2));  // 5
console.log(safeDivide(10, 0));  // null (หารด้วย 0)
```

---

## ขั้นตอนที่ 583: Object.hasOwn()

```javascript
// ตัวอย่างที่ 34: Object.hasOwn() (ES2022)
const obj5 = {
  name: "สมชาย",
  toString: "overridden" // override inherited method
};

// hasOwnProperty ปัญหา: ถูก override ได้
// obj5.hasOwnProperty("name") อาจ error ถ้า hasOwnProperty ถูก override

// Object.hasOwn() ปลอดภัยกว่า
console.log(Object.hasOwn(obj5, "name"));     // true
console.log(Object.hasOwn(obj5, "toString")); // true (own property)
console.log(Object.hasOwn(obj5, "valueOf"));  // false (inherited)

// กับ null prototype object
const nullProto = Object.create(null);
nullProto.key = "value";

// nullProto.hasOwnProperty("key") จะ error! ไม่มี method นี้
console.log(Object.hasOwn(nullProto, "key"));  // true (ปลอดภัย)
```

---

## ขั้นตอนที่ 584: การตรวจสอบ Properties

```javascript
// ตัวอย่างที่ 35: in operator vs hasOwnProperty vs Object.hasOwn
function Vehicle(type) {
  this.type = type;
}
Vehicle.prototype.move = function() {};
Vehicle.prototype.fuel = "gasoline";

const car2 = new Vehicle("sedan");

// in operator: รวม inherited
console.log("type" in car2);     // true (own)
console.log("fuel" in car2);     // true (inherited)
console.log("move" in car2);     // true (inherited method)
console.log("color" in car2);    // false

// hasOwnProperty: เฉพาะ own
console.log(car2.hasOwnProperty("type")); // true
console.log(car2.hasOwnProperty("fuel")); // false

// Object.hasOwn: เฉพาะ own (ES2022)
console.log(Object.hasOwn(car2, "type")); // true
console.log(Object.hasOwn(car2, "fuel")); // false
```

---

## ขั้นตอนที่ 585: การ Iterate Object

```javascript
// ตัวอย่างที่ 36: วิธีต่างๆ ในการ iterate
const person2 = {
  name: "สมชาย",
  age: 25,
  email: "somchai@example.com"
};

// for...in - รวม inherited, enumerable
for (const key in person2) {
  if (Object.hasOwn(person2, key)) { // กรอง inherited
    console.log(`${key}: ${person2[key]}`);
  }
}

// Object.keys
Object.keys(person2).forEach(key => {
  console.log(`${key}: ${person2[key]}`);
});

// Object.entries
for (const [key, value] of Object.entries(person2)) {
  console.log(`${key}: ${value}`);
}

// Object.values
const values2 = Object.values(person2);
console.log("ค่าทั้งหมด:", values2);
```

```javascript
// ตัวอย่างที่ 37: iterate กับ Symbol keys
const id = Symbol("id");
const type = Symbol("type");

const product2 = {
  name: "iPhone",
  price: 35000,
  [id]: "PROD-001",
  [type]: "electronics"
};

// for...in และ Object.keys ไม่เห็น Symbols
console.log("Object.keys:", Object.keys(product2)); // ["name", "price"]

// เข้าถึง Symbol keys
console.log("getOwnPropertySymbols:", Object.getOwnPropertySymbols(product2));
// [Symbol(id), Symbol(type)]

// Reflect.ownKeys รวมทุกอย่าง
console.log("Reflect.ownKeys:", Reflect.ownKeys(product2));
// ["name", "price", Symbol(id), Symbol(type)]

// เข้าถึง Symbol value
console.log("id:", product2[id]);   // "PROD-001"
console.log("type:", product2[type]); // "electronics"
```

---

## ขั้นตอนที่ 586: การแปลงระหว่าง Object และ Map

```javascript
// ตัวอย่างที่ 38: Object vs Map
// Object: keys ต้องเป็น string หรือ Symbol
// Map: keys เป็นอะไรก็ได้

// Object -> Map
const obj6 = { a: 1, b: 2, c: 3 };
const map3 = new Map(Object.entries(obj6));
console.log("Object to Map:", map3);

// Map -> Object
const map4 = new Map([
  ["name", "สมชาย"],
  ["age", 25],
  [Symbol("id"), 123] // Symbol key จะหายไปตอนแปลง
]);

const fromMap3 = Object.fromEntries(map4); // Symbol ถูกละเว้น
console.log("Map to Object:", fromMap3);
```

```javascript
// ตัวอย่างที่ 39: เมื่อควรใช้ Map แทน Object
// ใช้ Map เมื่อ:
// 1. Keys ไม่ใช่ string
// 2. จำเป็นต้องรู้จำนวน entries
// 3. ต้องการ insertion order ที่ถูกต้อง
// 4. เพิ่ม/ลบบ่อย

// Key ที่เป็น object
const keyMap = new Map();
const key1 = { id: 1 };
const key2 = { id: 2 };

keyMap.set(key1, "ค่าสำหรับ key1");
keyMap.set(key2, "ค่าสำหรับ key2");

console.log(keyMap.get(key1)); // "ค่าสำหรับ key1"
console.log(keyMap.size);      // 2

// สิ่งที่ไม่ทำได้กับ Object
const metadata = new Map();
metadata.set(document.body, { clicks: 0, views: 100 }); // DOM element เป็น key
```

---

## ขั้นตอนที่ 587: Deep Cloning Objects

```javascript
// ตัวอย่างที่ 40: Shallow vs Deep clone
const original2 = {
  name: "สมชาย",
  address: {
    city: "กรุงเทพ",
    street: "สุขุมวิท"
  },
  hobbies: ["อ่านหนังสือ", "เล่นกีฬา"]
};

// Shallow clone (spread)
const shallow2 = { ...original2 };
shallow2.name = "ประทีป"; // ไม่กระทบ original
shallow2.address.city = "เชียงใหม่"; // กระทบ original!

console.log("original.address.city:", original2.address.city); // "เชียงใหม่" - เปลี่ยน!
```

```javascript
// ตัวอย่างที่ 41: JSON clone (deep แต่มีข้อจำกัด)
const original3 = {
  name: "สมชาย",
  address: { city: "กรุงเทพ" },
  hobbies: ["อ่านหนังสือ"],
  // ข้อจำกัด: สิ่งต่อไปนี้จะหายหรือเสีย:
  fn: function() { return "hello"; }, // Function จะหาย
  date: new Date(),                   // Date เป็น string
  und: undefined,                     // undefined จะหาย
  regexp: /test/g,                    // RegExp เป็น {}
  sym: Symbol("id"),                  // Symbol จะหาย
  map5: new Map()                      // Map เป็น {}
};

const jsonClone = JSON.parse(JSON.stringify(original3));
console.log("JSON clone:", jsonClone);
// date เป็น string, fn หาย, und หาย, regexp เป็น {}
```

```javascript
// ตัวอย่างที่ 42: structuredClone() - ES2022 (deep clone ที่ดีกว่า)
const obj7 = {
  name: "สมชาย",
  address: { city: "กรุงเทพ" },
  hobbies: ["อ่านหนังสือ", "เล่นกีฬา"],
  date: new Date(),
  map6: new Map([["a", 1]]),
  set: new Set([1, 2, 3]),
  typedArr: new Uint8Array([1, 2, 3])
};

const deepClone = structuredClone(obj7);
deepClone.address.city = "เชียงใหม่"; // ไม่กระทบ original

console.log("original.city:", obj7.address.city); // "กรุงเทพ" - ไม่เปลี่ยน!
console.log("clone.city:", deepClone.address.city); // "เชียงใหม่"
console.log("date instanceof Date:", deepClone.date instanceof Date); // true
console.log("map6 instanceof Map:", deepClone.map6 instanceof Map); // true
```

```javascript
// ตัวอย่างที่ 43: Custom deep clone สำหรับ special cases
function deepCloneCustom(value, seen = new WeakMap()) {
  // Primitives
  if (value === null || typeof value !== "object" && typeof value !== "function") {
    return value;
  }
  
  // Handle circular references
  if (seen.has(value)) return seen.get(value);
  
  // Arrays
  if (Array.isArray(value)) {
    const copy = [];
    seen.set(value, copy);
    value.forEach((item, i) => {
      copy[i] = deepCloneCustom(item, seen);
    });
    return copy;
  }
  
  // Date
  if (value instanceof Date) {
    return new Date(value.getTime());
  }
  
  // RegExp
  if (value instanceof RegExp) {
    return new RegExp(value.source, value.flags);
  }
  
  // Map
  if (value instanceof Map) {
    const copy = new Map();
    seen.set(value, copy);
    value.forEach((v, k) => {
      copy.set(deepCloneCustom(k, seen), deepCloneCustom(v, seen));
    });
    return copy;
  }
  
  // Set
  if (value instanceof Set) {
    const copy = new Set();
    seen.set(value, copy);
    value.forEach(v => copy.add(deepCloneCustom(v, seen)));
    return copy;
  }
  
  // Plain objects
  const copy = Object.create(Object.getPrototypeOf(value));
  seen.set(value, copy);
  
  Reflect.ownKeys(value).forEach(key => {
    copy[key] = deepCloneCustom(value[key], seen);
  });
  
  return copy;
}

// ทดสอบกับ circular reference
const circular = { name: "circular" };
circular.self = circular; // circular reference

const cloned2 = deepCloneCustom(circular);
console.log("cloned.name:", cloned2.name);
console.log("circular handled:", cloned2.self !== circular); // true
```

---

## ขั้นตอนที่ 588-590: แบบฝึกหัด

```javascript
// แบบฝึกหัดที่ 1: สร้าง Observable Object
function createObservable(target) {
  const listeners = new Map();
  
  function notify(prop, oldValue, newValue) {
    const propListeners = listeners.get(prop) || [];
    const anyListeners = listeners.get("*") || [];
    
    [...propListeners, ...anyListeners].forEach(fn => {
      fn({ prop, oldValue, newValue });
    });
  }
  
  const proxy = new Proxy(target, {
    set(obj, prop, value) {
      const oldValue = obj[prop];
      obj[prop] = value;
      
      if (oldValue !== value) {
        notify(prop, oldValue, value);
      }
      
      return true;
    }
  });
  
  return {
    proxy,
    
    watch(prop, callback) {
      if (!listeners.has(prop)) {
        listeners.set(prop, []);
      }
      listeners.get(prop).push(callback);
      
      // Unwatch function
      return () => {
        const list = listeners.get(prop);
        const index = list.indexOf(callback);
        if (index > -1) list.splice(index, 1);
      };
    }
  };
}

// ใช้งาน
const { proxy: userState, watch } = createObservable({
  name: "สมชาย",
  age: 25,
  score: 100
});

const unwatch = watch("score", ({ prop, oldValue, newValue }) => {
  console.log(`${prop} เปลี่ยนจาก ${oldValue} เป็น ${newValue}`);
});

watch("*", ({ prop, oldValue, newValue }) => {
  console.log(`[Any] ${prop}: ${oldValue} -> ${newValue}`);
});

userState.score = 150; // "score เปลี่ยนจาก 100 เป็น 150"
userState.name = "ประทีป"; // "[Any] name: สมชาย -> ประทีป"
unwatch(); // หยุด watch score
userState.score = 200; // ไม่ได้ยิน score แล้ว แต่ยัง watch *
```

```javascript
// แบบฝึกหัดที่ 2: Schema Validation Object
function createValidatedObject(schema) {
  const data = {};
  
  const validated = Object.defineProperties({}, 
    Object.fromEntries(
      Object.entries(schema).map(([key, rules]) => [
        key,
        {
          get() { return data[key]; },
          set(value) {
            // Type checking
            if (rules.type && typeof value !== rules.type) {
              throw new TypeError(`${key} ต้องเป็น ${rules.type}`);
            }
            
            // Required
            if (rules.required && (value === undefined || value === null || value === "")) {
              throw new Error(`${key} ห้ามว่าง`);
            }
            
            // Min/Max for numbers
            if (rules.min !== undefined && value < rules.min) {
              throw new RangeError(`${key} ต้องไม่ต่ำกว่า ${rules.min}`);
            }
            if (rules.max !== undefined && value > rules.max) {
              throw new RangeError(`${key} ต้องไม่เกิน ${rules.max}`);
            }
            
            // Min/Max length for strings
            if (rules.minLength !== undefined && value.length < rules.minLength) {
              throw new RangeError(`${key} ต้องมีอย่างน้อย ${rules.minLength} ตัวอักษร`);
            }
            
            // Pattern
            if (rules.pattern && !rules.pattern.test(value)) {
              throw new Error(`${key} ไม่ตรงรูปแบบที่กำหนด`);
            }
            
            data[key] = value;
          },
          enumerable: true,
          configurable: false
        }
      ])
    )
  );
  
  return validated;
}

// ใช้งาน
const userForm = createValidatedObject({
  name: { type: "string", required: true, minLength: 2 },
  age: { type: "number", min: 18, max: 120 },
  email: { type: "string", pattern: /^[^\s@]+@[^\s@]+\.[^\s@]+$/ }
});

try {
  userForm.name = "สมชาย";
  userForm.age = 25;
  userForm.email = "somchai@example.com";
  console.log("ข้อมูลถูกต้อง:", { ...userForm });
} catch (err) {
  console.error("Validation error:", err.message);
}

try {
  userForm.age = 15; // Error: อายุต้องไม่ต่ำกว่า 18
} catch (err) {
  console.error(err.message);
}

try {
  userForm.email = "invalid-email"; // Error: ไม่ตรงรูปแบบ
} catch (err) {
  console.error(err.message);
}
```

```javascript
// แบบฝึกหัดที่ 3: Proxy-based Namespace Object
function createNamespace(separator = ".") {
  const data = Object.create(null);
  
  return new Proxy(data, {
    get(target, prop) {
      if (prop in target) return target[prop];
      
      // สร้าง nested namespace โดยอัตโนมัติ
      if (typeof prop === "string") {
        const nested = createNamespace(separator);
        target[prop] = nested;
        return nested;
      }
    },
    
    set(target, prop, value) {
      if (prop.includes(separator)) {
        const keys = prop.split(separator);
        let current = target;
        
        keys.slice(0, -1).forEach(key => {
          if (!current[key]) current[key] = Object.create(null);
          current = current[key];
        });
        
        current[keys[keys.length - 1]] = value;
      } else {
        target[prop] = value;
      }
      
      return true;
    }
  });
}

const config5 = createNamespace();

// ตั้งค่าแบบ path
config5["server.host"] = "localhost";
config5["server.port"] = 3000;
config5["db.host"] = "db.example.com";
config5["db.credentials.username"] = "admin";

// เข้าถึง nested
config5.app.name = "My App";
config5.app.version = "1.0.0";

console.log("server.host:", config5["server.host"]);
console.log("app.name:", config5.app.name);
```

```javascript
// แบบฝึกหัดที่ 4: Object Diff
function objectDiff(obj1, obj2) {
  const allKeys = new Set([
    ...Object.keys(obj1),
    ...Object.keys(obj2)
  ]);
  
  const diff = {
    added: {},
    removed: {},
    changed: {},
    unchanged: {}
  };
  
  allKeys.forEach(key => {
    if (!(key in obj1)) {
      diff.added[key] = obj2[key];
    } else if (!(key in obj2)) {
      diff.removed[key] = obj1[key];
    } else if (!Object.is(obj1[key], obj2[key])) {
      diff.changed[key] = { from: obj1[key], to: obj2[key] };
    } else {
      diff.unchanged[key] = obj1[key];
    }
  });
  
  return diff;
}

const before = {
  name: "สมชาย",
  age: 25,
  email: "old@example.com",
  city: "กรุงเทพ"
};

const after = {
  name: "สมชาย",
  age: 26, // เปลี่ยน
  email: "new@example.com", // เปลี่ยน
  phone: "0812345678", // เพิ่มใหม่
  // city หาย
};

const diff = objectDiff(before, after);
console.log("เพิ่ม:", diff.added);
console.log("ลบ:", diff.removed);
console.log("เปลี่ยน:", diff.changed);
console.log("ไม่เปลี่ยน:", diff.unchanged);
```

```javascript
// แบบฝึกหัดที่ 5: Fluent Object Builder
class PersonBuilder {
  constructor() {
    this._data = {};
  }
  
  static create() {
    return new PersonBuilder();
  }
  
  name(value) {
    if (!value || typeof value !== "string") {
      throw new TypeError("ชื่อต้องเป็น string");
    }
    this._data.name = value;
    return this; // chainable
  }
  
  age(value) {
    if (value < 0 || value > 150) {
      throw new RangeError("อายุไม่ถูกต้อง");
    }
    this._data.age = value;
    return this;
  }
  
  email(value) {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (!emailRegex.test(value)) {
      throw new Error("รูปแบบ email ไม่ถูกต้อง");
    }
    this._data.email = value;
    return this;
  }
  
  address(street, city, zip) {
    this._data.address = { street, city, zip };
    return this;
  }
  
  role(value) {
    const validRoles = ["admin", "user", "moderator"];
    if (!validRoles.includes(value)) {
      throw new Error(`role ต้องเป็น: ${validRoles.join(", ")}`);
    }
    this._data.role = value;
    return this;
  }
  
  withDefaults(defaults = {}) {
    this._data = { ...defaults, ...this._data };
    return this;
  }
  
  build() {
    // ตรวจสอบ required fields
    const required = ["name", "email"];
    required.forEach(field => {
      if (!this._data[field]) {
        throw new Error(`ต้องกำหนด ${field}`);
      }
    });
    
    // สร้าง immutable object
    return Object.freeze({ ...this._data });
  }
}

// ใช้งาน
const person3 = PersonBuilder.create()
  .name("สมชาย รักไทย")
  .age(25)
  .email("somchai@example.com")
  .address("123 สุขุมวิท", "กรุงเทพ", "10110")
  .role("user")
  .withDefaults({ active: true, createdAt: new Date().toISOString() })
  .build();

console.log("Person:", person3);
console.log("frozen:", Object.isFrozen(person3)); // true

// ลอง mutate - ไม่ได้
person3.name = "เปลี่ยนชื่อ";
console.log("ชื่อยังเดิม:", person3.name);
```

---

## ตารางสรุป Object Methods

| Method | คำอธิบาย |
|--------|-----------|
| `Object.create(proto)` | สร้าง object ด้วย prototype ที่กำหนด |
| `Object.defineProperty()` | กำหนด property descriptor |
| `Object.defineProperties()` | กำหนด หลาย property descriptors |
| `Object.getOwnPropertyDescriptor()` | ดู descriptor ของ property |
| `Object.getOwnPropertyDescriptors()` | ดู descriptors ทั้งหมด |
| `Object.getOwnPropertyNames()` | keys ทั้งหมด (รวม non-enumerable) |
| `Object.getPrototypeOf()` | ดู prototype |
| `Object.setPrototypeOf()` | กำหนด prototype |
| `Object.keys()` | own + enumerable string keys |
| `Object.values()` | own + enumerable values |
| `Object.entries()` | own + enumerable [key, value] pairs |
| `Object.fromEntries()` | สร้าง object จาก entries |
| `Object.assign()` | copy properties ระหว่าง objects |
| `Object.freeze()` | ล็อคทั้งหมด |
| `Object.seal()` | ล็อค structure แต่แก้ค่าได้ |
| `Object.preventExtensions()` | ล็อคไม่ให้เพิ่ม property |
| `Object.is()` | เปรียบเทียบ strict + handle NaN/±0 |
| `Object.hasOwn()` | ตรวจสอบ own property (ปลอดภัย) |
| `structuredClone()` | deep clone |

---

*จบ Part 30: Object Methods ขั้นสูง*

---

## สรุปท้ายบท: Parts 26-30

ใน 5 Parts นี้เราได้เรียนรู้:

1. **Promises**: การจัดการ asynchronous code ด้วย Promise chain และ methods ต่างๆ
2. **Async/Await**: Syntax ที่ทำให้ asynchronous code อ่านง่ายขึ้น
3. **Fetch API**: การสื่อสารกับ HTTP servers อย่างมืออาชีพ
4. **Array Methods**: การประมวลผล array อย่างมีประสิทธิภาพ
5. **Object Methods**: การควบคุม objects อย่างละเอียด

ทักษะเหล่านี้เป็นพื้นฐานสำคัญของการพัฒนา JavaScript สมัยใหม่!
