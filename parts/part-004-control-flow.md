# ส่วนที่ 4: การควบคุมการทำงาน (Control Flow)

## คำอธิบาย
ในส่วนนี้คุณจะได้เรียนรู้การควบคุมการทำงานของโปรแกรมใน JavaScript ทั้งหมด ตั้งแต่ if/else, switch, while loops ไปจนถึง labeled statements และ nested loops เนื้อหาครอบคลุม Step 51-70

---

## Step 51: if / else if / else

```javascript
// ตัวอย่างที่ 1: if พื้นฐาน
const temperature = 35;

if (temperature > 30) {
    console.log('อากาศร้อนมาก!');
}

// ตัวอย่างที่ 2: if/else
const age = 17;

if (age >= 18) {
    console.log('คุณเป็นผู้ใหญ่');
} else {
    console.log('คุณยังเป็นเยาวชน');
}

// ตัวอย่างที่ 3: if/else if/else
const score = 75;

if (score >= 90) {
    console.log('เกรด A - ดีเยี่ยม!');
} else if (score >= 80) {
    console.log('เกรด B - ดีมาก');
} else if (score >= 70) {
    console.log('เกรด C - ดี');
} else if (score >= 60) {
    console.log('เกรด D - พอใช้');
} else {
    console.log('เกรด F - ไม่ผ่าน');
}
```

```javascript
// ตัวอย่างที่ 4: Nested if
const isLoggedIn = true;
const isAdmin = false;
const isPremium = true;

if (isLoggedIn) {
    if (isAdmin) {
        console.log('ยินดีต้อนรับ ผู้ดูแลระบบ');
    } else if (isPremium) {
        console.log('ยินดีต้อนรับ สมาชิก Premium');
    } else {
        console.log('ยินดีต้อนรับ สมาชิก');
    }
} else {
    console.log('กรุณาเข้าสู่ระบบก่อน');
}

// ตัวอย่างที่ 5: Truthy/Falsy ใน if condition
const name = '';        // falsy
const count = 0;       // falsy
const items = [];      // truthy!
const user = null;     // falsy

if (!name) console.log('ไม่มีชื่อ');
if (!count) console.log('ไม่มีจำนวน');
if (items) console.log('มี items array (แม้ว่างเปล่า)');
if (!user) console.log('ไม่มีผู้ใช้');
```

```javascript
// ตัวอย่างที่ 6: Guard Clauses Pattern (Early Return)
// แบบที่ไม่ดี - Deeply nested
function processOrderBad(order) {
    if (order) {
        if (order.items && order.items.length > 0) {
            if (order.payment) {
                if (order.payment.status === 'paid') {
                    console.log('ประมวลผลคำสั่งซื้อ');
                    return 'success';
                }
            }
        }
    }
    return 'failed';
}

// แบบที่ดี - Guard Clauses
function processOrder(order) {
    if (!order) return 'ไม่มีข้อมูลออเดอร์';
    if (!order.items || order.items.length === 0) return 'ไม่มีรายการสินค้า';
    if (!order.payment) return 'ไม่มีข้อมูลการชำระเงิน';
    if (order.payment.status !== 'paid') return 'ยังไม่ได้ชำระเงิน';
    
    // ถึงตรงนี้แน่ใจว่าทุกอย่างถูกต้อง
    console.log('ประมวลผลคำสั่งซื้อ...');
    return 'success';
}

console.log(processOrder(null));
console.log(processOrder({ items: [] }));
console.log(processOrder({ items: [1], payment: { status: 'pending' } }));
console.log(processOrder({ items: [1], payment: { status: 'paid' } }));
```

```javascript
// ตัวอย่างที่ 7: If กับ Type Checking
function processInput(value) {
    if (value === null || value === undefined) {
        return 'ค่าว่าง';
    }
    
    if (typeof value === 'number') {
        if (isNaN(value)) return 'ตัวเลขไม่ถูกต้อง';
        if (!isFinite(value)) return 'ตัวเลขไม่สิ้นสุด';
        return `ตัวเลข: ${value}`;
    }
    
    if (typeof value === 'string') {
        if (value.trim() === '') return 'ข้อความว่าง';
        return `ข้อความ: "${value}"`;
    }
    
    if (Array.isArray(value)) {
        return `Array มี ${value.length} elements`;
    }
    
    if (typeof value === 'object') {
        return `Object มี ${Object.keys(value).length} keys`;
    }
    
    return `ชนิดอื่น: ${typeof value}`;
}

const testValues = [null, undefined, 42, NaN, Infinity, '', '  ', 'hello', [], [1,2,3], {}, {a:1,b:2}];
testValues.forEach(v => console.log(processInput(v)));
```

---

## Step 52: switch Statement

```javascript
// ตัวอย่างที่ 8: switch พื้นฐาน
const day = new Date().getDay();

switch (day) {
    case 0:
        console.log('วันอาทิตย์');
        break;
    case 1:
        console.log('วันจันทร์');
        break;
    case 2:
        console.log('วันอังคาร');
        break;
    case 3:
        console.log('วันพุธ');
        break;
    case 4:
        console.log('วันพฤหัสบดี');
        break;
    case 5:
        console.log('วันศุกร์');
        break;
    case 6:
        console.log('วันเสาร์');
        break;
    default:
        console.log('ไม่รู้จักวัน');
}
```

```javascript
// ตัวอย่างที่ 9: switch Fall-through (ไม่มี break)
const month = 4; // เมษายน

switch (month) {
    case 12:
    case 1:
    case 2:
        console.log('ฤดูหนาว');
        break;
    case 3:
    case 4:
    case 5:
        console.log('ฤดูร้อน');  // เดือน 4 จะแสดงนี้
        break;
    case 6:
    case 7:
    case 8:
    case 9:
    case 10:
        console.log('ฤดูฝน');
        break;
    default:
        console.log('ไม่รู้จักเดือน');
}
// ฤดูร้อน

// ตัวอย่างที่ 10: switch กับ string
const command = 'start';

switch (command) {
    case 'start':
    case 'begin':
        console.log('เริ่มต้น...');
        break;
    case 'stop':
    case 'end':
        console.log('หยุด...');
        break;
    case 'pause':
        console.log('หยุดชั่วคราว...');
        break;
    default:
        console.log(`ไม่รู้จักคำสั่ง: ${command}`);
}
```

```javascript
// ตัวอย่างที่ 11: switch vs if/else - เมื่อไหรใช้อะไร
// switch เหมาะเมื่อตรวจสอบค่าเดียวหลาย case
// if/else เหมาะเมื่อมี conditions ซับซ้อน

// ตัวอย่างที่ดีสำหรับ switch
function getMonthName(month) {
    switch (month) {
        case 1: return 'มกราคม';
        case 2: return 'กุมภาพันธ์';
        case 3: return 'มีนาคม';
        case 4: return 'เมษายน';
        case 5: return 'พฤษภาคม';
        case 6: return 'มิถุนายน';
        case 7: return 'กรกฎาคม';
        case 8: return 'สิงหาคม';
        case 9: return 'กันยายน';
        case 10: return 'ตุลาคม';
        case 11: return 'พฤศจิกายน';
        case 12: return 'ธันวาคม';
        default: return 'ไม่รู้จักเดือน';
    }
}

// ทางเลือก - Object lookup (มักเร็วกว่า)
const MONTHS = {
    1: 'มกราคม', 2: 'กุมภาพันธ์', 3: 'มีนาคม',
    4: 'เมษายน', 5: 'พฤษภาคม', 6: 'มิถุนายน',
    7: 'กรกฎาคม', 8: 'สิงหาคม', 9: 'กันยายน',
    10: 'ตุลาคม', 11: 'พฤศจิกายน', 12: 'ธันวาคม'
};

const getMonth = (n) => MONTHS[n] ?? 'ไม่รู้จักเดือน';
console.log(getMonth(5)); // พฤษภาคม
```

```javascript
// ตัวอย่างที่ 12: switch ใน state machine
class TrafficLight {
    #state = 'red';
    
    next() {
        switch (this.#state) {
            case 'red':
                this.#state = 'green';
                break;
            case 'green':
                this.#state = 'yellow';
                break;
            case 'yellow':
                this.#state = 'red';
                break;
        }
        return this;
    }
    
    get status() {
        switch (this.#state) {
            case 'red': return '🔴 หยุด';
            case 'yellow': return '🟡 ระวัง';
            case 'green': return '🟢 ไป';
        }
    }
}

const light = new TrafficLight();
for (let i = 0; i < 7; i++) {
    console.log(light.status);
    light.next();
}
```

---

## Step 53: while Loop

```javascript
// ตัวอย่างที่ 13: while พื้นฐาน
let i = 0;
while (i < 5) {
    console.log(`รอบที่ ${i + 1}`);
    i++;
}

// ตัวอย่างที่ 14: while กับ user input (จำลอง)
function simulateUserInput() {
    const inputs = ['n', 'n', 'y']; // จำลอง user กด y ครั้งที่ 3
    let index = 0;
    
    while (true) {
        const input = inputs[index++];
        console.log(`User กด: ${input}`);
        
        if (input === 'y') {
            console.log('ผู้ใช้ยืนยัน!');
            break;
        }
        
        if (index >= inputs.length) {
            console.log('หมดเวลา');
            break;
        }
    }
}

simulateUserInput();
```

```javascript
// ตัวอย่างที่ 15: while สำหรับประมวลผลข้อมูล
function processQueue(items) {
    const queue = [...items];
    const results = [];
    
    while (queue.length > 0) {
        const item = queue.shift(); // ดึงจากหน้า queue
        console.log(`ประมวลผล: ${item}`);
        results.push(item.toUpperCase());
    }
    
    return results;
}

const tasks = ['task1', 'task2', 'task3', 'task4'];
const processed = processQueue(tasks);
console.log('ผลลัพธ์:', processed);

// ตัวอย่างที่ 16: while กับ recursion-like
function countDown(n) {
    while (n > 0) {
        process.stdout.write(`${n} `);
        n--;
    }
    console.log('Go!');
}
countDown(5); // 5 4 3 2 1 Go!

// ตัวอย่างที่ 17: while ค้นหาใน linked list (concept)
function findInList(head, target) {
    let current = head;
    let steps = 0;
    
    while (current !== null) {
        steps++;
        if (current.value === target) {
            return { found: true, steps };
        }
        current = current.next;
    }
    
    return { found: false, steps };
}

// สร้าง linked list อย่างง่าย
const list = {
    value: 1,
    next: { value: 2, next: { value: 3, next: { value: 4, next: null } } }
};

console.log(findInList(list, 3)); // { found: true, steps: 3 }
console.log(findInList(list, 5)); // { found: false, steps: 4 }
```

---

## Step 54: do...while Loop

```javascript
// ตัวอย่างที่ 18: do...while พื้นฐาน
// ต่างจาก while: ทำงานอย่างน้อย 1 ครั้งเสมอ
let attempt = 0;
do {
    attempt++;
    console.log(`ความพยายามครั้งที่ ${attempt}`);
} while (attempt < 3);
// ความพยายามครั้งที่ 1
// ความพยายามครั้งที่ 2
// ความพยายามครั้งที่ 3

// แม้ condition เป็น false ตั้งแต่แรก ก็ยังทำ 1 ครั้ง
let x = 10;
do {
    console.log('ทำงานแม้ condition เป็น false'); // แสดงครั้งเดียว
} while (x < 5);
```

```javascript
// ตัวอย่างที่ 19: do...while สำหรับ Menu
function showMenu() {
    const options = ['1', '2', '3', '4'];
    let choice;
    let callCount = 0;
    
    do {
        // จำลองการแสดง menu และรับ input
        const inputs = ['5', 'abc', '2']; // user กรอก invalid แล้วค่อยใส่ถูก
        choice = inputs[callCount++];
        
        console.log(`\n=== เมนูหลัก ===`);
        console.log('1. ดูสินค้า');
        console.log('2. เพิ่มสินค้า');
        console.log('3. ลบสินค้า');
        console.log('4. ออก');
        console.log(`คุณเลือก: ${choice}`);
        
        if (!options.includes(choice)) {
            console.log('เลือกไม่ถูกต้อง กรุณาเลือกใหม่');
        }
    } while (!options.includes(choice));
    
    console.log(`\nคุณเลือกตัวเลือก: ${choice}`);
    return choice;
}

showMenu();
```

```javascript
// ตัวอย่างที่ 20: do...while สำหรับ Retry Logic
async function retryOperation(fn, maxRetries = 3, delay = 1000) {
    let retries = 0;
    let lastError;
    
    do {
        try {
            const result = await fn();
            console.log(`สำเร็จหลังลองครั้งที่ ${retries + 1}`);
            return result;
        } catch (error) {
            lastError = error;
            retries++;
            if (retries < maxRetries) {
                console.log(`ล้มเหลวครั้งที่ ${retries}, ลองใหม่...`);
                // await sleep(delay);
            }
        }
    } while (retries < maxRetries);
    
    throw new Error(`ล้มเหลวหลังจากลอง ${maxRetries} ครั้ง: ${lastError.message}`);
}

// ตัวอย่างจำลอง
let callCount = 0;
function unreliableAPI() {
    callCount++;
    if (callCount < 3) throw new Error('Connection failed');
    return { data: 'success' };
}

// จำลองแบบ synchronous
function syncRetry(fn, maxRetries = 3) {
    let retries = 0;
    do {
        try {
            return fn();
        } catch (e) {
            retries++;
            console.log(`Retry ${retries}...`);
        }
    } while (retries < maxRetries);
    return null;
}

callCount = 0;
const result = syncRetry(unreliableAPI);
console.log(result); // { data: 'success' }
```

---

## Step 55: for Loop

```javascript
// ตัวอย่างที่ 21: for พื้นฐาน
for (let i = 0; i < 5; i++) {
    console.log(`i = ${i}`);
}

// ตัวอย่างที่ 22: for วนถอยหลัง
for (let i = 5; i >= 1; i--) {
    process.stdout.write(`${i} `);
}
console.log(); // newline

// ตัวอย่างที่ 23: for ด้วย step ต่างๆ
// Step = 2
for (let i = 0; i <= 10; i += 2) {
    process.stdout.write(`${i} `);
}
console.log(); // 0 2 4 6 8 10

// Step = 3
for (let i = 0; i <= 30; i += 3) {
    process.stdout.write(`${i} `);
}
console.log(); // 0 3 6 9 12 15 18 21 24 27 30
```

```javascript
// ตัวอย่างที่ 24: for กับ Array
const fruits = ['แอปเปิ้ล', 'กล้วย', 'ส้ม', 'มะม่วง', 'สับปะรด'];

// วนด้วย index
for (let i = 0; i < fruits.length; i++) {
    console.log(`${i + 1}. ${fruits[i]}`);
}

// ตัวอย่างที่ 25: for กับ String
const text = 'สวัสดีโลก';
for (let i = 0; i < text.length; i++) {
    process.stdout.write(text[i] + '-');
}
console.log();

// ตัวอย่างที่ 26: for ซ้อน (Nested)
// ตารางสูตรคูณ
for (let i = 1; i <= 5; i++) {
    let row = '';
    for (let j = 1; j <= 5; j++) {
        row += `${i * j}`.padStart(4);
    }
    console.log(row);
}
```

```javascript
// ตัวอย่างที่ 27: for กับ Matrix (2D Array)
const matrix = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
];

// แสดง matrix
console.log('Matrix:');
for (let row = 0; row < matrix.length; row++) {
    let rowStr = '';
    for (let col = 0; col < matrix[row].length; col++) {
        rowStr += matrix[row][col].toString().padStart(3);
    }
    console.log(rowStr);
}

// หาผลรวมทั้งหมด
let sum = 0;
for (let row = 0; row < matrix.length; row++) {
    for (let col = 0; col < matrix[row].length; col++) {
        sum += matrix[row][col];
    }
}
console.log(`ผลรวม: ${sum}`); // 45

// Transpose matrix
const transposed = [];
for (let col = 0; col < matrix[0].length; col++) {
    transposed[col] = [];
    for (let row = 0; row < matrix.length; row++) {
        transposed[col][row] = matrix[row][col];
    }
}
console.log('Transposed:', transposed);
```

```javascript
// ตัวอย่างที่ 28: Classic Algorithms ด้วย for loop
// Bubble Sort
function bubbleSort(arr) {
    const n = arr.length;
    const sorted = [...arr];
    
    for (let i = 0; i < n - 1; i++) {
        let swapped = false;
        
        for (let j = 0; j < n - i - 1; j++) {
            if (sorted[j] > sorted[j + 1]) {
                [sorted[j], sorted[j + 1]] = [sorted[j + 1], sorted[j]];
                swapped = true;
            }
        }
        
        if (!swapped) break; // optimize: หยุดถ้าไม่มีการ swap
    }
    
    return sorted;
}

// Selection Sort
function selectionSort(arr) {
    const sorted = [...arr];
    const n = sorted.length;
    
    for (let i = 0; i < n - 1; i++) {
        let minIndex = i;
        for (let j = i + 1; j < n; j++) {
            if (sorted[j] < sorted[minIndex]) {
                minIndex = j;
            }
        }
        if (minIndex !== i) {
            [sorted[i], sorted[minIndex]] = [sorted[minIndex], sorted[i]];
        }
    }
    
    return sorted;
}

const numbers = [64, 34, 25, 12, 22, 11, 90];
console.log('Original:', numbers);
console.log('Bubble Sort:', bubbleSort(numbers));
console.log('Selection Sort:', selectionSort(numbers));
```

---

## Step 56: for...in Loop

```javascript
// ตัวอย่างที่ 29: for...in กับ Object
const student = {
    name: 'สมชาย',
    age: 20,
    grade: 'A',
    score: 95
};

for (const key in student) {
    console.log(`${key}: ${student[key]}`);
}
// name: สมชาย
// age: 20
// grade: A
// score: 95

// ตัวอย่างที่ 30: for...in ตรวจสอบ own properties
const parent = { inherited: true };
const child = Object.create(parent);
child.own = 'own property';

for (const key in child) {
    if (child.hasOwnProperty(key)) {
        console.log(`Own: ${key} = ${child[key]}`);
    } else {
        console.log(`Inherited: ${key} = ${child[key]}`);
    }
}
```

```javascript
// ตัวอย่างที่ 31: for...in กับ Array (ไม่แนะนำ!)
const arr = [10, 20, 30];
arr.customProp = 'extra'; // เพิ่ม property

// for...in บน array - อันตราย!
for (const key in arr) {
    console.log(`${key}: ${arr[key]}`);
}
// 0: 10, 1: 20, 2: 30, customProp: extra

// แนะนำใช้ for...of หรือ forEach แทน
arr.forEach((val, idx) => console.log(`${idx}: ${val}`));

// ตัวอย่างที่ 32: Dynamic Object Traversal
function deepPrint(obj, indent = 0) {
    for (const key in obj) {
        if (!obj.hasOwnProperty(key)) continue;
        
        const value = obj[key];
        const prefix = '  '.repeat(indent);
        
        if (typeof value === 'object' && value !== null && !Array.isArray(value)) {
            console.log(`${prefix}${key}:`);
            deepPrint(value, indent + 1);
        } else {
            console.log(`${prefix}${key}: ${Array.isArray(value) ? JSON.stringify(value) : value}`);
        }
    }
}

const config = {
    app: {
        name: 'MyApp',
        version: '1.0.0',
        features: ['auth', 'api', 'ui']
    },
    database: {
        host: 'localhost',
        port: 5432
    }
};

deepPrint(config);
```

---

## Step 57: for...of Loop

```javascript
// ตัวอย่างที่ 33: for...of กับ Array
const colors = ['แดง', 'เขียว', 'น้ำเงิน', 'เหลือง'];

for (const color of colors) {
    console.log(color);
}

// กับ index ด้วย entries()
for (const [index, color] of colors.entries()) {
    console.log(`${index + 1}. ${color}`);
}

// ตัวอย่างที่ 34: for...of กับ String
const message = 'Hello!';
for (const char of message) {
    process.stdout.write(char + ' ');
}
console.log(); // H e l l o !

// ตัวอย่างที่ 35: for...of กับ Set
const uniqueValues = new Set([1, 2, 2, 3, 3, 3, 4]);
for (const value of uniqueValues) {
    process.stdout.write(`${value} `);
}
console.log(); // 1 2 3 4

// ตัวอย่างที่ 36: for...of กับ Map
const userMap = new Map([
    ['alice', { age: 25, role: 'admin' }],
    ['bob', { age: 30, role: 'user' }],
    ['charlie', { age: 22, role: 'user' }]
]);

for (const [name, data] of userMap) {
    console.log(`${name}: อายุ ${data.age}, บทบาท ${data.role}`);
}
```

```javascript
// ตัวอย่างที่ 37: for...of กับ Generator
function* range(start, end, step = 1) {
    for (let i = start; i < end; i += step) {
        yield i;
    }
}

for (const n of range(0, 10, 2)) {
    process.stdout.write(`${n} `);
}
console.log(); // 0 2 4 6 8

// ตัวอย่างที่ 38: for...of กับ Iterable custom object
class Counter {
    constructor(from, to) {
        this.from = from;
        this.to = to;
    }
    
    [Symbol.iterator]() {
        let current = this.from;
        const last = this.to;
        
        return {
            next() {
                if (current <= last) {
                    return { value: current++, done: false };
                }
                return { value: undefined, done: true };
            }
        };
    }
}

const counter = new Counter(1, 5);
for (const n of counter) {
    process.stdout.write(`${n} `);
}
console.log(); // 1 2 3 4 5
```

```javascript
// ตัวอย่างที่ 39: for...of vs forEach vs for - เมื่อไหรใช้อะไร
const data = [1, 2, 3, 4, 5];

// forEach: สำหรับ side effects, ใช้ index ได้, ไม่สามารถ break
data.forEach(item => process.stdout.write(`${item} `));
console.log();

// for...of: สำหรับ iteration, ใช้ break/continue ได้, ทำงานกับ iterable ทุกอย่าง
for (const item of data) {
    if (item === 3) break;
    process.stdout.write(`${item} `);
}
console.log(); // 1 2

// for loop: เมื่อต้องการ index, การ skip, หรือ modify ระหว่างวน
for (let i = 0; i < data.length; i += 2) {
    process.stdout.write(`${data[i]} `);
}
console.log(); // 1 3 5
```

---

## Step 58: break Statement

```javascript
// ตัวอย่างที่ 40: break พื้นฐาน
for (let i = 0; i < 10; i++) {
    if (i === 5) break; // หยุด loop เมื่อ i === 5
    process.stdout.write(`${i} `);
}
console.log(); // 0 1 2 3 4

// ตัวอย่างที่ 41: break ใน while
let num = 1;
while (true) { // infinite loop
    if (num > 100 && num % 7 === 0) {
        console.log(`เลขแรกที่มากกว่า 100 และหาร 7 ลงตัว: ${num}`);
        break;
    }
    num++;
}

// ตัวอย่างที่ 42: break ใน switch
function processCommand(cmd) {
    let result = '';
    
    switch (cmd) {
        case 'quit':
        case 'exit':
            result = 'กำลังออกจากโปรแกรม...';
            break;
        case 'help':
            result = 'คำสั่งที่มี: quit, exit, help, status';
            break;
        case 'status':
            result = 'ระบบทำงานปกติ';
            break;
        default:
            result = `ไม่รู้จักคำสั่ง: ${cmd}`;
    }
    
    return result;
}

['help', 'status', 'quit', 'xyz'].forEach(cmd => {
    console.log(`> ${cmd}: ${processCommand(cmd)}`);
});
```

```javascript
// ตัวอย่างที่ 43: break ในการค้นหา
function findFirst(arr, predicate) {
    let found = null;
    let foundIndex = -1;
    
    for (let i = 0; i < arr.length; i++) {
        if (predicate(arr[i])) {
            found = arr[i];
            foundIndex = i;
            break; // หยุดทันทีที่เจอ
        }
    }
    
    return { value: found, index: foundIndex };
}

const numbers = [5, 12, 8, 130, 44, 60, 7];
const result = findFirst(numbers, n => n > 100);
console.log(result); // { value: 130, index: 3 }

// เทียบกับ Array method
const foundIndex = numbers.findIndex(n => n > 100);
console.log(foundIndex); // 3
```

---

## Step 59: continue Statement

```javascript
// ตัวอย่างที่ 44: continue พื้นฐาน
for (let i = 0; i < 10; i++) {
    if (i % 2 === 0) continue; // ข้ามเลขคู่
    process.stdout.write(`${i} `);
}
console.log(); // 1 3 5 7 9

// ตัวอย่างที่ 45: continue กับ data filtering
const students = [
    { name: 'สมชาย', score: 85, active: true },
    { name: 'สมหญิง', score: 45, active: true },
    { name: 'สมศักดิ์', score: 92, active: false },
    { name: 'สมปอง', score: 72, active: true },
    { name: 'สมใจ', score: 38, active: true }
];

console.log('นักเรียน Active ที่ผ่าน (>=60):');
for (const student of students) {
    if (!student.active) continue;      // ข้ามนักเรียนที่ inactive
    if (student.score < 60) continue;   // ข้ามนักเรียนที่ไม่ผ่าน
    
    console.log(`${student.name}: ${student.score}`);
}
```

```javascript
// ตัวอย่างที่ 46: continue ใน while
let n = 0;
const results = [];

while (n < 30) {
    n++;
    if (n % 2 === 0) continue;  // ข้ามเลขคู่
    if (n % 5 === 0) continue;  // ข้าม multiple ของ 5
    results.push(n);
}

console.log('เลขคี่ที่ไม่ใช่ multiple ของ 5:', results.join(', '));
// 1, 3, 7, 9, 11, 13, 17, 19, 21, 23, 27, 29

// ตัวอย่างที่ 47: continue vs filter
// continue เหมาะกับ loop ที่ทำงานอื่นด้วย
// filter เหมาะกับการกรอง array อย่างเดียว

const nums = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

// ด้วย continue
const oddsWithContinue = [];
for (const n of nums) {
    if (n % 2 === 0) continue;
    oddsWithContinue.push(n * 2);
}

// ด้วย filter + map (แนะนำสำหรับ array)
const oddsWithFilter = nums.filter(n => n % 2 !== 0).map(n => n * 2);

console.log(oddsWithContinue); // [2, 6, 10, 14, 18]
console.log(oddsWithFilter);   // [2, 6, 10, 14, 18]
```

---

## Step 60: Labeled Statements

```javascript
// ตัวอย่างที่ 48: Labeled break
outer: for (let i = 0; i < 5; i++) {
    for (let j = 0; j < 5; j++) {
        if (i === 2 && j === 2) {
            console.log(`พบค่าที่ต้องการที่ i=${i}, j=${j}`);
            break outer; // หยุด outer loop ด้วย
        }
        process.stdout.write(`(${i},${j}) `);
    }
}
console.log('\nหลัง labeled break');

// ตัวอย่างที่ 49: Labeled continue
console.log('\nLabeled continue:');
outerLoop: for (let i = 0; i < 3; i++) {
    for (let j = 0; j < 3; j++) {
        if (j === 1) continue outerLoop; // ข้ามไป outer loop iteration ถัดไป
        console.log(`i=${i}, j=${j}`);
    }
}
// แสดงเฉพาะ j=0 สำหรับแต่ละ i
```

```javascript
// ตัวอย่างที่ 50: Practical use ของ labeled statements
function findPairWithSum(matrix, target) {
    search: for (let row = 0; row < matrix.length; row++) {
        for (let col = 0; col < matrix[row].length; col++) {
            for (let r2 = row; r2 < matrix.length; r2++) {
                for (let c2 = (r2 === row ? col + 1 : 0); c2 < matrix[r2].length; c2++) {
                    if (matrix[row][col] + matrix[r2][c2] === target) {
                        console.log(`Found: ${matrix[row][col]} + ${matrix[r2][c2]} = ${target}`);
                        console.log(`At: [${row}][${col}] and [${r2}][${c2}]`);
                        break search;
                    }
                }
            }
        }
    }
}

const grid = [
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
];

findPairWithSum(grid, 10); // Found: 1 + 9 = 10
```

---

## Step 61: Nested Loops

```javascript
// ตัวอย่างที่ 51: ตารางสูตรคูณแบบสวยงาม
console.log('=== ตารางสูตรคูณ ===');
// Header
process.stdout.write('   ');
for (let i = 1; i <= 10; i++) {
    process.stdout.write(String(i).padStart(5));
}
console.log();
console.log('-'.repeat(55));

for (let i = 1; i <= 10; i++) {
    process.stdout.write(String(i).padStart(2) + ' |');
    for (let j = 1; j <= 10; j++) {
        process.stdout.write(String(i * j).padStart(5));
    }
    console.log();
}

// ตัวอย่างที่ 52: Pattern Printing
function printPattern(type, size) {
    switch (type) {
        case 'square':
            for (let i = 0; i < size; i++) {
                console.log('*'.repeat(size));
            }
            break;
            
        case 'triangle':
            for (let i = 1; i <= size; i++) {
                console.log('*'.repeat(i));
            }
            break;
            
        case 'pyramid':
            for (let i = 1; i <= size; i++) {
                const spaces = ' '.repeat(size - i);
                const stars = '*'.repeat(2 * i - 1);
                console.log(spaces + stars);
            }
            break;
            
        case 'hollow':
            for (let i = 0; i < size; i++) {
                let row = '';
                for (let j = 0; j < size; j++) {
                    if (i === 0 || i === size-1 || j === 0 || j === size-1) {
                        row += '*';
                    } else {
                        row += ' ';
                    }
                }
                console.log(row);
            }
            break;
    }
}

printPattern('pyramid', 5);
console.log();
printPattern('hollow', 6);
```

```javascript
// ตัวอย่างที่ 53: Matrix Operations
class Matrix {
    #data;
    #rows;
    #cols;
    
    constructor(rows, cols, fill = 0) {
        this.#rows = rows;
        this.#cols = cols;
        this.#data = Array.from({ length: rows }, () => Array(cols).fill(fill));
    }
    
    static from2DArray(arr) {
        const m = new Matrix(arr.length, arr[0].length);
        for (let i = 0; i < arr.length; i++) {
            for (let j = 0; j < arr[i].length; j++) {
                m.set(i, j, arr[i][j]);
            }
        }
        return m;
    }
    
    get(row, col) { return this.#data[row][col]; }
    set(row, col, value) { this.#data[row][col] = value; }
    
    multiply(other) {
        if (this.#cols !== other.#rows) throw new Error('Invalid dimensions');
        
        const result = new Matrix(this.#rows, other.#cols);
        for (let i = 0; i < this.#rows; i++) {
            for (let j = 0; j < other.#cols; j++) {
                let sum = 0;
                for (let k = 0; k < this.#cols; k++) {
                    sum += this.get(i, k) * other.get(k, j);
                }
                result.set(i, j, sum);
            }
        }
        return result;
    }
    
    toString() {
        return this.#data.map(row => 
            row.map(n => String(n).padStart(6)).join('')
        ).join('\n');
    }
}

const m1 = Matrix.from2DArray([[1, 2], [3, 4]]);
const m2 = Matrix.from2DArray([[5, 6], [7, 8]]);
const product = m1.multiply(m2);

console.log('A:');
console.log(m1.toString());
console.log('\nB:');
console.log(m2.toString());
console.log('\nA × B:');
console.log(product.toString());
```

---

## Step 62: Loop Performance

```javascript
// ตัวอย่างที่ 54: Loop Performance Comparison
function benchmark(name, fn, iterations = 1000000) {
    const start = Date.now();
    fn(iterations);
    const end = Date.now();
    console.log(`${name}: ${end - start}ms`);
}

// for loop
benchmark('for', (n) => {
    let sum = 0;
    for (let i = 0; i < n; i++) sum += i;
    return sum;
});

// while loop
benchmark('while', (n) => {
    let sum = 0;
    let i = 0;
    while (i < n) { sum += i; i++; }
    return sum;
});

// ตัวอย่างที่ 55: Cache array length
const bigArray = new Array(1000000).fill(0).map((_, i) => i);

// ไม่ดี: ตรวจ .length ทุก iteration
console.time('without cache');
for (let i = 0; i < bigArray.length; i++) {
    bigArray[i] * 2;
}
console.timeEnd('without cache');

// ดีกว่า: cache length ไว้
console.time('with cache');
for (let i = 0, len = bigArray.length; i < len; i++) {
    bigArray[i] * 2;
}
console.timeEnd('with cache');
```

---

## Step 63-70: โปรแกรมตัวอย่างที่ซับซ้อน

```javascript
// ตัวอย่างที่ 56: Maze Solver (BFS)
'use strict';

function solveMaze(maze) {
    const rows = maze.length;
    const cols = maze[0].length;
    
    // หา start (S) และ end (E)
    let start, end;
    for (let r = 0; r < rows; r++) {
        for (let c = 0; c < cols; c++) {
            if (maze[r][c] === 'S') start = [r, c];
            if (maze[r][c] === 'E') end = [r, c];
        }
    }
    
    // BFS
    const queue = [[...start, [start]]];
    const visited = new Set([`${start[0]},${start[1]}`]);
    const directions = [[-1,0], [1,0], [0,-1], [0,1]];
    
    while (queue.length > 0) {
        const [row, col, path] = queue.shift();
        
        if (row === end[0] && col === end[1]) {
            return path;
        }
        
        for (const [dr, dc] of directions) {
            const newRow = row + dr;
            const newCol = col + dc;
            const key = `${newRow},${newCol}`;
            
            if (newRow >= 0 && newRow < rows &&
                newCol >= 0 && newCol < cols &&
                maze[newRow][newCol] !== '#' &&
                !visited.has(key)) {
                
                visited.add(key);
                queue.push([newRow, newCol, [...path, [newRow, newCol]]]);
            }
        }
    }
    
    return null;
}

const maze = [
    ['S', '.', '#', '.', '.'],
    ['#', '.', '#', '.', '#'],
    ['#', '.', '.', '.', '#'],
    ['#', '#', '#', '.', '#'],
    ['.', '.', '.', '.', 'E']
];

const path = solveMaze(maze);
if (path) {
    console.log(`พบเส้นทาง! ใช้ ${path.length} ก้าว`);
    
    // แสดง maze พร้อม path
    const displayMaze = maze.map(row => [...row]);
    path.forEach(([r, c]) => {
        if (displayMaze[r][c] === '.') displayMaze[r][c] = '·';
    });
    displayMaze.forEach(row => console.log(row.join(' ')));
} else {
    console.log('ไม่พบเส้นทาง');
}
```

```javascript
// ตัวอย่างที่ 57: Conway's Game of Life
'use strict';

class GameOfLife {
    #grid;
    #rows;
    #cols;
    #generation = 0;
    
    constructor(rows, cols) {
        this.#rows = rows;
        this.#cols = cols;
        this.#grid = Array.from({ length: rows }, () => 
            Array.from({ length: cols }, () => Math.random() < 0.3)
        );
    }
    
    #countNeighbors(row, col) {
        let count = 0;
        for (let dr = -1; dr <= 1; dr++) {
            for (let dc = -1; dc <= 1; dc++) {
                if (dr === 0 && dc === 0) continue;
                const r = (row + dr + this.#rows) % this.#rows;
                const c = (col + dc + this.#cols) % this.#cols;
                if (this.#grid[r][c]) count++;
            }
        }
        return count;
    }
    
    step() {
        const newGrid = Array.from({ length: this.#rows }, () => 
            Array(this.#cols).fill(false)
        );
        
        for (let r = 0; r < this.#rows; r++) {
            for (let c = 0; c < this.#cols; c++) {
                const neighbors = this.#countNeighbors(r, c);
                const alive = this.#grid[r][c];
                
                if (alive) {
                    newGrid[r][c] = neighbors === 2 || neighbors === 3;
                } else {
                    newGrid[r][c] = neighbors === 3;
                }
            }
        }
        
        this.#grid = newGrid;
        this.#generation++;
    }
    
    get aliveCount() {
        let count = 0;
        for (const row of this.#grid) {
            for (const cell of row) {
                if (cell) count++;
            }
        }
        return count;
    }
    
    display() {
        console.log(`Generation: ${this.#generation}, Alive: ${this.aliveCount}`);
        for (const row of this.#grid) {
            console.log(row.map(c => c ? '■' : '□').join(''));
        }
    }
}

const game = new GameOfLife(8, 16);
game.display();
console.log('\n--- 5 generations later ---\n');
for (let i = 0; i < 5; i++) game.step();
game.display();
```

```javascript
// ตัวอย่างที่ 58: CSV Parser
'use strict';

function parseCSV(csv, options = {}) {
    const {
        delimiter = ',',
        hasHeader = true,
        skipEmpty = true
    } = options;
    
    const lines = csv.trim().split('\n');
    const result = [];
    let headers = [];
    
    for (let lineIndex = 0; lineIndex < lines.length; lineIndex++) {
        const line = lines[lineIndex].trim();
        
        if (skipEmpty && line === '') continue;
        
        // Parse fields (handle quoted values)
        const fields = [];
        let current = '';
        let inQuotes = false;
        
        for (let charIndex = 0; charIndex < line.length; charIndex++) {
            const char = line[charIndex];
            
            if (char === '"') {
                if (inQuotes && line[charIndex + 1] === '"') {
                    current += '"';
                    charIndex++;
                } else {
                    inQuotes = !inQuotes;
                }
            } else if (char === delimiter && !inQuotes) {
                fields.push(current);
                current = '';
            } else {
                current += char;
            }
        }
        fields.push(current);
        
        if (hasHeader && lineIndex === 0) {
            headers = fields;
            continue;
        }
        
        if (hasHeader) {
            const row = {};
            for (let i = 0; i < headers.length; i++) {
                row[headers[i]] = fields[i] ?? '';
            }
            result.push(row);
        } else {
            result.push(fields);
        }
    }
    
    return result;
}

const csvData = `name,age,city,occupation
สมชาย,25,กรุงเทพ,นักพัฒนา
สมหญิง,22,เชียงใหม่,ครู
"สมศักดิ์, Jr.",30,ขอนแก่น,"แพทย์"`;

const parsed = parseCSV(csvData);
console.table(parsed);
```

```javascript
// ตัวอย่างที่ 59: Pagination System
'use strict';

function paginate(items, page, pageSize) {
    const totalItems = items.length;
    const totalPages = Math.ceil(totalItems / pageSize);
    const currentPage = Math.max(1, Math.min(page, totalPages));
    const startIndex = (currentPage - 1) * pageSize;
    const endIndex = Math.min(startIndex + pageSize, totalItems);
    
    return {
        items: items.slice(startIndex, endIndex),
        pagination: {
            currentPage,
            totalPages,
            totalItems,
            pageSize,
            startIndex: startIndex + 1,
            endIndex,
            hasPrev: currentPage > 1,
            hasNext: currentPage < totalPages,
            prevPage: currentPage - 1,
            nextPage: currentPage + 1
        }
    };
}

// สร้างข้อมูล 50 รายการ
const allItems = Array.from({ length: 50 }, (_, i) => ({
    id: i + 1,
    name: `สินค้า ${i + 1}`,
    price: Math.floor(Math.random() * 1000) + 100
}));

// แสดงหน้า 3
const { items, pagination } = paginate(allItems, 3, 10);
console.log(`หน้า ${pagination.currentPage}/${pagination.totalPages}`);
console.log(`แสดงรายการที่ ${pagination.startIndex}-${pagination.endIndex} จาก ${pagination.totalItems}`);
console.table(items.map(({ id, name, price }) => ({ id, name, price: `${price} บาท` })));
console.log(`ก่อนหน้า: ${pagination.hasPrev ? `หน้า ${pagination.prevPage}` : 'ไม่มี'}`);
console.log(`ถัดไป: ${pagination.hasNext ? `หน้า ${pagination.nextPage}` : 'ไม่มี'}`);
```

```javascript
// ตัวอย่างที่ 60: Binary Search Tree
'use strict';

class BSTNode {
    constructor(value) {
        this.value = value;
        this.left = null;
        this.right = null;
    }
}

class BST {
    #root = null;
    
    insert(value) {
        if (!this.#root) {
            this.#root = new BSTNode(value);
            return this;
        }
        
        let current = this.#root;
        while (true) {
            if (value === current.value) return this;
            
            if (value < current.value) {
                if (!current.left) {
                    current.left = new BSTNode(value);
                    return this;
                }
                current = current.left;
            } else {
                if (!current.right) {
                    current.right = new BSTNode(value);
                    return this;
                }
                current = current.right;
            }
        }
    }
    
    contains(value) {
        let current = this.#root;
        while (current) {
            if (value === current.value) return true;
            current = value < current.value ? current.left : current.right;
        }
        return false;
    }
    
    // In-order traversal (sorted)
    inOrder() {
        const result = [];
        const stack = [];
        let current = this.#root;
        
        while (current || stack.length > 0) {
            while (current) {
                stack.push(current);
                current = current.left;
            }
            current = stack.pop();
            result.push(current.value);
            current = current.right;
        }
        
        return result;
    }
    
    get min() {
        if (!this.#root) return null;
        let current = this.#root;
        while (current.left) current = current.left;
        return current.value;
    }
    
    get max() {
        if (!this.#root) return null;
        let current = this.#root;
        while (current.right) current = current.right;
        return current.value;
    }
}

const bst = new BST();
[5, 3, 7, 1, 4, 6, 8, 2].forEach(n => bst.insert(n));

console.log('In-order (sorted):', bst.inOrder());
console.log('Min:', bst.min);
console.log('Max:', bst.max);
console.log('Contains 4:', bst.contains(4));
console.log('Contains 9:', bst.contains(9));
```

```javascript
// ตัวอย่างที่ 61: Word Frequency Counter
'use strict';

function analyzeText(text) {
    // Tokenize
    const words = text.toLowerCase()
        .replace(/[^฀-๿a-z0-9\s]/g, '')
        .split(/\s+/)
        .filter(w => w.length > 0);
    
    // Count frequencies
    const freq = new Map();
    for (const word of words) {
        freq.set(word, (freq.get(word) || 0) + 1);
    }
    
    // Sort by frequency
    const sorted = [...freq.entries()].sort((a, b) => b[1] - a[1]);
    
    // Statistics
    const totalWords = words.length;
    const uniqueWords = freq.size;
    const avgFreq = totalWords / uniqueWords;
    
    return {
        totalWords,
        uniqueWords,
        avgFreq: avgFreq.toFixed(2),
        top10: sorted.slice(0, 10),
        frequencies: Object.fromEntries(sorted)
    };
}

const sampleText = `
JavaScript เป็นภาษาโปรแกรมมิ่งที่ดีมาก
JavaScript ใช้ได้ทั้ง frontend และ backend
การเรียน JavaScript ต้องฝึกเขียนโค้ดทุกวัน
โค้ด JavaScript สามารถรันใน browser หรือ Node.js ได้
`;

const analysis = analyzeText(sampleText);
console.log(`คำทั้งหมด: ${analysis.totalWords}`);
console.log(`คำไม่ซ้ำ: ${analysis.uniqueWords}`);
console.log('คำที่พบบ่อย:');
analysis.top10.forEach(([word, count], i) => {
    console.log(`  ${i + 1}. "${word}": ${count} ครั้ง`);
});
```

```javascript
// ตัวอย่างที่ 62: Sudoku Validator
'use strict';

function isValidSudoku(board) {
    const size = 9;
    
    // ตรวจ row
    for (let row = 0; row < size; row++) {
        const seen = new Set();
        for (let col = 0; col < size; col++) {
            const val = board[row][col];
            if (val === 0) continue;
            if (seen.has(val)) {
                console.log(`ซ้ำใน row ${row}`);
                return false;
            }
            seen.add(val);
        }
    }
    
    // ตรวจ column
    for (let col = 0; col < size; col++) {
        const seen = new Set();
        for (let row = 0; row < size; row++) {
            const val = board[row][col];
            if (val === 0) continue;
            if (seen.has(val)) {
                console.log(`ซ้ำใน column ${col}`);
                return false;
            }
            seen.add(val);
        }
    }
    
    // ตรวจ 3x3 boxes
    for (let boxRow = 0; boxRow < 3; boxRow++) {
        for (let boxCol = 0; boxCol < 3; boxCol++) {
            const seen = new Set();
            for (let r = 0; r < 3; r++) {
                for (let c = 0; c < 3; c++) {
                    const val = board[boxRow * 3 + r][boxCol * 3 + c];
                    if (val === 0) continue;
                    if (seen.has(val)) {
                        console.log(`ซ้ำใน box (${boxRow},${boxCol})`);
                        return false;
                    }
                    seen.add(val);
                }
            }
        }
    }
    
    return true;
}

const validBoard = [
    [5, 3, 0, 0, 7, 0, 0, 0, 0],
    [6, 0, 0, 1, 9, 5, 0, 0, 0],
    [0, 9, 8, 0, 0, 0, 0, 6, 0],
    [8, 0, 0, 0, 6, 0, 0, 0, 3],
    [4, 0, 0, 8, 0, 3, 0, 0, 1],
    [7, 0, 0, 0, 2, 0, 0, 0, 6],
    [0, 6, 0, 0, 0, 0, 2, 8, 0],
    [0, 0, 0, 4, 1, 9, 0, 0, 5],
    [0, 0, 0, 0, 8, 0, 0, 7, 9]
];

console.log('Valid Sudoku:', isValidSudoku(validBoard));
```

---

## สรุป Step 51-70

ในส่วนนี้คุณได้เรียนรู้:
- **if/else**: การตรวจสอบเงื่อนไข, guard clauses pattern
- **switch**: case, break, fall-through, default
- **while**: การวนซ้ำพร้อมเงื่อนไข
- **do...while**: วนอย่างน้อย 1 ครั้ง, retry pattern
- **for**: วนด้วย counter, nested loops
- **for...in**: วน property ของ object
- **for...of**: วน iterable (array, string, map, set)
- **break**: หยุด loop
- **continue**: ข้าม iteration
- **Labeled statements**: break/continue ไปยัง loop เฉพาะ

---

## แบบฝึกหัด

### ระดับพื้นฐาน

**แบบฝึกหัดที่ 1**: FizzBuzz Variations
```javascript
// เขียน FizzBuzz ที่ซับซ้อนขึ้น:
// - ถ้าหาร 3 ลงตัว: "Fizz"
// - ถ้าหาร 5 ลงตัว: "Buzz"  
// - ถ้าหาร 7 ลงตัว: "Jazz"
// - รวมกันได้: "FizzBuzz", "FizzJazz", "BuzzJazz", "FizzBuzzJazz"
// แสดง 1-100
```

**แบบฝึกหัดที่ 2**: Number Patterns
```javascript
// แสดงรูปแบบต่อไปนี้:
// 1
// 2 2
// 3 3 3
// 4 4 4 4
// 5 5 5 5 5
```

### ระดับกลาง

**แบบฝึกหัดที่ 3**: Prime Sieve (Sieve of Eratosthenes)
```javascript
// หาเลขเฉพาะทั้งหมดถึง 100 ด้วย Sieve of Eratosthenes
function sieve(n) {
    const isPrime = new Array(n + 1).fill(true);
    // TODO: implement
    return isPrime;
}
```

**แบบฝึกหัดที่ 4**: Flattening Nested Array
```javascript
// Flatten array ซ้อนกันหลายชั้น
// [1, [2, [3, [4, [5]]]]] -> [1, 2, 3, 4, 5]
function flatten(arr) {
    // TODO: implement without using Array.flat()
}
```

### ระดับสูง

**แบบฝึกหัดที่ 5**: Spiral Matrix
```javascript
// สร้าง Spiral Matrix ขนาด n×n
// n=4:
// 1  2  3  4
// 12 13 14  5
// 11 16 15  6
// 10  9  8  7
function spiralMatrix(n) {
    // TODO: implement
}
```

---

## เฉลยแบบฝึกหัด

**เฉลยที่ 3**: Prime Sieve
```javascript
function sieve(n) {
    const isPrime = new Array(n + 1).fill(true);
    isPrime[0] = isPrime[1] = false;
    
    for (let i = 2; i * i <= n; i++) {
        if (isPrime[i]) {
            for (let j = i * i; j <= n; j += i) {
                isPrime[j] = false;
            }
        }
    }
    
    return Array.from({ length: n + 1 }, (_, i) => i).filter(i => isPrime[i]);
}

console.log(sieve(100));
```

**เฉลยที่ 5**: Spiral Matrix
```javascript
function spiralMatrix(n) {
    const matrix = Array.from({ length: n }, () => Array(n).fill(0));
    let top = 0, bottom = n - 1, left = 0, right = n - 1;
    let num = 1;
    
    while (top <= bottom && left <= right) {
        for (let i = left; i <= right; i++) matrix[top][i] = num++;
        top++;
        for (let i = top; i <= bottom; i++) matrix[i][right] = num++;
        right--;
        for (let i = right; i >= left; i--) matrix[bottom][i] = num++;
        bottom--;
        for (let i = bottom; i >= top; i--) matrix[i][left] = num++;
        left++;
    }
    
    return matrix;
}

const spiral = spiralMatrix(4);
spiral.forEach(row => console.log(row.map(n => String(n).padStart(3)).join('')));
```

---

*ส่วนถัดไป: Part 5 - Functions (ฟังก์ชัน)*
