# Part 90: System Design สำหรับ JavaScript Developer
## ขั้นตอนที่ 1771-1790: การออกแบบระบบขนาดใหญ่

System Design คือทักษะสำคัญที่ Developer ระดับ Senior ต้องมี เป็นการออกแบบสถาปัตยกรรมที่รองรับการขยายตัวและใช้งานจริงในระดับ Production

---

## ขั้นตอนที่ 1771: System Design Fundamentals

### คำถามที่ต้องถามก่อน Design

```
1. Scale Requirements:
   - มีผู้ใช้กี่คน? (DAU/MAU)
   - Requests per second (RPS)?
   - ข้อมูลมีขนาดเท่าไหร่?
   - Growth rate คือเท่าไหร่?

2. Availability Requirements:
   - ต้องการ uptime กี่ %? (99.9% = 8.7 ชั่วโมง/ปี downtime)
   - มี single point of failure ได้ไหม?
   - Recovery time objective (RTO) คือเท่าไหร่?

3. Consistency Requirements:
   - ต้องการ strong consistency หรือ eventual consistency?
   - Latency requirements คือเท่าไหร่?

4. Feature Requirements:
   - Core features คืออะไร?
   - Nice-to-have features?
   - Constraints ที่มี?
```

### Back-of-envelope Calculations

```javascript
// ตัวเลขที่ต้องรู้จำ

const UNITS = {
  KB: 1024,
  MB: 1024 * 1024,
  GB: 1024 * 1024 * 1024,
  TB: 1024 * 1024 * 1024 * 1024,
};

const TIME = {
  SECOND: 1,
  MINUTE: 60,
  HOUR: 3600,
  DAY: 86400,
  MONTH: 30 * 86400,
  YEAR: 365 * 86400,
};

// ตัวอย่าง: Twitter-like system
const dau = 100_000_000; // 100M daily active users
const tweetsPerDay = dau * 2; // 200M tweets/day
const readWriteRatio = 100; // 100 reads per write
const readsPerDay = tweetsPerDay * readWriteRatio; // 20B reads/day

const writesPerSecond = tweetsPerDay / TIME.DAY;
// 200M / 86400 ≈ 2315 writes/second

const readsPerSecond = readsPerDay / TIME.DAY;
// 20B / 86400 ≈ 231,500 reads/second

// Storage
const avgTweetSize = 280; // bytes
const storagePerDay = tweetsPerDay * avgTweetSize;
// 200M * 280 bytes ≈ 56GB/day

const storagePerYear = storagePerDay * 365;
// ≈ 20TB/year

console.log({
  writesPerSecond: Math.round(writesPerSecond),
  readsPerSecond: Math.round(readsPerSecond),
  storageGBPerDay: (storagePerDay / UNITS.GB).toFixed(1),
  storageTBPerYear: (storagePerYear / UNITS.TB).toFixed(1),
});
```

---

## ขั้นตอนที่ 1772: Scalability - Vertical vs Horizontal

### Vertical Scaling (Scale Up)

```
เพิ่ม resources ให้ server เดียวกัน:
Server: 4 CPU, 8GB RAM
  ↓ Scale Up
Server: 16 CPU, 64GB RAM

ข้อดี:
✅ ง่าย ไม่ต้องเปลี่ยน code
✅ ไม่มีปัญหา distributed systems

ข้อเสีย:
❌ มี ceiling (ขนาดสูงสุดที่ hardware รองรับ)
❌ Single point of failure
❌ ราคาแพงมากเมื่อขนาดใหญ่
```

### Horizontal Scaling (Scale Out)

```
เพิ่มจำนวน servers:
Server 1
Server 2  <- Load Balancer -> Clients
Server 3

ข้อดี:
✅ ไม่มี ceiling ทางทฤษฎี
✅ High availability
✅ Cost effective

ข้อเสีย:
❌ ซับซ้อนกว่า (distributed systems problems)
❌ State management ยาก
❌ Network latency
```

```javascript
// ปัญหาของ Horizontal Scaling: Session State

// ❌ ปัญหา: Session ถูกเก็บที่ Server 1
// Client -> Load Balancer -> Server 2 (ไม่มี session!)

// ✅ วิธีแก้ 1: Sticky Sessions (IP Hash)
// Nginx config:
// upstream backend {
//   ip_hash;  # Client IP เดิม -> Server เดิมเสมอ
//   server server1:3000;
//   server server2:3000;
// }

// ✅ วิธีแก้ 2: Centralized Session Store (Redis)
import { createClient } from 'redis';
import session from 'express-session';
import RedisStore from 'connect-redis';

const redisClient = createClient({ url: process.env.REDIS_URL });
await redisClient.connect();

app.use(session({
  store: new RedisStore({ client: redisClient }),
  secret: process.env.SESSION_SECRET,
  resave: false,
  saveUninitialized: false,
  cookie: {
    secure: true,
    httpOnly: true,
    maxAge: 24 * 60 * 60 * 1000, // 24 ชั่วโมง
  },
}));

// ✅ วิธีแก้ 3: Stateless (JWT)
// ไม่มี session บน server เลย ข้อมูลอยู่ใน token
```

---

## ขั้นตอนที่ 1773: Load Balancing

```javascript
// Load Balancing Algorithms

// 1. Round Robin - เวียนไปแต่ละ server
let currentIndex = 0;
const servers = ['server1', 'server2', 'server3'];

function roundRobin() {
  const server = servers[currentIndex];
  currentIndex = (currentIndex + 1) % servers.length;
  return server;
}

// 2. Weighted Round Robin - บาง server รับ load มากกว่า
const weightedServers = [
  { host: 'server1', weight: 3 },  // รับ 3 requests
  { host: 'server2', weight: 2 },  // รับ 2 requests
  { host: 'server3', weight: 1 },  // รับ 1 request
];

function weightedRoundRobin(servers) {
  const pool = [];
  servers.forEach(s => {
    for (let i = 0; i < s.weight; i++) {
      pool.push(s.host);
    }
  });
  
  let idx = 0;
  return () => {
    const server = pool[idx];
    idx = (idx + 1) % pool.length;
    return server;
  };
}

// 3. Least Connections - ส่งไปที่ server มี connections น้อยสุด
const serverConnections = new Map([
  ['server1', 0],
  ['server2', 0],
  ['server3', 0],
]);

function leastConnections() {
  let minConnections = Infinity;
  let selectedServer = null;
  
  for (const [server, connections] of serverConnections) {
    if (connections < minConnections) {
      minConnections = connections;
      selectedServer = server;
    }
  }
  
  serverConnections.set(selectedServer, minConnections + 1);
  return selectedServer;
}

// 4. IP Hash - Client IP กำหนด server
function ipHash(clientIp, servers) {
  let hash = 0;
  for (let i = 0; i < clientIp.length; i++) {
    hash = (hash * 31 + clientIp.charCodeAt(i)) % servers.length;
  }
  return servers[hash];
}
```

```javascript
// Health Checks สำหรับ Load Balancer
class LoadBalancer {
  constructor() {
    this.servers = [
      { host: 'server1:3000', healthy: true, failures: 0 },
      { host: 'server2:3000', healthy: true, failures: 0 },
      { host: 'server3:3000', healthy: true, failures: 0 },
    ];
    
    this.currentIndex = 0;
    this.startHealthChecks();
  }
  
  async checkHealth(server) {
    try {
      const response = await fetch(`http://${server.host}/health`, {
        signal: AbortSignal.timeout(5000),
      });
      
      if (response.ok) {
        server.healthy = true;
        server.failures = 0;
      } else {
        throw new Error(`HTTP ${response.status}`);
      }
    } catch (error) {
      server.failures++;
      console.log(`Health check failed for ${server.host}: ${error.message}`);
      
      if (server.failures >= 3) {
        server.healthy = false;
        console.log(`Marking ${server.host} as unhealthy`);
      }
    }
  }
  
  startHealthChecks() {
    setInterval(() => {
      this.servers.forEach(server => this.checkHealth(server));
    }, 10000); // ทุก 10 วินาที
  }
  
  getNextServer() {
    const healthyServers = this.servers.filter(s => s.healthy);
    
    if (healthyServers.length === 0) {
      throw new Error('No healthy servers available');
    }
    
    const server = healthyServers[this.currentIndex % healthyServers.length];
    this.currentIndex++;
    return server.host;
  }
  
  async forward(request) {
    const serverHost = this.getNextServer();
    const targetUrl = `http://${serverHost}${request.url}`;
    
    return fetch(targetUrl, {
      method: request.method,
      headers: request.headers,
      body: request.body,
    });
  }
}
```

---

## ขั้นตอนที่ 1774: Caching Strategies

### Cache หลายระดับ

```
Browser Cache (L1) - เร็วที่สุด, บนเครื่องผู้ใช้
CDN Cache (L2) - Edge locations ทั่วโลก
Application Cache (L3) - Redis, Memcached
Database Cache (L4) - Query cache, Buffer pool
```

### CDN (Content Delivery Network)

```javascript
// Next.js กับ CDN Headers
export async function GET(request) {
  const data = await getStaticContent();
  
  return Response.json(data, {
    headers: {
      // Cache 24 ชั่วโมงที่ CDN, 1 ชั่วโมงที่ browser
      'Cache-Control': 'public, s-maxage=86400, max-age=3600, stale-while-revalidate=86400',
      'CDN-Cache-Control': 'max-age=86400',
      'Vary': 'Accept-Encoding',
    },
  });
}

// สำหรับ user-specific content
export async function GET(request) {
  const userId = getUserFromRequest(request);
  const data = await getUserData(userId);
  
  return Response.json(data, {
    headers: {
      // ห้าม CDN cache, browser cache 5 นาที
      'Cache-Control': 'private, max-age=300',
    },
  });
}
```

---

## ขั้นตอนที่ 1775: Redis สำหรับ Caching

```javascript
// Redis Caching Patterns
import { createClient } from 'redis';

const redis = createClient({ url: process.env.REDIS_URL });
await redis.connect();

// 1. Cache-Aside Pattern (Lazy Loading)
async function getProduct(productId) {
  const cacheKey = `product:${productId}`;
  
  // ลองดึงจาก cache ก่อน
  const cached = await redis.get(cacheKey);
  if (cached) {
    console.log('Cache HIT:', cacheKey);
    return JSON.parse(cached);
  }
  
  // Cache miss - ดึงจาก database
  console.log('Cache MISS:', cacheKey);
  const product = await db.findProductById(productId);
  
  if (product) {
    // เก็บลง cache 1 ชั่วโมง
    await redis.setEx(cacheKey, 3600, JSON.stringify(product));
  }
  
  return product;
}

// 2. Write-Through Pattern
async function updateProduct(productId, data) {
  // อัพเดท database
  const updated = await db.updateProduct(productId, data);
  
  // อัพเดท cache ทันที
  const cacheKey = `product:${productId}`;
  await redis.setEx(cacheKey, 3600, JSON.stringify(updated));
  
  // Invalidate related caches
  await redis.del(`products:list`);
  await redis.del(`products:category:${updated.category}`);
  
  return updated;
}

// 3. Write-Behind Pattern (Async write to database)
async function incrementViewCount(productId) {
  const key = `views:${productId}`;
  const newCount = await redis.incr(key);
  
  // Schedule database write (ไม่รอ)
  if (newCount % 100 === 0) {
    // เขียนลง database ทุก 100 views
    setImmediate(async () => {
      const count = await redis.get(key);
      await db.updateViewCount(productId, parseInt(count));
    });
  }
  
  return newCount;
}

// 4. Cache Stampede Prevention (Single Flight Pattern)
const pendingRequests = new Map();

async function getWithSingleFlight(key, fetcher, ttl = 3600) {
  // ตรวจสอบ cache ก่อน
  const cached = await redis.get(key);
  if (cached) return JSON.parse(cached);
  
  // ถ้ามีการ fetch อยู่แล้ว รอผลจาก request เดิม
  if (pendingRequests.has(key)) {
    return pendingRequests.get(key);
  }
  
  // สร้าง promise ใหม่
  const promise = fetcher()
    .then(async (data) => {
      await redis.setEx(key, ttl, JSON.stringify(data));
      pendingRequests.delete(key);
      return data;
    })
    .catch(error => {
      pendingRequests.delete(key);
      throw error;
    });
  
  pendingRequests.set(key, promise);
  return promise;
}

// ใช้งาน
const product = await getWithSingleFlight(
  `product:${productId}`,
  () => db.findProductById(productId),
  3600
);
```

```javascript
// Redis Data Structures สำหรับ System Design

// 1. Sorted Set สำหรับ Leaderboard
async function updateScore(userId, score) {
  await redis.zAdd('leaderboard', [{ score, value: userId.toString() }]);
}

async function getTopPlayers(count = 10) {
  const players = await redis.zRangeWithScores('leaderboard', 0, count - 1, {
    REV: true, // highest score first
  });
  
  return players.map((p, index) => ({
    rank: index + 1,
    userId: p.value,
    score: p.score,
  }));
}

async function getUserRank(userId) {
  const rank = await redis.zRevRank('leaderboard', userId.toString());
  const score = await redis.zScore('leaderboard', userId.toString());
  return { rank: rank + 1, score };
}

// 2. Hash สำหรับ User Sessions
async function saveSession(sessionId, userData) {
  await redis.hSet(`session:${sessionId}`, userData);
  await redis.expire(`session:${sessionId}`, 86400); // 24 ชั่วโมง
}

async function getSessionField(sessionId, field) {
  return redis.hGet(`session:${sessionId}`, field);
}

// 3. Set สำหรับ Unique Visitors
async function trackVisit(userId, date) {
  const key = `visitors:${date}`;
  await redis.sAdd(key, userId.toString());
  await redis.expire(key, 7 * 86400); // เก็บ 7 วัน
}

async function getUniqueVisitorsCount(date) {
  return redis.sCard(`visitors:${date}`);
}

// 4. HyperLogLog สำหรับ Approximate Count (ใช้ memory น้อยกว่า)
async function trackPageView(url, userId) {
  const key = `pageviews:${url}`;
  await redis.pfAdd(key, userId.toString());
}

async function getApproximateViewers(url) {
  return redis.pfCount(`pageviews:${url}`);
}
```

---

## ขั้นตอนที่ 1776: Cache Invalidation

```javascript
// Cache Invalidation Strategies

// 1. TTL-based (Simplest)
await redis.setEx(key, 300, JSON.stringify(data)); // expire ใน 5 นาที

// 2. Tag-based Invalidation
class TaggedCache {
  constructor(redis) {
    this.redis = redis;
  }
  
  async set(key, value, tags = [], ttl = 3600) {
    const pipeline = this.redis.multi();
    
    // เก็บ value
    pipeline.setEx(`cache:${key}`, ttl, JSON.stringify(value));
    
    // ลงทะเบียน key ไปยัง tags
    tags.forEach(tag => {
      pipeline.sAdd(`tag:${tag}`, `cache:${key}`);
    });
    
    await pipeline.exec();
  }
  
  async get(key) {
    const value = await this.redis.get(`cache:${key}`);
    return value ? JSON.parse(value) : null;
  }
  
  async invalidateTag(tag) {
    const keys = await this.redis.sMembers(`tag:${tag}`);
    
    if (keys.length > 0) {
      const pipeline = this.redis.multi();
      pipeline.del(...keys);         // ลบ cached values
      pipeline.del(`tag:${tag}`);   // ลบ tag set
      await pipeline.exec();
      
      console.log(`Invalidated ${keys.length} entries for tag: ${tag}`);
    }
  }
}

// ใช้งาน
const cache = new TaggedCache(redis);

// เก็บ product กับ tags หลายอัน
await cache.set(
  `product:123`,
  productData,
  ['products', 'category:electronics', 'brand:apple'],
  3600
);

// เมื่อสินค้าในหมวด electronics เปลี่ยน invalidate ทั้งหมด
await cache.invalidateTag('category:electronics');
```

---

## ขั้นตอนที่ 1777: Database Scaling

### Connection Pooling

```javascript
// pg-pool สำหรับ PostgreSQL
import { Pool } from 'pg';

const pool = new Pool({
  host: process.env.DB_HOST,
  user: process.env.DB_USER,
  password: process.env.DB_PASSWORD,
  database: process.env.DB_NAME,
  
  // Connection pool settings
  min: 2,          // minimum connections ที่ keep alive
  max: 20,         // maximum connections
  idleTimeoutMillis: 30000,    // ปิด connection ที่ idle 30 วินาที
  connectionTimeoutMillis: 2000, // timeout เมื่อรอ connection
  
  // SSL
  ssl: process.env.NODE_ENV === 'production' ? { rejectUnauthorized: false } : false,
});

pool.on('error', (err) => {
  console.error('Unexpected error on idle client', err);
});

// Wrapper function
async function query(text, params) {
  const start = Date.now();
  
  try {
    const result = await pool.query(text, params);
    const duration = Date.now() - start;
    
    if (duration > 1000) {
      console.warn('Slow query detected:', { text, duration, rows: result.rowCount });
    }
    
    return result;
  } catch (error) {
    console.error('Query error:', { text, error: error.message });
    throw error;
  }
}

// ใช้ transaction
async function withTransaction(callback) {
  const client = await pool.connect();
  
  try {
    await client.query('BEGIN');
    const result = await callback(client);
    await client.query('COMMIT');
    return result;
  } catch (error) {
    await client.query('ROLLBACK');
    throw error;
  } finally {
    client.release();
  }
}

// ตัวอย่าง transaction
const result = await withTransaction(async (client) => {
  await client.query(
    'UPDATE accounts SET balance = balance - $1 WHERE id = $2',
    [amount, fromId]
  );
  await client.query(
    'UPDATE accounts SET balance = balance + $1 WHERE id = $2',
    [amount, toId]
  );
  return { success: true };
});
```

### Read Replicas

```javascript
// Read/Write Splitting
import { Pool } from 'pg';

const writePool = new Pool({
  host: process.env.DB_PRIMARY_HOST,
  // ...
});

const readPool = new Pool({
  host: process.env.DB_REPLICA_HOST,
  // ...
});

// Database client ที่รองรับ read replicas
class DatabaseClient {
  async query(sql, params) {
    // SELECT ไป replica
    if (sql.trim().toUpperCase().startsWith('SELECT')) {
      return readPool.query(sql, params);
    }
    // INSERT, UPDATE, DELETE ไป primary
    return writePool.query(sql, params);
  }
  
  async transaction(callback) {
    // Transactions ไป primary เสมอ
    const client = await writePool.connect();
    try {
      await client.query('BEGIN');
      const result = await callback(client);
      await client.query('COMMIT');
      return result;
    } catch (e) {
      await client.query('ROLLBACK');
      throw e;
    } finally {
      client.release();
    }
  }
}

const db = new DatabaseClient();

// Auto routing
const users = await db.query('SELECT * FROM users WHERE active = true');  // -> replica
await db.query('UPDATE users SET last_login = NOW() WHERE id = $1', [userId]); // -> primary
```

### Database Sharding

```javascript
// Horizontal Sharding - แบ่ง data ไปหลาย databases
class ShardedDatabase {
  constructor(shards) {
    this.shards = shards; // [pool1, pool2, pool3, ...]
  }
  
  // ระบุ shard จาก user ID
  getShard(userId) {
    const shardIndex = userId % this.shards.length;
    return this.shards[shardIndex];
  }
  
  async getUser(userId) {
    const shard = this.getShard(userId);
    return shard.query('SELECT * FROM users WHERE id = $1', [userId]);
  }
  
  async createUser(userData) {
    const userId = generateUserId();
    const shard = this.getShard(userId);
    
    await shard.query(
      'INSERT INTO users (id, name, email) VALUES ($1, $2, $3)',
      [userId, userData.name, userData.email]
    );
    
    return userId;
  }
  
  // Cross-shard query (ยุ่งยากกว่า)
  async getAllActiveUsers() {
    // ต้อง query ทุก shard แล้วรวมผล
    const results = await Promise.all(
      this.shards.map(shard => 
        shard.query('SELECT * FROM users WHERE active = true')
      )
    );
    
    return results.flatMap(r => r.rows);
  }
}

// Hash-based sharding
function hashBasedShard(key, numShards) {
  // สร้าง consistent hash
  let hash = 0;
  for (let i = 0; i < key.length; i++) {
    hash = ((hash << 5) - hash) + key.charCodeAt(i);
    hash |= 0; // Convert to 32bit integer
  }
  return Math.abs(hash) % numShards;
}

// Range-based sharding
function rangeBasedShard(userId) {
  if (userId < 10_000_000) return 'shard-1';
  if (userId < 20_000_000) return 'shard-2';
  if (userId < 30_000_000) return 'shard-3';
  return 'shard-4';
}
```

---

## ขั้นตอนที่ 1778: Message Queues กับ BullMQ

```javascript
// BullMQ - Job Queue บน Redis
import { Queue, Worker, QueueEvents } from 'bullmq';
import { createClient } from 'redis';

const connection = { host: 'localhost', port: 6379 };

// สร้าง Queues
const emailQueue = new Queue('emails', { connection });
const imageQueue = new Queue('image-processing', { connection });
const notificationQueue = new Queue('notifications', { connection });

// Email Worker
const emailWorker = new Worker(
  'emails',
  async (job) => {
    const { to, subject, body, template } = job.data;
    
    console.log(`Processing email job ${job.id}: ${subject} to ${to}`);
    
    try {
      await sendEmail({ to, subject, body, template });
      return { sent: true, timestamp: Date.now() };
    } catch (error) {
      console.error(`Email job ${job.id} failed:`, error.message);
      throw error; // BullMQ จะ retry อัตโนมัติ
    }
  },
  {
    connection,
    concurrency: 10, // process 10 jobs พร้อมกัน
    
    // Retry configuration
    defaultJobOptions: {
      attempts: 3,
      backoff: {
        type: 'exponential',
        delay: 1000,
      },
    },
  }
);

// Image Processing Worker
const imageWorker = new Worker(
  'image-processing',
  async (job) => {
    const { imageUrl, operations } = job.data;
    
    // Update progress
    await job.updateProgress(0);
    
    const image = await downloadImage(imageUrl);
    await job.updateProgress(25);
    
    const processed = await applyOperations(image, operations);
    await job.updateProgress(75);
    
    const outputUrl = await uploadProcessedImage(processed);
    await job.updateProgress(100);
    
    return { outputUrl };
  },
  {
    connection,
    concurrency: 3, // image processing ใช้ CPU มาก
  }
);

// Event Listeners
emailWorker.on('completed', (job) => {
  console.log(`Email job ${job.id} completed`);
});

emailWorker.on('failed', (job, error) => {
  console.error(`Email job ${job.id} failed after ${job.attemptsMade} attempts:`, error.message);
});

// เพิ่ม Jobs
async function sendWelcomeEmail(userId, email) {
  const job = await emailQueue.add(
    'welcome',
    { to: email, subject: 'ยินดีต้อนรับ!', template: 'welcome' },
    {
      delay: 5000, // รอ 5 วินาทีก่อนส่ง
      attempts: 3,
      removeOnComplete: 100, // เก็บ 100 completed jobs
      removeOnFail: 50,
    }
  );
  
  console.log(`Added email job ${job.id}`);
  return job;
}

// Scheduled Jobs (cron)
await emailQueue.add(
  'daily-newsletter',
  { template: 'newsletter' },
  {
    repeat: {
      cron: '0 9 * * *', // ทุกวัน 9 โมงเช้า
    },
  }
);

// Job Priority
await emailQueue.add(
  'urgent-notification',
  { to: 'admin@example.com', subject: 'Alert!' },
  { priority: 1 } // priority 1 = สูงสุด
);

await emailQueue.add(
  'low-priority-report',
  { to: 'user@example.com', subject: 'Report' },
  { priority: 10 }
);
```

---

## ขั้นตอนที่ 1779: Rate Limiting Algorithms

```javascript
// Rate Limiting Algorithms

// 1. Fixed Window
class FixedWindowRateLimiter {
  constructor(redis, limit, windowSeconds) {
    this.redis = redis;
    this.limit = limit;
    this.window = windowSeconds;
  }
  
  async check(key) {
    const now = Math.floor(Date.now() / 1000);
    const windowKey = `ratelimit:${key}:${Math.floor(now / this.window)}`;
    
    const count = await this.redis.incr(windowKey);
    await this.redis.expire(windowKey, this.window * 2);
    
    return {
      allowed: count <= this.limit,
      current: count,
      limit: this.limit,
      remaining: Math.max(0, this.limit - count),
      resetAt: (Math.floor(now / this.window) + 1) * this.window,
    };
  }
}

// 2. Sliding Window Log
class SlidingWindowLogLimiter {
  constructor(redis, limit, windowSeconds) {
    this.redis = redis;
    this.limit = limit;
    this.window = windowSeconds * 1000; // milliseconds
  }
  
  async check(key) {
    const now = Date.now();
    const windowStart = now - this.window;
    
    const pipeline = this.redis.multi();
    
    // ลบ old entries
    pipeline.zRemRangeByScore(`ratelimit:${key}`, '-inf', windowStart);
    
    // เพิ่ม current request
    pipeline.zAdd(`ratelimit:${key}`, [{ score: now, value: now.toString() }]);
    
    // นับ requests ใน window
    pipeline.zCard(`ratelimit:${key}`);
    
    // Set expiry
    pipeline.pExpire(`ratelimit:${key}`, this.window * 2);
    
    const results = await pipeline.exec();
    const count = results[2];
    
    return {
      allowed: count <= this.limit,
      current: count,
      limit: this.limit,
      remaining: Math.max(0, this.limit - count),
    };
  }
}

// 3. Token Bucket
class TokenBucketLimiter {
  constructor(redis, capacity, refillRate) {
    this.redis = redis;
    this.capacity = capacity;      // maximum tokens
    this.refillRate = refillRate;  // tokens per second
  }
  
  async check(key) {
    const now = Date.now();
    const bucketKey = `bucket:${key}`;
    
    // ดึง bucket state
    const bucketData = await this.redis.hGetAll(bucketKey);
    
    let tokens = parseFloat(bucketData.tokens) || this.capacity;
    let lastRefill = parseInt(bucketData.lastRefill) || now;
    
    // คำนวณ tokens ที่ refill
    const timePassed = (now - lastRefill) / 1000; // seconds
    const refilled = timePassed * this.refillRate;
    tokens = Math.min(this.capacity, tokens + refilled);
    
    if (tokens >= 1) {
      tokens -= 1;
      
      await this.redis.hSet(bucketKey, {
        tokens: tokens.toString(),
        lastRefill: now.toString(),
      });
      await this.redis.expire(bucketKey, 3600);
      
      return { allowed: true, tokens: Math.floor(tokens) };
    }
    
    return { allowed: false, tokens: 0 };
  }
}

// Express Middleware
function rateLimitMiddleware(limiter) {
  return async (req, res, next) => {
    const key = req.ip || req.headers['x-forwarded-for'];
    
    try {
      const result = await limiter.check(key);
      
      // ส่ง rate limit info ใน headers
      res.set({
        'X-RateLimit-Limit': result.limit,
        'X-RateLimit-Remaining': result.remaining,
        'X-RateLimit-Reset': result.resetAt,
      });
      
      if (!result.allowed) {
        return res.status(429).json({
          error: 'Too Many Requests',
          message: 'กรุณารอสักครู่ก่อนส่ง request อีกครั้ง',
          retryAfter: result.resetAt - Math.floor(Date.now() / 1000),
        });
      }
      
      next();
    } catch (error) {
      // ถ้า rate limiter fail ให้ผ่านไปก่อน (fail open)
      console.error('Rate limiter error:', error);
      next();
    }
  };
}

// ใช้งาน
const apiLimiter = new TokenBucketLimiter(redis, 100, 10); // 100 tokens, 10/sec refill
app.use('/api/', rateLimitMiddleware(apiLimiter));
```

---

## ขั้นตอนที่ 1780: API Gateway Patterns

```javascript
// API Gateway Pattern
import express from 'express';
import { createProxyMiddleware } from 'http-proxy-middleware';

const gateway = express();

// Services
const services = {
  users: 'http://users-service:3001',
  products: 'http://products-service:3002',
  orders: 'http://orders-service:3003',
  payments: 'http://payments-service:3004',
};

// Authentication Middleware
async function authenticate(req, res, next) {
  const token = req.headers.authorization?.replace('Bearer ', '');
  
  if (!token) {
    return res.status(401).json({ error: 'Token required' });
  }
  
  try {
    const user = await verifyToken(token);
    req.user = user;
    next();
  } catch {
    res.status(401).json({ error: 'Invalid token' });
  }
}

// Rate Limiting per user
async function rateLimit(req, res, next) {
  const userId = req.user?.id || req.ip;
  const result = await limiter.check(userId);
  
  if (!result.allowed) {
    return res.status(429).json({ error: 'Rate limit exceeded' });
  }
  
  next();
}

// Request ID
function requestId(req, res, next) {
  req.requestId = crypto.randomUUID();
  res.setHeader('X-Request-ID', req.requestId);
  next();
}

// Logging
function logRequest(req, res, next) {
  const start = Date.now();
  
  res.on('finish', () => {
    console.log({
      requestId: req.requestId,
      method: req.method,
      url: req.url,
      userId: req.user?.id,
      status: res.statusCode,
      duration: Date.now() - start,
    });
  });
  
  next();
}

// Global middlewares
gateway.use(requestId);
gateway.use(logRequest);
gateway.use(authenticate);
gateway.use(rateLimit);

// Route to services
gateway.use('/api/users', createProxyMiddleware({
  target: services.users,
  changeOrigin: true,
  pathRewrite: { '^/api/users': '/users' },
  on: {
    error: (err, req, res) => {
      res.status(503).json({ error: 'User service unavailable' });
    },
  },
}));

gateway.use('/api/products', createProxyMiddleware({
  target: services.products,
  changeOrigin: true,
  pathRewrite: { '^/api/products': '/products' },
}));

gateway.listen(3000);
```

---

## ขั้นตอนที่ 1781: Microservices Communication

```javascript
// Microservices Communication Patterns

// 1. Synchronous (HTTP/gRPC)
// Direct HTTP calls ระหว่าง services
async function getUserOrders(userId) {
  // User Service ถาม Order Service
  const orders = await fetch(`http://orders-service/orders?userId=${userId}`)
    .then(r => r.json());
  
  return orders;
}

// 2. Asynchronous (Message Queue)
// Services communicate through messages
class OrderService {
  constructor(messageQueue) {
    this.mq = messageQueue;
  }
  
  async createOrder(orderData) {
    // สร้าง order ใน database
    const order = await db.createOrder(orderData);
    
    // Publish event (ไม่รอ response)
    await this.mq.publish('order.created', {
      orderId: order.id,
      userId: order.userId,
      items: order.items,
      total: order.total,
    });
    
    return order;
  }
}

// InventoryService subscribes ไปฟัง events
class InventoryService {
  constructor(messageQueue) {
    // Subscribe ไปยัง order events
    messageQueue.subscribe('order.created', this.handleOrderCreated.bind(this));
  }
  
  async handleOrderCreated(event) {
    for (const item of event.items) {
      await db.decrementStock(item.productId, item.quantity);
    }
    
    console.log(`Inventory updated for order ${event.orderId}`);
  }
}

// 3. Event Sourcing Pattern
class EventStore {
  constructor(db, redis) {
    this.db = db;
    this.redis = redis;
  }
  
  async append(aggregateId, event) {
    const storedEvent = {
      id: crypto.randomUUID(),
      aggregateId,
      type: event.type,
      data: event.data,
      timestamp: new Date().toISOString(),
      version: await this.getNextVersion(aggregateId),
    };
    
    await this.db.query(
      'INSERT INTO events (id, aggregate_id, type, data, timestamp, version) VALUES ($1, $2, $3, $4, $5, $6)',
      [storedEvent.id, storedEvent.aggregateId, storedEvent.type,
       JSON.stringify(storedEvent.data), storedEvent.timestamp, storedEvent.version]
    );
    
    // Publish ไปยัง message bus
    await this.redis.publish('events', JSON.stringify(storedEvent));
    
    return storedEvent;
  }
  
  async getEvents(aggregateId, fromVersion = 0) {
    const result = await this.db.query(
      'SELECT * FROM events WHERE aggregate_id = $1 AND version > $2 ORDER BY version',
      [aggregateId, fromVersion]
    );
    return result.rows;
  }
  
  async getNextVersion(aggregateId) {
    const result = await this.db.query(
      'SELECT MAX(version) as max_version FROM events WHERE aggregate_id = $1',
      [aggregateId]
    );
    return (result.rows[0].max_version || 0) + 1;
  }
}
```

---

## ขั้นตอนที่ 1782: Observability

```javascript
// Logging, Metrics, Tracing

// 1. Structured Logging
import pino from 'pino';

const logger = pino({
  level: process.env.LOG_LEVEL || 'info',
  transport: process.env.NODE_ENV !== 'production'
    ? { target: 'pino-pretty' }
    : undefined,
  formatters: {
    level: (label) => ({ level: label }),
  },
  base: {
    pid: process.pid,
    hostname: os.hostname(),
    service: 'user-service',
    version: process.env.APP_VERSION,
  },
});

// Request logging middleware
app.use((req, res, next) => {
  const start = Date.now();
  const requestId = crypto.randomUUID();
  
  // Attach logger กับ request context
  req.log = logger.child({
    requestId,
    method: req.method,
    url: req.url,
    userId: req.user?.id,
  });
  
  req.log.info('Request started');
  
  res.on('finish', () => {
    req.log.info({
      statusCode: res.statusCode,
      duration: Date.now() - start,
    }, 'Request completed');
  });
  
  next();
});

// ใช้งาน
app.get('/users/:id', async (req, res) => {
  req.log.info({ userId: req.params.id }, 'Fetching user');
  
  try {
    const user = await getUser(req.params.id);
    req.log.info({ found: !!user }, 'User fetch complete');
    res.json(user);
  } catch (error) {
    req.log.error({ error: error.message }, 'Failed to fetch user');
    res.status(500).json({ error: 'Internal server error' });
  }
});
```

```javascript
// 2. Metrics with Prometheus
import promClient from 'prom-client';

// Register default metrics (CPU, memory, event loop)
promClient.collectDefaultMetrics();

// Custom metrics
const httpRequestDuration = new promClient.Histogram({
  name: 'http_request_duration_seconds',
  help: 'Duration of HTTP requests in seconds',
  labelNames: ['method', 'route', 'status_code'],
  buckets: [0.001, 0.005, 0.015, 0.05, 0.1, 0.2, 0.3, 0.4, 0.5, 1, 2],
});

const activeConnections = new promClient.Gauge({
  name: 'active_connections',
  help: 'Number of active connections',
});

const requestCounter = new promClient.Counter({
  name: 'http_requests_total',
  help: 'Total number of HTTP requests',
  labelNames: ['method', 'route', 'status_code'],
});

// Middleware
app.use((req, res, next) => {
  const timer = httpRequestDuration.startTimer();
  
  res.on('finish', () => {
    const labels = {
      method: req.method,
      route: req.route?.path || 'unknown',
      status_code: res.statusCode,
    };
    
    timer(labels);
    requestCounter.inc(labels);
  });
  
  next();
});

// Metrics endpoint
app.get('/metrics', async (req, res) => {
  res.set('Content-Type', promClient.register.contentType);
  res.end(await promClient.register.metrics());
});
```

```javascript
// 3. Distributed Tracing (OpenTelemetry)
import { NodeSDK } from '@opentelemetry/sdk-node';
import { getNodeAutoInstrumentations } from '@opentelemetry/auto-instrumentations-node';
import { OTLPTraceExporter } from '@opentelemetry/exporter-trace-otlp-http';

const sdk = new NodeSDK({
  traceExporter: new OTLPTraceExporter({
    url: 'http://jaeger:4318/v1/traces',
  }),
  instrumentations: [getNodeAutoInstrumentations()],
  serviceName: 'user-service',
  serviceVersion: '1.0.0',
});

sdk.start();

// Manual spans
import { trace, SpanStatusCode } from '@opentelemetry/api';

const tracer = trace.getTracer('user-service');

async function getUserWithTracing(userId) {
  return tracer.startActiveSpan('getUserById', async (span) => {
    span.setAttribute('user.id', userId);
    
    try {
      // Check cache
      const cached = await tracer.startActiveSpan('cache.get', async (cacheSpan) => {
        cacheSpan.setAttribute('cache.key', `user:${userId}`);
        const result = await redis.get(`user:${userId}`);
        cacheSpan.setAttribute('cache.hit', !!result);
        cacheSpan.end();
        return result;
      });
      
      if (cached) {
        span.setAttribute('cache.hit', true);
        span.end();
        return JSON.parse(cached);
      }
      
      // Query DB
      const user = await tracer.startActiveSpan('db.query', async (dbSpan) => {
        dbSpan.setAttribute('db.statement', 'SELECT * FROM users WHERE id = ?');
        const result = await db.findById(userId);
        dbSpan.end();
        return result;
      });
      
      span.setAttribute('user.found', !!user);
      span.end();
      return user;
      
    } catch (error) {
      span.recordException(error);
      span.setStatus({ code: SpanStatusCode.ERROR, message: error.message });
      span.end();
      throw error;
    }
  });
}
```

---

## ขั้นตอนที่ 1783: Designing URL Shortener

```javascript
// URL Shortener System Design
// Requirements:
// - 100M URLs shortened per day
// - 10x reads vs writes
// - URLs expire after 1 year
// - Custom aliases
// - Analytics

// API Design:
// POST /shorten -> { shortCode }
// GET /:shortCode -> 302 Redirect to long URL
// GET /:shortCode/stats -> Analytics

// Calculations:
// Writes: 100M / 86400 ≈ 1157 writes/second
// Reads: 1157 * 10 = 11570 reads/second
// Storage: 100M * 365 days * 500 bytes = ~18TB/year

// Short Code Generator
function generateShortCode(length = 7) {
  const chars = 'abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789';
  let code = '';
  for (let i = 0; i < length; i++) {
    code += chars[Math.floor(Math.random() * chars.length)];
  }
  return code; // 62^7 = 3.5 trillion unique codes
}

// Base62 encoding จาก ID
function toBase62(num) {
  const chars = '0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ';
  let result = '';
  
  while (num > 0) {
    result = chars[num % 62] + result;
    num = Math.floor(num / 62);
  }
  
  return result.padStart(7, '0');
}

// Service
class URLShortenerService {
  constructor(db, redis) {
    this.db = db;
    this.redis = redis;
  }
  
  async shorten(longUrl, options = {}) {
    const { customAlias, userId, expiresIn = 365 * 24 * 3600 } = options;
    
    // Validate URL
    try {
      new URL(longUrl);
    } catch {
      throw new Error('Invalid URL');
    }
    
    // Check if URL already shortened
    const existing = await this.db.query(
      'SELECT short_code FROM urls WHERE long_url = $1 AND user_id = $2',
      [longUrl, userId || null]
    );
    
    if (existing.rows.length > 0) {
      return { shortCode: existing.rows[0].short_code };
    }
    
    // Generate short code
    let shortCode = customAlias || generateShortCode();
    
    // Check collision
    while (await this.exists(shortCode)) {
      shortCode = generateShortCode();
    }
    
    // Save to database
    const expiresAt = new Date(Date.now() + expiresIn * 1000);
    
    await this.db.query(
      'INSERT INTO urls (short_code, long_url, user_id, expires_at, created_at) VALUES ($1, $2, $3, $4, NOW())',
      [shortCode, longUrl, userId || null, expiresAt]
    );
    
    // Cache ใน Redis
    await this.redis.setEx(
      `url:${shortCode}`,
      Math.min(expiresIn, 3600), // cache ไม่เกิน 1 ชั่วโมง
      longUrl
    );
    
    return { shortCode, shortUrl: `https://short.ly/${shortCode}` };
  }
  
  async resolve(shortCode) {
    // Cache-first
    const cached = await this.redis.get(`url:${shortCode}`);
    if (cached) {
      // Async analytics update
      this.recordClick(shortCode).catch(() => {});
      return cached;
    }
    
    // Database lookup
    const result = await this.db.query(
      'SELECT long_url, expires_at FROM urls WHERE short_code = $1',
      [shortCode]
    );
    
    if (!result.rows.length) {
      return null;
    }
    
    const { long_url, expires_at } = result.rows[0];
    
    // Check expiry
    if (new Date(expires_at) < new Date()) {
      return null; // expired
    }
    
    // Cache
    const ttl = Math.floor((new Date(expires_at) - new Date()) / 1000);
    await this.redis.setEx(`url:${shortCode}`, Math.min(ttl, 3600), long_url);
    
    // Analytics
    this.recordClick(shortCode).catch(() => {});
    
    return long_url;
  }
  
  async recordClick(shortCode) {
    const today = new Date().toISOString().split('T')[0];
    
    // Increment counters
    await Promise.all([
      this.redis.incr(`clicks:${shortCode}:total`),
      this.redis.incr(`clicks:${shortCode}:${today}`),
    ]);
    
    // Flush to DB periodically (done by a background job)
  }
  
  async getStats(shortCode) {
    const [total, dailyClicks] = await Promise.all([
      this.redis.get(`clicks:${shortCode}:total`),
      this.getDailyClicks(shortCode, 7),
    ]);
    
    return {
      shortCode,
      totalClicks: parseInt(total) || 0,
      dailyClicks,
    };
  }
  
  async exists(shortCode) {
    const result = await this.db.query(
      'SELECT 1 FROM urls WHERE short_code = $1',
      [shortCode]
    );
    return result.rows.length > 0;
  }
}
```

---

## ขั้นตอนที่ 1784: Designing Notification System

```javascript
// Notification System Design
// Requirements:
// - Push notifications (mobile, web)
// - Email notifications
// - SMS
// - In-app notifications
// - Digest (รวมหลาย notifications เป็น 1 email)
// - User preferences

// Notification Types & Channels
const NOTIFICATION_TYPES = {
  ORDER_CREATED: { channels: ['push', 'email'] },
  ORDER_SHIPPED: { channels: ['push', 'email', 'sms'] },
  ORDER_DELIVERED: { channels: ['push', 'email'] },
  PAYMENT_FAILED: { channels: ['push', 'email', 'sms'] },
  PROMOTION: { channels: ['push', 'email'] },
  SOCIAL: { channels: ['push', 'inapp'] },
};

class NotificationService {
  constructor({ emailQueue, pushQueue, smsQueue, db, redis }) {
    this.emailQueue = emailQueue;
    this.pushQueue = pushQueue;
    this.smsQueue = smsQueue;
    this.db = db;
    this.redis = redis;
  }
  
  async sendNotification(userId, type, data) {
    // ดึง user preferences
    const prefs = await this.getUserPreferences(userId);
    const channels = NOTIFICATION_TYPES[type]?.channels || ['inapp'];
    
    // บันทึก in-app notification
    const notification = await this.saveNotification(userId, type, data);
    
    // ส่งตาม channels ที่ user ต้องการ
    const sendPromises = channels
      .filter(channel => prefs[channel] !== false)
      .map(channel => this.sendToChannel(channel, userId, notification));
    
    await Promise.allSettled(sendPromises);
    
    return notification;
  }
  
  async sendToChannel(channel, userId, notification) {
    switch (channel) {
      case 'email':
        return this.emailQueue.add('notification', {
          to: await this.getUserEmail(userId),
          subject: notification.title,
          template: notification.type.toLowerCase(),
          data: notification.data,
        });
      
      case 'push':
        const tokens = await this.getUserPushTokens(userId);
        return Promise.all(tokens.map(token =>
          this.pushQueue.add('push', {
            token,
            title: notification.title,
            body: notification.body,
            data: { notificationId: notification.id },
          })
        ));
      
      case 'sms':
        return this.smsQueue.add('sms', {
          phone: await this.getUserPhone(userId),
          message: notification.body,
        });
      
      case 'inapp':
        // Real-time via WebSocket
        this.broadcastToUser(userId, 'notification', notification);
        break;
    }
  }
  
  async getUserPreferences(userId) {
    const cacheKey = `notif-prefs:${userId}`;
    const cached = await this.redis.get(cacheKey);
    
    if (cached) return JSON.parse(cached);
    
    const result = await this.db.query(
      'SELECT preferences FROM user_notifications WHERE user_id = $1',
      [userId]
    );
    
    const prefs = result.rows[0]?.preferences || {
      email: true,
      push: true,
      sms: false,
      inapp: true,
    };
    
    await this.redis.setEx(cacheKey, 300, JSON.stringify(prefs));
    return prefs;
  }
  
  // Batch notifications (Digest)
  async scheduleDigest(userId, notifications) {
    const digestKey = `digest:${userId}`;
    
    // เพิ่มไปยัง digest list
    await this.redis.rPush(digestKey, JSON.stringify(notifications));
    
    // Schedule digest email ถ้ายังไม่มี scheduled
    const scheduled = await this.redis.get(`digest-scheduled:${userId}`);
    if (!scheduled) {
      // ส่ง digest 1 ชั่วโมงหลังจาก notification แรก
      await this.emailQueue.add(
        'digest',
        { userId },
        { delay: 3600 * 1000 }
      );
      
      await this.redis.setEx(`digest-scheduled:${userId}`, 3600, '1');
    }
  }
}
```

---

## ขั้นตอนที่ 1785: Designing Real-time Chat System

```javascript
// Real-time Chat System Design
// Requirements:
// - 1-on-1 messages
// - Group chat (up to 100 members)
// - Message history
// - Read receipts
// - Online status
// - File sharing

// Architecture:
// Client <-> Load Balancer (Sticky) <-> Chat Servers <-> Redis Pub/Sub
//                                                     <-> Message Queue
//                                                     <-> Database

// Message Schema
const MessageSchema = {
  id: 'uuid',
  conversationId: 'uuid',
  senderId: 'uuid',
  content: 'string',
  type: 'text | image | file | video',
  status: 'sending | sent | delivered | read',
  createdAt: 'timestamp',
  metadata: {
    fileUrl: 'string?',
    fileName: 'string?',
    fileSize: 'number?',
    mimeType: 'string?',
  },
};

// Chat Service
class ChatService {
  constructor({ io, db, redis, fileStorage }) {
    this.io = io;
    this.db = db;
    this.redis = redis;
    this.fileStorage = fileStorage;
  }
  
  async sendMessage(senderId, conversationId, content, type = 'text') {
    // สร้าง message
    const message = {
      id: crypto.randomUUID(),
      conversationId,
      senderId,
      content,
      type,
      status: 'sent',
      createdAt: new Date().toISOString(),
    };
    
    // บันทึก message (async)
    const savePromise = this.db.query(
      `INSERT INTO messages (id, conversation_id, sender_id, content, type, status, created_at)
       VALUES ($1, $2, $3, $4, $5, $6, $7)`,
      [message.id, message.conversationId, message.senderId, 
       message.content, message.type, message.status, message.createdAt]
    );
    
    // Cache recent messages ใน Redis
    const cacheKey = `messages:${conversationId}:recent`;
    const cachePromise = this.redis.multi()
      .lPush(cacheKey, JSON.stringify(message))
      .lTrim(cacheKey, 0, 99) // เก็บ 100 messages ล่าสุด
      .expire(cacheKey, 3600)
      .exec();
    
    await Promise.all([savePromise, cachePromise]);
    
    // Deliver ไปยัง recipients
    const recipients = await this.getConversationMembers(conversationId);
    
    for (const recipientId of recipients) {
      if (recipientId !== senderId) {
        // ถ้า online ส่งผ่าน WebSocket
        const isOnline = await this.isUserOnline(recipientId);
        
        if (isOnline) {
          this.io.to(`user:${recipientId}`).emit('newMessage', message);
        } else {
          // ส่ง push notification
          await this.sendPushNotification(recipientId, message);
        }
        
        // ทำเครื่องหมาย delivered
        this.markDelivered(message.id, recipientId).catch(() => {});
      }
    }
    
    return message;
  }
  
  async getMessages(conversationId, before = null, limit = 50) {
    // ลองดึงจาก cache ก่อน
    if (!before) {
      const cacheKey = `messages:${conversationId}:recent`;
      const cached = await this.redis.lRange(cacheKey, 0, limit - 1);
      
      if (cached.length >= limit) {
        return cached.map(m => JSON.parse(m)).reverse();
      }
    }
    
    // ดึงจาก database
    const query = before
      ? `SELECT * FROM messages WHERE conversation_id = $1 AND created_at < $2
         ORDER BY created_at DESC LIMIT $3`
      : `SELECT * FROM messages WHERE conversation_id = $1
         ORDER BY created_at DESC LIMIT $2`;
    
    const params = before 
      ? [conversationId, before, limit] 
      : [conversationId, limit];
    
    const result = await this.db.query(query, params);
    return result.rows.reverse();
  }
  
  async isUserOnline(userId) {
    const online = await this.redis.get(`online:${userId}`);
    return !!online;
  }
  
  async setUserOnline(userId, socketId) {
    await this.redis.setEx(`online:${userId}`, 30, socketId);
    
    // Broadcast online status ไปยัง contacts
    const contacts = await this.getUserContacts(userId);
    contacts.forEach(contactId => {
      this.io.to(`user:${contactId}`).emit('userOnline', { userId });
    });
  }
}
```

---

## ขั้นตอนที่ 1786: Designing Rate Limiter System

```javascript
// Distributed Rate Limiter System Design
// ใช้ Redis สำหรับ distributed counting

class DistributedRateLimiter {
  constructor(redis) {
    this.redis = redis;
  }
  
  // Sliding Window Counter with Redis
  async slidingWindowLimit(key, limit, windowMs) {
    const now = Date.now();
    const windowStart = now - windowMs;
    const bucketKey = `rl:${key}`;
    
    // Lua script สำหรับ atomic operations
    const script = `
      local key = KEYS[1]
      local now = tonumber(ARGV[1])
      local window_start = tonumber(ARGV[2])
      local limit = tonumber(ARGV[3])
      local window_ms = tonumber(ARGV[4])
      
      -- ลบ old entries
      redis.call('ZREMRANGEBYSCORE', key, '-inf', window_start)
      
      -- นับ entries ปัจจุบัน
      local count = redis.call('ZCARD', key)
      
      if count < limit then
        -- เพิ่ม entry ใหม่
        redis.call('ZADD', key, now, now .. '-' .. math.random())
        redis.call('PEXPIRE', key, window_ms * 2)
        return {1, count + 1, limit - count - 1}
      else
        -- หา earliest entry
        local earliest = redis.call('ZRANGE', key, 0, 0, 'WITHSCORES')
        local reset_at = tonumber(earliest[2]) + window_ms
        return {0, count, 0, reset_at}
      end
    `;
    
    const result = await this.redis.eval(
      script,
      1,           // number of keys
      bucketKey,   // key
      now.toString(),
      windowStart.toString(),
      limit.toString(),
      windowMs.toString()
    );
    
    const [allowed, current, remaining, resetAt] = result;
    
    return {
      allowed: allowed === 1,
      current,
      remaining,
      resetAt: resetAt || null,
      limit,
    };
  }
  
  // Per-endpoint rate limits
  async checkAPI(userId, endpoint) {
    const rules = [
      // Global user limit
      { key: `user:${userId}`, limit: 1000, window: 60 * 1000 },
      // Per-endpoint limit
      { key: `user:${userId}:${endpoint}`, limit: 100, window: 60 * 1000 },
      // IP-based limit
      { key: `ip:${userId}`, limit: 500, window: 60 * 1000 },
    ];
    
    for (const rule of rules) {
      const result = await this.slidingWindowLimit(rule.key, rule.limit, rule.window);
      
      if (!result.allowed) {
        return {
          allowed: false,
          reason: `Rate limit exceeded for ${rule.key}`,
          resetAt: result.resetAt,
          retryAfter: Math.ceil((result.resetAt - Date.now()) / 1000),
        };
      }
    }
    
    return { allowed: true };
  }
}
```

---

## ขั้นตอนที่ 1787: Service Mesh Overview

```javascript
// Service Mesh Concepts
// Service mesh ช่วยจัดการ:
// - Service discovery
// - Load balancing
// - Circuit breaking
// - Retry logic
// - Observability
// - Security (mTLS)

// Circuit Breaker Pattern
class CircuitBreaker {
  constructor(options = {}) {
    this.threshold = options.threshold || 5;       // failures before open
    this.timeout = options.timeout || 60000;       // ms before half-open
    this.successThreshold = options.successThreshold || 2; // successes to close
    
    this.state = 'CLOSED';
    this.failureCount = 0;
    this.successCount = 0;
    this.nextAttempt = Date.now();
  }
  
  async execute(fn) {
    if (this.state === 'OPEN') {
      if (Date.now() < this.nextAttempt) {
        throw new Error('Circuit breaker is OPEN');
      }
      
      // Try half-open
      this.state = 'HALF_OPEN';
    }
    
    try {
      const result = await fn();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }
  
  onSuccess() {
    this.failureCount = 0;
    
    if (this.state === 'HALF_OPEN') {
      this.successCount++;
      
      if (this.successCount >= this.successThreshold) {
        this.state = 'CLOSED';
        this.successCount = 0;
        console.log('Circuit Breaker: HALF_OPEN -> CLOSED');
      }
    }
  }
  
  onFailure() {
    this.failureCount++;
    this.successCount = 0;
    
    if (this.failureCount >= this.threshold || this.state === 'HALF_OPEN') {
      this.state = 'OPEN';
      this.nextAttempt = Date.now() + this.timeout;
      console.log(`Circuit Breaker: -> OPEN (next attempt: ${new Date(this.nextAttempt).toISOString()})`);
    }
  }
}

// Retry with Exponential Backoff
async function withRetry(fn, options = {}) {
  const {
    maxAttempts = 3,
    baseDelay = 1000,
    maxDelay = 30000,
    shouldRetry = (error) => true,
  } = options;
  
  let lastError;
  
  for (let attempt = 1; attempt <= maxAttempts; attempt++) {
    try {
      return await fn();
    } catch (error) {
      lastError = error;
      
      if (attempt === maxAttempts || !shouldRetry(error)) {
        throw error;
      }
      
      const delay = Math.min(baseDelay * Math.pow(2, attempt - 1), maxDelay);
      const jitter = Math.random() * delay * 0.1; // 10% jitter
      
      console.log(`Attempt ${attempt} failed, retrying in ${Math.round(delay + jitter)}ms...`);
      await new Promise(r => setTimeout(r, delay + jitter));
    }
  }
  
  throw lastError;
}

// ใช้งาน
const circuitBreaker = new CircuitBreaker({ threshold: 5, timeout: 60000 });

async function callExternalService(data) {
  return circuitBreaker.execute(async () => {
    return withRetry(
      () => fetch('https://external-api.example.com/endpoint', {
        method: 'POST',
        body: JSON.stringify(data),
        signal: AbortSignal.timeout(5000),
      }).then(r => r.json()),
      {
        maxAttempts: 3,
        shouldRetry: (err) => err.name !== 'AbortError', // ไม่ retry timeout
      }
    );
  });
}
```

---

## ขั้นตอนที่ 1788: Consistency Patterns

```javascript
// CAP Theorem
// ระบบ distributed เลือกได้แค่ 2 จาก 3:
// - Consistency: ทุก node เห็นข้อมูลเดียวกัน
// - Availability: ทุก request ได้รับ response
// - Partition Tolerance: ระบบทำงานแม้ network ขาด

// 1. Strong Consistency (CP)
// ทุก read เห็น write ล่าสุดเสมอ
// ตัวอย่าง: bank transactions, inventory

async function decrementStock(productId, quantity) {
  // ใช้ SERIALIZABLE transaction
  return db.transaction(async (trx) => {
    // Lock สำหรับ update
    const result = await trx.raw(
      'SELECT stock FROM products WHERE id = ? FOR UPDATE',
      [productId]
    );
    
    const currentStock = result[0].stock;
    
    if (currentStock < quantity) {
      throw new Error('Insufficient stock');
    }
    
    await trx('products')
      .where({ id: productId })
      .update({ stock: currentStock - quantity });
    
    return currentStock - quantity;
  });
}

// 2. Eventual Consistency (AP)
// ข้อมูลอาจไม่ตรงกันชั่วคราว แต่ converge ในที่สุด
// ตัวอย่าง: social media likes, view counts, shopping cart

async function addToCart(userId, productId, quantity) {
  // เขียนลง Redis ทันที (fast)
  await redis.hSet(`cart:${userId}`, productId, quantity);
  
  // Sync ลง database async (eventual)
  await cartSyncQueue.add('syncCart', { userId, productId, quantity });
  
  return { success: true };
}

// 3. Read-your-writes Consistency
// User เห็น write ของตัวเองเสมอ

async function postComment(userId, postId, content) {
  const comment = await db.createComment({ userId, postId, content });
  
  // Invalidate user's cache
  await redis.del(`comments:${postId}:user:${userId}`);
  
  return comment;
}

async function getComments(postId, requestUserId) {
  // ใช้ primary สำหรับ user ที่เพิ่งเขียน
  const useReplica = !(await userJustWrote(requestUserId, postId));
  
  const db = useReplica ? readPool : writePool;
  return db.query('SELECT * FROM comments WHERE post_id = $1', [postId]);
}
```

---

## ขั้นตอนที่ 1789: System Design Interview Tips

```
Framework สำหรับ System Design Interview:

1. Clarify Requirements (5 นาที)
   - Functional requirements: อะไรที่ระบบต้องทำ?
   - Non-functional requirements: scale, latency, availability?
   - Constraints: ข้อจำกัดที่มี?

2. Estimation (5 นาที)
   - DAU/MAU
   - Read/Write ratio
   - Data storage
   - Bandwidth
   
3. High-level Design (10 นาที)
   - API endpoints
   - Core components
   - Data flow
   
4. Detailed Design (15 นาที)
   - เลือก component ที่ interviewer สนใจ
   - ลงรายละเอียด
   - พูดถึง trade-offs
   
5. Scale (5 นาที)
   - Bottlenecks ที่เห็น
   - วิธีแก้ไข
```

```javascript
// Common System Design Components

const COMPONENTS = {
  // Storage
  relationalDB: 'PostgreSQL, MySQL - ACID, complex queries',
  noSQL: 'MongoDB, DynamoDB - flexible schema, horizontal scale',
  redis: 'Cache, sessions, real-time',
  blob: 'S3, GCS - files, images, videos',
  search: 'Elasticsearch - full-text search',
  
  // Compute
  apiGateway: 'Entry point, auth, rate limiting',
  loadBalancer: 'Traffic distribution',
  cdnEdge: 'Static assets, edge caching',
  
  // Communication
  messageQueue: 'Kafka, RabbitMQ, BullMQ - async processing',
  webSocket: 'Real-time bidirectional',
  sse: 'Server to client streaming',
  
  // Observability
  logging: 'ELK Stack, CloudWatch',
  metrics: 'Prometheus, Grafana',
  tracing: 'Jaeger, Zipkin, Datadog',
};

// Trade-off Matrix
const TRADEOFFS = {
  sqlVsNoSQL: {
    sql: 'Strong consistency, complex queries, ACID',
    noSQL: 'Flexible schema, horizontal scaling, eventual consistency',
    chooseSQL: 'financial data, complex relationships',
    chooseNoSQL: 'user profiles, activity logs, high write throughput',
  },
  
  cacheVsDB: {
    cache: 'Fast (< 1ms), limited space, volatile',
    db: 'Slow (> 10ms), unlimited, persistent',
    strategy: 'Cache frequently read, rarely changed data',
  },
  
  syncVsAsync: {
    sync: 'Simple, immediate consistency',
    async: 'Better performance, complex error handling',
    chooseSync: 'Real-time responses, simple workflows',
    chooseAsync: 'Heavy processing, external services, bulk operations',
  },
};
```

---

## ขั้นตอนที่ 1790: Complete System Design Example

```javascript
// Twitter-like System - Full Design

// System Requirements:
// - 100M DAU
// - 500M tweets/day
// - 5:95 write:read ratio
// - Timeline generation

// Capacity:
// Writes: 500M / 86400 ≈ 5787 tweets/sec
// Reads: 5787 * 19 ≈ 110k reads/sec
// Storage: 500M * 300 bytes = 150GB/day

// Architecture:
/*
Client -> CDN -> API Gateway -> Services:
  - Tweet Service -> Cassandra (tweets)
  - User Service -> PostgreSQL (users)
  - Timeline Service -> Redis (timelines)
  - Search Service -> Elasticsearch
  - Notification Service -> Kafka -> Push/Email/SMS
  - Media Service -> S3 + CDN
*/

// Tweet Service
class TweetService {
  async createTweet(userId, content, mediaUrls = []) {
    const tweet = {
      id: this.generateSnowflakeId(),
      userId,
      content,
      mediaUrls,
      likes: 0,
      retweets: 0,
      createdAt: Date.now(),
    };
    
    // Write to Cassandra (high write throughput)
    await cassandra.execute(
      'INSERT INTO tweets (id, user_id, content, media_urls, created_at) VALUES (?, ?, ?, ?, ?)',
      [tweet.id, tweet.userId, tweet.content, tweet.mediaUrls, tweet.createdAt]
    );
    
    // Publish event
    await kafka.produce('tweet.created', tweet);
    
    return tweet;
  }
  
  // Snowflake ID: timestamp + machine_id + sequence
  generateSnowflakeId() {
    const timestamp = BigInt(Date.now());
    const machineId = BigInt(process.env.MACHINE_ID || 1);
    const sequence = BigInt(this.sequence++ % 4096);
    
    return (timestamp << 22n) | (machineId << 12n) | sequence;
  }
}

// Timeline Service (Fan-out approach)
class TimelineService {
  constructor({ redis, db, kafka }) {
    this.redis = redis;
    this.db = db;
    
    // Subscribe to tweet events
    kafka.subscribe('tweet.created', this.handleNewTweet.bind(this));
  }
  
  async handleNewTweet(tweet) {
    // ดึง followers ของ tweeter
    const followers = await this.getFollowers(tweet.userId);
    
    // Fan-out write: เพิ่ม tweet ไปยัง timeline ของทุก follower
    // (สำหรับ user ทั่วไปที่มี followers ไม่มาก)
    if (followers.length <= 10000) {
      const pipeline = this.redis.multi();
      
      followers.forEach(followerId => {
        const timelineKey = `timeline:${followerId}`;
        pipeline.lPush(timelineKey, JSON.stringify({
          tweetId: tweet.id,
          userId: tweet.userId,
          timestamp: tweet.createdAt,
        }));
        pipeline.lTrim(timelineKey, 0, 999); // เก็บ 1000 entries
      });
      
      await pipeline.exec();
    }
    // สำหรับ celebrities (มี followers มาก) ใช้ fan-out on read
  }
  
  async getTimeline(userId, cursor = null, limit = 20) {
    const timelineKey = `timeline:${userId}`;
    
    // ดึงจาก Redis
    let items = await this.redis.lRange(timelineKey, 0, limit + 10);
    
    if (items.length < limit) {
      // Backfill จาก database ถ้า cache ไม่พอ
      items = await this.getTimelineFromDB(userId, limit);
      
      // Warm up cache
      if (items.length > 0) {
        const pipeline = this.redis.multi();
        pipeline.del(timelineKey);
        items.slice(0, 1000).forEach(item => {
          pipeline.rPush(timelineKey, JSON.stringify(item));
        });
        pipeline.expire(timelineKey, 7 * 24 * 3600); // 1 สัปดาห์
        await pipeline.exec();
      }
    }
    
    // Merge กับ tweets จาก celebrities (fan-out on read)
    const celebrities = await this.getFollowedCelebrities(userId);
    
    if (celebrities.length > 0) {
      const celebTweets = await this.getCelebrityTweets(celebrities, limit);
      items = this.mergeSortedTimelines(items, celebTweets, limit);
    }
    
    // Hydrate tweets (ดึงรายละเอียด)
    const tweetIds = items.map(i => JSON.parse(i).tweetId);
    const tweets = await this.getTweetsByIds(tweetIds);
    
    return tweets;
  }
}
```

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: Design Pastebin
ออกแบบระบบ Pastebin ที่:
- สร้าง/แก้ไข/ลบ pastes
- Syntax highlighting
- Expiry
- Private/Public pastes
- 10M DAU, 1M pastes/day

### แบบฝึกหัดที่ 2: Design Instagram
ออกแบบ Instagram ที่:
- อัปโหลดรูปภาพ/วิดีโอ
- News feed
- Stories (24h expire)
- Notifications
- 500M DAU

### แบบฝึกหัดที่ 3: Design Uber
ออกแบบ Uber ที่:
- Matching riders/drivers
- Real-time location tracking
- Pricing
- Payment
- 5M rides/day

### แบบฝึกหัดที่ 4: Design Distributed Cache
ออกแบบ distributed cache (เหมือน Redis) ที่:
- GET/SET operations
- TTL support
- Eviction policy (LRU)
- Consistent hashing
- Replication

### แบบฝึกหัดที่ 5: Design Search Engine
ออกแบบ search engine (เหมือน Google) ที่:
- Web crawler
- Indexing
- Ranking (PageRank)
- Query processing
- Auto-complete

---

## สรุป

System Design เป็นทักษะที่ต้องฝึกฝนด้วยการออกแบบระบบจริงๆ และศึกษา case studies จากบริษัทใหญ่:

| Company | Blog | Technologies |
|---------|------|--------------|
| Netflix | netflixtechblog.com | Microservices, Cassandra, Kafka |
| Uber | eng.uber.com | Go, MySQL, Redis, Kafka |
| Twitter | engineering.twitter.com | Scala, Cassandra, Manhattan |
| Airbnb | medium.com/airbnb-engineering | Ruby, MySQL, Kafka, Spark |
| Meta | engineering.fb.com | PHP/Hack, MySQL, HBase, Cassandra |

**แนวทางการศึกษาต่อ:**
1. อ่าน "Designing Data-Intensive Applications" โดย Martin Kleppmann
2. อ่าน "System Design Interview" โดย Alex Xu
3. ฝึก design ระบบต่างๆ บน paper ก่อน code
4. ศึกษา open source projects ที่ scale ดี (Redis, Kafka, Nginx)
5. สร้างของจริงเพื่อเจอปัญหา production จริงๆ

นี่คือส่วนสุดท้ายของ JavaScript Course Level Advanced! คุณได้เรียนรู้เนื้อหาครบถ้วนตั้งแต่พื้นฐานจนถึง System Design ระดับ Production แล้ว!
