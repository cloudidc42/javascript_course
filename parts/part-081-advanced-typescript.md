# Part 81: Advanced TypeScript Patterns (ขั้นสูง)
## Steps 1591-1610

TypeScript ขั้นสูงช่วยให้เราเขียนโค้ดที่ type-safe มากขึ้น และสามารถแสดงความตั้งใจได้ชัดเจนยิ่งขึ้น
ในบทนี้เราจะเรียนรู้ pattern ต่างๆ ที่ใช้ในโปรเจกต์ระดับ production จริงๆ

---

## Step 1591: Advanced Generic Constraints (ข้อจำกัด Generic ขั้นสูง)

Generic constraints ช่วยให้เราระบุว่า type parameter ต้องมี property หรือ method อะไรบ้าง

```typescript
// ข้อจำกัดพื้นฐาน - T ต้องมี property length
function getLength<T extends { length: number }>(value: T): number {
  return value.length;
}

console.log(getLength("hello"));        // 5
console.log(getLength([1, 2, 3]));      // 3
console.log(getLength({ length: 10 })); // 10
// console.log(getLength(42));          // Error! number ไม่มี length

// Constraint หลายอย่าง
interface Printable {
  print(): void;
}

interface Serializable {
  serialize(): string;
}

function processItem<T extends Printable & Serializable>(item: T): string {
  item.print();
  return item.serialize();
}

// Constraint ด้วย keyof
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const person = { name: "สมชาย", age: 30, city: "กรุงเทพ" };
const name = getProperty(person, "name");   // type: string
const age = getProperty(person, "age");     // type: number
// getProperty(person, "email");            // Error! 'email' ไม่ใช่ key ของ person

// Constraint ด้วย constructor
function createInstance<T>(constructor: new () => T): T {
  return new constructor();
}

class Dog {
  name = "บัดดี้";
  bark() { return "โฮ่ง!"; }
}

const dog = createInstance(Dog);
console.log(dog.bark()); // "โฮ่ง!"

// Constructor พร้อม arguments
function createWithArgs<T>(
  constructor: new (...args: any[]) => T,
  ...args: any[]
): T {
  return new constructor(...args);
}

class Cat {
  constructor(public name: string, public age: number) {}
  meow() { return `${this.name} พูดว่า เมี้ยว!`; }
}

const cat = createWithArgs(Cat, "มิ้งค์", 3);
console.log(cat.meow()); // "มิ้งค์ พูดว่า เมี้ยว!"
```

---

## Step 1592: Generic Utility Function Patterns

```typescript
// Utility functions ที่ใช้บ่อยใน TypeScript
// 1. Safe object property access
function safeGet<T, K extends keyof T>(
  obj: T | null | undefined,
  key: K
): T[K] | undefined {
  return obj?.[key];
}

const user = { name: "สมหญิง", email: "som@example.com" };
const email = safeGet(user, "email");     // string | undefined
const nullEmail = safeGet(null, "email"); // undefined

// 2. Type-safe array filter with type guard
function filterByType<T, U extends T>(
  arr: T[],
  predicate: (item: T) => item is U
): U[] {
  return arr.filter(predicate);
}

type Shape = { kind: "circle"; radius: number } | { kind: "square"; side: number };

function isCircle(shape: Shape): shape is { kind: "circle"; radius: number } {
  return shape.kind === "circle";
}

const shapes: Shape[] = [
  { kind: "circle", radius: 5 },
  { kind: "square", side: 3 },
  { kind: "circle", radius: 8 },
];

const circles = filterByType(shapes, isCircle);
// circles: { kind: "circle"; radius: number }[]

// 3. Memoization พร้อม generic
function memoize<T extends (...args: any[]) => any>(fn: T): T {
  const cache = new Map<string, ReturnType<T>>();
  
  return ((...args: Parameters<T>): ReturnType<T> => {
    const key = JSON.stringify(args);
    if (cache.has(key)) {
      return cache.get(key)!;
    }
    const result = fn(...args);
    cache.set(key, result);
    return result;
  }) as T;
}

function expensiveCalculation(n: number): number {
  console.log(`คำนวณสำหรับ ${n}...`);
  return n * n;
}

const memoized = memoize(expensiveCalculation);
console.log(memoized(5));  // คำนวณ... 25
console.log(memoized(5));  // ดึงจาก cache: 25
console.log(memoized(10)); // คำนวณ... 100

// 4. Pipeline pattern
function pipe<T>(...fns: Array<(arg: T) => T>): (arg: T) => T {
  return (arg: T) => fns.reduce((acc, fn) => fn(acc), arg);
}

const processNumber = pipe(
  (n: number) => n * 2,
  (n: number) => n + 10,
  (n: number) => n / 2
);

console.log(processNumber(5)); // ((5*2)+10)/2 = 10
```

---

## Step 1593: Recursive Generic Types (Generic Types แบบเรียกซ้ำ)

```typescript
// Recursive type สำหรับ deep nested structures
type DeepReadonly<T> = {
  readonly [K in keyof T]: T[K] extends object ? DeepReadonly<T[K]> : T[K];
};

interface Config {
  server: {
    host: string;
    port: number;
    ssl: {
      enabled: boolean;
      cert: string;
    };
  };
  database: {
    url: string;
    maxConnections: number;
  };
}

type ReadonlyConfig = DeepReadonly<Config>;

const config: ReadonlyConfig = {
  server: {
    host: "localhost",
    port: 3000,
    ssl: { enabled: true, cert: "cert.pem" }
  },
  database: {
    url: "postgres://localhost/mydb",
    maxConnections: 10
  }
};

// config.server.host = "newhost";  // Error! readonly
// config.server.ssl.enabled = false; // Error! deeply readonly

// Recursive type สำหรับ JSON
type JSONValue =
  | string
  | number
  | boolean
  | null
  | JSONValue[]
  | { [key: string]: JSONValue };

const data: JSONValue = {
  name: "สมชาย",
  age: 30,
  scores: [95, 87, 92],
  address: {
    city: "กรุงเทพ",
    zip: "10100"
  }
};

// Deep partial type
type DeepPartial<T> = {
  [K in keyof T]?: T[K] extends object ? DeepPartial<T[K]> : T[K];
};

function updateConfig(
  current: Config,
  update: DeepPartial<Config>
): Config {
  return {
    server: {
      ...current.server,
      ...update.server,
      ssl: {
        ...current.server.ssl,
        ...update.server?.ssl
      }
    },
    database: {
      ...current.database,
      ...update.database
    }
  };
}

// Tree structure แบบ recursive
type TreeNode<T> = {
  value: T;
  children?: TreeNode<T>[];
};

function mapTree<T, U>(
  node: TreeNode<T>,
  fn: (value: T) => U
): TreeNode<U> {
  return {
    value: fn(node.value),
    children: node.children?.map(child => mapTree(child, fn))
  };
}

const numberTree: TreeNode<number> = {
  value: 1,
  children: [
    { value: 2, children: [{ value: 4 }, { value: 5 }] },
    { value: 3, children: [{ value: 6 }] }
  ]
};

const stringTree = mapTree(numberTree, n => `Node ${n}`);
console.log(stringTree.value); // "Node 1"
```

---

## Step 1594: Variadic Tuple Types (Tuple Types แบบ Variadic)

```typescript
// Variadic tuple types (TypeScript 4.0+)
// ทำให้ tuple สามารถ "spread" ได้

// Basic variadic tuple
type Concat<T extends unknown[], U extends unknown[]> = [...T, ...U];

type Result1 = Concat<[string, number], [boolean, Date]>;
// [string, number, boolean, Date]

// Prepend type to tuple
type Prepend<T, U extends unknown[]> = [T, ...U];
type NumberFirst = Prepend<number, [string, boolean]>;
// [number, string, boolean]

// Tail of tuple
type Tail<T extends unknown[]> = T extends [unknown, ...infer Rest] ? Rest : never;
type TailResult = Tail<[string, number, boolean]>;
// [number, boolean]

// Head of tuple
type Head<T extends unknown[]> = T extends [infer First, ...unknown[]] ? First : never;
type HeadResult = Head<[string, number, boolean]>;
// string

// Function argument spreading
function concat<T extends unknown[], U extends unknown[]>(
  arr1: [...T],
  arr2: [...U]
): [...T, ...U] {
  return [...arr1, ...arr2];
}

const result = concat([1, "hello", true], [42, new Date()]);
// type: [number, string, boolean, number, Date]

// Curry function with variadic tuples
type Curry<
  T extends unknown[],
  R
> = T extends []
  ? R
  : T extends [infer First, ...infer Rest]
  ? (arg: First) => Curry<Rest, R>
  : never;

// ตัวอย่าง partial application
function partial<T extends unknown[], R>(
  fn: (...args: T) => R,
  ...firstArgs: Partial<T>
): (...restArgs: any[]) => R {
  return (...restArgs: any[]) => fn(...(firstArgs as any), ...restArgs);
}

function add(a: number, b: number, c: number): number {
  return a + b + c;
}

const add5 = partial(add, 5);
console.log(add5(3, 2)); // 10

// Named tuple elements
type Range = [start: number, end: number, step?: number];

function createRange(range: Range): number[] {
  const [start, end, step = 1] = range;
  const result: number[] = [];
  for (let i = start; i < end; i += step) {
    result.push(i);
  }
  return result;
}

const range1 = createRange([0, 10]);        // [0,1,2,3,4,5,6,7,8,9]
const range2 = createRange([0, 10, 2]);     // [0,2,4,6,8]
```

---

## Step 1595: Template Literal Types

```typescript
// Template literal types (TypeScript 4.1+)
// สร้าง string types แบบ dynamic

// Basic template literal type
type Greeting = `Hello, ${string}!`;
const greeting: Greeting = "Hello, สมชาย!"; // OK
// const bad: Greeting = "Hi there";        // Error!

// Union distribution in template literals
type Color = "red" | "green" | "blue";
type Size = "small" | "medium" | "large";

type ColorSize = `${Color}-${Size}`;
// "red-small" | "red-medium" | "red-large" | "green-small" | ...

// Event naming convention
type EventName<T extends string> = `on${Capitalize<T>}`;
type ClickEvent = EventName<"click">;    // "onClick"
type HoverEvent = EventName<"hover">;   // "onHover"

// API endpoint generator
type APIEndpoint<T extends string> = `/api/v1/${T}`;
type UserEndpoint = APIEndpoint<"users">;    // "/api/v1/users"
type PostEndpoint = APIEndpoint<"posts">;    // "/api/v1/posts"

// Getters and setters
type GetterSetter<T extends string> = {
  [K in T as `get${Capitalize<K>}`]: () => string;
} & {
  [K in T as `set${Capitalize<K>}`]: (value: string) => void;
};

type PersonAccessors = GetterSetter<"name" | "email" | "phone">;
// {
//   getName: () => string;
//   setName: (value: string) => void;
//   getEmail: () => string;
//   setEmail: (value: string) => void;
//   getPhone: () => string;
//   setPhone: (value: string) => void;
// }

// CSS property names
type CSSProperty = "margin" | "padding" | "border";
type CSSDirection = "top" | "right" | "bottom" | "left";
type CSSWithDirection = `${CSSProperty}-${CSSDirection}`;
// "margin-top" | "margin-right" | ... | "border-left"

// Event handler extraction
function createEventHandlers<T extends string>(events: T[]): {
  [K in T as `on${Capitalize<K>}`]: (handler: () => void) => void;
} {
  const handlers = {} as any;
  for (const event of events) {
    const key = `on${event.charAt(0).toUpperCase()}${event.slice(1)}`;
    handlers[key] = (handler: () => void) => {
      document.addEventListener(event, handler);
    };
  }
  return handlers;
}

// Type-safe SQL query builder (concept)
type TableName = "users" | "posts" | "comments";
type SelectQuery<T extends TableName> = `SELECT * FROM ${T}`;
type InsertQuery<T extends TableName> = `INSERT INTO ${T} VALUES (?)`;

type UserSelect = SelectQuery<"users">;  // "SELECT * FROM users"
```

---

## Step 1596: Mapped Type Modifiers (+/-)

```typescript
// Mapped type modifiers: + เพิ่ม, - ลบ

// เพิ่ม readonly และ optional
type Freeze<T> = {
  +readonly [K in keyof T]+?: T[K];
};

// ลบ readonly และ optional (ทำให้ required)
type Mutable<T> = {
  -readonly [K in keyof T]-?: T[K];
};

type Required<T> = {
  [K in keyof T]-?: T[K];
};

// ตัวอย่าง
interface OptionalUser {
  name?: string;
  readonly email?: string;
  age?: number;
}

type MutableUser = Mutable<OptionalUser>;
// {
//   name: string;    // ไม่ใช่ optional แล้ว, ไม่ใช่ readonly แล้ว
//   email: string;   // เช่นกัน
//   age: number;
// }

// Partial and Required implementations
type MyPartial<T> = {
  [K in keyof T]+?: T[K];
};

type MyRequired<T> = {
  [K in keyof T]-?: T[K];
};

type MyReadonly<T> = {
  +readonly [K in keyof T]: T[K];
};

type MyWritable<T> = {
  -readonly [K in keyof T]: T[K];
};

// ตัวอย่างการใช้งาน
interface Product {
  readonly id: string;
  readonly createdAt: Date;
  name: string;
  price: number;
  description?: string;
}

// เมื่อสร้าง product ใหม่ ไม่ต้องการ id และ createdAt
type CreateProductInput = Omit<MyWritable<MyPartial<Product>>, "id" | "createdAt"> & {
  name: string;   // name เป็น required
  price: number;  // price เป็น required
};

// เมื่ออัปเดต product สามารถเปลี่ยนแค่บางส่วนได้
type UpdateProductInput = Partial<Omit<Product, "id" | "createdAt">>;

function createProduct(input: CreateProductInput): Product {
  return {
    id: Math.random().toString(36).slice(2),
    createdAt: new Date(),
    description: "",
    ...input
  };
}
```

---

## Step 1597: Key Remapping in Mapped Types

```typescript
// Key remapping (TypeScript 4.1+) ด้วย as clause

// เปลี่ยน key ให้เป็น getters
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};

interface Person {
  name: string;
  age: number;
  email: string;
}

type PersonGetters = Getters<Person>;
// {
//   getName: () => string;
//   getAge: () => number;
//   getEmail: () => string;
// }

// กรองเฉพาะ keys บางประเภท
type FilterByValueType<T, ValueType> = {
  [K in keyof T as T[K] extends ValueType ? K : never]: T[K];
};

interface Mixed {
  name: string;
  age: number;
  active: boolean;
  score: number;
  description: string;
}

type StringProps = FilterByValueType<Mixed, string>;
// { name: string; description: string; }

type NumberProps = FilterByValueType<Mixed, number>;
// { age: number; score: number; }

// Event map pattern
type EventMap = {
  click: { x: number; y: number };
  keydown: { key: string; ctrlKey: boolean };
  resize: { width: number; height: number };
};

type EventHandlers<T extends Record<string, any>> = {
  [K in keyof T as `on${Capitalize<string & K>}`]?: (event: T[K]) => void;
};

type AppEventHandlers = EventHandlers<EventMap>;
// {
//   onClick?: (event: { x: number; y: number }) => void;
//   onKeydown?: (event: { key: string; ctrlKey: boolean }) => void;
//   onResize?: (event: { width: number; height: number }) => void;
// }

// Inverse mapping
type Flip<T extends Record<PropertyKey, PropertyKey>> = {
  [K in keyof T as T[K]]: K;
};

type Status = {
  active: "ACTIVE";
  inactive: "INACTIVE";
  pending: "PENDING";
};

type InverseStatus = Flip<Status>;
// {
//   ACTIVE: "active";
//   INACTIVE: "inactive";
//   PENDING: "pending";
// }
```

---

## Step 1598: Conditional Distribution

```typescript
// Distributive conditional types
// เมื่อใช้ T extends U โดยที่ T เป็น union type, TypeScript จะ distribute ให้อัตโนมัติ

type IsString<T> = T extends string ? true : false;

type Test1 = IsString<string>;          // true
type Test2 = IsString<number>;          // false
type Test3 = IsString<string | number>; // boolean (true | false)

// ป้องกัน distribution ด้วย []
type IsStringExact<T> = [T] extends [string] ? true : false;

type Test4 = IsStringExact<string | number>; // false (ไม่ distribute)

// Exclude and Extract implementations
type MyExclude<T, U> = T extends U ? never : T;
type MyExtract<T, U> = T extends U ? T : never;

type Nums = 1 | 2 | 3 | 4 | 5;
type EvenNums = MyExtract<Nums, 2 | 4 | 6>;    // 2 | 4
type OddNums = MyExclude<Nums, 2 | 4 | 6>;     // 1 | 3 | 5

// NonNullable implementation
type MyNonNullable<T> = T extends null | undefined ? never : T;

type MaybeString = string | null | undefined;
type DefinitelyString = MyNonNullable<MaybeString>; // string

// Union to intersection (advanced)
type UnionToIntersection<U> = (
  U extends any ? (arg: U) => void : never
) extends (arg: infer I) => void
  ? I
  : never;

type Union = { a: string } | { b: number } | { c: boolean };
type Intersection = UnionToIntersection<Union>;
// { a: string } & { b: number } & { c: boolean }

// Last element of union
type UnionToTuple<T> = UnionToIntersection<
  T extends any ? () => T : never
> extends () => infer R
  ? [...UnionToTuple<Exclude<T, R>>, R]
  : [];

// ตัวอย่าง distribution กับ functions
type FunctionTypes = string | number | boolean;
type FunctionToReturn<T> = T extends any ? () => T : never;

type Functions = FunctionToReturn<FunctionTypes>;
// (() => string) | (() => number) | (() => boolean)
```

---

## Step 1599: Inferring Within Conditional Types

```typescript
// infer keyword ใน conditional types

// ReturnType implementation
type MyReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

function fetchUser(): Promise<{ name: string; age: number }> {
  return Promise.resolve({ name: "สมชาย", age: 30 });
}

type FetchUserReturn = MyReturnType<typeof fetchUser>;
// Promise<{ name: string; age: number }>

// Awaited implementation (unwrap Promise)
type MyAwaited<T> = T extends Promise<infer U>
  ? U extends Promise<any>
    ? MyAwaited<U>
    : U
  : T;

type UserData = MyAwaited<Promise<Promise<{ name: string }>>>;
// { name: string }

// Parameters implementation
type MyParameters<T> = T extends (...args: infer P) => any ? P : never;

function greet(name: string, age: number, greeting: string): string {
  return `${greeting}, ${name}! You are ${age} years old.`;
}

type GreetParams = MyParameters<typeof greet>;
// [name: string, age: number, greeting: string]

// ConstructorParameters
type MyConstructorParameters<T extends new (...args: any[]) => any> =
  T extends new (...args: infer P) => any ? P : never;

class DatabaseConnection {
  constructor(
    public host: string,
    public port: number,
    public database: string
  ) {}
}

type DBConnParams = MyConstructorParameters<typeof DatabaseConnection>;
// [host: string, port: number, database: string]

// Infer nested types
type UnpackArray<T> = T extends Array<infer U> ? U : T;
type UnpackPromise<T> = T extends Promise<infer U> ? U : T;

type ArrayItem = UnpackArray<string[]>; // string
type PromiseResult = UnpackPromise<Promise<number>>; // number

// Infer function return inside conditional
type AsyncReturnType<T extends (...args: any[]) => Promise<any>> =
  T extends (...args: any[]) => Promise<infer R> ? R : never;

async function getUsers(): Promise<Array<{ id: number; name: string }>> {
  return [];
}

type UsersData = AsyncReturnType<typeof getUsers>;
// Array<{ id: number; name: string }>
```

---

## Step 1600: Type-Level Programming Patterns

```typescript
// การเขียนโปรแกรมระดับ type (type-level programming)

// Arithmetic at type level (count with tuples)
type Length<T extends any[]> = T["length"];
type BuildTuple<L extends number, T extends any[] = []> =
  T["length"] extends L ? T : BuildTuple<L, [...T, unknown]>;

type Five = Length<BuildTuple<5>>; // 5

// Add two numbers at type level
type Add<A extends number, B extends number> = Length<
  [...BuildTuple<A>, ...BuildTuple<B>]
>;

type Sum = Add<3, 4>; // 7

// String manipulation at type level
type StringToTuple<S extends string, T extends string[] = []> =
  S extends `${infer Char}${infer Rest}`
    ? StringToTuple<Rest, [...T, Char]>
    : T;

type Chars = StringToTuple<"hello">; // ["h", "e", "l", "l", "o"]

// Reverse a tuple
type Reverse<T extends any[]> = T extends [infer First, ...infer Rest]
  ? [...Reverse<Rest>, First]
  : [];

type Reversed = Reverse<[1, 2, 3, 4, 5]>; // [5, 4, 3, 2, 1]

// Flatten nested arrays at type level
type Flatten<T extends any[]> = T extends [infer First, ...infer Rest]
  ? First extends any[]
    ? [...Flatten<First>, ...Flatten<Rest>]
    : [First, ...Flatten<Rest>]
  : [];

type Flat = Flatten<[1, [2, 3], [4, [5, 6]]]>; // [1, 2, 3, 4, 5, 6]

// Type-safe path accessor
type PathValue<T, P extends string> =
  P extends `${infer Key}.${infer Rest}`
    ? Key extends keyof T
      ? PathValue<T[Key], Rest>
      : never
    : P extends keyof T
    ? T[P]
    : never;

interface AppState {
  user: {
    profile: {
      name: string;
      email: string;
    };
    settings: {
      theme: "light" | "dark";
      language: string;
    };
  };
  posts: Array<{
    title: string;
    content: string;
  }>;
}

type UserName = PathValue<AppState, "user.profile.name">;     // string
type UserTheme = PathValue<AppState, "user.settings.theme">; // "light" | "dark"
```

---

## Step 1601: Builder Pattern with TypeScript

```typescript
// Builder Pattern ที่ type-safe

class QueryBuilder<T extends Record<string, any>> {
  private tableName: string = "";
  private conditions: string[] = [];
  private selectedFields: (keyof T)[] = [];
  private limitCount?: number;
  private offsetCount?: number;
  private orderByField?: keyof T;
  private orderDirection: "ASC" | "DESC" = "ASC";

  constructor(table: string) {
    this.tableName = table;
  }

  select<K extends keyof T>(...fields: K[]): QueryBuilder<Pick<T, K>> {
    this.selectedFields = fields as any;
    return this as any;
  }

  where(condition: string): this {
    this.conditions.push(condition);
    return this;
  }

  limit(count: number): this {
    this.limitCount = count;
    return this;
  }

  offset(count: number): this {
    this.offsetCount = count;
    return this;
  }

  orderBy(field: keyof T, direction: "ASC" | "DESC" = "ASC"): this {
    this.orderByField = field;
    this.orderDirection = direction;
    return this;
  }

  build(): string {
    let query = `SELECT ${
      this.selectedFields.length
        ? this.selectedFields.join(", ")
        : "*"
    } FROM ${this.tableName}`;
    
    if (this.conditions.length) {
      query += ` WHERE ${this.conditions.join(" AND ")}`;
    }
    if (this.orderByField) {
      query += ` ORDER BY ${String(this.orderByField)} ${this.orderDirection}`;
    }
    if (this.limitCount !== undefined) {
      query += ` LIMIT ${this.limitCount}`;
    }
    if (this.offsetCount !== undefined) {
      query += ` OFFSET ${this.offsetCount}`;
    }
    return query;
  }
}

interface User {
  id: number;
  name: string;
  email: string;
  age: number;
  createdAt: Date;
}

const query = new QueryBuilder<User>("users")
  .select("id", "name", "email")
  .where("age > 18")
  .where("active = true")
  .orderBy("name", "ASC")
  .limit(10)
  .offset(0)
  .build();

console.log(query);
// SELECT id, name, email FROM users WHERE age > 18 AND active = true ORDER BY name ASC LIMIT 10 OFFSET 0
```

---

## Step 1602: Fluent API Typing

```typescript
// Fluent API ที่ type-safe ด้วย method chaining

// Step builder pattern - บังคับให้เรียก method ตามลำดับ
interface HasName {
  setAge(age: number): HasAge;
}

interface HasAge {
  setEmail(email: string): HasEmail;
}

interface HasEmail {
  build(): UserProfile;
}

interface UserProfile {
  name: string;
  age: number;
  email: string;
}

class UserProfileBuilder implements HasName, HasAge, HasEmail {
  private profile: Partial<UserProfile> = {};

  static create(name: string): HasName {
    const builder = new UserProfileBuilder();
    builder.profile.name = name;
    return builder;
  }

  setAge(age: number): HasAge {
    this.profile.age = age;
    return this;
  }

  setEmail(email: string): HasEmail {
    this.profile.email = email;
    return this;
  }

  build(): UserProfile {
    if (!this.profile.name || !this.profile.age || !this.profile.email) {
      throw new Error("ข้อมูลไม่ครบ");
    }
    return this.profile as UserProfile;
  }
}

// ต้องเรียกตามลำดับ: create -> setAge -> setEmail -> build
const user = UserProfileBuilder
  .create("สมชาย")
  .setAge(30)
  .setEmail("somchai@example.com")
  .build();

// TypeScript validation pipeline
type Validator<T, R = T> = (value: T) => { valid: true; value: R } | { valid: false; error: string };

class ValidationChain<T> {
  private validators: Validator<T>[] = [];

  addRule(validator: Validator<T>): this {
    this.validators.push(validator);
    return this;
  }

  validate(value: T): { valid: boolean; value?: T; errors: string[] } {
    const errors: string[] = [];
    let currentValue = value;

    for (const validator of this.validators) {
      const result = validator(currentValue);
      if (!result.valid) {
        errors.push(result.error);
      } else {
        currentValue = result.value;
      }
    }

    return {
      valid: errors.length === 0,
      value: errors.length === 0 ? currentValue : undefined,
      errors
    };
  }
}

const emailValidator = new ValidationChain<string>()
  .addRule(value => value.length > 0
    ? { valid: true, value: value.toLowerCase() }
    : { valid: false, error: "email ต้องไม่ว่าง" }
  )
  .addRule(value => value.includes("@")
    ? { valid: true, value }
    : { valid: false, error: "email ต้องมี @" }
  )
  .addRule(value => value.includes(".")
    ? { valid: true, value }
    : { valid: false, error: "email ต้องมี domain" }
  );

console.log(emailValidator.validate("Test@Example.COM"));
// { valid: true, value: "test@example.com", errors: [] }

console.log(emailValidator.validate("invalid-email"));
// { valid: false, errors: ["email ต้องมี @"] }
```

---

## Step 1603: Phantom Types

```typescript
// Phantom types - ใช้ type parameter ที่ไม่ปรากฏใน runtime
// เพื่อแยกแยะ values ที่มี underlying type เหมือนกันแต่ความหมายต่างกัน

// ตัวอย่าง: USD vs THB
type Currency = "USD" | "THB" | "EUR";
type Money<C extends Currency> = { readonly amount: number; readonly _currency: C };

function createMoney<C extends Currency>(amount: number, currency: C): Money<C> {
  return { amount, _currency: currency };
}

const usd = createMoney(100, "USD");
const thb = createMoney(3500, "THB");

function addMoney<C extends Currency>(a: Money<C>, b: Money<C>): Money<C> {
  return createMoney(a.amount + b.amount, a._currency);
}

addMoney(usd, createMoney(50, "USD"));   // OK
// addMoney(usd, thb);                   // Error! ไม่สามารถบวก USD กับ THB โดยตรง

// Validated vs Unvalidated
declare const _brand: unique symbol;

type Branded<T, B extends string> = T & { [_brand]: B };
type ValidEmail = Branded<string, "ValidEmail">;
type RawInput = string;

function validateEmail(email: RawInput): ValidEmail | null {
  if (/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(email)) {
    return email as ValidEmail;
  }
  return null;
}

function sendEmail(to: ValidEmail, subject: string, body: string): void {
  console.log(`ส่ง email ไปยัง ${to}: ${subject}`);
}

const rawEmail: RawInput = "user@example.com";
// sendEmail(rawEmail, "Test", "Body"); // Error! ต้อง validate ก่อน

const validEmail = validateEmail(rawEmail);
if (validEmail) {
  sendEmail(validEmail, "ยืนยัน email", "กรุณายืนยัน email ของคุณ"); // OK
}

// Units of measure
type Meters = Branded<number, "Meters">;
type Kilograms = Branded<number, "Kilograms">;
type Seconds = Branded<number, "Seconds">;

function toMeters(value: number): Meters {
  return value as Meters;
}

function toKilograms(value: number): Kilograms {
  return value as Kilograms;
}

function calculateSpeed(distance: Meters, time: Seconds): number {
  return distance / time;
}

const distance = toMeters(100);
const time = 9.58 as Seconds;
const speed = calculateSpeed(distance, time); // OK

// calculateSpeed(distance, distance);  // Error! ใส่ units ผิด
```

---

## Step 1604: Brand Types

```typescript
// Brand types เป็นรูปแบบที่ใช้บ่อยในการทำ type-safe APIs

// Simple brand
type UserId = string & { readonly __brand: "UserId" };
type PostId = string & { readonly __brand: "PostId" };

function createUserId(id: string): UserId {
  return id as UserId;
}

function createPostId(id: string): PostId {
  return id as PostId;
}

interface UserRepo {
  findById(id: UserId): Promise<User | null>;
}

interface PostRepo {
  findById(id: PostId): Promise<Post | null>;
}

interface User {
  id: UserId;
  name: string;
}

interface Post {
  id: PostId;
  title: string;
  authorId: UserId;
}

// ไม่สามารถใส่ UserId ใน PostId และกลับกัน
const userId = createUserId("user-123");
const postId = createPostId("post-456");

// userRepo.findById(postId);  // Error!
// postRepo.findById(userId);  // Error!

// Brand ที่ซับซ้อนขึ้น - Positive number
type PositiveNumber = number & { readonly _brand: "Positive" };

function positive(n: number): PositiveNumber {
  if (n <= 0) throw new Error("ต้องเป็นตัวเลขที่มากกว่า 0");
  return n as PositiveNumber;
}

function divide(numerator: number, denominator: PositiveNumber): number {
  return numerator / denominator; // ไม่ต้องเช็คว่า denominator เป็น 0
}

const result = divide(10, positive(2)); // 5

// ตัวอย่าง NonEmptyArray
type NonEmptyArray<T> = [T, ...T[]];

function first<T>(arr: NonEmptyArray<T>): T {
  return arr[0]; // ไม่ต้องเช็คว่า array ว่างไหม
}

function toNonEmpty<T>(arr: T[]): NonEmptyArray<T> {
  if (arr.length === 0) throw new Error("Array ต้องไม่ว่าง");
  return arr as NonEmptyArray<T>;
}

const items = ["a", "b", "c"];
const nonEmpty = toNonEmpty(items);
const firstItem = first(nonEmpty); // string (ไม่ใช่ string | undefined)

// Readonly brand
type Immutable<T> = { readonly [K in keyof T]: Immutable<T[K]> };
```

---

## Step 1605: Type-Safe Event Emitter

```typescript
// Type-safe Event Emitter

type EventMap = Record<string, any>;

type EventKey<T extends EventMap> = string & keyof T;

type EventReceiver<T> = (params: T) => void;

class TypedEventEmitter<T extends EventMap> {
  private listeners: {
    [K in keyof T]?: Set<EventReceiver<T[K]>>;
  } = {};

  on<K extends EventKey<T>>(event: K, listener: EventReceiver<T[K]>): () => void {
    if (!this.listeners[event]) {
      this.listeners[event] = new Set();
    }
    this.listeners[event]!.add(listener);
    
    // return unsubscribe function
    return () => this.off(event, listener);
  }

  once<K extends EventKey<T>>(event: K, listener: EventReceiver<T[K]>): void {
    const wrapper = (data: T[K]) => {
      listener(data);
      this.off(event, wrapper);
    };
    this.on(event, wrapper);
  }

  off<K extends EventKey<T>>(event: K, listener: EventReceiver<T[K]>): void {
    this.listeners[event]?.delete(listener);
  }

  emit<K extends EventKey<T>>(event: K, data: T[K]): void {
    this.listeners[event]?.forEach(listener => listener(data));
  }
}

// กำหนด events ของ app
interface AppEvents {
  userLoggedIn: { userId: string; username: string; timestamp: Date };
  userLoggedOut: { userId: string };
  messageReceived: { from: string; to: string; content: string; timestamp: Date };
  errorOccurred: { code: number; message: string; stack?: string };
  dataUpdated: { entityType: string; entityId: string; changes: Record<string, any> };
}

const emitter = new TypedEventEmitter<AppEvents>();

// Subscribe to events
const unsubscribe = emitter.on("userLoggedIn", ({ userId, username, timestamp }) => {
  console.log(`ผู้ใช้ ${username} (${userId}) เข้าสู่ระบบเมื่อ ${timestamp.toISOString()}`);
});

emitter.on("errorOccurred", ({ code, message }) => {
  console.error(`Error ${code}: ${message}`);
});

// Emit events
emitter.emit("userLoggedIn", {
  userId: "user-123",
  username: "สมชาย",
  timestamp: new Date()
});

// emitter.emit("userLoggedIn", { userId: "123" }); // Error! ข้อมูลไม่ครบ

// Unsubscribe
unsubscribe();
```

---

## Step 1606: Exhaustive Pattern Matching

```typescript
// Exhaustive pattern matching - TypeScript ตรวจสอบว่า handle ทุก case

// ใช้ never type เพื่อ exhaustive check
function assertNever(value: never): never {
  throw new Error(`Unexpected value: ${JSON.stringify(value)}`);
}

type Shape =
  | { kind: "circle"; radius: number }
  | { kind: "rectangle"; width: number; height: number }
  | { kind: "triangle"; base: number; height: number };

function calculateArea(shape: Shape): number {
  switch (shape.kind) {
    case "circle":
      return Math.PI * shape.radius ** 2;
    case "rectangle":
      return shape.width * shape.height;
    case "triangle":
      return (shape.base * shape.height) / 2;
    default:
      return assertNever(shape); // TypeScript จะ error ถ้าเพิ่ม type ใหม่แล้วไม่ handle
  }
}

// Result type pattern
type Result<T, E = Error> =
  | { success: true; data: T }
  | { success: false; error: E };

function handleResult<T, E = Error>(
  result: Result<T, E>,
  handlers: {
    onSuccess: (data: T) => void;
    onError: (error: E) => void;
  }
): void {
  if (result.success) {
    handlers.onSuccess(result.data);
  } else {
    handlers.onError(result.error);
  }
}

// Option type (Maybe monad)
type Option<T> = { isSome: true; value: T } | { isSome: false };

function some<T>(value: T): Option<T> {
  return { isSome: true, value };
}

const none: Option<never> = { isSome: false };

function mapOption<T, U>(option: Option<T>, fn: (value: T) => U): Option<U> {
  if (option.isSome) {
    return some(fn(option.value));
  }
  return none;
}

function flatMapOption<T, U>(option: Option<T>, fn: (value: T) => Option<U>): Option<U> {
  if (option.isSome) {
    return fn(option.value);
  }
  return none;
}

// ตัวอย่างการใช้
const userOption = some({ name: "สมชาย", email: "som@example.com" });
const emailOption = mapOption(userOption, user => user.email);

if (emailOption.isSome) {
  console.log(`Email: ${emailOption.value}`);
}
```

---

## Step 1607: Decorators with Metadata

```typescript
// Decorators (TypeScript experimental)
// tsconfig: { "experimentalDecorators": true, "emitDecoratorMetadata": true }

// Class decorator
function sealed(constructor: Function) {
  Object.seal(constructor);
  Object.seal(constructor.prototype);
}

@sealed
class BankAccount {
  private balance: number = 0;
  
  deposit(amount: number) {
    this.balance += amount;
  }
  
  withdraw(amount: number): boolean {
    if (amount > this.balance) return false;
    this.balance -= amount;
    return true;
  }
}

// Method decorator สำหรับ logging
function log(
  target: any,
  propertyKey: string,
  descriptor: PropertyDescriptor
): PropertyDescriptor {
  const originalMethod = descriptor.value;
  
  descriptor.value = function(...args: any[]) {
    console.log(`เรียก ${propertyKey} ด้วย args:`, args);
    const result = originalMethod.apply(this, args);
    console.log(`${propertyKey} คืนค่า:`, result);
    return result;
  };
  
  return descriptor;
}

// Property decorator
function required(target: any, propertyKey: string): void {
  let value: any;
  
  Object.defineProperty(target, propertyKey, {
    get() { return value; },
    set(newValue: any) {
      if (newValue === null || newValue === undefined || newValue === "") {
        throw new Error(`${propertyKey} ต้องมีค่า`);
      }
      value = newValue;
    }
  });
}

// Parameter decorator
function validate(
  target: any,
  propertyKey: string,
  parameterIndex: number
): void {
  const existingParams: number[] =
    Reflect.getOwnMetadata("validate:params", target, propertyKey) || [];
  existingParams.push(parameterIndex);
  Reflect.defineMetadata("validate:params", existingParams, target, propertyKey);
}

class UserService {
  @log
  createUser(name: string, email: string): { id: string; name: string; email: string } {
    return {
      id: Math.random().toString(36).slice(2),
      name,
      email
    };
  }

  @log
  deleteUser(id: string): boolean {
    console.log(`ลบผู้ใช้ ${id}`);
    return true;
  }
}

const service = new UserService();
service.createUser("สมชาย", "som@example.com");
// Output:
// เรียก createUser ด้วย args: ["สมชาย", "som@example.com"]
// createUser คืนค่า: { id: "...", name: "สมชาย", email: "som@example.com" }
```

---

## Step 1608: Module Augmentation for Libraries

```typescript
// Module augmentation - การเพิ่ม types ให้กับ library ที่มีอยู่แล้ว

// สมมติ library มี interface นี้
// (ใน node_modules/some-library/index.d.ts)
// interface Request {
//   user?: { id: string };
// }

// เราสามารถเพิ่ม property ได้
declare module "express" {
  interface Request {
    user?: {
      id: string;
      name: string;
      role: "admin" | "user" | "moderator";
      permissions: string[];
    };
    requestId?: string;
    startTime?: number;
  }
}

// Augment Array prototype
interface Array<T> {
  first(): T | undefined;
  last(): T | undefined;
  groupBy<K extends PropertyKey>(
    fn: (item: T) => K
  ): Record<K, T[]>;
}

// Implementation
Array.prototype.first = function() {
  return this[0];
};

Array.prototype.last = function() {
  return this[this.length - 1];
};

Array.prototype.groupBy = function(fn) {
  return this.reduce((groups, item) => {
    const key = fn(item);
    if (!groups[key]) groups[key] = [];
    groups[key].push(item);
    return groups;
  }, {} as any);
};

// ตัวอย่างการใช้
const users = [
  { name: "สมชาย", role: "admin" as const },
  { name: "สมหญิง", role: "user" as const },
  { name: "สมศักดิ์", role: "admin" as const },
  { name: "สมพร", role: "user" as const },
];

const grouped = users.groupBy(u => u.role);
// { admin: [...], user: [...] }

// Global augmentation
declare global {
  interface Window {
    myApp?: {
      version: string;
      debug: boolean;
    };
  }
  
  interface String {
    toSlug(): string;
  }
}

String.prototype.toSlug = function() {
  return this.toLowerCase().replace(/\s+/g, "-").replace(/[^\w-]/g, "");
};

const slug = "Hello World! ยินดีต้อนรับ".toSlug();
console.log(slug); // "hello-world--"
```

---

## Step 1609: Advanced tsconfig Options

```json
// tsconfig.json ที่ครอบคลุม

{
  "compilerOptions": {
    // Target และ Module
    "target": "ES2022",
    "module": "ESNext",
    "moduleResolution": "bundler",
    "lib": ["ES2022", "DOM", "DOM.Iterable"],
    
    // Strict mode options
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "strictBindCallApply": true,
    "strictPropertyInitialization": true,
    "noImplicitThis": true,
    "alwaysStrict": true,
    
    // Additional checks
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true,
    "exactOptionalPropertyTypes": true,
    "noUncheckedIndexedAccess": true,
    
    // Module features
    "esModuleInterop": true,
    "allowSyntheticDefaultImports": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    
    // Decorators
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true,
    
    // Paths and directories
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"],
      "@components/*": ["src/components/*"],
      "@utils/*": ["src/utils/*"]
    },
    "rootDir": "src",
    "outDir": "dist",
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    
    // Performance
    "incremental": true,
    "tsBuildInfoFile": ".tsbuildinfo",
    "skipLibCheck": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist", "**/*.spec.ts"]
}
```

```typescript
// tsconfig paths usage
// ใช้ path alias แทน relative imports ยาวๆ

// แทนที่จะเขียน:
// import { Button } from "../../../components/ui/Button";

// เขียนได้ว่า:
// import { Button } from "@components/ui/Button";

// noUncheckedIndexedAccess ตัวอย่าง
const arr = [1, 2, 3];
const first = arr[0]; // type: number | undefined (ด้วย noUncheckedIndexedAccess)

if (first !== undefined) {
  console.log(first.toFixed(2)); // OK
}

// exactOptionalPropertyTypes ตัวอย่าง
interface Config {
  timeout?: number; // ต้องใส่ number เท่านั้น หรือไม่ใส่เลย
  // ไม่สามารถ set เป็น undefined ได้โดยตรง
}

// โดยไม่มี exactOptionalPropertyTypes:
// const config: Config = { timeout: undefined }; // OK

// ด้วย exactOptionalPropertyTypes:
// const config: Config = { timeout: undefined }; // Error!
const config: Config = { timeout: 5000 }; // OK
const config2: Config = {}; // OK
```

---

## Step 1610: TypeScript Compiler API Basics

```typescript
// TypeScript Compiler API - ใช้ TypeScript เป็น library

import * as ts from "typescript";

// 1. สร้าง TypeScript program
function createProgram(fileNames: string[]): ts.Program {
  const options: ts.CompilerOptions = {
    target: ts.ScriptTarget.ES2022,
    strict: true,
    noEmit: true
  };
  
  return ts.createProgram(fileNames, options);
}

// 2. วิเคราะห์ AST
function analyzeFile(fileName: string): void {
  const program = createProgram([fileName]);
  const sourceFile = program.getSourceFile(fileName);
  
  if (!sourceFile) {
    console.error(`ไม่พบไฟล์ ${fileName}`);
    return;
  }
  
  // Traverse AST
  function visit(node: ts.Node): void {
    if (ts.isFunctionDeclaration(node) && node.name) {
      console.log(`พบ function: ${node.name.text}`);
    }
    
    if (ts.isClassDeclaration(node) && node.name) {
      console.log(`พบ class: ${node.name.text}`);
    }
    
    if (ts.isInterfaceDeclaration(node)) {
      console.log(`พบ interface: ${node.name.text}`);
    }
    
    ts.forEachChild(node, visit);
  }
  
  visit(sourceFile);
}

// 3. สร้าง TypeScript code ด้วย Compiler API
function generateInterface(name: string, properties: Record<string, string>): string {
  const factory = ts.factory;
  
  const members = Object.entries(properties).map(([propName, propType]) =>
    factory.createPropertySignature(
      undefined,
      factory.createIdentifier(propName),
      undefined,
      factory.createTypeReferenceNode(propType)
    )
  );
  
  const interfaceDecl = factory.createInterfaceDeclaration(
    [factory.createToken(ts.SyntaxKind.ExportKeyword)],
    factory.createIdentifier(name),
    undefined,
    undefined,
    members
  );
  
  const printer = ts.createPrinter();
  const resultFile = ts.createSourceFile(
    "temp.ts",
    "",
    ts.ScriptTarget.Latest,
    false,
    ts.ScriptKind.TS
  );
  
  return printer.printNode(ts.EmitHint.Unspecified, interfaceDecl, resultFile);
}

// สร้าง interface จากข้อมูล
const code = generateInterface("User", {
  id: "string",
  name: "string",
  age: "number"
});

console.log(code);
// export interface User {
//     id: string;
//     name: string;
//     age: number;
// }

// 4. Type checking
function checkTypes(fileName: string): ts.Diagnostic[] {
  const program = createProgram([fileName]);
  const diagnostics = ts.getPreEmitDiagnostics(program);
  return Array.from(diagnostics);
}
```

---

## แบบฝึกหัด (Exercises)

### แบบฝึกหัดที่ 1: Generic Query Builder
สร้าง type-safe query builder ที่รองรับ:
- `select()` - เลือก columns
- `where()` - กรองข้อมูล
- `join()` - join ตาราง
- `limit()` และ `offset()`
- `orderBy()`

```typescript
// TODO: สร้าง TypeSafeQueryBuilder<T>
// ที่มี method chaining และ type checking
interface Exercise1 {
  // เพิ่ม interface ที่นี่
}
```

### แบบฝึกหัดที่ 2: Recursive Type
สร้าง `DeepMerge<A, B>` type ที่ merge object แบบ deep:

```typescript
type DeepMerge<A, B> = {
  // TODO: implement
};

type Result = DeepMerge<
  { a: string; b: { c: number; d: boolean } },
  { b: { c: string; e: Date }; f: number }
>;
// Expected: { a: string; b: { c: string; d: boolean; e: Date }; f: number }
```

### แบบฝึกหัดที่ 3: Type-Safe Router
สร้าง type-safe router ที่:
- กำหนด routes แบบ type-safe
- params extraction ถูกต้อง
- query string typing

```typescript
type Routes = {
  "/users": { params: {}; query: { page?: string; limit?: string } };
  "/users/:id": { params: { id: string }; query: {} };
  "/posts/:postId/comments/:commentId": { 
    params: { postId: string; commentId: string }; 
    query: {} 
  };
};

// TODO: สร้าง Router class ที่ใช้ Routes type
```

### แบบฝึกหัดที่ 4: State Machine
สร้าง type-safe state machine:

```typescript
type TrafficLight = {
  states: "red" | "yellow" | "green";
  transitions: {
    red: "green";
    green: "yellow";
    yellow: "red";
  };
};

// TODO: สร้าง StateMachine<T extends { states: string; transitions: Record<string, string> }>
// ที่บังคับให้ transition ถูกต้อง
```

### แบบฝึกหัดที่ 5: Validation Library
สร้าง mini validation library ที่ type-safe:

```typescript
// TODO: สร้าง schema validators ที่:
// - z.string().min(3).max(50).email()
// - z.number().positive().max(100)
// - z.object({ name: z.string(), age: z.number() })
// - infer TypeScript type จาก schema

type UserSchema = typeof userSchema; // ต้อง infer เป็น { name: string; age: number }
```

---

## สรุป (Summary)

ในบทนี้เราได้เรียนรู้:

1. **Advanced Generic Constraints** - การใช้ extends เพื่อจำกัด type parameters
2. **Generic Utility Functions** - Pattern ทั่วไปสำหรับ utility functions
3. **Recursive Generic Types** - Types ที่อ้างอิงตัวเอง
4. **Variadic Tuple Types** - Tuple ที่ยืดหยุ่นได้
5. **Template Literal Types** - สร้าง string types แบบ dynamic
6. **Mapped Type Modifiers** - + และ - เพื่อแก้ไข modifiers
7. **Key Remapping** - เปลี่ยน key names ใน mapped types
8. **Conditional Distribution** - Distribution ของ union types ใน conditionals
9. **infer keyword** - การ extract types จาก conditional types
10. **Type-Level Programming** - การเขียนโปรแกรมที่ type level
11. **Builder Pattern** - Pattern สำหรับสร้าง objects อย่าง type-safe
12. **Fluent API** - Method chaining ที่ type-safe
13. **Phantom Types** - Types ที่ไม่มีผลที่ runtime
14. **Brand Types** - การแยกแยะ values ด้วย type branding
15. **Type-Safe Event Emitter** - Event emitter ที่ fully typed
16. **Exhaustive Matching** - ตรวจสอบว่า handle ทุก case
17. **Decorators** - Metadata-based decorators
18. **Module Augmentation** - เพิ่ม types ให้กับ libraries
19. **Advanced tsconfig** - การตั้งค่า TypeScript compiler
20. **Compiler API** - ใช้ TypeScript เป็น library

TypeScript ขั้นสูงช่วยให้เราสร้าง APIs ที่ type-safe, self-documenting, และยากที่จะใช้ผิด
