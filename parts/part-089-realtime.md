# Part 89: Real-time Applications
## ขั้นตอนที่ 1751-1770: แอปพลิเคชันแบบ Real-time

Real-time applications ช่วยให้ผู้ใช้หลายคนสามารถโต้ตอบกันได้ทันที ไม่ว่าจะเป็น chat, live scores, collaboration tools หรือ live notifications

---

## ขั้นตอนที่ 1751: Real-time Communication Approaches เปรียบเทียบ

```
1. Short Polling
   Client -----------> Server (request)
   Client <----------- Server (response, empty if no update)
   Client -----------> Server (request again after delay)
   
   เหมาะกับ: simple cases, ไม่ต้องการ real-time จริงๆ
   ข้อเสีย: server load สูง, latency สูง

2. Long Polling
   Client -----------> Server (request)
   // Server hold connection ไว้
   // เมื่อมีข้อมูลใหม่:
   Client <----------- Server (response with data)
   Client -----------> Server (request again immediately)
   
   เหมาะกับ: low-frequency updates
   ข้อเสีย: complex server, connection management

3. Server-Sent Events (SSE)
   Client -----------> Server (EventSource connection)
   Client <----------- Server (event stream, one-way)
   Client <----------- Server (more events...)
   
   เหมาะกับ: server -> client updates เท่านั้น (notifications, feeds)
   ข้อดี: simple, auto-reconnect, HTTP/2 friendly

4. WebSocket
   Client <----------> Server (full-duplex, persistent)
   
   เหมาะกับ: chat, games, collaboration
   ข้อดี: low latency, bidirectional
   ข้อเสีย: complex scaling, stateful

5. WebRTC
   Client <----------> Client (peer-to-peer)
   
   เหมาะกับ: video/audio calls, file sharing
```

---

## ขั้นตอนที่ 1752: Short Polling

```javascript
// Short Polling - วิธีง่ายที่สุด แต่ไม่ efficient

// Client
class ShortPoller {
  constructor(url, interval = 5000) {
    this.url = url;
    this.interval = interval;
    this.timer = null;
    this.lastEtag = null;
  }
  
  start(callback) {
    const poll = async () => {
      try {
        const headers = {};
        if (this.lastEtag) {
          headers['If-None-Match'] = this.lastEtag;
        }
        
        const response = await fetch(this.url, { headers });
        
        if (response.status === 304) {
          // Not Modified - ไม่มีข้อมูลใหม่
          return;
        }
        
        // เก็บ ETag สำหรับ conditional requests
        this.lastEtag = response.headers.get('ETag');
        
        const data = await response.json();
        callback(data);
        
      } catch (error) {
        console.error('Polling error:', error);
      }
    };
    
    // Poll ทันทีครั้งแรก
    poll();
    
    // Poll ทุก interval
    this.timer = setInterval(poll, this.interval);
  }
  
  stop() {
    if (this.timer) {
      clearInterval(this.timer);
      this.timer = null;
    }
  }
}

// ใช้งาน
const poller = new ShortPoller('/api/notifications', 10000); // ทุก 10 วินาที

poller.start((data) => {
  console.log('New notifications:', data);
  updateNotificationBadge(data.count);
});

// หยุดเมื่อไม่ต้องการ
// poller.stop();
```

```javascript
// Server สำหรับ Short Polling
import express from 'express';
import crypto from 'crypto';

const app = express();
let notifications = [
  { id: 1, message: 'ยินดีต้อนรับ!', read: false, timestamp: Date.now() }
];

app.get('/api/notifications', (req, res) => {
  const unread = notifications.filter(n => !n.read);
  
  // สร้าง ETag จาก content
  const content = JSON.stringify(unread);
  const etag = '"' + crypto.createHash('md5').update(content).digest('hex') + '"';
  
  // Check if client has current version
  if (req.headers['if-none-match'] === etag) {
    return res.status(304).end();
  }
  
  res.set('ETag', etag);
  res.json({ notifications: unread, count: unread.length });
});
```

---

## ขั้นตอนที่ 1753: Long Polling

```javascript
// Long Polling - Server holds connection จนมีข้อมูลใหม่

// Server (Express.js)
import express from 'express';
import EventEmitter from 'events';

const app = express();
const emitter = new EventEmitter();
let clients = new Set();

// Endpoint สำหรับ long polling
app.get('/api/updates', async (req, res) => {
  const clientId = req.query.clientId || Date.now().toString();
  const lastMessageId = parseInt(req.query.lastId) || 0;
  
  // ตั้ง timeout เพื่อไม่ให้ connection ค้างนานเกินไป
  const timeout = setTimeout(() => {
    // ไม่มีข้อมูลใหม่ใน 30 วินาที - ส่ง empty response
    res.json({ messages: [], clientId, lastId: lastMessageId });
    cleanup();
  }, 30000);
  
  function cleanup() {
    clearTimeout(timeout);
    emitter.removeListener('newMessage', handler);
    clients.delete(clientId);
  }
  
  function handler(message) {
    if (message.id > lastMessageId) {
      // มีข้อมูลใหม่ - ส่งทันที
      res.json({ messages: [message], clientId, lastId: message.id });
      cleanup();
    }
  }
  
  // รอ event ใหม่
  emitter.on('newMessage', handler);
  clients.add(clientId);
  
  // Handle client disconnect
  req.on('close', cleanup);
});

// Endpoint สำหรับส่ง message ใหม่
app.post('/api/messages', express.json(), (req, res) => {
  const message = {
    id: Date.now(),
    text: req.body.text,
    user: req.body.user,
    timestamp: new Date().toISOString(),
  };
  
  // Emit event ให้ clients ที่กำลัง long poll
  emitter.emit('newMessage', message);
  
  res.json(message);
});

app.listen(3000);
```

```javascript
// Client สำหรับ Long Polling
class LongPollClient {
  constructor(url) {
    this.url = url;
    this.clientId = crypto.randomUUID();
    this.lastId = 0;
    this.running = false;
    this.onMessage = null;
  }
  
  async start() {
    this.running = true;
    
    while (this.running) {
      try {
        const url = `${this.url}?clientId=${this.clientId}&lastId=${this.lastId}`;
        const response = await fetch(url, { signal: AbortSignal.timeout(35000) });
        const data = await response.json();
        
        if (data.messages?.length > 0) {
          for (const msg of data.messages) {
            this.onMessage?.(msg);
          }
          this.lastId = Math.max(this.lastId, ...data.messages.map(m => m.id));
        }
        
      } catch (error) {
        if (error.name === 'AbortError') continue; // timeout - retry
        
        console.error('Long poll error:', error);
        // Wait ก่อน retry
        await new Promise(r => setTimeout(r, 3000));
      }
    }
  }
  
  stop() {
    this.running = false;
  }
  
  async send(text, user) {
    return fetch(`${this.url.replace('/updates', '/messages')}`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ text, user }),
    }).then(r => r.json());
  }
}

const client = new LongPollClient('/api/updates');
client.onMessage = (msg) => {
  console.log(`${msg.user}: ${msg.text}`);
};

client.start();
```

---

## ขั้นตอนที่ 1754: Server-Sent Events (SSE) - Server Implementation

```javascript
// SSE Server (Node.js/Express)
import express from 'express';
import EventEmitter from 'events';

const app = express();
const globalEmitter = new EventEmitter();

// SSE Helper
function createSSEStream(res) {
  // Set SSE headers
  res.setHeader('Content-Type', 'text/event-stream');
  res.setHeader('Cache-Control', 'no-cache');
  res.setHeader('Connection', 'keep-alive');
  res.setHeader('X-Accel-Buffering', 'no'); // สำหรับ nginx
  res.flushHeaders();
  
  return {
    // ส่ง event ธรรมดา
    send(data, eventType = null) {
      if (eventType) {
        res.write(`event: ${eventType}\n`);
      }
      res.write(`data: ${JSON.stringify(data)}\n\n`);
    },
    
    // ส่ง comment (ping เพื่อ keep connection alive)
    ping() {
      res.write(': ping\n\n');
    },
    
    // ส่งพร้อม id (สำหรับ reconnect)
    sendWithId(id, data, eventType = null) {
      res.write(`id: ${id}\n`);
      if (eventType) res.write(`event: ${eventType}\n`);
      res.write(`data: ${JSON.stringify(data)}\n\n`);
    },
  };
}

// SSE Endpoint
app.get('/api/events', (req, res) => {
  const stream = createSSEStream(res);
  const userId = req.query.userId;
  
  // ส่ง initial connection event
  stream.send({ type: 'connected', userId, timestamp: Date.now() }, 'connect');
  
  // Ping ทุก 30 วินาที เพื่อ keep alive
  const pingInterval = setInterval(() => {
    stream.ping();
  }, 30000);
  
  // Handle custom events
  function handleNotification(notification) {
    // ส่งเฉพาะ notification ที่ตรงกับ user
    if (!notification.userId || notification.userId === userId) {
      stream.sendWithId(
        notification.id,
        notification,
        'notification'
      );
    }
  }
  
  function handleBroadcast(message) {
    stream.send(message, 'broadcast');
  }
  
  globalEmitter.on('notification', handleNotification);
  globalEmitter.on('broadcast', handleBroadcast);
  
  // Handle client disconnect
  req.on('close', () => {
    clearInterval(pingInterval);
    globalEmitter.off('notification', handleNotification);
    globalEmitter.off('broadcast', handleBroadcast);
    console.log(`Client ${userId} disconnected`);
  });
});

// Endpoint สำหรับ trigger events
app.post('/api/notify', express.json(), (req, res) => {
  const notification = {
    id: Date.now(),
    userId: req.body.userId,
    type: req.body.type,
    message: req.body.message,
    timestamp: new Date().toISOString(),
  };
  
  globalEmitter.emit('notification', notification);
  res.json({ sent: true, notification });
});

app.post('/api/broadcast', express.json(), (req, res) => {
  globalEmitter.emit('broadcast', {
    type: 'announcement',
    message: req.body.message,
    timestamp: new Date().toISOString(),
  });
  res.json({ sent: true });
});
```

---

## ขั้นตอนที่ 1755: EventSource Client

```javascript
// EventSource API (Browser built-in)
class SSEClient {
  constructor(url) {
    this.url = url;
    this.eventSource = null;
    this.handlers = new Map();
    this.reconnectDelay = 1000;
    this.maxReconnectDelay = 30000;
  }
  
  connect(lastEventId = null) {
    let url = this.url;
    if (lastEventId) {
      url += `?lastEventId=${lastEventId}`;
    }
    
    this.eventSource = new EventSource(url);
    
    // Handle connection open
    this.eventSource.onopen = () => {
      console.log('SSE connected');
      this.reconnectDelay = 1000; // reset delay
      this.trigger('open');
    };
    
    // Handle errors
    this.eventSource.onerror = (error) => {
      console.error('SSE error:', error);
      this.trigger('error', error);
      
      if (this.eventSource.readyState === EventSource.CLOSED) {
        // Auto reconnect with exponential backoff
        console.log(`Reconnecting in ${this.reconnectDelay}ms...`);
        setTimeout(() => {
          this.reconnectDelay = Math.min(
            this.reconnectDelay * 2,
            this.maxReconnectDelay
          );
          this.connect(this.lastEventId);
        }, this.reconnectDelay);
      }
    };
    
    // Handle default message
    this.eventSource.onmessage = (event) => {
      this.lastEventId = event.lastEventId;
      const data = JSON.parse(event.data);
      this.trigger('message', data);
    };
    
    // Register custom event listeners
    this.handlers.forEach((handler, eventName) => {
      if (eventName !== 'message' && eventName !== 'open' && eventName !== 'error') {
        this.eventSource.addEventListener(eventName, (event) => {
          this.lastEventId = event.lastEventId;
          const data = JSON.parse(event.data);
          handler(data);
        });
      }
    });
  }
  
  on(eventName, handler) {
    this.handlers.set(eventName, handler);
    
    if (this.eventSource) {
      this.eventSource.addEventListener(eventName, (event) => {
        const data = JSON.parse(event.data);
        handler(data);
      });
    }
    
    return this;
  }
  
  trigger(eventName, data) {
    const handler = this.handlers.get(eventName);
    if (handler) handler(data);
  }
  
  disconnect() {
    if (this.eventSource) {
      this.eventSource.close();
      this.eventSource = null;
    }
  }
  
  get isConnected() {
    return this.eventSource?.readyState === EventSource.OPEN;
  }
}

// ใช้งาน
const sse = new SSEClient('/api/events?userId=123');

sse
  .on('open', () => {
    console.log('Connected to server!');
    updateConnectionStatus('connected');
  })
  .on('notification', (notification) => {
    console.log('New notification:', notification);
    showNotification(notification.message);
  })
  .on('broadcast', (message) => {
    console.log('Broadcast:', message);
    showAnnouncement(message.message);
  })
  .on('error', () => {
    updateConnectionStatus('reconnecting...');
  });

sse.connect();

// หยุดเมื่อออกจากหน้า
window.addEventListener('beforeunload', () => {
  sse.disconnect();
});
```

---

## ขั้นตอนที่ 1756: SSE กับ Custom Events

```javascript
// SSE สำหรับ Live Score Updates
// Server
import express from 'express';
import EventEmitter from 'events';

const app = express();
const scoreEmitter = new EventEmitter();

// Mock scores data
const scores = {
  'match-1': { home: 'ทีม A', away: 'ทีม B', homeScore: 0, awayScore: 0, minute: 0 },
  'match-2': { home: 'ทีม C', away: 'ทีม D', homeScore: 0, awayScore: 0, minute: 0 },
};

// SSE Endpoint สำหรับ match
app.get('/api/matches/:matchId/stream', (req, res) => {
  const { matchId } = req.params;
  
  if (!scores[matchId]) {
    return res.status(404).send('Match not found');
  }
  
  res.setHeader('Content-Type', 'text/event-stream');
  res.setHeader('Cache-Control', 'no-cache');
  res.setHeader('Connection', 'keep-alive');
  res.flushHeaders();
  
  // ส่งข้อมูล initial
  res.write(`event: init\n`);
  res.write(`data: ${JSON.stringify(scores[matchId])}\n\n`);
  
  function sendScore(update) {
    if (update.matchId === matchId) {
      res.write(`id: ${Date.now()}\n`);
      res.write(`event: score\n`);
      res.write(`data: ${JSON.stringify(update)}\n\n`);
    }
  }
  
  function sendGoal(goal) {
    if (goal.matchId === matchId) {
      res.write(`event: goal\n`);
      res.write(`data: ${JSON.stringify(goal)}\n\n`);
    }
  }
  
  function sendCard(card) {
    if (card.matchId === matchId) {
      res.write(`event: card\n`);
      res.write(`data: ${JSON.stringify(card)}\n\n`);
    }
  }
  
  scoreEmitter.on('scoreUpdate', sendScore);
  scoreEmitter.on('goal', sendGoal);
  scoreEmitter.on('card', sendCard);
  
  req.on('close', () => {
    scoreEmitter.off('scoreUpdate', sendScore);
    scoreEmitter.off('goal', sendGoal);
    scoreEmitter.off('card', sendCard);
  });
});

// Simulate live match updates
setInterval(() => {
  const matchId = 'match-1';
  const score = scores[matchId];
  
  score.minute = Math.min(score.minute + 1, 90);
  
  // สุ่มให้มีประตู
  if (Math.random() < 0.05) { // 5% โอกาสมีประตูต่อนาที
    const isHome = Math.random() > 0.5;
    if (isHome) score.homeScore++;
    else score.awayScore++;
    
    const goal = {
      matchId,
      team: isHome ? 'home' : 'away',
      scorer: isHome ? score.home : score.away,
      minute: score.minute,
    };
    
    scoreEmitter.emit('goal', goal);
    console.log(`GOAL! ${goal.scorer} - ${score.homeScore}:${score.awayScore}`);
  }
  
  scoreEmitter.emit('scoreUpdate', { matchId, ...score });
}, 1000);
```

```javascript
// Client สำหรับ Live Score
function watchMatch(matchId) {
  const eventSource = new EventSource(`/api/matches/${matchId}/stream`);
  const scoreDisplay = document.getElementById('score');
  const eventsLog = document.getElementById('events');
  
  function addEvent(text, type = '') {
    const el = document.createElement('div');
    el.className = `event ${type}`;
    el.textContent = `[${new Date().toLocaleTimeString()}] ${text}`;
    eventsLog.insertBefore(el, eventsLog.firstChild);
  }
  
  eventSource.addEventListener('init', (e) => {
    const data = JSON.parse(e.data);
    scoreDisplay.textContent = `${data.home} ${data.homeScore}:${data.awayScore} ${data.away}`;
    addEvent('เริ่มรับข้อมูลสด');
  });
  
  eventSource.addEventListener('score', (e) => {
    const data = JSON.parse(e.data);
    scoreDisplay.textContent = `${data.home} ${data.homeScore}:${data.awayScore} ${data.away}`;
    document.getElementById('minute').textContent = `นาทีที่ ${data.minute}'`;
  });
  
  eventSource.addEventListener('goal', (e) => {
    const data = JSON.parse(e.data);
    addEvent(`⚽ ประตู! ${data.scorer} (${data.minute}')`, 'goal');
    // แสดง animation
    scoreDisplay.classList.add('goal-animation');
    setTimeout(() => scoreDisplay.classList.remove('goal-animation'), 2000);
  });
  
  eventSource.addEventListener('card', (e) => {
    const data = JSON.parse(e.data);
    const cardEmoji = data.color === 'red' ? '🟥' : '🟨';
    addEvent(`${cardEmoji} ${data.player} - ${data.team} (${data.minute}')`, 'card');
  });
  
  eventSource.onerror = () => {
    addEvent('การเชื่อมต่อขาด กำลังเชื่อมต่อใหม่...', 'error');
  };
  
  return eventSource;
}
```

---

## ขั้นตอนที่ 1757: WebSocket Protocol ในเชิงลึก

```
WebSocket Handshake:
Client -> Server:
  GET /ws HTTP/1.1
  Host: example.com
  Upgrade: websocket
  Connection: Upgrade
  Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
  Sec-WebSocket-Version: 13

Server -> Client:
  HTTP/1.1 101 Switching Protocols
  Upgrade: websocket
  Connection: Upgrade
  Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=

WebSocket Frame Format:
  FIN (1 bit): ถ้าเป็น final fragment
  RSV1-3 (3 bits): extensions
  Opcode (4 bits): 0x0=continuation, 0x1=text, 0x2=binary, 0x8=close, 0x9=ping, 0xA=pong
  MASK (1 bit): client->server ต้อง mask เสมอ
  Payload length (7/16/64 bits)
  Masking key (32 bits, if MASK=1)
  Payload data
```

```javascript
// WebSocket ใน Browser (Native API)
const ws = new WebSocket('wss://example.com/ws');

// Connection opened
ws.addEventListener('open', (event) => {
  console.log('WebSocket connected');
  
  // ส่ง text message
  ws.send('สวัสดี!');
  
  // ส่ง JSON
  ws.send(JSON.stringify({ type: 'hello', data: 'world' }));
  
  // ส่ง Binary data
  const buffer = new ArrayBuffer(4);
  const view = new DataView(buffer);
  view.setInt32(0, 12345);
  ws.send(buffer);
});

// Message received
ws.addEventListener('message', (event) => {
  if (typeof event.data === 'string') {
    // Text message
    try {
      const json = JSON.parse(event.data);
      handleJsonMessage(json);
    } catch {
      console.log('Text message:', event.data);
    }
  } else if (event.data instanceof ArrayBuffer) {
    // Binary data
    console.log('Binary data received');
  } else if (event.data instanceof Blob) {
    // Blob data
    event.data.arrayBuffer().then(buffer => {
      console.log('Blob as ArrayBuffer:', buffer);
    });
  }
});

// Connection closed
ws.addEventListener('close', (event) => {
  console.log('WebSocket closed:', event.code, event.reason);
  
  // Close codes:
  // 1000: Normal closure
  // 1001: Going away (server shutting down or browser navigation)
  // 1006: Abnormal closure (no close frame)
  // 1011: Server error
});

// Error
ws.addEventListener('error', (error) => {
  console.error('WebSocket error:', error);
});

// ตรวจสอบ connection state
console.log(ws.readyState);
// 0: CONNECTING
// 1: OPEN
// 2: CLOSING
// 3: CLOSED

// ปิด connection
ws.close(1000, 'Done'); // code, reason
```

---

## ขั้นตอนที่ 1758: ws Library (Node.js)

```javascript
// WebSocket Server ด้วย ws library
import { WebSocketServer, WebSocket } from 'ws';
import { createServer } from 'http';
import express from 'express';

const app = express();
const server = createServer(app);

const wss = new WebSocketServer({ 
  server,
  path: '/ws',
  // options
  maxPayload: 1024 * 1024, // 1MB max message size
  clientTracking: true,    // track connected clients
});

// Track connected clients
const clients = new Map(); // clientId -> ws

function broadcast(data, excludeClient = null) {
  const message = JSON.stringify(data);
  
  clients.forEach((client, clientId) => {
    if (client !== excludeClient && client.readyState === WebSocket.OPEN) {
      client.send(message);
    }
  });
}

wss.on('connection', (ws, req) => {
  const clientId = crypto.randomUUID();
  const ip = req.headers['x-forwarded-for'] || req.socket.remoteAddress;
  
  clients.set(clientId, ws);
  ws.clientId = clientId;
  
  console.log(`Client ${clientId} connected from ${ip}`);
  console.log(`Total connections: ${clients.size}`);
  
  // ส่ง clientId ให้ client
  ws.send(JSON.stringify({ type: 'connected', clientId }));
  
  // แจ้งทุกคนว่ามีคนใหม่เข้ามา
  broadcast({ type: 'userJoined', clientId, total: clients.size }, ws);
  
  // Handle messages
  ws.on('message', (data, isBinary) => {
    let message;
    
    try {
      message = JSON.parse(data.toString());
    } catch {
      ws.send(JSON.stringify({ type: 'error', message: 'Invalid JSON' }));
      return;
    }
    
    console.log(`Message from ${clientId}:`, message);
    
    switch (message.type) {
      case 'ping':
        ws.send(JSON.stringify({ type: 'pong', timestamp: Date.now() }));
        break;
        
      case 'broadcast':
        broadcast({
          type: 'message',
          from: clientId,
          data: message.data,
          timestamp: Date.now(),
        }, ws);
        break;
        
      case 'private':
        const target = clients.get(message.to);
        if (target && target.readyState === WebSocket.OPEN) {
          target.send(JSON.stringify({
            type: 'privateMessage',
            from: clientId,
            data: message.data,
          }));
        }
        break;
        
      default:
        ws.send(JSON.stringify({ type: 'error', message: 'Unknown message type' }));
    }
  });
  
  // Handle client disconnect
  ws.on('close', (code, reason) => {
    clients.delete(clientId);
    broadcast({ type: 'userLeft', clientId, total: clients.size });
    console.log(`Client ${clientId} disconnected: ${code} ${reason}`);
  });
  
  // Handle errors
  ws.on('error', (error) => {
    console.error(`Client ${clientId} error:`, error);
    clients.delete(clientId);
  });
  
  // Heartbeat - detect zombi connections
  ws.isAlive = true;
  ws.on('pong', () => { ws.isAlive = true; });
});

// Heartbeat interval
const heartbeatInterval = setInterval(() => {
  wss.clients.forEach((ws) => {
    if (!ws.isAlive) {
      clients.delete(ws.clientId);
      return ws.terminate();
    }
    
    ws.isAlive = false;
    ws.ping();
  });
}, 30000);

wss.on('close', () => {
  clearInterval(heartbeatInterval);
});

server.listen(3000, () => {
  console.log('WebSocket server running on port 3000');
});
```

---

## ขั้นตอนที่ 1759: Socket.io - Setup

```bash
# ติดตั้ง
npm install socket.io          # Server
npm install socket.io-client   # Client
```

```javascript
// Socket.io Server
import { createServer } from 'http';
import { Server } from 'socket.io';
import express from 'express';

const app = express();
const httpServer = createServer(app);

const io = new Server(httpServer, {
  cors: {
    origin: ['http://localhost:3001', 'https://myapp.com'],
    methods: ['GET', 'POST'],
    credentials: true,
  },
  
  // Transports (fallback order)
  transports: ['websocket', 'polling'],
  
  // Ping/pong for detecting disconnections
  pingTimeout: 20000,
  pingInterval: 25000,
});

io.on('connection', (socket) => {
  console.log('Socket connected:', socket.id);
  console.log('Transport:', socket.conn.transport.name);
  
  // Socket data
  socket.on('setUsername', (username) => {
    socket.data.username = username;
    console.log(`Socket ${socket.id} is now ${username}`);
  });
  
  // Basic message
  socket.on('message', (data) => {
    console.log('Received:', data);
    socket.emit('reply', { original: data, timestamp: Date.now() });
  });
  
  socket.on('disconnect', (reason) => {
    console.log(`Socket ${socket.id} disconnected: ${reason}`);
  });
});

httpServer.listen(3000);
```

```javascript
// Socket.io Client (Browser)
import { io } from 'socket.io-client';

const socket = io('https://example.com', {
  autoConnect: true,
  reconnection: true,
  reconnectionDelay: 1000,
  reconnectionDelayMax: 5000,
  reconnectionAttempts: Infinity,
  timeout: 20000,
  transports: ['websocket'], // บังคับใช้ WebSocket
});

socket.on('connect', () => {
  console.log('Connected! Socket ID:', socket.id);
  socket.emit('setUsername', 'สมชาย');
});

socket.on('disconnect', (reason) => {
  console.log('Disconnected:', reason);
  
  if (reason === 'io server disconnect') {
    socket.connect(); // reconnect manually
  }
});

socket.on('connect_error', (error) => {
  console.error('Connection error:', error.message);
});

socket.on('reply', (data) => {
  console.log('Got reply:', data);
});

// ส่ง message
socket.emit('message', { text: 'สวัสดี!', userId: 123 });
```

---

## ขั้นตอนที่ 1760: Rooms และ Namespaces

```javascript
// Rooms - group sockets
io.on('connection', (socket) => {
  // เข้า room
  socket.on('joinRoom', (roomId) => {
    socket.join(roomId);
    
    // แจ้งสมาชิกในห้อง
    socket.to(roomId).emit('userJoined', {
      userId: socket.id,
      username: socket.data.username,
    });
    
    // ส่งรายชื่อสมาชิกปัจจุบัน
    const roomSockets = io.sockets.adapter.rooms.get(roomId);
    socket.emit('roomInfo', {
      roomId,
      members: roomSockets ? [...roomSockets] : [],
    });
    
    console.log(`${socket.id} joined room: ${roomId}`);
  });
  
  // ออก room
  socket.on('leaveRoom', (roomId) => {
    socket.leave(roomId);
    
    socket.to(roomId).emit('userLeft', {
      userId: socket.id,
      username: socket.data.username,
    });
    
    console.log(`${socket.id} left room: ${roomId}`);
  });
  
  // ส่งข้อความใน room
  socket.on('roomMessage', ({ roomId, message }) => {
    const fullMessage = {
      id: Date.now(),
      from: socket.id,
      username: socket.data.username,
      message,
      timestamp: new Date().toISOString(),
    };
    
    // ส่งให้ทุกคนใน room (รวมตัวเอง)
    io.to(roomId).emit('message', fullMessage);
    
    // ส่งให้ทุกคนใน room (ไม่รวมตัวเอง)
    // socket.to(roomId).emit('message', fullMessage);
  });
  
  // ส่งให้ทุก room ที่ socket เป็นสมาชิก
  socket.on('globalMessage', (message) => {
    const rooms = [...socket.rooms].filter(r => r !== socket.id);
    rooms.forEach(room => {
      io.to(room).emit('message', {
        from: socket.id,
        message,
        broadcast: true,
      });
    });
  });
});

// Namespace - คล้าย room แต่ separate URL
const chatNS = io.of('/chat');
const gameNS = io.of('/game');

chatNS.on('connection', (socket) => {
  console.log('Chat namespace connected:', socket.id);
  socket.emit('welcome', 'ยินดีต้อนรับสู่ห้อง Chat');
});

gameNS.on('connection', (socket) => {
  console.log('Game namespace connected:', socket.id);
  socket.emit('welcome', 'ยินดีต้อนรับสู่ Game');
});

// Client เชื่อมต่อ namespace
const chatSocket = io('/chat');
const gameSocket = io('/game');
```

---

## ขั้นตอนที่ 1761: Broadcasting

```javascript
// Broadcasting Patterns

io.on('connection', (socket) => {
  // 1. ส่งให้ทุกคน (รวมตัวเอง)
  io.emit('event', data);
  
  // 2. ส่งให้ทุกคน (ไม่รวมตัวเอง)
  socket.broadcast.emit('event', data);
  
  // 3. ส่งให้คนใน room (รวมตัวเอง)
  io.to('room1').emit('event', data);
  
  // 4. ส่งให้คนใน room (ไม่รวมตัวเอง)
  socket.to('room1').emit('event', data);
  
  // 5. ส่งให้หลาย rooms
  io.to('room1').to('room2').emit('event', data);
  
  // 6. ส่งให้ specific socket
  io.to(socketId).emit('event', data);
  
  // 7. ส่งให้ทุกคนใน namespace
  io.of('/chat').emit('event', data);
  
  // 8. ไม่ส่งให้บาง rooms (except)
  io.except('room1').emit('event', data);
  socket.broadcast.except('room1').emit('event', data);
});

// Server-side broadcast (นอก connection handler)
function notifyUser(userId, event, data) {
  io.to(userId).emit(event, data);
}

function broadcastToRoom(roomId, event, data) {
  io.to(roomId).emit(event, data);
}

// ส่งหลาย events พร้อมกัน (volatile = ไม่ guarantee delivery)
socket.volatile.emit('live-data', fastUpdatingData);
```

---

## ขั้นตอนที่ 1762: Acknowledgments

```javascript
// Acknowledgments - ยืนยันว่า client ได้รับข้อความ

// Server
io.on('connection', (socket) => {
  // รอ acknowledgment จาก client
  socket.on('orderProduct', (orderData, callback) => {
    console.log('Order received:', orderData);
    
    // ทำการสั่งซื้อ
    processOrder(orderData)
      .then(order => {
        // ส่ง acknowledgment กลับ
        callback({ success: true, orderId: order.id });
      })
      .catch(error => {
        callback({ success: false, error: error.message });
      });
  });
  
  // ส่ง event พร้อมรอ ack จาก client (timeout 5 วินาที)
  socket.timeout(5000).emit('confirmDelivery', { orderId: 123 }, (err, response) => {
    if (err) {
      console.log('Client ไม่ตอบ (timeout)');
    } else {
      console.log('Client ยืนยัน:', response);
    }
  });
});

// Client
socket.emit('orderProduct', { 
  productId: 'P001', 
  quantity: 2,
  address: '123 ถนนสุขุมวิท'
}, (response) => {
  if (response.success) {
    console.log('สั่งซื้อสำเร็จ! Order ID:', response.orderId);
    showSuccessMessage(response.orderId);
  } else {
    console.error('สั่งซื้อล้มเหลว:', response.error);
    showErrorMessage(response.error);
  }
});

// Client รอ confirmation จาก server
socket.on('confirmDelivery', (data, callback) => {
  console.log('ยืนยันการส่งสินค้า order:', data.orderId);
  callback({ confirmed: true, timestamp: Date.now() });
});
```

---

## ขั้นตอนที่ 1763: Authentication กับ Socket.io

```javascript
// Socket.io Authentication Middleware

// Middleware ตรวจสอบ token
io.use(async (socket, next) => {
  try {
    const token = socket.handshake.auth.token || 
                  socket.handshake.headers.authorization?.replace('Bearer ', '');
    
    if (!token) {
      return next(new Error('Authentication required'));
    }
    
    const user = await verifyJWT(token);
    
    // เก็บ user data ใน socket
    socket.data.user = user;
    socket.data.userId = user.id;
    
    next();
  } catch (error) {
    next(new Error('Invalid token: ' + error.message));
  }
});

// ใช้งานใน connection handler
io.on('connection', (socket) => {
  const user = socket.data.user;
  console.log(`User ${user.name} (${user.id}) connected`);
  
  // User-specific room
  socket.join(`user:${user.id}`);
  
  // Role-based events
  if (user.role === 'admin') {
    socket.join('admins');
  }
});

// Client ส่ง token
const socket = io('https://example.com', {
  auth: {
    token: localStorage.getItem('authToken'),
  },
});

socket.on('connect_error', (err) => {
  if (err.message === 'Authentication required') {
    redirectToLogin();
  }
});

// Refresh token
socket.on('tokenExpired', () => {
  refreshToken().then(newToken => {
    socket.auth = { token: newToken };
    socket.connect(); // Reconnect กับ token ใหม่
  });
});
```

---

## ขั้นตอนที่ 1764: Live Chat Application

```javascript
// Complete Live Chat Application

// Server: chat-server.js
import { createServer } from 'http';
import { Server } from 'socket.io';
import express from 'express';

const app = express();
const httpServer = createServer(app);
const io = new Server(httpServer, { cors: { origin: '*' } });

// In-memory storage (ใช้ Redis ใน production)
const rooms = new Map();
const userSockets = new Map(); // userId -> Set<socketId>

function getRoom(roomId) {
  if (!rooms.has(roomId)) {
    rooms.set(roomId, {
      id: roomId,
      messages: [],
      members: new Set(),
    });
  }
  return rooms.get(roomId);
}

io.on('connection', (socket) => {
  let currentUser = null;
  let currentRooms = new Set();
  
  // Login
  socket.on('login', ({ userId, username }) => {
    currentUser = { id: userId, username };
    socket.data.user = currentUser;
    
    // Track user sockets (สำหรับ online status)
    if (!userSockets.has(userId)) {
      userSockets.set(userId, new Set());
    }
    userSockets.get(userId).add(socket.id);
    
    // Broadcast online status
    io.emit('userOnline', { userId, username });
    
    socket.emit('loginSuccess', { userId, username });
  });
  
  // Join Room
  socket.on('joinRoom', async ({ roomId }) => {
    if (!currentUser) return socket.emit('error', { message: 'Please login first' });
    
    const room = getRoom(roomId);
    
    socket.join(roomId);
    room.members.add(currentUser.id);
    currentRooms.add(roomId);
    
    // ส่ง message history
    socket.emit('roomHistory', {
      roomId,
      messages: room.messages.slice(-50), // 50 messages ล่าสุด
      members: [...room.members],
    });
    
    // แจ้งสมาชิกในห้อง
    socket.to(roomId).emit('userJoinedRoom', {
      roomId,
      user: currentUser,
    });
  });
  
  // Send Message
  socket.on('sendMessage', ({ roomId, message, type = 'text' }) => {
    if (!currentUser) return;
    
    const room = getRoom(roomId);
    
    const msg = {
      id: `msg_${Date.now()}_${Math.random().toString(36).substr(2, 9)}`,
      roomId,
      userId: currentUser.id,
      username: currentUser.username,
      message,
      type,
      timestamp: new Date().toISOString(),
      readBy: [currentUser.id],
    };
    
    room.messages.push(msg);
    
    // เก็บแค่ 1000 messages
    if (room.messages.length > 1000) {
      room.messages = room.messages.slice(-1000);
    }
    
    // Broadcast ไปทุกคนในห้อง
    io.to(roomId).emit('newMessage', msg);
  });
  
  // Typing Indicator
  socket.on('typing', ({ roomId, isTyping }) => {
    socket.to(roomId).emit('userTyping', {
      userId: currentUser?.id,
      username: currentUser?.username,
      isTyping,
    });
  });
  
  // Message Read Receipt
  socket.on('markRead', ({ roomId, messageId }) => {
    const room = getRoom(roomId);
    const msg = room.messages.find(m => m.id === messageId);
    
    if (msg && !msg.readBy.includes(currentUser.id)) {
      msg.readBy.push(currentUser.id);
      io.to(roomId).emit('messageRead', {
        messageId,
        userId: currentUser.id,
      });
    }
  });
  
  // Disconnect
  socket.on('disconnect', () => {
    if (!currentUser) return;
    
    const userSocketSet = userSockets.get(currentUser.id);
    userSocketSet?.delete(socket.id);
    
    if (!userSocketSet?.size) {
      userSockets.delete(currentUser.id);
      io.emit('userOffline', { userId: currentUser.id });
    }
    
    // Leave all rooms
    currentRooms.forEach(roomId => {
      const room = rooms.get(roomId);
      if (room) {
        room.members.delete(currentUser.id);
        socket.to(roomId).emit('userLeftRoom', {
          roomId,
          user: currentUser,
        });
      }
    });
  });
});

httpServer.listen(3000);
```

---

## ขั้นตอนที่ 1765: Real-time Notifications

```javascript
// Notification System
// server/notifications.js

import { Server } from 'socket.io';

export class NotificationService {
  constructor(io) {
    this.io = io;
    this.userSockets = new Map(); // userId -> socketId
  }
  
  registerUser(userId, socketId) {
    // User อาจมีหลาย connections (หลาย devices)
    if (!this.userSockets.has(userId)) {
      this.userSockets.set(userId, new Set());
    }
    this.userSockets.get(userId).add(socketId);
  }
  
  unregisterUser(userId, socketId) {
    this.userSockets.get(userId)?.delete(socketId);
    if (!this.userSockets.get(userId)?.size) {
      this.userSockets.delete(userId);
    }
  }
  
  // ส่ง notification ให้ specific user
  async notifyUser(userId, notification) {
    const notificationData = {
      id: `notif_${Date.now()}`,
      ...notification,
      timestamp: new Date().toISOString(),
      read: false,
    };
    
    // บันทึกลง database
    await saveNotificationToDB(userId, notificationData);
    
    // ส่ง real-time ถ้า user online
    this.io.to(`user:${userId}`).emit('notification', notificationData);
    
    return notificationData;
  }
  
  // ส่ง notification ให้ multiple users
  async notifyUsers(userIds, notification) {
    return Promise.all(userIds.map(id => this.notifyUser(id, notification)));
  }
  
  // Broadcast ทุกคน
  broadcastAll(notification) {
    this.io.emit('broadcast', {
      id: `broadcast_${Date.now()}`,
      ...notification,
      timestamp: new Date().toISOString(),
    });
  }
}

// ใช้งาน
io.on('connection', (socket) => {
  socket.on('authenticate', (userId) => {
    socket.join(`user:${userId}`);
    notificationService.registerUser(userId, socket.id);
  });
  
  socket.on('disconnect', () => {
    if (socket.userId) {
      notificationService.unregisterUser(socket.userId, socket.id);
    }
  });
});

// ส่ง notification จาก API
app.post('/api/admin/notify', async (req, res) => {
  const { userId, type, title, body } = req.body;
  
  const notification = await notificationService.notifyUser(userId, {
    type,
    title,
    body,
  });
  
  res.json({ sent: true, notification });
});
```

---

## ขั้นตอนที่ 1766: Collaborative Editing Concept

```javascript
// Operational Transformation (OT) สำหรับ collaborative editing
// (Simplified version)

class CollaborativeDocument {
  constructor(id, initialContent = '') {
    this.id = id;
    this.content = initialContent;
    this.version = 0;
    this.pendingOps = [];
  }
  
  apply(operation) {
    if (operation.type === 'insert') {
      const { position, text } = operation;
      this.content = 
        this.content.slice(0, position) + 
        text + 
        this.content.slice(position);
    } else if (operation.type === 'delete') {
      const { position, length } = operation;
      this.content = 
        this.content.slice(0, position) + 
        this.content.slice(position + length);
    }
    
    this.version++;
    return this;
  }
  
  // Transform operation ที่เข้ามาใหม่ เทียบกับ operations ที่เกิดขึ้นแล้ว
  transform(op, againstOp) {
    if (op.type === 'insert' && againstOp.type === 'insert') {
      if (againstOp.position <= op.position) {
        return { ...op, position: op.position + againstOp.text.length };
      }
    }
    
    if (op.type === 'delete' && againstOp.type === 'insert') {
      if (againstOp.position <= op.position) {
        return { ...op, position: op.position + againstOp.text.length };
      }
    }
    
    // ... more transformation rules
    return op;
  }
}

// Socket.io handler สำหรับ collaborative editing
const documents = new Map();

io.on('connection', (socket) => {
  socket.on('openDocument', ({ docId }) => {
    let doc = documents.get(docId);
    if (!doc) {
      doc = new CollaborativeDocument(docId);
      documents.set(docId, doc);
    }
    
    socket.join(`doc:${docId}`);
    socket.emit('documentState', {
      content: doc.content,
      version: doc.version,
    });
  });
  
  socket.on('operation', ({ docId, operation, clientVersion }) => {
    const doc = documents.get(docId);
    if (!doc) return;
    
    // Transform operation ถ้า client version ไม่ตรงกับ server
    let op = operation;
    const opsSince = doc.pendingOps.slice(clientVersion);
    
    for (const historicOp of opsSince) {
      op = doc.transform(op, historicOp);
    }
    
    // Apply operation
    doc.apply(op);
    doc.pendingOps.push(op);
    
    // Broadcast ไปยัง clients อื่นใน document
    socket.to(`doc:${docId}`).emit('remoteOperation', {
      operation: op,
      version: doc.version,
      userId: socket.data.userId,
    });
    
    // Acknowledge ไปยัง sender
    socket.emit('operationAck', { 
      version: doc.version,
      operation: op,
    });
  });
  
  // Cursor position sharing
  socket.on('cursorMove', ({ docId, position }) => {
    socket.to(`doc:${docId}`).emit('remoteCursor', {
      userId: socket.data.userId,
      username: socket.data.username,
      position,
    });
  });
});
```

---

## ขั้นตอนที่ 1767: Scaling WebSocket Servers

```javascript
// Socket.io กับ Redis Adapter (สำหรับหลาย servers)
import { createClient } from 'redis';
import { createAdapter } from '@socket.io/redis-adapter';

const pubClient = createClient({ url: process.env.REDIS_URL });
const subClient = pubClient.duplicate();

await Promise.all([pubClient.connect(), subClient.connect()]);

io.adapter(createAdapter(pubClient, subClient));

// ตอนนี้ Socket.io จะ sync events ระหว่าง servers ผ่าน Redis
// Server 1 socket.to('room1').emit(...) -> Redis pub -> Server 2 ส่งให้ clients ใน room1
```

```javascript
// Horizontal Scaling Architecture
/*
Load Balancer (Nginx/AWS ALB)
├── Server 1 (WebSocket)
│   └── Redis Adapter -> Redis Pub/Sub
├── Server 2 (WebSocket)  
│   └── Redis Adapter -> Redis Pub/Sub
└── Server 3 (WebSocket)
    └── Redis Adapter -> Redis Pub/Sub

Note: ต้องใช้ Sticky Sessions (same client -> same server)
      หรือใช้ HTTP upgrade ที่ load balancer
*/

// Nginx config สำหรับ WebSocket + Sticky Sessions
/*
upstream websocket_servers {
    ip_hash; # Sticky sessions based on IP
    server server1:3000;
    server server2:3000;
    server server3:3000;
}

server {
    location /socket.io/ {
        proxy_pass http://websocket_servers;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
*/
```

---

## ขั้นตอนที่ 1768: Building Real-time Dashboard

```javascript
// Real-time Analytics Dashboard
// server.js
import { createServer } from 'http';
import { Server } from 'socket.io';
import express from 'express';

const app = express();
const io = new Server(createServer(app));

// Metrics state
const metrics = {
  activeUsers: 0,
  requestsPerSecond: 0,
  averageResponseTime: 0,
  errorRate: 0,
  cpuUsage: 0,
  memoryUsage: 0,
};

// Simulate metric updates
setInterval(() => {
  metrics.activeUsers = Math.floor(100 + Math.random() * 900);
  metrics.requestsPerSecond = Math.floor(50 + Math.random() * 450);
  metrics.averageResponseTime = Math.floor(20 + Math.random() * 180);
  metrics.errorRate = +(Math.random() * 5).toFixed(2);
  metrics.cpuUsage = +(20 + Math.random() * 60).toFixed(1);
  metrics.memoryUsage = +(40 + Math.random() * 40).toFixed(1);
  
  // Broadcast ไปยัง dashboard viewers
  io.to('dashboard').emit('metricsUpdate', {
    ...metrics,
    timestamp: Date.now(),
  });
}, 1000);

io.on('connection', (socket) => {
  // Dashboard viewers
  socket.on('watchDashboard', () => {
    socket.join('dashboard');
    
    // ส่ง current metrics ทันที
    socket.emit('metricsUpdate', {
      ...metrics,
      timestamp: Date.now(),
    });
  });
  
  socket.on('unwatchDashboard', () => {
    socket.leave('dashboard');
  });
});
```

```javascript
// Client Dashboard Component
'use client';
import { useEffect, useState, useRef } from 'react';
import { io } from 'socket.io-client';

export default function RealTimeDashboard() {
  const [metrics, setMetrics] = useState(null);
  const [history, setHistory] = useState([]);
  const socketRef = useRef(null);
  
  useEffect(() => {
    const socket = io('https://api.example.com');
    socketRef.current = socket;
    
    socket.on('connect', () => {
      socket.emit('watchDashboard');
    });
    
    socket.on('metricsUpdate', (data) => {
      setMetrics(data);
      setHistory(prev => {
        const newHistory = [...prev, data];
        return newHistory.slice(-60); // เก็บ 60 seconds
      });
    });
    
    return () => {
      socket.emit('unwatchDashboard');
      socket.disconnect();
    };
  }, []);
  
  if (!metrics) return <div>กำลังโหลด...</div>;
  
  return (
    <div className="dashboard">
      <div className="metric-card">
        <h3>Active Users</h3>
        <div className="value">{metrics.activeUsers.toLocaleString()}</div>
        <div className="trend">
          {history.length > 1 
            ? metrics.activeUsers > history[history.length - 2]?.activeUsers 
              ? '↑' 
              : '↓'
            : '-'
          }
        </div>
      </div>
      
      <div className="metric-card">
        <h3>Requests/sec</h3>
        <div className="value">{metrics.requestsPerSecond}</div>
      </div>
      
      <div className="metric-card">
        <h3>Avg Response Time</h3>
        <div className="value" style={{ color: metrics.averageResponseTime > 100 ? 'red' : 'green' }}>
          {metrics.averageResponseTime}ms
        </div>
      </div>
      
      <div className="metric-card">
        <h3>Error Rate</h3>
        <div className="value" style={{ color: metrics.errorRate > 3 ? 'red' : 'green' }}>
          {metrics.errorRate}%
        </div>
      </div>
      
      <div className="metric-card">
        <h3>CPU Usage</h3>
        <div className="progress-bar">
          <div style={{ width: `${metrics.cpuUsage}%`, background: metrics.cpuUsage > 80 ? 'red' : 'blue' }} />
        </div>
        <div>{metrics.cpuUsage}%</div>
      </div>
    </div>
  );
}
```

---

## ขั้นตอนที่ 1769: WebRTC Basics

```javascript
// WebRTC สำหรับ Video Call (แบบง่าย)
// Signaling server ใช้ Socket.io

// Server (Signaling)
io.on('connection', (socket) => {
  socket.on('joinCall', ({ roomId }) => {
    socket.join(roomId);
    
    const room = io.sockets.adapter.rooms.get(roomId);
    const numClients = room ? room.size : 0;
    
    if (numClients === 1) {
      socket.emit('created', roomId);
    } else if (numClients === 2) {
      socket.to(roomId).emit('join', roomId);
      socket.emit('joined', roomId);
    } else {
      socket.emit('full', roomId);
    }
  });
  
  // Relay SDP offer/answer และ ICE candidates
  socket.on('offer', ({ roomId, sdp }) => {
    socket.to(roomId).emit('offer', { sdp, from: socket.id });
  });
  
  socket.on('answer', ({ roomId, sdp }) => {
    socket.to(roomId).emit('answer', { sdp, from: socket.id });
  });
  
  socket.on('candidate', ({ roomId, candidate }) => {
    socket.to(roomId).emit('candidate', { candidate, from: socket.id });
  });
});

// Client
class VideoCall {
  constructor(socket) {
    this.socket = socket;
    this.localStream = null;
    this.peerConnection = null;
  }
  
  async startCall(roomId) {
    // ขอ permission กล้อง/ไมค์
    this.localStream = await navigator.mediaDevices.getUserMedia({
      video: true,
      audio: true,
    });
    
    document.getElementById('localVideo').srcObject = this.localStream;
    
    this.setupPeerConnection();
    this.socket.emit('joinCall', { roomId });
  }
  
  setupPeerConnection() {
    this.peerConnection = new RTCPeerConnection({
      iceServers: [
        { urls: 'stun:stun.l.google.com:19302' },
        { urls: 'stun:stun1.l.google.com:19302' },
      ],
    });
    
    // เพิ่ม local stream
    this.localStream.getTracks().forEach(track => {
      this.peerConnection.addTrack(track, this.localStream);
    });
    
    // Handle remote stream
    this.peerConnection.ontrack = (event) => {
      document.getElementById('remoteVideo').srcObject = event.streams[0];
    };
    
    // Handle ICE candidates
    this.peerConnection.onicecandidate = (event) => {
      if (event.candidate) {
        this.socket.emit('candidate', {
          roomId: this.roomId,
          candidate: event.candidate,
        });
      }
    };
  }
  
  async createOffer() {
    const offer = await this.peerConnection.createOffer();
    await this.peerConnection.setLocalDescription(offer);
    
    this.socket.emit('offer', {
      roomId: this.roomId,
      sdp: offer,
    });
  }
  
  async handleOffer(sdp) {
    await this.peerConnection.setRemoteDescription(sdp);
    
    const answer = await this.peerConnection.createAnswer();
    await this.peerConnection.setLocalDescription(answer);
    
    this.socket.emit('answer', {
      roomId: this.roomId,
      sdp: answer,
    });
  }
}
```

---

## ขั้นตอนที่ 1770: Production Best Practices

```javascript
// Production WebSocket Best Practices

// 1. Connection limits และ rate limiting
const connectionRateLimiter = new Map();

io.use((socket, next) => {
  const ip = socket.handshake.headers['x-forwarded-for'] || socket.handshake.address;
  const now = Date.now();
  
  if (!connectionRateLimiter.has(ip)) {
    connectionRateLimiter.set(ip, { count: 0, resetAt: now + 60000 });
  }
  
  const limiter = connectionRateLimiter.get(ip);
  
  if (now > limiter.resetAt) {
    limiter.count = 0;
    limiter.resetAt = now + 60000;
  }
  
  limiter.count++;
  
  if (limiter.count > 100) { // max 100 connections/minute per IP
    return next(new Error('Too many connections'));
  }
  
  next();
});

// 2. Message size validation
io.use((socket, next) => {
  socket.use(([event, ...args], next) => {
    const size = JSON.stringify(args).length;
    
    if (size > 100 * 1024) { // 100KB max
      return next(new Error('Message too large'));
    }
    
    next();
  });
  
  next();
});

// 3. Graceful shutdown
process.on('SIGTERM', () => {
  console.log('SIGTERM received, shutting down gracefully...');
  
  io.close(() => {
    console.log('Socket.io server closed');
    httpServer.close(() => {
      console.log('HTTP server closed');
      process.exit(0);
    });
  });
  
  // Force shutdown after 30 seconds
  setTimeout(() => {
    process.exit(1);
  }, 30000);
});

// 4. Monitoring
io.engine.on('connection_error', (err) => {
  console.error('Connection error:', {
    req: err.req?.url,
    code: err.code,
    message: err.message,
    context: err.context,
  });
  
  // ส่งไป monitoring system
  metrics.increment('socket.connection_error', { code: err.code });
});

// Track metrics
let connectionCount = 0;
let messageCount = 0;

io.on('connection', (socket) => {
  connectionCount++;
  metrics.gauge('socket.connections', connectionCount);
  
  socket.onAny(() => {
    messageCount++;
    metrics.increment('socket.messages');
  });
  
  socket.on('disconnect', () => {
    connectionCount--;
    metrics.gauge('socket.connections', connectionCount);
  });
});
```

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: Live Chat Room
สร้าง full-stack chat application ที่:
- Multiple rooms
- User authentication
- Typing indicators
- Read receipts
- File sharing (ใช้ Socket.io ส่ง base64 encoded)
- Emoji reactions

### แบบฝึกหัดที่ 2: Real-time Kanban Board
สร้าง collaborative Kanban board ที่:
- Drag-and-drop cards
- Real-time sync ระหว่าง users
- Show who's editing what
- Conflict resolution
- undo/redo

### แบบฝึกหัดที่ 3: Live Sports Dashboard
สร้าง live sports dashboard ที่:
- แสดง live scores
- Real-time updates ทุกนาที
- Push notifications เมื่อมีประตู
- Multiple matches พร้อมกัน
- Historical data

### แบบฝึกหัดที่ 4: Collaborative Code Editor
สร้าง collaborative code editor ที่:
- Real-time syntax highlighting
- Multiple cursors (แต่ละ user สีต่างกัน)
- Operational Transformation สำหรับ conflict resolution
- Chat sidebar

### แบบฝึกหัดที่ 5: Real-time Auction System
สร้าง live auction ที่:
- Real-time bid updates
- Countdown timer
- Auto-increment bidding
- Notification เมื่อถูก outbid
- Winner announcement

---

## สรุป

| Technology | Use Case | Pros | Cons |
|-----------|----------|------|------|
| Short Polling | Simple periodic updates | ง่าย, HTTP | Server load |
| Long Polling | Infrequent updates | HTTP-friendly | Complex, latency |
| SSE | Server → Client only | Simple, auto-reconnect | One-directional |
| WebSocket | Bidirectional, real-time | Low latency | Complex scaling |
| Socket.io | Full-featured real-time | Easy, fallbacks | Overhead |
| WebRTC | P2P, video/audio | Low latency | Complex NAT |

ในส่วนถัดไปเราจะเรียนเรื่อง System Design สำหรับ JavaScript Developer!
