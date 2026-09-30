# Part 28: Fetch API
## ขั้นตอนที่ 531-550

---

## บทนำ

Fetch API คือ interface ที่ทันสมัยสำหรับทำ HTTP requests ใน JavaScript แทนที่ XMLHttpRequest (XHR) รุ่นเก่า มันสร้างบน Promises ทำให้ใช้งานง่ายและ integrate กับ async/await ได้อย่างสวยงาม

---

## ขั้นตอนที่ 531: Fetch API คืออะไร

```javascript
// ตัวอย่างที่ 1: การใช้ fetch พื้นฐาน
// fetch คืน Promise ที่ resolve เป็น Response object
fetch("https://jsonplaceholder.typicode.com/users/1")
  .then(response => {
    console.log("Status:", response.status);       // 200
    console.log("OK:", response.ok);               // true
    console.log("URL:", response.url);
    return response.json(); // แปลง body เป็น JSON
  })
  .then(user => {
    console.log("ผู้ใช้:", user.name);
    console.log("อีเมล:", user.email);
  })
  .catch(err => {
    console.error("ข้อผิดพลาด:", err.message);
  });
```

```javascript
// ตัวอย่างที่ 2: fetch ด้วย async/await
async function getUser(userId) {
  try {
    const response = await fetch(`https://jsonplaceholder.typicode.com/users/${userId}`);
    
    if (!response.ok) {
      throw new Error(`HTTP Error: ${response.status}`);
    }
    
    const user = await response.json();
    return user;
  } catch (err) {
    console.error("ไม่สามารถดึงข้อมูลผู้ใช้:", err.message);
    throw err;
  }
}

getUser(1).then(user => console.log("ผู้ใช้:", user.name));
```

---

## ขั้นตอนที่ 532: GET Request พื้นฐาน

```javascript
// ตัวอย่างที่ 3: GET request ง่ายๆ
async function fetchPosts() {
  const response = await fetch("https://jsonplaceholder.typicode.com/posts");
  const posts = await response.json();
  
  console.log(`ดึงข้อมูลได้ ${posts.length} โพสต์`);
  
  // แสดง 3 โพสต์แรก
  posts.slice(0, 3).forEach(post => {
    console.log(`[${post.id}] ${post.title}`);
  });
  
  return posts;
}

fetchPosts();
```

```javascript
// ตัวอย่างที่ 4: GET ด้วย query parameters
async function searchUsers(query) {
  const url = new URL("https://jsonplaceholder.typicode.com/users");
  url.searchParams.set("username", query);
  url.searchParams.set("_limit", "5");
  
  console.log("URL:", url.toString());
  
  const response = await fetch(url);
  return response.json();
}

searchUsers("Bret").then(users => {
  console.log("ผลการค้นหา:", users);
});
```

```javascript
// ตัวอย่างที่ 5: GET กับ URL parameters
async function getPostsByUser(userId) {
  const params = new URLSearchParams({
    userId: userId,
    _sort: "title",
    _order: "asc",
    _limit: "5"
  });
  
  const response = await fetch(
    `https://jsonplaceholder.typicode.com/posts?${params}`
  );
  
  const posts = await response.json();
  console.log(`โพสต์ของผู้ใช้ ${userId}:`, posts.length, "รายการ");
  return posts;
}

getPostsByUser(1);
```

---

## ขั้นตอนที่ 533: Response Object

```javascript
// ตัวอย่างที่ 6: คุณสมบัติของ Response object
async function exploreResponse() {
  const response = await fetch("https://jsonplaceholder.typicode.com/posts/1");
  
  // สถานะ
  console.log("status:", response.status);         // 200
  console.log("statusText:", response.statusText); // "OK"
  console.log("ok:", response.ok);                 // true (200-299)
  
  // URL
  console.log("url:", response.url);
  console.log("redirected:", response.redirected);
  
  // Headers
  console.log("Content-Type:", response.headers.get("Content-Type"));
  console.log("X-Powered-By:", response.headers.get("X-Powered-By"));
  
  // body (stream) - อ่านได้แค่ครั้งเดียว!
  console.log("bodyUsed:", response.bodyUsed); // false
  
  const data = await response.json();
  console.log("bodyUsed after json():", response.bodyUsed); // true
  
  return data;
}

exploreResponse();
```

```javascript
// ตัวอย่างที่ 7: Response.json()
async function parseJSON() {
  const response = await fetch("https://jsonplaceholder.typicode.com/posts/1");
  
  // response.json() แปลง JSON string เป็น JavaScript object
  const post = await response.json();
  
  console.log("ประเภท:", typeof post); // object
  console.log("ID:", post.id);
  console.log("หัวเรื่อง:", post.title);
  console.log("เนื้อหา:", post.body.substring(0, 50) + "...");
}

parseJSON();
```

```javascript
// ตัวอย่างที่ 8: Response.text()
async function parseText() {
  const response = await fetch("https://jsonplaceholder.typicode.com/posts/1");
  
  // response.text() คืน string ดิบๆ
  const text = await response.text();
  
  console.log("ประเภท:", typeof text); // string
  console.log("เนื้อหา:", text.substring(0, 100));
  
  // แปลงเอง
  const obj = JSON.parse(text);
  console.log("แปลงเป็น object:", obj.id);
}

parseText();
```

```javascript
// ตัวอย่างที่ 9: Response.blob() สำหรับไฟล์ binary
async function downloadImage() {
  const response = await fetch("https://via.placeholder.com/300x200");
  
  // response.blob() คืน Blob object (ไฟล์ binary)
  const blob = await response.blob();
  
  console.log("ขนาดไฟล์:", blob.size, "bytes");
  console.log("ประเภทไฟล์:", blob.type); // image/png
  
  // ใน browser สามารถสร้าง object URL ได้
  // const url = URL.createObjectURL(blob);
  // imgElement.src = url;
  
  return blob;
}

// downloadImage();
```

```javascript
// ตัวอย่างที่ 10: Response.arrayBuffer()
async function getArrayBuffer() {
  const response = await fetch("https://via.placeholder.com/100");
  const buffer = await response.arrayBuffer();
  
  console.log("ขนาด buffer:", buffer.byteLength, "bytes");
  
  // ใช้กับ Web Audio API, WebGL หรือการประมวลผล binary data
  const uint8Array = new Uint8Array(buffer);
  console.log("First bytes:", uint8Array.slice(0, 10));
}

// getArrayBuffer();
```

---

## ขั้นตอนที่ 534: การจัดการ Error ใน Fetch

```javascript
// ตัวอย่างที่ 11: ข้อผิดพลาดที่พบบ่อย
// fetch จะ reject เฉพาะ Network errors เท่านั้น!
// HTTP error codes (404, 500) ไม่ทำให้ reject!

async function fetchWithProperErrorHandling(url) {
  try {
    const response = await fetch(url);
    
    // ต้องตรวจสอบ response.ok เอง!
    if (!response.ok) {
      const errorBody = await response.text();
      throw new Error(
        `HTTP ${response.status}: ${response.statusText}\n${errorBody}`
      );
    }
    
    return await response.json();
  } catch (err) {
    if (err instanceof TypeError) {
      // Network error, DNS failure, CORS error
      console.error("Network Error:", err.message);
    } else {
      // HTTP error หรือ parsing error
      console.error("Error:", err.message);
    }
    throw err;
  }
}

// ทดสอบ
fetchWithProperErrorHandling("https://jsonplaceholder.typicode.com/posts/1")
  .then(post => console.log("สำเร็จ:", post.title));

fetchWithProperErrorHandling("https://jsonplaceholder.typicode.com/posts/9999")
  .catch(err => console.error("ไม่พบ:", err.message));
```

```javascript
// ตัวอย่างที่ 12: สร้าง utility function สำหรับ safe fetch
class FetchError extends Error {
  constructor(message, status, data) {
    super(message);
    this.name = "FetchError";
    this.status = status;
    this.data = data;
  }
}

async function safeFetch(url, options = {}) {
  const response = await fetch(url, options);
  
  let data;
  const contentType = response.headers.get("Content-Type") || "";
  
  // พยายามแปลง body
  if (contentType.includes("application/json")) {
    data = await response.json();
  } else {
    data = await response.text();
  }
  
  if (!response.ok) {
    throw new FetchError(
      `HTTP ${response.status}: ${response.statusText}`,
      response.status,
      data
    );
  }
  
  return data;
}

// ใช้งาน
safeFetch("https://jsonplaceholder.typicode.com/posts/1")
  .then(data => console.log("ข้อมูล:", data.title))
  .catch(err => {
    if (err instanceof FetchError) {
      console.error(`HTTP Error ${err.status}:`, err.message);
    } else {
      console.error("Network Error:", err.message);
    }
  });
```

---

## ขั้นตอนที่ 535: POST Request กับ JSON Body

```javascript
// ตัวอย่างที่ 13: POST request พื้นฐาน
async function createPost(postData) {
  const response = await fetch("https://jsonplaceholder.typicode.com/posts", {
    method: "POST",
    headers: {
      "Content-Type": "application/json"
    },
    body: JSON.stringify(postData)
  });
  
  if (!response.ok) {
    throw new Error(`HTTP Error: ${response.status}`);
  }
  
  const newPost = await response.json();
  console.log("สร้างโพสต์สำเร็จ! ID:", newPost.id);
  return newPost;
}

createPost({
  title: "โพสต์ใหม่ของฉัน",
  body: "เนื้อหาของโพสต์...",
  userId: 1
});
```

```javascript
// ตัวอย่างที่ 14: POST ด้วย complex data
async function createUser(userData) {
  const requestBody = {
    ...userData,
    createdAt: new Date().toISOString(),
    role: "user",
    settings: {
      notifications: true,
      newsletter: false
    }
  };
  
  const response = await fetch("https://jsonplaceholder.typicode.com/users", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "X-Request-ID": crypto.randomUUID?.() || Date.now().toString()
    },
    body: JSON.stringify(requestBody)
  });
  
  if (!response.ok) {
    const errorData = await response.json().catch(() => null);
    throw new Error(`Failed to create user: ${errorData?.message || response.statusText}`);
  }
  
  return response.json();
}

createUser({
  name: "สมชาย รักไทย",
  email: "somchai@example.com",
  phone: "0812345678"
}).then(user => console.log("สร้างผู้ใช้:", user));
```

---

## ขั้นตอนที่ 536: Request Headers

```javascript
// ตัวอย่างที่ 15: กำหนด headers ต่างๆ
async function fetchWithHeaders(url) {
  const response = await fetch(url, {
    headers: {
      // ประเภทเนื้อหาที่ต้องการ
      "Accept": "application/json",
      
      // สำหรับ POST/PUT
      "Content-Type": "application/json",
      
      // Custom headers
      "X-Api-Version": "2",
      "X-Request-Source": "web-app"
    }
  });
  
  return response.json();
}
```

```javascript
// ตัวอย่างที่ 16: Headers object
async function fetchWithHeadersObject(url) {
  const headers = new Headers();
  headers.append("Accept", "application/json");
  headers.append("X-Custom-Header", "value");
  
  // ตรวจสอบ headers
  console.log("Has Accept:", headers.has("Accept"));
  console.log("Accept value:", headers.get("Accept"));
  
  const response = await fetch(url, { headers });
  return response.json();
}
```

```javascript
// ตัวอย่างที่ 17: Headers จาก Response
async function inspectResponseHeaders() {
  const response = await fetch("https://jsonplaceholder.typicode.com/posts/1");
  
  // อ่าน headers ทั้งหมด
  response.headers.forEach((value, name) => {
    console.log(`${name}: ${value}`);
  });
  
  // อ่าน header เฉพาะ
  const contentType = response.headers.get("content-type");
  const cacheControl = response.headers.get("cache-control");
  
  console.log("Content-Type:", contentType);
  console.log("Cache-Control:", cacheControl);
}

inspectResponseHeaders();
```

---

## ขั้นตอนที่ 537: PUT, PATCH, DELETE Requests

```javascript
// ตัวอย่างที่ 18: PUT request - แทนที่ข้อมูลทั้งหมด
async function updatePost(postId, postData) {
  const response = await fetch(
    `https://jsonplaceholder.typicode.com/posts/${postId}`,
    {
      method: "PUT",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(postData)
    }
  );
  
  if (!response.ok) {
    throw new Error(`ไม่สามารถอัปเดตโพสต์: ${response.status}`);
  }
  
  const updated = await response.json();
  console.log("อัปเดตสำเร็จ:", updated);
  return updated;
}

updatePost(1, {
  id: 1,
  title: "หัวเรื่องที่อัปเดตแล้ว",
  body: "เนื้อหาใหม่ทั้งหมด",
  userId: 1
});
```

```javascript
// ตัวอย่างที่ 19: PATCH request - อัปเดตบางส่วน
async function patchPost(postId, changes) {
  const response = await fetch(
    `https://jsonplaceholder.typicode.com/posts/${postId}`,
    {
      method: "PATCH",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(changes)
    }
  );
  
  if (!response.ok) {
    throw new Error(`ไม่สามารถแก้ไขโพสต์: ${response.status}`);
  }
  
  const patched = await response.json();
  console.log("แก้ไขสำเร็จ:", patched);
  return patched;
}

// แก้ไขแค่หัวเรื่อง
patchPost(1, { title: "แก้ไขแค่หัวเรื่อง" });
```

```javascript
// ตัวอย่างที่ 20: DELETE request
async function deletePost(postId) {
  const response = await fetch(
    `https://jsonplaceholder.typicode.com/posts/${postId}`,
    {
      method: "DELETE"
    }
  );
  
  if (!response.ok) {
    throw new Error(`ไม่สามารถลบโพสต์: ${response.status}`);
  }
  
  // DELETE มักส่งกลับ 200 หรือ 204 (No Content)
  if (response.status === 204) {
    console.log(`ลบโพสต์ ${postId} สำเร็จ (No Content)`);
    return null;
  }
  
  const result = await response.json();
  console.log(`ลบโพสต์ ${postId} สำเร็จ`);
  return result;
}

deletePost(1);
```

---

## ขั้นตอนที่ 538: URL Parameters และ Query Strings

```javascript
// ตัวอย่างที่ 21: สร้าง URL ด้วย URLSearchParams
function buildUrl(baseUrl, params) {
  const url = new URL(baseUrl);
  
  Object.entries(params).forEach(([key, value]) => {
    if (value !== undefined && value !== null) {
      url.searchParams.append(key, value);
    }
  });
  
  return url.toString();
}

// ทดสอบ
const searchUrl = buildUrl("https://api.example.com/search", {
  q: "JavaScript",
  category: "tutorial",
  lang: "th",
  page: 1,
  limit: 20,
  sortBy: undefined  // จะถูกข้ามไป
});

console.log("URL:", searchUrl);
// https://api.example.com/search?q=JavaScript&category=tutorial&lang=th&page=1&limit=20
```

```javascript
// ตัวอย่างที่ 22: Query string ซับซ้อน
async function searchProducts(filters) {
  const {
    keyword = "",
    categories = [],
    minPrice,
    maxPrice,
    sort = "relevance",
    page = 1,
    pageSize = 20
  } = filters;
  
  const params = new URLSearchParams();
  
  if (keyword) params.set("q", keyword);
  
  // Array values
  categories.forEach(cat => params.append("category", cat));
  
  if (minPrice !== undefined) params.set("price_min", minPrice);
  if (maxPrice !== undefined) params.set("price_max", maxPrice);
  
  params.set("sort", sort);
  params.set("page", page);
  params.set("per_page", pageSize);
  
  const url = `https://api.shop.example.com/products?${params}`;
  console.log("Search URL:", url);
  
  // จริงๆ จะ fetch แต่ในตัวอย่างนี้แค่แสดง URL
  return url;
}

searchProducts({
  keyword: "iPhone",
  categories: ["smartphones", "electronics"],
  minPrice: 10000,
  maxPrice: 50000,
  sort: "price_asc"
});
```

---

## ขั้นตอนที่ 539: Authentication กับ Bearer Tokens

```javascript
// ตัวอย่างที่ 23: Bearer Token Authentication
class ApiClient {
  constructor(baseUrl) {
    this.baseUrl = baseUrl;
    this.token = null;
  }
  
  setToken(token) {
    this.token = token;
  }
  
  getAuthHeaders() {
    const headers = {
      "Content-Type": "application/json"
    };
    
    if (this.token) {
      headers["Authorization"] = `Bearer ${this.token}`;
    }
    
    return headers;
  }
  
  async get(path) {
    const response = await fetch(`${this.baseUrl}${path}`, {
      headers: this.getAuthHeaders()
    });
    
    if (response.status === 401) {
      throw new Error("ไม่ได้รับอนุญาต: Token หมดอายุหรือไม่ถูกต้อง");
    }
    
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`);
    }
    
    return response.json();
  }
  
  async post(path, data) {
    const response = await fetch(`${this.baseUrl}${path}`, {
      method: "POST",
      headers: this.getAuthHeaders(),
      body: JSON.stringify(data)
    });
    
    if (!response.ok) {
      const errorData = await response.json().catch(() => ({}));
      throw new Error(errorData.message || `HTTP ${response.status}`);
    }
    
    return response.json();
  }
}

// ใช้งาน
const client = new ApiClient("https://jsonplaceholder.typicode.com");
client.setToken("my-secret-jwt-token");

client.get("/posts/1")
  .then(post => console.log("โพสต์:", post.title))
  .catch(err => console.error("ข้อผิดพลาด:", err.message));
```

```javascript
// ตัวอย่างที่ 24: Auto token refresh
class AuthenticatedClient {
  constructor(baseUrl, authConfig) {
    this.baseUrl = baseUrl;
    this.accessToken = authConfig.accessToken;
    this.refreshToken = authConfig.refreshToken;
    this.onTokenRefresh = authConfig.onTokenRefresh;
  }
  
  async request(path, options = {}) {
    let response = await this.makeRequest(path, options);
    
    // ถ้า token หมดอายุ (401) ให้ refresh แล้วลองใหม่
    if (response.status === 401) {
      try {
        await this.refreshAccessToken();
        response = await this.makeRequest(path, options);
      } catch (refreshError) {
        throw new Error("ต้อง login ใหม่");
      }
    }
    
    if (!response.ok) {
      throw new Error(`HTTP ${response.status}`);
    }
    
    return response.json();
  }
  
  async makeRequest(path, options) {
    return fetch(`${this.baseUrl}${path}`, {
      ...options,
      headers: {
        "Content-Type": "application/json",
        "Authorization": `Bearer ${this.accessToken}`,
        ...options.headers
      }
    });
  }
  
  async refreshAccessToken() {
    const response = await fetch(`${this.baseUrl}/auth/refresh`, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ refreshToken: this.refreshToken })
    });
    
    if (!response.ok) {
      throw new Error("ไม่สามารถ refresh token ได้");
    }
    
    const { accessToken } = await response.json();
    this.accessToken = accessToken;
    
    if (this.onTokenRefresh) {
      this.onTokenRefresh(accessToken);
    }
  }
}
```

---

## ขั้นตอนที่ 540: Fetch กับ Async/Await

```javascript
// ตัวอย่างที่ 25: ดึงข้อมูลหลายๆ อย่างพร้อมกัน
async function loadDashboardData(userId) {
  const baseUrl = "https://jsonplaceholder.typicode.com";
  
  try {
    const [user, posts, todos] = await Promise.all([
      fetch(`${baseUrl}/users/${userId}`).then(r => r.json()),
      fetch(`${baseUrl}/posts?userId=${userId}&_limit=5`).then(r => r.json()),
      fetch(`${baseUrl}/todos?userId=${userId}&_limit=10`).then(r => r.json())
    ]);
    
    const completedTodos = todos.filter(t => t.completed).length;
    
    return {
      user,
      recentPosts: posts,
      todosSummary: {
        total: todos.length,
        completed: completedTodos,
        pending: todos.length - completedTodos
      }
    };
  } catch (err) {
    console.error("โหลด dashboard ล้มเหลว:", err.message);
    throw err;
  }
}

loadDashboardData(1).then(dashboard => {
  console.log("ผู้ใช้:", dashboard.user.name);
  console.log("โพสต์ล่าสุด:", dashboard.recentPosts.length, "รายการ");
  console.log("Todos:", dashboard.todosSummary);
});
```

```javascript
// ตัวอย่างที่ 26: Fetch กับ error handling ครบถ้วน
async function robustFetch(url, options = {}) {
  const {
    timeout = 10000,
    retries = 3,
    retryDelay = 1000,
    ...fetchOptions
  } = options;
  
  for (let attempt = 1; attempt <= retries; attempt++) {
    const controller = new AbortController();
    const timeoutId = setTimeout(() => controller.abort(), timeout);
    
    try {
      const response = await fetch(url, {
        ...fetchOptions,
        signal: controller.signal
      });
      
      clearTimeout(timeoutId);
      
      if (!response.ok) {
        const error = new Error(`HTTP ${response.status}: ${response.statusText}`);
        error.status = response.status;
        error.response = response;
        
        // ไม่ retry สำหรับ 4xx errors
        if (response.status >= 400 && response.status < 500) {
          throw error;
        }
        
        throw error;
      }
      
      return response;
    } catch (err) {
      clearTimeout(timeoutId);
      
      if (err.name === "AbortError") {
        throw new Error(`Request timeout หลังจาก ${timeout}ms`);
      }
      
      if (attempt === retries || (err.status >= 400 && err.status < 500)) {
        throw err;
      }
      
      console.log(`Retry ${attempt}/${retries} หลังจาก ${retryDelay}ms...`);
      await new Promise(resolve => setTimeout(resolve, retryDelay * attempt));
    }
  }
}

// ใช้งาน
robustFetch("https://jsonplaceholder.typicode.com/posts/1", {
  timeout: 5000,
  retries: 3
})
.then(res => res.json())
.then(post => console.log("ข้อมูล:", post.title))
.catch(err => console.error("ล้มเหลว:", err.message));
```

---

## ขั้นตอนที่ 541: AbortController เพื่อยกเลิก Request

```javascript
// ตัวอย่างที่ 27: ยกเลิก fetch ด้วย AbortController
const controller = new AbortController();
const { signal } = controller;

// เริ่ม request
fetch("https://jsonplaceholder.typicode.com/posts", { signal })
  .then(res => res.json())
  .then(posts => console.log("ได้รับ", posts.length, "โพสต์"))
  .catch(err => {
    if (err.name === "AbortError") {
      console.log("Request ถูกยกเลิก");
    } else {
      console.error("Error:", err.message);
    }
  });

// ยกเลิกหลังจาก 100ms
setTimeout(() => {
  controller.abort();
  console.log("ส่งคำสั่งยกเลิก...");
}, 100);
```

```javascript
// ตัวอย่างที่ 28: React-like component ที่ cleanup request
class DataLoader {
  constructor() {
    this.controllers = new Map();
  }
  
  async load(key, url) {
    // ยกเลิก request เก่าสำหรับ key เดียวกัน
    if (this.controllers.has(key)) {
      this.controllers.get(key).abort();
    }
    
    const controller = new AbortController();
    this.controllers.set(key, controller);
    
    try {
      const response = await fetch(url, {
        signal: controller.signal
      });
      
      if (!response.ok) {
        throw new Error(`HTTP ${response.status}`);
      }
      
      const data = await response.json();
      this.controllers.delete(key);
      return data;
    } catch (err) {
      this.controllers.delete(key);
      
      if (err.name !== "AbortError") {
        throw err;
      }
      
      return null; // ถูกยกเลิก
    }
  }
  
  cancelAll() {
    this.controllers.forEach(ctrl => ctrl.abort());
    this.controllers.clear();
  }
}

// ใช้งาน
const loader = new DataLoader();

// โหลด post ที่ 1
loader.load("post", "https://jsonplaceholder.typicode.com/posts/1")
  .then(post => post && console.log("Post 1:", post.title));

// โหลด post ที่ 2 ทันที (จะยกเลิก request แรก)
setTimeout(() => {
  loader.load("post", "https://jsonplaceholder.typicode.com/posts/2")
    .then(post => post && console.log("Post 2:", post.title));
}, 50);
```

---

## ขั้นตอนที่ 542: Fetch Timeout

```javascript
// ตัวอย่างที่ 29: สร้าง timeout wrapper
function fetchWithTimeout(url, options = {}, timeoutMs = 5000) {
  const controller = new AbortController();
  const timeoutId = setTimeout(() => controller.abort(), timeoutMs);
  
  return fetch(url, {
    ...options,
    signal: controller.signal
  })
  .then(response => {
    clearTimeout(timeoutId);
    return response;
  })
  .catch(err => {
    clearTimeout(timeoutId);
    if (err.name === "AbortError") {
      throw new Error(`Request timeout: เกิน ${timeoutMs}ms`);
    }
    throw err;
  });
}

// ใช้งาน
fetchWithTimeout("https://jsonplaceholder.typicode.com/posts/1", {}, 5000)
  .then(res => res.json())
  .then(data => console.log("ได้รับข้อมูล:", data.title))
  .catch(err => console.error("ข้อผิดพลาด:", err.message));
```

---

## ขั้นตอนที่ 543: CORS Overview

```javascript
// ตัวอย่างที่ 30: เข้าใจ CORS
// CORS = Cross-Origin Resource Sharing
// เป็น security mechanism ของ browser

// Same-origin: protocol + domain + port เหมือนกัน
// https://example.com/page.html ดึงข้อมูลจาก https://example.com/api ✓
// https://example.com/page.html ดึงข้อมูลจาก https://api.example.com ✗ (CORS)

// fetch mode สำหรับ CORS
async function fetchWithCorsOptions(url) {
  const response = await fetch(url, {
    mode: "cors",        // ค่าเริ่มต้น: ต้องการ CORS headers
    // mode: "no-cors",  // ไม่ต้องการ CORS headers (ได้ opaque response)
    // mode: "same-origin", // เฉพาะ same-origin เท่านั้น
    
    credentials: "omit",      // ค่าเริ่มต้น: ไม่ส่ง cookies
    // credentials: "include",  // ส่ง cookies ด้วย
    // credentials: "same-origin", // ส่ง cookies เฉพาะ same-origin
  });
  
  return response.json();
}

// ตัวอย่าง preflight request
// Browser จะส่ง OPTIONS request ก่อน ถ้า:
// 1. method ไม่ใช่ GET, HEAD, POST
// 2. Content-Type ไม่ใช่ text/plain, multipart/form-data, application/x-www-form-urlencoded
// 3. มี custom headers
```

```javascript
// ตัวอย่างที่ 31: Proxy pattern เพื่อแก้ปัญหา CORS
// ในกรณีที่ API ไม่รองรับ CORS
// สามารถใช้ backend proxy แทน

// Client -> Your Backend -> External API
// Your Backend มี CORS headers ที่ถูกต้อง

// Express.js example (backend)
/*
app.get('/api/external/:path', async (req, res) => {
  const response = await fetch(`https://external-api.com/${req.params.path}`);
  const data = await response.json();
  
  res.setHeader('Access-Control-Allow-Origin', 'https://your-frontend.com');
  res.json(data);
});
*/

// หรือใช้ CORS proxy services (สำหรับ development เท่านั้น)
async function fetchViaCorsProxy(targetUrl) {
  const proxyUrl = `https://corsproxy.io/?${encodeURIComponent(targetUrl)}`;
  const response = await fetch(proxyUrl);
  return response.json();
}
```

---

## ขั้นตอนที่ 544: การอัปโหลดไฟล์ด้วย FormData

```javascript
// ตัวอย่างที่ 32: อัปโหลดไฟล์เดียว
async function uploadFile(file) {
  const formData = new FormData();
  formData.append("file", file);
  formData.append("description", "รูปภาพของฉัน");
  formData.append("uploadedAt", new Date().toISOString());
  
  const response = await fetch("https://api.example.com/upload", {
    method: "POST",
    // ไม่ต้องกำหนด Content-Type - browser จะกำหนดให้เอง
    body: formData
  });
  
  if (!response.ok) {
    throw new Error("อัปโหลดล้มเหลว");
  }
  
  return response.json();
}

// จาก HTML <input type="file"> element
// const fileInput = document.querySelector('#file-input');
// const file = fileInput.files[0];
// uploadFile(file);
```

```javascript
// ตัวอย่างที่ 33: อัปโหลดหลายไฟล์พร้อมกัน
async function uploadMultipleFiles(files, metadata) {
  const formData = new FormData();
  
  // เพิ่มไฟล์หลายไฟล์
  Array.from(files).forEach((file, index) => {
    formData.append(`file_${index}`, file, file.name);
  });
  
  // เพิ่ม metadata
  formData.append("metadata", JSON.stringify(metadata));
  formData.append("userId", "123");
  
  const response = await fetch("https://api.example.com/upload/multiple", {
    method: "POST",
    body: formData
  });
  
  return response.json();
}
```

```javascript
// ตัวอย่างที่ 34: อัปโหลดพร้อม progress tracking
async function uploadWithProgress(file, onProgress) {
  return new Promise((resolve, reject) => {
    const xhr = new XMLHttpRequest();
    const formData = new FormData();
    formData.append("file", file);
    
    // Track upload progress
    xhr.upload.onprogress = (event) => {
      if (event.lengthComputable) {
        const percentage = Math.round((event.loaded / event.total) * 100);
        onProgress(percentage);
      }
    };
    
    xhr.onload = () => {
      if (xhr.status >= 200 && xhr.status < 300) {
        resolve(JSON.parse(xhr.responseText));
      } else {
        reject(new Error(`Upload failed: ${xhr.status}`));
      }
    };
    
    xhr.onerror = () => reject(new Error("Network error"));
    
    xhr.open("POST", "https://api.example.com/upload");
    xhr.send(formData);
  });
}

// ใช้งาน
// uploadWithProgress(file, (progress) => {
//   console.log(`อัปโหลด: ${progress}%`);
// });
```

---

## ขั้นตอนที่ 545: สร้าง Base URL Helper

```javascript
// ตัวอย่างที่ 35: Helper class สำหรับ base URL
class HttpClient {
  constructor(config = {}) {
    this.baseUrl = config.baseUrl || "";
    this.defaultHeaders = config.headers || {};
    this.timeout = config.timeout || 10000;
  }
  
  buildUrl(path, params = {}) {
    const url = new URL(path, this.baseUrl);
    
    Object.entries(params).forEach(([key, value]) => {
      if (value !== undefined && value !== null) {
        url.searchParams.set(key, value);
      }
    });
    
    return url.toString();
  }
  
  async request(method, path, options = {}) {
    const { params, body, headers = {} } = options;
    
    const url = this.buildUrl(path, params);
    const controller = new AbortController();
    const timeoutId = setTimeout(() => controller.abort(), this.timeout);
    
    try {
      const response = await fetch(url, {
        method,
        headers: {
          ...this.defaultHeaders,
          ...headers,
          ...(body ? { "Content-Type": "application/json" } : {})
        },
        body: body ? JSON.stringify(body) : undefined,
        signal: controller.signal
      });
      
      clearTimeout(timeoutId);
      
      if (!response.ok) {
        const errorData = await response.json().catch(() => null);
        const error = new Error(`HTTP ${response.status}: ${response.statusText}`);
        error.status = response.status;
        error.data = errorData;
        throw error;
      }
      
      // ตรวจสอบว่ามี response body ไหม
      const contentType = response.headers.get("Content-Type") || "";
      if (contentType.includes("application/json")) {
        return response.json();
      }
      
      return response.text();
    } catch (err) {
      clearTimeout(timeoutId);
      if (err.name === "AbortError") {
        throw new Error(`Timeout: ${url}`);
      }
      throw err;
    }
  }
  
  get(path, params) {
    return this.request("GET", path, { params });
  }
  
  post(path, body, options = {}) {
    return this.request("POST", path, { ...options, body });
  }
  
  put(path, body, options = {}) {
    return this.request("PUT", path, { ...options, body });
  }
  
  patch(path, body, options = {}) {
    return this.request("PATCH", path, { ...options, body });
  }
  
  delete(path, options = {}) {
    return this.request("DELETE", path, options);
  }
}

// ใช้งาน
const http = new HttpClient({
  baseUrl: "https://jsonplaceholder.typicode.com",
  headers: {
    "X-App-Version": "1.0.0"
  },
  timeout: 5000
});

async function testHttpClient() {
  const posts = await http.get("/posts", { _limit: 3 });
  console.log("โพสต์:", posts.map(p => p.title));
  
  const newPost = await http.post("/posts", {
    title: "โพสต์ใหม่",
    body: "เนื้อหา",
    userId: 1
  });
  console.log("สร้างโพสต์:", newPost);
}

testHttpClient();
```

---

## ขั้นตอนที่ 546: สร้าง API Client Class

```javascript
// ตัวอย่างที่ 36: API Client ครบถ้วน
class BlogApiClient {
  constructor(baseUrl) {
    this.http = new HttpClient({ baseUrl });
  }
  
  // Posts
  async getPosts(options = {}) {
    const { page = 1, limit = 10, userId } = options;
    return this.http.get("/posts", {
      _page: page,
      _limit: limit,
      userId
    });
  }
  
  async getPost(id) {
    return this.http.get(`/posts/${id}`);
  }
  
  async createPost(data) {
    return this.http.post("/posts", data);
  }
  
  async updatePost(id, data) {
    return this.http.put(`/posts/${id}`, { id, ...data });
  }
  
  async patchPost(id, changes) {
    return this.http.patch(`/posts/${id}`, changes);
  }
  
  async deletePost(id) {
    return this.http.delete(`/posts/${id}`);
  }
  
  // Comments
  async getPostComments(postId) {
    return this.http.get(`/posts/${postId}/comments`);
  }
  
  // Users
  async getUsers() {
    return this.http.get("/users");
  }
  
  async getUser(id) {
    return this.http.get(`/users/${id}`);
  }
  
  async getUserPosts(userId) {
    return this.http.get("/posts", { userId });
  }
}

// ใช้งาน
const api = new BlogApiClient("https://jsonplaceholder.typicode.com");

async function demonstrateBlogApi() {
  // ดึงโพสต์หน้าแรก
  const posts = await api.getPosts({ page: 1, limit: 5 });
  console.log("โพสต์:", posts.length, "รายการ");
  
  // ดึงโพสต์เดียว
  const post = await api.getPost(1);
  console.log("โพสต์ที่ 1:", post.title);
  
  // ดึงคอมเมนต์ของโพสต์
  const comments = await api.getPostComments(1);
  console.log("คอมเมนต์:", comments.length, "รายการ");
  
  // สร้างโพสต์ใหม่
  const newPost = await api.createPost({
    title: "โพสต์ใหม่ของฉัน",
    body: "เนื้อหาโพสต์",
    userId: 1
  });
  console.log("สร้างโพสต์ใหม่ ID:", newPost.id);
}

demonstrateBlogApi();
```

---

## ขั้นตอนที่ 547: Interceptors Pattern

```javascript
// ตัวอย่างที่ 37: Interceptors เหมือน Axios
class FetchInterceptor {
  constructor() {
    this.requestInterceptors = [];
    this.responseInterceptors = [];
  }
  
  addRequestInterceptor(onFulfilled, onRejected) {
    this.requestInterceptors.push({ onFulfilled, onRejected });
    return this.requestInterceptors.length - 1;
  }
  
  addResponseInterceptor(onFulfilled, onRejected) {
    this.responseInterceptors.push({ onFulfilled, onRejected });
    return this.responseInterceptors.length - 1;
  }
  
  async fetch(url, config = {}) {
    // Apply request interceptors
    let processedConfig = { url, ...config };
    for (const interceptor of this.requestInterceptors) {
      try {
        processedConfig = await interceptor.onFulfilled(processedConfig);
      } catch (err) {
        if (interceptor.onRejected) {
          processedConfig = await interceptor.onRejected(err);
        } else {
          throw err;
        }
      }
    }
    
    // Make actual request
    let response;
    try {
      response = await fetch(processedConfig.url, processedConfig);
    } catch (err) {
      for (const interceptor of this.responseInterceptors) {
        if (interceptor.onRejected) {
          try {
            response = await interceptor.onRejected(err);
          } catch (e) {
            throw e;
          }
        }
      }
      if (!response) throw err;
    }
    
    // Apply response interceptors
    for (const interceptor of this.responseInterceptors) {
      try {
        response = await interceptor.onFulfilled(response);
      } catch (err) {
        if (interceptor.onRejected) {
          response = await interceptor.onRejected(err);
        } else {
          throw err;
        }
      }
    }
    
    return response;
  }
}

// ใช้งาน
const fetchClient = new FetchInterceptor();

// เพิ่ม Auth interceptor
fetchClient.addRequestInterceptor(config => {
  const token = "my-auth-token";
  config.headers = {
    ...config.headers,
    Authorization: `Bearer ${token}`
  };
  console.log("Request interceptor: เพิ่ม token");
  return config;
});

// เพิ่ม Logging interceptor
fetchClient.addResponseInterceptor(async response => {
  console.log(`Response interceptor: ${response.status} ${response.url}`);
  return response;
});

fetchClient.fetch("https://jsonplaceholder.typicode.com/posts/1")
  .then(res => res.json())
  .then(data => console.log("ข้อมูล:", data.title));
```

---

## ขั้นตอนที่ 548: Caching Responses

```javascript
// ตัวอย่างที่ 38: Cache API ใน browser
async function fetchWithCache(url, maxAge = 300000) {
  // ใช้ browser Cache API
  if (!('caches' in window)) {
    return fetch(url).then(r => r.json());
  }
  
  const cache = await caches.open("api-cache-v1");
  const cachedResponse = await cache.match(url);
  
  if (cachedResponse) {
    const cachedAt = cachedResponse.headers.get("X-Cached-At");
    const age = Date.now() - parseInt(cachedAt);
    
    if (age < maxAge) {
      console.log("Cache hit:", url);
      return cachedResponse.json();
    }
    
    console.log("Cache expired:", url);
  }
  
  console.log("Fetching:", url);
  const response = await fetch(url);
  
  // สร้าง response ใหม่พร้อม custom header
  const responseToCache = new Response(await response.clone().blob(), {
    status: response.status,
    headers: {
      ...Object.fromEntries(response.headers),
      "X-Cached-At": Date.now().toString()
    }
  });
  
  await cache.put(url, responseToCache);
  return response.json();
}
```

```javascript
// ตัวอย่างที่ 39: Memory Cache แบบง่าย
class MemoryCache {
  constructor(defaultTtl = 5 * 60 * 1000) {
    this.cache = new Map();
    this.defaultTtl = defaultTtl;
  }
  
  set(key, value, ttl = this.defaultTtl) {
    this.cache.set(key, {
      value,
      expiresAt: Date.now() + ttl
    });
  }
  
  get(key) {
    const item = this.cache.get(key);
    if (!item) return null;
    
    if (Date.now() > item.expiresAt) {
      this.cache.delete(key);
      return null;
    }
    
    return item.value;
  }
  
  has(key) {
    return this.get(key) !== null;
  }
  
  delete(key) {
    this.cache.delete(key);
  }
  
  clear() {
    this.cache.clear();
  }
}

class CachedApiClient {
  constructor(baseUrl) {
    this.baseUrl = baseUrl;
    this.cache = new MemoryCache();
  }
  
  async get(path, params = {}) {
    const url = new URL(path, this.baseUrl);
    Object.entries(params).forEach(([k, v]) => url.searchParams.set(k, v));
    const cacheKey = url.toString();
    
    const cached = this.cache.get(cacheKey);
    if (cached) {
      console.log("Cache hit:", path);
      return cached;
    }
    
    const response = await fetch(cacheKey);
    const data = await response.json();
    
    this.cache.set(cacheKey, data);
    return data;
  }
  
  invalidate(path) {
    const url = new URL(path, this.baseUrl).toString();
    this.cache.delete(url);
  }
}

const cachedApi = new CachedApiClient("https://jsonplaceholder.typicode.com");

async function testCache() {
  console.time("first fetch");
  const posts1 = await cachedApi.get("/posts", { _limit: 10 });
  console.timeEnd("first fetch");
  
  console.time("second fetch (cached)");
  const posts2 = await cachedApi.get("/posts", { _limit: 10 });
  console.timeEnd("second fetch (cached)");
  
  console.log("Results same:", posts1.length === posts2.length);
}

testCache();
```

---

## ขั้นตอนที่ 549: Streaming Responses

```javascript
// ตัวอย่างที่ 40: อ่าน streaming response
async function fetchWithProgress(url, onProgress) {
  const response = await fetch(url);
  
  const contentLength = response.headers.get("Content-Length");
  const total = contentLength ? parseInt(contentLength) : null;
  
  const reader = response.body.getReader();
  const chunks = [];
  let received = 0;
  
  while (true) {
    const { done, value } = await reader.read();
    
    if (done) break;
    
    chunks.push(value);
    received += value.length;
    
    if (total && onProgress) {
      onProgress(Math.round((received / total) * 100));
    }
  }
  
  // รวม chunks
  const allBytes = new Uint8Array(received);
  let position = 0;
  for (const chunk of chunks) {
    allBytes.set(chunk, position);
    position += chunk.length;
  }
  
  return new TextDecoder("utf-8").decode(allBytes);
}

fetchWithProgress(
  "https://jsonplaceholder.typicode.com/posts",
  (progress) => console.log(`Progress: ${progress}%`)
).then(text => {
  const data = JSON.parse(text);
  console.log("ดาวน์โหลดสำเร็จ:", data.length, "โพสต์");
});
```

---

## ขั้นตอนที่ 550: แบบฝึกหัด

```javascript
// แบบฝึกหัดที่ 1: สร้าง TODO App API Client
class TodoApiClient {
  constructor() {
    this.baseUrl = "https://jsonplaceholder.typicode.com";
  }
  
  async getTodos(userId) {
    const params = userId ? `?userId=${userId}` : "";
    const response = await fetch(`${this.baseUrl}/todos${params}`);
    return response.json();
  }
  
  async createTodo(data) {
    const response = await fetch(`${this.baseUrl}/todos`, {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify(data)
    });
    return response.json();
  }
  
  async completeTodo(id) {
    const response = await fetch(`${this.baseUrl}/todos/${id}`, {
      method: "PATCH",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ completed: true })
    });
    return response.json();
  }
  
  async deleteTodo(id) {
    await fetch(`${this.baseUrl}/todos/${id}`, { method: "DELETE" });
    return { deleted: true, id };
  }
  
  async getStats(userId) {
    const todos = await this.getTodos(userId);
    const completed = todos.filter(t => t.completed);
    const pending = todos.filter(t => !t.completed);
    
    return {
      total: todos.length,
      completed: completed.length,
      pending: pending.length,
      completionRate: `${Math.round((completed.length / todos.length) * 100)}%`
    };
  }
}

const todoApi = new TodoApiClient();

async function demonstrateTodos() {
  // ดึง todos ทั้งหมดของผู้ใช้ที่ 1
  const todos = await todoApi.getTodos(1);
  console.log("Todos:", todos.slice(0, 3));
  
  // สร้าง todo ใหม่
  const newTodo = await todoApi.createTodo({
    title: "เรียนรู้ Fetch API",
    completed: false,
    userId: 1
  });
  console.log("Todo ใหม่:", newTodo);
  
  // ดูสถิติ
  const stats = await todoApi.getStats(1);
  console.log("สถิติ:", stats);
}

demonstrateTodos();
```

```javascript
// แบบฝึกหัดที่ 2: Multi-source Data Aggregator
async function aggregateUserData(userId) {
  const BASE = "https://jsonplaceholder.typicode.com";
  
  async function safeFetch(url) {
    try {
      const res = await fetch(url);
      return res.ok ? res.json() : null;
    } catch {
      return null;
    }
  }
  
  // ดึงข้อมูลทั้งหมดพร้อมกัน
  const [user, posts, todos, albums] = await Promise.all([
    safeFetch(`${BASE}/users/${userId}`),
    safeFetch(`${BASE}/posts?userId=${userId}`),
    safeFetch(`${BASE}/todos?userId=${userId}`),
    safeFetch(`${BASE}/albums?userId=${userId}`)
  ]);
  
  if (!user) throw new Error(`ไม่พบผู้ใช้ ID: ${userId}`);
  
  const completedTodos = (todos || []).filter(t => t.completed).length;
  
  return {
    user: {
      id: user.id,
      name: user.name,
      email: user.email,
      website: user.website,
      company: user.company?.name
    },
    statistics: {
      totalPosts: posts?.length || 0,
      totalTodos: todos?.length || 0,
      completedTodos,
      todoCompletionRate: todos?.length 
        ? `${Math.round(completedTodos / todos.length * 100)}%`
        : "N/A",
      totalAlbums: albums?.length || 0
    },
    recentPosts: (posts || []).slice(0, 3).map(p => ({
      id: p.id,
      title: p.title.substring(0, 50)
    }))
  };
}

aggregateUserData(1).then(data => {
  console.log("ข้อมูลผู้ใช้:", data.user.name);
  console.log("สถิติ:", data.statistics);
  console.log("โพสต์ล่าสุด:", data.recentPosts);
});
```

```javascript
// แบบฝึกหัดที่ 3: Real-time polling
class PollingService {
  constructor(url, interval = 5000) {
    this.url = url;
    this.interval = interval;
    this.isRunning = false;
    this.lastData = null;
    this.listeners = new Set();
    this.timer = null;
  }
  
  subscribe(listener) {
    this.listeners.add(listener);
    return () => this.listeners.delete(listener); // unsubscribe
  }
  
  notify(data, changed) {
    this.listeners.forEach(listener => listener(data, changed));
  }
  
  async poll() {
    try {
      const response = await fetch(this.url);
      const data = await response.json();
      
      const changed = JSON.stringify(data) !== JSON.stringify(this.lastData);
      this.lastData = data;
      
      if (changed) {
        this.notify(data, true);
      }
    } catch (err) {
      console.error("Polling error:", err.message);
    }
  }
  
  start() {
    if (this.isRunning) return;
    this.isRunning = true;
    
    this.poll(); // เรียกทันที
    this.timer = setInterval(() => this.poll(), this.interval);
    
    console.log(`เริ่ม polling ทุก ${this.interval}ms`);
  }
  
  stop() {
    if (!this.isRunning) return;
    this.isRunning = false;
    clearInterval(this.timer);
    this.timer = null;
    console.log("หยุด polling");
  }
}

// ใช้งาน
const poller = new PollingService(
  "https://jsonplaceholder.typicode.com/posts/1",
  10000
);

const unsubscribe = poller.subscribe((data, changed) => {
  if (changed) {
    console.log("ข้อมูลเปลี่ยนแปลง:", data.title);
  }
});

poller.start();

// หยุดหลัง 30 วินาที
setTimeout(() => {
  poller.stop();
  unsubscribe();
}, 30000);
```

---

## สรุป Part 28: Fetch API

| เมธอด/คุณสมบัติ | คำอธิบาย |
|----------------|-----------|
| `fetch(url, options)` | ส่ง HTTP request |
| `response.ok` | `true` ถ้า status 200-299 |
| `response.status` | HTTP status code |
| `response.json()` | แปลง body เป็น JSON |
| `response.text()` | แปลง body เป็น string |
| `response.blob()` | แปลง body เป็น Blob |
| `response.headers.get()` | อ่าน response header |
| `AbortController` | ยกเลิก fetch request |
| `FormData` | ส่งข้อมูล form/ไฟล์ |
| `URLSearchParams` | สร้าง query string |

---

*ต่อไป: Part 29 - Array Methods ขั้นสูง*
