# Part 87: Edge Computing กับ JavaScript
## ขั้นตอนที่ 1711-1730: การประมวลผลที่ขอบเครือข่าย

Edge Computing คือการย้ายการประมวลผลไปยังจุดที่ใกล้กับผู้ใช้ที่สุด แทนที่จะส่งทุก request ไปยัง data center กลาง ทำให้ latency ต่ำลงอย่างมาก

---

## ขั้นตอนที่ 1711: Edge Computing คืออะไร

### ความแตกต่างระหว่าง Traditional vs Edge

```
Traditional (CSR/SSR):
ผู้ใช้ (Bangkok) → Internet → Server (Singapore/US) → Response
Latency: 150-300ms

Edge Computing:
ผู้ใช้ (Bangkok) → Edge Node (Bangkok/Singapore) → Response
Latency: 10-30ms
```

### Edge Infrastructure Providers

```
Cloudflare Workers:
- 300+ PoP (Point of Presence) ทั่วโลก
- JavaScript/WebAssembly
- V8 isolates (ไม่ใช่ containers)

Vercel Edge Functions:
- ใช้ Cloudflare infrastructure
- Integrated กับ Next.js
- Edge Middleware

Deno Deploy:
- Deno runtime
- TypeScript first
- Global distribution

Fastly Compute@Edge:
- WebAssembly focused
- Rust/Go/JavaScript
- Enterprise grade
```

---

## ขั้นตอนที่ 1712: Edge Runtime vs Node.js Runtime

### สิ่งที่ Edge Runtime มี

```javascript
// Web APIs ที่ Edge Runtime รองรับ
// (เหมือนกับ Browser + เพิ่มเติม)

// ✅ Fetch API
const response = await fetch('https://api.example.com/data');
const data = await response.json();

// ✅ Web Crypto API
const key = await crypto.subtle.generateKey(
  { name: 'AES-GCM', length: 256 },
  true,
  ['encrypt', 'decrypt']
);

// ✅ TextEncoder/TextDecoder
const encoder = new TextEncoder();
const data2 = encoder.encode('สวัสดี');

// ✅ URL API
const url = new URL('https://example.com/path?foo=bar');
console.log(url.hostname); // example.com
console.log(url.searchParams.get('foo')); // bar

// ✅ Streams API
const stream = new ReadableStream({
  start(controller) {
    controller.enqueue(new Uint8Array([1, 2, 3]));
    controller.close();
  }
});

// ✅ Request/Response (Web standard)
const req = new Request('https://example.com', {
  method: 'POST',
  body: JSON.stringify({ hello: 'world' }),
  headers: { 'Content-Type': 'application/json' },
});

// ✅ Cache API (บาง platform)
const cache = await caches.open('my-cache');
```

### สิ่งที่ Edge Runtime ไม่มี

```javascript
// ❌ Node.js built-in modules
const fs = require('fs');           // ไม่มี
const path = require('path');       // ไม่มี
const os = require('os');           // ไม่มี
const child_process = require('child_process'); // ไม่มี

// ❌ Node.js APIs
process.env                          // มีแค่บาง platform
Buffer                               // ไม่มี (ใช้ ArrayBuffer แทน)

// ❌ Heavy npm packages ที่ใช้ Node.js internals
const bcrypt = require('bcrypt');   // ใช้ไม่ได้ (ใช้ crypto ของ Web แทน)
const fs = require('fs-extra');     // ใช้ไม่ได้

// ✅ ทางเลือกที่ใช้งานได้บน Edge
// แทน Buffer ใช้ ArrayBuffer + Uint8Array
const uint8 = new Uint8Array([1, 2, 3]);

// แทน bcrypt ใช้ Web Crypto
async function hashPassword(password) {
  const encoder = new TextEncoder();
  const data = encoder.encode(password);
  const hash = await crypto.subtle.digest('SHA-256', data);
  return btoa(String.fromCharCode(...new Uint8Array(hash)));
}
```

---

## ขั้นตอนที่ 1713: Cloudflare Workers - Setup และ Hello World

### Installation

```bash
# ติดตั้ง Wrangler CLI
npm install -g wrangler

# Login
wrangler login

# สร้าง project ใหม่
npm create cloudflare@latest my-worker
cd my-worker

# development
wrangler dev

# deploy
wrangler deploy
```

### Hello World Worker

```javascript
// src/index.js
export default {
  // fetch handler: รับ HTTP requests
  async fetch(request, env, ctx) {
    const url = new URL(request.url);
    
    if (url.pathname === '/') {
      return new Response('สวัสดี จาก Cloudflare Workers!', {
        headers: {
          'Content-Type': 'text/plain; charset=utf-8',
        },
      });
    }
    
    if (url.pathname === '/json') {
      return Response.json({
        message: 'Hello from Edge!',
        timestamp: Date.now(),
        cf: request.cf, // Cloudflare metadata
      });
    }
    
    return new Response('Not Found', { status: 404 });
  },
};
```

```toml
# wrangler.toml
name = "my-worker"
main = "src/index.js"
compatibility_date = "2024-01-01"

[vars]
MY_VAR = "hello"
```

---

## ขั้นตอนที่ 1714: Request/Response API

```javascript
// Cloudflare Workers - Full HTTP handling
export default {
  async fetch(request, env, ctx) {
    const url = new URL(request.url);
    const method = request.method;
    
    // Router
    if (method === 'GET' && url.pathname === '/api/users') {
      return handleGetUsers(request, env);
    }
    
    if (method === 'POST' && url.pathname === '/api/users') {
      return handleCreateUser(request, env);
    }
    
    return new Response('Not Found', { status: 404 });
  },
};

async function handleGetUsers(request, env) {
  // อ่าน query parameters
  const url = new URL(request.url);
  const page = parseInt(url.searchParams.get('page')) || 1;
  const limit = parseInt(url.searchParams.get('limit')) || 10;
  
  // อ่าน headers
  const authHeader = request.headers.get('Authorization');
  if (!authHeader?.startsWith('Bearer ')) {
    return new Response('Unauthorized', { status: 401 });
  }
  
  const users = [
    { id: 1, name: 'สมชาย', email: 'somchai@example.com' },
    { id: 2, name: 'สมหญิง', email: 'somying@example.com' },
  ];
  
  return new Response(JSON.stringify({ users, page, limit }), {
    headers: {
      'Content-Type': 'application/json',
      'Cache-Control': 'public, max-age=60',
      'X-Total-Count': String(users.length),
    },
  });
}

async function handleCreateUser(request, env) {
  // อ่าน JSON body
  let body;
  try {
    body = await request.json();
  } catch {
    return new Response('Invalid JSON', { status: 400 });
  }
  
  const { name, email } = body;
  
  if (!name || !email) {
    return Response.json(
      { error: 'name และ email จำเป็น' },
      { status: 422 }
    );
  }
  
  // สร้าง user (mock)
  const user = { id: Date.now(), name, email, createdAt: new Date().toISOString() };
  
  return Response.json(user, { status: 201 });
}
```

### Middleware Pattern

```javascript
// Middleware pattern ใน Cloudflare Workers
async function withAuth(request, env) {
  const token = request.headers.get('Authorization')?.replace('Bearer ', '');
  
  if (!token) {
    return new Response('Unauthorized', { status: 401 });
  }
  
  try {
    // Verify JWT using Web Crypto
    const payload = await verifyJWT(token, env.JWT_SECRET);
    return payload;
  } catch {
    return new Response('Invalid token', { status: 401 });
  }
}

async function withCORS(response, origin) {
  const allowedOrigins = ['https://myapp.com', 'https://www.myapp.com'];
  
  if (allowedOrigins.includes(origin)) {
    response.headers.set('Access-Control-Allow-Origin', origin);
    response.headers.set('Access-Control-Allow-Methods', 'GET, POST, PUT, DELETE');
    response.headers.set('Access-Control-Allow-Headers', 'Content-Type, Authorization');
  }
  
  return response;
}

export default {
  async fetch(request, env, ctx) {
    const origin = request.headers.get('Origin') || '';
    
    // Handle preflight
    if (request.method === 'OPTIONS') {
      return new Response(null, {
        headers: {
          'Access-Control-Allow-Origin': origin,
          'Access-Control-Allow-Methods': 'GET, POST',
          'Access-Control-Allow-Headers': 'Content-Type, Authorization',
        },
      });
    }
    
    // Auth middleware
    const url = new URL(request.url);
    if (url.pathname.startsWith('/api/protected/')) {
      const authResult = await withAuth(request, env);
      if (authResult instanceof Response) return authResult; // auth error
      request.user = authResult; // attach user to request
    }
    
    const response = await handleRequest(request, env);
    return withCORS(response, origin);
  },
};
```

---

## ขั้นตอนที่ 1715: KV Storage

```javascript
// wrangler.toml
/*
[[kv_namespaces]]
binding = "MY_KV"
id = "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
*/

export default {
  async fetch(request, env, ctx) {
    const url = new URL(request.url);
    const key = url.searchParams.get('key');
    
    if (request.method === 'GET') {
      // อ่านค่าจาก KV
      const value = await env.MY_KV.get(key);
      
      if (!value) {
        return new Response('Not found', { status: 404 });
      }
      
      return new Response(value);
    }
    
    if (request.method === 'PUT') {
      const body = await request.text();
      
      // เขียนค่าลง KV
      await env.MY_KV.put(key, body, {
        expirationTtl: 3600, // หมดอายุใน 1 ชั่วโมง
      });
      
      return new Response('OK');
    }
    
    if (request.method === 'DELETE') {
      await env.MY_KV.delete(key);
      return new Response('Deleted');
    }
    
    return new Response('Method not allowed', { status: 405 });
  },
};
```

### Session Management ด้วย KV

```javascript
// Session management with KV
const SESSION_TTL = 86400; // 24 ชั่วโมง

async function createSession(env, userId, userData) {
  const sessionId = crypto.randomUUID();
  
  await env.SESSIONS.put(
    `session:${sessionId}`,
    JSON.stringify({
      userId,
      userData,
      createdAt: Date.now(),
    }),
    {
      expirationTtl: SESSION_TTL,
    }
  );
  
  return sessionId;
}

async function getSession(env, sessionId) {
  const data = await env.SESSIONS.get(`session:${sessionId}`, 'json');
  return data;
}

async function deleteSession(env, sessionId) {
  await env.SESSIONS.delete(`session:${sessionId}`);
}

// ใช้งาน
export default {
  async fetch(request, env) {
    const url = new URL(request.url);
    
    if (url.pathname === '/login' && request.method === 'POST') {
      const { username, password } = await request.json();
      
      // ตรวจสอบ credentials (simplified)
      const user = await authenticateUser(env, username, password);
      
      if (!user) {
        return Response.json({ error: 'Invalid credentials' }, { status: 401 });
      }
      
      const sessionId = await createSession(env, user.id, {
        name: user.name,
        email: user.email,
        role: user.role,
      });
      
      return new Response(JSON.stringify({ success: true }), {
        headers: {
          'Set-Cookie': `session=${sessionId}; HttpOnly; Secure; SameSite=Strict; Max-Age=${SESSION_TTL}`,
          'Content-Type': 'application/json',
        },
      });
    }
    
    if (url.pathname === '/me') {
      const cookies = parseCookies(request.headers.get('Cookie') || '');
      const sessionId = cookies.session;
      
      if (!sessionId) {
        return Response.json({ error: 'Not logged in' }, { status: 401 });
      }
      
      const session = await getSession(env, sessionId);
      
      if (!session) {
        return Response.json({ error: 'Session expired' }, { status: 401 });
      }
      
      return Response.json({ user: session.userData });
    }
  },
};

function parseCookies(cookieHeader) {
  return Object.fromEntries(
    cookieHeader.split(';').map(c => {
      const [k, v] = c.trim().split('=');
      return [k, decodeURIComponent(v)];
    })
  );
}
```

---

## ขั้นตอนที่ 1716: Durable Objects

Durable Objects คือ stateful serverless objects ที่รันบน Cloudflare Workers สามารถเก็บ state และรับ WebSocket connections ได้

```javascript
// Durable Object Class
export class Counter {
  constructor(state, env) {
    this.state = state;
    this.env = env;
  }
  
  async fetch(request) {
    const url = new URL(request.url);
    
    if (url.pathname === '/increment') {
      // อ่าน state ปัจจุบัน
      let count = (await this.state.storage.get('count')) || 0;
      count++;
      
      // บันทึก state
      await this.state.storage.put('count', count);
      
      return Response.json({ count });
    }
    
    if (url.pathname === '/value') {
      const count = (await this.state.storage.get('count')) || 0;
      return Response.json({ count });
    }
    
    if (url.pathname === '/reset') {
      await this.state.storage.delete('count');
      return Response.json({ count: 0 });
    }
    
    return new Response('Not found', { status: 404 });
  }
}

// Worker ที่ใช้ Durable Object
export default {
  async fetch(request, env) {
    const url = new URL(request.url);
    const counterId = url.searchParams.get('id') || 'default';
    
    // สร้าง/ดึง Durable Object instance
    const id = env.COUNTER.idFromName(counterId);
    const counter = env.COUNTER.get(id);
    
    // Forward request ไปยัง Durable Object
    return counter.fetch(request);
  },
};
```

```toml
# wrangler.toml
[[durable_objects.bindings]]
name = "COUNTER"
class_name = "Counter"

[[migrations]]
tag = "v1"
new_classes = ["Counter"]
```

### Real-time Counter ด้วย Durable Objects + WebSocket

```javascript
// Live counter ที่ sync ระหว่าง users
export class LiveCounter {
  constructor(state, env) {
    this.state = state;
    this.env = env;
    this.connections = new Set();
  }
  
  async fetch(request) {
    const upgradeHeader = request.headers.get('Upgrade');
    
    if (upgradeHeader === 'websocket') {
      return this.handleWebSocket(request);
    }
    
    return new Response('Expected WebSocket', { status: 400 });
  }
  
  async handleWebSocket(request) {
    const { 0: client, 1: server } = new WebSocketPair();
    
    server.accept();
    this.connections.add(server);
    
    // ส่ง count ปัจจุบันไปยัง client ใหม่
    const count = (await this.state.storage.get('count')) || 0;
    server.send(JSON.stringify({ type: 'count', value: count }));
    
    server.addEventListener('message', async (event) => {
      const data = JSON.parse(event.data);
      
      if (data.type === 'increment') {
        let count = (await this.state.storage.get('count')) || 0;
        count++;
        await this.state.storage.put('count', count);
        
        // Broadcast ไปทุก connections
        const message = JSON.stringify({ type: 'count', value: count });
        for (const conn of this.connections) {
          try {
            conn.send(message);
          } catch {
            this.connections.delete(conn);
          }
        }
      }
    });
    
    server.addEventListener('close', () => {
      this.connections.delete(server);
    });
    
    return new Response(null, {
      status: 101,
      webSocket: client,
    });
  }
}
```

---

## ขั้นตอนที่ 1717: D1 - SQLite at Edge

```javascript
// D1: SQLite database ที่รันบน Cloudflare network

// สร้าง D1 database
// wrangler d1 create my-database

// wrangler.toml
/*
[[d1_databases]]
binding = "DB"
database_name = "my-database"
database_id = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
*/

// Schema migration
// wrangler d1 execute my-database --file=./schema.sql
```

```sql
-- schema.sql
CREATE TABLE IF NOT EXISTS users (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  name TEXT NOT NULL,
  email TEXT UNIQUE NOT NULL,
  created_at TEXT DEFAULT (datetime('now'))
);

CREATE TABLE IF NOT EXISTS products (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  name TEXT NOT NULL,
  price REAL NOT NULL,
  stock INTEGER DEFAULT 0,
  category TEXT,
  created_at TEXT DEFAULT (datetime('now'))
);

CREATE INDEX IF NOT EXISTS idx_products_category ON products(category);
```

```javascript
// Worker กับ D1
export default {
  async fetch(request, env) {
    const url = new URL(request.url);
    
    if (url.pathname === '/products' && request.method === 'GET') {
      const category = url.searchParams.get('category');
      const page = parseInt(url.searchParams.get('page')) || 1;
      const limit = 20;
      const offset = (page - 1) * limit;
      
      let query = 'SELECT * FROM products';
      const params = [];
      
      if (category) {
        query += ' WHERE category = ?';
        params.push(category);
      }
      
      query += ' ORDER BY created_at DESC LIMIT ? OFFSET ?';
      params.push(limit, offset);
      
      // D1 query
      const { results } = await env.DB.prepare(query)
        .bind(...params)
        .all();
      
      // Count total
      const countQuery = category
        ? 'SELECT COUNT(*) as count FROM products WHERE category = ?'
        : 'SELECT COUNT(*) as count FROM products';
      
      const { results: countResult } = await env.DB.prepare(countQuery)
        .bind(...(category ? [category] : []))
        .all();
      
      return Response.json({
        products: results,
        total: countResult[0].count,
        page,
        limit,
      });
    }
    
    if (url.pathname === '/products' && request.method === 'POST') {
      const { name, price, stock, category } = await request.json();
      
      if (!name || !price) {
        return Response.json(
          { error: 'name และ price จำเป็น' },
          { status: 422 }
        );
      }
      
      const { success, meta } = await env.DB.prepare(
        'INSERT INTO products (name, price, stock, category) VALUES (?, ?, ?, ?)'
      )
        .bind(name, price, stock || 0, category || null)
        .run();
      
      if (!success) {
        return Response.json({ error: 'ไม่สามารถสร้าง product' }, { status: 500 });
      }
      
      return Response.json(
        { id: meta.last_row_id, name, price, stock, category },
        { status: 201 }
      );
    }
    
    if (url.pathname.startsWith('/products/') && request.method === 'GET') {
      const id = url.pathname.split('/')[2];
      
      const product = await env.DB.prepare(
        'SELECT * FROM products WHERE id = ?'
      )
        .bind(id)
        .first();
      
      if (!product) {
        return Response.json({ error: 'ไม่พบสินค้า' }, { status: 404 });
      }
      
      return Response.json(product);
    }
    
    return new Response('Not Found', { status: 404 });
  },
};
```

### D1 Transactions

```javascript
// D1 Transaction
async function transferMoney(env, fromId, toId, amount) {
  // D1 batch statements (atomic)
  const results = await env.DB.batch([
    env.DB.prepare(
      'UPDATE accounts SET balance = balance - ? WHERE id = ? AND balance >= ?'
    ).bind(amount, fromId, amount),
    
    env.DB.prepare(
      'UPDATE accounts SET balance = balance + ? WHERE id = ?'
    ).bind(amount, toId),
    
    env.DB.prepare(
      'INSERT INTO transactions (from_id, to_id, amount) VALUES (?, ?, ?)'
    ).bind(fromId, toId, amount),
  ]);
  
  // ตรวจสอบว่า debit สำเร็จ
  if (results[0].meta.changes === 0) {
    throw new Error('ยอดเงินไม่เพียงพอ');
  }
  
  return results[2].meta.last_row_id;
}
```

---

## ขั้นตอนที่ 1718: R2 - Object Storage

```javascript
// R2: Object storage เหมือน S3 แต่ไม่มี egress fees

// wrangler.toml
/*
[[r2_buckets]]
binding = "MY_BUCKET"
bucket_name = "my-bucket"
*/

export default {
  async fetch(request, env) {
    const url = new URL(request.url);
    const key = url.pathname.substring(1); // ตัด leading /
    
    if (request.method === 'GET') {
      const object = await env.MY_BUCKET.get(key);
      
      if (!object) {
        return new Response('Not found', { status: 404 });
      }
      
      const headers = new Headers();
      object.writeHttpMetadata(headers);
      headers.set('etag', object.httpEtag);
      
      return new Response(object.body, { headers });
    }
    
    if (request.method === 'PUT') {
      await env.MY_BUCKET.put(key, request.body, {
        httpMetadata: {
          contentType: request.headers.get('content-type') || 'application/octet-stream',
        },
        customMetadata: {
          uploadedAt: new Date().toISOString(),
          uploadedBy: request.headers.get('x-user-id') || 'anonymous',
        },
      });
      
      return Response.json({ success: true, key });
    }
    
    if (request.method === 'DELETE') {
      await env.MY_BUCKET.delete(key);
      return new Response('Deleted');
    }
    
    if (request.method === 'GET' && key === '') {
      // List objects
      const listed = await env.MY_BUCKET.list({
        limit: 100,
        prefix: url.searchParams.get('prefix') || '',
      });
      
      return Response.json({
        objects: listed.objects.map(obj => ({
          key: obj.key,
          size: obj.size,
          uploaded: obj.uploaded,
        })),
        truncated: listed.truncated,
      });
    }
  },
};
```

### File Upload กับ Presigned URLs

```javascript
// สร้าง presigned URL สำหรับ upload โดยตรงจาก client
export default {
  async fetch(request, env) {
    const url = new URL(request.url);
    
    if (url.pathname === '/upload-url' && request.method === 'POST') {
      const { filename, contentType } = await request.json();
      
      // สร้าง unique key
      const key = `uploads/${crypto.randomUUID()}-${filename}`;
      
      // สร้าง presigned URL (Cloudflare R2)
      // Note: R2 presigned URLs ต้องใช้ AWS S3 compatible API
      const presignedUrl = await generatePresignedUrl(env, key, contentType);
      
      return Response.json({
        uploadUrl: presignedUrl,
        key,
        expiresIn: 3600,
      });
    }
    
    if (url.pathname === '/files' && request.method === 'GET') {
      const listed = await env.MY_BUCKET.list({ prefix: 'uploads/' });
      
      return Response.json({
        files: listed.objects.map(obj => ({
          key: obj.key,
          url: `${new URL(request.url).origin}/files/${obj.key}`,
          size: obj.size,
          uploadedAt: obj.uploaded,
        })),
      });
    }
  },
};
```

---

## ขั้นตอนที่ 1719: Vercel Edge Functions

```javascript
// app/api/personalize/route.js (Next.js)
export const runtime = 'edge';

export async function GET(request) {
  const { searchParams } = new URL(request.url);
  
  // Vercel-specific headers
  const country = request.headers.get('x-vercel-ip-country') || 'TH';
  const city = request.headers.get('x-vercel-ip-city') || 'Bangkok';
  const latitude = request.headers.get('x-vercel-ip-latitude');
  const longitude = request.headers.get('x-vercel-ip-longitude');
  
  // Personalized content based on location
  const content = await getPersonalizedContent(country, city);
  
  return Response.json({
    content,
    location: { country, city, latitude, longitude },
  });
}
```

```javascript
// middleware.ts (Next.js Edge Middleware)
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(request: NextRequest) {
  const { pathname } = request.nextUrl;
  const country = request.geo?.country || 'TH';
  const city = request.geo?.city || 'Bangkok';
  
  // Geo-based redirect
  if (pathname === '/') {
    const locale = getLocaleFromCountry(country);
    
    if (locale !== 'th') {
      return NextResponse.redirect(
        new URL(`/${locale}${pathname}`, request.url)
      );
    }
  }
  
  // Feature flags (ดึงจาก Edge Config)
  const response = NextResponse.next();
  
  // เพิ่ม headers สำหรับ analytics
  response.headers.set('x-country', country);
  response.headers.set('x-city', city);
  
  return response;
}

function getLocaleFromCountry(country: string): string {
  const map: Record<string, string> = {
    'TH': 'th',
    'US': 'en',
    'JP': 'ja',
    'CN': 'zh',
  };
  return map[country] || 'en';
}

export const config = {
  matcher: ['/((?!_next/static|_next/image|favicon.ico).*)'],
};
```

### Edge Config (Vercel)

```javascript
// ดึง feature flags จาก Edge Config
import { createClient } from '@vercel/edge-config';

export const runtime = 'edge';

export async function GET(request: Request) {
  const edgeConfig = createClient(process.env.EDGE_CONFIG);
  
  // ดึง feature flag
  const isNewCheckoutEnabled = await edgeConfig.get('new-checkout-enabled');
  const maintenanceMode = await edgeConfig.get('maintenance-mode');
  
  if (maintenanceMode) {
    return Response.json(
      { error: 'ระบบกำลังปรับปรุง' },
      { status: 503 }
    );
  }
  
  return Response.json({
    features: {
      newCheckout: isNewCheckoutEnabled,
    },
  });
}
```

---

## ขั้นตอนที่ 1720: Deno Deploy

```javascript
// main.ts (Deno Deploy)
import { serve } from "https://deno.land/std@0.208.0/http/server.ts";

serve(async (req: Request) => {
  const url = new URL(req.url);
  
  if (url.pathname === "/") {
    return new Response("สวัสดีจาก Deno Deploy!", {
      headers: { "content-type": "text/plain; charset=utf-8" },
    });
  }
  
  if (url.pathname === "/api/time") {
    return Response.json({
      time: new Date().toISOString(),
      timezone: Intl.DateTimeFormat().resolvedOptions().timeZone,
    });
  }
  
  return new Response("Not Found", { status: 404 });
});
```

```typescript
// Deno Deploy กับ KV
// main.ts
import { serve } from "https://deno.land/std@0.208.0/http/server.ts";

const kv = await Deno.openKv();

serve(async (req: Request) => {
  const url = new URL(req.url);
  
  if (url.pathname === "/counter" && req.method === "POST") {
    // Atomic increment
    const key = ["counter", "visits"];
    
    const result = await kv.atomic()
      .mutate({
        type: "sum",
        key,
        value: new Deno.KvU64(1n),
      })
      .commit();
    
    if (!result.ok) {
      return Response.json({ error: "Failed" }, { status: 500 });
    }
    
    const entry = await kv.get<Deno.KvU64>(key);
    return Response.json({ count: Number(entry.value?.value ?? 0n) });
  }
  
  if (url.pathname === "/counter" && req.method === "GET") {
    const entry = await kv.get<Deno.KvU64>(["counter", "visits"]);
    return Response.json({ count: Number(entry.value?.value ?? 0n) });
  }
  
  return new Response("Not Found", { status: 404 });
});
```

---

## ขั้นตอนที่ 1721: Web APIs ที่ Available บน Edge

```javascript
// ตัวอย่าง Web APIs บน Edge Runtime

// 1. Web Crypto API - สำหรับ cryptography
async function signRequest(secret, body) {
  const encoder = new TextEncoder();
  const key = await crypto.subtle.importKey(
    'raw',
    encoder.encode(secret),
    { name: 'HMAC', hash: 'SHA-256' },
    false,
    ['sign']
  );
  
  const signature = await crypto.subtle.sign(
    'HMAC',
    key,
    encoder.encode(body)
  );
  
  return btoa(String.fromCharCode(...new Uint8Array(signature)));
}

async function verifySignature(secret, body, signature) {
  const expected = await signRequest(secret, body);
  return expected === signature;
}

// ใช้งาน
export default {
  async fetch(request, env) {
    if (request.method === 'POST') {
      const body = await request.text();
      const signature = request.headers.get('x-signature');
      
      const isValid = await verifySignature(
        env.WEBHOOK_SECRET,
        body,
        signature
      );
      
      if (!isValid) {
        return new Response('Invalid signature', { status: 401 });
      }
      
      // Process webhook
      const data = JSON.parse(body);
      await processWebhook(data);
      
      return Response.json({ received: true });
    }
  },
};
```

```javascript
// 2. Streams API - สำหรับ streaming responses
export default {
  async fetch(request) {
    const encoder = new TextEncoder();
    
    const stream = new ReadableStream({
      async start(controller) {
        // Stream ข้อมูลทีละ chunk
        for (let i = 0; i < 10; i++) {
          await new Promise(r => setTimeout(r, 100));
          
          const chunk = encoder.encode(
            `data: ${JSON.stringify({ count: i, time: Date.now() })}\n\n`
          );
          
          controller.enqueue(chunk);
        }
        
        controller.close();
      },
    });
    
    return new Response(stream, {
      headers: {
        'Content-Type': 'text/event-stream',
        'Cache-Control': 'no-cache',
        'Connection': 'keep-alive',
      },
    });
  },
};
```

```javascript
// 3. URL Pattern API
const pattern = new URLPattern({ pathname: '/users/:id/posts/:postId' });

export default {
  async fetch(request) {
    const match = pattern.exec(request.url);
    
    if (match) {
      const { id, postId } = match.pathname.groups;
      return Response.json({ userId: id, postId });
    }
    
    return new Response('Not Found', { status: 404 });
  },
};
```

---

## ขั้นตอนที่ 1722: Limitations ของ Edge Runtime

```javascript
// Edge Runtime Limitations

// 1. CPU Time Limit (ปกติ 50ms บน Cloudflare Workers)
// ห้ามทำงาน CPU-intensive นาน
export default {
  async fetch(request) {
    // ❌ BAD: CPU-intensive work
    const result = computeHeavyWork(); // อาจ timeout
    
    // ✅ GOOD: ส่งไปทำที่อื่น
    const job = await queueHeavyWork(request.body);
    return Response.json({ jobId: job.id });
  },
};

// 2. Memory Limit (128MB บน Cloudflare Workers)
// ❌ ห้ามโหลด ML models ขนาดใหญ่
// ✅ ใช้ AI Gateway หรือ Workers AI

// 3. ไม่มี filesystem
// ❌ ใช้ fs module ไม่ได้
// ✅ ใช้ R2 หรือ KV แทน

// 4. Cold start (แต่ Edge เร็วกว่า Lambda มาก)
// Cloudflare Workers: ~5ms cold start (V8 isolates ไม่ใช่ containers)
// Lambda: 100-3000ms cold start

// 5. Package compatibility
// หลาย npm packages ที่ใช้ Node.js APIs จะไม่ทำงาน
// ต้องเลือก packages ที่รองรับ Edge Runtime

// การตรวจสอบว่า package ทำงานบน Edge ได้ไหม
// npm search "edge runtime" หรือ ดูใน package.json ว่ามี "edge" exports
```

---

## ขั้นตอนที่ 1723: Use Cases - A/B Testing

```javascript
// A/B Testing ที่ Edge
export default {
  async fetch(request, env) {
    const url = new URL(request.url);
    
    // ตรวจสอบว่ามี A/B test cookie อยู่แล้วไหม
    const cookies = parseCookies(request.headers.get('Cookie') || '');
    let bucket = cookies['ab-test'];
    
    if (!bucket) {
      // สุ่ม bucket ใหม่
      bucket = Math.random() < 0.5 ? 'A' : 'B';
    }
    
    // ดึงหน้าที่เหมาะสม
    let targetUrl = request.url;
    if (bucket === 'B' && url.pathname === '/checkout') {
      targetUrl = url.origin + '/checkout-v2' + url.search;
    }
    
    // Forward request
    const response = await fetch(targetUrl, {
      method: request.method,
      headers: request.headers,
      body: request.body,
    });
    
    // Clone response เพื่อแก้ไข headers
    const newResponse = new Response(response.body, response);
    newResponse.headers.append(
      'Set-Cookie',
      `ab-test=${bucket}; Path=/; Max-Age=86400; SameSite=Lax`
    );
    newResponse.headers.set('X-AB-Bucket', bucket);
    
    return newResponse;
  },
};
```

---

## ขั้นตอนที่ 1724: Personalization ที่ Edge

```javascript
// Personalization ตาม geolocation และ user data
export default {
  async fetch(request, env) {
    const country = request.headers.get('cf-ipcountry') || 'TH';
    const city = request.cf?.city || 'Bangkok';
    const cookies = parseCookies(request.headers.get('Cookie') || '');
    const userId = cookies['user-id'];
    
    // ดึง user preferences จาก KV
    let userPrefs = null;
    if (userId) {
      userPrefs = await env.USER_PREFS.get(`user:${userId}`, 'json');
    }
    
    // Personalization rules
    const locale = getLocale(country, userPrefs?.language);
    const currency = getCurrency(country, userPrefs?.currency);
    const recommendations = await getRecommendations(userId, country, env);
    
    // Fetch original page
    const response = await fetch(request);
    const html = await response.text();
    
    // Inject personalized data
    const personalizedHtml = html
      .replace('__LOCALE__', locale)
      .replace('__CURRENCY__', currency)
      .replace('__RECOMMENDATIONS__', JSON.stringify(recommendations));
    
    return new Response(personalizedHtml, {
      headers: response.headers,
    });
  },
};
```

---

## ขั้นตอนที่ 1725: Authentication ที่ Edge

```javascript
// JWT verification ที่ Edge (ไม่ต้องผ่าน origin server)
export default {
  async fetch(request, env) {
    const url = new URL(request.url);
    
    // Protected routes
    if (url.pathname.startsWith('/api/protected/')) {
      const authHeader = request.headers.get('Authorization');
      
      if (!authHeader?.startsWith('Bearer ')) {
        return Response.json(
          { error: 'กรุณา login ก่อน' },
          {
            status: 401,
            headers: { 'WWW-Authenticate': 'Bearer' },
          }
        );
      }
      
      const token = authHeader.substring(7);
      
      try {
        const payload = await verifyJWT(token, env.JWT_PUBLIC_KEY);
        
        // ตรวจสอบ expiry
        if (payload.exp < Date.now() / 1000) {
          return Response.json({ error: 'Token หมดอายุ' }, { status: 401 });
        }
        
        // เพิ่ม user info ใน headers
        const modifiedRequest = new Request(request, {
          headers: {
            ...Object.fromEntries(request.headers),
            'x-user-id': payload.sub,
            'x-user-role': payload.role,
          },
        });
        
        return fetch(modifiedRequest);
      } catch (e) {
        return Response.json({ error: 'Token ไม่ถูกต้อง' }, { status: 401 });
      }
    }
    
    return fetch(request);
  },
};

// JWT verification ด้วย Web Crypto
async function verifyJWT(token, publicKeyPem) {
  const [headerB64, payloadB64, signatureB64] = token.split('.');
  
  const header = JSON.parse(atob(headerB64));
  const payload = JSON.parse(atob(payloadB64));
  
  if (header.alg !== 'RS256') {
    throw new Error('Unsupported algorithm');
  }
  
  const publicKey = await importPublicKey(publicKeyPem);
  
  const data = new TextEncoder().encode(`${headerB64}.${payloadB64}`);
  const signature = base64UrlDecode(signatureB64);
  
  const valid = await crypto.subtle.verify(
    { name: 'RSASSA-PKCS1-v1_5', hash: 'SHA-256' },
    publicKey,
    signature,
    data
  );
  
  if (!valid) throw new Error('Invalid signature');
  
  return payload;
}
```

---

## ขั้นตอนที่ 1726: Geolocation ที่ Edge

```javascript
// Cloudflare Workers Geolocation
export default {
  async fetch(request) {
    const cf = request.cf;
    
    // Cloudflare geo data
    const geoInfo = {
      country: cf.country,           // "TH"
      city: cf.city,                 // "Bangkok"
      continent: cf.continent,       // "AS"
      latitude: cf.latitude,         // "13.75398"
      longitude: cf.longitude,       // "100.50144"
      postalCode: cf.postalCode,     // "10100"
      timezone: cf.timezone,         // "Asia/Bangkok"
      region: cf.region,             // "Bangkok"
      regionCode: cf.regionCode,     // "10"
      asn: cf.asn,                   // AS number
      isp: cf.asOrganization,        // ISP name
    };
    
    // Content localization ตาม country
    const content = getLocalizedContent(cf.country);
    
    // Price localization
    const pricing = getPricingForCountry(cf.country);
    
    // Shipping availability
    const shippingAvailable = isShippingAvailable(cf.country);
    
    return Response.json({
      geo: geoInfo,
      content,
      pricing,
      shippingAvailable,
    });
  },
};

function getLocalizedContent(country) {
  const contentMap = {
    'TH': { currency: 'THB', language: 'th', shippingDays: '1-3' },
    'US': { currency: 'USD', language: 'en', shippingDays: '3-7' },
    'JP': { currency: 'JPY', language: 'ja', shippingDays: '2-5' },
  };
  
  return contentMap[country] || contentMap['US'];
}
```

---

## ขั้นตอนที่ 1727: WebSockets ที่ Edge

```javascript
// WebSocket ใน Cloudflare Workers (ผ่าน Durable Objects)
export class ChatRoom {
  constructor(state, env) {
    this.state = state;
    this.env = env;
    this.sessions = new Map();
  }
  
  async fetch(request) {
    if (request.headers.get('Upgrade') !== 'websocket') {
      return new Response('Expected WebSocket', { status: 400 });
    }
    
    const { 0: client, 1: server } = new WebSocketPair();
    server.accept();
    
    // สร้าง session
    const sessionId = crypto.randomUUID();
    const url = new URL(request.url);
    const username = url.searchParams.get('username') || 'Anonymous';
    
    this.sessions.set(sessionId, { server, username });
    
    // แจ้งว่ามีคนเข้าร่วม
    this.broadcast({
      type: 'join',
      username,
      message: `${username} เข้าร่วมห้องสนทนา`,
      timestamp: Date.now(),
    }, sessionId);
    
    server.addEventListener('message', async (event) => {
      const data = JSON.parse(event.data);
      
      if (data.type === 'message') {
        // บันทึก message ลง history
        const history = await this.state.storage.get('history') || [];
        history.push({
          sessionId,
          username,
          message: data.message,
          timestamp: Date.now(),
        });
        
        // เก็บแค่ 100 messages ล่าสุด
        if (history.length > 100) history.shift();
        await this.state.storage.put('history', history);
        
        // Broadcast ไปทุกคน
        this.broadcast({
          type: 'message',
          username,
          message: data.message,
          timestamp: Date.now(),
        });
      }
    });
    
    server.addEventListener('close', () => {
      this.sessions.delete(sessionId);
      this.broadcast({
        type: 'leave',
        username,
        message: `${username} ออกจากห้องสนทนา`,
        timestamp: Date.now(),
      });
    });
    
    // ส่ง history ให้คนเข้าใหม่
    const history = await this.state.storage.get('history') || [];
    server.send(JSON.stringify({ type: 'history', history }));
    
    return new Response(null, {
      status: 101,
      webSocket: client,
    });
  }
  
  broadcast(message, excludeSessionId = null) {
    const data = JSON.stringify(message);
    for (const [sessionId, session] of this.sessions) {
      if (sessionId !== excludeSessionId) {
        try {
          session.server.send(data);
        } catch {
          this.sessions.delete(sessionId);
        }
      }
    }
  }
}

export default {
  async fetch(request, env) {
    const url = new URL(request.url);
    const roomId = url.searchParams.get('room') || 'general';
    
    const id = env.CHAT_ROOM.idFromName(roomId);
    const room = env.CHAT_ROOM.get(id);
    
    return room.fetch(request);
  },
};
```

---

## ขั้นตอนที่ 1728: Edge Middleware Patterns

```javascript
// Rate Limiting ที่ Edge
export default {
  async fetch(request, env) {
    const ip = request.headers.get('CF-Connecting-IP');
    const key = `rate-limit:${ip}`;
    
    // อ่าน current count
    const current = await env.RATE_LIMIT.get(key);
    const count = current ? parseInt(current) : 0;
    
    const LIMIT = 100; // requests ต่อนาที
    
    if (count >= LIMIT) {
      return new Response('Too Many Requests', {
        status: 429,
        headers: {
          'Retry-After': '60',
          'X-RateLimit-Limit': String(LIMIT),
          'X-RateLimit-Remaining': '0',
        },
      });
    }
    
    // Increment counter
    await env.RATE_LIMIT.put(key, String(count + 1), {
      expirationTtl: 60,
    });
    
    const response = await fetch(request);
    
    // เพิ่ม rate limit headers
    const newResponse = new Response(response.body, response);
    newResponse.headers.set('X-RateLimit-Limit', String(LIMIT));
    newResponse.headers.set('X-RateLimit-Remaining', String(LIMIT - count - 1));
    
    return newResponse;
  },
};
```

```javascript
// Image Optimization ที่ Edge
export default {
  async fetch(request) {
    const url = new URL(request.url);
    
    if (url.pathname.startsWith('/images/')) {
      const width = parseInt(url.searchParams.get('w')) || null;
      const height = parseInt(url.searchParams.get('h')) || null;
      const quality = parseInt(url.searchParams.get('q')) || 80;
      const format = url.searchParams.get('f') || 'webp';
      
      // Cloudflare Image Resizing
      return fetch(request, {
        cf: {
          image: {
            width,
            height,
            quality,
            format,
            fit: 'cover',
          },
        },
      });
    }
    
    return fetch(request);
  },
};
```

---

## ขั้นตอนที่ 1729: Workers AI

```javascript
// Cloudflare Workers AI
export default {
  async fetch(request, env) {
    const url = new URL(request.url);
    
    if (url.pathname === '/api/generate-text') {
      const { prompt } = await request.json();
      
      const response = await env.AI.run('@cf/meta/llama-2-7b-chat-int8', {
        prompt,
        max_tokens: 500,
      });
      
      return Response.json({ text: response.response });
    }
    
    if (url.pathname === '/api/classify-image') {
      const formData = await request.formData();
      const imageFile = formData.get('image');
      const imageBuffer = await imageFile.arrayBuffer();
      
      const results = await env.AI.run('@cf/microsoft/resnet-50', {
        image: [...new Uint8Array(imageBuffer)],
      });
      
      return Response.json({ classifications: results });
    }
    
    if (url.pathname === '/api/embed') {
      const { text } = await request.json();
      
      const result = await env.AI.run('@cf/baai/bge-small-en-v1.5', {
        text: Array.isArray(text) ? text : [text],
      });
      
      return Response.json({ embeddings: result.data });
    }
    
    return new Response('Not Found', { status: 404 });
  },
};
```

---

## ขั้นตอนที่ 1730: Edge Caching Strategies

```javascript
// Cache API ที่ Edge
export default {
  async fetch(request, env, ctx) {
    const cache = caches.default;
    const cacheKey = new Request(request.url, request);
    
    // ตรวจสอบ cache ก่อน
    let response = await cache.match(cacheKey);
    
    if (response) {
      // Cache hit!
      const newResponse = new Response(response.body, response);
      newResponse.headers.set('X-Cache', 'HIT');
      return newResponse;
    }
    
    // Cache miss - fetch จาก origin
    response = await fetch(request);
    
    // Cache เฉพาะ successful responses
    if (response.ok) {
      const responseToCache = new Response(response.clone().body, {
        ...response,
        headers: {
          ...Object.fromEntries(response.headers),
          'Cache-Control': 'public, max-age=3600',
          'X-Cache': 'MISS',
        },
      });
      
      // Store in cache (async - ไม่รอ)
      ctx.waitUntil(cache.put(cacheKey, responseToCache));
    }
    
    const newResponse = new Response(response.body, response);
    newResponse.headers.set('X-Cache', 'MISS');
    return newResponse;
  },
};
```

```javascript
// Stale-While-Revalidate pattern
export default {
  async fetch(request, env, ctx) {
    const cache = caches.default;
    const cacheKey = request.url;
    
    const cached = await cache.match(cacheKey);
    
    if (cached) {
      const age = Date.now() - parseInt(cached.headers.get('X-Cached-At') || '0');
      const maxAge = 60 * 1000; // 1 นาที
      const staleWindow = 5 * 60 * 1000; // 5 นาที stale window
      
      if (age < maxAge) {
        // Fresh - return immediately
        return cached;
      }
      
      if (age < maxAge + staleWindow) {
        // Stale - return stale, revalidate in background
        ctx.waitUntil(revalidateCache(request, env, cache, cacheKey));
        
        const staleResponse = new Response(cached.body, cached);
        staleResponse.headers.set('X-Cache', 'STALE');
        return staleResponse;
      }
    }
    
    // Expired or no cache - fetch fresh
    return revalidateCache(request, env, cache, cacheKey);
  },
};

async function revalidateCache(request, env, cache, cacheKey) {
  const response = await fetch(request);
  
  const toCache = new Response(response.clone().body, {
    ...response,
    headers: {
      ...Object.fromEntries(response.headers),
      'X-Cached-At': String(Date.now()),
      'X-Cache': 'FRESH',
    },
  });
  
  await cache.put(cacheKey, toCache);
  
  return new Response(response.body, {
    ...response,
    headers: {
      ...Object.fromEntries(response.headers),
      'X-Cache': 'FRESH',
    },
  });
}
```

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: สร้าง URL Shortener ด้วย Cloudflare Workers + KV
สร้าง URL shortener ที่:
- รับ URL ยาว แล้วสร้าง short code
- Redirect short URL ไปยัง long URL
- เก็บ click statistics
- มี expiry สำหรับ links

### แบบฝึกหัดที่ 2: Edge Authentication Middleware
สร้าง middleware ที่:
- Verify JWT tokens ที่ Edge (ไม่ผ่าน origin)
- Rate limit ตาม user/IP
- Block requests จาก blocked countries
- Log failed auth attempts ไปยัง Analytics Engine

### แบบฝึกหัดที่ 3: Real-time Dashboard ด้วย Durable Objects
สร้าง real-time dashboard ที่:
- ใช้ Durable Objects เก็บ state
- WebSocket สำหรับ real-time updates
- Multiple rooms/dashboards
- Persistent history

### แบบฝึกหัดที่ 4: Image Proxy กับ Cache
สร้าง image proxy ที่:
- รับ image URL และ resize parameters
- Cache images ที่ Edge
- รองรับ WebP conversion
- มี rate limiting ต่อ IP

### แบบฝึกหัดที่ 5: A/B Testing Platform
สร้าง A/B testing ที่:
- กำหนด experiments ผ่าน Edge Config หรือ KV
- Consistent bucket assignment ต่อ user
- Track conversions
- Dashboard สรุปผลลัพธ์

---

## สรุป

Edge Computing ใน JavaScript เปิดความเป็นไปได้ใหม่ๆ:

| Feature | Cloudflare | Vercel | Deno Deploy |
|---------|------------|--------|-------------|
| Latency | ~5ms | ~10ms | ~10ms |
| Cold start | <5ms | ~10ms | ~10ms |
| Runtime | V8 Isolates | V8 Isolates | Deno |
| Storage | KV, D1, R2, DO | Edge Config | Deno KV |
| AI | Workers AI | AI SDK | - |
| Free tier | 100k req/day | 500k req/mo | 100k req/day |

ในส่วนถัดไปเราจะเรียนเรื่อง AI/ML ใน JavaScript!
