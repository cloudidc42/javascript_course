# Part 96: Game Development ด้วย JavaScript (Steps 1891-1910)

## บทนำ

การพัฒนาเกมด้วย JavaScript ได้กลายเป็นหนึ่งในสาขาที่น่าตื่นเต้นที่สุดในวงการพัฒนาเว็บ ด้วย HTML5 Canvas, WebGL, และ framework อย่าง Phaser.js และ Three.js ทำให้เราสามารถสร้างเกมที่ซับซ้อนและสวยงามได้โดยตรงในเบราว์เซอร์ บทนี้จะพาคุณตั้งแต่พื้นฐานการพัฒนาเกมไปจนถึงการสร้างเกม 3D ด้วย WebGL

---

## Step 1891: Game Development Basics

### แนวคิดพื้นฐานของการพัฒนาเกม

เกมทุกประเภทมีโครงสร้างพื้นฐานเหมือนกัน:

1. **Game Loop** - วงจรหลักที่ทำงานซ้ำๆ
2. **Update** - อัปเดต state ของเกม
3. **Render** - วาดภาพบนหน้าจอ
4. **Input** - รับ input จากผู้เล่น
5. **Collision Detection** - ตรวจสอบการชน
6. **Game State** - จัดการ state ของเกม (menu, playing, game over)

```javascript
// โครงสร้างพื้นฐานของเกม
class Game {
  constructor(canvas) {
    this.canvas = canvas;
    this.ctx = canvas.getContext('2d');
    this.lastTime = 0;
    this.isRunning = false;
    this.entities = [];
    this.score = 0;
    this.gameState = 'menu'; // menu, playing, paused, gameOver
  }

  start() {
    this.isRunning = true;
    this.gameState = 'playing';
    requestAnimationFrame(this.gameLoop.bind(this));
  }

  stop() {
    this.isRunning = false;
  }

  gameLoop(timestamp) {
    if (!this.isRunning) return;

    // คำนวณ delta time (เวลาที่ผ่านไปในแต่ละ frame)
    const deltaTime = (timestamp - this.lastTime) / 1000; // แปลงเป็นวินาที
    this.lastTime = timestamp;

    // อัปเดตและวาด
    this.update(deltaTime);
    this.render();

    // วน loop ต่อ
    requestAnimationFrame(this.gameLoop.bind(this));
  }

  update(deltaTime) {
    // อัปเดต entities ทั้งหมด
    this.entities.forEach(entity => {
      entity.update(deltaTime);
    });

    // ตรวจสอบ collision
    this.checkCollisions();

    // ลบ entities ที่ไม่ใช้แล้ว
    this.entities = this.entities.filter(entity => entity.active);
  }

  render() {
    // ล้างหน้าจอ
    this.ctx.clearRect(0, 0, this.canvas.width, this.canvas.height);

    // วาด background
    this.renderBackground();

    // วาด entities
    this.entities.forEach(entity => {
      entity.render(this.ctx);
    });

    // วาด UI
    this.renderUI();
  }

  renderBackground() {
    this.ctx.fillStyle = '#1a1a2e';
    this.ctx.fillRect(0, 0, this.canvas.width, this.canvas.height);
  }

  renderUI() {
    this.ctx.fillStyle = 'white';
    this.ctx.font = '20px Arial';
    this.ctx.fillText(`Score: ${this.score}`, 10, 30);
  }

  checkCollisions() {
    // ตรวจสอบ collision ระหว่าง entities
    for (let i = 0; i < this.entities.length; i++) {
      for (let j = i + 1; j < this.entities.length; j++) {
        if (this.entities[i].collidesWith(this.entities[j])) {
          this.entities[i].onCollision(this.entities[j]);
          this.entities[j].onCollision(this.entities[i]);
        }
      }
    }
  }
}

// Entity พื้นฐาน
class Entity {
  constructor(x, y, width, height) {
    this.x = x;
    this.y = y;
    this.width = width;
    this.height = height;
    this.velX = 0;
    this.velY = 0;
    this.active = true;
  }

  update(deltaTime) {
    this.x += this.velX * deltaTime;
    this.y += this.velY * deltaTime;
  }

  render(ctx) {
    ctx.fillStyle = 'white';
    ctx.fillRect(this.x, this.y, this.width, this.height);
  }

  // AABB Collision Detection
  collidesWith(other) {
    return (
      this.x < other.x + other.width &&
      this.x + this.width > other.x &&
      this.y < other.y + other.height &&
      this.y + this.height > other.y
    );
  }

  onCollision(other) {
    // Override ใน subclass
  }
}
```

---

## Step 1892: Game Loop และ Delta Time

### ทำไมต้องใช้ Delta Time?

Delta time คือเวลาที่ผ่านไประหว่าง frame สองอัน การใช้ delta time ทำให้เกมทำงานในความเร็วเดียวกันบนทุกเครื่อง

```javascript
// Game Loop ที่ดี
class GameLoop {
  constructor() {
    this.fps = 60;
    this.targetDelta = 1000 / this.fps;
    this.lastTime = 0;
    this.accumulator = 0;
    this.fixedDelta = 1 / this.fps;
  }

  // Fixed Timestep Game Loop (สำหรับ physics ที่แม่นยำ)
  fixedUpdate(timestamp) {
    const delta = timestamp - this.lastTime;
    this.lastTime = timestamp;

    this.accumulator += delta / 1000;

    // อัปเดต physics ด้วย fixed timestep
    while (this.accumulator >= this.fixedDelta) {
      this.physics.update(this.fixedDelta);
      this.accumulator -= this.fixedDelta;
    }

    // Interpolate สำหรับการ render ที่ smooth
    const alpha = this.accumulator / this.fixedDelta;
    this.render(alpha);

    requestAnimationFrame(this.fixedUpdate.bind(this));
  }
}

// FPS Counter
class FPSCounter {
  constructor() {
    this.frames = 0;
    this.lastTime = performance.now();
    this.fps = 0;
    this.sampleSize = 60;
    this.samples = [];
  }

  update() {
    const now = performance.now();
    const delta = now - this.lastTime;
    this.lastTime = now;

    this.samples.push(delta);
    if (this.samples.length > this.sampleSize) {
      this.samples.shift();
    }

    const average = this.samples.reduce((a, b) => a + b, 0) / this.samples.length;
    this.fps = Math.round(1000 / average);
  }

  render(ctx) {
    ctx.fillStyle = 'yellow';
    ctx.font = '14px monospace';
    ctx.fillText(`FPS: ${this.fps}`, 10, 50);
  }
}

// ตัวอย่างการใช้งาน
const canvas = document.getElementById('gameCanvas');
const ctx = canvas.getContext('2d');
const fpsCounter = new FPSCounter();

let lastTime = 0;

function gameLoop(timestamp) {
  const deltaTime = Math.min((timestamp - lastTime) / 1000, 0.05); // cap ที่ 50ms
  lastTime = timestamp;

  fpsCounter.update();

  // Update
  update(deltaTime);

  // Render
  render();
  fpsCounter.render(ctx);

  requestAnimationFrame(gameLoop);
}

requestAnimationFrame(gameLoop);
```

---

## Step 1893: Canvas API สำหรับ Game Development

### Canvas 2D Context

```javascript
// Canvas utilities สำหรับเกม
class CanvasRenderer {
  constructor(canvas) {
    this.canvas = canvas;
    this.ctx = canvas.getContext('2d');
    this.width = canvas.width;
    this.height = canvas.height;
  }

  // วาด sprite จาก spritesheet
  drawSprite(image, srcX, srcY, srcW, srcH, destX, destY, destW, destH) {
    this.ctx.drawImage(image, srcX, srcY, srcW, srcH, destX, destY, destW, destH);
  }

  // วาด sprite ที่หมุน
  drawRotatedSprite(image, x, y, width, height, angle) {
    this.ctx.save();
    this.ctx.translate(x + width / 2, y + height / 2);
    this.ctx.rotate(angle);
    this.ctx.drawImage(image, -width / 2, -height / 2, width, height);
    this.ctx.restore();
  }

  // วาดข้อความพร้อม shadow
  drawText(text, x, y, options = {}) {
    const {
      color = 'white',
      font = '20px Arial',
      align = 'left',
      shadow = true,
      shadowColor = 'rgba(0,0,0,0.5)',
      shadowBlur = 3
    } = options;

    this.ctx.font = font;
    this.ctx.textAlign = align;

    if (shadow) {
      this.ctx.shadowColor = shadowColor;
      this.ctx.shadowBlur = shadowBlur;
    }

    this.ctx.fillStyle = color;
    this.ctx.fillText(text, x, y);
    this.ctx.shadowBlur = 0;
  }

  // วาด progress bar
  drawProgressBar(x, y, width, height, value, maxValue, options = {}) {
    const {
      bgColor = '#333',
      fillColor = '#0f0',
      borderColor = 'white',
      borderWidth = 2
    } = options;

    // Background
    this.ctx.fillStyle = bgColor;
    this.ctx.fillRect(x, y, width, height);

    // Fill
    const fillWidth = (value / maxValue) * width;
    this.ctx.fillStyle = fillColor;
    this.ctx.fillRect(x, y, fillWidth, height);

    // Border
    this.ctx.strokeStyle = borderColor;
    this.ctx.lineWidth = borderWidth;
    this.ctx.strokeRect(x, y, width, height);
  }

  // วาด particle
  drawParticle(x, y, radius, color, alpha = 1) {
    this.ctx.globalAlpha = alpha;
    this.ctx.beginPath();
    this.ctx.arc(x, y, radius, 0, Math.PI * 2);
    this.ctx.fillStyle = color;
    this.ctx.fill();
    this.ctx.globalAlpha = 1;
  }

  // Camera transform
  applyCamera(camera) {
    this.ctx.save();
    this.ctx.translate(-camera.x + this.width / 2, -camera.y + this.height / 2);
  }

  resetCamera() {
    this.ctx.restore();
  }
}

// Spritesheet Animation
class AnimationController {
  constructor(spritesheet, frameWidth, frameHeight) {
    this.spritesheet = spritesheet;
    this.frameWidth = frameWidth;
    this.frameHeight = frameHeight;
    this.animations = {};
    this.currentAnimation = null;
    this.currentFrame = 0;
    this.elapsedTime = 0;
  }

  addAnimation(name, frames, fps) {
    this.animations[name] = {
      frames, // array of [col, row] positions
      fps,
      frameDuration: 1 / fps
    };
  }

  play(name, loop = true) {
    if (this.currentAnimation !== name) {
      this.currentAnimation = name;
      this.currentFrame = 0;
      this.elapsedTime = 0;
      this.loop = loop;
    }
  }

  update(deltaTime) {
    if (!this.currentAnimation) return;

    const anim = this.animations[this.currentAnimation];
    this.elapsedTime += deltaTime;

    if (this.elapsedTime >= anim.frameDuration) {
      this.elapsedTime -= anim.frameDuration;
      this.currentFrame++;

      if (this.currentFrame >= anim.frames.length) {
        if (this.loop) {
          this.currentFrame = 0;
        } else {
          this.currentFrame = anim.frames.length - 1;
        }
      }
    }
  }

  render(ctx, x, y, scaleX = 1, scaleY = 1) {
    if (!this.currentAnimation) return;

    const anim = this.animations[this.currentAnimation];
    const [col, row] = anim.frames[this.currentFrame];

    ctx.save();
    ctx.translate(x, y);
    ctx.scale(scaleX, scaleY);

    ctx.drawImage(
      this.spritesheet,
      col * this.frameWidth,
      row * this.frameHeight,
      this.frameWidth,
      this.frameHeight,
      -this.frameWidth / 2,
      -this.frameHeight / 2,
      this.frameWidth,
      this.frameHeight
    );

    ctx.restore();
  }
}
```

---

## Step 1894: Input Handling

### การจัดการ Input

```javascript
// Input Manager ที่ครอบคลุม
class InputManager {
  constructor() {
    this.keys = {};
    this.mouse = {
      x: 0,
      y: 0,
      buttons: {},
      wheel: 0
    };
    this.gamepad = null;
    this.touches = {};

    this.setupListeners();
  }

  setupListeners() {
    // Keyboard
    window.addEventListener('keydown', (e) => {
      this.keys[e.code] = true;
      this.keys[e.key] = true;
    });

    window.addEventListener('keyup', (e) => {
      this.keys[e.code] = false;
      this.keys[e.key] = false;
    });

    // Mouse
    window.addEventListener('mousemove', (e) => {
      const rect = document.getElementById('gameCanvas').getBoundingClientRect();
      this.mouse.x = e.clientX - rect.left;
      this.mouse.y = e.clientY - rect.top;
    });

    window.addEventListener('mousedown', (e) => {
      this.mouse.buttons[e.button] = true;
    });

    window.addEventListener('mouseup', (e) => {
      this.mouse.buttons[e.button] = false;
    });

    window.addEventListener('wheel', (e) => {
      this.mouse.wheel = e.deltaY;
    });

    // Touch
    window.addEventListener('touchstart', (e) => {
      e.preventDefault();
      Array.from(e.changedTouches).forEach(touch => {
        this.touches[touch.identifier] = { x: touch.clientX, y: touch.clientY };
      });
    }, { passive: false });

    window.addEventListener('touchmove', (e) => {
      e.preventDefault();
      Array.from(e.changedTouches).forEach(touch => {
        this.touches[touch.identifier] = { x: touch.clientX, y: touch.clientY };
      });
    }, { passive: false });

    window.addEventListener('touchend', (e) => {
      Array.from(e.changedTouches).forEach(touch => {
        delete this.touches[touch.identifier];
      });
    });

    // Gamepad
    window.addEventListener('gamepadconnected', (e) => {
      console.log('Gamepad connected:', e.gamepad.id);
      this.gamepad = e.gamepad;
    });

    window.addEventListener('gamepaddisconnected', () => {
      this.gamepad = null;
    });
  }

  isKeyDown(key) {
    return this.keys[key] === true;
  }

  isMouseDown(button = 0) {
    return this.mouse.buttons[button] === true;
  }

  getGamepadAxis(axisIndex) {
    const gamepads = navigator.getGamepads();
    if (gamepads[0]) {
      return gamepads[0].axes[axisIndex];
    }
    return 0;
  }

  isGamepadButtonDown(buttonIndex) {
    const gamepads = navigator.getGamepads();
    if (gamepads[0]) {
      return gamepads[0].buttons[buttonIndex].pressed;
    }
    return false;
  }

  // Virtual joystick สำหรับ mobile
  createVirtualJoystick(canvas) {
    return new VirtualJoystick(canvas);
  }

  update() {
    // Reset wheel ทุก frame
    this.mouse.wheel = 0;
  }
}

// Virtual Joystick สำหรับ Mobile
class VirtualJoystick {
  constructor(canvas) {
    this.canvas = canvas;
    this.ctx = canvas.getContext('2d');
    this.active = false;
    this.centerX = 0;
    this.centerY = 0;
    this.currentX = 0;
    this.currentY = 0;
    this.radius = 50;
    this.maxDistance = 40;
    this.identifier = null;

    this.setupListeners();
  }

  setupListeners() {
    this.canvas.addEventListener('touchstart', (e) => {
      const touch = e.touches[0];
      const rect = this.canvas.getBoundingClientRect();
      const x = touch.clientX - rect.left;
      const y = touch.clientY - rect.top;

      // แสดง joystick ที่จุดที่กด
      this.active = true;
      this.centerX = x;
      this.centerY = y;
      this.currentX = x;
      this.currentY = y;
      this.identifier = touch.identifier;
    });

    this.canvas.addEventListener('touchmove', (e) => {
      if (!this.active) return;

      const touch = Array.from(e.touches).find(t => t.identifier === this.identifier);
      if (!touch) return;

      const rect = this.canvas.getBoundingClientRect();
      const x = touch.clientX - rect.left;
      const y = touch.clientY - rect.top;

      const dx = x - this.centerX;
      const dy = y - this.centerY;
      const distance = Math.sqrt(dx * dx + dy * dy);

      if (distance <= this.maxDistance) {
        this.currentX = x;
        this.currentY = y;
      } else {
        const angle = Math.atan2(dy, dx);
        this.currentX = this.centerX + Math.cos(angle) * this.maxDistance;
        this.currentY = this.centerY + Math.sin(angle) * this.maxDistance;
      }
    });

    this.canvas.addEventListener('touchend', () => {
      this.active = false;
      this.identifier = null;
    });
  }

  getX() {
    if (!this.active) return 0;
    return (this.currentX - this.centerX) / this.maxDistance;
  }

  getY() {
    if (!this.active) return 0;
    return (this.currentY - this.centerY) / this.maxDistance;
  }

  render(ctx) {
    if (!this.active) return;

    // วงกลม outer
    ctx.beginPath();
    ctx.arc(this.centerX, this.centerY, this.radius, 0, Math.PI * 2);
    ctx.strokeStyle = 'rgba(255, 255, 255, 0.3)';
    ctx.lineWidth = 2;
    ctx.stroke();

    // วงกลม inner (thumb)
    ctx.beginPath();
    ctx.arc(this.currentX, this.currentY, this.radius / 2, 0, Math.PI * 2);
    ctx.fillStyle = 'rgba(255, 255, 255, 0.5)';
    ctx.fill();
  }
}
```

---

## Step 1895: Particle System

### ระบบ Particle Effect

```javascript
// Particle System
class Particle {
  constructor(x, y, options = {}) {
    this.x = x;
    this.y = y;
    this.vx = (Math.random() - 0.5) * (options.speed || 200);
    this.vy = (Math.random() - 0.5) * (options.speed || 200) - (options.gravity || 50);
    this.life = 1.0;
    this.decay = options.decay || (Math.random() * 0.5 + 0.5);
    this.size = options.size || (Math.random() * 10 + 5);
    this.color = options.color || `hsl(${Math.random() * 360}, 100%, 60%)`;
    this.gravity = options.gravity || 200;
    this.active = true;
  }

  update(deltaTime) {
    this.x += this.vx * deltaTime;
    this.y += this.vy * deltaTime;
    this.vy += this.gravity * deltaTime;
    this.life -= this.decay * deltaTime;
    this.size *= 0.99;

    if (this.life <= 0) {
      this.active = false;
    }
  }

  render(ctx) {
    ctx.globalAlpha = this.life;
    ctx.beginPath();
    ctx.arc(this.x, this.y, this.size, 0, Math.PI * 2);
    ctx.fillStyle = this.color;
    ctx.fill();
    ctx.globalAlpha = 1;
  }
}

class ParticleSystem {
  constructor(maxParticles = 1000) {
    this.particles = [];
    this.maxParticles = maxParticles;
    this.pool = []; // Object pool สำหรับ performance
  }

  emit(x, y, count, options = {}) {
    for (let i = 0; i < count; i++) {
      if (this.particles.length >= this.maxParticles) break;
      this.particles.push(new Particle(x, y, options));
    }
  }

  // Explosion effect
  explosion(x, y, options = {}) {
    const count = options.count || 30;
    const colors = ['#ff4444', '#ff8800', '#ffff00', '#ffffff'];

    for (let i = 0; i < count; i++) {
      const angle = (Math.PI * 2 * i) / count;
      const speed = (Math.random() * 200 + 100);
      const particle = new Particle(x, y, {
        ...options,
        color: colors[Math.floor(Math.random() * colors.length)]
      });
      particle.vx = Math.cos(angle) * speed;
      particle.vy = Math.sin(angle) * speed;
      this.particles.push(particle);
    }
  }

  // Trail effect
  trail(x, y, color = '#0099ff') {
    this.particles.push(new Particle(x, y, {
      speed: 20,
      decay: 2,
      size: 5,
      color,
      gravity: 0
    }));
  }

  update(deltaTime) {
    this.particles.forEach(p => p.update(deltaTime));
    this.particles = this.particles.filter(p => p.active);
  }

  render(ctx) {
    this.particles.forEach(p => p.render(ctx));
  }
}

// ตัวอย่างการใช้งาน: Firework
class Firework {
  constructor(canvas) {
    this.canvas = canvas;
    this.ctx = canvas.getContext('2d');
    this.particles = new ParticleSystem(2000);
    this.rockets = [];
  }

  launch(x) {
    this.rockets.push({
      x,
      y: this.canvas.height,
      vy: -600,
      color: `hsl(${Math.random() * 360}, 100%, 70%)`
    });
  }

  update(deltaTime) {
    // อัปเดต rockets
    this.rockets = this.rockets.filter(rocket => {
      rocket.y += rocket.vy * deltaTime;

      // เพิ่ม trail
      this.particles.trail(rocket.x, rocket.y, rocket.color);

      // ระเบิดเมื่อถึงจุดสูงสุด
      if (rocket.vy >= 0 || rocket.y < this.canvas.height * 0.3) {
        this.particles.explosion(rocket.x, rocket.y, {
          count: 50,
          speed: 300,
          gravity: 200
        });
        return false;
      }

      rocket.vy += 500 * deltaTime; // gravity
      return true;
    });

    this.particles.update(deltaTime);
  }

  render() {
    // Background fade effect
    this.ctx.fillStyle = 'rgba(0, 0, 0, 0.1)';
    this.ctx.fillRect(0, 0, this.canvas.width, this.canvas.height);

    // วาด rockets
    this.rockets.forEach(rocket => {
      this.ctx.beginPath();
      this.ctx.arc(rocket.x, rocket.y, 3, 0, Math.PI * 2);
      this.ctx.fillStyle = rocket.color;
      this.ctx.fill();
    });

    this.particles.render(this.ctx);
  }
}
```

---

## Step 1896: Phaser.js Framework

### การติดตั้งและตั้งค่า Phaser.js

```html
<!-- ติดตั้ง Phaser ผ่าน CDN -->
<script src="https://cdn.jsdelivr.net/npm/phaser@3.60.0/dist/phaser.min.js"></script>

<!-- หรือใช้ npm -->
<!-- npm install phaser -->
```

```javascript
// การสร้างเกมด้วย Phaser.js
const config = {
  type: Phaser.AUTO, // AUTO จะเลือก WebGL ถ้าทำได้ ไม่งั้นใช้ Canvas
  width: 800,
  height: 600,
  backgroundColor: '#1a1a2e',
  physics: {
    default: 'arcade',
    arcade: {
      gravity: { y: 400 },
      debug: false
    }
  },
  scene: [BootScene, GameScene, UIScene]
};

const game = new Phaser.Game(config);

// Boot Scene - โหลด assets
class BootScene extends Phaser.Scene {
  constructor() {
    super({ key: 'BootScene' });
  }

  preload() {
    // แสดง loading bar
    const progressBar = this.add.graphics();
    const progressBox = this.add.graphics();
    progressBox.fillStyle(0x222222, 0.8);
    progressBox.fillRect(240, 270, 320, 50);

    const width = this.cameras.main.width;
    const height = this.cameras.main.height;

    const percentText = this.make.text({
      x: width / 2,
      y: height / 2 - 5,
      text: '0%',
      style: { font: '18px monospace', fill: '#ffffff' }
    });
    percentText.setOrigin(0.5, 0.5);

    this.load.on('progress', (value) => {
      percentText.setText(parseInt(value * 100) + '%');
      progressBar.clear();
      progressBar.fillStyle(0xffffff, 1);
      progressBar.fillRect(250, 280, 300 * value, 30);
    });

    this.load.on('complete', () => {
      progressBar.destroy();
      progressBox.destroy();
      percentText.destroy();
    });

    // โหลด assets
    this.load.image('player', 'assets/player.png');
    this.load.image('platform', 'assets/platform.png');
    this.load.image('coin', 'assets/coin.png');
    this.load.spritesheet('player-run', 'assets/player-run.png', {
      frameWidth: 48,
      frameHeight: 48
    });
    this.load.tilemapTiledJSON('level1', 'assets/level1.json');
    this.load.audio('jump', 'assets/jump.mp3');
    this.load.audio('coin-collect', 'assets/coin.mp3');
    this.load.audio('bgm', 'assets/bgm.mp3');
  }

  create() {
    this.scene.start('GameScene');
  }
}

// Game Scene หลัก
class GameScene extends Phaser.Scene {
  constructor() {
    super({ key: 'GameScene' });
    this.score = 0;
    this.lives = 3;
  }

  create() {
    // สร้าง background parallax
    this.bg1 = this.add.image(400, 300, 'background-far').setScrollFactor(0.1);
    this.bg2 = this.add.image(400, 300, 'background-mid').setScrollFactor(0.5);

    // สร้าง tilemap
    const map = this.make.tilemap({ key: 'level1' });
    const tileset = map.addTilesetImage('tiles', 'tileset');
    this.groundLayer = map.createLayer('Ground', tileset, 0, 0);
    this.groundLayer.setCollisionByProperty({ collides: true });

    // สร้าง player
    this.player = this.physics.add.sprite(100, 450, 'player');
    this.player.setCollideWorldBounds(true);
    this.player.setBounce(0.1);

    // สร้าง animations
    this.anims.create({
      key: 'idle',
      frames: this.anims.generateFrameNumbers('player-run', { start: 0, end: 3 }),
      frameRate: 8,
      repeat: -1
    });

    this.anims.create({
      key: 'run',
      frames: this.anims.generateFrameNumbers('player-run', { start: 4, end: 11 }),
      frameRate: 12,
      repeat: -1
    });

    this.anims.create({
      key: 'jump',
      frames: this.anims.generateFrameNumbers('player-run', { start: 12, end: 15 }),
      frameRate: 8,
      repeat: 0
    });

    // Collision
    this.physics.add.collider(this.player, this.groundLayer);

    // สร้าง coins
    this.coins = this.physics.add.staticGroup();
    for (let i = 0; i < 20; i++) {
      const coin = this.coins.create(
        Phaser.Math.Between(100, 700),
        Phaser.Math.Between(100, 500),
        'coin'
      );
    }

    // Overlap coins
    this.physics.add.overlap(this.player, this.coins, this.collectCoin, null, this);

    // Camera
    this.cameras.main.setBounds(0, 0, map.widthInPixels, map.heightInPixels);
    this.cameras.main.startFollow(this.player, true, 0.1, 0.1);

    // Input
    this.cursors = this.input.keyboard.createCursorKeys();
    this.wasd = this.input.keyboard.addKeys({
      up: Phaser.Input.Keyboard.KeyCodes.W,
      left: Phaser.Input.Keyboard.KeyCodes.A,
      right: Phaser.Input.Keyboard.KeyCodes.D
    });

    // Sound
    this.jumpSound = this.sound.add('jump', { volume: 0.5 });
    this.coinSound = this.sound.add('coin-collect', { volume: 0.7 });
    this.bgm = this.sound.add('bgm', { loop: true, volume: 0.3 });
    this.bgm.play();

    // UI
    this.scene.launch('UIScene', { score: this.score, lives: this.lives });
    this.uiScene = this.scene.get('UIScene');
  }

  collectCoin(player, coin) {
    coin.destroy();
    this.score += 10;
    this.coinSound.play();
    this.uiScene.updateScore(this.score);

    // Particle effect
    const particles = this.add.particles('coin');
    const emitter = particles.createEmitter({
      x: coin.x,
      y: coin.y,
      speed: { min: 100, max: 200 },
      scale: { start: 0.5, end: 0 },
      lifespan: 500,
      quantity: 5,
      on: false
    });
    emitter.explode();
    this.time.delayedCall(600, () => particles.destroy());
  }

  update() {
    const onGround = this.player.body.blocked.down;

    // Movement
    if (this.cursors.left.isDown || this.wasd.left.isDown) {
      this.player.setVelocityX(-200);
      this.player.flipX = true;
      if (onGround) this.player.anims.play('run', true);
    } else if (this.cursors.right.isDown || this.wasd.right.isDown) {
      this.player.setVelocityX(200);
      this.player.flipX = false;
      if (onGround) this.player.anims.play('run', true);
    } else {
      this.player.setVelocityX(0);
      if (onGround) this.player.anims.play('idle', true);
    }

    // Jump
    if ((this.cursors.up.isDown || this.wasd.up.isDown) && onGround) {
      this.player.setVelocityY(-600);
      this.player.anims.play('jump', true);
      this.jumpSound.play();
    }

    // ตรวจสอบ out of bounds
    if (this.player.y > this.physics.world.bounds.height) {
      this.playerDie();
    }
  }

  playerDie() {
    this.lives--;
    if (this.lives <= 0) {
      this.bgm.stop();
      this.scene.start('GameOverScene', { score: this.score });
    } else {
      this.player.setPosition(100, 450);
      this.uiScene.updateLives(this.lives);
    }
  }
}

// UI Scene
class UIScene extends Phaser.Scene {
  constructor() {
    super({ key: 'UIScene' });
  }

  init(data) {
    this.score = data.score;
    this.lives = data.lives;
  }

  create() {
    this.scoreText = this.add.text(16, 16, `Score: ${this.score}`, {
      fontSize: '24px',
      fill: '#fff',
      stroke: '#000',
      strokeThickness: 3
    });

    this.livesText = this.add.text(16, 48, `Lives: ${this.lives}`, {
      fontSize: '24px',
      fill: '#fff',
      stroke: '#000',
      strokeThickness: 3
    });
  }

  updateScore(score) {
    this.score = score;
    this.scoreText.setText(`Score: ${score}`);
  }

  updateLives(lives) {
    this.lives = lives;
    this.livesText.setText(`Lives: ${lives}`);
  }
}
```

---

## Step 1897: Phaser Physics

### Arcade Physics และ Matter.js

```javascript
// Arcade Physics - เร็ว ง่าย สำหรับเกม platformer
class ArcadePhysicsDemo extends Phaser.Scene {
  create() {
    // Static group สำหรับ platforms
    const platforms = this.physics.add.staticGroup();
    platforms.create(400, 568, 'ground').setScale(2).refreshBody();
    platforms.create(600, 400, 'platform');
    platforms.create(50, 250, 'platform');
    platforms.create(750, 220, 'platform');

    // Dynamic body
    const player = this.physics.add.sprite(100, 450, 'player');
    player.setCollideWorldBounds(true);
    player.setGravityY(300); // gravity เพิ่มเติมจาก world gravity

    // Collider
    this.physics.add.collider(player, platforms);

    // Velocity
    player.setVelocityX(160);
    player.setVelocityY(-330);

    // Bounce
    player.setBounce(0.2, 0.2);

    // Debug
    this.physics.world.createDebugGraphic();
  }
}

// Matter.js Physics - สำหรับ physics ที่ซับซ้อน
class MatterPhysicsDemo extends Phaser.Scene {
  constructor() {
    super({
      key: 'MatterPhysicsDemo',
      physics: {
        default: 'matter',
        matter: {
          gravity: { y: 1 },
          debug: true
        }
      }
    });
  }

  create() {
    // สร้าง body ด้วยรูปทรงต่างๆ
    const box = this.matter.add.image(400, 100, 'crate');
    box.setFriction(0.005);
    box.setFrictionAir(0.001);
    box.setBounce(0.3);

    // สร้าง polygon
    const star = this.matter.add.polygon(200, 50, 5, 40, {
      restitution: 0.8,
      friction: 0.05
    });

    // Compound body
    const compound = this.matter.add.gameObject(this.add.sprite(500, 50, 'player'), {
      shape: {
        type: 'fromVerts',
        verts: [
          { x: 0, y: 0 },
          { x: 50, y: 0 },
          { x: 50, y: 50 },
          { x: 25, y: 70 },
          { x: 0, y: 50 }
        ]
      }
    });

    // Ground
    this.matter.add.rectangle(400, 580, 800, 20, { isStatic: true });

    // Constraints
    const ballA = this.matter.add.circle(200, 200, 20);
    const ballB = this.matter.add.circle(300, 200, 20);
    const constraint = this.matter.add.constraint(ballA, ballB, 100, 0.5);

    // Events
    this.matter.world.on('collisionstart', (event) => {
      event.pairs.forEach(pair => {
        console.log('Collision:', pair.bodyA.label, pair.bodyB.label);
      });
    });
  }
}
```

---

## Step 1898: Tilemap System

### การใช้งาน Tiled Map Editor

```javascript
// สร้าง Tilemap จาก Tiled Editor
class TilemapGame extends Phaser.Scene {
  create() {
    // สร้าง tilemap
    const map = this.make.tilemap({ key: 'level1' });

    // เพิ่ม tileset
    const groundTiles = map.addTilesetImage('ground', 'ground-tiles');
    const decorTiles = map.addTilesetImage('decoration', 'decor-tiles');

    // สร้าง layers
    const backgroundLayer = map.createLayer('Background', groundTiles, 0, 0);
    const groundLayer = map.createLayer('Ground', groundTiles, 0, 0);
    const platformLayer = map.createLayer('Platforms', groundTiles, 0, 0);
    const decorLayer = map.createLayer('Decoration', decorTiles, 0, 0);
    const foregroundLayer = map.createLayer('Foreground', decorTiles, 0, 0);

    // ตั้งค่า collision
    groundLayer.setCollisionByProperty({ collides: true });
    platformLayer.setCollisionByExclusion([-1]);

    // ปรับ depth
    backgroundLayer.setDepth(-2);
    groundLayer.setDepth(-1);
    decorLayer.setDepth(0);
    foregroundLayer.setDepth(2); // หน้า player

    // อ่าน object layer จาก Tiled
    const spawnPoint = map.findObject('Objects', obj => obj.name === 'Spawn');
    const enemies = map.filterObjects('Objects', obj => obj.type === 'Enemy');
    const coins = map.filterObjects('Objects', obj => obj.type === 'Coin');

    // สร้าง player จาก spawn point
    this.player = this.physics.add.sprite(spawnPoint.x, spawnPoint.y, 'player');
    this.physics.add.collider(this.player, groundLayer);

    // สร้าง enemies จาก object layer
    this.enemies = enemies.map(enemyData => {
      const enemy = this.physics.add.sprite(enemyData.x, enemyData.y, 'enemy');
      enemy.patrol = {
        startX: enemyData.x,
        distance: enemyData.properties?.find(p => p.name === 'patrol')?.value || 100
      };
      return enemy;
    });

    // Dynamic tilemap (เปลี่ยน tile ได้)
    this.groundLayer = groundLayer;
  }

  // ทำลาย tile เมื่อถูกยิง
  destroyTile(x, y) {
    const tile = this.groundLayer.getTileAtWorldXY(x, y);
    if (tile && tile.properties.destructible) {
      this.groundLayer.removeTileAtWorldXY(x, y);

      // Particle effect
      this.particles.explode(10, x, y);
    }
  }

  // เปลี่ยน tile (เช่น ประตูเปิด)
  replaceTile(worldX, worldY, tileIndex) {
    this.groundLayer.putTileAtWorldXY(tileIndex, worldX, worldY);
  }
}
```

---

## Step 1899: Collision Detection จาก Scratch

### การเขียน Collision Detection เอง

```javascript
// AABB (Axis-Aligned Bounding Box) Collision
function aabbCollision(a, b) {
  return (
    a.x < b.x + b.width &&
    a.x + a.width > b.x &&
    a.y < b.y + b.height &&
    a.y + a.height > b.y
  );
}

// MTV (Minimum Translation Vector) สำหรับ push-out
function aabbMTV(a, b) {
  const dx = (a.x + a.width / 2) - (b.x + b.width / 2);
  const dy = (a.y + a.height / 2) - (b.y + b.height / 2);
  const overlapX = (a.width / 2 + b.width / 2) - Math.abs(dx);
  const overlapY = (a.height / 2 + b.height / 2) - Math.abs(dy);

  if (overlapX <= 0 || overlapY <= 0) return null;

  if (overlapX < overlapY) {
    return { x: dx > 0 ? overlapX : -overlapX, y: 0 };
  } else {
    return { x: 0, y: dy > 0 ? overlapY : -overlapY };
  }
}

// Circle Collision
function circleCollision(a, b) {
  const dx = a.x - b.x;
  const dy = a.y - b.y;
  const distance = Math.sqrt(dx * dx + dy * dy);
  return distance < a.radius + b.radius;
}

// SAT (Separating Axis Theorem) สำหรับ convex polygons
function satCollision(polygon1, polygon2) {
  const polygons = [polygon1, polygon2];

  for (let i = 0; i < polygons.length; i++) {
    const polygon = polygons[i];

    for (let j = 0; j < polygon.vertices.length; j++) {
      const vertex1 = polygon.vertices[j];
      const vertex2 = polygon.vertices[(j + 1) % polygon.vertices.length];

      // Normal ของ edge
      const normal = {
        x: -(vertex2.y - vertex1.y),
        y: vertex2.x - vertex1.x
      };

      // Normalize
      const len = Math.sqrt(normal.x * normal.x + normal.y * normal.y);
      normal.x /= len;
      normal.y /= len;

      // Project polygons
      const proj1 = projectPolygon(polygon1, normal);
      const proj2 = projectPolygon(polygon2, normal);

      // Check separation
      if (proj1.max < proj2.min || proj2.max < proj1.min) {
        return false; // No collision
      }
    }
  }

  return true; // Collision
}

function projectPolygon(polygon, axis) {
  let min = Infinity;
  let max = -Infinity;

  polygon.vertices.forEach(vertex => {
    const projection = vertex.x * axis.x + vertex.y * axis.y;
    min = Math.min(min, projection);
    max = Math.max(max, projection);
  });

  return { min, max };
}

// Broad Phase - Spatial Grid
class SpatialGrid {
  constructor(cellSize = 100) {
    this.cellSize = cellSize;
    this.cells = new Map();
  }

  _getKey(x, y) {
    return `${Math.floor(x / this.cellSize)},${Math.floor(y / this.cellSize)}`;
  }

  insert(entity) {
    const startX = Math.floor(entity.x / this.cellSize);
    const startY = Math.floor(entity.y / this.cellSize);
    const endX = Math.floor((entity.x + entity.width) / this.cellSize);
    const endY = Math.floor((entity.y + entity.height) / this.cellSize);

    for (let x = startX; x <= endX; x++) {
      for (let y = startY; y <= endY; y++) {
        const key = `${x},${y}`;
        if (!this.cells.has(key)) {
          this.cells.set(key, []);
        }
        this.cells.get(key).push(entity);
      }
    }
  }

  query(entity) {
    const nearby = new Set();
    const startX = Math.floor(entity.x / this.cellSize);
    const startY = Math.floor(entity.y / this.cellSize);
    const endX = Math.floor((entity.x + entity.width) / this.cellSize);
    const endY = Math.floor((entity.y + entity.height) / this.cellSize);

    for (let x = startX; x <= endX; x++) {
      for (let y = startY; y <= endY; y++) {
        const cell = this.cells.get(`${x},${y}`);
        if (cell) {
          cell.forEach(e => {
            if (e !== entity) nearby.add(e);
          });
        }
      }
    }

    return Array.from(nearby);
  }

  clear() {
    this.cells.clear();
  }
}

// Quadtree สำหรับ collision ที่มี entities จำนวนมาก
class QuadTree {
  constructor(boundary, capacity = 4) {
    this.boundary = boundary; // { x, y, width, height }
    this.capacity = capacity;
    this.points = [];
    this.divided = false;
  }

  subdivide() {
    const { x, y, width, height } = this.boundary;
    const hw = width / 2;
    const hh = height / 2;

    this.northeast = new QuadTree({ x: x + hw, y, width: hw, height: hh }, this.capacity);
    this.northwest = new QuadTree({ x, y, width: hw, height: hh }, this.capacity);
    this.southeast = new QuadTree({ x: x + hw, y: y + hh, width: hw, height: hh }, this.capacity);
    this.southwest = new QuadTree({ x, y: y + hh, width: hw, height: hh }, this.capacity);
    this.divided = true;
  }

  insert(point) {
    if (!this.contains(point)) return false;

    if (this.points.length < this.capacity) {
      this.points.push(point);
      return true;
    }

    if (!this.divided) this.subdivide();

    return (
      this.northeast.insert(point) ||
      this.northwest.insert(point) ||
      this.southeast.insert(point) ||
      this.southwest.insert(point)
    );
  }

  contains(point) {
    const { x, y, width, height } = this.boundary;
    return point.x >= x && point.x < x + width &&
           point.y >= y && point.y < y + height;
  }

  query(range, found = []) {
    if (!this.intersects(range)) return found;

    this.points.forEach(point => {
      if (this.intersects(range, point)) {
        found.push(point);
      }
    });

    if (this.divided) {
      this.northeast.query(range, found);
      this.northwest.query(range, found);
      this.southeast.query(range, found);
      this.southwest.query(range, found);
    }

    return found;
  }

  intersects(range, point = null) {
    if (point) {
      return point.x >= range.x && point.x < range.x + range.width &&
             point.y >= range.y && point.y < range.y + range.height;
    }

    const { x, y, width, height } = this.boundary;
    return !(range.x > x + width || range.x + range.width < x ||
             range.y > y + height || range.y + range.height < y);
  }
}
```

---

## Step 1900: Entity Component System (ECS)

### ECS Pattern สำหรับ Game Architecture

```javascript
// ECS Implementation
class World {
  constructor() {
    this.entities = new Map();
    this.components = new Map();
    this.systems = [];
    this.nextEntityId = 0;
  }

  createEntity() {
    const id = this.nextEntityId++;
    this.entities.set(id, new Set());
    return id;
  }

  destroyEntity(entityId) {
    // ลบ components ทั้งหมด
    if (this.entities.has(entityId)) {
      const components = this.entities.get(entityId);
      components.forEach(componentName => {
        this.components.get(componentName)?.delete(entityId);
      });
      this.entities.delete(entityId);
    }
  }

  addComponent(entityId, component) {
    const componentName = component.constructor.name;

    if (!this.components.has(componentName)) {
      this.components.set(componentName, new Map());
    }

    this.components.get(componentName).set(entityId, component);
    this.entities.get(entityId).add(componentName);
  }

  removeComponent(entityId, ComponentClass) {
    const componentName = ComponentClass.name;
    this.components.get(componentName)?.delete(entityId);
    this.entities.get(entityId)?.delete(componentName);
  }

  getComponent(entityId, ComponentClass) {
    return this.components.get(ComponentClass.name)?.get(entityId);
  }

  hasComponent(entityId, ComponentClass) {
    return this.components.get(ComponentClass.name)?.has(entityId) ?? false;
  }

  // Query entities ที่มี components ครบตามที่ต้องการ
  query(...ComponentClasses) {
    const results = [];

    this.entities.forEach((components, entityId) => {
      const hasAll = ComponentClasses.every(
        ComponentClass => this.hasComponent(entityId, ComponentClass)
      );

      if (hasAll) {
        results.push(entityId);
      }
    });

    return results;
  }

  addSystem(system) {
    this.systems.push(system);
    system.world = this;
  }

  update(deltaTime) {
    this.systems.forEach(system => system.update(deltaTime));
  }
}

// Components
class TransformComponent {
  constructor(x = 0, y = 0, rotation = 0, scale = 1) {
    this.x = x;
    this.y = y;
    this.rotation = rotation;
    this.scale = scale;
  }
}

class VelocityComponent {
  constructor(vx = 0, vy = 0) {
    this.vx = vx;
    this.vy = vy;
  }
}

class RenderComponent {
  constructor(sprite, width, height, color = 'white') {
    this.sprite = sprite;
    this.width = width;
    this.height = height;
    this.color = color;
    this.visible = true;
  }
}

class HealthComponent {
  constructor(maxHp) {
    this.hp = maxHp;
    this.maxHp = maxHp;
    this.alive = true;
  }

  takeDamage(amount) {
    this.hp = Math.max(0, this.hp - amount);
    if (this.hp === 0) this.alive = false;
  }

  heal(amount) {
    this.hp = Math.min(this.maxHp, this.hp + amount);
  }
}

class ColliderComponent {
  constructor(width, height, offsetX = 0, offsetY = 0) {
    this.width = width;
    this.height = height;
    this.offsetX = offsetX;
    this.offsetY = offsetY;
    this.layer = 'default';
    this.mask = ['default'];
  }
}

class PlayerInputComponent {
  constructor() {
    this.moveLeft = false;
    this.moveRight = false;
    this.jump = false;
  }
}

// Systems
class MovementSystem {
  update(deltaTime) {
    const entities = this.world.query(TransformComponent, VelocityComponent);

    entities.forEach(entityId => {
      const transform = this.world.getComponent(entityId, TransformComponent);
      const velocity = this.world.getComponent(entityId, VelocityComponent);

      transform.x += velocity.vx * deltaTime;
      transform.y += velocity.vy * deltaTime;
    });
  }
}

class PlayerInputSystem {
  constructor(inputManager) {
    this.inputManager = inputManager;
  }

  update() {
    const entities = this.world.query(PlayerInputComponent, VelocityComponent);

    entities.forEach(entityId => {
      const input = this.world.getComponent(entityId, PlayerInputComponent);
      const velocity = this.world.getComponent(entityId, VelocityComponent);

      input.moveLeft = this.inputManager.isKeyDown('ArrowLeft');
      input.moveRight = this.inputManager.isKeyDown('ArrowRight');
      input.jump = this.inputManager.isKeyDown('ArrowUp');

      if (input.moveLeft) velocity.vx = -200;
      else if (input.moveRight) velocity.vx = 200;
      else velocity.vx = 0;

      if (input.jump) velocity.vy = -500;
    });
  }
}

class RenderSystem {
  constructor(ctx) {
    this.ctx = ctx;
  }

  update() {
    const entities = this.world.query(TransformComponent, RenderComponent);

    entities.forEach(entityId => {
      const transform = this.world.getComponent(entityId, TransformComponent);
      const render = this.world.getComponent(entityId, RenderComponent);

      if (!render.visible) return;

      this.ctx.save();
      this.ctx.translate(transform.x, transform.y);
      this.ctx.rotate(transform.rotation);
      this.ctx.scale(transform.scale, transform.scale);

      if (render.sprite) {
        this.ctx.drawImage(
          render.sprite,
          -render.width / 2,
          -render.height / 2,
          render.width,
          render.height
        );
      } else {
        this.ctx.fillStyle = render.color;
        this.ctx.fillRect(-render.width / 2, -render.height / 2, render.width, render.height);
      }

      this.ctx.restore();
    });
  }
}

// ตัวอย่างการใช้งาน ECS
const world = new World();
const input = new InputManager();

// เพิ่ม systems
world.addSystem(new PlayerInputSystem(input));
world.addSystem(new MovementSystem());
world.addSystem(new RenderSystem(ctx));

// สร้าง player entity
const playerId = world.createEntity();
world.addComponent(playerId, new TransformComponent(100, 300));
world.addComponent(playerId, new VelocityComponent());
world.addComponent(playerId, new RenderComponent(null, 48, 48, '#00ff88'));
world.addComponent(playerId, new HealthComponent(100));
world.addComponent(playerId, new ColliderComponent(40, 48));
world.addComponent(playerId, new PlayerInputComponent());

// สร้าง enemy entity
const enemyId = world.createEntity();
world.addComponent(enemyId, new TransformComponent(500, 300));
world.addComponent(enemyId, new VelocityComponent(-50, 0));
world.addComponent(enemyId, new RenderComponent(null, 40, 40, '#ff4444'));
world.addComponent(enemyId, new HealthComponent(50));
world.addComponent(enemyId, new ColliderComponent(38, 38));
```

---

## Step 1901: Three.js สำหรับ 3D Games

### ติดตั้งและตั้งค่า Three.js

```html
<script src="https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/three@0.160.0/examples/js/controls/OrbitControls.js"></script>
```

```javascript
// Three.js Setup พื้นฐาน
class ThreeJSGame {
  constructor(container) {
    this.container = container;
    this.setupRenderer();
    this.setupScene();
    this.setupCamera();
    this.setupLights();
    this.setupControls();
    this.setupEventListeners();
    this.animate();
  }

  setupRenderer() {
    this.renderer = new THREE.WebGLRenderer({
      antialias: true,
      alpha: false
    });
    this.renderer.setPixelRatio(window.devicePixelRatio);
    this.renderer.setSize(window.innerWidth, window.innerHeight);
    this.renderer.shadowMap.enabled = true;
    this.renderer.shadowMap.type = THREE.PCFSoftShadowMap;
    this.renderer.outputEncoding = THREE.sRGBEncoding;
    this.renderer.toneMapping = THREE.ACESFilmicToneMapping;
    this.renderer.toneMappingExposure = 1;
    this.container.appendChild(this.renderer.domElement);
  }

  setupScene() {
    this.scene = new THREE.Scene();
    this.scene.background = new THREE.Color(0x0a0a1a);
    this.scene.fog = new THREE.FogExp2(0x0a0a1a, 0.05);
  }

  setupCamera() {
    this.camera = new THREE.PerspectiveCamera(
      75, // FOV
      window.innerWidth / window.innerHeight, // Aspect
      0.1, // Near
      1000 // Far
    );
    this.camera.position.set(0, 5, 10);
    this.camera.lookAt(0, 0, 0);
  }

  setupLights() {
    // Ambient Light
    const ambientLight = new THREE.AmbientLight(0x404040, 0.5);
    this.scene.add(ambientLight);

    // Directional Light (Sun)
    this.sunLight = new THREE.DirectionalLight(0xffffff, 1);
    this.sunLight.position.set(50, 50, 25);
    this.sunLight.castShadow = true;
    this.sunLight.shadow.camera.near = 0.5;
    this.sunLight.shadow.camera.far = 500;
    this.sunLight.shadow.camera.left = -50;
    this.sunLight.shadow.camera.right = 50;
    this.sunLight.shadow.camera.top = 50;
    this.sunLight.shadow.camera.bottom = -50;
    this.sunLight.shadow.mapSize.width = 2048;
    this.sunLight.shadow.mapSize.height = 2048;
    this.scene.add(this.sunLight);

    // Point Light
    const pointLight = new THREE.PointLight(0x0088ff, 2, 20);
    pointLight.position.set(-5, 5, 5);
    this.scene.add(pointLight);

    // Hemisphere Light
    const hemiLight = new THREE.HemisphereLight(0x87ceeb, 0x4a3728, 0.3);
    this.scene.add(hemiLight);
  }

  setupControls() {
    this.controls = new THREE.OrbitControls(this.camera, this.renderer.domElement);
    this.controls.enableDamping = true;
    this.controls.dampingFactor = 0.05;
    this.controls.maxPolarAngle = Math.PI / 2;
  }

  setupEventListeners() {
    window.addEventListener('resize', () => {
      this.camera.aspect = window.innerWidth / window.innerHeight;
      this.camera.updateProjectionMatrix();
      this.renderer.setSize(window.innerWidth, window.innerHeight);
    });
  }

  animate() {
    requestAnimationFrame(this.animate.bind(this));
    const delta = this.clock?.getDelta() || 0;
    this.update(delta);
    this.controls.update();
    this.renderer.render(this.scene, this.camera);
  }

  update(delta) {
    // Override
  }
}
```

---

## Step 1902: Three.js Geometries และ Materials

```javascript
// Geometries
class GeometryShowcase {
  constructor(scene) {
    this.scene = scene;
    this.objects = [];
    this.createObjects();
  }

  createObjects() {
    // Box
    const boxGeo = new THREE.BoxGeometry(1, 1, 1);

    // Sphere
    const sphereGeo = new THREE.SphereGeometry(0.5, 32, 32);

    // Cylinder
    const cylinderGeo = new THREE.CylinderGeometry(0.3, 0.5, 1.5, 16);

    // Torus
    const torusGeo = new THREE.TorusGeometry(0.5, 0.2, 16, 100);

    // Torus Knot
    const knotGeo = new THREE.TorusKnotGeometry(0.5, 0.15, 100, 16);

    // Custom geometry
    const customGeo = new THREE.BufferGeometry();
    const vertices = new Float32Array([
      -1, 0, 0,  1, 0, 0,  0, 2, 0,  // Triangle 1
      -1, 0, 0,  0, 0, -1,  0, 2, 0,  // Triangle 2
    ]);
    customGeo.setAttribute('position', new THREE.BufferAttribute(vertices, 3));
    customGeo.computeVertexNormals();

    // Materials
    const materials = [
      // Standard (ใช้ PBR)
      new THREE.MeshStandardMaterial({
        color: 0xff4444,
        roughness: 0.5,
        metalness: 0.3
      }),

      // Phong
      new THREE.MeshPhongMaterial({
        color: 0x44ff44,
        shininess: 100,
        specular: 0x00ff00
      }),

      // Toon
      new THREE.MeshToonMaterial({
        color: 0x4444ff
      }),

      // Wireframe
      new THREE.MeshBasicMaterial({
        color: 0xffffff,
        wireframe: true
      }),

      // Normal
      new THREE.MeshNormalMaterial(),

      // Physical (Realistic)
      new THREE.MeshPhysicalMaterial({
        color: 0xdddddd,
        roughness: 0.1,
        metalness: 0.9,
        clearcoat: 1.0,
        clearcoatRoughness: 0.1
      })
    ];

    const geometries = [boxGeo, sphereGeo, cylinderGeo, torusGeo, knotGeo, customGeo];

    geometries.forEach((geo, i) => {
      const mesh = new THREE.Mesh(geo, materials[i]);
      mesh.position.x = (i - 2.5) * 3;
      mesh.castShadow = true;
      mesh.receiveShadow = true;
      this.scene.add(mesh);
      this.objects.push(mesh);
    });
  }

  update(delta) {
    this.objects.forEach((obj, i) => {
      obj.rotation.x += delta * 0.5;
      obj.rotation.y += delta * (0.5 + i * 0.1);
    });
  }
}

// Texture Loading
async function loadTextures() {
  const loader = new THREE.TextureLoader();

  const loadTexture = (path) => new Promise((resolve) => {
    loader.load(path, resolve);
  });

  const [colorMap, normalMap, roughnessMap, aoMap] = await Promise.all([
    loadTexture('textures/color.jpg'),
    loadTexture('textures/normal.jpg'),
    loadTexture('textures/roughness.jpg'),
    loadTexture('textures/ao.jpg')
  ]);

  // ตั้งค่า texture
  [colorMap, normalMap, roughnessMap, aoMap].forEach(tex => {
    tex.wrapS = THREE.RepeatWrapping;
    tex.wrapT = THREE.RepeatWrapping;
    tex.repeat.set(4, 4);
  });

  return new THREE.MeshStandardMaterial({
    map: colorMap,
    normalMap,
    roughnessMap,
    aoMap,
    aoMapIntensity: 1
  });
}

// Environment Map (สำหรับ reflections)
function setupEnvironmentMap(scene, renderer) {
  const pmremGenerator = new THREE.PMREMGenerator(renderer);
  pmremGenerator.compileEquirectangularShader();

  const envTexture = new THREE.RGBELoader()
    .load('textures/hdr-env.hdr', (texture) => {
      const envMap = pmremGenerator.fromEquirectangular(texture).texture;
      scene.environment = envMap;
      scene.background = envMap;
      texture.dispose();
      pmremGenerator.dispose();
    });
}
```

---

## Step 1903: Three.js 3D Game

### สร้างเกม 3D พื้นฐาน

```javascript
// 3D Platformer Game
class Game3D extends ThreeJSGame {
  constructor(container) {
    super(container);
    this.clock = new THREE.Clock();
    this.player = null;
    this.platforms = [];
    this.coins = [];
    this.enemies = [];
    this.score = 0;
    this.inputManager = new InputManager();

    this.setupGame();
  }

  setupGame() {
    this.createGround();
    this.createPlatforms();
    this.createPlayer();
    this.createCoins();
    this.createEnemies();
    this.setupCamera();
  }

  createGround() {
    const groundGeo = new THREE.BoxGeometry(50, 1, 50);
    const groundMat = new THREE.MeshStandardMaterial({
      color: 0x2d5a27,
      roughness: 0.8
    });
    const ground = new THREE.Mesh(groundGeo, groundMat);
    ground.position.y = -0.5;
    ground.receiveShadow = true;
    this.scene.add(ground);
  }

  createPlatforms() {
    const platformPositions = [
      { x: 5, y: 2, z: 0, w: 5, h: 0.5, d: 3 },
      { x: -3, y: 4, z: -5, w: 4, h: 0.5, d: 4 },
      { x: 8, y: 6, z: -8, w: 3, h: 0.5, d: 3 },
      { x: 0, y: 8, z: -12, w: 6, h: 0.5, d: 2 }
    ];

    platformPositions.forEach(pos => {
      const geo = new THREE.BoxGeometry(pos.w, pos.h, pos.d);
      const mat = new THREE.MeshStandardMaterial({ color: 0x8b4513 });
      const platform = new THREE.Mesh(geo, mat);
      platform.position.set(pos.x, pos.y, pos.z);
      platform.castShadow = true;
      platform.receiveShadow = true;
      this.scene.add(platform);

      this.platforms.push({
        mesh: platform,
        bounds: new THREE.Box3().setFromObject(platform)
      });
    });
  }

  createPlayer() {
    const geometry = new THREE.CapsuleGeometry(0.4, 0.8, 4, 8);
    const material = new THREE.MeshStandardMaterial({ color: 0x0088ff });
    this.player = new THREE.Mesh(geometry, material);
    this.player.position.set(0, 2, 0);
    this.player.castShadow = true;

    // Player state
    this.player.velocity = new THREE.Vector3();
    this.player.onGround = false;
    this.player.speed = 8;
    this.player.jumpForce = 12;

    this.scene.add(this.player);
  }

  createCoins() {
    const coinGeo = new THREE.CylinderGeometry(0.3, 0.3, 0.1, 16);
    const coinMat = new THREE.MeshStandardMaterial({
      color: 0xffcc00,
      metalness: 0.8,
      roughness: 0.2
    });

    for (let i = 0; i < 20; i++) {
      const coin = new THREE.Mesh(coinGeo, coinMat);
      coin.position.set(
        (Math.random() - 0.5) * 30,
        Math.random() * 5 + 1,
        (Math.random() - 0.5) * 30
      );
      coin.rotation.x = Math.PI / 2;
      this.scene.add(coin);
      this.coins.push(coin);
    }
  }

  createEnemies() {
    const enemyGeo = new THREE.BoxGeometry(0.8, 0.8, 0.8);
    const enemyMat = new THREE.MeshStandardMaterial({ color: 0xff4444 });

    for (let i = 0; i < 5; i++) {
      const enemy = new THREE.Mesh(enemyGeo, enemyMat);
      enemy.position.set(
        (Math.random() - 0.5) * 20,
        0.4,
        (Math.random() - 0.5) * 20
      );
      enemy.direction = Math.random() * Math.PI * 2;
      enemy.speed = 3;
      this.scene.add(enemy);
      this.enemies.push(enemy);
    }
  }

  setupCamera() {
    // Third-person camera
    this.cameraOffset = new THREE.Vector3(0, 5, 8);
    this.cameraTarget = new THREE.Vector3();
  }

  update(delta) {
    this.handleInput(delta);
    this.updatePhysics(delta);
    this.updateCamera();
    this.updateCoins(delta);
    this.updateEnemies(delta);
    this.checkCollisions();
  }

  handleInput(delta) {
    const input = this.inputManager;
    const velocity = this.player.velocity;

    // Camera-relative movement
    const cameraDir = new THREE.Vector3();
    this.camera.getWorldDirection(cameraDir);
    cameraDir.y = 0;
    cameraDir.normalize();

    const cameraRight = new THREE.Vector3();
    cameraRight.crossVectors(cameraDir, new THREE.Vector3(0, 1, 0));

    const moveDir = new THREE.Vector3();

    if (input.isKeyDown('ArrowUp') || input.isKeyDown('KeyW')) {
      moveDir.add(cameraDir);
    }
    if (input.isKeyDown('ArrowDown') || input.isKeyDown('KeyS')) {
      moveDir.sub(cameraDir);
    }
    if (input.isKeyDown('ArrowLeft') || input.isKeyDown('KeyA')) {
      moveDir.sub(cameraRight);
    }
    if (input.isKeyDown('ArrowRight') || input.isKeyDown('KeyD')) {
      moveDir.add(cameraRight);
    }

    if (moveDir.length() > 0) {
      moveDir.normalize();
      velocity.x = moveDir.x * this.player.speed;
      velocity.z = moveDir.z * this.player.speed;

      // Rotate player to face movement direction
      const targetAngle = Math.atan2(moveDir.x, moveDir.z);
      this.player.rotation.y = targetAngle;
    } else {
      velocity.x *= 0.8;
      velocity.z *= 0.8;
    }

    // Jump
    if (input.isKeyDown('Space') && this.player.onGround) {
      velocity.y = this.player.jumpForce;
      this.player.onGround = false;
    }
  }

  updatePhysics(delta) {
    const vel = this.player.velocity;
    const pos = this.player.position;

    // Gravity
    vel.y -= 25 * delta;

    // Clamp velocity
    vel.y = Math.max(vel.y, -30);

    // Apply velocity
    pos.x += vel.x * delta;
    pos.y += vel.y * delta;
    pos.z += vel.z * delta;

    // Ground check
    if (pos.y < 0.9) {
      pos.y = 0.9;
      vel.y = 0;
      this.player.onGround = true;
    }

    // Platform collision
    this.platforms.forEach(platform => {
      const bounds = new THREE.Box3().setFromObject(platform.mesh);
      const playerBox = new THREE.Box3().setFromCenterAndSize(
        pos,
        new THREE.Vector3(0.8, 1.6, 0.8)
      );

      if (playerBox.intersectsBox(bounds)) {
        // Push player up (landing on top)
        if (vel.y < 0 && pos.y > platform.mesh.position.y) {
          pos.y = bounds.max.y + 0.9;
          vel.y = 0;
          this.player.onGround = true;
        }
      }
    });
  }

  updateCamera() {
    const targetPos = this.player.position.clone().add(this.cameraOffset);
    this.camera.position.lerp(targetPos, 0.1);
    this.cameraTarget.lerp(this.player.position, 0.1);
    this.camera.lookAt(this.cameraTarget);
  }

  updateCoins(delta) {
    this.coins.forEach(coin => {
      coin.rotation.z += delta * 2;
      coin.position.y += Math.sin(Date.now() * 0.003) * 0.002;
    });
  }

  updateEnemies(delta) {
    this.enemies.forEach(enemy => {
      enemy.position.x += Math.cos(enemy.direction) * enemy.speed * delta;
      enemy.position.z += Math.sin(enemy.direction) * enemy.speed * delta;

      if (Math.abs(enemy.position.x) > 20 || Math.abs(enemy.position.z) > 20) {
        enemy.direction += Math.PI;
      }

      enemy.rotation.y = enemy.direction;
    });
  }

  checkCollisions() {
    const playerPos = this.player.position;

    // Coin collection
    this.coins = this.coins.filter(coin => {
      const dist = playerPos.distanceTo(coin.position);
      if (dist < 1) {
        this.scene.remove(coin);
        this.score += 10;
        console.log('Score:', this.score);
        return false;
      }
      return true;
    });

    // Enemy collision
    this.enemies.forEach(enemy => {
      const dist = playerPos.distanceTo(enemy.position);
      if (dist < 1.2) {
        console.log('Game Over!');
        this.player.position.set(0, 2, 0);
        this.player.velocity.set(0, 0, 0);
      }
    });
  }
}
```

---

## Step 1904-1910: WebGL Basics

### WebGL พื้นฐาน

```javascript
// WebGL Fundamentals
class WebGLRenderer {
  constructor(canvas) {
    this.gl = canvas.getContext('webgl2');
    if (!this.gl) {
      throw new Error('WebGL2 not supported');
    }
    this.programs = new Map();
    this.buffers = new Map();
    this.textures = new Map();
  }

  // Shader compilation
  createShader(type, source) {
    const gl = this.gl;
    const shader = gl.createShader(type);
    gl.shaderSource(shader, source);
    gl.compileShader(shader);

    if (!gl.getShaderParameter(shader, gl.COMPILE_STATUS)) {
      const error = gl.getShaderInfoLog(shader);
      gl.deleteShader(shader);
      throw new Error(`Shader compilation failed: ${error}`);
    }

    return shader;
  }

  createProgram(vertexSource, fragmentSource) {
    const gl = this.gl;
    const vertexShader = this.createShader(gl.VERTEX_SHADER, vertexSource);
    const fragmentShader = this.createShader(gl.FRAGMENT_SHADER, fragmentSource);

    const program = gl.createProgram();
    gl.attachShader(program, vertexShader);
    gl.attachShader(program, fragmentShader);
    gl.linkProgram(program);

    if (!gl.getProgramParameter(program, gl.LINK_STATUS)) {
      const error = gl.getProgramInfoLog(program);
      throw new Error(`Program linking failed: ${error}`);
    }

    return program;
  }
}

// Simple WebGL Triangle
const vertexShaderSource = `
  attribute vec2 a_position;
  attribute vec3 a_color;
  
  uniform mat3 u_matrix;
  
  varying vec3 v_color;
  
  void main() {
    vec3 position = u_matrix * vec3(a_position, 1.0);
    gl_Position = vec4(position.xy, 0, 1);
    v_color = a_color;
  }
`;

const fragmentShaderSource = `
  precision mediump float;
  
  varying vec3 v_color;
  
  void main() {
    gl_FragColor = vec4(v_color, 1.0);
  }
`;

function drawTriangle(gl) {
  const program = createProgram(gl, vertexShaderSource, fragmentShaderSource);
  gl.useProgram(program);

  const positions = new Float32Array([
    0.0,  0.5,  // top
   -0.5, -0.5,  // bottom-left
    0.5, -0.5   // bottom-right
  ]);

  const colors = new Float32Array([
    1.0, 0.0, 0.0,  // red
    0.0, 1.0, 0.0,  // green
    0.0, 0.0, 1.0   // blue
  ]);

  // Position buffer
  const posBuffer = gl.createBuffer();
  gl.bindBuffer(gl.ARRAY_BUFFER, posBuffer);
  gl.bufferData(gl.ARRAY_BUFFER, positions, gl.STATIC_DRAW);

  const posLocation = gl.getAttribLocation(program, 'a_position');
  gl.enableVertexAttribArray(posLocation);
  gl.vertexAttribPointer(posLocation, 2, gl.FLOAT, false, 0, 0);

  // Color buffer
  const colorBuffer = gl.createBuffer();
  gl.bindBuffer(gl.ARRAY_BUFFER, colorBuffer);
  gl.bufferData(gl.ARRAY_BUFFER, colors, gl.STATIC_DRAW);

  const colorLocation = gl.getAttribLocation(program, 'a_color');
  gl.enableVertexAttribArray(colorLocation);
  gl.vertexAttribPointer(colorLocation, 3, gl.FLOAT, false, 0, 0);

  // Draw
  gl.viewport(0, 0, gl.canvas.width, gl.canvas.height);
  gl.clearColor(0, 0, 0, 1);
  gl.clear(gl.COLOR_BUFFER_BIT);
  gl.drawArrays(gl.TRIANGLES, 0, 3);
}
```

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: Breakout Game
สร้างเกม Breakout ด้วย Canvas API:
- Ball ที่เด้งไปมา
- Paddle ที่ควบคุมได้
- Bricks หลายแถว
- Score และ Lives
- Power-ups

### แบบฝึกหัดที่ 2: Tower Defense
สร้างเกม Tower Defense:
- Grid-based map
- Enemy path finding (A*)
- Tower ที่ place ได้
- Upgrade system
- Wave system

### แบบฝึกหัดที่ 3: 3D Space Shooter
สร้างเกม 3D Space Shooter ด้วย Three.js:
- Player spaceship ที่ควบคุมได้
- Enemy spaceships
- Laser bullets
- Explosions ด้วย particles
- Score system

### แบบฝึกหัดที่ 4: ECS Roguelike
สร้าง Roguelike game โดยใช้ ECS pattern:
- Procedural dungeon generation
- Turn-based combat
- Item system
- Status effects

---

## สรุป

ในบทนี้เราได้เรียนรู้:

1. **Game Loop** และ Delta Time สำหรับการอัปเดตที่สม่ำเสมอ
2. **Canvas API** สำหรับการวาด 2D graphics
3. **Input Handling** ทั้ง keyboard, mouse, touch และ gamepad
4. **Particle System** สำหรับ visual effects
5. **Phaser.js** framework สำหรับสร้างเกม 2D
6. **Arcade Physics** และ **Matter.js** สำหรับ physics
7. **Tilemap System** สำหรับสร้าง level
8. **Collision Detection** ทั้งแบบ AABB, Circle, SAT
9. **Entity Component System** สำหรับ architecture ที่ยืดหยุ่น
10. **Three.js** สำหรับ 3D graphics
11. **WebGL** พื้นฐาน

การพัฒนาเกมเป็นสาขาที่ต้องการความรู้หลากหลายด้าน ตั้งแต่ mathematics ไปจนถึง graphics programming แต่ JavaScript มีเครื่องมือที่ดีมากที่ทำให้การสร้างเกมเป็นเรื่องสนุกและเข้าถึงได้
