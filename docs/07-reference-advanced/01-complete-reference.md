---
title: "Section 1: Complete Reference Guide"
parent: "Part 7: Reference, Troubleshooting & Advanced"
nav_order: 1
---

# Section 1: Complete Reference Guide

**Part 7: Reference, Troubleshooting & Advanced**

---

## 📋 Overview

This section provides a comprehensive, copy-paste reference for all GitHub Copilot features, commands, and configurations. Bookmark this page for quick access to keyboard shortcuts, slash commands, context references, settings, and file format specifications.

**What you'll find here:**
- Keyboard shortcuts cheat sheet
- Slash commands reference
- Context references (@workspace, @file, etc.)
- Settings reference
- Custom instructions syntax
- Agent file format specification
- Prompt file format
- MCP configuration reference

**Time to read:** 30 minutes (reference material - use as needed)

---

## ⌨️ Keyboard Shortcuts Cheat Sheet

### Essential Shortcuts (Memorize These)

| Action | Windows/Linux | macOS | Description |
|--------|---------------|-------|-------------|
| **Accept suggestion** | `Tab` | `Tab` | Accept inline completion |
| **Reject suggestion** | `Esc` | `Esc` | Dismiss current suggestion |
| **Next suggestion** | `Alt+]` | `Option+]` | Cycle to next suggestion |
| **Previous suggestion** | `Alt+[` | `Option+[` | Cycle to previous suggestion |
| **All suggestions** | `Ctrl+Enter` | `Cmd+Enter` | Open suggestions panel (see all options) |
| **Inline chat** | `Ctrl+I` | `Cmd+I` | Open inline chat in editor |
| **Chat view** | `Ctrl+Alt+I` | `Cmd+Option+I` | Open dedicated Chat view |
| **Quick chat** | `Ctrl+Shift+Alt+L` | `Cmd+Shift+Option+L` | Quick chat overlay |

### Advanced Shortcuts

| Action | Windows/Linux | macOS | Description |
|--------|---------------|-------|-------------|
| **Explain this** | `Ctrl+I` then type | `Cmd+I` then type | Explain selected code |
| **Fix this** | `Ctrl+I` then `/fix` | `Cmd+I` then `/fix` | Fix selected code |
| **Generate tests** | `Ctrl+I` then `/tests` | `Cmd+I` then `/tests` | Generate tests for selection |
| **Start debugging** | `Ctrl+I` then `/startDebugging` | `Cmd+I` then `/startDebugging` | Debug current file |
| **Open keyboard shortcuts editor** | `Ctrl+K Ctrl+S` | `Cmd+K Cmd+S` | Customize shortcuts |

### Chat Navigation Shortcuts

| Action | Windows/Linux | macOS | Description |
|--------|---------------|-------|-------------|
| **Focus chat input** | `Ctrl+L` (in Chat view) | `Cmd+L` | Focus on chat input field |
| **Submit prompt** | `Enter` | `Enter` | Send chat message |
| **New line in prompt** | `Shift+Enter` | `Shift+Enter` | Add line break in prompt |
| **Clear chat** | Type `/clear` | Type `/clear` | Clear chat history |
| **Copy code block** | Hover + Click Copy | Hover + Click Copy | Copy code from response |

### Inline Suggestion Shortcuts

| Action | Windows/Linux | macOS | Description |
|--------|---------------|-------|-------------|
| **Accept word** | `Ctrl+Right` | `Cmd+Right` | Accept next word of suggestion |
| **Accept line** | - | - | (Use Tab for full line) |
| **Trigger manually** | `Alt+\` | `Option+\` | Manually trigger completion |

---

## 💬 Slash Commands Reference

### Built-in Slash Commands

Type `/` in the chat input to see all available commands. Here are the most important ones:

#### Code Generation & Modification

| Command | Description | Example Use Case |
|---------|-------------|------------------|
| `/doc` | Generate documentation | Generate JSDoc for selected function |
| `/explain` | Explain code or concept | Understand unfamiliar code |
| `/fix` | Fix problems in code | Resolve linter errors or bugs |
| `/tests` | Generate test cases | Create unit tests for function |
| `/setupTests` | Setup testing framework | Initialize Jest or Pytest |
| `/fixTestFailure` | Fix failing tests | Debug test failures |
| `/new` | Scaffold new project/file | Create new React component |
| `/newNotebook` | Create Jupyter notebook | Start data science workflow |

#### Workflow Commands

| Command | Description | Example Use Case |
|---------|-------------|------------------|
| `/clear` | Clear chat history | Start fresh conversation |
| `/help` | Show available commands | Discover all commands |
| `/search` | Generate search query | Find code across workspace |
| `/startDebugging` | Start debugging session | Debug current file |

### Custom Slash Commands

Create custom slash commands using **prompt files** (`.prompt.md`):

**Location:** `.vscode/prompts/` or `~/.vscode/prompts/` (user profile)

**Example:** `.vscode/prompts/review-security.prompt.md`

```markdown
---
description: Comprehensive security audit
---

Perform a security audit of the selected code:
1. Identify all input sources
2. Check for injection vulnerabilities (SQL, XSS, command injection)
3. Verify authentication and authorization checks
4. Look for secrets or hardcoded credentials
5. Review error handling for information leakage
6. Check for insecure dependencies

Provide findings in this format:
- 🔴 **Critical**: [description]
- 🟡 **Warning**: [description]
- 🟢 **Good**: [description]
```

**Usage:** Type `/review-security` in Chat view

---

## 🔗 Context References

### Context Reference Syntax

Type `#` in chat to see all available context references. Use these to give Copilot precise context.

#### File & Folder References

| Reference | Syntax | Example |
|-----------|--------|---------|
| **Specific file** | `#file:path/to/file` | `#file:src/auth.ts` |
| **Folder** | `#folder:path/to/folder` | `#folder:src/components` |
| **Current file** | `#file` | Automatically includes active file |
| **Code selection** | `#selection` | References selected text |

**Pro tip:** You can also **drag and drop** files from Explorer into the chat input!

#### Workspace & Codebase References

| Reference | Syntax | Description | When to Use |
|-----------|--------|-------------|-------------|
| **Entire workspace** | `#codebase` | All indexed files | "Find all API endpoints" |
| **Workspace context** | `@workspace` | Intelligent workspace search | Most questions about your code |

**Difference between `#codebase` and `@workspace`:**
- `#codebase`: Explicit reference to entire codebase (forces workspace search)
- `@workspace`: Smart context - Copilot decides what files are relevant

#### Code Structure References

| Reference | Syntax | Example |
|-----------|--------|---------|
| **Symbol** | `#symbolName` | `#UserController` (class/function/type) |
| **Usages** | `#usages` | Find all references to symbol |

**Note:** To reference a symbol, ensure the file containing it is open in the editor.

#### Development Context References

| Reference | Syntax | Description |
|-----------|--------|-------------|
| **Terminal output** | `#terminalSelection` | Currently selected terminal text |
| **Terminal last command** | `#terminalLastCommand` | Output of last terminal command |
| **Test failures** | `#testFailure` | Failed test context |
| **Problems** | `#problems` | Linter/compiler errors |
| **Source control changes** | `#scmChanges` | Staged or unstaged changes |

#### Tool References

| Reference | Syntax | Description |
|-----------|--------|-------------|
| **Fetch web content** | `#fetch <url>` | Retrieve content from URL |
| **GitHub repo search** | `#githubRepo <owner/repo>` | Search public GitHub repository |
| **List directory** | `#listDirectory` | List files in directory |

**Example:**
```
How does routing work? #githubRepo vercel/next.js
```

### Context Best Practices

**DO ✅:**
- Use `@workspace` for most questions about your code
- Reference specific files when you know exactly what's relevant
- Use `#codebase` when you need comprehensive workspace search
- Combine multiple context types: `@workspace #file:config.ts #terminalLastCommand`

**DON'T ❌:**
- Overload with too many file references (Copilot may hit token limits)
- Forget that implicit context (current file, selection) is automatically included
- Use `#codebase` for simple questions (it's slower and more expensive)

---

## ⚙️ Settings Reference

### Essential Settings

Configure these in VS Code settings (`Ctrl+,` or `Cmd+,`):

#### Inline Suggestions Settings

| Setting | Default | Description |
|---------|---------|-------------|
| `github.copilot.enable` | `{"*": true}` | Enable Copilot for all languages or specific ones |
| `editor.inlineSuggest.enabled` | `true` | Enable inline suggestions |
| `editor.inlineSuggest.showToolbar` | `"onHover"` | Show toolbar with suggestions |
| `editor.inlineSuggest.syntaxHighlightingEnabled` | `true` | Syntax highlighting in suggestions |

**Example: Enable only for specific languages**

```json
{
  "github.copilot.enable": {
    "*": false,
    "python": true,
    "javascript": true,
    "typescript": true
  }
}
```

#### Chat Settings

| Setting | Default | Description |
|---------|---------|-------------|
| `github.copilot.chat.enabled` | `true` | Enable Copilot Chat |
| `inlineChat.finishOnType` | `false` | Auto-finish inline chat when typing |
| `inlineChat.holdToSpeech` | `true` | Hold `Ctrl+I` for voice input |
| `chat.promptFiles` | `false` | Enable custom prompt files |

#### Advanced Chat Settings

| Setting | Description |
|---------|-------------|
| `github.copilot.chat.pullRequestDescriptionGeneration.instructions` | Custom instructions for PR descriptions |
| `github.copilot.chat.reviewSelection.instructions` | Custom instructions for code review |
| `github.copilot.chat.codeGeneration.instructions` | Instructions for code generation |

**Example: Custom PR description instructions**

```json
{
  "github.copilot.chat.pullRequestDescriptionGeneration.instructions": [
    { "text": "Always include a list of key changes." },
    { "text": "Add testing notes section." },
    { "file": ".github/pr-template.md" }
  ]
}
```

#### Context & Performance Settings

| Setting | Default | Description |
|---------|---------|-------------|
| `github.copilot.advanced.debug` | `false` | Enable debug logging |
| `github.copilot.advanced.telemetry` | `"enabled"` | Telemetry collection |

#### Workspace Trust Settings

| Setting | Default | Description |
|---------|---------|-------------|
| `security.workspace.trust.enabled` | `true` | Enable workspace trust |

**For sensitive projects:** Disable Copilot in untrusted workspaces:

```json
{
  "github.copilot.enable": {
    "*": false
  }
}
```

### Settings Hierarchy

Settings are applied in this order (later overrides earlier):

1. **Default settings** (built-in)
2. **User settings** (`~/.config/Code/User/settings.json`)
3. **Workspace settings** (`.vscode/settings.json`)
4. **Folder settings** (in multi-root workspaces)

**Recommendation:** Use **workspace settings** for team-shared configuration.

---

## 📝 Custom Instructions Syntax

### Basic Custom Instructions

**File:** `.github/copilot-instructions.md`

**Purpose:** Provide project-specific context and coding standards to Copilot.

**Example:**

```markdown
# Project: E-Commerce Platform

## Tech Stack
- **Frontend**: React 18 with TypeScript
- **Backend**: Node.js 20 + Express
- **Database**: PostgreSQL 15 with Prisma ORM
- **Testing**: Jest + React Testing Library

## Coding Standards
- Use functional components with hooks (no class components)
- All API endpoints must have OpenAPI documentation
- Error handling via centralized error middleware
- Async/await only (no .then() chains)
- Prefer arrow functions over function declarations

## Naming Conventions
- Components: PascalCase (e.g., `UserProfile.tsx`)
- Functions: camelCase (e.g., `getUserById`)
- Constants: UPPER_SNAKE_CASE (e.g., `API_BASE_URL`)
- Files: kebab-case (e.g., `user-profile.tsx`)

## Test Requirements
- 80% code coverage minimum
- Unit tests for all business logic
- Integration tests for API endpoints
- E2E tests for critical user flows

## Security Requirements
- Input validation with Zod schemas
- SQL injection prevention via Prisma (no raw queries)
- XSS protection with DOMPurify
- Rate limiting on all public endpoints
- Authentication using JWT with refresh tokens
- Secrets in environment variables only
```

### Language-Specific Instructions

**File pattern:** `.github/copilot-instructions.md` with frontmatter

**Specify `applyTo` glob pattern:**

```markdown
---
applyTo: "**/*.ts"
---
# TypeScript-Specific Instructions

- Always use explicit return types for functions
- Prefer `interface` over `type` for object shapes
- Use `readonly` for immutable properties
- Enable strict mode checks
```

```markdown
---
applyTo: "**/*.test.ts"
---
# Test File Instructions

- Use `describe` blocks to group related tests
- Test names should be descriptive sentences
- Always test happy path, edge cases, and error cases
- Use `beforeEach` for test setup
- Mock external dependencies
```

### Feature-Specific Instructions

**File pattern:** `.github/copilot-instructions-<feature>.md`

**Example:** `.github/copilot-instructions-security.md`

```markdown
---
applyTo: "src/api/**/*.ts"
---
# API Security Instructions

## Authentication
- All API endpoints require authentication unless explicitly marked public
- Use `authenticate` middleware for protected routes
- Validate JWT tokens on every request

## Input Validation
- Validate all request bodies with Zod schemas
- Sanitize user input to prevent XSS
- Use parameterized queries to prevent SQL injection

## Error Responses
- Never expose stack traces in production
- Return generic error messages to clients
- Log detailed errors server-side only
```

---

## 🤖 Agent File Format Specification

### Agent File Structure

**File pattern:** `.vscode/agents/<name>.agent.md`

**Format:** Markdown with YAML frontmatter

### Complete Agent Specification

```markdown
---
name: Agent Name
description: Brief description of what this agent does
tools:
  - read_files
  - list_directory
  - search
  - fetch
  - usages
  - githubRepo
handoffs:
  - label: Button text for handoff
    agent: target-agent-name
    prompt: Optional prompt to send to target agent
    send: true|false
---

# Agent Instructions

Your detailed instructions for the agent go here.

## Agent Persona

You are a [role] specializing in [domain].

## Focus Areas

- Focus area 1
- Focus area 2
- Focus area 3

## Workflow

1. Step 1
2. Step 2
3. Step 3

## Output Format

Provide results in this structure:
- Category 1: [description]
- Category 2: [description]

## Examples

### Example 1
**Input:** [description]
**Output:** [expected output]

### Example 2
**Input:** [description]
**Output:** [expected output]
```

### Available Tools

| Tool | Description | Use Case |
|------|-------------|----------|
| `read_files` | Read file contents | Code review, analysis |
| `list_directory` | List directory contents | Project exploration |
| `search` | Search workspace | Find patterns, symbols |
| `fetch` | Fetch web content | Get documentation |
| `usages` | Find symbol references | Dependency analysis |
| `githubRepo` | Search GitHub repos | Research best practices |
| `problems` | Access linter errors | Quality checks |
| `runSubagent` | Delegate to subagent | Complex workflows |

### Agent Examples

#### Security Review Agent

```markdown
---
name: Security Reviewer
description: Specialized agent for security audits and threat modeling
tools:
  - read_files
  - list_directory
  - search
  - problems
---

# Security Review Agent

You are a security expert reviewing code for vulnerabilities.

## Focus Areas

- **Injection vulnerabilities**: SQL, XSS, command injection
- **Authentication bypasses**: Broken auth, session management
- **Data exposure**: Sensitive data in logs, responses, client-side code
- **Dependency vulnerabilities**: Outdated or vulnerable packages
- **Configuration issues**: Exposed secrets, debug mode in production

## Review Process

1. Analyze authentication and authorization flows
2. Identify all user input points
3. Check for injection vulnerabilities
4. Review error handling and logging
5. Verify secret management
6. Check dependency versions

## Output Format

Provide findings categorized by severity:
- 🔴 **Critical**: Immediate security risk requiring urgent fix
- 🟡 **Warning**: Potential security issue requiring attention
- 🟢 **Good**: Security best practice correctly implemented
- 💡 **Suggestion**: Recommendation for security improvement

For each finding, include:
- **Location**: File and line number
- **Description**: What the issue is
- **Impact**: Potential consequences
- **Recommendation**: How to fix it
- **Example**: Code snippet if applicable
```

#### Performance Optimization Agent

```markdown
---
name: Performance Optimizer
description: Analyzes code for performance bottlenecks and optimizations
tools:
  - read_files
  - search
  - usages
handoffs:
  - label: Implement Optimizations
    agent: implementation
    prompt: Implement the performance optimizations outlined above
    send: false
---

# Performance Optimization Agent

You are a performance expert analyzing code for optimization opportunities.

## Analysis Areas

### Frontend Performance
- Bundle size analysis
- Component render optimization
- Lazy loading opportunities
- Image optimization
- Code splitting

### Backend Performance
- Database query optimization
- Caching strategies
- API response times
- Memory usage
- Algorithm complexity

## Workflow

1. Identify performance bottlenecks
2. Analyze algorithmic complexity
3. Review database queries
4. Check for unnecessary re-renders (React)
5. Suggest caching strategies
6. Provide implementation recommendations

## Output Format

### Performance Issues
- 🔴 **Critical**: Severe performance impact
- 🟡 **Moderate**: Noticeable impact
- 🟢 **Minor**: Small optimization opportunity

### Each Issue
- **Location**: File and function
- **Issue**: Description of bottleneck
- **Current Complexity**: O(n^2), etc.
- **Impact**: User experience impact
- **Solution**: Specific optimization
- **Expected Improvement**: Estimated gain
```

---

## 📄 Prompt File Format

### Prompt File Structure

**File pattern:** `.vscode/prompts/<name>.prompt.md`

**Format:** Markdown with YAML frontmatter

### Complete Prompt Specification

```markdown
---
description: Brief description shown in command palette
agent: optional-agent-name
---

Your prompt instructions here.

You can use variables:
- ${workspaceFolder}
- ${workspaceFolderBasename}
- ${file}
- ${fileBasename}
- ${fileDirname}
- ${selection}
- ${selectedText}
- ${input:variableName}
- ${input:variableName:placeholder}

You can reference files using Markdown links:
[Configuration](./config.json)

You can reference tools:
#tool:githubRepo
#tool:fetch
```

### Prompt File Examples

#### Code Review Prompt

**File:** `.vscode/prompts/review-pr.prompt.md`

```markdown
---
description: Review pull request changes comprehensively
---

Review the following code changes for:

## Code Quality
- Readability and maintainability
- Adherence to coding standards
- Proper error handling
- Code duplication

## Functionality
- Logic correctness
- Edge case handling
- Potential bugs

## Performance
- Algorithmic efficiency
- Resource usage
- Database query optimization

## Security
- Input validation
- Authentication/authorization
- Data sanitization
- Secrets management

## Testing
- Test coverage
- Test quality
- Missing test cases

Provide feedback in this format:
- ✅ **Approved**: [what's good]
- 💡 **Suggestions**: [improvements]
- ⚠️ **Concerns**: [issues to address]
- ❌ **Blockers**: [must fix before merge]
```

#### Refactoring Prompt

**File:** `.vscode/prompts/refactor-legacy.prompt.md`

```markdown
---
description: Refactor legacy code to modern patterns
agent: refactoring
---

Refactor the selected code to modern best practices:

## Modernization Goals
1. Convert to ES6+ syntax (arrow functions, destructuring, etc.)
2. Replace callbacks with async/await
3. Add TypeScript types if applicable
4. Improve error handling
5. Extract reusable functions
6. Add comprehensive documentation

## Constraints
- Maintain existing functionality exactly
- Keep public API unchanged
- Ensure backward compatibility
- Add tests for refactored code

## Output
- Original code analysis
- Step-by-step refactoring plan
- Refactored code with explanations
- Test cases for verification
```

#### Documentation Generation Prompt

**File:** `.vscode/prompts/document-api.prompt.md`

```markdown
---
description: Generate comprehensive API documentation
---

Generate detailed API documentation for the selected code:

## Include
- Function/method signature with types
- Purpose and description
- Parameters (name, type, description, required/optional)
- Return value (type, description)
- Throws/Errors (when and why)
- Usage examples (at least 2)
- Edge cases and special behaviors

## Format
Use JSDoc for JavaScript/TypeScript:
```typescript
/**
 * @description Brief description
 * @param {type} name - Description
 * @returns {type} Description
 * @throws {ErrorType} When this happens
 * @example
 * // Example usage
 * const result = functionName(arg1, arg2);
 */
```

For Python, use Google-style docstrings:
```python
"""
Brief description.

Args:
    param1 (type): Description
    param2 (type): Description

Returns:
    type: Description

Raises:
    ExceptionType: When this happens

Examples:
    >>> function_name(arg1, arg2)
    expected_output
"""
```
```

---

## 🔌 MCP Configuration Reference

### MCP Server Configuration Format

**File:** `.vscode/mcp.json` (workspace) or `~/.vscode/mcp.json` (user)

**Format:** JSON

### Configuration Structure

```json
{
  "servers": {
    "server-name": {
      "type": "stdio",
      "command": "command-to-run",
      "args": ["arg1", "arg2"],
      "env": {
        "ENV_VAR": "value"
      }
    }
  },
  "inputs": [
    {
      "id": "api-key",
      "type": "secret",
      "label": "API Key",
      "description": "Enter your API key"
    }
  ]
}
```

### Transport Types

#### 1. Standard I/O (stdio)

**Use for:** Node.js or Python MCP servers

```json
{
  "servers": {
    "my-mcp-server": {
      "type": "stdio",
      "command": "node",
      "args": ["/path/to/server.js"],
      "env": {
        "API_KEY": "${input:api-key}"
      }
    }
  }
}
```

#### 2. Server-Sent Events (SSE)

**Use for:** HTTP-based MCP servers

```json
{
  "servers": {
    "remote-mcp": {
      "type": "sse",
      "url": "https://api.example.com/mcp",
      "headers": {
        "Authorization": "Bearer ${input:api-token}"
      }
    }
  }
}
```

### Complete MCP Configuration Examples

#### Example 1: GitHub MCP Server

```json
{
  "servers": {
    "github": {
      "type": "stdio",
      "command": "npx",
      "args": [
        "-y",
        "@modelcontextprotocol/server-github"
      ],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${input:github-token}"
      }
    }
  },
  "inputs": [
    {
      "id": "github-token",
      "type": "secret",
      "label": "GitHub Personal Access Token",
      "description": "Enter your GitHub PAT with repo access"
    }
  ]
}
```

#### Example 2: Custom Python MCP Server

```json
{
  "servers": {
    "data-analyzer": {
      "type": "stdio",
      "command": "python",
      "args": [
        "${workspaceFolder}/mcp-servers/analyzer.py"
      ],
      "env": {
        "DATABASE_URL": "${input:db-url}",
        "LOG_LEVEL": "info"
      }
    }
  },
  "inputs": [
    {
      "id": "db-url",
      "type": "secret",
      "label": "Database URL",
      "description": "PostgreSQL connection string"
    }
  ]
}
```

#### Example 3: Multiple MCP Servers

```json
{
  "servers": {
    "github": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-github"],
      "env": {
        "GITHUB_PERSONAL_ACCESS_TOKEN": "${input:github-token}"
      }
    },
    "playwright": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@playwright/mcp-server"]
    },
    "context7": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@upstash/context7-mcp"],
      "env": {
        "CONTEXT7_API_KEY": "${input:context7-key}"
      }
    }
  },
  "inputs": [
    {
      "id": "github-token",
      "type": "secret",
      "label": "GitHub Token",
      "description": "GitHub Personal Access Token"
    },
    {
      "id": "context7-key",
      "type": "secret",
      "label": "Context7 API Key",
      "description": "Your Context7 API key from https://context7.com"
    }
  ]
}
```

### Input Variables

| Type | Description | Use Case |
|------|-------------|----------|
| `secret` | Securely store API keys | Authentication tokens |
| `text` | Plain text input | Non-sensitive configuration |

### Predefined Variables

Use these in your MCP configuration:

| Variable | Description | Example |
|----------|-------------|---------|
| `${workspaceFolder}` | Workspace root path | `/path/to/project` |
| `${workspaceFolderBasename}` | Workspace folder name | `my-project` |
| `${input:id}` | User input variable | `${input:api-key}` |

### MCP Server Management Commands

| Command | Description |
|---------|-------------|
| `MCP: Add Server` | Add new MCP server |
| `MCP: List Servers` | View all configured servers |
| `MCP: Start Server` | Start a stopped server |
| `MCP: Stop Server` | Stop a running server |
| `MCP: View Logs` | View server logs |
| `MCP: Open User Configuration` | Edit user MCP config |
| `MCP: Open Workspace Folder Configuration` | Edit workspace MCP config |

---

## 📚 Quick Reference Tables

### Chat Modes Decision Matrix

| Your Goal | Use This Mode | Why |
|-----------|---------------|-----|
| Ask question | **Ask** | Read-only, explanations |
| Edit 1 file | **Edit** or Inline Chat | Targeted changes |
| Plan feature | **Plan** | Creates roadmap, no changes |
| Multi-file feature | **Agent** | Autonomous implementation |
| Complex analysis | **Subagent** | Isolated context |

### Model Selection Quick Guide

| Task Type | Recommended Model | Why |
|-----------|------------------|-----|
| Don't know | **Auto** ⭐ | Let Copilot choose (recommended) |
| Daily coding | Claude Sonnet 4.5 | Balanced performance |
| Quick questions | Claude Haiku 4.5 | Fastest responses |
| Complex architecture | Claude Opus 4.5 or GPT-5 | Deepest reasoning |
| Multi-file refactoring | Claude Sonnet 4.5 | Excellent at this |
| Long context | Gemini 2.5 Pro | 1M+ tokens |
| Fast reasoning | GPT-5 mini | Speed + intelligence |

### File Format Summary

| File Type | Location | Purpose | Extension |
|-----------|----------|---------|-----------|
| **Custom Instructions** | `.github/` | Project coding standards | `.md` |
| **Custom Agent** | `.vscode/agents/` | Specialized AI assistants | `.agent.md` |
| **Prompt File** | `.vscode/prompts/` | Reusable prompts | `.prompt.md` |
| **MCP Configuration** | `.vscode/` or `~/` | MCP server setup | `mcp.json` |

---

## 💡 Pro Tips

### Keyboard Efficiency

1. **Memorize the big 3:**
   - `Ctrl+I` (inline chat)
   - `Ctrl+Alt+I` (Chat view)
   - `Tab` (accept suggestion)

2. **Use `Ctrl+Enter` liberally** to see all suggestion alternatives

3. **Rebind if needed:**
   - `Ctrl+K Ctrl+S` → Search "copilot" → Customize shortcuts

### Context Mastery

1. **Start with `@workspace`** for most questions
2. **Use drag-and-drop** instead of typing file paths
3. **Chain context types:**
   ```
   @workspace #file:config.ts #terminalLastCommand Fix the error
   ```

### Prompt File Power

1. **Create prompts for recurring tasks:**
   - `/review-security`
   - `/document-api`
   - `/refactor-legacy`

2. **Share prompts with team** via `.vscode/prompts/` in version control

3. **Use variables** for flexibility:
   ```markdown
   Review ${file} for ${input:review-type:code quality}
   ```

### Agent Workflow

1. **Create agent for each specialized task:**
   - Security review
   - Performance optimization
   - Documentation generation
   - Test coverage analysis

2. **Use handoffs** to chain agents together
3. **Store agents in `.vscode/agents/`** and commit to repo

### MCP Best Practices

1. **Store credentials in inputs**, never hardcode
2. **Use workspace MCP config** for team-shared servers
3. **Test servers** with `MCP: List Servers` → Start/View Logs
4. **Sync MCP config** across devices with Settings Sync

---

## 🔖 Copy-Paste Templates

### Minimal Custom Instructions

```markdown
# Project: [Project Name]

## Tech Stack
- [Technology 1]
- [Technology 2]

## Coding Standards
- [Standard 1]
- [Standard 2]
```

### Basic Agent

```markdown
---
name: My Agent
description: What this agent does
tools:
  - read_files
  - search
---

# My Agent

You are a [role] specializing in [domain].

## Focus
- [Focus area 1]
- [Focus area 2]
```

### Simple Prompt File

```markdown
---
description: What this prompt does
---

Your prompt instructions here.
```

### MCP Server (stdio)

```json
{
  "servers": {
    "my-server": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "package-name"],
      "env": {
        "API_KEY": "${input:api-key}"
      }
    }
  },
  "inputs": [
    {
      "id": "api-key",
      "type": "secret",
      "label": "API Key"
    }
  ]
}
```

---

## 📖 Next Steps

Now that you have the complete reference:

1. **Bookmark this page** for quick access
2. **Customize your shortcuts** (`Ctrl+K Ctrl+S`)
3. **Create your first custom agent** (Section 2 of Part 3)
4. **Set up MCP servers** (Section 4 of Part 3)
5. **Troubleshoot issues** (Next section)

**Next:** [Section 2: Troubleshooting & FAQs](02-troubleshooting-faqs.md)

---

**Part 7: Reference, Troubleshooting & Advanced**  
Section 1 of 2 | [Next: Troubleshooting →](02-troubleshooting-faqs.md)

