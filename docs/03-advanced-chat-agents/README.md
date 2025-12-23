# Part 3: Advanced Chat & Agent Development

**Target Audience**: Senior developers ready to master Copilot's chat and agent capabilities  
**Time to Complete**: 4 hours  
**Prerequisites**: Completed Parts 1 & 2

---

## 📋 Overview

This part teaches you how to become a GitHub Copilot power user by mastering its chat interface, building custom agents, engineering effective prompts, and extending capabilities with external tools through the Model Context Protocol (MCP).

By the end of this part, you'll be able to:
- Switch between chat modes strategically for different tasks
- Build a library of custom agents for your team's workflows
- Write prompts that consistently produce high-quality results
- Integrate external tools and services via MCP servers
- Create custom MCP servers for your specific needs

---

## 🗂️ Sections

### [Section 1: Chat Modes Mastery](./01-chat-modes-mastery.md)
**Time**: 1 hour

Master GitHub Copilot's four agent modes and learn when to use each one.

**What you'll learn:**
- **Ask mode**: Questions and exploration without code changes
- **Edit mode**: Targeted, surgical code modifications
- **Plan mode**: Strategic planning before implementation
- **Agent mode**: Autonomous multi-file development
- **Subagents**: Delegating complex subtasks
- **Mode switching strategies**: Choosing the right mode for the job
- **Common anti-patterns**: What not to do

**Key takeaways:**
- Plan first with Plan mode, implement with Agent mode
- Use Edit mode for single-file changes
- Subagents prevent context pollution
- Always review Agent's work before committing

---

### [Section 2: Custom Agents & Instructions](./02-custom-agents-instructions.md)
**Time**: 1.5 hours

Create specialized AI assistants and define repository-wide coding standards.

**What you'll learn:**
- **Custom instructions**: `.github/copilot-instructions.md` for team standards
- **Custom agents**: `.agent.md` files for specialized workflows
- **Building agent libraries**: Security, Performance, Documentation, Database, Testing agents
- **Sharing agents**: Team collaboration and versioning
- **Best practices**: Agent design principles

**Key takeaways:**
- Custom instructions apply to all chat interactions
- Custom agents are switchable personas for specific tasks
- Build a standard library of agents for common workflows
- Commit agents to `.github/agents/` for team sharing
- Use both instructions and agents together

**Standard agent library:**
- Security Reviewer (OWASP Top 10 audits)
- Performance Optimizer (bottleneck detection)
- Documentation Generator (API docs, README)
- Database Expert (schema design, query optimization)
- Test Generator (comprehensive test suites)
- API Designer (RESTful API design)

---

### [Section 3: Prompt Engineering for Copilot Chat](./03-prompt-engineering.md)
**Time**: 1 hour

Learn the art of crafting effective prompts for consistent, high-quality results.

**What you'll learn:**
- **Prompt structure**: Context + Task + Constraints
- **Context references**: `@workspace`, `@file`, `@selection`, `@terminal`
- **Output format control**: Structured text, tables, code-only, diagrams
- **Chain-of-thought prompting**: Step-by-step reasoning
- **Multi-shot examples**: Providing patterns to follow
- **Iterative refinement**: Progressive enhancement
- **Common anti-patterns**: What to avoid

**Key takeaways:**
- Effective prompts have three parts: context, task, constraints
- Use `@workspace` for codebase-wide questions
- Use `@file:path` for specific file context
- Specify output format explicitly
- Provide examples for consistency
- Iterate and refine progressively

**Prompt templates:**
- Security review checklist
- Performance optimization request
- Code review template
- Refactoring request
- Feature implementation template

---

### [Section 4: MCP Server Integration](./04-mcp-integration.md)
**Time**: 1.5 hours

Extend Copilot with external tools, databases, APIs, and custom services.

**What you'll learn:**
- **MCP architecture**: Client-server model for tool integration
- **Installing MCP servers**: User-level, workspace-level, dev containers
- **Popular servers**: Context7, GitHub, Playwright, PostgreSQL, Fetch
- **Custom MCP servers**: Building your own in Node.js or Python
- **Development mode**: File watching and debugging
- **Security**: Best practices for secrets and access control
- **Troubleshooting**: Common issues and solutions

**Key takeaways:**
- MCP extends Agent mode with external capabilities
- Context7 provides up-to-date library documentation
- GitHub server enables repository integration
- Create custom servers for internal tools
- Use environment variables for secrets
- Configure workspace MCP for team sharing

**Popular MCP servers:**
- **Context7**: Version-specific documentation
- **GitHub**: Repos, PRs, issues, commits
- **Playwright**: Browser automation
- **PostgreSQL**: Database operations
- **Memory**: Persistent context across sessions
- **Fetch**: HTTP requests to external APIs

---

## 🎯 Learning Path

### Hour 1: Master Chat Modes
1. Read Section 1: Chat Modes Mastery
2. Practice switching between modes
3. Try Plan → Agent workflow
4. Experiment with subagents

**Checkpoint**: Can explain when to use each mode

---

### Hour 2: Build Custom Agents
1. Read Section 2: Custom Agents & Instructions
2. Create `.github/copilot-instructions.md` for your project
3. Build your first custom agent (Security Reviewer recommended)
4. Test agents on real code

**Checkpoint**: Have working copilot-instructions.md and 1+ custom agent

---

### Hour 3: Perfect Your Prompts
1. Read Section 3: Prompt Engineering
2. Practice three-part prompt structure
3. Experiment with context references
4. Try chain-of-thought prompting
5. Use prompt templates

**Checkpoint**: Can write effective prompts consistently

---

### Hour 4: Integrate MCP Servers
1. Read Section 4: MCP Server Integration
2. Install Context7 and GitHub MCP servers
3. Configure workspace MCP (`.vscode/mcp.json`)
4. (Optional) Create a simple custom MCP server
5. Document MCP setup for team

**Checkpoint**: Have working MCP servers configured

---

## 🚀 Quick Wins

### 5-Minute Win: Switch Modes
```markdown
# Try each mode on a simple task
[Ask] "Explain this authentication code"
[Edit] Select code, Ctrl+I, "Add error handling"
[Plan] "Plan OAuth2 implementation"
[Agent] "Implement the plan"
```

### 15-Minute Win: First Custom Agent
```bash
# Create security reviewer agent
mkdir -p .github/agents
# Copy template from Section 2
# Test it: "Review this code for security issues"
```

### 30-Minute Win: Perfect Prompt
```markdown
# Use three-part structure
@file:src/api/users.ts

Refactor this endpoint to:
1. Add input validation with Zod
2. Implement rate limiting
3. Add comprehensive error handling

Requirements:
- Follow existing API patterns
- Add unit tests
- No breaking changes
```

### 1-Hour Win: MCP Integration
```json
// Add to .vscode/mcp.json
{
  "servers": {
    "context7": {
      "type": "http",
      "url": "https://mcp.context7.com/mcp"
    }
  }
}
```
Test: "Implement Prisma migrations. use context7"

---

## 💼 Real-World Scenarios

### Scenario 1: Implementing a New Feature

```markdown
# Step 1: Plan (Plan mode)
"Plan implementation of real-time notifications with WebSockets"

# Step 2: Review plan, approve

# Step 3: Implement (Agent mode)
"Implement the approved plan"

# Step 4: Refine (Edit mode)
Select specific function, "Add error recovery for disconnections"

# Step 5: Validate (Ask mode)
"@workspace Are there any edge cases we're not handling?"
```

---

### Scenario 2: Security Audit

```markdown
# Step 1: Switch to Security Reviewer agent

# Step 2: Comprehensive audit
"@workspace Review the authentication system for OWASP Top 10 vulnerabilities"

# Step 3: Fix critical issues (Agent mode)
"Fix the SQL injection vulnerability identified in the user search endpoint"

# Step 4: Verify (Security Reviewer again)
"Re-audit the user search endpoint"
```

---

### Scenario 3: Database Schema Design

```markdown
# Step 1: Use Database Expert agent

# Step 2: Gather requirements
"@workspace Analyze the current user data access patterns"

# Step 3: Design (Plan mode + Database Expert)
"Design an optimized schema for multi-tenant SaaS with row-level security"

# Step 4: MCP Integration (if PostgreSQL server configured)
"Compare the proposed schema with our current production schema"

# Step 5: Implement (Agent mode)
"Generate Prisma schema and migration files"
```

---

### Scenario 4: Performance Optimization

```markdown
# Step 1: Performance Optimizer agent

# Step 2: Analyze
"@workspace Analyze performance of the product listing page"

# Step 3: Prioritize
Agent identifies: N+1 query (high impact), unoptimized images (medium), missing cache (medium)

# Step 4: Fix high-impact issue (Agent mode)
"Fix the N+1 query in product listing with eager loading"

# Step 5: Verify
"@workspace Explain the query execution plan now"
```

---

## 📊 Progress Checklist

### Mastery Indicators

**Chat Modes:**
- [ ] Can explain when to use each of the 4 modes
- [ ] Have used Plan → Agent workflow successfully
- [ ] Understand subagents and when to use them
- [ ] Can switch modes mid-conversation

**Custom Agents:**
- [ ] Created `.github/copilot-instructions.md` for project
- [ ] Built at least 2 custom agents
- [ ] Shared agents with team (committed to repo)
- [ ] Documented agent library

**Prompt Engineering:**
- [ ] Write prompts with Context + Task + Constraints
- [ ] Use context references (@workspace, @file, etc.)
- [ ] Can specify output formats
- [ ] Apply chain-of-thought for complex problems
- [ ] Have personal prompt template library

**MCP Integration:**
- [ ] Installed and configured 2+ MCP servers
- [ ] Have workspace MCP configuration (`.vscode/mcp.json`)
- [ ] Documented MCP setup for team
- [ ] (Bonus) Created custom MCP server

---

## 🎓 Deliverables

By completing Part 3, you should have:

### 1. Workspace Configuration
```
your-repo/
├── .github/
│   ├── copilot-instructions.md      ← Team coding standards
│   └── agents/
│       ├── security-reviewer.agent.md
│       ├── performance-optimizer.agent.md
│       ├── doc-generator.agent.md
│       └── README.md                 ← Agent library docs
└── .vscode/
    ├── mcp.json                      ← MCP server config
    └── MCP_SETUP.md                  ← Setup instructions
```

### 2. Personal Skills
- Strategic mode switching
- Effective prompt engineering
- Custom agent creation
- MCP server integration

### 3. Team Resources
- Shared custom instructions
- Agent library (3-6 agents)
- MCP server configuration
- Documentation for team adoption

---

## 🔗 Navigation

**Previous**: [Part 2: VS Code Integration](../02-vscode-integration/README.md)  
**Next**: [Part 4: Enterprise & Team Collaboration](../04-enterprise-team/README.md)  
**Home**: [Guide Home](../../README.md)

---

## 📚 Additional Resources

### Official Documentation
- [VS Code Copilot Chat](https://code.visualstudio.com/docs/copilot/chat/)
- [GitHub Copilot Agent Modes](https://code.visualstudio.com/docs/copilot/copilot-coding-agent)
- [Custom Agents Documentation](https://code.visualstudio.com/docs/copilot/customization/custom-agents)
- [Model Context Protocol](https://modelcontextprotocol.io/)

### Community Resources
- [MCP Servers Directory](https://github.com/modelcontextprotocol/servers)
- [Awesome Copilot Instructions](https://github.com/code-and-sorts/awesome-copilot-instructions)
- [Prompt Engineering Guide](https://www.promptingguide.ai/)

---

## 💡 Tips for Success

1. **Start with Plan mode** for complex features—it saves time
2. **Build your agent library gradually**—start with Security Reviewer
3. **Document your prompts**—create a template library
4. **Share with your team**—commit agents and instructions to Git
5. **Iterate on agents**—they improve with feedback
6. **Use MCP for real data**—Context7 and GitHub are game-changers
7. **Review Agent's work**—always verify before committing

---

## ❓ Common Questions

**Q: Which mode should I default to?**
A: Ask mode for questions, Edit/Inline Chat for single-file changes, Plan+Agent for complex features.

**Q: How many custom agents should I create?**
A: Start with 3-5: Security, Performance, Documentation, Database, Testing.

**Q: Are custom instructions always active?**
A: Yes, they apply to all chat interactions in that repository.

**Q: Can I use MCP servers in custom agents?**
A: Yes! Reference MCP tools in agent configuration with `tools: ["mcp-server/*"]`.

**Q: What's the difference between custom agents and MCP?**
A: Custom agents are AI personas; MCP servers provide tools (external capabilities).

**Q: Should I commit MCP configuration?**
A: Yes, commit `.vscode/mcp.json` but use environment variables for secrets.

---

**Part 3 Complete!** 🎉

You've mastered advanced Copilot features. You can now:
- ✅ Switch between chat modes strategically
- ✅ Build and share custom agents
- ✅ Write effective prompts consistently
- ✅ Integrate external tools via MCP

**Ready for Part 4?** Learn how to deploy Copilot across your entire organization.

---

**Last Updated**: December 2025  
**Version**: 1.0

