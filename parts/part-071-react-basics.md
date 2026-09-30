# Part 71: React.js พื้นฐาน (Steps 1391-1410)

## บทนำ

React.js คือ JavaScript library สำหรับสร้าง User Interface (UI) ที่พัฒนาโดย Facebook (Meta) ในปี 2013 React ได้เปลี่ยนวิธีการพัฒนา web application ไปอย่างสิ้นเชิง ด้วยแนวคิด component-based architecture และ Virtual DOM ที่ทำให้การพัฒนา UI ซับซ้อนเป็นเรื่องง่ายขึ้น

ในส่วนนี้เราจะเรียนรู้พื้นฐาน React ตั้งแต่การติดตั้งจนถึงการสร้าง component ที่มีประสิทธิภาพ

---

## Step 1391: React คืออะไร และทำไมถึงใช้ React

### React คืออะไร?

React เป็น **JavaScript library** (ไม่ใช่ framework) สำหรับสร้าง UI โดยมีหลักการสำคัญคือ:

1. **Component-Based**: แบ่ง UI เป็น component ย่อยๆ ที่นำมาประกอบกัน
2. **Declarative**: บอกว่าต้องการอะไร ไม่ใช่บอกวิธีทำ
3. **Learn Once, Write Anywhere**: ใช้ได้ทั้ง web, mobile (React Native), desktop

### ทำไมถึงใช้ React?

```jsx
// แบบ Vanilla JavaScript (Imperative - บอกวิธีทำ)
const div = document.createElement('div');
div.className = 'container';
const h1 = document.createElement('h1');
h1.textContent = 'Hello World';
div.appendChild(h1);
document.body.appendChild(div);

// แบบ React (Declarative - บอกว่าต้องการอะไร)
function App() {
  return (
    <div className="container">
      <h1>Hello World</h1>
    </div>
  );
}
```

### ความได้เปรียบของ React

```jsx
// 1. Component Reusability - ใช้ซ้ำได้
function Button({ text, onClick, variant = 'primary' }) {
  return (
    <button className={`btn btn-${variant}`} onClick={onClick}>
      {text}
    </button>
  );
}

// ใช้ซ้ำได้หลายที่
function App() {
  return (
    <div>
      <Button text="บันทึก" onClick={() => console.log('saved')} />
      <Button text="ยกเลิก" onClick={() => console.log('cancelled')} variant="secondary" />
      <Button text="ลบ" onClick={() => console.log('deleted')} variant="danger" />
    </div>
  );
}

// 2. State Management - จัดการ state ได้ง่าย
function Counter() {
  const [count, setCount] = React.useState(0);
  
  return (
    <div>
      <p>นับ: {count}</p>
      <button onClick={() => setCount(count + 1)}>เพิ่ม</button>
      <button onClick={() => setCount(count - 1)}>ลด</button>
    </div>
  );
}
```

---

## Step 1392: Virtual DOM คืออะไร

### DOM จริง vs Virtual DOM

```
DOM จริง (Real DOM):
┌─────────────────────────────────────┐
│  document                           │
│  └── html                          │
│      ├── head                      │
│      └── body                      │
│          ├── div#root              │
│          │   ├── h1               │
│          │   └── p                │
│          └── footer               │
└─────────────────────────────────────┘

Virtual DOM (React's copy in memory):
┌─────────────────────────────────────┐
│  React Virtual DOM Object          │
│  {                                  │
│    type: 'div',                     │
│    props: { id: 'root' },          │
│    children: [                      │
│      { type: 'h1', ... },          │
│      { type: 'p', ... }            │
│    ]                                │
│  }                                  │
└─────────────────────────────────────┘
```

### วิธีที่ Virtual DOM ทำงาน

```javascript
// กระบวนการ Reconciliation ของ React

// 1. มี state เปลี่ยน
// 2. React สร้าง Virtual DOM ใหม่
// 3. React เปรียบเทียบ (diffing) Virtual DOM เก่ากับใหม่
// 4. React อัพเดทเฉพาะส่วนที่เปลี่ยนใน Real DOM

// ตัวอย่าง: ก่อนและหลัง state เปลี่ยน

// ก่อน:
const virtualDOM_before = {
  type: 'div',
  props: {},
  children: [
    { type: 'h1', props: {}, children: ['สวัสดี'] },
    { type: 'p', props: {}, children: ['นับ: 0'] }  // เปลี่ยน
  ]
};

// หลัง:
const virtualDOM_after = {
  type: 'div',
  props: {},
  children: [
    { type: 'h1', props: {}, children: ['สวัสดี'] },  // ไม่เปลี่ยน
    { type: 'p', props: {}, children: ['นับ: 1'] }    // เปลี่ยน
  ]
};

// React จะอัพเดทเฉพาะ <p> ไม่อัพเดท <h1>
```

### ประโยชน์ของ Virtual DOM

```jsx
// ถ้าไม่มี Virtual DOM (ทำแบบ manual)
function updateUI(count) {
  // ต้องค้นหาและอัพเดท DOM ทุกครั้ง
  document.getElementById('counter').textContent = count;
  document.getElementById('status').textContent = count > 0 ? 'บวก' : 'ลบหรือศูนย์';
  document.getElementById('double').textContent = count * 2;
  // ... อาจพลาดหรือทำผิดลำดับได้
}

// แบบ React (อัพเดทอัตโนมัติ)
function Counter() {
  const [count, setCount] = React.useState(0);
  
  // React จัดการ DOM อัพเดทให้เองโดยอัตโนมัติ
  return (
    <div>
      <span id="counter">{count}</span>
      <span id="status">{count > 0 ? 'บวก' : 'ลบหรือศูนย์'}</span>
      <span id="double">{count * 2}</span>
    </div>
  );
}
```

---

## Step 1393: Create React App vs Vite Setup

### วิธีที่ 1: Create React App (CRA)

```bash
# ติดตั้ง Create React App
npx create-react-app my-app
cd my-app
npm start

# โครงสร้างโปรเจค CRA
my-app/
├── node_modules/
├── public/
│   ├── index.html
│   ├── favicon.ico
│   └── manifest.json
├── src/
│   ├── App.css
│   ├── App.js
│   ├── App.test.js
│   ├── index.css
│   ├── index.js
│   └── logo.svg
├── .gitignore
├── package.json
└── README.md
```

### วิธีที่ 2: Vite (แนะนำ - เร็วกว่ามาก)

```bash
# ติดตั้งด้วย Vite
npm create vite@latest my-react-app -- --template react
cd my-react-app
npm install
npm run dev

# หรือสำหรับ TypeScript
npm create vite@latest my-react-app -- --template react-ts

# โครงสร้างโปรเจค Vite
my-react-app/
├── node_modules/
├── public/
│   └── vite.svg
├── src/
│   ├── assets/
│   │   └── react.svg
│   ├── App.css
│   ├── App.jsx
│   ├── index.css
│   └── main.jsx
├── .gitignore
├── index.html
├── package.json
└── vite.config.js
```

### ไฟล์หลักของ React App

```jsx
// src/main.jsx (entry point)
import React from 'react';
import ReactDOM from 'react-dom/client';
import App from './App';
import './index.css';

// สร้าง root element และ render App
const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```

```jsx
// src/App.jsx
import './App.css';

function App() {
  return (
    <div className="App">
      <header className="App-header">
        <h1>สวัสดี React!</h1>
        <p>ยินดีต้อนรับสู่ React</p>
      </header>
    </div>
  );
}

export default App;
```

```html
<!-- index.html -->
<!DOCTYPE html>
<html lang="th">
  <head>
    <meta charset="UTF-8" />
    <link rel="icon" type="image/svg+xml" href="/vite.svg" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>My React App</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.jsx"></script>
  </body>
</html>
```

### vite.config.js

```javascript
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';

export default defineConfig({
  plugins: [react()],
  server: {
    port: 3000,
    open: true
  },
  build: {
    outDir: 'dist'
  }
});
```

---

## Step 1394: JSX Syntax

JSX (JavaScript XML) คือ syntax extension ของ JavaScript ที่ทำให้เขียน HTML-like code ใน JavaScript ได้

### กฎพื้นฐานของ JSX

```jsx
// 1. ต้องมี parent element เดียว
// ผิด:
function Wrong() {
  return (
    <h1>หัวข้อ</h1>
    <p>เนื้อหา</p>  // Error!
  );
}

// ถูก: ใช้ div ครอบ
function Right() {
  return (
    <div>
      <h1>หัวข้อ</h1>
      <p>เนื้อหา</p>
    </div>
  );
}

// ถูก: ใช้ Fragment
function AlsoRight() {
  return (
    <>
      <h1>หัวข้อ</h1>
      <p>เนื้อหา</p>
    </>
  );
}
```

```jsx
// 2. JSX expressions ใช้ {} 
function JSXExpressions() {
  const name = 'สมชาย';
  const age = 25;
  const isAdmin = true;
  const items = ['แอปเปิล', 'กล้วย', 'ส้ม'];
  
  return (
    <div>
      <p>ชื่อ: {name}</p>
      <p>อายุ: {age} ปี</p>
      <p>เป็นแอดมิน: {isAdmin ? 'ใช่' : 'ไม่ใช่'}</p>
      <p>2 + 2 = {2 + 2}</p>
      <p>วันที่: {new Date().toLocaleDateString('th-TH')}</p>
      <ul>
        {items.map(item => <li key={item}>{item}</li>)}
      </ul>
    </div>
  );
}
```

```jsx
// 3. className แทน class, htmlFor แทน for
function FormExample() {
  return (
    <div>
      {/* ใช้ className แทน class */}
      <div className="container">
        {/* ใช้ htmlFor แทน for */}
        <label htmlFor="email">อีเมล:</label>
        <input 
          type="email" 
          id="email"
          className="form-input"
        />
      </div>
    </div>
  );
}
```

```jsx
// 4. Attributes ใช้ camelCase
function StyleExample() {
  return (
    <div>
      {/* onclick → onClick */}
      <button onClick={() => alert('คลิก!')}>คลิกฉัน</button>
      
      {/* style ใช้ object แทน string */}
      <p style={{ color: 'red', fontSize: '16px', backgroundColor: '#fff' }}>
        ข้อความสีแดง
      </p>
      
      {/* tabindex → tabIndex */}
      <input tabIndex={1} />
    </div>
  );
}
```

```jsx
// 5. Self-closing tags
function SelfClosing() {
  return (
    <div>
      <img src="photo.jpg" alt="รูปภาพ" />
      <br />
      <hr />
      <input type="text" />
      {/* Custom component ก็ต้องปิด */}
      <MyComponent />
    </div>
  );
}
```

```jsx
// 6. Comments ใน JSX
function WithComments() {
  return (
    <div>
      {/* นี่คือ comment ใน JSX */}
      <p>เนื้อหา</p>
      {/* 
        Comment
        หลายบรรทัด
      */}
    </div>
  );
}
```

### JSX คือ JavaScript

```jsx
// JSX จะถูกแปลงเป็น JavaScript โดย Babel
// JSX:
const element = <h1 className="title">สวัสดี</h1>;

// หลังแปลง:
const element = React.createElement(
  'h1',
  { className: 'title' },
  'สวัสดี'
);

// ซับซ้อนขึ้น:
const element2 = (
  <div className="container">
    <h1>หัวข้อ</h1>
    <p>เนื้อหา</p>
  </div>
);

// หลังแปลง:
const element2 = React.createElement(
  'div',
  { className: 'container' },
  React.createElement('h1', null, 'หัวข้อ'),
  React.createElement('p', null, 'เนื้อหา')
);
```

---

## Step 1395: Rendering Elements

```jsx
// การ render element ใน React
import React from 'react';
import ReactDOM from 'react-dom/client';

// สร้าง element
const element = <h1>สวัสดี React!</h1>;

// Render เข้าไปใน DOM
const root = ReactDOM.createRoot(document.getElementById('root'));
root.render(element);
```

```jsx
// Rendering dynamic content
function Clock() {
  const [time, setTime] = React.useState(new Date());
  
  React.useEffect(() => {
    const timer = setInterval(() => {
      setTime(new Date());
    }, 1000);
    return () => clearInterval(timer);
  }, []);
  
  return (
    <div>
      <h1>เวลาปัจจุบัน</h1>
      <p>{time.toLocaleTimeString('th-TH')}</p>
    </div>
  );
}
```

```jsx
// Elements เป็น immutable
// React จะ re-render เมื่อ state หรือ props เปลี่ยน
function Temperature() {
  const [celsius, setCelsius] = React.useState(0);
  
  const fahrenheit = (celsius * 9/5) + 32;
  const kelvin = celsius + 273.15;
  
  return (
    <div>
      <input
        type="number"
        value={celsius}
        onChange={e => setCelsius(Number(e.target.value))}
      />
      <p>°C: {celsius}</p>
      <p>°F: {fahrenheit.toFixed(2)}</p>
      <p>K: {kelvin.toFixed(2)}</p>
    </div>
  );
}
```

---

## Step 1396: Components - Functional Components

### Functional Components คืออะไร

```jsx
// Functional Component พื้นฐาน
function Greeting() {
  return <h1>สวัสดี!</h1>;
}

// Arrow function version
const Greeting2 = () => <h1>สวัสดี!</h1>;

// ใช้ใน App
function App() {
  return (
    <div>
      <Greeting />
      <Greeting2 />
    </div>
  );
}
```

```jsx
// Component ที่มี logic
function UserCard() {
  const user = {
    name: 'สมชาย ใจดี',
    role: 'นักพัฒนา',
    email: 'somchai@example.com',
    avatar: 'https://via.placeholder.com/100'
  };
  
  const joinDate = new Date('2020-01-15');
  const yearsWorking = new Date().getFullYear() - joinDate.getFullYear();
  
  return (
    <div className="user-card">
      <img src={user.avatar} alt={user.name} />
      <div className="user-info">
        <h2>{user.name}</h2>
        <p className="role">{user.role}</p>
        <p className="email">{user.email}</p>
        <p className="years">ทำงานมา {yearsWorking} ปี</p>
      </div>
    </div>
  );
}
```

```jsx
// Composing Components
function Header() {
  return (
    <header>
      <nav>
        <a href="/">หน้าแรก</a>
        <a href="/about">เกี่ยวกับ</a>
        <a href="/contact">ติดต่อ</a>
      </nav>
    </header>
  );
}

function Hero() {
  return (
    <section className="hero">
      <h1>ยินดีต้อนรับ</h1>
      <p>เว็บไซต์ตัวอย่าง</p>
      <button>เริ่มต้น</button>
    </section>
  );
}

function Footer() {
  return (
    <footer>
      <p>© 2024 My Website</p>
    </footer>
  );
}

function App() {
  return (
    <div>
      <Header />
      <Hero />
      <Footer />
    </div>
  );
}
```

### กฎการตั้งชื่อ Component

```jsx
// Component ต้องขึ้นต้นด้วยตัวพิมพ์ใหญ่
function MyComponent() { return <div>Component</div>; }  // ✓
function myComponent() { return <div>ผิด</div>; }        // ✗

// ถ้าขึ้นต้นด้วยตัวเล็ก React จะคิดว่าเป็น HTML tag
function App() {
  return (
    <div>
      <MyComponent />  {/* React รู้ว่าเป็น Component */}
      <myComponent />  {/* React คิดว่าเป็น HTML tag ที่ไม่รู้จัก */}
    </div>
  );
}
```

---

## Step 1397: Props - การส่งข้อมูลให้ Components

```jsx
// Props พื้นฐาน
function Welcome(props) {
  return <h1>สวัสดี, {props.name}!</h1>;
}

// ใช้งาน
function App() {
  return (
    <div>
      <Welcome name="สมชาย" />
      <Welcome name="สมหญิง" />
      <Welcome name="ทุกคน" />
    </div>
  );
}
```

```jsx
// Destructuring Props (แนะนำ)
function ProductCard({ name, price, image, category, inStock }) {
  return (
    <div className="product-card">
      <img src={image} alt={name} />
      <div className="product-info">
        <span className="category">{category}</span>
        <h3>{name}</h3>
        <p className="price">฿{price.toLocaleString()}</p>
        <span className={`stock ${inStock ? 'in-stock' : 'out-of-stock'}`}>
          {inStock ? 'มีสินค้า' : 'สินค้าหมด'}
        </span>
      </div>
    </div>
  );
}

// ใช้งาน
function App() {
  return (
    <div className="products">
      <ProductCard
        name="MacBook Pro"
        price={89900}
        image="/macbook.jpg"
        category="แล็ปท็อป"
        inStock={true}
      />
      <ProductCard
        name="iPhone 15"
        price={35900}
        image="/iphone.jpg"
        category="สมาร์ทโฟน"
        inStock={false}
      />
    </div>
  );
}
```

```jsx
// Props ทุกประเภท
function AllPropsDemo({
  text,          // string
  number,        // number
  isActive,      // boolean
  items,         // array
  user,          // object
  onClick,       // function
  children,      // children components
}) {
  return (
    <div>
      <p>{text}</p>
      <p>{number}</p>
      <p>{isActive ? 'เปิดใช้' : 'ปิดใช้'}</p>
      <ul>{items.map(i => <li key={i}>{i}</li>)}</ul>
      <p>{user.name} - {user.email}</p>
      <button onClick={onClick}>คลิก</button>
      <div>{children}</div>
    </div>
  );
}

function App() {
  return (
    <AllPropsDemo
      text="ข้อความตัวอย่าง"
      number={42}
      isActive={true}
      items={['หนึ่ง', 'สอง', 'สาม']}
      user={{ name: 'สมชาย', email: 'somchai@example.com' }}
      onClick={() => alert('คลิก!')}
    >
      <p>นี่คือ children</p>
    </AllPropsDemo>
  );
}
```

```jsx
// Spreading Props
function Button({ className, children, ...rest }) {
  return (
    <button 
      className={`btn ${className}`} 
      {...rest}
    >
      {children}
    </button>
  );
}

function App() {
  return (
    <Button
      className="primary"
      onClick={() => console.log('คลิก')}
      disabled={false}
      type="submit"
    >
      ส่งข้อมูล
    </Button>
  );
}
```

---

## Step 1398: PropTypes และ defaultProps

```jsx
import PropTypes from 'prop-types';

// ติดตั้ง: npm install prop-types

function UserProfile({ name, age, email, role, avatar }) {
  return (
    <div className="user-profile">
      <img src={avatar} alt={name} />
      <h2>{name}</h2>
      <p>อายุ: {age}</p>
      <p>อีเมล: {email}</p>
      <p>บทบาท: {role}</p>
    </div>
  );
}

// กำหนด PropTypes
UserProfile.propTypes = {
  name: PropTypes.string.isRequired,
  age: PropTypes.number.isRequired,
  email: PropTypes.string.isRequired,
  role: PropTypes.oneOf(['admin', 'user', 'moderator']),
  avatar: PropTypes.string
};

// กำหนดค่า default
UserProfile.defaultProps = {
  role: 'user',
  avatar: 'https://via.placeholder.com/100'
};
```

```jsx
// PropTypes ประเภทต่างๆ
import PropTypes from 'prop-types';

function ComplexComponent({
  name,
  age,
  active,
  tags,
  address,
  render,
  children,
  style,
  onClick,
  status,
  data
}) {
  return <div>{name}</div>;
}

ComplexComponent.propTypes = {
  // Primitive types
  name: PropTypes.string,
  age: PropTypes.number,
  active: PropTypes.bool,
  
  // Array
  tags: PropTypes.arrayOf(PropTypes.string),
  
  // Object
  address: PropTypes.shape({
    street: PropTypes.string,
    city: PropTypes.string,
    zip: PropTypes.string
  }),
  
  // Function
  render: PropTypes.func,
  
  // React elements
  children: PropTypes.node,  // anything renderable
  style: PropTypes.object,
  
  // Event handler
  onClick: PropTypes.func,
  
  // One of specific values
  status: PropTypes.oneOf(['pending', 'active', 'inactive']),
  
  // One of types
  data: PropTypes.oneOfType([
    PropTypes.string,
    PropTypes.number,
    PropTypes.object
  ]),
  
  // Custom validator
  positiveNumber: function(props, propName, componentName) {
    if (props[propName] < 0) {
      return new Error(`${propName} ต้องเป็นจำนวนบวก`);
    }
  }
};
```

---

## Step 1399: State กับ useState Hook

```jsx
import React, { useState } from 'react';

// useState พื้นฐาน
function Counter() {
  // [state, setState] = useState(initialValue)
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>นับ: {count}</p>
      <button onClick={() => setCount(count + 1)}>เพิ่ม</button>
      <button onClick={() => setCount(count - 1)}>ลด</button>
      <button onClick={() => setCount(0)}>รีเซ็ต</button>
    </div>
  );
}
```

```jsx
// useState กับ Object
function UserForm() {
  const [user, setUser] = useState({
    firstName: '',
    lastName: '',
    email: '',
    age: 0
  });
  
  // อัพเดท state แบบ spread
  const handleChange = (field) => (e) => {
    setUser(prev => ({
      ...prev,
      [field]: e.target.value
    }));
  };
  
  return (
    <form>
      <input
        type="text"
        placeholder="ชื่อ"
        value={user.firstName}
        onChange={handleChange('firstName')}
      />
      <input
        type="text"
        placeholder="นามสกุล"
        value={user.lastName}
        onChange={handleChange('lastName')}
      />
      <input
        type="email"
        placeholder="อีเมล"
        value={user.email}
        onChange={handleChange('email')}
      />
      <input
        type="number"
        placeholder="อายุ"
        value={user.age}
        onChange={handleChange('age')}
      />
      <pre>{JSON.stringify(user, null, 2)}</pre>
    </form>
  );
}
```

```jsx
// useState กับ Array
function TodoList() {
  const [todos, setTodos] = useState([
    { id: 1, text: 'เรียน React', done: false },
    { id: 2, text: 'สร้างโปรเจค', done: false }
  ]);
  const [newTodo, setNewTodo] = useState('');
  
  const addTodo = () => {
    if (!newTodo.trim()) return;
    setTodos(prev => [
      ...prev,
      { id: Date.now(), text: newTodo, done: false }
    ]);
    setNewTodo('');
  };
  
  const toggleTodo = (id) => {
    setTodos(prev => prev.map(todo =>
      todo.id === id ? { ...todo, done: !todo.done } : todo
    ));
  };
  
  const deleteTodo = (id) => {
    setTodos(prev => prev.filter(todo => todo.id !== id));
  };
  
  return (
    <div>
      <div>
        <input
          value={newTodo}
          onChange={e => setNewTodo(e.target.value)}
          placeholder="เพิ่มรายการ..."
          onKeyPress={e => e.key === 'Enter' && addTodo()}
        />
        <button onClick={addTodo}>เพิ่ม</button>
      </div>
      <ul>
        {todos.map(todo => (
          <li key={todo.id}>
            <input
              type="checkbox"
              checked={todo.done}
              onChange={() => toggleTodo(todo.id)}
            />
            <span style={{ textDecoration: todo.done ? 'line-through' : 'none' }}>
              {todo.text}
            </span>
            <button onClick={() => deleteTodo(todo.id)}>ลบ</button>
          </li>
        ))}
      </ul>
      <p>เสร็จแล้ว {todos.filter(t => t.done).length}/{todos.length} รายการ</p>
    </div>
  );
}
```

```jsx
// Functional Update (ใช้เมื่ออ้างอิง state เดิม)
function Counter() {
  const [count, setCount] = useState(0);
  
  // ✗ อาจมีปัญหาเมื่อ setState ถูกเรียกหลายครั้ง
  const badIncrement = () => {
    setCount(count + 1);
    setCount(count + 1);  // ยังเป็น count เดิม!
  };
  
  // ✓ ใช้ functional update
  const goodIncrement = () => {
    setCount(prev => prev + 1);
    setCount(prev => prev + 1);  // ใช้ค่าล่าสุด
  };
  
  return (
    <div>
      <p>{count}</p>
      <button onClick={badIncrement}>เพิ่มแบบผิด</button>
      <button onClick={goodIncrement}>เพิ่มแบบถูก</button>
    </div>
  );
}
```

---

## Step 1400: Event Handling

```jsx
// Event Handlers พื้นฐาน
function EventDemo() {
  const handleClick = () => {
    console.log('คลิกแล้ว!');
  };
  
  const handleClickWithEvent = (e) => {
    console.log('คลิกที่:', e.target);
    console.log('ตำแหน่ง:', e.clientX, e.clientY);
  };
  
  const handleInput = (e) => {
    console.log('ค่า:', e.target.value);
  };
  
  const handleSubmit = (e) => {
    e.preventDefault();  // ป้องกัน form submit ปกติ
    console.log('ส่งฟอร์มแล้ว');
  };
  
  return (
    <div>
      <button onClick={handleClick}>คลิก</button>
      <button onClick={handleClickWithEvent}>คลิกพร้อม event</button>
      <input onChange={handleInput} placeholder="พิมพ์อะไรก็ได้" />
      <form onSubmit={handleSubmit}>
        <button type="submit">ส่ง</button>
      </form>
    </div>
  );
}
```

```jsx
// ส่ง Parameter ให้ Handler
function ButtonList() {
  const items = ['แดง', 'เขียว', 'น้ำเงิน'];
  
  const handleColorClick = (color) => {
    alert(`เลือกสี: ${color}`);
  };
  
  return (
    <div>
      {items.map(color => (
        <button 
          key={color} 
          // ✓ ใช้ arrow function
          onClick={() => handleColorClick(color)}
        >
          {color}
        </button>
      ))}
    </div>
  );
}
```

```jsx
// Event Types ต่างๆ
function AllEvents() {
  return (
    <div>
      {/* Mouse Events */}
      <button
        onClick={() => console.log('click')}
        onDoubleClick={() => console.log('double click')}
        onMouseEnter={() => console.log('mouse enter')}
        onMouseLeave={() => console.log('mouse leave')}
        onMouseDown={() => console.log('mouse down')}
        onMouseUp={() => console.log('mouse up')}
      >
        Mouse Events
      </button>
      
      {/* Keyboard Events */}
      <input
        onKeyDown={e => console.log('keydown:', e.key)}
        onKeyUp={e => console.log('keyup:', e.key)}
        onKeyPress={e => console.log('keypress:', e.key)}
      />
      
      {/* Focus Events */}
      <input
        onFocus={() => console.log('focused')}
        onBlur={() => console.log('blurred')}
        onChange={e => console.log('changed:', e.target.value)}
      />
      
      {/* Form Events */}
      <form onSubmit={e => { e.preventDefault(); console.log('submitted'); }}>
        <button type="submit">Submit</button>
      </form>
      
      {/* Drag Events */}
      <div
        draggable
        onDragStart={() => console.log('drag start')}
        onDragEnd={() => console.log('drag end')}
      >
        ลากได้
      </div>
    </div>
  );
}
```

---

## Step 1401: Conditional Rendering

```jsx
// วิธีที่ 1: if statement
function UserStatus({ isLoggedIn }) {
  if (isLoggedIn) {
    return <h1>ยินดีต้อนรับกลับ!</h1>;
  }
  return <h1>กรุณาเข้าสู่ระบบ</h1>;
}
```

```jsx
// วิธีที่ 2: Ternary Operator
function UserGreeting({ user }) {
  return (
    <div>
      <h1>{user ? `สวัสดี, ${user.name}!` : 'สวัสดี, ผู้เยี่ยมชม!'}</h1>
      
      {user ? (
        <button>ออกจากระบบ</button>
      ) : (
        <button>เข้าสู่ระบบ</button>
      )}
    </div>
  );
}
```

```jsx
// วิธีที่ 3: && Operator (แสดงเมื่อ condition เป็น true)
function Notifications({ count }) {
  return (
    <div>
      <h1>กล่องจดหมาย</h1>
      {/* แสดงเฉพาะเมื่อ count > 0 */}
      {count > 0 && (
        <p>คุณมี {count} ข้อความใหม่</p>
      )}
      {/* ระวัง! 0 จะถูก render */}
      {count && <p>จำนวน: {count}</p>}  {/* ถ้า count=0 จะแสดง 0 */}
      {count > 0 && <p>จำนวน: {count}</p>}  {/* แบบนี้ถูกต้อง */}
    </div>
  );
}
```

```jsx
// Conditional Rendering ซับซ้อน
function Dashboard({ user, isLoading, error }) {
  if (isLoading) {
    return <div className="loading">กำลังโหลด...</div>;
  }
  
  if (error) {
    return <div className="error">เกิดข้อผิดพลาด: {error.message}</div>;
  }
  
  if (!user) {
    return <div className="empty">ไม่พบข้อมูลผู้ใช้</div>;
  }
  
  return (
    <div className="dashboard">
      <h1>สวัสดี, {user.name}!</h1>
      
      {user.isAdmin && (
        <div className="admin-panel">
          <h2>แผงควบคุมผู้ดูแล</h2>
          <button>จัดการผู้ใช้</button>
        </div>
      )}
      
      {user.hasNotifications && (
        <div className="notifications">
          <span className="badge">{user.notificationCount}</span>
          <p>คุณมีการแจ้งเตือนใหม่</p>
        </div>
      )}
      
      <div className="stats">
        {user.role === 'admin' ? (
          <AdminStats data={user.adminData} />
        ) : (
          <UserStats data={user.userData} />
        )}
      </div>
    </div>
  );
}
```

---

## Step 1402: Lists และ Keys

```jsx
// Rendering Lists
function FruitList() {
  const fruits = ['แอปเปิล', 'กล้วย', 'ส้ม', 'มะม่วง', 'สตรอเบอร์รี'];
  
  return (
    <ul>
      {fruits.map((fruit, index) => (
        <li key={index}>{fruit}</li>
        // key ควรใช้ id ไม่ใช่ index เพราะอาจมีปัญหาเมื่อ list เปลี่ยน
      ))}
    </ul>
  );
}
```

```jsx
// ใช้ Unique ID เป็น Key
function UserList() {
  const users = [
    { id: 'u001', name: 'สมชาย', role: 'admin' },
    { id: 'u002', name: 'สมหญิง', role: 'user' },
    { id: 'u003', name: 'สมศักดิ์', role: 'moderator' }
  ];
  
  return (
    <table>
      <thead>
        <tr>
          <th>ID</th>
          <th>ชื่อ</th>
          <th>บทบาท</th>
        </tr>
      </thead>
      <tbody>
        {users.map(user => (
          <tr key={user.id}>
            <td>{user.id}</td>
            <td>{user.name}</td>
            <td>{user.role}</td>
          </tr>
        ))}
      </tbody>
    </table>
  );
}
```

```jsx
// Lists กับ Component
function ProductItem({ product, onAddToCart }) {
  return (
    <div className="product-item">
      <img src={product.image} alt={product.name} />
      <h3>{product.name}</h3>
      <p>฿{product.price.toLocaleString()}</p>
      <button onClick={() => onAddToCart(product)}>เพิ่มในตะกร้า</button>
    </div>
  );
}

function ProductList() {
  const [products] = React.useState([
    { id: 1, name: 'สินค้า A', price: 999, image: '/img1.jpg' },
    { id: 2, name: 'สินค้า B', price: 1499, image: '/img2.jpg' },
    { id: 3, name: 'สินค้า C', price: 2999, image: '/img3.jpg' }
  ]);
  
  const [cart, setCart] = React.useState([]);
  
  const handleAddToCart = (product) => {
    setCart(prev => {
      const existing = prev.find(item => item.id === product.id);
      if (existing) {
        return prev.map(item =>
          item.id === product.id
            ? { ...item, qty: item.qty + 1 }
            : item
        );
      }
      return [...prev, { ...product, qty: 1 }];
    });
  };
  
  return (
    <div>
      <div className="product-grid">
        {products.map(product => (
          <ProductItem
            key={product.id}
            product={product}
            onAddToCart={handleAddToCart}
          />
        ))}
      </div>
      <div className="cart">
        <h3>ตะกร้า ({cart.reduce((sum, item) => sum + item.qty, 0)} ชิ้น)</h3>
        {cart.map(item => (
          <div key={item.id}>
            {item.name} x {item.qty}
          </div>
        ))}
      </div>
    </div>
  );
}
```

---

## Step 1403: Lifting State Up

```jsx
// ปัญหา: State อยู่คนละ component ต้องการแชร์กัน
// แก้ด้วยการ "ยก" state ขึ้นไปอยู่ที่ parent

function TemperatureInput({ scale, temperature, onTemperatureChange }) {
  const scaleNames = { c: 'เซลเซียส', f: 'ฟาเรนไฮต์' };
  
  return (
    <fieldset>
      <legend>อุณหภูมิเป็น{scaleNames[scale]}</legend>
      <input
        value={temperature}
        onChange={e => onTemperatureChange(e.target.value)}
      />
    </fieldset>
  );
}

function BoilingPoint({ celsius }) {
  if (celsius >= 100) {
    return <p>น้ำกำลังเดือด!</p>;
  }
  return <p>น้ำยังไม่เดือด</p>;
}

// Parent component ถือ state
function Calculator() {
  const [temperature, setTemperature] = React.useState('');
  const [scale, setScale] = React.useState('c');
  
  const handleCelsiusChange = (temp) => {
    setScale('c');
    setTemperature(temp);
  };
  
  const handleFahrenheitChange = (temp) => {
    setScale('f');
    setTemperature(temp);
  };
  
  const celsius = scale === 'f' 
    ? ((parseFloat(temperature) - 32) * 5/9).toFixed(2) 
    : temperature;
  const fahrenheit = scale === 'c' 
    ? (parseFloat(temperature) * 9/5 + 32).toFixed(2) 
    : temperature;
  
  return (
    <div>
      <TemperatureInput
        scale="c"
        temperature={celsius}
        onTemperatureChange={handleCelsiusChange}
      />
      <TemperatureInput
        scale="f"
        temperature={fahrenheit}
        onTemperatureChange={handleFahrenheitChange}
      />
      <BoilingPoint celsius={parseFloat(celsius)} />
    </div>
  );
}
```

---

## Step 1404: Component Composition

```jsx
// Containment - ใช้ children
function Card({ title, children, className }) {
  return (
    <div className={`card ${className}`}>
      <div className="card-header">
        <h3>{title}</h3>
      </div>
      <div className="card-body">
        {children}
      </div>
    </div>
  );
}

// Named Slots Pattern
function PageLayout({ header, sidebar, main, footer }) {
  return (
    <div className="page">
      <header className="page-header">{header}</header>
      <div className="page-content">
        <aside className="page-sidebar">{sidebar}</aside>
        <main className="page-main">{main}</main>
      </div>
      <footer className="page-footer">{footer}</footer>
    </div>
  );
}

function App() {
  return (
    <PageLayout
      header={<nav>เมนู</nav>}
      sidebar={<ul><li>ลิงก์ 1</li><li>ลิงก์ 2</li></ul>}
      main={
        <div>
          <Card title="บัตรข้อมูล" className="blue">
            <p>เนื้อหาของบัตร</p>
            <button>ดูเพิ่มเติม</button>
          </Card>
        </div>
      }
      footer={<p>© 2024</p>}
    />
  );
}
```

```jsx
// Render Props Pattern
function MouseTracker({ render }) {
  const [position, setPosition] = React.useState({ x: 0, y: 0 });
  
  const handleMouseMove = (e) => {
    setPosition({ x: e.clientX, y: e.clientY });
  };
  
  return (
    <div onMouseMove={handleMouseMove} style={{ height: '300px', border: '1px solid' }}>
      {render(position)}
    </div>
  );
}

function App() {
  return (
    <MouseTracker
      render={({ x, y }) => (
        <p>ตำแหน่งเมาส์: {x}, {y}</p>
      )}
    />
  );
}
```

---

## Step 1405: Fragments

```jsx
import React, { Fragment } from 'react';

// ปัญหา: ต้องใช้ wrapper div ที่ไม่จำเป็น
function TableRow() {
  // ผิด: td ไม่สามารถอยู่ใน div ได้
  return (
    <div>
      <td>Cell 1</td>
      <td>Cell 2</td>
    </div>
  );
}

// แก้ด้วย Fragment
function TableRow() {
  return (
    <Fragment>
      <td>Cell 1</td>
      <td>Cell 2</td>
    </Fragment>
  );
}

// Short syntax <>
function TableRow() {
  return (
    <>
      <td>Cell 1</td>
      <td>Cell 2</td>
    </>
  );
}

// Fragment กับ key (ต้องใช้ <Fragment> ไม่ใช่ <>)
function OrderList({ items }) {
  return (
    <dl>
      {items.map(item => (
        <Fragment key={item.id}>
          <dt>{item.term}</dt>
          <dd>{item.description}</dd>
        </Fragment>
      ))}
    </dl>
  );
}
```

---

## Step 1406: Refs กับ useRef

```jsx
import React, { useRef } from 'react';

// useRef สำหรับ DOM access
function TextInput() {
  const inputRef = useRef(null);
  
  const focusInput = () => {
    inputRef.current.focus();
  };
  
  const clearInput = () => {
    inputRef.current.value = '';
    inputRef.current.focus();
  };
  
  return (
    <div>
      <input ref={inputRef} type="text" placeholder="พิมพ์อะไรก็ได้" />
      <button onClick={focusInput}>โฟกัส</button>
      <button onClick={clearInput}>ล้าง</button>
    </div>
  );
}
```

```jsx
// useRef สำหรับเก็บค่าที่ไม่ trigger re-render
function Timer() {
  const [time, setTime] = React.useState(0);
  const [isRunning, setIsRunning] = React.useState(false);
  const intervalRef = useRef(null);
  
  const start = () => {
    if (!isRunning) {
      setIsRunning(true);
      intervalRef.current = setInterval(() => {
        setTime(prev => prev + 1);
      }, 1000);
    }
  };
  
  const stop = () => {
    if (isRunning) {
      clearInterval(intervalRef.current);
      setIsRunning(false);
    }
  };
  
  const reset = () => {
    clearInterval(intervalRef.current);
    setIsRunning(false);
    setTime(0);
  };
  
  return (
    <div>
      <h2>เวลา: {time} วินาที</h2>
      <button onClick={start} disabled={isRunning}>เริ่ม</button>
      <button onClick={stop} disabled={!isRunning}>หยุด</button>
      <button onClick={reset}>รีเซ็ต</button>
    </div>
  );
}
```

```jsx
// useRef สำหรับ previous value
function CounterWithHistory() {
  const [count, setCount] = React.useState(0);
  const prevCountRef = useRef(0);
  
  React.useEffect(() => {
    prevCountRef.current = count;
  });
  
  const prevCount = prevCountRef.current;
  
  return (
    <div>
      <p>ปัจจุบัน: {count}</p>
      <p>ก่อนหน้า: {prevCount}</p>
      <button onClick={() => setCount(c => c + 1)}>เพิ่ม</button>
    </div>
  );
}
```

---

## Step 1407: Forms - Controlled Components

```jsx
// Controlled Component - React ควบคุม input value
function ControlledForm() {
  const [formData, setFormData] = React.useState({
    username: '',
    password: '',
    email: '',
    gender: '',
    country: '',
    newsletter: false,
    bio: ''
  });
  
  const handleChange = (e) => {
    const { name, value, type, checked } = e.target;
    setFormData(prev => ({
      ...prev,
      [name]: type === 'checkbox' ? checked : value
    }));
  };
  
  const handleSubmit = (e) => {
    e.preventDefault();
    console.log('ข้อมูลฟอร์ม:', formData);
    alert('ส่งข้อมูลสำเร็จ!');
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label>ชื่อผู้ใช้:</label>
        <input
          type="text"
          name="username"
          value={formData.username}
          onChange={handleChange}
          required
        />
      </div>
      
      <div>
        <label>รหัสผ่าน:</label>
        <input
          type="password"
          name="password"
          value={formData.password}
          onChange={handleChange}
          required
        />
      </div>
      
      <div>
        <label>อีเมล:</label>
        <input
          type="email"
          name="email"
          value={formData.email}
          onChange={handleChange}
        />
      </div>
      
      <div>
        <label>เพศ:</label>
        <select name="gender" value={formData.gender} onChange={handleChange}>
          <option value="">เลือกเพศ</option>
          <option value="male">ชาย</option>
          <option value="female">หญิง</option>
          <option value="other">อื่นๆ</option>
        </select>
      </div>
      
      <div>
        <label>ประเทศ:</label>
        <select name="country" value={formData.country} onChange={handleChange}>
          <option value="">เลือกประเทศ</option>
          <option value="th">ไทย</option>
          <option value="us">สหรัฐอเมริกา</option>
          <option value="jp">ญี่ปุ่น</option>
        </select>
      </div>
      
      <div>
        <label>
          <input
            type="checkbox"
            name="newsletter"
            checked={formData.newsletter}
            onChange={handleChange}
          />
          รับจดหมายข่าว
        </label>
      </div>
      
      <div>
        <label>ประวัติ:</label>
        <textarea
          name="bio"
          value={formData.bio}
          onChange={handleChange}
          rows={4}
          placeholder="เล่าเกี่ยวกับตัวเอง..."
        />
      </div>
      
      <button type="submit">ลงทะเบียน</button>
    </form>
  );
}
```

```jsx
// Form Validation
function RegistrationForm() {
  const [values, setValues] = React.useState({
    email: '',
    password: '',
    confirmPassword: ''
  });
  const [errors, setErrors] = React.useState({});
  const [touched, setTouched] = React.useState({});
  
  const validate = (values) => {
    const errors = {};
    
    if (!values.email) {
      errors.email = 'กรุณากรอกอีเมล';
    } else if (!/^[A-Z0-9._%+-]+@[A-Z0-9.-]+\.[A-Z]{2,}$/i.test(values.email)) {
      errors.email = 'รูปแบบอีเมลไม่ถูกต้อง';
    }
    
    if (!values.password) {
      errors.password = 'กรุณากรอกรหัสผ่าน';
    } else if (values.password.length < 8) {
      errors.password = 'รหัสผ่านต้องมีอย่างน้อย 8 ตัวอักษร';
    }
    
    if (!values.confirmPassword) {
      errors.confirmPassword = 'กรุณายืนยันรหัสผ่าน';
    } else if (values.confirmPassword !== values.password) {
      errors.confirmPassword = 'รหัสผ่านไม่ตรงกัน';
    }
    
    return errors;
  };
  
  const handleChange = (e) => {
    const { name, value } = e.target;
    setValues(prev => ({ ...prev, [name]: value }));
    
    if (touched[name]) {
      const newErrors = validate({ ...values, [name]: value });
      setErrors(prev => ({ ...prev, [name]: newErrors[name] }));
    }
  };
  
  const handleBlur = (e) => {
    const { name } = e.target;
    setTouched(prev => ({ ...prev, [name]: true }));
    const newErrors = validate(values);
    setErrors(prev => ({ ...prev, [name]: newErrors[name] }));
  };
  
  const handleSubmit = (e) => {
    e.preventDefault();
    const allTouched = { email: true, password: true, confirmPassword: true };
    setTouched(allTouched);
    const validationErrors = validate(values);
    setErrors(validationErrors);
    
    if (Object.keys(validationErrors).length === 0) {
      alert('ลงทะเบียนสำเร็จ!');
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <div>
        <label>อีเมล:</label>
        <input
          name="email"
          type="email"
          value={values.email}
          onChange={handleChange}
          onBlur={handleBlur}
          className={errors.email && touched.email ? 'error' : ''}
        />
        {touched.email && errors.email && (
          <span className="error-message">{errors.email}</span>
        )}
      </div>
      
      <div>
        <label>รหัสผ่าน:</label>
        <input
          name="password"
          type="password"
          value={values.password}
          onChange={handleChange}
          onBlur={handleBlur}
          className={errors.password && touched.password ? 'error' : ''}
        />
        {touched.password && errors.password && (
          <span className="error-message">{errors.password}</span>
        )}
      </div>
      
      <div>
        <label>ยืนยันรหัสผ่าน:</label>
        <input
          name="confirmPassword"
          type="password"
          value={values.confirmPassword}
          onChange={handleChange}
          onBlur={handleBlur}
        />
        {touched.confirmPassword && errors.confirmPassword && (
          <span className="error-message">{errors.confirmPassword}</span>
        )}
      </div>
      
      <button type="submit">ลงทะเบียน</button>
    </form>
  );
}
```

---

## Step 1408: CSS Modules

```jsx
// CSS Modules - ป้องกัน class name conflicts

// Button.module.css
/*
.button {
  padding: 10px 20px;
  border-radius: 4px;
  border: none;
  cursor: pointer;
  font-size: 16px;
}

.primary {
  background-color: #007bff;
  color: white;
}

.secondary {
  background-color: #6c757d;
  color: white;
}

.danger {
  background-color: #dc3545;
  color: white;
}

.small {
  padding: 5px 10px;
  font-size: 12px;
}

.large {
  padding: 15px 30px;
  font-size: 20px;
}
*/

// Button.jsx
import styles from './Button.module.css';
import clsx from 'clsx';  // npm install clsx

function Button({ variant = 'primary', size, children, className, ...props }) {
  return (
    <button
      className={clsx(
        styles.button,
        styles[variant],
        size && styles[size],
        className
      )}
      {...props}
    >
      {children}
    </button>
  );
}

// ใช้งาน
function App() {
  return (
    <div>
      <Button variant="primary">ปุ่มหลัก</Button>
      <Button variant="secondary" size="small">ปุ่มรอง (เล็ก)</Button>
      <Button variant="danger" size="large">ลบ (ใหญ่)</Button>
    </div>
  );
}
```

---

## Step 1409: Inline Styles

```jsx
// Inline Styles ใน React
function InlineStyleDemo() {
  const containerStyle = {
    display: 'flex',
    flexDirection: 'column',
    alignItems: 'center',
    padding: '20px',
    backgroundColor: '#f5f5f5'
  };
  
  const titleStyle = {
    fontSize: '24px',
    fontWeight: 'bold',
    color: '#333',
    marginBottom: '10px'
  };
  
  return (
    <div style={containerStyle}>
      <h1 style={titleStyle}>หัวข้อ</h1>
      <p style={{ color: '#666', lineHeight: 1.5 }}>เนื้อหา</p>
    </div>
  );
}
```

```jsx
// Dynamic Styles
function TrafficLight() {
  const [light, setLight] = React.useState('red');
  
  const colors = {
    red: '#ff0000',
    yellow: '#ffff00',
    green: '#00ff00'
  };
  
  const lightStyle = (color) => ({
    width: '80px',
    height: '80px',
    borderRadius: '50%',
    backgroundColor: light === color ? colors[color] : '#333',
    margin: '10px auto',
    transition: 'background-color 0.3s'
  });
  
  const cycle = () => {
    setLight(prev => {
      if (prev === 'red') return 'green';
      if (prev === 'green') return 'yellow';
      return 'red';
    });
  };
  
  return (
    <div style={{ textAlign: 'center' }}>
      <div style={{ background: '#222', padding: '20px', display: 'inline-block', borderRadius: '10px' }}>
        <div style={lightStyle('red')} />
        <div style={lightStyle('yellow')} />
        <div style={lightStyle('green')} />
      </div>
      <br />
      <button onClick={cycle}>เปลี่ยนไฟ</button>
    </div>
  );
}
```

---

## Step 1410: Component Lifecycle Overview

```jsx
import React, { useState, useEffect } from 'react';

// Lifecycle ของ Functional Component ผ่าน useEffect

function LifecycleDemo({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  
  // componentDidMount - ทำงานครั้งแรกเมื่อ mount
  useEffect(() => {
    console.log('Component mounted');
    
    // componentWillUnmount - cleanup function
    return () => {
      console.log('Component unmounted');
    };
  }, []); // empty array = run once on mount
  
  // componentDidUpdate - ทำงานเมื่อ dependency เปลี่ยน
  useEffect(() => {
    if (userId) {
      setLoading(true);
      fetch(`https://jsonplaceholder.typicode.com/users/${userId}`)
        .then(r => r.json())
        .then(data => {
          setUser(data);
          setLoading(false);
        });
    }
  }, [userId]); // ทำงานเมื่อ userId เปลี่ยน
  
  if (loading) return <p>กำลังโหลด...</p>;
  if (!user) return <p>ไม่พบข้อมูล</p>;
  
  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
}

// Demo Component
function App() {
  const [userId, setUserId] = useState(1);
  const [showDemo, setShowDemo] = useState(true);
  
  return (
    <div>
      <div>
        {[1, 2, 3].map(id => (
          <button key={id} onClick={() => setUserId(id)}>
            ผู้ใช้ {id}
          </button>
        ))}
        <button onClick={() => setShowDemo(!showDemo)}>
          {showDemo ? 'ซ่อน' : 'แสดง'}
        </button>
      </div>
      {showDemo && <LifecycleDemo userId={userId} />}
    </div>
  );
}
```

---

## สรุปแนวคิดสำคัญ

```jsx
// รวม Pattern ที่สำคัญ
function BestPracticesDemo() {
  // 1. Destructuring Props
  // 2. useState
  // 3. Conditional Rendering
  // 4. List rendering with key
  // 5. Event Handling
  // 6. Form Control
  
  const [items, setItems] = useState([
    { id: 1, name: 'รายการ 1', done: false },
    { id: 2, name: 'รายการ 2', done: true }
  ]);
  const [input, setInput] = useState('');
  const [filter, setFilter] = useState('all');
  
  const addItem = () => {
    if (input.trim()) {
      setItems(prev => [...prev, { id: Date.now(), name: input, done: false }]);
      setInput('');
    }
  };
  
  const toggle = (id) => {
    setItems(prev => prev.map(item =>
      item.id === id ? { ...item, done: !item.done } : item
    ));
  };
  
  const filteredItems = items.filter(item => {
    if (filter === 'active') return !item.done;
    if (filter === 'done') return item.done;
    return true;
  });
  
  return (
    <div className="todo-app">
      <h1>รายการงาน</h1>
      
      <div className="input-row">
        <input
          value={input}
          onChange={e => setInput(e.target.value)}
          onKeyPress={e => e.key === 'Enter' && addItem()}
          placeholder="เพิ่มรายการ..."
        />
        <button onClick={addItem}>เพิ่ม</button>
      </div>
      
      <div className="filters">
        {['all', 'active', 'done'].map(f => (
          <button
            key={f}
            onClick={() => setFilter(f)}
            className={filter === f ? 'active' : ''}
          >
            {f === 'all' ? 'ทั้งหมด' : f === 'active' ? 'ยังไม่เสร็จ' : 'เสร็จแล้ว'}
          </button>
        ))}
      </div>
      
      {filteredItems.length === 0 ? (
        <p className="empty">ไม่มีรายการ</p>
      ) : (
        <ul>
          {filteredItems.map(item => (
            <li key={item.id} className={item.done ? 'done' : ''}>
              <input
                type="checkbox"
                checked={item.done}
                onChange={() => toggle(item.id)}
              />
              {item.name}
            </li>
          ))}
        </ul>
      )}
      
      <p className="summary">
        เหลือ {items.filter(i => !i.done).length} รายการ
      </p>
    </div>
  );
}
```

---

## แบบฝึกหัด (Exercises)

### แบบฝึกหัดที่ 1: Profile Card Component
สร้าง component `ProfileCard` ที่รับ props: `name`, `title`, `bio`, `avatar`, `skills` (array), `social` (object)

### แบบฝึกหัดที่ 2: Shopping Cart
สร้าง shopping cart app ที่มี:
- แสดงสินค้า (รูป, ชื่อ, ราคา)
- เพิ่ม/ลด จำนวน
- ลบสินค้าออกจากตะกร้า
- แสดงยอดรวม

### แบบฝึกหัดที่ 3: Form Validation
สร้างฟอร์มสมัครสมาชิกที่ validate:
- ชื่อ (ต้องมีอย่างน้อย 2 ตัวอักษร)
- อีเมล (รูปแบบถูกต้อง)
- รหัสผ่าน (8+ ตัว, มีตัวเลขและตัวอักษร)
- ยืนยันรหัสผ่าน (ต้องตรงกัน)

### แบบฝึกหัดที่ 4: Quiz App
สร้าง quiz app ที่:
- แสดงคำถามทีละข้อ
- มีตัวเลือก 4 ข้อ
- บอกผลถูก/ผิดทันที
- แสดงคะแนนสุดท้าย

### แบบฝึกหัดที่ 5: Color Picker
สร้าง color picker ที่:
- มี sliders สำหรับ R, G, B
- แสดงสีที่เลือก
- แสดงค่า HEX
- copy HEX code ได้

---

## สรุปท้ายส่วน

ในส่วนนี้เราได้เรียนรู้:

- **React คืออะไร**: library สำหรับสร้าง UI แบบ component-based
- **Virtual DOM**: กลไกที่ React ใช้ update DOM อย่างมีประสิทธิภาพ
- **การตั้งค่า**: ใช้ Vite สำหรับโปรเจคใหม่ (เร็วกว่า CRA)
- **JSX**: syntax ที่ผสม HTML ใน JavaScript
- **Components**: ฟังก์ชันที่ return JSX
- **Props**: ส่งข้อมูลจาก parent ไป child
- **State**: ข้อมูลที่เปลี่ยนได้และทำให้ component re-render
- **Events**: handle user interactions
- **Conditional Rendering**: แสดง UI ตามเงื่อนไข
- **Lists**: render array ด้วย map()
- **Forms**: controlled components ด้วย useState

ส่วนถัดไป (Part 72) เราจะเรียน React Hooks เชิงลึก!
