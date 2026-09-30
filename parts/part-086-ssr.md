# Part 86: Server-Side Rendering (SSR)
## ขั้นตอนที่ 1691-1710: การเรนเดอร์ฝั่งเซิร์ฟเวอร์

SSR (Server-Side Rendering) คือเทคนิคที่ HTML ถูกสร้างบนเซิร์ฟเวอร์แทนที่จะสร้างในเบราว์เซอร์ของผู้ใช้ ทำให้เว็บไซต์โหลดเร็วขึ้น และ SEO ดีขึ้น

---

## ขั้นตอนที่ 1691: CSR vs SSR vs SSG vs ISR เปรียบเทียบ

### Client-Side Rendering (CSR)
ผู้ใช้ได้รับ HTML เปล่า แล้ว JavaScript โหลดและสร้าง DOM ขึ้นมา

```html
<!-- CSR: HTML ที่ได้รับจากเซิร์ฟเวอร์ -->
<!DOCTYPE html>
<html>
<head><title>CSR App</title></head>
<body>
  <div id="root"></div>
  <!-- JavaScript bundle จะสร้าง DOM ทั้งหมด -->
  <script src="/bundle.js"></script>
</body>
</html>
```

```javascript
// CSR: React ทำงานในเบราว์เซอร์ทั้งหมด
// bundle.js
import React from 'react';
import { createRoot } from 'react-dom/client';

function App() {
  const [data, setData] = React.useState(null);
  
  React.useEffect(() => {
    // fetch หลังจาก JavaScript โหลดแล้ว
    fetch('/api/data')
      .then(r => r.json())
      .then(setData);
  }, []);
  
  if (!data) return <div>Loading...</div>;
  return <div>{data.title}</div>;
}

const root = createRoot(document.getElementById('root'));
root.render(<App />);
```

**ข้อดีของ CSR:**
- หลังโหลดครั้งแรก navigation เร็วมาก
- Server load น้อย
- ง่ายต่อการ deploy เป็น static files

**ข้อเสียของ CSR:**
- Time to First Contentful Paint (FCP) ช้า
- SEO แย่ (บอท Google อาจไม่รอ JS execute)
- JavaScript bundle ใหญ่

---

### Server-Side Rendering (SSR)

```javascript
// SSR: HTML สร้างบนเซิร์ฟเวอร์ทุก request
// pages/index.js (Next.js Pages Router)
export default function HomePage({ products }) {
  return (
    <div>
      <h1>สินค้าทั้งหมด</h1>
      {products.map(p => (
        <div key={p.id}>{p.name} - {p.price}</div>
      ))}
    </div>
  );
}

// ฟังก์ชันนี้รันบนเซิร์ฟเวอร์ทุกครั้งที่มี request
export async function getServerSideProps(context) {
  const { req, res, params, query } = context;
  
  // ดึงข้อมูลจาก database หรือ API
  const products = await fetchProductsFromDB();
  
  return {
    props: {
      products,
    },
  };
}
```

**ข้อดีของ SSR:**
- FCP เร็ว (HTML พร้อมเนื้อหา)
- SEO ดีมาก
- ข้อมูลใหม่เสมอ (every request)

**ข้อเสียของ SSR:**
- Server load สูง
- ทุก request ต้องรอ server
- TTFB (Time to First Byte) อาจช้า

---

### Static Site Generation (SSG)

```javascript
// SSG: HTML สร้างเวลา build
// pages/blog/[slug].js
export default function BlogPost({ post }) {
  return (
    <article>
      <h1>{post.title}</h1>
      <p>{post.content}</p>
    </article>
  );
}

// รันเวลา build เท่านั้น
export async function getStaticProps({ params }) {
  const post = await fetchPostBySlug(params.slug);
  
  return {
    props: { post },
    // revalidate: 60, // ถ้าใช้ ISR
  };
}

// บอก Next.js ว่ามี paths อะไรบ้าง
export async function getStaticPaths() {
  const posts = await fetchAllPosts();
  
  return {
    paths: posts.map(p => ({
      params: { slug: p.slug }
    })),
    fallback: false, // 404 ถ้าไม่มี
  };
}
```

---

### Incremental Static Regeneration (ISR)

```javascript
// ISR: SSG + auto revalidation
export async function getStaticProps({ params }) {
  const product = await fetchProduct(params.id);
  
  return {
    props: { product },
    // revalidate ทุก 60 วินาที
    revalidate: 60,
  };
}
```

### เปรียบเทียบแบบตาราง

| Feature | CSR | SSR | SSG | ISR |
|---------|-----|-----|-----|-----|
| SEO | ไม่ดี | ดีมาก | ดีมาก | ดีมาก |
| Build time | เร็ว | เร็ว | ช้า | ปานกลาง |
| Runtime | ช้า FCP | ช้า TTFB | เร็วมาก | เร็วมาก |
| Data freshness | Real-time | Real-time | Stale | Configurable |
| Server load | ต่ำ | สูง | ต่ำมาก | ต่ำ |

---

## ขั้นตอนที่ 1692: ทำไม SSR ถึงสำคัญ

### SEO และ Initial Load

```javascript
// ตัวอย่างเปรียบเทียบ HTML ที่ Google Bot เห็น

// CSR - Google Bot เห็นแค่นี้ (ก่อน JS execute):
`
<html>
  <body>
    <div id="root"></div>
  </body>
</html>
`

// SSR - Google Bot เห็นเนื้อหาจริง:
`
<html>
  <body>
    <div id="root">
      <h1>iPhone 15 Pro Max</h1>
      <p>ราคา: 49,900 บาท</p>
      <p>สต็อก: มีของ</p>
    </div>
  </body>
</html>
`
```

### Core Web Vitals

```javascript
// การวัด Web Vitals
import { getCLS, getFID, getFCP, getLCP, getTTFB } from 'web-vitals';

function sendToAnalytics(metric) {
  console.log(metric.name, metric.value);
  // ส่งไป Google Analytics หรือ monitoring service
}

// SSR ช่วยปรับปรุง:
// LCP (Largest Contentful Paint) - เนื้อหาหลักโหลดเร็ว
// FCP (First Contentful Paint) - เห็น content แรก
// TTFB (Time to First Byte) - ควรต่ำกว่า 200ms

getCLS(sendToAnalytics);
getFID(sendToAnalytics);
getFCP(sendToAnalytics);
getLCP(sendToAnalytics);
getTTFB(sendToAnalytics);
```

---

## ขั้นตอนที่ 1693: Next.js SSR กับ Server Components

### App Router (Next.js 13+)

```javascript
// app/page.js - Server Component โดย default
// ไม่ต้องเขียน 'use server' - เป็น server component เลย

async function getProducts() {
  // ดึงข้อมูลโดยตรงจาก server
  // ไม่ต้องผ่าน API route
  const products = await db.query('SELECT * FROM products');
  return products;
}

export default async function HomePage() {
  // await โดยตรงได้ใน Server Component
  const products = await getProducts();
  
  return (
    <main>
      <h1>ร้านค้าออนไลน์</h1>
      <ul>
        {products.map(product => (
          <li key={product.id}>
            <span>{product.name}</span>
            <span>฿{product.price.toLocaleString()}</span>
          </li>
        ))}
      </ul>
    </main>
  );
}
```

```javascript
// app/products/[id]/page.js
import { notFound } from 'next/navigation';

async function getProduct(id) {
  const product = await db.query(
    'SELECT * FROM products WHERE id = ?',
    [id]
  );
  return product[0] || null;
}

// generateMetadata สำหรับ SEO
export async function generateMetadata({ params }) {
  const product = await getProduct(params.id);
  
  if (!product) return { title: 'ไม่พบสินค้า' };
  
  return {
    title: product.name,
    description: product.description,
    openGraph: {
      title: product.name,
      description: product.description,
      images: [product.imageUrl],
    },
  };
}

export default async function ProductPage({ params }) {
  const product = await getProduct(params.id);
  
  if (!product) notFound();
  
  return (
    <div>
      <h1>{product.name}</h1>
      <p>{product.description}</p>
      <p className="price">฿{product.price.toLocaleString()}</p>
    </div>
  );
}
```

### Client Component

```javascript
// app/components/AddToCart.js
'use client'; // บอกว่าเป็น Client Component

import { useState } from 'react';

export default function AddToCart({ productId, price }) {
  const [quantity, setQuantity] = useState(1);
  const [loading, setLoading] = useState(false);
  
  async function handleAddToCart() {
    setLoading(true);
    try {
      await fetch('/api/cart', {
        method: 'POST',
        body: JSON.stringify({ productId, quantity }),
      });
      alert('เพิ่มลงตะกร้าแล้ว!');
    } finally {
      setLoading(false);
    }
  }
  
  return (
    <div>
      <input
        type="number"
        value={quantity}
        onChange={e => setQuantity(Number(e.target.value))}
        min={1}
      />
      <button onClick={handleAddToCart} disabled={loading}>
        {loading ? 'กำลังเพิ่ม...' : `เพิ่มลงตะกร้า (฿${(price * quantity).toLocaleString()})`}
      </button>
    </div>
  );
}
```

```javascript
// app/products/[id]/page.js - รวม Server + Client Components
import AddToCart from '@/components/AddToCart';

export default async function ProductPage({ params }) {
  const product = await getProduct(params.id);
  
  return (
    <div>
      {/* Server Component: render static content */}
      <h1>{product.name}</h1>
      <p>{product.description}</p>
      
      {/* Client Component: interactive */}
      <AddToCart productId={product.id} price={product.price} />
    </div>
  );
}
```

---

## ขั้นตอนที่ 1694: getServerSideProps (Pages Router)

```javascript
// pages/products/index.js
import Head from 'next/head';

export default function ProductsPage({ products, totalCount, page }) {
  return (
    <>
      <Head>
        <title>สินค้าทั้งหมด ({totalCount} รายการ)</title>
      </Head>
      
      <h1>สินค้าทั้งหมด</h1>
      <p>พบ {totalCount} รายการ</p>
      
      <div className="products-grid">
        {products.map(product => (
          <ProductCard key={product.id} product={product} />
        ))}
      </div>
      
      <Pagination currentPage={page} total={totalCount} />
    </>
  );
}

export async function getServerSideProps(context) {
  const {
    req,      // HTTP request object
    res,      // HTTP response object
    params,   // dynamic route params
    query,    // query string (?page=2&sort=price)
    locale,   // i18n locale
    preview,  // preview mode
  } = context;
  
  // ดึง query parameters
  const page = parseInt(query.page) || 1;
  const sort = query.sort || 'name';
  const perPage = 20;
  
  try {
    const [products, totalCount] = await Promise.all([
      db.query(
        `SELECT * FROM products ORDER BY ${sort} LIMIT ? OFFSET ?`,
        [perPage, (page - 1) * perPage]
      ),
      db.query('SELECT COUNT(*) as count FROM products'),
    ]);
    
    return {
      props: {
        products,
        totalCount: totalCount[0].count,
        page,
      },
    };
  } catch (error) {
    // Redirect ถ้า error
    return {
      redirect: {
        destination: '/error',
        permanent: false,
      },
    };
  }
}
```

### Cookie และ Authentication ใน getServerSideProps

```javascript
// pages/dashboard.js
import { parse } from 'cookie';
import jwt from 'jsonwebtoken';

export default function Dashboard({ user, stats }) {
  return (
    <div>
      <h1>ยินดีต้อนรับ, {user.name}</h1>
      <p>ยอดขายวันนี้: ฿{stats.todaySales.toLocaleString()}</p>
    </div>
  );
}

export async function getServerSideProps({ req, res }) {
  // อ่าน cookie จาก request
  const cookies = parse(req.headers.cookie || '');
  const token = cookies.authToken;
  
  if (!token) {
    return {
      redirect: {
        destination: '/login',
        permanent: false,
      },
    };
  }
  
  try {
    const user = jwt.verify(token, process.env.JWT_SECRET);
    const stats = await fetchUserStats(user.id);
    
    // Set cache headers
    res.setHeader('Cache-Control', 'private, max-age=0, must-revalidate');
    
    return {
      props: { user, stats },
    };
  } catch {
    // Token invalid
    res.setHeader('Set-Cookie', 'authToken=; Path=/; Max-Age=0');
    return {
      redirect: {
        destination: '/login',
        permanent: false,
      },
    };
  }
}
```

---

## ขั้นตอนที่ 1695: Streaming SSR กับ Suspense

```javascript
// app/page.js - Streaming SSR
import { Suspense } from 'react';

// Component ที่โหลดช้า
async function SlowDataComponent() {
  // จำลอง slow database query
  await new Promise(r => setTimeout(r, 2000));
  const data = await fetchExpensiveData();
  
  return (
    <div>
      {data.map(item => (
        <div key={item.id}>{item.name}</div>
      ))}
    </div>
  );
}

// Fast component
async function FastDataComponent() {
  const data = await fetchFastData();
  return <div>{data.title}</div>;
}

export default function Page() {
  return (
    <div>
      {/* ส่วนนี้ render ทันที */}
      <h1>หน้าหลัก</h1>
      
      {/* ส่วนนี้โหลดเร็ว */}
      <Suspense fallback={<div>กำลังโหลด...</div>}>
        <FastDataComponent />
      </Suspense>
      
      {/* ส่วนนี้ stream ทีหลัง */}
      <Suspense fallback={<LoadingSpinner />}>
        <SlowDataComponent />
      </Suspense>
    </div>
  );
}
```

### Loading UI

```javascript
// app/products/loading.js
// Next.js จะแสดง loading UI ขณะรอ page โหลด

export default function Loading() {
  return (
    <div className="loading-container">
      <div className="skeleton-grid">
        {Array.from({ length: 12 }).map((_, i) => (
          <div key={i} className="skeleton-card">
            <div className="skeleton-image" />
            <div className="skeleton-text" />
            <div className="skeleton-text short" />
          </div>
        ))}
      </div>
    </div>
  );
}
```

```javascript
// app/products/error.js
'use client';

export default function Error({ error, reset }) {
  return (
    <div className="error-container">
      <h2>เกิดข้อผิดพลาด!</h2>
      <p>{error.message}</p>
      <button onClick={() => reset()}>
        ลองอีกครั้ง
      </button>
    </div>
  );
}
```

---

## ขั้นตอนที่ 1696: React Server Components Model

### Server Component ทำได้:
```javascript
// app/components/ServerSideData.js
// ไม่มี 'use client' = Server Component

import { db } from '@/lib/database';
import { readFile } from 'fs/promises';
import { cookies, headers } from 'next/headers';

export default async function ServerSideData() {
  // 1. เข้าถึง database โดยตรง
  const users = await db.select().from('users').limit(10);
  
  // 2. อ่านไฟล์จาก filesystem
  const config = JSON.parse(
    await readFile('./config.json', 'utf-8')
  );
  
  // 3. อ่าน cookies/headers (server-side เท่านั้น)
  const cookieStore = cookies();
  const userId = cookieStore.get('userId')?.value;
  
  const headersList = headers();
  const userAgent = headersList.get('user-agent');
  
  // 4. ใช้ environment variables ที่เป็น secret ได้
  const apiData = await fetch('https://api.example.com/data', {
    headers: {
      Authorization: `Bearer ${process.env.SECRET_API_KEY}`,
    },
  });
  
  return (
    <div>
      <p>Users: {users.length}</p>
      <p>Config version: {config.version}</p>
    </div>
  );
}
```

### Server Component ทำไม่ได้:
```javascript
// สิ่งที่ Server Component ทำไม่ได้:
// - useState, useEffect, useRef (ต้องใช้ Client Component)
// - Event handlers (onClick, onChange)
// - Browser APIs (window, document, localStorage)
// - Context (createContext/useContext)

// ตัวอย่างที่ผิด:
async function BadServerComponent() {
  const [count, setCount] = useState(0); // ERROR!
  
  useEffect(() => { // ERROR!
    document.title = 'Hello';
  }, []);
  
  return <button onClick={() => setCount(c => c + 1)}>{count}</button>; // ERROR!
}
```

### Composition Pattern

```javascript
// app/page.js - ผสม Server + Client Components

// Server Component (outer)
import InteractiveButton from './InteractiveButton';

async function ServerWrapper() {
  const data = await fetchData(); // Server only
  
  return (
    <div>
      <h1>{data.title}</h1>
      {/* ส่ง server data ไปยัง client component */}
      <InteractiveButton
        initialCount={data.count}
        label={data.label}
      />
    </div>
  );
}

// components/InteractiveButton.js
'use client';
import { useState } from 'react';

export default function InteractiveButton({ initialCount, label }) {
  const [count, setCount] = useState(initialCount);
  
  return (
    <button onClick={() => setCount(c => c + 1)}>
      {label}: {count}
    </button>
  );
}
```

---

## ขั้นตอนที่ 1697: Data Fetching Patterns ใน SSR

### Parallel Data Fetching

```javascript
// app/dashboard/page.js
async function getDashboardData(userId) {
  // ดึงข้อมูลพร้อมกัน - เร็วกว่าดึงทีละอัน
  const [user, orders, stats, notifications] = await Promise.all([
    fetchUser(userId),
    fetchUserOrders(userId),
    fetchUserStats(userId),
    fetchNotifications(userId),
  ]);
  
  return { user, orders, stats, notifications };
}

export default async function DashboardPage() {
  const userId = await getCurrentUserId();
  const { user, orders, stats, notifications } = await getDashboardData(userId);
  
  return (
    <div>
      <UserProfile user={user} />
      <StatsCards stats={stats} />
      <RecentOrders orders={orders} />
      <NotificationBell count={notifications.unread} />
    </div>
  );
}
```

### Sequential Data Fetching (เมื่อจำเป็น)

```javascript
// บางครั้งต้องรอลำดับ
async function getNestedData() {
  const user = await fetchUser();
  // ต้องรู้ user.organizationId ก่อน
  const org = await fetchOrganization(user.organizationId);
  // ต้องรู้ org.teamIds ก่อน
  const teams = await fetchTeams(org.teamIds);
  
  return { user, org, teams };
}
```

### Data Fetching กับ Cache

```javascript
// app/lib/data.js
import { unstable_cache } from 'next/cache';

// cache function สำหรับ SSR
const getProducts = unstable_cache(
  async (category) => {
    console.log('Fetching from database...'); // รัน 1 ครั้ง แล้ว cache
    return await db.query(
      'SELECT * FROM products WHERE category = ?',
      [category]
    );
  },
  ['products'], // cache key prefix
  {
    revalidate: 300, // cache 5 นาที
    tags: ['products'], // revalidate tag
  }
);

// ใน page.js
export default async function Page({ params }) {
  const products = await getProducts(params.category);
  return <ProductList products={products} />;
}

// Revalidate เมื่อข้อมูลเปลี่ยน
// app/api/revalidate/route.js
import { revalidateTag } from 'next/cache';

export async function POST(request) {
  const { tag } = await request.json();
  revalidateTag(tag);
  return Response.json({ revalidated: true });
}
```

---

## ขั้นตอนที่ 1698: Caching ใน SSR Applications

### Next.js Caching Layers

```javascript
// 1. Request Memoization
// fetch ที่เรียก URL เดียวกันใน request เดียว จะ memoize อัตโนมัติ
async function getUser(id) {
  const res = await fetch(`https://api.example.com/users/${id}`);
  return res.json();
}

// ทั้งสอง component เรียก getUser(1) แต่ HTTP request จริงแค่ 1 ครั้ง
async function UserName({ userId }) {
  const user = await getUser(userId); // fetch ครั้งที่ 1
  return <span>{user.name}</span>;
}

async function UserEmail({ userId }) {
  const user = await getUser(userId); // memoized! ไม่ fetch ซ้ำ
  return <span>{user.email}</span>;
}
```

```javascript
// 2. Data Cache (Full Route Cache)
// fetch กับ cache options
export default async function Page() {
  // cache ตลอดไป (ค่า default ใน Next.js 13)
  const staticData = await fetch('https://api.example.com/static');
  
  // ไม่ cache (SSR behavior)
  const dynamicData = await fetch('https://api.example.com/dynamic', {
    cache: 'no-store',
  });
  
  // cache กับ revalidation
  const revalidatedData = await fetch('https://api.example.com/products', {
    next: { revalidate: 3600 }, // 1 ชั่วโมง
  });
  
  // cache ด้วย tags
  const taggedData = await fetch('https://api.example.com/items', {
    next: { tags: ['items'] },
  });
}
```

```javascript
// 3. Router Cache (Client-side)
// Next.js cache navigation ใน browser อัตโนมัติ
// prefetch หน้าที่ Link ชี้ไป

import Link from 'next/link';

function Navigation() {
  return (
    <nav>
      {/* Next.js prefetch หน้านี้โดยอัตโนมัติ */}
      <Link href="/products" prefetch={true}>สินค้า</Link>
      
      {/* ปิด prefetch */}
      <Link href="/admin" prefetch={false}>Admin</Link>
    </nav>
  );
}
```

### Cache Invalidation

```javascript
// app/actions/products.js
'use server';

import { revalidatePath, revalidateTag } from 'next/cache';

export async function updateProduct(id, data) {
  await db.update('products').where({ id }).set(data);
  
  // Revalidate specific path
  revalidatePath('/products');
  revalidatePath(`/products/${id}`);
  
  // Revalidate by tag
  revalidateTag('products');
}

export async function createProduct(data) {
  const product = await db.insert('products').values(data);
  
  revalidateTag('products');
  revalidatePath('/products');
  
  return product;
}
```

---

## ขั้นตอนที่ 1699: Revalidation Patterns

### On-demand Revalidation

```javascript
// app/api/revalidate/route.js
import { revalidatePath, revalidateTag } from 'next/cache';
import { headers } from 'next/headers';

export async function POST(request) {
  // ตรวจสอบ secret token
  const headersList = headers();
  const secret = headersList.get('x-revalidate-token');
  
  if (secret !== process.env.REVALIDATE_TOKEN) {
    return Response.json({ error: 'Unauthorized' }, { status: 401 });
  }
  
  const body = await request.json();
  
  if (body.type === 'path') {
    revalidatePath(body.path);
    return Response.json({ revalidated: true, path: body.path });
  }
  
  if (body.type === 'tag') {
    revalidateTag(body.tag);
    return Response.json({ revalidated: true, tag: body.tag });
  }
  
  return Response.json({ error: 'Invalid type' }, { status: 400 });
}
```

### Time-based Revalidation

```javascript
// Revalidate ทุก N วินาที (ISR)
export const revalidate = 60; // ทั้ง page segment นี้ revalidate ทุก 60 วินาที

export default async function Page() {
  const data = await fetch('https://api.example.com/data', {
    next: { revalidate: 60 }
  });
  
  return <div>{/* content */}</div>;
}
```

---

## ขั้นตอนที่ 1700: Hydration และ Hydration Errors

### Hydration คืออะไร

```javascript
// Server ส่ง HTML นี้มา:
// <div id="root"><button>Count: 0</button></div>

// Client รับ HTML แล้ว React "hydrate" (attach event handlers):
import { hydrateRoot } from 'react-dom/client';
import App from './App';

hydrateRoot(
  document.getElementById('root'),
  <App /> // React rebuild virtual DOM แล้วเปรียบเทียบกับ real DOM
);
```

### Hydration Mismatch Errors

```javascript
// ปัญหาที่พบบ่อย: Server HTML ≠ Client render

// ตัวอย่าง ERROR: ใช้ Date/Math.random() โดยตรง
function BadComponent() {
  // ERROR: server render วันที่หนึ่ง, client render อีกวันที่
  return <div>เวลาปัจจุบัน: {new Date().toLocaleTimeString()}</div>;
}

// ตัวอย่าง ERROR: ใช้ localStorage
function BadComponent2() {
  // ERROR: localStorage ไม่มีบน server
  const theme = localStorage.getItem('theme'); // ReferenceError on server
  return <div className={theme}>Content</div>;
}

// การแก้ไข: ใช้ useEffect หรือ suppressHydrationWarning
function GoodComponent() {
  const [time, setTime] = useState('');
  
  useEffect(() => {
    // รันเฉพาะ client-side
    setTime(new Date().toLocaleTimeString());
    
    const interval = setInterval(() => {
      setTime(new Date().toLocaleTimeString());
    }, 1000);
    
    return () => clearInterval(interval);
  }, []);
  
  return <div>เวลาปัจจุบัน: {time || 'กำลังโหลด...'}</div>;
}
```

```javascript
// แก้ปัญหา localStorage
function ThemeComponent() {
  const [theme, setTheme] = useState('light'); // default value
  const [mounted, setMounted] = useState(false);
  
  useEffect(() => {
    setMounted(true);
    const savedTheme = localStorage.getItem('theme') || 'light';
    setTheme(savedTheme);
  }, []);
  
  // render เหมือนกันทั้ง server และ initial client render
  if (!mounted) return <div className="light">Content</div>;
  
  return <div className={theme}>Content</div>;
}
```

```javascript
// suppressHydrationWarning สำหรับ dynamic content ที่ตั้งใจ
function Clock() {
  return (
    <time
      suppressHydrationWarning
      dateTime={new Date().toISOString()}
    >
      {new Date().toLocaleString()}
    </time>
  );
}
```

---

## ขั้นตอนที่ 1701: Selective Hydration

```javascript
// React 18+ Selective Hydration
// ใช้ Suspense เพื่อให้ React hydrate แบบ selective

import { Suspense } from 'react';

// ส่วนนี้ hydrate ก่อน (ถ้า user interact)
function CriticalInteractiveSection() {
  return (
    <div>
      <SearchBar />
      <NavigationMenu />
    </div>
  );
}

// ส่วนนี้ hydrate ทีหลัง
function LessImportantSection() {
  return (
    <div>
      <Comments />
      <RelatedPosts />
      <Newsletter />
    </div>
  );
}

export default function Page() {
  return (
    <div>
      {/* Hydrate ทันที */}
      <CriticalInteractiveSection />
      
      {/* Lazy hydration - ใช้ dynamic import */}
      <Suspense fallback={null}>
        <LessImportantSection />
      </Suspense>
    </div>
  );
}
```

```javascript
// next/dynamic สำหรับ lazy loading
import dynamic from 'next/dynamic';

// ไม่ SSR component นี้เลย
const HeavyMap = dynamic(() => import('@/components/Map'), {
  ssr: false,
  loading: () => <div>กำลังโหลดแผนที่...</div>,
});

// SSR แต่ hydrate lazy
const Chart = dynamic(() => import('@/components/Chart'), {
  ssr: true,
  loading: () => <div>กำลังโหลดกราฟ...</div>,
});

export default function Page() {
  return (
    <div>
      <HeavyMap /> {/* ไม่ render บน server */}
      <Chart />   {/* render บน server แต่ hydrate lazy */}
    </div>
  );
}
```

---

## ขั้นตอนที่ 1702: Islands Architecture

Islands Architecture คือแนวคิดที่หน้าเว็บส่วนใหญ่เป็น static HTML (ไม่มี JavaScript) มีเฉพาะ "islands" ที่ต้องการ interactivity เท่านั้นที่มี JavaScript

```javascript
// Astro: Islands Architecture
---
// Component script (runs on server)
import ProductList from '../components/ProductList.astro';
import AddToCartButton from '../components/AddToCartButton.jsx';

const products = await fetch('/api/products').then(r => r.json());
---

<html>
  <body>
    <!-- Static HTML: ไม่มี JavaScript -->
    <header>
      <h1>ร้านค้าออนไลน์</h1>
    </header>
    
    <!-- Static Island: render บน server ไม่มี JS -->
    <ProductList products={products} />
    
    <!-- Interactive Island: มี JavaScript (React/Vue/Svelte) -->
    <AddToCartButton client:load productId="123" />
    
    <!-- Lazy Island: โหลด JS เมื่อ visible -->
    <Newsletter client:visible />
    
    <!-- Idle Island: โหลด JS เมื่อ browser idle -->
    <ChatWidget client:idle />
  </body>
</html>
```

```javascript
// React ใน Astro (Island)
// src/components/AddToCartButton.jsx
import { useState } from 'react';

export default function AddToCartButton({ productId }) {
  const [added, setAdded] = useState(false);
  
  return (
    <button
      onClick={() => {
        addToCart(productId);
        setAdded(true);
      }}
      className={added ? 'added' : ''}
    >
      {added ? 'เพิ่มแล้ว ✓' : 'เพิ่มลงตะกร้า'}
    </button>
  );
}
```

---

## ขั้นตอนที่ 1703: Nuxt.js สำหรับ Vue SSR

```javascript
// nuxt.config.ts
export default defineNuxtConfig({
  ssr: true, // เปิด SSR (default)
  
  // หรือเลือกแบบ Hybrid
  routeRules: {
    '/': { prerender: true },     // SSG
    '/products/**': { ssr: true }, // SSR
    '/admin/**': { ssr: false },   // SPA
  },
});
```

```vue
<!-- pages/products/index.vue -->
<template>
  <div>
    <h1>สินค้าทั้งหมด</h1>
    <div v-if="pending">กำลังโหลด...</div>
    <div v-else-if="error">เกิดข้อผิดพลาด: {{ error.message }}</div>
    <ul v-else>
      <li v-for="product in products" :key="product.id">
        {{ product.name }} - {{ product.price }}
      </li>
    </ul>
  </div>
</template>

<script setup>
// useFetch รัน SSR + client
const { data: products, pending, error } = await useFetch('/api/products');

// useAsyncData สำหรับ custom data fetching
const { data: stats } = await useAsyncData('stats', () => 
  $fetch('/api/stats')
);
</script>
```

```javascript
// server/api/products.get.ts (Nuxt server routes)
import { defineEventHandler, getQuery } from 'h3';

export default defineEventHandler(async (event) => {
  const query = getQuery(event);
  const page = parseInt(query.page as string) || 1;
  
  const products = await useStorage().getItem('products') 
    || await fetchFromDatabase();
  
  return {
    products: products.slice((page-1)*20, page*20),
    total: products.length,
    page,
  };
});
```

---

## ขั้นตอนที่ 1704: Remix.js Concepts

```javascript
// Remix: Web Standards First
// app/routes/products.$id.tsx

import { json, redirect } from '@remix-run/node';
import { useLoaderData, Form, useActionData } from '@remix-run/react';

// loader = getServerSideProps
export async function loader({ params, request }) {
  const product = await getProduct(params.id);
  
  if (!product) {
    throw new Response('ไม่พบสินค้า', { status: 404 });
  }
  
  return json({ product });
}

// action = form submission handler
export async function action({ params, request }) {
  const formData = await request.formData();
  const quantity = parseInt(formData.get('quantity'));
  
  if (quantity < 1) {
    return json({ error: 'จำนวนต้องมากกว่า 0' }, { status: 400 });
  }
  
  await addToCart(params.id, quantity);
  
  return redirect('/cart');
}

export default function ProductPage() {
  const { product } = useLoaderData();
  const actionData = useActionData();
  
  return (
    <div>
      <h1>{product.name}</h1>
      <p>฿{product.price.toLocaleString()}</p>
      
      <Form method="post">
        <input type="number" name="quantity" defaultValue={1} min={1} />
        {actionData?.error && <p className="error">{actionData.error}</p>}
        <button type="submit">เพิ่มลงตะกร้า</button>
      </Form>
    </div>
  );
}
```

```javascript
// Remix Error Boundary
export function ErrorBoundary() {
  const error = useRouteError();
  
  if (isRouteErrorResponse(error)) {
    return (
      <div>
        <h1>{error.status} - {error.statusText}</h1>
        <p>{error.data}</p>
      </div>
    );
  }
  
  return (
    <div>
      <h1>เกิดข้อผิดพลาดที่ไม่คาดคิด</h1>
      <p>{error.message}</p>
    </div>
  );
}
```

---

## ขั้นตอนที่ 1705: Astro Overview

```javascript
// astro.config.mjs
import { defineConfig } from 'astro/config';
import react from '@astrojs/react';
import tailwind from '@astrojs/tailwind';

export default defineConfig({
  integrations: [react(), tailwind()],
  output: 'hybrid', // static + SSR
  adapter: node({ mode: 'standalone' }),
});
```

```astro
---
// src/pages/blog/[slug].astro
import Layout from '../../layouts/Layout.astro';
import { getCollection } from 'astro:content';

export async function getStaticPaths() {
  const posts = await getCollection('blog');
  return posts.map(post => ({
    params: { slug: post.slug },
    props: { post },
  }));
}

const { post } = Astro.props;
const { Content } = await post.render();
---

<Layout title={post.data.title}>
  <article>
    <h1>{post.data.title}</h1>
    <time>{post.data.date.toLocaleDateString('th-TH')}</time>
    <Content />
  </article>
</Layout>
```

```astro
---
// src/pages/shop.astro (SSR mode)
export const prerender = false; // ปิด SSG สำหรับ page นี้

const products = await fetch('https://api.example.com/products')
  .then(r => r.json());
---

<html>
<body>
  <h1>ร้านค้า</h1>
  {products.map(p => (
    <div>
      <h2>{p.name}</h2>
      <p>฿{p.price}</p>
    </div>
  ))}
</body>
</html>
```

---

## ขั้นตอนที่ 1706: SvelteKit Overview

```javascript
// svelte.config.js
import adapter from '@sveltejs/adapter-node'; // หรือ adapter-vercel, adapter-cloudflare

export default {
  kit: {
    adapter: adapter(),
  },
};
```

```javascript
// src/routes/products/+page.server.js
// load function = getServerSideProps
export async function load({ params, fetch, cookies, request }) {
  const session = cookies.get('session');
  
  if (!session) {
    redirect(307, '/login');
  }
  
  const [products, categories] = await Promise.all([
    fetch('/api/products').then(r => r.json()),
    fetch('/api/categories').then(r => r.json()),
  ]);
  
  return {
    products,
    categories,
  };
}

// actions = form handlers
export const actions = {
  addToCart: async ({ request, cookies }) => {
    const data = await request.formData();
    const productId = data.get('productId');
    const quantity = data.get('quantity');
    
    await addToCart(cookies.get('userId'), productId, quantity);
    
    return { success: true };
  },
};
```

```svelte
<!-- src/routes/products/+page.svelte -->
<script>
  export let data; // จาก load function
  
  let { products, categories } = data;
</script>

<h1>สินค้าทั้งหมด</h1>

<form method="POST" action="?/addToCart">
  <select name="productId">
    {#each products as product}
      <option value={product.id}>{product.name}</option>
    {/each}
  </select>
  <input type="number" name="quantity" value="1" min="1" />
  <button type="submit">เพิ่มลงตะกร้า</button>
</form>
```

---

## ขั้นตอนที่ 1707: Edge SSR

### Next.js Edge Runtime

```javascript
// app/api/personalize/route.js
export const runtime = 'edge'; // ใช้ Edge Runtime

export async function GET(request) {
  const { searchParams } = new URL(request.url);
  const userId = searchParams.get('userId');
  
  // Edge Runtime: Web APIs เท่านั้น
  // - Fetch API
  // - Web Crypto API
  // - TextEncoder/TextDecoder
  // ไม่มี: Node.js fs, path, crypto (Node version), etc.
  
  const geo = request.geo; // Vercel geo data
  const country = geo?.country || 'TH';
  
  // ดึงข้อมูลแบบ personalize ตาม country
  const content = await fetchPersonalizedContent(userId, country);
  
  return Response.json(content);
}
```

```javascript
// middleware.js - Edge Middleware
import { NextResponse } from 'next/server';

export function middleware(request) {
  const { pathname } = request.nextUrl;
  const country = request.geo?.country || 'TH';
  
  // Redirect ตาม country
  if (pathname === '/') {
    if (country === 'US') {
      return NextResponse.redirect(new URL('/en', request.url));
    }
    if (country === 'TH') {
      return NextResponse.redirect(new URL('/th', request.url));
    }
  }
  
  // A/B Testing
  if (pathname === '/checkout') {
    const bucket = Math.random() > 0.5 ? 'A' : 'B';
    const response = NextResponse.next();
    response.cookies.set('ab-test', bucket);
    return response;
  }
  
  return NextResponse.next();
}

export const config = {
  matcher: ['/', '/checkout'],
};
```

---

## ขั้นตอนที่ 1708: Server Actions

```javascript
// app/actions.js
'use server';

import { revalidatePath } from 'next/cache';
import { redirect } from 'next/navigation';

export async function createPost(formData) {
  const title = formData.get('title');
  const content = formData.get('content');
  
  // Validate
  if (!title || title.length < 3) {
    return { error: 'ชื่อต้องมีอย่างน้อย 3 ตัวอักษร' };
  }
  
  // Save to database
  const post = await db.insert('posts').values({ title, content });
  
  // Revalidate cache
  revalidatePath('/posts');
  
  // Redirect
  redirect(`/posts/${post.id}`);
}
```

```javascript
// app/new-post/page.js
import { createPost } from '@/app/actions';

export default function NewPostPage() {
  return (
    <form action={createPost}>
      <input name="title" placeholder="ชื่อบทความ" required />
      <textarea name="content" placeholder="เนื้อหา" required />
      <button type="submit">สร้างบทความ</button>
    </form>
  );
}
```

```javascript
// ใช้กับ useFormState และ useFormStatus
'use client';
import { useFormState, useFormStatus } from 'react-dom';
import { createPost } from '@/app/actions';

function SubmitButton() {
  const { pending } = useFormStatus();
  
  return (
    <button type="submit" disabled={pending}>
      {pending ? 'กำลังสร้าง...' : 'สร้างบทความ'}
    </button>
  );
}

export default function NewPostForm() {
  const [state, formAction] = useFormState(createPost, null);
  
  return (
    <form action={formAction}>
      {state?.error && <p className="error">{state.error}</p>}
      <input name="title" placeholder="ชื่อบทความ" />
      <textarea name="content" placeholder="เนื้อหา" />
      <SubmitButton />
    </form>
  );
}
```

---

## ขั้นตอนที่ 1709: Performance Optimization ใน SSR

### Image Optimization

```javascript
// Next.js Image Component
import Image from 'next/image';

export default function ProductImage({ product }) {
  return (
    <Image
      src={product.imageUrl}
      alt={product.name}
      width={800}
      height={600}
      priority // สำหรับ above-the-fold images (LCP)
      placeholder="blur"
      blurDataURL={product.blurDataURL}
      sizes="(max-width: 768px) 100vw, 50vw"
    />
  );
}
```

### Font Optimization

```javascript
// app/layout.js
import { Sarabun } from 'next/font/google';

const sarabun = Sarabun({
  subsets: ['thai', 'latin'],
  weight: ['400', '700'],
  display: 'swap',
});

export default function RootLayout({ children }) {
  return (
    <html lang="th" className={sarabun.className}>
      <body>{children}</body>
    </html>
  );
}
```

### Bundle Analysis

```bash
# วิเคราะห์ bundle size
ANALYZE=true next build

# ดู bundle ใน browser
npx @next/bundle-analyzer
```

```javascript
// next.config.js
const withBundleAnalyzer = require('@next/bundle-analyzer')({
  enabled: process.env.ANALYZE === 'true',
});

module.exports = withBundleAnalyzer({
  // config ปกติ
});
```

---

## ขั้นตอนที่ 1710: Advanced SSR Patterns

### Partial Prerendering (PPR) - Next.js 14+

```javascript
// app/product/[id]/page.js
import { Suspense } from 'react';

// Static part: prerendered
async function ProductInfo({ id }) {
  const product = await getProductStatic(id);
  return (
    <div>
      <h1>{product.name}</h1>
      <p>{product.description}</p>
    </div>
  );
}

// Dynamic part: rendered at request time
async function DynamicPrice({ id }) {
  const pricing = await getRealtimePrice(id); // ข้อมูล real-time
  return <p>ราคา: ฿{pricing.current.toLocaleString()}</p>;
}

async function StockStatus({ id }) {
  const stock = await getRealtimeStock(id);
  return <p>สต็อก: {stock.available} ชิ้น</p>;
}

export default function ProductPage({ params }) {
  return (
    <div>
      {/* Static - prerendered */}
      <ProductInfo id={params.id} />
      
      {/* Dynamic - SSR per request */}
      <Suspense fallback={<PriceSkeleton />}>
        <DynamicPrice id={params.id} />
      </Suspense>
      
      <Suspense fallback={<StockSkeleton />}>
        <StockStatus id={params.id} />
      </Suspense>
    </div>
  );
}
```

### Parallel Routes

```javascript
// app/@modal/(.)[productId]/page.js - Intercepting Routes
// เปิด modal เมื่อ navigate แต่ full page เมื่อ refresh

export default function ProductModal({ params }) {
  return (
    <Modal>
      <ProductDetails id={params.productId} />
    </Modal>
  );
}
```

```javascript
// app/layout.js
export default function Layout({ children, modal }) {
  return (
    <>
      {children}
      {modal} {/* modal slot */}
    </>
  );
}
```

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: สร้าง SSR E-commerce Page
สร้าง Next.js App Router หน้า product listing ที่:
- ดึงข้อมูลจาก API (mock ได้)
- มี pagination
- มี SEO metadata
- ใช้ Suspense สำหรับ loading states
- แยก Server/Client components อย่างถูกต้อง

### แบบฝึกหัดที่ 2: Implement Caching Strategy
สร้าง caching layer ที่:
- Cache product list 5 นาที
- Cache individual product 1 ชั่วโมง
- Revalidate เมื่อมีการ update product
- มี API endpoint สำหรับ manual revalidation

### แบบฝึกหัดที่ 3: Fix Hydration Errors
แก้ไข component ต่อไปนี้:
```javascript
// Component นี้มี hydration error - แก้ไขให้ถูกต้อง
function ProblematicComponent() {
  const isLoggedIn = localStorage.getItem('isLoggedIn');
  const currentTime = new Date().toLocaleString();
  const randomId = Math.random().toString(36);
  
  return (
    <div id={randomId}>
      {isLoggedIn ? 'ยินดีต้อนรับ' : 'กรุณา Login'}
      <p>เวลา: {currentTime}</p>
    </div>
  );
}
```

### แบบฝึกหัดที่ 4: Edge Middleware
สร้าง Next.js middleware ที่:
- Redirect ผู้ใช้จากประเทศต่าง ๆ ไปยัง locale ที่เหมาะสม
- ทำ A/B testing สำหรับ homepage
- Block ตาม IP/Country
- Log request ไปยัง analytics

### แบบฝึกหัดที่ 5: SSR Performance
วิเคราะห์และปรับปรุง SSR performance:
- วัด TTFB, FCP, LCP
- ระบุ bottleneck ใน data fetching
- Implement parallel data fetching
- เพิ่ม appropriate caching

---

## สรุป

| Pattern | เมื่อใช้ | ตัวอย่าง |
|---------|----------|----------|
| SSR | ข้อมูล real-time + SEO | หน้าสินค้า, ราคา |
| SSG | เนื้อหาคงที่ | Blog, documentation |
| ISR | เนื้อหาเปลี่ยนบ้าง | Product catalog |
| CSR | Interactive app | Dashboard, admin |
| PPR | ส่วนผสม static + dynamic | Product page |
| Edge SSR | Personalization, low latency | A/B testing, geo |

ในส่วนถัดไปเราจะเรียนเรื่อง Edge Computing กับ JavaScript ซึ่งเกี่ยวข้องกัน SSR ที่ Edge!
