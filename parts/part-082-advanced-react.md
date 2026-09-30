# Part 82: Advanced React Patterns (Pattern ขั้นสูงของ React)
## Steps 1611-1630

React มี patterns หลากหลายที่ช่วยให้เราเขียน components ที่ reusable, flexible, และ maintainable
ในบทนี้เราจะเรียนรู้ patterns ระดับ advanced ที่ใช้ใน production apps จริงๆ

---

## Step 1611: Compound Components Pattern

Compound Components Pattern ช่วยให้ components หลายอันทำงานร่วมกันโดยแชร์ state ผ่าน Context

```tsx
import React, { createContext, useContext, useState } from "react";

// Compound Components - ตัวอย่าง Accordion
interface AccordionContextType {
  activeItem: string | null;
  setActiveItem: (id: string | null) => void;
}

const AccordionContext = createContext<AccordionContextType | null>(null);

function useAccordionContext() {
  const context = useContext(AccordionContext);
  if (!context) {
    throw new Error("useAccordionContext ต้องใช้ภายใน Accordion");
  }
  return context;
}

// Parent component
function Accordion({ children }: { children: React.ReactNode }) {
  const [activeItem, setActiveItem] = useState<string | null>(null);
  
  return (
    <AccordionContext.Provider value={{ activeItem, setActiveItem }}>
      <div className="accordion">{children}</div>
    </AccordionContext.Provider>
  );
}

// Sub-components
function AccordionItem({
  id,
  children,
}: {
  id: string;
  children: React.ReactNode;
}) {
  const { activeItem, setActiveItem } = useAccordionContext();
  const isOpen = activeItem === id;

  return (
    <div className={`accordion-item ${isOpen ? "open" : ""}`}>
      {React.Children.map(children, child => {
        if (React.isValidElement(child)) {
          return React.cloneElement(child as React.ReactElement<any>, {
            isOpen,
            onToggle: () => setActiveItem(isOpen ? null : id)
          });
        }
        return child;
      })}
    </div>
  );
}

function AccordionHeader({
  children,
  isOpen,
  onToggle,
}: {
  children: React.ReactNode;
  isOpen?: boolean;
  onToggle?: () => void;
}) {
  return (
    <button
      className="accordion-header"
      onClick={onToggle}
      aria-expanded={isOpen}
    >
      {children}
      <span>{isOpen ? "▲" : "▼"}</span>
    </button>
  );
}

function AccordionBody({
  children,
  isOpen,
}: {
  children: React.ReactNode;
  isOpen?: boolean;
}) {
  if (!isOpen) return null;
  return <div className="accordion-body">{children}</div>;
}

// ผูก sub-components เข้ากับ parent
Accordion.Item = AccordionItem;
Accordion.Header = AccordionHeader;
Accordion.Body = AccordionBody;

// การใช้งาน
function App() {
  return (
    <Accordion>
      <Accordion.Item id="1">
        <Accordion.Header>ส่วนที่ 1</Accordion.Header>
        <Accordion.Body>เนื้อหาส่วนที่ 1</Accordion.Body>
      </Accordion.Item>
      <Accordion.Item id="2">
        <Accordion.Header>ส่วนที่ 2</Accordion.Header>
        <Accordion.Body>เนื้อหาส่วนที่ 2</Accordion.Body>
      </Accordion.Item>
    </Accordion>
  );
}
```

---

## Step 1612: Render Props Pattern

Render Props ส่ง function เป็น prop เพื่อให้ parent กำหนด UI ได้เอง

```tsx
import React, { useState, useEffect, useCallback } from "react";

// Mouse position render prop
interface MousePosition {
  x: number;
  y: number;
}

interface MouseTrackerProps {
  render: (position: MousePosition) => React.ReactNode;
}

function MouseTracker({ render }: MouseTrackerProps) {
  const [position, setPosition] = useState<MousePosition>({ x: 0, y: 0 });

  useEffect(() => {
    const handleMouseMove = (e: MouseEvent) => {
      setPosition({ x: e.clientX, y: e.clientY });
    };

    window.addEventListener("mousemove", handleMouseMove);
    return () => window.removeEventListener("mousemove", handleMouseMove);
  }, []);

  return <>{render(position)}</>;
}

// การใช้งาน
function App() {
  return (
    <MouseTracker
      render={({ x, y }) => (
        <div>
          <h2>ตำแหน่งเมาส์: ({x}, {y})</h2>
          <div
            style={{
              position: "fixed",
              left: x,
              top: y,
              width: 20,
              height: 20,
              background: "red",
              borderRadius: "50%",
              transform: "translate(-50%, -50%)",
              pointerEvents: "none"
            }}
          />
        </div>
      )}
    />
  );
}

// Data fetcher render prop
interface FetchResult<T> {
  data: T | null;
  loading: boolean;
  error: Error | null;
  refetch: () => void;
}

interface DataFetcherProps<T> {
  url: string;
  render: (result: FetchResult<T>) => React.ReactNode;
}

function DataFetcher<T>({ url, render }: DataFetcherProps<T>) {
  const [state, setState] = useState<Omit<FetchResult<T>, "refetch">>({
    data: null,
    loading: true,
    error: null
  });

  const fetchData = useCallback(async () => {
    setState(prev => ({ ...prev, loading: true, error: null }));
    try {
      const response = await fetch(url);
      if (!response.ok) throw new Error(`HTTP ${response.status}`);
      const data = await response.json();
      setState({ data, loading: false, error: null });
    } catch (error) {
      setState({ data: null, loading: false, error: error as Error });
    }
  }, [url]);

  useEffect(() => {
    fetchData();
  }, [fetchData]);

  return <>{render({ ...state, refetch: fetchData })}</>;
}

// ใช้งาน
interface User {
  id: number;
  name: string;
  email: string;
}

function UserList() {
  return (
    <DataFetcher<User[]>
      url="https://jsonplaceholder.typicode.com/users"
      render={({ data, loading, error, refetch }) => {
        if (loading) return <div>กำลังโหลด...</div>;
        if (error) return <div>เกิดข้อผิดพลาด: {error.message} <button onClick={refetch}>ลองใหม่</button></div>;
        if (!data) return null;
        
        return (
          <ul>
            {data.map(user => (
              <li key={user.id}>{user.name} - {user.email}</li>
            ))}
          </ul>
        );
      }}
    />
  );
}
```

---

## Step 1613: Higher-Order Components (HOC) Pattern

HOC คือ function ที่รับ component และคืน component ใหม่ที่มีความสามารถเพิ่มขึ้น

```tsx
import React, { ComponentType, useEffect, useRef } from "react";

// withLogging HOC - เพิ่ม logging ให้ component
function withLogging<P extends object>(
  WrappedComponent: ComponentType<P>,
  componentName?: string
): ComponentType<P> {
  const name = componentName || WrappedComponent.displayName || WrappedComponent.name;
  
  function WithLogging(props: P) {
    const renderCount = useRef(0);
    renderCount.current += 1;
    
    useEffect(() => {
      console.log(`${name} mounted`);
      return () => console.log(`${name} unmounted`);
    }, []);
    
    useEffect(() => {
      console.log(`${name} rendered (ครั้งที่ ${renderCount.current})`);
    });
    
    return <WrappedComponent {...props} />;
  }
  
  WithLogging.displayName = `withLogging(${name})`;
  return WithLogging;
}

// withAuth HOC - ตรวจสอบ authentication
interface AuthProps {
  isAuthenticated: boolean;
  user: { name: string; role: string } | null;
}

function withAuth<P extends object>(
  WrappedComponent: ComponentType<P & AuthProps>,
  requiredRole?: string
): ComponentType<Omit<P, keyof AuthProps>> {
  function WithAuth(props: Omit<P, keyof AuthProps>) {
    // สมมติดึงจาก auth store
    const isAuthenticated = true;
    const user = { name: "สมชาย", role: "admin" };

    if (!isAuthenticated) {
      return <div>กรุณาเข้าสู่ระบบ</div>;
    }

    if (requiredRole && user.role !== requiredRole) {
      return <div>คุณไม่มีสิทธิ์เข้าถึงหน้านี้</div>;
    }

    return (
      <WrappedComponent
        {...(props as P)}
        isAuthenticated={isAuthenticated}
        user={user}
      />
    );
  }
  
  WithAuth.displayName = `withAuth(${WrappedComponent.displayName || WrappedComponent.name})`;
  return WithAuth;
}

// withErrorBoundary HOC
interface ErrorBoundaryState {
  hasError: boolean;
  error: Error | null;
}

function withErrorBoundary<P extends object>(
  WrappedComponent: ComponentType<P>,
  FallbackComponent?: ComponentType<{ error: Error; reset: () => void }>
): ComponentType<P> {
  class ErrorBoundary extends React.Component<
    P,
    ErrorBoundaryState
  > {
    state: ErrorBoundaryState = { hasError: false, error: null };

    static getDerivedStateFromError(error: Error): ErrorBoundaryState {
      return { hasError: true, error };
    }

    componentDidCatch(error: Error, errorInfo: React.ErrorInfo) {
      console.error("Error caught:", error, errorInfo);
    }

    reset = () => {
      this.setState({ hasError: false, error: null });
    };

    render() {
      if (this.state.hasError && this.state.error) {
        if (FallbackComponent) {
          return <FallbackComponent error={this.state.error} reset={this.reset} />;
        }
        return (
          <div>
            <h2>เกิดข้อผิดพลาด!</h2>
            <p>{this.state.error.message}</p>
            <button onClick={this.reset}>ลองใหม่</button>
          </div>
        );
      }
      return <WrappedComponent {...this.props} />;
    }
  }
  
  return ErrorBoundary;
}

// การใช้งาน
interface DashboardProps extends AuthProps {
  title: string;
}

function Dashboard({ title, user }: DashboardProps) {
  return <h1>{title} - {user?.name}</h1>;
}

const EnhancedDashboard = withErrorBoundary(
  withLogging(
    withAuth(Dashboard, "admin")
  )
);
```

---

## Step 1614: Control Props Pattern

Control Props ให้ผู้ใช้ควบคุม state ของ component จากภายนอก

```tsx
import React, { useState, useCallback } from "react";

interface SwitchProps {
  // Controlled props
  checked?: boolean;
  onChange?: (checked: boolean) => void;
  
  // Uncontrolled default
  defaultChecked?: boolean;
  
  // Other props
  label?: string;
  disabled?: boolean;
}

function Switch({
  checked: controlledChecked,
  onChange,
  defaultChecked = false,
  label,
  disabled = false
}: SwitchProps) {
  // Internal state สำหรับ uncontrolled mode
  const [internalChecked, setInternalChecked] = useState(defaultChecked);
  
  // ถ้ามี checked prop ก็เป็น controlled, ไม่งั้นเป็น uncontrolled
  const isControlled = controlledChecked !== undefined;
  const checked = isControlled ? controlledChecked : internalChecked;
  
  const handleToggle = useCallback(() => {
    if (disabled) return;
    
    if (isControlled) {
      onChange?.(!checked);
    } else {
      setInternalChecked(prev => !prev);
      onChange?.(!checked);
    }
  }, [disabled, isControlled, onChange, checked]);

  return (
    <div className="switch-container">
      <button
        role="switch"
        aria-checked={checked}
        aria-label={label}
        disabled={disabled}
        onClick={handleToggle}
        className={`switch ${checked ? "switch--on" : "switch--off"} ${disabled ? "switch--disabled" : ""}`}
      >
        <span className="switch-thumb" />
      </button>
      {label && <span className="switch-label">{label}</span>}
    </div>
  );
}

// การใช้งานแบบ Uncontrolled
function UncontrolledExample() {
  return (
    <Switch
      defaultChecked={true}
      label="เปิดใช้งาน"
      onChange={(checked) => console.log("เปลี่ยนเป็น:", checked)}
    />
  );
}

// การใช้งานแบบ Controlled
function ControlledExample() {
  const [enabled, setEnabled] = useState(false);
  
  return (
    <div>
      <Switch
        checked={enabled}
        onChange={setEnabled}
        label="Dark Mode"
      />
      <p>สถานะ: {enabled ? "เปิด" : "ปิด"}</p>
    </div>
  );
}

// useControllableState hook - reusable pattern
function useControllableState<T>({
  controlled,
  defaultValue,
  onChange
}: {
  controlled?: T;
  defaultValue: T;
  onChange?: (value: T) => void;
}): [T, (value: T) => void] {
  const [internalState, setInternalState] = useState(defaultValue);
  const isControlled = controlled !== undefined;
  const state = isControlled ? controlled : internalState;

  const setState = useCallback((newValue: T) => {
    if (!isControlled) {
      setInternalState(newValue);
    }
    onChange?.(newValue);
  }, [isControlled, onChange]);

  return [state, setState];
}
```

---

## Step 1615: Props Getters Pattern

Props Getters ให้ flexibility ในการกำหนด props ของ elements

```tsx
import React, { useState, useCallback, MouseEvent, KeyboardEvent } from "react";

interface UseToggleOptions {
  initialValue?: boolean;
  onChange?: (value: boolean) => void;
}

interface ToggleResult {
  on: boolean;
  toggle: () => void;
  getTogglerProps: (props?: React.ButtonHTMLAttributes<HTMLButtonElement>) => React.ButtonHTMLAttributes<HTMLButtonElement>;
  getContainerProps: (props?: React.HTMLAttributes<HTMLDivElement>) => React.HTMLAttributes<HTMLDivElement>;
}

function useToggle({ initialValue = false, onChange }: UseToggleOptions = {}): ToggleResult {
  const [on, setOn] = useState(initialValue);

  const toggle = useCallback(() => {
    const newValue = !on;
    setOn(newValue);
    onChange?.(newValue);
  }, [on, onChange]);

  const getTogglerProps = useCallback(
    (
      props: React.ButtonHTMLAttributes<HTMLButtonElement> = {}
    ): React.ButtonHTMLAttributes<HTMLButtonElement> => ({
      "aria-pressed": on,
      onClick(event: MouseEvent<HTMLButtonElement>) {
        props.onClick?.(event);
        if (!event.defaultPrevented) {
          toggle();
        }
      },
      onKeyDown(event: KeyboardEvent<HTMLButtonElement>) {
        props.onKeyDown?.(event);
        if (!event.defaultPrevented && (event.key === " " || event.key === "Enter")) {
          event.preventDefault();
          toggle();
        }
      },
      ...props
    }),
    [on, toggle]
  );

  const getContainerProps = useCallback(
    (
      props: React.HTMLAttributes<HTMLDivElement> = {}
    ): React.HTMLAttributes<HTMLDivElement> => ({
      role: "group",
      ...props
    }),
    []
  );

  return { on, toggle, getTogglerProps, getContainerProps };
}

// การใช้งาน
function ToggleExample() {
  const { on, getTogglerProps, getContainerProps } = useToggle({
    initialValue: false,
    onChange: (value) => console.log("toggled:", value)
  });

  return (
    <div {...getContainerProps({ className: "toggle-group" })}>
      <button
        {...getTogglerProps({
          className: `btn ${on ? "btn-primary" : "btn-secondary"}`,
          style: { borderRadius: 20 }
        })}
      >
        {on ? "เปิด" : "ปิด"}
      </button>
      <p>สถานะ: {on ? "✓ เปิดใช้งาน" : "✗ ปิดใช้งาน"}</p>
    </div>
  );
}
```

---

## Step 1616: State Reducer Pattern

State Reducer ให้ผู้ใช้ควบคุม state transitions ได้เอง

```tsx
import React, { useReducer, useCallback } from "react";

// Counter ที่ใช้ State Reducer pattern
interface CounterState {
  count: number;
  isDisabled: boolean;
}

type CounterAction =
  | { type: "increment"; step?: number }
  | { type: "decrement"; step?: number }
  | { type: "reset" }
  | { type: "disable" }
  | { type: "enable" };

function defaultCounterReducer(
  state: CounterState,
  action: CounterAction
): CounterState {
  switch (action.type) {
    case "increment":
      return { ...state, count: state.count + (action.step ?? 1) };
    case "decrement":
      return { ...state, count: state.count - (action.step ?? 1) };
    case "reset":
      return { ...state, count: 0 };
    case "disable":
      return { ...state, isDisabled: true };
    case "enable":
      return { ...state, isDisabled: false };
    default:
      return state;
  }
}

interface UseCounterOptions {
  initialCount?: number;
  min?: number;
  max?: number;
  stateReducer?: (state: CounterState, action: CounterAction, defaultReducer: typeof defaultCounterReducer) => CounterState;
}

function useCounter({
  initialCount = 0,
  min = -Infinity,
  max = Infinity,
  stateReducer = defaultCounterReducer
}: UseCounterOptions = {}) {
  const reducer = useCallback(
    (state: CounterState, action: CounterAction): CounterState => {
      const newState = stateReducer(state, action, defaultCounterReducer);
      
      // Clamp count between min and max
      return {
        ...newState,
        count: Math.min(Math.max(newState.count, min), max)
      };
    },
    [stateReducer, min, max]
  );

  const [state, dispatch] = useReducer(reducer, {
    count: initialCount,
    isDisabled: false
  });

  const increment = useCallback((step?: number) => dispatch({ type: "increment", step }), []);
  const decrement = useCallback((step?: number) => dispatch({ type: "decrement", step }), []);
  const reset = useCallback(() => dispatch({ type: "reset" }), []);

  return { ...state, increment, decrement, reset };
}

// การใช้งานพื้นฐาน
function BasicCounter() {
  const { count, increment, decrement, reset } = useCounter({
    initialCount: 0,
    min: 0,
    max: 10
  });

  return (
    <div>
      <button onClick={() => decrement()}>-</button>
      <span>{count}</span>
      <button onClick={() => increment()}>+</button>
      <button onClick={reset}>รีเซ็ต</button>
    </div>
  );
}

// การใช้งานพร้อม custom state reducer
function CustomCounter() {
  const { count, increment, decrement, reset } = useCounter({
    initialCount: 0,
    // Custom reducer ที่บล็อคเลขคู่
    stateReducer(state, action, defaultReducer) {
      const newState = defaultReducer(state, action);
      
      // ถ้าเลขใหม่เป็นเลขคู่ ให้ข้ามไปอีก 1
      if (newState.count % 2 === 0 && action.type === "increment") {
        return { ...newState, count: newState.count + 1 };
      }
      
      return newState;
    }
  });

  return (
    <div>
      <button onClick={() => decrement()}>-</button>
      <span>{count} (เฉพาะเลขคี่)</span>
      <button onClick={() => increment()}>+</button>
    </div>
  );
}
```

---

## Step 1617: Provider Pattern

```tsx
import React, { createContext, useContext, useState, useReducer } from "react";

// Multi-context provider pattern
interface ThemeContextType {
  theme: "light" | "dark";
  toggleTheme: () => void;
}

interface LanguageContextType {
  language: "th" | "en";
  setLanguage: (lang: "th" | "en") => void;
  t: (key: string) => string;
}

interface UserContextType {
  user: { id: string; name: string; role: string } | null;
  login: (credentials: { username: string; password: string }) => Promise<void>;
  logout: () => void;
}

const ThemeContext = createContext<ThemeContextType | undefined>(undefined);
const LanguageContext = createContext<LanguageContextType | undefined>(undefined);
const UserContext = createContext<UserContextType | undefined>(undefined);

// Custom hooks
function useTheme() {
  const ctx = useContext(ThemeContext);
  if (!ctx) throw new Error("useTheme ต้องอยู่ใน ThemeProvider");
  return ctx;
}

function useLanguage() {
  const ctx = useContext(LanguageContext);
  if (!ctx) throw new Error("useLanguage ต้องอยู่ใน LanguageProvider");
  return ctx;
}

function useUser() {
  const ctx = useContext(UserContext);
  if (!ctx) throw new Error("useUser ต้องอยู่ใน UserProvider");
  return ctx;
}

// Providers
const translations = {
  th: { greeting: "สวัสดี", logout: "ออกจากระบบ", settings: "การตั้งค่า" },
  en: { greeting: "Hello", logout: "Logout", settings: "Settings" }
};

function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState<"light" | "dark">("light");
  const toggleTheme = () => setTheme(t => t === "light" ? "dark" : "light");

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      <div data-theme={theme}>{children}</div>
    </ThemeContext.Provider>
  );
}

function LanguageProvider({ children }: { children: React.ReactNode }) {
  const [language, setLanguage] = useState<"th" | "en">("th");
  
  const t = (key: string): string => {
    return (translations[language] as any)[key] || key;
  };

  return (
    <LanguageContext.Provider value={{ language, setLanguage, t }}>
      {children}
    </LanguageContext.Provider>
  );
}

function UserProvider({ children }: { children: React.ReactNode }) {
  const [user, setUser] = useState<UserContextType["user"]>(null);

  const login = async (credentials: { username: string; password: string }) => {
    // จำลอง API call
    await new Promise(resolve => setTimeout(resolve, 1000));
    setUser({ id: "1", name: credentials.username, role: "user" });
  };

  const logout = () => setUser(null);

  return (
    <UserContext.Provider value={{ user, login, logout }}>
      {children}
    </UserContext.Provider>
  );
}

// AppProvider รวม providers ทั้งหมด
function AppProvider({ children }: { children: React.ReactNode }) {
  return (
    <ThemeProvider>
      <LanguageProvider>
        <UserProvider>
          {children}
        </UserProvider>
      </LanguageProvider>
    </ThemeProvider>
  );
}

// Component ที่ใช้ providers
function Header() {
  const { theme, toggleTheme } = useTheme();
  const { t, language, setLanguage } = useLanguage();
  const { user, logout } = useUser();

  return (
    <header>
      <h1>{t("greeting")}, {user?.name || "ผู้เยี่ยมชม"}!</h1>
      <button onClick={toggleTheme}>
        {theme === "light" ? "🌙" : "☀️"}
      </button>
      <button onClick={() => setLanguage(language === "th" ? "en" : "th")}>
        {language.toUpperCase()}
      </button>
      {user && <button onClick={logout}>{t("logout")}</button>}
    </header>
  );
}
```

---

## Step 1618: React.forwardRef และ useImperativeHandle

```tsx
import React, { forwardRef, useImperativeHandle, useRef, useState } from "react";

// forwardRef - ส่ง ref ผ่าน component
interface InputProps extends React.InputHTMLAttributes<HTMLInputElement> {
  label: string;
  error?: string;
}

const Input = forwardRef<HTMLInputElement, InputProps>(
  ({ label, error, ...props }, ref) => {
    return (
      <div className="input-wrapper">
        <label className="input-label">{label}</label>
        <input
          ref={ref}
          className={`input ${error ? "input--error" : ""}`}
          {...props}
        />
        {error && <span className="input-error">{error}</span>}
      </div>
    );
  }
);

Input.displayName = "Input";

// useImperativeHandle - กำหนด API ของ ref
interface ModalHandle {
  open: () => void;
  close: () => void;
  isOpen: boolean;
}

interface ModalProps {
  title: string;
  children: React.ReactNode;
  onClose?: () => void;
}

const Modal = forwardRef<ModalHandle, ModalProps>(
  ({ title, children, onClose }, ref) => {
    const [isOpen, setIsOpen] = useState(false);

    useImperativeHandle(ref, () => ({
      open() {
        setIsOpen(true);
      },
      close() {
        setIsOpen(false);
        onClose?.();
      },
      get isOpen() {
        return isOpen;
      }
    }));

    if (!isOpen) return null;

    return (
      <div className="modal-overlay">
        <div className="modal">
          <div className="modal-header">
            <h2>{title}</h2>
            <button onClick={() => {
              setIsOpen(false);
              onClose?.();
            }}>✕</button>
          </div>
          <div className="modal-body">{children}</div>
        </div>
      </div>
    );
  }
);

Modal.displayName = "Modal";

// การใช้งาน
function App() {
  const inputRef = useRef<HTMLInputElement>(null);
  const modalRef = useRef<ModalHandle>(null);

  return (
    <div>
      <Input
        ref={inputRef}
        label="ชื่อ"
        placeholder="กรอกชื่อ"
        error=""
      />
      
      <button onClick={() => {
        inputRef.current?.focus();
      }}>
        Focus Input
      </button>
      
      <button onClick={() => modalRef.current?.open()}>
        เปิด Modal
      </button>
      
      <Modal
        ref={modalRef}
        title="ยืนยันการกระทำ"
        onClose={() => console.log("modal closed")}
      >
        <p>คุณแน่ใจหรือไม่?</p>
        <button onClick={() => modalRef.current?.close()}>ยืนยัน</button>
      </Modal>
    </div>
  );
}
```

---

## Step 1619: React.lazy และ Suspense

```tsx
import React, { lazy, Suspense, useState } from "react";

// Code splitting ด้วย React.lazy
const HeavyDashboard = lazy(() => import("./HeavyDashboard"));
const DataVisualization = lazy(() =>
  import("./DataVisualization").then(module => ({
    default: module.DataVisualization
  }))
);

// Custom loading component
function LoadingSpinner({ message = "กำลังโหลด..." }: { message?: string }) {
  return (
    <div className="loading">
      <div className="spinner" />
      <p>{message}</p>
    </div>
  );
}

// Error boundary สำหรับ Suspense
class SuspenseBoundary extends React.Component<
  { children: React.ReactNode; fallback: React.ReactNode },
  { hasError: boolean; error?: Error }
> {
  state = { hasError: false };

  static getDerivedStateFromError(error: Error) {
    return { hasError: true, error };
  }

  render() {
    if (this.state.hasError) {
      return (
        <div className="error">
          <h2>โหลด component ไม่สำเร็จ</h2>
          <button onClick={() => this.setState({ hasError: false })}>
            ลองใหม่
          </button>
        </div>
      );
    }
    return (
      <Suspense fallback={this.props.fallback}>
        {this.props.children}
      </Suspense>
    );
  }
}

// Route-based code splitting
function Router() {
  const [currentRoute, setCurrentRoute] = useState<"home" | "dashboard" | "charts">("home");
  
  const routes = {
    home: lazy(() => import("./pages/Home")),
    dashboard: lazy(() => import("./pages/Dashboard")),
    charts: lazy(() => import("./pages/Charts"))
  };
  
  const CurrentPage = routes[currentRoute];

  return (
    <div>
      <nav>
        <button onClick={() => setCurrentRoute("home")}>หน้าหลัก</button>
        <button onClick={() => setCurrentRoute("dashboard")}>Dashboard</button>
        <button onClick={() => setCurrentRoute("charts")}>กราฟ</button>
      </nav>
      
      <SuspenseBoundary fallback={<LoadingSpinner message="กำลังโหลดหน้า..." />}>
        <CurrentPage />
      </SuspenseBoundary>
    </div>
  );
}

// Lazy loading images
const LazyImage = lazy(() =>
  Promise.resolve({
    default: ({ src, alt }: { src: string; alt: string }) => (
      <img src={src} alt={alt} />
    )
  })
);

// Preloading ล่วงหน้า
function preloadComponent(importFn: () => Promise<any>) {
  const component = lazy(importFn);
  // Trigger the import
  importFn().catch(() => {});
  return component;
}

// Preload dashboard เมื่อ hover ปุ่ม
const PreloadedDashboard = preloadComponent(() => import("./pages/Dashboard"));
```

---

## Step 1620: Error Boundaries

```tsx
import React, { Component, ErrorInfo, ReactNode } from "react";

interface ErrorBoundaryState {
  hasError: boolean;
  error: Error | null;
  errorInfo: ErrorInfo | null;
}

interface ErrorBoundaryProps {
  children: ReactNode;
  fallback?: ReactNode | ((error: Error, reset: () => void) => ReactNode);
  onError?: (error: Error, errorInfo: ErrorInfo) => void;
}

class ErrorBoundary extends Component<ErrorBoundaryProps, ErrorBoundaryState> {
  state: ErrorBoundaryState = {
    hasError: false,
    error: null,
    errorInfo: null
  };

  static getDerivedStateFromError(error: Error): Partial<ErrorBoundaryState> {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, errorInfo: ErrorInfo) {
    this.setState({ errorInfo });
    this.props.onError?.(error, errorInfo);
    
    // Log to error tracking service
    console.error("Error caught by ErrorBoundary:", error, errorInfo);
  }

  reset = () => {
    this.setState({ hasError: false, error: null, errorInfo: null });
  };

  render() {
    if (this.state.hasError && this.state.error) {
      const { fallback } = this.props;
      
      if (typeof fallback === "function") {
        return fallback(this.state.error, this.reset);
      }
      
      if (fallback) return fallback;
      
      return (
        <div className="error-boundary">
          <h2>เกิดข้อผิดพลาด!</h2>
          <p>{this.state.error.message}</p>
          {process.env.NODE_ENV === "development" && (
            <details>
              <summary>รายละเอียดข้อผิดพลาด</summary>
              <pre>{this.state.errorInfo?.componentStack}</pre>
            </details>
          )}
          <button onClick={this.reset}>ลองใหม่</button>
        </div>
      );
    }

    return this.props.children;
  }
}

// useErrorBoundary hook (React 18+)
// นำมาใช้กับ functional components
function useThrowError() {
  const [error, setError] = React.useState<Error | null>(null);
  
  if (error) throw error;
  
  return {
    throwError: setError
  };
}

// ตัวอย่าง component ที่อาจเกิด error
function UnstableComponent() {
  const [shouldError, setShouldError] = React.useState(false);
  
  if (shouldError) {
    throw new Error("Component เกิดข้อผิดพลาด!");
  }
  
  return (
    <div>
      <p>Component ปกติ</p>
      <button onClick={() => setShouldError(true)}>
        จำลองข้อผิดพลาด
      </button>
    </div>
  );
}

// การใช้งาน
function App() {
  return (
    <ErrorBoundary
      fallback={(error, reset) => (
        <div style={{ color: "red", padding: 20 }}>
          <h3>Oops! {error.message}</h3>
          <button onClick={reset}>กู้คืน</button>
        </div>
      )}
      onError={(error) => {
        // ส่ง error ไปยัง Sentry, LogRocket, etc.
        console.log("Reporting error:", error.message);
      }}
    >
      <UnstableComponent />
    </ErrorBoundary>
  );
}
```

---

## Step 1621: React.memo Deep Dive

```tsx
import React, { memo, useState, useCallback, useMemo, useRef, useEffect } from "react";

// React.memo พร้อม custom comparison
interface ListItemProps {
  id: number;
  name: string;
  tags: string[];
  onSelect: (id: number) => void;
}

// ไม่ใช้ memo - re-render ทุกครั้งที่ parent re-render
function ListItemBad({ id, name, tags, onSelect }: ListItemProps) {
  console.log(`ListItem ${id} rendered`);
  return (
    <div onClick={() => onSelect(id)}>
      {name} - {tags.join(", ")}
    </div>
  );
}

// ใช้ memo พื้นฐาน
const ListItemMemo = memo(ListItemBad);

// ใช้ memo พร้อม custom comparison
const ListItemCustom = memo(
  ({ id, name, tags, onSelect }: ListItemProps) => {
    console.log(`ListItemCustom ${id} rendered`);
    return (
      <div onClick={() => onSelect(id)}>
        {name} - {tags.join(", ")}
      </div>
    );
  },
  (prevProps, nextProps) => {
    // คืน true = ไม่ re-render (props เหมือนเดิม)
    // คืน false = re-render
    return (
      prevProps.id === nextProps.id &&
      prevProps.name === nextProps.name &&
      prevProps.tags.length === nextProps.tags.length &&
      prevProps.tags.every((tag, i) => tag === nextProps.tags[i]) &&
      prevProps.onSelect === nextProps.onSelect
    );
  }
);

// ตัวอย่างการใช้ useCallback กับ memo
function ExpensiveList() {
  const [items, setItems] = useState([
    { id: 1, name: "Item 1", tags: ["a", "b"] },
    { id: 2, name: "Item 2", tags: ["c", "d"] },
    { id: 3, name: "Item 3", tags: ["e", "f"] },
  ]);
  const [selectedId, setSelectedId] = useState<number | null>(null);
  const [filter, setFilter] = useState("");

  // ถ้าไม่ใช้ useCallback, onSelect จะเป็น function ใหม่ทุก render
  // ทำให้ memo ไม่ทำงาน
  const handleSelect = useCallback((id: number) => {
    setSelectedId(id);
  }, []); // dependency array ว่าง = function ไม่เปลี่ยน

  // useMemo สำหรับ filtered items
  const filteredItems = useMemo(
    () => items.filter(item => item.name.includes(filter)),
    [items, filter]
  );

  return (
    <div>
      <input
        value={filter}
        onChange={e => setFilter(e.target.value)}
        placeholder="ค้นหา..."
      />
      <div>Selected: {selectedId}</div>
      {filteredItems.map(item => (
        <ListItemCustom
          key={item.id}
          {...item}
          onSelect={handleSelect}
        />
      ))}
    </div>
  );
}

// เมื่อใดควรใช้ memo
// 1. Component re-render บ่อยโดยไม่จำเป็น
// 2. Component มี props ที่ไม่เปลี่ยนบ่อย
// 3. Component มี render cost สูง

// เมื่อใดไม่ควรใช้ memo
// 1. Component ที่ render เร็ว
// 2. Props เปลี่ยนเกือบทุก render
// 3. Component ที่มี children เยอะ (overhead ของการ compare อาจสูงกว่า re-render)
```

---

## Step 1622: useTransition และ startTransition (React 18)

```tsx
import React, { useState, useTransition, startTransition, useDeferredValue } from "react";

// useTransition - mark state updates as non-urgent
function SearchWithTransition() {
  const [query, setQuery] = useState("");
  const [results, setResults] = useState<string[]>([]);
  const [isPending, startTransitionFn] = useTransition();

  // Simulate expensive search
  function performSearch(q: string): string[] {
    const allItems = Array.from({ length: 10000 }, (_, i) => `Item ${i + 1}`);
    return allItems.filter(item => 
      item.toLowerCase().includes(q.toLowerCase())
    );
  }

  function handleSearch(e: React.ChangeEvent<HTMLInputElement>) {
    const value = e.target.value;
    setQuery(value); // urgent update - input ต้อง update ทันที
    
    startTransitionFn(() => {
      // non-urgent update - สามารถรอได้
      setResults(performSearch(value));
    });
  }

  return (
    <div>
      <input
        value={query}
        onChange={handleSearch}
        placeholder="ค้นหา..."
      />
      {isPending && <span>🔍 กำลังค้นหา...</span>}
      <ul>
        {results.slice(0, 20).map((item, i) => (
          <li key={i}>{item}</li>
        ))}
      </ul>
      <p>พบ {results.length} รายการ</p>
    </div>
  );
}

// useDeferredValue - defer ค่าของ value
function DeferredSearchExample() {
  const [query, setQuery] = useState("");
  const deferredQuery = useDeferredValue(query);
  
  // Component นี้จะใช้ deferredQuery ซึ่งอาจล่าช้ากว่า query จริง
  const isStale = query !== deferredQuery;

  return (
    <div>
      <input
        value={query}
        onChange={e => setQuery(e.target.value)}
        placeholder="ค้นหา..."
      />
      <div style={{ opacity: isStale ? 0.5 : 1 }}>
        <ExpensiveResults query={deferredQuery} />
      </div>
    </div>
  );
}

// Expensive component ที่ใช้ deferredValue
const ExpensiveResults = React.memo(({ query }: { query: string }) => {
  // Simulate expensive rendering
  const startTime = performance.now();
  while (performance.now() - startTime < 1) {} // Artificial delay
  
  return (
    <div>
      <p>ผลลัพธ์สำหรับ: "{query}"</p>
    </div>
  );
});
```

---

## Step 1623: React Server Components

```tsx
// React Server Components (RSC) - ทำงานบน server เท่านั้น
// ไม่มี JavaScript ส่งไปยัง client

// Server Component (app/users/page.tsx - Next.js 13+)
// ไม่มี "use client" directive = เป็น Server Component โดย default

async function UsersPage() {
  // ดึงข้อมูลจาก database โดยตรง (ไม่ผ่าน API)
  const users = await db.query("SELECT * FROM users LIMIT 10");
  
  return (
    <div>
      <h1>รายชื่อผู้ใช้</h1>
      {users.map(user => (
        <UserCard key={user.id} user={user} />
      ))}
    </div>
  );
}

// Server Component ที่ดึงข้อมูลของตัวเอง
async function UserCard({ user }: { user: User }) {
  // สามารถดึง additional data ได้
  const profile = await fetchUserProfile(user.id);
  
  return (
    <div className="user-card">
      <img src={profile.avatar} alt={user.name} />
      <h2>{user.name}</h2>
      <p>{profile.bio}</p>
      {/* Client Component ผสมกับ Server Component ได้ */}
      <LikeButton userId={user.id} initialLikes={profile.likes} />
    </div>
  );
}

// Client Component ต้องมี "use client" directive
"use client";

import { useState } from "react";

function LikeButton({ userId, initialLikes }: { userId: string; initialLikes: number }) {
  const [likes, setLikes] = useState(initialLikes);
  const [liked, setLiked] = useState(false);

  const handleLike = async () => {
    if (liked) return;
    setLiked(true);
    setLikes(prev => prev + 1);
    await fetch(`/api/users/${userId}/like`, { method: "POST" });
  };

  return (
    <button onClick={handleLike} disabled={liked}>
      ❤️ {likes}
    </button>
  );
}

// Server Actions (Next.js 14+)
"use server";

async function createUser(formData: FormData) {
  const name = formData.get("name") as string;
  const email = formData.get("email") as string;
  
  // ทำงานบน server โดยตรง
  await db.insert("users", { name, email });
  
  // revalidate cache
  revalidatePath("/users");
}

// ใช้กับ form
function CreateUserForm() {
  return (
    <form action={createUser}>
      <input name="name" placeholder="ชื่อ" />
      <input name="email" placeholder="Email" />
      <button type="submit">สร้างผู้ใช้</button>
    </form>
  );
}
```

---

## Step 1624: Streaming SSR

```tsx
// Streaming SSR กับ React 18 + Next.js

// Loading UI ที่แสดงระหว่าง stream
// app/dashboard/loading.tsx
export default function DashboardLoading() {
  return (
    <div className="dashboard-skeleton">
      <div className="skeleton header" />
      <div className="skeleton stats" />
      <div className="skeleton chart" />
    </div>
  );
}

// app/dashboard/page.tsx - Streaming ข้อมูล
import { Suspense } from "react";

export default function Dashboard() {
  return (
    <div>
      {/* แสดงทันที */}
      <h1>Dashboard</h1>
      
      {/* Stream ทีละส่วน */}
      <Suspense fallback={<div>โหลด Stats...</div>}>
        <DashboardStats />
      </Suspense>
      
      <Suspense fallback={<div>โหลดกราฟ...</div>}>
        <RevenueChart />
      </Suspense>
      
      <Suspense fallback={<div>โหลดตาราง...</div>}>
        <RecentOrders />
      </Suspense>
    </div>
  );
}

// แต่ละส่วนโหลดข้อมูลของตัวเอง
async function DashboardStats() {
  const stats = await fetchStats(); // async call
  return <StatsGrid data={stats} />;
}

async function RevenueChart() {
  const data = await fetchRevenueData(); // async call (อาจช้ากว่า)
  return <Chart data={data} />;
}

// Parallel data fetching
async function ParallelDataPage() {
  // ดึงข้อมูลพร้อมกัน (ไม่รอเป็น sequence)
  const [users, posts, comments] = await Promise.all([
    fetchUsers(),
    fetchPosts(),
    fetchComments()
  ]);
  
  return (
    <div>
      <UserList users={users} />
      <PostList posts={posts} />
      <CommentList comments={comments} />
    </div>
  );
}
```

---

## Step 1625: Performance Optimization Strategies

```tsx
import React, { memo, useMemo, useCallback, useState, useRef, useEffect } from "react";

// 1. Virtualization สำหรับ long lists
function VirtualizedList({ items }: { items: string[] }) {
  const [scrollTop, setScrollTop] = useState(0);
  const containerRef = useRef<HTMLDivElement>(null);
  const ITEM_HEIGHT = 50;
  const VISIBLE_COUNT = 20;
  
  const startIndex = Math.floor(scrollTop / ITEM_HEIGHT);
  const endIndex = Math.min(startIndex + VISIBLE_COUNT, items.length);
  const visibleItems = items.slice(startIndex, endIndex);
  const totalHeight = items.length * ITEM_HEIGHT;
  const offsetY = startIndex * ITEM_HEIGHT;

  return (
    <div
      ref={containerRef}
      style={{ height: 400, overflow: "auto" }}
      onScroll={e => setScrollTop(e.currentTarget.scrollTop)}
    >
      <div style={{ height: totalHeight, position: "relative" }}>
        <div style={{ transform: `translateY(${offsetY}px)` }}>
          {visibleItems.map((item, i) => (
            <div key={startIndex + i} style={{ height: ITEM_HEIGHT }}>
              {item}
            </div>
          ))}
        </div>
      </div>
    </div>
  );
}

// 2. Debouncing search input
function useDebounce<T>(value: T, delay: number): T {
  const [debouncedValue, setDebouncedValue] = useState(value);

  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);

    return () => clearTimeout(timer);
  }, [value, delay]);

  return debouncedValue;
}

function SearchInput({ onSearch }: { onSearch: (q: string) => void }) {
  const [query, setQuery] = useState("");
  const debouncedQuery = useDebounce(query, 300);

  useEffect(() => {
    if (debouncedQuery) {
      onSearch(debouncedQuery);
    }
  }, [debouncedQuery, onSearch]);

  return (
    <input
      value={query}
      onChange={e => setQuery(e.target.value)}
      placeholder="ค้นหา..."
    />
  );
}

// 3. Image lazy loading
function LazyLoadImage({ src, alt }: { src: string; alt: string }) {
  const imgRef = useRef<HTMLImageElement>(null);
  const [isLoaded, setIsLoaded] = useState(false);
  const [isInView, setIsInView] = useState(false);

  useEffect(() => {
    const observer = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting) {
          setIsInView(true);
          observer.disconnect();
        }
      },
      { threshold: 0.1 }
    );

    if (imgRef.current) {
      observer.observe(imgRef.current);
    }

    return () => observer.disconnect();
  }, []);

  return (
    <div ref={imgRef} className="lazy-image">
      {isInView ? (
        <img
          src={src}
          alt={alt}
          onLoad={() => setIsLoaded(true)}
          style={{ opacity: isLoaded ? 1 : 0, transition: "opacity 0.3s" }}
        />
      ) : (
        <div className="image-placeholder" />
      )}
    </div>
  );
}

// 4. Context optimization ด้วย splitting
// แยก context ที่ update บ่อยออกจาก context ที่ไม่ค่อย update
interface StableContextType {
  user: { id: string; name: string } | null;
  theme: "light" | "dark";
}

interface VolatileContextType {
  notifications: number;
  isLoading: boolean;
}

const StableContext = React.createContext<StableContextType>({} as StableContextType);
const VolatileContext = React.createContext<VolatileContextType>({} as VolatileContextType);

// Components ที่ subscribe เฉพาะ context ที่ต้องการ
const UserAvatar = memo(() => {
  const { user } = React.useContext(StableContext);
  // ไม่ re-render เมื่อ notifications เปลี่ยน
  return <div>{user?.name[0]}</div>;
});

const NotificationBadge = memo(() => {
  const { notifications } = React.useContext(VolatileContext);
  // re-render เฉพาะเมื่อ notifications เปลี่ยน
  return <span>{notifications}</span>;
});
```

---

## Step 1626-1630: Code Splitting และ Performance Patterns

```tsx
// Advanced code splitting strategies

// 1. Route-based splitting
const routes = {
  "/": lazy(() => import("./pages/Home")),
  "/about": lazy(() => import("./pages/About")),
  "/dashboard": lazy(() => import("./pages/Dashboard")),
  "/users": lazy(() => import("./pages/Users")),
  "/settings": lazy(() => import("./pages/Settings")),
};

// 2. Feature-based splitting
const RichTextEditor = lazy(() => import("./features/RichTextEditor"));
const VideoPlayer = lazy(() => import("./features/VideoPlayer"));
const DataGrid = lazy(() => import("./features/DataGrid"));

// 3. Library splitting
function DatePicker({ value, onChange }: { value: Date; onChange: (date: Date) => void }) {
  const [CalendarComponent, setCalendarComponent] = useState<React.ComponentType<any> | null>(null);

  const loadCalendar = async () => {
    const { Calendar } = await import("react-calendar");
    setCalendarComponent(() => Calendar);
  };

  return (
    <div>
      <input
        value={value.toLocaleDateString("th-TH")}
        onFocus={loadCalendar}
        readOnly
      />
      {CalendarComponent && (
        <Suspense fallback={<div>โหลด Calendar...</div>}>
          <CalendarComponent value={value} onChange={onChange} />
        </Suspense>
      )}
    </div>
  );
}

// 4. Intersection Observer สำหรับ lazy loading sections
function LazySection({
  children,
  placeholder,
}: {
  children: React.ReactNode;
  placeholder?: React.ReactNode;
}) {
  const [isVisible, setIsVisible] = useState(false);
  const ref = useRef<HTMLDivElement>(null);

  useEffect(() => {
    const observer = new IntersectionObserver(
      ([entry]) => {
        if (entry.isIntersecting) {
          setIsVisible(true);
          observer.disconnect();
        }
      },
      { rootMargin: "200px" }
    );

    if (ref.current) observer.observe(ref.current);
    return () => observer.disconnect();
  }, []);

  return (
    <div ref={ref}>
      {isVisible ? children : (placeholder || <div style={{ height: 200 }} />)}
    </div>
  );
}

// 5. Web Workers สำหรับ expensive computations
function useWorker<T, R>(
  worker: Worker,
  input: T
): { result: R | null; loading: boolean; error: Error | null } {
  const [state, setState] = useState<{
    result: R | null;
    loading: boolean;
    error: Error | null;
  }>({ result: null, loading: false, error: null });

  useEffect(() => {
    setState({ result: null, loading: true, error: null });
    
    worker.postMessage(input);
    
    worker.onmessage = (e: MessageEvent<R>) => {
      setState({ result: e.data, loading: false, error: null });
    };
    
    worker.onerror = (e: ErrorEvent) => {
      setState({ result: null, loading: false, error: new Error(e.message) });
    };

    return () => {
      worker.onmessage = null;
      worker.onerror = null;
    };
  }, [worker, input]);

  return state;
}
```

---

## แบบฝึกหัด (Exercises)

### แบบฝึกหัดที่ 1: Compound Components
สร้าง `<Tabs>` component แบบ Compound Components:

```tsx
// การใช้งานที่ต้องการ
<Tabs defaultTab="tab1">
  <Tabs.List>
    <Tabs.Tab id="tab1">แท็บ 1</Tabs.Tab>
    <Tabs.Tab id="tab2">แท็บ 2</Tabs.Tab>
  </Tabs.List>
  <Tabs.Panels>
    <Tabs.Panel id="tab1">เนื้อหาแท็บ 1</Tabs.Panel>
    <Tabs.Panel id="tab2">เนื้อหาแท็บ 2</Tabs.Panel>
  </Tabs.Panels>
</Tabs>
```

### แบบฝึกหัดที่ 2: Custom Hook พร้อม Render Props
สร้าง `useInfiniteScroll` hook:

```tsx
function useInfiniteScroll<T>({
  fetchFn,
  pageSize
}: {
  fetchFn: (page: number) => Promise<T[]>;
  pageSize: number;
}): {
  items: T[];
  loadMore: () => void;
  hasMore: boolean;
  loading: boolean;
  error: Error | null;
}
```

### แบบฝึกหัดที่ 3: HOC Composition
สร้าง `compose` utility สำหรับ HOCs:

```tsx
// ใช้งาน:
const EnhancedComponent = compose(
  withAuth,
  withLogging,
  withErrorBoundary,
)(MyComponent);
```

### แบบฝึกหัดที่ 4: State Machine Hook
สร้าง `useStateMachine` hook:

```tsx
const { state, send } = useStateMachine({
  initial: "idle",
  states: {
    idle: { on: { FETCH: "loading" } },
    loading: { on: { SUCCESS: "success", ERROR: "error" } },
    success: { on: { RESET: "idle" } },
    error: { on: { RETRY: "loading", RESET: "idle" } }
  }
});
```

### แบบฝึกหัดที่ 5: Virtual List
สร้าง `VirtualList` component ที่:
- แสดง 100,000 items ได้อย่าง smooth
- รองรับ variable height items
- มี search/filter functionality
- มี scroll-to-item method

---

## สรุป (Summary)

ใน Part 82 นี้เราได้เรียนรู้ React patterns ขั้นสูง:

1. **Compound Components** - Components ที่ทำงานร่วมกันผ่าน Context
2. **Render Props** - ส่ง render function เป็น prop
3. **HOC** - Function ที่เพิ่มความสามารถให้ components
4. **Control Props** - ให้ผู้ใช้ควบคุม state จากภายนอก
5. **Props Getters** - ให้ flexibility ในการกำหนด props
6. **State Reducer** - ให้ผู้ใช้ customize state transitions
7. **Provider Pattern** - แชร์ state ผ่าน Context
8. **forwardRef & useImperativeHandle** - expose imperative API
9. **React.lazy & Suspense** - Code splitting
10. **Error Boundaries** - จัดการ errors
11. **React.memo** - Prevent unnecessary re-renders
12. **useTransition** - Mark non-urgent updates
13. **useDeferredValue** - Defer value updates
14. **Server Components** - Rendering on server
15. **Streaming SSR** - Stream HTML ทีละส่วน
16. **Performance Optimization** - Virtualization, debounce, lazy loading
17. **Code Splitting** - ลด bundle size
18. **Web Workers** - Offload heavy computations

การเลือกใช้ pattern ที่เหมาะสมช่วยให้ code อ่านง่าย maintain ได้ง่าย และ perform ดีขึ้น
