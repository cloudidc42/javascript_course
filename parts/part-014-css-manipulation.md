# Part 14: CSS Manipulation ด้วย JavaScript (Steps 251-270)

## บทนำ

JavaScript สามารถควบคุม CSS ได้โดยตรง ตั้งแต่การเปลี่ยน inline styles ไปจนถึงการสร้าง animations ที่ซับซ้อน ทำให้เว็บ Interactive และมีชีวิตชีวา

---

## Step 251: Inline Styles กับ element.style

```html
<!DOCTYPE html>
<html lang="th">
<body>
  <div id="box" style="width:100px; height:100px; background:red;">Box</div>

  <script>
    const box = document.getElementById('box');

    // อ่าน inline style
    console.log(box.style.width);           // "100px"
    console.log(box.style.backgroundColor); // "" (ใช้ background ไม่ใช่ backgroundColor)
    console.log(box.style.background);      // "red"

    // camelCase mapping
    // CSS property    -> JavaScript property
    // background-color -> backgroundColor
    // font-size        -> fontSize
    // border-radius    -> borderRadius
    // z-index          -> zIndex
    // margin-top       -> marginTop
    // padding-left     -> paddingLeft
    // flex-direction   -> flexDirection
    // pointer-events   -> pointerEvents
    // text-transform   -> textTransform
    // list-style-type  -> listStyleType

    // ตั้งค่า style
    box.style.width = '200px';
    box.style.height = '200px';
    box.style.backgroundColor = 'blue';
    box.style.color = 'white';
    box.style.fontSize = '18px';
    box.style.fontWeight = 'bold';
    box.style.borderRadius = '8px';
    box.style.boxShadow = '0 4px 8px rgba(0,0,0,0.3)';
    box.style.display = 'flex';
    box.style.alignItems = 'center';
    box.style.justifyContent = 'center';
    box.style.cursor = 'pointer';
    box.style.transition = 'all 0.3s ease';

    // ลบ style
    box.style.backgroundColor = ''; // คืนค่าให้ stylesheet ควบคุม
    box.style.removeProperty('border-radius'); // อีกวิธี

    // cssText - อ่าน/เขียน style ทั้งหมด
    console.log(box.style.cssText);
    // "width: 200px; height: 200px; color: white; ..."

    // เขียนทับ
    box.style.cssText = 'width:150px; height:150px; background:green;';

    // เพิ่มเติม (ไม่ลบของเก่า)
    box.style.cssText += '; color:white; font-size:16px;';

    // setProperty และ getPropertyValue
    box.style.setProperty('background-color', 'purple');
    box.style.setProperty('margin-top', '20px');
    console.log(box.style.getPropertyValue('background-color')); // "purple"
    console.log(box.style.getPropertyValue('margin-top')); // "20px"

    // Priority
    box.style.setProperty('background', 'orange', 'important');
    console.log(box.style.getPropertyPriority('background')); // "important"

    // ตัวอย่าง: dynamic button states
    function setButtonLoading(btn, loading) {
      if (loading) {
        btn.style.opacity = '0.7';
        btn.style.cursor = 'not-allowed';
        btn.style.pointerEvents = 'none';
      } else {
        btn.style.opacity = '';
        btn.style.cursor = '';
        btn.style.pointerEvents = '';
      }
    }
  </script>
</body>
</html>
```

---

## Step 252: getComputedStyle

```html
<!DOCTYPE html>
<html lang="th">
<head>
  <style>
    #demo {
      font-size: 16px;
      color: #333;
      padding: 20px;
      margin: 10px;
      background: linear-gradient(to right, #e3f2fd, #bbdefb);
    }
    #demo p {
      font-size: 1.2em; /* relative */
      line-height: 1.5;
    }
    @media (max-width: 600px) {
      #demo { font-size: 14px; }
    }
  </style>
</head>
<body>
  <div id="demo">
    <p id="para">Text here</p>
  </div>

  <script>
    const demo = document.getElementById('demo');
    const para = document.getElementById('para');

    // getComputedStyle - ได้ค่า CSS จริงที่ computed แล้ว
    // รวม stylesheet + inherited + browser defaults
    const styles = window.getComputedStyle(demo);

    // ได้ค่าที่ computed (ไม่ใช่ relative values)
    console.log(styles.fontSize);        // "16px"
    console.log(styles.color);           // "rgb(51, 51, 51)"
    console.log(styles.backgroundColor); // computed value
    console.log(styles.padding);         // "20px"
    console.log(styles.paddingTop);      // "20px"
    console.log(styles.margin);          // "10px"
    console.log(styles.width);           // computed width in px
    console.log(styles.display);         // "block"
    console.log(styles.position);        // "static"
    console.log(styles.boxSizing);       // "content-box"

    // Paragraph: font-size 1.2em relative ถึง parent (16px)
    const paraStyles = window.getComputedStyle(para);
    console.log(paraStyles.fontSize);    // "19.2px" (16 * 1.2)
    console.log(paraStyles.lineHeight);  // "28.8px" (19.2 * 1.5)

    // getPropertyValue
    console.log(styles.getPropertyValue('font-size'));   // "16px"
    console.log(styles.getPropertyValue('background'));  // computed bg

    // Read-only
    // styles.fontSize = '20px'; // Error: read-only!

    // Pseudo-elements
    const beforeStyle = window.getComputedStyle(demo, '::before');
    console.log(beforeStyle.content); // หากมี ::before

    // ตรวจสอบ visibility
    function isVisible(element) {
      const style = window.getComputedStyle(element);
      return style.display !== 'none' &&
             style.visibility !== 'hidden' &&
             style.opacity !== '0';
    }

    // ดึงค่าตัวเลข
    function getNumericStyle(element, property) {
      return parseFloat(window.getComputedStyle(element)[property]);
    }

    console.log(getNumericStyle(demo, 'fontSize'));  // 16
    console.log(getNumericStyle(demo, 'paddingTop')); // 20

    // compare inline vs computed
    demo.style.fontSize = '20px';
    console.log(demo.style.fontSize);                           // "20px" (inline)
    console.log(window.getComputedStyle(demo).fontSize);        // "20px" (computed)

    demo.style.fontSize = '';
    console.log(demo.style.fontSize);                           // "" (inline empty)
    console.log(window.getComputedStyle(demo).fontSize);        // "16px" (from stylesheet)
  </script>
</body>
</html>
```

---

## Step 253: CSS Custom Properties (Variables) ใน JavaScript

```html
<!DOCTYPE html>
<html lang="th">
<head>
  <style>
    :root {
      --primary-color: #4285f4;
      --secondary-color: #ea4335;
      --text-color: #333;
      --bg-color: #fff;
      --font-size-base: 16px;
      --border-radius: 8px;
      --spacing: 16px;
      --transition: 0.3s ease;
    }

    .themed-card {
      background: var(--bg-color);
      color: var(--text-color);
      border-radius: var(--border-radius);
      padding: var(--spacing);
      border: 2px solid var(--primary-color);
      transition: all var(--transition);
    }

    .btn-primary {
      background: var(--primary-color);
      color: white;
      padding: 10px 20px;
      border: none;
      border-radius: var(--border-radius);
      cursor: pointer;
    }
  </style>
</head>
<body>
  <div class="themed-card" id="card">
    <h2>Themed Card</h2>
    <p>เนื้อหา...</p>
    <button class="btn-primary">Button</button>
  </div>

  <div id="controls">
    <label>Primary Color: <input type="color" id="color-picker" value="#4285f4"></label>
    <label>Font Size: <input type="range" id="font-size" min="12" max="24" value="16"></label>
    <label>Border Radius: <input type="range" id="border-radius" min="0" max="20" value="8"></label>
    <button id="dark-mode">Dark Mode</button>
  </div>

  <script>
    const root = document.documentElement;

    // อ่าน CSS variable
    const primaryColor = getComputedStyle(root).getPropertyValue('--primary-color').trim();
    console.log('Primary color:', primaryColor); // "#4285f4"

    // ตั้งค่า CSS variable
    root.style.setProperty('--primary-color', '#e91e63');

    // อ่าน CSS variable จาก element
    const card = document.getElementById('card');
    const cardBg = getComputedStyle(card).getPropertyValue('--bg-color').trim();
    console.log('Card bg:', cardBg); // "#fff"

    // ตั้งค่า variable บน element เฉพาะ
    card.style.setProperty('--spacing', '32px');

    // Real-time theme customization
    document.getElementById('color-picker').addEventListener('input', (e) => {
      root.style.setProperty('--primary-color', e.target.value);
    });

    document.getElementById('font-size').addEventListener('input', (e) => {
      root.style.setProperty('--font-size-base', e.target.value + 'px');
      document.body.style.fontSize = e.target.value + 'px';
    });

    document.getElementById('border-radius').addEventListener('input', (e) => {
      root.style.setProperty('--border-radius', e.target.value + 'px');
    });

    // Dark mode toggle
    let isDark = false;
    document.getElementById('dark-mode').addEventListener('click', () => {
      isDark = !isDark;
      if (isDark) {
        root.style.setProperty('--bg-color', '#1a1a1a');
        root.style.setProperty('--text-color', '#f0f0f0');
        root.style.setProperty('--primary-color', '#82b1ff');
      } else {
        root.style.removeProperty('--bg-color');
        root.style.removeProperty('--text-color');
        root.style.removeProperty('--primary-color');
      }
    });

    // ลบ CSS variable
    root.style.removeProperty('--primary-color'); // คืนค่าเดิม

    // Dynamic theme system
    const themes = {
      blue: {
        '--primary-color': '#1565c0',
        '--secondary-color': '#e91e63',
        '--bg-color': '#e3f2fd',
      },
      green: {
        '--primary-color': '#2e7d32',
        '--secondary-color': '#f57f17',
        '--bg-color': '#e8f5e9',
      },
      purple: {
        '--primary-color': '#6a1b9a',
        '--secondary-color': '#00897b',
        '--bg-color': '#f3e5f5',
      },
    };

    function applyTheme(themeName) {
      const theme = themes[themeName];
      if (!theme) return;
      Object.entries(theme).forEach(([property, value]) => {
        root.style.setProperty(property, value);
      });
      localStorage.setItem('theme', themeName);
    }

    // โหลด theme จาก localStorage
    const savedTheme = localStorage.getItem('theme');
    if (savedTheme) applyTheme(savedTheme);
  </script>
</body>
</html>
```

---

## Step 254: classList Methods

```html
<!DOCTYPE html>
<html lang="th">
<head>
  <style>
    .card { padding: 20px; margin: 10px; border: 1px solid #ddd; border-radius: 8px; }
    .card.active { border-color: blue; background: #e8f0fe; }
    .card.featured { border-color: gold; background: #fffde7; }
    .card.hidden { display: none; }
    .card.large { font-size: 1.2em; padding: 30px; }
    .card.animated { transition: all 0.3s ease; }
    .card.scale-up { transform: scale(1.05); }
    .card.fade-out { opacity: 0; }

    .tab { display: none; }
    .tab.active { display: block; }
    .tab-btn { padding: 8px 16px; cursor: pointer; border: 1px solid #ccc; background: #f5f5f5; }
    .tab-btn.active { background: white; border-bottom: none; }
    .accordion-content { display: none; padding: 10px; }
    .accordion-content.open { display: block; }
  </style>
</head>
<body>
  <div id="card1" class="card animated">Card 1</div>
  <div id="card2" class="card animated featured">Card 2 (featured)</div>

  <div id="tabs">
    <button class="tab-btn active" data-tab="tab1">Tab 1</button>
    <button class="tab-btn" data-tab="tab2">Tab 2</button>
    <button class="tab-btn" data-tab="tab3">Tab 3</button>
  </div>
  <div id="tab1" class="tab active">Content Tab 1</div>
  <div id="tab2" class="tab">Content Tab 2</div>
  <div id="tab3" class="tab">Content Tab 3</div>

  <div id="accordion">
    <div class="accordion-item">
      <button class="accordion-header">ส่วนที่ 1 ▼</button>
      <div class="accordion-content">เนื้อหาส่วนที่ 1...</div>
    </div>
    <div class="accordion-item">
      <button class="accordion-header">ส่วนที่ 2 ▼</button>
      <div class="accordion-content">เนื้อหาส่วนที่ 2...</div>
    </div>
    <div class="accordion-item">
      <button class="accordion-header">ส่วนที่ 3 ▼</button>
      <div class="accordion-content">เนื้อหาส่วนที่ 3...</div>
    </div>
  </div>

  <script>
    const card1 = document.getElementById('card1');
    const card2 = document.getElementById('card2');

    // add - เพิ่ม class
    card1.classList.add('active');
    card1.classList.add('large', 'highlighted'); // หลาย class
    console.log(card1.className); // "card animated active large highlighted"

    // remove - ลบ class
    card1.classList.remove('highlighted');
    card1.classList.remove('large', 'active'); // หลาย class
    console.log(card1.classList.length); // 2

    // toggle - เพิ่ม/ลบสลับกัน
    card1.addEventListener('click', () => {
      card1.classList.toggle('active');
      card1.classList.toggle('scale-up');
      setTimeout(() => card1.classList.remove('scale-up'), 200);
    });

    // contains - ตรวจสอบ
    console.log(card2.classList.contains('featured')); // true
    console.log(card2.classList.contains('active'));   // false

    // replace - แทนที่
    card2.classList.replace('featured', 'active');
    console.log(card2.classList.contains('featured')); // false
    console.log(card2.classList.contains('active'));   // true

    // Tabs component
    const tabButtons = document.querySelectorAll('.tab-btn');
    const tabs = document.querySelectorAll('.tab');

    tabButtons.forEach(btn => {
      btn.addEventListener('click', () => {
        // Remove active from all
        tabButtons.forEach(b => b.classList.remove('active'));
        tabs.forEach(t => t.classList.remove('active'));

        // Add active to clicked
        btn.classList.add('active');
        const tabId = btn.dataset.tab;
        document.getElementById(tabId).classList.add('active');
      });
    });

    // Accordion
    document.querySelectorAll('.accordion-header').forEach(header => {
      header.addEventListener('click', () => {
        const content = header.nextElementSibling;
        const isOpen = content.classList.contains('open');

        // Close all
        document.querySelectorAll('.accordion-content').forEach(c => {
          c.classList.remove('open');
        });

        // Open clicked (ถ้าไม่ได้เปิดอยู่แล้ว)
        if (!isOpen) {
          content.classList.add('open');
          header.textContent = header.textContent.replace('▼', '▲');
        } else {
          header.textContent = header.textContent.replace('▲', '▼');
        }
      });
    });

    // Toggle multiple class states
    const states = ['default', 'loading', 'success', 'error'];
    let currentState = 0;

    function cycleState(element) {
      states.forEach(s => element.classList.remove(s));
      element.classList.add(states[currentState]);
      currentState = (currentState + 1) % states.length;
    }
  </script>
</body>
</html>
```

---

## Step 255: CSS Transitions ที่ Trigger ด้วย JS

```html
<!DOCTYPE html>
<html lang="th">
<head>
  <style>
    .box {
      width: 100px; height: 100px;
      background: #4285f4;
      border-radius: 8px;
      transition: all 0.4s ease;
    }
    .box.expanded { width: 300px; height: 150px; }
    .box.moved { transform: translateX(200px); }
    .box.rotated { transform: rotate(45deg); }
    .box.colored { background: #e91e63; }
    .box.faded { opacity: 0; }

    .slide-panel {
      width: 250px;
      background: #333;
      color: white;
      padding: 20px;
      transform: translateX(-100%);
      transition: transform 0.3s ease;
      position: fixed; left: 0; top: 0;
      height: 100%;
    }
    .slide-panel.open { transform: translateX(0); }

    .dropdown-menu {
      max-height: 0;
      overflow: hidden;
      transition: max-height 0.3s ease, opacity 0.3s ease;
      opacity: 0;
      background: white;
      border: 1px solid #ddd;
    }
    .dropdown-menu.open {
      max-height: 200px;
      opacity: 1;
    }

    .fade-element {
      opacity: 0;
      transform: translateY(20px);
      transition: opacity 0.5s ease, transform 0.5s ease;
    }
    .fade-element.visible {
      opacity: 1;
      transform: translateY(0);
    }
  </style>
</head>
<body>
  <div class="box" id="box">Hover/Click</div>
  <br>

  <button id="expand">Expand</button>
  <button id="move">Move</button>
  <button id="rotate">Rotate</button>
  <button id="fade">Fade</button>
  <button id="reset">Reset</button>

  <br><br>
  <button id="toggle-panel">Toggle Sidebar</button>
  <div class="slide-panel" id="panel">
    <p>Sidebar Content</p>
    <button id="close-panel">Close</button>
  </div>

  <br>
  <button id="toggle-dropdown">Dropdown</button>
  <div class="dropdown-menu" id="dropdown">
    <div style="padding:10px">Item 1</div>
    <div style="padding:10px">Item 2</div>
    <div style="padding:10px">Item 3</div>
  </div>

  <script>
    const box = document.getElementById('box');

    document.getElementById('expand').addEventListener('click', () => {
      box.classList.toggle('expanded');
    });

    document.getElementById('move').addEventListener('click', () => {
      box.classList.toggle('moved');
    });

    document.getElementById('rotate').addEventListener('click', () => {
      box.classList.toggle('rotated');
    });

    document.getElementById('fade').addEventListener('click', () => {
      box.classList.toggle('faded');
    });

    document.getElementById('reset').addEventListener('click', () => {
      box.className = 'box';
    });

    // Hover effects
    box.addEventListener('mouseenter', () => {
      box.classList.add('colored');
    });
    box.addEventListener('mouseleave', () => {
      box.classList.remove('colored');
    });

    // transitionend event
    box.addEventListener('transitionend', (e) => {
      console.log(`Transition ended: ${e.propertyName}`);
    });

    // Sidebar
    const panel = document.getElementById('panel');
    document.getElementById('toggle-panel').addEventListener('click', () => {
      panel.classList.toggle('open');
    });
    document.getElementById('close-panel').addEventListener('click', () => {
      panel.classList.remove('open');
    });

    // Dropdown
    const dropdown = document.getElementById('dropdown');
    document.getElementById('toggle-dropdown').addEventListener('click', () => {
      dropdown.classList.toggle('open');
    });

    document.addEventListener('click', (e) => {
      if (!e.target.closest('#toggle-dropdown') && !e.target.closest('#dropdown')) {
        dropdown.classList.remove('open');
      }
    });

    // Fade in elements on scroll
    const fadeElements = document.querySelectorAll('.fade-element');
    const observer = new IntersectionObserver((entries) => {
      entries.forEach((entry, i) => {
        if (entry.isIntersecting) {
          setTimeout(() => {
            entry.target.classList.add('visible');
          }, i * 100); // stagger
          observer.unobserve(entry.target);
        }
      });
    }, { threshold: 0.1 });

    fadeElements.forEach(el => observer.observe(el));

    // Promise-based transition
    function transitionEnd(element) {
      return new Promise(resolve => {
        element.addEventListener('transitionend', resolve, { once: true });
      });
    }

    async function animateOut(element) {
      element.classList.add('faded');
      await transitionEnd(element);
      element.remove();
    }
  </script>
</body>
</html>
```

---

## Step 256: requestAnimationFrame

```html
<!DOCTYPE html>
<html lang="th">
<head>
  <style>
    canvas { border: 1px solid #ccc; display: block; }
    #ball {
      width: 50px; height: 50px;
      background: radial-gradient(circle at 30% 30%, #ff6b6b, #c0392b);
      border-radius: 50%;
      position: absolute;
    }
    #counter { font-size: 48px; font-weight: bold; text-align: center; }
  </style>
</head>
<body>
  <div id="ball"></div>

  <canvas id="canvas" width="600" height="200"></canvas>
  <button id="start">Start</button>
  <button id="stop">Stop</button>
  <button id="fps-start">Show FPS</button>

  <div id="counter">0</div>
  <button id="count-start">Count to 100</button>

  <script>
    // requestAnimationFrame (rAF) คือการขอให้ browser เรียก function
    // ก่อน repaint ครั้งถัดไป (~60fps = ทุก 16.67ms)

    // รูปแบบพื้นฐาน
    let animId;

    function myAnimation(timestamp) {
      // timestamp คือ DOMHighResTimeStamp (ms จากเริ่ม page)
      // ทำ animation...
      animId = requestAnimationFrame(myAnimation);
    }

    // เริ่ม animation
    animId = requestAnimationFrame(myAnimation);

    // หยุด animation
    cancelAnimationFrame(animId);

    // ตัวอย่าง: bouncing ball
    const ball = document.getElementById('ball');
    let bx = 0, by = 100;
    let vx = 3, vy = 2;
    const containerW = window.innerWidth - 50;
    const containerH = window.innerHeight - 50;

    function animateBall() {
      bx += vx;
      by += vy;

      if (bx <= 0 || bx >= containerW) vx *= -1;
      if (by <= 0 || by >= containerH) vy *= -1;

      ball.style.left = bx + 'px';
      ball.style.top = by + 'px';

      requestAnimationFrame(animateBall);
    }
    // requestAnimationFrame(animateBall);

    // Canvas animation
    const canvas = document.getElementById('canvas');
    const ctx = canvas.getContext('2d');
    let running = false;
    let animHandle;
    let particles = [];

    class Particle {
      constructor() {
        this.reset();
      }
      reset() {
        this.x = Math.random() * canvas.width;
        this.y = canvas.height + 10;
        this.vx = (Math.random() - 0.5) * 2;
        this.vy = -(Math.random() * 2 + 1);
        this.life = 1;
        this.decay = Math.random() * 0.02 + 0.005;
        this.size = Math.random() * 8 + 2;
        this.color = `hsl(${Math.random() * 360}, 70%, 60%)`;
      }
      update() {
        this.x += this.vx;
        this.y += this.vy;
        this.life -= this.decay;
        if (this.life <= 0) this.reset();
      }
      draw() {
        ctx.globalAlpha = this.life;
        ctx.fillStyle = this.color;
        ctx.beginPath();
        ctx.arc(this.x, this.y, this.size, 0, Math.PI * 2);
        ctx.fill();
      }
    }

    for (let i = 0; i < 50; i++) particles.push(new Particle());

    function drawFrame() {
      ctx.clearRect(0, 0, canvas.width, canvas.height);
      ctx.globalAlpha = 1;
      particles.forEach(p => { p.update(); p.draw(); });
      if (running) animHandle = requestAnimationFrame(drawFrame);
    }

    document.getElementById('start').addEventListener('click', () => {
      if (!running) {
        running = true;
        drawFrame();
      }
    });

    document.getElementById('stop').addEventListener('click', () => {
      running = false;
      cancelAnimationFrame(animHandle);
    });

    // FPS Counter
    let frameCount = 0;
    let lastTime = performance.now();
    let fpsRunning = false;

    function countFPS(timestamp) {
      frameCount++;
      if (timestamp - lastTime >= 1000) {
        console.log(`FPS: ${frameCount}`);
        document.title = `FPS: ${frameCount}`;
        frameCount = 0;
        lastTime = timestamp;
      }
      if (fpsRunning) requestAnimationFrame(countFPS);
    }

    document.getElementById('fps-start').addEventListener('click', () => {
      fpsRunning = !fpsRunning;
      if (fpsRunning) requestAnimationFrame(countFPS);
    });

    // Smooth counter animation
    const counterEl = document.getElementById('counter');
    function animateCounter(start, end, duration) {
      const startTime = performance.now();
      function update(timestamp) {
        const elapsed = timestamp - startTime;
        const progress = Math.min(elapsed / duration, 1);
        // easing function
        const eased = 1 - Math.pow(1 - progress, 3); // ease out cubic
        counterEl.textContent = Math.round(start + (end - start) * eased);
        if (progress < 1) requestAnimationFrame(update);
      }
      requestAnimationFrame(update);
    }

    document.getElementById('count-start').addEventListener('click', () => {
      animateCounter(0, 100, 2000);
    });
  </script>
</body>
</html>
```

---

## Step 257: CSS Animations ที่ควบคุมด้วย JS

```html
<!DOCTYPE html>
<html lang="th">
<head>
  <style>
    @keyframes spin {
      from { transform: rotate(0deg); }
      to { transform: rotate(360deg); }
    }
    @keyframes pulse {
      0%, 100% { transform: scale(1); opacity: 1; }
      50% { transform: scale(1.1); opacity: 0.8; }
    }
    @keyframes slide-in {
      from { transform: translateX(-100%); opacity: 0; }
      to { transform: translateX(0); opacity: 1; }
    }
    @keyframes bounce {
      0%, 100% { transform: translateY(0); animation-timing-function: ease-out; }
      50% { transform: translateY(-30px); animation-timing-function: ease-in; }
    }
    @keyframes shake {
      0%, 100% { transform: translateX(0); }
      10%, 30%, 50%, 70%, 90% { transform: translateX(-5px); }
      20%, 40%, 60%, 80% { transform: translateX(5px); }
    }

    .spinner {
      width: 50px; height: 50px;
      border: 5px solid #ddd;
      border-top-color: #4285f4;
      border-radius: 50%;
    }
    .box { width: 100px; height: 100px; background: #4285f4; border-radius: 8px; }
    .error-input { border: 2px solid red; }

    /* CSS animation classes */
    .spinning { animation: spin 1s linear infinite; }
    .pulsing { animation: pulse 1s ease-in-out infinite; }
    .sliding-in { animation: slide-in 0.5s ease forwards; }
    .bouncing { animation: bounce 0.6s ease-in-out infinite; }
    .shaking { animation: shake 0.5s ease-in-out; }
  </style>
</head>
<body>
  <div class="spinner" id="spinner"></div>
  <div class="box" id="box"></div>
  <input type="text" id="error-input" placeholder="Click me to see shake">

  <button id="start-spin">Spin</button>
  <button id="stop-spin">Stop Spin</button>
  <button id="pulse">Pulse</button>
  <button id="slide">Slide</button>
  <button id="bounce">Bounce</button>
  <button id="shake-input">Shake Input</button>

  <div id="notification-container" style="position:fixed;top:10px;right:10px;"></div>

  <script>
    const spinner = document.getElementById('spinner');
    const box = document.getElementById('box');
    const errorInput = document.getElementById('error-input');

    // เพิ่ม/ลบ animation class
    document.getElementById('start-spin').addEventListener('click', () => {
      spinner.classList.add('spinning');
    });

    document.getElementById('stop-spin').addEventListener('click', () => {
      spinner.classList.remove('spinning');
    });

    document.getElementById('pulse').addEventListener('click', () => {
      box.classList.toggle('pulsing');
    });

    document.getElementById('slide').addEventListener('click', () => {
      box.classList.remove('sliding-in');
      // Force reflow เพื่อ restart animation
      void box.offsetWidth;
      box.classList.add('sliding-in');
    });

    document.getElementById('bounce').addEventListener('click', () => {
      box.classList.toggle('bouncing');
    });

    // Shake animation บน input error
    document.getElementById('shake-input').addEventListener('click', () => {
      errorInput.classList.add('error-input', 'shaking');
      errorInput.addEventListener('animationend', () => {
        errorInput.classList.remove('shaking');
      }, { once: true });
    });

    // Web Animations API (WAAPI) - modern approach
    function bounceElement(element) {
      return element.animate([
        { transform: 'translateY(0)', offset: 0 },
        { transform: 'translateY(-30px)', offset: 0.5 },
        { transform: 'translateY(0)', offset: 1 },
      ], {
        duration: 600,
        easing: 'ease-in-out',
        iterations: 3,
      });
    }

    // Control animation playback
    const spinAnimation = spinner.animate(
      [{ transform: 'rotate(0deg)' }, { transform: 'rotate(360deg)' }],
      { duration: 1000, iterations: Infinity }
    );
    spinAnimation.pause(); // เริ่ม pause

    document.getElementById('start-spin').addEventListener('click', () => {
      spinAnimation.play();
    });
    document.getElementById('stop-spin').addEventListener('click', () => {
      spinAnimation.pause();
    });

    // Animation events
    spinner.addEventListener('animationstart', () => console.log('Animation started'));
    spinner.addEventListener('animationend', () => console.log('Animation ended'));
    spinner.addEventListener('animationiteration', () => console.log('Animation looped'));

    // Notification toast
    function showToast(message, type = 'info', duration = 3000) {
      const container = document.getElementById('notification-container');
      const toast = document.createElement('div');

      const colors = { info: '#2196F3', success: '#4CAF50', warning: '#FF9800', error: '#F44336' };
      toast.style.cssText = `
        background: ${colors[type]};
        color: white;
        padding: 12px 20px;
        border-radius: 8px;
        margin-bottom: 8px;
        min-width: 200px;
        box-shadow: 0 4px 12px rgba(0,0,0,0.3);
        animation: slide-in 0.3s ease;
      `;
      toast.textContent = message;
      container.appendChild(toast);

      setTimeout(() => {
        toast.animate(
          [{ opacity: 1, transform: 'translateX(0)' }, { opacity: 0, transform: 'translateX(100%)' }],
          { duration: 300, fill: 'forwards' }
        ).onfinish = () => toast.remove();
      }, duration);
    }

    showToast('ยินดีต้อนรับ!', 'success');
    setTimeout(() => showToast('การเชื่อมต่อขาดหาย', 'error'), 1000);
  </script>
</body>
</html>
```

---

## Step 258: getBoundingClientRect และ Dimensions

```html
<!DOCTYPE html>
<html lang="th">
<body>
  <div id="target" style="
    width: 200px;
    height: 100px;
    background: #4285f4;
    color: white;
    margin: 50px;
    padding: 20px;
    border: 5px solid #1565c0;
    position: relative;
  ">Target Element</div>

  <div id="info" style="background:#f5f5f5;padding:10px;font-family:monospace;"></div>

  <script>
    const target = document.getElementById('target');
    const info = document.getElementById('info');

    function updateInfo() {
      const rect = target.getBoundingClientRect();

      // getBoundingClientRect - relative to viewport
      // top, left, right, bottom, width, height, x, y
      info.innerHTML = `
        <strong>getBoundingClientRect():</strong><br>
        top: ${Math.round(rect.top)}, bottom: ${Math.round(rect.bottom)}<br>
        left: ${Math.round(rect.left)}, right: ${Math.round(rect.right)}<br>
        width: ${Math.round(rect.width)}, height: ${Math.round(rect.height)}<br>
        x: ${Math.round(rect.x)}, y: ${Math.round(rect.y)}<br>
        <br>
        <strong>offsetTop/Left (relative to offsetParent):</strong><br>
        offsetTop: ${target.offsetTop}, offsetLeft: ${target.offsetLeft}<br>
        <br>
        <strong>offsetWidth/Height (including padding & border):</strong><br>
        offsetWidth: ${target.offsetWidth}, offsetHeight: ${target.offsetHeight}<br>
        <br>
        <strong>clientWidth/Height (including padding, NOT border):</strong><br>
        clientWidth: ${target.clientWidth}, clientHeight: ${target.clientHeight}<br>
        clientTop: ${target.clientTop}, clientLeft: ${target.clientLeft}<br>
        <br>
        <strong>scrollWidth/Height:</strong><br>
        scrollWidth: ${target.scrollWidth}, scrollHeight: ${target.scrollHeight}<br>
        scrollTop: ${target.scrollTop}, scrollLeft: ${target.scrollLeft}<br>
      `;
    }

    updateInfo();
    window.addEventListener('scroll', updateInfo);
    window.addEventListener('resize', updateInfo);

    // ตรวจสอบว่า element อยู่ใน viewport
    function isInViewport(element) {
      const rect = element.getBoundingClientRect();
      return (
        rect.top >= 0 &&
        rect.left >= 0 &&
        rect.bottom <= (window.innerHeight || document.documentElement.clientHeight) &&
        rect.right <= (window.innerWidth || document.documentElement.clientWidth)
      );
    }

    // ตรวจสอบว่า element อยู่ใน viewport บางส่วน
    function isPartiallyVisible(element) {
      const rect = element.getBoundingClientRect();
      const windowHeight = window.innerHeight || document.documentElement.clientHeight;
      const windowWidth = window.innerWidth || document.documentElement.clientWidth;
      return !(rect.bottom < 0 || rect.top > windowHeight || rect.right < 0 || rect.left > windowWidth);
    }

    // คำนวณ position relative to document (ไม่ใช่ viewport)
    function getDocumentOffset(element) {
      const rect = element.getBoundingClientRect();
      return {
        top: rect.top + window.pageYOffset,
        left: rect.left + window.pageXOffset,
      };
    }

    // ระยะห่างระหว่าง elements
    function getDistance(el1, el2) {
      const r1 = el1.getBoundingClientRect();
      const r2 = el2.getBoundingClientRect();
      const c1 = { x: r1.left + r1.width / 2, y: r1.top + r1.height / 2 };
      const c2 = { x: r2.left + r2.width / 2, y: r2.top + r2.height / 2 };
      return Math.sqrt(Math.pow(c2.x - c1.x, 2) + Math.pow(c2.y - c1.y, 2));
    }

    // ตรวจสอบ overlap
    function isOverlapping(el1, el2) {
      const r1 = el1.getBoundingClientRect();
      const r2 = el2.getBoundingClientRect();
      return !(r1.right < r2.left || r1.left > r2.right ||
               r1.bottom < r2.top || r1.top > r2.bottom);
    }

    // เลื่อนไปที่ element
    function scrollToElement(element, offset = 0) {
      const top = getDocumentOffset(element).top - offset;
      window.scrollTo({ top, behavior: 'smooth' });
    }
  </script>
</body>
</html>
```

---

## Step 259: Scroll Control

```html
<!DOCTYPE html>
<html lang="th">
<head>
  <style>
    #scroll-container { height: 400px; overflow-y: scroll; border: 1px solid #ccc; }
    #scroll-content { height: 2000px; background: linear-gradient(to bottom, #e3f2fd, #bbdefb); padding: 20px; }
    .sticky-header { position: sticky; top: 0; background: #4285f4; color: white; padding: 10px; z-index: 10; }
    .section { padding: 20px; margin: 20px 0; background: white; border-radius: 8px; }
    #progress-bar { position: fixed; top: 0; left: 0; height: 4px; background: #4285f4; width: 0; z-index: 1000; }
    #toc { position: fixed; right: 20px; top: 100px; }
    #toc a { display: block; padding: 5px; text-decoration: none; color: #666; font-size: 0.85em; }
    #toc a.active { color: #4285f4; font-weight: bold; }
  </style>
</head>
<body>
  <div id="progress-bar"></div>

  <div id="toc">
    <strong>สารบัญ</strong>
    <a href="#section1">ส่วนที่ 1</a>
    <a href="#section2">ส่วนที่ 2</a>
    <a href="#section3">ส่วนที่ 3</a>
  </div>

  <h1 id="section1" class="section">ส่วนที่ 1 - Scroll Control</h1>
  <div style="height:600px;background:#e8f5e9;padding:20px;">เนื้อหา 1...</div>

  <h1 id="section2" class="section">ส่วนที่ 2</h1>
  <div style="height:600px;background:#fff3e0;padding:20px;">เนื้อหา 2...</div>

  <h1 id="section3" class="section">ส่วนที่ 3</h1>
  <div style="height:600px;background:#fce4ec;padding:20px;">เนื้อหา 3...</div>

  <button id="back-top" style="position:fixed;bottom:20px;right:20px;display:none;padding:10px 15px;background:#4285f4;color:white;border:none;border-radius:50%;cursor:pointer;">↑</button>

  <script>
    const progressBar = document.getElementById('progress-bar');
    const backTop = document.getElementById('back-top');

    // window.scroll methods
    // window.scrollTo(x, y) - scroll to absolute position
    // window.scrollBy(x, y) - scroll by amount
    // window.scrollTo({ top, left, behavior })

    // Scroll progress bar
    window.addEventListener('scroll', () => {
      const scrollTop = window.scrollY;
      const docHeight = document.documentElement.scrollHeight - window.innerHeight;
      const progress = docHeight > 0 ? (scrollTop / docHeight) * 100 : 0;
      progressBar.style.width = progress + '%';

      // Back to top button
      backTop.style.display = scrollTop > 300 ? 'block' : 'none';

      // Active TOC item
      const sections = document.querySelectorAll('[id^="section"]');
      const tocLinks = document.querySelectorAll('#toc a');

      sections.forEach((section, i) => {
        const rect = section.getBoundingClientRect();
        if (rect.top <= 100 && rect.bottom >= 100) {
          tocLinks.forEach(link => link.classList.remove('active'));
          tocLinks[i]?.classList.add('active');
        }
      });
    });

    // Back to top
    backTop.addEventListener('click', () => {
      window.scrollTo({ top: 0, behavior: 'smooth' });
    });

    // Smooth scroll for TOC links
    document.querySelectorAll('#toc a').forEach(link => {
      link.addEventListener('click', (e) => {
        e.preventDefault();
        const target = document.querySelector(link.getAttribute('href'));
        if (target) {
          const offset = 80;
          const top = target.getBoundingClientRect().top + window.scrollY - offset;
          window.scrollTo({ top, behavior: 'smooth' });
        }
      });
    });

    // element.scrollIntoView
    // element.scrollIntoView(true) - scroll จน element อยู่ top
    // element.scrollIntoView(false) - scroll จน element อยู่ bottom
    // element.scrollIntoView({ behavior: 'smooth', block: 'center' })

    // ตรวจสอบ scroll direction
    let lastScrollY = 0;
    let scrollDirection = 'down';

    window.addEventListener('scroll', () => {
      const currentScrollY = window.scrollY;
      scrollDirection = currentScrollY > lastScrollY ? 'down' : 'up';
      lastScrollY = currentScrollY;
    });

    // Lock scroll (เช่น เมื่อเปิด modal)
    function lockScroll() {
      const scrollY = window.scrollY;
      document.body.style.position = 'fixed';
      document.body.style.top = `-${scrollY}px`;
      document.body.style.width = '100%';
    }

    function unlockScroll() {
      const scrollY = parseInt(document.body.style.top || '0', 10) * -1;
      document.body.style.position = '';
      document.body.style.top = '';
      document.body.style.width = '';
      window.scrollTo(0, scrollY);
    }

    // Element scroll
    // element.scrollTop = 0; - scroll element to top
    // element.scrollTo({ top: 100, behavior: 'smooth' });
    // element.scrollBy({ top: 50, behavior: 'smooth' });
  </script>
</body>
</html>
```

---

## Step 260: Element Visibility Detection

```html
<!DOCTYPE html>
<html lang="th">
<head>
  <style>
    .reveal-item {
      opacity: 0;
      transform: translateY(40px);
      transition: opacity 0.6s ease, transform 0.6s ease;
      padding: 30px;
      margin: 30px 0;
      background: white;
      border-radius: 12px;
      box-shadow: 0 2px 10px rgba(0,0,0,0.1);
    }
    .reveal-item.revealed {
      opacity: 1;
      transform: translateY(0);
    }
    .lazy-img {
      width: 100%;
      height: 200px;
      background: #eee;
      object-fit: cover;
    }
    .counter-item { text-align: center; padding: 20px; }
    .counter-number { font-size: 48px; font-weight: bold; color: #4285f4; }
  </style>
</head>
<body>
  <div style="height:100vh;display:flex;align-items:center;justify-content:center;background:#f5f5f5;">
    <h1>เลื่อนลงเพื่อดู effects ↓</h1>
  </div>

  <div class="reveal-item">
    <h2>Reveal on Scroll</h2>
    <p>เนื้อหานี้จะ fade in เมื่อ scroll มาถึง</p>
    <img data-src="https://picsum.photos/600/200?random=1" class="lazy-img" alt="Lazy">
  </div>

  <div class="reveal-item" style="transition-delay: 0.2s;">
    <h2>Card 2</h2>
    <p>การเลื่อนลงมาเพื่อดูเนื้อหา</p>
    <img data-src="https://picsum.photos/600/200?random=2" class="lazy-img" alt="Lazy">
  </div>

  <div class="reveal-item counter-item">
    <div class="counter-number" data-target="1500">0</div>
    <p>Users</p>
  </div>

  <div class="reveal-item counter-item">
    <div class="counter-number" data-target="250">0</div>
    <p>Projects</p>
  </div>

  <script>
    // Intersection Observer for reveal
    const revealObserver = new IntersectionObserver((entries) => {
      entries.forEach(entry => {
        if (entry.isIntersecting) {
          entry.target.classList.add('revealed');
          revealObserver.unobserve(entry.target);
        }
      });
    }, { threshold: 0.15 });

    document.querySelectorAll('.reveal-item').forEach(el => {
      revealObserver.observe(el);
    });

    // Lazy loading images
    const imgObserver = new IntersectionObserver((entries) => {
      entries.forEach(entry => {
        if (entry.isIntersecting) {
          const img = entry.target;
          if (img.dataset.src) {
            img.src = img.dataset.src;
            img.removeAttribute('data-src');
            imgObserver.unobserve(img);
          }
        }
      });
    }, { rootMargin: '100px' }); // load ก่อน 100px

    document.querySelectorAll('img[data-src]').forEach(img => {
      imgObserver.observe(img);
    });

    // Counter animation on visibility
    const counterObserver = new IntersectionObserver((entries) => {
      entries.forEach(entry => {
        if (entry.isIntersecting) {
          animateCounter(entry.target);
          counterObserver.unobserve(entry.target);
        }
      });
    }, { threshold: 0.5 });

    function animateCounter(el) {
      const target = Number(el.dataset.target);
      const duration = 2000;
      const startTime = performance.now();

      function update(timestamp) {
        const elapsed = timestamp - startTime;
        const progress = Math.min(elapsed / duration, 1);
        const eased = 1 - Math.pow(1 - progress, 3);
        el.textContent = Math.round(target * eased).toLocaleString();
        if (progress < 1) requestAnimationFrame(update);
      }
      requestAnimationFrame(update);
    }

    document.querySelectorAll('.counter-number').forEach(el => {
      counterObserver.observe(el);
    });

    // Video autoplay on visibility
    document.querySelectorAll('video[data-autoplay]').forEach(video => {
      const videoObserver = new IntersectionObserver((entries) => {
        entries.forEach(entry => {
          if (entry.isIntersecting) {
            video.play();
          } else {
            video.pause();
          }
        });
      }, { threshold: 0.5 });
      videoObserver.observe(video);
    });
  </script>
</body>
</html>
```

---

## Step 261: Dark Mode Toggle

```html
<!DOCTYPE html>
<html lang="th" data-theme="light">
<head>
  <style>
    :root[data-theme="light"] {
      --bg: #ffffff;
      --surface: #f5f5f5;
      --text: #333333;
      --text-secondary: #666666;
      --border: #dddddd;
      --primary: #4285f4;
      --shadow: rgba(0,0,0,0.1);
    }

    :root[data-theme="dark"] {
      --bg: #121212;
      --surface: #1e1e1e;
      --text: #e0e0e0;
      --text-secondary: #aaaaaa;
      --border: #333333;
      --primary: #82b1ff;
      --shadow: rgba(0,0,0,0.5);
    }

    * { transition: background-color 0.3s ease, color 0.3s ease, border-color 0.3s ease; }

    body { background: var(--bg); color: var(--text); font-family: sans-serif; margin: 0; padding: 20px; }

    .card {
      background: var(--surface);
      border: 1px solid var(--border);
      border-radius: 12px;
      padding: 20px;
      margin: 16px 0;
      box-shadow: 0 2px 8px var(--shadow);
    }

    .nav {
      display: flex;
      justify-content: space-between;
      align-items: center;
      padding: 12px 20px;
      background: var(--surface);
      border-bottom: 1px solid var(--border);
    }

    .theme-toggle {
      background: var(--surface);
      border: 2px solid var(--border);
      border-radius: 50px;
      padding: 6px 12px;
      cursor: pointer;
      font-size: 16px;
      transition: all 0.3s;
    }

    .theme-toggle:hover { background: var(--primary); color: white; }

    p { color: var(--text-secondary); }
    a { color: var(--primary); }
  </style>
</head>
<body>
  <nav class="nav">
    <h2>My App</h2>
    <button class="theme-toggle" id="theme-btn">🌙 Dark Mode</button>
  </nav>

  <div class="card">
    <h2>Card Title</h2>
    <p>เนื้อหาของ card นี้จะเปลี่ยนสีตาม theme</p>
    <a href="#">อ่านเพิ่มเติม</a>
  </div>

  <div class="card">
    <h2>Another Card</h2>
    <p>Dark mode ใช้ CSS variables ทำให้ switch ง่าย</p>
  </div>

  <script>
    const html = document.documentElement;
    const themeBtn = document.getElementById('theme-btn');

    // โหลด theme จาก localStorage
    const savedTheme = localStorage.getItem('theme') || 
      (window.matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light');
    
    setTheme(savedTheme);

    function setTheme(theme) {
      html.setAttribute('data-theme', theme);
      localStorage.setItem('theme', theme);
      
      if (theme === 'dark') {
        themeBtn.textContent = '☀️ Light Mode';
      } else {
        themeBtn.textContent = '🌙 Dark Mode';
      }
    }

    themeBtn.addEventListener('click', () => {
      const currentTheme = html.getAttribute('data-theme');
      setTheme(currentTheme === 'dark' ? 'light' : 'dark');
    });

    // System theme preference change
    window.matchMedia('(prefers-color-scheme: dark)').addEventListener('change', (e) => {
      if (!localStorage.getItem('theme')) { // only if user hasn't set manually
        setTheme(e.matches ? 'dark' : 'light');
      }
    });

    // Three-way theme switcher (light, dark, system)
    function applySystemTheme() {
      const isDark = window.matchMedia('(prefers-color-scheme: dark)').matches;
      html.setAttribute('data-theme', isDark ? 'dark' : 'light');
    }

    // ตัวอย่าง: theme with more options
    const themes = {
      light: { '--bg': '#fff', '--text': '#333', '--primary': '#4285f4' },
      dark: { '--bg': '#121212', '--text': '#e0e0e0', '--primary': '#82b1ff' },
      sepia: { '--bg': '#f5f0e8', '--text': '#5c4033', '--primary': '#8b5e3c' },
    };

    function applyTheme(themeName) {
      const theme = themes[themeName];
      if (!theme) return;
      Object.entries(theme).forEach(([key, value]) => {
        html.style.setProperty(key, value);
      });
    }
  </script>
</body>
</html>
```

---

## Steps 262-270: เพิ่มเติม CSS Manipulation

### Step 262: Dynamic Stylesheet

```javascript
// สร้าง stylesheet ใหม่
const style = document.createElement('style');
style.id = 'dynamic-styles';
document.head.appendChild(style);

// เพิ่ม rules
const sheet = style.sheet;
sheet.insertRule('.highlight { background: yellow; }', 0);
sheet.insertRule('.bold { font-weight: bold; }', 1);

console.log(sheet.cssRules.length); // 2
console.log(sheet.cssRules[0].cssText); // ".highlight { background: yellow; }"

// ลบ rule
sheet.deleteRule(0);

// แก้ไข rule
sheet.insertRule(':root { --brand: #e91e63; }', 0);

// ใช้ CSSStyleSheet API (modern)
const newSheet = new CSSStyleSheet();
newSheet.insertRule('.dynamic { color: red; }');
document.adoptedStyleSheets = [...document.adoptedStyleSheets, newSheet];

// inline style injection
function injectStyles(css) {
  const style = document.createElement('style');
  style.textContent = css;
  document.head.appendChild(style);
  return style;
}

const injected = injectStyles(`
  .theme-red { --primary: #e53935; }
  .theme-green { --primary: #43a047; }
`);

// cleanup
// injected.remove();
```

### Step 263: Element Positioning

```javascript
// การ position element ด้วย JS

// Tooltip ที่ follow cursor
function createTooltip(content) {
  const tooltip = document.createElement('div');
  tooltip.style.cssText = `
    position: fixed;
    background: rgba(0,0,0,0.8);
    color: white;
    padding: 6px 12px;
    border-radius: 4px;
    font-size: 14px;
    pointer-events: none;
    z-index: 9999;
    transform: translate(-50%, -120%);
    white-space: nowrap;
  `;
  tooltip.textContent = content;
  document.body.appendChild(tooltip);

  document.addEventListener('mousemove', (e) => {
    tooltip.style.left = e.clientX + 'px';
    tooltip.style.top = e.clientY + 'px';
  });

  return tooltip;
}

// Position dropdown below trigger
function positionDropdown(trigger, dropdown) {
  const rect = trigger.getBoundingClientRect();
  const dropRect = dropdown.getBoundingClientRect();
  const viewportH = window.innerHeight;

  let top = rect.bottom + window.scrollY;
  let left = rect.left + window.scrollX;

  // ถ้าออกนอก viewport ด้านล่าง ให้แสดงด้านบน
  if (rect.bottom + dropRect.height > viewportH) {
    top = rect.top + window.scrollY - dropRect.height;
  }

  // ถ้าออกนอก viewport ด้านขวา ให้ align ขวา
  if (left + dropRect.width > window.innerWidth) {
    left = rect.right + window.scrollX - dropRect.width;
  }

  dropdown.style.position = 'absolute';
  dropdown.style.top = top + 'px';
  dropdown.style.left = left + 'px';
}

// Sticky element
function makeSticky(element, offset = 0) {
  const originalTop = element.getBoundingClientRect().top + window.scrollY;

  window.addEventListener('scroll', () => {
    if (window.scrollY + offset >= originalTop) {
      element.style.position = 'fixed';
      element.style.top = offset + 'px';
    } else {
      element.style.position = '';
      element.style.top = '';
    }
  });
}
```

### Step 264: Print Styles

```javascript
// Print style control
function printPage() {
  window.print();
}

// เพิ่ม print styles
const printStyle = document.createElement('style');
printStyle.media = 'print';
printStyle.textContent = `
  .no-print { display: none !important; }
  .print-only { display: block !important; }
  body { font-size: 12pt; }
  a { color: black; text-decoration: none; }
  a[href]::after { content: " (" attr(href) ")"; }
`;
document.head.appendChild(printStyle);

// Before/After print events
window.addEventListener('beforeprint', () => {
  document.body.classList.add('printing');
  console.log('About to print');
});

window.addEventListener('afterprint', () => {
  document.body.classList.remove('printing');
  console.log('Done printing');
});

// Print specific element
function printElement(selector) {
  const element = document.querySelector(selector);
  if (!element) return;

  const original = document.body.innerHTML;
  document.body.innerHTML = element.outerHTML;
  window.print();
  document.body.innerHTML = original;
  location.reload();
}
```

### Step 265-270: Animation Helpers

```javascript
// Easing functions
const Easing = {
  linear: t => t,
  easeIn: t => t * t * t,
  easeOut: t => 1 - Math.pow(1 - t, 3),
  easeInOut: t => t < 0.5 ? 4 * t * t * t : 1 - Math.pow(-2 * t + 2, 3) / 2,
  elastic: t => t === 0 ? 0 : t === 1 ? 1 : Math.pow(2, -10 * t) * Math.sin((t * 10 - 0.75) * ((2 * Math.PI) / 3)) + 1,
  bounce: t => {
    const n1 = 7.5625, d1 = 2.75;
    if (t < 1 / d1) return n1 * t * t;
    if (t < 2 / d1) return n1 * (t -= 1.5 / d1) * t + 0.75;
    if (t < 2.5 / d1) return n1 * (t -= 2.25 / d1) * t + 0.9375;
    return n1 * (t -= 2.625 / d1) * t + 0.984375;
  },
};

// Generic animate function
function animate({ duration, from, to, easing = Easing.easeOut, onUpdate, onDone }) {
  const startTime = performance.now();
  function frame(timestamp) {
    const elapsed = timestamp - startTime;
    const progress = Math.min(elapsed / duration, 1);
    const easedProgress = easing(progress);
    const value = from + (to - from) * easedProgress;
    onUpdate(value, easedProgress);
    if (progress < 1) {
      requestAnimationFrame(frame);
    } else {
      onDone?.();
    }
  }
  requestAnimationFrame(frame);
}

// ใช้งาน
animate({
  duration: 1000,
  from: 0,
  to: 100,
  easing: Easing.bounce,
  onUpdate: (value) => {
    document.getElementById('counter').textContent = Math.round(value);
  },
  onDone: () => console.log('Done!'),
});

// Animate multiple properties
function animateElement(element, properties, duration, easing = Easing.easeOut) {
  const startValues = {};
  const endValues = {};

  Object.entries(properties).forEach(([prop, end]) => {
    startValues[prop] = parseFloat(getComputedStyle(element)[prop]) || 0;
    endValues[prop] = end;
  });

  animate({
    duration,
    from: 0,
    to: 1,
    easing,
    onUpdate: (t) => {
      Object.entries(endValues).forEach(([prop, end]) => {
        const start = startValues[prop];
        element.style[prop] = `${start + (end - start) * t}px`;
      });
    },
  });
}

// ตัวอย่าง: animate width และ height
// animateElement(box, { width: 300, height: 200 }, 500);
```

---

## สรุป Steps 251-270

| Step | หัวข้อ |
|------|--------|
| 251 | Inline Styles |
| 252 | getComputedStyle |
| 253 | CSS Custom Properties |
| 254 | classList Methods |
| 255 | CSS Transitions |
| 256 | requestAnimationFrame |
| 257 | CSS Animations |
| 258 | getBoundingClientRect |
| 259 | Scroll Control |
| 260 | Visibility Detection |
| 261 | Dark Mode Toggle |
| 262 | Dynamic Stylesheets |
| 263 | Element Positioning |
| 264 | Print Styles |
| 265-270 | Animation Helpers |

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: Theme Switcher
สร้าง theme switcher ที่:
- มี 4+ themes
- บันทึก theme ใน localStorage
- Smooth transition ระหว่าง themes
- รองรับ system dark/light mode

### แบบฝึกหัดที่ 2: Parallax Effect
สร้าง parallax scrolling ที่:
- ใช้ requestAnimationFrame
- หลาย layers ที่ scroll ต่างความเร็ว
- Optimized สำหรับ performance

### แบบฝึกหัดที่ 3: Animated Card Gallery
สร้าง gallery ที่:
- Cards reveal on scroll
- Hover animations
- Click animation
- Staggered entrance

### แบบฝึกหัดที่ 4: CSS Variable Theme Builder
สร้าง visual theme editor ที่:
- แก้ไข colors, fonts, sizes
- Live preview
- Export CSS variables
- Save/Load themes

### แบบฝึกหัดที่ 5: Progress Loader
สร้าง progress loader ที่:
- Animated progress bar
- Circular progress indicator
- Percentage counter
- Different easing functions
