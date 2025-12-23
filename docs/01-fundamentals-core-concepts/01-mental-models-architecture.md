---
title: "Section 1: Mental Models & Architecture"
parent: "Part 1: Fundamentals & Core Concepts"
nav_order: 1
---

# Part 1, Section 1: Mental Models & Architecture

**Part of**: [VS Code & GitHub Copilot: Complete Technical Guide](../../README.md)  
**Estimated reading time**: 30 minutes  
**Prerequisites**: None - start here!

---

## 📋 Overview

Before diving into GitHub Copilot, you need to understand **how it thinks** and **how you should think about it**. This section builds the mental models that will make everything else click.

**What you'll learn:**
- The GitHub Copilot ecosystem and its components
- How different AI models work and when to use each
- The architecture of inline completions, chat, and agents
- How workspace context powers intelligent suggestions

**Why this matters:** Understanding these concepts prevents frustration and unlocks Copilot's full potential. You'll know *why* something works, not just *how* to use it.

---

## 🧠 The GitHub Copilot Ecosystem

### Three Interaction Modes

Think of GitHub Copilot as having **three distinct personalities**, each optimized for different tasks:

```mermaid
graph TB
    subgraph "GitHub Copilot Ecosystem"
        A["1️⃣ INLINE COMPLETIONS<br/>(As you type)"]
        B["2️⃣ CHAT INTERFACE<br/>(Conversational)"]
        C["3️⃣ AGENTS<br/>(Autonomous)"]
    end
    
    A --> A1["• Real-time<br/>• Tab to accept<br/>• Context-aware<br/>• GPT-4.1 model"]
    B --> B1["• Ask questions<br/>• Explain code<br/>• Refactor<br/>• Multiple models<br/>• Multi-turn"]
    C --> C1["• Plan<br/>• Edit<br/>• Ask<br/>• Agent<br/>• Subagents"]
    
    style A fill:#e1f5ff
    style B fill:#fff4e1
    style C fill:#f0e1ff
```

### 1️⃣ Inline Completions: Your Coding Copilot

**What it is:** Real-time code suggestions as you type, like an extremely intelligent autocomplete.

**How it works:**
- Analyzes your current file, cursor position, and surrounding code
- Predicts what you're likely to write next
- Shows suggestions in ghost text (gray text)
- Accept with `Tab`, reject with `Esc`

**Powered by:** GPT-4.1 (optimized for code completions, trained on high-quality GitHub repositories)

**Best for:**
- ✅ Writing function implementations
- ✅ Completing boilerplate code
- ✅ Generating repetitive patterns
- ✅ Writing tests following existing patterns
- ✅ Implementing standard algorithms

**Example:**

```typescript
// You type this function signature:
function calculateCompoundInterest(principal: number, rate: number, years: number) {
  // Copilot suggests the entire implementation:
  const amount = principal * Math.pow(1 + rate, years);
  return amount - principal;
}
```

**Mental model:** Think of it as **pair programming with someone who's seen millions of code examples** and knows what usually comes next.

---

### 2️⃣ Chat Interface: Your AI Pair Programmer

**What it is:** Conversational AI assistant that understands your entire codebase and can explain, refactor, and generate code through dialogue.

**Three ways to access it:**

| Interface | Shortcut | Best For | Context |
|-----------|----------|----------|---------|
| **Quick Chat** | `Ctrl+Shift+Alt+L` | Fast questions, syntax help | Current file |
| **Inline Chat** | `Ctrl+I` | Edit current file, refactor | Current selection |
| **Chat View** | `Ctrl+Alt+I` | Complex features, planning | Full workspace |

**Powered by:** Multiple AI models (you can choose or use Auto selection)

**Best for:**
- ✅ Understanding unfamiliar code
- ✅ Planning feature implementation
- ✅ Refactoring complex logic
- ✅ Debugging issues
- ✅ Generating documentation
- ✅ Code reviews

**Example:**

```
You: "@workspace How does authentication work in this app?"

Copilot: "Based on your workspace, authentication uses JWT tokens:

1. User logs in via POST /api/auth/login (src/routes/auth.ts)
2. Server validates credentials (src/services/auth-service.ts)
3. JWT token generated with 24h expiry (src/utils/jwt.ts)
4. Token stored in httpOnly cookie
5. Protected routes verify token via middleware (src/middleware/auth.ts)

The flow uses bcrypt for password hashing and includes rate limiting."
```

**Mental model:** Think of it as **having a senior developer who's already read your entire codebase** and can answer questions or help implement features.

---

### 3️⃣ Agents: Your Autonomous Development Team

**What it is:** AI agents that can autonomously plan, read files, make changes, and execute multi-step tasks across your entire workspace.

**Four built-in agent modes:**

```mermaid
graph LR
    A["ASK Mode<br/>━━━━━━<br/>Question<br/>& explain<br/><br/>Read-only"] 
    B["EDIT Mode<br/>━━━━━━<br/>Single<br/>file edit<br/><br/>Moderate<br/>autonomy"]
    C["PLAN Mode<br/>━━━━━━<br/>Strategy<br/>& roadmap<br/><br/>Read-only<br/>+ planning"]
    D["AGENT Mode<br/>━━━━━━<br/>Multi-file<br/>changes<br/><br/>Full<br/>autonomy"]
    
    A -->|Simple| B -->|More Complex| C -->|Most Complex| D
    
    style A fill:#d4edda
    style B fill:#fff3cd
    style C fill:#cfe2ff
    style D fill:#f8d7da
```

**Agent mode details:**

| Mode | Autonomy | Capabilities | Use Case Example |
|------|----------|--------------|------------------|
| **Ask** | Read-only | Answers questions, explains code | "Explain this authentication flow" |
| **Edit** | Single file | Targeted code changes | "Add error handling to this function" |
| **Plan** | Read-only | Creates implementation roadmap | "Plan OAuth2 implementation strategy" |
| **Agent** | Full | Multi-file changes, testing, refactoring | "Implement OAuth2 across all controllers" |

**Subagents:** For complex tasks, agents can spawn **subagents** to handle subtasks in isolation, preventing context pollution.

**Mental model:** Think of it as **having a junior developer you can delegate complete features to**, with the ability to read your codebase, make changes, and even test their work.

---

## 🤖 AI Model Family: Choose Your Brain

GitHub Copilot supports **multiple AI models** with different strengths. Understanding them helps you work smarter and faster.

### ⭐ The Easy Choice: AUTO Mode

**Recommendation for 90% of tasks:** Use **Auto** and let Copilot choose the best model.

**Auto mode automatically selects based on:**
- Task complexity
- Available models and rate limits
- Cost optimization
- Performance requirements

**When to use Auto:**
- ✅ You're unsure which model to choose
- ✅ You want automatic cost optimization
- ✅ You want to avoid rate limiting
- ✅ You trust Copilot to make the right choice

**When to manually select:**
- You need specific model capabilities (e.g., Gemini 2.5 Pro for massive context)
- You're comparing model outputs
- You have specific requirements ("must use Claude only")

---

### 🎯 The Model Lineup (December 2025)

**Quick reference table:**

| Model | Speed | Quality | Cost | Best For |
|-------|-------|---------|------|----------|
| **Auto** | Varies | Adaptive | Optimized | Let Copilot decide ⭐ |
| **Claude Opus 4.5** | Slower | Highest | Premium | Architecture, complex reasoning |
| **Claude Sonnet 4.5** | Medium | High | Standard | Multi-file work, daily coding ⭐ |
| **Claude Haiku 4.5** | Fastest | Good | Lower | Quick tasks, docs, boilerplate ⚡ |
| **GPT-5.1-Codex-Max** | Slow | Highest | Premium | Marathon coding sessions |
| **GPT-5** | Slow | Very High | Premium | Complex analysis, deep reasoning |
| **GPT-5 mini** | Fast | High | Standard | Fast deep reasoning |
| **GPT-4.1** | Fast | High | Included | Inline suggestions (default) |
| **Gemini 2.5 Pro** | Medium | High | Premium | Long context (1M+ tokens) |

---

### 📊 Model Deep Dive

#### 🏆 Claude Opus 4.5 - The Architect
```
Strengths: Deepest reasoning, complex problem-solving
Speed:     ⚫⚫⚪⚪⚪ (Slow but thorough)
Quality:   ⚫⚫⚫⚫⚫ (Highest)
Cost:      💰💰💰 (Premium)
```

**Use when:**
- Designing system architecture
- Solving complex algorithmic problems
- Debugging intricate issues
- Making critical technical decisions

**Example task:** "Design a scalable microservices architecture for a real-time trading platform with eventual consistency guarantees."

---

#### ⚖️ Claude Sonnet 4.5 - The Workhorse ⭐
```
Strengths: Balanced speed + quality, excellent multi-file context
Speed:     ⚫⚫⚫⚪⚪ (Medium)
Quality:   ⚫⚫⚫⚫⚪ (High)
Cost:      💰💰 (Standard)
```

**Use when:**
- Daily development work (most common choice)
- Multi-file refactoring
- Code reviews
- Understanding large codebases
- Implementing features

**Why it's popular:** Perfect balance of speed and quality. Can switch between quick answers and deep problem-solving.

**Example task:** "Refactor the authentication module to use dependency injection and add comprehensive error handling."

---

#### ⚡ Claude Haiku 4.5 - The Sprinter
```
Strengths: Fastest responses, cost-effective, solid quality
Speed:     ⚫⚫⚫⚫⚫ (Fastest)
Quality:   ⚫⚫⚫⚪⚪ (Good)
Cost:      💰 (Lower)
```

**Use when:**
- Writing documentation
- Generating boilerplate
- Simple refactoring
- Quick questions
- Test generation
- Scaffolding code

**Cost saver:** Use this for 40-50% of daily tasks to significantly reduce costs.

**Example task:** "Generate JSDoc comments for all functions in this file" or "Create a Jest test suite for this utility module."

---

#### 🧠 GPT-5 - The Deep Thinker
```
Strengths: Step-by-step reasoning, complex analysis
Speed:     ⚫⚫⚪⚪⚪ (Slow)
Quality:   ⚫⚫⚫⚫⚫ (Very High)
Cost:      💰💰💰 (Premium)
```

**Use when:**
- Multi-step debugging
- Complex code analysis
- Understanding intricate systems
- Research tasks

**Example task:** "Analyze why this distributed system is experiencing race conditions and propose a comprehensive solution."

---

#### ⚡🧠 GPT-5 mini - Fast + Smart
```
Strengths: Fast responses with good reasoning
Speed:     ⚫⚫⚫⚫⚪ (Fast)
Quality:   ⚫⚫⚫⚫⚪ (High)
Cost:      💰💰 (Standard)
```

**Use when:**
- Need reasoning but want speed
- Daily development with quality
- Balanced performance

**Example task:** "Optimize this database query and explain the performance improvements."

---

#### 💪 GPT-5.1-Codex-Max - The Marathon Runner
```
Strengths: Optimized for long coding sessions, cybersecurity
Speed:     ⚫⚫⚪⚪⚪ (Slow for complex work)
Quality:   ⚫⚫⚫⚫⚫ (Highest for sustained work)
Cost:      💰💰💰 (Premium)
```

**Use when:**
- Implementing large features
- Security-critical code
- Extended development sessions
- Complex system implementation

**Example task:** "Implement a complete OAuth2 flow with PKCE, token rotation, and security hardening."

---

#### 📄 Gemini 2.5 Pro - The Context King
```
Strengths: Massive context window (1M+ tokens), advanced reasoning
Speed:     ⚫⚫⚫⚪⚪ (Medium)
Quality:   ⚫⚫⚫⚫⚪ (High)
Cost:      💰💰💰 (Premium)
```

**Use when:**
- Analyzing entire modules
- Reviewing large logs
- Understanding massive codebases
- Working with very long documents

**Example task:** "Analyze all API endpoints in this workspace and identify security vulnerabilities and performance bottlenecks."

---

#### 🎯 GPT-4.1 - The Inline Specialist
```
Strengths: Optimized for autocomplete, trained on quality GitHub code
Speed:     ⚫⚫⚫⚫⚪ (Fast)
Quality:   ⚫⚫⚫⚫⚪ (High for completions)
Cost:      ✅ Included in all plans
```

**Use when:**
- Inline code completions (automatic)
- Quick coding tasks
- General development

**Note:** This is the **default model for inline autocomplete** - you don't manually select it.

---

### 🎯 Model Selection Strategy

**Decision tree:**

```mermaid
graph TD
    START["Need to choose a model?"]
    
    START -->|Unsure?| AUTO["Use Auto ⭐<br/>(Recommended)"]
    START -->|Know what you need?| KNOW["Select specific model"]
    
    KNOW --> ARCH["Architecture/<br/>Complex design"]
    KNOW --> MULTI["Multi-file<br/>refactoring"]
    KNOW --> DAILY["Daily coding"]
    KNOW --> DOCS["Documentation/<br/>boilerplate"]
    KNOW --> DEBUG["Complex<br/>debugging"]
    KNOW --> QUICK["Quick<br/>questions"]
    KNOW --> LONG["Long context<br/>analysis"]
    KNOW --> MARATHON["Marathon<br/>feature dev"]
    
    ARCH --> ARCH_M["Claude Opus 4.5<br/>or GPT-5"]
    MULTI --> MULTI_M["Claude Sonnet 4.5 ⭐"]
    DAILY --> DAILY_M["Claude Sonnet 4.5<br/>or GPT-5 mini"]
    DOCS --> DOCS_M["Claude Haiku 4.5 ⚡"]
    DEBUG --> DEBUG_M["GPT-5 or<br/>Claude Opus 4.5"]
    QUICK --> QUICK_M["Claude Haiku 4.5 ⚡"]
    LONG --> LONG_M["Gemini 2.5 Pro"]
    MARATHON --> MARATHON_M["GPT-5.1-Codex-Max"]
    
    style AUTO fill:#90EE90,stroke:#2d862d,stroke-width:3px
    style MULTI_M fill:#90EE90
    style DOCS_M fill:#FFD700
    style QUICK_M fill:#FFD700
```

---

### 💰 Model Availability by Subscription

| Subscription | Model Access | Manual Selection | Default |
|--------------|--------------|------------------|---------|
| **Free** | Limited | ❌ No | GPT-4.1 (inline) |
| **Individual** ($10/mo) | Standard | ❌ No | GPT-4.1 (inline) |
| **Pro** ($20/mo) | All models | ✅ Yes | Auto (recommended) |
| **Pro+** ($40/mo) | All + priority | ✅ Yes | Auto (recommended) |
| **Business** ($19/user/mo) | Standard | Coming soon | Claude Sonnet 4.5 |
| **Enterprise** ($39/user/mo) | Standard | Coming soon | Claude Sonnet 4.5 |

**Cost optimization tip:** Premium models (Opus, GPT-5, Gemini Pro) incur additional per-request fees. Use Auto to let Copilot optimize costs automatically, or use Claude Haiku 4.5 for simple tasks.

---

## 🔄 Agent Architecture: Four Modes Explained

### Mental Model: Delegation Hierarchy

Think of agent modes as **different levels of delegation** you'd give to a team member:

```mermaid
graph TD
    A["ASK: 'Tell me about...'"] --> A1["No changes, just information"]
    B["EDIT: 'Fix this specific function'"] --> B1["Targeted changes to one file"]
    C["PLAN: 'How should we implement feature X?'"] --> C1["Strategic roadmap, no code changes yet"]
    D["AGENT: 'Implement feature X across the codebase'"] --> D1["Full autonomy, multi-file implementation"]
    
    style A fill:#d4edda
    style A1 fill:#d4edda,stroke:#28a745
    style B fill:#fff3cd
    style B1 fill:#fff3cd,stroke:#ffc107
    style C fill:#cfe2ff
    style C1 fill:#cfe2ff,stroke:#0d6efd
    style D fill:#f8d7da
    style D1 fill:#f8d7da,stroke:#dc3545
```

---

### Ask Mode: The Consultant

**When to use:**
- Understanding unfamiliar code
- Explaining architectural decisions
- Learning how systems work
- Getting recommendations

**Capabilities:**
- ✅ Read files across workspace
- ✅ Explain code and architecture
- ✅ Answer technical questions
- ❌ Cannot make changes

**Example interaction:**

```
You: "@workspace Explain how user permissions are enforced"

Ask mode response:
"Permission enforcement uses a middleware-based approach:

1. AuthMiddleware (src/middleware/auth.ts) verifies JWT token
2. PermissionsMiddleware (src/middleware/permissions.ts) checks user role
3. Role definitions in src/models/Role.ts (Admin, User, Guest)
4. Each route declares required permissions via decorator
5. Database queries filtered by user permissions via Prisma middleware

Critical files:
- src/middleware/permissions.ts (enforcement logic)
- src/decorators/require-permission.ts (route declarations)
- prisma/schema.prisma (Permission model)"
```

---

### Edit Mode: The Focused Developer

**When to use:**
- Refactoring a single function
- Adding features to one file
- Fixing bugs in specific code
- Targeted improvements

**Capabilities:**
- ✅ Read context from workspace
- ✅ Edit single file efficiently
- ✅ Suggest improvements
- ⚠️ Limited to focused changes

**Example interaction:**

```
You (in auth.ts): "Add comprehensive error handling and input validation"

Edit mode:
- Reads current function
- Adds try-catch blocks
- Validates all inputs with Zod schemas
- Handles edge cases
- Updates return types
- Makes changes directly to the file
```

---

### Plan Mode: The Architect

**When to use:**
- Starting complex features
- Refactoring large systems
- Strategic technical decisions
- Before making major changes

**Capabilities:**
- ✅ Read entire workspace
- ✅ Create step-by-step roadmap
- ✅ Identify dependencies
- ✅ Estimate complexity
- ❌ Does not make code changes

**Example interaction:**

```
You: "Plan implementation of two-factor authentication"

Plan mode response:
"## 2FA Implementation Plan

### Phase 1: Infrastructure (2 hours)
1. Add TOTP library (speakeasy)
2. Database migration for 2FA fields
3. Update User model with 2FA properties

### Phase 2: Backend API (3 hours)
1. POST /api/2fa/setup - Generate QR code
2. POST /api/2fa/verify - Verify TOTP code
3. POST /api/2fa/disable - Disable 2FA
4. Modify login flow to check 2FA

### Phase 3: Frontend UI (3 hours)
1. 2FA setup page with QR code
2. TOTP verification input
3. Backup codes display
4. Settings toggle

### Phase 4: Testing (2 hours)
1. Unit tests for TOTP generation/verification
2. Integration tests for API endpoints
3. E2E test for complete flow

Files to modify:
- prisma/schema.prisma
- src/routes/auth.ts
- src/services/two-factor-auth.ts (new)
- src/components/TwoFactorSetup.tsx (new)

Dependencies:
- speakeasy (TOTP library)
- qrcode (QR generation)

Estimated total: 10 hours"
```

**Pro tip:** Always use Plan mode first for complex features, review the plan, then switch to Agent mode for implementation.

---

### Agent Mode: The Autonomous Developer

**When to use:**
- Implementing complete features
- Multi-file refactoring
- Complex bug fixes
- System-wide changes

**Capabilities:**
- ✅ Read entire workspace
- ✅ Create and modify multiple files
- ✅ Execute multi-step tasks
- ✅ Generate tests
- ✅ Update documentation
- ✅ Self-correct if errors occur

**Example interaction:**

```
You: "Implement two-factor authentication as planned"

Agent mode:
1. ✅ Creates database migration
2. ✅ Installs dependencies (speakeasy, qrcode)
3. ✅ Creates TwoFactorAuthService class
4. ✅ Updates auth routes with 2FA endpoints
5. ✅ Modifies login flow
6. ✅ Creates React components
7. ✅ Generates unit tests
8. ✅ Generates integration tests
9. ✅ Updates API documentation
10. ✅ Reports completion

(Agent makes 15 file changes autonomously)
```

**Autonomy level:** High - Agent can make decisions, but you should review all changes.

---

### Subagents: Isolated Task Execution

**What are subagents?**
- Spawned by main agent for complex subtasks
- Run in isolated context (prevents pollution)
- Return results to main agent
- Automatically managed

**When agents use subagents:**
- Task has distinct phases
- Need to focus on specific module
- Prevent context overload
- Parallel execution possible

**Example:**

```mermaid
graph TD
    MAIN["Main Agent Task:<br/>Implement OAuth2 + payment integration"]
    
    MAIN --> SUB1["Subagent 1:<br/>OAuth2 implementation"]
    MAIN --> SUB2["Subagent 2:<br/>Payment integration"]
    
    SUB1 --> SUB1C["Focused context:<br/>just auth system"]
    SUB2 --> SUB2C["Focused context:<br/>just payment system"]
    
    SUB1C --> MERGE["Results merged<br/>by main agent"]
    SUB2C --> MERGE
    
    style MAIN fill:#0066cc,color:#fff,stroke:#003d7a,stroke-width:3px
    style SUB1 fill:#d4edda
    style SUB2 fill:#cfe2ff
    style MERGE fill:#fff3cd,stroke:#ffc107,stroke-width:2px
```

**Mental model:** Subagents are like **delegating subtasks to specialists** who focus only on their area.

---

## 🌍 Workspace Context Model

### What is Workspace Context?

**Definition:** All the information Copilot knows about your project to provide intelligent suggestions.

```mermaid
graph TB
    WC["WORKSPACE CONTEXT"]
    
    WC --> FS["📁 File Structure"]
    WC --> OF["📝 Open Files"]
    WC --> CF["🔧 Configuration"]
    WC --> GH["📜 Git History"]
    WC --> CI["📋 Custom Instructions"]
    WC --> DP["🔌 Dependencies"]
    
    FS --> FS1["• All directories<br/>• File relationships"]
    OF --> OF1["• Currently editing<br/>• Recently viewed"]
    CF --> CF1["• package.json<br/>• tsconfig.json<br/>• .env.example"]
    GH --> GH1["• Recent commits<br/>• Changed files"]
    CI --> CI1["• .github/copilot-<br/>  instructions.md"]
    DP --> DP1["• Imported libraries"]
    
    style WC fill:#e1f5ff,stroke:#0066cc,stroke-width:3px
    style FS fill:#d4edda
    style OF fill:#fff3cd
    style CF fill:#cfe2ff
    style GH fill:#f8d7da
    style CI fill:#e7d4f5
    style DP fill:#d1ecf1
```

---

### The @workspace Command

**Most powerful context reference:** `@workspace` gives Copilot access to your entire codebase.

**Without @workspace:**
```
You: "How does authentication work?"
Copilot: [Generic OAuth2 explanation from training data]
```

**With @workspace:**
```
You: "@workspace How does authentication work?"
Copilot: [Detailed explanation of YOUR specific implementation, 
         referencing actual files, your database schema, 
         your middleware, your configuration]
```

**What @workspace includes:**
- ✅ All files in workspace (respects .gitignore)
- ✅ File structure and organization
- ✅ Dependencies and imports
- ✅ Configuration files
- ✅ Custom instructions
- ✅ Recent Git changes
- ❌ Does NOT include node_modules, build artifacts

---

### Context Optimization

**Key insight:** Workspace context is FREE (included in subscription) but affects response speed.

**Optimization strategies:**

1. **Exclude unnecessary files:**
   ```
   # Add to .gitignore (Copilot respects it)
   node_modules/
   dist/
   build/
   .next/
   *.log
   ```

2. **Use .copilotignore for sensitive files:**
   ```
   # .copilotignore
   .env
   secrets/
   *.key
   *.pem
   ```

3. **Scope your questions:**
   ```
   ❌ Vague: "@workspace Find bugs"
   ✅ Specific: "@workspace Review src/auth/ for security issues"
   ```

4. **Workspace size guidelines:**
   - **Small** (<100 files): Optimal performance
   - **Medium** (100-1000 files): Good performance
   - **Large** (1000-10000 files): Slower, use scoped questions
   - **Massive** (10000+ files): Consider multi-root workspace

---

### Context References

**Other context references beyond @workspace:**

| Reference | Scope | Example |
|-----------|-------|---------|
| `@workspace` | Entire workspace | `@workspace Find all API endpoints` |
| `@file:path/to/file` | Specific file | `@file:src/auth.ts Explain this module` |
| `@selection` | Selected code | `@selection Refactor this function` |
| `@terminal` | Terminal output | `@terminal Why did this command fail?` |

---

## 🔌 MCP (Model Context Protocol) Integration

### What is MCP?

**Definition:** Model Context Protocol (MCP) is a standard protocol for connecting AI models to external tools and data sources.

```mermaid
graph TD
    GC["GitHub Copilot Chat"]
    
    GC --> MCP1["MCP Server 1:<br/>GitHub API"]
    GC --> MCP2["MCP Server 2:<br/>Database"]
    GC --> MCP3["MCP Server 3:<br/>Custom Tools"]
    
    MCP1 --> MCP1A["Read issues, PRs, repos"]
    MCP2 --> MCP2A["Query production data"]
    MCP3 --> MCP3A["Your domain-specific tools"]
    
    style GC fill:#0066cc,color:#fff,stroke:#003d7a,stroke-width:3px
    style MCP1 fill:#d4edda
    style MCP2 fill:#cfe2ff
    style MCP3 fill:#fff3cd
    style MCP1A fill:#d4edda,stroke:#28a745
    style MCP2A fill:#cfe2ff,stroke:#0d6efd
    style MCP3A fill:#fff3cd,stroke:#ffc107
```

**Why MCP matters:**
- Extends Copilot beyond code
- Connects to APIs, databases, external services
- Enables custom integrations
- Standardized protocol (works across AI tools)

**Example use cases:**
- Query GitHub issues directly from Chat
- Read production database for debugging
- Call internal APIs for documentation
- Integrate with Jira, Linear, etc.

**Configuration:** MCP servers are configured in VS Code settings and appear as tools in Chat.

---

## 📏 Token Limits & Context Windows

### Understanding Tokens

**What is a token?**
- Roughly 4 characters of text
- 100 tokens ≈ 75 words
- Code uses more tokens than prose

**Why it matters:** Each model has a maximum context window (how much it can "remember" in one conversation).

---

### Context Windows by Model (December 2025)

| Model | Context Window | Practical Meaning |
|-------|---------------|-------------------|
| **GPT-4.1** | ~128K tokens | ~96,000 words or ~3,000 lines of code |
| **GPT-5 / GPT-5 mini** | ~128-200K tokens | ~150,000 words or ~5,000 lines of code |
| **GPT-5.1-Codex-Max** | Extended | Optimized for long tasks |
| **Claude Sonnet 4.5** | ~200K tokens | ~150,000 words or ~5,000 lines of code |
| **Claude Opus 4.5** | ~200K tokens | ~150,000 words or ~5,000 lines of code |
| **Claude Haiku 4.5** | ~200K tokens | ~150,000 words or ~5,000 lines of code |
| **Gemini 2.5 Pro** | **~1M+ tokens** | ~750,000 words or ~25,000 lines of code |

**Visual comparison:**

```mermaid
graph LR
    subgraph "Context Window Sizes"
        GPT4["GPT-4.1<br/>128K tokens<br/>~3,000 lines"]
        GPT5["GPT-5 series<br/>128-200K tokens<br/>~5,000 lines"]
        CLAUDE["Claude series<br/>200K tokens<br/>~5,000 lines"]
        GEMINI["Gemini 2.5 Pro<br/>1M+ tokens<br/>~25,000 lines"]
    end
    
    GPT4 -.->|5x larger| GEMINI
    GPT5 -.->|5x larger| GEMINI
    CLAUDE -.->|5x larger| GEMINI
    
    style GPT4 fill:#fff3cd
    style GPT5 fill:#cfe2ff
    style CLAUDE fill:#d4edda
    style GEMINI fill:#90EE90,stroke:#2d862d,stroke-width:3px
```

**Winner for long context:** Gemini 2.5 Pro (1M+ tokens)

---

### Practical Implications

**What fits in context:**

**Small workspace (GPT-4.1, ~128K tokens):**
- ✅ 50-100 average-sized files
- ✅ Full conversation history
- ✅ All open files + workspace structure

**Medium workspace (Claude models, ~200K tokens):**
- ✅ 100-200 files
- ✅ Complex multi-file features
- ✅ Extensive conversation history

**Large workspace (Gemini 2.5 Pro, ~1M tokens):**
- ✅ Entire medium-sized codebase
- ✅ Multiple related projects
- ✅ Very long analysis tasks

**When you exceed context:**
- Older messages are automatically removed
- Use subagents for isolated tasks
- Scope questions with `@file:` instead of `@workspace`

---

## 🎯 Key Takeaways

### 1. Three Interaction Modes

```mermaid
graph LR
    IC["Inline Completions"]
    CH["Chat"]
    AG["Agents"]
    
    IC --> IC1["Real-time suggestions<br/>(Tab to accept)"]
    CH --> CH1["Conversational assistance<br/>(ask, explain, refactor)"]
    AG --> AG1["Autonomous development<br/>(plan and implement features)"]
    
    style IC fill:#e1f5ff
    style CH fill:#fff4e1
    style AG fill:#f0e1ff
```

**Rule of thumb:**

```mermaid
graph TD
    Q["What do you need?"]
    
    Q --> Q1["Suggestion while typing?"]
    Q --> Q2["Understand or refactor?"]
    Q --> Q3["Complete feature built?"]
    
    Q1 --> A1["→ Inline (automatic)"]
    Q2 --> A2["→ Chat"]
    Q3 --> A3["→ Agent"]
    
    style Q fill:#e1f5ff,stroke:#0066cc,stroke-width:2px
    style A1 fill:#d4edda
    style A2 fill:#fff3cd
    style A3 fill:#f8d7da
```

---

### 2. Agent Mode Hierarchy

```mermaid
graph LR
    ASK["Ask<br/>━━━<br/>Questions only<br/>(read-only)"]
    EDIT["Edit<br/>━━━<br/>Single file changes<br/>(targeted)"]
    PLAN["Plan<br/>━━━<br/>Strategy & roadmap<br/>(no changes)"]
    AGENT["Agent<br/>━━━<br/>Multi-file implementation<br/>(full autonomy)"]
    
    ASK --> EDIT --> PLAN --> AGENT
    
    style ASK fill:#d4edda
    style EDIT fill:#fff3cd
    style PLAN fill:#cfe2ff
    style AGENT fill:#f8d7da
```

**Best practice:** Plan first, then Agent. Review everything before committing.

---

### 3. Workspace Context is Free

- ✅ `@workspace` gives Copilot access to entire codebase
- ✅ No cost penalty (included in subscription)
- ✅ Automatically includes files, structure, config, Git history
- ⚠️ Affects response speed (optimize by excluding node_modules, build artifacts)

**Golden rule:** Always use `@workspace` for project-specific questions.

---

### 4. Model Selection Made Easy

```mermaid
graph TD
    START["Choose Model"]
    
    START --> AUTO["90% of time:<br/>Use AUTO ⭐<br/>(let Copilot choose)"]
    START --> MANUAL["Manual Selection"]
    
    MANUAL --> SPEED["Need speed:<br/>Claude Haiku 4.5 ⚡"]
    MANUAL --> DAILY["Daily work:<br/>Claude Sonnet 4.5"]
    MANUAL --> COMPLEX["Complex:<br/>Claude Opus 4.5<br/>or GPT-5"]
    MANUAL --> LONG["Long context:<br/>Gemini 2.5 Pro"]
    
    style AUTO fill:#90EE90,stroke:#2d862d,stroke-width:3px
    style SPEED fill:#FFD700
    style DAILY fill:#87CEEB
    style COMPLEX fill:#FFB6C1
    style LONG fill:#DDA0DD
```

**Don't overthink it:** Auto mode is excellent and optimizes automatically.

---

### 5. MCP Extends Copilot

- MCP = External tool integration
- Connects Copilot to APIs, databases, services
- Configured in VS Code settings
- Enables custom workflows

**Explore later:** You don't need MCP to start - it's advanced.

---

### 6. Token Limits Matter for Large Projects

- Most models: 128-200K tokens (~5,000 lines of code)
- Gemini 2.5 Pro: 1M+ tokens (massive context)
- Exceeded context? Use `@file:` instead of `@workspace`

---

## ⚠️ Common Pitfalls

### ❌ Pitfall 1: Not Using @workspace
**Problem:** Generic answers instead of project-specific ones  
**Solution:** Always start with `@workspace` for code questions

### ❌ Pitfall 2: Using Agent Mode for Simple Tasks
**Problem:** Overkill, slower, more expensive  
**Solution:** Use Inline Chat (Ctrl+I) for single-file edits

### ❌ Pitfall 3: Not Reviewing Generated Code
**Problem:** Bugs, security issues, incorrect logic  
**Solution:** **Always review and test** AI-generated code

### ❌ Pitfall 4: Vague Prompts
**Problem:** "Make this better" → generic suggestions  
**Solution:** "Add error handling, input validation, and logging with specific error messages"

### ❌ Pitfall 5: Wrong Model for Task
**Problem:** Using premium Claude Opus 4.5 for simple docs (costly)  
**Solution:** Use Auto, or Claude Haiku 4.5 for simple tasks

---

## 🚀 Next Steps

Now that you understand **how Copilot thinks**, you're ready to:

1. **[Part 1, Section 2: Environment & Project Setup](02-environment-project-setup.md)**
   - Install VS Code and Copilot extension
   - Configure workspace for optimal performance
   - Set up custom instructions

2. **[Quick Start: First 5 Minutes](../../README.md#-get-your-first-win-in-5-minutes)**
   - Get immediate productivity wins
   - Try inline completions, chat, and agents

3. **[Learning Paths](../../README.md#-learning-paths)**
   - Choose your journey: 15-minute quick start, 4-hour mastery, or team enablement

---

## 📚 References

- [GitHub Copilot Official Docs](https://docs.github.com/copilot)
- [VS Code Copilot Documentation](https://code.visualstudio.com/docs/copilot)
- [Model Context Protocol (MCP)](https://modelcontextprotocol.io)
- [VS Code Extension API](https://code.visualstudio.com/api)

---

**Next:** [Section 2: Environment & Project Setup →](02-environment-project-setup.md)

**Back to:** [Guide Home](../../README.md)

