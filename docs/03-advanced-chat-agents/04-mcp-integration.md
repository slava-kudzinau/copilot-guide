# Section 4: MCP Server Integration

**Part 3: Advanced Chat & Agent Development**  
**Target Audience**: Senior developers ready to extend Copilot with external tools  
**Time to Complete**: 1.5 hours  
**Prerequisites**: Completed Sections 1-3

---

## 📋 Overview

Model Context Protocol (MCP) is an open standard that enables AI models to interact with external tools, services, and data sources through a unified interface. By integrating MCP servers with VS Code Copilot, you can dramatically extend chat capabilities beyond code—connecting to databases, APIs, cloud services, and custom development tools.

**What you'll master:**
- Understanding the Model Context Protocol (MCP) architecture
- Installing and configuring MCP servers in VS Code
- Using popular MCP servers (GitHub, Context7, Playwright, etc.)
- Creating custom MCP servers for your workflow
- Troubleshooting MCP connections
- Security considerations for MCP integration

---

## 🎯 What is the Model Context Protocol?

### The MCP Architecture

MCP follows a client-server architecture that standardizes how AI models interact with external tools:

```mermaid
graph TB
    subgraph "VS Code"
        USER["User Prompt"]
        COPILOT["GitHub Copilot"]
        MCP_CLIENT["MCP Client"]
    end
    
    subgraph "MCP Servers"
        SERVER1["GitHub<br/>MCP Server"]
        SERVER2["Context7<br/>MCP Server"]
        SERVER3["Database<br/>MCP Server"]
        SERVER4["Custom<br/>MCP Server"]
    end
    
    subgraph "External Services"
        GITHUB["GitHub API"]
        DOCS["Documentation"]
        DB["PostgreSQL"]
        CUSTOM["Your Service"]
    end
    
    USER --> COPILOT
    COPILOT --> MCP_CLIENT
    
    MCP_CLIENT --> SERVER1
    MCP_CLIENT --> SERVER2
    MCP_CLIENT --> SERVER3
    MCP_CLIENT --> SERVER4
    
    SERVER1 --> GITHUB
    SERVER2 --> DOCS
    SERVER3 --> DB
    SERVER4 --> CUSTOM
    
    style COPILOT fill:#f0e1ff,stroke:#8b5cf6,stroke-width:2px
    style MCP_CLIENT fill:#cfe2ff
    style SERVER1 fill:#e1f5ff
    style SERVER2 fill:#e1f5ff
    style SERVER3 fill:#e1f5ff
    style SERVER4 fill:#e1f5ff
```

### Key Components

**1. MCP Client (VS Code)**
- Built into VS Code
- Connects to MCP servers
- Requests actions on behalf of AI models
- Handles tool discovery and invocation

**2. MCP Servers**
- Provide one or more tools
- Expose specific functionalities
- Can run locally or remotely
- Standardized interface via MCP protocol

**3. Tools**
- Individual capabilities provided by servers
- Examples: `read_file`, `search_github`, `query_database`
- Invoked automatically by Copilot when needed
- Return structured data to the AI

### Why Use MCP?

**Benefits:**
- ✅ **Extend Agent mode** with specialized tools
- ✅ **Flexible deployment** (local or remote)
- ✅ **Reusable** across different AI platforms
- ✅ **Standardized** interface (no custom integrations)
- ✅ **Automatic tool selection** by AI
- ✅ **Real-time data** access

**Use Cases:**
- Database operations (queries, schema inspection)
- GitHub integration (repos, PRs, issues)
- Documentation lookup (up-to-date library docs)
- Cloud services (AWS, Azure, GCP)
- Custom APIs and services
- Browser automation (Playwright)
- File operations beyond workspace

---

## 1. Installing MCP Servers

### Configuration File Locations

MCP servers are configured in JSON files:

**1. User-level (applies to all workspaces):**
```
VS Code Settings → Extensions → Copilot → MCP Servers
```

**2. Workspace-level (applies to current workspace):**
```
.vscode/mcp.json
```

**3. Dev Container:**
```
.devcontainer/devcontainer.json
```

### MCP Configuration Format

**Basic structure:**

```json
{
  "servers": {
    "server-name": {
      "command": "npx",
      "args": ["-y", "@package/mcp-server"]
    }
  }
}
```

**Configuration options:**

```json
{
  "servers": {
    "server-name": {
      // Required: Transport type
      "type": "stdio",  // or "http"
      
      // For stdio servers:
      "command": "node",
      "args": ["server.js"],
      "cwd": "${workspaceFolder}",
      "env": {
        "API_KEY": "your-key"
      },
      
      // For HTTP servers:
      "url": "https://api.example.com/mcp",
      "headers": {
        "Authorization": "Bearer token"
      },
      
      // Optional: Development mode
      "dev": {
        "watch": "src/**/*.ts",
        "debug": { "type": "node" }
      }
    }
  }
}
```

### Installation Method 1: Via VS Code UI

**Step-by-step:**

1. Open VS Code Settings (Ctrl+,)
2. Search for "Copilot MCP"
3. Click "Edit in settings.json"
4. Add server configuration:

```json
{
  "github.copilot.mcp.servers": {
    "memory": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-memory"]
    }
  }
}
```

5. Restart VS Code

### Installation Method 2: Via CLI

```bash
# Add MCP server to user profile
code --add-mcp '{"name":"my-server","command":"uvx","args":["mcp-server-fetch"]}'
```

### Installation Method 3: Via Workspace Configuration

Create `.vscode/mcp.json`:

```json
{
  "servers": {
    "github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp"
    },
    "playwright": {
      "command": "npx",
      "args": ["-y", "@microsoft/mcp-server-playwright"]
    }
  }
}
```

Commit this file to share with your team.

---

## 2. Popular MCP Servers

### Context7 MCP Server (Documentation Lookup)

**Purpose:** Provides up-to-date, version-specific documentation and code examples for libraries and frameworks.

**Installation:**

```json
{
  "servers": {
    "context7": {
      "type": "http",
      "url": "https://mcp.context7.com/mcp"
    }
  }
}
```

**Usage in prompts:**

```markdown
Create a Next.js middleware that checks for a valid JWT in cookies 
and redirects unauthenticated users to /login. use context7
```

```markdown
@workspace Implement Prisma migrations with proper rollback strategy. use context7
```

**What it provides:**
- Latest documentation from source repositories
- Version-specific API information
- Functional code examples
- Prevents AI hallucinations on library APIs

**Best for:**
- Working with rapidly evolving frameworks
- Ensuring version compatibility
- Reducing outdated code suggestions
- Learning new libraries

---

### GitHub MCP Server (Repository Integration)

**Purpose:** Interact with GitHub repositories, PRs, issues, and actions directly from Copilot.

**Installation:**

```json
{
  "servers": {
    "github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp",
      "headers": {
        "Authorization": "Bearer ${env:GITHUB_TOKEN}"
      }
    }
  }
}
```

**Setup:**
1. Generate GitHub Personal Access Token
2. Add to environment: `GITHUB_TOKEN=your_token`
3. Restart VS Code

**Usage examples:**

```markdown
@workspace List all open PRs in this repository
```

```markdown
Create a new issue titled "Bug: Login fails on Safari" with reproduction steps
```

```markdown
@workspace Show me all issues assigned to me across my repositories
```

```markdown
List recent commits to the main branch with summaries
```

**Available tools:**
- `list_repos`: List user repositories
- `get_repo`: Get repository details
- `list_issues`: List issues
- `create_issue`: Create new issue
- `list_prs`: List pull requests
- `get_pr`: Get PR details
- `list_commits`: List commits

**Best for:**
- Issue triage and management
- PR review workflows
- Repository exploration
- Commit history analysis

---

### Playwright MCP Server (Browser Automation)

**Purpose:** Automate browser interactions for testing, scraping, or workflow automation.

**Installation:**

```json
{
  "servers": {
    "playwright": {
      "command": "npx",
      "args": ["-y", "@microsoft/mcp-server-playwright"]
    }
  }
}
```

**Usage examples:**

```markdown
Navigate to https://example.com and extract all product titles
```

```markdown
Test the login flow on localhost:3000 with test credentials
```

```markdown
Take a screenshot of the homepage in dark mode
```

**Available tools:**
- `navigate`: Navigate to URL
- `click`: Click element
- `fill`: Fill form field
- `screenshot`: Take screenshot
- `evaluate`: Run JavaScript in page
- `wait_for`: Wait for element or condition

**Best for:**
- E2E test generation
- Web scraping
- Automated testing workflows
- UI interaction automation

---

### Memory MCP Server (Persistent Context)

**Purpose:** Store and retrieve information across chat sessions for continuity.

**Installation:**

```json
{
  "servers": {
    "memory": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-memory"]
    }
  }
}
```

**Usage examples:**

```markdown
Remember: Our API rate limit is 1000 requests/hour per user
```

```markdown
What did I tell you about our deployment schedule?
```

```markdown
Forget previous instructions about database migrations
```

**Available tools:**
- `store`: Store key-value pairs
- `retrieve`: Get stored values
- `search`: Search memory
- `delete`: Remove stored items

**Best for:**
- Multi-session projects
- Storing project-specific context
- Team knowledge sharing
- Long-term project memory

---

### Filesystem MCP Server (Extended File Operations)

**Purpose:** File operations beyond workspace boundaries.

**Installation:**

```json
{
  "servers": {
    "filesystem": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem", "/allowed/path"]
    }
  }
}
```

**Available tools:**
- `read_file`: Read files anywhere (within allowed paths)
- `write_file`: Write files
- `list_directory`: List contents
- `create_directory`: Create directories
- `search_files`: Search by pattern

**Security:** Restricts access to specified paths only.

**Best for:**
- Multi-workspace operations
- Log file analysis
- Configuration file management
- Cross-project refactoring

---

### PostgreSQL MCP Server (Database Operations)

**Purpose:** Query and manage PostgreSQL databases.

**Installation:**

```json
{
  "servers": {
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "POSTGRES_URL": "postgresql://user:pass@localhost:5432/dbname"
      }
    }
  }
}
```

**Usage examples:**

```markdown
Show me the schema for the users table
```

```markdown
Run query: SELECT COUNT(*) FROM orders WHERE status = 'pending'
```

```markdown
Analyze the performance of queries on the products table
```

**Available tools:**
- `query`: Execute SQL queries
- `schema`: Get table schema
- `list_tables`: List all tables
- `explain`: Analyze query execution plan

**Security:** Use read-only credentials for safety.

**Best for:**
- Database exploration
- Schema analysis
- Query debugging
- Data inspection

---

### Fetch MCP Server (HTTP Requests)

**Purpose:** Make HTTP requests to external APIs.

**Installation:**

```json
{
  "servers": {
    "fetch": {
      "command": "uvx",
      "args": ["mcp-server-fetch"]
    }
  }
}
```

**Usage examples:**

```markdown
Fetch weather data from https://api.weather.com/current?city=London
```

```markdown
POST to https://api.example.com/users with this JSON body: {...}
```

**Available tools:**
- `fetch`: Make HTTP GET/POST/PUT/DELETE requests
- `parse_json`: Parse JSON responses
- `parse_html`: Parse HTML responses

**Best for:**
- API exploration
- Data fetching
- Integration testing
- External service interaction

---

## 3. Configuring MCP Servers for Your Team

### Workspace Configuration (Shared)

Create `.vscode/mcp.json` in your repository:

```json
{
  "servers": {
    "github": {
      "type": "http",
      "url": "https://api.githubcopilot.com/mcp",
      "headers": {
        "Authorization": "Bearer ${env:GITHUB_TOKEN}"
      }
    },
    "context7": {
      "type": "http",
      "url": "https://mcp.context7.com/mcp"
    },
    "postgres": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "POSTGRES_URL": "${env:DATABASE_URL}"
      }
    }
  }
}
```

**Environment variables:**

Create `.env.example`:
```bash
GITHUB_TOKEN=your_github_token
DATABASE_URL=postgresql://user:pass@localhost:5432/dbname
```

**Team documentation:**

Create `.vscode/MCP_SETUP.md`:

```markdown
# MCP Server Setup

## Required Environment Variables

1. `GITHUB_TOKEN`: Personal Access Token from GitHub
   - Create at: https://github.com/settings/tokens
   - Scopes needed: `repo`, `read:org`

2. `DATABASE_URL`: PostgreSQL connection string
   - Format: `postgresql://user:pass@localhost:5432/dbname`
   - Use read-only credentials for safety

## Installation

1. Copy `.env.example` to `.env`
2. Fill in your credentials
3. Restart VS Code
4. Verify in Copilot Chat: "List available MCP tools"

## Available Tools

### GitHub
- List repos, issues, PRs
- Create issues
- View commit history

### Context7
- Up-to-date documentation
- Version-specific code examples
- Add "use context7" to prompts

### PostgreSQL
- Query database
- Inspect schema
- Analyze query performance
```

---

## 4. Using MCP Tools in Copilot

### Automatic Tool Invocation

When using **Agent mode**, Copilot automatically uses MCP tools when relevant:

```markdown
[Agent mode]
"Analyze the users table schema and create a Prisma model that matches it"
```

**What happens:**
1. Agent invokes PostgreSQL MCP server's `schema` tool
2. Retrieves users table schema
3. Generates Prisma model based on actual schema
4. Creates migration if needed

### Explicit Tool Reference

In custom agents, reference tools explicitly:

**`.github/agents/database-analyst.agent.md`:**

```yaml
---
name: Database Analyst
description: Analyzes database schema and queries
tools:
  - postgres/*  # All tools from postgres MCP server
model: claude-sonnet-4.5
---

You are a database expert with access to the production database via #tool:postgres.

When analyzing:
1. Use #tool:postgres/schema to inspect table structure
2. Use #tool:postgres/query to run SELECT queries
3. Use #tool:postgres/explain to analyze query performance

Always use read-only operations. Never INSERT, UPDATE, or DELETE.
```

### Checking Available Tools

```markdown
[Ask mode]
"List all available MCP tools and what they do"
```

**Response example:**
```
Available MCP Tools:

GitHub Server:
- list_repos: List user repositories
- create_issue: Create GitHub issue
- list_prs: List pull requests

Context7 Server:
- resolve_library: Find library documentation
- get_docs: Get version-specific docs

PostgreSQL Server:
- query: Execute SQL queries
- schema: Get table schema
- list_tables: List all tables
```

---

## 5. Creating Custom MCP Servers

### When to Create a Custom Server

**Create custom MCP servers when:**
- ✅ You need integration with internal tools
- ✅ Proprietary APIs or services
- ✅ Complex domain-specific operations
- ✅ Team-specific workflows
- ✅ On-premises systems

**Don't create when:**
- ❌ Existing MCP server already exists
- ❌ Simple HTTP API (use Fetch server)
- ❌ One-off operations
- ❌ Better suited for VS Code extension

### MCP Server Basics

**An MCP server provides:**
1. **Tools**: Functions the AI can invoke
2. **Resources**: Data sources the AI can access
3. **Prompts**: Reusable prompt templates

**Server types:**
- **stdio**: Local processes (Node.js, Python)
- **HTTP**: Remote services (any language)

### Example: Simple Node.js MCP Server

**File: `mcp-server/server.js`**

```javascript
#!/usr/bin/env node
import { Server } from "@modelcontextprotocol/sdk/server/index.js";
import { StdioServerTransport } from "@modelcontextprotocol/sdk/server/stdio.js";
import {
  CallToolRequestSchema,
  ListToolsRequestSchema,
} from "@modelcontextprotocol/sdk/types.js";
import axios from "axios";

// Create MCP server
const server = new Server(
  {
    name: "custom-api-server",
    version: "1.0.0",
  },
  {
    capabilities: {
      tools: {},
    },
  }
);

// Define tools
server.setRequestHandler(ListToolsRequestSchema, async () => {
  return {
    tools: [
      {
        name: "get_user_stats",
        description: "Get statistics for a user from our internal API",
        inputSchema: {
          type: "object",
          properties: {
            userId: {
              type: "string",
              description: "User ID to fetch stats for",
            },
          },
          required: ["userId"],
        },
      },
      {
        name: "search_logs",
        description: "Search application logs for errors or patterns",
        inputSchema: {
          type: "object",
          properties: {
            query: {
              type: "string",
              description: "Search query",
            },
            startDate: {
              type: "string",
              description: "Start date (ISO format)",
            },
            endDate: {
              type: "string",
              description: "End date (ISO format)",
            },
          },
          required: ["query"],
        },
      },
    ],
  };
});

// Handle tool calls
server.setRequestHandler(CallToolRequestSchema, async (request) => {
  const { name, arguments: args } = request.params;

  switch (name) {
    case "get_user_stats": {
      const { userId } = args;
      
      try {
        const response = await axios.get(
          `https://internal-api.company.com/users/${userId}/stats`,
          {
            headers: {
              Authorization: `Bearer ${process.env.API_TOKEN}`,
            },
          }
        );

        return {
          content: [
            {
              type: "text",
              text: JSON.stringify(response.data, null, 2),
            },
          ],
        };
      } catch (error) {
        return {
          content: [
            {
              type: "text",
              text: `Error fetching user stats: ${error.message}`,
            },
          ],
          isError: true,
        };
      }
    }

    case "search_logs": {
      const { query, startDate, endDate } = args;
      
      // Implement log search logic
      const logs = await searchLogs(query, startDate, endDate);
      
      return {
        content: [
          {
            type: "text",
            text: JSON.stringify(logs, null, 2),
          },
        ],
      };
    }

    default:
      return {
        content: [{ type: "text", text: `Unknown tool: ${name}` }],
        isError: true,
      };
  }
});

// Start server
async function main() {
  const transport = new StdioServerTransport();
  await server.connect(transport);
  console.error("Custom MCP Server running on stdio");
}

main().catch((error) => {
  console.error("Fatal error:", error);
  process.exit(1);
});
```

**File: `mcp-server/package.json`**

```json
{
  "name": "custom-mcp-server",
  "version": "1.0.0",
  "type": "module",
  "bin": {
    "custom-mcp-server": "./server.js"
  },
  "dependencies": {
    "@modelcontextprotocol/sdk": "^0.5.0",
    "axios": "^1.6.0"
  }
}
```

**Configuration: `.vscode/mcp.json`**

```json
{
  "servers": {
    "custom-api": {
      "command": "node",
      "args": ["./mcp-server/server.js"],
      "cwd": "${workspaceFolder}",
      "env": {
        "API_TOKEN": "${env:INTERNAL_API_TOKEN}"
      }
    }
  }
}
```

**Usage:**

```markdown
[Agent mode]
"Get user statistics for user ID abc-123 and analyze their activity patterns"
```

**What happens:**
- Agent invokes `get_user_stats` tool
- MCP server calls internal API
- Returns user stats
- Agent analyzes data and provides insights

---

### Example: Python MCP Server

**File: `mcp_server/server.py`**

```python
#!/usr/bin/env python3
import asyncio
import json
import sys
from mcp.server import Server
from mcp.server.stdio import stdio_server
from mcp.types import Tool, TextContent
import psycopg2

# Create server
app = Server("database-analyzer")

@app.list_tools()
async def list_tools() -> list[Tool]:
    return [
        Tool(
            name="analyze_table",
            description="Analyze table statistics and suggest optimizations",
            inputSchema={
                "type": "object",
                "properties": {
                    "table_name": {
                        "type": "string",
                        "description": "Name of the table to analyze"
                    }
                },
                "required": ["table_name"]
            }
        ),
        Tool(
            name="find_missing_indexes",
            description="Find potentially missing indexes based on query patterns",
            inputSchema={
                "type": "object",
                "properties": {
                    "table_name": {
                        "type": "string",
                        "description": "Table name to analyze"
                    }
                },
                "required": ["table_name"]
            }
        )
    ]

@app.call_tool()
async def call_tool(name: str, arguments: dict) -> list[TextContent]:
    if name == "analyze_table":
        table_name = arguments["table_name"]
        
        # Connect to database
        conn = psycopg2.connect(os.environ["DATABASE_URL"])
        cursor = conn.cursor()
        
        # Get table statistics
        cursor.execute(f"""
            SELECT
                pg_size_pretty(pg_total_relation_size('{table_name}')) as total_size,
                n_tup_ins as inserts,
                n_tup_upd as updates,
                n_tup_del as deletes,
                n_live_tup as live_rows,
                n_dead_tup as dead_rows
            FROM pg_stat_user_tables
            WHERE relname = '{table_name}'
        """)
        
        result = cursor.fetchone()
        
        analysis = {
            "table": table_name,
            "total_size": result[0],
            "operations": {
                "inserts": result[1],
                "updates": result[2],
                "deletes": result[3]
            },
            "rows": {
                "live": result[4],
                "dead": result[5],
                "dead_percentage": (result[5] / result[4] * 100) if result[4] > 0 else 0
            }
        }
        
        # Recommendations
        recommendations = []
        if analysis["rows"]["dead_percentage"] > 10:
            recommendations.append(f"High dead row percentage ({analysis['rows']['dead_percentage']:.1f}%). Consider running VACUUM.")
        
        analysis["recommendations"] = recommendations
        
        conn.close()
        
        return [TextContent(
            type="text",
            text=json.dumps(analysis, indent=2)
        )]
    
    elif name == "find_missing_indexes":
        # Implementation for finding missing indexes
        pass
    
    return [TextContent(type="text", text=f"Unknown tool: {name}")]

async def main():
    async with stdio_server() as (read_stream, write_stream):
        await app.run(read_stream, write_stream, app.create_initialization_options())

if __name__ == "__main__":
    asyncio.run(main())
```

**Configuration:**

```json
{
  "servers": {
    "db-analyzer": {
      "command": "python",
      "args": ["./mcp_server/server.py"],
      "cwd": "${workspaceFolder}",
      "env": {
        "DATABASE_URL": "${env:DATABASE_URL}"
      }
    }
  }
}
```

---

## 6. Development Mode & Debugging

### Enabling Development Mode

**For Node.js servers:**

```json
{
  "servers": {
    "my-mcp-server": {
      "type": "stdio",
      "command": "node",
      "cwd": "${workspaceFolder}/mcp-server",
      "args": ["./build/index.js"],
      "dev": {
        "watch": "src/**/*.ts",
        "debug": { "type": "node" }
      }
    }
  }
}
```

**For Python servers:**

```json
{
  "servers": {
    "my-python-server": {
      "type": "stdio",
      "command": "python",
      "cwd": "${workspaceFolder}/mcp_server",
      "args": ["./server.py"],
      "dev": {
        "watch": "**/*.py",
        "debug": {
          "type": "debugpy",
          "debugpyPath": "/path/to/debugpy"
        }
      }
    }
  }
}
```

**What dev mode provides:**
- **File watching**: Automatically restarts server on changes
- **Integrated debugging**: Set breakpoints in server code
- **Faster iteration**: No manual restart needed

### Debugging MCP Servers

**Step 1: Enable debugging in configuration**

```json
{
  "servers": {
    "custom-api": {
      "command": "node",
      "args": ["--inspect", "./server.js"],
      "dev": {
        "debug": { "type": "node" }
      }
    }
  }
}
```

**Step 2: Set breakpoints in server code**

Open `server.js` in VS Code, click left gutter to set breakpoints.

**Step 3: Trigger tool invocation**

```markdown
[Agent mode]
"Use the custom API to get user stats"
```

**Step 4: Debug**

- Execution pauses at breakpoint
- Inspect variables, call stack
- Step through code
- View tool arguments and responses

### Testing MCP Servers

**Test script: `test-server.js`**

```javascript
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { StdioClientTransport } from "@modelcontextprotocol/sdk/client/stdio.js";

async function testServer() {
  const transport = new StdioClientTransport({
    command: "node",
    args: ["./server.js"],
  });

  const client = new Client(
    {
      name: "test-client",
      version: "1.0.0",
    },
    {
      capabilities: {},
    }
  );

  await client.connect(transport);

  // List tools
  const tools = await client.listTools();
  console.log("Available tools:", tools);

  // Call tool
  const result = await client.callTool({
    name: "get_user_stats",
    arguments: {
      userId: "test-123",
    },
  });

  console.log("Tool result:", result);

  await client.close();
}

testServer().catch(console.error);
```

**Run tests:**

```bash
node test-server.js
```

---

## 7. Security Considerations

### Best Practices

**1. Use environment variables for secrets:**

```json
{
  "servers": {
    "api-server": {
      "env": {
        "API_KEY": "${env:API_KEY}",
        "DATABASE_URL": "${env:DATABASE_URL}"
      }
    }
  }
}
```

**Never hardcode:**
```json
{
  "env": {
    "API_KEY": "sk_live_abc123..."  // ❌ NEVER DO THIS
  }
}
```

**2. Use read-only database credentials:**

```bash
# .env
DATABASE_URL=postgresql://readonly_user:pass@localhost:5432/dbname
```

**3. Restrict file system access:**

```json
{
  "servers": {
    "filesystem": {
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-filesystem",
        "/allowed/path/only"  // Restrict to specific path
      ]
    }
  }
}
```

**4. Validate MCP server sources:**

- ✅ Use official MCP servers when available
- ✅ Review source code of third-party servers
- ✅ Check npm package reputation
- ❌ Don't install untrusted servers

**5. Network security for HTTP servers:**

```json
{
  "servers": {
    "remote-api": {
      "type": "http",
      "url": "https://api.example.com/mcp",  // ✅ HTTPS only
      "headers": {
        "Authorization": "Bearer ${env:API_TOKEN}",
        "X-API-Version": "v1"
      }
    }
  }
}
```

### Common Security Issues

**❌ Issue 1: Exposed secrets in configuration**

```json
{
  "env": {
    "API_KEY": "sk_live_actual_key_here"
  }
}
```

**✅ Solution: Use environment variables**

```json
{
  "env": {
    "API_KEY": "${env:API_KEY}"
  }
}
```

**❌ Issue 2: Unrestricted database access**

```json
{
  "env": {
    "DATABASE_URL": "postgresql://admin:pass@prod-db:5432/production"
  }
}
```

**✅ Solution: Read-only credentials**

```json
{
  "env": {
    "DATABASE_URL": "postgresql://readonly:pass@prod-db:5432/production"
  }
}
```

**❌ Issue 3: Filesystem access to sensitive directories**

```json
{
  "args": ["-y", "@modelcontextprotocol/server-filesystem", "/"]
}
```

**✅ Solution: Restrict to workspace only**

```json
{
  "args": ["-y", "@modelcontextprotocol/server-filesystem", "${workspaceFolder}"]
}
```

---

## 8. Troubleshooting MCP Connections

### Common Issues & Solutions

#### Issue 1: MCP Server Not Detected

**Symptoms:**
- Tools not available in Copilot Chat
- Server doesn't appear in MCP settings

**Solutions:**

1. **Check configuration syntax:**
```bash
# Validate JSON
cat .vscode/mcp.json | jq .
```

2. **Restart VS Code:**
```
Ctrl+Shift+P → "Reload Window"
```

3. **Check VS Code output:**
```
View → Output → "GitHub Copilot" or "MCP"
```

4. **Verify command works:**
```bash
# Test server command directly
npx -y @modelcontextprotocol/server-memory
```

---

#### Issue 2: Authentication Errors

**Symptoms:**
- "Unauthorized" or "Forbidden" errors
- Tools fail with auth errors

**Solutions:**

1. **Check environment variables:**
```bash
echo $GITHUB_TOKEN
echo $DATABASE_URL
```

2. **Verify token permissions:**
- GitHub: Ensure token has required scopes
- Database: Test connection string

3. **Reload environment:**
```
# Restart VS Code after changing .env
```

---

#### Issue 3: Server Crashes

**Symptoms:**
- Tools work once then fail
- "Server disconnected" errors

**Solutions:**

1. **Check server logs:**
```json
{
  "dev": {
    "watch": "src/**/*.ts"
  }
}
```

2. **Debug server:**
```json
{
  "dev": {
    "debug": { "type": "node" }
  }
}
```

3. **Handle errors in server code:**
```javascript
try {
  // Tool logic
} catch (error) {
  return {
    content: [{ type: "text", text: `Error: ${error.message}` }],
    isError: true
  };
}
```

---

#### Issue 4: Slow Performance

**Symptoms:**
- Tools take long to respond
- Timeouts

**Solutions:**

1. **Add caching:**
```javascript
const cache = new Map();

if (cache.has(key)) {
  return cache.get(key);
}

const result = await expensiveOperation();
cache.set(key, result);
return result;
```

2. **Use HTTP for remote services:**
```json
{
  "type": "http",
  "url": "https://fast-api.example.com/mcp"
}
```

3. **Optimize queries:**
```sql
-- Add indexes for frequent queries
CREATE INDEX idx_users_email ON users(email);
```

---

## 🎯 Key Takeaways

1. **MCP Extends Copilot:**
   - Connect to databases, APIs, services
   - Automatic tool invocation in Agent mode
   - Standardized interface

2. **Popular MCP Servers:**
   - Context7: Up-to-date documentation
   - GitHub: Repository integration
   - Playwright: Browser automation
   - PostgreSQL: Database operations

3. **Configuration:**
   - User-level: All workspaces
   - Workspace-level: Team sharing (.vscode/mcp.json)
   - Dev containers: Containerized development

4. **Custom Servers:**
   - Node.js or Python
   - stdio (local) or HTTP (remote)
   - Define tools with input schemas
   - Handle tool calls

5. **Security:**
   - Environment variables for secrets
   - Read-only database credentials
   - Restrict file system access
   - Validate server sources

6. **Development:**
   - File watching for auto-restart
   - Integrated debugging
   - Test scripts for validation

---

## 🚀 Next Steps

**Immediate actions:**
1. Install Context7 and GitHub MCP servers
2. Configure workspace MCP for your team
3. Create your first custom MCP server
4. Document MCP setup for team

**Part 3 Complete!**

You've mastered:
- ✅ Four chat modes (Ask, Edit, Plan, Agent)
- ✅ Custom agents and instructions
- ✅ Prompt engineering techniques
- ✅ MCP server integration

**Up Next: Part 4 - Enterprise & Team Collaboration**

---

## 📚 Additional Resources

- [Model Context Protocol Specification](https://modelcontextprotocol.io/)
- [VS Code MCP Documentation](https://code.visualstudio.com/docs/copilot/chat/mcp-servers)
- [MCP SDK (Node.js)](https://github.com/modelcontextprotocol/sdk)
- [MCP Servers Directory](https://github.com/modelcontextprotocol/servers)

---

**Previous**: [Section 3: Prompt Engineering](./03-prompt-engineering.md)  
**Next**: [Part 4: Enterprise & Team Collaboration](../04-enterprise-team/README.md)  
**Up**: [Part 3 README](./README.md)

