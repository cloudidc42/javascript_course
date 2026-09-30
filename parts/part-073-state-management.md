# Part 73: State Management (Steps 1431-1450)

## บทนำ

เมื่อ application ของเราเติบโตขึ้น การจัดการ state เริ่มซับซ้อน component ต่างๆ ต้องการแชร์ข้อมูลกัน และการส่ง props หลายชั้น (prop drilling) กลายเป็นปัญหา ในส่วนนี้เราจะเรียนรู้วิธีจัดการ state อย่างมีประสิทธิภาพด้วย Context API, Zustand, และ Redux Toolkit

---

## Step 1431: ปัญหาของ State Management

### Prop Drilling Problem

```jsx
// ปัญหา: ต้องส่ง user ผ่านหลายชั้น
function App() {
  const [user, setUser] = useState({ name: 'สมชาย', role: 'admin' });
  
  return <Dashboard user={user} setUser={setUser} />;
}

function Dashboard({ user, setUser }) {
  // Dashboard ไม่ได้ใช้ user โดยตรง แต่ต้องส่งต่อ
  return (
    <div>
      <Sidebar user={user} setUser={setUser} />
      <MainContent user={user} />
    </div>
  );
}

function Sidebar({ user, setUser }) {
  // Sidebar ก็ส่งต่ออีก
  return <UserMenu user={user} setUser={setUser} />;
}

function UserMenu({ user, setUser }) {
  // UserMenu ต้องการ user จริงๆ
  return (
    <div>
      <p>{user.name}</p>
      <button onClick={() => setUser(null)}>ออกจากระบบ</button>
    </div>
  );
}
```

### State Synchronization Problem

```jsx
// ปัญหา: state อยู่หลายที่, อาจไม่ sync กัน
function HeaderCart() {
  const [cartCount, setCartCount] = useState(0);
  // ...
}

function SidebarCart() {
  const [cartCount, setCartCount] = useState(0);  // ต่างกัน!
  // ...
}

// เมื่อเพิ่มสินค้า ต้องอัพเดทหลายที่
```

### เมื่อไหรควรใช้ Global State?

```
Local State (useState):
✓ Form values ใน component เดียว
✓ UI state (open/close, hover)
✓ Data เฉพาะ component

Global State:
✓ User authentication
✓ Shopping cart
✓ Theme/Language settings
✓ Data ที่หลาย component ใช้ร่วม
✓ Cache จาก API
```

---

## Step 1432: Context API สำหรับ State Management

```jsx
import { createContext, useContext, useReducer, useCallback } from 'react';

// Shopping Cart ด้วย Context + useReducer

// Types
const CART_ACTIONS = {
  ADD_ITEM: 'ADD_ITEM',
  REMOVE_ITEM: 'REMOVE_ITEM',
  UPDATE_QUANTITY: 'UPDATE_QUANTITY',
  CLEAR_CART: 'CLEAR_CART',
  APPLY_DISCOUNT: 'APPLY_DISCOUNT'
};

// Initial State
const initialCartState = {
  items: [],
  discount: 0,
  couponCode: null
};

// Reducer
function cartReducer(state, action) {
  switch (action.type) {
    case CART_ACTIONS.ADD_ITEM: {
      const existing = state.items.find(item => item.id === action.payload.id);
      if (existing) {
        return {
          ...state,
          items: state.items.map(item =>
            item.id === action.payload.id
              ? { ...item, quantity: item.quantity + 1 }
              : item
          )
        };
      }
      return {
        ...state,
        items: [...state.items, { ...action.payload, quantity: 1 }]
      };
    }
    
    case CART_ACTIONS.REMOVE_ITEM:
      return {
        ...state,
        items: state.items.filter(item => item.id !== action.payload)
      };
    
    case CART_ACTIONS.UPDATE_QUANTITY: {
      const { id, quantity } = action.payload;
      if (quantity <= 0) {
        return {
          ...state,
          items: state.items.filter(item => item.id !== id)
        };
      }
      return {
        ...state,
        items: state.items.map(item =>
          item.id === id ? { ...item, quantity } : item
        )
      };
    }
    
    case CART_ACTIONS.CLEAR_CART:
      return { ...initialCartState };
    
    case CART_ACTIONS.APPLY_DISCOUNT:
      return {
        ...state,
        discount: action.payload.discount,
        couponCode: action.payload.code
      };
    
    default:
      return state;
  }
}

// Context
const CartContext = createContext(null);

// Provider
export function CartProvider({ children }) {
  const [state, dispatch] = useReducer(cartReducer, initialCartState);
  
  const addItem = useCallback((item) => {
    dispatch({ type: CART_ACTIONS.ADD_ITEM, payload: item });
  }, []);
  
  const removeItem = useCallback((id) => {
    dispatch({ type: CART_ACTIONS.REMOVE_ITEM, payload: id });
  }, []);
  
  const updateQuantity = useCallback((id, quantity) => {
    dispatch({ type: CART_ACTIONS.UPDATE_QUANTITY, payload: { id, quantity } });
  }, []);
  
  const clearCart = useCallback(() => {
    dispatch({ type: CART_ACTIONS.CLEAR_CART });
  }, []);
  
  const applyDiscount = useCallback((code, discount) => {
    dispatch({ type: CART_ACTIONS.APPLY_DISCOUNT, payload: { code, discount } });
  }, []);
  
  // Computed values
  const totalItems = state.items.reduce((sum, item) => sum + item.quantity, 0);
  const subtotal = state.items.reduce((sum, item) => sum + item.price * item.quantity, 0);
  const discountAmount = subtotal * (state.discount / 100);
  const total = subtotal - discountAmount;
  
  const value = {
    items: state.items,
    discount: state.discount,
    couponCode: state.couponCode,
    totalItems,
    subtotal,
    discountAmount,
    total,
    addItem,
    removeItem,
    updateQuantity,
    clearCart,
    applyDiscount
  };
  
  return <CartContext.Provider value={value}>{children}</CartContext.Provider>;
}

// Custom Hook
export function useCart() {
  const context = useContext(CartContext);
  if (!context) throw new Error('useCart ต้องใช้ใน CartProvider');
  return context;
}
```

```jsx
// Components ที่ใช้ Cart Context
function ProductCard({ product }) {
  const { addItem } = useCart();
  
  return (
    <div className="product-card">
      <img src={product.image} alt={product.name} />
      <h3>{product.name}</h3>
      <p>฿{product.price.toLocaleString()}</p>
      <button onClick={() => addItem(product)}>เพิ่มในตะกร้า</button>
    </div>
  );
}

function CartIcon() {
  const { totalItems } = useCart();
  
  return (
    <div style={{ position: 'relative', display: 'inline-block' }}>
      🛒
      {totalItems > 0 && (
        <span style={{
          position: 'absolute', top: -8, right: -8,
          background: 'red', color: 'white',
          borderRadius: '50%', width: 20, height: 20,
          display: 'flex', alignItems: 'center', justifyContent: 'center',
          fontSize: 12
        }}>
          {totalItems}
        </span>
      )}
    </div>
  );
}

function CartSummary() {
  const { items, subtotal, discount, discountAmount, total, updateQuantity, removeItem, clearCart } = useCart();
  
  if (items.length === 0) return <p>ตะกร้าของคุณว่างเปล่า</p>;
  
  return (
    <div>
      {items.map(item => (
        <div key={item.id} style={{ display: 'flex', gap: 10, marginBottom: 10 }}>
          <span>{item.name}</span>
          <input
            type="number"
            value={item.quantity}
            min={0}
            onChange={e => updateQuantity(item.id, Number(e.target.value))}
            style={{ width: 60 }}
          />
          <span>฿{(item.price * item.quantity).toLocaleString()}</span>
          <button onClick={() => removeItem(item.id)}>ลบ</button>
        </div>
      ))}
      
      <hr />
      <p>ราคาก่อนลด: ฿{subtotal.toLocaleString()}</p>
      {discount > 0 && <p>ส่วนลด {discount}%: -฿{discountAmount.toLocaleString()}</p>}
      <p style={{ fontWeight: 'bold' }}>รวม: ฿{total.toLocaleString()}</p>
      
      <button onClick={clearCart}>ล้างตะกร้า</button>
    </div>
  );
}
```

---

## Step 1433: ข้อจำกัดของ Context API

```jsx
// ปัญหา 1: Re-render ทั้งหมด
// เมื่อ context value เปลี่ยน ทุก consumer จะ re-render

const AppContext = createContext(null);

function AppProvider({ children }) {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState('light');
  const [notifications, setNotifications] = useState([]);
  
  // ❌ ปัญหา: ถ้า notifications เปลี่ยน ทั้ง user และ theme consumer จะ re-render ด้วย
  const value = { user, setUser, theme, setTheme, notifications, setNotifications };
  
  return <AppContext.Provider value={value}>{children}</AppContext.Provider>;
}

// แก้ปัญหา: แยก context ตาม concern
const UserContext = createContext(null);
const ThemeContext = createContext(null);
const NotificationContext = createContext(null);

function BetterAppProvider({ children }) {
  const [user, setUser] = useState(null);
  const [theme, setTheme] = useState('light');
  const [notifications, setNotifications] = useState([]);
  
  return (
    <UserContext.Provider value={{ user, setUser }}>
      <ThemeContext.Provider value={{ theme, setTheme }}>
        <NotificationContext.Provider value={{ notifications, setNotifications }}>
          {children}
        </NotificationContext.Provider>
      </ThemeContext.Provider>
    </UserContext.Provider>
  );
}
```

```jsx
// ปัญหา 2: Context ไม่เหมาะกับข้อมูลที่ update บ่อย
// เช่น mouse position, scroll position, real-time data

// สำหรับข้อมูลที่ update บ่อย ใช้ Zustand หรือ Jotai แทน
```

---

## Step 1434: Zustand Introduction และ Setup

```bash
# ติดตั้ง Zustand
npm install zustand

# Zustand เป็น lightweight state management
# ไม่ต้องใช้ Provider
# ใช้ hooks-based API
# Minimal boilerplate
```

```jsx
// Zustand store พื้นฐาน
import { create } from 'zustand';

// สร้าง store
const useCounterStore = create((set) => ({
  count: 0,
  
  increment: () => set((state) => ({ count: state.count + 1 })),
  decrement: () => set((state) => ({ count: state.count - 1 })),
  reset: () => set({ count: 0 }),
  setCount: (value) => set({ count: value })
}));

// ใช้ใน component - ไม่ต้องมี Provider!
function Counter() {
  const { count, increment, decrement, reset } = useCounterStore();
  
  return (
    <div>
      <p>นับ: {count}</p>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
      <button onClick={reset}>รีเซ็ต</button>
    </div>
  );
}

// ใช้ได้จากที่ไหนก็ได้ในแอป
function CounterDisplay() {
  const count = useCounterStore((state) => state.count);  // Select เฉพาะ count
  return <span>จำนวน: {count}</span>;
}
```

---

## Step 1435: Zustand Store Creation เชิงลึก

```jsx
import { create } from 'zustand';
import { persist, devtools } from 'zustand/middleware';

// Store ที่ซับซ้อน - Shopping Cart
const useCartStore = create(
  devtools(
    persist(
      (set, get) => ({
        // State
        items: [],
        isOpen: false,
        
        // Computed (getter)
        get totalItems() {
          return get().items.reduce((sum, item) => sum + item.quantity, 0);
        },
        get total() {
          return get().items.reduce((sum, item) => sum + item.price * item.quantity, 0);
        },
        
        // Actions
        openCart: () => set({ isOpen: true }),
        closeCart: () => set({ isOpen: false }),
        toggleCart: () => set((state) => ({ isOpen: !state.isOpen })),
        
        addItem: (product) => set((state) => {
          const existing = state.items.find(item => item.id === product.id);
          if (existing) {
            return {
              items: state.items.map(item =>
                item.id === product.id
                  ? { ...item, quantity: item.quantity + 1 }
                  : item
              )
            };
          }
          return { items: [...state.items, { ...product, quantity: 1 }] };
        }),
        
        removeItem: (id) => set((state) => ({
          items: state.items.filter(item => item.id !== id)
        })),
        
        updateQuantity: (id, quantity) => set((state) => {
          if (quantity <= 0) {
            return { items: state.items.filter(item => item.id !== id) };
          }
          return {
            items: state.items.map(item =>
              item.id === id ? { ...item, quantity } : item
            )
          };
        }),
        
        clearCart: () => set({ items: [] }),
      }),
      {
        name: 'cart-storage',  // localStorage key
        partialize: (state) => ({ items: state.items })  // persist เฉพาะ items
      }
    ),
    { name: 'CartStore' }  // DevTools name
  )
);

export default useCartStore;
```

---

## Step 1436: Actions ใน Zustand

```jsx
import { create } from 'zustand';

// Actions แบบต่างๆ
const useUserStore = create((set, get) => ({
  user: null,
  users: [],
  isLoading: false,
  error: null,
  
  // Simple action
  setUser: (user) => set({ user }),
  
  // Action ที่อ่าน state เดิม
  toggleFavorite: (userId) => set((state) => ({
    users: state.users.map(u =>
      u.id === userId ? { ...u, favorite: !u.favorite } : u
    )
  })),
  
  // Async action
  fetchUsers: async () => {
    set({ isLoading: true, error: null });
    try {
      const response = await fetch('/api/users');
      const users = await response.json();
      set({ users, isLoading: false });
    } catch (error) {
      set({ error: error.message, isLoading: false });
    }
  },
  
  // Action ที่ใช้ get() เพื่ออ่าน state ปัจจุบัน
  getUserById: (id) => {
    return get().users.find(u => u.id === id);
  },
  
  // Action ที่ส่งออก action อื่น
  deleteUserAndRefresh: async (id) => {
    await fetch(`/api/users/${id}`, { method: 'DELETE' });
    await get().fetchUsers();  // เรียก action อื่น
  }
}));
```

---

## Step 1437: Selectors ใน Zustand

```jsx
// Selectors: เลือกเฉพาะส่วนที่ต้องการ
// ป้องกัน re-render ที่ไม่จำเป็น

const useCartStore = create((set) => ({
  items: [],
  discount: 0,
  addItem: (item) => set((state) => ({ items: [...state.items, item] }))
}));

// ❌ ไม่ดี: subscribe ทั้ง store
function TotalDisplay() {
  const store = useCartStore();  // re-render เมื่อ store เปลี่ยนแม้แต่ส่วนเล็กน้อย
  const total = store.items.reduce((sum, item) => sum + item.price * item.quantity, 0);
  return <span>฿{total}</span>;
}

// ✅ ดี: select เฉพาะที่ต้องการ
function TotalDisplay() {
  const total = useCartStore((state) =>
    state.items.reduce((sum, item) => sum + item.price * item.quantity, 0)
  );
  return <span>฿{total}</span>;
}

// ✅ ดีมาก: สร้าง selector แยก
const selectTotal = (state) =>
  state.items.reduce((sum, item) => sum + item.price * item.quantity, 0);

const selectItemCount = (state) =>
  state.items.reduce((sum, item) => sum + item.quantity, 0);

const selectDiscount = (state) => state.discount;

function CartSummary() {
  const total = useCartStore(selectTotal);
  const count = useCartStore(selectItemCount);
  
  return (
    <div>
      <p>{count} ชิ้น</p>
      <p>฿{total.toLocaleString()}</p>
    </div>
  );
}
```

```jsx
// Shallow comparison สำหรับ object selectors
import { shallow } from 'zustand/shallow';

function CartActions() {
  // ✅ ใช้ shallow เมื่อ select หลาย values
  const { addItem, removeItem, clearCart } = useCartStore(
    (state) => ({
      addItem: state.addItem,
      removeItem: state.removeItem,
      clearCart: state.clearCart
    }),
    shallow  // เปรียบเทียบแบบ shallow แทน reference
  );
  
  return (
    <div>
      <button onClick={clearCart}>ล้างตะกร้า</button>
    </div>
  );
}
```

---

## Step 1438: Redux Overview

```
Redux ทำงานอย่างไร:

┌─────────┐    dispatch(action)    ┌─────────┐
│  View   │ ──────────────────→  │  Store  │
│(React)  │                       │         │
└─────────┘                       └────┬────┘
     ↑                                 │
     │         new state               │ reducer(state, action)
     └─────────────────────────────────┘

1. User กระทำ action (เช่น คลิกปุ่ม)
2. Dispatch action ไปยัง store
3. Reducer รับ action และ state เดิม สร้าง state ใหม่
4. Store อัพเดท state ใหม่
5. Component ที่ subscribe จะ re-render
```

### ข้อดีของ Redux

```
1. Predictable: state เปลี่ยนได้แค่ผ่าน reducer
2. Centralized: state ทั้งหมดอยู่ที่เดียว
3. Debuggable: Redux DevTools ดู time-travel debugging
4. Testable: pure functions ทดสอบง่าย
```

---

## Step 1439: Redux Toolkit Setup

```bash
# ติดตั้ง
npm install @reduxjs/toolkit react-redux
```

```jsx
// store/index.js - Configure Store
import { configureStore } from '@reduxjs/toolkit';
import counterReducer from './counterSlice';
import cartReducer from './cartSlice';
import userReducer from './userSlice';

export const store = configureStore({
  reducer: {
    counter: counterReducer,
    cart: cartReducer,
    user: userReducer
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware({
      serializableCheck: {
        // ละเว้น non-serializable values บางอัน
        ignoredActions: ['user/setToken'],
        ignoredPaths: ['user.expiry']
      }
    })
});

// TypeScript: export types
export type RootState = ReturnType<typeof store.getState>;
export type AppDispatch = typeof store.dispatch;
```

```jsx
// main.jsx - Wrap App ด้วย Provider
import { Provider } from 'react-redux';
import { store } from './store';

ReactDOM.createRoot(document.getElementById('root')).render(
  <Provider store={store}>
    <App />
  </Provider>
);
```

---

## Step 1440: createSlice

```jsx
// store/counterSlice.js
import { createSlice, PayloadAction } from '@reduxjs/toolkit';

// createSlice รวม action creators และ reducer ไว้ด้วยกัน
const counterSlice = createSlice({
  name: 'counter',
  initialState: {
    value: 0,
    step: 1,
    history: []
  },
  reducers: {
    // Immer ทำให้เขียน mutation code ได้ (ไม่ต้อง spread)
    increment: (state) => {
      state.value += state.step;
      state.history.push({ action: 'increment', value: state.value });
    },
    decrement: (state) => {
      state.value -= state.step;
      state.history.push({ action: 'decrement', value: state.value });
    },
    reset: (state) => {
      state.value = 0;
      state.history = [];
    },
    setStep: (state, action) => {
      state.step = action.payload;
    },
    incrementByAmount: (state, action) => {
      state.value += action.payload;
    }
  }
});

// Export action creators
export const { increment, decrement, reset, setStep, incrementByAmount } = counterSlice.actions;

// Export reducer
export default counterSlice.reducer;

// Selectors
export const selectCount = (state) => state.counter.value;
export const selectStep = (state) => state.counter.step;
export const selectHistory = (state) => state.counter.history;
```

```jsx
// store/cartSlice.js - ตัวอย่างที่ซับซ้อนกว่า
import { createSlice } from '@reduxjs/toolkit';

const cartSlice = createSlice({
  name: 'cart',
  initialState: {
    items: [],
    status: 'idle'
  },
  reducers: {
    addToCart: (state, action) => {
      const existing = state.items.find(item => item.id === action.payload.id);
      if (existing) {
        existing.quantity += 1;  // Immer: ทำ mutation ได้
      } else {
        state.items.push({ ...action.payload, quantity: 1 });
      }
    },
    removeFromCart: (state, action) => {
      state.items = state.items.filter(item => item.id !== action.payload);
    },
    updateQuantity: (state, action) => {
      const { id, quantity } = action.payload;
      const item = state.items.find(item => item.id === id);
      if (item) {
        if (quantity <= 0) {
          state.items = state.items.filter(item => item.id !== id);
        } else {
          item.quantity = quantity;
        }
      }
    },
    clearCart: (state) => {
      state.items = [];
    }
  }
});

export const { addToCart, removeFromCart, updateQuantity, clearCart } = cartSlice.actions;
export default cartSlice.reducer;

// Selectors
export const selectCartItems = (state) => state.cart.items;
export const selectCartTotal = (state) =>
  state.cart.items.reduce((sum, item) => sum + item.price * item.quantity, 0);
export const selectCartCount = (state) =>
  state.cart.items.reduce((sum, item) => sum + item.quantity, 0);
```

---

## Step 1441: useSelector และ useDispatch

```jsx
import { useSelector, useDispatch } from 'react-redux';
import { increment, decrement, reset, setStep, selectCount, selectStep } from './store/counterSlice';

function Counter() {
  const dispatch = useDispatch();
  
  // useSelector: อ่าน state จาก store
  const count = useSelector(selectCount);
  const step = useSelector(selectStep);
  
  return (
    <div>
      <p>นับ: {count}</p>
      <p>ก้าว: {step}</p>
      
      <button onClick={() => dispatch(increment())}>+{step}</button>
      <button onClick={() => dispatch(decrement())}>-{step}</button>
      <button onClick={() => dispatch(reset())}>รีเซ็ต</button>
      
      <div>
        <label>ก้าวต่อครั้ง: </label>
        <input
          type="number"
          value={step}
          min={1}
          onChange={e => dispatch(setStep(Number(e.target.value)))}
        />
      </div>
    </div>
  );
}
```

```jsx
// Cart Component
import { useSelector, useDispatch } from 'react-redux';
import { addToCart, removeFromCart, updateQuantity, clearCart, selectCartItems, selectCartTotal, selectCartCount } from './store/cartSlice';

function ProductCard({ product }) {
  const dispatch = useDispatch();
  
  return (
    <div>
      <h3>{product.name}</h3>
      <p>฿{product.price}</p>
      <button onClick={() => dispatch(addToCart(product))}>
        เพิ่มในตะกร้า
      </button>
    </div>
  );
}

function CartSidebar() {
  const dispatch = useDispatch();
  const items = useSelector(selectCartItems);
  const total = useSelector(selectCartTotal);
  const count = useSelector(selectCartCount);
  
  return (
    <div>
      <h2>ตะกร้า ({count})</h2>
      
      {items.map(item => (
        <div key={item.id}>
          <span>{item.name}</span>
          <input
            type="number"
            value={item.quantity}
            min={0}
            onChange={e => dispatch(updateQuantity({ id: item.id, quantity: Number(e.target.value) }))}
          />
          <span>฿{(item.price * item.quantity).toLocaleString()}</span>
          <button onClick={() => dispatch(removeFromCart(item.id))}>ลบ</button>
        </div>
      ))}
      
      <hr />
      <p>รวม: ฿{total.toLocaleString()}</p>
      <button onClick={() => dispatch(clearCart())}>ล้างตะกร้า</button>
    </div>
  );
}
```

---

## Step 1442: RTK Query สำหรับ Data Fetching

```jsx
// store/api.js - สร้าง API slice ด้วย RTK Query
import { createApi, fetchBaseQuery } from '@reduxjs/toolkit/query/react';

export const apiSlice = createApi({
  reducerPath: 'api',
  baseQuery: fetchBaseQuery({
    baseUrl: 'https://jsonplaceholder.typicode.com',
    prepareHeaders: (headers, { getState }) => {
      const token = getState().auth?.token;
      if (token) {
        headers.set('authorization', `Bearer ${token}`);
      }
      return headers;
    }
  }),
  tagTypes: ['User', 'Post', 'Comment'],
  endpoints: (builder) => ({
    // GET /users
    getUsers: builder.query({
      query: () => '/users',
      providesTags: ['User']
    }),
    
    // GET /users/:id
    getUserById: builder.query({
      query: (id) => `/users/${id}`,
      providesTags: (result, error, id) => [{ type: 'User', id }]
    }),
    
    // POST /users
    createUser: builder.mutation({
      query: (newUser) => ({
        url: '/users',
        method: 'POST',
        body: newUser
      }),
      invalidatesTags: ['User']  // Refetch users หลัง create
    }),
    
    // PUT /users/:id
    updateUser: builder.mutation({
      query: ({ id, ...updates }) => ({
        url: `/users/${id}`,
        method: 'PUT',
        body: updates
      }),
      invalidatesTags: (result, error, { id }) => [{ type: 'User', id }]
    }),
    
    // DELETE /users/:id
    deleteUser: builder.mutation({
      query: (id) => ({
        url: `/users/${id}`,
        method: 'DELETE'
      }),
      invalidatesTags: ['User']
    }),
    
    // GET /posts?userId=:userId
    getPostsByUser: builder.query({
      query: (userId) => `/posts?userId=${userId}`,
      providesTags: (result, error, userId) =>
        result
          ? [...result.map(({ id }) => ({ type: 'Post', id })), { type: 'Post', list: userId }]
          : [{ type: 'Post', list: userId }]
    })
  })
});

// Export auto-generated hooks
export const {
  useGetUsersQuery,
  useGetUserByIdQuery,
  useCreateUserMutation,
  useUpdateUserMutation,
  useDeleteUserMutation,
  useGetPostsByUserQuery
} = apiSlice;
```

```jsx
// store/index.js - เพิ่ม API reducer
import { configureStore } from '@reduxjs/toolkit';
import { apiSlice } from './api';

export const store = configureStore({
  reducer: {
    [apiSlice.reducerPath]: apiSlice.reducer,
    // ... other reducers
  },
  middleware: (getDefaultMiddleware) =>
    getDefaultMiddleware().concat(apiSlice.middleware)
});
```

```jsx
// ใช้งาน RTK Query hooks
function UserList() {
  const {
    data: users,
    isLoading,
    isError,
    error,
    refetch
  } = useGetUsersQuery();
  
  if (isLoading) return <p>กำลังโหลด...</p>;
  if (isError) return <p>ข้อผิดพลาด: {error.message}</p>;
  
  return (
    <div>
      <button onClick={refetch}>โหลดใหม่</button>
      {users?.map(user => (
        <UserCard key={user.id} user={user} />
      ))}
    </div>
  );
}

function UserDetail({ userId }) {
  const { data: user, isLoading } = useGetUserByIdQuery(userId, {
    skip: !userId  // skip ถ้าไม่มี userId
  });
  
  return isLoading ? <p>โหลด...</p> : <div>{user?.name}</div>;
}

function CreateUserForm() {
  const [createUser, { isLoading }] = useCreateUserMutation();
  const [name, setName] = useState('');
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    try {
      await createUser({ name }).unwrap();
      setName('');
      alert('สร้างผู้ใช้สำเร็จ!');
    } catch (err) {
      alert('เกิดข้อผิดพลาด: ' + err.message);
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input value={name} onChange={e => setName(e.target.value)} />
      <button disabled={isLoading}>สร้าง</button>
    </form>
  );
}
```

---

## Step 1443: Immer สำหรับ Immutable Updates

```jsx
import produce from 'immer';
// npm install immer

// ปัญหา Immutable Updates แบบเดิม
const state = {
  user: {
    name: 'สมชาย',
    address: {
      city: 'กรุงเทพ',
      zip: '10100'
    }
  },
  posts: [
    { id: 1, title: 'โพสต์แรก', likes: 5 }
  ]
};

// ❌ แบบเดิม: verbose และเกิดข้อผิดพลาดง่าย
const newState = {
  ...state,
  user: {
    ...state.user,
    address: {
      ...state.user.address,
      city: 'เชียงใหม่'
    }
  }
};

// ✅ ด้วย Immer: เขียน mutation เหมือนปกติ
const newState2 = produce(state, (draft) => {
  draft.user.address.city = 'เชียงใหม่';
});

// Immer กับ Array operations
const updatedState = produce(state, (draft) => {
  // เพิ่มโพสต์
  draft.posts.push({ id: 2, title: 'โพสต์ใหม่', likes: 0 });
  
  // อัพเดทโพสต์
  const post = draft.posts.find(p => p.id === 1);
  if (post) post.likes += 1;
});
```

```jsx
// RTK ใช้ Immer โดยอัตโนมัติ ใน createSlice
const postsSlice = createSlice({
  name: 'posts',
  initialState: {
    items: [],
    status: 'idle'
  },
  reducers: {
    addPost: (state, action) => {
      // เขียนแบบ mutation ได้เลย (Immer จัดการให้)
      state.items.push(action.payload);
    },
    likePost: (state, action) => {
      const post = state.items.find(p => p.id === action.payload);
      if (post) post.likes += 1;
    },
    updatePost: (state, action) => {
      const { id, ...changes } = action.payload;
      const post = state.items.find(p => p.id === id);
      if (post) Object.assign(post, changes);
    }
  }
});
```

---

## Step 1444: Async Thunks ใน Redux Toolkit

```jsx
import { createSlice, createAsyncThunk } from '@reduxjs/toolkit';

// createAsyncThunk สร้าง async action
export const fetchUsers = createAsyncThunk(
  'users/fetchAll',  // action type prefix
  async (_, { rejectWithValue }) => {
    try {
      const response = await fetch('https://jsonplaceholder.typicode.com/users');
      if (!response.ok) throw new Error('Fetch failed');
      return await response.json();
    } catch (error) {
      return rejectWithValue(error.message);
    }
  }
);

export const createUser = createAsyncThunk(
  'users/create',
  async (userData, { rejectWithValue }) => {
    try {
      const response = await fetch('https://jsonplaceholder.typicode.com/users', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(userData)
      });
      return await response.json();
    } catch (error) {
      return rejectWithValue(error.message);
    }
  }
);

// Slice ที่จัดการ async states
const usersSlice = createSlice({
  name: 'users',
  initialState: {
    items: [],
    status: 'idle',  // 'idle' | 'loading' | 'succeeded' | 'failed'
    error: null,
    createStatus: 'idle'
  },
  reducers: {
    clearError: (state) => { state.error = null; }
  },
  extraReducers: (builder) => {
    // fetchUsers
    builder
      .addCase(fetchUsers.pending, (state) => {
        state.status = 'loading';
      })
      .addCase(fetchUsers.fulfilled, (state, action) => {
        state.status = 'succeeded';
        state.items = action.payload;
      })
      .addCase(fetchUsers.rejected, (state, action) => {
        state.status = 'failed';
        state.error = action.payload;
      });
    
    // createUser
    builder
      .addCase(createUser.pending, (state) => {
        state.createStatus = 'loading';
      })
      .addCase(createUser.fulfilled, (state, action) => {
        state.createStatus = 'succeeded';
        state.items.push(action.payload);
      })
      .addCase(createUser.rejected, (state, action) => {
        state.createStatus = 'failed';
        state.error = action.payload;
      });
  }
});

export default usersSlice.reducer;
```

```jsx
// ใช้งาน async thunks
function UserManager() {
  const dispatch = useDispatch();
  const { items: users, status, error } = useSelector(state => state.users);
  const [newUserName, setNewUserName] = useState('');
  
  useEffect(() => {
    if (status === 'idle') {
      dispatch(fetchUsers());
    }
  }, [status, dispatch]);
  
  const handleCreate = async () => {
    if (!newUserName.trim()) return;
    try {
      await dispatch(createUser({ name: newUserName })).unwrap();
      setNewUserName('');
      alert('สร้างผู้ใช้สำเร็จ!');
    } catch (err) {
      alert('เกิดข้อผิดพลาด: ' + err);
    }
  };
  
  if (status === 'loading') return <p>กำลังโหลด...</p>;
  if (status === 'failed') return <p>Error: {error}</p>;
  
  return (
    <div>
      <div>
        <input value={newUserName} onChange={e => setNewUserName(e.target.value)} />
        <button onClick={handleCreate}>สร้างผู้ใช้</button>
      </div>
      {users.map(user => <div key={user.id}>{user.name}</div>)}
    </div>
  );
}
```

---

## Step 1445: Jotai Overview

```jsx
// npm install jotai

import { atom, useAtom, useAtomValue, useSetAtom } from 'jotai';

// Atoms - unit ของ state
const countAtom = atom(0);
const nameAtom = atom('สมชาย');
const darkModeAtom = atom(false);

// Derived atoms (computed)
const doubleCountAtom = atom((get) => get(countAtom) * 2);

const userAtom = atom({ name: '', email: '' });

// Write atom
const incrementAtom = atom(
  null,  // no read
  (get, set, amount = 1) => {
    set(countAtom, get(countAtom) + amount);
  }
);

// Async atom
const userDataAtom = atom(async () => {
  const response = await fetch('https://jsonplaceholder.typicode.com/users/1');
  return response.json();
});

// ใช้งาน
function Counter() {
  const [count, setCount] = useAtom(countAtom);
  const double = useAtomValue(doubleCountAtom);
  
  return (
    <div>
      <p>count: {count}</p>
      <p>double: {double}</p>
      <button onClick={() => setCount(c => c + 1)}>+</button>
    </div>
  );
}

// useAtomValue: อ่านอย่างเดียว (ไม่ trigger re-render จาก setter)
function CountDisplay() {
  const count = useAtomValue(countAtom);
  return <span>{count}</span>;
}

// useSetAtom: set อย่างเดียว (ไม่ subscribe ค่า)
function IncrementButton() {
  const setCount = useSetAtom(countAtom);
  return <button onClick={() => setCount(c => c + 1)}>+</button>;
}
```

---

## Step 1446: Recoil Overview

```jsx
// npm install recoil

import { RecoilRoot, atom, selector, useRecoilState, useRecoilValue, useSetRecoilState } from 'recoil';

// Atoms
const textState = atom({
  key: 'textState',  // unique key
  default: ''
});

const charCountState = atom({
  key: 'charCountState',
  default: 0
});

// Selectors (derived state)
const charCountSelector = selector({
  key: 'charCountSelector',
  get: ({ get }) => {
    const text = get(textState);
    return text.length;
  }
});

// Async Selector
const userInfoSelector = selector({
  key: 'userInfo',
  get: async ({ get }) => {
    const userId = get(selectedUserIdState);
    const response = await fetch(`/api/users/${userId}`);
    return response.json();
  }
});

// Provider
function App() {
  return (
    <RecoilRoot>
      <TextInput />
      <CharacterCount />
    </RecoilRoot>
  );
}

function TextInput() {
  const [text, setText] = useRecoilState(textState);
  return (
    <input value={text} onChange={e => setText(e.target.value)} />
  );
}

function CharacterCount() {
  const count = useRecoilValue(charCountSelector);
  return <p>จำนวนตัวอักษร: {count}</p>;
}
```

---

## Step 1447: Comparing State Management Solutions

```
เปรียบเทียบ Solutions:

┌────────────────┬───────────┬───────────┬───────────┬──────────┐
│ Feature        │ Context   │ Zustand   │ Redux/RTK │  Jotai   │
├────────────────┼───────────┼───────────┼───────────┼──────────┤
│ Bundle Size    │  0 KB*    │  ~3 KB    │  ~15 KB   │  ~3 KB   │
│ Learning Curve │  Low      │  Low      │  Medium   │  Low     │
│ Boilerplate    │  Medium   │  Minimal  │  Moderate │  Minimal │
│ DevTools       │  Limited  │  Yes      │  Excellent│  Yes     │
│ TypeScript     │  Good     │  Excellent│  Excellent│  Good    │
│ Server Side    │  Good     │  Good     │  Good     │  Good    │
│ Scale          │  Medium   │  High     │  High     │  High    │
│ Async          │  Manual   │  Built-in │  RTK Query│  Built-in│
└────────────────┴───────────┴───────────┴───────────┴──────────┘
* Context ไม่มี bundle เพิ่ม เพราะเป็นส่วนของ React
```

---

## Step 1448: เมื่อไหรควรใช้อะไร

```jsx
// 1. Context API: ใช้เมื่อ
// - State ที่เปลี่ยนไม่บ่อย (theme, auth, i18n)
// - ไม่ต้องการ external dependency
// - Team เริ่มต้นกับ React

// ✅ เหมาะสมกับ Context
const ThemeContext = createContext('light');
const LanguageContext = createContext('th');
const AuthContext = createContext(null);

// 2. Zustand: ใช้เมื่อ
// - ต้องการ simplicity + power
// - State ที่ component หลายอันใช้
// - ไม่ต้องการ Redux complexity
// - โปรเจคขนาดกลาง

// ✅ เหมาะสมกับ Zustand
const useCartStore = create((set) => ({ items: [], addItem: (item) => set(s => ({ items: [...s.items, item] })) }));
const useUIStore = create((set) => ({ sidebar: false, toggleSidebar: () => set(s => ({ sidebar: !s.sidebar })) }));

// 3. Redux Toolkit: ใช้เมื่อ
// - โปรเจคขนาดใหญ่, หลาย developer
// - ต้องการ powerful DevTools
// - มี complex state logic
// - ต้องการ RTK Query

// ✅ เหมาะสมกับ Redux
// - Enterprise applications
// - Complex data flows
// - Strict team conventions needed

// 4. Jotai/Recoil: ใช้เมื่อ
// - ต้องการ fine-grained reactivity
// - Atomic state management
// - Complex computed state
```

---

## Step 1449: Best Practices สำหรับ State Management

```jsx
// 1. โครงสร้าง store ที่ดี
// store/
//   index.js          - configure store
//   features/
//     auth/
//       authSlice.js
//       authSelectors.js
//       authThunks.js
//     products/
//       productsSlice.js
//       productsSelectors.js
//     cart/
//       cartSlice.js
//   api/
//     apiSlice.js

// 2. Normalize state สำหรับ data ที่ reference กัน
// ❌ Nested
const state = {
  posts: [
    {
      id: 1,
      title: 'โพสต์',
      author: { id: 1, name: 'สมชาย' },
      comments: [{ id: 1, text: 'ดีมาก' }]
    }
  ]
};

// ✅ Normalized (อัพเดทง่ายกว่า)
const normalizedState = {
  posts: {
    byId: { 1: { id: 1, title: 'โพสต์', authorId: 1, commentIds: [1] } },
    allIds: [1]
  },
  users: {
    byId: { 1: { id: 1, name: 'สมชาย' } },
    allIds: [1]
  },
  comments: {
    byId: { 1: { id: 1, text: 'ดีมาก' } },
    allIds: [1]
  }
};
```

```jsx
// 3. TypeScript กับ Redux
import { TypedUseSelectorHook, useDispatch, useSelector } from 'react-redux';

// Typed hooks
type RootState = ReturnType<typeof store.getState>;
type AppDispatch = typeof store.dispatch;

export const useAppDispatch = () => useDispatch<AppDispatch>();
export const useAppSelector: TypedUseSelectorHook<RootState> = useSelector;

// ใช้งาน
function TypedCounter() {
  const dispatch = useAppDispatch();
  const count = useAppSelector((state) => state.counter.value);
  // count มี type number โดยอัตโนมัติ
}
```

---

## Step 1450: Complete Example - Full App State

```jsx
// ตัวอย่าง app ที่ใช้ state management ครบถ้วน

// 1. Local State (useState) - สำหรับ UI state
function SearchBar() {
  const [query, setQuery] = useState('');
  const [isOpen, setIsOpen] = useState(false);
  
  return <input value={query} onChange={e => setQuery(e.target.value)} />;
}

// 2. Context - สำหรับ app-wide settings
const AppContext = createContext(null);
function AppProvider({ children }) {
  const [theme, setTheme] = useState('light');
  const [language, setLanguage] = useState('th');
  return (
    <AppContext.Provider value={{ theme, setTheme, language, setLanguage }}>
      {children}
    </AppContext.Provider>
  );
}

// 3. Zustand - สำหรับ feature state
const useProductStore = create((set, get) => ({
  products: [],
  filters: { category: 'all', priceRange: [0, 10000] },
  
  fetchProducts: async () => {
    const data = await fetch('/api/products').then(r => r.json());
    set({ products: data });
  },
  
  setFilter: (key, value) => set(state => ({
    filters: { ...state.filters, [key]: value }
  })),
  
  get filteredProducts() {
    const { products, filters } = get();
    return products.filter(p => {
      if (filters.category !== 'all' && p.category !== filters.category) return false;
      if (p.price < filters.priceRange[0] || p.price > filters.priceRange[1]) return false;
      return true;
    });
  }
}));

// 4. RTK Query - สำหรับ server state
const { useGetProductsQuery, useCreateOrderMutation } = apiSlice;

function ProductPage() {
  const { theme } = useContext(AppContext);
  const [searchQuery, setSearchQuery] = useState('');
  const { filteredProducts, setFilter } = useProductStore();
  const { data: featuredProducts } = useGetProductsQuery({ featured: true });
  const [createOrder] = useCreateOrderMutation();
  
  return (
    <div className={`app ${theme}`}>
      <input value={searchQuery} onChange={e => setSearchQuery(e.target.value)} />
      <select onChange={e => setFilter('category', e.target.value)}>
        <option value="all">ทั้งหมด</option>
        <option value="electronics">อิเล็กทรอนิกส์</option>
      </select>
      {filteredProducts.map(product => (
        <div key={product.id} onClick={() => createOrder({ productId: product.id })}>
          {product.name}
        </div>
      ))}
    </div>
  );
}
```

---

## แบบฝึกหัด (Exercises)

### แบบฝึกหัดที่ 1: Context Auth System
สร้าง auth system ด้วย Context API ที่มี:
- Login/Logout
- Protected routes
- User profile update
- Token persistence ใน localStorage

### แบบฝึกหัดที่ 2: Zustand Shopping App
สร้าง shopping app ด้วย Zustand ที่มี:
- Product list (fetch จาก API)
- Cart management
- Wishlist
- Order history
- localStorage persistence

### แบบฝึกหัดที่ 3: Redux Todo App
สร้าง todo app ด้วย Redux Toolkit ที่มี:
- CRUD todos
- Filter by status
- Categories
- Async fetch จาก API
- RTK Query integration

### แบบฝึกหัดที่ 4: เปรียบเทียบ Solutions
ทำ task เดิม (counter + todo) ด้วย Context, Zustand, Redux แล้วเปรียบเทียบ:
- Lines of code
- Performance (React DevTools Profiler)
- Developer experience

### แบบฝึกหัดที่ 5: Real-world App
สร้าง mini social media app:
- Authentication (Context หรือ RTK Query)
- Posts feed (RTK Query)
- Like/Comment (Zustand หรือ Redux)
- Real-time notifications (WebSocket + Zustand)

---

## สรุปท้ายส่วน

ใน Part 73 นี้เราได้เรียนรู้:

- **ปัญหาของ State Management**: prop drilling, state sync
- **Context API**: ดีสำหรับ state ที่ไม่ค่อยเปลี่ยน, ข้อจำกัดคือ re-render ทั้งหมด
- **Zustand**: ง่าย, lightweight, ไม่ต้องมี Provider
- **Redux Toolkit**: มาตรฐาน, เหมาะกับโปรเจคใหญ่, devtools เยี่ยม
- **RTK Query**: data fetching + caching อัตโนมัติ
- **Immer**: เขียน immutable updates แบบ mutation
- **Jotai/Recoil**: atomic state management
- **เมื่อไหรควรใช้อะไร**: ขึ้นอยู่กับขนาดและความซับซ้อนของโปรเจค

Part 74 จะเรียน Vue.js พื้นฐาน!
