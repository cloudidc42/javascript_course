# Part 53: Design Patterns - Behavioral Patterns (Steps 1031-1050)

## การออกแบบ Patterns สำหรับพฤติกรรมของวัตถุ

**คำอธิบาย:** Behavioral Patterns เกี่ยวข้องกับการสื่อสารและพฤติกรรมระหว่าง objects เราจะเรียนรู้ Observer, Strategy, Command, Iterator, Mediator, State, Chain of Responsibility, Template Method, Visitor และ Memento patterns

---

## Step 1031: Behavioral Patterns Overview

```javascript
// Behavioral Patterns ช่วยจัดการ:
// - การสื่อสารระหว่าง objects
// - การแบ่งความรับผิดชอบ
// - การเปลี่ยนพฤติกรรมแบบ dynamic

const behavioralPatterns = {
  Observer: 'ติดตามการเปลี่ยนแปลง (subscribe/publish)',
  Strategy: 'เลือก algorithm ได้ตอน runtime',
  Command: 'encapsulate actions เป็น objects',
  Iterator: 'เข้าถึง elements ทีละตัว',
  Mediator: 'ศูนย์กลางการสื่อสาร',
  State: 'เปลี่ยนพฤติกรรมตาม state',
  ChainOfResponsibility: 'ส่งต่อ request ตามลำดับ',
  TemplateMethod: 'โครงสร้างใน base, รายละเอียดใน subclass',
  Visitor: 'เพิ่ม operation โดยไม่แก้ class',
  Memento: 'บันทึก/กู้คืน state'
};

Object.entries(behavioralPatterns).forEach(([name, desc]) => {
  console.log(`${name}: ${desc}`);
});
```

---

## Step 1032: Observer Pattern - พื้นฐาน

**Observer Pattern** ช่วยให้ objects รับรู้การเปลี่ยนแปลงของ objects อื่นโดยไม่ต้อง polling

```javascript
// ========================
// Observer Pattern - Custom EventEmitter
// ========================

class EventEmitter {
  constructor() {
    this._events = new Map();
    this._maxListeners = 10;
  }
  
  on(event, listener) {
    if (!this._events.has(event)) {
      this._events.set(event, []);
    }
    
    const listeners = this._events.get(event);
    
    if (listeners.length >= this._maxListeners) {
      console.warn(`Warning: ${event} has ${listeners.length} listeners. Possible memory leak.`);
    }
    
    listeners.push(listener);
    return this; // chaining
  }
  
  once(event, listener) {
    const wrapper = (...args) => {
      listener.apply(this, args);
      this.off(event, wrapper);
    };
    wrapper._original = listener;
    return this.on(event, wrapper);
  }
  
  off(event, listener) {
    if (!this._events.has(event)) return this;
    
    const listeners = this._events.get(event);
    const filtered = listeners.filter(l => l !== listener && l._original !== listener);
    
    if (filtered.length === 0) {
      this._events.delete(event);
    } else {
      this._events.set(event, filtered);
    }
    
    return this;
  }
  
  emit(event, ...args) {
    if (!this._events.has(event)) return false;
    
    const listeners = [...this._events.get(event)]; // copy เพื่อป้องกัน side effect
    listeners.forEach(listener => {
      try {
        listener.apply(this, args);
      } catch (error) {
        console.error(`Error in ${event} listener:`, error);
      }
    });
    
    return true;
  }
  
  removeAllListeners(event) {
    if (event) {
      this._events.delete(event);
    } else {
      this._events.clear();
    }
    return this;
  }
  
  listenerCount(event) {
    return this._events.get(event)?.length || 0;
  }
  
  eventNames() {
    return [...this._events.keys()];
  }
}

// ใช้งาน
const emitter = new EventEmitter();

// สมัครรับ events
const onUserCreated = (user) => {
  console.log('New user created:', user.name);
};

const onUserCreatedWelcome = (user) => {
  console.log(`Welcome ${user.name}! Check your email.`);
};

emitter.on('user:created', onUserCreated);
emitter.on('user:created', onUserCreatedWelcome);
emitter.on('user:deleted', (user) => {
  console.log(`User ${user.name} deleted`);
});

// Once - ฟังแค่ครั้งเดียว
emitter.once('server:started', (port) => {
  console.log(`Server started on port ${port}`);
});

// Emit events
emitter.emit('user:created', { id: 1, name: 'สมชาย', email: 'test@example.com' });
emitter.emit('user:created', { id: 2, name: 'สมหญิง', email: 'test2@example.com' });
emitter.emit('server:started', 3000);
emitter.emit('server:started', 3000); // ครั้งที่ 2 จะไม่ทำงาน

// ยกเลิกการรับ event
emitter.off('user:created', onUserCreated);
emitter.emit('user:created', { id: 3, name: 'วิชัย' }); // มีแค่ welcome message

console.log('Events:', emitter.eventNames());
console.log('user:created listeners:', emitter.listenerCount('user:created'));
```

---

## Step 1033: Observer Pattern - Store Pattern (React-like)

```javascript
// ========================
// Store Pattern (Redux-like)
// ========================

class Store extends EventEmitter {
  constructor(reducer, initialState = {}) {
    super();
    this._reducer = reducer;
    this._state = initialState;
    this._middlewares = [];
    this._history = [];
  }
  
  getState() {
    return { ...this._state };
  }
  
  use(middleware) {
    this._middlewares.push(middleware);
    return this;
  }
  
  dispatch(action) {
    // รัน middleware
    const chain = this._middlewares.reduce(
      (next, middleware) => (action) => middleware(this)(next)(action),
      (action) => this._processAction(action)
    );
    
    return chain(action);
  }
  
  _processAction(action) {
    const prevState = this._state;
    this._state = this._reducer(this._state, action);
    
    this._history.push({ action, prevState, newState: { ...this._state } });
    
    this.emit('change', this._state, prevState, action);
    this.emit(`action:${action.type}`, this._state, prevState);
    
    return action;
  }
  
  subscribe(listener) {
    this.on('change', listener);
    return () => this.off('change', listener);
  }
  
  getHistory() {
    return [...this._history];
  }
}

// Reducer
function userReducer(state = { users: [], loading: false, error: null }, action) {
  switch (action.type) {
    case 'FETCH_USERS_START':
      return { ...state, loading: true, error: null };
    
    case 'FETCH_USERS_SUCCESS':
      return { ...state, loading: false, users: action.payload };
    
    case 'FETCH_USERS_FAILURE':
      return { ...state, loading: false, error: action.payload };
    
    case 'ADD_USER':
      return {
        ...state,
        users: [...state.users, { id: Date.now(), ...action.payload }]
      };
    
    case 'UPDATE_USER':
      return {
        ...state,
        users: state.users.map(user =>
          user.id === action.payload.id
            ? { ...user, ...action.payload }
            : user
        )
      };
    
    case 'DELETE_USER':
      return {
        ...state,
        users: state.users.filter(user => user.id !== action.payload)
      };
    
    default:
      return state;
  }
}

// Middleware
const loggingMiddleware = (store) => (next) => (action) => {
  console.log('Dispatching:', action);
  const result = next(action);
  console.log('New state:', store.getState());
  return result;
};

// สร้าง store
const store = new Store(userReducer);
store.use(loggingMiddleware);

// Subscribe
const unsubscribe = store.subscribe((newState, prevState, action) => {
  console.log(`\nState changed by action: ${action.type}`);
  console.log('Users count:', newState.users.length);
});

// Dispatch actions
store.dispatch({ type: 'ADD_USER', payload: { name: 'สมชาย', email: 'somchai@example.com' } });
store.dispatch({ type: 'ADD_USER', payload: { name: 'สมหญิง', email: 'somying@example.com' } });
store.dispatch({ type: 'FETCH_USERS_START' });
store.dispatch({
  type: 'FETCH_USERS_SUCCESS',
  payload: [
    { id: 1, name: 'วิชัย' },
    { id: 2, name: 'วิไล' }
  ]
});

unsubscribe(); // ยกเลิก subscription
store.dispatch({ type: 'ADD_USER', payload: { name: 'ทดสอบ' } }); // จะไม่ trigger listener
```

---

## Step 1034: Strategy Pattern

**Strategy Pattern** ช่วยให้เปลี่ยน algorithm ได้ตอน runtime โดยไม่แก้ client code

```javascript
// ========================
// Strategy Pattern - Payment
// ========================

// Strategy Interface
class PaymentStrategy {
  pay(amount, details) {
    throw new Error('Not implemented');
  }
  
  validate(details) {
    return { valid: true };
  }
  
  getName() {
    return 'Unknown Payment';
  }
}

// Concrete Strategies
class CreditCardStrategy extends PaymentStrategy {
  validate(details) {
    const errors = [];
    if (!details.cardNumber || details.cardNumber.length !== 16) {
      errors.push('Invalid card number');
    }
    if (!details.cvv || details.cvv.length < 3) {
      errors.push('Invalid CVV');
    }
    if (!details.expiryDate) {
      errors.push('Expiry date required');
    }
    return { valid: errors.length === 0, errors };
  }
  
  pay(amount, details) {
    const validation = this.validate(details);
    if (!validation.valid) {
      throw new Error(`Payment failed: ${validation.errors.join(', ')}`);
    }
    
    const maskedCard = `****-****-****-${details.cardNumber.slice(-4)}`;
    return {
      success: true,
      transactionId: `CC_${Date.now()}`,
      amount,
      method: this.getName(),
      card: maskedCard,
      message: `Charged ฿${amount.toLocaleString()} to ${maskedCard}`
    };
  }
  
  getName() { return 'Credit Card'; }
}

class PromptPayStrategy extends PaymentStrategy {
  validate(details) {
    const errors = [];
    if (!details.phoneOrIdNumber) {
      errors.push('Phone or ID number required for PromptPay');
    }
    return { valid: errors.length === 0, errors };
  }
  
  pay(amount, details) {
    return {
      success: true,
      transactionId: `PP_${Date.now()}`,
      amount,
      method: this.getName(),
      qrCode: `data:image/png;base64,QR_${Date.now()}`, // จำลอง QR
      message: `PromptPay QR generated for ฿${amount.toLocaleString()}`
    };
  }
  
  getName() { return 'PromptPay'; }
}

class CryptoStrategy extends PaymentStrategy {
  constructor(currency = 'BTC') {
    super();
    this.currency = currency;
    this.exchangeRate = currency === 'BTC' ? 2000000 : 5000; // จำลองอัตราแลกเปลี่ยน
  }
  
  pay(amount, details) {
    const cryptoAmount = amount / this.exchangeRate;
    return {
      success: true,
      transactionId: `CRYPTO_${Date.now()}`,
      amount,
      method: this.getName(),
      cryptoAmount: cryptoAmount.toFixed(8),
      currency: this.currency,
      address: details.walletAddress || `0x${Math.random().toString(16).substr(2, 40)}`,
      message: `Pay ${cryptoAmount.toFixed(8)} ${this.currency}`
    };
  }
  
  getName() { return `Crypto (${this.currency})`; }
}

class BankTransferStrategy extends PaymentStrategy {
  validate(details) {
    const errors = [];
    if (!details.bankCode) errors.push('Bank code required');
    if (!details.accountNumber) errors.push('Account number required');
    return { valid: errors.length === 0, errors };
  }
  
  pay(amount, details) {
    return {
      success: true,
      transactionId: `BANK_${Date.now()}`,
      amount,
      method: this.getName(),
      bankRef: `REF_${Math.random().toString(36).substr(2, 8).toUpperCase()}`,
      message: `Bank transfer initiated for ฿${amount.toLocaleString()}`
    };
  }
  
  getName() { return 'Bank Transfer'; }
}

// Context
class ShoppingCart {
  constructor() {
    this._items = [];
    this._paymentStrategy = null;
  }
  
  addItem(name, price, quantity = 1) {
    this._items.push({ name, price, quantity });
    return this;
  }
  
  removeItem(name) {
    this._items = this._items.filter(item => item.name !== name);
    return this;
  }
  
  getTotal() {
    return this._items.reduce((sum, item) => sum + (item.price * item.quantity), 0);
  }
  
  setPaymentStrategy(strategy) {
    this._paymentStrategy = strategy;
    return this;
  }
  
  checkout(paymentDetails = {}) {
    if (!this._paymentStrategy) {
      throw new Error('Payment strategy not set');
    }
    
    if (this._items.length === 0) {
      throw new Error('Cart is empty');
    }
    
    const total = this.getTotal();
    const result = this._paymentStrategy.pay(total, paymentDetails);
    
    console.log(`\n=== Checkout via ${this._paymentStrategy.getName()} ===`);
    console.log(`Items: ${this._items.map(i => i.name).join(', ')}`);
    console.log(`Total: ฿${total.toLocaleString()}`);
    console.log(`Result: ${result.message}`);
    
    return result;
  }
}

// ใช้งาน
const cart = new ShoppingCart();
cart.addItem('MacBook Pro', 65000)
    .addItem('Magic Mouse', 2500)
    .addItem('Monitor', 12000);

// เปลี่ยน payment strategy ได้
cart.setPaymentStrategy(new CreditCardStrategy());
cart.checkout({ cardNumber: '1234567890123456', cvv: '123', expiryDate: '12/26' });

cart.setPaymentStrategy(new PromptPayStrategy());
cart.checkout({ phoneOrIdNumber: '0812345678' });

cart.setPaymentStrategy(new CryptoStrategy('ETH'));
cart.checkout({ walletAddress: '0xABC123' });

// ========================
// Strategy Pattern - Sorting
// ========================

class SortStrategy {
  sort(data) { throw new Error('Not implemented'); }
  getName() { return 'Unknown'; }
}

class BubbleSortStrategy extends SortStrategy {
  sort(data) {
    const arr = [...data];
    const n = arr.length;
    
    for (let i = 0; i < n - 1; i++) {
      for (let j = 0; j < n - i - 1; j++) {
        if (arr[j] > arr[j + 1]) {
          [arr[j], arr[j + 1]] = [arr[j + 1], arr[j]];
        }
      }
    }
    
    return arr;
  }
  
  getName() { return 'Bubble Sort O(n²)'; }
}

class QuickSortStrategy extends SortStrategy {
  sort(data) {
    if (data.length <= 1) return data;
    
    const pivot = data[Math.floor(data.length / 2)];
    const left = data.filter(x => x < pivot);
    const middle = data.filter(x => x === pivot);
    const right = data.filter(x => x > pivot);
    
    return [...this.sort(left), ...middle, ...this.sort(right)];
  }
  
  getName() { return 'Quick Sort O(n log n)'; }
}

class MergeSortStrategy extends SortStrategy {
  sort(data) {
    if (data.length <= 1) return data;
    
    const mid = Math.floor(data.length / 2);
    const left = this.sort(data.slice(0, mid));
    const right = this.sort(data.slice(mid));
    
    return this._merge(left, right);
  }
  
  _merge(left, right) {
    const result = [];
    let i = 0, j = 0;
    
    while (i < left.length && j < right.length) {
      if (left[i] <= right[j]) {
        result.push(left[i++]);
      } else {
        result.push(right[j++]);
      }
    }
    
    return [...result, ...left.slice(i), ...right.slice(j)];
  }
  
  getName() { return 'Merge Sort O(n log n)'; }
}

class DataProcessor {
  constructor(strategy) {
    this._strategy = strategy;
  }
  
  setStrategy(strategy) {
    this._strategy = strategy;
    return this;
  }
  
  processData(data) {
    console.log(`\nSorting with ${this._strategy.getName()}`);
    const start = Date.now();
    const sorted = this._strategy.sort(data);
    const time = Date.now() - start;
    console.log(`Time: ${time}ms, First 5: [${sorted.slice(0, 5).join(', ')}]`);
    return sorted;
  }
}

// ใช้งาน
const data = Array.from({ length: 1000 }, () => Math.floor(Math.random() * 10000));
const processor = new DataProcessor(new BubbleSortStrategy());

processor.processData(data);
processor.setStrategy(new QuickSortStrategy()).processData(data);
processor.setStrategy(new MergeSortStrategy()).processData(data);
```

---

## Step 1035: Command Pattern

**Command Pattern** encapsulate request เป็น object ทำให้ทำ undo/redo ได้

```javascript
// ========================
// Command Pattern - Text Editor
// ========================

class TextEditor {
  constructor() {
    this._text = '';
    this._commandHistory = [];
    this._undoStack = [];
    this._redoStack = [];
  }
  
  getText() {
    return this._text;
  }
  
  executeCommand(command) {
    command.execute(this);
    this._undoStack.push(command);
    this._redoStack = []; // clear redo เมื่อมี command ใหม่
    return this;
  }
  
  undo() {
    if (this._undoStack.length === 0) {
      console.log('Nothing to undo');
      return this;
    }
    
    const command = this._undoStack.pop();
    command.undo(this);
    this._redoStack.push(command);
    return this;
  }
  
  redo() {
    if (this._redoStack.length === 0) {
      console.log('Nothing to redo');
      return this;
    }
    
    const command = this._redoStack.pop();
    command.execute(this);
    this._undoStack.push(command);
    return this;
  }
  
  getHistory() {
    return this._undoStack.map(cmd => cmd.toString());
  }
}

// Command Interface
class TextCommand {
  execute(editor) { throw new Error('Not implemented'); }
  undo(editor) { throw new Error('Not implemented'); }
  toString() { return 'Unknown Command'; }
}

// Concrete Commands
class InsertTextCommand extends TextCommand {
  constructor(text, position = null) {
    super();
    this._text = text;
    this._position = position;
    this._insertedAt = null;
  }
  
  execute(editor) {
    if (this._position !== null) {
      const current = editor.getText();
      this._insertedAt = this._position;
      editor._text = current.slice(0, this._position) + this._text + current.slice(this._position);
    } else {
      this._insertedAt = editor.getText().length;
      editor._text += this._text;
    }
  }
  
  undo(editor) {
    const current = editor.getText();
    editor._text = current.slice(0, this._insertedAt) + 
                   current.slice(this._insertedAt + this._text.length);
  }
  
  toString() { return `Insert("${this._text}" at ${this._insertedAt})`; }
}

class DeleteTextCommand extends TextCommand {
  constructor(start, end) {
    super();
    this._start = start;
    this._end = end;
    this._deletedText = null;
  }
  
  execute(editor) {
    const current = editor.getText();
    this._deletedText = current.slice(this._start, this._end);
    editor._text = current.slice(0, this._start) + current.slice(this._end);
  }
  
  undo(editor) {
    const current = editor.getText();
    editor._text = current.slice(0, this._start) + this._deletedText + current.slice(this._start);
  }
  
  toString() { return `Delete(${this._start}-${this._end}: "${this._deletedText}")`; }
}

class ReplaceTextCommand extends TextCommand {
  constructor(searchText, replaceText) {
    super();
    this._searchText = searchText;
    this._replaceText = replaceText;
    this._originalText = null;
  }
  
  execute(editor) {
    this._originalText = editor.getText();
    editor._text = this._originalText.split(this._searchText).join(this._replaceText);
  }
  
  undo(editor) {
    editor._text = this._originalText;
  }
  
  toString() { return `Replace("${this._searchText}" → "${this._replaceText}")`; }
}

class FormatCommand extends TextCommand {
  constructor(type) {
    super();
    this._type = type;
    this._originalText = null;
  }
  
  execute(editor) {
    this._originalText = editor.getText();
    switch (this._type) {
      case 'uppercase':
        editor._text = editor.getText().toUpperCase();
        break;
      case 'lowercase':
        editor._text = editor.getText().toLowerCase();
        break;
      case 'trim':
        editor._text = editor.getText().trim();
        break;
      case 'reverse':
        editor._text = editor.getText().split('').reverse().join('');
        break;
    }
  }
  
  undo(editor) {
    editor._text = this._originalText;
  }
  
  toString() { return `Format(${this._type})`; }
}

// Macro Command (Composite Command)
class MacroCommand extends TextCommand {
  constructor(name, commands = []) {
    super();
    this._name = name;
    this._commands = commands;
  }
  
  add(command) {
    this._commands.push(command);
    return this;
  }
  
  execute(editor) {
    this._commands.forEach(cmd => cmd.execute(editor));
  }
  
  undo(editor) {
    [...this._commands].reverse().forEach(cmd => cmd.undo(editor));
  }
  
  toString() { return `Macro(${this._name}: ${this._commands.length} commands)`; }
}

// ใช้งาน
const editor = new TextEditor();

editor.executeCommand(new InsertTextCommand('สวัสดี '));
console.log('After insert:', editor.getText());

editor.executeCommand(new InsertTextCommand('ชาวโลก!'));
console.log('After insert:', editor.getText());

editor.executeCommand(new FormatCommand('uppercase'));
console.log('After uppercase:', editor.getText());

editor.undo();
console.log('After undo:', editor.getText());

editor.undo();
console.log('After undo:', editor.getText());

editor.redo();
console.log('After redo:', editor.getText());

// Macro Command
const formatting = new MacroCommand('Format Document')
  .add(new ReplaceTextCommand('สวัสดี', 'Hello'))
  .add(new FormatCommand('trim'));

editor.executeCommand(formatting);
console.log('After macro:', editor.getText());

editor.undo();
console.log('After undo macro:', editor.getText());

console.log('\nCommand History:');
editor.getHistory().forEach(cmd => console.log(' -', cmd));
```

---

## Step 1036: Iterator Pattern

**Iterator Pattern** ให้วิธีการเข้าถึง elements ทีละตัวโดยไม่เปิดเผยโครงสร้างภายใน

```javascript
// ========================
// Custom Iterator
// ========================

// Range Iterator
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
        if ((step > 0 && current <= end) || (step < 0 && current >= end)) {
          const value = current;
          current += step;
          return { value, done: false };
        }
        return { value: undefined, done: true };
      },
      
      [Symbol.iterator]() { return this; }
    };
  }
  
  toArray() {
    return [...this];
  }
  
  map(fn) {
    return [...this].map(fn);
  }
  
  filter(fn) {
    return [...this].filter(fn);
  }
}

// ใช้งาน Range
const range = new Range(1, 10);
for (const num of range) {
  process.stdout.write(num + ' ');
}
console.log();

const evenNumbers = new Range(0, 20, 2).toArray();
console.log('Even:', evenNumbers);

const countdown = new Range(10, 0, -1).toArray();
console.log('Countdown:', countdown);

// ========================
// Tree Iterator
// ========================

class TreeNode {
  constructor(value) {
    this.value = value;
    this.left = null;
    this.right = null;
  }
}

class BinaryTree {
  constructor() {
    this.root = null;
  }
  
  insert(value) {
    const node = new TreeNode(value);
    
    if (!this.root) {
      this.root = node;
      return this;
    }
    
    const insertAt = (current) => {
      if (value < current.value) {
        if (!current.left) current.left = node;
        else insertAt(current.left);
      } else {
        if (!current.right) current.right = node;
        else insertAt(current.right);
      }
    };
    
    insertAt(this.root);
    return this;
  }
  
  // In-order traversal (left, root, right)
  [Symbol.iterator]() {
    const values = [];
    
    const inOrder = (node) => {
      if (!node) return;
      inOrder(node.left);
      values.push(node.value);
      inOrder(node.right);
    };
    
    inOrder(this.root);
    let index = 0;
    
    return {
      next() {
        if (index < values.length) {
          return { value: values[index++], done: false };
        }
        return { value: undefined, done: true };
      }
    };
  }
  
  // Generator-based traversal
  *preOrder() {
    function* traverse(node) {
      if (!node) return;
      yield node.value;
      yield* traverse(node.left);
      yield* traverse(node.right);
    }
    yield* traverse(this.root);
  }
  
  *postOrder() {
    function* traverse(node) {
      if (!node) return;
      yield* traverse(node.left);
      yield* traverse(node.right);
      yield node.value;
    }
    yield* traverse(this.root);
  }
  
  *levelOrder() {
    if (!this.root) return;
    
    const queue = [this.root];
    
    while (queue.length > 0) {
      const node = queue.shift();
      yield node.value;
      
      if (node.left) queue.push(node.left);
      if (node.right) queue.push(node.right);
    }
  }
}

const tree = new BinaryTree();
[5, 3, 7, 1, 4, 6, 8, 2].forEach(v => tree.insert(v));

console.log('In-order:', [...tree]);
console.log('Pre-order:', [...tree.preOrder()]);
console.log('Post-order:', [...tree.postOrder()]);
console.log('Level-order:', [...tree.levelOrder()]);

// ========================
// Infinite Iterator (Generator)
// ========================

function* fibonacci() {
  let [a, b] = [0, 1];
  while (true) {
    yield a;
    [a, b] = [b, a + b];
  }
}

function take(iterator, n) {
  const result = [];
  for (const value of iterator) {
    result.push(value);
    if (result.length >= n) break;
  }
  return result;
}

console.log('Fibonacci:', take(fibonacci(), 15));

// Generator สำหรับ pagination
function* paginate(data, pageSize) {
  let page = 0;
  while (page * pageSize < data.length) {
    yield data.slice(page * pageSize, (page + 1) * pageSize);
    page++;
  }
}

const items = Array.from({ length: 23 }, (_, i) => `Item ${i + 1}`);
const pages = paginate(items, 5);

for (const page of pages) {
  console.log('Page:', page.join(', '));
}
```

---

## Step 1037: Mediator Pattern

**Mediator Pattern** ให้ objects สื่อสารกันผ่าน mediator แทนที่จะสื่อสารตรง

```javascript
// ========================
// Mediator Pattern - Chat Room
// ========================

class ChatMediator {
  constructor(name) {
    this._name = name;
    this._users = new Map();
    this._messageHistory = [];
    this._bannedUsers = new Set();
  }
  
  addUser(user) {
    this._users.set(user.name, user);
    user.setMediator(this);
    this._broadcast(`${user.name} เข้าร่วมห้อง ${this._name}`, 'system', null);
    return this;
  }
  
  removeUser(username) {
    if (this._users.has(username)) {
      this._users.delete(username);
      this._broadcast(`${username} ออกจากห้อง`, 'system', null);
    }
    return this;
  }
  
  banUser(username) {
    this._bannedUsers.add(username);
    this.removeUser(username);
    this._broadcast(`${username} ถูก ban`, 'system', null);
  }
  
  send(message, fromUser, toUser = null) {
    if (this._bannedUsers.has(fromUser.name)) {
      fromUser.receive({ type: 'error', content: 'คุณถูก ban จากห้องนี้' });
      return;
    }
    
    const messageObj = {
      id: Date.now(),
      from: fromUser.name,
      to: toUser?.name || 'all',
      content: message,
      timestamp: new Date().toISOString(),
      type: toUser ? 'private' : 'public'
    };
    
    this._messageHistory.push(messageObj);
    
    if (toUser) {
      // Private message
      toUser.receive(messageObj);
      fromUser.receive({ ...messageObj, note: '(ส่งแล้ว)' });
    } else {
      // Broadcast
      this._broadcast(messageObj, 'message', fromUser.name);
    }
  }
  
  _broadcast(messageOrContent, type, excludeUser) {
    const message = typeof messageOrContent === 'string'
      ? { type, content: messageOrContent, from: 'System', timestamp: new Date().toISOString() }
      : messageOrContent;
    
    this._users.forEach((user, name) => {
      if (name !== excludeUser) {
        user.receive(message);
      }
    });
  }
  
  getHistory(limit = 50) {
    return this._messageHistory.slice(-limit);
  }
  
  getUsers() {
    return [...this._users.keys()];
  }
}

class ChatUser {
  constructor(name) {
    this.name = name;
    this._mediator = null;
    this._inbox = [];
  }
  
  setMediator(mediator) {
    this._mediator = mediator;
  }
  
  send(message, toUser = null) {
    if (!this._mediator) throw new Error('Not in any chat room');
    this._mediator.send(message, this, toUser);
  }
  
  receive(message) {
    this._inbox.push(message);
    if (message.type === 'system') {
      console.log(`[System] ${message.content}`);
    } else if (message.type === 'private') {
      console.log(`[Private] ${message.from} → ${message.to}: ${message.content}`);
    } else {
      console.log(`[${message.from}] ${message.content}`);
    }
  }
  
  getInbox() {
    return [...this._inbox];
  }
}

// ใช้งาน
const chatRoom = new ChatMediator('JavaScript Thailand');

const alice = new ChatUser('Alice');
const bob = new ChatUser('Bob');
const charlie = new ChatUser('Charlie');

chatRoom
  .addUser(alice)
  .addUser(bob)
  .addUser(charlie);

alice.send('สวัสดีทุกคน!');
bob.send('สวัสดีครับ Alice');
charlie.send('ยินดีที่ได้รู้จัก');

// Private message
alice.send('Bob ขอถามเรื่อง project หน่อยนะ', bob);

// ========================
// Mediator Pattern - Air Traffic Control
// ========================

class ATCMediator {
  constructor() {
    this._aircrafts = new Map();
    this._runways = new Set(['RWY-01', 'RWY-02', 'RWY-03']);
    this._inUseRunways = new Map();
  }
  
  register(aircraft) {
    this._aircrafts.set(aircraft.callsign, aircraft);
    aircraft.setATC(this);
    this._notify(`${aircraft.callsign} ลงทะเบียนแล้ว`);
  }
  
  requestLanding(aircraft) {
    const availableRunway = [...this._runways]
      .find(runway => !this._inUseRunways.has(runway));
    
    if (availableRunway) {
      this._inUseRunways.set(availableRunway, aircraft.callsign);
      aircraft.receive(`${aircraft.callsign}, cleared to land runway ${availableRunway}`);
      
      // แจ้ง aircraft อื่น
      this._aircrafts.forEach((ac) => {
        if (ac.callsign !== aircraft.callsign) {
          ac.receive(`Traffic alert: ${aircraft.callsign} landing on ${availableRunway}`);
        }
      });
      
      return availableRunway;
    } else {
      aircraft.receive(`${aircraft.callsign}, hold. All runways occupied.`);
      return null;
    }
  }
  
  clearRunway(runwayId, aircraft) {
    this._inUseRunways.delete(runwayId);
    this._notify(`${runwayId} cleared by ${aircraft.callsign}`);
  }
  
  _notify(message) {
    console.log(`[ATC] ${message}`);
  }
}

class Aircraft {
  constructor(callsign, type) {
    this.callsign = callsign;
    this.type = type;
    this._atc = null;
  }
  
  setATC(atc) {
    this._atc = atc;
  }
  
  requestLanding() {
    console.log(`${this.callsign}: Requesting landing clearance`);
    return this._atc.requestLanding(this);
  }
  
  completeLanding(runway) {
    console.log(`${this.callsign}: Landed on ${runway}`);
    this._atc.clearRunway(runway, this);
  }
  
  receive(message) {
    console.log(`${this.callsign} received: ${message}`);
  }
}

const atc = new ATCMediator();
const tg1234 = new Aircraft('TG1234', 'Boeing 777');
const bg5678 = new Aircraft('BG5678', 'Airbus A320');
const nk9012 = new Aircraft('NK9012', 'Boeing 737');

atc.register(tg1234);
atc.register(bg5678);
atc.register(nk9012);

const runway1 = tg1234.requestLanding();
const runway2 = bg5678.requestLanding();
const runway3 = nk9012.requestLanding();

if (runway1) tg1234.completeLanding(runway1);
const runway4 = nk9012.requestLanding(); // ลองใหม่
```

---

## Step 1038: State Pattern

**State Pattern** ให้ object เปลี่ยนพฤติกรรมตาม state ปัจจุบัน

```javascript
// ========================
// State Pattern - Traffic Light
// ========================

class TrafficLight {
  constructor() {
    this._states = {
      red: new RedState(this),
      yellow: new YellowState(this),
      green: new GreenState(this)
    };
    this._currentState = this._states.red;
    this._observers = [];
  }
  
  setState(stateName) {
    this._currentState = this._states[stateName];
    this._notifyObservers();
  }
  
  getCurrentState() {
    return this._currentState;
  }
  
  // Forward to current state
  change() {
    this._currentState.change();
  }
  
  pedestrianCross() {
    return this._currentState.pedestrianCross();
  }
  
  emergencyOverride() {
    this._currentState.emergencyOverride();
  }
  
  addObserver(observer) {
    this._observers.push(observer);
  }
  
  _notifyObservers() {
    this._observers.forEach(obs => obs.onStateChange(this._currentState));
  }
  
  getStatus() {
    return `Traffic Light: ${this._currentState.getName()} - ${this._currentState.getDescription()}`;
  }
}

class TrafficLightState {
  constructor(light) {
    this._light = light;
  }
  
  change() { throw new Error('Not implemented'); }
  pedestrianCross() { throw new Error('Not implemented'); }
  emergencyOverride() {
    this._light.setState('red');
    console.log('Emergency override! All red.');
  }
  getName() { return 'Unknown'; }
  getDescription() { return ''; }
  getColor() { return '#000'; }
  getDuration() { return 0; } // seconds
}

class RedState extends TrafficLightState {
  change() {
    console.log('Red → Green');
    this._light.setState('green');
  }
  
  pedestrianCross() {
    return { allowed: true, countdown: this.getDuration() };
  }
  
  getName() { return 'Red'; }
  getDescription() { return 'หยุดรถ'; }
  getColor() { return '#FF0000'; }
  getDuration() { return 30; }
}

class YellowState extends TrafficLightState {
  change() {
    console.log('Yellow → Red');
    this._light.setState('red');
  }
  
  pedestrianCross() {
    return { allowed: false, message: 'กำลังเปลี่ยนไฟ กรุณารอ' };
  }
  
  getName() { return 'Yellow'; }
  getDescription() { return 'เตรียมหยุด'; }
  getColor() { return '#FFFF00'; }
  getDuration() { return 5; }
}

class GreenState extends TrafficLightState {
  change() {
    console.log('Green → Yellow');
    this._light.setState('yellow');
  }
  
  pedestrianCross() {
    return { allowed: false, message: 'ห้ามข้ามถนน' };
  }
  
  getName() { return 'Green'; }
  getDescription() { return 'ไปได้'; }
  getColor() { return '#00FF00'; }
  getDuration() { return 45; }
}

// ใช้งาน
const trafficLight = new TrafficLight();

trafficLight.addObserver({
  onStateChange(state) {
    console.log(`🚦 Changed to ${state.getName()} (${state.getDuration()}s)`);
  }
});

console.log(trafficLight.getStatus());
console.log('Pedestrian:', trafficLight.pedestrianCross());

trafficLight.change(); // Red → Green
console.log('Pedestrian:', trafficLight.pedestrianCross());

trafficLight.change(); // Green → Yellow
trafficLight.change(); // Yellow → Red

trafficLight.emergencyOverride(); // Override to Red

// ========================
// Finite State Machine
// ========================

class FSM {
  constructor(initialState) {
    this._state = initialState;
    this._transitions = new Map();
    this._entryActions = new Map();
    this._exitActions = new Map();
    this._history = [];
  }
  
  addTransition(fromState, event, toState, action = null) {
    const key = `${fromState}:${event}`;
    this._transitions.set(key, { toState, action });
    return this;
  }
  
  onEnter(state, action) {
    this._entryActions.set(state, action);
    return this;
  }
  
  onExit(state, action) {
    this._exitActions.set(state, action);
    return this;
  }
  
  dispatch(event, data = null) {
    const key = `${this._state}:${event}`;
    const transition = this._transitions.get(key);
    
    if (!transition) {
      throw new Error(`No transition from '${this._state}' on '${event}'`);
    }
    
    const prevState = this._state;
    
    // Exit action
    const exitAction = this._exitActions.get(prevState);
    if (exitAction) exitAction(data);
    
    // Transition action
    if (transition.action) transition.action(data);
    
    // Change state
    this._state = transition.toState;
    this._history.push({ from: prevState, event, to: this._state, data });
    
    // Entry action
    const entryAction = this._entryActions.get(this._state);
    if (entryAction) entryAction(data);
    
    return this;
  }
  
  can(event) {
    return this._transitions.has(`${this._state}:${event}`);
  }
  
  getState() {
    return this._state;
  }
  
  getHistory() {
    return [...this._history];
  }
}

// Order State Machine
const orderFSM = new FSM('pending')
  .addTransition('pending', 'confirm', 'confirmed', (data) => {
    console.log('Order confirmed:', data?.orderId);
  })
  .addTransition('confirmed', 'ship', 'shipped', (data) => {
    console.log('Order shipped with tracking:', data?.tracking);
  })
  .addTransition('shipped', 'deliver', 'delivered', (data) => {
    console.log('Order delivered!');
  })
  .addTransition('delivered', 'review', 'completed')
  .addTransition('pending', 'cancel', 'cancelled')
  .addTransition('confirmed', 'cancel', 'cancelled')
  
  .onEnter('confirmed', () => console.log('[State] Now: CONFIRMED - Preparing order'))
  .onEnter('shipped', () => console.log('[State] Now: SHIPPED - On the way'))
  .onEnter('delivered', () => console.log('[State] Now: DELIVERED'))
  .onExit('pending', () => console.log('[Event] Left pending state'));

console.log('\n=== Order State Machine ===');
console.log('Current state:', orderFSM.getState());

orderFSM.dispatch('confirm', { orderId: 'ORD-001' });
orderFSM.dispatch('ship', { tracking: 'TH123456789' });
orderFSM.dispatch('deliver');

console.log('Can review?', orderFSM.can('review'));
orderFSM.dispatch('review');

console.log('Final state:', orderFSM.getState());
console.log('History:', orderFSM.getHistory().map(h => `${h.from} --[${h.event}]--> ${h.to}`));
```

---

## Step 1039: Chain of Responsibility Pattern

**Chain of Responsibility** ส่ง request ผ่านลำดับ handlers จนกว่าจะได้รับการจัดการ

```javascript
// ========================
// Chain of Responsibility
// ========================

class Handler {
  constructor(name) {
    this._name = name;
    this._next = null;
  }
  
  setNext(handler) {
    this._next = handler;
    return handler; // สำหรับ chaining
  }
  
  handle(request) {
    if (this._next) {
      return this._next.handle(request);
    }
    return null; // ไม่มีใครจัดการ
  }
  
  getName() { return this._name; }
}

// ========================
// Support System
// ========================

class SupportTicket {
  constructor(id, category, priority, description, userId) {
    this.id = id;
    this.category = category; // 'billing', 'technical', 'general'
    this.priority = priority; // 1-5 (5 = most urgent)
    this.description = description;
    this.userId = userId;
    this.resolvedBy = null;
    this.resolution = null;
  }
}

class Level1Support extends Handler {
  handle(ticket) {
    if (ticket.priority <= 2 && ticket.category === 'general') {
      ticket.resolvedBy = this.getName();
      ticket.resolution = 'Resolved by L1 support - FAQ answer provided';
      console.log(`[${this.getName()}] Resolved ticket #${ticket.id}`);
      return ticket;
    }
    
    console.log(`[${this.getName()}] Cannot handle #${ticket.id} (priority ${ticket.priority}, ${ticket.category}), escalating...`);
    return super.handle(ticket);
  }
}

class Level2Support extends Handler {
  handle(ticket) {
    if (ticket.priority <= 3 && ['general', 'billing'].includes(ticket.category)) {
      ticket.resolvedBy = this.getName();
      ticket.resolution = 'Resolved by L2 support - Account adjusted';
      console.log(`[${this.getName()}] Resolved ticket #${ticket.id}`);
      return ticket;
    }
    
    console.log(`[${this.getName()}] Cannot handle #${ticket.id}, escalating...`);
    return super.handle(ticket);
  }
}

class TechnicalSupport extends Handler {
  handle(ticket) {
    if (ticket.category === 'technical') {
      ticket.resolvedBy = this.getName();
      ticket.resolution = 'Resolved by Technical Support - Bug fix deployed';
      console.log(`[${this.getName()}] Resolved ticket #${ticket.id}`);
      return ticket;
    }
    
    console.log(`[${this.getName()}] Cannot handle #${ticket.id}, escalating...`);
    return super.handle(ticket);
  }
}

class ManagementSupport extends Handler {
  handle(ticket) {
    if (ticket.priority >= 4) {
      ticket.resolvedBy = this.getName();
      ticket.resolution = 'Resolved by Management - Priority resolution';
      console.log(`[${this.getName()}] Resolved URGENT ticket #${ticket.id}`);
      return ticket;
    }
    
    ticket.resolvedBy = this.getName();
    ticket.resolution = 'Resolved by Management - Final escalation';
    console.log(`[${this.getName()}] Resolved ticket #${ticket.id} (last resort)`);
    return ticket;
  }
}

// สร้าง chain
const l1 = new Level1Support('L1 Support');
const l2 = new Level2Support('L2 Support');
const tech = new TechnicalSupport('Technical Team');
const mgmt = new ManagementSupport('Management');

// เชื่อม chain: L1 → L2 → Tech → Management
l1.setNext(l2).setNext(tech).setNext(mgmt);

// ทดสอบ tickets
const tickets = [
  new SupportTicket(1, 'general', 1, 'How to change password?', 'user1'),
  new SupportTicket(2, 'billing', 2, 'Wrong charge on my account', 'user2'),
  new SupportTicket(3, 'technical', 3, 'App crashes on login', 'user3'),
  new SupportTicket(4, 'billing', 4, 'Urgent: Fraudulent charges', 'user4'),
  new SupportTicket(5, 'general', 5, 'CRITICAL: Data breach suspected', 'user5'),
];

tickets.forEach(ticket => {
  console.log(`\nProcessing ticket #${ticket.id} (${ticket.category}, priority ${ticket.priority})`);
  const result = l1.handle(ticket);
  if (result) {
    console.log(`  Resolution: ${result.resolution}`);
  }
});

// ========================
// Middleware-like Chain (Express pattern)
// ========================

class MiddlewareHandler {
  constructor(middleware) {
    this._middleware = middleware;
    this._next = null;
  }
  
  setNext(handler) {
    this._next = handler;
    return handler;
  }
  
  async handle(context) {
    const next = async () => {
      if (this._next) {
        return this._next.handle(context);
      }
    };
    
    return this._middleware(context, next);
  }
}

class Pipeline {
  constructor() {
    this._handlers = [];
    this._head = null;
    this._tail = null;
  }
  
  use(middleware) {
    const handler = new MiddlewareHandler(middleware);
    
    if (!this._head) {
      this._head = handler;
      this._tail = handler;
    } else {
      this._tail.setNext(handler);
      this._tail = handler;
    }
    
    return this;
  }
  
  async process(context) {
    if (!this._head) return context;
    return this._head.handle(context);
  }
}

// ใช้งาน
const pipeline = new Pipeline();

pipeline
  .use(async (ctx, next) => {
    console.log('1. Auth check');
    ctx.authenticated = true;
    await next();
    console.log('1. After handler');
  })
  .use(async (ctx, next) => {
    console.log('2. Rate limit check');
    ctx.rateOk = true;
    await next();
  })
  .use(async (ctx, next) => {
    console.log('3. Validate input');
    ctx.valid = ctx.data && ctx.data.length > 0;
    if (!ctx.valid) {
      ctx.error = 'Invalid input';
      return;
    }
    await next();
  })
  .use(async (ctx, next) => {
    console.log('4. Process request');
    ctx.result = `Processed: ${ctx.data}`;
  });

async function runPipeline() {
  const context = { data: 'Hello World' };
  await pipeline.process(context);
  console.log('Final context:', context);
}

runPipeline();
```

---

## Step 1040: Template Method Pattern

**Template Method** กำหนดโครงสร้างของ algorithm ใน base class แต่ให้ subclass override ขั้นตอนเฉพาะ

```javascript
// ========================
// Template Method Pattern
// ========================

class DataProcessor {
  // Template method - โครงสร้างหลักไม่เปลี่ยน
  process(data) {
    const validated = this.validate(data);
    if (!validated) {
      throw new Error('Validation failed');
    }
    
    const parsed = this.parse(data);
    const transformed = this.transform(parsed);
    const result = this.format(transformed);
    
    this.afterProcess(result);
    return result;
  }
  
  // Steps ที่ subclass ต้อง implement
  validate(data) { throw new Error('Not implemented'); }
  parse(data) { throw new Error('Not implemented'); }
  transform(data) { throw new Error('Not implemented'); }
  format(data) { throw new Error('Not implemented'); }
  
  // Hook method - optional override
  afterProcess(result) {
    console.log('Processing complete');
  }
}

class CSVProcessor extends DataProcessor {
  validate(data) {
    return typeof data === 'string' && data.includes(',');
  }
  
  parse(data) {
    const lines = data.trim().split('\n');
    const headers = lines[0].split(',').map(h => h.trim());
    
    return lines.slice(1).map(line => {
      const values = line.split(',');
      return headers.reduce((obj, header, i) => {
        obj[header] = values[i]?.trim() || '';
        return obj;
      }, {});
    });
  }
  
  transform(data) {
    return data.map(row => ({
      ...row,
      // แปลง numeric strings
      ...Object.entries(row).reduce((acc, [key, value]) => {
        const num = parseFloat(value);
        if (!isNaN(num)) acc[key] = num;
        return acc;
      }, {})
    }));
  }
  
  format(data) {
    return {
      type: 'csv',
      count: data.length,
      data
    };
  }
  
  afterProcess(result) {
    console.log(`CSV processed: ${result.count} records`);
  }
}

class JSONProcessor extends DataProcessor {
  validate(data) {
    try {
      JSON.parse(data);
      return true;
    } catch {
      return false;
    }
  }
  
  parse(data) {
    return JSON.parse(data);
  }
  
  transform(data) {
    // Normalize nested objects
    const normalize = (obj, prefix = '') => {
      if (typeof obj !== 'object' || obj === null) return { [prefix]: obj };
      
      return Object.entries(obj).reduce((acc, [key, value]) => {
        const fullKey = prefix ? `${prefix}.${key}` : key;
        if (typeof value === 'object' && !Array.isArray(value)) {
          Object.assign(acc, normalize(value, fullKey));
        } else {
          acc[fullKey] = value;
        }
        return acc;
      }, {});
    };
    
    return Array.isArray(data) ? data.map(item => normalize(item)) : normalize(data);
  }
  
  format(data) {
    return {
      type: 'json',
      normalized: true,
      data
    };
  }
}

class XMLProcessor extends DataProcessor {
  validate(data) {
    return typeof data === 'string' && data.startsWith('<') && data.endsWith('>');
  }
  
  parse(data) {
    // Simple XML parser (จำลอง)
    const result = {};
    const matches = data.matchAll(/<(\w+)[^>]*>(.*?)<\/\1>/gs);
    
    for (const [, tag, content] of matches) {
      result[tag] = content.trim();
    }
    
    return result;
  }
  
  transform(data) {
    return Object.entries(data).reduce((acc, [key, value]) => {
      acc[key] = value.replace(/&lt;/g, '<').replace(/&gt;/g, '>').replace(/&amp;/g, '&');
      return acc;
    }, {});
  }
  
  format(data) {
    return {
      type: 'xml',
      data
    };
  }
}

// ใช้งาน
const csvData = `name,age,email
สมชาย,25,somchai@example.com
สมหญิง,30,somying@example.com
วิชัย,22,wichai@example.com`;

const jsonData = JSON.stringify({
  user: { name: 'สมชาย', address: { city: 'Bangkok', country: 'Thailand' } },
  role: 'admin'
});

const xmlData = `<user><name>สมชาย</name><email>somchai@example.com</email></user>`;

const csvProcessor = new CSVProcessor();
const jsonProcessor = new JSONProcessor();
const xmlProcessor = new XMLProcessor();

console.log('CSV Result:', csvProcessor.process(csvData));
console.log('JSON Result:', jsonProcessor.process(jsonData));
console.log('XML Result:', xmlProcessor.process(xmlData));
```

---

## Step 1041: Memento Pattern

**Memento Pattern** บันทึกและกู้คืน state ของ object

```javascript
// ========================
// Memento Pattern
// ========================

// Memento - เก็บ state
class EditorMemento {
  constructor(content, cursorPosition, selection, timestamp) {
    this._content = content;
    this._cursorPosition = cursorPosition;
    this._selection = selection;
    this._timestamp = timestamp || new Date().toISOString();
    Object.freeze(this); // immutable
  }
  
  getContent() { return this._content; }
  getCursorPosition() { return this._cursorPosition; }
  getSelection() { return this._selection ? { ...this._selection } : null; }
  getTimestamp() { return this._timestamp; }
  
  toString() {
    return `Snapshot at ${this._timestamp}: "${this._content.substring(0, 30)}..."`;
  }
}

// Originator
class RichTextEditor {
  constructor() {
    this._content = '';
    this._cursorPosition = 0;
    this._selection = null;
    this._formatting = {};
  }
  
  type(text) {
    this._content = this._content.slice(0, this._cursorPosition) 
                  + text 
                  + this._content.slice(this._cursorPosition);
    this._cursorPosition += text.length;
    return this;
  }
  
  delete(chars = 1) {
    if (this._cursorPosition > 0) {
      this._content = this._content.slice(0, this._cursorPosition - chars)
                    + this._content.slice(this._cursorPosition);
      this._cursorPosition -= chars;
    }
    return this;
  }
  
  moveCursor(position) {
    this._cursorPosition = Math.max(0, Math.min(position, this._content.length));
    return this;
  }
  
  select(start, end) {
    this._selection = { start, end };
    return this;
  }
  
  clearSelection() {
    this._selection = null;
    return this;
  }
  
  getContent() {
    return this._content;
  }
  
  getCursor() {
    return this._cursorPosition;
  }
  
  // สร้าง snapshot
  save() {
    return new EditorMemento(
      this._content,
      this._cursorPosition,
      this._selection ? { ...this._selection } : null
    );
  }
  
  // กู้คืน snapshot
  restore(memento) {
    this._content = memento.getContent();
    this._cursorPosition = memento.getCursorPosition();
    this._selection = memento.getSelection();
    return this;
  }
}

// Caretaker - จัดการ mementos
class EditorHistory {
  constructor(maxHistory = 50) {
    this._mementos = [];
    this._currentIndex = -1;
    this._maxHistory = maxHistory;
  }
  
  save(editor) {
    // ลบ future history หลัง current position
    this._mementos = this._mementos.slice(0, this._currentIndex + 1);
    
    // บันทึก memento
    this._mementos.push(editor.save());
    this._currentIndex++;
    
    // จำกัด history size
    if (this._mementos.length > this._maxHistory) {
      this._mementos.shift();
      this._currentIndex--;
    }
    
    return this;
  }
  
  undo(editor) {
    if (this._currentIndex <= 0) {
      console.log('Nothing to undo');
      return false;
    }
    
    this._currentIndex--;
    editor.restore(this._mementos[this._currentIndex]);
    return true;
  }
  
  redo(editor) {
    if (this._currentIndex >= this._mementos.length - 1) {
      console.log('Nothing to redo');
      return false;
    }
    
    this._currentIndex++;
    editor.restore(this._mementos[this._currentIndex]);
    return true;
  }
  
  canUndo() { return this._currentIndex > 0; }
  canRedo() { return this._currentIndex < this._mementos.length - 1; }
  
  getHistory() {
    return this._mementos.map((m, i) => ({
      index: i,
      current: i === this._currentIndex,
      snapshot: m.toString()
    }));
  }
}

// ใช้งาน
const textEditor = new RichTextEditor();
const history = new EditorHistory();

// เริ่มต้น
history.save(textEditor);

textEditor.type('สวัสดีชาวโลก');
history.save(textEditor);

textEditor.type(' นี่คือ JavaScript');
history.save(textEditor);

textEditor.type(' Pattern!');
history.save(textEditor);

console.log('Current:', textEditor.getContent());

history.undo(textEditor);
console.log('After undo 1:', textEditor.getContent());

history.undo(textEditor);
console.log('After undo 2:', textEditor.getContent());

history.redo(textEditor);
console.log('After redo:', textEditor.getContent());

textEditor.type(' (แก้ไขใหม่)');
history.save(textEditor);
console.log('After new edit:', textEditor.getContent());

console.log('\nHistory:');
history.getHistory().forEach(h => {
  const marker = h.current ? '→' : ' ';
  console.log(`${marker} [${h.index}] ${h.snapshot}`);
});
```

---

## Step 1042: Visitor Pattern

**Visitor Pattern** เพิ่ม operation ใหม่ให้ objects โดยไม่ต้องแก้ classes ของพวกมัน

```javascript
// ========================
// Visitor Pattern
// ========================

// Visitor Interface
class FileSystemVisitor {
  visitFile(file) { throw new Error('Not implemented'); }
  visitDirectory(directory) { throw new Error('Not implemented'); }
}

// File system nodes
class FileNode {
  constructor(name, size, extension) {
    this.name = name;
    this.size = size;
    this.extension = extension;
  }
  
  accept(visitor) {
    return visitor.visitFile(this);
  }
}

class DirectoryNode {
  constructor(name) {
    this.name = name;
    this._children = [];
  }
  
  addChild(node) {
    this._children.push(node);
    return this;
  }
  
  getChildren() {
    return this._children;
  }
  
  accept(visitor) {
    return visitor.visitDirectory(this);
  }
}

// Concrete Visitors
class SizeCalculator extends FileSystemVisitor {
  constructor() {
    super();
    this._totalSize = 0;
  }
  
  visitFile(file) {
    this._totalSize += file.size;
    return file.size;
  }
  
  visitDirectory(directory) {
    let dirSize = 0;
    directory.getChildren().forEach(child => {
      dirSize += child.accept(this);
    });
    return dirSize;
  }
  
  getTotalSize() { return this._totalSize; }
}

class FileCounter extends FileSystemVisitor {
  constructor() {
    super();
    this._counts = {};
  }
  
  visitFile(file) {
    const ext = file.extension || 'no-ext';
    this._counts[ext] = (this._counts[ext] || 0) + 1;
    return 1;
  }
  
  visitDirectory(directory) {
    let count = 0;
    directory.getChildren().forEach(child => {
      count += child.accept(this);
    });
    return count;
  }
  
  getCounts() { return { ...this._counts }; }
}

class SecurityAuditor extends FileSystemVisitor {
  constructor() {
    super();
    this._issues = [];
  }
  
  visitFile(file) {
    // ตรวจสอบ sensitive files
    const sensitivePatterns = ['.env', '.key', 'password', 'secret', 'private'];
    if (sensitivePatterns.some(p => file.name.toLowerCase().includes(p))) {
      this._issues.push({
        type: 'sensitive_file',
        severity: 'HIGH',
        path: file.name,
        message: `Sensitive file detected: ${file.name}`
      });
    }
    
    // ตรวจสอบขนาดไฟล์
    if (file.size > 100 * 1024 * 1024) { // > 100MB
      this._issues.push({
        type: 'large_file',
        severity: 'MEDIUM',
        path: file.name,
        message: `Large file: ${file.name} (${Math.round(file.size / 1024 / 1024)}MB)`
      });
    }
    
    return 0;
  }
  
  visitDirectory(directory) {
    directory.getChildren().forEach(child => child.accept(this));
    return 0;
  }
  
  getIssues() { return [...this._issues]; }
}

class SearchVisitor extends FileSystemVisitor {
  constructor(query) {
    super();
    this._query = query.toLowerCase();
    this._results = [];
    this._currentPath = [];
  }
  
  visitFile(file) {
    if (file.name.toLowerCase().includes(this._query)) {
      this._results.push({
        type: 'file',
        name: file.name,
        path: [...this._currentPath, file.name].join('/'),
        size: file.size
      });
    }
    return null;
  }
  
  visitDirectory(directory) {
    if (directory.name.toLowerCase().includes(this._query)) {
      this._results.push({
        type: 'directory',
        name: directory.name,
        path: [...this._currentPath, directory.name].join('/')
      });
    }
    
    this._currentPath.push(directory.name);
    directory.getChildren().forEach(child => child.accept(this));
    this._currentPath.pop();
    
    return null;
  }
  
  getResults() { return [...this._results]; }
}

// สร้าง file system
const root = new DirectoryNode('project');
const src = new DirectoryNode('src');
const config = new DirectoryNode('config');

src.addChild(new FileNode('index.js', 5000, 'js'))
   .addChild(new FileNode('utils.js', 3000, 'js'))
   .addChild(new FileNode('password-utils.js', 1500, 'js')); // sensitive!

config.addChild(new FileNode('.env', 500, 'env')) // sensitive!
      .addChild(new FileNode('database.config.js', 800, 'js'))
      .addChild(new FileNode('app.config.js', 600, 'js'));

root.addChild(src)
    .addChild(config)
    .addChild(new FileNode('package.json', 1200, 'json'))
    .addChild(new FileNode('README.md', 5000, 'md'));

// ใช้ visitors
const sizeCalc = new SizeCalculator();
root.accept(sizeCalc);
console.log('Total size:', sizeCalc.getTotalSize(), 'bytes');

const fileCounter = new FileCounter();
root.accept(fileCounter);
console.log('File counts:', fileCounter.getCounts());

const auditor = new SecurityAuditor();
root.accept(auditor);
console.log('Security issues:', auditor.getIssues());

const searcher = new SearchVisitor('config');
root.accept(searcher);
console.log('Search results:', searcher.getResults());
```

---

## Step 1043-1050: รวมทุก Behavioral Patterns

```javascript
// ========================
// Task Management System
// ========================

// Observer: TaskEventBus
class TaskEventBus extends EventEmitter {
  static _instance = null;
  
  constructor() {
    if (TaskEventBus._instance) return TaskEventBus._instance;
    super();
    TaskEventBus._instance = this;
  }
  
  static getInstance() {
    if (!TaskEventBus._instance) new TaskEventBus();
    return TaskEventBus._instance;
  }
}

// State: TaskState FSM
class TaskStateMachine {
  constructor() {
    this._state = 'todo';
    this._transitions = {
      'todo:start': 'in_progress',
      'in_progress:pause': 'paused',
      'paused:resume': 'in_progress',
      'in_progress:complete': 'done',
      'in_progress:block': 'blocked',
      'blocked:unblock': 'in_progress',
      'todo:cancel': 'cancelled',
      'in_progress:cancel': 'cancelled',
      'paused:cancel': 'cancelled'
    };
  }
  
  can(action) {
    return Boolean(this._transitions[`${this._state}:${action}`]);
  }
  
  transition(action) {
    const key = `${this._state}:${action}`;
    if (!this._transitions[key]) {
      throw new Error(`Cannot '${action}' from state '${this._state}'`);
    }
    const prev = this._state;
    this._state = this._transitions[key];
    return { from: prev, to: this._state, action };
  }
  
  getState() { return this._state; }
}

// Command: TaskCommands
class TaskCommand {
  execute(task) { throw new Error('Not implemented'); }
  undo(task) { throw new Error('Not implemented'); }
}

class AssignTaskCommand extends TaskCommand {
  constructor(assignee) {
    super();
    this._assignee = assignee;
    this._prevAssignee = null;
  }
  
  execute(task) {
    this._prevAssignee = task.assignee;
    task.assignee = this._assignee;
    TaskEventBus.getInstance().emit('task:assigned', { task, assignee: this._assignee });
  }
  
  undo(task) {
    task.assignee = this._prevAssignee;
  }
}

class ChangePriorityCommand extends TaskCommand {
  constructor(priority) {
    super();
    this._priority = priority;
    this._prevPriority = null;
  }
  
  execute(task) {
    this._prevPriority = task.priority;
    task.priority = this._priority;
    TaskEventBus.getInstance().emit('task:priority_changed', { task, priority: this._priority });
  }
  
  undo(task) {
    task.priority = this._prevPriority;
  }
}

// Strategy: Priority Calculation
class PriorityStrategy {
  calculate(task) { throw new Error('Not implemented'); }
}

class SimplePriority extends PriorityStrategy {
  calculate(task) {
    const weights = { critical: 100, high: 75, medium: 50, low: 25 };
    return weights[task.priority] || 0;
  }
}

class DueDatePriority extends PriorityStrategy {
  calculate(task) {
    if (!task.dueDate) return 50;
    
    const now = Date.now();
    const due = new Date(task.dueDate).getTime();
    const hoursLeft = (due - now) / (1000 * 3600);
    
    if (hoursLeft < 0) return 200; // overdue
    if (hoursLeft < 24) return 150;
    if (hoursLeft < 72) return 100;
    if (hoursLeft < 168) return 75; // within a week
    return 50;
  }
}

// Main Task class
class Task {
  constructor(data) {
    this.id = data.id || `TASK-${Date.now()}`;
    this.title = data.title;
    this.description = data.description || '';
    this.assignee = data.assignee || null;
    this.priority = data.priority || 'medium';
    this.dueDate = data.dueDate || null;
    this.tags = data.tags || [];
    this.comments = [];
    
    this._stateMachine = new TaskStateMachine();
    this._commandHistory = [];
    this._priorityStrategy = new SimplePriority();
  }
  
  getState() {
    return this._stateMachine.getState();
  }
  
  do(action) {
    const transition = this._stateMachine.transition(action);
    TaskEventBus.getInstance().emit(`task:${action}ed`, { task: this, transition });
    return this;
  }
  
  can(action) {
    return this._stateMachine.can(action);
  }
  
  execute(command) {
    command.execute(this);
    this._commandHistory.push(command);
    return this;
  }
  
  undo() {
    const command = this._commandHistory.pop();
    if (command) command.undo(this);
    return this;
  }
  
  setPriorityStrategy(strategy) {
    this._priorityStrategy = strategy;
    return this;
  }
  
  getScore() {
    return this._priorityStrategy.calculate(this);
  }
  
  addComment(author, text) {
    this.comments.push({ author, text, timestamp: new Date().toISOString() });
    TaskEventBus.getInstance().emit('task:commented', { task: this, author, text });
    return this;
  }
  
  toString() {
    return `[${this.getState().toUpperCase()}] ${this.id}: ${this.title} (score: ${this.getScore()})`;
  }
}

// ใช้งาน
const bus = TaskEventBus.getInstance();

// Subscribe to events
bus.on('task:started', ({ task }) => console.log(`Task started: ${task.title}`));
bus.on('task:completed', ({ task }) => console.log(`Task completed: ${task.title} 🎉`));
bus.on('task:assigned', ({ task, assignee }) => console.log(`Task ${task.id} assigned to ${assignee}`));

// สร้าง tasks
const task1 = new Task({
  title: 'Implement Observer Pattern',
  priority: 'high',
  dueDate: new Date(Date.now() + 3600000 * 24).toISOString()
});

const task2 = new Task({
  title: 'Write unit tests',
  priority: 'medium'
});

// Commands
task1.execute(new AssignTaskCommand('สมชาย'));
task1.execute(new ChangePriorityCommand('critical'));

// State transitions
task1.do('start');
task1.addComment('สมชาย', 'เริ่มทำแล้ว');
task1.do('complete');

console.log(task1.toString());

// Strategy
task2.setPriorityStrategy(new DueDatePriority());
console.log('Task2 score (DueDatePriority):', task2.getScore());

task2.do('start');
task2.do('block');
console.log(task2.toString());

// Undo
task1.undo(); // undo priority change
console.log('After undo priority:', task1.priority);
```

---

## แบบฝึกหัด (Exercises)

### ระดับ Easy

**Exercise 1:** Observer Pattern
```javascript
// สร้าง Stock Price Monitor:
// - StockTicker: emit events เมื่อราคาเปลี่ยน
// - PriceAlert: แจ้งเตือนเมื่อราคาสูงกว่า/ต่ำกว่า threshold
// - Portfolio: อัพเดท value อัตโนมัติ

class StockTicker extends EventEmitter {
  // emit('priceChange', { symbol, price, change })
}

class PriceAlert {
  constructor(symbol, threshold, type) { // type: 'above' | 'below'
    // Subscribe to StockTicker
    // Alert เมื่อราคา pass threshold
  }
}
```

**Exercise 2:** Strategy Pattern
```javascript
// สร้าง Discount Calculator:
// - NoDiscount: ไม่ลด
// - PercentDiscount: ลด %
// - FixedDiscount: ลดจำนวนคงที่
// - TieredDiscount: ลดตาม tier (buy more, save more)
// - BuyXGetYFree: ซื้อ X ได้ Y ฟรี

class DiscountStrategy {
  apply(price, quantity) {}
}

class ShoppingCart {
  setDiscountStrategy(strategy) {}
  calculateTotal() {}
}
```

**Exercise 3:** Command Pattern  
```javascript
// สร้าง Smart Home controller:
// - TurnOnLight(room)
// - TurnOffLight(room)
// - SetTemperature(degrees)
// - LockDoor(doorId)
// - MacroCommand 'Leave Home': lock doors + turn off lights
// - undo/redo support

class SmartHomeController {
  // execute(command)
  // undo()
  // redo()
  // createMacro(name, ...commands)
}
```

### ระดับ Hard

**Exercise 4:** State Machine
```javascript
// สร้าง Vending Machine ด้วย State Pattern:
// States: idle, has_money, dispensing, out_of_stock
// Actions: insert_coin, select_product, dispense, return_change
// 
// Rules:
// - ใส่เหรียญก่อนเลือกสินค้า
// - ถ้าเงินไม่พอ ต้องใส่เพิ่ม
// - ถ้าสินค้าหมด บอก out_of_stock
// - dispense แล้วคืนเงินทอน

class VendingMachine {
  // state-driven behavior
}
```

**Exercise 5:** Observer + Command + Memento
```javascript
// สร้าง Collaborative Document Editor:
// - Multiple users สามารถ edit พร้อมกัน
// - Changes broadcast ผ่าน Observer
// - ทุก edit เป็น Command (undo/redo per user)
// - Auto-save ด้วย Memento ทุก 30 seconds

class CollaborativeEditor {
  addUser(userId) {}
  removeUser(userId) {}
  applyChange(userId, change) {} // broadcasts to all
  undoForUser(userId) {}
  redoForUser(userId) {}
  autoSave() {}
  restore(snapshotId) {}
}
```

---

## สรุป Behavioral Patterns

```javascript
// Quick Reference

// Observer
class Subject {
  subscribe(listener) {}
  unsubscribe(listener) {}
  notify(data) {}
}

// Strategy
class Context {
  setStrategy(strategy) {}
  executeStrategy(...args) { return this._strategy.execute(...args); }
}

// Command
class Invoker {
  execute(command) { command.execute(); this._history.push(command); }
  undo() { this._history.pop().undo(); }
}

// State
class Context2 {
  setState(state) { this._state = state; }
  request() { this._state.handle(this); }
}

// Chain of Responsibility
class Handler2 {
  setNext(handler) { this._next = handler; return handler; }
  handle(request) { return this._next?.handle(request); }
}

// Template Method
class AbstractAlgorithm {
  run() { this.step1(); this.step2(); this.step3(); } // อย่าแก้!
  step1() { throw new Error('override me'); }
  step2() { throw new Error('override me'); }
  step3() {} // optional hook
}
```

---

**ขั้นตอนต่อไป:** ใน Part 54 เราจะเรียนรู้ **Functional Programming** ซึ่งเป็นแนวคิดการเขียนโปรแกรมที่แตกต่างออกไป
