# Part 91: Code Architecture Patterns (Steps 1791-1810)

## สถาปัตยกรรมโค้ดที่ดี - Clean Architecture และ DDD ใน JavaScript

ในส่วนนี้เราจะเรียนรู้เกี่ยวกับสถาปัตยกรรมโค้ดที่ทำให้แอปพลิเคชันของเรามีโครงสร้างที่ดี บำรุงรักษาง่าย และขยายได้ โดยครอบคลุม Clean Architecture, Hexagonal Architecture, Domain-Driven Design และหลักการ SOLID

---

## Step 1791: Clean Architecture Overview

Clean Architecture คือแนวคิดที่ Robert C. Martin (Uncle Bob) คิดขึ้น เพื่อแยกส่วนต่างๆ ของแอปพลิเคชันออกจากกัน ทำให้แต่ละส่วนทดสอบได้อิสระและเปลี่ยนแปลงได้โดยไม่กระทบส่วนอื่น

```
// โครงสร้าง Clean Architecture
// 
//  ┌─────────────────────────────────┐
//  │         Frameworks & Drivers    │  (Express, React, Database)
//  │  ┌───────────────────────────┐  │
//  │  │    Interface Adapters     │  │  (Controllers, Presenters, Gateways)
//  │  │  ┌─────────────────────┐ │  │
//  │  │  │  Application Logic  │ │  │  (Use Cases)
//  │  │  │  ┌───────────────┐ │ │  │
//  │  │  │  │  Domain Logic │ │ │  │  (Entities, Business Rules)
//  │  │  │  └───────────────┘ │ │  │
//  │  │  └─────────────────────┘ │  │
//  │  └───────────────────────────┘  │
//  └─────────────────────────────────┘
//
// กฎ: dependencies ไหลเข้าด้านใน (inward)
// ชั้นในสุดไม่รู้จักชั้นนอก
```

```javascript
// ตัวอย่างโครงสร้างโปรเจค Clean Architecture

// src/
//   domain/
//     entities/
//       User.js
//       Order.js
//     repositories/
//       IUserRepository.js      <- interfaces/ports
//       IOrderRepository.js
//     services/
//       UserDomainService.js
//     value-objects/
//       Email.js
//       Money.js
//   application/
//     use-cases/
//       CreateUser.js
//       PlaceOrder.js
//     dto/
//       UserDto.js
//   infrastructure/
//     repositories/
//       MongoUserRepository.js   <- implementations/adapters
//       PgOrderRepository.js
//     web/
//       routes/
//         userRoutes.js
//       controllers/
//         UserController.js
//   index.js

// domain/entities/User.js - ชั้นในสุด ไม่ import อะไรจากนอก
class User {
  constructor({ id, name, email, createdAt }) {
    this.id = id;
    this.name = name;
    this.email = email;
    this.createdAt = createdAt || new Date();
    this.validate();
  }

  validate() {
    if (!this.name || this.name.trim().length < 2) {
      throw new Error('ชื่อต้องมีอย่างน้อย 2 ตัวอักษร');
    }
    if (!this.email || !this.email.includes('@')) {
      throw new Error('อีเมลไม่ถูกต้อง');
    }
  }

  changeName(newName) {
    if (!newName || newName.trim().length < 2) {
      throw new Error('ชื่อใหม่ต้องมีอย่างน้อย 2 ตัวอักษร');
    }
    this.name = newName;
    return this;
  }

  toJSON() {
    return {
      id: this.id,
      name: this.name,
      email: this.email,
      createdAt: this.createdAt,
    };
  }
}

module.exports = { User };
```

```javascript
// domain/repositories/IUserRepository.js - Interface/Port
// ใน JavaScript เราใช้ abstract class หรือ comment แทน interface
class IUserRepository {
  async findById(id) {
    throw new Error('findById ต้อง implement');
  }
  
  async findByEmail(email) {
    throw new Error('findByEmail ต้อง implement');
  }
  
  async save(user) {
    throw new Error('save ต้อง implement');
  }
  
  async delete(id) {
    throw new Error('delete ต้อง implement');
  }
}

module.exports = { IUserRepository };
```

---

## Step 1792: Hexagonal Architecture (Ports and Adapters)

Hexagonal Architecture หรือ Ports and Adapters คือสถาปัตยกรรมที่ทำให้ application core ไม่ขึ้นกับ infrastructure

```javascript
// Core Application (Hexagon)
// Ports = interfaces ที่ core ต้องการ
// Adapters = implementations จากภายนอก

// ports/UserRepositoryPort.js
class UserRepositoryPort {
  async findById(id) { throw new Error('Not implemented'); }
  async save(user) { throw new Error('Not implemented'); }
  async findAll() { throw new Error('Not implemented'); }
}

// ports/NotificationPort.js
class NotificationPort {
  async sendEmail(to, subject, body) { throw new Error('Not implemented'); }
  async sendSMS(phone, message) { throw new Error('Not implemented'); }
}

// core/UserService.js - Application Core
class UserService {
  constructor(userRepository, notificationService) {
    this.userRepository = userRepository; // Port
    this.notificationService = notificationService; // Port
  }

  async registerUser(userData) {
    // Validate
    const existingUser = await this.userRepository.findByEmail(userData.email);
    if (existingUser) {
      throw new Error('อีเมลนี้ถูกใช้งานแล้ว');
    }

    // Create entity
    const user = new User({
      id: generateId(),
      ...userData,
    });

    // Save
    await this.userRepository.save(user);

    // Notify
    await this.notificationService.sendEmail(
      user.email,
      'ยินดีต้อนรับ!',
      `สวัสดี ${user.name}, บัญชีของคุณถูกสร้างแล้ว`
    );

    return user;
  }
}

// adapters/MongoUserRepository.js - Adapter
const mongoose = require('mongoose');

class MongoUserRepository extends UserRepositoryPort {
  constructor(UserModel) {
    super();
    this.UserModel = UserModel;
  }

  async findById(id) {
    const doc = await this.UserModel.findById(id);
    if (!doc) return null;
    return new User({
      id: doc._id.toString(),
      name: doc.name,
      email: doc.email,
      createdAt: doc.createdAt,
    });
  }

  async findByEmail(email) {
    const doc = await this.UserModel.findOne({ email });
    if (!doc) return null;
    return new User({
      id: doc._id.toString(),
      name: doc.name,
      email: doc.email,
      createdAt: doc.createdAt,
    });
  }

  async save(user) {
    await this.UserModel.findByIdAndUpdate(
      user.id,
      { name: user.name, email: user.email },
      { upsert: true, new: true }
    );
    return user;
  }

  async findAll() {
    const docs = await this.UserModel.find({});
    return docs.map(doc => new User({
      id: doc._id.toString(),
      name: doc.name,
      email: doc.email,
      createdAt: doc.createdAt,
    }));
  }
}

// adapters/SendGridEmailAdapter.js - Adapter
const sgMail = require('@sendgrid/mail');

class SendGridEmailAdapter extends NotificationPort {
  constructor(apiKey) {
    super();
    sgMail.setApiKey(apiKey);
  }

  async sendEmail(to, subject, body) {
    await sgMail.send({
      to,
      from: 'noreply@example.com',
      subject,
      html: body,
    });
  }

  async sendSMS(phone, message) {
    // SMS ไม่รองรับใน SendGrid ต้อง throw
    throw new Error('SendGrid ไม่รองรับ SMS');
  }
}

// Composition Root - เชื่อมต่อทุกอย่าง
function createApplication() {
  const userRepository = new MongoUserRepository(UserModel);
  const notificationService = new SendGridEmailAdapter(process.env.SENDGRID_API_KEY);
  const userService = new UserService(userRepository, notificationService);
  return { userService };
}
```

---

## Step 1793: Domain-Driven Design (DDD) Concepts

DDD คือแนวทางการออกแบบซอฟต์แวร์ที่เน้นที่ domain logic เป็นหลัก

```javascript
// DDD Building Blocks:
// 1. Entities - มี identity
// 2. Value Objects - ไม่มี identity
// 3. Aggregates - กลุ่มของ entities
// 4. Domain Services - logic ที่ไม่เหมาะกับ entity เดียว
// 5. Repositories - abstraction สำหรับ data access
// 6. Domain Events - events ที่เกิดขึ้นใน domain
// 7. Factories - สร้าง complex objects

// ตัวอย่าง E-commerce Domain

// Value Object: Money
class Money {
  constructor(amount, currency) {
    if (typeof amount !== 'number' || amount < 0) {
      throw new Error('จำนวนเงินต้องเป็นตัวเลขบวก');
    }
    if (!['THB', 'USD', 'EUR'].includes(currency)) {
      throw new Error('สกุลเงินไม่รองรับ');
    }
    // Value Objects เป็น immutable
    Object.freeze({ amount, currency });
    this._amount = amount;
    this._currency = currency;
  }

  get amount() { return this._amount; }
  get currency() { return this._currency; }

  add(other) {
    if (this._currency !== other._currency) {
      throw new Error('ไม่สามารถบวกเงินต่างสกุลได้');
    }
    return new Money(this._amount + other._amount, this._currency);
  }

  subtract(other) {
    if (this._currency !== other._currency) {
      throw new Error('ไม่สามารถลบเงินต่างสกุลได้');
    }
    if (this._amount < other._amount) {
      throw new Error('เงินไม่พอ');
    }
    return new Money(this._amount - other._amount, this._currency);
  }

  multiply(factor) {
    return new Money(Math.round(this._amount * factor * 100) / 100, this._currency);
  }

  equals(other) {
    return this._amount === other._amount && this._currency === other._currency;
  }

  isGreaterThan(other) {
    if (this._currency !== other._currency) throw new Error('ต่างสกุลเงิน');
    return this._amount > other._amount;
  }

  toString() {
    return `${this._amount} ${this._currency}`;
  }
}

// ใช้งาน
const price = new Money(100, 'THB');
const tax = price.multiply(0.07);
const total = price.add(tax);
console.log(total.toString()); // 107 THB
```

```javascript
// Entity: Product
class Product {
  constructor({ id, name, price, stock, categoryId }) {
    this._id = id;
    this._name = name;
    this._price = price instanceof Money ? price : new Money(price.amount, price.currency);
    this._stock = stock;
    this._categoryId = categoryId;
    this._validate();
  }

  get id() { return this._id; }
  get name() { return this._name; }
  get price() { return this._price; }
  get stock() { return this._stock; }
  get categoryId() { return this._categoryId; }
  get isAvailable() { return this._stock > 0; }

  _validate() {
    if (!this._name || this._name.trim().length < 2) {
      throw new Error('ชื่อสินค้าต้องมีอย่างน้อย 2 ตัวอักษร');
    }
    if (this._stock < 0) {
      throw new Error('จำนวนสต็อกต้องไม่ติดลบ');
    }
  }

  reduceStock(quantity) {
    if (quantity <= 0) throw new Error('จำนวนต้องมากกว่า 0');
    if (this._stock < quantity) throw new Error('สต็อกไม่พอ');
    this._stock -= quantity;
    return this;
  }

  increaseStock(quantity) {
    if (quantity <= 0) throw new Error('จำนวนต้องมากกว่า 0');
    this._stock += quantity;
    return this;
  }

  changePrice(newPrice) {
    if (!(newPrice instanceof Money)) throw new Error('ราคาต้องเป็น Money object');
    this._price = newPrice;
    return this;
  }
}
```

---

## Step 1794: Repository Pattern

Repository Pattern คือ abstraction layer ระหว่าง business logic และ data access

```javascript
// Generic Repository Base
class BaseRepository {
  constructor(model) {
    this.model = model;
  }

  async findById(id) {
    return await this.model.findById(id);
  }

  async findAll(options = {}) {
    const { page = 1, limit = 10, sort = { createdAt: -1 } } = options;
    const skip = (page - 1) * limit;
    return await this.model.find({}).sort(sort).skip(skip).limit(limit);
  }

  async create(data) {
    const entity = new this.model(data);
    return await entity.save();
  }

  async update(id, data) {
    return await this.model.findByIdAndUpdate(id, data, { new: true });
  }

  async delete(id) {
    return await this.model.findByIdAndDelete(id);
  }

  async count(filter = {}) {
    return await this.model.countDocuments(filter);
  }
}

// Specific Repository
class ProductRepository extends BaseRepository {
  constructor(ProductModel) {
    super(ProductModel);
  }

  async findByCategory(categoryId, options = {}) {
    const { page = 1, limit = 10 } = options;
    const skip = (page - 1) * limit;
    
    const [items, total] = await Promise.all([
      this.model.find({ categoryId }).skip(skip).limit(limit),
      this.model.countDocuments({ categoryId }),
    ]);

    return {
      items,
      total,
      page,
      totalPages: Math.ceil(total / limit),
    };
  }

  async findAvailable() {
    return await this.model.find({ stock: { $gt: 0 } });
  }

  async findByPriceRange(min, max, currency = 'THB') {
    return await this.model.find({
      'price.amount': { $gte: min, $lte: max },
      'price.currency': currency,
    });
  }

  async searchByName(query) {
    return await this.model.find({
      name: { $regex: query, $options: 'i' },
    });
  }

  async updateStock(productId, quantity, operation = 'reduce') {
    const update = operation === 'reduce'
      ? { $inc: { stock: -quantity } }
      : { $inc: { stock: quantity } };
    
    return await this.model.findByIdAndUpdate(
      productId,
      update,
      { new: true }
    );
  }
}

// In-Memory Repository for Testing
class InMemoryProductRepository {
  constructor() {
    this.products = new Map();
    this.nextId = 1;
  }

  async findById(id) {
    return this.products.get(id) || null;
  }

  async findAll() {
    return Array.from(this.products.values());
  }

  async save(product) {
    if (!product.id) {
      product._id = String(this.nextId++);
    }
    this.products.set(product.id, product);
    return product;
  }

  async delete(id) {
    const existed = this.products.has(id);
    this.products.delete(id);
    return existed;
  }

  async findByCategory(categoryId) {
    return Array.from(this.products.values())
      .filter(p => p.categoryId === categoryId);
  }

  clear() {
    this.products.clear();
    this.nextId = 1;
  }
}

// ใช้งาน
async function demonstrateRepository() {
  const repo = new InMemoryProductRepository();

  const product = new Product({
    id: '1',
    name: 'สินค้าทดสอบ',
    price: new Money(299, 'THB'),
    stock: 100,
    categoryId: 'cat-1',
  });

  await repo.save(product);

  const found = await repo.findById('1');
  console.log(found.name); // สินค้าทดสอบ

  const byCategory = await repo.findByCategory('cat-1');
  console.log(byCategory.length); // 1
}
```

---

## Step 1795: Service Layer Pattern

Service Layer รวม use cases ของ application ไว้ในที่เดียว

```javascript
// application/services/OrderService.js
class OrderService {
  constructor({
    orderRepository,
    productRepository,
    userRepository,
    paymentService,
    notificationService,
    eventBus,
  }) {
    this.orderRepository = orderRepository;
    this.productRepository = productRepository;
    this.userRepository = userRepository;
    this.paymentService = paymentService;
    this.notificationService = notificationService;
    this.eventBus = eventBus;
  }

  async createOrder(userId, items) {
    // 1. ตรวจสอบ user
    const user = await this.userRepository.findById(userId);
    if (!user) throw new Error('ไม่พบผู้ใช้');

    // 2. ตรวจสอบสินค้าและสต็อก
    const orderItems = [];
    let totalAmount = new Money(0, 'THB');

    for (const item of items) {
      const product = await this.productRepository.findById(item.productId);
      if (!product) throw new Error(`ไม่พบสินค้า: ${item.productId}`);
      if (!product.isAvailable) throw new Error(`สินค้าหมด: ${product.name}`);
      if (product.stock < item.quantity) {
        throw new Error(`สต็อกไม่พอ: ${product.name}`);
      }

      const itemTotal = product.price.multiply(item.quantity);
      totalAmount = totalAmount.add(itemTotal);
      orderItems.push({
        productId: product.id,
        productName: product.name,
        quantity: item.quantity,
        unitPrice: product.price,
        total: itemTotal,
      });
    }

    // 3. สร้าง Order
    const order = new Order({
      id: generateId(),
      userId,
      items: orderItems,
      totalAmount,
      status: 'PENDING',
      createdAt: new Date(),
    });

    // 4. บันทึก Order
    await this.orderRepository.save(order);

    // 5. ลดสต็อก
    for (const item of items) {
      await this.productRepository.updateStock(item.productId, item.quantity, 'reduce');
    }

    // 6. Publish event
    await this.eventBus.publish(new OrderCreatedEvent({
      orderId: order.id,
      userId,
      totalAmount,
    }));

    // 7. แจ้งเตือน
    await this.notificationService.sendEmail(
      user.email,
      'คำสั่งซื้อของคุณได้รับแล้ว',
      `คำสั่งซื้อ #${order.id} ยอดรวม ${totalAmount.toString()}`
    );

    return order;
  }

  async cancelOrder(orderId, reason) {
    const order = await this.orderRepository.findById(orderId);
    if (!order) throw new Error('ไม่พบคำสั่งซื้อ');
    if (!order.canBeCancelled) throw new Error('ไม่สามารถยกเลิกได้');

    // คืนสต็อก
    for (const item of order.items) {
      await this.productRepository.updateStock(
        item.productId,
        item.quantity,
        'increase'
      );
    }

    order.cancel(reason);
    await this.orderRepository.save(order);

    await this.eventBus.publish(new OrderCancelledEvent({
      orderId: order.id,
      reason,
    }));

    return order;
  }

  async getOrderHistory(userId, options = {}) {
    return await this.orderRepository.findByUser(userId, options);
  }
}

module.exports = { OrderService };
```

---

## Step 1796: Domain Models and Entities

```javascript
// domain/entities/Order.js
class Order {
  constructor({ id, userId, items, totalAmount, status, createdAt }) {
    this._id = id;
    this._userId = userId;
    this._items = items;
    this._totalAmount = totalAmount;
    this._status = status || 'PENDING';
    this._createdAt = createdAt || new Date();
    this._updatedAt = new Date();
    this._domainEvents = [];
    this._validate();
  }

  get id() { return this._id; }
  get userId() { return this._userId; }
  get items() { return [...this._items]; }
  get totalAmount() { return this._totalAmount; }
  get status() { return this._status; }
  get createdAt() { return this._createdAt; }
  get domainEvents() { return [...this._domainEvents]; }
  
  get canBeCancelled() {
    return ['PENDING', 'CONFIRMED'].includes(this._status);
  }
  
  get isCompleted() { return this._status === 'DELIVERED'; }

  _validate() {
    if (!this._userId) throw new Error('Order ต้องมี userId');
    if (!this._items || this._items.length === 0) {
      throw new Error('Order ต้องมีสินค้าอย่างน้อย 1 ชิ้น');
    }
  }

  confirm() {
    if (this._status !== 'PENDING') {
      throw new Error('สามารถยืนยันได้เฉพาะ order ที่ pending เท่านั้น');
    }
    this._status = 'CONFIRMED';
    this._updatedAt = new Date();
    this._addDomainEvent(new OrderConfirmedEvent({ orderId: this._id }));
    return this;
  }

  ship() {
    if (this._status !== 'CONFIRMED') {
      throw new Error('สามารถจัดส่งได้เฉพาะ order ที่ confirmed เท่านั้น');
    }
    this._status = 'SHIPPED';
    this._updatedAt = new Date();
    this._addDomainEvent(new OrderShippedEvent({ orderId: this._id }));
    return this;
  }

  deliver() {
    if (this._status !== 'SHIPPED') {
      throw new Error('สามารถส่งมอบได้เฉพาะ order ที่ shipped เท่านั้น');
    }
    this._status = 'DELIVERED';
    this._updatedAt = new Date();
    this._addDomainEvent(new OrderDeliveredEvent({ orderId: this._id }));
    return this;
  }

  cancel(reason) {
    if (!this.canBeCancelled) {
      throw new Error('ไม่สามารถยกเลิก order นี้ได้');
    }
    this._status = 'CANCELLED';
    this._cancellationReason = reason;
    this._updatedAt = new Date();
    this._addDomainEvent(new OrderCancelledEvent({
      orderId: this._id,
      reason,
    }));
    return this;
  }

  _addDomainEvent(event) {
    this._domainEvents.push(event);
  }

  clearDomainEvents() {
    this._domainEvents = [];
  }

  toJSON() {
    return {
      id: this._id,
      userId: this._userId,
      items: this._items,
      totalAmount: this._totalAmount,
      status: this._status,
      createdAt: this._createdAt,
      updatedAt: this._updatedAt,
    };
  }
}

module.exports = { Order };
```

---

## Step 1797: Value Objects

Value Objects เป็น objects ที่ไม่มี identity แต่มีค่า และเป็น immutable

```javascript
// domain/value-objects/Email.js
class Email {
  constructor(value) {
    if (!Email.isValid(value)) {
      throw new Error(`"${value}" ไม่ใช่อีเมลที่ถูกต้อง`);
    }
    this._value = value.toLowerCase().trim();
    Object.freeze(this);
  }

  get value() { return this._value; }

  static isValid(value) {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    return typeof value === 'string' && emailRegex.test(value);
  }

  equals(other) {
    if (!(other instanceof Email)) return false;
    return this._value === other._value;
  }

  toString() { return this._value; }
}

// domain/value-objects/Address.js
class Address {
  constructor({ street, city, province, postalCode, country = 'TH' }) {
    if (!street) throw new Error('ต้องระบุที่อยู่');
    if (!city) throw new Error('ต้องระบุเมือง');
    if (!province) throw new Error('ต้องระบุจังหวัด');
    if (!postalCode || !/^\d{5}$/.test(postalCode)) {
      throw new Error('รหัสไปรษณีย์ต้องเป็นตัวเลข 5 หลัก');
    }

    this._street = street;
    this._city = city;
    this._province = province;
    this._postalCode = postalCode;
    this._country = country;
    Object.freeze(this);
  }

  get street() { return this._street; }
  get city() { return this._city; }
  get province() { return this._province; }
  get postalCode() { return this._postalCode; }
  get country() { return this._country; }

  equals(other) {
    if (!(other instanceof Address)) return false;
    return (
      this._street === other._street &&
      this._city === other._city &&
      this._province === other._province &&
      this._postalCode === other._postalCode
    );
  }

  toString() {
    return `${this._street}, ${this._city}, ${this._province} ${this._postalCode}`;
  }
}

// domain/value-objects/PhoneNumber.js
class PhoneNumber {
  constructor(value) {
    const cleaned = value.replace(/[-\s()]/g, '');
    if (!PhoneNumber.isValid(cleaned)) {
      throw new Error('เบอร์โทรศัพท์ไม่ถูกต้อง');
    }
    this._value = cleaned;
    Object.freeze(this);
  }

  static isValid(value) {
    // ตรวจสอบเบอร์โทรไทย
    return /^(0[689]\d{8}|02\d{7})$/.test(value);
  }

  get value() { return this._value; }

  format() {
    if (this._value.startsWith('02')) {
      return `${this._value.slice(0, 2)}-${this._value.slice(2, 5)}-${this._value.slice(5)}`;
    }
    return `${this._value.slice(0, 3)}-${this._value.slice(3, 6)}-${this._value.slice(6)}`;
  }

  equals(other) {
    if (!(other instanceof PhoneNumber)) return false;
    return this._value === other._value;
  }

  toString() { return this.format(); }
}

// ใช้งาน
const email = new Email('user@example.com');
const address = new Address({
  street: '123 ถ.สุขุมวิท',
  city: 'กรุงเทพฯ',
  province: 'กรุงเทพมหานคร',
  postalCode: '10110',
});
const phone = new PhoneNumber('0812345678');

console.log(email.toString());    // user@example.com
console.log(address.toString());  // 123 ถ.สุขุมวิท, กรุงเทพฯ, กรุงเทพมหานคร 10110
console.log(phone.format());      // 081-234-5678
```

---

## Step 1798: Aggregate Roots

Aggregate Root คือ entity หลักที่ควบคุม aggregate ทั้งหมด

```javascript
// domain/aggregates/ShoppingCart.js
class CartItem {
  constructor({ productId, productName, quantity, unitPrice }) {
    this.productId = productId;
    this.productName = productName;
    this.quantity = quantity;
    this.unitPrice = unitPrice instanceof Money
      ? unitPrice
      : new Money(unitPrice.amount, unitPrice.currency);
  }

  get total() {
    return this.unitPrice.multiply(this.quantity);
  }

  increaseQuantity(amount) {
    return new CartItem({
      ...this,
      quantity: this.quantity + amount,
    });
  }
}

// ShoppingCart เป็น Aggregate Root
class ShoppingCart {
  constructor({ id, userId, items = [], createdAt }) {
    this._id = id;
    this._userId = userId;
    this._items = new Map(items.map(item => [item.productId, new CartItem(item)]));
    this._createdAt = createdAt || new Date();
  }

  get id() { return this._id; }
  get userId() { return this._userId; }
  
  get items() {
    return Array.from(this._items.values());
  }

  get itemCount() {
    return Array.from(this._items.values())
      .reduce((sum, item) => sum + item.quantity, 0);
  }

  get totalAmount() {
    const items = Array.from(this._items.values());
    if (items.length === 0) return new Money(0, 'THB');
    return items.reduce(
      (total, item) => total.add(item.total),
      new Money(0, 'THB')
    );
  }

  get isEmpty() { return this._items.size === 0; }

  addItem(product, quantity = 1) {
    if (quantity <= 0) throw new Error('จำนวนต้องมากกว่า 0');
    if (!product.isAvailable) throw new Error(`${product.name} หมดสต็อก`);
    
    if (this._items.has(product.id)) {
      const existing = this._items.get(product.id);
      const newQuantity = existing.quantity + quantity;
      if (newQuantity > product.stock) {
        throw new Error(`สต็อกไม่พอ: มีเพียง ${product.stock} ชิ้น`);
      }
      this._items.set(product.id, existing.increaseQuantity(quantity));
    } else {
      if (quantity > product.stock) {
        throw new Error(`สต็อกไม่พอ: มีเพียง ${product.stock} ชิ้น`);
      }
      this._items.set(product.id, new CartItem({
        productId: product.id,
        productName: product.name,
        quantity,
        unitPrice: product.price,
      }));
    }

    return this;
  }

  removeItem(productId) {
    if (!this._items.has(productId)) {
      throw new Error('ไม่พบสินค้านี้ในตะกร้า');
    }
    this._items.delete(productId);
    return this;
  }

  updateQuantity(productId, quantity) {
    if (quantity <= 0) {
      return this.removeItem(productId);
    }
    
    if (!this._items.has(productId)) {
      throw new Error('ไม่พบสินค้านี้ในตะกร้า');
    }

    const item = this._items.get(productId);
    this._items.set(productId, new CartItem({
      ...item,
      quantity,
    }));

    return this;
  }

  clear() {
    this._items.clear();
    return this;
  }

  checkout() {
    if (this.isEmpty) throw new Error('ตะกร้าว่างเปล่า');
    
    const orderItems = this.items.map(item => ({
      productId: item.productId,
      productName: item.productName,
      quantity: item.quantity,
      unitPrice: item.unitPrice,
      total: item.total,
    }));

    return {
      userId: this._userId,
      items: orderItems,
      totalAmount: this.totalAmount,
    };
  }
}
```

---

## Step 1799: Domain Events

Domain Events คือ events ที่เกิดขึ้นใน domain business

```javascript
// domain/events/DomainEvent.js
class DomainEvent {
  constructor(eventName, payload) {
    this.eventName = eventName;
    this.payload = payload;
    this.occurredAt = new Date();
    this.id = `${eventName}_${Date.now()}_${Math.random().toString(36).slice(2)}`;
  }
}

// domain/events/order-events.js
class OrderCreatedEvent extends DomainEvent {
  constructor({ orderId, userId, totalAmount }) {
    super('ORDER_CREATED', { orderId, userId, totalAmount });
  }
}

class OrderConfirmedEvent extends DomainEvent {
  constructor({ orderId }) {
    super('ORDER_CONFIRMED', { orderId });
  }
}

class OrderShippedEvent extends DomainEvent {
  constructor({ orderId, trackingNumber }) {
    super('ORDER_SHIPPED', { orderId, trackingNumber });
  }
}

class OrderCancelledEvent extends DomainEvent {
  constructor({ orderId, reason }) {
    super('ORDER_CANCELLED', { orderId, reason });
  }
}

// infrastructure/EventBus.js
class EventBus {
  constructor() {
    this._handlers = new Map();
  }

  subscribe(eventName, handler) {
    if (!this._handlers.has(eventName)) {
      this._handlers.set(eventName, []);
    }
    this._handlers.get(eventName).push(handler);
    
    // Return unsubscribe function
    return () => {
      const handlers = this._handlers.get(eventName);
      const index = handlers.indexOf(handler);
      if (index > -1) handlers.splice(index, 1);
    };
  }

  async publish(event) {
    const handlers = this._handlers.get(event.eventName) || [];
    const results = await Promise.allSettled(
      handlers.map(handler => handler(event))
    );
    
    // Log failures
    results.forEach((result, i) => {
      if (result.status === 'rejected') {
        console.error(
          `Handler ${i} failed for event ${event.eventName}:`,
          result.reason
        );
      }
    });
  }
}

// ตัวอย่างการใช้ Domain Events
const eventBus = new EventBus();

// Subscribe to events
eventBus.subscribe('ORDER_CREATED', async (event) => {
  console.log(`Order created: ${event.payload.orderId}`);
  // ส่งอีเมลยืนยัน
  await sendConfirmationEmail(event.payload);
});

eventBus.subscribe('ORDER_CREATED', async (event) => {
  // Update inventory analytics
  await updateInventoryStats(event.payload);
});

eventBus.subscribe('ORDER_SHIPPED', async (event) => {
  // ส่ง SMS tracking
  await sendTrackingSMS(event.payload);
});

// Publish event
await eventBus.publish(new OrderCreatedEvent({
  orderId: '123',
  userId: 'user-1',
  totalAmount: new Money(500, 'THB'),
}));
```

---

## Step 1800: SOLID Principles with JavaScript Examples

```javascript
// S - Single Responsibility Principle (SRP)
// แต่ละ class มีความรับผิดชอบเดียว

// ❌ ผิด - User class รับผิดชอบหลายอย่าง
class BadUser {
  constructor(name, email) {
    this.name = name;
    this.email = email;
  }

  save() { /* save to DB */ }
  
  sendEmail() { /* send email */ }
  
  renderProfilePage() { /* return HTML */ }
  
  calculateAge() { /* calculate age */ }
}

// ✅ ถูก - แยกความรับผิดชอบ
class User {
  constructor(name, email, birthDate) {
    this.name = name;
    this.email = email;
    this.birthDate = birthDate;
  }
}

class UserRepository {
  async save(user) { /* save to DB */ }
  async findById(id) { /* find from DB */ }
}

class UserEmailService {
  async sendWelcomeEmail(user) { /* send email */ }
  async sendPasswordReset(user) { /* send email */ }
}

class UserAgeCalculator {
  calculateAge(birthDate) {
    const today = new Date();
    const age = today.getFullYear() - birthDate.getFullYear();
    return age;
  }
}
```

```javascript
// O - Open/Closed Principle (OCP)
// เปิดสำหรับการขยาย แต่ปิดสำหรับการแก้ไข

// ❌ ผิด - ต้องแก้ไข function ทุกครั้งที่เพิ่ม discount type
function calculateDiscountBad(order, discountType) {
  if (discountType === 'PERCENTAGE') {
    return order.total * 0.1;
  } else if (discountType === 'FIXED') {
    return 50;
  } else if (discountType === 'VIP') {
    return order.total * 0.2;
  }
  // ถ้าเพิ่ม type ใหม่ ต้องแก้ function นี้
}

// ✅ ถูก - ใช้ Strategy pattern
class PercentageDiscount {
  constructor(percentage) {
    this.percentage = percentage;
  }
  calculate(order) {
    return order.total * (this.percentage / 100);
  }
}

class FixedDiscount {
  constructor(amount) {
    this.amount = amount;
  }
  calculate(order) {
    return this.amount;
  }
}

class VIPDiscount {
  constructor(vipLevel) {
    this.vipLevel = vipLevel;
  }
  calculate(order) {
    const rates = { gold: 0.15, platinum: 0.25, diamond: 0.35 };
    return order.total * (rates[this.vipLevel] || 0);
  }
}

// เพิ่ม discount type ใหม่โดยไม่ต้องแก้โค้ดเก่า
class BuyOneGetOneDiscount {
  calculate(order) {
    const cheapestItem = Math.min(...order.items.map(i => i.price));
    return cheapestItem;
  }
}

function applyDiscount(order, discountStrategy) {
  return discountStrategy.calculate(order);
}

// ใช้งาน
const order = { total: 1000, items: [{ price: 200 }, { price: 300 }] };
console.log(applyDiscount(order, new PercentageDiscount(10)));  // 100
console.log(applyDiscount(order, new FixedDiscount(50)));       // 50
console.log(applyDiscount(order, new VIPDiscount('gold')));     // 150
```

```javascript
// L - Liskov Substitution Principle (LSP)
// Subclass ต้องใช้แทน parent class ได้

// ❌ ผิด - Square ทำให้ Rectangle มีพฤติกรรมผิด
class Rectangle {
  constructor(width, height) {
    this.width = width;
    this.height = height;
  }
  setWidth(w) { this.width = w; }
  setHeight(h) { this.height = h; }
  getArea() { return this.width * this.height; }
}

class BadSquare extends Rectangle {
  setWidth(w) {
    this.width = w;
    this.height = w; // ทำให้ behavior เปลี่ยน!
  }
  setHeight(h) {
    this.height = h;
    this.width = h; // ทำให้ behavior เปลี่ยน!
  }
}

// ✅ ถูก - ใช้ interface แยก
class Shape {
  getArea() { throw new Error('Must implement'); }
  getPerimeter() { throw new Error('Must implement'); }
}

class Rectangle extends Shape {
  constructor(width, height) {
    super();
    this.width = width;
    this.height = height;
  }
  getArea() { return this.width * this.height; }
  getPerimeter() { return 2 * (this.width + this.height); }
}

class Square extends Shape {
  constructor(side) {
    super();
    this.side = side;
  }
  getArea() { return this.side ** 2; }
  getPerimeter() { return 4 * this.side; }
}

// ฟังก์ชันนี้ทำงานได้กับทุก Shape
function printShapeInfo(shape) {
  console.log(`Area: ${shape.getArea()}`);
  console.log(`Perimeter: ${shape.getPerimeter()}`);
}

printShapeInfo(new Rectangle(4, 5)); // Area: 20, Perimeter: 18
printShapeInfo(new Square(4));       // Area: 16, Perimeter: 16
```

```javascript
// I - Interface Segregation Principle (ISP)
// ไม่บังคับ implement methods ที่ไม่ใช้

// ❌ ผิด - Interface ใหญ่เกินไป
class BadWorker {
  work() { throw new Error('Must implement'); }
  eat() { throw new Error('Must implement'); }
  sleep() { throw new Error('Must implement'); }
}

class Robot extends BadWorker {
  work() { console.log('Working...'); }
  eat() { throw new Error('Robot ไม่กินข้าว!'); } // ปัญหา!
  sleep() { throw new Error('Robot ไม่นอน!'); }    // ปัญหา!
}

// ✅ ถูก - แยก interfaces
class Workable {
  work() { throw new Error('Must implement'); }
}

class Eatable {
  eat() { throw new Error('Must implement'); }
}

class Sleepable {
  sleep() { throw new Error('Must implement'); }
}

// Mixin pattern สำหรับ JavaScript
const WorkableMixin = (Base) => class extends Base {
  work() { throw new Error('Must implement work'); }
};

const EatableMixin = (Base) => class extends Base {
  eat() { throw new Error('Must implement eat'); }
};

class HumanWorker extends EatableMixin(WorkableMixin(class {})) {
  work() { console.log('Human working...'); }
  eat() { console.log('Human eating...'); }
}

class RobotWorker extends WorkableMixin(class {}) {
  work() { console.log('Robot working...'); }
}
```

```javascript
// D - Dependency Inversion Principle (DIP)
// Depend on abstractions, not concretions

// ❌ ผิด - high-level module ขึ้นกับ low-level module
class BadOrderService {
  constructor() {
    // ขึ้นกับ concrete implementation
    this.db = new MySQLDatabase();
    this.emailService = new SendGridService();
  }

  async createOrder(data) {
    await this.db.save(data);
    await this.emailService.send(data.email, 'Order created');
  }
}

// ✅ ถูก - inject abstractions
class OrderService {
  // รับ abstractions ผ่าน constructor
  constructor(orderRepository, notificationService) {
    this.orderRepository = orderRepository; // abstraction
    this.notificationService = notificationService; // abstraction
  }

  async createOrder(data) {
    await this.orderRepository.save(data);
    await this.notificationService.notify(data.email, 'Order created');
  }
}

// ใช้งาน - สามารถเปลี่ยน implementation ได้
const prodService = new OrderService(
  new MySQLOrderRepository(),
  new SendGridNotificationService()
);

const testService = new OrderService(
  new InMemoryOrderRepository(),
  new MockNotificationService()
);
```

---

## Step 1801: Dependency Injection

```javascript
// Dependency Injection Container แบบ manual

class DIContainer {
  constructor() {
    this._bindings = new Map();
    this._singletons = new Map();
  }

  // ลงทะเบียน factory
  bind(token, factory, { singleton = false } = {}) {
    this._bindings.set(token, { factory, singleton });
    return this;
  }

  // ลงทะเบียน instance
  instance(token, value) {
    this._singletons.set(token, value);
    return this;
  }

  // สร้าง/resolve dependency
  resolve(token) {
    // ตรวจสอบ singleton ที่สร้างไว้แล้ว
    if (this._singletons.has(token)) {
      return this._singletons.get(token);
    }

    const binding = this._bindings.get(token);
    if (!binding) {
      throw new Error(`ไม่พบ binding สำหรับ: ${String(token)}`);
    }

    const instance = binding.factory(this);

    // เก็บ singleton
    if (binding.singleton) {
      this._singletons.set(token, instance);
    }

    return instance;
  }

  // helper สำหรับ class injection
  make(Class, ...extraArgs) {
    return new Class(this, ...extraArgs);
  }
}

// Symbols สำหรับ tokens
const TOKENS = {
  DATABASE: Symbol('DATABASE'),
  USER_REPO: Symbol('USER_REPO'),
  EMAIL_SERVICE: Symbol('EMAIL_SERVICE'),
  USER_SERVICE: Symbol('USER_SERVICE'),
  ORDER_REPO: Symbol('ORDER_REPO'),
  ORDER_SERVICE: Symbol('ORDER_SERVICE'),
};

// ตั้งค่า container
function setupContainer(config) {
  const container = new DIContainer();

  // Infrastructure
  container.bind(TOKENS.DATABASE, () => new Database(config.dbUrl), { singleton: true });
  
  container.bind(TOKENS.EMAIL_SERVICE, () =>
    new SendGridEmailService(config.sendgridKey)
  );

  // Repositories
  container.bind(TOKENS.USER_REPO, (c) =>
    new MongoUserRepository(c.resolve(TOKENS.DATABASE))
  );

  container.bind(TOKENS.ORDER_REPO, (c) =>
    new MongoOrderRepository(c.resolve(TOKENS.DATABASE))
  );

  // Services
  container.bind(TOKENS.USER_SERVICE, (c) =>
    new UserService({
      userRepository: c.resolve(TOKENS.USER_REPO),
      emailService: c.resolve(TOKENS.EMAIL_SERVICE),
    }),
    { singleton: true }
  );

  container.bind(TOKENS.ORDER_SERVICE, (c) =>
    new OrderService({
      orderRepository: c.resolve(TOKENS.ORDER_REPO),
      userRepository: c.resolve(TOKENS.USER_REPO),
      notificationService: c.resolve(TOKENS.EMAIL_SERVICE),
    }),
    { singleton: true }
  );

  return container;
}

// ใช้งาน
const container = setupContainer({
  dbUrl: process.env.DATABASE_URL,
  sendgridKey: process.env.SENDGRID_KEY,
});

const userService = container.resolve(TOKENS.USER_SERVICE);
const orderService = container.resolve(TOKENS.ORDER_SERVICE);
```

---

## Step 1802: IoC Container (tsyringe, inversify)

```javascript
// ใช้ tsyringe (TypeScript/JavaScript DI)
// npm install tsyringe reflect-metadata

// tsyringe ใช้ decorators (ต้องใช้ TypeScript หรือ Babel)
// ตัวอย่างแบบ JavaScript ล้วน

// การใช้ inversify
// npm install inversify reflect-metadata

// Alternative: awilix (ใช้ได้กับ JavaScript ล้วน)
// npm install awilix

const { createContainer, asClass, asValue, InjectionMode } = require('awilix');

// สร้าง container
const container = createContainer({
  injectionMode: InjectionMode.PROXY,
});

// Register
container.register({
  // Infrastructure
  database: asClass(Database, { lifetime: 'SINGLETON' }),
  emailService: asClass(SendGridEmailService, { lifetime: 'SCOPED' }),
  
  // Repositories
  userRepository: asClass(MongoUserRepository, { lifetime: 'SCOPED' }),
  orderRepository: asClass(MongoOrderRepository, { lifetime: 'SCOPED' }),
  
  // Services
  userService: asClass(UserService, { lifetime: 'SCOPED' }),
  orderService: asClass(OrderService, { lifetime: 'SCOPED' }),
  
  // Config
  config: asValue({
    dbUrl: process.env.DATABASE_URL,
    apiKey: process.env.API_KEY,
  }),
});

// awilix inject ด้วย destructuring
class UserService {
  // awilix จะ inject ตาม parameter names
  constructor({ userRepository, emailService, config }) {
    this.userRepository = userRepository;
    this.emailService = emailService;
    this.config = config;
  }
}

class MongoUserRepository {
  constructor({ database }) {
    this.database = database;
  }
}

// ใช้งาน
const userService = container.resolve('userService');

// กับ Express middleware
const scopePerRequest = require('awilix-express').scopePerRequest;
app.use(scopePerRequest(container));

app.get('/users/:id', async (req, res) => {
  const { userService } = req.container.cradle; // scoped per request
  const user = await userService.findById(req.params.id);
  res.json(user);
});
```

---

## Step 1803: Feature-based Folder Structure

```
// Feature-based structure (แนะนำสำหรับ medium-large apps)

// src/
//   features/
//     auth/
//       auth.controller.js
//       auth.service.js
//       auth.repository.js
//       auth.routes.js
//       auth.middleware.js
//       auth.validator.js
//       auth.test.js
//       index.js           <- public API
//     users/
//       user.entity.js
//       user.controller.js
//       user.service.js
//       user.repository.js
//       user.routes.js
//       user.dto.js
//       user.test.js
//       index.js
//     orders/
//       order.entity.js
//       order.controller.js
//       order.service.js
//       order.repository.js
//       order.routes.js
//       order.test.js
//       index.js
//     products/
//       ...
//   shared/
//     middleware/
//       authenticate.js
//       rateLimit.js
//       errorHandler.js
//     utils/
//       validators.js
//       logger.js
//       helpers.js
//     database/
//       connection.js
//       BaseRepository.js
//     events/
//       EventBus.js
//   app.js
//   index.js
```

```javascript
// features/users/index.js - Public API
// ชั้นนอกสุดรู้จักแค่ index.js ไม่รู้รายละเอียดข้างใน
const { UserService } = require('./user.service');
const { UserController } = require('./user.controller');
const { userRoutes } = require('./user.routes');
const { UserRepository } = require('./user.repository');

module.exports = {
  UserService,
  UserController,
  userRoutes,
  UserRepository,
};

// features/orders/order.service.js
// import จาก users feature ผ่าน index.js เท่านั้น
const { UserService } = require('../users'); // ✅ ถูก
// const { UserService } = require('../users/user.service'); // ❌ ผิด
```

---

## Step 1804: Module Boundaries

```javascript
// การกำหนด module boundaries ที่ดี

// barrel exports - รวม exports ใน index.js
// features/products/index.js
module.exports = {
  // Export เฉพาะสิ่งที่ต้องการให้ภายนอกเข้าถึง
  ProductService: require('./product.service').ProductService,
  ProductController: require('./product.controller').ProductController,
  productRoutes: require('./product.routes').productRoutes,
  // ไม่ export internal implementations
};

// Avoiding circular dependencies
// Module A imports from Module B
// Module B imports from Module A -> Circular!

// แก้ไขด้วยการสร้าง shared module
// features/shared/events.js <- ทั้ง A และ B import จาก shared

// ตรวจสอบ circular dependencies
// npm install madge
// npx madge --circular src/

// ตัวอย่างการแยก boundary ที่ดี
// users module ไม่รู้จัก orders module
// orders module ใช้ user id แต่ไม่ import User entity โดยตรง

// ❌ ผิด - circular dependency
// orders/order.service.js imports from users/user.service.js
// users/user.service.js imports from orders/order.service.js

// ✅ ถูก - ใช้ events
// orders/order.service.js publishes OrderCreated event
// users/user.service.js subscribes to OrderCreated event

// Dependency Graph ที่ดี:
// Infrastructure -> Domain
// Application -> Domain
// Infrastructure -> Application (ผ่าน interfaces)
// Framework -> Application

// เครื่องมือตรวจสอบ
// ESLint plugin: eslint-plugin-import
// rules: import/no-cycle
```

---

## Step 1805: Anti-patterns to Avoid

```javascript
// 1. God Object - object ที่รู้จักทุกอย่างและทำทุกอย่าง
// ❌ ผิด
class Application {
  connectDatabase() {}
  handleHTTP() {}
  sendEmail() {}
  processPayment() {}
  generateReport() {}
  manageUsers() {}
  trackAnalytics() {}
  // ... 100 methods
}

// 2. Spaghetti Code - code ที่พันกันอย่างซับซ้อน
// ❌ ผิด
async function doEverything(req, res) {
  const db = new Database();
  const user = await db.query(`SELECT * FROM users WHERE id = ${req.params.id}`);
  if (user) {
    const orders = await db.query(`SELECT * FROM orders WHERE user_id = ${user.id}`);
    if (orders.length > 0) {
      for (const order of orders) {
        if (order.status === 'pending') {
          await db.query(`UPDATE orders SET status = 'processing' WHERE id = ${order.id}`);
          // ...more inline logic
        }
      }
    }
    res.json(user);
  } else {
    res.status(404).json({ error: 'Not found' });
  }
}

// 3. Anemic Domain Model - entities ที่ไม่มี behavior
// ❌ ผิด
class AnemicOrder {
  // เพียงแค่ data container ไม่มี logic
  constructor(id, status, items) {
    this.id = id;
    this.status = status;
    this.items = items;
  }
}

// OrderService ต้องรู้เรื่อง Order internals มากเกินไป
class AnemicOrderService {
  cancel(order) {
    if (order.status !== 'pending') { // ต้องรู้ logic นี้
      throw new Error('Cannot cancel');
    }
    order.status = 'cancelled'; // manipulate state โดยตรง
  }
}

// ✅ ถูก - Rich Domain Model
class RichOrder {
  cancel() {
    if (!this.canBeCancelled) { // logic อยู่ใน entity
      throw new Error('Cannot cancel');
    }
    this._status = 'cancelled';
  }
}

// 4. Primitive Obsession - ใช้ primitive types แทน Value Objects
// ❌ ผิด
class PrimitiveUser {
  constructor(email, phone, zipCode) {
    this.email = email;      // string - ไม่ validate
    this.phone = phone;      // string - ไม่ validate
    this.zipCode = zipCode;  // string - ไม่ validate
  }
}

// ✅ ถูก
class ProperUser {
  constructor(email, phone, address) {
    this.email = new Email(email);
    this.phone = new PhoneNumber(phone);
    this.address = new Address(address);
  }
}

// 5. Feature Envy - method ที่ใช้ data จาก object อื่นมากเกินไป
// ❌ ผิด
class OrderCalculator {
  calculateTotal(order) {
    // เข้าถึง order.items.forEach ...item.product.price... มากเกินไป
    let total = 0;
    order.items.forEach(item => {
      total += item.product.price * item.quantity;
      if (item.product.category === 'electronics') {
        total += item.product.price * 0.07; // VAT
      }
    });
    return total;
  }
}

// ✅ ถูก - logic อยู่ใน Order
class BetterOrder {
  get total() {
    return this.items.reduce((sum, item) => sum + item.subtotal, 0);
  }
}

class BetterOrderItem {
  get subtotal() {
    return this.price * this.quantity;
  }
}
```

---

## Step 1806: Code Review Checklist

```javascript
// Code Review Checklist สำหรับ Architecture

/*
## Architecture
☐ ไม่มี circular dependencies
☐ Dependencies ไหลเข้าด้านใน (domain ไม่รู้จัก infrastructure)
☐ แต่ละ module มี clear boundary
☐ ไม่ใช้ concrete implementations ใน domain layer

## Domain
☐ Entities มี validation
☐ Entities มี behavior (ไม่ใช่แค่ getters/setters)
☐ Value Objects เป็น immutable
☐ Domain events ถูก publish เมื่อ state เปลี่ยน
☐ Business rules อยู่ใน domain ไม่ใช่ services

## Application Services
☐ Use cases ชัดเจน ทำงานเดียว
☐ Error handling ครบถ้วน
☐ Transaction boundaries ถูกต้อง

## Infrastructure
☐ Implements domain interfaces
☐ ไม่มี domain logic ใน repositories
☐ Connection errors handled properly

## Testing
☐ Unit tests สำหรับ domain logic
☐ Integration tests สำหรับ repositories
☐ E2E tests สำหรับ critical flows
☐ Test coverage >= 80%

## Performance
☐ ไม่มี N+1 query problems
☐ Indexes ถูกต้อง
☐ Pagination สำหรับ list queries

## Security
☐ Input validation
☐ Authorization checks
☐ No SQL injection / NoSQL injection
*/

// ตัวอย่าง automated architecture checks
const depcheck = require('dependency-cruiser');

// .dependency-cruiser.js
module.exports = {
  forbidden: [
    {
      name: 'no-circular',
      severity: 'error',
      comment: 'No circular dependencies',
      from: {},
      to: {
        circular: true,
      },
    },
    {
      name: 'domain-not-import-infrastructure',
      severity: 'error',
      comment: 'Domain layer ต้องไม่ import infrastructure',
      from: {
        path: '^src/domain',
      },
      to: {
        path: '^src/infrastructure',
      },
    },
    {
      name: 'domain-not-import-application',
      severity: 'error',
      comment: 'Domain layer ต้องไม่ import application layer',
      from: {
        path: '^src/domain',
      },
      to: {
        path: '^src/application',
      },
    },
  ],
};
```

---

## Step 1807: Practical Example - Complete Feature Implementation

```javascript
// ตัวอย่างการ implement feature สมบูรณ์ตาม Clean Architecture

// === DOMAIN LAYER ===

// domain/entities/BlogPost.js
class BlogPost {
  constructor({ id, title, content, authorId, tags = [], publishedAt = null }) {
    this._id = id;
    this._title = title;
    this._content = content;
    this._authorId = authorId;
    this._tags = tags;
    this._publishedAt = publishedAt;
    this._status = publishedAt ? 'PUBLISHED' : 'DRAFT';
    this._createdAt = new Date();
    this._domainEvents = [];
    this._validate();
  }

  get id() { return this._id; }
  get title() { return this._title; }
  get content() { return this._content; }
  get authorId() { return this._authorId; }
  get tags() { return [...this._tags]; }
  get status() { return this._status; }
  get isDraft() { return this._status === 'DRAFT'; }
  get isPublished() { return this._status === 'PUBLISHED'; }
  get domainEvents() { return [...this._domainEvents]; }

  _validate() {
    if (!this._title || this._title.trim().length < 3) {
      throw new Error('หัวข้อต้องมีอย่างน้อย 3 ตัวอักษร');
    }
    if (!this._content || this._content.trim().length < 10) {
      throw new Error('เนื้อหาต้องมีอย่างน้อย 10 ตัวอักษร');
    }
    if (!this._authorId) {
      throw new Error('ต้องระบุ author');
    }
  }

  publish() {
    if (this.isPublished) throw new Error('โพสต์ถูก publish แล้ว');
    this._status = 'PUBLISHED';
    this._publishedAt = new Date();
    this._domainEvents.push({
      type: 'BLOG_POST_PUBLISHED',
      payload: { postId: this._id, authorId: this._authorId },
    });
    return this;
  }

  unpublish() {
    if (!this.isPublished) throw new Error('โพสต์ยังไม่ได้ publish');
    this._status = 'DRAFT';
    this._publishedAt = null;
    return this;
  }

  update({ title, content, tags }) {
    if (title !== undefined) {
      if (title.trim().length < 3) throw new Error('หัวข้อสั้นเกินไป');
      this._title = title;
    }
    if (content !== undefined) this._content = content;
    if (tags !== undefined) this._tags = tags;
    return this;
  }

  clearEvents() { this._domainEvents = []; }
}

// domain/repositories/IBlogPostRepository.js
class IBlogPostRepository {
  async findById(id) { throw new Error('Must implement'); }
  async findByAuthor(authorId, options) { throw new Error('Must implement'); }
  async findPublished(options) { throw new Error('Must implement'); }
  async save(post) { throw new Error('Must implement'); }
  async delete(id) { throw new Error('Must implement'); }
}

// === APPLICATION LAYER ===

// application/use-cases/CreateBlogPost.js
class CreateBlogPostUseCase {
  constructor({ blogPostRepository, eventBus }) {
    this.blogPostRepository = blogPostRepository;
    this.eventBus = eventBus;
  }

  async execute({ title, content, authorId, tags, publish = false }) {
    const post = new BlogPost({
      id: generateId(),
      title,
      content,
      authorId,
      tags,
    });

    if (publish) {
      post.publish();
    }

    await this.blogPostRepository.save(post);

    // Publish domain events
    for (const event of post.domainEvents) {
      await this.eventBus.publish(event);
    }
    post.clearEvents();

    return post;
  }
}

// application/use-cases/PublishBlogPost.js
class PublishBlogPostUseCase {
  constructor({ blogPostRepository, userRepository, eventBus }) {
    this.blogPostRepository = blogPostRepository;
    this.userRepository = userRepository;
    this.eventBus = eventBus;
  }

  async execute({ postId, requesterId }) {
    const post = await this.blogPostRepository.findById(postId);
    if (!post) throw new Error('ไม่พบโพสต์');

    // Authorization
    if (post.authorId !== requesterId) {
      const requester = await this.userRepository.findById(requesterId);
      if (!requester.isAdmin) {
        throw new Error('ไม่มีสิทธิ์ publish โพสต์นี้');
      }
    }

    post.publish();
    await this.blogPostRepository.save(post);

    for (const event of post.domainEvents) {
      await this.eventBus.publish(event);
    }
    post.clearEvents();

    return post;
  }
}

// === INFRASTRUCTURE LAYER ===

// infrastructure/repositories/MongoBlogPostRepository.js
const mongoose = require('mongoose');

const BlogPostSchema = new mongoose.Schema({
  title: { type: String, required: true, index: true },
  content: { type: String, required: true },
  authorId: { type: mongoose.Types.ObjectId, ref: 'User', index: true },
  tags: [{ type: String }],
  status: { type: String, enum: ['DRAFT', 'PUBLISHED'], default: 'DRAFT' },
  publishedAt: { type: Date, default: null },
}, { timestamps: true });

// Full-text search index
BlogPostSchema.index({ title: 'text', content: 'text' });

const BlogPostModel = mongoose.model('BlogPost', BlogPostSchema);

class MongoBlogPostRepository extends IBlogPostRepository {
  async findById(id) {
    const doc = await BlogPostModel.findById(id);
    return doc ? this._toDomain(doc) : null;
  }

  async findByAuthor(authorId, { page = 1, limit = 10, status } = {}) {
    const filter = { authorId };
    if (status) filter.status = status;

    const [docs, total] = await Promise.all([
      BlogPostModel.find(filter)
        .sort({ createdAt: -1 })
        .skip((page - 1) * limit)
        .limit(limit),
      BlogPostModel.countDocuments(filter),
    ]);

    return {
      posts: docs.map(d => this._toDomain(d)),
      total,
      page,
      totalPages: Math.ceil(total / limit),
    };
  }

  async findPublished({ page = 1, limit = 10, tag } = {}) {
    const filter = { status: 'PUBLISHED' };
    if (tag) filter.tags = tag;

    const [docs, total] = await Promise.all([
      BlogPostModel.find(filter)
        .sort({ publishedAt: -1 })
        .skip((page - 1) * limit)
        .limit(limit),
      BlogPostModel.countDocuments(filter),
    ]);

    return {
      posts: docs.map(d => this._toDomain(d)),
      total,
      page,
      totalPages: Math.ceil(total / limit),
    };
  }

  async save(post) {
    await BlogPostModel.findByIdAndUpdate(
      post.id,
      {
        title: post.title,
        content: post.content,
        authorId: post.authorId,
        tags: post.tags,
        status: post.status,
        publishedAt: post._publishedAt,
      },
      { upsert: true, new: true }
    );
    return post;
  }

  async delete(id) {
    await BlogPostModel.findByIdAndDelete(id);
  }

  _toDomain(doc) {
    return new BlogPost({
      id: doc._id.toString(),
      title: doc.title,
      content: doc.content,
      authorId: doc.authorId.toString(),
      tags: doc.tags,
      publishedAt: doc.publishedAt,
    });
  }
}

// === INTERFACE LAYER ===

// infrastructure/web/controllers/BlogPostController.js
class BlogPostController {
  constructor({ createBlogPostUseCase, publishBlogPostUseCase }) {
    this.createBlogPostUseCase = createBlogPostUseCase;
    this.publishBlogPostUseCase = publishBlogPostUseCase;
  }

  async create(req, res, next) {
    try {
      const { title, content, tags, publish } = req.body;
      const authorId = req.user.id;

      const post = await this.createBlogPostUseCase.execute({
        title, content, authorId, tags, publish,
      });

      res.status(201).json({
        success: true,
        data: post.toJSON(),
      });
    } catch (error) {
      next(error);
    }
  }

  async publish(req, res, next) {
    try {
      const post = await this.publishBlogPostUseCase.execute({
        postId: req.params.id,
        requesterId: req.user.id,
      });

      res.json({
        success: true,
        data: post.toJSON(),
      });
    } catch (error) {
      next(error);
    }
  }
}
```

---

## Step 1808: Testing Architecture

```javascript
// การทดสอบ Clean Architecture
// Unit tests ทดสอบ domain layer โดยไม่ต้องการ infrastructure

// tests/domain/BlogPost.test.js
describe('BlogPost', () => {
  describe('constructor', () => {
    it('ควรสร้าง BlogPost ที่ valid ได้', () => {
      const post = new BlogPost({
        id: '1',
        title: 'หัวข้อทดสอบ',
        content: 'เนื้อหาทดสอบอย่างน้อย 10 ตัวอักษร',
        authorId: 'author-1',
      });
      expect(post.status).toBe('DRAFT');
      expect(post.isDraft).toBe(true);
    });

    it('ควร throw error เมื่อหัวข้อสั้นเกินไป', () => {
      expect(() => new BlogPost({
        id: '1',
        title: 'AB', // สั้นเกินไป
        content: 'เนื้อหาทดสอบ',
        authorId: 'author-1',
      })).toThrow('หัวข้อต้องมีอย่างน้อย 3 ตัวอักษร');
    });
  });

  describe('publish', () => {
    it('ควร publish post ได้', () => {
      const post = new BlogPost({
        id: '1',
        title: 'หัวข้อทดสอบ',
        content: 'เนื้อหาทดสอบอย่างน้อย 10 ตัวอักษร',
        authorId: 'author-1',
      });

      post.publish();
      expect(post.status).toBe('PUBLISHED');
      expect(post.isPublished).toBe(true);
    });

    it('ควร emit BLOG_POST_PUBLISHED event', () => {
      const post = new BlogPost({
        id: '1',
        title: 'หัวข้อทดสอบ',
        content: 'เนื้อหาทดสอบอย่างน้อย 10 ตัวอักษร',
        authorId: 'author-1',
      });

      post.publish();
      
      expect(post.domainEvents).toHaveLength(1);
      expect(post.domainEvents[0].type).toBe('BLOG_POST_PUBLISHED');
    });

    it('ควร throw error เมื่อ publish post ที่ published แล้ว', () => {
      const post = new BlogPost({
        id: '1',
        title: 'หัวข้อทดสอบ',
        content: 'เนื้อหาทดสอบอย่างน้อย 10 ตัวอักษร',
        authorId: 'author-1',
        publishedAt: new Date(),
      });

      expect(() => post.publish()).toThrow('โพสต์ถูก publish แล้ว');
    });
  });
});

// tests/application/CreateBlogPost.test.js
describe('CreateBlogPostUseCase', () => {
  let useCase;
  let mockRepository;
  let mockEventBus;

  beforeEach(() => {
    mockRepository = {
      save: jest.fn().mockResolvedValue(undefined),
      findById: jest.fn(),
    };
    
    mockEventBus = {
      publish: jest.fn().mockResolvedValue(undefined),
    };

    useCase = new CreateBlogPostUseCase({
      blogPostRepository: mockRepository,
      eventBus: mockEventBus,
    });
  });

  it('ควรสร้าง blog post ได้', async () => {
    const result = await useCase.execute({
      title: 'หัวข้อทดสอบ',
      content: 'เนื้อหาทดสอบอย่างน้อย 10 ตัวอักษร',
      authorId: 'author-1',
      tags: ['test'],
    });

    expect(result.status).toBe('DRAFT');
    expect(mockRepository.save).toHaveBeenCalledTimes(1);
  });

  it('ควร publish post เมื่อ publish = true', async () => {
    const result = await useCase.execute({
      title: 'หัวข้อทดสอบ',
      content: 'เนื้อหาทดสอบอย่างน้อย 10 ตัวอักษร',
      authorId: 'author-1',
      publish: true,
    });

    expect(result.status).toBe('PUBLISHED');
    expect(mockEventBus.publish).toHaveBeenCalledTimes(1);
  });
});
```

---

## Step 1809: Performance Considerations in Architecture

```javascript
// Lazy Loading Repositories
class LazyUserRepository {
  constructor(factory) {
    this._factory = factory;
    this._instance = null;
  }

  get instance() {
    if (!this._instance) {
      this._instance = this._factory();
    }
    return this._instance;
  }

  async findById(id) {
    return this.instance.findById(id);
  }
}

// Read Model (CQRS pattern)
// แยก Read และ Write models
class UserWriteModel {
  // ใช้สำหรับ Commands (create, update, delete)
  constructor(db) { this.db = db; }
  
  async save(user) {
    await this.db.users.insertOne(user.toDocument());
  }
}

class UserReadModel {
  // ใช้สำหรับ Queries (read operations)
  // อาจ optimize แยกต่างหาก (caching, denormalized data)
  constructor(db, cache) {
    this.db = db;
    this.cache = cache;
  }

  async findById(id) {
    const cacheKey = `user:${id}`;
    const cached = await this.cache.get(cacheKey);
    if (cached) return JSON.parse(cached);

    const user = await this.db.users.findOne({ _id: id });
    if (user) {
      await this.cache.setex(cacheKey, 300, JSON.stringify(user));
    }
    return user;
  }

  async findWithOrderStats(userId) {
    // Denormalized query สำหรับ performance
    const result = await this.db.users.aggregate([
      { $match: { _id: userId } },
      {
        $lookup: {
          from: 'orders',
          localField: '_id',
          foreignField: 'userId',
          as: 'orders',
        },
      },
      {
        $addFields: {
          totalOrders: { $size: '$orders' },
          totalSpent: { $sum: '$orders.totalAmount' },
        },
      },
      { $project: { orders: 0 } },
    ]).next();
    return result;
  }
}
```

---

## Step 1810: Summary and Best Practices

```javascript
// สรุป Best Practices

// 1. ใช้ Clean Architecture เพื่อแยก concerns
//    - Domain layer: business logic
//    - Application layer: use cases
//    - Infrastructure layer: technical details
//    - Interface layer: UI, API

// 2. Value Objects สำหรับ domain concepts
class PostTitle {
  constructor(value) {
    if (!value || value.trim().length < 3 || value.length > 200) {
      throw new Error('หัวข้อต้องมี 3-200 ตัวอักษร');
    }
    this._value = value.trim();
    Object.freeze(this);
  }
  get value() { return this._value; }
  toString() { return this._value; }
}

// 3. Domain Events สำหรับ side effects
// 4. Repository Pattern สำหรับ data access
// 5. Dependency Injection สำหรับ flexibility

// 6. Testing Pyramid
//    - Unit tests: test domain logic (fast, many)
//    - Integration tests: test repositories (medium)
//    - E2E tests: test API flows (slow, few)

// 7. SOLID ทุก principle มีความสำคัญ
// 8. Feature-based folder structure สำหรับ large apps

// Project Checklist
const architectureChecklist = {
  domainLayer: [
    '✅ Entities มี identity และ behavior',
    '✅ Value Objects เป็น immutable',
    '✅ Aggregates มี clear boundaries',
    '✅ Domain Events ถูก publish',
  ],
  applicationLayer: [
    '✅ Use cases มีความรับผิดชอบเดียว',
    '✅ Orchestrates domain objects',
    '✅ ไม่มี infrastructure details',
  ],
  infrastructureLayer: [
    '✅ Implements domain interfaces',
    '✅ Error handling ครบถ้วน',
  ],
  testing: [
    '✅ Unit tests ครอบคลุม domain',
    '✅ Integration tests สำหรับ repos',
  ],
};

console.log(architectureChecklist);
```

---

## แบบฝึกหัด (Exercises)

### Exercise 1: Library Management System
สร้าง domain model สำหรับระบบห้องสมุด:
- `Book` entity (id, title, ISBN, author, copies)
- `Member` entity (id, name, email, borrowedBooks)
- `BorrowRecord` entity
- `Money` value object สำหรับค่าปรับ
- Use cases: BorrowBook, ReturnBook, CheckAvailability

### Exercise 2: Implement Repository
- สร้าง `InMemoryBookRepository` สำหรับ testing
- สร้าง `MongoBookRepository` สำหรับ production
- ทดสอบทั้งสองด้วย interface เดียวกัน

### Exercise 3: Domain Events
- เพิ่ม domain events ให้กับ Library system
  - `BookBorrowed`
  - `BookReturned`
  - `OverdueBookDetected`
- สร้าง EventBus และ handlers

### Exercise 4: Dependency Injection
- สร้าง DI Container
- Register repositories, services
- ใช้ container ใน Express routes

### Exercise 5: SOLID Refactoring
นำโค้ดต่อไปนี้มา refactor:
```javascript
class UserManager {
  constructor() {
    this.db = new PostgreSQL();
    this.mailer = new NodeMailer();
  }
  
  createUser(data) { /* validates, saves, sends email */ }
  updateUser(id, data) { /* ... */ }
  deleteUser(id) { /* ... */ }
  sendWelcomeEmail(email) { /* ... */ }
  generateReport() { /* ... */ }
  exportToCsv() { /* ... */ }
}
```

---

*จบ Part 91: Code Architecture Patterns*
*ต่อไป Part 92: Open Source Contribution*
