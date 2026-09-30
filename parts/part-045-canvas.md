# Part 45: Canvas API (Steps 871-890)

## บทนำ

HTML5 Canvas API ให้ความสามารถในการวาดกราฟิก 2D (และ 3D ด้วย WebGL) โดยตรงบนเว็บไซต์ด้วย JavaScript ไม่ว่าจะเป็นการวาดรูปทรงเรขาคณิต, ข้อความ, รูปภาพ, animation, หรือแม้แต่เกม

ในบทนี้เราจะเรียนรู้:
- Canvas element และ rendering context
- การวาดรูปทรงต่างๆ
- Styles, gradients, patterns
- Transformations
- Animation
- Pixel manipulation
- การสร้าง Drawing Application

---

## Step 871: HTML5 Canvas Element

```javascript
// HTML:
// <canvas id="myCanvas" width="800" height="600"></canvas>

// JavaScript - การเริ่มต้นใช้งาน Canvas
const canvas = document.getElementById('myCanvas');

// ตรวจสอบการรองรับ
if (!canvas.getContext) {
  console.error('Browser ไม่รองรับ Canvas');
}

// ดึง 2D rendering context
const ctx = canvas.getContext('2d');

// Canvas attributes
console.log('Width:', canvas.width);   // 800
console.log('Height:', canvas.height); // 600

// ตั้งค่า size ด้วย JavaScript
canvas.width = 1200;
canvas.height = 800;

// หมายเหตุ: CSS size vs Canvas resolution
// canvas.width/height = ความละเอียดของ pixel buffer
// CSS width/height = ขนาดที่แสดงบนหน้าจอ
// ถ้า CSS size ≠ canvas size จะมีการ scale

// High DPI / Retina support
function createHiDPICanvas(width, height) {
  const devicePixelRatio = window.devicePixelRatio || 1;
  const canvas = document.createElement('canvas');
  
  // ตั้ง CSS size
  canvas.style.width = width + 'px';
  canvas.style.height = height + 'px';
  
  // ตั้ง actual resolution (x2 สำหรับ Retina)
  canvas.width = width * devicePixelRatio;
  canvas.height = height * devicePixelRatio;
  
  const ctx = canvas.getContext('2d');
  
  // Scale context เพื่อชดเชย
  ctx.scale(devicePixelRatio, devicePixelRatio);
  
  return { canvas, ctx };
}

const { canvas: hiDpiCanvas, ctx: hiDpiCtx } = createHiDPICanvas(800, 600);
document.body.appendChild(hiDpiCanvas);

// Canvas coordinate system
// (0,0) = top-left corner
// x เพิ่มไปทางขวา
// y เพิ่มลงล่าง
// (canvas.width, canvas.height) = bottom-right corner
```

---

## Step 872: Drawing Rectangles

```javascript
// วาดสี่เหลี่ยม - วิธีที่ง่ายที่สุด

const canvas = document.getElementById('myCanvas');
const ctx = canvas.getContext('2d');

// fillRect(x, y, width, height) - สี่เหลี่ยมทึบ
ctx.fillStyle = 'blue';
ctx.fillRect(10, 10, 200, 100);

ctx.fillStyle = 'red';
ctx.fillRect(50, 50, 150, 100); // ทับบางส่วน

// strokeRect(x, y, width, height) - กรอบสี่เหลี่ยม
ctx.strokeStyle = 'green';
ctx.lineWidth = 3;
ctx.strokeRect(200, 50, 150, 100);

// clearRect(x, y, width, height) - ลบพื้นที่
ctx.clearRect(80, 70, 60, 40); // ลบส่วนที่ทับซ้อน

// วาด rectangles หลายอัน
function drawRectangleGrid(ctx, cols, rows, cellSize, gap) {
  const colors = ['#FF6B6B', '#4ECDC4', '#45B7D1', '#96CEB4', '#FFEAA7'];
  
  for (let row = 0; row < rows; row++) {
    for (let col = 0; col < cols; col++) {
      const x = col * (cellSize + gap) + gap;
      const y = row * (cellSize + gap) + gap;
      const colorIndex = (row * cols + col) % colors.length;
      
      ctx.fillStyle = colors[colorIndex];
      ctx.fillRect(x, y, cellSize, cellSize);
    }
  }
}

drawRectangleGrid(ctx, 5, 4, 60, 5);

// Rounded Rectangle (RoundRect) - ES2022
function drawRoundedRect(ctx, x, y, width, height, radius) {
  if (typeof ctx.roundRect === 'function') {
    // Modern API
    ctx.beginPath();
    ctx.roundRect(x, y, width, height, radius);
    ctx.fill();
    ctx.stroke();
  } else {
    // Fallback
    ctx.beginPath();
    ctx.moveTo(x + radius, y);
    ctx.lineTo(x + width - radius, y);
    ctx.arcTo(x + width, y, x + width, y + radius, radius);
    ctx.lineTo(x + width, y + height - radius);
    ctx.arcTo(x + width, y + height, x + width - radius, y + height, radius);
    ctx.lineTo(x + radius, y + height);
    ctx.arcTo(x, y + height, x, y + height - radius, radius);
    ctx.lineTo(x, y + radius);
    ctx.arcTo(x, y, x + radius, y, radius);
    ctx.closePath();
    ctx.fill();
    ctx.stroke();
  }
}

ctx.fillStyle = '#3498db';
ctx.strokeStyle = '#2980b9';
ctx.lineWidth = 2;
drawRoundedRect(ctx, 300, 200, 200, 100, 20);
```

---

## Step 873: Drawing Paths

```javascript
// Paths - วาดเส้นและรูปทรงซับซ้อน

const ctx = canvas.getContext('2d');

// Path พื้นฐาน
ctx.beginPath();           // เริ่ม path ใหม่
ctx.moveTo(50, 50);        // ย้าย cursor ไปที่จุดเริ่มต้น
ctx.lineTo(200, 50);       // วาดเส้นไปยัง
ctx.lineTo(200, 150);      // วาดเส้นต่อ
ctx.lineTo(50, 150);       // วาดเส้นต่อ
ctx.closePath();           // ปิด path (เส้นกลับไปยังจุดเริ่มต้น)
ctx.stroke();              // วาดกรอบ
ctx.fillStyle = 'lightblue';
ctx.fill();                // เติมสี

// วาดรูปสามเหลี่ยม
function drawTriangle(ctx, x1, y1, x2, y2, x3, y3, options = {}) {
  ctx.save();
  
  ctx.beginPath();
  ctx.moveTo(x1, y1);
  ctx.lineTo(x2, y2);
  ctx.lineTo(x3, y3);
  ctx.closePath();
  
  if (options.fill) {
    ctx.fillStyle = options.fill;
    ctx.fill();
  }
  
  if (options.stroke) {
    ctx.strokeStyle = options.stroke;
    ctx.lineWidth = options.lineWidth || 1;
    ctx.stroke();
  }
  
  ctx.restore();
}

drawTriangle(ctx, 300, 50, 250, 150, 350, 150, {
  fill: '#e74c3c',
  stroke: '#c0392b',
  lineWidth: 2
});

// วาดดาว
function drawStar(ctx, cx, cy, spikes, outerRadius, innerRadius, color = 'gold') {
  let rot = Math.PI / 2 * 3; // เริ่มจากด้านบน
  const step = Math.PI / spikes;
  
  ctx.save();
  ctx.beginPath();
  ctx.moveTo(cx, cy - outerRadius);
  
  for (let i = 0; i < spikes; i++) {
    // จุดนอก
    ctx.lineTo(
      cx + Math.cos(rot) * outerRadius,
      cy + Math.sin(rot) * outerRadius
    );
    rot += step;
    
    // จุดใน
    ctx.lineTo(
      cx + Math.cos(rot) * innerRadius,
      cy + Math.sin(rot) * innerRadius
    );
    rot += step;
  }
  
  ctx.lineTo(cx, cy - outerRadius);
  ctx.closePath();
  
  ctx.fillStyle = color;
  ctx.fill();
  ctx.strokeStyle = 'orange';
  ctx.lineWidth = 1;
  ctx.stroke();
  
  ctx.restore();
}

drawStar(ctx, 400, 300, 5, 50, 20);

// วาดลูกศร
function drawArrow(ctx, fromX, fromY, toX, toY, options = {}) {
  const headLen = options.headLen || 15;
  const headAngle = options.headAngle || Math.PI / 6;
  const color = options.color || '#333';
  const lineWidth = options.lineWidth || 2;
  
  const angle = Math.atan2(toY - fromY, toX - fromX);
  
  ctx.save();
  ctx.strokeStyle = color;
  ctx.fillStyle = color;
  ctx.lineWidth = lineWidth;
  
  // วาดตัวลูกศร
  ctx.beginPath();
  ctx.moveTo(fromX, fromY);
  ctx.lineTo(toX, toY);
  ctx.stroke();
  
  // วาดหัวลูกศร
  ctx.beginPath();
  ctx.moveTo(toX, toY);
  ctx.lineTo(
    toX - headLen * Math.cos(angle - headAngle),
    toY - headLen * Math.sin(angle - headAngle)
  );
  ctx.lineTo(
    toX - headLen * Math.cos(angle + headAngle),
    toY - headLen * Math.sin(angle + headAngle)
  );
  ctx.closePath();
  ctx.fill();
  
  ctx.restore();
}

drawArrow(ctx, 100, 300, 250, 200, { color: '#3498db', lineWidth: 3 });
```

---

## Step 874: Drawing Circles and Arcs

```javascript
// arc(x, y, radius, startAngle, endAngle, anticlockwise)
// มุมเป็น radians: 0 = ขวา, Math.PI/2 = ล่าง, Math.PI = ซ้าย

const ctx = canvas.getContext('2d');

// วาดวงกลม
ctx.beginPath();
ctx.arc(200, 200, 80, 0, Math.PI * 2); // วงกลมเต็ม
ctx.fillStyle = '#3498db';
ctx.fill();
ctx.strokeStyle = '#2980b9';
ctx.lineWidth = 3;
ctx.stroke();

// วาดครึ่งวงกลม
ctx.beginPath();
ctx.arc(400, 200, 60, 0, Math.PI); // ครึ่งบน
ctx.fillStyle = '#e74c3c';
ctx.fill();

// วาด arc (ส่วนโค้ง)
ctx.beginPath();
ctx.arc(300, 300, 100, Math.PI * 0.25, Math.PI * 0.75);
ctx.strokeStyle = 'purple';
ctx.lineWidth = 5;
ctx.stroke();

// วาด pie chart
function drawPieChart(ctx, cx, cy, radius, data) {
  let startAngle = -Math.PI / 2; // เริ่มจากด้านบน
  
  const total = data.reduce((sum, item) => sum + item.value, 0);
  
  data.forEach(item => {
    const sliceAngle = (item.value / total) * Math.PI * 2;
    
    ctx.beginPath();
    ctx.moveTo(cx, cy);
    ctx.arc(cx, cy, radius, startAngle, startAngle + sliceAngle);
    ctx.closePath();
    
    ctx.fillStyle = item.color;
    ctx.fill();
    ctx.strokeStyle = 'white';
    ctx.lineWidth = 2;
    ctx.stroke();
    
    // Label
    const labelAngle = startAngle + sliceAngle / 2;
    const labelX = cx + Math.cos(labelAngle) * (radius * 0.7);
    const labelY = cy + Math.sin(labelAngle) * (radius * 0.7);
    
    ctx.fillStyle = 'white';
    ctx.font = 'bold 14px Sarabun';
    ctx.textAlign = 'center';
    ctx.textBaseline = 'middle';
    ctx.fillText(`${Math.round(item.value / total * 100)}%`, labelX, labelY);
    
    startAngle += sliceAngle;
  });
}

drawPieChart(ctx, 400, 300, 150, [
  { value: 30, color: '#FF6B6B', label: 'อาหาร' },
  { value: 25, color: '#4ECDC4', label: 'เดินทาง' },
  { value: 20, color: '#45B7D1', label: 'ที่พัก' },
  { value: 15, color: '#96CEB4', label: 'ความบันเทิง' },
  { value: 10, color: '#FFEAA7', label: 'อื่นๆ' }
]);

// Donut chart
function drawDonutChart(ctx, cx, cy, outerRadius, innerRadius, data) {
  let startAngle = -Math.PI / 2;
  const total = data.reduce((sum, item) => sum + item.value, 0);
  
  data.forEach(item => {
    const sliceAngle = (item.value / total) * Math.PI * 2;
    
    ctx.beginPath();
    ctx.arc(cx, cy, outerRadius, startAngle, startAngle + sliceAngle);
    ctx.arc(cx, cy, innerRadius, startAngle + sliceAngle, startAngle, true);
    ctx.closePath();
    
    ctx.fillStyle = item.color;
    ctx.fill();
    ctx.strokeStyle = 'white';
    ctx.lineWidth = 2;
    ctx.stroke();
    
    startAngle += sliceAngle;
  });
  
  // Center text
  ctx.fillStyle = '#333';
  ctx.font = 'bold 20px Sarabun';
  ctx.textAlign = 'center';
  ctx.textBaseline = 'middle';
  ctx.fillText('ยอดขาย', cx, cy - 10);
  ctx.fillText('2024', cx, cy + 15);
}
```

---

## Step 875: Curves - Bezier and Quadratic

```javascript
// Quadratic Bezier Curve - มีจุดควบคุม 1 จุด
// quadraticCurveTo(cpx, cpy, x, y)

const ctx = canvas.getContext('2d');

ctx.beginPath();
ctx.moveTo(50, 200);              // จุดเริ่มต้น
ctx.quadraticCurveTo(
  200, 50,   // Control point
  350, 200   // จุดสิ้นสุด
);
ctx.strokeStyle = '#e74c3c';
ctx.lineWidth = 3;
ctx.stroke();

// แสดง control point
ctx.beginPath();
ctx.arc(200, 50, 5, 0, Math.PI * 2);
ctx.fillStyle = 'red';
ctx.fill();

// Cubic Bezier Curve - มีจุดควบคุม 2 จุด
// bezierCurveTo(cp1x, cp1y, cp2x, cp2y, x, y)

ctx.beginPath();
ctx.moveTo(50, 350);              // จุดเริ่มต้น
ctx.bezierCurveTo(
  150, 200,   // Control point 1
  300, 500,   // Control point 2
  450, 350    // จุดสิ้นสุด
);
ctx.strokeStyle = '#3498db';
ctx.lineWidth = 3;
ctx.stroke();

// Wave effect ด้วย Bezier curves
function drawWave(ctx, x, y, width, amplitude, frequency, color = '#3498db') {
  ctx.beginPath();
  ctx.moveTo(x, y);
  
  const segments = Math.ceil(width / (1 / frequency * 100));
  const segWidth = width / segments;
  
  for (let i = 0; i < segments; i++) {
    const x1 = x + i * segWidth;
    const x2 = x + (i + 1) * segWidth;
    const cp1x = x1 + segWidth / 3;
    const cp2x = x1 + segWidth * 2 / 3;
    const waveY = i % 2 === 0 ? amplitude : -amplitude;
    
    ctx.bezierCurveTo(
      cp1x, y + waveY,
      cp2x, y + waveY,
      x2, y
    );
  }
  
  ctx.strokeStyle = color;
  ctx.lineWidth = 2;
  ctx.stroke();
}

// วาด wave หลายชั้น
for (let i = 0; i < 3; i++) {
  drawWave(ctx, 0, 100 + i * 30, 800, 20, 0.05, 
    `hsla(${200 + i * 30}, 70%, 50%, ${0.5 + i * 0.2})`);
}

// Heart shape
function drawHeart(ctx, cx, cy, size, color = 'red') {
  ctx.save();
  ctx.beginPath();
  
  ctx.moveTo(cx, cy + size / 4);
  
  ctx.bezierCurveTo(
    cx, cy - size / 2,
    cx - size, cy - size / 2,
    cx - size, cy
  );
  
  ctx.bezierCurveTo(
    cx - size, cy + size * 0.7,
    cx, cy + size,
    cx, cy + size * 1.2
  );
  
  ctx.bezierCurveTo(
    cx, cy + size,
    cx + size, cy + size * 0.7,
    cx + size, cy
  );
  
  ctx.bezierCurveTo(
    cx + size, cy - size / 2,
    cx, cy - size / 2,
    cx, cy + size / 4
  );
  
  ctx.fillStyle = color;
  ctx.fill();
  
  ctx.restore();
}
```

---

## Step 876: Styles - Colors, Lines

```javascript
// Line Styles

const ctx = canvas.getContext('2d');

// lineWidth
ctx.lineWidth = 1;
ctx.beginPath();
ctx.moveTo(50, 50); ctx.lineTo(200, 50);
ctx.stroke();

ctx.lineWidth = 5;
ctx.beginPath();
ctx.moveTo(50, 80); ctx.lineTo(200, 80);
ctx.stroke();

ctx.lineWidth = 10;
ctx.beginPath();
ctx.moveTo(50, 120); ctx.lineTo(200, 120);
ctx.stroke();

// lineCap: 'butt', 'round', 'square'
['butt', 'round', 'square'].forEach((cap, i) => {
  ctx.save();
  ctx.lineWidth = 15;
  ctx.lineCap = cap;
  ctx.strokeStyle = ['#e74c3c', '#3498db', '#2ecc71'][i];
  
  ctx.beginPath();
  ctx.moveTo(50, 200 + i * 40);
  ctx.lineTo(200, 200 + i * 40);
  ctx.stroke();
  ctx.restore();
});

// lineJoin: 'miter', 'round', 'bevel'
['miter', 'round', 'bevel'].forEach((join, i) => {
  ctx.save();
  ctx.lineWidth = 15;
  ctx.lineJoin = join;
  ctx.strokeStyle = ['#e74c3c', '#3498db', '#2ecc71'][i];
  
  ctx.beginPath();
  ctx.moveTo(300 + i * 100, 50);
  ctx.lineTo(350 + i * 100, 150);
  ctx.lineTo(400 + i * 100, 50);
  ctx.stroke();
  ctx.restore();
});

// Dashed lines
ctx.setLineDash([10, 5]);           // [ขีด, ช่องว่าง]
ctx.lineDashOffset = 0;             // เลื่อน pattern

ctx.beginPath();
ctx.moveTo(50, 350); ctx.lineTo(400, 350);
ctx.strokeStyle = '#333';
ctx.lineWidth = 2;
ctx.stroke();

ctx.setLineDash([20, 5, 5, 5]);    // Pattern ซับซ้อน
ctx.beginPath();
ctx.moveTo(50, 380); ctx.lineTo(400, 380);
ctx.stroke();

ctx.setLineDash([]);  // กลับเป็น solid line

// Alpha / Transparency
ctx.globalAlpha = 0.5;  // 50% transparent
ctx.fillStyle = 'red';
ctx.fillRect(100, 100, 100, 100);

ctx.globalAlpha = 0.8;
ctx.fillStyle = 'blue';
ctx.fillRect(150, 150, 100, 100);

ctx.globalAlpha = 1.0; // กลับเป็น opaque

// Fill with RGBA colors
ctx.fillStyle = 'rgba(255, 0, 0, 0.3)';
ctx.fillRect(200, 100, 100, 100);
```

---

## Step 877: Gradients

```javascript
// Linear Gradient
function createLinearGradientBackground(ctx, width, height) {
  // createLinearGradient(x0, y0, x1, y1)
  // (x0,y0) = จุดเริ่มต้น, (x1,y1) = จุดสิ้นสุด
  
  const gradient = ctx.createLinearGradient(0, 0, width, 0);
  
  // addColorStop(position, color) - position 0 ถึง 1
  gradient.addColorStop(0, '#FF6B6B');
  gradient.addColorStop(0.5, '#4ECDC4');
  gradient.addColorStop(1, '#45B7D1');
  
  ctx.fillStyle = gradient;
  ctx.fillRect(0, 0, width, height);
  
  return gradient;
}

// Diagonal gradient
const diagGradient = ctx.createLinearGradient(0, 0, 400, 400);
diagGradient.addColorStop(0, 'navy');
diagGradient.addColorStop(1, 'lightblue');

ctx.fillStyle = diagGradient;
ctx.fillRect(0, 0, 400, 400);

// Radial Gradient
function createRadialGlow(ctx, cx, cy, radius, innerColor, outerColor) {
  // createRadialGradient(x0, y0, r0, x1, y1, r1)
  const gradient = ctx.createRadialGradient(cx, cy, 0, cx, cy, radius);
  
  gradient.addColorStop(0, innerColor);
  gradient.addColorStop(1, outerColor);
  
  ctx.fillStyle = gradient;
  ctx.beginPath();
  ctx.arc(cx, cy, radius, 0, Math.PI * 2);
  ctx.fill();
  
  return gradient;
}

// Glowing circle
createRadialGlow(ctx, 200, 200, 100, 'white', 'transparent');
createRadialGlow(ctx, 200, 200, 80, 'yellow', 'transparent');
createRadialGlow(ctx, 200, 200, 50, 'orange', 'transparent');

// Conic Gradient (ES2021+)
// const conicGradient = ctx.createConicGradient(startAngle, x, y);

// Complex gradient - Sunset
function drawSunset(ctx, width, height) {
  const skyGradient = ctx.createLinearGradient(0, 0, 0, height * 0.7);
  skyGradient.addColorStop(0, '#1a1a2e');
  skyGradient.addColorStop(0.4, '#16213e');
  skyGradient.addColorStop(0.7, '#e94560');
  skyGradient.addColorStop(1, '#f5a623');
  
  ctx.fillStyle = skyGradient;
  ctx.fillRect(0, 0, width, height * 0.7);
  
  // Sun glow
  const sunGradient = ctx.createRadialGradient(
    width / 2, height * 0.6, 0,
    width / 2, height * 0.6, 120
  );
  sunGradient.addColorStop(0, 'rgba(255, 255, 255, 0.9)');
  sunGradient.addColorStop(0.2, 'rgba(255, 200, 50, 0.7)');
  sunGradient.addColorStop(0.6, 'rgba(255, 100, 50, 0.3)');
  sunGradient.addColorStop(1, 'rgba(255, 100, 50, 0)');
  
  ctx.fillStyle = sunGradient;
  ctx.fillRect(0, 0, width, height);
  
  // Sea
  const seaGradient = ctx.createLinearGradient(0, height * 0.7, 0, height);
  seaGradient.addColorStop(0, '#2980b9');
  seaGradient.addColorStop(1, '#1a5276');
  
  ctx.fillStyle = seaGradient;
  ctx.fillRect(0, height * 0.7, width, height * 0.3);
  
  // Sun
  ctx.beginPath();
  ctx.arc(width / 2, height * 0.68, 40, 0, Math.PI * 2);
  ctx.fillStyle = '#f39c12';
  ctx.fill();
}

drawSunset(ctx, canvas.width, canvas.height);
```

---

## Step 878: Patterns

```javascript
// createPattern(image, repetition)
// repetition: 'repeat', 'repeat-x', 'repeat-y', 'no-repeat'

// Pattern จาก image
function createImagePattern(ctx, imageUrl, repetition = 'repeat') {
  return new Promise((resolve) => {
    const img = new Image();
    img.onload = () => {
      const pattern = ctx.createPattern(img, repetition);
      resolve(pattern);
    };
    img.src = imageUrl;
  });
}

// Pattern จาก Canvas อื่น (สร้าง pattern เอง)
function createCustomPattern(ctx, size, drawFn) {
  const patternCanvas = document.createElement('canvas');
  patternCanvas.width = size;
  patternCanvas.height = size;
  const patternCtx = patternCanvas.getContext('2d');
  
  drawFn(patternCtx, size);
  
  return ctx.createPattern(patternCanvas, 'repeat');
}

// สร้าง polka dot pattern
const polkaDotPattern = createCustomPattern(ctx, 20, (pCtx, size) => {
  pCtx.fillStyle = '#f0f0f0';
  pCtx.fillRect(0, 0, size, size);
  
  pCtx.beginPath();
  pCtx.arc(size / 2, size / 2, 4, 0, Math.PI * 2);
  pCtx.fillStyle = '#3498db';
  pCtx.fill();
});

ctx.fillStyle = polkaDotPattern;
ctx.fillRect(0, 0, 400, 300);

// Stripe pattern
const stripePattern = createCustomPattern(ctx, 10, (pCtx, size) => {
  pCtx.fillStyle = 'white';
  pCtx.fillRect(0, 0, size, size);
  
  pCtx.fillStyle = '#e74c3c';
  pCtx.fillRect(0, 0, size / 2, size);
});

ctx.fillStyle = stripePattern;
ctx.fillRect(0, 300, 400, 300);

// Checkerboard pattern
const checkerPattern = createCustomPattern(ctx, 20, (pCtx, size) => {
  const half = size / 2;
  
  pCtx.fillStyle = '#fff';
  pCtx.fillRect(0, 0, size, size);
  
  pCtx.fillStyle = '#ddd';
  pCtx.fillRect(0, 0, half, half);
  pCtx.fillRect(half, half, half, half);
});

ctx.fillStyle = checkerPattern;
ctx.fillRect(400, 0, 400, 600);
```

---

## Step 879: Drawing Text

```javascript
// Text Rendering

const ctx = canvas.getContext('2d');

// ตั้งค่า font
ctx.font = '30px Sarabun';  // size fontFamily
// รูปแบบ: [font-style] [font-weight] font-size font-family
ctx.font = 'bold 30px "Sarabun", sans-serif';
ctx.font = 'italic 20px Arial';
ctx.font = '900 italic 24px Georgia';

// fillText(text, x, y, maxWidth?)
ctx.fillStyle = '#333';
ctx.fillText('สวัสดีโลก!', 100, 100);
ctx.fillText('Hello World!', 100, 140);

// strokeText - กรอบข้อความ
ctx.strokeStyle = '#e74c3c';
ctx.lineWidth = 1;
ctx.strokeText('Outline Text', 100, 200);

// textAlign: 'left', 'right', 'center', 'start', 'end'
const centerX = canvas.width / 2;

ctx.textAlign = 'left';
ctx.fillText('ซ้าย', 200, 250);

ctx.textAlign = 'center';
ctx.fillText('กลาง', centerX, 250);

ctx.textAlign = 'right';
ctx.fillText('ขวา', 600, 250);

// textBaseline: 'top', 'hanging', 'middle', 'alphabetic', 'ideographic', 'bottom'
const y = 350;

['top', 'middle', 'alphabetic', 'bottom'].forEach((baseline, i) => {
  ctx.textBaseline = baseline;
  ctx.fillText(baseline, 50 + i * 150, y);
  
  // เส้นอ้างอิง
  ctx.save();
  ctx.strokeStyle = 'red';
  ctx.lineWidth = 0.5;
  ctx.beginPath();
  ctx.moveTo(0, y);
  ctx.lineTo(canvas.width, y);
  ctx.stroke();
  ctx.restore();
});

// measureText - วัดขนาดข้อความ
ctx.font = '20px Arial';
const textMetrics = ctx.measureText('Hello World');
console.log('Width:', textMetrics.width);
console.log('Height approx:', 20); // font size

// Text wrapping (Canvas ไม่มี built-in word wrap)
function wrapText(ctx, text, x, y, maxWidth, lineHeight) {
  const words = text.split(' ');
  let line = '';
  
  for (let i = 0; i < words.length; i++) {
    const testLine = line + words[i] + ' ';
    const { width } = ctx.measureText(testLine);
    
    if (width > maxWidth && i > 0) {
      ctx.fillText(line, x, y);
      line = words[i] + ' ';
      y += lineHeight;
    } else {
      line = testLine;
    }
  }
  
  ctx.fillText(line, x, y);
  return y + lineHeight;
}

ctx.fillStyle = '#333';
ctx.font = '18px Sarabun';
ctx.textAlign = 'left';
ctx.textBaseline = 'top';

wrapText(
  ctx,
  'JavaScript เป็นภาษาโปรแกรมมิ่งที่ทรงพลังและยืดหยุ่นมาก ใช้ได้ทั้ง frontend และ backend',
  50, 400, 300, 28
);

// Text with shadow
ctx.save();
ctx.shadowColor = 'rgba(0, 0, 0, 0.3)';
ctx.shadowBlur = 5;
ctx.shadowOffsetX = 3;
ctx.shadowOffsetY = 3;
ctx.font = 'bold 40px Sarabun';
ctx.fillStyle = '#2c3e50';
ctx.textAlign = 'center';
ctx.fillText('Canvas API', canvas.width / 2, 80);
ctx.restore();
```

---

## Step 880: Drawing Images

```javascript
// drawImage - วาดรูปภาพบน Canvas

const ctx = canvas.getContext('2d');

// สร้าง Image object
const img = new Image();
img.onload = () => {
  // drawImage(image, dx, dy) - วาดที่ตำแหน่ง x, y
  ctx.drawImage(img, 0, 0);
  
  // drawImage(image, dx, dy, dWidth, dHeight) - ปรับขนาด
  ctx.drawImage(img, 0, 0, 200, 150);
  
  // drawImage(image, sx, sy, sWidth, sHeight, dx, dy, dWidth, dHeight)
  // sx, sy = จุดเริ่มต้นจากรูปต้นฉบับ
  // sWidth, sHeight = ขนาดที่จะตัดจากรูปต้นฉบับ
  // dx, dy = ตำแหน่งที่วาดบน canvas
  // dWidth, dHeight = ขนาดที่จะวาดบน canvas
  ctx.drawImage(img, 50, 50, 100, 100, 300, 200, 200, 200); // crop
};
img.src = 'path/to/image.jpg';

// วาดรูปจาก URL (async)
async function drawImageFromUrl(ctx, url, x, y, width, height) {
  return new Promise((resolve, reject) => {
    const img = new Image();
    img.crossOrigin = 'anonymous'; // สำหรับ images จาก domain อื่น
    
    img.onload = () => {
      if (width && height) {
        ctx.drawImage(img, x, y, width, height);
      } else {
        ctx.drawImage(img, x, y);
      }
      resolve(img);
    };
    
    img.onerror = (e) => reject(new Error('โหลดรูปไม่สำเร็จ'));
    img.src = url;
  });
}

// Image Filter ด้วย CSS
ctx.filter = 'grayscale(100%)';
ctx.drawImage(img, 0, 0);

ctx.filter = 'blur(4px)';
ctx.drawImage(img, 200, 0);

ctx.filter = 'brightness(150%) contrast(120%)';
ctx.drawImage(img, 400, 0);

ctx.filter = 'none'; // reset

// วาดรูปภาพแบบ cover (เต็มพื้นที่)
function drawImageCover(ctx, img, x, y, width, height) {
  const imgAspect = img.width / img.height;
  const canvasAspect = width / height;
  
  let sx, sy, sw, sh;
  
  if (imgAspect > canvasAspect) {
    // รูปกว้างกว่า - crop ด้านข้าง
    sh = img.height;
    sw = img.height * canvasAspect;
    sx = (img.width - sw) / 2;
    sy = 0;
  } else {
    // รูปสูงกว่า - crop ด้านบนล่าง
    sw = img.width;
    sh = img.width / canvasAspect;
    sx = 0;
    sy = (img.height - sh) / 2;
  }
  
  ctx.drawImage(img, sx, sy, sw, sh, x, y, width, height);
}

// วาดรูปภาพแบบ contain (รูปเต็ม ไม่ crop)
function drawImageContain(ctx, img, x, y, width, height) {
  const imgAspect = img.width / img.height;
  const canvasAspect = width / height;
  
  let dw, dh, dx, dy;
  
  if (imgAspect > canvasAspect) {
    dw = width;
    dh = width / imgAspect;
    dx = x;
    dy = y + (height - dh) / 2;
  } else {
    dh = height;
    dw = height * imgAspect;
    dx = x + (width - dw) / 2;
    dy = y;
  }
  
  ctx.drawImage(img, dx, dy, dw, dh);
}

// วาดรูปแบบ circular
function drawCircularImage(ctx, img, cx, cy, radius) {
  ctx.save();
  
  ctx.beginPath();
  ctx.arc(cx, cy, radius, 0, Math.PI * 2);
  ctx.clip(); // clip ให้อยู่ในวงกลม
  
  ctx.drawImage(
    img,
    cx - radius,
    cy - radius,
    radius * 2,
    radius * 2
  );
  
  ctx.restore();
  
  // วาด border
  ctx.beginPath();
  ctx.arc(cx, cy, radius, 0, Math.PI * 2);
  ctx.strokeStyle = 'white';
  ctx.lineWidth = 3;
  ctx.stroke();
}
```

---

## Step 881: Transformations

```javascript
// Transformations

const ctx = canvas.getContext('2d');

// translate(x, y) - เลื่อน origin
ctx.translate(100, 100);
ctx.fillStyle = 'red';
ctx.fillRect(0, 0, 50, 50); // วาดที่ (100,100) จริงๆ

// reset
ctx.translate(-100, -100);

// rotate(angle) - หมุน (radians)
ctx.save(); // บันทึก state
ctx.translate(200, 200); // ย้าย origin ไปที่จุดหมุน
ctx.rotate(Math.PI / 4); // หมุน 45 องศา
ctx.fillRect(-25, -25, 50, 50); // วาดสี่เหลี่ยมที่กึ่งกลางจุดหมุน
ctx.restore(); // คืน state

// scale(x, y) - ขยาย/ย่อ
ctx.save();
ctx.scale(2, 2); // ขยาย 2 เท่าทั้ง x และ y
ctx.fillRect(50, 50, 100, 100); // ดูใหญ่ขึ้น 2 เท่า
ctx.restore();

ctx.save();
ctx.scale(-1, 1); // กลับด้านซ้าย-ขวา
ctx.translate(-canvas.width, 0);
// วาดอะไรก็ตาม จะเป็น mirror
ctx.restore();

// transform(a, b, c, d, e, f) - matrix transform
// [a c e]   [scale-x  skew-x  translate-x]
// [b d f] = [skew-y   scale-y translate-y]
// [0 0 1]   [0        0       1           ]

ctx.save();
ctx.transform(1, 0.5, -0.5, 1, 200, 100); // skew
ctx.fillStyle = 'blue';
ctx.fillRect(0, 0, 100, 100);
ctx.restore();

// setTransform - reset dan apply transform
ctx.setTransform(1, 0, 0, 1, 0, 0); // identity matrix = reset

// การประยุกต์: Clock
function drawClock(ctx, cx, cy, radius) {
  ctx.save();
  ctx.translate(cx, cy);
  
  // วงกลม
  ctx.beginPath();
  ctx.arc(0, 0, radius, 0, Math.PI * 2);
  ctx.fillStyle = '#fff';
  ctx.fill();
  ctx.strokeStyle = '#333';
  ctx.lineWidth = 3;
  ctx.stroke();
  
  // ขีดบอกชั่วโมง
  for (let i = 0; i < 12; i++) {
    const angle = (i / 12) * Math.PI * 2 - Math.PI / 2;
    const isMainHour = i % 3 === 0;
    const len = isMainHour ? 10 : 6;
    
    ctx.save();
    ctx.rotate(angle);
    ctx.strokeStyle = '#333';
    ctx.lineWidth = isMainHour ? 3 : 1;
    ctx.beginPath();
    ctx.moveTo(0, -(radius - len));
    ctx.lineTo(0, -radius);
    ctx.stroke();
    ctx.restore();
  }
  
  const now = new Date();
  const hours = now.getHours() % 12 + now.getMinutes() / 60;
  const minutes = now.getMinutes() + now.getSeconds() / 60;
  const seconds = now.getSeconds();
  
  // เข็มชั่วโมง
  ctx.save();
  ctx.rotate((hours / 12) * Math.PI * 2 - Math.PI / 2);
  ctx.strokeStyle = '#333';
  ctx.lineWidth = 5;
  ctx.lineCap = 'round';
  ctx.beginPath();
  ctx.moveTo(-10, 0);
  ctx.lineTo(radius * 0.5, 0);
  ctx.stroke();
  ctx.restore();
  
  // เข็มนาที
  ctx.save();
  ctx.rotate((minutes / 60) * Math.PI * 2 - Math.PI / 2);
  ctx.strokeStyle = '#555';
  ctx.lineWidth = 3;
  ctx.lineCap = 'round';
  ctx.beginPath();
  ctx.moveTo(-12, 0);
  ctx.lineTo(radius * 0.75, 0);
  ctx.stroke();
  ctx.restore();
  
  // เข็มวินาที
  ctx.save();
  ctx.rotate((seconds / 60) * Math.PI * 2 - Math.PI / 2);
  ctx.strokeStyle = 'red';
  ctx.lineWidth = 1.5;
  ctx.lineCap = 'round';
  ctx.beginPath();
  ctx.moveTo(-15, 0);
  ctx.lineTo(radius * 0.85, 0);
  ctx.stroke();
  ctx.restore();
  
  // จุดกลาง
  ctx.beginPath();
  ctx.arc(0, 0, 4, 0, Math.PI * 2);
  ctx.fillStyle = 'red';
  ctx.fill();
  
  ctx.restore();
}
```

---

## Step 882: Save and Restore State

```javascript
// save() และ restore() - จัดการ state

const ctx = canvas.getContext('2d');

// State ที่ถูก save/restore:
// - fillStyle, strokeStyle
// - lineWidth, lineCap, lineJoin
// - font, textAlign, textBaseline
// - globalAlpha, globalCompositeOperation
// - shadowColor, shadowBlur, shadowOffset
// - transform matrix
// - clip region

// ตัวอย่างการใช้งาน
ctx.fillStyle = 'blue';
ctx.font = '20px Arial';

ctx.save(); // บันทึก state 1

  ctx.fillStyle = 'red';  // เปลี่ยน style
  ctx.font = '30px Times';
  ctx.fillText('Red', 100, 100);
  
  ctx.save(); // บันทึก state 2
  
    ctx.fillStyle = 'green';
    ctx.font = '40px Courier';
    ctx.fillText('Green', 100, 150);
  
  ctx.restore(); // คืน state 2 (= state 1)
  
  ctx.fillText('Back to Red', 100, 200);

ctx.restore(); // คืน state 1

ctx.fillText('Back to Blue', 100, 250); // blue, 20px Arial

// Pattern สำหรับ reusable components
function drawButton(ctx, x, y, width, height, text, style = {}) {
  ctx.save(); // save state ก่อน
  
  const {
    background = '#3498db',
    textColor = 'white',
    borderRadius = 8,
    fontSize = 16,
    font = 'Sarabun'
  } = style;
  
  // Background
  ctx.fillStyle = background;
  ctx.beginPath();
  ctx.roundRect(x, y, width, height, borderRadius);
  ctx.fill();
  
  // Shadow
  ctx.shadowColor = 'rgba(0, 0, 0, 0.2)';
  ctx.shadowBlur = 5;
  ctx.shadowOffsetY = 2;
  ctx.fill();
  
  // Reset shadow
  ctx.shadowColor = 'transparent';
  
  // Text
  ctx.fillStyle = textColor;
  ctx.font = `bold ${fontSize}px ${font}`;
  ctx.textAlign = 'center';
  ctx.textBaseline = 'middle';
  ctx.fillText(text, x + width / 2, y + height / 2);
  
  ctx.restore(); // restore state หลัง
}

drawButton(ctx, 50, 50, 150, 50, 'ยืนยัน', { background: '#27ae60' });
drawButton(ctx, 220, 50, 150, 50, 'ยกเลิก', { background: '#e74c3c' });
drawButton(ctx, 390, 50, 150, 50, 'ดูข้อมูล', { background: '#3498db' });
```

---

## Step 883: Clipping

```javascript
// Clipping - ตัดพื้นที่การวาด

const ctx = canvas.getContext('2d');

// Circular clip
ctx.save();

// กำหนด clip region
ctx.beginPath();
ctx.arc(200, 200, 100, 0, Math.PI * 2);
ctx.clip();

// วาดรูปภาพ - จะถูก clip เป็นวงกลม
const img = new Image();
img.onload = () => {
  ctx.drawImage(img, 100, 100, 200, 200);
  ctx.restore(); // restore clip
};
img.src = 'photo.jpg';

// Text clip
ctx.save();

ctx.font = 'bold 80px Arial';
ctx.textAlign = 'center';
ctx.textBaseline = 'middle';
ctx.fillText('CLIP', canvas.width / 2, canvas.height / 2);
ctx.clip(); // clip ตาม text shape (ไม่ค่อย work ใน practice)

ctx.restore();

// Polygon clip
function clipToPolygon(ctx, points) {
  ctx.beginPath();
  ctx.moveTo(points[0].x, points[0].y);
  
  for (let i = 1; i < points.length; i++) {
    ctx.lineTo(points[i].x, points[i].y);
  }
  
  ctx.closePath();
  ctx.clip();
}

ctx.save();
clipToPolygon(ctx, [
  { x: 200, y: 100 },
  { x: 350, y: 100 },
  { x: 400, y: 250 },
  { x: 275, y: 350 },
  { x: 150, y: 250 }
]);

// วาดอะไรก็ตาม จะถูก clip
ctx.fillStyle = '#3498db';
ctx.fillRect(0, 0, canvas.width, canvas.height);

ctx.restore();
```

---

## Step 884: Pixel Manipulation

```javascript
// ImageData - จัดการ pixels โดยตรง

const ctx = canvas.getContext('2d');

// getImageData(x, y, width, height)
// คืน ImageData object ที่มี data array ของ RGBA values
const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);

// imageData.data = Uint8ClampedArray
// ทุก 4 elements = 1 pixel [R, G, B, A]
// R, G, B, A แต่ละค่า 0-255

console.log('Width:', imageData.width);
console.log('Height:', imageData.height);
console.log('Data length:', imageData.data.length); // width * height * 4

// Helper: แปลง x,y เป็น index
function getPixelIndex(imageData, x, y) {
  return (y * imageData.width + x) * 4;
}

// อ่าน pixel สีที่ตำแหน่ง x, y
function getPixel(imageData, x, y) {
  const index = getPixelIndex(imageData, x, y);
  return {
    r: imageData.data[index],
    g: imageData.data[index + 1],
    b: imageData.data[index + 2],
    a: imageData.data[index + 3]
  };
}

// เขียน pixel
function setPixel(imageData, x, y, r, g, b, a = 255) {
  const index = getPixelIndex(imageData, x, y);
  imageData.data[index] = r;
  imageData.data[index + 1] = g;
  imageData.data[index + 2] = b;
  imageData.data[index + 3] = a;
}

// Image Filters

// Grayscale filter
function applyGrayscale(imageData) {
  const data = imageData.data;
  
  for (let i = 0; i < data.length; i += 4) {
    const r = data[i];
    const g = data[i + 1];
    const b = data[i + 2];
    
    // Luminance formula
    const gray = 0.2126 * r + 0.7152 * g + 0.0722 * b;
    
    data[i] = data[i + 1] = data[i + 2] = gray;
  }
  
  return imageData;
}

// Invert filter
function applyInvert(imageData) {
  const data = imageData.data;
  
  for (let i = 0; i < data.length; i += 4) {
    data[i] = 255 - data[i];
    data[i + 1] = 255 - data[i + 1];
    data[i + 2] = 255 - data[i + 2];
  }
  
  return imageData;
}

// Brightness filter
function applyBrightness(imageData, factor) {
  const data = imageData.data;
  
  for (let i = 0; i < data.length; i += 4) {
    data[i] = Math.min(255, data[i] * factor);
    data[i + 1] = Math.min(255, data[i + 1] * factor);
    data[i + 2] = Math.min(255, data[i + 2] * factor);
  }
  
  return imageData;
}

// Blur filter (Box blur)
function applyBlur(imageData, radius) {
  const { width, height, data } = imageData;
  const newData = new Uint8ClampedArray(data);
  
  for (let y = 0; y < height; y++) {
    for (let x = 0; x < width; x++) {
      let r = 0, g = 0, b = 0, count = 0;
      
      for (let dy = -radius; dy <= radius; dy++) {
        for (let dx = -radius; dx <= radius; dx++) {
          const nx = x + dx;
          const ny = y + dy;
          
          if (nx >= 0 && nx < width && ny >= 0 && ny < height) {
            const i = (ny * width + nx) * 4;
            r += data[i];
            g += data[i + 1];
            b += data[i + 2];
            count++;
          }
        }
      }
      
      const idx = (y * width + x) * 4;
      newData[idx] = r / count;
      newData[idx + 1] = g / count;
      newData[idx + 2] = b / count;
      newData[idx + 3] = data[idx + 3];
    }
  }
  
  imageData.data.set(newData);
  return imageData;
}

// Pixel color picker
canvas.addEventListener('click', (e) => {
  const rect = canvas.getBoundingClientRect();
  const x = Math.floor((e.clientX - rect.left) * (canvas.width / rect.width));
  const y = Math.floor((e.clientY - rect.top) * (canvas.height / rect.height));
  
  const imageData = ctx.getImageData(x, y, 1, 1);
  const pixel = getPixel(imageData, 0, 0);
  
  console.log(`Pixel at (${x}, ${y}): R=${pixel.r} G=${pixel.g} B=${pixel.b} A=${pixel.a}`);
  console.log(`Hex: #${pixel.r.toString(16).padStart(2,'0')}${pixel.g.toString(16).padStart(2,'0')}${pixel.b.toString(16).padStart(2,'0')}`);
});

// putImageData - วาง imageData กลับ
// ctx.putImageData(imageData, x, y);
// ctx.putImageData(imageData, x, y, dirtyX, dirtyY, dirtyWidth, dirtyHeight);

// ตัวอย่าง: วาดรูปและ apply filter
const imgForFilter = new Image();
imgForFilter.onload = () => {
  // วาดรูปต้นฉบับ
  ctx.drawImage(imgForFilter, 0, 0, 300, 200);
  
  // ดึง imageData และ apply filter
  const data = ctx.getImageData(0, 0, 300, 200);
  applyGrayscale(data);
  
  // วาง imageData ที่ filter แล้ว
  ctx.putImageData(data, 310, 0);
};
imgForFilter.src = 'test.jpg';
```

---

## Step 885: Canvas Animation

```javascript
// Animation ด้วย requestAnimationFrame

class CanvasAnimator {
  constructor(canvas) {
    this.canvas = canvas;
    this.ctx = canvas.getContext('2d');
    this.animationId = null;
    this.fps = 0;
    this.lastTime = 0;
    this.objects = [];
  }
  
  addObject(obj) {
    this.objects.push(obj);
    return this;
  }
  
  start() {
    if (this.animationId) return;
    this.animate(0);
  }
  
  stop() {
    if (this.animationId) {
      cancelAnimationFrame(this.animationId);
      this.animationId = null;
    }
  }
  
  animate(timestamp) {
    // คำนวณ delta time
    const deltaTime = timestamp - this.lastTime;
    this.lastTime = timestamp;
    
    // คำนวณ FPS
    this.fps = Math.round(1000 / deltaTime) || 0;
    
    // ล้าง canvas
    this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);
    
    // อัพเดตและวาด objects
    this.objects.forEach(obj => {
      obj.update?.(deltaTime / 1000, this.canvas); // delta in seconds
      obj.draw?.(this.ctx);
    });
    
    // แสดง FPS
    this.ctx.fillStyle = 'black';
    this.ctx.font = '14px monospace';
    this.ctx.fillText(`FPS: ${this.fps}`, 10, 20);
    
    this.animationId = requestAnimationFrame((ts) => this.animate(ts));
  }
}

// Animated Ball
class Ball {
  constructor(canvas) {
    this.reset(canvas);
  }
  
  reset(canvas) {
    this.x = Math.random() * canvas.width;
    this.y = Math.random() * canvas.height;
    this.vx = (Math.random() - 0.5) * 400; // pixels per second
    this.vy = (Math.random() - 0.5) * 400;
    this.radius = 10 + Math.random() * 20;
    this.color = `hsl(${Math.random() * 360}, 70%, 60%)`;
  }
  
  update(delta, canvas) {
    this.x += this.vx * delta;
    this.y += this.vy * delta;
    
    // Bounce off walls
    if (this.x - this.radius < 0) {
      this.x = this.radius;
      this.vx = Math.abs(this.vx);
    }
    if (this.x + this.radius > canvas.width) {
      this.x = canvas.width - this.radius;
      this.vx = -Math.abs(this.vx);
    }
    if (this.y - this.radius < 0) {
      this.y = this.radius;
      this.vy = Math.abs(this.vy);
    }
    if (this.y + this.radius > canvas.height) {
      this.y = canvas.height - this.radius;
      this.vy = -Math.abs(this.vy);
    }
  }
  
  draw(ctx) {
    ctx.save();
    ctx.beginPath();
    ctx.arc(this.x, this.y, this.radius, 0, Math.PI * 2);
    ctx.fillStyle = this.color;
    ctx.fill();
    ctx.restore();
  }
}

// Particle System
class Particle {
  constructor(x, y) {
    this.x = x;
    this.y = y;
    this.vx = (Math.random() - 0.5) * 200;
    this.vy = -100 - Math.random() * 200;
    this.life = 1.0;
    this.decay = 0.5 + Math.random() * 0.5; // per second
    this.size = 3 + Math.random() * 5;
    this.color = `hsl(${Math.random() * 60 + 10}, 100%, 60%)`;
  }
  
  update(delta) {
    this.x += this.vx * delta;
    this.y += this.vy * delta;
    this.vy += 200 * delta; // gravity
    this.life -= this.decay * delta;
  }
  
  draw(ctx) {
    if (this.life <= 0) return;
    
    ctx.save();
    ctx.globalAlpha = this.life;
    ctx.beginPath();
    ctx.arc(this.x, this.y, this.size * this.life, 0, Math.PI * 2);
    ctx.fillStyle = this.color;
    ctx.fill();
    ctx.restore();
  }
  
  isDead() {
    return this.life <= 0;
  }
}

class ParticleSystem {
  constructor() {
    this.particles = [];
  }
  
  emit(x, y, count = 20) {
    for (let i = 0; i < count; i++) {
      this.particles.push(new Particle(x, y));
    }
  }
  
  update(delta) {
    this.particles.forEach(p => p.update(delta));
    this.particles = this.particles.filter(p => !p.isDead());
  }
  
  draw(ctx) {
    this.particles.forEach(p => p.draw(ctx));
  }
}

// ใช้งาน
const animator = new CanvasAnimator(canvas);
const particles = new ParticleSystem();

// เพิ่ม balls
for (let i = 0; i < 10; i++) {
  animator.addObject(new Ball(canvas));
}

// เพิ่ม particle system
animator.addObject({
  update: (delta) => particles.update(delta),
  draw: (ctx) => particles.draw(ctx)
});

// คลิกเพื่อสร้าง particles
canvas.addEventListener('click', (e) => {
  const rect = canvas.getBoundingClientRect();
  const x = e.clientX - rect.left;
  const y = e.clientY - rect.top;
  particles.emit(x, y, 30);
});

animator.start();
```

---

## Step 886-890: Drawing Application

```javascript
// Mini Drawing Application

class DrawingApp {
  constructor(canvas) {
    this.canvas = canvas;
    this.ctx = canvas.getContext('2d');
    this.isDrawing = false;
    this.tool = 'pen';
    this.color = '#000000';
    this.lineWidth = 3;
    this.lastX = 0;
    this.lastY = 0;
    this.startX = 0;
    this.startY = 0;
    this.history = [];
    this.historyIndex = -1;
    
    this.#setupEventListeners();
    this.#saveState(); // บันทึก state เริ่มต้น
  }
  
  #getPos(e) {
    const rect = this.canvas.getBoundingClientRect();
    const clientX = e.touches ? e.touches[0].clientX : e.clientX;
    const clientY = e.touches ? e.touches[0].clientY : e.clientY;
    
    return {
      x: (clientX - rect.left) * (this.canvas.width / rect.width),
      y: (clientY - rect.top) * (this.canvas.height / rect.height)
    };
  }
  
  #setupEventListeners() {
    const canvas = this.canvas;
    
    // Mouse events
    canvas.addEventListener('mousedown', (e) => this.#startDrawing(e));
    canvas.addEventListener('mousemove', (e) => this.#draw(e));
    canvas.addEventListener('mouseup', () => this.#stopDrawing());
    canvas.addEventListener('mouseout', () => this.#stopDrawing());
    
    // Touch events
    canvas.addEventListener('touchstart', (e) => {
      e.preventDefault();
      this.#startDrawing(e);
    });
    canvas.addEventListener('touchmove', (e) => {
      e.preventDefault();
      this.#draw(e);
    });
    canvas.addEventListener('touchend', () => this.#stopDrawing());
  }
  
  #startDrawing(e) {
    this.isDrawing = true;
    const { x, y } = this.#getPos(e);
    
    this.lastX = x;
    this.lastY = y;
    this.startX = x;
    this.startY = y;
    
    if (this.tool === 'fill') {
      this.#floodFill(x, y);
    }
  }
  
  #draw(e) {
    if (!this.isDrawing) return;
    
    const { x, y } = this.#getPos(e);
    const ctx = this.ctx;
    
    ctx.strokeStyle = this.color;
    ctx.lineWidth = this.lineWidth;
    ctx.lineCap = 'round';
    ctx.lineJoin = 'round';
    
    switch (this.tool) {
      case 'pen':
        ctx.beginPath();
        ctx.moveTo(this.lastX, this.lastY);
        ctx.lineTo(x, y);
        ctx.stroke();
        break;
        
      case 'eraser':
        ctx.save();
        ctx.globalCompositeOperation = 'destination-out';
        ctx.beginPath();
        ctx.arc(x, y, this.lineWidth * 5, 0, Math.PI * 2);
        ctx.fill();
        ctx.restore();
        break;
        
      case 'rect':
        // วาด preview - ต้องลบ preview เก่าก่อน
        const savedImage = this.history[this.historyIndex];
        if (savedImage) {
          ctx.putImageData(savedImage, 0, 0);
        }
        ctx.beginPath();
        ctx.strokeRect(
          this.startX, this.startY,
          x - this.startX, y - this.startY
        );
        break;
        
      case 'circle':
        const savedImg = this.history[this.historyIndex];
        if (savedImg) ctx.putImageData(savedImg, 0, 0);
        
        const radius = Math.sqrt(
          Math.pow(x - this.startX, 2) + 
          Math.pow(y - this.startY, 2)
        );
        
        ctx.beginPath();
        ctx.arc(this.startX, this.startY, radius, 0, Math.PI * 2);
        ctx.stroke();
        break;
        
      case 'line':
        const savedL = this.history[this.historyIndex];
        if (savedL) ctx.putImageData(savedL, 0, 0);
        
        ctx.beginPath();
        ctx.moveTo(this.startX, this.startY);
        ctx.lineTo(x, y);
        ctx.stroke();
        break;
    }
    
    this.lastX = x;
    this.lastY = y;
  }
  
  #stopDrawing() {
    if (this.isDrawing) {
      this.isDrawing = false;
      this.#saveState();
    }
  }
  
  #saveState() {
    // ลบ redo history
    this.history.splice(this.historyIndex + 1);
    
    // บันทึก state ปัจจุบัน
    const imageData = this.ctx.getImageData(0, 0, this.canvas.width, this.canvas.height);
    this.history.push(imageData);
    this.historyIndex++;
    
    // จำกัด history ที่ 50
    if (this.history.length > 50) {
      this.history.shift();
      this.historyIndex--;
    }
  }
  
  // Flood fill algorithm
  #floodFill(startX, startY) {
    startX = Math.floor(startX);
    startY = Math.floor(startY);
    
    const imageData = this.ctx.getImageData(0, 0, this.canvas.width, this.canvas.height);
    const targetColor = this.#getPixelColor(imageData, startX, startY);
    const fillColor = this.#hexToRgba(this.color);
    
    if (this.#colorMatch(targetColor, fillColor)) return;
    
    const stack = [[startX, startY]];
    
    while (stack.length > 0) {
      const [x, y] = stack.pop();
      
      if (x < 0 || x >= this.canvas.width || y < 0 || y >= this.canvas.height) continue;
      
      const current = this.#getPixelColor(imageData, x, y);
      if (!this.#colorMatch(current, targetColor)) continue;
      
      this.#setPixelColor(imageData, x, y, fillColor);
      
      stack.push([x + 1, y], [x - 1, y], [x, y + 1], [x, y - 1]);
    }
    
    this.ctx.putImageData(imageData, 0, 0);
  }
  
  #getPixelColor(imageData, x, y) {
    const index = (y * imageData.width + x) * 4;
    return [
      imageData.data[index],
      imageData.data[index + 1],
      imageData.data[index + 2],
      imageData.data[index + 3]
    ];
  }
  
  #setPixelColor(imageData, x, y, [r, g, b, a]) {
    const index = (y * imageData.width + x) * 4;
    imageData.data[index] = r;
    imageData.data[index + 1] = g;
    imageData.data[index + 2] = b;
    imageData.data[index + 3] = a;
  }
  
  #colorMatch(c1, c2, tolerance = 30) {
    return Math.abs(c1[0] - c2[0]) <= tolerance &&
           Math.abs(c1[1] - c2[1]) <= tolerance &&
           Math.abs(c1[2] - c2[2]) <= tolerance;
  }
  
  #hexToRgba(hex) {
    const result = /^#?([a-f\d]{2})([a-f\d]{2})([a-f\d]{2})$/i.exec(hex);
    return result ? [
      parseInt(result[1], 16),
      parseInt(result[2], 16),
      parseInt(result[3], 16),
      255
    ] : [0, 0, 0, 255];
  }
  
  // Public methods
  setTool(tool) {
    this.tool = tool;
  }
  
  setColor(color) {
    this.color = color;
  }
  
  setLineWidth(width) {
    this.lineWidth = width;
  }
  
  undo() {
    if (this.historyIndex > 0) {
      this.historyIndex--;
      this.ctx.putImageData(this.history[this.historyIndex], 0, 0);
    }
  }
  
  redo() {
    if (this.historyIndex < this.history.length - 1) {
      this.historyIndex++;
      this.ctx.putImageData(this.history[this.historyIndex], 0, 0);
    }
  }
  
  clear() {
    this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);
    this.#saveState();
  }
  
  // Export as image
  toDataURL(type = 'image/png', quality = 0.92) {
    return this.canvas.toDataURL(type, quality);
  }
  
  download(filename = 'drawing.png') {
    const link = document.createElement('a');
    link.download = filename;
    link.href = this.toDataURL();
    link.click();
  }
  
  // ได้ blob สำหรับ upload
  toBlob(callback, type = 'image/png') {
    this.canvas.toBlob(callback, type);
  }
}

// ใช้งาน Drawing App
const drawingCanvas = document.getElementById('drawing-canvas');
const app = new DrawingApp(drawingCanvas);

// Toolbar
document.getElementById('tool-pen')?.addEventListener('click', () => app.setTool('pen'));
document.getElementById('tool-eraser')?.addEventListener('click', () => app.setTool('eraser'));
document.getElementById('tool-rect')?.addEventListener('click', () => app.setTool('rect'));
document.getElementById('tool-circle')?.addEventListener('click', () => app.setTool('circle'));
document.getElementById('tool-line')?.addEventListener('click', () => app.setTool('line'));
document.getElementById('tool-fill')?.addEventListener('click', () => app.setTool('fill'));

document.getElementById('color-picker')?.addEventListener('change', (e) => {
  app.setColor(e.target.value);
});

document.getElementById('line-width')?.addEventListener('input', (e) => {
  app.setLineWidth(parseInt(e.target.value));
});

document.getElementById('btn-undo')?.addEventListener('click', () => app.undo());
document.getElementById('btn-redo')?.addEventListener('click', () => app.redo());
document.getElementById('btn-clear')?.addEventListener('click', () => app.clear());
document.getElementById('btn-download')?.addEventListener('click', () => app.download());

// Keyboard shortcuts
document.addEventListener('keydown', (e) => {
  if (e.ctrlKey || e.metaKey) {
    if (e.key === 'z') {
      e.preventDefault();
      app.undo();
    }
    if (e.key === 'y') {
      e.preventDefault();
      app.redo();
    }
    if (e.key === 's') {
      e.preventDefault();
      app.download('my-drawing.png');
    }
  }
});

console.log('Drawing App พร้อมใช้งาน!');
console.log('เครื่องมือ: pen, eraser, rect, circle, line, fill');
console.log('Shortcuts: Ctrl+Z (Undo), Ctrl+Y (Redo), Ctrl+S (Save)');
```

---

## แบบฝึกหัด (Exercises)

### แบบฝึกหัดที่ 1: Bar Chart Component

```javascript
// TODO: สร้าง BarChart class ที่:
// 1. รับข้อมูลในรูป array of { label, value, color }
// 2. วาด bar chart ที่มี animation
// 3. มี tooltip เมื่อ hover
// 4. Support horizontal และ vertical bar
// 5. มี legend

class BarChart {
  constructor(canvas, data, options = {}) {
    // TODO
  }
  
  draw() {
    // TODO
  }
  
  animate() {
    // TODO: bars เติบโตจาก 0 ขึ้นมา
  }
}
```

### แบบฝึกหัดที่ 2: Simple Game

```javascript
// TODO: สร้าง Snake Game โดยใช้ Canvas:
// 1. Snake เดินไปเรื่อยๆ
// 2. ควบคุมด้วย arrow keys
// 3. กิน food เพื่อเพิ่มความยาว
// 4. Game over เมื่อชนขอบหรือชนตัวเอง
// 5. แสดง score

class SnakeGame {
  // TODO: implement
}
```

### แบบฝึกหัดที่ 3: Image Editor

```javascript
// TODO: เพิ่ม filters ให้กับ Drawing App:
// 1. Brightness / Contrast slider
// 2. Saturation filter
// 3. Blur effect
// 4. Pixelate effect
// 5. Apply filter preview แบบ real-time

class ImageFilters {
  // TODO
  static brightness(imageData, level) {}
  static contrast(imageData, level) {}
  static saturation(imageData, level) {}
  static blur(imageData, radius) {}
  static pixelate(imageData, size) {}
}
```

---

## สรุป

Canvas API เป็นเครื่องมือที่ทรงพลังสำหรับ graphics programming บนเว็บ:

1. **Rectangles**: `fillRect`, `strokeRect`, `clearRect`
2. **Paths**: `beginPath`, `moveTo`, `lineTo`, `arc`, `bezierCurveTo`
3. **Styles**: colors, gradients, patterns, line styles
4. **Text**: `fillText`, `strokeText`, font properties
5. **Images**: `drawImage` ด้วย various options
6. **Transformations**: translate, rotate, scale, matrix
7. **State**: `save()` / `restore()` สำหรับ nested state
8. **Pixels**: `getImageData` / `putImageData` สำหรับ effects
9. **Animation**: `requestAnimationFrame` สำหรับ smooth animation

Canvas เหมาะสำหรับ:
- Data visualization (charts, graphs)
- Image processing
- Games
- Drawing applications
- Generative art

สำหรับ 3D graphics ให้ดู WebGL หรือ Three.js ที่ใช้ canvas ด้วยเช่นกัน

---

*จบ Part 45: Canvas API*
