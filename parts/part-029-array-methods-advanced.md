# Part 29: Array Methods ขั้นสูง (Advanced Array Methods)
## ขั้นตอนที่ 551-570

---

## บทนำ

Array methods เป็นหนึ่งในเครื่องมือที่ทรงพลังที่สุดใน JavaScript โดยเฉพาะเมื่อนำมาใช้ร่วมกันแบบ chaining ใน Part นี้เราจะเรียนรู้การใช้งานขั้นสูงของ methods ที่มีอยู่ รวมถึง methods ใหม่ๆ ที่เพิ่งเข้ามา

---

## ขั้นตอนที่ 551: Deep Dive ใน map()

```javascript
// ตัวอย่างที่ 1: map() พื้นฐาน ทบทวน
const numbers = [1, 2, 3, 4, 5];
const doubled = numbers.map(n => n * 2);
console.log("สองเท่า:", doubled); // [2, 4, 6, 8, 10]

// map() ส่ง 3 arguments ให้ callback: (element, index, array)
const withIndex = numbers.map((n, index) => `[${index}]: ${n}`);
console.log("พร้อม index:", withIndex);
```

```javascript
// ตัวอย่างที่ 2: map() กับ object transformation ซับซ้อน
const employees = [
  { id: 1, firstName: "สมชาย", lastName: "รักไทย", salary: 50000, department: "IT" },
  { id: 2, firstName: "สมหญิง", lastName: "ใจดี", salary: 45000, department: "HR" },
  { id: 3, firstName: "ประทีป", lastName: "สว่าง", salary: 60000, department: "Finance" },
  { id: 4, firstName: "มาลี", lastName: "ดอกไม้", salary: 55000, department: "IT" }
];

// แปลงรูปแบบข้อมูล
const transformed = employees.map(emp => ({
  id: emp.id,
  fullName: `${emp.firstName} ${emp.lastName}`,
  department: emp.department,
  salary: {
    base: emp.salary,
    monthly: emp.salary,
    annual: emp.salary * 12,
    formatted: `฿${emp.salary.toLocaleString()}`
  },
  initials: `${emp.firstName[0]}${emp.lastName[0]}`
}));

console.log("แปลงข้อมูลแล้ว:");
transformed.forEach(emp => {
  console.log(`${emp.fullName} (${emp.initials}): ${emp.salary.formatted}`);
});
```

```javascript
// ตัวอย่างที่ 3: map() กับ nested objects
const orders = [
  {
    id: "ORD001",
    customer: { name: "สมชาย", email: "somchai@example.com" },
    items: [
      { product: "iPhone 15", price: 35000, qty: 1 },
      { product: "AirPods Pro", price: 8500, qty: 2 }
    ],
    status: "delivered",
    date: "2024-01-15"
  },
  {
    id: "ORD002",
    customer: { name: "สมหญิง", email: "somying@example.com" },
    items: [
      { product: "MacBook Pro", price: 75000, qty: 1 }
    ],
    status: "pending",
    date: "2024-01-16"
  }
];

const orderSummaries = orders.map(order => {
  const total = order.items.reduce((sum, item) => sum + (item.price * item.qty), 0);
  const itemCount = order.items.reduce((count, item) => count + item.qty, 0);
  
  return {
    id: order.id,
    customerName: order.customer.name,
    itemCount,
    total: `฿${total.toLocaleString()}`,
    status: order.status === "delivered" ? "จัดส่งแล้ว" : "รอดำเนินการ",
    date: new Date(order.date).toLocaleDateString("th-TH")
  };
});

console.log("สรุปคำสั่งซื้อ:");
orderSummaries.forEach(s => {
  console.log(`${s.id}: ${s.customerName} - ${s.total} (${s.status})`);
});
```

```javascript
// ตัวอย่างที่ 4: map() สำหรับ data normalization
const rawApiData = [
  { user_id: 1, user_name: "somchai", user_email: "SOMCHAI@EXAMPLE.COM", created_at: "2024-01-01T00:00:00Z" },
  { user_id: 2, user_name: "somying", user_email: "SOMYING@EXAMPLE.COM", created_at: "2024-01-02T00:00:00Z" }
];

function normalizeUser(raw) {
  return {
    id: raw.user_id,
    username: raw.user_name,
    email: raw.user_email.toLowerCase(),
    createdAt: new Date(raw.created_at),
    createdAtFormatted: new Date(raw.created_at).toLocaleDateString("th-TH"),
    isRecent: (new Date() - new Date(raw.created_at)) < 7 * 24 * 60 * 60 * 1000
  };
}

const normalizedUsers = rawApiData.map(normalizeUser);
console.log("ข้อมูลที่ normalize แล้ว:", normalizedUsers);
```

---

## ขั้นตอนที่ 552: Deep Dive ใน filter()

```javascript
// ตัวอย่างที่ 5: filter() กับ conditions ซับซ้อน
const products = [
  { id: 1, name: "iPhone 15 Pro", price: 45000, category: "phone", stock: 10, rating: 4.8 },
  { id: 2, name: "Samsung S24", price: 38000, category: "phone", stock: 0, rating: 4.5 },
  { id: 3, name: "MacBook Pro", price: 75000, category: "laptop", stock: 5, rating: 4.9 },
  { id: 4, name: "iPad Air", price: 25000, category: "tablet", stock: 8, rating: 4.6 },
  { id: 5, name: "AirPods Pro", price: 8500, category: "audio", stock: 15, rating: 4.7 },
  { id: 6, name: "Dell XPS", price: 55000, category: "laptop", stock: 3, rating: 4.3 }
];

// filter แบบง่าย
const inStock = products.filter(p => p.stock > 0);
console.log("มีสินค้า:", inStock.length, "รายการ");

// filter หลาย conditions
const affordable = products.filter(p => 
  p.stock > 0 && 
  p.price < 50000 && 
  p.rating >= 4.5
);
console.log("ราคาไม่แพง + มีสินค้า + rating ดี:", affordable.map(p => p.name));
```

```javascript
// ตัวอย่างที่ 6: filter() ด้วย function ที่ reusable
function createFilter(conditions) {
  return function(item) {
    return Object.entries(conditions).every(([key, condition]) => {
      if (typeof condition === "function") {
        return condition(item[key], item);
      }
      return item[key] === condition;
    });
  };
}

const phonesInBudget = products.filter(createFilter({
  category: "phone",
  stock: (stock) => stock > 0,
  price: (price) => price <= 40000
}));

console.log("โทรศัพท์ราคาไม่เกิน 40,000 ที่มีในสต็อก:");
phonesInBudget.forEach(p => console.log(`  - ${p.name}: ฿${p.price.toLocaleString()}`));
```

```javascript
// ตัวอย่างที่ 7: filter() กับ unique values
function unique(array, keyFn = x => x) {
  const seen = new Set();
  return array.filter(item => {
    const key = keyFn(item);
    if (seen.has(key)) return false;
    seen.add(key);
    return true;
  });
}

const data = [
  { id: 1, category: "phone", name: "iPhone" },
  { id: 2, category: "phone", name: "Samsung" },
  { id: 3, category: "laptop", name: "MacBook" },
  { id: 4, category: "laptop", name: "Dell" },
  { id: 5, category: "tablet", name: "iPad" }
];

const uniqueCategories = unique(data, item => item.category);
console.log("หมวดหมู่ไม่ซ้ำ:", uniqueCategories.map(d => d.category));
```

```javascript
// ตัวอย่างที่ 8: filter() สำหรับการค้นหา
function searchProducts(products, query) {
  const lowerQuery = query.toLowerCase().trim();
  
  if (!lowerQuery) return products;
  
  return products.filter(product => {
    return (
      product.name.toLowerCase().includes(lowerQuery) ||
      product.category.toLowerCase().includes(lowerQuery) ||
      product.price.toString().includes(lowerQuery)
    );
  });
}

const searchResults = searchProducts(products, "phone");
console.log("ค้นหา 'phone':", searchResults.map(p => p.name));

const priceSearch = searchProducts(products, "38000");
console.log("ค้นหาราคา '38000':", priceSearch.map(p => p.name));
```

---

## ขั้นตอนที่ 553: Deep Dive ใน reduce()

```javascript
// ตัวอย่างที่ 9: reduce() สร้าง object จาก array
const users = [
  { id: 1, name: "สมชาย", role: "admin" },
  { id: 2, name: "สมหญิง", role: "user" },
  { id: 3, name: "ประทีป", role: "user" },
  { id: 4, name: "มาลี", role: "admin" }
];

// สร้าง lookup object จาก id
const usersById = users.reduce((acc, user) => {
  acc[user.id] = user;
  return acc;
}, {});

console.log("ค้นหาด้วย ID 2:", usersById[2].name);
console.log("ค้นหาด้วย ID 4:", usersById[4].name);
```

```javascript
// ตัวอย่างที่ 10: reduce() สำหรับ grouping
const transactions = [
  { id: 1, type: "income", amount: 50000, category: "salary" },
  { id: 2, type: "expense", amount: 12000, category: "rent" },
  { id: 3, type: "expense", amount: 3000, category: "food" },
  { id: 4, type: "income", amount: 5000, category: "freelance" },
  { id: 5, type: "expense", amount: 1500, category: "food" },
  { id: 6, type: "expense", amount: 500, category: "transport" },
  { id: 7, type: "income", amount: 2000, category: "bonus" }
];

// จัดกลุ่มตาม type
const byType = transactions.reduce((acc, t) => {
  if (!acc[t.type]) acc[t.type] = [];
  acc[t.type].push(t);
  return acc;
}, {});

console.log("รายรับ:", byType.income?.length, "รายการ");
console.log("รายจ่าย:", byType.expense?.length, "รายการ");

// คำนวณยอดรวม
const summary = transactions.reduce((acc, t) => {
  if (t.type === "income") {
    acc.totalIncome += t.amount;
  } else {
    acc.totalExpense += t.amount;
  }
  acc.balance = acc.totalIncome - acc.totalExpense;
  return acc;
}, { totalIncome: 0, totalExpense: 0, balance: 0 });

console.log("สรุปการเงิน:");
console.log(`  รายรับ: ฿${summary.totalIncome.toLocaleString()}`);
console.log(`  รายจ่าย: ฿${summary.totalExpense.toLocaleString()}`);
console.log(`  ยอดคงเหลือ: ฿${summary.balance.toLocaleString()}`);
```

```javascript
// ตัวอย่างที่ 11: reduce() สร้าง nested structure
const flatCategories = [
  { id: 1, name: "อิเล็กทรอนิกส์", parentId: null },
  { id: 2, name: "โทรศัพท์", parentId: 1 },
  { id: 3, name: "แล็ปท็อป", parentId: 1 },
  { id: 4, name: "เสื้อผ้า", parentId: null },
  { id: 5, name: "เสื้อผู้ชาย", parentId: 4 },
  { id: 6, name: "เสื้อผู้หญิง", parentId: 4 },
  { id: 7, name: "iPhone", parentId: 2 },
  { id: 8, name: "Samsung", parentId: 2 }
];

function buildTree(items) {
  const map = items.reduce((acc, item) => {
    acc[item.id] = { ...item, children: [] };
    return acc;
  }, {});
  
  const roots = [];
  
  Object.values(map).forEach(item => {
    if (item.parentId === null) {
      roots.push(item);
    } else if (map[item.parentId]) {
      map[item.parentId].children.push(item);
    }
  });
  
  return roots;
}

const tree = buildTree(flatCategories);
console.log("โครงสร้างต้นไม้:");
tree.forEach(root => {
  console.log(`${root.name}`);
  root.children.forEach(child => {
    console.log(`  └─ ${child.name}`);
    child.children.forEach(grandchild => {
      console.log(`     └─ ${grandchild.name}`);
    });
  });
});
```

```javascript
// ตัวอย่างที่ 12: reduce() สำหรับ pipeline
function pipe(...fns) {
  return function(value) {
    return fns.reduce((acc, fn) => fn(acc), value);
  };
}

const processNumber = pipe(
  n => n * 2,       // คูณ 2
  n => n + 10,      // บวก 10
  n => n.toFixed(2), // ทศนิยม 2 ตำแหน่ง
  s => `ผลลัพธ์: ${s}` // เพิ่ม label
);

console.log(processNumber(5));   // "ผลลัพธ์: 20.00"
console.log(processNumber(15));  // "ผลลัพธ์: 40.00"

// Pipeline สำหรับ data processing
const processUsers = pipe(
  users => users.filter(u => u.role === "admin"),
  users => users.map(u => u.name),
  names => names.join(", ")
);

console.log("ผู้ดูแลระบบ:", processUsers(users));
```

---

## ขั้นตอนที่ 554: reduceRight()

```javascript
// ตัวอย่างที่ 13: reduceRight() - ทำงานจากขวาไปซ้าย
const numbers = [1, 2, 3, 4, 5];

// reduce ปกติ: 1, 12, 123, 1234, 12345
const leftToRight = numbers.reduce((acc, n) => `${acc}${n}`, "");
console.log("ซ้ายไปขวา:", leftToRight); // "12345"

// reduceRight: 5, 54, 543, 5432, 54321
const rightToLeft = numbers.reduceRight((acc, n) => `${acc}${n}`, "");
console.log("ขวาไปซ้าย:", rightToLeft); // "54321"
```

```javascript
// ตัวอย่างที่ 14: reduceRight() สำหรับ compose
function compose(...fns) {
  return function(value) {
    return fns.reduceRight((acc, fn) => fn(acc), value);
  };
}

// compose รันจากขวาไปซ้าย (กลับจาก pipe)
const transform = compose(
  s => `[${s}]`,    // ขั้นที่ 3
  s => s.toUpperCase(), // ขั้นที่ 2
  s => s.trim()     // ขั้นที่ 1 (รันก่อน)
);

console.log(transform("  hello world  ")); // "[HELLO WORLD]"
```

```javascript
// ตัวอย่างที่ 15: reduceRight() กับ nested structures
function flattenDeepRight(arr) {
  return arr.reduceRight((acc, item) => {
    if (Array.isArray(item)) {
      return [...flattenDeepRight(item), ...acc];
    }
    return [item, ...acc];
  }, []);
}

const nested = [1, [2, [3, [4, [5]]]], 6, [7, 8]];
console.log("แบน:", flattenDeepRight(nested)); // [1, 2, 3, 4, 5, 6, 7, 8]
```

---

## ขั้นตอนที่ 555: การเชื่อมต่อ Array Methods (Chaining)

```javascript
// ตัวอย่างที่ 16: Chaining พื้นฐาน
const scores = [
  { name: "สมชาย", score: 85, subject: "คณิต" },
  { name: "สมหญิง", score: 92, subject: "ไทย" },
  { name: "ประทีป", score: 78, subject: "คณิต" },
  { name: "มาลี", score: 95, subject: "ไทย" },
  { name: "วิรัตน์", score: 65, subject: "คณิต" },
  { name: "นาตยา", score: 88, subject: "ไทย" }
];

// chain: filter -> map -> sort -> slice
const topMathStudents = scores
  .filter(s => s.subject === "คณิต")
  .filter(s => s.score >= 70)
  .map(s => ({ ...s, grade: s.score >= 90 ? "A" : s.score >= 80 ? "B" : "C" }))
  .sort((a, b) => b.score - a.score)
  .slice(0, 3);

console.log("นักเรียนคณิตยอดเยี่ยม:");
topMathStudents.forEach((s, i) => {
  console.log(`  ${i + 1}. ${s.name}: ${s.score} (เกรด ${s.grade})`);
});
```

```javascript
// ตัวอย่างที่ 17: Complex chaining สำหรับ report
const salesData = [
  { month: "ม.ค.", rep: "สมชาย", product: "A", amount: 150000 },
  { month: "ม.ค.", rep: "สมหญิง", product: "B", amount: 200000 },
  { month: "ก.พ.", rep: "สมชาย", product: "A", amount: 180000 },
  { month: "ก.พ.", rep: "ประทีป", product: "C", amount: 120000 },
  { month: "มี.ค.", rep: "สมหญิง", product: "B", amount: 250000 },
  { month: "มี.ค.", rep: "สมชาย", product: "A", amount: 160000 }
];

const topPerformers = salesData
  .reduce((acc, sale) => {
    const existing = acc.find(item => item.rep === sale.rep);
    if (existing) {
      existing.total += sale.amount;
      existing.deals++;
    } else {
      acc.push({ rep: sale.rep, total: sale.amount, deals: 1 });
    }
    return acc;
  }, [])
  .map(item => ({
    ...item,
    average: Math.round(item.total / item.deals),
    formatted: `฿${item.total.toLocaleString()}`
  }))
  .sort((a, b) => b.total - a.total)
  .map((item, index) => ({ rank: index + 1, ...item }));

console.log("ผลการขาย:");
topPerformers.forEach(p => {
  console.log(`${p.rank}. ${p.rep}: ${p.formatted} (${p.deals} ดีล)`);
});
```

---

## ขั้นตอนที่ 556: Performance กับ Chaining

```javascript
// ตัวอย่างที่ 18: เปรียบเทียบ performance
const bigArray = Array.from({ length: 100000 }, (_, i) => ({
  id: i,
  value: Math.random() * 1000,
  active: Math.random() > 0.3
}));

// Chaining หลายรอบ - สร้าง intermediate arrays
console.time("chaining");
const result1 = bigArray
  .filter(x => x.active)
  .map(x => x.value)
  .filter(v => v > 500)
  .reduce((sum, v) => sum + v, 0);
console.timeEnd("chaining");
console.log("ผลลัพธ์ 1:", result1.toFixed(2));

// reduce เดียว - ไม่สร้าง intermediate arrays
console.time("single reduce");
const result2 = bigArray.reduce((sum, x) => {
  if (x.active && x.value > 500) {
    return sum + x.value;
  }
  return sum;
}, 0);
console.timeEnd("single reduce");
console.log("ผลลัพธ์ 2:", result2.toFixed(2));
```

```javascript
// ตัวอย่างที่ 19: เมื่อควรใช้ chaining vs single pass
// Chaining: อ่านง่าย, maintain ง่าย
// Single pass: เร็วกว่าสำหรับ arrays ขนาดใหญ่

// สำหรับ arrays เล็กๆ ให้ใช้ chaining เพื่อ readability
const smallArray = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
const readableResult = smallArray
  .filter(n => n % 2 === 0)
  .map(n => n * n)
  .reduce((sum, n) => sum + n, 0);

console.log("ผลรวมกำลังสองของเลขคู่:", readableResult); // 220
```

---

## ขั้นตอนที่ 557: flat() และ flatMap()

```javascript
// ตัวอย่างที่ 20: flat() พื้นฐาน
const nested1 = [1, [2, 3], [4, 5]];
console.log("flat() level 1:", nested1.flat()); // [1, 2, 3, 4, 5]

const nested2 = [1, [2, [3, [4, [5]]]]];
console.log("flat() level 1:", nested2.flat());    // [1, 2, [3, [4, [5]]]]
console.log("flat() level 2:", nested2.flat(2));   // [1, 2, 3, [4, [5]]]
console.log("flat() Infinity:", nested2.flat(Infinity)); // [1, 2, 3, 4, 5]
```

```javascript
// ตัวอย่างที่ 21: flatMap() - map + flat ในขั้นตอนเดียว
const sentences = ["สวัสดี โลก", "เรียน JavaScript", "ภาษา ไทย"];

// แบบ map + flat
const words1 = sentences.map(s => s.split(" ")).flat();
console.log("map + flat:", words1);

// แบบ flatMap
const words2 = sentences.flatMap(s => s.split(" "));
console.log("flatMap:", words2);
// ผลเหมือนกัน: ["สวัสดี", "โลก", "เรียน", "JavaScript", "ภาษา", "ไทย"]
```

```javascript
// ตัวอย่างที่ 22: flatMap() ใน real-world scenarios
const departments = [
  {
    name: "IT",
    employees: ["สมชาย", "ประทีป", "วิรัตน์"]
  },
  {
    name: "HR",
    employees: ["สมหญิง", "มาลี"]
  },
  {
    name: "Finance",
    employees: ["นาตยา", "สุรชัย"]
  }
];

// ดึง employees ทั้งหมด
const allEmployees = departments.flatMap(dept => dept.employees);
console.log("พนักงานทั้งหมด:", allEmployees);
// ["สมชาย", "ประทีป", "วิรัตน์", "สมหญิง", "มาลี", "นาตยา", "สุรชัย"]

// เพิ่มข้อมูล department ด้วย
const employeesWithDept = departments.flatMap(dept =>
  dept.employees.map(name => ({
    name,
    department: dept.name
  }))
);
console.log("พร้อม department:", employeesWithDept);
```

```javascript
// ตัวอย่างที่ 23: flatMap() กับ conditional results
const items = [1, 2, 3, 4, 5, 6, 7, 8];

// แทนที่แต่ละค่าด้วย array (หรือ empty array เพื่อลบ)
const expanded = items.flatMap(n => {
  if (n % 2 === 0) {
    return [n, n * 10]; // เลขคู่: ขยายเป็น 2 ค่า
  } else if (n === 5) {
    return []; // ลบ 5 ออก
  }
  return [n]; // เลขคี่อื่นๆ: คงเดิม
});

console.log("ขยาย:", expanded);
// [1, 2, 20, 3, 4, 40, 6, 60, 7, 8, 80]
```

---

## ขั้นตอนที่ 558: Array.from() กับ Mapping Function

```javascript
// ตัวอย่างที่ 24: Array.from() พื้นฐาน
// จาก string
const chars = Array.from("สวัสดี");
console.log("ตัวอักษร:", chars); // ['ส', 'ว', 'ั', 'ส', 'ด', 'ี']

// จาก Set
const uniqueNums = Array.from(new Set([1, 2, 2, 3, 3, 3]));
console.log("ไม่ซ้ำ:", uniqueNums); // [1, 2, 3]

// จาก Map
const map = new Map([["a", 1], ["b", 2], ["c", 3]]);
const fromMap = Array.from(map);
console.log("จาก Map:", fromMap); // [['a', 1], ['b', 2], ['c', 3]]
```

```javascript
// ตัวอย่างที่ 25: Array.from() กับ mapping function
// Array.from(arrayLike, mapFn)
const squares = Array.from({ length: 5 }, (_, i) => (i + 1) ** 2);
console.log("กำลังสอง:", squares); // [1, 4, 9, 16, 25]

const chars2 = Array.from({ length: 5 }, (_, i) => String.fromCharCode(65 + i));
console.log("ตัวอักษร:", chars2); // ['A', 'B', 'C', 'D', 'E']

// จาก NodeList ใน browser
// const divs = Array.from(document.querySelectorAll('div'));
```

---

## ขั้นตอนที่ 559: สร้าง Ranges ด้วย Array.from()

```javascript
// ตัวอย่างที่ 26: สร้าง range ของตัวเลข
function range(start, end, step = 1) {
  const length = Math.ceil((end - start) / step);
  return Array.from({ length }, (_, i) => start + (i * step));
}

console.log("1 ถึง 5:", range(1, 6));         // [1, 2, 3, 4, 5]
console.log("0 ถึง 10 step 2:", range(0, 11, 2)); // [0, 2, 4, 6, 8, 10]
console.log("1.0 ถึง 2.0 step 0.1:", range(1.0, 2.1, 0.1).map(n => n.toFixed(1)));
```

```javascript
// ตัวอย่างที่ 27: สร้าง array ต่างๆ ด้วย Array.from()
// วันในสัปดาห์
const weekdays = Array.from({ length: 7 }, (_, i) => {
  const days = ["อาทิตย์", "จันทร์", "อังคาร", "พุธ", "พฤหัส", "ศุกร์", "เสาร์"];
  return days[i];
});
console.log("วันในสัปดาห์:", weekdays);

// เดือนในปี
const months = Array.from({ length: 12 }, (_, i) => {
  return new Date(2024, i, 1).toLocaleString("th-TH", { month: "long" });
});
console.log("เดือน:", months);

// ตาราง multiplication
const multiplication = Array.from({ length: 10 }, (_, i) => 
  Array.from({ length: 10 }, (_, j) => (i + 1) * (j + 1))
);
console.log("สูตร 3:", multiplication[2]); // สูตรคูณ 3
```

```javascript
// ตัวอย่างที่ 28: ใช้ Array.from() สำหรับ pagination
function paginate(array, pageSize) {
  const pageCount = Math.ceil(array.length / pageSize);
  return Array.from({ length: pageCount }, (_, i) =>
    array.slice(i * pageSize, (i + 1) * pageSize)
  );
}

const items = Array.from({ length: 23 }, (_, i) => `รายการ ${i + 1}`);
const pages = paginate(items, 5);

console.log("จำนวนหน้า:", pages.length);
pages.forEach((page, i) => {
  console.log(`หน้า ${i + 1}:`, page);
});
```

---

## ขั้นตอนที่ 560: Object.groupBy() - API ใหม่

```javascript
// ตัวอย่างที่ 29: Object.groupBy() (ES2024)
const inventory = [
  { name: "iPhone 15", category: "phone", price: 35000, inStock: true },
  { name: "Samsung S24", category: "phone", price: 38000, inStock: false },
  { name: "MacBook Pro", category: "laptop", price: 75000, inStock: true },
  { name: "Dell XPS", category: "laptop", price: 55000, inStock: true },
  { name: "iPad Air", category: "tablet", price: 25000, inStock: true },
  { name: "AirPods Pro", category: "audio", price: 8500, inStock: false }
];

// จัดกลุ่มตาม category
if (Object.groupBy) {
  const byCategory = Object.groupBy(inventory, item => item.category);
  
  console.log("หมวดหมู่ทั้งหมด:", Object.keys(byCategory));
  console.log("โทรศัพท์:", byCategory.phone?.map(p => p.name));
  console.log("แล็ปท็อป:", byCategory.laptop?.map(p => p.name));
}

// Polyfill สำหรับ environments ที่ยังไม่รองรับ
function groupBy(array, keyFn) {
  return array.reduce((groups, item) => {
    const key = keyFn(item);
    if (!groups[key]) groups[key] = [];
    groups[key].push(item);
    return groups;
  }, {});
}

const byCategory = groupBy(inventory, item => item.category);
console.log("groupBy (polyfill):", Object.keys(byCategory));
```

```javascript
// ตัวอย่างที่ 30: Map.groupBy() (ES2024)
const students = [
  { name: "สมชาย", grade: "A", score: 95 },
  { name: "สมหญิง", grade: "B", score: 85 },
  { name: "ประทีป", grade: "A", score: 92 },
  { name: "มาลี", grade: "C", score: 75 },
  { name: "วิรัตน์", grade: "B", score: 82 }
];

if (Map.groupBy) {
  const byGrade = Map.groupBy(students, s => s.grade);
  console.log("เกรด A:", byGrade.get("A")?.map(s => s.name));
  console.log("เกรด B:", byGrade.get("B")?.map(s => s.name));
}

// สร้าง statistics จากกลุ่ม
const gradeStats = groupBy(students, s => s.grade);
const stats = Object.entries(gradeStats).map(([grade, students]) => ({
  grade,
  count: students.length,
  avgScore: Math.round(students.reduce((sum, s) => sum + s.score, 0) / students.length)
})).sort((a, b) => a.grade.localeCompare(b.grade));

console.log("สถิติเกรด:", stats);
```

---

## ขั้นตอนที่ 561: sort() แบบละเอียด

```javascript
// ตัวอย่างที่ 31: ปัญหาของ sort() ที่ไม่ใช้ comparison function
const numbers = [10, 9, 8, 100, 200, 1, 2];
console.log("sort() ไม่มี comparator:", [...numbers].sort());
// [1, 10, 100, 2, 200, 8, 9] - ผิด! เรียงตาม string

console.log("sort() มี comparator:", [...numbers].sort((a, b) => a - b));
// [1, 2, 8, 9, 10, 100, 200] - ถูก!
```

```javascript
// ตัวอย่างที่ 32: Sort comparison function
// fn(a, b):
//   return < 0: a มาก่อน b
//   return > 0: b มาก่อน a
//   return = 0: ลำดับเดิม

// เรียงจากน้อยไปมาก
const asc = [5, 3, 1, 4, 2].sort((a, b) => a - b);
console.log("น้อยไปมาก:", asc); // [1, 2, 3, 4, 5]

// เรียงจากมากไปน้อย
const desc = [5, 3, 1, 4, 2].sort((a, b) => b - a);
console.log("มากไปน้อย:", desc); // [5, 4, 3, 2, 1]

// เรียง string
const names = ["ประทีป", "สมชาย", "มาลี", "สมหญิง", "วิรัตน์"];
const sortedNames = [...names].sort((a, b) => a.localeCompare(b, "th"));
console.log("เรียงชื่อ:", sortedNames);
```

```javascript
// ตัวอย่างที่ 33: Sort object array
const employees2 = [
  { name: "สมชาย", salary: 50000, hireDate: "2020-03-15" },
  { name: "สมหญิง", salary: 45000, hireDate: "2019-07-22" },
  { name: "ประทีป", salary: 60000, hireDate: "2021-01-10" },
  { name: "มาลี", salary: 55000, hireDate: "2018-11-05" }
];

// เรียงตาม salary จากมากไปน้อย
const bySalary = [...employees2].sort((a, b) => b.salary - a.salary);
console.log("เรียงตามเงินเดือน:");
bySalary.forEach(e => console.log(`  ${e.name}: ฿${e.salary.toLocaleString()}`));

// เรียงตาม hireDate
const byHireDate = [...employees2].sort((a, b) => 
  new Date(a.hireDate) - new Date(b.hireDate)
);
console.log("\nเรียงตามวันเริ่มงาน:");
byHireDate.forEach(e => console.log(`  ${e.name}: ${e.hireDate}`));

// เรียงตาม name
const byName = [...employees2].sort((a, b) => 
  a.name.localeCompare(b.name, "th")
);
console.log("\nเรียงตามชื่อ:");
byName.forEach(e => console.log(`  ${e.name}`));
```

---

## ขั้นตอนที่ 562: Stable Sort

```javascript
// ตัวอย่างที่ 34: Stable sort (ES2019+)
// JavaScript sort() เป็น stable ตาม spec ตั้งแต่ ES2019
// หมายความว่า elements ที่เท่ากันจะรักษาลำดับเดิม

const items = [
  { name: "B", priority: 1, order: 1 },
  { name: "A", priority: 2, order: 2 },
  { name: "C", priority: 1, order: 3 },
  { name: "D", priority: 2, order: 4 },
  { name: "E", priority: 1, order: 5 }
];

const sorted = [...items].sort((a, b) => a.priority - b.priority);
console.log("Stable sort - เรียงตาม priority (ลำดับเดิมรักษาไว้):");
sorted.forEach(item => {
  console.log(`  ${item.name} (priority: ${item.priority}, order: ${item.order})`);
});
// B, C, E (priority 1 รักษาลำดับ) แล้ว A, D (priority 2 รักษาลำดับ)
```

---

## ขั้นตอนที่ 563: Multi-Criteria Sorting

```javascript
// ตัวอย่างที่ 35: เรียงหลายเกณฑ์
function multiSort(...criteria) {
  return function compareFn(a, b) {
    for (const criterion of criteria) {
      const { key, direction = "asc", type = "default" } = 
        typeof criterion === "string" 
          ? { key: criterion } 
          : criterion;
      
      let aVal = typeof key === "function" ? key(a) : a[key];
      let bVal = typeof key === "function" ? key(b) : b[key];
      
      let result;
      if (type === "string") {
        result = aVal.localeCompare(bVal, "th");
      } else if (type === "date") {
        result = new Date(aVal) - new Date(bVal);
      } else {
        result = aVal < bVal ? -1 : aVal > bVal ? 1 : 0;
      }
      
      if (direction === "desc") result = -result;
      if (result !== 0) return result;
    }
    return 0;
  };
}

const employees3 = [
  { name: "สมชาย", department: "IT", salary: 50000, years: 5 },
  { name: "สมหญิง", department: "HR", salary: 45000, years: 3 },
  { name: "ประทีป", department: "IT", salary: 60000, years: 7 },
  { name: "มาลี", department: "HR", salary: 55000, years: 8 },
  { name: "วิรัตน์", department: "IT", salary: 50000, years: 2 }
];

// เรียงตาม department (A-Z) แล้ว salary (มากไปน้อย) แล้ว years (น้อยไปมาก)
const sorted3 = [...employees3].sort(multiSort(
  { key: "department", type: "string" },
  { key: "salary", direction: "desc" },
  { key: "years" }
));

console.log("เรียงหลายเกณฑ์:");
sorted3.forEach(e => {
  console.log(`  ${e.department} | ${e.name} | ฿${e.salary.toLocaleString()} | ${e.years} ปี`);
});
```

---

## ขั้นตอนที่ 564: Functional Programming ด้วย Array Methods

```javascript
// ตัวอย่างที่ 36: Immutable operations
const originalArray = [1, 2, 3, 4, 5];

// ไม่แก้ไข original array
const withNewElement = [...originalArray, 6];        // เพิ่มท้าย
const withPrepended = [0, ...originalArray];          // เพิ่มหน้า
const withoutThird = originalArray.filter((_, i) => i !== 2); // ลบ index 2
const updated = originalArray.map((v, i) => i === 1 ? 99 : v); // แก้ไข index 1

console.log("ต้นฉบับ:", originalArray);  // ไม่เปลี่ยน
console.log("เพิ่มท้าย:", withNewElement);
console.log("ลบ index 2:", withoutThird);
console.log("แก้ไข index 1:", updated);
```

```javascript
// ตัวอย่างที่ 37: Transducers - optimization สำหรับ large arrays
function transduce(transducer, reducer, initial, iterable) {
  const xf = transducer(reducer);
  let result = initial;
  
  for (const item of iterable) {
    result = xf(result, item);
  }
  
  return result;
}

// Filter transducer
const filter = predicate => reducer => (acc, item) => {
  return predicate(item) ? reducer(acc, item) : acc;
};

// Map transducer
const map = transform => reducer => (acc, item) => {
  return reducer(acc, transform(item));
};

// Compose transducers
function compose(...fns) {
  return fns.reduce((f, g) => (...args) => f(g(...args)));
}

const pushReducer = (acc, item) => {
  acc.push(item);
  return acc;
};

const nums = Array.from({ length: 100000 }, (_, i) => i);

// Transducer: filter even, then square
const xform = compose(
  filter(n => n % 2 === 0),
  map(n => n * n)
);

console.time("transducer");
const result = transduce(xform, pushReducer, [], nums);
console.timeEnd("transducer");
console.log("ผลลัพธ์ 3 ค่าแรก:", result.slice(0, 3)); // [0, 4, 16]
```

---

## ขั้นตอนที่ 565: Immutable Array Operations

```javascript
// ตัวอย่างที่ 38: Array methods ใหม่ที่ immutable (ES2023)
const arr = [1, 2, 3, 4, 5];

// toSorted() - sort โดยไม่แก้ต้นฉบับ
const sorted = arr.toSorted((a, b) => b - a);
console.log("sorted (immutable):", sorted);  // [5, 4, 3, 2, 1]
console.log("ต้นฉบับยังเดิม:", arr);         // [1, 2, 3, 4, 5]

// toReversed() - reverse โดยไม่แก้ต้นฉบับ
const reversed = arr.toReversed();
console.log("reversed (immutable):", reversed); // [5, 4, 3, 2, 1]
console.log("ต้นฉบับยังเดิม:", arr);             // [1, 2, 3, 4, 5]

// toSpliced() - splice โดยไม่แก้ต้นฉบับ
const spliced = arr.toSpliced(2, 1, 99, 100);
console.log("spliced (immutable):", spliced);   // [1, 2, 99, 100, 4, 5]
console.log("ต้นฉบับยังเดิม:", arr);             // [1, 2, 3, 4, 5]

// with() - แก้ไข element โดย index โดยไม่แก้ต้นฉบับ
const withChanged = arr.with(2, 99);
console.log("with() (immutable):", withChanged); // [1, 2, 99, 4, 5]
console.log("ต้นฉบับยังเดิม:", arr);              // [1, 2, 3, 4, 5]
```

```javascript
// ตัวอย่างที่ 39: Immutable operations สำหรับ state management
class ImmutableList {
  constructor(items = []) {
    this._items = Object.freeze([...items]);
  }
  
  get items() {
    return this._items;
  }
  
  get length() {
    return this._items.length;
  }
  
  push(...newItems) {
    return new ImmutableList([...this._items, ...newItems]);
  }
  
  pop() {
    return new ImmutableList(this._items.slice(0, -1));
  }
  
  remove(index) {
    return new ImmutableList(
      this._items.filter((_, i) => i !== index)
    );
  }
  
  update(index, value) {
    return new ImmutableList(
      this._items.map((item, i) => i === index ? value : item)
    );
  }
  
  filter(predicate) {
    return new ImmutableList(this._items.filter(predicate));
  }
  
  map(transform) {
    return new ImmutableList(this._items.map(transform));
  }
  
  sort(compareFn) {
    return new ImmutableList([...this._items].sort(compareFn));
  }
  
  toArray() {
    return [...this._items];
  }
}

// ใช้งาน
const list1 = new ImmutableList([1, 2, 3]);
const list2 = list1.push(4, 5);
const list3 = list2.remove(2);
const list4 = list3.update(0, 99);

console.log("list1:", list1.items); // [1, 2, 3]
console.log("list2:", list2.items); // [1, 2, 3, 4, 5]
console.log("list3:", list3.items); // [1, 2, 4, 5]
console.log("list4:", list4.items); // [99, 2, 4, 5]
```

---

## ขั้นตอนที่ 566-570: แบบฝึกหัดขั้นสูง

```javascript
// แบบฝึกหัดที่ 1: สร้าง Data Pipeline
class DataPipeline {
  constructor(data) {
    this.data = data;
    this.operations = [];
  }
  
  filter(predicate) {
    this.operations.push(data => data.filter(predicate));
    return this;
  }
  
  map(transform) {
    this.operations.push(data => data.map(transform));
    return this;
  }
  
  sort(compareFn) {
    this.operations.push(data => [...data].sort(compareFn));
    return this;
  }
  
  take(n) {
    this.operations.push(data => data.slice(0, n));
    return this;
  }
  
  skip(n) {
    this.operations.push(data => data.slice(n));
    return this;
  }
  
  groupBy(keyFn) {
    this.operations.push(data => {
      return data.reduce((groups, item) => {
        const key = keyFn(item);
        if (!groups[key]) groups[key] = [];
        groups[key].push(item);
        return groups;
      }, {});
    });
    return this;
  }
  
  execute() {
    return this.operations.reduce((data, op) => op(data), this.data);
  }
  
  // ดำเนินการ lazy (ไม่รันจนกว่าจะเรียก)
  static from(data) {
    return new DataPipeline(data);
  }
}

// ทดสอบ
const saleRecords = [
  { id: 1, product: "iPhone", amount: 35000, region: "กรุงเทพ", month: 1 },
  { id: 2, product: "iPad", amount: 25000, region: "เชียงใหม่", month: 1 },
  { id: 3, product: "MacBook", amount: 75000, region: "กรุงเทพ", month: 2 },
  { id: 4, product: "iPhone", amount: 35000, region: "ภูเก็ต", month: 2 },
  { id: 5, product: "AirPods", amount: 8500, region: "กรุงเทพ", month: 3 },
  { id: 6, product: "iPad", amount: 25000, region: "กรุงเทพ", month: 3 },
  { id: 7, product: "MacBook", amount: 75000, region: "เชียงใหม่", month: 1 }
];

// Pipeline ซับซ้อน
const result = DataPipeline.from(saleRecords)
  .filter(sale => sale.amount > 20000)
  .map(sale => ({
    ...sale,
    vat: sale.amount * 0.07,
    total: sale.amount * 1.07
  }))
  .sort((a, b) => b.total - a.total)
  .take(5)
  .execute();

console.log("ผลการขาย top 5:");
result.forEach(sale => {
  console.log(`${sale.product} @ ${sale.region}: ฿${sale.total.toFixed(0)}`);
});
```

```javascript
// แบบฝึกหัดที่ 2: Array Statistics
function arrayStats(arr) {
  if (arr.length === 0) return null;
  
  const sorted = [...arr].sort((a, b) => a - b);
  const sum = arr.reduce((s, n) => s + n, 0);
  const mean = sum / arr.length;
  
  // Median
  const mid = Math.floor(sorted.length / 2);
  const median = sorted.length % 2 === 0
    ? (sorted[mid - 1] + sorted[mid]) / 2
    : sorted[mid];
  
  // Mode
  const frequency = arr.reduce((acc, n) => {
    acc[n] = (acc[n] || 0) + 1;
    return acc;
  }, {});
  
  const maxFreq = Math.max(...Object.values(frequency));
  const modes = Object.entries(frequency)
    .filter(([_, freq]) => freq === maxFreq)
    .map(([n]) => Number(n));
  
  // Variance and Standard Deviation
  const variance = arr.reduce((sum, n) => sum + (n - mean) ** 2, 0) / arr.length;
  const stdDev = Math.sqrt(variance);
  
  return {
    count: arr.length,
    sum,
    min: sorted[0],
    max: sorted[sorted.length - 1],
    range: sorted[sorted.length - 1] - sorted[0],
    mean: mean.toFixed(2),
    median,
    mode: modes,
    variance: variance.toFixed(2),
    stdDev: stdDev.toFixed(2),
    quartiles: {
      Q1: sorted[Math.floor(arr.length * 0.25)],
      Q2: median,
      Q3: sorted[Math.floor(arr.length * 0.75)]
    }
  };
}

const testScores = [72, 85, 90, 88, 76, 92, 85, 79, 91, 85, 65, 88];
const stats = arrayStats(testScores);
console.log("สถิติคะแนน:");
console.log(`  จำนวน: ${stats.count}`);
console.log(`  ค่าเฉลี่ย: ${stats.mean}`);
console.log(`  ค่ากลาง: ${stats.median}`);
console.log(`  ฐานนิยม: ${stats.mode.join(", ")}`);
console.log(`  ส่วนเบี่ยงเบนมาตรฐาน: ${stats.stdDev}`);
console.log(`  ไตรภาค: Q1=${stats.quartiles.Q1}, Q2=${stats.quartiles.Q2}, Q3=${stats.quartiles.Q3}`);
```

```javascript
// แบบฝึกหัดที่ 3: Matrix operations ด้วย array methods
function createMatrix(rows, cols, fillFn = () => 0) {
  return Array.from({ length: rows }, (_, i) =>
    Array.from({ length: cols }, (_, j) => fillFn(i, j))
  );
}

function matrixMultiply(A, B) {
  const rows = A.length;
  const cols = B[0].length;
  const inner = B.length;
  
  return createMatrix(rows, cols, (i, j) =>
    Array.from({ length: inner }, (_, k) => A[i][k] * B[k][j])
      .reduce((sum, val) => sum + val, 0)
  );
}

function transposeMatrix(matrix) {
  return matrix[0].map((_, colIndex) =>
    matrix.map(row => row[colIndex])
  );
}

function flattenMatrix(matrix) {
  return matrix.flat();
}

// สร้างเมทริกซ์ตัวอย่าง
const identity3x3 = createMatrix(3, 3, (i, j) => i === j ? 1 : 0);
console.log("Identity matrix:");
identity3x3.forEach(row => console.log(row.join("  ")));

const A = [[1, 2], [3, 4]];
const B = [[5, 6], [7, 8]];
const C = matrixMultiply(A, B);
console.log("\nA × B =");
C.forEach(row => console.log(row));

const transposed = transposeMatrix([[1, 2, 3], [4, 5, 6]]);
console.log("\nTranspose:");
transposed.forEach(row => console.log(row));
```

```javascript
// แบบฝึกหัดที่ 4: Advanced filtering และ searching
class SmartArray {
  constructor(items) {
    this.items = items;
  }
  
  // Full text search
  search(query, fields) {
    const lowerQuery = query.toLowerCase();
    return this.items.filter(item =>
      fields.some(field => {
        const value = this.getNestedValue(item, field);
        return String(value).toLowerCase().includes(lowerQuery);
      })
    );
  }
  
  getNestedValue(obj, path) {
    return path.split(".").reduce((current, key) => current?.[key], obj);
  }
  
  // Range filter
  between(field, min, max) {
    return this.items.filter(item => {
      const value = this.getNestedValue(item, field);
      return value >= min && value <= max;
    });
  }
  
  // Sort by multiple fields
  sortBy(...fields) {
    return [...this.items].sort((a, b) => {
      for (const field of fields) {
        const asc = !field.startsWith("-");
        const key = asc ? field : field.slice(1);
        const aVal = this.getNestedValue(a, key);
        const bVal = this.getNestedValue(b, key);
        
        if (aVal < bVal) return asc ? -1 : 1;
        if (aVal > bVal) return asc ? 1 : -1;
      }
      return 0;
    });
  }
  
  // Paginate
  page(pageNum, pageSize) {
    const start = (pageNum - 1) * pageSize;
    return {
      items: this.items.slice(start, start + pageSize),
      total: this.items.length,
      pages: Math.ceil(this.items.length / pageSize),
      current: pageNum
    };
  }
}

const productList = [
  { id: 1, name: "iPhone 15 Pro", brand: "Apple", price: 45000, rating: 4.8, category: "phone" },
  { id: 2, name: "Samsung Galaxy", brand: "Samsung", price: 35000, rating: 4.5, category: "phone" },
  { id: 3, name: "MacBook Pro 14", brand: "Apple", price: 75000, rating: 4.9, category: "laptop" },
  { id: 4, name: "Dell XPS 15", brand: "Dell", price: 55000, rating: 4.3, category: "laptop" },
  { id: 5, name: "iPad Pro", brand: "Apple", price: 35000, rating: 4.7, category: "tablet" },
  { id: 6, name: "AirPods Pro", brand: "Apple", price: 8500, rating: 4.6, category: "audio" }
];

const smartArr = new SmartArray(productList);

// ค้นหา
const appleProducts = smartArr.search("apple", ["brand", "name"]);
console.log("Apple products:", appleProducts.map(p => p.name));

// กรองตามช่วงราคา
const midRange = smartArr.between("price", 30000, 60000);
console.log("ราคา 30k-60k:", midRange.map(p => p.name));

// เรียงหลายเกณฑ์
const sorted2 = smartArr.sortBy("category", "-rating");
console.log("เรียงตาม category แล้ว rating:");
sorted2.forEach(p => console.log(`  ${p.category} | ${p.name}: ★${p.rating}`));

// Pagination
const page1 = smartArr.page(1, 3);
console.log(`\nหน้า 1/${page1.pages}:`, page1.items.map(p => p.name));
```

---

## สรุป Part 29: Array Methods ขั้นสูง

| Method | คำอธิบาย |
|--------|-----------|
| `map()` | แปลงแต่ละ element, คืน array ใหม่ |
| `filter()` | กรอง elements ที่ผ่านเงื่อนไข |
| `reduce()` | รวม array เป็นค่าเดียว |
| `reduceRight()` | เหมือน reduce แต่ทำจากขวา |
| `flat()` | ทำให้ nested array แบนลง |
| `flatMap()` | map + flat(1) รวมกัน |
| `Array.from()` | สร้าง array จาก iterable |
| `Object.groupBy()` | จัดกลุ่ม elements (ES2024) |
| `sort()` | เรียงลำดับ (ต้องใช้ comparator) |
| `toSorted()` | sort แบบ immutable (ES2023) |
| `toReversed()` | reverse แบบ immutable (ES2023) |
| `with()` | แก้ไข element แบบ immutable (ES2023) |

---

*ต่อไป: Part 30 - Object Methods ขั้นสูง*
