# Part 60: GraphQL (Steps 1171-1190)

## บทนำ

GraphQL คือ query language สำหรับ API และ runtime สำหรับ executing queries นั้น สร้างโดย Facebook ในปี 2012 และ open source ในปี 2015 GraphQL ให้ client กำหนดเองว่าต้องการข้อมูลอะไร ทำให้ลด over-fetching และ under-fetching ที่เกิดขึ้นกับ REST API

---

## Step 1171: GraphQL คืออะไร และทำไมต้องใช้

```javascript
// ===== ปัญหาของ REST API ที่ GraphQL แก้ =====

// 1. Over-fetching: ได้ข้อมูลมากกว่าที่ต้องการ
// REST: GET /api/users/1
// Response: { id, name, email, phone, address, createdAt, updatedAt, ... }
// แต่เราต้องการแค่ name และ email!

// 2. Under-fetching: ต้อง request หลายครั้ง
// ต้องการ user + posts + comments
// REST: GET /api/users/1 + GET /api/users/1/posts + GET /api/posts/1/comments
// = 3 requests!

// GraphQL: 1 request เท่านั้น
/*
query {
  user(id: 1) {
    name
    email
    posts {
      title
      comments {
        content
      }
    }
  }
}
*/

// ===== GraphQL vs REST =====
/*
Feature           GraphQL              REST
Data fetching     Client-defined       Server-defined
Endpoint          Single (/graphql)    Multiple (/users, /posts, etc.)
Over-fetching     No                   Yes (usually)
Under-fetching    No                   Yes (need N+1 requests)
Versioning        Schema evolution     URL versioning (/v1, /v2)
Documentation     Auto (Schema)        Manual (Swagger)
Type system       Strong typed         No standard
Real-time         Subscriptions        WebSockets/SSE
Learning curve    Higher               Lower
*/

// ===== เมื่อไหร่ควรใช้ GraphQL? =====
// - หลาย clients ต้องการข้อมูลต่างกัน (mobile vs web)
// - Data relationships ซับซ้อน
// - Rapid product development
// - Real-time data

// ===== เมื่อไหร่ควรใช้ REST แทน? =====
// - Simple CRUD operations
// - File uploads (GraphQL ทำได้แต่ยาก)
// - HTTP caching สำคัญมาก
// - ทีมคุ้นเคยกับ REST
```

---

## Step 1172: Schema Definition Language (SDL)

```graphql
# ===== SDL Basics =====

# Scalar types: String, Int, Float, Boolean, ID
# ! = Non-null (required)
# [] = List

# Object Type
type User {
  id: ID!
  name: String!
  email: String!
  age: Int
  isActive: Boolean!
  profile: Profile
  posts: [Post!]!
  createdAt: String!
}

type Profile {
  bio: String
  avatar: String
  website: String
}

type Post {
  id: ID!
  title: String!
  content: String!
  published: Boolean!
  author: User!
  tags: [String!]!
  comments: [Comment!]!
  createdAt: String!
  updatedAt: String!
}

type Comment {
  id: ID!
  content: String!
  author: User!
  post: Post!
  createdAt: String!
}

# Enum Type
enum UserRole {
  ADMIN
  MODERATOR
  USER
}

enum PostStatus {
  DRAFT
  PUBLISHED
  ARCHIVED
}

# Input Type (สำหรับ mutations)
input CreateUserInput {
  name: String!
  email: String!
  password: String!
  role: UserRole = USER
}

input UpdateUserInput {
  name: String
  email: String
  bio: String
  website: String
}

input CreatePostInput {
  title: String!
  content: String!
  tags: [String!]!
  status: PostStatus = DRAFT
}

input PostFilterInput {
  status: PostStatus
  authorId: ID
  tag: String
  search: String
}

input PaginationInput {
  page: Int = 1
  limit: Int = 10
}

# Interface
interface Node {
  id: ID!
}

interface Timestamped {
  createdAt: String!
  updatedAt: String!
}

type Article implements Node & Timestamped {
  id: ID!
  title: String!
  createdAt: String!
  updatedAt: String!
}

# Union Type
union SearchResult = User | Post | Comment

# ===== Query Type (read operations) =====
type Query {
  # User queries
  user(id: ID!): User
  users(filter: UserFilterInput, pagination: PaginationInput): UserConnection!
  me: User
  
  # Post queries
  post(id: ID!): Post
  postBySlug(slug: String!): Post
  posts(filter: PostFilterInput, pagination: PaginationInput): PostConnection!
  
  # Search
  search(query: String!): [SearchResult!]!
}

# Pagination types
type UserConnection {
  nodes: [User!]!
  totalCount: Int!
  pageInfo: PageInfo!
}

type PostConnection {
  nodes: [Post!]!
  totalCount: Int!
  pageInfo: PageInfo!
}

type PageInfo {
  hasNextPage: Boolean!
  hasPreviousPage: Boolean!
  startCursor: String
  endCursor: String
}

# ===== Mutation Type (write operations) =====
type Mutation {
  # Auth
  register(input: CreateUserInput!): AuthPayload!
  login(email: String!, password: String!): AuthPayload!
  logout: Boolean!
  
  # User mutations
  updateUser(id: ID!, input: UpdateUserInput!): User!
  deleteUser(id: ID!): Boolean!
  
  # Post mutations
  createPost(input: CreatePostInput!): Post!
  updatePost(id: ID!, input: UpdatePostInput!): Post!
  publishPost(id: ID!): Post!
  deletePost(id: ID!): Boolean!
  
  # Comment mutations
  addComment(postId: ID!, content: String!): Comment!
  deleteComment(id: ID!): Boolean!
}

type AuthPayload {
  token: String!
  user: User!
}

# ===== Subscription Type (real-time) =====
type Subscription {
  postAdded: Post!
  commentAdded(postId: ID!): Comment!
  userStatusChanged(userId: ID!): UserStatusEvent!
}

type UserStatusEvent {
  userId: ID!
  isOnline: Boolean!
  lastSeen: String
}
```

---

## Step 1173: Setting Up Apollo Server

```bash
# ติดตั้ง packages
npm install @apollo/server graphql
npm install @apollo/server-express graphql express body-parser cors

# หรือ standalone
npm install @apollo/server graphql
```

```javascript
// server.js - Apollo Server Standalone
const { ApolloServer } = require('@apollo/server');
const { startStandaloneServer } = require('@apollo/server/standalone');

// Type definitions
const typeDefs = `#graphql
  type Book {
    id: ID!
    title: String!
    author: Author!
    year: Int
    genre: String
  }

  type Author {
    id: ID!
    name: String!
    books: [Book!]!
  }

  type Query {
    books: [Book!]!
    book(id: ID!): Book
    authors: [Author!]!
    author(id: ID!): Author
  }

  type Mutation {
    createBook(title: String!, authorId: ID!, year: Int, genre: String): Book!
    deleteBook(id: ID!): Boolean!
  }
`;

// Mock data
const authors = [
  { id: '1', name: 'J.K. Rowling' },
  { id: '2', name: 'George R.R. Martin' },
  { id: '3', name: 'Frank Herbert' }
];

const books = [
  { id: '1', title: 'Harry Potter', authorId: '1', year: 1997, genre: 'Fantasy' },
  { id: '2', title: 'A Game of Thrones', authorId: '2', year: 1996, genre: 'Fantasy' },
  { id: '3', title: 'Dune', authorId: '3', year: 1965, genre: 'Sci-Fi' }
];

// Resolvers
const resolvers = {
  Query: {
    books: () => books,
    book: (_, { id }) => books.find(b => b.id === id),
    authors: () => authors,
    author: (_, { id }) => authors.find(a => a.id === id)
  },
  
  Mutation: {
    createBook: (_, { title, authorId, year, genre }) => {
      const book = {
        id: String(books.length + 1),
        title,
        authorId,
        year,
        genre
      };
      books.push(book);
      return book;
    },
    
    deleteBook: (_, { id }) => {
      const index = books.findIndex(b => b.id === id);
      if (index === -1) return false;
      books.splice(index, 1);
      return true;
    }
  },
  
  // Type resolvers
  Book: {
    author: (book) => authors.find(a => a.id === book.authorId)
  },
  
  Author: {
    books: (author) => books.filter(b => b.authorId === author.id)
  }
};

// Create server
const server = new ApolloServer({ typeDefs, resolvers });

// Start server
async function startServer() {
  const { url } = await startStandaloneServer(server, {
    listen: { port: 4000 },
    context: async ({ req }) => {
      // Context factory - ใส่ข้อมูลที่ทุก resolver ต้องใช้
      const token = req.headers.authorization || '';
      const user = await getUserFromToken(token);
      return { user };
    }
  });
  
  console.log(`GraphQL server ready at ${url}`);
  console.log(`Playground available at ${url}`);
}

startServer();
```

---

## Step 1174: GraphQL กับ Express

```javascript
const express = require('express');
const { ApolloServer } = require('@apollo/server');
const { expressMiddleware } = require('@apollo/server/express4');
const { ApolloServerPluginDrainHttpServer } = require('@apollo/server/plugin/drainHttpServer');
const http = require('http');
const cors = require('cors');
const bodyParser = require('body-parser');

const typeDefs = `#graphql
  type Query {
    hello: String
    users: [User!]!
  }
  
  type User {
    id: ID!
    name: String!
    email: String!
  }
  
  type Mutation {
    createUser(name: String!, email: String!): User!
  }
`;

const users = [{ id: '1', name: 'Alice', email: 'alice@example.com' }];

const resolvers = {
  Query: {
    hello: () => 'Hello, GraphQL!',
    users: () => users
  },
  Mutation: {
    createUser: (_, { name, email }) => {
      const user = { id: String(users.length + 1), name, email };
      users.push(user);
      return user;
    }
  }
};

async function startApolloServer() {
  const app = express();
  const httpServer = http.createServer(app);
  
  const server = new ApolloServer({
    typeDefs,
    resolvers,
    plugins: [
      ApolloServerPluginDrainHttpServer({ httpServer })
    ],
    
    // Formatting errors
    formatError: (formattedError, error) => {
      // ไม่แสดง stack trace ใน production
      if (process.env.NODE_ENV === 'production') {
        return {
          message: formattedError.message,
          extensions: {
            code: formattedError.extensions?.code
          }
        };
      }
      return formattedError;
    }
  });
  
  await server.start();
  
  // Express middleware
  app.use(cors());
  app.use(bodyParser.json());
  
  // REST routes ยังใช้ได้
  app.get('/health', (req, res) => {
    res.json({ status: 'ok' });
  });
  
  // GraphQL endpoint
  app.use(
    '/graphql',
    cors(),
    bodyParser.json(),
    expressMiddleware(server, {
      context: async ({ req }) => {
        // Extract user from JWT
        const token = req.headers.authorization?.split(' ')[1];
        const user = token ? await verifyToken(token) : null;
        return { user, req };
      }
    })
  );
  
  await new Promise(resolve => httpServer.listen({ port: 4000 }, resolve));
  
  console.log('Server ready at http://localhost:4000/graphql');
}

startApolloServer();
```

---

## Step 1175: Queries

```graphql
# ===== Basic Queries =====

# Query ง่าย
query {
  users {
    id
    name
    email
  }
}

# Query พร้อม arguments
query {
  user(id: "1") {
    id
    name
    posts {
      id
      title
    }
  }
}

# Named query (แนะนำสำหรับ production)
query GetUser($id: ID!) {
  user(id: $id) {
    id
    name
    email
    profile {
      bio
      avatar
    }
  }
}

# Multiple queries ในครั้งเดียว
query GetDashboardData {
  me {
    name
    email
  }
  posts(filter: { status: PUBLISHED }, pagination: { limit: 5 }) {
    nodes {
      id
      title
      createdAt
    }
    totalCount
  }
  recentActivity {
    type
    timestamp
  }
}

# Aliases - query field เดียวกันหลายครั้ง
query GetTwoUsers {
  alice: user(id: "1") {
    name
    email
  }
  bob: user(id: "2") {
    name
    email
  }
}

# Nested queries
query GetPostWithDetails {
  post(id: "1") {
    id
    title
    content
    author {
      name
      email
      profile {
        avatar
      }
    }
    comments {
      id
      content
      author {
        name
      }
      createdAt
    }
    tags
  }
}
```

```javascript
// ===== Resolvers สำหรับ Queries =====

const { GraphQLError } = require('graphql');

const resolvers = {
  Query: {
    // Simple resolver
    hello: () => 'Hello, World!',
    
    // Resolver with args
    user: async (_, { id }, context) => {
      const user = await context.dataSources.userAPI.getUserById(id);
      if (!user) {
        throw new GraphQLError(`User ${id} not found`, {
          extensions: { code: 'NOT_FOUND' }
        });
      }
      return user;
    },
    
    // Resolver with filtering and pagination
    users: async (_, { filter = {}, pagination = {} }, context) => {
      const { page = 1, limit = 10 } = pagination;
      const { search, role, isActive } = filter;
      
      const result = await context.dataSources.userAPI.getUsers({
        search,
        role,
        isActive,
        page,
        limit
      });
      
      return {
        nodes: result.users,
        totalCount: result.total,
        pageInfo: {
          hasNextPage: page * limit < result.total,
          hasPreviousPage: page > 1
        }
      };
    },
    
    // Authenticated query
    me: async (_, __, context) => {
      if (!context.user) {
        throw new GraphQLError('Not authenticated', {
          extensions: { code: 'UNAUTHENTICATED' }
        });
      }
      return context.user;
    },
    
    // Search across types
    search: async (_, { query }, context) => {
      const [users, posts, comments] = await Promise.all([
        context.dataSources.userAPI.search(query),
        context.dataSources.postAPI.search(query),
        context.dataSources.commentAPI.search(query)
      ]);
      
      return [...users, ...posts, ...comments];
    }
  },
  
  // Type resolver: กำหนด __resolveType สำหรับ Union/Interface
  SearchResult: {
    __resolveType: (obj) => {
      if (obj.postId) return 'Comment';
      if (obj.content && obj.published !== undefined) return 'Post';
      return 'User';
    }
  }
};
```

---

## Step 1176: Mutations

```graphql
# ===== Mutations =====

# Create
mutation CreateUser($input: CreateUserInput!) {
  createUser(input: $input) {
    id
    name
    email
    role
    createdAt
  }
}

# Variables:
# {
#   "input": {
#     "name": "Alice",
#     "email": "alice@example.com",
#     "password": "Password123!"
#   }
# }

# Login
mutation Login($email: String!, $password: String!) {
  login(email: $email, password: $password) {
    token
    user {
      id
      name
      email
      role
    }
  }
}

# Update
mutation UpdatePost($id: ID!, $input: UpdatePostInput!) {
  updatePost(id: $id, input: $input) {
    id
    title
    content
    updatedAt
  }
}

# Delete
mutation DeletePost($id: ID!) {
  deletePost(id: $id)
}

# Multiple mutations (sequential)
mutation SetupProfile {
  updateUser(id: "1", input: { name: "Alice Smith" }) {
    id
    name
  }
  createPost(input: {
    title: "My First Post"
    content: "Hello, GraphQL!"
    tags: ["graphql", "nodejs"]
  }) {
    id
    title
  }
}
```

```javascript
// ===== Mutation Resolvers =====
const bcrypt = require('bcrypt');
const jwt = require('jsonwebtoken');

const mutationResolvers = {
  Mutation: {
    register: async (_, { input }, context) => {
      const { name, email, password } = input;
      
      // ตรวจสอบ email ซ้ำ
      const existing = await context.dataSources.userAPI.getUserByEmail(email);
      if (existing) {
        throw new GraphQLError('Email already registered', {
          extensions: {
            code: 'CONFLICT',
            field: 'email'
          }
        });
      }
      
      // Hash password
      const passwordHash = await bcrypt.hash(password, 12);
      
      // สร้าง user
      const user = await context.dataSources.userAPI.createUser({
        name,
        email,
        passwordHash,
        role: input.role || 'USER'
      });
      
      // สร้าง token
      const token = jwt.sign(
        { sub: user.id, email: user.email, role: user.role },
        process.env.JWT_SECRET,
        { expiresIn: '24h' }
      );
      
      return { token, user };
    },
    
    login: async (_, { email, password }, context) => {
      const user = await context.dataSources.userAPI.getUserByEmail(email);
      
      if (!user || !await bcrypt.compare(password, user.passwordHash)) {
        throw new GraphQLError('Invalid credentials', {
          extensions: { code: 'UNAUTHENTICATED' }
        });
      }
      
      const token = jwt.sign(
        { sub: user.id, email: user.email, role: user.role },
        process.env.JWT_SECRET,
        { expiresIn: '24h' }
      );
      
      return { token, user };
    },
    
    createPost: async (_, { input }, context) => {
      if (!context.user) {
        throw new GraphQLError('Not authenticated', {
          extensions: { code: 'UNAUTHENTICATED' }
        });
      }
      
      const post = await context.dataSources.postAPI.createPost({
        ...input,
        authorId: context.user.id
      });
      
      // Notify subscribers
      context.pubsub.publish('POST_ADDED', { postAdded: post });
      
      return post;
    },
    
    updatePost: async (_, { id, input }, context) => {
      if (!context.user) {
        throw new GraphQLError('Not authenticated', {
          extensions: { code: 'UNAUTHENTICATED' }
        });
      }
      
      const post = await context.dataSources.postAPI.getPostById(id);
      
      if (!post) {
        throw new GraphQLError('Post not found', {
          extensions: { code: 'NOT_FOUND' }
        });
      }
      
      // ตรวจสอบว่าเป็น author หรือ admin
      if (post.authorId !== context.user.id && context.user.role !== 'ADMIN') {
        throw new GraphQLError('Forbidden', {
          extensions: { code: 'FORBIDDEN' }
        });
      }
      
      return await context.dataSources.postAPI.updatePost(id, input);
    },
    
    deletePost: async (_, { id }, context) => {
      if (!context.user) {
        throw new GraphQLError('Not authenticated', {
          extensions: { code: 'UNAUTHENTICATED' }
        });
      }
      
      const post = await context.dataSources.postAPI.getPostById(id);
      if (!post) return false;
      
      if (post.authorId !== context.user.id && context.user.role !== 'ADMIN') {
        throw new GraphQLError('Forbidden', {
          extensions: { code: 'FORBIDDEN' }
        });
      }
      
      await context.dataSources.postAPI.deletePost(id);
      return true;
    }
  }
};
```

---

## Step 1177: Subscriptions (Real-time)

```javascript
// ===== Subscriptions ด้วย WebSocket =====

const { ApolloServer } = require('@apollo/server');
const { expressMiddleware } = require('@apollo/server/express4');
const { makeExecutableSchema } = require('@graphql-tools/schema');
const { WebSocketServer } = require('ws');
const { useServer } = require('graphql-ws/lib/use/ws');
const { PubSub } = require('graphql-subscriptions');
const { ApolloServerPluginDrainHttpServer } = require('@apollo/server/plugin/drainHttpServer');

const http = require('http');
const express = require('express');
const cors = require('cors');
const bodyParser = require('body-parser');

const pubsub = new PubSub();

const EVENTS = {
  POST_ADDED: 'POST_ADDED',
  COMMENT_ADDED: 'COMMENT_ADDED',
  MESSAGE_SENT: 'MESSAGE_SENT'
};

const typeDefs = `#graphql
  type Post {
    id: ID!
    title: String!
    content: String!
    author: User!
    createdAt: String!
  }
  
  type Comment {
    id: ID!
    content: String!
    author: User!
    post: Post!
  }
  
  type Message {
    id: ID!
    content: String!
    sender: User!
    roomId: String!
    createdAt: String!
  }
  
  type User {
    id: ID!
    name: String!
  }
  
  type Query {
    posts: [Post!]!
    messages(roomId: String!): [Message!]!
  }
  
  type Mutation {
    createPost(title: String!, content: String!): Post!
    addComment(postId: ID!, content: String!): Comment!
    sendMessage(roomId: String!, content: String!): Message!
  }
  
  type Subscription {
    postAdded: Post!
    commentAdded(postId: ID!): Comment!
    messageSent(roomId: String!): Message!
    userTyping(roomId: String!): String!
  }
`;

const resolvers = {
  Mutation: {
    createPost: async (_, { title, content }, context) => {
      const post = {
        id: String(Date.now()),
        title,
        content,
        authorId: context.user?.id || '1',
        createdAt: new Date().toISOString()
      };
      
      // Publish event
      pubsub.publish(EVENTS.POST_ADDED, { postAdded: post });
      
      return post;
    },
    
    addComment: async (_, { postId, content }, context) => {
      const comment = {
        id: String(Date.now()),
        postId,
        content,
        authorId: context.user?.id || '1',
        createdAt: new Date().toISOString()
      };
      
      pubsub.publish(EVENTS.COMMENT_ADDED, {
        commentAdded: comment,
        postId
      });
      
      return comment;
    },
    
    sendMessage: async (_, { roomId, content }, context) => {
      const message = {
        id: String(Date.now()),
        content,
        roomId,
        senderId: context.user?.id || '1',
        createdAt: new Date().toISOString()
      };
      
      pubsub.publish(EVENTS.MESSAGE_SENT, {
        messageSent: message,
        roomId
      });
      
      return message;
    }
  },
  
  Subscription: {
    postAdded: {
      subscribe: () => pubsub.asyncIterator([EVENTS.POST_ADDED])
    },
    
    commentAdded: {
      subscribe: (_, { postId }) => {
        return pubsub.asyncIterator([EVENTS.COMMENT_ADDED]);
      },
      resolve: (payload, { postId }) => {
        // กรองเฉพาะ comments ของ postId ที่ subscribe
        if (payload.postId !== postId) return null;
        return payload.commentAdded;
      }
    },
    
    messageSent: {
      subscribe: (_, { roomId }) => {
        return pubsub.asyncIterator([EVENTS.MESSAGE_SENT]);
      },
      resolve: (payload, { roomId }) => {
        if (payload.roomId !== roomId) return null;
        return payload.messageSent;
      }
    }
  }
};

// Setup server
async function startServer() {
  const app = express();
  const httpServer = http.createServer(app);
  
  // WebSocket server for subscriptions
  const wsServer = new WebSocketServer({
    server: httpServer,
    path: '/graphql'
  });
  
  const schema = makeExecutableSchema({ typeDefs, resolvers });
  
  const serverCleanup = useServer({
    schema,
    context: async (ctx) => {
      const token = ctx.connectionParams?.authorization;
      const user = token ? await verifyToken(token) : null;
      return { user, pubsub };
    }
  }, wsServer);
  
  const apolloServer = new ApolloServer({
    schema,
    plugins: [
      ApolloServerPluginDrainHttpServer({ httpServer }),
      {
        async serverWillStart() {
          return {
            async drainServer() {
              await serverCleanup.dispose();
            }
          };
        }
      }
    ]
  });
  
  await apolloServer.start();
  
  app.use('/graphql', cors(), bodyParser.json(),
    expressMiddleware(apolloServer, {
      context: async ({ req }) => {
        const token = req.headers.authorization?.split(' ')[1];
        const user = token ? await verifyToken(token) : null;
        return { user, pubsub };
      }
    })
  );
  
  await new Promise(resolve => httpServer.listen({ port: 4000 }, resolve));
  console.log('Server ready at http://localhost:4000/graphql');
  console.log('WebSocket ready at ws://localhost:4000/graphql');
}

startServer();
```

```graphql
# Client subscription examples
subscription OnNewPost {
  postAdded {
    id
    title
    author {
      name
    }
    createdAt
  }
}

subscription OnNewComment($postId: ID!) {
  commentAdded(postId: $postId) {
    id
    content
    author {
      name
    }
  }
}

subscription ChatRoom($roomId: String!) {
  messageSent(roomId: $roomId) {
    id
    content
    sender {
      name
    }
    createdAt
  }
}
```

---

## Step 1178: Resolvers ขั้นสูง

```javascript
// ===== Resolver arguments =====
// ทุก resolver รับ 4 arguments:
// 1. parent (root) - result จาก parent resolver
// 2. args - arguments จาก query/mutation
// 3. context - shared context (user, db, dataSources)
// 4. info - query metadata

const resolvers = {
  Query: {
    // parent = root query object (usually empty)
    posts: async (parent, args, context, info) => {
      console.log('Requested fields:', info.fieldNodes[0].selectionSet.selections.map(s => s.name.value));
      
      return context.db.posts.findAll();
    }
  },
  
  Post: {
    // parent = post object
    author: async (post, _, context) => {
      return context.db.users.findById(post.authorId);
    },
    
    comments: async (post, { limit = 10 }, context) => {
      return context.db.comments.findByPostId(post.id, { limit });
    },
    
    // computed field
    commentCount: async (post, _, context) => {
      return context.db.comments.countByPostId(post.id);
    },
    
    // field with default transformation
    excerpt: (post, { length = 200 }) => {
      return post.content.slice(0, length) + (post.content.length > length ? '...' : '');
    }
  },
  
  // Default field resolvers
  User: {
    // ถ้าไม่กำหนด resolver จะ return object property ที่มีชื่อตรงกัน
    // แต่ถ้าชื่อไม่ตรงต้องกำหนดเอง
    displayName: (user) => `${user.firstName} ${user.lastName}`,
    
    // Lazy load
    posts: async (user, _, context) => {
      // ไม่ load ถ้า client ไม่ได้ request field นี้
      return context.db.posts.findByAuthorId(user.id);
    }
  }
};

// ===== Resolver chaining =====
/*
Query:
  posts {          <- Query.posts resolver
    title          <- Post.title resolver (default)
    author {       <- Post.author resolver
      name         <- User.name resolver (default)
      posts {      <- User.posts resolver
        title      <- Post.title resolver (default)
      }
    }
  }
*/

// ===== Context =====
// Context factory ถูกเรียกทุก request
const context = async ({ req }) => {
  const token = req.headers.authorization?.slice(7);
  let user = null;
  
  if (token) {
    try {
      const decoded = jwt.verify(token, process.env.JWT_SECRET);
      user = await getUserById(decoded.sub);
    } catch {}
  }
  
  return {
    user,
    db,                    // database connection
    dataSources: {         // data access objects
      userAPI: new UserAPI(db),
      postAPI: new PostAPI(db)
    },
    pubsub,               // for subscriptions
    loaders: {            // DataLoader instances (N+1 prevention)
      userLoader: createUserLoader(db),
      postLoader: createPostLoader(db)
    }
  };
};
```

---

## Step 1179: Query Variables

```javascript
// ===== Query Variables =====
// Variables ทำให้ queries dynamic และปลอดภัยกว่า string interpolation

// ตัวอย่าง query
const GET_USER = `
  query GetUser($id: ID!) {
    user(id: $id) {
      id
      name
      email
    }
  }
`;

// Variables object
const variables = { id: '123' };

// ===== ใช้ใน Apollo Client (frontend) =====
// import { useQuery } from '@apollo/client';
// const { data } = useQuery(GET_USER, { variables: { id: '123' } });

// ===== ส่งผ่าน HTTP POST =====
// POST /graphql
// Content-Type: application/json
/*
{
  "query": "query GetUser($id: ID!) { user(id: $id) { id name } }",
  "variables": { "id": "123" }
}
*/

// ===== ตัวอย่าง variables ซับซ้อน =====
const CREATE_POST = `
  mutation CreatePost($input: CreatePostInput!) {
    createPost(input: $input) {
      id
      title
      slug
      author {
        name
      }
    }
  }
`;

const createPostVariables = {
  input: {
    title: 'My New Post',
    content: 'This is the content...',
    tags: ['graphql', 'javascript'],
    status: 'DRAFT'
  }
};

// ===== Default variables =====
const GET_POSTS = `
  query GetPosts($page: Int = 1, $limit: Int = 10, $status: PostStatus = PUBLISHED) {
    posts(pagination: { page: $page, limit: $limit }, filter: { status: $status }) {
      nodes {
        id
        title
        createdAt
      }
      totalCount
    }
  }
`;

// ===== Testing with Apollo Studio หรือ Insomnia =====
// Query:
// query GetPosts($page: Int, $limit: Int) {
//   posts(pagination: { page: $page, limit: $limit }) {
//     nodes { id title }
//     totalCount
//   }
// }
//
// Variables:
// { "page": 2, "limit": 5 }
```

---

## Step 1180: Fragments

```graphql
# ===== Fragments =====
# ใช้ fragment เพื่อ reuse selection sets

# กำหนด fragment
fragment UserBasic on User {
  id
  name
  email
}

fragment UserFull on User {
  ...UserBasic
  role
  createdAt
  profile {
    bio
    avatar
    website
  }
}

fragment PostPreview on Post {
  id
  title
  excerpt(length: 150)
  author {
    ...UserBasic
  }
  tags
  createdAt
}

fragment CommentDetails on Comment {
  id
  content
  author {
    ...UserBasic
  }
  createdAt
}

# ใช้ fragment
query GetFeed {
  posts {
    nodes {
      ...PostPreview
      commentCount
    }
  }
}

query GetPostDetails($id: ID!) {
  post(id: $id) {
    ...PostPreview
    content
    comments {
      ...CommentDetails
    }
  }
}

query GetProfile($id: ID!) {
  user(id: $id) {
    ...UserFull
    posts {
      ...PostPreview
    }
  }
}

# Inline fragments (สำหรับ Union/Interface)
query SearchAll($query: String!) {
  search(query: $query) {
    ... on User {
      id
      name
      email
    }
    ... on Post {
      id
      title
      author {
        name
      }
    }
    ... on Comment {
      id
      content
      post {
        title
      }
    }
  }
}
```

```javascript
// ===== Fragment ใน Apollo Client =====
const { gql } = require('@apollo/client');

const USER_BASIC_FRAGMENT = gql`
  fragment UserBasic on User {
    id
    name
    email
  }
`;

const GET_POSTS = gql`
  ${USER_BASIC_FRAGMENT}
  
  query GetPosts {
    posts {
      nodes {
        id
        title
        author {
          ...UserBasic
        }
      }
    }
  }
`;
```

---

## Step 1181: Directives

```graphql
# ===== Built-in Directives =====

# @include - include field if condition is true
query GetUser($id: ID!, $withPosts: Boolean!) {
  user(id: $id) {
    id
    name
    posts @include(if: $withPosts) {
      id
      title
    }
  }
}

# Variables: { "id": "1", "withPosts": true }

# @skip - skip field if condition is true
query GetUser($id: ID!, $skipProfile: Boolean!) {
  user(id: $id) {
    id
    name
    profile @skip(if: $skipProfile) {
      bio
      avatar
    }
  }
}

# @deprecated - mark field as deprecated
type User {
  id: ID!
  name: String!
  username: String! @deprecated(reason: "Use 'name' instead")
}

# ===== Custom Directives =====
# ใน schema
directive @auth(requires: Role = USER) on FIELD_DEFINITION
directive @rateLimit(max: Int!, window: Int!) on FIELD_DEFINITION
directive @cache(ttl: Int! = 60) on FIELD_DEFINITION
directive @uppercase on FIELD_DEFINITION

enum Role {
  ADMIN
  MODERATOR
  USER
}

type Query {
  me: User @auth
  adminPanel: AdminData @auth(requires: ADMIN)
  expensiveQuery: String @rateLimit(max: 10, window: 60)
  staticConfig: Config @cache(ttl: 3600)
}

type User {
  name: String @uppercase
}
```

```javascript
// ===== Custom directive implementation =====
const { mapSchema, getDirective, MapperKind } = require('@graphql-tools/utils');
const { defaultFieldResolver, GraphQLError } = require('graphql');

// @auth directive
function authDirectiveTransformer(schema) {
  return mapSchema(schema, {
    [MapperKind.OBJECT_FIELD]: (fieldConfig) => {
      const authDirective = getDirective(schema, fieldConfig, 'auth')?.[0];
      
      if (!authDirective) return fieldConfig;
      
      const { resolve = defaultFieldResolver } = fieldConfig;
      const requiredRole = authDirective.requires || 'USER';
      
      return {
        ...fieldConfig,
        resolve: async (source, args, context, info) => {
          if (!context.user) {
            throw new GraphQLError('Not authenticated', {
              extensions: { code: 'UNAUTHENTICATED' }
            });
          }
          
          const roles = ['USER', 'MODERATOR', 'ADMIN'];
          const userRoleIndex = roles.indexOf(context.user.role);
          const requiredRoleIndex = roles.indexOf(requiredRole);
          
          if (userRoleIndex < requiredRoleIndex) {
            throw new GraphQLError('Insufficient permissions', {
              extensions: { code: 'FORBIDDEN' }
            });
          }
          
          return resolve(source, args, context, info);
        }
      };
    }
  });
}

// @uppercase directive
function uppercaseDirectiveTransformer(schema) {
  return mapSchema(schema, {
    [MapperKind.OBJECT_FIELD]: (fieldConfig) => {
      const uppercaseDirective = getDirective(schema, fieldConfig, 'uppercase')?.[0];
      
      if (!uppercaseDirective) return fieldConfig;
      
      const { resolve = defaultFieldResolver } = fieldConfig;
      
      return {
        ...fieldConfig,
        resolve: async (...args) => {
          const result = await resolve(...args);
          return typeof result === 'string' ? result.toUpperCase() : result;
        }
      };
    }
  });
}

// Apply directives
let schema = makeExecutableSchema({ typeDefs, resolvers });
schema = authDirectiveTransformer(schema);
schema = uppercaseDirectiveTransformer(schema);
```

---

## Step 1182: N+1 Problem และ DataLoader

```javascript
// ===== N+1 Problem =====
// ปัญหา: query posts แล้วแต่ละ post ต้องดึง author แยกกัน

// Query:
// {
//   posts {
//     title
//     author { name }  <- N queries สำหรับ N posts!
//   }
// }

// ถ้ามี 10 posts = 11 queries (1 + 10)

// Naive resolver (มีปัญหา):
const badResolvers = {
  Post: {
    author: async (post, _, context) => {
      // ถูกเรียก 1 ครั้งต่อ post = N+1 queries!
      return context.db.users.findById(post.authorId);
    }
  }
};

// ===== DataLoader Solution =====
const DataLoader = require('dataloader'); // npm install dataloader

// สร้าง loader ที่ batch requests
function createUserLoader(db) {
  return new DataLoader(async (userIds) => {
    console.log(`Batch loading ${userIds.length} users:`, userIds);
    
    // Query ครั้งเดียวสำหรับทุก IDs
    const users = await db.users.findByIds(userIds);
    
    // Return ใน order เดียวกับ userIds
    return userIds.map(id => users.find(u => u.id === id) || null);
  });
}

function createPostLoader(db) {
  return new DataLoader(async (authorIds) => {
    const posts = await db.posts.findByAuthorIds(authorIds);
    
    return authorIds.map(authorId =>
      posts.filter(p => p.authorId === authorId)
    );
  });
}

// Context factory
const context = async ({ req }) => {
  return {
    user: await getUserFromToken(req.headers.authorization),
    db,
    loaders: {
      user: createUserLoader(db),     // สร้างใหม่ทุก request (per-request cache)
      postsByAuthor: createPostLoader(db)
    }
  };
};

// Resolver ที่ใช้ DataLoader
const optimizedResolvers = {
  Post: {
    author: async (post, _, context) => {
      // DataLoader batches ทุก calls ใน tick เดียวกัน
      return context.loaders.user.load(post.authorId);
    }
  },
  
  User: {
    posts: async (user, _, context) => {
      return context.loaders.postsByAuthor.load(user.id);
    }
  }
};

// ===== DataLoader features =====

// Caching (per-request)
const loader = new DataLoader(batchFn);
const result1 = await loader.load('1'); // db query
const result2 = await loader.load('1'); // from cache!

// Disable cache
const noCacheLoader = new DataLoader(batchFn, { cache: false });

// loadMany
const [user1, user2, user3] = await loader.loadMany(['1', '2', '3']);

// Prime (manually add to cache)
loader.prime('1', { id: '1', name: 'Alice' });

// Clear cache
loader.clear('1');
loader.clearAll();

// ===== ตัวอย่างจริง =====
class UserAPI {
  constructor(db) {
    this.db = db;
    this.loader = new DataLoader(this._batchGetUsers.bind(this));
  }
  
  async _batchGetUsers(ids) {
    const users = await this.db.query(
      'SELECT * FROM users WHERE id = ANY($1)',
      [ids]
    );
    
    const userMap = {};
    users.forEach(u => userMap[u.id] = u);
    
    return ids.map(id => userMap[id] || null);
  }
  
  async getById(id) {
    return this.loader.load(id);
  }
  
  async getMany(ids) {
    return this.loader.loadMany(ids);
  }
  
  async getByEmail(email) {
    // ไม่ใช้ loader เพราะ key ไม่ใช่ ID
    return this.db.query('SELECT * FROM users WHERE email = $1', [email]);
  }
}
```

---

## Step 1183: Authentication ใน GraphQL

```javascript
// ===== Authentication patterns =====

// Pattern 1: Context-based auth
const server = new ApolloServer({
  typeDefs,
  resolvers,
  context: async ({ req }) => {
    const token = req.headers.authorization?.split(' ')[1];
    let user = null;
    
    if (token) {
      try {
        const decoded = jwt.verify(token, process.env.JWT_SECRET);
        user = await getUserById(decoded.sub);
      } catch {}
    }
    
    return { user };
  }
});

// Resolver checks
const resolvers = {
  Query: {
    me: (_, __, context) => {
      if (!context.user) {
        throw new GraphQLError('Not authenticated', {
          extensions: { code: 'UNAUTHENTICATED' }
        });
      }
      return context.user;
    },
    
    adminUsers: (_, __, context) => {
      if (!context.user || context.user.role !== 'ADMIN') {
        throw new GraphQLError('Not authorized', {
          extensions: { code: 'FORBIDDEN' }
        });
      }
      return getAllUsers();
    }
  }
};

// Pattern 2: Shield middleware
const { shield, rule, and, or, not } = require('graphql-shield');
// npm install graphql-shield

const isAuthenticated = rule()((_, __, context) => {
  return context.user !== null;
});

const isAdmin = rule()((_, __, context) => {
  return context.user?.role === 'ADMIN';
});

const isModerator = rule()((_, __, context) => {
  return ['ADMIN', 'MODERATOR'].includes(context.user?.role);
});

const isPostAuthor = rule()(async (post, _, context) => {
  if (!context.user) return false;
  const fullPost = await getPostById(post.id);
  return fullPost.authorId === context.user.id;
});

const permissions = shield({
  Query: {
    me: isAuthenticated,
    users: and(isAuthenticated, isAdmin),
    posts: not(isAuthenticated), // public
  },
  Mutation: {
    createPost: isAuthenticated,
    updatePost: and(isAuthenticated, or(isPostAuthor, isAdmin)),
    deletePost: and(isAuthenticated, or(isPostAuthor, isAdmin)),
    manageUsers: and(isAuthenticated, isModerator)
  }
});

// Apply shield
const schemaWithPermissions = applyMiddleware(schema, permissions);
```

---

## Step 1184: Error Handling

```javascript
const { GraphQLError } = require('graphql');
const { ApolloServerErrorCode } = require('@apollo/server/errors');

// ===== Custom error codes =====
const ErrorCodes = {
  UNAUTHENTICATED: 'UNAUTHENTICATED',
  FORBIDDEN: 'FORBIDDEN',
  NOT_FOUND: 'NOT_FOUND',
  VALIDATION_ERROR: 'VALIDATION_ERROR',
  CONFLICT: 'CONFLICT',
  RATE_LIMITED: 'RATE_LIMITED',
  INTERNAL_ERROR: 'INTERNAL_ERROR'
};

// ===== Error classes =====
class AppError extends GraphQLError {
  constructor(message, code, extensions = {}) {
    super(message, {
      extensions: {
        code,
        ...extensions
      }
    });
    this.name = 'AppError';
  }
}

class NotFoundError extends AppError {
  constructor(resource, id) {
    super(
      `${resource} with id ${id} not found`,
      ErrorCodes.NOT_FOUND,
      { resource, id }
    );
  }
}

class ValidationError extends AppError {
  constructor(errors) {
    super(
      'Validation failed',
      ErrorCodes.VALIDATION_ERROR,
      { errors }
    );
  }
}

// ===== Resolvers กับ proper error handling =====
const resolvers = {
  Query: {
    user: async (_, { id }, context) => {
      try {
        const user = await context.db.users.findById(id);
        
        if (!user) {
          throw new NotFoundError('User', id);
        }
        
        return user;
      } catch (err) {
        if (err instanceof GraphQLError) throw err; // re-throw GraphQL errors
        
        // Log unexpected errors
        console.error('Unexpected error in user resolver:', err);
        
        throw new AppError(
          'Failed to fetch user',
          ErrorCodes.INTERNAL_ERROR
        );
      }
    }
  },
  
  Mutation: {
    createUser: async (_, { input }, context) => {
      // Validate input
      const errors = [];
      
      if (!input.email.includes('@')) {
        errors.push({ field: 'email', message: 'Invalid email' });
      }
      
      if (input.password.length < 8) {
        errors.push({ field: 'password', message: 'Password too short' });
      }
      
      if (errors.length > 0) {
        throw new ValidationError(errors);
      }
      
      // Check duplicate
      const existing = await context.db.users.findByEmail(input.email);
      if (existing) {
        throw new AppError(
          `Email ${input.email} is already registered`,
          ErrorCodes.CONFLICT,
          { field: 'email' }
        );
      }
      
      return context.db.users.create(input);
    }
  }
};

// ===== Error formatting =====
const server = new ApolloServer({
  typeDefs,
  resolvers,
  formatError: (formattedError, error) => {
    // ซ่อน internal errors ใน production
    if (
      process.env.NODE_ENV === 'production' &&
      formattedError.extensions?.code === ErrorCodes.INTERNAL_ERROR
    ) {
      return {
        message: 'Internal server error',
        extensions: { code: ErrorCodes.INTERNAL_ERROR }
      };
    }
    
    // Log all errors
    console.error('[GraphQL Error]', {
      message: formattedError.message,
      code: formattedError.extensions?.code,
      path: formattedError.path
    });
    
    return formattedError;
  }
});
```

---

## Step 1185: Code-first vs Schema-first

```javascript
// ===== Schema-first (SDL-first) =====
// เขียน schema ก่อนแล้วค่อยสร้าง resolvers

const typeDefs = `#graphql
  type User {
    id: ID!
    name: String!
    email: String!
  }
  
  type Query {
    user(id: ID!): User
    users: [User!]!
  }
`;

// ข้อดี: Schema ชัดเจน, อ่านง่าย, documentation เป็นส่วนหนึ่งของ code
// ข้อเสีย: ต้อง sync ระหว่าง schema และ resolvers, type safety ต่ำ

// ===== Code-first (Programmatic) =====
// สร้าง schema จาก code

const {
  GraphQLSchema,
  GraphQLObjectType,
  GraphQLString,
  GraphQLNonNull,
  GraphQLID,
  GraphQLList,
  GraphQLInt,
  GraphQLBoolean
} = require('graphql');

const UserType = new GraphQLObjectType({
  name: 'User',
  fields: () => ({
    id: { type: new GraphQLNonNull(GraphQLID) },
    name: { type: new GraphQLNonNull(GraphQLString) },
    email: { type: new GraphQLNonNull(GraphQLString) },
    posts: {
      type: new GraphQLList(PostType),
      resolve: (user, _, context) => context.db.posts.findByAuthorId(user.id)
    }
  })
});

const PostType = new GraphQLObjectType({
  name: 'Post',
  fields: () => ({
    id: { type: new GraphQLNonNull(GraphQLID) },
    title: { type: new GraphQLNonNull(GraphQLString) },
    author: {
      type: UserType,
      resolve: (post, _, context) => context.db.users.findById(post.authorId)
    }
  })
});

const QueryType = new GraphQLObjectType({
  name: 'Query',
  fields: {
    user: {
      type: UserType,
      args: { id: { type: new GraphQLNonNull(GraphQLID) } },
      resolve: (_, { id }, context) => context.db.users.findById(id)
    },
    users: {
      type: new GraphQLList(UserType),
      resolve: (_, __, context) => context.db.users.findAll()
    }
  }
});

const schema = new GraphQLSchema({ query: QueryType });

// ===== Code-first กับ TypeGraphQL (TypeScript) =====
// npm install type-graphql reflect-metadata
/*
@ObjectType()
class User {
  @Field(() => ID)
  id: string;

  @Field()
  name: string;

  @Field()
  email: string;

  @Field(() => [Post])
  posts: Post[];
}

@Resolver(User)
class UserResolver {
  @Query(() => [User])
  async users(@Ctx() context: Context) {
    return context.db.users.findAll();
  }
  
  @Query(() => User, { nullable: true })
  async user(@Arg('id', () => ID) id: string, @Ctx() context: Context) {
    return context.db.users.findById(id);
  }
  
  @Mutation(() => User)
  async createUser(@Arg('input') input: CreateUserInput) {
    return createUser(input);
  }
}
*/
```

---

## Step 1186: Apollo Client Overview

```javascript
// ===== Apollo Client (Frontend) =====
// npm install @apollo/client graphql

// ===== Setup (React) =====
import { ApolloClient, InMemoryCache, ApolloProvider, gql } from '@apollo/client';

const client = new ApolloClient({
  uri: 'http://localhost:4000/graphql',
  cache: new InMemoryCache(),
  headers: {
    authorization: localStorage.getItem('token') || ''
  }
});

// Wrap app
function App() {
  return (
    <ApolloProvider client={client}>
      <MyApp />
    </ApolloProvider>
  );
}

// ===== Queries =====
import { useQuery } from '@apollo/client';

const GET_POSTS = gql`
  query GetPosts($page: Int, $limit: Int) {
    posts(pagination: { page: $page, limit: $limit }) {
      nodes {
        id
        title
        author {
          name
        }
        createdAt
      }
      totalCount
    }
  }
`;

function PostList() {
  const { loading, error, data } = useQuery(GET_POSTS, {
    variables: { page: 1, limit: 10 }
  });
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  
  return (
    <ul>
      {data.posts.nodes.map(post => (
        <li key={post.id}>
          <h2>{post.title}</h2>
          <p>By {post.author.name}</p>
        </li>
      ))}
    </ul>
  );
}

// ===== Mutations =====
import { useMutation } from '@apollo/client';

const CREATE_POST = gql`
  mutation CreatePost($input: CreatePostInput!) {
    createPost(input: $input) {
      id
      title
      createdAt
    }
  }
`;

function CreatePostForm() {
  const [createPost, { loading, error }] = useMutation(CREATE_POST, {
    // Update cache after mutation
    update(cache, { data: { createPost } }) {
      cache.modify({
        fields: {
          posts(existing = { nodes: [] }) {
            const newPostRef = cache.writeFragment({
              data: createPost,
              fragment: gql`
                fragment NewPost on Post {
                  id
                  title
                }
              `
            });
            return {
              ...existing,
              nodes: [...existing.nodes, newPostRef]
            };
          }
        }
      });
    }
  });
  
  const handleSubmit = async (formData) => {
    try {
      const { data } = await createPost({
        variables: { input: formData }
      });
      console.log('Created:', data.createPost);
    } catch (err) {
      console.error('Error:', err);
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      {/* form fields */}
      <button type="submit" disabled={loading}>
        {loading ? 'Creating...' : 'Create Post'}
      </button>
    </form>
  );
}

// ===== Subscriptions =====
import { useSubscription, split } from '@apollo/client';
import { GraphQLWsLink } from '@apollo/client/link/subscriptions';
import { createClient } from 'graphql-ws';
import { getMainDefinition } from '@apollo/client/utilities';

const wsLink = new GraphQLWsLink(createClient({
  url: 'ws://localhost:4000/graphql',
  connectionParams: {
    authorization: localStorage.getItem('token') || ''
  }
}));

// Split traffic: queries/mutations go HTTP, subscriptions go WS
const splitLink = split(
  ({ query }) => {
    const def = getMainDefinition(query);
    return def.kind === 'OperationDefinition' && def.operation === 'subscription';
  },
  wsLink,
  httpLink
);

const client = new ApolloClient({
  link: splitLink,
  cache: new InMemoryCache()
});

// ใช้งาน subscription
const POST_ADDED = gql`
  subscription {
    postAdded {
      id
      title
      author { name }
    }
  }
`;

function LivePostFeed() {
  const { data } = useSubscription(POST_ADDED);
  
  return data ? (
    <div>New post: {data.postAdded.title}</div>
  ) : null;
}
```

---

## Step 1187: urql - GraphQL Client

```javascript
// urql - lightweight GraphQL client
// npm install urql graphql

// ===== Setup =====
import { createClient, Provider } from 'urql';

const client = createClient({
  url: 'http://localhost:4000/graphql',
  fetchOptions: {
    headers: {
      Authorization: `Bearer ${getToken()}`
    }
  }
});

function App() {
  return (
    <Provider value={client}>
      <MyApp />
    </Provider>
  );
}

// ===== Query =====
import { useQuery, gql } from 'urql';

const GET_USERS = gql`
  query {
    users {
      id
      name
      email
    }
  }
`;

function UserList() {
  const [result] = useQuery({ query: GET_USERS });
  const { data, fetching, error } = result;
  
  if (fetching) return <p>Loading...</p>;
  if (error) return <p>Error: {error.message}</p>;
  
  return (
    <ul>
      {data.users.map(user => (
        <li key={user.id}>{user.name}</li>
      ))}
    </ul>
  );
}

// ===== Mutation =====
import { useMutation } from 'urql';

const CREATE_USER = gql`
  mutation CreateUser($name: String!, $email: String!) {
    createUser(name: $name, email: $email) {
      id
      name
    }
  }
`;

function CreateUser() {
  const [result, createUser] = useMutation(CREATE_USER);
  
  const handleCreate = () => {
    createUser({ name: 'Alice', email: 'alice@example.com' });
  };
  
  return (
    <button onClick={handleCreate} disabled={result.fetching}>
      Create User
    </button>
  );
}
```

---

## Step 1188: GraphQL Best Practices

```javascript
// ===== 1. Schema Design =====
// - ใช้ descriptive names
// - Non-null (!) สำหรับ required fields
// - ใช้ Input types สำหรับ mutations
// - ใช้ Connection pattern สำหรับ pagination

// GOOD: Connection pattern
type PostConnection {
  edges: [PostEdge!]!
  pageInfo: PageInfo!
  totalCount: Int!
}

type PostEdge {
  node: Post!
  cursor: String!
}

// ===== 2. Resolver separation =====
// แยก resolver logic ออกจาก data access

// BAD:
const resolvers = {
  Query: {
    users: async (_, __, { db }) => {
      return db.query('SELECT * FROM users WHERE deleted_at IS NULL');
    }
  }
};

// GOOD:
class UserRepository {
  constructor(db) { this.db = db; }
  
  async findAll() {
    return this.db.query('SELECT * FROM users WHERE deleted_at IS NULL');
  }
  
  async findById(id) {
    return this.db.query('SELECT * FROM users WHERE id = $1', [id]);
  }
}

const resolvers = {
  Query: {
    users: (_, __, context) => context.repositories.user.findAll()
  }
};

// ===== 3. Batching with DataLoader =====
// ใช้ DataLoader เสมอสำหรับ related data

// ===== 4. Query depth limiting =====
const depthLimit = require('graphql-depth-limit');

const server = new ApolloServer({
  typeDefs,
  resolvers,
  validationRules: [depthLimit(10)] // จำกัด depth ที่ 10
});

// ===== 5. Query complexity =====
const { createComplexityLimitRule } = require('graphql-validation-complexity');

const server = new ApolloServer({
  validationRules: [
    createComplexityLimitRule(1000, {
      onCost: (cost) => console.log(`Query cost: ${cost}`)
    })
  ]
});

// ===== 6. Persisted queries =====
// เก็บ query โดย hash เพื่อลด bandwidth
// Apollo Persisted Queries

// ===== 7. Caching =====
const { InMemoryLRUCache } = require('@apollo/utils.keyvaluecache');
const responseCachePlugin = require('@apollo/server-plugin-response-cache');

const server = new ApolloServer({
  cache: new InMemoryLRUCache({ maxSize: Math.pow(2, 20) * 30 }),
  plugins: [responseCachePlugin()]
});

// กำหนด cache hint ใน schema
type Query {
  products: [Product] @cacheControl(maxAge: 30)
  categories: [Category] @cacheControl(maxAge: 3600)
}
```

---

## Step 1189: Complete GraphQL Application

```javascript
// ===== Complete Todo GraphQL API =====

const { ApolloServer } = require('@apollo/server');
const { startStandaloneServer } = require('@apollo/server/standalone');
const DataLoader = require('dataloader');

// Schema
const typeDefs = `#graphql
  enum Priority {
    LOW
    MEDIUM
    HIGH
  }

  enum Status {
    TODO
    IN_PROGRESS
    DONE
  }

  type User {
    id: ID!
    name: String!
    email: String!
    todos: [Todo!]!
    todoCount: Int!
  }

  type Todo {
    id: ID!
    title: String!
    description: String
    status: Status!
    priority: Priority!
    dueDate: String
    owner: User!
    tags: [String!]!
    createdAt: String!
    updatedAt: String!
  }

  input CreateTodoInput {
    title: String!
    description: String
    priority: Priority = MEDIUM
    dueDate: String
    tags: [String!] = []
  }

  input UpdateTodoInput {
    title: String
    description: String
    status: Status
    priority: Priority
    dueDate: String
    tags: [String!]
  }

  input TodoFilterInput {
    status: Status
    priority: Priority
    tag: String
    search: String
  }

  type TodoConnection {
    nodes: [Todo!]!
    totalCount: Int!
  }

  type AuthPayload {
    token: String!
    user: User!
  }

  type Query {
    me: User
    todos(filter: TodoFilterInput): TodoConnection!
    todo(id: ID!): Todo
    users: [User!]!
  }

  type Mutation {
    register(name: String!, email: String!, password: String!): AuthPayload!
    login(email: String!, password: String!): AuthPayload!
    
    createTodo(input: CreateTodoInput!): Todo!
    updateTodo(id: ID!, input: UpdateTodoInput!): Todo!
    deleteTodo(id: ID!): Boolean!
    completeTodo(id: ID!): Todo!
  }
`;

// In-memory stores
let users = [
  { id: '1', name: 'Alice', email: 'alice@example.com', passwordHash: 'hashed', createdAt: new Date().toISOString() }
];

let todos = [
  {
    id: '1', title: 'Learn GraphQL', description: 'Build a full GraphQL API',
    status: 'IN_PROGRESS', priority: 'HIGH', ownerId: '1',
    tags: ['learning', 'backend'], dueDate: '2024-12-31',
    createdAt: new Date().toISOString(), updatedAt: new Date().toISOString()
  },
  {
    id: '2', title: 'Setup CI/CD', description: null,
    status: 'TODO', priority: 'MEDIUM', ownerId: '1',
    tags: ['devops'], dueDate: null,
    createdAt: new Date().toISOString(), updatedAt: new Date().toISOString()
  }
];

let nextUserId = 2;
let nextTodoId = 3;

// DataLoaders
const createLoaders = () => ({
  user: new DataLoader(async (ids) => {
    const userMap = Object.fromEntries(users.map(u => [u.id, u]));
    return ids.map(id => userMap[id] || null);
  }),
  
  todosByUser: new DataLoader(async (userIds) => {
    return userIds.map(userId => todos.filter(t => t.ownerId === userId));
  })
});

// Resolvers
const resolvers = {
  Query: {
    me: (_, __, context) => {
      if (!context.user) throw new GraphQLError('Not authenticated');
      return users.find(u => u.id === context.user.id);
    },
    
    todos: (_, { filter = {} }, context) => {
      if (!context.user) throw new GraphQLError('Not authenticated');
      
      let result = todos.filter(t => t.ownerId === context.user.id);
      
      if (filter.status) result = result.filter(t => t.status === filter.status);
      if (filter.priority) result = result.filter(t => t.priority === filter.priority);
      if (filter.tag) result = result.filter(t => t.tags.includes(filter.tag));
      if (filter.search) result = result.filter(t =>
        t.title.toLowerCase().includes(filter.search.toLowerCase())
      );
      
      return { nodes: result, totalCount: result.length };
    },
    
    todo: (_, { id }, context) => {
      if (!context.user) throw new GraphQLError('Not authenticated');
      const todo = todos.find(t => t.id === id);
      if (!todo) throw new GraphQLError('Todo not found');
      if (todo.ownerId !== context.user.id) throw new GraphQLError('Forbidden');
      return todo;
    },
    
    users: () => users
  },
  
  Mutation: {
    register: async (_, { name, email, password }) => {
      if (users.find(u => u.email === email)) {
        throw new GraphQLError('Email already registered');
      }
      
      const user = {
        id: String(nextUserId++),
        name, email,
        passwordHash: await require('bcrypt').hash(password, 10),
        createdAt: new Date().toISOString()
      };
      
      users.push(user);
      const token = require('jsonwebtoken').sign({ sub: user.id }, 'secret', { expiresIn: '7d' });
      
      return { token, user };
    },
    
    login: async (_, { email, password }) => {
      const user = users.find(u => u.email === email);
      const valid = user && await require('bcrypt').compare(password, user.passwordHash);
      
      if (!valid) throw new GraphQLError('Invalid credentials');
      
      const token = require('jsonwebtoken').sign({ sub: user.id }, 'secret', { expiresIn: '7d' });
      return { token, user };
    },
    
    createTodo: (_, { input }, context) => {
      if (!context.user) throw new GraphQLError('Not authenticated');
      
      const todo = {
        id: String(nextTodoId++),
        ...input,
        status: 'TODO',
        ownerId: context.user.id,
        tags: input.tags || [],
        createdAt: new Date().toISOString(),
        updatedAt: new Date().toISOString()
      };
      
      todos.push(todo);
      return todo;
    },
    
    updateTodo: (_, { id, input }, context) => {
      if (!context.user) throw new GraphQLError('Not authenticated');
      
      const index = todos.findIndex(t => t.id === id);
      if (index === -1) throw new GraphQLError('Todo not found');
      if (todos[index].ownerId !== context.user.id) throw new GraphQLError('Forbidden');
      
      todos[index] = {
        ...todos[index],
        ...input,
        updatedAt: new Date().toISOString()
      };
      
      return todos[index];
    },
    
    deleteTodo: (_, { id }, context) => {
      if (!context.user) throw new GraphQLError('Not authenticated');
      
      const index = todos.findIndex(t => t.id === id);
      if (index === -1) throw new GraphQLError('Todo not found');
      if (todos[index].ownerId !== context.user.id) throw new GraphQLError('Forbidden');
      
      todos.splice(index, 1);
      return true;
    },
    
    completeTodo: (_, { id }, context) => {
      if (!context.user) throw new GraphQLError('Not authenticated');
      
      const todo = todos.find(t => t.id === id && t.ownerId === context.user.id);
      if (!todo) throw new GraphQLError('Todo not found');
      
      todo.status = 'DONE';
      todo.updatedAt = new Date().toISOString();
      return todo;
    }
  },
  
  Todo: {
    owner: (todo, _, context) => context.loaders.user.load(todo.ownerId)
  },
  
  User: {
    todos: (user, _, context) => context.loaders.todosByUser.load(user.id),
    todoCount: async (user, _, context) => {
      const userTodos = await context.loaders.todosByUser.load(user.id);
      return userTodos.length;
    }
  }
};

const { GraphQLError } = require('graphql');

const server = new ApolloServer({ typeDefs, resolvers });

startStandaloneServer(server, {
  listen: { port: 4000 },
  context: async ({ req }) => {
    const token = req.headers.authorization?.split(' ')[1];
    let user = null;
    
    if (token) {
      try {
        const decoded = require('jsonwebtoken').verify(token, 'secret');
        user = users.find(u => u.id === decoded.sub);
      } catch {}
    }
    
    return { user, loaders: createLoaders() };
  }
}).then(({ url }) => console.log(`Server ready at ${url}`));
```

---

## Step 1190: สรุปและ Best Practices

```javascript
// ===== GraphQL Checklist =====

/*
Schema Design:
✅ ใช้ descriptive, consistent naming
✅ Non-null fields สำหรับ required values
✅ Input types สำหรับทุก mutations
✅ Connection pattern สำหรับ pagination
✅ Union types สำหรับ polymorphic data
✅ Custom scalars (Date, JSON, URL)

Performance:
✅ DataLoader สำหรับทุก N+1 queries
✅ Depth limiting (graphql-depth-limit)
✅ Query complexity analysis
✅ Persisted queries ใน production
✅ Response caching ด้วย @cacheControl

Security:
✅ Authentication ใน context
✅ Authorization ใน resolvers/middleware
✅ Input validation
✅ Rate limiting
✅ Disable introspection ใน production (optional)

Development:
✅ GraphQL Playground ใน development
✅ Generated TypeScript types
✅ Integration tests
✅ Schema documentation
*/

// ===== Disable introspection ใน production =====
const { ApolloServerPluginLandingPageDisabled } = require('@apollo/server/plugin/disabled');
const { NoIntrospection } = require('graphql-disable-introspection');

const server = new ApolloServer({
  typeDefs,
  resolvers,
  plugins: [
    process.env.NODE_ENV === 'production'
      ? ApolloServerPluginLandingPageDisabled()
      : undefined
  ].filter(Boolean),
  validationRules: process.env.NODE_ENV === 'production'
    ? [NoIntrospection]
    : []
});

// ===== Schema stitching =====
// รวม multiple schemas เป็น single schema
const { stitchSchemas } = require('@graphql-tools/stitch');

const gatewaySchema = stitchSchemas({
  subschemas: [
    { schema: userSchema },
    { schema: postSchema },
    { schema: analyticsSchema }
  ]
});

// ===== Federation =====
// Microservices approach กับ Apollo Federation
// แต่ละ service เป็น subgraph
// Apollo Gateway รวมทุก subgraph
```

---

## สรุป Steps 1171-1190

| Step | หัวข้อ |
|------|--------|
| 1171 | GraphQL คืออะไรและทำไมต้องใช้ |
| 1172 | Schema Definition Language (SDL) |
| 1173 | Setting up Apollo Server |
| 1174 | GraphQL กับ Express |
| 1175 | Queries |
| 1176 | Mutations |
| 1177 | Subscriptions |
| 1178 | Resolvers ขั้นสูง |
| 1179 | Query Variables |
| 1180 | Fragments |
| 1181 | Directives |
| 1182 | N+1 Problem และ DataLoader |
| 1183 | Authentication |
| 1184 | Error Handling |
| 1185 | Code-first vs Schema-first |
| 1186 | Apollo Client |
| 1187 | urql |
| 1188 | Best Practices |
| 1189 | Complete GraphQL App |
| 1190 | สรุปและ Checklist |

---

## แบบฝึกหัดท้ายบท

### แบบฝึกหัดที่ 1: Blog GraphQL API
สร้าง GraphQL API สำหรับ blog ที่มี:
1. Schema: User, Post, Comment, Tag
2. Queries: posts พร้อม filtering/pagination, post detail, user profile
3. Mutations: CRUD posts, comments, user management
4. Subscriptions: real-time comments
5. DataLoader สำหรับป้องกัน N+1

```graphql
# Starter schema
type Query {
  posts(filter: PostFilter, pagination: PaginationInput): PostConnection!
  post(id: ID!): Post
  me: User
}

type Mutation {
  createPost(input: CreatePostInput!): Post!
  # เพิ่ม mutations อื่นๆ
}

type Subscription {
  commentAdded(postId: ID!): Comment!
}
```

### แบบฝึกหัดที่ 2: Task Manager
สร้าง task management GraphQL API:
1. Projects: สร้าง, invite members
2. Tasks: CRUD, assign, move between stages
3. Comments บน tasks
4. Real-time updates ด้วย subscriptions
5. Authorization: owner, member, viewer roles

### แบบฝึกหัดที่ 3: Chat Application
สร้าง chat app:
1. Rooms: create, join, list
2. Messages: send, delete, paginate (cursor-based)
3. Subscriptions: real-time messages, typing indicators
4. Online status
5. Message reactions

### แบบฝึกหัดที่ 4: E-Commerce GraphQL
แปลง REST E-Commerce API เป็น GraphQL:
1. Products + inventory
2. Cart management
3. Order lifecycle
4. User reviews
5. Search ด้วย Union types

### แบบฝึกหัดที่ 5: Comparison Project
สร้าง API เดียวกันด้วยทั้ง REST และ GraphQL แล้วเปรียบเทียบ:
1. ความซับซ้อนของ code
2. Performance (number of queries, response size)
3. Developer experience
4. Caching strategies

---

*จบ Part 60: GraphQL - ยินดีด้วยที่เรียนจบ Node.js, Express, RESTful API Design และ GraphQL แล้ว! คุณพร้อมสร้าง backend APIs แบบ professional แล้ว*

---

## ภาคผนวก: เปรียบเทียบ REST vs GraphQL

```javascript
// ===== ตัวอย่าง Request Comparison =====

// Scenario: หน้า Profile ต้องการ user info + latest 3 posts + total stats

// === REST ===
// 1. GET /api/users/123
//    Response: { id, name, email, bio, avatar, followerCount, createdAt }

// 2. GET /api/users/123/posts?limit=3&sort=createdAt
//    Response: [{ id, title, createdAt }]

// 3. GET /api/users/123/stats
//    Response: { totalPosts, totalComments, totalLikes }

// = 3 requests, หลาย fields ที่ไม่ได้ใช้

// === GraphQL ===
// 1 request:
/*
query UserProfile($userId: ID!) {
  user(id: $userId) {
    name
    bio
    avatar
    posts(limit: 3, sort: CREATED_DESC) {
      id
      title
      createdAt
    }
    stats {
      totalPosts
      totalComments
    }
  }
}
*/

// = 1 request, เฉพาะ fields ที่ต้องการ

// ===== เมื่อไหร่ใช้อะไร =====
const comparison = {
  useREST: [
    'Simple CRUD APIs',
    'File uploads',
    'HTTP caching สำคัญ',
    'Team ไม่คุ้น GraphQL',
    'Simple client (mobile apps)',
    'Public APIs'
  ],
  useGraphQL: [
    'หลาย clients ต้องการข้อมูลต่างกัน',
    'Complex data relationships',
    'Rapid iteration',
    'Real-time requirements',
    'Microservices aggregation',
    'Mobile apps ที่ bandwidth จำกัด'
  ]
};
```
