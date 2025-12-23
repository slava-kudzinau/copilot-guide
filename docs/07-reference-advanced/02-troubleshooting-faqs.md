# Section 2: Troubleshooting & FAQs

**Part 7: Reference, Troubleshooting & Advanced**

---

## 📋 Overview

This section provides solutions to common GitHub Copilot issues, troubleshooting guides, and answers to frequently asked questions. Use this as your first stop when encountering problems.

**What you'll find here:**
- Common issues and solutions
- Extension troubleshooting
- Network and connectivity problems
- Authentication issues
- Performance optimization
- Frequently asked questions

**Time to read:** 20 minutes (reference material - use as needed)

---

## 🔧 Common Issues & Solutions

### Issue 1: Extension Not Activating

**Symptoms:**
- Copilot icon not visible in status bar
- No inline suggestions appearing
- Chat view not available

**Solutions:**

#### Step 1: Check VS Code Version
```bash
# Check your VS Code version
code --version
```

**Required:** VS Code 1.98.0 or higher

**Fix:** Update VS Code to the latest version
- Help → Check for Updates
- Or download from https://code.visualstudio.com

#### Step 2: Verify Extension Installation
1. Open Extensions view (`Ctrl+Shift+X`)
2. Search for "GitHub Copilot"
3. Ensure **both** extensions are installed:
   - GitHub Copilot
   - GitHub Copilot Chat

**Fix:** Install or reinstall extensions
```
1. Extensions view (Ctrl+Shift+X)
2. Search "GitHub Copilot"
3. Click "Install" or "Reinstall"
4. Reload VS Code
```

#### Step 3: Check Extension Status
1. Click Copilot icon in status bar (bottom right)
2. Look for status message

**Possible statuses:**
- ✅ **Active**: Working correctly
- ⚠️ **Inactive**: Click to activate
- ❌ **Error**: See error message

#### Step 4: Restart Extension
**Command Palette (`Ctrl+Shift+P`):**
```
> Developer: Reload Window
```

**Or:**
```
> GitHub Copilot: Restart Extension
```

#### Step 5: Check Output Logs
**View logs:**
1. View → Output (`Ctrl+Shift+U`)
2. Select "GitHub Copilot" or "GitHub Copilot Chat" from dropdown

**Look for:**
- Error messages
- Authentication failures
- Network issues

---

### Issue 2: Authentication Failures

**Symptoms:**
- "GitHub Copilot could not connect to server"
- "Extension activation failed"
- "Authentication required"

**Solutions:**

#### Step 1: Sign Out and Sign In
1. Click Copilot icon in status bar
2. Select "Sign out"
3. Click Copilot icon again
4. Select "Sign in to GitHub"
5. Complete authentication in browser

#### Step 2: Check Subscription Status
**Verify you have an active Copilot subscription:**
1. Go to https://github.com/settings/copilot
2. Check subscription status
3. Ensure subscription is active

**Subscription tiers:**
- Free (limited)
- Individual ($10/mo)
- Business ($19/user/mo)
- Enterprise ($39/user/mo)

#### Step 3: Verify GitHub Connection
**Test GitHub connectivity:**
```bash
curl --verbose https://copilot-proxy.githubusercontent.com/_ping
```

**Expected:** HTTP 200 response

**If fails:** Network or firewall issue (see Issue 4)

#### Step 4: Regenerate Token
1. Sign out of Copilot (status bar icon)
2. Go to https://github.com/settings/tokens
3. Revoke any old "GitHub Copilot" tokens
4. Sign in to Copilot again (generates new token)

#### Step 5: Check Enterprise SSO
**For Enterprise users:**
1. Go to https://github.com/settings/tokens
2. Find "GitHub Copilot" token
3. Click "Configure SSO"
4. Authorize your organization

---

### Issue 3: No Suggestions Appearing

**Symptoms:**
- Copilot is active but no inline suggestions
- Suggestions appear sometimes but not always
- Specific file types not getting suggestions

**Solutions:**

#### Step 1: Check File Type Support
**Supported languages:**
- Python, JavaScript, TypeScript, Go, Ruby, Java, C/C++, C#, PHP, Rust, Kotlin, Swift, and 30+ more

**Not supported:**
- Binary files
- Very large files (>100KB may have issues)
- Encrypted files

#### Step 2: Verify Inline Suggestions Enabled
**Check settings:**
```json
{
  "editor.inlineSuggest.enabled": true,
  "github.copilot.enable": {
    "*": true
  }
}
```

**Keyboard shortcut:**
- Manually trigger: `Alt+\` (Windows/Linux) or `Option+\` (Mac)

#### Step 3: Check Language-Specific Settings
**Example: Enable for specific languages only**
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

**Fix:** Set `"*": true` to enable for all languages

#### Step 4: Clear Context and Retry
1. Save current file
2. Close and reopen file
3. Move cursor to new location
4. Start typing

#### Step 5: Check for Conflicts
**Possible conflicts:**
- Other AI code completion extensions
- Vim/Emacs extensions with custom keybindings
- Snippet extensions

**Fix:**
1. Disable other AI extensions temporarily
2. Check for keybinding conflicts (`Ctrl+K Ctrl+S`)

#### Step 6: File Size Limit
**Issue:** Files >100KB may not get suggestions

**Fix:**
- Split large files into smaller modules
- Use Chat instead for large files: `@file:large-file.ts`

---

### Issue 4: Network & Firewall Errors

**Symptoms:**
- "Could not connect to server"
- Timeout errors
- SSL/Certificate errors
- Proxy issues

**Solutions:**

#### Step 1: Test Connectivity
**Test GitHub Copilot endpoints:**
```bash
# Test main endpoint
curl --verbose https://copilot-proxy.githubusercontent.com/_ping

# Test Chat endpoint
curl --verbose https://api.githubcopilot.com/_ping
```

**Expected:** HTTP 200 response

#### Step 2: Check Proxy Configuration
**If behind corporate proxy:**

**VS Code settings:**
```json
{
  "http.proxy": "http://proxy.company.com:8080",
  "http.proxyStrictSSL": true
}
```

**Test proxy:**
```bash
curl --proxy http://proxy.company.com:8080 https://copilot-proxy.githubusercontent.com/_ping
```

#### Step 3: Certificate Issues
**If certificate errors:**

**Temporary workaround (not recommended for production):**
```json
{
  "http.proxyStrictSSL": false
}
```

**Proper fix:**
1. Install corporate root certificate
2. Configure VS Code to trust certificate
3. Contact IT department for certificate chain

#### Step 4: Firewall Rules
**Ensure firewall allows:**
- `*.github.com`
- `*.githubusercontent.com`
- `copilot-proxy.githubusercontent.com`
- `api.githubcopilot.com`

**Ports:**
- HTTPS (443)
- HTTP (80) for redirects

#### Step 5: VPN Issues
**If using VPN:**
1. Disconnect VPN temporarily
2. Test Copilot connectivity
3. If works without VPN: VPN blocking Copilot
4. Contact IT to whitelist Copilot domains

#### Step 6: Debug Logging
**Enable debug logging:**
```json
{
  "github.copilot.advanced.debug": true
}
```

**View logs:**
- Output panel → GitHub Copilot
- Look for connection errors, certificate issues

---

### Issue 5: Chat Not Responding

**Symptoms:**
- Chat view open but not responding to prompts
- Loading indicator stuck
- Chat shows error message
- Agent mode not working

**Solutions:**

#### Step 1: Check Chat Extension
**Verify GitHub Copilot Chat extension is installed:**
1. Extensions view (`Ctrl+Shift+X`)
2. Search "GitHub Copilot Chat"
3. Ensure enabled and up to date

#### Step 2: Restart Chat
**Keyboard Palette (`Ctrl+Shift+P`):**
```
> Developer: Reload Window
```

**Or close and reopen Chat view:**
- View → Command Palette → `> GitHub Copilot: Reset Chat`

#### Step 3: Clear Chat History
**In Chat view:**
- Type `/clear` and press Enter
- Or use trash icon in Chat view

#### Step 4: Check Agent Mode
**If Agent mode not working:**
1. Ensure you have Copilot Pro/Business/Enterprise subscription
2. Check agent picker (dropdown in Chat view)
3. Try switching between Ask/Edit/Plan/Agent modes

#### Step 5: Check Workspace Context
**If `@workspace` queries fail:**
1. Ensure workspace has indexable files
2. Check `.gitignore` isn't excluding everything
3. Try smaller workspace or specific files

#### Step 6: Token Limit Exceeded
**If chat stops mid-response:**
- Token limit reached
- Reduce context (fewer file references)
- Start new chat session (`/clear`)

---

### Issue 6: Slow Suggestions

**Symptoms:**
- Suggestions appear after long delay
- Chat responses very slow
- IDE feels sluggish with Copilot

**Solutions:**

#### Step 1: Check Internet Connection
**Test latency:**
```bash
ping github.com
```

**Expected:** <100ms response time

**If slow:** Internet connection issue

#### Step 2: Reduce Workspace Size
**Large workspaces slow down context:**
1. Close unnecessary folders in workspace
2. Use single-folder workspace instead of multi-root
3. Exclude large directories (node_modules, dist, etc.)

#### Step 3: Optimize Workspace
**Create `.copilotignore` (similar to `.gitignore`):**
```
node_modules/
dist/
build/
*.log
*.map
coverage/
.next/
.cache/
vendor/
```

#### Step 4: Disable Unused Extensions
**Other extensions can slow VS Code:**
1. Extensions view (`Ctrl+Shift+X`)
2. Disable extensions you don't actively use
3. Reload VS Code

#### Step 5: Choose Faster Model
**In Chat view:**
- Switch to **Claude Haiku 4.5** (fastest)
- Or **GPT-5 mini** (fast with reasoning)
- Or use **Auto** for automatic optimization

**Avoid for simple tasks:**
- Claude Opus 4.5 (slower but highest quality)
- GPT-5 (slower but deep reasoning)

#### Step 6: Close Unnecessary Files
**Open files are included in context:**
1. Close tabs you're not working on
2. Use `Ctrl+K W` to close all editors

---

### Issue 7: MCP Connection Issues

**Symptoms:**
- MCP server not starting
- Tools from MCP not available in chat
- MCP server errors in logs

**Solutions:**

#### Step 1: Check MCP Configuration
**Open MCP config:**
- Command Palette: `> MCP: Open Workspace Folder Configuration`
- File: `.vscode/mcp.json`

**Verify:**
- Valid JSON syntax
- Correct `command` and `args`
- Required environment variables set

#### Step 2: Test Server Manually
**Run MCP server command manually:**
```bash
# Example for Node.js MCP server
node /path/to/mcp-server.js
```

**Should start without errors**

#### Step 3: Check Server Logs
**View MCP server logs:**
1. Command Palette: `> MCP: List Servers`
2. Select your server
3. Choose "View Logs"

**Look for:**
- Startup errors
- Missing dependencies
- Authentication issues

#### Step 4: Install Dependencies
**For npm-based servers:**
```bash
npm install -g package-name
```

**For Python servers:**
```bash
pip install package-name
```

#### Step 5: Input Variables
**If using `${input:variable-name}`:**
1. Ensure input defined in `"inputs": []` section
2. Restart server after adding inputs
3. VS Code will prompt for input values

**Example:**
```json
{
  "servers": {
    "my-server": {
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

#### Step 6: Restart MCP Server
**Command Palette:**
```
> MCP: List Servers
> Select server → Restart
```

---

### Issue 8: Context Not Loading

**Symptoms:**
- `@workspace` returns no results
- File references not working
- Chat doesn't understand codebase

**Solutions:**

#### Step 1: Check Workspace Indexing
**VS Code indexes your workspace for search:**
- Wait for indexing to complete (check status bar)
- Large workspaces take longer

**Force reindex:**
- Command Palette: `> Developer: Reload Window`

#### Step 2: Check .gitignore
**Context uses files not ignored by .gitignore:**
- Verify important files aren't in .gitignore
- Check `.gitignore` patterns

**Note:** Open files bypass .gitignore

#### Step 3: Use Specific References
**Instead of `@workspace`, try:**
- `#file:path/to/file.ts` - Specific file
- `#folder:src/` - Specific folder
- Drag and drop files into chat

#### Step 4: Check File Size
**Very large files may not load:**
- Files >100KB may be truncated
- Split into smaller modules
- Or ask about specific sections

#### Step 5: Workspace Trust
**Untrusted workspaces have limited features:**
1. Command Palette: `> Workspace: Manage Workspace Trust`
2. Choose "Trust" if safe
3. Reload window

---

## ❓ Frequently Asked Questions

### Product & Subscription

#### Q: What is GitHub Copilot?
**A:** GitHub Copilot is an AI-powered coding assistant developed by GitHub/Microsoft that provides:
- Inline code completions as you type
- Chat-based coding assistance
- Autonomous agents for complex tasks
- Integration with VS Code and other IDEs

#### Q: Do I need GitHub Pro to use Copilot?
**A:** No. GitHub Copilot has separate subscriptions:
- **Free**: $0/mo (limited completions, basic chat)
- **Individual**: $10/mo (unlimited completions, full chat)
- **Business**: $19/user/mo (+ analytics, organization management)
- **Enterprise**: $39/user/mo (+ SSO, audit logs, policy controls)

#### Q: Can I try it for free?
**A:** Yes!
- Free plan available (with limitations)
- 30-day trial for paid plans
- Educational discounts for students and teachers

#### Q: What's included in my subscription?
**A:**
- **All plans**: Inline completions (GPT-4.1 model), basic chat
- **Individual+**: Unlimited completions, full chat, basic model selection
- **Pro/Pro+**: All models including Claude Opus 4.5, GPT-5, Gemini 2.5 Pro
- **Business**: + Usage analytics, organization management
- **Enterprise**: + SSO, audit logs, policy controls

---

### Getting Started

#### Q: What's the minimum VS Code version?
**A:** VS Code 1.98.0 or higher required.

**Check version:**
```bash
code --version
```

**Update:**
- Help → Check for Updates

#### Q: Which extensions do I need?
**A:** Install both:
1. **GitHub Copilot** - Inline completions
2. **GitHub Copilot Chat** - Chat interface

**Install:**
- Extensions view (`Ctrl+Shift+X`)
- Search "GitHub Copilot"
- Install both extensions

#### Q: How do I authenticate?
**A:**
1. Click Copilot icon in status bar
2. Select "Sign in to GitHub"
3. Complete authentication in browser
4. Return to VS Code

---

### Technical Questions

#### Q: How do I switch AI models?
**A:** 
- **Pro/Pro+ users**: Click model picker in Chat view → Choose model
- **Recommended**: Use **Auto** for automatic optimization
- **Available models**: Claude Opus/Sonnet/Haiku 4.5, GPT-5/5 mini/4.1, Gemini 2.5 Pro
- **Business/Enterprise**: Model selection coming soon (uses Claude Sonnet 4.5 default)

#### Q: What's the context window size?
**A:** Varies by model:
- **GPT-4.1**: ~128K tokens (~500KB code)
- **GPT-5/5 mini**: ~128-200K tokens
- **Claude Sonnet/Opus/Haiku 4.5**: ~200K tokens
- **Gemini 2.5 Pro**: ~1M+ tokens (longest)

#### Q: Can I use Copilot offline?
**A:** No. Copilot requires internet connection:
- Suggestions generated in cloud
- Authentication requires GitHub connection
- No offline mode available

#### Q: What languages are supported?
**A:** 30+ languages including:
- **Popular**: Python, JavaScript, TypeScript, Java, Go, C/C++, C#, Ruby, PHP, Rust
- **Others**: Kotlin, Swift, Shell, SQL, HTML, CSS, YAML, JSON, and more
- **AI quality varies**: Best for popular languages with more training data

#### Q: Does Copilot work in private repos?
**A:** Yes, with proper subscription:
- **Individual**: Works in your private repos
- **Business**: Works in organization repos
- **Enterprise**: Full private repo support

---

### Custom Instructions & Configuration

#### Q: How do custom instructions work?
**A:** Create `.github/copilot-instructions.md` in repo root:
- Copilot automatically reads and follows instructions
- Apply to all suggestions in workspace
- Can be language-specific with `applyTo` frontmatter

#### Q: Where should I put custom agents?
**A:** Store agents in `.vscode/agents/`:
- Workspace agents: `.vscode/agents/` (share with team)
- User agents: `~/.vscode/agents/` (personal)
- Format: `agent-name.agent.md`

#### Q: Can I share configurations with my team?
**A:** Yes:
- **Custom instructions**: `.github/copilot-instructions.md` (commit to repo)
- **Custom agents**: `.vscode/agents/*.agent.md` (commit to repo)
- **Prompt files**: `.vscode/prompts/*.prompt.md` (commit to repo)
- **MCP config**: `.vscode/mcp.json` (commit to repo)
- **Settings**: `.vscode/settings.json` (commit to repo)

**Exclude from version control:**
- `mcp.json` if it contains secrets (use `${input:}` variables instead)

---

### Security & Privacy

#### Q: Does GitHub train on my code?
**A:**
- **Free/Individual**: Code snippets may inform improvements (telemetry)
- **Business/Enterprise**: No training on customer code by default
- **All plans**: Can opt out of telemetry

**Opt out:**
```json
{
  "github.copilot.advanced.telemetry": "disabled"
}
```

#### Q: Is Copilot secure for enterprise use?
**A:** Yes:
- ✅ SOC 2 Type II certified
- ✅ GDPR compliant
- ✅ SAML SSO support (Enterprise)
- ✅ Audit logs (Enterprise)
- ✅ Data encryption in transit and at rest
- ✅ No code storage (processed and discarded)

#### Q: Can I block Copilot from certain files?
**A:** Yes, multiple ways:
1. **Workspace Trust**: Disable Copilot in untrusted workspaces
2. **Settings**: Disable for specific file patterns
3. **Organization policy**: Block at organization level (Enterprise)

**Example - disable for secrets:**
```json
{
  "github.copilot.enable": {
    "*": true,
    ".env": false,
    "*.key": false,
    "secrets.json": false
  }
}
```

#### Q: What data does Copilot send to GitHub?
**A:**
- Code snippets for context (current file, selection)
- File name and language
- Cursor position
- User prompts in chat
- Telemetry (if enabled)

**NOT sent:**
- Entire codebase (only indexed locally)
- Sensitive files (if properly configured)
- Personal data (unless in code you're editing)

---

### Workflows & Usage

#### Q: Should I always accept Copilot suggestions?
**A:** **NO!** Always review:
- Copilot is a tool, not a replacement for judgment
- Suggestions may have bugs, security issues, or be incorrect
- Test and validate all generated code
- Use as starting point, refine as needed

#### Q: Can Copilot help with code review?
**A:** Yes! Multiple approaches:
- Chat: `@workspace Review PR #123 for security issues`
- Custom agent: Create security reviewer agent
- Inline: `Ctrl+I` → "Review this function for bugs"

#### Q: How do I use Copilot for testing?
**A:**
1. **Inline**: Select function → `Ctrl+I` → "Generate tests"
2. **Chat**: `@workspace /tests` for entire module
3. **Slash command**: `/tests` in Chat view
4. **Setup**: `/setupTests` to configure test framework

#### Q: Can Copilot refactor my code?
**A:** Yes, excellent at refactoring:
- **Small**: `Ctrl+I` → "Refactor to use async/await"
- **Large**: Chat → `@workspace Refactor auth module to use new pattern`
- **Plan**: Switch to Plan mode → "Plan refactoring of legacy code"

---

### Performance & Optimization

#### Q: Why are suggestions slow?
**A:** Common causes:
1. **Large workspace**: Exclude unnecessary folders
2. **Slow internet**: Check connection speed
3. **Heavy model**: Switch to Claude Haiku 4.5 or use Auto
4. **Many open files**: Close unused tabs

**Optimization:**
- Create `.copilotignore` file
- Use faster model (Claude Haiku 4.5)
- Reduce workspace size
- Close unnecessary files

#### Q: How do I reduce costs?
**A:** For Pro/Pro+ with premium models:
1. **Use Auto** - Automatic model optimization ⭐
2. **Choose wisely**: Claude Haiku 4.5 for simple tasks
3. **Reserve premium**: Claude Opus 4.5/GPT-5 only for complex work
4. **Optimize prompts**: Reduce retries with clear prompts
5. **Monitor usage**: Check usage analytics (Business+)

#### Q: Can I use Copilot with large codebases?
**A:** Yes, but optimize:
- Exclude `node_modules`, `dist`, `build`
- Use `.copilotignore` file
- Reference specific files/folders instead of `@workspace`
- Consider multi-root workspace for modular projects
- Use `#folder:specific-module` instead of entire codebase

---

### Advanced Features

#### Q: What are MCP servers?
**A:** Model Context Protocol servers extend Copilot with custom tools:
- Connect to external APIs
- Access databases
- Integrate with tools (Playwright, GitHub API, etc.)
- Add custom functionality to Chat

**Setup:** Create `.vscode/mcp.json` with server configuration

#### Q: What are subagents?
**A:** Subagents are isolated agent instances:
- Delegate complex subtasks
- Prevent context pollution
- Run specialized analyses
- Chain multiple agents together

**Use in Agent mode:** Copilot automatically uses subagents for complex tasks

#### Q: Can I create custom slash commands?
**A:** Yes, using prompt files:
1. Create `.vscode/prompts/my-command.prompt.md`
2. Define prompt in file
3. Use as `/my-command` in Chat

#### Q: What's the difference between agents and prompt files?
**A:**
- **Agents** (`.agent.md`): Specialized AI personas with specific tools and workflows
- **Prompt files** (`.prompt.md`): Reusable prompt templates with variables

**Use agents for:** Security review, performance optimization, documentation
**Use prompts for:** Recurring questions, standardized formats, templates

---

### Troubleshooting

#### Q: Why aren't I getting suggestions?
**A:** Check:
1. ✅ Extension installed and activated
2. ✅ Authenticated to GitHub
3. ✅ Active subscription
4. ✅ File type supported
5. ✅ `editor.inlineSuggest.enabled: true`
6. ✅ Not in untrusted workspace

**Quick fix:** `Alt+\` to manually trigger

#### Q: "Extension activation failed" error?
**A:** Common fixes:
1. Sign out and sign in again
2. Check subscription at https://github.com/settings/copilot
3. Update VS Code to 1.98.0+
4. Reinstall extensions
5. Check network connectivity

#### Q: How do I view logs?
**A:**
1. View → Output (`Ctrl+Shift+U`)
2. Select "GitHub Copilot" or "GitHub Copilot Chat"
3. Look for errors

**Enable debug logging:**
```json
{
  "github.copilot.advanced.debug": true
}
```

#### Q: MCP server not working?
**A:** Troubleshoot:
1. Check `mcp.json` syntax
2. Test server command manually
3. View server logs (`> MCP: List Servers` → View Logs)
4. Restart server
5. Check input variables configured

---

## 🆘 Getting Help

### Self-Service Resources

1. **Official Documentation**
   - GitHub Copilot: https://docs.github.com/copilot
   - VS Code Copilot: https://code.visualstudio.com/docs/copilot

2. **Output Logs**
   - View → Output → GitHub Copilot
   - Enable debug: `"github.copilot.advanced.debug": true`

3. **Community Forums**
   - GitHub Community: https://github.community
   - VS Code Community: https://code.visualstudio.com/community

### Contact Support

1. **GitHub Support Portal**
   - https://support.github.com
   - Include: Error messages, log files, steps to reproduce

2. **Report Issues**
   - GitHub Copilot: https://github.com/github/copilot-feedback
   - VS Code: https://github.com/microsoft/vscode

3. **Feature Requests**
   - GitHub Copilot Feedback: https://github.com/github/copilot-feedback/discussions

### Before Contacting Support

Gather this information:
- ✅ VS Code version (`code --version`)
- ✅ Extension versions (Extensions view)
- ✅ Error message (exact text or screenshot)
- ✅ Steps to reproduce
- ✅ Output logs (View → Output → GitHub Copilot)
- ✅ Network test results (if connectivity issue)

---

## 🔍 Diagnostic Checklist

Use this checklist to diagnose issues:

### Quick Diagnostic

```
□ VS Code version 1.98.0+?
□ Extensions installed (Copilot + Copilot Chat)?
□ Authenticated to GitHub?
□ Active subscription?
□ Internet connection working?
□ Copilot icon in status bar?
□ Status bar shows "Active"?
```

### Detailed Diagnostic

```
□ Can access https://copilot-proxy.githubusercontent.com/_ping?
□ Can access https://api.githubcopilot.com/_ping?
□ Inline suggestions enabled in settings?
□ File type supported?
□ No conflicting extensions?
□ Workspace trusted?
□ Not behind firewall blocking GitHub?
□ Proxy configured correctly (if applicable)?
□ Logs show no errors?
□ Extension recently updated?
```

### Network Diagnostic

```bash
# Test GitHub connectivity
curl --verbose https://copilot-proxy.githubusercontent.com/_ping

# Test Chat endpoint
curl --verbose https://api.githubcopilot.com/_ping

# Test with proxy (if applicable)
curl --proxy http://proxy:8080 https://copilot-proxy.githubusercontent.com/_ping

# Check certificate (Windows/Linux)
curl --verbose --insecure https://copilot-proxy.githubusercontent.com/_ping
```

**Expected:** HTTP 200 response for all

---

## 💡 Pro Troubleshooting Tips

### Tip 1: Restart Workflow
**Most issues fixed by:**
1. Save all files
2. Close all editor tabs
3. Developer: Reload Window
4. Reopen your files

### Tip 2: Clean Slate
**For persistent issues:**
1. Sign out of Copilot
2. Close VS Code
3. Clear VS Code cache:
   - Windows: `%APPDATA%\Code\`
   - Mac: `~/Library/Application Support/Code/`
   - Linux: `~/.config/Code/`
4. Delete `Cache`, `CachedData` folders
5. Restart VS Code
6. Sign in to Copilot

### Tip 3: Isolate the Issue
**Test in minimal environment:**
1. Create new folder
2. Open in VS Code
3. Create simple test file (e.g., `test.js`)
4. Try Copilot suggestions

**If works:** Issue with your workspace
**If doesn't work:** Issue with setup/network

### Tip 4: Check Conflicts
**Disable other extensions temporarily:**
1. Extensions view
2. Disable all except Copilot
3. Test Copilot
4. Re-enable extensions one by one

### Tip 5: Network Bypass Test
**Test without VPN/proxy:**
1. Disconnect from VPN
2. Use direct internet connection
3. Test Copilot

**If works:** Network/firewall issue

---

## 📖 Next Steps

After troubleshooting:

1. **Bookmark this page** for future reference
2. **Join GitHub Community** for peer support
3. **Enable debug logging** for better diagnostics
4. **Contact support** if issues persist

**Previous:** [Section 1: Complete Reference →](01-complete-reference.md)

---

**Part 7: Reference, Troubleshooting & Advanced**  
Section 2 of 2 | [← Previous: Reference Guide](01-complete-reference.md)

