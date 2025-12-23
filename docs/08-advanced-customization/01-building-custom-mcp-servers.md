---
title: "Section 1: Building Custom MCP Servers"
parent: "Part 8: Advanced Customization"
nav_order: 1
---

# Building Custom MCP Servers

**Part 8, Section 1: Advanced Customization**  
**Target:** Developers building custom integrations  
**Time to complete:** 2-3 hours

---

## 📋 Overview

The Model Context Protocol (MCP) enables you to extend GitHub Copilot with custom tools, data sources, and capabilities. This section teaches you how to build production-ready MCP servers that integrate seamlessly with Copilot Chat in VS Code.

**What you'll learn:**
- MCP architecture and protocol fundamentals
- Building custom MCP servers from scratch
- Integrating external APIs and data sources
- Security best practices for MCP servers
- Testing and debugging MCP integrations
- Deploying and maintaining MCP servers

---

## 🏗️ MCP Architecture Overview

### What is MCP?

The Model Context Protocol (MCP) is an open protocol that standardizes how AI assistants connect to external data sources and tools. It enables Copilot to:
- Access external APIs and databases
- Execute custom workflows
- Retrieve real-time data
- Integrate with third-party services

```mermaid
graph LR
    COPILOT["GitHub Copilot<br/>Chat"]
    MCP["MCP Server<br/>(Your Code)"]
    API["External APIs<br/>Databases<br/>Services"]
    
    COPILOT -->|"MCP Protocol"| MCP
    MCP -->|"HTTP/gRPC/SQL"| API
    API -->|"Data"| MCP
    MCP -->|"Structured Response"| COPILOT
    
    style COPILOT fill:#e1f5ff
    style MCP fill:#90EE90,stroke:#2d862d,stroke-width:3px
    style API fill:#fff4e1
```

### MCP Components

**1. MCP Server (Your Code)**
- Implements MCP protocol
- Handles tool invocations
- Manages authentication
- Returns structured responses

**2. MCP Client (VS Code)**
- Built into GitHub Copilot extension
- Discovers and connects to MCP servers
- Routes tool calls from Copilot
- Displays results to users

**3. Tools (Capabilities)**
- Functions exposed to Copilot
- Define input schemas
- Execute custom logic
- Return formatted results

---

## 🚀 Quick Start: Your First MCP Server

### Prerequisites

```bash
# Required software
node --version  # v18+ required
npm --version   # v9+ required

# Install MCP SDK
npm install @modelcontextprotocol/sdk
```

### Minimal MCP Server (TypeScript)

Create `my-first-mcp-server/src/index.ts`:

```typescript
import { Server } from '@modelcontextprotocol/sdk/server/index.js';
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js';
import {
  CallToolRequestSchema,
  ListToolsRequestSchema,
} from '@modelcontextprotocol/sdk/types.js';

// Create MCP server instance
const server = new Server(
  {
    name: 'my-first-mcp-server',
    version: '1.0.0',
  },
  {
    capabilities: {
      tools: {},
    },
  }
);

// Define available tools
server.setRequestHandler(ListToolsRequestSchema, async () => {
  return {
    tools: [
      {
        name: 'get_current_time',
        description: 'Returns the current date and time',
        inputSchema: {
          type: 'object',
          properties: {
            timezone: {
              type: 'string',
              description: 'Timezone (e.g., "America/New_York")',
              default: 'UTC',
            },
          },
        },
      },
    ],
  };
});

// Handle tool invocations
server.setRequestHandler(CallToolRequestSchema, async (request) => {
  if (request.params.name === 'get_current_time') {
    const timezone = request.params.arguments?.timezone || 'UTC';
    const now = new Date().toLocaleString('en-US', { timeZone: timezone });
    
    return {
      content: [
        {
          type: 'text',
          text: `Current time in ${timezone}: ${now}`,
        },
      ],
    };
  }
  
  throw new Error(`Unknown tool: ${request.params.name}`);
});

// Start server
async function main() {
  const transport = new StdioServerTransport();
  await server.connect(transport);
  console.error('MCP server running on stdio');
}

main().catch(console.error);
```

### Configure VS Code

Add to `.vscode/mcp.json`:

```json
{
  "mcpServers": {
    "my-first-mcp-server": {
      "command": "node",
      "args": ["./my-first-mcp-server/dist/index.js"],
      "env": {}
    }
  }
}
```

### Test Your Server

```bash
# Build TypeScript
cd my-first-mcp-server
npm run build

# Restart VS Code
# Open Copilot Chat (Ctrl+Alt+I)
# Type: "What time is it in Tokyo?"
# Copilot will use your MCP tool!
```

---

## 🔧 Building Production-Ready MCP Servers

### 1. Project Structure

```
my-mcp-server/
├── src/
│   ├── index.ts              # Server entry point
│   ├── tools/                # Tool implementations
│   │   ├── database.ts       # Database tools
│   │   ├── api.ts            # API integration tools
│   │   └── utils.ts          # Utility tools
│   ├── auth/                 # Authentication
│   │   └── credentials.ts
│   ├── config/               # Configuration
│   │   └── settings.ts
│   └── types/                # TypeScript types
│       └── index.ts
├── tests/                    # Unit tests
│   └── tools.test.ts
├── package.json
├── tsconfig.json
└── README.md
```

### 2. Tool Definition Pattern

**Best practice structure for tools:**

```typescript
// src/tools/database.ts
import { z } from 'zod';

// Define input schema with validation
export const QueryDatabaseSchema = z.object({
  query: z.string().describe('SQL query to execute'),
  database: z.enum(['production', 'staging']).describe('Target database'),
  limit: z.number().optional().default(100).describe('Max rows to return'),
});

export type QueryDatabaseInput = z.infer<typeof QueryDatabaseSchema>;

// Tool implementation
export async function queryDatabase(input: QueryDatabaseInput) {
  // Validate input
  const validated = QueryDatabaseSchema.parse(input);
  
  // Security: Prevent dangerous queries
  if (validated.query.toLowerCase().includes('drop')) {
    throw new Error('DROP statements are not allowed');
  }
  
  // Execute query (example with PostgreSQL)
  const { Pool } = await import('pg');
  const pool = new Pool({
    connectionString: process.env.DATABASE_URL,
  });
  
  try {
    const result = await pool.query(validated.query, {
      rowMode: 'array',
      maxRows: validated.limit,
    });
    
    return {
      success: true,
      rows: result.rows,
      rowCount: result.rowCount,
    };
  } finally {
    await pool.end();
  }
}

// Tool metadata for MCP
export const QueryDatabaseTool = {
  name: 'query_database',
  description: 'Execute SQL queries against application databases',
  inputSchema: QueryDatabaseSchema,
  handler: queryDatabase,
};
```

### 3. Server Implementation with Multiple Tools

```typescript
// src/index.ts
import { Server } from '@modelcontextprotocol/sdk/server/index.js';
import { StdioServerTransport } from '@modelcontextprotocol/sdk/server/stdio.js';
import {
  CallToolRequestSchema,
  ListToolsRequestSchema,
} from '@modelcontextprotocol/sdk/types.js';
import { QueryDatabaseTool } from './tools/database.js';
import { FetchAPITool } from './tools/api.js';
import { TransformDataTool } from './tools/utils.js';

// Registry of all tools
const TOOLS = [QueryDatabaseTool, FetchAPITool, TransformDataTool];

const server = new Server(
  {
    name: 'production-mcp-server',
    version: '1.0.0',
  },
  {
    capabilities: {
      tools: {},
      logging: {},
    },
  }
);

// List available tools
server.setRequestHandler(ListToolsRequestSchema, async () => {
  return {
    tools: TOOLS.map((tool) => ({
      name: tool.name,
      description: tool.description,
      inputSchema: tool.inputSchema,
    })),
  };
});

// Handle tool calls
server.setRequestHandler(CallToolRequestSchema, async (request) => {
  const tool = TOOLS.find((t) => t.name === request.params.name);
  
  if (!tool) {
    throw new Error(`Unknown tool: ${request.params.name}`);
  }
  
  try {
    // Execute tool handler
    const result = await tool.handler(request.params.arguments);
    
    // Format response
    return {
      content: [
        {
          type: 'text',
          text: JSON.stringify(result, null, 2),
        },
      ],
    };
  } catch (error) {
    // Error handling
    return {
      content: [
        {
          type: 'text',
          text: `Error: ${error.message}`,
        },
      ],
      isError: true,
    };
  }
});

// Start server
async function main() {
  const transport = new StdioServerTransport();
  await server.connect(transport);
  
  // Log to stderr (stdout is reserved for MCP protocol)
  console.error('Production MCP server started');
  console.error(`Loaded ${TOOLS.length} tools`);
}

main().catch((error) => {
  console.error('Fatal error:', error);
  process.exit(1);
});
```

---

## 🔌 Real-World Integration Examples

### Example 1: Jira Integration

```typescript
// src/tools/jira.ts
import { z } from 'zod';
import fetch from 'node-fetch';

const CreateJiraIssueSchema = z.object({
  project: z.string().describe('Jira project key (e.g., "PROJ")'),
  summary: z.string().describe('Issue title'),
  description: z.string().describe('Issue description'),
  issueType: z.enum(['Bug', 'Task', 'Story']).describe('Issue type'),
  priority: z.enum(['Highest', 'High', 'Medium', 'Low']).optional(),
});

export async function createJiraIssue(input: z.infer<typeof CreateJiraIssueSchema>) {
  const validated = CreateJiraIssueSchema.parse(input);
  
  const response = await fetch(
    `${process.env.JIRA_URL}/rest/api/3/issue`,
    {
      method: 'POST',
      headers: {
        'Authorization': `Basic ${Buffer.from(
          `${process.env.JIRA_EMAIL}:${process.env.JIRA_API_TOKEN}`
        ).toString('base64')}`,
        'Content-Type': 'application/json',
      },
      body: JSON.stringify({
        fields: {
          project: { key: validated.project },
          summary: validated.summary,
          description: {
            type: 'doc',
            version: 1,
            content: [
              {
                type: 'paragraph',
                content: [{ type: 'text', text: validated.description }],
              },
            ],
          },
          issuetype: { name: validated.issueType },
          priority: validated.priority ? { name: validated.priority } : undefined,
        },
      }),
    }
  );
  
  if (!response.ok) {
    throw new Error(`Jira API error: ${response.statusText}`);
  }
  
  const issue = await response.json();
  return {
    success: true,
    issueKey: issue.key,
    issueUrl: `${process.env.JIRA_URL}/browse/${issue.key}`,
  };
}

export const CreateJiraIssueTool = {
  name: 'create_jira_issue',
  description: 'Create a new Jira issue in the specified project',
  inputSchema: CreateJiraIssueSchema,
  handler: createJiraIssue,
};
```

**Usage in Copilot Chat:**

```
User: "Create a Jira bug for the authentication timeout issue we just fixed"

Copilot: I'll create a Jira issue for you.
[Uses create_jira_issue tool]

Created issue: PROJ-1234
URL: https://your-company.atlassian.net/browse/PROJ-1234
```

### Example 2: Database Schema Inspector

```typescript
// src/tools/schema.ts
import { z } from 'zod';
import { Pool } from 'pg';

const InspectSchemaSchema = z.object({
  tableName: z.string().optional().describe('Specific table to inspect'),
});

export async function inspectSchema(input: z.infer<typeof InspectSchemaSchema>) {
  const pool = new Pool({ connectionString: process.env.DATABASE_URL });
  
  try {
    if (input.tableName) {
      // Get columns for specific table
      const result = await pool.query(`
        SELECT 
          column_name, 
          data_type, 
          is_nullable,
          column_default
        FROM information_schema.columns
        WHERE table_name = $1
        ORDER BY ordinal_position
      `, [input.tableName]);
      
      return {
        table: input.tableName,
        columns: result.rows,
      };
    } else {
      // List all tables
      const result = await pool.query(`
        SELECT 
          table_name,
          (SELECT COUNT(*) FROM information_schema.columns 
           WHERE table_name = t.table_name) as column_count
        FROM information_schema.tables t
        WHERE table_schema = 'public'
        ORDER BY table_name
      `);
      
      return {
        tables: result.rows,
      };
    }
  } finally {
    await pool.end();
  }
}

export const InspectSchemaTool = {
  name: 'inspect_database_schema',
  description: 'Inspect database schema (tables and columns)',
  inputSchema: InspectSchemaSchema,
  handler: inspectSchema,
};
```

**Usage in Copilot Chat:**

```
User: "@workspace What columns does the users table have?"

Copilot: Let me check the database schema.
[Uses inspect_database_schema tool]

The users table has these columns:
- id (integer, not null, primary key)
- email (varchar, not null, unique)
- created_at (timestamp, not null)
- updated_at (timestamp, not null)
```

### Example 3: Slack Notification

```typescript
// src/tools/slack.ts
import { z } from 'zod';
import { WebClient } from '@slack/web-api';

const SendSlackMessageSchema = z.object({
  channel: z.string().describe('Slack channel (e.g., "#engineering")'),
  message: z.string().describe('Message text'),
  threadTs: z.string().optional().describe('Thread timestamp for replies'),
});

export async function sendSlackMessage(input: z.infer<typeof SendSlackMessageSchema>) {
  const validated = SendSlackMessageSchema.parse(input);
  
  const slack = new WebClient(process.env.SLACK_BOT_TOKEN);
  
  const result = await slack.chat.postMessage({
    channel: validated.channel,
    text: validated.message,
    thread_ts: validated.threadTs,
  });
  
  return {
    success: result.ok,
    timestamp: result.ts,
    channel: result.channel,
    permalink: result.message?.permalink,
  };
}

export const SendSlackMessageTool = {
  name: 'send_slack_message',
  description: 'Send a message to a Slack channel',
  inputSchema: SendSlackMessageSchema,
  handler: sendSlackMessage,
};
```

---

## 🔒 Security Best Practices

### 1. Authentication & Authorization

```typescript
// src/auth/credentials.ts
import * as fs from 'fs';
import * as path from 'path';

export class CredentialManager {
  private credentials: Map<string, string>;
  
  constructor() {
    this.credentials = new Map();
    this.loadCredentials();
  }
  
  private loadCredentials() {
    // Load from environment variables
    const envVars = ['JIRA_API_TOKEN', 'SLACK_BOT_TOKEN', 'DATABASE_URL'];
    
    for (const varName of envVars) {
      const value = process.env[varName];
      if (value) {
        this.credentials.set(varName, value);
      }
    }
    
    // Optionally load from secure file
    const credsPath = path.join(process.env.HOME || '', '.mcp-credentials');
    if (fs.existsSync(credsPath)) {
      const fileContents = fs.readFileSync(credsPath, 'utf-8');
      const parsed = JSON.parse(fileContents);
      
      for (const [key, value] of Object.entries(parsed)) {
        this.credentials.set(key, value as string);
      }
    }
  }
  
  get(key: string): string {
    const value = this.credentials.get(key);
    if (!value) {
      throw new Error(`Missing credential: ${key}`);
    }
    return value;
  }
  
  has(key: string): boolean {
    return this.credentials.has(key);
  }
}

// Singleton instance
export const credentials = new CredentialManager();
```

### 2. Input Validation & Sanitization

```typescript
// src/tools/safe-query.ts
import { z } from 'zod';

// Whitelist of allowed SQL operations
const ALLOWED_OPERATIONS = ['SELECT', 'SHOW', 'DESCRIBE', 'EXPLAIN'];

// Blacklist of dangerous keywords
const DANGEROUS_KEYWORDS = [
  'DROP', 'DELETE', 'TRUNCATE', 'ALTER', 'CREATE',
  'INSERT', 'UPDATE', 'GRANT', 'REVOKE',
];

export function validateSQLQuery(query: string): void {
  const upperQuery = query.trim().toUpperCase();
  
  // Check if query starts with allowed operation
  const startsWithAllowed = ALLOWED_OPERATIONS.some(op => 
    upperQuery.startsWith(op)
  );
  
  if (!startsWithAllowed) {
    throw new Error(
      `Query must start with one of: ${ALLOWED_OPERATIONS.join(', ')}`
    );
  }
  
  // Check for dangerous keywords
  for (const keyword of DANGEROUS_KEYWORDS) {
    if (upperQuery.includes(keyword)) {
      throw new Error(`Dangerous keyword detected: ${keyword}`);
    }
  }
  
  // Additional checks
  if (upperQuery.includes(';') && upperQuery.indexOf(';') < upperQuery.length - 1) {
    throw new Error('Multiple statements not allowed');
  }
}

// Usage in tool
export async function safeQueryDatabase(input: { query: string }) {
  validateSQLQuery(input.query);
  
  // Proceed with query execution
  // ...
}
```

### 3. Rate Limiting

```typescript
// src/middleware/rate-limit.ts
export class RateLimiter {
  private requests: Map<string, number[]>;
  private readonly maxRequests: number;
  private readonly windowMs: number;
  
  constructor(maxRequests: number = 100, windowMs: number = 60000) {
    this.requests = new Map();
    this.maxRequests = maxRequests;
    this.windowMs = windowMs;
  }
  
  checkLimit(toolName: string): boolean {
    const now = Date.now();
    const timestamps = this.requests.get(toolName) || [];
    
    // Remove old timestamps outside window
    const validTimestamps = timestamps.filter(
      ts => now - ts < this.windowMs
    );
    
    if (validTimestamps.length >= this.maxRequests) {
      return false; // Rate limit exceeded
    }
    
    // Add current request
    validTimestamps.push(now);
    this.requests.set(toolName, validTimestamps);
    
    return true;
  }
  
  reset(toolName: string): void {
    this.requests.delete(toolName);
  }
}

// Usage in server
const rateLimiter = new RateLimiter(100, 60000); // 100 req/min

server.setRequestHandler(CallToolRequestSchema, async (request) => {
  if (!rateLimiter.checkLimit(request.params.name)) {
    throw new Error('Rate limit exceeded. Please try again later.');
  }
  
  // Process request
  // ...
});
```

### 4. Logging & Audit Trail

```typescript
// src/logging/audit.ts
import * as fs from 'fs';
import * as path from 'path';

export class AuditLogger {
  private logPath: string;
  
  constructor(logDir: string = './logs') {
    this.logPath = path.join(logDir, 'audit.log');
    
    // Ensure log directory exists
    if (!fs.existsSync(logDir)) {
      fs.mkdirSync(logDir, { recursive: true });
    }
  }
  
  log(event: {
    timestamp: Date;
    tool: string;
    input: any;
    output?: any;
    error?: string;
    duration: number;
  }): void {
    const logEntry = {
      ...event,
      timestamp: event.timestamp.toISOString(),
    };
    
    fs.appendFileSync(
      this.logPath,
      JSON.stringify(logEntry) + '\n',
      'utf-8'
    );
  }
}

// Usage in server
const auditLogger = new AuditLogger();

server.setRequestHandler(CallToolRequestSchema, async (request) => {
  const startTime = Date.now();
  
  try {
    const result = await tool.handler(request.params.arguments);
    
    auditLogger.log({
      timestamp: new Date(),
      tool: request.params.name,
      input: request.params.arguments,
      output: result,
      duration: Date.now() - startTime,
    });
    
    return result;
  } catch (error) {
    auditLogger.log({
      timestamp: new Date(),
      tool: request.params.name,
      input: request.params.arguments,
      error: error.message,
      duration: Date.now() - startTime,
    });
    
    throw error;
  }
});
```

---

## 🧪 Testing MCP Servers

### Unit Tests

```typescript
// tests/tools.test.ts
import { describe, it, expect, beforeAll, afterAll } from 'vitest';
import { queryDatabase } from '../src/tools/database';

describe('Database Tools', () => {
  beforeAll(async () => {
    // Setup test database
    process.env.DATABASE_URL = 'postgresql://test:test@localhost/test';
  });
  
  afterAll(async () => {
    // Cleanup
  });
  
  it('should execute SELECT query', async () => {
    const result = await queryDatabase({
      query: 'SELECT * FROM users LIMIT 5',
      database: 'staging',
      limit: 5,
    });
    
    expect(result.success).toBe(true);
    expect(result.rows).toHaveLength(5);
  });
  
  it('should reject DROP statement', async () => {
    await expect(
      queryDatabase({
        query: 'DROP TABLE users',
        database: 'staging',
      })
    ).rejects.toThrow('DROP statements are not allowed');
  });
  
  it('should respect row limit', async () => {
    const result = await queryDatabase({
      query: 'SELECT * FROM users',
      database: 'staging',
      limit: 10,
    });
    
    expect(result.rows.length).toBeLessThanOrEqual(10);
  });
});
```

### Integration Tests

```typescript
// tests/integration.test.ts
import { Server } from '@modelcontextprotocol/sdk/server/index.js';
import { CallToolRequestSchema } from '@modelcontextprotocol/sdk/types.js';

describe('MCP Server Integration', () => {
  let server: Server;
  
  beforeAll(() => {
    // Initialize server
    server = createTestServer();
  });
  
  it('should list all tools', async () => {
    const response = await server.request({
      method: 'tools/list',
    });
    
    expect(response.tools).toHaveLength(3);
    expect(response.tools[0].name).toBe('query_database');
  });
  
  it('should execute tool successfully', async () => {
    const response = await server.request({
      method: 'tools/call',
      params: {
        name: 'get_current_time',
        arguments: { timezone: 'UTC' },
      },
    });
    
    expect(response.content[0].type).toBe('text');
    expect(response.content[0].text).toContain('Current time');
  });
});
```

---

## 📦 Deployment Strategies

### 1. Local Development

```json
// .vscode/mcp.json
{
  "mcpServers": {
    "my-mcp-server": {
      "command": "node",
      "args": ["./dist/index.js"],
      "env": {
        "DATABASE_URL": "postgresql://localhost/dev",
        "JIRA_URL": "https://your-company.atlassian.net",
        "LOG_LEVEL": "debug"
      }
    }
  }
}
```

### 2. Team Distribution (npm Package)

```json
// package.json
{
  "name": "@your-company/mcp-server",
  "version": "1.0.0",
  "bin": {
    "company-mcp-server": "./dist/index.js"
  },
  "files": ["dist/**/*"],
  "scripts": {
    "build": "tsc",
    "prepublishOnly": "npm run build"
  }
}
```

**Team installation:**

```bash
# Install globally
npm install -g @your-company/mcp-server

# Configure in VS Code
# .vscode/mcp.json
{
  "mcpServers": {
    "company-tools": {
      "command": "company-mcp-server",
      "args": [],
      "env": {}
    }
  }
}
```

### 3. Docker Deployment

```dockerfile
# Dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY dist ./dist

USER node

CMD ["node", "dist/index.js"]
```

```yaml
# docker-compose.yml
version: '3.8'

services:
  mcp-server:
    build: .
    environment:
      - DATABASE_URL=${DATABASE_URL}
      - JIRA_API_TOKEN=${JIRA_API_TOKEN}
    volumes:
      - ./logs:/app/logs
    restart: unless-stopped
```

---

## 🔍 Debugging MCP Servers

### 1. Enable Debug Logging

```typescript
// src/index.ts
import { Server } from '@modelcontextprotocol/sdk/server/index.js';

const server = new Server(
  {
    name: 'my-mcp-server',
    version: '1.0.0',
  },
  {
    capabilities: {
      tools: {},
      logging: {},
    },
  }
);

// Log all requests
server.setRequestHandler(CallToolRequestSchema, async (request) => {
  console.error('[DEBUG] Tool called:', request.params.name);
  console.error('[DEBUG] Arguments:', JSON.stringify(request.params.arguments, null, 2));
  
  const startTime = Date.now();
  
  try {
    const result = await tool.handler(request.params.arguments);
    console.error('[DEBUG] Success in', Date.now() - startTime, 'ms');
    return result;
  } catch (error) {
    console.error('[DEBUG] Error:', error.message);
    throw error;
  }
});
```

### 2. VS Code Output Panel

```
1. Open VS Code Output panel (Ctrl+Shift+U)
2. Select "GitHub Copilot" from dropdown
3. Look for MCP server logs
4. Check for connection errors or tool failures
```

### 3. Manual Testing with MCP Inspector

```bash
# Install MCP Inspector
npm install -g @modelcontextprotocol/inspector

# Run your server with inspector
mcp-inspector node dist/index.js

# Opens web UI at http://localhost:5173
# Test tools interactively
```

---

## 📚 Advanced Patterns

### Pattern 1: Streaming Responses

```typescript
// For long-running operations
server.setRequestHandler(CallToolRequestSchema, async (request) => {
  if (request.params.name === 'analyze_large_file') {
    // Stream progress updates
    const stream = new ReadableStream({
      async start(controller) {
        controller.enqueue({ type: 'text', text: 'Starting analysis...\n' });
        
        // Process in chunks
        for (let i = 0; i < 100; i += 10) {
          await processChunk(i, i + 10);
          controller.enqueue({ 
            type: 'text', 
            text: `Progress: ${i + 10}%\n` 
          });
        }
        
        controller.enqueue({ type: 'text', text: 'Analysis complete!\n' });
        controller.close();
      },
    });
    
    return { content: stream };
  }
});
```

### Pattern 2: Tool Chaining

```typescript
// Tools can call other tools
export async function analyzeAndReport(input: { query: string }) {
  // Step 1: Query database
  const data = await queryDatabase({ query: input.query });
  
  // Step 2: Transform data
  const transformed = await transformData({ data: data.rows });
  
  // Step 3: Send Slack notification
  await sendSlackMessage({
    channel: '#analytics',
    message: `Analysis complete: ${transformed.summary}`,
  });
  
  return {
    success: true,
    data: transformed,
  };
}
```

### Pattern 3: Caching Results

```typescript
// src/cache/simple-cache.ts
export class SimpleCache<T> {
  private cache: Map<string, { value: T; expires: number }>;
  
  constructor(private ttlMs: number = 60000) {
    this.cache = new Map();
  }
  
  get(key: string): T | undefined {
    const entry = this.cache.get(key);
    if (!entry) return undefined;
    
    if (Date.now() > entry.expires) {
      this.cache.delete(key);
      return undefined;
    }
    
    return entry.value;
  }
  
  set(key: string, value: T): void {
    this.cache.set(key, {
      value,
      expires: Date.now() + this.ttlMs,
    });
  }
}

// Usage
const schemaCache = new SimpleCache<any>(300000); // 5 min TTL

export async function inspectSchema(input: { tableName: string }) {
  const cacheKey = `schema:${input.tableName}`;
  
  // Check cache first
  const cached = schemaCache.get(cacheKey);
  if (cached) {
    return cached;
  }
  
  // Fetch from database
  const result = await fetchSchemaFromDB(input.tableName);
  
  // Cache result
  schemaCache.set(cacheKey, result);
  
  return result;
}
```

---

## ✅ Key Takeaways

**MCP Server Essentials:**
- MCP extends Copilot with custom tools and data sources
- Use TypeScript + MCP SDK for type-safe implementations
- Define tools with clear schemas and descriptions
- Implement proper error handling and validation

**Security First:**
- Validate all inputs with Zod or similar
- Implement rate limiting for expensive operations
- Use environment variables for credentials
- Maintain audit logs for all tool invocations

**Production Readiness:**
- Write comprehensive unit and integration tests
- Implement caching for frequently accessed data
- Add detailed logging for debugging
- Document all tools and their usage

**Distribution:**
- Package as npm module for team distribution
- Use Docker for consistent deployment
- Configure via `.vscode/mcp.json` for VS Code
- Version your MCP servers properly

---

## 🚀 Next Steps

1. **Build your first MCP server** using the quick start example
2. **Integrate with your team's tools** (Jira, Slack, databases)
3. **Test thoroughly** with unit and integration tests
4. **Deploy to your team** via npm or Docker
5. **Monitor usage** through audit logs

**Continue to:** [Section 2: Training Team-Specific Agents](02-training-team-agents.md)

---

**Resources:**
- [MCP Specification](https://modelcontextprotocol.io)
- [MCP SDK Documentation](https://github.com/modelcontextprotocol/sdk)
- [VS Code MCP Configuration](https://code.visualstudio.com/docs/copilot/chat/mcp)

