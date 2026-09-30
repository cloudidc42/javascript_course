# ส่วนที่ 5: ฟังก์ชัน (Functions)

## คำอธิบาย
ฟังก์ชันคือส่วนที่สำคัญที่สุดใน JavaScript ในส่วนนี้คุณจะได้เรียนรู้ทุกอย่างเกี่ยวกับฟังก์ชัน ตั้งแต่พื้นฐานไปจนถึง arrow functions, closures, และ higher-order functions เนื้อหาครอบคลุม Step 71-90

---

## Step 71: Function Declaration vs Expression

### Function Declaration

```javascript
// ตัวอย่างที่ 1: Function Declaration
function greet(name) {
    return `สวัสดี ${name}!`;
}

console.log(greet('สมชาย')); // สวัสดี สมชาย!

// ตัวอย่างที่ 2: Function Declaration สามารถเรียกก่อนประกาศได้ (hoisting)
console.log(add(5, 3)); // 8 - ทำงานได้แม้ประกาศทีหลัง!

function add(a, b) {
    return a + b;
}

// ตัวอย่างที่ 3: Named Function ตรวจสอบตัวเองได้
function factorial(n) {
    if (n <= 1) return 1;
    return n * factorial(n - 1); // เรียกตัวเองได้
}

console.log(factorial(5)); // 120
```

### Function Expression

```javascript
// ตัวอย่างที่ 4: Function Expression
const multiply = function(a, b) {
    return a * b;
};

console.log(multiply(4, 5)); // 20

// ตัวอย่างที่ 5: Function Expression ไม่ถูก hoist
// console.log(divide(10, 2)); // ReferenceError! ยังไม่มี divide
const divide = function(a, b) {
    return a / b;
};
console.log(divide(10, 2)); // 5

// ตัวอย่างที่ 6: Named Function Expression (ใช้ชื่อใน stack trace)
const power = function pow(base, exp) {
    if (exp === 0) return 1;
    return base * pow(base, exp - 1); // ใช้ชื่อ pow ได้ภายใน
};

console.log(power(2, 8)); // 256
// console.log(pow(2, 8)); // ReferenceError: pow ไม่ available ภายนอก
```

```javascript
// ตัวอย่างที่ 7: ความแตกต่าง Declaration vs Expression
// Declaration: hoisted, มีชื่อใน scope
// Expression: ไม่ hoisted, assign ให้ตัวแปร

// ใช้เป็น callback
const numbers = [3, 1, 4, 1, 5, 9, 2, 6];

// Expression เป็น callback
const sorted = numbers.sort(function(a, b) { return a - b; });
console.log(sorted); // [1, 1, 2, 3, 4, 5, 6, 9]

// Declaration ก็ใช้เป็น callback ได้
function compare(a, b) { return a - b; }
const sorted2 = [...numbers].sort(compare);
console.log(sorted2);

// ตัวอย่างที่ 8: Function เป็น First-class citizen
function applyOperation(x, y, operation) {
    return operation(x, y);
}

function addFn(a, b) { return a + b; }
function subtractFn(a, b) { return a - b; }

console.log(applyOperation(10, 3, addFn));      // 13
console.log(applyOperation(10, 3, subtractFn)); // 7
console.log(applyOperation(10, 3, function(a, b) { return a * b; })); // 30
```

---

## Step 72: Parameters และ Arguments

```javascript
// ตัวอย่างที่ 9: Parameters vs Arguments
function greet(firstName, lastName) { // firstName, lastName = parameters
    return `สวัสดี ${firstName} ${lastName}`;
}

greet('สม', 'ชาย'); // 'สม', 'ชาย' = arguments

// ตัวอย่างที่ 10: Arguments object (ใน non-arrow function)
function sumAll() {
    console.log(arguments); // array-like object
    let total = 0;
    for (let i = 0; i < arguments.length; i++) {
        total += arguments[i];
    }
    return total;
}

console.log(sumAll(1, 2, 3));        // 6
console.log(sumAll(1, 2, 3, 4, 5)); // 15

// ตัวอย่างที่ 11: ส่ง arguments น้อยกว่า parameters
function show(a, b, c) {
    console.log(a, b, c);
}

show(1, 2);    // 1 2 undefined
show(1);       // 1 undefined undefined
show();        // undefined undefined undefined
```

```javascript
// ตัวอย่างที่ 12: ส่ง arguments มากกว่า parameters
function add(a, b) {
    return a + b; // รับแค่ 2 ตัว แต่ส่ง 3 ก็ไม่ error
}

console.log(add(1, 2, 3)); // 3 (ตัวที่ 3 ถูกเพิกเฉย)

// ตัวอย่างที่ 13: ส่ง function เป็น argument
function doMath(numbers, operation) {
    return numbers.reduce(operation, 0);
}

const nums = [1, 2, 3, 4, 5];
const sum = doMath(nums, (acc, n) => acc + n);
const product = doMath(nums, (acc, n) => acc * n || 1);
console.log(sum);     // 15
console.log(product); // 0 (ขึ้นอยู่กับ initial value)
```

---

## Step 73: Default Parameters

```javascript
// ตัวอย่างที่ 14: Default Parameters (ES6)
function greet(name = 'แขก', greeting = 'สวัสดี') {
    return `${greeting} ${name}!`;
}

console.log(greet());                    // สวัสดี แขก!
console.log(greet('สมชาย'));             // สวัสดี สมชาย!
console.log(greet('Bob', 'Hello'));     // Hello Bob!
console.log(greet(undefined, 'Hi'));   // Hi แขก! (undefined ใช้ default)
console.log(greet(null, 'Hi'));        // Hi null! (null ไม่ใช้ default)

// ตัวอย่างที่ 15: Default ใช้ค่าจากตัวแปร
const DEFAULT_TAX = 0.07;

function calculateTotal(price, quantity = 1, tax = DEFAULT_TAX) {
    const subtotal = price * quantity;
    return subtotal + (subtotal * tax);
}

console.log(calculateTotal(100));         // 107
console.log(calculateTotal(100, 3));     // 321
console.log(calculateTotal(100, 2, 0.1)); // 220

// ตัวอย่างที่ 16: Default ใช้ผลลัพธ์จาก expression
function createUser(
    name,
    id = Math.random().toString(36).substr(2, 9),
    createdAt = new Date().toISOString()
) {
    return { id, name, createdAt };
}

console.log(createUser('สมชาย'));
console.log(createUser('สมหญิง'));
```

```javascript
// ตัวอย่างที่ 17: Default ใช้ parameter ก่อนหน้า
function range(start, end = start + 10, step = 1) {
    const result = [];
    for (let i = start; i <= end; i += step) {
        result.push(i);
    }
    return result;
}

console.log(range(1));           // [1, 2, 3, ..., 11]
console.log(range(5, 15));      // [5, 6, 7, ..., 15]
console.log(range(0, 20, 5));  // [0, 5, 10, 15, 20]

// ตัวอย่างที่ 18: Default กับ Destructuring
function initConfig({
    host = 'localhost',
    port = 3000,
    ssl = false,
    debug = false,
    maxRetries = 3
} = {}) { // {} เป็น default ถ้าไม่ส่ง argument
    return { host, port, ssl, debug, maxRetries };
}

console.log(initConfig());                        // ทุก default
console.log(initConfig({ port: 8080, ssl: true })); // custom port & ssl
```

---

## Step 74: Rest Parameters

```javascript
// ตัวอย่างที่ 19: Rest Parameters
function sum(...numbers) {
    return numbers.reduce((total, n) => total + n, 0);
}

console.log(sum(1, 2, 3));           // 6
console.log(sum(1, 2, 3, 4, 5));   // 15
console.log(sum());                    // 0

// ตัวอย่างที่ 20: Rest กับ parameters ปกติ
function logMessage(level, ...messages) {
    const prefix = `[${level.toUpperCase()}]`;
    console.log(prefix, ...messages);
}

logMessage('info', 'เซิร์ฟเวอร์เริ่มแล้ว', 'พอร์ต 3000');
logMessage('error', 'เกิดข้อผิดพลาด', 'ไม่พบไฟล์', 'path/to/file');
logMessage('warn', 'หน่วยความจำเหลือน้อย');

// ตัวอย่างที่ 21: Rest ต้องเป็น parameter สุดท้าย
function firstAndRest(first, second, ...rest) {
    console.log('First:', first);
    console.log('Second:', second);
    console.log('Rest:', rest);
}

firstAndRest(1, 2, 3, 4, 5);
// First: 1, Second: 2, Rest: [3, 4, 5]
```

```javascript
// ตัวอย่างที่ 22: Rest vs Arguments object
// arguments: array-like, ไม่ใช้กับ arrow function, มีทุก argument
// rest: real array, ใช้กับทุก function type, มีเฉพาะที่ระบุ

// arguments
function withArguments() {
    console.log(typeof arguments); // object
    console.log(Array.isArray(arguments)); // false
    // ต้องแปลงก่อนใช้ Array methods
    const arr = Array.from(arguments);
    return arr.reduce((a, b) => a + b, 0);
}

// rest - แนะนำ
function withRest(...args) {
    console.log(Array.isArray(args)); // true
    return args.reduce((a, b) => a + b, 0);
}

// ตัวอย่างที่ 23: Rest ใน Higher-order function
function pipe(...fns) {
    return function(x) {
        return fns.reduce((result, fn) => fn(result), x);
    };
}

const processNumber = pipe(
    n => n * 2,
    n => n + 10,
    n => Math.round(n),
    n => `ผลลัพธ์: ${n}`
);

console.log(processNumber(5));    // ผลลัพธ์: 20
console.log(processNumber(3.5)); // ผลลัพธ์: 17
```

---

## Step 75: Return Values

```javascript
// ตัวอย่างที่ 24: Return Value types
function getNumber() { return 42; }
function getString() { return 'hello'; }
function getArray() { return [1, 2, 3]; }
function getObject() { return { name: 'สมชาย', age: 25 }; }
function getFunction() { return function() { return 'inner'; }; }
function noReturn() {} // คืน undefined

console.log(getNumber());   // 42
console.log(getString());   // hello
console.log(getArray());    // [1, 2, 3]
console.log(getObject());   // { name: 'สมชาย', age: 25 }
console.log(getFunction()()); // 'inner'
console.log(noReturn());    // undefined

// ตัวอย่างที่ 25: Multiple return paths
function classify(n) {
    if (n < 0) return 'negative';
    if (n === 0) return 'zero';
    if (n < 10) return 'small';
    if (n < 100) return 'medium';
    return 'large';
}

[-5, 0, 7, 42, 1000].forEach(n => {
    console.log(`${n}: ${classify(n)}`);
});
```

```javascript
// ตัวอย่างที่ 26: Return Object
function createPerson(name, age) {
    // Validation
    if (typeof name !== 'string' || name.trim() === '') {
        return { success: false, error: 'ชื่อไม่ถูกต้อง' };
    }
    if (typeof age !== 'number' || age < 0 || age > 150) {
        return { success: false, error: 'อายุไม่ถูกต้อง' };
    }
    
    return {
        success: true,
        data: { name: name.trim(), age, id: Date.now() }
    };
}

const result1 = createPerson('สมชาย', 25);
const result2 = createPerson('', 25);
const result3 = createPerson('สมหญิง', -1);

console.log(result1); // { success: true, data: {...} }
console.log(result2); // { success: false, error: 'ชื่อไม่ถูกต้อง' }
console.log(result3); // { success: false, error: 'อายุไม่ถูกต้อง' }

// ตัวอย่างที่ 27: Return Function (Closure)
function multiplier(factor) {
    return function(number) {
        return number * factor;
    };
}

const double = multiplier(2);
const triple = multiplier(3);
const tenTimes = multiplier(10);

console.log(double(5));   // 10
console.log(triple(5));   // 15
console.log(tenTimes(5)); // 50
```

```javascript
// ตัวอย่างที่ 28: Early Return Pattern
function validateForm(data) {
    if (!data) return { valid: false, error: 'ไม่มีข้อมูล' };
    if (!data.username) return { valid: false, error: 'กรุณากรอก username' };
    if (data.username.length < 3) return { valid: false, error: 'Username ต้องยาวกว่า 3 ตัวอักษร' };
    if (!data.email) return { valid: false, error: 'กรุณากรอก email' };
    if (!data.email.includes('@')) return { valid: false, error: 'Email ไม่ถูกต้อง' };
    if (!data.password) return { valid: false, error: 'กรุณากรอก password' };
    if (data.password.length < 8) return { valid: false, error: 'Password ต้องยาวอย่างน้อย 8 ตัว' };
    
    return { valid: true, data };
}

const forms = [
    null,
    { username: 'ab', email: 'test@x.com', password: '12345678' },
    { username: 'john', email: 'not-email', password: '12345678' },
    { username: 'john', email: 'john@example.com', password: '12345678' }
];

forms.forEach(form => {
    const { valid, error } = validateForm(form);
    console.log(valid ? 'ผ่านการตรวจสอบ' : `ผิดพลาด: ${error}`);
});
```

---

## Step 76: Scope

```javascript
// ตัวอย่างที่ 29: Global Scope
let globalVar = 'ตัวแปรระดับ Global';

function showGlobal() {
    console.log(globalVar); // เข้าถึงได้
    globalVar = 'ถูกแก้ไขจาก function'; // แก้ไขได้
}

showGlobal();
console.log(globalVar); // 'ถูกแก้ไขจาก function'

// ตัวอย่างที่ 30: Local/Function Scope
function outer() {
    let outerVar = 'ใน outer';
    
    function inner() {
        let innerVar = 'ใน inner';
        console.log(outerVar); // เข้าถึง outer ได้
        console.log(innerVar); // เข้าถึง inner ได้
    }
    
    inner();
    console.log(outerVar); // เข้าถึงได้
    // console.log(innerVar); // ReferenceError!
}

outer();
// console.log(outerVar); // ReferenceError!

// ตัวอย่างที่ 31: Block Scope
{
    let blockLet = 'block';
    const blockConst = 'block const';
    var blockVar = 'function scope!'; // รั่วออกมา
}

// console.log(blockLet);   // ReferenceError
// console.log(blockConst); // ReferenceError
console.log(blockVar);      // 'function scope!' (var รั่ว)
```

```javascript
// ตัวอย่างที่ 32: Scope Chain
const globalX = 'global';

function first() {
    const firstX = 'first';
    
    function second() {
        const secondX = 'second';
        
        function third() {
            // มีสิทธิ์เข้าถึงทุก scope ข้างบน
            console.log(globalX);  // 'global'
            console.log(firstX);  // 'first'
            console.log(secondX); // 'second'
        }
        
        third();
    }
    
    second();
}

first();

// ตัวอย่างที่ 33: Variable Shadowing
const value = 'global value';

function test() {
    const value = 'function value'; // shadow global
    console.log(value); // 'function value'
    
    {
        const value = 'block value'; // shadow function
        console.log(value); // 'block value'
    }
    
    console.log(value); // 'function value' (block shadow หมดแล้ว)
}

test();
console.log(value); // 'global value'
```

---

## Step 77: Hoisting

```javascript
// ตัวอย่างที่ 34: Function Declaration Hoisting
// เรียกก่อนประกาศได้
sayHello(); // "สวัสดี!" - ทำงานได้!

function sayHello() {
    console.log('สวัสดี!');
}

// ตัวอย่างที่ 35: var Hoisting
console.log(x); // undefined (ไม่ใช่ ReferenceError!)
var x = 5;
console.log(x); // 5

// JavaScript แปลเป็น:
// var x;
// console.log(x); // undefined
// x = 5;
// console.log(x); // 5

// ตัวอย่างที่ 36: let/const ไม่ hoisted (TDZ)
// console.log(y); // ReferenceError: Cannot access 'y' before initialization
let y = 10;
console.log(y); // 10

// ตัวอย่างที่ 37: Function Expression ไม่ hoisted
// greet(); // TypeError: greet is not a function
var greet = function() { return 'สวัสดี'; };
greet(); // OK
```

```javascript
// ตัวอย่างที่ 38: Hoisting ทำให้งง - ตัวอย่างจริง
function tricky() {
    console.log(foo); // undefined (var ถูก hoist)
    var foo = 'bar';
    console.log(foo); // 'bar'
    
    // แต่ถ้าเป็น function:
    inner(); // 'inner function!' - ทำงานได้
    
    function inner() {
        console.log('inner function!');
    }
}

tricky();

// ตัวอย่างที่ 39: Best practice - ประกาศก่อนใช้เสมอ
// แม้ว่า hoisting จะช่วยได้ แต่โค้ดจะอ่านง่ายกว่ามาก
function goodCode() {
    // ประกาศ function ก่อน
    function helper() {
        return 42;
    }
    
    // ประกาศตัวแปรก่อน
    const result = helper();
    
    // ใช้ทีหลัง
    console.log(result);
}
goodCode();
```

---

## Step 78: IIFE (Immediately Invoked Function Expression)

```javascript
// ตัวอย่างที่ 40: IIFE พื้นฐาน
(function() {
    console.log('IIFE ทำงานทันที!');
})();

// หรือแบบนี้
(function() {
    console.log('IIFE แบบที่ 2');
}());

// Arrow function IIFE
(() => {
    console.log('Arrow IIFE');
})();

// ตัวอย่างที่ 41: IIFE สำหรับ isolate scope
(function() {
    var privateVar = 'ไม่รั่วออกไปนอก';
    console.log(privateVar); // OK
})();

// console.log(privateVar); // ReferenceError!
```

```javascript
// ตัวอย่างที่ 42: IIFE ส่ง argument
(function(name, greeting) {
    console.log(`${greeting} ${name}!`);
})('สมชาย', 'สวัสดี');

// ตัวอย่างที่ 43: IIFE return value
const result = (function() {
    const secret = 42;
    return {
        getValue() { return secret; },
        getDouble() { return secret * 2; }
    };
})();

console.log(result.getValue());  // 42
console.log(result.getDouble()); // 84

// ตัวอย่างที่ 44: Module Pattern ด้วย IIFE
const Calculator = (function() {
    let history = []; // private
    
    function validate(a, b) { // private
        return typeof a === 'number' && typeof b === 'number';
    }
    
    return {
        add(a, b) {
            if (!validate(a, b)) throw new Error('Invalid input');
            const result = a + b;
            history.push(`${a} + ${b} = ${result}`);
            return result;
        },
        subtract(a, b) {
            const result = a - b;
            history.push(`${a} - ${b} = ${result}`);
            return result;
        },
        getHistory() {
            return [...history]; // copy เพื่อป้องกัน mutation
        },
        clearHistory() {
            history = [];
        }
    };
})();

console.log(Calculator.add(10, 5));      // 15
console.log(Calculator.subtract(10, 3)); // 7
console.log(Calculator.getHistory());    // ['10 + 5 = 15', '10 - 3 = 7']
```

---

## Step 79: Recursive Functions

```javascript
// ตัวอย่างที่ 45: Recursion พื้นฐาน - Factorial
function factorial(n) {
    // Base case
    if (n <= 1) return 1;
    // Recursive case
    return n * factorial(n - 1);
}

console.log(factorial(0)); // 1
console.log(factorial(5)); // 120
console.log(factorial(10)); // 3628800

// ตัวอย่างที่ 46: Fibonacci ด้วย Recursion
function fibonacci(n) {
    if (n <= 0) return 0;
    if (n === 1) return 1;
    return fibonacci(n - 1) + fibonacci(n - 2);
}

// slow! O(2^n) - ควรใช้ memoization
for (let i = 0; i <= 10; i++) {
    process.stdout.write(`${fibonacci(i)} `);
}
console.log(); // 0 1 1 2 3 5 8 13 21 34 55

// ตัวอย่างที่ 47: Fibonacci ด้วย Memoization
function fibMemo(n, memo = {}) {
    if (n in memo) return memo[n];
    if (n <= 0) return 0;
    if (n === 1) return 1;
    
    memo[n] = fibMemo(n - 1, memo) + fibMemo(n - 2, memo);
    return memo[n];
}

console.log(fibMemo(50)); // 12586269025 - เร็วมาก!
```

```javascript
// ตัวอย่างที่ 48: Recursive Object Operations
function deepClone(obj) {
    // Primitive values
    if (obj === null || typeof obj !== 'object') return obj;
    
    // Array
    if (Array.isArray(obj)) {
        return obj.map(item => deepClone(item));
    }
    
    // Object
    const cloned = {};
    for (const key in obj) {
        if (obj.hasOwnProperty(key)) {
            cloned[key] = deepClone(obj[key]);
        }
    }
    return cloned;
}

const original = {
    name: 'สมชาย',
    scores: [85, 92, 78],
    address: {
        city: 'กรุงเทพ',
        zip: '10400'
    }
};

const cloned = deepClone(original);
cloned.scores.push(95);
cloned.address.city = 'เชียงใหม่';

console.log('Original:', original); // ไม่เปลี่ยน
console.log('Cloned:', cloned);

// ตัวอย่างที่ 49: Recursive Search
function deepSearch(obj, target) {
    for (const key in obj) {
        if (!obj.hasOwnProperty(key)) continue;
        
        if (obj[key] === target) {
            return { found: true, path: [key] };
        }
        
        if (typeof obj[key] === 'object' && obj[key] !== null) {
            const result = deepSearch(obj[key], target);
            if (result.found) {
                return { found: true, path: [key, ...result.path] };
            }
        }
    }
    return { found: false, path: [] };
}

const data = {
    level1: {
        level2: {
            level3: {
                treasure: 'found!'
            }
        }
    }
};

const { found, path } = deepSearch(data, 'found!');
console.log(found); // true
console.log(path.join(' > ')); // level1 > level2 > level3 > treasure
```

```javascript
// ตัวอย่างที่ 50: Tree Operations
class TreeNode {
    constructor(value, children = []) {
        this.value = value;
        this.children = children;
    }
}

function treeSum(node) {
    if (!node) return 0;
    return node.value + node.children.reduce((sum, child) => sum + treeSum(child), 0);
}

function treeDepth(node) {
    if (!node || node.children.length === 0) return 1;
    return 1 + Math.max(...node.children.map(treeDepth));
}

function treeFlatten(node) {
    if (!node) return [];
    return [node.value, ...node.children.flatMap(treeFlatten)];
}

const tree = new TreeNode(1, [
    new TreeNode(2, [
        new TreeNode(5),
        new TreeNode(6)
    ]),
    new TreeNode(3, [
        new TreeNode(7)
    ]),
    new TreeNode(4)
]);

console.log('Sum:', treeSum(tree));        // 28
console.log('Depth:', treeDepth(tree));    // 3
console.log('Flat:', treeFlatten(tree));   // [1, 2, 5, 6, 3, 7, 4]
```

---

## Step 80: Arrow Functions

```javascript
// ตัวอย่างที่ 51: Arrow Function Syntax
// Regular function
const greet1 = function(name) {
    return `สวัสดี ${name}`;
};

// Arrow function
const greet2 = (name) => {
    return `สวัสดี ${name}`;
};

// Short arrow (implicit return)
const greet3 = (name) => `สวัสดี ${name}`;

// Single parameter - ไม่ต้องใส่วงเล็บ
const greet4 = name => `สวัสดี ${name}`;

// No parameters
const sayHi = () => 'สวัสดี!';

console.log(greet1('สมชาย')); // สวัสดี สมชาย
console.log(greet4('สมหญิง')); // สวัสดี สมหญิง
console.log(sayHi()); // สวัสดี!
```

```javascript
// ตัวอย่างที่ 52: Return Object จาก Arrow Function
// ต้องใส่ () ครอบ object
const createPoint = (x, y) => ({ x, y });
const getUser = (name, age) => ({ name, age, createdAt: new Date() });

console.log(createPoint(3, 4));     // { x: 3, y: 4 }
console.log(getUser('สมชาย', 25)); // { name: 'สมชาย', age: 25, ... }

// ตัวอย่างที่ 53: Arrow Functions กับ this
class Timer {
    constructor() {
        this.seconds = 0;
    }
    
    start() {
        // ถ้าใช้ function จะหา this ไม่เจอ
        // setInterval(function() {
        //     this.seconds++; // this = undefined (strict) หรือ window (non-strict)
        // }, 1000);
        
        // Arrow function ใช้ this จาก lexical scope
        setInterval(() => {
            this.seconds++;
            if (this.seconds % 5 === 0) {
                console.log(`ผ่านมา ${this.seconds} วินาที`);
            }
        }, 1000);
    }
}

// ตัวอย่างที่ 54: Arrow Functions ไม่มี arguments object
function regularFn() {
    console.log(arguments); // [Arguments] { '0': 1, '1': 2, '2': 3 }
}
regularFn(1, 2, 3);

const arrowFn = () => {
    // console.log(arguments); // ReferenceError!
    // ใช้ rest แทน
};
```

```javascript
// ตัวอย่างที่ 55: Arrow Functions ใน Array Methods
const numbers = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

// map
const doubled = numbers.map(n => n * 2);
console.log(doubled);

// filter
const evens = numbers.filter(n => n % 2 === 0);
console.log(evens);

// reduce
const sum = numbers.reduce((acc, n) => acc + n, 0);
console.log(sum);

// find
const firstBig = numbers.find(n => n > 7);
console.log(firstBig); // 8

// every / some
console.log(numbers.every(n => n > 0)); // true
console.log(numbers.some(n => n > 9));   // true

// Chaining
const result = numbers
    .filter(n => n % 2 !== 0)  // เลขคี่
    .map(n => n ** 2)            // ยกกำลังสอง
    .filter(n => n > 10)         // มากกว่า 10
    .reduce((sum, n) => sum + n, 0); // รวม

console.log(result); // 9+25+49+81 = 164... wait: 9,25,49,81 -> 1^2=1, 3^2=9, 5^2=25, 7^2=49, 9^2=81 -> filter>10: 25,49,81 -> sum=155
```

---

## Step 81: Higher-Order Functions

```javascript
// ตัวอย่างที่ 56: Higher-order Functions คืออะไร
// Function ที่รับ function เป็น argument หรือ return function

// รับ function เป็น argument
function doTwice(fn, value) {
    return fn(fn(value));
}

const addOne = x => x + 1;
console.log(doTwice(addOne, 5)); // 7

// return function
function multiplier(factor) {
    return number => number * factor;
}

const double = multiplier(2);
const triple = multiplier(3);
console.log(double(5));  // 10
console.log(triple(4));  // 12

// ตัวอย่างที่ 57: Function Composition
const compose = (...fns) => x => fns.reduceRight((acc, fn) => fn(acc), x);
const pipe2 = (...fns) => x => fns.reduce((acc, fn) => fn(acc), x);

const trim = str => str.trim();
const toLowerCase = str => str.toLowerCase();
const addExclaim = str => str + '!';

const processName = pipe2(trim, toLowerCase, addExclaim);
console.log(processName('  HELLO  ')); // hello!
```

```javascript
// ตัวอย่างที่ 58: Currying
function curry(fn) {
    return function curried(...args) {
        if (args.length >= fn.length) {
            return fn.apply(this, args);
        }
        return function(...moreArgs) {
            return curried.apply(this, args.concat(moreArgs));
        };
    };
}

function add(a, b, c) {
    return a + b + c;
}

const curriedAdd = curry(add);
console.log(curriedAdd(1)(2)(3)); // 6
console.log(curriedAdd(1, 2)(3)); // 6
console.log(curriedAdd(1)(2, 3)); // 6
console.log(curriedAdd(1, 2, 3)); // 6

// Practical currying
const curriedGreet = curry((greeting, title, name) => `${greeting}, ${title} ${name}!`);
const sayHello = curriedGreet('สวัสดี');
const greetMr = sayHello('คุณ');
console.log(greetMr('สมชาย'));   // สวัสดี, คุณ สมชาย!
console.log(greetMr('สมหญิง'));  // สวัสดี, คุณ สมหญิง!
```

```javascript
// ตัวอย่างที่ 59: Memoization (caching function results)
function memoize(fn) {
    const cache = new Map();
    
    return function(...args) {
        const key = JSON.stringify(args);
        
        if (cache.has(key)) {
            console.log(`Cache hit: ${key}`);
            return cache.get(key);
        }
        
        console.log(`Computing: ${key}`);
        const result = fn.apply(this, args);
        cache.set(key, result);
        return result;
    };
}

const expensiveCalc = memoize(function(n) {
    // จำลองการคำนวณที่ใช้เวลานาน
    let result = 0;
    for (let i = 0; i <= n; i++) result += i;
    return result;
});

console.log(expensiveCalc(100));  // Computing: [100] -> 5050
console.log(expensiveCalc(100));  // Cache hit: [100] -> 5050
console.log(expensiveCalc(200));  // Computing: [200] -> 20100
console.log(expensiveCalc(100));  // Cache hit: [100] -> 5050
```

---

## Step 82: Closures

```javascript
// ตัวอย่างที่ 60: Closure พื้นฐาน
function makeCounter(initialValue = 0) {
    let count = initialValue; // private state
    
    return {
        increment() { return ++count; },
        decrement() { return --count; },
        reset() { count = initialValue; return count; },
        getValue() { return count; }
    };
}

const counter1 = makeCounter();
const counter2 = makeCounter(10);

counter1.increment(); // 1
counter1.increment(); // 2
counter1.increment(); // 3
counter2.increment(); // 11
counter2.increment(); // 12

console.log(counter1.getValue()); // 3
console.log(counter2.getValue()); // 12
counter1.reset();
console.log(counter1.getValue()); // 0
```

```javascript
// ตัวอย่างที่ 61: Closure สำหรับ Private Data
function createBankAccount(ownerName, initialBalance) {
    let balance = initialBalance;
    const transactionHistory = [];
    
    function logTransaction(type, amount, newBalance) {
        transactionHistory.push({
            type,
            amount,
            balance: newBalance,
            timestamp: new Date().toLocaleString('th-TH')
        });
    }
    
    return {
        deposit(amount) {
            if (amount <= 0) throw new Error('จำนวนต้องเป็นบวก');
            balance += amount;
            logTransaction('deposit', amount, balance);
            return this;
        },
        
        withdraw(amount) {
            if (amount <= 0) throw new Error('จำนวนต้องเป็นบวก');
            if (amount > balance) throw new Error('ยอดเงินไม่เพียงพอ');
            balance -= amount;
            logTransaction('withdraw', amount, balance);
            return this;
        },
        
        get balance() { return balance; },
        get owner() { return ownerName; },
        getHistory() { return [...transactionHistory]; }
    };
}

const account = createBankAccount('สมชาย', 1000);
account.deposit(500).deposit(200).withdraw(300);

console.log(`เจ้าของ: ${account.owner}`);
console.log(`ยอดคงเหลือ: ${account.balance} บาท`);
console.table(account.getHistory());
```

```javascript
// ตัวอย่างที่ 62: Partial Application
function partial(fn, ...presetArgs) {
    return function(...laterArgs) {
        return fn(...presetArgs, ...laterArgs);
    };
}

function formatCurrency(symbol, decimals, amount) {
    return `${symbol}${amount.toFixed(decimals)}`;
}

const formatTHB = partial(formatCurrency, '฿', 2);
const formatUSD = partial(formatCurrency, '$', 2);
const formatJPY = partial(formatCurrency, '¥', 0);

console.log(formatTHB(1299.9));  // ฿1299.90
console.log(formatUSD(49.99));   // $49.99
console.log(formatJPY(1500));    // ¥1500
```

---

## Step 83: Function Methods (call, apply, bind)

```javascript
// ตัวอย่างที่ 63: call - เรียก function พร้อมกำหนด this
function introduce(greeting, punctuation) {
    console.log(`${greeting} ฉันชื่อ ${this.name} อายุ ${this.age}${punctuation}`);
}

const person1 = { name: 'สมชาย', age: 25 };
const person2 = { name: 'สมหญิง', age: 22 };

introduce.call(person1, 'สวัสดี', '!');   // สวัสดี ฉันชื่อ สมชาย อายุ 25!
introduce.call(person2, 'Hello', '.');    // Hello ฉันชื่อ สมหญิง อายุ 22.

// ตัวอย่างที่ 64: apply - เหมือน call แต่ argument เป็น array
introduce.apply(person1, ['Hi', '?']); // Hi ฉันชื่อ สมชาย อายุ 25?

// use case: ส่ง array ไปให้ function ที่รับ individual args
const nums = [3, 1, 4, 1, 5, 9, 2, 6];
console.log(Math.max.apply(null, nums)); // 9 (เหมือน Math.max(...nums))
console.log(Math.min.apply(null, nums)); // 1

// ตัวอย่างที่ 65: bind - สร้าง function ใหม่ที่ this ถูก fixed
const boundIntroduce = introduce.bind(person1, 'สวัสดี');
boundIntroduce('!');  // สวัสดี ฉันชื่อ สมชาย อายุ 25!
boundIntroduce('?');  // สวัสดี ฉันชื่อ สมชาย อายุ 25?
```

```javascript
// ตัวอย่างที่ 66: bind สำหรับ event handlers
class Button {
    constructor(label) {
        this.label = label;
        this.clickCount = 0;
    }
    
    handleClick() {
        this.clickCount++;
        console.log(`${this.label} ถูกกด ${this.clickCount} ครั้ง`);
    }
    
    setupListener() {
        // bind this เพื่อให้ handleClick ใช้ this ของ Button instance
        const handler = this.handleClick.bind(this);
        
        // Simulate clicks
        handler();
        handler();
        handler();
    }
}

const btn = new Button('ส่งข้อมูล');
btn.setupListener();
// ส่งข้อมูล ถูกกด 1 ครั้ง
// ส่งข้อมูล ถูกกด 2 ครั้ง
// ส่งข้อมูล ถูกกด 3 ครั้ง
```

---

## Step 84: Generator Functions

```javascript
// ตัวอย่างที่ 67: Generator พื้นฐาน
function* simpleGenerator() {
    console.log('Step 1');
    yield 1;
    console.log('Step 2');
    yield 2;
    console.log('Step 3');
    yield 3;
    console.log('Done');
}

const gen = simpleGenerator();
console.log(gen.next()); // Step 1, { value: 1, done: false }
console.log(gen.next()); // Step 2, { value: 2, done: false }
console.log(gen.next()); // Step 3, { value: 3, done: false }
console.log(gen.next()); // Done, { value: undefined, done: true }

// ตัวอย่างที่ 68: Generator สำหรับ range
function* range(start, end, step = 1) {
    for (let i = start; i <= end; i += step) {
        yield i;
    }
}

for (const n of range(1, 10, 2)) {
    process.stdout.write(`${n} `);
}
console.log(); // 1 3 5 7 9

// ตัวอย่างที่ 69: Infinite Generator
function* infiniteCounter(start = 0) {
    let n = start;
    while (true) {
        yield n++;
    }
}

const counter = infiniteCounter(1);
// เรียกแค่ 5 ครั้ง
for (let i = 0; i < 5; i++) {
    process.stdout.write(`${counter.next().value} `);
}
console.log(); // 1 2 3 4 5
```

---

## Step 85-90: โปรแกรมตัวอย่างขั้นสูง

```javascript
// ตัวอย่างที่ 70: Event System ด้วย Functions
'use strict';

class EventEmitter {
    #events = new Map();
    
    on(event, listener) {
        if (!this.#events.has(event)) {
            this.#events.set(event, []);
        }
        this.#events.get(event).push(listener);
        return () => this.off(event, listener); // return unsubscribe function
    }
    
    once(event, listener) {
        const wrapper = (...args) => {
            listener(...args);
            this.off(event, wrapper);
        };
        return this.on(event, wrapper);
    }
    
    off(event, listener) {
        if (!this.#events.has(event)) return;
        const listeners = this.#events.get(event).filter(l => l !== listener);
        this.#events.set(event, listeners);
    }
    
    emit(event, ...args) {
        if (!this.#events.has(event)) return;
        this.#events.get(event).forEach(listener => listener(...args));
    }
    
    removeAllListeners(event) {
        if (event) {
            this.#events.delete(event);
        } else {
            this.#events.clear();
        }
    }
}

const emitter = new EventEmitter();

// Subscribe
const unsubLogin = emitter.on('login', (user) => {
    console.log(`${user.name} เข้าสู่ระบบ`);
});

emitter.on('login', (user) => {
    console.log(`บันทึก log: ${user.name} login ที่ ${new Date().toLocaleString('th-TH')}`);
});

emitter.once('firstVisit', (user) => {
    console.log(`ยินดีต้อนรับสู่ระบบ ${user.name}!`);
});

// Emit events
emitter.emit('login', { name: 'สมชาย', id: 1 });
emitter.emit('firstVisit', { name: 'สมชาย', id: 1 });
emitter.emit('firstVisit', { name: 'สมชาย', id: 1 }); // จะไม่แสดงแล้ว

// Unsubscribe
unsubLogin();
emitter.emit('login', { name: 'สมหญิง', id: 2 }); // แสดงแค่ log
```

```javascript
// ตัวอย่างที่ 71: State Machine ด้วย Functions
'use strict';

function createStateMachine(config) {
    let currentState = config.initial;
    const listeners = [];
    
    function transition(event) {
        const stateConfig = config.states[currentState];
        if (!stateConfig || !stateConfig.on || !stateConfig.on[event]) {
            console.warn(`ไม่สามารถ transition จาก ${currentState} ด้วย ${event}`);
            return false;
        }
        
        const nextState = stateConfig.on[event];
        const prevState = currentState;
        
        // Run exit action
        stateConfig.exit?.();
        
        currentState = nextState;
        
        // Run entry action
        config.states[currentState].entry?.();
        
        // Notify listeners
        listeners.forEach(l => l(prevState, currentState, event));
        
        return true;
    }
    
    return {
        send: transition,
        get state() { return currentState; },
        onChange(listener) {
            listeners.push(listener);
            return () => listeners.splice(listeners.indexOf(listener), 1);
        }
    };
}

const trafficLight = createStateMachine({
    initial: 'red',
    states: {
        red: {
            entry: () => console.log('🔴 หยุด!'),
            on: { CHANGE: 'green' }
        },
        green: {
            entry: () => console.log('🟢 ไป!'),
            on: { CHANGE: 'yellow' }
        },
        yellow: {
            entry: () => console.log('🟡 ระวัง!'),
            on: { CHANGE: 'red' }
        }
    }
});

trafficLight.onChange((from, to, event) => {
    console.log(`  State: ${from} -> ${to} (${event})`);
});

console.log('เริ่มต้น:');
trafficLight.send('CHANGE');
trafficLight.send('CHANGE');
trafficLight.send('CHANGE');
```

```javascript
// ตัวอย่างที่ 72: Observable / Reactive Programming Concept
'use strict';

function createObservable(subscribeFn) {
    return {
        subscribe(observer) {
            const safeObserver = {
                next: (value) => observer.next?.(value),
                error: (err) => observer.error?.(err),
                complete: () => observer.complete?.()
            };
            return subscribeFn(safeObserver);
        },
        
        pipe(...operators) {
            return operators.reduce((obs, op) => op(obs), this);
        }
    };
}

// Operators
function map(transform) {
    return (source) => createObservable((observer) => {
        return source.subscribe({
            next: (value) => observer.next(transform(value)),
            error: (err) => observer.error(err),
            complete: () => observer.complete()
        });
    });
}

function filter(predicate) {
    return (source) => createObservable((observer) => {
        return source.subscribe({
            next: (value) => predicate(value) && observer.next(value),
            error: (err) => observer.error(err),
            complete: () => observer.complete()
        });
    });
}

// สร้าง Observable จาก array
function from(arr) {
    return createObservable((observer) => {
        arr.forEach(item => observer.next(item));
        observer.complete();
    });
}

// ทดสอบ
const numbers$ = from([1, 2, 3, 4, 5, 6, 7, 8, 9, 10]);

numbers$.pipe(
    filter(n => n % 2 === 0),
    map(n => n * n)
).subscribe({
    next: (value) => process.stdout.write(`${value} `),
    complete: () => console.log('complete!')
});
// 4 16 36 64 100 complete!
```

```javascript
// ตัวอย่างที่ 73: Builder Pattern ด้วย Functions
'use strict';

function createQueryBuilder(tableName) {
    const state = {
        table: tableName,
        conditions: [],
        orderBy: [],
        selectedColumns: ['*'],
        limitValue: null,
        offsetValue: null
    };
    
    const builder = {
        select(...columns) {
            state.selectedColumns = columns;
            return builder;
        },
        
        where(condition) {
            state.conditions.push(condition);
            return builder;
        },
        
        orderByColumn(column, direction = 'ASC') {
            state.orderBy.push(`${column} ${direction}`);
            return builder;
        },
        
        limit(n) {
            state.limitValue = n;
            return builder;
        },
        
        offset(n) {
            state.offsetValue = n;
            return builder;
        },
        
        build() {
            let query = `SELECT ${state.selectedColumns.join(', ')} FROM ${state.table}`;
            
            if (state.conditions.length > 0) {
                query += ` WHERE ${state.conditions.join(' AND ')}`;
            }
            
            if (state.orderBy.length > 0) {
                query += ` ORDER BY ${state.orderBy.join(', ')}`;
            }
            
            if (state.limitValue !== null) {
                query += ` LIMIT ${state.limitValue}`;
            }
            
            if (state.offsetValue !== null) {
                query += ` OFFSET ${state.offsetValue}`;
            }
            
            return query;
        }
    };
    
    return builder;
}

const query = createQueryBuilder('users')
    .select('id', 'name', 'email', 'created_at')
    .where('active = 1')
    .where('age >= 18')
    .orderByColumn('created_at', 'DESC')
    .orderByColumn('name', 'ASC')
    .limit(10)
    .offset(20)
    .build();

console.log(query);
```

```javascript
// ตัวอย่างที่ 74: Async Function Patterns (แบบ synchronous simulation)
'use strict';

// Promise wrapper
function delay(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
}

function fetchUser(id) {
    return new Promise((resolve, reject) => {
        setTimeout(() => {
            if (id <= 0) reject(new Error('Invalid ID'));
            else resolve({ id, name: `User ${id}`, email: `user${id}@example.com` });
        }, 100);
    });
}

function fetchOrders(userId) {
    return new Promise(resolve => {
        setTimeout(() => {
            resolve([
                { id: 1, product: 'สินค้า A', amount: 299 },
                { id: 2, product: 'สินค้า B', amount: 499 }
            ]);
        }, 100);
    });
}

// Async/Await pattern
async function getUserWithOrders(userId) {
    try {
        const user = await fetchUser(userId);
        const orders = await fetchOrders(user.id);
        
        return {
            ...user,
            orders,
            totalSpent: orders.reduce((sum, o) => sum + o.amount, 0)
        };
    } catch (error) {
        console.error('เกิดข้อผิดพลาด:', error.message);
        return null;
    }
}

// เรียกใช้
getUserWithOrders(1).then(result => {
    if (result) {
        console.log(`ผู้ใช้: ${result.name}`);
        console.log(`ออเดอร์: ${result.orders.length} รายการ`);
        console.log(`ยอดรวม: ${result.totalSpent} บาท`);
    }
});
```

```javascript
// ตัวอย่างที่ 75: Function Composition Framework
'use strict';

// Compose utilities
const compose = (...fns) => x => fns.reduceRight((v, f) => f(v), x);
const pipe3 = (...fns) => x => fns.reduce((v, f) => f(v), x);

// Transformers
const trim = str => str.trim();
const lowercase = str => str.toLowerCase();
const capitalize = str => str.charAt(0).toUpperCase() + str.slice(1);
const removeSpecialChars = str => str.replace(/[^a-zA-Z0-9ก-๙\s]/g, '');
const compressSpaces = str => str.replace(/\s+/g, ' ');
const wrap = (prefix, suffix) => str => `${prefix}${str}${suffix}`;
const truncateTo = maxLen => str => str.length > maxLen ? str.slice(0, maxLen) + '...' : str;

// Create named pipelines
const sanitizeInput = pipe3(trim, removeSpecialChars, compressSpaces);
const formatName = pipe3(sanitizeInput, lowercase, capitalize);
const formatTitle = pipe3(
    sanitizeInput,
    compressSpaces,
    truncateTo(50),
    wrap('[', ']')
);

// ทดสอบ
console.log(sanitizeInput('  Hello,   World!!!  ')); // Hello   World
console.log(formatName('  jOHN   DOE  '));            // John doe
console.log(formatTitle('  JavaScript คือภาษาโปรแกรมมิ่งที่ดีที่สุดในโลก!!!  '));
```

```javascript
// ตัวอย่างที่ 76: Dependency Injection ด้วย Functions
'use strict';

// Services
function createLogger(prefix) {
    return {
        log: (msg) => console.log(`[${prefix}] ${msg}`),
        error: (msg) => console.error(`[${prefix}] ERROR: ${msg}`),
        warn: (msg) => console.warn(`[${prefix}] WARN: ${msg}`)
    };
}

function createCache() {
    const store = new Map();
    return {
        get: (key) => store.get(key),
        set: (key, value, ttl = 60000) => {
            store.set(key, { value, expiresAt: Date.now() + ttl });
        },
        has: (key) => {
            if (!store.has(key)) return false;
            const { expiresAt } = store.get(key);
            if (Date.now() > expiresAt) {
                store.delete(key);
                return false;
            }
            return true;
        },
        clear: () => store.clear()
    };
}

// Factory ที่ inject dependencies
function createUserService({ logger, cache, httpClient }) {
    return {
        async getUser(id) {
            const cacheKey = `user:${id}`;
            
            if (cache.has(cacheKey)) {
                logger.log(`Cache hit for user ${id}`);
                return cache.get(cacheKey).value;
            }
            
            logger.log(`Fetching user ${id} from API`);
            try {
                // const user = await httpClient.get(`/users/${id}`);
                const user = { id, name: `User ${id}` }; // mock
                cache.set(cacheKey, user);
                return user;
            } catch (error) {
                logger.error(`Failed to fetch user ${id}: ${error.message}`);
                throw error;
            }
        }
    };
}

// Assemble
const userService = createUserService({
    logger: createLogger('UserService'),
    cache: createCache(),
    httpClient: { get: async (url) => fetch(url) } // real httpClient
});

userService.getUser(1).then(user => console.log(user));
userService.getUser(1).then(user => console.log(user)); // from cache
```

```javascript
// ตัวอย่างที่ 77: Strategy Pattern ด้วย Functions
'use strict';

// Sorting strategies
const sortStrategies = {
    bubble(arr) {
        const result = [...arr];
        for (let i = 0; i < result.length - 1; i++) {
            for (let j = 0; j < result.length - i - 1; j++) {
                if (result[j] > result[j + 1]) {
                    [result[j], result[j + 1]] = [result[j + 1], result[j]];
                }
            }
        }
        return result;
    },
    
    quickSort(arr) {
        if (arr.length <= 1) return arr;
        const pivot = arr[Math.floor(arr.length / 2)];
        const left = arr.filter(x => x < pivot);
        const middle = arr.filter(x => x === pivot);
        const right = arr.filter(x => x > pivot);
        return [...this.quickSort(left), ...middle, ...this.quickSort(right)];
    },
    
    builtin(arr) {
        return [...arr].sort((a, b) => a - b);
    }
};

function createSorter(strategy) {
    return {
        sort(arr) { return strategy(arr); },
        change(newStrategy) { strategy = newStrategy; }
    };
}

const testArr = [64, 34, 25, 12, 22, 11, 90, 7, 45, 3];

console.log('Bubble:', sortStrategies.bubble(testArr));
console.log('Quick:', sortStrategies.quickSort(testArr));
console.log('Built-in:', sortStrategies.builtin(testArr));

// Benchmark
function benchmarkSort(name, fn, arr) {
    const start = Date.now();
    for (let i = 0; i < 100; i++) fn(arr);
    const time = Date.now() - start;
    console.log(`${name}: ${time}ms (100 iterations)`);
}

const largeArr = Array.from({ length: 1000 }, () => Math.floor(Math.random() * 10000));
benchmarkSort('Bubble', sortStrategies.bubble.bind(sortStrategies), largeArr);
benchmarkSort('Quick', sortStrategies.quickSort.bind(sortStrategies), largeArr);
benchmarkSort('Built-in', sortStrategies.builtin.bind(sortStrategies), largeArr);
```

```javascript
// ตัวอย่างที่ 78: Middleware Pattern
'use strict';

function createMiddlewareChain() {
    const middlewares = [];
    
    return {
        use(middleware) {
            middlewares.push(middleware);
            return this;
        },
        
        run(context) {
            let index = 0;
            
            function next(ctx) {
                if (index >= middlewares.length) return;
                const middleware = middlewares[index++];
                middleware(ctx, next);
            }
            
            next(context);
            return context;
        }
    };
}

// Middleware functions
function authMiddleware(ctx, next) {
    if (!ctx.user) {
        ctx.error = 'Unauthorized';
        return;
    }
    console.log(`Auth: ${ctx.user.name} authenticated`);
    next(ctx);
}

function loggingMiddleware(ctx, next) {
    console.log(`Request: ${ctx.method} ${ctx.path}`);
    const start = Date.now();
    next(ctx);
    console.log(`Response time: ${Date.now() - start}ms`);
}

function validationMiddleware(ctx, next) {
    if (!ctx.body) {
        ctx.error = 'No request body';
        return;
    }
    console.log('Validation: OK');
    next(ctx);
}

function handlerMiddleware(ctx, next) {
    ctx.response = { status: 200, data: `Processed: ${JSON.stringify(ctx.body)}` };
    console.log('Handler executed');
}

// สร้าง chain
const chain = createMiddlewareChain()
    .use(loggingMiddleware)
    .use(authMiddleware)
    .use(validationMiddleware)
    .use(handlerMiddleware);

// ทดสอบ
const ctx1 = {
    method: 'POST',
    path: '/api/data',
    user: { name: 'สมชาย' },
    body: { message: 'Hello' }
};

const result1 = chain.run(ctx1);
console.log('Result:', result1.response);

console.log('\n--- Without user ---');
const ctx2 = { method: 'GET', path: '/api/data', body: {} };
const result2 = chain.run(ctx2);
console.log('Error:', result2.error);
```

```javascript
// ตัวอย่างที่ 79: Functional Data Processing Pipeline
'use strict';

// Lazy evaluation pipeline
class LazyPipeline {
    #source;
    #operations = [];
    
    constructor(iterable) {
        this.#source = iterable;
    }
    
    filter(predicate) {
        this.#operations.push({ type: 'filter', fn: predicate });
        return this;
    }
    
    map(transform) {
        this.#operations.push({ type: 'map', fn: transform });
        return this;
    }
    
    take(n) {
        this.#operations.push({ type: 'take', count: n });
        return this;
    }
    
    *[Symbol.iterator]() {
        let count = Infinity;
        
        for (const op of this.#operations) {
            if (op.type === 'take') count = op.count;
        }
        
        let yielded = 0;
        outer: for (const item of this.#source) {
            let current = item;
            
            for (const op of this.#operations) {
                if (op.type === 'filter' && !op.fn(current)) continue outer;
                if (op.type === 'map') current = op.fn(current);
                if (op.type === 'take') { /* handled above */ }
            }
            
            yield current;
            yielded++;
            if (yielded >= count) break;
        }
    }
    
    toArray() { return [...this]; }
    
    reduce(fn, initial) {
        let acc = initial;
        for (const item of this) {
            acc = fn(acc, item);
        }
        return acc;
    }
}

// Generator สำหรับ infinite sequence
function* integers(start = 0) {
    while (true) yield start++;
}

// ใช้ LazyPipeline กับ infinite sequence
const result = new LazyPipeline(integers(1))
    .filter(n => n % 2 !== 0)    // เลขคี่
    .filter(n => n % 5 !== 0)    // ไม่ใช่ multiple ของ 5
    .map(n => n * n)              // ยกกำลังสอง
    .take(10)                     // เอาแค่ 10 ตัวแรก
    .toArray();

console.log(result);
// [1, 9, 49, 121, 169, 289, 361, 529, 625, 841]
```

```javascript
// ตัวอย่างที่ 80: Full Application Example - Mini Task Manager
'use strict';

const TaskManager = (() => {
    const PRIORITY = { LOW: 1, MEDIUM: 2, HIGH: 3, CRITICAL: 4 };
    const STATUS = { PENDING: 'pending', IN_PROGRESS: 'in_progress', DONE: 'done', CANCELLED: 'cancelled' };
    
    let tasks = [];
    let nextId = 1;
    const listeners = new Map();
    
    function emit(event, data) {
        listeners.get(event)?.forEach(l => l(data));
    }
    
    function on(event, listener) {
        if (!listeners.has(event)) listeners.set(event, []);
        listeners.get(event).push(listener);
        return () => {
            const arr = listeners.get(event);
            arr.splice(arr.indexOf(listener), 1);
        };
    }
    
    function create({ title, description = '', priority = PRIORITY.MEDIUM, tags = [], dueDate = null }) {
        if (!title?.trim()) throw new Error('Title is required');
        
        const task = {
            id: nextId++,
            title: title.trim(),
            description,
            priority,
            tags,
            dueDate,
            status: STATUS.PENDING,
            createdAt: new Date(),
            updatedAt: new Date()
        };
        
        tasks.push(task);
        emit('created', task);
        return task;
    }
    
    function update(id, changes) {
        const task = tasks.find(t => t.id === id);
        if (!task) throw new Error(`Task ${id} not found`);
        
        Object.assign(task, changes, { updatedAt: new Date() });
        emit('updated', task);
        return task;
    }
    
    function remove(id) {
        const index = tasks.findIndex(t => t.id === id);
        if (index === -1) throw new Error(`Task ${id} not found`);
        
        const [removed] = tasks.splice(index, 1);
        emit('removed', removed);
        return removed;
    }
    
    function query({ status, priority, tag, overdue } = {}) {
        return tasks.filter(task => {
            if (status && task.status !== status) return false;
            if (priority && task.priority !== priority) return false;
            if (tag && !task.tags.includes(tag)) return false;
            if (overdue && task.dueDate && new Date(task.dueDate) < new Date() && task.status !== STATUS.DONE) return false;
            return true;
        });
    }
    
    function getStats() {
        const total = tasks.length;
        const byStatus = Object.fromEntries(
            Object.entries(STATUS).map(([key, value]) => [
                value,
                tasks.filter(t => t.status === value).length
            ])
        );
        const byPriority = Object.fromEntries(
            Object.entries(PRIORITY).map(([key, value]) => [
                key,
                tasks.filter(t => t.priority === value).length
            ])
        );
        
        return { total, byStatus, byPriority };
    }
    
    return { PRIORITY, STATUS, create, update, remove, query, getStats, on };
})();

// ทดสอบ
TaskManager.on('created', task => console.log(`✓ สร้าง: ${task.title}`));
TaskManager.on('updated', task => console.log(`✎ อัพเดท: ${task.title} -> ${task.status}`));

const t1 = TaskManager.create({
    title: 'เขียน unit tests',
    priority: TaskManager.PRIORITY.HIGH,
    tags: ['dev', 'testing']
});

const t2 = TaskManager.create({
    title: 'Review PR',
    priority: TaskManager.PRIORITY.MEDIUM,
    tags: ['dev']
});

const t3 = TaskManager.create({
    title: 'Update documentation',
    priority: TaskManager.PRIORITY.LOW,
    tags: ['docs']
});

TaskManager.update(t1.id, { status: TaskManager.STATUS.IN_PROGRESS });
TaskManager.update(t2.id, { status: TaskManager.STATUS.DONE });

console.log('\n=== สถิติ ===');
const stats = TaskManager.getStats();
console.log('ทั้งหมด:', stats.total);
console.log('สถานะ:', stats.byStatus);
console.log('Priority:', stats.byPriority);

console.log('\n=== งานที่ยังไม่เสร็จ ===');
const pending = TaskManager.query({ status: TaskManager.STATUS.PENDING });
pending.concat(TaskManager.query({ status: TaskManager.STATUS.IN_PROGRESS }))
    .forEach(t => console.log(`- [${Object.keys(TaskManager.PRIORITY).find(k => TaskManager.PRIORITY[k] === t.priority)}] ${t.title}`));
```

---

## สรุป Step 71-90

ในส่วนนี้คุณได้เรียนรู้:
- **Function Declaration vs Expression**: syntax, hoisting, use cases
- **Parameters & Arguments**: passing values, arguments object
- **Default Parameters**: ES6, expressions, destructuring
- **Rest Parameters**: ...args, difference from arguments
- **Return Values**: types, patterns, early return
- **Scope**: global, function, block, scope chain, shadowing
- **Hoisting**: var, function declaration, TDZ
- **IIFE**: isolation, module pattern
- **Recursion**: base case, memoization, tree operations
- **Arrow Functions**: syntax, this binding, limitations
- **Higher-order Functions**: compose, curry, memoize
- **Closures**: private data, factory functions
- **call, apply, bind**: this manipulation
- **Generators**: yield, infinite sequences

---

## แบบฝึกหัด

### ระดับพื้นฐาน

**แบบฝึกหัดที่ 1**: Function Types
```javascript
// สร้างฟังก์ชันคำนวณ BMI ในสองรูปแบบ:
// 1. Function Declaration
// 2. Arrow Function
// BMI = weight(kg) / height(m)^2
// โดย: <18.5 = Underweight, 18.5-24.9 = Normal, 25-29.9 = Overweight, >=30 = Obese
```

**แบบฝึกหัดที่ 2**: Closures
```javascript
// สร้าง function makeAdder(n) ที่ return function ที่บวก n ให้กับ argument
// ตัวอย่าง:
const add5 = makeAdder(5);
const add10 = makeAdder(10);
// console.log(add5(3));   // 8
// console.log(add10(3));  // 13
// console.log(add5(add10(1))); // 16
```

### ระดับกลาง

**แบบฝึกหัดที่ 3**: Higher-order Functions
```javascript
// สร้าง function retry(fn, maxAttempts, delay) ที่:
// - ลอง call fn()
// - ถ้า throw error ลองใหม่จนถึง maxAttempts ครั้ง
// - ถ้ายังไม่ได้ throw error สุดท้าย
// - ถ้าสำเร็จ return ค่า
```

**แบบฝึกหัดที่ 4**: Curry
```javascript
// สร้าง curry function ที่ใช้กับฟังก์ชันนี้:
function formatDate(year, month, day) {
    return `${year}-${String(month).padStart(2,'0')}-${String(day).padStart(2,'0')}`;
}
// ให้ทำให้สามารถเรียกได้ทั้ง:
// formatDate(2024, 1, 15)
// formatDate(2024)(1)(15)
// formatDate(2024, 1)(15)
```

### ระดับสูง

**แบบฝึกหัดที่ 5**: Build a mini-Redux
```javascript
// สร้าง createStore(reducer, initialState) ที่มี:
// - getState() - ดูค่าปัจจุบัน
// - dispatch(action) - ส่ง action เพื่อเปลี่ยน state
// - subscribe(listener) - รับแจ้งเมื่อ state เปลี่ยน
// - unsubscribe - return จาก subscribe

// Reducer ตัวอย่าง:
function counterReducer(state = 0, action) {
    switch (action.type) {
        case 'INCREMENT': return state + (action.amount || 1);
        case 'DECREMENT': return state - (action.amount || 1);
        case 'RESET': return 0;
        default: return state;
    }
}
```

---

## เฉลยแบบฝึกหัด

**เฉลยที่ 5**: Mini-Redux
```javascript
'use strict';

function createStore(reducer, initialState) {
    let state = initialState ?? reducer(undefined, { type: '@@INIT' });
    const listeners = [];
    
    return {
        getState() { return state; },
        
        dispatch(action) {
            state = reducer(state, action);
            listeners.forEach(l => l(state));
            return action;
        },
        
        subscribe(listener) {
            listeners.push(listener);
            return function unsubscribe() {
                const index = listeners.indexOf(listener);
                listeners.splice(index, 1);
            };
        }
    };
}

const store = createStore(counterReducer);

const unsubscribe = store.subscribe(state => {
    console.log(`State changed: ${state}`);
});

store.dispatch({ type: 'INCREMENT' });         // 1
store.dispatch({ type: 'INCREMENT', amount: 5 }); // 6
store.dispatch({ type: 'DECREMENT', amount: 2 }); // 4
console.log('Current:', store.getState());     // 4
unsubscribe();
store.dispatch({ type: 'RESET' });            // ไม่แสดง (unsubscribed)
console.log('After reset:', store.getState()); // 0
```

---

## แหล่งข้อมูลเพิ่มเติม

- **MDN - Functions**: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Functions
- **You Don't Know JS**: https://github.com/getify/You-Dont-Know-JS
- **JavaScript.info - Functions**: https://javascript.info/function-basics
- **Eloquent JavaScript**: https://eloquentjavascript.net/

---

*ยินดีด้วย! คุณเรียนจบ 5 บทแรกของ JavaScript Course แล้ว*

*หัวข้อถัดไปที่ควรเรียนต่อ:*
- *Part 6: Arrays & Array Methods*
- *Part 7: Objects & Prototypes*
- *Part 8: Classes & OOP*
- *Part 9: Promises & Async/Await*
- *Part 10: DOM Manipulation*
