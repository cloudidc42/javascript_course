# Part 15: Error Handling พื้นฐาน (Steps 271-290)

## บทนำ

Error Handling คือการจัดการกับข้อผิดพลาดที่อาจเกิดขึ้นในโปรแกรม การรู้จักจัดการ errors อย่างถูกต้องทำให้โปรแกรมทำงานได้อย่างราบรื่น แม้เกิด error ก็ไม่ crash และยังให้ feedback ที่มีประโยชน์แก่ผู้ใช้

---

## Step 271: ประเภทของ JavaScript Errors

JavaScript มี built-in Error types หลายประเภท:

```javascript
// 1. SyntaxError - ข้อผิดพลาดด้านไวยากรณ์ (เกิดตอน parse, ก่อน run)
// try { eval('let x = ;'); } catch(e) { console.log(e instanceof SyntaxError); } // true

// ตัวอย่าง SyntaxError:
// let x = ; // SyntaxError: Unexpected token ;
// function foo( { } // SyntaxError: missing )

// 2. TypeError - ใช้ค่าที่ไม่ถูก type
try {
  null.property; // Cannot read properties of null
} catch (e) {
  console.log(e instanceof TypeError); // true
  console.log(e.name);    // "TypeError"
  console.log(e.message); // "Cannot read properties of null..."
}

try {
  undefined();  // undefined is not a function
} catch (e) {
  console.log(e instanceof TypeError); // true
}

try {
  const obj = {};
  obj.method(); // TypeError: obj.method is not a function
} catch (e) {
  console.log(e.name); // "TypeError"
}

// 3. ReferenceError - ใช้ตัวแปรที่ไม่ได้ declare
try {
  console.log(undeclaredVariable);
} catch (e) {
  console.log(e instanceof ReferenceError); // true
  console.log(e.message); // "undeclaredVariable is not defined"
}

try {
  foo(); // ReferenceError: foo is not defined (ก่อน hoisting)
  const foo = () => {};
} catch (e) {
  console.log(e.name); // "ReferenceError"
}

// 4. RangeError - ค่าออกนอก valid range
try {
  new Array(-1); // Invalid array length
} catch (e) {
  console.log(e instanceof RangeError); // true
}

try {
  const n = 1;
  n.toFixed(200); // toFixed() digits argument must be between 0 and 100
} catch (e) {
  console.log(e.name); // "RangeError"
}

try {
  function infinite() { infinite(); }
  infinite(); // Maximum call stack size exceeded
} catch (e) {
  console.log(e instanceof RangeError); // true ใน V8
}

// 5. URIError - ใช้กับ URI functions ไม่ถูกต้อง
try {
  decodeURIComponent('%');
} catch (e) {
  console.log(e instanceof URIError); // true
}

// 6. EvalError - (rare) errors ใน eval()
// ปัจจุบัน browsers ส่วนใหญ่ throw TypeError แทน

// 7. AggregateError - รวม errors หลายอัน (ES2021)
try {
  throw new AggregateError([
    new Error('Error 1'),
    new TypeError('Error 2'),
  ], 'Multiple errors occurred');
} catch (e) {
  console.log(e.message);  // "Multiple errors occurred"
  console.log(e.errors);   // [Error, TypeError]
}

// ตัวอย่างจาก Promise.any()
async function exampleAggregateError() {
  try {
    await Promise.any([
      Promise.reject(new Error('fail 1')),
      Promise.reject(new Error('fail 2')),
    ]);
  } catch (e) {
    console.log(e instanceof AggregateError); // true
    console.log(e.errors.map(err => err.message)); // ["fail 1", "fail 2"]
  }
}
```

---

## Step 272: try...catch Statement

```javascript
// try...catch คือ syntax สำหรับจัดการ exceptions

// รูปแบบพื้นฐาน
try {
  // โค้ดที่อาจเกิด error
  const result = JSON.parse('invalid json');
  console.log(result); // จะไม่ถึงบรรทัดนี้
} catch (error) {
  // จัดการ error ที่นี่
  console.log('Error caught:', error.message);
}

// ตัวอย่างจริง
function divide(a, b) {
  try {
    if (b === 0) throw new Error('Cannot divide by zero');
    return a / b;
  } catch (e) {
    console.error('Division error:', e.message);
    return null;
  }
}

console.log(divide(10, 2));  // 5
console.log(divide(10, 0));  // null (error handled)

// catch ดัก error ทุกประเภท
try {
  const data = null;
  const value = data.property; // TypeError
} catch (e) {
  if (e instanceof TypeError) {
    console.log('Type error:', e.message);
  } else {
    console.log('Unknown error:', e);
  }
}

// try...catch กับ async code
// catch ไม่ดัก async errors ที่ไม่ได้ await!
try {
  setTimeout(() => {
    // throw new Error('Async error!'); // จะไม่ถูกดัก!
  }, 100);
} catch (e) {
  console.log('This will NOT catch the setTimeout error');
}

// ต้อง handle ใน callback เอง
setTimeout(() => {
  try {
    throw new Error('Error in setTimeout');
  } catch (e) {
    console.log('Caught inside setTimeout:', e.message);
  }
}, 100);

// ตัวอย่าง: safe JSON parse
function safeParseJSON(jsonString, defaultValue = null) {
  try {
    return JSON.parse(jsonString);
  } catch (e) {
    console.warn('JSON parse failed:', e.message);
    return defaultValue;
  }
}

console.log(safeParseJSON('{"name":"John"}')); // { name: "John" }
console.log(safeParseJSON('invalid json', {})); // {}
console.log(safeParseJSON(null, []));           // []

// ตัวอย่าง: safe DOM access
function safeGetElement(id) {
  try {
    const el = document.getElementById(id);
    if (!el) throw new Error(`Element #${id} not found`);
    return el;
  } catch (e) {
    console.warn(e.message);
    return null;
  }
}

// catch ตัวแปร error เป็น optional (ES2019+)
try {
  JSON.parse('{invalid}');
} catch {
  // ไม่ต้องใช้ error variable
  console.log('JSON parsing failed');
}
```

---

## Step 273: finally Block

```javascript
// finally - ทำงานเสมอ ไม่ว่าจะ error หรือไม่

// รูปแบบ
try {
  // โค้ด
} catch (error) {
  // จัดการ error
} finally {
  // ทำงานเสมอ (cleanup)
}

// ตัวอย่าง: resource cleanup
function processFile() {
  let resource = null;
  try {
    resource = openResource(); // สมมติว่า open resource
    const data = readData(resource);
    return processData(data);
  } catch (e) {
    console.error('Error processing file:', e.message);
    throw e; // rethrow
  } finally {
    // cleanup เสมอ ไม่ว่าจะเกิด error หรือไม่
    if (resource) {
      closeResource(resource);
      console.log('Resource closed');
    }
  }
}

// finally กับ return
function example1() {
  try {
    return 'try';
  } finally {
    console.log('finally runs'); // ยังทำงาน
  }
}
console.log(example1()); // logs "finally runs", returns "try"

// finally return override
function example2() {
  try {
    return 'try value';
  } finally {
    return 'finally value'; // override return จาก try!
  }
}
console.log(example2()); // "finally value" (ไม่แนะนำ!)

// ตัวอย่าง: Database connection
async function withDatabase(operation) {
  let connection = null;
  try {
    connection = await connectToDatabase();
    return await operation(connection);
  } catch (error) {
    console.error('Database error:', error.message);
    throw error;
  } finally {
    if (connection) {
      await connection.close();
      console.log('Database connection closed');
    }
  }
}

// ตัวอย่าง: Loading state management
async function fetchDataWithLoading(url) {
  const loadingEl = document.getElementById('loading');
  const errorEl = document.getElementById('error');

  try {
    loadingEl?.classList.add('visible');
    errorEl?.classList.remove('visible');

    const response = await fetch(url);
    if (!response.ok) throw new Error(`HTTP ${response.status}`);
    return await response.json();
  } catch (error) {
    errorEl?.textContent = error.message;
    errorEl?.classList.add('visible');
    throw error;
  } finally {
    loadingEl?.classList.remove('visible'); // เสมอ
  }
}

// ตัวอย่าง: Timer
function measureTime(fn) {
  const start = performance.now();
  try {
    return fn();
  } finally {
    const duration = performance.now() - start;
    console.log(`Execution time: ${duration.toFixed(2)}ms`);
  }
}

measureTime(() => {
  let sum = 0;
  for (let i = 0; i < 1000000; i++) sum += i;
  return sum;
});
```

---

## Step 274: Rethrowing Errors

```javascript
// Rethrowing: catch error แต่ throw ต่อ
// ใช้เมื่อต้องการ handle บาง errors แต่ปล่อยอื่นๆ ขึ้นไป

function processUserInput(input) {
  try {
    if (typeof input !== 'string') {
      throw new TypeError('Input must be a string');
    }
    if (input.length === 0) {
      throw new RangeError('Input cannot be empty');
    }
    return input.toUpperCase();
  } catch (e) {
    // Handle เฉพาะ RangeError
    if (e instanceof RangeError) {
      console.warn('Empty input, using default');
      return 'DEFAULT';
    }
    // Rethrow ที่ไม่ใช่ RangeError
    throw e;
  }
}

try {
  processUserInput('');     // RangeError -> handled, returns "DEFAULT"
  processUserInput(123);    // TypeError -> rethrown
} catch (e) {
  console.error('Unhandled error:', e.message); // TypeError here
}

// ตัวอย่างเพิ่มเติม: Network error handling
async function fetchUser(id) {
  try {
    const response = await fetch(`/api/users/${id}`);
    const data = await response.json();
    return data;
  } catch (e) {
    // ดักเฉพาะ network errors
    if (e instanceof TypeError && e.message.includes('fetch')) {
      throw new NetworkError('ไม่สามารถเชื่อมต่อ server ได้');
    }
    // Rethrow อื่นๆ
    throw e;
  }
}

// Rethrow กับ async/await
async function safeOperation() {
  try {
    await riskyOperation();
  } catch (e) {
    // Log และ rethrow
    console.error('[safeOperation] Error:', e);
    // เพิ่ม context
    const enhancedError = new Error(`Operation failed: ${e.message}`);
    enhancedError.cause = e; // ES2022 error.cause
    throw enhancedError;
  }
}

// error.cause (ES2022)
try {
  try {
    JSON.parse('{bad}');
  } catch (parseError) {
    throw new Error('Failed to process config', { cause: parseError });
  }
} catch (e) {
  console.log(e.message); // "Failed to process config"
  console.log(e.cause);   // SyntaxError: Unexpected token b...
}

// Rethrow pattern ที่ดี
function handleApiError(error) {
  if (error.response) {
    // Server responded with error status
    switch (error.response.status) {
      case 400: throw new ValidationError(error.response.data.message);
      case 401: throw new AuthError('กรุณาเข้าสู่ระบบ');
      case 403: throw new PermissionError('ไม่มีสิทธิ์เข้าถึง');
      case 404: throw new NotFoundError('ไม่พบข้อมูล');
      case 500: throw new ServerError('เกิดข้อผิดพลาดที่ server');
      default: throw error;
    }
  } else if (error.request) {
    throw new NetworkError('ไม่สามารถเชื่อมต่อได้');
  } else {
    throw error;
  }
}
```

---

## Step 275: Error Object Properties

```javascript
// Error object มี properties หลัก:
// - name: ชื่อ error type
// - message: คำอธิบาย error
// - stack: stack trace
// - cause: (ES2022) original error ที่ทำให้เกิด error นี้

const error = new Error('Something went wrong');
console.log(error.name);    // "Error"
console.log(error.message); // "Something went wrong"
console.log(error.stack);
// Error: Something went wrong
//     at <anonymous>:1:15
//     at ...

// Error types มี name ต่างกัน
console.log(new TypeError('type issue').name);     // "TypeError"
console.log(new RangeError('range issue').name);   // "RangeError"
console.log(new SyntaxError('syntax').name);       // "SyntaxError"
console.log(new ReferenceError('ref').name);       // "ReferenceError"

// Stack trace - แสดง call stack ที่นำไปสู่ error
function level3() { throw new Error('Deep error'); }
function level2() { level3(); }
function level1() { level2(); }

try {
  level1();
} catch (e) {
  console.log(e.stack);
  // Error: Deep error
  //     at level3 (script.js:1)
  //     at level2 (script.js:2)
  //     at level1 (script.js:3)
  //     at <anonymous>:1:1
}

// Parse stack trace
function parseStack(error) {
  if (!error.stack) return [];
  const lines = error.stack.split('\n');
  return lines.slice(1).map(line => {
    const match = line.match(/at (.+?) \((.+?):(\d+):(\d+)\)/);
    if (match) {
      return { fn: match[1], file: match[2], line: parseInt(match[3]), col: parseInt(match[4]) };
    }
    return { raw: line.trim() };
  });
}

try {
  null.prop;
} catch (e) {
  const stack = parseStack(e);
  console.log('First frame:', stack[0]);
}

// error.cause (ES2022)
try {
  try {
    JSON.parse('{invalid}');
  } catch (original) {
    throw new Error('Config parse failed', { cause: original });
  }
} catch (e) {
  console.log(e.message);         // "Config parse failed"
  console.log(e.cause?.message);  // "Unexpected token i..."
  
  // เดินตาม cause chain
  let current = e;
  while (current) {
    console.log(`- ${current.message}`);
    current = current.cause;
  }
}

// Error.captureStackTrace (V8 specific)
// ปรับ stack trace ใน custom errors
class CustomError extends Error {
  constructor(message) {
    super(message);
    this.name = 'CustomError';
    if (Error.captureStackTrace) {
      Error.captureStackTrace(this, CustomError);
    }
  }
}
```

---

## Step 276: Custom Error Classes

```javascript
// สร้าง custom error โดย extend Error class

class AppError extends Error {
  constructor(message, code, statusCode = 500) {
    super(message);           // call parent constructor
    this.name = 'AppError';   // กำหนด name
    this.code = code;         // custom property
    this.statusCode = statusCode;
    this.timestamp = new Date().toISOString();

    // Ensure instanceof ทำงานถูกต้อง (สำหรับ transpiled code)
    Object.setPrototypeOf(this, new.target.prototype);

    // Capture stack trace (V8)
    if (Error.captureStackTrace) {
      Error.captureStackTrace(this, this.constructor);
    }
  }

  toJSON() {
    return {
      name: this.name,
      message: this.message,
      code: this.code,
      statusCode: this.statusCode,
      timestamp: this.timestamp,
    };
  }
}

// สร้าง specific error types
class ValidationError extends AppError {
  constructor(message, field, value) {
    super(message, 'VALIDATION_ERROR', 400);
    this.name = 'ValidationError';
    this.field = field;
    this.value = value;
  }
}

class AuthError extends AppError {
  constructor(message = 'Unauthorized') {
    super(message, 'AUTH_ERROR', 401);
    this.name = 'AuthError';
  }
}

class NotFoundError extends AppError {
  constructor(resource, id) {
    super(`${resource} with id ${id} not found`, 'NOT_FOUND', 404);
    this.name = 'NotFoundError';
    this.resource = resource;
    this.resourceId = id;
  }
}

class NetworkError extends AppError {
  constructor(message = 'Network request failed') {
    super(message, 'NETWORK_ERROR', 503);
    this.name = 'NetworkError';
  }
}

class DatabaseError extends AppError {
  constructor(message, query) {
    super(message, 'DB_ERROR', 500);
    this.name = 'DatabaseError';
    this.query = query;
  }
}

// ใช้งาน
function validateAge(age) {
  if (typeof age !== 'number') {
    throw new ValidationError('Age must be a number', 'age', age);
  }
  if (age < 0 || age > 150) {
    throw new ValidationError('Age must be between 0 and 150', 'age', age);
  }
  return true;
}

function findUser(id) {
  const users = [{ id: 1, name: 'Alice' }];
  const user = users.find(u => u.id === id);
  if (!user) throw new NotFoundError('User', id);
  return user;
}

// Test custom errors
try {
  validateAge('twenty');
} catch (e) {
  console.log(e instanceof ValidationError); // true
  console.log(e instanceof AppError);        // true
  console.log(e instanceof Error);           // true
  console.log(e.name);        // "ValidationError"
  console.log(e.message);     // "Age must be a number"
  console.log(e.field);       // "age"
  console.log(e.statusCode);  // 400
  console.log(JSON.stringify(e.toJSON(), null, 2));
}

try {
  findUser(999);
} catch (e) {
  console.log(e instanceof NotFoundError); // true
  console.log(e.message);    // "User with id 999 not found"
  console.log(e.resource);   // "User"
  console.log(e.resourceId); // 999
}

// Error factory pattern
function createError(type, ...args) {
  const errors = {
    validation: (msg, field) => new ValidationError(msg, field),
    auth: (msg) => new AuthError(msg),
    notFound: (resource, id) => new NotFoundError(resource, id),
    network: (msg) => new NetworkError(msg),
  };
  const factory = errors[type];
  if (!factory) throw new Error(`Unknown error type: ${type}`);
  return factory(...args);
}
```

---

## Step 277: Extending Error สำหรับ Application

```javascript
// Error hierarchy สำหรับ web application

class BaseError extends Error {
  constructor(message, options = {}) {
    super(message, { cause: options.cause });
    this.name = this.constructor.name;
    this.code = options.code || 'UNKNOWN_ERROR';
    this.statusCode = options.statusCode || 500;
    this.data = options.data || null;
    this.timestamp = Date.now();
    Object.setPrototypeOf(this, new.target.prototype);
  }

  isOperational() {
    return this.statusCode < 500; // operational errors (4xx)
  }

  toResponse() {
    return {
      error: {
        code: this.code,
        message: this.message,
        ...(this.data && { data: this.data }),
      },
    };
  }
}

// HTTP Errors
class HttpError extends BaseError {
  constructor(statusCode, message, code) {
    super(message, { statusCode, code });
    this.name = 'HttpError';
  }
}

class BadRequestError extends HttpError {
  constructor(message = 'Bad Request', data) {
    super(400, message, 'BAD_REQUEST');
    this.data = data;
  }
}

class UnauthorizedError extends HttpError {
  constructor(message = 'Unauthorized') {
    super(401, message, 'UNAUTHORIZED');
  }
}

class ForbiddenError extends HttpError {
  constructor(message = 'Forbidden') {
    super(403, message, 'FORBIDDEN');
  }
}

class NotFoundError extends HttpError {
  constructor(resource = 'Resource') {
    super(404, `${resource} not found`, 'NOT_FOUND');
    this.resource = resource;
  }
}

class ConflictError extends HttpError {
  constructor(message = 'Conflict') {
    super(409, message, 'CONFLICT');
  }
}

class InternalServerError extends HttpError {
  constructor(message = 'Internal Server Error') {
    super(500, message, 'INTERNAL_ERROR');
  }
}

// Business Logic Errors
class ValidationErrors extends BaseError {
  constructor(errors) {
    super('Validation failed', { statusCode: 422, code: 'VALIDATION_FAILED' });
    this.errors = errors; // { field: [messages] }
  }

  addError(field, message) {
    if (!this.errors[field]) this.errors[field] = [];
    this.errors[field].push(message);
    return this;
  }

  hasErrors() {
    return Object.keys(this.errors).length > 0;
  }

  toResponse() {
    return {
      error: {
        code: this.code,
        message: this.message,
        errors: this.errors,
      },
    };
  }
}

// ใช้งาน
function validateRegistration(data) {
  const errors = new ValidationErrors({});

  if (!data.email) errors.addError('email', 'จำเป็นต้องกรอก');
  else if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(data.email))
    errors.addError('email', 'รูปแบบไม่ถูกต้อง');

  if (!data.password) errors.addError('password', 'จำเป็นต้องกรอก');
  else if (data.password.length < 8)
    errors.addError('password', 'ต้องมีอย่างน้อย 8 ตัวอักษร');

  if (errors.hasErrors()) throw errors;
  return true;
}

try {
  validateRegistration({ email: 'bad-email', password: '123' });
} catch (e) {
  if (e instanceof ValidationErrors) {
    console.log(e.errors);
    // { email: ["รูปแบบไม่ถูกต้อง"], password: ["ต้องมีอย่างน้อย 8 ตัวอักษร"] }
  }
}

// Global error handler
function handleError(error) {
  if (error instanceof ValidationErrors) {
    return { status: 422, body: error.toResponse() };
  } else if (error instanceof HttpError) {
    if (!error.isOperational()) {
      // Log server errors
      console.error('[FATAL]', error);
    }
    return { status: error.statusCode, body: error.toResponse() };
  } else {
    // Unexpected error
    console.error('[UNEXPECTED]', error);
    return { status: 500, body: { error: { code: 'INTERNAL_ERROR', message: 'เกิดข้อผิดพลาด' } } };
  }
}
```

---

## Step 278: window.onerror และ Global Error Handling

```javascript
// window.onerror - catch uncaught errors

window.onerror = function(message, source, lineno, colno, error) {
  console.log('Global error caught:');
  console.log('Message:', message);
  console.log('Source:', source);
  console.log('Line:', lineno, 'Col:', colno);
  console.log('Error object:', error);

  // Send to error tracking service
  sendErrorToServer({
    message,
    source,
    lineno,
    colno,
    stack: error?.stack,
    timestamp: new Date().toISOString(),
    url: window.location.href,
    userAgent: navigator.userAgent,
  });

  return true; // prevent default error handling (console error)
};

// window.addEventListener('error') - similar but more info
window.addEventListener('error', (event) => {
  console.log('Error event:', {
    message: event.message,
    filename: event.filename,
    lineno: event.lineno,
    colno: event.colno,
    error: event.error,
  });

  // ดัก resource loading errors (img, script, css)
  if (event.target !== window && event.target.tagName) {
    console.log(`Resource failed to load: ${event.target.src || event.target.href}`);
  }
}, true); // capture phase สำหรับ resource errors

// ดัก unhandled promise rejections
window.addEventListener('unhandledrejection', (event) => {
  console.error('Unhandled Promise rejection:', event.reason);
  
  // Prevent default (suppress console warning)
  event.preventDefault();
  
  // Send to tracking
  sendErrorToServer({
    type: 'unhandledRejection',
    message: event.reason?.message || String(event.reason),
    stack: event.reason?.stack,
  });
});

// ดัก rejections ที่ handle ทีหลัง (เกิน 1 tick)
window.addEventListener('rejectionhandled', (event) => {
  console.log('Promise rejection was handled late:', event.promise);
});

// สร้าง error tracking middleware
function sendErrorToServer(errorData) {
  // ใน production ส่งไป error tracking service
  if (process.env.NODE_ENV === 'production') {
    fetch('/api/errors', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(errorData),
      keepalive: true, // ส่งแม้ page unload
    }).catch(console.error);
  }
}

// ตัวอย่าง: Error Boundary สำหรับ vanilla JS
function safeRender(fn, container) {
  try {
    const result = fn();
    if (typeof result === 'string') {
      container.innerHTML = result;
    }
  } catch (e) {
    container.innerHTML = `
      <div style="color:red;padding:20px;background:#fff0f0;border:1px solid red;border-radius:8px;">
        <strong>⚠️ เกิดข้อผิดพลาด</strong>
        <p>${e.message}</p>
        <button onclick="location.reload()">โหลดหน้าใหม่</button>
      </div>
    `;
    console.error('Render error:', e);
  }
}
```

---

## Step 279: Unhandled Promise Rejections

```javascript
// Promises ต้องมี .catch() เสมอ มิฉะนั้นจะเป็น unhandled rejection

// BAD: ไม่มี error handling
async function badExample() {
  const data = await fetch('/api/nonexistent'); // อาจ fail!
  return data.json();
}

// GOOD: มี error handling
async function goodExample() {
  try {
    const response = await fetch('/api/data');
    if (!response.ok) throw new Error(`HTTP ${response.status}`);
    return await response.json();
  } catch (e) {
    console.error('Fetch failed:', e.message);
    return null;
  }
}

// Promise chain - ต้องมี .catch() ท้าย chain
fetch('/api/data')
  .then(res => res.json())
  .then(data => processData(data))
  .catch(e => console.error('Chain error:', e));

// ถ้าลืม .catch()
const promise = Promise.reject(new Error('Forgotten!'));
// ถ้าไม่ handle = UnhandledPromiseRejection warning

// Handle ในทุกกรณี
promise.catch(e => console.error('Handled:', e.message));

// Async function error propagation
async function step1() { throw new Error('Step 1 failed'); }
async function step2() { await step1(); }
async function step3() {
  try {
    await step2();
  } catch (e) {
    console.error('Caught in step3:', e.message); // "Step 1 failed"
  }
}
step3();

// Promise.allSettled - ไม่ throw แม้บาง promise fail
async function fetchAll(urls) {
  const results = await Promise.allSettled(
    urls.map(url => fetch(url).then(r => r.json()))
  );

  results.forEach((result, i) => {
    if (result.status === 'fulfilled') {
      console.log(`URL ${i}: success`, result.value);
    } else {
      console.log(`URL ${i}: failed`, result.reason.message);
    }
  });

  return results
    .filter(r => r.status === 'fulfilled')
    .map(r => r.value);
}

// รูปแบบที่ดีสำหรับ async operations
async function safeAsync(promise) {
  try {
    const data = await promise;
    return [null, data]; // [error, data] pattern
  } catch (error) {
    return [error, null];
  }
}

async function example() {
  const [error, data] = await safeAsync(fetch('/api/data').then(r => r.json()));
  if (error) {
    console.error('Error:', error.message);
    return;
  }
  console.log('Data:', data);
}

// Timeout wrapper
function withTimeout(promise, ms, message = 'Operation timed out') {
  const timeout = new Promise((_, reject) =>
    setTimeout(() => reject(new Error(message)), ms)
  );
  return Promise.race([promise, timeout]);
}

async function fetchWithTimeout(url, ms = 5000) {
  try {
    return await withTimeout(
      fetch(url).then(r => r.json()),
      ms,
      `Request to ${url} timed out after ${ms}ms`
    );
  } catch (e) {
    console.error(e.message);
    throw e;
  }
}
```

---

## Step 280: Error Handling Best Practices

```javascript
// 1. Never swallow errors silently
// BAD:
try {
  riskyOperation();
} catch (e) {
  // ไม่ทำอะไรเลย! อย่าทำแบบนี้
}

// GOOD:
try {
  riskyOperation();
} catch (e) {
  console.error('riskyOperation failed:', e.message);
  // หรือ handle appropriately
  throw e; // หรือ rethrow ถ้าไม่รู้จะทำอะไร
}

// 2. Be specific about what you catch
// BAD:
try {
  JSON.parse(input);
  validateData(data);
  saveToDatabase(data);
} catch (e) {
  console.error('Something failed'); // ไม่รู้ว่า step ไหน fail
}

// GOOD:
let parsedData;
try {
  parsedData = JSON.parse(input);
} catch (e) {
  throw new ValidationError('Invalid JSON input', { cause: e });
}

try {
  validateData(parsedData);
} catch (e) {
  throw new ValidationError(e.message);
}

try {
  await saveToDatabase(parsedData);
} catch (e) {
  throw new DatabaseError('Failed to save', { cause: e });
}

// 3. Provide meaningful error messages
// BAD:
throw new Error('Error');
throw new Error('Failed');
throw new Error('Invalid');

// GOOD:
throw new ValidationError(`Email "${email}" is not a valid email address`);
throw new NotFoundError(`User with ID ${userId} does not exist`);
throw new AuthError(`Session expired at ${expiryTime}. Please log in again.`);

// 4. Log errors appropriately
const logger = {
  debug: (...args) => console.debug(...args),
  info: (...args) => console.info(...args),
  warn: (...args) => console.warn(...args),
  error: (...args) => {
    console.error(...args);
    // Send to error tracking in production
  },
};

// 5. Error recovery strategies
async function withRetry(fn, options = {}) {
  const { retries = 3, delay = 1000, backoff = 2 } = options;
  let lastError;

  for (let attempt = 0; attempt <= retries; attempt++) {
    try {
      return await fn();
    } catch (e) {
      lastError = e;
      if (attempt < retries) {
        const waitTime = delay * Math.pow(backoff, attempt);
        console.warn(`Attempt ${attempt + 1} failed, retrying in ${waitTime}ms...`);
        await new Promise(resolve => setTimeout(resolve, waitTime));
      }
    }
  }
  throw lastError;
}

// ใช้งาน
async function reliableFetch(url) {
  return withRetry(() => fetch(url).then(r => r.json()), {
    retries: 3,
    delay: 500,
  });
}

// 6. Fail fast
function createUser(data) {
  // Validate early, fail fast
  if (!data) throw new ValidationError('User data is required');
  if (!data.email) throw new ValidationError('Email is required');
  if (!data.password) throw new ValidationError('Password is required');

  // ถ้าผ่านมาถึงนี่ data valid แล้ว
  return saveUser(data);
}
```

---

## Step 281: Debugging กับ console methods

```javascript
// Console methods ต่างๆ สำหรับ debugging

// console.log - ทั่วไป
console.log('Hello', 'World');
console.log({ name: 'Alice', age: 30 });
console.log([1, 2, 3]);

// console.error - แสดงเป็น error (สีแดง)
console.error('Error occurred!', new Error('Details'));

// console.warn - แสดงเป็น warning (สีเหลือง)
console.warn('Deprecated function used');

// console.info - informational
console.info('Server started on port 3000');

// console.debug - debug level
console.debug('Variable value:', someVar);

// console.dir - แสดง object properties
console.dir(document.body);
console.dir(document.body, { depth: 2 });

// console.table - แสดง array of objects เป็น table
const users = [
  { id: 1, name: 'Alice', role: 'admin' },
  { id: 2, name: 'Bob', role: 'user' },
  { id: 3, name: 'Charlie', role: 'user' },
];
console.table(users);
console.table(users, ['name', 'role']); // เฉพาะ columns ที่ต้องการ

// console.group / console.groupEnd - จัดกลุ่ม logs
console.group('User Details');
console.log('Name: Alice');
console.log('Age: 30');
console.group('Address');
console.log('City: Bangkok');
console.log('Country: Thailand');
console.groupEnd();
console.groupEnd();

// console.groupCollapsed - collapsed by default
console.groupCollapsed('Collapsed Group');
console.log('Hidden by default');
console.groupEnd();

// console.time / console.timeEnd - measure time
console.time('loop');
let sum = 0;
for (let i = 0; i < 1000000; i++) sum += i;
console.timeEnd('loop'); // "loop: 5.123ms"

// console.timeLog - log intermediate time
console.time('task');
await fetchData();
console.timeLog('task', 'after fetch'); // "task: 245ms after fetch"
await processData();
console.timeEnd('task'); // "task: 312ms"

// console.count / console.countReset - count calls
function onClick() {
  console.count('Button clicked');
}
// Call 3 times:
// Button clicked: 1
// Button clicked: 2
// Button clicked: 3
console.countReset('Button clicked');

// console.assert - log only if condition is false
const x = 5;
console.assert(x > 0, 'x should be positive'); // ไม่แสดง
console.assert(x > 10, 'x should be greater than 10', { x }); // แสดง

// console.trace - show stack trace
function foo() {
  function bar() {
    console.trace('Trace from bar');
  }
  bar();
}
foo();
// Trace from bar
//     at bar
//     at foo
//     at <anonymous>

// console.clear - clear console
// console.clear();

// Styled console output
console.log('%cStyled text', 'color: blue; font-size: 20px; font-weight: bold;');
console.log(
  '%c Success %c Error',
  'background: green; color: white; padding: 2px 6px; border-radius: 3px;',
  'background: red; color: white; padding: 2px 6px; border-radius: 3px;'
);

// Conditional logging
const DEBUG = true;
const log = {
  debug: (...args) => DEBUG && console.debug('[DEBUG]', ...args),
  info: (...args) => console.info('[INFO]', ...args),
  warn: (...args) => console.warn('[WARN]', ...args),
  error: (...args) => console.error('[ERROR]', ...args),
};

log.debug('Variable:', someVar); // เฉพาะ DEBUG mode
log.info('Operation completed');
```

---

## Step 282: Defensive Programming

```javascript
// Defensive Programming: เขียนโค้ดที่รับมือกับ unexpected inputs

// 1. Guard Clauses - return early
function processOrder(order) {
  // ตรวจสอบก่อน
  if (!order) {
    console.warn('processOrder: order is null/undefined');
    return null;
  }
  if (!order.items || order.items.length === 0) {
    throw new ValidationError('Order must have at least one item');
  }
  if (!order.userId) {
    throw new ValidationError('Order must have a userId');
  }

  // Process order...
  return calculateTotal(order);
}

// 2. Null coalescing และ Optional chaining
function getUsername(user) {
  return user?.profile?.username ?? 'Anonymous';
}

function getAddress(user) {
  return {
    street: user?.address?.street ?? 'N/A',
    city: user?.address?.city ?? 'N/A',
    zip: user?.address?.zip ?? 'N/A',
  };
}

// 3. Default values
function createUser({
  name = 'Anonymous',
  role = 'user',
  settings = {},
  tags = [],
} = {}) {
  return { name, role, settings, tags, createdAt: new Date() };
}

createUser();          // ใช้ defaults ทั้งหมด
createUser({ name: 'Alice' }); // name ต่างออกไป, ที่เหลือ default

// 4. Type checking
function safeDivide(a, b) {
  if (typeof a !== 'number' || typeof b !== 'number') {
    throw new TypeError(`Expected numbers, got ${typeof a} and ${typeof b}`);
  }
  if (isNaN(a) || isNaN(b)) {
    throw new RangeError('Arguments must not be NaN');
  }
  if (!isFinite(a) || !isFinite(b)) {
    throw new RangeError('Arguments must be finite');
  }
  if (b === 0) {
    throw new RangeError('Cannot divide by zero');
  }
  return a / b;
}

// 5. Array safety
function safeFirstItem(arr, defaultValue = null) {
  if (!Array.isArray(arr) || arr.length === 0) return defaultValue;
  return arr[0];
}

function safeGet(obj, path, defaultValue = undefined) {
  if (!obj || typeof obj !== 'object') return defaultValue;
  const keys = path.split('.');
  let current = obj;
  for (const key of keys) {
    if (current === null || current === undefined) return defaultValue;
    current = current[key];
  }
  return current ?? defaultValue;
}

console.log(safeGet({ user: { name: 'Alice' } }, 'user.name'));    // "Alice"
console.log(safeGet({ user: {} }, 'user.address.city', 'N/A'));    // "N/A"
console.log(safeGet(null, 'anything', 'default'));                  // "default"

// 6. Input sanitization
function sanitizeHTML(str) {
  const div = document.createElement('div');
  div.textContent = str;
  return div.innerHTML;
}

function sanitizeInput(input, maxLength = 255) {
  if (typeof input !== 'string') return '';
  return input.trim().slice(0, maxLength);
}

// 7. Error boundaries in React-like pattern
class ErrorBoundary {
  constructor(fallback) {
    this.fallback = fallback;
    this.error = null;
  }

  wrap(fn) {
    try {
      return fn();
    } catch (e) {
      this.error = e;
      console.error('Boundary caught:', e);
      return this.fallback(e);
    }
  }
}

const boundary = new ErrorBoundary(
  (e) => `<div class="error">เกิดข้อผิดพลาด: ${e.message}</div>`
);

const html = boundary.wrap(() => {
  // อาจ throw error
  return renderComponent();
});
```

---

## Step 283: Error Handling กับ Async/Await

```javascript
// Patterns สำหรับ async error handling

// Pattern 1: try/catch กับ async/await
async function fetchUser(id) {
  try {
    const response = await fetch(`/api/users/${id}`);
    
    if (response.status === 404) {
      throw new NotFoundError('User', id);
    }
    if (!response.ok) {
      throw new HttpError(response.status, `HTTP error: ${response.status}`);
    }
    
    return await response.json();
  } catch (e) {
    if (e instanceof NotFoundError) {
      return null; // handle gracefully
    }
    console.error('fetchUser failed:', e);
    throw e; // rethrow
  }
}

// Pattern 2: Result type (Go-style)
async function tryCatch(promise) {
  try {
    return { data: await promise, error: null };
  } catch (error) {
    return { data: null, error };
  }
}

async function example() {
  const { data, error } = await tryCatch(fetchUser(1));
  if (error) {
    console.error('Error:', error.message);
    return;
  }
  console.log('User:', data);
}

// Pattern 3: Chaining async operations with error handling
async function createPost(userId, postData) {
  const user = await fetchUser(userId);
  if (!user) throw new NotFoundError('User', userId);

  const validated = validatePost(postData);
  if (!validated.isValid) throw new ValidationError(validated.errors.join(', '));

  const saved = await savePost({ ...postData, userId: user.id });
  await notifyFollowers(user, saved);

  return saved;
}

// Pattern 4: Parallel with individual error handling
async function loadPageData(userId) {
  const [userResult, postsResult, commentsResult] = await Promise.allSettled([
    fetchUser(userId),
    fetchPosts(userId),
    fetchComments(userId),
  ]);

  return {
    user: userResult.status === 'fulfilled' ? userResult.value : null,
    posts: postsResult.status === 'fulfilled' ? postsResult.value : [],
    comments: commentsResult.status === 'fulfilled' ? commentsResult.value : [],
    errors: [userResult, postsResult, commentsResult]
      .filter(r => r.status === 'rejected')
      .map(r => r.reason.message),
  };
}

// Pattern 5: Async error middleware
async function withErrorHandling(fn, handlers = {}) {
  try {
    return await fn();
  } catch (e) {
    const handler = handlers[e.constructor.name] || handlers.default;
    if (handler) return handler(e);
    throw e;
  }
}

const result = await withErrorHandling(
  () => fetchUser(999),
  {
    NotFoundError: (e) => ({ fallback: true, message: e.message }),
    NetworkError: (e) => ({ retry: true }),
    default: (e) => { throw e; },
  }
);
```

---

## Step 284: Logging และ Error Monitoring

```javascript
// Error logging system

class Logger {
  #levels = { debug: 0, info: 1, warn: 2, error: 3 };
  #currentLevel;
  #transports = [];

  constructor(level = 'info') {
    this.#currentLevel = this.#levels[level] || 1;
  }

  addTransport(transport) {
    this.#transports.push(transport);
    return this;
  }

  #log(level, message, data = {}) {
    if (this.#levels[level] < this.#currentLevel) return;

    const entry = {
      level,
      message,
      data,
      timestamp: new Date().toISOString(),
      url: window?.location?.href,
      userAgent: navigator?.userAgent,
    };

    this.#transports.forEach(transport => {
      try { transport(entry); } catch (e) { /* ignore transport errors */ }
    });
  }

  debug(message, data) { this.#log('debug', message, data); }
  info(message, data) { this.#log('info', message, data); }
  warn(message, data) { this.#log('warn', message, data); }
  error(message, data) { this.#log('error', message, data); }

  child(context) {
    const child = new Logger();
    child.#currentLevel = this.#currentLevel;
    child.#transports = this.#transports;
    return new Proxy(child, {
      get(target, prop) {
        const original = target[prop];
        if (['debug','info','warn','error'].includes(prop)) {
          return (msg, extra = {}) => original.call(target, msg, { ...context, ...extra });
        }
        return original;
      }
    });
  }
}

// Console transport
const consoleTransport = (entry) => {
  const fn = console[entry.level] || console.log;
  fn(`[${entry.timestamp}] [${entry.level.toUpperCase()}] ${entry.message}`, entry.data);
};

// Remote transport
const remoteTransport = async (entry) => {
  if (entry.level !== 'error') return;
  try {
    await fetch('/api/logs', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify(entry),
      keepalive: true,
    });
  } catch (e) { /* fail silently */ }
};

// LocalStorage transport (for offline)
const localTransport = (entry) => {
  try {
    const logs = JSON.parse(localStorage.getItem('error_logs') || '[]');
    logs.push(entry);
    if (logs.length > 100) logs.shift(); // keep last 100
    localStorage.setItem('error_logs', JSON.stringify(logs));
  } catch (e) { /* fail silently */ }
};

const logger = new Logger('debug')
  .addTransport(consoleTransport)
  .addTransport(remoteTransport)
  .addTransport(localTransport);

const userLogger = logger.child({ module: 'user', userId: '123' });
userLogger.info('User logged in');
userLogger.error('Payment failed', { orderId: 'ORD-001', amount: 150 });
```

---

## Step 285: Steps 286-290 - Advanced Error Handling

### Step 285: Error Boundary Pattern

```javascript
// Error Boundary สำหรับ component system

class ComponentError extends Error {
  constructor(message, component, originalError) {
    super(message);
    this.name = 'ComponentError';
    this.component = component;
    this.originalError = originalError;
  }
}

function createSafeComponent(Component) {
  return class SafeComponent {
    constructor(props) {
      this.props = props;
      this.error = null;
    }

    render() {
      if (this.error) {
        return `<div class="error-boundary">
          <p>Component failed to render: ${this.error.message}</p>
        </div>`;
      }
      try {
        return new Component(this.props).render();
      } catch (e) {
        this.error = e;
        console.error('Component error:', e);
        return this.render(); // render error state
      }
    }
  };
}
```

### Step 286: Structured Error Responses

```javascript
// API Error Response Pattern

function createErrorResponse(error, requestId) {
  const response = {
    success: false,
    requestId,
    timestamp: new Date().toISOString(),
    error: {
      code: error.code || 'UNKNOWN_ERROR',
      message: error.message,
      statusCode: error.statusCode || 500,
    },
  };

  if (error instanceof ValidationErrors) {
    response.error.fields = error.errors;
  }

  if (process.env.NODE_ENV !== 'production') {
    response.error.stack = error.stack;
  }

  return response;
}

// Client-side API error handling
async function apiCall(url, options = {}) {
  const response = await fetch(url, {
    headers: { 'Content-Type': 'application/json' },
    ...options,
  });

  const data = await response.json();

  if (!response.ok) {
    const error = new HttpError(
      response.status,
      data.error?.message || 'API request failed',
      data.error?.code
    );
    error.fields = data.error?.fields;
    throw error;
  }

  return data;
}
```

### Step 287: Circuit Breaker Pattern

```javascript
// Circuit Breaker - ป้องกัน cascade failures

class CircuitBreaker {
  #state = 'CLOSED'; // CLOSED, OPEN, HALF_OPEN
  #failures = 0;
  #successes = 0;
  #lastFailureTime = null;

  constructor(options = {}) {
    this.threshold = options.threshold || 5;
    this.timeout = options.timeout || 60000; // 1 minute
    this.successThreshold = options.successThreshold || 2;
  }

  async execute(fn) {
    if (this.#state === 'OPEN') {
      if (Date.now() - this.#lastFailureTime >= this.timeout) {
        this.#state = 'HALF_OPEN';
        console.log('Circuit Breaker: HALF_OPEN');
      } else {
        throw new Error('Circuit is OPEN - service unavailable');
      }
    }

    try {
      const result = await fn();
      this.#onSuccess();
      return result;
    } catch (e) {
      this.#onFailure();
      throw e;
    }
  }

  #onSuccess() {
    if (this.#state === 'HALF_OPEN') {
      this.#successes++;
      if (this.#successes >= this.successThreshold) {
        this.#state = 'CLOSED';
        this.#failures = 0;
        console.log('Circuit Breaker: CLOSED (recovered)');
      }
    } else {
      this.#failures = 0;
    }
  }

  #onFailure() {
    this.#failures++;
    this.#lastFailureTime = Date.now();
    if (this.#failures >= this.threshold) {
      this.#state = 'OPEN';
      console.log(`Circuit Breaker: OPEN after ${this.#failures} failures`);
    }
  }

  get state() { return this.#state; }
}

const breaker = new CircuitBreaker({ threshold: 3, timeout: 30000 });

async function callExternalService(data) {
  return breaker.execute(() => fetch('/api/external', {
    method: 'POST',
    body: JSON.stringify(data),
  }).then(r => r.json()));
}
```

### Step 288: Error Context และ Correlation

```javascript
// Error context สำหรับ debugging

class ErrorContext {
  static #context = {};
  static #requestId = null;

  static setRequestId(id) {
    this.#requestId = id;
  }

  static set(key, value) {
    this.#context[key] = value;
  }

  static get(key) {
    return this.#context[key];
  }

  static getAll() {
    return { ...this.#context, requestId: this.#requestId };
  }

  static clear() {
    this.#context = {};
    this.#requestId = null;
  }

  static enrichError(error) {
    error.context = this.getAll();
    return error;
  }
}

// ใช้งาน
async function handleRequest(req) {
  ErrorContext.setRequestId(req.headers['x-request-id'] || generateId());
  ErrorContext.set('userId', req.user?.id);
  ErrorContext.set('path', req.path);

  try {
    return await processRequest(req);
  } catch (e) {
    ErrorContext.enrichError(e);
    logger.error('Request failed', { error: e, context: e.context });
    throw e;
  } finally {
    ErrorContext.clear();
  }
}
```

### Step 289: Testing Error Scenarios

```javascript
// Unit tests สำหรับ error handling

describe('Error Handling Tests', () => {
  describe('ValidationError', () => {
    test('should throw ValidationError for invalid email', () => {
      expect(() => validateEmail('invalid')).toThrow(ValidationError);
      expect(() => validateEmail('invalid')).toThrow('Invalid email format');
    });

    test('should not throw for valid email', () => {
      expect(() => validateEmail('user@example.com')).not.toThrow();
    });
  });

  describe('async error handling', () => {
    test('should handle network error', async () => {
      global.fetch = jest.fn().mockRejectedValue(new TypeError('Network error'));
      await expect(fetchUser(1)).rejects.toThrow(NetworkError);
    });

    test('should handle 404 response', async () => {
      global.fetch = jest.fn().mockResolvedValue({
        ok: false, status: 404,
        json: () => Promise.resolve({ error: { message: 'Not found' } }),
      });
      await expect(fetchUser(999)).rejects.toThrow(NotFoundError);
    });
  });
});

// Manual test utility
async function runTests(tests) {
  const results = { pass: 0, fail: 0, errors: [] };

  for (const [name, fn] of Object.entries(tests)) {
    try {
      await fn();
      results.pass++;
      console.log(`✓ ${name}`);
    } catch (e) {
      results.fail++;
      results.errors.push({ test: name, error: e.message });
      console.error(`✗ ${name}: ${e.message}`);
    }
  }

  console.log(`\n${results.pass} passed, ${results.fail} failed`);
  return results;
}

// ใช้งาน
runTests({
  'validateEmail should throw for empty': () => {
    let threw = false;
    try { validateEmail(''); } catch (e) { threw = true; }
    if (!threw) throw new Error('Expected to throw');
  },
  'safeParseJSON should return default for invalid JSON': () => {
    const result = safeParseJSON('bad json', { default: true });
    if (!result.default) throw new Error('Expected default value');
  },
});
```

### Step 290: Complete Error Handling System

```javascript
// สรุป: Complete Error Handling System

// Error types hierarchy
class AppError extends Error {
  constructor(message, code, statusCode = 500, data = null) {
    super(message, { cause: data?.cause });
    this.name = this.constructor.name;
    this.code = code;
    this.statusCode = statusCode;
    this.data = data;
    this.timestamp = new Date().toISOString();
    Object.setPrototypeOf(this, new.target.prototype);
  }
  isOperational() { return this.statusCode < 500; }
}

// Specific errors
class ValidationError extends AppError {
  constructor(msg, field, value) {
    super(msg, 'VALIDATION_ERROR', 400);
    this.field = field;
    this.value = value;
  }
}
class AuthError extends AppError {
  constructor(msg = 'Unauthorized') { super(msg, 'AUTH_ERROR', 401); }
}
class ForbiddenError extends AppError {
  constructor(msg = 'Forbidden') { super(msg, 'FORBIDDEN', 403); }
}
class NotFoundError extends AppError {
  constructor(resource) { super(`${resource} not found`, 'NOT_FOUND', 404); }
}
class NetworkError extends AppError {
  constructor(msg = 'Network error') { super(msg, 'NETWORK_ERROR', 503); }
}

// Global error handler
class GlobalErrorHandler {
  static init() {
    window.addEventListener('error', this.handleWindowError.bind(this));
    window.addEventListener('unhandledrejection', this.handlePromiseRejection.bind(this));
  }

  static handleWindowError(event) {
    const { message, filename, lineno, colno, error } = event;
    this.report({ type: 'window', message, filename, lineno, colno, error });
  }

  static handlePromiseRejection(event) {
    event.preventDefault();
    this.report({ type: 'promise', reason: event.reason });
  }

  static report(info) {
    console.error('[GlobalErrorHandler]', info);
    // Send to monitoring service
    this.sendToMonitoring(info);
  }

  static sendToMonitoring(data) {
    navigator.sendBeacon('/api/errors', JSON.stringify({
      ...data,
      timestamp: new Date().toISOString(),
      url: location.href,
      userId: window.__userId,
    }));
  }
}

// Initialize
GlobalErrorHandler.init();

// Application error handler function
function handleAppError(error, context = {}) {
  const enriched = {
    name: error.name || 'Error',
    message: error.message,
    code: error.code,
    statusCode: error.statusCode || 500,
    stack: error.stack,
    context,
    timestamp: new Date().toISOString(),
  };

  if (error instanceof ValidationError) {
    showUserMessage('ข้อมูลไม่ถูกต้อง: ' + error.message, 'warning');
  } else if (error instanceof AuthError) {
    showUserMessage('กรุณาเข้าสู่ระบบ', 'error');
    redirectToLogin();
  } else if (error instanceof NotFoundError) {
    showUserMessage('ไม่พบข้อมูลที่ต้องการ', 'warning');
  } else if (error instanceof NetworkError) {
    showUserMessage('ไม่สามารถเชื่อมต่อได้ กรุณาตรวจสอบ internet', 'error');
  } else {
    showUserMessage('เกิดข้อผิดพลาด กรุณาลองใหม่อีกครั้ง', 'error');
    // Log unexpected errors
    logger.error('Unexpected error', enriched);
  }
}

function showUserMessage(message, type = 'info') {
  const el = document.createElement('div');
  el.className = `alert alert-${type}`;
  el.textContent = message;
  document.body.prepend(el);
  setTimeout(() => el.remove(), 5000);
}

console.log('Error Handling System initialized!');
```

---

## สรุป Steps 271-290

| Step | หัวข้อ |
|------|--------|
| 271 | Error Types (SyntaxError, TypeError, ReferenceError, RangeError) |
| 272 | try...catch |
| 273 | finally block |
| 274 | Rethrowing errors |
| 275 | Error object properties |
| 276 | Custom Error classes |
| 277 | Application error hierarchy |
| 278 | window.onerror / global error handling |
| 279 | Unhandled Promise rejections |
| 280 | Error handling best practices |
| 281 | console debugging methods |
| 282 | Defensive programming |
| 283 | Async/await error patterns |
| 284 | Logging and monitoring |
| 285 | Error boundary pattern |
| 286 | Structured error responses |
| 287 | Circuit breaker pattern |
| 288 | Error context and correlation |
| 289 | Testing error scenarios |
| 290 | Complete error handling system |

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: Custom Error Hierarchy
สร้าง error hierarchy สำหรับ e-commerce app:
- ProductNotFoundError
- OutOfStockError  
- PaymentError (subclasses: CardDeclinedError, InsufficientFundsError)
- ShippingError

### แบบฝึกหัดที่ 2: Retry Mechanism
สร้าง retry wrapper ที่:
- Exponential backoff
- Maximum retries limit
- Retry เฉพาะ retryable errors (network, timeout)
- Circuit breaker integration

### แบบฝึกหัดที่ 3: Error Reporting Dashboard
สร้าง dashboard ที่:
- แสดง errors ที่เกิดขึ้น
- Filter ตาม type/severity
- แสดง stack trace
- Export error logs

### แบบฝึกหัดที่ 4: Form Validation Error System
สร้าง validation system ที่:
- ใช้ custom error classes
- Accumulate multiple validation errors
- แสดง errors แบบ inline และ summary
- i18n error messages

### แบบฝึกหัดที่ 5: API Client ที่มี Error Handling ครบ
สร้าง API client ที่:
- Automatic retry with backoff
- Circuit breaker
- Request/Response logging
- Error transformation
- Timeout handling
