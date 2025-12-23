---
title: "Section 2: Backend Development Workflows"
parent: "Part 5: Specialized Workflows & Domains"
nav_order: 2
---

# Section 2: Backend Development Workflows

**Part of:** [Part 5: Specialized Workflows & Domains](README.md)  
**Related:** [Core Workflows](../01-fundamentals-core-concepts/03-core-workflows.md) | [Custom Agents](../03-advanced-chat-agents/02-custom-agents-instructions.md)

---

## 📋 Overview

This section demonstrates how to leverage GitHub Copilot for backend development across various frameworks and patterns. From API endpoint generation to database design, ORM queries, authentication systems, and error handling, Copilot accelerates server-side development significantly.

**What you'll learn:**
- REST API and GraphQL endpoint generation
- Database schema design and migrations
- ORM query patterns (Prisma, TypeORM, Sequelize, SQLAlchemy)
- Authentication and authorization implementation
- Middleware and error handling
- API documentation generation (OpenAPI/Swagger)
- Testing backend endpoints

**Time to master:** 50-60 minutes

---

## 🎯 Key Takeaways

1. **API scaffolding:** Copilot generates complete CRUD endpoints with validation
2. **Database design:** Auto-generate schemas, migrations, and seed data
3. **ORM mastery:** Copilot understands Prisma, TypeORM, Sequelize patterns
4. **Security built-in:** Generate auth middleware, JWT handling, input sanitization
5. **Documentation automation:** Auto-generate OpenAPI specs from code
6. **Testing shortcuts:** Generate integration tests for all endpoints

---

## 🚀 Quick Wins: Backend Development

### Pattern 1: Express REST API Endpoint

**Generate complete CRUD endpoints:**

```
Chat → Agent mode

"Create Express REST API for a blog with:
- Posts (title, content, author, createdAt)
- CRUD operations
- Input validation with express-validator
- Error handling middleware
- TypeScript types
- OpenAPI documentation comments"
```

**Result:**

```typescript
// types/post.ts
export interface Post {
  id: string;
  title: string;
  content: string;
  author: string;
  createdAt: Date;
  updatedAt: Date;
}

export interface CreatePostDto {
  title: string;
  content: string;
  author: string;
}

export interface UpdatePostDto {
  title?: string;
  content?: string;
}

// middleware/errorHandler.ts
import { Request, Response, NextFunction } from 'express';

export class AppError extends Error {
  statusCode: number;
  isOperational: boolean;

  constructor(message: string, statusCode: number) {
    super(message);
    this.statusCode = statusCode;
    this.isOperational = true;
    Error.captureStackTrace(this, this.constructor);
  }
}

export const errorHandler = (
  err: Error | AppError,
  req: Request,
  res: Response,
  next: NextFunction
) => {
  if (err instanceof AppError) {
    return res.status(err.statusCode).json({
      status: 'error',
      message: err.message
    });
  }

  console.error('ERROR 💥', err);
  res.status(500).json({
    status: 'error',
    message: 'Something went wrong!'
  });
};

// routes/posts.ts
import { Router } from 'express';
import { body, param, validationResult } from 'express-validator';
import { PostController } from '../controllers/postController';
import { AppError } from '../middleware/errorHandler';

const router = Router();
const postController = new PostController();

/**
 * @swagger
 * /api/posts:
 *   get:
 *     summary: Get all posts
 *     tags: [Posts]
 *     responses:
 *       200:
 *         description: List of posts
 */
router.get('/', postController.getAllPosts);

/**
 * @swagger
 * /api/posts/{id}:
 *   get:
 *     summary: Get post by ID
 *     tags: [Posts]
 *     parameters:
 *       - in: path
 *         name: id
 *         required: true
 *         schema:
 *           type: string
 */
router.get(
  '/:id',
  param('id').isUUID(),
  postController.getPostById
);

/**
 * @swagger
 * /api/posts:
 *   post:
 *     summary: Create new post
 *     tags: [Posts]
 *     requestBody:
 *       required: true
 *       content:
 *         application/json:
 *           schema:
 *             type: object
 *             properties:
 *               title:
 *                 type: string
 *               content:
 *                 type: string
 *               author:
 *                 type: string
 */
router.post(
  '/',
  [
    body('title').trim().isLength({ min: 3, max: 200 }),
    body('content').trim().isLength({ min: 10 }),
    body('author').trim().notEmpty()
  ],
  postController.createPost
);

/**
 * @swagger
 * /api/posts/{id}:
 *   put:
 *     summary: Update post
 *     tags: [Posts]
 */
router.put(
  '/:id',
  [
    param('id').isUUID(),
    body('title').optional().trim().isLength({ min: 3, max: 200 }),
    body('content').optional().trim().isLength({ min: 10 })
  ],
  postController.updatePost
);

/**
 * @swagger
 * /api/posts/{id}:
 *   delete:
 *     summary: Delete post
 *     tags: [Posts]
 */
router.delete(
  '/:id',
  param('id').isUUID(),
  postController.deletePost
);

export default router;

// controllers/postController.ts
import { Request, Response, NextFunction } from 'express';
import { validationResult } from 'express-validator';
import { PostService } from '../services/postService';
import { AppError } from '../middleware/errorHandler';

export class PostController {
  private postService = new PostService();

  getAllPosts = async (req: Request, res: Response, next: NextFunction) => {
    try {
      const posts = await this.postService.findAll();
      res.json({
        status: 'success',
        data: { posts }
      });
    } catch (error) {
      next(error);
    }
  };

  getPostById = async (req: Request, res: Response, next: NextFunction) => {
    try {
      const errors = validationResult(req);
      if (!errors.isEmpty()) {
        throw new AppError('Invalid post ID', 400);
      }

      const post = await this.postService.findById(req.params.id);
      
      if (!post) {
        throw new AppError('Post not found', 404);
      }

      res.json({
        status: 'success',
        data: { post }
      });
    } catch (error) {
      next(error);
    }
  };

  createPost = async (req: Request, res: Response, next: NextFunction) => {
    try {
      const errors = validationResult(req);
      if (!errors.isEmpty()) {
        return res.status(400).json({
          status: 'error',
          errors: errors.array()
        });
      }

      const post = await this.postService.create(req.body);
      
      res.status(201).json({
        status: 'success',
        data: { post }
      });
    } catch (error) {
      next(error);
    }
  };

  updatePost = async (req: Request, res: Response, next: NextFunction) => {
    try {
      const errors = validationResult(req);
      if (!errors.isEmpty()) {
        return res.status(400).json({
          status: 'error',
          errors: errors.array()
        });
      }

      const post = await this.postService.update(req.params.id, req.body);
      
      if (!post) {
        throw new AppError('Post not found', 404);
      }

      res.json({
        status: 'success',
        data: { post }
      });
    } catch (error) {
      next(error);
    }
  };

  deletePost = async (req: Request, res: Response, next: NextFunction) => {
    try {
      const errors = validationResult(req);
      if (!errors.isEmpty()) {
        throw new AppError('Invalid post ID', 400);
      }

      await this.postService.delete(req.params.id);
      
      res.status(204).send();
    } catch (error) {
      next(error);
    }
  };
}
```

---

### Pattern 2: Prisma Schema & Queries

**Generate Prisma schema:**

```
Chat:

"Create Prisma schema for an e-commerce platform with:
- Users (email, password, role)
- Products (name, description, price, stock)
- Orders (user, items, status, total)
- OrderItems (product, quantity, price)
Include relations and indexes"
```

**Result:**

```prisma
// prisma/schema.prisma
generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

enum UserRole {
  ADMIN
  USER
  GUEST
}

enum OrderStatus {
  PENDING
  PROCESSING
  SHIPPED
  DELIVERED
  CANCELLED
}

model User {
  id        String   @id @default(uuid())
  email     String   @unique
  password  String
  firstName String
  lastName  String
  role      UserRole @default(USER)
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
  
  orders    Order[]
  
  @@index([email])
  @@map("users")
}

model Product {
  id          String   @id @default(uuid())
  name        String
  description String?  @db.Text
  price       Decimal  @db.Decimal(10, 2)
  stock       Int      @default(0)
  imageUrl    String?
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
  
  orderItems  OrderItem[]
  
  @@index([name])
  @@map("products")
}

model Order {
  id        String      @id @default(uuid())
  userId    String
  total     Decimal     @db.Decimal(10, 2)
  status    OrderStatus @default(PENDING)
  createdAt DateTime    @default(now())
  updatedAt DateTime    @updatedAt
  
  user      User        @relation(fields: [userId], references: [id], onDelete: Cascade)
  items     OrderItem[]
  
  @@index([userId])
  @@index([status])
  @@map("orders")
}

model OrderItem {
  id        String  @id @default(uuid())
  orderId   String
  productId String
  quantity  Int
  price     Decimal @db.Decimal(10, 2)
  
  order     Order   @relation(fields: [orderId], references: [id], onDelete: Cascade)
  product   Product @relation(fields: [productId], references: [id])
  
  @@unique([orderId, productId])
  @@index([orderId])
  @@index([productId])
  @@map("order_items")
}
```

**Generate Prisma service layer:**

```
Chat:

"Create a Prisma service for Order management with:
- Create order with items
- Get order with user and products
- Update order status
- Calculate order total
- Get user order history
Include TypeScript types and error handling"
```

**Result:**

```typescript
// services/orderService.ts
import { PrismaClient, Order, OrderStatus, Prisma } from '@prisma/client';

const prisma = new PrismaClient();

export interface CreateOrderDto {
  userId: string;
  items: {
    productId: string;
    quantity: number;
  }[];
}

export interface OrderWithDetails extends Order {
  user: {
    id: string;
    email: string;
    firstName: string;
    lastName: string;
  };
  items: {
    id: string;
    quantity: number;
    price: Prisma.Decimal;
    product: {
      id: string;
      name: string;
      imageUrl: string | null;
    };
  }[];
}

export class OrderService {
  async createOrder(dto: CreateOrderDto): Promise<OrderWithDetails> {
    // Fetch products to calculate prices
    const products = await prisma.product.findMany({
      where: {
        id: { in: dto.items.map(item => item.productId) }
      }
    });

    // Validate stock availability
    for (const item of dto.items) {
      const product = products.find(p => p.id === item.productId);
      if (!product) {
        throw new Error(`Product ${item.productId} not found`);
      }
      if (product.stock < item.quantity) {
        throw new Error(`Insufficient stock for product ${product.name}`);
      }
    }

    // Calculate total
    const total = dto.items.reduce((sum, item) => {
      const product = products.find(p => p.id === item.productId)!;
      return sum + Number(product.price) * item.quantity;
    }, 0);

    // Create order with items in a transaction
    const order = await prisma.$transaction(async (tx) => {
      // Create order
      const newOrder = await tx.order.create({
        data: {
          userId: dto.userId,
          total: new Prisma.Decimal(total),
          status: OrderStatus.PENDING,
          items: {
            create: dto.items.map(item => {
              const product = products.find(p => p.id === item.productId)!;
              return {
                productId: item.productId,
                quantity: item.quantity,
                price: product.price
              };
            })
          }
        },
        include: {
          user: {
            select: {
              id: true,
              email: true,
              firstName: true,
              lastName: true
            }
          },
          items: {
            include: {
              product: {
                select: {
                  id: true,
                  name: true,
                  imageUrl: true
                }
              }
            }
          }
        }
      });

      // Update product stock
      for (const item of dto.items) {
        await tx.product.update({
          where: { id: item.productId },
          data: {
            stock: { decrement: item.quantity }
          }
        });
      }

      return newOrder;
    });

    return order;
  }

  async getOrderById(orderId: string): Promise<OrderWithDetails | null> {
    return prisma.order.findUnique({
      where: { id: orderId },
      include: {
        user: {
          select: {
            id: true,
            email: true,
            firstName: true,
            lastName: true
          }
        },
        items: {
          include: {
            product: {
              select: {
                id: true,
                name: true,
                imageUrl: true
              }
            }
          }
        }
      }
    });
  }

  async updateOrderStatus(orderId: string, status: OrderStatus): Promise<Order> {
    return prisma.order.update({
      where: { id: orderId },
      data: { status }
    });
  }

  async getUserOrders(userId: string): Promise<OrderWithDetails[]> {
    return prisma.order.findMany({
      where: { userId },
      include: {
        user: {
          select: {
            id: true,
            email: true,
            firstName: true,
            lastName: true
          }
        },
        items: {
          include: {
            product: {
              select: {
                id: true,
                name: true,
                imageUrl: true
              }
            }
          }
        }
      },
      orderBy: { createdAt: 'desc' }
    });
  }

  async getOrdersByStatus(status: OrderStatus): Promise<OrderWithDetails[]> {
    return prisma.order.findMany({
      where: { status },
      include: {
        user: {
          select: {
            id: true,
            email: true,
            firstName: true,
            lastName: true
          }
        },
        items: {
          include: {
            product: {
              select: {
                id: true,
                name: true,
                imageUrl: true
              }
            }
          }
        }
      },
      orderBy: { createdAt: 'desc' }
    });
  }

  async cancelOrder(orderId: string): Promise<Order> {
    return prisma.$transaction(async (tx) => {
      // Get order with items
      const order = await tx.order.findUnique({
        where: { id: orderId },
        include: { items: true }
      });

      if (!order) {
        throw new Error('Order not found');
      }

      if (order.status !== OrderStatus.PENDING) {
        throw new Error('Only pending orders can be cancelled');
      }

      // Restore product stock
      for (const item of order.items) {
        await tx.product.update({
          where: { id: item.productId },
          data: {
            stock: { increment: item.quantity }
          }
        });
      }

      // Update order status
      return tx.order.update({
        where: { id: orderId },
        data: { status: OrderStatus.CANCELLED }
      });
    });
  }
}
```

---

### Pattern 3: Authentication & Authorization

**Generate JWT authentication system:**

```
Chat:

"Create JWT authentication system with:
- User registration with bcrypt password hashing
- Login with JWT token generation
- Auth middleware to protect routes
- Refresh token mechanism
- Password reset flow
TypeScript and Express"
```

**Result:**

```typescript
// utils/jwt.ts
import jwt from 'jsonwebtoken';

const JWT_SECRET = process.env.JWT_SECRET || 'your-secret-key';
const JWT_REFRESH_SECRET = process.env.JWT_REFRESH_SECRET || 'your-refresh-secret';
const JWT_EXPIRES_IN = '15m';
const JWT_REFRESH_EXPIRES_IN = '7d';

export interface TokenPayload {
  userId: string;
  email: string;
  role: string;
}

export const generateAccessToken = (payload: TokenPayload): string => {
  return jwt.sign(payload, JWT_SECRET, { expiresIn: JWT_EXPIRES_IN });
};

export const generateRefreshToken = (payload: TokenPayload): string => {
  return jwt.sign(payload, JWT_REFRESH_SECRET, { expiresIn: JWT_REFRESH_EXPIRES_IN });
};

export const verifyAccessToken = (token: string): TokenPayload => {
  return jwt.verify(token, JWT_SECRET) as TokenPayload;
};

export const verifyRefreshToken = (token: string): TokenPayload => {
  return jwt.verify(token, JWT_REFRESH_SECRET) as TokenPayload;
};

// services/authService.ts
import bcrypt from 'bcryptjs';
import { PrismaClient, User } from '@prisma/client';
import { generateAccessToken, generateRefreshToken, TokenPayload } from '../utils/jwt';

const prisma = new PrismaClient();

export interface RegisterDto {
  email: string;
  password: string;
  firstName: string;
  lastName: string;
}

export interface LoginDto {
  email: string;
  password: string;
}

export interface AuthResponse {
  user: Omit<User, 'password'>;
  accessToken: string;
  refreshToken: string;
}

export class AuthService {
  async register(dto: RegisterDto): Promise<AuthResponse> {
    // Check if user exists
    const existingUser = await prisma.user.findUnique({
      where: { email: dto.email }
    });

    if (existingUser) {
      throw new Error('User with this email already exists');
    }

    // Hash password
    const hashedPassword = await bcrypt.hash(dto.password, 12);

    // Create user
    const user = await prisma.user.create({
      data: {
        email: dto.email,
        password: hashedPassword,
        firstName: dto.firstName,
        lastName: dto.lastName
      }
    });

    // Generate tokens
    const tokenPayload: TokenPayload = {
      userId: user.id,
      email: user.email,
      role: user.role
    };

    const accessToken = generateAccessToken(tokenPayload);
    const refreshToken = generateRefreshToken(tokenPayload);

    // Remove password from response
    const { password, ...userWithoutPassword } = user;

    return {
      user: userWithoutPassword,
      accessToken,
      refreshToken
    };
  }

  async login(dto: LoginDto): Promise<AuthResponse> {
    // Find user
    const user = await prisma.user.findUnique({
      where: { email: dto.email }
    });

    if (!user) {
      throw new Error('Invalid credentials');
    }

    // Verify password
    const isPasswordValid = await bcrypt.compare(dto.password, user.password);

    if (!isPasswordValid) {
      throw new Error('Invalid credentials');
    }

    // Generate tokens
    const tokenPayload: TokenPayload = {
      userId: user.id,
      email: user.email,
      role: user.role
    };

    const accessToken = generateAccessToken(tokenPayload);
    const refreshToken = generateRefreshToken(tokenPayload);

    // Remove password from response
    const { password, ...userWithoutPassword } = user;

    return {
      user: userWithoutPassword,
      accessToken,
      refreshToken
    };
  }

  async refreshAccessToken(refreshToken: string): Promise<{ accessToken: string }> {
    try {
      const decoded = verifyRefreshToken(refreshToken);

      // Verify user still exists
      const user = await prisma.user.findUnique({
        where: { id: decoded.userId }
      });

      if (!user) {
        throw new Error('User not found');
      }

      const tokenPayload: TokenPayload = {
        userId: user.id,
        email: user.email,
        role: user.role
      };

      const accessToken = generateAccessToken(tokenPayload);

      return { accessToken };
    } catch (error) {
      throw new Error('Invalid refresh token');
    }
  }
}

// middleware/auth.ts
import { Request, Response, NextFunction } from 'express';
import { verifyAccessToken, TokenPayload } from '../utils/jwt';

export interface AuthRequest extends Request {
  user?: TokenPayload;
}

export const authenticate = (
  req: AuthRequest,
  res: Response,
  next: NextFunction
) => {
  try {
    const authHeader = req.headers.authorization;

    if (!authHeader || !authHeader.startsWith('Bearer ')) {
      return res.status(401).json({
        status: 'error',
        message: 'No token provided'
      });
    }

    const token = authHeader.substring(7);
    const decoded = verifyAccessToken(token);

    req.user = decoded;
    next();
  } catch (error) {
    return res.status(401).json({
      status: 'error',
      message: 'Invalid or expired token'
    });
  }
};

export const authorize = (...roles: string[]) => {
  return (req: AuthRequest, res: Response, next: NextFunction) => {
    if (!req.user) {
      return res.status(401).json({
        status: 'error',
        message: 'Not authenticated'
      });
    }

    if (!roles.includes(req.user.role)) {
      return res.status(403).json({
        status: 'error',
        message: 'Insufficient permissions'
      });
    }

    next();
  };
};

// Usage example
// router.get('/admin', authenticate, authorize('ADMIN'), adminController.getDashboard);
```

---

### Pattern 4: Python FastAPI Backend

**Generate FastAPI application:**

```
Chat:

"Create FastAPI application for a task management API with:
- CRUD operations for tasks
- Pydantic models for validation
- SQLAlchemy ORM
- Authentication with OAuth2
- Async endpoints
- OpenAPI documentation"
```

**Result:**

```python
# models.py
from sqlalchemy import Column, String, Boolean, DateTime, ForeignKey
from sqlalchemy.orm import relationship
from sqlalchemy.ext.declarative import declarative_base
from datetime import datetime
import uuid

Base = declarative_base()

def generate_uuid():
    return str(uuid.uuid4())

class User(Base):
    __tablename__ = "users"
    
    id = Column(String, primary_key=True, default=generate_uuid)
    email = Column(String, unique=True, index=True, nullable=False)
    hashed_password = Column(String, nullable=False)
    full_name = Column(String)
    is_active = Column(Boolean, default=True)
    created_at = Column(DateTime, default=datetime.utcnow)
    
    tasks = relationship("Task", back_populates="owner", cascade="all, delete-orphan")

class Task(Base):
    __tablename__ = "tasks"
    
    id = Column(String, primary_key=True, default=generate_uuid)
    title = Column(String, nullable=False)
    description = Column(String)
    completed = Column(Boolean, default=False)
    owner_id = Column(String, ForeignKey("users.id"), nullable=False)
    created_at = Column(DateTime, default=datetime.utcnow)
    updated_at = Column(DateTime, default=datetime.utcnow, onupdate=datetime.utcnow)
    
    owner = relationship("User", back_populates="tasks")

# schemas.py
from pydantic import BaseModel, EmailStr, Field
from datetime import datetime
from typing import Optional

class UserBase(BaseModel):
    email: EmailStr
    full_name: Optional[str] = None

class UserCreate(UserBase):
    password: str = Field(..., min_length=8)

class UserResponse(UserBase):
    id: str
    is_active: bool
    created_at: datetime
    
    class Config:
        from_attributes = True

class TaskBase(BaseModel):
    title: str = Field(..., min_length=1, max_length=200)
    description: Optional[str] = None
    completed: bool = False

class TaskCreate(TaskBase):
    pass

class TaskUpdate(BaseModel):
    title: Optional[str] = Field(None, min_length=1, max_length=200)
    description: Optional[str] = None
    completed: Optional[bool] = None

class TaskResponse(TaskBase):
    id: str
    owner_id: str
    created_at: datetime
    updated_at: datetime
    
    class Config:
        from_attributes = True

class Token(BaseModel):
    access_token: str
    token_type: str

class TokenData(BaseModel):
    email: Optional[str] = None

# database.py
from sqlalchemy.ext.asyncio import create_async_engine, AsyncSession
from sqlalchemy.orm import sessionmaker
import os

DATABASE_URL = os.getenv("DATABASE_URL", "sqlite+aiosqlite:///./test.db")

engine = create_async_engine(DATABASE_URL, echo=True)
async_session_maker = sessionmaker(engine, class_=AsyncSession, expire_on_commit=False)

async def get_db():
    async with async_session_maker() as session:
        yield session

# auth.py
from passlib.context import CryptContext
from jose import JWTError, jwt
from datetime import datetime, timedelta
from fastapi import Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy.future import select
import os

SECRET_KEY = os.getenv("SECRET_KEY", "your-secret-key-here")
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

pwd_context = CryptContext(schemes=["bcrypt"], deprecated="auto")
oauth2_scheme = OAuth2PasswordBearer(tokenUrl="token")

def verify_password(plain_password: str, hashed_password: str) -> bool:
    return pwd_context.verify(plain_password, hashed_password)

def get_password_hash(password: str) -> str:
    return pwd_context.hash(password)

def create_access_token(data: dict, expires_delta: Optional[timedelta] = None):
    to_encode = data.copy()
    if expires_delta:
        expire = datetime.utcnow() + expires_delta
    else:
        expire = datetime.utcnow() + timedelta(minutes=15)
    to_encode.update({"exp": expire})
    encoded_jwt = jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
    return encoded_jwt

async def get_current_user(
    token: str = Depends(oauth2_scheme),
    db: AsyncSession = Depends(get_db)
):
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Could not validate credentials",
        headers={"WWW-Authenticate": "Bearer"},
    )
    try:
        payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
        email: str = payload.get("sub")
        if email is None:
            raise credentials_exception
    except JWTError:
        raise credentials_exception
    
    result = await db.execute(select(User).filter(User.email == email))
    user = result.scalars().first()
    if user is None:
        raise credentials_exception
    return user

# main.py
from fastapi import FastAPI, Depends, HTTPException, status
from fastapi.security import OAuth2PasswordRequestForm
from sqlalchemy.ext.asyncio import AsyncSession
from sqlalchemy.future import select
from typing import List
import models, schemas, database, auth

app = FastAPI(
    title="Task Management API",
    description="A simple task management API built with FastAPI",
    version="1.0.0"
)

@app.on_event("startup")
async def startup():
    async with database.engine.begin() as conn:
        await conn.run_sync(models.Base.metadata.create_all)

@app.post("/register", response_model=schemas.UserResponse, status_code=status.HTTP_201_CREATED)
async def register(user: schemas.UserCreate, db: AsyncSession = Depends(database.get_db)):
    """Register a new user"""
    # Check if user exists
    result = await db.execute(select(models.User).filter(models.User.email == user.email))
    if result.scalars().first():
        raise HTTPException(status_code=400, detail="Email already registered")
    
    # Create user
    hashed_password = auth.get_password_hash(user.password)
    db_user = models.User(
        email=user.email,
        hashed_password=hashed_password,
        full_name=user.full_name
    )
    db.add(db_user)
    await db.commit()
    await db.refresh(db_user)
    return db_user

@app.post("/token", response_model=schemas.Token)
async def login(
    form_data: OAuth2PasswordRequestForm = Depends(),
    db: AsyncSession = Depends(database.get_db)
):
    """Login and get access token"""
    result = await db.execute(select(models.User).filter(models.User.email == form_data.username))
    user = result.scalars().first()
    
    if not user or not auth.verify_password(form_data.password, user.hashed_password):
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Incorrect email or password",
            headers={"WWW-Authenticate": "Bearer"},
        )
    
    access_token = auth.create_access_token(
        data={"sub": user.email},
        expires_delta=timedelta(minutes=auth.ACCESS_TOKEN_EXPIRE_MINUTES)
    )
    return {"access_token": access_token, "token_type": "bearer"}

@app.get("/tasks", response_model=List[schemas.TaskResponse])
async def get_tasks(
    db: AsyncSession = Depends(database.get_db),
    current_user: models.User = Depends(auth.get_current_user)
):
    """Get all tasks for current user"""
    result = await db.execute(
        select(models.Task).filter(models.Task.owner_id == current_user.id)
    )
    tasks = result.scalars().all()
    return tasks

@app.post("/tasks", response_model=schemas.TaskResponse, status_code=status.HTTP_201_CREATED)
async def create_task(
    task: schemas.TaskCreate,
    db: AsyncSession = Depends(database.get_db),
    current_user: models.User = Depends(auth.get_current_user)
):
    """Create a new task"""
    db_task = models.Task(**task.dict(), owner_id=current_user.id)
    db.add(db_task)
    await db.commit()
    await db.refresh(db_task)
    return db_task

@app.get("/tasks/{task_id}", response_model=schemas.TaskResponse)
async def get_task(
    task_id: str,
    db: AsyncSession = Depends(database.get_db),
    current_user: models.User = Depends(auth.get_current_user)
):
    """Get a specific task"""
    result = await db.execute(
        select(models.Task).filter(
            models.Task.id == task_id,
            models.Task.owner_id == current_user.id
        )
    )
    task = result.scalars().first()
    if not task:
        raise HTTPException(status_code=404, detail="Task not found")
    return task

@app.put("/tasks/{task_id}", response_model=schemas.TaskResponse)
async def update_task(
    task_id: str,
    task_update: schemas.TaskUpdate,
    db: AsyncSession = Depends(database.get_db),
    current_user: models.User = Depends(auth.get_current_user)
):
    """Update a task"""
    result = await db.execute(
        select(models.Task).filter(
            models.Task.id == task_id,
            models.Task.owner_id == current_user.id
        )
    )
    task = result.scalars().first()
    if not task:
        raise HTTPException(status_code=404, detail="Task not found")
    
    # Update fields
    for field, value in task_update.dict(exclude_unset=True).items():
        setattr(task, field, value)
    
    await db.commit()
    await db.refresh(task)
    return task

@app.delete("/tasks/{task_id}", status_code=status.HTTP_204_NO_CONTENT)
async def delete_task(
    task_id: str,
    db: AsyncSession = Depends(database.get_db),
    current_user: models.User = Depends(auth.get_current_user)
):
    """Delete a task"""
    result = await db.execute(
        select(models.Task).filter(
            models.Task.id == task_id,
            models.Task.owner_id == current_user.id
        )
    )
    task = result.scalars().first()
    if not task:
        raise HTTPException(status_code=404, detail="Task not found")
    
    await db.delete(task)
    await db.commit()
    return None
```

---

## 📚 API Documentation Generation

### Pattern 5: OpenAPI/Swagger Documentation

**For Express/TypeScript:**

```
Chat:

"Configure Swagger/OpenAPI documentation for Express API with:
- Auto-generated from JSDoc comments
- Interactive API explorer
- Request/response schemas
- Authentication documentation"
```

**Result:**

```typescript
// swagger.ts
import swaggerJsdoc from 'swagger-jsdoc';
import swaggerUi from 'swagger-ui-express';
import { Express } from 'express';

const options: swaggerJsdoc.Options = {
  definition: {
    openapi: '3.0.0',
    info: {
      title: 'Blog API',
      version: '1.0.0',
      description: 'A simple blog API with authentication',
      contact: {
        name: 'API Support',
        email: 'support@example.com'
      }
    },
    servers: [
      {
        url: 'http://localhost:3000',
        description: 'Development server'
      },
      {
        url: 'https://api.example.com',
        description: 'Production server'
      }
    ],
    components: {
      securitySchemes: {
        bearerAuth: {
          type: 'http',
          scheme: 'bearer',
          bearerFormat: 'JWT'
        }
      },
      schemas: {
        Post: {
          type: 'object',
          required: ['title', 'content', 'author'],
          properties: {
            id: {
              type: 'string',
              format: 'uuid',
              description: 'Unique identifier'
            },
            title: {
              type: 'string',
              minLength: 3,
              maxLength: 200,
              description: 'Post title'
            },
            content: {
              type: 'string',
              minLength: 10,
              description: 'Post content'
            },
            author: {
              type: 'string',
              description: 'Author name'
            },
            createdAt: {
              type: 'string',
              format: 'date-time'
            },
            updatedAt: {
              type: 'string',
              format: 'date-time'
            }
          }
        },
        Error: {
          type: 'object',
          properties: {
            status: {
              type: 'string',
              example: 'error'
            },
            message: {
              type: 'string',
              example: 'An error occurred'
            }
          }
        }
      }
    },
    security: [
      {
        bearerAuth: []
      }
    ]
  },
  apis: ['./src/routes/*.ts', './src/controllers/*.ts']
};

const swaggerSpec = swaggerJsdoc(options);

export const setupSwagger = (app: Express) => {
  app.use('/api-docs', swaggerUi.serve, swaggerUi.setup(swaggerSpec, {
    customCss: '.swagger-ui .topbar { display: none }',
    customSiteTitle: 'Blog API Documentation'
  }));
  
  // Serve swagger spec as JSON
  app.get('/api-docs.json', (req, res) => {
    res.setHeader('Content-Type', 'application/json');
    res.send(swaggerSpec);
  });
};
```

---

## 🧪 Backend Testing

### Pattern 6: Integration Tests

**Generate API integration tests:**

```
Chat:

"Create Jest integration tests for the Post API with:
- Test POST /api/posts (create)
- Test GET /api/posts (list all)
- Test GET /api/posts/:id (get one)
- Test PUT /api/posts/:id (update)
- Test DELETE /api/posts/:id (delete)
- Setup test database
- Mock authentication"
```

**Result:**

```typescript
// tests/setup.ts
import { PrismaClient } from '@prisma/client';

const prisma = new PrismaClient();

export const setupTestDb = async () => {
  // Clean database before tests
  await prisma.orderItem.deleteMany();
  await prisma.order.deleteMany();
  await prisma.product.deleteMany();
  await prisma.user.deleteMany();
};

export const teardownTestDb = async () => {
  await prisma.$disconnect();
};

// tests/posts.test.ts
import request from 'supertest';
import app from '../src/app';
import { setupTestDb, teardownTestDb } from './setup';
import { generateAccessToken } from '../src/utils/jwt';

describe('Posts API', () => {
  let authToken: string;
  let testUserId: string;
  let testPostId: string;

  beforeAll(async () => {
    await setupTestDb();
    
    // Create test user and get token
    const userResponse = await request(app)
      .post('/api/auth/register')
      .send({
        email: 'test@example.com',
        password: 'Test123!',
        firstName: 'Test',
        lastName: 'User'
      });
    
    testUserId = userResponse.body.data.user.id;
    authToken = userResponse.body.data.accessToken;
  });

  afterAll(async () => {
    await teardownTestDb();
  });

  describe('POST /api/posts', () => {
    it('should create a new post with valid data', async () => {
      const response = await request(app)
        .post('/api/posts')
        .set('Authorization', `Bearer ${authToken}`)
        .send({
          title: 'Test Post',
          content: 'This is a test post content',
          author: 'Test Author'
        });

      expect(response.status).toBe(201);
      expect(response.body.status).toBe('success');
      expect(response.body.data.post).toHaveProperty('id');
      expect(response.body.data.post.title).toBe('Test Post');
      
      testPostId = response.body.data.post.id;
    });

    it('should return 400 with invalid data', async () => {
      const response = await request(app)
        .post('/api/posts')
        .set('Authorization', `Bearer ${authToken}`)
        .send({
          title: 'Te', // Too short
          content: 'Short'
        });

      expect(response.status).toBe(400);
      expect(response.body.status).toBe('error');
      expect(response.body.errors).toBeDefined();
    });

    it('should return 401 without authentication', async () => {
      const response = await request(app)
        .post('/api/posts')
        .send({
          title: 'Test Post',
          content: 'This is a test post content',
          author: 'Test Author'
        });

      expect(response.status).toBe(401);
    });
  });

  describe('GET /api/posts', () => {
    it('should return all posts', async () => {
      const response = await request(app)
        .get('/api/posts')
        .set('Authorization', `Bearer ${authToken}`);

      expect(response.status).toBe(200);
      expect(response.body.status).toBe('success');
      expect(Array.isArray(response.body.data.posts)).toBe(true);
      expect(response.body.data.posts.length).toBeGreaterThan(0);
    });
  });

  describe('GET /api/posts/:id', () => {
    it('should return a post by ID', async () => {
      const response = await request(app)
        .get(`/api/posts/${testPostId}`)
        .set('Authorization', `Bearer ${authToken}`);

      expect(response.status).toBe(200);
      expect(response.body.status).toBe('success');
      expect(response.body.data.post.id).toBe(testPostId);
    });

    it('should return 404 for non-existent post', async () => {
      const fakeId = '00000000-0000-0000-0000-000000000000';
      const response = await request(app)
        .get(`/api/posts/${fakeId}`)
        .set('Authorization', `Bearer ${authToken}`);

      expect(response.status).toBe(404);
    });

    it('should return 400 for invalid ID format', async () => {
      const response = await request(app)
        .get('/api/posts/invalid-id')
        .set('Authorization', `Bearer ${authToken}`);

      expect(response.status).toBe(400);
    });
  });

  describe('PUT /api/posts/:id', () => {
    it('should update a post', async () => {
      const response = await request(app)
        .put(`/api/posts/${testPostId}`)
        .set('Authorization', `Bearer ${authToken}`)
        .send({
          title: 'Updated Test Post',
          content: 'This is updated content'
        });

      expect(response.status).toBe(200);
      expect(response.body.status).toBe('success');
      expect(response.body.data.post.title).toBe('Updated Test Post');
    });

    it('should return 404 for non-existent post', async () => {
      const fakeId = '00000000-0000-0000-0000-000000000000';
      const response = await request(app)
        .put(`/api/posts/${fakeId}`)
        .set('Authorization', `Bearer ${authToken}`)
        .send({
          title: 'Updated Title'
        });

      expect(response.status).toBe(404);
    });
  });

  describe('DELETE /api/posts/:id', () => {
    it('should delete a post', async () => {
      const response = await request(app)
        .delete(`/api/posts/${testPostId}`)
        .set('Authorization', `Bearer ${authToken}`);

      expect(response.status).toBe(204);
    });

    it('should return 404 when deleting non-existent post', async () => {
      const response = await request(app)
        .delete(`/api/posts/${testPostId}`)
        .set('Authorization', `Bearer ${authToken}`);

      expect(response.status).toBe(404);
    });
  });
});
```

---

## 🎯 Custom Instructions for Backend

Create `.github/copilot-instructions.md`:

```markdown
# Backend Development Standards

## Project Structure
- `/src/routes` - API route definitions
- `/src/controllers` - Request handlers
- `/src/services` - Business logic layer
- `/src/models` - Database models (Prisma/TypeORM)
- `/src/middleware` - Express middleware
- `/src/utils` - Helper functions
- `/tests` - Integration and unit tests

## API Design
- Use RESTful conventions
- Version APIs (/api/v1/)
- Return consistent response format:
  ```json
  {
    "status": "success" | "error",
    "data": { ... },
    "message": "..."
  }
  ```
- Use proper HTTP status codes

## Error Handling
- All async functions wrapped in try-catch
- Custom AppError class for operational errors
- Centralized error handling middleware
- Never expose internal error details to clients

## Security
- Input validation on all endpoints (express-validator/Zod)
- SQL injection prevention via ORM parameterization
- XSS protection with helmet
- Rate limiting on auth endpoints
- JWT for authentication
- CORS configuration for production

## Database
- Use Prisma for PostgreSQL/MySQL
- Migrations for schema changes
- Seed data for development
- Transactions for multi-step operations
- Indexes on frequently queried fields

## Testing
- Integration tests for all API endpoints
- Unit tests for business logic
- Test database separate from development
- 80% code coverage minimum

## Documentation
- OpenAPI/Swagger for all endpoints
- JSDoc comments for complex functions
- README with setup instructions
```

---

## 💡 Pro Tips

### 1. **Database Migration Workflow**

```
Chat:

"Generate Prisma migration for adding user authentication:
- Add password field to User
- Add role enum (admin, user)
- Add timestamps
- Create migration script"
```

### 2. **GraphQL Schema Generation**

```
Chat:

"Create GraphQL schema with resolvers for:
- User (id, email, posts)
- Post (id, title, content, author)
- Mutations: createPost, updatePost, deletePost
- Queries: posts, post(id), userPosts(userId)
Use type-graphql with TypeScript"
```

### 3. **Microservices Communication**

```
Chat:

"Create message queue pattern with RabbitMQ:
- Order service publishes order.created event
- Inventory service consumes and updates stock
- Notification service sends email
Include TypeScript types and error handling"
```

---

## 🚨 Common Pitfalls

### ❌ Don't:
- ❌ Skip input validation
- ❌ Expose internal errors to clients
- ❌ Store passwords in plain text
- ❌ Hardcode secrets (use environment variables)
- ❌ Skip database indexes on large tables
- ❌ Forget to handle race conditions in transactions

### ✅ Do:
- ✅ Validate all user input
- ✅ Use parameterized queries (ORM)
- ✅ Implement rate limiting
- ✅ Log errors for debugging
- ✅ Use transactions for data integrity
- ✅ Write integration tests

---

## 📚 Related Resources

**Official Documentation:**
- [Express.js Guide](https://expressjs.com/)
- [FastAPI Docs](https://fastapi.tiangolo.com/)
- [Prisma Documentation](https://www.prisma.io/docs)
- [TypeORM Documentation](https://typeorm.io/)
- [JWT.io](https://jwt.io/)
- [OpenAPI Specification](https://swagger.io/specification/)

**Copilot Features Used:**
- Inline completions for endpoint scaffolding
- Chat Agent mode for multi-file API generation
- Custom instructions for framework patterns
- Slash commands for test generation

---

## 🎯 Next Steps

- **Continue to:** [Data Engineering & SQL →](03-data-engineering-sql.md)
- **Related:** [Testing Patterns](../06-quality-performance/01-test-generation.md)
- **Practice:** Build a complete CRUD API with authentication
- **Experiment:** Create custom backend agent for security audits

---

**Updated:** December 2025  
**Part 5, Section 2 of 4**

