# Part 49: Advanced DOM Manipulation

## DOM ขั้นสูงสำหรับ Web Developers

Part นี้ครอบคลุม APIs ขั้นสูงของ DOM ที่ช่วยให้สร้าง web components, animations, และ interactions ที่ซับซ้อนได้อย่างมีประสิทธิภาพ

**Steps 951-970**

---

## Step 951: DocumentFragment สำหรับ Batch Operations

```javascript
// ปัญหา: การแทรก DOM หลายครั้งทำให้เกิด reflow หลายครั้ง

// วิธีที่ช้า (หลาย reflows)
const container = document.getElementById("list");
for (let i = 0; i < 1000; i++) {
  const li = document.createElement("li");
  li.textContent = `รายการ ${i + 1}`;
  container.appendChild(li);  // เกิด reflow ทุกครั้ง!
}

// วิธีที่เร็ว: ใช้ DocumentFragment
const fragment = document.createDocumentFragment();

for (let i = 0; i < 1000; i++) {
  const li = document.createElement("li");
  li.textContent = `รายการ ${i + 1}`;
  fragment.appendChild(li);  // ไม่เกิด reflow
}

// เพิ่มเข้า DOM ครั้งเดียว (เกิด reflow แค่ครั้งเดียว)
container.appendChild(fragment);
```

```javascript
// DocumentFragment ขั้นสูง

function renderTable(data, columns) {
  const tbody = document.createElement("tbody");
  const fragment = document.createDocumentFragment();
  
  data.forEach(row => {
    const tr = document.createElement("tr");
    
    columns.forEach(col => {
      const td = document.createElement("td");
      
      if (col.render) {
        td.innerHTML = col.render(row[col.key], row);
      } else {
        td.textContent = row[col.key] ?? "-";
      }
      
      if (col.className) {
        td.className = col.className;
      }
      
      tr.appendChild(td);
    });
    
    fragment.appendChild(tr);
  });
  
  tbody.appendChild(fragment);
  return tbody;
}

// การใช้งาน
const columns = [
  { key: "name", render: (val) => `<strong>${val}</strong>` },
  { key: "email" },
  { key: "score", className: "number", render: (val) => val.toFixed(2) },
  { key: "status", render: (val, row) => `<span class="badge ${val}">${val}</span>` }
];

const users = [
  { name: "สมชาย", email: "somchai@example.com", score: 95.5, status: "active" },
  { name: "สมหญิง", email: "somying@example.com", score: 82.3, status: "inactive" }
];

const table = document.querySelector("table");
table.appendChild(renderTable(users, columns));
```

```javascript
// Benchmark: Fragment vs Direct DOM
function benchmark() {
  const container1 = document.getElementById("container1");
  const container2 = document.getElementById("container2");
  const n = 5000;
  
  // Direct DOM manipulation
  console.time("direct");
  for (let i = 0; i < n; i++) {
    const div = document.createElement("div");
    div.textContent = i;
    container1.appendChild(div);
  }
  console.timeEnd("direct");
  
  // Fragment approach
  console.time("fragment");
  const frag = document.createDocumentFragment();
  for (let i = 0; i < n; i++) {
    const div = document.createElement("div");
    div.textContent = i;
    frag.appendChild(div);
  }
  container2.appendChild(frag);
  console.timeEnd("fragment");
}
```

---

## Step 952: Virtual DOM Concept

```javascript
// Virtual DOM เป็นแนวคิดที่ React ทำให้โด่งดัง
// ไอเดียหลัก: สร้าง representation ของ DOM ใน memory ก่อน
// แล้ว diff กับ real DOM เพื่อ update เฉพาะส่วนที่เปลี่ยน

// Virtual DOM element
function createElement(type, props = {}, ...children) {
  return {
    type,
    props: { ...props, children: children.flat() }
  };
}

// Diff algorithm (simplified)
function diff(oldVNode, newVNode) {
  if (!oldVNode) {
    return { type: "CREATE", newVNode };
  }
  
  if (!newVNode) {
    return { type: "REMOVE" };
  }
  
  if (typeof oldVNode !== typeof newVNode) {
    return { type: "REPLACE", newVNode };
  }
  
  if (typeof newVNode === "string" || typeof newVNode === "number") {
    if (oldVNode !== newVNode) {
      return { type: "TEXT", content: newVNode };
    }
    return null;
  }
  
  if (oldVNode.type !== newVNode.type) {
    return { type: "REPLACE", newVNode };
  }
  
  // Compare props
  const propPatches = diffProps(oldVNode.props, newVNode.props);
  
  // Compare children
  const childPatches = diffChildren(
    oldVNode.props.children || [],
    newVNode.props.children || []
  );
  
  if (!propPatches && !childPatches) return null;
  
  return { type: "UPDATE", propPatches, childPatches };
}

function diffProps(oldProps = {}, newProps = {}) {
  const patches = {};
  let hasDiff = false;
  
  // ตรวจสอบ props ที่เปลี่ยน
  for (const key of Object.keys(newProps)) {
    if (key !== "children" && oldProps[key] !== newProps[key]) {
      patches[key] = newProps[key];
      hasDiff = true;
    }
  }
  
  // ตรวจสอบ props ที่ถูกลบ
  for (const key of Object.keys(oldProps)) {
    if (key !== "children" && !(key in newProps)) {
      patches[key] = null;
      hasDiff = true;
    }
  }
  
  return hasDiff ? patches : null;
}

function diffChildren(oldChildren, newChildren) {
  const patches = [];
  const max = Math.max(oldChildren.length, newChildren.length);
  
  for (let i = 0; i < max; i++) {
    patches.push(diff(oldChildren[i], newChildren[i]));
  }
  
  return patches.some(p => p !== null) ? patches : null;
}

// Apply patches to real DOM
function patch(domNode, patches) {
  if (!patches) return domNode;
  
  switch (patches.type) {
    case "CREATE":
      const newNode = createDOMNode(patches.newVNode);
      domNode.parentNode?.replaceChild(newNode, domNode);
      return newNode;
      
    case "REMOVE":
      domNode.parentNode?.removeChild(domNode);
      return null;
      
    case "REPLACE":
      const replaced = createDOMNode(patches.newVNode);
      domNode.parentNode?.replaceChild(replaced, domNode);
      return replaced;
      
    case "TEXT":
      domNode.textContent = patches.content;
      return domNode;
      
    case "UPDATE":
      if (patches.propPatches) {
        applyProps(domNode, patches.propPatches);
      }
      if (patches.childPatches) {
        patches.childPatches.forEach((childPatch, i) => {
          patch(domNode.childNodes[i], childPatch);
        });
      }
      return domNode;
  }
}

function createDOMNode(vNode) {
  if (typeof vNode === "string" || typeof vNode === "number") {
    return document.createTextNode(String(vNode));
  }
  
  const el = document.createElement(vNode.type);
  applyProps(el, vNode.props);
  
  (vNode.props.children || []).forEach(child => {
    el.appendChild(createDOMNode(child));
  });
  
  return el;
}

function applyProps(el, props) {
  Object.entries(props).forEach(([key, value]) => {
    if (key === "children") return;
    if (key.startsWith("on")) {
      const event = key.slice(2).toLowerCase();
      el.addEventListener(event, value);
    } else if (key === "className") {
      el.className = value || "";
    } else if (value === null || value === undefined) {
      el.removeAttribute(key);
    } else {
      el.setAttribute(key, value);
    }
  });
}
```

---

## Step 953: Shadow DOM

```javascript
// Shadow DOM ช่วย encapsulate styles และ markup ของ component
// ป้องกัน styles leak เข้าออก component

// สร้าง Shadow DOM
const host = document.getElementById("shadow-host");
const shadow = host.attachShadow({ mode: "open" });
// mode: "open" - accessible จาก outside
// mode: "closed" - ไม่ accessible จาก outside

// เพิ่ม content ลงใน Shadow DOM
shadow.innerHTML = `
  <style>
    /* Styles นี้จะไม่ affect global DOM */
    :host {
      display: block;
      border: 2px solid #007bff;
      padding: 16px;
      border-radius: 8px;
    }
    
    h2 { color: #007bff; }
    
    button {
      background: #007bff;
      color: white;
      border: none;
      padding: 8px 16px;
      border-radius: 4px;
      cursor: pointer;
    }
    
    button:hover {
      background: #0056b3;
    }
  </style>
  
  <h2>Shadow DOM Component</h2>
  <p>Content ใน Shadow DOM ถูก encapsulate</p>
  <button id="shadow-btn">คลิกฉัน</button>
  <slot></slot>
`;

// Access elements ใน Shadow DOM
const shadowBtn = shadow.getElementById("shadow-btn");
shadowBtn.addEventListener("click", () => {
  alert("ปุ่มใน Shadow DOM ถูกกด");
});

// Closed Shadow DOM - ไม่สามารถ access ได้จากภายนอก
const closedHost = document.getElementById("closed-host");
const closedShadow = closedHost.attachShadow({ mode: "closed" });
// closedHost.shadowRoot === null (ไม่สามารถ access ได้)
```

---

## Step 954: Custom Elements (Web Components)

```javascript
// สร้าง Custom Element

// 1. Custom Element แบบ Autonomous (ใหม่ทั้งหมด)
class UserCard extends HTMLElement {
  // กำหนด attributes ที่ต้องการ observe
  static get observedAttributes() {
    return ["name", "email", "avatar", "role"];
  }
  
  constructor() {
    super();
    // สร้าง Shadow DOM
    this.shadow = this.attachShadow({ mode: "open" });
    this.render();
  }
  
  // Lifecycle callbacks
  connectedCallback() {
    console.log("UserCard เพิ่มเข้า DOM");
    this.setupEventListeners();
  }
  
  disconnectedCallback() {
    console.log("UserCard ถูกลบออกจาก DOM");
    this.cleanup();
  }
  
  attributeChangedCallback(name, oldValue, newValue) {
    if (oldValue !== newValue) {
      this.render();
    }
  }
  
  adoptedCallback() {
    console.log("UserCard ย้ายไป document ใหม่");
  }
  
  render() {
    const name = this.getAttribute("name") || "ไม่ระบุชื่อ";
    const email = this.getAttribute("email") || "";
    const avatar = this.getAttribute("avatar") || `https://ui-avatars.com/api/?name=${encodeURIComponent(name)}`;
    const role = this.getAttribute("role") || "User";
    
    this.shadow.innerHTML = `
      <style>
        :host {
          display: inline-block;
          width: 250px;
        }
        
        .card {
          background: white;
          border-radius: 12px;
          padding: 24px;
          box-shadow: 0 4px 20px rgba(0,0,0,0.1);
          text-align: center;
          transition: transform 0.2s;
        }
        
        .card:hover {
          transform: translateY(-4px);
        }
        
        .avatar {
          width: 80px;
          height: 80px;
          border-radius: 50%;
          object-fit: cover;
          border: 3px solid #007bff;
        }
        
        .name {
          font-size: 1.2rem;
          font-weight: bold;
          margin: 12px 0 4px;
          color: #333;
        }
        
        .email {
          color: #666;
          font-size: 0.9rem;
        }
        
        .role {
          display: inline-block;
          background: #007bff;
          color: white;
          padding: 2px 12px;
          border-radius: 12px;
          font-size: 0.8rem;
          margin-top: 8px;
        }
        
        .actions {
          margin-top: 16px;
          display: flex;
          gap: 8px;
          justify-content: center;
        }
        
        button {
          padding: 6px 12px;
          border-radius: 6px;
          border: none;
          cursor: pointer;
          font-size: 0.85rem;
        }
        
        .btn-primary {
          background: #007bff;
          color: white;
        }
        
        .btn-secondary {
          background: #e9ecef;
          color: #333;
        }
      </style>
      
      <div class="card">
        <img class="avatar" src="${avatar}" alt="${name}">
        <div class="name">${name}</div>
        <div class="email">${email}</div>
        <span class="role">${role}</span>
        <div class="actions">
          <button class="btn-primary" id="view-btn">ดูโปรไฟล์</button>
          <button class="btn-secondary" id="msg-btn">ส่งข้อความ</button>
        </div>
        <slot></slot>
      </div>
    `;
  }
  
  setupEventListeners() {
    this.shadow.getElementById("view-btn")?.addEventListener("click", () => {
      this.dispatchEvent(new CustomEvent("viewProfile", {
        bubbles: true,
        composed: true,  // ผ่าน Shadow DOM boundary
        detail: { email: this.getAttribute("email") }
      }));
    });
    
    this.shadow.getElementById("msg-btn")?.addEventListener("click", () => {
      this.dispatchEvent(new CustomEvent("sendMessage", {
        bubbles: true,
        composed: true,
        detail: { email: this.getAttribute("email") }
      }));
    });
  }
  
  cleanup() {
    // Remove event listeners, timers, etc.
  }
}

// ลงทะเบียน Custom Element
customElements.define("user-card", UserCard);
```

```html
<!-- การใช้งานใน HTML -->
<user-card
  name="สมชาย ใจดี"
  email="somchai@example.com"
  role="Admin"
  avatar="https://example.com/avatar.jpg">
  <!-- Slot content -->
  <p>เพิ่มเติม</p>
</user-card>
```

```javascript
// รับ event จาก Custom Element
document.addEventListener("viewProfile", (event) => {
  console.log("View profile:", event.detail.email);
  window.location.href = `/profile/${event.detail.email}`;
});

document.addEventListener("sendMessage", (event) => {
  console.log("Send message to:", event.detail.email);
  openMessageDialog(event.detail.email);
});
```

---

## Step 955: Custom Elements - Built-in Extends

```javascript
// 2. Custom Element แบบ Customized Built-in
// Extends existing HTML elements

class FancyButton extends HTMLButtonElement {
  constructor() {
    super();
    this.classList.add("fancy-button");
    this.setupRipple();
  }
  
  static get observedAttributes() {
    return ["loading", "variant"];
  }
  
  connectedCallback() {
    this.setAttribute("type", this.getAttribute("type") || "button");
  }
  
  attributeChangedCallback(name, oldVal, newVal) {
    if (name === "loading") {
      this.handleLoadingState(newVal !== null);
    } else if (name === "variant") {
      this.updateVariant(newVal);
    }
  }
  
  handleLoadingState(isLoading) {
    if (isLoading) {
      this._originalText = this.textContent;
      this.textContent = "กำลังโหลด...";
      this.disabled = true;
      this.classList.add("loading");
    } else {
      this.textContent = this._originalText || this.textContent;
      this.disabled = false;
      this.classList.remove("loading");
    }
  }
  
  updateVariant(variant) {
    this.className = `fancy-button ${variant ? `fancy-button--${variant}` : ""}`;
  }
  
  setupRipple() {
    this.addEventListener("click", (e) => {
      const ripple = document.createElement("span");
      const rect = this.getBoundingClientRect();
      const size = Math.max(rect.width, rect.height);
      
      ripple.style.cssText = `
        position: absolute;
        border-radius: 50%;
        background: rgba(255,255,255,0.4);
        transform: scale(0);
        animation: ripple 0.6s linear;
        width: ${size}px;
        height: ${size}px;
        left: ${e.clientX - rect.left - size/2}px;
        top: ${e.clientY - rect.top - size/2}px;
      `;
      
      this.style.position = "relative";
      this.style.overflow = "hidden";
      this.appendChild(ripple);
      
      ripple.addEventListener("animationend", () => ripple.remove());
    });
  }
}

customElements.define("fancy-button", FancyButton, { extends: "button" });

// การใช้งาน
// <button is="fancy-button" variant="primary">คลิก</button>
```

---

## Step 956: HTML Template Element

```javascript
// <template> element ไม่แสดงผลแต่เก็บ markup ไว้ใช้ภายหลัง

// HTML
// <template id="card-template">
//   <div class="card">
//     <img src="" alt="" class="card-img">
//     <div class="card-body">
//       <h3 class="card-title"></h3>
//       <p class="card-description"></p>
//       <button class="card-btn">ดูเพิ่มเติม</button>
//     </div>
//   </div>
// </template>

// JavaScript
function createCard(data) {
  const template = document.getElementById("card-template");
  const clone = template.content.cloneNode(true);  // deep clone
  
  clone.querySelector(".card-img").src = data.image;
  clone.querySelector(".card-img").alt = data.title;
  clone.querySelector(".card-title").textContent = data.title;
  clone.querySelector(".card-description").textContent = data.description;
  
  const btn = clone.querySelector(".card-btn");
  btn.addEventListener("click", () => {
    window.location.href = `/product/${data.id}`;
  });
  
  return clone;
}

// สร้างหลาย cards
const products = [
  { id: 1, title: "สินค้า A", description: "คำอธิบาย A", image: "/img/a.jpg" },
  { id: 2, title: "สินค้า B", description: "คำอธิบาย B", image: "/img/b.jpg" }
];

const container = document.getElementById("products");
const fragment = document.createDocumentFragment();

products.forEach(product => {
  fragment.appendChild(createCard(product));
});

container.appendChild(fragment);
```

---

## Step 957: Slot Element

```javascript
// Slot ช่วยให้ใส่ content ลงใน Shadow DOM จากภายนอก

class TabsComponent extends HTMLElement {
  constructor() {
    super();
    this.shadow = this.attachShadow({ mode: "open" });
  }
  
  connectedCallback() {
    this.render();
    this.setupTabs();
  }
  
  render() {
    this.shadow.innerHTML = `
      <style>
        :host { display: block; }
        
        .tabs-header {
          display: flex;
          border-bottom: 2px solid #dee2e6;
          gap: 4px;
        }
        
        .tab-button {
          padding: 8px 16px;
          border: none;
          background: transparent;
          cursor: pointer;
          border-bottom: 2px solid transparent;
          margin-bottom: -2px;
        }
        
        .tab-button.active {
          border-bottom-color: #007bff;
          color: #007bff;
          font-weight: bold;
        }
        
        .tab-panels {
          padding: 16px 0;
        }
        
        ::slotted([slot]) {
          display: none;
        }
        
        ::slotted([slot].active) {
          display: block;
        }
      </style>
      
      <div class="tabs-header">
        <!-- Tab buttons จะถูกสร้างโดย JavaScript -->
      </div>
      
      <div class="tab-panels">
        <slot></slot>
      </div>
    `;
  }
  
  setupTabs() {
    const tabs = this.querySelectorAll("[slot]");
    const header = this.shadow.querySelector(".tabs-header");
    
    tabs.forEach((tab, index) => {
      const btn = document.createElement("button");
      btn.className = "tab-button" + (index === 0 ? " active" : "");
      btn.textContent = tab.getAttribute("slot");
      
      btn.addEventListener("click", () => {
        this.activateTab(tab.getAttribute("slot"), btn);
      });
      
      header.appendChild(btn);
    });
    
    // Activate first tab
    if (tabs.length > 0) {
      tabs[0].classList.add("active");
    }
  }
  
  activateTab(slotName, clickedBtn) {
    // Deactivate all
    this.querySelectorAll("[slot]").forEach(tab => tab.classList.remove("active"));
    this.shadow.querySelectorAll(".tab-button").forEach(btn => btn.classList.remove("active"));
    
    // Activate selected
    this.querySelector(`[slot="${slotName}"]`)?.classList.add("active");
    clickedBtn.classList.add("active");
  }
}

customElements.define("tabs-component", TabsComponent);
```

```html
<!-- การใช้งาน -->
<tabs-component>
  <div slot="ข้อมูลทั่วไป">
    <h2>ข้อมูลทั่วไป</h2>
    <p>เนื้อหาสำหรับแท็บแรก</p>
  </div>
  
  <div slot="การตั้งค่า">
    <h2>การตั้งค่า</h2>
    <form>...</form>
  </div>
  
  <div slot="ประวัติ">
    <h2>ประวัติ</h2>
    <ul>...</ul>
  </div>
</tabs-component>
```

---

## Step 958: Element.animate() - Web Animations API

```javascript
// Web Animations API ให้ประสิทธิภาพดีกว่า CSS animations ในบางกรณี

const element = document.getElementById("box");

// animate() syntax
const animation = element.animate(
  // Keyframes (array หรือ object)
  [
    { transform: "translateX(0)", opacity: 1 },
    { transform: "translateX(200px)", opacity: 0.5, offset: 0.7 },
    { transform: "translateX(300px)", opacity: 0 }
  ],
  // Timing options
  {
    duration: 1000,      // milliseconds
    delay: 500,          // delay ก่อนเริ่ม
    endDelay: 200,       // delay หลังจบ
    fill: "forwards",    // none, forwards, backwards, both
    iterations: 3,       // จำนวนรอบ (Infinity = loop)
    direction: "alternate",  // normal, reverse, alternate, alternate-reverse
    easing: "ease-in-out",   // CSS easing หรือ cubic-bezier
    composite: "replace"      // replace, add, accumulate
  }
);

// Control the animation
animation.pause();
animation.play();
animation.reverse();
animation.cancel();
animation.finish();

// Events
animation.addEventListener("finish", () => {
  console.log("Animation เสร็จแล้ว");
});

animation.addEventListener("cancel", () => {
  console.log("Animation ถูกยกเลิก");
});

// Properties
console.log("Current time:", animation.currentTime);
console.log("Play state:", animation.playState); // idle, running, paused, finished
console.log("Effect:", animation.effect);

// Promise-based
animation.finished.then(() => {
  element.style.display = "none";
});
```

```javascript
// ตัวอย่างที่ใช้งานจริง

// เปิด Modal ด้วย animation
function openModal(modalId) {
  const modal = document.getElementById(modalId);
  modal.style.display = "flex";
  
  modal.animate(
    [
      { opacity: 0, transform: "scale(0.9)" },
      { opacity: 1, transform: "scale(1)" }
    ],
    { duration: 200, easing: "ease-out", fill: "forwards" }
  );
}

async function closeModal(modalId) {
  const modal = document.getElementById(modalId);
  
  const animation = modal.animate(
    [
      { opacity: 1, transform: "scale(1)" },
      { opacity: 0, transform: "scale(0.9)" }
    ],
    { duration: 150, easing: "ease-in", fill: "forwards" }
  );
  
  await animation.finished;
  modal.style.display = "none";
}

// Card flip animation
function flipCard(card) {
  const isFlipped = card.dataset.flipped === "true";
  const front = card.querySelector(".front");
  const back = card.querySelector(".back");
  
  if (!isFlipped) {
    front.animate(
      [{ transform: "rotateY(0deg)" }, { transform: "rotateY(-90deg)" }],
      { duration: 200, fill: "forwards" }
    ).finished.then(() => {
      front.style.display = "none";
      back.style.display = "block";
      back.animate(
        [{ transform: "rotateY(90deg)" }, { transform: "rotateY(0deg)" }],
        { duration: 200, fill: "forwards" }
      );
    });
    card.dataset.flipped = "true";
  } else {
    back.animate(
      [{ transform: "rotateY(0deg)" }, { transform: "rotateY(90deg)" }],
      { duration: 200, fill: "forwards" }
    ).finished.then(() => {
      back.style.display = "none";
      front.style.display = "block";
      front.animate(
        [{ transform: "rotateY(-90deg)" }, { transform: "rotateY(0deg)" }],
        { duration: 200, fill: "forwards" }
      );
    });
    card.dataset.flipped = "false";
  }
}
```

---

## Step 959: FLIP Animation Technique

```javascript
// FLIP = First, Last, Invert, Play
// เทคนิคสำหรับทำ animation ที่ smooth มากๆ

class FLIPAnimation {
  constructor(elements) {
    this.elements = typeof elements === "string"
      ? document.querySelectorAll(elements)
      : elements;
  }
  
  // Step 1: First - บันทึก position ปัจจุบัน
  record() {
    this.firstPositions = new Map();
    this.elements.forEach(el => {
      const rect = el.getBoundingClientRect();
      this.firstPositions.set(el, {
        x: rect.left,
        y: rect.top,
        width: rect.width,
        height: rect.height
      });
    });
    return this;
  }
  
  // Step 2: Last - เปลี่ยน DOM ให้เป็น state ใหม่
  // (เรียก DOM mutation หรือ CSS change ข้างนอก)
  
  // Step 3 & 4: Invert & Play
  animate(duration = 300, easing = "ease-in-out") {
    this.elements.forEach(el => {
      const first = this.firstPositions.get(el);
      if (!first) return;
      
      // Step 3: Invert - หา offset จาก last position ไปยัง first
      const last = el.getBoundingClientRect();
      const deltaX = first.x - last.left;
      const deltaY = first.y - last.top;
      const scaleX = first.width / last.width;
      const scaleY = first.height / last.height;
      
      // ถ้าไม่มีการเปลี่ยน ข้ามไป
      if (!deltaX && !deltaY && scaleX === 1 && scaleY === 1) return;
      
      // Step 4: Play - animate จาก inverted state ไปยัง final state
      el.animate(
        [
          {
            transform: `translate(${deltaX}px, ${deltaY}px) scale(${scaleX}, ${scaleY})`,
            transformOrigin: "top left"
          },
          { transform: "none" }
        ],
        {
          duration,
          easing,
          fill: "none"
        }
      );
    });
    
    return this;
  }
}

// ตัวอย่าง: Grid reorder animation
function reorderGrid() {
  const items = document.querySelectorAll(".grid-item");
  
  // Record first positions
  const flip = new FLIPAnimation(items);
  flip.record();
  
  // Change DOM (shuffle)
  const container = document.querySelector(".grid");
  const arr = [...items];
  arr.sort(() => Math.random() - 0.5);
  arr.forEach(item => container.appendChild(item));
  
  // Animate from old to new positions
  flip.animate(400, "ease-in-out");
}

// ตัวอย่าง: List sort animation
function sortList(listId) {
  const list = document.getElementById(listId);
  const items = [...list.querySelectorAll("li")];
  
  const flip = new FLIPAnimation(items);
  flip.record();
  
  // Sort items
  items.sort((a, b) => a.textContent.localeCompare(b.textContent, "th"));
  items.forEach(item => list.appendChild(item));
  
  flip.animate(300);
}
```

---

## Step 960: Drag and Drop API

```javascript
// HTML5 Drag and Drop API

// HTML
// <div draggable="true" id="drag-item">ลาก</div>
// <div id="drop-zone">วางที่นี่</div>

// Drag source
const dragItem = document.getElementById("drag-item");

dragItem.addEventListener("dragstart", (e) => {
  console.log("เริ่มลาก");
  e.dataTransfer.setData("text/plain", e.target.id);
  e.dataTransfer.setData("application/json", JSON.stringify({ id: "item-1", type: "card" }));
  e.dataTransfer.effectAllowed = "copyMove";
  
  // Custom drag image
  const ghost = e.target.cloneNode(true);
  ghost.style.opacity = "0.5";
  document.body.appendChild(ghost);
  e.dataTransfer.setDragImage(ghost, 0, 0);
  setTimeout(() => ghost.remove(), 0);
  
  e.target.classList.add("dragging");
});

dragItem.addEventListener("drag", (e) => {
  // กำลังลากอยู่
});

dragItem.addEventListener("dragend", (e) => {
  console.log("หยุดลาก");
  e.target.classList.remove("dragging");
  
  if (e.dataTransfer.dropEffect === "move") {
    e.target.remove();
  }
});

// Drop zone
const dropZone = document.getElementById("drop-zone");

dropZone.addEventListener("dragover", (e) => {
  e.preventDefault();  // ต้อง prevent default เพื่อให้ drop ได้
  e.dataTransfer.dropEffect = "move";
  dropZone.classList.add("drag-over");
});

dropZone.addEventListener("dragenter", (e) => {
  e.preventDefault();
  dropZone.classList.add("drag-over");
});

dropZone.addEventListener("dragleave", (e) => {
  // ตรวจสอบว่าออกจาก zone จริงๆ ไม่ใช่แค่เข้า child element
  if (!dropZone.contains(e.relatedTarget)) {
    dropZone.classList.remove("drag-over");
  }
});

dropZone.addEventListener("drop", (e) => {
  e.preventDefault();
  dropZone.classList.remove("drag-over");
  
  const id = e.dataTransfer.getData("text/plain");
  const data = JSON.parse(e.dataTransfer.getData("application/json") || "{}");
  
  console.log("วาง:", id, data);
  
  // Handle files
  const files = e.dataTransfer.files;
  if (files.length > 0) {
    handleDroppedFiles(files);
  }
  
  // Move element
  const draggedEl = document.getElementById(id);
  if (draggedEl) {
    dropZone.appendChild(draggedEl);
  }
});
```

```javascript
// Sortable List ด้วย Drag and Drop

class SortableList {
  constructor(listElement) {
    this.list = listElement;
    this.dragging = null;
    this.placeholder = this.createPlaceholder();
    
    this.setupItems();
  }
  
  createPlaceholder() {
    const el = document.createElement("li");
    el.className = "sortable-placeholder";
    el.style.cssText = `
      height: 50px;
      background: rgba(0,123,255,0.1);
      border: 2px dashed #007bff;
      border-radius: 8px;
      list-style: none;
    `;
    return el;
  }
  
  setupItems() {
    this.list.querySelectorAll("li").forEach(item => {
      this.makeItemDraggable(item);
    });
    
    // Observe for new items
    const observer = new MutationObserver((mutations) => {
      mutations.forEach(mutation => {
        mutation.addedNodes.forEach(node => {
          if (node.tagName === "LI") {
            this.makeItemDraggable(node);
          }
        });
      });
    });
    
    observer.observe(this.list, { childList: true });
  }
  
  makeItemDraggable(item) {
    item.draggable = true;
    
    item.addEventListener("dragstart", (e) => {
      this.dragging = item;
      item.classList.add("sortable-dragging");
      e.dataTransfer.effectAllowed = "move";
    });
    
    item.addEventListener("dragend", () => {
      item.classList.remove("sortable-dragging");
      this.placeholder.remove();
      this.dragging = null;
    });
    
    item.addEventListener("dragover", (e) => {
      e.preventDefault();
      if (item === this.dragging) return;
      
      const rect = item.getBoundingClientRect();
      const midY = rect.top + rect.height / 2;
      
      if (e.clientY < midY) {
        item.parentNode.insertBefore(this.placeholder, item);
      } else {
        item.parentNode.insertBefore(this.placeholder, item.nextSibling);
      }
    });
  }
  
  setupDropZone() {
    this.list.addEventListener("drop", (e) => {
      e.preventDefault();
      if (!this.dragging) return;
      
      this.placeholder.parentNode?.insertBefore(this.dragging, this.placeholder);
      this.placeholder.remove();
      
      // Dispatch order changed event
      this.list.dispatchEvent(new CustomEvent("orderChanged", {
        detail: { items: [...this.list.querySelectorAll("li")] }
      }));
    });
    
    this.list.addEventListener("dragover", (e) => {
      e.preventDefault();
    });
  }
}
```

---

## Step 961: Clipboard Events

```javascript
// Clipboard API - อ่านและเขียน clipboard

// Modern Clipboard API (async)
async function copyToClipboard(text) {
  try {
    await navigator.clipboard.writeText(text);
    console.log("คัดลอกสำเร็จ:", text);
  } catch (err) {
    console.error("คัดลอกไม่สำเร็จ:", err);
  }
}

async function readFromClipboard() {
  try {
    const text = await navigator.clipboard.readText();
    console.log("Clipboard:", text);
    return text;
  } catch (err) {
    console.error("อ่าน clipboard ไม่สำเร็จ:", err);
    return null;
  }
}

// Copy rich content (HTML, images)
async function copyHTML(htmlString) {
  const type = "text/html";
  const blob = new Blob([htmlString], { type });
  const data = [new ClipboardItem({ [type]: blob })];
  
  try {
    await navigator.clipboard.write(data);
    console.log("Copy HTML สำเร็จ");
  } catch (err) {
    console.error("Copy ไม่สำเร็จ:", err);
  }
}

// Clipboard events
document.addEventListener("copy", (event) => {
  const selection = window.getSelection();
  if (!selection.toString()) return;
  
  // เพิ่มข้อความ credit เมื่อ copy
  const copied = selection.toString();
  const creditText = `\n\n-- คัดลอกจาก ${window.location.href}`;
  
  event.clipboardData.setData("text/plain", copied + creditText);
  event.preventDefault();
  
  console.log("User copied:", copied);
});

document.addEventListener("cut", (event) => {
  const selection = window.getSelection();
  console.log("User cut:", selection.toString());
});

document.addEventListener("paste", async (event) => {
  const text = event.clipboardData.getData("text/plain");
  const html = event.clipboardData.getData("text/html");
  const files = event.clipboardData.files;
  
  console.log("Paste text:", text);
  console.log("Paste HTML:", html);
  
  if (files.length > 0) {
    // Handle pasted images
    for (const file of files) {
      if (file.type.startsWith("image/")) {
        const url = URL.createObjectURL(file);
        const img = document.createElement("img");
        img.src = url;
        document.getElementById("paste-area").appendChild(img);
      }
    }
  }
});
```

---

## Step 962: Context Menu Event

```javascript
// Custom Context Menu

class ContextMenu {
  constructor(options) {
    this.menu = null;
    this.items = options.items || [];
    this.target = null;
    
    this.setup();
  }
  
  setup() {
    document.addEventListener("contextmenu", this.handleContextMenu.bind(this));
    document.addEventListener("click", this.hideMenu.bind(this));
    document.addEventListener("keydown", (e) => {
      if (e.key === "Escape") this.hideMenu();
    });
    
    window.addEventListener("scroll", this.hideMenu.bind(this));
  }
  
  handleContextMenu(event) {
    event.preventDefault();
    
    this.target = event.target;
    
    // หา items สำหรับ target นี้
    const contextItems = this.getItemsForTarget(this.target);
    if (!contextItems.length) return;
    
    this.showMenu(event.clientX, event.clientY, contextItems);
  }
  
  getItemsForTarget(target) {
    return this.items.filter(item => {
      if (item.selector) {
        return target.closest(item.selector);
      }
      return true;
    });
  }
  
  showMenu(x, y, items) {
    this.hideMenu();
    
    this.menu = document.createElement("div");
    this.menu.className = "context-menu";
    this.menu.style.cssText = `
      position: fixed;
      left: ${x}px;
      top: ${y}px;
      background: white;
      border: 1px solid #ccc;
      border-radius: 8px;
      box-shadow: 0 4px 20px rgba(0,0,0,0.15);
      z-index: 9999;
      min-width: 180px;
      overflow: hidden;
    `;
    
    items.forEach(item => {
      if (item.separator) {
        const sep = document.createElement("hr");
        sep.style.cssText = "margin: 4px 0; border-color: #eee;";
        this.menu.appendChild(sep);
        return;
      }
      
      const menuItem = document.createElement("div");
      menuItem.className = "context-menu-item";
      menuItem.style.cssText = `
        padding: 10px 16px;
        cursor: pointer;
        display: flex;
        align-items: center;
        gap: 8px;
        font-size: 14px;
        color: ${item.danger ? "#dc3545" : "#333"};
      `;
      
      if (item.icon) {
        const icon = document.createElement("span");
        icon.textContent = item.icon;
        menuItem.appendChild(icon);
      }
      
      const text = document.createElement("span");
      text.textContent = item.label;
      menuItem.appendChild(text);
      
      if (item.shortcut) {
        const shortcut = document.createElement("span");
        shortcut.textContent = item.shortcut;
        shortcut.style.cssText = "margin-left: auto; color: #999; font-size: 12px;";
        menuItem.appendChild(shortcut);
      }
      
      menuItem.addEventListener("mouseenter", () => {
        menuItem.style.background = "#f8f9fa";
      });
      
      menuItem.addEventListener("mouseleave", () => {
        menuItem.style.background = "transparent";
      });
      
      menuItem.addEventListener("click", () => {
        item.action(this.target, item);
        this.hideMenu();
      });
      
      if (item.disabled) {
        menuItem.style.opacity = "0.5";
        menuItem.style.pointerEvents = "none";
      }
      
      this.menu.appendChild(menuItem);
    });
    
    document.body.appendChild(this.menu);
    
    // Adjust position ถ้าออกนอก viewport
    const rect = this.menu.getBoundingClientRect();
    if (rect.right > window.innerWidth) {
      this.menu.style.left = `${window.innerWidth - rect.width - 8}px`;
    }
    if (rect.bottom > window.innerHeight) {
      this.menu.style.top = `${window.innerHeight - rect.height - 8}px`;
    }
  }
  
  hideMenu() {
    this.menu?.remove();
    this.menu = null;
  }
}

// การใช้งาน
const contextMenu = new ContextMenu({
  items: [
    {
      icon: "✏️",
      label: "แก้ไข",
      shortcut: "Ctrl+E",
      selector: ".editable",
      action: (target) => {
        target.contentEditable = "true";
        target.focus();
      }
    },
    {
      icon: "📋",
      label: "คัดลอก",
      shortcut: "Ctrl+C",
      action: async (target) => {
        await navigator.clipboard.writeText(target.textContent);
        alert("คัดลอกแล้ว!");
      }
    },
    { separator: true },
    {
      icon: "🗑️",
      label: "ลบ",
      shortcut: "Del",
      selector: ".deletable",
      danger: true,
      action: (target) => {
        if (confirm("ต้องการลบ?")) target.remove();
      }
    }
  ]
});
```

---

## Step 963: Selection API

```javascript
// Selection API - จัดการ text selection

// รับ current selection
const selection = window.getSelection();

if (selection.rangeCount > 0) {
  const range = selection.getRangeAt(0);
  const text = selection.toString();
  
  console.log("Selected text:", text);
  console.log("Start container:", range.startContainer);
  console.log("End container:", range.endContainer);
  console.log("Start offset:", range.startOffset);
  console.log("End offset:", range.endOffset);
  console.log("Collapsed:", range.collapsed);
}

// เลือก text ด้วย JavaScript
function selectElementText(element) {
  const selection = window.getSelection();
  selection.removeAllRanges();
  
  const range = document.createRange();
  range.selectNodeContents(element);
  selection.addRange(range);
}

// เลือก text บางส่วน
function selectText(element, start, end) {
  const selection = window.getSelection();
  selection.removeAllRanges();
  
  const textNode = element.firstChild;
  if (!textNode) return;
  
  const range = document.createRange();
  range.setStart(textNode, start);
  range.setEnd(textNode, end);
  selection.addRange(range);
}

// Highlight text ที่ถูก select
function highlightSelection(color = "yellow") {
  const selection = window.getSelection();
  if (!selection.rangeCount) return;
  
  const range = selection.getRangeAt(0);
  
  const highlight = document.createElement("mark");
  highlight.style.backgroundColor = color;
  
  try {
    range.surroundContents(highlight);
  } catch (e) {
    // range crosses element boundaries
    const fragment = range.extractContents();
    highlight.appendChild(fragment);
    range.insertNode(highlight);
  }
  
  selection.removeAllRanges();
}

// Popup toolbar เมื่อ select text
document.addEventListener("mouseup", (event) => {
  const selection = window.getSelection();
  const text = selection.toString().trim();
  
  if (!text) {
    document.getElementById("selection-toolbar")?.remove();
    return;
  }
  
  const toolbar = document.createElement("div");
  toolbar.id = "selection-toolbar";
  toolbar.style.cssText = `
    position: fixed;
    left: ${event.clientX - 75}px;
    top: ${event.clientY - 50}px;
    background: #333;
    color: white;
    padding: 4px;
    border-radius: 4px;
    display: flex;
    gap: 4px;
    z-index: 9999;
  `;
  
  const actions = [
    { icon: "📋", action: () => navigator.clipboard.writeText(text) },
    { icon: "🔍", action: () => window.open(`https://www.google.com/search?q=${encodeURIComponent(text)}`) },
    { icon: "⭐", action: () => highlightSelection("#ffff00") }
  ];
  
  actions.forEach(({ icon, action }) => {
    const btn = document.createElement("button");
    btn.textContent = icon;
    btn.style.cssText = "background: transparent; border: none; cursor: pointer; font-size: 16px;";
    btn.onclick = (e) => { e.stopPropagation(); action(); toolbar.remove(); };
    toolbar.appendChild(btn);
  });
  
  document.getElementById("selection-toolbar")?.remove();
  document.body.appendChild(toolbar);
});
```

---

## Step 964: Range API

```javascript
// Range API - จัดการ document ranges

// สร้าง Range
const range = document.createRange();

// กำหนด start และ end points
const startElement = document.getElementById("start-point");
const endElement = document.getElementById("end-point");

range.setStartBefore(startElement);
range.setEndAfter(endElement);

// Methods ที่มีประโยชน์
const rect = range.getBoundingClientRect();  // ขนาดของ range
const rects = range.getClientRects();        // ทุก rect ของ range
const text = range.toString();               // text ภายใน range
const fragment = range.cloneContents();      // copy content

// แทรก node เข้า range
const em = document.createElement("em");
em.textContent = "เน้นข้อความนี้";
range.insertNode(em);

// Wrap range ด้วย element
function wrapRange(range, element) {
  const fragment = range.extractContents();
  element.appendChild(fragment);
  range.insertNode(element);
}

// ตัวอย่าง: Text Annotation
class TextAnnotator {
  constructor(container) {
    this.container = container;
    this.annotations = [];
    this.setupSelection();
  }
  
  setupSelection() {
    this.container.addEventListener("mouseup", () => {
      const selection = window.getSelection();
      if (!selection.toString().trim()) return;
      
      const range = selection.getRangeAt(0);
      this.showAnnotationMenu(range);
    });
  }
  
  showAnnotationMenu(range) {
    const rect = range.getBoundingClientRect();
    
    const menu = document.createElement("div");
    menu.className = "annotation-menu";
    menu.style.cssText = `
      position: fixed;
      left: ${rect.left + rect.width / 2 - 75}px;
      top: ${rect.top - 40}px;
      background: white;
      border: 1px solid #ccc;
      border-radius: 4px;
      padding: 4px 8px;
      display: flex;
      gap: 4px;
      box-shadow: 0 2px 8px rgba(0,0,0,0.15);
      z-index: 999;
    `;
    
    const colors = ["#FFFF00", "#90EE90", "#FFB6C1", "#87CEEB"];
    colors.forEach(color => {
      const btn = document.createElement("button");
      btn.style.cssText = `
        width: 20px; height: 20px;
        background: ${color};
        border: 1px solid #999;
        border-radius: 50%;
        cursor: pointer;
      `;
      btn.onclick = () => {
        this.annotate(range, color);
        menu.remove();
      };
      menu.appendChild(btn);
    });
    
    document.body.appendChild(menu);
    
    setTimeout(() => {
      document.addEventListener("click", () => menu.remove(), { once: true });
    }, 0);
  }
  
  annotate(range, color) {
    const annotation = {
      id: Date.now(),
      text: range.toString(),
      color,
      timestamp: new Date()
    };
    
    const highlight = document.createElement("mark");
    highlight.dataset.annotationId = annotation.id;
    highlight.style.backgroundColor = color;
    highlight.title = `Annotated: ${annotation.timestamp.toLocaleString()}`;
    
    try {
      range.surroundContents(highlight);
      this.annotations.push(annotation);
      window.getSelection().removeAllRanges();
    } catch (e) {
      console.error("ไม่สามารถ annotate ได้:", e);
    }
  }
  
  removeAnnotation(id) {
    const el = document.querySelector(`[data-annotation-id="${id}"]`);
    if (el) {
      const parent = el.parentNode;
      while (el.firstChild) {
        parent.insertBefore(el.firstChild, el);
      }
      el.remove();
    }
    this.annotations = this.annotations.filter(a => a.id !== id);
  }
}
```

---

## Step 965: TreeWalker

```javascript
// TreeWalker ช่วย traverse DOM tree อย่างมีประสิทธิภาพ

// สร้าง TreeWalker
const walker = document.createTreeWalker(
  document.body,              // root node
  NodeFilter.SHOW_TEXT,       // filter: เฉพาะ text nodes
  {
    acceptNode: (node) => {
      // custom filter
      if (node.textContent.trim() === "") {
        return NodeFilter.FILTER_REJECT;
      }
      return NodeFilter.FILTER_ACCEPT;
    }
  }
);

// Traverse nodes
const textNodes = [];
let currentNode = walker.nextNode();

while (currentNode) {
  textNodes.push(currentNode);
  currentNode = walker.nextNode();
}

console.log(`พบ ${textNodes.length} text nodes`);
```

```javascript
// ตัวอย่าง: Find and highlight text

function findAndHighlight(searchText, container = document.body) {
  const regex = new RegExp(searchText, "gi");
  const walker = document.createTreeWalker(
    container,
    NodeFilter.SHOW_TEXT,
    null
  );
  
  const matches = [];
  let node;
  
  while ((node = walker.nextNode())) {
    const text = node.textContent;
    if (regex.test(text)) {
      matches.push(node);
    }
  }
  
  // Highlight matches
  matches.forEach(textNode => {
    const parent = textNode.parentNode;
    if (!parent || parent.tagName === "SCRIPT" || parent.tagName === "STYLE") return;
    
    const html = textNode.textContent.replace(regex, match =>
      `<mark class="search-highlight">${match}</mark>`
    );
    
    const wrapper = document.createElement("span");
    wrapper.innerHTML = html;
    parent.replaceChild(wrapper, textNode);
  });
  
  return document.querySelectorAll(".search-highlight").length;
}

// ตัวอย่าง: Count words
function countWords(element) {
  const walker = document.createTreeWalker(
    element,
    NodeFilter.SHOW_TEXT
  );
  
  let count = 0;
  let node;
  
  while ((node = walker.nextNode())) {
    const words = node.textContent.trim().split(/\s+/);
    count += words.filter(w => w.length > 0).length;
  }
  
  return count;
}

// ตัวอย่าง: Extract all links with context
function extractLinksWithContext(container = document.body) {
  const results = [];
  const walker = document.createTreeWalker(
    container,
    NodeFilter.SHOW_ELEMENT,
    {
      acceptNode: (node) => {
        return node.tagName === "A"
          ? NodeFilter.FILTER_ACCEPT
          : NodeFilter.FILTER_SKIP;
      }
    }
  );
  
  let node;
  while ((node = walker.nextNode())) {
    const context = node.closest("p, li, td, th, div")?.textContent.trim().slice(0, 100);
    results.push({
      href: node.href,
      text: node.textContent.trim(),
      context
    });
  }
  
  return results;
}
```

---

## Step 966: NodeIterator

```javascript
// NodeIterator คล้าย TreeWalker แต่ simpler

const iterator = document.createNodeIterator(
  document.body,
  NodeFilter.SHOW_ELEMENT | NodeFilter.SHOW_TEXT,
  {
    acceptNode: (node) => {
      if (node.nodeType === Node.TEXT_NODE) {
        return node.textContent.trim()
          ? NodeFilter.FILTER_ACCEPT
          : NodeFilter.FILTER_REJECT;
      }
      return NodeFilter.FILTER_ACCEPT;
    }
  }
);

// Forward traversal
let node = iterator.nextNode();
while (node) {
  console.log(node.nodeName, node.nodeType);
  node = iterator.nextNode();
}

// Backward traversal
node = iterator.previousNode();
while (node) {
  console.log(node.nodeName);
  node = iterator.previousNode();
}

// Difference from TreeWalker:
// - NodeIterator ไม่มี parent/firstChild/lastChild/nextSibling/previousSibling
// - NodeIterator ไม่รับ NodeFilter.FILTER_REJECT สำหรับ elements (ใช้ FILTER_SKIP แทน)
// - TreeWalker ให้ navigate in-place, NodeIterator ให้ flat list

// ตัวอย่าง: Remove empty nodes
function removeEmptyNodes(container) {
  const iterator = document.createNodeIterator(
    container,
    NodeFilter.SHOW_ELEMENT
  );
  
  const emptyNodes = [];
  let node;
  
  while ((node = iterator.nextNode())) {
    if (
      node !== container &&
      !node.hasChildNodes() &&
      node.textContent.trim() === ""
    ) {
      emptyNodes.push(node);
    }
  }
  
  emptyNodes.forEach(node => node.remove());
  return emptyNodes.length;
}
```

---

## Step 967: XPath ใน JavaScript

```javascript
// XPath ช่วย query DOM ด้วย path expressions

// ใช้ document.evaluate()
function xpathQuery(xpath, context = document, type = XPathResult.ORDERED_NODE_SNAPSHOT_TYPE) {
  return document.evaluate(
    xpath,
    context,
    null,           // namespace resolver
    type,
    null            // reuse existing result
  );
}

// XPath examples
// หา element ทั้งหมดที่มี class "highlight"
const result = xpathQuery("//*[contains(@class, 'highlight')]");
const nodes = [];
for (let i = 0; i < result.snapshotLength; i++) {
  nodes.push(result.snapshotItem(i));
}
console.log("Found:", nodes.length, "elements");

// หา paragraphs ที่มีข้อความ "JavaScript"
const jsParas = xpathQuery("//p[contains(text(), 'JavaScript')]");

// หา input ที่ required
const requiredInputs = xpathQuery("//input[@required]");

// หา elements ที่ไม่มี children
const leafNodes = xpathQuery("//*[not(*) and normalize-space(text())]");

// XPath สำหรับ Tables
const tableData = xpathQuery("//table//tr[position() > 1]//td[2]");

// Helper function สำหรับ XPath
function findByXPath(xpath, context = document) {
  const result = document.evaluate(
    xpath,
    context,
    null,
    XPathResult.ORDERED_NODE_SNAPSHOT_TYPE,
    null
  );
  
  const nodes = [];
  for (let i = 0; i < result.snapshotLength; i++) {
    nodes.push(result.snapshotItem(i));
  }
  return nodes;
}

function findFirstByXPath(xpath, context = document) {
  const result = document.evaluate(
    xpath,
    context,
    null,
    XPathResult.FIRST_ORDERED_NODE_TYPE,
    null
  );
  return result.singleNodeValue;
}

// ตัวอย่างใช้งาน
const allLinks = findByXPath("//a[@href and not(@href='#')]");
const navLinks = findByXPath("//nav//a");
const firstH2 = findFirstByXPath("//h2");
```

---

## Step 968: Web Components - Complete Example

```javascript
// สร้าง Dropdown Component ที่ใช้งานได้จริง

class SmartDropdown extends HTMLElement {
  static get observedAttributes() {
    return ["placeholder", "value", "disabled", "multiple"];
  }
  
  constructor() {
    super();
    this.shadow = this.attachShadow({ mode: "open" });
    this._options = [];
    this._selectedValues = new Set();
    this._isOpen = false;
  }
  
  connectedCallback() {
    this.parseOptions();
    this.render();
    this.setupEvents();
  }
  
  parseOptions() {
    const optionElements = this.querySelectorAll("option");
    this._options = Array.from(optionElements).map(opt => ({
      value: opt.value,
      label: opt.textContent.trim(),
      disabled: opt.disabled,
      group: opt.parentElement.tagName === "OPTGROUP" 
        ? opt.parentElement.getAttribute("label") 
        : null
    }));
    
    // Set default values
    const selectedOptions = this.querySelectorAll("option[selected]");
    selectedOptions.forEach(opt => this._selectedValues.add(opt.value));
  }
  
  render() {
    const placeholder = this.getAttribute("placeholder") || "เลือก...";
    const isMultiple = this.hasAttribute("multiple");
    const isDisabled = this.hasAttribute("disabled");
    
    const selectedLabels = this._options
      .filter(opt => this._selectedValues.has(opt.value))
      .map(opt => opt.label);
    
    const displayText = selectedLabels.length > 0
      ? isMultiple 
        ? `${selectedLabels.length} รายการที่เลือก`
        : selectedLabels[0]
      : placeholder;
    
    this.shadow.innerHTML = `
      <style>
        :host {
          display: block;
          position: relative;
          font-family: inherit;
        }
        
        .trigger {
          width: 100%;
          padding: 8px 36px 8px 12px;
          border: 1px solid #ced4da;
          border-radius: 6px;
          background: white;
          cursor: pointer;
          display: flex;
          align-items: center;
          justify-content: space-between;
          font-size: 14px;
          transition: border-color 0.2s;
          box-sizing: border-box;
        }
        
        .trigger:hover:not(.disabled) {
          border-color: #007bff;
        }
        
        .trigger.open {
          border-color: #007bff;
          box-shadow: 0 0 0 3px rgba(0,123,255,0.15);
        }
        
        .trigger.disabled {
          background: #f8f9fa;
          cursor: not-allowed;
          color: #6c757d;
        }
        
        .arrow {
          transition: transform 0.2s;
        }
        
        .trigger.open .arrow {
          transform: rotate(180deg);
        }
        
        .dropdown {
          position: absolute;
          top: calc(100% + 4px);
          left: 0;
          right: 0;
          background: white;
          border: 1px solid #ced4da;
          border-radius: 6px;
          box-shadow: 0 4px 20px rgba(0,0,0,0.1);
          z-index: 1000;
          max-height: 250px;
          overflow-y: auto;
          display: none;
        }
        
        .dropdown.open {
          display: block;
        }
        
        .search-input {
          width: 100%;
          padding: 8px 12px;
          border: none;
          border-bottom: 1px solid #eee;
          outline: none;
          font-size: 14px;
          box-sizing: border-box;
        }
        
        .option {
          padding: 8px 12px;
          cursor: pointer;
          display: flex;
          align-items: center;
          gap: 8px;
        }
        
        .option:hover:not(.disabled) {
          background: #f8f9fa;
        }
        
        .option.selected {
          color: #007bff;
          font-weight: 600;
        }
        
        .option.disabled {
          color: #adb5bd;
          cursor: not-allowed;
        }
        
        .checkbox {
          width: 16px;
          height: 16px;
          border: 2px solid #ced4da;
          border-radius: 3px;
          display: flex;
          align-items: center;
          justify-content: center;
          flex-shrink: 0;
        }
        
        .option.selected .checkbox {
          background: #007bff;
          border-color: #007bff;
        }
        
        .group-label {
          padding: 4px 12px;
          font-size: 11px;
          text-transform: uppercase;
          color: #6c757d;
          font-weight: 600;
          background: #f8f9fa;
        }
        
        .no-results {
          padding: 12px;
          text-align: center;
          color: #6c757d;
          font-style: italic;
        }
      </style>
      
      <div class="trigger ${this._isOpen ? "open" : ""} ${isDisabled ? "disabled" : ""}"
           id="trigger">
        <span>${displayText}</span>
        <span class="arrow">▼</span>
      </div>
      
      <div class="dropdown ${this._isOpen ? "open" : ""}" id="dropdown">
        <input class="search-input" placeholder="ค้นหา..." id="search">
        <div id="options-container">
          ${this.renderOptions(this._options)}
        </div>
      </div>
    `;
  }
  
  renderOptions(options) {
    if (!options.length) {
      return '<div class="no-results">ไม่พบผลลัพธ์</div>';
    }
    
    const isMultiple = this.hasAttribute("multiple");
    let html = "";
    let lastGroup = null;
    
    options.forEach(opt => {
      if (opt.group !== lastGroup) {
        if (opt.group) {
          html += `<div class="group-label">${opt.group}</div>`;
        }
        lastGroup = opt.group;
      }
      
      const isSelected = this._selectedValues.has(opt.value);
      html += `
        <div class="option ${isSelected ? "selected" : ""} ${opt.disabled ? "disabled" : ""}"
             data-value="${opt.value}">
          ${isMultiple ? `<span class="checkbox">${isSelected ? "✓" : ""}</span>` : ""}
          ${opt.label}
        </div>
      `;
    });
    
    return html;
  }
  
  setupEvents() {
    const trigger = this.shadow.getElementById("trigger");
    const dropdown = this.shadow.getElementById("dropdown");
    const search = this.shadow.getElementById("search");
    
    trigger.addEventListener("click", () => {
      if (this.hasAttribute("disabled")) return;
      this.toggle();
    });
    
    search.addEventListener("input", (e) => {
      const query = e.target.value.toLowerCase();
      const filtered = this._options.filter(opt =>
        opt.label.toLowerCase().includes(query)
      );
      this.shadow.getElementById("options-container").innerHTML =
        this.renderOptions(filtered);
      this.setupOptionEvents();
    });
    
    this.setupOptionEvents();
    
    // Close on outside click
    document.addEventListener("click", (e) => {
      if (!this.contains(e.target) && !this.shadow.contains(e.target)) {
        this.close();
      }
    });
  }
  
  setupOptionEvents() {
    this.shadow.querySelectorAll(".option:not(.disabled)").forEach(opt => {
      opt.addEventListener("click", () => {
        const value = opt.dataset.value;
        
        if (this.hasAttribute("multiple")) {
          if (this._selectedValues.has(value)) {
            this._selectedValues.delete(value);
          } else {
            this._selectedValues.add(value);
          }
        } else {
          this._selectedValues.clear();
          this._selectedValues.add(value);
          this.close();
        }
        
        this.render();
        this.setupEvents();
        this.dispatchEvent(new CustomEvent("change", {
          detail: { value: this.value },
          bubbles: true
        }));
      });
    });
  }
  
  toggle() {
    this._isOpen ? this.close() : this.open();
  }
  
  open() {
    this._isOpen = true;
    this.render();
    this.setupEvents();
    this.shadow.getElementById("search")?.focus();
  }
  
  close() {
    this._isOpen = false;
    this.render();
    this.setupEvents();
  }
  
  get value() {
    const values = [...this._selectedValues];
    return this.hasAttribute("multiple") ? values : values[0] || "";
  }
  
  set value(val) {
    this._selectedValues.clear();
    if (Array.isArray(val)) {
      val.forEach(v => this._selectedValues.add(v));
    } else if (val) {
      this._selectedValues.add(val);
    }
    this.render();
    this.setupEvents();
  }
}

customElements.define("smart-dropdown", SmartDropdown);
```

---

## Step 969: Advanced DOM Patterns

```javascript
// Pattern 1: Render Queue สำหรับ batch DOM updates

class RenderQueue {
  constructor() {
    this.queue = [];
    this.scheduled = false;
  }
  
  add(fn) {
    this.queue.push(fn);
    
    if (!this.scheduled) {
      this.scheduled = true;
      requestAnimationFrame(() => {
        this.flush();
      });
    }
  }
  
  flush() {
    const updates = this.queue.splice(0);
    
    // Group reads and writes to avoid layout thrashing
    const reads = updates.filter(fn => fn.type === "read");
    const writes = updates.filter(fn => fn.type === "write" || !fn.type);
    
    // Do all reads first
    reads.forEach(fn => fn());
    // Then all writes
    writes.forEach(fn => fn());
    
    this.scheduled = false;
  }
}

const renderQueue = new RenderQueue();

// Pattern 2: Element Pool (Object Pool สำหรับ DOM elements)
class ElementPool {
  constructor(tagName, initialSize = 10) {
    this.tagName = tagName;
    this.available = [];
    this.inUse = new WeakSet();
    
    for (let i = 0; i < initialSize; i++) {
      this.available.push(document.createElement(tagName));
    }
  }
  
  acquire() {
    let el = this.available.pop();
    
    if (!el) {
      el = document.createElement(this.tagName);
    }
    
    this.inUse.add(el);
    return el;
  }
  
  release(el) {
    if (!this.inUse.has(el)) return;
    
    // Reset element
    el.innerHTML = "";
    el.className = "";
    el.removeAttribute("style");
    
    this.inUse.delete(el);
    this.available.push(el);
  }
}

// Pattern 3: DOM Diff Updater
class DOMUpdater {
  update(container, newHTML) {
    const temp = document.createElement("div");
    temp.innerHTML = newHTML;
    
    this.diffAndPatch(container, temp);
  }
  
  diffAndPatch(oldNode, newNode) {
    const oldChildren = [...oldNode.children];
    const newChildren = [...newNode.children];
    const max = Math.max(oldChildren.length, newChildren.length);
    
    for (let i = 0; i < max; i++) {
      const old = oldChildren[i];
      const next = newChildren[i];
      
      if (!old && next) {
        oldNode.appendChild(next.cloneNode(true));
      } else if (old && !next) {
        old.remove();
      } else if (old.tagName !== next.tagName) {
        old.replaceWith(next.cloneNode(true));
      } else {
        this.updateAttributes(old, next);
        if (old.innerHTML !== next.innerHTML) {
          if (!old.children.length && !next.children.length) {
            old.textContent = next.textContent;
          } else {
            this.diffAndPatch(old, next);
          }
        }
      }
    }
  }
  
  updateAttributes(old, next) {
    const oldAttrs = new Set([...old.attributes].map(a => a.name));
    const newAttrs = new Set([...next.attributes].map(a => a.name));
    
    for (const attr of oldAttrs) {
      if (!newAttrs.has(attr)) {
        old.removeAttribute(attr);
      }
    }
    
    for (const attr of next.attributes) {
      if (old.getAttribute(attr.name) !== attr.value) {
        old.setAttribute(attr.name, attr.value);
      }
    }
  }
}
```

---

## Step 970: Complete Web Component Library

```javascript
// ตัวอย่าง: Modal Web Component

class ModalDialog extends HTMLElement {
  static get observedAttributes() {
    return ["open", "size", "closable"];
  }
  
  constructor() {
    super();
    this.shadow = this.attachShadow({ mode: "open" });
    this.render();
  }
  
  connectedCallback() {
    this.setupEvents();
    document.addEventListener("keydown", this._handleKeydown = (e) => {
      if (e.key === "Escape" && this.hasAttribute("open")) {
        this.close();
      }
    });
  }
  
  disconnectedCallback() {
    document.removeEventListener("keydown", this._handleKeydown);
  }
  
  attributeChangedCallback(name, oldVal, newVal) {
    if (name === "open") {
      this.handleOpenChange(newVal !== null);
    }
    this.render();
    this.setupEvents();
  }
  
  render() {
    const size = this.getAttribute("size") || "medium";
    const closable = this.getAttribute("closable") !== "false";
    
    const sizes = {
      small: "400px",
      medium: "600px",
      large: "900px",
      fullscreen: "100%"
    };
    
    this.shadow.innerHTML = `
      <style>
        :host {
          display: ${this.hasAttribute("open") ? "flex" : "none"};
          position: fixed;
          inset: 0;
          background: rgba(0,0,0,0.5);
          z-index: 1000;
          align-items: center;
          justify-content: center;
          padding: 16px;
        }
        
        .modal {
          background: white;
          border-radius: 12px;
          max-width: ${sizes[size] || sizes.medium};
          width: 100%;
          max-height: 90vh;
          display: flex;
          flex-direction: column;
          box-shadow: 0 20px 60px rgba(0,0,0,0.3);
          animation: modalIn 0.3s ease;
        }
        
        @keyframes modalIn {
          from { transform: translateY(-30px); opacity: 0; }
          to { transform: none; opacity: 1; }
        }
        
        .modal-header {
          padding: 20px 24px 16px;
          border-bottom: 1px solid #eee;
          display: flex;
          align-items: center;
          justify-content: space-between;
        }
        
        .modal-title {
          font-size: 1.2rem;
          font-weight: 600;
          color: #333;
          margin: 0;
        }
        
        .close-btn {
          background: none;
          border: none;
          font-size: 20px;
          cursor: pointer;
          color: #666;
          padding: 4px;
          border-radius: 4px;
          line-height: 1;
          display: ${closable ? "block" : "none"};
        }
        
        .close-btn:hover { color: #333; background: #f5f5f5; }
        
        .modal-body {
          padding: 24px;
          overflow-y: auto;
          flex: 1;
        }
        
        .modal-footer {
          padding: 16px 24px;
          border-top: 1px solid #eee;
          display: flex;
          justify-content: flex-end;
          gap: 8px;
        }
        
        ::slotted([slot="footer"]) {
          display: contents;
        }
      </style>
      
      <div class="modal" role="dialog" aria-modal="true">
        <div class="modal-header">
          <h2 class="modal-title">
            <slot name="title">ไม่มีชื่อ</slot>
          </h2>
          <button class="close-btn" id="close-btn" aria-label="ปิด">✕</button>
        </div>
        
        <div class="modal-body">
          <slot></slot>
        </div>
        
        <div class="modal-footer">
          <slot name="footer">
            <button id="cancel-btn">ยกเลิก</button>
            <button id="confirm-btn" style="background:#007bff;color:white;border:none;padding:8px 16px;border-radius:6px;cursor:pointer;">ตกลง</button>
          </slot>
        </div>
      </div>
    `;
  }
  
  setupEvents() {
    this.shadow.getElementById("close-btn")?.addEventListener("click", () => this.close());
    this.shadow.getElementById("cancel-btn")?.addEventListener("click", () => this.close());
    this.shadow.getElementById("confirm-btn")?.addEventListener("click", () => {
      this.dispatchEvent(new CustomEvent("confirm", { bubbles: true }));
      this.close();
    });
    
    // Click outside to close
    this.addEventListener("click", (e) => {
      if (e.target === this) this.close();
    });
  }
  
  handleOpenChange(isOpen) {
    if (isOpen) {
      document.body.style.overflow = "hidden";
      this.dispatchEvent(new CustomEvent("open", { bubbles: true }));
    } else {
      document.body.style.overflow = "";
      this.dispatchEvent(new CustomEvent("close", { bubbles: true }));
    }
  }
  
  open() {
    this.setAttribute("open", "");
  }
  
  close() {
    this.removeAttribute("open");
  }
  
  get isOpen() {
    return this.hasAttribute("open");
  }
}

customElements.define("modal-dialog", ModalDialog);
```

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: Custom Element - Rating Component
สร้าง `<star-rating>` Web Component ที่:
- แสดงดาว 5 ดวง
- Interactive (click เพื่อ rate)
- Emit `ratingChanged` event

### แบบฝึกหัดที่ 2: FLIP Animation Grid
สร้าง photo grid ที่:
- Filter ตาม category
- ใช้ FLIP animation เมื่อ filter เปลี่ยน

### แบบฝึกหัดที่ 3: Drag and Drop Kanban
สร้าง Kanban board ที่:
- Drag cards ระหว่าง columns
- Visual feedback ขณะลาก
- LocalStorage persistence

---

## สรุป Part 49

Advanced DOM Manipulation ครอบคลุม:

1. **DocumentFragment**: Batch DOM operations เพื่อ performance
2. **Shadow DOM**: Encapsulate components
3. **Custom Elements**: สร้าง HTML elements ใหม่
4. **Web Animations API**: Smooth animations ด้วย JavaScript
5. **FLIP Technique**: Animation ที่ smooth ระหว่าง layout changes
6. **Drag and Drop**: Native drag interactions
7. **Selection/Range API**: จัดการ text selections
8. **TreeWalker/NodeIterator**: Traverse DOM อย่างมีประสิทธิภาพ
