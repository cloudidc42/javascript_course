# Part 27: Async/Await
## ขั้นตอนที่ 511-530

---

## บทนำ

Async/Await เป็น syntax ที่ทำให้การเขียนโค้ด asynchronous ดูเหมือนการเขียนโค้ด synchronous ทำให้อ่านและดูแลรักษาได้ง่ายขึ้นมาก มันถูกสร้างขึ้นมาบน Promise และเป็นส่วนหนึ่งของ ES2017

---

## ขั้นตอนที่ 511: Async/Await คืออะไร

```javascript
// ตัวอย่างที่ 1: เปรียบเทียบ Promise chain vs Async/Await
// แบบ Promise chain
function getUserDataPromise(userId) {
  return fetchUser(userId)
    .then(user => fetchOrders(user.id))
    .then(orders => fetchOrderDetails(orders[0].id))
    .then(details => {
      console.log("รายละเอียด:", details);
      return details;
    })
    .catch(err => console.error("ข้อผิดพลาด:", err));
}

// แบบ Async/Await - อ่านง่ายกว่ามาก
async function getUserDataAsync(userId) {
  try {
    const user = await fetchUser(userId);
    const orders = await fetchOrders(user.id);
    const details = await fetchOrderDetails(orders[0].id);
    console.log("รายละเอียด:", details);
    return details;
  } catch (err) {
    console.error("ข้อผิดพลาด:", err);
  }
}
```

```javascript
// ตัวอย่างที่ 2: ฟังก์ชัน helper สำหรับตัวอย่าง
function delay(ms, value) {
  return new Promise(resolve => setTimeout(() => resolve(value), ms));
}

function fetchUser(id) {
  return delay(300, { id, name: "สมชาย", email: "somchai@example.com" });
}

function fetchOrders(userId) {
  return delay(400, [
    { id: 1, product: "iPhone 15", price: 35000 },
    { id: 2, product: "AirPods Pro", price: 8500 }
  ]);
}

function fetchOrderDetails(orderId) {
  return delay(200, {
    id: orderId,
    status: "จัดส่งแล้ว",
    trackingNumber: "TH1234567890"
  });
}
```

---

## ขั้นตอนที่ 512: การประกาศ async function

```javascript
// ตัวอย่างที่ 3: วิธีประกาศ async function
// 1. Function Declaration
async function greet(name) {
  return `สวัสดี, ${name}!`;
}

// 2. Function Expression
const greetExpr = async function(name) {
  return `สวัสดี, ${name}!`;
};

// 3. Arrow Function
const greetArrow = async (name) => {
  return `สวัสดี, ${name}!`;
};

// 4. Method ใน Object
const obj = {
  async greet(name) {
    return `สวัสดี, ${name}!`;
  }
};

// 5. Method ใน Class
class Greeter {
  async greet(name) {
    return `สวัสดี, ${name}!`;
  }
}

// ทดสอบ
greet("สมชาย").then(msg => console.log(msg));
greetArrow("สมหญิง").then(msg => console.log(msg));
```

```javascript
// ตัวอย่างที่ 4: async function คืน Promise เสมอ
async function getNumber() {
  return 42;
}

// เหมือนกับ
function getNumberPromise() {
  return Promise.resolve(42);
}

// ทั้งคู่คือ Promise
getNumber().then(n => console.log("async:", n));   // 42
getNumberPromise().then(n => console.log("promise:", n)); // 42

console.log(getNumber() instanceof Promise); // true
```

```javascript
// ตัวอย่างที่ 5: async function ที่ throw error คือ rejected Promise
async function failingFunction() {
  throw new Error("เกิดข้อผิดพลาด!");
}

// เหมือนกับ
function failingPromise() {
  return Promise.reject(new Error("เกิดข้อผิดพลาด!"));
}

failingFunction()
  .catch(err => console.error("async error:", err.message));

failingPromise()
  .catch(err => console.error("promise error:", err.message));
```

---

## ขั้นตอนที่ 513: คีย์เวิร์ด await

```javascript
// ตัวอย่างที่ 6: await พื้นฐาน
async function main() {
  console.log("เริ่มต้น");
  
  const result = await delay(1000, "ค่าที่รอ");
  console.log("ค่าที่ได้:", result);
  
  console.log("สิ้นสุด");
}

main();
// Output:
// เริ่มต้น
// (รอ 1 วินาที)
// ค่าที่ได้: ค่าที่รอ
// สิ้นสุด
```

```javascript
// ตัวอย่างที่ 7: await ใช้ได้เฉพาะใน async function
// สิ่งนี้จะ error:
// const result = await delay(1000); // SyntaxError!

// ต้องอยู่ใน async function:
async function correct() {
  const result = await delay(1000, "สำเร็จ");
  return result;
}
```

```javascript
// ตัวอย่างที่ 8: await กับค่าที่ไม่ใช่ Promise
async function awaitNonPromise() {
  const a = await 42;          // await กับตัวเลข = 42
  const b = await "สวัสดี";   // await กับ string = "สวัสดี"
  const c = await null;         // await กับ null = null
  const d = await undefined;    // await กับ undefined = undefined
  
  console.log(a, b, c, d);
  
  // await จะ wrap ค่าใน Promise.resolve() ก่อน
  // จึงทำงานได้เสมอ
}

awaitNonPromise();
```

---

## ขั้นตอนที่ 514: ค่าที่ส่งกลับจาก async function

```javascript
// ตัวอย่างที่ 9: return ค่าจาก async function
async function calculate(a, b) {
  await delay(100);
  return a + b; // ส่ง resolved Promise กลับ
}

async function getUser() {
  await delay(200);
  return { id: 1, name: "สมชาย" }; // ส่ง object กลับ
}

async function getArray() {
  await delay(100);
  return [1, 2, 3, 4, 5]; // ส่ง array กลับ
}

// รับค่าจาก async function
calculate(10, 20)
  .then(result => console.log("ผลลัพธ์:", result)); // 30

getUser()
  .then(user => console.log("ผู้ใช้:", user.name));

// หรือใช้ await ใน async function อื่น
async function displayResults() {
  const sum = await calculate(10, 20);
  const user = await getUser();
  const arr = await getArray();
  
  console.log("ผลบวก:", sum);
  console.log("ผู้ใช้:", user.name);
  console.log("อาร์เรย์:", arr);
}

displayResults();
```

---

## ขั้นตอนที่ 515: การจัดการ Error ด้วย try/catch

```javascript
// ตัวอย่างที่ 10: try/catch พื้นฐาน
async function riskyOperation() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      reject(new Error("การดำเนินงานล้มเหลว!"));
    }, 500);
  });
}

async function main() {
  try {
    const result = await riskyOperation();
    console.log("สำเร็จ:", result);
  } catch (error) {
    console.error("จับ error ได้:", error.message);
  }
}

main();
```

```javascript
// ตัวอย่างที่ 11: จัดการ error หลายชนิด
async function processPayment(amount) {
  if (amount <= 0) {
    throw new RangeError("จำนวนเงินต้องมากกว่า 0");
  }
  if (typeof amount !== 'number') {
    throw new TypeError("จำนวนเงินต้องเป็นตัวเลข");
  }
  
  await delay(500);
  
  if (amount > 100000) {
    throw new Error("จำนวนเงินเกินวงเงิน");
  }
  
  return { success: true, transactionId: `TXN${Date.now()}` };
}

async function handlePayment(amount) {
  try {
    const result = await processPayment(amount);
    console.log("ชำระเงินสำเร็จ:", result.transactionId);
    return result;
  } catch (error) {
    if (error instanceof RangeError) {
      console.error("จำนวนเงินไม่ถูกต้อง:", error.message);
    } else if (error instanceof TypeError) {
      console.error("ประเภทข้อมูลไม่ถูกต้อง:", error.message);
    } else {
      console.error("ข้อผิดพลาดทั่วไป:", error.message);
    }
    throw error; // ส่ง error ต่อไป
  }
}

handlePayment(5000)
  .then(() => console.log("เสร็จสิ้น"))
  .catch(() => {});

handlePayment(-100)
  .catch(err => console.log("จัดการที่ outer level:", err.message));
```

```javascript
// ตัวอย่างที่ 12: try/catch/finally
async function fetchAndProcess() {
  let connection = null;
  
  try {
    console.log("เปิด connection...");
    connection = await openConnection();
    
    console.log("ดึงข้อมูล...");
    const data = await fetchData(connection);
    
    console.log("ประมวลผล...");
    const result = await processData(data);
    
    return result;
  } catch (error) {
    console.error("เกิดข้อผิดพลาด:", error.message);
    throw error;
  } finally {
    if (connection) {
      console.log("ปิด connection...");
      await closeConnection(connection);
    }
  }
}

// Helper functions
async function openConnection() {
  await delay(100);
  return { id: "conn-1", status: "open" };
}

async function fetchData(conn) {
  await delay(300);
  return { records: [1, 2, 3] };
}

async function processData(data) {
  await delay(200);
  return { processed: data.records.length };
}

async function closeConnection(conn) {
  await delay(50);
  console.log(`Connection ${conn.id} ปิดแล้ว`);
}

fetchAndProcess().then(result => console.log("ผลลัพธ์:", result));
```

---

## ขั้นตอนที่ 516: Async Functions คือ Promises

```javascript
// ตัวอย่างที่ 13: ใช้ async function กับ .then()
async function getData() {
  await delay(300);
  return { data: "ข้อมูลจาก API" };
}

// เรียก async function = ได้ Promise กลับมา
getData()
  .then(result => console.log("ข้อมูล:", result.data))
  .catch(err => console.error("ข้อผิดพลาด:", err));
```

```javascript
// ตัวอย่างที่ 14: combine async/await กับ Promise methods
async function task1() {
  await delay(300);
  return "ผลลัพธ์ task 1";
}

async function task2() {
  await delay(200);
  return "ผลลัพธ์ task 2";
}

async function task3() {
  await delay(400);
  return "ผลลัพธ์ task 3";
}

// ใช้ Promise.all กับ async functions
async function runAllTasks() {
  const results = await Promise.all([task1(), task2(), task3()]);
  console.log("ผลลัพธ์ทั้งหมด:", results);
}

runAllTasks();
```

---

## ขั้นตอนที่ 517: Sequential vs Parallel Execution

```javascript
// ตัวอย่างที่ 15: Sequential - ทำงานตามลำดับ (ช้า)
async function sequentialExecution() {
  console.time("sequential");
  
  const a = await delay(500, "A"); // รอ 500ms
  const b = await delay(500, "B"); // รอ 500ms
  const c = await delay(500, "C"); // รอ 500ms
  
  console.timeEnd("sequential"); // ประมาณ 1500ms
  console.log(a, b, c);
}

sequentialExecution();
```

```javascript
// ตัวอย่างที่ 16: Parallel - ทำงานพร้อมกัน (เร็ว)
async function parallelExecution() {
  console.time("parallel");
  
  const [a, b, c] = await Promise.all([
    delay(500, "A"),
    delay(500, "B"),
    delay(500, "C")
  ]);
  
  console.timeEnd("parallel"); // ประมาณ 500ms
  console.log(a, b, c);
}

parallelExecution();
```

```javascript
// ตัวอย่างที่ 17: เมื่อไหรควรใช้ sequential vs parallel
async function userDashboard(userId) {
  // SEQUENTIAL: ต้องการ userId ก่อน จึงดึง user ก่อน
  const user = await fetchUser(userId);
  
  // PARALLEL: ดึงข้อมูลเหล่านี้พร้อมกันได้ เพราะไม่ขึ้นต่อกัน
  const [orders, profile, notifications] = await Promise.all([
    fetchOrders(user.id),
    fetchProfile(user.id),
    fetchNotifications(user.id)
  ]);
  
  // SEQUENTIAL: ต้องการ orders ก่อน จึงดึง details ได้
  const orderDetails = await fetchOrderDetails(orders[0]?.id);
  
  return { user, orders, profile, notifications, orderDetails };
}

// Helper functions
async function fetchProfile(userId) {
  await delay(200);
  return { userId, bio: "นักพัฒนา", avatar: "avatar.jpg" };
}

async function fetchNotifications(userId) {
  await delay(300);
  return [{ id: 1, message: "คำสั่งซื้อถูกจัดส่งแล้ว" }];
}

userDashboard(1).then(data => {
  console.log("ผู้ใช้:", data.user.name);
  console.log("คำสั่งซื้อ:", data.orders.length);
});
```

---

## ขั้นตอนที่ 518: Promise.all กับ async/await

```javascript
// ตัวอย่างที่ 18: Promise.all รูปแบบต่างๆ
async function fetchMultipleUsers(userIds) {
  // วิธีที่ 1: สร้าง array of promises ก่อน
  const userPromises = userIds.map(id => fetchUser(id));
  const users = await Promise.all(userPromises);
  return users;
}

async function fetchMultipleUsersV2(userIds) {
  // วิธีที่ 2: สร้าง inline
  const users = await Promise.all(userIds.map(id => fetchUser(id)));
  return users;
}

async function fetchMultipleUsersV3(userIds) {
  // วิธีที่ 3: ใช้ async ใน map
  const users = await Promise.all(
    userIds.map(async id => {
      const user = await fetchUser(id);
      return { ...user, fetchedAt: Date.now() };
    })
  );
  return users;
}

// ทดสอบ
fetchMultipleUsers([1, 2, 3])
  .then(users => console.log("ผู้ใช้ทั้งหมด:", users.map(u => u.name)));
```

```javascript
// ตัวอย่างที่ 19: Promise.allSettled กับ async/await
async function fetchAllUsersSafe(userIds) {
  const results = await Promise.allSettled(
    userIds.map(id => fetchUser(id))
  );
  
  const successful = results
    .filter(r => r.status === "fulfilled")
    .map(r => r.value);
  
  const failed = results
    .filter(r => r.status === "rejected")
    .map(r => r.reason.message);
  
  return { successful, failed };
}

fetchAllUsersSafe([1, 2, 3])
  .then(({ successful, failed }) => {
    console.log("สำเร็จ:", successful.length);
    console.log("ล้มเหลว:", failed.length);
  });
```

---

## ขั้นตอนที่ 519: Async Iteration ด้วย for-await-of

```javascript
// ตัวอย่างที่ 20: for-await-of กับ array of Promises
async function processPromisesInOrder() {
  const promises = [
    delay(100, "หนึ่ง"),
    delay(300, "สอง"),
    delay(200, "สาม")
  ];
  
  // for-await-of รอแต่ละ Promise ตามลำดับ
  for await (const value of promises) {
    console.log("ค่า:", value);
  }
}

processPromisesInOrder();
// หนึ่ง (100ms)
// สอง (300ms)
// สาม (200ms) - รอสองก่อน แม้จะเสร็จก่อน
```

```javascript
// ตัวอย่างที่ 21: for-await-of กับ async generator
async function* generateNumbers(start, end) {
  for (let i = start; i <= end; i++) {
    await delay(100);
    yield i;
  }
}

async function processNumbers() {
  for await (const num of generateNumbers(1, 5)) {
    console.log("ตัวเลข:", num);
    // ประมวลผลทีละตัว
  }
}

processNumbers();
```

```javascript
// ตัวอย่างที่ 22: for-await-of กับ paginated API
async function* fetchAllPages(baseUrl) {
  let page = 1;
  let hasMore = true;
  
  while (hasMore) {
    const data = await fetchPage(baseUrl, page);
    yield data.items;
    
    hasMore = data.hasNextPage;
    page++;
  }
}

async function fetchPage(url, page) {
  await delay(200);
  // จำลองข้อมูล paginated
  const totalPages = 3;
  return {
    items: Array.from({ length: 5 }, (_, i) => ({
      id: (page - 1) * 5 + i + 1,
      name: `รายการ ${(page - 1) * 5 + i + 1}`
    })),
    hasNextPage: page < totalPages,
    currentPage: page
  };
}

async function getAllItems() {
  const allItems = [];
  
  for await (const items of fetchAllPages("https://api.example.com/items")) {
    console.log(`โหลดหน้า: ${items.length} รายการ`);
    allItems.push(...items);
  }
  
  console.log("รายการทั้งหมด:", allItems.length);
  return allItems;
}

getAllItems();
```

---

## ขั้นตอนที่ 520: Async Generators

```javascript
// ตัวอย่างที่ 23: Async Generator พื้นฐาน
async function* countdown(from) {
  for (let i = from; i >= 0; i--) {
    await delay(500);
    yield i;
  }
}

async function runCountdown() {
  for await (const count of countdown(5)) {
    console.log(count === 0 ? "ปล่อย!" : `${count}...`);
  }
}

runCountdown();
```

```javascript
// ตัวอย่างที่ 24: Async Generator สำหรับ streaming data
async function* streamDataFromAPI(batchSize = 10) {
  let offset = 0;
  let hasMore = true;
  
  while (hasMore) {
    const batch = await fetchBatch(offset, batchSize);
    
    if (batch.length === 0) {
      hasMore = false;
    } else {
      yield batch;
      offset += batchSize;
    }
  }
}

async function fetchBatch(offset, limit) {
  await delay(300);
  const totalRecords = 35;
  
  if (offset >= totalRecords) return [];
  
  const end = Math.min(offset + limit, totalRecords);
  return Array.from({ length: end - offset }, (_, i) => ({
    id: offset + i + 1,
    value: `ข้อมูล ${offset + i + 1}`
  }));
}

async function processStream() {
  let totalProcessed = 0;
  
  for await (const batch of streamDataFromAPI(10)) {
    totalProcessed += batch.length;
    console.log(`ประมวลผล batch: ${batch.length} รายการ (รวม: ${totalProcessed})`);
  }
  
  console.log("เสร็จสิ้น ประมวลผลทั้งหมด:", totalProcessed, "รายการ");
}

processStream();
```

---

## ขั้นตอนที่ 521: Top-Level Await

```javascript
// ตัวอย่างที่ 25: Top-level await ใน ES modules
// ไฟล์: config.mjs
// const config = await fetchConfig(); // ใช้ได้ใน module!
// export default config;

// จำลองใน Node.js
async function main() {
  // ในไฟล์ .mjs สามารถใช้ await โดยตรงได้
  // const config = await loadConfig();
  
  // แต่ในโค้ดปกติต้องใช้ async wrapper
  const config = await loadConfig();
  console.log("Config โหลดแล้ว:", config);
}

async function loadConfig() {
  await delay(200);
  return {
    dbHost: "localhost",
    dbPort: 5432,
    apiKey: "secret-key"
  };
}

main();
```

```javascript
// ตัวอย่างที่ 26: Pattern สำหรับ top-level async code
// วิธีที่ 1: IIFE (Immediately Invoked Function Expression)
(async () => {
  try {
    const data = await fetchUser(1);
    console.log("ผู้ใช้:", data.name);
  } catch (err) {
    console.error("ข้อผิดพลาด:", err.message);
    process.exit(1);
  }
})();

// วิธีที่ 2: Named async function ที่เรียกใช้ทันที
async function init() {
  const user = await fetchUser(1);
  const orders = await fetchOrders(user.id);
  console.log(`${user.name} มี ${orders.length} คำสั่งซื้อ`);
}

init().catch(err => {
  console.error("Initialization failed:", err);
  process.exit(1);
});
```

---

## ขั้นตอนที่ 522: การแปลง Promise Chain เป็น Async/Await

```javascript
// ตัวอย่างที่ 27: Promise chain ที่ซับซ้อน
// แบบ Promise
function processOrderPromise(orderId) {
  return fetchOrder(orderId)
    .then(order => {
      if (!order) throw new Error("ไม่พบคำสั่งซื้อ");
      return validateOrder(order);
    })
    .then(validOrder => {
      return Promise.all([
        applyDiscount(validOrder),
        calculateShipping(validOrder)
      ]);
    })
    .then(([discountedOrder, shippingCost]) => {
      return chargeCustomer({
        ...discountedOrder,
        shipping: shippingCost
      });
    })
    .then(chargeResult => {
      return sendConfirmation(chargeResult);
    })
    .catch(err => {
      console.error("Order processing failed:", err.message);
      throw err;
    });
}

// แบบ Async/Await - ชัดเจนกว่า
async function processOrderAsync(orderId) {
  try {
    const order = await fetchOrder(orderId);
    if (!order) throw new Error("ไม่พบคำสั่งซื้อ");
    
    const validOrder = await validateOrder(order);
    
    const [discountedOrder, shippingCost] = await Promise.all([
      applyDiscount(validOrder),
      calculateShipping(validOrder)
    ]);
    
    const chargeResult = await chargeCustomer({
      ...discountedOrder,
      shipping: shippingCost
    });
    
    return await sendConfirmation(chargeResult);
  } catch (err) {
    console.error("Order processing failed:", err.message);
    throw err;
  }
}

// Helper functions
async function fetchOrder(id) {
  await delay(200);
  return { id, product: "iPhone 15", price: 35000, customerId: 1 };
}

async function validateOrder(order) {
  await delay(100);
  return { ...order, validated: true };
}

async function applyDiscount(order) {
  await delay(150);
  return { ...order, discount: 500, finalPrice: order.price - 500 };
}

async function calculateShipping(order) {
  await delay(100);
  return 150; // ค่าจัดส่ง
}

async function chargeCustomer(order) {
  await delay(500);
  return { ...order, paid: true, transactionId: `TXN${Date.now()}` };
}

async function sendConfirmation(order) {
  await delay(200);
  console.log(`ยืนยันคำสั่งซื้อ ${order.id}: ชำระ ${order.finalPrice + order.shipping} บาท`);
  return order;
}

processOrderAsync(1).then(result => {
  console.log("การดำเนินการเสร็จสิ้น:", result.transactionId);
});
```

---

## ขั้นตอนที่ 523: รูปแบบการจัดการ Error ขั้นสูง

```javascript
// ตัวอย่างที่ 28: Error handling ระดับต่างๆ
async function level3() {
  throw new Error("error จาก level 3");
}

async function level2() {
  try {
    await level3();
  } catch (err) {
    // จัดการ error บางส่วน แล้วส่งต่อ
    console.log("level 2 จัดการ:", err.message);
    throw new Error("error จาก level 2 (wrapped)");
  }
}

async function level1() {
  try {
    await level2();
  } catch (err) {
    console.error("level 1 จัดการสุดท้าย:", err.message);
  }
}

level1();
```

```javascript
// ตัวอย่างที่ 29: Wrapper function สำหรับ error handling
function withErrorHandling(asyncFn) {
  return async function(...args) {
    try {
      return [null, await asyncFn(...args)];
    } catch (error) {
      return [error, null];
    }
  };
}

// ใช้งาน
const safeGetUser = withErrorHandling(fetchUser);
const safeGetOrders = withErrorHandling(fetchOrders);

async function safeMain() {
  const [userErr, user] = await safeGetUser(1);
  if (userErr) {
    console.error("ไม่สามารถดึงข้อมูลผู้ใช้:", userErr.message);
    return;
  }
  
  const [ordersErr, orders] = await safeGetOrders(user.id);
  if (ordersErr) {
    console.error("ไม่สามารถดึงคำสั่งซื้อ:", ordersErr.message);
    return;
  }
  
  console.log(`${user.name} มี ${orders.length} คำสั่งซื้อ`);
}

safeMain();
```

```javascript
// ตัวอย่างที่ 30: Error context enrichment
class AppError extends Error {
  constructor(message, context = {}) {
    super(message);
    this.name = "AppError";
    this.context = context;
    this.timestamp = new Date().toISOString();
  }
}

async function enrichedErrorHandling(userId) {
  try {
    const user = await fetchUser(userId);
    const orders = await fetchOrders(user.id);
    return { user, orders };
  } catch (error) {
    throw new AppError("ไม่สามารถโหลดข้อมูลผู้ใช้", {
      userId,
      originalError: error.message,
      stack: error.stack
    });
  }
}

enrichedErrorHandling(999)
  .catch(err => {
    if (err instanceof AppError) {
      console.error("App Error:", err.message);
      console.error("Context:", err.context);
      console.error("Time:", err.timestamp);
    }
  });
```

---

## ขั้นตอนที่ 524: Timeout Pattern

```javascript
// ตัวอย่างที่ 31: Timeout ด้วย async/await
async function withTimeout(promise, timeoutMs) {
  const timeoutPromise = new Promise((_, reject) => {
    setTimeout(() => {
      reject(new Error(`Timeout หลังจาก ${timeoutMs}ms`));
    }, timeoutMs);
  });
  
  return Promise.race([promise, timeoutPromise]);
}

async function slowOperation() {
  await delay(2000);
  return "ผลลัพธ์ที่ช้า";
}

async function fastOperation() {
  await delay(500);
  return "ผลลัพธ์ที่เร็ว";
}

async function main() {
  try {
    // Fast operation - สำเร็จ
    const fastResult = await withTimeout(fastOperation(), 1000);
    console.log("Fast:", fastResult);
    
    // Slow operation - timeout
    const slowResult = await withTimeout(slowOperation(), 1000);
    console.log("Slow:", slowResult);
  } catch (err) {
    console.error("ข้อผิดพลาด:", err.message);
  }
}

main();
```

```javascript
// ตัวอย่างที่ 32: Timeout พร้อม AbortController
async function fetchWithAbort(url, timeoutMs) {
  const controller = new AbortController();
  const { signal } = controller;
  
  const timeout = setTimeout(() => {
    controller.abort();
  }, timeoutMs);
  
  try {
    const response = await fetch(url, { signal });
    clearTimeout(timeout);
    return response;
  } catch (err) {
    clearTimeout(timeout);
    if (err.name === "AbortError") {
      throw new Error(`Request timeout หลังจาก ${timeoutMs}ms`);
    }
    throw err;
  }
}
```

---

## ขั้นตอนที่ 525: Retry Pattern

```javascript
// ตัวอย่างที่ 33: Retry ด้วย async/await
async function retry(asyncFn, options = {}) {
  const {
    maxAttempts = 3,
    delay: retryDelay = 1000,
    backoff = 2, // exponential backoff
    onRetry = null
  } = options;
  
  let lastError;
  
  for (let attempt = 1; attempt <= maxAttempts; attempt++) {
    try {
      return await asyncFn();
    } catch (error) {
      lastError = error;
      
      if (attempt === maxAttempts) break;
      
      const waitTime = retryDelay * Math.pow(backoff, attempt - 1);
      
      if (onRetry) {
        onRetry(attempt, error, waitTime);
      }
      
      await delay(waitTime);
    }
  }
  
  throw new Error(`ล้มเหลวหลังจาก ${maxAttempts} ครั้ง: ${lastError.message}`);
}

// ฟังก์ชันที่ไม่เสถียร
let callCount = 0;
async function unstableApi() {
  callCount++;
  await delay(200);
  
  if (callCount < 3) {
    throw new Error(`API Error (ครั้งที่ ${callCount})`);
  }
  
  return { data: "สำเร็จ!", calls: callCount };
}

// ใช้งาน
retry(unstableApi, {
  maxAttempts: 5,
  delay: 500,
  backoff: 1.5,
  onRetry: (attempt, error, waitTime) => {
    console.log(`ความพยายามครั้งที่ ${attempt} ล้มเหลว: ${error.message}`);
    console.log(`รอ ${waitTime}ms แล้วลองใหม่...`);
  }
})
.then(result => console.log("สำเร็จ:", result))
.catch(err => console.error("ล้มเหลว:", err.message));
```

---

## ขั้นตอนที่ 526: Async Functions ใน Array Methods

```javascript
// ตัวอย่างที่ 34: ระวัง! async ใน forEach ไม่รอ
async function wrongApproach() {
  const ids = [1, 2, 3];
  
  // ผิด! forEach ไม่รอ async callbacks
  ids.forEach(async (id) => {
    const user = await fetchUser(id);
    console.log("ผู้ใช้:", user.name); // ลำดับไม่แน่นอน
  });
  
  console.log("สิ้นสุด?"); // อาจพิมพ์ก่อน users!
}

// ถูก - ใช้ for...of
async function correctApproach() {
  const ids = [1, 2, 3];
  
  for (const id of ids) {
    const user = await fetchUser(id);
    console.log("ผู้ใช้:", user.name); // ตามลำดับแน่นอน
  }
  
  console.log("สิ้นสุด"); // พิมพ์หลัง users เสมอ
}

// ถูก - ใช้ Promise.all กับ map (parallel)
async function parallelApproach() {
  const ids = [1, 2, 3];
  
  const users = await Promise.all(ids.map(id => fetchUser(id)));
  users.forEach(user => console.log("ผู้ใช้:", user.name));
}
```

```javascript
// ตัวอย่างที่ 35: async reduce
async function sumUserAges(userIds) {
  const sum = await userIds.reduce(async (sumPromise, id) => {
    const currentSum = await sumPromise; // รอ promise ก่อนหน้า
    const user = await fetchUser(id);
    return currentSum + (user.age || 0);
  }, Promise.resolve(0));
  
  return sum;
}

// หรือวิธีที่อ่านง่ายกว่า
async function sumUserAgesClean(userIds) {
  const users = await Promise.all(userIds.map(id => fetchUser(id)));
  return users.reduce((sum, user) => sum + (user.age || 0), 0);
}
```

---

## ขั้นตอนที่ 527: Real-World Async/Await Examples

```javascript
// ตัวอย่างที่ 36: Authentication System
class AuthService {
  constructor() {
    this.currentUser = null;
    this.token = null;
  }
  
  async login(email, password) {
    try {
      const credentials = await this.validateCredentials(email, password);
      const token = await this.generateToken(credentials.userId);
      const user = await this.getUserProfile(credentials.userId);
      
      this.currentUser = user;
      this.token = token;
      
      await this.logLoginEvent(user.id);
      
      return { success: true, user, token };
    } catch (error) {
      await this.logFailedLogin(email, error.message);
      throw error;
    }
  }
  
  async validateCredentials(email, password) {
    await delay(300);
    
    const validUsers = {
      "admin@example.com": { userId: 1, password: "admin123" }
    };
    
    const user = validUsers[email];
    
    if (!user || user.password !== password) {
      throw new Error("อีเมลหรือรหัสผ่านไม่ถูกต้อง");
    }
    
    return user;
  }
  
  async generateToken(userId) {
    await delay(100);
    return `JWT.${userId}.${Date.now()}.secret`;
  }
  
  async getUserProfile(userId) {
    await delay(200);
    return {
      id: userId,
      name: "ผู้ดูแลระบบ",
      email: "admin@example.com",
      role: "admin"
    };
  }
  
  async logLoginEvent(userId) {
    await delay(50);
    console.log(`บันทึก: ผู้ใช้ ${userId} เข้าสู่ระบบ`);
  }
  
  async logFailedLogin(email, reason) {
    await delay(50);
    console.log(`บันทึกการเข้าสู่ระบบล้มเหลว: ${email} - ${reason}`);
  }
  
  async logout() {
    if (!this.currentUser) {
      throw new Error("ไม่มีผู้ใช้ที่เข้าสู่ระบบ");
    }
    
    const userId = this.currentUser.id;
    this.currentUser = null;
    this.token = null;
    
    await this.logLogoutEvent(userId);
    return { success: true };
  }
  
  async logLogoutEvent(userId) {
    await delay(50);
    console.log(`บันทึก: ผู้ใช้ ${userId} ออกจากระบบ`);
  }
}

// ใช้งาน
const auth = new AuthService();

async function demonstrateAuth() {
  try {
    // Login สำเร็จ
    const result = await auth.login("admin@example.com", "admin123");
    console.log("เข้าสู่ระบบสำเร็จ:", result.user.name);
    console.log("Token:", result.token.substring(0, 20) + "...");
    
    // Logout
    await auth.logout();
    console.log("ออกจากระบบสำเร็จ");
    
    // Login ล้มเหลว
    await auth.login("wrong@example.com", "wrongpassword");
  } catch (err) {
    console.error("ข้อผิดพลาด:", err.message);
  }
}

demonstrateAuth();
```

---

## ขั้นตอนที่ 528: Testing Async Functions

```javascript
// ตัวอย่างที่ 37: เขียน async tests
// รูปแบบ simple testing framework
async function test(description, fn) {
  try {
    await fn();
    console.log(`✓ ${description}`);
  } catch (err) {
    console.error(`✗ ${description}: ${err.message}`);
  }
}

function assert(condition, message) {
  if (!condition) {
    throw new Error(`Assertion failed: ${message}`);
  }
}

// Test async functions
async function runTests() {
  await test("fetchUser ส่งคืน user object", async () => {
    const user = await fetchUser(1);
    assert(user !== null, "user ควรไม่เป็น null");
    assert(user.id === 1, "user.id ควรเป็น 1");
    assert(typeof user.name === "string", "user.name ควรเป็น string");
  });
  
  await test("fetchOrders ส่งคืน array", async () => {
    const orders = await fetchOrders(1);
    assert(Array.isArray(orders), "orders ควรเป็น array");
    assert(orders.length > 0, "orders ควรมีข้อมูล");
  });
  
  await test("withTimeout ทำงานถูกต้อง", async () => {
    const result = await withTimeout(delay(100, "เร็ว"), 1000);
    assert(result === "เร็ว", "ควรได้ผลลัพธ์จาก fast operation");
    
    try {
      await withTimeout(delay(2000, "ช้า"), 500);
      assert(false, "ควร throw timeout error");
    } catch (err) {
      assert(err.message.includes("Timeout"), "ควรเป็น timeout error");
    }
  });
}

runTests();
```

---

## ขั้นตอนที่ 529: Performance Patterns

```javascript
// ตัวอย่างที่ 38: Memoization สำหรับ async functions
function memoizeAsync(fn) {
  const cache = new Map();
  
  return async function(...args) {
    const key = JSON.stringify(args);
    
    if (cache.has(key)) {
      console.log(`Cache hit สำหรับ args: ${key}`);
      return cache.get(key);
    }
    
    const result = await fn(...args);
    cache.set(key, result);
    return result;
  };
}

const cachedFetchUser = memoizeAsync(async (id) => {
  console.log(`ดึงข้อมูลผู้ใช้ id: ${id} จาก API`);
  await delay(500);
  return { id, name: `ผู้ใช้ ${id}` };
});

async function testMemoization() {
  // เรียกครั้งแรก - ดึงจาก API
  const user1 = await cachedFetchUser(1);
  console.log("ครั้งที่ 1:", user1.name);
  
  // เรียกครั้งที่สอง - ดึงจาก cache (เร็วกว่า)
  const user1Again = await cachedFetchUser(1);
  console.log("ครั้งที่ 2:", user1Again.name);
  
  // เรียก user ใหม่ - ดึงจาก API
  const user2 = await cachedFetchUser(2);
  console.log("ผู้ใช้ใหม่:", user2.name);
}

testMemoization();
```

```javascript
// ตัวอย่างที่ 39: Rate Limiting
function createRateLimiter(maxCalls, periodMs) {
  let calls = 0;
  let resetTime = Date.now() + periodMs;
  
  return async function rateLimited(fn) {
    const now = Date.now();
    
    if (now > resetTime) {
      calls = 0;
      resetTime = now + periodMs;
    }
    
    if (calls >= maxCalls) {
      const waitTime = resetTime - now;
      console.log(`Rate limit ถึงแล้ว รอ ${waitTime}ms`);
      await delay(waitTime);
      calls = 0;
      resetTime = Date.now() + periodMs;
    }
    
    calls++;
    return fn();
  };
}

const rateLimited = createRateLimiter(3, 2000); // 3 calls per 2 seconds

async function makeApiCall(id) {
  return rateLimited(async () => {
    await delay(100);
    return `ผลลัพธ์ ${id}`;
  });
}

// ทดสอบ rate limiting
async function testRateLimit() {
  const calls = Array.from({ length: 7 }, (_, i) => makeApiCall(i + 1));
  const results = await Promise.all(calls);
  console.log("ผลลัพธ์ทั้งหมด:", results);
}

testRateLimit();
```

---

## ขั้นตอนที่ 530: แบบฝึกหัด

```javascript
// แบบฝึกหัดที่ 1: แปลง callback เป็น async/await
// Callback-based function
function getWeatherByCity(city, callback) {
  setTimeout(() => {
    const weatherData = {
      "กรุงเทพ": { temp: 32, humidity: 80, desc: "ร้อน" },
      "เชียงใหม่": { temp: 25, humidity: 70, desc: "เย็น" },
      "ภูเก็ต": { temp: 30, humidity: 85, desc: "ชื้น" }
    };
    
    const data = weatherData[city];
    if (data) {
      callback(null, data);
    } else {
      callback(new Error(`ไม่พบข้อมูลสำหรับ ${city}`));
    }
  }, 500);
}

// แปลงเป็น Promise
function getWeatherPromise(city) {
  return new Promise((resolve, reject) => {
    getWeatherByCity(city, (err, data) => {
      if (err) reject(err);
      else resolve(data);
    });
  });
}

// ใช้งานด้วย async/await
async function showWeather(cities) {
  console.log("สภาพอากาศวันนี้:");
  
  for (const city of cities) {
    try {
      const weather = await getWeatherPromise(city);
      console.log(`${city}: ${weather.temp}°C, ${weather.desc} (ความชื้น ${weather.humidity}%)`);
    } catch (err) {
      console.error(`${city}: ${err.message}`);
    }
  }
}

showWeather(["กรุงเทพ", "เชียงใหม่", "พัทยา", "ภูเก็ต"]);
```

```javascript
// แบบฝึกหัดที่ 2: Async Pipeline
async function processImage(imageUrl) {
  // Step 1: Download
  async function downloadImage(url) {
    console.log("กำลังดาวน์โหลด:", url);
    await delay(500);
    return { url, size: 2048, format: "jpg" };
  }
  
  // Step 2: Validate
  async function validateImage(image) {
    await delay(100);
    if (image.size > 10000) throw new Error("ไฟล์ใหญ่เกินไป");
    return { ...image, valid: true };
  }
  
  // Step 3: Resize
  async function resizeImage(image) {
    console.log("กำลัง resize...");
    await delay(300);
    return { ...image, width: 800, height: 600, resized: true };
  }
  
  // Step 4: Compress
  async function compressImage(image) {
    console.log("กำลัง compress...");
    await delay(200);
    return { ...image, compressed: true, newSize: Math.floor(image.size * 0.7) };
  }
  
  // Step 5: Upload
  async function uploadImage(image) {
    console.log("กำลังอัปโหลด...");
    await delay(400);
    return {
      ...image,
      uploadUrl: `https://cdn.example.com/${Date.now()}.jpg`,
      uploaded: true
    };
  }
  
  // Pipeline
  const downloaded = await downloadImage(imageUrl);
  const validated = await validateImage(downloaded);
  const resized = await resizeImage(validated);
  const compressed = await compressImage(resized);
  const uploaded = await uploadImage(compressed);
  
  return uploaded;
}

processImage("https://example.com/photo.jpg")
  .then(result => {
    console.log("สำเร็จ! URL:", result.uploadUrl);
    console.log(`ลดขนาดจาก ${result.size} เป็น ${result.newSize} bytes`);
  })
  .catch(err => console.error("ล้มเหลว:", err.message));
```

```javascript
// แบบฝึกหัดที่ 3: Async State Machine
async function orderStateMachine(orderId) {
  const states = {
    CREATED: "สร้างคำสั่งซื้อ",
    PAYMENT_PENDING: "รอชำระเงิน",
    PAID: "ชำระเงินแล้ว",
    PROCESSING: "กำลังดำเนินการ",
    SHIPPED: "จัดส่งแล้ว",
    DELIVERED: "ส่งถึงแล้ว"
  };
  
  const transitions = {
    CREATED: () => transitionTo("PAYMENT_PENDING"),
    PAYMENT_PENDING: () => processPaymentState(),
    PAID: () => transitionTo("PROCESSING"),
    PROCESSING: () => transitionTo("SHIPPED"),
    SHIPPED: () => transitionTo("DELIVERED"),
    DELIVERED: () => console.log("คำสั่งซื้อเสร็จสิ้น!")
  };
  
  let currentState = "CREATED";
  
  async function transitionTo(newState) {
    await delay(200);
    console.log(`${states[currentState]} → ${states[newState]}`);
    currentState = newState;
    return currentState;
  }
  
  async function processPaymentState() {
    console.log("กำลังประมวลผลการชำระเงิน...");
    await delay(1000);
    const success = Math.random() > 0.2; // 80% success
    
    if (success) {
      return transitionTo("PAID");
    } else {
      throw new Error("การชำระเงินล้มเหลว");
    }
  }
  
  // Run state machine
  try {
    await transitions[currentState]();
    await transitions[currentState]();
    await transitions[currentState]();
    await transitions[currentState]();
    await transitions[currentState]();
    transitions[currentState]();
    console.log(`คำสั่งซื้อ ${orderId}: ${states[currentState]}`);
  } catch (err) {
    console.error(`คำสั่งซื้อ ${orderId} ล้มเหลว:`, err.message);
  }
}

orderStateMachine("ORD001");
```

```javascript
// แบบฝึกหัดที่ 4: Concurrent Tasks with Limit
async function processWithLimit(tasks, limit) {
  const results = [];
  const executing = new Set();
  
  for (const task of tasks) {
    const promise = Promise.resolve().then(() => task());
    results.push(promise);
    executing.add(promise);
    
    const clean = () => executing.delete(promise);
    promise.then(clean, clean);
    
    if (executing.size >= limit) {
      await Promise.race(executing);
    }
  }
  
  return Promise.all(results);
}

// สร้าง tasks จำลอง
function createTask(id) {
  return async () => {
    console.log(`เริ่ม task ${id}`);
    await delay(Math.random() * 1000 + 500);
    console.log(`เสร็จ task ${id}`);
    return `ผลลัพธ์ ${id}`;
  };
}

const tasks = Array.from({ length: 10 }, (_, i) => createTask(i + 1));

console.time("ทำงานพร้อมกัน 3 tasks");
processWithLimit(tasks, 3).then(results => {
  console.timeEnd("ทำงานพร้อมกัน 3 tasks");
  console.log("ผลลัพธ์:", results);
});
```

```javascript
// แบบฝึกหัดที่ 5: สร้าง async queue
class AsyncQueue {
  constructor() {
    this.queue = [];
    this.processing = false;
  }
  
  async enqueue(task, priority = 0) {
    return new Promise((resolve, reject) => {
      this.queue.push({ task, resolve, reject, priority });
      this.queue.sort((a, b) => b.priority - a.priority);
      this.processNext();
    });
  }
  
  async processNext() {
    if (this.processing || this.queue.length === 0) return;
    
    this.processing = true;
    const { task, resolve, reject } = this.queue.shift();
    
    try {
      const result = await task();
      resolve(result);
    } catch (err) {
      reject(err);
    } finally {
      this.processing = false;
      this.processNext();
    }
  }
}

const queue = new AsyncQueue();

async function testQueue() {
  const tasks = [
    queue.enqueue(async () => {
      await delay(500);
      return "Task ปกติ 1";
    }, 1),
    queue.enqueue(async () => {
      await delay(300);
      return "Task ด่วน 1";
    }, 5),
    queue.enqueue(async () => {
      await delay(400);
      return "Task ปกติ 2";
    }, 1),
    queue.enqueue(async () => {
      await delay(200);
      return "Task ด่วน 2";
    }, 5)
  ];
  
  const results = await Promise.all(tasks);
  console.log("ผลลัพธ์ตามลำดับ priority:");
  results.forEach(r => console.log(" -", r));
}

testQueue();
```

---

## สรุป Part 27: Async/Await

| แนวคิด | คำอธิบาย |
|--------|-----------|
| `async function` | ประกาศฟังก์ชัน asynchronous ที่คืน Promise เสมอ |
| `await` | รอ Promise ให้ settle ก่อนดำเนินต่อ |
| `try/catch` | จัดการ error ใน async function |
| `Promise.all` | รัน async operations แบบ parallel |
| `for-await-of` | iterate ผ่าน async iterable |
| Sequential | `await` แต่ละ operation ตามลำดับ |
| Parallel | `Promise.all([...])` สำหรับ operations อิสระต่อกัน |

---

*ต่อไป: Part 28 - Fetch API*
