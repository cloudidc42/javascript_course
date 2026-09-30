# Part 39: Proxy และ Reflect (Steps 751-770)

## บทนำ

**Proxy** และ **Reflect** เป็น meta-programming features ที่เพิ่มใน ES6 ที่ช่วยให้เราสามารถ "intercept" และ "customize" พฤติกรรมพื้นฐานของ JavaScript objects ได้

- **Proxy**: wrapper ที่ดักจับ (intercept) operations บน object
- **Reflect**: API ที่ให้ access ไปยัง built-in operations

---

## Step 751: Proxy Concept

```javascript
// Proxy ห่อหุ้ม object อื่น (target) และ intercept operations

const target = {
  name: "สมชาย",
  age: 30,
};

const handler = {
  get(target, property, receiver) {
    console.log(`Getting: ${property}`);
    return Reflect.get(target, property, receiver);
  },
  
  set(target, property, value, receiver) {
    console.log(`Setting: ${property} = ${value}`);
    return Reflect.set(target, property, value, receiver);
  },
};

const proxy = new Proxy(target, handler);

console.log(proxy.name);  // Getting: name -> "สมชาย"
proxy.age = 31;           // Setting: age = 31
console.log(proxy.age);   // Getting: age -> 31
```

```javascript
// Proxy สามารถ wrap ทุก object ได้
// 1. wrap array
const arrProxy = new Proxy([], {
  set(target, index, value) {
    console.log(`Array[${index}] = ${value}`);
    target[index] = value;
    return true;
  }
});
arrProxy.push(1, 2, 3);

// 2. wrap function
const fnProxy = new Proxy(function(x) { return x * 2; }, {
  apply(target, thisArg, args) {
    console.log(`Called with: ${args}`);
    return target.apply(thisArg, args);
  }
});
console.log(fnProxy(5)); // Called with: 5 -> 10

// 3. wrap class constructor
const ClassProxy = new Proxy(class Person {
  constructor(name) {
    this.name = name;
  }
}, {
  construct(target, args) {
    console.log(`Constructing with: ${args}`);
    return new target(...args);
  }
});

const person = new ClassProxy("สมชาย");
console.log(person.name); // "สมชาย"
```

---

## Step 752: Creating a Proxy

```javascript
// Syntax: new Proxy(target, handler)
// target: object ที่จะ wrap
// handler: object ที่มี "traps" (interceptors)

// Handler ว่าง = pass-through proxy
const emptyHandler = {};
const passThroughProxy = new Proxy({ x: 1 }, emptyHandler);
console.log(passThroughProxy.x); // 1 (ผ่านไปตรง ๆ)

// ตรวจสอบว่าเป็น Proxy
// ไม่มีวิธีตรงที่จะตรวจสอบ (by design)
// แต่ Proxy ยัง instanceof ได้
class MyClass {}
const myProxy = new Proxy(new MyClass(), {});
console.log(myProxy instanceof MyClass); // true
```

---

## Step 753: Handler Trap - get

```javascript
// get trap: ดักจับการอ่าน property

// ตัวอย่าง 1: Default value
function withDefaults(obj, defaults) {
  return new Proxy(obj, {
    get(target, property) {
      if (property in target) {
        return target[property];
      }
      if (property in defaults) {
        return defaults[property];
      }
      return undefined;
    }
  });
}

const settings = withDefaults(
  { theme: "dark" },
  { theme: "light", language: "en", fontSize: 14 }
);

console.log(settings.theme);    // "dark" (override)
console.log(settings.language); // "en" (default)
console.log(settings.fontSize); // 14 (default)
console.log(settings.unknown);  // undefined
```

```javascript
// ตัวอย่าง 2: Lazy loading
function lazyLoad(loaders) {
  const cache = {};
  
  return new Proxy({}, {
    get(target, property) {
      if (property in cache) {
        return cache[property];
      }
      
      if (property in loaders) {
        console.log(`Loading: ${property}`);
        cache[property] = loaders[property]();
        return cache[property];
      }
      
      return undefined;
    }
  });
}

const heavyModules = lazyLoad({
  database: () => ({ connect: () => "connected" }),
  cache: () => ({ get: (k) => `value_${k}` }),
  mailer: () => ({ send: (to) => `sent to ${to}` }),
});

// ยังไม่ load อะไรเลย
console.log("Modules defined");

// Load เมื่อต้องการ
heavyModules.database.connect(); // Loading: database
heavyModules.database.connect(); // (ไม่ load ซ้ำ เพราะ cache)
```

```javascript
// ตัวอย่าง 3: Chained property access ที่ปลอดภัย
function safeAccess(obj) {
  return new Proxy(obj ?? {}, {
    get(target, property) {
      const value = target[property];
      
      if (value === null || value === undefined) {
        return safeAccess(null); // คืน proxy ของ null
      }
      
      if (typeof value === "object") {
        return safeAccess(value);
      }
      
      return value;
    }
  });
}

const data = {
  user: {
    profile: {
      name: "สมชาย",
    }
  }
};

const safe = safeAccess(data);
console.log(safe.user.profile.name);     // "สมชาย"
console.log(safe.user.address.city);     // {} (ไม่ error)
console.log(safe.nonexistent.deep.path); // {} (ไม่ error)
```

---

## Step 754: Handler Trap - set

```javascript
// set trap: ดักจับการ assign ค่า

// ตัวอย่าง 1: Type validation
function createTypedObject(schema) {
  const obj = {};
  
  return new Proxy(obj, {
    set(target, property, value) {
      const type = schema[property];
      
      if (type && typeof value !== type) {
        throw new TypeError(
          `Property "${property}" must be of type "${type}", got "${typeof value}"`
        );
      }
      
      target[property] = value;
      return true;
    }
  });
}

const person = createTypedObject({
  name: "string",
  age: "number",
  active: "boolean",
});

person.name = "สมชาย"; // OK
person.age = 30;        // OK
person.active = true;   // OK

try {
  person.age = "thirty"; // TypeError!
} catch (e) {
  console.log(e.message);
}
```

```javascript
// ตัวอย่าง 2: Observable (Reactive) Object
function makeObservable(obj) {
  const observers = new Map();
  
  const proxy = new Proxy(obj, {
    set(target, property, value) {
      const oldValue = target[property];
      target[property] = value;
      
      if (oldValue !== value) {
        const listeners = observers.get(property) || [];
        listeners.forEach((fn) => fn(value, oldValue, property));
      }
      
      return true;
    }
  });
  
  proxy.watch = function(property, callback) {
    if (!observers.has(property)) {
      observers.set(property, []);
    }
    observers.get(property).push(callback);
    
    return () => {
      const list = observers.get(property);
      const idx = list.indexOf(callback);
      if (idx > -1) list.splice(idx, 1);
    };
  };
  
  return proxy;
}

const state = makeObservable({ count: 0, name: "initial" });

const unwatch = state.watch("count", (newVal, oldVal) => {
  console.log(`count changed: ${oldVal} -> ${newVal}`);
});

state.count = 1; // count changed: 0 -> 1
state.count = 2; // count changed: 1 -> 2
state.count = 2; // (ไม่มี notification เพราะค่าไม่เปลี่ยน)

unwatch(); // หยุด watch
state.count = 3; // (ไม่มี notification)
```

```javascript
// ตัวอย่าง 3: Immutable object
function immutable(obj) {
  return new Proxy(obj, {
    set(target, property) {
      throw new TypeError(`Cannot set property "${property}" on immutable object`);
    },
    
    deleteProperty(target, property) {
      throw new TypeError(`Cannot delete property "${property}" on immutable object`);
    },
    
    get(target, property) {
      const value = target[property];
      if (typeof value === "object" && value !== null) {
        return immutable(value); // nested objects ก็ immutable
      }
      return value;
    }
  });
}

const frozen = immutable({ x: 1, nested: { y: 2 } });
console.log(frozen.x); // 1
try {
  frozen.x = 2; // TypeError
} catch (e) {
  console.log(e.message);
}
```

---

## Step 755: Handler Trap - has

```javascript
// has trap: ดักจับ 'in' operator

// ตัวอย่าง 1: Range check
function inRange(min, max) {
  return new Proxy({}, {
    has(target, value) {
      const num = Number(value);
      return !isNaN(num) && num >= min && num <= max;
    }
  });
}

const teenRange = inRange(13, 19);
console.log(13 in teenRange); // true
console.log(25 in teenRange); // false
console.log(19 in teenRange); // true
console.log(12 in teenRange); // false

// ใช้ใน conditional
const age = 16;
if (age in teenRange) {
  console.log("เป็นวัยรุ่น");
}
```

```javascript
// ตัวอย่าง 2: Case-insensitive property check
function caseInsensitive(obj) {
  return new Proxy(obj, {
    has(target, property) {
      return Object.keys(target).some(
        (k) => k.toLowerCase() === property.toLowerCase()
      );
    },
    
    get(target, property) {
      const key = Object.keys(target).find(
        (k) => k.toLowerCase() === property.toLowerCase()
      );
      return key ? target[key] : undefined;
    }
  });
}

const headers = caseInsensitive({
  "Content-Type": "application/json",
  "Authorization": "Bearer token123",
});

console.log("content-type" in headers); // true
console.log("AUTHORIZATION" in headers); // true
console.log(headers["content-type"]);    // "application/json"
```

---

## Step 756: Handler Trap - deleteProperty

```javascript
// deleteProperty trap: ดักจับ delete operator

// ตัวอย่าง 1: ป้องกันการลบ required properties
function protectedObject(obj, required = []) {
  return new Proxy(obj, {
    deleteProperty(target, property) {
      if (required.includes(property)) {
        throw new Error(`Cannot delete required property: ${property}`);
      }
      delete target[property];
      return true;
    }
  });
}

const config = protectedObject(
  { host: "localhost", port: 3000, optional: "value" },
  ["host", "port"]
);

delete config.optional;    // OK
try {
  delete config.host;       // Error!
} catch (e) {
  console.log(e.message);
}

console.log(config.host);     // "localhost" (ยังอยู่)
console.log(config.optional); // undefined (ถูกลบ)
```

```javascript
// ตัวอย่าง 2: Audit log สำหรับ deletions
function withDeletionLog(obj) {
  const deletionLog = [];
  
  const proxy = new Proxy(obj, {
    deleteProperty(target, property) {
      const value = target[property];
      deletionLog.push({
        property,
        value,
        deletedAt: new Date().toISOString(),
      });
      delete target[property];
      return true;
    }
  });
  
  proxy.getDeletionLog = () => [...deletionLog];
  
  return proxy;
}

const data = withDeletionLog({ a: 1, b: 2, c: 3 });
delete data.a;
delete data.c;

console.log(data.getDeletionLog());
// [{ property: 'a', value: 1, deletedAt: ... }, { property: 'c', value: 3, ... }]
```

---

## Step 757: Handler Trap - apply

```javascript
// apply trap: ดักจับการเรียก function

// ตัวอย่าง 1: Function timing
function timed(fn, label = fn.name) {
  return new Proxy(fn, {
    apply(target, thisArg, args) {
      const start = performance.now();
      const result = Reflect.apply(target, thisArg, args);
      const end = performance.now();
      console.log(`${label} took ${(end - start).toFixed(2)}ms`);
      return result;
    }
  });
}

function fibonacci(n) {
  if (n <= 1) return n;
  return fibonacci(n - 1) + fibonacci(n - 2);
}

const timedFib = timed(fibonacci, "fibonacci");
timedFib(30); // fibonacci took 12.34ms
```

```javascript
// ตัวอย่าง 2: Function call logging
function logged(fn) {
  return new Proxy(fn, {
    apply(target, thisArg, args) {
      console.log(`Calling: ${target.name}(${args.map(JSON.stringify).join(", ")})`);
      const result = Reflect.apply(target, thisArg, args);
      console.log(`Result: ${JSON.stringify(result)}`);
      return result;
    }
  });
}

const add = logged(function add(a, b) { return a + b; });
add(3, 4); // Calling: add(3, 4) -> Result: 7
```

```javascript
// ตัวอย่าง 3: Curried function
function curry(fn) {
  const arity = fn.length;
  
  function curried(...args) {
    if (args.length >= arity) {
      return fn.apply(this, args);
    }
    
    return new Proxy(curried, {
      apply(target, thisArg, newArgs) {
        return curried(...args, ...newArgs);
      }
    });
  }
  
  return curried;
}

const curriedAdd = curry((a, b, c) => a + b + c);
console.log(curriedAdd(1)(2)(3));   // 6
console.log(curriedAdd(1, 2)(3));   // 6
console.log(curriedAdd(1)(2, 3));   // 6
console.log(curriedAdd(1, 2, 3));   // 6
```

---

## Step 758: Handler Trap - construct

```javascript
// construct trap: ดักจับ new operator

// ตัวอย่าง 1: Singleton pattern
function singleton(Class) {
  let instance = null;
  
  return new Proxy(Class, {
    construct(target, args) {
      if (!instance) {
        instance = new target(...args);
      }
      return instance;
    }
  });
}

class Database {
  constructor(url) {
    this.url = url;
    console.log(`Connecting to: ${url}`);
  }
  
  query(sql) {
    return `Results from ${this.url}: ${sql}`;
  }
}

const SingletonDB = singleton(Database);

const db1 = new SingletonDB("mongodb://localhost");
const db2 = new SingletonDB("mongodb://remote");   // ไม่สร้างใหม่

console.log(db1 === db2);    // true (instance เดียวกัน)
console.log(db2.url);        // "mongodb://localhost" (instance แรก)
```

```javascript
// ตัวอย่าง 2: Instance counting / tracking
function tracked(Class) {
  let instanceCount = 0;
  const instances = new WeakSet();
  
  const TrackedClass = new Proxy(Class, {
    construct(target, args) {
      const instance = new target(...args);
      instanceCount++;
      instances.add(instance);
      console.log(`Created instance #${instanceCount} of ${Class.name}`);
      return instance;
    }
  });
  
  TrackedClass.getCount = () => instanceCount;
  
  return TrackedClass;
}

const TrackedUser = tracked(class User {
  constructor(name) {
    this.name = name;
  }
});

new TrackedUser("สมชาย"); // Created instance #1 of User
new TrackedUser("สมศรี"); // Created instance #2 of User
new TrackedUser("วิภา");  // Created instance #3 of User

console.log(TrackedUser.getCount()); // 3
```

---

## Step 759: Revocable Proxies

```javascript
// Proxy.revocable() สร้าง proxy ที่ revoke ได้

const { proxy, revoke } = Proxy.revocable(
  { name: "สมชาย", age: 30 },
  {
    get(target, property) {
      console.log(`Getting: ${property}`);
      return target[property];
    }
  }
);

console.log(proxy.name);  // Getting: name -> "สมชาย"
console.log(proxy.age);   // Getting: age -> 30

// Revoke proxy
revoke();

try {
  console.log(proxy.name); // TypeError: Cannot perform 'get' on a proxy that has been revoked
} catch (e) {
  console.log(e.message);
}
```

```javascript
// Use Case: Temporary access
function createTempAccess(data, durationMs) {
  const { proxy, revoke } = Proxy.revocable(data, {
    get(target, property) {
      return target[property];
    }
  });
  
  // Auto-revoke หลังจาก duration
  setTimeout(() => {
    console.log("Access expired!");
    revoke();
  }, durationMs);
  
  return proxy;
}

const tempData = createTempAccess(
  { secret: "s3cr3t", public: "visible" },
  2000 // 2 seconds
);

console.log(tempData.secret); // "s3cr3t"
setTimeout(() => {
  try {
    console.log(tempData.secret);
  } catch (e) {
    console.log("Access denied:", e.message);
  }
}, 3000);
```

---

## Step 760: Reflect API

```javascript
// Reflect มี static methods ที่ correspond กับ Proxy traps

// Reflect.get - เหมือน obj[key]
const obj = { name: "สมชาย", age: 30 };
console.log(Reflect.get(obj, "name")); // "สมชาย"

// Reflect.set - เหมือน obj[key] = value
Reflect.set(obj, "age", 31);
console.log(obj.age); // 31

// Reflect.has - เหมือน key in obj
console.log(Reflect.has(obj, "name"));  // true
console.log(Reflect.has(obj, "email")); // false

// Reflect.deleteProperty - เหมือน delete obj[key]
Reflect.deleteProperty(obj, "age");
console.log(obj.age); // undefined

// Reflect.ownKeys - รวม Symbol keys
const sym = Symbol("sym");
const obj2 = { a: 1, b: 2, [sym]: 3 };
console.log(Reflect.ownKeys(obj2)); // ['a', 'b', Symbol(sym)]

// Reflect.apply - เหมือน fn.apply(thisArg, args)
function sum(...args) {
  return args.reduce((a, b) => a + b, 0);
}
console.log(Reflect.apply(sum, null, [1, 2, 3, 4, 5])); // 15

// Reflect.construct - เหมือน new Constructor(...args)
class Point {
  constructor(x, y) {
    this.x = x;
    this.y = y;
  }
}
const point = Reflect.construct(Point, [3, 4]);
console.log(point); // Point { x: 3, y: 4 }

// Reflect.defineProperty
Reflect.defineProperty(obj2, "c", {
  value: 42,
  writable: false,
  enumerable: true,
  configurable: false
});
console.log(obj2.c); // 42

// Reflect.getOwnPropertyDescriptor
const desc = Reflect.getOwnPropertyDescriptor(obj2, "c");
console.log(desc); // { value: 42, writable: false, ... }

// Reflect.getPrototypeOf / setPrototypeOf
const proto = { greet() { return "Hello"; } };
const child = Object.create(proto);
console.log(Reflect.getPrototypeOf(child) === proto); // true

// Reflect.isExtensible / preventExtensions
console.log(Reflect.isExtensible(obj2)); // true
Reflect.preventExtensions(obj2);
console.log(Reflect.isExtensible(obj2)); // false
```

---

## Step 761: Reflect Methods Mirror Proxy Traps

```javascript
// Reflect methods คู่กันกับ Proxy traps ทุกตัว

const TRAPS_AND_REFLECT = {
  "get trap": "Reflect.get(target, property, receiver)",
  "set trap": "Reflect.set(target, property, value, receiver)",
  "has trap": "Reflect.has(target, property)",
  "deleteProperty trap": "Reflect.deleteProperty(target, property)",
  "apply trap": "Reflect.apply(target, thisArg, argumentsList)",
  "construct trap": "Reflect.construct(target, argumentsList, newTarget)",
  "defineProperty trap": "Reflect.defineProperty(target, property, descriptor)",
  "getOwnPropertyDescriptor trap": "Reflect.getOwnPropertyDescriptor(target, property)",
  "getPrototypeOf trap": "Reflect.getPrototypeOf(target)",
  "setPrototypeOf trap": "Reflect.setPrototypeOf(target, prototype)",
  "isExtensible trap": "Reflect.isExtensible(target)",
  "preventExtensions trap": "Reflect.preventExtensions(target)",
  "ownKeys trap": "Reflect.ownKeys(target)",
};

console.log("Proxy trap to Reflect API mapping complete");
```

---

## Step 762: ทำไมต้องใช้ Reflect ใน Proxy Traps

```javascript
// ปัญหาถ้าไม่ใช้ Reflect - prototype chain อาจพัง

class Animal {
  constructor(name) {
    this.name = name;
  }
  
  speak() {
    return `${this.name} makes a sound`;
  }
}

class Dog extends Animal {
  speak() {
    return `${this.name} barks`;
  }
}

// ❌ ไม่ดี - ไม่ใช้ Reflect
const badProxy = new Proxy(new Dog("Rex"), {
  get(target, property) {
    return target[property]; // อาจมีปัญหากับ getter ที่ใช้ this
  }
});

// ✅ ดี - ใช้ Reflect ที่ส่ง receiver ถูกต้อง
const goodProxy = new Proxy(new Dog("Rex"), {
  get(target, property, receiver) {
    return Reflect.get(target, property, receiver);
    // receiver = proxy ทำให้ this ใน getter ถูกต้อง
  }
});
```

```javascript
// ตัวอย่างที่แสดงความสำคัญของ receiver

const parent = {
  get value() {
    return this._value;
  },
  set value(v) {
    this._value = v;
  }
};

const child = Object.create(parent);
child._value = 42;

// ❌ ไม่ดี
const badProxy2 = new Proxy(child, {
  get(target, property) {
    return target[property]; // getter ของ parent จะใช้ target (child) ไม่ใช่ proxy
  }
});

// ✅ ดี
const goodProxy2 = new Proxy(child, {
  get(target, property, receiver) {
    return Reflect.get(target, property, receiver); // receiver = proxy
  }
});

console.log(goodProxy2.value); // 42 (ถูกต้อง)
```

---

## Step 763: Practical Proxy Use Case - Validation

```javascript
// Validation Proxy: ตรวจสอบ type และ constraints

function createValidatedObject(schema) {
  const validators = {
    string: (v) => typeof v === "string",
    number: (v) => typeof v === "number" && !isNaN(v),
    boolean: (v) => typeof v === "boolean",
    email: (v) => typeof v === "string" && /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(v),
    positiveNumber: (v) => typeof v === "number" && v > 0,
    nonEmptyString: (v) => typeof v === "string" && v.trim().length > 0,
  };
  
  const obj = {};
  
  return new Proxy(obj, {
    set(target, property, value) {
      const rule = schema[property];
      
      if (!rule) {
        throw new Error(`Property "${property}" is not in schema`);
      }
      
      const { type, required, min, max, minLength, maxLength } = rule;
      
      // Type check
      if (type && !validators[type]?.(value)) {
        throw new TypeError(`"${property}" must be of type "${type}", got "${typeof value}"`);
      }
      
      // Number range
      if (min !== undefined && value < min) {
        throw new RangeError(`"${property}" must be >= ${min}, got ${value}`);
      }
      if (max !== undefined && value > max) {
        throw new RangeError(`"${property}" must be <= ${max}, got ${value}`);
      }
      
      // String length
      if (minLength !== undefined && value.length < minLength) {
        throw new RangeError(`"${property}" must have length >= ${minLength}`);
      }
      if (maxLength !== undefined && value.length > maxLength) {
        throw new RangeError(`"${property}" must have length <= ${maxLength}`);
      }
      
      Reflect.set(target, property, value);
      return true;
    }
  });
}

const user = createValidatedObject({
  name: { type: "nonEmptyString", minLength: 2, maxLength: 50 },
  age: { type: "positiveNumber", min: 1, max: 150 },
  email: { type: "email" },
});

user.name = "สมชาย";
user.age = 30;
user.email = "somchai@example.com";

console.log(user.name);  // "สมชาย"
console.log(user.age);   // 30
console.log(user.email); // "somchai@example.com"

try {
  user.age = -5; // RangeError
} catch (e) {
  console.log(e.message);
}

try {
  user.email = "invalid"; // TypeError
} catch (e) {
  console.log(e.message);
}
```

---

## Step 764: Practical Proxy Use Case - Observable Objects

```javascript
// Full reactive system

class Reactive {
  static #watchers = new WeakMap();
  static #currentWatcher = null;
  
  static makeReactive(obj) {
    const deps = new Map(); // property -> Set of watchers
    
    return new Proxy(obj, {
      get(target, property, receiver) {
        // Track dependency ถ้ามี watcher กำลัง run อยู่
        if (Reactive.#currentWatcher) {
          if (!deps.has(property)) {
            deps.set(property, new Set());
          }
          deps.get(property).add(Reactive.#currentWatcher);
        }
        
        const value = Reflect.get(target, property, receiver);
        
        // ถ้าเป็น object ทำให้ reactive ด้วย
        if (value && typeof value === "object") {
          return Reactive.makeReactive(value);
        }
        
        return value;
      },
      
      set(target, property, value, receiver) {
        const result = Reflect.set(target, property, value, receiver);
        
        // Notify watchers
        const watchers = deps.get(property);
        if (watchers) {
          watchers.forEach((watcher) => watcher());
        }
        
        return result;
      }
    });
  }
  
  static watch(fn) {
    const watcher = () => {
      Reactive.#currentWatcher = watcher;
      fn();
      Reactive.#currentWatcher = null;
    };
    
    watcher();
    return watcher;
  }
  
  static computed(fn) {
    let cachedValue;
    let dirty = true;
    
    Reactive.watch(() => {
      dirty = true;
      cachedValue = fn();
    });
    
    return {
      get value() {
        if (dirty) {
          cachedValue = fn();
          dirty = false;
        }
        return cachedValue;
      }
    };
  }
}

// ทดสอบ
const state = Reactive.makeReactive({
  firstName: "สมชาย",
  lastName: "ใจดี",
  age: 30,
});

// Watch: run ทันทีและ re-run เมื่อ dependencies เปลี่ยน
Reactive.watch(() => {
  console.log(`Name: ${state.firstName} ${state.lastName}`);
});
// Name: สมชาย ใจดี

state.firstName = "สมหญิง";
// Name: สมหญิง ใจดี

state.lastName = "สวยงาม";
// Name: สมหญิง สวยงาม
```

---

## Step 765: Practical Proxy Use Case - Lazy Initialization

```javascript
// Lazy initialization: สร้าง heavy objects เมื่อต้องการ

function lazyObject(factory) {
  let instance = null;
  let initialized = false;
  
  return new Proxy({}, {
    get(_, property) {
      if (!initialized) {
        instance = factory();
        initialized = true;
        console.log("Instance created");
      }
      return Reflect.get(instance, property, instance);
    },
    
    set(_, property, value) {
      if (!initialized) {
        instance = factory();
        initialized = true;
      }
      return Reflect.set(instance, property, value);
    }
  });
}

class HeavyService {
  constructor() {
    console.log("HeavyService constructor called (expensive!)");
    this.data = new Array(1000).fill(0);
  }
  
  process(input) {
    return `Processed: ${input}`;
  }
}

const lazyService = lazyObject(() => new HeavyService());

console.log("Service defined but not yet created");
// ยังไม่มีการสร้าง HeavyService

const result = lazyService.process("hello");
// HeavyService constructor called (expensive!)
// Instance created

console.log(result); // "Processed: hello"
console.log(lazyService.process("world")); // (ไม่มีการสร้างใหม่)
```

---

## Step 766: Practical Proxy Use Case - Logging

```javascript
// Deep logging proxy
function createLoggingProxy(obj, label = "Object") {
  function makeProxy(target, path) {
    return new Proxy(target, {
      get(t, property, receiver) {
        const value = Reflect.get(t, property, receiver);
        const fullPath = path ? `${path}.${String(property)}` : String(property);
        
        if (typeof value === "function") {
          return new Proxy(value, {
            apply(fn, thisArg, args) {
              console.log(`[${label}] Calling ${fullPath}(${args.map(JSON.stringify).join(", ")})`);
              const result = Reflect.apply(fn, thisArg, args);
              console.log(`[${label}] ${fullPath} returned: ${JSON.stringify(result)}`);
              return result;
            }
          });
        }
        
        if (value && typeof value === "object") {
          console.log(`[${label}] Getting ${fullPath} (object)`);
          return makeProxy(value, fullPath);
        }
        
        console.log(`[${label}] Getting ${fullPath} = ${JSON.stringify(value)}`);
        return value;
      },
      
      set(t, property, value) {
        const fullPath = path ? `${path}.${String(property)}` : String(property);
        console.log(`[${label}] Setting ${fullPath} = ${JSON.stringify(value)}`);
        return Reflect.set(t, property, value);
      }
    });
  }
  
  return makeProxy(obj, "");
}

const api = createLoggingProxy({
  user: {
    name: "สมชาย",
    getFullName() {
      return `Dr. ${this.name}`;
    }
  }
}, "API");

const name = api.user.name;
api.user.name = "สมหญิง";
api.user.getFullName();
```

---

## Step 767: Practical Proxy Use Case - Caching

```javascript
// Memoization Proxy สำหรับ methods

function memoizeProxy(obj) {
  const cache = new WeakMap();
  
  return new Proxy(obj, {
    get(target, property, receiver) {
      const value = Reflect.get(target, property, receiver);
      
      if (typeof value !== "function") return value;
      
      // สร้าง memoized version ของ method
      return new Proxy(value, {
        apply(fn, thisArg, args) {
          if (!cache.has(fn)) {
            cache.set(fn, new Map());
          }
          
          const fnCache = cache.get(fn);
          const key = JSON.stringify(args);
          
          if (fnCache.has(key)) {
            console.log(`Cache hit for ${property}(${args})`);
            return fnCache.get(key);
          }
          
          const result = Reflect.apply(fn, thisArg, args);
          fnCache.set(key, result);
          console.log(`Computed ${property}(${args}) = ${result}`);
          return result;
        }
      });
    }
  });
}

const calculator = memoizeProxy({
  factorial(n) {
    if (n <= 1) return 1;
    return n * this.factorial(n - 1);
  },
  
  fibonacci(n) {
    if (n <= 1) return n;
    return this.fibonacci(n - 1) + this.fibonacci(n - 2);
  }
});

calculator.factorial(5); // คำนวณ
calculator.factorial(5); // Cache hit
calculator.fibonacci(10); // คำนวณ
```

---

## Step 768: Proxy กับ Class

```javascript
// Proxy กับ class เพื่อสร้าง DAO pattern

function createDAO(tableName, db) {
  const proto = {
    async save() {
      const id = this.id;
      const data = { ...this };
      delete data.id;
      
      if (id) {
        await db.update(tableName, id, data);
        console.log(`Updated ${tableName}:${id}`);
      } else {
        this.id = await db.insert(tableName, data);
        console.log(`Inserted ${tableName}:${this.id}`);
      }
      
      return this;
    },
    
    async delete() {
      if (!this.id) throw new Error("Cannot delete unsaved record");
      await db.delete(tableName, this.id);
      console.log(`Deleted ${tableName}:${this.id}`);
    }
  };
  
  return new Proxy(function() {}, {
    construct(_, args) {
      const instance = Object.create(proto);
      Object.assign(instance, ...args);
      return instance;
    },
    
    get(target, property) {
      // Static methods
      if (property === "find") {
        return async (id) => {
          const data = await db.find(tableName, id);
          if (!data) return null;
          const instance = Object.create(proto);
          return Object.assign(instance, data);
        };
      }
      
      if (property === "findAll") {
        return async (where = {}) => {
          const records = await db.findAll(tableName, where);
          return records.map((data) => {
            const instance = Object.create(proto);
            return Object.assign(instance, data);
          });
        };
      }
      
      return Reflect.get(target, property);
    }
  });
}

// Mock DB
const mockDB = {
  _data: new Map(),
  _nextId: 1,
  
  async insert(table, data) {
    const id = this._nextId++;
    this._data.set(`${table}:${id}`, { id, ...data });
    return id;
  },
  
  async update(table, id, data) {
    const key = `${table}:${id}`;
    const existing = this._data.get(key);
    this._data.set(key, { ...existing, ...data });
  },
  
  async find(table, id) {
    return this._data.get(`${table}:${id}`) || null;
  },
  
  async findAll(table, where) {
    const results = [];
    for (const [key, value] of this._data) {
      if (key.startsWith(`${table}:`)) {
        const matches = Object.entries(where).every(([k, v]) => value[k] === v);
        if (matches || Object.keys(where).length === 0) {
          results.push(value);
        }
      }
    }
    return results;
  },
};

const User = createDAO("users", mockDB);

async function demo() {
  const user = new User({ name: "สมชาย", email: "somchai@example.com" });
  await user.save(); // Inserted users:1
  
  user.name = "สมชาย (แก้ไข)";
  await user.save(); // Updated users:1
  
  const found = await User.find(1);
  console.log(found.name); // "สมชาย (แก้ไข)"
  
  const all = await User.findAll();
  console.log(all.length); // 1
}

demo();
```

---

## Step 769: Meta-programming Patterns

```javascript
// Pattern 1: Auto-binding
function autoBind(obj) {
  return new Proxy(obj, {
    get(target, property, receiver) {
      const value = Reflect.get(target, property, receiver);
      
      if (typeof value === "function") {
        return value.bind(target);
      }
      
      return value;
    }
  });
}

class EventHandler {
  constructor() {
    this.count = 0;
  }
  
  handleClick() {
    this.count++;
    console.log(`Click count: ${this.count}`);
  }
}

const handler = autoBind(new EventHandler());
const { handleClick } = handler; // extract method
handleClick(); // Click count: 1 (ยังทำงานได้ เพราะ bound)
handleClick(); // Click count: 2
```

```javascript
// Pattern 2: Method chaining สำหรับ query builder

function createQueryBuilder(baseQuery = {}) {
  const query = { ...baseQuery };
  
  return new Proxy(query, {
    get(target, method) {
      if (method === "build") {
        return () => ({ ...target });
      }
      
      if (method === "execute") {
        return () => {
          console.log("Executing query:", JSON.stringify(target, null, 2));
          // ทำ actual query
          return Promise.resolve([]);
        };
      }
      
      return (...args) => {
        if (method === "where") {
          target.conditions = [
            ...(target.conditions || []),
            ...args
          ];
        } else if (method === "limit") {
          target.limit = args[0];
        } else if (method === "offset") {
          target.offset = args[0];
        } else if (method === "orderBy") {
          target.orderBy = { field: args[0], direction: args[1] || "ASC" };
        } else if (method === "select") {
          target.fields = args;
        }
        
        return createQueryBuilder(target);
      };
    }
  });
}

const query = createQueryBuilder()
  .select("id", "name", "email")
  .where("age > 18")
  .where("active = true")
  .orderBy("name", "DESC")
  .limit(10)
  .offset(20);

console.log(query.build());
```

---

## Step 770: ตัวอย่างขั้นสูง - Complete Meta-programming System

```javascript
// Class decorator system ด้วย Proxy

function sealed(Class) {
  return new Proxy(Class, {
    construct(target, args) {
      const instance = new target(...args);
      return Object.seal(instance);
    }
  });
}

function readonly(Class) {
  return new Proxy(Class, {
    construct(target, args) {
      const instance = new target(...args);
      return new Proxy(instance, {
        set(obj, property, value) {
          if (obj.hasOwnProperty(property)) {
            throw new TypeError(`Cannot assign to read-only property: ${property}`);
          }
          return Reflect.set(obj, property, value);
        }
      });
    }
  });
}

function logged(Class) {
  return new Proxy(Class, {
    construct(target, args) {
      console.log(`Creating ${target.name}`);
      const instance = new target(...args);
      
      return new Proxy(instance, {
        get(obj, property) {
          const value = obj[property];
          if (typeof value === "function") {
            return new Proxy(value, {
              apply(fn, thisArg, fnArgs) {
                console.log(`${target.name}.${property}(${fnArgs.map(JSON.stringify).join(", ")})`);
                return fn.apply(obj, fnArgs);
              }
            });
          }
          return value;
        }
      });
    }
  });
}

function compose(...decorators) {
  return function(Class) {
    return decorators.reduce((C, d) => d(C), Class);
  };
}

// ใช้งาน
const decoratedUser = compose(
  logged,
  sealed
)(class User {
  constructor(name, email) {
    this.name = name;
    this.email = email;
  }
  
  greet() {
    return `Hello, I'm ${this.name}`;
  }
  
  update(data) {
    Object.assign(this, data);
    return this;
  }
});

const user = new decoratedUser("สมชาย", "somchai@example.com");
// Creating User

console.log(user.greet());
// User.greet()
// "Hello, I'm สมชาย"

user.update({ name: "สมหญิง" });
// User.update({"name":"สมหญิง"})
console.log(user.name); // "สมหญิง"
```

---

## แบบฝึกหัด

### Easy
1. สร้าง Proxy ที่ทำให้ทุก property access เป็น case-insensitive
2. สร้าง logging proxy ที่แสดง log ทุก operation
3. สร้าง Proxy ที่ไม่ให้เพิ่ม property ใหม่

### Medium
4. สร้าง `observable(obj)` ที่ใช้ Proxy เพื่อ watch changes
5. Implement `Validator` class ที่ใช้ Proxy สำหรับ form validation
6. สร้าง API mock ด้วย Proxy ที่ intercept method calls

### Hard
7. Implement reactive data binding ระหว่าง 2 objects
8. สร้าง permission system โดยใช้ Proxy ที่ control access
9. Implement JSON Schema validator โดยใช้ Proxy

### Solution ตัวอย่าง

```javascript
// 1. Case-insensitive Proxy
function caseInsensitiveProxy(obj) {
  function findKey(target, property) {
    if (typeof property !== "string") return property;
    
    // ลองหา exact match ก่อน
    if (property in target) return property;
    
    // หา case-insensitive match
    const lower = property.toLowerCase();
    return Object.keys(target).find((k) => k.toLowerCase() === lower) || property;
  }
  
  return new Proxy(obj, {
    get(target, property, receiver) {
      return Reflect.get(target, findKey(target, property), receiver);
    },
    
    set(target, property, value, receiver) {
      return Reflect.set(target, findKey(target, property), value, receiver);
    },
    
    has(target, property) {
      return Reflect.has(target, findKey(target, property));
    },
    
    deleteProperty(target, property) {
      return Reflect.deleteProperty(target, findKey(target, property));
    }
  });
}

const headers = caseInsensitiveProxy({
  "Content-Type": "application/json",
  "Authorization": "Bearer token",
});

console.log(headers["content-type"]);   // "application/json"
console.log(headers["AUTHORIZATION"]); // "Bearer token"
console.log("authorization" in headers); // true

// 4. Observable
function observable(obj) {
  const handlers = new Map();
  
  const proxy = new Proxy(obj, {
    set(target, property, value) {
      const old = target[property];
      Reflect.set(target, property, value);
      
      if (old !== value) {
        (handlers.get(property) || []).forEach((fn) => fn(value, old));
        (handlers.get("*") || []).forEach((fn) => fn(property, value, old));
      }
      return true;
    }
  });
  
  proxy.$watch = (property, fn) => {
    if (!handlers.has(property)) handlers.set(property, []);
    handlers.get(property).push(fn);
    return () => {
      const arr = handlers.get(property);
      const i = arr.indexOf(fn);
      if (i > -1) arr.splice(i, 1);
    };
  };
  
  return proxy;
}

const state = observable({ x: 1, y: 2 });

state.$watch("x", (newVal, oldVal) => {
  console.log(`x: ${oldVal} -> ${newVal}`);
});

state.$watch("*", (prop, newVal, oldVal) => {
  console.log(`Any change: ${prop} ${oldVal} -> ${newVal}`);
});

state.x = 10;
// x: 1 -> 10
// Any change: x 1 -> 10

state.y = 20;
// Any change: y 2 -> 20
```

---

## สรุป

| Proxy Trap | Operation | Reflect Equivalent |
|------------|-----------|-------------------|
| `get` | `obj.prop` | `Reflect.get()` |
| `set` | `obj.prop = v` | `Reflect.set()` |
| `has` | `prop in obj` | `Reflect.has()` |
| `deleteProperty` | `delete obj.prop` | `Reflect.deleteProperty()` |
| `apply` | `fn()` | `Reflect.apply()` |
| `construct` | `new Fn()` | `Reflect.construct()` |
| `ownKeys` | `Object.keys()` | `Reflect.ownKeys()` |

**เมื่อไหร่ควรใช้ Proxy:**
- Validation / type checking
- Observable / reactive data
- Logging / debugging
- Caching / memoization
- Lazy initialization
- Access control / permissions
- API mocking

**ข้อควรระวัง:**
- Proxy มี performance overhead
- บาง built-in operations ไม่ผ่าน Proxy (เช่น private class fields)
- ใช้ `Reflect` เสมอใน traps เพื่อรักษา correct `this` binding
