---
title: "Section 2: Custom Agents & Instructions"
parent: "Part 3: Advanced Chat & Agent Development"
nav_order: 2
---

# Section 2: Custom Agents & Instructions

**Part 3: Advanced Chat & Agent Development**  
**Target Audience**: Senior developers ready to customize Copilot for their team  
**Time to Complete**: 1.5 hours  
**Prerequisites**: Completed Section 1 (Chat Modes Mastery)

---

## 📋 Overview

Custom agents and instructions are the most powerful way to tailor GitHub Copilot to your team's workflows, coding standards, and specialized domains. This section covers everything from creating your first custom agent to building a library of specialized assistants for security, performance, documentation, and more.

**What you'll master:**
- Creating `.agent.md` files for specialized workflows
- Writing effective custom instructions in `.github/copilot-instructions.md`
- Building a library of reusable agents for your team
- Agent personas (Security, Performance, Documentation, DevOps)
- Sharing and versioning agents across teams
- Best practices for agent design

---

## 🎯 Custom Agents vs Custom Instructions

### Quick Comparison

| Feature | Custom Agents | Custom Instructions |
|---------|--------------|-------------------|
| **File** | `.agent.md` | `.github/copilot-instructions.md` |
| **Location** | `.github/agents/` or user profile | `.github/` (repo root) or `AGENTS.md` |
| **Purpose** | Specialized assistant personas | Global coding standards/guidelines |
| **Switchable** | ✅ Yes (via agent picker) | ❌ No (always active) |
| **Tools** | Configurable per agent | All tools available |
| **Model** | Can specify per agent | Uses selected model |
| **Scope** | Task-specific | Repository-wide |

### When to Use Each

**Use Custom Agents when:**
- ✅ You need specialized behavior for specific tasks
- ✅ Different team members have different workflows
- ✅ You want to restrict tools for certain operations
- ✅ You need different AI models for different tasks
- ✅ You want reusable "personas" (Security, DevOps, etc.)

**Use Custom Instructions when:**
- ✅ You want consistent coding standards across all chats
- ✅ Team conventions should always apply
- ✅ Tech stack and patterns are standard
- ✅ Security rules apply to all code
- ✅ No need for task-specific switching

**Best practice:** Use both together!
- Custom instructions for universal standards
- Custom agents for specialized workflows

---

## 1. Custom Instructions: Repository-Wide Standards

### What are Custom Instructions?

**Custom instructions** are repository-wide guidelines that Copilot automatically follows for all code suggestions and chat interactions. They're defined in `.github/copilot-instructions.md` and apply to every developer working in the repository.

**Think of them as:**
- Your team's coding standards documented for AI
- Automatic onboarding for Copilot
- Consistent code style enforcement
- Repository-specific best practices

### Creating Your First Custom Instructions File

#### Location and Setup

**Option 1: Repository-wide (Recommended for teams)**
```
your-repo/
├── .github/
│   └── copilot-instructions.md    ← Applies to entire repo
└── src/
```

**Option 2: Consolidated (Multiple agents)**
```
your-repo/
├── AGENTS.md                       ← All agents in one file
└── src/
```

**Option 3: User-level (Personal use)**
- Stored in VS Code user profile
- Applied across all workspaces
- Good for personal preferences

#### Creating the File

**Method 1: Manual creation**
```bash
mkdir -p .github
touch .github/copilot-instructions.md
```

**Method 2: Let Copilot generate it**
```markdown
[Agent mode]
"Generate a copilot-instructions.md file for our React TypeScript e-commerce project with 
best practices for component structure, state management, and API calls"
```

### Custom Instructions File Structure

**Basic template:**

```markdown
# Project: [Your Project Name]

## Tech Stack
- Language: TypeScript 5.0
- Framework: React 18 with Next.js 14
- State Management: Zustand
- Database: PostgreSQL 15 with Prisma ORM
- Testing: Jest + React Testing Library
- Styling: Tailwind CSS

## Coding Standards

### TypeScript
- Use strict mode with all TypeScript checks enabled
- Define interfaces for all props and function parameters
- Use type inference where obvious, explicit types otherwise
- Avoid `any` type - use `unknown` or proper types
- Use enums for fixed sets of values

### React Components
- Use functional components with hooks (no class components)
- Extract custom hooks for reusable logic
- One component per file
- Props interface named after component: `interface ButtonProps`
- Use composition over prop drilling

### File Naming
- Components: PascalCase (e.g., `UserProfile.tsx`)
- Hooks: camelCase with 'use' prefix (e.g., `useAuth.ts`)
- Utilities: camelCase (e.g., `formatDate.ts`)
- Constants: UPPER_SNAKE_CASE (e.g., `API_ENDPOINTS.ts`)

### API Integration
- All API calls via centralized API client (`src/lib/api.ts`)
- Use React Query for data fetching and caching
- Error handling with custom error boundaries
- No direct `fetch()` calls in components

### Error Handling
- Use try-catch for async operations
- Custom error classes for different error types
- Error boundaries for React component errors
- Centralized error logging via `logger.error()`
- User-friendly error messages (no stack traces in UI)

### Testing
- Unit tests for all business logic functions
- Component tests for UI components (RTL)
- Integration tests for API routes
- E2E tests for critical user flows (Playwright)
- Minimum 80% code coverage
- Test file naming: `*.test.ts` or `*.spec.ts`

## Security Requirements
- Input validation with Zod schemas on all API endpoints
- SQL injection prevention via Prisma (no raw queries)
- XSS protection with proper escaping (React handles most)
- CSRF tokens on state-changing requests
- Rate limiting on public endpoints (10 req/min per IP)
- Secrets stored in environment variables, never in code
- Authentication checks on all protected routes
- Password hashing with bcrypt (min 12 rounds)

## Performance Guidelines
- Use React.memo for expensive components
- Implement virtualization for long lists (react-window)
- Lazy load routes and heavy components
- Optimize images (Next.js Image component)
- Database queries should use indexes
- API responses should be paginated (max 100 items)
- Bundle size monitored (max 300KB initial JS)

## Documentation
- JSDoc for all public functions and components
- README.md in each major directory explaining its purpose
- ADRs (Architecture Decision Records) for significant decisions
- API documentation with OpenAPI/Swagger
- Inline comments for complex logic only (code should be self-documenting)

## Git Workflow
- Branch naming: `feature/`, `fix/`, `refactor/`, `docs/`
- Commit messages: Conventional Commits format
- PR title must explain what and why
- All PRs require at least one approval
- All tests must pass before merge
- No direct commits to `main` branch

## Environment
- Node.js 20+ required
- Use `pnpm` as package manager (not npm or yarn)
- Environment variables in `.env.local` (never commit)
- Development server: `pnpm dev` on port 3000
- Database: PostgreSQL via Docker Compose

## Common Patterns

### API Route Pattern
```typescript
// app/api/users/route.ts
import { NextRequest, NextResponse } from 'next/server';
import { z } from 'zod';
import { prisma } from '@/lib/prisma';
import { authenticate } from '@/lib/auth';

const createUserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(1),
});

export async function POST(req: NextRequest) {
  try {
    // Authenticate
    const user = await authenticate(req);
    if (!user) {
      return NextResponse.json({ error: 'Unauthorized' }, { status: 401 });
    }

    // Validate input
    const body = await req.json();
    const data = createUserSchema.parse(body);

    // Business logic
    const newUser = await prisma.user.create({ data });

    return NextResponse.json(newUser, { status: 201 });
  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json({ error: error.errors }, { status: 400 });
    }
    console.error('Error creating user:', error);
    return NextResponse.json({ error: 'Internal server error' }, { status: 500 });
  }
}
```

### React Component Pattern
```typescript
// components/UserCard.tsx
import { User } from '@/types/user';

interface UserCardProps {
  user: User;
  onEdit?: (user: User) => void;
}

export function UserCard({ user, onEdit }: UserCardProps) {
  return (
    <div className="card">
      <h3>{user.name}</h3>
      <p>{user.email}</p>
      {onEdit && (
        <button onClick={() => onEdit(user)}>Edit</button>
      )}
    </div>
  );
}
```

## What NOT to Do
- ❌ Never use `var` (use `const` or `let`)
- ❌ Never commit console.log statements to production
- ❌ Never store secrets in code or committed files
- ❌ Never make database queries in React components
- ❌ Never use `// @ts-ignore` (fix the type issue instead)
- ❌ Never catch errors without logging them
- ❌ Never trust user input without validation
```

### Real-World Custom Instructions Examples

#### Example 1: Python Django Project

**`.github/copilot-instructions.md`:**

```markdown
# Project: Enterprise CRM System

## Tech Stack
- Python 3.11+
- Django 5.0 with Django REST Framework
- PostgreSQL 15
- Celery for background tasks
- Redis for caching
- Pytest for testing

## Code Style
- Follow PEP 8 strictly
- Use Black for formatting (line length 100)
- Use type hints for all function signatures
- Docstrings in Google format
- All imports organized: stdlib → third-party → local

## Django Patterns

### Views
- Use class-based views for CRUD operations
- Function views for simple operations
- Always include permission classes
- Validate input with DRF serializers

### Models
- Always include created_at and updated_at timestamps
- Use Django's built-in User model (don't create custom unless necessary)
- Foreign keys should have related_name
- Add indexes for frequently queried fields
- Override `__str__` method for all models

### Serializers
- One serializer per model minimum
- Separate serializers for read (detailed) and write (minimal)
- Use ModelSerializer for simple cases
- Validate data in serializer validate_* methods

### URL Patterns
- Use path() not url() (deprecated)
- Name all URL patterns for reverse()
- API URLs under `/api/v1/` namespace
- Use viewset routers for standard REST APIs

## Testing
- Minimum 90% code coverage
- Use pytest fixtures for reusable setup
- Mock external API calls
- Test file naming: `test_*.py`
- Organize: `tests/unit/`, `tests/integration/`, `tests/e2e/`

## Security
- CSRF protection enabled for all state-changing views
- Use Django's authentication backend (no custom auth)
- Secrets in environment variables only
- SQL injection prevention via ORM (no raw SQL)
- XSS prevention via template auto-escaping
- All API endpoints require authentication unless explicitly public

## Performance
- Use `select_related()` and `prefetch_related()` to avoid N+1
- Database queries should be indexed
- Use caching for expensive operations (Redis)
- Background tasks via Celery for anything > 1 second
- Pagination for all list endpoints (max 100 items)

## Documentation
- Docstrings for all classes and public methods
- API documentation via drf-spectacular (OpenAPI)
- README in each app directory

## Common Patterns

### API View Pattern
```python
# api/views.py
from rest_framework import viewsets, permissions
from rest_framework.decorators import action
from rest_framework.response import Response
from .models import Customer
from .serializers import CustomerSerializer, CustomerDetailSerializer

class CustomerViewSet(viewsets.ModelViewSet):
    """
    ViewSet for managing customers.
    
    Provides standard CRUD operations plus custom actions.
    """
    queryset = Customer.objects.all()
    permission_classes = [permissions.IsAuthenticated]
    
    def get_serializer_class(self):
        if self.action == 'list':
            return CustomerSerializer
        return CustomerDetailSerializer
    
    def get_queryset(self):
        # Optimize query with related objects
        return Customer.objects.select_related('account').prefetch_related('orders')
    
    @action(detail=True, methods=['post'])
    def send_welcome_email(self, request, pk=None):
        """Send welcome email to customer."""
        customer = self.get_object()
        # Async task
        from .tasks import send_welcome_email_task
        send_welcome_email_task.delay(customer.id)
        return Response({'status': 'email queued'})
```

### Model Pattern
```python
# models.py
from django.db import models
from django.contrib.auth import get_user_model

User = get_user_model()

class Customer(models.Model):
    """Represents a customer in the CRM system."""
    
    user = models.OneToOneField(
        User,
        on_delete=models.CASCADE,
        related_name='customer_profile'
    )
    company_name = models.CharField(max_length=255, db_index=True)
    industry = models.CharField(max_length=100)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        ordering = ['-created_at']
        indexes = [
            models.Index(fields=['company_name']),
        ]
    
    def __str__(self):
        return f"{self.company_name} ({self.user.email})"
```
```

#### Example 2: Node.js Express Microservices

**`.github/copilot-instructions.md`:**

```markdown
# Project: Microservices Platform

## Architecture
- Microservices architecture (12+ services)
- Node.js 20 + TypeScript 5
- Express.js for HTTP servers
- RabbitMQ for event bus
- PostgreSQL + Redis
- Docker containers
- Kubernetes deployment

## Service Structure
```
service-name/
├── src/
│   ├── controllers/      # HTTP handlers
│   ├── services/         # Business logic
│   ├── models/           # Database models
│   ├── events/           # Event handlers
│   ├── middleware/       # Express middleware
│   ├── utils/            # Utilities
│   └── app.ts            # Express app
├── tests/
├── Dockerfile
└── package.json
```

## Coding Standards

### TypeScript
- Strict mode enabled
- No implicit any
- All async functions must have Promise return type
- Use interfaces for data structures
- Use types for unions/intersections

### Error Handling
- Custom error classes extending Error
- Centralized error middleware
- Structured error logging (Winston)
- Never expose internal errors to clients

```typescript
// utils/errors.ts
export class AppError extends Error {
  constructor(
    public statusCode: number,
    public message: string,
    public isOperational = true
  ) {
    super(message);
    Error.captureStackTrace(this, this.constructor);
  }
}

export class NotFoundError extends AppError {
  constructor(resource: string) {
    super(404, `${resource} not found`);
  }
}

export class ValidationError extends AppError {
  constructor(message: string) {
    super(400, message);
  }
}
```

### Event-Driven Patterns
- All state changes emit events
- Events named in past tense (UserCreated, OrderPlaced)
- Event payload includes timestamp and version
- Idempotency keys for event processing

```typescript
// events/emitter.ts
import { EventEmitter } from '@/lib/eventBus';

interface UserCreatedEvent {
  type: 'UserCreated';
  data: {
    userId: string;
    email: string;
    timestamp: string;
  };
}

export async function publishUserCreated(userId: string, email: string) {
  await EventEmitter.publish<UserCreatedEvent>({
    type: 'UserCreated',
    data: {
      userId,
      email,
      timestamp: new Date().toISOString(),
    },
  });
}
```

### API Design
- RESTful conventions
- Versioned: `/api/v1/...`
- JSON:API specification
- OpenAPI documentation
- Rate limiting (Redis)
- Authentication via JWT
- CORS configured per environment

### Database
- Use Knex.js query builder or Prisma ORM
- Migrations for all schema changes
- Transactions for multi-step operations
- Connection pooling configured
- Read replicas for heavy read services

### Testing
- Unit tests: 80%+ coverage
- Integration tests for APIs
- Contract tests between services
- Load tests for critical paths
- Test files: `*.test.ts`

### Observability
- Structured logging with Winston
- Request ID propagation
- OpenTelemetry tracing
- Prometheus metrics
- Health check endpoints

## Service Communication
- Synchronous: REST API calls via axios
- Asynchronous: RabbitMQ events
- Retry logic with exponential backoff
- Circuit breakers for external services

## Security
- JWT authentication (256-bit secret)
- Rate limiting (100 req/min per user)
- Input validation with Zod
- SQL injection prevention via ORM
- Secrets in Kubernetes secrets (not env files)
- OWASP Top 10 compliance

## Deployment
- One service per Docker container
- Health checks: `/health` and `/ready`
- Graceful shutdown on SIGTERM
- Rolling updates (zero downtime)
- Resource limits in k8s manifests
```

### Best Practices for Writing Custom Instructions

**1. Start with your tech stack:**
```markdown
## Tech Stack
- Language and version
- Framework and major libraries
- Database
- Testing tools
- Deployment platform
```

**2. Be specific about conventions:**
```markdown
# ❌ Vague
"Use good naming conventions"

# ✅ Specific
"File naming: PascalCase for components (UserCard.tsx), camelCase for utilities (formatDate.ts)"
```

**3. Include concrete examples:**
```markdown
## API Route Pattern
```typescript
[actual code example]
```
```

**4. Specify what NOT to do:**
```markdown
## What NOT to Do
- ❌ Never use var (use const or let)
- ❌ Never commit API keys
- ❌ Never use any type
```

**5. Keep it updated:**
```markdown
# Update when:
- Tech stack changes
- Team adopts new patterns
- New security requirements
- Framework version updates
```

---

## 2. Custom Agents: Specialized Assistants

### What are Custom Agents?

**Custom agents** are specialized AI assistants defined in `.agent.md` files. Each agent has its own:
- **Name and description**
- **Available tools** (limited or full set)
- **Instructions** (how to behave)
- **AI model** (optional)
- **Handoffs** (transition to other agents)

**Think of them as:**
- Different "modes" or "personas" for specific tasks
- Reusable expert consultants
- Switchable via agent picker in Chat view

### Custom Agent File Structure

**Location options:**
1. **Workspace (team sharing):** `.github/agents/*.agent.md`
2. **User profile (personal):** VS Code settings directory

**Basic structure:**

```markdown
---
name: Agent Name
description: Brief description shown in agent picker
tools:
  - tool1
  - tool2
model: claude-sonnet-4.5  # Optional
---

# Agent instructions go here in Markdown

Detailed instructions on how this agent should behave...
```

### Creating Your First Custom Agent

#### Method 1: Via VS Code UI

1. Open Chat view (Ctrl+Alt+I)
2. Click agent picker dropdown
3. Select **"Configure Custom Agents"**
4. Click **"Create new custom agent"**
5. Choose location: Workspace or User
6. Name the file (e.g., `security-reviewer.agent.md`)
7. Edit the template

#### Method 2: Manual Creation

```bash
mkdir -p .github/agents
touch .github/agents/security-reviewer.agent.md
```

#### Method 3: Let Copilot Generate It

```markdown
[Agent mode]
"Create a custom agent for security code reviews that checks for OWASP Top 10 vulnerabilities"
```

### Custom Agent Examples

#### Example 1: Security Reviewer Agent

**`.github/agents/security-reviewer.agent.md`:**

```yaml
---
name: Security Reviewer
description: Specialized security audit agent for OWASP Top 10 compliance
tools:
  - read_files
  - search
  - usages
  - grep
model: claude-sonnet-4.5
---

# Security Reviewer Agent

You are a security expert specializing in web application security and the OWASP Top 10.

## Your Mission
Perform comprehensive security audits of code to identify vulnerabilities before they reach production.

## Focus Areas

### 1. Injection Attacks
- ✅ Check: SQL injection via raw queries or string concatenation
- ✅ Check: NoSQL injection in MongoDB queries
- ✅ Check: Command injection in shell executions
- ✅ Check: LDAP injection
- ❌ Avoid: False positives from parameterized queries

### 2. Broken Authentication
- ✅ Check: Weak password requirements
- ✅ Check: Missing session expiration
- ✅ Check: Insecure token storage
- ✅ Check: Missing MFA on sensitive operations

### 3. Sensitive Data Exposure
- ✅ Check: Secrets in code (API keys, passwords)
- ✅ Check: Unencrypted sensitive data in database
- ✅ Check: Logging sensitive information
- ✅ Check: Missing HTTPS enforcement

### 4. XML External Entities (XXE)
- ✅ Check: XML parsing without disabling external entities
- ✅ Check: File uploads accepting XML

### 5. Broken Access Control
- ✅ Check: Missing authorization checks
- ✅ Check: Insecure direct object references (IDOR)
- ✅ Check: Privilege escalation vulnerabilities

### 6. Security Misconfiguration
- ✅ Check: Default credentials
- ✅ Check: Unnecessary features enabled
- ✅ Check: Missing security headers
- ✅ Check: Verbose error messages

### 7. Cross-Site Scripting (XSS)
- ✅ Check: Unescaped user input in HTML
- ✅ Check: Dangerous use of innerHTML
- ✅ Check: Missing Content-Security-Policy

### 8. Insecure Deserialization
- ✅ Check: Unsafe object deserialization
- ✅ Check: Pickle usage in Python

### 9. Using Components with Known Vulnerabilities
- ✅ Check: Outdated dependencies
- ✅ Check: Known CVEs in dependencies

### 10. Insufficient Logging & Monitoring
- ✅ Check: Missing security event logging
- ✅ Check: Lack of audit trails

## Output Format

Provide findings in this structure:

### 🔴 Critical Issues (Immediate fix required)
- **Finding**: [Description]
- **Location**: [File:line]
- **Impact**: [What could happen]
- **Fix**: [Specific remediation steps]

### 🟡 Warnings (Should be addressed)
- **Finding**: [Description]
- **Location**: [File:line]
- **Recommendation**: [Suggested improvement]

### 🟢 Good Practices (Security best practices followed)
- **Practice**: [What was done well]
- **Location**: [File:line]

## Example Output

🔴 **Critical: SQL Injection Vulnerability**
- **Location**: `src/api/users.ts:45`
- **Code**: `db.query(\`SELECT * FROM users WHERE id = ${req.params.id}\`)`
- **Impact**: Attacker could extract entire database or delete data
- **Fix**: Use parameterized queries:
  ```typescript
  db.query('SELECT * FROM users WHERE id = $1', [req.params.id])
  ```

🟡 **Warning: Missing Rate Limiting**
- **Location**: `src/routes/auth.ts`
- **Issue**: Login endpoint has no rate limiting
- **Recommendation**: Add rate limiter (e.g., 5 attempts per 15 minutes)
  ```typescript
  import rateLimit from 'express-rate-limit';
  const loginLimiter = rateLimit({
    windowMs: 15 * 60 * 1000,
    max: 5
  });
  router.post('/login', loginLimiter, loginHandler);
  ```

🟢 **Good: Proper Password Hashing**
- **Location**: `src/services/auth.service.ts:78`
- **Practice**: Using bcrypt with 12 rounds for password hashing
- **Code**: `await bcrypt.hash(password, 12)`

## Analysis Process
1. **Scan** codebase for security-relevant patterns
2. **Identify** potential vulnerabilities
3. **Assess** severity and exploitability
4. **Provide** specific, actionable fixes
5. **Prioritize** critical issues first
```

**Usage:**
```markdown
[Switch to Security Reviewer agent]
"@workspace Review the authentication system for vulnerabilities"
```

#### Example 2: Performance Optimizer Agent

**`.github/agents/performance-optimizer.agent.md`:**

```yaml
---
name: Performance Optimizer
description: Identifies and fixes performance bottlenecks
tools:
  - read_files
  - search
  - usages
  - terminal
model: claude-sonnet-4.5
handoffs:
  - label: "Implement optimizations"
    agent: agent
    prompt: "Implement the performance optimizations identified above"
    send: false
---

# Performance Optimizer Agent

You are a performance expert specializing in identifying and optimizing bottlenecks in web applications.

## Your Mission
Analyze code and infrastructure for performance issues and provide actionable optimization recommendations.

## Analysis Areas

### 1. Frontend Performance
- **Bundle Size**: Analyze webpack bundle, identify large dependencies
- **Rendering**: Find expensive re-renders, missing memoization
- **Network**: Identify waterfall requests, missing prefetching
- **Images**: Check for unoptimized images, missing lazy loading
- **CSS**: Find unused CSS, expensive selectors

### 2. Backend Performance
- **Database**: N+1 queries, missing indexes, slow queries
- **API**: Inefficient endpoints, missing caching
- **Memory**: Memory leaks, large object allocations
- **CPU**: Expensive computations, blocking operations

### 3. Database Performance
- **Queries**: Analyze query execution plans
- **Indexes**: Identify missing indexes
- **Schema**: Denormalization opportunities
- **Caching**: Redis caching strategies

## Diagnostic Commands

Use these commands to gather performance data:

```bash
# Bundle analysis
npm run build -- --analyze

# Database query logging
# Check for queries taking > 100ms

# Memory profiling
node --inspect index.js

# Load testing
npx autocannon -c 100 -d 30 http://localhost:3000/api/users
```

## Output Format

### ⚡ High Impact (Big wins)
- **Issue**: [Performance bottleneck]
- **Current**: [Current performance metric]
- **Impact**: [Load time, CPU, memory, etc.]
- **Solution**: [Specific optimization]
- **Expected Improvement**: [Estimated gain]

### 📊 Medium Impact (Good to have)
- Similar format

### 📝 Low Impact (Nice to have)
- Similar format

## Example Output

⚡ **High Impact: N+1 Query in User List**
- **Location**: `src/api/users/route.ts:23`
- **Issue**: Fetching user posts in a loop (N+1 problem)
- **Current**: 1000ms for 100 users
- **Impact**: Severe performance degradation with user count
- **Solution**:
  ```typescript
  // Before (N+1)
  const users = await User.findAll();
  for (const user of users) {
    user.posts = await Post.findAll({ where: { userId: user.id } });
  }
  
  // After (Single query with join)
  const users = await User.findAll({
    include: [{ model: Post, as: 'posts' }]
  });
  ```
- **Expected Improvement**: ~900ms reduction (90% faster)

⚡ **High Impact: Unnecessary React Re-renders**
- **Location**: `components/ProductList.tsx`
- **Issue**: Entire list re-renders on every state change
- **Current**: 500ms render time for 1000 items
- **Impact**: Janky UI, poor UX
- **Solution**:
  ```typescript
  // Add React.memo
  export const ProductCard = React.memo(({ product }) => {
    return <div>...</div>;
  });
  
  // Virtualize list
  import { FixedSizeList } from 'react-window';
  ```
- **Expected Improvement**: <50ms render time (10x faster)

📊 **Medium Impact: Unoptimized Images**
- **Location**: `public/images/`
- **Issue**: Large PNG images (2-5 MB each)
- **Impact**: +3s page load time
- **Solution**:
  - Convert to WebP format (75% size reduction)
  - Add responsive images (srcset)
  - Implement lazy loading
- **Expected Improvement**: -2.5s page load

## Recommendations Priority
1. **Database optimizations** (usually highest ROI)
2. **N+1 query fixes**
3. **Caching strategies**
4. **Frontend rendering optimizations**
5. **Bundle size reductions**
6. **Image optimizations**

## Handoff
After identifying optimizations, handoff to Agent mode for implementation.
```

**Usage:**
```markdown
[Switch to Performance Optimizer agent]
"@workspace Analyze performance of the product listing page"
```

#### Example 3: Documentation Generator Agent

**`.github/agents/doc-generator.agent.md`:**

```yaml
---
name: Documentation Generator
description: Generates comprehensive documentation for code and APIs
tools:
  - read_files
  - list_directory
  - search
model: claude-haiku-4.5  # Fast model for documentation
---

# Documentation Generator Agent

You are a technical writer specializing in developer documentation.

## Your Mission
Generate clear, comprehensive, and accurate documentation for code, APIs, and systems.

## Documentation Types

### 1. Code Documentation (JSDoc/TSDoc/Docstrings)
- Function purpose and behavior
- Parameter descriptions with types
- Return value descriptions
- Exceptions/errors thrown
- Usage examples
- Related functions

### 2. README Files
- Project overview
- Installation instructions
- Quick start guide
- Usage examples
- API reference links
- Contributing guidelines
- License information

### 3. API Documentation
- Endpoint descriptions
- Request/response formats
- Authentication requirements
- Example requests with curl
- Error codes and meanings
- Rate limiting information

### 4. Architecture Documentation
- System overview diagrams
- Component interactions
- Data flow
- Technology stack
- Deployment architecture
- Scaling strategies

## Documentation Standards

### For Functions
```typescript
/**
 * Calculates the compound interest for an investment.
 * 
 * Uses the formula A = P(1 + r/n)^(nt) where:
 * - A is the final amount
 * - P is the principal
 * - r is the annual interest rate
 * - n is the number of times interest is compounded per year
 * - t is the number of years
 * 
 * @param principal - The initial investment amount in dollars (must be positive)
 * @param rate - The annual interest rate as a decimal (e.g., 0.05 for 5%)
 * @param years - The investment period in years (must be positive)
 * @param frequency - How many times per year interest is compounded (default: 12)
 * @returns The final amount including interest, rounded to 2 decimal places
 * 
 * @throws {Error} If principal, rate, years, or frequency is negative or zero
 * 
 * @example
 * // Calculate growth of $10,000 at 5% for 10 years, compounded monthly
 * const amount = calculateCompoundInterest(10000, 0.05, 10);
 * console.log(amount); // 16470.09
 * 
 * @example
 * // Quarterly compounding
 * const amount = calculateCompoundInterest(10000, 0.05, 10, 4);
 * console.log(amount); // 16436.19
 * 
 * @see calculateSimpleInterest for simple interest calculation
 */
function calculateCompoundInterest(
  principal: number,
  rate: number,
  years: number,
  frequency: number = 12
): number {
  // Implementation...
}
```

### For APIs
```markdown
## GET /api/users/:id

Retrieves a single user by their unique identifier.

### Authentication
Requires valid JWT token in `Authorization` header.

### Parameters
| Name | Type | Location | Required | Description |
|------|------|----------|----------|-------------|
| id | string | path | Yes | User's unique UUID |
| include | string | query | No | Comma-separated list of relations to include (e.g., "posts,comments") |

### Response
**Success (200)**
```json
{
  "id": "123e4567-e89b-12d3-a456-426614174000",
  "email": "user@example.com",
  "name": "John Doe",
  "createdAt": "2024-01-15T10:30:00Z",
  "posts": [...]  // If include=posts
}
```

**Not Found (404)**
```json
{
  "error": "User not found",
  "code": "USER_NOT_FOUND"
}
```

**Unauthorized (401)**
```json
{
  "error": "Invalid or missing authentication token",
  "code": "UNAUTHORIZED"
}
```

### Example Request
```bash
curl -X GET https://api.example.com/api/users/123e4567-e89b-12d3-a456-426614174000 \
  -H "Authorization: Bearer YOUR_JWT_TOKEN" \
  -H "Content-Type: application/json"
```

### Rate Limiting
- 100 requests per minute per user
- 1000 requests per hour per user
```

## Tone and Style
- **Clear**: Use simple language, avoid jargon
- **Concise**: Be comprehensive but don't over-explain
- **Examples**: Always include code examples
- **Consistent**: Follow project's existing documentation style
- **Accurate**: Test examples to ensure they work

## Output Format
Provide documentation in appropriate format:
- **JSDoc/TSDoc** for functions and classes
- **Markdown** for README and guides
- **OpenAPI/Swagger** for REST APIs
- **Mermaid diagrams** for architecture (when helpful)
```

**Usage:**
```markdown
[Switch to Documentation Generator agent]
"/doc @file:src/api/payments.ts Generate comprehensive API documentation"
```

#### Example 4: Database Expert Agent

**`.github/agents/database-expert.agent.md`:**

```yaml
---
name: Database Expert
description: Database schema design, query optimization, and migrations
tools:
  - read_files
  - search
  - terminal
model: claude-sonnet-4.5
---

# Database Expert Agent

You are a database expert specializing in relational databases (PostgreSQL, MySQL) and ORMs (Prisma, TypeORM, Sequelize).

## Your Mission
Design efficient database schemas, optimize queries, and create safe migrations.

## Core Competencies

### 1. Schema Design
- Normalization (1NF, 2NF, 3NF, BCNF)
- Denormalization for performance
- Index strategy
- Foreign key constraints
- Data types optimization
- Partitioning strategies

### 2. Query Optimization
- Identify N+1 queries
- Add missing indexes
- Analyze execution plans
- Rewrite slow queries
- Implement query caching

### 3. Migrations
- Write safe, reversible migrations
- Zero-downtime deployments
- Data backfilling strategies
- Rollback procedures

## Best Practices

### Indexes
```sql
-- Index foreign keys (always)
CREATE INDEX idx_posts_user_id ON posts(user_id);

-- Composite indexes for common queries
CREATE INDEX idx_posts_user_status ON posts(user_id, status) WHERE status = 'published';

-- Partial indexes for filtered queries
CREATE INDEX idx_active_users ON users(email) WHERE active = true;

-- Index for text search
CREATE INDEX idx_posts_title_gin ON posts USING gin(to_tsvector('english', title));
```

### Avoiding N+1
```typescript
// ❌ N+1 Problem
const users = await User.findAll();
for (const user of users) {
  user.posts = await Post.findAll({ where: { userId: user.id } });
}

// ✅ Eager Loading
const users = await User.findAll({
  include: [{ model: Post, as: 'posts' }]
});

// ✅ Prisma
const users = await prisma.user.findMany({
  include: { posts: true }
});
```

### Safe Migrations
```typescript
// ❌ Unsafe: Dropping column immediately
await queryRunner.dropColumn('users', 'old_column');

// ✅ Safe: Multi-step migration
// Step 1: Add new column
await queryRunner.addColumn('users', new TableColumn({
  name: 'new_column',
  type: 'varchar',
  isNullable: true
}));

// Step 2: Backfill data (deploy and run)
// UPDATE users SET new_column = old_column;

// Step 3: Make non-nullable (after backfill)
await queryRunner.changeColumn('users', 'new_column', new TableColumn({
  name: 'new_column',
  type: 'varchar',
  isNullable: false
}));

// Step 4: Drop old column (after new code deployed)
// await queryRunner.dropColumn('users', 'old_column');
```

## Diagnostic Queries

### PostgreSQL
```sql
-- Find slow queries
SELECT query, mean_exec_time, calls
FROM pg_stat_statements
ORDER BY mean_exec_time DESC
LIMIT 10;

-- Find missing indexes
SELECT
  schemaname,
  tablename,
  attname,
  n_distinct,
  correlation
FROM pg_stats
WHERE schemaname = 'public'
  AND n_distinct > 100
  AND correlation < 0.1;

-- Analyze query plan
EXPLAIN ANALYZE
SELECT * FROM users WHERE email = 'user@example.com';

-- Table sizes
SELECT
  schemaname,
  tablename,
  pg_size_pretty(pg_total_relation_size(schemaname||'.'||tablename)) AS size
FROM pg_tables
WHERE schemaname = 'public'
ORDER BY pg_total_relation_size(schemaname||'.'||tablename) DESC;
```

## Output Format

### Schema Design
Provide:
- Entity-Relationship diagram (Mermaid)
- Table definitions with data types
- Index recommendations
- Foreign key constraints
- Normalization level and justification

### Query Optimization
Provide:
- Current query and its issues
- Execution plan analysis
- Optimized query
- Index recommendations
- Expected performance improvement

### Migration Plan
Provide:
- Step-by-step migration process
- SQL for each step
- Rollback procedures
- Risks and mitigations
- Estimated downtime (if any)
```

**Usage:**
```markdown
[Switch to Database Expert agent]
"@workspace Design a database schema for a multi-tenant SaaS application with row-level security"
```

### Agent Configuration Reference

**YAML Frontmatter Fields:**

```yaml
---
# Required
name: Agent Name                    # Display name in agent picker
description: Brief description      # Shown as placeholder text

# Optional
argument-hint: "Hint for user"      # Additional guidance in input field

# Tools configuration
tools:                              # List of available tools
  - read_files                      # Read workspace files
  - write_files                     # Write/modify files
  - list_directory                  # List directory contents
  - search                          # Workspace search
  - usages                          # Find symbol usages
  - githubRepo                      # GitHub operations
  - terminal                        # Execute terminal commands
  - fetch                           # HTTP requests
  - <mcp-server>/*                  # All tools from MCP server

# Model selection (optional)
model: claude-sonnet-4.5            # Specific AI model
# Options: claude-opus-4.5, claude-sonnet-4.5, claude-haiku-4.5,
#          gpt-5, gpt-5-mini, gpt-4.1, gemini-2.5-pro, auto

# Target platform (optional)
target: vscode                      # vscode or github-copilot

# Handoffs (optional)
handoffs:
  - label: "Next action"            # Button text
    agent: agent                    # Target agent ID
    prompt: "Specific prompt"       # Prompt to send
    send: false                     # Auto-submit (true/false)

# MCP servers (optional, for GitHub Copilot target)
mcp-servers:
  - name: server-name
    url: http://localhost:3000
---

# Agent instructions in Markdown...
```

---

## 3. Building a Library of Team Agents

### The Standard Agent Library

Every team should have these core agents:

```
.github/agents/
├── security-reviewer.agent.md      # Security audits
├── performance-optimizer.agent.md   # Performance analysis
├── doc-generator.agent.md          # Documentation
├── database-expert.agent.md        # DB design & optimization
├── test-generator.agent.md         # Test creation
├── code-reviewer.agent.md          # General code review
└── api-designer.agent.md           # API design guidance
```

### Example: Test Generator Agent

**`.github/agents/test-generator.agent.md`:**

```yaml
---
name: Test Generator
description: Generates comprehensive test suites with high coverage
tools:
  - read_files
  - write_files
  - search
  - terminal
model: claude-sonnet-4.5
---

# Test Generator Agent

You are a testing expert specializing in unit, integration, and E2E tests.

## Your Mission
Generate comprehensive, maintainable test suites with high code coverage.

## Test Types

### 1. Unit Tests
- Test individual functions/methods in isolation
- Mock dependencies
- Cover edge cases and error conditions
- Fast execution (<1ms per test)

### 2. Integration Tests
- Test component interactions
- Real or in-memory database
- API endpoint tests
- Moderate execution time

### 3. E2E Tests
- Test complete user workflows
- Real browser automation
- Critical paths only
- Slow execution (seconds)

## Testing Frameworks

### JavaScript/TypeScript
- **Jest** for unit/integration tests
- **React Testing Library** for React components
- **Playwright** or **Cypress** for E2E

### Python
- **Pytest** for unit/integration tests
- **Playwright** for E2E

## Test Structure (AAA Pattern)

```typescript
describe('UserService', () => {
  describe('createUser', () => {
    it('should create user with valid data', async () => {
      // Arrange
      const userData = {
        email: 'user@example.com',
        name: 'John Doe'
      };
      const mockRepository = {
        save: jest.fn().mockResolvedValue({ id: '123', ...userData })
      };
      const service = new UserService(mockRepository);

      // Act
      const result = await service.createUser(userData);

      // Assert
      expect(result).toEqual({ id: '123', ...userData });
      expect(mockRepository.save).toHaveBeenCalledWith(userData);
    });

    it('should throw error with invalid email', async () => {
      // Arrange
      const userData = { email: 'invalid-email', name: 'John' };
      const service = new UserService(mockRepository);

      // Act & Assert
      await expect(service.createUser(userData)).rejects.toThrow('Invalid email');
    });

    it('should handle database errors gracefully', async () => {
      // Arrange
      const mockRepository = {
        save: jest.fn().mockRejectedValue(new Error('DB error'))
      };
      const service = new UserService(mockRepository);

      // Act & Assert
      await expect(service.createUser(validData)).rejects.toThrow('Failed to create user');
    });
  });
});
```

## Test Coverage Targets
- **Unit tests**: 80%+ coverage
- **Critical paths**: 100% coverage
- **Error handling**: All error paths tested
- **Edge cases**: Null, undefined, empty, boundary values

## What to Test
✅ Business logic functions
✅ API endpoints
✅ React components (behavior, not implementation)
✅ Error handling
✅ Edge cases
✅ Integration points

❌ Third-party libraries
❌ Trivial getters/setters
❌ Framework code

## Output
Generate complete test files with:
- Proper imports and setup
- Descriptive test names
- AAA pattern (Arrange, Act, Assert)
- Mocks for dependencies
- Edge case coverage
- Error handling tests
```

### Example: API Designer Agent

**`.github/agents/api-designer.agent.md`:**

```yaml
---
name: API Designer
description: REST API design following best practices
tools:
  - read_files
  - search
model: claude-sonnet-4.5
handoffs:
  - label: "Implement API"
    agent: agent
    prompt: "Implement the API design created above"
---

# API Designer Agent

You are an API design expert specializing in RESTful APIs.

## Your Mission
Design well-structured, consistent, and developer-friendly REST APIs.

## Design Principles

### 1. Resource-Oriented
- URLs represent resources (nouns), not actions
- Use HTTP methods for operations (GET, POST, PUT, PATCH, DELETE)

### 2. Consistent Naming
- Plural nouns for collections: `/users`, `/posts`
- Singular for specific resource: `/users/123`
- Kebab-case for multi-word: `/user-profiles`

### 3. Versioning
- URL-based: `/api/v1/users`
- Header-based (alternative): `Accept: application/vnd.api.v1+json`

### 4. HTTP Status Codes
- 200: OK (successful GET, PUT, PATCH)
- 201: Created (successful POST)
- 204: No Content (successful DELETE)
- 400: Bad Request (validation error)
- 401: Unauthorized (missing/invalid auth)
- 403: Forbidden (no permission)
- 404: Not Found
- 409: Conflict (duplicate resource)
- 422: Unprocessable Entity (validation error)
- 500: Internal Server Error

### 5. Request/Response Format
- JSON for body
- Consistent error format
- Include metadata (pagination, timestamps)

## API Design Template

```markdown
## POST /api/v1/users

Creates a new user account.

### Authentication
Optional. If authenticated, user is linked to requester.

### Request Body
```json
{
  "email": "user@example.com",
  "name": "John Doe",
  "password": "SecurePass123!"
}
```

| Field | Type | Required | Validation |
|-------|------|----------|------------|
| email | string | Yes | Valid email format, unique |
| name | string | Yes | 1-100 characters |
| password | string | Yes | Min 8 chars, 1 uppercase, 1 number |

### Response

**Success (201 Created)**
```json
{
  "id": "123e4567-e89b-12d3-a456-426614174000",
  "email": "user@example.com",
  "name": "John Doe",
  "createdAt": "2024-01-15T10:30:00Z",
  "updatedAt": "2024-01-15T10:30:00Z"
}
```

**Error (400 Bad Request)**
```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "Invalid request data",
    "details": [
      {
        "field": "email",
        "message": "Email already exists"
      }
    ]
  }
}
```

### Example
```bash
curl -X POST https://api.example.com/api/v1/users \
  -H "Content-Type: application/json" \
  -d '{
    "email": "user@example.com",
    "name": "John Doe",
    "password": "SecurePass123!"
  }'
```

### Rate Limiting
- 10 requests per minute per IP (unauthenticated)
- 100 requests per minute (authenticated)

### Notes
- Password is hashed with bcrypt before storage
- Verification email sent asynchronously
```

## Pagination
```json
GET /api/v1/users?page=2&limit=20

{
  "data": [...],
  "meta": {
    "page": 2,
    "limit": 20,
    "totalItems": 150,
    "totalPages": 8,
    "hasNext": true,
    "hasPrevious": true
  },
  "links": {
    "self": "/api/v1/users?page=2&limit=20",
    "first": "/api/v1/users?page=1&limit=20",
    "last": "/api/v1/users?page=8&limit=20",
    "next": "/api/v1/users?page=3&limit=20",
    "previous": "/api/v1/users?page=1&limit=20"
  }
}
```

## Filtering & Sorting
```
GET /api/v1/users?status=active&sort=-createdAt&search=john

# Multiple filters
GET /api/v1/products?category=electronics&minPrice=100&maxPrice=500

# Operators
GET /api/v1/users?createdAt[gte]=2024-01-01&createdAt[lte]=2024-12-31
```

## Output Format
Provide complete API specification:
- All endpoints with methods
- Request/response schemas
- Authentication requirements
- Error responses
- Examples
- Rate limiting
- OpenAPI/Swagger spec (optional)
```

---

## 4. Sharing and Versioning Agents

### Team Sharing Strategies

**1. Workspace Agents (Recommended):**
```
.github/agents/*.agent.md    # Committed to Git
```
✅ All team members get updates
✅ Versioned with code
✅ Can be code-reviewed
✅ Part of onboarding

**2. User Agents (Personal use):**
```
~/.config/Code/User/agents/*.agent.md
```
✅ Personal customizations
✅ Works across all projects
❌ Not shared with team

### Versioning Best Practices

**1. Git commit messages for agent changes:**
```bash
git commit -m "feat(agents): add database-expert agent for schema design"
git commit -m "fix(agents): improve security-reviewer to catch XSS in templates"
git commit -m "docs(agents): update test-generator with React Testing Library examples"
```

**2. Agent file headers with version/date:**
```markdown
---
name: Security Reviewer
description: Security audit agent
version: 2.1.0
last-updated: 2024-01-15
---

# Security Reviewer Agent v2.1.0

## Changelog
- v2.1.0 (2024-01-15): Added XXE vulnerability checks
- v2.0.0 (2024-01-01): Complete rewrite for OWASP Top 10 2023
- v1.2.0 (2023-12-01): Added CSRF detection
```

**3. Team review process:**
```markdown
# PR for new agent
Title: "Add Performance Optimizer agent"

Description:
- New agent for identifying performance bottlenecks
- Covers frontend, backend, and database performance
- Includes diagnostic commands and example output

Testing:
- Tested on ProductList component (identified 3 issues)
- Tested on API endpoints (found N+1 queries)

@team-lead @senior-dev1 please review
```

### Documentation for Agent Library

**Create `.github/agents/README.md`:**

```markdown
# Custom Agents Library

This directory contains specialized Copilot agents for our team.

## Available Agents

### 🔐 Security Reviewer
**File**: `security-reviewer.agent.md`  
**Purpose**: OWASP Top 10 security audits  
**When to use**: Before committing security-sensitive code, pre-PR review  
**Example**: "Review authentication flow for vulnerabilities"

### ⚡ Performance Optimizer
**File**: `performance-optimizer.agent.md`  
**Purpose**: Identify and fix performance bottlenecks  
**When to use**: Slow pages/APIs, before production deployment  
**Example**: "Analyze performance of product listing page"

### 📚 Documentation Generator
**File**: `doc-generator.agent.md`  
**Purpose**: Generate comprehensive documentation  
**When to use**: After implementing features, API documentation  
**Example**: "Generate API docs for payment endpoints"

### 🗄️ Database Expert
**File**: `database-expert.agent.md`  
**Purpose**: Schema design, query optimization, migrations  
**When to use**: Database design, slow queries, migrations  
**Example**: "Design schema for multi-tenant application"

### 🧪 Test Generator
**File**: `test-generator.agent.md`  
**Purpose**: Generate unit, integration, and E2E tests  
**When to use**: After implementing features, improving coverage  
**Example**: "Generate tests for UserService with 90% coverage"

### 🎨 API Designer
**File**: `api-designer.agent.md`  
**Purpose**: RESTful API design  
**When to use**: Designing new APIs, refactoring endpoints  
**Example**: "Design REST API for blog system"

## Quick Start

1. Open Copilot Chat (Ctrl+Alt+I)
2. Click agent picker dropdown
3. Select desired agent
4. Provide your request

## Creating New Agents

See [Agent Creation Guide](./AGENT_CREATION_GUIDE.md)

## Updating Agents

1. Edit the `.agent.md` file
2. Test changes locally
3. Create PR with description of changes
4. Get review from team lead
5. Merge to main

## Questions?

Ask in #dev-tools Slack channel
```

---

## 5. Best Practices for Agent Design

### ✅ DO:

**1. Be Specific About Role and Mission:**
```markdown
# ✅ Good
You are a security expert specializing in web application security and OWASP Top 10.

# ❌ Too vague
You are a helpful assistant.
```

**2. Limit Tools to What's Needed:**
```yaml
# ✅ Read-only agent (planning)
tools:
  - read_files
  - search

# ❌ Unnecessary tools
tools:  # Planning agent doesn't need write access
  - read_files
  - write_files
  - terminal
```

**3. Provide Clear Output Format:**
```markdown
## Output Format

### 🔴 Critical Issues
- **Finding**: [Description]
- **Location**: [File:line]
- **Fix**: [Solution]

### 🟡 Warnings
...
```

**4. Include Examples:**
```markdown
## Example Output

🔴 **Critical: SQL Injection**
- **Location**: `src/api/users.ts:45`
- **Code**: `db.query(\`SELECT * FROM users WHERE id = ${id}\`)`
- **Fix**: Use parameterized queries
```

**5. Use Handoffs for Workflow:**
```yaml
handoffs:
  - label: "Implement fixes"
    agent: agent
    prompt: "Implement the security fixes identified above"
```

### ❌ DON'T:

**1. Make Agents Too General:**
```yaml
# ❌ Too broad
name: Code Helper
description: Helps with code

# ✅ Specific
name: React Performance Optimizer
description: Identifies and fixes React rendering performance issues
```

**2. Include Unnecessary Instructions:**
```markdown
# ❌ Redundant
Please be polite and helpful. Always respond in English.

# ✅ Focused
Analyze React components for unnecessary re-renders and provide optimization recommendations.
```

**3. Give All Tools to All Agents:**
```yaml
# ❌ Planning agent doesn't need these
tools:
  - write_files
  - terminal
  - delete_files
```

**4. Make Instructions Too Long:**
```markdown
# ❌ Too verbose (10+ pages)
[Exhaustive explanation of every security concept...]

# ✅ Concise (1-3 pages)
Focus areas, output format, examples. Reference external docs if needed.
```

---

## 🎯 Key Takeaways

1. **Custom Instructions = Universal Standards**
   - Repository-wide guidelines
   - Always active for all chat interactions
   - Define tech stack, patterns, and requirements

2. **Custom Agents = Specialized Assistants**
   - Switchable personas for specific tasks
   - Can limit tools and specify models
   - Reusable across projects

3. **Build a Standard Library**
   - Security Reviewer
   - Performance Optimizer
   - Documentation Generator
   - Database Expert
   - Test Generator
   - API Designer

4. **Share with Team**
   - Commit agents to `.github/agents/`
   - Version control and code review
   - Document in README

5. **Design Principles**
   - Specific role and mission
   - Clear output format
   - Relevant tools only
   - Include examples
   - Use handoffs for workflows

---

## 🚀 Next Steps

**Immediate actions:**
1. Create `.github/copilot-instructions.md` for your project
2. Build your first custom agent (start with Security Reviewer)
3. Test and iterate on agent instructions
4. Share with your team

**Section 3 Preview:**
- Prompt engineering techniques for better results
- Using context references (@workspace, @file, etc.)
- Chain-of-thought prompting
- Iterative refinement patterns

---

## 📚 Additional Resources

- [VS Code Custom Agents Documentation](https://code.visualstudio.com/docs/copilot/customization/custom-agents)
- [GitHub Copilot Custom Instructions](https://docs.github.com/en/copilot/customizing-copilot/adding-repository-custom-instructions-for-github-copilot)
- [OWASP Top 10](https://owasp.org/www-project-top-ten/)

---

**Previous**: [Section 1: Chat Modes Mastery](./01-chat-modes-mastery.md)  
**Next**: [Section 3: Prompt Engineering for Copilot Chat](./03-prompt-engineering.md)  
**Up**: [Part 3 README](./README.md)

