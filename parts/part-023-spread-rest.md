# Part 23: Spread และ Rest Operators (Steps 431-450)

## บทนำ

Spread (`...`) และ Rest (`...`) operators ใช้ syntax เดียวกันคือ `...` แต่ทำงานตรงข้ามกัน:
- **Rest** = รวบรวมค่าหลายๆ ค่าเป็น array เดียว (ใช้ใน parameters และ destructuring)
- **Spread** = กระจาย iterable ออกเป็นธาตุแยกกัน (ใช้ใน function calls และ literals)

---

## Step 431: Rest Parameter Syntax

```javascript
// Rest parameter ต้องอยู่ตำแหน่งสุดท้าย
function sum(...numbers) {
  return numbers.reduce((total, n) => total + n, 0);
}

console.log(sum(1, 2, 3));          // 6
console.log(sum(1, 2, 3, 4, 5));    // 15
console.log(sum());                  // 0

// Rest กับ required parameters
function logInfo(level, message, ...details) {
  console.log(`[${level}] ${message}`);
  if (details.length > 0) {
    console.log('Details:', ...details);
  }
}

logInfo('ERROR', 'Connection failed');
// [ERROR] Connection failed

logInfo('INFO', 'User logged in', 'userId: 123', 'ip: 192.168.1.1');
// [INFO] User logged in
// Details: userId: 123 ip: 192.168.1.1
```

```javascript
// Rest ต้องอยู่ท้ายสุดเสมอ
// function invalid(a, ...rest, b) {} // SyntaxError!

// Rest ใน arrow functions
const multiply = (factor, ...nums) => nums.map(n => n * factor);
console.log(multiply(2, 1, 2, 3, 4)); // [2, 4, 6, 8]

// Rest เพื่อ accept unlimited arguments
const makeTag = (tagName, ...children) => ({
  type: tagName,
  children: children.flat(),
});

const div = makeTag('div', 'Hello', ' ', 'World');
console.log(div); // { type: 'div', children: ['Hello', ' ', 'World'] }
```

---

## Step 432: Rest vs arguments Object

```javascript
// arguments object - เฉพาะ regular functions
function oldStyle() {
  console.log(typeof arguments); // 'object'
  console.log(Array.isArray(arguments)); // false! (array-like, not array)
  
  // ต้องแปลงเป็น array ก่อน
  const args = Array.from(arguments);
  // หรือ
  const args2 = Array.prototype.slice.call(arguments);
  
  return args.reduce((sum, n) => sum + n, 0);
}

// Rest parameter - เป็น real Array
function newStyle(...numbers) {
  console.log(Array.isArray(numbers)); // true
  return numbers.reduce((sum, n) => sum + n, 0);
}

console.log(oldStyle(1, 2, 3)); // 6
console.log(newStyle(1, 2, 3)); // 6
```

```javascript
// ความแตกต่างสำคัญ

// 1. arguments มีค่า arguments.length ที่ถูกต้องเสมอ
// Rest ก็มี length ที่ถูกต้อง

// 2. arguments ไม่มีใน arrow functions
const arrowArgs = (...args) => args; // ต้องใช้ rest
// const arrowArgs2 = () => arguments; // ReferenceError ใน strict mode

// 3. arguments มี callee property (deprecated)
function withCallee() {
  // console.log(arguments.callee); // deprecated, error ใน strict mode
}

// 4. Rest parameters ไม่นับ default parameters
function test(a, b = 10, ...rest) {
  console.log(rest); // ไม่รวม b
}
test(1, 2, 3, 4); // [3, 4]

// 5. arguments นับทุก argument ที่ส่งมา
function testArgs(a, b = 10) {
  console.log(arguments.length); // นับตาม argument ที่ส่งมาจริง
}
testArgs(1); // 1
testArgs(1, 2, 3); // 3
```

---

## Step 433: Rest กับ Destructuring

```javascript
// Rest ใน array destructuring
const [first, second, ...remaining] = [1, 2, 3, 4, 5];
console.log(first);     // 1
console.log(second);    // 2
console.log(remaining); // [3, 4, 5]

// Rest ใน object destructuring
const { name, age, ...otherProps } = {
  name: 'สมชาย',
  age: 25,
  city: 'กรุงเทพ',
  hobby: 'coding',
};
console.log(name, age);  // 'สมชาย' 25
console.log(otherProps); // { city: 'กรุงเทพ', hobby: 'coding' }

// ใช้ในการ filter properties
function sanitizeUser(user) {
  const { password, token, refreshToken, ...safeData } = user;
  return safeData;
}

const user = {
  id: 1,
  name: 'สมชาย',
  email: 'test@test.com',
  password: 'secret123',
  token: 'jwt-token',
  refreshToken: 'refresh-token',
};

console.log(sanitizeUser(user));
// { id: 1, name: 'สมชาย', email: 'test@test.com' }
```

```javascript
// Rest ซ้อน destructuring
const {
  a,
  b: { c, ...bRest },
  ...mainRest
} = { a: 1, b: { c: 2, d: 3, e: 4 }, f: 5, g: 6 };

console.log(a);       // 1
console.log(c);       // 2
console.log(bRest);   // { d: 3, e: 4 }
console.log(mainRest); // { f: 5, g: 6 }
```

---

## Step 434: Spread Operator กับ Arrays - พื้นฐาน

```javascript
// กระจาย array เป็น arguments
function sum(a, b, c) {
  return a + b + c;
}

const nums = [1, 2, 3];
console.log(sum(...nums)); // 6
// เท่ากับ sum(1, 2, 3)

// กระจายบางส่วน
console.log(sum(1, ...nums.slice(1))); // 6

// Math functions
const numbers = [3, 1, 4, 1, 5, 9, 2, 6];
console.log(Math.max(...numbers)); // 9
console.log(Math.min(...numbers)); // 1

// Array.push กับ spread
const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];
arr1.push(...arr2);
console.log(arr1); // [1, 2, 3, 4, 5, 6]
```

---

## Step 435: Spread เพื่อ Copy Arrays

```javascript
// Shallow copy ด้วย spread
const original = [1, 2, 3, 4, 5];
const copy = [...original];

copy.push(6);
console.log(original); // [1, 2, 3, 4, 5] (ไม่เปลี่ยน)
console.log(copy);     // [1, 2, 3, 4, 5, 6]

// เปรียบเทียบกับแบบเก่า
const copyOld = original.slice();
const copyOld2 = Array.from(original);
const copyOld3 = [].concat(original);

// ใช้ spread ใน immutable operations
const todos = [
  { id: 1, text: 'เรียน JS', done: false },
  { id: 2, text: 'ทำโปรเจค', done: false },
];

// เพิ่ม todo ใหม่ (immutable)
const addTodo = (todos, newTodo) => [...todos, newTodo];

// ลบ todo (immutable)
const removeTodo = (todos, id) => todos.filter(t => t.id !== id);

// อัพเดท todo (immutable)
const toggleTodo = (todos, id) =>
  todos.map(t => t.id === id ? { ...t, done: !t.done } : t);

const updated = addTodo(todos, { id: 3, text: 'Deploy', done: false });
console.log(updated.length); // 3
console.log(todos.length);   // 2 (original unchanged)
```

---

## Step 436: Spread เพื่อ Merge Arrays

```javascript
// รวม arrays
const fruits = ['แอปเปิล', 'กล้วย'];
const veggies = ['แครอท', 'บรอกโคลี'];
const allFood = [...fruits, ...veggies];
console.log(allFood); // ['แอปเปิล', 'กล้วย', 'แครอท', 'บรอกโคลี']

// แทรกระหว่าง arrays
const numbers = [1, 2, 3, 4, 5];
const withInsert = [...numbers.slice(0, 2), 99, ...numbers.slice(2)];
console.log(withInsert); // [1, 2, 99, 3, 4, 5]

// รวมหลาย arrays
const mergeAll = (...arrays) => [].concat(...arrays);
// หรือ
const mergeAll2 = (...arrays) => arrays.flat();
// หรือ
const mergeAll3 = (...arrays) => arrays.reduce((acc, arr) => [...acc, ...arr], []);

console.log(mergeAll([1, 2], [3, 4], [5, 6])); // [1, 2, 3, 4, 5, 6]

// Unique values ด้วย Set + spread
const withDuplicates = [1, 2, 2, 3, 3, 3, 4];
const unique = [...new Set(withDuplicates)];
console.log(unique); // [1, 2, 3, 4]
```

```javascript
// ใช้ในการสร้าง immutable arrays
const initialState = {
  items: ['A', 'B', 'C'],
};

// เพิ่ม item ที่ index ที่ต้องการ (immutable)
function insertAt(arr, index, item) {
  return [...arr.slice(0, index), item, ...arr.slice(index)];
}

// ลบ item ที่ index (immutable)
function removeAt(arr, index) {
  return [...arr.slice(0, index), ...arr.slice(index + 1)];
}

// แทนที่ item ที่ index (immutable)
function replaceAt(arr, index, newItem) {
  return [...arr.slice(0, index), newItem, ...arr.slice(index + 1)];
}

console.log(insertAt(initialState.items, 1, 'X')); // ['A', 'X', 'B', 'C']
console.log(removeAt(initialState.items, 1));      // ['A', 'C']
console.log(replaceAt(initialState.items, 1, 'Z')); // ['A', 'Z', 'C']
```

---

## Step 437: Spread เพื่อส่ง Array เป็น Function Arguments

```javascript
// แบบเก่า: Function.prototype.apply
const args = [1, 2, 3];
const old = Math.max.apply(null, args);

// แบบใหม่: spread
const modern = Math.max(...args);
console.log(modern); // 3

// ส่งหลาย arrays เป็น arguments
function createDate(year, month, day) {
  return new Date(year, month - 1, day);
}

const dateArray = [2024, 1, 15];
const date = createDate(...dateArray);
console.log(date); // Mon Jan 15 2024

// Spread ร่วมกับ arguments อื่นๆ
function format(prefix, ...messages) {
  return messages.map(m => `${prefix}: ${m}`);
}

const msgs = ['Hello', 'World', 'Test'];
console.log(format('INFO', ...msgs));
// ['INFO: Hello', 'INFO: World', 'INFO: Test']
```

```javascript
// ตัวอย่างจริง: console.log formatting
function log(level, ...args) {
  const prefix = {
    info: '[INFO]',
    warn: '[WARN]',
    error: '[ERROR]',
  }[level] || '[LOG]';
  
  console.log(prefix, ...args);
}

log('info', 'Server started on port', 3000);
// [INFO] Server started on port 3000

log('error', 'Failed to connect:', new Error('Connection refused'));
// [ERROR] Failed to connect: Error: Connection refused

// ใช้ spread กับ Math methods
const data = [5, 2, 8, 1, 9, 3, 7, 4, 6];
console.log(Math.max(...data)); // 9
console.log(Math.min(...data)); // 1

// หาจำนวนระหว่าง min และ max ที่ใหญ่ที่สุด
function range(min, max) {
  return max - min;
}
console.log(range(Math.min(...data), Math.max(...data))); // 8
```

---

## Step 438: Spread Operator กับ Objects

```javascript
// Copy object ด้วย spread
const original = { name: 'สมชาย', age: 25 };
const copy = { ...original };

copy.name = 'สมหญิง'; // ไม่กระทบ original
console.log(original.name); // 'สมชาย'
console.log(copy.name);     // 'สมหญิง'

// เพิ่ม properties
const withExtra = { ...original, city: 'กรุงเทพ' };
console.log(withExtra); // { name: 'สมชาย', age: 25, city: 'กรุงเทพ' }

// override properties
const updated = { ...original, age: 26 };
console.log(updated); // { name: 'สมชาย', age: 26 }

// ลำดับสำคัญ! property ที่มาทีหลังจะ override
const obj1 = { a: 1, b: 2 };
const obj2 = { b: 3, c: 4 };

console.log({ ...obj1, ...obj2 }); // { a: 1, b: 3, c: 4 } - b ถูก override
console.log({ ...obj2, ...obj1 }); // { b: 2, c: 4, a: 1 } - a ถูก override
```

---

## Step 439: Spread เพื่อ Copy Objects (Shallow vs Deep)

```javascript
// SHALLOW copy - เพียงชั้นเดียว
const original = {
  name: 'สมชาย',
  address: {
    city: 'กรุงเทพ',
    street: 'สุขุมวิท',
  },
  hobbies: ['coding', 'reading'],
};

const shallowCopy = { ...original };

// Primitive ไม่กระทบ
shallowCopy.name = 'สมหญิง';
console.log(original.name);     // 'สมชาย' (ไม่เปลี่ยน)
console.log(shallowCopy.name);  // 'สมหญิง'

// Reference types กระทบ!
shallowCopy.address.city = 'เชียงใหม่';
console.log(original.address.city);    // 'เชียงใหม่' (เปลี่ยนด้วย!)
console.log(shallowCopy.address.city); // 'เชียงใหม่'

shallowCopy.hobbies.push('gaming');
console.log(original.hobbies); // ['coding', 'reading', 'gaming'] (เปลี่ยนด้วย!)
```

```javascript
// DEEP copy - ทุกชั้น
// วิธีที่ 1: JSON (ง่ายแต่มีข้อจำกัด)
const deepCopy1 = JSON.parse(JSON.stringify(original));
// ข้อจำกัด: ไม่รองรับ undefined, function, Date, Map, Set, circular refs

// วิธีที่ 2: structuredClone (ES2022)
const deepCopy2 = structuredClone(original);

// วิธีที่ 3: recursive function
function deepClone(obj) {
  if (obj === null || typeof obj !== 'object') return obj;
  if (obj instanceof Date) return new Date(obj);
  if (obj instanceof Array) return obj.map(item => deepClone(item));
  
  return Object.fromEntries(
    Object.entries(obj).map(([key, value]) => [key, deepClone(value)])
  );
}

const deepCopy3 = deepClone(original);
deepCopy3.address.city = 'ภูเก็ต';
console.log(original.address.city); // 'กรุงเทพ' (ไม่เปลี่ยน - deep copy!)
```

---

## Step 440: Spread เพื่อ Merge Objects

```javascript
// รวม objects
const defaults = {
  theme: 'light',
  language: 'th',
  fontSize: 14,
  notifications: true,
};

const userPreferences = {
  theme: 'dark',
  fontSize: 16,
};

const finalConfig = { ...defaults, ...userPreferences };
console.log(finalConfig);
// { theme: 'dark', language: 'th', fontSize: 16, notifications: true }

// ลำดับสำคัญ: userPreferences override defaults

// Deep merge (อย่างง่าย)
function shallowMerge(...objects) {
  return Object.assign({}, ...objects);
}

function deepMerge(target, ...sources) {
  if (!sources.length) return target;
  const source = sources.shift();
  
  for (const key in source) {
    if (typeof source[key] === 'object' && !Array.isArray(source[key])) {
      if (!target[key]) target[key] = {};
      deepMerge(target[key], source[key]);
    } else {
      target[key] = source[key];
    }
  }
  
  return deepMerge(target, ...sources);
}

const base = { a: 1, nested: { x: 1, y: 2 } };
const override = { b: 2, nested: { y: 99, z: 3 } };

console.log({ ...base, ...override });
// { a: 1, nested: { y: 99, z: 3 }, b: 2 } - nested ถูกแทนที่ทั้ง object!

console.log(deepMerge({}, base, override));
// { a: 1, nested: { x: 1, y: 99, z: 3 }, b: 2 } - merge อย่างถูกต้อง
```

```javascript
// Conditional spreading
function buildQuery(base, options = {}) {
  return {
    ...base,
    ...(options.includeDeleted && { deletedAt: { $ne: null } }),
    ...(options.page && { skip: (options.page - 1) * 10, limit: 10 }),
    ...(options.sort && { sort: options.sort }),
  };
}

console.log(buildQuery({ type: 'user' }));
// { type: 'user' }

console.log(buildQuery({ type: 'user' }, { page: 2, sort: 'name' }));
// { type: 'user', skip: 10, limit: 10, sort: 'name' }

console.log(buildQuery({ type: 'user' }, { includeDeleted: true, page: 1 }));
// { type: 'user', deletedAt: { $ne: null }, skip: 0, limit: 10 }
```

---

## Step 441: Spread กับ Strings

```javascript
// String เป็น iterable จึง spread ได้
const str = 'สวัสดี';
const chars = [...str];
console.log(chars); // ['ส', 'ว', 'า', 'ส', 'ด', 'ี']

// แปลง string เป็น array ของ characters
// ดีกว่า split('') สำหรับ Unicode
const emoji = '👋🌍';
const emojiChars = [...emoji];
console.log(emojiChars); // ['👋', '🌍']
// split('') จะให้ผลผิดกับ emoji

// นับ characters ที่ถูกต้อง
console.log('👋🌍'.length);        // 4 (ผิด - byte count)
console.log([...'👋🌍'].length);   // 2 (ถูกต้อง)

// Reverse string ที่มี Unicode
function reverseString(str) {
  return [...str].reverse().join('');
}

console.log(reverseString('Hello')); // 'olleH'
console.log(reverseString('สวัสดี')); // กลับข้อความภาษาไทย

// Unique characters
const uniqueChars = [...new Set('aabbccdd')];
console.log(uniqueChars); // ['a', 'b', 'c', 'd']
console.log(uniqueChars.join('')); // 'abcd'
```

---

## Step 442: Spread กับ Map และ Set

```javascript
// Set operations ด้วย spread
const setA = new Set([1, 2, 3, 4, 5]);
const setB = new Set([3, 4, 5, 6, 7]);

// Union
const union = new Set([...setA, ...setB]);
console.log([...union]); // [1, 2, 3, 4, 5, 6, 7]

// Intersection
const intersection = new Set([...setA].filter(x => setB.has(x)));
console.log([...intersection]); // [3, 4, 5]

// Difference
const difference = new Set([...setA].filter(x => !setB.has(x)));
console.log([...difference]); // [1, 2]

// Map ด้วย spread
const map = new Map([['a', 1], ['b', 2], ['c', 3]]);
const mapArray = [...map];
console.log(mapArray); // [['a', 1], ['b', 2], ['c', 3]]

// Copy Map
const mapCopy = new Map([...map]);
mapCopy.set('d', 4);
console.log(map.size);     // 3 (ไม่เปลี่ยน)
console.log(mapCopy.size); // 4
```

```javascript
// แปลง Map เป็น Object
const map2 = new Map([['name', 'สมชาย'], ['age', 25]]);
const obj = Object.fromEntries(map2);
// หรือ
const obj2 = { ...Object.fromEntries(map2) };
console.log(obj); // { name: 'สมชาย', age: 25 }

// แปลง Object เป็น Map
const person = { name: 'สมหญิง', age: 30 };
const personMap = new Map(Object.entries(person));
console.log(personMap.get('name')); // 'สมหญิง'

// Merge Maps
const map3 = new Map([['a', 1], ['b', 2]]);
const map4 = new Map([['b', 99], ['c', 3]]);
const merged = new Map([...map3, ...map4]);
console.log([...merged]); // [['a', 1], ['b', 99], ['c', 3]]
// b ถูก override ด้วยค่าจาก map4
```

---

## Step 443: Shallow Copy vs Deep Copy - เจาะลึก

```javascript
// เมื่อไหร่ shallow copy พอ
// - เมื่อ object มีเฉพาะ primitive values
const primitive = { name: 'สมชาย', age: 25, active: true };
const shallowOk = { ...primitive };
shallowOk.name = 'changed'; // ปลอดภัย - primitive ไม่กระทบ original

// เมื่อไหร่ต้องการ deep copy
// - เมื่อ object มี nested objects หรือ arrays
const withNested = {
  name: 'สมชาย',
  settings: { theme: 'dark', language: 'th' },
  items: [1, 2, 3],
};

const shallowBad = { ...withNested };
shallowBad.settings.theme = 'light'; // กระทบ original!
shallowBad.items.push(4);            // กระทบ original!

// วิธีแก้: deep copy nested
const properCopy = {
  ...withNested,
  settings: { ...withNested.settings }, // copy nested object
  items: [...withNested.items],          // copy array
};

properCopy.settings.theme = 'light'; // ไม่กระทบ original
properCopy.items.push(4);            // ไม่กระทบ original

console.log(withNested.settings.theme); // 'dark'
console.log(withNested.items.length);   // 3
```

```javascript
// ตัวอย่างจริง: Redux reducer patterns
// ถูกต้อง: สร้าง object ใหม่ทุกครั้ง
function todoReducer(state = [], action) {
  switch (action.type) {
    case 'ADD_TODO':
      return [...state, action.payload]; // ✓ new array
    
    case 'TOGGLE_TODO':
      return state.map(todo =>
        todo.id === action.id
          ? { ...todo, completed: !todo.completed } // ✓ new object
          : todo
      );
    
    case 'UPDATE_NESTED':
      return state.map(todo =>
        todo.id === action.id
          ? {
              ...todo,
              metadata: {
                ...todo.metadata, // ✓ copy nested
                updatedAt: Date.now(),
              }
            }
          : todo
      );
    
    default:
      return state;
  }
}
```

---

## Step 444: Practical Patterns - Function Composition

```javascript
// Currying ด้วย rest/spread
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) {
      return fn(...args);
    }
    return function(...moreArgs) {
      return curried(...args, ...moreArgs);
    };
  };
}

const add = curry((a, b, c) => a + b + c);
console.log(add(1)(2)(3));    // 6
console.log(add(1, 2)(3));    // 6
console.log(add(1)(2, 3));    // 6
console.log(add(1, 2, 3));    // 6

// Partial application
function partial(fn, ...preArgs) {
  return function(...laterArgs) {
    return fn(...preArgs, ...laterArgs);
  };
}

const multiply = (a, b) => a * b;
const double = partial(multiply, 2);
const triple = partial(multiply, 3);

console.log(double(5));  // 10
console.log(triple(5));  // 15
```

```javascript
// Function composition
const compose = (...fns) => x => fns.reduceRight((acc, fn) => fn(acc), x);
const pipe = (...fns) => x => fns.reduce((acc, fn) => fn(acc), x);

const double = x => x * 2;
const addTen = x => x + 10;
const square = x => x * x;

const transform = compose(square, addTen, double);
// = square(addTen(double(x)))
console.log(transform(5)); // square(addTen(10)) = square(20) = 400

const transform2 = pipe(double, addTen, square);
// = square(addTen(double(x)))
console.log(transform2(5)); // 400
```

---

## Step 445: Practical Patterns - Object Utilities

```javascript
// merge function
function merge(...objects) {
  return objects.reduce((acc, obj) => ({ ...acc, ...obj }), {});
}

const base = { a: 1, b: 2 };
const extra = { c: 3, d: 4 };
const override = { b: 99 };
console.log(merge(base, extra, override));
// { a: 1, b: 99, c: 3, d: 4 }

// defaults function (ไม่ override existing)
function defaults(obj, ...sources) {
  return sources.reduce((acc, source) => ({
    ...source,
    ...acc, // acc มาทีหลัง = ไม่ถูก override
  }), obj);
}

console.log(defaults({ a: 1 }, { a: 99, b: 2 }, { c: 3 }));
// { a: 1, b: 2, c: 3 } - a ไม่ถูก override

// pick และ omit ด้วย spread
const pick = (obj, keys) =>
  keys.reduce((acc, key) => key in obj ? { ...acc, [key]: obj[key] } : acc, {});

const omit = (obj, keys) => {
  const { ...copy } = obj;
  keys.forEach(key => delete copy[key]);
  return copy;
};

const user = { id: 1, name: 'สมชาย', password: 'secret', email: 'test@test.com' };
console.log(pick(user, ['id', 'name', 'email']));
// { id: 1, name: 'สมชาย', email: 'test@test.com' }

console.log(omit(user, ['password']));
// { id: 1, name: 'สมชาย', email: 'test@test.com' }
```

---

## Step 446: Practical Patterns - Array Utilities

```javascript
// flatten ด้วย spread และ recursion
function flatten(arr, depth = Infinity) {
  if (depth === 0) return [...arr];
  return arr.reduce((acc, item) => {
    if (Array.isArray(item)) {
      return [...acc, ...flatten(item, depth - 1)];
    }
    return [...acc, item];
  }, []);
}

const nested = [1, [2, [3, [4, [5]]]]];
console.log(flatten(nested, 1)); // [1, 2, [3, [4, [5]]]]
console.log(flatten(nested, 2)); // [1, 2, 3, [4, [5]]]
console.log(flatten(nested));    // [1, 2, 3, 4, 5]

// chunk array
function chunk(arr, size) {
  const result = [];
  for (let i = 0; i < arr.length; i += size) {
    result.push(arr.slice(i, i + size));
  }
  return result;
}

const arr = [1, 2, 3, 4, 5, 6, 7, 8, 9];
console.log(chunk(arr, 3)); // [[1,2,3], [4,5,6], [7,8,9]]

// zip arrays
function zip(...arrays) {
  const maxLength = Math.max(...arrays.map(a => a.length));
  return Array.from({ length: maxLength }, (_, i) =>
    arrays.map(arr => arr[i])
  );
}

console.log(zip([1, 2, 3], ['a', 'b', 'c']));
// [[1, 'a'], [2, 'b'], [3, 'c']]

// unzip
function unzip(zipped) {
  return zipped[0].map((_, i) => zipped.map(row => row[i]));
}

const zipped = [[1, 'a'], [2, 'b'], [3, 'c']];
const [numbers, letters] = unzip(zipped);
console.log(numbers); // [1, 2, 3]
console.log(letters); // ['a', 'b', 'c']
```

---

## Step 447: Spread กับ Class Inheritance Patterns

```javascript
// Mixin pattern ด้วย spread
const Serializable = {
  serialize() {
    return JSON.stringify(this);
  },
  
  toJSON() {
    return Object.fromEntries(
      Object.entries(this).filter(([key]) => !key.startsWith('_'))
    );
  }
};

const Validatable = {
  validate() {
    const rules = this._validationRules || {};
    const errors = {};
    
    for (const [field, rule] of Object.entries(rules)) {
      if (rule.required && !this[field]) {
        errors[field] = 'Required field';
      }
      if (rule.minLength && this[field]?.length < rule.minLength) {
        errors[field] = `Min length: ${rule.minLength}`;
      }
    }
    
    return Object.keys(errors).length ? errors : null;
  }
};

// สร้าง class ที่ใช้ mixins
class User {
  constructor(data) {
    Object.assign(this, Serializable, Validatable, data);
    this._validationRules = {
      name: { required: true, minLength: 2 },
      email: { required: true },
    };
  }
}

const user = new User({ name: 'สมชาย', email: 'test@test.com' });
console.log(user.validate()); // null (valid)
console.log(user.serialize()); // JSON string
```

---

## Step 448: Spread ใน Template Patterns

```javascript
// HTML template generation
function createElement(tag, props = {}, ...children) {
  const attrs = Object.entries(props)
    .map(([key, val]) => ` ${key}="${val}"`)
    .join('');
  
  const childContent = children
    .flat()
    .map(child => typeof child === 'string' ? child : child.toString())
    .join('');
  
  return `<${tag}${attrs}>${childContent}</${tag}>`;
}

const li = (text) => createElement('li', {}, text);
const ul = (...items) => createElement('ul', { class: 'list' }, ...items.map(li));

console.log(ul('แอปเปิล', 'กล้วย', 'ส้ม'));
// <ul class="list"><li>แอปเปิล</li><li>กล้วย</li><li>ส้ม</li></ul>
```

```javascript
// Event system
class EventBus {
  #listeners = {};
  
  on(event, ...handlers) {
    if (!this.#listeners[event]) {
      this.#listeners[event] = [];
    }
    this.#listeners[event].push(...handlers);
    return this;
  }
  
  emit(event, ...args) {
    (this.#listeners[event] || []).forEach(handler => handler(...args));
    return this;
  }
  
  off(event, handler) {
    if (this.#listeners[event]) {
      this.#listeners[event] = this.#listeners[event].filter(h => h !== handler);
    }
    return this;
  }
}

const bus = new EventBus();
const log1 = (msg) => console.log('Handler 1:', msg);
const log2 = (msg) => console.log('Handler 2:', msg);

bus.on('message', log1, log2).emit('message', 'สวัสดี!');
// Handler 1: สวัสดี!
// Handler 2: สวัสดี!
```

---

## Step 449: Performance Considerations

```javascript
// Spread กับ large arrays: ข้อควรระวัง
const largeArray = Array.from({ length: 100000 }, (_, i) => i);

// ✓ ปลอดภัย: concat, slice (ไม่มี stack overflow)
const safe1 = [].concat(largeArray);
const safe2 = largeArray.slice();

// ⚠️ อาจช้ากว่า: spread
const spread1 = [...largeArray];

// ✗ อันตราย: function arguments spread กับ array ใหญ่มาก
// Math.max(...largeArray); // อาจ stack overflow กับ array ใหญ่มาก

// ✓ ปลอดภัยกว่า:
const max = largeArray.reduce((m, v) => Math.max(m, v), -Infinity);

// Benchmark สั้นๆ
console.time('slice');
for (let i = 0; i < 100; i++) largeArray.slice();
console.timeEnd('slice');

console.time('spread');
for (let i = 0; i < 100; i++) [...largeArray];
console.timeEnd('spread');
```

```javascript
// Object spread performance
const obj = Object.fromEntries(
  Array.from({ length: 1000 }, (_, i) => [`key${i}`, i])
);

// ✓ spread สำหรับ object ทั่วไปปลอดภัย
const copy = { ...obj };

// ✓ Object.assign เร็วกว่า spread สำหรับ objects ขนาดใหญ่บางกรณี
const assign = Object.assign({}, obj);

// ✓ สำหรับ immutable updates: spread เป็น idiomatic JavaScript
const updated = { ...obj, key500: 999 };
```

---

## Step 450: สรุปและ Patterns สำคัญ

```javascript
// Pattern 1: Immutable Update
const updateUser = (users, id, updates) =>
  users.map(user => user.id === id ? { ...user, ...updates } : user);

// Pattern 2: Merge with Override
const withDefaults = (options) => ({
  timeout: 3000,
  retries: 3,
  ...options, // override defaults
});

// Pattern 3: Clone + Transform
const transformItems = (items, transform) =>
  [...items].map(transform);

// Pattern 4: Collect Variadic Args
const makeList = (title, ...items) => ({ title, items });

// Pattern 5: Spread Conditionally
const buildRequest = (url, method, body, options = {}) => ({
  url,
  method,
  ...(body && { body: JSON.stringify(body) }),
  headers: {
    'Content-Type': 'application/json',
    ...(options.auth && { Authorization: `Bearer ${options.auth}` }),
  },
});

console.log(buildRequest('/api/users', 'GET'));
// { url: '/api/users', method: 'GET', headers: { 'Content-Type': '...' } }

console.log(buildRequest('/api/users', 'POST', { name: 'สมชาย' }, { auth: 'token123' }));
// { url: '...', method: 'POST', body: '{"name":"สมชาย"}', headers: { ..., Authorization: 'Bearer token123' } }
```

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: Array Operations

```javascript
// Task: สร้าง functional array utilities

// 1. สร้าง function ที่รับ arrays หลายชุดและ return unique values
function uniqueMerge(...arrays) {
  return [...new Set(arrays.flat())];
}

console.log(uniqueMerge([1, 2, 3], [2, 3, 4], [3, 4, 5]));
// [1, 2, 3, 4, 5]

// 2. สร้าง function สำหรับ rotate array
function rotate(arr, n = 1) {
  const normalized = ((n % arr.length) + arr.length) % arr.length;
  return [...arr.slice(normalized), ...arr.slice(0, normalized)];
}

console.log(rotate([1, 2, 3, 4, 5], 2));  // [3, 4, 5, 1, 2]
console.log(rotate([1, 2, 3, 4, 5], -2)); // [4, 5, 1, 2, 3]
```

### แบบฝึกหัดที่ 2: Object Manipulation

```javascript
// Task: สร้าง object transformation functions

// 1. mapValues - apply transform ให้ทุก value
function mapValues(obj, fn) {
  return Object.fromEntries(
    Object.entries(obj).map(([key, value]) => [key, fn(value, key)])
  );
}

const prices = { apple: 10, banana: 5, orange: 8 };
const discounted = mapValues(prices, price => price * 0.9);
console.log(discounted); // { apple: 9, banana: 4.5, orange: 7.2 }

// 2. filterObject - filter ด้วย predicate
function filterObject(obj, predicate) {
  return Object.fromEntries(
    Object.entries(obj).filter(([key, value]) => predicate(value, key))
  );
}

const scores = { A: 95, B: 45, C: 72, D: 38, E: 88 };
const passing = filterObject(scores, score => score >= 50);
console.log(passing); // { A: 95, C: 72, E: 88 }
```

### แบบฝึกหัดที่ 3: สร้าง Pipeline System

```javascript
// Task: สร้าง data processing pipeline

function createPipeline(...transforms) {
  return function(initialData) {
    return transforms.reduce((data, transform) => transform(data), initialData);
  };
}

const rawUsers = [
  { id: 1, name: 'SOMCHAI', age: 17, email: 'somchai@test.com' },
  { id: 2, name: 'SOMYING', age: 25, email: 'SOMYING@TEST.COM' },
  { id: 3, name: 'SOMSRI', age: 15, email: 'somsri@test.com' },
  { id: 4, name: 'SOMDET', age: 32, email: 'somdet@test.com' },
];

const processUsers = createPipeline(
  // Step 1: normalize names
  users => users.map(u => ({
    ...u,
    name: u.name.charAt(0) + u.name.slice(1).toLowerCase()
  })),
  
  // Step 2: normalize emails
  users => users.map(u => ({ ...u, email: u.email.toLowerCase() })),
  
  // Step 3: filter adults only
  users => users.filter(u => u.age >= 18),
  
  // Step 4: add computed field
  users => users.map(u => ({
    ...u,
    isAdult: u.age >= 21,
  }))
);

console.log(processUsers(rawUsers));
// [
//   { id: 2, name: 'Somying', age: 25, email: 'somying@test.com', isAdult: true },
//   { id: 4, name: 'Somdet', age: 32, email: 'somdet@test.com', isAdult: true },
// ]
```

---

## สรุปบทนี้

ในบทนี้เราได้เรียนรู้:

1. **Rest Parameters** - รวบรวม arguments เป็น array (ใช้ใน function parameters)
2. **Rest vs arguments** - Rest เป็น real Array, ใช้ได้ใน arrow functions
3. **Rest ใน Destructuring** - เก็บส่วนที่เหลือ
4. **Spread กับ Arrays** - copy, merge, pass as arguments
5. **Spread กับ Objects** - copy, merge, update immutably
6. **Spread กับ Strings และ Iterables** - Unicode-safe operations
7. **Shallow vs Deep copy** - เข้าใจข้อจำกัดของ spread
8. **Practical Patterns** - immutable updates, conditional spreading, pipelines

**หลักการสำคัญ:**
- Rest รวบรวม, Spread กระจาย - syntax เดียวกัน ทิศทางตรงข้าม
- Spread เป็น shallow copy เสมอ
- ใช้ spread สำหรับ immutable updates ใน state management
- Conditional spreading `...(condition && { key: value })` มีประโยชน์มาก
