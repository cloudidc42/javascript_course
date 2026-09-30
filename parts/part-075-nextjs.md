# Part 75: Next.js (Steps 1471-1490)

## บทนำ

Next.js คือ React framework ที่พัฒนาโดย Vercel ซึ่งเพิ่มความสามารถพิเศษให้ React ได้แก่ Server-Side Rendering (SSR), Static Site Generation (SSG), App Router, Server Components และอื่นๆ อีกมากมาย

Next.js แก้ปัญหาหลักของ React SPA:
1. SEO ไม่ดีเพราะ content render ที่ client
2. Loading time แรกช้า
3. ไม่มี routing built-in
4. API routes ต้องการ server แยก

---

## Step 1471: Next.js คืออะไร

### ความสามารถหลักของ Next.js

```
Next.js Features:
✓ File-based Routing
✓ Server Components (React 18)
✓ Server Actions
✓ API Routes
✓ SSR, SSG, ISR
✓ Image Optimization
✓ Font Optimization
✓ TypeScript support
✓ CSS Modules / Tailwind
✓ Edge Runtime
✓ Deploy ง่ายบน Vercel
```

### App Router vs Pages Router

```
Pages Router (เดิม):
pages/
├── index.js          → /
├── about.js          → /about
├── users/
│   ├── index.js      → /users
│   └── [id].js       → /users/:id
└── api/
    └── hello.js      → /api/hello

App Router (ใหม่ - Next.js 13+):
app/
├── page.tsx          → /
├── layout.tsx        → root layout
├── about/
│   └── page.tsx      → /about
├── users/
│   ├── page.tsx      → /users
│   └── [id]/
│       └── page.tsx  → /users/:id
└── api/
    └── hello/
        └── route.ts  → /api/hello
```

---

## Step 1472: การสร้างโปรเจค Next.js

```bash
# สร้างโปรเจค Next.js ใหม่
npx create-next-app@latest my-app

# หรือกับ options
npx create-next-app@latest my-app \
  --typescript \
  --tailwind \
  --eslint \
  --app \
  --src-dir \
  --import-alias "@/*"

# โครงสร้างโปรเจค
my-app/
├── src/
│   └── app/
│       ├── layout.tsx
│       ├── page.tsx
│       ├── globals.css
│       └── favicon.ico
├── public/
├── next.config.js
├── package.json
├── tailwind.config.js
└── tsconfig.json
```

```javascript
// next.config.js
/** @type {import('next').NextConfig} */
const nextConfig = {
  // Enable experimental features
  experimental: {
    serverActions: true
  },
  
  // Image domains ที่อนุญาต
  images: {
    remotePatterns: [
      {
        protocol: 'https',
        hostname: 'images.unsplash.com'
      },
      {
        protocol: 'https',
        hostname: 'via.placeholder.com'
      }
    ]
  },
  
  // Redirect
  async redirects() {
    return [
      {
        source: '/old-page',
        destination: '/new-page',
        permanent: true
      }
    ]
  },
  
  // Rewrite (proxy)
  async rewrites() {
    return [
      {
        source: '/api/:path*',
        destination: 'https://external-api.com/:path*'
      }
    ]
  }
}

module.exports = nextConfig
```

---

## Step 1473: App Router Folder Structure

```
app/
├── layout.tsx              ← Root Layout (required)
├── page.tsx                ← Home page
├── loading.tsx             ← Loading UI
├── error.tsx               ← Error UI
├── not-found.tsx           ← 404 page
├── globals.css
│
├── about/
│   └── page.tsx
│
├── blog/
│   ├── layout.tsx          ← Nested layout
│   ├── page.tsx            ← Blog list
│   ├── loading.tsx
│   └── [slug]/
│       ├── page.tsx        ← Blog post
│       └── opengraph-image.tsx  ← OG Image
│
├── products/
│   ├── page.tsx
│   └── [id]/
│       └── page.tsx
│
├── (auth)/                 ← Route Group (ไม่ส่งผลต่อ URL)
│   ├── login/
│   │   └── page.tsx
│   └── register/
│       └── page.tsx
│
├── (dashboard)/            ← Route Group
│   ├── layout.tsx          ← Dashboard layout
│   ├── dashboard/
│   │   └── page.tsx
│   └── settings/
│       └── page.tsx
│
└── api/
    └── users/
        └── route.ts        ← API Route Handler
```

---

## Step 1474: page.tsx, layout.tsx, loading.tsx, error.tsx, not-found.tsx

```tsx
// app/layout.tsx - Root Layout
import type { Metadata } from 'next'
import { Inter } from 'next/font/google'
import './globals.css'

const inter = Inter({ subsets: ['latin'] })

// Metadata สำหรับ SEO
export const metadata: Metadata = {
  title: {
    default: 'My App',
    template: '%s | My App'
  },
  description: 'แอปพลิเคชันตัวอย่าง Next.js',
  keywords: ['nextjs', 'react', 'typescript'],
  authors: [{ name: 'สมชาย' }],
  openGraph: {
    type: 'website',
    locale: 'th_TH',
    url: 'https://myapp.com',
    title: 'My App',
    description: 'แอปพลิเคชันตัวอย่าง',
    images: [{ url: 'https://myapp.com/og.png' }]
  }
}

export default function RootLayout({
  children
}: {
  children: React.ReactNode
}) {
  return (
    <html lang="th">
      <body className={inter.className}>
        <header>
          <nav>เมนู</nav>
        </header>
        <main>{children}</main>
        <footer>Footer</footer>
      </body>
    </html>
  )
}
```

```tsx
// app/page.tsx - Home Page
import Link from 'next/link'

export default function HomePage() {
  return (
    <div>
      <h1>ยินดีต้อนรับ</h1>
      <p>นี่คือหน้าแรกของ Next.js App</p>
      <Link href="/about">เกี่ยวกับเรา</Link>
    </div>
  )
}
```

```tsx
// app/loading.tsx - Loading UI
export default function Loading() {
  return (
    <div className="loading-container">
      <div className="spinner" />
      <p>กำลังโหลด...</p>
    </div>
  )
}
```

```tsx
// app/error.tsx - Error Boundary
'use client'

import { useEffect } from 'react'

export default function Error({
  error,
  reset
}: {
  error: Error & { digest?: string }
  reset: () => void
}) {
  useEffect(() => {
    // Log error to service
    console.error(error)
  }, [error])

  return (
    <div>
      <h2>เกิดข้อผิดพลาด!</h2>
      <p>{error.message}</p>
      <button onClick={reset}>ลองใหม่</button>
    </div>
  )
}
```

```tsx
// app/not-found.tsx - 404 Page
import Link from 'next/link'

export default function NotFound() {
  return (
    <div>
      <h2>ไม่พบหน้านี้ (404)</h2>
      <p>ขออภัย ไม่พบหน้าที่คุณต้องการ</p>
      <Link href="/">กลับหน้าแรก</Link>
    </div>
  )
}
```

---

## Step 1475: Server Components vs Client Components

```tsx
// Server Component (DEFAULT ใน App Router)
// ทำงานบน server, ไม่มี JavaScript ส่งไป client
// ใช้ได้: fetch data, access backend directly, read env vars
// ใช้ไม่ได้: useState, useEffect, event handlers, browser APIs

// app/users/page.tsx
import { db } from '@/lib/db'

// ดึงข้อมูลจาก database โดยตรง
async function getUsersFromDB() {
  const users = await db.query('SELECT * FROM users')
  return users
}

export default async function UsersPage() {
  // Server Component สามารถ async ได้
  const users = await getUsersFromDB()
  
  return (
    <div>
      <h1>รายการผู้ใช้ ({users.length})</h1>
      <ul>
        {users.map(user => (
          <li key={user.id}>
            {user.name} - {user.email}
          </li>
        ))}
      </ul>
    </div>
  )
}
```

```tsx
// Client Component - ต้องมี 'use client' ที่บนสุด
// ทำงานบน browser
// ใช้ได้: useState, useEffect, event handlers, browser APIs
// ใช้ไม่ได้: direct DB access, read server-side env vars

'use client'

import { useState, useEffect } from 'react'

export default function Counter() {
  const [count, setCount] = useState(0)
  
  return (
    <div>
      <p>นับ: {count}</p>
      <button onClick={() => setCount(count + 1)}>+</button>
    </div>
  )
}
```

```tsx
// การผสม Server และ Client Components
// app/products/page.tsx (Server Component)
import ProductFilters from './_components/ProductFilters'  // Client
import ProductGrid from './_components/ProductGrid'        // Server

interface Product {
  id: number
  name: string
  price: number
  category: string
}

async function getProducts(): Promise<Product[]> {
  const res = await fetch('https://fakestoreapi.com/products', {
    cache: 'force-cache'  // static/cached
  })
  return res.json()
}

export default async function ProductsPage() {
  const products = await getProducts()
  
  return (
    <div>
      <h1>สินค้าทั้งหมด</h1>
      {/* Client Component สำหรับ interaction */}
      <ProductFilters />
      {/* Server Component สำหรับแสดงข้อมูล */}
      <ProductGrid products={products} />
    </div>
  )
}
```

---

## Step 1476: Routing - Nested Routes, Dynamic Routes, Catch-all

```tsx
// Nested Routes
// app/blog/layout.tsx
export default function BlogLayout({ children }: { children: React.ReactNode }) {
  return (
    <div className="blog-layout">
      <aside>
        <BlogSidebar />
      </aside>
      <main>{children}</main>
    </div>
  )
}

// app/blog/page.tsx → /blog
export default function BlogPage() {
  return <h1>บทความทั้งหมด</h1>
}

// app/blog/[slug]/page.tsx → /blog/:slug
interface BlogPostPageProps {
  params: { slug: string }
}

export default function BlogPostPage({ params }: BlogPostPageProps) {
  return <h1>บทความ: {params.slug}</h1>
}
```

```tsx
// Dynamic Routes
// app/users/[id]/page.tsx
interface UserPageProps {
  params: { id: string }
  searchParams: { tab?: string }
}

async function getUser(id: string) {
  const res = await fetch(`https://jsonplaceholder.typicode.com/users/${id}`)
  if (!res.ok) throw new Error('User not found')
  return res.json()
}

export default async function UserPage({ params, searchParams }: UserPageProps) {
  const user = await getUser(params.id)
  const activeTab = searchParams.tab || 'profile'
  
  return (
    <div>
      <h1>{user.name}</h1>
      <p>Tab: {activeTab}</p>
    </div>
  )
}

// Generate static paths (optional)
export async function generateStaticParams() {
  const users = await fetch('https://jsonplaceholder.typicode.com/users').then(r => r.json())
  return users.map((user: { id: number }) => ({
    id: user.id.toString()
  }))
}

// Generate metadata dynamically
export async function generateMetadata({ params }: UserPageProps) {
  const user = await getUser(params.id)
  return {
    title: user.name,
    description: `โปรไฟล์ของ ${user.name}`
  }
}
```

```tsx
// Catch-all Routes
// app/docs/[...slug]/page.tsx → /docs/a, /docs/a/b, /docs/a/b/c

interface DocsPageProps {
  params: { slug: string[] }
}

export default function DocsPage({ params }: DocsPageProps) {
  const path = params.slug?.join('/') || 'index'
  
  return (
    <div>
      <h1>เอกสาร: {path}</h1>
    </div>
  )
}

// Optional catch-all: app/docs/[[...slug]]/page.tsx
// ทำให้ /docs ก็ match ด้วย (slug เป็น undefined)
```

```tsx
// Route Groups
// (auth) ไม่ส่งผลต่อ URL
// app/(auth)/login/page.tsx → /login  (ไม่ใช่ /auth/login)
// app/(auth)/register/page.tsx → /register

// Parallel Routes
// app/@modal/page.tsx - render พร้อมกัน
// app/@main/page.tsx

// app/layout.tsx
export default function Layout({
  children,
  modal
}: {
  children: React.ReactNode
  modal: React.ReactNode
}) {
  return (
    <div>
      {children}
      {modal}
    </div>
  )
}
```

---

## Step 1477: Link Component และ Navigation

```tsx
'use client'

import Link from 'next/link'
import { useRouter, usePathname, useSearchParams } from 'next/navigation'

// Link Component
export function Navigation() {
  const pathname = usePathname()
  
  const navItems = [
    { href: '/', label: 'หน้าแรก' },
    { href: '/about', label: 'เกี่ยวกับ' },
    { href: '/blog', label: 'บทความ' },
    { href: '/contact', label: 'ติดต่อ' }
  ]
  
  return (
    <nav>
      {navItems.map(item => (
        <Link
          key={item.href}
          href={item.href}
          className={pathname === item.href ? 'active' : ''}
          prefetch={true}  // prefetch on hover (default)
        >
          {item.label}
        </Link>
      ))}
    </nav>
  )
}

// Programmatic Navigation
export function SearchForm() {
  const router = useRouter()
  const searchParams = useSearchParams()
  
  const handleSearch = (query: string) => {
    const params = new URLSearchParams(searchParams.toString())
    params.set('q', query)
    router.push(`/search?${params.toString()}`)
  }
  
  const goBack = () => router.back()
  const goForward = () => router.forward()
  const refresh = () => router.refresh()
  
  return (
    <div>
      <input onChange={e => handleSearch(e.target.value)} />
      <button onClick={goBack}>ย้อนกลับ</button>
    </div>
  )
}

// Link กับ Dynamic Route
export function UserLinks() {
  const users = [
    { id: 1, name: 'สมชาย' },
    { id: 2, name: 'สมหญิง' }
  ]
  
  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>
          {/* Object syntax */}
          <Link href={{ pathname: '/users/[id]', query: { id: user.id } }}>
            {user.name}
          </Link>
          {/* String syntax */}
          <Link href={`/users/${user.id}`}>{user.name}</Link>
        </li>
      ))}
    </ul>
  )
}
```

---

## Step 1478: Loading UI และ Suspense

```tsx
// app/products/loading.tsx - Automatic Loading UI
export default function ProductsLoading() {
  return (
    <div className="loading">
      {/* Skeleton Loading */}
      <div className="grid grid-cols-3 gap-4">
        {Array.from({ length: 6 }).map((_, i) => (
          <div key={i} className="skeleton-card animate-pulse">
            <div className="skeleton-image bg-gray-200 h-48 rounded" />
            <div className="skeleton-text bg-gray-200 h-4 rounded mt-2" />
            <div className="skeleton-text bg-gray-200 h-4 rounded mt-1 w-3/4" />
          </div>
        ))}
      </div>
    </div>
  )
}
```

```tsx
// ใช้ Suspense สำหรับ partial loading
import { Suspense } from 'react'

// app/dashboard/page.tsx
async function RevenueChart() {
  const data = await fetch('/api/revenue', { cache: 'no-store' }).then(r => r.json())
  return <Chart data={data} />
}

async function LatestInvoices() {
  const invoices = await fetch('/api/invoices/latest').then(r => r.json())
  return <InvoiceList invoices={invoices} />
}

export default function DashboardPage() {
  return (
    <div>
      <h1>Dashboard</h1>
      
      {/* แต่ละส่วน load แยกกัน ไม่รอกัน */}
      <Suspense fallback={<div>กำลังโหลด Chart...</div>}>
        <RevenueChart />
      </Suspense>
      
      <Suspense fallback={<div>กำลังโหลด Invoices...</div>}>
        <LatestInvoices />
      </Suspense>
    </div>
  )
}
```

---

## Step 1479: Server Actions

```tsx
// app/actions.ts - Server Actions
'use server'

import { revalidatePath } from 'next/cache'
import { redirect } from 'next/navigation'
import { z } from 'zod'  // npm install zod

// Schema validation
const CreatePostSchema = z.object({
  title: z.string().min(1, 'กรุณากรอกหัวข้อ').max(100),
  content: z.string().min(10, 'เนื้อหาต้องมีอย่างน้อย 10 ตัวอักษร'),
  category: z.enum(['tech', 'lifestyle', 'travel'])
})

// Server Action สำหรับสร้างโพสต์
export async function createPost(formData: FormData) {
  const validatedFields = CreatePostSchema.safeParse({
    title: formData.get('title'),
    content: formData.get('content'),
    category: formData.get('category')
  })
  
  if (!validatedFields.success) {
    return {
      errors: validatedFields.error.flatten().fieldErrors
    }
  }
  
  const { title, content, category } = validatedFields.data
  
  try {
    // บันทึกใน database
    await db.post.create({
      data: { title, content, category }
    })
    
    // Revalidate cache
    revalidatePath('/blog')
    
    // Redirect
    redirect('/blog')
  } catch (error) {
    return { errors: { general: ['เกิดข้อผิดพลาด กรุณาลองใหม่'] } }
  }
}

// Server Action สำหรับ delete
export async function deletePost(id: number) {
  await db.post.delete({ where: { id } })
  revalidatePath('/blog')
}

// Server Action สำหรับ toggle
export async function toggleLike(postId: number) {
  const session = await getSession()
  if (!session) redirect('/login')
  
  await db.like.upsert({
    where: { userId_postId: { userId: session.userId, postId } },
    create: { userId: session.userId, postId },
    update: {}
  })
  
  revalidatePath(`/blog/${postId}`)
}
```

```tsx
// ใช้ Server Actions ใน Form
'use client'

import { useFormState, useFormStatus } from 'react-dom'
import { createPost } from '@/app/actions'

// SubmitButton แยกออกมา เพื่อใช้ useFormStatus
function SubmitButton() {
  const { pending } = useFormStatus()
  
  return (
    <button type="submit" disabled={pending}>
      {pending ? 'กำลังบันทึก...' : 'บันทึก'}
    </button>
  )
}

// Form Component
export function CreatePostForm() {
  const [state, dispatch] = useFormState(createPost, undefined)
  
  return (
    <form action={dispatch}>
      <div>
        <label htmlFor="title">หัวข้อ</label>
        <input id="title" name="title" required />
        {state?.errors?.title && (
          <p className="error">{state.errors.title[0]}</p>
        )}
      </div>
      
      <div>
        <label htmlFor="content">เนื้อหา</label>
        <textarea id="content" name="content" rows={6} />
        {state?.errors?.content && (
          <p className="error">{state.errors.content[0]}</p>
        )}
      </div>
      
      <div>
        <label htmlFor="category">หมวดหมู่</label>
        <select id="category" name="category">
          <option value="tech">เทคโนโลยี</option>
          <option value="lifestyle">ไลฟ์สไตล์</option>
          <option value="travel">ท่องเที่ยว</option>
        </select>
      </div>
      
      {state?.errors?.general && (
        <p className="error">{state.errors.general[0]}</p>
      )}
      
      <SubmitButton />
    </form>
  )
}
```

---

## Step 1480: Data Fetching ใน Next.js

```tsx
// 1. Static Data (SSG) - cached indefinitely
async function getStaticData() {
  const res = await fetch('https://api.example.com/posts', {
    cache: 'force-cache'  // default
  })
  return res.json()
}

// 2. Dynamic Data (SSR) - ดึงใหม่ทุก request
async function getDynamicData() {
  const res = await fetch('https://api.example.com/user', {
    cache: 'no-store'  // ไม่ cache
  })
  return res.json()
}

// 3. Revalidate (ISR) - revalidate ทุก N วินาที
async function getRevalidatedData() {
  const res = await fetch('https://api.example.com/products', {
    next: { revalidate: 3600 }  // revalidate ทุก 1 ชั่วโมง
  })
  return res.json()
}

// 4. Tag-based revalidation
async function getTaggedData() {
  const res = await fetch('https://api.example.com/posts', {
    next: {
      tags: ['posts'],
      revalidate: 300
    }
  })
  return res.json()
}
```

```tsx
// Parallel Data Fetching - ดึงพร้อมกัน
export default async function DashboardPage() {
  // ✅ ดี: parallel fetching
  const [users, posts, analytics] = await Promise.all([
    fetch('/api/users').then(r => r.json()),
    fetch('/api/posts').then(r => r.json()),
    fetch('/api/analytics').then(r => r.json())
  ])
  
  // ❌ ไม่ดี: sequential fetching
  // const users = await fetch('/api/users').then(r => r.json())
  // const posts = await fetch('/api/posts').then(r => r.json())  // รอ users
  
  return (
    <div>
      <p>ผู้ใช้: {users.length}</p>
      <p>โพสต์: {posts.length}</p>
      <p>ผู้เยี่ยมชม: {analytics.visitors}</p>
    </div>
  )
}
```

```tsx
// Sequential Fetching เมื่อข้อมูลขึ้นอยู่กัน
export default async function UserWithPostsPage({ params }: { params: { id: string } }) {
  // ต้องได้ user ก่อน แล้วค่อยดึง posts
  const user = await fetch(`/api/users/${params.id}`).then(r => r.json())
  const posts = await fetch(`/api/users/${params.id}/posts`).then(r => r.json())
  
  return (
    <div>
      <h1>{user.name}</h1>
      <ul>{posts.map((p: any) => <li key={p.id}>{p.title}</li>)}</ul>
    </div>
  )
}
```

---

## Step 1481: Caching Strategies และ Revalidation

```tsx
// revalidatePath - revalidate ทุก route ที่ใช้ path นี้
import { revalidatePath } from 'next/cache'

export async function updateUser(id: number, data: any) {
  await db.user.update({ where: { id }, data })
  
  revalidatePath('/users')           // revalidate /users
  revalidatePath(`/users/${id}`)     // revalidate /users/:id
  revalidatePath('/users', 'layout') // revalidate layout
  revalidatePath('/users', 'page')   // revalidate page
}
```

```tsx
// revalidateTag - revalidate ทุก fetch ที่มี tag นี้
import { revalidateTag } from 'next/cache'

// Fetch data with tag
async function getProducts() {
  const res = await fetch('/api/products', {
    next: { tags: ['products', 'all'] }
  })
  return res.json()
}

// Revalidate by tag
export async function addProduct(data: any) {
  await db.product.create({ data })
  revalidateTag('products')  // revalidate ทุก fetch ที่ tag 'products'
}
```

```tsx
// On-demand Revalidation via API Route
// app/api/revalidate/route.ts
import { NextRequest } from 'next/server'
import { revalidatePath, revalidateTag } from 'next/cache'

export async function POST(request: NextRequest) {
  const secret = request.nextUrl.searchParams.get('secret')
  
  if (secret !== process.env.REVALIDATE_SECRET) {
    return Response.json({ message: 'Invalid secret' }, { status: 401 })
  }
  
  const body = await request.json()
  
  if (body.tag) {
    revalidateTag(body.tag)
    return Response.json({ revalidated: true, tag: body.tag })
  }
  
  if (body.path) {
    revalidatePath(body.path)
    return Response.json({ revalidated: true, path: body.path })
  }
  
  return Response.json({ message: 'tag or path required' }, { status: 400 })
}
```

---

## Step 1482: API Routes (Route Handlers)

```tsx
// app/api/users/route.ts
import { NextRequest, NextResponse } from 'next/server'

// GET /api/users
export async function GET(request: NextRequest) {
  const searchParams = request.nextUrl.searchParams
  const page = Number(searchParams.get('page') || '1')
  const limit = Number(searchParams.get('limit') || '10')
  
  try {
    const users = await db.user.findMany({
      skip: (page - 1) * limit,
      take: limit,
      orderBy: { createdAt: 'desc' }
    })
    
    const total = await db.user.count()
    
    return NextResponse.json({
      users,
      pagination: { page, limit, total, pages: Math.ceil(total / limit) }
    })
  } catch (error) {
    return NextResponse.json({ error: 'Internal Server Error' }, { status: 500 })
  }
}

// POST /api/users
export async function POST(request: NextRequest) {
  const body = await request.json()
  
  try {
    const user = await db.user.create({ data: body })
    return NextResponse.json(user, { status: 201 })
  } catch (error) {
    return NextResponse.json({ error: 'Create failed' }, { status: 400 })
  }
}
```

```tsx
// app/api/users/[id]/route.ts
import { NextRequest, NextResponse } from 'next/server'

interface RouteParams {
  params: { id: string }
}

// GET /api/users/:id
export async function GET(request: NextRequest, { params }: RouteParams) {
  const user = await db.user.findUnique({
    where: { id: Number(params.id) }
  })
  
  if (!user) {
    return NextResponse.json({ error: 'Not found' }, { status: 404 })
  }
  
  return NextResponse.json(user)
}

// PUT /api/users/:id
export async function PUT(request: NextRequest, { params }: RouteParams) {
  const body = await request.json()
  const user = await db.user.update({
    where: { id: Number(params.id) },
    data: body
  })
  return NextResponse.json(user)
}

// DELETE /api/users/:id
export async function DELETE(request: NextRequest, { params }: RouteParams) {
  await db.user.delete({ where: { id: Number(params.id) } })
  return new NextResponse(null, { status: 204 })
}
```

```tsx
// Headers, Cookies, และ Auth ใน Route Handlers
import { cookies, headers } from 'next/headers'

export async function GET(request: NextRequest) {
  // อ่าน headers
  const requestHeaders = headers()
  const authHeader = requestHeaders.get('authorization')
  
  // อ่าน cookies
  const cookieStore = cookies()
  const token = cookieStore.get('token')?.value
  
  // Set cookie ใน response
  const response = NextResponse.json({ data: 'ok' })
  response.cookies.set('session', 'abc123', {
    httpOnly: true,
    secure: process.env.NODE_ENV === 'production',
    maxAge: 60 * 60 * 24 * 7  // 1 week
  })
  
  return response
}
```

---

## Step 1483: Middleware

```tsx
// middleware.ts (root level)
import { NextResponse } from 'next/server'
import type { NextRequest } from 'next/server'

export function middleware(request: NextRequest) {
  const { pathname } = request.nextUrl
  
  // ตรวจสอบ authentication
  const token = request.cookies.get('token')?.value
  
  // Protected routes
  const protectedPaths = ['/dashboard', '/profile', '/settings']
  const isProtectedPath = protectedPaths.some(path => pathname.startsWith(path))
  
  if (isProtectedPath && !token) {
    const loginUrl = new URL('/login', request.url)
    loginUrl.searchParams.set('redirect', pathname)
    return NextResponse.redirect(loginUrl)
  }
  
  // Auth routes (redirect ถ้า login แล้ว)
  const authPaths = ['/login', '/register']
  const isAuthPath = authPaths.includes(pathname)
  
  if (isAuthPath && token) {
    return NextResponse.redirect(new URL('/dashboard', request.url))
  }
  
  // Rate limiting (basic)
  const ip = request.ip || 'anonymous'
  // ในโปรเจคจริง ใช้ Redis สำหรับ rate limiting
  
  // Geolocation
  const country = request.geo?.country || 'TH'
  
  // Add headers
  const response = NextResponse.next()
  response.headers.set('x-country', country)
  
  return response
}

// กำหนด paths ที่ middleware ทำงาน
export const config = {
  matcher: [
    // ทำงานกับทุก path ยกเว้น static files
    '/((?!_next/static|_next/image|favicon.ico).*)',
    
    // หรือเฉพาะบาง path
    '/dashboard/:path*',
    '/api/:path*'
  ]
}
```

---

## Step 1484: next/image Optimization

```tsx
import Image from 'next/image'

// Image Component
export function ProductImages() {
  return (
    <div>
      {/* Local Image */}
      <Image
        src="/images/product.jpg"
        alt="สินค้า"
        width={800}
        height={600}
        priority  // โหลดก่อน (LCP image)
        placeholder="blur"
        blurDataURL="data:image/jpeg;base64,..."
      />
      
      {/* Remote Image (ต้อง config ใน next.config.js) */}
      <Image
        src="https://images.unsplash.com/photo-..."
        alt="Remote image"
        width={400}
        height={300}
        quality={85}
      />
      
      {/* Fill parent container */}
      <div style={{ position: 'relative', height: '300px' }}>
        <Image
          src="/hero.jpg"
          alt="Hero"
          fill
          style={{ objectFit: 'cover' }}
          sizes="100vw"
        />
      </div>
      
      {/* Responsive sizes */}
      <Image
        src="/product.jpg"
        alt="Product"
        width={800}
        height={600}
        sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
      />
    </div>
  )
}
```

---

## Step 1485: next/font

```tsx
// app/layout.tsx
import { Inter, Noto_Sans_Thai } from 'next/font/google'
import localFont from 'next/font/local'

// Google Fonts
const inter = Inter({
  subsets: ['latin'],
  display: 'swap',
  variable: '--font-inter'
})

const notoSansThai = Noto_Sans_Thai({
  subsets: ['thai'],
  weight: ['300', '400', '500', '700'],
  display: 'swap',
  variable: '--font-noto-sans-thai'
})

// Local Font
const myFont = localFont({
  src: [
    { path: '../public/fonts/MyFont-Regular.woff2', weight: '400' },
    { path: '../public/fonts/MyFont-Bold.woff2', weight: '700' }
  ],
  variable: '--font-my-font'
})

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html lang="th" className={`${inter.variable} ${notoSansThai.variable}`}>
      <body className={notoSansThai.className}>
        {children}
      </body>
    </html>
  )
}
```

```css
/* globals.css */
:root {
  --font-inter: 'Inter', sans-serif;
  --font-noto-sans-thai: 'Noto Sans Thai', sans-serif;
}

body {
  font-family: var(--font-noto-sans-thai), var(--font-inter);
}
```

---

## Step 1486: Environment Variables

```bash
# .env.local (ไม่ commit)
DATABASE_URL="postgresql://user:password@localhost:5432/mydb"
JWT_SECRET="super-secret-key"
NEXT_PUBLIC_API_URL="https://api.example.com"
STRIPE_SECRET_KEY="sk_test_..."
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY="pk_test_..."

# .env (default, สามารถ commit)
NEXT_PUBLIC_APP_NAME="My App"
NEXT_PUBLIC_APP_URL="http://localhost:3000"
```

```tsx
// Server-side: ใช้ได้ทั้ง public และ private
// app/api/payment/route.ts
export async function POST() {
  const secretKey = process.env.STRIPE_SECRET_KEY  // ✅ server only
  const publicKey = process.env.NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY  // ✅
  // ...
}

// Client-side: ใช้ได้แค่ NEXT_PUBLIC_*
'use client'
export function PaymentForm() {
  const apiUrl = process.env.NEXT_PUBLIC_API_URL  // ✅ public
  // const secret = process.env.STRIPE_SECRET_KEY  // ❌ undefined บน client!
}

// Type-safe environment variables
declare namespace NodeJS {
  interface ProcessEnv {
    DATABASE_URL: string
    JWT_SECRET: string
    NEXT_PUBLIC_API_URL: string
  }
}
```

```tsx
// Validate environment variables at startup
import { z } from 'zod'

const envSchema = z.object({
  DATABASE_URL: z.string().url(),
  JWT_SECRET: z.string().min(32),
  NEXT_PUBLIC_API_URL: z.string().url()
})

// ถ้า .env ไม่ครบ จะ throw error ตอน start
const env = envSchema.parse(process.env)
export default env
```

---

## Step 1487: Authentication ใน Next.js

```tsx
// ใช้ NextAuth.js
// npm install next-auth

// app/api/auth/[...nextauth]/route.ts
import NextAuth from 'next-auth'
import CredentialsProvider from 'next-auth/providers/credentials'
import GoogleProvider from 'next-auth/providers/google'
import GitHubProvider from 'next-auth/providers/github'

const handler = NextAuth({
  providers: [
    CredentialsProvider({
      name: 'Credentials',
      credentials: {
        email: { label: "อีเมล", type: "email" },
        password: { label: "รหัสผ่าน", type: "password" }
      },
      async authorize(credentials) {
        if (!credentials?.email || !credentials?.password) return null
        
        const user = await db.user.findUnique({
          where: { email: credentials.email }
        })
        
        if (!user) return null
        
        const isValid = await bcrypt.compare(credentials.password, user.password)
        if (!isValid) return null
        
        return { id: user.id.toString(), email: user.email, name: user.name }
      }
    }),
    GoogleProvider({
      clientId: process.env.GOOGLE_CLIENT_ID!,
      clientSecret: process.env.GOOGLE_CLIENT_SECRET!
    })
  ],
  
  callbacks: {
    async jwt({ token, user }) {
      if (user) token.id = user.id
      return token
    },
    async session({ session, token }) {
      if (session.user) session.user.id = token.id as string
      return session
    }
  },
  
  pages: {
    signIn: '/login',
    error: '/auth/error'
  }
})

export { handler as GET, handler as POST }
```

```tsx
// ใช้ session ใน Server Component
import { getServerSession } from 'next-auth'
import { authOptions } from '@/lib/auth'

export default async function ProtectedPage() {
  const session = await getServerSession(authOptions)
  
  if (!session) {
    redirect('/login')
  }
  
  return (
    <div>
      <h1>สวัสดี, {session.user?.name}</h1>
    </div>
  )
}
```

---

## Step 1488: Complete Next.js App Example

```tsx
// app/(dashboard)/dashboard/page.tsx
import { Suspense } from 'react'
import { getServerSession } from 'next-auth'
import { redirect } from 'next/navigation'
import { authOptions } from '@/lib/auth'
import DashboardStats from './_components/DashboardStats'
import RecentOrders from './_components/RecentOrders'
import RevenueChart from './_components/RevenueChart'
import { StatsSkeleton, OrdersSkeleton, ChartSkeleton } from './_components/Skeletons'

export const metadata = {
  title: 'Dashboard'
}

export default async function DashboardPage() {
  const session = await getServerSession(authOptions)
  if (!session) redirect('/login')
  
  return (
    <div className="dashboard">
      <h1>Dashboard - สวัสดี {session.user?.name}</h1>
      
      {/* Parallel loading ด้วย Suspense */}
      <div className="stats-grid">
        <Suspense fallback={<StatsSkeleton />}>
          <DashboardStats userId={session.user.id} />
        </Suspense>
      </div>
      
      <div className="dashboard-content">
        <Suspense fallback={<ChartSkeleton />}>
          <RevenueChart />
        </Suspense>
        
        <Suspense fallback={<OrdersSkeleton />}>
          <RecentOrders />
        </Suspense>
      </div>
    </div>
  )
}
```

```tsx
// app/(dashboard)/dashboard/_components/DashboardStats.tsx
interface StatsData {
  totalRevenue: number
  totalOrders: number
  activeUsers: number
  conversionRate: number
}

async function getStats(userId: string): Promise<StatsData> {
  const res = await fetch(`${process.env.NEXT_PUBLIC_API_URL}/stats/${userId}`, {
    next: { revalidate: 60 }  // cache 1 นาที
  })
  return res.json()
}

export default async function DashboardStats({ userId }: { userId: string }) {
  const stats = await getStats(userId)
  
  const statCards = [
    { title: 'รายได้รวม', value: `฿${stats.totalRevenue.toLocaleString()}`, icon: '💰', color: 'blue' },
    { title: 'คำสั่งซื้อ', value: stats.totalOrders.toString(), icon: '📦', color: 'green' },
    { title: 'ผู้ใช้ active', value: stats.activeUsers.toString(), icon: '👥', color: 'purple' },
    { title: 'Conversion', value: `${stats.conversionRate}%`, icon: '📊', color: 'orange' }
  ]
  
  return (
    <div className="grid grid-cols-2 md:grid-cols-4 gap-4">
      {statCards.map(card => (
        <div key={card.title} className={`stat-card ${card.color}`}>
          <span className="icon">{card.icon}</span>
          <div>
            <p className="title">{card.title}</p>
            <p className="value">{card.value}</p>
          </div>
        </div>
      ))}
    </div>
  )
}
```

---

## Step 1489: Deployment กับ Vercel

```bash
# 1. สร้าง account บน vercel.com
# 2. ติดตั้ง Vercel CLI
npm install -g vercel

# 3. Deploy
vercel

# หรือ deploy ไปยัง production
vercel --prod

# Environment Variables ผ่าน CLI
vercel env add DATABASE_URL
vercel env add JWT_SECRET

# หรือผ่าน vercel.com/project/settings/environment-variables
```

```json
// vercel.json - configuration
{
  "buildCommand": "npm run build",
  "outputDirectory": ".next",
  "framework": "nextjs",
  "regions": ["sin1"],  // Singapore
  "env": {
    "NEXT_PUBLIC_APP_URL": "https://myapp.vercel.app"
  },
  "headers": [
    {
      "source": "/api/(.*)",
      "headers": [
        { "key": "Access-Control-Allow-Origin", "value": "*" },
        { "key": "Access-Control-Allow-Methods", "value": "GET,POST,PUT,DELETE,OPTIONS" }
      ]
    }
  ],
  "rewrites": [
    {
      "source": "/old-path",
      "destination": "/new-path"
    }
  ]
}
```

```bash
# Deploy ผ่าน GitHub Actions
# .github/workflows/deploy.yml
name: Deploy to Vercel
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: 18
      - run: npm ci
      - run: npm run build
      - uses: amondnet/vercel-action@v25
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.ORG_ID }}
          vercel-project-id: ${{ secrets.PROJECT_ID }}
```

---

## Step 1490: Best Practices และ Performance

```tsx
// 1. เลือก rendering strategy ให้เหมาะสม

// Static (SSG) - ข้อมูลไม่ค่อยเปลี่ยน
export const dynamic = 'force-static'

// Dynamic (SSR) - ข้อมูลเปลี่ยนทุก request
export const dynamic = 'force-dynamic'

// Default (static + revalidate)
export const revalidate = 3600  // revalidate ทุก 1 ชั่วโมง
```

```tsx
// 2. Optimize Images และ Fonts ด้วย next/image และ next/font

// 3. Code Splitting ด้วย dynamic import
import dynamic from 'next/dynamic'

const HeavyChart = dynamic(() => import('./HeavyChart'), {
  loading: () => <p>กำลังโหลด Chart...</p>,
  ssr: false  // ไม่ render ฝั่ง server
})

const ClientOnlyMap = dynamic(() => import('./Map'), { ssr: false })

// 4. Prefetching
import Link from 'next/link'

// Link prefetch อัตโนมัติเมื่อ hover (production)
<Link href="/products" prefetch={true}>สินค้า</Link>
```

```tsx
// 5. Bundle Analysis
// next.config.js
const withBundleAnalyzer = require('@next/bundle-analyzer')({
  enabled: process.env.ANALYZE === 'true'
})

module.exports = withBundleAnalyzer({
  // config
})

// รัน: ANALYZE=true npm run build
```

```tsx
// 6. Database Optimization - ใช้ Connection Pooling
import { PrismaClient } from '@prisma/client'

// ป้องกัน multiple connections ใน development
const globalForPrisma = global as unknown as { prisma: PrismaClient }

export const prisma =
  globalForPrisma.prisma ||
  new PrismaClient({
    log: process.env.NODE_ENV === 'development' ? ['query'] : []
  })

if (process.env.NODE_ENV !== 'production') globalForPrisma.prisma = prisma
```

```tsx
// 7. Error Handling ครบถ้วน
// app/error.tsx - Global error
// app/(dashboard)/error.tsx - Dashboard error
// app/api/users/route.ts - API error

export async function GET() {
  try {
    const data = await riskyOperation()
    return NextResponse.json(data)
  } catch (error) {
    if (error instanceof NotFoundError) {
      return NextResponse.json({ error: 'Not found' }, { status: 404 })
    }
    if (error instanceof UnauthorizedError) {
      return NextResponse.json({ error: 'Unauthorized' }, { status: 401 })
    }
    console.error('Unexpected error:', error)
    return NextResponse.json({ error: 'Internal server error' }, { status: 500 })
  }
}
```

```tsx
// 8. Type Safety ด้วย TypeScript + Zod
import { z } from 'zod'

// Schema validation
const UserSchema = z.object({
  id: z.number(),
  name: z.string(),
  email: z.string().email(),
  role: z.enum(['admin', 'user'])
})

type User = z.infer<typeof UserSchema>

// API validation
export async function POST(request: Request) {
  const body = await request.json()
  const result = UserSchema.omit({ id: true }).safeParse(body)
  
  if (!result.success) {
    return NextResponse.json({
      error: 'Validation failed',
      details: result.error.flatten()
    }, { status: 400 })
  }
  
  // result.data มี type ที่ถูกต้อง
  const user = await createUser(result.data)
  return NextResponse.json(user)
}
```

---

## Complete Full-Stack Example

```tsx
// app/blog/page.tsx - Blog listing with search
import { Suspense } from 'react'
import Link from 'next/link'
import Image from 'next/image'

interface Post {
  id: number
  title: string
  excerpt: string
  image: string
  author: string
  category: string
  publishedAt: string
  slug: string
}

async function getPosts(search?: string, category?: string): Promise<Post[]> {
  const params = new URLSearchParams()
  if (search) params.set('q', search)
  if (category && category !== 'all') params.set('category', category)
  
  const res = await fetch(
    `${process.env.NEXT_PUBLIC_API_URL}/posts?${params}`,
    { next: { revalidate: 300, tags: ['posts'] } }
  )
  
  if (!res.ok) throw new Error('Failed to fetch posts')
  return res.json()
}

interface BlogPageProps {
  searchParams: { q?: string; category?: string }
}

const categories = ['all', 'tech', 'lifestyle', 'travel', 'food']

export default async function BlogPage({ searchParams }: BlogPageProps) {
  const posts = await getPosts(searchParams.q, searchParams.category)
  
  return (
    <div className="blog-page">
      {/* Search */}
      <form className="search-form">
        <input
          name="q"
          defaultValue={searchParams.q}
          placeholder="ค้นหาบทความ..."
          className="search-input"
        />
        <button type="submit">ค้นหา</button>
      </form>
      
      {/* Categories */}
      <div className="categories">
        {categories.map(cat => (
          <Link
            key={cat}
            href={{ pathname: '/blog', query: { ...searchParams, category: cat } }}
            className={`category-btn ${searchParams.category === cat || (!searchParams.category && cat === 'all') ? 'active' : ''}`}
          >
            {cat === 'all' ? 'ทั้งหมด' : cat}
          </Link>
        ))}
      </div>
      
      {/* Posts Grid */}
      {posts.length === 0 ? (
        <p className="no-results">ไม่พบบทความที่ค้นหา</p>
      ) : (
        <div className="posts-grid">
          {posts.map(post => (
            <article key={post.id} className="post-card">
              <Link href={`/blog/${post.slug}`}>
                <div className="post-image">
                  <Image
                    src={post.image}
                    alt={post.title}
                    fill
                    sizes="(max-width: 768px) 100vw, 33vw"
                    className="object-cover"
                  />
                </div>
                <div className="post-content">
                  <span className="category">{post.category}</span>
                  <h2 className="title">{post.title}</h2>
                  <p className="excerpt">{post.excerpt}</p>
                  <div className="meta">
                    <span>{post.author}</span>
                    <span>{new Date(post.publishedAt).toLocaleDateString('th-TH')}</span>
                  </div>
                </div>
              </Link>
            </article>
          ))}
        </div>
      )}
    </div>
  )
}
```

---

## แบบฝึกหัด (Exercises)

### แบบฝึกหัดที่ 1: Personal Blog
สร้าง personal blog ด้วย Next.js App Router:
- หน้า list บทความ
- หน้า detail บทความ (dynamic route)
- Markdown rendering
- SEO optimization
- Static Generation

### แบบฝึกหัดที่ 2: E-Commerce Shop
สร้าง e-commerce shop:
- Product listing กับ search/filter
- Product detail page
- Shopping cart (Zustand + localStorage)
- Checkout form + Server Action
- Order confirmation

### แบบฝึกหัดที่ 3: Dashboard
สร้าง admin dashboard:
- Authentication (NextAuth)
- Protected routes (Middleware)
- Stats cards (Server Components)
- Data table กับ pagination
- Charts

### แบบฝึกหัดที่ 4: API + CRUD
สร้าง full CRUD app:
- Route Handlers สำหรับ REST API
- Server Actions สำหรับ forms
- Optimistic updates
- Error handling
- Input validation ด้วย Zod

### แบบฝึกหัดที่ 5: Real-time App
สร้าง app ที่มี real-time features:
- Server-Sent Events หรือ WebSocket
- Live notifications
- Real-time data updates
- Polling ด้วย SWR/React Query

---

## สรุปท้ายส่วน

ใน Part 75 นี้เราได้เรียนรู้:

- **Next.js คืออะไร**: React framework ที่เพิ่ม SSR, SSG, routing ฯลฯ
- **App Router**: ระบบ routing ใหม่ที่ใช้ folder structure
- **Special Files**: page.tsx, layout.tsx, loading.tsx, error.tsx, not-found.tsx
- **Server vs Client Components**: เลือกให้เหมาะกับงาน
- **Routing**: nested, dynamic, catch-all routes
- **Data Fetching**: static, dynamic, revalidate strategies
- **Server Actions**: form handling บน server
- **API Routes**: Route Handlers สำหรับ REST API
- **Middleware**: authentication, rate limiting, redirects
- **next/image**: image optimization อัตโนมัติ
- **next/font**: font optimization ไม่มี layout shift
- **Environment Variables**: public vs private
- **Deployment**: Vercel deployment

จบ Part 75 แล้ว! คุณได้เรียนรู้ React, Vue, Next.js ครบถ้วน พร้อมสำหรับการพัฒนา web application ระดับ production แล้ว!
