# Part 100: โปรเจค World-Class: SaaS Application (Steps 1971-2000+)

## บทนำ

ในบทสุดท้ายนี้ เราจะสร้าง **DevCollab** - Real-time Collaborative Code Editor ระดับ production ที่ครบทุกฟีเจอร์ของ SaaS application สมัยใหม่ โปรเจคนี้จะรวมความรู้ทั้งหมดที่เรียนมาตลอดคอร์สนี้

---

## Step 1971: System Architecture

### Architecture Overview

```
┌──────────────────────────────────────────────────────────────────────┐
│                        DevCollab Architecture                        │
├──────────────────────────────────────────────────────────────────────┤
│                                                                      │
│  Client (Browser)                                                    │
│  ┌─────────────────────────────────────┐                           │
│  │ Next.js 14 + TypeScript             │                           │
│  │ ┌──────────────┐ ┌───────────────┐  │                           │
│  │ │  CodeMirror 6 │ │  React UI     │  │                           │
│  │ │  + Yjs CRDT  │ │  + TailwindCSS│  │                           │
│  │ └──────┬───────┘ └───────────────┘  │                           │
│  └────────┼────────────────────────────┘                           │
│           │ WebSocket / REST                                         │
│  ┌────────▼────────────────────────────────────┐                   │
│  │              API Gateway / Load Balancer     │                   │
│  └──────────────────────┬──────────────────────┘                   │
│                         │                                            │
│  ┌──────────────────────▼──────────────────────┐                   │
│  │              Node.js API Server              │                   │
│  │  ┌──────────┐ ┌──────────┐ ┌─────────────┐ │                   │
│  │  │  REST    │ │WebSocket │ │  Worker     │ │                   │
│  │  │  Routes  │ │ Server   │ │  (Queue)    │ │                   │
│  │  └────┬─────┘ └────┬─────┘ └──────┬──────┘ │                   │
│  └───────┼─────────────┼──────────────┼────────┘                   │
│          │             │              │                              │
│  ┌───────▼─────────────▼──────────────▼────────┐                   │
│  │              Data Layer                      │                   │
│  │  ┌──────────┐ ┌──────────┐ ┌─────────────┐ │                   │
│  │  │PostgreSQL│ │  Redis   │ │    S3       │ │                   │
│  │  │ + Prisma │ │ (Cache + │ │  (Files)    │ │                   │
│  │  │          │ │  Pub/Sub)│ │             │ │                   │
│  │  └──────────┘ └──────────┘ └─────────────┘ │                   │
│  └─────────────────────────────────────────────┘                   │
│                                                                      │
│  External Services                                                   │
│  ┌──────────┐ ┌──────────┐ ┌──────────┐ ┌────────────┐           │
│  │ GitHub   │ │  Claude  │ │  Stripe  │ │   SendGrid │           │
│  │  OAuth   │ │   API    │ │ Billing  │ │   Email    │           │
│  └──────────┘ └──────────┘ └──────────┘ └────────────┘           │
└──────────────────────────────────────────────────────────────────────┘
```

---

## Step 1972: Database Schema

### Prisma Schema

```prisma
// prisma/schema.prisma
generator client {
  provider        = "prisma-client-js"
  previewFeatures = ["fullTextSearch", "fullTextIndex"]
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// User & Auth
model User {
  id            String    @id @default(cuid())
  email         String    @unique
  name          String
  username      String    @unique
  avatarUrl     String?
  bio           String?
  githubId      String?   @unique
  
  role          UserRole  @default(USER)
  
  // Subscription
  stripeCustomerId     String?  @unique
  stripeSubscriptionId String?  @unique
  subscriptionStatus   SubscriptionStatus @default(FREE)
  subscriptionPlanId   String?
  
  // Timestamps
  createdAt     DateTime  @default(now())
  updatedAt     DateTime  @updatedAt
  lastActiveAt  DateTime?
  
  // Relations
  sessions      Session[]
  teams         TeamMember[]
  ownedTeams    Team[]    @relation("TeamOwner")
  projects      ProjectMember[]
  ownedProjects Project[] @relation("ProjectOwner")
  documents     Document[] @relation("DocumentOwner")
  aiUsage       AIUsage[]
  
  @@index([email])
  @@index([username])
}

model Session {
  id           String   @id @default(cuid())
  userId       String
  token        String   @unique
  refreshToken String   @unique
  expiresAt    DateTime
  deviceInfo   Json?
  ipAddress    String?
  
  createdAt    DateTime @default(now())
  
  user         User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  
  @@index([token])
  @@index([userId])
}

// Team
model Team {
  id          String   @id @default(cuid())
  name        String
  slug        String   @unique
  description String?
  avatarUrl   String?
  ownerId     String
  
  // Subscription
  stripeCustomerId     String?  @unique
  subscriptionStatus   SubscriptionStatus @default(FREE)
  
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  
  owner       User     @relation("TeamOwner", fields: [ownerId], references: [id])
  members     TeamMember[]
  projects    Project[]
  
  @@index([slug])
}

model TeamMember {
  id       String     @id @default(cuid())
  teamId   String
  userId   String
  role     TeamRole   @default(MEMBER)
  
  invitedAt  DateTime @default(now())
  joinedAt   DateTime?
  
  team     Team       @relation(fields: [teamId], references: [id], onDelete: Cascade)
  user     User       @relation(fields: [userId], references: [id], onDelete: Cascade)
  
  @@unique([teamId, userId])
  @@index([teamId])
  @@index([userId])
}

// Project
model Project {
  id          String        @id @default(cuid())
  name        String
  description String?
  ownerId     String
  teamId      String?
  visibility  Visibility    @default(PRIVATE)
  language    String        @default("javascript")
  
  // Git integration
  githubRepoId    String?
  githubRepoName  String?
  githubDefaultBranch String?
  
  createdAt   DateTime     @default(now())
  updatedAt   DateTime     @updatedAt
  
  owner       User          @relation("ProjectOwner", fields: [ownerId], references: [id])
  team        Team?         @relation(fields: [teamId], references: [id])
  members     ProjectMember[]
  documents   Document[]
  
  @@index([ownerId])
  @@index([teamId])
}

model ProjectMember {
  id        String      @id @default(cuid())
  projectId String
  userId    String
  role      ProjectRole @default(VIEWER)
  
  addedAt   DateTime    @default(now())
  
  project   Project     @relation(fields: [projectId], references: [id], onDelete: Cascade)
  user      User        @relation(fields: [userId], references: [id], onDelete: Cascade)
  
  @@unique([projectId, userId])
}

// Document (Code File)
model Document {
  id          String     @id @default(cuid())
  projectId   String
  ownerId     String
  name        String
  path        String
  language    String     @default("javascript")
  content     String     @db.Text
  
  // CRDT state (Yjs)
  yState      Bytes?
  
  // Version control
  version     Int        @default(1)
  
  // S3 for large files
  s3Key       String?
  
  createdAt   DateTime   @default(now())
  updatedAt   DateTime   @updatedAt
  
  project     Project    @relation(fields: [projectId], references: [id], onDelete: Cascade)
  owner       User       @relation("DocumentOwner", fields: [ownerId], references: [id])
  versions    DocumentVersion[]
  sessions    CollabSession[]
  comments    Comment[]
  
  @@unique([projectId, path])
  @@index([projectId])
  @@index([ownerId])
}

model DocumentVersion {
  id          String   @id @default(cuid())
  documentId  String
  version     Int
  content     String   @db.Text
  authorId    String
  message     String?
  
  createdAt   DateTime @default(now())
  
  document    Document @relation(fields: [documentId], references: [id], onDelete: Cascade)
  
  @@unique([documentId, version])
  @@index([documentId])
}

// Collaboration
model CollabSession {
  id          String   @id @default(cuid())
  documentId  String
  userId      String
  socketId    String   @unique
  cursor      Json?    // { line, col }
  selection   Json?    // { from, to }
  color       String
  
  connectedAt  DateTime @default(now())
  lastActiveAt DateTime @default(now())
  
  document    Document @relation(fields: [documentId], references: [id], onDelete: Cascade)
  
  @@index([documentId])
  @@index([userId])
}

// Chat
model ChatRoom {
  id        String    @id @default(cuid())
  projectId String    @unique
  
  messages  ChatMessage[]
}

model ChatMessage {
  id          String    @id @default(cuid())
  roomId      String
  userId      String
  content     String
  type        MessageType @default(TEXT)
  
  createdAt   DateTime  @default(now())
  editedAt    DateTime?
  
  room        ChatRoom  @relation(fields: [roomId], references: [id], onDelete: Cascade)
  
  @@index([roomId])
  @@index([userId])
}

// AI Usage
model AIUsage {
  id           String   @id @default(cuid())
  userId       String
  documentId   String?
  type         AIRequestType
  promptTokens Int
  completionTokens Int
  cost         Decimal  @db.Decimal(10, 6)
  
  createdAt    DateTime @default(now())
  
  user         User     @relation(fields: [userId], references: [id], onDelete: Cascade)
  
  @@index([userId])
  @@index([createdAt])
}

// Comment
model Comment {
  id         String    @id @default(cuid())
  documentId String
  userId     String
  content    String
  lineNumber Int?
  resolved   Boolean   @default(false)
  
  createdAt  DateTime  @default(now())
  updatedAt  DateTime  @updatedAt
  
  document   Document  @relation(fields: [documentId], references: [id], onDelete: Cascade)
  
  @@index([documentId])
}

// Enums
enum UserRole {
  USER
  ADMIN
  SUPERADMIN
}

enum TeamRole {
  OWNER
  ADMIN
  MEMBER
  VIEWER
}

enum ProjectRole {
  OWNER
  EDITOR
  VIEWER
}

enum Visibility {
  PUBLIC
  PRIVATE
  TEAM
}

enum SubscriptionStatus {
  FREE
  PRO
  TEAM
  ENTERPRISE
  PAST_DUE
  CANCELED
}

enum MessageType {
  TEXT
  CODE
  SYSTEM
  AI
}

enum AIRequestType {
  COMPLETION
  EXPLAIN
  REFACTOR
  DEBUG
  REVIEW
}
```

---

## Step 1973: Backend API

### Express + TypeScript API Server

```typescript
// src/server.ts
import express from 'express';
import { createServer } from 'http';
import { Server as SocketServer } from 'socket.io';
import cors from 'cors';
import helmet from 'helmet';
import compression from 'compression';
import morgan from 'morgan';
import { createBullBoard } from '@bull-board/api';
import { BullMQAdapter } from '@bull-board/api/bullMQAdapter';
import { ExpressAdapter } from '@bull-board/express';

import { router as apiRouter } from './routes';
import { socketHandler } from './socket';
import { errorHandler } from './middleware/errorHandler';
import { authenticate } from './middleware/auth';
import { rateLimiter } from './middleware/rateLimiter';
import { requestLogger } from './middleware/requestLogger';
import { emailQueue, aiQueue } from './queues';
import { prisma } from './lib/prisma';
import { redis } from './lib/redis';
import { logger } from './lib/logger';

const app = express();
const httpServer = createServer(app);

// Socket.IO
const io = new SocketServer(httpServer, {
  cors: {
    origin: process.env.FRONTEND_URL || 'http://localhost:3000',
    credentials: true
  },
  transports: ['websocket', 'polling'],
  pingTimeout: 10000,
  pingInterval: 5000
});

// Middleware
app.use(helmet({ contentSecurityPolicy: false }));
app.use(cors({
  origin: process.env.FRONTEND_URL,
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization']
}));
app.use(compression());
app.use(express.json({ limit: '10mb' }));
app.use(express.urlencoded({ extended: true }));
app.use(requestLogger());
app.use(rateLimiter);

// Health check
app.get('/health', async (req, res) => {
  try {
    await prisma.$queryRaw`SELECT 1`;
    await redis.ping();

    res.json({
      status: 'healthy',
      timestamp: new Date().toISOString(),
      uptime: process.uptime(),
      version: process.env.APP_VERSION || '1.0.0'
    });
  } catch (error) {
    res.status(503).json({ status: 'unhealthy', error: String(error) });
  }
});

// Bull Board (job queue dashboard)
if (process.env.NODE_ENV !== 'production') {
  const { router: bullBoardRouter } = createBullBoard({
    queues: [
      new BullMQAdapter(emailQueue),
      new BullMQAdapter(aiQueue)
    ],
    serverAdapter: new ExpressAdapter()
  });
  app.use('/admin/queues', bullBoardRouter);
}

// Routes
app.use('/api/v1', apiRouter);

// Socket.IO handlers
socketHandler(io);

// Error handler
app.use(errorHandler);

const PORT = parseInt(process.env.PORT || '3000', 10);

httpServer.listen(PORT, () => {
  logger.info({ port: PORT }, 'Server started');
});

// Graceful shutdown
process.on('SIGTERM', async () => {
  logger.info('SIGTERM received. Shutting down gracefully...');
  
  httpServer.close(async () => {
    await prisma.$disconnect();
    await redis.quit();
    logger.info('Server closed');
    process.exit(0);
  });
  
  // Force close after 10 seconds
  setTimeout(() => {
    logger.error('Could not close connections in time, forcefully shutting down');
    process.exit(1);
  }, 10000);
});

export { app, io };
```

```typescript
// src/routes/documents.ts
import { Router } from 'express';
import { prisma } from '../lib/prisma';
import { authenticate } from '../middleware/auth';
import { authorize } from '../middleware/authorize';
import { validate } from '../middleware/validate';
import { DocumentSchema } from '../schemas/document';
import { S3Service } from '../lib/s3';
import { aiQueue } from '../queues';
import Anthropic from '@anthropic-ai/sdk';

const router = Router();
const anthropic = new Anthropic({ apiKey: process.env.ANTHROPIC_API_KEY });

// GET /documents/:id
router.get('/:id',
  authenticate,
  authorize('document', 'read'),
  async (req, res, next) => {
    try {
      const document = await prisma.document.findUnique({
        where: { id: req.params.id },
        include: {
          owner: { select: { id: true, name: true, avatarUrl: true } },
          project: { select: { id: true, name: true } }
        }
      });

      if (!document) {
        return res.status(404).json({ error: 'Document not found' });
      }

      // โหลด content จาก S3 ถ้าไฟล์ใหญ่
      let content = document.content;
      if (document.s3Key) {
        content = await S3Service.getContent(document.s3Key);
      }

      res.json({ ...document, content });
    } catch (error) {
      next(error);
    }
  }
);

// POST /documents
router.post('/',
  authenticate,
  validate(DocumentSchema.create),
  async (req, res, next) => {
    try {
      const { projectId, name, path, language, content } = req.body;

      // ตรวจสอบ permission
      const membership = await prisma.projectMember.findUnique({
        where: { projectId_userId: { projectId, userId: req.user.id } }
      });

      if (!membership || membership.role === 'VIEWER') {
        return res.status(403).json({ error: 'Insufficient permissions' });
      }

      // ตรวจสอบ path ซ้ำ
      const existing = await prisma.document.findUnique({
        where: { projectId_path: { projectId, path } }
      });

      if (existing) {
        return res.status(409).json({ error: 'File already exists at this path' });
      }

      // บันทึก content ใหญ่ใน S3
      let s3Key: string | undefined;
      let dbContent = content;

      if (content.length > 1_000_000) { // > 1MB
        s3Key = `documents/${projectId}/${path}`;
        await S3Service.upload(s3Key, content, 'text/plain');
        dbContent = ''; // อย่าบันทึกใน DB
      }

      const document = await prisma.document.create({
        data: {
          projectId,
          ownerId: req.user.id,
          name,
          path,
          language,
          content: dbContent,
          s3Key
        }
      });

      res.status(201).json(document);
    } catch (error) {
      next(error);
    }
  }
);

// PUT /documents/:id/content
router.put('/:id/content',
  authenticate,
  authorize('document', 'edit'),
  async (req, res, next) => {
    try {
      const { content, version } = req.body;

      // Optimistic concurrency control
      const current = await prisma.document.findUnique({
        where: { id: req.params.id },
        select: { version: true }
      });

      if (!current) {
        return res.status(404).json({ error: 'Document not found' });
      }

      if (current.version !== version) {
        return res.status(409).json({
          error: 'Conflict',
          currentVersion: current.version
        });
      }

      const updated = await prisma.document.update({
        where: { id: req.params.id, version }, // atomic check
        data: {
          content,
          version: { increment: 1 },
          updatedAt: new Date()
        }
      });

      // บันทึก version history
      await prisma.documentVersion.create({
        data: {
          documentId: req.params.id,
          version: updated.version,
          content,
          authorId: req.user.id
        }
      });

      res.json({ version: updated.version });
    } catch (error) {
      if (error.code === 'P2025') {
        return res.status(409).json({ error: 'Version conflict' });
      }
      next(error);
    }
  }
);

// POST /documents/:id/ai/complete
router.post('/:id/ai/complete',
  authenticate,
  async (req, res, next) => {
    try {
      const { code, language, cursor, context } = req.body;

      // ตรวจสอบ quota
      const usage = await checkAIQuota(req.user.id);
      if (!usage.canUse) {
        return res.status(429).json({
          error: 'AI quota exceeded',
          resetAt: usage.resetAt
        });
      }

      // Streaming response
      res.setHeader('Content-Type', 'text/event-stream');
      res.setHeader('Cache-Control', 'no-cache');
      res.setHeader('Connection', 'keep-alive');

      const stream = anthropic.messages.stream({
        model: 'claude-3-5-sonnet-20241022',
        max_tokens: 1024,
        system: `You are an expert ${language} developer. Complete the code at the cursor position. Return ONLY the completion text without any explanation or markdown.`,
        messages: [{
          role: 'user',
          content: `Complete this ${language} code:\n\n\`\`\`${language}\n${code}\n\`\`\`\n\nCursor is at position: ${cursor.line}:${cursor.col}\nContext: ${context || 'none'}`
        }]
      });

      let totalTokens = 0;

      stream
        .on('text', (text) => {
          res.write(`data: ${JSON.stringify({ text })}\n\n`);
        })
        .on('message', (message) => {
          totalTokens = message.usage.input_tokens + message.usage.output_tokens;
        })
        .on('end', async () => {
          res.write('data: [DONE]\n\n');
          res.end();

          // บันทึก usage
          await recordAIUsage(req.user.id, req.params.id, 'COMPLETION', totalTokens);
        })
        .on('error', (error) => {
          console.error('AI stream error:', error);
          res.write(`data: ${JSON.stringify({ error: 'AI error' })}\n\n`);
          res.end();
        });

    } catch (error) {
      next(error);
    }
  }
);

async function checkAIQuota(userId: string) {
  const user = await prisma.user.findUnique({
    where: { id: userId },
    select: { subscriptionStatus: true }
  });

  const limits = {
    FREE: 50,
    PRO: 1000,
    TEAM: 5000,
    ENTERPRISE: -1 // unlimited
  };

  const limit = limits[user?.subscriptionStatus || 'FREE'];
  if (limit === -1) return { canUse: true };

  const today = new Date();
  today.setHours(0, 0, 0, 0);

  const usage = await prisma.aIUsage.count({
    where: {
      userId,
      createdAt: { gte: today }
    }
  });

  return {
    canUse: usage < limit,
    used: usage,
    limit,
    resetAt: new Date(today.getTime() + 24 * 60 * 60 * 1000)
  };
}

async function recordAIUsage(
  userId: string,
  documentId: string,
  type: string,
  tokens: number
) {
  const cost = tokens * 0.000003; // $3 per 1M tokens estimate

  await prisma.aIUsage.create({
    data: {
      userId,
      documentId,
      type,
      promptTokens: Math.floor(tokens * 0.7),
      completionTokens: Math.floor(tokens * 0.3),
      cost
    }
  });
}

export { router };
```

---

## Step 1974: WebSocket Collaboration

### Real-time Collaboration ด้วย Socket.IO + Yjs

```typescript
// src/socket/index.ts
import { Server, Socket } from 'socket.io';
import * as Y from 'yjs';
import { encoding, decoding } from 'lib0';
import { redis } from '../lib/redis';
import { prisma } from '../lib/prisma';
import { verifyToken } from '../lib/auth';
import { logger } from '../lib/logger';

// Yjs documents cache
const yjsDocs = new Map<string, Y.Doc>();

// Cursor colors สำหรับ collaborators
const CURSOR_COLORS = [
  '#FF6B6B', '#4ECDC4', '#45B7D1', '#96CEB4',
  '#FFEAA7', '#DDA0DD', '#98FB98', '#87CEEB'
];

export function socketHandler(io: Server) {
  // Middleware: Authentication
  io.use(async (socket, next) => {
    const token = socket.handshake.auth.token;
    if (!token) {
      return next(new Error('Authentication required'));
    }

    try {
      const payload = verifyToken(token);
      const user = await prisma.user.findUnique({
        where: { id: payload.userId },
        select: { id: true, name: true, avatarUrl: true }
      });

      if (!user) {
        return next(new Error('User not found'));
      }

      socket.data.user = user;
      next();
    } catch (error) {
      next(new Error('Invalid token'));
    }
  });

  io.on('connection', (socket: Socket) => {
    const user = socket.data.user;
    logger.info({ userId: user.id, socketId: socket.id }, 'User connected');

    // Join document room
    socket.on('document:join', async ({ documentId }) => {
      try {
        // ตรวจสอบ permission
        const document = await prisma.document.findUnique({
          where: { id: documentId },
          include: {
            project: {
              include: {
                members: { where: { userId: user.id } }
              }
            }
          }
        });

        if (!document) {
          socket.emit('error', { message: 'Document not found' });
          return;
        }

        const member = document.project.members[0];
        if (!member && document.project.visibility === 'PRIVATE') {
          socket.emit('error', { message: 'Access denied' });
          return;
        }

        // Join Socket.IO room
        socket.join(`doc:${documentId}`);

        // ดึง or สร้าง Yjs doc
        let ydoc = yjsDocs.get(documentId);
        if (!ydoc) {
          ydoc = new Y.Doc();

          // โหลด state จาก Redis
          const savedState = await redis.get(`ydoc:${documentId}`);
          if (savedState) {
            const state = Buffer.from(savedState, 'base64');
            Y.applyUpdate(ydoc, state);
          } else if (document.yState) {
            Y.applyUpdate(ydoc, document.yState);
          } else {
            // Initialize ด้วย content จาก DB
            const ytext = ydoc.getText('content');
            ytext.insert(0, document.content);
          }

          // Listen สำหรับ changes
          ydoc.on('update', async (update: Uint8Array) => {
            // Broadcast ไปยัง clients อื่นในห้อง
            socket.to(`doc:${documentId}`).emit('document:update', {
              update: Buffer.from(update).toString('base64'),
              documentId
            });

            // บันทึกใน Redis (debounced)
            const state = Y.encodeStateAsUpdate(ydoc!);
            await redis.setEx(
              `ydoc:${documentId}`,
              3600, // 1 ชั่วโมง
              Buffer.from(state).toString('base64')
            );
          });

          yjsDocs.set(documentId, ydoc);
        }

        // ส่ง initial state
        const state = Y.encodeStateAsUpdate(ydoc);
        socket.emit('document:state', {
          documentId,
          state: Buffer.from(state).toString('base64'),
          version: document.version
        });

        // บันทึก session
        const color = CURSOR_COLORS[
          parseInt(user.id.slice(-2), 16) % CURSOR_COLORS.length
        ];

        await prisma.collabSession.upsert({
          where: { socketId: socket.id },
          create: {
            documentId,
            userId: user.id,
            socketId: socket.id,
            color
          },
          update: {
            documentId,
            lastActiveAt: new Date()
          }
        });

        // แจ้ง users อื่น
        const sessions = await prisma.collabSession.findMany({
          where: { documentId },
          select: {
            userId: true,
            color: true,
            cursor: true
          }
        });

        socket.to(`doc:${documentId}`).emit('user:joined', {
          userId: user.id,
          name: user.name,
          avatarUrl: user.avatarUrl,
          color
        });

        socket.emit('users:active', {
          documentId,
          users: sessions
        });

      } catch (error) {
        logger.error({ error, userId: user.id }, 'Error joining document');
        socket.emit('error', { message: 'Internal error' });
      }
    });

    // รับ Yjs updates
    socket.on('document:update', async ({ documentId, update }) => {
      try {
        const ydoc = yjsDocs.get(documentId);
        if (!ydoc) return;

        const updateBuffer = Buffer.from(update, 'base64');
        Y.applyUpdate(ydoc, updateBuffer);

        // บันทึกใน DB ทุก 30 วินาที (debounced via Redis)
        const lockKey = `save_lock:${documentId}`;
        const hasLock = await redis.set(lockKey, '1', {
          NX: true,
          EX: 30
        });

        if (hasLock) {
          // Queue การบันทึก
          setTimeout(async () => {
            try {
              const currentDoc = yjsDocs.get(documentId);
              if (!currentDoc) return;

              const content = currentDoc.getText('content').toString();
              const yState = Buffer.from(Y.encodeStateAsUpdate(currentDoc));

              await prisma.document.update({
                where: { id: documentId },
                data: {
                  content: content.length > 1_000_000 ? '' : content,
                  yState,
                  version: { increment: 1 },
                  updatedAt: new Date()
                }
              });

              await redis.del(lockKey);
            } catch (error) {
              logger.error({ error, documentId }, 'Error saving document');
            }
          }, 30000);
        }

      } catch (error) {
        logger.error({ error }, 'Error applying update');
      }
    });

    // Cursor position
    socket.on('cursor:move', async ({ documentId, cursor }) => {
      socket.to(`doc:${documentId}`).emit('cursor:update', {
        userId: user.id,
        cursor,
        timestamp: Date.now()
      });

      // บันทึก cursor ใน Redis (TTL สั้น)
      await redis.setEx(
        `cursor:${documentId}:${user.id}`,
        10,
        JSON.stringify(cursor)
      );
    });

    // Selection
    socket.on('selection:change', ({ documentId, selection }) => {
      socket.to(`doc:${documentId}`).emit('selection:update', {
        userId: user.id,
        selection
      });
    });

    // Chat
    socket.on('chat:send', async ({ projectId, content, type = 'TEXT' }) => {
      try {
        const room = await prisma.chatRoom.upsert({
          where: { projectId },
          create: { projectId },
          update: {}
        });

        const message = await prisma.chatMessage.create({
          data: {
            roomId: room.id,
            userId: user.id,
            content,
            type
          }
        });

        io.to(`project:${projectId}`).emit('chat:message', {
          id: message.id,
          userId: user.id,
          userName: user.name,
          userAvatar: user.avatarUrl,
          content: message.content,
          type: message.type,
          createdAt: message.createdAt
        });
      } catch (error) {
        logger.error({ error }, 'Error sending chat message');
      }
    });

    // Code execution (sandboxed)
    socket.on('code:execute', async ({ documentId, language, code }) => {
      try {
        socket.emit('code:executing', { documentId });

        const result = await executeCode(code, language);

        socket.emit('code:result', {
          documentId,
          output: result.output,
          error: result.error,
          exitCode: result.exitCode,
          duration: result.duration
        });
      } catch (error) {
        socket.emit('code:result', {
          documentId,
          error: 'Execution failed',
          exitCode: 1
        });
      }
    });

    // Disconnect
    socket.on('disconnect', async () => {
      try {
        logger.info({ userId: user.id, socketId: socket.id }, 'User disconnected');

        const session = await prisma.collabSession.findUnique({
          where: { socketId: socket.id }
        });

        if (session) {
          await prisma.collabSession.delete({
            where: { socketId: socket.id }
          });

          io.to(`doc:${session.documentId}`).emit('user:left', {
            userId: user.id
          });
        }
      } catch (error) {
        logger.error({ error }, 'Error on disconnect');
      }
    });
  });
}

// Sandboxed code execution
async function executeCode(code: string, language: string) {
  // ใช้ Docker container แยกสำหรับ execution ที่ปลอดภัย
  const { exec } = require('child_process');
  const { promisify } = require('util');
  const execAsync = promisify(exec);

  const dockerImages = {
    javascript: 'node:20-alpine',
    python: 'python:3.11-alpine',
    typescript: 'node:20-alpine'
  };

  const image = dockerImages[language as keyof typeof dockerImages];
  if (!image) {
    return { error: 'Unsupported language', exitCode: 1, duration: 0 };
  }

  const start = Date.now();

  try {
    const { stdout, stderr } = await execAsync(
      `docker run --rm --memory=64m --cpus=0.1 --network=none --read-only --tmpfs /tmp --timeout=10 ${image} ${language === 'javascript' ? 'node' : 'python'} -e "${code.replace(/"/g, '\\"')}"`,
      { timeout: 15000 }
    );

    return {
      output: stdout,
      error: stderr,
      exitCode: 0,
      duration: Date.now() - start
    };
  } catch (error: any) {
    return {
      output: '',
      error: error.stderr || error.message,
      exitCode: error.code || 1,
      duration: Date.now() - start
    };
  }
}
```

---

## Step 1975: Frontend - Next.js

### CodeMirror Editor Component

```typescript
// components/Editor/index.tsx
'use client';

import React, { useEffect, useRef, useState, useCallback } from 'react';
import { EditorView, basicSetup } from 'codemirror';
import { EditorState, StateField, StateEffect } from '@codemirror/state';
import { keymap } from '@codemirror/view';
import { indentWithTab } from '@codemirror/commands';
import { javascript } from '@codemirror/lang-javascript';
import { python } from '@codemirror/lang-python';
import { rust } from '@codemirror/lang-rust';
import { oneDark } from '@codemirror/theme-one-dark';
import * as Y from 'yjs';
import { yCollab } from 'y-codemirror.next';
import { WebsocketProvider } from 'y-websocket';
import { useSocket } from '@/hooks/useSocket';
import { useUser } from '@/hooks/useUser';

interface EditorProps {
  documentId: string;
  language: string;
  readOnly?: boolean;
  onSave?: (content: string) => void;
}

const languageExtensions: Record<string, () => any> = {
  javascript: javascript,
  typescript: () => javascript({ typescript: true }),
  python: python,
  rust: rust
};

export function Editor({ documentId, language, readOnly, onSave }: EditorProps) {
  const editorRef = useRef<HTMLDivElement>(null);
  const viewRef = useRef<EditorView | null>(null);
  const ydocRef = useRef<Y.Doc | null>(null);
  const providerRef = useRef<WebsocketProvider | null>(null);
  const [isConnected, setIsConnected] = useState(false);
  const [collaborators, setCollaborators] = useState<any[]>([]);
  const { user } = useUser();
  const { socket } = useSocket();

  useEffect(() => {
    if (!editorRef.current || !user) return;

    // สร้าง Yjs document
    const ydoc = new Y.Doc();
    ydocRef.current = ydoc;
    const ytext = ydoc.getText('content');

    // WebSocket provider สำหรับ sync
    const wsUrl = process.env.NEXT_PUBLIC_WS_URL || 'ws://localhost:3001';
    const provider = new WebsocketProvider(
      `${wsUrl}/collab`,
      documentId,
      ydoc,
      { params: { token: user.token } }
    );
    providerRef.current = provider;

    // Awareness (cursors, selections)
    const awareness = provider.awareness;
    awareness.setLocalStateField('user', {
      id: user.id,
      name: user.name,
      color: user.cursorColor || '#007AFF'
    });

    awareness.on('change', () => {
      const states = Array.from(awareness.getStates().values());
      const others = states
        .filter((s: any) => s.user && s.user.id !== user.id)
        .map((s: any) => s.user);
      setCollaborators(others);
    });

    provider.on('status', ({ status }: { status: string }) => {
      setIsConnected(status === 'connected');
    });

    // Language extension
    const langExtension = languageExtensions[language]?.() || javascript();

    // สร้าง EditorView
    const extensions = [
      basicSetup,
      langExtension,
      oneDark,
      keymap.of([indentWithTab]),
      yCollab(ytext, awareness, {
        undoManager: new Y.UndoManager(ytext)
      }),
      EditorView.updateListener.of((update) => {
        if (update.docChanged && onSave) {
          // Debounce save
          clearTimeout(viewRef.current?.dom.dataset.saveTimeout as unknown as number);
          const timeout = setTimeout(() => {
            onSave(update.state.doc.toString());
          }, 1000);
          viewRef.current!.dom.dataset.saveTimeout = timeout as unknown as string;
        }
      }),
      EditorView.editable.of(!readOnly),
      EditorView.theme({
        '&': { height: '100%', fontSize: '14px', fontFamily: "'JetBrains Mono', monospace" },
        '.cm-scroller': { overflow: 'auto' },
        '.cm-content': { padding: '8px 0' }
      })
    ];

    const view = new EditorView({
      state: EditorState.create({ extensions }),
      parent: editorRef.current
    });

    viewRef.current = view;

    return () => {
      view.destroy();
      provider.disconnect();
      ydoc.destroy();
    };
  }, [documentId, language, user]);

  // AI Code Completion
  const triggerAICompletion = useCallback(async () => {
    if (!viewRef.current) return;

    const view = viewRef.current;
    const cursor = view.state.selection.main.head;
    const code = view.state.doc.toString();
    const line = view.state.doc.lineAt(cursor);

    try {
      const response = await fetch(`/api/v1/documents/${documentId}/ai/complete`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          code,
          language,
          cursor: { line: line.number, col: cursor - line.from }
        })
      });

      if (!response.ok || !response.body) return;

      const reader = response.body.getReader();
      const decoder = new TextDecoder();
      let completion = '';

      while (true) {
        const { done, value } = await reader.read();
        if (done) break;

        const text = decoder.decode(value);
        const lines = text.split('\n');

        for (const line of lines) {
          if (line.startsWith('data: ')) {
            const data = line.slice(6);
            if (data === '[DONE]') break;
            try {
              const { text: chunk } = JSON.parse(data);
              completion += chunk;
            } catch {}
          }
        }
      }

      // แทรก completion
      view.dispatch({
        changes: {
          from: cursor,
          to: cursor,
          insert: completion
        }
      });
    } catch (error) {
      console.error('AI completion error:', error);
    }
  }, [documentId, language]);

  return (
    <div className="relative h-full flex flex-col bg-gray-900">
      {/* Toolbar */}
      <div className="flex items-center justify-between px-4 py-2 bg-gray-800 border-b border-gray-700">
        <div className="flex items-center gap-2">
          <span className={`w-2 h-2 rounded-full ${isConnected ? 'bg-green-400' : 'bg-red-400'}`} />
          <span className="text-sm text-gray-400">
            {isConnected ? 'Connected' : 'Disconnected'}
          </span>
        </div>

        {/* Collaborators */}
        <div className="flex items-center gap-1">
          {collaborators.map(collab => (
            <div
              key={collab.id}
              className="w-7 h-7 rounded-full flex items-center justify-center text-xs font-bold text-white"
              style={{ backgroundColor: collab.color }}
              title={collab.name}
            >
              {collab.name[0]}
            </div>
          ))}
        </div>

        {/* AI Button */}
        <button
          onClick={triggerAICompletion}
          className="flex items-center gap-2 px-3 py-1.5 bg-purple-600 hover:bg-purple-700 text-white text-sm rounded-md transition-colors"
        >
          ✨ AI Complete
        </button>
      </div>

      {/* Editor */}
      <div ref={editorRef} className="flex-1 overflow-hidden" />
    </div>
  );
}
```

---

## Step 1976: Authentication

### Next.js Auth ด้วย JWT + OAuth

```typescript
// app/api/auth/[...nextauth]/route.ts
import NextAuth from 'next-auth';
import GithubProvider from 'next-auth/providers/github';
import CredentialsProvider from 'next-auth/providers/credentials';
import { prisma } from '@/lib/prisma';
import bcrypt from 'bcryptjs';
import { signJWT } from '@/lib/jwt';

export const authOptions = {
  providers: [
    GithubProvider({
      clientId: process.env.GITHUB_CLIENT_ID!,
      clientSecret: process.env.GITHUB_CLIENT_SECRET!,
      authorization: {
        params: {
          scope: 'read:user user:email repo'
        }
      }
    }),
    CredentialsProvider({
      name: 'credentials',
      credentials: {
        email: { label: 'Email', type: 'email' },
        password: { label: 'Password', type: 'password' }
      },
      async authorize(credentials) {
        if (!credentials?.email || !credentials.password) return null;

        const user = await prisma.user.findUnique({
          where: { email: credentials.email }
        });

        if (!user || !user.passwordHash) return null;

        const isValid = await bcrypt.compare(credentials.password, user.passwordHash);
        if (!isValid) return null;

        return {
          id: user.id,
          email: user.email,
          name: user.name,
          image: user.avatarUrl
        };
      }
    })
  ],

  callbacks: {
    async signIn({ user, account, profile }) {
      if (account?.provider === 'github') {
        // สร้างหรืออัปเดต user
        await prisma.user.upsert({
          where: { email: user.email! },
          create: {
            email: user.email!,
            name: user.name!,
            username: (profile as any).login,
            avatarUrl: user.image,
            githubId: String((profile as any).id)
          },
          update: {
            name: user.name!,
            avatarUrl: user.image,
            lastActiveAt: new Date()
          }
        });
      }
      return true;
    },

    async jwt({ token, user, account }) {
      if (user) {
        const dbUser = await prisma.user.findUnique({
          where: { email: user.email! },
          select: { id: true, role: true, subscriptionStatus: true }
        });

        token.userId = dbUser?.id;
        token.role = dbUser?.role;
        token.subscription = dbUser?.subscriptionStatus;
      }
      return token;
    },

    async session({ session, token }) {
      session.user.id = token.userId as string;
      session.user.role = token.role as string;
      session.user.subscription = token.subscription as string;
      return session;
    }
  },

  pages: {
    signIn: '/auth/login',
    error: '/auth/error'
  },

  session: {
    strategy: 'jwt',
    maxAge: 30 * 24 * 60 * 60 // 30 วัน
  }
};

const handler = NextAuth(authOptions);
export { handler as GET, handler as POST };
```

---

## Step 1977: Stripe Billing

### การจัดการ Subscription ด้วย Stripe

```typescript
// app/api/billing/route.ts
import Stripe from 'stripe';
import { prisma } from '@/lib/prisma';
import { auth } from '@/lib/auth';

const stripe = new Stripe(process.env.STRIPE_SECRET_KEY!, {
  apiVersion: '2023-10-16'
});

// Plans
const PLANS = {
  FREE: { price: 0, features: { aiRequests: 50, storage: '1GB', team: 1 } },
  PRO: {
    priceId: process.env.STRIPE_PRO_PRICE_ID!,
    price: 12,
    features: { aiRequests: 1000, storage: '10GB', team: 5 }
  },
  TEAM: {
    priceId: process.env.STRIPE_TEAM_PRICE_ID!,
    price: 49,
    features: { aiRequests: 5000, storage: '100GB', team: 25 }
  }
};

// POST /api/billing/subscribe
export async function POST(req: Request) {
  const session = await auth();
  if (!session) return Response.json({ error: 'Unauthorized' }, { status: 401 });

  const { plan } = await req.json();
  const priceId = PLANS[plan as keyof typeof PLANS]?.priceId;

  if (!priceId) {
    return Response.json({ error: 'Invalid plan' }, { status: 400 });
  }

  const user = await prisma.user.findUnique({
    where: { id: session.user.id }
  });

  if (!user) return Response.json({ error: 'User not found' }, { status: 404 });

  // สร้าง/ดึง Stripe customer
  let customerId = user.stripeCustomerId;
  if (!customerId) {
    const customer = await stripe.customers.create({
      email: user.email,
      name: user.name,
      metadata: { userId: user.id }
    });
    customerId = customer.id;

    await prisma.user.update({
      where: { id: user.id },
      data: { stripeCustomerId: customerId }
    });
  }

  // สร้าง checkout session
  const checkoutSession = await stripe.checkout.sessions.create({
    customer: customerId,
    payment_method_types: ['card'],
    mode: 'subscription',
    line_items: [{ price: priceId, quantity: 1 }],
    success_url: `${process.env.NEXT_PUBLIC_URL}/billing/success?session_id={CHECKOUT_SESSION_ID}`,
    cancel_url: `${process.env.NEXT_PUBLIC_URL}/billing`,
    subscription_data: {
      metadata: { userId: user.id, plan }
    },
    allow_promotion_codes: true,
    billing_address_collection: 'auto'
  });

  return Response.json({ url: checkoutSession.url });
}

// Stripe Webhook Handler
export async function handleStripeWebhook(req: Request) {
  const sig = req.headers.get('stripe-signature')!;
  const body = await req.text();

  let event: Stripe.Event;
  try {
    event = stripe.webhooks.constructEvent(
      body,
      sig,
      process.env.STRIPE_WEBHOOK_SECRET!
    );
  } catch {
    return Response.json({ error: 'Invalid signature' }, { status: 400 });
  }

  switch (event.type) {
    case 'checkout.session.completed': {
      const session = event.data.object as Stripe.Checkout.Session;
      await handleCheckoutComplete(session);
      break;
    }

    case 'customer.subscription.updated': {
      const subscription = event.data.object as Stripe.Subscription;
      await handleSubscriptionUpdate(subscription);
      break;
    }

    case 'customer.subscription.deleted': {
      const subscription = event.data.object as Stripe.Subscription;
      await handleSubscriptionCancel(subscription);
      break;
    }

    case 'invoice.payment_failed': {
      const invoice = event.data.object as Stripe.Invoice;
      await handlePaymentFailed(invoice);
      break;
    }
  }

  return Response.json({ received: true });
}

async function handleCheckoutComplete(session: Stripe.Checkout.Session) {
  const subscription = await stripe.subscriptions.retrieve(
    session.subscription as string
  );

  const userId = subscription.metadata.userId;
  const plan = subscription.metadata.plan;

  await prisma.user.update({
    where: { id: userId },
    data: {
      stripeSubscriptionId: subscription.id,
      subscriptionStatus: plan as any
    }
  });
}

async function handleSubscriptionUpdate(subscription: Stripe.Subscription) {
  const userId = subscription.metadata.userId;

  const statusMap: Record<string, string> = {
    active: subscription.metadata.plan || 'PRO',
    past_due: 'PAST_DUE',
    canceled: 'FREE',
    trialing: subscription.metadata.plan || 'PRO'
  };

  await prisma.user.update({
    where: { stripeSubscriptionId: subscription.id },
    data: {
      subscriptionStatus: (statusMap[subscription.status] || 'FREE') as any
    }
  });
}

async function handleSubscriptionCancel(subscription: Stripe.Subscription) {
  await prisma.user.update({
    where: { stripeSubscriptionId: subscription.id },
    data: {
      subscriptionStatus: 'FREE',
      stripeSubscriptionId: null
    }
  });
}

async function handlePaymentFailed(invoice: Stripe.Invoice) {
  // ส่ง email แจ้งเตือน
  const customer = await stripe.customers.retrieve(invoice.customer as string);
  if ('email' in customer && customer.email) {
    await sendPaymentFailedEmail(customer.email);
  }
}
```

---

## Step 1978: Deployment

### Docker Compose สำหรับ Production

```yaml
# docker-compose.production.yml
version: '3.9'

services:
  # Next.js Frontend
  frontend:
    image: ${ECR_URL}/devcollab-frontend:${VERSION}
    environment:
      NEXT_PUBLIC_API_URL: https://api.devcollab.io
      NEXT_PUBLIC_WS_URL: wss://api.devcollab.io
      NEXTAUTH_URL: https://devcollab.io
      NEXTAUTH_SECRET: ${NEXTAUTH_SECRET}
    restart: unless-stopped
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.frontend.rule=Host(`devcollab.io`)"
      - "traefik.http.routers.frontend.tls.certresolver=letsencrypt"

  # API Server
  api:
    image: ${ECR_URL}/devcollab-api:${VERSION}
    environment:
      NODE_ENV: production
      DATABASE_URL: ${DATABASE_URL}
      REDIS_URL: ${REDIS_URL}
      JWT_SECRET: ${JWT_SECRET}
      ANTHROPIC_API_KEY: ${ANTHROPIC_API_KEY}
      STRIPE_SECRET_KEY: ${STRIPE_SECRET_KEY}
    restart: unless-stopped
    deploy:
      replicas: 3
      resources:
        limits:
          cpus: '1'
          memory: 1G
    healthcheck:
      test: ["CMD", "wget", "-q", "--spider", "http://localhost:3000/health"]
      interval: 30s
      timeout: 10s
      retries: 3
    labels:
      - "traefik.enable=true"
      - "traefik.http.routers.api.rule=Host(`api.devcollab.io`)"
      - "traefik.http.routers.api.tls.certresolver=letsencrypt"

  # Reverse Proxy
  traefik:
    image: traefik:v3.0
    command:
      - "--providers.docker=true"
      - "--providers.docker.swarmmode=true"
      - "--entrypoints.web.address=:80"
      - "--entrypoints.websecure.address=:443"
      - "--certificatesresolvers.letsencrypt.acme.email=admin@devcollab.io"
      - "--certificatesresolvers.letsencrypt.acme.storage=/letsencrypt/acme.json"
      - "--certificatesresolvers.letsencrypt.acme.httpchallenge.entrypoint=web"
      - "--api.dashboard=true"
    ports:
      - "80:80"
      - "443:443"
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:ro
      - letsencrypt:/letsencrypt
    restart: unless-stopped

volumes:
  letsencrypt:
```

### Kubernetes Deployment (สำหรับ scale ใหญ่)

```yaml
# k8s/api-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: devcollab-api
  namespace: production
spec:
  replicas: 3
  selector:
    matchLabels:
      app: devcollab-api
  template:
    metadata:
      labels:
        app: devcollab-api
    spec:
      containers:
        - name: api
          image: ${ECR_URL}/devcollab-api:${VERSION}
          ports:
            - containerPort: 3000
          env:
            - name: NODE_ENV
              value: "production"
            - name: DATABASE_URL
              valueFrom:
                secretKeyRef:
                  name: devcollab-secrets
                  key: database-url
          resources:
            requests:
              memory: "256Mi"
              cpu: "250m"
            limits:
              memory: "1Gi"
              cpu: "1000m"
          livenessProbe:
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 30
            periodSeconds: 10
          readinessProbe:
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 5
            periodSeconds: 5
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchExpressions:
                  - key: app
                    operator: In
                    values:
                      - devcollab-api
              topologyKey: kubernetes.io/hostname

---
apiVersion: v1
kind: Service
metadata:
  name: devcollab-api
  namespace: production
spec:
  selector:
    app: devcollab-api
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
  type: ClusterIP

---
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: devcollab-api-hpa
  namespace: production
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: devcollab-api
  minReplicas: 3
  maxReplicas: 20
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 80
```

---

## Step 1979-2000: Final Steps และ Launch Checklist

### Pre-Launch Checklist

```typescript
// scripts/prelaunch-check.ts
const checks = [
  // Security
  { name: 'SSL Certificate', check: checkSSL },
  { name: 'Security Headers', check: checkSecurityHeaders },
  { name: 'Rate Limiting', check: checkRateLimiting },
  { name: 'Input Validation', check: checkInputValidation },
  { name: 'Auth Security', check: checkAuthSecurity },
  
  // Performance
  { name: 'Database Indexes', check: checkDatabaseIndexes },
  { name: 'Query Performance', check: checkQueryPerformance },
  { name: 'Cache Hit Rate', check: checkCacheHitRate },
  { name: 'API Response Times', check: checkAPIResponseTimes },
  { name: 'Bundle Size', check: checkBundleSize },
  
  // Reliability
  { name: 'Health Checks', check: checkHealthEndpoints },
  { name: 'Error Monitoring', check: checkSentrySetup },
  { name: 'Logging', check: checkLoggingSetup },
  { name: 'Backups', check: checkDatabaseBackups },
  { name: 'Graceful Shutdown', check: checkGracefulShutdown },
  
  // Legal
  { name: 'Privacy Policy', check: checkPrivacyPolicy },
  { name: 'Terms of Service', check: checkTermsOfService },
  { name: 'GDPR Compliance', check: checkGDPRCompliance },
  { name: 'Cookie Consent', check: checkCookieConsent }
];

async function runChecks() {
  console.log('🚀 Running pre-launch checks...\n');
  let passed = 0;
  let failed = 0;

  for (const check of checks) {
    try {
      await check.check();
      console.log(`✅ ${check.name}`);
      passed++;
    } catch (error) {
      console.log(`❌ ${check.name}: ${error.message}`);
      failed++;
    }
  }

  console.log(`\n📊 Results: ${passed} passed, ${failed} failed`);

  if (failed > 0) {
    process.exit(1);
  }
}

runChecks();
```

### Performance Optimization

```typescript
// Next.js Configuration
// next.config.ts
import type { NextConfig } from 'next';

const nextConfig: NextConfig = {
  // Compression
  compress: true,
  
  // Image optimization
  images: {
    domains: ['avatars.githubusercontent.com', 's3.amazonaws.com'],
    formats: ['image/webp', 'image/avif'],
    minimumCacheTTL: 60 * 60 * 24 * 30 // 30 วัน
  },
  
  // Bundle analysis
  bundleAnalyzer: process.env.ANALYZE === 'true',
  
  // Headers
  async headers() {
    return [
      {
        source: '/(.*)',
        headers: [
          { key: 'X-Frame-Options', value: 'DENY' },
          { key: 'X-Content-Type-Options', value: 'nosniff' },
          { key: 'Referrer-Policy', value: 'strict-origin-when-cross-origin' },
          { key: 'Permissions-Policy', value: 'camera=(), microphone=()' }
        ]
      },
      {
        source: '/api/(.*)',
        headers: [
          { key: 'Cache-Control', value: 'no-store' }
        ]
      }
    ];
  },
  
  // Redirects
  async redirects() {
    return [
      {
        source: '/home',
        destination: '/',
        permanent: true
      }
    ];
  },
  
  // Experimental features
  experimental: {
    serverActions: { allowedOrigins: ['devcollab.io'] },
    turbo: {}
  }
};

export default nextConfig;
```

### Monitoring Dashboard

```typescript
// src/routes/metrics.ts
import { Router } from 'express';
import { prisma } from '../lib/prisma';
import { redis } from '../lib/redis';
import os from 'os';

const router = Router();

// Prometheus metrics
router.get('/metrics', async (req, res) => {
  const metrics = await collectMetrics();

  // Prometheus format
  const output = Object.entries(metrics)
    .map(([key, value]) => `devcollab_${key} ${value}`)
    .join('\n');

  res.setHeader('Content-Type', 'text/plain');
  res.send(output);
});

async function collectMetrics() {
  const now = new Date();
  const hourAgo = new Date(now.getTime() - 60 * 60 * 1000);

  const [
    totalUsers,
    activeUsers,
    totalDocuments,
    activeSessions,
    aiRequestsHour
  ] = await Promise.all([
    prisma.user.count(),
    prisma.user.count({ where: { lastActiveAt: { gte: hourAgo } } }),
    prisma.document.count(),
    prisma.collabSession.count(),
    prisma.aIUsage.count({ where: { createdAt: { gte: hourAgo } } })
  ]);

  return {
    users_total: totalUsers,
    users_active_1h: activeUsers,
    documents_total: totalDocuments,
    collab_sessions_active: activeSessions,
    ai_requests_1h: aiRequestsHour,
    memory_used_bytes: process.memoryUsage().heapUsed,
    cpu_usage_percent: os.loadavg()[0] / os.cpus().length * 100,
    uptime_seconds: process.uptime()
  };
}

export { router };
```

---

## สรุปโปรเจค DevCollab

### สิ่งที่เราสร้าง

DevCollab เป็น SaaS application ระดับ production ที่รวบรวมความรู้ทั้งหมดจากคอร์สนี้:

1. **Real-time Collaboration** - Yjs CRDT + WebSocket
2. **Code Editor** - CodeMirror 6 พร้อม syntax highlighting
3. **AI Integration** - Claude API สำหรับ code completion
4. **Authentication** - NextAuth.js + GitHub OAuth + JWT
5. **Billing** - Stripe subscription management
6. **Database** - PostgreSQL + Prisma ORM
7. **Caching** - Redis สำหรับ performance
8. **File Storage** - AWS S3
9. **Email** - SendGrid integration
10. **Monitoring** - Sentry + DataDog + Prometheus
11. **CI/CD** - GitHub Actions
12. **Containerization** - Docker + Kubernetes
13. **Infrastructure** - AWS ECS/Fargate
14. **Security** - Helmet, Rate limiting, Input validation

### เส้นทางต่อไป

หลังจากจบคอร์สนี้ คุณสามารถเรียนรู้เพิ่มเติมได้ใน:

- **WebAssembly (WASM)** - สำหรับ performance-critical code
- **Edge Computing** - Cloudflare Workers, Vercel Edge
- **AI/ML** - TensorFlow.js, ONNX runtime
- **Blockchain** - Web3, Ethereum, Solidity
- **Systems Programming** - Rust + WebAssembly
- **Computer Graphics** - WebGPU, GLSL shaders
- **Distributed Systems** - Event sourcing, CQRS
- **Platform Engineering** - Backstage, Internal developer platforms

### ขอแสดงความยินดี!

คุณได้เรียนรู้ JavaScript ตั้งแต่พื้นฐานไปจนถึงระดับ world-class production application ความรู้ที่ได้จากคอร์สนี้จะเป็นรากฐานที่แข็งแกร่งสำหรับการพัฒนา software ในอนาคต

**จำไว้เสมอว่า**: การเขียนโค้ดที่ดีไม่ใช่แค่การทำให้มันทำงานได้ แต่คือการทำให้มัน:
- **อ่านง่าย** สำหรับทีม
- **ดูแลรักษาได้** ในระยะยาว
- **ทดสอบได้** เสมอ
- **ปลอดภัย** จากการโจมตี
- **มีประสิทธิภาพ** สำหรับผู้ใช้
- **Reliable** สำหรับ production

Happy Coding! 🚀
