# Part 20: โปรเจค - เว็บแอพพลิเคชันพื้นฐาน
## Steps 371-390

---

## บทนำ

ในบทสุดท้ายนี้เราจะสร้างโปรเจคที่สมบูรณ์ 5 โปรเจค ได้แก่ Todo List App, Calculator App, Quiz App, Weather App และ Note Taking App ทุกโปรเจคมีโค้ดที่ใช้งานได้จริงพร้อม HTML, CSS และ JavaScript ครบ

---

## Step 371-374: โปรเจค 1 - Todo List App

```html
<!DOCTYPE html>
<html lang="th">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>รายการสิ่งที่ต้องทำ</title>
  <style>
    * {
      box-sizing: border-box;
      margin: 0;
      padding: 0;
    }

    body {
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
      min-height: 100vh;
      display: flex;
      align-items: center;
      justify-content: center;
      padding: 20px;
    }

    .app {
      background: white;
      border-radius: 16px;
      box-shadow: 0 20px 60px rgba(0, 0, 0, 0.3);
      width: 100%;
      max-width: 480px;
      overflow: hidden;
    }

    .app-header {
      background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
      color: white;
      padding: 24px;
      text-align: center;
    }

    .app-header h1 {
      font-size: 28px;
      font-weight: 700;
      margin-bottom: 4px;
    }

    .app-header p {
      opacity: 0.85;
      font-size: 14px;
    }

    .stats {
      display: flex;
      gap: 16px;
      justify-content: center;
      margin-top: 16px;
    }

    .stat {
      background: rgba(255,255,255,0.2);
      border-radius: 8px;
      padding: 8px 16px;
      text-align: center;
    }

    .stat-number {
      display: block;
      font-size: 24px;
      font-weight: 700;
    }

    .stat-label {
      font-size: 11px;
      opacity: 0.85;
    }

    .input-section {
      padding: 20px;
      border-bottom: 1px solid #eee;
    }

    .input-group {
      display: flex;
      gap: 8px;
    }

    .todo-input {
      flex: 1;
      padding: 12px 16px;
      border: 2px solid #e0e0e0;
      border-radius: 8px;
      font-size: 15px;
      transition: border-color 0.2s;
      outline: none;
    }

    .todo-input:focus {
      border-color: #667eea;
    }

    .add-btn {
      padding: 12px 20px;
      background: linear-gradient(135deg, #667eea, #764ba2);
      color: white;
      border: none;
      border-radius: 8px;
      cursor: pointer;
      font-size: 20px;
      font-weight: bold;
      transition: transform 0.1s, opacity 0.2s;
    }

    .add-btn:hover {
      opacity: 0.9;
      transform: scale(1.05);
    }

    .filters {
      display: flex;
      padding: 12px 20px;
      gap: 8px;
      border-bottom: 1px solid #eee;
      background: #f8f9fa;
    }

    .filter-btn {
      padding: 6px 14px;
      border: none;
      border-radius: 20px;
      cursor: pointer;
      font-size: 13px;
      background: transparent;
      color: #666;
      transition: all 0.2s;
    }

    .filter-btn.active {
      background: #667eea;
      color: white;
    }

    .todo-list {
      max-height: 400px;
      overflow-y: auto;
      padding: 12px;
    }

    .todo-item {
      display: flex;
      align-items: center;
      gap: 12px;
      padding: 12px;
      margin-bottom: 8px;
      background: #f8f9fa;
      border-radius: 10px;
      transition: all 0.2s;
      animation: slideIn 0.3s ease;
    }

    @keyframes slideIn {
      from { opacity: 0; transform: translateY(-10px); }
      to { opacity: 1; transform: translateY(0); }
    }

    .todo-item:hover {
      background: #f0f0f0;
    }

    .todo-item.completed .todo-text {
      text-decoration: line-through;
      color: #aaa;
    }

    .todo-checkbox {
      width: 22px;
      height: 22px;
      cursor: pointer;
      accent-color: #667eea;
      flex-shrink: 0;
    }

    .todo-text {
      flex: 1;
      font-size: 15px;
      color: #333;
    }

    .todo-text.editing {
      border: none;
      background: transparent;
      outline: none;
      border-bottom: 2px solid #667eea;
      width: 100%;
    }

    .todo-date {
      font-size: 11px;
      color: #aaa;
      white-space: nowrap;
    }

    .todo-actions {
      display: flex;
      gap: 4px;
    }

    .btn-icon {
      width: 30px;
      height: 30px;
      border: none;
      border-radius: 6px;
      cursor: pointer;
      font-size: 14px;
      display: flex;
      align-items: center;
      justify-content: center;
      transition: all 0.2s;
    }

    .btn-edit { background: #e3f2fd; color: #1976d2; }
    .btn-delete { background: #ffebee; color: #c62828; }
    .btn-edit:hover { background: #1976d2; color: white; }
    .btn-delete:hover { background: #c62828; color: white; }

    .empty-state {
      text-align: center;
      padding: 40px;
      color: #aaa;
    }

    .empty-state .emoji {
      font-size: 48px;
      display: block;
      margin-bottom: 12px;
    }

    .app-footer {
      padding: 16px 20px;
      border-top: 1px solid #eee;
      display: flex;
      justify-content: space-between;
      align-items: center;
      background: #f8f9fa;
    }

    .footer-info {
      font-size: 13px;
      color: #888;
    }

    .clear-completed {
      padding: 6px 12px;
      border: none;
      background: #ffebee;
      color: #c62828;
      border-radius: 6px;
      cursor: pointer;
      font-size: 13px;
    }

    .clear-completed:hover {
      background: #c62828;
      color: white;
    }
  </style>
</head>
<body>
  <div class="app">
    <div class="app-header">
      <h1>✅ รายการสิ่งที่ต้องทำ</h1>
      <p>จัดการงานของคุณได้อย่างมีประสิทธิภาพ</p>
      <div class="stats">
        <div class="stat">
          <span class="stat-number" id="totalCount">0</span>
          <span class="stat-label">ทั้งหมด</span>
        </div>
        <div class="stat">
          <span class="stat-number" id="doneCount">0</span>
          <span class="stat-label">เสร็จแล้ว</span>
        </div>
        <div class="stat">
          <span class="stat-number" id="pendingCount">0</span>
          <span class="stat-label">รอดำเนินการ</span>
        </div>
      </div>
    </div>

    <div class="input-section">
      <div class="input-group">
        <input
          type="text"
          id="todoInput"
          class="todo-input"
          placeholder="เพิ่มรายการใหม่... (กด Enter)"
          maxlength="100"
        />
        <button class="add-btn" onclick="addTodo()">+</button>
      </div>
    </div>

    <div class="filters">
      <button class="filter-btn active" data-filter="all" onclick="setFilter('all')">ทั้งหมด</button>
      <button class="filter-btn" data-filter="pending" onclick="setFilter('pending')">รอดำเนินการ</button>
      <button class="filter-btn" data-filter="completed" onclick="setFilter('completed')">เสร็จแล้ว</button>
    </div>

    <div class="todo-list" id="todoList"></div>

    <div class="app-footer">
      <span class="footer-info" id="footerInfo"></span>
      <button class="clear-completed" onclick="clearCompleted()">ลบที่เสร็จแล้ว</button>
    </div>
  </div>

  <script>
    // ========================================
    // STATE MANAGEMENT
    // ========================================

    let todos = [];
    let currentFilter = 'all';
    const STORAGE_KEY = 'todo_app_v1';

    // โหลดข้อมูลจาก localStorage
    function loadTodos() {
      try {
        const saved = localStorage.getItem(STORAGE_KEY);
        todos = saved ? JSON.parse(saved) : getDefaultTodos();
      } catch {
        todos = getDefaultTodos();
      }
    }

    // ตัวอย่างข้อมูลเริ่มต้น
    function getDefaultTodos() {
      return [
        { id: 1, text: 'เรียน JavaScript', completed: true, createdAt: Date.now() - 86400000 },
        { id: 2, text: 'สร้างโปรเจค Todo App', completed: true, createdAt: Date.now() - 3600000 },
        { id: 3, text: 'ฝึก CSS และ HTML', completed: false, createdAt: Date.now() - 1800000 },
        { id: 4, text: 'อ่านเอกสาร MDN', completed: false, createdAt: Date.now() },
      ];
    }

    // บันทึกลง localStorage
    function saveTodos() {
      localStorage.setItem(STORAGE_KEY, JSON.stringify(todos));
    }

    // ========================================
    // CRUD OPERATIONS
    // ========================================

    function addTodo() {
      const input = document.getElementById('todoInput');
      const text = input.value.trim();

      if (!text) {
        input.focus();
        input.style.borderColor = '#e74c3c';
        setTimeout(() => input.style.borderColor = '', 1000);
        return;
      }

      const todo = {
        id: Date.now(),
        text,
        completed: false,
        createdAt: Date.now()
      };

      todos.unshift(todo);
      input.value = '';
      input.focus();

      saveTodos();
      render();
    }

    function toggleTodo(id) {
      const todo = todos.find(t => t.id === id);
      if (todo) {
        todo.completed = !todo.completed;
        todo.completedAt = todo.completed ? Date.now() : null;
        saveTodos();
        render();
      }
    }

    function deleteTodo(id) {
      todos = todos.filter(t => t.id !== id);
      saveTodos();
      render();
    }

    function editTodo(id) {
      const todo = todos.find(t => t.id === id);
      if (!todo) return;

      const newText = prompt('แก้ไขรายการ:', todo.text);
      if (newText !== null && newText.trim()) {
        todo.text = newText.trim();
        saveTodos();
        render();
      }
    }

    function clearCompleted() {
      const completedCount = todos.filter(t => t.completed).length;
      if (completedCount === 0) return;

      if (confirm(`ต้องการลบรายการที่เสร็จแล้ว ${completedCount} รายการหรือไม่?`)) {
        todos = todos.filter(t => !t.completed);
        saveTodos();
        render();
      }
    }

    // ========================================
    // FILTERING
    // ========================================

    function setFilter(filter) {
      currentFilter = filter;

      // Update active button
      document.querySelectorAll('.filter-btn').forEach(btn => {
        btn.classList.toggle('active', btn.dataset.filter === filter);
      });

      render();
    }

    function getFilteredTodos() {
      switch (currentFilter) {
        case 'pending':   return todos.filter(t => !t.completed);
        case 'completed': return todos.filter(t => t.completed);
        default:          return todos;
      }
    }

    // ========================================
    // RENDERING
    // ========================================

    function formatDate(timestamp) {
      const diff = Date.now() - timestamp;
      const minutes = Math.floor(diff / 60000);
      const hours = Math.floor(diff / 3600000);
      const days = Math.floor(diff / 86400000);

      if (minutes < 1) return 'เมื่อกี้';
      if (minutes < 60) return `${minutes} นาทีที่แล้ว`;
      if (hours < 24) return `${hours} ชั่วโมงที่แล้ว`;
      return `${days} วันที่แล้ว`;
    }

    function render() {
      const list = document.getElementById('todoList');
      const filtered = getFilteredTodos();

      // Update stats
      const total = todos.length;
      const done = todos.filter(t => t.completed).length;
      const pending = total - done;

      document.getElementById('totalCount').textContent = total;
      document.getElementById('doneCount').textContent = done;
      document.getElementById('pendingCount').textContent = pending;

      // Footer info
      document.getElementById('footerInfo').textContent =
        filtered.length === 0 ? '' : `แสดง ${filtered.length} จาก ${total} รายการ`;

      // Render list
      if (filtered.length === 0) {
        list.innerHTML = `
          <div class="empty-state">
            <span class="emoji">${currentFilter === 'completed' ? '🎉' : '📝'}</span>
            <p>${
              currentFilter === 'all' ? 'ยังไม่มีรายการ เพิ่มได้เลย!' :
              currentFilter === 'pending' ? 'ไม่มีรายการรอดำเนินการ' :
              'ยังไม่มีรายการที่เสร็จแล้ว'
            }</p>
          </div>`;
        return;
      }

      list.innerHTML = filtered.map(todo => `
        <div class="todo-item ${todo.completed ? 'completed' : ''}" id="todo-${todo.id}">
          <input
            type="checkbox"
            class="todo-checkbox"
            ${todo.completed ? 'checked' : ''}
            onchange="toggleTodo(${todo.id})"
          />
          <span class="todo-text">${escapeHtml(todo.text)}</span>
          <span class="todo-date">${formatDate(todo.createdAt)}</span>
          <div class="todo-actions">
            <button class="btn-icon btn-edit" onclick="editTodo(${todo.id})" title="แก้ไข">✏️</button>
            <button class="btn-icon btn-delete" onclick="deleteTodo(${todo.id})" title="ลบ">🗑️</button>
          </div>
        </div>
      `).join('');
    }

    function escapeHtml(text) {
      const div = document.createElement('div');
      div.appendChild(document.createTextNode(text));
      return div.innerHTML;
    }

    // ========================================
    // EVENT LISTENERS
    // ========================================

    document.getElementById('todoInput').addEventListener('keydown', (e) => {
      if (e.key === 'Enter') addTodo();
    });

    // ========================================
    // INITIALIZE
    // ========================================

    loadTodos();
    render();
  </script>
</body>
</html>
```

### วิธีรันและปรับปรุง Todo App

บันทึกไฟล์เป็น `todo.html` แล้วเปิดในเบราว์เซอร์ได้เลย

**ความสามารถที่อาจเพิ่มได้:**
- Drag and drop เพื่อจัดลำดับ
- Categories/Tags
- Due dates
- Priority levels
- Export to PDF

---

## Step 375-378: โปรเจค 2 - Calculator App

```html
<!DOCTYPE html>
<html lang="th">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>เครื่องคิดเลข</title>
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; }

    body {
      font-family: 'Segoe UI', sans-serif;
      background: #1a1a2e;
      display: flex;
      align-items: center;
      justify-content: center;
      min-height: 100vh;
      padding: 20px;
    }

    .calculator {
      background: #16213e;
      border-radius: 24px;
      padding: 24px;
      box-shadow: 0 25px 80px rgba(0,0,0,0.5);
      width: 320px;
    }

    .display {
      background: #0f3460;
      border-radius: 16px;
      padding: 20px 16px 16px;
      margin-bottom: 20px;
      text-align: right;
      min-height: 100px;
      display: flex;
      flex-direction: column;
      justify-content: flex-end;
    }

    .history {
      color: #4a7c94;
      font-size: 14px;
      min-height: 20px;
      margin-bottom: 8px;
      overflow: hidden;
      text-overflow: ellipsis;
      white-space: nowrap;
    }

    .current {
      color: white;
      font-size: 42px;
      font-weight: 300;
      overflow: hidden;
      text-overflow: ellipsis;
      white-space: nowrap;
    }

    .current.small { font-size: 28px; }

    .buttons {
      display: grid;
      grid-template-columns: repeat(4, 1fr);
      gap: 10px;
    }

    .btn {
      padding: 18px;
      border: none;
      border-radius: 12px;
      font-size: 18px;
      font-weight: 500;
      cursor: pointer;
      transition: all 0.1s;
      outline: none;
    }

    .btn:active {
      transform: scale(0.94);
    }

    .btn-num {
      background: #1a1a2e;
      color: white;
    }

    .btn-num:hover { background: #252540; }

    .btn-op {
      background: #0f3460;
      color: #4fc3f7;
    }

    .btn-op:hover { background: #1565c0; color: white; }
    .btn-op.active { background: white; color: #0f3460; }

    .btn-clear {
      background: #4a1942;
      color: #ff8a80;
    }

    .btn-clear:hover { background: #7b1fa2; color: white; }

    .btn-equals {
      background: linear-gradient(135deg, #667eea, #764ba2);
      color: white;
      grid-column: span 2;
    }

    .btn-equals:hover { opacity: 0.9; }

    .btn-zero { grid-column: span 2; }

    .btn-special {
      background: #1a3a4a;
      color: #80cbc4;
    }

    .btn-special:hover { background: #00695c; color: white; }
  </style>
</head>
<body>
  <div class="calculator">
    <div class="display">
      <div class="history" id="history"></div>
      <div class="current" id="display">0</div>
    </div>

    <div class="buttons">
      <!-- Row 1 -->
      <button class="btn btn-clear" onclick="calc.clear()">C</button>
      <button class="btn btn-special" onclick="calc.toggleSign()">+/-</button>
      <button class="btn btn-special" onclick="calc.percent()">%</button>
      <button class="btn btn-op" data-op="/" onclick="calc.setOp('/')">÷</button>

      <!-- Row 2 -->
      <button class="btn btn-num" onclick="calc.inputNum(7)">7</button>
      <button class="btn btn-num" onclick="calc.inputNum(8)">8</button>
      <button class="btn btn-num" onclick="calc.inputNum(9)">9</button>
      <button class="btn btn-op" data-op="*" onclick="calc.setOp('*')">×</button>

      <!-- Row 3 -->
      <button class="btn btn-num" onclick="calc.inputNum(4)">4</button>
      <button class="btn btn-num" onclick="calc.inputNum(5)">5</button>
      <button class="btn btn-num" onclick="calc.inputNum(6)">6</button>
      <button class="btn btn-op" data-op="-" onclick="calc.setOp('-')">−</button>

      <!-- Row 4 -->
      <button class="btn btn-num" onclick="calc.inputNum(1)">1</button>
      <button class="btn btn-num" onclick="calc.inputNum(2)">2</button>
      <button class="btn btn-num" onclick="calc.inputNum(3)">3</button>
      <button class="btn btn-op" data-op="+" onclick="calc.setOp('+')">+</button>

      <!-- Row 5 -->
      <button class="btn btn-num btn-zero" onclick="calc.inputNum(0)">0</button>
      <button class="btn btn-num" onclick="calc.inputDot()">.</button>
      <button class="btn btn-equals" onclick="calc.calculate()">=</button>
    </div>
  </div>

  <script>
    const calc = {
      // STATE
      displayValue: '0',
      firstOperand: null,
      waitingForSecond: false,
      operator: null,

      // DISPLAY
      updateDisplay() {
        const display = document.getElementById('display');
        let val = this.displayValue;

        // เพิ่ม comma สำหรับตัวเลขใหญ่
        if (!val.includes('.') && !isNaN(val)) {
          const num = parseFloat(val);
          if (Math.abs(num) >= 1000 && num !== Infinity) {
            val = num.toLocaleString('en-US');
          }
        }

        display.textContent = val;
        display.className = 'current' + (val.length > 9 ? ' small' : '');
      },

      updateHistory(text) {
        document.getElementById('history').textContent = text;
      },

      // HIGHLIGHT ACTIVE OPERATOR
      highlightOp(op) {
        document.querySelectorAll('.btn-op').forEach(btn => {
          btn.classList.toggle('active', btn.dataset.op === op);
        });
      },

      // INPUT
      inputNum(num) {
        if (this.waitingForSecond) {
          this.displayValue = String(num);
          this.waitingForSecond = false;
        } else {
          this.displayValue = this.displayValue === '0'
            ? String(num)
            : this.displayValue + num;
        }
        this.updateDisplay();
      },

      inputDot() {
        if (this.waitingForSecond) {
          this.displayValue = '0.';
          this.waitingForSecond = false;
          this.updateDisplay();
          return;
        }
        if (!this.displayValue.includes('.')) {
          this.displayValue += '.';
          this.updateDisplay();
        }
      },

      // OPERATORS
      setOp(op) {
        const inputValue = parseFloat(this.displayValue);

        if (this.operator && this.waitingForSecond) {
          this.operator = op;
          this.highlightOp(op);
          return;
        }

        if (this.firstOperand === null) {
          this.firstOperand = inputValue;
        } else if (this.operator) {
          const result = this.performCalc(this.firstOperand, inputValue, this.operator);
          this.displayValue = String(parseFloat(result.toFixed(10)));
          this.firstOperand = parseFloat(this.displayValue);
          this.updateDisplay();
        }

        this.operator = op;
        this.waitingForSecond = true;
        this.highlightOp(op);

        const opSymbol = { '+': '+', '-': '−', '*': '×', '/': '÷' }[op];
        this.updateHistory(`${this.firstOperand} ${opSymbol}`);
      },

      performCalc(first, second, op) {
        switch (op) {
          case '+': return first + second;
          case '-': return first - second;
          case '*': return first * second;
          case '/':
            if (second === 0) throw new Error('หารด้วยศูนย์ไม่ได้!');
            return first / second;
          default: return second;
        }
      },

      calculate() {
        if (this.operator === null || this.waitingForSecond) return;

        const inputValue = parseFloat(this.displayValue);
        const opSymbol = { '+': '+', '-': '−', '*': '×', '/': '÷' }[this.operator];

        try {
          const result = this.performCalc(this.firstOperand, inputValue, this.operator);
          this.updateHistory(`${this.firstOperand} ${opSymbol} ${inputValue} =`);
          this.displayValue = String(parseFloat(result.toFixed(10)));
          this.firstOperand = null;
          this.operator = null;
          this.waitingForSecond = false;
          this.highlightOp(null);
          this.updateDisplay();
        } catch (error) {
          this.displayValue = 'Error';
          this.updateDisplay();
          this.updateHistory(error.message);
          setTimeout(() => this.clear(), 2000);
        }
      },

      // SPECIAL OPERATIONS
      clear() {
        this.displayValue = '0';
        this.firstOperand = null;
        this.waitingForSecond = false;
        this.operator = null;
        this.highlightOp(null);
        this.updateDisplay();
        this.updateHistory('');
      },

      toggleSign() {
        this.displayValue = String(-parseFloat(this.displayValue));
        this.updateDisplay();
      },

      percent() {
        this.displayValue = String(parseFloat(this.displayValue) / 100);
        this.updateDisplay();
      }
    };

    // Keyboard support
    document.addEventListener('keydown', (e) => {
      if (e.key >= '0' && e.key <= '9') calc.inputNum(parseInt(e.key));
      else if (e.key === '.') calc.inputDot();
      else if (e.key === '+') calc.setOp('+');
      else if (e.key === '-') calc.setOp('-');
      else if (e.key === '*') calc.setOp('*');
      else if (e.key === '/') { e.preventDefault(); calc.setOp('/'); }
      else if (e.key === 'Enter' || e.key === '=') calc.calculate();
      else if (e.key === 'Escape') calc.clear();
      else if (e.key === '%') calc.percent();
    });
  </script>
</body>
</html>
```

---

## Step 379-382: โปรเจค 3 - Quiz App

```html
<!DOCTYPE html>
<html lang="th">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Quiz JavaScript</title>
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; }

    body {
      font-family: 'Segoe UI', sans-serif;
      background: linear-gradient(135deg, #1a237e 0%, #311b92 100%);
      min-height: 100vh;
      display: flex;
      align-items: center;
      justify-content: center;
      padding: 20px;
    }

    .quiz-container {
      background: white;
      border-radius: 20px;
      width: 100%;
      max-width: 600px;
      overflow: hidden;
      box-shadow: 0 30px 80px rgba(0,0,0,0.4);
    }

    /* HEADER */
    .quiz-header {
      background: linear-gradient(135deg, #1a237e, #311b92);
      color: white;
      padding: 20px 24px;
      display: flex;
      justify-content: space-between;
      align-items: center;
    }

    .quiz-title { font-size: 20px; font-weight: 700; }
    .quiz-stats { display: flex; gap: 16px; font-size: 14px; }
    .stat-badge {
      background: rgba(255,255,255,0.2);
      padding: 4px 12px;
      border-radius: 12px;
    }

    /* PROGRESS BAR */
    .progress-container {
      background: #f0f0f0;
      height: 6px;
    }

    .progress-bar {
      height: 6px;
      background: linear-gradient(90deg, #7c4dff, #e040fb);
      transition: width 0.5s ease;
    }

    /* QUESTION */
    .question-section {
      padding: 32px 28px 20px;
    }

    .question-number {
      font-size: 12px;
      color: #7c4dff;
      font-weight: 600;
      text-transform: uppercase;
      letter-spacing: 1px;
      margin-bottom: 8px;
    }

    .question-text {
      font-size: 22px;
      color: #1a1a2e;
      font-weight: 600;
      line-height: 1.5;
    }

    /* OPTIONS */
    .options {
      padding: 0 28px 24px;
      display: grid;
      gap: 10px;
    }

    .option {
      padding: 14px 18px;
      border: 2px solid #e0e0e0;
      border-radius: 12px;
      cursor: pointer;
      transition: all 0.2s;
      display: flex;
      align-items: center;
      gap: 12px;
      font-size: 15px;
      color: #333;
      background: white;
    }

    .option:hover:not(.disabled) {
      border-color: #7c4dff;
      background: #f3f0ff;
      transform: translateX(4px);
    }

    .option.correct {
      border-color: #4caf50;
      background: #e8f5e9;
      color: #1b5e20;
    }

    .option.wrong {
      border-color: #f44336;
      background: #ffebee;
      color: #b71c1c;
    }

    .option.disabled {
      cursor: not-allowed;
      opacity: 0.8;
    }

    .option-letter {
      width: 28px;
      height: 28px;
      border-radius: 50%;
      background: #f0f0f0;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 12px;
      font-weight: 700;
      flex-shrink: 0;
    }

    .option.correct .option-letter { background: #4caf50; color: white; }
    .option.wrong .option-letter { background: #f44336; color: white; }

    /* EXPLANATION */
    .explanation {
      display: none;
      margin: 0 28px 20px;
      padding: 14px 16px;
      border-radius: 10px;
      font-size: 14px;
      line-height: 1.6;
    }

    .explanation.show { display: block; }
    .explanation.correct-exp { background: #e8f5e9; color: #2e7d32; border-left: 4px solid #4caf50; }
    .explanation.wrong-exp { background: #fff3e0; color: #e65100; border-left: 4px solid #ff9800; }

    /* NAVIGATION */
    .nav-section {
      padding: 16px 28px 24px;
      display: flex;
      justify-content: space-between;
      align-items: center;
      border-top: 1px solid #f0f0f0;
    }

    .timer {
      display: flex;
      align-items: center;
      gap: 6px;
      font-size: 18px;
      font-weight: 700;
      color: #333;
    }

    .timer.warning { color: #f44336; animation: pulse 1s infinite; }

    @keyframes pulse { 0%,100% { opacity: 1; } 50% { opacity: 0.5; } }

    .next-btn {
      padding: 12px 28px;
      background: linear-gradient(135deg, #7c4dff, #e040fb);
      color: white;
      border: none;
      border-radius: 10px;
      cursor: pointer;
      font-size: 15px;
      font-weight: 600;
      transition: all 0.2s;
      display: none;
    }

    .next-btn.show { display: block; }
    .next-btn:hover { opacity: 0.9; transform: scale(1.03); }

    /* SCREENS */
    .screen { display: none; }
    .screen.active { display: block; }

    /* START SCREEN */
    .start-screen {
      padding: 48px 28px;
      text-align: center;
    }

    .start-screen .icon { font-size: 64px; margin-bottom: 16px; }
    .start-screen h2 { font-size: 26px; color: #1a1a2e; margin-bottom: 12px; }
    .start-screen p { color: #666; margin-bottom: 32px; line-height: 1.6; }

    .start-btn {
      padding: 14px 40px;
      background: linear-gradient(135deg, #7c4dff, #e040fb);
      color: white;
      border: none;
      border-radius: 12px;
      font-size: 18px;
      font-weight: 700;
      cursor: pointer;
      transition: all 0.2s;
    }

    .start-btn:hover { transform: scale(1.05); opacity: 0.9; }

    /* RESULT SCREEN */
    .result-screen {
      padding: 40px 28px;
      text-align: center;
    }

    .result-circle {
      width: 140px;
      height: 140px;
      border-radius: 50%;
      margin: 0 auto 24px;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      font-size: 42px;
      font-weight: 800;
    }

    .result-circle.excellent { background: #e8f5e9; color: #1b5e20; border: 5px solid #4caf50; }
    .result-circle.good { background: #e3f2fd; color: #0d47a1; border: 5px solid #2196f3; }
    .result-circle.poor { background: #ffebee; color: #b71c1c; border: 5px solid #f44336; }

    .result-circle span { font-size: 14px; font-weight: 400; }
    .result-message { font-size: 22px; font-weight: 700; margin-bottom: 8px; }
    .result-sub { color: #666; margin-bottom: 24px; }

    .result-stats {
      display: flex;
      justify-content: center;
      gap: 20px;
      margin-bottom: 28px;
      flex-wrap: wrap;
    }

    .result-stat {
      background: #f8f9fa;
      padding: 12px 20px;
      border-radius: 10px;
      text-align: center;
    }

    .result-stat-num { font-size: 24px; font-weight: 700; color: #7c4dff; }
    .result-stat-label { font-size: 12px; color: #888; }

    .retry-btn {
      padding: 12px 32px;
      background: linear-gradient(135deg, #7c4dff, #e040fb);
      color: white;
      border: none;
      border-radius: 10px;
      font-size: 16px;
      font-weight: 600;
      cursor: pointer;
    }
  </style>
</head>
<body>
  <div class="quiz-container">
    <!-- START SCREEN -->
    <div class="screen active" id="startScreen">
      <div class="start-screen">
        <div class="icon">🧠</div>
        <h2>ทดสอบความรู้ JavaScript</h2>
        <p>
          คำถามทั้งหมด 10 ข้อเกี่ยวกับ JavaScript พื้นฐาน<br>
          มีเวลา 30 วินาทีต่อคำถาม<br>
          พร้อมแล้วก็เริ่มได้เลย!
        </p>
        <button class="start-btn" onclick="quiz.start()">เริ่มทำแบบทดสอบ</button>
      </div>
    </div>

    <!-- QUIZ SCREEN -->
    <div class="screen" id="quizScreen">
      <div class="quiz-header">
        <div class="quiz-title">🧠 JavaScript Quiz</div>
        <div class="quiz-stats">
          <span class="stat-badge" id="questionBadge">1/10</span>
          <span class="stat-badge" id="scoreBadge">คะแนน: 0</span>
        </div>
      </div>
      <div class="progress-container">
        <div class="progress-bar" id="progressBar" style="width: 10%"></div>
      </div>
      <div class="question-section">
        <div class="question-number" id="questionNum">คำถามที่ 1</div>
        <div class="question-text" id="questionText"></div>
      </div>
      <div class="options" id="optionsContainer"></div>
      <div class="explanation" id="explanation"></div>
      <div class="nav-section">
        <div class="timer" id="timer">⏱️ 30</div>
        <button class="next-btn" id="nextBtn" onclick="quiz.next()">ถัดไป →</button>
      </div>
    </div>

    <!-- RESULT SCREEN -->
    <div class="screen" id="resultScreen">
      <div class="result-screen">
        <div class="result-circle" id="resultCircle">
          <span id="resultScore">0</span>
          <span>คะแนน</span>
        </div>
        <div class="result-message" id="resultMessage"></div>
        <div class="result-sub" id="resultSub"></div>
        <div class="result-stats">
          <div class="result-stat">
            <div class="result-stat-num" id="correctCount">0</div>
            <div class="result-stat-label">ตอบถูก</div>
          </div>
          <div class="result-stat">
            <div class="result-stat-num" id="wrongCount">0</div>
            <div class="result-stat-label">ตอบผิด</div>
          </div>
          <div class="result-stat">
            <div class="result-stat-num" id="timeCount">0s</div>
            <div class="result-stat-label">เวลาเฉลี่ย</div>
          </div>
        </div>
        <button class="retry-btn" onclick="quiz.restart()">🔄 ทำอีกครั้ง</button>
      </div>
    </div>
  </div>

  <script>
    // ========================================
    // QUESTIONS DATABASE
    // ========================================

    const QUESTIONS = [
      {
        question: "ผลลัพธ์ของ typeof null ใน JavaScript คืออะไร?",
        options: ["'null'", "'undefined'", "'object'", "'boolean'"],
        correct: 2,
        explanation: "typeof null คืน 'object' นี่เป็น bug ที่มีมาตั้งแต่ต้นของ JavaScript และคงอยู่เพื่อ backward compatibility"
      },
      {
        question: "ข้อใดคือวิธีประกาศ arrow function ที่ถูกต้อง?",
        options: [
          "function => (x) { return x * 2; }",
          "const double = (x) => x * 2;",
          "const double = function(x) => x * 2;",
          "arrow double(x) { return x * 2; }"
        ],
        correct: 1,
        explanation: "Arrow function มี syntax: (parameters) => expression หรือ (parameters) => { statements }"
      },
      {
        question: "Array method ใดที่สร้าง array ใหม่โดยไม่เปลี่ยน original?",
        options: ["push()", "pop()", "map()", "sort()"],
        correct: 2,
        explanation: "map() สร้าง array ใหม่โดยไม่แตะ original array ส่วน push(), pop() และ sort() เปลี่ยน array เดิม"
      },
      {
        question: "ค่าของ 0 == false ใน JavaScript คือ?",
        options: ["true", "false", "undefined", "TypeError"],
        correct: 0,
        explanation: "== ใช้ type coercion ทำให้ 0 และ false ถูกแปลงเป็นชนิดเดียวกัน ผลคือ true ดังนั้นควรใช้ === แทน"
      },
      {
        question: "closure ใน JavaScript คืออะไร?",
        options: [
          "วิธีปิด browser tab",
          "function ที่ยังเข้าถึง variables ของ outer scope ได้แม้จะ return แล้ว",
          "วิธีลบ variable ออกจาก memory",
          "syntax สำหรับ try-catch"
        ],
        correct: 1,
        explanation: "Closure คือ function ที่จดจำ environment ที่มันถูกสร้างขึ้น ทำให้เข้าถึง variables ของ outer scope ได้แม้ function นั้นจะ return ไปแล้ว"
      },
      {
        question: "Promise.all() แตกต่างจาก Promise.race() อย่างไร?",
        options: [
          "Promise.all() รอทุก promise เสร็จ, Promise.race() รอแค่อันแรก",
          "Promise.all() ทำงานแบบ sequential, Promise.race() แบบ parallel",
          "ไม่มีความแตกต่าง",
          "Promise.all() สำหรับ async/await, Promise.race() สำหรับ callbacks"
        ],
        correct: 0,
        explanation: "Promise.all() รอให้ทุก promise resolve (หรือ reject ทันทีถ้ามีอันใดอัน reject) ส่วน Promise.race() resolve/reject ตาม promise แรกที่เสร็จ"
      },
      {
        question: "let กับ var ต่างกันอย่างไร?",
        options: [
          "let ใช้ใน Node.js, var ใช้ใน browser",
          "let มี block scope, var มี function scope",
          "let เร็วกว่า var",
          "ไม่มีความแตกต่าง"
        ],
        correct: 1,
        explanation: "let มี block scope ({}) ส่วน var มี function scope และถูก hoisted ขึ้น ทำให้ let ปลอดภัยกว่าและคาดเดาพฤติกรรมได้ง่ายกว่า"
      },
      {
        question: "เมธอด JSON.stringify() ทำอะไร?",
        options: [
          "แปลง JSON string เป็น JavaScript object",
          "แปลง JavaScript value เป็น JSON string",
          "ตรวจสอบว่า string เป็น JSON ที่ถูกต้องหรือไม่",
          "ลบ whitespace จาก JSON"
        ],
        correct: 1,
        explanation: "JSON.stringify() แปลง JavaScript values เช่น objects, arrays เป็น JSON string ส่วน JSON.parse() ทำตรงข้าม"
      },
      {
        question: "event delegation คืออะไร?",
        options: [
          "การส่ง events ระหว่าง components",
          "การ add event listener ที่ parent element แทนที่จะเป็นแต่ละ child",
          "การลบ event listeners เมื่อไม่ต้องการ",
          "การใช้ custom events"
        ],
        correct: 1,
        explanation: "Event delegation ใช้ประโยชน์จาก event bubbling โดย add listener ที่ parent element เพื่อจัดการ events จาก children ทั้งหมด ทำให้ประหยัด memory"
      },
      {
        question: "async/await คือ syntax sugar สำหรับอะไร?",
        options: [
          "Callbacks",
          "Event Emitters",
          "Promises",
          "Generators"
        ],
        correct: 2,
        explanation: "async/await เป็น syntax sugar บน Promises ทำให้โค้ด asynchronous อ่านง่ายขึ้นโดยดูเหมือน synchronous code"
      }
    ];

    // ========================================
    // QUIZ ENGINE
    // ========================================

    const quiz = {
      questions: [],
      currentIndex: 0,
      score: 0,
      answered: false,
      timerInterval: null,
      timeLeft: 30,
      timeTaken: [],

      start() {
        this.questions = [...QUESTIONS].sort(() => Math.random() - 0.5);
        this.currentIndex = 0;
        this.score = 0;
        this.timeTaken = [];

        this.showScreen('quizScreen');
        this.showQuestion();
      },

      showQuestion() {
        const q = this.questions[this.currentIndex];
        this.answered = false;

        // Update header
        const num = this.currentIndex + 1;
        const total = this.questions.length;
        document.getElementById('questionBadge').textContent = `${num}/${total}`;
        document.getElementById('scoreBadge').textContent = `คะแนน: ${this.score}`;
        document.getElementById('progressBar').style.width = `${(num / total) * 100}%`;

        // Question
        document.getElementById('questionNum').textContent = `คำถามที่ ${num}`;
        document.getElementById('questionText').textContent = q.question;

        // Options
        const letters = ['A', 'B', 'C', 'D'];
        document.getElementById('optionsContainer').innerHTML = q.options.map((opt, i) => `
          <button class="option" onclick="quiz.answer(${i})">
            <span class="option-letter">${letters[i]}</span>
            ${escapeHtml(opt)}
          </button>
        `).join('');

        // Hide explanation and next button
        const exp = document.getElementById('explanation');
        exp.className = 'explanation';
        exp.textContent = '';
        document.getElementById('nextBtn').className = 'next-btn';

        // Start timer
        this.startTimer();
      },

      startTimer() {
        this.timeLeft = 30;
        this.questionStartTime = Date.now();
        this.updateTimer();

        this.timerInterval = setInterval(() => {
          this.timeLeft--;
          this.updateTimer();

          if (this.timeLeft <= 0) {
            clearInterval(this.timerInterval);
            this.timeExpired();
          }
        }, 1000);
      },

      updateTimer() {
        const el = document.getElementById('timer');
        el.textContent = `⏱️ ${this.timeLeft}`;
        el.className = 'timer' + (this.timeLeft <= 10 ? ' warning' : '');
      },

      timeExpired() {
        if (!this.answered) {
          this.answered = true;
          this.timeTaken.push(30);
          this.showAnswer(-1); // -1 = no answer (timeout)
        }
      },

      answer(selectedIndex) {
        if (this.answered) return;
        this.answered = true;

        clearInterval(this.timerInterval);
        const elapsed = Math.round((Date.now() - this.questionStartTime) / 1000);
        this.timeTaken.push(elapsed);

        this.showAnswer(selectedIndex);
      },

      showAnswer(selectedIndex) {
        const q = this.questions[this.currentIndex];
        const options = document.querySelectorAll('.option');

        // ทำ options disabled
        options.forEach(opt => opt.classList.add('disabled'));

        if (selectedIndex === q.correct) {
          // ถูก!
          options[selectedIndex].classList.add('correct');
          this.score += 10;

          const exp = document.getElementById('explanation');
          exp.className = 'explanation correct-exp show';
          exp.innerHTML = `✅ <strong>ถูกต้อง!</strong> ${q.explanation}`;
        } else {
          // ผิด
          if (selectedIndex >= 0) {
            options[selectedIndex].classList.add('wrong');
          }
          options[q.correct].classList.add('correct'); // แสดงคำตอบที่ถูก

          const exp = document.getElementById('explanation');
          exp.className = 'explanation wrong-exp show';
          exp.innerHTML = `❌ <strong>ไม่ถูกต้อง</strong> ${q.explanation}`;
        }

        // Update score display
        document.getElementById('scoreBadge').textContent = `คะแนน: ${this.score}`;

        // Show next button
        const nextBtn = document.getElementById('nextBtn');
        nextBtn.className = 'next-btn show';
        nextBtn.textContent = this.currentIndex < this.questions.length - 1 ? 'ถัดไป →' : 'ดูผลลัพธ์ 🎯';
      },

      next() {
        this.currentIndex++;
        if (this.currentIndex < this.questions.length) {
          this.showQuestion();
        } else {
          this.showResults();
        }
      },

      showResults() {
        const total = this.questions.length;
        const percentage = Math.round((this.score / (total * 10)) * 100);
        const correct = this.score / 10;
        const wrong = total - correct;
        const avgTime = Math.round(this.timeTaken.reduce((a, b) => a + b, 0) / total);

        document.getElementById('resultScore').textContent = `${percentage}%`;
        document.getElementById('correctCount').textContent = correct;
        document.getElementById('wrongCount').textContent = wrong;
        document.getElementById('timeCount').textContent = `${avgTime}s`;

        const circle = document.getElementById('resultCircle');
        let message, sub;

        if (percentage >= 80) {
          circle.className = 'result-circle excellent';
          message = '🌟 ยอดเยี่ยม!';
          sub = 'คุณมีความรู้ JavaScript ในระดับดีมาก';
        } else if (percentage >= 50) {
          circle.className = 'result-circle good';
          message = '👍 ดีมาก!';
          sub = 'ยังมีบางเรื่องที่ควรทบทวนเพิ่มเติม';
        } else {
          circle.className = 'result-circle poor';
          message = '📚 ต้องฝึกเพิ่มอีก!';
          sub = 'แนะนำให้กลับไปทบทวนเนื้อหาอีกครั้ง';
        }

        document.getElementById('resultMessage').textContent = message;
        document.getElementById('resultSub').textContent = sub;

        this.showScreen('resultScreen');
      },

      restart() {
        this.showScreen('startScreen');
      },

      showScreen(screenId) {
        document.querySelectorAll('.screen').forEach(s => s.classList.remove('active'));
        document.getElementById(screenId).classList.add('active');
      }
    };

    function escapeHtml(text) {
      const div = document.createElement('div');
      div.appendChild(document.createTextNode(text));
      return div.innerHTML;
    }
  </script>
</body>
</html>
```

---

## Step 383-386: โปรเจค 4 - Weather App

```html
<!DOCTYPE html>
<html lang="th">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Weather App</title>
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; }

    body {
      font-family: 'Segoe UI', sans-serif;
      background: linear-gradient(135deg, #0f2027 0%, #203a43 50%, #2c5364 100%);
      min-height: 100vh;
      display: flex;
      align-items: center;
      justify-content: center;
      padding: 20px;
    }

    .weather-app {
      width: 100%;
      max-width: 420px;
      color: white;
    }

    /* SEARCH */
    .search-section { margin-bottom: 24px; }

    .search-box {
      display: flex;
      background: rgba(255,255,255,0.15);
      backdrop-filter: blur(10px);
      border-radius: 50px;
      overflow: hidden;
      border: 1px solid rgba(255,255,255,0.2);
    }

    .search-input {
      flex: 1;
      padding: 14px 20px;
      background: transparent;
      border: none;
      color: white;
      font-size: 16px;
      outline: none;
    }

    .search-input::placeholder { color: rgba(255,255,255,0.6); }

    .search-btn {
      padding: 14px 20px;
      background: rgba(255,255,255,0.2);
      border: none;
      color: white;
      cursor: pointer;
      font-size: 18px;
      transition: background 0.2s;
    }

    .search-btn:hover { background: rgba(255,255,255,0.3); }

    /* WEATHER CARD */
    .weather-card {
      background: rgba(255,255,255,0.12);
      backdrop-filter: blur(20px);
      border-radius: 24px;
      padding: 28px;
      border: 1px solid rgba(255,255,255,0.15);
      display: none;
    }

    .weather-card.show { display: block; }

    .city-name {
      font-size: 28px;
      font-weight: 700;
      margin-bottom: 4px;
    }

    .date-time {
      color: rgba(255,255,255,0.7);
      font-size: 14px;
      margin-bottom: 24px;
    }

    .main-weather {
      display: flex;
      align-items: center;
      gap: 16px;
      margin-bottom: 24px;
    }

    .weather-icon { font-size: 80px; line-height: 1; }

    .temp-info { flex: 1; }

    .temperature {
      font-size: 64px;
      font-weight: 300;
      line-height: 1;
      margin-bottom: 4px;
    }

    .feels-like { font-size: 14px; color: rgba(255,255,255,0.7); }
    .description { font-size: 18px; text-transform: capitalize; }

    /* DETAILS */
    .weather-details {
      display: grid;
      grid-template-columns: repeat(2, 1fr);
      gap: 12px;
      margin-bottom: 24px;
    }

    .detail-card {
      background: rgba(255,255,255,0.1);
      border-radius: 14px;
      padding: 14px;
      display: flex;
      align-items: center;
      gap: 10px;
    }

    .detail-icon { font-size: 24px; }

    .detail-info { flex: 1; }
    .detail-label { font-size: 12px; color: rgba(255,255,255,0.6); }
    .detail-value { font-size: 18px; font-weight: 600; }

    /* FORECAST */
    .forecast-title {
      font-size: 14px;
      color: rgba(255,255,255,0.7);
      margin-bottom: 12px;
      text-transform: uppercase;
      letter-spacing: 1px;
    }

    .forecast-row {
      display: flex;
      justify-content: space-between;
      align-items: center;
      gap: 8px;
    }

    .forecast-item {
      flex: 1;
      background: rgba(255,255,255,0.08);
      border-radius: 12px;
      padding: 12px 8px;
      text-align: center;
    }

    .fc-day { font-size: 12px; color: rgba(255,255,255,0.6); margin-bottom: 6px; }
    .fc-icon { font-size: 24px; margin-bottom: 6px; }
    .fc-temp { font-size: 16px; font-weight: 600; }
    .fc-temp small { font-size: 12px; color: rgba(255,255,255,0.5); }

    /* LOADING */
    .loading {
      display: none;
      text-align: center;
      padding: 40px;
    }

    .loading.show { display: block; }

    .spinner {
      width: 40px;
      height: 40px;
      border: 3px solid rgba(255,255,255,0.2);
      border-top-color: white;
      border-radius: 50%;
      animation: spin 0.8s linear infinite;
      margin: 0 auto 12px;
    }

    @keyframes spin { to { transform: rotate(360deg); } }

    /* ERROR */
    .error {
      display: none;
      background: rgba(244,67,54,0.2);
      border: 1px solid rgba(244,67,54,0.4);
      border-radius: 12px;
      padding: 16px;
      text-align: center;
    }

    .error.show { display: block; }
    .error .error-icon { font-size: 32px; margin-bottom: 8px; }
  </style>
</head>
<body>
  <div class="weather-app">
    <div class="search-section">
      <div class="search-box">
        <input
          type="text"
          id="cityInput"
          class="search-input"
          placeholder="ค้นหาเมือง... (เช่น Bangkok, Tokyo)"
        />
        <button class="search-btn" onclick="searchWeather()">🔍</button>
      </div>
    </div>

    <div class="loading" id="loading">
      <div class="spinner"></div>
      <p>กำลังโหลดข้อมูลอากาศ...</p>
    </div>

    <div class="error" id="errorMsg">
      <div class="error-icon">😕</div>
      <p id="errorText">ไม่พบข้อมูลสำหรับเมืองนี้</p>
    </div>

    <div class="weather-card" id="weatherCard">
      <div class="city-name" id="cityName"></div>
      <div class="date-time" id="dateTime"></div>

      <div class="main-weather">
        <div class="weather-icon" id="weatherIcon"></div>
        <div class="temp-info">
          <div class="temperature" id="temperature"></div>
          <div class="description" id="description"></div>
          <div class="feels-like" id="feelsLike"></div>
        </div>
      </div>

      <div class="weather-details">
        <div class="detail-card">
          <div class="detail-icon">💧</div>
          <div class="detail-info">
            <div class="detail-label">ความชื้น</div>
            <div class="detail-value" id="humidity"></div>
          </div>
        </div>
        <div class="detail-card">
          <div class="detail-icon">🌬️</div>
          <div class="detail-info">
            <div class="detail-label">ความเร็วลม</div>
            <div class="detail-value" id="windSpeed"></div>
          </div>
        </div>
        <div class="detail-card">
          <div class="detail-icon">👁️</div>
          <div class="detail-info">
            <div class="detail-label">ทัศนวิสัย</div>
            <div class="detail-value" id="visibility"></div>
          </div>
        </div>
        <div class="detail-card">
          <div class="detail-icon">🌡️</div>
          <div class="detail-info">
            <div class="detail-label">ความกดอากาศ</div>
            <div class="detail-value" id="pressure"></div>
          </div>
        </div>
      </div>

      <div class="forecast-title">🗓️ พยากรณ์ 5 วัน</div>
      <div class="forecast-row" id="forecastRow"></div>
    </div>
  </div>

  <script>
    // NOTE: ใส่ API key จาก openweathermap.org
    // สมัครฟรีที่ https://openweathermap.org/api
    const API_KEY = 'YOUR_API_KEY_HERE'; // แทนด้วย API key จริง
    const BASE_URL = 'https://api.openweathermap.org/data/2.5';

    // Weather icon mapping
    const WEATHER_ICONS = {
      '01': '☀️', '02': '⛅', '03': '☁️', '04': '☁️',
      '09': '🌧️', '10': '🌦️', '11': '⛈️', '13': '❄️', '50': '🌫️'
    };

    // Demo data สำหรับแสดงตัวอย่างโดยไม่ต้องมี API key
    const DEMO_DATA = {
      name: 'Bangkok',
      sys: { country: 'TH' },
      main: { temp: 32, feels_like: 38, humidity: 75, pressure: 1008 },
      weather: [{ description: 'partly cloudy', icon: '02d' }],
      wind: { speed: 12 },
      visibility: 10000,
      coord: { lat: 13.75, lon: 100.52 },
      dt: Date.now() / 1000
    };

    const DEMO_FORECAST = {
      list: Array.from({ length: 5 }, (_, i) => ({
        dt: (Date.now() / 1000) + (i + 1) * 86400,
        main: { temp_max: 30 + Math.random() * 5, temp_min: 25 + Math.random() * 3 },
        weather: [{ icon: ['01d', '02d', '10d', '01d', '03d'][i] }]
      }))
    };

    function getWeatherIcon(iconCode) {
      const code = iconCode.slice(0, 2);
      return WEATHER_ICONS[code] || '🌤️';
    }

    function formatDate(timestamp) {
      return new Date(timestamp * 1000).toLocaleDateString('th-TH', {
        weekday: 'long', year: 'numeric', month: 'long', day: 'numeric'
      });
    }

    function getDayName(timestamp) {
      const days = ['อา.', 'จ.', 'อ.', 'พ.', 'พฤ.', 'ศ.', 'ส.'];
      return days[new Date(timestamp * 1000).getDay()];
    }

    function showLoading() {
      document.getElementById('loading').classList.add('show');
      document.getElementById('weatherCard').classList.remove('show');
      document.getElementById('errorMsg').classList.remove('show');
    }

    function showError(msg) {
      document.getElementById('loading').classList.remove('show');
      document.getElementById('weatherCard').classList.remove('show');
      document.getElementById('errorMsg').classList.add('show');
      document.getElementById('errorText').textContent = msg;
    }

    function displayWeather(data, forecastData) {
      document.getElementById('loading').classList.remove('show');
      document.getElementById('errorMsg').classList.remove('show');
      document.getElementById('weatherCard').classList.add('show');

      // Basic info
      document.getElementById('cityName').textContent = `${data.name}, ${data.sys.country}`;
      document.getElementById('dateTime').textContent = formatDate(data.dt);
      document.getElementById('weatherIcon').textContent = getWeatherIcon(data.weather[0].icon);
      document.getElementById('temperature').textContent = `${Math.round(data.main.temp)}°C`;
      document.getElementById('description').textContent = data.weather[0].description;
      document.getElementById('feelsLike').textContent = `รู้สึกเหมือน ${Math.round(data.main.feels_like)}°C`;

      // Details
      document.getElementById('humidity').textContent = `${data.main.humidity}%`;
      document.getElementById('windSpeed').textContent = `${Math.round(data.wind.speed)} km/h`;
      document.getElementById('visibility').textContent = `${(data.visibility / 1000).toFixed(1)} km`;
      document.getElementById('pressure').textContent = `${data.main.pressure} hPa`;

      // Forecast
      if (forecastData) {
        // Get one entry per day
        const dailyForecasts = forecastData.list.slice(0, 5);

        document.getElementById('forecastRow').innerHTML = dailyForecasts.map(day => `
          <div class="forecast-item">
            <div class="fc-day">${getDayName(day.dt)}</div>
            <div class="fc-icon">${getWeatherIcon(day.weather[0].icon)}</div>
            <div class="fc-temp">
              ${Math.round(day.main.temp_max)}°
              <small>/${Math.round(day.main.temp_min)}°</small>
            </div>
          </div>
        `).join('');
      }
    }

    async function fetchWeatherData(city) {
      // ถ้ายังไม่มี API key ให้แสดง demo data
      if (API_KEY === 'YOUR_API_KEY_HERE') {
        // Simulate API delay
        await new Promise(resolve => setTimeout(resolve, 800));

        if (city.toLowerCase() !== 'demo' && city.toLowerCase() !== 'bangkok' && city.trim() !== '') {
          // Fake some variation for different cities
          const fakeData = {
            ...DEMO_DATA,
            name: city,
            main: {
              ...DEMO_DATA.main,
              temp: 20 + Math.random() * 20,
              feels_like: 22 + Math.random() * 20,
              humidity: 40 + Math.random() * 50
            }
          };
          return { current: fakeData, forecast: DEMO_FORECAST };
        }

        return { current: DEMO_DATA, forecast: DEMO_FORECAST };
      }

      // Real API call
      const [currentRes, forecastRes] = await Promise.all([
        fetch(`${BASE_URL}/weather?q=${encodeURIComponent(city)}&units=metric&appid=${API_KEY}&lang=th`),
        fetch(`${BASE_URL}/forecast/daily?q=${encodeURIComponent(city)}&cnt=5&units=metric&appid=${API_KEY}`)
      ]);

      if (!currentRes.ok) {
        if (currentRes.status === 404) throw new Error('ไม่พบเมืองที่ค้นหา');
        if (currentRes.status === 401) throw new Error('API Key ไม่ถูกต้อง');
        throw new Error(`เกิดข้อผิดพลาด: ${currentRes.status}`);
      }

      const [current, forecast] = await Promise.all([
        currentRes.json(),
        forecastRes.json()
      ]);

      return { current, forecast };
    }

    async function searchWeather() {
      const city = document.getElementById('cityInput').value.trim();
      if (!city) return;

      showLoading();

      try {
        const { current, forecast } = await fetchWeatherData(city);
        displayWeather(current, forecast);
      } catch (error) {
        showError(error.message || 'เกิดข้อผิดพลาดในการโหลดข้อมูล');
      }
    }

    // Enter key to search
    document.getElementById('cityInput').addEventListener('keydown', (e) => {
      if (e.key === 'Enter') searchWeather();
    });

    // Load Bangkok on start
    document.getElementById('cityInput').value = 'Bangkok';
    searchWeather();
  </script>
</body>
</html>
```

**หมายเหตุ:** สำหรับ Weather App จริง ต้องสมัคร API key ฟรีที่ [openweathermap.org](https://openweathermap.org/api) และแทน `YOUR_API_KEY_HERE` ด้วย key จริง แต่แอพจะทำงานด้วย demo data ได้ทันที

---

## Step 387-390: โปรเจค 5 - Note Taking App กับ localStorage

```html
<!DOCTYPE html>
<html lang="th">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>โน้ตแอป</title>
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; }

    :root {
      --primary: #6c63ff;
      --primary-light: #f0efff;
      --sidebar-bg: #2d2b55;
      --text: #1a1a2e;
      --text-light: #666;
      --border: #e0e0e0;
      --bg: #f5f5f5;
    }

    body {
      font-family: 'Segoe UI', sans-serif;
      background: var(--bg);
      height: 100vh;
      display: flex;
      overflow: hidden;
    }

    /* SIDEBAR */
    .sidebar {
      width: 280px;
      background: var(--sidebar-bg);
      color: white;
      display: flex;
      flex-direction: column;
      flex-shrink: 0;
    }

    .sidebar-header {
      padding: 20px;
      border-bottom: 1px solid rgba(255,255,255,0.1);
    }

    .sidebar-header h1 { font-size: 20px; margin-bottom: 12px; }

    .new-note-btn {
      width: 100%;
      padding: 10px;
      background: var(--primary);
      color: white;
      border: none;
      border-radius: 8px;
      cursor: pointer;
      font-size: 14px;
      font-weight: 600;
      transition: opacity 0.2s;
    }

    .new-note-btn:hover { opacity: 0.9; }

    .search-container { padding: 12px 16px; }

    .search-box {
      width: 100%;
      padding: 8px 12px;
      background: rgba(255,255,255,0.1);
      border: 1px solid rgba(255,255,255,0.15);
      border-radius: 8px;
      color: white;
      font-size: 13px;
      outline: none;
    }

    .search-box::placeholder { color: rgba(255,255,255,0.4); }
    .search-box:focus { border-color: var(--primary); }

    .notes-list {
      flex: 1;
      overflow-y: auto;
      padding: 8px;
    }

    .note-item {
      padding: 12px;
      border-radius: 10px;
      cursor: pointer;
      transition: background 0.15s;
      margin-bottom: 4px;
    }

    .note-item:hover { background: rgba(255,255,255,0.1); }
    .note-item.active { background: var(--primary); }

    .note-item-title {
      font-size: 14px;
      font-weight: 600;
      white-space: nowrap;
      overflow: hidden;
      text-overflow: ellipsis;
      margin-bottom: 4px;
    }

    .note-item-preview {
      font-size: 12px;
      opacity: 0.65;
      white-space: nowrap;
      overflow: hidden;
      text-overflow: ellipsis;
    }

    .note-item-date {
      font-size: 11px;
      opacity: 0.5;
      margin-top: 4px;
    }

    .no-notes {
      text-align: center;
      padding: 32px 16px;
      opacity: 0.5;
      font-size: 13px;
    }

    /* EDITOR */
    .editor-area {
      flex: 1;
      display: flex;
      flex-direction: column;
      overflow: hidden;
    }

    .editor-toolbar {
      padding: 12px 20px;
      background: white;
      border-bottom: 1px solid var(--border);
      display: flex;
      align-items: center;
      gap: 8px;
    }

    .editor-toolbar input[type="text"] {
      flex: 1;
      padding: 8px 12px;
      border: 1px solid var(--border);
      border-radius: 8px;
      font-size: 16px;
      outline: none;
    }

    .editor-toolbar input:focus { border-color: var(--primary); }

    .toolbar-btn {
      padding: 8px 14px;
      border: none;
      border-radius: 8px;
      cursor: pointer;
      font-size: 13px;
      transition: all 0.15s;
    }

    .btn-save { background: var(--primary); color: white; }
    .btn-save:hover { opacity: 0.9; }

    .btn-delete-note { background: #ffebee; color: #c62828; }
    .btn-delete-note:hover { background: #c62828; color: white; }

    .btn-export { background: #e8f5e9; color: #2e7d32; }
    .btn-export:hover { background: #2e7d32; color: white; }

    .format-bar {
      padding: 8px 20px;
      background: #f8f9fa;
      border-bottom: 1px solid var(--border);
      display: flex;
      gap: 4px;
    }

    .fmt-btn {
      width: 32px;
      height: 32px;
      border: 1px solid var(--border);
      border-radius: 6px;
      background: white;
      cursor: pointer;
      font-size: 13px;
      display: flex;
      align-items: center;
      justify-content: center;
      transition: all 0.15s;
    }

    .fmt-btn:hover { background: var(--primary-light); border-color: var(--primary); }

    .note-editor {
      flex: 1;
      padding: 24px 28px;
      font-size: 15px;
      line-height: 1.8;
      border: none;
      resize: none;
      outline: none;
      color: var(--text);
      background: white;
      font-family: 'Segoe UI', sans-serif;
    }

    /* EMPTY STATE */
    .empty-editor {
      flex: 1;
      display: flex;
      flex-direction: column;
      align-items: center;
      justify-content: center;
      color: var(--text-light);
      gap: 12px;
    }

    .empty-editor .icon { font-size: 64px; }
    .empty-editor p { font-size: 16px; }

    /* STATUS BAR */
    .status-bar {
      padding: 6px 20px;
      background: #f8f9fa;
      border-top: 1px solid var(--border);
      display: flex;
      justify-content: space-between;
      font-size: 12px;
      color: var(--text-light);
    }

    /* TAG SYSTEM */
    .tag {
      display: inline-block;
      padding: 2px 8px;
      border-radius: 10px;
      font-size: 11px;
      margin-right: 4px;
      background: var(--primary-light);
      color: var(--primary);
    }

    /* RESPONSIVE */
    @media (max-width: 600px) {
      .sidebar { width: 100%; position: absolute; z-index: 10; height: 100%; transform: translateX(-100%); transition: transform 0.3s; }
      .sidebar.open { transform: translateX(0); }
    }
  </style>
</head>
<body>

  <!-- SIDEBAR -->
  <div class="sidebar" id="sidebar">
    <div class="sidebar-header">
      <h1>📝 โน้ตของฉัน</h1>
      <button class="new-note-btn" onclick="app.createNote()">+ โน้ตใหม่</button>
    </div>

    <div class="search-container">
      <input type="text" class="search-box" id="searchBox" placeholder="ค้นหาโน้ต..." oninput="app.search(this.value)" />
    </div>

    <div class="notes-list" id="notesList"></div>
  </div>

  <!-- EDITOR -->
  <div class="editor-area" id="editorArea">
    <div class="empty-editor" id="emptyEditor">
      <span class="icon">📝</span>
      <p>เลือกโน้ตหรือสร้างโน้ตใหม่</p>
      <button class="new-note-btn" style="width:auto;padding:10px 24px" onclick="app.createNote()">+ โน้ตใหม่</button>
    </div>

    <div id="activeEditor" style="display:none;flex-direction:column;flex:1;overflow:hidden;">
      <div class="editor-toolbar">
        <input type="text" id="noteTitle" placeholder="หัวข้อโน้ต" oninput="app.onTitleChange()" />
        <button class="toolbar-btn btn-save" onclick="app.saveNote()">💾 บันทึก</button>
        <button class="toolbar-btn btn-export" onclick="app.exportNote()">📤 Export</button>
        <button class="toolbar-btn btn-delete-note" onclick="app.deleteCurrentNote()">🗑️ ลบ</button>
      </div>

      <div class="format-bar">
        <button class="fmt-btn" onclick="app.format('bold')" title="Bold"><b>B</b></button>
        <button class="fmt-btn" onclick="app.format('italic')" title="Italic"><i>I</i></button>
        <button class="fmt-btn" onclick="app.format('underline')" title="Underline"><u>U</u></button>
        <button class="fmt-btn" onclick="app.format('insertUnorderedList')" title="Bullet List">• List</button>
        <button class="fmt-btn" onclick="app.format('insertOrderedList')" title="Numbered List">1. List</button>
        <button class="fmt-btn" onclick="app.addCodeBlock()" title="Code Block">&lt;/&gt;</button>
      </div>

      <div
        id="noteEditor"
        class="note-editor"
        contenteditable="true"
        oninput="app.onContentChange()"
        placeholder="เริ่มเขียนโน้ตได้เลย..."
      ></div>

      <div class="status-bar">
        <span id="wordCount">0 คำ</span>
        <span id="lastSaved">ยังไม่ได้บันทึก</span>
      </div>
    </div>
  </div>

  <script>
    const app = {
      notes: [],
      activeId: null,
      unsaved: false,
      autoSaveTimer: null,
      STORAGE_KEY: 'notes_app_v2',

      // ========== INIT ==========
      init() {
        this.loadNotes();
        this.renderList();
        if (this.notes.length > 0) {
          this.openNote(this.notes[0].id);
        }
      },

      // ========== STORAGE ==========
      loadNotes() {
        try {
          const saved = localStorage.getItem(this.STORAGE_KEY);
          this.notes = saved ? JSON.parse(saved) : this.getDefaultNotes();
        } catch {
          this.notes = this.getDefaultNotes();
        }
      },

      saveToStorage() {
        localStorage.setItem(this.STORAGE_KEY, JSON.stringify(this.notes));
      },

      getDefaultNotes() {
        return [
          {
            id: '1',
            title: 'ยินดีต้อนรับ! 👋',
            content: '<p>ยินดีต้อนรับสู่โน้ตแอป!</p><p>คุณสามารถ:</p><ul><li>สร้างโน้ตใหม่ด้วยปุ่ม "+ โน้ตใหม่"</li><li>แก้ไขโน้ตโดยคลิกที่รายการ</li><li>ค้นหาโน้ตด้วย search box</li><li>Export โน้ตเป็นไฟล์ .txt</li></ul>',
            createdAt: Date.now(),
            updatedAt: Date.now()
          }
        ];
      },

      // ========== CRUD ==========
      createNote() {
        const note = {
          id: Date.now().toString(),
          title: 'โน้ตใหม่',
          content: '',
          createdAt: Date.now(),
          updatedAt: Date.now()
        };
        this.notes.unshift(note);
        this.saveToStorage();
        this.renderList();
        this.openNote(note.id);
        setTimeout(() => document.getElementById('noteTitle').select(), 50);
      },

      openNote(id) {
        if (this.unsaved && this.activeId && !confirm('มีการแก้ไขที่ยังไม่บันทึก ต้องการออกหรือไม่?')) {
          return;
        }

        this.activeId = id;
        const note = this.getNote(id);
        if (!note) return;

        document.getElementById('emptyEditor').style.display = 'none';
        document.getElementById('activeEditor').style.display = 'flex';

        document.getElementById('noteTitle').value = note.title;
        document.getElementById('noteEditor').innerHTML = note.content;

        this.unsaved = false;
        this.updateStatus();
        this.renderList();

        document.getElementById('noteEditor').focus();
      },

      saveNote() {
        if (!this.activeId) return;
        const note = this.getNote(this.activeId);
        if (!note) return;

        note.title = document.getElementById('noteTitle').value.trim() || 'ไม่มีหัวข้อ';
        note.content = document.getElementById('noteEditor').innerHTML;
        note.updatedAt = Date.now();

        this.saveToStorage();
        this.renderList();
        this.unsaved = false;
        this.updateStatus(true);
      },

      deleteCurrentNote() {
        if (!this.activeId) return;
        const note = this.getNote(this.activeId);
        if (!note) return;

        if (!confirm(`ต้องการลบโน้ต "${note.title}" หรือไม่?`)) return;

        this.notes = this.notes.filter(n => n.id !== this.activeId);
        this.saveToStorage();

        // Open next note or show empty state
        if (this.notes.length > 0) {
          this.activeId = null;
          this.openNote(this.notes[0].id);
        } else {
          this.activeId = null;
          document.getElementById('emptyEditor').style.display = 'flex';
          document.getElementById('activeEditor').style.display = 'none';
        }

        this.renderList();
      },

      // ========== FORMATTING ==========
      format(command) {
        document.getElementById('noteEditor').focus();
        document.execCommand(command, false, null);
        this.onContentChange();
      },

      addCodeBlock() {
        const code = prompt('ใส่ code:');
        if (!code) return;
        document.getElementById('noteEditor').focus();
        document.execCommand('insertHTML', false,
          `<pre style="background:#f0f0f0;padding:12px;border-radius:6px;font-family:monospace;margin:8px 0">${escapeHtml(code)}</pre>`
        );
        this.onContentChange();
      },

      // ========== EVENT HANDLERS ==========
      onTitleChange() {
        this.unsaved = true;
        this.scheduleAutoSave();
      },

      onContentChange() {
        this.unsaved = true;
        this.scheduleAutoSave();
        this.updateWordCount();
      },

      scheduleAutoSave() {
        clearTimeout(this.autoSaveTimer);
        this.autoSaveTimer = setTimeout(() => this.saveNote(), 2000);
      },

      // ========== SEARCH ==========
      search(query) {
        this.renderList(query.toLowerCase());
      },

      // ========== EXPORT ==========
      exportNote() {
        if (!this.activeId) return;
        const note = this.getNote(this.activeId);
        if (!note) return;

        const text = [
          note.title,
          '='.repeat(note.title.length),
          '',
          document.getElementById('noteEditor').innerText,
          '',
          `สร้างเมื่อ: ${new Date(note.createdAt).toLocaleString('th-TH')}`,
          `แก้ไขล่าสุด: ${new Date(note.updatedAt).toLocaleString('th-TH')}`
        ].join('\n');

        const blob = new Blob([text], { type: 'text/plain;charset=utf-8' });
        const url = URL.createObjectURL(blob);
        const a = document.createElement('a');
        a.href = url;
        a.download = `${note.title.replace(/[^a-z0-9ก-ฮ]/gi, '_')}.txt`;
        a.click();
        URL.revokeObjectURL(url);
      },

      // ========== HELPERS ==========
      getNote(id) {
        return this.notes.find(n => n.id === id);
      },

      formatDate(timestamp) {
        const diff = Date.now() - timestamp;
        if (diff < 60000) return 'เมื่อกี้';
        if (diff < 3600000) return `${Math.floor(diff/60000)} นาทีที่แล้ว`;
        if (diff < 86400000) return `${Math.floor(diff/3600000)} ชั่วโมงที่แล้ว`;
        return new Date(timestamp).toLocaleDateString('th-TH');
      },

      updateWordCount() {
        const text = document.getElementById('noteEditor').innerText;
        const words = text.trim().split(/\s+/).filter(w => w).length;
        document.getElementById('wordCount').textContent = `${words} คำ`;
      },

      updateStatus(saved = false) {
        const note = this.getNote(this.activeId);
        if (!note) return;
        if (saved) {
          document.getElementById('lastSaved').textContent =
            `บันทึกเมื่อ ${new Date().toLocaleTimeString('th-TH')}`;
        }
        this.updateWordCount();
      },

      // ========== RENDERING ==========
      renderList(searchQuery = '') {
        const list = document.getElementById('notesList');
        let filtered = this.notes;

        if (searchQuery) {
          filtered = this.notes.filter(n =>
            n.title.toLowerCase().includes(searchQuery) ||
            n.content.toLowerCase().includes(searchQuery)
          );
        }

        if (filtered.length === 0) {
          list.innerHTML = `<div class="no-notes">${searchQuery ? 'ไม่พบโน้ตที่ค้นหา' : 'ยังไม่มีโน้ต'}</div>`;
          return;
        }

        list.innerHTML = filtered.map(note => {
          const isActive = note.id === this.activeId;
          const preview = note.content.replace(/<[^>]+>/g, '').slice(0, 60);

          return `
            <div class="note-item ${isActive ? 'active' : ''}" onclick="app.openNote('${note.id}')">
              <div class="note-item-title">${escapeHtml(note.title)}</div>
              <div class="note-item-preview">${escapeHtml(preview) || 'ไม่มีเนื้อหา'}</div>
              <div class="note-item-date">${this.formatDate(note.updatedAt)}</div>
            </div>
          `;
        }).join('');
      }
    };

    function escapeHtml(text) {
      const div = document.createElement('div');
      div.appendChild(document.createTextNode(String(text)));
      return div.innerHTML;
    }

    // Keyboard shortcuts
    document.addEventListener('keydown', (e) => {
      if ((e.ctrlKey || e.metaKey) && e.key === 's') {
        e.preventDefault();
        app.saveNote();
      }
      if ((e.ctrlKey || e.metaKey) && e.key === 'n') {
        e.preventDefault();
        app.createNote();
      }
    });

    // Start the app
    app.init();
  </script>
</body>
</html>
```

---

## แบบฝึกหัด (Exercises)

### แบบฝึกหัดที่ 1: เพิ่มฟีเจอร์ Todo App
เพิ่มฟีเจอร์ต่อไปนี้ใน Todo App:
- Priority level (สูง/กลาง/ต่ำ) แสดงสีต่างกัน
- Due date picker
- Drag & Drop เพื่อ reorder
- Import/Export เป็น JSON file

### แบบฝึกหัดที่ 2: เพิ่มฟีเจอร์ Calculator
เพิ่มใน Calculator:
- Scientific mode (sin, cos, tan, sqrt, ลอก)
- History ของการคำนวณ
- Copy result to clipboard
- Keyboard number pad support

### แบบฝึกหัดที่ 3: เพิ่มฟีเจอร์ Quiz App
เพิ่มใน Quiz:
- เพิ่มคำถามใหม่ผ่าน UI
- Save high scores ลง localStorage
- Share result as image
- Timer bar แบบ visual

### แบบฝึกหัดที่ 4: เพิ่มฟีเจอร์ Weather App
เพิ่มใน Weather App:
- Favorite cities
- Unit switcher (°C / °F)
- UV Index
- Air quality index
- Hourly forecast

### แบบฝึกหัดที่ 5: เพิ่มฟีเจอร์ Note App
เพิ่มใน Note App:
- Markdown preview mode
- Tags/Categories
- Color themes สำหรับโน้ต
- Sync ระหว่าง tabs ด้วย localStorage events
- Print function

---

## สรุป Part 20 และ Beginner Course

ในบทนี้เราได้สร้างโปรเจค 5 โปรเจคที่สมบูรณ์:

### 1. Todo List App (Steps 371-374)
- CRUD operations ครบ
- Filter (all/pending/completed)
- LocalStorage persistence
- Animation และ responsive design

### 2. Calculator App (Steps 375-378)
- Operations พื้นฐาน (+, -, *, /)
- Keyboard support
- Format numbers ด้วย toLocaleString
- Error handling (divide by zero)
- Operator state management

### 3. Quiz App (Steps 379-382)
- Question randomization
- Timer ต่อคำถาม
- Score tracking
- Explanation สำหรับแต่ละข้อ
- Result summary

### 4. Weather App (Steps 383-386)
- Async/Await กับ Fetch API
- Real API integration (OpenWeather)
- Demo mode (ไม่ต้องการ API key)
- 5-day forecast
- Error handling

### 5. Note Taking App (Steps 387-390)
- Rich text editor (contenteditable)
- Real-time auto-save
- Search functionality
- Export to file
- Keyboard shortcuts

---

## สิ่งที่เรียนมาตลอด Beginner Course

ตลอด 20 Parts ที่ผ่านมา (Steps 1-390) เราได้เรียนรู้:

- **พื้นฐาน JavaScript**: Variables, Data Types, Operators
- **Control Flow**: if/else, switch, loops
- **Functions**: Declaration, Expression, Arrow, Parameters
- **Arrays**: Methods, forEach, map, filter, reduce
- **Objects**: Creation, Methods, Destructuring
- **ES6+**: Template literals, Spread, Rest, Modules
- **Async JS**: Callbacks, Promises, Async/Await
- **DOM**: Select, Modify, Events, Forms
- **JSON**: Parse, Stringify, Use cases
- **Web Storage**: localStorage, sessionStorage
- **Regular Expressions**: Patterns, Validation
- **Debugging**: DevTools, Console, Breakpoints
- **Projects**: 5 complete web applications

ขอแสดงความยินดีที่จบ Beginner Course! 🎉
