# Part 31: Classes และ Object-Oriented Programming (Steps 591-610)

## บทนำ

Object-Oriented Programming (OOP) เป็นแนวคิดการเขียนโปรแกรมที่จัดการโค้ดในรูปแบบของ "วัตถุ" (Objects) ซึ่งประกอบด้วยข้อมูล (Properties) และฟังก์ชัน (Methods) ที่ทำงานร่วมกัน JavaScript รองรับ OOP ผ่านคลาส (Classes) ซึ่ง introduced ใน ES6 โดยเป็น syntactic sugar บน prototype-based inheritance

---

## Step 591: แนวคิดพื้นฐานของ OOP - The Four Pillars

OOP มีหลักการสำคัญ 4 ข้อที่เรียกว่า "Four Pillars of OOP"

### 1. Encapsulation (การห่อหุ้ม)

การรวมข้อมูลและฟังก์ชันที่ทำงานกับข้อมูลนั้นไว้ด้วยกัน และซ่อนรายละเอียดภายในจากภายนอก

```javascript
// ตัวอย่าง Encapsulation - ห่อหุ้มข้อมูลและ behavior ไว้ด้วยกัน
class BankAccount {
  #balance = 0;  // private field - ซ่อนจากภายนอก
  #owner;

  constructor(owner, initialBalance) {
    this.#owner = owner;
    this.#balance = initialBalance;
  }

  // method สาธารณะที่ควบคุมการเข้าถึงข้อมูล
  deposit(amount) {
    if (amount <= 0) throw new Error('จำนวนเงินต้องมากกว่า 0');
    this.#balance += amount;
    console.log(`ฝากเงิน ${amount} บาท ยอดคงเหลือ: ${this.#balance} บาท`);
  }

  withdraw(amount) {
    if (amount <= 0) throw new Error('จำนวนเงินต้องมากกว่า 0');
    if (amount > this.#balance) throw new Error('ยอดเงินไม่เพียงพอ');
    this.#balance -= amount;
    console.log(`ถอนเงิน ${amount} บาท ยอดคงเหลือ: ${this.#balance} บาท`);
  }

  getBalance() {
    return this.#balance;
  }
}

const account = new BankAccount('สมชาย', 1000);
account.deposit(500);    // ฝากเงิน 500 บาท ยอดคงเหลือ: 1500 บาท
account.withdraw(200);   // ถอนเงิน 200 บาท ยอดคงเหลือ: 1300 บาท
console.log(account.getBalance()); // 1300

// account.#balance  // Error! ไม่สามารถเข้าถึงได้จากภายนอก
```

### 2. Inheritance (การสืบทอด)

คลาสลูกสามารถสืบทอด properties และ methods จากคลาสแม่ได้

```javascript
// Base class (คลาสแม่)
class Animal {
  constructor(name, sound) {
    this.name = name;
    this.sound = sound;
  }

  makeSound() {
    console.log(`${this.name} พูดว่า: ${this.sound}`);
  }

  describe() {
    return `ฉันคือ ${this.name}`;
  }
}

// Derived class (คลาสลูก) - สืบทอดจาก Animal
class Dog extends Animal {
  constructor(name, breed) {
    super(name, 'โฮ่ง');  // เรียก constructor ของ parent
    this.breed = breed;
  }

  fetch() {
    console.log(`${this.name} วิ่งไปเก็บลูกบอล!`);
  }
}

class Cat extends Animal {
  constructor(name) {
    super(name, 'เมี้ยว');
  }

  purr() {
    console.log(`${this.name} กรนเพลิน...`);
  }
}

const dog = new Dog('บัดดี้', 'โกลเด้น');
const cat = new Cat('วิสกี้');

dog.makeSound();  // บัดดี้ พูดว่า: โฮ่ง
dog.fetch();      // บัดดี้ วิ่งไปเก็บลูกบอล!
cat.makeSound();  // วิสกี้ พูดว่า: เมี้ยว
cat.purr();       // วิสกี้ กรนเพลิน...
```

### 3. Polymorphism (หลายรูปแบบ)

วัตถุต่างชนิดสามารถตอบสนองต่อ method เดียวกันได้ในแบบของตัวเอง

```javascript
class Shape {
  area() {
    throw new Error('Subclass ต้องนิยาม method area()');
  }

  toString() {
    return `รูปทรง: ${this.constructor.name}, พื้นที่: ${this.area().toFixed(2)}`;
  }
}

class Circle extends Shape {
  constructor(radius) {
    super();
    this.radius = radius;
  }

  area() {
    return Math.PI * this.radius ** 2;
  }
}

class Rectangle extends Shape {
  constructor(width, height) {
    super();
    this.width = width;
    this.height = height;
  }

  area() {
    return this.width * this.height;
  }
}

class Triangle extends Shape {
  constructor(base, height) {
    super();
    this.base = base;
    this.height = height;
  }

  area() {
    return (this.base * this.height) / 2;
  }
}

// Polymorphism - เรียก area() กับ object ต่างชนิด
const shapes = [
  new Circle(5),
  new Rectangle(4, 6),
  new Triangle(3, 8)
];

shapes.forEach(shape => {
  console.log(shape.toString());
});
// รูปทรง: Circle, พื้นที่: 78.54
// รูปทรง: Rectangle, พื้นที่: 24.00
// รูปทรง: Triangle, พื้นที่: 12.00
```

### 4. Abstraction (การซ่อนความซับซ้อน)

ซ่อนรายละเอียดการทำงานภายใน และแสดงเฉพาะส่วนที่จำเป็น

```javascript
class DatabaseConnection {
  #connection = null;
  #connectionString;

  constructor(connectionString) {
    this.#connectionString = connectionString;
  }

  // Public interface - ง่ายต่อการใช้งาน
  async query(sql) {
    await this.#ensureConnected();
    return this.#executeQuery(sql);
  }

  async disconnect() {
    if (this.#connection) {
      this.#connection = null;
      console.log('ยกเลิกการเชื่อมต่อแล้ว');
    }
  }

  // Private methods - ซ่อนรายละเอียด
  async #ensureConnected() {
    if (!this.#connection) {
      await this.#connect();
    }
  }

  async #connect() {
    // รายละเอียดการเชื่อมต่อที่ซับซ้อน
    console.log(`กำลังเชื่อมต่อกับ ${this.#connectionString}...`);
    this.#connection = { active: true };
  }

  async #executeQuery(sql) {
    console.log(`รัน query: ${sql}`);
    return { rows: [], rowCount: 0 };
  }
}
```

---

## Step 592: Class Syntax พื้นฐาน

```javascript
// การสร้าง Class
class Person {
  // Constructor method - เรียกอัตโนมัติเมื่อสร้าง object
  constructor(name, age) {
    this.name = name;
    this.age = age;
  }

  // Instance method
  greet() {
    return `สวัสดี ฉันชื่อ ${this.name} อายุ ${this.age} ปี`;
  }

  // Instance method อีกอัน
  haveBirthday() {
    this.age++;
    console.log(`สุขสันต์วันเกิด ${this.name}! อายุ ${this.age} ปีแล้ว`);
  }

  // toString override
  toString() {
    return `Person(${this.name}, ${this.age})`;
  }
}

// สร้าง instance
const person1 = new Person('อลิส', 25);
const person2 = new Person('บ็อบ', 30);

console.log(person1.greet());  // สวัสดี ฉันชื่อ อลิส อายุ 25 ปี
console.log(person2.greet());  // สวัสดี ฉันชื่อ บ็อบ อายุ 30 ปี

person1.haveBirthday();  // สุขสันต์วันเกิด อลิส! อายุ 26 ปีแล้ว

console.log(person1.toString());  // Person(อลิส, 26)
console.log(`${person1}`);        // Person(อลิส, 26) - ใช้ toString อัตโนมัติ
```

```javascript
// Class expressions (อีกรูปแบบหนึ่ง)
const Car = class {
  constructor(brand, model, year) {
    this.brand = brand;
    this.model = model;
    this.year = year;
  }

  describe() {
    return `${this.year} ${this.brand} ${this.model}`;
  }
};

// Named class expression
const Vehicle = class VehicleClass {
  constructor(type) {
    this.type = type;
  }
  // VehicleClass.name === 'VehicleClass'
};

const car = new Car('Toyota', 'Camry', 2023);
console.log(car.describe()); // 2023 Toyota Camry
```

---

## Step 593: Constructor Method

```javascript
class Product {
  constructor(name, price, category) {
    // Validation ใน constructor
    if (!name || typeof name !== 'string') {
      throw new Error('name ต้องเป็น string ที่ไม่ว่างเปล่า');
    }
    if (typeof price !== 'number' || price < 0) {
      throw new Error('price ต้องเป็นตัวเลขที่ไม่ติดลบ');
    }

    this.name = name;
    this.price = price;
    this.category = category || 'ทั่วไป';
    this.createdAt = new Date();
    this.id = Product.#generateId();
  }

  static #idCounter = 0;

  static #generateId() {
    return ++Product.#idCounter;
  }

  toString() {
    return `[${this.id}] ${this.name} - ราคา ${this.price} บาท (${this.category})`;
  }
}

const p1 = new Product('แล็ปท็อป', 25000, 'อิเล็กทรอนิกส์');
const p2 = new Product('เมาส์', 500);

console.log(p1.toString()); // [1] แล็ปท็อป - ราคา 25000 บาท (อิเล็กทรอนิกส์)
console.log(p2.toString()); // [2] เมาส์ - ราคา 500 บาท (ทั่วไป)

// Constructor validation
try {
  const bad = new Product('', -100);
} catch (e) {
  console.log(e.message); // name ต้องเป็น string ที่ไม่ว่างเปล่า
}
```

```javascript
// Constructor พร้อม default values
class Config {
  constructor({
    host = 'localhost',
    port = 3000,
    debug = false,
    timeout = 5000,
    maxRetries = 3
  } = {}) {
    this.host = host;
    this.port = port;
    this.debug = debug;
    this.timeout = timeout;
    this.maxRetries = maxRetries;
  }

  getConnectionString() {
    return `${this.host}:${this.port}`;
  }
}

const defaultConfig = new Config();
console.log(defaultConfig.getConnectionString()); // localhost:3000

const customConfig = new Config({ host: 'api.example.com', port: 8080, debug: true });
console.log(customConfig.getConnectionString()); // api.example.com:8080
```

---

## Step 594: Instance Methods

```javascript
class Stack {
  #items = [];

  // Instance methods
  push(item) {
    this.#items.push(item);
    return this; // method chaining
  }

  pop() {
    if (this.isEmpty()) throw new Error('Stack ว่างเปล่า');
    return this.#items.pop();
  }

  peek() {
    if (this.isEmpty()) throw new Error('Stack ว่างเปล่า');
    return this.#items[this.#items.length - 1];
  }

  isEmpty() {
    return this.#items.length === 0;
  }

  get size() {
    return this.#items.length;
  }

  clear() {
    this.#items = [];
    return this;
  }

  toArray() {
    return [...this.#items];
  }

  toString() {
    return `Stack(${this.#items.join(', ')})`;
  }
}

const stack = new Stack();
stack.push(1).push(2).push(3);  // method chaining
console.log(stack.toString());   // Stack(1, 2, 3)
console.log(stack.peek());       // 3
console.log(stack.pop());        // 3
console.log(stack.size);         // 2
console.log(stack.toArray());    // [1, 2]
```

```javascript
class Queue {
  #items = [];

  enqueue(item) {
    this.#items.push(item);
    return this;
  }

  dequeue() {
    if (this.isEmpty()) throw new Error('Queue ว่างเปล่า');
    return this.#items.shift();
  }

  front() {
    if (this.isEmpty()) throw new Error('Queue ว่างเปล่า');
    return this.#items[0];
  }

  isEmpty() {
    return this.#items.length === 0;
  }

  get size() {
    return this.#items.length;
  }

  // Iterator - ให้ใช้ for...of ได้
  [Symbol.iterator]() {
    return this.#items[Symbol.iterator]();
  }
}

const q = new Queue();
q.enqueue('งาน A').enqueue('งาน B').enqueue('งาน C');
console.log(q.front());    // งาน A
console.log(q.dequeue());  // งาน A
console.log(q.size);       // 2

for (const item of q) {
  console.log(item); // งาน B, งาน C
}
```

---

## Step 595: Instance Properties และ Class Fields

```javascript
class Counter {
  // Class field declarations (ES2022)
  count = 0;           // public field
  #step = 1;           // private field
  #history = [];

  constructor(initialValue = 0, step = 1) {
    this.count = initialValue;
    this.#step = step;
  }

  increment() {
    this.#history.push(this.count);
    this.count += this.#step;
    return this;
  }

  decrement() {
    this.#history.push(this.count);
    this.count -= this.#step;
    return this;
  }

  reset() {
    this.#history.push(this.count);
    this.count = 0;
    return this;
  }

  getHistory() {
    return [...this.#history];
  }

  undo() {
    if (this.#history.length === 0) return this;
    this.count = this.#history.pop();
    return this;
  }
}

const counter = new Counter(10, 5);
counter.increment().increment().decrement();
console.log(counter.count);        // 15
console.log(counter.getHistory()); // [10, 15]
counter.undo();
console.log(counter.count);        // 15 (undo ครั้งล่าสุด)
```

---

## Step 596: Static Methods และ Static Properties

Static methods/properties เป็นของคลาส ไม่ใช่ของ instance

```javascript
class MathHelper {
  // Static property
  static PI = 3.14159265358979;
  static E  = 2.71828182845905;

  // Static methods - เรียกผ่าน class ไม่ใช่ instance
  static add(a, b) { return a + b; }
  static subtract(a, b) { return a - b; }
  static multiply(a, b) { return a * b; }
  static divide(a, b) {
    if (b === 0) throw new Error('ไม่สามารถหารด้วยศูนย์ได้');
    return a / b;
  }

  static circleArea(radius) {
    return MathHelper.PI * radius ** 2;
  }

  static clamp(value, min, max) {
    return Math.min(Math.max(value, min), max);
  }

  static lerp(start, end, t) {
    return start + (end - start) * t;
  }
}

// เรียกผ่าน class โดยตรง
console.log(MathHelper.add(5, 3));         // 8
console.log(MathHelper.circleArea(5));     // 78.539...
console.log(MathHelper.clamp(15, 0, 10)); // 10
console.log(MathHelper.lerp(0, 100, 0.5));// 50
```

```javascript
class User {
  static #users = [];
  static #nextId = 1;

  constructor(name, email) {
    this.id = User.#nextId++;
    this.name = name;
    this.email = email;
    User.#users.push(this);
  }

  // Static factory methods
  static create(name, email) {
    return new User(name, email);
  }

  static findById(id) {
    return User.#users.find(u => u.id === id);
  }

  static findByEmail(email) {
    return User.#users.find(u => u.email === email);
  }

  static getAll() {
    return [...User.#users];
  }

  static get count() {
    return User.#users.length;
  }

  // Static utility
  static isValidEmail(email) {
    return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email);
  }

  toString() {
    return `User(${this.id}: ${this.name})`;
  }
}

const u1 = User.create('อลิส', 'alice@example.com');
const u2 = User.create('บ็อบ', 'bob@example.com');
const u3 = User.create('ชาร์ลี', 'charlie@example.com');

console.log(User.count); // 3
console.log(User.findById(2).toString()); // User(2: บ็อบ)
console.log(User.isValidEmail('test@test.com')); // true
console.log(User.getAll().map(u => u.name)); // ['อลิส', 'บ็อบ', 'ชาร์ลี']
```

---

## Step 597: Private Fields (#field)

Private fields ถูก introduced ใน ES2022 - ขึ้นต้นด้วย `#`

```javascript
class Temperature {
  #celsius;  // private field ต้องประกาศก่อนใช้

  constructor(celsius) {
    this.#celsius = celsius;
  }

  get celsius() {
    return this.#celsius;
  }

  set celsius(value) {
    if (typeof value !== 'number') throw new TypeError('ต้องเป็นตัวเลข');
    this.#celsius = value;
  }

  get fahrenheit() {
    return this.#celsius * 9/5 + 32;
  }

  set fahrenheit(value) {
    this.#celsius = (value - 32) * 5/9;
  }

  get kelvin() {
    return this.#celsius + 273.15;
  }

  toString() {
    return `${this.#celsius}°C / ${this.fahrenheit.toFixed(1)}°F / ${this.kelvin.toFixed(1)}K`;
  }
}

const temp = new Temperature(100);
console.log(temp.toString()); // 100°C / 212.0°F / 373.1K

temp.fahrenheit = 32;
console.log(temp.celsius); // 0

// temp.#celsius  // SyntaxError! ไม่สามารถเข้าถึงได้
```

```javascript
class LinkedList {
  #head = null;
  #size = 0;

  // Private Node class-like structure
  #createNode(value) {
    return { value, next: null };
  }

  append(value) {
    const node = this.#createNode(value);
    if (!this.#head) {
      this.#head = node;
    } else {
      let current = this.#head;
      while (current.next) {
        current = current.next;
      }
      current.next = node;
    }
    this.#size++;
    return this;
  }

  prepend(value) {
    const node = this.#createNode(value);
    node.next = this.#head;
    this.#head = node;
    this.#size++;
    return this;
  }

  remove(value) {
    if (!this.#head) return false;

    if (this.#head.value === value) {
      this.#head = this.#head.next;
      this.#size--;
      return true;
    }

    let current = this.#head;
    while (current.next) {
      if (current.next.value === value) {
        current.next = current.next.next;
        this.#size--;
        return true;
      }
      current = current.next;
    }
    return false;
  }

  get size() { return this.#size; }

  toArray() {
    const result = [];
    let current = this.#head;
    while (current) {
      result.push(current.value);
      current = current.next;
    }
    return result;
  }

  [Symbol.iterator]() {
    let current = this.#head;
    return {
      next() {
        if (current) {
          const value = current.value;
          current = current.next;
          return { value, done: false };
        }
        return { done: true };
      }
    };
  }
}

const list = new LinkedList();
list.append(1).append(2).append(3).prepend(0);
console.log(list.toArray()); // [0, 1, 2, 3]
list.remove(2);
console.log(list.toArray()); // [0, 1, 3]
console.log([...list]);      // [0, 1, 3]
```

---

## Step 598: Private Methods (#method)

```javascript
class PasswordManager {
  #passwords = new Map();
  #encryptionKey;

  constructor(masterKey) {
    this.#encryptionKey = masterKey;
  }

  // Private methods
  #encrypt(text) {
    // จำลองการเข้ารหัส (ในจริงควรใช้ crypto library)
    return btoa(text + this.#encryptionKey);
  }

  #decrypt(encrypted) {
    const decoded = atob(encrypted);
    return decoded.replace(this.#encryptionKey, '');
  }

  #validateKey(key) {
    if (!key || typeof key !== 'string') {
      throw new Error('Key ต้องเป็น string ที่ไม่ว่างเปล่า');
    }
  }

  // Public interface
  save(key, password) {
    this.#validateKey(key);
    this.#passwords.set(key, this.#encrypt(password));
    console.log(`บันทึก password สำหรับ ${key} แล้ว`);
  }

  get(key) {
    this.#validateKey(key);
    const encrypted = this.#passwords.get(key);
    if (!encrypted) return null;
    return this.#decrypt(encrypted);
  }

  delete(key) {
    return this.#passwords.delete(key);
  }

  has(key) {
    return this.#passwords.has(key);
  }

  get count() {
    return this.#passwords.size;
  }
}

const pm = new PasswordManager('mySecretKey');
pm.save('gmail', 'myGmailPass123');
pm.save('github', 'githubSecure456');

console.log(pm.get('gmail'));   // myGmailPass123
console.log(pm.has('github')); // true
console.log(pm.count);         // 2
```

---

## Step 599: Getters และ Setters

```javascript
class Circle {
  #radius;

  constructor(radius) {
    this.radius = radius; // ใช้ setter
  }

  get radius() {
    return this.#radius;
  }

  set radius(value) {
    if (typeof value !== 'number' || value <= 0) {
      throw new Error('radius ต้องเป็นตัวเลขที่มากกว่า 0');
    }
    this.#radius = value;
  }

  get diameter() {
    return this.#radius * 2;
  }

  set diameter(value) {
    this.radius = value / 2; // ใช้ radius setter
  }

  get area() {
    return Math.PI * this.#radius ** 2;
  }

  get circumference() {
    return 2 * Math.PI * this.#radius;
  }
}

const c = new Circle(5);
console.log(c.radius);       // 5
console.log(c.diameter);     // 10
console.log(c.area.toFixed(2)); // 78.54

c.diameter = 20;
console.log(c.radius); // 10

try {
  c.radius = -5; // Error!
} catch (e) {
  console.log(e.message); // radius ต้องเป็นตัวเลขที่มากกว่า 0
}
```

```javascript
class FullName {
  #firstName;
  #lastName;

  constructor(firstName, lastName) {
    this.firstName = firstName;
    this.lastName = lastName;
  }

  get firstName() { return this.#firstName; }
  set firstName(value) {
    this.#firstName = value.trim();
  }

  get lastName() { return this.#lastName; }
  set lastName(value) {
    this.#lastName = value.trim();
  }

  get fullName() {
    return `${this.#firstName} ${this.#lastName}`;
  }

  set fullName(value) {
    const parts = value.trim().split(' ');
    this.#firstName = parts[0];
    this.#lastName = parts.slice(1).join(' ');
  }

  get initials() {
    return `${this.#firstName[0]}.${this.#lastName[0]}.`;
  }
}

const name = new FullName('สมชาย', 'ใจดี');
console.log(name.fullName); // สมชาย ใจดี
console.log(name.initials); // ส.ใ.

name.fullName = 'นางสาว อรุณี สว่างฟ้า';
console.log(name.firstName); // นางสาว
console.log(name.lastName);  // อรุณี สว่างฟ้า
```

---

## Step 600: Class Inheritance ด้วย extends

```javascript
class Vehicle {
  #make;
  #model;
  #year;
  #fuelLevel = 100;

  constructor(make, model, year) {
    this.#make = make;
    this.#model = model;
    this.#year = year;
  }

  get make() { return this.#make; }
  get model() { return this.#model; }
  get year() { return this.#year; }
  get fuelLevel() { return this.#fuelLevel; }

  fuel(amount) {
    this.#fuelLevel = Math.min(100, this.#fuelLevel + amount);
    return this;
  }

  drive(km) {
    const consumption = this.#calculateConsumption(km);
    if (consumption > this.#fuelLevel) {
      throw new Error('น้ำมันไม่เพียงพอ!');
    }
    this.#fuelLevel -= consumption;
    console.log(`ขับไป ${km} กม. เหลือน้ำมัน ${this.#fuelLevel.toFixed(1)}%`);
    return this;
  }

  // Protected-like method ที่ subclass สามารถ override ได้
  #calculateConsumption(km) {
    return km * 0.1; // default: 10 กม./ลิตร
  }

  toString() {
    return `${this.#year} ${this.#make} ${this.#model}`;
  }
}

class Car extends Vehicle {
  #doors;
  #passengers = 0;

  constructor(make, model, year, doors = 4) {
    super(make, model, year);
    this.#doors = doors;
  }

  get doors() { return this.#doors; }

  loadPassengers(count) {
    this.#passengers = count;
    return this;
  }

  describe() {
    return `${this.toString()} - ${this.#doors} ประตู, ${this.#passengers} คน`;
  }
}

class Truck extends Vehicle {
  #payload; // น้ำหนักบรรทุก (กก.)
  #loadWeight = 0;

  constructor(make, model, year, payload) {
    super(make, model, year);
    this.#payload = payload;
  }

  load(weight) {
    if (weight > this.#payload) {
      throw new Error(`น้ำหนักเกิน! สูงสุด ${this.#payload} กก.`);
    }
    this.#loadWeight = weight;
    return this;
  }

  describe() {
    return `${this.toString()} - บรรทุก ${this.#loadWeight}/${this.#payload} กก.`;
  }
}

const car = new Car('Toyota', 'Camry', 2023);
car.loadPassengers(4).fuel(50);
console.log(car.describe()); // 2023 Toyota Camry - 4 ประตู, 4 คน

const truck = new Truck('Isuzu', 'D-Max', 2022, 1000);
truck.load(800);
console.log(truck.describe()); // 2022 Isuzu D-Max - บรรทุก 800/1000 กก.
```

---

## Step 601: super() ใน Constructor

```javascript
class Animal {
  constructor(name, age, sound) {
    this.name = name;
    this.age = age;
    this.sound = sound;
    this.alive = true;
  }

  makeSound() {
    if (!this.alive) return console.log(`${this.name} is no longer with us`);
    console.log(`${this.name}: ${this.sound}!`);
  }
}

class Dog extends Animal {
  constructor(name, age, breed) {
    // super() ต้องเรียกก่อนใช้ this ใน derived class
    super(name, age, 'โฮ่งโฮ่ง');
    this.breed = breed;
    this.trained = false;
  }

  train() {
    this.trained = true;
    console.log(`${this.name} ได้รับการฝึกแล้ว!`);
    return this;
  }

  describe() {
    return `${this.name} (${this.breed}) อายุ ${this.age} ปี ${this.trained ? '- ผ่านการฝึกแล้ว' : ''}`;
  }
}

class GuideDog extends Dog {
  constructor(name, age, breed, handler) {
    super(name, age, breed); // เรียก Dog constructor
    this.handler = handler;
    this.train(); // ฝึกอัตโนมัติ
  }

  guide() {
    console.log(`${this.name} กำลังนำทาง ${this.handler}`);
  }
}

const dog = new Dog('เร็กซ์', 3, 'เยอรมันเชพเพิร์ด');
dog.makeSound(); // เร็กซ์: โฮ่งโฮ่ง!
dog.train();     // เร็กซ์ ได้รับการฝึกแล้ว!

const guide = new GuideDog('ลัคกี้', 4, 'แล็บราดอร์', 'นายสมชาย');
guide.guide(); // ลัคกี้ กำลังนำทาง นายสมชาย
console.log(guide.describe()); // ลัคกี้ (แล็บราดอร์) อายุ 4 ปี - ผ่านการฝึกแล้ว
```

---

## Step 602: super.method() - เรียก Parent Methods

```javascript
class Logger {
  log(message) {
    console.log(`[LOG] ${new Date().toISOString()}: ${message}`);
  }

  error(message) {
    console.error(`[ERROR] ${message}`);
  }
}

class TimedLogger extends Logger {
  #startTime;

  constructor() {
    super();
    this.#startTime = Date.now();
  }

  log(message) {
    const elapsed = Date.now() - this.#startTime;
    super.log(`[+${elapsed}ms] ${message}`); // เรียก parent method
  }

  error(message) {
    const elapsed = Date.now() - this.#startTime;
    super.error(`[+${elapsed}ms] ${message}`);
  }
}

class PrefixLogger extends TimedLogger {
  #prefix;

  constructor(prefix) {
    super();
    this.#prefix = prefix;
  }

  log(message) {
    super.log(`${this.#prefix} ${message}`); // เรียก TimedLogger.log
  }
}

const logger = new PrefixLogger('[API]');
logger.log('Server started');  // [LOG] 2024-...: [+2ms] [API] Server started
logger.error('Connection failed'); // [ERROR] [+5ms] Connection failed
```

```javascript
class Shape {
  constructor(color = 'black') {
    this.color = color;
  }

  toString() {
    return `Shape(color: ${this.color})`;
  }

  describe() {
    return `สีของรูปทรง: ${this.color}`;
  }
}

class ColoredCircle extends Shape {
  constructor(radius, color) {
    super(color);
    this.radius = radius;
  }

  toString() {
    const parentStr = super.toString(); // เรียก Shape.toString()
    return `Circle(radius: ${this.radius}) extends ${parentStr}`;
  }

  describe() {
    const base = super.describe(); // เรียก Shape.describe()
    return `${base}, รัศมี: ${this.radius}`;
  }
}

const cc = new ColoredCircle(5, 'red');
console.log(cc.toString()); // Circle(radius: 5) extends Shape(color: red)
console.log(cc.describe()); // สีของรูปทรง: red, รัศมี: 5
```

---

## Step 603: Method Overriding

```javascript
class PaymentProcessor {
  processPayment(amount) {
    console.log(`กำลังดำเนินการชำระเงิน ${amount} บาท...`);
    return { success: true, amount };
  }

  refund(transactionId) {
    console.log(`คืนเงิน transaction: ${transactionId}`);
    return { success: true, transactionId };
  }

  toString() {
    return `PaymentProcessor`;
  }
}

class CreditCardProcessor extends PaymentProcessor {
  #cardNumber;
  #expiryDate;

  constructor(cardNumber, expiryDate) {
    super();
    this.#cardNumber = cardNumber;
    this.#expiryDate = expiryDate;
  }

  // Override processPayment
  processPayment(amount) {
    console.log(`ตรวจสอบบัตรเครดิต ${this.#maskCard()}...`);
    if (!this.#isValid()) throw new Error('บัตรหมดอายุ');
    return super.processPayment(amount); // เรียก parent method ด้วย
  }

  #maskCard() {
    return `****-****-****-${this.#cardNumber.slice(-4)}`;
  }

  #isValid() {
    const [month, year] = this.#expiryDate.split('/').map(Number);
    const now = new Date();
    return (year + 2000) > now.getFullYear() ||
           ((year + 2000) === now.getFullYear() && month >= now.getMonth() + 1);
  }

  toString() {
    return `CreditCardProcessor(${this.#maskCard()})`;
  }
}

class PayPalProcessor extends PaymentProcessor {
  #email;

  constructor(email) {
    super();
    this.#email = email;
  }

  processPayment(amount) {
    console.log(`เชื่อมต่อ PayPal account: ${this.#email}`);
    console.log(`ขออนุมัติการชำระเงิน...`);
    return super.processPayment(amount);
  }

  toString() {
    return `PayPalProcessor(${this.#email})`;
  }
}

// Polymorphism ในการทำงาน
function checkout(processor, amount) {
  console.log(`\n=== Checkout ด้วย ${processor.toString()} ===`);
  try {
    const result = processor.processPayment(amount);
    console.log(`สำเร็จ! ราคา ${result.amount} บาท`);
  } catch (e) {
    console.log(`ล้มเหลว: ${e.message}`);
  }
}

const cc = new CreditCardProcessor('4111111111111111', '12/25');
const pp = new PayPalProcessor('user@example.com');

checkout(cc, 500);
checkout(pp, 1000);
```

---

## Step 604: instanceof Operator

```javascript
class Animal {}
class Dog extends Animal {}
class GoldenRetriever extends Dog {}

const dog = new GoldenRetriever();

console.log(dog instanceof GoldenRetriever); // true
console.log(dog instanceof Dog);             // true
console.log(dog instanceof Animal);          // true
console.log(dog instanceof Object);          // true
console.log(dog instanceof Array);           // false

// instanceof ใช้ตรวจสอบ type
function processAnimal(animal) {
  if (animal instanceof Dog) {
    console.log('เป็นสุนัข - สามารถฝึกได้');
  } else if (animal instanceof Animal) {
    console.log('เป็นสัตว์ทั่วไป');
  } else {
    console.log('ไม่ใช่สัตว์');
  }
}

processAnimal(new GoldenRetriever()); // เป็นสุนัข - สามารถฝึกได้
processAnimal(new Animal());          // เป็นสัตว์ทั่วไป
processAnimal({ name: 'ไม่ใช่สัตว์' }); // ไม่ใช่สัตว์
```

```javascript
// instanceof กับ Symbols
class MyArray {
  static [Symbol.hasInstance](instance) {
    return Array.isArray(instance);
  }
}

console.log([] instanceof MyArray); // true
console.log({} instanceof MyArray); // false

// ตรวจสอบ constructor
class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }
}

const p = new Point(1, 2);
console.log(p.constructor === Point);         // true
console.log(p.constructor.name);              // 'Point'
console.log(Object.getPrototypeOf(p) === Point.prototype); // true
```

---

## Step 605: Abstract Class Pattern

JavaScript ไม่มี abstract class โดยตรง แต่เราสร้างได้ด้วย pattern:

```javascript
class AbstractShape {
  constructor() {
    if (new.target === AbstractShape) {
      throw new Error('ไม่สามารถสร้าง AbstractShape โดยตรงได้');
    }
  }

  // Abstract methods - ต้อง implement ใน subclass
  area() {
    throw new Error(`${this.constructor.name} ต้อง implement method area()`);
  }

  perimeter() {
    throw new Error(`${this.constructor.name} ต้อง implement method perimeter()`);
  }

  // Concrete method - ใช้ได้เลย
  describe() {
    return `${this.constructor.name}: พื้นที่=${this.area().toFixed(2)}, เส้นรอบวง=${this.perimeter().toFixed(2)}`;
  }

  compareTo(other) {
    return this.area() - other.area();
  }
}

class Rectangle extends AbstractShape {
  constructor(width, height) {
    super();
    this.width = width;
    this.height = height;
  }

  area() { return this.width * this.height; }
  perimeter() { return 2 * (this.width + this.height); }
}

class Circle extends AbstractShape {
  constructor(radius) {
    super();
    this.radius = radius;
  }

  area() { return Math.PI * this.radius ** 2; }
  perimeter() { return 2 * Math.PI * this.radius; }
}

// ทดสอบ
try {
  const abs = new AbstractShape(); // Error!
} catch (e) {
  console.log(e.message); // ไม่สามารถสร้าง AbstractShape โดยตรงได้
}

const rect = new Rectangle(4, 6);
const circ = new Circle(3);

console.log(rect.describe()); // Rectangle: พื้นที่=24.00, เส้นรอบวง=20.00
console.log(circ.describe()); // Circle: พื้นที่=28.27, เส้นรอบวง=18.85

// เรียงตามพื้นที่
const shapes = [circ, rect];
shapes.sort((a, b) => a.compareTo(b));
shapes.forEach(s => console.log(s.describe()));
```

---

## Step 606: Mixin Pattern

Mixin ช่วยแก้ปัญหา multiple inheritance ที่ JavaScript ไม่รองรับ

```javascript
// Mixin functions
const Serializable = (Base) => class extends Base {
  serialize() {
    return JSON.stringify(this);
  }

  static deserialize(json) {
    const data = JSON.parse(json);
    return Object.assign(new this(), data);
  }

  toJSON() {
    return Object.fromEntries(
      Object.entries(this).filter(([key]) => !key.startsWith('_'))
    );
  }
};

const Timestamped = (Base) => class extends Base {
  constructor(...args) {
    super(...args);
    this.createdAt = new Date().toISOString();
    this.updatedAt = new Date().toISOString();
  }

  touch() {
    this.updatedAt = new Date().toISOString();
    return this;
  }
};

const Validatable = (Base) => class extends Base {
  validate() {
    const rules = this.constructor.validationRules || {};
    const errors = [];

    for (const [field, rule] of Object.entries(rules)) {
      const value = this[field];
      if (rule.required && (value === undefined || value === null || value === '')) {
        errors.push(`${field} เป็นค่าที่จำเป็น`);
      }
      if (rule.minLength && value && value.length < rule.minLength) {
        errors.push(`${field} ต้องมีอย่างน้อย ${rule.minLength} ตัวอักษร`);
      }
      if (rule.maxLength && value && value.length > rule.maxLength) {
        errors.push(`${field} ต้องไม่เกิน ${rule.maxLength} ตัวอักษร`);
      }
    }

    return { valid: errors.length === 0, errors };
  }
};

// ใช้ Mixins
class User extends Serializable(Timestamped(Validatable(class {}))) {
  static validationRules = {
    name: { required: true, minLength: 2, maxLength: 50 },
    email: { required: true }
  };

  constructor(name, email) {
    super();
    this.name = name;
    this.email = email;
  }
}

const user = new User('สมชาย', 'somchai@test.com');
console.log(user.validate()); // { valid: true, errors: [] }

const json = user.serialize();
console.log(json); // JSON string

const invalid = new User('', 'test@test.com');
console.log(invalid.validate()); // { valid: false, errors: ['name เป็นค่าที่จำเป็น'] }
```

---

## Step 607: Factory Functions vs Classes

```javascript
// Factory Function approach
function createPerson(name, age) {
  // private state
  let _age = age;
  const _history = [];

  return {
    get name() { return name; },
    get age() { return _age; },

    haveBirthday() {
      _history.push(_age);
      _age++;
    },

    getHistory() { return [..._history]; },

    toString() { return `Person(${name}, ${_age})`; }
  };
}

// Class approach
class PersonClass {
  #age;
  #history = [];
  #name;

  constructor(name, age) {
    this.#name = name;
    this.#age = age;
  }

  get name() { return this.#name; }
  get age() { return this.#age; }

  haveBirthday() {
    this.#history.push(this.#age);
    this.#age++;
  }

  getHistory() { return [...this.#history]; }
  toString() { return `Person(${this.#name}, ${this.#age})`; }
}

// เปรียบเทียบการใช้งาน
const factoryPerson = createPerson('อลิส', 25);
const classPerson = new PersonClass('บ็อบ', 30);

// ทั้งสองทำงานเหมือนกัน
factoryPerson.haveBirthday();
classPerson.haveBirthday();

console.log(factoryPerson.age); // 26
console.log(classPerson.age);   // 31

// ความแตกต่าง
console.log(factoryPerson instanceof Object);     // true
console.log(classPerson instanceof PersonClass);   // true

// Factory: แต่ละ instance มี methods ของตัวเอง (memory)
// Class: methods อยู่ใน prototype (ประหยัด memory)
console.log(factoryPerson.haveBirthday === createPerson('x', 0).haveBirthday); // false!
const p2 = new PersonClass('y', 0);
console.log(classPerson.haveBirthday === p2.haveBirthday); // true! shared prototype
```

---

## Step 608: Real-World OOP Example - Shopping Cart

```javascript
class Product {
  #id;
  #name;
  #price;
  #stock;

  constructor(id, name, price, stock) {
    this.#id = id;
    this.#name = name;
    this.#price = price;
    this.#stock = stock;
  }

  get id() { return this.#id; }
  get name() { return this.#name; }
  get price() { return this.#price; }
  get stock() { return this.#stock; }

  reduceStock(qty) {
    if (qty > this.#stock) throw new Error(`สินค้า ${this.#name} ไม่เพียงพอ`);
    this.#stock -= qty;
  }

  toString() {
    return `${this.#name} (฿${this.#price})`;
  }
}

class CartItem {
  #product;
  #quantity;

  constructor(product, quantity = 1) {
    this.#product = product;
    this.#quantity = quantity;
  }

  get product() { return this.#product; }
  get quantity() { return this.#quantity; }
  get subtotal() { return this.#product.price * this.#quantity; }

  increaseQty(amount = 1) {
    this.#quantity += amount;
    return this;
  }

  decreaseQty(amount = 1) {
    this.#quantity = Math.max(0, this.#quantity - amount);
    return this;
  }
}

class ShoppingCart {
  #items = new Map();
  #discount = 0;

  addItem(product, quantity = 1) {
    if (this.#items.has(product.id)) {
      this.#items.get(product.id).increaseQty(quantity);
    } else {
      this.#items.set(product.id, new CartItem(product, quantity));
    }
    return this;
  }

  removeItem(productId) {
    this.#items.delete(productId);
    return this;
  }

  applyDiscount(percent) {
    if (percent < 0 || percent > 100) throw new Error('ส่วนลดต้องอยู่ระหว่าง 0-100%');
    this.#discount = percent;
    return this;
  }

  get subtotal() {
    let total = 0;
    for (const item of this.#items.values()) {
      total += item.subtotal;
    }
    return total;
  }

  get discountAmount() {
    return this.subtotal * (this.#discount / 100);
  }

  get total() {
    return this.subtotal - this.discountAmount;
  }

  get itemCount() {
    let count = 0;
    for (const item of this.#items.values()) {
      count += item.quantity;
    }
    return count;
  }

  summary() {
    console.log('=== สรุปตะกร้าสินค้า ===');
    for (const item of this.#items.values()) {
      console.log(`  ${item.product.name} x${item.quantity} = ฿${item.subtotal}`);
    }
    console.log(`ราคาก่อนส่วนลด: ฿${this.subtotal}`);
    if (this.#discount > 0) {
      console.log(`ส่วนลด ${this.#discount}%: -฿${this.discountAmount}`);
    }
    console.log(`รวมทั้งหมด: ฿${this.total}`);
  }

  checkout() {
    // ตรวจสอบ stock
    for (const item of this.#items.values()) {
      item.product.reduceStock(item.quantity);
    }
    console.log('ชำระเงินสำเร็จ!');
    this.#items.clear();
    return this.total;
  }
}

// ใช้งาน
const laptop = new Product(1, 'แล็ปท็อป', 25000, 10);
const mouse  = new Product(2, 'เมาส์ไร้สาย', 800, 50);
const keyboard = new Product(3, 'คีย์บอร์ด', 1500, 30);

const cart = new ShoppingCart();
cart
  .addItem(laptop)
  .addItem(mouse, 2)
  .addItem(keyboard)
  .applyDiscount(10);

cart.summary();
/*
=== สรุปตะกร้าสินค้า ===
  แล็ปท็อป x1 = ฿25000
  เมาส์ไร้สาย x2 = ฿1600
  คีย์บอร์ด x1 = ฿1500
ราคาก่อนส่วนลด: ฿28100
ส่วนลด 10%: -฿2810
รวมทั้งหมด: ฿25290
*/
```

---

## Step 609: Event System with OOP

```javascript
class EventEmitter {
  #listeners = new Map();

  on(event, listener) {
    if (!this.#listeners.has(event)) {
      this.#listeners.set(event, []);
    }
    this.#listeners.get(event).push(listener);
    return this; // chaining
  }

  off(event, listener) {
    if (!this.#listeners.has(event)) return this;
    const list = this.#listeners.get(event);
    const index = list.indexOf(listener);
    if (index > -1) list.splice(index, 1);
    return this;
  }

  once(event, listener) {
    const wrapper = (...args) => {
      listener(...args);
      this.off(event, wrapper);
    };
    return this.on(event, wrapper);
  }

  emit(event, ...args) {
    if (!this.#listeners.has(event)) return false;
    this.#listeners.get(event).forEach(listener => listener(...args));
    return true;
  }

  listenerCount(event) {
    return this.#listeners.get(event)?.length ?? 0;
  }
}

class Store extends EventEmitter {
  #state;

  constructor(initialState) {
    super();
    this.#state = initialState;
  }

  get state() {
    return { ...this.#state };
  }

  setState(newState) {
    const prevState = this.#state;
    this.#state = { ...this.#state, ...newState };
    this.emit('change', this.#state, prevState);
    return this;
  }

  subscribe(listener) {
    this.on('change', listener);
    return () => this.off('change', listener); // unsubscribe function
  }
}

const store = new Store({ count: 0, user: null });

const unsubscribe = store.subscribe((newState, prevState) => {
  console.log(`State เปลี่ยน: count ${prevState.count} -> ${newState.count}`);
});

store.setState({ count: 1 }); // State เปลี่ยน: count 0 -> 1
store.setState({ count: 2 }); // State เปลี่ยน: count 1 -> 2

unsubscribe(); // หยุด listen

store.setState({ count: 3 }); // ไม่มีการแจ้ง (unsubscribed แล้ว)
console.log(store.state.count); // 3
```

---

## Step 610: Design Patterns ด้วย Classes

### Singleton Pattern

```javascript
class AppConfig {
  static #instance = null;

  #settings = {};

  constructor() {
    if (AppConfig.#instance) {
      return AppConfig.#instance;
    }
    AppConfig.#instance = this;
  }

  static getInstance() {
    if (!AppConfig.#instance) {
      new AppConfig();
    }
    return AppConfig.#instance;
  }

  set(key, value) {
    this.#settings[key] = value;
    return this;
  }

  get(key) {
    return this.#settings[key];
  }

  getAll() {
    return { ...this.#settings };
  }
}

const config1 = AppConfig.getInstance();
const config2 = AppConfig.getInstance();

config1.set('theme', 'dark').set('language', 'th');
console.log(config2.get('theme'));    // dark
console.log(config1 === config2);     // true - same instance
```

### Observer Pattern

```javascript
class Observable {
  #observers = new Set();

  subscribe(observer) {
    this.#observers.add(observer);
    return () => this.#observers.delete(observer); // cleanup
  }

  notify(data) {
    this.#observers.forEach(obs => obs(data));
  }
}

class DataSource extends Observable {
  #data = [];

  add(item) {
    this.#data.push(item);
    this.notify({ type: 'ADD', item, data: [...this.#data] });
    return this;
  }

  remove(index) {
    const item = this.#data.splice(index, 1)[0];
    this.notify({ type: 'REMOVE', item, data: [...this.#data] });
    return this;
  }

  get data() { return [...this.#data]; }
}

const source = new DataSource();

const cleanup = source.subscribe(({ type, item }) => {
  console.log(`[Observer] ${type}: ${item}`);
});

source.add('แอปเปิล').add('กล้วย').remove(0);
// [Observer] ADD: แอปเปิล
// [Observer] ADD: กล้วย
// [Observer] REMOVE: แอปเปิล

cleanup(); // unsubscribe

source.add('ส้ม'); // ไม่มี notification
console.log(source.data); // ['กล้วย', 'ส้ม']
```

### Builder Pattern

```javascript
class QueryBuilder {
  #table = '';
  #conditions = [];
  #orderBy = [];
  #limitValue = null;
  #offsetValue = 0;
  #columns = ['*'];

  from(table) {
    this.#table = table;
    return this;
  }

  select(...columns) {
    this.#columns = columns;
    return this;
  }

  where(condition) {
    this.#conditions.push(condition);
    return this;
  }

  order(column, direction = 'ASC') {
    this.#orderBy.push(`${column} ${direction}`);
    return this;
  }

  limit(value) {
    this.#limitValue = value;
    return this;
  }

  offset(value) {
    this.#offsetValue = value;
    return this;
  }

  build() {
    if (!this.#table) throw new Error('ต้องระบุ table ด้วย .from()');

    let query = `SELECT ${this.#columns.join(', ')} FROM ${this.#table}`;

    if (this.#conditions.length > 0) {
      query += ` WHERE ${this.#conditions.join(' AND ')}`;
    }

    if (this.#orderBy.length > 0) {
      query += ` ORDER BY ${this.#orderBy.join(', ')}`;
    }

    if (this.#limitValue !== null) {
      query += ` LIMIT ${this.#limitValue}`;
    }

    if (this.#offsetValue > 0) {
      query += ` OFFSET ${this.#offsetValue}`;
    }

    return query;
  }
}

const query = new QueryBuilder()
  .from('users')
  .select('id', 'name', 'email')
  .where('age >= 18')
  .where('active = true')
  .order('name')
  .limit(10)
  .offset(20)
  .build();

console.log(query);
// SELECT id, name, email FROM users WHERE age >= 18 AND active = true ORDER BY name ASC LIMIT 10 OFFSET 20
```

---

## แบบฝึกหัด (Exercises)

### แบบฝึกหัดที่ 1: Library Management System
สร้างระบบจัดการห้องสมุด:

```javascript
// โจทย์: สร้าง classes เหล่านี้
// - Book: id, title, author, isbn, available
// - Member: id, name, email, borrowedBooks[]
// - Library: จัดการหนังสือและสมาชิก
//   - addBook(book)
//   - addMember(member)
//   - borrowBook(memberId, bookId)
//   - returnBook(memberId, bookId)
//   - searchBooks(query)
//   - getBorrowedBooks(memberId)

class Book {
  #id;
  #title;
  #author;
  #isbn;
  #available = true;

  constructor(id, title, author, isbn) {
    this.#id = id;
    this.#title = title;
    this.#author = author;
    this.#isbn = isbn;
  }

  get id() { return this.#id; }
  get title() { return this.#title; }
  get author() { return this.#author; }
  get isbn() { return this.#isbn; }
  get available() { return this.#available; }

  borrow() {
    if (!this.#available) throw new Error(`หนังสือ "${this.#title}" ถูกยืมไปแล้ว`);
    this.#available = false;
  }

  return_() {
    this.#available = true;
  }

  toString() {
    return `"${this.#title}" โดย ${this.#author} [${this.#available ? 'ว่าง' : 'ถูกยืม'}]`;
  }
}

class Member {
  #id;
  #name;
  #email;
  #borrowedBooks = [];

  constructor(id, name, email) {
    this.#id = id;
    this.#name = name;
    this.#email = email;
  }

  get id() { return this.#id; }
  get name() { return this.#name; }
  get borrowedBooks() { return [...this.#borrowedBooks]; }

  borrow(book) {
    if (this.#borrowedBooks.length >= 3) {
      throw new Error(`${this.#name} ยืมหนังสือได้สูงสุด 3 เล่ม`);
    }
    book.borrow();
    this.#borrowedBooks.push(book);
  }

  return_(book) {
    const index = this.#borrowedBooks.findIndex(b => b.id === book.id);
    if (index === -1) throw new Error('ไม่พบหนังสือในรายการที่ยืม');
    this.#borrowedBooks.splice(index, 1);
    book.return_();
  }
}

class Library {
  #books = new Map();
  #members = new Map();

  addBook(book) {
    this.#books.set(book.id, book);
    return this;
  }

  addMember(member) {
    this.#members.set(member.id, member);
    return this;
  }

  borrowBook(memberId, bookId) {
    const member = this.#members.get(memberId);
    const book = this.#books.get(bookId);

    if (!member) throw new Error('ไม่พบสมาชิก');
    if (!book) throw new Error('ไม่พบหนังสือ');

    member.borrow(book);
    console.log(`${member.name} ยืม "${book.title}" สำเร็จ`);
  }

  returnBook(memberId, bookId) {
    const member = this.#members.get(memberId);
    const book = this.#books.get(bookId);

    if (!member) throw new Error('ไม่พบสมาชิก');
    if (!book) throw new Error('ไม่พบหนังสือ');

    member.return_(book);
    console.log(`${member.name} คืน "${book.title}" สำเร็จ`);
  }

  searchBooks(query) {
    const q = query.toLowerCase();
    return Array.from(this.#books.values()).filter(b =>
      b.title.toLowerCase().includes(q) ||
      b.author.toLowerCase().includes(q)
    );
  }

  getAvailableBooks() {
    return Array.from(this.#books.values()).filter(b => b.available);
  }
}

// ทดสอบ
const library = new Library();

library
  .addBook(new Book(1, 'JavaScript: The Good Parts', 'Douglas Crockford', '978-0596517748'))
  .addBook(new Book(2, 'Clean Code', 'Robert C. Martin', '978-0132350884'))
  .addBook(new Book(3, 'You Don\'t Know JS', 'Kyle Simpson', '978-1491924464'));

library
  .addMember(new Member(1, 'สมชาย', 'somchai@example.com'))
  .addMember(new Member(2, 'สมหญิง', 'somying@example.com'));

library.borrowBook(1, 1); // สมชาย ยืม JavaScript: The Good Parts สำเร็จ
library.borrowBook(1, 2); // สมชาย ยืม Clean Code สำเร็จ

console.log('หนังสือที่ว่าง:');
library.getAvailableBooks().forEach(b => console.log(' -', b.toString()));

const results = library.searchBooks('javascript');
console.log('\nค้นหา "javascript":');
results.forEach(b => console.log(' -', b.toString()));
```

### แบบฝึกหัดที่ 2: RPG Character System
```javascript
// สร้างระบบ RPG:
// - Character (base class): name, hp, maxHp, level, xp
// - Warrior extends Character: armor, battleCry()
// - Mage extends Character: mana, castSpell(spell)
// - Archer extends Character: arrows, shoot(target)
// - มี Mixin ที่เพิ่ม skill ให้ character

// ลองทำเอง!
```

### แบบฝึกหัดที่ 3: Promise Implementation
```javascript
// สร้าง SimplePromise class ที่ทำงานคล้าย Promise จริง:
// - constructor รับ executor function
// - then(onFulfilled, onRejected)
// - catch(onRejected)  
// - finally(onFinally)
// - static resolve(value)
// - static reject(reason)

// ลองทำเอง!
```

---

## สรุป

ใน Part 31 เราได้เรียนรู้:

1. **Four Pillars of OOP**: Encapsulation, Inheritance, Polymorphism, Abstraction
2. **Class Syntax**: การสร้างและใช้งาน class ใน JavaScript
3. **Constructor**: การกำหนดค่าเริ่มต้น
4. **Instance Methods/Properties**: methods และ properties ของแต่ละ instance
5. **Static Methods/Properties**: shared ระหว่าง instance ทั้งหมด
6. **Private Fields (#)**: การซ่อนข้อมูล
7. **Getters/Setters**: การควบคุมการเข้าถึง property
8. **Inheritance**: extends, super()
9. **Method Overriding**: การ override methods
10. **instanceof**: การตรวจสอบ type
11. **Abstract Class Pattern**: pattern สำหรับ abstract class
12. **Mixin Pattern**: multiple inheritance ใน JavaScript
13. **Design Patterns**: Singleton, Observer, Builder

ใน Part 32 เราจะเรียนรู้เรื่อง Prototype Chain ซึ่งเป็นพื้นฐานของ OOP ใน JavaScript!
