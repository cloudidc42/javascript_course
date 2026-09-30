# Part 62: Database - MongoDB กับ JavaScript (Steps 1211-1230)

## บทนำ

MongoDB เป็น NoSQL database ที่เก็บข้อมูลในรูปแบบ Document คล้าย JSON ทำให้เหมาะกับการพัฒนา JavaScript บทนี้จะสอนการใช้งาน MongoDB ผ่าน Mongoose ซึ่งเป็น ODM (Object Document Mapper) ที่นิยมที่สุด

---

## Step 1211: MongoDB คืออะไร?

MongoDB เป็น Document Database ที่เก็บข้อมูลเป็น BSON (Binary JSON)

```javascript
// เปรียบเทียบ SQL กับ MongoDB
// SQL:
// Database -> Table -> Row -> Column
// MongoDB:
// Database -> Collection -> Document -> Field

// ตัวอย่าง SQL Table
// users table:
// | id | name  | email            | age |
// | 1  | Alice | alice@example.com| 25  |
// | 2  | Bob   | bob@example.com  | 30  |

// ตัวอย่าง MongoDB Collection
const mongoDocument = {
  _id: "507f1f77bcf86cd799439011", // ObjectId
  name: "Alice",
  email: "alice@example.com",
  age: 25,
  // MongoDB สามารถเก็บ nested objects
  address: {
    street: "123 Main St",
    city: "Bangkok",
    country: "Thailand"
  },
  // และ arrays
  hobbies: ["reading", "coding", "photography"],
  // รวมถึง array of objects
  posts: [
    { title: "Hello World", createdAt: new Date() }
  ],
  createdAt: new Date()
};
```

### ข้อดีของ MongoDB

```javascript
// 1. Schema-less (ยืดหยุ่น)
// ไม่ต้องกำหนด schema ล่วงหน้า
// document แต่ละอันสามารถมี field ต่างกันได้

// 2. Horizontal Scaling
// รองรับ sharding (แบ่งข้อมูลหลาย server)

// 3. Aggregation Pipeline
// วิเคราะห์ข้อมูลซับซ้อนได้

// 4. Full-text Search
// มี text index ในตัว

// 5. Geospatial Queries
// รองรับ location-based queries

// เมื่อไรควรใช้ MongoDB
// - ข้อมูลมี schema ที่เปลี่ยนแปลงบ่อย
// - ข้อมูลมี nested structure ซับซ้อน
// - ต้องการ scale แบบ horizontal
// - ข้อมูลไม่มี relationships ซับซ้อน

// เมื่อไรควรใช้ SQL
// - ข้อมูลมี relationships ซับซ้อน
// - ต้องการ ACID transactions อย่างเต็มรูปแบบ
// - ข้อมูลมี schema ที่ชัดเจนและไม่เปลี่ยน
// - ต้องการ reporting และ analytics ซับซ้อน
```

---

## Step 1212: BSON Types

```javascript
// BSON (Binary JSON) รองรับ types มากกว่า JSON

const mongoose = require('mongoose');

// Types ที่ใช้บ่อย
const bsonTypes = {
  // String
  name: "Alice",
  
  // Number (32-bit integer, 64-bit integer, double)
  age: 25,
  salary: 50000.00,
  
  // Boolean
  isActive: true,
  
  // Date
  createdAt: new Date(),
  
  // ObjectId (12-byte identifier)
  _id: new mongoose.Types.ObjectId(),
  
  // Array
  tags: ["javascript", "nodejs"],
  
  // Embedded Document
  address: {
    city: "Bangkok"
  },
  
  // Null
  middleName: null,
  
  // Binary Data
  // profileImage: new mongoose.Types.Buffer(imageBuffer),
  
  // Regular Expression
  pattern: /hello/i,
  
  // Decimal128 (สำหรับตัวเลขที่ต้องการความแม่นยำสูง)
  price: mongoose.Types.Decimal128.fromString('19.99')
};

// ObjectId
const id = new mongoose.Types.ObjectId();
console.log(id.toString()); // "507f1f77bcf86cd799439011"
console.log(id.getTimestamp()); // วันที่สร้าง ObjectId
```

---

## Step 1213: การติดตั้งและเชื่อมต่อ MongoDB

```javascript
// ติดตั้ง: npm install mongoose dotenv

// .env file
// MONGODB_URI=mongodb://localhost:27017/myapp
// หรือ MongoDB Atlas:
// MONGODB_URI=mongodb+srv://user:password@cluster.mongodb.net/myapp

require('dotenv').config();
const mongoose = require('mongoose');

// การเชื่อมต่อพื้นฐาน
async function connectDB() {
  try {
    await mongoose.connect(process.env.MONGODB_URI, {
      // Options สำหรับ mongoose 6+
      // ส่วนใหญ่ไม่ต้องใส่แล้ว (default ดีอยู่แล้ว)
    });
    console.log('เชื่อมต่อ MongoDB สำเร็จ');
  } catch (err) {
    console.error('เชื่อมต่อ MongoDB ล้มเหลว:', err);
    process.exit(1);
  }
}

// รับ connection events
mongoose.connection.on('connected', () => {
  console.log('Mongoose เชื่อมต่อแล้ว');
});

mongoose.connection.on('error', (err) => {
  console.error('Mongoose error:', err);
});

mongoose.connection.on('disconnected', () => {
  console.log('Mongoose ตัดการเชื่อมต่อ');
});

// ปิดการเชื่อมต่อเมื่อ app ปิด
process.on('SIGINT', async () => {
  await mongoose.connection.close();
  console.log('ปิดการเชื่อมต่อ MongoDB');
  process.exit(0);
});

connectDB();
```

### Multiple Connections

```javascript
// เชื่อมต่อหลาย database
const mongoose = require('mongoose');

const conn1 = mongoose.createConnection(process.env.DB1_URI);
const conn2 = mongoose.createConnection(process.env.DB2_URI);

// สร้าง model บน specific connection
const User = conn1.model('User', userSchema);
const Analytics = conn2.model('Analytics', analyticsSchema);
```

---

## Step 1214: Mongoose Schema

```javascript
const mongoose = require('mongoose');
const { Schema } = mongoose;

// Schema พื้นฐาน
const userSchema = new Schema({
  // String types
  name: {
    type: String,
    required: [true, 'กรุณาใส่ชื่อ'],
    trim: true,
    minlength: [2, 'ชื่อต้องมีอย่างน้อย 2 ตัวอักษร'],
    maxlength: [100, 'ชื่อยาวเกินไป']
  },
  
  email: {
    type: String,
    required: true,
    unique: true,
    lowercase: true,
    validate: {
      validator: function(v) {
        return /^\w+([.-]?\w+)*@\w+([.-]?\w+)*(\.\w{2,3})+$/.test(v);
      },
      message: 'Email ไม่ถูกต้อง'
    }
  },
  
  // Number
  age: {
    type: Number,
    min: [0, 'อายุต้องมากกว่า 0'],
    max: [150, 'อายุไม่ถูกต้อง']
  },
  
  // Boolean
  isActive: {
    type: Boolean,
    default: true
  },
  
  // Enum
  role: {
    type: String,
    enum: {
      values: ['user', 'admin', 'moderator'],
      message: 'Role ไม่ถูกต้อง'
    },
    default: 'user'
  },
  
  // Date
  birthday: Date,
  
  // Array of strings
  hobbies: [String],
  
  // Array of objects
  addresses: [{
    street: String,
    city: String,
    country: String,
    isDefault: { type: Boolean, default: false }
  }],
  
  // Nested object
  social: {
    facebook: String,
    twitter: String,
    instagram: String
  },
  
  // Reference to another model
  createdBy: {
    type: Schema.Types.ObjectId,
    ref: 'User'
  }
}, {
  // Schema options
  timestamps: true, // เพิ่ม createdAt, updatedAt อัตโนมัติ
  versionKey: '__v', // เปลี่ยนชื่อ version key
  
  // Custom toJSON
  toJSON: {
    virtuals: true,
    transform: function(doc, ret) {
      delete ret.__v;
      return ret;
    }
  }
});
```

---

## Step 1215: Schema Types และ Validators

```javascript
const mongoose = require('mongoose');
const { Schema } = mongoose;

// Custom Validators
const productSchema = new Schema({
  name: {
    type: String,
    required: true
  },
  
  price: {
    type: Number,
    required: true,
    validate: {
      validator: function(v) {
        return v > 0;
      },
      message: 'ราคาต้องมากกว่า 0'
    }
  },
  
  // Async validator
  sku: {
    type: String,
    validate: {
      validator: async function(v) {
        const count = await Product.countDocuments({ sku: v });
        return count === 0;
      },
      message: 'SKU นี้มีอยู่แล้ว'
    }
  },
  
  // หลาย validators
  discount: {
    type: Number,
    validate: [
      {
        validator: (v) => v >= 0,
        message: 'ส่วนลดต้องไม่ติดลบ'
      },
      {
        validator: (v) => v <= 100,
        message: 'ส่วนลดต้องไม่เกิน 100%'
      }
    ]
  }
});

// Virtual Fields (ไม่ถูกเก็บใน DB)
userSchema.virtual('fullName').get(function() {
  return `${this.firstName} ${this.lastName}`;
});

userSchema.virtual('age').get(function() {
  if (!this.birthday) return null;
  const today = new Date();
  const birth = new Date(this.birthday);
  return today.getFullYear() - birth.getFullYear();
});

// Instance Methods
userSchema.methods.toSafeObject = function() {
  const user = this.toObject();
  delete user.password;
  delete user.refreshTokens;
  return user;
};

userSchema.methods.hasPermission = function(permission) {
  return this.permissions.includes(permission);
};

// Static Methods
userSchema.statics.findByEmail = function(email) {
  return this.findOne({ email: email.toLowerCase() });
};

userSchema.statics.findActiveUsers = function() {
  return this.find({ isActive: true });
};

const User = mongoose.model('User', userSchema);
```

---

## Step 1216: CRUD - Create

```javascript
const mongoose = require('mongoose');

const User = mongoose.model('User', userSchema);

// 1. new + save()
async function createUser1() {
  const user = new User({
    name: 'Alice',
    email: 'alice@example.com',
    role: 'user'
  });
  
  const savedUser = await user.save();
  console.log('สร้าง user:', savedUser._id);
  return savedUser;
}

// 2. Model.create()
async function createUser2() {
  const user = await User.create({
    name: 'Bob',
    email: 'bob@example.com',
    role: 'user'
  });
  console.log('สร้าง user:', user._id);
  return user;
}

// 3. insertMany()
async function createMultipleUsers() {
  const users = await User.insertMany([
    { name: 'Charlie', email: 'charlie@example.com' },
    { name: 'Diana', email: 'diana@example.com' },
    { name: 'Eve', email: 'eve@example.com' }
  ], { ordered: false }); // ordered: false จะไม่หยุดเมื่อมี error
  
  console.log(`สร้าง ${users.length} users`);
  return users;
}

// จัดการ errors
async function createUserWithValidation() {
  try {
    const user = await User.create({
      name: 'A', // สั้นเกินไป
      email: 'invalid-email',
    });
  } catch (err) {
    if (err.name === 'ValidationError') {
      const errors = Object.values(err.errors).map(e => e.message);
      console.error('Validation errors:', errors);
    } else if (err.code === 11000) {
      // Duplicate key error
      console.error('Email นี้มีอยู่แล้ว');
    } else {
      throw err;
    }
  }
}
```

---

## Step 1217: CRUD - Read

```javascript
// find() - ค้นหาหลาย documents
async function findUsers() {
  // หาทุก user
  const allUsers = await User.find();
  
  // หาด้วย condition
  const activeUsers = await User.find({ isActive: true });
  
  // หาแบบ complex
  const adminUsers = await User.find({
    role: 'admin',
    isActive: true
  });
  
  return adminUsers;
}

// findOne() - ค้นหาเอกสารแรกที่ตรง
async function findOneUser() {
  const user = await User.findOne({ email: 'alice@example.com' });
  if (!user) {
    console.log('ไม่พบ user');
    return null;
  }
  return user;
}

// findById() - ค้นหาด้วย ID
async function findUserById(id) {
  try {
    const user = await User.findById(id);
    if (!user) {
      throw new Error('ไม่พบ user');
    }
    return user;
  } catch (err) {
    if (err.name === 'CastError') {
      throw new Error('ID ไม่ถูกต้อง');
    }
    throw err;
  }
}

// Projection - เลือก fields ที่ต้องการ
async function findWithProjection() {
  // เลือกเฉพาะบาง fields
  const users = await User.find({}, 'name email role'); // include
  
  // ไม่เอาบาง fields
  const usersNoPassword = await User.find({}, { password: 0, __v: 0 }); // exclude
  
  // แบบ object
  const usersSelect = await User.find().select('name email -_id');
  
  return users;
}

// Count Documents
async function countUsers() {
  const total = await User.countDocuments();
  const active = await User.countDocuments({ isActive: true });
  console.log(`ทั้งหมด: ${total}, Active: ${active}`);
  return { total, active };
}
```

---

## Step 1218: CRUD - Update

```javascript
// updateOne() - อัพเดตเอกสารแรกที่ตรง
async function updateOneUser(email, newData) {
  const result = await User.updateOne(
    { email },          // filter
    { $set: newData }   // update
  );
  
  console.log(`อัพเดต ${result.modifiedCount} documents`);
  return result;
}

// updateMany() - อัพเดตหลาย documents
async function deactivateOldUsers() {
  const threeMonthsAgo = new Date();
  threeMonthsAgo.setMonth(threeMonthsAgo.getMonth() - 3);
  
  const result = await User.updateMany(
    { lastLoginAt: { $lt: threeMonthsAgo } },
    { $set: { isActive: false } }
  );
  
  console.log(`ปิดการใช้งาน ${result.modifiedCount} users`);
}

// findByIdAndUpdate() - หาและอัพเดต ส่งค่ากลับ
async function updateUserById(id, newData) {
  const user = await User.findByIdAndUpdate(
    id,
    { $set: newData },
    {
      new: true,      // ส่งเอกสารที่อัพเดตแล้วกลับ
      runValidators: true  // run validators
    }
  );
  
  if (!user) {
    throw new Error('ไม่พบ user');
  }
  
  return user;
}

// findOneAndUpdate()
async function findAndUpdate() {
  const user = await User.findOneAndUpdate(
    { email: 'alice@example.com' },
    { $set: { name: 'Alice Smith' } },
    { new: true }
  );
  return user;
}

// Update Operators
async function demonstrateUpdateOperators(userId) {
  // $set - กำหนดค่า
  await User.updateOne({ _id: userId }, { $set: { name: 'New Name' } });
  
  // $unset - ลบ field
  await User.updateOne({ _id: userId }, { $unset: { temporaryField: '' } });
  
  // $inc - เพิ่มตัวเลข
  await User.updateOne({ _id: userId }, { $inc: { loginCount: 1 } });
  
  // $push - เพิ่มใน array
  await User.updateOne({ _id: userId }, { $push: { hobbies: 'gaming' } });
  
  // $pull - ลบออกจาก array
  await User.updateOne({ _id: userId }, { $pull: { hobbies: 'gaming' } });
  
  // $addToSet - เพิ่มใน array (ไม่ซ้ำ)
  await User.updateOne({ _id: userId }, { $addToSet: { tags: 'javascript' } });
  
  // $push กับ $each
  await User.updateOne({ _id: userId }, {
    $push: {
      hobbies: {
        $each: ['cooking', 'traveling'],
        $slice: 5 // เก็บแค่ 5 อัน
      }
    }
  });
}
```

---

## Step 1219: CRUD - Delete

```javascript
// deleteOne()
async function deleteOneUser(email) {
  const result = await User.deleteOne({ email });
  
  if (result.deletedCount === 0) {
    throw new Error('ไม่พบ user ที่ต้องการลบ');
  }
  
  console.log('ลบ user สำเร็จ');
  return result;
}

// deleteMany()
async function deleteInactiveUsers() {
  const result = await User.deleteMany({ isActive: false });
  console.log(`ลบ ${result.deletedCount} users`);
  return result;
}

// findByIdAndDelete() - หาและลบ ส่งเอกสารที่ลบกลับ
async function deleteUserById(id) {
  const user = await User.findByIdAndDelete(id);
  
  if (!user) {
    throw new Error('ไม่พบ user');
  }
  
  console.log('ลบ user:', user.name);
  return user;
}

// Soft Delete (แนะนำกว่าการลบจริง)
const userSchemaWithSoftDelete = new mongoose.Schema({
  name: String,
  email: String,
  deletedAt: {
    type: Date,
    default: null
  },
  isDeleted: {
    type: Boolean,
    default: false
  }
});

async function softDeleteUser(userId) {
  await User.findByIdAndUpdate(userId, {
    isDeleted: true,
    deletedAt: new Date()
  });
}

// ใน find ต้องกรองออกเสมอ
async function findActiveUsersOnly() {
  return User.find({ isDeleted: false });
}
```

---

## Step 1220: Query Operators

```javascript
// Comparison Operators
async function queryExamples() {
  // $eq - เท่ากับ
  await User.find({ age: { $eq: 25 } });
  
  // $ne - ไม่เท่ากับ
  await User.find({ role: { $ne: 'admin' } });
  
  // $gt, $gte - มากกว่า, มากกว่าหรือเท่ากับ
  await User.find({ age: { $gt: 18 } });
  await User.find({ age: { $gte: 18 } });
  
  // $lt, $lte - น้อยกว่า, น้อยกว่าหรือเท่ากับ
  await User.find({ age: { $lt: 65 } });
  await User.find({ age: { $lte: 65 } });
  
  // $in - อยู่ใน array
  await User.find({ role: { $in: ['admin', 'moderator'] } });
  
  // $nin - ไม่อยู่ใน array
  await User.find({ role: { $nin: ['guest', 'banned'] } });
  
  // Range
  await User.find({ age: { $gte: 18, $lte: 65 } });
}

// Logical Operators
async function logicalQueryExamples() {
  // $and - ทุก condition ต้องเป็นจริง
  await User.find({
    $and: [
      { age: { $gte: 18 } },
      { isActive: true }
    ]
  });
  
  // $or - อย่างน้อย 1 condition เป็นจริง
  await User.find({
    $or: [
      { role: 'admin' },
      { role: 'moderator' }
    ]
  });
  
  // $nor - ทุก condition ต้องเป็นเท็จ
  await User.find({
    $nor: [
      { isDeleted: true },
      { isBanned: true }
    ]
  });
  
  // $not
  await User.find({ age: { $not: { $lt: 18 } } });
}

// Element Operators
async function elementQueryExamples() {
  // $exists - field มีอยู่
  await User.find({ phone: { $exists: true } });
  await User.find({ deletedAt: { $exists: false } });
  
  // $type - ประเภทของ field
  await User.find({ age: { $type: 'number' } });
}

// Array Operators
async function arrayQueryExamples() {
  // หา documents ที่มี element ใน array
  await User.find({ hobbies: 'coding' });
  
  // $all - มีทุก elements
  await User.find({ hobbies: { $all: ['coding', 'reading'] } });
  
  // $size - ขนาดของ array
  await User.find({ hobbies: { $size: 3 } });
  
  // $elemMatch - element ตรงกับ condition
  await User.find({
    addresses: {
      $elemMatch: {
        city: 'Bangkok',
        isDefault: true
      }
    }
  });
}

// Text Search
async function textSearchExample() {
  // ต้องสร้าง text index ก่อน
  // userSchema.index({ name: 'text', bio: 'text' });
  
  const users = await User.find({
    $text: { $search: 'developer javascript' }
  }, {
    score: { $meta: 'textScore' }
  }).sort({
    score: { $meta: 'textScore' }
  });
  
  return users;
}
```

---

## Step 1221: Sorting และ Pagination

```javascript
// Sorting
async function sortingExamples() {
  // เรียงตาม field เดียว
  const usersByName = await User.find().sort({ name: 1 }); // 1 = ascending
  const usersByAgeDesc = await User.find().sort({ age: -1 }); // -1 = descending
  
  // เรียงหลาย fields
  const sortedUsers = await User.find()
    .sort({ role: 1, name: 1 });
  
  // แบบ string
  const users = await User.find().sort('name -createdAt');
  
  return users;
}

// Pagination ด้วย skip/limit
async function paginateUsers(page = 1, limit = 10) {
  const skip = (page - 1) * limit;
  
  const [users, total] = await Promise.all([
    User.find()
      .skip(skip)
      .limit(limit)
      .sort({ createdAt: -1 }),
    User.countDocuments()
  ]);
  
  return {
    users,
    pagination: {
      page,
      limit,
      total,
      totalPages: Math.ceil(total / limit),
      hasMore: page < Math.ceil(total / limit)
    }
  };
}

// Cursor-based Pagination (ดีกว่า skip สำหรับ large dataset)
async function cursorPaginate(lastId, limit = 10) {
  const query = lastId
    ? { _id: { $gt: lastId } }
    : {};
  
  const users = await User.find(query)
    .limit(limit + 1) // +1 เพื่อเช็คว่ามีหน้าถัดไปหรือไม่
    .sort({ _id: 1 });
  
  const hasMore = users.length > limit;
  const items = hasMore ? users.slice(0, limit) : users;
  const nextCursor = hasMore ? items[items.length - 1]._id : null;
  
  return { items, hasMore, nextCursor };
}

// Query Builder Pattern
async function buildQuery(filters) {
  let query = User.find();
  
  if (filters.name) {
    query = query.where('name').regex(new RegExp(filters.name, 'i'));
  }
  
  if (filters.role) {
    query = query.where('role').equals(filters.role);
  }
  
  if (filters.minAge) {
    query = query.where('age').gte(parseInt(filters.minAge));
  }
  
  if (filters.isActive !== undefined) {
    query = query.where('isActive').equals(filters.isActive === 'true');
  }
  
  // Sorting
  const sortField = filters.sortBy || 'createdAt';
  const sortOrder = filters.sortOrder === 'asc' ? 1 : -1;
  query = query.sort({ [sortField]: sortOrder });
  
  // Pagination
  const page = parseInt(filters.page) || 1;
  const limit = parseInt(filters.limit) || 10;
  query = query.skip((page - 1) * limit).limit(limit);
  
  return query.exec();
}
```

---

## Step 1222: Population (References)

```javascript
const mongoose = require('mongoose');

// Schema กับ References
const postSchema = new mongoose.Schema({
  title: {
    type: String,
    required: true
  },
  content: String,
  author: {
    type: mongoose.Schema.Types.ObjectId,
    ref: 'User',  // อ้างอิงไปยัง User model
    required: true
  },
  categories: [{
    type: mongoose.Schema.Types.ObjectId,
    ref: 'Category'
  }],
  comments: [{
    text: String,
    author: {
      type: mongoose.Schema.Types.ObjectId,
      ref: 'User'
    },
    createdAt: { type: Date, default: Date.now }
  }],
  createdAt: { type: Date, default: Date.now }
});

const Post = mongoose.model('Post', postSchema);

// Basic Populate
async function getPostWithAuthor(postId) {
  const post = await Post.findById(postId)
    .populate('author');
  
  return post;
}

// Select fields from populated document
async function getPostWithPartialAuthor(postId) {
  const post = await Post.findById(postId)
    .populate('author', 'name email -_id');
  
  return post;
}

// Populate หลาย fields
async function getPostWithAll(postId) {
  const post = await Post.findById(postId)
    .populate('author', 'name email')
    .populate('categories', 'name slug');
  
  return post;
}

// Populate กับ conditions
async function getPostWithActiveAuthor(postId) {
  const post = await Post.findById(postId)
    .populate({
      path: 'author',
      match: { isActive: true },
      select: 'name email'
    });
  
  // ถ้า author ไม่ active, post.author จะเป็น null
  return post;
}

// Deep Populate (populate nested references)
async function deepPopulate(postId) {
  const post = await Post.findById(postId)
    .populate({
      path: 'comments.author',
      select: 'name'
    });
  
  return post;
}

// Virtual Populate (ไม่ต้องเก็บ reference ใน document)
userSchema.virtual('posts', {
  ref: 'Post',
  localField: '_id',
  foreignField: 'author'
});

// ใช้งาน
async function getUserWithPosts(userId) {
  const user = await User.findById(userId)
    .populate('posts', 'title createdAt');
  
  return user;
}
```

---

## Step 1223: Indexing

```javascript
const mongoose = require('mongoose');

// การสร้าง Index ใน Schema
const productSchema = new mongoose.Schema({
  name: {
    type: String,
    required: true,
    index: true // simple index
  },
  
  sku: {
    type: String,
    unique: true // unique index
  },
  
  price: Number,
  category: String,
  tags: [String],
  
  // Geospatial
  location: {
    type: { type: String, default: 'Point' },
    coordinates: [Number] // [longitude, latitude]
  },
  
  createdAt: { type: Date, default: Date.now }
});

// Compound Index
productSchema.index({ category: 1, price: 1 });

// Text Index สำหรับ full-text search
productSchema.index({ name: 'text', description: 'text' });

// Sparse Index (เฉพาะ documents ที่มี field นั้น)
productSchema.index({ deletedAt: 1 }, { sparse: true });

// TTL Index (ลบ document อัตโนมัติ)
const sessionSchema = new mongoose.Schema({
  userId: mongoose.Schema.Types.ObjectId,
  data: Object,
  createdAt: { type: Date, default: Date.now }
});

// ลบ session หลัง 24 ชั่วโมง
sessionSchema.index({ createdAt: 1 }, { expireAfterSeconds: 86400 });

// Geospatial Index
productSchema.index({ location: '2dsphere' });

// การ query ด้วย geospatial
async function findNearby(longitude, latitude, maxDistance) {
  return Product.find({
    location: {
      $near: {
        $geometry: {
          type: 'Point',
          coordinates: [longitude, latitude]
        },
        $maxDistance: maxDistance // meters
      }
    }
  });
}

// อธิบาย query plan
async function explainQuery() {
  const explanation = await User.find({ email: 'test@example.com' })
    .explain('executionStats');
  
  console.log('Index used:', explanation.queryPlanner.winningPlan);
}
```

---

## Step 1224: Aggregation Pipeline

```javascript
const mongoose = require('mongoose');

// Aggregation Pipeline คือการประมวลผลข้อมูลแบบ step-by-step

async function aggregationExamples() {
  // ตัวอย่าง 1: Count users by role
  const usersByRole = await User.aggregate([
    {
      $group: {
        _id: '$role',
        count: { $sum: 1 }
      }
    },
    {
      $sort: { count: -1 }
    }
  ]);
  
  console.log('Users by role:', usersByRole);
  
  // ตัวอย่าง 2: Total sales by month
  const salesByMonth = await Order.aggregate([
    {
      $match: {
        status: 'completed',
        createdAt: {
          $gte: new Date('2024-01-01'),
          $lt: new Date('2025-01-01')
        }
      }
    },
    {
      $group: {
        _id: {
          year: { $year: '$createdAt' },
          month: { $month: '$createdAt' }
        },
        totalSales: { $sum: '$total' },
        orderCount: { $sum: 1 }
      }
    },
    {
      $sort: { '_id.year': 1, '_id.month': 1 }
    }
  ]);
  
  return { usersByRole, salesByMonth };
}

// Pipeline Stages ที่ใช้บ่อย
async function pipelineStages() {
  const result = await Post.aggregate([
    // $match - กรอง documents
    { $match: { isPublished: true } },
    
    // $lookup - join กับ collection อื่น
    {
      $lookup: {
        from: 'users',        // collection ที่ต้อง join
        localField: 'author', // field ใน Post
        foreignField: '_id',  // field ใน User
        as: 'authorInfo'      // ชื่อ field ผลลัพธ์
      }
    },
    
    // $unwind - แตก array ออกเป็น documents
    { $unwind: '$authorInfo' },
    
    // $project - เลือก/แก้ไข fields
    {
      $project: {
        title: 1,
        content: 1,
        'authorInfo.name': 1,
        'authorInfo.email': 1,
        commentCount: { $size: '$comments' }
      }
    },
    
    // $addFields - เพิ่ม fields
    {
      $addFields: {
        titleLength: { $strLenCP: '$title' }
      }
    },
    
    // $sort
    { $sort: { createdAt: -1 } },
    
    // $skip และ $limit สำหรับ pagination
    { $skip: 0 },
    { $limit: 10 }
  ]);
  
  return result;
}

// ตัวอย่างซับซ้อน: Analytics Dashboard
async function getDashboardStats() {
  const [totalUsers, recentPosts, topAuthors] = await Promise.all([
    User.countDocuments({ isActive: true }),
    
    Post.find({ isPublished: true })
      .sort({ createdAt: -1 })
      .limit(5)
      .populate('author', 'name'),
    
    Post.aggregate([
      { $match: { isPublished: true } },
      {
        $group: {
          _id: '$author',
          postCount: { $sum: 1 },
          totalViews: { $sum: '$viewCount' }
        }
      },
      { $sort: { postCount: -1 } },
      { $limit: 5 },
      {
        $lookup: {
          from: 'users',
          localField: '_id',
          foreignField: '_id',
          as: 'authorInfo'
        }
      },
      { $unwind: '$authorInfo' },
      {
        $project: {
          'authorInfo.name': 1,
          'authorInfo.email': 1,
          postCount: 1,
          totalViews: 1
        }
      }
    ])
  ]);
  
  return { totalUsers, recentPosts, topAuthors };
}
```

---

## Step 1225: Mongoose Middleware (Hooks)

```javascript
const mongoose = require('mongoose');
const bcrypt = require('bcryptjs');

const userSchema = new mongoose.Schema({
  name: String,
  email: String,
  password: String,
  updatedAt: Date
});

// Pre save hook
userSchema.pre('save', async function(next) {
  // this = document ที่กำลังจะ save
  
  // Hash password ถ้าถูกแก้ไข
  if (this.isModified('password')) {
    this.password = await bcrypt.hash(this.password, 12);
  }
  
  // อัพเดต timestamp
  this.updatedAt = new Date();
  
  next();
});

// Pre save validation
userSchema.pre('save', function(next) {
  if (this.email && !this.email.includes('@')) {
    next(new Error('Email ไม่ถูกต้อง'));
    return;
  }
  next();
});

// Post save hook
userSchema.post('save', function(doc) {
  console.log('บันทึก user แล้ว:', doc._id);
  // ส่ง notification, อัพเดต cache, etc.
});

// Pre find
userSchema.pre(/^find/, function(next) {
  // this = query
  // กรอง deleted users ออกอัตโนมัติ
  this.where({ isDeleted: { $ne: true } });
  next();
});

// Post find
userSchema.post('find', function(docs) {
  console.log(`พบ ${docs.length} users`);
});

// Pre delete
userSchema.pre('deleteOne', { document: true }, function(next) {
  console.log('กำลังลบ user:', this._id);
  // cleanup related data
  next();
});

// Pre aggregate
userSchema.pre('aggregate', function(next) {
  // เพิ่ม match stage เพื่อกรองข้อมูล
  this.pipeline().unshift({ $match: { isDeleted: { $ne: true } } });
  next();
});

// Pre validate
userSchema.pre('validate', function(next) {
  if (this.isNew) {
    console.log('กำลัง validate document ใหม่');
  }
  next();
});

// Query Middleware
userSchema.pre('findOneAndUpdate', function(next) {
  // this = query object
  const update = this.getUpdate();
  
  // เพิ่ม updatedAt อัตโนมัติ
  if (!update.$set) update.$set = {};
  update.$set.updatedAt = new Date();
  
  next();
});
```

---

## Step 1226: Transactions

```javascript
const mongoose = require('mongoose');

// Transactions รองรับใน MongoDB 4.0+ และ Replica Set หรือ Sharded Cluster

async function transferMoney(fromId, toId, amount) {
  const session = await mongoose.startSession();
  
  try {
    session.startTransaction();
    
    // ดึงข้อมูล accounts
    const from = await Account.findById(fromId).session(session);
    const to = await Account.findById(toId).session(session);
    
    if (!from || !to) {
      throw new Error('ไม่พบ account');
    }
    
    if (from.balance < amount) {
      throw new Error('ยอดเงินไม่เพียงพอ');
    }
    
    // อัพเดต balances
    from.balance -= amount;
    to.balance += amount;
    
    await from.save({ session });
    await to.save({ session });
    
    // สร้าง transaction record
    await Transaction.create([{
      from: fromId,
      to: toId,
      amount,
      type: 'transfer',
      status: 'completed'
    }], { session });
    
    // Commit transaction
    await session.commitTransaction();
    console.log('โอนเงินสำเร็จ');
    
  } catch (err) {
    // Rollback ถ้ามี error
    await session.abortTransaction();
    console.error('โอนเงินล้มเหลว:', err.message);
    throw err;
  } finally {
    session.endSession();
  }
}

// withTransaction helper (จัดการ retry อัตโนมัติ)
async function createOrderWithTransaction(userId, items) {
  const session = await mongoose.startSession();
  
  const result = await session.withTransaction(async () => {
    // ตรวจสอบ stock
    for (const item of items) {
      const product = await Product.findById(item.productId).session(session);
      
      if (!product || product.stock < item.quantity) {
        throw new Error(`สินค้า ${item.productId} ไม่เพียงพอ`);
      }
      
      // ลด stock
      await Product.findByIdAndUpdate(
        item.productId,
        { $inc: { stock: -item.quantity } },
        { session }
      );
    }
    
    // สร้าง order
    const total = items.reduce((sum, item) => sum + item.price * item.quantity, 0);
    
    const [order] = await Order.create([{
      userId,
      items,
      total,
      status: 'pending'
    }], { session });
    
    return order;
  });
  
  session.endSession();
  return result;
}
```

---

## Step 1227: Advanced Queries

```javascript
// Lean Query (ส่ง plain object แทน Mongoose document)
async function fastQuery() {
  // lean() ทำให้ query เร็วขึ้น แต่ไม่มี methods
  const users = await User.find({ isActive: true }).lean();
  
  // users เป็น plain JavaScript objects
  // ไม่มี .save(), virtuals, etc.
  
  return users;
}

// Distinct
async function getDistinctValues() {
  const roles = await User.distinct('role');
  console.log('Roles:', roles); // ['user', 'admin', 'moderator']
  
  // Distinct กับ filter
  const activeCities = await User.distinct('address.city', { isActive: true });
  return { roles, activeCities };
}

// BulkWrite สำหรับ operations จำนวนมาก
async function bulkOperations() {
  const result = await User.bulkWrite([
    // Insert
    {
      insertOne: {
        document: { name: 'New User', email: 'new@example.com' }
      }
    },
    // Update
    {
      updateOne: {
        filter: { email: 'alice@example.com' },
        update: { $set: { name: 'Alice Smith' } }
      }
    },
    // Delete
    {
      deleteOne: {
        filter: { email: 'old@example.com' }
      }
    },
    // Replace
    {
      replaceOne: {
        filter: { _id: 'someId' },
        replacement: { name: 'Replaced User', email: 'replaced@example.com' }
      }
    }
  ]);
  
  console.log('Inserted:', result.insertedCount);
  console.log('Modified:', result.modifiedCount);
  console.log('Deleted:', result.deletedCount);
}

// Watch (Change Streams)
async function watchChanges() {
  const changeStream = User.watch();
  
  changeStream.on('change', (change) => {
    console.log('มีการเปลี่ยนแปลง:', change.operationType);
    
    if (change.operationType === 'insert') {
      console.log('เพิ่ม user ใหม่:', change.fullDocument._id);
    }
    
    if (change.operationType === 'update') {
      console.log('อัพเดต user:', change.documentKey._id);
    }
  });
  
  // ใน production ต้อง handle errors และ close stream ด้วย
}
```

---

## Step 1228: Error Handling

```javascript
// จัดการ Mongoose Errors

async function handleMongooseErrors(req, res) {
  try {
    await User.create(req.body);
    res.status(201).json({ message: 'สร้างสำเร็จ' });
  } catch (err) {
    // Validation Error
    if (err.name === 'ValidationError') {
      const errors = {};
      
      Object.keys(err.errors).forEach(key => {
        errors[key] = err.errors[key].message;
      });
      
      return res.status(400).json({
        error: 'ข้อมูลไม่ถูกต้อง',
        details: errors
      });
    }
    
    // Duplicate Key Error
    if (err.code === 11000) {
      const field = Object.keys(err.keyValue)[0];
      return res.status(400).json({
        error: `${field} นี้มีอยู่แล้ว`
      });
    }
    
    // Cast Error (ID ไม่ถูกต้อง)
    if (err.name === 'CastError') {
      return res.status(400).json({
        error: 'ID ไม่ถูกต้อง'
      });
    }
    
    // Document Not Found
    if (err.name === 'DocumentNotFoundError') {
      return res.status(404).json({
        error: 'ไม่พบข้อมูล'
      });
    }
    
    console.error('Unexpected error:', err);
    res.status(500).json({ error: 'เกิดข้อผิดพลาด' });
  }
}

// Global Error Middleware สำหรับ Mongoose
function mongooseErrorMiddleware(err, req, res, next) {
  if (err.name === 'ValidationError') {
    return res.status(400).json({
      status: 'error',
      message: 'Validation ล้มเหลว',
      errors: Object.values(err.errors).map(e => ({
        field: e.path,
        message: e.message
      }))
    });
  }
  
  if (err.code === 11000) {
    const field = Object.keys(err.keyValue)[0];
    return res.status(400).json({
      status: 'error',
      message: `ค่าของ ${field} ซ้ำกัน`
    });
  }
  
  next(err);
}
```

---

## Step 1229: Performance Optimization

```javascript
// เคล็ดลับการเพิ่มประสิทธิภาพ

// 1. ใช้ Lean Queries เมื่อไม่ต้องการ Mongoose features
async function getLeanUsers() {
  return User.find({ isActive: true })
    .select('name email')
    .lean()
    .exec();
}

// 2. Projection - เลือกเฉพาะ fields ที่ใช้
async function getMinimalData() {
  return User.find().select('name email -_id');
}

// 3. Index การ query บ่อย
// สร้าง index ใน schema:
userSchema.index({ email: 1 });
userSchema.index({ role: 1, isActive: 1 });
userSchema.index({ createdAt: -1 });

// 4. Pagination แทน load ทั้งหมด
async function getPage(page, limit) {
  return User.find()
    .skip((page - 1) * limit)
    .limit(limit)
    .lean();
}

// 5. Caching ด้วย Redis
const redis = require('redis');
const client = redis.createClient();

async function getUserWithCache(userId) {
  const cacheKey = `user:${userId}`;
  
  // ตรวจสอบ cache
  const cached = await client.get(cacheKey);
  if (cached) {
    return JSON.parse(cached);
  }
  
  // ดึงจาก database
  const user = await User.findById(userId).lean();
  
  // บันทึกใน cache (expire 5 นาที)
  await client.setEx(cacheKey, 300, JSON.stringify(user));
  
  return user;
}

// 6. Read Preference สำหรับ Replica Set
async function readFromSecondary() {
  return User.find()
    .read('secondary') // อ่านจาก secondary replica
    .lean();
}

// 7. Batch Operations
async function batchInsert(users) {
  // insertMany เร็วกว่า create หลายๆ ครั้ง
  return User.insertMany(users, { ordered: false });
}
```

---

## Step 1230: Repository Pattern

```javascript
// Repository Pattern สำหรับ clean architecture

class UserRepository {
  constructor(model) {
    this.model = model;
  }
  
  async findAll(filters = {}, options = {}) {
    const { page = 1, limit = 10, sort = '-createdAt', select } = options;
    
    let query = this.model.find(filters);
    
    if (select) query = query.select(select);
    query = query.sort(sort)
      .skip((page - 1) * limit)
      .limit(limit)
      .lean();
    
    const [items, total] = await Promise.all([
      query.exec(),
      this.model.countDocuments(filters)
    ]);
    
    return {
      items,
      pagination: {
        page,
        limit,
        total,
        totalPages: Math.ceil(total / limit)
      }
    };
  }
  
  async findById(id) {
    const item = await this.model.findById(id).lean();
    if (!item) throw new Error(`ไม่พบ ${this.model.modelName}`);
    return item;
  }
  
  async findOne(filters) {
    return this.model.findOne(filters).lean();
  }
  
  async create(data) {
    return this.model.create(data);
  }
  
  async updateById(id, data) {
    const item = await this.model.findByIdAndUpdate(
      id,
      { $set: data },
      { new: true, runValidators: true }
    );
    if (!item) throw new Error(`ไม่พบ ${this.model.modelName}`);
    return item;
  }
  
  async deleteById(id) {
    const item = await this.model.findByIdAndDelete(id);
    if (!item) throw new Error(`ไม่พบ ${this.model.modelName}`);
    return item;
  }
  
  async exists(filters) {
    const count = await this.model.countDocuments(filters);
    return count > 0;
  }
}

// การใช้งาน
const userRepo = new UserRepository(User);

// ใน service
async function getUserService(id) {
  return userRepo.findById(id);
}

async function createUserService(data) {
  const exists = await userRepo.exists({ email: data.email });
  if (exists) throw new Error('Email นี้มีอยู่แล้ว');
  return userRepo.create(data);
}
```

---

## แบบฝึกหัด

### ระดับเริ่มต้น
1. สร้าง Schema สำหรับ Product ที่มี name, price, stock, category
2. เขียน CRUD operations สำหรับ User collection
3. ทำ pagination สำหรับ Post collection

### ระดับกลาง
4. สร้าง Blog ที่มี User, Post, Comment พร้อม population
5. เขียน aggregation เพื่อหา top 10 authors ที่มี posts มากที่สุด
6. implement soft delete สำหรับ User model

### ระดับสูง
7. สร้างระบบ E-commerce ที่มี Product, Order, Payment พร้อม transactions
8. เขียน custom plugin สำหรับ Mongoose (เช่น pagination plugin)
9. implement change streams เพื่อ real-time notifications
10. สร้าง Repository pattern ครบสำหรับทุก models

---

*Part 62 จบแล้ว ต่อไปเป็น Part 63: PostgreSQL กับ JavaScript*
