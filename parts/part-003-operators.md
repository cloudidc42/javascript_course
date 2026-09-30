# ส่วนที่ 3: ตัวดำเนินการ (Operators)

## คำอธิบาย
ในส่วนนี้คุณจะได้เรียนรู้ตัวดำเนินการทุกประเภทใน JavaScript ตั้งแต่ arithmetic จนถึง nullish coalescing พร้อมเข้าใจ operator precedence เนื้อหาครอบคลุม Step 31-50

---

## Step 31: Arithmetic Operators (ตัวดำเนินการทางคณิตศาสตร์)

```javascript
// ตัวอย่างที่ 1: Arithmetic Operators พื้นฐาน
let a = 10;
let b = 3;

console.log(a + b);  // 13  (บวก)
console.log(a - b);  // 7   (ลบ)
console.log(a * b);  // 30  (คูณ)
console.log(a / b);  // 3.3333... (หาร)
console.log(a % b);  // 1   (เศษจากการหาร / modulo)
console.log(a ** b); // 1000 (ยกกำลัง ES2016)

// ตัวอย่างที่ 2: Modulo use cases
// ตรวจสอบเลขคู่/คี่
console.log(4 % 2 === 0);  // true  (เลขคู่)
console.log(7 % 2 === 0);  // false (เลขคี่)

// วน cycle (0, 1, 2, 0, 1, 2, ...)
for (let i = 0; i < 9; i++) {
    process.stdout.write(`${i % 3} `);
}
// 0 1 2 0 1 2 0 1 2

// ตรวจสอบปีอธิกสุรทิน
function isLeapYear(year) {
    return (year % 4 === 0 && year % 100 !== 0) || (year % 400 === 0);
}
console.log(isLeapYear(2024)); // true
console.log(isLeapYear(1900)); // false
console.log(isLeapYear(2000)); // true
```

```javascript
// ตัวอย่างที่ 3: Exponentiation (**) operator
console.log(2 ** 10);   // 1024
console.log(2 ** 0.5);  // 1.414... (square root)
console.log((-2) ** 3); // -8
console.log(4 ** 0.5);  // 2

// เทียบกับ Math.pow
console.log(Math.pow(2, 10)); // 1024 (เหมือนกัน)

// ตัวอย่างที่ 4: Unary operators
let x = 5;
console.log(+x);   // 5  (unary plus)
console.log(-x);   // -5 (unary minus)
console.log(+'42');  // 42  (แปลง string เป็น number)
console.log(+'');    // 0
console.log(+true);  // 1
console.log(+false); // 0
console.log(+null);  // 0
console.log(+'abc'); // NaN

// ตัวอย่างที่ 5: Increment/Decrement
let n = 5;
console.log(n++); // 5 (post-increment: คืนค่าก่อน แล้วค่อยเพิ่ม)
console.log(n);   // 6
console.log(++n); // 7 (pre-increment: เพิ่มก่อน แล้วค่อยคืนค่า)
console.log(n--); // 7 (post-decrement)
console.log(n);   // 6
console.log(--n); // 5 (pre-decrement)
```

```javascript
// ตัวอย่างที่ 6: Arithmetic กับ String (+ operator)
console.log(1 + '2');      // "12"
console.log('1' + 2);      // "12"
console.log(1 + 2 + '3'); // "33" (1+2=3 แล้ว 3+'3')
console.log('1' + 2 + 3); // "123"
console.log('5' - 3);     // 2 (string แปลงเป็น number สำหรับ -)
console.log('5' * '3');   // 15
console.log('10' / '2');  // 5
console.log('abc' - 1);   // NaN

// ตัวอย่างที่ 7: โปรแกรมคำนวณสูตรคณิตศาสตร์
function quadratic(a, b, c) {
    // คำนวณ x จาก ax² + bx + c = 0
    const discriminant = b ** 2 - 4 * a * c;
    
    if (discriminant < 0) {
        return 'ไม่มีคำตอบจริง';
    } else if (discriminant === 0) {
        const x = -b / (2 * a);
        return `x = ${x}`;
    } else {
        const x1 = (-b + Math.sqrt(discriminant)) / (2 * a);
        const x2 = (-b - Math.sqrt(discriminant)) / (2 * a);
        return `x1 = ${x1.toFixed(4)}, x2 = ${x2.toFixed(4)}`;
    }
}

console.log(quadratic(1, -5, 6));   // x1 = 3, x2 = 2
console.log(quadratic(1, 2, 1));    // x = -1
console.log(quadratic(1, 1, 1));    // ไม่มีคำตอบจริง
```

---

## Step 32: Assignment Operators

```javascript
// ตัวอย่างที่ 8: Assignment Operators ทั้งหมด
let num = 10;

num += 5;   // num = num + 5  -> 15
console.log(num); // 15

num -= 3;   // num = num - 3  -> 12
console.log(num); // 12

num *= 2;   // num = num * 2  -> 24
console.log(num); // 24

num /= 4;   // num = num / 4  -> 6
console.log(num); // 6

num %= 4;   // num = num % 4  -> 2
console.log(num); // 2

num **= 3;  // num = num ** 3 -> 8
console.log(num); // 8
```

```javascript
// ตัวอย่างที่ 9: Logical Assignment Operators (ES2021)
let a = null;
let b = 'existing';
let c = 0;

// Nullish assignment ??=
a ??= 'default';      // เพิ่มค่าถ้า a เป็น null หรือ undefined
console.log(a);        // 'default'

b ??= 'new value';    // b ไม่ใช่ null/undefined จึงไม่เปลี่ยน
console.log(b);        // 'existing'

// OR assignment ||=
c ||= 42;             // เพิ่มค่าถ้า c เป็น falsy
console.log(c);        // 42 (0 เป็น falsy)

let d = 5;
d ||= 100;            // 5 เป็น truthy จึงไม่เปลี่ยน
console.log(d);        // 5

// AND assignment &&=
let e = 10;
e &&= e * 2;          // คูณ 2 ถ้า e เป็น truthy
console.log(e);        // 20

let f = 0;
f &&= f * 2;          // 0 เป็น falsy จึงไม่เปลี่ยน
console.log(f);        // 0
```

```javascript
// ตัวอย่างที่ 10: Destructuring Assignment
// Object destructuring assignment
let firstName, lastName;
({ firstName, lastName } = { firstName: 'สมชาย', lastName: 'ใจดี' });
console.log(firstName, lastName);

// Array destructuring assignment
let x, y;
[x, y] = [10, 20];
console.log(x, y); // 10 20

// Swap
[x, y] = [y, x];
console.log(x, y); // 20 10

// ตัวอย่างที่ 11: Chained Assignment
let p, q, r;
p = q = r = 5;
console.log(p, q, r); // 5 5 5
```

---

## Step 33: Comparison Operators

```javascript
// ตัวอย่างที่ 12: Comparison Operators ทั้งหมด
console.log('=== Loose Equality (==) ===');
console.log(1 == 1);       // true
console.log(1 == '1');     // true  (type coercion!)
console.log(0 == false);   // true
console.log(0 == '');      // true
console.log(null == undefined); // true

console.log('\n=== Strict Equality (===) ===');
console.log(1 === 1);      // true
console.log(1 === '1');    // false (ต่างชนิด)
console.log(0 === false);  // false
console.log(null === undefined); // false

console.log('\n=== Inequality ===');
console.log(1 != 2);      // true
console.log(1 != '1');    // false (coercion: 1 == '1' เป็น true)
console.log(1 !== '1');   // true  (strict: ต่างชนิด)
```

```javascript
// ตัวอย่างที่ 13: Relational Operators
console.log(5 > 3);   // true
console.log(5 < 3);   // false
console.log(5 >= 5);  // true
console.log(5 <= 4);  // false

// เปรียบเทียบ string
console.log('apple' < 'banana');  // true (เปรียบเทียบ Unicode)
console.log('b' > 'a');           // true
console.log('10' > '9');          // false! ('1' < '9' ใน string)
console.log(10 > 9);              // true  (number เปรียบเทียบถูกต้อง)

// เปรียบเทียบ null และ undefined
console.log(null > 0);   // false
console.log(null == 0);  // false
console.log(null >= 0);  // true  (แปลก!)
console.log(null < 1);   // true
console.log(undefined > 0);  // false
console.log(undefined < 0);  // false
console.log(undefined == 0); // false

// ตัวอย่างที่ 14: Object comparison
const obj1 = { a: 1 };
const obj2 = { a: 1 };
const obj3 = obj1;

console.log(obj1 == obj2);  // false (คนละ reference)
console.log(obj1 === obj2); // false
console.log(obj1 === obj3); // true  (reference เดียวกัน)
```

---

## Step 34: Logical Operators

```javascript
// ตัวอย่างที่ 15: AND (&&) Operator
console.log(true && true);   // true
console.log(true && false);  // false
console.log(false && true);  // false
console.log(false && false); // false

// Short-circuit evaluation
console.log(false && 'hello'); // false (หยุดที่ false)
console.log(true && 'hello');  // 'hello' (return ค่าสุดท้าย)
console.log(0 && 'hello');     // 0      (0 เป็น falsy)
console.log(1 && 'hello');     // 'hello'

// use case: conditional execution
const user = { name: 'สมชาย' };
user && console.log('User exists:', user.name);

// ตัวอย่างที่ 16: OR (||) Operator
console.log(true || false);  // true
console.log(false || true);  // true
console.log(false || false); // false

// Short-circuit evaluation
console.log(false || 'default');  // 'default'
console.log(true || 'default');   // true (หยุดที่ truthy)
console.log('' || 'fallback');    // 'fallback'
console.log('value' || 'fallback'); // 'value'

// use case: default values (แบบเก่า)
function greet(name) {
    name = name || 'แขก';  // ถ้าไม่ส่ง name ใช้ 'แขก'
    console.log(`สวัสดี ${name}`);
}
greet('สมชาย'); // สวัสดี สมชาย
greet();        // สวัสดี แขก

// ปัญหา: ถ้า name เป็น '' หรือ 0 จะใช้ fallback!
greet('');      // สวัสดี แขก (อาจไม่ต้องการแบบนี้)
```

```javascript
// ตัวอย่างที่ 17: NOT (!) Operator
console.log(!true);   // false
console.log(!false);  // true
console.log(!0);      // true
console.log(!1);      // false
console.log(!'');     // true
console.log(!'hi');   // false
console.log(!null);   // true
console.log(!undefined); // true

// Double NOT - แปลงเป็น boolean
console.log(!!0);     // false
console.log(!!1);     // true
console.log(!!'');    // false
console.log(!!'hi');  // true

// ตัวอย่างที่ 18: Complex Logical Expressions
const age = 20;
const hasID = true;
const isMember = false;

// ต้องมีอายุ 18+ และมี ID หรือเป็น member
const canEnter = (age >= 18 && hasID) || isMember;
console.log(canEnter); // true

// Login validation
const username = 'admin';
const password = '12345';
const isValid = username !== '' && password.length >= 6;
console.log(isValid); // false (password สั้นเกิน)
```

```javascript
// ตัวอย่างที่ 19: Logical Operators กับ Non-Boolean Values
// && returns: ค่า falsy แรก หรือค่าสุดท้ายถ้าทุกอย่าง truthy
console.log(1 && 2 && 3);   // 3 (ทุกอย่าง truthy, return สุดท้าย)
console.log(1 && 0 && 3);   // 0 (0 เป็น falsy แรก)
console.log(0 && 1 && 2);   // 0

// || returns: ค่า truthy แรก หรือค่าสุดท้ายถ้าทุกอย่าง falsy
console.log(1 || 2 || 3);   // 1 (truthy แรก)
console.log(0 || '' || 3);  // 3
console.log(0 || '' || 0);  // 0 (ทุกอย่าง falsy, return สุดท้าย)

// Practical use
const config = {
    host: '',
    port: 0,
    name: 'mydb'
};

// || ไม่เหมาะ - 0 และ '' เป็น falsy
const host = config.host || 'localhost'; // 'localhost' (แม้ host เป็น '')
const port = config.port || 5432;       // 5432 (แม้ port เป็น 0!)

console.log(host, port);
```

---

## Step 35: Nullish Coalescing (??)

```javascript
// ตัวอย่างที่ 20: ?? Operator (ES2020)
// Return ค่าขวาเฉพาะเมื่อค่าซ้ายเป็น null หรือ undefined
console.log(null ?? 'default');      // 'default'
console.log(undefined ?? 'default'); // 'default'
console.log(0 ?? 'default');         // 0  (0 ไม่ใช่ null/undefined)
console.log('' ?? 'default');        // '' ('' ไม่ใช่ null/undefined)
console.log(false ?? 'default');     // false

// เปรียบเทียบกับ ||
const val1 = 0;
console.log(val1 || 'fallback');  // 'fallback' (0 เป็น falsy)
console.log(val1 ?? 'fallback');  // 0 (0 ไม่ใช่ null/undefined)

// ตัวอย่างที่ 21: Use cases ที่เหมาะสม
function createUser(options) {
    const {
        name,
        age = null,
        premium = false,
        theme = 'light',
        score = 0
    } = options;
    
    return {
        name: name ?? 'Anonymous',
        age: age ?? 'ไม่ระบุ',
        premium: premium ?? false,
        theme: theme ?? 'light',
        score: score ?? 0  // score 0 ต้องรักษาไว้!
    };
}

console.log(createUser({ name: 'สมชาย', score: 0 }));
// { name: 'สมชาย', age: 'ไม่ระบุ', premium: false, theme: 'light', score: 0 }
```

```javascript
// ตัวอย่างที่ 22: Nullish Assignment (??=)
let settings = {
    theme: null,
    lang: 'th',
    fontSize: 0
};

settings.theme ??= 'light';    // null -> 'light'
settings.lang ??= 'en';        // 'th' ไม่เปลี่ยน
settings.fontSize ??= 14;      // 0 ไม่เปลี่ยน! (0 ไม่ใช่ null)

console.log(settings);
// { theme: 'light', lang: 'th', fontSize: 0 }

// ตัวอย่างที่ 23: รวมกับ Optional Chaining
const users = {
    alice: { profile: { name: 'Alice', age: 25 } },
    bob: null
};

console.log(users.alice?.profile?.name ?? 'Unknown'); // Alice
console.log(users.bob?.profile?.name ?? 'Unknown');   // Unknown
console.log(users.charlie?.profile?.name ?? 'Unknown'); // Unknown
```

---

## Step 36: Optional Chaining (?.)

```javascript
// ตัวอย่างที่ 24: Optional Chaining พื้นฐาน
const user = {
    name: 'สมชาย',
    address: {
        city: 'กรุงเทพ'
    }
};

// แบบเก่า - verbose
const city1 = user && user.address && user.address.city;

// Optional Chaining - สั้นกว่า
const city2 = user?.address?.city;
const zip = user?.address?.zip;  // undefined (ไม่มี zip)

console.log(city2); // กรุงเทพ
console.log(zip);   // undefined (ไม่ error!)

// ตัวอย่างที่ 25: กับ method call
const obj = {
    greet() { return 'สวัสดี!'; }
};

console.log(obj.greet?.());    // 'สวัสดี!'
console.log(obj.goodbye?.());  // undefined (method ไม่มี)
// obj.goodbye(); // TypeError ถ้าไม่ใช้ ?.

// ตัวอย่างที่ 26: กับ array
const arr = [1, 2, 3];
const noArr = null;

console.log(arr?.[0]);    // 1
console.log(noArr?.[0]);  // undefined (ไม่ error)
```

```javascript
// ตัวอย่างที่ 27: Real-world use cases
function processApiResponse(response) {
    // response อาจมีโครงสร้างที่ไม่แน่นอน
    const userId = response?.data?.user?.id;
    const email = response?.data?.user?.contact?.email ?? 'N/A';
    const firstTag = response?.data?.tags?.[0]?.name ?? 'No tags';
    
    return { userId, email, firstTag };
}

// ทดสอบกับข้อมูลต่างๆ
console.log(processApiResponse({
    data: {
        user: {
            id: 123,
            contact: { email: 'user@example.com' }
        },
        tags: [{ name: 'javascript' }, { name: 'web' }]
    }
}));
// { userId: 123, email: 'user@example.com', firstTag: 'javascript' }

console.log(processApiResponse(null));
// { userId: undefined, email: 'N/A', firstTag: 'No tags' }

console.log(processApiResponse({ data: null }));
// { userId: undefined, email: 'N/A', firstTag: 'No tags' }
```

---

## Step 37: Ternary Operator

```javascript
// ตัวอย่างที่ 28: Ternary Operator พื้นฐาน
// condition ? valueIfTrue : valueIfFalse
const age = 20;
const status = age >= 18 ? 'ผู้ใหญ่' : 'เยาวชน';
console.log(status); // ผู้ใหญ่

// เทียบกับ if/else
let status2;
if (age >= 18) {
    status2 = 'ผู้ใหญ่';
} else {
    status2 = 'เยาวชน';
}

// ตัวอย่างที่ 29: Ternary ใน template literal
const score = 85;
console.log(`คุณ${score >= 60 ? 'ผ่าน' : 'ไม่ผ่าน'}การสอบ`);
console.log(`เกรด: ${score >= 90 ? 'A' : score >= 80 ? 'B' : score >= 70 ? 'C' : 'D'}`);
```

```javascript
// ตัวอย่างที่ 30: Nested Ternary (ไม่แนะนำถ้ายาวเกิน)
function getGrade(score) {
    return score >= 90 ? 'A' :
           score >= 80 ? 'B' :
           score >= 70 ? 'C' :
           score >= 60 ? 'D' : 'F';
}
console.log(getGrade(95));  // A
console.log(getGrade(72));  // C
console.log(getGrade(50));  // F

// ตัวอย่างที่ 31: Ternary ใน JSX (React pattern)
const isLoggedIn = true;
const greeting = isLoggedIn ? 'ยินดีต้อนรับกลับ!' : 'กรุณาเข้าสู่ระบบ';
console.log(greeting);

// ตัวอย่างที่ 32: Conditional rendering
const items = ['a', 'b', 'c'];
const list = items.length > 0 ? `มี ${items.length} รายการ` : 'ไม่มีรายการ';
console.log(list);

// ตัวอย่างที่ 33: ternary กับ function call
function showMessage(isError) {
    (isError ? console.error : console.log)('ข้อความนี้');
}
showMessage(false); // ใช้ console.log
showMessage(true);  // ใช้ console.error
```

---

## Step 38: Bitwise Operators

```javascript
// ตัวอย่างที่ 34: Bitwise operators
// ทำงานกับ bits (binary representation)
const x = 5;   // binary: 0101
const y = 3;   // binary: 0011

console.log(x & y);   // 1   (AND:  0101 & 0011 = 0001)
console.log(x | y);   // 7   (OR:   0101 | 0011 = 0111)
console.log(x ^ y);   // 6   (XOR:  0101 ^ 0011 = 0110)
console.log(~x);       // -6  (NOT:  ~0101 = -(5+1))
console.log(x << 1);  // 10  (Left shift:  0101 << 1 = 1010)
console.log(x >> 1);  // 2   (Right shift: 0101 >> 1 = 0010)
console.log(x >>> 1); // 2   (Unsigned right shift)

// ตัวอย่างที่ 35: Practical uses ของ bitwise
// ตรวจสอบเลขคู่อย่างรวดเร็ว
function isEvenFast(n) {
    return (n & 1) === 0;
}
console.log(isEvenFast(4));  // true
console.log(isEvenFast(7));  // false

// ทำให้เป็น integer (เร็วกว่า Math.floor สำหรับบวก)
console.log(4.7 | 0);    // 4
console.log(~~4.7);      // 4 (double bitwise NOT)
```

```javascript
// ตัวอย่างที่ 36: Permission System ด้วย Bitwise
const PERMISSIONS = {
    READ:    0b0001,  // 1
    WRITE:   0b0010,  // 2
    DELETE:  0b0100,  // 4
    ADMIN:   0b1000   // 8
};

function hasPermission(userPerms, permission) {
    return (userPerms & permission) !== 0;
}

function grantPermission(userPerms, permission) {
    return userPerms | permission;
}

function revokePermission(userPerms, permission) {
    return userPerms & ~permission;
}

let userPerms = PERMISSIONS.READ | PERMISSIONS.WRITE; // 3 (011)
console.log('READ:', hasPermission(userPerms, PERMISSIONS.READ));   // true
console.log('WRITE:', hasPermission(userPerms, PERMISSIONS.WRITE)); // true
console.log('DELETE:', hasPermission(userPerms, PERMISSIONS.DELETE)); // false

// เพิ่ม DELETE
userPerms = grantPermission(userPerms, PERMISSIONS.DELETE);
console.log('\nหลังเพิ่ม DELETE:');
console.log('DELETE:', hasPermission(userPerms, PERMISSIONS.DELETE)); // true

// ลบ WRITE
userPerms = revokePermission(userPerms, PERMISSIONS.WRITE);
console.log('\nหลังลบ WRITE:');
console.log('WRITE:', hasPermission(userPerms, PERMISSIONS.WRITE)); // false
```

---

## Step 39: Comma Operator

```javascript
// ตัวอย่างที่ 37: Comma Operator
// ประเมินนิพจน์ทั้งหมดและคืนค่าสุดท้าย
let result = (1 + 2, 3 + 4, 5 + 6);
console.log(result); // 11 (ค่าสุดท้าย)

// ใช้ใน for loop หลายตัวแปร
for (let i = 0, j = 10; i < 5; i++, j--) {
    process.stdout.write(`(${i},${j}) `);
}
// (0,10) (1,9) (2,8) (3,7) (4,6)

// ตัวอย่างที่ 38: void Operator
console.log(void 0);         // undefined
console.log(void 'anything'); // undefined
// ใช้บน href ใน HTML:
// <a href="javascript:void(0)">คลิกที่นี่</a>

// ตัวอย่างที่ 39: typeof Operator กับ undeclared variable
// ไม่ throw error แม้ตัวแปรไม่ได้ประกาศ
console.log(typeof undeclaredVar); // "undefined" (ไม่ error)
// console.log(undeclaredVar);     // ReferenceError!
```

---

## Step 40: in และ instanceof Operators

```javascript
// ตัวอย่างที่ 40: in Operator
const person = { name: 'สมชาย', age: 25, city: 'กรุงเทพ' };

console.log('name' in person);  // true
console.log('age' in person);   // true
console.log('email' in person); // false

// กับ array (ตรวจ index)
const fruits = ['แอปเปิ้ล', 'กล้วย', 'ส้ม'];
console.log(0 in fruits);       // true
console.log(2 in fruits);       // true
console.log(3 in fruits);       // false

// Prototype properties ก็ true
console.log('toString' in person); // true (inherited)
console.log(person.hasOwnProperty('toString')); // false (inherited)
console.log(person.hasOwnProperty('name'));     // true (own)

// ตัวอย่างที่ 41: instanceof Operator
class Animal {
    constructor(name) { this.name = name; }
}

class Dog extends Animal {
    bark() { return 'โฮ่ง!'; }
}

const rex = new Dog('Rex');

console.log(rex instanceof Dog);     // true
console.log(rex instanceof Animal);  // true (inheritance)
console.log(rex instanceof Object);  // true (ทุกอย่างเป็น Object)
console.log(rex instanceof Array);   // false

// ตัวอย่างที่ 42: ใช้ instanceof สำหรับ type checking
function processInput(input) {
    if (input instanceof Array) {
        return `Array ที่มี ${input.length} elements`;
    } else if (input instanceof Date) {
        return `Date: ${input.toLocaleDateString('th-TH')}`;
    } else if (input instanceof RegExp) {
        return `RegExp: ${input.source}`;
    } else {
        return `Other: ${typeof input}`;
    }
}

console.log(processInput([1, 2, 3]));          // Array ที่มี 3 elements
console.log(processInput(new Date()));          // Date: วันปัจจุบัน
console.log(processInput(/hello/));             // RegExp: hello
console.log(processInput('text'));              // Other: string
```

---

## Step 41: Operator Precedence (ลำดับความสำคัญ)

```javascript
// ตัวอย่างที่ 43: Operator Precedence
// ลำดับจากสูงไปต่ำ (บางส่วน):
// 21: ()  - Grouping
// 20: .  ?. []  - Member access
// 19: new  - Constructor
// 18: ()  - Function call
// 17: ++  -- (postfix)
// 16: !  ~  +  -  typeof  void  delete  ++  -- (prefix)
// 15: **  - Exponentiation
// 14: *  /  %
// 13: +  -
// 12: <<  >>  >>>
// 11: <  <=  >  >=  in  instanceof
// 10: ==  !=  ===  !==
//  9: &
//  8: ^
//  7: |
//  6: &&
//  5: ??
//  4: ||
//  3: ?:  - Ternary
//  2: =  +=  -=  etc. (Assignment)
//  1: ,  - Comma

// ตัวอย่างที่ 44: เหตุผลที่ต้องรู้ precedence
console.log(2 + 3 * 4);    // 14 (ไม่ใช่ 20!)
console.log((2 + 3) * 4);  // 20

console.log(true || false && false); // true (&&  ก่อน ||)
console.log((true || false) && false); // false

console.log(1 + 2 === 3);    // true (1+2 ก่อน แล้วค่อย ===)
console.log(typeof 1 + 2);   // "number2" (typeof ก่อน แล้ว +"2")
console.log(typeof (1 + 2)); // "number"
```

```javascript
// ตัวอย่างที่ 45: Association (ซ้าย vs ขวา)
// Left-to-right (ส่วนใหญ่)
console.log(1 - 2 - 3);    // -4 (ซ้ายไปขวา: (1-2)-3)

// Right-to-left (Assignment, **)
let a = b = c = 5; // c=5 แล้ว b=5 แล้ว a=5 (ขวาไปซ้าย)

console.log(2 ** 3 ** 2);  // 512 (ขวาไปซ้าย: 2**(3**2) = 2**9 = 512)
console.log((2 ** 3) ** 2); // 64

// ตัวอย่างที่ 46: Grouping ด้วย ()
const total = (price * quantity) - (discount * price * quantity);
const condition = (a > 0) && (b > 0) || (c > 0);
const value = (flag1 || flag2) && (flag3 || flag4);
```

---

## Step 42: Short-Circuit Evaluation (การประเมินแบบลัดวงจร)

```javascript
// ตัวอย่างที่ 47: Short-circuit ใน && 
let count = 0;
function increment() {
    count++;
    return true;
}

false && increment(); // increment ไม่ถูกเรียก!
console.log(count);   // 0

true && increment();  // increment ถูกเรียก
console.log(count);   // 1

// ตัวอย่างที่ 48: Short-circuit ใน ||
let log = [];

function getValue1() {
    log.push('getValue1');
    return false;
}

function getValue2() {
    log.push('getValue2');
    return true;
}

function getValue3() {
    log.push('getValue3');
    return true;
}

const result = getValue1() || getValue2() || getValue3();
console.log(log);    // ['getValue1', 'getValue2'] (getValue3 ไม่ถูกเรียก)
console.log(result); // true
```

```javascript
// ตัวอย่างที่ 49: Practical Short-circuit patterns
// Pattern 1: Guard clause
function processUser(user) {
    user && user.active && sendEmail(user.email);
}

function sendEmail(email) {
    console.log(`ส่งอีเมลไปที่ ${email}`);
}

// Pattern 2: Default value ด้วย ||
function getConfig(userConfig) {
    return {
        theme: userConfig.theme || 'light',
        lang: userConfig.lang || 'th',
        font: userConfig.font || 'Sarabun'
    };
}

// Pattern 3: Conditional function call
const debug = true;
debug && console.log('Debug mode on');

// Pattern 4: ใช้กับ React (render conditionally)
const showModal = true;
// JSX: {showModal && <Modal />}
```

---

## Step 43: Spread และ Rest Operators

```javascript
// ตัวอย่างที่ 50: Spread Operator (...)
// 1. รวม arrays
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];
const merged = [...arr1, ...arr2];
console.log(merged); // [1, 2, 3, 4, 5, 6]

// 2. Copy array
const original = [1, 2, 3];
const copy = [...original];
copy.push(4);
console.log(original); // [1, 2, 3] (ไม่เปลี่ยน)
console.log(copy);     // [1, 2, 3, 4]

// 3. String เป็น array
console.log([...'hello']); // ['h', 'e', 'l', 'l', 'o']

// 4. Set เป็น array (unique values)
const unique = [...new Set([1, 2, 2, 3, 3, 4])];
console.log(unique); // [1, 2, 3, 4]

// ตัวอย่างที่ 51: Spread กับ Object
const defaults = { color: 'red', size: 'medium', weight: 100 };
const custom = { color: 'blue', weight: 150 };
const merged2 = { ...defaults, ...custom };
console.log(merged2); // { color: 'blue', size: 'medium', weight: 150 }

// 5. Function arguments
function sum(a, b, c) { return a + b + c; }
const nums = [1, 2, 3];
console.log(sum(...nums)); // 6
```

```javascript
// ตัวอย่างที่ 52: Rest Parameters
function sumAll(...numbers) {
    return numbers.reduce((total, n) => total + n, 0);
}

console.log(sumAll(1, 2, 3));       // 6
console.log(sumAll(1, 2, 3, 4, 5)); // 15

// Rest กับ parameters อื่น
function printUser(name, age, ...hobbies) {
    console.log(`ชื่อ: ${name}, อายุ: ${age}`);
    console.log('งานอดิเรก:', hobbies.join(', '));
}

printUser('สมชาย', 25, 'อ่านหนังสือ', 'เล่นกีฬา', 'ถ่ายรูป');

// ตัวอย่างที่ 53: Rest ใน destructuring
const [head, ...tail] = [1, 2, 3, 4, 5];
console.log(head); // 1
console.log(tail); // [2, 3, 4, 5]

const { name, ...rest } = { name: 'สมชาย', age: 25, city: 'กรุงเทพ' };
console.log(name); // สมชาย
console.log(rest); // { age: 25, city: 'กรุงเทพ' }
```

---

## Step 44: Expression vs Statement

```javascript
// ตัวอย่างที่ 54: Expression vs Statement
// Expression: มีค่า สามารถใช้ใน expression อื่นได้
let x = 5;           // 5 เป็น expression
x + 3;               // 8 เป็น expression
x > 3 ? 'big' : 'small'; // ternary expression
x++; // expression (มีค่าเป็น 5)

// Statement: คำสั่ง ไม่มีค่า
if (x > 0) {}       // if statement
for (let i; i<10; i++) {} // for statement

// Expression Statement: expression ที่ใช้เป็น statement
console.log('hello'); // expression ที่ใช้เป็น statement

// ตัวอย่างที่ 55: Comma Expression
let a = (1, 2, 3);  // a = 3 (ค่าสุดท้าย)
console.log(a);      // 3

// for loop ที่ซับซ้อน
for (let i = 0, j = 10; i < 5; i++, j -= 2) {
    console.log(i, j);
}
```

---

## Step 45: โปรแกรมตัวอย่างรวม Operators

```javascript
// ตัวอย่างที่ 56: Shopping Cart System
'use strict';

const TAX_RATE = 0.07; // 7% VAT
const SHIPPING_THRESHOLD = 500; // ส่งฟรีเมื่อซื้อครบ 500

function calculateCart(items, couponCode) {
    // คำนวณยอดรวมก่อนส่วนลด
    const subtotal = items.reduce((sum, item) => {
        return sum + (item.price * item.quantity);
    }, 0);
    
    // ส่วนลด coupon
    const discounts = {
        'SAVE10': 0.10,
        'SAVE20': 0.20,
        'HALFOFF': 0.50
    };
    const discountRate = discounts[couponCode] ?? 0;
    const discountAmount = subtotal * discountRate;
    
    // ราคาหลังส่วนลด
    const afterDiscount = subtotal - discountAmount;
    
    // ค่าขนส่ง
    const shipping = afterDiscount >= SHIPPING_THRESHOLD ? 0 : 50;
    
    // ภาษี
    const tax = afterDiscount * TAX_RATE;
    
    // ยอดสุทธิ
    const total = afterDiscount + shipping + tax;
    
    return {
        subtotal,
        discountRate: discountRate * 100,
        discountAmount,
        afterDiscount,
        shipping,
        tax,
        total
    };
}

const cartItems = [
    { name: 'เสื้อยืด', price: 299, quantity: 2 },
    { name: 'กางเกง', price: 499, quantity: 1 },
    { name: 'รองเท้า', price: 899, quantity: 1 }
];

const receipt = calculateCart(cartItems, 'SAVE10');

console.log('=== ใบเสร็จรับเงิน ===');
cartItems.forEach(item => {
    const itemTotal = item.price * item.quantity;
    console.log(`${item.name.padEnd(15)} x${item.quantity} = ${itemTotal.toLocaleString()} บาท`);
});
console.log('-'.repeat(35));
console.log(`ราคารวม:      ${receipt.subtotal.toLocaleString()} บาท`);
console.log(`ส่วนลด ${receipt.discountRate}%:   -${receipt.discountAmount.toLocaleString()} บาท`);
console.log(`หลังส่วนลด:   ${receipt.afterDiscount.toLocaleString()} บาท`);
console.log(`ค่าขนส่ง:     ${receipt.shipping === 0 ? 'ฟรี!' : receipt.shipping + ' บาท'}`);
console.log(`ภาษี 7%:      ${receipt.tax.toFixed(2)} บาท`);
console.log('='.repeat(35));
console.log(`ยอดสุทธิ:     ${receipt.total.toFixed(2)} บาท`);
```

```javascript
// ตัวอย่างที่ 57: Form Validation System
'use strict';

const validators = {
    required: (value) => 
        value != null && String(value).trim() !== '' 
            ? null 
            : 'จำเป็นต้องกรอกข้อมูล',
    
    minLength: (min) => (value) => 
        String(value).length >= min 
            ? null 
            : `ต้องมีอย่างน้อย ${min} ตัวอักษร`,
    
    maxLength: (max) => (value) => 
        String(value).length <= max 
            ? null 
            : `ต้องมีไม่เกิน ${max} ตัวอักษร`,
    
    email: (value) => 
        /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value) 
            ? null 
            : 'รูปแบบอีเมลไม่ถูกต้อง',
    
    phone: (value) => 
        /^(06|08|09)\d{8}$/.test(value) 
            ? null 
            : 'รูปแบบเบอร์โทรไม่ถูกต้อง',
    
    min: (minimum) => (value) => 
        Number(value) >= minimum 
            ? null 
            : `ต้องมีค่าอย่างน้อย ${minimum}`,
    
    max: (maximum) => (value) => 
        Number(value) <= maximum 
            ? null 
            : `ต้องมีค่าไม่เกิน ${maximum}`
};

function validate(data, rules) {
    const errors = {};
    
    for (const [field, fieldRules] of Object.entries(rules)) {
        const value = data[field];
        const fieldErrors = [];
        
        for (const rule of fieldRules) {
            const error = rule(value);
            if (error) fieldErrors.push(error);
        }
        
        if (fieldErrors.length > 0) {
            errors[field] = fieldErrors;
        }
    }
    
    return {
        isValid: Object.keys(errors).length === 0,
        errors
    };
}

// ทดสอบ
const formData = {
    name: 'ส',
    email: 'invalid-email',
    phone: '1234567890',
    age: 15,
    password: '123'
};

const rules = {
    name: [validators.required, validators.minLength(2), validators.maxLength(50)],
    email: [validators.required, validators.email],
    phone: [validators.required, validators.phone],
    age: [validators.required, validators.min(18), validators.max(100)],
    password: [validators.required, validators.minLength(8)]
};

const result = validate(formData, rules);
console.log('ผ่านการตรวจสอบ:', result.isValid);
console.log('ข้อผิดพลาด:', result.errors);
```

```javascript
// ตัวอย่างที่ 58: Expression evaluator
'use strict';

function safeEval(expression, variables = {}) {
    // แทนที่ตัวแปรด้วยค่า
    let expr = expression;
    for (const [key, value] of Object.entries(variables)) {
        expr = expr.replace(new RegExp(`\\b${key}\\b`, 'g'), value);
    }
    
    // คำนวณเฉพาะนิพจน์ที่ปลอดภัย
    if (!/^[\d\s\+\-\*\/\.\(\)%\^]+$/.test(expr)) {
        throw new Error('Invalid expression');
    }
    
    try {
        return Function(`'use strict'; return (${expr})`)();
    } catch {
        throw new Error('Evaluation failed');
    }
}

// ตัวอย่างการใช้
const vars = { x: 10, y: 5, z: 2 };
console.log(safeEval('x + y * z', vars));   // 10 + 5*2 = 20
console.log(safeEval('(x + y) * z', vars)); // (10+5)*2 = 30
console.log(safeEval('x / z + y', vars));   // 10/2 + 5 = 10
```

```javascript
// ตัวอย่างที่ 59: Operator Chaining Pattern
'use strict';

class NumberBuilder {
    #value;
    
    constructor(value = 0) {
        this.#value = value;
    }
    
    add(n) { this.#value += n; return this; }
    subtract(n) { this.#value -= n; return this; }
    multiply(n) { this.#value *= n; return this; }
    divide(n) { this.#value /= n; return this; }
    abs() { this.#value = Math.abs(this.#value); return this; }
    round(decimals = 0) { 
        this.#value = parseFloat(this.#value.toFixed(decimals)); 
        return this; 
    }
    
    get value() { return this.#value; }
    
    toString() { return String(this.#value); }
}

const result = new NumberBuilder(100)
    .add(50)
    .multiply(2)
    .subtract(100)
    .divide(3)
    .round(2)
    .value;

console.log(result); // (100+50)*2 - 100 = 200, 200/3 = 66.67
```

```javascript
// ตัวอย่างที่ 60: Operator ใน Real-world use case
'use strict';

// ระบบคะแนน เกมส์
class GameScore {
    #playerName;
    #score = 0;
    #level = 1;
    #lives = 3;
    #multiplier = 1;
    
    constructor(playerName) {
        this.#playerName = playerName;
    }
    
    addPoints(points) {
        this.#score += points * this.#multiplier;
        
        // Level up ทุก 1000 คะแนน
        const newLevel = Math.floor(this.#score / 1000) + 1;
        if (newLevel > this.#level) {
            this.#level = newLevel;
            this.#multiplier = 1 + (this.#level - 1) * 0.5;
            console.log(`Level up! Level ${this.#level}, Multiplier: ${this.#multiplier}x`);
        }
        
        return this;
    }
    
    loseLife() {
        this.#lives = Math.max(0, --this.#lives);
        return this;
    }
    
    get isAlive() { return this.#lives > 0; }
    
    get status() {
        return {
            player: this.#playerName,
            score: this.#score,
            level: this.#level,
            lives: this.#lives,
            multiplier: `${this.#multiplier}x`,
            isAlive: this.isAlive
        };
    }
}

const player = new GameScore('สมชาย');
player.addPoints(500).addPoints(300).addPoints(400);
player.loseLife();
console.log(player.status);
```

---

## Step 46-50: โปรแกรมขั้นสูง

```javascript
// ตัวอย่างที่ 61: Pipeline Pattern กับ Operators
'use strict';

// Pipeline ด้วย reduce
const pipe = (...fns) => x => fns.reduce((v, f) => f(v), x);

const processText = pipe(
    text => text.trim(),
    text => text.toLowerCase(),
    text => text.replace(/\s+/g, ' '),
    text => text.split(' ').map(w => w[0].toUpperCase() + w.slice(1)).join(' ')
);

console.log(processText('  hello   WORLD  JavaScript  '));
// Hello World Javascript
```

```javascript
// ตัวอย่างที่ 62: Operator Overloading Simulation
'use strict';

class Vector {
    constructor(x, y) {
        this.x = x;
        this.y = y;
    }
    
    add(other) { return new Vector(this.x + other.x, this.y + other.y); }
    subtract(other) { return new Vector(this.x - other.x, this.y - other.y); }
    scale(factor) { return new Vector(this.x * factor, this.y * factor); }
    dot(other) { return this.x * other.x + this.y * other.y; }
    
    get magnitude() { return Math.sqrt(this.x ** 2 + this.y ** 2); }
    get normalized() {
        const mag = this.magnitude;
        return mag === 0 ? new Vector(0, 0) : new Vector(this.x / mag, this.y / mag);
    }
    
    toString() { return `Vector(${this.x}, ${this.y})`; }
}

const v1 = new Vector(3, 4);
const v2 = new Vector(1, 2);

console.log(v1.add(v2).toString());         // Vector(4, 6)
console.log(v1.subtract(v2).toString());    // Vector(2, 2)
console.log(v1.scale(2).toString());        // Vector(6, 8)
console.log(v1.dot(v2));                    // 11
console.log(v1.magnitude);                  // 5
console.log(v1.normalized.toString());      // Vector(0.6, 0.8)
```

```javascript
// ตัวอย่างที่ 63: Comparison Utilities
'use strict';

const compare = {
    // Deep equality
    deepEqual(a, b) {
        if (a === b) return true;
        if (typeof a !== typeof b) return false;
        if (a === null || b === null) return false;
        
        if (Array.isArray(a) && Array.isArray(b)) {
            if (a.length !== b.length) return false;
            return a.every((item, i) => this.deepEqual(item, b[i]));
        }
        
        if (typeof a === 'object') {
            const keysA = Object.keys(a);
            const keysB = Object.keys(b);
            if (keysA.length !== keysB.length) return false;
            return keysA.every(key => this.deepEqual(a[key], b[key]));
        }
        
        return false;
    },
    
    // Sort comparator
    byField: (field) => (a, b) => {
        if (a[field] < b[field]) return -1;
        if (a[field] > b[field]) return 1;
        return 0;
    },
    
    // Multi-sort
    multiSort: (...comparators) => (a, b) => {
        for (const cmp of comparators) {
            const result = cmp(a, b);
            if (result !== 0) return result;
        }
        return 0;
    }
};

const obj1 = { a: 1, b: { c: [1, 2, 3] } };
const obj2 = { a: 1, b: { c: [1, 2, 3] } };
const obj3 = { a: 1, b: { c: [1, 2, 4] } };

console.log(compare.deepEqual(obj1, obj2)); // true
console.log(compare.deepEqual(obj1, obj3)); // false

const people = [
    { name: 'สมชาย', age: 25 },
    { name: 'สมหญิง', age: 22 },
    { name: 'สมศักดิ์', age: 25 },
    { name: 'สมปอง', age: 30 }
];

const sorted = [...people].sort(compare.multiSort(
    compare.byField('age'),
    compare.byField('name')
));
console.log(sorted.map(p => `${p.name}(${p.age})`).join(', '));
// สมหญิง(22), สมชาย(25), สมศักดิ์(25), สมปอง(30)
```

```javascript
// ตัวอย่างที่ 64: Logical Operators ใน Configuration
'use strict';

class AppConfig {
    #config;
    
    constructor(userConfig = {}) {
        const defaults = {
            theme: 'light',
            lang: 'th',
            fontSize: 14,
            autoSave: true,
            notifications: true,
            maxItems: 50
        };
        
        // Merge configs - user config overrides defaults
        this.#config = { ...defaults, ...userConfig };
    }
    
    get(key) {
        // Optional chaining + nullish coalescing
        return this.#config?.[key] ?? null;
    }
    
    set(key, value) {
        // Logical AND to ensure key exists first
        key in this.#config && (this.#config[key] = value);
        return this;
    }
    
    setIfAbsent(key, value) {
        this.#config[key] ??= value;
        return this;
    }
    
    isDarkMode() {
        return this.#config.theme === 'dark';
    }
    
    toString() {
        return JSON.stringify(this.#config, null, 2);
    }
}

const config = new AppConfig({ theme: 'dark', fontSize: 16 });
config.set('lang', 'en').setIfAbsent('fontSize', 12);

console.log('Theme:', config.get('theme'));    // dark
console.log('Lang:', config.get('lang'));      // en
console.log('Font:', config.get('fontSize')); // 16 (ไม่เปลี่ยนเพราะมีอยู่แล้ว)
console.log('Dark mode:', config.isDarkMode()); // true
```

```javascript
// ตัวอย่างที่ 65: Arithmetic ใน Animation (concept)
'use strict';

// Easing functions สำหรับ animation
const easing = {
    linear: t => t,
    
    easeIn: t => t * t,
    easeOut: t => 1 - (1 - t) ** 2,
    easeInOut: t => t < 0.5 ? 2 * t * t : 1 - (-2 * t + 2) ** 2 / 2,
    
    bounce: t => {
        if (t < 1 / 2.75) return 7.5625 * t * t;
        if (t < 2 / 2.75) return 7.5625 * (t -= 1.5 / 2.75) * t + 0.75;
        if (t < 2.5 / 2.75) return 7.5625 * (t -= 2.25 / 2.75) * t + 0.9375;
        return 7.5625 * (t -= 2.625 / 2.75) * t + 0.984375;
    }
};

function interpolate(from, to, t, easingFn = easing.linear) {
    const easedT = easingFn(t);
    return from + (to - from) * easedT;
}

// แสดง progress animation
console.log('=== Animation Progress ===');
for (let t = 0; t <= 1; t += 0.1) {
    const pos = interpolate(0, 100, t, easing.easeInOut);
    const bar = '█'.repeat(Math.round(pos / 5));
    console.log(`${(t * 100).toFixed(0).padStart(3)}%: ${bar.padEnd(20)} ${pos.toFixed(1)}`);
}
```

```javascript
// ตัวอย่างที่ 66: Complex Validation with Operators
'use strict';

function validatePassword(password) {
    const checks = {
        length: password.length >= 8,
        hasUppercase: /[A-Z]/.test(password),
        hasLowercase: /[a-z]/.test(password),
        hasNumber: /\d/.test(password),
        hasSpecial: /[!@#$%^&*(),.?":{}|<>]/.test(password)
    };
    
    const passed = Object.values(checks).filter(Boolean).length;
    const strength = passed <= 2 ? 'อ่อน' : passed <= 4 ? 'ปานกลาง' : 'แข็งแรง';
    
    return {
        ...checks,
        passed,
        strength,
        isValid: checks.length && (passed >= 4)
    };
}

const passwords = ['abc', 'password123', 'Password123!', 'P@ssw0rd!!'];
passwords.forEach(pw => {
    const result = validatePassword(pw);
    console.log(`\n"${pw}":`);
    console.log(`  ความยาว >= 8: ${result.length ? '✓' : '✗'}`);
    console.log(`  ตัวพิมพ์ใหญ่: ${result.hasUppercase ? '✓' : '✗'}`);
    console.log(`  ตัวพิมพ์เล็ก: ${result.hasLowercase ? '✓' : '✗'}`);
    console.log(`  ตัวเลข: ${result.hasNumber ? '✓' : '✗'}`);
    console.log(`  อักขระพิเศษ: ${result.hasSpecial ? '✓' : '✗'}`);
    console.log(`  ความแข็งแรง: ${result.strength}`);
});
```

```javascript
// ตัวอย่างที่ 67: Bitwise สำหรับ Color Manipulation
'use strict';

function hexToRgb(hex) {
    const n = parseInt(hex.slice(1), 16);
    return {
        r: (n >> 16) & 0xFF,
        g: (n >> 8) & 0xFF,
        b: n & 0xFF
    };
}

function rgbToHex(r, g, b) {
    return '#' + [r, g, b].map(c => c.toString(16).padStart(2, '0')).join('');
}

function lighten(hex, amount) {
    const { r, g, b } = hexToRgb(hex);
    const factor = 1 + amount / 100;
    return rgbToHex(
        Math.min(255, Math.round(r * factor)),
        Math.min(255, Math.round(g * factor)),
        Math.min(255, Math.round(b * factor))
    );
}

function darken(hex, amount) {
    const { r, g, b } = hexToRgb(hex);
    const factor = 1 - amount / 100;
    return rgbToHex(
        Math.max(0, Math.round(r * factor)),
        Math.max(0, Math.round(g * factor)),
        Math.max(0, Math.round(b * factor))
    );
}

const baseColor = '#3498DB';
console.log('Base:', baseColor);
console.log('RGB:', hexToRgb(baseColor));
console.log('Lighten 20%:', lighten(baseColor, 20));
console.log('Darken 20%:', darken(baseColor, 20));
```

---

## สรุป Step 31-50

ในส่วนนี้คุณได้เรียนรู้:
- **Arithmetic**: +, -, *, /, %, ** และ increment/decrement
- **Assignment**: =, +=, -=, *=, /=, %= พร้อม ??=, ||=, &&=
- **Comparison**: ==, ===, !=, !==, >, <, >=, <=
- **Logical**: &&, ||, ! และ short-circuit evaluation
- **Nullish Coalescing**: ?? และ ??=
- **Optional Chaining**: ?.
- **Ternary**: condition ? true : false
- **Bitwise**: &, |, ^, ~, <<, >>, >>>
- **Spread/Rest**: ...
- **in, instanceof**: type checking
- **Operator Precedence**: ลำดับความสำคัญ

---

## แบบฝึกหัด

### ระดับพื้นฐาน

**แบบฝึกหัดที่ 1**: Arithmetic Operations
```javascript
// คำนวณ:
// 1. พื้นที่วงกลม รัศมี = 7
// 2. ปริมาตรทรงกลม รัศมี = 5
// 3. ดอกเบี้ยทบต้น: เงินต้น 10,000, ดอกเบี้ย 5%/ปี, 3 ปี
// TODO: เขียนโค้ดของคุณที่นี่
```

**แบบฝึกหัดที่ 2**: Logical Operators
```javascript
const user = { role: 'admin', active: true, premium: false };
// ตรวจสอบ:
// - สามารถลบข้อมูลได้หรือไม่ (role === 'admin' && active)
// - แสดงโฆษณาหรือไม่ (!premium && active)
// - ใช้ feature premium ได้ไหม (premium || role === 'admin')
```

### ระดับกลาง

**แบบฝึกหัดที่ 3**: Safe Object Access
```javascript
const response = {
    status: 200,
    data: {
        users: [
            { id: 1, name: 'สมชาย', email: 'somchai@example.com' }
        ]
    }
};

// ดึงข้อมูล email ของ user แรก
// ถ้าไม่มีข้อมูลให้ return 'N/A'
// ใช้ optional chaining และ nullish coalescing
```

**แบบฝึกหัดที่ 4**: Number Processing
```javascript
'use strict';
// สร้างฟังก์ชัน formatNumber(n) ที่:
// - ถ้า n >= 1,000,000 -> "1.0M"
// - ถ้า n >= 1,000 -> "1.0K"  
// - ไม่งั้น -> n ตามปกติ
// ตัวอย่าง: 1500000 -> "1.5M", 2500 -> "2.5K", 999 -> "999"
```

### ระดับสูง

**แบบฝึกหัดที่ 5**: Build a Validator
สร้าง validation library ที่ใช้ operator ต่างๆ เพื่อตรวจสอบ:
- เบอร์บัตรประจำตัวไทย (13 หลัก)
- รหัสไปรษณีย์ไทย (5 หลัก)
- วันเกิดในรูปแบบ DD/MM/YYYY

---

## เฉลยแบบฝึกหัด

**เฉลยที่ 4**: formatNumber
```javascript
'use strict';
function formatNumber(n) {
    if (n >= 1_000_000) return `${(n / 1_000_000).toFixed(1)}M`;
    if (n >= 1_000) return `${(n / 1_000).toFixed(1)}K`;
    return String(n);
}

console.log(formatNumber(1500000));  // 1.5M
console.log(formatNumber(2500));     // 2.5K
console.log(formatNumber(999));      // 999
console.log(formatNumber(1000000));  // 1.0M
```

---

*ส่วนถัดไป: Part 4 - Control Flow (การควบคุมการทำงาน)*
