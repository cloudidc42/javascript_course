# Part 67: Performance Optimization (Steps 1311-1330)

## บทนำ: ทำไม Performance ถึงสำคัญ

- เว็บที่โหลด 1 วินาที → Conversion Rate เพิ่ม 7%
- 53% ของ mobile users ออกจากเว็บถ้าโหลดช้ากว่า 3 วินาที
- Google ใช้ Page Speed เป็น ranking factor

```
Performance = Good UX = Better Business Results
```

---

## Step 1311: วัด Performance - performance.now()

```javascript
// วัดเวลา execution
const start = performance.now();

// โค้ดที่ต้องการวัด
for (let i = 0; i < 1000000; i++) {
  Math.sqrt(i);
}

const end = performance.now();
console.log(`ใช้เวลา: ${end - start} ms`);

// ตัวอย่างจริง: วัดเวลา fetch
async function fetchWithTiming(url) {
  const start = performance.now();
  
  try {
    const response = await fetch(url);
    const data = await response.json();
    
    const end = performance.now();
    console.log(`Fetch ใช้เวลา: ${(end - start).toFixed(2)} ms`);
    
    return data;
  } catch (error) {
    const end = performance.now();
    console.error(`Fetch failed ใน ${(end - start).toFixed(2)} ms:`, error);
    throw error;
  }
}

// เปรียบเทียบ algorithms
function bubbleSort(arr) {
  const start = performance.now();
  // ... sort logic
  const end = performance.now();
  return { sorted: arr, time: end - start };
}

function quickSort(arr) {
  const start = performance.now();
  // ... sort logic
  const end = performance.now();
  return { sorted: arr, time: end - start };
}

const largeArray = Array.from({ length: 10000 }, () => Math.random());
const bubble = bubbleSort([...largeArray]);
const quick = quickSort([...largeArray]);

console.log(`Bubble Sort: ${bubble.time.toFixed(2)} ms`);
console.log(`Quick Sort: ${quick.time.toFixed(2)} ms`);
```

---

## Step 1312: Performance Marks และ Measures

```javascript
// Performance Timeline API
// เพิ่ม marks
performance.mark('app-start');

// โหลด components
loadHeader();
performance.mark('header-loaded');

loadMain();
performance.mark('main-loaded');

loadFooter();
performance.mark('footer-loaded');

// วัดระหว่าง marks
performance.measure('header-time', 'app-start', 'header-loaded');
performance.measure('main-time', 'header-loaded', 'main-loaded');
performance.measure('footer-time', 'main-loaded', 'footer-loaded');
performance.measure('total-time', 'app-start', 'footer-loaded');

// ดูผลลัพธ์
const measures = performance.getEntriesByType('measure');
measures.forEach(measure => {
  console.log(`${measure.name}: ${measure.duration.toFixed(2)} ms`);
});

// ล้าง marks
performance.clearMarks();
performance.clearMeasures();

// ===== User Timing API =====

class PerformanceTracker {
  start(name) {
    performance.mark(`${name}-start`);
  }
  
  end(name) {
    performance.mark(`${name}-end`);
    performance.measure(name, `${name}-start`, `${name}-end`);
    
    const entries = performance.getEntriesByName(name, 'measure');
    const duration = entries[entries.length - 1].duration;
    
    // ส่งไป analytics
    this.reportToAnalytics(name, duration);
    
    return duration;
  }
  
  reportToAnalytics(name, duration) {
    // ส่งไป Google Analytics, Datadog, etc.
    if (window.gtag) {
      window.gtag('event', 'timing_complete', {
        name,
        value: Math.round(duration),
        event_category: 'Performance',
      });
    }
  }
}

const tracker = new PerformanceTracker();

async function loadDashboard() {
  tracker.start('dashboard-load');
  
  const [users, products, orders] = await Promise.all([
    fetchUsers(),
    fetchProducts(),
    fetchOrders(),
  ]);
  
  renderDashboard({ users, products, orders });
  
  const duration = tracker.end('dashboard-load');
  console.log(`Dashboard loaded ใน ${duration.toFixed(0)} ms`);
}
```

---

## Step 1313: Core Web Vitals

### Largest Contentful Paint (LCP)

```javascript
// วัด LCP
const observer = new PerformanceObserver((entryList) => {
  const entries = entryList.getEntries();
  const lastEntry = entries[entries.length - 1];
  
  console.log('LCP:', lastEntry.startTime, 'ms');
  console.log('LCP Element:', lastEntry.element);
  
  // ค่าที่ดี: < 2.5 วินาที
  // ค่าที่ต้องปรับปรุง: 2.5 - 4 วินาที
  // ค่าแย่: > 4 วินาที
});

observer.observe({ entryTypes: ['largest-contentful-paint'] });

// ปรับปรุง LCP
// 1. Preload สำหรับรูปภาพหลัก
// <link rel="preload" href="hero.jpg" as="image">

// 2. ใช้ CDN
// 3. Optimize รูปภาพ
// 4. Remove render-blocking resources
```

### First Input Delay (FID) / Interaction to Next Paint (INP)

```javascript
// วัด FID
const observer = new PerformanceObserver((entryList) => {
  for (const entry of entryList.getEntries()) {
    const delay = entry.processingStart - entry.startTime;
    console.log('FID:', delay, 'ms');
    console.log('Input type:', entry.name);
  }
});

observer.observe({ entryTypes: ['first-input'] });

// วัด INP (ทุก interactions)
const inpObserver = new PerformanceObserver((entryList) => {
  for (const entry of entryList.getEntries()) {
    if (entry.duration > 200) {
      console.warn('Slow interaction:', entry.name, entry.duration, 'ms');
    }
  }
});

inpObserver.observe({ entryTypes: ['event'], buffered: true });

// ปรับปรุง INP
// - ลด JavaScript bundle size
// - ใช้ code splitting
// - Defer non-critical JS
```

### Cumulative Layout Shift (CLS)

```javascript
// วัด CLS
let clsValue = 0;
let clsEntries = [];

const observer = new PerformanceObserver((entryList) => {
  for (const entry of entryList.getEntries()) {
    // ไม่นับ shift ที่เกิดจาก user interaction
    if (!entry.hadRecentInput) {
      clsValue += entry.value;
      clsEntries.push(entry);
    }
  }
  
  console.log('Current CLS:', clsValue.toFixed(4));
  // ค่าที่ดี: < 0.1
});

observer.observe({ entryTypes: ['layout-shift'], buffered: true });

// วิธีลด CLS:
// 1. กำหนด width/height ให้รูปภาพ
// <img src="photo.jpg" width="400" height="300" alt="">

// 2. กำหนด space สำหรับ dynamic content
// .ad-container { min-height: 250px; }

// 3. ใช้ CSS transform แทน top/left

// 4. Avoid inserting content above existing content
```

### ใช้ web-vitals library

```javascript
// npm install web-vitals
import { getCLS, getFID, getFCP, getLCP, getTTFB, getINP } from 'web-vitals';

function sendToAnalytics({ name, delta, value, id }) {
  // ส่งไป analytics service
  console.log({ name, delta, value, id });
  
  // Google Analytics 4
  gtag('event', name, {
    event_category: 'Web Vitals',
    event_label: id,
    value: Math.round(name === 'CLS' ? delta * 1000 : delta),
    non_interaction: true,
  });
}

getCLS(sendToAnalytics);
getFID(sendToAnalytics);
getFCP(sendToAnalytics);
getLCP(sendToAnalytics);
getTTFB(sendToAnalytics);
getINP(sendToAnalytics);
```

---

## Step 1314: DOM Manipulation Performance

```javascript
// ❌ แย่: อ่านและเขียน DOM สลับกัน (Layout Thrashing)
function badExample() {
  const elements = document.querySelectorAll('.item');
  
  elements.forEach(el => {
    const height = el.offsetHeight;  // อ่าน (cause reflow)
    el.style.height = height + 10 + 'px';  // เขียน (invalidate layout)
    // Loop รอบต่อไป: อ่านอีกครั้ง → reflow อีกครั้ง!
  });
}

// ✅ ดี: อ่านทั้งหมดก่อน แล้วค่อยเขียน
function goodExample() {
  const elements = document.querySelectorAll('.item');
  
  // Phase 1: อ่านทั้งหมด
  const heights = [...elements].map(el => el.offsetHeight);
  
  // Phase 2: เขียนทั้งหมด
  elements.forEach((el, i) => {
    el.style.height = heights[i] + 10 + 'px';
  });
}

// ✅ ดีกว่า: ใช้ requestAnimationFrame
function optimizedExample() {
  const elements = document.querySelectorAll('.item');
  
  // อ่านใน requestAnimationFrame
  requestAnimationFrame(() => {
    const heights = [...elements].map(el => el.offsetHeight);
    
    // เขียนใน requestAnimationFrame ถัดไป
    requestAnimationFrame(() => {
      elements.forEach((el, i) => {
        el.style.height = heights[i] + 10 + 'px';
      });
    });
  });
}

// ===== DocumentFragment สำหรับ batch DOM updates =====

// ❌ แย่: เพิ่ม DOM elements ทีละตัว
function addItemsSlow(items) {
  const list = document.getElementById('list');
  items.forEach(item => {
    const li = document.createElement('li');
    li.textContent = item;
    list.appendChild(li); // DOM update ทุกครั้ง!
  });
}

// ✅ ดี: ใช้ DocumentFragment
function addItemsFast(items) {
  const list = document.getElementById('list');
  const fragment = document.createDocumentFragment();
  
  items.forEach(item => {
    const li = document.createElement('li');
    li.textContent = item;
    fragment.appendChild(li); // ไม่ trigger DOM update
  });
  
  list.appendChild(fragment); // DOM update ครั้งเดียว!
}

// ===== innerHTML vs DOM API =====

// ✅ innerHTML เร็วสำหรับ large HTML strings
function renderList(items) {
  const list = document.getElementById('list');
  list.innerHTML = items.map(item => `<li>${item.name}</li>`).join('');
}

// ⚠️ แต่ระวัง XSS! ใช้กับ trusted data เท่านั้น
// สำหรับ user input ต้องใช้ textContent หรือ DOM API
function renderUserContent(userInput) {
  const div = document.createElement('div');
  div.textContent = userInput; // ปลอดภัยจาก XSS
  document.body.appendChild(div);
}
```

---

## Step 1315: Reflow และ Repaint

```javascript
// Properties ที่ทำให้เกิด Reflow (ช้า)
// Layout properties: width, height, top, left, margin, padding, border
// Reading: offsetWidth, offsetHeight, clientWidth, getBoundingClientRect()

// Properties ที่ทำให้เกิดเพียง Repaint (เร็วกว่า)
// color, background, box-shadow, outline, visibility

// Properties ที่ไม่ทำให้เกิด Reflow หรือ Repaint (เร็วที่สุด)
// transform, opacity → ใช้ GPU compositing

// ===== CSS Containment =====
// บอก browser ว่า element นี้ independent

/*
.widget {
  contain: layout;         /* layout changes ไม่กระทบ outside
  contain: style;          /* style changes ไม่กระทบ outside
  contain: paint;          /* repaint ไม่กระทบ outside
  contain: size;           /* size ไม่ขึ้นกับ children
  contain: strict;         /* all of above
}
*/

// ===== will-change =====
// แจ้งบราวเซอร์ล่วงหน้าว่า property จะเปลี่ยน

// ❌ อย่าใช้กับทุก element (เปลือง memory)
// * { will-change: transform; }

// ✅ ใช้เฉพาะกับ elements ที่จะ animate จริงๆ
function prepareAnimation(element) {
  element.style.willChange = 'transform, opacity';
  
  // หลัง animate เสร็จ ลบออก
  element.addEventListener('animationend', () => {
    element.style.willChange = 'auto';
  }, { once: true });
}

// ===== Transform แทน Position =====

// ❌ แย่: เปลี่ยน left/top ทำให้ reflow
function animateSlow(element) {
  let pos = 0;
  setInterval(() => {
    pos += 1;
    element.style.left = pos + 'px'; // Reflow!
  }, 16);
}

// ✅ ดี: ใช้ transform (GPU compositing, ไม่ reflow)
function animateFast(element) {
  let pos = 0;
  function step() {
    pos += 1;
    element.style.transform = `translateX(${pos}px)`; // Composite only!
    requestAnimationFrame(step);
  }
  requestAnimationFrame(step);
}
```

---

## Step 1316: Lazy Loading

```javascript
// ===== Lazy Loading Images =====

// วิธีที่ 1: Native lazy loading (แนะนำ)
// <img src="image.jpg" loading="lazy" alt="...">

// วิธีที่ 2: Intersection Observer
class LazyImageLoader {
  constructor(selector = 'img[data-src]') {
    this.observer = new IntersectionObserver(
      this.handleIntersection.bind(this),
      {
        rootMargin: '200px 0px', // เริ่มโหลดก่อน 200px
        threshold: 0.01,
      }
    );
    
    this.init(selector);
  }
  
  init(selector) {
    const images = document.querySelectorAll(selector);
    images.forEach(img => this.observer.observe(img));
  }
  
  handleIntersection(entries) {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        this.loadImage(entry.target);
        this.observer.unobserve(entry.target);
      }
    });
  }
  
  loadImage(img) {
    const src = img.dataset.src;
    if (!src) return;
    
    // Placeholder blur effect
    img.style.filter = 'blur(5px)';
    img.style.transition = 'filter 0.3s';
    
    const tempImg = new Image();
    tempImg.onload = () => {
      img.src = src;
      img.style.filter = 'none';
      img.removeAttribute('data-src');
    };
    tempImg.src = src;
  }
}

// ใช้งาน
const loader = new LazyImageLoader();

// HTML:
// <img data-src="photo.jpg" src="placeholder.jpg" alt="Photo">

// ===== Lazy Loading Components =====

// React: React.lazy
import React, { lazy, Suspense } from 'react';

const HeavyChart = lazy(() => import('./HeavyChart'));
const VideoPlayer = lazy(() => import('./VideoPlayer'));

function App() {
  return (
    <div>
      <Suspense fallback={<div>กำลังโหลด...</div>}>
        <HeavyChart data={chartData} />
      </Suspense>
      
      <Suspense fallback={<div>กำลังโหลดวิดีโอ...</div>}>
        <VideoPlayer url={videoUrl} />
      </Suspense>
    </div>
  );
}

// Dynamic import
async function loadChart() {
  const { Chart } = await import('./Chart.js');
  return new Chart('#canvas', config);
}

// โหลดเมื่อ user hover
document.getElementById('chart-container').addEventListener('mouseenter', async () => {
  const { Chart } = await import('./Chart.js');
  new Chart('#canvas', config);
}, { once: true });
```

---

## Step 1317: Code Splitting

```javascript
// ===== Webpack/Vite Code Splitting =====

// วิธีที่ 1: Dynamic import
// แต่ละ chunk จะเป็นไฟล์แยก

// main.js
import('./homepage.js').then(module => {
  module.initHomepage();
});

// Route-based splitting
const routes = {
  '/': () => import('./pages/Home.js'),
  '/products': () => import('./pages/Products.js'),
  '/cart': () => import('./pages/Cart.js'),
  '/profile': () => import('./pages/Profile.js'),
};

async function navigate(path) {
  const loadPage = routes[path];
  if (!loadPage) return;
  
  const { default: Page } = await loadPage();
  new Page().render();
}

// ===== Vite Configuration =====
// vite.config.js
import { defineConfig } from 'vite';

export default defineConfig({
  build: {
    rollupOptions: {
      output: {
        // แยก vendor libraries เป็น chunk แยก
        manualChunks: {
          'vendor': ['react', 'react-dom'],
          'charts': ['recharts', 'd3'],
          'utils': ['lodash', 'date-fns'],
        },
        
        // หรือใช้ function
        manualChunks(id) {
          if (id.includes('node_modules')) {
            if (id.includes('react')) return 'react-vendor';
            if (id.includes('lodash')) return 'utils';
            return 'vendor';
          }
        },
      },
    },
    
    // กำหนดขนาด chunk warning
    chunkSizeWarningLimit: 500, // KB
  },
});

// ===== Preloading สำหรับ critical chunks =====
// <link rel="preload" href="/chunk-homepage.js" as="script">

// Prefetch สำหรับ likely next navigation
// <link rel="prefetch" href="/chunk-products.js">

// ใน JavaScript
function prefetchNextPage(route) {
  const link = document.createElement('link');
  link.rel = 'prefetch';
  link.href = routes[route];
  document.head.appendChild(link);
}

// Prefetch เมื่อ hover link
document.querySelectorAll('a').forEach(link => {
  link.addEventListener('mouseenter', () => {
    const route = new URL(link.href).pathname;
    prefetchNextPage(route);
  }, { once: true });
});
```

---

## Step 1318: Tree Shaking

```javascript
// ===== Tree Shaking คืออะไร =====
// กระบวนการลบ dead code (code ที่ไม่ได้ใช้) ออกจาก bundle

// ❌ นำเข้าทั้ง library (ไม่ tree-shakeable)
import _ from 'lodash';
const result = _.map([1, 2, 3], x => x * 2);
// Bundle ขนาด: ~70KB

// ✅ นำเข้าเฉพาะที่ต้องการ (tree-shakeable)
import map from 'lodash/map';
const result = map([1, 2, 3], x => x * 2);
// Bundle ขนาด: ~2KB

// ✅ ดีที่สุด: ใช้ ES modules ที่รองรับ tree shaking
import { map, filter } from 'lodash-es';

// ===== เขียน code ที่ tree-shakeable =====

// ❌ Side effects - ไม่ tree-shakeable
(function() {
  window.myLib = { ... };
})();

// ✅ Named exports - tree-shakeable
export function formatDate(date) { ... }
export function formatCurrency(amount) { ... }
export function formatPhone(phone) { ... }

// ผู้ใช้นำเข้าเฉพาะที่ต้องการ
import { formatDate } from './utils';
// formatCurrency และ formatPhone จะถูก tree-shaken ออก

// ===== package.json sideEffects =====
// บอก bundler ว่าไฟล์ไหนมี side effects
{
  "sideEffects": false  // ทุกไฟล์ไม่มี side effects
}

// หรือระบุไฟล์ที่มี side effects
{
  "sideEffects": [
    "*.css",
    "./src/polyfills.js"
  ]
}

// ===== Analyze Bundle =====

// webpack-bundle-analyzer
// npm install --save-dev webpack-bundle-analyzer

// vite-bundle-visualizer
// npm install --save-dev vite-bundle-visualizer

// vite.config.js
import { visualizer } from 'rollup-plugin-visualizer';

export default defineConfig({
  plugins: [
    visualizer({
      filename: 'stats.html',
      open: true,
      gzipSize: true,
      brotliSize: true,
    }),
  ],
});
```

---

## Step 1319: Debounce และ Throttle

```javascript
// ===== Debounce =====
// รันหลังจาก event หยุดแล้ว N ms

function debounce(fn, delay) {
  let timer = null;
  
  return function(...args) {
    clearTimeout(timer);
    timer = setTimeout(() => {
      fn.apply(this, args);
      timer = null;
    }, delay);
  };
}

// ตัวอย่าง: Search input
const searchInput = document.getElementById('search');
const search = debounce(async (query) => {
  const results = await fetch(`/api/search?q=${query}`).then(r => r.json());
  renderResults(results);
}, 300);

searchInput.addEventListener('input', (e) => {
  search(e.target.value);
});

// ===== Throttle =====
// รันอย่างน้อยทุก N ms

function throttle(fn, limit) {
  let inThrottle = false;
  
  return function(...args) {
    if (!inThrottle) {
      fn.apply(this, args);
      inThrottle = true;
      setTimeout(() => {
        inThrottle = false;
      }, limit);
    }
  };
}

// ตัวอย่าง: Scroll event
const handleScroll = throttle(() => {
  const scrollTop = window.pageYOffset;
  updateProgressBar(scrollTop);
  toggleStickyHeader(scrollTop);
}, 16); // ~60fps

window.addEventListener('scroll', handleScroll);

// ===== Debounce ด้วย leading edge =====
function debounceLeading(fn, delay) {
  let timer = null;
  
  return function(...args) {
    if (!timer) {
      fn.apply(this, args); // รันทันที
    }
    clearTimeout(timer);
    timer = setTimeout(() => {
      timer = null;
    }, delay);
  };
}

// ===== Advanced Debounce =====
function advancedDebounce(fn, delay, options = {}) {
  const { leading = false, trailing = true, maxWait } = options;
  let timer = null;
  let lastCallTime = 0;
  let lastInvokeTime = 0;
  
  return function(...args) {
    const now = Date.now();
    const timeSinceLastCall = now - lastCallTime;
    const timeSinceLastInvoke = now - lastInvokeTime;
    
    lastCallTime = now;
    
    // Leading edge
    if (leading && !timer) {
      fn.apply(this, args);
      lastInvokeTime = now;
    }
    
    clearTimeout(timer);
    
    // maxWait check
    if (maxWait && timeSinceLastInvoke >= maxWait) {
      fn.apply(this, args);
      lastInvokeTime = now;
      return;
    }
    
    // Trailing edge
    if (trailing) {
      timer = setTimeout(() => {
        fn.apply(this, args);
        lastInvokeTime = Date.now();
        timer = null;
      }, delay);
    }
  };
}

// ใช้ lodash debounce/throttle ในงานจริง
// import { debounce, throttle } from 'lodash-es';
```

---

## Step 1320: Event Delegation

```javascript
// ===== Event Delegation =====
// แทนที่จะ attach event ให้ทุก element
// ให้ attach ที่ parent element แทน

// ❌ แย่: event listener ทุก element (เปลือง memory)
function addEventsBad() {
  const buttons = document.querySelectorAll('.product-card .add-to-cart');
  buttons.forEach(btn => {
    btn.addEventListener('click', (e) => {
      const productId = e.target.closest('.product-card').dataset.id;
      addToCart(productId);
    });
  });
}

// ✅ ดี: event delegation
function addEventsGood() {
  const productList = document.getElementById('product-list');
  
  productList.addEventListener('click', (e) => {
    // ตรวจสอบว่า click ที่ปุ่ม add-to-cart
    const addToCartBtn = e.target.closest('.add-to-cart');
    if (!addToCartBtn) return;
    
    const productCard = e.target.closest('.product-card');
    const productId = productCard?.dataset.id;
    
    if (productId) {
      addToCart(productId);
    }
  });
  
  // ยังจัดการ events อื่นได้ในที่เดียว
  productList.addEventListener('click', (e) => {
    const wishlistBtn = e.target.closest('.wishlist-btn');
    if (wishlistBtn) {
      const productId = wishlistBtn.closest('.product-card').dataset.id;
      addToWishlist(productId);
    }
    
    const viewBtn = e.target.closest('.view-details');
    if (viewBtn) {
      const productId = viewBtn.closest('.product-card').dataset.id;
      viewProductDetails(productId);
    }
  });
}

// ===== Event Delegation สำหรับ Dynamic Content =====
// event delegation ทำงานกับ elements ที่เพิ่มมาทีหลังด้วย!

document.getElementById('table-body').addEventListener('click', (e) => {
  const row = e.target.closest('tr');
  if (!row) return;
  
  const action = e.target.dataset.action;
  const id = row.dataset.id;
  
  switch (action) {
    case 'edit':
      editItem(id);
      break;
    case 'delete':
      deleteItem(id);
      break;
    case 'view':
      viewItem(id);
      break;
  }
});

// เพิ่ม row ใหม่ได้เลย โดยไม่ต้อง attach event อีก
function addNewRow(item) {
  const tbody = document.getElementById('table-body');
  const tr = document.createElement('tr');
  tr.dataset.id = item.id;
  tr.innerHTML = `
    <td>${item.name}</td>
    <td>${item.price}</td>
    <td>
      <button data-action="edit">แก้ไข</button>
      <button data-action="delete">ลบ</button>
      <button data-action="view">ดูรายละเอียด</button>
    </td>
  `;
  tbody.appendChild(tr);
  // event จะทำงานได้ทันทีโดยไม่ต้อง attach!
}
```

---

## Step 1321: requestAnimationFrame Optimization

```javascript
// ===== requestAnimationFrame (rAF) =====
// รัน callback ก่อนที่ browser จะ render frame ถัดไป
// ทำให้ animation smooth ที่ 60fps

// ❌ แย่: ใช้ setInterval (ไม่ sync กับ refresh rate)
function animateBad(element) {
  let x = 0;
  setInterval(() => {
    x += 5;
    element.style.transform = `translateX(${x}px)`;
  }, 16); // อาจไม่ตรงกับ refresh rate
}

// ✅ ดี: ใช้ requestAnimationFrame
function animateGood(element) {
  let x = 0;
  let animationId;
  
  function step(timestamp) {
    x += 5;
    element.style.transform = `translateX(${x}px)`;
    
    if (x < 300) {
      animationId = requestAnimationFrame(step);
    }
  }
  
  animationId = requestAnimationFrame(step);
  
  // หยุด animation
  // cancelAnimationFrame(animationId);
}

// ===== Smooth Animation ด้วย easing =====
function smoothAnimate(element, targetX, duration = 1000) {
  const startX = parseFloat(element.dataset.x || 0);
  const distance = targetX - startX;
  let startTime = null;
  
  // Easing function (ease-in-out)
  function easeInOut(t) {
    return t < 0.5
      ? 2 * t * t
      : -1 + (4 - 2 * t) * t;
  }
  
  function step(currentTime) {
    if (!startTime) startTime = currentTime;
    
    const elapsed = currentTime - startTime;
    const progress = Math.min(elapsed / duration, 1);
    const eased = easeInOut(progress);
    
    const currentX = startX + distance * eased;
    element.style.transform = `translateX(${currentX}px)`;
    element.dataset.x = currentX;
    
    if (progress < 1) {
      requestAnimationFrame(step);
    }
  }
  
  requestAnimationFrame(step);
}

// ===== FPS Monitor =====
class FPSMonitor {
  constructor() {
    this.fps = 0;
    this.frames = 0;
    this.lastTime = performance.now();
    this.active = false;
  }
  
  start() {
    this.active = true;
    this.tick();
  }
  
  stop() {
    this.active = false;
  }
  
  tick() {
    if (!this.active) return;
    
    this.frames++;
    const now = performance.now();
    const elapsed = now - this.lastTime;
    
    if (elapsed >= 1000) {
      this.fps = Math.round(this.frames * 1000 / elapsed);
      this.frames = 0;
      this.lastTime = now;
      
      // แสดงใน UI
      document.getElementById('fps-counter').textContent = `FPS: ${this.fps}`;
      
      // Warning ถ้า FPS ต่ำ
      if (this.fps < 30) {
        console.warn(`Low FPS: ${this.fps}`);
      }
    }
    
    requestAnimationFrame(() => this.tick());
  }
}

const monitor = new FPSMonitor();
monitor.start();

// ===== Batch DOM updates ด้วย rAF =====
class BatchDOMUpdater {
  constructor() {
    this.pendingUpdates = [];
    this.scheduled = false;
  }
  
  schedule(updateFn) {
    this.pendingUpdates.push(updateFn);
    
    if (!this.scheduled) {
      this.scheduled = true;
      requestAnimationFrame(() => this.flush());
    }
  }
  
  flush() {
    const updates = this.pendingUpdates.splice(0);
    updates.forEach(fn => fn());
    this.scheduled = false;
  }
}

const domUpdater = new BatchDOMUpdater();

// หลาย components ขอ update พร้อมกัน → รวม batch เดียว
function updateComponent1() {
  domUpdater.schedule(() => {
    document.getElementById('comp1').textContent = 'Updated';
  });
}

function updateComponent2() {
  domUpdater.schedule(() => {
    document.getElementById('comp2').style.opacity = '0.5';
  });
}
```

---

## Step 1322: Memory Leaks

```javascript
// ===== ประเภท Memory Leaks =====

// 1. Event Listeners ที่ไม่ได้ลบ
class ComponentWithLeak {
  constructor() {
    // ❌ Leak: handler reference ถูก bind ใหม่ทุกครั้ง
    window.addEventListener('resize', this.handleResize.bind(this));
  }
  
  // ไม่มีวิธีลบ listener เพราะ bind() สร้าง function ใหม่
}

class ComponentFixed {
  constructor() {
    // ✅ เก็บ reference
    this.boundResize = this.handleResize.bind(this);
    window.addEventListener('resize', this.boundResize);
  }
  
  handleResize() {
    console.log('resized');
  }
  
  destroy() {
    // ✅ ลบเมื่อ component ถูก destroy
    window.removeEventListener('resize', this.boundResize);
  }
}

// 2. Closures ที่ hold reference ไว้
function createLeak() {
  const largeData = new Array(1000000).fill('data');
  
  // ❌ largeData จะไม่ถูก garbage collected
  // เพราะ timer closure ยัง reference อยู่
  const timer = setInterval(() => {
    console.log(largeData[0]);
  }, 1000);
  
  // ✅ ต้อง clear timer เมื่อไม่ต้องการ
  return () => clearInterval(timer);
}

// 3. DOM References
function domLeak() {
  const container = document.getElementById('container');
  
  // ❌ เก็บ reference ไว้ใน object ที่ live นาน
  const cache = {};
  
  function cacheElement(id) {
    // cache[id] จะ hold DOM element ไว้แม้ element นั้นถูกลบจาก DOM
    cache[id] = document.getElementById(id);
  }
  
  // ✅ ใช้ WeakRef แทน
  const weakCache = new Map();
  
  function cacheElementWeak(id) {
    weakCache.set(id, new WeakRef(document.getElementById(id)));
  }
  
  function getElement(id) {
    return weakCache.get(id)?.deref();
  }
}

// 4. setTimeout/setInterval ที่ไม่ clear
function createTimerLeak() {
  let count = 0;
  
  // ❌ ถ้า component ถูก unmount แต่ timer ยังทำงาน
  setInterval(() => {
    count++;
    updateUI(count); // อาจ throw error ถ้า UI element ถูก remove
  }, 1000);
}

// ===== Detecting Memory Leaks =====

// ใช้ Chrome DevTools:
// 1. Performance tab → Record → ดู Memory graph
// 2. Memory tab → Take Heap Snapshot
// 3. Memory tab → Allocation instrumentation

// หรือใช้ JS
function detectMemoryLeak() {
  if (!performance.memory) return;
  
  const { usedJSHeapSize, totalJSHeapSize, jsHeapSizeLimit } = performance.memory;
  
  console.log({
    used: Math.round(usedJSHeapSize / 1024 / 1024) + ' MB',
    total: Math.round(totalJSHeapSize / 1024 / 1024) + ' MB',
    limit: Math.round(jsHeapSizeLimit / 1024 / 1024) + ' MB',
    usage: Math.round(usedJSHeapSize / jsHeapSizeLimit * 100) + '%',
  });
}

setInterval(detectMemoryLeak, 5000);
```

---

## Step 1323: WeakMap และ WeakRef

```javascript
// ===== WeakMap =====
// เหมือน Map แต่ keys เป็น objects และไม่ prevent GC

const domData = new WeakMap();

function attachData(element, data) {
  domData.set(element, data);
}

function getData(element) {
  return domData.get(element);
}

// เมื่อ element ถูก remove จาก DOM และไม่มี reference อื่น
// GC จะลบข้อมูลใน WeakMap อัตโนมัติ

// ตัวอย่าง: Cache component state
const componentState = new WeakMap();

class MyComponent {
  constructor(element) {
    this.element = element;
    componentState.set(element, {
      clicks: 0,
      data: null,
      initialized: false,
    });
  }
  
  click() {
    const state = componentState.get(this.element);
    state.clicks++;
    this.update();
  }
  
  update() {
    const state = componentState.get(this.element);
    this.element.querySelector('.count').textContent = state.clicks;
  }
  
  destroy() {
    // ไม่ต้อง cleanup WeakMap เอง!
    // เมื่อ element ถูก GC, WeakMap entry ก็จะถูก GC ด้วย
    this.element.remove();
  }
}

// ===== WeakRef =====
// Reference ที่ไม่ prevent GC

class Cache {
  constructor() {
    this.cache = new Map();
  }
  
  set(key, value) {
    this.cache.set(key, new WeakRef(value));
  }
  
  get(key) {
    const ref = this.cache.get(key);
    if (!ref) return undefined;
    
    const value = ref.deref();
    if (value === undefined) {
      // Object ถูก GC แล้ว → ลบ cache entry
      this.cache.delete(key);
      return undefined;
    }
    
    return value;
  }
}

// ===== FinalizationRegistry =====
// ทำงานเมื่อ object ถูก GC

const registry = new FinalizationRegistry((key) => {
  console.log(`Object "${key}" ถูก garbage collected`);
  // ทำ cleanup เพิ่มเติม
});

function createTrackedObject(name) {
  const obj = { name, data: new Array(1000).fill('data') };
  registry.register(obj, name);
  return obj;
}

let obj = createTrackedObject('my-object');
obj = null; // หลัง GC จะเห็น log
```

---

## Step 1324: Caching Strategies

```javascript
// ===== In-Memory Cache =====

class MemoryCache {
  constructor(maxSize = 100, ttl = 5 * 60 * 1000) { // 5 นาที
    this.cache = new Map();
    this.maxSize = maxSize;
    this.defaultTTL = ttl;
  }
  
  set(key, value, ttl = this.defaultTTL) {
    // LRU: ถ้าเต็ม ลบ entry เก่าสุด
    if (this.cache.size >= this.maxSize) {
      const firstKey = this.cache.keys().next().value;
      this.cache.delete(firstKey);
    }
    
    this.cache.set(key, {
      value,
      expires: Date.now() + ttl,
    });
  }
  
  get(key) {
    const entry = this.cache.get(key);
    if (!entry) return undefined;
    
    if (Date.now() > entry.expires) {
      this.cache.delete(key);
      return undefined;
    }
    
    // Move to end (LRU)
    this.cache.delete(key);
    this.cache.set(key, entry);
    
    return entry.value;
  }
  
  delete(key) {
    return this.cache.delete(key);
  }
  
  clear() {
    this.cache.clear();
  }
  
  has(key) {
    const entry = this.cache.get(key);
    if (!entry) return false;
    if (Date.now() > entry.expires) {
      this.cache.delete(key);
      return false;
    }
    return true;
  }
}

// ===== Memoization =====

function memoize(fn, keyFn = (...args) => JSON.stringify(args)) {
  const cache = new Map();
  
  return function(...args) {
    const key = keyFn(...args);
    
    if (cache.has(key)) {
      return cache.get(key);
    }
    
    const result = fn.apply(this, args);
    cache.set(key, result);
    return result;
  };
}

// ตัวอย่าง: Fibonacci ที่เร็วขึ้นมาก
const fib = memoize(function(n) {
  if (n <= 1) return n;
  return fib(n - 1) + fib(n - 2);
});

console.log(fib(50)); // เร็วมาก เพราะ cache

// ===== React useMemo/useCallback =====
import { useMemo, useCallback } from 'react';

function ProductList({ products, filters }) {
  // useMemo: คำนวณ derived data
  const filteredProducts = useMemo(() => {
    return products.filter(p => {
      if (filters.minPrice && p.price < filters.minPrice) return false;
      if (filters.maxPrice && p.price > filters.maxPrice) return false;
      if (filters.category && p.category !== filters.category) return false;
      return true;
    });
  }, [products, filters]); // คำนวณใหม่เมื่อ products หรือ filters เปลี่ยน
  
  // useCallback: cache function reference
  const handleAddToCart = useCallback((productId) => {
    addToCart(productId);
  }, []); // ไม่สร้าง function ใหม่ทุก render
  
  return (
    <ul>
      {filteredProducts.map(product => (
        <ProductCard
          key={product.id}
          product={product}
          onAddToCart={handleAddToCart}
        />
      ))}
    </ul>
  );
}
```

---

## Step 1325: Network Optimization

```javascript
// ===== HTTP/2 Server Push =====
// Server ส่ง resources ที่ client จะต้องใช้ล่วงหน้า
// (จัดการที่ server-side)

// ===== Resource Hints =====
// preconnect: เชื่อมต่อ server ล่วงหน้า
// <link rel="preconnect" href="https://api.example.com">

// dns-prefetch: resolve DNS ล่วงหน้า
// <link rel="dns-prefetch" href="https://cdn.example.com">

// preload: โหลด resource สำคัญล่วงหน้า
// <link rel="preload" href="/fonts/main.woff2" as="font" crossorigin>

// prefetch: โหลด resource ที่จะใช้ในอนาคต
// <link rel="prefetch" href="/next-page.js">

// ===== Request Batching =====

// ❌ แย่: หลาย requests แยกกัน
async function loadUserDataBad(userId) {
  const profile = await fetch(`/api/users/${userId}`);
  const orders = await fetch(`/api/users/${userId}/orders`);
  const wishlist = await fetch(`/api/users/${userId}/wishlist`);
  return { profile, orders, wishlist };
}

// ✅ ดีขึ้น: Parallel requests
async function loadUserDataGood(userId) {
  const [profile, orders, wishlist] = await Promise.all([
    fetch(`/api/users/${userId}`).then(r => r.json()),
    fetch(`/api/users/${userId}/orders`).then(r => r.json()),
    fetch(`/api/users/${userId}/wishlist`).then(r => r.json()),
  ]);
  return { profile, orders, wishlist };
}

// ✅ ดีที่สุด: Single request (GraphQL / Custom batch endpoint)
async function loadUserDataBest(userId) {
  const response = await fetch('/api/batch', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      requests: [
        { id: 'profile', url: `/api/users/${userId}` },
        { id: 'orders', url: `/api/users/${userId}/orders` },
        { id: 'wishlist', url: `/api/users/${userId}/wishlist` },
      ],
    }),
  });
  
  const data = await response.json();
  return {
    profile: data.profile,
    orders: data.orders,
    wishlist: data.wishlist,
  };
}

// ===== Service Worker Caching =====
// sw.js

const CACHE_NAME = 'app-v1';
const STATIC_ASSETS = [
  '/',
  '/styles/main.css',
  '/scripts/app.js',
  '/images/logo.png',
];

// Install: cache static assets
self.addEventListener('install', (event) => {
  event.waitUntil(
    caches.open(CACHE_NAME).then((cache) => {
      return cache.addAll(STATIC_ASSETS);
    })
  );
  self.skipWaiting();
});

// Fetch: serve from cache
self.addEventListener('fetch', (event) => {
  const { request } = event;
  const url = new URL(request.url);
  
  // Cache First: static assets
  if (STATIC_ASSETS.includes(url.pathname)) {
    event.respondWith(
      caches.match(request).then(cached => {
        return cached || fetch(request);
      })
    );
    return;
  }
  
  // Network First: API calls
  if (url.pathname.startsWith('/api/')) {
    event.respondWith(
      fetch(request)
        .then(response => {
          const clone = response.clone();
          caches.open(CACHE_NAME).then(cache => {
            cache.put(request, clone);
          });
          return response;
        })
        .catch(() => {
          return caches.match(request);
        })
    );
    return;
  }
});
```

---

## Step 1326: Web Workers

```javascript
// ===== Web Workers =====
// รันโค้ดใน background thread
// ไม่ block UI thread

// worker.js - runs ใน background
self.addEventListener('message', (event) => {
  const { type, data } = event.data;
  
  switch (type) {
    case 'SORT':
      const sorted = heavySort(data.array);
      self.postMessage({ type: 'SORT_DONE', result: sorted });
      break;
      
    case 'CALCULATE':
      const result = heavyCalculation(data.n);
      self.postMessage({ type: 'CALC_DONE', result });
      break;
      
    case 'PROCESS_IMAGE':
      const processed = processImageData(data.imageData);
      self.postMessage({ type: 'IMAGE_DONE', result: processed }, [processed.buffer]);
      break;
  }
});

function heavySort(array) {
  // การเรียง array ขนาดใหญ่
  return array.sort((a, b) => a - b);
}

function heavyCalculation(n) {
  // การคำนวณหนัก เช่น prime numbers
  const primes = [];
  for (let i = 2; i <= n; i++) {
    if (isPrime(i)) primes.push(i);
  }
  return primes;
}

function isPrime(n) {
  for (let i = 2; i <= Math.sqrt(n); i++) {
    if (n % i === 0) return false;
  }
  return true;
}

// main.js - ใช้ Worker
class WorkerPool {
  constructor(workerScript, poolSize = navigator.hardwareConcurrency || 4) {
    this.workers = [];
    this.queue = [];
    this.activeJobs = new Map();
    this.jobId = 0;
    
    // สร้าง worker pool
    for (let i = 0; i < poolSize; i++) {
      const worker = new Worker(workerScript);
      worker.onmessage = this.handleMessage.bind(this, worker);
      worker.idle = true;
      this.workers.push(worker);
    }
  }
  
  handleMessage(worker, event) {
    const { id, result, error } = event.data;
    const { resolve, reject } = this.activeJobs.get(id);
    
    this.activeJobs.delete(id);
    worker.idle = true;
    
    if (error) {
      reject(new Error(error));
    } else {
      resolve(result);
    }
    
    // ทำงาน job ถัดไปใน queue
    this.processQueue();
  }
  
  getIdleWorker() {
    return this.workers.find(w => w.idle);
  }
  
  processQueue() {
    if (this.queue.length === 0) return;
    const worker = this.getIdleWorker();
    if (!worker) return;
    
    const { id, message } = this.queue.shift();
    worker.idle = false;
    worker.postMessage({ ...message, id });
  }
  
  run(message) {
    return new Promise((resolve, reject) => {
      const id = this.jobId++;
      this.activeJobs.set(id, { resolve, reject });
      
      const worker = this.getIdleWorker();
      if (worker) {
        worker.idle = false;
        worker.postMessage({ ...message, id });
      } else {
        this.queue.push({ id, message });
      }
    });
  }
  
  terminate() {
    this.workers.forEach(w => w.terminate());
  }
}

// ใช้งาน
const pool = new WorkerPool('./worker.js', 4);

async function processLargeDataset(data) {
  console.log('เริ่มประมวลผล...');
  
  // แบ่งงานให้ workers
  const chunkSize = Math.ceil(data.length / 4);
  const chunks = [];
  for (let i = 0; i < data.length; i += chunkSize) {
    chunks.push(data.slice(i, i + chunkSize));
  }
  
  // รัน parallel ใน workers
  const results = await Promise.all(
    chunks.map(chunk => pool.run({ type: 'SORT', data: { array: chunk } }))
  );
  
  // Merge results
  return results.flat().sort((a, b) => a - b);
}
```

---

## Step 1327: WASM สำหรับ Heavy Computation

```javascript
// WebAssembly (WASM) - ใช้สำหรับงานหนัก
// เขียนด้วย C/C++/Rust แล้ว compile เป็น WASM

// ตัวอย่าง: โหลดและใช้ WASM module
async function loadWasm(wasmUrl) {
  const response = await fetch(wasmUrl);
  const buffer = await response.arrayBuffer();
  const { instance } = await WebAssembly.instantiate(buffer, {
    env: {
      memory: new WebAssembly.Memory({ initial: 256 }),
      // ฟังก์ชันที่ WASM เรียกได้
      consoleLog: (value) => console.log(value),
    },
  });
  
  return instance.exports;
}

// ใช้งาน
async function runWasm() {
  const wasm = await loadWasm('/math.wasm');
  
  // เรียกฟังก์ชันจาก WASM
  const result = wasm.fibonacci(40);
  console.log('Fibonacci(40):', result);
  
  // WASM เร็วกว่า JS สำหรับ CPU-intensive tasks
  console.time('wasm-fib');
  wasm.fibonacci(45);
  console.timeEnd('wasm-fib');
  
  console.time('js-fib');
  jsFibonacci(45);
  console.timeEnd('js-fib');
}

// ===== ใช้ AssemblyScript =====
// ภาษาคล้าย TypeScript ที่ compile เป็น WASM

// assembly/index.ts (AssemblyScript)
export function add(a: i32, b: i32): i32 {
  return a + b;
}

export function fibonacci(n: i32): i64 {
  if (n <= 1) return n;
  let a: i64 = 0;
  let b: i64 = 1;
  for (let i: i32 = 2; i <= n; i++) {
    let temp = a + b;
    a = b;
    b = temp;
  }
  return b;
}
```

---

## Step 1328: Virtual Scrolling

```javascript
// Virtual Scrolling - render เฉพาะ visible items

class VirtualList {
  constructor(container, items, itemHeight = 50) {
    this.container = container;
    this.items = items;
    this.itemHeight = itemHeight;
    this.visibleCount = Math.ceil(container.clientHeight / itemHeight) + 2;
    this.startIndex = 0;
    
    this.init();
  }
  
  init() {
    // Container ต้องมี overflow: auto/scroll
    this.container.style.overflow = 'auto';
    this.container.style.position = 'relative';
    
    // Phantom element สำหรับ scroll height
    this.phantom = document.createElement('div');
    this.phantom.style.height = `${this.items.length * this.itemHeight}px`;
    this.phantom.style.pointerEvents = 'none';
    this.container.appendChild(this.phantom);
    
    // Visible area
    this.visibleArea = document.createElement('div');
    this.visibleArea.style.position = 'absolute';
    this.visibleArea.style.top = '0';
    this.visibleArea.style.left = '0';
    this.visibleArea.style.right = '0';
    this.container.appendChild(this.visibleArea);
    
    // Scroll listener
    this.container.addEventListener('scroll', 
      throttle(() => this.render(), 16)
    );
    
    this.render();
  }
  
  render() {
    const scrollTop = this.container.scrollTop;
    this.startIndex = Math.floor(scrollTop / this.itemHeight);
    const endIndex = Math.min(
      this.startIndex + this.visibleCount,
      this.items.length
    );
    
    // Offset สำหรับ visible area
    const offsetY = this.startIndex * this.itemHeight;
    this.visibleArea.style.transform = `translateY(${offsetY}px)`;
    
    // Render visible items
    this.visibleArea.innerHTML = '';
    for (let i = this.startIndex; i < endIndex; i++) {
      const item = document.createElement('div');
      item.style.height = `${this.itemHeight}px`;
      item.style.display = 'flex';
      item.style.alignItems = 'center';
      item.style.padding = '0 16px';
      item.textContent = this.items[i];
      this.visibleArea.appendChild(item);
    }
  }
}

// ใช้งาน: render 100,000 items ได้อย่าง smooth
const items = Array.from({ length: 100000 }, (_, i) => `รายการที่ ${i + 1}`);
const list = new VirtualList(document.getElementById('list'), items, 50);
```

---

## Step 1329: Performance Monitoring

```javascript
// ===== Navigation Timing API =====

window.addEventListener('load', () => {
  const timing = performance.getEntriesByType('navigation')[0];
  
  const metrics = {
    // DNS lookup time
    dns: timing.domainLookupEnd - timing.domainLookupStart,
    
    // TCP connection time
    tcp: timing.connectEnd - timing.connectStart,
    
    // TLS handshake time
    tls: timing.secureConnectionStart > 0 
      ? timing.connectEnd - timing.secureConnectionStart 
      : 0,
    
    // Time to First Byte (TTFB)
    ttfb: timing.responseStart - timing.requestStart,
    
    // Download time
    download: timing.responseEnd - timing.responseStart,
    
    // DOM processing time
    domProcessing: timing.domComplete - timing.domInteractive,
    
    // Total page load time
    total: timing.loadEventEnd - timing.startTime,
  };
  
  console.table(metrics);
  
  // ส่งไป monitoring service
  sendMetrics(metrics);
});

// ===== Resource Timing =====
const resources = performance.getEntriesByType('resource');
resources.forEach(resource => {
  if (resource.duration > 1000) { // slow resource (> 1 second)
    console.warn('Slow resource:', resource.name, resource.duration.toFixed(0), 'ms');
  }
});

// ===== Long Tasks API =====
const observer = new PerformanceObserver((list) => {
  list.getEntries().forEach((entry) => {
    // Long task = > 50ms ที่ block main thread
    console.warn('Long task detected!', {
      duration: entry.duration.toFixed(0) + 'ms',
      start: entry.startTime.toFixed(0) + 'ms',
    });
  });
});

observer.observe({ entryTypes: ['longtask'] });

// ===== Custom Performance Dashboard =====
class PerformanceDashboard {
  constructor() {
    this.metrics = [];
    this.startMonitoring();
  }
  
  startMonitoring() {
    // LCP
    new PerformanceObserver(list => {
      const entry = list.getEntries().pop();
      this.record('LCP', entry.startTime);
    }).observe({ entryTypes: ['largest-contentful-paint'] });
    
    // CLS
    let cls = 0;
    new PerformanceObserver(list => {
      list.getEntries().forEach(entry => {
        if (!entry.hadRecentInput) cls += entry.value;
        this.record('CLS', cls);
      });
    }).observe({ entryTypes: ['layout-shift'] });
    
    // INP
    new PerformanceObserver(list => {
      list.getEntries().forEach(entry => {
        if (entry.duration > 40) {
          this.record('Slow Interaction', entry.duration);
        }
      });
    }).observe({ entryTypes: ['event'] });
    
    // Memory (every 30 seconds)
    setInterval(() => {
      if (performance.memory) {
        this.record('Memory', performance.memory.usedJSHeapSize / 1024 / 1024);
      }
    }, 30000);
  }
  
  record(name, value) {
    this.metrics.push({ name, value, time: Date.now() });
    this.render();
  }
  
  render() {
    const dashboard = document.getElementById('perf-dashboard');
    if (!dashboard) return;
    
    const latest = {};
    this.metrics.forEach(m => { latest[m.name] = m.value; });
    
    dashboard.innerHTML = Object.entries(latest)
      .map(([name, value]) => `
        <div class="metric">
          <span class="name">${name}</span>
          <span class="value">${typeof value === 'number' ? value.toFixed(2) : value}</span>
        </div>
      `)
      .join('');
  }
}

const dashboard = new PerformanceDashboard();
```

---

## Step 1330: Performance Best Practices Summary

```javascript
// ===== รวม Best Practices =====

// 1. Critical Path Optimization
/*
<head>
  <!-- Critical CSS inline -->
  <style>/* above-the-fold styles *\/</style>
  
  <!-- Preload critical resources -->
  <link rel="preload" href="font.woff2" as="font" crossorigin>
  <link rel="preload" href="hero.jpg" as="image">
  
  <!-- Defer non-critical JS -->
  <script src="app.js" defer></script>
  <script src="analytics.js" async></script>
</head>
*/

// 2. Image Optimization
// <img src="image.webp" loading="lazy" decoding="async"
//      width="800" height="600" alt="...">
// ใช้ WebP/AVIF format
// ใช้ srcset สำหรับ responsive images

// 3. Font Optimization
/*
@font-face {
  font-family: 'MyFont';
  src: url('font.woff2') format('woff2');
  font-display: swap; /* แสดง system font ก่อน แล้ว swap *\/
}
*/

// 4. Bundle Optimization
// - Code splitting
// - Tree shaking
// - Minification + compression (gzip/brotli)
// - Cache busting (hash in filename)

// 5. Runtime Performance
// - Debounce/throttle event handlers
// - Virtual scrolling สำหรับ large lists
// - Web Workers สำหรับ CPU-intensive tasks
// - Avoid layout thrashing
// - Use transform/opacity for animations

// ===== Performance Budget =====
const performanceBudget = {
  // Page weight
  totalPageSize: 1.5 * 1024 * 1024, // 1.5MB
  
  // JavaScript
  jsSize: 300 * 1024, // 300KB
  
  // Images
  imageSize: 800 * 1024, // 800KB
  
  // Core Web Vitals
  LCP: 2500,    // ms
  INP: 200,     // ms
  CLS: 0.1,
  
  // Loading
  TTFB: 800,    // ms
  TTI: 3800,    // ms (Time to Interactive)
};

function checkBudget(metrics) {
  const violations = [];
  
  if (metrics.LCP > performanceBudget.LCP) {
    violations.push(`LCP ${metrics.LCP}ms เกิน ${performanceBudget.LCP}ms`);
  }
  
  if (metrics.CLS > performanceBudget.CLS) {
    violations.push(`CLS ${metrics.CLS} เกิน ${performanceBudget.CLS}`);
  }
  
  if (violations.length > 0) {
    console.error('Performance Budget Violations:', violations);
    // ส่ง alert ไปทีม
  }
}
```

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: Performance Measurement
วัดเวลาของ functions เหล่านี้:
- ค้นหาใน array 100,000 elements ด้วย `find()` vs binary search
- เรียง array 10,000 elements ด้วย bubble sort vs native `sort()`

### แบบฝึกหัดที่ 2: Debounce/Throttle
- สร้าง search input ที่ใช้ debounce 300ms
- สร้าง scroll progress bar ที่ใช้ throttle 16ms

### แบบฝึกหัดที่ 3: Virtual List
สร้าง virtual list ที่แสดง 1,000,000 รายการ โดยแต่ละรายการมีความสูง 60px

### แบบฝึกหัดที่ 4: Web Worker
- สร้าง worker ที่คำนวณ prime numbers ถึง 1,000,000
- Main thread ต้องยังทำงานได้ปกติระหว่างที่ worker ทำงาน

### แบบฝึกหัดที่ 5: Memory Leak
หา memory leak ในโค้ดต่อไปนี้:
```javascript
class EventBus {
  constructor() {
    this.listeners = {};
  }
  
  on(event, callback) {
    if (!this.listeners[event]) {
      this.listeners[event] = [];
    }
    this.listeners[event].push(callback);
  }
  
  emit(event, data) {
    (this.listeners[event] || []).forEach(cb => cb(data));
  }
}

// ทุกครั้งที่ component สร้าง event listener แต่ไม่เคย off
function createComponent() {
  const bus = window.eventBus;
  bus.on('data-update', (data) => {
    // process data
    const result = processData(data);
    renderResult(result);
  });
}

// สร้าง components ซ้ำๆ
setInterval(createComponent, 1000);
```

---

## สรุป

| เทคนิค | ปัญหาที่แก้ |
|--------|------------|
| Code Splitting | Bundle ขนาดใหญ่ |
| Lazy Loading | โหลด resources ที่ไม่จำเป็น |
| Debounce/Throttle | Events ถี่เกินไป |
| Virtual Scrolling | Render items จำนวนมาก |
| Web Workers | Block main thread |
| Memory Management | Memory leaks |
| Event Delegation | Event listeners มากเกินไป |
| requestAnimationFrame | Animation ไม่ smooth |
| Tree Shaking | Bundle มี dead code |
| Caching | Request ซ้ำๆ |
