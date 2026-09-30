# Part 26: Promises (Promises ใน JavaScript)
## ขั้นตอนที่ 491-510

---

## บทนำ

Promise คือวัตถุ (object) ที่แสดงถึงผลลัพธ์ของการดำเนินงานแบบ asynchronous ที่อาจจะสำเร็จหรือล้มเหลวก็ได้ในอนาคต

ก่อนจะมี Promise เราต้องใช้ callback functions ซึ่งทำให้เกิดปัญหาที่เรียกว่า "Callback Hell" หรือ "Pyramid of Doom"

---

## ขั้นตอนที่ 491: ปัญหาของ Callback Hell

### ทำไมถึงต้องมี Promise?

ลองดูตัวอย่างโค้ดที่ใช้ callback:

```javascript
// ตัวอย่างที่ 1: Callback Hell - โค้ดที่อ่านยากมาก
function getUserFromDatabase(userId, callback) {
  setTimeout(() => {
    const user = { id: userId, name: "สมชาย", email: "somchai@example.com" };
    callback(null, user);
  }, 1000);
}

function getOrdersForUser(userId, callback) {
  setTimeout(() => {
    const orders = [
      { id: 1, product: "iPhone", price: 35000 },
      { id: 2, product: "iPad", price: 25000 }
    ];
    callback(null, orders);
  }, 1000);
}

function getProductDetails(orderId, callback) {
  setTimeout(() => {
    const product = { id: orderId, details: "รายละเอียดสินค้า...", inStock: true };
    callback(null, product);
  }, 1000);
}

function sendConfirmationEmail(userId, callback) {
  setTimeout(() => {
    callback(null, { success: true, message: "ส่งอีเมลสำเร็จ" });
  }, 1000);
}

// Callback Hell - ยากต่อการอ่านและดูแลรักษา
getUserFromDatabase(1, function(err, user) {
  if (err) {
    console.error("เกิดข้อผิดพลาดในการดึงข้อมูลผู้ใช้:", err);
    return;
  }
  console.log("ผู้ใช้:", user.name);
  
  getOrdersForUser(user.id, function(err, orders) {
    if (err) {
      console.error("เกิดข้อผิดพลาดในการดึงคำสั่งซื้อ:", err);
      return;
    }
    console.log("คำสั่งซื้อ:", orders.length, "รายการ");
    
    getProductDetails(orders[0].id, function(err, product) {
      if (err) {
        console.error("เกิดข้อผิดพลาดในการดึงรายละเอียดสินค้า:", err);
        return;
      }
      console.log("รายละเอียดสินค้า:", product.details);
      
      sendConfirmationEmail(user.id, function(err, result) {
        if (err) {
          console.error("เกิดข้อผิดพลาดในการส่งอีเมล:", err);
          return;
        }
        console.log("ผลลัพธ์:", result.message);
        // ยิ่งเพิ่ม callback มากขึ้น ยิ่งยากต่อการอ่าน
      });
    });
  });
});
```

```javascript
// ตัวอย่างที่ 2: ปัญหาของ Callback - Error handling ซับซ้อน
function loadScript(src, callback) {
  const script = document.createElement('script');
  script.src = src;
  
  script.onload = function() {
    callback(null, script);
  };
  
  script.onerror = function() {
    callback(new Error(`ไม่สามารถโหลด script จาก ${src}`));
  };
  
  document.head.append(script);
}

// ต้องโหลด 3 scripts ตามลำดับ
loadScript('/js/jquery.js', function(err, script) {
  if (err) {
    console.error(err);
    return;
  }
  loadScript('/js/bootstrap.js', function(err, script) {
    if (err) {
      console.error(err);
      return;
    }
    loadScript('/js/app.js', function(err, script) {
      if (err) {
        console.error(err);
        return;
      }
      // เริ่มทำงานได้แล้ว แต่โค้ดมันลึกมาก!
      initApp();
    });
  });
});
```

---

## ขั้นตอนที่ 492: สถานะของ Promise (Promise States)

Promise มีสถานะได้ 3 แบบ:

```javascript
// ตัวอย่างที่ 3: สถานะของ Promise
// 1. pending - กำลังรอผลลัพธ์ (ยังไม่สำเร็จหรือล้มเหลว)
// 2. fulfilled - สำเร็จ (มีค่าผลลัพธ์)
// 3. rejected - ล้มเหลว (มีเหตุผลของความล้มเหลว)

// เมื่อ Promise อยู่ในสถานะ fulfilled หรือ rejected
// จะเรียกว่า "settled" (สำเร็จแล้ว ไม่ว่าจะเป็นแบบไหน)

// สร้าง Promise ที่อยู่ในสถานะ pending
const pendingPromise = new Promise((resolve, reject) => {
  // ยังไม่เรียก resolve หรือ reject
  // Promise จะอยู่ในสถานะ pending ตลอด
});

console.log(pendingPromise); // Promise { <pending> }
```

```javascript
// ตัวอย่างที่ 4: Promise ที่ fulfilled
const fulfilledPromise = new Promise((resolve, reject) => {
  resolve("สำเร็จแล้ว!"); // เรียก resolve เพื่อทำให้ fulfilled
});

console.log(fulfilledPromise); // Promise { 'สำเร็จแล้ว!' }

fulfilledPromise.then(value => {
  console.log("ค่าที่ได้:", value); // "ค่าที่ได้: สำเร็จแล้ว!"
});
```

```javascript
// ตัวอย่างที่ 5: Promise ที่ rejected
const rejectedPromise = new Promise((resolve, reject) => {
  reject(new Error("เกิดข้อผิดพลาด!")); // เรียก reject เพื่อทำให้ rejected
});

rejectedPromise.catch(error => {
  console.log("ข้อผิดพลาด:", error.message); // "ข้อผิดพลาด: เกิดข้อผิดพลาด!"
});
```

---

## ขั้นตอนที่ 493: การสร้าง Promise ด้วย new Promise()

```javascript
// ตัวอย่างที่ 6: โครงสร้างพื้นฐานของ Promise
const myPromise = new Promise((resolve, reject) => {
  // executor function - รันทันทีเมื่อสร้าง Promise
  
  // ทำงาน asynchronous
  setTimeout(() => {
    const success = true;
    
    if (success) {
      resolve("งานสำเร็จ!"); // เรียกเมื่อทำงานสำเร็จ
    } else {
      reject(new Error("งานล้มเหลว!")); // เรียกเมื่อทำงานล้มเหลว
    }
  }, 1000);
});

// ใช้งาน Promise
myPromise
  .then(result => console.log("ผลลัพธ์:", result))
  .catch(error => console.log("ข้อผิดพลาด:", error.message));
```

```javascript
// ตัวอย่างที่ 7: Promise สำหรับดึงข้อมูลจากฐานข้อมูล
function getUserById(id) {
  return new Promise((resolve, reject) => {
    // จำลองการดึงข้อมูลจาก database
    setTimeout(() => {
      const users = {
        1: { id: 1, name: "สมชาย", age: 25 },
        2: { id: 2, name: "สมหญิง", age: 30 },
        3: { id: 3, name: "ประทีป", age: 22 }
      };
      
      const user = users[id];
      
      if (user) {
        resolve(user); // พบผู้ใช้
      } else {
        reject(new Error(`ไม่พบผู้ใช้ที่มี ID: ${id}`)); // ไม่พบผู้ใช้
      }
    }, 500);
  });
}

// ทดสอบ
getUserById(1)
  .then(user => console.log("พบผู้ใช้:", user.name))
  .catch(err => console.error("ข้อผิดพลาด:", err.message));

getUserById(99)
  .then(user => console.log("พบผู้ใช้:", user.name))
  .catch(err => console.error("ข้อผิดพลาด:", err.message));
```

---

## ขั้นตอนที่ 494: resolve และ reject

```javascript
// ตัวอย่างที่ 8: resolve รับค่าได้หลากหลายประเภท
function getNumber() {
  return new Promise(resolve => {
    resolve(42); // ตัวเลข
  });
}

function getString() {
  return new Promise(resolve => {
    resolve("สวัสดี"); // string
  });
}

function getObject() {
  return new Promise(resolve => {
    resolve({ name: "สมชาย", age: 25 }); // object
  });
}

function getArray() {
  return new Promise(resolve => {
    resolve([1, 2, 3, 4, 5]); // array
  });
}

// ใช้งาน
getNumber().then(n => console.log("ตัวเลข:", n));
getString().then(s => console.log("ข้อความ:", s));
getObject().then(obj => console.log("ออบเจ็กต์:", obj));
getArray().then(arr => console.log("อาร์เรย์:", arr));
```

```javascript
// ตัวอย่างที่ 9: reject ควรใช้ Error object
function validateAge(age) {
  return new Promise((resolve, reject) => {
    if (typeof age !== 'number') {
      reject(new TypeError("อายุต้องเป็นตัวเลข"));
      return;
    }
    
    if (age < 0) {
      reject(new RangeError("อายุต้องไม่ติดลบ"));
      return;
    }
    
    if (age > 150) {
      reject(new RangeError("อายุไม่สมเหตุสมผล"));
      return;
    }
    
    resolve(age);
  });
}

validateAge(25)
  .then(age => console.log("อายุถูกต้อง:", age));

validateAge(-5)
  .catch(err => console.error(`${err.constructor.name}: ${err.message}`));

validateAge("ยี่สิบห้า")
  .catch(err => console.error(`${err.constructor.name}: ${err.message}`));
```

```javascript
// ตัวอย่างที่ 10: resolve และ reject เรียกได้แค่ครั้งเดียว
const promise = new Promise((resolve, reject) => {
  resolve("ผลลัพธ์แรก"); // สถานะเปลี่ยนเป็น fulfilled
  resolve("ผลลัพธ์สอง"); // ถูกละเว้น!
  reject(new Error("ข้อผิดพลาด")); // ถูกละเว้น!
});

promise.then(value => console.log("ค่า:", value)); // "ค่า: ผลลัพธ์แรก"
```

---

## ขั้นตอนที่ 495: เมธอด .then()

```javascript
// ตัวอย่างที่ 11: การใช้ .then() พื้นฐาน
const promise = new Promise(resolve => {
  setTimeout(() => resolve(10), 1000);
});

// .then() รับ callback ที่จะถูกเรียกเมื่อ Promise fulfilled
promise.then(function(value) {
  console.log("ค่าที่ได้:", value); // "ค่าที่ได้: 10"
});

// ใช้ arrow function
promise.then(value => {
  console.log("ค่าที่ได้:", value);
});
```

```javascript
// ตัวอย่างที่ 12: .then() มี 2 arguments
const successPromise = new Promise(resolve => resolve("สำเร็จ"));
const failPromise = new Promise((resolve, reject) => reject(new Error("ล้มเหลว")));

// argument แรกคือ onFulfilled, argument สองคือ onRejected
successPromise.then(
  value => console.log("สำเร็จ:", value),      // จะถูกเรียก
  error => console.log("ล้มเหลว:", error.message) // ไม่ถูกเรียก
);

failPromise.then(
  value => console.log("สำเร็จ:", value),      // ไม่ถูกเรียก
  error => console.log("ล้มเหลว:", error.message) // จะถูกเรียก
);
```

---

## ขั้นตอนที่ 496: เมธอด .catch()

```javascript
// ตัวอย่างที่ 13: การใช้ .catch()
function divideNumbers(a, b) {
  return new Promise((resolve, reject) => {
    if (b === 0) {
      reject(new Error("ไม่สามารถหารด้วย 0 ได้"));
    } else {
      resolve(a / b);
    }
  });
}

// ใช้ .catch() จัดการ error
divideNumbers(10, 2)
  .then(result => console.log("ผลลัพธ์:", result))
  .catch(err => console.error("ข้อผิดพลาด:", err.message));

divideNumbers(10, 0)
  .then(result => console.log("ผลลัพธ์:", result))
  .catch(err => console.error("ข้อผิดพลาด:", err.message));
```

```javascript
// ตัวอย่างที่ 14: .catch() ดักจับ Error ที่ throw ใน .then()
Promise.resolve(1)
  .then(value => {
    console.log("ค่าเริ่มต้น:", value);
    throw new Error("เกิดข้อผิดพลาดใน .then()"); // throw error
    return value * 2; // บรรทัดนี้ไม่ทำงาน
  })
  .then(value => {
    console.log("ค่าที่สอง:", value); // ไม่ถูกเรียก
  })
  .catch(err => {
    console.error("ดักจับได้:", err.message); // "ดักจับได้: เกิดข้อผิดพลาดใน .then()"
  });
```

```javascript
// ตัวอย่างที่ 15: .catch() กลับมาเป็น fulfilled Promise
Promise.reject(new Error("ข้อผิดพลาดเริ่มต้น"))
  .catch(err => {
    console.log("จัดการข้อผิดพลาด:", err.message);
    return "ค่าฟื้นตัว"; // ส่งค่ากลับจาก .catch()
  })
  .then(value => {
    console.log("ต่อจาก .catch():", value); // "ต่อจาก .catch(): ค่าฟื้นตัว"
  });
```

---

## ขั้นตอนที่ 497: เมธอด .finally()

```javascript
// ตัวอย่างที่ 16: การใช้ .finally()
function fetchData(url) {
  return new Promise((resolve, reject) => {
    console.log("กำลังโหลดข้อมูล...");
    setTimeout(() => {
      if (url.includes("error")) {
        reject(new Error("ไม่สามารถโหลดข้อมูลได้"));
      } else {
        resolve({ data: "ข้อมูลจาก " + url });
      }
    }, 1000);
  });
}

// .finally() ทำงานเสมอ ไม่ว่าจะ fulfilled หรือ rejected
fetchData("https://api.example.com/data")
  .then(data => console.log("ข้อมูล:", data))
  .catch(err => console.error("ข้อผิดพลาด:", err.message))
  .finally(() => {
    console.log("เสร็จสิ้นการโหลด (ไม่ว่าจะสำเร็จหรือล้มเหลว)");
    // ใช้สำหรับ cleanup เช่น ปิด loading spinner
  });
```

```javascript
// ตัวอย่างที่ 17: .finally() ไม่รับค่าและส่งค่าต่อ
Promise.resolve("ค่าเริ่มต้น")
  .finally(() => {
    console.log("finally ทำงาน");
    // ส่งค่ากลับจาก finally จะถูกละเว้น!
    return "ค่าจาก finally"; // ถูกละเว้น
  })
  .then(value => {
    console.log("ค่าที่ได้:", value); // "ค่าที่ได้: ค่าเริ่มต้น"
  });

Promise.reject(new Error("ข้อผิดพลาด"))
  .finally(() => {
    console.log("finally ทำงานแม้ rejected");
  })
  .catch(err => {
    console.error("ข้อผิดพลาด:", err.message); // ยังคง reject อยู่
  });
```

```javascript
// ตัวอย่างที่ 18: ใช้ .finally() สำหรับ loading state
async function loadUserProfile(userId) {
  let isLoading = true;
  console.log("กำลังโหลด:", isLoading);
  
  return getUserById(userId)
    .then(user => {
      console.log("ข้อมูลผู้ใช้:", user.name);
      return user;
    })
    .catch(err => {
      console.error("โหลดข้อมูลไม่สำเร็จ:", err.message);
      throw err; // ส่ง error ต่อไป
    })
    .finally(() => {
      isLoading = false;
      console.log("หยุดโหลด:", isLoading);
    });
}
```

---

## ขั้นตอนที่ 498: Promise Chaining (การเชื่อมต่อ Promise)

```javascript
// ตัวอย่างที่ 19: Promise Chaining พื้นฐาน
Promise.resolve(1)
  .then(value => value + 1)   // 1 -> 2
  .then(value => value * 2)   // 2 -> 4
  .then(value => value - 1)   // 4 -> 3
  .then(value => console.log("ผลลัพธ์:", value)); // "ผลลัพธ์: 3"
```

```javascript
// ตัวอย่างที่ 20: Chaining กับ asynchronous operations
function delay(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}

function fetchUser(id) {
  return delay(500).then(() => ({
    id,
    name: "สมชาย",
    orderId: 101
  }));
}

function fetchOrder(orderId) {
  return delay(500).then(() => ({
    id: orderId,
    product: "iPhone 15",
    price: 35000
  }));
}

function calculateTotal(order) {
  return delay(200).then(() => ({
    ...order,
    tax: order.price * 0.07,
    total: order.price * 1.07
  }));
}

// เชื่อมต่อ Promise chain
fetchUser(1)
  .then(user => {
    console.log("ผู้ใช้:", user.name);
    return fetchOrder(user.orderId); // คืน Promise ใหม่
  })
  .then(order => {
    console.log("คำสั่งซื้อ:", order.product);
    return calculateTotal(order);
  })
  .then(result => {
    console.log(`ราคา: ${result.price} บาท`);
    console.log(`ภาษี: ${result.tax.toFixed(2)} บาท`);
    console.log(`รวม: ${result.total.toFixed(2)} บาท`);
  })
  .catch(err => console.error("เกิดข้อผิดพลาด:", err.message));
```

---

## ขั้นตอนที่ 499: การส่งค่ากลับจาก .then()

```javascript
// ตัวอย่างที่ 21: ส่งค่าปกติจาก .then()
Promise.resolve(10)
  .then(value => {
    return value * 2; // ส่งค่าตัวเลขกลับ
  })
  .then(value => {
    console.log("ค่าที่ได้:", value); // 20
    return { number: value, text: `จำนวน ${value}` };
  })
  .then(obj => {
    console.log("ออบเจ็กต์:", obj); // { number: 20, text: 'จำนวน 20' }
  });
```

```javascript
// ตัวอย่างที่ 22: ส่ง Promise กลับจาก .then()
function getPostById(id) {
  return new Promise(resolve => {
    setTimeout(() => {
      resolve({ id, title: `โพสต์ที่ ${id}`, authorId: id * 2 });
    }, 300);
  });
}

function getAuthorById(id) {
  return new Promise(resolve => {
    setTimeout(() => {
      resolve({ id, name: `ผู้เขียน ${id}`, email: `author${id}@example.com` });
    }, 300);
  });
}

// เมื่อ .then() ส่ง Promise กลับ chain จะรอให้ Promise นั้น settle ก่อน
getPostById(5)
  .then(post => {
    console.log("โพสต์:", post.title);
    return getAuthorById(post.authorId); // คืน Promise
  })
  .then(author => {
    // author คือค่าที่ getAuthorById resolve ไป
    console.log("ผู้เขียน:", author.name);
    console.log("อีเมล:", author.email);
  });
```

---

## ขั้นตอนที่ 500: การส่ง Promise กลับจาก .then()

```javascript
// ตัวอย่างที่ 23: การแปลงข้อมูลใน chain
function fetchProducts() {
  return new Promise(resolve => {
    setTimeout(() => {
      resolve([
        { id: 1, name: "iPhone", priceUSD: 999, category: "phone" },
        { id: 2, name: "iPad", priceUSD: 799, category: "tablet" },
        { id: 3, name: "MacBook", priceUSD: 1299, category: "laptop" }
      ]);
    }, 500);
  });
}

function getExchangeRate() {
  return new Promise(resolve => {
    setTimeout(() => {
      resolve(35.5); // 1 USD = 35.5 THB
    }, 300);
  });
}

fetchProducts()
  .then(products => {
    return getExchangeRate().then(rate => {
      // แปลงราคาเป็น THB
      return products.map(p => ({
        ...p,
        priceTHB: (p.priceUSD * rate).toFixed(2)
      }));
    });
  })
  .then(productsWithTHB => {
    productsWithTHB.forEach(p => {
      console.log(`${p.name}: $${p.priceUSD} = ฿${p.priceTHB}`);
    });
  });
```

---

## ขั้นตอนที่ 501: การจัดการ Error ใน Promise Chains

```javascript
// ตัวอย่างที่ 24: Error propagation ใน chain
function step1() {
  return Promise.resolve("ขั้นตอนที่ 1 สำเร็จ");
}

function step2(input) {
  return new Promise((resolve, reject) => {
    reject(new Error("ขั้นตอนที่ 2 ล้มเหลว!"));
  });
}

function step3(input) {
  return Promise.resolve("ขั้นตอนที่ 3 สำเร็จ");
}

step1()
  .then(result => {
    console.log(result);
    return step2(result);
  })
  .then(result => {
    // ข้ามขั้นตอนนี้เพราะ step2 rejected
    console.log(result);
    return step3(result);
  })
  .then(result => {
    // ข้ามขั้นตอนนี้ด้วย
    console.log(result);
  })
  .catch(err => {
    // จัดการ error ทั้งหมดที่นี่
    console.error("ดักจับ error:", err.message);
  });
```

```javascript
// ตัวอย่างที่ 25: จัดการ Error แบบละเอียด
function fetchUserData(userId) {
  return new Promise((resolve, reject) => {
    if (!userId) {
      reject(new TypeError("ต้องระบุ userId"));
      return;
    }
    setTimeout(() => resolve({ id: userId, name: "สมชาย" }), 200);
  });
}

function processUserData(user) {
  return new Promise((resolve, reject) => {
    if (!user.email) {
      reject(new Error("ผู้ใช้ไม่มีอีเมล"));
      return;
    }
    resolve({ ...user, processed: true });
  });
}

// จัดการ error ต่างประเภทในจุดต่างกัน
fetchUserData(1)
  .then(user => processUserData(user))
  .then(processed => console.log("ประมวลผลสำเร็จ:", processed))
  .catch(err => {
    if (err instanceof TypeError) {
      console.error("ข้อมูลไม่ถูกต้อง:", err.message);
    } else {
      console.error("ข้อผิดพลาดทั่วไป:", err.message);
    }
  });
```

---

## ขั้นตอนที่ 502: Promise.resolve() และ Promise.reject()

```javascript
// ตัวอย่างที่ 26: Promise.resolve()
// สร้าง Promise ที่ fulfilled ทันที
const p1 = Promise.resolve(42);
const p2 = Promise.resolve("สวัสดี");
const p3 = Promise.resolve({ name: "สมชาย" });

p1.then(v => console.log("p1:", v));
p2.then(v => console.log("p2:", v));
p3.then(v => console.log("p3:", v.name));
```

```javascript
// ตัวอย่างที่ 27: Promise.resolve() กับ Promise อื่น
const existingPromise = new Promise(resolve => {
  setTimeout(() => resolve("ค่าจาก Promise เดิม"), 500);
});

// Promise.resolve() กับ Promise จะคืน Promise นั้นเลย
const wrapped = Promise.resolve(existingPromise);
console.log(wrapped === existingPromise); // true

wrapped.then(value => console.log("ค่า:", value)); // "ค่า: ค่าจาก Promise เดิม"
```

```javascript
// ตัวอย่างที่ 28: Promise.reject()
const rejected = Promise.reject(new Error("ข้อผิดพลาดทันที"));

rejected.catch(err => console.error("Error:", err.message));

// ใช้ประโยชน์ใน function ที่อาจ return Promise หรือ non-Promise
function maybeAsync(value) {
  if (value < 0) {
    return Promise.reject(new RangeError("ค่าต้องไม่ติดลบ"));
  }
  return Promise.resolve(value * 2);
}

maybeAsync(5).then(v => console.log("ผลลัพธ์:", v));
maybeAsync(-1).catch(err => console.error("ข้อผิดพลาด:", err.message));
```

---

## ขั้นตอนที่ 503: Promise.all() - การรันแบบขนาน

```javascript
// ตัวอย่างที่ 29: Promise.all() พื้นฐาน
const p1 = new Promise(resolve => setTimeout(() => resolve("A"), 300));
const p2 = new Promise(resolve => setTimeout(() => resolve("B"), 100));
const p3 = new Promise(resolve => setTimeout(() => resolve("C"), 200));

// รอทุก Promise ให้ fulfilled ก่อน
Promise.all([p1, p2, p3])
  .then(results => {
    console.log("ผลลัพธ์:", results); // ["A", "B", "C"] - ตามลำดับ ไม่ใช่ตามเวลา
  });
```

```javascript
// ตัวอย่างที่ 30: Promise.all() กับ real-world scenario
function fetchUserProfile(userId) {
  return new Promise(resolve => {
    setTimeout(() => resolve({ id: userId, name: "สมชาย", age: 25 }), 400);
  });
}

function fetchUserOrders(userId) {
  return new Promise(resolve => {
    setTimeout(() => resolve([{ id: 1, product: "iPhone" }, { id: 2, product: "iPad" }]), 600);
  });
}

function fetchUserAddresses(userId) {
  return new Promise(resolve => {
    setTimeout(() => resolve([{ id: 1, street: "ถนนสุขุมวิท", city: "กรุงเทพ" }]), 300);
  });
}

console.time("ดึงข้อมูลทั้งหมด");

// ดึงข้อมูลทั้งหมดพร้อมกัน (ใช้เวลาสูงสุด 600ms ไม่ใช่ 1300ms)
Promise.all([
  fetchUserProfile(1),
  fetchUserOrders(1),
  fetchUserAddresses(1)
])
.then(([profile, orders, addresses]) => {
  console.timeEnd("ดึงข้อมูลทั้งหมด");
  console.log("โปรไฟล์:", profile.name);
  console.log("คำสั่งซื้อ:", orders.length, "รายการ");
  console.log("ที่อยู่:", addresses[0].city);
});
```

```javascript
// ตัวอย่างที่ 31: Promise.all() กับ rejection
const promises = [
  Promise.resolve("A"),
  Promise.reject(new Error("B ล้มเหลว")),
  Promise.resolve("C")
];

// Promise.all() จะ reject ทันทีที่มี Promise ใดๆ reject
Promise.all(promises)
  .then(results => console.log("สำเร็จ:", results))
  .catch(err => console.error("ล้มเหลว:", err.message)); // "ล้มเหลว: B ล้มเหลว"
```

---

## ขั้นตอนที่ 504: Promise.allSettled() - รับผลทั้งหมดไม่ว่าจะเป็นอะไร

```javascript
// ตัวอย่างที่ 32: Promise.allSettled()
const promises = [
  Promise.resolve("สำเร็จ A"),
  Promise.reject(new Error("ล้มเหลว B")),
  Promise.resolve("สำเร็จ C"),
  Promise.reject(new Error("ล้มเหลว D"))
];

// รอทุก Promise ให้ settle (ไม่ว่าจะ fulfilled หรือ rejected)
Promise.allSettled(promises)
  .then(results => {
    results.forEach((result, index) => {
      if (result.status === "fulfilled") {
        console.log(`Promise ${index}: สำเร็จ -`, result.value);
      } else {
        console.log(`Promise ${index}: ล้มเหลว -`, result.reason.message);
      }
    });
  });

// Output:
// Promise 0: สำเร็จ - สำเร็จ A
// Promise 1: ล้มเหลว - ล้มเหลว B
// Promise 2: สำเร็จ - สำเร็จ C
// Promise 3: ล้มเหลว - ล้มเหลว D
```

```javascript
// ตัวอย่างที่ 33: ใช้ allSettled เพื่อ report ผลทั้งหมด
function sendEmailToUser(userId) {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      // จำลองว่าบางคนส่งสำเร็จ บางคนไม่
      if (userId % 2 === 0) {
        resolve({ userId, status: "ส่งสำเร็จ" });
      } else {
        reject(new Error(`ผู้ใช้ ${userId}: ไม่สามารถส่งอีเมลได้`));
      }
    }, Math.random() * 500);
  });
}

const userIds = [1, 2, 3, 4, 5];

Promise.allSettled(userIds.map(id => sendEmailToUser(id)))
  .then(results => {
    const successful = results.filter(r => r.status === "fulfilled");
    const failed = results.filter(r => r.status === "rejected");
    
    console.log(`ส่งสำเร็จ: ${successful.length} คน`);
    console.log(`ส่งไม่สำเร็จ: ${failed.length} คน`);
    
    failed.forEach(f => console.error("ข้อผิดพลาด:", f.reason.message));
  });
```

---

## ขั้นตอนที่ 505: Promise.race() - อันแรกที่ settle

```javascript
// ตัวอย่างที่ 34: Promise.race() พื้นฐาน
const fast = new Promise(resolve => setTimeout(() => resolve("เร็ว"), 100));
const slow = new Promise(resolve => setTimeout(() => resolve("ช้า"), 1000));
const medium = new Promise(resolve => setTimeout(() => resolve("กลาง"), 500));

// คืนผลของ Promise แรกที่ settle
Promise.race([fast, slow, medium])
  .then(result => console.log("ชนะ:", result)); // "ชนะ: เร็ว"
```

```javascript
// ตัวอย่างที่ 35: ใช้ race เป็น timeout
function fetchWithTimeout(url, timeoutMs) {
  const fetchPromise = new Promise(resolve => {
    // จำลองการ fetch
    setTimeout(() => resolve(`ข้อมูลจาก ${url}`), 2000);
  });
  
  const timeoutPromise = new Promise((_, reject) => {
    setTimeout(() => {
      reject(new Error(`Timeout: ${url} ใช้เวลาเกิน ${timeoutMs}ms`));
    }, timeoutMs);
  });
  
  return Promise.race([fetchPromise, timeoutPromise]);
}

// Timeout 3 วินาที - สำเร็จ
fetchWithTimeout("https://api.example.com/data", 3000)
  .then(data => console.log("ได้รับ:", data))
  .catch(err => console.error("ข้อผิดพลาด:", err.message));

// Timeout 1 วินาที - timeout
fetchWithTimeout("https://api.slow.com/data", 1000)
  .then(data => console.log("ได้รับ:", data))
  .catch(err => console.error("ข้อผิดพลาด:", err.message));
```

```javascript
// ตัวอย่างที่ 36: race กับ rejected Promise
const p1 = new Promise((_, reject) => setTimeout(() => reject(new Error("A ล้มเหลว")), 100));
const p2 = new Promise(resolve => setTimeout(() => resolve("B สำเร็จ"), 500));

Promise.race([p1, p2])
  .then(result => console.log("ผลลัพธ์:", result))
  .catch(err => console.error("ข้อผิดพลาด:", err.message)); // "ข้อผิดพลาด: A ล้มเหลว"
```

---

## ขั้นตอนที่ 506: Promise.any() - อันแรกที่ fulfill

```javascript
// ตัวอย่างที่ 37: Promise.any()
const p1 = new Promise((_, reject) => setTimeout(() => reject(new Error("A ล้มเหลว")), 100));
const p2 = new Promise(resolve => setTimeout(() => resolve("B สำเร็จ"), 500));
const p3 = new Promise((_, reject) => setTimeout(() => reject(new Error("C ล้มเหลว")), 300));

// คืนผลของ Promise แรกที่ fulfill (ไม่สนใจ rejected)
Promise.any([p1, p2, p3])
  .then(result => console.log("อันแรกที่สำเร็จ:", result)); // "อันแรกที่สำเร็จ: B สำเร็จ"
```

```javascript
// ตัวอย่างที่ 38: Promise.any() กับ AggregateError
const allFail = [
  Promise.reject(new Error("ล้มเหลว A")),
  Promise.reject(new Error("ล้มเหลว B")),
  Promise.reject(new Error("ล้มเหลว C"))
];

// ถ้าทุก Promise rejected จะ throw AggregateError
Promise.any(allFail)
  .then(result => console.log("สำเร็จ:", result))
  .catch(err => {
    console.error("ประเภท:", err.constructor.name); // "AggregateError"
    console.error("ข้อความ:", err.message); // "All promises were rejected"
    console.error("ข้อผิดพลาดทั้งหมด:", err.errors);
  });
```

```javascript
// ตัวอย่างที่ 39: ใช้ any สำหรับ fallback servers
function fetchFromServer(serverUrl) {
  return new Promise((resolve, reject) => {
    const delay = serverUrl.includes("fast") ? 200 : 
                  serverUrl.includes("slow") ? 2000 : 500;
    setTimeout(() => {
      if (serverUrl.includes("down")) {
        reject(new Error(`เซิร์ฟเวอร์ ${serverUrl} ไม่พร้อมใช้งาน`));
      } else {
        resolve(`ข้อมูลจาก ${serverUrl}`);
      }
    }, delay);
  });
}

// ลองดึงข้อมูลจากหลายเซิร์ฟเวอร์ เอาอันแรกที่ตอบสนอง
Promise.any([
  fetchFromServer("https://server1-down.com"),
  fetchFromServer("https://server2-slow.com"),
  fetchFromServer("https://server3-fast.com")
])
.then(data => console.log("ได้ข้อมูล:", data))
.catch(err => console.error("ทุกเซิร์ฟเวอร์ล้มเหลว:", err.message));
```

---

## ขั้นตอนที่ 507: การแปลง Callback เป็น Promise (Promisifying)

```javascript
// ตัวอย่างที่ 40: Promisify Node.js callback
// รูปแบบ callback ของ Node.js: function(error, result)

// callback-based function (ลักษณะ Node.js)
function readFileCallback(path, callback) {
  setTimeout(() => {
    if (path.includes("not-found")) {
      callback(new Error(`ไม่พบไฟล์: ${path}`));
    } else {
      callback(null, `เนื้อหาของ ${path}`);
    }
  }, 500);
}

// แปลงเป็น Promise-based
function readFilePromise(path) {
  return new Promise((resolve, reject) => {
    readFileCallback(path, (error, data) => {
      if (error) {
        reject(error);
      } else {
        resolve(data);
      }
    });
  });
}

// ใช้งาน
readFilePromise("document.txt")
  .then(content => console.log("เนื้อหา:", content))
  .catch(err => console.error("ข้อผิดพลาด:", err.message));
```

```javascript
// ตัวอย่างที่ 41: สร้าง utility function สำหรับ promisify
function promisify(callbackFn) {
  return function(...args) {
    return new Promise((resolve, reject) => {
      callbackFn(...args, (error, result) => {
        if (error) {
          reject(error);
        } else {
          resolve(result);
        }
      });
    });
  };
}

// ใช้ promisify กับ callback functions ต่างๆ
const readFileProm = promisify(readFileCallback);

readFileProm("config.json")
  .then(content => console.log("อ่านสำเร็จ:", content))
  .catch(err => console.error("ข้อผิดพลาด:", err.message));
```

```javascript
// ตัวอย่างที่ 42: Promisify setTimeout
function wait(ms) {
  return new Promise(resolve => setTimeout(resolve, ms));
}

// ใช้งาน
console.log("เริ่ม");
wait(1000)
  .then(() => console.log("รอ 1 วินาที"))
  .then(() => wait(500))
  .then(() => console.log("รออีก 0.5 วินาที"))
  .then(() => console.log("เสร็จ"));
```

---

## ขั้นตอนที่ 508: ตัวอย่าง Real-World Promises

```javascript
// ตัวอย่างที่ 43: Shopping Cart System
class ShoppingCart {
  constructor() {
    this.items = [];
  }
  
  addItem(productId, quantity) {
    return this.fetchProduct(productId)
      .then(product => {
        return this.checkStock(product, quantity)
          .then(available => {
            if (!available) {
              throw new Error(`สินค้า ${product.name} ไม่เพียงพอ`);
            }
            this.items.push({ product, quantity });
            return this;
          });
      });
  }
  
  fetchProduct(id) {
    return new Promise(resolve => {
      setTimeout(() => {
        const products = {
          1: { id: 1, name: "iPhone 15", price: 35000 },
          2: { id: 2, name: "AirPods", price: 8500 }
        };
        resolve(products[id] || null);
      }, 200);
    });
  }
  
  checkStock(product, quantity) {
    return new Promise(resolve => {
      setTimeout(() => {
        const stock = { 1: 10, 2: 5 };
        resolve((stock[product.id] || 0) >= quantity);
      }, 100);
    });
  }
  
  checkout() {
    const total = this.items.reduce((sum, item) => 
      sum + (item.product.price * item.quantity), 0);
    
    return this.processPayment(total)
      .then(payment => {
        return this.sendOrderConfirmation(payment);
      });
  }
  
  processPayment(amount) {
    return new Promise(resolve => {
      setTimeout(() => {
        resolve({
          success: true,
          transactionId: `TXN${Date.now()}`,
          amount
        });
      }, 800);
    });
  }
  
  sendOrderConfirmation(payment) {
    return new Promise(resolve => {
      setTimeout(() => {
        resolve({
          orderId: `ORD${Date.now()}`,
          transactionId: payment.transactionId,
          amount: payment.amount,
          message: "คำสั่งซื้อสำเร็จ!"
        });
      }, 300);
    });
  }
}

// ใช้งาน
const cart = new ShoppingCart();

cart.addItem(1, 2)
  .then(() => cart.addItem(2, 1))
  .then(() => cart.checkout())
  .then(order => {
    console.log("หมายเลขคำสั่งซื้อ:", order.orderId);
    console.log("ยอดรวม:", order.amount, "บาท");
    console.log(order.message);
  })
  .catch(err => console.error("ข้อผิดพลาด:", err.message));
```

```javascript
// ตัวอย่างที่ 44: Weather App ที่ใช้ Promise
function getCurrentLocation() {
  return new Promise((resolve, reject) => {
    // จำลอง geolocation
    setTimeout(() => {
      resolve({ lat: 13.7563, lon: 100.5018 }); // กรุงเทพ
    }, 500);
  });
}

function getWeatherByCoords(lat, lon) {
  return new Promise(resolve => {
    setTimeout(() => {
      resolve({
        city: "กรุงเทพมหานคร",
        temp: 32,
        humidity: 80,
        description: "มีเมฆบางส่วน",
        wind: 15
      });
    }, 800);
  });
}

function getForecast(city) {
  return new Promise(resolve => {
    setTimeout(() => {
      resolve([
        { day: "พรุ่งนี้", temp: 31, desc: "ฝนตก" },
        { day: "มะรืน", temp: 30, desc: "มีเมฆ" },
        { day: "3 วัน", temp: 33, desc: "แดดออก" }
      ]);
    }, 600);
  });
}

function displayWeather() {
  console.log("กำลังค้นหาตำแหน่ง...");
  
  getCurrentLocation()
    .then(coords => {
      console.log(`ตำแหน่ง: ${coords.lat}, ${coords.lon}`);
      return Promise.all([
        getWeatherByCoords(coords.lat, coords.lon),
        // ดึงพยากรณ์ไปพร้อมกัน แต่ต้องรู้ city ก่อน
      ]);
    })
    .then(([weather]) => {
      console.log(`\nสภาพอากาศ: ${weather.city}`);
      console.log(`อุณหภูมิ: ${weather.temp}°C`);
      console.log(`ความชื้น: ${weather.humidity}%`);
      console.log(`สภาพ: ${weather.description}`);
      
      return getForecast(weather.city);
    })
    .then(forecast => {
      console.log("\nพยากรณ์อากาศ:");
      forecast.forEach(f => console.log(`  ${f.day}: ${f.temp}°C - ${f.desc}`));
    })
    .catch(err => console.error("ข้อผิดพลาด:", err.message));
}

displayWeather();
```

---

## ขั้นตอนที่ 509: รูปแบบ Promise ขั้นสูง

```javascript
// ตัวอย่างที่ 45: Sequential Promise execution
async function executeSequentially(tasks) {
  const results = [];
  
  for (const task of tasks) {
    const result = await task();
    results.push(result);
  }
  
  return results;
}

// หรือด้วย reduce
function runSequentially(promises) {
  return promises.reduce((chain, promiseFn) => {
    return chain.then(results => {
      return promiseFn().then(result => [...results, result]);
    });
  }, Promise.resolve([]));
}

const tasks = [
  () => new Promise(resolve => setTimeout(() => resolve("ขั้นตอน 1"), 300)),
  () => new Promise(resolve => setTimeout(() => resolve("ขั้นตอน 2"), 200)),
  () => new Promise(resolve => setTimeout(() => resolve("ขั้นตอน 3"), 100))
];

runSequentially(tasks)
  .then(results => console.log("ผลลัพธ์:", results));
```

```javascript
// ตัวอย่างที่ 46: Promise Queue - จำกัดจำนวน concurrent requests
function createQueue(concurrency) {
  let running = 0;
  const queue = [];
  
  function runNext() {
    if (running >= concurrency || queue.length === 0) return;
    
    running++;
    const { task, resolve, reject } = queue.shift();
    
    task()
      .then(resolve)
      .catch(reject)
      .finally(() => {
        running--;
        runNext();
      });
  }
  
  return function enqueue(task) {
    return new Promise((resolve, reject) => {
      queue.push({ task, resolve, reject });
      runNext();
    });
  };
}

// จำกัด 2 requests พร้อมกัน
const queue = createQueue(2);

const tasks = Array.from({ length: 6 }, (_, i) => {
  return queue(() => new Promise(resolve => {
    console.log(`เริ่ม Task ${i + 1}`);
    setTimeout(() => {
      console.log(`เสร็จ Task ${i + 1}`);
      resolve(`ผลลัพธ์ ${i + 1}`);
    }, Math.random() * 1000);
  }));
});

Promise.all(tasks).then(results => {
  console.log("ทุก task เสร็จแล้ว:", results);
});
```

```javascript
// ตัวอย่างที่ 47: Retry Pattern
function retryPromise(promiseFn, maxRetries, delay = 1000) {
  return new Promise((resolve, reject) => {
    let attempts = 0;
    
    function attempt() {
      attempts++;
      console.log(`ความพยายามครั้งที่ ${attempts}`);
      
      promiseFn()
        .then(resolve)
        .catch(err => {
          if (attempts >= maxRetries) {
            reject(new Error(`ล้มเหลวหลังจาก ${attempts} ครั้ง: ${err.message}`));
          } else {
            console.log(`รอ ${delay}ms แล้วลองใหม่...`);
            setTimeout(attempt, delay);
          }
        });
    }
    
    attempt();
  });
}

// ฟังก์ชันที่ล้มเหลว 2 ครั้งแรก
let callCount = 0;
function unreliableApi() {
  callCount++;
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      if (callCount < 3) {
        reject(new Error(`API ล้มเหลวครั้งที่ ${callCount}`));
      } else {
        resolve("API สำเร็จ!");
      }
    }, 200);
  });
}

retryPromise(unreliableApi, 5, 500)
  .then(result => console.log("สำเร็จ:", result))
  .catch(err => console.error("ล้มเหลว:", err.message));
```

---

## ขั้นตอนที่ 510: แบบฝึกหัด

```javascript
// แบบฝึกหัดที่ 1: สร้าง Promise สำหรับ coin flip
// สร้างฟังก์ชัน flipCoin() ที่คืน Promise
// - 50% โอกาสได้ "หัว" (resolve)
// - 50% โอกาสได้ "ก้อย" (reject)

function flipCoin() {
  return new Promise((resolve, reject) => {
    setTimeout(() => {
      const result = Math.random() < 0.5 ? "หัว" : "ก้อย";
      if (result === "หัว") {
        resolve("หัว - คุณชนะ!");
      } else {
        reject(new Error("ก้อย - คุณแพ้!"));
      }
    }, 500);
  });
}

// ทดสอบ 5 ครั้ง
Promise.allSettled(
  Array.from({ length: 5 }, () => flipCoin())
).then(results => {
  const wins = results.filter(r => r.status === "fulfilled").length;
  const losses = results.filter(r => r.status === "rejected").length;
  console.log(`ชนะ: ${wins} ครั้ง, แพ้: ${losses} ครั้ง`);
});
```

```javascript
// แบบฝึกหัดที่ 2: Promise Chain สำหรับระบบสมาชิก
// สร้าง chain ที่:
// 1. ตรวจสอบ email ว่าถูกต้องไหม
// 2. ตรวจสอบว่า email นี้ไม่ได้ใช้แล้ว
// 3. สร้าง account
// 4. ส่ง welcome email

function validateEmail(email) {
  return new Promise((resolve, reject) => {
    const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
    if (emailRegex.test(email)) {
      resolve(email);
    } else {
      reject(new Error("รูปแบบ email ไม่ถูกต้อง"));
    }
  });
}

function checkEmailAvailability(email) {
  return new Promise((resolve, reject) => {
    const usedEmails = ["used@example.com", "taken@example.com"];
    setTimeout(() => {
      if (usedEmails.includes(email)) {
        reject(new Error("Email นี้ถูกใช้ไปแล้ว"));
      } else {
        resolve(email);
      }
    }, 300);
  });
}

function createAccount(email) {
  return new Promise(resolve => {
    setTimeout(() => {
      resolve({
        id: Math.floor(Math.random() * 1000),
        email,
        createdAt: new Date().toISOString()
      });
    }, 500);
  });
}

function sendWelcomeEmail(account) {
  return new Promise(resolve => {
    setTimeout(() => {
      console.log(`ส่ง welcome email ไปยัง ${account.email}`);
      resolve({ ...account, welcomeEmailSent: true });
    }, 200);
  });
}

// ทดสอบ
function registerUser(email) {
  return validateEmail(email)
    .then(validEmail => checkEmailAvailability(validEmail))
    .then(availableEmail => createAccount(availableEmail))
    .then(account => sendWelcomeEmail(account))
    .then(account => {
      console.log(`สมัครสมาชิกสำเร็จ! ID: ${account.id}`);
      return account;
    });
}

registerUser("newuser@example.com")
  .then(account => console.log("บัญชีใหม่:", account))
  .catch(err => console.error("ข้อผิดพลาด:", err.message));

registerUser("invalid-email")
  .catch(err => console.error("ข้อผิดพลาด:", err.message));

registerUser("used@example.com")
  .catch(err => console.error("ข้อผิดพลาด:", err.message));
```

```javascript
// แบบฝึกหัดที่ 3: Promise.all สำหรับ Dashboard
// สร้าง dashboard ที่ดึงข้อมูล 4 อย่างพร้อมกัน:
// - ยอดขายวันนี้
// - จำนวนผู้ใช้งาน
// - คำสั่งซื้อที่รอดำเนินการ
// - รายได้เดือนนี้

function getSalesToday() {
  return new Promise(resolve => {
    setTimeout(() => resolve({ total: 125000, count: 47 }), 300);
  });
}

function getActiveUsers() {
  return new Promise(resolve => {
    setTimeout(() => resolve({ online: 234, total: 1520 }), 500);
  });
}

function getPendingOrders() {
  return new Promise(resolve => {
    setTimeout(() => resolve({ count: 18, urgent: 3 }), 200);
  });
}

function getMonthlyRevenue() {
  return new Promise(resolve => {
    setTimeout(() => resolve({ revenue: 2850000, growth: 12.5 }), 600);
  });
}

console.time("Dashboard Load");

Promise.all([
  getSalesToday(),
  getActiveUsers(),
  getPendingOrders(),
  getMonthlyRevenue()
]).then(([sales, users, orders, revenue]) => {
  console.timeEnd("Dashboard Load");
  console.log("\n=== Dashboard ===");
  console.log(`ยอดขายวันนี้: ฿${sales.total.toLocaleString()} (${sales.count} รายการ)`);
  console.log(`ผู้ใช้งานตอนนี้: ${users.online} / ${users.total} คน`);
  console.log(`คำสั่งซื้อรอดำเนินการ: ${orders.count} รายการ (เร่งด่วน: ${orders.urgent})`);
  console.log(`รายได้เดือนนี้: ฿${revenue.revenue.toLocaleString()} (+${revenue.growth}%)`);
});
```

```javascript
// แบบฝึกหัดที่ 4: สร้าง cache ด้วย Promise
class PromiseCache {
  constructor(ttl = 5000) {
    this.cache = new Map();
    this.ttl = ttl;
  }
  
  get(key, fetchFn) {
    const cached = this.cache.get(key);
    
    if (cached && Date.now() - cached.timestamp < this.ttl) {
      console.log(`Cache hit: ${key}`);
      return Promise.resolve(cached.value);
    }
    
    console.log(`Cache miss: ${key}`);
    return fetchFn().then(value => {
      this.cache.set(key, { value, timestamp: Date.now() });
      return value;
    });
  }
  
  invalidate(key) {
    this.cache.delete(key);
  }
  
  clear() {
    this.cache.clear();
  }
}

// ใช้งาน
const cache = new PromiseCache(3000);

function fetchUserFromApi(id) {
  console.log(`API call สำหรับ user ${id}`);
  return new Promise(resolve => {
    setTimeout(() => {
      resolve({ id, name: `ผู้ใช้ ${id}`, email: `user${id}@example.com` });
    }, 500);
  });
}

// เรียกครั้งแรก - ดึงจาก API
cache.get(`user-1`, () => fetchUserFromApi(1))
  .then(user => console.log("ครั้งที่ 1:", user.name));

// เรียกครั้งที่สอง - ดึงจาก cache
setTimeout(() => {
  cache.get(`user-1`, () => fetchUserFromApi(1))
    .then(user => console.log("ครั้งที่ 2 (จาก cache):", user.name));
}, 1000);
```

```javascript
// แบบฝึกหัดที่ 5: Promise-based Event System
class EventEmitter {
  constructor() {
    this.listeners = new Map();
  }
  
  on(event, callback) {
    if (!this.listeners.has(event)) {
      this.listeners.set(event, []);
    }
    this.listeners.get(event).push(callback);
  }
  
  emit(event, data) {
    const callbacks = this.listeners.get(event) || [];
    const promises = callbacks.map(cb => Promise.resolve(cb(data)));
    return Promise.all(promises);
  }
  
  once(event) {
    return new Promise(resolve => {
      const handler = (data) => {
        this.listeners.get(event).splice(
          this.listeners.get(event).indexOf(handler), 1
        );
        resolve(data);
      };
      this.on(event, handler);
    });
  }
}

// ใช้งาน
const emitter = new EventEmitter();

emitter.on("userLogin", user => {
  console.log(`ผู้ใช้ ${user.name} เข้าสู่ระบบ`);
  return `บันทึก: ${user.name} เข้าสู่ระบบ`;
});

emitter.on("userLogin", user => {
  console.log(`ส่ง notification ให้ ${user.name}`);
  return `Notification ส่งแล้ว`;
});

// รอ event ครั้งเดียว
emitter.once("specialEvent").then(data => {
  console.log("Special event:", data);
});

// ยิง event
emitter.emit("userLogin", { id: 1, name: "สมชาย" })
  .then(results => console.log("Handlers ทำงานแล้ว:", results));

emitter.emit("specialEvent", "เหตุการณ์พิเศษ!");
```

---

## สรุป Part 26: Promises

| เมธอด | คำอธิบาย |
|-------|-----------|
| `new Promise(fn)` | สร้าง Promise ใหม่ |
| `resolve(value)` | ทำให้ Promise fulfilled |
| `reject(error)` | ทำให้ Promise rejected |
| `.then(fn)` | จัดการผลลัพธ์เมื่อ fulfilled |
| `.catch(fn)` | จัดการ error เมื่อ rejected |
| `.finally(fn)` | ทำงานเสมอ ไม่ว่าจะ fulfilled หรือ rejected |
| `Promise.resolve()` | สร้าง fulfilled Promise ทันที |
| `Promise.reject()` | สร้าง rejected Promise ทันที |
| `Promise.all([])` | รอทุก Promise, fail fast |
| `Promise.allSettled([])` | รอทุก Promise, รับผลทั้งหมด |
| `Promise.race([])` | คืนผลของ Promise แรกที่ settle |
| `Promise.any([])` | คืนผลของ Promise แรกที่ fulfilled |

---

*ต่อไป: Part 27 - Async/Await*
