# Part 48: Intersection Observer API และ Observer APIs

## Observer APIs คืออะไร?

Observer APIs เป็นกลุ่ม API ที่ช่วยให้เราสังเกตการเปลี่ยนแปลงต่างๆ ใน browser อย่างมีประสิทธิภาพ โดยไม่ต้อง polling หรือใช้ scroll event listener ที่ใช้ทรัพยากรมาก

**Steps 931-950**

---

## Step 931: Intersection Observer คืออะไร

```javascript
// Intersection Observer API ช่วยให้เราตรวจสอบว่า element
// มีส่วนที่มองเห็นได้ใน viewport หรือใน container element อื่น

// ปัญหาของวิธีเดิม (scroll event):
window.addEventListener("scroll", () => {
  const elements = document.querySelectorAll(".lazy-image");
  elements.forEach(el => {
    const rect = el.getBoundingClientRect();
    // getBoundingClientRect() บังคับให้ browser คำนวณ layout ใหม่
    // ทุกครั้งที่เรียก = performance แย่มาก
    if (rect.top < window.innerHeight) {
      loadImage(el);
    }
  });
});

// ปัญหา: scroll event ยิงทุก pixel ที่เลื่อน
// getBoundingClientRect() ทำให้เกิด reflow
// รวมกันแล้วทำให้ UI กระตุก

// วิธีใหม่ด้วย Intersection Observer:
// - ไม่ต้อง scroll listener
// - Browser จัดการ calculation เอง (efficient)
// - Callback ยิงเฉพาะเมื่อ state เปลี่ยน
// - รองรับ threshold หลายค่า
```

---

## Step 932: สร้าง IntersectionObserver

```javascript
// syntax พื้นฐาน
const observer = new IntersectionObserver(callback, options);

// callback function
function callback(entries, observer) {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      console.log("Element เข้ามาใน viewport:", entry.target);
    } else {
      console.log("Element ออกจาก viewport:", entry.target);
    }
  });
}

// options
const options = {
  root: null,          // null = viewport
  rootMargin: "0px",   // margin รอบๆ root
  threshold: 0.1       // 10% ของ element ต้องมองเห็น
};

// สร้าง observer
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    console.log("Target:", entry.target.id);
    console.log("Is Intersecting:", entry.isIntersecting);
    console.log("Intersection Ratio:", entry.intersectionRatio);
    console.log("Bounding Client Rect:", entry.boundingClientRect);
    console.log("Intersection Rect:", entry.intersectionRect);
    console.log("Root Bounds:", entry.rootBounds);
    console.log("Time:", entry.time);
  });
}, { threshold: [0, 0.25, 0.5, 0.75, 1] });

// เริ่ม observe element
const element = document.querySelector(".observe-me");
observer.observe(element);
```

---

## Step 933: Options - root, rootMargin, threshold

```javascript
// root: element ที่ใช้เป็น viewport
// ถ้าไม่ระบุหรือ null = browser viewport

// ตัวอย่าง: observe ใน container element
const container = document.getElementById("scrollable-container");

const observer = new IntersectionObserver(
  (entries) => {
    entries.forEach(entry => {
      console.log(`${entry.target.id}: ${entry.isIntersecting ? "visible" : "hidden"}`);
    });
  },
  {
    root: container,  // ใช้ container เป็น viewport
    rootMargin: "10px 20px 30px 40px",  // top right bottom left
    threshold: 0.5  // 50% ต้องมองเห็น
  }
);
```

```javascript
// rootMargin: เพิ่ม/ลด margin รอบๆ root
// ใช้ negative ค่าเพื่อลด viewport ที่สังเกต
// ใช้ positive ค่าเพื่อเพิ่ม detection area

// ตัวอย่าง: load content ก่อนที่จะถึง viewport 200px
const lazyObserver = new IntersectionObserver(
  (entries) => {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        loadContent(entry.target);
      }
    });
  },
  {
    rootMargin: "200px 0px"  // เริ่ม detect เมื่อ element อยู่ห่าง 200px จาก bottom
  }
);
```

```javascript
// threshold: ค่าหรือ array ของค่า (0 ถึง 1)
// 0 = trigger เมื่อ element เริ่มเข้า viewport
// 1 = trigger เมื่อ element ทั้งหมดอยู่ใน viewport
// [0, 0.5, 1] = trigger ที่ 0%, 50%, 100%

const progressObserver = new IntersectionObserver(
  (entries) => {
    entries.forEach(entry => {
      const ratio = entry.intersectionRatio;
      entry.target.style.opacity = ratio;
      entry.target.style.transform = `scale(${0.5 + ratio * 0.5})`;
    });
  },
  {
    threshold: Array.from({ length: 101 }, (_, i) => i / 100)
    // threshold ทุก 1% = [0, 0.01, 0.02, ..., 1.0]
  }
);
```

---

## Step 934: IntersectionObserverEntry

```javascript
// entry object มี properties ดังนี้:

const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    // entry.target - element ที่กำลัง observe
    const element = entry.target;
    
    // entry.isIntersecting - boolean ว่า element อยู่ใน viewport
    const isVisible = entry.isIntersecting;
    
    // entry.intersectionRatio - สัดส่วนที่มองเห็น (0.0 - 1.0)
    const visiblePercent = Math.round(entry.intersectionRatio * 100);
    console.log(`${element.id} มองเห็น ${visiblePercent}%`);
    
    // entry.boundingClientRect - DOMRect ของ element
    const rect = entry.boundingClientRect;
    console.log(`ขนาด: ${rect.width}x${rect.height}`);
    
    // entry.intersectionRect - ส่วนที่ overlap กับ viewport
    const intersection = entry.intersectionRect;
    console.log(`พื้นที่ที่มองเห็น: ${intersection.width}x${intersection.height}`);
    
    // entry.rootBounds - DOMRect ของ root (viewport)
    const viewportRect = entry.rootBounds;
    if (viewportRect) {
      console.log(`Viewport ขนาด: ${viewportRect.width}x${viewportRect.height}`);
    }
    
    // entry.time - timestamp ของ event (milliseconds)
    console.log(`เวลา: ${entry.time}ms`);
  });
});
```

---

## Step 935: Observing และ Unobserving Elements

```javascript
// สร้าง observer
const observer = new IntersectionObserver(handleIntersection);

// Observe element เดียว
const element = document.getElementById("target");
observer.observe(element);

// Observe หลาย elements
const allTargets = document.querySelectorAll(".animate-on-scroll");
allTargets.forEach(el => observer.observe(el));

// Unobserve เมื่อไม่ต้องการ observe อีกแล้ว (ช่วย performance)
function handleIntersection(entries) {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      // ทำงานที่ต้องการ
      animateElement(entry.target);
      
      // หยุด observe เมื่อ animate แล้ว (one-time animation)
      observer.unobserve(entry.target);
    }
  });
}

// Disconnect - หยุด observe ทั้งหมด
observer.disconnect();

// ตัวอย่าง: Auto cleanup
class ScrollAnimator {
  constructor() {
    this.observer = new IntersectionObserver(
      this.handleIntersection.bind(this),
      { threshold: 0.2 }
    );
    this.animatedElements = new WeakSet();
  }
  
  observe(elements) {
    if (elements instanceof Element) {
      this.observer.observe(elements);
    } else {
      elements.forEach(el => this.observer.observe(el));
    }
  }
  
  handleIntersection(entries) {
    entries.forEach(entry => {
      if (entry.isIntersecting && !this.animatedElements.has(entry.target)) {
        this.animatedElements.add(entry.target);
        entry.target.classList.add("animated");
        this.observer.unobserve(entry.target);
      }
    });
  }
  
  destroy() {
    this.observer.disconnect();
  }
}
```

---

## Step 936: Lazy Loading Images

```javascript
// HTML
// <img data-src="/images/photo.jpg" alt="..." class="lazy" src="/images/placeholder.svg">

// JavaScript
function setupLazyLoading() {
  const lazyImages = document.querySelectorAll("img.lazy[data-src]");
  
  if (!lazyImages.length) return;
  
  // ตรวจสอบว่า browser รองรับ Intersection Observer
  if (!("IntersectionObserver" in window)) {
    // Fallback: โหลดทั้งหมดทันที
    lazyImages.forEach(img => {
      img.src = img.dataset.src;
      if (img.dataset.srcset) img.srcset = img.dataset.srcset;
    });
    return;
  }
  
  const imageObserver = new IntersectionObserver(
    (entries, observer) => {
      entries.forEach(entry => {
        if (!entry.isIntersecting) return;
        
        const img = entry.target;
        
        // โหลดภาพ
        img.src = img.dataset.src;
        
        // รองรับ srcset สำหรับ responsive images
        if (img.dataset.srcset) {
          img.srcset = img.dataset.srcset;
        }
        
        img.classList.remove("lazy");
        img.classList.add("loaded");
        
        // Fade in effect
        img.style.opacity = "0";
        img.onload = () => {
          img.style.transition = "opacity 0.3s";
          img.style.opacity = "1";
        };
        
        observer.unobserve(img);
      });
    },
    {
      rootMargin: "200px 0px",  // เริ่มโหลดก่อน 200px
      threshold: 0
    }
  );
  
  lazyImages.forEach(img => imageObserver.observe(img));
}

// เรียกใช้
setupLazyLoading();

// Lazy load background images
function lazyLoadBackgrounds() {
  const elements = document.querySelectorAll("[data-bg]");
  
  const observer = new IntersectionObserver((entries) => {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        entry.target.style.backgroundImage = `url(${entry.target.dataset.bg})`;
        entry.target.classList.add("bg-loaded");
        observer.unobserve(entry.target);
      }
    });
  });
  
  elements.forEach(el => observer.observe(el));
}
```

---

## Step 937: Infinite Scroll

```javascript
// ตัวอย่าง Infinite Scroll

class InfiniteScroll {
  constructor(options) {
    this.container = options.container;
    this.loadMore = options.loadMore;
    this.loading = false;
    this.page = 1;
    this.hasMore = true;
    
    this.sentinel = this.createSentinel();
    this.container.appendChild(this.sentinel);
    
    this.observer = new IntersectionObserver(
      this.handleIntersection.bind(this),
      {
        root: null,
        rootMargin: "0px 0px 400px 0px",  // เริ่มโหลดก่อน 400px
        threshold: 0
      }
    );
    
    this.observer.observe(this.sentinel);
  }
  
  createSentinel() {
    const sentinel = document.createElement("div");
    sentinel.className = "scroll-sentinel";
    sentinel.style.cssText = "height: 1px; width: 100%;";
    return sentinel;
  }
  
  async handleIntersection(entries) {
    const entry = entries[0];
    
    if (entry.isIntersecting && !this.loading && this.hasMore) {
      await this.fetchMore();
    }
  }
  
  async fetchMore() {
    this.loading = true;
    this.showLoading();
    
    try {
      const newItems = await this.loadMore(this.page);
      
      if (newItems.length === 0) {
        this.hasMore = false;
        this.showEndMessage();
        this.observer.unobserve(this.sentinel);
      } else {
        this.renderItems(newItems);
        this.page++;
      }
    } catch (error) {
      this.showError(error);
    } finally {
      this.loading = false;
      this.hideLoading();
    }
  }
  
  renderItems(items) {
    const fragment = document.createDocumentFragment();
    
    items.forEach(item => {
      const el = document.createElement("div");
      el.className = "item";
      el.innerHTML = `
        <h3>${item.title}</h3>
        <p>${item.description}</p>
        <img data-src="${item.image}" class="lazy" alt="${item.title}">
      `;
      fragment.appendChild(el);
    });
    
    // Insert before sentinel
    this.container.insertBefore(fragment, this.sentinel);
    
    // Setup lazy loading สำหรับรูปภาพใหม่
    setupLazyLoading();
  }
  
  showLoading() {
    const loader = document.createElement("div");
    loader.id = "scroll-loader";
    loader.innerHTML = `
      <div class="spinner"></div>
      <p>กำลังโหลด...</p>
    `;
    this.container.insertBefore(loader, this.sentinel);
  }
  
  hideLoading() {
    document.getElementById("scroll-loader")?.remove();
  }
  
  showEndMessage() {
    const msg = document.createElement("p");
    msg.textContent = "ไม่มีข้อมูลเพิ่มเติม";
    msg.className = "end-message";
    this.container.appendChild(msg);
  }
  
  showError(error) {
    const errorEl = document.createElement("div");
    errorEl.className = "error";
    errorEl.innerHTML = `
      <p>เกิดข้อผิดพลาด: ${error.message}</p>
      <button onclick="infiniteScroll.fetchMore()">ลองใหม่</button>
    `;
    this.container.insertBefore(errorEl, this.sentinel);
  }
  
  destroy() {
    this.observer.disconnect();
    this.sentinel.remove();
  }
}

// การใช้งาน
const infiniteScroll = new InfiniteScroll({
  container: document.getElementById("items-container"),
  loadMore: async (page) => {
    const response = await fetch(`/api/items?page=${page}&limit=20`);
    const data = await response.json();
    return data.items;
  }
});
```

---

## Step 938: Sticky Headers Detection

```javascript
// Detect เมื่อ header กลายเป็น sticky

function setupStickyHeader() {
  const header = document.querySelector(".sticky-header");
  const sentinel = document.createElement("div");
  
  // สร้าง sentinel ไว้ก่อนหน้า header
  sentinel.style.cssText = "height: 1px; position: absolute; top: 0; width: 100%;";
  header.parentElement.insertBefore(sentinel, header);
  
  const observer = new IntersectionObserver(
    ([entry]) => {
      // เมื่อ sentinel หายออกจาก viewport = header sticky แล้ว
      header.classList.toggle("is-sticky", !entry.isIntersecting);
      
      if (!entry.isIntersecting) {
        header.style.boxShadow = "0 2px 10px rgba(0,0,0,0.1)";
      } else {
        header.style.boxShadow = "none";
      }
    },
    {
      threshold: [1],  // threshold 1 = trigger เมื่อ sentinel ออกจาก viewport สมบูรณ์
      rootMargin: `-${header.offsetHeight}px 0px 0px 0px`
    }
  );
  
  observer.observe(sentinel);
}

setupStickyHeader();
```

---

## Step 939: Scroll Progress Indicator

```javascript
// Progress bar แสดงการอ่านบทความ

function setupReadingProgress() {
  const progressBar = document.getElementById("reading-progress");
  const article = document.querySelector("article");
  
  if (!progressBar || !article) return;
  
  // ใช้ multiple thresholds เพื่อ smooth progress
  const thresholds = Array.from({ length: 100 }, (_, i) => i / 100);
  
  const observer = new IntersectionObserver(
    ([entry]) => {
      if (entry.isIntersecting) {
        const ratio = entry.intersectionRatio;
        progressBar.style.width = `${ratio * 100}%`;
        
        // เปลี่ยนสี progress bar ตาม progress
        const hue = Math.round(ratio * 120); // จาก 0 (red) ถึง 120 (green)
        progressBar.style.backgroundColor = `hsl(${hue}, 80%, 50%)`;
      }
    },
    {
      root: null,
      threshold: thresholds
    }
  );
  
  observer.observe(article);
}

// Scroll progress ที่แม่นยำกว่า
function setupDetailedProgress() {
  const progressBar = document.getElementById("progress-bar");
  const content = document.getElementById("content");
  
  let ticking = false;
  
  const observer = new IntersectionObserver(
    (entries) => {
      entries.forEach(entry => {
        if (!ticking) {
          requestAnimationFrame(() => {
            const scrolled = -entry.boundingClientRect.top;
            const total = entry.boundingClientRect.height - window.innerHeight;
            const progress = Math.min(Math.max(scrolled / total, 0), 1);
            
            progressBar.style.width = `${progress * 100}%`;
            progressBar.setAttribute("aria-valuenow", Math.round(progress * 100));
            
            ticking = false;
          });
          ticking = true;
        }
      });
    },
    {
      threshold: Array.from({ length: 1000 }, (_, i) => i / 1000)
    }
  );
  
  observer.observe(content);
}
```

---

## Step 940: Animate on Scroll

```javascript
// Animate elements เมื่อเข้า viewport

// CSS สำหรับ animations
const animationCSS = `
  .animate-fade-in {
    opacity: 0;
    transition: opacity 0.6s ease;
  }
  
  .animate-slide-up {
    opacity: 0;
    transform: translateY(50px);
    transition: opacity 0.6s ease, transform 0.6s ease;
  }
  
  .animate-slide-left {
    opacity: 0;
    transform: translateX(-50px);
    transition: opacity 0.6s ease, transform 0.6s ease;
  }
  
  .animate-scale {
    opacity: 0;
    transform: scale(0.8);
    transition: opacity 0.6s ease, transform 0.6s ease;
  }
  
  .animated {
    opacity: 1 !important;
    transform: none !important;
  }
`;

// ใส่ CSS ลงใน document
const style = document.createElement("style");
style.textContent = animationCSS;
document.head.appendChild(style);

// JavaScript สำหรับ trigger animations
class ScrollAnimations {
  constructor() {
    this.observer = new IntersectionObserver(
      this.handleIntersection.bind(this),
      {
        threshold: 0.15,
        rootMargin: "0px 0px -50px 0px"  // trigger เมื่อ element อยู่สูงกว่า bottom 50px
      }
    );
    
    this.init();
  }
  
  init() {
    const animatableElements = document.querySelectorAll(
      ".animate-fade-in, .animate-slide-up, .animate-slide-left, .animate-scale, [data-animation]"
    );
    
    animatableElements.forEach((el, index) => {
      // เพิ่ม delay สำหรับ elements ที่อยู่ใกล้กัน
      const delay = el.dataset.delay || (index % 5) * 100;
      el.style.transitionDelay = `${delay}ms`;
      
      this.observer.observe(el);
    });
  }
  
  handleIntersection(entries) {
    entries.forEach(entry => {
      if (entry.isIntersecting) {
        const el = entry.target;
        
        // ใช้ custom animation ถ้าระบุไว้
        if (el.dataset.animation) {
          el.classList.add(el.dataset.animation);
        }
        
        el.classList.add("animated");
        
        // Unobserve หลัง animate แล้ว (ทำแค่ครั้งเดียว)
        this.observer.unobserve(el);
      }
    });
  }
  
  destroy() {
    this.observer.disconnect();
  }
}

const scrollAnimations = new ScrollAnimations();
```

---

## Step 941: Performance vs Scroll Event Listener

```javascript
// เปรียบเทียบ performance

// วิธีเก่า: scroll event listener
let scrollTimeout;
let elementPositions = [];

function initScrollListener() {
  // ต้องคำนวณ positions ทุกครั้งที่ scroll
  window.addEventListener("scroll", () => {
    // การเรียก getBoundingClientRect ทุก scroll event = ช้ามาก
    const elements = document.querySelectorAll(".lazy");
    elements.forEach(el => {
      const rect = el.getBoundingClientRect();  // forced reflow!
      if (rect.top < window.innerHeight + 200) {
        loadElement(el);
      }
    });
  });
}

// วิธีใหม่: Intersection Observer
function initIntersectionObserver() {
  const observer = new IntersectionObserver(
    (entries) => {
      // callback ยิงแค่เมื่อ state เปลี่ยน ไม่ใช่ทุก pixel
      entries.forEach(entry => {
        if (entry.isIntersecting) {
          loadElement(entry.target);
          observer.unobserve(entry.target);
        }
      });
    },
    { rootMargin: "200px" }
  );
  
  document.querySelectorAll(".lazy").forEach(el => observer.observe(el));
}

// Benchmark
function benchmark() {
  const results = { scroll: 0, observer: 0 };
  const iterations = 1000;
  
  // Scroll listener approach
  let start = performance.now();
  for (let i = 0; i < iterations; i++) {
    document.querySelectorAll(".lazy").forEach(el => {
      const rect = el.getBoundingClientRect();  // forced reflow
      if (rect.top < window.innerHeight) {
        // do something
      }
    });
  }
  results.scroll = performance.now() - start;
  
  console.log(`Scroll approach: ${results.scroll.toFixed(2)}ms`);
  console.log(`Observer approach: เกือบ 0ms (browser จัดการ)`);
  console.log(`Scroll ช้ากว่า ~${Math.round(results.scroll)}x`);
}
```

---

## Step 942: MutationObserver - Observing DOM Changes

```javascript
// MutationObserver ช่วยให้สังเกต DOM mutations

// สร้าง observer
const mutationObserver = new MutationObserver((mutations) => {
  mutations.forEach(mutation => {
    console.log("Mutation type:", mutation.type);
    
    if (mutation.type === "childList") {
      console.log("Nodes เพิ่มเข้า:", mutation.addedNodes);
      console.log("Nodes ถูกลบ:", mutation.removedNodes);
    } else if (mutation.type === "attributes") {
      console.log("Attribute เปลี่ยน:", mutation.attributeName);
      console.log("ค่าเก่า:", mutation.oldValue);
    } else if (mutation.type === "characterData") {
      console.log("Text เปลี่ยนแปลง");
    }
  });
});

// กำหนด element ที่จะ observe
const target = document.getElementById("dynamic-content");

// options
const config = {
  childList: true,       // observe child nodes เพิ่ม/ลบ
  attributes: true,      // observe attribute changes
  characterData: true,   // observe text content changes
  subtree: true,         // observe ทั้ง subtree
  attributeOldValue: true,  // เก็บ old attribute value
  characterDataOldValue: true  // เก็บ old text value
};

mutationObserver.observe(target, config);

// หยุด observe
mutationObserver.disconnect();
```

---

## Step 943: MutationObserver Use Cases

```javascript
// Use Case 1: ตรวจจับ elements ที่ถูกเพิ่มเข้า DOM โดย third-party library

const bodyObserver = new MutationObserver((mutations) => {
  mutations.forEach(mutation => {
    mutation.addedNodes.forEach(node => {
      if (node.nodeType !== Node.ELEMENT_NODE) return;
      
      // ตรวจหา ads ที่ถูกเพิ่มโดย ad library
      if (node.classList.contains("ad-container")) {
        node.style.display = "none";
      }
      
      // Auto-initialize components
      if (node.dataset.component) {
        initializeComponent(node);
      }
      
      // Setup lazy loading สำหรับ images ใหม่
      const newImages = node.querySelectorAll("img[data-src]");
      newImages.forEach(img => lazyObserver.observe(img));
    });
  });
});

bodyObserver.observe(document.body, { childList: true, subtree: true });
```

```javascript
// Use Case 2: Watch สำหรับ class changes

function watchClassChange(element, className, callback) {
  const observer = new MutationObserver((mutations) => {
    mutations.forEach(mutation => {
      if (mutation.type === "attributes" && mutation.attributeName === "class") {
        const hasClass = element.classList.contains(className);
        const hadClass = mutation.oldValue?.split(" ").includes(className);
        
        if (hasClass !== hadClass) {
          callback(hasClass, mutation.oldValue);
        }
      }
    });
  });
  
  observer.observe(element, {
    attributes: true,
    attributeOldValue: true,
    attributeFilter: ["class"]
  });
  
  return () => observer.disconnect();
}

// การใช้งาน
const unwatch = watchClassChange(
  document.getElementById("modal"),
  "visible",
  (isVisible) => {
    console.log(`Modal ${isVisible ? "เปิด" : "ปิด"} แล้ว`);
    if (isVisible) {
      document.body.style.overflow = "hidden";
    } else {
      document.body.style.overflow = "";
    }
  }
);

// หยุด watch
unwatch();
```

```javascript
// Use Case 3: Content Security Monitor (XSS detection)

const securityObserver = new MutationObserver((mutations) => {
  mutations.forEach(mutation => {
    mutation.addedNodes.forEach(node => {
      if (node.nodeType !== Node.ELEMENT_NODE) return;
      
      // ตรวจหา script tags ที่ถูกเพิ่มแบบ dynamic
      if (node.tagName === "SCRIPT") {
        console.warn("Script tag ถูกเพิ่มเข้า DOM:", node.src || "inline");
        // อาจ remove หรือ report
      }
      
      // ตรวจหา event handlers ที่ inline
      const elements = [node, ...node.querySelectorAll("*")];
      elements.forEach(el => {
        if (el.getAttribute && 
            Array.from(el.attributes).some(attr => attr.name.startsWith("on"))) {
          console.warn("Inline event handler detected:", el.outerHTML);
        }
      });
    });
  });
});

securityObserver.observe(document.body, { childList: true, subtree: true });
```

```javascript
// Use Case 4: Form Auto-save

function setupAutoSave(formId) {
  const form = document.getElementById(formId);
  if (!form) return;
  
  const saveData = debounce(() => {
    const formData = new FormData(form);
    const data = Object.fromEntries(formData);
    localStorage.setItem(`form-draft-${formId}`, JSON.stringify(data));
    showSaveIndicator();
  }, 1000);
  
  const observer = new MutationObserver((mutations) => {
    mutations.forEach(mutation => {
      if (mutation.type === "characterData" || 
          mutation.attributeName === "value") {
        saveData();
      }
    });
  });
  
  // Observe ทุก input ใน form
  form.querySelectorAll("input, textarea, select").forEach(input => {
    // Listen ด้วย event listener ด้วย (MutationObserver ไม่ detect user input)
    input.addEventListener("input", saveData);
    input.addEventListener("change", saveData);
  });
  
  // Observe DOM structure changes (เมื่อมี fields เพิ่มใหม่)
  observer.observe(form, { childList: true, subtree: true });
  
  // โหลด draft ถ้ามี
  const saved = localStorage.getItem(`form-draft-${formId}`);
  if (saved) {
    try {
      const data = JSON.parse(saved);
      Object.entries(data).forEach(([name, value]) => {
        const input = form.querySelector(`[name="${name}"]`);
        if (input) input.value = value;
      });
    } catch {}
  }
}

function debounce(fn, delay) {
  let timer;
  return (...args) => {
    clearTimeout(timer);
    timer = setTimeout(() => fn(...args), delay);
  };
}
```

---

## Step 944: MutationObserver - Virtual DOM Concept

```javascript
// จำลองแนวคิด Virtual DOM อย่างง่าย

class MiniReactivity {
  constructor(rootElement) {
    this.root = rootElement;
    this.listeners = new Map();
    this.setupObserver();
  }
  
  setupObserver() {
    this.observer = new MutationObserver((mutations) => {
      mutations.forEach(mutation => {
        if (mutation.type === "attributes") {
          const element = mutation.target;
          const attrName = mutation.attributeName;
          const oldValue = mutation.oldValue;
          const newValue = element.getAttribute(attrName);
          
          this.emit(`${element.id}:${attrName}`, { oldValue, newValue, element });
        } else if (mutation.type === "childList") {
          mutation.addedNodes.forEach(node => {
            if (node.dataset?.bind) {
              this.bindElement(node);
            }
          });
        }
      });
    });
    
    this.observer.observe(this.root, {
      attributes: true,
      attributeOldValue: true,
      childList: true,
      subtree: true
    });
  }
  
  on(event, handler) {
    if (!this.listeners.has(event)) {
      this.listeners.set(event, []);
    }
    this.listeners.get(event).push(handler);
  }
  
  emit(event, data) {
    const handlers = this.listeners.get(event) || [];
    handlers.forEach(handler => handler(data));
  }
  
  bindElement(element) {
    const key = element.dataset.bind;
    
    // Watch attribute changes ของ element นี้
    this.on(`${element.id}:data-value`, ({ newValue }) => {
      element.textContent = newValue;
    });
  }
}
```

---

## Step 945: ResizeObserver

```javascript
// ResizeObserver สังเกตการเปลี่ยนขนาดของ elements

// สร้าง ResizeObserver
const resizeObserver = new ResizeObserver((entries) => {
  entries.forEach(entry => {
    const element = entry.target;
    
    // entry.contentRect - ขนาด content area
    const { width, height } = entry.contentRect;
    console.log(`${element.id}: ${width}px x ${height}px`);
    
    // entry.borderBoxSize - ขนาด รวม border
    // entry.contentBoxSize - ขนาด content เท่านั้น
    // entry.devicePixelContentBoxSize - ขนาดใน device pixels
    
    if (entry.borderBoxSize) {
      const { inlineSize, blockSize } = entry.borderBoxSize[0];
      console.log(`Border box: ${inlineSize}px x ${blockSize}px`);
    }
  });
});

// Observe element
const container = document.getElementById("resizable-container");
resizeObserver.observe(container);

// Unobserve
resizeObserver.unobserve(container);

// Disconnect ทั้งหมด
resizeObserver.disconnect();
```

```javascript
// Use Case: Responsive Component

class ResponsiveChart {
  constructor(containerId, data) {
    this.container = document.getElementById(containerId);
    this.data = data;
    this.canvas = document.createElement("canvas");
    this.container.appendChild(this.canvas);
    
    this.render();
    this.setupResizeObserver();
  }
  
  setupResizeObserver() {
    this.resizeObserver = new ResizeObserver(
      debounce((entries) => {
        const entry = entries[0];
        const { width, height } = entry.contentRect;
        
        this.canvas.width = width;
        this.canvas.height = height;
        
        this.render();
      }, 100)
    );
    
    this.resizeObserver.observe(this.container);
  }
  
  render() {
    const ctx = this.canvas.getContext("2d");
    const width = this.canvas.width;
    const height = this.canvas.height;
    
    // Clear canvas
    ctx.clearRect(0, 0, width, height);
    
    // วาด chart ตาม current size
    this.drawChart(ctx, width, height);
  }
  
  drawChart(ctx, width, height) {
    const maxValue = Math.max(...this.data);
    const barWidth = (width - 40) / this.data.length;
    const barSpacing = 5;
    
    ctx.fillStyle = "#007bff";
    
    this.data.forEach((value, index) => {
      const barHeight = ((value / maxValue) * (height - 40));
      const x = 20 + index * barWidth + barSpacing;
      const y = height - 20 - barHeight;
      
      ctx.fillRect(x, y, barWidth - barSpacing * 2, barHeight);
    });
  }
  
  destroy() {
    this.resizeObserver.disconnect();
  }
}

// การใช้งาน
const chart = new ResponsiveChart("chart-container", [30, 80, 45, 60, 90, 25, 70]);
```

---

## Step 946: ResizeObserver - Advanced Patterns

```javascript
// Container Queries simulation ด้วย ResizeObserver
// (ก่อนที่ Container Queries CSS จะ supported ทุก browser)

class ContainerQuery {
  constructor(element, queries) {
    this.element = element;
    this.queries = queries;  // { small: 400, medium: 768, large: 1200 }
    this.currentBreakpoint = null;
    
    this.observer = new ResizeObserver(([entry]) => {
      const width = entry.contentRect.width;
      this.applyBreakpoint(width);
    });
    
    this.observer.observe(element);
    
    // ใช้ initial size
    this.applyBreakpoint(element.offsetWidth);
  }
  
  applyBreakpoint(width) {
    let breakpoint = "default";
    
    for (const [name, minWidth] of Object.entries(this.queries)) {
      if (width >= minWidth) {
        breakpoint = name;
      }
    }
    
    if (breakpoint !== this.currentBreakpoint) {
      this.currentBreakpoint = breakpoint;
      
      // ลบ class เก่าทั้งหมด
      Object.keys(this.queries).forEach(name => {
        this.element.classList.remove(`cq-${name}`);
      });
      
      // เพิ่ม class ใหม่
      if (breakpoint !== "default") {
        this.element.classList.add(`cq-${breakpoint}`);
      }
      
      // Dispatch custom event
      this.element.dispatchEvent(new CustomEvent("breakpointChange", {
        detail: { breakpoint, width },
        bubbles: true
      }));
    }
  }
  
  destroy() {
    this.observer.disconnect();
  }
}

// การใช้งาน
const card = document.querySelector(".card");
const containerQuery = new ContainerQuery(card, {
  small: 200,
  medium: 400,
  large: 600
});

// CSS
// .card.cq-small { font-size: 14px; }
// .card.cq-medium { font-size: 16px; flex-direction: row; }
// .card.cq-large { font-size: 18px; padding: 2rem; }
```

---

## Step 947: PerformanceObserver

```javascript
// PerformanceObserver สังเกต performance metrics

// สังเกต Long Tasks (งานที่ใช้เวลา > 50ms)
const longTaskObserver = new PerformanceObserver((list) => {
  const entries = list.getEntries();
  
  entries.forEach(entry => {
    console.warn(`Long Task detected: ${entry.duration.toFixed(2)}ms`);
    console.log(`Start time: ${entry.startTime.toFixed(2)}ms`);
    console.log(`Attribution:`, entry.attribution);
  });
});

longTaskObserver.observe({ type: "longtask", buffered: true });

// สังเกต Largest Contentful Paint (LCP)
const lcpObserver = new PerformanceObserver((list) => {
  const entries = list.getEntries();
  const lastEntry = entries.at(-1);
  
  console.log(`LCP: ${lastEntry.startTime.toFixed(2)}ms`);
  console.log(`LCP Element:`, lastEntry.element);
  console.log(`LCP Size: ${lastEntry.size}px`);
});

lcpObserver.observe({ type: "largest-contentful-paint", buffered: true });

// สังเกต First Input Delay (FID)
const fidObserver = new PerformanceObserver((list) => {
  list.getEntries().forEach(entry => {
    const delay = entry.processingStart - entry.startTime;
    console.log(`FID: ${delay.toFixed(2)}ms`);
    
    if (delay > 100) {
      console.warn("FID > 100ms - ควรปรับปรุง!");
    }
  });
});

fidObserver.observe({ type: "first-input", buffered: true });

// สังเกต Cumulative Layout Shift (CLS)
let clsScore = 0;
const clsObserver = new PerformanceObserver((list) => {
  list.getEntries().forEach(entry => {
    if (!entry.hadRecentInput) {
      clsScore += entry.value;
      console.log(`CLS score: ${clsScore.toFixed(4)}`);
    }
  });
});

clsObserver.observe({ type: "layout-shift", buffered: true });
```

```javascript
// Core Web Vitals Monitor
class WebVitalsMonitor {
  constructor() {
    this.metrics = {
      lcp: null,
      fid: null,
      cls: 0,
      fcp: null,
      ttfb: null
    };
    
    this.setupObservers();
  }
  
  setupObservers() {
    // LCP
    new PerformanceObserver((list) => {
      const entries = list.getEntries();
      const lcp = entries.at(-1);
      this.metrics.lcp = lcp.startTime;
      this.reportMetric("LCP", lcp.startTime, this.getLCPRating(lcp.startTime));
    }).observe({ type: "largest-contentful-paint", buffered: true });
    
    // FID
    new PerformanceObserver((list) => {
      list.getEntries().forEach(entry => {
        const fid = entry.processingStart - entry.startTime;
        this.metrics.fid = fid;
        this.reportMetric("FID", fid, this.getFIDRating(fid));
      });
    }).observe({ type: "first-input", buffered: true });
    
    // CLS
    new PerformanceObserver((list) => {
      list.getEntries().forEach(entry => {
        if (!entry.hadRecentInput) {
          this.metrics.cls += entry.value;
          this.reportMetric("CLS", this.metrics.cls, this.getCLSRating(this.metrics.cls));
        }
      });
    }).observe({ type: "layout-shift", buffered: true });
    
    // FCP
    new PerformanceObserver((list) => {
      list.getEntries().forEach(entry => {
        if (entry.name === "first-contentful-paint") {
          this.metrics.fcp = entry.startTime;
          this.reportMetric("FCP", entry.startTime, this.getFCPRating(entry.startTime));
        }
      });
    }).observe({ type: "paint", buffered: true });
  }
  
  getLCPRating(value) {
    if (value <= 2500) return "good";
    if (value <= 4000) return "needs-improvement";
    return "poor";
  }
  
  getFIDRating(value) {
    if (value <= 100) return "good";
    if (value <= 300) return "needs-improvement";
    return "poor";
  }
  
  getCLSRating(value) {
    if (value <= 0.1) return "good";
    if (value <= 0.25) return "needs-improvement";
    return "poor";
  }
  
  getFCPRating(value) {
    if (value <= 1800) return "good";
    if (value <= 3000) return "needs-improvement";
    return "poor";
  }
  
  reportMetric(name, value, rating) {
    const formatted = name === "CLS" ? value.toFixed(4) : `${value.toFixed(0)}ms`;
    console.log(`[${rating.toUpperCase()}] ${name}: ${formatted}`);
    
    // ส่งไปยัง analytics
    if (window.gtag) {
      window.gtag("event", name, {
        value: Math.round(name === "CLS" ? value * 1000 : value),
        metric_rating: rating
      });
    }
  }
  
  getReport() {
    return { ...this.metrics };
  }
}

const monitor = new WebVitalsMonitor();
```

---

## Step 948: Practical Example - Virtual List

```javascript
// Virtual List ด้วย IntersectionObserver
// แสดงเฉพาะ items ที่อยู่ใน viewport

class VirtualList {
  constructor(container, items, itemHeight = 50) {
    this.container = container;
    this.items = items;
    this.itemHeight = itemHeight;
    this.visibleItems = new Map();
    
    this.container.style.overflow = "auto";
    this.container.style.position = "relative";
    
    // สร้าง spacer สำหรับ total height
    this.spacer = document.createElement("div");
    this.spacer.style.height = `${items.length * itemHeight}px`;
    this.container.appendChild(this.spacer);
    
    this.setupObserver();
    this.renderVisible();
  }
  
  setupObserver() {
    this.observer = new IntersectionObserver(
      this.handleIntersection.bind(this),
      {
        root: this.container,
        rootMargin: `${this.itemHeight * 5}px 0px`,
        threshold: 0
      }
    );
  }
  
  renderVisible() {
    const containerHeight = this.container.clientHeight;
    const scrollTop = this.container.scrollTop;
    
    const startIndex = Math.floor(scrollTop / this.itemHeight);
    const visibleCount = Math.ceil(containerHeight / this.itemHeight) + 10;
    const endIndex = Math.min(startIndex + visibleCount, this.items.length - 1);
    
    for (let i = startIndex; i <= endIndex; i++) {
      if (!this.visibleItems.has(i)) {
        this.renderItem(i);
      }
    }
    
    // Remove items ที่ไม่ได้ใช้
    for (const [index, el] of this.visibleItems) {
      if (index < startIndex - 20 || index > endIndex + 20) {
        el.remove();
        this.visibleItems.delete(index);
      }
    }
  }
  
  renderItem(index) {
    const item = this.items[index];
    const el = document.createElement("div");
    el.className = "virtual-item";
    el.style.cssText = `
      position: absolute;
      top: ${index * this.itemHeight}px;
      left: 0;
      right: 0;
      height: ${this.itemHeight}px;
      display: flex;
      align-items: center;
      padding: 0 16px;
      border-bottom: 1px solid #eee;
    `;
    el.textContent = typeof item === "string" ? item : JSON.stringify(item);
    
    this.container.appendChild(el);
    this.visibleItems.set(index, el);
  }
  
  init() {
    this.container.addEventListener("scroll", () => {
      requestAnimationFrame(() => this.renderVisible());
    });
    
    this.renderVisible();
  }
}

// การใช้งาน
const container = document.getElementById("list-container");
container.style.height = "500px";

const items = Array.from({ length: 10000 }, (_, i) => `รายการที่ ${i + 1}`);
const virtualList = new VirtualList(container, items);
virtualList.init();
```

---

## Step 949: ตัวอย่าง All Observers Together

```javascript
// dashboard.js - ใช้ Observers ทั้งหมดร่วมกัน

class ObserverDashboard {
  constructor() {
    this.setupIntersectionObserver();
    this.setupMutationObserver();
    this.setupResizeObserver();
    this.setupPerformanceObserver();
  }
  
  setupIntersectionObserver() {
    // Lazy load sections
    this.intersectionObs = new IntersectionObserver(
      (entries) => {
        entries.forEach(entry => {
          if (entry.isIntersecting) {
            this.loadSection(entry.target);
          }
        });
      },
      { threshold: 0.1, rootMargin: "100px" }
    );
    
    document.querySelectorAll("[data-lazy-section]").forEach(section => {
      this.intersectionObs.observe(section);
    });
  }
  
  setupMutationObserver() {
    // Watch for dynamic content additions
    this.mutationObs = new MutationObserver((mutations) => {
      mutations.forEach(mutation => {
        mutation.addedNodes.forEach(node => {
          if (node.nodeType === Node.ELEMENT_NODE) {
            // Auto-initialize any new components
            this.initNewComponents(node);
          }
        });
      });
    });
    
    this.mutationObs.observe(document.body, {
      childList: true,
      subtree: true
    });
  }
  
  setupResizeObserver() {
    // Make charts responsive
    this.resizeObs = new ResizeObserver(
      debounce((entries) => {
        entries.forEach(entry => {
          this.resizeChart(entry.target, entry.contentRect);
        });
      }, 250)
    );
    
    document.querySelectorAll(".chart-container").forEach(container => {
      this.resizeObs.observe(container);
    });
  }
  
  setupPerformanceObserver() {
    // Monitor performance
    try {
      this.perfObs = new PerformanceObserver((list) => {
        list.getEntries().forEach(entry => {
          if (entry.duration > 50) {
            console.warn(`Long task: ${entry.duration.toFixed(0)}ms`);
          }
        });
      });
      
      this.perfObs.observe({ type: "longtask" });
    } catch (e) {
      console.log("Long Task Observer not supported");
    }
  }
  
  async loadSection(section) {
    const sectionId = section.dataset.lazySection;
    section.innerHTML = `<div class="loading">กำลังโหลด ${sectionId}...</div>`;
    
    try {
      const response = await fetch(`/api/sections/${sectionId}`);
      const html = await response.text();
      section.innerHTML = html;
      this.intersectionObs.unobserve(section);
    } catch (error) {
      section.innerHTML = `<div class="error">โหลดไม่สำเร็จ</div>`;
    }
  }
  
  initNewComponents(element) {
    element.querySelectorAll("[data-component]").forEach(comp => {
      const componentName = comp.dataset.component;
      if (window.Components?.[componentName]) {
        new window.Components[componentName](comp);
      }
    });
  }
  
  resizeChart(container, rect) {
    const chart = container.querySelector("canvas");
    if (chart) {
      chart.width = rect.width;
      chart.height = rect.height;
      // Re-render chart
    }
  }
  
  destroy() {
    this.intersectionObs.disconnect();
    this.mutationObs.disconnect();
    this.resizeObs.disconnect();
    this.perfObs?.disconnect();
  }
}

const dashboard = new ObserverDashboard();
```

---

## Step 950: Best Practices และ Patterns

```javascript
// 1. Observer Manager - จัดการ observers ทั้งหมด

class ObserverManager {
  constructor() {
    this._observers = new Map();
  }
  
  register(name, observer) {
    if (this._observers.has(name)) {
      this._observers.get(name).disconnect();
    }
    this._observers.set(name, observer);
    return observer;
  }
  
  get(name) {
    return this._observers.get(name);
  }
  
  disconnect(name) {
    const observer = this._observers.get(name);
    if (observer) {
      observer.disconnect();
      this._observers.delete(name);
    }
  }
  
  disconnectAll() {
    this._observers.forEach(observer => observer.disconnect());
    this._observers.clear();
  }
}

const observers = new ObserverManager();

// ลงทะเบียน observers
observers.register("lazy", new IntersectionObserver(handleLazyLoad));
observers.register("mutations", new MutationObserver(handleMutations));
observers.register("resize", new ResizeObserver(handleResize));

// Cleanup เมื่อหน้า unload
window.addEventListener("beforeunload", () => {
  observers.disconnectAll();
});
```

```javascript
// 2. Observable Element decorator

function makeObservable(elementOrSelector, options = {}) {
  const element = typeof elementOrSelector === "string"
    ? document.querySelector(elementOrSelector)
    : elementOrSelector;
  
  if (!element) return null;
  
  const state = {
    isVisible: false,
    visibilityRatio: 0,
    width: element.offsetWidth,
    height: element.offsetHeight
  };
  
  const listeners = { visibility: [], resize: [], mutation: [] };
  
  // Intersection Observer
  const io = new IntersectionObserver(([entry]) => {
    state.isVisible = entry.isIntersecting;
    state.visibilityRatio = entry.intersectionRatio;
    listeners.visibility.forEach(fn => fn(state));
  }, options.intersection || { threshold: [0, 0.5, 1] });
  
  io.observe(element);
  
  // Resize Observer
  const ro = new ResizeObserver(([entry]) => {
    state.width = entry.contentRect.width;
    state.height = entry.contentRect.height;
    listeners.resize.forEach(fn => fn(state));
  });
  
  ro.observe(element);
  
  // Mutation Observer
  const mo = new MutationObserver((mutations) => {
    listeners.mutation.forEach(fn => fn(mutations, state));
  });
  
  mo.observe(element, options.mutation || { childList: true, attributes: true });
  
  return {
    element,
    state,
    on(event, fn) {
      listeners[event]?.push(fn);
      return this;
    },
    off(event, fn) {
      if (listeners[event]) {
        listeners[event] = listeners[event].filter(f => f !== fn);
      }
      return this;
    },
    destroy() {
      io.disconnect();
      ro.disconnect();
      mo.disconnect();
    }
  };
}

// การใช้งาน
const card = makeObservable("#product-card");

card.on("visibility", ({ isVisible, visibilityRatio }) => {
  console.log(`Product card ${isVisible ? "visible" : "hidden"}: ${(visibilityRatio * 100).toFixed(0)}%`);
});

card.on("resize", ({ width, height }) => {
  console.log(`Card resized to: ${width}x${height}`);
});
```

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: Lazy Load Gallery
สร้าง image gallery ที่:
- Lazy load รูปภาพด้วย IntersectionObserver
- Placeholder blur effect ขณะโหลด
- Support สำหรับ responsive images

### แบบฝึกหัดที่ 2: Infinite Scroll News
สร้าง news feed ที่:
- โหลดข่าวเพิ่มเมื่อ scroll ถึงด้านล่าง
- แสดง loading indicator
- จัดการ error state

### แบบฝึกหัดที่ 3: Live Form Validation Monitor
สร้าง form ที่ใช้ MutationObserver เพื่อ:
- Auto-validate เมื่อ value เปลี่ยน
- Show/hide validation messages
- Track form completion progress

---

## สรุป Part 48

Observer APIs ให้เราสังเกต events ที่ browser จัดการให้อย่างมีประสิทธิภาพ:

1. **IntersectionObserver**: สังเกตว่า element อยู่ใน viewport
   - Lazy loading, Infinite scroll, Animate on scroll
2. **MutationObserver**: สังเกต DOM changes
   - Auto-initialize components, Security monitoring
3. **ResizeObserver**: สังเกตการเปลี่ยนขนาด
   - Responsive charts, Container queries
4. **PerformanceObserver**: สังเกต performance metrics
   - Core Web Vitals, Long Task detection

**ข้อได้เปรียบ**: ทำงานแบบ asynchronous, ไม่ block UI, browser optimize ให้
