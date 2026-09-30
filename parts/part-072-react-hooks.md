# Part 72: React Hooks (Steps 1411-1430)

## บทนำ

React Hooks คือฟังก์ชันพิเศษที่เพิ่มเข้ามาใน React 16.8 ที่ทำให้ functional components สามารถใช้ features ของ React ได้ เช่น state, lifecycle, context และอื่นๆ โดยไม่ต้องใช้ class components

Hooks แก้ปัญหาหลักๆ ของ React เดิม:
1. Logic ที่ใช้ซ้ำได้ยากใน class components
2. Component ที่ซับซ้อนเกินไปเมื่อใช้ lifecycle methods
3. Class syntax ที่สับสน (this binding)

---

## Step 1411: กฎของ Hooks (Rules of Hooks)

### กฎที่ต้องปฏิบัติตามเสมอ

```jsx
// ❌ ผิด: เรียก Hook ใน condition
function BadExample() {
  if (someCondition) {
    const [count, setCount] = useState(0);  // ผิด!
  }
  return <div />;
}

// ✅ ถูก: เรียก Hook ที่ top level เสมอ
function GoodExample() {
  const [count, setCount] = useState(0);  // ถูก
  
  if (someCondition) {
    // logic ใน condition
  }
  return <div>{count}</div>;
}
```

```jsx
// ❌ ผิด: เรียก Hook ใน loop
function BadLoop() {
  const items = [1, 2, 3];
  items.forEach(item => {
    const [value, setValue] = useState(item);  // ผิด!
  });
  return <div />;
}

// ✅ ถูก: เรียก Hook ที่ top level
function GoodLoop() {
  // ถ้าต้องการหลาย states ให้ประกาศแยก
  const [val1, setVal1] = useState(1);
  const [val2, setVal2] = useState(2);
  const [val3, setVal3] = useState(3);
  return <div />;
}
```

```jsx
// ❌ ผิด: เรียก Hook ใน regular function (ไม่ใช่ Component หรือ Custom Hook)
function notAComponent() {
  const [count] = useState(0);  // ผิด!
}

// ✅ ถูก: เรียกใน React component หรือ custom hook เท่านั้น
function ReactComponent() {
  const [count] = useState(0);  // ถูก
  return <div>{count}</div>;
}

function useCustomHook() {
  const [count] = useState(0);  // ถูก (custom hook ขึ้นต้นด้วย "use")
  return count;
}
```

### ทำไมถึงมีกฎนี้

React ติดตาม Hook calls ด้วยลำดับ (order) ดังนั้นถ้า Hook ถูกเรียกในลำดับที่ไม่คงที่ React จะเกิด error

---

## Step 1412: useState เชิงลึก

```jsx
import { useState } from 'react';

// 1. Lazy Initialization - สำหรับค่า initial ที่คำนวณหนัก
function HeavyInitialState() {
  // ❌ ไม่ดี: คำนวณทุก render
  const [items, setItems] = useState(computeExpensiveItems());
  
  // ✅ ดี: คำนวณครั้งเดียวเมื่อ mount
  const [items2, setItems2] = useState(() => computeExpensiveItems());
  
  return <div>{items.length} items</div>;
}

function computeExpensiveItems() {
  console.log('Computing...'); // ดูว่าเรียกกี่ครั้ง
  return Array.from({ length: 1000 }, (_, i) => ({ id: i, value: Math.random() }));
}
```

```jsx
// 2. State กับ Objects - ต้อง spread เสมอ
function ObjectState() {
  const [config, setConfig] = useState({
    theme: 'light',
    language: 'th',
    fontSize: 16,
    notifications: true
  });
  
  // ❌ ผิด: Object mutation
  const badUpdate = () => {
    config.theme = 'dark';
    setConfig(config);  // ไม่ trigger re-render เพราะ reference เดิม
  };
  
  // ✅ ถูก: สร้าง object ใหม่
  const goodUpdate = (key, value) => {
    setConfig(prev => ({ ...prev, [key]: value }));
  };
  
  return (
    <div>
      <p>Theme: {config.theme}</p>
      <button onClick={() => goodUpdate('theme', config.theme === 'light' ? 'dark' : 'light')}>
        Toggle Theme
      </button>
    </div>
  );
}
```

```jsx
// 3. State กับ Arrays
function ArrayState() {
  const [list, setList] = useState([1, 2, 3]);
  
  // เพิ่มรายการ
  const addItem = (item) => {
    setList(prev => [...prev, item]);
  };
  
  // ลบรายการ
  const removeItem = (index) => {
    setList(prev => prev.filter((_, i) => i !== index));
  };
  
  // อัพเดทรายการ
  const updateItem = (index, newValue) => {
    setList(prev => prev.map((item, i) => i === index ? newValue : item));
  };
  
  // เรียงลำดับ
  const sortAsc = () => {
    setList(prev => [...prev].sort((a, b) => a - b));
  };
  
  return (
    <div>
      <ul>
        {list.map((item, i) => (
          <li key={i}>
            {item}
            <button onClick={() => removeItem(i)}>ลบ</button>
          </li>
        ))}
      </ul>
      <button onClick={() => addItem(Math.floor(Math.random() * 100))}>เพิ่ม</button>
      <button onClick={sortAsc}>เรียงจากน้อยไปมาก</button>
    </div>
  );
}
```

```jsx
// 4. Multiple useState vs Single Object
function MultiStateExample() {
  // แบบแยก (ดีกว่าสำหรับค่าที่ไม่เกี่ยวกัน)
  const [username, setUsername] = useState('');
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState(null);
  
  // แบบรวม (ดีกว่าสำหรับค่าที่เกี่ยวกัน)
  const [formState, setFormState] = useState({
    username: '',
    email: '',
    password: ''
  });
  
  return <div />;
}
```

---

## Step 1413: useEffect - Side Effects

```jsx
import { useEffect, useState } from 'react';

// useEffect เบื้องต้น
function BasicEffect() {
  const [count, setCount] = useState(0);
  
  // ทำงานทุก render
  useEffect(() => {
    document.title = `คลิก ${count} ครั้ง`;
  });
  
  // ทำงานครั้งเดียว (mount)
  useEffect(() => {
    console.log('Component mounted');
  }, []);
  
  // ทำงานเมื่อ count เปลี่ยน
  useEffect(() => {
    console.log('count changed to:', count);
  }, [count]);
  
  return (
    <button onClick={() => setCount(c => c + 1)}>
      คลิก {count} ครั้ง
    </button>
  );
}
```

```jsx
// Dependency Array ต่างๆ
function EffectDependencies() {
  const [a, setA] = useState(0);
  const [b, setB] = useState(0);
  
  // ไม่มี dependency: ทำงานทุก render
  useEffect(() => {
    console.log('ทุก render');
  });
  
  // [] empty: ทำงานครั้งเดียว
  useEffect(() => {
    console.log('mount เท่านั้น');
  }, []);
  
  // [a]: ทำงานเมื่อ a เปลี่ยน
  useEffect(() => {
    console.log('a เปลี่ยนเป็น:', a);
  }, [a]);
  
  // [a, b]: ทำงานเมื่อ a หรือ b เปลี่ยน
  useEffect(() => {
    console.log('a หรือ b เปลี่ยน');
  }, [a, b]);
  
  return (
    <div>
      <button onClick={() => setA(a + 1)}>a: {a}</button>
      <button onClick={() => setB(b + 1)}>b: {b}</button>
    </div>
  );
}
```

---

## Step 1414: useEffect Cleanup

```jsx
// Cleanup ป้องกัน memory leaks
function CleanupExamples() {
  const [isOnline, setIsOnline] = useState(navigator.onLine);
  
  // 1. Event listener cleanup
  useEffect(() => {
    const handleOnline = () => setIsOnline(true);
    const handleOffline = () => setIsOnline(false);
    
    window.addEventListener('online', handleOnline);
    window.addEventListener('offline', handleOffline);
    
    // Cleanup: ลบ event listeners
    return () => {
      window.removeEventListener('online', handleOnline);
      window.removeEventListener('offline', handleOffline);
    };
  }, []);
  
  return <p>สถานะ: {isOnline ? 'ออนไลน์' : 'ออฟไลน์'}</p>;
}
```

```jsx
// Timer cleanup
function Timer() {
  const [seconds, setSeconds] = useState(0);
  const [isRunning, setIsRunning] = useState(false);
  
  useEffect(() => {
    if (!isRunning) return;
    
    const interval = setInterval(() => {
      setSeconds(s => s + 1);
    }, 1000);
    
    // Cleanup: ล้าง interval เมื่อ component unmount หรือ isRunning เปลี่ยน
    return () => clearInterval(interval);
  }, [isRunning]);
  
  return (
    <div>
      <p>{seconds} วินาที</p>
      <button onClick={() => setIsRunning(!isRunning)}>
        {isRunning ? 'หยุด' : 'เริ่ม'}
      </button>
      <button onClick={() => { setIsRunning(false); setSeconds(0); }}>
        รีเซ็ต
      </button>
    </div>
  );
}
```

```jsx
// Subscription cleanup (WebSocket, EventSource)
function LiveFeed() {
  const [messages, setMessages] = useState([]);
  
  useEffect(() => {
    // สมมติว่า subscribe ไปยัง WebSocket
    const subscription = {
      cancel: null
    };
    
    // ใน project จริงจะใช้:
    // const ws = new WebSocket('ws://...');
    // ws.onmessage = (e) => setMessages(prev => [...prev, e.data]);
    
    // Simulate subscription
    const id = setInterval(() => {
      setMessages(prev => [
        ...prev,
        `ข้อความ ${Date.now()}`
      ].slice(-5));  // เก็บแค่ 5 ข้อความล่าสุด
    }, 2000);
    
    subscription.cancel = () => clearInterval(id);
    
    return () => subscription.cancel();
  }, []);
  
  return (
    <div>
      <h3>Live Feed</h3>
      {messages.map((msg, i) => (
        <p key={i}>{msg}</p>
      ))}
    </div>
  );
}
```

---

## Step 1415: useEffect Patterns

```jsx
// Pattern 1: Fetch Data
function FetchData() {
  const [users, setUsers] = useState([]);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    let cancelled = false;
    
    const fetchUsers = async () => {
      try {
        setLoading(true);
        const response = await fetch('https://jsonplaceholder.typicode.com/users');
        if (!response.ok) throw new Error('ดึงข้อมูลไม่สำเร็จ');
        const data = await response.json();
        
        // ป้องกัน setState เมื่อ component unmount แล้ว
        if (!cancelled) {
          setUsers(data);
          setError(null);
        }
      } catch (err) {
        if (!cancelled) {
          setError(err.message);
        }
      } finally {
        if (!cancelled) {
          setLoading(false);
        }
      }
    };
    
    fetchUsers();
    
    return () => { cancelled = true; };
  }, []);
  
  if (loading) return <p>กำลังโหลด...</p>;
  if (error) return <p>ข้อผิดพลาด: {error}</p>;
  
  return (
    <ul>
      {users.map(user => (
        <li key={user.id}>{user.name} - {user.email}</li>
      ))}
    </ul>
  );
}
```

```jsx
// Pattern 2: Search with Debounce
function SearchUsers() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  const [loading, setLoading] = useState(false);
  
  useEffect(() => {
    if (!query.trim()) {
      setResults([]);
      return;
    }
    
    setLoading(true);
    const timer = setTimeout(async () => {
      try {
        const res = await fetch(
          `https://jsonplaceholder.typicode.com/users?username=${query}`
        );
        const data = await res.json();
        setResults(data);
      } catch (err) {
        console.error(err);
      } finally {
        setLoading(false);
      }
    }, 500);  // debounce 500ms
    
    return () => clearTimeout(timer);
  }, [query]);
  
  return (
    <div>
      <input
        value={query}
        onChange={e => setQuery(e.target.value)}
        placeholder="ค้นหาผู้ใช้..."
      />
      {loading && <p>กำลังค้นหา...</p>}
      {results.map(user => (
        <div key={user.id}>{user.name}</div>
      ))}
    </div>
  );
}
```

---

## Step 1416: useContext

```jsx
import { createContext, useContext, useState } from 'react';

// สร้าง Context
const ThemeContext = createContext({
  theme: 'light',
  toggleTheme: () => {}
});

// Provider Component
function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  
  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  };
  
  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

// Custom Hook สำหรับใช้ Context
function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme ต้องใช้ใน ThemeProvider');
  }
  return context;
}

// Component ที่ใช้ Context
function ThemedButton() {
  const { theme, toggleTheme } = useTheme();
  
  return (
    <button
      style={{
        backgroundColor: theme === 'light' ? '#fff' : '#333',
        color: theme === 'light' ? '#333' : '#fff',
        padding: '10px 20px',
        border: '1px solid #ccc',
        borderRadius: '4px'
      }}
      onClick={toggleTheme}
    >
      Toggle Theme ({theme})
    </button>
  );
}

function ThemedCard({ title, content }) {
  const { theme } = useTheme();
  
  return (
    <div style={{
      background: theme === 'light' ? '#f5f5f5' : '#444',
      color: theme === 'light' ? '#333' : '#fff',
      padding: '20px',
      borderRadius: '8px',
      margin: '10px 0'
    }}>
      <h3>{title}</h3>
      <p>{content}</p>
    </div>
  );
}

// App ใช้ Provider ครอบ
function App() {
  return (
    <ThemeProvider>
      <div>
        <ThemedButton />
        <ThemedCard title="บัตร 1" content="เนื้อหาของบัตร" />
        <ThemedCard title="บัตร 2" content="เนื้อหาอีกอัน" />
      </div>
    </ThemeProvider>
  );
}
```

```jsx
// Context สำหรับ Authentication
const AuthContext = createContext(null);

function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(false);
  
  const login = async (email, password) => {
    setLoading(true);
    try {
      // จำลองการ login
      await new Promise(resolve => setTimeout(resolve, 1000));
      setUser({ id: 1, email, name: 'สมชาย' });
    } finally {
      setLoading(false);
    }
  };
  
  const logout = () => setUser(null);
  
  return (
    <AuthContext.Provider value={{ user, loading, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

function useAuth() {
  return useContext(AuthContext);
}

function LoginPage() {
  const { login, loading } = useAuth();
  const [email, setEmail] = useState('');
  const [password, setPassword] = useState('');
  
  const handleSubmit = (e) => {
    e.preventDefault();
    login(email, password);
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input value={email} onChange={e => setEmail(e.target.value)} type="email" placeholder="อีเมล" />
      <input value={password} onChange={e => setPassword(e.target.value)} type="password" placeholder="รหัสผ่าน" />
      <button disabled={loading}>{loading ? 'กำลังเข้าสู่ระบบ...' : 'เข้าสู่ระบบ'}</button>
    </form>
  );
}
```

---

## Step 1417: useReducer

```jsx
import { useReducer } from 'react';

// useReducer เหมาะกับ state ที่ซับซ้อน

// 1. กำหนด initial state
const initialState = {
  count: 0,
  step: 1,
  history: []
};

// 2. สร้าง reducer function
function counterReducer(state, action) {
  switch (action.type) {
    case 'INCREMENT':
      return {
        ...state,
        count: state.count + state.step,
        history: [...state.history, state.count + state.step]
      };
    case 'DECREMENT':
      return {
        ...state,
        count: state.count - state.step,
        history: [...state.history, state.count - state.step]
      };
    case 'RESET':
      return { ...initialState };
    case 'SET_STEP':
      return { ...state, step: action.payload };
    case 'SET_COUNT':
      return { ...state, count: action.payload };
    default:
      throw new Error(`Unknown action type: ${action.type}`);
  }
}

// 3. ใช้ useReducer ใน Component
function Counter() {
  const [state, dispatch] = useReducer(counterReducer, initialState);
  
  return (
    <div>
      <p>นับ: {state.count}</p>
      <p>ก้าว: {state.step}</p>
      
      <div>
        <button onClick={() => dispatch({ type: 'INCREMENT' })}>+</button>
        <button onClick={() => dispatch({ type: 'DECREMENT' })}>-</button>
        <button onClick={() => dispatch({ type: 'RESET' })}>รีเซ็ต</button>
      </div>
      
      <div>
        <label>ก้าวต่อครั้ง: </label>
        <input
          type="number"
          value={state.step}
          min={1}
          onChange={e => dispatch({ type: 'SET_STEP', payload: Number(e.target.value) })}
        />
      </div>
      
      <div>
        <h4>ประวัติ:</h4>
        {state.history.map((val, i) => <span key={i}>{val} </span>)}
      </div>
    </div>
  );
}
```

```jsx
// useReducer สำหรับ Form State
const formInitial = {
  values: { name: '', email: '', message: '' },
  errors: {},
  isSubmitting: false,
  isSubmitted: false
};

function formReducer(state, action) {
  switch (action.type) {
    case 'CHANGE':
      return {
        ...state,
        values: { ...state.values, [action.field]: action.value },
        errors: { ...state.errors, [action.field]: '' }
      };
    case 'VALIDATE':
      const errors = {};
      if (!state.values.name) errors.name = 'กรุณากรอกชื่อ';
      if (!state.values.email) errors.email = 'กรุณากรอกอีเมล';
      if (!state.values.message) errors.message = 'กรุณากรอกข้อความ';
      return { ...state, errors };
    case 'SUBMIT_START':
      return { ...state, isSubmitting: true };
    case 'SUBMIT_SUCCESS':
      return { ...state, isSubmitting: false, isSubmitted: true };
    case 'SUBMIT_ERROR':
      return { ...state, isSubmitting: false };
    case 'RESET':
      return { ...formInitial };
    default:
      return state;
  }
}

function ContactForm() {
  const [state, dispatch] = useReducer(formReducer, formInitial);
  
  const handleChange = (field) => (e) => {
    dispatch({ type: 'CHANGE', field, value: e.target.value });
  };
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    dispatch({ type: 'VALIDATE' });
    
    if (Object.keys(state.errors).some(k => state.errors[k])) return;
    
    dispatch({ type: 'SUBMIT_START' });
    await new Promise(r => setTimeout(r, 1000));
    dispatch({ type: 'SUBMIT_SUCCESS' });
  };
  
  if (state.isSubmitted) {
    return (
      <div>
        <p>ส่งข้อความสำเร็จ!</p>
        <button onClick={() => dispatch({ type: 'RESET' })}>ส่งใหม่</button>
      </div>
    );
  }
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        value={state.values.name}
        onChange={handleChange('name')}
        placeholder="ชื่อ"
      />
      {state.errors.name && <span>{state.errors.name}</span>}
      
      <input
        value={state.values.email}
        onChange={handleChange('email')}
        placeholder="อีเมล"
      />
      {state.errors.email && <span>{state.errors.email}</span>}
      
      <textarea
        value={state.values.message}
        onChange={handleChange('message')}
        placeholder="ข้อความ"
      />
      {state.errors.message && <span>{state.errors.message}</span>}
      
      <button disabled={state.isSubmitting}>
        {state.isSubmitting ? 'กำลังส่ง...' : 'ส่ง'}
      </button>
    </form>
  );
}
```

---

## Step 1418: useRef เชิงลึก

```jsx
import { useRef, useEffect } from 'react';

// useRef สำหรับ DOM manipulation
function VideoPlayer() {
  const videoRef = useRef(null);
  const [isPlaying, setIsPlaying] = useState(false);
  
  const handlePlayPause = () => {
    if (isPlaying) {
      videoRef.current.pause();
    } else {
      videoRef.current.play();
    }
    setIsPlaying(!isPlaying);
  };
  
  const handleSeek = (time) => {
    videoRef.current.currentTime = time;
  };
  
  return (
    <div>
      <video ref={videoRef} src="/video.mp4" width="400" />
      <div>
        <button onClick={handlePlayPause}>
          {isPlaying ? 'หยุด' : 'เล่น'}
        </button>
        <button onClick={() => handleSeek(0)}>ย้อนไปต้น</button>
        <button onClick={() => handleSeek(30)}>ข้ามไป 30s</button>
      </div>
    </div>
  );
}
```

```jsx
// useRef สำหรับ mutable value ที่ไม่ trigger re-render
function RenderCounter() {
  const [count, setCount] = useState(0);
  const renderCount = useRef(0);
  const lastRenderTime = useRef(Date.now());
  
  // เพิ่ม renderCount ทุกครั้งที่ render
  renderCount.current += 1;
  
  const now = Date.now();
  const timeSinceLastRender = now - lastRenderTime.current;
  lastRenderTime.current = now;
  
  return (
    <div>
      <p>Count: {count}</p>
      <p>Render ครั้งที่: {renderCount.current}</p>
      <p>ห่างจาก render ก่อน: {timeSinceLastRender}ms</p>
      <button onClick={() => setCount(c => c + 1)}>เพิ่ม</button>
    </div>
  );
}
```

```jsx
// useRef สำหรับ scroll
function ScrollExample() {
  const topRef = useRef(null);
  const bottomRef = useRef(null);
  const sectionRefs = useRef({});
  
  const scrollToTop = () => {
    topRef.current?.scrollIntoView({ behavior: 'smooth' });
  };
  
  const scrollToBottom = () => {
    bottomRef.current?.scrollIntoView({ behavior: 'smooth' });
  };
  
  const scrollToSection = (id) => {
    sectionRefs.current[id]?.scrollIntoView({ behavior: 'smooth' });
  };
  
  const sections = ['บทที่ 1', 'บทที่ 2', 'บทที่ 3', 'บทที่ 4'];
  
  return (
    <div>
      <div ref={topRef} style={{ padding: '20px', background: '#f0f0f0' }}>
        <p>ด้านบน</p>
        <div>
          {sections.map(s => (
            <button key={s} onClick={() => scrollToSection(s)}>{s}</button>
          ))}
        </div>
      </div>
      
      {sections.map(section => (
        <div
          key={section}
          ref={el => sectionRefs.current[section] = el}
          style={{ height: '400px', padding: '20px', borderTop: '1px solid #ccc' }}
        >
          <h2>{section}</h2>
          <p>เนื้อหาของ{section}</p>
        </div>
      ))}
      
      <div ref={bottomRef} style={{ padding: '20px' }}>
        <p>ด้านล่างสุด</p>
        <button onClick={scrollToTop}>กลับด้านบน</button>
      </div>
    </div>
  );
}
```

---

## Step 1419: useMemo

```jsx
import { useMemo, useState } from 'react';

// useMemo cache ผลการคำนวณ
function ExpensiveCalculation() {
  const [numbers, setNumbers] = useState([1, 2, 3, 4, 5, 10, 20, 100]);
  const [multiplier, setMultiplier] = useState(1);
  const [unrelated, setUnrelated] = useState(0);
  
  // ❌ ไม่ใช้ useMemo: คำนวณทุก render แม้ numbers ไม่เปลี่ยน
  const sumWithoutMemo = numbers.reduce((a, b) => a + b, 0) * multiplier;
  
  // ✅ ใช้ useMemo: คำนวณใหม่เฉพาะเมื่อ numbers หรือ multiplier เปลี่ยน
  const sum = useMemo(() => {
    console.log('Computing sum...');
    // จำลองการคำนวณที่หนัก
    return numbers.reduce((a, b) => a + b, 0) * multiplier;
  }, [numbers, multiplier]);
  
  const stats = useMemo(() => {
    const max = Math.max(...numbers);
    const min = Math.min(...numbers);
    const avg = sum / multiplier / numbers.length;
    return { max, min, avg };
  }, [numbers, sum, multiplier]);
  
  return (
    <div>
      <p>ผลรวม: {sum}</p>
      <p>Max: {stats.max}, Min: {stats.min}, Avg: {stats.avg.toFixed(2)}</p>
      
      <button onClick={() => setMultiplier(m => m + 1)}>เพิ่ม multiplier ({multiplier})</button>
      <button onClick={() => setUnrelated(u => u + 1)}>unrelated ({unrelated})</button>
      <button onClick={() => setNumbers([...numbers, Math.floor(Math.random() * 100)])}>
        เพิ่มตัวเลข
      </button>
    </div>
  );
}
```

```jsx
// useMemo สำหรับ filter/sort
function FilterableList() {
  const [items] = useState(() =>
    Array.from({ length: 10000 }, (_, i) => ({
      id: i,
      name: `รายการ ${i}`,
      category: ['A', 'B', 'C'][i % 3],
      value: Math.random() * 100
    }))
  );
  
  const [search, setSearch] = useState('');
  const [category, setCategory] = useState('all');
  const [sortBy, setSortBy] = useState('name');
  const [counter, setCounter] = useState(0);
  
  // useMemo ป้องกัน re-filter เมื่อ counter เปลี่ยน
  const filteredAndSorted = useMemo(() => {
    console.log('Filtering and sorting...');
    let result = items;
    
    if (search) {
      result = result.filter(item =>
        item.name.toLowerCase().includes(search.toLowerCase())
      );
    }
    
    if (category !== 'all') {
      result = result.filter(item => item.category === category);
    }
    
    result = [...result].sort((a, b) => {
      if (sortBy === 'name') return a.name.localeCompare(b.name);
      return a.value - b.value;
    });
    
    return result.slice(0, 50);  // แสดงแค่ 50 รายการ
  }, [items, search, category, sortBy]);
  
  return (
    <div>
      <div>
        <input value={search} onChange={e => setSearch(e.target.value)} placeholder="ค้นหา..." />
        <select value={category} onChange={e => setCategory(e.target.value)}>
          <option value="all">ทั้งหมด</option>
          <option value="A">A</option>
          <option value="B">B</option>
          <option value="C">C</option>
        </select>
        <select value={sortBy} onChange={e => setSortBy(e.target.value)}>
          <option value="name">เรียงตามชื่อ</option>
          <option value="value">เรียงตามค่า</option>
        </select>
        <button onClick={() => setCounter(c => c + 1)}>counter: {counter}</button>
      </div>
      
      <p>แสดง {filteredAndSorted.length} รายการ</p>
      <ul>
        {filteredAndSorted.map(item => (
          <li key={item.id}>
            [{item.category}] {item.name} - {item.value.toFixed(2)}
          </li>
        ))}
      </ul>
    </div>
  );
}
```

---

## Step 1420: useCallback

```jsx
import { useCallback, useState, memo } from 'react';

// useCallback cache ฟังก์ชัน
// ปัญหา: ทุก render สร้าง function ใหม่ ทำให้ child re-render

// Child component ที่ใช้ memo
const Button = memo(function Button({ onClick, children }) {
  console.log(`Button "${children}" re-rendered`);
  return <button onClick={onClick}>{children}</button>;
});

function Parent() {
  const [count, setCount] = useState(0);
  const [name, setName] = useState('');
  
  // ❌ สร้าง function ใหม่ทุก render
  const handleClickBad = () => {
    setCount(c => c + 1);
  };
  
  // ✅ cache function, สร้างใหม่เฉพาะเมื่อ dependency เปลี่ยน
  const handleClickGood = useCallback(() => {
    setCount(c => c + 1);
  }, []); // ไม่มี dependency เพราะใช้ functional update
  
  return (
    <div>
      <p>Count: {count}</p>
      <input value={name} onChange={e => setName(e.target.value)} placeholder="พิมพ์ชื่อ" />
      
      <Button onClick={handleClickBad}>ปุ่มแย่ (re-render ทุกครั้ง)</Button>
      <Button onClick={handleClickGood}>ปุ่มดี (re-render เมื่อจำเป็น)</Button>
    </div>
  );
}
```

```jsx
// useCallback กับ props
function DataList({ items, onItemSelect }) {
  const [selectedId, setSelectedId] = useState(null);
  
  // Cache handler ที่ขึ้นอยู่กับ onItemSelect
  const handleSelect = useCallback((item) => {
    setSelectedId(item.id);
    onItemSelect(item);
  }, [onItemSelect]);
  
  return (
    <ul>
      {items.map(item => (
        <ListItem
          key={item.id}
          item={item}
          isSelected={item.id === selectedId}
          onSelect={handleSelect}
        />
      ))}
    </ul>
  );
}

const ListItem = memo(function ListItem({ item, isSelected, onSelect }) {
  return (
    <li
      style={{ background: isSelected ? '#e0f0ff' : 'transparent' }}
      onClick={() => onSelect(item)}
    >
      {item.name}
    </li>
  );
});
```

---

## Step 1421: Custom Hooks

```jsx
// Custom Hook: useFetch
function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    let cancelled = false;
    
    const fetchData = async () => {
      setLoading(true);
      setError(null);
      
      try {
        const response = await fetch(url);
        if (!response.ok) throw new Error(`HTTP error! status: ${response.status}`);
        const json = await response.json();
        if (!cancelled) setData(json);
      } catch (err) {
        if (!cancelled) setError(err.message);
      } finally {
        if (!cancelled) setLoading(false);
      }
    };
    
    fetchData();
    return () => { cancelled = true; };
  }, [url]);
  
  return { data, loading, error };
}

// ใช้งาน
function UserProfile({ userId }) {
  const { data: user, loading, error } = useFetch(
    `https://jsonplaceholder.typicode.com/users/${userId}`
  );
  
  if (loading) return <p>กำลังโหลด...</p>;
  if (error) return <p>Error: {error}</p>;
  if (!user) return null;
  
  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
}
```

---

## Step 1422: Custom Hook - useLocalStorage

```jsx
// useLocalStorage
function useLocalStorage(key, initialValue) {
  const [storedValue, setStoredValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error('Error reading localStorage:', error);
      return initialValue;
    }
  });
  
  const setValue = useCallback((value) => {
    try {
      const valueToStore = value instanceof Function ? value(storedValue) : value;
      setStoredValue(valueToStore);
      window.localStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (error) {
      console.error('Error setting localStorage:', error);
    }
  }, [key, storedValue]);
  
  const removeValue = useCallback(() => {
    try {
      window.localStorage.removeItem(key);
      setStoredValue(initialValue);
    } catch (error) {
      console.error('Error removing from localStorage:', error);
    }
  }, [key, initialValue]);
  
  return [storedValue, setValue, removeValue];
}

// ใช้งาน
function SettingsPage() {
  const [theme, setTheme, removeTheme] = useLocalStorage('theme', 'light');
  const [language, setLanguage] = useLocalStorage('language', 'th');
  
  return (
    <div>
      <h2>การตั้งค่า</h2>
      <div>
        <label>ธีม:</label>
        <select value={theme} onChange={e => setTheme(e.target.value)}>
          <option value="light">สว่าง</option>
          <option value="dark">มืด</option>
        </select>
      </div>
      <div>
        <label>ภาษา:</label>
        <select value={language} onChange={e => setLanguage(e.target.value)}>
          <option value="th">ไทย</option>
          <option value="en">English</option>
        </select>
      </div>
      <button onClick={removeTheme}>รีเซ็ตธีม</button>
      <pre>{JSON.stringify({ theme, language }, null, 2)}</pre>
    </div>
  );
}
```

---

## Step 1423: Custom Hook - useDebounce และ useThrottle

```jsx
// useDebounce
function useDebounce(value, delay = 300) {
  const [debouncedValue, setDebouncedValue] = useState(value);
  
  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);
    
    return () => clearTimeout(timer);
  }, [value, delay]);
  
  return debouncedValue;
}

// useThrottle
function useThrottle(value, interval = 300) {
  const [throttledValue, setThrottledValue] = useState(value);
  const lastUpdated = useRef(Date.now());
  
  useEffect(() => {
    const now = Date.now();
    const timeSince = now - lastUpdated.current;
    
    if (timeSince >= interval) {
      setThrottledValue(value);
      lastUpdated.current = now;
    } else {
      const timer = setTimeout(() => {
        setThrottledValue(value);
        lastUpdated.current = Date.now();
      }, interval - timeSince);
      
      return () => clearTimeout(timer);
    }
  }, [value, interval]);
  
  return throttledValue;
}

// ใช้งาน
function SearchBox() {
  const [input, setInput] = useState('');
  const debouncedSearch = useDebounce(input, 500);
  const { data, loading } = useFetch(
    debouncedSearch
      ? `https://jsonplaceholder.typicode.com/posts?title_like=${debouncedSearch}`
      : null
  );
  
  return (
    <div>
      <input
        value={input}
        onChange={e => setInput(e.target.value)}
        placeholder="ค้นหา (debounced 500ms)..."
      />
      <p>ค้นหา: "{debouncedSearch}"</p>
      {loading && <p>กำลังค้นหา...</p>}
      {data?.slice(0, 5).map(post => (
        <div key={post.id}>{post.title}</div>
      ))}
    </div>
  );
}
```

---

## Step 1424: Custom Hook - useMediaQuery และ useWindowSize

```jsx
// useMediaQuery
function useMediaQuery(query) {
  const [matches, setMatches] = useState(false);
  
  useEffect(() => {
    const media = window.matchMedia(query);
    setMatches(media.matches);
    
    const handler = (e) => setMatches(e.matches);
    media.addEventListener('change', handler);
    
    return () => media.removeEventListener('change', handler);
  }, [query]);
  
  return matches;
}

// useWindowSize
function useWindowSize() {
  const [size, setSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight
  });
  
  useEffect(() => {
    const handleResize = () => {
      setSize({
        width: window.innerWidth,
        height: window.innerHeight
      });
    };
    
    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);
  
  return size;
}

// ใช้งาน
function ResponsiveComponent() {
  const isMobile = useMediaQuery('(max-width: 768px)');
  const isTablet = useMediaQuery('(min-width: 769px) and (max-width: 1024px)');
  const isDesktop = useMediaQuery('(min-width: 1025px)');
  const { width, height } = useWindowSize();
  
  const getDevice = () => {
    if (isMobile) return 'มือถือ';
    if (isTablet) return 'แท็บเล็ต';
    return 'คอมพิวเตอร์';
  };
  
  return (
    <div>
      <p>อุปกรณ์: {getDevice()}</p>
      <p>ขนาดหน้าจอ: {width} x {height}</p>
      
      {isMobile && <MobileMenu />}
      {!isMobile && <DesktopMenu />}
    </div>
  );
}
```

---

## Step 1425: Custom Hook - usePrevious

```jsx
// usePrevious
function usePrevious(value) {
  const ref = useRef();
  
  useEffect(() => {
    ref.current = value;
  });
  
  return ref.current;
}

// useChangeLog
function useChangeLog(value, label = 'value') {
  const [changes, setChanges] = useState([]);
  const previous = usePrevious(value);
  
  useEffect(() => {
    if (previous !== undefined && previous !== value) {
      setChanges(prev => [
        ...prev,
        {
          from: previous,
          to: value,
          time: new Date().toLocaleTimeString('th-TH')
        }
      ].slice(-10));
    }
  }, [value, previous]);
  
  return { previous, changes };
}

// ใช้งาน
function CounterWithLog() {
  const [count, setCount] = useState(0);
  const { previous, changes } = useChangeLog(count, 'count');
  
  return (
    <div>
      <p>ปัจจุบัน: {count}</p>
      <p>ก่อนหน้า: {previous ?? 'N/A'}</p>
      <button onClick={() => setCount(c => c + 1)}>+</button>
      <button onClick={() => setCount(c => c - 1)}>-</button>
      <button onClick={() => setCount(0)}>รีเซ็ต</button>
      
      <h4>ประวัติการเปลี่ยนแปลง:</h4>
      {changes.map((change, i) => (
        <p key={i}>
          {change.time}: {change.from} → {change.to}
        </p>
      ))}
    </div>
  );
}
```

---

## Step 1426: React.memo

```jsx
import { memo, useState, useCallback } from 'react';

// React.memo ป้องกัน re-render ที่ไม่จำเป็น
// Component จะ re-render เฉพาะเมื่อ props เปลี่ยน

// ไม่มี memo: re-render ทุกครั้งที่ parent render
function ExpensiveChild({ value }) {
  console.log('ExpensiveChild rendered');
  // จำลองการ render ที่หนัก
  let result = 0;
  for (let i = 0; i < 1000000; i++) result += i;
  
  return <div>Value: {value}</div>;
}

// มี memo: re-render เฉพาะเมื่อ value เปลี่ยน
const MemoizedChild = memo(function ExpensiveChild({ value }) {
  console.log('MemoizedChild rendered');
  return <div>Value: {value}</div>;
});

// Custom comparison function
const areEqual = (prevProps, nextProps) => {
  // return true = ไม่ re-render
  // return false = re-render
  return prevProps.value === nextProps.value;
};

const MemoWithCustomComparison = memo(function MyComponent({ value }) {
  return <div>{value}</div>;
}, areEqual);

function Parent() {
  const [count, setCount] = useState(0);
  const [text, setText] = useState('');
  
  return (
    <div>
      <input value={text} onChange={e => setText(e.target.value)} placeholder="พิมพ์อะไรก็ได้" />
      <button onClick={() => setCount(c => c + 1)}>count: {count}</button>
      
      {/* Re-render ทุกครั้งที่ Parent render */}
      <ExpensiveChild value="static" />
      
      {/* Re-render เฉพาะเมื่อ value เปลี่ยน */}
      <MemoizedChild value="static" />
    </div>
  );
}
```

---

## Step 1427: useLayoutEffect

```jsx
import { useLayoutEffect, useEffect, useRef, useState } from 'react';

// useLayoutEffect ทำงาน synchronously หลัง DOM update
// แต่ก่อนที่ browser จะ paint

function Tooltip({ children, text }) {
  const [position, setPosition] = useState({ top: 0, left: 0 });
  const tooltipRef = useRef(null);
  const triggerRef = useRef(null);
  
  // ใช้ useLayoutEffect เพื่อวัด DOM ก่อน paint
  // ป้องกัน flickering
  useLayoutEffect(() => {
    if (tooltipRef.current && triggerRef.current) {
      const trigger = triggerRef.current.getBoundingClientRect();
      const tooltip = tooltipRef.current.getBoundingClientRect();
      
      setPosition({
        top: trigger.top - tooltip.height - 8,
        left: trigger.left + trigger.width / 2 - tooltip.width / 2
      });
    }
  });
  
  return (
    <span style={{ position: 'relative' }}>
      <span ref={triggerRef}>{children}</span>
      <div
        ref={tooltipRef}
        style={{
          position: 'fixed',
          top: position.top,
          left: position.left,
          background: '#333',
          color: '#fff',
          padding: '4px 8px',
          borderRadius: '4px',
          fontSize: '14px',
          pointerEvents: 'none'
        }}
      >
        {text}
      </div>
    </span>
  );
}

// useEffect vs useLayoutEffect
function EffectComparison() {
  const [show, setShow] = useState(false);
  
  useEffect(() => {
    // Browser paint ไปแล้ว
    // ดีสำหรับ: fetch data, subscriptions, timers
  });
  
  useLayoutEffect(() => {
    // ก่อน browser paint
    // ดีสำหรับ: DOM measurements, animations
  });
  
  return <button onClick={() => setShow(!show)}>Toggle</button>;
}
```

---

## Step 1428: React 18 Hooks - useId, useTransition, useDeferredValue

```jsx
import { useId, useTransition, useDeferredValue, useState } from 'react';

// useId: สร้าง unique ID ที่ consistent ระหว่าง server และ client
function FormField({ label, type = 'text' }) {
  const id = useId();
  
  return (
    <div>
      <label htmlFor={id}>{label}</label>
      <input id={id} type={type} />
    </div>
  );
}

// ใช้หลาย instance โดยไม่ conflict
function LoginForm() {
  return (
    <form>
      <FormField label="อีเมล" type="email" />  {/* id: :r0: */}
      <FormField label="รหัสผ่าน" type="password" />  {/* id: :r1: */}
    </form>
  );
}
```

```jsx
// useTransition: mark non-urgent updates
function TabSwitcher() {
  const [activeTab, setActiveTab] = useState('tab1');
  const [isPending, startTransition] = useTransition();
  
  const handleTabChange = (tab) => {
    // Mark เป็น non-urgent transition
    startTransition(() => {
      setActiveTab(tab);
    });
  };
  
  return (
    <div>
      <div className="tabs">
        {['tab1', 'tab2', 'tab3'].map(tab => (
          <button
            key={tab}
            onClick={() => handleTabChange(tab)}
            disabled={isPending}
            style={{ opacity: isPending ? 0.5 : 1 }}
          >
            {tab}
            {isPending && activeTab === tab && ' (กำลังโหลด...)'}
          </button>
        ))}
      </div>
      
      {isPending ? (
        <p>กำลังเปลี่ยนหน้า...</p>
      ) : (
        <TabContent tab={activeTab} />
      )}
    </div>
  );
}

function TabContent({ tab }) {
  // จำลอง component ที่หนัก
  const items = Array.from({ length: 1000 }, (_, i) => `${tab} item ${i}`);
  return (
    <ul>
      {items.slice(0, 10).map(item => (
        <li key={item}>{item}</li>
      ))}
    </ul>
  );
}
```

```jsx
// useDeferredValue: defer ค่าที่ไม่จำเป็นต้องอัพเดทเร็ว
function SearchResults({ query }) {
  // Defer การ render ผลลัพธ์
  // ให้ input response ก่อน แล้วค่อย update results
  
  const deferredQuery = useDeferredValue(query);
  
  // Stale content ถ้า deferredQuery ไม่ตรงกับ query
  const isStale = deferredQuery !== query;
  
  const results = useMemo(() => {
    // จำลอง heavy computation
    return Array.from({ length: 1000 }, (_, i) => `${deferredQuery} result ${i}`)
      .filter(r => r.includes(deferredQuery));
  }, [deferredQuery]);
  
  return (
    <ul style={{ opacity: isStale ? 0.5 : 1 }}>
      {results.slice(0, 10).map(result => (
        <li key={result}>{result}</li>
      ))}
    </ul>
  );
}

function SearchPage() {
  const [query, setQuery] = useState('');
  
  return (
    <div>
      <input
        value={query}
        onChange={e => setQuery(e.target.value)}
        placeholder="ค้นหา..."
      />
      <SearchResults query={query} />
    </div>
  );
}
```

---

## Step 1429: Building Complete Custom Hooks Library

```jsx
// useToggle
function useToggle(initial = false) {
  const [value, setValue] = useState(initial);
  
  const toggle = useCallback(() => setValue(v => !v), []);
  const setTrue = useCallback(() => setValue(true), []);
  const setFalse = useCallback(() => setValue(false), []);
  
  return [value, { toggle, setTrue, setFalse }];
}

// useCounter
function useCounter(initial = 0, options = {}) {
  const { min = -Infinity, max = Infinity, step = 1 } = options;
  const [count, setCount] = useState(initial);
  
  const increment = useCallback(() => {
    setCount(c => Math.min(c + step, max));
  }, [step, max]);
  
  const decrement = useCallback(() => {
    setCount(c => Math.max(c - step, min));
  }, [step, min]);
  
  const reset = useCallback(() => setCount(initial), [initial]);
  const set = useCallback((value) => {
    setCount(Math.min(Math.max(value, min), max));
  }, [min, max]);
  
  return { count, increment, decrement, reset, set };
}

// useInterval
function useInterval(callback, delay) {
  const savedCallback = useRef(callback);
  
  useEffect(() => {
    savedCallback.current = callback;
  }, [callback]);
  
  useEffect(() => {
    if (delay === null) return;
    
    const id = setInterval(() => savedCallback.current(), delay);
    return () => clearInterval(id);
  }, [delay]);
}

// useOnClickOutside
function useOnClickOutside(ref, handler) {
  useEffect(() => {
    const listener = (event) => {
      if (!ref.current || ref.current.contains(event.target)) return;
      handler(event);
    };
    
    document.addEventListener('mousedown', listener);
    document.addEventListener('touchstart', listener);
    
    return () => {
      document.removeEventListener('mousedown', listener);
      document.removeEventListener('touchstart', listener);
    };
  }, [ref, handler]);
}

// useKeyPress
function useKeyPress(targetKey) {
  const [keyPressed, setKeyPressed] = useState(false);
  
  useEffect(() => {
    const downHandler = ({ key }) => {
      if (key === targetKey) setKeyPressed(true);
    };
    const upHandler = ({ key }) => {
      if (key === targetKey) setKeyPressed(false);
    };
    
    window.addEventListener('keydown', downHandler);
    window.addEventListener('keyup', upHandler);
    
    return () => {
      window.removeEventListener('keydown', downHandler);
      window.removeEventListener('keyup', upHandler);
    };
  }, [targetKey]);
  
  return keyPressed;
}

// ใช้งาน hooks ทั้งหมด
function HooksDemo() {
  const [isOpen, { toggle: toggleMenu }] = useToggle(false);
  const { count, increment, decrement, reset } = useCounter(0, { min: 0, max: 10 });
  const menuRef = useRef(null);
  const escPressed = useKeyPress('Escape');
  
  useOnClickOutside(menuRef, () => {
    if (isOpen) toggleMenu();
  });
  
  useEffect(() => {
    if (escPressed && isOpen) toggleMenu();
  }, [escPressed, isOpen, toggleMenu]);
  
  useInterval(() => {
    console.log('tick');
  }, isOpen ? 1000 : null);
  
  return (
    <div>
      <div ref={menuRef}>
        <button onClick={toggleMenu}>เมนู ({isOpen ? 'เปิด' : 'ปิด'})</button>
        {isOpen && (
          <ul>
            <li>รายการ 1</li>
            <li>รายการ 2</li>
          </ul>
        )}
      </div>
      
      <div>
        <button onClick={decrement}>-</button>
        <span>{count}</span>
        <button onClick={increment}>+</button>
        <button onClick={reset}>รีเซ็ต</button>
      </div>
      
      <p>กด Escape เพื่อปิดเมนู: {escPressed ? 'กด' : 'ไม่กด'}</p>
    </div>
  );
}
```

---

## Step 1430: สรุปและ Best Practices

```jsx
// Best Practices สำหรับ Hooks

// 1. แยก concerns ด้วย custom hooks
function useUserData(userId) {
  const { data: user, loading, error } = useFetch(`/api/users/${userId}`);
  const { data: posts } = useFetch(user ? `/api/users/${userId}/posts` : null);
  
  return { user, posts, loading, error };
}

// 2. ใช้ TypeScript สำหรับ type safety
// function useLocalStorage<T>(key: string, initialValue: T): [T, (value: T) => void]

// 3. ตั้งชื่อ custom hook ขึ้นต้นด้วย "use"
function useFeatureFlag(flagName) {
  // ✅ ถูก
}

function featureFlag(flagName) {
  // ❌ ผิด - ไม่ใช่ custom hook
}

// 4. Document ว่า hook return อะไร
/**
 * useCounter - จัดการ counter state
 * @param {number} initial - ค่าเริ่มต้น
 * @returns {{ count: number, increment: () => void, decrement: () => void, reset: () => void }}
 */

// 5. อย่า optimize ก่อนจำเป็น
// ใช้ useMemo/useCallback เฉพาะเมื่อมีปัญหา performance จริงๆ

function EfficientComponent({ data, onSelect }) {
  // ✅ ดี: useMemo สำหรับ expensive computation
  const processedData = useMemo(() => {
    return data.filter(item => item.active)
              .sort((a, b) => b.score - a.score)
              .slice(0, 10);
  }, [data]);
  
  // ✅ ดี: useCallback สำหรับ function ที่ส่งให้ memo component
  const handleClick = useCallback((item) => {
    onSelect(item);
  }, [onSelect]);
  
  return (
    <ul>
      {processedData.map(item => (
        <MemoizedItem key={item.id} item={item} onClick={handleClick} />
      ))}
    </ul>
  );
}
```

---

## แบบฝึกหัด (Exercises)

### แบบฝึกหัดที่ 1: useForm Custom Hook
สร้าง `useForm` hook ที่จัดการ:
- form values
- validation rules
- error messages
- submit handling
- reset functionality

### แบบฝึกหัดที่ 2: usePagination Hook
สร้าง `usePagination` hook ที่:
- รับ total items และ items per page
- คำนวณหน้าปัจจุบัน
- มี next/prev/goTo functions
- return page numbers สำหรับแสดง

### แบบฝึกหัดที่ 3: useInfiniteScroll Hook
สร้าง hook ที่ detect เมื่อ scroll ถึงด้านล่าง และ fetch ข้อมูลเพิ่ม

### แบบฝึกหัดที่ 4: Context + Reducer
สร้าง global shopping cart โดยใช้ Context + useReducer สำหรับ:
- เพิ่ม/ลบสินค้า
- เปลี่ยนจำนวน
- คำนวณยอดรวม
- clear cart

### แบบฝึกหัดที่ 5: Performance Optimization
สร้าง component ที่แสดง list 10,000 รายการ พร้อม search และ filter โดยใช้ useMemo, useCallback และ React.memo

---

## สรุปท้ายส่วน

ใน Part 72 นี้เราได้เรียนรู้:

- **Rules of Hooks**: เรียกที่ top level, ใน component หรือ custom hook เท่านั้น
- **useState**: จัดการ state ทุกประเภท, lazy init, functional updates
- **useEffect**: side effects, dependencies, cleanup
- **useContext**: global state ผ่าน Context
- **useReducer**: complex state ด้วย reducer pattern
- **useRef**: DOM access และ mutable values
- **useMemo**: cache expensive calculations
- **useCallback**: cache functions เพื่อ optimize
- **Custom Hooks**: สร้าง useFetch, useLocalStorage, useDebounce, useMediaQuery, usePrevious ฯลฯ
- **React.memo**: ป้องกัน re-render ที่ไม่จำเป็น
- **React 18 Hooks**: useId, useTransition, useDeferredValue

Part 73 จะเรียนเรื่อง State Management ด้วย Zustand และ Redux Toolkit!
