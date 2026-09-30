# Part 63: Database - PostgreSQL กับ JavaScript (Steps 1231-1250)

## บทนำ

PostgreSQL เป็น Relational Database ที่ทรงพลังและรองรับ ACID transactions อย่างสมบูรณ์ บทนี้จะสอนการใช้งาน PostgreSQL ผ่าน Node.js โดยใช้ทั้ง pg (node-postgres) และ Prisma ORM

---

## Step 1231: SQL vs NoSQL

```javascript
// เปรียบเทียบ SQL (PostgreSQL) กับ NoSQL (MongoDB)

// SQL Characteristics:
// - Structured schema (ต้องกำหนดโครงสร้างล่วงหน้า)
// - ACID transactions
// - Relationships ด้วย Foreign Keys
// - Complex JOIN queries
// - Vertical scaling ปกติ
// - SQL query language

// NoSQL Characteristics:
// - Flexible schema
// - BASE transactions (Basically Available, Soft state, Eventually consistent)
// - Embedded documents หรือ References
// - Denormalized data
// - Horizontal scaling ง่าย
// - Query language ของแต่ละ database

// เมื่อไรใช้ PostgreSQL:
// - ข้อมูลมี relationships ซับซ้อน
// - ต้องการ ACID compliance เต็มรูปแบบ
// - Financial data, Inventory management
// - Complex reporting และ analytics
// - ข้อมูลที่มี schema ชัดเจน

// ตัวอย่าง Schema SQL
/*
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  email VARCHAR(255) UNIQUE NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE posts (
  id SERIAL PRIMARY KEY,
  title VARCHAR(500) NOT NULL,
  content TEXT,
  user_id INTEGER REFERENCES users(id) ON DELETE CASCADE,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
*/
```

---

## Step 1232: PostgreSQL Setup

```javascript
// ติดตั้ง: npm install pg

// connection string format:
// postgresql://username:password@host:port/database

// .env
// DATABASE_URL=postgresql://user:password@localhost:5432/myapp

const { Pool, Client } = require('pg');

// Pool - เหมาะสำหรับ web applications (แนะนำ)
const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  // หรือใส่แยก:
  host: 'localhost',
  port: 5432,
  database: 'myapp',
  user: 'postgres',
  password: 'password',
  max: 20,              // จำนวน connections สูงสุด
  idleTimeoutMillis: 30000,
  connectionTimeoutMillis: 2000,
});

// Client - เหมาะสำหรับ single operations หรือ transactions
async function singleQuery() {
  const client = new Client({
    connectionString: process.env.DATABASE_URL
  });
  
  await client.connect();
  
  try {
    const result = await client.query('SELECT NOW()');
    console.log('เวลา:', result.rows[0]);
  } finally {
    await client.end();
  }
}

// ทดสอบการเชื่อมต่อ
async function testConnection() {
  try {
    const result = await pool.query('SELECT NOW() as current_time');
    console.log('เชื่อมต่อ PostgreSQL สำเร็จ:', result.rows[0].current_time);
  } catch (err) {
    console.error('เชื่อมต่อล้มเหลว:', err.message);
    process.exit(1);
  }
}

testConnection();
```

---

## Step 1233: Basic SQL Queries

```javascript
const { pool } = require('./db');

// SELECT
async function selectExamples() {
  // SELECT ทั้งหมด
  const all = await pool.query('SELECT * FROM users');
  console.log('ทั้งหมด:', all.rows);
  
  // SELECT บาง columns
  const names = await pool.query('SELECT id, name, email FROM users');
  
  // SELECT พร้อม WHERE
  const active = await pool.query(
    'SELECT * FROM users WHERE is_active = true'
  );
  
  // SELECT พร้อม ORDER BY
  const sorted = await pool.query(
    'SELECT * FROM users ORDER BY name ASC, created_at DESC'
  );
  
  // SELECT พร้อม LIMIT และ OFFSET
  const page1 = await pool.query(
    'SELECT * FROM users ORDER BY id LIMIT 10 OFFSET 0'
  );
  
  return all.rows;
}

// INSERT
async function insertExamples() {
  // INSERT หนึ่ง row
  const result = await pool.query(
    'INSERT INTO users (name, email) VALUES ($1, $2) RETURNING *',
    ['Alice', 'alice@example.com']
  );
  console.log('สร้าง user:', result.rows[0]);
  
  return result.rows[0];
}

// UPDATE
async function updateExamples() {
  const result = await pool.query(
    'UPDATE users SET name = $1 WHERE id = $2 RETURNING *',
    ['Alice Smith', 1]
  );
  
  if (result.rowCount === 0) {
    throw new Error('ไม่พบ user');
  }
  
  return result.rows[0];
}

// DELETE
async function deleteExamples() {
  const result = await pool.query(
    'DELETE FROM users WHERE id = $1 RETURNING *',
    [1]
  );
  
  if (result.rowCount === 0) {
    throw new Error('ไม่พบ user');
  }
  
  return result.rows[0];
}
```

---

## Step 1234: Parameterized Queries (SQL Injection Prevention)

```javascript
// ❌ อันตราย! SQL Injection
async function dangerousQuery(userId) {
  // อย่าทำแบบนี้!
  const query = `SELECT * FROM users WHERE id = ${userId}`;
  // ถ้า userId = "1 OR 1=1" จะดึงข้อมูลทั้งหมด!
  return pool.query(query);
}

// ✅ ปลอดภัย - Parameterized Query
async function safeQuery(userId) {
  // ใช้ $1, $2, ... แทน values
  const query = 'SELECT * FROM users WHERE id = $1';
  const values = [userId];
  return pool.query(query, values);
}

// ตัวอย่าง CRUD พร้อม parameterized queries
const userDB = {
  // Create
  async create({ name, email, password }) {
    const query = `
      INSERT INTO users (name, email, password, created_at)
      VALUES ($1, $2, $3, NOW())
      RETURNING id, name, email, created_at
    `;
    const result = await pool.query(query, [name, email, password]);
    return result.rows[0];
  },
  
  // Read by ID
  async findById(id) {
    const result = await pool.query(
      'SELECT id, name, email, role, is_active, created_at FROM users WHERE id = $1',
      [id]
    );
    return result.rows[0] || null;
  },
  
  // Read by email
  async findByEmail(email) {
    const result = await pool.query(
      'SELECT * FROM users WHERE email = $1',
      [email]
    );
    return result.rows[0] || null;
  },
  
  // Read all with pagination
  async findAll({ page = 1, limit = 10, search = '' }) {
    const offset = (page - 1) * limit;
    const searchPattern = `%${search}%`;
    
    const [rowsResult, countResult] = await Promise.all([
      pool.query(
        `SELECT id, name, email, role, created_at 
         FROM users 
         WHERE (name ILIKE $1 OR email ILIKE $1)
         ORDER BY created_at DESC 
         LIMIT $2 OFFSET $3`,
        [searchPattern, limit, offset]
      ),
      pool.query(
        'SELECT COUNT(*) FROM users WHERE (name ILIKE $1 OR email ILIKE $1)',
        [searchPattern]
      )
    ]);
    
    return {
      users: rowsResult.rows,
      total: parseInt(countResult.rows[0].count),
      page,
      limit
    };
  },
  
  // Update
  async update(id, { name, email }) {
    const result = await pool.query(
      `UPDATE users SET name = $1, email = $2, updated_at = NOW() 
       WHERE id = $3 
       RETURNING id, name, email, updated_at`,
      [name, email, id]
    );
    
    if (result.rowCount === 0) throw new Error('ไม่พบ user');
    return result.rows[0];
  },
  
  // Delete
  async delete(id) {
    const result = await pool.query(
      'DELETE FROM users WHERE id = $1 RETURNING *',
      [id]
    );
    
    if (result.rowCount === 0) throw new Error('ไม่พบ user');
    return result.rows[0];
  }
};
```

---

## Step 1235: Transactions กับ pg

```javascript
// Transactions รับประกัน ACID:
// A - Atomicity: ทุก operation สำเร็จหรือล้มเหลวพร้อมกัน
// C - Consistency: ข้อมูลถูกต้องเสมอ
// I - Isolation: transactions แยกกัน
// D - Durability: committed data ถาวร

async function transferMoney(fromUserId, toUserId, amount) {
  const client = await pool.connect();
  
  try {
    // เริ่ม transaction
    await client.query('BEGIN');
    
    // ดึงยอดเงิน
    const fromResult = await client.query(
      'SELECT balance FROM accounts WHERE user_id = $1 FOR UPDATE',
      [fromUserId]
    );
    
    if (fromResult.rows.length === 0) {
      throw new Error('ไม่พบ account ต้นทาง');
    }
    
    const fromBalance = parseFloat(fromResult.rows[0].balance);
    
    if (fromBalance < amount) {
      throw new Error('ยอดเงินไม่เพียงพอ');
    }
    
    // หัก account ต้นทาง
    await client.query(
      'UPDATE accounts SET balance = balance - $1 WHERE user_id = $2',
      [amount, fromUserId]
    );
    
    // เพิ่ม account ปลายทาง
    await client.query(
      'UPDATE accounts SET balance = balance + $1 WHERE user_id = $2',
      [amount, toUserId]
    );
    
    // บันทึก transaction log
    await client.query(
      `INSERT INTO transactions (from_user_id, to_user_id, amount, type) 
       VALUES ($1, $2, $3, 'transfer')`,
      [fromUserId, toUserId, amount]
    );
    
    // Commit
    await client.query('COMMIT');
    console.log('โอนเงินสำเร็จ');
    
  } catch (err) {
    // Rollback
    await client.query('ROLLBACK');
    console.error('โอนเงินล้มเหลว:', err.message);
    throw err;
  } finally {
    // คืน connection กลับ pool
    client.release();
  }
}

// Savepoints
async function complexTransaction() {
  const client = await pool.connect();
  
  try {
    await client.query('BEGIN');
    
    // ทำงานที่ 1
    await client.query('INSERT INTO orders (status) VALUES ($1)', ['pending']);
    
    // สร้าง savepoint
    await client.query('SAVEPOINT before_inventory');
    
    try {
      // ทำงานที่ 2 (อาจ fail)
      await client.query(
        'UPDATE inventory SET stock = stock - 1 WHERE id = $1',
        [productId]
      );
    } catch (err) {
      // Rollback ไปยัง savepoint (ไม่ roll back ทั้งหมด)
      await client.query('ROLLBACK TO SAVEPOINT before_inventory');
      // ทำงานอื่นแทน
      await client.query(
        'INSERT INTO backorders (product_id) VALUES ($1)',
        [productId]
      );
    }
    
    await client.query('COMMIT');
  } catch (err) {
    await client.query('ROLLBACK');
    throw err;
  } finally {
    client.release();
  }
}
```

---

## Step 1236: JOIN Queries

```javascript
// INNER JOIN - เอาเฉพาะที่มีใน ทั้งสอง tables
async function getPostsWithAuthors() {
  const result = await pool.query(`
    SELECT 
      p.id,
      p.title,
      p.content,
      p.created_at,
      u.id as author_id,
      u.name as author_name,
      u.email as author_email
    FROM posts p
    INNER JOIN users u ON p.user_id = u.id
    ORDER BY p.created_at DESC
  `);
  
  return result.rows;
}

// LEFT JOIN - เอาทุกอันจาก left table
async function getUsersWithPostCount() {
  const result = await pool.query(`
    SELECT 
      u.id,
      u.name,
      u.email,
      COUNT(p.id) as post_count
    FROM users u
    LEFT JOIN posts p ON u.id = p.user_id
    GROUP BY u.id, u.name, u.email
    ORDER BY post_count DESC
  `);
  
  return result.rows;
}

// Multiple JOINs
async function getPostsWithCommentsAndTags() {
  const result = await pool.query(`
    SELECT 
      p.id,
      p.title,
      u.name as author,
      COUNT(DISTINCT c.id) as comment_count,
      ARRAY_AGG(DISTINCT t.name) as tags
    FROM posts p
    INNER JOIN users u ON p.user_id = u.id
    LEFT JOIN comments c ON p.id = c.post_id
    LEFT JOIN post_tags pt ON p.id = pt.post_id
    LEFT JOIN tags t ON pt.tag_id = t.id
    WHERE p.is_published = true
    GROUP BY p.id, p.title, u.name
    ORDER BY p.created_at DESC
  `);
  
  return result.rows;
}

// Subquery
async function getUsersWithMostRecentPost() {
  const result = await pool.query(`
    SELECT 
      u.*,
      latest_post.title as latest_post_title,
      latest_post.created_at as latest_post_date
    FROM users u
    LEFT JOIN LATERAL (
      SELECT title, created_at
      FROM posts
      WHERE user_id = u.id
      ORDER BY created_at DESC
      LIMIT 1
    ) latest_post ON true
  `);
  
  return result.rows;
}
```

---

## Step 1237: Prisma ORM - Setup

```javascript
// ติดตั้ง: npm install prisma @prisma/client
// npx prisma init

// prisma/schema.prisma
/*
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

model User {
  id        Int      @id @default(autoincrement())
  name      String
  email     String   @unique
  password  String
  role      Role     @default(USER)
  isActive  Boolean  @default(true) @map("is_active")
  createdAt DateTime @default(now()) @map("created_at")
  updatedAt DateTime @updatedAt @map("updated_at")
  
  posts     Post[]
  comments  Comment[]
  
  @@map("users")
}

model Post {
  id          Int       @id @default(autoincrement())
  title       String
  content     String?
  isPublished Boolean   @default(false) @map("is_published")
  viewCount   Int       @default(0) @map("view_count")
  authorId    Int       @map("author_id")
  createdAt   DateTime  @default(now()) @map("created_at")
  updatedAt   DateTime  @updatedAt @map("updated_at")
  
  author      User      @relation(fields: [authorId], references: [id])
  comments    Comment[]
  tags        Tag[]     @relation("PostTags")
  
  @@map("posts")
}

model Comment {
  id        Int      @id @default(autoincrement())
  content   String
  postId    Int      @map("post_id")
  authorId  Int      @map("author_id")
  createdAt DateTime @default(now()) @map("created_at")
  
  post      Post     @relation(fields: [postId], references: [id])
  author    User     @relation(fields: [authorId], references: [id])
  
  @@map("comments")
}

model Tag {
  id    Int    @id @default(autoincrement())
  name  String @unique
  slug  String @unique
  
  posts Post[] @relation("PostTags")
  
  @@map("tags")
}

enum Role {
  USER
  MODERATOR
  ADMIN
}
*/

// การ generate client และ migrate
// npx prisma generate
// npx prisma migrate dev --name init
// npx prisma migrate deploy (production)
// npx prisma studio (GUI)
```

---

## Step 1238: Prisma Client - CRUD

```javascript
const { PrismaClient } = require('@prisma/client');

const prisma = new PrismaClient({
  log: ['query', 'info', 'warn', 'error'], // logging
});

// Create
async function createUser() {
  const user = await prisma.user.create({
    data: {
      name: 'Alice',
      email: 'alice@example.com',
      password: 'hashed_password'
    }
  });
  console.log('สร้าง user:', user.id);
  return user;
}

// Create กับ nested data
async function createUserWithPost() {
  const user = await prisma.user.create({
    data: {
      name: 'Bob',
      email: 'bob@example.com',
      password: 'hashed_password',
      posts: {
        create: [
          {
            title: 'Hello World',
            content: 'My first post'
          }
        ]
      }
    },
    include: {
      posts: true
    }
  });
  return user;
}

// Read
async function findUsers() {
  // หาทั้งหมด
  const all = await prisma.user.findMany();
  
  // หาด้วย filter
  const activeUsers = await prisma.user.findMany({
    where: { isActive: true }
  });
  
  // หาด้วย ID
  const user = await prisma.user.findUnique({
    where: { id: 1 }
  });
  
  // หาด้วย email (unique field)
  const byEmail = await prisma.user.findUnique({
    where: { email: 'alice@example.com' }
  });
  
  // หาอันแรก
  const first = await prisma.user.findFirst({
    where: { role: 'ADMIN' },
    orderBy: { createdAt: 'asc' }
  });
  
  return { all, activeUsers, user };
}

// Update
async function updateUser() {
  // Update by ID
  const updated = await prisma.user.update({
    where: { id: 1 },
    data: { name: 'Alice Smith' }
  });
  
  // Update หลาย records
  const result = await prisma.user.updateMany({
    where: { isActive: false },
    data: { role: 'USER' }
  });
  console.log(`อัพเดต ${result.count} records`);
  
  // Upsert (update หรือ create ถ้าไม่มี)
  const user = await prisma.user.upsert({
    where: { email: 'alice@example.com' },
    update: { name: 'Alice Updated' },
    create: {
      email: 'alice@example.com',
      name: 'Alice New',
      password: 'password'
    }
  });
  
  return updated;
}

// Delete
async function deleteUser() {
  // Delete by ID
  const deleted = await prisma.user.delete({
    where: { id: 1 }
  });
  
  // Delete หลาย records
  const result = await prisma.user.deleteMany({
    where: { isActive: false }
  });
  
  return deleted;
}
```

---

## Step 1239: Prisma - Relations

```javascript
// One-to-Many: User -> Posts
async function userWithPosts(userId) {
  const user = await prisma.user.findUnique({
    where: { id: userId },
    include: {
      posts: {
        where: { isPublished: true },
        orderBy: { createdAt: 'desc' },
        take: 5,
        select: {
          id: true,
          title: true,
          createdAt: true
        }
      }
    }
  });
  return user;
}

// Nested Include
async function postWithEverything(postId) {
  const post = await prisma.post.findUnique({
    where: { id: postId },
    include: {
      author: {
        select: { id: true, name: true, email: true }
      },
      comments: {
        include: {
          author: {
            select: { id: true, name: true }
          }
        },
        orderBy: { createdAt: 'asc' }
      },
      tags: true
    }
  });
  return post;
}

// Many-to-Many: Posts <-> Tags
async function connectTagToPost(postId, tagId) {
  const post = await prisma.post.update({
    where: { id: postId },
    data: {
      tags: {
        connect: { id: tagId }
      }
    },
    include: { tags: true }
  });
  return post;
}

async function disconnectTagFromPost(postId, tagId) {
  return prisma.post.update({
    where: { id: postId },
    data: {
      tags: {
        disconnect: { id: tagId }
      }
    }
  });
}

// Create กับ Relations
async function createPostWithTags(authorId, title, tagNames) {
  // หรือสร้าง tags ใหม่ถ้ายังไม่มี
  const post = await prisma.post.create({
    data: {
      title,
      authorId,
      tags: {
        connectOrCreate: tagNames.map(name => ({
          where: { name },
          create: { name, slug: name.toLowerCase().replace(/\s+/g, '-') }
        }))
      }
    },
    include: { tags: true }
  });
  return post;
}
```

---

## Step 1240: Prisma - Filtering และ Pagination

```javascript
// Filtering ที่ซับซ้อน
async function advancedFiltering() {
  const posts = await prisma.post.findMany({
    where: {
      AND: [
        { isPublished: true },
        {
          OR: [
            { title: { contains: 'javascript', mode: 'insensitive' } },
            { content: { contains: 'javascript', mode: 'insensitive' } }
          ]
        },
        {
          author: {
            isActive: true
          }
        },
        {
          createdAt: {
            gte: new Date('2024-01-01'),
            lte: new Date('2024-12-31')
          }
        }
      ]
    },
    include: {
      author: { select: { name: true } },
      tags: { select: { name: true } }
    }
  });
  
  return posts;
}

// String Filters
async function stringFilters() {
  // contains - มีข้อความนี้อยู่
  await prisma.user.findMany({
    where: { name: { contains: 'Alice' } }
  });
  
  // startsWith
  await prisma.user.findMany({
    where: { email: { startsWith: 'admin' } }
  });
  
  // endsWith
  await prisma.user.findMany({
    where: { email: { endsWith: '@company.com' } }
  });
  
  // Case insensitive
  await prisma.user.findMany({
    where: { name: { contains: 'alice', mode: 'insensitive' } }
  });
}

// Pagination
async function paginate(page = 1, perPage = 10, filters = {}) {
  const skip = (page - 1) * perPage;
  
  const [items, total] = await Promise.all([
    prisma.post.findMany({
      where: filters,
      skip,
      take: perPage,
      orderBy: { createdAt: 'desc' },
      include: {
        author: { select: { name: true } }
      }
    }),
    prisma.post.count({ where: filters })
  ]);
  
  return {
    items,
    meta: {
      page,
      perPage,
      total,
      lastPage: Math.ceil(total / perPage),
      hasNextPage: page < Math.ceil(total / perPage),
      hasPrevPage: page > 1
    }
  };
}

// Cursor-based Pagination
async function cursorPaginate(cursor, take = 10) {
  const items = await prisma.post.findMany({
    take,
    skip: cursor ? 1 : 0,
    cursor: cursor ? { id: cursor } : undefined,
    where: { isPublished: true },
    orderBy: { id: 'asc' }
  });
  
  const nextCursor = items.length === take ? items[items.length - 1].id : null;
  
  return { items, nextCursor };
}
```

---

## Step 1241: Prisma - Transactions

```javascript
// Prisma Transaction
async function createOrderWithPrisma(userId, items) {
  return prisma.$transaction(async (tx) => {
    // ตรวจสอบ stock
    const products = await tx.product.findMany({
      where: {
        id: { in: items.map(i => i.productId) }
      }
    });
    
    for (const item of items) {
      const product = products.find(p => p.id === item.productId);
      
      if (!product || product.stock < item.quantity) {
        throw new Error(`สินค้า ${item.productId} ไม่เพียงพอ`);
      }
    }
    
    // สร้าง order
    const total = items.reduce((sum, item) => {
      const product = products.find(p => p.id === item.productId);
      return sum + product.price * item.quantity;
    }, 0);
    
    const order = await tx.order.create({
      data: {
        userId,
        total,
        status: 'PENDING',
        items: {
          create: items.map(item => {
            const product = products.find(p => p.id === item.productId);
            return {
              productId: item.productId,
              quantity: item.quantity,
              price: product.price
            };
          })
        }
      },
      include: { items: true }
    });
    
    // ลด stock
    await Promise.all(items.map(item =>
      tx.product.update({
        where: { id: item.productId },
        data: { stock: { decrement: item.quantity } }
      })
    ));
    
    return order;
  });
}

// Interactive Transaction
async function interactiveTransaction() {
  const [user, post] = await prisma.$transaction([
    prisma.user.create({
      data: { name: 'Test', email: 'test@test.com', password: 'pass' }
    }),
    prisma.post.create({
      data: { title: 'First Post', authorId: 1 }
    })
  ]);
  
  return { user, post };
}
```

---

## Step 1242: Knex.js Query Builder

```javascript
// ติดตั้ง: npm install knex pg
// knex เป็น query builder ที่ flexible กว่า ORM

const knex = require('knex')({
  client: 'postgresql',
  connection: {
    host: 'localhost',
    port: 5432,
    database: 'myapp',
    user: 'postgres',
    password: 'password'
  },
  pool: { min: 2, max: 10 },
  migrations: {
    tableName: 'knex_migrations',
    directory: './migrations'
  }
});

// CRUD ด้วย Knex
async function knexExamples() {
  // SELECT
  const users = await knex('users')
    .select('id', 'name', 'email')
    .where({ is_active: true })
    .orderBy('name', 'asc')
    .limit(10);
  
  // SELECT กับ JOIN
  const postsWithAuthors = await knex('posts')
    .join('users', 'posts.user_id', 'users.id')
    .select(
      'posts.id',
      'posts.title',
      'users.name as author_name'
    )
    .where('posts.is_published', true);
  
  // INSERT
  const [userId] = await knex('users')
    .insert({ name: 'Alice', email: 'alice@example.com' })
    .returning('id');
  
  // UPDATE
  await knex('users')
    .where({ id: userId })
    .update({ name: 'Alice Smith' });
  
  // DELETE
  await knex('users')
    .where({ id: userId })
    .delete();
  
  return users;
}

// Knex Migrations
async function createMigration() {
  // migrations/20240101_create_users.js
  
  exports.up = async function(knex) {
    await knex.schema.createTable('users', (table) => {
      table.increments('id').primary();
      table.string('name', 100).notNullable();
      table.string('email', 255).unique().notNullable();
      table.string('password').notNullable();
      table.enum('role', ['user', 'admin', 'moderator']).defaultTo('user');
      table.boolean('is_active').defaultTo(true);
      table.timestamp('created_at').defaultTo(knex.fn.now());
      table.timestamp('updated_at').defaultTo(knex.fn.now());
    });
  };
  
  exports.down = async function(knex) {
    await knex.schema.dropTable('users');
  };
}

// Knex Transactions
async function knexTransaction(fromId, toId, amount) {
  return knex.transaction(async (trx) => {
    const from = await trx('accounts').where({ id: fromId }).first();
    
    if (from.balance < amount) {
      throw new Error('ยอดเงินไม่เพียงพอ');
    }
    
    await trx('accounts').where({ id: fromId }).decrement('balance', amount);
    await trx('accounts').where({ id: toId }).increment('balance', amount);
    
    await trx('transactions').insert({
      from_id: fromId,
      to_id: toId,
      amount
    });
  });
}
```

---

## Step 1243: Database Design Patterns

```javascript
// 1. Soft Delete Pattern
/*
ALTER TABLE users 
ADD COLUMN deleted_at TIMESTAMP;
*/

async function softDelete(id) {
  return prisma.user.update({
    where: { id },
    data: { deletedAt: new Date() }
  });
}

// ต้องกรอง soft deleted records เสมอ
async function findActiveUsers() {
  return prisma.user.findMany({
    where: { deletedAt: null }
  });
}

// 2. Audit Trail Pattern
/*
CREATE TABLE audit_logs (
  id SERIAL PRIMARY KEY,
  table_name VARCHAR(100),
  record_id INTEGER,
  action VARCHAR(20),  -- INSERT, UPDATE, DELETE
  old_values JSONB,
  new_values JSONB,
  user_id INTEGER,
  created_at TIMESTAMP DEFAULT NOW()
);
*/

async function createWithAudit(table, data, userId) {
  const client = await pool.connect();
  
  try {
    await client.query('BEGIN');
    
    const result = await client.query(
      `INSERT INTO ${table} (${Object.keys(data).join(',')}) 
       VALUES (${Object.keys(data).map((_, i) => `$${i+1}`).join(',')})
       RETURNING *`,
      Object.values(data)
    );
    
    await client.query(
      `INSERT INTO audit_logs (table_name, record_id, action, new_values, user_id)
       VALUES ($1, $2, $3, $4, $5)`,
      [table, result.rows[0].id, 'INSERT', JSON.stringify(result.rows[0]), userId]
    );
    
    await client.query('COMMIT');
    return result.rows[0];
  } catch (err) {
    await client.query('ROLLBACK');
    throw err;
  } finally {
    client.release();
  }
}

// 3. Optimistic Locking
/*
ALTER TABLE products ADD COLUMN version INTEGER DEFAULT 0;
*/

async function updateWithOptimisticLock(id, currentVersion, newData) {
  const result = await pool.query(
    `UPDATE products 
     SET ${Object.keys(newData).map((k, i) => `${k} = $${i+1}`).join(',')},
         version = version + 1
     WHERE id = $${Object.keys(newData).length + 1}
       AND version = $${Object.keys(newData).length + 2}
     RETURNING *`,
    [...Object.values(newData), id, currentVersion]
  );
  
  if (result.rowCount === 0) {
    throw new Error('ข้อมูลถูกแก้ไขโดยคนอื่น กรุณาลองใหม่');
  }
  
  return result.rows[0];
}
```

---

## Step 1244: Indexing ใน PostgreSQL

```javascript
// ประเภท Index ใน PostgreSQL

// 1. B-tree Index (default) - เหมาะสำหรับ equality และ range queries
/*
CREATE INDEX idx_users_email ON users (email);
CREATE INDEX idx_posts_created_at ON posts (created_at DESC);
*/

// 2. Composite Index
/*
CREATE INDEX idx_posts_user_published ON posts (user_id, is_published);
*/

// 3. Partial Index - Index เฉพาะบาง rows
/*
CREATE INDEX idx_active_users ON users (email) WHERE is_active = true;
CREATE INDEX idx_published_posts ON posts (created_at DESC) WHERE is_published = true;
*/

// 4. GIN Index สำหรับ full-text search
/*
CREATE INDEX idx_posts_search ON posts USING gin(to_tsvector('english', title || ' ' || content));
*/

// 5. GiST Index สำหรับ geometric types
/*
CREATE INDEX idx_locations ON places USING gist(coordinates);
*/

// การตรวจสอบการใช้งาน Index
async function analyzeQuery(query, values = []) {
  const explanation = await pool.query(
    `EXPLAIN ANALYZE ${query}`,
    values
  );
  
  console.log('Query Plan:');
  explanation.rows.forEach(row => {
    console.log(row['QUERY PLAN']);
  });
}

// ตัวอย่าง
async function checkIndexUsage() {
  await analyzeQuery(
    'SELECT * FROM users WHERE email = $1',
    ['test@example.com']
  );
  
  // EXPLAIN ANALYZE แสดงว่า index ถูกใช้หรือไม่
  // Index Scan: ดี - ใช้ index
  // Seq Scan: อาจแย่ - scan ทั้งหมด (เหมาะเมื่อ data น้อย)
}

// Prisma กับ Index
/*
// ใน schema.prisma
model User {
  id    Int    @id @default(autoincrement())
  email String @unique  // สร้าง unique index
  name  String
  role  Role

  @@index([role])                    // B-tree index
  @@index([name, email])             // Composite index
  @@index([createdAt(sort: Desc)])   // Descending index
}
*/
```

---

## Step 1245: Raw Queries กับ Prisma

```javascript
// เมื่อ Prisma ไม่รองรับ query ที่ต้องการ ใช้ raw queries

const { PrismaClient } = require('@prisma/client');
const prisma = new PrismaClient();

// queryRaw - ส่ง values อย่างปลอดภัย
async function rawQuery() {
  // ใช้ Prisma.sql สำหรับ parameterized queries
  const { Prisma } = require('@prisma/client');
  
  const email = 'alice@example.com';
  
  const users = await prisma.$queryRaw`
    SELECT id, name, email 
    FROM users 
    WHERE email = ${email}
  `;
  
  return users;
}

// executeRaw - สำหรับ INSERT/UPDATE/DELETE ที่ Prisma ทำเองไม่ได้
async function rawExecute() {
  const result = await prisma.$executeRaw`
    UPDATE users 
    SET login_count = login_count + 1, 
        last_login_at = NOW()
    WHERE id = ${userId}
  `;
  
  console.log(`อัพเดต ${result} rows`);
}

// Full-text Search
async function fullTextSearch(searchTerm) {
  const users = await prisma.$queryRaw`
    SELECT id, name, email,
           ts_rank(
             to_tsvector('english', name || ' ' || COALESCE(bio, '')),
             plainto_tsquery('english', ${searchTerm})
           ) as rank
    FROM users
    WHERE to_tsvector('english', name || ' ' || COALESCE(bio, ''))
      @@ plainto_tsquery('english', ${searchTerm})
    ORDER BY rank DESC
    LIMIT 10
  `;
  
  return users;
}
```

---

## Step 1246: Connection Pooling

```javascript
// Connection Pool จัดการการเชื่อมต่อให้มีประสิทธิภาพ

const { Pool } = require('pg');

// Configuration สำหรับ production
const pool = new Pool({
  connectionString: process.env.DATABASE_URL,
  max: 20,                    // จำนวน connections สูงสุด
  min: 2,                     // จำนวน connections น้อยสุด
  idleTimeoutMillis: 30000,   // ปิด connection ที่ไม่ใช้ใน 30 วินาที
  connectionTimeoutMillis: 2000, // timeout ในการสร้าง connection ใหม่
  maxUses: 7500               // ปิด connection หลังใช้งาน 7500 ครั้ง
});

// Monitor pool events
pool.on('connect', (client) => {
  console.log('สร้าง connection ใหม่');
});

pool.on('remove', (client) => {
  console.log('ลบ connection');
});

pool.on('error', (err, client) => {
  console.error('Pool error:', err);
});

// PgBouncer สำหรับ high-traffic applications
// PgBouncer เป็น connection pooler อีก layer หนึ่ง
// ช่วยลด overhead ของ connection สำหรับ serverless functions

// ตรวจสอบสถานะ pool
async function getPoolStatus() {
  return {
    total: pool.totalCount,
    idle: pool.idleCount,
    waiting: pool.waitingCount
  };
}
```

---

## Step 1247: Database Migrations

```javascript
// Prisma Migrations

// 1. สร้าง migration
// npx prisma migrate dev --name add_user_bio

// 2. Migration file ที่ถูกสร้าง
// prisma/migrations/20240101000000_add_user_bio/migration.sql
/*
-- AlterTable
ALTER TABLE "users" ADD COLUMN "bio" TEXT;
*/

// 3. Apply migration ใน production
// npx prisma migrate deploy

// Custom Migration Scripts
// prisma/migrations/20240101000001_seed_roles/migration.sql
/*
INSERT INTO roles (name, description) VALUES 
('admin', 'System administrator'),
('moderator', 'Content moderator'),
('user', 'Regular user')
ON CONFLICT (name) DO NOTHING;
*/

// Prisma Seed
// prisma/seed.ts
const { PrismaClient } = require('@prisma/client');
const bcrypt = require('bcryptjs');

const prisma = new PrismaClient();

async function seed() {
  // สร้าง admin user
  const adminPassword = await bcrypt.hash('Admin@1234', 12);
  
  const admin = await prisma.user.upsert({
    where: { email: 'admin@example.com' },
    update: {},
    create: {
      name: 'Admin',
      email: 'admin@example.com',
      password: adminPassword,
      role: 'ADMIN'
    }
  });
  
  console.log('สร้าง admin:', admin.id);
  
  // สร้าง sample data
  const tags = await Promise.all([
    prisma.tag.upsert({
      where: { slug: 'javascript' },
      update: {},
      create: { name: 'JavaScript', slug: 'javascript' }
    }),
    prisma.tag.upsert({
      where: { slug: 'nodejs' },
      update: {},
      create: { name: 'Node.js', slug: 'nodejs' }
    })
  ]);
  
  console.log('สร้าง tags:', tags.map(t => t.name));
}

seed()
  .catch(console.error)
  .finally(() => prisma.$disconnect());
```

---

## Step 1248: Express กับ PostgreSQL

```javascript
// Full Express API กับ PostgreSQL
const express = require('express');
const router = express.Router();
const { PrismaClient } = require('@prisma/client');

const prisma = new PrismaClient();

// GET /users - รายชื่อ users
router.get('/', async (req, res) => {
  try {
    const { page = 1, limit = 10, search, role } = req.query;
    
    const where = {};
    if (search) {
      where.OR = [
        { name: { contains: search, mode: 'insensitive' } },
        { email: { contains: search, mode: 'insensitive' } }
      ];
    }
    if (role) where.role = role;
    
    const [users, total] = await Promise.all([
      prisma.user.findMany({
        where,
        skip: (parseInt(page) - 1) * parseInt(limit),
        take: parseInt(limit),
        orderBy: { createdAt: 'desc' },
        select: {
          id: true,
          name: true,
          email: true,
          role: true,
          isActive: true,
          createdAt: true
        }
      }),
      prisma.user.count({ where })
    ]);
    
    res.json({
      users,
      pagination: {
        page: parseInt(page),
        limit: parseInt(limit),
        total,
        totalPages: Math.ceil(total / parseInt(limit))
      }
    });
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

// GET /users/:id
router.get('/:id', async (req, res) => {
  try {
    const user = await prisma.user.findUnique({
      where: { id: parseInt(req.params.id) },
      include: {
        posts: {
          where: { isPublished: true },
          orderBy: { createdAt: 'desc' },
          take: 5
        }
      }
    });
    
    if (!user) {
      return res.status(404).json({ error: 'ไม่พบ user' });
    }
    
    res.json(user);
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

// POST /users
router.post('/', async (req, res) => {
  try {
    const { name, email, password, role } = req.body;
    
    const exists = await prisma.user.findUnique({ where: { email } });
    if (exists) {
      return res.status(400).json({ error: 'Email นี้มีอยู่แล้ว' });
    }
    
    const user = await prisma.user.create({
      data: { name, email, password, role },
      select: { id: true, name: true, email: true, role: true, createdAt: true }
    });
    
    res.status(201).json(user);
  } catch (err) {
    if (err.code === 'P2002') {
      return res.status(400).json({ error: 'Email ซ้ำ' });
    }
    res.status(500).json({ error: err.message });
  }
});
```

---

## Step 1249: Error Handling กับ Prisma

```javascript
const { Prisma } = require('@prisma/client');

function handlePrismaError(err, res) {
  // Unique constraint violation
  if (err instanceof Prisma.PrismaClientKnownRequestError) {
    switch (err.code) {
      case 'P2002': {
        const field = err.meta?.target?.[0];
        return res.status(400).json({
          error: `${field} ซ้ำกัน`
        });
      }
      case 'P2003': {
        return res.status(400).json({
          error: 'Foreign key ไม่ถูกต้อง'
        });
      }
      case 'P2025': {
        return res.status(404).json({
          error: 'ไม่พบข้อมูล'
        });
      }
      default: {
        console.error('Prisma error:', err.code, err.message);
        return res.status(500).json({ error: 'Database error' });
      }
    }
  }
  
  // Validation error
  if (err instanceof Prisma.PrismaClientValidationError) {
    return res.status(400).json({
      error: 'ข้อมูลไม่ถูกต้อง',
      details: err.message
    });
  }
  
  // Connection error
  if (err instanceof Prisma.PrismaClientInitializationError) {
    return res.status(503).json({
      error: 'ไม่สามารถเชื่อมต่อ database'
    });
  }
  
  return res.status(500).json({ error: 'Internal server error' });
}

// Global Error Middleware
function prismaErrorMiddleware(err, req, res, next) {
  if (err instanceof Prisma.PrismaClientKnownRequestError ||
      err instanceof Prisma.PrismaClientValidationError) {
    return handlePrismaError(err, res);
  }
  next(err);
}
```

---

## Step 1250: Performance Tips

```javascript
// 1. Select เฉพาะ fields ที่ใช้
const users = await prisma.user.findMany({
  select: { id: true, name: true } // ไม่ดึง password, bio, etc.
});

// 2. ใช้ index สำหรับ query ที่บ่อย
// @@index([email]) ใน schema

// 3. ใช้ count ก่อน find เมื่อ result อาจเป็น 0
const count = await prisma.post.count({ where: filters });
if (count === 0) return { items: [], meta: { total: 0 } };
const items = await prisma.post.findMany({ where: filters });

// 4. Batch queries ด้วย $transaction
const [users, posts] = await prisma.$transaction([
  prisma.user.findMany(),
  prisma.post.findMany()
]);

// 5. Connection Management
const prisma = new PrismaClient({
  datasources: {
    db: {
      url: process.env.DATABASE_URL
    }
  }
});

// ปิด connection เมื่อ shutdown
process.on('SIGTERM', async () => {
  await prisma.$disconnect();
  process.exit(0);
});

// 6. Database Query Logging
const prisma = new PrismaClient({
  log: [
    { emit: 'event', level: 'query' }
  ]
});

prisma.$on('query', (e) => {
  if (e.duration > 1000) { // log queries ที่ใช้เวลา > 1 วินาที
    console.warn('Slow query:', e.query, 'Duration:', e.duration, 'ms');
  }
});
```

---

## แบบฝึกหัด

### ระดับเริ่มต้น
1. สร้าง PostgreSQL schema สำหรับ Blog (users, posts, comments)
2. เขียน CRUD API สำหรับ User ด้วย pg
3. ป้องกัน SQL injection ด้วย parameterized queries

### ระดับกลาง
4. สร้าง Prisma schema พร้อม migrations
5. implement paginated search สำหรับ posts
6. สร้างระบบ Blog API ด้วย Prisma (CRUD, relations, pagination)

### ระดับสูง
7. สร้างระบบ E-commerce ด้วย PostgreSQL transactions
8. implement full-text search ด้วย PostgreSQL
9. เขียน audit logging system
10. Optimize queries ด้วยการใช้ EXPLAIN ANALYZE

---

*Part 63 จบแล้ว ต่อไปเป็น Part 64: Testing - Unit Tests*
