# Part 51: Design Patterns - Creational Patterns (Steps 991-1010)

## การออกแบบ Patterns สำหรับการสร้างวัตถุ

**คำอธิบาย:** ในส่วนนี้เราจะเรียนรู้เกี่ยวกับ Design Patterns ซึ่งเป็นแนวทางการแก้ปัญหาที่ผ่านการพิสูจน์แล้วในการพัฒนาซอฟต์แวร์ โดยเฉพาะ Creational Patterns ที่เกี่ยวข้องกับการสร้างวัตถุ (Objects)

---

## Step 991: Design Patterns คืออะไร?

**Design Patterns** คือแนวทางการแก้ปัญหาที่ถูกนำมาใช้ซ้ำได้ (reusable solutions) สำหรับปัญหาที่พบบ่อยในการออกแบบซอฟต์แวร์ ไม่ใช่โค้ดที่พร้อมใช้ แต่เป็น "แม่แบบ" ที่บอกวิธีการแก้ปัญหา

### Gang of Four (GoF)

ในปี 1994 นักพัฒนา 4 คน (Erich Gamma, Richard Helm, Ralph Johnson, John Vlissides) ได้เขียนหนังสือ "Design Patterns: Elements of Reusable Object-Oriented Software" ซึ่งรวบรวม 23 patterns แบ่งเป็น 3 หมวดหลัก:

1. **Creational Patterns** - เกี่ยวกับการสร้างวัตถุ
2. **Structural Patterns** - เกี่ยวกับโครงสร้างของวัตถุ
3. **Behavioral Patterns** - เกี่ยวกับพฤติกรรมของวัตถุ

```javascript
// ทำไมต้องใช้ Design Patterns?
// ปัญหา: การสร้างวัตถุโดยตรงทำให้โค้ดยุ่งเหยิงและยากต่อการแก้ไข

// แบบไม่ดี - สร้างวัตถุโดยตรง
class DatabaseConnection {
  constructor(host, port, username, password, database) {
    this.host = host;
    this.port = port;
    this.username = username;
    this.password = password;
    this.database = database;
    this.connection = null;
  }
  
  connect() {
    // การเชื่อมต่อฐานข้อมูล
    this.connection = `Connected to ${this.host}:${this.port}/${this.database}`;
    return this.connection;
  }
}

// หลายที่ในโค้ดสร้าง instance ใหม่
const db1 = new DatabaseConnection('localhost', 5432, 'admin', 'pass', 'mydb');
const db2 = new DatabaseConnection('localhost', 5432, 'admin', 'pass', 'mydb');
const db3 = new DatabaseConnection('localhost', 5432, 'admin', 'pass', 'mydb');

// db1 !== db2 !== db3 - ทรัพยากรสูญเปล่า!
console.log(db1 === db2); // false - ปัญหา!
```

---

## Step 992: Creational Patterns Overview

Creational Patterns มี 6 แบบหลักใน JavaScript:

| Pattern | วัตถุประสงค์ |
|---------|------------|
| Singleton | สร้าง instance เดียวในระบบ |
| Factory Method | สร้างวัตถุโดยไม่ระบุคลาสที่แน่นอน |
| Abstract Factory | สร้างกลุ่มของวัตถุที่เกี่ยวข้องกัน |
| Builder | สร้างวัตถุซับซ้อนทีละขั้นตอน |
| Prototype | สร้างวัตถุใหม่โดยการ clone |
| Object Pool | จัดการ pool ของวัตถุที่ใช้ซ้ำได้ |

```javascript
// ภาพรวมของ Creational Patterns
console.log('=== Creational Patterns ===');

// 1. Singleton - มีแค่ 1 instance
// 2. Factory - ไม่ต้องรู้ว่า new อะไร
// 3. Builder - สร้างทีละขั้น
// 4. Prototype - clone จากของที่มีอยู่

// แต่ละแบบแก้ปัญหาต่างกัน
const patterns = {
  Singleton: 'ใช้เมื่อต้องการ instance เดียวทั่วระบบ',
  Factory: 'ใช้เมื่อไม่รู้ว่าจะ new คลาสไหน',
  Builder: 'ใช้เมื่อวัตถุมี config ซับซ้อน',
  Prototype: 'ใช้เมื่อต้องการ clone วัตถุที่มีต้นทุนสูง'
};

Object.entries(patterns).forEach(([name, use]) => {
  console.log(`${name}: ${use}`);
});
```

---

## Step 993: Singleton Pattern - พื้นฐาน

**Singleton Pattern** ทำให้แน่ใจว่าคลาสมีเพียง instance เดียว และมี global point of access

```javascript
// ========================
// Singleton แบบ Classic
// ========================

class Singleton {
  constructor() {
    if (Singleton.instance) {
      return Singleton.instance;
    }
    this.id = Math.random().toString(36).substr(2, 9);
    this.createdAt = new Date();
    Singleton.instance = this;
  }
  
  getId() {
    return this.id;
  }
}

const s1 = new Singleton();
const s2 = new Singleton();
const s3 = new Singleton();

console.log(s1 === s2); // true
console.log(s2 === s3); // true
console.log(s1.getId() === s2.getId()); // true - เป็น instance เดียวกัน

// ========================
// Singleton แบบ Module Pattern
// ========================

const SingletonModule = (() => {
  let instance = null;
  
  function createInstance() {
    return {
      id: Math.random().toString(36).substr(2, 9),
      data: {},
      
      set(key, value) {
        this.data[key] = value;
      },
      
      get(key) {
        return this.data[key];
      }
    };
  }
  
  return {
    getInstance() {
      if (!instance) {
        instance = createInstance();
      }
      return instance;
    }
  };
})();

const m1 = SingletonModule.getInstance();
const m2 = SingletonModule.getInstance();

console.log(m1 === m2); // true
m1.set('name', 'JavaScript Course');
console.log(m2.get('name')); // 'JavaScript Course' - เข้าถึง instance เดียวกัน
```

---

## Step 994: Singleton Pattern - ด้วย Closure

```javascript
// ========================
// Singleton ด้วย Closure
// ========================

function createSingleton(initialValue = {}) {
  let instance = null;
  
  class SingletonClass {
    constructor(data) {
      this.data = { ...data };
      this.createdAt = new Date().toISOString();
    }
    
    getData() {
      return { ...this.data };
    }
    
    setData(key, value) {
      this.data[key] = value;
    }
    
    toString() {
      return `Singleton(created: ${this.createdAt})`;
    }
  }
  
  return {
    getInstance() {
      if (!instance) {
        instance = new SingletonClass(initialValue);
      }
      return instance;
    },
    
    resetInstance() {
      // ใช้สำหรับ testing เท่านั้น
      instance = null;
    }
  };
}

// ใช้งาน
const ConfigSingleton = createSingleton({ env: 'development' });
const config1 = ConfigSingleton.getInstance();
const config2 = ConfigSingleton.getInstance();

config1.setData('apiUrl', 'https://api.example.com');
console.log(config2.getData().apiUrl); // 'https://api.example.com'
console.log(config1 === config2); // true

// ========================
// Singleton ด้วย Symbol
// ========================

const INSTANCE_KEY = Symbol('instance');

class AppState {
  constructor() {
    if (AppState[INSTANCE_KEY]) {
      return AppState[INSTANCE_KEY];
    }
    
    this.state = {
      user: null,
      theme: 'light',
      language: 'th',
      notifications: []
    };
    
    AppState[INSTANCE_KEY] = this;
  }
  
  getState() {
    return { ...this.state };
  }
  
  setState(updates) {
    this.state = { ...this.state, ...updates };
  }
  
  addNotification(notification) {
    this.state.notifications.push({
      id: Date.now(),
      ...notification
    });
  }
}

const state1 = new AppState();
const state2 = new AppState();

state1.setState({ user: { name: 'สมชาย', email: 'somchai@example.com' } });
console.log(state2.getState().user); // { name: 'สมชาย', ... }
console.log(state1 === state2); // true
```

---

## Step 995: Singleton Pattern - การใช้งานจริง

```javascript
// ========================
// Database Connection Singleton
// ========================

class DatabaseConnection {
  constructor() {
    if (DatabaseConnection._instance) {
      return DatabaseConnection._instance;
    }
    
    this._connection = null;
    this._isConnected = false;
    this._queryCount = 0;
    this._config = {
      host: process.env.DB_HOST || 'localhost',
      port: process.env.DB_PORT || 5432,
      database: process.env.DB_NAME || 'myapp'
    };
    
    DatabaseConnection._instance = this;
  }
  
  async connect() {
    if (this._isConnected) {
      console.log('Already connected to database');
      return this._connection;
    }
    
    // จำลองการเชื่อมต่อ
    await new Promise(resolve => setTimeout(resolve, 100));
    this._connection = `DB Connection to ${this._config.host}:${this._config.port}/${this._config.database}`;
    this._isConnected = true;
    console.log(`Connected: ${this._connection}`);
    return this._connection;
  }
  
  async query(sql, params = []) {
    if (!this._isConnected) {
      throw new Error('Not connected to database');
    }
    
    this._queryCount++;
    console.log(`Query #${this._queryCount}: ${sql}`);
    
    // จำลองผลลัพธ์
    return { rows: [], count: 0, query: sql, params };
  }
  
  disconnect() {
    this._connection = null;
    this._isConnected = false;
    console.log('Disconnected from database');
  }
  
  getStats() {
    return {
      isConnected: this._isConnected,
      queryCount: this._queryCount,
      config: { ...this._config }
    };
  }
}

// ใช้งาน
async function useDatabase() {
  const db1 = new DatabaseConnection();
  const db2 = new DatabaseConnection();
  
  console.log(db1 === db2); // true
  
  await db1.connect();
  await db2.connect(); // จะไม่เชื่อมต่อใหม่
  
  await db1.query('SELECT * FROM users');
  await db2.query('SELECT * FROM products');
  
  console.log(db1.getStats()); // queryCount: 2 (นับรวมกัน)
}

useDatabase().catch(console.error);

// ========================
// Logger Singleton
// ========================

class Logger {
  constructor() {
    if (Logger._instance) {
      return Logger._instance;
    }
    
    this._logs = [];
    this._level = 'INFO';
    this._levels = { DEBUG: 0, INFO: 1, WARN: 2, ERROR: 3 };
    
    Logger._instance = this;
  }
  
  setLevel(level) {
    if (this._levels[level] === undefined) {
      throw new Error(`Invalid log level: ${level}`);
    }
    this._level = level;
  }
  
  _shouldLog(level) {
    return this._levels[level] >= this._levels[this._level];
  }
  
  _log(level, message, data = null) {
    if (!this._shouldLog(level)) return;
    
    const entry = {
      timestamp: new Date().toISOString(),
      level,
      message,
      data
    };
    
    this._logs.push(entry);
    
    const prefix = `[${entry.timestamp}] [${level}]`;
    const output = data 
      ? `${prefix} ${message} ${JSON.stringify(data)}`
      : `${prefix} ${message}`;
    
    if (level === 'ERROR') {
      console.error(output);
    } else if (level === 'WARN') {
      console.warn(output);
    } else {
      console.log(output);
    }
  }
  
  debug(message, data) { this._log('DEBUG', message, data); }
  info(message, data) { this._log('INFO', message, data); }
  warn(message, data) { this._log('WARN', message, data); }
  error(message, data) { this._log('ERROR', message, data); }
  
  getLogs(level = null) {
    if (level) {
      return this._logs.filter(log => log.level === level);
    }
    return [...this._logs];
  }
  
  clearLogs() {
    this._logs = [];
  }
}

// ใช้งาน
const logger1 = new Logger();
const logger2 = new Logger();

console.log(logger1 === logger2); // true

logger1.info('แอปพลิเคชันเริ่มทำงาน');
logger2.warn('ตรวจพบการใช้งานผิดปกติ');
logger1.error('เกิดข้อผิดพลาด', { code: 500 });

console.log(logger2.getLogs()); // มี 3 logs ทั้งที่ใช้ logger1 และ logger2
```

---

## Step 996: Singleton Pattern - Config Manager

```javascript
// ========================
// Configuration Manager Singleton
// ========================

class ConfigManager {
  constructor() {
    if (ConfigManager._instance) {
      return ConfigManager._instance;
    }
    
    this._config = {};
    this._defaults = {};
    this._validators = {};
    this._initialized = false;
    
    ConfigManager._instance = this;
  }
  
  initialize(config = {}) {
    if (this._initialized) {
      console.warn('ConfigManager already initialized');
      return this;
    }
    
    this._config = { ...this._defaults, ...config };
    this._initialized = true;
    return this;
  }
  
  setDefault(key, value) {
    this._defaults[key] = value;
    if (!this._config.hasOwnProperty(key)) {
      this._config[key] = value;
    }
    return this;
  }
  
  setValidator(key, validatorFn) {
    this._validators[key] = validatorFn;
    return this;
  }
  
  get(key, defaultValue = undefined) {
    const keys = key.split('.');
    let value = this._config;
    
    for (const k of keys) {
      if (value === null || value === undefined) {
        return defaultValue;
      }
      value = value[k];
    }
    
    return value !== undefined ? value : defaultValue;
  }
  
  set(key, value) {
    if (this._validators[key]) {
      const error = this._validators[key](value);
      if (error) {
        throw new Error(`Config validation failed for '${key}': ${error}`);
      }
    }
    
    const keys = key.split('.');
    let obj = this._config;
    
    for (let i = 0; i < keys.length - 1; i++) {
      if (!obj[keys[i]] || typeof obj[keys[i]] !== 'object') {
        obj[keys[i]] = {};
      }
      obj = obj[keys[i]];
    }
    
    obj[keys[keys.length - 1]] = value;
    return this;
  }
  
  getAll() {
    return JSON.parse(JSON.stringify(this._config));
  }
  
  reset() {
    this._config = { ...this._defaults };
    this._initialized = false;
    return this;
  }
}

// ใช้งาน
const config = new ConfigManager();

// ตั้งค่า default
config
  .setDefault('app.name', 'MyApp')
  .setDefault('app.version', '1.0.0')
  .setDefault('server.port', 3000)
  .setDefault('server.host', 'localhost')
  .setDefault('database.maxConnections', 10);

// ตั้ง validator
config.setValidator('server.port', (value) => {
  if (typeof value !== 'number') return 'Port must be a number';
  if (value < 1 || value > 65535) return 'Port must be between 1 and 65535';
  return null; // ผ่าน
});

// initialize
config.initialize({
  'server.port': 8080,
  'database.host': 'db.example.com'
});

const config2 = new ConfigManager();
console.log(config === config2); // true
console.log(config2.get('app.name')); // 'MyApp'
console.log(config2.get('server.port')); // 8080

try {
  config.set('server.port', 99999); // จะ throw error
} catch (e) {
  console.error(e.message); // 'Config validation failed for server.port: Port must be between 1 and 65535'
}
```

---

## Step 997: Factory Pattern - Simple Factory

**Factory Pattern** สร้างวัตถุโดยไม่ต้องระบุคลาสที่แน่นอน ทำให้ผู้ใช้ไม่ต้องรู้รายละเอียดการสร้าง

```javascript
// ========================
// Simple Factory
// ========================

// ปัญหาแบบเดิม
function createAnimal(type) {
  if (type === 'dog') {
    return { type: 'dog', sound: 'Woof', legs: 4 };
  } else if (type === 'cat') {
    return { type: 'cat', sound: 'Meow', legs: 4 };
  } else if (type === 'bird') {
    return { type: 'bird', sound: 'Tweet', legs: 2 };
  }
  throw new Error(`Unknown animal type: ${type}`);
}

// Better: Simple Factory ด้วย Registry
class AnimalFactory {
  constructor() {
    this._creators = new Map();
  }
  
  register(type, creator) {
    this._creators.set(type, creator);
    return this; // สำหรับ chaining
  }
  
  create(type, ...args) {
    const creator = this._creators.get(type);
    if (!creator) {
      throw new Error(`No creator registered for type: ${type}`);
    }
    return creator(...args);
  }
  
  getTypes() {
    return [...this._creators.keys()];
  }
}

// กำหนดคลาสสัตว์
class Dog {
  constructor(name) {
    this.type = 'dog';
    this.name = name;
    this.sound = 'Woof';
    this.legs = 4;
  }
  
  speak() {
    return `${this.name} says: ${this.sound}!`;
  }
}

class Cat {
  constructor(name) {
    this.type = 'cat';
    this.name = name;
    this.sound = 'Meow';
    this.legs = 4;
  }
  
  speak() {
    return `${this.name} says: ${this.sound}!`;
  }
}

class Bird {
  constructor(name) {
    this.type = 'bird';
    this.name = name;
    this.sound = 'Tweet';
    this.legs = 2;
  }
  
  speak() {
    return `${this.name} says: ${this.sound}!`;
  }
  
  fly() {
    return `${this.name} is flying!`;
  }
}

// ลงทะเบียนและใช้งาน
const animalFactory = new AnimalFactory();

animalFactory
  .register('dog', (name) => new Dog(name))
  .register('cat', (name) => new Cat(name))
  .register('bird', (name) => new Bird(name));

const dog = animalFactory.create('dog', 'บัดดี้');
const cat = animalFactory.create('cat', 'วิสเกอร์ส');
const bird = animalFactory.create('bird', 'ทวีตตี้');

console.log(dog.speak()); // 'บัดดี้ says: Woof!'
console.log(cat.speak()); // 'วิสเกอร์ส says: Meow!'
console.log(bird.fly());  // 'ทวีตตี้ is flying!'

console.log(animalFactory.getTypes()); // ['dog', 'cat', 'bird']

// เพิ่ม type ใหม่ได้ง่าย
class Fish {
  constructor(name) {
    this.type = 'fish';
    this.name = name;
    this.sound = '..';
    this.legs = 0;
  }
  
  speak() {
    return `${this.name} says: ${this.sound} (ปลาพูดไม่ได้)`;
  }
  
  swim() {
    return `${this.name} is swimming!`;
  }
}

animalFactory.register('fish', (name) => new Fish(name));
const fish = animalFactory.create('fish', 'นีโม่');
console.log(fish.swim()); // 'นีโม่ is swimming!'
```

---

## Step 998: Factory Method Pattern

```javascript
// ========================
// Factory Method Pattern
// ========================

// Abstract Creator (base class)
class DocumentCreator {
  // Factory Method - subclass ต้องกำหนด
  createDocument(content) {
    throw new Error('Subclass must implement createDocument()');
  }
  
  // Template method ที่ใช้ factory method
  exportDocument(content, filename) {
    const doc = this.createDocument(content); // เรียก factory method
    doc.save(filename);
    doc.validate();
    return `Exported: ${filename}`;
  }
  
  previewDocument(content) {
    const doc = this.createDocument(content);
    return doc.preview();
  }
}

// Concrete Creators
class PDFCreator extends DocumentCreator {
  createDocument(content) {
    return new PDFDocument(content);
  }
}

class WordCreator extends DocumentCreator {
  createDocument(content) {
    return new WordDocument(content);
  }
}

class HTMLCreator extends DocumentCreator {
  createDocument(content) {
    return new HTMLDocument(content);
  }
}

// Concrete Products
class PDFDocument {
  constructor(content) {
    this.content = content;
    this.type = 'PDF';
    this.metadata = { pages: 1, size: 'A4' };
  }
  
  save(filename) {
    console.log(`Saving PDF: ${filename}.pdf`);
  }
  
  validate() {
    console.log('Validating PDF structure...');
    return true;
  }
  
  preview() {
    return `[PDF Preview] ${this.content.substring(0, 100)}...`;
  }
}

class WordDocument {
  constructor(content) {
    this.content = content;
    this.type = 'DOCX';
    this.styles = ['Heading 1', 'Normal', 'Bold'];
  }
  
  save(filename) {
    console.log(`Saving Word Document: ${filename}.docx`);
  }
  
  validate() {
    console.log('Validating Word document format...');
    return true;
  }
  
  preview() {
    return `[Word Preview] ${this.content.substring(0, 100)}...`;
  }
}

class HTMLDocument {
  constructor(content) {
    this.content = content;
    this.type = 'HTML';
    this.tags = [];
  }
  
  save(filename) {
    console.log(`Saving HTML: ${filename}.html`);
  }
  
  validate() {
    console.log('Validating HTML structure...');
    return true;
  }
  
  preview() {
    return `[HTML Preview] <p>${this.content.substring(0, 100)}</p>`;
  }
}

// ใช้งาน
const content = 'นี่คือเนื้อหาเอกสารสำคัญของเรา ประกอบด้วยข้อมูลต่างๆ มากมาย';

const pdfCreator = new PDFCreator();
const wordCreator = new WordCreator();
const htmlCreator = new HTMLCreator();

pdfCreator.exportDocument(content, 'report');
wordCreator.exportDocument(content, 'report');
htmlCreator.exportDocument(content, 'report');

// ฟังก์ชันที่ทำงานกับ creator ใดก็ได้
function processDocument(creator, content, filename) {
  console.log(`Preview: ${creator.previewDocument(content)}`);
  return creator.exportDocument(content, filename);
}

processDocument(pdfCreator, content, 'document');
processDocument(wordCreator, content, 'document');
```

---

## Step 999: Abstract Factory Pattern

```javascript
// ========================
// Abstract Factory Pattern
// ========================

// ใช้สร้างกลุ่มของวัตถุที่เกี่ยวข้องกัน

// Abstract Products
class Button {
  render() { throw new Error('Not implemented'); }
  onClick(handler) { throw new Error('Not implemented'); }
}

class TextInput {
  render() { throw new Error('Not implemented'); }
  getValue() { throw new Error('Not implemented'); }
}

class Checkbox {
  render() { throw new Error('Not implemented'); }
  isChecked() { throw new Error('Not implemented'); }
}

// Concrete Products - Light Theme
class LightButton extends Button {
  constructor(label) {
    super();
    this.label = label;
    this.style = 'background: white; color: black; border: 1px solid gray';
  }
  
  render() {
    return `<button style="${this.style}">${this.label}</button>`;
  }
  
  onClick(handler) {
    console.log(`Light button "${this.label}" clicked`);
    handler && handler();
  }
}

class LightTextInput extends TextInput {
  constructor(placeholder) {
    super();
    this.placeholder = placeholder;
    this.value = '';
    this.style = 'background: white; color: black; border: 1px solid #ccc';
  }
  
  render() {
    return `<input type="text" placeholder="${this.placeholder}" style="${this.style}">`;
  }
  
  getValue() {
    return this.value;
  }
}

class LightCheckbox extends Checkbox {
  constructor(label) {
    super();
    this.label = label;
    this.checked = false;
    this.style = 'accent-color: blue';
  }
  
  render() {
    const checked = this.checked ? 'checked' : '';
    return `<input type="checkbox" ${checked} style="${this.style}"> ${this.label}`;
  }
  
  isChecked() {
    return this.checked;
  }
}

// Concrete Products - Dark Theme
class DarkButton extends Button {
  constructor(label) {
    super();
    this.label = label;
    this.style = 'background: #333; color: white; border: 1px solid #555';
  }
  
  render() {
    return `<button style="${this.style}">${this.label}</button>`;
  }
  
  onClick(handler) {
    console.log(`Dark button "${this.label}" clicked`);
    handler && handler();
  }
}

class DarkTextInput extends TextInput {
  constructor(placeholder) {
    super();
    this.placeholder = placeholder;
    this.value = '';
    this.style = 'background: #222; color: white; border: 1px solid #444';
  }
  
  render() {
    return `<input type="text" placeholder="${this.placeholder}" style="${this.style}">`;
  }
  
  getValue() {
    return this.value;
  }
}

class DarkCheckbox extends Checkbox {
  constructor(label) {
    super();
    this.label = label;
    this.checked = false;
    this.style = 'accent-color: cyan';
  }
  
  render() {
    const checked = this.checked ? 'checked' : '';
    return `<input type="checkbox" ${checked} style="${this.style}"> ${this.label}`;
  }
  
  isChecked() {
    return this.checked;
  }
}

// Abstract Factory
class UIFactory {
  createButton(label) { throw new Error('Not implemented'); }
  createTextInput(placeholder) { throw new Error('Not implemented'); }
  createCheckbox(label) { throw new Error('Not implemented'); }
}

// Concrete Factories
class LightThemeFactory extends UIFactory {
  createButton(label) {
    return new LightButton(label);
  }
  
  createTextInput(placeholder) {
    return new LightTextInput(placeholder);
  }
  
  createCheckbox(label) {
    return new LightCheckbox(label);
  }
}

class DarkThemeFactory extends UIFactory {
  createButton(label) {
    return new DarkButton(label);
  }
  
  createTextInput(placeholder) {
    return new DarkTextInput(placeholder);
  }
  
  createCheckbox(label) {
    return new DarkCheckbox(label);
  }
}

// Application ที่ทำงานกับ factory ใดก็ได้
class LoginForm {
  constructor(factory) {
    this.factory = factory;
    this.components = {};
  }
  
  build() {
    this.components.usernameInput = this.factory.createTextInput('กรุณาใส่ชื่อผู้ใช้');
    this.components.passwordInput = this.factory.createTextInput('กรุณาใส่รหัสผ่าน');
    this.components.rememberMe = this.factory.createCheckbox('จำฉันไว้');
    this.components.loginButton = this.factory.createButton('เข้าสู่ระบบ');
    this.components.registerButton = this.factory.createButton('สมัครสมาชิก');
    return this;
  }
  
  render() {
    return Object.values(this.components)
      .map(comp => comp.render())
      .join('\n');
  }
}

// ใช้งาน
const lightFactory = new LightThemeFactory();
const darkFactory = new DarkThemeFactory();

const lightForm = new LoginForm(lightFactory).build();
const darkForm = new LoginForm(darkFactory).build();

console.log('=== Light Theme Form ===');
console.log(lightForm.render());

console.log('\n=== Dark Theme Form ===');
console.log(darkForm.render());
```

---

## Step 1000: Builder Pattern - พื้นฐาน

**Builder Pattern** แยกการสร้างวัตถุซับซ้อนออกจากการใช้งาน

```javascript
// ========================
// Builder Pattern - พื้นฐาน
// ========================

// ปัญหา: Constructor ที่มีพารามิเตอร์มากเกินไป
// new Pizza(size, crust, sauce, cheese, toppings, extraCheese, well_done, ...)

// วิธีแก้: Builder Pattern
class Pizza {
  constructor(builder) {
    this.size = builder.size;
    this.crust = builder.crust;
    this.sauce = builder.sauce;
    this.cheese = builder.cheese;
    this.toppings = builder.toppings;
    this.extraCheese = builder.extraCheese;
    this.wellDone = builder.wellDone;
  }
  
  toString() {
    const toppings = this.toppings.length > 0 
      ? this.toppings.join(', ')
      : 'ไม่มีท็อปปิ้ง';
    
    return `
Pizza Details:
  ขนาด: ${this.size}
  แป้ง: ${this.crust}
  ซอส: ${this.sauce}
  ชีส: ${this.cheese}
  ท็อปปิ้ง: ${toppings}
  Extra Cheese: ${this.extraCheese ? 'ใช่' : 'ไม่'}
  Well Done: ${this.wellDone ? 'ใช่' : 'ไม่'}
    `.trim();
  }
}

class PizzaBuilder {
  constructor() {
    // ค่า default
    this.size = 'Medium';
    this.crust = 'Regular';
    this.sauce = 'Tomato';
    this.cheese = 'Mozzarella';
    this.toppings = [];
    this.extraCheese = false;
    this.wellDone = false;
  }
  
  setSize(size) {
    const validSizes = ['Small', 'Medium', 'Large', 'XLarge'];
    if (!validSizes.includes(size)) {
      throw new Error(`Invalid size. Must be one of: ${validSizes.join(', ')}`);
    }
    this.size = size;
    return this; // สำหรับ method chaining
  }
  
  setCrust(crust) {
    const validCrusts = ['Thin', 'Regular', 'Thick', 'Stuffed'];
    if (!validCrusts.includes(crust)) {
      throw new Error(`Invalid crust. Must be one of: ${validCrusts.join(', ')}`);
    }
    this.crust = crust;
    return this;
  }
  
  setSauce(sauce) {
    this.sauce = sauce;
    return this;
  }
  
  setCheese(cheese) {
    this.cheese = cheese;
    return this;
  }
  
  addTopping(topping) {
    this.toppings.push(topping);
    return this;
  }
  
  addToppings(...toppings) {
    this.toppings.push(...toppings);
    return this;
  }
  
  withExtraCheese() {
    this.extraCheese = true;
    return this;
  }
  
  wellDone() {
    this.wellDone = true;
    return this;
  }
  
  build() {
    // Validation
    if (!this.size || !this.crust) {
      throw new Error('Size and crust are required');
    }
    return new Pizza(this);
  }
}

// ใช้งาน
const pizza1 = new PizzaBuilder()
  .setSize('Large')
  .setCrust('Thin')
  .setSauce('BBQ')
  .setCheese('Cheddar')
  .addToppings('Pepperoni', 'Mushroom', 'Onion')
  .withExtraCheese()
  .build();

console.log(pizza1.toString());

const pizza2 = new PizzaBuilder()
  .setSize('Small')
  .setCrust('Stuffed')
  .setSauce('Tomato')
  .build();

console.log(pizza2.toString());
```

---

## Step 1001: Builder Pattern - Fluent Interface

```javascript
// ========================
// Builder Pattern - Fluent Interface สำหรับ HTTP Request
// ========================

class HttpRequest {
  constructor(config) {
    this.method = config.method;
    this.url = config.url;
    this.headers = config.headers;
    this.body = config.body;
    this.timeout = config.timeout;
    this.retries = config.retries;
    this.auth = config.auth;
  }
  
  async execute() {
    console.log(`${this.method} ${this.url}`);
    console.log('Headers:', this.headers);
    if (this.body) console.log('Body:', this.body);
    
    // จำลองการส่ง request
    return {
      status: 200,
      data: { message: 'Success' }
    };
  }
}

class RequestBuilder {
  constructor() {
    this._config = {
      method: 'GET',
      url: '',
      headers: {},
      body: null,
      timeout: 30000,
      retries: 0,
      auth: null
    };
  }
  
  get(url) {
    this._config.method = 'GET';
    this._config.url = url;
    return this;
  }
  
  post(url) {
    this._config.method = 'POST';
    this._config.url = url;
    return this;
  }
  
  put(url) {
    this._config.method = 'PUT';
    this._config.url = url;
    return this;
  }
  
  delete(url) {
    this._config.method = 'DELETE';
    this._config.url = url;
    return this;
  }
  
  header(name, value) {
    this._config.headers[name] = value;
    return this;
  }
  
  headers(headers) {
    this._config.headers = { ...this._config.headers, ...headers };
    return this;
  }
  
  contentType(type) {
    return this.header('Content-Type', type);
  }
  
  json(body) {
    this._config.body = JSON.stringify(body);
    return this.contentType('application/json');
  }
  
  formData(data) {
    this._config.body = new URLSearchParams(data).toString();
    return this.contentType('application/x-www-form-urlencoded');
  }
  
  auth(token) {
    this._config.auth = token;
    return this.header('Authorization', `Bearer ${token}`);
  }
  
  basicAuth(username, password) {
    const credentials = Buffer.from(`${username}:${password}`).toString('base64');
    return this.header('Authorization', `Basic ${credentials}`);
  }
  
  timeout(ms) {
    this._config.timeout = ms;
    return this;
  }
  
  retry(times) {
    this._config.retries = times;
    return this;
  }
  
  build() {
    if (!this._config.url) {
      throw new Error('URL is required');
    }
    return new HttpRequest(this._config);
  }
  
  async send() {
    return this.build().execute();
  }
}

// ใช้งาน
const request = new RequestBuilder()
  .post('https://api.example.com/users')
  .headers({
    'Accept': 'application/json',
    'X-Request-ID': '123456'
  })
  .json({
    name: 'สมชาย ใจดี',
    email: 'somchai@example.com',
    role: 'user'
  })
  .auth('my-jwt-token')
  .timeout(5000)
  .retry(3)
  .build();

request.execute();

// ========================
// Builder สำหรับ SQL Query
// ========================

class QueryBuilder {
  constructor() {
    this._table = '';
    this._columns = ['*'];
    this._conditions = [];
    this._orderBy = [];
    this._limit = null;
    this._offset = null;
    this._joins = [];
    this._params = [];
  }
  
  from(table) {
    this._table = table;
    return this;
  }
  
  select(...columns) {
    this._columns = columns;
    return this;
  }
  
  where(condition, ...params) {
    this._conditions.push(condition);
    this._params.push(...params);
    return this;
  }
  
  andWhere(condition, ...params) {
    return this.where(condition, ...params);
  }
  
  join(table, condition) {
    this._joins.push(`JOIN ${table} ON ${condition}`);
    return this;
  }
  
  leftJoin(table, condition) {
    this._joins.push(`LEFT JOIN ${table} ON ${condition}`);
    return this;
  }
  
  orderBy(column, direction = 'ASC') {
    this._orderBy.push(`${column} ${direction}`);
    return this;
  }
  
  limit(n) {
    this._limit = n;
    return this;
  }
  
  offset(n) {
    this._offset = n;
    return this;
  }
  
  build() {
    if (!this._table) throw new Error('Table name is required');
    
    let sql = `SELECT ${this._columns.join(', ')} FROM ${this._table}`;
    
    if (this._joins.length > 0) {
      sql += ' ' + this._joins.join(' ');
    }
    
    if (this._conditions.length > 0) {
      sql += ' WHERE ' + this._conditions.join(' AND ');
    }
    
    if (this._orderBy.length > 0) {
      sql += ' ORDER BY ' + this._orderBy.join(', ');
    }
    
    if (this._limit !== null) {
      sql += ` LIMIT ${this._limit}`;
    }
    
    if (this._offset !== null) {
      sql += ` OFFSET ${this._offset}`;
    }
    
    return { sql, params: this._params };
  }
}

// ใช้งาน
const query = new QueryBuilder()
  .from('users u')
  .select('u.id', 'u.name', 'u.email', 'r.name AS role')
  .leftJoin('roles r', 'u.role_id = r.id')
  .where('u.active = ?', 1)
  .where('u.created_at > ?', '2024-01-01')
  .orderBy('u.name')
  .limit(20)
  .offset(0)
  .build();

console.log('SQL:', query.sql);
console.log('Params:', query.params);
```

---

## Step 1002: Builder Pattern - Director Class

```javascript
// ========================
// Builder Pattern ด้วย Director
// ========================

// Product
class Computer {
  constructor() {
    this.cpu = '';
    this.ram = '';
    this.storage = '';
    this.gpu = '';
    this.motherboard = '';
    this.powerSupply = '';
    this.cooling = '';
    this.case_ = '';
  }
  
  getSpecs() {
    return `
Computer Specs:
  CPU: ${this.cpu}
  RAM: ${this.ram}
  Storage: ${this.storage}
  GPU: ${this.gpu}
  Motherboard: ${this.motherboard}
  Power Supply: ${this.powerSupply}
  Cooling: ${this.cooling}
  Case: ${this.case_}
    `.trim();
  }
}

// Builder Interface
class ComputerBuilder {
  setCPU(cpu) { throw new Error('Not implemented'); }
  setRAM(ram) { throw new Error('Not implemented'); }
  setStorage(storage) { throw new Error('Not implemented'); }
  setGPU(gpu) { throw new Error('Not implemented'); }
  setMotherboard(mb) { throw new Error('Not implemented'); }
  setPowerSupply(psu) { throw new Error('Not implemented'); }
  setCooling(cooling) { throw new Error('Not implemented'); }
  setCase(case_) { throw new Error('Not implemented'); }
  build() { throw new Error('Not implemented'); }
}

// Concrete Builder
class StandardComputerBuilder extends ComputerBuilder {
  constructor() {
    super();
    this._computer = new Computer();
  }
  
  reset() {
    this._computer = new Computer();
    return this;
  }
  
  setCPU(cpu) {
    this._computer.cpu = cpu;
    return this;
  }
  
  setRAM(ram) {
    this._computer.ram = ram;
    return this;
  }
  
  setStorage(storage) {
    this._computer.storage = storage;
    return this;
  }
  
  setGPU(gpu) {
    this._computer.gpu = gpu;
    return this;
  }
  
  setMotherboard(mb) {
    this._computer.motherboard = mb;
    return this;
  }
  
  setPowerSupply(psu) {
    this._computer.powerSupply = psu;
    return this;
  }
  
  setCooling(cooling) {
    this._computer.cooling = cooling;
    return this;
  }
  
  setCase(case_) {
    this._computer.case_ = case_;
    return this;
  }
  
  build() {
    const computer = this._computer;
    this.reset();
    return computer;
  }
}

// Director
class ComputerDirector {
  constructor(builder) {
    this._builder = builder;
  }
  
  setBuilder(builder) {
    this._builder = builder;
  }
  
  buildGamingPC() {
    return this._builder
      .setCPU('Intel Core i9-14900K')
      .setRAM('64GB DDR5 6000MHz')
      .setStorage('2TB NVMe SSD')
      .setGPU('NVIDIA RTX 4090')
      .setMotherboard('ASUS ROG Maximus Z790')
      .setPowerSupply('1000W 80+ Platinum')
      .setCooling('360mm AIO Liquid Cooling')
      .setCase('Lian Li O11 Dynamic')
      .build();
  }
  
  buildWorkstation() {
    return this._builder
      .setCPU('AMD Ryzen Threadripper 7970X')
      .setRAM('128GB ECC DDR5')
      .setStorage('4TB NVMe RAID 0')
      .setGPU('NVIDIA RTX A6000')
      .setMotherboard('ASUS Pro WS TRX50-SAGE WIFI')
      .setPowerSupply('1600W Titanium')
      .setCooling('Custom Water Cooling')
      .setCase('Fractal Design Define 7 XL')
      .build();
  }
  
  buildOfficePC() {
    return this._builder
      .setCPU('Intel Core i5-13400')
      .setRAM('16GB DDR5 4800MHz')
      .setStorage('512GB SSD')
      .setGPU('Intel UHD Graphics 730 (Integrated)')
      .setMotherboard('ASUS PRIME B660M-A')
      .setPowerSupply('450W 80+ Bronze')
      .setCooling('Stock CPU Cooler')
      .setCase('Fractal Design Pop Mini')
      .build();
  }
}

// ใช้งาน
const builder = new StandardComputerBuilder();
const director = new ComputerDirector(builder);

const gamingPC = director.buildGamingPC();
console.log('=== Gaming PC ===');
console.log(gamingPC.getSpecs());

const workstation = director.buildWorkstation();
console.log('\n=== Workstation ===');
console.log(workstation.getSpecs());

const officePC = director.buildOfficePC();
console.log('\n=== Office PC ===');
console.log(officePC.getSpecs());

// สร้าง custom spec
const customPC = builder
  .setCPU('AMD Ryzen 7 7800X3D')
  .setRAM('32GB DDR5')
  .setStorage('1TB NVMe SSD')
  .setGPU('RX 7900 XTX')
  .setMotherboard('MSI MAG X670E Tomahawk')
  .setPowerSupply('750W 80+ Gold')
  .setCooling('Noctua NH-D15')
  .setCase('Phanteks Eclipse P500A')
  .build();

console.log('\n=== Custom PC ===');
console.log(customPC.getSpecs());
```

---

## Step 1003: Prototype Pattern

**Prototype Pattern** สร้างวัตถุใหม่โดยการ clone วัตถุที่มีอยู่แล้ว แทนที่จะสร้างใหม่ตั้งแต่ต้น

```javascript
// ========================
// Prototype Pattern พื้นฐาน
// ========================

// ปัญหา: การสร้างวัตถุที่มีต้นทุนสูง
class ExpensiveObject {
  constructor() {
    // จำลอง initialization ที่ใช้เวลานาน
    console.log('Creating expensive object...');
    this.data = {};
    this.timestamp = Date.now();
    
    // จำลองการโหลดข้อมูลจาก database
    for (let i = 0; i < 1000; i++) {
      this.data[`key_${i}`] = `value_${i}`;
    }
  }
}

// ใช้ Prototype
class Shape {
  constructor(color, x, y) {
    this.color = color;
    this.x = x;
    this.y = y;
  }
  
  clone() {
    // Shallow copy
    return Object.assign(Object.create(Object.getPrototypeOf(this)), this);
  }
  
  deepClone() {
    // Deep copy
    return JSON.parse(JSON.stringify(this));
  }
  
  move(x, y) {
    this.x = x;
    this.y = y;
    return this;
  }
  
  setColor(color) {
    this.color = color;
    return this;
  }
  
  toString() {
    return `${this.constructor.name}(color=${this.color}, x=${this.x}, y=${this.y})`;
  }
}

class Circle extends Shape {
  constructor(color, x, y, radius) {
    super(color, x, y);
    this.radius = radius;
  }
  
  getArea() {
    return Math.PI * this.radius ** 2;
  }
  
  toString() {
    return `${super.toString()}, radius=${this.radius}`;
  }
}

class Rectangle extends Shape {
  constructor(color, x, y, width, height) {
    super(color, x, y);
    this.width = width;
    this.height = height;
  }
  
  getArea() {
    return this.width * this.height;
  }
  
  toString() {
    return `${super.toString()}, ${this.width}x${this.height}`;
  }
}

// ใช้งาน Prototype
const originalCircle = new Circle('blue', 0, 0, 50);
const clonedCircle = originalCircle.clone();

// แก้ไข clone โดยไม่กระทบ original
clonedCircle.setColor('red').move(100, 100);
clonedCircle.radius = 30;

console.log('Original:', originalCircle.toString());
// Circle(color=blue, x=0, y=0), radius=50

console.log('Cloned:', clonedCircle.toString());
// Circle(color=red, x=100, y=100), radius=30

// ========================
// Prototype Registry
// ========================

class PrototypeRegistry {
  constructor() {
    this._prototypes = new Map();
  }
  
  register(name, prototype) {
    this._prototypes.set(name, prototype);
    return this;
  }
  
  create(name, overrides = {}) {
    const prototype = this._prototypes.get(name);
    if (!prototype) {
      throw new Error(`No prototype registered for: ${name}`);
    }
    const clone = prototype.clone();
    Object.assign(clone, overrides);
    return clone;
  }
  
  getNames() {
    return [...this._prototypes.keys()];
  }
}

// สร้าง registry และลงทะเบียน prototypes
const registry = new PrototypeRegistry();

registry
  .register('small-circle', new Circle('gray', 0, 0, 10))
  .register('medium-circle', new Circle('gray', 0, 0, 25))
  .register('large-circle', new Circle('gray', 0, 0, 50))
  .register('button-shape', new Rectangle('blue', 0, 0, 120, 40))
  .register('card-shape', new Rectangle('white', 0, 0, 300, 200));

// สร้างวัตถุจาก prototypes
const redCircle = registry.create('medium-circle', { color: 'red', x: 50, y: 50 });
const greenCircle = registry.create('large-circle', { color: 'green', x: 200, y: 100 });
const button = registry.create('button-shape', { color: 'primary', x: 10, y: 10 });

console.log(redCircle.toString());
console.log(greenCircle.toString());
console.log(button.toString());
```

---

## Step 1004: Prototype Pattern - Deep Clone

```javascript
// ========================
// Deep Clone สำหรับ Complex Objects
// ========================

class UserProfile {
  constructor(data) {
    this.id = data.id;
    this.name = data.name;
    this.email = data.email;
    this.address = { ...data.address };
    this.preferences = { ...data.preferences };
    this.tags = [...(data.tags || [])];
    this.metadata = data.metadata ? JSON.parse(JSON.stringify(data.metadata)) : {};
  }
  
  clone() {
    return new UserProfile({
      id: this.id,
      name: this.name,
      email: this.email,
      address: { ...this.address },
      preferences: { ...this.preferences },
      tags: [...this.tags],
      metadata: JSON.parse(JSON.stringify(this.metadata))
    });
  }
  
  withId(id) {
    const clone = this.clone();
    clone.id = id;
    return clone;
  }
  
  withName(name) {
    const clone = this.clone();
    clone.name = name;
    return clone;
  }
  
  withTag(tag) {
    const clone = this.clone();
    if (!clone.tags.includes(tag)) {
      clone.tags.push(tag);
    }
    return clone;
  }
  
  withPreference(key, value) {
    const clone = this.clone();
    clone.preferences[key] = value;
    return clone;
  }
  
  toJSON() {
    return {
      id: this.id,
      name: this.name,
      email: this.email,
      address: this.address,
      preferences: this.preferences,
      tags: this.tags,
      metadata: this.metadata
    };
  }
}

// ใช้งาน
const baseProfile = new UserProfile({
  id: 1,
  name: 'สมชาย',
  email: 'somchai@example.com',
  address: {
    street: '123 ถนนสุขุมวิท',
    city: 'กรุงเทพฯ',
    country: 'ไทย'
  },
  preferences: {
    theme: 'dark',
    language: 'th',
    notifications: true
  },
  tags: ['user', 'active'],
  metadata: {
    loginCount: 0,
    lastLogin: null
  }
});

// Clone และแก้ไข
const adminProfile = baseProfile
  .withId(999)
  .withName('ผู้ดูแลระบบ')
  .withTag('admin')
  .withTag('super-user')
  .withPreference('theme', 'light');

// ตรวจสอบว่า original ไม่ถูกแก้ไข
console.log('Original:', JSON.stringify(baseProfile.toJSON(), null, 2));
console.log('Admin:', JSON.stringify(adminProfile.toJSON(), null, 2));

// ========================
// Structural Clone สำหรับ Game Objects
// ========================

class GameObject {
  constructor(config) {
    this.id = config.id || Math.random().toString(36).substr(2, 9);
    this.type = config.type;
    this.position = { ...config.position };
    this.stats = { ...config.stats };
    this.abilities = config.abilities ? [...config.abilities] : [];
    this.equipment = config.equipment ? [...config.equipment] : [];
  }
  
  clone() {
    return new GameObject({
      type: this.type,
      position: { ...this.position },
      stats: { ...this.stats },
      abilities: [...this.abilities],
      equipment: [...this.equipment]
    });
  }
  
  spawnAt(x, y) {
    const instance = this.clone();
    instance.position = { x, y };
    return instance;
  }
  
  toString() {
    return `${this.type} at (${this.position.x}, ${this.position.y}) HP: ${this.stats.hp}`;
  }
}

// Template
const goblinTemplate = new GameObject({
  type: 'Goblin',
  position: { x: 0, y: 0 },
  stats: { hp: 50, attack: 10, defense: 5, speed: 8 },
  abilities: ['bite', 'scratch'],
  equipment: ['rusty-knife']
});

// Spawn หลาย instance จาก template
const goblins = [];
for (let i = 0; i < 5; i++) {
  goblins.push(goblinTemplate.spawnAt(i * 100, 200));
}

goblins.forEach(g => console.log(g.toString()));

// แก้ไข instance หนึ่งโดยไม่กระทบอื่น
goblins[0].stats.hp = 10; // Goblin 0 ถูกโจมตี
console.log('\nAfter attack:');
goblins.forEach(g => console.log(g.toString()));
// Goblin 0: HP 10, อื่นๆ ยัง 50
```

---

## Step 1005: Object Pool Pattern

```javascript
// ========================
// Object Pool Pattern
// ========================

// ใช้เมื่อการสร้าง/ทำลายวัตถุมีต้นทุนสูง
// เช่น database connections, thread pools, socket connections

class Pool {
  constructor(factory, options = {}) {
    this._factory = factory;
    this._min = options.min || 2;
    this._max = options.max || 10;
    this._available = [];
    this._inUse = new Set();
    this._waiting = [];
    
    // สร้าง initial pool
    for (let i = 0; i < this._min; i++) {
      this._available.push(this._factory());
    }
  }
  
  async acquire() {
    if (this._available.length > 0) {
      const obj = this._available.pop();
      this._inUse.add(obj);
      return obj;
    }
    
    if (this._inUse.size < this._max) {
      const obj = this._factory();
      this._inUse.add(obj);
      return obj;
    }
    
    // รอจนกว่าจะมีว่าง
    return new Promise((resolve) => {
      this._waiting.push(resolve);
    });
  }
  
  release(obj) {
    if (!this._inUse.has(obj)) {
      throw new Error('Object not from this pool');
    }
    
    this._inUse.delete(obj);
    
    if (this._waiting.length > 0) {
      const resolve = this._waiting.shift();
      this._inUse.add(obj);
      resolve(obj);
    } else if (this._available.length < this._min) {
      this._available.push(obj);
    }
    // ถ้าเกิน min ปล่อยทิ้ง (garbage collected)
  }
  
  getStats() {
    return {
      available: this._available.length,
      inUse: this._inUse.size,
      waiting: this._waiting.length,
      total: this._available.length + this._inUse.size
    };
  }
  
  async withObject(callback) {
    const obj = await this.acquire();
    try {
      return await callback(obj);
    } finally {
      this.release(obj);
    }
  }
}

// ตัวอย่าง: Connection Pool
class DatabaseConnectionSimulator {
  constructor(id) {
    this.id = id;
    this._connected = false;
    this._queryCount = 0;
    console.log(`Created connection #${id}`);
    this.connect();
  }
  
  connect() {
    this._connected = true;
    console.log(`Connection #${this.id} connected`);
  }
  
  async query(sql) {
    this._queryCount++;
    await new Promise(resolve => setTimeout(resolve, 10));
    return { result: `Query from conn #${this.id}: ${sql}`, count: this._queryCount };
  }
  
  reset() {
    // Reset state ก่อน return to pool
    this._queryCount = 0;
    return this;
  }
}

let connectionIdCounter = 0;

async function demonstratePool() {
  const pool = new Pool(
    () => new DatabaseConnectionSimulator(++connectionIdCounter),
    { min: 2, max: 5 }
  );
  
  console.log('Initial pool stats:', pool.getStats());
  
  // ใช้หลาย connections พร้อมกัน
  const promises = [];
  for (let i = 0; i < 8; i++) {
    promises.push(
      pool.withObject(async (conn) => {
        const result = await conn.query(`SELECT * FROM table_${i}`);
        return result;
      })
    );
  }
  
  const results = await Promise.all(promises);
  console.log('All queries completed:', results.length);
  console.log('Final pool stats:', pool.getStats());
}

demonstratePool().catch(console.error);

// ========================
// Particle Pool สำหรับ Game
// ========================

class Particle {
  constructor() {
    this.reset();
  }
  
  reset() {
    this.x = 0;
    this.y = 0;
    this.vx = 0;
    this.vy = 0;
    this.alpha = 1;
    this.color = 'white';
    this.size = 2;
    this.lifetime = 0;
    this.maxLifetime = 60; // frames
    this.active = false;
  }
  
  init(x, y, color) {
    this.x = x;
    this.y = y;
    this.vx = (Math.random() - 0.5) * 5;
    this.vy = (Math.random() - 0.5) * 5;
    this.alpha = 1;
    this.color = color;
    this.size = Math.random() * 3 + 1;
    this.lifetime = 0;
    this.maxLifetime = Math.random() * 60 + 30;
    this.active = true;
    return this;
  }
  
  update() {
    if (!this.active) return false;
    
    this.x += this.vx;
    this.y += this.vy;
    this.vy += 0.1; // gravity
    this.alpha -= 1 / this.maxLifetime;
    this.lifetime++;
    
    if (this.lifetime >= this.maxLifetime || this.alpha <= 0) {
      this.active = false;
      return false; // ส่งคืน pool
    }
    
    return true; // ยังคง active
  }
}

class ParticleSystem {
  constructor(maxParticles = 500) {
    this._pool = [];
    this._active = [];
    
    // สร้าง pool ล่วงหน้า
    for (let i = 0; i < maxParticles; i++) {
      this._pool.push(new Particle());
    }
  }
  
  emit(x, y, color = 'orange', count = 10) {
    for (let i = 0; i < count; i++) {
      const particle = this._pool.pop();
      if (!particle) break; // pool หมด
      
      particle.init(x, y, color);
      this._active.push(particle);
    }
  }
  
  update() {
    const stillActive = [];
    
    for (const particle of this._active) {
      if (particle.update()) {
        stillActive.push(particle);
      } else {
        particle.reset();
        this._pool.push(particle); // คืน pool
      }
    }
    
    this._active = stillActive;
  }
  
  getStats() {
    return {
      active: this._active.length,
      pooled: this._pool.length,
      total: this._active.length + this._pool.length
    };
  }
}

// จำลองระบบ particle
const particles = new ParticleSystem(100);

// จำลอง game loop
for (let frame = 0; frame < 5; frame++) {
  if (frame % 2 === 0) {
    particles.emit(100, 100, 'red', 15);
    particles.emit(200, 150, 'blue', 10);
  }
  
  particles.update();
  console.log(`Frame ${frame}:`, particles.getStats());
}
```

---

## Step 1006: Singleton Thread Safety & Lazy Initialization

```javascript
// ========================
// Singleton ด้วย Lazy Initialization
// ========================

class HeavyResource {
  constructor() {
    console.log('Initializing heavy resource...');
    // จำลอง heavy initialization
    this._data = new Array(10000).fill(0).map((_, i) => i * 2);
    this._initialized = true;
    this._timestamp = Date.now();
  }
  
  process(input) {
    return this._data[input % this._data.length];
  }
}

// Lazy Singleton - สร้างเมื่อใช้งานครั้งแรก
const LazyHeavyResource = (() => {
  let instance = null;
  let initializationPromise = null;
  
  return {
    // Synchronous version
    getInstance() {
      if (!instance) {
        instance = new HeavyResource();
      }
      return instance;
    },
    
    // Async version (สำหรับ async initialization)
    async getInstanceAsync() {
      if (instance) return instance;
      
      // ป้องกัน race condition
      if (initializationPromise) {
        return initializationPromise;
      }
      
      initializationPromise = new Promise(async (resolve) => {
        await new Promise(r => setTimeout(r, 100)); // จำลอง async init
        instance = new HeavyResource();
        resolve(instance);
        initializationPromise = null;
      });
      
      return initializationPromise;
    },
    
    // สำหรับ testing
    reset() {
      instance = null;
      initializationPromise = null;
    }
  };
})();

// ใช้งาน
console.log('Before first access - no instance created');
const resource1 = LazyHeavyResource.getInstance();
const resource2 = LazyHeavyResource.getInstance();
console.log(resource1 === resource2); // true

// ========================
// Singleton ด้วย WeakRef (Node 14+)
// ========================

class CacheManager {
  #cache = new Map();
  #maxSize;
  #hitCount = 0;
  #missCount = 0;
  
  static #instance = null;
  
  constructor(maxSize = 100) {
    if (CacheManager.#instance) {
      return CacheManager.#instance;
    }
    
    this.#maxSize = maxSize;
    CacheManager.#instance = this;
  }
  
  get(key) {
    if (this.#cache.has(key)) {
      this.#hitCount++;
      return this.#cache.get(key);
    }
    this.#missCount++;
    return null;
  }
  
  set(key, value, ttl = 60000) {
    if (this.#cache.size >= this.#maxSize) {
      // Remove oldest entry (LRU-like)
      const firstKey = this.#cache.keys().next().value;
      this.#cache.delete(firstKey);
    }
    
    const entry = {
      value,
      expiresAt: Date.now() + ttl,
      createdAt: Date.now()
    };
    
    this.#cache.set(key, entry);
    
    // Auto expire
    setTimeout(() => {
      this.#cache.delete(key);
    }, ttl);
  }
  
  delete(key) {
    return this.#cache.delete(key);
  }
  
  clear() {
    this.#cache.clear();
  }
  
  getStats() {
    const totalRequests = this.#hitCount + this.#missCount;
    return {
      size: this.#cache.size,
      maxSize: this.#maxSize,
      hitCount: this.#hitCount,
      missCount: this.#missCount,
      hitRate: totalRequests > 0 
        ? `${((this.#hitCount / totalRequests) * 100).toFixed(1)}%`
        : '0%'
    };
  }
}

// ใช้งาน
const cache1 = new CacheManager(50);
const cache2 = new CacheManager(100); // จะ return instance เดิม

console.log(cache1 === cache2); // true

cache1.set('user:1', { name: 'สมชาย', email: 'test@example.com' });
console.log(cache2.get('user:1')); // จะเจอ key นี้ เพราะเป็น instance เดียวกัน
console.log(cache2.getStats());
```

---

## Step 1007: Factory Pattern - Advanced

```javascript
// ========================
// Factory Pattern สำหรับสร้าง UI Components
// ========================

class UIComponent {
  constructor(props) {
    this.id = props.id || `comp-${Math.random().toString(36).substr(2, 6)}`;
    this.className = props.className || '';
    this.children = props.children || [];
    this.events = {};
  }
  
  on(event, handler) {
    if (!this.events[event]) {
      this.events[event] = [];
    }
    this.events[event].push(handler);
    return this;
  }
  
  emit(event, data) {
    (this.events[event] || []).forEach(handler => handler(data));
  }
  
  render() {
    throw new Error('render() must be implemented');
  }
}

class ButtonComponent extends UIComponent {
  constructor(props) {
    super(props);
    this.label = props.label || 'Click me';
    this.variant = props.variant || 'primary';
    this.disabled = props.disabled || false;
  }
  
  render() {
    return `<button 
      id="${this.id}" 
      class="btn btn-${this.variant} ${this.className}" 
      ${this.disabled ? 'disabled' : ''}
    >${this.label}</button>`;
  }
}

class InputComponent extends UIComponent {
  constructor(props) {
    super(props);
    this.type = props.type || 'text';
    this.placeholder = props.placeholder || '';
    this.value = props.value || '';
    this.required = props.required || false;
  }
  
  render() {
    return `<input 
      id="${this.id}"
      type="${this.type}"
      class="input ${this.className}"
      placeholder="${this.placeholder}"
      value="${this.value}"
      ${this.required ? 'required' : ''}
    >`;
  }
}

class SelectComponent extends UIComponent {
  constructor(props) {
    super(props);
    this.options = props.options || [];
    this.value = props.value || '';
    this.placeholder = props.placeholder || 'เลือก...';
  }
  
  render() {
    const options = [
      `<option value="">${this.placeholder}</option>`,
      ...this.options.map(opt => 
        `<option value="${opt.value}" ${opt.value === this.value ? 'selected' : ''}>${opt.label}</option>`
      )
    ].join('\n');
    
    return `<select id="${this.id}" class="select ${this.className}">\n${options}\n</select>`;
  }
}

class CardComponent extends UIComponent {
  constructor(props) {
    super(props);
    this.title = props.title || '';
    this.content = props.content || '';
    this.footer = props.footer || '';
    this.image = props.image || null;
  }
  
  render() {
    const imageHtml = this.image 
      ? `<img src="${this.image}" class="card-image" alt="${this.title}">` 
      : '';
    
    const footerHtml = this.footer 
      ? `<div class="card-footer">${this.footer}</div>`
      : '';
    
    return `
<div id="${this.id}" class="card ${this.className}">
  ${imageHtml}
  <div class="card-body">
    <h3 class="card-title">${this.title}</h3>
    <p class="card-content">${this.content}</p>
  </div>
  ${footerHtml}
</div>`.trim();
  }
}

// Component Factory
class ComponentFactory {
  static _registry = new Map([
    ['button', ButtonComponent],
    ['input', InputComponent],
    ['select', SelectComponent],
    ['card', CardComponent]
  ]);
  
  static register(type, ComponentClass) {
    this._registry.set(type, ComponentClass);
  }
  
  static create(type, props = {}) {
    const ComponentClass = this._registry.get(type);
    if (!ComponentClass) {
      throw new Error(`Unknown component type: ${type}. Available: ${[...this._registry.keys()].join(', ')}`);
    }
    return new ComponentClass(props);
  }
  
  static createForm(schema) {
    return schema.fields.map(fieldConfig => 
      this.create(fieldConfig.type, fieldConfig)
    );
  }
}

// ใช้งาน
const button = ComponentFactory.create('button', {
  label: 'บันทึก',
  variant: 'success',
  className: 'mt-4'
});

const emailInput = ComponentFactory.create('input', {
  type: 'email',
  placeholder: 'กรอกอีเมล',
  required: true
});

const roleSelect = ComponentFactory.create('select', {
  options: [
    { value: 'admin', label: 'ผู้ดูแลระบบ' },
    { value: 'user', label: 'ผู้ใช้ทั่วไป' },
    { value: 'moderator', label: 'ผู้ดูแล' }
  ],
  placeholder: 'เลือกบทบาท'
});

console.log(button.render());
console.log(emailInput.render());
console.log(roleSelect.render());

// สร้าง form จาก schema
const formFields = ComponentFactory.createForm({
  fields: [
    { type: 'input', id: 'name', placeholder: 'ชื่อ-นามสกุล', required: true },
    { type: 'input', id: 'email', type: 'email', placeholder: 'อีเมล', required: true },
    { type: 'select', id: 'role', options: [{ value: 'user', label: 'User' }] },
    { type: 'button', label: 'ส่งข้อมูล', variant: 'primary' }
  ]
});

formFields.forEach(field => console.log(field.render()));
```

---

## Step 1008: Builder Pattern - Configuration Builder

```javascript
// ========================
// Configuration Builder - Express-like
// ========================

class AppConfig {
  constructor(config) {
    Object.assign(this, config);
    Object.freeze(this); // ทำให้ immutable
  }
}

class AppConfigBuilder {
  constructor() {
    this._config = {
      name: 'MyApp',
      version: '1.0.0',
      environment: 'development',
      server: {
        host: 'localhost',
        port: 3000,
        cors: false,
        compression: false,
        rateLimit: null
      },
      database: {
        url: null,
        maxConnections: 5,
        timeout: 30000,
        retry: {
          times: 3,
          interval: 1000
        }
      },
      auth: {
        enabled: false,
        secret: null,
        expiresIn: '24h',
        algorithm: 'HS256'
      },
      cache: {
        enabled: false,
        ttl: 3600,
        max: 100
      },
      logging: {
        level: 'info',
        format: 'json',
        file: null
      },
      middleware: [],
      plugins: []
    };
  }
  
  // App info
  name(name) {
    this._config.name = name;
    return this;
  }
  
  version(version) {
    this._config.version = version;
    return this;
  }
  
  environment(env) {
    const validEnvs = ['development', 'staging', 'production', 'test'];
    if (!validEnvs.includes(env)) {
      throw new Error(`Invalid environment: ${env}`);
    }
    this._config.environment = env;
    return this;
  }
  
  // Server config
  port(port) {
    this._config.server.port = port;
    return this;
  }
  
  host(host) {
    this._config.server.host = host;
    return this;
  }
  
  enableCors(options = true) {
    this._config.server.cors = options;
    return this;
  }
  
  enableCompression() {
    this._config.server.compression = true;
    return this;
  }
  
  rateLimit(requests, windowMs) {
    this._config.server.rateLimit = { requests, windowMs };
    return this;
  }
  
  // Database config
  database(url, options = {}) {
    this._config.database = {
      ...this._config.database,
      url,
      ...options
    };
    return this;
  }
  
  // Auth config
  enableAuth(secret, options = {}) {
    this._config.auth = {
      enabled: true,
      secret,
      expiresIn: options.expiresIn || '24h',
      algorithm: options.algorithm || 'HS256'
    };
    return this;
  }
  
  // Cache config
  enableCache(options = {}) {
    this._config.cache = {
      enabled: true,
      ttl: options.ttl || 3600,
      max: options.max || 100
    };
    return this;
  }
  
  // Logging config
  logging(level, format = 'json', file = null) {
    this._config.logging = { level, format, file };
    return this;
  }
  
  // Middleware
  use(middleware) {
    this._config.middleware.push(middleware);
    return this;
  }
  
  // Plugin
  plugin(plugin) {
    this._config.plugins.push(plugin);
    return this;
  }
  
  // Validation
  validate() {
    const errors = [];
    
    if (this._config.environment === 'production') {
      if (!this._config.database.url) {
        errors.push('Database URL required in production');
      }
      if (this._config.auth.enabled && !this._config.auth.secret) {
        errors.push('Auth secret required in production');
      }
    }
    
    if (errors.length > 0) {
      throw new Error(`Config validation failed:\n${errors.join('\n')}`);
    }
    
    return true;
  }
  
  build() {
    this.validate();
    return new AppConfig(this._config);
  }
}

// ใช้งาน
const devConfig = new AppConfigBuilder()
  .name('My Thai App')
  .version('2.1.0')
  .environment('development')
  .port(4000)
  .enableCors({ origin: 'http://localhost:3000' })
  .enableCompression()
  .rateLimit(100, 60000)
  .database('postgresql://localhost/mydb_dev', {
    maxConnections: 5
  })
  .enableAuth('dev-secret-key', { expiresIn: '7d' })
  .enableCache({ ttl: 300, max: 50 })
  .logging('debug', 'pretty')
  .build();

console.log('Dev Config:', JSON.stringify(devConfig, null, 2));
```

---

## Step 1009: Advanced Factory - Plugin System

```javascript
// ========================
// Plugin System ด้วย Factory Pattern
// ========================

class PluginRegistry {
  constructor() {
    this._plugins = new Map();
    this._hooks = new Map();
  }
  
  // ลงทะเบียน plugin
  register(name, pluginFactory, options = {}) {
    if (this._plugins.has(name)) {
      console.warn(`Plugin '${name}' already registered, overwriting...`);
    }
    
    this._plugins.set(name, {
      factory: pluginFactory,
      options,
      instance: null
    });
    
    return this;
  }
  
  // สร้าง plugin instance
  create(name, config = {}) {
    const plugin = this._plugins.get(name);
    if (!plugin) {
      throw new Error(`Plugin '${name}' not found`);
    }
    
    if (!plugin.instance) {
      plugin.instance = plugin.factory({
        ...plugin.options,
        ...config
      });
    }
    
    return plugin.instance;
  }
  
  // Hook system
  addHook(hookName, handler) {
    if (!this._hooks.has(hookName)) {
      this._hooks.set(hookName, []);
    }
    this._hooks.get(hookName).push(handler);
    return this;
  }
  
  async runHook(hookName, context) {
    const handlers = this._hooks.get(hookName) || [];
    let result = context;
    
    for (const handler of handlers) {
      result = await handler(result) || result;
    }
    
    return result;
  }
  
  getPluginNames() {
    return [...this._plugins.keys()];
  }
}

// Plugin examples
const createLoggerPlugin = (config) => ({
  name: 'logger',
  log(message, level = 'INFO') {
    const timestamp = new Date().toISOString();
    const format = config.format === 'json' 
      ? JSON.stringify({ timestamp, level, message })
      : `[${timestamp}] [${level}] ${message}`;
    console.log(format);
  },
  info(msg) { this.log(msg, 'INFO'); },
  error(msg) { this.log(msg, 'ERROR'); }
});

const createCachePlugin = (config) => {
  const cache = new Map();
  const maxSize = config.maxSize || 100;
  
  return {
    name: 'cache',
    get(key) {
      return cache.get(key);
    },
    set(key, value) {
      if (cache.size >= maxSize) {
        const firstKey = cache.keys().next().value;
        cache.delete(firstKey);
      }
      cache.set(key, { value, timestamp: Date.now() });
    },
    has(key) {
      return cache.has(key);
    },
    clear() {
      cache.clear();
    },
    getSize() {
      return cache.size;
    }
  };
};

const createAuthPlugin = (config) => ({
  name: 'auth',
  generateToken(payload) {
    // จำลอง JWT generation
    const header = btoa(JSON.stringify({ alg: 'HS256', typ: 'JWT' }));
    const body = btoa(JSON.stringify({
      ...payload,
      iat: Date.now(),
      exp: Date.now() + (config.expiresIn || 3600000)
    }));
    const signature = btoa(`${header}.${body}.${config.secret}`);
    return `${header}.${body}.${signature}`;
  },
  
  verifyToken(token) {
    try {
      const [header, body, sig] = token.split('.');
      const payload = JSON.parse(atob(body));
      
      if (payload.exp < Date.now()) {
        return { valid: false, error: 'Token expired' };
      }
      
      const expectedSig = btoa(`${header}.${body}.${config.secret}`);
      if (sig !== expectedSig) {
        return { valid: false, error: 'Invalid signature' };
      }
      
      return { valid: true, payload };
    } catch (e) {
      return { valid: false, error: 'Invalid token format' };
    }
  }
});

// สร้างและใช้งาน Plugin Registry
const registry = new PluginRegistry();

registry
  .register('logger', createLoggerPlugin, { format: 'pretty' })
  .register('cache', createCachePlugin, { maxSize: 200 })
  .register('auth', createAuthPlugin, { 
    secret: 'my-secret-key',
    expiresIn: 3600000 
  });

// ใช้งาน plugins
const logger = registry.create('logger');
const cache = registry.create('cache');
const auth = registry.create('auth');

logger.info('Application started');

cache.set('user:1', { name: 'สมชาย' });
console.log(cache.get('user:1'));

const token = auth.generateToken({ userId: 1, role: 'admin' });
console.log('Token generated:', token.substring(0, 50) + '...');

const verification = auth.verifyToken(token);
console.log('Token valid:', verification.valid);
```

---

## Step 1010: รวม Patterns และ Best Practices

```javascript
// ========================
// รวม Creational Patterns ทั้งหมด
// ========================

// Application ที่ใช้ทุก pattern รวมกัน

// 1. Singleton - App Configuration
class AppConfiguration {
  static _instance = null;
  
  constructor() {
    if (AppConfiguration._instance) {
      return AppConfiguration._instance;
    }
    this._settings = {};
    AppConfiguration._instance = this;
  }
  
  set(key, value) {
    this._settings[key] = value;
    return this;
  }
  
  get(key) {
    return this._settings[key];
  }
  
  static getInstance() {
    if (!AppConfiguration._instance) {
      new AppConfiguration();
    }
    return AppConfiguration._instance;
  }
}

// 2. Abstract Factory - Payment System
class PaymentProcessor {
  charge(amount) { throw new Error('Not implemented'); }
  refund(transactionId) { throw new Error('Not implemented'); }
  getBalance() { throw new Error('Not implemented'); }
}

class StripeProcessor extends PaymentProcessor {
  constructor(config) {
    super();
    this.apiKey = config.apiKey;
    this.name = 'Stripe';
  }
  
  charge(amount) {
    return { success: true, transactionId: `stripe_${Date.now()}`, amount, processor: this.name };
  }
  
  refund(transactionId) {
    return { success: true, transactionId: `refund_${transactionId}`, processor: this.name };
  }
  
  getBalance() {
    return { balance: 50000, currency: 'THB', processor: this.name };
  }
}

class OmiseProcessor extends PaymentProcessor {
  constructor(config) {
    super();
    this.publicKey = config.publicKey;
    this.secretKey = config.secretKey;
    this.name = 'Omise';
  }
  
  charge(amount) {
    return { success: true, transactionId: `omise_${Date.now()}`, amount, processor: this.name };
  }
  
  refund(transactionId) {
    return { success: true, transactionId: `refund_${transactionId}`, processor: this.name };
  }
  
  getBalance() {
    return { balance: 25000, currency: 'THB', processor: this.name };
  }
}

class PaymentFactory {
  static create(type, config) {
    const processors = {
      stripe: StripeProcessor,
      omise: OmiseProcessor
    };
    
    const ProcessorClass = processors[type.toLowerCase()];
    if (!ProcessorClass) {
      throw new Error(`Unsupported payment processor: ${type}`);
    }
    
    return new ProcessorClass(config);
  }
}

// 3. Builder - Order Builder
class Order {
  constructor(builder) {
    this.orderId = `ORD-${Date.now()}`;
    this.customerId = builder.customerId;
    this.items = builder.items;
    this.shippingAddress = builder.shippingAddress;
    this.paymentMethod = builder.paymentMethod;
    this.discount = builder.discount;
    this.notes = builder.notes;
    this.createdAt = new Date().toISOString();
    
    this.subtotal = this.calculateSubtotal();
    this.total = this.calculateTotal();
  }
  
  calculateSubtotal() {
    return this.items.reduce((sum, item) => sum + (item.price * item.quantity), 0);
  }
  
  calculateTotal() {
    return this.subtotal - (this.subtotal * this.discount / 100);
  }
  
  toString() {
    return `Order #${this.orderId} - Total: ฿${this.total.toLocaleString()}`;
  }
}

class OrderBuilder {
  constructor(customerId) {
    this.customerId = customerId;
    this.items = [];
    this.shippingAddress = null;
    this.paymentMethod = null;
    this.discount = 0;
    this.notes = '';
  }
  
  addItem(product, quantity, price) {
    this.items.push({ product, quantity, price });
    return this;
  }
  
  shipTo(address) {
    this.shippingAddress = address;
    return this;
  }
  
  payWith(method) {
    this.paymentMethod = method;
    return this;
  }
  
  applyDiscount(percent) {
    this.discount = percent;
    return this;
  }
  
  withNotes(notes) {
    this.notes = notes;
    return this;
  }
  
  build() {
    if (this.items.length === 0) throw new Error('Order must have at least one item');
    if (!this.shippingAddress) throw new Error('Shipping address required');
    if (!this.paymentMethod) throw new Error('Payment method required');
    return new Order(this);
  }
}

// 4. Prototype - Product Template
class Product {
  constructor(data) {
    Object.assign(this, data);
  }
  
  clone() {
    return new Product({ ...this });
  }
  
  applyDiscount(percent) {
    const cloned = this.clone();
    cloned.price = this.price * (1 - percent / 100);
    cloned.originalPrice = this.price;
    cloned.discountPercent = percent;
    return cloned;
  }
}

// ========================
// ใช้งานรวมกัน
// ========================

// 1. Config
const appConfig = AppConfiguration.getInstance();
appConfig
  .set('app.name', 'ThaiShop')
  .set('app.currency', 'THB')
  .set('payment.defaultProcessor', 'omise');

// 2. Factory สร้าง payment processor
const paymentProcessor = PaymentFactory.create(
  appConfig.get('payment.defaultProcessor'),
  { publicKey: 'pkey_test_123', secretKey: 'skey_test_456' }
);

// 3. Prototype สำหรับ products
const baseProduct = new Product({
  id: 'TSHIRT-001',
  name: 'เสื้อยืด JavaScript',
  price: 399,
  category: 'clothing',
  inStock: true
});

const saleProduct = baseProduct.applyDiscount(20);

// 4. Builder สร้าง order
const order = new OrderBuilder('customer-123')
  .addItem(baseProduct.name, 2, baseProduct.price)
  .addItem(saleProduct.name, 1, saleProduct.price)
  .shipTo({
    street: '123 ถนนสุขุมวิท',
    city: 'กรุงเทพฯ',
    postalCode: '10110'
  })
  .payWith('omise_creditcard')
  .applyDiscount(5)
  .withNotes('ส่งด่วน')
  .build();

console.log(order.toString());
console.log('Payment:', JSON.stringify(paymentProcessor.charge(order.total), null, 2));

// 5. Singleton Logger
const logger = new Logger();
logger.info('Order created', { orderId: order.orderId });
logger.info('Payment processed', { amount: order.total });
```

---

## แบบฝึกหัด (Exercises)

### ระดับ Easy

**Exercise 1:** Singleton Pattern
```javascript
// สร้าง EventBus Singleton ที่:
// - มีเพียง instance เดียว
// - มีเมธอด subscribe(event, callback)
// - มีเมธอด publish(event, data)
// - มีเมธอด unsubscribe(event, callback)
// - แสดงผลว่าทุก instance เป็น object เดียวกัน

class EventBus {
  // TODO: implement singleton
  
  subscribe(event, callback) {
    // TODO
  }
  
  publish(event, data) {
    // TODO
  }
  
  unsubscribe(event, callback) {
    // TODO
  }
}

// Test
const bus1 = new EventBus();
const bus2 = new EventBus();
console.log(bus1 === bus2); // ต้อง true

bus1.subscribe('userLogin', (data) => console.log('User logged in:', data));
bus2.publish('userLogin', { userId: 1 }); // ต้อง trigger callback ด้านบน
```

**Exercise 2:** Simple Factory
```javascript
// สร้าง VehicleFactory ที่สร้าง:
// - Car: มี speed, fuel, doors
// - Motorcycle: มี speed, fuel, type ('sport', 'cruiser')
// - Truck: มี speed, fuel, capacity
// แต่ละ vehicle ต้องมีเมธอด describe()

class VehicleFactory {
  // TODO: implement
}

// Test
const car = VehicleFactory.create('car', { speed: 200 });
const moto = VehicleFactory.create('motorcycle', { type: 'sport' });
const truck = VehicleFactory.create('truck', { capacity: 5000 });

console.log(car.describe());
console.log(moto.describe());
console.log(truck.describe());
```

### ระดับ Medium

**Exercise 3:** Builder Pattern
```javascript
// สร้าง EmailBuilder ที่มี:
// .from(email) 
// .to(...emails)
// .cc(...emails)
// .bcc(...emails)
// .subject(text)
// .body(html)
// .attach(filename, content)
// .send() -> จำลองการส่ง
// Validation: from, to, subject, body ต้องมี

class Email {
  // TODO
}

class EmailBuilder {
  // TODO
}

// Test
const email = new EmailBuilder()
  .from('sender@example.com')
  .to('recipient@example.com', 'another@example.com')
  .subject('ทดสอบ Email Builder')
  .body('<p>สวัสดี!</p>')
  .attach('report.pdf', 'base64content...')
  .build();

email.send();
```

**Exercise 4:** Prototype Pattern
```javascript
// สร้าง Character system สำหรับ RPG game:
// - Character มี: name, class, stats, skills, equipment
// - มีเมธอด clone() ที่ deep clone ทุกอย่าง
// - มีเมธอด level up ที่ไม่กระทบ original
// - สร้าง Character templates และ spawn characters จาก templates

class Character {
  // TODO
  
  clone() {
    // Deep clone
  }
  
  levelUp() {
    // Return cloned and leveled up character
  }
}

// Test
const warriorTemplate = new Character({
  class: 'Warrior',
  stats: { hp: 100, mp: 30, attack: 15 },
  skills: ['Slash', 'Shield Block'],
  equipment: ['Iron Sword', 'Wooden Shield']
});

const warrior1 = warriorTemplate.clone();
warrior1.name = 'สมหมาย';
warrior1.levelUp().levelUp();

const warrior2 = warriorTemplate.clone();
warrior2.name = 'สมชาย';

// warrior1 และ warrior2 ต้องแยกจากกัน ไม่กระทบ template
```

### ระดับ Hard

**Exercise 5:** รวมทุก Patterns
```javascript
// สร้าง Game Engine ขนาดเล็กที่ใช้:
// - Singleton: GameManager (จัดการ state ของเกม)
// - Factory: EntityFactory (สร้าง game entities)
// - Builder: LevelBuilder (สร้าง level)
// - Prototype: EntityTemplate (clone entities)
// - Object Pool: BulletPool (จัดการ bullets)

class GameManager {
  // Singleton
  // state: running, paused, game_over
  // score, level, lives
}

class EntityFactory {
  // สร้าง: player, enemy, powerup, obstacle
}

class LevelBuilder {
  // สร้าง level ด้วย:
  // .setSize(width, height)
  // .addSpawnPoint(x, y, entityType)
  // .addObstacles(count, type)
  // .setBackground(image)
  // .setTimeLimit(seconds)
  // .build()
}

class EntityTemplate {
  // Prototype สำหรับ clone entities
}

class BulletPool {
  // Object Pool สำหรับ bullets
  // acquire(), release(), getActiveCount()
}

// ใช้งาน
const game = GameManager.getInstance();
const factory = new EntityFactory();
const level = new LevelBuilder()
  .setSize(800, 600)
  .addSpawnPoint(100, 300, 'player')
  .addSpawnPoint(700, 100, 'enemy')
  .addObstacles(10, 'rock')
  .setTimeLimit(120)
  .build();
```

---

## สรุป Creational Patterns

| Pattern | ใช้เมื่อ | ข้อดี | ข้อเสีย |
|---------|---------|------|--------|
| Singleton | ต้องการ instance เดียว | ประหยัดทรัพยากร | ยาก test, global state |
| Factory | ไม่รู้ว่าจะ new อะไร | ยืดหยุ่น, ง่ายขยาย | เพิ่ม class |
| Abstract Factory | กลุ่ม objects ที่เกี่ยวกัน | สม่ำเสมอ | ซับซ้อน |
| Builder | object ซับซ้อนมาก params | อ่านง่าย, flexible | เพิ่มโค้ด |
| Prototype | clone object แทน new | ประสิทธิภาพสูง | deep clone ยุ่งยาก |
| Object Pool | สร้าง/ทำลายมีต้นทุนสูง | ประสิทธิภาพสูง | จัดการซับซ้อน |

```javascript
// Quick Reference
// Singleton
const instance = MySingleton.getInstance();

// Factory
const obj = Factory.create('type', options);

// Builder  
const product = new ProductBuilder()
  .setX(...)
  .setY(...)
  .build();

// Prototype
const clone = original.clone();

// Object Pool
const pool = new Pool(factory, { min: 2, max: 10 });
const obj = await pool.acquire();
pool.release(obj);
```

---

**ขั้นตอนต่อไป:** ใน Part 52 เราจะเรียนรู้ **Structural Patterns** ที่เกี่ยวกับการจัดโครงสร้างของ objects และ classes
