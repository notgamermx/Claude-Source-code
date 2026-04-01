# Claude Code — Source Code Analysis

> This document provides an analysis of the community-reconstructed, reverse-engineered source code for Anthropic's `claude-code` CLI based on the `sanbuphy/claude-code-source-code` repository.

---

## 1. Overview of the Reconstructed Source

Unlike the published `claude-code` NPM package (which is a single 13MB minified `cli.js` file), this repository contains the **un-minified, reconstructed TypeScript source files**. 

It reveals how Anthropic actually structured the project during development.

**Key Technologies Used:**
- **Language**: TypeScript (`.ts` and `.tsx`)
- **Runtime & Bundler**: Bun (`bun:bundle` is used heavily, and there is a `bun.lock` file)
- **Terminal UI**: React and Ink (`.tsx` files are used to write React components that render in the terminal)
- **API Client**: `@anthropic-ai/sdk`

---

## 2. Directory Structure

The source is cleanly separated into several key domain areas:

### `/src` - The Core Application
This is where the agentic logic, terminal interface, and tool definitions live.

- **`cli/` and `entrypoints/`**: Handle parsing terminal arguments, instantiating the SDK, and starting the interactive session.
- **`components/` and `screens/`**: Define the Terminal UI using React (Ink). Examples include `MessageSelector.tsx`, `interactiveHelpers.tsx`, etc.
- **`tools/`**: Contains the implementations of the 19 tools we saw in the types (e.g., `FileEditTool`, `BashTool`, `AgentTool`).
- **`query/` and `QueryEngine.ts`**: The core loop that talks to the Anthropic API, handles permission denials, usage tracking, sub-agent coordination, and context memory limits.
- **`commands.ts`**: Implements slash commands (`/cost`, `/bug`, `/compact`, etc.).
- **`services/api/claude.ts`**: The direct wrapper around the Anthropic API and retry logic.

### `/tools` - Tool Capabilities
A top-level directory containing Python/Shell scripts that the Javascript tools shell out to (such as git worktree isolation scripts, Python notebook runners, etc.).

### `/stubs` and `/vendor`
Stubs for native dependencies and pre-compiled binaries (like `ripgrep` and `audio-capture`).

---

## 3. Notable Architectural Discoveries

From reading the recreated TypeScript files (like `QueryEngine.ts`), several advanced architectural patterns emerge:

### The "QueryEngine" State Machine
The core intelligence lives in `QueryEngine.ts`. 
- Every message is processed through `processUserInput()`.
- It dynamically updates a `ToolPermissionContext`.
- It tracks a `totalBudgetUsd` to prevent runaway spending loop.
- It intercepts specific tools (like sub-agents) and processes them locally to spawn new isolated processes.

### Memory & System Prompt Assembly
The system prompt is dynamically assembled right before sending using `fetchSystemPromptParts()`. It collects:
1. Base Anthropic system instructions
2. User-provided `CLAUDE.md` context (`customSystemPrompt`)
3. Working directory state
4. Tool schemas
5. "Dynamic Skill Dir Triggers" — Claude can "discover" new skills mid-conversation.

### Advanced "Snip" Compaction
The code includes a feature called `HISTORY_SNIP`. When the token window limits are reached, it doesn't just trim old messages. It generates a `compact_boundary` system message that mathematically summarises the discarded context while preserving references, ensuring the agent doesn't "hallucinate" missing history.

### The "Coordinator" Mode (`COORDINATOR_MODE`)
There is conditionally compiled code for a "Coordinator" mode (`getCoordinatorUserContext`). This implies Anthropic uses, or intends to use, Claude Code as a generic MCP (Model Context Protocol) router where multiple distinct LLM agents talk to each other to solve larger repository-wide tasks.

---

## 4. How to Run This Version

If you want to run this un-minified version instead of the official NPM installed version:

```bash
# Navigate to the cloned repo
cd claude-code-source-code

# Install dependencies
npm install

# Run the un-minified entrypoint
# (assuming you have your ANTHROPIC_API_KEY set)
npm start 
```

*Note: Since this is an unofficial reconstructed version, behavior may differ slightly from the official minified release due to reversed variables or experimental feature flags (like `CLAUDE_COWORK_MEMORY_PATH_OVERRIDE`) left enabled.*
