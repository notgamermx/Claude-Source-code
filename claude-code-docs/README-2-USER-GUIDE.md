# Claude Code — How to Use It (User Guide)

> A practical guide to installing, configuring, and using Claude Code effectively.

---

## Installation

### Prerequisites
- **Node.js 18+** installed ([download](https://nodejs.org/))
- An **Anthropic API key** ([get one](https://console.anthropic.com/))

### Install Globally

```bash
npm install -g @anthropic-ai/claude-code
```

### Verify Installation

```bash
claude --version
# Output: 2.1.89 (or latest)
```

---

## Quick Start

### 1. Set Your API Key

```bash
# Option A: Environment variable
export ANTHROPIC_API_KEY="sk-ant-..."

# Option B: Claude will prompt you on first run
```

### 2. Navigate to Your Project

```bash
cd /path/to/your/project
```

### 3. Launch Claude Code

```bash
claude
```

That's it! You're now in an interactive AI coding session.

---

## CLI Commands & Flags

### Basic Usage

```bash
# Interactive mode (default)
claude

# One-shot prompt mode (-p flag)
claude -p "explain how the auth module works"

# Pipe input
cat error.log | claude -p "what's causing this error?"

# Resume previous conversation
claude --resume

# Print output as JSON
claude -p "list all API routes" --output-format json
```

### Common Flags

| Flag | Description |
|------|-------------|
| `-p "prompt"` | Run a single prompt and exit (non-interactive) |
| `--resume` | Resume the previous conversation |
| `--model <model>` | Override the default model |
| `--debug` / `-d` | Enable debug logging |
| `--version` | Print version |
| `--help` | Show all available options |
| `--bare` | Simple/minimal output mode |
| `-e KEY=value` | Set environment variables |
| `--output-format json` | Output results as JSON |

---

## What You Can Ask Claude Code to Do

### 🔍 Understand Code
```
"Explain how the authentication middleware works"
"What does the function processPayment do?"
"Find all places where we handle user sessions"
"Summarize the architecture of this project"
```

### ✏️ Edit Code
```
"Add input validation to the signup form"
"Refactor the UserService to use dependency injection"
"Fix the bug where null emails crash the app"
"Add TypeScript types to the utils module"
```

### 🧪 Test & Debug
```
"Write unit tests for the CartService"
"Run the tests and fix any failures"
"Debug why the API returns 500 on /api/users"
"Add error handling to the database connection"
```

### 🔧 DevOps & Git
```
"Create a new branch called feature/dark-mode"
"Commit these changes with a descriptive message"
"Show me the diff of recent changes"
"Set up a GitHub Actions CI pipeline"
```

### 🌐 Research & Learn
```
"Search for the best React state management library for our use case"
"What's the latest syntax for Next.js 15 app router?"
"Fetch the documentation for the Stripe API"
```

---

## Slash Commands (Inside Claude Code)

While in an interactive session, you can use these special commands:

| Command | What It Does |
|---------|-------------|
| `/help` | Show all available commands |
| `/bug` | Report a bug to Anthropic |
| `/clear` | Clear the conversation history |
| `/compact` | Manually compact the context to save tokens |
| `/cost` | Show current session cost and token usage |
| `/doctor` | Diagnose configuration issues |
| `/init` | Initialize a CLAUDE.md file for your project |
| `/login` | Authenticate with Anthropic |
| `/logout` | Sign out |
| `/model` | Switch the active model |
| `/permissions` | View/manage tool permissions |
| `/status` | Show session status |
| `/todo` | View/manage the task list |

---

## Configuration with CLAUDE.md

Create a `CLAUDE.md` file in your project root to give Claude persistent context:

```markdown
# Project: My E-Commerce App

## Tech Stack
- Next.js 15 with App Router
- PostgreSQL with Prisma ORM
- Tailwind CSS
- Stripe for payments

## Conventions
- Use TypeScript strict mode
- Components go in src/components/
- API routes go in src/app/api/
- Use server components by default
- Always write tests for new features

## Important Notes
- Never modify the migration files directly
- The admin panel uses a separate auth system
- Environment variables are in .env.local (never commit)
```

### CLAUDE.md Locations (in priority order)
1. **Project root** — `./CLAUDE.md`
2. **Home directory** — `~/.claude/CLAUDE.md` (global settings)
3. **Additional directories** — configurable

---

## Permission System

Claude Code asks for permission before potentially destructive operations:

### What Requires Permission
- ✅ **File edits** — modifying existing files
- ✅ **File writes** — creating new files
- ✅ **Shell commands** — running bash/terminal commands
- ✅ **Web requests** — fetching external URLs

### Permission Modes

```bash
# Default: asks for each operation
claude

# Accept all file edits automatically
claude --mode acceptEdits

# Plan mode: creates a plan first, then asks for approval
claude --mode plan

# Skip all permissions (use with caution!)
claude --mode bypassPermissions

# Non-interactive: doesn't ask, skips on uncertainty
claude --mode dontAsk
```

---

## Multi-Agent & Background Tasks

Claude Code can spawn **sub-agents** for parallel work:

```
"Can you refactor the API routes and simultaneously update the tests?"
```

Claude will:
1. Spawn a sub-agent to refactor API routes
2. Spawn another sub-agent for test updates
3. Both work in parallel (optionally in separate git worktrees)
4. Results are merged back

### Git Worktree Isolation
Sub-agents can use **git worktree isolation** to work on separate copies of the repo, preventing conflicts between parallel tasks.

---

## MCP (Model Context Protocol) Integration

Claude Code supports **MCP servers** — external tools that extend Claude's capabilities:

```json
// .claude/mcp.json
{
  "servers": {
    "my-database": {
      "command": "npx",
      "args": ["@my-org/mcp-database-server"],
      "env": {
        "DATABASE_URL": "postgresql://..."
      }
    }
  }
}
```

This lets Claude interact with databases, APIs, or custom tools through MCP.

---

## Cost & Token Management

### Monitor Costs
```
/cost
```
Shows:
- Total tokens used (input + output)
- Cache read/creation tokens
- Estimated cost in USD
- Session duration

### Budget Tips
- Use `/compact` to reduce context when it gets large
- Use `claude-haiku-4-5` for simple tasks (cheaper)
- Use `-p` mode for quick one-shot questions
- The `--model` flag lets you choose cost-appropriate models

---

## Environment Variables

| Variable | Purpose |
|----------|---------|
| `ANTHROPIC_API_KEY` | Your API key |
| `ANTHROPIC_AUTH_TOKEN` | OAuth/bearer token |
| `ANTHROPIC_BASE_URL` | Custom API endpoint |
| `CLAUDE_CONFIG_DIR` | Custom config directory (default: `~/.claude`) |
| `CLAUDE_CODE_DEBUG_LOGS_DIR` | Debug log output directory |
| `DEBUG=1` | Enable debug mode |
| `CLAUDE_CODE_SIMPLE=1` | Bare/simple output mode |

---

## Tips & Best Practices

### 1. Be Specific
```
❌ "Fix the bug"
✅ "Fix the null pointer exception in UserService.getProfile() when the user has no avatar"
```

### 2. Provide Context
```
✅ "In the auth module (src/auth/), add rate limiting to the login endpoint. 
    We use Express and Redis for caching."
```

### 3. Use CLAUDE.md
Set up project context once, and every session benefits from it.

### 4. Break Down Large Tasks
```
✅ "Let's refactor the payment system. Start by:
    1. Listing all files that handle payments
    2. Creating a plan for the refactor
    3. Then we'll execute step by step"
```

### 5. Review Changes
Always review Claude's edits before committing. Use:
```bash
git diff
```

---

## Troubleshooting

### "Authentication Error"
```bash
# Check your API key
echo $ANTHROPIC_API_KEY

# Re-login
claude /login
```

### "Command not found: claude"
```bash
# Reinstall globally
npm install -g @anthropic-ai/claude-code

# Or check npm global bin path
npm config get prefix
```

### "Context too large"
```
/compact
# This summarizes the conversation to free up token space
```

### Debug Mode
```bash
claude --debug
# Or
claude -d
```

---

## Connect & Get Help

- 📖 **Documentation:** [code.claude.com/docs](https://code.claude.com/docs/en/overview)
- 🐛 **Report Bugs:** Use `/bug` command or [GitHub Issues](https://github.com/anthropics/claude-code/issues)
- 💬 **Discord:** [Claude Developers Discord](https://anthropic.com/discord)
- 🏠 **Homepage:** [claude.com/product/claude-code](https://claude.com/product/claude-code)

---

*This guide is based on `@anthropic-ai/claude-code` v2.1.89.*
