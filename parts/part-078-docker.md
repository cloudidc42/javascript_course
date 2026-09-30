# Part 78: Docker กับ Node.js (Steps 1531-1550)

## บทนำ: Docker คืออะไร และทำไมถึงต้องใช้?

**Docker** คือ platform สำหรับสร้าง, ส่ง, และรัน applications ใน **containers** - environment ที่แยกตัวเองออกจาก host system แต่ใช้ kernel ร่วมกัน

### ปัญหาที่ Docker แก้ไข

```
"It works on my machine!" - ปัญหาคลาสสิกของ developer

Developer A (macOS):
  - Node.js 18
  - PostgreSQL 14
  - Redis 7
  - MongoDB 6

Server (Ubuntu):
  - Node.js 16  ← ต่างกัน!
  - PostgreSQL 12  ← ต่างกัน!

ผลลัพธ์: แอพทำงานในเครื่อง dev แต่ production พัง

Docker แก้ไข:
  ทุกคนรัน container เดียวกัน = ผลลัพธ์เหมือนกัน 100%
```

---

## Step 1531: Containers vs Virtual Machines

### Virtual Machines

```
Physical Server
├── Host OS (Linux/Windows)
└── Hypervisor (VMware, VirtualBox, KVM)
    ├── VM 1
    │   ├── Guest OS (Ubuntu) ← ขนาดใหญ่ ~1-10 GB
    │   ├── Libraries
    │   └── App 1
    ├── VM 2
    │   ├── Guest OS (Windows) ← OS แยก!
    │   └── App 2
    └── VM 3
        ├── Guest OS (CentOS)
        └── App 3

ข้อเสีย:
- ขนาดใหญ่ (GB)
- เริ่มต้นช้า (นาที)
- Resource overhead สูง
```

### Containers

```
Physical Server
├── Host OS (Linux/Windows)
└── Docker Engine
    ├── Container 1
    │   ├── App 1 + Libraries  ← เล็กกว่ามาก ~MB
    │   └── ใช้ Host OS Kernel ร่วมกัน
    ├── Container 2
    │   ├── App 2 + Libraries
    │   └── ใช้ Host OS Kernel ร่วมกัน
    └── Container 3

ข้อดี:
- ขนาดเล็ก (MB)
- เริ่มต้นเร็ว (วินาที)
- Resource overhead ต่ำ
```

---

## Step 1532: Docker Concepts

### Docker Image

**Image** คือ blueprint หรือ template สำหรับสร้าง container - read-only

```
Image ประกอบด้วย layers:
┌─────────────────────────┐
│   App Layer             │ ← โค้ดของเรา
├─────────────────────────┤
│   npm Dependencies      │ ← node_modules
├─────────────────────────┤
│   Node.js Runtime       │ ← Node.js
├─────────────────────────┤
│   Ubuntu Base           │ ← Ubuntu OS
└─────────────────────────┘

แต่ละ layer คือ diff จาก layer ก่อนหน้า
Layers ถูก cache เพื่อ build เร็วขึ้น
```

### Docker Container

**Container** คือ running instance ของ Image

```
Image (read-only) + Writable Layer = Container

Image
├── Ubuntu layer (shared)
├── Node.js layer (shared)
└── App layer (shared)
    Container 1: + Writable layer (ข้อมูล runtime)
    Container 2: + Writable layer (แยกกัน)
    Container 3: + Writable layer (แยกกัน)
```

### Docker Registry

**Registry** คือที่เก็บ Images

```
Docker Hub (hub.docker.com) ← Public registry
  - node:20-alpine
  - nginx:latest
  - postgres:15

Private Registry:
  - Amazon ECR
  - Google Container Registry
  - GitHub Container Registry (ghcr.io)
  - Self-hosted (Harbor)
```

---

## Step 1533: ติดตั้ง Docker

```bash
# Ubuntu/Debian
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh

# เพิ่ม user เข้า docker group (ไม่ต้อง sudo)
sudo usermod -aG docker $USER
newgrp docker

# ทดสอบ
docker --version
docker run hello-world
```

### คำสั่ง Docker พื้นฐาน

```bash
# Pull image
docker pull node:20-alpine

# List images
docker images
docker image ls

# Run container
docker run node:20-alpine node --version

# Run interactive
docker run -it node:20-alpine sh

# Run in background
docker run -d -p 3000:3000 --name my-app node:20-alpine

# List running containers
docker ps

# List all containers (รวม stopped)
docker ps -a

# Stop container
docker stop my-app

# Remove container
docker rm my-app

# Remove image
docker rmi node:20-alpine

# Logs
docker logs my-app
docker logs -f my-app  # follow

# Execute command in running container
docker exec -it my-app sh

# Copy file
docker cp my-app:/app/logs ./logs
```

---

## Step 1534: Dockerfile สำหรับ Node.js

**Dockerfile** คือ script สำหรับสร้าง Docker Image

```dockerfile
# Dockerfile - Node.js App

# Base image
FROM node:20-alpine

# กำหนด working directory
WORKDIR /app

# Copy package files ก่อน (ใช้ประโยชน์จาก layer caching)
COPY package*.json ./

# Install dependencies
RUN npm ci --only=production

# Copy source code
COPY . .

# Expose port
EXPOSE 3000

# คำสั่งเริ่มต้น app
CMD ["node", "server.js"]
```

### Dockerfile Instructions ที่สำคัญ

```dockerfile
# FROM - base image
FROM node:20-alpine
FROM node:20-alpine AS builder  # named stage

# WORKDIR - ตั้งค่า working directory
WORKDIR /app

# COPY - copy ไฟล์จาก host เข้า image
COPY . .                    # ก็อปทุกอย่าง
COPY package*.json ./       # ก็อปเฉพาะ package.json
COPY --from=builder /app/dist ./dist  # copy จาก stage อื่น

# RUN - รัน command ตอน build image
RUN npm ci
RUN apt-get update && apt-get install -y curl

# ENV - ตั้งค่า environment variable
ENV NODE_ENV=production
ENV PORT=3000

# ARG - build argument (ส่งตอน docker build)
ARG BUILD_VERSION=latest
ENV APP_VERSION=${BUILD_VERSION}

# EXPOSE - บอก Docker ว่า app ใช้ port อะไร
EXPOSE 3000

# VOLUME - mount point
VOLUME ["/app/data"]

# USER - กำหนด user ที่รัน
USER node

# CMD - คำสั่งเริ่มต้น (override ได้)
CMD ["node", "server.js"]
CMD ["npm", "start"]

# ENTRYPOINT - คำสั่งหลัก (override ยากกว่า)
ENTRYPOINT ["node"]
CMD ["server.js"]
```

---

## Step 1535: Multi-Stage Builds

Multi-stage builds ลดขนาด final image โดยแยก build stage กับ runtime stage

```dockerfile
# Dockerfile.multistage

##############
# Stage 1: Build
##############
FROM node:20-alpine AS builder

WORKDIR /app

COPY package*.json ./
RUN npm ci  # install ALL dependencies (รวม dev)

COPY . .
RUN npm run build  # TypeScript → JavaScript, Webpack, etc.
RUN npm run test   # รัน tests ใน build stage


##############
# Stage 2: Production
##############
FROM node:20-alpine AS production

# ติดตั้ง security patches
RUN apk add --no-cache dumb-init

# สร้าง non-root user
RUN addgroup -g 1001 -S nodejs && \
    adduser -S nextjs -u 1001

WORKDIR /app

# Copy เฉพาะที่จำเป็นจาก build stage
COPY --from=builder --chown=nextjs:nodejs /app/dist ./dist
COPY --from=builder /app/node_modules ./node_modules
COPY package*.json ./

# ใช้ non-root user
USER nextjs

EXPOSE 3000

# ใช้ dumb-init เพื่อ handle signals ได้ถูกต้อง
ENTRYPOINT ["dumb-init", "--"]
CMD ["node", "dist/server.js"]
```

```bash
# Build และดู size difference
docker build -t my-app:full -f Dockerfile .
docker build -t my-app:multi -f Dockerfile.multistage .

docker images my-app
# REPOSITORY    TAG      SIZE
# my-app        full     1.2 GB   ← ใหญ่มาก (มี dev tools)
# my-app        multi    85 MB    ← เล็กกว่า 14x
```

### ตัวอย่าง React App Multi-stage

```dockerfile
# Dockerfile สำหรับ React App

##############
# Stage 1: Build React App
##############
FROM node:20-alpine AS react-builder

WORKDIR /app

COPY package*.json ./
RUN npm ci

COPY . .
RUN npm run build  # สร้าง dist/ folder


##############
# Stage 2: Nginx Serve
##############
FROM nginx:alpine AS production

# Copy React build ไปยัง nginx
COPY --from=react-builder /app/dist /usr/share/nginx/html

# Custom nginx config
COPY nginx.conf /etc/nginx/conf.d/default.conf

EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

```nginx
# nginx.conf
server {
    listen 80;
    server_name localhost;
    
    root /usr/share/nginx/html;
    index index.html;
    
    # Handle React Router
    location / {
        try_files $uri $uri/ /index.html;
    }
    
    # Cache static assets
    location /assets/ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }
    
    # gzip
    gzip on;
    gzip_types text/plain text/css application/json application/javascript;
}
```

---

## Step 1536: .dockerignore

```
# .dockerignore
node_modules          # ← สำคัญมาก! ไม่ก็อป node_modules
dist                  # build output
.git
.gitignore
.env
.env.*
*.md
*.log
coverage
.nyc_output
.cache
.vscode
.idea
Dockerfile
docker-compose*.yml
```

### ทำไม .dockerignore ถึงสำคัญ?

```bash
# ไม่มี .dockerignore:
# Docker ส่ง context ทั้ง node_modules ไปด้วย
# Context size: 500 MB (node_modules อาจใหญ่มาก)

# มี .dockerignore:
# Docker ส่งแค่ source code
# Context size: 2 MB

docker build -t my-app .
# Sending build context to Docker daemon: 2.1MB  ← ดี!
```

---

## Step 1537: Building และ Running Containers

```bash
# Build image
docker build -t my-app:latest .
docker build -t my-app:1.0.0 .

# Build พร้อม ARG
docker build \
  --build-arg BUILD_VERSION=1.0.0 \
  --build-arg NODE_ENV=production \
  -t my-app:1.0.0 .

# Build ด้วย Dockerfile อื่น
docker build -f Dockerfile.prod -t my-app:prod .

# Run container
docker run my-app:latest

# Run พร้อม port mapping (host:container)
docker run -p 3000:3000 my-app:latest

# Run พร้อม environment variables
docker run \
  -e NODE_ENV=production \
  -e DATABASE_URL=postgresql://... \
  -p 3000:3000 \
  my-app:latest

# Run พร้อม .env file
docker run --env-file .env -p 3000:3000 my-app:latest

# Run พร้อม volume mount
docker run \
  -v $(pwd)/logs:/app/logs \
  -p 3000:3000 \
  my-app:latest

# Run แบบ background
docker run -d \
  --name my-app-container \
  --restart unless-stopped \
  -p 3000:3000 \
  my-app:latest
```

---

## Step 1538: Docker Compose - Overview

**Docker Compose** ช่วยจัดการ multi-container applications

### docker-compose.yml พื้นฐาน

```yaml
# docker-compose.yml

version: '3.8'

services:
  app:
    build: .
    ports:
      - "3000:3000"
    environment:
      NODE_ENV: development
    volumes:
      - ./src:/app/src  # hot reload
    depends_on:
      - db
  
  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
    volumes:
      - postgres_data:/var/lib/postgresql/data
    ports:
      - "5432:5432"

volumes:
  postgres_data:
```

```bash
# Start services
docker compose up

# Start ใน background
docker compose up -d

# Stop services
docker compose down

# Stop และลบ volumes
docker compose down -v

# View logs
docker compose logs -f app

# Rebuild
docker compose up --build

# Scale service
docker compose up --scale app=3
```

---

## Step 1539: docker-compose.yml ครบถ้วน

```yaml
# docker-compose.yml - Node.js + PostgreSQL + Redis Stack

version: '3.8'

services:
  # Node.js API
  api:
    build:
      context: .
      dockerfile: Dockerfile
      target: development  # ใช้ development stage
      args:
        - NODE_ENV=development
    
    ports:
      - "3000:3000"
      - "9229:9229"  # Node.js debugger
    
    environment:
      NODE_ENV: development
      PORT: 3000
      DATABASE_URL: postgresql://postgres:password@db:5432/myapp
      REDIS_URL: redis://redis:6379
      JWT_SECRET: dev-secret-key
    
    volumes:
      # Hot reload: sync source files
      - ./src:/app/src:delegated
      - ./package.json:/app/package.json
      - /app/node_modules  # ไม่ sync node_modules
    
    depends_on:
      db:
        condition: service_healthy
      redis:
        condition: service_started
    
    restart: unless-stopped
    
    networks:
      - app-network
    
    command: npm run dev  # Override CMD
  
  # PostgreSQL Database
  db:
    image: postgres:15-alpine
    
    environment:
      POSTGRES_DB: myapp
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
    
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./docker/db/init.sql:/docker-entrypoint-initdb.d/init.sql
    
    ports:
      - "5432:5432"
    
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres -d myapp"]
      interval: 10s
      timeout: 5s
      retries: 5
      start_period: 30s
    
    networks:
      - app-network
  
  # Redis Cache
  redis:
    image: redis:7-alpine
    
    command: redis-server --requirepass redispassword --appendonly yes
    
    volumes:
      - redis_data:/data
    
    ports:
      - "6379:6379"
    
    networks:
      - app-network
  
  # Adminer (Database UI)
  adminer:
    image: adminer:latest
    ports:
      - "8080:8080"
    depends_on:
      - db
    networks:
      - app-network
  
  # Redis Commander (Redis UI)
  redis-commander:
    image: rediscommander/redis-commander:latest
    environment:
      REDIS_HOSTS: redis:redis:6379:0:redispassword
    ports:
      - "8081:8081"
    depends_on:
      - redis
    networks:
      - app-network
  
  # Nginx Reverse Proxy (Production)
  nginx:
    image: nginx:alpine
    volumes:
      - ./docker/nginx/nginx.conf:/etc/nginx/nginx.conf
    ports:
      - "80:80"
      - "443:443"
    depends_on:
      - api
    networks:
      - app-network
    profiles:
      - production  # รันเฉพาะใน production profile

# Networks
networks:
  app-network:
    driver: bridge

# Volumes
volumes:
  postgres_data:
    driver: local
  redis_data:
    driver: local
```

### Production docker-compose.yml

```yaml
# docker-compose.prod.yml

version: '3.8'

services:
  api:
    build:
      context: .
      target: production
    
    environment:
      NODE_ENV: production
    
    # ไม่มี volume mount สำหรับ hot reload
    
    deploy:
      replicas: 2
      update_config:
        parallelism: 1
        delay: 10s
      restart_policy:
        condition: on-failure
```

```bash
# รัน development
docker compose up

# รัน production
docker compose -f docker-compose.yml -f docker-compose.prod.yml up
```

---

## Step 1540: Environment Variables ใน Docker

```bash
# วิธีต่างๆ ในการส่ง env vars

# 1. Inline -e flag
docker run -e NODE_ENV=production my-app

# 2. .env file
docker run --env-file .env my-app

# 3. docker-compose.yml
# (ดูตัวอย่างด้านบน)
```

```yaml
# docker-compose.yml - Environment Variables

services:
  app:
    environment:
      # Inline values
      NODE_ENV: production
      PORT: 3000
      
      # จาก host environment
      DATABASE_URL: ${DATABASE_URL}
      
      # จาก .env file (อ่านอัตโนมัติ)
    
    env_file:
      - .env
      - .env.production
```

```bash
# .env file
DATABASE_URL=postgresql://user:pass@localhost:5432/mydb
REDIS_URL=redis://localhost:6379
JWT_SECRET=super-secret-key-change-in-production
```

---

## Step 1541: Docker Volumes

**Volumes** ใช้สำหรับ persistent data - ข้อมูลไม่หายเมื่อ container ถูกลบ

### ประเภท Volumes

```bash
# 1. Named Volume (แนะนำ)
docker run -v my-data:/app/data my-app
# Data อยู่ใน Docker managed location

# 2. Bind Mount (sync กับ host filesystem)
docker run -v $(pwd)/data:/app/data my-app
# Data อยู่ใน host filesystem ตรงๆ

# 3. tmpfs Mount (in-memory เท่านั้น)
docker run --tmpfs /tmp my-app
```

### Named Volumes ใน docker-compose

```yaml
services:
  db:
    volumes:
      # Named volume (persist ข้ามการ restart)
      - postgres_data:/var/lib/postgresql/data
  
  app:
    volumes:
      # Bind mount (dev: sync code changes)
      - ./src:/app/src
      
      # Named volume (persist uploads)
      - uploads:/app/uploads
      
      # Anonymous volume (ป้องกัน overwrite)
      - /app/node_modules

volumes:
  postgres_data:
  uploads:
```

### Backup และ Restore Volumes

```bash
# Backup volume
docker run --rm \
  -v postgres_data:/data \
  -v $(pwd):/backup \
  alpine tar czf /backup/postgres_data.tar.gz -C /data .

# Restore volume
docker run --rm \
  -v postgres_data:/data \
  -v $(pwd):/backup \
  alpine tar xzf /backup/postgres_data.tar.gz -C /data
```

---

## Step 1542: Docker Networking

```bash
# List networks
docker network ls

# Create network
docker network create my-network

# Connect container to network
docker network connect my-network my-container

# Inspect network
docker network inspect my-network
```

### Network Drivers

```yaml
networks:
  frontend:
    driver: bridge  # Default - containers communicate by name
  
  backend:
    driver: bridge
  
  host_network:
    driver: host    # Share host network (no isolation)
  
  overlay:
    driver: overlay  # Multi-host (Docker Swarm)
```

### Container Communication

```yaml
# docker-compose.yml
services:
  api:
    networks:
      - frontend
      - backend
  
  db:
    networks:
      - backend  # เข้าถึงได้จาก api แต่ไม่ใช่จาก nginx
  
  nginx:
    networks:
      - frontend  # เข้าถึงได้จาก internet

networks:
  frontend:
  backend:
```

```javascript
// ใน Node.js app - ใช้ service name แทน IP
const db = new Pool({
  host: 'db',  // ← ชื่อ service ใน docker-compose
  port: 5432,
  database: 'myapp',
})

const redis = new Redis({
  host: 'redis',  // ← ชื่อ service
  port: 6379,
})
```

---

## Step 1543: Pushing ไปยัง Docker Hub

```bash
# 1. Login
docker login
# Username: your-username
# Password: your-password

# 2. Tag image
docker tag my-app:latest yourusername/my-app:latest
docker tag my-app:latest yourusername/my-app:1.0.0

# 3. Push
docker push yourusername/my-app:latest
docker push yourusername/my-app:1.0.0

# 4. Pull
docker pull yourusername/my-app:latest
```

### GitHub Container Registry

```bash
# Login ด้วย GitHub token
echo $GITHUB_TOKEN | docker login ghcr.io -u USERNAME --password-stdin

# Tag
docker tag my-app:latest ghcr.io/username/my-app:latest

# Push
docker push ghcr.io/username/my-app:latest
```

---

## Step 1544: Dockerfile สำหรับ Production

```dockerfile
# Dockerfile.production - Best Practices

##############
# Stage 1: Dependencies
##############
FROM node:20-alpine AS deps

# Check https://github.com/nodejs/docker-node/tree/main#nodealpine
RUN apk add --no-cache libc6-compat

WORKDIR /app
COPY package*.json ./

# Install production deps only
RUN npm ci --only=production --ignore-scripts


##############
# Stage 2: Build
##############
FROM node:20-alpine AS builder

WORKDIR /app

# Copy all deps (including dev)
COPY package*.json ./
RUN npm ci

# Copy source
COPY . .

# Build
RUN npm run build


##############
# Stage 3: Production Runner
##############
FROM node:20-alpine AS runner

# Security: ไม่รันเป็น root
RUN addgroup --system --gid 1001 nodejs
RUN adduser --system --uid 1001 nodeuser

# Install dumb-init สำหรับ proper signal handling
RUN apk add --no-cache dumb-init

WORKDIR /app

ENV NODE_ENV production

# Copy จาก previous stages
COPY --from=deps --chown=nodeuser:nodejs /app/node_modules ./node_modules
COPY --from=builder --chown=nodeuser:nodejs /app/dist ./dist
COPY --chown=nodeuser:nodejs package.json ./

USER nodeuser

EXPOSE 3000

ENV PORT 3000

# Health check
HEALTHCHECK --interval=30s --timeout=10s --start-period=5s --retries=3 \
  CMD node healthcheck.js

# ใช้ dumb-init เป็น PID 1 (handle SIGTERM properly)
ENTRYPOINT ["dumb-init", "--"]
CMD ["node", "dist/server.js"]
```

```javascript
// healthcheck.js
const http = require('http')

const options = {
  host: 'localhost',
  port: process.env.PORT || 3000,
  path: '/health',
  timeout: 2000,
}

const req = http.request(options, (res) => {
  if (res.statusCode === 200) {
    process.exit(0)
  } else {
    process.exit(1)
  }
})

req.on('error', () => process.exit(1))
req.on('timeout', () => process.exit(1))
req.end()
```

---

## Step 1545: Container Orchestration - Kubernetes Overview

**Kubernetes (K8s)** คือ platform สำหรับ manage containers ในระดับ production

```
ทำไมต้องการ Kubernetes?
- หลาย containers ต้องการ scale อัตโนมัติ
- Load balancing
- Self-healing (restart containers ที่ fail)
- Rolling deployments
- Secrets management
- Storage orchestration
```

### Kubernetes Concepts

```yaml
# deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: my-app
spec:
  replicas: 3  # รัน 3 copies
  selector:
    matchLabels:
      app: my-app
  template:
    metadata:
      labels:
        app: my-app
    spec:
      containers:
        - name: my-app
          image: my-app:1.0.0
          ports:
            - containerPort: 3000
          resources:
            requests:
              memory: "64Mi"
              cpu: "250m"
            limits:
              memory: "128Mi"
              cpu: "500m"
          readinessProbe:
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 5
          livenessProbe:
            httpGet:
              path: /health
              port: 3000
            initialDelaySeconds: 15

---
# service.yaml
apiVersion: v1
kind: Service
metadata:
  name: my-app-service
spec:
  selector:
    app: my-app
  ports:
    - protocol: TCP
      port: 80
      targetPort: 3000
  type: LoadBalancer
```

---

## Step 1546: Docker ใน CI/CD

```yaml
# .github/workflows/docker-ci.yml
name: Docker CI/CD

on:
  push:
    branches: [main]
  pull_request:

jobs:
  build-and-push:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
      
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
      
      - name: Login to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ secrets.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}
      
      - name: Extract metadata
        id: meta
        uses: docker/metadata-action@v5
        with:
          images: username/my-app
          tags: |
            type=ref,event=branch
            type=ref,event=pr
            type=semver,pattern={{version}}
            type=sha,prefix=
      
      - name: Build and push
        uses: docker/build-push-action@v5
        with:
          context: .
          push: ${{ github.event_name != 'pull_request' }}
          tags: ${{ steps.meta.outputs.tags }}
          labels: ${{ steps.meta.outputs.labels }}
          cache-from: type=gha
          cache-to: type=gha,mode=max
      
      - name: Security scan
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: username/my-app:latest
          format: 'table'
          exit-code: '1'
          severity: 'CRITICAL,HIGH'
```

---

## Step 1547: Docker Compose สำหรับ Development

```yaml
# docker-compose.dev.yml
version: '3.8'

services:
  api:
    build:
      context: .
      target: development
    
    volumes:
      - .:/app
      - /app/node_modules
    
    ports:
      - "3000:3000"
      - "9229:9229"  # Debug port
    
    command: npm run dev
    
    environment:
      NODE_ENV: development
      DEBUG: "app:*"
    
    stdin_open: true
    tty: true
  
  db:
    image: postgres:15-alpine
    volumes:
      - postgres_dev:/var/lib/postgresql/data
    environment:
      POSTGRES_DB: myapp_dev
      POSTGRES_USER: dev
      POSTGRES_PASSWORD: devpassword
    ports:
      - "5432:5432"
  
  redis:
    image: redis:7-alpine
    ports:
      - "6379:6379"
  
  mailhog:
    image: mailhog/mailhog
    ports:
      - "1025:1025"  # SMTP
      - "8025:8025"  # Web UI

volumes:
  postgres_dev:
```

---

## Step 1548: Production Best Practices

### 1. ใช้ Non-root User

```dockerfile
# สร้าง user สำหรับ app
RUN addgroup -g 1001 -S nodejs
RUN adduser -S nextjs -u 1001

USER nextjs  # ← ใช้ non-root user เสมอ
```

### 2. Minimal Base Images

```dockerfile
# ❌ ใหญ่เกินไป
FROM node:20

# ✅ ใช้ Alpine หรือ slim
FROM node:20-alpine
FROM node:20-slim

# ✅ ใช้ distroless (ขนาดเล็กสุด, security สูงสุด)
FROM gcr.io/distroless/nodejs20-debian11
```

### 3. Layer Caching

```dockerfile
# ❌ Copy ทุกอย่างก่อน = cache miss ทุกครั้งที่ code เปลี่ยน
COPY . .
RUN npm install

# ✅ Copy package.json ก่อน = cache hit เมื่อ code เปลี่ยนแต่ deps ไม่เปลี่ยน
COPY package*.json ./
RUN npm ci
COPY . .
```

### 4. Secrets Management

```dockerfile
# ❌ ห้าม hardcode secrets ใน Dockerfile
ENV API_KEY=secret123

# ✅ ใช้ environment variables ตอน runtime
# docker run -e API_KEY=$API_KEY my-app

# ✅ ใช้ Docker secrets (Swarm)
# หรือ Kubernetes Secrets
```

### 5. Health Checks

```dockerfile
HEALTHCHECK --interval=30s --timeout=10s --start-period=60s --retries=3 \
  CMD wget --quiet --tries=1 --spider http://localhost:3000/health || exit 1
```

### 6. Signal Handling

```javascript
// server.js - Handle graceful shutdown
process.on('SIGTERM', async () => {
  console.log('SIGTERM received, shutting down gracefully')
  
  // Stop accepting new requests
  server.close(() => {
    console.log('HTTP server closed')
  })
  
  // Close database connections
  await db.pool.end()
  
  // Close Redis
  await redis.quit()
  
  process.exit(0)
})

process.on('SIGINT', () => process.emit('SIGTERM'))
```

---

## Step 1549: ตัวอย่าง Complete Node.js Docker Setup

```dockerfile
# Dockerfile
# ════════════════════════════
# Stage 1: Install Dependencies
# ════════════════════════════
FROM node:20-alpine AS deps

RUN apk add --no-cache libc6-compat
WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production


# ════════════════════════════
# Stage 2: Build TypeScript
# ════════════════════════════
FROM node:20-alpine AS builder

WORKDIR /app
COPY package*.json ./
RUN npm ci

COPY tsconfig.json ./
COPY src/ ./src/

RUN npm run build


# ════════════════════════════
# Stage 3: Production Image
# ════════════════════════════
FROM node:20-alpine AS runner

RUN apk add --no-cache dumb-init curl

RUN addgroup --system --gid 1001 appgroup && \
    adduser --system --uid 1001 --ingroup appgroup appuser

WORKDIR /app

COPY --from=deps --chown=appuser:appgroup /app/node_modules ./node_modules
COPY --from=builder --chown=appuser:appgroup /app/dist ./dist
COPY --chown=appuser:appgroup package.json ./

USER appuser

ENV NODE_ENV=production
ENV PORT=3000

EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=5s --start-period=10s --retries=3 \
  CMD curl -f http://localhost:3000/health || exit 1

ENTRYPOINT ["dumb-init", "--"]
CMD ["node", "dist/index.js"]
```

---

## Step 1550: Complete docker-compose.yml สำหรับ Full-Stack App

```yaml
# docker-compose.yml
version: '3.8'

x-common-env: &common-env
  TZ: Asia/Bangkok
  NODE_ENV: development

services:
  # ─────────────────────────
  # API Server
  # ─────────────────────────
  api:
    build:
      context: ./backend
      dockerfile: Dockerfile
      target: development
    container_name: myapp-api
    <<: *common-env
    environment:
      PORT: 3001
      DATABASE_URL: postgresql://${DB_USER}:${DB_PASS}@postgres:5432/${DB_NAME}
      REDIS_URL: redis://:${REDIS_PASS}@redis:6379
      JWT_SECRET: ${JWT_SECRET}
      MINIO_ENDPOINT: minio
      MINIO_PORT: 9000
      MINIO_ACCESS_KEY: ${MINIO_ACCESS_KEY}
      MINIO_SECRET_KEY: ${MINIO_SECRET_KEY}
    volumes:
      - ./backend/src:/app/src
      - ./backend/package.json:/app/package.json
      - /app/node_modules
    ports:
      - "3001:3001"
      - "9229:9229"
    depends_on:
      postgres:
        condition: service_healthy
      redis:
        condition: service_started
    networks:
      - internal
    restart: unless-stopped
  
  # ─────────────────────────
  # Frontend
  # ─────────────────────────
  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
      target: development
    container_name: myapp-frontend
    environment:
      VITE_API_URL: http://localhost:3001
    volumes:
      - ./frontend/src:/app/src
      - /app/node_modules
    ports:
      - "3000:3000"
    networks:
      - internal
    restart: unless-stopped
  
  # ─────────────────────────
  # PostgreSQL
  # ─────────────────────────
  postgres:
    image: postgres:15-alpine
    container_name: myapp-postgres
    environment:
      POSTGRES_DB: ${DB_NAME:-myapp}
      POSTGRES_USER: ${DB_USER:-postgres}
      POSTGRES_PASSWORD: ${DB_PASS:-password}
    volumes:
      - postgres_data:/var/lib/postgresql/data
      - ./docker/postgres/init.sql:/docker-entrypoint-initdb.d/init.sql
    ports:
      - "5432:5432"
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER:-postgres} -d ${DB_NAME:-myapp}"]
      interval: 5s
      timeout: 5s
      retries: 5
    networks:
      - internal
  
  # ─────────────────────────
  # Redis
  # ─────────────────────────
  redis:
    image: redis:7-alpine
    container_name: myapp-redis
    command: redis-server --requirepass ${REDIS_PASS:-redispassword}
    volumes:
      - redis_data:/data
    ports:
      - "6379:6379"
    networks:
      - internal
  
  # ─────────────────────────
  # MinIO (Object Storage)
  # ─────────────────────────
  minio:
    image: minio/minio:latest
    container_name: myapp-minio
    command: server /data --console-address ":9001"
    environment:
      MINIO_ROOT_USER: ${MINIO_ACCESS_KEY:-minioadmin}
      MINIO_ROOT_PASSWORD: ${MINIO_SECRET_KEY:-minioadmin123}
    volumes:
      - minio_data:/data
    ports:
      - "9000:9000"
      - "9001:9001"
    networks:
      - internal

networks:
  internal:
    driver: bridge

volumes:
  postgres_data:
  redis_data:
  minio_data:
```

---

## แบบฝึกหัด

### แบบฝึกหัดที่ 1: Dockerize Express App

สร้าง Dockerfile สำหรับ Express API ที่มี:
- TypeScript compilation
- Multi-stage build
- Non-root user
- Health check endpoint

### แบบฝึกหัดที่ 2: Docker Compose Stack

สร้าง docker-compose.yml สำหรับ:
- Node.js API
- PostgreSQL database
- Redis cache
- Adminer (DB UI)

ทดสอบว่าทุก services communicate กันได้

### แบบฝึกหัดที่ 3: Optimize Image Size

เอา Dockerfile ที่มีอยู่แล้ว:
1. เพิ่ม .dockerignore
2. ใช้ multi-stage build
3. เปลี่ยนเป็น Alpine base
4. วัด size ก่อนและหลัง

---

## สรุป Part 78

1. **Docker** แก้ปัญหา "works on my machine" ด้วย containers
2. **Dockerfile**: สร้าง image พร้อม best practices
3. **Multi-stage builds**: ลดขนาด image อย่างมาก
4. **docker-compose**: จัดการ multi-container applications
5. **Volumes**: persistent data storage
6. **Networking**: container communication
7. **Production practices**: non-root user, health checks, signal handling
