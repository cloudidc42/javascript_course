# Part 32: Prototype Chain (Steps 611-630)

## บทนำ

Prototype Chain คือหัวใจสำคัญของ JavaScript's object system ก่อนที่จะมี class syntax ใน ES6 การสร้าง OOP ใน JavaScript ทำผ่าน prototype-based inheritance ซึ่งยังคงทำงานอยู่เบื้องหลัง class syntax ปัจจุบัน การเข้าใจ prototype chain จะทำให้เข้าใจ JavaScript อย่างลึกซึ้ง

---

## Step 611: Prototype คืออะไร?

ทุก object ใน JavaScript มี property ภายในที่เรียกว่า `[[Prototype]]` ซึ่งชี้ไปที่ object อื่น

```javascript
// Object ธรรมดา
const person = {
  name: 'สมชาย',
  greet() {
    return `สวัสดี ฉัน${this.name}`;
  }
};

// JavaScript lookup: ถ้าหา property ไม่เจอใน object
// จะไปหาใน prototype ต่อไปเรื่อยๆ จนถึง null
const employee = {
  company: 'Acme Corp',
  // employee มี [[Prototype]] ชี้ไปที่ person
};

Object.setPrototypeOf(employee, person);

console.log(employee.name);    // 'สมชาย' (จาก prototype chain)
console.log(employee.company); // 'Acme Corp' (own property)
console.log(employee.greet()); // 'สวัสดี ฉันสมชาย' (จาก prototype chain)
```

```javascript
// Prototype chain visualization
const a = { x: 1 };
const b = Object.create(a); // b's [[Prototype]] = a
b.y = 2;
const c = Object.create(b); // c's [[Prototype]] = b
c.z = 3;

console.log(c.z); // 3 (own property)
console.log(c.y); // 2 (ใน b)
console.log(c.x); // 1 (ใน a)
console.log(c.w); // undefined (ไม่มี และถึง null แล้ว)

// Visualize chain:
// c -> b -> a -> Object.prototype -> null
console.log(Object.getPrototypeOf(c) === b);           // true
console.log(Object.getPrototypeOf(b) === a);           // true
console.log(Object.getPrototypeOf(a) === Object.prototype); // true
console.log(Object.getPrototypeOf(Object.prototype)); // null (จุดสิ้นสุด)
```

---

## Step 612: `__proto__` vs `prototype`

คนมักสับสนระหว่างสอง property นี้:

```javascript
// __proto__ คือ accessor สำหรับ [[Prototype]] ของ object instance
const obj = {};
console.log(obj.__proto__ === Object.prototype); // true

// prototype คือ property ของ function (constructor)
// ใช้เป็น [[Prototype]] สำหรับ objects ที่สร้างด้วย new
function Person(name) {
  this.name = name;
}

const p = new Person('อลิส');
console.log(p.__proto__ === Person.prototype); // true
console.log(Person.prototype.constructor === Person); // true

// ความสัมพันธ์:
// p.__proto__ === Person.prototype ← ของ instance
// Person.prototype === { constructor: Person, ... } ← ของ function
```

```javascript
// ตัวอย่างชัดเจน
function Dog(name) {
  this.name = name;
}

// เพิ่ม method ผ่าน prototype
Dog.prototype.bark = function() {
  return `${this.name}: โฮ่ง!`;
};

Dog.prototype.type = 'สุนัข';

const d1 = new Dog('บัดดี้');
const d2 = new Dog('มักซ์');

// ทั้ง d1 และ d2 ใช้ prototype เดียวกัน
console.log(d1.bark()); // บัดดี้: โฮ่ง!
console.log(d2.bark()); // มักซ์: โฮ่ง!
console.log(d1.type);   // สุนัข
console.log(d2.type);   // สุนัข

// __proto__ ของ instance ชี้ไปที่ Dog.prototype
console.log(d1.__proto__ === Dog.prototype); // true
console.log(d1.__proto__ === d2.__proto__);  // true - shared!

// แต่ own property ของ instance ไม่ใช่ shared
console.log(d1.hasOwnProperty('name')); // true
console.log(d1.hasOwnProperty('bark')); // false (อยู่ใน prototype)
```

---

## Step 613: Prototype Chain Lookup

JavaScript ค้นหา property ตาม chain จนพบหรือถึง null

```javascript
// การ lookup ตาม chain
function trace(obj, prop) {
  let current = obj;
  let depth = 0;

  while (current !== null) {
    if (Object.prototype.hasOwnProperty.call(current, prop)) {
      const location = depth === 0 ? 'own property' : `prototype level ${depth}`;
      console.log(`พบ "${prop}" ที่ ${location}:`, current[prop]);
      return;
    }
    current = Object.getPrototypeOf(current);
    depth++;
  }
  console.log(`ไม่พบ "${prop}" ใน prototype chain`);
}

const grandparent = { ancestorProp: 'จากบรรพบุรุษ' };
const parent = Object.create(grandparent);
parent.parentProp = 'จากพ่อแม่';
const child = Object.create(parent);
child.childProp = 'ของตัวเอง';

trace(child, 'childProp');    // พบ "childProp" ที่ own property
trace(child, 'parentProp');   // พบ "parentProp" ที่ prototype level 1
trace(child, 'ancestorProp'); // พบ "ancestorProp" ที่ prototype level 2
trace(child, 'toString');     // พบ "toString" ที่ prototype level 3 (Object.prototype)
trace(child, 'nonExistent');  // ไม่พบ "nonExistent" ใน prototype chain
```

```javascript
// Method resolution order
class A {
  method() { return 'A.method()'; }
  shared() { return 'A.shared()'; }
}

class B extends A {
  method() { return `B.method() -> ${super.method()}`; }
  // inherited shared()
}

class C extends B {
  method() { return `C.method() -> ${super.method()}`; }
  // inherited shared()
}

const c = new C();
console.log(c.method());  // C.method() -> B.method() -> A.method()
console.log(c.shared());  // A.shared()

// Prototype chain สำหรับ c:
// c -> C.prototype -> B.prototype -> A.prototype -> Object.prototype -> null
```

---

## Step 614: Object.prototype Methods

Object.prototype มี methods ที่ inherited โดย objects ทุกตัว

```javascript
const obj = { x: 1, y: 2 };

// hasOwnProperty - ตรวจสอบว่า property นั้นเป็น own property
console.log(obj.hasOwnProperty('x'));        // true
console.log(obj.hasOwnProperty('toString')); // false (inherited)

// toString - แปลงเป็น string
console.log(obj.toString()); // [object Object]

// valueOf - ค่า primitive
console.log(obj.valueOf()); // { x: 1, y: 2 }

// isPrototypeOf - ตรวจสอบ prototype chain
const parent = {};
const child = Object.create(parent);
const grandchild = Object.create(child);

console.log(parent.isPrototypeOf(child));      // true
console.log(parent.isPrototypeOf(grandchild)); // true (ผ่าน chain)
console.log(child.isPrototypeOf(parent));      // false

// propertyIsEnumerable
const arr = [1, 2, 3];
console.log(arr.propertyIsEnumerable(0));        // true (own, enumerable)
console.log(arr.propertyIsEnumerable('length')); // false (own, non-enumerable)
console.log(arr.propertyIsEnumerable('push'));   // false (inherited)
```

```javascript
// Object static methods สำหรับ prototype
const proto = {
  greet() { return `สวัสดี ${this.name}`; }
};

const obj1 = Object.create(proto);
obj1.name = 'อลิส';

// Object.getPrototypeOf
console.log(Object.getPrototypeOf(obj1) === proto); // true

// Object.setPrototypeOf (ไม่แนะนำ - ช้า)
const obj2 = { name: 'บ็อบ' };
Object.setPrototypeOf(obj2, proto);
console.log(obj2.greet()); // สวัสดี บ็อบ

// Object.keys vs for...in
const parent2 = { inherited: 'yes' };
const child2 = Object.create(parent2);
child2.own = 'yes';

console.log(Object.keys(child2)); // ['own'] - own enumerable only
for (const key in child2) {
  console.log(key); // 'own', 'inherited' - including inherited
}

// Object.getOwnPropertyNames
const obj3 = Object.create({}, {
  visible: { value: 1, enumerable: true },
  hidden: { value: 2, enumerable: false }
});

console.log(Object.keys(obj3));                 // ['visible']
console.log(Object.getOwnPropertyNames(obj3));  // ['visible', 'hidden']
```

---

## Step 615: Setting Prototypes

```javascript
// วิธีการตั้งค่า prototype

// 1. Object.create()
const animal = {
  breathe() { console.log(`${this.name} หายใจ`); },
  eat(food) { console.log(`${this.name} กิน ${food}`); }
};

const dog = Object.create(animal);
dog.name = 'บัดดี้';
dog.bark = function() { console.log(`${this.name}: โฮ่ง!`); };

dog.breathe(); // บัดดี้ หายใจ
dog.bark();    // บัดดี้: โฮ่ง!

// 2. Constructor functions
function Bird(name, canFly) {
  this.name = name;
  this.canFly = canFly;
}
Bird.prototype.tweet = function() {
  console.log(`${this.name}: จิ้บๆ`);
};

const sparrow = new Bird('กระจอก', true);
sparrow.tweet(); // กระจอก: จิ้บๆ

// 3. Class syntax (ES6)
class Fish {
  constructor(name) {
    this.name = name;
  }
  swim() { console.log(`${this.name} ว่ายน้ำ`); }
}

const nemo = new Fish('นีโม');
nemo.swim(); // นีโม ว่ายน้ำ

// 4. Object.assign ไม่เหมือน prototype
const base = { x: 1 };
const ext = Object.assign({}, base, { y: 2 });
// ext เป็น own properties ทั้งหมด ไม่ใช่ prototype chain
```

---

## Step 616: Constructor Functions (Pre-Class Era)

```javascript
// ก่อนมี class syntax คนใช้ constructor functions
function Person(name, age) {
  // ค่าที่กำหนดใน constructor เป็น own properties ของ instance
  this.name = name;
  this.age = age;
}

// เพิ่ม methods ผ่าน prototype (shared ระหว่าง instances ทุกตัว)
Person.prototype.greet = function() {
  return `สวัสดี ฉันชื่อ ${this.name}`;
};

Person.prototype.haveBirthday = function() {
  this.age++;
  return this;
};

Person.prototype.toString = function() {
  return `Person(${this.name}, ${this.age})`;
};

// "Static method"
Person.create = function(name, age) {
  return new Person(name, age);
};

// ใช้งาน
const alice = new Person('อลิส', 25);
const bob = Person.create('บ็อบ', 30);

console.log(alice.greet()); // สวัสดี ฉันชื่อ อลิส
alice.haveBirthday();
console.log(alice.age);     // 26

// Methods shared กัน
console.log(alice.greet === bob.greet); // true - same function in prototype!
```

```javascript
// Inheritance ก่อน ES6 (เพื่อให้เข้าใจ class ทำงานอย่างไร)
function Animal(name, sound) {
  this.name = name;
  this.sound = sound;
}

Animal.prototype.makeSound = function() {
  console.log(`${this.name}: ${this.sound}`);
};

// Dog สืบทอดจาก Animal
function Dog(name, breed) {
  Animal.call(this, name, 'โฮ่ง'); // เหมือน super()
  this.breed = breed;
}

// ตั้งค่า prototype chain
Dog.prototype = Object.create(Animal.prototype);
Dog.prototype.constructor = Dog; // fix constructor reference

Dog.prototype.fetch = function() {
  console.log(`${this.name} เก็บลูกบอล!`);
};

const rex = new Dog('เร็กซ์', 'เยอรมันเชพเพิร์ด');
rex.makeSound(); // เร็กซ์: โฮ่ง
rex.fetch();     // เร็กซ์ เก็บลูกบอล!

console.log(rex instanceof Dog);    // true
console.log(rex instanceof Animal); // true
console.log(rex.constructor === Dog); // true (เพราะ fix ไว้)
```

---

## Step 617: Prototype Property บน Functions

```javascript
// ทุก function มี .prototype property
function MyFunc() {}
console.log(typeof MyFunc.prototype); // 'object'
console.log(MyFunc.prototype.constructor === MyFunc); // true

// Arrow functions ไม่มี .prototype
const ArrowFunc = () => {};
console.log(ArrowFunc.prototype); // undefined

// Methods ใน class ไม่มี .prototype
class MyClass {
  method() {}
}
const instance = new MyClass();
console.log(instance.method.prototype); // undefined? No:
// instance.method เป็น bound method

// Built-in functions มี prototype ด้วย
console.log(Array.prototype.map);  // function
console.log(String.prototype.toUpperCase); // function

// เพิ่ม method ใน built-in prototype (monkey patching - ไม่แนะนำ!)
// แต่ควรรู้ว่าทำได้
String.prototype.isPalindrome = function() {
  const str = this.toLowerCase().replace(/[^a-z0-9]/g, '');
  return str === str.split('').reverse().join('');
};

console.log('racecar'.isPalindrome()); // true
console.log('hello'.isPalindrome());   // false
// ลบออกหลังทดสอบ
delete String.prototype.isPalindrome;
```

---

## Step 618: new Keyword Internals

```javascript
// new ทำอะไรบ้าง? มาดูขั้นตอน:
function simulateNew(Constructor, ...args) {
  // 1. สร้าง empty object
  const obj = {};

  // 2. ตั้งค่า [[Prototype]] ให้ชี้ไปที่ Constructor.prototype
  Object.setPrototypeOf(obj, Constructor.prototype);
  // เทียบเท่า: obj.__proto__ = Constructor.prototype;

  // 3. เรียก Constructor กับ obj เป็น this
  const result = Constructor.apply(obj, args);

  // 4. ถ้า Constructor return object ให้ใช้ผลนั้น
  //    ถ้าไม่ ให้ใช้ obj ที่สร้างไว้
  return (typeof result === 'object' && result !== null) ? result : obj;
}

function Point(x, y) {
  this.x = x;
  this.y = y;
}

Point.prototype.toString = function() {
  return `(${this.x}, ${this.y})`;
};

const p1 = new Point(1, 2);
const p2 = simulateNew(Point, 3, 4);

console.log(p1.toString()); // (1, 2)
console.log(p2.toString()); // (3, 4)
console.log(p1 instanceof Point); // true
console.log(p2 instanceof Point); // true
```

```javascript
// เข้าใจ new.target
function Foo() {
  if (!new.target) {
    // ถ้าไม่ได้เรียกด้วย new
    return new Foo();
  }
  this.value = 42;
}

const f1 = new Foo();
const f2 = Foo(); // เรียกแบบ function ธรรมดา แต่ก็ได้ instance

console.log(f1.value); // 42
console.log(f2.value); // 42

// new.target ใน class
class Base {
  constructor() {
    console.log('new.target:', new.target.name);
  }
}

class Derived extends Base {
  constructor() {
    super(); // จะ log 'Derived' ไม่ใช่ 'Base'
  }
}

new Base();    // new.target: Base
new Derived(); // new.target: Derived
```

---

## Step 619: Object.create() Revisited

```javascript
// Object.create สร้าง object พร้อมกำหนด prototype
const vehicleProto = {
  start() { console.log(`${this.brand} เริ่มเครื่อง`); },
  stop() { console.log(`${this.brand} ดับเครื่อง`); },
  describe() { return `${this.year} ${this.brand} ${this.model}`; }
};

// สร้าง car ที่ใช้ vehicleProto เป็น prototype
const car1 = Object.create(vehicleProto);
car1.brand = 'Toyota';
car1.model = 'Camry';
car1.year = 2023;

const car2 = Object.create(vehicleProto);
car2.brand = 'Honda';
car2.model = 'Civic';
car2.year = 2022;

car1.start();           // Toyota เริ่มเครื่อง
console.log(car1.describe()); // 2023 Toyota Camry
console.log(car1.start === car2.start); // true - shared!
```

```javascript
// Object.create พร้อม property descriptors
const template = {
  greet() { return `Hello, ${this.name}`; }
};

const obj = Object.create(template, {
  name: {
    value: 'World',
    writable: true,
    enumerable: true,
    configurable: true
  },
  id: {
    value: Math.random(),
    writable: false,    // read-only
    enumerable: false,  // ไม่ปรากฏใน for...in
    configurable: false // ไม่สามารถลบหรือเปลี่ยนแปลงได้
  }
});

console.log(obj.greet()); // Hello, World
console.log(Object.keys(obj)); // ['name'] - id ถูกซ่อน

// Object.create(null) - สร้าง pure object ไม่มี prototype
const pureObj = Object.create(null);
pureObj.key = 'value';
console.log(Object.getPrototypeOf(pureObj)); // null
// pureObj.toString() // TypeError! ไม่มี inherited methods
// ใช้สำหรับ pure hash map
```

---

## Step 620: Prototype Inheritance Patterns

```javascript
// Pattern 1: Object.create inheritance
function createAnimal(name, sound) {
  return {
    name,
    sound,
    makeSound() { console.log(`${this.name}: ${this.sound}`); }
  };
}

function createDog(name, breed) {
  const animal = createAnimal(name, 'โฮ่ง');
  return Object.assign(Object.create(animal), {
    breed,
    fetch() { console.log(`${this.name} เก็บของ!`); }
  });
}

const dog = createDog('แม็กซ์', 'โกลเด้น');
dog.makeSound(); // แม็กซ์: โฮ่ง
dog.fetch();     // แม็กซ์ เก็บของ!

// Pattern 2: Classical via Constructor Functions
function Shape(color) {
  this.color = color;
}
Shape.prototype.describe = function() {
  return `${this.constructor.name} สีว่าง ${this.color}`;
};

function Circle(color, radius) {
  Shape.call(this, color);
  this.radius = radius;
}
Circle.prototype = Object.create(Shape.prototype);
Circle.prototype.constructor = Circle;
Circle.prototype.area = function() {
  return Math.PI * this.radius ** 2;
};

const c = new Circle('แดง', 5);
console.log(c.describe()); // Circle สีว่าง แดง
console.log(c.area().toFixed(2)); // 78.54
```

```javascript
// Pattern 3: Class-based (modern)
class Collection {
  #items;

  constructor(items = []) {
    this.#items = [...items];
  }

  add(item) {
    this.#items.push(item);
    return this;
  }

  remove(item) {
    const i = this.#items.indexOf(item);
    if (i > -1) this.#items.splice(i, 1);
    return this;
  }

  has(item) {
    return this.#items.includes(item);
  }

  get size() { return this.#items.length; }

  [Symbol.iterator]() {
    return this.#items[Symbol.iterator]();
  }

  toArray() { return [...this.#items]; }
}

class UniqueCollection extends Collection {
  add(item) {
    if (!this.has(item)) {
      super.add(item);
    }
    return this;
  }
}

const uc = new UniqueCollection();
uc.add(1).add(2).add(1).add(3);
console.log(uc.toArray()); // [1, 2, 3]
console.log(uc.size);      // 3
```

---

## Step 621: hasOwnProperty

```javascript
const parent = {
  inherited: 'ของ parent',
  toString() { return 'custom toString'; }
};

const child = Object.create(parent);
child.own = 'ของ child';
child.also = 'ของ child เช่นกัน';

// hasOwnProperty ตรวจสอบ own property เท่านั้น
console.log(child.hasOwnProperty('own'));       // true
console.log(child.hasOwnProperty('also'));      // true
console.log(child.hasOwnProperty('inherited')); // false
console.log(child.hasOwnProperty('toString'));  // false

// Object.hasOwn (ES2022 - recommended)
console.log(Object.hasOwn(child, 'own'));       // true
console.log(Object.hasOwn(child, 'inherited')); // false

// ทำไมต้องใช้ hasOwnProperty?
function processObject(obj) {
  for (const key in obj) {
    // for...in วิ่งผ่าน inherited properties ด้วย
    if (Object.hasOwn(obj, key)) {
      console.log(`Own: ${key} = ${obj[key]}`);
    } else {
      console.log(`Inherited: ${key}`);
    }
  }
}

processObject(child);
// Own: own = ของ child
// Own: also = ของ child เช่นกัน
// Inherited: inherited
```

```javascript
// ปัญหากับ hasOwnProperty ที่ถูก override
const malicious = Object.create(null);
malicious.hasOwnProperty = () => true; // override

// อันตราย!
// malicious.hasOwnProperty('anything') // ไม่ทำงานถูกต้อง

// วิธีที่ปลอดภัย
function safeHasOwn(obj, prop) {
  return Object.prototype.hasOwnProperty.call(obj, prop);
  // หรือใช้ Object.hasOwn(obj, prop) ใน ES2022
}

// กับ pure objects (null prototype)
const pureHash = Object.create(null);
pureHash.key = 'value';
// pureHash.hasOwnProperty('key') // TypeError! ไม่มี method นี้
console.log(Object.hasOwn(pureHash, 'key')); // true - ปลอดภัย
```

---

## Step 622: Property Shadowing

```javascript
// เมื่อ child กำหนด property ที่มีชื่อเดียวกับ prototype
const proto = {
  name: 'default name',
  greet() { return `สวัสดี ฉัน ${this.name}`; }
};

const obj = Object.create(proto);

console.log(obj.name);    // 'default name' (จาก prototype)
console.log(obj.greet()); // 'สวัสดี ฉัน default name'

// กำหนด own property - "shadows" prototype property
obj.name = 'own name';

console.log(obj.name);    // 'own name' (own property)
console.log(obj.greet()); // 'สวัสดี ฉัน own name'

// ลบ own property เพื่อ reveal prototype
delete obj.name;
console.log(obj.name);    // 'default name' (กลับมาใช้ prototype)
```

```javascript
// Shadowing กับ non-writable properties
const base = {};
Object.defineProperty(base, 'fixed', {
  value: 42,
  writable: false,
  enumerable: true,
  configurable: false
});

const derived = Object.create(base);

// ใน strict mode
'use strict';
try {
  derived.fixed = 100; // TypeError ใน strict mode!
} catch (e) {
  console.log(e.message); // Cannot assign to read only property
}

// ไม่มี strict mode - การ assign จะ silent fail
// แต่ไม่สร้าง own property!

// วิธีที่ถูก - ใช้ defineProperty
Object.defineProperty(derived, 'fixed', {
  value: 100,
  writable: true,
  enumerable: true,
  configurable: true
});
console.log(derived.fixed); // 100 (own property)
console.log(base.fixed);    // 42 (ยังอยู่)
```

---

## Step 623: Prototype Pollution (Security Concern)

```javascript
// Prototype Pollution - การโจมตีด้านความปลอดภัยที่สำคัญ

// ช่องโหว่: deep merge function ที่ไม่ปลอดภัย
function unsafeMerge(target, source) {
  for (const key in source) {
    if (typeof source[key] === 'object') {
      if (!target[key]) target[key] = {};
      unsafeMerge(target[key], source[key]);
    } else {
      target[key] = source[key];
    }
  }
  return target;
}

// การโจมตี
const maliciousPayload = JSON.parse('{"__proto__": {"isAdmin": true}}');
const userConfig = {};

unsafeMerge(userConfig, maliciousPayload);

// ตอนนี้ทุก object ได้รับ isAdmin: true!
const newUser = {};
console.log(newUser.isAdmin); // true ← อันตราย!

// ทำความสะอาด (reset prototype)
delete Object.prototype.isAdmin;

// วิธีป้องกัน Prototype Pollution:
function safeMerge(target, source) {
  for (const key of Object.keys(source)) { // ใช้ Object.keys แทน for...in
    // ตรวจสอบ key ที่อันตราย
    if (key === '__proto__' || key === 'constructor' || key === 'prototype') {
      continue; // ข้าม!
    }

    if (typeof source[key] === 'object' && source[key] !== null) {
      if (!Object.hasOwn(target, key)) target[key] = {};
      safeMerge(target[key], source[key]);
    } else {
      target[key] = source[key];
    }
  }
  return target;
}

const safe = {};
safeMerge(safe, maliciousPayload);
const testObj = {};
console.log(testObj.isAdmin); // undefined ← ปลอดภัย!
```

```javascript
// การป้องกันอื่นๆ
// 1. Object.freeze สำหรับ Object.prototype
// Object.freeze(Object.prototype); // ทำให้ prototype immutable

// 2. สร้าง object ที่ไม่มี prototype
const safeHash = Object.create(null);
safeHash['__proto__'] = 'ไม่อันตราย'; // เป็นแค่ string
console.log(Object.getPrototypeOf(safeHash)); // null

// 3. ใช้ Map แทน plain object สำหรับ user data
const safeStore = new Map();
safeStore.set('__proto__', 'ไม่อันตราย'); // ปลอดภัยอย่างสมบูรณ์
console.log(safeStore.get('__proto__')); // 'ไม่อันตราย'
```

---

## Step 624: Class Sugar over Prototypes

Classes เป็น syntactic sugar บน prototype system:

```javascript
// Class syntax
class Animal {
  constructor(name) {
    this.name = name;
  }
  speak() {
    return `${this.name} ส่งเสียง`;
  }
  static create(name) {
    return new Animal(name);
  }
}

// Equivalent prototype code
function AnimalProto(name) {
  this.name = name;
}
AnimalProto.prototype.speak = function() {
  return `${this.name} ส่งเสียง`;
};
AnimalProto.create = function(name) {
  return new AnimalProto(name);
};

// ทั้งสองสร้าง structure เดียวกัน
const a1 = new Animal('สิงโต');
const a2 = new AnimalProto('เสือ');

console.log(a1.speak()); // สิงโต ส่งเสียง
console.log(a2.speak()); // เสือ ส่งเสียง

// ตรวจสอบว่า class methods อยู่ใน prototype
console.log(typeof Animal.prototype.speak);      // 'function'
console.log(typeof AnimalProto.prototype.speak); // 'function'

// Class methods ใน prototype ไม่ enumerable
const descriptor = Object.getOwnPropertyDescriptor(Animal.prototype, 'speak');
console.log(descriptor.enumerable); // false (class default)

const descriptor2 = Object.getOwnPropertyDescriptor(AnimalProto.prototype, 'speak');
console.log(descriptor2.enumerable); // true (ใส่ตรงๆ)
```

```javascript
// Class inheritance = prototype chain
class Vehicle {
  constructor(speed) {
    this.speed = speed;
  }
  move() { return `เคลื่อนที่ด้วยความเร็ว ${this.speed}`; }
}

class Car extends Vehicle {
  constructor(speed, brand) {
    super(speed);
    this.brand = brand;
  }
  honk() { return `${this.brand}: บีบแตร!`; }
}

// ตรวจสอบ prototype chain
const car = new Car(120, 'Toyota');

console.log(Object.getPrototypeOf(car) === Car.prototype);     // true
console.log(Object.getPrototypeOf(Car.prototype) === Vehicle.prototype); // true
console.log(Object.getPrototypeOf(Vehicle.prototype) === Object.prototype); // true

// Methods อยู่ใน respective prototypes
console.log(Object.hasOwn(Car.prototype, 'honk')); // true
console.log(Object.hasOwn(Car.prototype, 'move')); // false (ใน Vehicle.prototype)
console.log(Object.hasOwn(Vehicle.prototype, 'move')); // true
```

---

## Step 625: Checking Prototypes

```javascript
// วิธีต่างๆ ในการตรวจสอบ prototype
class A {}
class B extends A {}
class C extends B {}

const c = new C();

// 1. instanceof
console.log(c instanceof C); // true
console.log(c instanceof B); // true
console.log(c instanceof A); // true

// 2. isPrototypeOf
console.log(C.prototype.isPrototypeOf(c)); // true
console.log(B.prototype.isPrototypeOf(c)); // true
console.log(A.prototype.isPrototypeOf(c)); // true

// 3. Object.getPrototypeOf
console.log(Object.getPrototypeOf(c) === C.prototype);           // true
console.log(Object.getPrototypeOf(Object.getPrototypeOf(c)) === B.prototype); // true

// 4. constructor property
console.log(c.constructor === C); // true
console.log(c.constructor.name);  // 'C'

// 5. Object.prototype.toString (type tag)
const tag = Object.prototype.toString.call(c);
console.log(tag); // [object Object]

// Custom Symbol.toStringTag
class MyMap extends Map {
  get [Symbol.toStringTag]() {
    return 'MyMap';
  }
}
const myMap = new MyMap();
console.log(Object.prototype.toString.call(myMap)); // [object MyMap]
```

```javascript
// ฟังก์ชัน utility สำหรับตรวจสอบ prototype chain
function getPrototypeChain(obj) {
  const chain = [];
  let current = obj;

  while (current !== null) {
    chain.push(current.constructor?.name || 'Anonymous');
    current = Object.getPrototypeOf(current);
  }

  return chain;
}

class Animal {}
class Dog extends Animal {}
class Poodle extends Dog {}

const poodle = new Poodle();
console.log(getPrototypeChain(poodle));
// ['Poodle', 'Dog', 'Animal', 'Object', 'undefined']
// (undefined เพราะ Object.prototype.constructor = Object, แล้ว getPrototypeOf = null)
```

---

## Step 626: Performance Implications

```javascript
// Prototype chain lookup มีผลต่อ performance
// Property ที่อยู่ลึกกว่าจะช้ากว่า

// ทดสอบ performance (จำลอง)
function measureLookup(obj, prop, iterations = 1000000) {
  const start = performance.now();
  for (let i = 0; i < iterations; i++) {
    const _ = obj[prop]; // access property
  }
  return performance.now() - start;
}

// สร้าง chain ที่ลึก
let deepObj = { prop: 'value' };
for (let i = 0; i < 10; i++) {
  deepObj = Object.create(deepObj);
}
deepObj.ownProp = 'own';

// own property vs prototype chain
// (ในทางปฏิบัติ V8 optimize ดีมาก แต่ยังมีความแตกต่าง)
const t1 = measureLookup(deepObj, 'ownProp', 1000000);
const t2 = measureLookup(deepObj, 'prop', 1000000);

console.log(`Own property: ${t1.toFixed(2)}ms`);
console.log(`Deep chain: ${t2.toFixed(2)}ms`);
```

```javascript
// Best Practices สำหรับ performance

// 1. จัดการ prototype chain ให้สั้น
// ไม่แนะนำ - chain ยาวเกินไป
class Level1 {}
class Level2 extends Level1 {}
class Level3 extends Level2 {}
class Level4 extends Level3 {}
class Level5 extends Level4 {} // chain ยาว 6 levels

// แนะนำ - ใช้ composition แทน deep inheritance
class Component {
  constructor(capabilities) {
    Object.assign(this, capabilities);
  }
}

// 2. Cache property lookups ที่ใช้บ่อย
class PerformantClass {
  constructor() {
    this.data = [1, 2, 3, 4, 5];
    // Cache method reference
    this._forEach = Array.prototype.forEach.bind(this.data);
  }

  process() {
    this._forEach(item => {
      // ทำงาน
    });
  }
}

// 3. ใช้ Object.freeze สำหรับ immutable objects
const CONFIG = Object.freeze({
  maxRetries: 3,
  timeout: 5000,
  baseUrl: 'https://api.example.com'
});

// CONFIG.maxRetries = 10; // silent fail (non-strict) หรือ TypeError (strict)
```

---

## Step 627: Object.prototype Method Tricks

```javascript
// toString สำหรับ type checking
function typeOf(value) {
  return Object.prototype.toString.call(value).slice(8, -1);
}

console.log(typeOf(42));         // 'Number'
console.log(typeOf('hello'));    // 'String'
console.log(typeOf(true));       // 'Boolean'
console.log(typeOf(null));       // 'Null'
console.log(typeOf(undefined));  // 'Undefined'
console.log(typeOf([]));         // 'Array'
console.log(typeOf({}));         // 'Object'
console.log(typeOf(() => {}));   // 'Function'
console.log(typeOf(new Map()));  // 'Map'
console.log(typeOf(new Set()));  // 'Set'
console.log(typeOf(/regex/));    // 'RegExp'
console.log(typeOf(new Date())); // 'Date'

// เปรียบเทียบกับ typeof
console.log(typeof null);        // 'object' ← bug ใน JS!
console.log(typeOf(null));       // 'Null' ← ถูกต้อง
```

```javascript
// valueOf และ toString สำหรับ type coercion
class Money {
  constructor(amount, currency = 'THB') {
    this.amount = amount;
    this.currency = currency;
  }

  valueOf() {
    // ใช้ในการเปรียบเทียบตัวเลข
    return this.amount;
  }

  toString() {
    return `${this.amount} ${this.currency}`;
  }

  [Symbol.toPrimitive](hint) {
    if (hint === 'number') return this.amount;
    if (hint === 'string') return this.toString();
    return this.amount; // default
  }
}

const price1 = new Money(100);
const price2 = new Money(200);

console.log(price1 + price2);    // 300 (valueOf)
console.log(price1 > 50);        // true (valueOf)
console.log(`ราคา: ${price1}`);  // ราคา: 100 THB (toString)
console.log(price1 * 2);         // 200 (Symbol.toPrimitive number hint)
```

---

## Step 628: Inheritance Best Practices

```javascript
// LSP: Liskov Substitution Principle
// Subclass ควรสามารถแทนที่ parent class ได้

class Rectangle {
  constructor(width, height) {
    this.width = width;
    this.height = height;
  }

  setWidth(w) { this.width = w; return this; }
  setHeight(h) { this.height = h; return this; }

  get area() { return this.width * this.height; }
}

// ละเมิด LSP! Square สืบทอด Rectangle แต่ behavior ต่างกัน
class BadSquare extends Rectangle {
  setWidth(w) {
    this.width = w;
    this.height = w; // force equal sides
    return this;
  }
  setHeight(h) {
    this.width = h;
    this.height = h;
    return this;
  }
}

function testRectangle(rect) {
  rect.setWidth(5).setHeight(10);
  console.log('Expected area: 50, Got:', rect.area);
  // BadSquare จะได้ 100 ไม่ใช่ 50 → ละเมิด LSP
}

testRectangle(new Rectangle(0, 0));  // Expected: 50, Got: 50 ✓
testRectangle(new BadSquare(0, 0));  // Expected: 50, Got: 100 ✗

// วิธีที่ดีกว่า: ไม่ให้ Square สืบทอด Rectangle
class Square {
  constructor(side) {
    this.side = side;
  }
  setSide(s) { this.side = s; return this; }
  get area() { return this.side ** 2; }
}
```

```javascript
// Composition over Inheritance
// แทนที่จะสร้าง hierarchy ลึก ให้ใช้ composition

const canSwim = {
  swim() { return `${this.name} ว่ายน้ำ`; }
};

const canFly = {
  fly() { return `${this.name} บิน`; }
};

const canRun = {
  run() { return `${this.name} วิ่ง`; }
};

// สร้าง animal ด้วย composition
function createDuck(name) {
  return {
    name,
    ...canSwim,
    ...canFly,
    quack() { return `${name}: ก้าก!`; }
  };
}

function createFish(name) {
  return {
    name,
    ...canSwim
  };
}

const duck = createDuck('โดนัลด์');
console.log(duck.swim());  // โดนัลด์ ว่ายน้ำ
console.log(duck.fly());   // โดนัลด์ บิน
console.log(duck.quack()); // โดนัลด์: ก้าก!

const fish = createFish('นีโม');
console.log(fish.swim()); // นีโม ว่ายน้ำ
// fish.fly() // TypeError - ปลาบินไม่ได้
```

---

## Step 629: Advanced Prototype Manipulation

```javascript
// Reflect API
class Person {
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }
  greet() { return `สวัสดี ${this.name}`; }
}

const p = new Person('อลิส', 25);

// Reflect.ownKeys - ดึง all own keys รวม Symbol
const sym = Symbol('secret');
p[sym] = 'hidden';
console.log(Reflect.ownKeys(p)); // ['name', 'age', Symbol(secret)]

// Reflect.getPrototypeOf
console.log(Reflect.getPrototypeOf(p) === Person.prototype); // true

// Reflect.has (คล้าย 'in' operator)
console.log(Reflect.has(p, 'name'));   // true
console.log(Reflect.has(p, 'greet')); // true (ใน prototype)

// Proxy สำหรับ intercept prototype operations
const handler = {
  get(target, prop, receiver) {
    if (prop === 'greet') {
      return function() {
        return `[intercepted] ${Reflect.get(target, prop, receiver).call(this)}`;
      };
    }
    return Reflect.get(target, prop, receiver);
  }
};

const proxied = new Proxy(p, handler);
console.log(proxied.greet()); // [intercepted] สวัสดี อลิส
console.log(proxied.name);    // อลิส (ไม่ intercept)
```

---

## Step 630: สรุปและ Real-World Example

```javascript
// Real-world example: Observable State Management
class Observable {
  #value;
  #observers = [];

  constructor(initialValue) {
    this.#value = initialValue;
  }

  get value() { return this.#value; }

  set value(newVal) {
    const oldVal = this.#value;
    this.#value = newVal;
    this.#notify(newVal, oldVal);
  }

  #notify(newVal, oldVal) {
    this.#observers.forEach(obs => obs(newVal, oldVal));
  }

  subscribe(observer) {
    this.#observers.push(observer);
    return () => {
      this.#observers = this.#observers.filter(o => o !== observer);
    };
  }
}

class ComputedObservable extends Observable {
  #dependencies;
  #compute;
  #cleanups = [];

  constructor(compute, ...dependencies) {
    super(compute(...dependencies.map(d => d.value)));
    this.#compute = compute;
    this.#dependencies = dependencies;

    // Subscribe ไปยัง dependencies
    this.#cleanups = dependencies.map(dep =>
      dep.subscribe(() => {
        this.value = compute(...dependencies.map(d => d.value));
      })
    );
  }

  destroy() {
    this.#cleanups.forEach(cleanup => cleanup());
  }
}

// ใช้งาน
const firstName = new Observable('สมชาย');
const lastName = new Observable('ใจดี');
const fullName = new ComputedObservable(
  (f, l) => `${f} ${l}`,
  firstName,
  lastName
);

// Subscribe ดูการเปลี่ยนแปลง
fullName.subscribe((newVal) => {
  console.log(`ชื่อเปลี่ยนเป็น: ${newVal}`);
});

console.log(fullName.value); // สมชาย ใจดี
firstName.value = 'สมหญิง'; // ชื่อเปลี่ยนเป็น: สมหญิง ใจดี
lastName.value = 'รักดี';   // ชื่อเปลี่ยนเป็น: สมหญิง รักดี
```

---

## แบบฝึกหัด (Exercises)

### แบบฝึกหัดที่ 1: ตรวจสอบ Prototype Chain
```javascript
// เขียนฟังก์ชันที่แสดง prototype chain ของ object ให้ครบ
function visualizeChain(obj) {
  // TODO: แสดง chain แบบ:
  // instance
  //   -> ClassName.prototype
  //     -> ParentClassName.prototype
  //       -> Object.prototype
  //         -> null
}

class A { methodA() {} }
class B extends A { methodB() {} }
class C extends B { methodC() {} }

visualizeChain(new C());
```

### แบบฝึกหัดที่ 2: Deep Clone ที่ปลอดภัย
```javascript
// สร้าง deepClone ที่:
// 1. ป้องกัน prototype pollution
// 2. handle circular references
// 3. copy ทั้ง Map, Set, Array, Date
function deepClone(obj) {
  // TODO
}
```

### แบบฝึกหัดที่ 3: Observable Array
```javascript
// สร้าง ObservableArray ที่ notify เมื่อมีการเปลี่ยนแปลง:
// - push, pop, shift, unshift, splice
// - sort, reverse
// ใช้ Proxy
class ObservableArray {
  // TODO
}
```

### แบบฝึกหัดที่ 4: Mixin System
```javascript
// สร้าง system สำหรับ mixins ที่:
// 1. ไม่ override existing methods
// 2. แก้ conflict ได้
// 3. ใช้ประโยชน์จาก prototype chain
function applyMixins(target, ...mixins) {
  // TODO
}
```

---

## สรุป

ใน Part 32 เราได้เรียนรู้:

1. **Prototype คืออะไร**: ทุก object มี [[Prototype]] ที่ชี้ไป object อื่น
2. **__proto__ vs prototype**: ความแตกต่างระหว่าง instance กับ constructor
3. **Prototype Chain Lookup**: การค้นหา property ตาม chain
4. **Object.prototype Methods**: hasOwnProperty, isPrototypeOf และอื่นๆ
5. **Constructor Functions**: รูปแบบก่อน ES6 class
6. **new Keyword Internals**: ขั้นตอนการทำงาน
7. **Object.create()**: สร้าง object พร้อมกำหนด prototype
8. **Prototype Pollution**: ช่องโหว่ด้านความปลอดภัย
9. **Class = Syntax Sugar**: classes ทำงานบน prototype system
10. **Performance**: ผลกระทบของ prototype chain ต่อ performance

ใน Part 33 เราจะเรียนรู้เรื่อง Closures!
