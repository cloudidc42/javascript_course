# Part 41: Error Handling ขั้นสูง (Steps 791-810)

## บทนำ

การจัดการข้อผิดพลาด (Error Handling) เป็นทักษะสำคัญที่นักพัฒนา JavaScript ต้องมี โปรแกรมที่ดีไม่ใช่แค่โปรแกรมที่ทำงานถูกต้อง แต่ยังต้องจัดการกับข้อผิดพลาดได้อย่างสวยงามและมีประสิทธิภาพ

ในบทนี้เราจะเรียนรู้:
- ประเภทของ Error ใน JavaScript
- การสร้าง Custom Error Classes
- กลยุทธ์การจัดการข้อผิดพลาดระดับขั้นสูง
- Pattern การออกแบบระบบที่ทนทานต่อข้อผิดพลาด

---

## Step 791: JavaScript Error Types พื้นฐาน

JavaScript มี Error types ในตัวหลายชนิด แต่ละชนิดมีวัตถุประสงค์ต่างกัน

### Error (Base Class)

```javascript
// Error พื้นฐาน - class แม่ของ Error ทุกชนิด
const err = new Error('เกิดข้อผิดพลาด');
console.log(err.message);  // 'เกิดข้อผิดพลาด'
console.log(err.name);     // 'Error'
console.log(err.stack);    // stack trace

// Properties ของ Error
try {
  throw new Error('ข้อความแสดงข้อผิดพลาด');
} catch (e) {
  console.log('name:', e.name);       // 'Error'
  console.log('message:', e.message); // 'ข้อความแสดงข้อผิดพลาด'
  console.log('stack:', e.stack);     // stack trace string
  console.log(e instanceof Error);    // true
}
```

### TypeError

```javascript
// TypeError - ใช้ type ไม่ถูกต้อง
try {
  null.property;  // ไม่สามารถเข้าถึง property ของ null ได้
} catch (e) {
  console.log(e instanceof TypeError);  // true
  console.log(e.name);    // 'TypeError'
  console.log(e.message); // 'Cannot read properties of null...'
}

// สร้าง TypeError เอง
function divide(a, b) {
  if (typeof a !== 'number' || typeof b !== 'number') {
    throw new TypeError(`คาดหวัง number แต่ได้รับ ${typeof a} และ ${typeof b}`);
  }
  return a / b;
}

try {
  divide('10', 2);
} catch (e) {
  console.log(e.message); // 'คาดหวัง number แต่ได้รับ string และ number'
}
```

### RangeError

```javascript
// RangeError - ค่าอยู่นอกช่วงที่อนุญาต
try {
  new Array(-1);  // ขนาด array ต้องเป็นบวก
} catch (e) {
  console.log(e instanceof RangeError); // true
  console.log(e.name);    // 'RangeError'
  console.log(e.message); // 'Invalid array length'
}

// ตัวอย่างเพิ่มเติม
try {
  const num = 1.5;
  num.toFixed(200);  // มากเกินไป
} catch (e) {
  console.log(e instanceof RangeError); // true
}

// สร้าง RangeError เอง
function setAge(age) {
  if (age < 0 || age > 150) {
    throw new RangeError(`อายุ ${age} ไม่ถูกต้อง ต้องอยู่ระหว่าง 0-150`);
  }
  return age;
}

try {
  setAge(-5);
} catch (e) {
  console.log(e.message); // 'อายุ -5 ไม่ถูกต้อง ต้องอยู่ระหว่าง 0-150'
}
```

### ReferenceError

```javascript
// ReferenceError - อ้างอิงตัวแปรที่ไม่มีอยู่
try {
  console.log(undeclaredVariable);
} catch (e) {
  console.log(e instanceof ReferenceError); // true
  console.log(e.name);    // 'ReferenceError'
  console.log(e.message); // 'undeclaredVariable is not defined'
}

// Temporal Dead Zone
try {
  console.log(myLet);
  let myLet = 5;
} catch (e) {
  console.log(e instanceof ReferenceError); // true
}
```

### SyntaxError

```javascript
// SyntaxError - โค้ดมี syntax ผิด
try {
  eval('if (');  // syntax ผิด
} catch (e) {
  console.log(e instanceof SyntaxError); // true
  console.log(e.name);    // 'SyntaxError'
}

try {
  JSON.parse('{invalid json}');  // JSON syntax ผิด
} catch (e) {
  console.log(e instanceof SyntaxError); // true
  console.log(e.message); // 'Unexpected token...'
}
```

### URIError

```javascript
// URIError - URI ไม่ถูกต้อง
try {
  decodeURIComponent('%');  // % ไม่สมบูรณ์
} catch (e) {
  console.log(e instanceof URIError); // true
  console.log(e.name);    // 'URIError'
}

try {
  decodeURI('%2');  // URI sequence ไม่สมบูรณ์
} catch (e) {
  console.log(e instanceof URIError); // true
}
```

### EvalError (Legacy)

```javascript
// EvalError - ปัจจุบันแทบไม่ใช้แล้ว แต่ยังมีอยู่
// ใน modern JS ไม่ throw EvalError อีกต่อไป
// แต่สามารถสร้างได้
const evalErr = new EvalError('eval() ถูกใช้ผิดวิธี');
console.log(evalErr.name);    // 'EvalError'
console.log(evalErr.message); // 'eval() ถูกใช้ผิดวิธี'
```

---

## Step 792: Custom Error Classes

การสร้าง Custom Error ช่วยให้โค้ดชัดเจนและจัดการได้ง่ายกว่า

```javascript
// Custom Error พื้นฐาน
class AppError extends Error {
  constructor(message, code) {
    super(message);
    this.name = 'AppError';
    this.code = code;
    // ช่วยให้ instanceof ทำงานถูกต้องใน ES5 transpile
    if (Error.captureStackTrace) {
      Error.captureStackTrace(this, AppError);
    }
  }
}

// ใช้งาน
try {
  throw new AppError('เกิดข้อผิดพลาดในแอปพลิเคชัน', 'APP_ERROR');
} catch (e) {
  console.log(e instanceof AppError);  // true
  console.log(e instanceof Error);     // true
  console.log(e.name);    // 'AppError'
  console.log(e.message); // 'เกิดข้อผิดพลาดในแอปพลิเคชัน'
  console.log(e.code);    // 'APP_ERROR'
}
```

```javascript
// Custom Error ที่สมบูรณ์กว่า
class CustomError extends Error {
  constructor(message, options = {}) {
    super(message);
    
    // ตั้งชื่อ
    this.name = this.constructor.name;
    
    // ข้อมูลเพิ่มเติม
    this.code = options.code || 'UNKNOWN_ERROR';
    this.statusCode = options.statusCode || 500;
    this.details = options.details || null;
    this.timestamp = new Date().toISOString();
    
    // Capture stack trace
    if (Error.captureStackTrace) {
      Error.captureStackTrace(this, this.constructor);
    }
  }
  
  // แปลงเป็น JSON
  toJSON() {
    return {
      name: this.name,
      message: this.message,
      code: this.code,
      statusCode: this.statusCode,
      details: this.details,
      timestamp: this.timestamp
    };
  }
  
  // แสดงข้อมูลแบบสวยงาม
  toString() {
    return `[${this.code}] ${this.name}: ${this.message}`;
  }
}

// ทดสอบ
const err = new CustomError('ไม่พบข้อมูล', {
  code: 'NOT_FOUND',
  statusCode: 404,
  details: { id: 123 }
});

console.log(err.toString());
// '[NOT_FOUND] CustomError: ไม่พบข้อมูล'

console.log(JSON.stringify(err.toJSON(), null, 2));
```

---

## Step 793: Error Codes และ Categories

การจัดระเบียบ Error ด้วย codes และ categories ช่วยให้จัดการได้ง่ายขึ้น

```javascript
// Error Codes constants
const ErrorCodes = Object.freeze({
  // Authentication & Authorization
  UNAUTHORIZED: 'UNAUTHORIZED',
  FORBIDDEN: 'FORBIDDEN',
  TOKEN_EXPIRED: 'TOKEN_EXPIRED',
  INVALID_CREDENTIALS: 'INVALID_CREDENTIALS',
  
  // Validation
  VALIDATION_ERROR: 'VALIDATION_ERROR',
  INVALID_INPUT: 'INVALID_INPUT',
  MISSING_FIELD: 'MISSING_FIELD',
  INVALID_FORMAT: 'INVALID_FORMAT',
  
  // Resource
  NOT_FOUND: 'NOT_FOUND',
  ALREADY_EXISTS: 'ALREADY_EXISTS',
  RESOURCE_LOCKED: 'RESOURCE_LOCKED',
  
  // Network
  NETWORK_ERROR: 'NETWORK_ERROR',
  TIMEOUT: 'TIMEOUT',
  CONNECTION_REFUSED: 'CONNECTION_REFUSED',
  
  // Database
  DB_ERROR: 'DB_ERROR',
  DB_CONNECTION_ERROR: 'DB_CONNECTION_ERROR',
  DB_QUERY_ERROR: 'DB_QUERY_ERROR',
  
  // Business Logic
  INSUFFICIENT_FUNDS: 'INSUFFICIENT_FUNDS',
  QUOTA_EXCEEDED: 'QUOTA_EXCEEDED',
  RATE_LIMIT_EXCEEDED: 'RATE_LIMIT_EXCEEDED',
});

// Error Categories
const ErrorCategories = {
  CLIENT: 'CLIENT',      // ความผิดพลาดจากฝั่ง client
  SERVER: 'SERVER',      // ความผิดพลาดจากฝั่ง server
  NETWORK: 'NETWORK',    // ปัญหาเครือข่าย
  DATABASE: 'DATABASE',  // ปัญหาฐานข้อมูล
  SECURITY: 'SECURITY',  // ปัญหาความปลอดภัย
};

// Mapping ระหว่าง code กับ category
const ErrorCodeToCategory = {
  [ErrorCodes.UNAUTHORIZED]: ErrorCategories.SECURITY,
  [ErrorCodes.FORBIDDEN]: ErrorCategories.SECURITY,
  [ErrorCodes.TOKEN_EXPIRED]: ErrorCategories.SECURITY,
  [ErrorCodes.VALIDATION_ERROR]: ErrorCategories.CLIENT,
  [ErrorCodes.NOT_FOUND]: ErrorCategories.CLIENT,
  [ErrorCodes.DB_ERROR]: ErrorCategories.DATABASE,
  [ErrorCodes.NETWORK_ERROR]: ErrorCategories.NETWORK,
};

function getErrorCategory(code) {
  return ErrorCodeToCategory[code] || ErrorCategories.SERVER;
}

// ตัวอย่างการใช้งาน
console.log(getErrorCategory(ErrorCodes.UNAUTHORIZED)); // 'SECURITY'
console.log(getErrorCategory(ErrorCodes.NOT_FOUND));    // 'CLIENT'
```

---

## Step 794: HTTP Error Classes

การสร้าง HTTP Error classes สำหรับ web applications

```javascript
// Base HTTP Error
class HttpError extends Error {
  constructor(message, statusCode = 500, code = 'HTTP_ERROR') {
    super(message);
    this.name = this.constructor.name;
    this.statusCode = statusCode;
    this.code = code;
    this.isHttpError = true;
    
    if (Error.captureStackTrace) {
      Error.captureStackTrace(this, this.constructor);
    }
  }
  
  get isClientError() {
    return this.statusCode >= 400 && this.statusCode < 500;
  }
  
  get isServerError() {
    return this.statusCode >= 500;
  }
  
  toJSON() {
    return {
      error: {
        name: this.name,
        message: this.message,
        code: this.code,
        statusCode: this.statusCode
      }
    };
  }
}

// 400 Bad Request
class BadRequestError extends HttpError {
  constructor(message = 'คำขอไม่ถูกต้อง', details = null) {
    super(message, 400, 'BAD_REQUEST');
    this.details = details;
  }
}

// 401 Unauthorized
class UnauthorizedError extends HttpError {
  constructor(message = 'ต้องการการยืนยันตัวตน') {
    super(message, 401, 'UNAUTHORIZED');
  }
}

// 403 Forbidden
class ForbiddenError extends HttpError {
  constructor(message = 'ไม่มีสิทธิ์เข้าถึง') {
    super(message, 403, 'FORBIDDEN');
  }
}

// 404 Not Found
class NotFoundError extends HttpError {
  constructor(resource = 'ทรัพยากร', id = null) {
    const message = id 
      ? `ไม่พบ ${resource} ที่มี id: ${id}`
      : `ไม่พบ ${resource}`;
    super(message, 404, 'NOT_FOUND');
    this.resource = resource;
    this.resourceId = id;
  }
}

// 409 Conflict
class ConflictError extends HttpError {
  constructor(message = 'ข้อมูลขัดแย้งกัน') {
    super(message, 409, 'CONFLICT');
  }
}

// 422 Unprocessable Entity
class UnprocessableEntityError extends HttpError {
  constructor(message = 'ข้อมูลไม่ถูกต้อง', errors = []) {
    super(message, 422, 'UNPROCESSABLE_ENTITY');
    this.errors = errors;
  }
}

// 429 Too Many Requests
class TooManyRequestsError extends HttpError {
  constructor(retryAfter = null) {
    super('คำขอมากเกินไป กรุณารอสักครู่', 429, 'TOO_MANY_REQUESTS');
    this.retryAfter = retryAfter;
  }
}

// 500 Internal Server Error
class InternalServerError extends HttpError {
  constructor(message = 'เกิดข้อผิดพลาดภายในเซิร์ฟเวอร์') {
    super(message, 500, 'INTERNAL_SERVER_ERROR');
  }
}

// 503 Service Unavailable
class ServiceUnavailableError extends HttpError {
  constructor(service = 'บริการ', retryAfter = null) {
    super(`${service} ไม่พร้อมใช้งาน`, 503, 'SERVICE_UNAVAILABLE');
    this.service = service;
    this.retryAfter = retryAfter;
  }
}

// ตัวอย่างการใช้งาน
function getUserById(id) {
  const users = { 1: { name: 'สมชาย' }, 2: { name: 'สมหญิง' } };
  const user = users[id];
  
  if (!user) {
    throw new NotFoundError('ผู้ใช้', id);
  }
  
  return user;
}

try {
  getUserById(999);
} catch (e) {
  if (e instanceof NotFoundError) {
    console.log(`Status: ${e.statusCode}`); // 404
    console.log(e.message); // 'ไม่พบ ผู้ใช้ ที่มี id: 999'
  }
}

// Error handler middleware (Express-style)
function errorHandler(err, req, res, next) {
  if (err instanceof HttpError) {
    return res.status(err.statusCode).json(err.toJSON());
  }
  
  // Unknown error
  console.error('Unexpected error:', err);
  return res.status(500).json({
    error: {
      message: 'เกิดข้อผิดพลาดที่ไม่คาดคิด',
      code: 'INTERNAL_SERVER_ERROR'
    }
  });
}
```

---

## Step 795: Validation Error Classes

```javascript
// Validation Error ที่ซับซ้อน
class ValidationError extends Error {
  constructor(message = 'ข้อมูลไม่ถูกต้อง') {
    super(message);
    this.name = 'ValidationError';
    this.errors = [];
    this.statusCode = 422;
    
    if (Error.captureStackTrace) {
      Error.captureStackTrace(this, ValidationError);
    }
  }
  
  addError(field, message, value = undefined) {
    this.errors.push({ field, message, value });
    return this;
  }
  
  hasErrors() {
    return this.errors.length > 0;
  }
  
  getFieldErrors(field) {
    return this.errors.filter(e => e.field === field);
  }
  
  toJSON() {
    return {
      message: this.message,
      errors: this.errors
    };
  }
}

// Validator class ที่ใช้ ValidationError
class Validator {
  constructor(data) {
    this.data = data;
    this.error = new ValidationError();
  }
  
  required(field, message) {
    if (!this.data[field] && this.data[field] !== 0) {
      this.error.addError(field, message || `${field} จำเป็นต้องกรอก`);
    }
    return this;
  }
  
  minLength(field, min, message) {
    const value = this.data[field];
    if (value && value.length < min) {
      this.error.addError(
        field, 
        message || `${field} ต้องมีความยาวอย่างน้อย ${min} ตัวอักษร`,
        value
      );
    }
    return this;
  }
  
  maxLength(field, max, message) {
    const value = this.data[field];
    if (value && value.length > max) {
      this.error.addError(
        field,
        message || `${field} ต้องมีความยาวไม่เกิน ${max} ตัวอักษร`,
        value
      );
    }
    return this;
  }
  
  email(field, message) {
    const value = this.data[field];
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (value && !emailRegex.test(value)) {
      this.error.addError(
        field,
        message || `${field} ไม่ใช่อีเมลที่ถูกต้อง`,
        value
      );
    }
    return this;
  }
  
  min(field, min, message) {
    const value = this.data[field];
    if (value !== undefined && value < min) {
      this.error.addError(
        field,
        message || `${field} ต้องมีค่าอย่างน้อย ${min}`,
        value
      );
    }
    return this;
  }
  
  max(field, max, message) {
    const value = this.data[field];
    if (value !== undefined && value > max) {
      this.error.addError(
        field,
        message || `${field} ต้องมีค่าไม่เกิน ${max}`,
        value
      );
    }
    return this;
  }
  
  custom(field, fn, message) {
    const value = this.data[field];
    if (!fn(value)) {
      this.error.addError(field, message, value);
    }
    return this;
  }
  
  validate() {
    if (this.error.hasErrors()) {
      throw this.error;
    }
    return this.data;
  }
}

// ตัวอย่างการใช้งาน
function createUser(userData) {
  new Validator(userData)
    .required('username', 'กรุณากรอกชื่อผู้ใช้')
    .minLength('username', 3, 'ชื่อผู้ใช้ต้องมีอย่างน้อย 3 ตัวอักษร')
    .maxLength('username', 20, 'ชื่อผู้ใช้ต้องไม่เกิน 20 ตัวอักษร')
    .required('email', 'กรุณากรอกอีเมล')
    .email('email', 'อีเมลไม่ถูกต้อง')
    .required('age', 'กรุณากรอกอายุ')
    .min('age', 18, 'ต้องมีอายุอย่างน้อย 18 ปี')
    .max('age', 120, 'อายุไม่ถูกต้อง')
    .validate();
  
  console.log('ผู้ใช้ถูกสร้างสำเร็จ:', userData);
}

try {
  createUser({ username: 'ab', email: 'invalid-email', age: 15 });
} catch (e) {
  if (e instanceof ValidationError) {
    console.log('Validation errors:');
    e.errors.forEach(err => {
      console.log(`  ${err.field}: ${err.message}`);
    });
  }
}
// Output:
// Validation errors:
//   username: ชื่อผู้ใช้ต้องมีอย่างน้อย 3 ตัวอักษร
//   email: อีเมลไม่ถูกต้อง
//   age: ต้องมีอายุอย่างน้อย 18 ปี
```

---

## Step 796: Database Error Classes

```javascript
// Database Error hierarchy
class DatabaseError extends Error {
  constructor(message, options = {}) {
    super(message);
    this.name = this.constructor.name;
    this.code = options.code || 'DB_ERROR';
    this.query = options.query || null;
    this.params = options.params || null;
    this.originalError = options.originalError || null;
    
    if (Error.captureStackTrace) {
      Error.captureStackTrace(this, this.constructor);
    }
  }
}

class ConnectionError extends DatabaseError {
  constructor(host, port, originalError = null) {
    super(`ไม่สามารถเชื่อมต่อฐานข้อมูลที่ ${host}:${port}`);
    this.code = 'DB_CONNECTION_ERROR';
    this.host = host;
    this.port = port;
    this.originalError = originalError;
  }
}

class QueryError extends DatabaseError {
  constructor(query, params, originalError = null) {
    super(`Query ไม่สำเร็จ: ${originalError?.message || 'unknown error'}`);
    this.code = 'DB_QUERY_ERROR';
    this.query = query;
    this.params = params;
    this.originalError = originalError;
  }
}

class DuplicateKeyError extends DatabaseError {
  constructor(table, field, value) {
    super(`ข้อมูลซ้ำใน ${table}: ${field} = ${value}`);
    this.code = 'DB_DUPLICATE_KEY';
    this.table = table;
    this.field = field;
    this.value = value;
  }
}

class RecordNotFoundError extends DatabaseError {
  constructor(table, criteria) {
    super(`ไม่พบข้อมูลใน ${table}`);
    this.code = 'DB_RECORD_NOT_FOUND';
    this.table = table;
    this.criteria = criteria;
  }
}

class TransactionError extends DatabaseError {
  constructor(operation, originalError = null) {
    super(`Transaction ไม่สำเร็จในขั้นตอน: ${operation}`);
    this.code = 'DB_TRANSACTION_ERROR';
    this.operation = operation;
    this.originalError = originalError;
  }
}

// ตัวอย่าง Database wrapper
class DatabaseClient {
  constructor(config) {
    this.config = config;
    this.connected = false;
  }
  
  async connect() {
    try {
      // Simulate connection
      if (!this.config.host) {
        throw new Error('Host not specified');
      }
      this.connected = true;
      console.log('เชื่อมต่อฐานข้อมูลสำเร็จ');
    } catch (error) {
      throw new ConnectionError(
        this.config.host || 'unknown',
        this.config.port || 5432,
        error
      );
    }
  }
  
  async query(sql, params = []) {
    if (!this.connected) {
      throw new ConnectionError(this.config.host, this.config.port);
    }
    
    try {
      // Simulate query execution
      console.log('Running query:', sql, params);
      return { rows: [], rowCount: 0 };
    } catch (error) {
      if (error.code === '23505') {
        // PostgreSQL unique violation
        throw new DuplicateKeyError('unknown', 'unknown', 'unknown');
      }
      throw new QueryError(sql, params, error);
    }
  }
  
  async transaction(fn) {
    try {
      await this.query('BEGIN');
      const result = await fn(this);
      await this.query('COMMIT');
      return result;
    } catch (error) {
      await this.query('ROLLBACK');
      if (error instanceof DatabaseError) throw error;
      throw new TransactionError('execute', error);
    }
  }
}

// ใช้งาน
async function createOrder(userId, items) {
  const db = new DatabaseClient({ host: 'localhost', port: 5432 });
  
  try {
    await db.connect();
    
    return await db.transaction(async (client) => {
      const order = await client.query(
        'INSERT INTO orders (user_id) VALUES ($1) RETURNING id',
        [userId]
      );
      
      for (const item of items) {
        await client.query(
          'INSERT INTO order_items (order_id, product_id, qty) VALUES ($1, $2, $3)',
          [order.rows[0]?.id, item.productId, item.qty]
        );
      }
      
      return order.rows[0];
    });
    
  } catch (error) {
    if (error instanceof DuplicateKeyError) {
      console.error('ข้อมูลซ้ำ:', error.message);
    } else if (error instanceof ConnectionError) {
      console.error('เชื่อมต่อไม่ได้:', error.message);
    } else if (error instanceof TransactionError) {
      console.error('Transaction ผิดพลาด:', error.message);
    } else {
      throw error; // re-throw error ที่ไม่รู้จัก
    }
  }
}
```

---

## Step 797: Error Handling Strategies

### Strategy 1: Try-Catch-Finally Pattern

```javascript
// Pattern พื้นฐาน
async function fetchData(url) {
  let response;
  
  try {
    response = await fetch(url);
    
    if (!response.ok) {
      throw new HttpError(`HTTP error! status: ${response.status}`, response.status);
    }
    
    const data = await response.json();
    return data;
    
  } catch (error) {
    if (error instanceof HttpError) {
      // จัดการ HTTP errors
      console.error(`HTTP ${error.statusCode}: ${error.message}`);
      return null;
    }
    
    if (error instanceof SyntaxError) {
      // JSON parse error
      console.error('ไม่สามารถแปลง JSON ได้:', error.message);
      return null;
    }
    
    // Network errors, etc.
    console.error('เกิดข้อผิดพลาดที่ไม่คาดคิด:', error.message);
    throw error; // re-throw
    
  } finally {
    // ทำงานเสมอ ไม่ว่า try หรือ catch จะทำงาน
    console.log('Fetch operation completed');
  }
}
```

### Strategy 2: Error Wrapping

```javascript
// Error wrapping - ห่อ low-level error เป็น high-level error
class ServiceError extends Error {
  constructor(service, operation, cause) {
    super(`${service} ล้มเหลวในการ ${operation}: ${cause.message}`);
    this.name = 'ServiceError';
    this.service = service;
    this.operation = operation;
    this.cause = cause; // ES2022 standard
    
    if (Error.captureStackTrace) {
      Error.captureStackTrace(this, ServiceError);
    }
  }
}

async function getUserProfile(userId) {
  try {
    const user = await db.findUser(userId);
    const profile = await api.getProfile(user.profileId);
    return { ...user, ...profile };
  } catch (error) {
    // Wrap ข้อผิดพลาด original เป็น ServiceError
    throw new ServiceError('UserService', 'getUserProfile', error);
  }
}
```

### Strategy 3: Result Pattern (Functional approach)

```javascript
// Result type - ไม่ throw exception แต่ return object
class Result {
  constructor(value, error) {
    this._value = value;
    this._error = error;
    this._isOk = error === null;
  }
  
  static ok(value) {
    return new Result(value, null);
  }
  
  static err(error) {
    return new Result(null, error instanceof Error ? error : new Error(String(error)));
  }
  
  get isOk() { return this._isOk; }
  get isErr() { return !this._isOk; }
  get value() { 
    if (!this._isOk) throw new Error('Cannot get value of error result');
    return this._value; 
  }
  get error() { return this._error; }
  
  map(fn) {
    if (this._isOk) {
      try {
        return Result.ok(fn(this._value));
      } catch (e) {
        return Result.err(e);
      }
    }
    return this;
  }
  
  flatMap(fn) {
    if (this._isOk) {
      try {
        return fn(this._value);
      } catch (e) {
        return Result.err(e);
      }
    }
    return this;
  }
  
  getOrElse(defaultValue) {
    return this._isOk ? this._value : defaultValue;
  }
  
  getOrThrow() {
    if (!this._isOk) throw this._error;
    return this._value;
  }
  
  match({ ok, err }) {
    return this._isOk ? ok(this._value) : err(this._error);
  }
}

// ใช้งาน Result pattern
function divide(a, b) {
  if (b === 0) {
    return Result.err(new RangeError('ไม่สามารถหารด้วย 0 ได้'));
  }
  return Result.ok(a / b);
}

function parseInt10(str) {
  const num = parseInt(str, 10);
  if (isNaN(num)) {
    return Result.err(new TypeError(`"${str}" ไม่ใช่ตัวเลข`));
  }
  return Result.ok(num);
}

// Chain operations
const result = parseInt10('10')
  .flatMap(n => divide(n, 2))
  .map(n => n * 3);

result.match({
  ok: value => console.log('ผลลัพธ์:', value),    // 'ผลลัพธ์: 15'
  err: error => console.log('ข้อผิดพลาด:', error.message)
});

// กรณีมีข้อผิดพลาด
const errorResult = parseInt10('abc')
  .flatMap(n => divide(n, 2));

errorResult.match({
  ok: value => console.log('ผลลัพธ์:', value),
  err: error => console.log('ข้อผิดพลาด:', error.message) // '"abc" ไม่ใช่ตัวเลข'
});
```

---

## Step 798: Global Error Handlers (Browser)

```javascript
// window.onerror - จับ uncaught errors
window.onerror = function(message, source, lineno, colno, error) {
  console.error('Global error caught:');
  console.error('  Message:', message);
  console.error('  Source:', source);
  console.error('  Line:', lineno, 'Col:', colno);
  console.error('  Error object:', error);
  
  // ส่ง error ไปยัง logging service
  logErrorToService({
    type: 'UNCAUGHT_ERROR',
    message,
    source,
    lineno,
    colno,
    stack: error?.stack
  });
  
  // return true เพื่อป้องกัน default browser error handling
  // return false (default) แสดง error ใน console ตามปกติ
  return false;
};

// window.onunhandledrejection - จับ unhandled Promise rejections
window.addEventListener('unhandledrejection', function(event) {
  console.error('Unhandled Promise Rejection:');
  console.error('  Reason:', event.reason);
  console.error('  Promise:', event.promise);
  
  // ส่ง error ไปยัง logging service
  logErrorToService({
    type: 'UNHANDLED_REJECTION',
    reason: event.reason?.message || String(event.reason),
    stack: event.reason?.stack
  });
  
  // ป้องกัน default handling (แสดงใน console)
  event.preventDefault();
});

// ตัวอย่าง error logging service
function logErrorToService(errorData) {
  // ในการใช้งานจริง ส่ง error ไปยัง server
  console.log('Logging to error service:', errorData);
  
  // ตัวอย่าง fetch
  // fetch('/api/errors', {
  //   method: 'POST',
  //   headers: { 'Content-Type': 'application/json' },
  //   body: JSON.stringify({
  //     ...errorData,
  //     userAgent: navigator.userAgent,
  //     url: window.location.href,
  //     timestamp: new Date().toISOString()
  //   })
  // }).catch(console.error);
}

// ตัวอย่าง errors ที่จะถูก global handlers จับ
// setTimeout(() => {
//   throw new Error('ข้อผิดพลาดใน setTimeout');
// }, 1000);

// Promise.reject(new Error('Unhandled rejection'));
```

---

## Step 799: Node.js Global Error Handlers

```javascript
// Node.js process error handlers

// process.on('uncaughtException') - จับ synchronous uncaught errors
process.on('uncaughtException', (error, origin) => {
  console.error('Uncaught Exception:');
  console.error('  Error:', error.message);
  console.error('  Origin:', origin);
  console.error('  Stack:', error.stack);
  
  // Log to file or monitoring service
  logError('uncaughtException', error, { origin });
  
  // IMPORTANT: หลังจาก uncaughtException ควร exit process
  // เพราะ application state อาจ inconsistent
  process.exit(1);
});

// process.on('unhandledRejection') - จับ unhandled Promise rejections
process.on('unhandledRejection', (reason, promise) => {
  console.error('Unhandled Rejection at:', promise);
  console.error('Reason:', reason);
  
  if (reason instanceof Error) {
    console.error('Stack:', reason.stack);
  }
  
  // Log แต่ไม่จำเป็นต้อง exit (ขึ้นอยู่กับ policy)
  logError('unhandledRejection', reason);
  
  // In newer Node.js versions, process will exit automatically
  // unless you handle this event
});

// SIGTERM และ SIGINT สำหรับ graceful shutdown
process.on('SIGTERM', async () => {
  console.log('SIGTERM received, shutting down gracefully...');
  await gracefulShutdown();
  process.exit(0);
});

process.on('SIGINT', async () => {
  console.log('SIGINT (Ctrl+C) received, shutting down gracefully...');
  await gracefulShutdown();
  process.exit(0);
});

async function gracefulShutdown() {
  // ปิด connections ที่เปิดอยู่
  // await database.close();
  // await server.close();
  console.log('Graceful shutdown completed');
}

function logError(type, error, extra = {}) {
  const errorLog = {
    type,
    message: error?.message || String(error),
    stack: error?.stack,
    timestamp: new Date().toISOString(),
    ...extra
  };
  
  // เขียนลงไฟล์หรือ monitoring service
  console.error(JSON.stringify(errorLog));
}
```

---

## Step 800: Centralized Error Handling

```javascript
// Centralized Error Handler class
class ErrorHandler {
  constructor(options = {}) {
    this.loggers = options.loggers || [console];
    this.reporters = options.reporters || [];
    this.errorCounts = new Map();
    this.suppressedErrors = new Set();
  }
  
  // จัดการ error หลัก
  handle(error, context = {}) {
    // นับ errors
    this.#trackError(error);
    
    // Log error
    this.#log(error, context);
    
    // Report ถ้าสำคัญ
    if (this.#isImportant(error)) {
      this.#report(error, context);
    }
    
    return this.#createResponse(error);
  }
  
  #trackError(error) {
    const key = `${error.name}:${error.code || 'unknown'}`;
    this.errorCounts.set(key, (this.errorCounts.get(key) || 0) + 1);
  }
  
  #log(error, context) {
    const logData = {
      timestamp: new Date().toISOString(),
      name: error.name,
      message: error.message,
      code: error.code,
      statusCode: error.statusCode,
      stack: error.stack,
      context
    };
    
    this.loggers.forEach(logger => {
      if (error.statusCode >= 500 || !error.statusCode) {
        logger.error?.('[ERROR]', logData) || console.error('[ERROR]', logData);
      } else {
        logger.warn?.('[WARN]', logData) || console.warn('[WARN]', logData);
      }
    });
  }
  
  #isImportant(error) {
    // ข้อผิดพลาด server-side สำคัญ
    if (!error.statusCode || error.statusCode >= 500) return true;
    // ข้อผิดพลาด security สำคัญ
    if (['UNAUTHORIZED', 'FORBIDDEN'].includes(error.code)) return true;
    return false;
  }
  
  #report(error, context) {
    this.reporters.forEach(reporter => {
      reporter.report(error, context).catch(e => {
        console.error('Failed to report error:', e.message);
      });
    });
  }
  
  #createResponse(error) {
    if (error instanceof HttpError) {
      return {
        success: false,
        error: {
          message: error.message,
          code: error.code,
          statusCode: error.statusCode
        }
      };
    }
    
    // Unknown errors - ไม่เปิดเผยรายละเอียด
    return {
      success: false,
      error: {
        message: 'เกิดข้อผิดพลาดภายใน',
        code: 'INTERNAL_ERROR',
        statusCode: 500
      }
    };
  }
  
  // ดู error statistics
  getStats() {
    const stats = {};
    this.errorCounts.forEach((count, key) => {
      stats[key] = count;
    });
    return stats;
  }
}

// Singleton instance
const errorHandler = new ErrorHandler({
  reporters: [
    {
      report: async (error, context) => {
        // ส่ง error ไปยัง Sentry, Datadog, etc.
        console.log('Reporting to monitoring service:', error.message);
      }
    }
  ]
});

// Express middleware
function createErrorMiddleware(handler) {
  return function(err, req, res, next) {
    const response = handler.handle(err, {
      method: req.method,
      url: req.url,
      headers: req.headers,
      body: req.body
    });
    
    const statusCode = err.statusCode || 500;
    res.status(statusCode).json(response);
  };
}
```

---

## Step 801: Error Logging Patterns

```javascript
// Logger class
class Logger {
  constructor(options = {}) {
    this.level = options.level || 'info';
    this.prefix = options.prefix || '';
    this.transports = options.transports || ['console'];
    
    this.levels = {
      debug: 0,
      info: 1,
      warn: 2,
      error: 3,
      fatal: 4
    };
  }
  
  #shouldLog(level) {
    return this.levels[level] >= this.levels[this.level];
  }
  
  #format(level, message, data) {
    return {
      level,
      message: this.prefix ? `[${this.prefix}] ${message}` : message,
      timestamp: new Date().toISOString(),
      ...data
    };
  }
  
  #output(level, logData) {
    this.transports.forEach(transport => {
      if (transport === 'console') {
        const method = level === 'error' || level === 'fatal' ? 'error' :
                       level === 'warn' ? 'warn' : 'log';
        console[method](JSON.stringify(logData));
      }
    });
  }
  
  debug(message, data = {}) {
    if (this.#shouldLog('debug')) {
      this.#output('debug', this.#format('debug', message, data));
    }
  }
  
  info(message, data = {}) {
    if (this.#shouldLog('info')) {
      this.#output('info', this.#format('info', message, data));
    }
  }
  
  warn(message, data = {}) {
    if (this.#shouldLog('warn')) {
      this.#output('warn', this.#format('warn', message, data));
    }
  }
  
  error(message, error = null, data = {}) {
    if (this.#shouldLog('error')) {
      const errorData = error ? {
        errorName: error.name,
        errorMessage: error.message,
        errorCode: error.code,
        stack: error.stack
      } : {};
      
      this.#output('error', this.#format('error', message, { ...errorData, ...data }));
    }
  }
  
  fatal(message, error = null, data = {}) {
    if (this.#shouldLog('fatal')) {
      const errorData = error ? {
        errorName: error.name,
        errorMessage: error.message,
        errorCode: error.code,
        stack: error.stack
      } : {};
      
      this.#output('fatal', this.#format('fatal', message, { ...errorData, ...data }));
    }
  }
}

// ใช้งาน Logger
const logger = new Logger({
  level: 'debug',
  prefix: 'MyApp',
  transports: ['console']
});

logger.debug('เริ่มต้น process', { userId: 123 });
logger.info('ผู้ใช้เข้าสู่ระบบ', { userId: 123, email: 'test@example.com' });

try {
  throw new Error('DB connection failed');
} catch (error) {
  logger.error('ไม่สามารถเชื่อมต่อฐานข้อมูลได้', error, { host: 'localhost' });
}

// Structured error logging pattern
function logError(logger, error, context = {}) {
  const logData = {
    errorType: error.constructor.name,
    code: error.code,
    statusCode: error.statusCode,
    ...context
  };
  
  if (error.statusCode && error.statusCode < 500) {
    logger.warn(`Client error: ${error.message}`, logData);
  } else {
    logger.error(`Server error: ${error.message}`, error, logData);
  }
}
```

---

## Step 802: User-Friendly Error Messages

```javascript
// Error message mapping
const UserFriendlyMessages = {
  // Network
  'NETWORK_ERROR': 'ไม่สามารถเชื่อมต่ออินเทอร์เน็ตได้ กรุณาตรวจสอบการเชื่อมต่อของท่าน',
  'TIMEOUT': 'การเชื่อมต่อหมดเวลา กรุณาลองใหม่อีกครั้ง',
  
  // Authentication
  'UNAUTHORIZED': 'กรุณาเข้าสู่ระบบก่อนดำเนินการ',
  'TOKEN_EXPIRED': 'เซสชันของท่านหมดอายุ กรุณาเข้าสู่ระบบใหม่',
  'INVALID_CREDENTIALS': 'ชื่อผู้ใช้หรือรหัสผ่านไม่ถูกต้อง',
  
  // Authorization
  'FORBIDDEN': 'ท่านไม่มีสิทธิ์ดำเนินการนี้',
  
  // Resource
  'NOT_FOUND': 'ไม่พบข้อมูลที่ต้องการ',
  'ALREADY_EXISTS': 'ข้อมูลนี้มีอยู่แล้วในระบบ',
  
  // Validation
  'VALIDATION_ERROR': 'ข้อมูลที่กรอกไม่ถูกต้อง กรุณาตรวจสอบและลองใหม่',
  
  // Rate limiting
  'TOO_MANY_REQUESTS': 'มีคำขอมากเกินไป กรุณารอสักครู่แล้วลองใหม่',
  
  // Default
  'INTERNAL_SERVER_ERROR': 'เกิดข้อผิดพลาดที่ไม่คาดคิด กรุณาลองใหม่ภายหลัง',
  'DEFAULT': 'เกิดข้อผิดพลาด กรุณาลองใหม่อีกครั้ง'
};

function getUserFriendlyMessage(error) {
  if (error.code && UserFriendlyMessages[error.code]) {
    return UserFriendlyMessages[error.code];
  }
  
  if (error.statusCode) {
    const statusMessages = {
      400: 'คำขอไม่ถูกต้อง กรุณาตรวจสอบข้อมูล',
      401: 'กรุณาเข้าสู่ระบบก่อน',
      403: 'ไม่มีสิทธิ์เข้าถึง',
      404: 'ไม่พบข้อมูลที่ต้องการ',
      429: 'คำขอมากเกินไป กรุณารอสักครู่',
      500: 'เกิดข้อผิดพลาดภายในเซิร์ฟเวอร์',
      502: 'เซิร์ฟเวอร์ไม่พร้อมใช้งาน',
      503: 'บริการไม่พร้อมใช้งาน กรุณาลองใหม่ภายหลัง'
    };
    
    return statusMessages[error.statusCode] || UserFriendlyMessages.DEFAULT;
  }
  
  return UserFriendlyMessages.DEFAULT;
}

// Toast notification helper
function showErrorNotification(error) {
  const message = getUserFriendlyMessage(error);
  
  // ในการใช้งานจริง
  // toast.error(message);
  
  console.log('Error notification:', message);
  return message;
}

// ตัวอย่างการใช้งาน
const errors = [
  Object.assign(new Error(), { code: 'UNAUTHORIZED', statusCode: 401 }),
  Object.assign(new Error(), { code: 'NOT_FOUND', statusCode: 404 }),
  Object.assign(new Error(), { statusCode: 503 }),
  new Error('Unknown error')
];

errors.forEach(err => {
  console.log(getUserFriendlyMessage(err));
});
```

---

## Step 803: Error Boundaries Concept

```javascript
// Error Boundary concept (สำหรับ vanilla JS)
class ErrorBoundary {
  constructor(options = {}) {
    this.fallback = options.fallback || this.#defaultFallback;
    this.onError = options.onError || console.error;
    this.errors = [];
  }
  
  #defaultFallback(error, context) {
    return {
      type: 'error',
      message: getUserFriendlyMessage(error),
      retry: context.retry
    };
  }
  
  // Wrap synchronous operations
  wrap(fn, context = {}) {
    return (...args) => {
      try {
        return fn(...args);
      } catch (error) {
        this.errors.push(error);
        this.onError(error, context);
        return this.fallback(error, context);
      }
    };
  }
  
  // Wrap async operations
  wrapAsync(fn, context = {}) {
    return async (...args) => {
      try {
        return await fn(...args);
      } catch (error) {
        this.errors.push(error);
        this.onError(error, context);
        return this.fallback(error, context);
      }
    };
  }
  
  // Boundary สำหรับ component
  createComponentBoundary(component, fallbackComponent) {
    return {
      render: this.wrapAsync(
        async (props) => component(props),
        { 
          component: component.name,
          retry: async (props) => component(props)
        }
      )
    };
  }
}

// ใช้งาน
const boundary = new ErrorBoundary({
  fallback: (error, context) => {
    console.log('Fallback triggered:', error.message);
    return { error: true, message: getUserFriendlyMessage(error) };
  },
  onError: (error, context) => {
    console.error('Error in boundary:', error.message, context);
  }
});

// Wrap function
const safeGetUser = boundary.wrapAsync(
  async (id) => {
    if (id < 0) throw new Error('Invalid ID');
    return { id, name: 'User ' + id };
  },
  { operation: 'getUser' }
);

// ใช้งาน
safeGetUser(1).then(result => console.log(result));  // { id: 1, name: 'User 1' }
safeGetUser(-1).then(result => console.log(result)); // Fallback result
```

---

## Step 804: Retry Pattern with Exponential Backoff

```javascript
// Retry configuration
const defaultRetryConfig = {
  maxRetries: 3,
  baseDelay: 1000,    // 1 second
  maxDelay: 30000,    // 30 seconds
  factor: 2,          // exponential factor
  jitter: true,       // เพิ่ม randomness เพื่อป้องกัน thundering herd
  retryOn: (error) => {
    // Retry เฉพาะ transient errors
    if (error.statusCode >= 500) return true;
    if (error.code === 'NETWORK_ERROR') return true;
    if (error.code === 'TIMEOUT') return true;
    return false;
  }
};

// คำนวณ delay
function calculateDelay(attempt, config) {
  const { baseDelay, maxDelay, factor, jitter } = config;
  
  // Exponential backoff: baseDelay * factor^attempt
  let delay = baseDelay * Math.pow(factor, attempt);
  
  // Cap at maxDelay
  delay = Math.min(delay, maxDelay);
  
  // เพิ่ม jitter (±20%)
  if (jitter) {
    const jitterAmount = delay * 0.2;
    delay = delay - jitterAmount + (Math.random() * jitterAmount * 2);
  }
  
  return Math.floor(delay);
}

// Retry function
async function withRetry(fn, config = {}) {
  const retryConfig = { ...defaultRetryConfig, ...config };
  let lastError;
  
  for (let attempt = 0; attempt <= retryConfig.maxRetries; attempt++) {
    try {
      const result = await fn();
      
      if (attempt > 0) {
        console.log(`สำเร็จหลังจากลองครั้งที่ ${attempt + 1}`);
      }
      
      return result;
      
    } catch (error) {
      lastError = error;
      
      // ตรวจสอบว่าควร retry หรือไม่
      if (!retryConfig.retryOn(error) || attempt === retryConfig.maxRetries) {
        throw error;
      }
      
      const delay = calculateDelay(attempt, retryConfig);
      console.log(`ครั้งที่ ${attempt + 1} ล้มเหลว: ${error.message}`);
      console.log(`รอ ${delay}ms ก่อนลองใหม่...`);
      
      await new Promise(resolve => setTimeout(resolve, delay));
    }
  }
  
  throw lastError;
}

// ตัวอย่าง: API call with retry
let callCount = 0;

async function unstableApiCall() {
  callCount++;
  console.log(`API call attempt #${callCount}`);
  
  // Simulate failure for first 2 attempts
  if (callCount < 3) {
    const err = new Error('Server temporarily unavailable');
    err.statusCode = 503;
    throw err;
  }
  
  return { data: 'success', attempt: callCount };
}

// Test retry
async function testRetry() {
  callCount = 0; // reset
  
  try {
    const result = await withRetry(unstableApiCall, {
      maxRetries: 3,
      baseDelay: 100, // Short delay for testing
    });
    console.log('Result:', result);
  } catch (error) {
    console.error('All retries failed:', error.message);
  }
}

// testRetry();

// Retry decorator
function retryable(config = {}) {
  return function(target, key, descriptor) {
    const originalMethod = descriptor.value;
    
    descriptor.value = async function(...args) {
      return withRetry(
        () => originalMethod.apply(this, args),
        config
      );
    };
    
    return descriptor;
  };
}
```

---

## Step 805: Circuit Breaker Pattern

```javascript
// Circuit Breaker states
const CircuitState = {
  CLOSED: 'CLOSED',   // ปกติ - request ผ่านได้
  OPEN: 'OPEN',       // เปิด - block request ทั้งหมด
  HALF_OPEN: 'HALF_OPEN' // กึ่งเปิด - ทดสอบว่า service กลับมาหรือยัง
};

class CircuitBreaker {
  constructor(options = {}) {
    this.name = options.name || 'CircuitBreaker';
    this.threshold = options.threshold || 5;       // จำนวน failures ก่อน open
    this.resetTimeout = options.resetTimeout || 60000; // 60 seconds
    this.halfOpenRequests = options.halfOpenRequests || 1; // requests ใน half-open
    
    this.state = CircuitState.CLOSED;
    this.failureCount = 0;
    this.successCount = 0;
    this.lastFailureTime = null;
    this.halfOpenAttempts = 0;
    
    this.onStateChange = options.onStateChange || ((from, to) => {
      console.log(`Circuit Breaker [${this.name}]: ${from} -> ${to}`);
    });
  }
  
  #setState(newState) {
    const oldState = this.state;
    this.state = newState;
    
    if (oldState !== newState) {
      this.onStateChange(oldState, newState);
    }
  }
  
  #canAttempt() {
    switch (this.state) {
      case CircuitState.CLOSED:
        return true;
        
      case CircuitState.OPEN:
        // ตรวจสอบว่าถึงเวลา reset หรือยัง
        const timeSinceFailure = Date.now() - this.lastFailureTime;
        if (timeSinceFailure >= this.resetTimeout) {
          this.#setState(CircuitState.HALF_OPEN);
          this.halfOpenAttempts = 0;
          return true;
        }
        return false;
        
      case CircuitState.HALF_OPEN:
        return this.halfOpenAttempts < this.halfOpenRequests;
        
      default:
        return false;
    }
  }
  
  #onSuccess() {
    this.failureCount = 0;
    
    if (this.state === CircuitState.HALF_OPEN) {
      this.successCount++;
      if (this.successCount >= this.halfOpenRequests) {
        this.successCount = 0;
        this.#setState(CircuitState.CLOSED);
      }
    }
  }
  
  #onFailure(error) {
    this.failureCount++;
    this.lastFailureTime = Date.now();
    
    if (this.state === CircuitState.HALF_OPEN) {
      // กลับไป OPEN ทันที
      this.#setState(CircuitState.OPEN);
    } else if (this.failureCount >= this.threshold) {
      this.#setState(CircuitState.OPEN);
    }
  }
  
  async execute(fn) {
    if (!this.#canAttempt()) {
      const err = new Error(
        `Circuit Breaker [${this.name}] เปิดอยู่ - service ไม่พร้อมใช้งาน`
      );
      err.code = 'CIRCUIT_OPEN';
      throw err;
    }
    
    if (this.state === CircuitState.HALF_OPEN) {
      this.halfOpenAttempts++;
    }
    
    try {
      const result = await fn();
      this.#onSuccess();
      return result;
    } catch (error) {
      this.#onFailure(error);
      throw error;
    }
  }
  
  getStatus() {
    return {
      name: this.name,
      state: this.state,
      failureCount: this.failureCount,
      lastFailureTime: this.lastFailureTime,
      nextAttemptTime: this.state === CircuitState.OPEN 
        ? new Date(this.lastFailureTime + this.resetTimeout) 
        : null
    };
  }
  
  reset() {
    this.state = CircuitState.CLOSED;
    this.failureCount = 0;
    this.successCount = 0;
    this.lastFailureTime = null;
    this.halfOpenAttempts = 0;
  }
}

// ใช้งาน Circuit Breaker
const breaker = new CircuitBreaker({
  name: 'PaymentService',
  threshold: 3,
  resetTimeout: 5000, // 5 seconds for testing
  onStateChange: (from, to) => {
    console.log(`Payment circuit: ${from} → ${to}`);
  }
});

let paymentCallCount = 0;

async function processPayment(amount) {
  paymentCallCount++;
  
  // Simulate intermittent failures
  if (paymentCallCount <= 3) {
    throw new Error('Payment service unavailable');
  }
  
  return { success: true, transactionId: 'TXN' + Date.now() };
}

// ทดสอบ
async function testCircuitBreaker() {
  for (let i = 0; i < 8; i++) {
    try {
      const result = await breaker.execute(() => processPayment(100));
      console.log(`Payment ${i + 1}:`, result);
    } catch (error) {
      console.log(`Payment ${i + 1} failed:`, error.message);
    }
    console.log('Status:', breaker.getStatus().state);
    await new Promise(r => setTimeout(r, 100));
  }
}
```

---

## Step 806: Graceful Degradation

```javascript
// Graceful Degradation - ลดระดับการทำงานเมื่อเกิด error

class GracefulService {
  constructor(options = {}) {
    this.primaryService = options.primary;
    this.fallbackService = options.fallback;
    this.cache = options.cache || new Map();
    this.cacheTTL = options.cacheTTL || 5 * 60 * 1000; // 5 minutes
  }
  
  async #getCached(key) {
    const cached = this.cache.get(key);
    if (cached && Date.now() - cached.timestamp < this.cacheTTL) {
      return { data: cached.data, fromCache: true };
    }
    return null;
  }
  
  async #setCache(key, data) {
    this.cache.set(key, {
      data,
      timestamp: Date.now()
    });
  }
  
  async getData(key, options = {}) {
    // 1. ลองดึงจาก primary service
    try {
      const data = await this.primaryService.get(key);
      await this.#setCache(key, data); // อัพเดต cache
      return { data, source: 'primary' };
    } catch (primaryError) {
      console.warn(`Primary service failed: ${primaryError.message}`);
      
      // 2. ลองดึงจาก fallback service
      if (this.fallbackService) {
        try {
          const data = await this.fallbackService.get(key);
          return { data, source: 'fallback', degraded: true };
        } catch (fallbackError) {
          console.warn(`Fallback service failed: ${fallbackError.message}`);
        }
      }
      
      // 3. ลองดึงจาก cache
      const cached = await this.#getCached(key);
      if (cached) {
        return { ...cached, source: 'cache', degraded: true, stale: true };
      }
      
      // 4. Return default value ถ้ามี
      if (options.defaultValue !== undefined) {
        return { data: options.defaultValue, source: 'default', degraded: true };
      }
      
      // 5. Throw error ถ้าทำอะไรไม่ได้เลย
      throw new Error(`ไม่สามารถดึงข้อมูล '${key}' ได้จากทุกแหล่ง`);
    }
  }
}

// ตัวอย่าง Recommendation system กับ graceful degradation
class RecommendationService {
  async getPersonalized(userId) {
    // Primary: ML-based recommendations
    return ['product_1', 'product_2', 'product_3'];
  }
}

class FallbackRecommendationService {
  async get(key) {
    // Fallback: rule-based or popular items
    return ['popular_1', 'popular_2', 'popular_3'];
  }
}

// Feature flags สำหรับ graceful degradation
class FeatureFlags {
  constructor(flags = {}) {
    this.flags = flags;
    this.defaults = {};
  }
  
  isEnabled(feature) {
    return this.flags[feature] ?? this.defaults[feature] ?? false;
  }
  
  withFallback(feature, primaryFn, fallbackFn) {
    if (this.isEnabled(feature)) {
      try {
        return primaryFn();
      } catch (error) {
        console.warn(`Feature ${feature} failed, using fallback`);
        return fallbackFn();
      }
    }
    return fallbackFn();
  }
}

const features = new FeatureFlags({
  'new-payment-flow': false,
  'ai-recommendations': true,
  'real-time-updates': true
});

// ใช้งาน
const renderPayment = features.withFallback(
  'new-payment-flow',
  () => console.log('Using new payment UI'),
  () => console.log('Using classic payment UI')
);
```

---

## Step 807-810: แบบฝึกหัดและตัวอย่างครบวงจร

### ตัวอย่าง: Error Handling System สมบูรณ์

```javascript
// ระบบ Error Handling ครบวงจรสำหรับ API Client

class ApiError extends Error {
  constructor(message, statusCode, data = null) {
    super(message);
    this.name = 'ApiError';
    this.statusCode = statusCode;
    this.data = data;
  }
}

class ApiClient {
  constructor(baseUrl, options = {}) {
    this.baseUrl = baseUrl;
    this.timeout = options.timeout || 10000;
    this.retries = options.retries || 3;
    this.circuitBreaker = new CircuitBreaker({
      name: 'API',
      threshold: 5,
      resetTimeout: 30000
    });
  }
  
  async #makeRequest(path, options = {}) {
    const url = `${this.baseUrl}${path}`;
    const controller = new AbortController();
    const timeoutId = setTimeout(() => controller.abort(), this.timeout);
    
    try {
      const response = await fetch(url, {
        ...options,
        signal: controller.signal,
        headers: {
          'Content-Type': 'application/json',
          ...options.headers
        }
      });
      
      clearTimeout(timeoutId);
      
      if (!response.ok) {
        const errorData = await response.json().catch(() => null);
        throw new ApiError(
          errorData?.message || `HTTP ${response.status}`,
          response.status,
          errorData
        );
      }
      
      return await response.json();
      
    } catch (error) {
      clearTimeout(timeoutId);
      
      if (error.name === 'AbortError') {
        const timeoutError = new ApiError('Request timed out', 408);
        timeoutError.code = 'TIMEOUT';
        throw timeoutError;
      }
      
      if (error instanceof TypeError && error.message.includes('fetch')) {
        const networkError = new ApiError('Network error', 0);
        networkError.code = 'NETWORK_ERROR';
        throw networkError;
      }
      
      throw error;
    }
  }
  
  async get(path, options = {}) {
    return this.circuitBreaker.execute(() =>
      withRetry(
        () => this.#makeRequest(path, { method: 'GET', ...options }),
        {
          maxRetries: this.retries,
          retryOn: (error) => {
            return error.code === 'TIMEOUT' || 
                   error.code === 'NETWORK_ERROR' ||
                   (error.statusCode >= 500);
          }
        }
      )
    );
  }
  
  async post(path, body, options = {}) {
    return this.circuitBreaker.execute(() =>
      this.#makeRequest(path, {
        method: 'POST',
        body: JSON.stringify(body),
        ...options
      })
    );
  }
}

// ใช้งาน
const api = new ApiClient('https://api.example.com');

async function fetchUserWithErrorHandling(userId) {
  try {
    const user = await api.get(`/users/${userId}`);
    return { success: true, user };
  } catch (error) {
    if (error instanceof ApiError) {
      switch (error.statusCode) {
        case 401:
          return { success: false, action: 'redirect_to_login' };
        case 403:
          return { success: false, action: 'show_forbidden_message' };
        case 404:
          return { success: false, action: 'show_not_found' };
        case 429:
          return { success: false, action: 'show_rate_limit_message' };
        default:
          return { success: false, action: 'show_error_message', message: error.message };
      }
    }
    
    if (error.code === 'CIRCUIT_OPEN') {
      return { success: false, action: 'show_maintenance_message' };
    }
    
    console.error('Unexpected error:', error);
    return { success: false, action: 'show_generic_error' };
  }
}
```

---

## แบบฝึกหัด (Exercises)

### แบบฝึกหัดที่ 1: สร้าง Error Hierarchy

สร้าง error hierarchy สำหรับระบบ e-commerce:

```javascript
// TODO: สร้าง classes เหล่านี้
// - OrderError (base)
//   - OrderNotFoundError
//   - OrderAlreadyCancelledError
//   - InsufficientStockError
//   - PaymentFailedError
//     - CardDeclinedError
//     - InsufficientFundsError

// แต่ละ class ควรมี:
// - statusCode ที่เหมาะสม
// - code string ที่ unique
// - toJSON() method
// - getUserMessage() method ที่ return ข้อความภาษาไทย
```

### แบบฝึกหัดที่ 2: Retry with Conditions

```javascript
// TODO: ปรับปรุง withRetry ให้รองรับ:
// 1. onRetry callback ที่รับ (error, attempt, delay) parameters
// 2. maxTotalTime - หยุดลอง ถ้าเวลารวมเกิน
// 3. retryOn สามารถเป็น async function ได้
// 4. ถ้า error เป็น ValidationError ไม่ต้อง retry

async function withRetryAdvanced(fn, config) {
  // TODO: implement here
}
```

### แบบฝึกหัดที่ 3: Circuit Breaker Dashboard

```javascript
// TODO: สร้าง CircuitBreakerMonitor class ที่:
// 1. Track ทุก CircuitBreaker instances
// 2. มี getStats() method ที่ return statistics ของทุก breaker
// 3. มี getHealth() method ที่ return overall health
// 4. มี reset(name) method ที่ reset specific breaker

class CircuitBreakerMonitor {
  // TODO: implement
}
```

### แบบฝึกหัดที่ 4: Error Aggregator

```javascript
// TODO: สร้าง ErrorAggregator ที่:
// 1. รวม validation errors จากหลาย sources
// 2. Group errors โดย field หรือ category
// 3. สามารถ merge errors จาก multiple validators ได้
// 4. Generate human-readable summary

class ErrorAggregator {
  // TODO: implement
}
```

### แบบฝึกหัดที่ 5: Error Boundary with Fallback UI

```javascript
// TODO: สร้าง component wrapper ที่:
// 1. Wrap any async function
// 2. แสดง loading state ระหว่างดำเนินการ
// 3. แสดง error state พร้อม retry button ถ้าเกิด error
// 4. Cache ผลลัพธ์สำเร็จ

function createAsyncComponent(asyncFn, options = {}) {
  // TODO: implement
  // options: { loadingComponent, errorComponent, cacheTime }
}
```

---

## สรุป

ในบทนี้เราได้เรียนรู้:

1. **Error Types**: JavaScript มี Error types หลักอยู่ 7 ชนิด แต่ละชนิดมีวัตถุประสงค์ต่างกัน
2. **Custom Errors**: สร้าง Error classes เองช่วยให้โค้ดชัดเจนและ maintainable
3. **Error Hierarchy**: จัดระเบียบ errors ด้วย inheritance และ codes
4. **Error Strategies**: มีหลายวิธีในการ handle errors - try-catch, Result pattern, Error Boundaries
5. **Global Handlers**: ดักจับ errors ที่ไม่ได้ถูก handle ด้วย global handlers
6. **Logging**: บันทึก errors อย่างมีระบบเพื่อ debugging และ monitoring
7. **Retry & Circuit Breaker**: Pattern ที่ช่วยให้ระบบทนทานต่อ transient failures
8. **Graceful Degradation**: ลดระดับการทำงานอย่างสวยงามเมื่อเกิดปัญหา

---

*ต่อไป: Part 42 - Browser APIs*
