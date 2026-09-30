# Part 80: โปรเจค: Full-Stack Application - Task Management System (Steps 1571-1590)

## บทนำ: Task Management System

ในบทนี้เราจะสร้าง **Task Management System** แบบ Full-Stack ที่สมบูรณ์ครอบคลุม:

- **Backend**: Node.js + Express + MongoDB + Socket.io
- **Frontend**: React + TypeScript + Vite + TailwindCSS
- **DevOps**: Docker + Docker Compose + GitHub Actions CI/CD

### ฟีเจอร์ที่จะสร้าง

1. User Authentication (JWT)
2. Projects CRUD
3. Tasks CRUD พร้อม Kanban Board
4. Real-time Updates (WebSocket)
5. File Upload สำหรับ attachments
6. Responsive Design

---

## Step 1571: โครงสร้างโปรเจค

```
task-manager/
├── backend/
│   ├── src/
│   │   ├── config/
│   │   │   ├── database.js
│   │   │   └── redis.js
│   │   ├── middleware/
│   │   │   ├── auth.js
│   │   │   ├── validate.js
│   │   │   └── upload.js
│   │   ├── models/
│   │   │   ├── User.js
│   │   │   ├── Project.js
│   │   │   └── Task.js
│   │   ├── routes/
│   │   │   ├── auth.routes.js
│   │   │   ├── project.routes.js
│   │   │   └── task.routes.js
│   │   ├── controllers/
│   │   │   ├── auth.controller.js
│   │   │   ├── project.controller.js
│   │   │   └── task.controller.js
│   │   ├── services/
│   │   │   ├── auth.service.js
│   │   │   └── email.service.js
│   │   ├── socket/
│   │   │   └── socketHandler.js
│   │   ├── utils/
│   │   │   ├── ApiError.js
│   │   │   └── catchAsync.js
│   │   └── app.js
│   ├── tests/
│   ├── Dockerfile
│   └── package.json
│
├── frontend/
│   ├── src/
│   │   ├── api/
│   │   ├── components/
│   │   ├── hooks/
│   │   ├── pages/
│   │   ├── store/
│   │   ├── types/
│   │   └── App.tsx
│   ├── Dockerfile
│   └── package.json
│
├── docker-compose.yml
└── .github/
    └── workflows/
        └── ci.yml
```

---

## Step 1572: Backend Setup

### package.json

```json
{
  "name": "task-manager-backend",
  "version": "1.0.0",
  "scripts": {
    "start": "node dist/app.js",
    "dev": "nodemon --exec ts-node src/app.ts",
    "build": "tsc",
    "test": "jest",
    "test:coverage": "jest --coverage"
  },
  "dependencies": {
    "bcryptjs": "^2.4.3",
    "compression": "^1.7.4",
    "cors": "^2.8.5",
    "dotenv": "^16.3.1",
    "express": "^4.18.2",
    "express-rate-limit": "^7.1.5",
    "express-validator": "^7.0.1",
    "helmet": "^7.1.0",
    "ioredis": "^5.3.2",
    "jsonwebtoken": "^9.0.2",
    "mongoose": "^8.0.3",
    "multer": "^1.4.5-lts.1",
    "nodemailer": "^6.9.7",
    "sharp": "^0.33.0",
    "socket.io": "^4.6.2",
    "winston": "^3.11.0"
  },
  "devDependencies": {
    "@types/bcryptjs": "^2.4.6",
    "@types/cors": "^2.8.17",
    "@types/express": "^4.17.21",
    "@types/jsonwebtoken": "^9.0.5",
    "@types/multer": "^1.4.11",
    "@types/node": "^20.10.6",
    "@types/nodemailer": "^6.4.14",
    "jest": "^29.7.0",
    "nodemon": "^3.0.2",
    "ts-jest": "^29.1.1",
    "typescript": "^5.3.3"
  }
}
```

### app.ts

```typescript
// backend/src/app.ts
import express from 'express'
import cors from 'cors'
import helmet from 'helmet'
import compression from 'compression'
import rateLimit from 'express-rate-limit'
import { createServer } from 'http'
import { Server as SocketIOServer } from 'socket.io'
import mongoose from 'mongoose'
import path from 'path'

import authRoutes from './routes/auth.routes'
import projectRoutes from './routes/project.routes'
import taskRoutes from './routes/task.routes'
import uploadRoutes from './routes/upload.routes'
import { setupSocketHandlers } from './socket/socketHandler'
import { errorHandler } from './middleware/errorHandler'
import logger from './utils/logger'

const app = express()
const httpServer = createServer(app)

// Socket.io setup
export const io = new SocketIOServer(httpServer, {
  cors: {
    origin: process.env.FRONTEND_URL || 'http://localhost:3000',
    methods: ['GET', 'POST'],
  },
})

// Middleware
app.use(helmet())
app.use(compression())
app.use(cors({
  origin: process.env.FRONTEND_URL || 'http://localhost:3000',
  credentials: true,
}))
app.use(express.json({ limit: '10mb' }))
app.use(express.urlencoded({ extended: true, limit: '10mb' }))

// Rate limiting
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000,
  max: 100,
  message: { error: 'Too many requests, please try again later' },
})
app.use('/api', limiter)

// Static files (uploads)
app.use('/uploads', express.static(path.join(__dirname, '../uploads')))

// Routes
app.use('/api/auth', authRoutes)
app.use('/api/projects', projectRoutes)
app.use('/api/tasks', taskRoutes)
app.use('/api/upload', uploadRoutes)

// Health check
app.get('/health', (req, res) => {
  res.json({
    status: 'healthy',
    timestamp: new Date().toISOString(),
    uptime: process.uptime(),
  })
})

// 404 handler
app.use('*', (req, res) => {
  res.status(404).json({ error: 'Route not found' })
})

// Error handler
app.use(errorHandler)

// Socket handlers
setupSocketHandlers(io)

// Connect to MongoDB
mongoose.connect(process.env.MONGODB_URL as string)
  .then(() => logger.info('Connected to MongoDB'))
  .catch(err => {
    logger.error('MongoDB connection error:', err)
    process.exit(1)
  })

const PORT = process.env.PORT || 3001
httpServer.listen(PORT, () => {
  logger.info(`Server running on port ${PORT}`)
})

export default app
```

---

## Step 1573: Models

### User Model

```typescript
// backend/src/models/User.ts
import mongoose, { Document, Schema } from 'mongoose'
import bcrypt from 'bcryptjs'

export interface IUser extends Document {
  _id: mongoose.Types.ObjectId
  name: string
  email: string
  password: string
  avatar?: string
  role: 'user' | 'admin'
  isActive: boolean
  refreshToken?: string
  createdAt: Date
  updatedAt: Date
  comparePassword(candidatePassword: string): Promise<boolean>
}

const userSchema = new Schema<IUser>(
  {
    name: {
      type: String,
      required: [true, 'Name is required'],
      trim: true,
      minlength: [2, 'Name must be at least 2 characters'],
      maxlength: [50, 'Name must not exceed 50 characters'],
    },
    email: {
      type: String,
      required: [true, 'Email is required'],
      unique: true,
      lowercase: true,
      trim: true,
      match: [/^\S+@\S+\.\S+$/, 'Please enter a valid email'],
    },
    password: {
      type: String,
      required: [true, 'Password is required'],
      minlength: [8, 'Password must be at least 8 characters'],
      select: false,  // ไม่ return password ใน queries
    },
    avatar: String,
    role: {
      type: String,
      enum: ['user', 'admin'],
      default: 'user',
    },
    isActive: {
      type: Boolean,
      default: true,
    },
    refreshToken: {
      type: String,
      select: false,
    },
  },
  {
    timestamps: true,
    toJSON: {
      transform(doc, ret) {
        delete ret.password
        delete ret.refreshToken
        return ret
      },
    },
  }
)

// Hash password ก่อน save
userSchema.pre('save', async function(next) {
  if (!this.isModified('password')) return next()
  this.password = await bcrypt.hash(this.password, 12)
  next()
})

// Method: เปรียบเทียบ password
userSchema.methods.comparePassword = async function(candidatePassword: string) {
  return bcrypt.compare(candidatePassword, this.password)
}

export const User = mongoose.model<IUser>('User', userSchema)
```

### Project Model

```typescript
// backend/src/models/Project.ts
import mongoose, { Document, Schema } from 'mongoose'

export interface IProject extends Document {
  _id: mongoose.Types.ObjectId
  title: string
  description?: string
  owner: mongoose.Types.ObjectId
  members: Array<{
    user: mongoose.Types.ObjectId
    role: 'admin' | 'editor' | 'viewer'
    joinedAt: Date
  }>
  status: 'active' | 'completed' | 'archived'
  color: string
  deadline?: Date
  createdAt: Date
  updatedAt: Date
}

const projectSchema = new Schema<IProject>(
  {
    title: {
      type: String,
      required: [true, 'Project title is required'],
      trim: true,
      minlength: 3,
      maxlength: 100,
    },
    description: {
      type: String,
      maxlength: 500,
    },
    owner: {
      type: Schema.Types.ObjectId,
      ref: 'User',
      required: true,
    },
    members: [
      {
        user: {
          type: Schema.Types.ObjectId,
          ref: 'User',
          required: true,
        },
        role: {
          type: String,
          enum: ['admin', 'editor', 'viewer'],
          default: 'editor',
        },
        joinedAt: {
          type: Date,
          default: Date.now,
        },
      },
    ],
    status: {
      type: String,
      enum: ['active', 'completed', 'archived'],
      default: 'active',
    },
    color: {
      type: String,
      default: '#3B82F6',
    },
    deadline: Date,
  },
  { timestamps: true }
)

// Index
projectSchema.index({ owner: 1, status: 1 })
projectSchema.index({ 'members.user': 1 })

export const Project = mongoose.model<IProject>('Project', projectSchema)
```

### Task Model

```typescript
// backend/src/models/Task.ts
import mongoose, { Document, Schema } from 'mongoose'

export type TaskStatus = 'todo' | 'in_progress' | 'review' | 'done'
export type TaskPriority = 'low' | 'medium' | 'high' | 'urgent'

export interface ITask extends Document {
  _id: mongoose.Types.ObjectId
  title: string
  description?: string
  project: mongoose.Types.ObjectId
  status: TaskStatus
  priority: TaskPriority
  assignee?: mongoose.Types.ObjectId
  reporter: mongoose.Types.ObjectId
  dueDate?: Date
  tags: string[]
  attachments: Array<{
    filename: string
    url: string
    size: number
    mimetype: string
    uploadedBy: mongoose.Types.ObjectId
    uploadedAt: Date
  }>
  comments: Array<{
    _id: mongoose.Types.ObjectId
    author: mongoose.Types.ObjectId
    content: string
    createdAt: Date
  }>
  order: number
  createdAt: Date
  updatedAt: Date
}

const taskSchema = new Schema<ITask>(
  {
    title: {
      type: String,
      required: [true, 'Task title is required'],
      trim: true,
      minlength: 2,
      maxlength: 200,
    },
    description: {
      type: String,
      maxlength: 2000,
    },
    project: {
      type: Schema.Types.ObjectId,
      ref: 'Project',
      required: true,
    },
    status: {
      type: String,
      enum: ['todo', 'in_progress', 'review', 'done'],
      default: 'todo',
    },
    priority: {
      type: String,
      enum: ['low', 'medium', 'high', 'urgent'],
      default: 'medium',
    },
    assignee: {
      type: Schema.Types.ObjectId,
      ref: 'User',
    },
    reporter: {
      type: Schema.Types.ObjectId,
      ref: 'User',
      required: true,
    },
    dueDate: Date,
    tags: [{ type: String, trim: true }],
    attachments: [
      {
        filename: String,
        url: String,
        size: Number,
        mimetype: String,
        uploadedBy: { type: Schema.Types.ObjectId, ref: 'User' },
        uploadedAt: { type: Date, default: Date.now },
      },
    ],
    comments: [
      {
        author: { type: Schema.Types.ObjectId, ref: 'User', required: true },
        content: { type: String, required: true, maxlength: 1000 },
        createdAt: { type: Date, default: Date.now },
      },
    ],
    order: {
      type: Number,
      default: 0,
    },
  },
  { timestamps: true }
)

taskSchema.index({ project: 1, status: 1, order: 1 })
taskSchema.index({ assignee: 1, status: 1 })

export const Task = mongoose.model<ITask>('Task', taskSchema)
```

---

## Step 1574: Authentication

```typescript
// backend/src/controllers/auth.controller.ts
import { Request, Response } from 'express'
import jwt from 'jsonwebtoken'
import { User } from '../models/User'
import { catchAsync } from '../utils/catchAsync'
import { ApiError } from '../utils/ApiError'
import redisClient from '../config/redis'

const JWT_SECRET = process.env.JWT_SECRET as string
const JWT_EXPIRES_IN = process.env.JWT_EXPIRES_IN || '15m'
const REFRESH_TOKEN_EXPIRES_IN = '7d'

function generateTokens(userId: string) {
  const accessToken = jwt.sign(
    { userId },
    JWT_SECRET,
    { expiresIn: JWT_EXPIRES_IN }
  )
  
  const refreshToken = jwt.sign(
    { userId },
    process.env.REFRESH_TOKEN_SECRET as string,
    { expiresIn: REFRESH_TOKEN_EXPIRES_IN }
  )
  
  return { accessToken, refreshToken }
}

export const register = catchAsync(async (req: Request, res: Response) => {
  const { name, email, password } = req.body
  
  // Check if user exists
  const existingUser = await User.findOne({ email })
  if (existingUser) {
    throw new ApiError('Email already in use', 409)
  }
  
  // Create user
  const user = await User.create({ name, email, password })
  
  // Generate tokens
  const { accessToken, refreshToken } = generateTokens(user._id.toString())
  
  // Store refresh token in Redis (7 days)
  await redisClient.setex(
    `refresh:${user._id}`,
    7 * 24 * 60 * 60,
    refreshToken
  )
  
  res.status(201).json({
    success: true,
    user: {
      id: user._id,
      name: user.name,
      email: user.email,
      avatar: user.avatar,
      role: user.role,
    },
    tokens: { accessToken, refreshToken },
  })
})

export const login = catchAsync(async (req: Request, res: Response) => {
  const { email, password } = req.body
  
  // Find user with password
  const user = await User.findOne({ email }).select('+password')
  if (!user) {
    throw new ApiError('Invalid email or password', 401)
  }
  
  // Check password
  const isPasswordValid = await user.comparePassword(password)
  if (!isPasswordValid) {
    throw new ApiError('Invalid email or password', 401)
  }
  
  // Check if account is active
  if (!user.isActive) {
    throw new ApiError('Account is deactivated', 403)
  }
  
  // Generate tokens
  const { accessToken, refreshToken } = generateTokens(user._id.toString())
  
  // Store refresh token
  await redisClient.setex(
    `refresh:${user._id}`,
    7 * 24 * 60 * 60,
    refreshToken
  )
  
  res.json({
    success: true,
    user: {
      id: user._id,
      name: user.name,
      email: user.email,
      avatar: user.avatar,
      role: user.role,
    },
    tokens: { accessToken, refreshToken },
  })
})

export const refreshToken = catchAsync(async (req: Request, res: Response) => {
  const { refreshToken } = req.body
  
  if (!refreshToken) {
    throw new ApiError('Refresh token required', 401)
  }
  
  // Verify refresh token
  const decoded = jwt.verify(
    refreshToken,
    process.env.REFRESH_TOKEN_SECRET as string
  ) as { userId: string }
  
  // Check if refresh token is valid in Redis
  const storedToken = await redisClient.get(`refresh:${decoded.userId}`)
  if (!storedToken || storedToken !== refreshToken) {
    throw new ApiError('Invalid refresh token', 401)
  }
  
  // Generate new tokens
  const tokens = generateTokens(decoded.userId)
  
  // Update refresh token in Redis
  await redisClient.setex(
    `refresh:${decoded.userId}`,
    7 * 24 * 60 * 60,
    tokens.refreshToken
  )
  
  res.json({ success: true, tokens })
})

export const logout = catchAsync(async (req: Request, res: Response) => {
  const userId = req.user!._id.toString()
  
  // Remove refresh token from Redis
  await redisClient.del(`refresh:${userId}`)
  
  res.json({ success: true, message: 'Logged out successfully' })
})

export const getMe = catchAsync(async (req: Request, res: Response) => {
  res.json({ success: true, user: req.user })
})
```

---

## Step 1575: Middleware

```typescript
// backend/src/middleware/auth.ts
import { Request, Response, NextFunction } from 'express'
import jwt from 'jsonwebtoken'
import { User, IUser } from '../models/User'
import { ApiError } from '../utils/ApiError'
import { catchAsync } from '../utils/catchAsync'
import redisClient from '../config/redis'

declare global {
  namespace Express {
    interface Request {
      user?: IUser
    }
  }
}

export const protect = catchAsync(
  async (req: Request, res: Response, next: NextFunction) => {
    // Get token
    let token: string | undefined
    
    if (req.headers.authorization?.startsWith('Bearer ')) {
      token = req.headers.authorization.split(' ')[1]
    }
    
    if (!token) {
      throw new ApiError('Authentication required', 401)
    }
    
    // Verify token
    const decoded = jwt.verify(token, process.env.JWT_SECRET as string) as {
      userId: string
    }
    
    // Check if token is blacklisted (logout)
    const isBlacklisted = await redisClient.get(`blacklist:${token}`)
    if (isBlacklisted) {
      throw new ApiError('Token is no longer valid', 401)
    }
    
    // Get user
    const user = await User.findById(decoded.userId)
    if (!user || !user.isActive) {
      throw new ApiError('User not found', 401)
    }
    
    req.user = user
    next()
  }
)

export const restrictTo = (...roles: string[]) => {
  return (req: Request, res: Response, next: NextFunction) => {
    if (!req.user || !roles.includes(req.user.role)) {
      throw new ApiError('You do not have permission to perform this action', 403)
    }
    next()
  }
}
```

```typescript
// backend/src/middleware/validate.ts
import { body, validationResult } from 'express-validator'
import { Request, Response, NextFunction } from 'express'
import { ApiError } from '../utils/ApiError'

export function validate(validations: any[]) {
  return async (req: Request, res: Response, next: NextFunction) => {
    await Promise.all(validations.map((v: any) => v.run(req)))
    
    const errors = validationResult(req)
    if (!errors.isEmpty()) {
      const messages = errors.array().map(e => e.msg).join(', ')
      throw new ApiError(messages, 400)
    }
    
    next()
  }
}

// Validation rules
export const registerValidation = [
  body('name')
    .trim()
    .notEmpty().withMessage('Name is required')
    .isLength({ min: 2, max: 50 }).withMessage('Name must be 2-50 characters'),
  
  body('email')
    .trim()
    .isEmail().withMessage('Invalid email address')
    .normalizeEmail(),
  
  body('password')
    .isLength({ min: 8 }).withMessage('Password must be at least 8 characters')
    .matches(/\d/).withMessage('Password must contain a number')
    .matches(/[a-z]/).withMessage('Password must contain a lowercase letter')
    .matches(/[A-Z]/).withMessage('Password must contain an uppercase letter'),
]

export const createProjectValidation = [
  body('title')
    .trim()
    .notEmpty().withMessage('Title is required')
    .isLength({ min: 3, max: 100 }).withMessage('Title must be 3-100 characters'),
  
  body('description')
    .optional()
    .isLength({ max: 500 }).withMessage('Description max 500 characters'),
]
```

```typescript
// backend/src/middleware/upload.ts
import multer from 'multer'
import path from 'path'
import { v4 as uuidv4 } from 'uuid'
import { ApiError } from '../utils/ApiError'

const storage = multer.diskStorage({
  destination: (req, file, cb) => {
    cb(null, path.join(__dirname, '../../uploads'))
  },
  filename: (req, file, cb) => {
    const uniqueName = `${uuidv4()}${path.extname(file.originalname)}`
    cb(null, uniqueName)
  },
})

const fileFilter = (req: any, file: Express.Multer.File, cb: multer.FileFilterCallback) => {
  const allowedTypes = [
    'image/jpeg', 'image/png', 'image/gif', 'image/webp',
    'application/pdf',
    'application/msword',
    'application/vnd.openxmlformats-officedocument.wordprocessingml.document',
    'text/plain',
  ]
  
  if (allowedTypes.includes(file.mimetype)) {
    cb(null, true)
  } else {
    cb(new ApiError('File type not allowed', 400))
  }
}

export const upload = multer({
  storage,
  fileFilter,
  limits: {
    fileSize: 10 * 1024 * 1024,  // 10 MB
    files: 5,
  },
})
```

---

## Step 1576: Project Controller

```typescript
// backend/src/controllers/project.controller.ts
import { Request, Response } from 'express'
import mongoose from 'mongoose'
import { Project } from '../models/Project'
import { Task } from '../models/Task'
import { catchAsync } from '../utils/catchAsync'
import { ApiError } from '../utils/ApiError'
import { io } from '../app'

export const createProject = catchAsync(async (req: Request, res: Response) => {
  const { title, description, color, deadline } = req.body
  const userId = req.user!._id
  
  const project = await Project.create({
    title,
    description,
    color,
    deadline,
    owner: userId,
    members: [{ user: userId, role: 'admin' }],
  })
  
  await project.populate('owner', 'name email avatar')
  
  res.status(201).json({ success: true, project })
})

export const getProjects = catchAsync(async (req: Request, res: Response) => {
  const userId = req.user!._id
  const { status, page = 1, limit = 10 } = req.query
  
  const query: any = {
    $or: [
      { owner: userId },
      { 'members.user': userId },
    ],
  }
  
  if (status) query.status = status
  
  const skip = (Number(page) - 1) * Number(limit)
  
  const [projects, total] = await Promise.all([
    Project.find(query)
      .populate('owner', 'name email avatar')
      .populate('members.user', 'name email avatar')
      .sort({ updatedAt: -1 })
      .skip(skip)
      .limit(Number(limit)),
    Project.countDocuments(query),
  ])
  
  // เพิ่ม task stats สำหรับแต่ละ project
  const projectsWithStats = await Promise.all(
    projects.map(async project => {
      const taskStats = await Task.aggregate([
        { $match: { project: project._id } },
        { $group: { _id: '$status', count: { $sum: 1 } } },
      ])
      
      const stats = {
        todo: 0,
        in_progress: 0,
        review: 0,
        done: 0,
        total: 0,
      }
      
      taskStats.forEach(s => {
        stats[s._id as keyof typeof stats] = s.count
        stats.total += s.count
      })
      
      return { ...project.toJSON(), taskStats: stats }
    })
  )
  
  res.json({
    success: true,
    projects: projectsWithStats,
    pagination: {
      page: Number(page),
      limit: Number(limit),
      total,
      pages: Math.ceil(total / Number(limit)),
    },
  })
})

export const updateProject = catchAsync(async (req: Request, res: Response) => {
  const { id } = req.params
  const userId = req.user!._id
  
  const project = await Project.findById(id)
  if (!project) throw new ApiError('Project not found', 404)
  
  // Check permission
  const member = project.members.find(m => m.user.toString() === userId.toString())
  const isOwner = project.owner.toString() === userId.toString()
  
  if (!isOwner && (!member || member.role !== 'admin')) {
    throw new ApiError('You do not have permission to update this project', 403)
  }
  
  const allowed = ['title', 'description', 'color', 'deadline', 'status']
  const updates: any = {}
  
  allowed.forEach(field => {
    if (req.body[field] !== undefined) {
      updates[field] = req.body[field]
    }
  })
  
  const updatedProject = await Project.findByIdAndUpdate(
    id,
    updates,
    { new: true, runValidators: true }
  ).populate('owner members.user', 'name email avatar')
  
  // Notify project members via Socket.io
  io.to(`project:${id}`).emit('project:updated', updatedProject)
  
  res.json({ success: true, project: updatedProject })
})

export const deleteProject = catchAsync(async (req: Request, res: Response) => {
  const { id } = req.params
  const userId = req.user!._id
  
  const project = await Project.findById(id)
  if (!project) throw new ApiError('Project not found', 404)
  
  if (project.owner.toString() !== userId.toString()) {
    throw new ApiError('Only project owner can delete this project', 403)
  }
  
  // Delete all tasks in project
  await Task.deleteMany({ project: id })
  
  await project.deleteOne()
  
  io.to(`project:${id}`).emit('project:deleted', { projectId: id })
  
  res.json({ success: true, message: 'Project deleted successfully' })
})

export const addMember = catchAsync(async (req: Request, res: Response) => {
  const { id } = req.params
  const { userId, role = 'editor' } = req.body
  
  const project = await Project.findById(id)
  if (!project) throw new ApiError('Project not found', 404)
  
  // Check if already a member
  const existing = project.members.find(m => m.user.toString() === userId)
  if (existing) throw new ApiError('User is already a member', 400)
  
  project.members.push({
    user: new mongoose.Types.ObjectId(userId),
    role,
    joinedAt: new Date(),
  })
  
  await project.save()
  await project.populate('members.user', 'name email avatar')
  
  res.json({ success: true, project })
})
```

---

## Step 1577: Task Controller

```typescript
// backend/src/controllers/task.controller.ts
import { Request, Response } from 'express'
import { Task } from '../models/Task'
import { Project } from '../models/Project'
import { catchAsync } from '../utils/catchAsync'
import { ApiError } from '../utils/ApiError'
import { io } from '../app'

export const createTask = catchAsync(async (req: Request, res: Response) => {
  const { title, description, projectId, status, priority, assignee, dueDate, tags } = req.body
  const userId = req.user!._id
  
  // Check project access
  const project = await Project.findById(projectId)
  if (!project) throw new ApiError('Project not found', 404)
  
  const isMember = project.members.some(m => m.user.toString() === userId.toString())
  const isOwner = project.owner.toString() === userId.toString()
  
  if (!isOwner && !isMember) {
    throw new ApiError('Access denied', 403)
  }
  
  // Get max order for status column
  const maxOrder = await Task.findOne({ project: projectId, status: status || 'todo' })
    .sort('-order')
    .select('order')
  
  const task = await Task.create({
    title,
    description,
    project: projectId,
    status: status || 'todo',
    priority: priority || 'medium',
    assignee,
    reporter: userId,
    dueDate,
    tags,
    order: maxOrder ? maxOrder.order + 1 : 0,
  })
  
  await task.populate([
    { path: 'assignee', select: 'name email avatar' },
    { path: 'reporter', select: 'name email avatar' },
  ])
  
  // Notify project room
  io.to(`project:${projectId}`).emit('task:created', task)
  
  res.status(201).json({ success: true, task })
})

export const getTasksByProject = catchAsync(async (req: Request, res: Response) => {
  const { projectId } = req.params
  const { status, assignee, priority, search } = req.query
  
  const query: any = { project: projectId }
  if (status) query.status = status
  if (assignee) query.assignee = assignee
  if (priority) query.priority = priority
  if (search) {
    query.$or = [
      { title: { $regex: search, $options: 'i' } },
      { description: { $regex: search, $options: 'i' } },
      { tags: { $in: [new RegExp(search as string, 'i')] } },
    ]
  }
  
  const tasks = await Task.find(query)
    .populate('assignee reporter', 'name email avatar')
    .sort({ status: 1, order: 1 })
  
  // Group by status สำหรับ Kanban view
  const kanban: Record<string, typeof tasks> = {
    todo: [],
    in_progress: [],
    review: [],
    done: [],
  }
  
  tasks.forEach(task => {
    if (kanban[task.status]) {
      kanban[task.status].push(task)
    }
  })
  
  res.json({ success: true, tasks, kanban })
})

export const updateTask = catchAsync(async (req: Request, res: Response) => {
  const { id } = req.params
  const userId = req.user!._id
  
  const task = await Task.findById(id).populate('project')
  if (!task) throw new ApiError('Task not found', 404)
  
  const allowed = ['title', 'description', 'status', 'priority', 'assignee', 'dueDate', 'tags']
  const updates: any = {}
  
  allowed.forEach(field => {
    if (req.body[field] !== undefined) updates[field] = req.body[field]
  })
  
  const updatedTask = await Task.findByIdAndUpdate(id, updates, {
    new: true,
    runValidators: true,
  }).populate('assignee reporter', 'name email avatar')
  
  // Emit to project room
  io.to(`project:${task.project._id}`).emit('task:updated', updatedTask)
  
  res.json({ success: true, task: updatedTask })
})

export const reorderTasks = catchAsync(async (req: Request, res: Response) => {
  const { tasks } = req.body
  // tasks = [{ id, status, order }]
  
  const bulkOps = tasks.map(({ id, status, order }: any) => ({
    updateOne: {
      filter: { _id: id },
      update: { status, order },
    },
  }))
  
  await Task.bulkWrite(bulkOps)
  
  const projectId = req.body.projectId
  io.to(`project:${projectId}`).emit('tasks:reordered', tasks)
  
  res.json({ success: true })
})

export const addComment = catchAsync(async (req: Request, res: Response) => {
  const { id } = req.params
  const { content } = req.body
  const userId = req.user!._id
  
  const task = await Task.findByIdAndUpdate(
    id,
    {
      $push: {
        comments: {
          author: userId,
          content,
        },
      },
    },
    { new: true }
  ).populate('comments.author', 'name email avatar')
  
  if (!task) throw new ApiError('Task not found', 404)
  
  const newComment = task.comments[task.comments.length - 1]
  
  io.to(`task:${id}`).emit('comment:added', {
    taskId: id,
    comment: newComment,
  })
  
  res.status(201).json({ success: true, comment: newComment })
})

export const uploadAttachment = catchAsync(async (req: Request, res: Response) => {
  const { id } = req.params
  const userId = req.user!._id
  
  if (!req.files || !Array.isArray(req.files) || req.files.length === 0) {
    throw new ApiError('No files uploaded', 400)
  }
  
  const attachments = req.files.map(file => ({
    filename: file.originalname,
    url: `/uploads/${file.filename}`,
    size: file.size,
    mimetype: file.mimetype,
    uploadedBy: userId,
  }))
  
  const task = await Task.findByIdAndUpdate(
    id,
    { $push: { attachments: { $each: attachments } } },
    { new: true }
  )
  
  if (!task) throw new ApiError('Task not found', 404)
  
  res.json({ success: true, attachments })
})
```

---

## Step 1578: Socket.io Setup

```typescript
// backend/src/socket/socketHandler.ts
import { Server, Socket } from 'socket.io'
import jwt from 'jsonwebtoken'
import { User } from '../models/User'
import logger from '../utils/logger'

interface AuthSocket extends Socket {
  user?: any
}

export function setupSocketHandlers(io: Server) {
  // Authentication middleware
  io.use(async (socket: AuthSocket, next) => {
    const token = socket.handshake.auth.token || socket.handshake.headers.authorization?.split(' ')[1]
    
    if (!token) {
      return next(new Error('Authentication required'))
    }
    
    try {
      const decoded = jwt.verify(token, process.env.JWT_SECRET as string) as { userId: string }
      const user = await User.findById(decoded.userId).select('-password')
      
      if (!user) return next(new Error('User not found'))
      
      socket.user = user
      next()
    } catch (error) {
      next(new Error('Invalid token'))
    }
  })
  
  io.on('connection', (socket: AuthSocket) => {
    logger.info(`User connected: ${socket.user?.name} (${socket.id})`)
    
    // Join project room
    socket.on('join:project', (projectId: string) => {
      socket.join(`project:${projectId}`)
      logger.info(`${socket.user?.name} joined project room: ${projectId}`)
    })
    
    // Leave project room
    socket.on('leave:project', (projectId: string) => {
      socket.leave(`project:${projectId}`)
    })
    
    // Join task room (สำหรับ real-time comments)
    socket.on('join:task', (taskId: string) => {
      socket.join(`task:${taskId}`)
    })
    
    socket.on('leave:task', (taskId: string) => {
      socket.leave(`task:${taskId}`)
    })
    
    // Typing indicator
    socket.on('typing:start', (data: { taskId: string }) => {
      socket.to(`task:${data.taskId}`).emit('typing:start', {
        user: { id: socket.user._id, name: socket.user.name },
        taskId: data.taskId,
      })
    })
    
    socket.on('typing:stop', (data: { taskId: string }) => {
      socket.to(`task:${data.taskId}`).emit('typing:stop', {
        userId: socket.user._id,
        taskId: data.taskId,
      })
    })
    
    // Disconnect
    socket.on('disconnect', () => {
      logger.info(`User disconnected: ${socket.user?.name}`)
    })
  })
}
```

---

## Step 1579: Routes

```typescript
// backend/src/routes/auth.routes.ts
import { Router } from 'express'
import * as authController from '../controllers/auth.controller'
import { protect } from '../middleware/auth'
import { validate, registerValidation } from '../middleware/validate'
import { body } from 'express-validator'

const router = Router()

router.post('/register', validate(registerValidation), authController.register)
router.post('/login', validate([
  body('email').isEmail().withMessage('Invalid email'),
  body('password').notEmpty().withMessage('Password is required'),
]), authController.login)
router.post('/refresh-token', authController.refreshToken)
router.post('/logout', protect, authController.logout)
router.get('/me', protect, authController.getMe)

export default router
```

```typescript
// backend/src/routes/project.routes.ts
import { Router } from 'express'
import * as projectController from '../controllers/project.controller'
import { protect } from '../middleware/auth'
import { validate, createProjectValidation } from '../middleware/validate'

const router = Router()

router.use(protect)  // ทุก routes ต้อง auth

router.get('/', projectController.getProjects)
router.post('/', validate(createProjectValidation), projectController.createProject)
router.get('/:id', projectController.getProjectById)
router.put('/:id', projectController.updateProject)
router.delete('/:id', projectController.deleteProject)
router.post('/:id/members', projectController.addMember)
router.delete('/:id/members/:userId', projectController.removeMember)

export default router
```

```typescript
// backend/src/routes/task.routes.ts
import { Router } from 'express'
import * as taskController from '../controllers/task.controller'
import { protect } from '../middleware/auth'
import { upload } from '../middleware/upload'

const router = Router()

router.use(protect)

router.get('/project/:projectId', taskController.getTasksByProject)
router.post('/', taskController.createTask)
router.get('/:id', taskController.getTaskById)
router.put('/:id', taskController.updateTask)
router.delete('/:id', taskController.deleteTask)
router.post('/:id/comments', taskController.addComment)
router.post('/:id/attachments', upload.array('files', 5), taskController.uploadAttachment)
router.put('/reorder', taskController.reorderTasks)

export default router
```

---

## Step 1580: Unit Tests (Backend)

```typescript
// backend/tests/auth.test.ts
import request from 'supertest'
import mongoose from 'mongoose'
import { MongoMemoryServer } from 'mongodb-memory-server'
import app from '../src/app'
import { User } from '../src/models/User'

let mongoServer: MongoMemoryServer

beforeAll(async () => {
  mongoServer = await MongoMemoryServer.create()
  await mongoose.connect(mongoServer.getUri())
})

afterAll(async () => {
  await mongoose.disconnect()
  await mongoServer.stop()
})

afterEach(async () => {
  await User.deleteMany({})
})

describe('Auth Controller', () => {
  describe('POST /api/auth/register', () => {
    it('should register a new user', async () => {
      const response = await request(app)
        .post('/api/auth/register')
        .send({
          name: 'Test User',
          email: 'test@example.com',
          password: 'Test1234!',
        })
      
      expect(response.status).toBe(201)
      expect(response.body.success).toBe(true)
      expect(response.body.user.email).toBe('test@example.com')
      expect(response.body.tokens.accessToken).toBeDefined()
      expect(response.body.user.password).toBeUndefined()
    })
    
    it('should not register with duplicate email', async () => {
      await User.create({
        name: 'Existing User',
        email: 'test@example.com',
        password: 'Test1234!',
      })
      
      const response = await request(app)
        .post('/api/auth/register')
        .send({
          name: 'New User',
          email: 'test@example.com',
          password: 'Test1234!',
        })
      
      expect(response.status).toBe(409)
    })
    
    it('should validate input', async () => {
      const response = await request(app)
        .post('/api/auth/register')
        .send({
          name: 'T',  // too short
          email: 'not-an-email',
          password: '123',  // too short
        })
      
      expect(response.status).toBe(400)
    })
  })
  
  describe('POST /api/auth/login', () => {
    beforeEach(async () => {
      await request(app)
        .post('/api/auth/register')
        .send({
          name: 'Test User',
          email: 'test@example.com',
          password: 'Test1234!',
        })
    })
    
    it('should login successfully', async () => {
      const response = await request(app)
        .post('/api/auth/login')
        .send({
          email: 'test@example.com',
          password: 'Test1234!',
        })
      
      expect(response.status).toBe(200)
      expect(response.body.tokens.accessToken).toBeDefined()
    })
    
    it('should not login with wrong password', async () => {
      const response = await request(app)
        .post('/api/auth/login')
        .send({
          email: 'test@example.com',
          password: 'WrongPassword',
        })
      
      expect(response.status).toBe(401)
    })
  })
})
```

---

## Step 1581: Frontend Setup

```json
{
  "name": "task-manager-frontend",
  "version": "1.0.0",
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "preview": "vite preview",
    "test": "vitest",
    "lint": "eslint src --ext .ts,.tsx"
  },
  "dependencies": {
    "@dnd-kit/core": "^6.1.0",
    "@dnd-kit/sortable": "^8.0.0",
    "@tanstack/react-query": "^5.14.2",
    "axios": "^1.6.3",
    "clsx": "^2.0.0",
    "date-fns": "^3.1.0",
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "react-hook-form": "^7.49.2",
    "react-router-dom": "^6.21.1",
    "socket.io-client": "^4.6.2",
    "zod": "^3.22.4",
    "zustand": "^4.4.7"
  },
  "devDependencies": {
    "@types/react": "^18.2.46",
    "@types/react-dom": "^18.2.18",
    "@vitejs/plugin-react": "^4.2.1",
    "autoprefixer": "^10.4.16",
    "postcss": "^8.4.32",
    "tailwindcss": "^3.4.0",
    "typescript": "^5.3.3",
    "vite": "^5.0.10",
    "vitest": "^1.1.0"
  }
}
```

---

## Step 1582: Frontend Types และ API

```typescript
// frontend/src/types/index.ts
export interface User {
  id: string
  name: string
  email: string
  avatar?: string
  role: 'user' | 'admin'
}

export interface Project {
  _id: string
  title: string
  description?: string
  owner: User
  members: Array<{
    user: User
    role: 'admin' | 'editor' | 'viewer'
    joinedAt: string
  }>
  status: 'active' | 'completed' | 'archived'
  color: string
  deadline?: string
  taskStats: {
    todo: number
    in_progress: number
    review: number
    done: number
    total: number
  }
  createdAt: string
  updatedAt: string
}

export type TaskStatus = 'todo' | 'in_progress' | 'review' | 'done'
export type TaskPriority = 'low' | 'medium' | 'high' | 'urgent'

export interface Task {
  _id: string
  title: string
  description?: string
  project: string | Project
  status: TaskStatus
  priority: TaskPriority
  assignee?: User
  reporter: User
  dueDate?: string
  tags: string[]
  attachments: Attachment[]
  comments: Comment[]
  order: number
  createdAt: string
  updatedAt: string
}

export interface Attachment {
  _id: string
  filename: string
  url: string
  size: number
  mimetype: string
  uploadedBy: User
  uploadedAt: string
}

export interface Comment {
  _id: string
  author: User
  content: string
  createdAt: string
}

export interface KanbanBoard {
  todo: Task[]
  in_progress: Task[]
  review: Task[]
  done: Task[]
}
```

```typescript
// frontend/src/api/index.ts
import axios from 'axios'
import { useAuthStore } from '../store/authStore'

const BASE_URL = import.meta.env.VITE_API_URL || 'http://localhost:3001/api'

export const apiClient = axios.create({
  baseURL: BASE_URL,
  headers: { 'Content-Type': 'application/json' },
})

// Request interceptor: เพิ่ม auth token
apiClient.interceptors.request.use(config => {
  const token = useAuthStore.getState().accessToken
  if (token) {
    config.headers.Authorization = `Bearer ${token}`
  }
  return config
})

// Response interceptor: จัดการ token expiry
apiClient.interceptors.response.use(
  response => response,
  async error => {
    const original = error.config
    
    if (error.response?.status === 401 && !original._retry) {
      original._retry = true
      
      try {
        const refreshToken = useAuthStore.getState().refreshToken
        const response = await axios.post(`${BASE_URL}/auth/refresh-token`, { refreshToken })
        
        const { accessToken } = response.data.tokens
        useAuthStore.getState().setTokens(accessToken, refreshToken!)
        
        original.headers.Authorization = `Bearer ${accessToken}`
        return apiClient(original)
      } catch {
        useAuthStore.getState().logout()
        window.location.href = '/login'
      }
    }
    
    return Promise.reject(error)
  }
)
```

---

## Step 1583: Auth Store (Zustand)

```typescript
// frontend/src/store/authStore.ts
import { create } from 'zustand'
import { persist } from 'zustand/middleware'
import type { User } from '../types'

interface AuthState {
  user: User | null
  accessToken: string | null
  refreshToken: string | null
  isAuthenticated: boolean
  setUser: (user: User) => void
  setTokens: (accessToken: string, refreshToken: string) => void
  logout: () => void
}

export const useAuthStore = create<AuthState>()(
  persist(
    (set) => ({
      user: null,
      accessToken: null,
      refreshToken: null,
      isAuthenticated: false,
      
      setUser: (user) => set({ user, isAuthenticated: true }),
      
      setTokens: (accessToken, refreshToken) =>
        set({ accessToken, refreshToken }),
      
      logout: () => set({
        user: null,
        accessToken: null,
        refreshToken: null,
        isAuthenticated: false,
      }),
    }),
    {
      name: 'auth-storage',
      partialize: (state) => ({
        user: state.user,
        accessToken: state.accessToken,
        refreshToken: state.refreshToken,
        isAuthenticated: state.isAuthenticated,
      }),
    }
  )
)
```

---

## Step 1584: React Components - Login Page

```tsx
// frontend/src/pages/LoginPage.tsx
import { useState } from 'react'
import { useNavigate, Link } from 'react-router-dom'
import { useForm } from 'react-hook-form'
import { zodResolver } from '@hookform/resolvers/zod'
import { z } from 'zod'
import { apiClient } from '../api'
import { useAuthStore } from '../store/authStore'

const loginSchema = z.object({
  email: z.string().email('Invalid email'),
  password: z.string().min(1, 'Password is required'),
})

type LoginFormData = z.infer<typeof loginSchema>

export default function LoginPage() {
  const navigate = useNavigate()
  const { setUser, setTokens } = useAuthStore()
  const [error, setError] = useState('')
  
  const {
    register,
    handleSubmit,
    formState: { errors, isSubmitting },
  } = useForm<LoginFormData>({
    resolver: zodResolver(loginSchema),
  })
  
  const onSubmit = async (data: LoginFormData) => {
    try {
      setError('')
      const response = await apiClient.post('/auth/login', data)
      const { user, tokens } = response.data
      
      setUser(user)
      setTokens(tokens.accessToken, tokens.refreshToken)
      
      navigate('/dashboard')
    } catch (err: any) {
      setError(err.response?.data?.error || 'Login failed')
    }
  }
  
  return (
    <div className="min-h-screen flex items-center justify-center bg-gray-50">
      <div className="max-w-md w-full bg-white rounded-xl shadow-sm p-8">
        <div className="text-center mb-8">
          <h1 className="text-2xl font-bold text-gray-900">Welcome back</h1>
          <p className="text-gray-500 mt-1">Sign in to your account</p>
        </div>
        
        {error && (
          <div className="bg-red-50 text-red-600 px-4 py-3 rounded-lg mb-6 text-sm">
            {error}
          </div>
        )}
        
        <form onSubmit={handleSubmit(onSubmit)} className="space-y-4">
          <div>
            <label className="block text-sm font-medium text-gray-700 mb-1">
              Email
            </label>
            <input
              {...register('email')}
              type="email"
              className="w-full px-3 py-2 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500"
              placeholder="you@example.com"
            />
            {errors.email && (
              <p className="text-red-500 text-sm mt-1">{errors.email.message}</p>
            )}
          </div>
          
          <div>
            <label className="block text-sm font-medium text-gray-700 mb-1">
              Password
            </label>
            <input
              {...register('password')}
              type="password"
              className="w-full px-3 py-2 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-blue-500"
              placeholder="••••••••"
            />
            {errors.password && (
              <p className="text-red-500 text-sm mt-1">{errors.password.message}</p>
            )}
          </div>
          
          <button
            type="submit"
            disabled={isSubmitting}
            className="w-full py-2.5 bg-blue-600 text-white rounded-lg font-medium hover:bg-blue-700 disabled:opacity-50 disabled:cursor-not-allowed transition-colors"
          >
            {isSubmitting ? 'Signing in...' : 'Sign in'}
          </button>
        </form>
        
        <p className="text-center text-sm text-gray-500 mt-6">
          Don't have an account?{' '}
          <Link to="/register" className="text-blue-600 hover:underline">
            Create one
          </Link>
        </p>
      </div>
    </div>
  )
}
```

---

## Step 1585: Kanban Board Component

```tsx
// frontend/src/components/KanbanBoard.tsx
import { useState, useCallback } from 'react'
import {
  DndContext,
  DragOverlay,
  closestCorners,
  PointerSensor,
  useSensor,
  useSensors,
  type DragStartEvent,
  type DragEndEvent,
} from '@dnd-kit/core'
import {
  SortableContext,
  verticalListSortingStrategy,
  arrayMove,
} from '@dnd-kit/sortable'
import { useSortable } from '@dnd-kit/sortable'
import { CSS } from '@dnd-kit/utilities'
import type { Task, TaskStatus, KanbanBoard } from '../types'
import { apiClient } from '../api'

const COLUMNS: { id: TaskStatus; title: string; color: string }[] = [
  { id: 'todo', title: 'To Do', color: 'bg-gray-100' },
  { id: 'in_progress', title: 'In Progress', color: 'bg-blue-50' },
  { id: 'review', title: 'Review', color: 'bg-yellow-50' },
  { id: 'done', title: 'Done', color: 'bg-green-50' },
]

const PRIORITY_COLORS: Record<string, string> = {
  low: 'bg-gray-100 text-gray-600',
  medium: 'bg-blue-100 text-blue-600',
  high: 'bg-orange-100 text-orange-600',
  urgent: 'bg-red-100 text-red-600',
}

interface TaskCardProps {
  task: Task
  onClick: (task: Task) => void
}

function TaskCard({ task, onClick }: TaskCardProps) {
  const {
    attributes,
    listeners,
    setNodeRef,
    transform,
    transition,
    isDragging,
  } = useSortable({ id: task._id })
  
  const style = {
    transform: CSS.Transform.toString(transform),
    transition,
    opacity: isDragging ? 0.5 : 1,
  }
  
  return (
    <div
      ref={setNodeRef}
      style={style}
      {...attributes}
      {...listeners}
      onClick={() => onClick(task)}
      className="bg-white rounded-lg p-3 shadow-sm border border-gray-200 cursor-pointer hover:shadow-md transition-shadow"
    >
      <div className="flex items-start justify-between gap-2 mb-2">
        <h3 className="text-sm font-medium text-gray-900 flex-1">{task.title}</h3>
        <span className={`text-xs px-2 py-0.5 rounded-full shrink-0 ${PRIORITY_COLORS[task.priority]}`}>
          {task.priority}
        </span>
      </div>
      
      {task.description && (
        <p className="text-xs text-gray-500 mb-2 line-clamp-2">{task.description}</p>
      )}
      
      {task.tags.length > 0 && (
        <div className="flex flex-wrap gap-1 mb-2">
          {task.tags.slice(0, 3).map(tag => (
            <span key={tag} className="text-xs bg-gray-100 text-gray-600 px-2 py-0.5 rounded">
              {tag}
            </span>
          ))}
        </div>
      )}
      
      <div className="flex items-center justify-between mt-2">
        {task.assignee ? (
          <div className="flex items-center gap-1">
            <div className="w-6 h-6 rounded-full bg-blue-500 text-white text-xs flex items-center justify-center">
              {task.assignee.name[0].toUpperCase()}
            </div>
            <span className="text-xs text-gray-500">{task.assignee.name}</span>
          </div>
        ) : (
          <span className="text-xs text-gray-400">Unassigned</span>
        )}
        
        {task.dueDate && (
          <span className="text-xs text-gray-500">
            {new Date(task.dueDate).toLocaleDateString('th-TH', { month: 'short', day: 'numeric' })}
          </span>
        )}
      </div>
      
      {(task.comments.length > 0 || task.attachments.length > 0) && (
        <div className="flex items-center gap-3 mt-2 text-xs text-gray-400">
          {task.comments.length > 0 && <span>💬 {task.comments.length}</span>}
          {task.attachments.length > 0 && <span>📎 {task.attachments.length}</span>}
        </div>
      )}
    </div>
  )
}

interface KanbanBoardProps {
  projectId: string
  initialBoard: KanbanBoard
  onTaskClick: (task: Task) => void
  onAddTask: (status: TaskStatus) => void
}

export default function KanbanBoard({
  projectId,
  initialBoard,
  onTaskClick,
  onAddTask,
}: KanbanBoardProps) {
  const [board, setBoard] = useState(initialBoard)
  const [activeTask, setActiveTask] = useState<Task | null>(null)
  
  const sensors = useSensors(
    useSensor(PointerSensor, {
      activationConstraint: { distance: 8 },
    })
  )
  
  const findColumn = useCallback((taskId: string): TaskStatus | null => {
    for (const status of Object.keys(board) as TaskStatus[]) {
      if (board[status].some(t => t._id === taskId)) {
        return status
      }
    }
    return null
  }, [board])
  
  const handleDragStart = (event: DragStartEvent) => {
    const taskId = event.active.id as string
    const status = findColumn(taskId)
    if (status) {
      const task = board[status].find(t => t._id === taskId)
      setActiveTask(task || null)
    }
  }
  
  const handleDragEnd = async (event: DragEndEvent) => {
    const { active, over } = event
    setActiveTask(null)
    
    if (!over) return
    
    const activeId = active.id as string
    const overId = over.id as string
    
    const sourceCol = findColumn(activeId)
    const targetCol = (Object.keys(board) as TaskStatus[]).includes(overId as TaskStatus)
      ? overId as TaskStatus
      : findColumn(overId)
    
    if (!sourceCol || !targetCol) return
    
    if (sourceCol === targetCol && activeId !== overId) {
      // Same column reorder
      const tasks = board[sourceCol]
      const oldIndex = tasks.findIndex(t => t._id === activeId)
      const newIndex = tasks.findIndex(t => t._id === overId)
      
      const newTasks = arrayMove(tasks, oldIndex, newIndex)
      setBoard(prev => ({ ...prev, [sourceCol]: newTasks }))
      
      // Update server
      await apiClient.put('/tasks/reorder', {
        projectId,
        tasks: newTasks.map((t, i) => ({ id: t._id, status: sourceCol, order: i })),
      })
    } else if (sourceCol !== targetCol) {
      // Move to different column
      const sourceTasks = [...board[sourceCol]]
      const targetTasks = [...board[targetCol]]
      
      const taskIndex = sourceTasks.findIndex(t => t._id === activeId)
      const [task] = sourceTasks.splice(taskIndex, 1)
      
      const newTask = { ...task, status: targetCol }
      targetTasks.push(newTask)
      
      setBoard(prev => ({
        ...prev,
        [sourceCol]: sourceTasks,
        [targetCol]: targetTasks,
      }))
      
      // Update server
      await apiClient.put(`/tasks/${activeId}`, { status: targetCol })
    }
  }
  
  return (
    <DndContext
      sensors={sensors}
      collisionDetection={closestCorners}
      onDragStart={handleDragStart}
      onDragEnd={handleDragEnd}
    >
      <div className="flex gap-4 overflow-x-auto pb-4 h-full">
        {COLUMNS.map(col => (
          <div
            key={col.id}
            className={`flex-shrink-0 w-72 rounded-xl ${col.color} p-3`}
          >
            <div className="flex items-center justify-between mb-3">
              <h2 className="font-semibold text-gray-700">
                {col.title}
                <span className="ml-2 text-xs text-gray-500">
                  ({board[col.id].length})
                </span>
              </h2>
              <button
                onClick={() => onAddTask(col.id)}
                className="text-gray-400 hover:text-gray-600 text-xl leading-none"
              >
                +
              </button>
            </div>
            
            <SortableContext
              items={board[col.id].map(t => t._id)}
              strategy={verticalListSortingStrategy}
            >
              <div className="space-y-2 min-h-24">
                {board[col.id].map(task => (
                  <TaskCard
                    key={task._id}
                    task={task}
                    onClick={onTaskClick}
                  />
                ))}
              </div>
            </SortableContext>
          </div>
        ))}
      </div>
      
      <DragOverlay>
        {activeTask && (
          <div className="opacity-90 rotate-2">
            <TaskCard task={activeTask} onClick={() => {}} />
          </div>
        )}
      </DragOverlay>
    </DndContext>
  )
}
```

---

## Step 1586: Real-time Updates Hook

```typescript
// frontend/src/hooks/useSocket.ts
import { useEffect, useRef, useCallback } from 'react'
import { io, Socket } from 'socket.io-client'
import { useAuthStore } from '../store/authStore'

const SOCKET_URL = import.meta.env.VITE_SOCKET_URL || 'http://localhost:3001'

export function useSocket() {
  const socketRef = useRef<Socket | null>(null)
  const { accessToken } = useAuthStore()
  
  useEffect(() => {
    if (!accessToken) return
    
    socketRef.current = io(SOCKET_URL, {
      auth: { token: accessToken },
      reconnectionDelay: 1000,
      reconnectionAttempts: 5,
    })
    
    socketRef.current.on('connect', () => {
      console.log('Socket connected')
    })
    
    socketRef.current.on('disconnect', () => {
      console.log('Socket disconnected')
    })
    
    socketRef.current.on('connect_error', (err) => {
      console.error('Socket error:', err.message)
    })
    
    return () => {
      socketRef.current?.disconnect()
    }
  }, [accessToken])
  
  const joinProject = useCallback((projectId: string) => {
    socketRef.current?.emit('join:project', projectId)
  }, [])
  
  const leaveProject = useCallback((projectId: string) => {
    socketRef.current?.emit('leave:project', projectId)
  }, [])
  
  const on = useCallback((event: string, handler: (...args: any[]) => void) => {
    socketRef.current?.on(event, handler)
    return () => {
      socketRef.current?.off(event, handler)
    }
  }, [])
  
  return { joinProject, leaveProject, on, socket: socketRef.current }
}
```

```typescript
// frontend/src/hooks/useProjectRealtime.ts
import { useEffect } from 'react'
import { useQueryClient } from '@tanstack/react-query'
import { useSocket } from './useSocket'
import type { Task, Project } from '../types'

export function useProjectRealtime(projectId: string) {
  const queryClient = useQueryClient()
  const { joinProject, leaveProject, on } = useSocket()
  
  useEffect(() => {
    joinProject(projectId)
    
    // Task created
    const offTaskCreated = on('task:created', (task: Task) => {
      queryClient.setQueryData(['tasks', projectId], (old: any) => {
        if (!old) return old
        return {
          ...old,
          kanban: {
            ...old.kanban,
            [task.status]: [...(old.kanban[task.status] || []), task],
          },
        }
      })
    })
    
    // Task updated
    const offTaskUpdated = on('task:updated', (updatedTask: Task) => {
      queryClient.setQueryData(['tasks', projectId], (old: any) => {
        if (!old) return old
        const newKanban = { ...old.kanban }
        
        // Remove from all columns
        Object.keys(newKanban).forEach(col => {
          newKanban[col] = newKanban[col].filter((t: Task) => t._id !== updatedTask._id)
        })
        
        // Add to correct column
        newKanban[updatedTask.status] = [...newKanban[updatedTask.status], updatedTask]
        
        return { ...old, kanban: newKanban }
      })
    })
    
    // Project updated
    const offProjectUpdated = on('project:updated', (updatedProject: Project) => {
      queryClient.setQueryData(['project', projectId], updatedProject)
    })
    
    return () => {
      leaveProject(projectId)
      offTaskCreated()
      offTaskUpdated()
      offProjectUpdated()
    }
  }, [projectId, joinProject, leaveProject, on, queryClient])
}
```

---

## Step 1587: Frontend Dockerfile

```dockerfile
# frontend/Dockerfile

# Stage 1: Dependencies
FROM node:20-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci

# Stage 2: Build
FROM node:20-alpine AS builder
WORKDIR /app
COPY --from=deps /app/node_modules ./node_modules
COPY . .
ARG VITE_API_URL
ENV VITE_API_URL=${VITE_API_URL}
RUN npm run build

# Stage 3: Production (Nginx)
FROM nginx:alpine AS production
COPY --from=builder /app/dist /usr/share/nginx/html
COPY nginx.conf /etc/nginx/conf.d/default.conf
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]

# Development stage
FROM node:20-alpine AS development
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
EXPOSE 3000
CMD ["npm", "run", "dev", "--", "--host", "0.0.0.0"]
```

```dockerfile
# backend/Dockerfile

FROM node:20-alpine AS deps
WORKDIR /app
COPY package*.json ./
RUN npm ci --only=production

FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

FROM node:20-alpine AS production
RUN apk add --no-cache dumb-init
RUN addgroup -g 1001 -S nodejs && adduser -S nodeapp -u 1001

WORKDIR /app
COPY --from=deps --chown=nodeapp:nodejs /app/node_modules ./node_modules
COPY --from=builder --chown=nodeapp:nodejs /app/dist ./dist
COPY --chown=nodeapp:nodejs package.json ./

USER nodeapp
EXPOSE 3001

HEALTHCHECK --interval=30s --timeout=5s --retries=3 \
  CMD wget -qO- http://localhost:3001/health || exit 1

ENTRYPOINT ["dumb-init", "--"]
CMD ["node", "dist/app.js"]

# Development stage
FROM node:20-alpine AS development
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
EXPOSE 3001 9229
CMD ["npm", "run", "dev"]
```

---

## Step 1588: docker-compose.yml

```yaml
# docker-compose.yml
version: '3.8'

services:
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
      target: development
    ports:
      - "3000:3000"
    volumes:
      - ./frontend/src:/app/src
      - /app/node_modules
    environment:
      VITE_API_URL: http://localhost:3001/api
      VITE_SOCKET_URL: http://localhost:3001
    depends_on:
      - backend
    networks:
      - app-network

  backend:
    build:
      context: ./backend
      dockerfile: Dockerfile
      target: development
    ports:
      - "3001:3001"
    volumes:
      - ./backend/src:/app/src
      - /app/node_modules
      - ./uploads:/app/uploads
    environment:
      NODE_ENV: development
      PORT: 3001
      MONGODB_URL: mongodb://mongo:27017/taskmanager
      REDIS_URL: redis://redis:6379
      JWT_SECRET: dev-jwt-secret-change-in-production
      REFRESH_TOKEN_SECRET: dev-refresh-secret
      FRONTEND_URL: http://localhost:3000
    depends_on:
      mongo:
        condition: service_started
      redis:
        condition: service_started
    networks:
      - app-network

  mongo:
    image: mongo:7
    volumes:
      - mongo_data:/data/db
    ports:
      - "27017:27017"
    networks:
      - app-network

  redis:
    image: redis:7-alpine
    volumes:
      - redis_data:/data
    ports:
      - "6379:6379"
    networks:
      - app-network

  mongo-express:
    image: mongo-express
    ports:
      - "8081:8081"
    environment:
      ME_CONFIG_MONGODB_SERVER: mongo
      ME_CONFIG_BASICAUTH_USERNAME: admin
      ME_CONFIG_BASICAUTH_PASSWORD: admin
    depends_on:
      - mongo
    networks:
      - app-network

networks:
  app-network:
    driver: bridge

volumes:
  mongo_data:
  redis_data:
```

---

## Step 1589: GitHub Actions CI/CD

```yaml
# .github/workflows/ci.yml
name: CI/CD Pipeline

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  # ─────────────────────────
  # Backend Tests
  # ─────────────────────────
  backend-test:
    name: Backend Tests
    runs-on: ubuntu-latest
    
    services:
      mongodb:
        image: mongo:7
        ports:
          - 27017:27017
      redis:
        image: redis:7-alpine
        ports:
          - 6379:6379
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
          cache-dependency-path: backend/package-lock.json
      
      - name: Install dependencies
        working-directory: backend
        run: npm ci
      
      - name: TypeScript check
        working-directory: backend
        run: npx tsc --noEmit
      
      - name: Run tests
        working-directory: backend
        run: npm test -- --coverage
        env:
          NODE_ENV: test
          MONGODB_URL: mongodb://localhost:27017/test
          REDIS_URL: redis://localhost:6379
          JWT_SECRET: test-secret
          REFRESH_TOKEN_SECRET: test-refresh-secret
      
      - name: Upload coverage
        uses: actions/upload-artifact@v3
        with:
          name: backend-coverage
          path: backend/coverage/
  
  # ─────────────────────────
  # Frontend Tests & Build
  # ─────────────────────────
  frontend-ci:
    name: Frontend CI
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
          cache-dependency-path: frontend/package-lock.json
      
      - name: Install dependencies
        working-directory: frontend
        run: npm ci
      
      - name: TypeScript check
        working-directory: frontend
        run: npx tsc --noEmit
      
      - name: Lint
        working-directory: frontend
        run: npm run lint
      
      - name: Test
        working-directory: frontend
        run: npm test -- --run
      
      - name: Build
        working-directory: frontend
        run: npm run build
        env:
          VITE_API_URL: https://api.example.com/api
      
      - name: Upload build
        uses: actions/upload-artifact@v3
        with:
          name: frontend-dist
          path: frontend/dist/
  
  # ─────────────────────────
  # Docker Build & Push
  # ─────────────────────────
  docker-build:
    name: Build Docker Images
    needs: [backend-test, frontend-ci]
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
      
      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}
      
      - name: Build and push backend
        uses: docker/build-push-action@v5
        with:
          context: ./backend
          target: production
          push: true
          tags: ${{ secrets.DOCKER_USERNAME }}/task-manager-backend:latest
          cache-from: type=gha
          cache-to: type=gha,mode=max
      
      - name: Build and push frontend
        uses: docker/build-push-action@v5
        with:
          context: ./frontend
          target: production
          push: true
          tags: ${{ secrets.DOCKER_USERNAME }}/task-manager-frontend:latest
          build-args: VITE_API_URL=${{ secrets.API_URL }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
  
  # ─────────────────────────
  # Deploy
  # ─────────────────────────
  deploy:
    name: Deploy to Production
    needs: docker-build
    runs-on: ubuntu-latest
    environment:
      name: production
      url: https://taskmanager.example.com
    
    steps:
      - name: Deploy via SSH
        uses: appleboy/ssh-action@v1
        with:
          host: ${{ secrets.SERVER_HOST }}
          username: ${{ secrets.SERVER_USER }}
          key: ${{ secrets.SSH_PRIVATE_KEY }}
          script: |
            cd /opt/task-manager
            docker compose pull
            docker compose up -d
            docker system prune -f
            echo "Deployed successfully!"
```

---

## Step 1590: สรุปโปรเจค และ Next Steps

### สิ่งที่เราสร้างแล้ว

**Backend (Node.js + Express + MongoDB)**
- JWT Authentication พร้อม refresh tokens
- Projects CRUD API
- Tasks CRUD API พร้อม Kanban reordering
- File uploads
- Real-time WebSocket
- Input validation ด้วย express-validator
- Unit tests ด้วย Jest + supertest

**Frontend (React + TypeScript + Vite)**
- Authentication flow
- Dashboard ด้วย React Query
- Kanban board ด้วย @dnd-kit
- Real-time updates ด้วย Socket.io
- File upload UI
- Responsive design ด้วย Tailwind

**DevOps**
- Multi-stage Dockerfiles
- docker-compose สำหรับ development
- GitHub Actions CI/CD pipeline

### สิ่งที่สามารถเพิ่มเติม

```
ฟีเจอร์เพิ่มเติม:
├── Email notifications (Nodemailer + templates)
├── OAuth2 (Google, GitHub login)
├── Team collaboration (shared workspaces)
├── Time tracking
├── Gantt chart view
├── Report generation (PDF)
├── Mobile app (React Native)
└── Advanced permissions (row-level security)

Technical improvements:
├── GraphQL API
├── Redis caching สำหรับ frequently accessed data
├── Elasticsearch สำหรับ search
├── Rate limiting per user
├── API versioning
└── Kubernetes deployment
```

### Quick Start

```bash
# Clone โปรเจค
git clone https://github.com/username/task-manager.git
cd task-manager

# Copy environment file
cp .env.example .env

# Start ด้วย Docker Compose
docker compose up -d

# App พร้อมใช้ที่:
# Frontend: http://localhost:3000
# Backend: http://localhost:3001
# API Docs: http://localhost:3001/api-docs
# MongoDB UI: http://localhost:8081
```

---

## สรุป Part 80

ในบทนี้เราได้สร้าง Full-Stack Application ครบสมบูรณ์:

1. **Backend Architecture**: MVC pattern, clean code, error handling
2. **Authentication**: JWT + Refresh Tokens + Redis
3. **Real-time**: Socket.io สำหรับ collaboration
4. **File Handling**: Multer upload
5. **Testing**: Unit tests ด้วย Jest
6. **Frontend**: React + TypeScript + Zustand + React Query
7. **UI**: Kanban board ด้วย @dnd-kit
8. **Docker**: Multi-stage builds
9. **CI/CD**: GitHub Actions pipeline
10. **Production Ready**: Health checks, graceful shutdown, logging
