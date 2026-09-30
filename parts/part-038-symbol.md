# Part 38: Symbol (Steps 731-750)

## บทนำ

**Symbol** เป็น primitive type ที่ถูกเพิ่มใน ES6 เป็น type ที่ 7 ของ JavaScript (ร่วมกับ undefined, null, boolean, number, string, bigint)

Symbol มีคุณสมบัติพิเศษคือ **ค่าที่สร้างจะ unique เสมอ** ไม่มีค่าไหนเท่ากับ Symbol อื่น และสิ่งนี้ทำให้ Symbol เหมาะสำหรับ:
- สร้าง property keys ที่ไม่ชนกับ key อื่น
- สร้าง "meta-behaviors" ผ่าน well-known Symbols
- สร้าง constants ที่ unique

---

## Step 731: What is Symbol และการสร้าง Symbols

```javascript
// สร้าง Symbol ด้วย Symbol() function (ไม่ใช้ new)
const sym1 = Symbol();
const sym2 = Symbol();

console.log(sym1 === sym2); // false (ทุก Symbol unique)
console.log(typeof sym1);   // "symbol"

// Symbol พร้อม description (เป็น string สำหรับ debug เท่านั้น)
const sym3 = Symbol("my symbol");
const sym4 = Symbol("my symbol");

console.log(sym3 === sym4); // false (ยังคง unique แม้ description เหมือนกัน)
console.log(sym3.toString()); // "Symbol(my symbol)"
```

```javascript
// Symbol ไม่สามารถ instantiate ด้วย new ได้
try {
  const sym = new Symbol(); // TypeError!
} catch (e) {
  console.log(e.message); // "Symbol is not a constructor"
}

// Symbol ไม่สามารถ convert เป็น string อัตโนมัติ
const sym = Symbol("test");
try {
  console.log("Symbol: " + sym); // TypeError!
} catch (e) {
  console.log(e.message); // Cannot convert a Symbol value to a string
}

// ต้องใช้ template literal หรือ .toString()
console.log(`Symbol: ${sym}`);       // "Symbol: Symbol(test)"
console.log("Symbol: " + sym.toString()); // "Symbol: Symbol(test)"
```

---

## Step 732: Symbol.description

```javascript
// .description property (ES2019) - คืน description string
const sym1 = Symbol("hello world");
console.log(sym1.description); // "hello world"

const sym2 = Symbol();
console.log(sym2.description); // undefined

// ต่างจาก .toString()
console.log(sym1.toString());   // "Symbol(hello world)"
console.log(sym1.description);  // "hello world"
```

```javascript
// ใช้ description สำหรับ debugging
function createDebugSymbol(name) {
  const sym = Symbol(`DEBUG:${name}`);
  console.log(`Created symbol: ${sym.description}`);
  return sym;
}

const dbgSym = createDebugSymbol("userId");
console.log(dbgSym.description); // "DEBUG:userId"
```

---

## Step 733: Symbols เป็น Unique (Symbol('a') !== Symbol('a'))

```javascript
// ทุก Symbol call สร้างค่าใหม่เสมอ
const a = Symbol("key");
const b = Symbol("key");
const c = Symbol("key");

console.log(a === b); // false
console.log(b === c); // false
console.log(a === c); // false

// จำนวน Symbols ไม่จำกัด ทุกอันต่างกัน
const symbols = Array.from({ length: 1000 }, (_, i) => Symbol(`sym-${i}`));
const allUnique = symbols.every((s, i) =>
  symbols.every((t, j) => i === j || s !== t)
);
console.log(allUnique); // true
```

```javascript
// ตัวอย่าง: Enum-like constants
const Direction = {
  NORTH: Symbol("NORTH"),
  SOUTH: Symbol("SOUTH"),
  EAST: Symbol("EAST"),
  WEST: Symbol("WEST"),
};

function move(direction) {
  switch (direction) {
    case Direction.NORTH:
      return "Moving North";
    case Direction.SOUTH:
      return "Moving South";
    case Direction.EAST:
      return "Moving East";
    case Direction.WEST:
      return "Moving West";
    default:
      return "Unknown direction";
  }
}

console.log(move(Direction.NORTH));  // "Moving North"
console.log(move(Direction.SOUTH));  // "Moving South"
console.log(move("NORTH"));          // "Unknown direction" (string ≠ Symbol)

// แตกต่างจาก string constants ที่อาจชนกัน
const RED = "color"; // อาจชนกับ const อื่น
const STOP = "color"; // ชนกัน!
const RED_SYM = Symbol("color");
const STOP_SYM = Symbol("color");
console.log(RED === STOP);          // true (ชนกัน - ปัญหา!)
console.log(RED_SYM === STOP_SYM); // false (ไม่ชน - ปลอดภัย)
```

---

## Step 734: Using Symbol as Object Keys

```javascript
// Symbol เป็น property key ได้
const id = Symbol("id");
const user = {
  name: "สมชาย",
  age: 30,
  [id]: "user-001", // computed property name ด้วย []
};

console.log(user[id]);   // "user-001"
console.log(user.name);  // "สมชาย"

// Symbol keys ไม่ปรากฏใน for-in
for (const key in user) {
  console.log(key); // "name", "age" (ไม่มี id)
}

// Symbol keys ไม่ปรากฏใน Object.keys()
console.log(Object.keys(user));   // ["name", "age"]

// ต้องใช้ Object.getOwnPropertySymbols() เพื่อดู Symbol keys
console.log(Object.getOwnPropertySymbols(user)); // [Symbol(id)]

// หรือ Reflect.ownKeys() ที่รวมทั้งหมด
console.log(Reflect.ownKeys(user)); // ["name", "age", Symbol(id)]
```

```javascript
// Symbol keys ไม่ถูก enumerate ใน JSON.stringify
const secret = Symbol("secret");
const obj = {
  name: "public data",
  [secret]: "private data",
};

const json = JSON.stringify(obj);
console.log(json); // {"name":"public data"} (ไม่มี secret)
```

```javascript
// ตัวอย่าง: เพิ่ม property ใน 3rd party object ปลอดภัย
const thirdPartyObj = { data: "important" };

// วิธีเก่า (อาจชน)
// thirdPartyObj._myLib_id = "123"; // ugly & might conflict

// วิธีที่ดีกว่า
const MY_LIB_ID = Symbol("my-lib-id");
thirdPartyObj[MY_LIB_ID] = "123"; // ปลอดภัย ไม่ชน

console.log(thirdPartyObj[MY_LIB_ID]); // "123"
console.log(thirdPartyObj.data);        // "important" (ยังอยู่)
```

---

## Step 735: Symbol vs String Keys

```javascript
// ความแตกต่างระหว่าง Symbol และ String keys

const strKey = "id";
const symKey = Symbol("id");

const obj = {
  [strKey]: "string-value",
  [symKey]: "symbol-value",
};

// เข้าถึงต่างกัน
console.log(obj["id"]);    // "string-value"
console.log(obj[strKey]);  // "string-value"
console.log(obj[symKey]);  // "symbol-value"
console.log(obj.id);       // "string-value"

// String key ปรากฏทุกที่
console.log(Object.keys(obj));          // ["id"]
console.log(JSON.stringify(obj));        // {"id":"string-value"}

// Symbol key ซ่อนอยู่
console.log(Object.getOwnPropertySymbols(obj)); // [Symbol(id)]
```

```javascript
// ตัวอย่าง: Library ที่ใช้ Symbol สำหรับ internal state
const INTERNAL = Symbol("internal");

class EventEmitter {
  constructor() {
    this[INTERNAL] = {
      events: new Map(),
    };
  }
  
  on(event, listener) {
    const events = this[INTERNAL].events;
    if (!events.has(event)) {
      events.set(event, []);
    }
    events.get(event).push(listener);
    return this;
  }
  
  emit(event, ...args) {
    const listeners = this[INTERNAL].events.get(event) || [];
    listeners.forEach((listener) => listener(...args));
    return this;
  }
  
  off(event, listener) {
    const events = this[INTERNAL].events;
    if (events.has(event)) {
      const filtered = events.get(event).filter((l) => l !== listener);
      events.set(event, filtered);
    }
    return this;
  }
}

const emitter = new EventEmitter();

const handler = (data) => console.log("Received:", data);
emitter.on("data", handler);
emitter.emit("data", "Hello!"); // "Received: Hello!"

// Internal state ถูกซ่อน
console.log(Object.keys(emitter)); // [] (ว่าง)
```

---

## Step 736: Well-known Symbols Overview

JavaScript มี **well-known Symbols** ที่ใช้เป็น hook สำหรับ built-in behaviors:

```javascript
// รายการ well-known Symbols
console.log(Symbol.iterator);        // สำหรับ for-of, spread
console.log(Symbol.asyncIterator);   // สำหรับ for-await-of
console.log(Symbol.toPrimitive);     // สำหรับ type conversion
console.log(Symbol.toStringTag);     // สำหรับ Object.prototype.toString
console.log(Symbol.hasInstance);     // สำหรับ instanceof
console.log(Symbol.isConcatSpreadable); // สำหรับ Array.prototype.concat
console.log(Symbol.species);         // สำหรับ derived classes
console.log(Symbol.match);           // สำหรับ String.prototype.match
console.log(Symbol.replace);         // สำหรับ String.prototype.replace
console.log(Symbol.search);          // สำหรับ String.prototype.search
console.log(Symbol.split);           // สำหรับ String.prototype.split
console.log(Symbol.unscopables);     // สำหรับ with statement
```

---

## Step 737: Symbol.iterator - Custom Iterables

```javascript
// กำหนด custom iteration behavior
class Range {
  constructor(start, end, step = 1) {
    this.start = start;
    this.end = end;
    this.step = step;
  }
  
  [Symbol.iterator]() {
    let current = this.start;
    const { end, step } = this;
    
    return {
      next() {
        if (current <= end) {
          const value = current;
          current += step;
          return { value, done: false };
        }
        return { value: undefined, done: true };
      },
      
      [Symbol.iterator]() {
        return this;
      }
    };
  }
}

const range = new Range(1, 10, 2);
for (const num of range) {
  console.log(num); // 1, 3, 5, 7, 9
}

console.log([...new Range(0, 5)]); // [0, 1, 2, 3, 4, 5]
```

```javascript
// ตัวอย่าง: Matrix iterables
class Matrix {
  #data;
  #rows;
  #cols;
  
  constructor(rows, cols) {
    this.#rows = rows;
    this.#cols = cols;
    this.#data = Array.from({ length: rows }, () => new Array(cols).fill(0));
  }
  
  set(row, col, value) {
    this.#data[row][col] = value;
    return this;
  }
  
  get(row, col) {
    return this.#data[row][col];
  }
  
  // iterate rows
  *rows() {
    for (const row of this.#data) {
      yield row;
    }
  }
  
  // iterate columns
  *cols() {
    for (let col = 0; col < this.#cols; col++) {
      yield this.#data.map((row) => row[col]);
    }
  }
  
  // default: iterate all values in row-major order
  [Symbol.iterator]() {
    const data = this.#data;
    let row = 0, col = 0;
    
    return {
      next() {
        if (row >= data.length) return { done: true };
        
        const value = data[row][col];
        col++;
        if (col >= data[0].length) {
          col = 0;
          row++;
        }
        
        return { value, done: false };
      }
    };
  }
}

const m = new Matrix(2, 3);
m.set(0, 0, 1).set(0, 1, 2).set(0, 2, 3)
 .set(1, 0, 4).set(1, 1, 5).set(1, 2, 6);

console.log([...m]); // [1, 2, 3, 4, 5, 6]
```

---

## Step 738: Symbol.toPrimitive - Custom Type Conversion

```javascript
// Symbol.toPrimitive กำหนดการ convert เป็น primitive

class Temperature {
  constructor(celsius) {
    this.celsius = celsius;
  }
  
  [Symbol.toPrimitive](hint) {
    switch (hint) {
      case "number":
        return this.celsius; // คืนเป็น number
      case "string":
        return `${this.celsius}°C`;
      default: // "default" hint
        return this.celsius;
    }
  }
  
  get fahrenheit() {
    return this.celsius * 9 / 5 + 32;
  }
}

const temp = new Temperature(100);

// number context
console.log(+temp);        // 100
console.log(temp + 0);     // 100
console.log(temp > 37);    // true

// string context
console.log(`${temp}`);    // "100°C"
console.log(String(temp)); // "100°C"

// default context (unary + and == use this)
console.log(temp == 100);  // true
```

```javascript
// ตัวอย่าง: Money class
class Money {
  constructor(amount, currency = "THB") {
    this.amount = amount;
    this.currency = currency;
  }
  
  [Symbol.toPrimitive](hint) {
    if (hint === "number") return this.amount;
    if (hint === "string") return `${this.amount} ${this.currency}`;
    return this.amount; // default
  }
  
  add(other) {
    if (this.currency !== other.currency) {
      throw new Error("Currency mismatch");
    }
    return new Money(this.amount + other.amount, this.currency);
  }
  
  toString() {
    return `${this.amount.toLocaleString()} ${this.currency}`;
  }
}

const price = new Money(1000, "THB");
const tax = new Money(70, "THB");
const total = price.add(tax);

console.log(`ราคา: ${price}`);    // "ราคา: 1000 THB"
console.log(`ภาษี: ${tax}`);      // "ภาษี: 70 THB"
console.log(`รวม: ${total}`);     // "รวม: 1070 THB"
console.log(price > 500);          // true (number context)
console.log(+price + +tax);       // 1070 (number)
```

---

## Step 739: Symbol.hasInstance - Custom instanceof

```javascript
// Symbol.hasInstance กำหนดพฤติกรรมของ instanceof

class EvenNumber {
  static [Symbol.hasInstance](value) {
    return typeof value === "number" && value % 2 === 0;
  }
}

console.log(4 instanceof EvenNumber);   // true
console.log(5 instanceof EvenNumber);   // false
console.log(100 instanceof EvenNumber); // true
console.log("4" instanceof EvenNumber); // false
```

```javascript
// ตัวอย่าง: Custom type checking
class TypeChecker {
  constructor(typeName, checkFn) {
    this.typeName = typeName;
    this.checkFn = checkFn;
  }
  
  static [Symbol.hasInstance](value) {
    // ไม่ควรใช้ในกรณีนี้เพราะ TypeChecker ไม่ได้ represent type เฉพาะ
    return false;
  }
}

const Integer = {
  [Symbol.hasInstance](value) {
    return Number.isInteger(value);
  }
};

const PositiveNumber = {
  [Symbol.hasInstance](value) {
    return typeof value === "number" && value > 0;
  }
};

const NonEmptyString = {
  [Symbol.hasInstance](value) {
    return typeof value === "string" && value.length > 0;
  }
};

console.log(42 instanceof Integer);        // true
console.log(42.5 instanceof Integer);      // false
console.log(42 instanceof PositiveNumber); // true
console.log(-5 instanceof PositiveNumber); // false
console.log("hello" instanceof NonEmptyString); // true
console.log("" instanceof NonEmptyString);      // false
```

---

## Step 740: Symbol.toStringTag - Custom toString

```javascript
// Symbol.toStringTag กำหนดผลของ Object.prototype.toString.call()

class MyClass {
  get [Symbol.toStringTag]() {
    return "MyClass";
  }
}

const obj = new MyClass();
console.log(Object.prototype.toString.call(obj)); // "[object MyClass]"
console.log(obj.toString()); // "[object MyClass]" (ถ้าไม่ override toString)
```

```javascript
// Built-in types มี toStringTag
console.log(Object.prototype.toString.call([]));          // "[object Array]"
console.log(Object.prototype.toString.call(new Map()));   // "[object Map]"
console.log(Object.prototype.toString.call(new Set()));   // "[object Set]"
console.log(Object.prototype.toString.call(Promise.resolve())); // "[object Promise]"
console.log(Object.prototype.toString.call(/regex/));    // "[object RegExp]"
console.log(Object.prototype.toString.call(new Date())); // "[object Date]"
```

```javascript
// ใช้สำหรับ type checking ที่แม่นยำ
function getType(value) {
  return Object.prototype.toString.call(value).slice(8, -1);
}

console.log(getType([]));          // "Array"
console.log(getType({}));          // "Object"
console.log(getType(new Map()));   // "Map"
console.log(getType(null));        // "Null"
console.log(getType(undefined));   // "Undefined"
console.log(getType(42));          // "Number"
console.log(getType("hello"));     // "String"
console.log(getType(true));        // "Boolean"

// Custom class
class DatabaseConnection {
  get [Symbol.toStringTag]() {
    return "DatabaseConnection";
  }
}

const conn = new DatabaseConnection();
console.log(getType(conn)); // "DatabaseConnection"
```

---

## Step 741: Symbol.species - Custom Species

```javascript
// Symbol.species กำหนด constructor ที่ใช้สร้าง derived objects

class MyArray extends Array {
  static get [Symbol.species]() {
    return Array; // return Array ธรรมดา (ไม่ใช่ MyArray)
  }
  
  sum() {
    return this.reduce((a, b) => a + b, 0);
  }
}

const myArr = new MyArray(1, 2, 3, 4, 5);
console.log(myArr instanceof MyArray); // true
console.log(myArr.sum());              // 15

// map, filter, slice จะสร้าง Array ธรรมดา (ไม่ใช่ MyArray)
const doubled = myArr.map((x) => x * 2);
console.log(doubled instanceof MyArray); // false (เพราะ species = Array)
console.log(doubled instanceof Array);   // true

// ถ้าไม่มี Symbol.species จะสร้าง MyArray
class MyArray2 extends Array {
  sum() {
    return this.reduce((a, b) => a + b, 0);
  }
}

const arr2 = new MyArray2(1, 2, 3);
const doubled2 = arr2.map((x) => x * 2);
console.log(doubled2 instanceof MyArray2); // true
console.log(doubled2.sum());               // 12
```

```javascript
// Symbol.species กับ Promise
class MyPromise extends Promise {
  static get [Symbol.species]() {
    return Promise; // .then() จะคืน Promise ธรรมดา
  }
  
  myMethod() {
    return this.then((v) => v * 2);
  }
}

const mp = new MyPromise((resolve) => resolve(5));
const result = mp.then((v) => v + 1);
console.log(result instanceof Promise);   // true
console.log(result instanceof MyPromise); // false (เพราะ species = Promise)
```

---

## Step 742: Symbol.for() Global Registry

```javascript
// Symbol.for() สร้างหรือดึง Symbol จาก global registry
const sym1 = Symbol.for("shared-key");
const sym2 = Symbol.for("shared-key");

console.log(sym1 === sym2); // true! (ค่าเดียวกัน)

// ต่างจาก Symbol() ที่สร้างใหม่เสมอ
const local1 = Symbol("local");
const local2 = Symbol("local");
console.log(local1 === local2); // false
```

```javascript
// ใช้ Symbol.for() เมื่อต้องการ share Symbol ระหว่าง modules
// module-a.js
const EVENT_READY = Symbol.for("myapp.events.ready");
// module-b.js
const EVENT_READY = Symbol.for("myapp.events.ready"); // ค่าเดียวกัน

// ตัวอย่าง: Plugin system
const PLUGIN_READY = Symbol.for("plugin.ready");
const PLUGIN_DESTROY = Symbol.for("plugin.destroy");

class PluginBase {
  [PLUGIN_READY]() {
    console.log(`${this.constructor.name} is ready`);
  }
  
  [PLUGIN_DESTROY]() {
    console.log(`${this.constructor.name} destroyed`);
  }
}

class MyPlugin extends PluginBase {
  [PLUGIN_READY]() {
    super[PLUGIN_READY]();
    console.log("MyPlugin: setting up...");
  }
}

function initPlugin(plugin) {
  const readyFn = plugin[Symbol.for("plugin.ready")];
  if (readyFn) readyFn.call(plugin);
}

initPlugin(new MyPlugin());
```

---

## Step 743: Symbol.keyFor()

```javascript
// Symbol.keyFor() - หา key จาก global Symbol registry
const globalSym = Symbol.for("my-key");
const localSym = Symbol("my-key");

console.log(Symbol.keyFor(globalSym)); // "my-key" (ถ้า registered)
console.log(Symbol.keyFor(localSym));  // undefined (ไม่ได้ register)

// ใช้ตรวจสอบว่า Symbol มาจาก global registry หรือไม่
function isGlobalSymbol(sym) {
  return Symbol.keyFor(sym) !== undefined;
}

console.log(isGlobalSymbol(Symbol.for("test"))); // true
console.log(isGlobalSymbol(Symbol("test")));     // false
console.log(isGlobalSymbol(Symbol.iterator));    // false (well-known ไม่ได้อยู่ใน global registry)
```

---

## Step 744: Symbols ใน JSON

```javascript
// Symbol keys ไม่รวมอยู่ใน JSON
const sym = Symbol("key");
const obj = {
  normalKey: "normal value",
  [sym]: "symbol value",
  nested: {
    [Symbol("nested")]: "nested symbol",
    data: 42,
  },
};

const json = JSON.stringify(obj);
console.log(json);
// {"normalKey":"normal value","nested":{"data":42}}
// Symbol keys หายไปทั้งหมด

// ทางเลือก: serialize Symbols เอง
function serializeWithSymbols(obj) {
  const symbolKeys = Object.getOwnPropertySymbols(obj);
  const result = { ...obj };
  
  symbolKeys.forEach((sym) => {
    result[sym.toString()] = obj[sym];
  });
  
  return JSON.stringify(result);
}
```

```javascript
// Symbol values ก็ไม่รวมเช่นกัน
const obj2 = {
  key: Symbol("value"), // Symbol เป็น value
  arr: [1, Symbol("x"), 2],
};

console.log(JSON.stringify(obj2));
// {"arr":[1,null,2]} -- Symbol value กลายเป็น undefined (และ null ใน array)
```

---

## Step 745: Use Cases สำหรับ Symbols

### Use Case 1: Unique Constants / Enums

```javascript
// Enums ที่ unique และ self-documenting
const Status = Object.freeze({
  PENDING: Symbol("PENDING"),
  ACTIVE: Symbol("ACTIVE"),
  INACTIVE: Symbol("INACTIVE"),
  DELETED: Symbol("DELETED"),
});

class User {
  #status = Status.PENDING;
  
  activate() {
    this.#status = Status.ACTIVE;
    return this;
  }
  
  deactivate() {
    this.#status = Status.INACTIVE;
    return this;
  }
  
  delete() {
    this.#status = Status.DELETED;
    return this;
  }
  
  get status() {
    return this.#status;
  }
  
  get statusName() {
    return this.#status.description;
  }
  
  isActive() {
    return this.#status === Status.ACTIVE;
  }
}

const user = new User();
console.log(user.statusName); // "PENDING"
user.activate();
console.log(user.isActive()); // true
console.log(user.status === Status.ACTIVE); // true
```

### Use Case 2: Protocol / Duck Typing

```javascript
// Symbol สำหรับ define protocols
const Serializable = Symbol("Serializable");
const Comparable = Symbol("Comparable");

class Product {
  constructor(name, price) {
    this.name = name;
    this.price = price;
  }
  
  [Serializable]() {
    return JSON.stringify({ name: this.name, price: this.price });
  }
  
  [Comparable](other) {
    return this.price - other.price;
  }
}

function serialize(obj) {
  if (typeof obj[Serializable] === "function") {
    return obj[Serializable]();
  }
  throw new Error("Object is not Serializable");
}

function compare(a, b) {
  if (typeof a[Comparable] === "function") {
    return a[Comparable](b);
  }
  throw new Error("Object is not Comparable");
}

const apple = new Product("Apple", 20);
const banana = new Product("Banana", 15);

console.log(serialize(apple));   // '{"name":"Apple","price":20}'
console.log(compare(apple, banana) > 0); // true (apple more expensive)
```

### Use Case 3: Mixin Protocols

```javascript
// ใช้ Symbol สำหรับ mixin interface
const Flyable = Symbol("Flyable");
const Swimmable = Symbol("Swimmable");

const FlyMixin = {
  [Flyable]() {
    return `${this.name} is flying`;
  }
};

const SwimMixin = {
  [Swimmable]() {
    return `${this.name} is swimming`;
  }
};

class Duck {
  constructor(name) {
    this.name = name;
    Object.assign(this, FlyMixin, SwimMixin);
  }
}

class Fish {
  constructor(name) {
    this.name = name;
    Object.assign(this, SwimMixin);
  }
}

const donald = new Duck("Donald");
console.log(donald[Flyable]());  // "Donald is flying"
console.log(donald[Swimmable]()); // "Donald is swimming"

const nemo = new Fish("Nemo");
console.log(nemo[Swimmable]()); // "Nemo is swimming"
// nemo[Flyable]() -- TypeError (Fish can't fly)
```

---

## Step 746: Symbol.isConcatSpreadable

```javascript
// Symbol.isConcatSpreadable กำหนดพฤติกรรมใน Array.prototype.concat

const arr1 = [1, 2, 3];
const arr2 = [4, 5, 6];

// ปกติ array จะถูก spread ใน concat
console.log([].concat(arr1, arr2)); // [1, 2, 3, 4, 5, 6]

// ปิด concat spreading
const arr3 = [7, 8, 9];
arr3[Symbol.isConcatSpreadable] = false;
console.log([].concat(arr1, arr3)); // [1, 2, 3, [7, 8, 9]]

// เปิด concat spreading สำหรับ non-array object
const arrayLike = { 0: "a", 1: "b", length: 2 };
arrayLike[Symbol.isConcatSpreadable] = true;
console.log([].concat(["x"], arrayLike)); // ['x', 'a', 'b']
```

---

## Step 747: Symbol.match, Symbol.replace, Symbol.search, Symbol.split

```javascript
// กำหนด custom behavior สำหรับ String methods

class CaseInsensitiveMatcher {
  constructor(str) {
    this.str = str.toLowerCase();
  }
  
  [Symbol.match](string) {
    const lowerString = string.toLowerCase();
    const index = lowerString.indexOf(this.str);
    if (index === -1) return null;
    return [string.slice(index, index + this.str.length)];
  }
  
  [Symbol.search](string) {
    return string.toLowerCase().indexOf(this.str);
  }
  
  [Symbol.replace](string, replacement) {
    return string.replace(new RegExp(this.str, "gi"), replacement);
  }
  
  [Symbol.split](string) {
    return string.split(new RegExp(this.str, "gi"));
  }
}

const matcher = new CaseInsensitiveMatcher("hello");
const text = "Hello World, HELLO JavaScript, hello!";

console.log(text.match(matcher));    // ['Hello']
console.log(text.search(matcher));   // 0
console.log(text.replace(matcher, "Hi")); // "Hi World, Hi JavaScript, Hi!"
console.log(text.split(matcher));    // ['', ' World, ', ' JavaScript, ', '!']
```

---

## Step 748: ตัวอย่างขั้นสูง - Symbol ใน Framework/Library

```javascript
// Design pattern: Observable ด้วย Symbol

const OBSERVERS = Symbol("observers");
const NOTIFY = Symbol("notify");

class Observable {
  constructor() {
    this[OBSERVERS] = new Map();
  }
  
  on(event, handler) {
    if (!this[OBSERVERS].has(event)) {
      this[OBSERVERS].set(event, new Set());
    }
    this[OBSERVERS].get(event).add(handler);
    
    // คืน unsubscribe function
    return () => this.off(event, handler);
  }
  
  off(event, handler) {
    const handlers = this[OBSERVERS].get(event);
    if (handlers) handlers.delete(handler);
  }
  
  [NOTIFY](event, data) {
    const handlers = this[OBSERVERS].get(event) || new Set();
    handlers.forEach((handler) => handler(data));
  }
}

class Store extends Observable {
  #state;
  
  constructor(initialState) {
    super();
    this.#state = { ...initialState };
  }
  
  get(key) {
    return this.#state[key];
  }
  
  set(key, value) {
    const oldValue = this.#state[key];
    this.#state[key] = value;
    
    if (oldValue !== value) {
      this[NOTIFY]("change", { key, oldValue, newValue: value });
      this[NOTIFY](`change:${key}`, { oldValue, newValue: value });
    }
    
    return this;
  }
  
  getState() {
    return { ...this.#state };
  }
}

const store = new Store({ count: 0, name: "initial" });

const unsubscribe = store.on("change", ({ key, oldValue, newValue }) => {
  console.log(`${key}: ${oldValue} -> ${newValue}`);
});

store.on("change:count", ({ newValue }) => {
  console.log(`Count is now: ${newValue}`);
});

store.set("count", 1);
// count: 0 -> 1
// Count is now: 1

store.set("count", 2);
// count: 1 -> 2
// Count is now: 2

unsubscribe(); // ยกเลิก subscription แรก

store.set("count", 3);
// Count is now: 3 (handler แรกไม่ถูกเรียก)

// Internal state ถูกซ่อนด้วย Symbol
console.log(Object.keys(store)); // [] (ว่าง)
```

---

## Step 749: Symbol กับ Proxy

```javascript
// Symbol properties ทำงานกับ Proxy
const PRIVATE = Symbol("private");

function createSecureObject(data) {
  const obj = {
    public: data.public,
    [PRIVATE]: data.private,
  };
  
  return new Proxy(obj, {
    get(target, prop) {
      if (prop === PRIVATE) {
        throw new Error("Access to private data denied");
      }
      return Reflect.get(target, prop);
    },
    
    set(target, prop, value) {
      if (prop === PRIVATE) {
        throw new Error("Cannot modify private data");
      }
      return Reflect.set(target, prop, value);
    },
    
    ownKeys(target) {
      // ซ่อน Symbol keys
      return Reflect.ownKeys(target).filter(
        (key) => typeof key !== "symbol"
      );
    }
  });
}

const secure = createSecureObject({
  public: "This is public",
  private: "This is secret",
});

console.log(secure.public); // "This is public"
try {
  console.log(secure[PRIVATE]); // Error: Access to private data denied
} catch (e) {
  console.log(e.message);
}
```

---

## Step 750: ตัวอย่างสมบูรณ์ - Symbol-based Type System

```javascript
// Type system ขนาดเล็กโดยใช้ Symbols

const TYPE = Symbol("type");
const VALIDATE = Symbol("validate");
const COERCE = Symbol("coerce");

// Type definitions
const Types = {
  String: {
    [TYPE]: "String",
    [VALIDATE]: (v) => typeof v === "string",
    [COERCE]: (v) => String(v),
  },
  
  Number: {
    [TYPE]: "Number",
    [VALIDATE]: (v) => typeof v === "number" && !isNaN(v),
    [COERCE]: (v) => Number(v),
  },
  
  Boolean: {
    [TYPE]: "Boolean",
    [VALIDATE]: (v) => typeof v === "boolean",
    [COERCE]: (v) => Boolean(v),
  },
  
  Email: {
    [TYPE]: "Email",
    [VALIDATE]: (v) => typeof v === "string" && /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(v),
    [COERCE]: (v) => String(v).toLowerCase().trim(),
  },
  
  PositiveInt: {
    [TYPE]: "PositiveInt",
    [VALIDATE]: (v) => Number.isInteger(v) && v > 0,
    [COERCE]: (v) => Math.abs(Math.round(Number(v))),
  },
};

class Schema {
  #fields;
  
  constructor(fields) {
    this.#fields = fields;
  }
  
  validate(data) {
    const errors = [];
    
    for (const [fieldName, type] of Object.entries(this.#fields)) {
      const value = data[fieldName];
      
      if (!type[VALIDATE](value)) {
        errors.push({
          field: fieldName,
          type: type[TYPE],
          value,
          message: `Expected ${type[TYPE]} for field "${fieldName}", got "${typeof value}"`,
        });
      }
    }
    
    return {
      valid: errors.length === 0,
      errors,
    };
  }
  
  coerce(data) {
    const result = {};
    
    for (const [fieldName, type] of Object.entries(this.#fields)) {
      result[fieldName] = type[COERCE](data[fieldName]);
    }
    
    return result;
  }
}

// ใช้งาน
const UserSchema = new Schema({
  name: Types.String,
  age: Types.PositiveInt,
  email: Types.Email,
  active: Types.Boolean,
});

const rawData = {
  name: "สมชาย",
  age: "30",     // string แทนที่จะเป็น number
  email: "somchai@EXAMPLE.com",
  active: "true", // string แทน boolean
};

// Validate ก่อน
const { valid, errors } = UserSchema.validate(rawData);
console.log("Valid:", valid); // false

if (!valid) {
  errors.forEach((e) => console.log(e.message));
}

// Coerce ให้ถูกต้อง
const coerced = UserSchema.coerce(rawData);
console.log(coerced);
// { name: 'สมชาย', age: 30, email: 'somchai@example.com', active: true }

// Validate อีกครั้ง
const { valid: valid2 } = UserSchema.validate(coerced);
console.log("Valid after coerce:", valid2); // true
```

---

## แบบฝึกหัด

### Easy
1. สร้าง Symbol constants สำหรับ HTTP methods (GET, POST, PUT, DELETE, PATCH)
2. ใช้ Symbol.toPrimitive เพื่อสร้าง Duration class (ที่ convert เป็น seconds เมื่อเป็น number)
3. เพิ่ม Symbol.toStringTag ให้กับ custom classes

### Medium
4. สร้าง EventSystem ที่ใช้ Symbol เป็น event names เพื่อป้องกัน name collision
5. Implement Symbol.hasInstance สำหรับ URL validator
6. สร้าง Comparable mixin โดยใช้ Symbol ที่ทำให้ class รองรับ `<`, `>`, `<=`, `>=`

### Hard
7. Implement type-safe Record ที่ใช้ Symbol สำหรับ field descriptors
8. สร้าง mini Redux โดยใช้ Symbol สำหรับ action types
9. Implement method decorator system โดยใช้ Symbols

### Solution ตัวอย่าง

```javascript
// 2. Duration class
class Duration {
  constructor({ hours = 0, minutes = 0, seconds = 0 } = {}) {
    this.hours = hours;
    this.minutes = minutes;
    this.seconds = seconds;
  }
  
  get totalSeconds() {
    return this.hours * 3600 + this.minutes * 60 + this.seconds;
  }
  
  [Symbol.toPrimitive](hint) {
    if (hint === "number") return this.totalSeconds;
    if (hint === "string") {
      const h = String(this.hours).padStart(2, "0");
      const m = String(this.minutes).padStart(2, "0");
      const s = String(this.seconds).padStart(2, "0");
      return `${h}:${m}:${s}`;
    }
    return this.totalSeconds;
  }
  
  add(other) {
    const totalSecs = +this + +other;
    return new Duration({
      hours: Math.floor(totalSecs / 3600),
      minutes: Math.floor((totalSecs % 3600) / 60),
      seconds: totalSecs % 60,
    });
  }
}

const d1 = new Duration({ hours: 1, minutes: 30, seconds: 0 });
const d2 = new Duration({ minutes: 45, seconds: 30 });

console.log(`${d1}`);          // "01:30:00"
console.log(+d1);               // 5400 (seconds)
console.log(d1 > d2);           // true
const total = d1.add(d2);
console.log(`${total}`);       // "02:15:30"

// 5. URL validator instanceof
const ValidURL = {
  [Symbol.hasInstance](value) {
    try {
      new URL(value);
      return true;
    } catch {
      return false;
    }
  }
};

console.log("https://example.com" instanceof ValidURL); // true
console.log("not-a-url" instanceof ValidURL);           // false
console.log("ftp://files.example.com" instanceof ValidURL); // true
```

---

## สรุป

| Symbol | ใช้สำหรับ |
|--------|----------|
| `Symbol()` | สร้าง unique identifier |
| `Symbol.for()` | Global registry (shared across modules) |
| `Symbol.iterator` | Custom iteration |
| `Symbol.asyncIterator` | Custom async iteration |
| `Symbol.toPrimitive` | Custom type conversion |
| `Symbol.toStringTag` | Custom toString description |
| `Symbol.hasInstance` | Custom instanceof |
| `Symbol.species` | Custom derived class constructor |
| `Symbol.isConcatSpreadable` | Custom concat behavior |

**เมื่อไหร่ควรใช้ Symbol:**
- ต้องการ unique constants / enums
- ต้องการเพิ่ม property ที่ไม่ชนกับ property อื่น
- ต้องการ implement protocols/interfaces
- ต้องการ well-known hooks (iterator, toPrimitive, etc.)
- ต้องการ "private" properties (semi-private จริง ๆ)
