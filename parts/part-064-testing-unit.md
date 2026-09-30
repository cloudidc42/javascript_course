# Part 64: Testing - Unit Tests (Steps 1251-1270)

## บทนำ

การ testing เป็นทักษะสำคัญที่นักพัฒนาทุกคนต้องมี บทนี้จะสอนการเขียน Unit Tests ด้วย Jest ตั้งแต่พื้นฐานจนถึงเทคนิคขั้นสูง รวมถึง mocking และ TDD

---

## Step 1251: ทำไมต้อง Testing?

```javascript
// ปัญหาที่เกิดขึ้นเมื่อไม่มี tests

// function ที่ดูเหมือนทำงานถูก แต่มี bugs ซ่อนอยู่
function calculateDiscount(price, discountPercent) {
  return price - (price * discountPercent / 100);
}

// ทดสอบบางกรณี
console.log(calculateDiscount(100, 10)); // 90 ✓
console.log(calculateDiscount(200, 20)); // 160 ✓

// แต่ถ้า...
console.log(calculateDiscount(100, 110)); // -10 ❌ (ราคาติดลบ!)
console.log(calculateDiscount(-100, 10)); // -90 ❌ (ราคาติดลบ!)
console.log(calculateDiscount(100, 0));   // 100 ✓ แต่ logic ถูกไหม?

// ถ้ามี tests:
// test('should not allow discount > 100%', () => {
//   expect(() => calculateDiscount(100, 110)).toThrow();
// });

// ประโยชน์ของ Tests
// 1. หา bugs เร็วขึ้น
// 2. Refactor โดยมั่นใจ
// 3. Documentation ที่ "ทำงานได้จริง"
// 4. Improve code design (code ที่ test ได้ มักออกแบบดีกว่า)
// 5. ลด regression bugs
```

---

## Step 1252: Testing Pyramid

```javascript
// Testing Pyramid
//
//         /\
//        /  \
//       / E2E\      <- น้อยที่สุด, ช้าที่สุด, ราคาแพงที่สุด
//      /------\
//     /        \
//    /Integration\  <- ปานกลาง
//   /------------\
//  /              \
// /   Unit Tests   \ <- มากที่สุด, เร็วที่สุด, ราคาถูกที่สุด
// /________________\

// Unit Tests:
// - ทดสอบ function/module แยกกัน
// - เร็วมาก (milliseconds)
// - ควรมีมากที่สุด
// - ง่ายต่อการ debug

// Integration Tests:
// - ทดสอบหลาย components ทำงานร่วมกัน
// - เช่น API + Database
// - ช้ากว่า Unit Tests
// - ควรมีพอเหมาะ

// E2E (End-to-End) Tests:
// - ทดสอบ user flow จริง
// - เช่น เปิดเบราว์เซอร์ กรอกฟอร์ม ส่ง
// - ช้าที่สุด
// - ควรมีน้อยที่สุด

// 70/20/10 Rule (โดยประมาณ)
// 70% Unit Tests
// 20% Integration Tests
// 10% E2E Tests
```

---

## Step 1253: Jest Setup

```javascript
// ติดตั้ง: npm install --save-dev jest

// package.json
/*
{
  "scripts": {
    "test": "jest",
    "test:watch": "jest --watch",
    "test:coverage": "jest --coverage"
  },
  "jest": {
    "testEnvironment": "node",
    "testMatch": ["**/__tests__/**/*.js", "**/*.test.js", "**/*.spec.js"],
    "collectCoverageFrom": [
      "src/**/*.js",
      "!src/**/*.test.js"
    ],
    "coverageThreshold": {
      "global": {
        "branches": 80,
        "functions": 80,
        "lines": 80,
        "statements": 80
      }
    }
  }
}
*/

// Jest configuration file (jest.config.js)
module.exports = {
  testEnvironment: 'node',
  
  // Transform files (สำหรับ ES Modules)
  // transform: {
  //   '^.+\\.js$': 'babel-jest'
  // },
  
  // Setup files
  setupFilesAfterFramework: ['./jest.setup.js'],
  
  // Module name mapping
  moduleNameMapper: {
    '^@/(.*)$': '<rootDir>/src/$1'
  },
  
  // Timeout
  testTimeout: 10000,
  
  // Coverage
  collectCoverage: false,
  coverageDirectory: 'coverage',
  coverageReporters: ['text', 'lcov', 'html']
};
```

---

## Step 1254: Writing Your First Test

```javascript
// src/math.js
function add(a, b) {
  return a + b;
}

function subtract(a, b) {
  return a - b;
}

function multiply(a, b) {
  return a * b;
}

function divide(a, b) {
  if (b === 0) {
    throw new Error('หารด้วย 0 ไม่ได้');
  }
  return a / b;
}

module.exports = { add, subtract, multiply, divide };

// src/math.test.js
const { add, subtract, multiply, divide } = require('./math');

// describe - จัดกลุ่ม tests ที่เกี่ยวข้อง
describe('Math functions', () => {
  
  // test หรือ it - เขียน test case แต่ละอัน
  test('add ควรบวกตัวเลขสองตัว', () => {
    // expect(actual).matcher(expected)
    expect(add(1, 2)).toBe(3);
    expect(add(-1, 1)).toBe(0);
    expect(add(0, 0)).toBe(0);
    expect(add(1.5, 2.5)).toBe(4);
  });
  
  it('subtract ควรลบตัวเลขสองตัว', () => {
    expect(subtract(5, 3)).toBe(2);
    expect(subtract(0, 5)).toBe(-5);
  });
  
  test('multiply ควรคูณตัวเลขสองตัว', () => {
    expect(multiply(3, 4)).toBe(12);
    expect(multiply(-2, 3)).toBe(-6);
    expect(multiply(0, 100)).toBe(0);
  });
  
  describe('divide', () => {
    test('ควรหารตัวเลขสองตัว', () => {
      expect(divide(10, 2)).toBe(5);
      expect(divide(7, 2)).toBe(3.5);
    });
    
    test('ควร throw error เมื่อหารด้วย 0', () => {
      expect(() => divide(10, 0)).toThrow('หารด้วย 0 ไม่ได้');
      expect(() => divide(10, 0)).toThrow(Error);
    });
  });
});
```

---

## Step 1255: Matchers

```javascript
// Jest มี matchers หลายประเภท

describe('Jest Matchers', () => {
  
  // toBe - ใช้ === (strict equality)
  test('toBe', () => {
    expect(1 + 1).toBe(2);
    expect('hello').toBe('hello');
    expect(true).toBe(true);
  });
  
  // toEqual - deep equality (สำหรับ objects/arrays)
  test('toEqual', () => {
    expect({ a: 1, b: 2 }).toEqual({ a: 1, b: 2 });
    expect([1, 2, 3]).toEqual([1, 2, 3]);
    
    // toBe จะ fail สำหรับ objects
    expect({ a: 1 }).not.toBe({ a: 1 }); // คนละ reference
    expect({ a: 1 }).toEqual({ a: 1 }); // ✓
  });
  
  // Truthiness
  test('truthiness', () => {
    expect(true).toBeTruthy();
    expect(1).toBeTruthy();
    expect('hello').toBeTruthy();
    
    expect(false).toBeFalsy();
    expect(0).toBeFalsy();
    expect('').toBeFalsy();
    expect(null).toBeFalsy();
    expect(undefined).toBeFalsy();
    
    expect(null).toBeNull();
    expect(undefined).toBeUndefined();
    expect('hello').toBeDefined();
    expect(undefined).not.toBeDefined();
  });
  
  // Numbers
  test('numbers', () => {
    expect(10).toBeGreaterThan(5);
    expect(10).toBeGreaterThanOrEqual(10);
    expect(5).toBeLessThan(10);
    expect(5).toBeLessThanOrEqual(5);
    
    // Floating point - ใช้ toBeCloseTo
    expect(0.1 + 0.2).not.toBe(0.3); // floating point ไม่แน่นอน
    expect(0.1 + 0.2).toBeCloseTo(0.3, 5); // 5 decimal places
  });
  
  // Strings
  test('strings', () => {
    expect('hello world').toContain('world');
    expect('hello world').toMatch(/world/);
    expect('hello world').toMatch('world');
    expect('hello').toHaveLength(5);
  });
  
  // Arrays
  test('arrays', () => {
    const fruits = ['apple', 'banana', 'orange'];
    
    expect(fruits).toContain('banana');
    expect(fruits).toHaveLength(3);
    expect(fruits).toEqual(expect.arrayContaining(['apple', 'orange']));
  });
  
  // Objects
  test('objects', () => {
    const user = { id: 1, name: 'Alice', email: 'alice@example.com' };
    
    expect(user).toHaveProperty('name');
    expect(user).toHaveProperty('name', 'Alice');
    expect(user).toHaveProperty('id', 1);
    expect(user).toMatchObject({ name: 'Alice' }); // partial match
  });
  
  // Errors
  test('errors', () => {
    const throwError = () => { throw new Error('test error'); };
    const throwTypeError = () => { throw new TypeError('type error'); };
    
    expect(throwError).toThrow();
    expect(throwError).toThrow('test error');
    expect(throwError).toThrow(Error);
    expect(throwTypeError).toThrow(TypeError);
    expect(throwTypeError).toThrow('type error');
  });
  
  // not - ใช้ negate matcher ใดก็ได้
  test('not', () => {
    expect(1).not.toBe(2);
    expect('hello').not.toContain('world');
    expect([1, 2, 3]).not.toContain(4);
  });
});
```

---

## Step 1256: Async Tests

```javascript
// Testing async functions

// 1. Return Promise
test('async ด้วย return promise', () => {
  return fetch('https://api.example.com/users')
    .then(res => res.json())
    .then(data => {
      expect(data).toBeDefined();
    });
});

// 2. async/await (แนะนำ)
test('async ด้วย async/await', async () => {
  const data = await fetch('https://api.example.com/users')
    .then(res => res.json());
  
  expect(data).toBeDefined();
});

// 3. done callback (เก่า ไม่แนะนำ)
test('async ด้วย done', (done) => {
  setTimeout(() => {
    expect(1 + 1).toBe(2);
    done();
  }, 100);
});

// 4. resolves/rejects
test('promise resolves', async () => {
  await expect(Promise.resolve('hello')).resolves.toBe('hello');
  await expect(Promise.resolve({ id: 1 })).resolves.toEqual({ id: 1 });
});

test('promise rejects', async () => {
  await expect(Promise.reject(new Error('failed'))).rejects.toThrow('failed');
  await expect(Promise.reject(new Error('error'))).rejects.toBeInstanceOf(Error);
});

// ตัวอย่างจริง
async function fetchUser(id) {
  if (!id) throw new Error('ต้องระบุ id');
  
  const response = await fetch(`/api/users/${id}`);
  if (!response.ok) {
    throw new Error(`HTTP error: ${response.status}`);
  }
  return response.json();
}

test('fetchUser ควรดึงข้อมูล user', async () => {
  // Mock fetch
  global.fetch = jest.fn().mockResolvedValue({
    ok: true,
    json: jest.fn().mockResolvedValue({ id: 1, name: 'Alice' })
  });
  
  const user = await fetchUser(1);
  expect(user).toEqual({ id: 1, name: 'Alice' });
});

test('fetchUser ควร throw เมื่อไม่มี id', async () => {
  await expect(fetchUser()).rejects.toThrow('ต้องระบุ id');
});
```

---

## Step 1257: Setup และ Teardown

```javascript
// beforeEach, afterEach, beforeAll, afterAll

describe('Database tests', () => {
  let db;
  let testUser;
  
  // ทำงานก่อน tests ทั้งหมดใน describe นี้ (ทำครั้งเดียว)
  beforeAll(async () => {
    db = await connectTestDB();
    console.log('เชื่อมต่อ test database แล้ว');
  });
  
  // ทำงานหลัง tests ทั้งหมดใน describe นี้ (ทำครั้งเดียว)
  afterAll(async () => {
    await db.close();
    console.log('ปิด test database แล้ว');
  });
  
  // ทำงานก่อนแต่ละ test
  beforeEach(async () => {
    // สร้างข้อมูลทดสอบ
    testUser = await db.users.create({
      name: 'Test User',
      email: `test-${Date.now()}@example.com`
    });
  });
  
  // ทำงานหลังแต่ละ test
  afterEach(async () => {
    // ทำความสะอาด
    if (testUser) {
      await db.users.delete(testUser.id);
    }
  });
  
  test('ควรสร้าง user สำเร็จ', async () => {
    expect(testUser).toBeDefined();
    expect(testUser.id).toBeDefined();
    expect(testUser.name).toBe('Test User');
  });
  
  test('ควรอัพเดต user สำเร็จ', async () => {
    const updated = await db.users.update(testUser.id, { name: 'Updated' });
    expect(updated.name).toBe('Updated');
  });
});

// Global setup/teardown
// jest.config.js
// globalSetup: './jest.globalSetup.js',
// globalTeardown: './jest.globalTeardown.js',

// jest.globalSetup.js
module.exports = async function() {
  // ทำงานก่อน test suite ทั้งหมด
  process.env.TEST_DB_URL = 'postgresql://test:test@localhost/testdb';
};

// jest.globalTeardown.js
module.exports = async function() {
  // ทำงานหลัง test suite ทั้งหมด
};
```

---

## Step 1258: Mocking - jest.fn()

```javascript
// Mock Functions ใช้สำหรับ:
// 1. แทนที่ dependencies ที่ expensive หรือ side effects
// 2. ควบคุม return values
// 3. ตรวจสอบว่า function ถูกเรียก

describe('jest.fn()', () => {
  test('สร้าง mock function พื้นฐาน', () => {
    const mockFn = jest.fn();
    
    // เรียกใช้
    mockFn('hello');
    mockFn('world');
    
    // ตรวจสอบการเรียก
    expect(mockFn).toHaveBeenCalled();
    expect(mockFn).toHaveBeenCalledTimes(2);
    expect(mockFn).toHaveBeenCalledWith('hello');
    expect(mockFn).toHaveBeenLastCalledWith('world');
    
    // ดู call history
    console.log(mockFn.mock.calls); // [['hello'], ['world']]
    console.log(mockFn.mock.calls[0]); // ['hello']
  });
  
  test('mock กับ return value', () => {
    const mockFn = jest.fn().mockReturnValue(42);
    
    expect(mockFn()).toBe(42);
    expect(mockFn()).toBe(42); // คืนค่าเดิมเสมอ
    
    // Return different values
    const mockFn2 = jest.fn()
      .mockReturnValueOnce(1)
      .mockReturnValueOnce(2)
      .mockReturnValue(99); // default
    
    expect(mockFn2()).toBe(1);
    expect(mockFn2()).toBe(2);
    expect(mockFn2()).toBe(99);
    expect(mockFn2()).toBe(99);
  });
  
  test('mock async function', async () => {
    const mockAsync = jest.fn()
      .mockResolvedValue({ id: 1, name: 'Alice' });
    
    const result = await mockAsync();
    expect(result).toEqual({ id: 1, name: 'Alice' });
    
    // Mock rejection
    const mockReject = jest.fn()
      .mockRejectedValue(new Error('ล้มเหลว'));
    
    await expect(mockReject()).rejects.toThrow('ล้มเหลว');
  });
  
  test('mock implementation', () => {
    const mockFn = jest.fn().mockImplementation((x) => x * 2);
    
    expect(mockFn(5)).toBe(10);
    expect(mockFn(3)).toBe(6);
  });
});
```

---

## Step 1259: Mocking Modules

```javascript
// jest.mock() - mock module ทั้งหมด

// src/emailService.js
const nodemailer = require('nodemailer');

async function sendWelcomeEmail(email, name) {
  const transporter = nodemailer.createTransport({
    host: 'smtp.example.com',
    port: 587,
    auth: { user: 'user', pass: 'pass' }
  });
  
  await transporter.sendMail({
    from: 'noreply@example.com',
    to: email,
    subject: `ยินดีต้อนรับ ${name}!`,
    text: `สวัสดี ${name} ขอบคุณที่สมัครสมาชิก`
  });
  
  return true;
}

module.exports = { sendWelcomeEmail };

// src/emailService.test.js
const nodemailer = require('nodemailer');
const { sendWelcomeEmail } = require('./emailService');

// Mock nodemailer module
jest.mock('nodemailer');

describe('EmailService', () => {
  let mockSendMail;
  let mockTransporter;
  
  beforeEach(() => {
    mockSendMail = jest.fn().mockResolvedValue({ messageId: '123' });
    mockTransporter = { sendMail: mockSendMail };
    
    nodemailer.createTransport.mockReturnValue(mockTransporter);
  });
  
  afterEach(() => {
    jest.clearAllMocks();
  });
  
  test('ควรส่ง email สำเร็จ', async () => {
    const result = await sendWelcomeEmail('alice@example.com', 'Alice');
    
    expect(result).toBe(true);
    expect(nodemailer.createTransport).toHaveBeenCalledTimes(1);
    expect(mockSendMail).toHaveBeenCalledWith({
      from: 'noreply@example.com',
      to: 'alice@example.com',
      subject: 'ยินดีต้อนรับ Alice!',
      text: expect.stringContaining('Alice')
    });
  });
  
  test('ควร throw error เมื่อส่ง email ล้มเหลว', async () => {
    mockSendMail.mockRejectedValue(new Error('SMTP error'));
    
    await expect(sendWelcomeEmail('test@example.com', 'Test'))
      .rejects.toThrow('SMTP error');
  });
});
```

---

## Step 1260: Spies

```javascript
// jest.spyOn() - monitor method บน existing object
// ต่างจาก jest.fn() ที่ spy ใช้ implementation เดิม

const userService = {
  async findUser(id) {
    // ดึงจาก database จริง
    return db.users.findById(id);
  },
  
  async updateUser(id, data) {
    const user = await this.findUser(id);
    return db.users.update(id, { ...user, ...data });
  }
};

test('updateUser ควรเรียก findUser', async () => {
  // Spy บน findUser
  const spy = jest.spyOn(userService, 'findUser')
    .mockResolvedValue({ id: 1, name: 'Alice' });
  
  await userService.updateUser(1, { name: 'Alice Updated' });
  
  expect(spy).toHaveBeenCalledWith(1);
  
  // Restore implementation เดิม
  spy.mockRestore();
});

// Spy บน built-in methods
test('spy บน console.log', () => {
  const consoleSpy = jest.spyOn(console, 'log').mockImplementation(() => {});
  
  function greet(name) {
    console.log(`สวัสดี ${name}`);
  }
  
  greet('Alice');
  
  expect(consoleSpy).toHaveBeenCalledWith('สวัสดี Alice');
  
  consoleSpy.mockRestore();
});

// Spy บน Date
test('spy บน Date', () => {
  const mockDate = new Date('2024-01-01T00:00:00.000Z');
  const dateSpy = jest.spyOn(global, 'Date').mockImplementation(() => mockDate);
  
  const result = new Date();
  expect(result).toEqual(mockDate);
  
  dateSpy.mockRestore();
});
```

---

## Step 1261: Testing Pure Functions

```javascript
// Pure Functions ทดสอบง่ายที่สุด

// src/utils/format.js
function formatDate(date) {
  if (!(date instanceof Date)) {
    throw new TypeError('ต้องเป็น Date object');
  }
  
  const year = date.getFullYear();
  const month = String(date.getMonth() + 1).padStart(2, '0');
  const day = String(date.getDate()).padStart(2, '0');
  
  return `${year}-${month}-${day}`;
}

function formatCurrency(amount, currency = 'THB') {
  if (typeof amount !== 'number') {
    throw new TypeError('amount ต้องเป็นตัวเลข');
  }
  
  return new Intl.NumberFormat('th-TH', {
    style: 'currency',
    currency
  }).format(amount);
}

function slugify(text) {
  return text
    .toLowerCase()
    .trim()
    .replace(/\s+/g, '-')
    .replace(/[^\w-]+/g, '');
}

function truncate(text, maxLength, suffix = '...') {
  if (text.length <= maxLength) return text;
  return text.slice(0, maxLength - suffix.length) + suffix;
}

module.exports = { formatDate, formatCurrency, slugify, truncate };

// src/utils/format.test.js
const { formatDate, formatCurrency, slugify, truncate } = require('./format');

describe('formatDate', () => {
  test('ควรแปลง Date เป็น YYYY-MM-DD', () => {
    expect(formatDate(new Date('2024-01-15'))).toBe('2024-01-15');
    expect(formatDate(new Date('2024-12-31'))).toBe('2024-12-31');
    expect(formatDate(new Date('2024-06-01'))).toBe('2024-06-01');
  });
  
  test('ควร throw TypeError เมื่อไม่ใช่ Date', () => {
    expect(() => formatDate('2024-01-15')).toThrow(TypeError);
    expect(() => formatDate(null)).toThrow(TypeError);
    expect(() => formatDate(1234567890)).toThrow(TypeError);
  });
  
  test('ควร pad เดือนและวันที่ถูกต้อง', () => {
    const date = new Date('2024-03-05');
    expect(formatDate(date)).toBe('2024-03-05'); // ไม่ใช่ 2024-3-5
  });
});

describe('slugify', () => {
  test('ควรแปลง text เป็น slug', () => {
    expect(slugify('Hello World')).toBe('hello-world');
    expect(slugify('  Hello   World  ')).toBe('hello-world');
    expect(slugify('Hello, World!')).toBe('hello-world');
  });
  
  test('ควรจัดการ special characters', () => {
    expect(slugify('Node.js Tutorial')).toBe('nodejs-tutorial');
    expect(slugify('100% Discount')).toBe('100-discount');
  });
  
  test('ควร handle empty string', () => {
    expect(slugify('')).toBe('');
  });
});

describe('truncate', () => {
  test('ควรตัดข้อความที่ยาวเกิน', () => {
    expect(truncate('Hello World', 8)).toBe('Hello...');
    expect(truncate('Hello World', 5)).toBe('He...');
  });
  
  test('ไม่ควรตัดข้อความที่สั้นกว่า maxLength', () => {
    expect(truncate('Hello', 10)).toBe('Hello');
    expect(truncate('Hello', 5)).toBe('Hello');
  });
  
  test('ควรใช้ custom suffix', () => {
    expect(truncate('Hello World', 8, ' →')).toBe('Hello W →');
  });
});
```

---

## Step 1262: Testing Classes

```javascript
// src/BankAccount.js
class BankAccount {
  constructor(owner, initialBalance = 0) {
    if (!owner) throw new Error('ต้องระบุเจ้าของบัญชี');
    if (initialBalance < 0) throw new Error('ยอดเงินเริ่มต้นต้องไม่ติดลบ');
    
    this.owner = owner;
    this.balance = initialBalance;
    this.transactions = [];
  }
  
  deposit(amount) {
    if (amount <= 0) throw new Error('จำนวนเงินต้องมากกว่า 0');
    
    this.balance += amount;
    this.transactions.push({
      type: 'deposit',
      amount,
      date: new Date(),
      balance: this.balance
    });
    
    return this.balance;
  }
  
  withdraw(amount) {
    if (amount <= 0) throw new Error('จำนวนเงินต้องมากกว่า 0');
    if (amount > this.balance) throw new Error('ยอดเงินไม่เพียงพอ');
    
    this.balance -= amount;
    this.transactions.push({
      type: 'withdrawal',
      amount,
      date: new Date(),
      balance: this.balance
    });
    
    return this.balance;
  }
  
  getStatement() {
    return {
      owner: this.owner,
      balance: this.balance,
      transactions: this.transactions
    };
  }
}

module.exports = BankAccount;

// src/BankAccount.test.js
const BankAccount = require('./BankAccount');

describe('BankAccount', () => {
  let account;
  
  beforeEach(() => {
    account = new BankAccount('Alice', 1000);
  });
  
  describe('constructor', () => {
    test('ควรสร้าง account พร้อม initial balance', () => {
      expect(account.owner).toBe('Alice');
      expect(account.balance).toBe(1000);
      expect(account.transactions).toHaveLength(0);
    });
    
    test('ควร throw เมื่อไม่มีชื่อเจ้าของ', () => {
      expect(() => new BankAccount()).toThrow('ต้องระบุเจ้าของบัญชี');
    });
    
    test('ควร throw เมื่อ initial balance ติดลบ', () => {
      expect(() => new BankAccount('Bob', -100)).toThrow('ยอดเงินเริ่มต้นต้องไม่ติดลบ');
    });
    
    test('ควร default initial balance เป็น 0', () => {
      const acc = new BankAccount('Bob');
      expect(acc.balance).toBe(0);
    });
  });
  
  describe('deposit', () => {
    test('ควรเพิ่มยอดเงิน', () => {
      account.deposit(500);
      expect(account.balance).toBe(1500);
    });
    
    test('ควรคืนค่า balance ใหม่', () => {
      const newBalance = account.deposit(500);
      expect(newBalance).toBe(1500);
    });
    
    test('ควรบันทึก transaction', () => {
      account.deposit(500);
      
      expect(account.transactions).toHaveLength(1);
      expect(account.transactions[0]).toMatchObject({
        type: 'deposit',
        amount: 500,
        balance: 1500
      });
    });
    
    test('ควร throw เมื่อ amount <= 0', () => {
      expect(() => account.deposit(0)).toThrow('จำนวนเงินต้องมากกว่า 0');
      expect(() => account.deposit(-100)).toThrow('จำนวนเงินต้องมากกว่า 0');
    });
  });
  
  describe('withdraw', () => {
    test('ควรลดยอดเงิน', () => {
      account.withdraw(300);
      expect(account.balance).toBe(700);
    });
    
    test('ควร throw เมื่อ balance ไม่พอ', () => {
      expect(() => account.withdraw(2000)).toThrow('ยอดเงินไม่เพียงพอ');
    });
    
    test('ควรถอนได้เท่ากับยอดคงเหลือ', () => {
      expect(() => account.withdraw(1000)).not.toThrow();
      expect(account.balance).toBe(0);
    });
  });
  
  describe('getStatement', () => {
    test('ควรส่ง statement ที่ถูกต้อง', () => {
      account.deposit(500);
      account.withdraw(200);
      
      const statement = account.getStatement();
      
      expect(statement.owner).toBe('Alice');
      expect(statement.balance).toBe(1300);
      expect(statement.transactions).toHaveLength(2);
    });
  });
});
```

---

## Step 1263: Code Coverage

```javascript
// รัน: npx jest --coverage

// ผลลัพธ์ coverage report:
// ---------------------|---------|----------|---------|---------|
// File                 | % Stmts | % Branch | % Funcs | % Lines |
// ---------------------|---------|----------|---------|---------|
// All files            |   85.71 |    83.33 |     100 |   85.71 |
//  BankAccount.js      |   85.71 |    83.33 |     100 |   85.71 |
// ---------------------|---------|----------|---------|---------|

// Metrics:
// Statements - จำนวน statements ที่ถูกรัน
// Branches - จำนวน branches (if/else) ที่ถูก test
// Functions - จำนวน functions ที่ถูกเรียก
// Lines - จำนวน lines ที่ถูกรัน

// Coverage ไม่ได้หมายความว่า code ถูกต้อง!
// คือแค่บอกว่า code บรรทัดไหนถูกรัน

// ตัวอย่าง 100% coverage แต่ยังมี bug
function isAdult(age) {
  return age >= 18;
}

// Test นี้ให้ 100% coverage
test('isAdult', () => {
  expect(isAdult(20)).toBe(true);
  expect(isAdult(15)).toBe(false);
});

// แต่ไม่ได้ test:
// - age = 18 (boundary)
// - age = undefined
// - age = '20' (string)
// - negative age

// Better tests
test('isAdult boundary cases', () => {
  expect(isAdult(18)).toBe(true);    // boundary
  expect(isAdult(17)).toBe(false);   // boundary - 1
  expect(isAdult(0)).toBe(false);    // zero
  expect(isAdult(100)).toBe(true);   // large number
});
```

---

## Step 1264: Snapshot Testing

```javascript
// Snapshot Testing - บันทึก "snapshot" ของ output
// และ fail ถ้า output เปลี่ยนไป

const renderer = require('react-test-renderer'); // สำหรับ React
// หรือ serializer สำหรับ plain objects

// ตัวอย่าง snapshot ของ function output
function generateHTML(data) {
  return `
    <div class="user">
      <h2>${data.name}</h2>
      <p>${data.email}</p>
    </div>
  `;
}

test('generateHTML snapshot', () => {
  const html = generateHTML({ name: 'Alice', email: 'alice@example.com' });
  expect(html).toMatchSnapshot();
});

// ครั้งแรก: สร้าง snapshot file
// ครั้งต่อไป: เปรียบเทียบกับ snapshot ที่บันทึกไว้

// อัพเดต snapshot: npx jest --updateSnapshot หรือ npx jest -u

// Inline Snapshot
test('inline snapshot', () => {
  const data = { id: 1, name: 'Alice' };
  expect(data).toMatchInlineSnapshot(`
    Object {
      "id": 1,
      "name": "Alice",
    }
  `);
});

// ข้อควรระวัง:
// - อย่า snapshot ข้อมูลที่เปลี่ยนบ่อย (เช่น timestamps)
// - ตรวจสอบ snapshot ก่อน commit
// - อย่า update snapshot โดยไม่ตรวจสอบ

// ถ้ามี dynamic data ต้องจัดการก่อน
test('snapshot with dynamic data', () => {
  const user = {
    id: 1,
    name: 'Alice',
    createdAt: new Date() // dynamic!
  };
  
  // แทนที่ dynamic value
  expect({
    ...user,
    createdAt: expect.any(Date)
  }).toMatchSnapshot();
});
```

---

## Step 1265: TDD (Test-Driven Development)

```javascript
// TDD Workflow:
// 1. Red: เขียน test ที่ fail ก่อน
// 2. Green: เขียน code ให้ test ผ่าน
// 3. Refactor: ปรับปรุง code โดยไม่ให้ test fail

// ตัวอย่าง: สร้าง Stack data structure

// STEP 1: Red - เขียน tests ก่อน
describe('Stack', () => {
  test('ควรสร้าง empty stack', () => {
    const stack = new Stack();
    expect(stack.isEmpty()).toBe(true);
    expect(stack.size()).toBe(0);
  });
  
  test('ควร push items', () => {
    const stack = new Stack();
    stack.push(1);
    stack.push(2);
    expect(stack.size()).toBe(2);
    expect(stack.isEmpty()).toBe(false);
  });
  
  test('ควร pop items (LIFO)', () => {
    const stack = new Stack();
    stack.push(1);
    stack.push(2);
    stack.push(3);
    
    expect(stack.pop()).toBe(3);
    expect(stack.pop()).toBe(2);
    expect(stack.pop()).toBe(1);
    expect(stack.pop()).toBeUndefined();
  });
  
  test('ควร peek ไม่ต้อง remove', () => {
    const stack = new Stack();
    stack.push(1);
    stack.push(2);
    
    expect(stack.peek()).toBe(2);
    expect(stack.size()).toBe(2); // ยังมี 2 items
  });
  
  test('ควร throw เมื่อ pop จาก empty stack', () => {
    const stack = new Stack();
    expect(() => stack.pop()).not.toThrow(); // return undefined, ไม่ throw
  });
});

// STEP 2: Green - เขียน code ให้ผ่าน
class Stack {
  constructor() {
    this._items = [];
  }
  
  push(item) {
    this._items.push(item);
  }
  
  pop() {
    return this._items.pop();
  }
  
  peek() {
    return this._items[this._items.length - 1];
  }
  
  size() {
    return this._items.length;
  }
  
  isEmpty() {
    return this._items.length === 0;
  }
}

// STEP 3: Refactor - ปรับปรุง code
// ในที่นี้ code ค่อนข้าง clean แล้ว

// เพิ่ม features ใหม่ด้วย TDD
describe('Stack - เพิ่มเติม', () => {
  test('ควรมี clear method', () => {
    const stack = new Stack();
    stack.push(1);
    stack.push(2);
    stack.clear();
    expect(stack.isEmpty()).toBe(true);
  });
  
  test('ควรมี contains method', () => {
    const stack = new Stack();
    stack.push(1);
    stack.push(2);
    
    expect(stack.contains(1)).toBe(true);
    expect(stack.contains(3)).toBe(false);
  });
});

// เพิ่ม methods:
Stack.prototype.clear = function() {
  this._items = [];
};

Stack.prototype.contains = function(item) {
  return this._items.includes(item);
};
```

---

## Step 1266: Testing Express Routes

```javascript
// ทดสอบ Express routes ด้วย supertest
// ติดตั้ง: npm install --save-dev supertest

// app.js
const express = require('express');
const app = express();

app.use(express.json());

const users = [
  { id: 1, name: 'Alice', email: 'alice@example.com' },
  { id: 2, name: 'Bob', email: 'bob@example.com' }
];

app.get('/users', (req, res) => {
  res.json(users);
});

app.get('/users/:id', (req, res) => {
  const user = users.find(u => u.id === parseInt(req.params.id));
  if (!user) return res.status(404).json({ error: 'ไม่พบ user' });
  res.json(user);
});

app.post('/users', (req, res) => {
  const { name, email } = req.body;
  if (!name || !email) {
    return res.status(400).json({ error: 'ต้องระบุ name และ email' });
  }
  
  const newUser = { id: users.length + 1, name, email };
  users.push(newUser);
  res.status(201).json(newUser);
});

module.exports = app;

// app.test.js
const request = require('supertest');
const app = require('./app');

describe('Users API', () => {
  describe('GET /users', () => {
    test('ควรส่ง list of users', async () => {
      const response = await request(app)
        .get('/users')
        .expect(200)
        .expect('Content-Type', /json/);
      
      expect(response.body).toBeInstanceOf(Array);
      expect(response.body.length).toBeGreaterThan(0);
    });
  });
  
  describe('GET /users/:id', () => {
    test('ควรส่ง user เมื่อพบ', async () => {
      const response = await request(app)
        .get('/users/1')
        .expect(200);
      
      expect(response.body).toMatchObject({
        id: 1,
        name: 'Alice'
      });
    });
    
    test('ควรส่ง 404 เมื่อไม่พบ', async () => {
      const response = await request(app)
        .get('/users/999')
        .expect(404);
      
      expect(response.body.error).toBeDefined();
    });
  });
  
  describe('POST /users', () => {
    test('ควรสร้าง user ใหม่', async () => {
      const response = await request(app)
        .post('/users')
        .send({ name: 'Charlie', email: 'charlie@example.com' })
        .expect(201);
      
      expect(response.body).toMatchObject({
        name: 'Charlie',
        email: 'charlie@example.com'
      });
      expect(response.body.id).toBeDefined();
    });
    
    test('ควรส่ง 400 เมื่อข้อมูลไม่ครบ', async () => {
      await request(app)
        .post('/users')
        .send({ name: 'No Email' })
        .expect(400);
    });
  });
});
```

---

## Step 1267: Mocking External Services

```javascript
// Mock API calls

// src/weatherService.js
const axios = require('axios');

async function getCurrentWeather(city) {
  const response = await axios.get(`https://api.weather.com/v1/current`, {
    params: {
      q: city,
      apikey: process.env.WEATHER_API_KEY
    }
  });
  
  return {
    city: response.data.location.name,
    temperature: response.data.current.temp_c,
    description: response.data.current.condition.text
  };
}

module.exports = { getCurrentWeather };

// src/weatherService.test.js
const axios = require('axios');
const { getCurrentWeather } = require('./weatherService');

jest.mock('axios');

describe('WeatherService', () => {
  afterEach(() => {
    jest.clearAllMocks();
  });
  
  test('ควรดึงข้อมูลอากาศสำเร็จ', async () => {
    // Setup mock
    axios.get.mockResolvedValue({
      data: {
        location: { name: 'Bangkok' },
        current: {
          temp_c: 32,
          condition: { text: 'Sunny' }
        }
      }
    });
    
    const weather = await getCurrentWeather('Bangkok');
    
    expect(weather).toEqual({
      city: 'Bangkok',
      temperature: 32,
      description: 'Sunny'
    });
    
    expect(axios.get).toHaveBeenCalledWith(
      'https://api.weather.com/v1/current',
      expect.objectContaining({
        params: { q: 'Bangkok', apikey: undefined }
      })
    );
  });
  
  test('ควรจัดการ API error', async () => {
    axios.get.mockRejectedValue(new Error('Network error'));
    
    await expect(getCurrentWeather('Bangkok'))
      .rejects.toThrow('Network error');
  });
});
```

---

## Step 1268: Custom Matchers

```javascript
// สร้าง custom matchers

// จาก jest.setup.js หรือใน test file

expect.extend({
  toBeValidEmail(received) {
    const emailRegex = /^\w+([.-]?\w+)*@\w+([.-]?\w+)*(\.\w{2,3})+$/;
    const pass = emailRegex.test(received);
    
    if (pass) {
      return {
        message: () => `expected ${received} not to be a valid email`,
        pass: true
      };
    } else {
      return {
        message: () => `expected ${received} to be a valid email`,
        pass: false
      };
    }
  },
  
  toBeWithinRange(received, floor, ceiling) {
    const pass = received >= floor && received <= ceiling;
    
    return {
      message: () => pass
        ? `expected ${received} not to be within range ${floor} - ${ceiling}`
        : `expected ${received} to be within range ${floor} - ${ceiling}`,
      pass
    };
  },
  
  toHaveValidFields(received, requiredFields) {
    const missingFields = requiredFields.filter(f => !(f in received));
    
    return {
      message: () => missingFields.length > 0
        ? `missing fields: ${missingFields.join(', ')}`
        : `all required fields present`,
      pass: missingFields.length === 0
    };
  }
});

// การใช้งาน
test('custom matchers', () => {
  expect('alice@example.com').toBeValidEmail();
  expect('not-an-email').not.toBeValidEmail();
  
  expect(5).toBeWithinRange(1, 10);
  expect(15).not.toBeWithinRange(1, 10);
  
  const user = { id: 1, name: 'Alice', email: 'alice@example.com' };
  expect(user).toHaveValidFields(['id', 'name', 'email']);
});
```

---

## Step 1269: Testing Utilities

```javascript
// Helper functions สำหรับ tests

// test-utils/createMockUser.js
function createMockUser(overrides = {}) {
  return {
    id: Math.floor(Math.random() * 1000),
    name: 'Test User',
    email: `test-${Date.now()}@example.com`,
    role: 'user',
    isActive: true,
    createdAt: new Date(),
    ...overrides
  };
}

function createMockPost(overrides = {}) {
  return {
    id: Math.floor(Math.random() * 1000),
    title: 'Test Post',
    content: 'Test content',
    isPublished: false,
    viewCount: 0,
    authorId: 1,
    createdAt: new Date(),
    ...overrides
  };
}

module.exports = { createMockUser, createMockPost };

// การใช้งาน
const { createMockUser } = require('./test-utils/createMockUser');

test('ควรสร้าง user ด้วย defaults', () => {
  const user = createMockUser();
  
  expect(user.name).toBe('Test User');
  expect(user.role).toBe('user');
  expect(user.isActive).toBe(true);
});

test('ควรสร้าง admin user', () => {
  const admin = createMockUser({ role: 'admin', name: 'Admin User' });
  
  expect(admin.role).toBe('admin');
  expect(admin.name).toBe('Admin User');
  expect(admin.isActive).toBe(true); // ใช้ default
});

// Test với timeout
test('async operation with timeout', async () => {
  jest.setTimeout(10000); // เพิ่ม timeout เป็น 10 วินาที
  
  const result = await longRunningOperation();
  expect(result).toBeDefined();
}, 10000); // หรือใส่เป็น argument ที่ 3
```

---

## Step 1270: Best Practices

```javascript
// Best practices สำหรับ Unit Tests

// 1. One assertion per test (หรือน้อยที่สุด)
// ❌ 
test('user', () => {
  const user = createUser('Alice', 'alice@example.com');
  expect(user.name).toBe('Alice');
  expect(user.email).toBe('alice@example.com');
  expect(user.isActive).toBe(true);
  expect(user.role).toBe('user');
  // ถ้า fail ไม่รู้ว่า assertion ไหนผิด
});

// ✓ Better
describe('createUser', () => {
  let user;
  
  beforeEach(() => {
    user = createUser('Alice', 'alice@example.com');
  });
  
  test('ควรตั้ง name', () => {
    expect(user.name).toBe('Alice');
  });
  
  test('ควรตั้ง email', () => {
    expect(user.email).toBe('alice@example.com');
  });
  
  test('ควร active โดย default', () => {
    expect(user.isActive).toBe(true);
  });
});

// 2. ตั้งชื่อ test ให้อธิบายชัดเจน
// ❌
test('user create', () => { ... });

// ✓
test('createUser ควรสร้าง user พร้อม default values', () => { ... });
test('createUser ควร throw เมื่อ email ไม่ถูกต้อง', () => { ... });

// 3. AAA Pattern (Arrange-Act-Assert)
test('deposit', () => {
  // Arrange
  const account = new BankAccount('Alice', 1000);
  
  // Act
  const newBalance = account.deposit(500);
  
  // Assert
  expect(newBalance).toBe(1500);
});

// 4. Test Isolation
// แต่ละ test ต้อง independent กัน
// ไม่ควร depend on state จาก test อื่น

// 5. Clear mocks between tests
afterEach(() => {
  jest.clearAllMocks();
});

// 6. Test edge cases
test('divide โดย 0', () => {
  expect(() => divide(10, 0)).toThrow();
});

test('empty array', () => {
  expect(sum([])).toBe(0);
});

test('negative numbers', () => {
  expect(add(-1, -2)).toBe(-3);
});

// 7. ไม่ test implementation details
// ❌ (test internal state)
test('users array', () => {
  service.addUser('Alice');
  expect(service._users).toHaveLength(1); // internal!
});

// ✓ (test behavior)
test('ควรเพิ่ม user', () => {
  service.addUser('Alice');
  expect(service.getUserCount()).toBe(1);
});
```

---

## แบบฝึกหัด

### ระดับเริ่มต้น
1. เขียน tests สำหรับ `calculateArea(shape, dimensions)` function
2. ทดสอบ `validatePassword(password)` ด้วย edge cases ต่างๆ
3. เขียน tests สำหรับ `Queue` data structure

### ระดับกลาง
4. สร้าง Mock สำหรับ database ใน User service tests
5. ทดสอบ async function ที่ call API
6. เขียน tests ครอบคลุม 90%+ ของ TodoList class

### ระดับสูง
7. implement TDD สำหรับ Calculator ที่รองรับ +, -, *, /, %
8. เขียน custom matchers สำหรับ Thai phone number validation
9. สร้าง test suite ครบสำหรับ BankAccount พร้อม edge cases
10. Mock module dependencies ทั้งหมดของ UserService

---

*Part 64 จบแล้ว ต่อไปเป็น Part 65: Testing - Integration Tests*
