# Part 70: TypeScript ขั้นสูง (Steps 1371-1390)

## บทนำ: TypeScript ขั้นสูง

หลังจากเรียนรู้พื้นฐาน TypeScript แล้ว ส่วนนี้จะครอบคลุมความสามารถขั้นสูงที่ช่วยให้เขียน type-safe code ได้อย่างยืดหยุ่นและทรงพลัง

---

## Step 1371: Mapped Types

Mapped Types ช่วยสร้าง type ใหม่จาก type เดิมโดยเปลี่ยน properties

```typescript
// ===== Mapped Types พื้นฐาน =====

interface User {
  id: number;
  name: string;
  email: string;
  age: number;
}

// ทำให้ทุก property เป็น optional
type Partial<T> = {
  [K in keyof T]?: T[K];
};

type PartialUser = Partial<User>;
// { id?: number; name?: string; email?: string; age?: number; }

// ทำให้ทุก property เป็น required
type Required<T> = {
  [K in keyof T]-?: T[K]; // -? ลบ optional
};

// ทำให้ทุก property เป็น readonly
type Readonly<T> = {
  readonly [K in keyof T]: T[K];
};

// ===== Custom Mapped Types =====

// Nullable - ทำให้ทุก property เป็น T | null
type Nullable<T> = {
  [K in keyof T]: T[K] | null;
};

type NullableUser = Nullable<User>;
// { id: number | null; name: string | null; ... }

// Stringify - แปลงทุก property เป็น string
type Stringify<T> = {
  [K in keyof T]: string;
};

// MakeOptional - ทำให้บาง properties เป็น optional
type MakeOptional<T, K extends keyof T> = Omit<T, K> & Partial<Pick<T, K>>;

type UserWithOptionalEmail = MakeOptional<User, 'email' | 'age'>;
// { id: number; name: string; email?: string; age?: number; }

// ===== Remapping Keys =====

// เปลี่ยน key names ด้วย as
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};

type UserGetters = Getters<User>;
// { getId: () => number; getName: () => string; getEmail: () => string; getAge: () => number; }

// เปลี่ยนเป็น setters
type Setters<T> = {
  [K in keyof T as `set${Capitalize<string & K>}`]: (value: T[K]) => void;
};

// Filter properties โดย type
type FilterByType<T, ValueType> = {
  [K in keyof T as T[K] extends ValueType ? K : never]: T[K];
};

interface Mixed {
  name: string;
  age: number;
  active: boolean;
  email: string;
  score: number;
}

type StringProps = FilterByType<Mixed, string>;
// { name: string; email: string; }

type NumberProps = FilterByType<Mixed, number>;
// { age: number; score: number; }

// ===== Practical Example: Form Types =====

type FormValue<T> = {
  [K in keyof T]: {
    value: T[K];
    error: string | null;
    touched: boolean;
  };
};

interface LoginForm {
  email: string;
  password: string;
  rememberMe: boolean;
}

type LoginFormState = FormValue<LoginForm>;
// {
//   email: { value: string; error: string | null; touched: boolean; };
//   password: { value: string; error: string | null; touched: boolean; };
//   rememberMe: { value: boolean; error: string | null; touched: boolean; };
// }

const formState: LoginFormState = {
  email: { value: '', error: null, touched: false },
  password: { value: '', error: null, touched: false },
  rememberMe: { value: false, error: null, touched: false },
};
```

---

## Step 1372: Conditional Types

```typescript
// ===== Conditional Types =====
// T extends U ? TrueType : FalseType

type IsString<T> = T extends string ? 'yes' : 'no';

type A = IsString<string>;  // 'yes'
type B = IsString<number>;  // 'no'
type C = IsString<'hello'>; // 'yes' (string literal extends string)

// ===== Practical Conditional Types =====

// ทำให้เป็น non-nullable
type NonNullable<T> = T extends null | undefined ? never : T;

type MaybeString = string | null | undefined;
type DefiniteString = NonNullable<MaybeString>; // string

// ===== infer keyword =====
// ดึง type จาก conditional type

// ดึง return type ของ function
type ReturnType<T extends (...args: any) => any> =
  T extends (...args: any) => infer R ? R : never;

function getUser(): { id: number; name: string } {
  return { id: 1, name: 'Alice' };
}

type UserType = ReturnType<typeof getUser>;
// { id: number; name: string }

// ดึง parameter types
type Parameters<T extends (...args: any) => any> =
  T extends (...args: infer P) => any ? P : never;

function createUser(name: string, age: number, role: 'admin' | 'user') {}
type CreateUserParams = Parameters<typeof createUser>;
// [name: string, age: number, role: 'admin' | 'user']

// ดึง first argument
type FirstArg<T extends (...args: any) => any> =
  T extends (first: infer F, ...rest: any) => any ? F : never;

type First = FirstArg<typeof createUser>; // string

// ดึง element type จาก array
type ElementType<T extends readonly unknown[]> =
  T extends readonly (infer E)[] ? E : never;

type NumbersElement = ElementType<number[]>; // number
type StringsElement = ElementType<string[]>; // string

// ดึง type จาก Promise
type UnwrapPromise<T> = T extends Promise<infer U> ? UnwrapPromise<U> : T;

type Str = UnwrapPromise<Promise<string>>;                  // string
type Num = UnwrapPromise<Promise<Promise<number>>>;         // number
type Complex = UnwrapPromise<Promise<Promise<User>>>;       // User

// ===== Distributive Conditional Types =====
// ถ้า T เป็น union, conditional type จะ distribute

type ToArray<T> = T extends any ? T[] : never;

type StringOrNumberArray = ToArray<string | number>;
// string[] | number[] (ไม่ใช่ (string | number)[])

// หยุด distribution ด้วย brackets
type ToArrayNonDist<T> = [T] extends [any] ? T[] : never;
type Mixed = ToArrayNonDist<string | number>;
// (string | number)[]

// ===== Complex Conditional Types =====

// Deep partial
type DeepPartial<T> = T extends object ? {
  [K in keyof T]?: DeepPartial<T[K]>;
} : T;

interface Config {
  database: {
    host: string;
    port: number;
    credentials: {
      username: string;
      password: string;
    };
  };
  server: {
    port: number;
    cors: {
      origins: string[];
    };
  };
}

type PartialConfig = DeepPartial<Config>;
// ทุก level เป็น optional

// Flatten union type
type Flatten<T> = T extends Array<infer Item> ? Item : T;

type FlatString = Flatten<string[]>;     // string
type FlatNumber = Flatten<number[]>;     // number
type PlainString = Flatten<string>;      // string (ไม่ใช่ array)
```

---

## Step 1373: Template Literal Types

```typescript
// ===== Template Literal Types =====
// TypeScript 4.1+

// Basic
type Greeting = `Hello, ${string}!`;
const g: Greeting = 'Hello, World!';     // ✅
// const bad: Greeting = 'Hi there';    ❌

// ===== ผสมกับ Union Types =====

type EventName = 'click' | 'focus' | 'blur' | 'change';
type HandlerName = `on${Capitalize<EventName>}`;
// 'onClick' | 'onFocus' | 'onBlur' | 'onChange'

type CSSProperty = 'margin' | 'padding' | 'border';
type CSSDirection = 'Top' | 'Right' | 'Bottom' | 'Left';
type CSSExpanded = `${CSSProperty}${CSSDirection}`;
// 'marginTop' | 'marginRight' | 'marginBottom' | 'marginLeft' |
// 'paddingTop' | 'paddingRight' | ... | 'borderLeft'

// ===== String Manipulation Types =====
// Uppercase, Lowercase, Capitalize, Uncapitalize

type Upper = Uppercase<'hello'>;          // 'HELLO'
type Lower = Lowercase<'WORLD'>;          // 'world'
type Cap = Capitalize<'typescript'>;      // 'Typescript'
type Uncap = Uncapitalize<'TypeScript'>;  // 'typeScript'

// ===== Practical: Route Types =====

type HttpMethod = 'get' | 'post' | 'put' | 'patch' | 'delete';
type Route = `/${string}`;
type ApiEndpoint = `${Uppercase<HttpMethod>} ${Route}`;
// 'GET /...' | 'POST /...' | 'PUT /...' | etc.

// ===== Getter/Setter Types =====

type Getters<T extends object> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};

type Setters<T extends object> = {
  [K in keyof T as `set${Capitalize<string & K>}`]: (value: T[K]) => void;
};

type Watchers<T extends object> = {
  [K in keyof T as `watch${Capitalize<string & K>}`]: (
    callback: (newValue: T[K], oldValue: T[K]) => void
  ) => () => void;
};

interface UserState {
  name: string;
  age: number;
  email: string;
}

type UserGetters = Getters<UserState>;
// { getName: () => string; getAge: () => number; getEmail: () => string }

type UserSetters = Setters<UserState>;
// { setName: (value: string) => void; setAge: (value: number) => void; ... }

type UserWatchers = Watchers<UserState>;
// { watchName: (callback: ...) => () => void; ... }

// ===== Path Types =====

// Deep paths ใน object (TypeScript 4.1+)
type PathImpl<T, K extends keyof T> =
  K extends string
    ? T[K] extends Record<string, any>
      ? T[K] extends ArrayLike<any>
        ? K | `${K}.${PathImpl<T[K], Exclude<keyof T[K], keyof any[]>>}`
        : K | `${K}.${PathImpl<T[K], keyof T[K]>}`
      : K
    : never;

type Path<T> = PathImpl<T, keyof T> | keyof T;

interface AppState {
  user: {
    profile: {
      name: string;
      email: string;
    };
    settings: {
      theme: 'light' | 'dark';
      language: string;
    };
  };
  products: {
    list: string[];
    selected: number | null;
  };
}

type AppPath = Path<AppState>;
// 'user' | 'products' | 'user.profile' | 'user.settings' | 
// 'user.profile.name' | 'user.profile.email' | ...

// ===== CSS-in-JS Types =====

type CSSValue = string | number;
type CSSProperties = {
  [K in keyof CSSStyleDeclaration]?: CSSValue;
};

type StyledProps<T extends string> = Record<T, CSSProperties>;
```

---

## Step 1374: Utility Types

```typescript
// ===== Built-in Utility Types =====

interface User {
  id: number;
  name: string;
  email: string;
  age: number;
  role: 'admin' | 'user';
  createdAt: Date;
  updatedAt: Date;
}

// ===== Partial<T> =====
// ทำให้ทุก property เป็น optional
type UserUpdate = Partial<User>;
// ใช้สำหรับ PATCH endpoints
async function updateUser(id: number, data: Partial<User>): Promise<User> {
  // data สามารถมีแค่บาง properties
  return fetch(`/api/users/${id}`, { method: 'PATCH', body: JSON.stringify(data) })
    .then(r => r.json());
}

// ===== Required<T> =====
// ทำให้ทุก property เป็น required
interface Options {
  timeout?: number;
  retries?: number;
  baseUrl?: string;
}

type RequiredOptions = Required<Options>;
// { timeout: number; retries: number; baseUrl: string; }

// ===== Readonly<T> =====
// ทำให้ทุก property เป็น readonly
type ReadonlyUser = Readonly<User>;
const user: ReadonlyUser = { id: 1, name: 'Alice', email: 'alice@example.com', age: 25, role: 'user', createdAt: new Date(), updatedAt: new Date() };
// user.name = 'Bob'; ❌ Error

// ===== Pick<T, K> =====
// เลือก subset ของ properties
type UserPreview = Pick<User, 'id' | 'name' | 'email'>;
// { id: number; name: string; email: string; }

type UserCard = Pick<User, 'id' | 'name' | 'role'>;
// { id: number; name: string; role: 'admin' | 'user'; }

// ===== Omit<T, K> =====
// ลบ properties ออก
type UserWithoutDates = Omit<User, 'createdAt' | 'updatedAt'>;
// { id: number; name: string; email: string; age: number; role: 'admin' | 'user'; }

type CreateUserDTO = Omit<User, 'id' | 'createdAt' | 'updatedAt'>;
// { name: string; email: string; age: number; role: 'admin' | 'user'; }

// ===== Exclude<T, U> =====
// ลบ types จาก union
type Status = 'active' | 'inactive' | 'deleted' | 'pending';
type ActiveStatus = Exclude<Status, 'deleted' | 'inactive'>;
// 'active' | 'pending'

type NonString = Exclude<string | number | boolean, string>;
// number | boolean

// ===== Extract<T, U> =====
// เอาเฉพาะ types ที่ match
type StringOrNumber = string | number | boolean;
type OnlyStringOrNumber = Extract<StringOrNumber, string | number>;
// string | number

// ===== NonNullable<T> =====
// ลบ null และ undefined
type MaybeString = string | null | undefined;
type DefiniteString = NonNullable<MaybeString>;
// string

// ===== ReturnType<T> =====
// ดึง return type ของ function
function getProduct() {
  return { id: 1, name: 'Laptop', price: 45000 };
}

type Product = ReturnType<typeof getProduct>;
// { id: number; name: string; price: number; }

// ===== Parameters<T> =====
// ดึง parameter types
function createUser(name: string, age: number, role: 'admin' | 'user') {}

type CreateUserParams = Parameters<typeof createUser>;
// [name: string, age: number, role: 'admin' | 'user']

// ===== ConstructorParameters<T> =====
class ApiClient {
  constructor(
    private baseUrl: string,
    private timeout: number = 30000,
    private retries: number = 3,
  ) {}
}

type ApiClientParams = ConstructorParameters<typeof ApiClient>;
// [baseUrl: string, timeout?: number, retries?: number]

// ===== InstanceType<T> =====
// ดึง type ที่ได้จาก new Class()
type ApiClientInstance = InstanceType<typeof ApiClient>;
// ApiClient

// ===== Awaited<T> =====
// TypeScript 4.5+ - unwrap Promise type
type AsyncUser = Promise<Promise<User>>;
type UnwrappedUser = Awaited<AsyncUser>; // User

async function fetchUsers(): Promise<User[]> {
  return [];
}

type UsersArray = Awaited<ReturnType<typeof fetchUsers>>;
// User[]

// ===== Combining Utility Types =====

// Create Read-only version without internal fields
type PublicUser = Readonly<Omit<User, 'updatedAt' | 'createdAt'>>;

// Optional updates without id
type UserPatch = Partial<Omit<User, 'id' | 'createdAt' | 'updatedAt'>>;

// Required subset
type RequiredProfile = Required<Pick<User, 'name' | 'email'>>;
```

---

## Step 1375: Decorators

```typescript
// ===== Decorators (Experimental) =====
// ต้องเปิด experimentalDecorators ใน tsconfig.json

// ===== Class Decorators =====

function Singleton<T extends { new(...args: any[]): {} }>(constructor: T) {
  let instance: T;
  
  return class extends constructor {
    constructor(...args: any[]) {
      if (instance) return instance;
      super(...args);
      instance = this as any;
    }
  };
}

@Singleton
class DatabaseConnection {
  private connection: any;
  
  connect(url: string) {
    this.connection = { url };
    console.log(`Connected to ${url}`);
  }
}

const db1 = new DatabaseConnection();
const db2 = new DatabaseConnection();
console.log(db1 === db2); // true

// Logging decorator
function Log(constructor: Function) {
  const originalMethods = Object.getOwnPropertyNames(constructor.prototype);
  
  originalMethods.forEach(method => {
    if (method === 'constructor') return;
    
    const original = constructor.prototype[method];
    constructor.prototype[method] = function(...args: any[]) {
      console.log(`Calling ${constructor.name}.${method} with:`, args);
      const result = original.apply(this, args);
      console.log(`${constructor.name}.${method} returned:`, result);
      return result;
    };
  });
}

// ===== Method Decorators =====

function Memoize(
  target: any,
  propertyKey: string,
  descriptor: PropertyDescriptor
) {
  const originalMethod = descriptor.value;
  const cache = new Map();
  
  descriptor.value = function(...args: any[]) {
    const key = JSON.stringify(args);
    
    if (cache.has(key)) {
      console.log(`Cache hit for ${propertyKey}(${key})`);
      return cache.get(key);
    }
    
    const result = originalMethod.apply(this, args);
    cache.set(key, result);
    return result;
  };
  
  return descriptor;
}

function Validate(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalMethod = descriptor.value;
  
  descriptor.value = function(...args: any[]) {
    // Validate args
    args.forEach((arg, i) => {
      if (arg === null || arg === undefined) {
        throw new Error(`Argument ${i} of ${propertyKey} cannot be null/undefined`);
      }
    });
    
    return originalMethod.apply(this, args);
  };
  
  return descriptor;
}

class Calculator {
  @Memoize
  expensiveOperation(n: number): number {
    console.log(`Computing for ${n}...`);
    return n * n * n;
  }
  
  @Validate
  divide(a: number, b: number): number {
    if (b === 0) throw new Error('Division by zero');
    return a / b;
  }
}

// ===== Property Decorators =====

function MinLength(min: number) {
  return function(target: any, propertyKey: string) {
    let value: string;
    
    Object.defineProperty(target, propertyKey, {
      get: () => value,
      set: (newValue: string) => {
        if (newValue.length < min) {
          throw new Error(`${propertyKey} must be at least ${min} characters`);
        }
        value = newValue;
      },
    });
  };
}

function MaxValue(max: number) {
  return function(target: any, propertyKey: string) {
    let value: number;
    
    Object.defineProperty(target, propertyKey, {
      get: () => value,
      set: (newValue: number) => {
        if (newValue > max) {
          throw new Error(`${propertyKey} cannot exceed ${max}`);
        }
        value = newValue;
      },
    });
  };
}

class User {
  @MinLength(3)
  username: string = '';
  
  @MaxValue(120)
  age: number = 0;
}

// ===== Parameter Decorators =====

function Required(target: any, propertyKey: string, parameterIndex: number) {
  const requiredParams: number[] = Reflect.getMetadata('required', target, propertyKey) || [];
  requiredParams.push(parameterIndex);
  Reflect.defineMetadata('required', requiredParams, target, propertyKey);
}
```

---

## Step 1376: Declaration Merging

```typescript
// ===== Declaration Merging =====
// TypeScript รวม multiple declarations ของ name เดียวกัน

// ===== Interface Merging =====

interface Validator {
  validate(value: string): boolean;
}

interface Validator {
  validate(value: number): boolean; // เพิ่ม overload
  getErrors(): string[];             // เพิ่ม method ใหม่
}

// ตอนนี้ Validator มีทั้งหมด:
// validate(value: string): boolean
// validate(value: number): boolean
// getErrors(): string[]

// ===== Namespace Merging =====

function createLogger(name: string): Logger {
  return { log: (msg) => console.log(`[${name}] ${msg}`) };
}

namespace createLogger {
  export function debug(msg: string) {
    console.debug(`[DEBUG] ${msg}`);
  }
  
  export function warn(msg: string) {
    console.warn(`[WARN] ${msg}`);
  }
}

// ตอนนี้ createLogger เป็นทั้ง function และ namespace
const logger = createLogger('App');
createLogger.debug('Test message');

// ===== Module Augmentation =====
// เพิ่ม types ให้ existing modules

// หรือในไฟล์แยก: augments.d.ts
declare module 'express' {
  interface Request {
    user?: {
      id: number;
      email: string;
      role: 'admin' | 'user';
    };
    sessionId?: string;
  }
}

// ตอนนี้ req.user จะมี type ที่กำหนด
import { Request, Response } from 'express';

function authMiddleware(req: Request, res: Response, next: Function) {
  req.user = { id: 1, email: 'user@example.com', role: 'user' };
  next();
}

function getUserId(req: Request): number | undefined {
  return req.user?.id; // ✅ TypeScript รู้ว่า req.user มี id
}

// ===== Global Augmentation =====

declare global {
  interface Window {
    myApp: {
      version: string;
      config: Record<string, unknown>;
    };
  }
  
  interface Array<T> {
    groupBy<K extends string | number>(
      keyFn: (item: T) => K
    ): Record<K, T[]>;
  }
}

// ตอนนี้ TypeScript รู้จัก window.myApp
window.myApp = { version: '1.0.0', config: {} };

// และ Array.groupBy
[1, 2, 3, 4, 5].groupBy(n => n % 2 === 0 ? 'even' : 'odd');
```

---

## Step 1377: Declaration Files (.d.ts)

```typescript
// ===== Declaration Files =====
// ไฟล์ที่บอก TypeScript เกี่ยวกับ types ของ JavaScript libraries

// ===== สร้าง .d.ts สำหรับ JS Library =====

// lib/math-utils.js (JavaScript library)
function add(a, b) { return a + b; }
function formatCurrency(amount, currency) { return `${currency}${amount}`; }

// lib/math-utils.d.ts (Declaration file)
declare function add(a: number, b: number): number;
declare function formatCurrency(amount: number, currency?: string): string;

// ===== Module Declaration =====

// หากมี module ที่ไม่มี type definitions
// declare module 'some-js-library' {
//   export function doSomething(value: string): void;
//   export const version: string;
//   export default class Client { ... }
// }

// ===== Ambient Declarations =====

// Global variables
declare const __DEV__: boolean;
declare const __VERSION__: string;
declare const process: {
  env: {
    NODE_ENV: 'development' | 'production' | 'test';
    [key: string]: string | undefined;
  };
  exit(code?: number): never;
};

// Global functions
declare function require(module: string): any;

// ===== Complex Declaration Example =====

// สมมติ library มี API แบบนี้
// const client = new APIClient({ baseUrl: 'https://api.example.com' });
// client.get('/users').then(data => console.log(data));
// client.post('/users', { name: 'Alice' });

declare class APIClient {
  constructor(config: APIClientConfig);
  
  get<T = unknown>(path: string, options?: RequestOptions): Promise<T>;
  post<T = unknown>(path: string, data?: unknown, options?: RequestOptions): Promise<T>;
  put<T = unknown>(path: string, data?: unknown, options?: RequestOptions): Promise<T>;
  patch<T = unknown>(path: string, data?: unknown, options?: RequestOptions): Promise<T>;
  delete<T = unknown>(path: string, options?: RequestOptions): Promise<T>;
  
  setHeader(key: string, value: string): void;
  removeHeader(key: string): void;
}

declare interface APIClientConfig {
  baseUrl: string;
  timeout?: number;
  headers?: Record<string, string>;
  retries?: number;
}

declare interface RequestOptions {
  headers?: Record<string, string>;
  timeout?: number;
  signal?: AbortSignal;
}

declare module 'api-client' {
  export = APIClient;
}

// ===== Writing Type Definitions สำหรับ Common Patterns =====

// Observer Pattern
declare interface Observable<T> {
  subscribe(observer: Observer<T>): Subscription;
  pipe<R>(...operators: OperatorFn[]): Observable<R>;
}

declare interface Observer<T> {
  next(value: T): void;
  error?(err: unknown): void;
  complete?(): void;
}

declare interface Subscription {
  unsubscribe(): void;
  closed: boolean;
}

type OperatorFn = (source: Observable<any>) => Observable<any>;

// Plugin Pattern
declare interface Plugin {
  name: string;
  version: string;
  install(app: Application, options?: Record<string, unknown>): void;
}

declare interface Application {
  use(plugin: Plugin, options?: Record<string, unknown>): this;
  component(name: string, component: unknown): this;
  config: ApplicationConfig;
}

declare interface ApplicationConfig {
  globalProperties: Record<string, unknown>;
  errorHandler?: (err: unknown, instance: unknown, info: string) => void;
}
```

---

## Step 1378: infer Keyword ขั้นสูง

```typescript
// ===== Advanced infer =====

// ===== Infer ใน Nested Types =====

// ดึง value type จาก Map
type MapValue<T> = T extends Map<any, infer V> ? V : never;
type ValueOfStringMap = MapValue<Map<string, number>>; // number

// ดึง type จาก Set
type SetValue<T> = T extends Set<infer V> ? V : never;
type SetElement = SetValue<Set<string>>; // string

// ดึง type จาก Promise (recursive)
type DeepAwaited<T> = T extends Promise<infer U> ? DeepAwaited<U> : T;
type Deep = DeepAwaited<Promise<Promise<Promise<string>>>>; // string

// ===== Infer กับ Function Types =====

// ดึง last argument
type LastArg<T extends (...args: any) => any> =
  T extends (...args: [...infer Init, infer Last]) => any ? Last : never;

function fn(a: string, b: number, c: boolean) {}
type Last = LastArg<typeof fn>; // boolean

// Curry types
type Curried<T extends (...args: any) => any> =
  T extends (first: infer F, ...rest: infer R) => infer Return
    ? R extends []
      ? (arg: F) => Return
      : (arg: F) => Curried<(...args: R) => Return>
    : T;

// ===== Infer กับ Tuple Types =====

// Reverse tuple
type Reverse<T extends any[]> =
  T extends [infer First, ...infer Rest]
    ? [...Reverse<Rest>, First]
    : T;

type Rev = Reverse<[1, 2, 3]>; // [3, 2, 1]

// ===== Real-world: Middleware Types =====

type Middleware<T extends object = {}> = (
  context: T,
  next: () => Promise<void>
) => Promise<void>;

type MiddlewareStack<T extends object = {}> = Middleware<T>[];

// Pipeline type
type PipelineResult<T, Fns extends ((input: any) => any)[]> =
  Fns extends []
    ? T
    : Fns extends [(input: infer A) => infer B, ...infer Rest]
      ? Rest extends ((input: any) => any)[]
        ? PipelineResult<B, Rest>
        : B
      : T;
```

---

## Step 1379: Variadic Tuple Types

```typescript
// ===== Variadic Tuple Types (TypeScript 4.0+) =====

// Spread ใน tuple
type Strings = [string, ...string[]];
type NumbersThenString = [...number[], string];
type StringsAndNumber = [string, ...string[], number];

// ===== Concat Types =====

type Concat<T extends unknown[], U extends unknown[]> = [...T, ...U];

type Numbers = [1, 2, 3];
type Letters = ['a', 'b', 'c'];
type Combined = Concat<Numbers, Letters>; // [1, 2, 3, 'a', 'b', 'c']

// ===== Function Composition =====

type Head<T extends unknown[]> = T extends [infer H, ...any[]] ? H : never;
type Tail<T extends unknown[]> = T extends [any, ...infer T] ? T : never;

// Type-safe pipe
type Pipe<Fns extends ((arg: any) => any)[]> =
  Fns extends []
    ? never
    : Fns extends [infer F]
      ? F
      : Fns extends [(arg: infer A) => infer B, ...infer Rest]
        ? Rest extends ((arg: B) => any)[]
          ? (arg: A) => ReturnType<Last<Rest>>
          : never
        : never;

type Last<T extends any[]> = T extends [...any[], infer L] ? L : never;

// ===== Spread Parameters =====

function merge<T extends object[]>(...objects: T): UnionToIntersection<T[number]> {
  return Object.assign({}, ...objects) as any;
}

type UnionToIntersection<U> =
  (U extends any ? (k: U) => void : never) extends (k: infer I) => void ? I : never;

const merged = merge(
  { name: 'Alice' },
  { age: 25 },
  { email: 'alice@example.com' }
);
// type: { name: string } & { age: number } & { email: string }

// ===== Typed printf สำหรับ format strings =====

type ParseFormat<T extends string> =
  T extends `${string}%s${infer Rest}`
    ? [string, ...ParseFormat<Rest>]
    : T extends `${string}%d${infer Rest}`
      ? [number, ...ParseFormat<Rest>]
      : [];

function printf<T extends string>(format: T, ...args: ParseFormat<T>): string {
  let result = format;
  let i = 0;
  return result.replace(/%[sd]/g, () => String(args[i++]));
}

// ✅ ต้องใส่ args ตาม format
printf('Hello %s, you are %d years old', 'Alice', 25);
// printf('Hello %s', 42); ❌ Error: expected string, got number
```

---

## Step 1380: Recursive Types

```typescript
// ===== Recursive Types =====

// ===== JSON Type =====
type JSONPrimitive = string | number | boolean | null;
type JSONValue = JSONPrimitive | JSONObject | JSONArray;
type JSONObject = { [key: string]: JSONValue };
type JSONArray = JSONValue[];

// ===== Deep Readonly =====
type DeepReadonly<T> =
  T extends (infer R)[]
    ? DeepReadonlyArray<R>
    : T extends object
      ? DeepReadonlyObject<T>
      : T;

type DeepReadonlyArray<T> = ReadonlyArray<DeepReadonly<T>>;
type DeepReadonlyObject<T> = {
  readonly [K in keyof T]: DeepReadonly<T[K]>;
};

interface Config {
  database: {
    host: string;
    port: number;
    credentials: {
      username: string;
      password: string;
    };
  };
}

type ReadonlyConfig = DeepReadonly<Config>;
const config: ReadonlyConfig = {
  database: {
    host: 'localhost',
    port: 5432,
    credentials: {
      username: 'admin',
      password: 'secret',
    },
  },
};

// config.database.host = 'other'; ❌ Error
// config.database.credentials.password = 'new'; ❌ Error

// ===== Deep Required =====
type DeepRequired<T> = {
  [K in keyof T]-?: T[K] extends object ? DeepRequired<T[K]> : T[K];
};

// ===== Path ใน object =====
type Paths<T, K extends keyof T = keyof T> =
  K extends string
    ? T[K] extends Record<string, any>
      ? `${K}` | `${K}.${Paths<T[K]>}`
      : `${K}`
    : never;

interface AppState {
  user: {
    name: string;
    address: {
      city: string;
      zip: string;
    };
  };
  theme: 'light' | 'dark';
}

type AppPaths = Paths<AppState>;
// 'user' | 'user.name' | 'user.address' | 'user.address.city' | 'user.address.zip' | 'theme'

// ===== Get type at path =====
type Get<T, P extends string> =
  P extends `${infer K}.${infer Rest}`
    ? K extends keyof T
      ? Get<T[K], Rest>
      : never
    : P extends keyof T
      ? T[P]
      : never;

type UserName = Get<AppState, 'user.name'>; // string
type CityType = Get<AppState, 'user.address.city'>; // string
type ThemeType = Get<AppState, 'theme'>; // 'light' | 'dark'

// ===== Flatten Type =====
type Flatten<T> =
  T extends Array<infer Item>
    ? Flatten<Item>
    : T;

type NestedArray = number[][][];
type FlatNumber = Flatten<NestedArray>; // number
```

---

## Step 1381: TypeScript กับ Express

```typescript
// ===== TypeScript + Express =====
// npm install express @types/express

import express, { Request, Response, NextFunction } from 'express';

// ===== Extend Express Request =====

declare global {
  namespace Express {
    interface Request {
      user?: AuthUser;
      requestId: string;
    }
  }
}

interface AuthUser {
  id: number;
  email: string;
  role: 'admin' | 'user';
}

// ===== Typed Request/Response =====

interface TypedRequest<Body = {}, Params = {}, Query = {}> extends Request {
  body: Body;
  params: Params & Record<string, string>;
  query: Query & Record<string, string | string[]>;
}

// ===== Controller Types =====

type AsyncController = (req: Request, res: Response, next: NextFunction) => Promise<void>;
type SyncController = (req: Request, res: Response, next: NextFunction) => void;
type Controller = AsyncController | SyncController;

// Wrapper สำหรับ async error handling
function asyncHandler(fn: AsyncController): Controller {
  return (req, res, next) => {
    Promise.resolve(fn(req, res, next)).catch(next);
  };
}

// ===== Typed Route Handlers =====

interface CreateUserBody {
  name: string;
  email: string;
  password: string;
  role?: 'admin' | 'user';
}

interface UpdateUserBody {
  name?: string;
  email?: string;
}

interface UserParams {
  id: string;
}

interface UserQuery {
  page?: string;
  limit?: string;
  search?: string;
}

// Route handlers ที่ type-safe
const createUserHandler = asyncHandler(async (
  req: TypedRequest<CreateUserBody>,
  res: Response
) => {
  const { name, email, password, role = 'user' } = req.body;
  
  const user = await userService.create({ name, email, password, role });
  res.status(201).json({ status: 'success', data: user });
});

const getUserHandler = asyncHandler(async (
  req: TypedRequest<{}, UserParams, UserQuery>,
  res: Response
) => {
  const { id } = req.params;
  const user = await userService.findById(parseInt(id));
  
  if (!user) {
    return res.status(404).json({ status: 'error', message: 'User not found' });
  }
  
  res.json({ status: 'success', data: user });
});

// ===== Middleware Types =====

type AuthMiddleware = (
  req: Request,
  res: Response,
  next: NextFunction
) => Promise<void> | void;

const authenticate: AuthMiddleware = async (req, res, next) => {
  const token = req.headers.authorization?.split(' ')[1];
  
  if (!token) {
    return res.status(401).json({ message: 'No token provided' });
  }
  
  try {
    const decoded = jwt.verify(token, process.env.JWT_SECRET!) as AuthUser;
    req.user = decoded;
    next();
  } catch {
    res.status(401).json({ message: 'Invalid token' });
  }
};

const authorize = (...roles: AuthUser['role'][]): AuthMiddleware =>
  (req, res, next) => {
    if (!req.user) {
      return res.status(401).json({ message: 'Not authenticated' });
    }
    
    if (!roles.includes(req.user.role)) {
      return res.status(403).json({ message: 'Insufficient permissions' });
    }
    
    next();
  };

// ===== Router Factory =====

function createRouter() {
  const router = express.Router();
  
  router.get('/', asyncHandler(getUserListHandler));
  router.get('/:id', asyncHandler(getUserHandler));
  router.post('/', authenticate, asyncHandler(createUserHandler));
  router.put('/:id', authenticate, authorize('admin'), asyncHandler(updateUserHandler));
  router.delete('/:id', authenticate, authorize('admin'), asyncHandler(deleteUserHandler));
  
  return router;
}

// ===== Error Types =====

class HttpError extends Error {
  constructor(
    public statusCode: number,
    message: string,
    public details?: unknown
  ) {
    super(message);
    this.name = 'HttpError';
  }
}

class ValidationError extends HttpError {
  constructor(errors: Record<string, string[]>) {
    super(400, 'Validation failed', errors);
    this.name = 'ValidationError';
  }
}

class NotFoundError extends HttpError {
  constructor(resource: string, id: string | number) {
    super(404, `${resource} with id ${id} not found`);
    this.name = 'NotFoundError';
  }
}

class UnauthorizedError extends HttpError {
  constructor(message = 'Unauthorized') {
    super(401, message);
    this.name = 'UnauthorizedError';
  }
}

// Error handler
const errorHandler = (
  err: Error,
  req: Request,
  res: Response,
  next: NextFunction
): void => {
  if (err instanceof HttpError) {
    res.status(err.statusCode).json({
      status: 'error',
      message: err.message,
      details: err.details,
    });
    return;
  }
  
  console.error(err.stack);
  res.status(500).json({
    status: 'error',
    message: 'Internal Server Error',
  });
};
```

---

## Step 1382: TypeScript กับ React

```typescript
// ===== TypeScript + React =====
// npm install react @types/react react-dom @types/react-dom

import React, { useState, useEffect, useRef, useCallback, useMemo, FC, ReactNode } from 'react';

// ===== Component Types =====

// Function Component
interface ButtonProps {
  label: string;
  onClick: () => void;
  variant?: 'primary' | 'secondary' | 'danger';
  disabled?: boolean;
  loading?: boolean;
  className?: string;
  children?: ReactNode;
}

const Button: FC<ButtonProps> = ({
  label,
  onClick,
  variant = 'primary',
  disabled = false,
  loading = false,
  className,
  children,
}) => {
  return (
    <button
      onClick={onClick}
      disabled={disabled || loading}
      className={`btn btn-${variant} ${className || ''}`}
    >
      {loading ? 'Loading...' : (children || label)}
    </button>
  );
};

// ===== Hooks Types =====

// useState
const [count, setCount] = useState<number>(0);
const [user, setUser] = useState<User | null>(null);
const [items, setItems] = useState<string[]>([]);

// useRef
const inputRef = useRef<HTMLInputElement>(null);
const timerRef = useRef<NodeJS.Timeout | null>(null);

function focusInput() {
  inputRef.current?.focus();
}

// useEffect
useEffect(() => {
  const subscription = dataSource.subscribe(setData);
  return () => subscription.unsubscribe();
}, [dataSource]);

// useCallback
const handleClick = useCallback((id: number) => {
  onSelect(id);
}, [onSelect]);

// useMemo
const filteredItems = useMemo(() => {
  return items.filter(item => item.includes(searchTerm));
}, [items, searchTerm]);

// ===== Custom Hooks =====

interface UseApiResult<T> {
  data: T | null;
  loading: boolean;
  error: Error | null;
  refetch: () => void;
}

function useApi<T>(url: string): UseApiResult<T> {
  const [data, setData] = useState<T | null>(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);
  
  const fetchData = useCallback(async () => {
    setLoading(true);
    setError(null);
    
    try {
      const response = await fetch(url);
      if (!response.ok) throw new Error(`HTTP error: ${response.status}`);
      const json: T = await response.json();
      setData(json);
    } catch (err) {
      setError(err instanceof Error ? err : new Error('Unknown error'));
    } finally {
      setLoading(false);
    }
  }, [url]);
  
  useEffect(() => {
    fetchData();
  }, [fetchData]);
  
  return { data, loading, error, refetch: fetchData };
}

// ใช้งาน
function UserList() {
  const { data: users, loading, error, refetch } = useApi<User[]>('/api/users');
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  if (!users) return null;
  
  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}

// ===== Generic Components =====

interface ListProps<T> {
  items: T[];
  renderItem: (item: T, index: number) => ReactNode;
  keyExtractor: (item: T) => string | number;
  emptyComponent?: ReactNode;
}

function List<T>({ items, renderItem, keyExtractor, emptyComponent }: ListProps<T>) {
  if (items.length === 0) {
    return <>{emptyComponent || <p>No items</p>}</>;
  }
  
  return (
    <ul>
      {items.map((item, index) => (
        <li key={keyExtractor(item)}>
          {renderItem(item, index)}
        </li>
      ))}
    </ul>
  );
}

// ใช้งาน
<List
  items={users}
  keyExtractor={user => user.id}
  renderItem={user => <span>{user.name}</span>}
  emptyComponent={<p>ไม่มีผู้ใช้</p>}
/>

// ===== Context Types =====

interface AuthContextValue {
  user: User | null;
  login: (email: string, password: string) => Promise<void>;
  logout: () => void;
  isAuthenticated: boolean;
  isLoading: boolean;
}

const AuthContext = React.createContext<AuthContextValue | null>(null);

function useAuth(): AuthContextValue {
  const context = React.useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth ต้องใช้ใน AuthProvider');
  }
  return context;
}

function AuthProvider({ children }: { children: ReactNode }) {
  const [user, setUser] = useState<User | null>(null);
  const [isLoading, setIsLoading] = useState(true);
  
  const login = async (email: string, password: string) => {
    const response = await fetch('/api/auth/login', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ email, password }),
    });
    
    if (!response.ok) throw new Error('Login failed');
    
    const { user } = await response.json();
    setUser(user);
  };
  
  const logout = () => {
    setUser(null);
    fetch('/api/auth/logout', { method: 'POST' });
  };
  
  return (
    <AuthContext.Provider value={{
      user,
      login,
      logout,
      isAuthenticated: user !== null,
      isLoading,
    }}>
      {children}
    </AuthContext.Provider>
  );
}
```

---

## Step 1383: Strict Mode tsconfig Options

```json
{
  "compilerOptions": {
    // ===== Strict Mode (รวมทุก options ด้านล่าง) =====
    "strict": true,
    
    // ห้าม type any โดยปริยาย
    // "noImplicitAny": true,
    
    // null/undefined ต้องตรวจสอบก่อนใช้
    // "strictNullChecks": true,
    
    // Function types เข้มงวด
    // "strictFunctionTypes": true,
    
    // bind, call, apply ต้อง match signature
    // "strictBindCallApply": true,
    
    // Class properties ต้องถูก initialize
    // "strictPropertyInitialization": true,
    
    // 'this' ต้องมี explicit type
    // "noImplicitThis": true,
    
    // ใส่ 'use strict' ใน output
    // "alwaysStrict": true,
    
    // ===== Additional Checks =====
    
    // Local variables ที่ไม่ได้ใช้ → Error
    "noUnusedLocals": true,
    
    // Parameters ที่ไม่ได้ใช้ → Error
    "noUnusedParameters": true,
    
    // ทุก code path ต้อง return
    "noImplicitReturns": true,
    
    // switch ต้องมี break/return ทุก case
    "noFallthroughCasesInSwitch": true,
    
    // Index signature ต้อง explicit
    "noUncheckedIndexedAccess": true,
    
    // Property access ต้องใช้ key ที่มีใน type
    "noPropertyAccessFromIndexSignature": false,
    
    // Optional properties ต้อง exact match
    "exactOptionalPropertyTypes": true,
  }
}
```

```typescript
// ===== ผลของ strict mode =====

// noImplicitAny
function process(value) { // ❌ Error: Parameter 'value' implicitly has an 'any' type
  return value;
}

function process2(value: string): string { // ✅
  return value;
}

// strictNullChecks
function getUser(id: number) {
  const users = [{ id: 1, name: 'Alice' }];
  return users.find(u => u.id === id);
  // Return type: { id: number; name: string; } | undefined
}

const user = getUser(1);
// user.name; ❌ Error: 'user' is possibly 'undefined'
user?.name; // ✅

// noUncheckedIndexedAccess
const arr = [1, 2, 3];
const first = arr[0]; // type: number | undefined (ไม่ใช่ number!)
if (first !== undefined) {
  console.log(first.toFixed(2)); // ✅
}

// strictPropertyInitialization
class User {
  name: string;        // ❌ Error: ต้อง initialize
  age: number = 0;     // ✅
  email!: string;      // ✅ บอกว่าจะ assign ทีหลัง
  
  constructor() {
    this.name = '';    // ✅ initialize ใน constructor
  }
}
```

---

## Step 1384: Type Predicates ขั้นสูง

```typescript
// ===== Advanced Type Predicates =====

// ===== Assertion Functions =====

function assert(condition: boolean, message?: string): asserts condition {
  if (!condition) {
    throw new Error(message || 'Assertion failed');
  }
}

function assertIsString(value: unknown, name?: string): asserts value is string {
  if (typeof value !== 'string') {
    throw new TypeError(
      `Expected ${name || 'value'} to be string, got ${typeof value}`
    );
  }
}

function assertIsDefined<T>(value: T | null | undefined): asserts value is T {
  if (value == null) {
    throw new Error('Expected defined value, got null or undefined');
  }
}

// การใช้งาน
function processConfig(rawConfig: unknown) {
  assert(typeof rawConfig === 'object' && rawConfig !== null, 'Config must be an object');
  // rawConfig: object ตอนนี้
  
  const config = rawConfig as Record<string, unknown>;
  
  assertIsString(config.apiUrl, 'config.apiUrl');
  // config.apiUrl: string ตอนนี้
  
  assertIsDefined(config.timeout);
  // config.timeout: {} ตอนนี้ (ไม่ใช่ null/undefined)
  
  return {
    apiUrl: config.apiUrl,
    timeout: config.timeout as number,
  };
}

// ===== Complex Type Guards =====

// Type guard ที่ตรวจสอบ structure
function isUser(value: unknown): value is User {
  if (typeof value !== 'object' || value === null) return false;
  
  const obj = value as Record<string, unknown>;
  
  return (
    typeof obj.id === 'number' &&
    typeof obj.name === 'string' &&
    typeof obj.email === 'string' &&
    (obj.role === 'admin' || obj.role === 'user')
  );
}

// Generic type guard factory
function hasProperties<T extends Record<string, unknown>>(
  value: unknown,
  properties: { [K in keyof T]: (val: unknown) => val is T[K] }
): value is T {
  if (typeof value !== 'object' || value === null) return false;
  
  return Object.entries(properties).every(([key, check]) => {
    return key in (value as object) && check((value as any)[key]);
  });
}

function isString(val: unknown): val is string {
  return typeof val === 'string';
}

function isNumber(val: unknown): val is number {
  return typeof val === 'number';
}

const isProduct = (val: unknown): val is Product =>
  hasProperties<Product>(val, {
    id: isNumber,
    name: isString,
    price: isNumber,
  });
```

---

## Step 1385-1390: Complete TypeScript Project

```typescript
// ===== Complete E-Commerce TypeScript Backend =====

// types/index.ts
export interface BaseEntity {
  id: number;
  createdAt: Date;
  updatedAt: Date;
}

export interface User extends BaseEntity {
  name: string;
  email: string;
  role: 'admin' | 'user';
  passwordHash: string;
}

export interface Product extends BaseEntity {
  name: string;
  description: string;
  price: number;
  stock: number;
  category: string;
  images: string[];
  active: boolean;
}

export interface CartItem {
  product: Product;
  quantity: number;
}

export interface Order extends BaseEntity {
  userId: number;
  items: OrderItem[];
  status: OrderStatus;
  total: number;
  shippingAddress: Address;
  paymentStatus: PaymentStatus;
}

export interface OrderItem {
  productId: number;
  name: string;
  price: number;
  quantity: number;
}

export interface Address {
  street: string;
  city: string;
  province: string;
  postalCode: string;
  country: string;
}

export type OrderStatus = 'pending' | 'confirmed' | 'shipped' | 'delivered' | 'cancelled';
export type PaymentStatus = 'unpaid' | 'paid' | 'refunded';

// DTOs
export type CreateUserDTO = Pick<User, 'name' | 'email'> & { password: string };
export type UpdateUserDTO = Partial<Pick<User, 'name' | 'email'>>;
export type CreateProductDTO = Omit<Product, keyof BaseEntity | 'active'>;
export type UpdateProductDTO = Partial<CreateProductDTO>;

// API Response types
export interface ApiResponse<T> {
  status: 'success' | 'error';
  data?: T;
  message?: string;
  errors?: Record<string, string[]>;
}

export interface PaginatedResponse<T> extends ApiResponse<T[]> {
  pagination: {
    page: number;
    limit: number;
    total: number;
    totalPages: number;
  };
}

// ===== Generic Repository Pattern =====

// repositories/base.repository.ts
export interface BaseRepository<T extends BaseEntity> {
  findById(id: number): Promise<T | null>;
  findAll(options?: QueryOptions): Promise<T[]>;
  create(data: Omit<T, keyof BaseEntity>): Promise<T>;
  update(id: number, data: Partial<T>): Promise<T | null>;
  delete(id: number): Promise<boolean>;
  count(filter?: Partial<T>): Promise<number>;
}

export interface QueryOptions {
  page?: number;
  limit?: number;
  orderBy?: string;
  order?: 'asc' | 'desc';
  filter?: Record<string, unknown>;
}

abstract class Repository<T extends BaseEntity> implements BaseRepository<T> {
  protected items: T[] = [];
  protected nextId = 1;
  
  async findById(id: number): Promise<T | null> {
    return this.items.find(item => item.id === id) ?? null;
  }
  
  async findAll(options: QueryOptions = {}): Promise<T[]> {
    const { page = 1, limit = 20, orderBy, order = 'asc' } = options;
    
    let result = [...this.items];
    
    if (orderBy) {
      result.sort((a, b) => {
        const aVal = (a as any)[orderBy];
        const bVal = (b as any)[orderBy];
        return order === 'asc'
          ? aVal > bVal ? 1 : -1
          : aVal < bVal ? 1 : -1;
      });
    }
    
    const start = (page - 1) * limit;
    return result.slice(start, start + limit);
  }
  
  async create(data: Omit<T, keyof BaseEntity>): Promise<T> {
    const entity = {
      ...data,
      id: this.nextId++,
      createdAt: new Date(),
      updatedAt: new Date(),
    } as T;
    
    this.items.push(entity);
    return entity;
  }
  
  async update(id: number, data: Partial<T>): Promise<T | null> {
    const index = this.items.findIndex(item => item.id === id);
    if (index === -1) return null;
    
    this.items[index] = {
      ...this.items[index],
      ...data,
      id,  // ไม่ให้ override id
      updatedAt: new Date(),
    };
    
    return this.items[index];
  }
  
  async delete(id: number): Promise<boolean> {
    const index = this.items.findIndex(item => item.id === id);
    if (index === -1) return false;
    
    this.items.splice(index, 1);
    return true;
  }
  
  async count(filter?: Partial<T>): Promise<number> {
    if (!filter) return this.items.length;
    
    return this.items.filter(item =>
      Object.entries(filter).every(([key, value]) =>
        (item as any)[key] === value
      )
    ).length;
  }
}

// ===== Specific Repositories =====

class UserRepository extends Repository<User> {
  async findByEmail(email: string): Promise<User | null> {
    return this.items.find(u => u.email === email) ?? null;
  }
  
  async findByRole(role: User['role']): Promise<User[]> {
    return this.items.filter(u => u.role === role);
  }
}

class ProductRepository extends Repository<Product> {
  async findByCategory(category: string): Promise<Product[]> {
    return this.items.filter(p => p.category === category && p.active);
  }
  
  async search(query: string): Promise<Product[]> {
    const lower = query.toLowerCase();
    return this.items.filter(p =>
      p.active && (
        p.name.toLowerCase().includes(lower) ||
        p.description.toLowerCase().includes(lower)
      )
    );
  }
  
  async updateStock(id: number, quantity: number): Promise<boolean> {
    const product = await this.findById(id);
    if (!product) return false;
    
    await this.update(id, { stock: product.stock + quantity });
    return true;
  }
}

// ===== Service Layer =====

class ProductService {
  constructor(private repo: ProductRepository) {}
  
  async getProducts(options?: QueryOptions & { category?: string }): Promise<PaginatedResponse<Product>> {
    const products = options?.category
      ? await this.repo.findByCategory(options.category)
      : await this.repo.findAll(options);
    
    const total = await this.repo.count(options?.category ? { category: options.category } : undefined);
    const page = options?.page ?? 1;
    const limit = options?.limit ?? 20;
    
    return {
      status: 'success',
      data: products,
      pagination: {
        page,
        limit,
        total,
        totalPages: Math.ceil(total / limit),
      },
    };
  }
  
  async getProduct(id: number): Promise<ApiResponse<Product>> {
    const product = await this.repo.findById(id);
    
    if (!product) {
      return { status: 'error', message: 'Product not found' };
    }
    
    return { status: 'success', data: product };
  }
  
  async createProduct(data: CreateProductDTO): Promise<ApiResponse<Product>> {
    const product = await this.repo.create({
      ...data,
      active: true,
    } as Omit<Product, keyof BaseEntity>);
    
    return { status: 'success', data: product };
  }
  
  async updateProduct(id: number, data: UpdateProductDTO): Promise<ApiResponse<Product>> {
    const product = await this.repo.update(id, data);
    
    if (!product) {
      return { status: 'error', message: 'Product not found' };
    }
    
    return { status: 'success', data: product };
  }
  
  async deleteProduct(id: number): Promise<ApiResponse<null>> {
    const deleted = await this.repo.delete(id);
    
    if (!deleted) {
      return { status: 'error', message: 'Product not found' };
    }
    
    return { status: 'success', message: 'Product deleted' };
  }
}

// ===== Type-safe Event System =====

type AppEvents = {
  'user:created': { user: User };
  'user:updated': { user: User; changes: Partial<User> };
  'order:placed': { order: Order };
  'order:shipped': { orderId: number; trackingNumber: string };
  'product:out-of-stock': { product: Product };
};

class TypedEventEmitter<TEvents extends Record<string, unknown>> {
  private handlers = new Map<string, Set<Function>>();
  
  on<K extends keyof TEvents>(event: K, handler: (data: TEvents[K]) => void): () => void {
    const key = event as string;
    if (!this.handlers.has(key)) {
      this.handlers.set(key, new Set());
    }
    this.handlers.get(key)!.add(handler);
    
    return () => this.off(event, handler);
  }
  
  off<K extends keyof TEvents>(event: K, handler: (data: TEvents[K]) => void): void {
    this.handlers.get(event as string)?.delete(handler);
  }
  
  emit<K extends keyof TEvents>(event: K, data: TEvents[K]): void {
    this.handlers.get(event as string)?.forEach(h => h(data));
  }
}

const eventBus = new TypedEventEmitter<AppEvents>();

// ✅ Type-safe event handling
eventBus.on('order:placed', ({ order }) => {
  console.log(`Order ${order.id} placed by user ${order.userId}`);
  // sendOrderConfirmationEmail(order);
});

eventBus.on('product:out-of-stock', ({ product }) => {
  console.warn(`Product ${product.name} is out of stock`);
  // notifySupplier(product);
});

// ✅ Type-safe event emitting
eventBus.emit('order:placed', { order: newOrder });
// eventBus.emit('order:placed', { orderId: 1 }); ❌ Error: ต้องมี order property
```

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: Mapped Types
สร้าง utility types:
- `Mutable<T>` - ลบ readonly จากทุก property
- `DeepMutable<T>` - ลบ readonly recursively
- `NullableProps<T, K>` - ทำให้บาง properties เป็น nullable

### แบบฝึกหัดที่ 2: Conditional Types
สร้าง:
- `IsArray<T>` - คืน true ถ้า T เป็น array
- `UnpackArray<T>` - unwrap array type
- `PromisifyAll<T>` - wrap ทุก function return type ด้วย Promise

### แบบฝึกหัดที่ 3: Template Literal Types
สร้าง router type system:
- `HttpRoute<Method, Path>` เช่น `'GET /users'`
- `ExtractMethod<T>` ดึง method จาก route string
- `ExtractPath<T>` ดึง path จาก route string

### แบบฝึกหัดที่ 4: Generic Repository
ขยาย `Repository<T>` ให้มี:
- `bulkCreate(data[])` - สร้างหลายรายการ
- `findOrCreate(data)` - หาหรือสร้าง
- `upsert(id, data)` - update ถ้ามี, create ถ้าไม่มี

### แบบฝึกหัดที่ 5: Type-Safe Builder Pattern
สร้าง `QueryBuilder<T>` ที่:
- `where(field: keyof T, value: T[K])` - filter
- `orderBy(field: keyof T, direction)` - sort
- `limit(n: number)` - limit results
- `build()` - คืน type-safe query object

---

## สรุป

| Feature | คำอธิบาย | ตัวอย่าง |
|---------|----------|----------|
| Mapped Types | สร้าง type จาก type อื่น | `{ [K in keyof T]: T[K] }` |
| Conditional Types | Type ที่ขึ้นกับ condition | `T extends U ? A : B` |
| Template Literal Types | Type จาก string patterns | `` `on${Capitalize<T>}` `` |
| Utility Types | Built-in type helpers | `Partial<T>`, `Pick<T,K>`, `Omit<T,K>` |
| infer | ดึง type จาก conditional | `T extends infer R ? R : never` |
| Variadic Tuples | Spread ใน tuple types | `[...T, ...U]` |
| Recursive Types | Types ที่อ้างถึงตัวเอง | `type Tree<T> = { value: T; children: Tree<T>[] }` |
| Declaration Files | Type definitions (.d.ts) | `declare module 'lib'` |
| Module Augmentation | เพิ่ม types ให้ existing modules | `declare module 'express'` |
| Decorators | Metadata annotations | `@Injectable`, `@Get('/')` |

```bash
# คำสั่งที่ใช้บ่อย
tsc --init                    # สร้าง tsconfig.json
tsc                           # compile TypeScript
tsc --watch                   # watch mode
tsc --noEmit                  # type-check เท่านั้น
ts-node src/index.ts          # รันโดยตรง
tsx src/index.ts              # รันด้วย tsx (เร็วกว่า)
npx tsc --strict              # compile ด้วย strict mode
```
