# Part 12: Events พื้นฐาน (Steps 211-230)

## บทนำ

Events คือสัญญาณที่เกิดขึ้นใน browser เมื่อมีการกระทำบางอย่าง เช่น คลิกเมาส์ กดแป้นพิมพ์ โหลดหน้าเว็บ ฯลฯ JavaScript สามารถ "ฟัง" events เหล่านี้และตอบสนองด้วย Event Handlers ทำให้เว็บไซต์มี Interactivity

---

## Step 211: What are Events

Events คือสิ่งที่เกิดขึ้นใน browser ที่ JavaScript สามารถตอบสนองได้

```html
<!DOCTYPE html>
<html lang="th">
<body>
  <button id="btn">คลิกฉัน</button>
  <div id="output"></div>

  <script>
    const btn = document.getElementById('btn');
    const output = document.getElementById('output');

    // วิธีที่ 1: HTML attribute (inline) - ไม่แนะนำ
    // <button onclick="handleClick()">คลิก</button>

    // วิธีที่ 2: DOM property
    // btn.onclick = function() { ... }

    // วิธีที่ 3: addEventListener (แนะนำที่สุด)
    btn.addEventListener('click', function(event) {
      output.textContent = `คลิกแล้ว! เวลา: ${new Date().toLocaleTimeString()}`;
      console.log('Event object:', event);
    });

    // ประเภทของ Events หลักๆ
    const eventTypes = {
      // Mouse Events
      mouse: ['click', 'dblclick', 'mousedown', 'mouseup', 'mousemove',
              'mouseover', 'mouseout', 'mouseenter', 'mouseleave', 'contextmenu'],

      // Keyboard Events
      keyboard: ['keydown', 'keyup', 'keypress'],

      // Form Events
      form: ['submit', 'change', 'input', 'focus', 'blur', 'reset'],

      // Document/Window Events
      window: ['load', 'DOMContentLoaded', 'resize', 'scroll', 'unload', 'beforeunload'],

      // Touch Events
      touch: ['touchstart', 'touchend', 'touchmove', 'touchcancel'],

      // Drag Events
      drag: ['dragstart', 'drag', 'dragend', 'dragover', 'dragenter', 'dragleave', 'drop'],

      // Media Events
      media: ['play', 'pause', 'ended', 'timeupdate', 'volumechange'],

      // Animation Events
      animation: ['animationstart', 'animationend', 'animationiteration'],

      // Transition Events
      transition: ['transitionstart', 'transitionend'],
    };

    console.log('Event types:', eventTypes);

    // Event Flow
    // 1. Capture Phase: จาก document ลงไปถึง target
    // 2. Target Phase: ถึง target element
    // 3. Bubble Phase: จาก target ขึ้นไปถึง document
  </script>
</body>
</html>
```

---

## Step 212: addEventListener และ removeEventListener

```html
<!DOCTYPE html>
<html lang="th">
<body>
  <button id="btn">Button</button>
  <button id="btn-once">คลิกได้ครั้งเดียว</button>
  <div id="hover-area" style="width:200px;height:100px;background:#eee;padding:10px;">
    Hover here
  </div>
  <div id="log"></div>

  <script>
    const btn = document.getElementById('btn');
    const btnOnce = document.getElementById('btn-once');
    const hoverArea = document.getElementById('hover-area');
    const log = document.getElementById('log');

    function addLog(msg) {
      const p = document.createElement('p');
      p.textContent = `${new Date().toLocaleTimeString()}: ${msg}`;
      log.prepend(p);
    }

    // addEventListener(type, listener, options)
    // type: ชื่อ event
    // listener: function ที่จะเรียกเมื่อ event เกิด
    // options: object หรือ boolean (useCapture)

    // เพิ่ม event listener
    function handleClick(e) {
      addLog(`คลิกที่ button (${e.type})`);
    }

    btn.addEventListener('click', handleClick);

    // เพิ่มหลาย listeners สำหรับ event เดียวกัน
    btn.addEventListener('click', (e) => {
      addLog('Second click listener');
    });

    // ลบ event listener (ต้องส่ง reference เดิม)
    function removeClickListener() {
      btn.removeEventListener('click', handleClick);
      addLog('Removed first click listener');
    }

    // สร้าง button ลบ listener
    const removeBtn = document.createElement('button');
    removeBtn.textContent = 'ลบ Listener แรก';
    removeBtn.addEventListener('click', removeClickListener);
    document.body.insertBefore(removeBtn, log);

    // Options object
    // capture: true = เรียก listener ใน capture phase
    // once: true = เรียก listener ครั้งเดียวแล้วลบอัตโนมัติ
    // passive: true = listener ไม่เรียก preventDefault (เพิ่ม performance)
    // signal: AbortSignal = ลบ listener เมื่อ signal abort

    // once option
    btnOnce.addEventListener('click', () => {
      addLog('นี่จะเกิดแค่ครั้งเดียว!');
    }, { once: true });

    // ลบด้วย AbortController
    const controller = new AbortController();
    hoverArea.addEventListener('mouseenter', () => {
      addLog('Mouse entered');
    }, { signal: controller.signal });

    hoverArea.addEventListener('mouseleave', () => {
      addLog('Mouse left');
    }, { signal: controller.signal });

    // สร้าง button ยกเลิก hover listeners
    const cancelBtn = document.createElement('button');
    cancelBtn.textContent = 'ยกเลิก Hover Listeners';
    cancelBtn.addEventListener('click', () => {
      controller.abort(); // ลบ listeners ทั้งหมดที่ใช้ signal นี้
      addLog('Hover listeners removed via AbortController');
    });
    document.body.insertBefore(cancelBtn, log);

    // Arrow function ไม่สามารถ removeEventListener ได้ (no reference)
    // btn.addEventListener('click', () => {}); // ลบไม่ได้!

    // ต้องเก็บ reference
    const namedHandler = () => addLog('Named arrow function');
    btn.addEventListener('click', namedHandler);
    // btn.removeEventListener('click', namedHandler); // ลบได้
  </script>
</body>
</html>
```

---

## Step 213: Event Object

```html
<!DOCTYPE html>
<html lang="th">
<body>
  <div id="box" style="width:300px;height:200px;background:#e0e0ff;cursor:pointer;">
    คลิกหรือ hover ที่นี่
  </div>
  <button id="btn" type="button">Button</button>
  <div id="info" style="margin-top:10px;font-family:monospace;"></div>

  <script>
    const box = document.getElementById('box');
    const btn = document.getElementById('btn');
    const info = document.getElementById('info');

    function showInfo(label, data) {
      info.innerHTML = `<strong>${label}</strong><pre>${JSON.stringify(data, null, 2)}</pre>`;
    }

    // Event Object Properties
    box.addEventListener('click', (e) => {
      showInfo('Click Event', {
        // ประเภทของ event
        type: e.type,            // "click"

        // Element ที่ถูก click
        target: e.target.tagName,        // "DIV"
        targetId: e.target.id,           // "box"

        // Element ที่ listener ถูก attach
        currentTarget: e.currentTarget.tagName, // "DIV" (same as target ในกรณีนี้)

        // Bubbling
        bubbles: e.bubbles,       // true (click bubbles)
        cancelable: e.cancelable, // true

        // Timestamp
        timeStamp: Math.round(e.timeStamp), // ms จากเริ่ม page load

        // Phase
        eventPhase: e.eventPhase, // 2 = AT_TARGET

        // Mouse position
        clientX: e.clientX, clientY: e.clientY, // relative to viewport
        pageX: e.pageX, pageY: e.pageY,         // relative to page
        screenX: e.screenX, screenY: e.screenY, // relative to screen
        offsetX: e.offsetX, offsetY: e.offsetY, // relative to element

        // Mouse buttons (0=left, 1=middle, 2=right)
        button: e.button,
        buttons: e.buttons, // bitmask ของ buttons ที่กด

        // Modifier keys
        ctrlKey: e.ctrlKey,
        altKey: e.altKey,
        shiftKey: e.shiftKey,
        metaKey: e.metaKey, // Cmd key บน Mac

        // Related target (สำหรับ mouseover/out)
        relatedTarget: e.relatedTarget ? e.relatedTarget.tagName : null,
      });
    });

    // isTrusted - ตรวจว่า event จาก user จริงๆ ไม่ใช่ dispatch ด้วย code
    btn.addEventListener('click', (e) => {
      console.log('isTrusted:', e.isTrusted); // true ถ้า user คลิก
    });

    // Dispatch event จาก code
    btn.dispatchEvent(new MouseEvent('click'));
    // isTrusted จะเป็น false

    // Event Phases
    // 1 = CAPTURING_PHASE
    // 2 = AT_TARGET
    // 3 = BUBBLING_PHASE
    console.log(Event.CAPTURING_PHASE); // 1
    console.log(Event.AT_TARGET);       // 2
    console.log(Event.BUBBLING_PHASE);  // 3

    // mousemove - track mouse position
    box.addEventListener('mousemove', (e) => {
      const rect = box.getBoundingClientRect();
      const x = e.clientX - rect.left;
      const y = e.clientY - rect.top;
      box.textContent = `X: ${Math.round(x)}, Y: ${Math.round(y)}`;
    });

    box.addEventListener('mouseleave', () => {
      box.textContent = 'คลิกหรือ hover ที่นี่';
    });
  </script>
</body>
</html>
```

---

## Step 214: Event Propagation - Bubbling และ Capturing

```html
<!DOCTYPE html>
<html lang="th">
<head>
  <style>
    #grandparent { padding: 20px; background: #ffcccc; }
    #parent { padding: 20px; background: #ccffcc; }
    #child { padding: 20px; background: #ccccff; cursor: pointer; }
    #log { margin-top: 10px; }
    .log-entry { padding: 3px; margin: 2px; border-radius: 3px; }
    .bubble { background: #fff3cd; }
    .capture { background: #d4edda; }
  </style>
</head>
<body>
  <div id="grandparent">
    Grandparent
    <div id="parent">
      Parent
      <div id="child">
        Child - คลิกที่นี่
      </div>
    </div>
  </div>
  <div id="log"></div>
  <button id="clear">Clear Log</button>

  <script>
    const log = document.getElementById('log');
    const grandparent = document.getElementById('grandparent');
    const parent = document.getElementById('parent');
    const child = document.getElementById('child');

    function addLog(msg, type = 'bubble') {
      const div = document.createElement('div');
      div.className = `log-entry ${type}`;
      div.textContent = msg;
      log.appendChild(div);
    }

    // Bubbling Phase (default) - เหตุการณ์จาก child ขึ้นไป parent
    grandparent.addEventListener('click', (e) => {
      addLog(`Bubble: Grandparent clicked (target: ${e.target.id})`, 'bubble');
    });

    parent.addEventListener('click', (e) => {
      addLog(`Bubble: Parent clicked (target: ${e.target.id})`, 'bubble');
    });

    child.addEventListener('click', (e) => {
      addLog(`Bubble: Child clicked (target: ${e.target.id})`, 'bubble');
    });

    // Capture Phase - เหตุการณ์จาก document ลงมา child
    grandparent.addEventListener('click', (e) => {
      addLog(`Capture: Grandparent (phase: ${e.eventPhase})`, 'capture');
    }, true); // true = useCapture

    parent.addEventListener('click', (e) => {
      addLog(`Capture: Parent (phase: ${e.eventPhase})`, 'capture');
    }, { capture: true });

    child.addEventListener('click', (e) => {
      addLog(`Capture: Child (phase: ${e.eventPhase})`, 'capture');
    }, { capture: true });

    // ลำดับที่ click child:
    // 1. Capture Grandparent (phase: 1)
    // 2. Capture Parent (phase: 1)
    // 3. Capture Child (phase: 2 = AT_TARGET)
    // 4. Bubble Child (phase: 2 = AT_TARGET)
    // 5. Bubble Parent (phase: 3)
    // 6. Bubble Grandparent (phase: 3)

    document.getElementById('clear').addEventListener('click', () => {
      log.innerHTML = '';
    });

    // ตัวอย่างเพิ่มเติม: click propagation
    addLog('--- คลิก Child เพื่อดู propagation order ---');
    addLog('สีเหลือง = Bubble, สีเขียว = Capture');
  </script>
</body>
</html>
```

---

## Step 215: stopPropagation และ stopImmediatePropagation

```html
<!DOCTYPE html>
<html lang="th">
<body>
  <div id="outer" style="padding:20px;background:#ffd;border:1px solid #ccc;">
    Outer
    <div id="inner" style="padding:20px;background:#dff;border:1px solid #ccc;margin-top:10px;">
      Inner - คลิกที่นี่
    </div>
  </div>
  <div id="output"></div>

  <script>
    const outer = document.getElementById('outer');
    const inner = document.getElementById('inner');
    const output = document.getElementById('output');

    let logs = [];
    function log(msg) {
      logs.push(msg);
      output.innerHTML = logs.map(l => `<p>${l}</p>`).join('');
    }

    // stopPropagation - หยุด event ไม่ให้ bubble ขึ้นไป parent
    inner.addEventListener('click', (e) => {
      log('Inner clicked');
      e.stopPropagation(); // หยุด bubbling
      // outer จะไม่ได้รับ event นี้
    });

    outer.addEventListener('click', (e) => {
      log('Outer clicked (จะไม่เห็น ถ้าคลิก inner)');
    });

    document.addEventListener('click', (e) => {
      log('Document clicked (จะไม่เห็น ถ้า inner stop propagation)');
    });

    // stopImmediatePropagation - หยุดทั้ง bubbling AND listener อื่นๆ บน element เดียวกัน
    const btn = document.createElement('button');
    btn.textContent = 'Test stopImmediatePropagation';
    document.body.appendChild(btn);

    btn.addEventListener('click', (e) => {
      log('Listener 1: ทำงาน');
      e.stopImmediatePropagation();
    });

    btn.addEventListener('click', (e) => {
      log('Listener 2: จะไม่เห็นข้อความนี้!');
    });

    btn.addEventListener('click', (e) => {
      log('Listener 3: จะไม่เห็นข้อความนี้เช่นกัน!');
    });

    // stopPropagation vs stopImmediatePropagation
    // stopPropagation: หยุด bubble ขึ้น parent แต่ listener อื่นบน element เดิมยังทำงาน
    // stopImmediatePropagation: หยุดทุกอย่าง รวมถึง listener อื่นบน element เดิม

    const clearBtn = document.createElement('button');
    clearBtn.textContent = 'Clear';
    clearBtn.addEventListener('click', () => {
      logs = [];
      output.innerHTML = '';
    });
    document.body.appendChild(clearBtn);
  </script>
</body>
</html>
```

---

## Step 216: preventDefault

```html
<!DOCTYPE html>
<html lang="th">
<body>
  <!-- Form ที่ prevent default submission -->
  <form id="login-form">
    <input type="text" id="username" placeholder="Username" required>
    <input type="password" id="password" placeholder="Password" required>
    <button type="submit">Login</button>
  </form>

  <!-- Link ที่ prevent default navigation -->
  <a href="https://example.com" id="custom-link">ไปยัง example.com (intercepted)</a>

  <!-- Checkbox ที่ prevent default check -->
  <label>
    <input type="checkbox" id="confirm-check">
    ฉันยอมรับเงื่อนไข (ต้องอ่านก่อน)
  </label>
  <p id="warning" style="color:red;display:none">กรุณาอ่านเงื่อนไขก่อน!</p>

  <div id="output"></div>

  <script>
    const output = document.getElementById('output');
    function log(msg) {
      const p = document.createElement('p');
      p.textContent = msg;
      output.prepend(p);
    }

    // preventDefault กับ Form Submit
    const form = document.getElementById('login-form');
    form.addEventListener('submit', (e) => {
      e.preventDefault(); // ป้องกันการ submit ไป server
      const username = document.getElementById('username').value;
      const password = document.getElementById('password').value;

      if (!username || !password) {
        log('กรุณากรอกข้อมูลให้ครบ');
        return;
      }

      log(`Logging in as: ${username}`);
      // ส่งข้อมูลด้วย fetch แทน
    });

    // preventDefault กับ Link
    const link = document.getElementById('custom-link');
    link.addEventListener('click', (e) => {
      e.preventDefault(); // ป้องกัน navigation
      const confirmed = confirm('แน่ใจหรือไม่ที่จะออกจากหน้านี้?');
      if (confirmed) {
        window.location.href = link.href; // navigate manually
      } else {
        log('ยกเลิกการไปยัง ' + link.href);
      }
    });

    // preventDefault กับ Checkbox
    let hasReadTerms = false;
    const checkbox = document.getElementById('confirm-check');
    const warning = document.getElementById('warning');

    checkbox.addEventListener('click', (e) => {
      if (!hasReadTerms) {
        e.preventDefault(); // ป้องกันการ check
        warning.style.display = 'block';
        setTimeout(() => {
          hasReadTerms = true;
          warning.style.display = 'none';
          checkbox.checked = true; // check manually หลัง 2 วินาที
          log('ตอนนี้ยอมรับเงื่อนไขได้แล้ว');
        }, 2000);
      }
    });

    // preventDefault กับ Context Menu
    document.addEventListener('contextmenu', (e) => {
      e.preventDefault();
      log('Right-click intercepted! No default context menu.');
    });

    // preventDefault กับ Drag
    document.addEventListener('dragover', (e) => {
      e.preventDefault(); // จำเป็นเพื่อให้ drop ทำงานได้
    });

    // ตรวจสอบว่า event สามารถ preventDefault ได้ไหม
    document.addEventListener('scroll', (e) => {
      console.log('Can cancel scroll:', e.cancelable); // false บน modern browsers
      // scroll event ใน passive listeners ไม่สามารถ preventDefault
    }, { passive: true });

    // cancelable property
    const clickEvent = new MouseEvent('click', { cancelable: true });
    console.log(clickEvent.cancelable); // true

    const nonCancelable = new Event('load', { cancelable: false });
    console.log(nonCancelable.cancelable); // false
  </script>
</body>
</html>
```

---

## Step 217: Mouse Events

```html
<!DOCTYPE html>
<html lang="th">
<head>
  <style>
    #canvas {
      width: 400px; height: 300px;
      background: #f5f5f5;
      border: 2px solid #333;
      cursor: crosshair;
      user-select: none;
    }
    #paint-canvas {
      width: 400px; height: 300px;
      background: white;
      border: 1px solid #ccc;
      cursor: crosshair;
    }
  </style>
</head>
<body>
  <h2>Mouse Events</h2>
  <div id="canvas">
    <p id="mouse-info">Move mouse here</p>
  </div>

  <h2>Drawing Canvas</h2>
  <canvas id="paint-canvas" width="400" height="300"></canvas>
  <button id="clear-canvas">Clear</button>

  <div id="right-click-menu" style="display:none;position:fixed;background:#fff;border:1px solid #ccc;padding:5px;z-index:1000;">
    <div class="menu-item">คัดลอก</div>
    <div class="menu-item">วาง</div>
    <div class="menu-item">ลบ</div>
  </div>

  <script>
    const mouseDiv = document.getElementById('canvas');
    const mouseInfo = document.getElementById('mouse-info');
    const contextMenu = document.getElementById('right-click-menu');

    // click - single click
    mouseDiv.addEventListener('click', (e) => {
      mouseInfo.textContent = `Click at (${e.offsetX}, ${e.offsetY})`;
    });

    // dblclick - double click
    mouseDiv.addEventListener('dblclick', (e) => {
      mouseInfo.textContent = `Double Click at (${e.offsetX}, ${e.offsetY})!`;
      mouseInfo.style.fontSize = '20px';
      setTimeout(() => mouseInfo.style.fontSize = '', 500);
    });

    // mousedown - กดปุ่มเมาส์
    mouseDiv.addEventListener('mousedown', (e) => {
      const buttons = { 0: 'ซ้าย', 1: 'กลาง', 2: 'ขวา' };
      mouseInfo.textContent = `MouseDown: ปุ่ม${buttons[e.button]} ที่ (${e.offsetX}, ${e.offsetY})`;
    });

    // mouseup - ปล่อยปุ่มเมาส์
    mouseDiv.addEventListener('mouseup', (e) => {
      mouseInfo.textContent = `MouseUp at (${e.offsetX}, ${e.offsetY})`;
    });

    // mousemove - เคลื่อนเมาส์
    mouseDiv.addEventListener('mousemove', (e) => {
      mouseDiv.style.background = `hsl(${e.offsetX}, 70%, 90%)`;
      mouseInfo.textContent = `Move: (${e.offsetX}, ${e.offsetY})`;
    });

    // mouseover vs mouseenter
    // mouseover: bubbles, เกิดเมื่อ enter ตัว element หรือ children
    // mouseenter: ไม่ bubble, เกิดเมื่อ enter ตัว element เท่านั้น
    mouseDiv.addEventListener('mouseenter', () => {
      mouseDiv.style.border = '3px solid blue';
    });

    mouseDiv.addEventListener('mouseleave', () => {
      mouseDiv.style.border = '2px solid #333';
      mouseDiv.style.background = '#f5f5f5';
    });

    // Custom Context Menu
    mouseDiv.addEventListener('contextmenu', (e) => {
      e.preventDefault();
      contextMenu.style.display = 'block';
      contextMenu.style.left = e.clientX + 'px';
      contextMenu.style.top = e.clientY + 'px';
    });

    document.addEventListener('click', () => {
      contextMenu.style.display = 'none';
    });

    // Drawing with canvas
    const canvas = document.getElementById('paint-canvas');
    const ctx = canvas.getContext('2d');
    let isDrawing = false;
    let lastX = 0, lastY = 0;

    canvas.addEventListener('mousedown', (e) => {
      isDrawing = true;
      [lastX, lastY] = [e.offsetX, e.offsetY];
    });

    canvas.addEventListener('mousemove', (e) => {
      if (!isDrawing) return;
      ctx.beginPath();
      ctx.moveTo(lastX, lastY);
      ctx.lineTo(e.offsetX, e.offsetY);
      ctx.stroke();
      [lastX, lastY] = [e.offsetX, e.offsetY];
    });

    canvas.addEventListener('mouseup', () => isDrawing = false);
    canvas.addEventListener('mouseleave', () => isDrawing = false);

    document.getElementById('clear-canvas').addEventListener('click', () => {
      ctx.clearRect(0, 0, canvas.width, canvas.height);
    });
  </script>
</body>
</html>
```

---

## Step 218: Keyboard Events

```html
<!DOCTYPE html>
<html lang="th">
<head>
  <style>
    #key-display {
      font-size: 24px;
      padding: 20px;
      background: #f0f0f0;
      text-align: center;
      margin: 10px 0;
      border-radius: 8px;
      min-height: 60px;
    }
    .key-badge {
      display: inline-block;
      background: #333;
      color: white;
      padding: 5px 10px;
      border-radius: 4px;
      margin: 2px;
      font-family: monospace;
    }
    #shortcut-demo { border: 1px solid #ccc; padding: 10px; }
    #game-area {
      width: 400px; height: 200px;
      background: #e8f4f8;
      position: relative;
      border: 2px solid #333;
      overflow: hidden;
    }
    #player {
      width: 30px; height: 30px;
      background: blue;
      position: absolute;
      border-radius: 50%;
    }
  </style>
</head>
<body>
  <h2>Keyboard Events</h2>

  <input type="text" id="key-input" placeholder="พิมพ์อะไรก็ได้...">
  <div id="key-display">กดแป้นพิมพ์</div>

  <div id="shortcut-demo">
    <p>Shortcuts:</p>
    <p>Ctrl+S = Save | Ctrl+Z = Undo | Ctrl+A = Select All</p>
    <p id="shortcut-result">ลอง shortcut...</p>
  </div>

  <h2>Game Demo (Arrow keys)</h2>
  <div id="game-area">
    <div id="player"></div>
  </div>

  <script>
    const keyInput = document.getElementById('key-input');
    const keyDisplay = document.getElementById('key-display');
    const shortcutResult = document.getElementById('shortcut-result');
    const player = document.getElementById('player');
    const gameArea = document.getElementById('game-area');

    // keydown - เกิดเมื่อกดปุ่ม (เกิดซ้ำถ้าค้างกด)
    // keyup - เกิดเมื่อปล่อยปุ่ม
    // keypress - deprecated (ไม่แนะนำใช้)

    // key properties:
    // e.key - human-readable key name ("a", "A", "Enter", "ArrowLeft", " ")
    // e.code - physical key ("KeyA", "Enter", "ArrowLeft", "Space")
    // e.keyCode - deprecated
    // e.which - deprecated

    keyInput.addEventListener('keydown', (e) => {
      const modifiers = [];
      if (e.ctrlKey) modifiers.push('Ctrl');
      if (e.altKey) modifiers.push('Alt');
      if (e.shiftKey) modifiers.push('Shift');
      if (e.metaKey) modifiers.push('Meta/Cmd');

      const keys = [...modifiers, e.key].join('+');
      const html = `
        <span class="key-badge">${e.key}</span>
        Code: <span class="key-badge">${e.code}</span>
        Combo: <span class="key-badge">${keys}</span>
        Repeat: ${e.repeat}
      `;
      keyDisplay.innerHTML = html;
    });

    // Keyboard Shortcuts
    document.addEventListener('keydown', (e) => {
      if (e.ctrlKey && e.key === 's') {
        e.preventDefault();
        shortcutResult.textContent = 'Ctrl+S: Saved!';
      }
      if (e.ctrlKey && e.key === 'z') {
        e.preventDefault();
        shortcutResult.textContent = 'Ctrl+Z: Undone!';
      }
      if (e.ctrlKey && e.key === 'a') {
        e.preventDefault();
        shortcutResult.textContent = 'Ctrl+A: Select All!';
      }
      if (e.key === 'Escape') {
        shortcutResult.textContent = 'Escape pressed';
      }
    });

    // ตรวจสอบการกดปุ่ม
    function isHotkey(event, combo) {
      // combo: "ctrl+s", "alt+f4", "shift+enter"
      const parts = combo.toLowerCase().split('+');
      const key = parts[parts.length - 1];
      const modifiers = parts.slice(0, -1);

      if (event.key.toLowerCase() !== key && event.code.toLowerCase() !== key) return false;
      if (modifiers.includes('ctrl') !== event.ctrlKey) return false;
      if (modifiers.includes('alt') !== event.altKey) return false;
      if (modifiers.includes('shift') !== event.shiftKey) return false;
      if (modifiers.includes('meta') !== event.metaKey) return false;

      return true;
    }

    // Game - move player with arrow keys
    let x = 10, y = 80;
    const speed = 5;
    const keys = {};

    document.addEventListener('keydown', (e) => {
      keys[e.code] = true;
    });

    document.addEventListener('keyup', (e) => {
      keys[e.code] = false;
    });

    function gameLoop() {
      if (keys['ArrowLeft'] || keys['KeyA']) x -= speed;
      if (keys['ArrowRight'] || keys['KeyD']) x += speed;
      if (keys['ArrowUp'] || keys['KeyW']) y -= speed;
      if (keys['ArrowDown'] || keys['KeyS']) y += speed;

      const maxX = gameArea.clientWidth - player.clientWidth;
      const maxY = gameArea.clientHeight - player.clientHeight;
      x = Math.max(0, Math.min(x, maxX));
      y = Math.max(0, Math.min(y, maxY));

      player.style.left = x + 'px';
      player.style.top = y + 'px';

      requestAnimationFrame(gameLoop);
    }

    gameLoop();

    // ป้องกัน special keys ในบาง inputs
    const numbersOnly = document.createElement('input');
    numbersOnly.placeholder = 'ตัวเลขเท่านั้น';
    numbersOnly.addEventListener('keydown', (e) => {
      const allowedKeys = ['Backspace', 'Delete', 'ArrowLeft', 'ArrowRight', 'Tab'];
      if (allowedKeys.includes(e.key)) return;
      if (/^\d$/.test(e.key)) return;
      e.preventDefault();
    });
    document.body.appendChild(numbersOnly);
  </script>
</body>
</html>
```

---

## Step 219: Form Events

```html
<!DOCTYPE html>
<html lang="th">
<head>
  <style>
    .form-group { margin: 10px 0; }
    label { display: block; font-weight: bold; }
    input, select, textarea { width: 100%; padding: 8px; box-sizing: border-box; }
    input:focus { outline: 2px solid blue; }
    .focused { background: #fffce0; }
    #live-preview { background: #f5f5f5; padding: 10px; border-radius: 4px; }
  </style>
</head>
<body>
  <form id="profile-form">
    <div class="form-group">
      <label for="name">ชื่อ:</label>
      <input type="text" id="name" placeholder="กรอกชื่อ">
    </div>
    <div class="form-group">
      <label for="email">Email:</label>
      <input type="email" id="email" placeholder="example@email.com">
    </div>
    <div class="form-group">
      <label for="bio">ประวัติ:</label>
      <textarea id="bio" rows="3" placeholder="เล่าเรื่องตัวเอง..."></textarea>
    </div>
    <div class="form-group">
      <label for="country">ประเทศ:</label>
      <select id="country">
        <option value="">-- เลือกประเทศ --</option>
        <option value="th">ไทย</option>
        <option value="jp">ญี่ปุ่น</option>
        <option value="us">สหรัฐอเมริกา</option>
      </select>
    </div>
    <button type="submit">บันทึก</button>
    <button type="reset">ล้างข้อมูล</button>
  </form>

  <div id="live-preview">
    <h3>Preview:</h3>
    <p id="preview-content">กรอกข้อมูลเพื่อดู preview...</p>
  </div>

  <script>
    const form = document.getElementById('profile-form');
    const nameInput = document.getElementById('name');
    const emailInput = document.getElementById('email');
    const bioInput = document.getElementById('bio');
    const countrySelect = document.getElementById('country');
    const preview = document.getElementById('preview-content');

    // focus - element ได้รับ focus
    nameInput.addEventListener('focus', (e) => {
      e.target.parentElement.classList.add('focused');
      console.log('Name field focused');
    });

    // blur - element เสีย focus
    nameInput.addEventListener('blur', (e) => {
      e.target.parentElement.classList.remove('focused');
      // Validate เมื่อออกจาก field
      if (!e.target.value.trim()) {
        e.target.style.border = '1px solid red';
      } else {
        e.target.style.border = '';
      }
    });

    // input - เกิดทุกครั้งที่ค่าเปลี่ยน (realtime)
    function updatePreview() {
      const data = {
        ชื่อ: nameInput.value || '-',
        email: emailInput.value || '-',
        ประวัติ: bioInput.value || '-',
        ประเทศ: countrySelect.options[countrySelect.selectedIndex]?.text || '-',
      };
      preview.innerHTML = Object.entries(data)
        .map(([k, v]) => `<strong>${k}:</strong> ${v}`)
        .join(' | ');
    }

    nameInput.addEventListener('input', updatePreview);
    emailInput.addEventListener('input', updatePreview);
    bioInput.addEventListener('input', updatePreview);

    // change - เกิดเมื่อค่าเปลี่ยนและ blur (สำหรับ text inputs)
    // สำหรับ select, checkbox, radio: เกิดทันที
    countrySelect.addEventListener('change', (e) => {
      console.log('Country changed to:', e.target.value);
      updatePreview();
    });

    emailInput.addEventListener('change', (e) => {
      console.log('Email changed (after blur):', e.target.value);
    });

    // input vs change:
    // input: เกิดทุก keystroke (realtime)
    // change: เกิดเมื่อค่าเปลี่ยนและออกจาก field

    // submit - form submission
    form.addEventListener('submit', (e) => {
      e.preventDefault();
      const formData = new FormData(form);
      const data = Object.fromEntries(formData);
      console.log('Form submitted:', data);
      alert('บันทึกข้อมูลแล้ว!\n' + JSON.stringify(data, null, 2));
    });

    // reset - form reset
    form.addEventListener('reset', (e) => {
      console.log('Form reset');
      setTimeout(updatePreview, 0); // update หลัง reset เสร็จ
    });

    // focusin/focusout - bubbles (focus/blur ไม่ bubble)
    form.addEventListener('focusin', (e) => {
      if (e.target.tagName === 'INPUT' || e.target.tagName === 'TEXTAREA') {
        e.target.style.backgroundColor = '#fffff0';
      }
    });

    form.addEventListener('focusout', (e) => {
      e.target.style.backgroundColor = '';
    });
  </script>
</body>
</html>
```

---

## Step 220: Window Events

```html
<!DOCTYPE html>
<html lang="th">
<body>
  <div id="loading" style="display:none;text-align:center;padding:20px;">Loading...</div>
  <div id="content">
    <h1>Window Events Demo</h1>
    <div id="size-info"></div>
    <div id="scroll-info"></div>
    <div style="height:2000px;background:linear-gradient(to bottom, #fff, #ddd);">
      Scroll down...
    </div>
  </div>

  <div id="back-to-top" style="
    display:none;
    position:fixed;
    bottom:20px;
    right:20px;
    background:blue;
    color:white;
    padding:10px 15px;
    border-radius:50px;
    cursor:pointer;
  ">↑ Top</div>

  <div id="scroll-progress" style="
    position:fixed;
    top:0;left:0;
    height:4px;
    background:blue;
    width:0%;
    transition:width 0.1s;
  "></div>

  <script>
    const sizeInfo = document.getElementById('size-info');
    const scrollInfo = document.getElementById('scroll-info');
    const backToTop = document.getElementById('back-to-top');
    const scrollProgress = document.getElementById('scroll-progress');

    // DOMContentLoaded - DOM พร้อมแล้ว (ก่อน images/CSS load)
    document.addEventListener('DOMContentLoaded', () => {
      console.log('DOM ready! (DOMContentLoaded)');
      // ปลอดภัยที่จะ access DOM
    });

    // load - ทุกอย่างโหลดเสร็จแล้ว (รวม images, CSS, scripts)
    window.addEventListener('load', () => {
      console.log('All resources loaded! (load)');
      document.getElementById('loading').style.display = 'none';
    });

    // resize - window ขนาดเปลี่ยน
    function updateSize() {
      sizeInfo.textContent = `Window: ${window.innerWidth}x${window.innerHeight} | Screen: ${screen.width}x${screen.height}`;
    }
    updateSize();

    // Debounce resize event เพื่อ performance
    let resizeTimer;
    window.addEventListener('resize', () => {
      clearTimeout(resizeTimer);
      resizeTimer = setTimeout(updateSize, 100);
    });

    // scroll - scroll เกิดขึ้น
    window.addEventListener('scroll', () => {
      const scrollY = window.scrollY || window.pageYOffset;
      const scrollX = window.scrollX || window.pageXOffset;
      const docHeight = document.documentElement.scrollHeight - window.innerHeight;
      const progress = docHeight > 0 ? (scrollY / docHeight) * 100 : 0;

      scrollInfo.textContent = `Scroll: Y=${Math.round(scrollY)}, X=${Math.round(scrollX)}`;
      scrollProgress.style.width = progress + '%';

      // Back to top button
      backToTop.style.display = scrollY > 300 ? 'block' : 'none';
    });

    // Back to top
    backToTop.addEventListener('click', () => {
      window.scrollTo({ top: 0, behavior: 'smooth' });
    });

    // beforeunload - ก่อนออกจากหน้า
    let hasUnsavedChanges = true;
    window.addEventListener('beforeunload', (e) => {
      if (hasUnsavedChanges) {
        e.preventDefault();
        e.returnValue = ''; // จำเป็น (Chrome)
        return 'มีข้อมูลที่ยังไม่ได้บันทึก ต้องการออกจากหน้านี้หรือไม่?';
      }
    });

    // hashchange - URL hash เปลี่ยน
    window.addEventListener('hashchange', (e) => {
      console.log('Hash changed from', e.oldURL, 'to', e.newURL);
      console.log('Current hash:', location.hash);
    });

    // online/offline - network status
    window.addEventListener('online', () => {
      console.log('Back online!');
      document.body.style.background = '#fff';
    });

    window.addEventListener('offline', () => {
      console.log('Offline!');
      document.body.style.background = '#fdd';
    });

    console.log('Navigator online:', navigator.onLine);

    // visibilitychange - tab ถูก switch
    document.addEventListener('visibilitychange', () => {
      if (document.hidden) {
        console.log('Page hidden - pause animations etc.');
        document.title = '(Away) - DOM Events';
      } else {
        console.log('Page visible again!');
        document.title = 'DOM Events Demo';
      }
    });
  </script>
</body>
</html>
```

---

## Step 221: Touch Events

```html
<!DOCTYPE html>
<html lang="th">
<head>
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <style>
    #touch-area {
      width: 100%;
      height: 300px;
      background: #e8f4f8;
      border: 2px solid #333;
      touch-action: none; /* ป้องกัน browser default touch behaviors */
      position: relative;
    }
    .touch-point {
      position: absolute;
      width: 40px; height: 40px;
      background: rgba(255, 100, 100, 0.7);
      border-radius: 50%;
      transform: translate(-50%, -50%);
      pointer-events: none;
    }
    #swipe-demo {
      width: 300px; height: 200px;
      background: #c8e6c9;
      border: 2px solid #333;
      display: flex; align-items: center; justify-content: center;
      font-size: 20px;
      user-select: none;
    }
  </style>
</head>
<body>
  <h2>Touch Events</h2>
  <div id="touch-area">Touch here (ใช้ touch screen หรือ DevTools mobile mode)</div>

  <h2>Swipe Demo</h2>
  <div id="swipe-demo">Swipe เพื่อเปลี่ยน</div>

  <div id="touch-log"></div>

  <script>
    const touchArea = document.getElementById('touch-area');
    const swipeDemo = document.getElementById('swipe-demo');
    const touchLog = document.getElementById('touch-log');
    const touchPoints = {};
    let colors = ['เขียว', 'น้ำเงิน', 'เหลือง', 'แดง'];
    let colorIndex = 0;

    function log(msg) {
      const p = document.createElement('p');
      p.textContent = msg;
      touchLog.prepend(p);
      if (touchLog.children.length > 5) touchLog.lastChild.remove();
    }

    // touchstart - เมื่อสัมผัส
    touchArea.addEventListener('touchstart', (e) => {
      e.preventDefault(); // ป้องกัน scroll
      for (const touch of e.changedTouches) {
        const point = document.createElement('div');
        point.className = 'touch-point';
        point.id = `touch-${touch.identifier}`;
        touchArea.appendChild(point);
        touchPoints[touch.identifier] = point;
        updateTouchPoint(touch);
      }
      log(`touchstart: ${e.touches.length} touch(es)`);
    }, { passive: false });

    // touchmove - เมื่อเลื่อนนิ้ว
    touchArea.addEventListener('touchmove', (e) => {
      e.preventDefault();
      for (const touch of e.changedTouches) {
        updateTouchPoint(touch);
      }
    }, { passive: false });

    // touchend - เมื่อยกนิ้ว
    touchArea.addEventListener('touchend', (e) => {
      for (const touch of e.changedTouches) {
        const point = touchPoints[touch.identifier];
        if (point) {
          point.remove();
          delete touchPoints[touch.identifier];
        }
      }
      log(`touchend: ${e.touches.length} touch(es) remaining`);
    });

    // touchcancel - เมื่อ touch ถูก interrupt (เช่น phone call)
    touchArea.addEventListener('touchcancel', (e) => {
      for (const touch of e.changedTouches) {
        const point = touchPoints[touch.identifier];
        if (point) {
          point.remove();
          delete touchPoints[touch.identifier];
        }
      }
      log('touchcancel!');
    });

    function updateTouchPoint(touch) {
      const rect = touchArea.getBoundingClientRect();
      const x = touch.clientX - rect.left;
      const y = touch.clientY - rect.top;
      const point = touchPoints[touch.identifier];
      if (point) {
        point.style.left = x + 'px';
        point.style.top = y + 'px';
      }
    }

    // Touch object properties:
    // touch.identifier - unique id
    // touch.clientX/Y - position relative to viewport
    // touch.pageX/Y - position relative to page
    // touch.screenX/Y - position relative to screen
    // touch.radiusX/Y, touch.rotationAngle - touch size/rotation
    // touch.force - pressure (0-1) on 3D Touch devices

    // Swipe detection
    let startX, startY;
    swipeDemo.addEventListener('touchstart', (e) => {
      startX = e.touches[0].clientX;
      startY = e.touches[0].clientY;
    });

    swipeDemo.addEventListener('touchend', (e) => {
      const dx = e.changedTouches[0].clientX - startX;
      const dy = e.changedTouches[0].clientY - startY;
      const absDx = Math.abs(dx);
      const absDy = Math.abs(dy);

      if (Math.max(absDx, absDy) < 30) return; // Too small

      let direction;
      if (absDx > absDy) {
        direction = dx > 0 ? 'ขวา →' : '← ซ้าย';
      } else {
        direction = dy > 0 ? 'ลง ↓' : '↑ ขึ้น';
      }

      colorIndex = (colorIndex + 1) % colors.length;
      swipeDemo.textContent = `Swipe ${direction}: ${colors[colorIndex]}`;
    });
  </script>
</body>
</html>
```

---

## Step 222: Custom Events

```html
<!DOCTYPE html>
<html lang="th">
<body>
  <div id="shopping-cart">
    <h2>ตะกร้าสินค้า (<span id="cart-count">0</span> รายการ)</h2>
    <ul id="cart-items"></ul>
    <p>รวม: <strong id="cart-total">0</strong> บาท</p>
  </div>

  <div id="products">
    <h2>สินค้า</h2>
    <div class="product" data-id="1" data-name="กาแฟ" data-price="80">
      กาแฟ - 80 บาท
      <button class="add-to-cart">เพิ่มลงตะกร้า</button>
    </div>
    <div class="product" data-id="2" data-name="เค้ก" data-price="120">
      เค้ก - 120 บาท
      <button class="add-to-cart">เพิ่มลงตะกร้า</button>
    </div>
  </div>

  <div id="notifications"></div>

  <script>
    // สร้าง Custom Event ด้วย CustomEvent constructor
    // CustomEvent(type, options)
    // options: { detail, bubbles, cancelable }

    // Event: add-to-cart
    function createAddToCartEvent(product) {
      return new CustomEvent('add-to-cart', {
        detail: { product },
        bubbles: true,
        cancelable: true,
      });
    }

    // Event: cart-updated
    function createCartUpdatedEvent(cart) {
      return new CustomEvent('cart-updated', {
        detail: { cart, total: cart.reduce((sum, item) => sum + item.price * item.qty, 0) },
        bubbles: false,
      });
    }

    // Cart state
    let cart = [];

    // Listen to custom events
    document.addEventListener('add-to-cart', (e) => {
      const { product } = e.detail;
      const existing = cart.find(item => item.id === product.id);

      if (existing) {
        existing.qty++;
      } else {
        cart.push({ ...product, qty: 1 });
      }

      // Dispatch another custom event
      document.dispatchEvent(createCartUpdatedEvent(cart));
      showNotification(`เพิ่ม "${product.name}" ลงตะกร้าแล้ว`);
    });

    document.addEventListener('cart-updated', (e) => {
      const { cart: updatedCart, total } = e.detail;
      renderCart(updatedCart, total);
    });

    function renderCart(cartItems, total) {
      const cartList = document.getElementById('cart-items');
      const cartCount = document.getElementById('cart-count');
      const cartTotal = document.getElementById('cart-total');

      cartList.innerHTML = '';
      cartItems.forEach(item => {
        const li = document.createElement('li');
        li.textContent = `${item.name} x${item.qty} = ${item.price * item.qty} บาท`;
        cartList.appendChild(li);
      });

      cartCount.textContent = cartItems.reduce((sum, item) => sum + item.qty, 0);
      cartTotal.textContent = total;
    }

    function showNotification(msg) {
      const notifContainer = document.getElementById('notifications');
      const notif = document.createElement('div');
      notif.textContent = msg;
      notif.style.cssText = 'background:#4caf50;color:white;padding:10px;margin:5px;border-radius:4px;';
      notifContainer.appendChild(notif);
      setTimeout(() => notif.remove(), 3000);
    }

    // Wire up add to cart buttons
    document.querySelectorAll('.add-to-cart').forEach(btn => {
      btn.addEventListener('click', (e) => {
        const productEl = e.target.closest('.product');
        const product = {
          id: Number(productEl.dataset.id),
          name: productEl.dataset.name,
          price: Number(productEl.dataset.price),
        };

        // Dispatch custom event
        const event = createAddToCartEvent(product);
        const proceed = productEl.dispatchEvent(event);

        if (!proceed) {
          console.log('Event was cancelled!');
        }
      });
    });

    // EventTarget.dispatchEvent() returns:
    // true - ถ้า event ไม่ถูก cancel
    // false - ถ้า event ถูก cancel (preventDefault)

    // สร้าง EventEmitter pattern
    class EventEmitter {
      constructor() {
        this._listeners = {};
      }

      on(event, listener) {
        if (!this._listeners[event]) {
          this._listeners[event] = [];
        }
        this._listeners[event].push(listener);
        return this;
      }

      off(event, listener) {
        if (this._listeners[event]) {
          this._listeners[event] = this._listeners[event].filter(l => l !== listener);
        }
        return this;
      }

      emit(event, ...args) {
        if (this._listeners[event]) {
          this._listeners[event].forEach(listener => listener(...args));
        }
        return this;
      }

      once(event, listener) {
        const wrapper = (...args) => {
          listener(...args);
          this.off(event, wrapper);
        };
        return this.on(event, wrapper);
      }
    }

    const emitter = new EventEmitter();
    emitter.on('data', (data) => console.log('Received:', data));
    emitter.emit('data', { message: 'Hello' });
  </script>
</body>
</html>
```

---

## Step 223: Event Delegation

```html
<!DOCTYPE html>
<html lang="th">
<body>
  <div id="menu">
    <button data-action="home">หน้าแรก</button>
    <button data-action="about">เกี่ยวกับ</button>
    <button data-action="contact">ติดต่อ</button>
  </div>

  <table id="data-table">
    <thead>
      <tr>
        <th data-sort="name">ชื่อ ▼</th>
        <th data-sort="age">อายุ</th>
        <th data-sort="score">คะแนน</th>
        <th>Actions</th>
      </tr>
    </thead>
    <tbody id="table-body">
      <tr data-id="1">
        <td>สมชาย</td><td>25</td><td>85</td>
        <td>
          <button class="action-btn" data-action="edit">แก้ไข</button>
          <button class="action-btn" data-action="delete">ลบ</button>
        </td>
      </tr>
      <tr data-id="2">
        <td>สมหญิง</td><td>30</td><td>92</td>
        <td>
          <button class="action-btn" data-action="edit">แก้ไข</button>
          <button class="action-btn" data-action="delete">ลบ</button>
        </td>
      </tr>
    </tbody>
  </table>

  <button id="add-row">เพิ่มแถว</button>
  <div id="output"></div>

  <script>
    const output = document.getElementById('output');
    function log(msg) {
      const p = document.createElement('p');
      p.textContent = msg;
      output.prepend(p);
    }

    // Event Delegation: แทนที่จะ add listener ให้ทุก button
    // ให้ add listener ที่ parent element แทน
    // ดีกว่าเพราะ:
    // 1. ลด memory usage
    // 2. รองรับ dynamically added elements
    // 3. code น้อยกว่า

    // BAD: listener ทุก button
    // document.querySelectorAll('#menu button').forEach(btn => {
    //   btn.addEventListener('click', handleMenu);
    // });

    // GOOD: delegation บน parent
    document.getElementById('menu').addEventListener('click', (e) => {
      const btn = e.target.closest('button[data-action]');
      if (!btn) return; // คลิกที่อื่น

      const action = btn.dataset.action;
      log(`Menu: ${action}`);

      switch (action) {
        case 'home': log('Navigating to home'); break;
        case 'about': log('Navigating to about'); break;
        case 'contact': log('Navigating to contact'); break;
      }
    });

    // Table row actions ด้วย delegation
    const tableBody = document.getElementById('table-body');
    tableBody.addEventListener('click', (e) => {
      const actionBtn = e.target.closest('.action-btn');
      if (!actionBtn) return;

      const row = actionBtn.closest('tr[data-id]');
      const rowId = row?.dataset.id;
      const action = actionBtn.dataset.action;

      if (action === 'delete') {
        if (confirm(`ลบแถว ID ${rowId}?`)) {
          row.remove();
          log(`ลบแถว ${rowId} แล้ว`);
        }
      } else if (action === 'edit') {
        const cells = row.querySelectorAll('td:not(:last-child)');
        if (actionBtn.textContent === 'แก้ไข') {
          cells.forEach(cell => {
            const input = document.createElement('input');
            input.value = cell.textContent;
            input.style.width = '100%';
            cell.textContent = '';
            cell.appendChild(input);
          });
          actionBtn.textContent = 'บันทึก';
        } else {
          cells.forEach(cell => {
            cell.textContent = cell.querySelector('input').value;
          });
          actionBtn.textContent = 'แก้ไข';
          log(`บันทึกแถว ${rowId} แล้ว`);
        }
      }
    });

    // เพิ่มแถวใหม่ - delegation ยังทำงานกับ element ใหม่
    let nextId = 3;
    document.getElementById('add-row').addEventListener('click', () => {
      const tr = document.createElement('tr');
      tr.dataset.id = nextId;
      tr.innerHTML = `
        <td>ใหม่${nextId}</td>
        <td>20</td>
        <td>75</td>
        <td>
          <button class="action-btn" data-action="edit">แก้ไข</button>
          <button class="action-btn" data-action="delete">ลบ</button>
        </td>
      `;
      tableBody.appendChild(tr);
      nextId++;
      log('เพิ่มแถวใหม่แล้ว (delegation ยังทำงานได้!)');
    });

    // Sorting headers ด้วย delegation
    document.querySelector('#data-table thead').addEventListener('click', (e) => {
      const th = e.target.closest('th[data-sort]');
      if (!th) return;

      const sortKey = th.dataset.sort;
      log(`Sorting by: ${sortKey}`);
      // implement sort logic here...
    });
  </script>
</body>
</html>
```

---

## Step 224: Once Option และ Passive Listeners

```html
<!DOCTYPE html>
<html lang="th">
<body>
  <button id="submit-once">Submit (ครั้งเดียว)</button>
  <button id="load-btn">Load Data (ครั้งเดียว)</button>
  <div id="scroll-area" style="height:200px;overflow-y:scroll;border:1px solid #ccc;">
    <div style="height:1000px;padding:20px;">Scroll ใน div นี้</div>
  </div>
  <div id="output"></div>

  <script>
    const output = document.getElementById('output');
    function log(msg) {
      const p = document.createElement('p');
      p.textContent = `${new Date().toLocaleTimeString()}: ${msg}`;
      output.prepend(p);
    }

    // once: true - listener ถูกลบอัตโนมัติหลังเรียกครั้งแรก
    const submitBtn = document.getElementById('submit-once');
    submitBtn.addEventListener('click', () => {
      log('Submitted! (คลิกครั้งต่อไปจะไม่มีผล)');
    }, { once: true });

    // ใช้กับ animation events
    const box = document.createElement('div');
    box.style.cssText = 'width:100px;height:100px;background:blue;transition:transform 0.5s;';
    document.body.appendChild(box);

    box.addEventListener('transitionend', () => {
      log('Transition ended (once)');
    }, { once: true });

    box.addEventListener('click', () => {
      box.style.transform = 'translateX(200px)';
    });

    // passive: true - listener ไม่เรียก preventDefault
    // ใช้กับ scroll/touch events เพื่อ performance ที่ดีขึ้น
    const scrollArea = document.getElementById('scroll-area');

    // Passive listener - browser ไม่ต้องรอ listener ก่อน scroll
    scrollArea.addEventListener('scroll', (e) => {
      // e.preventDefault(); // จะ error! passive listener ห้าม preventDefault
      log(`Scrolled: ${scrollArea.scrollTop}px`);
    }, { passive: true });

    // Touch passive (default บน modern browsers)
    scrollArea.addEventListener('touchmove', (e) => {
      // e.preventDefault(); // จะ error!
    }, { passive: true });

    // Non-passive เมื่อต้องการ preventDefault
    scrollArea.addEventListener('touchstart', (e) => {
      e.preventDefault(); // OK เพราะไม่ passive
    }, { passive: false });

    // load-btn: load data เพียงครั้งเดียว
    const loadBtn = document.getElementById('load-btn');
    async function loadData() {
      log('Loading data...');
      // simulate async load
      await new Promise(resolve => setTimeout(resolve, 1000));
      log('Data loaded!');
    }

    loadBtn.addEventListener('click', loadData, { once: true });
    loadBtn.addEventListener('click', () => {
      loadBtn.disabled = true;
      loadBtn.textContent = 'Data Loaded';
    }, { once: true });

    // สร้าง Promises ที่รอ event ด้วย once
    function waitForEvent(element, eventType) {
      return new Promise(resolve => {
        element.addEventListener(eventType, resolve, { once: true });
      });
    }

    // ใช้งาน:
    async function example() {
      log('Waiting for click...');
      const event = await waitForEvent(document.body, 'click');
      log(`Clicked at (${event.clientX}, ${event.clientY})`);
    }
    // example();
  </script>
</body>
</html>
```

---

## Step 225: ตัวอย่างประยุกต์ - Interactive Quiz

```html
<!DOCTYPE html>
<html lang="th">
<head>
  <style>
    .quiz-container { max-width: 600px; margin: 0 auto; }
    .question { margin: 20px 0; padding: 15px; background: #f5f5f5; border-radius: 8px; }
    .options { list-style: none; padding: 0; }
    .option { padding: 10px; margin: 5px 0; background: white; border: 2px solid #ddd; border-radius: 4px; cursor: pointer; transition: all 0.2s; }
    .option:hover { border-color: blue; background: #e8f0fe; }
    .option.correct { border-color: green; background: #e8f5e9; }
    .option.wrong { border-color: red; background: #ffebee; }
    .option.disabled { pointer-events: none; }
    #result { padding: 20px; text-align: center; border-radius: 8px; }
    #timer { font-size: 24px; font-weight: bold; color: #333; }
    .progress-bar { height: 8px; background: #ddd; border-radius: 4px; margin: 10px 0; }
    .progress-fill { height: 100%; background: blue; border-radius: 4px; transition: width 0.3s; }
  </style>
</head>
<body>
  <div class="quiz-container">
    <h1>Quiz: JavaScript Events</h1>
    <div id="timer">60</div>
    <div class="progress-bar"><div class="progress-fill" id="progress" style="width:0%"></div></div>
    <div id="quiz-area"></div>
    <div id="result" style="display:none;"></div>
  </div>

  <script>
    const questions = [
      {
        question: 'addEventListener ใช้สำหรับอะไร?',
        options: ['เพิ่ม event listener', 'ลบ element', 'สร้าง element', 'อ่าน attribute'],
        answer: 0
      },
      {
        question: 'event.preventDefault() ทำอะไร?',
        options: ['หยุด bubbling', 'ป้องกัน default behavior', 'ลบ event listener', 'สร้าง event'],
        answer: 1
      },
      {
        question: 'Event ใดที่เกิดขึ้นเมื่อ DOM พร้อมใช้งาน?',
        options: ['load', 'ready', 'DOMContentLoaded', 'domready'],
        answer: 2
      },
      {
        question: 'stopPropagation ทำอะไร?',
        options: ['ป้องกัน default', 'หยุด bubbling', 'ลบ listener', 'สร้าง event'],
        answer: 1
      },
      {
        question: 'Event Delegation คืออะไร?',
        options: [
          'การ add listener หลายอัน',
          'การ add listener ที่ parent แทน children',
          'การสร้าง custom event',
          'การลบ event listener'
        ],
        answer: 1
      },
    ];

    let currentQ = 0;
    let score = 0;
    let timeLeft = 60;
    let timer;

    const quizArea = document.getElementById('quiz-area');
    const result = document.getElementById('result');
    const timerEl = document.getElementById('timer');
    const progressEl = document.getElementById('progress');

    function showQuestion() {
      if (currentQ >= questions.length) {
        endQuiz();
        return;
      }

      const q = questions[currentQ];
      quizArea.innerHTML = `
        <div class="question">
          <p><strong>Q${currentQ + 1}/${questions.length}:</strong> ${q.question}</p>
          <ul class="options" id="options"></ul>
        </div>
      `;

      const optionsList = document.getElementById('options');
      q.options.forEach((opt, i) => {
        const li = document.createElement('li');
        li.className = 'option';
        li.textContent = opt;
        li.dataset.index = i;
        optionsList.appendChild(li);
      });

      progressEl.style.width = `${(currentQ / questions.length) * 100}%`;
    }

    // Event Delegation สำหรับ options
    quizArea.addEventListener('click', (e) => {
      const option = e.target.closest('.option');
      if (!option) return;

      const selectedIndex = Number(option.dataset.index);
      const correctIndex = questions[currentQ].answer;

      // Disable all options
      document.querySelectorAll('.option').forEach(opt => {
        opt.classList.add('disabled');
      });

      if (selectedIndex === correctIndex) {
        option.classList.add('correct');
        score++;
      } else {
        option.classList.add('wrong');
        document.querySelectorAll('.option')[correctIndex].classList.add('correct');
      }

      currentQ++;
      setTimeout(showQuestion, 1000);
    });

    function endQuiz() {
      clearInterval(timer);
      quizArea.style.display = 'none';
      result.style.display = 'block';
      result.style.background = score >= 3 ? '#e8f5e9' : '#ffebee';
      result.innerHTML = `
        <h2>${score >= 3 ? '🎉 ผ่าน!' : '😢 ไม่ผ่าน'}</h2>
        <p>คะแนน: ${score}/${questions.length}</p>
        <button id="restart">เล่นใหม่</button>
      `;
      document.getElementById('restart').addEventListener('click', restart);
    }

    function restart() {
      currentQ = 0;
      score = 0;
      timeLeft = 60;
      result.style.display = 'none';
      quizArea.style.display = 'block';
      startTimer();
      showQuestion();
    }

    function startTimer() {
      clearInterval(timer);
      timer = setInterval(() => {
        timeLeft--;
        timerEl.textContent = timeLeft;
        timerEl.style.color = timeLeft <= 10 ? 'red' : '#333';
        if (timeLeft <= 0) endQuiz();
      }, 1000);
    }

    startTimer();
    showQuestion();
  </script>
</body>
</html>
```

---

## Steps 226-230: เพิ่มเติม Event Patterns

### Step 226: Debounce และ Throttle

```javascript
// Debounce: รอ delay ms หลังจาก event สุดท้าย แล้วค่อยเรียก fn
function debounce(fn, delay) {
  let timer;
  return function (...args) {
    clearTimeout(timer);
    timer = setTimeout(() => fn.apply(this, args), delay);
  };
}

// Throttle: เรียก fn ได้แค่ครั้งเดียวต่อ limit ms
function throttle(fn, limit) {
  let lastCall = 0;
  return function (...args) {
    const now = Date.now();
    if (now - lastCall >= limit) {
      lastCall = now;
      return fn.apply(this, args);
    }
  };
}

// การใช้งาน
const searchInput = document.querySelector('#search');
const debouncedSearch = debounce((e) => {
  console.log('Searching for:', e.target.value);
  // fetch('/api/search?q=' + e.target.value)
}, 300);
searchInput?.addEventListener('input', debouncedSearch);

const throttledScroll = throttle(() => {
  console.log('Scroll position:', window.scrollY);
}, 100);
window.addEventListener('scroll', throttledScroll);

// Resize with debounce
const debouncedResize = debounce(() => {
  console.log('Resized to:', window.innerWidth, 'x', window.innerHeight);
}, 200);
window.addEventListener('resize', debouncedResize);
```

### Step 227: Event Emitter Class

```javascript
class TypedEventEmitter {
  #listeners = new Map();

  on(event, listener, options = {}) {
    if (!this.#listeners.has(event)) {
      this.#listeners.set(event, new Set());
    }
    const entry = { listener, once: options.once || false };
    this.#listeners.get(event).add(entry);
    return () => this.off(event, listener);
  }

  once(event, listener) {
    return this.on(event, listener, { once: true });
  }

  off(event, listener) {
    const listeners = this.#listeners.get(event);
    if (!listeners) return;
    for (const entry of listeners) {
      if (entry.listener === listener) {
        listeners.delete(entry);
        break;
      }
    }
  }

  emit(event, data) {
    const listeners = this.#listeners.get(event);
    if (!listeners) return;
    for (const entry of [...listeners]) {
      entry.listener(data);
      if (entry.once) listeners.delete(entry);
    }
  }

  listenerCount(event) {
    return this.#listeners.get(event)?.size || 0;
  }
}

const bus = new TypedEventEmitter();
const unsubscribe = bus.on('data', (data) => console.log('Got:', data));
bus.once('data', (data) => console.log('Once:', data));

bus.emit('data', { value: 1 }); // Got: ..., Once: ...
bus.emit('data', { value: 2 }); // Got: ... (once is gone)
unsubscribe(); // ลบ listener
bus.emit('data', { value: 3 }); // ไม่มีผล
```

### Step 228: Intersection Observer

```javascript
// ตรวจสอบว่า element อยู่ใน viewport หรือไม่
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      entry.target.classList.add('visible');
      entry.target.style.opacity = '1';
      entry.target.style.transform = 'translateY(0)';
    }
  });
}, {
  threshold: 0.1, // 10% ของ element ต้องเห็น
  rootMargin: '0px 0px -50px 0px',
});

// Lazy loading images
document.querySelectorAll('[data-src]').forEach(img => {
  observer.observe(img);
});

const imgObserver = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      const img = entry.target;
      img.src = img.dataset.src;
      imgObserver.unobserve(img);
    }
  });
});

document.querySelectorAll('img[data-src]').forEach(img => {
  imgObserver.observe(img);
});
```

### Step 229: MutationObserver

```javascript
// ตรวจสอบการเปลี่ยนแปลงใน DOM
const targetNode = document.getElementById('dynamic-content');

const mutationObserver = new MutationObserver((mutations) => {
  mutations.forEach(mutation => {
    console.log('Mutation type:', mutation.type);

    if (mutation.type === 'childList') {
      mutation.addedNodes.forEach(node => {
        console.log('Added:', node);
      });
      mutation.removedNodes.forEach(node => {
        console.log('Removed:', node);
      });
    }

    if (mutation.type === 'attributes') {
      console.log(`Attribute "${mutation.attributeName}" changed`);
      console.log('Old value:', mutation.oldValue);
    }

    if (mutation.type === 'characterData') {
      console.log('Text changed to:', mutation.target.data);
    }
  });
});

mutationObserver.observe(targetNode, {
  childList: true,     // ตรวจ children เพิ่ม/ลบ
  subtree: true,       // ตรวจ descendants ด้วย
  attributes: true,    // ตรวจ attributes
  attributeOldValue: true,
  characterData: true, // ตรวจ text content
  characterDataOldValue: true,
});

// หยุด observe
// mutationObserver.disconnect();
```

### Step 230: ResizeObserver

```javascript
// ตรวจสอบขนาดของ element เปลี่ยนแปลง
const resizeObserver = new ResizeObserver((entries) => {
  entries.forEach(entry => {
    const { width, height } = entry.contentRect;
    console.log(`Element ${entry.target.id}: ${width}x${height}`);

    // Responsive component ตาม size
    if (width < 400) {
      entry.target.classList.add('small');
      entry.target.classList.remove('large');
    } else {
      entry.target.classList.add('large');
      entry.target.classList.remove('small');
    }
  });
});

const elements = document.querySelectorAll('.resizable');
elements.forEach(el => resizeObserver.observe(el));
```

---

## สรุป Steps 211-230

| Step | หัวข้อ |
|------|--------|
| 211 | What are Events |
| 212 | addEventListener / removeEventListener |
| 213 | Event Object properties |
| 214 | Event Propagation (bubbling/capturing) |
| 215 | stopPropagation / stopImmediatePropagation |
| 216 | preventDefault |
| 217 | Mouse Events |
| 218 | Keyboard Events |
| 219 | Form Events |
| 220 | Window Events |
| 221 | Touch Events |
| 222 | Custom Events |
| 223 | Event Delegation |
| 224 | Once / Passive options |
| 225 | Quiz example |
| 226 | Debounce / Throttle |
| 227 | EventEmitter class |
| 228 | IntersectionObserver |
| 229 | MutationObserver |
| 230 | ResizeObserver |

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: Keyboard Shortcut Manager
สร้าง shortcut manager ที่:
- รองรับ combos เช่น Ctrl+S, Alt+F4
- แสดง popup เมื่อกด shortcut
- มี help overlay (กด ?) แสดง shortcuts ทั้งหมด

### แบบฝึกหัดที่ 2: Drag and Drop
สร้าง kanban board เบื้องต้น ที่:
- ลาก card ระหว่าง column ได้
- ใช้ dragstart, dragover, drop events
- แสดง visual feedback

### แบบฝึกหัดที่ 3: Infinite Scroll
สร้าง infinite scroll ที่:
- โหลด content ใหม่เมื่อ scroll ใกล้ถึง bottom
- ใช้ IntersectionObserver
- แสดง loading indicator

### แบบฝึกหัดที่ 4: Real-time Search
สร้าง search feature ที่:
- ใช้ debounce 300ms
- highlight ผลลัพธ์ที่ตรง
- แสดงจำนวนผลลัพธ์

### แบบฝึกหัดที่ 5: Event Bus
สร้าง Event Bus สำหรับ communication ระหว่าง components:
- publish/subscribe pattern
- wildcard events
- error handling
