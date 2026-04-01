# Claude Code — Complete Overview

> **Package:** `@anthropic-ai/claude-code` v2.1.89  
> **Author:** Anthropic PBC  
> **License:** Proprietary — [Legal & Compliance](https://code.claude.com/docs/en/legal-and-compliance)  
> **Requires:** Node.js ≥ 18.0.0

---

## What is Claude Code?

Claude Code is **Anthropic's agentic AI coding assistant** that lives in your terminal. It connects directly to Claude (Anthropic's LLM) and can:

- **Understand your entire codebase** — reads files, searches code, navigates projects
- **Edit files intelligently** — makes targeted edits using old/new string replacement
- **Run shell commands** — executes bash/terminal commands in a sandboxed environment
- **Handle Git workflows** — commits, branches, PRs, and merge conflict resolution
- **Search the web** — fetches documentation and searches for solutions
- **Work with notebooks** — edits Jupyter `.ipynb` cells
- **Manage tasks** — maintains a TODO list to track multi-step workflows
- **Spawn sub-agents** — delegates tasks to specialized background agents

---

## How It Works — Architecture

### Single-File Bundle

Claude Code ships as a **single minified JavaScript file** (`cli.js` — ~13MB, ~16,800 lines). The entire application is bundled into this one file, including:

- The Anthropic SDK client (for API communication)
- A full terminal UI (TUI) built with Ink/React for the terminal
- File system operations and sandboxing logic
- Git integration
- MCP (Model Context Protocol) server support
- Ripgrep binary (bundled per-platform in `/vendor/ripgrep/`)
- Sharp image processing (optional dependency)

### Entry Point Flow

```
cli.js (#!/usr/bin/env node)
  │
  ├── Parse CLI arguments (--help, --version, -p, etc.)
  ├── Authenticate (API key, OAuth token, or auth token)
  ├── Initialize session (unique UUID per session)
  ├── Load project context (CLAUDE.md files, .claude/ config)
  ├── Start main agent loop
  │     ├── Send messages to Claude API
  │     ├── Receive tool_use responses
  │     ├── Execute tools (Bash, FileEdit, FileRead, etc.)
  │     ├── Return tool results
  │     └── Loop until task complete
  └── Render terminal UI
```

### The Agent Loop

Claude Code operates on an **agentic loop pattern**:

1. **User sends a message** (natural language prompt)
2. **Claude API responds** with either text or `tool_use` blocks
3. **Tools are executed** locally (file reads, edits, shell commands, etc.)
4. **Tool results are sent back** to Claude as `tool_result` messages
5. **Claude continues reasoning** and may call more tools
6. **Loop ends** when Claude responds with only text (no more tool calls)

This is the same pattern used by all modern AI coding agents, but Claude Code's implementation is notable for its:
- **Permission system** — asks user approval for destructive operations
- **Sandboxed bash** — commands run in a restricted environment
- **Context management** — automatic compaction when context gets too large
- **Multi-agent support** — can spawn sub-agents for parallel work

---

## Core Tools (What Claude Can Do)

The SDK type definitions (`sdk-tools.d.ts`) reveal **19 built-in tools**:

| Tool | Purpose |
|------|---------|
| `Agent` | Spawn a sub-agent for parallel/background tasks |
| `Bash` | Execute shell commands (with timeout & sandbox options) |
| `TaskOutput` | Get output from background tasks |
| `ExitPlanMode` | Transition from planning to execution mode |
| `FileEdit` | Replace specific text in a file (old_string → new_string) |
| `FileRead` | Read file contents (text, images, PDFs, notebooks) |
| `FileWrite` | Write/create files |
| `Glob` | Find files matching glob patterns |
| `Grep` | Search file contents with regex (uses ripgrep) |
| `TaskStop` | Stop a background task |
| `ListMcpResources` | List available MCP server resources |
| `Mcp` | Execute MCP tool calls |
| `NotebookEdit` | Edit Jupyter notebook cells |
| `ReadMcpResource` | Read a specific MCP resource |
| `TodoWrite` | Manage a task/TODO list |
| `WebFetch` | Fetch and process web page content |
| `WebSearch` | Search the web |
| `AskUserQuestion` | Present multiple-choice questions to the user |
| `Config` | Get/set configuration values |

### Tool Input/Output Examples

**FileEdit** — The primary code editing tool:
```typescript
{
  file_path: "/absolute/path/to/file.ts",  // Must be absolute
  old_string: "const x = 1;",              // Exact match required
  new_string: "const x = 42;",             // Replacement text
  replace_all: false                        // Optional: replace all occurrences
}
```

**Bash** — Shell command execution:
```typescript
{
  command: "npm test",
  timeout: 60000,              // Max 600,000ms (10 minutes)
  description: "Run test suite",
  run_in_background: false,    // Optional: background execution
  dangerouslyDisableSandbox: false  // Override sandbox (dangerous!)
}
```

**Agent** — Sub-agent spawning:
```typescript
{
  description: "Fix CSS styles",
  prompt: "Update the button styles in components/Button.tsx...",
  model: "sonnet",              // Optional: sonnet, opus, or haiku
  run_in_background: true,      // Run asynchronously
  isolation: "worktree",        // Optional: use git worktree for isolation
  mode: "plan"                  // Permission mode
}
```

---

## File Structure

```
@anthropic-ai/claude-code/
├── cli.js              # Main application (minified, ~13MB)
├── package.json        # Package manifest
├── README.md           # Official README
├── LICENSE.md          # Proprietary license
├── sdk-tools.d.ts      # TypeScript type definitions for all tools
├── bun.lock            # Lock file (built with Bun)
├── node_modules/       # Optional dependencies (sharp for images)
└── vendor/
    └── ripgrep/        # Bundled ripgrep binaries per platform
        ├── arm64-darwin/
        ├── x64-darwin/
        ├── x64-linux/
        ├── x64-win32/
        └── COPYING
```

---

## Key Internal Systems

### 1. Session Management
- Each session gets a unique UUID
- Sessions track: cost, tokens used, duration, lines changed
- Sessions can be persisted and resumed
- Parent-child session relationships for sub-agents

### 2. Model Usage Tracking
- Tracks input/output tokens per model
- Tracks cache read/creation tokens
- Monitors total cost in USD
- Supports budget continuation counts

### 3. Permission Modes
- `default` — asks for permission on destructive operations
- `plan` — requires plan approval before execution
- `acceptEdits` — auto-accepts file edits
- `bypassPermissions` — skips all permission checks
- `dontAsk` — non-interactive mode

### 4. Context Compaction
- When context window fills up, Claude Code automatically compacts
- Summarizes conversation history to free up token space
- Preserves critical information while reducing token count

### 5. CLAUDE.md Configuration
Claude Code reads `CLAUDE.md` files from your project for:
- Project-specific instructions
- Custom commands and workflows
- Style preferences and conventions

---

## Supported Models

From the source code, Claude Code supports these model families:
- `claude-opus-4` / `claude-opus-4-1` (highest capability)
- `claude-sonnet-4` / `claude-sonnet-4-5` / `claude-sonnet-4-6`
- `claude-haiku-4-5` (fastest, cheapest)
- Legacy: `claude-3-7-sonnet`, `claude-3-5-haiku` (deprecated)

### Streaming Timeout
- Non-streaming requests: max 10 minutes (600,000ms)
- Some models (Opus 4) have 8,192 max output token defaults
- Streaming is **required** for operations that may exceed 10 minutes

---

## Data & Privacy

- Claude Code collects usage data (acceptance/rejections)
- Conversation data is sent to Anthropic's API
- The `/bug` command sends feedback reports
- Privacy safeguards include limited retention periods
- Full details: [Privacy Policy](https://www.anthropic.com/legal/privacy)

---

*This document is based on analysis of `@anthropic-ai/claude-code` v2.1.89.*
