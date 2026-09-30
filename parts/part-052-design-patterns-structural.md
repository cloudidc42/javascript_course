# Part 52: Design Patterns - Structural Patterns (Steps 1011-1030)

## การออกแบบ Patterns สำหรับโครงสร้างของวัตถุ

**คำอธิบาย:** Structural Patterns เกี่ยวข้องกับการจัดโครงสร้างของ objects และ classes เพื่อสร้างระบบที่ยืดหยุ่นและง่ายต่อการบำรุงรักษา เราจะเรียนรู้ Adapter, Decorator, Facade, Proxy, Bridge, Composite และ Flyweight patterns

---

## Step 1011: Structural Patterns Overview

Structural Patterns ช่วยในการจัดโครงสร้างของ objects:

| Pattern | หน้าที่ |
|---------|--------|
| Adapter | แปลง interface ให้ใช้งานร่วมกันได้ |
| Decorator | เพิ่มพฤติกรรมให้ object แบบ dynamic |
| Facade | สร้าง interface ที่เรียบง่ายสำหรับระบบซับซ้อน |
| Proxy | ควบคุมการเข้าถึง object |
| Bridge | แยก abstraction จาก implementation |
| Composite | จัดการ tree structure |
| Flyweight | แชร์ state เพื่อประหยัดหน่วยความจำ |

```javascript
// ทำไมต้องใช้ Structural Patterns?

// ปัญหา: เชื่อม API เก่ากับระบบใหม่
class OldPaymentSystem {
  makePayment(amount, currency) {
    return `Old: Paid ${amount} ${currency}`;
  }
}

class NewPaymentInterface {
  pay(paymentData) {
    // ต้องการ { amount, currency, method }
  }
}

// ไม่มี pattern ทำให้ทุกอย่างยุ่งเหยิง
// Adapter Pattern แก้ปัญหานี้ได้!
console.log('Structural Patterns ช่วยจัดโครงสร้างให้เป็นระเบียบ');
```

---

## Step 1012: Adapter Pattern - พื้นฐาน

**Adapter Pattern** ทำให้ interfaces ที่เข้ากันไม่ได้สามารถทำงานร่วมกันได้

```javascript
// ========================
// Class Adapter
// ========================

// Interface เก่า (Legacy)
class LegacyLogger {
  constructor() {
    this.logs = [];
  }
  
  writeLog(message, severity) {
    const entry = `[${severity}] ${new Date().toISOString()}: ${message}`;
    this.logs.push(entry);
    console.log(entry);
  }
  
  getLog(index) {
    return this.logs[index];
  }
  
  getLogsCount() {
    return this.logs.length;
  }
}

// Interface ใหม่ที่ต้องการ
// นี่คือ interface ที่ระบบใหม่คาดหวัง
class ModernLogger {
  debug(message) {}
  info(message) {}
  warn(message) {}
  error(message, error) {}
}

// Adapter: ปรับ LegacyLogger ให้เป็น ModernLogger
class LoggerAdapter extends ModernLogger {
  constructor() {
    super();
    this._legacyLogger = new LegacyLogger();
  }
  
  debug(message) {
    this._legacyLogger.writeLog(message, 'DEBUG');
  }
  
  info(message) {
    this._legacyLogger.writeLog(message, 'INFO');
  }
  
  warn(message) {
    this._legacyLogger.writeLog(message, 'WARN');
  }
  
  error(message, error) {
    const fullMessage = error 
      ? `${message}: ${error.message || error}`
      : message;
    this._legacyLogger.writeLog(fullMessage, 'ERROR');
  }
  
  getLegacyLogs() {
    return this._legacyLogger.logs;
  }
  
  getCount() {
    return this._legacyLogger.getLogsCount();
  }
}

// ========================
// Object Adapter
// ========================

// ระบบใหม่ที่เราพัฒนา
class NewApplication {
  constructor(logger) {
    this.logger = logger; // คาดหวัง ModernLogger interface
  }
  
  doWork() {
    this.logger.info('เริ่มทำงาน');
    this.logger.debug('กำลัง process data');
    
    try {
      // จำลองการทำงาน
      this.logger.info('งานเสร็จสิ้น');
    } catch (e) {
      this.logger.error('เกิดข้อผิดพลาด', e);
    }
  }
}

// ใช้งาน
const adapter = new LoggerAdapter();
const app = new NewApplication(adapter);
app.doWork();

console.log('Total logs:', adapter.getCount());
```

---

## Step 1013: Adapter Pattern - Third-party Library Adapter

```javascript
// ========================
// Adapter สำหรับ Third-party Libraries
// ========================

// สมมติเราใช้ library การชำระเงินที่ต่างกัน

// PayPal API (ของจริง มี interface แบบนี้)
class PayPalAPI {
  constructor(clientId, secret) {
    this.clientId = clientId;
    this.secret = secret;
    this.accessToken = null;
  }
  
  async authenticate() {
    // จำลอง OAuth
    this.accessToken = `paypal_token_${Date.now()}`;
    return this.accessToken;
  }
  
  async createOrder(currency, value) {
    if (!this.accessToken) await this.authenticate();
    
    return {
      id: `PAYPAL_${Date.now()}`,
      status: 'CREATED',
      amount: { currency_code: currency, value: value.toFixed(2) }
    };
  }
  
  async captureOrder(orderId) {
    return {
      id: orderId,
      status: 'COMPLETED',
      payer: { email_address: 'buyer@example.com' }
    };
  }
  
  async refundPayment(captureId, amount) {
    return {
      id: `REFUND_${captureId}`,
      status: 'COMPLETED',
      amount: { value: amount.toFixed(2) }
    };
  }
}

// Stripe API (ต่าง interface)
class StripeAPI {
  constructor(secretKey) {
    this.secretKey = secretKey;
  }
  
  async createPaymentIntent(amount, currency, metadata = {}) {
    // amount ใน satang (สตางค์)
    return {
      id: `pi_stripe_${Date.now()}`,
      amount,
      currency,
      status: 'requires_payment_method',
      client_secret: `pi_secret_${Date.now()}`,
      metadata
    };
  }
  
  async confirmPaymentIntent(paymentIntentId, paymentMethod) {
    return {
      id: paymentIntentId,
      status: 'succeeded',
      amount_received: 50000
    };
  }
  
  async createRefund(paymentIntentId, amount) {
    return {
      id: `re_stripe_${Date.now()}`,
      amount,
      status: 'succeeded',
      payment_intent: paymentIntentId
    };
  }
}

// Common interface ที่ระบบของเราต้องการ
class PaymentGateway {
  async pay(amount, currency, options) {
    throw new Error('Not implemented');
  }
  
  async refund(transactionId, amount) {
    throw new Error('Not implemented');
  }
  
  getGatewayName() {
    throw new Error('Not implemented');
  }
}

// PayPal Adapter
class PayPalAdapter extends PaymentGateway {
  constructor(clientId, secret) {
    super();
    this._api = new PayPalAPI(clientId, secret);
    this._orderMap = new Map(); // orderId -> captureId
  }
  
  async pay(amount, currency, options = {}) {
    const order = await this._api.createOrder(currency, amount);
    const capture = await this._api.captureOrder(order.id);
    this._orderMap.set(order.id, capture);
    
    return {
      success: capture.status === 'COMPLETED',
      transactionId: order.id,
      amount,
      currency,
      gateway: this.getGatewayName(),
      rawResponse: capture
    };
  }
  
  async refund(transactionId, amount) {
    const result = await this._api.refundPayment(transactionId, amount);
    
    return {
      success: result.status === 'COMPLETED',
      refundId: result.id,
      amount: parseFloat(result.amount.value),
      gateway: this.getGatewayName()
    };
  }
  
  getGatewayName() {
    return 'PayPal';
  }
}

// Stripe Adapter
class StripeAdapter extends PaymentGateway {
  constructor(secretKey) {
    super();
    this._api = new StripeAPI(secretKey);
  }
  
  async pay(amount, currency, options = {}) {
    // Stripe ใช้หน่วยเล็กที่สุด (satang สำหรับ THB)
    const stripeAmount = Math.round(amount * 100);
    
    const intent = await this._api.createPaymentIntent(
      stripeAmount,
      currency.toLowerCase(),
      options.metadata
    );
    
    const confirmed = await this._api.confirmPaymentIntent(
      intent.id,
      options.paymentMethod || 'pm_card_visa'
    );
    
    return {
      success: confirmed.status === 'succeeded',
      transactionId: confirmed.id,
      amount,
      currency,
      gateway: this.getGatewayName(),
      rawResponse: confirmed
    };
  }
  
  async refund(transactionId, amount) {
    // Stripe refund amount ใน satang
    const stripeAmount = Math.round(amount * 100);
    const result = await this._api.createRefund(transactionId, stripeAmount);
    
    return {
      success: result.status === 'succeeded',
      refundId: result.id,
      amount,
      gateway: this.getGatewayName()
    };
  }
  
  getGatewayName() {
    return 'Stripe';
  }
}

// ใช้งาน - ระบบไม่ต้องรู้ว่าใช้ gateway ไหน
async function processPayment(gateway, amount, currency) {
  console.log(`\nProcessing ${amount} ${currency} via ${gateway.getGatewayName()}`);
  
  const result = await gateway.pay(amount, currency);
  console.log('Payment result:', result.success ? 'SUCCESS' : 'FAILED');
  console.log('Transaction ID:', result.transactionId);
  
  if (result.success) {
    // Simulate refund
    const refundResult = await gateway.refund(result.transactionId, amount * 0.5);
    console.log('Partial refund:', refundResult.success ? 'SUCCESS' : 'FAILED');
  }
  
  return result;
}

const paypalGateway = new PayPalAdapter('client_id', 'secret');
const stripeGateway = new StripeAdapter('sk_test_key');

processPayment(paypalGateway, 500.00, 'THB');
processPayment(stripeGateway, 500.00, 'THB');
```

---

## Step 1014: Decorator Pattern - พื้นฐาน

**Decorator Pattern** เพิ่มพฤติกรรมให้ object แบบ dynamic โดยไม่ต้อง subclass

```javascript
// ========================
// Function Decorators
// ========================

// Decorator เป็น Higher-order function
function withTiming(fn) {
  return function(...args) {
    const start = performance.now ? performance.now() : Date.now();
    const result = fn.apply(this, args);
    const end = performance.now ? performance.now() : Date.now();
    console.log(`${fn.name} took ${(end - start).toFixed(2)}ms`);
    return result;
  };
}

function withLogging(fn) {
  return function(...args) {
    console.log(`Calling ${fn.name} with args:`, args);
    const result = fn.apply(this, args);
    console.log(`${fn.name} returned:`, result);
    return result;
  };
}

function withErrorHandling(fn) {
  return function(...args) {
    try {
      return fn.apply(this, args);
    } catch (error) {
      console.error(`Error in ${fn.name}:`, error.message);
      return null;
    }
  };
}

function withCache(fn) {
  const cache = new Map();
  
  return function(...args) {
    const key = JSON.stringify(args);
    
    if (cache.has(key)) {
      console.log(`Cache HIT for ${fn.name}`);
      return cache.get(key);
    }
    
    console.log(`Cache MISS for ${fn.name}`);
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}

// ใช้งาน decorators
function expensiveCalculation(n) {
  // จำลองการคำนวณที่หนัก
  let result = 0;
  for (let i = 0; i <= n; i++) {
    result += i;
  }
  return result;
}

// Stack decorators
const decoratedCalc = withTiming(
  withCache(
    withLogging(expensiveCalculation)
  )
);

console.log(decoratedCalc(1000));
console.log(decoratedCalc(1000)); // จะ cache hit ครั้งที่ 2
console.log(decoratedCalc(2000));

// ========================
// Decorator ด้วย compose
// ========================

const compose = (...fns) => fn => fns.reduceRight((acc, f) => f(acc), fn);

const withAuth = (fn) => function(...args) {
  const token = args[0]?.token;
  if (!token) throw new Error('Authentication required');
  return fn.apply(this, args);
};

const withValidation = (schema) => (fn) => function(...args) {
  const data = args[0];
  const errors = [];
  
  if (schema.required) {
    schema.required.forEach(field => {
      if (!data[field]) errors.push(`${field} is required`);
    });
  }
  
  if (errors.length > 0) throw new Error(errors.join(', '));
  return fn.apply(this, args);
};

function saveUser(userData) {
  console.log('Saving user:', userData);
  return { id: Date.now(), ...userData };
}

const safeSaveUser = compose(
  withAuth,
  withValidation({ required: ['name', 'email'] }),
  withLogging,
  withTiming
)(saveUser);

// Test
try {
  safesaveUser({ name: 'สมชาย', email: 'test@example.com', token: 'valid-token' });
} catch (e) {
  console.log('Error:', e.message);
}
```

---

## Step 1015: Decorator Pattern - Class Decorator

```javascript
// ========================
// Class-based Decorator
// ========================

// Interface หลัก
class Coffee {
  cost() {
    return 0;
  }
  
  description() {
    return 'Unknown coffee';
  }
}

// Base product
class SimpleCoffee extends Coffee {
  cost() {
    return 30;
  }
  
  description() {
    return 'กาแฟดำ';
  }
}

class Espresso extends Coffee {
  cost() {
    return 45;
  }
  
  description() {
    return 'เอสเปรสโซ่';
  }
}

// Decorator Base
class CoffeeDecorator extends Coffee {
  constructor(coffee) {
    super();
    this._coffee = coffee;
  }
  
  cost() {
    return this._coffee.cost();
  }
  
  description() {
    return this._coffee.description();
  }
}

// Concrete Decorators
class MilkDecorator extends CoffeeDecorator {
  constructor(coffee, amount = 1) {
    super(coffee);
    this._amount = amount;
  }
  
  cost() {
    return this._coffee.cost() + (10 * this._amount);
  }
  
  description() {
    const milkDesc = this._amount > 1 
      ? `นม ${this._amount}x` 
      : 'นม';
    return `${this._coffee.description()}, ${milkDesc}`;
  }
}

class SugarDecorator extends CoffeeDecorator {
  constructor(coffee, amount = 1) {
    super(coffee);
    this._amount = amount;
  }
  
  cost() {
    return this._coffee.cost() + (5 * this._amount);
  }
  
  description() {
    const sugarDesc = this._amount > 1 
      ? `น้ำตาล ${this._amount} ช้อน` 
      : 'น้ำตาล';
    return `${this._coffee.description()}, ${sugarDesc}`;
  }
}

class WhipDecorator extends CoffeeDecorator {
  cost() {
    return this._coffee.cost() + 20;
  }
  
  description() {
    return `${this._coffee.description()}, วิปครีม`;
  }
}

class VanillaDecorator extends CoffeeDecorator {
  cost() {
    return this._coffee.cost() + 15;
  }
  
  description() {
    return `${this._coffee.description()}, วานิลลา`;
  }
}

class ExtraShotDecorator extends CoffeeDecorator {
  constructor(coffee, shots = 1) {
    super(coffee);
    this._shots = shots;
  }
  
  cost() {
    return this._coffee.cost() + (20 * this._shots);
  }
  
  description() {
    return `${this._coffee.description()}, extra shot x${this._shots}`;
  }
}

// ใช้งาน
let myCoffee = new SimpleCoffee();
console.log(`${myCoffee.description()}: ฿${myCoffee.cost()}`);

// เพิ่ม decorators ทีละอัน
myCoffee = new MilkDecorator(myCoffee);
myCoffee = new SugarDecorator(myCoffee, 2);
myCoffee = new WhipDecorator(myCoffee);
myCoffee = new VanillaDecorator(myCoffee);

console.log(`${myCoffee.description()}: ฿${myCoffee.cost()}`);

// Espresso พิเศษ
let specialDrink = new Espresso();
specialDrink = new ExtraShotDecorator(specialDrink, 2);
specialDrink = new MilkDecorator(specialDrink, 2);
specialDrink = new VanillaDecorator(specialDrink);
specialDrink = new WhipDecorator(specialDrink);

console.log(`Special: ${specialDrink.description()}: ฿${specialDrink.cost()}`);

// ========================
// Decorator สำหรับ API Response
// ========================

class ApiResponse {
  constructor(data) {
    this.data = data;
    this.timestamp = new Date().toISOString();
  }
  
  toJSON() {
    return {
      data: this.data,
      timestamp: this.timestamp
    };
  }
}

class ResponseDecorator extends ApiResponse {
  constructor(response) {
    super(response.data);
    this._response = response;
  }
  
  toJSON() {
    return this._response.toJSON();
  }
}

class PaginatedResponse extends ResponseDecorator {
  constructor(response, pagination) {
    super(response);
    this._pagination = pagination;
  }
  
  toJSON() {
    return {
      ...this._response.toJSON(),
      pagination: this._pagination
    };
  }
}

class CachedResponse extends ResponseDecorator {
  constructor(response, cacheKey) {
    super(response);
    this._cacheKey = cacheKey;
    this._cachedAt = new Date().toISOString();
  }
  
  toJSON() {
    return {
      ...this._response.toJSON(),
      cache: {
        key: this._cacheKey,
        cachedAt: this._cachedAt
      }
    };
  }
}

class MetaResponse extends ResponseDecorator {
  constructor(response, meta) {
    super(response);
    this._meta = meta;
  }
  
  toJSON() {
    return {
      ...this._response.toJSON(),
      meta: this._meta
    };
  }
}

// ใช้งาน
const users = [
  { id: 1, name: 'สมชาย' },
  { id: 2, name: 'สมหญิง' }
];

let response = new ApiResponse(users);
response = new PaginatedResponse(response, { page: 1, total: 50, limit: 10 });
response = new CachedResponse(response, 'users:page:1');
response = new MetaResponse(response, { 
  requestId: 'req_123',
  processingTime: '45ms'
});

console.log(JSON.stringify(response.toJSON(), null, 2));
```

---

## Step 1016: Decorator Pattern - Middleware

```javascript
// ========================
// Middleware as Decorator (Express-like)
// ========================

class Request {
  constructor(method, path, body = {}, headers = {}) {
    this.method = method;
    this.path = path;
    this.body = body;
    this.headers = headers;
    this.params = {};
    this.query = {};
    this.user = null;
    this.startTime = Date.now();
  }
}

class Response {
  constructor() {
    this.statusCode = 200;
    this.headers = { 'Content-Type': 'application/json' };
    this.body = null;
    this._sent = false;
  }
  
  status(code) {
    this.statusCode = code;
    return this;
  }
  
  json(data) {
    this.body = data;
    this._sent = true;
    console.log(`Response ${this.statusCode}:`, JSON.stringify(data, null, 2));
    return this;
  }
  
  setHeader(name, value) {
    this.headers[name] = value;
    return this;
  }
}

// Middleware system
class MiddlewareChain {
  constructor() {
    this._middlewares = [];
    this._errorHandlers = [];
  }
  
  use(middleware) {
    this._middlewares.push(middleware);
    return this;
  }
  
  onError(handler) {
    this._errorHandlers.push(handler);
    return this;
  }
  
  async handle(req, res) {
    let index = 0;
    
    const next = async (error) => {
      if (error) {
        // ส่งต่อไปยัง error handlers
        for (const handler of this._errorHandlers) {
          await handler(error, req, res, next);
        }
        return;
      }
      
      if (index < this._middlewares.length) {
        const middleware = this._middlewares[index++];
        try {
          await middleware(req, res, next);
        } catch (err) {
          await next(err);
        }
      }
    };
    
    await next();
  }
}

// Middleware functions (Decorators)
function cors(options = {}) {
  return async (req, res, next) => {
    const origin = options.origin || '*';
    res.setHeader('Access-Control-Allow-Origin', origin);
    res.setHeader('Access-Control-Allow-Methods', 'GET,POST,PUT,DELETE,OPTIONS');
    res.setHeader('Access-Control-Allow-Headers', 'Content-Type, Authorization');
    
    if (req.method === 'OPTIONS') {
      res.status(204).json({});
      return;
    }
    
    await next();
  };
}

function requestLogger() {
  return async (req, res, next) => {
    const timestamp = new Date().toISOString();
    console.log(`[${timestamp}] ${req.method} ${req.path}`);
    
    await next();
    
    const duration = Date.now() - req.startTime;
    console.log(`[${timestamp}] Completed in ${duration}ms`);
  };
}

function authenticate() {
  return async (req, res, next) => {
    const authHeader = req.headers['authorization'];
    
    if (!authHeader) {
      return res.status(401).json({ error: 'Authentication required' });
    }
    
    const token = authHeader.replace('Bearer ', '');
    
    // จำลองการ verify token
    if (token === 'valid-token') {
      req.user = { id: 1, name: 'สมชาย', role: 'admin' };
      await next();
    } else {
      return res.status(401).json({ error: 'Invalid token' });
    }
  };
}

function validate(schema) {
  return async (req, res, next) => {
    const errors = [];
    
    if (schema.body) {
      Object.entries(schema.body).forEach(([field, rules]) => {
        if (rules.required && !req.body[field]) {
          errors.push(`${field} is required`);
        }
        if (rules.type && req.body[field] && typeof req.body[field] !== rules.type) {
          errors.push(`${field} must be ${rules.type}`);
        }
      });
    }
    
    if (errors.length > 0) {
      return res.status(400).json({ errors });
    }
    
    await next();
  };
}

function rateLimit(requests, windowMs) {
  const requestCounts = new Map();
  
  return async (req, res, next) => {
    const ip = req.headers['x-forwarded-for'] || 'unknown';
    const now = Date.now();
    const windowStart = now - windowMs;
    
    if (!requestCounts.has(ip)) {
      requestCounts.set(ip, []);
    }
    
    const counts = requestCounts.get(ip)
      .filter(time => time > windowStart);
    
    if (counts.length >= requests) {
      return res.status(429).json({ error: 'Too many requests' });
    }
    
    counts.push(now);
    requestCounts.set(ip, counts);
    
    await next();
  };
}

// ใช้งาน
const chain = new MiddlewareChain();

chain
  .use(requestLogger())
  .use(cors({ origin: 'https://example.com' }))
  .use(rateLimit(100, 60000))
  .use(authenticate())
  .use(validate({
    body: {
      name: { required: true, type: 'string' },
      email: { required: true, type: 'string' }
    }
  }))
  .use(async (req, res, next) => {
    // Route handler
    res.json({
      message: 'สร้างผู้ใช้สำเร็จ',
      user: { ...req.body, id: Date.now() }
    });
  })
  .onError(async (error, req, res, next) => {
    console.error('Error:', error);
    res.status(500).json({ error: 'Internal server error' });
  });

// ทดสอบ
const req1 = new Request('POST', '/users', 
  { name: 'สมชาย', email: 'somchai@example.com' },
  { authorization: 'Bearer valid-token' }
);
const res1 = new Response();
chain.handle(req1, res1);
```

---

## Step 1017: Facade Pattern

**Facade Pattern** ให้ simplified interface สำหรับ subsystem ซับซ้อน

```javascript
// ========================
// Facade Pattern
// ========================

// Complex Subsystem
class VideoProcessor {
  decode(file) {
    console.log(`Decoding video file: ${file}`);
    return { codec: 'H.264', duration: 120, frames: 3600 };
  }
  
  transcode(data, targetCodec) {
    console.log(`Transcoding to ${targetCodec}...`);
    return { ...data, codec: targetCodec, transcoded: true };
  }
  
  compress(data, quality) {
    console.log(`Compressing with quality ${quality}...`);
    return { ...data, quality, compressed: true, size: '50MB' };
  }
}

class AudioProcessor {
  extract(videoData) {
    console.log('Extracting audio track...');
    return { sampleRate: 44100, channels: 2, format: 'PCM' };
  }
  
  normalize(audioData) {
    console.log('Normalizing audio...');
    return { ...audioData, normalized: true, level: -14 };
  }
  
  encode(audioData, format) {
    console.log(`Encoding audio to ${format}...`);
    return { ...audioData, format, encoded: true };
  }
}

class ThumbnailGenerator {
  extract(file, timeCode) {
    console.log(`Extracting frame at ${timeCode}s...`);
    return { frame: Buffer.from('...'), timestamp: timeCode };
  }
  
  resize(frame, width, height) {
    console.log(`Resizing to ${width}x${height}...`);
    return { ...frame, width, height };
  }
  
  compress(image, quality) {
    console.log(`Compressing image with quality ${quality}...`);
    return { ...image, quality, format: 'JPEG' };
  }
  
  save(image, path) {
    console.log(`Saving thumbnail to ${path}...`);
    return { success: true, path };
  }
}

class SubtitleProcessor {
  detect(videoFile) {
    console.log('Detecting subtitles...');
    return [{ language: 'th', track: 0 }];
  }
  
  extract(videoFile, track) {
    console.log(`Extracting subtitle track ${track}...`);
    return [{ time: 0, text: 'สวัสดี' }, { time: 5, text: 'ยินดีต้อนรับ' }];
  }
  
  embed(videoData, subtitles) {
    console.log('Embedding subtitles...');
    return { ...videoData, subtitles: true };
  }
}

class FileManager {
  checkSpace(size) {
    console.log(`Checking disk space for ${size}...`);
    return true;
  }
  
  createTempDir() {
    const dir = `/tmp/video_${Date.now()}`;
    console.log(`Creating temp dir: ${dir}`);
    return dir;
  }
  
  move(from, to) {
    console.log(`Moving ${from} to ${to}`);
    return true;
  }
  
  cleanup(dir) {
    console.log(`Cleaning up ${dir}`);
    return true;
  }
}

// ===== Facade =====
class VideoConversionFacade {
  constructor() {
    this._videoProcessor = new VideoProcessor();
    this._audioProcessor = new AudioProcessor();
    this._thumbnailGen = new ThumbnailGenerator();
    this._subtitleProcessor = new SubtitleProcessor();
    this._fileManager = new FileManager();
  }
  
  convert(inputFile, options = {}) {
    const {
      targetFormat = 'mp4',
      quality = 'high',
      generateThumbnail = true,
      extractSubtitles = true,
      outputPath = './output'
    } = options;
    
    console.log(`\n=== Starting conversion of ${inputFile} ===`);
    
    // 1. ตรวจสอบ disk space
    this._fileManager.checkSpace('2GB');
    const tempDir = this._fileManager.createTempDir();
    
    // 2. Process video
    let videoData = this._videoProcessor.decode(inputFile);
    videoData = this._videoProcessor.transcode(videoData, targetFormat);
    videoData = this._videoProcessor.compress(videoData, quality);
    
    // 3. Process audio
    let audioData = this._audioProcessor.extract(videoData);
    audioData = this._audioProcessor.normalize(audioData);
    audioData = this._audioProcessor.encode(audioData, 'AAC');
    
    // 4. Generate thumbnail
    let thumbnailResult = null;
    if (generateThumbnail) {
      let thumbnail = this._thumbnailGen.extract(inputFile, 5);
      thumbnail = this._thumbnailGen.resize(thumbnail, 1280, 720);
      thumbnail = this._thumbnailGen.compress(thumbnail, 85);
      thumbnailResult = this._thumbnailGen.save(thumbnail, `${outputPath}/thumbnail.jpg`);
    }
    
    // 5. Handle subtitles
    let subtitleResult = null;
    if (extractSubtitles) {
      const subtitleTracks = this._subtitleProcessor.detect(inputFile);
      if (subtitleTracks.length > 0) {
        const subtitles = this._subtitleProcessor.extract(inputFile, 0);
        videoData = this._subtitleProcessor.embed(videoData, subtitles);
        subtitleResult = subtitleTracks;
      }
    }
    
    // 6. Save output
    const finalPath = `${outputPath}/output.${targetFormat}`;
    this._fileManager.move(tempDir, finalPath);
    
    // 7. Cleanup
    this._fileManager.cleanup(tempDir);
    
    console.log('=== Conversion complete! ===\n');
    
    return {
      success: true,
      outputPath: finalPath,
      thumbnail: thumbnailResult?.path,
      subtitles: subtitleResult,
      format: targetFormat,
      quality
    };
  }
  
  generateThumbnail(videoFile, outputPath, timeCode = 5) {
    let thumbnail = this._thumbnailGen.extract(videoFile, timeCode);
    thumbnail = this._thumbnailGen.resize(thumbnail, 1280, 720);
    thumbnail = this._thumbnailGen.compress(thumbnail, 85);
    return this._thumbnailGen.save(thumbnail, outputPath);
  }
}

// ใช้งาน - ง่ายมาก ไม่ต้องรู้ subsystem
const converter = new VideoConversionFacade();
const result = converter.convert('my-movie.mov', {
  targetFormat: 'mp4',
  quality: 'high',
  generateThumbnail: true,
  extractSubtitles: true,
  outputPath: './public/videos'
});

console.log('Result:', result);
```

---

## Step 1018: Proxy Pattern

**Proxy Pattern** ให้ object ตัวแทนที่ควบคุมการเข้าถึง object จริง

```javascript
// ========================
// Virtual Proxy - Lazy Loading
// ========================

class RealDatabase {
  constructor(connectionString) {
    console.log('Connecting to database...');
    // จำลอง expensive connection
    this._connectionString = connectionString;
    this._connected = true;
    this._data = new Map();
  }
  
  query(sql) {
    if (!this._connected) throw new Error('Not connected');
    console.log(`Executing: ${sql}`);
    return { rows: [], count: 0 };
  }
  
  insert(table, data) {
    if (!this._connected) throw new Error('Not connected');
    const id = Date.now();
    this._data.set(id, { table, ...data });
    return { id, inserted: true };
  }
}

// Virtual Proxy
class DatabaseProxy {
  constructor(connectionString) {
    this._connectionString = connectionString;
    this._realDatabase = null; // ยังไม่สร้าง!
  }
  
  _getDatabase() {
    if (!this._realDatabase) {
      console.log('Creating real database connection (lazy)...');
      this._realDatabase = new RealDatabase(this._connectionString);
    }
    return this._realDatabase;
  }
  
  query(sql) {
    return this._getDatabase().query(sql);
  }
  
  insert(table, data) {
    return this._getDatabase().insert(table, data);
  }
}

console.log('Proxy created - no connection yet');
const db = new DatabaseProxy('postgresql://localhost/mydb');
console.log('Still no connection...');

// connection สร้างเมื่อใช้งานครั้งแรก
db.query('SELECT 1');
db.insert('users', { name: 'สมชาย' });

// ========================
// Protection Proxy
// ========================

class BankAccount {
  constructor(owner, balance) {
    this._owner = owner;
    this._balance = balance;
    this._transactions = [];
  }
  
  deposit(amount) {
    this._balance += amount;
    this._transactions.push({ type: 'deposit', amount, date: new Date() });
    return this._balance;
  }
  
  withdraw(amount) {
    if (amount > this._balance) {
      throw new Error('Insufficient funds');
    }
    this._balance -= amount;
    this._transactions.push({ type: 'withdrawal', amount, date: new Date() });
    return this._balance;
  }
  
  getBalance() {
    return this._balance;
  }
  
  getTransactions() {
    return [...this._transactions];
  }
}

class SecureBankAccountProxy {
  constructor(account, currentUser) {
    this._account = account;
    this._currentUser = currentUser;
    this._log = [];
  }
  
  _isAuthorized(action) {
    // เพิ่มเงินได้ทุกคน
    if (action === 'deposit') return true;
    
    // ถอนได้เฉพาะเจ้าของ
    if (action === 'withdraw') {
      return this._currentUser === this._account._owner;
    }
    
    // ดูยอดได้เจ้าของและ admin
    if (action === 'view') {
      return this._currentUser === this._account._owner 
             || this._currentUser === 'admin';
    }
    
    return false;
  }
  
  _logAccess(action, args, result) {
    this._log.push({
      timestamp: new Date().toISOString(),
      user: this._currentUser,
      action,
      args,
      success: result !== false
    });
  }
  
  deposit(amount) {
    if (!this._isAuthorized('deposit')) {
      this._logAccess('deposit', [amount], false);
      throw new Error('Unauthorized');
    }
    
    if (amount <= 0) throw new Error('Amount must be positive');
    
    const result = this._account.deposit(amount);
    this._logAccess('deposit', [amount], result);
    return result;
  }
  
  withdraw(amount) {
    if (!this._isAuthorized('withdraw')) {
      this._logAccess('withdraw', [amount], false);
      throw new Error(`User ${this._currentUser} is not authorized to withdraw`);
    }
    
    if (amount <= 0) throw new Error('Amount must be positive');
    if (amount > 100000) throw new Error('Single withdrawal limit: 100,000');
    
    const result = this._account.withdraw(amount);
    this._logAccess('withdraw', [amount], result);
    return result;
  }
  
  getBalance() {
    if (!this._isAuthorized('view')) {
      this._logAccess('getBalance', [], false);
      throw new Error('Unauthorized to view balance');
    }
    
    const result = this._account.getBalance();
    this._logAccess('getBalance', [], result);
    return result;
  }
  
  getAuditLog() {
    return [...this._log];
  }
}

// ใช้งาน
const account = new BankAccount('สมชาย', 100000);
const ownerProxy = new SecureBankAccountProxy(account, 'สมชาย');
const hackerProxy = new SecureBankAccountProxy(account, 'hacker');

ownerProxy.deposit(50000);
console.log('Balance:', ownerProxy.getBalance());

try {
  hackerProxy.withdraw(10000); // จะ throw
} catch (e) {
  console.error(e.message);
}

try {
  hackerProxy.getBalance(); // จะ throw
} catch (e) {
  console.error(e.message);
}

console.log('Audit Log:', ownerProxy.getAuditLog());

// ========================
// Caching Proxy
// ========================

class WeatherService {
  async getWeather(city) {
    console.log(`Fetching real weather data for ${city}...`);
    // จำลอง API call
    await new Promise(resolve => setTimeout(resolve, 500));
    return {
      city,
      temperature: 28 + Math.random() * 5,
      humidity: 70 + Math.random() * 20,
      condition: 'Sunny',
      fetchedAt: new Date().toISOString()
    };
  }
}

class CachingWeatherProxy {
  constructor(service, cacheTTL = 300000) { // 5 minutes
    this._service = service;
    this._cache = new Map();
    this._cacheTTL = cacheTTL;
    this._stats = { hits: 0, misses: 0 };
  }
  
  async getWeather(city) {
    const cacheKey = city.toLowerCase();
    const cached = this._cache.get(cacheKey);
    
    if (cached && Date.now() - cached.timestamp < this._cacheTTL) {
      this._stats.hits++;
      console.log(`Cache HIT for ${city}`);
      return { ...cached.data, fromCache: true };
    }
    
    this._stats.misses++;
    console.log(`Cache MISS for ${city}`);
    const data = await this._service.getWeather(city);
    
    this._cache.set(cacheKey, {
      data,
      timestamp: Date.now()
    });
    
    return data;
  }
  
  getCacheStats() {
    const total = this._stats.hits + this._stats.misses;
    return {
      ...this._stats,
      total,
      hitRate: total > 0 ? `${(this._stats.hits / total * 100).toFixed(1)}%` : '0%',
      cachedCities: [...this._cache.keys()]
    };
  }
  
  invalidate(city) {
    this._cache.delete(city.toLowerCase());
  }
  
  clearCache() {
    this._cache.clear();
  }
}

// ใช้งาน
async function weatherDemo() {
  const weatherService = new WeatherService();
  const cachedWeather = new CachingWeatherProxy(weatherService);
  
  // ครั้งแรก - miss
  const bkk1 = await cachedWeather.getWeather('Bangkok');
  console.log('Bangkok weather:', bkk1.temperature.toFixed(1), '°C');
  
  // ครั้งที่สอง - hit
  const bkk2 = await cachedWeather.getWeather('Bangkok');
  console.log('Bangkok (cached):', bkk2.fromCache);
  
  const cm = await cachedWeather.getWeather('Chiang Mai');
  console.log('Chiang Mai:', cm.temperature.toFixed(1), '°C');
  
  console.log('Cache stats:', cachedWeather.getCacheStats());
}

weatherDemo();
```

---

## Step 1019: Proxy Pattern ด้วย JavaScript Proxy

```javascript
// ========================
// JavaScript Proxy Object
// ========================

// JavaScript มี built-in Proxy ที่ทรงพลังมาก

// 1. Validation Proxy
function createValidatedObject(target, validators) {
  return new Proxy(target, {
    set(obj, prop, value) {
      if (validators[prop]) {
        const error = validators[prop](value);
        if (error) {
          throw new TypeError(`Validation failed for ${prop}: ${error}`);
        }
      }
      obj[prop] = value;
      return true;
    },
    
    get(obj, prop) {
      if (prop in obj) {
        return obj[prop];
      }
      return undefined;
    }
  });
}

const user = createValidatedObject({}, {
  age: (value) => {
    if (typeof value !== 'number') return 'Age must be a number';
    if (value < 0 || value > 150) return 'Age must be between 0 and 150';
    return null;
  },
  email: (value) => {
    if (typeof value !== 'string') return 'Email must be a string';
    if (!value.includes('@')) return 'Invalid email format';
    return null;
  },
  name: (value) => {
    if (typeof value !== 'string') return 'Name must be a string';
    if (value.length < 2) return 'Name must be at least 2 characters';
    return null;
  }
});

user.name = 'สมชาย ใจดี';
user.email = 'somchai@example.com';
user.age = 25;
console.log(user);

try {
  user.age = -5; // จะ throw
} catch (e) {
  console.error(e.message);
}

try {
  user.email = 'invalid-email'; // จะ throw
} catch (e) {
  console.error(e.message);
}

// 2. Observable Proxy (Reactive)
function createObservable(target, onChange) {
  return new Proxy(target, {
    set(obj, prop, value) {
      const oldValue = obj[prop];
      obj[prop] = value;
      onChange(prop, oldValue, value);
      return true;
    },
    
    deleteProperty(obj, prop) {
      const oldValue = obj[prop];
      delete obj[prop];
      onChange(prop, oldValue, undefined);
      return true;
    }
  });
}

const state = createObservable(
  { count: 0, name: 'test', items: [] },
  (prop, oldValue, newValue) => {
    console.log(`State changed: ${prop} = ${JSON.stringify(oldValue)} → ${JSON.stringify(newValue)}`);
  }
);

state.count = 1;
state.count = 2;
state.name = 'ทดสอบ';

// 3. Logging Proxy
function createLoggingProxy(target, name = 'object') {
  return new Proxy(target, {
    get(obj, prop) {
      const value = obj[prop];
      if (typeof value === 'function') {
        return function(...args) {
          console.log(`[${name}] Calling ${prop}(${args.map(a => JSON.stringify(a)).join(', ')})`);
          const result = value.apply(obj, args);
          console.log(`[${name}] ${prop} returned:`, result);
          return result;
        };
      }
      console.log(`[${name}] Getting ${prop}: ${JSON.stringify(value)}`);
      return value;
    },
    
    set(obj, prop, value) {
      console.log(`[${name}] Setting ${prop} = ${JSON.stringify(value)}`);
      obj[prop] = value;
      return true;
    }
  });
}

const calculator = createLoggingProxy({
  add: (a, b) => a + b,
  subtract: (a, b) => a - b,
  multiply: (a, b) => a * b
}, 'Calculator');

calculator.add(5, 3);
calculator.multiply(4, 7);

// 4. Readonly Proxy
function createReadOnly(target) {
  return new Proxy(target, {
    set(obj, prop, value) {
      throw new TypeError(`Cannot set property '${prop}' - object is read-only`);
    },
    
    deleteProperty(obj, prop) {
      throw new TypeError(`Cannot delete property '${prop}' - object is read-only`);
    }
  });
}

const config = createReadOnly({
  apiUrl: 'https://api.example.com',
  version: '2.0',
  features: { auth: true }
});

console.log(config.apiUrl);

try {
  config.apiUrl = 'hacked!';
} catch (e) {
  console.error(e.message);
}
```

---

## Step 1020: Bridge Pattern

**Bridge Pattern** แยก abstraction ออกจาก implementation ทำให้ทั้งสองส่วนเปลี่ยนแปลงได้อิสระ

```javascript
// ========================
// Bridge Pattern
// ========================

// Implementation hierarchy
class Renderer {
  renderCircle(x, y, radius) { throw new Error('Not implemented'); }
  renderRectangle(x, y, w, h) { throw new Error('Not implemented'); }
  renderText(x, y, text) { throw new Error('Not implemented'); }
}

class SVGRenderer extends Renderer {
  renderCircle(x, y, radius) {
    return `<circle cx="${x}" cy="${y}" r="${radius}" fill="blue" />`;
  }
  
  renderRectangle(x, y, w, h) {
    return `<rect x="${x}" y="${y}" width="${w}" height="${h}" fill="red" />`;
  }
  
  renderText(x, y, text) {
    return `<text x="${x}" y="${y}" font-size="16">${text}</text>`;
  }
}

class CanvasRenderer extends Renderer {
  renderCircle(x, y, radius) {
    return `ctx.arc(${x}, ${y}, ${radius}, 0, Math.PI * 2); ctx.fill();`;
  }
  
  renderRectangle(x, y, w, h) {
    return `ctx.fillRect(${x}, ${y}, ${w}, ${h});`;
  }
  
  renderText(x, y, text) {
    return `ctx.fillText("${text}", ${x}, ${y});`;
  }
}

class ConsoleRenderer extends Renderer {
  renderCircle(x, y, radius) {
    return `[Circle at (${x},${y}) radius=${radius}]`;
  }
  
  renderRectangle(x, y, w, h) {
    return `[Rectangle at (${x},${y}) size=${w}x${h}]`;
  }
  
  renderText(x, y, text) {
    return `[Text at (${x},${y}): "${text}"]`;
  }
}

// Abstraction hierarchy
class Shape {
  constructor(renderer) {
    this.renderer = renderer;
  }
  
  draw() { throw new Error('Not implemented'); }
  
  move(x, y) {
    this.x += x;
    this.y += y;
    return this;
  }
}

class Circle extends Shape {
  constructor(renderer, x, y, radius) {
    super(renderer);
    this.x = x;
    this.y = y;
    this.radius = radius;
  }
  
  draw() {
    return this.renderer.renderCircle(this.x, this.y, this.radius);
  }
  
  scale(factor) {
    this.radius *= factor;
    return this;
  }
}

class Rectangle extends Shape {
  constructor(renderer, x, y, width, height) {
    super(renderer);
    this.x = x;
    this.y = y;
    this.width = width;
    this.height = height;
  }
  
  draw() {
    return this.renderer.renderRectangle(this.x, this.y, this.width, this.height);
  }
}

class TextShape extends Shape {
  constructor(renderer, x, y, text) {
    super(renderer);
    this.x = x;
    this.y = y;
    this.text = text;
  }
  
  draw() {
    return this.renderer.renderText(this.x, this.y, this.text);
  }
}

// ใช้งาน - เปลี่ยน renderer ได้โดยไม่ต้องแก้ shape
const svgRenderer = new SVGRenderer();
const canvasRenderer = new CanvasRenderer();
const consoleRenderer = new ConsoleRenderer();

// ใช้ renderer ต่างกัน
const circle1 = new Circle(svgRenderer, 100, 100, 50);
const circle2 = new Circle(canvasRenderer, 200, 200, 75);
const circle3 = new Circle(consoleRenderer, 300, 300, 25);

console.log('SVG Circle:', circle1.draw());
console.log('Canvas Circle:', circle2.draw());
console.log('Console Circle:', circle3.draw());

const rect = new Rectangle(svgRenderer, 10, 10, 200, 100);
console.log('SVG Rect:', rect.draw());

// เปลี่ยน renderer ได้ตอน runtime
circle1.renderer = canvasRenderer;
console.log('Circle1 now with Canvas:', circle1.draw());
```

---

## Step 1021: Composite Pattern

**Composite Pattern** จัดการ tree structure โดยทำให้ leaf และ composite ทำงานด้วยกันได้

```javascript
// ========================
// Composite Pattern - File System
// ========================

class FileSystemItem {
  constructor(name) {
    this.name = name;
    this.parent = null;
  }
  
  getSize() { throw new Error('Not implemented'); }
  getPath() {
    if (this.parent) {
      return `${this.parent.getPath()}/${this.name}`;
    }
    return `/${this.name}`;
  }
  display(indent = '') { throw new Error('Not implemented'); }
}

// Leaf
class File extends FileSystemItem {
  constructor(name, size, content = '') {
    super(name);
    this._size = size;
    this.content = content;
    this.extension = name.split('.').pop();
    this.createdAt = new Date();
  }
  
  getSize() {
    return this._size;
  }
  
  display(indent = '') {
    console.log(`${indent}📄 ${this.name} (${this._size} bytes)`);
  }
  
  read() {
    return this.content;
  }
}

// Composite
class Directory extends FileSystemItem {
  constructor(name) {
    super(name);
    this._children = [];
  }
  
  add(item) {
    item.parent = this;
    this._children.push(item);
    return this;
  }
  
  remove(item) {
    const index = this._children.indexOf(item);
    if (index !== -1) {
      this._children[index].parent = null;
      this._children.splice(index, 1);
    }
    return this;
  }
  
  getChild(name) {
    return this._children.find(child => child.name === name);
  }
  
  getSize() {
    return this._children.reduce((total, child) => total + child.getSize(), 0);
  }
  
  getChildren() {
    return [...this._children];
  }
  
  display(indent = '') {
    console.log(`${indent}📁 ${this.name}/ (${this.getSize()} bytes)`);
    this._children.forEach(child => child.display(indent + '  '));
  }
  
  find(predicate) {
    const results = [];
    
    for (const child of this._children) {
      if (predicate(child)) {
        results.push(child);
      }
      
      if (child instanceof Directory) {
        results.push(...child.find(predicate));
      }
    }
    
    return results;
  }
  
  getFileCount() {
    let count = 0;
    for (const child of this._children) {
      if (child instanceof File) {
        count++;
      } else if (child instanceof Directory) {
        count += child.getFileCount();
      }
    }
    return count;
  }
}

// สร้าง file system
const root = new Directory('project');

const src = new Directory('src');
const tests = new Directory('tests');
const docs = new Directory('docs');

src.add(new File('index.js', 1500, 'console.log("Hello")'))
   .add(new File('utils.js', 800))
   .add(new File('config.js', 400));

const components = new Directory('components');
components.add(new File('Button.jsx', 600))
          .add(new File('Input.jsx', 450))
          .add(new File('Modal.jsx', 750));
src.add(components);

tests.add(new File('index.test.js', 900))
     .add(new File('utils.test.js', 600));

docs.add(new File('README.md', 2000))
    .add(new File('API.md', 3500));

root.add(src)
    .add(tests)
    .add(docs)
    .add(new File('package.json', 500))
    .add(new File('.gitignore', 150));

// ใช้งาน
root.display();

console.log('\nTotal size:', root.getSize(), 'bytes');
console.log('Total files:', root.getFileCount());

// ค้นหาไฟล์ .js
const jsFiles = root.find(item => 
  item instanceof File && item.name.endsWith('.js')
);
console.log('\nJS files:');
jsFiles.forEach(f => console.log(f.getPath(), '-', f.getSize(), 'bytes'));

// ค้นหาไฟล์ขนาดใหญ่กว่า 1000 bytes
const largeFiles = root.find(item => 
  item instanceof File && item.getSize() > 1000
);
console.log('\nLarge files:');
largeFiles.forEach(f => console.log(f.getPath(), '-', f.getSize(), 'bytes'));
```

---

## Step 1022: Flyweight Pattern

**Flyweight Pattern** ลดการใช้หน่วยความจำโดยแชร์ state ร่วมกัน

```javascript
// ========================
// Flyweight Pattern
// ========================

// Intrinsic state (แชร์ได้)
class CharacterFlyweight {
  constructor(char, fontFamily, fontSize, bold, italic) {
    this.char = char;
    this.fontFamily = fontFamily;
    this.fontSize = fontSize;
    this.bold = bold;
    this.italic = italic;
  }
  
  render(x, y, color) {
    // Extrinsic state (x, y, color ไม่แชร์)
    return `Render '${this.char}' at (${x},${y}) color=${color} font=${this.fontFamily} ${this.fontSize}px ${this.bold ? 'bold' : ''} ${this.italic ? 'italic' : ''}`;
  }
  
  getMemorySize() {
    return 100; // จำลอง 100 bytes
  }
}

// Flyweight Factory
class CharacterFlyweightFactory {
  constructor() {
    this._flyweights = new Map();
    this._hitCount = 0;
    this._missCount = 0;
  }
  
  _getKey(char, fontFamily, fontSize, bold, italic) {
    return `${char}|${fontFamily}|${fontSize}|${bold}|${italic}`;
  }
  
  getFlyweight(char, fontFamily, fontSize, bold = false, italic = false) {
    const key = this._getKey(char, fontFamily, fontSize, bold, italic);
    
    if (!this._flyweights.has(key)) {
      this._flyweights.set(key, new CharacterFlyweight(char, fontFamily, fontSize, bold, italic));
      this._missCount++;
    } else {
      this._hitCount++;
    }
    
    return this._flyweights.get(key);
  }
  
  getStats() {
    return {
      uniqueFlyweights: this._flyweights.size,
      cacheHits: this._hitCount,
      cacheMisses: this._missCount,
      estimatedMemorySaved: `${this._hitCount * 100} bytes`
    };
  }
}

// Text editor ที่ใช้ Flyweight
class TextEditor {
  constructor() {
    this._factory = new CharacterFlyweightFactory();
    this._characters = [];
  }
  
  addCharacter(char, x, y, color, fontFamily = 'Arial', fontSize = 14, bold = false, italic = false) {
    const flyweight = this._factory.getFlyweight(char, fontFamily, fontSize, bold, italic);
    
    // Extrinsic state เก็บแยก
    this._characters.push({
      flyweight,
      x, y, color
    });
  }
  
  addText(text, startX, y, color, fontFamily = 'Arial', fontSize = 14) {
    for (let i = 0; i < text.length; i++) {
      this.addCharacter(text[i], startX + (i * fontSize * 0.6), y, color, fontFamily, fontSize);
    }
  }
  
  render() {
    return this._characters.map(({ flyweight, x, y, color }) =>
      flyweight.render(x, y, color)
    );
  }
  
  getMemoryStats() {
    const totalChars = this._characters.length;
    const flyweightStats = this._factory.getStats();
    const uniqueStyles = flyweightStats.uniqueFlyweights;
    
    return {
      totalCharacters: totalChars,
      uniqueStyles,
      ...flyweightStats,
      estimatedSavings: `Using ${uniqueStyles * 100} bytes instead of ${totalChars * 100} bytes (${(100 - (uniqueStyles / totalChars * 100)).toFixed(1)}% saved)`
    };
  }
}

// ใช้งาน
const editor = new TextEditor();

// เพิ่มข้อความมาก (มีตัวอักษรซ้ำๆ กัน)
const text = 'สวัสดีชาวโลก JavaScript เป็นภาษาที่น่าสนใจ!';
editor.addText(text, 10, 20, '#000000');
editor.addText(text, 10, 40, '#FF0000'); // สีแดง แต่ font เหมือนกัน
editor.addText(text, 10, 60, '#0000FF'); // สีน้ำเงิน

// ข้อความ bold
editor.addText('หัวข้อสำคัญ', 10, 80, '#333', 'Arial', 18, true);

console.log('\nMemory Stats:', editor.getMemoryStats());

// ========================
// Flyweight สำหรับ Particle System
// ========================

class ParticleFlyweight {
  constructor(color, shape) {
    this.color = color;
    this.shape = shape;
    this._texture = `texture_${color}_${shape}`; // จำลอง texture data
  }
  
  render(x, y, size, alpha) {
    // ใช้ shared texture แต่ position/size/alpha ต่างกัน
    return { texture: this._texture, x, y, size, alpha };
  }
}

class ParticleEffectSystem {
  constructor() {
    this._factory = new Map();
    this._particles = [];
  }
  
  _getFlyweight(color, shape) {
    const key = `${color}_${shape}`;
    if (!this._factory.has(key)) {
      this._factory.set(key, new ParticleFlyweight(color, shape));
    }
    return this._factory.get(key);
  }
  
  emit(color, shape, x, y, size = 5, alpha = 1) {
    const flyweight = this._getFlyweight(color, shape);
    this._particles.push({ flyweight, x, y, size, alpha });
  }
  
  emitExplosion(x, y, count = 50) {
    for (let i = 0; i < count; i++) {
      const angle = (Math.PI * 2 * i) / count;
      const distance = Math.random() * 100;
      this.emit(
        '#FF4500',
        'circle',
        x + Math.cos(angle) * distance,
        y + Math.sin(angle) * distance,
        Math.random() * 8 + 2,
        Math.random()
      );
    }
  }
  
  getStats() {
    return {
      particles: this._particles.length,
      flyweights: this._factory.size,
      ratio: `1:${Math.round(this._particles.length / this._factory.size)}`
    };
  }
}

const ps = new ParticleEffectSystem();
ps.emitExplosion(400, 300);
ps.emitExplosion(200, 200);
ps.emit('blue', 'star', 100, 100);
ps.emit('blue', 'star', 150, 120);
ps.emit('blue', 'star', 200, 140);

console.log('\nParticle System Stats:', ps.getStats());
```

---

## Step 1023: รวม Structural Patterns

```javascript
// ========================
// E-Commerce System ใช้ทุก Structural Pattern
// ========================

// 1. Facade - ระบบสั่งซื้อ
class ProductCatalog {
  getProduct(id) {
    return { id, name: `Product ${id}`, price: 100 * id, stock: 50 };
  }
  
  updateStock(productId, delta) {
    console.log(`Stock updated for product ${productId}: ${delta}`);
  }
}

class PricingEngine {
  calculate(product, quantity, userId) {
    let price = product.price;
    if (quantity > 10) price *= 0.9; // bulk discount
    if (userId === 'member') price *= 0.95; // member discount
    return price * quantity;
  }
  
  applyPromoCode(total, code) {
    if (code === 'THAI2024') return total * 0.8;
    return total;
  }
}

class PaymentGateway2 {
  charge(amount, paymentInfo) {
    console.log(`Charging ${amount} via ${paymentInfo.method}`);
    return { transactionId: `TXN_${Date.now()}`, success: true };
  }
}

class ShippingService {
  calculate(address, weight) {
    return { cost: 50, days: 3, carrier: 'Thailand Post' };
  }
  
  createShipment(orderId, address) {
    return { trackingNumber: `TH${Date.now()}`, orderId };
  }
}

class NotificationService {
  sendEmail(email, subject, body) {
    console.log(`Email to ${email}: ${subject}`);
  }
  
  sendSMS(phone, message) {
    console.log(`SMS to ${phone}: ${message}`);
  }
}

// Facade
class OrderFacade {
  constructor() {
    this._catalog = new ProductCatalog();
    this._pricing = new PricingEngine();
    this._payment = new PaymentGateway2();
    this._shipping = new ShippingService();
    this._notifications = new NotificationService();
  }
  
  placeOrder(orderData) {
    const { userId, items, address, paymentInfo, promoCode } = orderData;
    
    // 1. Get products
    const products = items.map(item => ({
      ...item,
      product: this._catalog.getProduct(item.productId)
    }));
    
    // 2. Calculate price
    let total = products.reduce((sum, item) => {
      return sum + this._pricing.calculate(item.product, item.quantity, userId);
    }, 0);
    
    if (promoCode) {
      total = this._pricing.applyPromoCode(total, promoCode);
    }
    
    // 3. Calculate shipping
    const shippingInfo = this._shipping.calculate(address, 1.5);
    total += shippingInfo.cost;
    
    // 4. Process payment
    const payment = this._payment.charge(total, paymentInfo);
    
    // 5. Update inventory
    products.forEach(item => {
      this._catalog.updateStock(item.productId, -item.quantity);
    });
    
    // 6. Create shipment
    const shipment = this._shipping.createShipment(`ORDER_${Date.now()}`, address);
    
    // 7. Send notifications
    this._notifications.sendEmail(
      orderData.email,
      'ยืนยันคำสั่งซื้อ',
      `คำสั่งซื้อของคุณ ${shipment.trackingNumber} กำลังดำเนินการ`
    );
    
    this._notifications.sendSMS(
      orderData.phone,
      `สั่งซื้อสำเร็จ! เลขติดตาม: ${shipment.trackingNumber}`
    );
    
    return {
      orderId: shipment.trackingNumber,
      total,
      transactionId: payment.transactionId,
      tracking: shipment.trackingNumber,
      estimatedDelivery: shippingInfo.days
    };
  }
}

// 2. Decorator - Add features to order
class OrderWithLogging {
  constructor(facade) {
    this._facade = facade;
    this._orders = [];
  }
  
  placeOrder(orderData) {
    console.log(`[${new Date().toISOString()}] Placing order for user ${orderData.userId}`);
    const result = this._facade.placeOrder(orderData);
    this._orders.push({ ...orderData, result, timestamp: Date.now() });
    console.log(`[${new Date().toISOString()}] Order completed: ${result.orderId}`);
    return result;
  }
  
  getOrderHistory() {
    return this._orders;
  }
}

// 3. Proxy - Rate limiting
class RateLimitedOrderProxy {
  constructor(facade, maxOrdersPerMinute = 5) {
    this._facade = facade;
    this._max = maxOrdersPerMinute;
    this._counts = new Map();
  }
  
  placeOrder(orderData) {
    const key = orderData.userId;
    const now = Date.now();
    const windowStart = now - 60000;
    
    if (!this._counts.has(key)) {
      this._counts.set(key, []);
    }
    
    const userCounts = this._counts.get(key).filter(t => t > windowStart);
    
    if (userCounts.length >= this._max) {
      throw new Error(`Rate limit exceeded. Max ${this._max} orders per minute`);
    }
    
    userCounts.push(now);
    this._counts.set(key, userCounts);
    
    return this._facade.placeOrder(orderData);
  }
}

// ใช้งาน
const orderSystem = new RateLimitedOrderProxy(
  new OrderWithLogging(
    new OrderFacade()
  )
);

const orderData = {
  userId: 'member',
  email: 'customer@example.com',
  phone: '0812345678',
  items: [
    { productId: 1, quantity: 3 },
    { productId: 2, quantity: 1 }
  ],
  address: {
    street: '123 สุขุมวิท',
    city: 'กรุงเทพฯ'
  },
  paymentInfo: { method: 'credit_card', token: 'tok_visa' },
  promoCode: 'THAI2024'
};

const result = orderSystem.placeOrder(orderData);
console.log('\nOrder Result:', result);
```

---

## Step 1024-1030: แบบฝึกหัดและสรุป

### แบบฝึกหัด (Exercises)

**Exercise 1: Adapter**
```javascript
// สร้าง Adapter สำหรับ SMS service:
// OldSMSService มี: sendText(phoneNumber, message)
// NewSMSInterface ต้องการ: send({ to, content, priority })
// สร้าง SMSAdapter

class OldSMSService {
  sendText(phoneNumber, message) {
    return `SMS sent to ${phoneNumber}: ${message}`;
  }
}

class SMSAdapter {
  // TODO: implement adapter
  // send({ to, content, priority })
}

// Test
const sms = new SMSAdapter();
sms.send({ to: '0812345678', content: 'สวัสดี', priority: 'high' });
```

**Exercise 2: Decorator**
```javascript
// สร้าง Decorators สำหรับ TextFormatter:
// - UpperCase: แปลงเป็นตัวพิมพ์ใหญ่
// - Trim: ตัดช่องว่าง
// - Truncate(n): ตัดข้อความที่ n ตัวอักษร
// - Prefix(text): เพิ่ม prefix
// - Suffix(text): เพิ่ม suffix
// สามารถ stack ได้

class TextFormatter {
  constructor(text) {
    this.text = text;
  }
  
  format() {
    return this.text;
  }
}

// ใช้งาน:
// const result = new Suffix('!')
//   (new Prefix('>>>')
//     (new Truncate(20)
//       (new Trim(
//         new TextFormatter('  Hello World  ')
//       ))
//     )
//   ).format();
```

**Exercise 3: Facade**
```javascript
// สร้าง Facade สำหรับระบบห้องสมุดออนไลน์:
// - BookCatalog: search(query), getBook(id)
// - UserService: getUser(userId), checkMembership(userId)
// - LoanSystem: createLoan(userId, bookId), returnBook(loanId)
// - NotificationService: notifyDue(userId, dueDate)
// - ReservationSystem: reserve(userId, bookId)
// 
// LibraryFacade ต้อง:
// - borrowBook(userId, bookId)
// - returnBook(userId, loanId)  
// - searchAndReserve(userId, query)

class LibraryFacade {
  // TODO
}
```

**Exercise 4: Composite**
```javascript
// สร้าง UI Component Tree:
// - Component (base): render(), getSize(), find(selector)
// - TextNode (leaf): text, style
// - Container (composite): addChild(), removeChild(), layout
// 
// ตัวอย่าง:
// const page = new Container('page')
//   .addChild(
//     new Container('header')
//       .addChild(new TextNode('Logo', { bold: true }))
//       .addChild(new TextNode('Menu'))
//   )
//   .addChild(
//     new Container('content')
//       .addChild(new TextNode('Hello World'))
//   );
// page.render();
// page.find('.header'); // หา container ชื่อ header
```

### สรุป Structural Patterns

| Pattern | เมื่อใช้ | ตัวอย่างจริง |
|---------|--------|------------|
| Adapter | เชื่อม API เก่ากับใหม่ | Payment gateway adapter |
| Decorator | เพิ่มฟีเจอร์โดยไม่แก้ code | Middleware, Coffee |
| Facade | ซ่อนความซับซ้อน | SDK, Library |
| Proxy | ควบคุมการเข้าถึง | Cache, Auth, Logging |
| Bridge | แยก abstraction/impl | Cross-platform rendering |
| Composite | tree structure | File system, UI components |
| Flyweight | ลด memory | Text editor, Particles |

```javascript
// Quick Reference

// Adapter
class NewAdapter extends NewInterface {
  constructor(oldSystem) { this._old = oldSystem; }
  newMethod(newParams) { return this._old.oldMethod(adaptParams(newParams)); }
}

// Decorator
class WithFeature extends BaseClass {
  constructor(wrapped) { this._wrapped = wrapped; }
  operation() { 
    // ทำก่อน
    const result = this._wrapped.operation();
    // ทำหลัง
    return result;
  }
}

// Facade
class SimplifiedInterface {
  constructor() {
    this._subsystem1 = new Subsystem1();
    this._subsystem2 = new Subsystem2();
  }
  simpleOperation() {
    this._subsystem1.complex1();
    this._subsystem2.complex2();
  }
}

// Proxy
class ControlledAccess {
  constructor(realObject) { this._real = realObject; }
  operation() {
    if (this._canAccess()) return this._real.operation();
    throw new Error('Access denied');
  }
}
```

---

**ขั้นตอนต่อไป:** ใน Part 53 เราจะเรียนรู้ **Behavioral Patterns** ที่เกี่ยวกับพฤติกรรมและการสื่อสารระหว่าง objects
