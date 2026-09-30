# Part 69: TypeScript พื้นฐาน (Steps 1351-1370)

## บทนำ: TypeScript คืออะไร

TypeScript คือ "Typed JavaScript" ที่พัฒนาโดย Microsoft เป็น superset ของ JavaScript โดยเพิ่ม:
- **Static Type Checking** - ตรวจสอบ types ตอน compile time
- **Better IDE Support** - autocomplete, refactoring ที่ดีกว่า
- **Early Error Detection** - พบ bugs ก่อน runtime
- **Better Documentation** - types เป็น documentation ในตัวเอง

```
TypeScript → (tsc compiler) → JavaScript
```

---

## Step 1351: ทำไมต้องใช้ TypeScript

```javascript
// JavaScript - ไม่มี type safety
function calculateTotal(price, quantity) {
  return price * quantity;
}

calculateTotal("100", 5);  // "100" * 5 = NaN? หรือ 500?
calculateTotal(100);       // undefined → NaN
calculateTotal();          // NaN
```

```typescript
// TypeScript - มี type safety
function calculateTotal(price: number, quantity: number): number {
  return price * quantity;
}

calculateTotal("100", 5);  // ❌ Error: Argument of type 'string' is not assignable to 'number'
calculateTotal(100);       // ❌ Error: Expected 2 arguments, but got 1
calculateTotal(100, 5);    // ✅ 500
```

### ประโยชน์จริงๆ ของ TypeScript

```typescript
// ตัวอย่าง: ทีมใหญ่ทำงานร่วมกัน

// user.ts - นักพัฒนา A เขียน
interface User {
  id: number;
  name: string;
  email: string;
  role: 'admin' | 'user' | 'moderator';
  createdAt: Date;
  address?: {
    street: string;
    city: string;
    country: string;
  };
}

function getUserFullName(user: User): string {
  return user.name;
  // user.fullName → ❌ Error: Property 'fullName' does not exist
}

// dashboard.ts - นักพัฒนา B เขียน
import { User } from './user';

function renderUserCard(user: User) {
  // IDE autocomplete ทำงาน!
  // user. → แสดง: id, name, email, role, createdAt, address
  
  return `
    <div class="user-card">
      <h3>${user.name}</h3>
      <p>${user.email}</p>
      <span class="role">${user.role}</span>
    </div>
  `;
}
```

---

## Step 1352: การติดตั้ง TypeScript

```bash
# ติดตั้ง TypeScript globally
npm install -g typescript

# ตรวจสอบ version
tsc --version

# ติดตั้ง TypeScript ใน project (แนะนำ)
npm install --save-dev typescript

# ติดตั้ง type definitions สำหรับ Node.js
npm install --save-dev @types/node

# สร้าง tsconfig.json
tsc --init

# Compile TypeScript
tsc

# Compile และ watch สำหรับ changes
tsc --watch

# รัน TypeScript โดยตรง (development)
npm install -g ts-node
ts-node app.ts

# tsx (เร็วกว่า ts-node)
npm install -g tsx
tsx app.ts
```

---

## Step 1353: tsconfig.json

```json
{
  "compilerOptions": {
    // ===== Target & Module =====
    "target": "ES2022",          // JavaScript version เป้าหมาย
    "module": "CommonJS",        // Module system (CommonJS, ESNext, etc.)
    "lib": ["ES2022", "DOM"],    // Type definitions ที่ใช้
    
    // ===== Output =====
    "outDir": "./dist",          // ที่บันทึก compiled JS
    "rootDir": "./src",          // ที่อยู่ TypeScript source
    "declaration": true,         // สร้าง .d.ts files
    "sourceMap": true,           // สร้าง source maps
    "removeComments": true,      // ลบ comments ใน output
    
    // ===== Strict Type Checking =====
    "strict": true,              // เปิด strict mode ทั้งหมด
    // Strict mode รวม:
    // "noImplicitAny": true,    // ห้าม type any โดยปริยาย
    // "strictNullChecks": true, // null/undefined ต้อง explicit
    // "strictFunctionTypes": true,
    // "strictBindCallApply": true,
    // "strictPropertyInitialization": true,
    // "noImplicitThis": true,
    // "alwaysStrict": true,
    
    // ===== Additional Checks =====
    "noUnusedLocals": true,      // Error ถ้ามี variables ที่ไม่ใช้
    "noUnusedParameters": true,  // Error ถ้ามี parameters ที่ไม่ใช้
    "noImplicitReturns": true,   // ทุก code path ต้อง return
    "noFallthroughCasesInSwitch": true, // ป้องกัน switch fallthrough
    "exactOptionalPropertyTypes": true,  // Optional properties เข้มงวด
    
    // ===== Module Resolution =====
    "moduleResolution": "node16", // หรือ "bundler" สำหรับ Vite/webpack
    "baseUrl": ".",
    "paths": {
      "@/*": ["./src/*"],
      "@components/*": ["./src/components/*"],
      "@utils/*": ["./src/utils/*"],
    },
    "esModuleInterop": true,     // Import CommonJS modules ได้ง่ายขึ้น
    "allowSyntheticDefaultImports": true,
    "resolveJsonModule": true,   // Import JSON files ได้
    
    // ===== JSX (สำหรับ React) =====
    "jsx": "react-jsx",          // หรือ "react", "preserve"
    
    // ===== Experimental =====
    "experimentalDecorators": true,
    "emitDecoratorMetadata": true,
  },
  
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist", "**/*.spec.ts", "**/*.test.ts"]
}
```

---

## Step 1354: Basic Types

```typescript
// ===== Primitive Types =====

// string
let name: string = 'สมชาย';
let greeting: string = `สวัสดี ${name}`;
let message: string = "Hello World";

// number - รวมทั้ง integer และ float
let age: number = 25;
let price: number = 19.99;
let hex: number = 0xff;
let binary: number = 0b1010;
let octal: number = 0o77;

// boolean
let isActive: boolean = true;
let isDeleted: boolean = false;

// bigint
let bigNumber: bigint = 9007199254740991n;

// symbol
let sym: symbol = Symbol('unique');

// ===== null และ undefined =====

// ด้วย strictNullChecks: true (แนะนำ)
let nullValue: null = null;
let undefinedValue: undefined = undefined;

// ไม่ assign null/undefined ให้ type อื่นได้
let str: string = null;      // ❌ Error
let str2: string | null = null; // ✅

// ===== any =====
// หลีกเลี่ยง! ปิด type checking
let anything: any = 'hello';
anything = 42;      // ✅ ไม่ error
anything = true;    // ✅ ไม่ error
anything.foo.bar;   // ✅ ไม่ error (แต่อาจ crash runtime!)

// ===== unknown =====
// ปลอดภัยกว่า any - ต้อง narrow type ก่อนใช้
let unknown: unknown = fetchData();

// ❌ Error: ต้อง narrow ก่อน
unknown.toUpperCase();

// ✅ ต้อง check type ก่อน
if (typeof unknown === 'string') {
  unknown.toUpperCase(); // ✅ ตอนนี้รู้ว่าเป็น string
}

// ===== never =====
// Type ที่ไม่มีค่าเลย

// Function ที่ไม่ return ค่า (throw หรือ infinite loop)
function throwError(message: string): never {
  throw new Error(message);
}

function infiniteLoop(): never {
  while (true) {}
}

// Exhaustive checking
type Shape = 'circle' | 'square' | 'triangle';

function getArea(shape: Shape, size: number): number {
  switch (shape) {
    case 'circle': return Math.PI * size ** 2;
    case 'square': return size ** 2;
    case 'triangle': return (size ** 2) / 2;
    default:
      const exhaustive: never = shape; // Error ถ้า case ไม่ครบ
      throw new Error(`Unknown shape: ${exhaustive}`);
  }
}

// ===== void =====
// Function ที่ไม่ return ค่า
function logMessage(msg: string): void {
  console.log(msg);
  // ไม่มี return
}

// void vs undefined
function returnsVoid(): void {
  return; // ✅
  return undefined; // ✅
}
```

---

## Step 1355: Type Annotations และ Type Inference

```typescript
// ===== Type Annotations (กำหนด type เอง) =====

// Variables
let username: string = 'alice';
let count: number = 0;
let flag: boolean = true;

// Functions
function add(a: number, b: number): number {
  return a + b;
}

// ===== Type Inference (TypeScript เดา type เอง) =====

// TypeScript รู้ว่า name เป็น string
let name = 'Alice'; // type: string (inferred)
// name = 42; ❌ Error: Type 'number' is not assignable to type 'string'

// TypeScript รู้ว่า sum เป็น number
let sum = 10 + 20; // type: number (inferred)

// Return type inference
function multiply(a: number, b: number) {
  return a * b; // TypeScript รู้ว่า return type คือ number
}

// Object inference
const config = {
  host: 'localhost',
  port: 3000,
  debug: true,
};
// config: { host: string; port: number; debug: boolean; }

// Array inference
const numbers = [1, 2, 3]; // type: number[]
const mixed = [1, 'hello', true]; // type: (number | string | boolean)[]

// ===== เมื่อไหร่ควรกำหนด type เอง =====

// 1. เมื่อ initialize ทีหลัง
let result: string;
if (condition) {
  result = 'success';
} else {
  result = 'failure';
}

// 2. เมื่อต้องการ type กว้างกว่า inference
let status: string | number = 'active'; // กว้างกว่า 'active'

// 3. Function parameters (ต้องกำหนดเสมอ)
function greet(name: string): string {
  return `Hello, ${name}!`;
}

// 4. เมื่อ return type ซับซ้อน
function getUser(id: number): Promise<User | null> {
  // ...
}
```

---

## Step 1356: Arrays และ Tuples

```typescript
// ===== Arrays =====

// Syntax แบบที่ 1: Type[]
let numbers: number[] = [1, 2, 3, 4, 5];
let names: string[] = ['Alice', 'Bob', 'Charlie'];
let flags: boolean[] = [true, false, true];

// Syntax แบบที่ 2: Array<Type>
let scores: Array<number> = [85, 90, 78];
let users: Array<string> = ['Alice', 'Bob'];

// Array ของ objects
interface Product {
  id: number;
  name: string;
  price: number;
}

let products: Product[] = [
  { id: 1, name: 'Laptop', price: 45000 },
  { id: 2, name: 'Mouse', price: 500 },
];

// Readonly array (ห้ามแก้ไข)
const immutableNumbers: readonly number[] = [1, 2, 3];
// immutableNumbers.push(4); ❌ Error
// immutableNumbers[0] = 10; ❌ Error

// ReadonlyArray<T>
const readonlyArr: ReadonlyArray<string> = ['a', 'b', 'c'];

// Nested arrays
let matrix: number[][] = [[1, 2], [3, 4], [5, 6]];

// Array methods ยังคง type-safe
const doubled = numbers.map(n => n * 2); // type: number[]
const sum = numbers.reduce((acc, n) => acc + n, 0); // type: number

// ===== Tuples =====
// Array ที่มีจำนวนและประเภทของ elements ที่แน่นอน

// Basic tuple
let coordinate: [number, number] = [10, 20];
let point3d: [number, number, number] = [1, 2, 3];

// Tuple ที่ types ต่างกัน
let entry: [string, number] = ['Alice', 25];
let person: [string, number, boolean] = ['Bob', 30, true];

// Access elements
let [x, y] = coordinate;
let [personName, personAge] = entry;

// Named tuples (TypeScript 4.0+)
type RGB = [red: number, green: number, blue: number];
let color: RGB = [255, 128, 0];

// Optional tuple elements
type HttpResponse = [statusCode: number, body: string, headers?: Record<string, string>];
let response: HttpResponse = [200, '{"ok": true}'];
let responseWithHeaders: HttpResponse = [200, '{"ok": true}', { 'Content-Type': 'application/json' }];

// Rest elements ใน tuple
type StringsThenNumber = [...string[], number];
let data: StringsThenNumber = ['a', 'b', 'c', 42];

// ตัวอย่างจริง: useState ใน React
function useState<T>(initial: T): [T, (value: T) => void] {
  let state = initial;
  const setState = (value: T) => { state = value; };
  return [state, setState];
}

const [count, setCount] = useState(0);
// count: number, setCount: (value: number) => void
```

---

## Step 1357: Enums

```typescript
// ===== Numeric Enums =====

enum Direction {
  Up,     // 0
  Down,   // 1
  Left,   // 2
  Right,  // 3
}

let move: Direction = Direction.Up;
console.log(move);           // 0
console.log(Direction[0]);   // "Up" (reverse mapping)
console.log(Direction.Up);   // 0

// กำหนดค่าเริ่มต้น
enum StatusCode {
  OK = 200,
  Created = 201,
  BadRequest = 400,
  Unauthorized = 401,
  Forbidden = 403,
  NotFound = 404,
  InternalServerError = 500,
}

function handleResponse(code: StatusCode) {
  switch (code) {
    case StatusCode.OK:
      return 'Success';
    case StatusCode.NotFound:
      return 'Not Found';
    default:
      return 'Other';
  }
}

// ===== String Enums =====
enum Color {
  Red = 'RED',
  Green = 'GREEN',
  Blue = 'BLUE',
  Yellow = 'YELLOW',
}

enum UserRole {
  Admin = 'admin',
  Moderator = 'moderator',
  User = 'user',
  Guest = 'guest',
}

function checkPermission(role: UserRole): boolean {
  return role === UserRole.Admin || role === UserRole.Moderator;
}

// ===== Const Enums =====
// ถูก inline แทนที่จะเป็น object (เร็วกว่า)
const enum LogLevel {
  Debug = 0,
  Info = 1,
  Warn = 2,
  Error = 3,
}

const level: LogLevel = LogLevel.Info;
// Compiled to: const level = 1; (ไม่มี LogLevel object)

// ===== Enums vs Union Types =====
// ปัจจุบัน นิยมใช้ union types มากกว่า enums

// Enum approach
enum Status {
  Active = 'active',
  Inactive = 'inactive',
  Pending = 'pending',
}

// Union type approach (แนะนำ)
type Status2 = 'active' | 'inactive' | 'pending';

// Union type ง่ายกว่า และ interop ดีกว่ากับ API responses
const status: Status2 = 'active'; // ไม่ต้อง Status2.Active

// ===== Computed Enum Members =====
enum FilePermission {
  None = 0,
  Read = 1 << 0,    // 1
  Write = 1 << 1,   // 2
  Execute = 1 << 2, // 4
  ReadWrite = Read | Write, // 3
  All = Read | Write | Execute, // 7
}

function hasPermission(userPerm: FilePermission, required: FilePermission): boolean {
  return (userPerm & required) === required;
}

const perm = FilePermission.Read | FilePermission.Write;
console.log(hasPermission(perm, FilePermission.Read)); // true
console.log(hasPermission(perm, FilePermission.Execute)); // false
```

---

## Step 1358: Object Types และ Interfaces

```typescript
// ===== Object Types =====

// Inline object type
function greet(person: { name: string; age: number }): string {
  return `สวัสดี ${person.name} อายุ ${person.age} ปี`;
}

// ===== Interfaces =====

interface Point {
  x: number;
  y: number;
}

interface Rectangle {
  width: number;
  height: number;
}

interface User {
  id: number;
  username: string;
  email: string;
  role: 'admin' | 'user';
  createdAt: Date;
}

// ===== Optional Properties =====

interface Config {
  host: string;
  port: number;
  database: string;
  password?: string;    // Optional
  timeout?: number;     // Optional
  ssl?: boolean;        // Optional
}

const config: Config = {
  host: 'localhost',
  port: 5432,
  database: 'mydb',
  // password ไม่จำเป็นต้องระบุ
};

function connectDB(config: Config) {
  const { host, port, password = '', timeout = 30000 } = config;
  console.log(`Connecting to ${host}:${port}`);
}

// ===== Readonly Properties =====

interface ImmutablePoint {
  readonly x: number;
  readonly y: number;
}

const point: ImmutablePoint = { x: 10, y: 20 };
// point.x = 30; ❌ Error: Cannot assign to 'x' because it is a read-only property

// ===== Index Signatures =====

interface StringMap {
  [key: string]: string;
}

const translations: StringMap = {
  hello: 'สวัสดี',
  goodbye: 'ลาก่อน',
  thanks: 'ขอบคุณ',
};

interface NumberDictionary {
  [key: string]: number;
  length: number;    // ✅ ต้องเป็น number
  // name: string;  ❌ Error: ขัดแย้งกับ index signature
}

// ===== Function Properties ใน Interface =====

interface Calculator {
  add(a: number, b: number): number;
  subtract: (a: number, b: number) => number;
  multiply(a: number, b: number): number;
}

const calc: Calculator = {
  add: (a, b) => a + b,
  subtract: (a, b) => a - b,
  multiply: (a, b) => a * b,
};

// ===== Interface Extending =====

interface Animal {
  name: string;
  age: number;
  speak(): string;
}

interface Pet extends Animal {
  owner: string;
  vaccinated: boolean;
}

interface Dog extends Pet {
  breed: string;
  trainedCommands: string[];
}

const myDog: Dog = {
  name: 'Buddy',
  age: 3,
  owner: 'Alice',
  vaccinated: true,
  breed: 'Labrador',
  trainedCommands: ['sit', 'stay', 'fetch'],
  speak: () => 'Woof!',
};

// ===== Interface Merging =====

interface Window {
  title: string;
}

interface Window {
  myCustomProperty: string; // เพิ่ม property ให้ Window interface
}

// ตอนนี้ Window มีทั้ง title และ myCustomProperty

// ===== Callable Interfaces =====

interface Formatter {
  (value: string): string;
  locale: string;
}

const formatter: Formatter = Object.assign(
  (value: string) => value.toUpperCase(),
  { locale: 'th-TH' }
);

console.log(formatter('hello')); // HELLO
console.log(formatter.locale);  // th-TH
```

---

## Step 1359: Type Aliases

```typescript
// ===== Type Aliases =====
// ใช้ type แทน interface สำหรับ primitives, unions, tuples, etc.

// Primitive types
type ID = string | number;
type Timestamp = number;
type Email = string;
type URL = string;

// Object types
type Point = {
  x: number;
  y: number;
};

type UserProfile = {
  id: ID;
  name: string;
  email: Email;
  avatar: URL;
  createdAt: Timestamp;
};

// Functions
type Transformer<T> = (value: T) => T;
type AsyncTransformer<T> = (value: T) => Promise<T>;
type Predicate<T> = (value: T) => boolean;
type Comparator<T> = (a: T, b: T) => number;

// ตัวอย่างการใช้
const double: Transformer<number> = (n) => n * 2;
const isEven: Predicate<number> = (n) => n % 2 === 0;
const compareNumbers: Comparator<number> = (a, b) => a - b;

// Union types ที่ซับซ้อน
type APIResponse<T> =
  | { status: 'success'; data: T; }
  | { status: 'error'; message: string; code: number }
  | { status: 'loading' };

// ===== Type vs Interface =====

// Interface - ใช้สำหรับ objects ที่อาจมี inheritance
interface IAnimal {
  name: string;
  speak(): string;
}

// Type - ยืดหยุ่นกว่า, ใช้ได้กับ primitives, unions, tuples
type Animal = {
  name: string;
  speak(): string;
};

type StringOrNumber = string | number;
type Pair<T> = [T, T];
type Nullable<T> = T | null;
type Optional<T> = T | undefined;

// ===== Recursive Types =====

type JSONValue =
  | string
  | number
  | boolean
  | null
  | JSONValue[]
  | { [key: string]: JSONValue };

const data: JSONValue = {
  name: 'Alice',
  age: 25,
  hobbies: ['coding', 'reading'],
  address: {
    city: 'Bangkok',
    country: 'Thailand',
  },
  active: true,
  deleted: null,
};

// Tree structure
type TreeNode<T> = {
  value: T;
  children: TreeNode<T>[];
};

const tree: TreeNode<string> = {
  value: 'root',
  children: [
    {
      value: 'child1',
      children: [
        { value: 'grandchild1', children: [] },
        { value: 'grandchild2', children: [] },
      ],
    },
    { value: 'child2', children: [] },
  ],
};
```

---

## Step 1360: Union Types

```typescript
// ===== Union Types =====
// Type ที่เป็นได้หลาย types

type StringOrNumber = string | number;

function format(value: StringOrNumber): string {
  if (typeof value === 'string') {
    return value.toUpperCase();
  }
  return value.toFixed(2);
}

console.log(format('hello'));  // HELLO
console.log(format(3.14159)); // 3.14

// ===== Discriminated Unions =====
// Union ที่แต่ละ variant มี "discriminant" property

type Shape =
  | { kind: 'circle'; radius: number }
  | { kind: 'rectangle'; width: number; height: number }
  | { kind: 'triangle'; base: number; height: number };

function getArea(shape: Shape): number {
  switch (shape.kind) {
    case 'circle':
      return Math.PI * shape.radius ** 2;
    case 'rectangle':
      return shape.width * shape.height;
    case 'triangle':
      return (shape.base * shape.height) / 2;
  }
}

// ===== Union ใน function parameters =====

type HTTPMethod = 'GET' | 'POST' | 'PUT' | 'PATCH' | 'DELETE';

function makeRequest(url: string, method: HTTPMethod = 'GET') {
  return fetch(url, { method });
}

makeRequest('/api/users', 'GET');
makeRequest('/api/users', 'POST');
// makeRequest('/api/users', 'CONNECT'); ❌ Error

// ===== Optional ด้วย Union =====

type MaybeString = string | null | undefined;

function processName(name: MaybeString): string {
  if (name == null) {  // ตรวจสอบทั้ง null และ undefined
    return 'ไม่ทราบชื่อ';
  }
  return name.trim().toUpperCase();
}

// ===== ตัวอย่างจริง: API Response =====

interface ApiSuccess<T> {
  status: 'success';
  data: T;
  meta?: {
    total: number;
    page: number;
  };
}

interface ApiError {
  status: 'error';
  message: string;
  code: number;
  details?: Record<string, string[]>;
}

type ApiResponse<T> = ApiSuccess<T> | ApiError;

interface User {
  id: number;
  name: string;
  email: string;
}

async function fetchUser(id: number): Promise<ApiResponse<User>> {
  try {
    const response = await fetch(`/api/users/${id}`);
    const data = await response.json();
    
    if (!response.ok) {
      return {
        status: 'error',
        message: data.message || 'User not found',
        code: response.status,
      };
    }
    
    return {
      status: 'success',
      data: data as User,
    };
  } catch (error) {
    return {
      status: 'error',
      message: 'Network error',
      code: 0,
    };
  }
}

// การใช้งาน
async function displayUser(id: number) {
  const result = await fetchUser(id);
  
  if (result.status === 'success') {
    console.log(result.data.name); // ✅ TypeScript รู้ว่ามี data
  } else {
    console.error(result.message); // ✅ TypeScript รู้ว่ามี message
    console.error(result.code);
  }
}
```

---

## Step 1361: Intersection Types

```typescript
// ===== Intersection Types (&) =====
// รวม types หลายอันเข้าด้วยกัน

type HasName = { name: string };
type HasAge = { age: number };
type HasEmail = { email: string };

// Intersection: ต้องมีทุก property
type Person = HasName & HasAge;
type Contact = HasName & HasEmail;
type FullProfile = HasName & HasAge & HasEmail;

const person: Person = { name: 'Alice', age: 25 };
const contact: Contact = { name: 'Bob', email: 'bob@example.com' };
const profile: FullProfile = { name: 'Charlie', age: 30, email: 'charlie@example.com' };

// ===== Mixins Pattern =====

interface Serializable {
  serialize(): string;
  deserialize(data: string): void;
}

interface Auditable {
  createdAt: Date;
  updatedAt: Date;
  createdBy: string;
}

interface Validatable {
  validate(): boolean;
  errors: string[];
}

type BaseEntity<T> = T & Serializable & Auditable;

interface UserData {
  id: number;
  username: string;
  email: string;
}

type UserEntity = BaseEntity<UserData>;

// ===== Function Overloading ด้วย Intersection =====

type Logger = {
  (message: string): void;
  (message: string, level: 'info' | 'warn' | 'error'): void;
  prefix: string;
};

// ===== Intersection กับ Union =====
// Distributive property

type A = { x: number };
type B = { y: string };
type C = { z: boolean };

// (A | B) & C = (A & C) | (B & C)
type Distributed = (A | B) & C;
// เหมือนกับ: ({ x: number; z: boolean }) | ({ y: string; z: boolean })

// ===== Extending ด้วย Intersection =====

interface BaseUser {
  id: number;
  name: string;
}

type AdminUser = BaseUser & {
  role: 'admin';
  permissions: string[];
};

type RegularUser = BaseUser & {
  role: 'user';
  subscription: 'free' | 'premium';
};

type SystemUser = AdminUser | RegularUser;

function getUserGreeting(user: SystemUser): string {
  if (user.role === 'admin') {
    return `สวัสดี Admin ${user.name} คุณมี ${user.permissions.length} permissions`;
  }
  return `สวัสดี ${user.name} คุณใช้แผน ${user.subscription}`;
}
```

---

## Step 1362: Function Types

```typescript
// ===== Function Types =====

// Function type annotation
function add(a: number, b: number): number {
  return a + b;
}

// Arrow function
const subtract = (a: number, b: number): number => a - b;

// Type alias สำหรับ function
type MathOperation = (a: number, b: number) => number;

const multiply: MathOperation = (a, b) => a * b;
const divide: MathOperation = (a, b) => a / b;

// ===== Optional Parameters =====

function greet(name: string, greeting?: string): string {
  return `${greeting ?? 'สวัสดี'} ${name}`;
}

greet('Alice');              // สวัสดี Alice
greet('Bob', 'ดีจัง');       // ดีจัง Bob

// ===== Default Parameters =====

function createUser(
  name: string,
  role: 'admin' | 'user' = 'user',
  active: boolean = true
): object {
  return { name, role, active };
}

createUser('Alice');                    // { name: 'Alice', role: 'user', active: true }
createUser('Bob', 'admin');            // { name: 'Bob', role: 'admin', active: true }
createUser('Charlie', 'user', false);  // { name: 'Charlie', role: 'user', active: false }

// ===== Rest Parameters =====

function sum(...numbers: number[]): number {
  return numbers.reduce((acc, n) => acc + n, 0);
}

sum(1, 2, 3);         // 6
sum(1, 2, 3, 4, 5);   // 15

function logMessages(prefix: string, ...messages: string[]): void {
  messages.forEach(msg => console.log(`[${prefix}] ${msg}`));
}

// ===== Function Overloads =====

function format(value: string): string;
function format(value: number): string;
function format(value: boolean): string;
function format(value: string | number | boolean): string {
  if (typeof value === 'string') return `"${value}"`;
  if (typeof value === 'number') return value.toFixed(2);
  return value ? 'true' : 'false';
}

format('hello');   // "hello"
format(3.14159);   // 3.14
format(true);      // true

// ===== Higher-Order Functions =====

function pipe<T>(...fns: Array<(value: T) => T>): (value: T) => T {
  return (value: T) => fns.reduce((acc, fn) => fn(acc), value);
}

const processString = pipe<string>(
  (s) => s.trim(),
  (s) => s.toLowerCase(),
  (s) => s.replace(/\s+/g, '-'),
);

processString('  Hello World  '); // hello-world

// ===== this Parameters =====

interface EventEmitter {
  on(event: string, callback: (this: EventEmitter, data: unknown) => void): void;
}

class MyEmitter implements EventEmitter {
  on(event: string, callback: (this: MyEmitter, data: unknown) => void): void {
    // ...
  }
}
```

---

## Step 1363: Generic Functions

```typescript
// ===== Generics =====
// Functions/Types ที่ทำงานกับหลาย types

// ===== Generic Function =====

// ❌ ไม่ดี: ใช้ any ทำให้ type safety หาย
function identityAny(value: any): any {
  return value;
}

// ✅ ดี: Generic function
function identity<T>(value: T): T {
  return value;
}

const str = identity('hello');   // type: string
const num = identity(42);        // type: number
const arr = identity([1, 2, 3]); // type: number[]

// ===== Generic Arrays =====

function first<T>(array: T[]): T | undefined {
  return array[0];
}

function last<T>(array: T[]): T | undefined {
  return array[array.length - 1];
}

function reverse<T>(array: T[]): T[] {
  return [...array].reverse();
}

function chunk<T>(array: T[], size: number): T[][] {
  const chunks: T[][] = [];
  for (let i = 0; i < array.length; i += size) {
    chunks.push(array.slice(i, i + size));
  }
  return chunks;
}

// ===== Multiple Type Parameters =====

function zip<T, U>(array1: T[], array2: U[]): [T, U][] {
  const length = Math.min(array1.length, array2.length);
  return Array.from({ length }, (_, i) => [array1[i], array2[i]]);
}

const names = ['Alice', 'Bob', 'Charlie'];
const scores = [95, 87, 92];
const combined = zip(names, scores);
// type: [string, number][]
// value: [['Alice', 95], ['Bob', 87], ['Charlie', 92]]

function pair<T, U>(first: T, second: U): [T, U] {
  return [first, second];
}

// ===== Generic Constraints =====

// ต้องมี property length
function getLength<T extends { length: number }>(value: T): number {
  return value.length;
}

getLength('hello');         // 5 ✅
getLength([1, 2, 3]);      // 3 ✅
getLength({ length: 10 }); // 10 ✅
// getLength(42);           ❌ Error: number ไม่มี length

// keyof constraint
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const user = { id: 1, name: 'Alice', email: 'alice@example.com' };
getProperty(user, 'name');  // type: string ✅
getProperty(user, 'id');    // type: number ✅
// getProperty(user, 'phone'); ❌ Error: 'phone' ไม่ใช่ key ของ user

// ===== Generic Utilities =====

function mapValues<T, U>(obj: Record<string, T>, fn: (value: T) => U): Record<string, U> {
  const result: Record<string, U> = {};
  for (const [key, value] of Object.entries(obj)) {
    result[key] = fn(value);
  }
  return result;
}

const prices = { apple: 30, banana: 20, orange: 35 };
const discounted = mapValues(prices, price => price * 0.9);
// type: Record<string, number>

function filter<T>(array: T[], predicate: (item: T) => boolean): T[] {
  return array.filter(predicate);
}

function groupBy<T, K extends string | number>(
  array: T[],
  getKey: (item: T) => K
): Record<K, T[]> {
  return array.reduce((groups, item) => {
    const key = getKey(item);
    if (!groups[key]) groups[key] = [];
    groups[key].push(item);
    return groups;
  }, {} as Record<K, T[]>);
}

interface Product {
  name: string;
  category: string;
  price: number;
}

const products: Product[] = [
  { name: 'Laptop', category: 'Electronics', price: 45000 },
  { name: 'Phone', category: 'Electronics', price: 15000 },
  { name: 'Shirt', category: 'Clothing', price: 500 },
];

const byCategory = groupBy(products, p => p.category);
// type: Record<string, Product[]>
```

---

## Step 1364: Generic Interfaces

```typescript
// ===== Generic Interfaces =====

// Repository pattern
interface Repository<T, ID = number> {
  findById(id: ID): Promise<T | null>;
  findAll(options?: QueryOptions): Promise<T[]>;
  create(data: Omit<T, 'id' | 'createdAt' | 'updatedAt'>): Promise<T>;
  update(id: ID, data: Partial<T>): Promise<T | null>;
  delete(id: ID): Promise<boolean>;
  count(filter?: Partial<T>): Promise<number>;
}

interface QueryOptions {
  limit?: number;
  offset?: number;
  orderBy?: string;
  orderDirection?: 'asc' | 'desc';
}

interface User {
  id: number;
  name: string;
  email: string;
  createdAt: Date;
  updatedAt: Date;
}

// Concrete implementation
class UserRepository implements Repository<User> {
  async findById(id: number): Promise<User | null> {
    // fetch from database
    return null;
  }
  
  async findAll(options?: QueryOptions): Promise<User[]> {
    return [];
  }
  
  async create(data: Omit<User, 'id' | 'createdAt' | 'updatedAt'>): Promise<User> {
    return {
      id: Math.random(),
      ...data,
      createdAt: new Date(),
      updatedAt: new Date(),
    };
  }
  
  async update(id: number, data: Partial<User>): Promise<User | null> {
    return null;
  }
  
  async delete(id: number): Promise<boolean> {
    return true;
  }
  
  async count(filter?: Partial<User>): Promise<number> {
    return 0;
  }
}

// ===== Generic Stack, Queue =====

class Stack<T> {
  private items: T[] = [];
  
  push(item: T): void {
    this.items.push(item);
  }
  
  pop(): T | undefined {
    return this.items.pop();
  }
  
  peek(): T | undefined {
    return this.items[this.items.length - 1];
  }
  
  get size(): number {
    return this.items.length;
  }
  
  isEmpty(): boolean {
    return this.items.length === 0;
  }
  
  toArray(): T[] {
    return [...this.items];
  }
}

const numberStack = new Stack<number>();
numberStack.push(1);
numberStack.push(2);
numberStack.push(3);
console.log(numberStack.pop()); // 3

// ===== Generic Event Emitter =====

type EventMap = Record<string, unknown>;

class TypedEventEmitter<T extends EventMap> {
  private listeners = new Map<keyof T, Set<Function>>();
  
  on<K extends keyof T>(event: K, listener: (data: T[K]) => void): this {
    if (!this.listeners.has(event)) {
      this.listeners.set(event, new Set());
    }
    this.listeners.get(event)!.add(listener);
    return this;
  }
  
  off<K extends keyof T>(event: K, listener: (data: T[K]) => void): this {
    this.listeners.get(event)?.delete(listener);
    return this;
  }
  
  emit<K extends keyof T>(event: K, data: T[K]): void {
    this.listeners.get(event)?.forEach(listener => listener(data));
  }
}

// กำหนด events และ data types
interface AppEvents {
  'user:login': { userId: number; timestamp: Date };
  'user:logout': { userId: number };
  'cart:add': { productId: number; quantity: number };
  'cart:remove': { productId: number };
  'order:created': { orderId: number; total: number };
}

const emitter = new TypedEventEmitter<AppEvents>();

emitter.on('user:login', ({ userId, timestamp }) => {
  console.log(`User ${userId} logged in at ${timestamp}`);
});

emitter.emit('user:login', { userId: 1, timestamp: new Date() });
// emitter.emit('user:login', { userId: 'abc' }); ❌ Error: ต้องเป็น number
```

---

## Step 1365: Type Assertions

```typescript
// ===== Type Assertions =====
// บอก TypeScript ว่า type คืออะไร (ไม่ safe!)

// Syntax: as Type
const myCanvas = document.getElementById('canvas') as HTMLCanvasElement;
const ctx = myCanvas.getContext('2d');

// Syntax: <Type> (ไม่ทำงานใน TSX)
const input = <HTMLInputElement>document.getElementById('name-input');

// ===== ตัวอย่างการใช้งาน =====

// เมื่อ TypeScript ไม่รู้ type แน่ชัด
async function fetchUser(id: number): Promise<unknown> {
  const response = await fetch(`/api/users/${id}`);
  return response.json();
}

async function main() {
  const data = await fetchUser(1);
  
  // TypeScript ไม่รู้ว่า data มี property อะไร
  // data.name; ❌ Error
  
  // ✅ Type assertion
  const user = data as { id: number; name: string; email: string };
  console.log(user.name);
}

// ===== Double Assertion =====
// เมื่อ types ไม่ compatible กัน

const x = 'hello' as unknown as number; // ⚠️ อันตราย!

// ===== as const =====
// ทำให้ type เป็น literal type

const config = {
  host: 'localhost',
  port: 3000,
} as const;
// type: { readonly host: "localhost"; readonly port: 3000; }
// ไม่ใช่ { host: string; port: number; }

const colors = ['red', 'green', 'blue'] as const;
// type: readonly ["red", "green", "blue"]

// ✅ ใช้ได้ดีกับ discriminated unions
const ROUTES = {
  HOME: '/',
  PRODUCTS: '/products',
  CART: '/cart',
} as const;

type Route = typeof ROUTES[keyof typeof ROUTES]; // '/' | '/products' | '/cart'

function navigate(route: Route) {
  window.location.href = route;
}

navigate('/');          // ✅
navigate('/products');  // ✅
// navigate('/about');  ❌ Error
```

---

## Step 1366: Type Narrowing

```typescript
// ===== Type Narrowing =====
// วิธีที่ TypeScript ลด type ให้แคบลงจาก broad type

// ===== typeof narrowing =====

function processValue(value: string | number | boolean) {
  if (typeof value === 'string') {
    // value: string
    return value.toUpperCase();
  } else if (typeof value === 'number') {
    // value: number
    return value.toFixed(2);
  } else {
    // value: boolean
    return value ? 'Yes' : 'No';
  }
}

// ===== instanceof narrowing =====

class Dog {
  bark(): string { return 'Woof!'; }
}

class Cat {
  meow(): string { return 'Meow!'; }
}

function makeSound(animal: Dog | Cat): string {
  if (animal instanceof Dog) {
    return animal.bark(); // animal: Dog
  } else {
    return animal.meow(); // animal: Cat
  }
}

// ===== in narrowing =====

interface Fish {
  swim(): void;
}

interface Bird {
  fly(): void;
}

function move(animal: Fish | Bird): void {
  if ('swim' in animal) {
    animal.swim(); // animal: Fish
  } else {
    animal.fly(); // animal: Bird
  }
}

// ===== Equality narrowing =====

function getGreeting(lang: 'th' | 'en' | 'jp'): string {
  if (lang === 'th') {
    return 'สวัสดี';
  } else if (lang === 'en') {
    return 'Hello';
  } else {
    // lang: 'jp'
    return 'こんにちは';
  }
}

// ===== Truthiness narrowing =====

function processOptional(value: string | null | undefined): string {
  if (value) {
    // value: string (ไม่ใช่ null หรือ undefined หรือ '')
    return value.toUpperCase();
  }
  return 'default';
}

// ===== Discriminated Union Narrowing =====

type Result<T> =
  | { success: true; data: T }
  | { success: false; error: string; code: number };

function handleResult<T>(result: Result<T>): T | null {
  if (result.success) {
    return result.data;      // result: { success: true; data: T }
  } else {
    console.error(result.error, result.code); // result: { success: false; ... }
    return null;
  }
}

// ===== Never Narrowing (Exhaustive check) =====

type Direction = 'north' | 'south' | 'east' | 'west';

function getOpposite(dir: Direction): Direction {
  switch (dir) {
    case 'north': return 'south';
    case 'south': return 'north';
    case 'east': return 'west';
    case 'west': return 'east';
    default:
      const _exhaustive: never = dir;
      throw new Error(`Unknown direction: ${_exhaustive}`);
  }
}
```

---

## Step 1367: Type Predicates

```typescript
// ===== Type Predicates (is keyword) =====

// ===== User-defined type guards =====

interface Cat {
  meow(): void;
  purr(): void;
}

interface Dog {
  bark(): void;
  fetch(): void;
}

// Type predicate function
function isCat(animal: Cat | Dog): animal is Cat {
  return 'meow' in animal;
}

function isDog(animal: Cat | Dog): animal is Dog {
  return 'bark' in animal;
}

function makeSound(animal: Cat | Dog): void {
  if (isCat(animal)) {
    animal.meow();  // ✅ TypeScript รู้ว่าเป็น Cat
    animal.purr();  // ✅
  } else {
    animal.bark();  // ✅ TypeScript รู้ว่าเป็น Dog
    animal.fetch(); // ✅
  }
}

// ===== ตัวอย่างจริง =====

interface UserWithRole {
  id: number;
  name: string;
  role: 'admin' | 'user';
}

interface AdminUser extends UserWithRole {
  role: 'admin';
  adminLevel: number;
  permissions: string[];
}

function isAdmin(user: UserWithRole): user is AdminUser {
  return user.role === 'admin';
}

function getAdminPanel(user: UserWithRole | null | undefined) {
  if (!user) return null;
  
  if (isAdmin(user)) {
    // user: AdminUser
    return `Admin Level: ${user.adminLevel}`;
  }
  
  return 'ไม่มีสิทธิ์เข้าถึง';
}

// ===== Array type predicates =====

const mixed: (string | number | null)[] = ['hello', 1, null, 'world', 2, null];

function isString(value: string | number | null): value is string {
  return typeof value === 'string';
}

function isNumber(value: string | number | null): value is number {
  return typeof value === 'number';
}

// Filter ด้วย type predicate - TypeScript รู้ type ที่แน่นอน
const strings = mixed.filter(isString); // type: string[]
const numbers = mixed.filter(isNumber); // type: number[]

// ✅ TypeScript รู้ว่า strings เป็น string[]
strings.forEach(s => console.log(s.toUpperCase()));

// ===== Assertion Functions =====

function assertIsString(value: unknown): asserts value is string {
  if (typeof value !== 'string') {
    throw new Error(`Expected string, got ${typeof value}`);
  }
}

function assertIsDefined<T>(value: T | null | undefined): asserts value is T {
  if (value == null) {
    throw new Error('Value is null or undefined');
  }
}

function processInput(input: unknown) {
  assertIsString(input);
  // ตอนนี้ TypeScript รู้ว่า input เป็น string
  console.log(input.toUpperCase());
}

const element = document.getElementById('app');
assertIsDefined(element);
// ตอนนี้ element ไม่ใช่ null
element.innerHTML = 'Hello!';
```

---

## Step 1368: Non-null Assertion

```typescript
// ===== Non-null Assertion Operator (!) =====

// บอก TypeScript ว่า value ไม่ใช่ null หรือ undefined
// ⚠️ ใช้เมื่อ developer แน่ใจว่าไม่ null แต่ TypeScript ไม่รู้

// ตัวอย่าง: DOM elements
const button = document.getElementById('submit-btn')!;
// Type: HTMLElement (ไม่ใช่ HTMLElement | null)
button.addEventListener('click', handleClick);

// ตัวอย่าง: Map.get()
const map = new Map<string, number>();
map.set('key', 42);

// ❌ TypeScript บอกว่า value อาจเป็น undefined
const value = map.get('key'); // type: number | undefined
// value.toFixed(2); ❌ Error

// ✅ ใช้ ! เมื่อ sure ว่ามีค่า
const value2 = map.get('key')!; // type: number
value2.toFixed(2); // ✅

// ===== แนะนำ: ใช้ optional chaining แทน =====
// ปลอดภัยกว่า

const element = document.getElementById('app');

// ❌ อาจ crash ถ้า element เป็น null
element!.innerHTML = 'Hello'; 

// ✅ ปลอดภัยกว่า
element?.innerHTML; // ไม่ crash, คืน undefined ถ้า null

// หรือ nullish coalescing
const text = element?.textContent ?? 'default';

// ===== กรณีที่ ! เหมาะสม =====

// 1. ใน test code
test('renders correctly', () => {
  const { container } = render(<App />);
  const button = container.querySelector('button')!;
  expect(button.textContent).toBe('Click me');
});

// 2. เมื่อ invariant ที่ runtime ค้ำประกัน
class EventEmitter {
  private handlers = new Map<string, Function[]>();
  
  emit(event: string) {
    // เรียกเมื่อมั่นใจว่า handlers มีอยู่แน่ๆ
    const fns = this.handlers.get(event)!;
    fns.forEach(fn => fn());
  }
}

// ===== Combining Techniques =====

interface Config {
  apiUrl?: string;
  timeout?: number;
}

class ApiClient {
  private config: Required<Config>;
  
  constructor(config: Config) {
    // Validate required fields
    if (!config.apiUrl) throw new Error('apiUrl is required');
    
    this.config = {
      apiUrl: config.apiUrl,
      timeout: config.timeout ?? 30000,
    };
  }
  
  async get(path: string) {
    return fetch(`${this.config.apiUrl}${path}`, {
      signal: AbortSignal.timeout(this.config.timeout),
    });
  }
}
```

---

## Step 1369-1370: TypeScript ในโปรเจค Node.js จริง

```typescript
// ===== Complete TypeScript Example =====
// src/index.ts

import express, { Request, Response, NextFunction } from 'express';
import { z } from 'zod';

const app = express();
app.use(express.json());

// ===== Types/Interfaces =====

interface User {
  id: number;
  name: string;
  email: string;
  role: 'admin' | 'user';
  createdAt: Date;
}

interface CreateUserDTO {
  name: string;
  email: string;
  role?: 'admin' | 'user';
}

interface UpdateUserDTO {
  name?: string;
  email?: string;
  role?: 'admin' | 'user';
}

interface PaginationOptions {
  page: number;
  limit: number;
}

interface PaginatedResult<T> {
  data: T[];
  total: number;
  page: number;
  totalPages: number;
}

// ===== Validation Schemas =====

const createUserSchema = z.object({
  name: z.string().min(1).max(100),
  email: z.string().email(),
  role: z.enum(['admin', 'user']).optional().default('user'),
});

const paginationSchema = z.object({
  page: z.string().transform(Number).default('1'),
  limit: z.string().transform(Number).default('20'),
});

// ===== Repository =====

class UserRepository {
  private users: User[] = [];
  private nextId = 1;
  
  async findAll(options: PaginationOptions): Promise<PaginatedResult<User>> {
    const { page, limit } = options;
    const start = (page - 1) * limit;
    const end = start + limit;
    const data = this.users.slice(start, end);
    
    return {
      data,
      total: this.users.length,
      page,
      totalPages: Math.ceil(this.users.length / limit),
    };
  }
  
  async findById(id: number): Promise<User | null> {
    return this.users.find(u => u.id === id) ?? null;
  }
  
  async findByEmail(email: string): Promise<User | null> {
    return this.users.find(u => u.email === email) ?? null;
  }
  
  async create(data: CreateUserDTO): Promise<User> {
    const user: User = {
      id: this.nextId++,
      name: data.name,
      email: data.email,
      role: data.role ?? 'user',
      createdAt: new Date(),
    };
    this.users.push(user);
    return user;
  }
  
  async update(id: number, data: UpdateUserDTO): Promise<User | null> {
    const index = this.users.findIndex(u => u.id === id);
    if (index === -1) return null;
    
    this.users[index] = { ...this.users[index], ...data };
    return this.users[index];
  }
  
  async delete(id: number): Promise<boolean> {
    const index = this.users.findIndex(u => u.id === id);
    if (index === -1) return false;
    
    this.users.splice(index, 1);
    return true;
  }
}

// ===== Controller =====

const userRepo = new UserRepository();

app.get('/api/users', async (req: Request, res: Response) => {
  const pagination = paginationSchema.parse(req.query);
  const result = await userRepo.findAll(pagination);
  res.json(result);
});

app.get('/api/users/:id', async (req: Request, res: Response) => {
  const id = parseInt(req.params.id);
  if (isNaN(id)) {
    return res.status(400).json({ message: 'Invalid ID' });
  }
  
  const user = await userRepo.findById(id);
  if (!user) {
    return res.status(404).json({ message: 'User not found' });
  }
  
  res.json(user);
});

app.post('/api/users', async (req: Request, res: Response) => {
  const result = createUserSchema.safeParse(req.body);
  if (!result.success) {
    return res.status(400).json({ errors: result.error.errors });
  }
  
  const existingUser = await userRepo.findByEmail(result.data.email);
  if (existingUser) {
    return res.status(409).json({ message: 'Email already exists' });
  }
  
  const user = await userRepo.create(result.data);
  res.status(201).json(user);
});

// Error handler
app.use((err: Error, req: Request, res: Response, next: NextFunction) => {
  console.error(err.stack);
  res.status(500).json({ message: 'Internal Server Error' });
});

app.listen(3000, () => console.log('Server running on port 3000'));
```

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: Type Definitions
สร้าง types สำหรับ E-Commerce:
- `Product` (id, name, price, category, stock, images)
- `CartItem` (product, quantity)
- `Order` (id, items, total, status, shippingAddress)
- `Address` (street, city, province, postalCode, country)

### แบบฝึกหัดที่ 2: Generic Functions
สร้าง utility functions:
- `groupBy<T>` - group array ตาม key
- `sortBy<T>` - sort array ตาม key
- `paginate<T>` - แบ่งหน้า array

### แบบฝึกหัดที่ 3: Type Narrowing
สร้าง function `renderWidget` ที่รับ union type:
```typescript
type Widget =
  | { type: 'button'; label: string; onClick: () => void }
  | { type: 'input'; placeholder: string; value: string }
  | { type: 'image'; src: string; alt: string };
```

### แบบฝึกหัดที่ 4: Interfaces
สร้าง interface hierarchy สำหรับระบบบัญชี:
- `Account` (id, balance, currency)
- `BankAccount extends Account` (accountNumber, bankName, holder)
- `CryptoWallet extends Account` (address, blockchain)

### แบบฝึกหัดที่ 5: Complete TypeScript Project
สร้าง TypeScript class `TaskManager`:
- Tasks มี: id, title, description, status, priority, dueDate
- Methods: createTask, updateTask, deleteTask, getTasksByStatus, getOverdueTasks
- ใช้ generics, interfaces, และ type narrowing

---

## สรุป

| Feature | ตัวอย่าง |
|---------|----------|
| Basic Types | `string`, `number`, `boolean`, `any`, `unknown`, `never` |
| Arrays | `number[]`, `Array<string>` |
| Tuples | `[string, number]` |
| Enums | `enum Color { Red, Green, Blue }` |
| Interfaces | `interface User { id: number; name: string }` |
| Type Aliases | `type ID = string \| number` |
| Union Types | `string \| number \| null` |
| Intersection Types | `A & B` |
| Generics | `function identity<T>(value: T): T` |
| Type Predicates | `function isString(x: unknown): x is string` |
| Type Narrowing | `typeof`, `instanceof`, `in`, discriminated unions |
| Non-null | `element!.innerHTML` |
