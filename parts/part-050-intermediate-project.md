# Part 50: โปรเจค Web App ระดับกลาง

## 5 โปรเจคสำหรับฝึก JavaScript ขั้นกลาง

Part นี้นำทักษะที่เรียนมาทั้งหมดมาประยุกต์ใช้สร้าง web applications จริง

**Steps 971-990**

---

## Step 971-974: โปรเจค 1 - Real-time Chat App

```html
<!DOCTYPE html>
<html lang="th">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Chat App</title>
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; }
    
    body {
      font-family: 'Sarabun', 'Segoe UI', sans-serif;
      background: #f0f2f5;
      height: 100vh;
      display: flex;
      align-items: center;
      justify-content: center;
    }
    
    .chat-container {
      width: 400px;
      height: 600px;
      background: white;
      border-radius: 16px;
      box-shadow: 0 8px 30px rgba(0,0,0,0.12);
      display: flex;
      flex-direction: column;
      overflow: hidden;
    }
    
    .chat-header {
      background: linear-gradient(135deg, #667eea, #764ba2);
      color: white;
      padding: 16px 20px;
      display: flex;
      align-items: center;
      gap: 12px;
    }
    
    .chat-avatar {
      width: 40px;
      height: 40px;
      border-radius: 50%;
      background: rgba(255,255,255,0.3);
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 20px;
    }
    
    .chat-info h3 { font-size: 16px; }
    .chat-info p { font-size: 12px; opacity: 0.8; }
    
    .online-dot {
      width: 8px;
      height: 8px;
      background: #4ade80;
      border-radius: 50%;
      margin-left: auto;
      animation: pulse 2s infinite;
    }
    
    @keyframes pulse {
      0%, 100% { opacity: 1; transform: scale(1); }
      50% { opacity: 0.7; transform: scale(1.2); }
    }
    
    .messages {
      flex: 1;
      overflow-y: auto;
      padding: 16px;
      display: flex;
      flex-direction: column;
      gap: 12px;
      scroll-behavior: smooth;
    }
    
    .messages::-webkit-scrollbar { width: 4px; }
    .messages::-webkit-scrollbar-thumb { background: #ddd; border-radius: 2px; }
    
    .message {
      display: flex;
      gap: 8px;
      animation: messageIn 0.3s ease;
    }
    
    @keyframes messageIn {
      from { opacity: 0; transform: translateY(10px); }
      to { opacity: 1; transform: none; }
    }
    
    .message.own {
      flex-direction: row-reverse;
    }
    
    .message-avatar {
      width: 32px;
      height: 32px;
      border-radius: 50%;
      background: #e9ecef;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 14px;
      flex-shrink: 0;
    }
    
    .message-content {
      max-width: 70%;
    }
    
    .message-name {
      font-size: 11px;
      color: #666;
      margin-bottom: 4px;
      padding: 0 4px;
    }
    
    .message.own .message-name {
      text-align: right;
    }
    
    .bubble {
      padding: 10px 14px;
      border-radius: 18px;
      font-size: 14px;
      line-height: 1.5;
      position: relative;
    }
    
    .message:not(.own) .bubble {
      background: #f0f2f5;
      color: #333;
      border-bottom-left-radius: 4px;
    }
    
    .message.own .bubble {
      background: linear-gradient(135deg, #667eea, #764ba2);
      color: white;
      border-bottom-right-radius: 4px;
    }
    
    .message-time {
      font-size: 10px;
      color: #999;
      margin-top: 4px;
      padding: 0 4px;
      text-align: right;
    }
    
    .message:not(.own) .message-time {
      text-align: left;
    }
    
    .message-actions {
      display: none;
      position: absolute;
      top: 50%;
      transform: translateY(-50%);
      right: -30px;
    }
    
    .message.own .message-actions {
      right: auto;
      left: -30px;
    }
    
    .bubble:hover .message-actions { display: flex; }
    
    .delete-btn {
      background: none;
      border: none;
      cursor: pointer;
      font-size: 14px;
      opacity: 0.6;
      padding: 2px;
    }
    
    .delete-btn:hover { opacity: 1; }
    
    .system-message {
      text-align: center;
      color: #999;
      font-size: 12px;
      padding: 4px 12px;
      background: #f8f9fa;
      border-radius: 12px;
      align-self: center;
    }
    
    .typing-indicator {
      display: none;
      align-items: center;
      gap: 8px;
      padding: 8px 16px;
    }
    
    .typing-indicator.active { display: flex; }
    
    .typing-dots {
      display: flex;
      gap: 3px;
    }
    
    .typing-dots span {
      width: 6px;
      height: 6px;
      background: #adb5bd;
      border-radius: 50%;
      animation: typingDot 1.4s infinite;
    }
    
    .typing-dots span:nth-child(2) { animation-delay: 0.2s; }
    .typing-dots span:nth-child(3) { animation-delay: 0.4s; }
    
    @keyframes typingDot {
      0%, 100% { transform: translateY(0); }
      50% { transform: translateY(-4px); }
    }
    
    .chat-input-area {
      padding: 16px;
      border-top: 1px solid #f0f2f5;
      display: flex;
      gap: 8px;
      align-items: flex-end;
    }
    
    .input-wrapper {
      flex: 1;
      background: #f0f2f5;
      border-radius: 20px;
      padding: 8px 16px;
      display: flex;
      align-items: center;
      gap: 8px;
    }
    
    #message-input {
      flex: 1;
      border: none;
      background: transparent;
      outline: none;
      font-size: 14px;
      font-family: inherit;
      resize: none;
      max-height: 100px;
    }
    
    .emoji-btn {
      background: none;
      border: none;
      font-size: 18px;
      cursor: pointer;
      opacity: 0.7;
    }
    
    .emoji-btn:hover { opacity: 1; }
    
    .send-btn {
      width: 40px;
      height: 40px;
      border-radius: 50%;
      background: linear-gradient(135deg, #667eea, #764ba2);
      color: white;
      border: none;
      cursor: pointer;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 18px;
      transition: transform 0.2s;
    }
    
    .send-btn:hover { transform: scale(1.1); }
    .send-btn:disabled { opacity: 0.5; cursor: not-allowed; transform: none; }
    
    .emoji-picker {
      display: none;
      position: absolute;
      bottom: 80px;
      right: 16px;
      background: white;
      border-radius: 12px;
      box-shadow: 0 4px 20px rgba(0,0,0,0.15);
      padding: 12px;
      width: 260px;
      flex-wrap: wrap;
      gap: 4px;
      z-index: 10;
    }
    
    .emoji-picker.show { display: flex; }
    
    .emoji-picker span {
      font-size: 22px;
      cursor: pointer;
      padding: 4px;
      border-radius: 6px;
      transition: background 0.1s;
    }
    
    .emoji-picker span:hover { background: #f0f2f5; }
  </style>
</head>
<body>

<div class="chat-container">
  <div class="chat-header">
    <div class="chat-avatar">🤖</div>
    <div class="chat-info">
      <h3>ChatBot สมาร์ท</h3>
      <p>กำลังออนไลน์</p>
    </div>
    <div class="online-dot"></div>
  </div>
  
  <div class="messages" id="messages">
    <div class="system-message">วันนี้ 10:00 น.</div>
  </div>
  
  <div class="typing-indicator" id="typing">
    <div class="message-avatar">🤖</div>
    <div class="typing-dots">
      <span></span><span></span><span></span>
    </div>
    <span style="font-size:12px;color:#999">กำลังพิมพ์...</span>
  </div>
  
  <div class="chat-input-area" style="position:relative">
    <div class="emoji-picker" id="emoji-picker"></div>
    
    <div class="input-wrapper">
      <textarea id="message-input" rows="1" placeholder="พิมพ์ข้อความ..."></textarea>
      <button class="emoji-btn" id="emoji-btn">😊</button>
    </div>
    
    <button class="send-btn" id="send-btn">➤</button>
  </div>
</div>

<script>
// Chat App JavaScript
const STORAGE_KEY = "chat_messages";
const CURRENT_USER = { name: "คุณ", avatar: "👤" };
const BOT_USER = { name: "ChatBot", avatar: "🤖" };

const messagesEl = document.getElementById("messages");
const messageInput = document.getElementById("message-input");
const sendBtn = document.getElementById("send-btn");
const typingIndicator = document.getElementById("typing");
const emojiBtn = document.getElementById("emoji-btn");
const emojiPicker = document.getElementById("emoji-picker");

// โหลด messages จาก localStorage
let messages = loadMessages();

// Bot responses ภาษาไทย
const botResponses = [
  "สวัสดีครับ! มีอะไรให้ช่วยไหมครับ? 😊",
  "เข้าใจแล้วครับ ขอบคุณสำหรับข้อมูล",
  "นั่นเป็นคำถามที่น่าสนใจมากเลยนะครับ",
  "ผมกำลังประมวลผลข้อมูลอยู่ครับ รอสักครู่นะครับ",
  "ขอบคุณที่แจ้งให้ทราบครับ!",
  "คุณต้องการความช่วยเหลืออะไรเพิ่มเติมไหมครับ?",
  "ยอดเยี่ยมมาก! 👍",
  "ผมเข้าใจสิ่งที่คุณพูดแล้วครับ",
  "โอเคครับ ผมจะดำเนินการให้เลยครับ",
  "หากมีข้อสงสัยเพิ่มเติม สอบถามได้เลยนะครับ 😄"
];

// Emojis สำหรับ picker
const EMOJIS = ["😊","😂","🥰","😎","🤔","😢","😡","🎉","👍","👋",
                 "❤️","🔥","⭐","🌟","💪","🙏","🎊","🤝","😀","🤗"];

// Initialize
function init() {
  renderMessages();
  setupEmojiPicker();
  setupAutoResize();
}

function loadMessages() {
  try {
    return JSON.parse(localStorage.getItem(STORAGE_KEY)) || getDefaultMessages();
  } catch {
    return getDefaultMessages();
  }
}

function saveMessages() {
  try {
    localStorage.setItem(STORAGE_KEY, JSON.stringify(messages));
  } catch (e) {
    console.warn("ไม่สามารถบันทึก messages:", e);
  }
}

function getDefaultMessages() {
  return [
    {
      id: 1,
      type: "received",
      user: BOT_USER,
      text: "สวัสดีครับ! ยินดีต้อนรับสู่ Chat App ✨ มีอะไรให้ช่วยไหมครับ?",
      time: new Date(Date.now() - 300000)
    }
  ];
}

function renderMessages() {
  // เก็บ scroll position
  const wasAtBottom = messagesEl.scrollHeight - messagesEl.scrollTop <= messagesEl.clientHeight + 50;
  
  // ล้าง messages
  messagesEl.innerHTML = "";
  
  let lastDate = null;
  
  messages.forEach(msg => {
    const msgDate = new Date(msg.time).toDateString();
    
    if (msgDate !== lastDate) {
      const dateEl = document.createElement("div");
      dateEl.className = "system-message";
      dateEl.textContent = formatDate(new Date(msg.time));
      messagesEl.appendChild(dateEl);
      lastDate = msgDate;
    }
    
    messagesEl.appendChild(createMessageEl(msg));
  });
  
  if (wasAtBottom) {
    scrollToBottom();
  }
}

function createMessageEl(msg) {
  const div = document.createElement("div");
  div.className = `message ${msg.type === "sent" ? "own" : ""}`;
  div.dataset.id = msg.id;
  
  div.innerHTML = `
    <div class="message-avatar">${msg.user.avatar}</div>
    <div class="message-content">
      <div class="message-name">${msg.user.name}</div>
      <div class="bubble">
        ${escapeHTML(msg.text)}
        <div class="message-actions">
          <button class="delete-btn" onclick="deleteMessage(${msg.id})" title="ลบ">🗑️</button>
        </div>
      </div>
      <div class="message-time">${formatTime(new Date(msg.time))}</div>
    </div>
  `;
  
  return div;
}

function sendMessage() {
  const text = messageInput.value.trim();
  if (!text) return;
  
  const msg = {
    id: Date.now(),
    type: "sent",
    user: CURRENT_USER,
    text,
    time: new Date()
  };
  
  messages.push(msg);
  saveMessages();
  
  messageInput.value = "";
  messageInput.style.height = "auto";
  
  renderMessages();
  scrollToBottom();
  
  // Bot responds
  simulateBotResponse();
}

function simulateBotResponse() {
  typingIndicator.classList.add("active");
  scrollToBottom();
  
  const delay = 1000 + Math.random() * 2000;
  
  setTimeout(() => {
    typingIndicator.classList.remove("active");
    
    const response = botResponses[Math.floor(Math.random() * botResponses.length)];
    
    const msg = {
      id: Date.now(),
      type: "received",
      user: BOT_USER,
      text: response,
      time: new Date()
    };
    
    messages.push(msg);
    saveMessages();
    renderMessages();
    scrollToBottom();
  }, delay);
}

function deleteMessage(id) {
  messages = messages.filter(m => m.id !== id);
  saveMessages();
  
  const el = document.querySelector(`[data-id="${id}"]`);
  if (el) {
    el.style.animation = "none";
    el.style.transition = "all 0.3s";
    el.style.opacity = "0";
    el.style.transform = "translateX(100px)";
    
    setTimeout(() => {
      el.remove();
    }, 300);
  }
}

function setupEmojiPicker() {
  EMOJIS.forEach(emoji => {
    const span = document.createElement("span");
    span.textContent = emoji;
    span.addEventListener("click", () => {
      messageInput.value += emoji;
      messageInput.focus();
      emojiPicker.classList.remove("show");
    });
    emojiPicker.appendChild(span);
  });
}

function setupAutoResize() {
  messageInput.addEventListener("input", () => {
    messageInput.style.height = "auto";
    messageInput.style.height = Math.min(messageInput.scrollHeight, 100) + "px";
  });
}

function scrollToBottom() {
  requestAnimationFrame(() => {
    messagesEl.scrollTop = messagesEl.scrollHeight;
  });
}

function formatTime(date) {
  return date.toLocaleTimeString("th-TH", { hour: "2-digit", minute: "2-digit" });
}

function formatDate(date) {
  const today = new Date();
  const yesterday = new Date(today - 86400000);
  
  if (date.toDateString() === today.toDateString()) return "วันนี้";
  if (date.toDateString() === yesterday.toDateString()) return "เมื่อวาน";
  return date.toLocaleDateString("th-TH", { day: "numeric", month: "long", year: "numeric" });
}

function escapeHTML(text) {
  const div = document.createElement("div");
  div.textContent = text;
  return div.innerHTML;
}

// Events
sendBtn.addEventListener("click", sendMessage);

messageInput.addEventListener("keydown", (e) => {
  if (e.key === "Enter" && !e.shiftKey) {
    e.preventDefault();
    sendMessage();
  }
});

emojiBtn.addEventListener("click", (e) => {
  e.stopPropagation();
  emojiPicker.classList.toggle("show");
});

document.addEventListener("click", () => {
  emojiPicker.classList.remove("show");
});

// Init
init();
</script>
</body>
</html>
```

---

## Step 975-978: โปรเจค 2 - Kanban Board

```html
<!DOCTYPE html>
<html lang="th">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Kanban Board</title>
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; }
    
    body {
      font-family: 'Sarabun', sans-serif;
      background: linear-gradient(135deg, #1e3c72, #2a5298);
      min-height: 100vh;
      padding: 20px;
    }
    
    header {
      color: white;
      text-align: center;
      margin-bottom: 24px;
    }
    
    header h1 { font-size: 28px; }
    header p { opacity: 0.7; margin-top: 4px; }
    
    .board {
      display: flex;
      gap: 16px;
      overflow-x: auto;
      padding-bottom: 20px;
    }
    
    .column {
      min-width: 280px;
      width: 280px;
      background: rgba(255,255,255,0.1);
      border-radius: 12px;
      backdrop-filter: blur(10px);
      display: flex;
      flex-direction: column;
      max-height: calc(100vh - 120px);
    }
    
    .column-header {
      padding: 16px;
      display: flex;
      align-items: center;
      justify-content: space-between;
      cursor: pointer;
    }
    
    .column-title-area {
      display: flex;
      align-items: center;
      gap: 8px;
    }
    
    .column-dot {
      width: 12px;
      height: 12px;
      border-radius: 50%;
    }
    
    .column-title {
      color: white;
      font-size: 15px;
      font-weight: 600;
    }
    
    .column-count {
      background: rgba(255,255,255,0.2);
      color: white;
      font-size: 11px;
      padding: 2px 8px;
      border-radius: 12px;
    }
    
    .column-add-btn {
      background: rgba(255,255,255,0.15);
      border: none;
      color: white;
      width: 28px;
      height: 28px;
      border-radius: 6px;
      font-size: 18px;
      cursor: pointer;
      display: flex;
      align-items: center;
      justify-content: center;
      transition: background 0.2s;
    }
    
    .column-add-btn:hover { background: rgba(255,255,255,0.25); }
    
    .cards-container {
      flex: 1;
      overflow-y: auto;
      padding: 0 12px 12px;
      display: flex;
      flex-direction: column;
      gap: 8px;
      min-height: 80px;
    }
    
    .cards-container.drag-over {
      background: rgba(255,255,255,0.05);
      border-radius: 8px;
    }
    
    .card {
      background: white;
      border-radius: 10px;
      padding: 14px;
      cursor: grab;
      transition: transform 0.2s, box-shadow 0.2s;
      box-shadow: 0 2px 8px rgba(0,0,0,0.1);
      position: relative;
    }
    
    .card:hover {
      transform: translateY(-2px);
      box-shadow: 0 6px 20px rgba(0,0,0,0.2);
    }
    
    .card.dragging {
      opacity: 0.5;
      cursor: grabbing;
    }
    
    .card-label {
      display: inline-block;
      font-size: 11px;
      padding: 2px 8px;
      border-radius: 10px;
      margin-bottom: 8px;
      font-weight: 600;
      text-transform: uppercase;
      letter-spacing: 0.5px;
    }
    
    .card-title {
      font-size: 14px;
      color: #333;
      margin-bottom: 8px;
      line-height: 1.4;
    }
    
    .card-description {
      font-size: 12px;
      color: #777;
      line-height: 1.5;
      margin-bottom: 10px;
    }
    
    .card-meta {
      display: flex;
      align-items: center;
      justify-content: space-between;
    }
    
    .card-assignee {
      display: flex;
      align-items: center;
      gap: 4px;
      font-size: 11px;
      color: #999;
    }
    
    .card-due {
      font-size: 11px;
      color: #999;
      display: flex;
      align-items: center;
      gap: 3px;
    }
    
    .card-due.overdue { color: #dc3545; }
    
    .card-actions {
      position: absolute;
      top: 8px;
      right: 8px;
      display: none;
      gap: 4px;
    }
    
    .card:hover .card-actions { display: flex; }
    
    .card-action-btn {
      background: white;
      border: 1px solid #eee;
      border-radius: 4px;
      cursor: pointer;
      font-size: 12px;
      padding: 2px 6px;
      transition: background 0.1s;
    }
    
    .card-action-btn:hover { background: #f8f9fa; }
    
    .add-card-form {
      margin: 0 12px 12px;
      display: none;
    }
    
    .add-card-form.show { display: block; }
    
    .add-card-input {
      width: 100%;
      padding: 10px 12px;
      border: none;
      border-radius: 8px;
      font-family: inherit;
      font-size: 13px;
      outline: none;
      resize: none;
      background: white;
      box-shadow: 0 2px 8px rgba(0,0,0,0.1);
    }
    
    .add-card-actions {
      display: flex;
      gap: 6px;
      margin-top: 6px;
    }
    
    .btn {
      padding: 7px 14px;
      border-radius: 6px;
      border: none;
      cursor: pointer;
      font-size: 13px;
      font-family: inherit;
      transition: background 0.2s;
    }
    
    .btn-primary { background: #007bff; color: white; }
    .btn-primary:hover { background: #0056b3; }
    .btn-ghost { background: rgba(255,255,255,0.1); color: white; }
    .btn-ghost:hover { background: rgba(255,255,255,0.2); }
    
    .placeholder-card {
      height: 60px;
      background: rgba(255,255,255,0.1);
      border: 2px dashed rgba(255,255,255,0.3);
      border-radius: 10px;
    }
    
    .add-column-btn {
      min-width: 200px;
      background: rgba(255,255,255,0.1);
      border: 2px dashed rgba(255,255,255,0.3);
      border-radius: 12px;
      color: white;
      font-size: 15px;
      cursor: pointer;
      padding: 16px;
      transition: background 0.2s;
      align-self: flex-start;
      display: flex;
      align-items: center;
      gap: 8px;
    }
    
    .add-column-btn:hover { background: rgba(255,255,255,0.15); }
    
    .modal-overlay {
      display: none;
      position: fixed;
      inset: 0;
      background: rgba(0,0,0,0.5);
      z-index: 100;
      align-items: center;
      justify-content: center;
    }
    
    .modal-overlay.show { display: flex; }
    
    .modal-card {
      background: white;
      border-radius: 16px;
      padding: 24px;
      width: 500px;
      max-width: 90vw;
    }
    
    .modal-card h3 { margin-bottom: 16px; }
    
    .form-group { margin-bottom: 14px; }
    
    .form-group label {
      display: block;
      font-size: 13px;
      color: #555;
      margin-bottom: 6px;
      font-weight: 600;
    }
    
    .form-control {
      width: 100%;
      padding: 8px 12px;
      border: 1px solid #ddd;
      border-radius: 8px;
      font-family: inherit;
      font-size: 13px;
      outline: none;
    }
    
    .form-control:focus { border-color: #007bff; }
    
    .label-options {
      display: flex;
      gap: 6px;
      flex-wrap: wrap;
    }
    
    .label-option {
      padding: 4px 12px;
      border-radius: 12px;
      cursor: pointer;
      font-size: 12px;
      font-weight: 600;
      border: 2px solid transparent;
      transition: border-color 0.2s;
    }
    
    .label-option.selected {
      border-color: #333;
    }
  </style>
</head>
<body>

<header>
  <h1>🗂️ Kanban Board</h1>
  <p>จัดการงานของคุณอย่างมีประสิทธิภาพ</p>
</header>

<div class="board" id="board">
  <!-- Columns จะถูก generate โดย JavaScript -->
  <button class="add-column-btn" id="add-column-btn">
    <span style="font-size:20px">+</span> เพิ่มคอลัมน์
  </button>
</div>

<!-- Card Detail Modal -->
<div class="modal-overlay" id="card-modal">
  <div class="modal-card">
    <h3 id="modal-title">เพิ่มการ์ดใหม่</h3>
    
    <div class="form-group">
      <label>ชื่องาน *</label>
      <input type="text" class="form-control" id="card-title-input" placeholder="ชื่องาน">
    </div>
    
    <div class="form-group">
      <label>รายละเอียด</label>
      <textarea class="form-control" id="card-desc-input" rows="3" placeholder="รายละเอียด (ถ้ามี)"></textarea>
    </div>
    
    <div class="form-group">
      <label>ป้ายกำกับ</label>
      <div class="label-options">
        <span class="label-option selected" data-label="bug" style="background:#ffebeb;color:#dc3545">Bug</span>
        <span class="label-option" data-label="feature" style="background:#e8f4fd;color:#0077cc">Feature</span>
        <span class="label-option" data-label="design" style="background:#f3e8ff;color:#7c3aed">Design</span>
        <span class="label-option" data-label="docs" style="background:#e8fff0;color:#059669">Docs</span>
        <span class="label-option" data-label="test" style="background:#fff3e0;color:#f57c00">Test</span>
      </div>
    </div>
    
    <div class="form-group">
      <label>ผู้รับผิดชอบ</label>
      <input type="text" class="form-control" id="card-assignee-input" placeholder="ชื่อผู้รับผิดชอบ">
    </div>
    
    <div class="form-group">
      <label>กำหนดส่ง</label>
      <input type="date" class="form-control" id="card-due-input">
    </div>
    
    <div style="display:flex;gap:8px;justify-content:flex-end">
      <button class="btn" id="modal-cancel" style="background:#f8f9fa">ยกเลิก</button>
      <button class="btn btn-primary" id="modal-save">บันทึก</button>
    </div>
  </div>
</div>

<script>
const BOARD_KEY = "kanban_board";

const LABEL_CONFIG = {
  bug: { text: "Bug", bg: "#ffebeb", color: "#dc3545" },
  feature: { text: "Feature", bg: "#e8f4fd", color: "#0077cc" },
  design: { text: "Design", bg: "#f3e8ff", color: "#7c3aed" },
  docs: { text: "Docs", bg: "#e8fff0", color: "#059669" },
  test: { text: "Test", bg: "#fff3e0", color: "#f57c00" }
};

const COLUMN_COLORS = ["#ef4444","#f59e0b","#10b981","#3b82f6","#8b5cf6","#ec4899"];

let boardData = loadBoard();
let draggedCard = null;
let dragSourceColumn = null;
let currentColumnId = null;
let editingCardId = null;
let selectedLabel = "bug";

function loadBoard() {
  try {
    return JSON.parse(localStorage.getItem(BOARD_KEY)) || getDefaultBoard();
  } catch {
    return getDefaultBoard();
  }
}

function getDefaultBoard() {
  return {
    columns: [
      {
        id: "backlog",
        title: "📋 Backlog",
        color: "#6c757d",
        cards: [
          { id: "c1", title: "ออกแบบ UI หน้าหลัก", description: "สร้าง mockup และ prototype", label: "design", assignee: "สมชาย", due: "2026-10-15" },
          { id: "c2", title: "เขียน test สำหรับ API", description: "Unit tests และ integration tests", label: "test", assignee: "สมหญิง", due: "2026-10-20" }
        ]
      },
      {
        id: "todo",
        title: "📝 Todo",
        color: "#0077cc",
        cards: [
          { id: "c3", title: "ติดตั้ง database", description: "ตั้งค่า PostgreSQL", label: "feature", assignee: "ประยุทธ์", due: "2026-10-10" }
        ]
      },
      {
        id: "doing",
        title: "⚡ In Progress",
        color: "#f59e0b",
        cards: [
          { id: "c4", title: "แก้ไข bug หน้า login", description: "Session ขาดหายบางครั้ง", label: "bug", assignee: "สมชาย", due: "2026-10-05" }
        ]
      },
      {
        id: "done",
        title: "✅ Done",
        color: "#10b981",
        cards: [
          { id: "c5", title: "สร้าง project structure", description: "จัดโครงสร้างโปรเจค", label: "docs", assignee: "ทีม", due: "2026-09-28" }
        ]
      }
    ]
  };
}

function saveBoard() {
  try {
    localStorage.setItem(BOARD_KEY, JSON.stringify(boardData));
  } catch (e) {}
}

function render() {
  const board = document.getElementById("board");
  const addBtn = document.getElementById("add-column-btn");
  
  board.innerHTML = "";
  
  boardData.columns.forEach(col => {
    board.appendChild(createColumnEl(col));
  });
  
  board.appendChild(addBtn);
}

function createColumnEl(col) {
  const el = document.createElement("div");
  el.className = "column";
  el.dataset.id = col.id;
  
  el.innerHTML = `
    <div class="column-header">
      <div class="column-title-area">
        <span class="column-dot" style="background:${col.color}"></span>
        <span class="column-title">${col.title}</span>
        <span class="column-count">${col.cards.length}</span>
      </div>
      <button class="column-add-btn" data-col="${col.id}">+</button>
    </div>
    <div class="cards-container" data-col="${col.id}"></div>
    <div class="add-card-form" id="form-${col.id}">
      <textarea class="add-card-input" rows="2" placeholder="ชื่องาน..." id="quick-input-${col.id}"></textarea>
      <div class="add-card-actions">
        <button class="btn btn-primary" onclick="quickAddCard('${col.id}')">เพิ่ม</button>
        <button class="btn btn-ghost" onclick="hideAddForm('${col.id}')">ยกเลิก</button>
      </div>
    </div>
  `;
  
  const container = el.querySelector(".cards-container");
  
  col.cards.forEach(card => {
    container.appendChild(createCardEl(card, col.id));
  });
  
  setupDropZone(container, col.id);
  
  el.querySelector(".column-add-btn").addEventListener("click", () => {
    showAddCardModal(col.id);
  });
  
  return el;
}

function createCardEl(card, colId) {
  const el = document.createElement("div");
  el.className = "card";
  el.draggable = true;
  el.dataset.id = card.id;
  el.dataset.col = colId;
  
  const label = LABEL_CONFIG[card.label] || LABEL_CONFIG.feature;
  const isOverdue = card.due && new Date(card.due) < new Date();
  
  el.innerHTML = `
    <div class="card-actions">
      <button class="card-action-btn" onclick="editCard('${card.id}','${colId}')">✏️</button>
      <button class="card-action-btn" onclick="deleteCard('${card.id}','${colId}')">🗑️</button>
    </div>
    <span class="card-label" style="background:${label.bg};color:${label.color}">${label.text}</span>
    <div class="card-title">${escapeHTML(card.title)}</div>
    ${card.description ? `<div class="card-description">${escapeHTML(card.description)}</div>` : ""}
    <div class="card-meta">
      <span class="card-assignee">👤 ${card.assignee || "ไม่ระบุ"}</span>
      ${card.due ? `<span class="card-due ${isOverdue ? "overdue" : ""}">📅 ${formatDue(card.due)}</span>` : ""}
    </div>
  `;
  
  // Drag events
  el.addEventListener("dragstart", (e) => {
    draggedCard = card;
    dragSourceColumn = colId;
    el.classList.add("dragging");
    e.dataTransfer.effectAllowed = "move";
  });
  
  el.addEventListener("dragend", () => {
    el.classList.remove("dragging");
    draggedCard = null;
    dragSourceColumn = null;
  });
  
  return el;
}

function setupDropZone(container, colId) {
  container.addEventListener("dragover", (e) => {
    e.preventDefault();
    e.dataTransfer.dropEffect = "move";
    container.classList.add("drag-over");
  });
  
  container.addEventListener("dragleave", (e) => {
    if (!container.contains(e.relatedTarget)) {
      container.classList.remove("drag-over");
    }
  });
  
  container.addEventListener("drop", (e) => {
    e.preventDefault();
    container.classList.remove("drag-over");
    
    if (!draggedCard || dragSourceColumn === colId) return;
    
    // ย้าย card
    const sourceCol = boardData.columns.find(c => c.id === dragSourceColumn);
    const targetCol = boardData.columns.find(c => c.id === colId);
    
    if (!sourceCol || !targetCol) return;
    
    sourceCol.cards = sourceCol.cards.filter(c => c.id !== draggedCard.id);
    targetCol.cards.push(draggedCard);
    
    saveBoard();
    render();
  });
}

function showAddCardModal(colId) {
  currentColumnId = colId;
  editingCardId = null;
  
  document.getElementById("modal-title").textContent = "เพิ่มการ์ดใหม่";
  document.getElementById("card-title-input").value = "";
  document.getElementById("card-desc-input").value = "";
  document.getElementById("card-assignee-input").value = "";
  document.getElementById("card-due-input").value = "";
  
  selectLabel("bug");
  document.getElementById("card-modal").classList.add("show");
  document.getElementById("card-title-input").focus();
}

function editCard(cardId, colId) {
  const col = boardData.columns.find(c => c.id === colId);
  const card = col?.cards.find(c => c.id === cardId);
  if (!card) return;
  
  currentColumnId = colId;
  editingCardId = cardId;
  
  document.getElementById("modal-title").textContent = "แก้ไขการ์ด";
  document.getElementById("card-title-input").value = card.title;
  document.getElementById("card-desc-input").value = card.description || "";
  document.getElementById("card-assignee-input").value = card.assignee || "";
  document.getElementById("card-due-input").value = card.due || "";
  
  selectLabel(card.label || "feature");
  document.getElementById("card-modal").classList.add("show");
}

function deleteCard(cardId, colId) {
  if (!confirm("ต้องการลบการ์ดนี้?")) return;
  
  const col = boardData.columns.find(c => c.id === colId);
  if (col) {
    col.cards = col.cards.filter(c => c.id !== cardId);
    saveBoard();
    render();
  }
}

function quickAddCard(colId) {
  const input = document.getElementById(`quick-input-${colId}`);
  const title = input.value.trim();
  if (!title) return;
  
  const col = boardData.columns.find(c => c.id === colId);
  if (col) {
    col.cards.push({
      id: "c" + Date.now(),
      title,
      label: "feature",
      assignee: "",
      due: ""
    });
    saveBoard();
    render();
  }
}

function hideAddForm(colId) {
  document.getElementById(`form-${colId}`).classList.remove("show");
}

function selectLabel(label) {
  selectedLabel = label;
  document.querySelectorAll(".label-option").forEach(opt => {
    opt.classList.toggle("selected", opt.dataset.label === label);
  });
}

function formatDue(dateStr) {
  const date = new Date(dateStr);
  return date.toLocaleDateString("th-TH", { month: "short", day: "numeric" });
}

function escapeHTML(text) {
  const div = document.createElement("div");
  div.textContent = text;
  return div.innerHTML;
}

// Label selection
document.querySelectorAll(".label-option").forEach(opt => {
  opt.addEventListener("click", () => selectLabel(opt.dataset.label));
});

// Modal events
document.getElementById("modal-cancel").addEventListener("click", () => {
  document.getElementById("card-modal").classList.remove("show");
});

document.getElementById("modal-save").addEventListener("click", () => {
  const title = document.getElementById("card-title-input").value.trim();
  if (!title) {
    document.getElementById("card-title-input").focus();
    return;
  }
  
  const cardData = {
    title,
    description: document.getElementById("card-desc-input").value.trim(),
    label: selectedLabel,
    assignee: document.getElementById("card-assignee-input").value.trim(),
    due: document.getElementById("card-due-input").value
  };
  
  const col = boardData.columns.find(c => c.id === currentColumnId);
  
  if (editingCardId) {
    const card = col.cards.find(c => c.id === editingCardId);
    if (card) Object.assign(card, cardData);
  } else {
    col.cards.push({ id: "c" + Date.now(), ...cardData });
  }
  
  saveBoard();
  render();
  document.getElementById("card-modal").classList.remove("show");
});

document.getElementById("add-column-btn").addEventListener("click", () => {
  const title = prompt("ชื่อคอลัมน์:");
  if (!title?.trim()) return;
  
  boardData.columns.push({
    id: "col" + Date.now(),
    title: title.trim(),
    color: COLUMN_COLORS[boardData.columns.length % COLUMN_COLORS.length],
    cards: []
  });
  
  saveBoard();
  render();
});

document.getElementById("card-modal").addEventListener("click", (e) => {
  if (e.target === document.getElementById("card-modal")) {
    document.getElementById("card-modal").classList.remove("show");
  }
});

// Init
render();
</script>
</body>
</html>
```

---

## Step 979-982: โปรเจค 3 - Image Gallery

```html
<!DOCTYPE html>
<html lang="th">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Image Gallery</title>
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; }
    
    body {
      font-family: 'Sarabun', sans-serif;
      background: #f8f9fa;
    }
    
    .gallery-header {
      background: white;
      padding: 20px;
      box-shadow: 0 2px 10px rgba(0,0,0,0.06);
      position: sticky;
      top: 0;
      z-index: 10;
    }
    
    .gallery-header h1 { font-size: 24px; color: #333; }
    
    .filter-bar {
      display: flex;
      gap: 8px;
      margin-top: 14px;
      flex-wrap: wrap;
    }
    
    .filter-btn {
      padding: 6px 16px;
      border: 2px solid #e9ecef;
      border-radius: 20px;
      background: white;
      cursor: pointer;
      font-size: 13px;
      transition: all 0.2s;
      font-family: inherit;
    }
    
    .filter-btn:hover { border-color: #007bff; color: #007bff; }
    
    .filter-btn.active {
      background: #007bff;
      border-color: #007bff;
      color: white;
    }
    
    .gallery-main { padding: 20px; }
    
    .gallery-grid {
      display: grid;
      grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
      gap: 16px;
    }
    
    .gallery-item {
      border-radius: 12px;
      overflow: hidden;
      background: white;
      box-shadow: 0 2px 8px rgba(0,0,0,0.08);
      cursor: pointer;
      transition: transform 0.2s, box-shadow 0.2s;
    }
    
    .gallery-item:hover {
      transform: translateY(-4px);
      box-shadow: 0 8px 25px rgba(0,0,0,0.15);
    }
    
    .gallery-item.hidden {
      display: none;
    }
    
    .img-wrapper {
      position: relative;
      padding-top: 66%;
      overflow: hidden;
      background: #f0f0f0;
    }
    
    .img-wrapper img {
      position: absolute;
      inset: 0;
      width: 100%;
      height: 100%;
      object-fit: cover;
      transition: transform 0.3s, opacity 0.3s;
      opacity: 0;
    }
    
    .img-wrapper img.loaded {
      opacity: 1;
    }
    
    .img-placeholder {
      position: absolute;
      inset: 0;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 40px;
      background: linear-gradient(135deg, #f0f0f0, #e0e0e0);
    }
    
    .gallery-item:hover .img-wrapper img {
      transform: scale(1.05);
    }
    
    .gallery-item-info {
      padding: 14px;
    }
    
    .gallery-item-info h3 {
      font-size: 14px;
      color: #333;
      margin-bottom: 6px;
    }
    
    .gallery-item-meta {
      display: flex;
      justify-content: space-between;
      align-items: center;
    }
    
    .gallery-tag {
      font-size: 11px;
      padding: 2px 10px;
      border-radius: 10px;
      background: #e8f4fd;
      color: #0077cc;
      font-weight: 600;
    }
    
    .gallery-likes {
      font-size: 12px;
      color: #999;
      display: flex;
      align-items: center;
      gap: 4px;
      cursor: pointer;
    }
    
    .gallery-likes.liked { color: #e74c3c; }
    
    /* Lightbox */
    .lightbox {
      display: none;
      position: fixed;
      inset: 0;
      background: rgba(0,0,0,0.9);
      z-index: 1000;
      align-items: center;
      justify-content: center;
    }
    
    .lightbox.show { display: flex; }
    
    .lightbox-img {
      max-width: 85vw;
      max-height: 80vh;
      object-fit: contain;
      border-radius: 8px;
      box-shadow: 0 20px 60px rgba(0,0,0,0.5);
      animation: lightboxIn 0.3s ease;
    }
    
    @keyframes lightboxIn {
      from { opacity: 0; transform: scale(0.9); }
      to { opacity: 1; transform: scale(1); }
    }
    
    .lightbox-nav {
      position: absolute;
      top: 50%;
      transform: translateY(-50%);
      background: rgba(255,255,255,0.15);
      border: none;
      color: white;
      font-size: 24px;
      width: 50px;
      height: 50px;
      border-radius: 50%;
      cursor: pointer;
      backdrop-filter: blur(4px);
      transition: background 0.2s;
    }
    
    .lightbox-nav:hover { background: rgba(255,255,255,0.25); }
    .lightbox-prev { left: 16px; }
    .lightbox-next { right: 16px; }
    
    .lightbox-close {
      position: absolute;
      top: 16px;
      right: 16px;
      background: rgba(255,255,255,0.15);
      border: none;
      color: white;
      font-size: 20px;
      width: 44px;
      height: 44px;
      border-radius: 50%;
      cursor: pointer;
      backdrop-filter: blur(4px);
    }
    
    .lightbox-info {
      position: absolute;
      bottom: 24px;
      left: 50%;
      transform: translateX(-50%);
      color: white;
      text-align: center;
      background: rgba(0,0,0,0.5);
      padding: 10px 20px;
      border-radius: 20px;
      backdrop-filter: blur(4px);
    }
    
    .lightbox-info h3 { font-size: 16px; margin-bottom: 4px; }
    .lightbox-counter { font-size: 12px; opacity: 0.7; }
  </style>
</head>
<body>

<div class="gallery-header">
  <h1>🖼️ Image Gallery</h1>
  <div class="filter-bar" id="filter-bar">
    <button class="filter-btn active" data-filter="all">ทั้งหมด</button>
  </div>
</div>

<div class="gallery-main">
  <div class="gallery-grid" id="gallery-grid"></div>
</div>

<div class="lightbox" id="lightbox">
  <button class="lightbox-close" id="lightbox-close">✕</button>
  <button class="lightbox-nav lightbox-prev" id="prev-btn">‹</button>
  <img class="lightbox-img" id="lightbox-img" src="" alt="">
  <button class="lightbox-nav lightbox-next" id="next-btn">›</button>
  <div class="lightbox-info">
    <h3 id="lightbox-title"></h3>
    <div class="lightbox-counter" id="lightbox-counter"></div>
  </div>
</div>

<script>
// Gallery Data
const GALLERY_DATA = [
  { id: 1, title: "ทิวทัศน์ภูเขา", category: "ธรรมชาติ", emoji: "⛰️", likes: 142 },
  { id: 2, title: "พระอาทิตย์ตก", category: "ธรรมชาติ", emoji: "🌅", likes: 89 },
  { id: 3, title: "เมืองกลางคืน", category: "สถาปัตยกรรม", emoji: "🌆", likes: 256 },
  { id: 4, title: "สัตว์ป่า", category: "สัตว์", emoji: "🦁", likes: 178 },
  { id: 5, title: "อาหารไทย", category: "อาหาร", emoji: "🍜", likes: 312 },
  { id: 6, title: "วัดไทย", category: "สถาปัตยกรรม", emoji: "🛕", likes: 95 },
  { id: 7, title: "ดอกไม้สวน", category: "ธรรมชาติ", emoji: "🌺", likes: 203 },
  { id: 8, title: "นกนางนวล", category: "สัตว์", emoji: "🦅", likes: 67 },
  { id: 9, title: "พิซซ่า", category: "อาหาร", emoji: "🍕", likes: 445 },
  { id: 10, title: "สะพาน", category: "สถาปัตยกรรม", emoji: "🌉", likes: 128 },
  { id: 11, title: "ทะเลสาบ", category: "ธรรมชาติ", emoji: "🏞️", likes: 167 },
  { id: 12, title: "แมวน้อย", category: "สัตว์", emoji: "🐱", likes: 523 }
];

let currentFilter = "all";
let lightboxIndex = 0;
let likedItems = new Set(JSON.parse(localStorage.getItem("liked_gallery") || "[]"));
const observer = new IntersectionObserver(handleLazyLoad, { rootMargin: "200px" });

function getFilteredItems() {
  return currentFilter === "all"
    ? GALLERY_DATA
    : GALLERY_DATA.filter(item => item.category === currentFilter);
}

function init() {
  setupFilters();
  renderGallery();
}

function setupFilters() {
  const categories = [...new Set(GALLERY_DATA.map(item => item.category))];
  const filterBar = document.getElementById("filter-bar");
  
  categories.forEach(cat => {
    const btn = document.createElement("button");
    btn.className = "filter-btn";
    btn.dataset.filter = cat;
    btn.textContent = cat;
    filterBar.appendChild(btn);
  });
  
  filterBar.addEventListener("click", (e) => {
    if (!e.target.classList.contains("filter-btn")) return;
    
    currentFilter = e.target.dataset.filter;
    
    document.querySelectorAll(".filter-btn").forEach(btn => {
      btn.classList.toggle("active", btn.dataset.filter === currentFilter);
    });
    
    filterGallery();
  });
}

function renderGallery() {
  const grid = document.getElementById("gallery-grid");
  grid.innerHTML = "";
  
  GALLERY_DATA.forEach((item, index) => {
    const el = createGalleryItem(item, index);
    grid.appendChild(el);
    observer.observe(el.querySelector("img"));
  });
}

function createGalleryItem(item, index) {
  const el = document.createElement("div");
  el.className = "gallery-item";
  el.dataset.id = item.id;
  el.dataset.category = item.category;
  
  const isLiked = likedItems.has(item.id);
  
  el.innerHTML = `
    <div class="img-wrapper">
      <div class="img-placeholder">${item.emoji}</div>
      <img data-src="https://picsum.photos/seed/${item.id}/400/300" alt="${item.title}" loading="lazy">
    </div>
    <div class="gallery-item-info">
      <h3>${item.title}</h3>
      <div class="gallery-item-meta">
        <span class="gallery-tag">${item.category}</span>
        <span class="gallery-likes ${isLiked ? "liked" : ""}" data-id="${item.id}">
          ${isLiked ? "❤️" : "🤍"} ${item.likes + (isLiked ? 1 : 0)}
        </span>
      </div>
    </div>
  `;
  
  el.querySelector(".img-wrapper").addEventListener("click", () => {
    const filtered = getFilteredItems();
    const idx = filtered.findIndex(i => i.id === item.id);
    openLightbox(idx);
  });
  
  el.querySelector(".gallery-likes").addEventListener("click", (e) => {
    e.stopPropagation();
    toggleLike(item.id, e.currentTarget);
  });
  
  return el;
}

function handleLazyLoad(entries) {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      const img = entry.target;
      img.src = img.dataset.src;
      img.onload = () => img.classList.add("loaded");
      observer.unobserve(img);
    }
  });
}

function filterGallery() {
  const filtered = getFilteredItems();
  const filteredIds = new Set(filtered.map(i => i.id));
  
  document.querySelectorAll(".gallery-item").forEach(el => {
    el.classList.toggle("hidden", !filteredIds.has(parseInt(el.dataset.id)));
  });
}

function toggleLike(id, el) {
  if (likedItems.has(id)) {
    likedItems.delete(id);
    el.classList.remove("liked");
    el.innerHTML = `🤍 ${GALLERY_DATA.find(i => i.id === id).likes}`;
  } else {
    likedItems.add(id);
    el.classList.add("liked");
    el.innerHTML = `❤️ ${GALLERY_DATA.find(i => i.id === id).likes + 1}`;
  }
  
  localStorage.setItem("liked_gallery", JSON.stringify([...likedItems]));
}

function openLightbox(index) {
  lightboxIndex = index;
  updateLightbox();
  document.getElementById("lightbox").classList.add("show");
  document.body.style.overflow = "hidden";
}

function closeLightbox() {
  document.getElementById("lightbox").classList.remove("show");
  document.body.style.overflow = "";
}

function updateLightbox() {
  const items = getFilteredItems();
  const item = items[lightboxIndex];
  
  if (!item) return;
  
  const img = document.getElementById("lightbox-img");
  img.style.opacity = "0";
  img.src = `https://picsum.photos/seed/${item.id}/1200/800`;
  img.onload = () => {
    img.style.transition = "opacity 0.3s";
    img.style.opacity = "1";
  };
  
  document.getElementById("lightbox-title").textContent = item.title;
  document.getElementById("lightbox-counter").textContent = `${lightboxIndex + 1} / ${items.length}`;
}

document.getElementById("lightbox-close").addEventListener("click", closeLightbox);

document.getElementById("prev-btn").addEventListener("click", () => {
  const items = getFilteredItems();
  lightboxIndex = (lightboxIndex - 1 + items.length) % items.length;
  updateLightbox();
});

document.getElementById("next-btn").addEventListener("click", () => {
  const items = getFilteredItems();
  lightboxIndex = (lightboxIndex + 1) % items.length;
  updateLightbox();
});

document.getElementById("lightbox").addEventListener("click", (e) => {
  if (e.target === document.getElementById("lightbox")) closeLightbox();
});

document.addEventListener("keydown", (e) => {
  if (!document.getElementById("lightbox").classList.contains("show")) return;
  
  if (e.key === "ArrowLeft") document.getElementById("prev-btn").click();
  if (e.key === "ArrowRight") document.getElementById("next-btn").click();
  if (e.key === "Escape") closeLightbox();
});

init();
</script>
</body>
</html>
```

---

## Step 983-986: โปรเจค 4 - Markdown Editor

```html
<!DOCTYPE html>
<html lang="th">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Markdown Editor</title>
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; }
    
    body { font-family: 'Sarabun', monospace; height: 100vh; display: flex; flex-direction: column; background: #1e1e2e; }
    
    .toolbar {
      background: #2d2d3f;
      padding: 10px 16px;
      display: flex;
      align-items: center;
      gap: 8px;
      border-bottom: 1px solid #3d3d5c;
    }
    
    .toolbar-title { color: #cdd6f4; font-size: 16px; font-weight: 600; margin-right: 8px; }
    
    .toolbar-btn {
      background: #3d3d5c;
      border: none;
      color: #cdd6f4;
      padding: 5px 10px;
      border-radius: 5px;
      cursor: pointer;
      font-size: 13px;
      font-family: inherit;
      transition: background 0.2s;
    }
    
    .toolbar-btn:hover { background: #4d4d6c; }
    
    .toolbar-sep { width: 1px; height: 20px; background: #4d4d6c; margin: 0 4px; }
    
    .editor-container {
      flex: 1;
      display: grid;
      grid-template-columns: 1fr 1fr;
      overflow: hidden;
    }
    
    .editor-pane, .preview-pane {
      display: flex;
      flex-direction: column;
      overflow: hidden;
    }
    
    .pane-header {
      background: #2d2d3f;
      padding: 8px 16px;
      font-size: 12px;
      font-weight: 600;
      color: #7f849c;
      text-transform: uppercase;
      letter-spacing: 1px;
      border-bottom: 1px solid #3d3d5c;
    }
    
    .editor-pane { border-right: 1px solid #3d3d5c; }
    
    #md-input {
      flex: 1;
      background: #1e1e2e;
      color: #cdd6f4;
      border: none;
      padding: 20px;
      font-family: 'Fira Code', 'Consolas', monospace;
      font-size: 14px;
      line-height: 1.8;
      resize: none;
      outline: none;
      tab-size: 2;
    }
    
    #md-input::placeholder { color: #585b70; }
    
    .preview-pane { background: #1e1e2e; }
    
    #preview {
      flex: 1;
      overflow-y: auto;
      padding: 20px;
      color: #cdd6f4;
    }
    
    #preview h1 { font-size: 2em; border-bottom: 2px solid #3d3d5c; padding-bottom: 8px; margin: 0 0 16px; color: #cba6f7; }
    #preview h2 { font-size: 1.5em; border-bottom: 1px solid #3d3d5c; padding-bottom: 4px; margin: 20px 0 12px; color: #89b4fa; }
    #preview h3 { font-size: 1.2em; margin: 16px 0 8px; color: #74c7ec; }
    #preview p { margin: 0 0 12px; line-height: 1.7; }
    #preview code { background: #2d2d3f; padding: 2px 6px; border-radius: 4px; font-family: monospace; color: #f38ba8; }
    #preview pre { background: #2d2d3f; padding: 16px; border-radius: 8px; overflow-x: auto; margin: 12px 0; }
    #preview pre code { background: none; padding: 0; color: #a6e3a1; }
    #preview blockquote { border-left: 3px solid #cba6f7; padding-left: 16px; color: #a6adc8; margin: 12px 0; }
    #preview ul, #preview ol { padding-left: 20px; margin: 0 0 12px; }
    #preview li { margin-bottom: 4px; line-height: 1.6; }
    #preview a { color: #89dceb; text-decoration: none; }
    #preview a:hover { text-decoration: underline; }
    #preview table { border-collapse: collapse; width: 100%; margin: 12px 0; }
    #preview th, #preview td { border: 1px solid #3d3d5c; padding: 8px 12px; text-align: left; }
    #preview th { background: #2d2d3f; }
    #preview hr { border: 1px solid #3d3d5c; margin: 16px 0; }
    #preview strong { color: #f9e2af; }
    #preview em { color: #fab387; font-style: italic; }
    
    .status-bar {
      background: #2d2d3f;
      padding: 4px 16px;
      font-size: 11px;
      color: #585b70;
      display: flex;
      gap: 16px;
      border-top: 1px solid #3d3d5c;
    }
  </style>
</head>
<body>

<div class="toolbar">
  <span class="toolbar-title">📝 Markdown Editor</span>
  <button class="toolbar-btn" onclick="insertMD('**', '**')"><strong>B</strong></button>
  <button class="toolbar-btn" onclick="insertMD('*', '*')"><em>I</em></button>
  <button class="toolbar-btn" onclick="insertMD('`', '`')">Code</button>
  <button class="toolbar-btn" onclick="insertLine('# ')">H1</button>
  <button class="toolbar-btn" onclick="insertLine('## ')">H2</button>
  <button class="toolbar-btn" onclick="insertLine('### ')">H3</button>
  <div class="toolbar-sep"></div>
  <button class="toolbar-btn" onclick="insertLine('- ')">List</button>
  <button class="toolbar-btn" onclick="insertLine('> ')">Quote</button>
  <button class="toolbar-btn" onclick="insertLink()">Link</button>
  <div class="toolbar-sep"></div>
  <button class="toolbar-btn" onclick="clearEditor()">Clear</button>
  <button class="toolbar-btn" onclick="downloadMD()">⬇ Export</button>
</div>

<div class="editor-container">
  <div class="editor-pane">
    <div class="pane-header">✏️ Editor</div>
    <textarea id="md-input" placeholder="เริ่มพิมพ์ Markdown ที่นี่...

# หัวข้อใหญ่
## หัวข้อรอง

**ตัวหนา** และ *ตัวเอียง*

- รายการที่ 1
- รายการที่ 2

> คำพูดที่ต้องการ quote

\`\`\`javascript
console.log('Hello, World!');
\`\`\`"></textarea>
  </div>
  
  <div class="preview-pane">
    <div class="pane-header">👁️ Preview</div>
    <div id="preview"></div>
  </div>
</div>

<div class="status-bar">
  <span id="stat-chars">ตัวอักษร: 0</span>
  <span id="stat-words">คำ: 0</span>
  <span id="stat-lines">บรรทัด: 0</span>
</div>

<script>
const input = document.getElementById("md-input");
const preview = document.getElementById("preview");

// Simple Markdown Parser
function parseMarkdown(md) {
  let html = md;
  
  // Escape HTML entities first
  html = html.replace(/&/g, "&amp;").replace(/</g, "&lt;").replace(/>/g, "&gt;");
  
  // Code blocks (```)
  html = html.replace(/```(\w*)\n?([\s\S]*?)```/g, (_, lang, code) => {
    return `<pre><code class="language-${lang}">${code.trim()}</code></pre>`;
  });
  
  // Inline code
  html = html.replace(/`([^`]+)`/g, "<code>$1</code>");
  
  // Headers
  html = html.replace(/^### (.+)$/gm, "<h3>$1</h3>");
  html = html.replace(/^## (.+)$/gm, "<h2>$1</h2>");
  html = html.replace(/^# (.+)$/gm, "<h1>$1</h1>");
  
  // Bold and italic
  html = html.replace(/\*\*\*(.+?)\*\*\*/g, "<strong><em>$1</em></strong>");
  html = html.replace(/\*\*(.+?)\*\*/g, "<strong>$1</strong>");
  html = html.replace(/\*(.+?)\*/g, "<em>$1</em>");
  html = html.replace(/__(.+?)__/g, "<strong>$1</strong>");
  html = html.replace(/_(.+?)_/g, "<em>$1</em>");
  
  // Strikethrough
  html = html.replace(/~~(.+?)~~/g, "<del>$1</del>");
  
  // Images
  html = html.replace(/!\[([^\]]*)\]\(([^)]+)\)/g, '<img src="$2" alt="$1" style="max-width:100%">');
  
  // Links
  html = html.replace(/\[([^\]]+)\]\(([^)]+)\)/g, '<a href="$2" target="_blank">$1</a>');
  
  // Horizontal rules
  html = html.replace(/^---+$/gm, "<hr>");
  
  // Blockquotes
  html = html.replace(/^> (.+)$/gm, "<blockquote>$1</blockquote>");
  
  // Unordered lists
  html = html.replace(/^(\s*[-*+] .+)(\n\s*[-*+] .+)*/gm, (match) => {
    const items = match.split("\n")
      .map(line => line.replace(/^\s*[-*+] /, "").trim())
      .filter(Boolean)
      .map(item => `<li>${item}</li>`)
      .join("\n");
    return `<ul>${items}</ul>`;
  });
  
  // Ordered lists
  html = html.replace(/^(\s*\d+\. .+)(\n\s*\d+\. .+)*/gm, (match) => {
    const items = match.split("\n")
      .map(line => line.replace(/^\s*\d+\. /, "").trim())
      .filter(Boolean)
      .map(item => `<li>${item}</li>`)
      .join("\n");
    return `<ol>${items}</ol>`;
  });
  
  // Tables
  html = html.replace(/\|(.+)\|\n\|[-|: ]+\|\n((?:\|.+\|\n?)+)/g, (match, header, body) => {
    const headers = header.split("|").map(h => h.trim()).filter(Boolean);
    const rows = body.trim().split("\n");
    
    const thead = "<tr>" + headers.map(h => `<th>${h}</th>`).join("") + "</tr>";
    const tbody = rows.map(row => {
      const cells = row.split("|").map(c => c.trim()).filter(Boolean);
      return "<tr>" + cells.map(c => `<td>${c}</td>`).join("") + "</tr>";
    }).join("\n");
    
    return `<table><thead>${thead}</thead><tbody>${tbody}</tbody></table>`;
  });
  
  // Paragraphs
  html = html.replace(/^(?!<[a-zA-Z])(.+)$/gm, (match) => {
    if (match.trim() && !match.startsWith("<")) {
      return `<p>${match}</p>`;
    }
    return match;
  });
  
  // Line breaks
  html = html.replace(/\n\n+/g, "\n");
  
  return html;
}

function updatePreview() {
  const md = input.value;
  preview.innerHTML = parseMarkdown(md);
  updateStats(md);
  saveDraft(md);
}

function updateStats(md) {
  const chars = md.length;
  const words = md.trim() ? md.trim().split(/\s+/).length : 0;
  const lines = md.split("\n").length;
  
  document.getElementById("stat-chars").textContent = `ตัวอักษร: ${chars}`;
  document.getElementById("stat-words").textContent = `คำ: ${words}`;
  document.getElementById("stat-lines").textContent = `บรรทัด: ${lines}`;
}

function saveDraft(content) {
  try {
    localStorage.setItem("md_draft", content);
  } catch {}
}

function insertMD(before, after) {
  const start = input.selectionStart;
  const end = input.selectionEnd;
  const selected = input.value.substring(start, end);
  
  const newText = selected ? `${before}${selected}${after}` : `${before}text${after}`;
  
  input.setRangeText(newText, start, end, "select");
  input.focus();
  updatePreview();
}

function insertLine(prefix) {
  const start = input.selectionStart;
  const lineStart = input.value.lastIndexOf("\n", start - 1) + 1;
  const lineEnd = input.value.indexOf("\n", start);
  const end = lineEnd === -1 ? input.value.length : lineEnd;
  const currentLine = input.value.substring(lineStart, end);
  
  const newLine = currentLine.startsWith(prefix)
    ? currentLine.slice(prefix.length)
    : prefix + currentLine;
  
  input.setRangeText(newLine, lineStart, end, "end");
  input.focus();
  updatePreview();
}

function insertLink() {
  const url = prompt("URL:");
  if (!url) return;
  const text = prompt("ข้อความ:", url);
  insertMD("", "");
  
  const start = input.selectionStart;
  const insert = `[${text || url}](${url})`;
  input.setRangeText(insert, start, start, "end");
  input.focus();
  updatePreview();
}

function clearEditor() {
  if (confirm("ล้างเนื้อหาทั้งหมด?")) {
    input.value = "";
    updatePreview();
  }
}

function downloadMD() {
  const blob = new Blob([input.value], { type: "text/markdown" });
  const url = URL.createObjectURL(blob);
  const a = document.createElement("a");
  a.href = url;
  a.download = "document.md";
  a.click();
  URL.revokeObjectURL(url);
}

// Tab key support
input.addEventListener("keydown", (e) => {
  if (e.key === "Tab") {
    e.preventDefault();
    const start = input.selectionStart;
    input.setRangeText("  ", start, start, "end");
    updatePreview();
  }
});

input.addEventListener("input", updatePreview);

// Load draft
const draft = localStorage.getItem("md_draft");
if (draft) {
  input.value = draft;
  updatePreview();
} else {
  // Default content
  input.value = `# ยินดีต้อนรับสู่ Markdown Editor! 🎉

## ทดสอบ Markdown

**ตัวหนา** และ *ตัวเอียง* และ ~~ขีดฆ่า~~

### รายการ
- รายการที่ 1
- รายการที่ 2
  - รายการย่อย

### ตัวอย่างโค้ด
\`\`\`javascript
function hello() {
  console.log("สวัสดีโลก!");
}
\`\`\`

> นี่คือข้อความ blockquote

| หัวข้อ 1 | หัวข้อ 2 |
|---------|---------|
| ข้อมูล 1 | ข้อมูล 2 |`;
  
  updatePreview();
}
</script>
</body>
</html>
```

---

## Step 987-990: โปรเจค 5 - Budget Tracker

```html
<!DOCTYPE html>
<html lang="th">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Budget Tracker</title>
  <style>
    * { box-sizing: border-box; margin: 0; padding: 0; }
    
    body { font-family: 'Sarabun', sans-serif; background: #f0f4f8; }
    
    .app-header {
      background: linear-gradient(135deg, #1a1a2e, #16213e, #0f3460);
      color: white;
      padding: 20px;
    }
    
    .header-content {
      max-width: 1100px;
      margin: 0 auto;
      display: flex;
      align-items: center;
      justify-content: space-between;
    }
    
    .header-content h1 { font-size: 22px; }
    
    .month-selector {
      background: rgba(255,255,255,0.1);
      border: none;
      color: white;
      padding: 6px 12px;
      border-radius: 8px;
      font-family: inherit;
      font-size: 14px;
      cursor: pointer;
    }
    
    .main-content {
      max-width: 1100px;
      margin: 0 auto;
      padding: 20px;
    }
    
    .summary-cards {
      display: grid;
      grid-template-columns: repeat(3, 1fr);
      gap: 16px;
      margin-bottom: 20px;
    }
    
    .summary-card {
      background: white;
      border-radius: 14px;
      padding: 20px;
      box-shadow: 0 2px 12px rgba(0,0,0,0.06);
    }
    
    .summary-card-label {
      font-size: 13px;
      color: #666;
      margin-bottom: 8px;
    }
    
    .summary-card-value {
      font-size: 26px;
      font-weight: 700;
    }
    
    .income-value { color: #10b981; }
    .expense-value { color: #ef4444; }
    .balance-value { color: #3b82f6; }
    
    .summary-card-change {
      font-size: 12px;
      color: #999;
      margin-top: 4px;
    }
    
    .content-grid {
      display: grid;
      grid-template-columns: 1fr 380px;
      gap: 20px;
    }
    
    .transactions-section, .sidebar {
      display: flex;
      flex-direction: column;
      gap: 16px;
    }
    
    .card {
      background: white;
      border-radius: 14px;
      padding: 20px;
      box-shadow: 0 2px 12px rgba(0,0,0,0.06);
    }
    
    .card-header {
      display: flex;
      align-items: center;
      justify-content: space-between;
      margin-bottom: 16px;
    }
    
    .card-title { font-size: 16px; font-weight: 600; }
    
    .add-btn {
      background: #3b82f6;
      color: white;
      border: none;
      padding: 7px 14px;
      border-radius: 8px;
      cursor: pointer;
      font-size: 13px;
      font-family: inherit;
    }
    
    .add-btn:hover { background: #2563eb; }
    
    .add-form {
      display: none;
      background: #f8fafc;
      border-radius: 10px;
      padding: 16px;
      margin-bottom: 16px;
    }
    
    .add-form.show { display: block; }
    
    .form-row {
      display: grid;
      grid-template-columns: 1fr 1fr;
      gap: 10px;
      margin-bottom: 10px;
    }
    
    .form-group label {
      display: block;
      font-size: 12px;
      color: #666;
      margin-bottom: 4px;
    }
    
    .form-input {
      width: 100%;
      padding: 8px 12px;
      border: 1px solid #e2e8f0;
      border-radius: 8px;
      font-family: inherit;
      font-size: 13px;
      outline: none;
    }
    
    .form-input:focus { border-color: #3b82f6; }
    
    .type-selector {
      display: flex;
      gap: 6px;
      margin-bottom: 10px;
    }
    
    .type-btn {
      flex: 1;
      padding: 7px;
      border-radius: 8px;
      border: 2px solid #e2e8f0;
      cursor: pointer;
      font-size: 13px;
      background: white;
      font-family: inherit;
    }
    
    .type-btn.income.active { border-color: #10b981; background: #f0fdf4; color: #10b981; }
    .type-btn.expense.active { border-color: #ef4444; background: #fef2f2; color: #ef4444; }
    
    .save-btn {
      width: 100%;
      padding: 9px;
      background: #3b82f6;
      color: white;
      border: none;
      border-radius: 8px;
      cursor: pointer;
      font-size: 14px;
      font-family: inherit;
    }
    
    .transaction-item {
      display: flex;
      align-items: center;
      gap: 12px;
      padding: 12px;
      border-radius: 10px;
      transition: background 0.15s;
    }
    
    .transaction-item:hover { background: #f8fafc; }
    
    .tx-icon {
      width: 40px;
      height: 40px;
      border-radius: 10px;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 18px;
      flex-shrink: 0;
    }
    
    .tx-info { flex: 1; min-width: 0; }
    
    .tx-title {
      font-size: 14px;
      font-weight: 500;
      color: #333;
      white-space: nowrap;
      overflow: hidden;
      text-overflow: ellipsis;
    }
    
    .tx-meta { font-size: 11px; color: #999; margin-top: 2px; }
    
    .tx-amount {
      font-size: 15px;
      font-weight: 600;
      flex-shrink: 0;
    }
    
    .tx-amount.income { color: #10b981; }
    .tx-amount.expense { color: #ef4444; }
    
    .tx-delete {
      background: none;
      border: none;
      cursor: pointer;
      opacity: 0;
      font-size: 14px;
      transition: opacity 0.2s;
    }
    
    .transaction-item:hover .tx-delete { opacity: 0.5; }
    .tx-delete:hover { opacity: 1 !important; }
    
    .empty-state {
      text-align: center;
      color: #aaa;
      padding: 30px;
      font-size: 14px;
    }
    
    .chart-container { position: relative; }
    
    canvas { width: 100% !important; }
    
    .category-list { display: flex; flex-direction: column; gap: 10px; }
    
    .category-item {
      display: flex;
      align-items: center;
      gap: 10px;
    }
    
    .cat-icon { font-size: 18px; width: 28px; text-align: center; }
    .cat-info { flex: 1; }
    
    .cat-name { font-size: 13px; font-weight: 500; }
    .cat-amount { font-size: 12px; color: #999; }
    
    .cat-progress {
      height: 4px;
      background: #e2e8f0;
      border-radius: 2px;
      margin-top: 3px;
      overflow: hidden;
    }
    
    .cat-progress-fill {
      height: 100%;
      border-radius: 2px;
      transition: width 0.5s ease;
    }

    @media (max-width: 768px) {
      .content-grid { grid-template-columns: 1fr; }
      .summary-cards { grid-template-columns: 1fr; }
    }
  </style>
</head>
<body>

<div class="app-header">
  <div class="header-content">
    <h1>💰 Budget Tracker</h1>
    <select class="month-selector" id="month-selector">
      <!-- จะถูก generate โดย JavaScript -->
    </select>
  </div>
</div>

<div class="main-content">
  <div class="summary-cards">
    <div class="summary-card">
      <div class="summary-card-label">📈 รายรับทั้งหมด</div>
      <div class="summary-card-value income-value" id="total-income">฿0</div>
      <div class="summary-card-change" id="income-change"></div>
    </div>
    <div class="summary-card">
      <div class="summary-card-label">📉 รายจ่ายทั้งหมด</div>
      <div class="summary-card-value expense-value" id="total-expense">฿0</div>
      <div class="summary-card-change" id="expense-change"></div>
    </div>
    <div class="summary-card">
      <div class="summary-card-label">💎 ยอดคงเหลือ</div>
      <div class="summary-card-value balance-value" id="balance">฿0</div>
      <div class="summary-card-change" id="balance-change"></div>
    </div>
  </div>
  
  <div class="content-grid">
    <div class="transactions-section">
      <div class="card">
        <div class="card-header">
          <span class="card-title">รายการธุรกรรม</span>
          <button class="add-btn" id="toggle-form">+ เพิ่ม</button>
        </div>
        
        <div class="add-form" id="add-form">
          <div class="type-selector">
            <button class="type-btn income active" id="type-income">📈 รายรับ</button>
            <button class="type-btn expense" id="type-expense">📉 รายจ่าย</button>
          </div>
          
          <div class="form-row">
            <div class="form-group">
              <label>รายการ</label>
              <input type="text" class="form-input" id="tx-title" placeholder="เช่น เงินเดือน">
            </div>
            <div class="form-group">
              <label>จำนวนเงิน (บาท)</label>
              <input type="number" class="form-input" id="tx-amount" placeholder="0" min="0" step="0.01">
            </div>
          </div>
          
          <div class="form-row">
            <div class="form-group">
              <label>หมวดหมู่</label>
              <select class="form-input" id="tx-category">
                <option value="เงินเดือน">💼 เงินเดือน</option>
                <option value="อาหาร">🍔 อาหาร</option>
                <option value="เดินทาง">🚗 เดินทาง</option>
                <option value="บันเทิง">🎮 บันเทิง</option>
                <option value="สุขภาพ">💊 สุขภาพ</option>
                <option value="ช้อปปิ้ง">🛍️ ช้อปปิ้ง</option>
                <option value="ค่าเช่า">🏠 ค่าเช่า</option>
                <option value="อื่นๆ">📦 อื่นๆ</option>
              </select>
            </div>
            <div class="form-group">
              <label>วันที่</label>
              <input type="date" class="form-input" id="tx-date">
            </div>
          </div>
          
          <button class="save-btn" id="save-tx">💾 บันทึก</button>
        </div>
        
        <div id="transactions-list"></div>
      </div>
    </div>
    
    <div class="sidebar">
      <div class="card">
        <div class="card-header">
          <span class="card-title">📊 สรุปรายจ่าย</span>
        </div>
        <div class="chart-container">
          <canvas id="expense-chart" height="200"></canvas>
        </div>
      </div>
      
      <div class="card">
        <div class="card-header">
          <span class="card-title">📂 ตามหมวดหมู่</span>
        </div>
        <div class="category-list" id="category-list"></div>
      </div>
    </div>
  </div>
</div>

<script>
const BUDGET_KEY = "budget_data";
const MONTH_KEY = "budget_month";

const CATEGORY_CONFIG = {
  "เงินเดือน": { icon: "💼", color: "#10b981" },
  "อาหาร": { icon: "🍔", color: "#f59e0b" },
  "เดินทาง": { icon: "🚗", color: "#3b82f6" },
  "บันเทิง": { icon: "🎮", color: "#8b5cf6" },
  "สุขภาพ": { icon: "💊", color: "#ef4444" },
  "ช้อปปิ้ง": { icon: "🛍️", color: "#ec4899" },
  "ค่าเช่า": { icon: "🏠", color: "#6366f1" },
  "อื่นๆ": { icon: "📦", color: "#9ca3af" }
};

let transactions = loadData();
let currentMonth = localStorage.getItem(MONTH_KEY) || getCurrentMonth();
let selectedType = "income";
let chartInstance = null;

function getCurrentMonth() {
  const now = new Date();
  return `${now.getFullYear()}-${String(now.getMonth() + 1).padStart(2, "0")}`;
}

function loadData() {
  try {
    return JSON.parse(localStorage.getItem(BUDGET_KEY)) || getSampleData();
  } catch {
    return getSampleData();
  }
}

function getSampleData() {
  const now = new Date();
  const m = `${now.getFullYear()}-${String(now.getMonth() + 1).padStart(2, "0")}`;
  
  return [
    { id: 1, type: "income", title: "เงินเดือน", amount: 35000, category: "เงินเดือน", date: `${m}-01` },
    { id: 2, type: "expense", title: "ค่าอาหารกลางวัน", amount: 150, category: "อาหาร", date: `${m}-02` },
    { id: 3, type: "expense", title: "ค่าน้ำมัน", amount: 500, category: "เดินทาง", date: `${m}-03` },
    { id: 4, type: "expense", title: "Netflix", amount: 299, category: "บันเทิง", date: `${m}-05` },
    { id: 5, type: "expense", title: "ค่าเช่าบ้าน", amount: 8000, category: "ค่าเช่า", date: `${m}-01` },
    { id: 6, type: "income", title: "งาน Freelance", amount: 5000, category: "อื่นๆ", date: `${m}-10` }
  ];
}

function saveData() {
  try {
    localStorage.setItem(BUDGET_KEY, JSON.stringify(transactions));
    localStorage.setItem(MONTH_KEY, currentMonth);
  } catch {}
}

function getMonthTransactions() {
  return transactions.filter(tx => tx.date.startsWith(currentMonth));
}

function formatCurrency(amount) {
  return `฿${amount.toLocaleString("th-TH", { minimumFractionDigits: 0, maximumFractionDigits: 2 })}`;
}

function updateSummary() {
  const monthTx = getMonthTransactions();
  const income = monthTx.filter(t => t.type === "income").reduce((s, t) => s + t.amount, 0);
  const expense = monthTx.filter(t => t.type === "expense").reduce((s, t) => s + t.amount, 0);
  const balance = income - expense;
  
  document.getElementById("total-income").textContent = formatCurrency(income);
  document.getElementById("total-expense").textContent = formatCurrency(expense);
  document.getElementById("balance").textContent = formatCurrency(balance);
  
  const balEl = document.getElementById("balance");
  balEl.style.color = balance >= 0 ? "#3b82f6" : "#ef4444";
}

function renderTransactions() {
  const list = document.getElementById("transactions-list");
  const monthTx = getMonthTransactions().sort((a, b) => new Date(b.date) - new Date(a.date));
  
  if (!monthTx.length) {
    list.innerHTML = `<div class="empty-state">📭 ยังไม่มีรายการในเดือนนี้</div>`;
    return;
  }
  
  list.innerHTML = monthTx.map(tx => {
    const cat = CATEGORY_CONFIG[tx.category] || { icon: "📦", color: "#9ca3af" };
    const date = new Date(tx.date).toLocaleDateString("th-TH", { day: "numeric", month: "short" });
    
    return `
      <div class="transaction-item">
        <div class="tx-icon" style="background:${cat.color}20;color:${cat.color}">${cat.icon}</div>
        <div class="tx-info">
          <div class="tx-title">${tx.title}</div>
          <div class="tx-meta">${tx.category} · ${date}</div>
        </div>
        <div class="tx-amount ${tx.type}">
          ${tx.type === "income" ? "+" : "-"}${formatCurrency(tx.amount)}
        </div>
        <button class="tx-delete" onclick="deleteTransaction(${tx.id})">🗑️</button>
      </div>
    `;
  }).join("");
}

function renderCategoryBreakdown() {
  const monthTx = getMonthTransactions().filter(t => t.type === "expense");
  const totalExpense = monthTx.reduce((s, t) => s + t.amount, 0);
  
  const categories = {};
  monthTx.forEach(tx => {
    categories[tx.category] = (categories[tx.category] || 0) + tx.amount;
  });
  
  const sorted = Object.entries(categories).sort((a, b) => b[1] - a[1]);
  const list = document.getElementById("category-list");
  
  if (!sorted.length) {
    list.innerHTML = `<div style="text-align:center;color:#aaa;font-size:13px;padding:16px">ยังไม่มีรายจ่าย</div>`;
    return;
  }
  
  list.innerHTML = sorted.map(([cat, amount]) => {
    const config = CATEGORY_CONFIG[cat] || { icon: "📦", color: "#9ca3af" };
    const pct = totalExpense > 0 ? Math.round(amount / totalExpense * 100) : 0;
    
    return `
      <div class="category-item">
        <span class="cat-icon">${config.icon}</span>
        <div class="cat-info">
          <div class="cat-name">${cat} <span style="color:#999;font-size:11px">${pct}%</span></div>
          <div class="cat-amount">${formatCurrency(amount)}</div>
          <div class="cat-progress">
            <div class="cat-progress-fill" style="width:${pct}%;background:${config.color}"></div>
          </div>
        </div>
      </div>
    `;
  }).join("");
}

function drawChart() {
  const canvas = document.getElementById("expense-chart");
  const ctx = canvas.getContext("2d");
  
  const monthTx = getMonthTransactions().filter(t => t.type === "expense");
  
  const categories = {};
  monthTx.forEach(tx => {
    categories[tx.category] = (categories[tx.category] || 0) + tx.amount;
  });
  
  const data = Object.entries(categories);
  if (!data.length) {
    ctx.clearRect(0, 0, canvas.width, canvas.height);
    ctx.fillStyle = "#aaa";
    ctx.font = "14px Sarabun, sans-serif";
    ctx.textAlign = "center";
    ctx.fillText("ยังไม่มีรายจ่าย", canvas.width / 2, canvas.height / 2);
    return;
  }
  
  const W = canvas.offsetWidth || 300;
  const H = 200;
  canvas.width = W;
  canvas.height = H;
  
  const cx = W / 2 - 40;
  const cy = H / 2;
  const radius = Math.min(cx, cy) - 20;
  const total = data.reduce((s, [, v]) => s + v, 0);
  
  ctx.clearRect(0, 0, W, H);
  
  let startAngle = -Math.PI / 2;
  
  data.forEach(([cat, amount]) => {
    const config = CATEGORY_CONFIG[cat] || { color: "#9ca3af" };
    const sliceAngle = (amount / total) * 2 * Math.PI;
    
    ctx.beginPath();
    ctx.moveTo(cx, cy);
    ctx.arc(cx, cy, radius, startAngle, startAngle + sliceAngle);
    ctx.closePath();
    ctx.fillStyle = config.color;
    ctx.fill();
    ctx.strokeStyle = "white";
    ctx.lineWidth = 2;
    ctx.stroke();
    
    startAngle += sliceAngle;
  });
  
  // Center hole
  ctx.beginPath();
  ctx.arc(cx, cy, radius * 0.5, 0, 2 * Math.PI);
  ctx.fillStyle = "white";
  ctx.fill();
  
  ctx.fillStyle = "#333";
  ctx.font = "bold 14px Sarabun, sans-serif";
  ctx.textAlign = "center";
  ctx.fillText(formatCurrency(total), cx, cy);
  
  // Legend
  const legendX = cx + radius + 20;
  let legendY = 20;
  
  data.slice(0, 5).forEach(([cat, amount]) => {
    const config = CATEGORY_CONFIG[cat] || { icon: "📦", color: "#9ca3af" };
    
    ctx.fillStyle = config.color;
    ctx.fillRect(legendX, legendY, 10, 10);
    
    ctx.fillStyle = "#333";
    ctx.font = "11px Sarabun, sans-serif";
    ctx.textAlign = "left";
    ctx.fillText(cat, legendX + 14, legendY + 9);
    
    legendY += 20;
  });
}

function setupMonthSelector() {
  const select = document.getElementById("month-selector");
  const months = [];
  
  for (let i = 0; i < 12; i++) {
    const date = new Date();
    date.setMonth(date.getMonth() - i);
    const value = `${date.getFullYear()}-${String(date.getMonth() + 1).padStart(2, "0")}`;
    const label = date.toLocaleDateString("th-TH", { year: "numeric", month: "long" });
    months.push({ value, label });
  }
  
  select.innerHTML = months.map(m =>
    `<option value="${m.value}" ${m.value === currentMonth ? "selected" : ""}>${m.label}</option>`
  ).join("");
  
  select.addEventListener("change", (e) => {
    currentMonth = e.target.value;
    saveData();
    renderAll();
  });
}

function renderAll() {
  updateSummary();
  renderTransactions();
  renderCategoryBreakdown();
  requestAnimationFrame(drawChart);
}

function deleteTransaction(id) {
  transactions = transactions.filter(t => t.id !== id);
  saveData();
  renderAll();
}

function addTransaction() {
  const title = document.getElementById("tx-title").value.trim();
  const amount = parseFloat(document.getElementById("tx-amount").value);
  const category = document.getElementById("tx-category").value;
  const date = document.getElementById("tx-date").value;
  
  if (!title || isNaN(amount) || amount <= 0 || !date) {
    alert("กรุณากรอกข้อมูลให้ครบถ้วน");
    return;
  }
  
  transactions.push({
    id: Date.now(),
    type: selectedType,
    title,
    amount,
    category,
    date
  });
  
  saveData();
  renderAll();
  
  document.getElementById("tx-title").value = "";
  document.getElementById("tx-amount").value = "";
  document.getElementById("add-form").classList.remove("show");
}

// Events
document.getElementById("toggle-form").addEventListener("click", () => {
  document.getElementById("add-form").classList.toggle("show");
});

document.getElementById("type-income").addEventListener("click", () => {
  selectedType = "income";
  document.getElementById("type-income").classList.add("active");
  document.getElementById("type-expense").classList.remove("active");
  
  const catSel = document.getElementById("tx-category");
  catSel.innerHTML = `
    <option value="เงินเดือน">💼 เงินเดือน</option>
    <option value="อื่นๆ">📦 รายรับอื่นๆ</option>
  `;
});

document.getElementById("type-expense").addEventListener("click", () => {
  selectedType = "expense";
  document.getElementById("type-expense").classList.add("active");
  document.getElementById("type-income").classList.remove("active");
  
  const catSel = document.getElementById("tx-category");
  catSel.innerHTML = `
    <option value="อาหาร">🍔 อาหาร</option>
    <option value="เดินทาง">🚗 เดินทาง</option>
    <option value="บันเทิง">🎮 บันเทิง</option>
    <option value="สุขภาพ">💊 สุขภาพ</option>
    <option value="ช้อปปิ้ง">🛍️ ช้อปปิ้ง</option>
    <option value="ค่าเช่า">🏠 ค่าเช่า</option>
    <option value="อื่นๆ">📦 อื่นๆ</option>
  `;
});

document.getElementById("save-tx").addEventListener("click", addTransaction);

document.getElementById("tx-date").valueAsDate = new Date();

// Resize chart
const resizeObserver = new ResizeObserver(() => {
  requestAnimationFrame(drawChart);
});

resizeObserver.observe(document.getElementById("expense-chart").parentElement);

// Init
setupMonthSelector();
renderAll();
</script>
</body>
</html>
```

---

## สรุป Part 50 - สิ่งที่ได้เรียนรู้จากโปรเจค

### โปรเจค 1: Chat App
- localStorage สำหรับ persist messages
- DOM manipulation สำหรับ render messages
- CSS animations สำหรับ smooth UX
- Auto-resize textarea

### โปรเจค 2: Kanban Board
- Drag and Drop API
- Dynamic DOM manipulation
- Complex data structures
- LocalStorage persistence

### โปรเจค 3: Image Gallery
- IntersectionObserver สำหรับ lazy loading
- CSS Grid สำหรับ responsive layout
- Lightbox pattern
- Filter functionality

### โปรเจค 4: Markdown Editor
- Custom markdown parser
- Split-pane editor
- Auto-save draft
- File download

### โปรเจค 5: Budget Tracker
- Canvas API สำหรับ pie chart
- Data aggregation และ statistics
- ResizeObserver สำหรับ responsive chart
- Complex state management

### แบบฝึกหัดท้าย

1. เพิ่ม features ให้ Chat App: attach files, read receipts
2. เพิ่ม Swimlane view ให้ Kanban Board
3. เพิ่ม zoom และ download ให้ Image Gallery
4. เพิ่ม export HTML ให้ Markdown Editor
5. เพิ่ม budget limits และ alerts ให้ Budget Tracker
