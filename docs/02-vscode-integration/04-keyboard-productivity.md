# Section 4: Keyboard Shortcuts & Productivity Hacks

**Part 2 > Section 4 of 4**  
**Time to Complete:** 45 minutes  
**Prerequisites:** Sections 1-3 completed

---

## 🎯 Learning Objectives

By the end of this section, you will:
- ✅ Master all essential Copilot keyboard shortcuts
- ✅ Create custom keybindings for frequent workflows
- ✅ Use multi-cursor editing with Copilot suggestions
- ✅ Integrate snippets with Copilot completions
- ✅ Configure Vim mode patterns (if applicable)
- ✅ Set up accessibility features for screen readers and keyboard-only navigation

---

## 📋 Table of Contents

1. [Essential Copilot Shortcuts](#essential-copilot-shortcuts)
2. [Advanced Navigation](#advanced-navigation)
3. [Custom Keybindings](#custom-keybindings)
4. [Multi-Cursor Workflows](#multi-cursor-workflows)
5. [Snippet Integration](#snippet-integration)
6. [Vim Mode Patterns](#vim-mode-patterns)
7. [Accessibility Features](#accessibility-features)
8. [Productivity Workflows](#productivity-workflows)

---

## Essential Copilot Shortcuts

### The Must-Know 10

**Memorize these first - they cover 80% of daily use:**

| Shortcut | Windows/Linux | macOS | Action |
|----------|---------------|-------|--------|
| **1** | `Tab` | `Tab` | Accept entire suggestion |
| **2** | `Esc` | `Esc` | Reject suggestion |
| **3** | `Ctrl+I` | `Cmd+I` | Open inline chat |
| **4** | `Ctrl+Alt+I` | `Cmd+Option+I` | Open Chat view |
| **5** | `Ctrl+Shift+Alt+L` | `Cmd+Shift+Option+L` | Open Quick Chat |
| **6** | `Alt+]` | `Option+]` | Next suggestion |
| **7** | `Alt+[` | `Option+[` | Previous suggestion |
| **8** | `Ctrl+Enter` | `Cmd+Enter` | Show all suggestions |
| **9** | `Ctrl+→` | `Cmd+→` | Accept next word |
| **10** | Right-click selection → "Copilot: Explain" | Explain code |

### Complete Copilot Shortcuts Reference

#### Inline Suggestions

| Action | Windows/Linux | macOS | Description |
|--------|---------------|-------|-------------|
| **Accept full suggestion** | `Tab` | `Tab` | Accept entire line/block |
| **Accept word** | `Ctrl+→` | `Cmd+→` | Accept next word only |
| **Reject** | `Esc` | `Esc` | Dismiss current suggestion |
| **Next suggestion** | `Alt+]` | `Option+]` | Cycle forward through alternatives |
| **Previous suggestion** | `Alt+[` | `Option+[` | Cycle backward |
| **Show all** | `Ctrl+Enter` | `Cmd+Enter` | Open suggestions panel (10+ options) |
| **Trigger manually** | `Alt+\` | `Option+\` | Force suggestion (if not appearing) |

#### Chat Interfaces

| Action | Windows/Linux | macOS | Description |
|--------|---------------|-------|-------------|
| **Quick Chat** | `Ctrl+Shift+Alt+L` | `Cmd+Shift+Option+L` | Overlay for fast questions |
| **Inline Chat** | `Ctrl+I` | `Cmd+I` | In-editor chat for refactoring |
| **Chat View** | `Ctrl+Alt+I` | `Cmd+Option+I` | Full sidebar chat panel |
| **Terminal Chat** | `Ctrl+I` (in terminal) | `Cmd+I` (in terminal) | Command suggestions |
| **Accept inline change** | `Ctrl+Enter` | `Cmd+Enter` | Apply inline chat edit |
| **Reject inline change** | `Esc` | `Esc` | Discard inline chat edit |
| **Clear chat** | `/clear` in chat | `/clear` in chat | Reset conversation |

#### Context Actions

| Action | Windows/Linux | macOS | Description |
|--------|---------------|-------|-------------|
| **Explain code** | Select → Right-click → "Copilot: Explain" | Explain selection |
| **Fix error** | On error → `Ctrl+.` → "Copilot: Fix" | Quick fix |
| **Generate tests** | `/tests` in chat | `/tests` in chat | Create test cases |
| **Generate docs** | `/doc` in chat | `/doc` in chat | Add documentation |
| **Refactor** | Select → `Ctrl+I` → "refactor this" | Refactor code |

### Shortcut Cheat Sheet (Printable)

```
╔══════════════════════════════════════════════════════════════╗
║           GitHub Copilot Shortcuts Cheat Sheet              ║
╠══════════════════════════════════════════════════════════════╣
║ INLINE SUGGESTIONS                                          ║
║  Tab             Accept suggestion                          ║
║  Esc             Reject suggestion                          ║
║  Alt+]           Next suggestion                            ║
║  Alt+[           Previous suggestion                        ║
║  Ctrl+Enter      Show all suggestions                       ║
║  Ctrl+→          Accept next word                           ║
╠══════════════════════════════════════════════════════════════╣
║ CHAT INTERFACES                                             ║
║  Ctrl+I          Inline chat (refactor in editor)           ║
║  Ctrl+Alt+I      Chat view (full sidebar)                   ║
║  Ctrl+Shift+Alt+L  Quick chat (overlay)                     ║
╠══════════════════════════════════════════════════════════════╣
║ QUICK ACTIONS                                               ║
║  /fix            Fix problems                               ║
║  /tests          Generate tests                             ║
║  /doc            Generate docs                              ║
║  /explain        Explain code                               ║
║  @workspace      Include workspace context                  ║
║  @file:path      Reference specific file                    ║
╚══════════════════════════════════════════════════════════════╝

Tip: Replace Ctrl with Cmd on macOS
```

---

## Advanced Navigation

### Editor Navigation with Copilot

**Moving between suggestions and code:**

| Action | Windows/Linux | macOS | Use Case |
|--------|---------------|-------|----------|
| **Go to definition** | `F12` | `F12` | Jump to function/class |
| **Peek definition** | `Alt+F12` | `Option+F12` | Inline definition view |
| **Go back** | `Alt+←` | `Cmd+←` | Navigate back |
| **Go forward** | `Alt+→` | `Cmd+→` | Navigate forward |
| **Go to symbol** | `Ctrl+Shift+O` | `Cmd+Shift+O` | File symbols |
| **Go to workspace symbol** | `Ctrl+T` | `Cmd+T` | Project-wide search |

**Quick file switching:**

| Action | Windows/Linux | macOS | Use Case |
|--------|---------------|-------|----------|
| **Quick Open** | `Ctrl+P` | `Cmd+P` | Open file by name |
| **Recent files** | `Ctrl+R` | `Cmd+R` | Recently opened |
| **Switch editor** | `Ctrl+Tab` | `Cmd+Tab` | Cycle open files |
| **Close editor** | `Ctrl+W` | `Cmd+W` | Close current file |
| **Reopen closed** | `Ctrl+Shift+T` | `Cmd+Shift+T` | Reopen last closed |

### Workflow: Keyboard-Only Development

**Complete feature implementation without mouse:**

```
1. Open file:                Ctrl+P → type filename → Enter
2. Navigate to function:     Ctrl+Shift+O → type function name → Enter
3. Start typing:             Copilot suggests
4. Accept suggestion:        Tab
5. Refactor:                 Ctrl+I → type instruction → Ctrl+Enter
6. Generate test:            Ctrl+Alt+I → "/tests" → Enter
7. Switch to test file:      Ctrl+P → type test filename → Enter
8. Run tests:                Ctrl+Shift+P → "Test: Run All Tests" → Enter
9. Go back to code:          Alt+← (or Ctrl+Tab)
10. Commit:                  Ctrl+Shift+G → type message → Ctrl+Enter
```

**Target time:** <2 minutes for simple feature with test

---

## Custom Keybindings

### Creating Custom Shortcuts

**Access keybindings:**

```
Method 1: UI
File → Preferences → Keyboard Shortcuts (Ctrl+K Ctrl+S)

Method 2: JSON
File → Preferences → Keyboard Shortcuts → Click {} icon (top right)
```

### Essential Custom Keybindings for Copilot

**Add to `keybindings.json`:**

```json
[
  // ========================================
  // Copilot Quick Actions
  // ========================================
  
  // Quick access to /tests command
  {
    "key": "ctrl+shift+t",
    "command": "workbench.action.chat.open",
    "args": { "query": "/tests" },
    "when": "editorTextFocus"
  },
  
  // Quick access to /doc command
  {
    "key": "ctrl+shift+d",
    "command": "workbench.action.chat.open",
    "args": { "query": "/doc" },
    "when": "editorTextFocus"
  },
  
  // Quick access to /explain
  {
    "key": "ctrl+shift+e",
    "command": "workbench.action.chat.open",
    "args": { "query": "/explain" }
  },
  
  // Quick access to /fix
  {
    "key": "ctrl+shift+f",
    "command": "workbench.action.chat.open",
    "args": { "query": "/fix" }
  },
  
  // ========================================
  // Copilot Toggle
  // ========================================
  
  // Toggle Copilot on/off globally
  {
    "key": "ctrl+shift+alt+c",
    "command": "github.copilot.toggleCopilot"
  },
  
  // ========================================
  // Chat Enhancements
  // ========================================
  
  // Open Chat with @workspace
  {
    "key": "ctrl+shift+alt+w",
    "command": "workbench.action.chat.open",
    "args": { "query": "@workspace " }
  },
  
  // Open Chat with @file for current file
  {
    "key": "ctrl+shift+alt+f",
    "command": "workbench.action.chat.open",
    "args": { "query": "@file " }
  },
  
  // ========================================
  // Inline Chat Enhancements
  // ========================================
  
  // Inline chat with "add tests"
  {
    "key": "ctrl+alt+t",
    "command": "inlineChat.start",
    "args": { "query": "add unit tests" },
    "when": "editorTextFocus"
  },
  
  // Inline chat with "add error handling"
  {
    "key": "ctrl+alt+e",
    "command": "inlineChat.start",
    "args": { "query": "add error handling" },
    "when": "editorTextFocus"
  },
  
  // Inline chat with "optimize performance"
  {
    "key": "ctrl+alt+o",
    "command": "inlineChat.start",
    "args": { "query": "optimize performance" },
    "when": "editorTextFocus"
  },
  
  // ========================================
  // Refactoring Shortcuts
  // ========================================
  
  // Extract function (with Copilot context)
  {
    "key": "ctrl+shift+r ctrl+e",
    "command": "editor.action.codeAction",
    "args": {
      "kind": "refactor.extract.function"
    },
    "when": "editorTextFocus"
  },
  
  // Rename symbol
  {
    "key": "ctrl+shift+r ctrl+r",
    "command": "editor.action.rename",
    "when": "editorTextFocus"
  },
  
  // ========================================
  // Productivity Boosters
  // ========================================
  
  // Open Copilot settings
  {
    "key": "ctrl+shift+alt+s",
    "command": "workbench.action.openSettings",
    "args": { "query": "copilot" }
  },
  
  // Show Copilot logs (troubleshooting)
  {
    "key": "ctrl+shift+alt+l",
    "command": "github.copilot.showLogs"
  },
  
  // ========================================
  // Advanced: Chord Shortcuts (Two-Key Sequences)
  // ========================================
  
  // Ctrl+K, Ctrl+C: Open Chat
  {
    "key": "ctrl+k ctrl+c",
    "command": "workbench.action.chat.open"
  },
  
  // Ctrl+K, Ctrl+T: Generate tests
  {
    "key": "ctrl+k ctrl+t",
    "command": "workbench.action.chat.open",
    "args": { "query": "/tests" }
  },
  
  // Ctrl+K, Ctrl+D: Generate docs
  {
    "key": "ctrl+k ctrl+d",
    "command": "workbench.action.chat.open",
    "args": { "query": "/doc" }
  }
]
```

### Context-Aware Keybindings

**Use `when` clause to make shortcuts context-specific:**

```json
[
  // Only in TypeScript files
  {
    "key": "ctrl+shift+i",
    "command": "inlineChat.start",
    "args": { "query": "add TypeScript type annotations" },
    "when": "editorTextFocus && editorLangId == 'typescript'"
  },
  
  // Only in Python files
  {
    "key": "ctrl+shift+i",
    "command": "inlineChat.start",
    "args": { "query": "add type hints and docstrings" },
    "when": "editorTextFocus && editorLangId == 'python'"
  },
  
  // Only when debugging
  {
    "key": "ctrl+shift+e",
    "command": "workbench.action.chat.open",
    "args": { "query": "explain this error" },
    "when": "inDebugMode"
  },
  
  // Only in test files
  {
    "key": "ctrl+shift+r",
    "command": "inlineChat.start",
    "args": { "query": "refactor this test for better readability" },
    "when": "editorTextFocus && resourceFilename =~ /\\.test\\.//"
  }
]
```

### When Clause Reference

Common `when` conditions:

| Condition | Meaning |
|-----------|---------|
| `editorTextFocus` | Editor has focus |
| `editorHasSelection` | Text is selected |
| `editorLangId == 'typescript'` | TypeScript file |
| `inDebugMode` | Debugging active |
| `terminalFocus` | Terminal has focus |
| `resourceFilename =~ /\\.test\\./` | Filename contains ".test." |

---

## Multi-Cursor Workflows

### Multi-Cursor Basics

| Action | Windows/Linux | macOS | Description |
|--------|---------------|-------|-------------|
| **Add cursor above** | `Ctrl+Alt+↑` | `Cmd+Option+↑` | New cursor on line above |
| **Add cursor below** | `Ctrl+Alt+↓` | `Cmd+Option+↓` | New cursor on line below |
| **Add cursor to selection** | `Alt+Click` | `Option+Click` | Click to add cursor |
| **Select all occurrences** | `Ctrl+Shift+L` | `Cmd+Shift+L` | Multi-cursor on all matches |
| **Select next occurrence** | `Ctrl+D` | `Cmd+D` | Add next match to selection |
| **Undo last cursor** | `Ctrl+U` | `Cmd+U` | Remove last cursor |
| **Escape all cursors** | `Esc` | `Esc` | Return to single cursor |

### Multi-Cursor + Copilot Pattern

**Scenario:** Add JSDoc to multiple functions

```typescript
// Before: 3 functions without docs

function calculateTotal(items: Item[]): number { ... }

function formatCurrency(amount: number): string { ... }

function validateItem(item: Item): boolean { ... }

// Workflow:
1. Place cursor above first function
2. Ctrl+Alt+↓ twice (3 cursors total)
3. Type: /**
4. Copilot suggests JSDoc for each function simultaneously
5. Tab to accept all three
```

**Result:**

```typescript
/**
 * Calculates the total price of all items
 * @param items Array of items to sum
 * @returns Total price
 */
function calculateTotal(items: Item[]): number { ... }

/**
 * Formats a number as currency
 * @param amount Amount to format
 * @returns Formatted currency string
 */
function formatCurrency(amount: number): string { ... }

/**
 * Validates that an item has all required fields
 * @param item Item to validate
 * @returns True if valid, false otherwise
 */
function validateItem(item: Item): boolean { ... }
```

### Advanced Multi-Cursor Patterns

**Pattern 1: Consistent refactoring across multiple lines**

```typescript
// Before: Inconsistent error handling
fetch(url1).then(res => res.json());
fetch(url2).then(res => res.json());
fetch(url3).then(res => res.json());

// Steps:
1. Select "fetch" in first line
2. Ctrl+D (macOS: Cmd+D) twice to select all 3
3. Ctrl+I (inline chat)
4. Type: "add error handling with try/catch"
5. Copilot refactors all 3 consistently
```

**Pattern 2: Add types to multiple parameters**

```typescript
// Before: Untyped parameters
function createUser(name, email, age) { ... }
function updateUser(id, name, email, age) { ... }

// Steps:
1. Multi-cursor on parameter lists
2. Start typing: ": "
3. Copilot suggests appropriate types for each
```

---

## Snippet Integration

### Using Snippets with Copilot

**Snippets + Copilot = Superpower**

1. **Trigger snippet** (prefix + Tab)
2. **Copilot completes** snippet placeholders intelligently
3. **Tab through** placeholders with smart completions

### Creating Copilot-Friendly Snippets

**Configure snippets:**

```
File → Preferences → Configure User Snippets → Select language
```

**TypeScript example (`typescript.json`):**

```json
{
  "React Functional Component with Copilot": {
    "prefix": "rfc",
    "body": [
      "import React from 'react';",
      "",
      "interface ${1:ComponentName}Props {",
      "  $2",
      "}",
      "",
      "export const ${1:ComponentName}: React.FC<${1:ComponentName}Props> = ({ $3 }) => {",
      "  $0",
      "};",
      ""
    ],
    "description": "React functional component (Copilot will suggest props)"
  },
  
  "API Route Handler": {
    "prefix": "apihandler",
    "body": [
      "export async function ${1:handlerName}(req: Request, res: Response): Promise<void> {",
      "  try {",
      "    $0",
      "  } catch (error) {",
      "    // Copilot will suggest error handling",
      "  }",
      "}"
    ],
    "description": "API handler with error handling"
  },
  
  "Test Suite": {
    "prefix": "describe",
    "body": [
      "describe('${1:TestSuite}', () => {",
      "  beforeEach(() => {",
      "    $2",
      "  });",
      "",
      "  it('${3:should do something}', () => {",
      "    $0",
      "  });",
      "});"
    ],
    "description": "Test suite (Copilot suggests test cases)"
  }
}
```

**Workflow:**

1. Type `rfc` → Tab (snippet expands)
2. Type component name → Tab (Copilot suggests props)
3. Tab through props → Copilot suggests implementation
4. Tab to body → Copilot suggests component logic

---

## Vim Mode Patterns

### Setup: Vim Extension + Copilot

**Install Vim extension:**

```
Extensions → Search "Vim" → Install "Vim" by vscodevim
```

**Configure Vim + Copilot:**

**settings.json:**

```json
{
  // Enable Vim mode
  "vim.enableNeovim": true,
  
  // Copilot works in Insert mode
  "vim.handleKeys": {
    "<C-i>": false,  // Allow Ctrl+I for inline chat
    "<C-enter>": false  // Allow Ctrl+Enter for accept
  },
  
  // Use Copilot suggestions in Vim insert mode
  "editor.inlineSuggest.enabled": true,
  
  // Map Vim commands to Copilot
  "vim.normalModeKeyBindingsNonRecursive": [
    {
      "before": ["<leader>", "c", "e"],
      "commands": ["workbench.action.chat.open"],
      "args": { "query": "/explain" }
    },
    {
      "before": ["<leader>", "c", "t"],
      "commands": ["workbench.action.chat.open"],
      "args": { "query": "/tests" }
    },
    {
      "before": ["<leader>", "c", "d"],
      "commands": ["workbench.action.chat.open"],
      "args": { "query": "/doc" }
    }
  ],
  
  // Leader key
  "vim.leader": " "  // Spacebar as leader
}
```

### Vim + Copilot Workflow

**Insert mode (suggestions active):**

```
i              → Enter insert mode
               → Start typing
               → Copilot suggests
Tab            → Accept suggestion
Esc            → Back to normal mode
```

**Normal mode (Copilot commands):**

```
Space + c + e  → Explain selection (/explain)
Space + c + t  → Generate tests (/tests)
Space + c + d  → Generate docs (/doc)
```

**Visual mode (refactoring):**

```
V              → Visual line mode
jjj            → Select 3 lines
Ctrl+I         → Inline chat
               → Type: "refactor to use map"
Ctrl+Enter     → Accept refactoring
```

### Vim Keybinding Examples

**Advanced Vim + Copilot mappings:**

```json
{
  "vim.normalModeKeyBindingsNonRecursive": [
    // Quick Chat with visual selection
    {
      "before": ["<leader>", "c", "c"],
      "commands": ["workbench.action.quickchat.toggle"]
    },
    
    // Inline refactor
    {
      "before": ["<leader>", "r"],
      "commands": ["inlineChat.start"],
      "args": { "query": "refactor this" }
    },
    
    // Add error handling
    {
      "before": ["<leader>", "e", "h"],
      "commands": ["inlineChat.start"],
      "args": { "query": "add error handling" }
    },
    
    // Optimize performance
    {
      "before": ["<leader>", "o", "p"],
      "commands": ["inlineChat.start"],
      "args": { "query": "optimize performance" }
    }
  ],
  
  "vim.visualModeKeyBindingsNonRecursive": [
    // Explain selected code
    {
      "before": ["<leader>", "e"],
      "commands": ["copilot.explainThis"]
    },
    
    // Generate tests for selection
    {
      "before": ["<leader>", "t"],
      "commands": ["workbench.action.chat.open"],
      "args": { "query": "@selection /tests" }
    }
  ]
}
```

---

## Accessibility Features

### Screen Reader Support

**Enable screen reader optimizations:**

```json
{
  // Optimize for screen readers
  "editor.accessibilitySupport": "on",
  
  // Announce inline suggestions
  "editor.inlineSuggest.accessibilityVerbose": true,
  
  // Copilot suggestions as accessible text
  "github.copilot.editor.enableAutoCompletions": true
}
```

**How it works:**

1. Copilot suggestion appears
2. Screen reader announces: "Suggestion available: [code snippet]"
3. Tab to accept, Esc to reject
4. Screen reader announces: "Suggestion accepted" or "Suggestion rejected"

### Keyboard-Only Navigation

**Navigating VS Code without mouse:**

| Action | Shortcut | Description |
|--------|----------|-------------|
| **Command Palette** | `Ctrl+Shift+P` | Access all commands |
| **Focus Explorer** | `Ctrl+Shift+E` | File tree |
| **Focus Editor** | `Ctrl+1` | Return to editor |
| **Focus Terminal** | `Ctrl+\`` | Toggle terminal |
| **Focus Sidebar** | `Ctrl+0` | First sidebar item |
| **Next panel** | `Ctrl+K Ctrl+↓` | Cycle panels |
| **Previous panel** | `Ctrl+K Ctrl+↑` | Reverse cycle |

### High Contrast Themes

**Enable high contrast for better visibility:**

```
File → Preferences → Color Theme → Search "High Contrast"
```

**Recommended themes with Copilot:**
- High Contrast (built-in)
- High Contrast Dark (built-in)

### Font Scaling for Presentations

**Zoom editor font (not UI):**

| Action | Windows/Linux | macOS | Description |
|--------|---------------|-------|-------------|
| **Zoom in** | `Ctrl+=` | `Cmd+=` | Increase font |
| **Zoom out** | `Ctrl+-` | `Cmd+-` | Decrease font |
| **Reset zoom** | `Ctrl+0` | `Cmd+0` | Default size |

**Configure font zoom:**

```json
{
  "editor.fontSize": 14,
  "editor.fontZoomIncrement": 1
}
```

**Keybindings for font zoom:**

```json
[
  {
    "key": "ctrl+=",
    "command": "editor.action.fontZoomIn"
  },
  {
    "key": "ctrl+-",
    "command": "editor.action.fontZoomOut"
  },
  {
    "key": "ctrl+0",
    "command": "editor.action.fontZoomReset"
  }
]
```

### Screencast Mode (For Demos)

**Show keystrokes on screen:**

```json
{
  // Enable screencast mode
  "screencastMode.onlyKeyboardShortcuts": true,
  "screencastMode.fontSize": 56,
  "screencastMode.verticalOffset": 10
}
```

**Toggle screencast mode:**

```
Ctrl+Shift+P → "Toggle Screencast Mode"
```

**Great for:**
- Demos and presentations
- Teaching Copilot workflows
- Recording tutorials

---

## Productivity Workflows

### Workflow 1: Test-Driven Development

**Keyboard-only TDD cycle:**

```
1. Create test file:           Ctrl+P → "MyComponent.test.tsx" → Enter
2. Generate test skeleton:     Ctrl+Alt+I → "/tests" → Enter
3. Review suggested tests:     Arrow keys
4. Accept tests:               Escape to close chat
5. Run tests:                  Ctrl+Shift+P → "Test: Run All" → Enter
6. Tests fail (red):           (expected)
7. Switch to implementation:   Ctrl+P → "MyComponent.tsx" → Enter
8. Implement feature:          Start typing → Tab (Copilot suggests)
9. Run tests again:            Ctrl+Shift+P → "Test: Run All" → Enter
10. Tests pass (green):        ✅ Done
```

**Target cycle time:** <3 minutes per feature

---

### Workflow 2: Code Review

**Review PR changes with Copilot:**

```
1. Open changed files:         Ctrl+Shift+G (Git view) → Arrow keys
2. Review first file:          Enter
3. Ask Copilot:                Ctrl+Alt+I → "@file Review for bugs and issues"
4. Read suggestions:           Scroll through response
5. Fix issues:                 Ctrl+I → "fix the issue mentioned" → Ctrl+Enter
6. Next file:                  Ctrl+Tab
7. Repeat 3-6 for all files
8. Write review comment:       Ctrl+Shift+G → Type comment
9. Approve/Request changes:    Click button (or use GitHub CLI)
```

---

### Workflow 3: Refactoring Session

**Large-scale refactoring:**

```
1. Plan refactoring:           Ctrl+Alt+I → Switch to Plan mode → Describe refactoring
2. Review plan:                Read Copilot's proposed steps
3. Accept plan:                Type "proceed"
4. Switch to Agent mode:       Click "Agent" in Chat view
5. Agent executes plan:        (Copilot makes multi-file changes)
6. Review changes:             Ctrl+Shift+G → Diff view
7. Test changes:               Ctrl+Shift+P → "Test: Run All" → Enter
8. If tests fail:              Ctrl+I → "fix test failures"
9. Commit:                     Ctrl+Shift+G → Type message → Ctrl+Enter
```

---

### Workflow 4: Documentation Sprint

**Generate docs for entire module:**

```
1. Open main file:             Ctrl+P → "UserService.ts" → Enter
2. Generate docs:              Ctrl+Alt+I → "/doc"
3. Accept suggestions:         Escape
4. Next file:                  Ctrl+P → "UserRepository.ts" → Enter
5. Repeat 2-4 for all files
6. Generate module README:     Ctrl+P → "README.md" → Enter
7. Generate content:           Ctrl+Alt+I → "@workspace Generate comprehensive README"
8. Review and edit:            Manual adjustments
9. Commit docs:                Ctrl+Shift+G → Commit
```

**Time savings:** 60-80% reduction vs manual documentation

---

## 🎯 Workshop: Build Muscle Memory

### Part 1: Essential Shortcuts (10 min)

**Practice each shortcut 5 times:**

1. **Accept suggestion:**
   - Type: `function hello`
   - Press `Tab` 5 times to practice acceptance

2. **Cycle suggestions:**
   - Type: `function calculate`
   - Press `Alt+]` 5 times (try different suggestions)

3. **Inline chat:**
   - Select any code
   - Press `Ctrl+I` 5 times
   - Type different instructions each time

**Goal:** <1 second reaction time for each shortcut

---

### Part 2: Create Custom Keybindings (15 min)

1. **Open keybindings.json:**
   ```
   Ctrl+K Ctrl+S → {} icon
   ```

2. **Add 3 custom shortcuts:**
   - One for /tests
   - One for /doc
   - One for /explain

3. **Test each shortcut:**
   - Use them in real code
   - Adjust if they feel awkward

---

### Part 3: Multi-Cursor Practice (10 min)

1. **Create a file with 5 similar functions**

2. **Add JSDoc to all using multi-cursor:**
   ```
   Ctrl+Alt+↓ (4 times to create 5 cursors)
   Type /**
   Tab (accept all 5 suggestions)
   ```

3. **Refactor all functions:**
   ```
   Select first function name
   Ctrl+D (4 times to select all 5)
   Ctrl+I → "add error handling"
   ```

**Goal:** Complete in <2 minutes

---

### Part 4: Speed Challenge (10 min)

**Task:** Complete this workflow as fast as possible

```
1. Create new TypeScript file
2. Generate a class with 3 methods
3. Generate tests for all 3 methods
4. Generate JSDoc for all methods
5. Refactor to use async/await
6. Generate README explaining the class
```

**Target times:**
- Beginner: <5 minutes
- Intermediate: <3 minutes
- Expert: <2 minutes

**Record your time:** ______ minutes

---

## 🎓 Key Takeaways

✅ **The Essential 10 shortcuts cover 80% of daily use**
- Memorize: Tab, Esc, Ctrl+I, Ctrl+Alt+I, Alt+], Alt+[

✅ **Custom keybindings save seconds per action**
- Create shortcuts for frequent tasks (/tests, /doc, /fix)
- Use context-aware bindings (language-specific)

✅ **Multi-cursor + Copilot = powerful batch operations**
- JSDoc for multiple functions
- Consistent refactoring
- Type annotations

✅ **Vim mode works great with Copilot**
- Insert mode: Copilot suggestions active
- Normal mode: Copilot commands via leader key

✅ **Accessibility features make Copilot available to all**
- Screen reader support
- Keyboard-only navigation
- High contrast themes

✅ **Keyboard-only workflows are faster than mouse**
- Target: <2 seconds per action
- Practice builds muscle memory

---

## 📖 Additional Resources

- [VS Code Keyboard Shortcuts (PDF)](https://code.visualstudio.com/shortcuts/keyboard-shortcuts-windows.pdf)
- [VS Code Keybindings Documentation](https://code.visualstudio.com/docs/getstarted/keybindings)
- [Vim Extension](https://marketplace.visualstudio.com/items?itemName=vscodevim.vim)
- [Accessibility Documentation](https://code.visualstudio.com/docs/editor/accessibility)

---

## ✅ Part 2 Complete!

Congratulations! You've mastered VS Code + Copilot integration:

- ✅ Extension architecture and three chat interfaces
- ✅ Workspace optimization for teams
- ✅ Context management and performance tuning
- ✅ Keyboard shortcuts and productivity workflows

### Your Productivity Gains

**Before Part 2:**
- Using Copilot with mouse
- Basic autocomplete only
- Slow context switching
- Manual configuration

**After Part 2:**
- Keyboard-driven workflows (<2s per action)
- All three chat interfaces mastered
- Optimized workspace (20-50% faster responses)
- Team-ready configuration

---

**Part 2 Navigation:**
- [← Section 3: Context & Performance](03-context-performance.md)
- [Next: Part 3 - Advanced Chat & Agents →](../../03-advanced-chat-agents/README.md)
- [↑ Back to Part 2 Overview](README.md)
- [↑ Back to Guide Home](../../README.md)

---

**Last Updated:** December 2025  
**Sources:** Official VS Code and GitHub Copilot documentation via Context7 MCP

---

## 🎉 Next Steps

**Continue your learning:**

**Part 3: Advanced Chat & Agent Development** (4 hours)
- Custom agents and instructions
- Prompt engineering
- MCP server integration
- Subagents for complex tasks

**Or choose your path:**
- **Team Leads:** Part 4 - Enterprise & Team Collaboration
- **Developers:** Part 5 - Specialized Workflows (Frontend/Backend/DevOps)
- **Quality Focus:** Part 6 - Testing & Performance

**Keep practicing:** The more you use keyboard shortcuts, the faster you become!

