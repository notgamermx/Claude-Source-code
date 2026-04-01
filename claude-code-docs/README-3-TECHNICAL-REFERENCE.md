# Claude Code — SDK, API & Technical Deep Dive

> A developer-focused guide to Claude Code's internal architecture, SDK integration,
> tool schemas, and how to extend or build on top of it.

---

## Table of Contents

1. [SDK & Programmatic Usage](#sdk--programmatic-usage)
2. [Complete Tool Schema Reference](#complete-tool-schema-reference)
3. [Anthropic API Integration](#anthropic-api-integration)
4. [Internal State Machine](#internal-state-machine)
5. [Bundled Dependencies](#bundled-dependencies)
6. [Source Code Analysis](#source-code-analysis)
7. [Security & Sandboxing](#security--sandboxing)
8. [Extending Claude Code](#extending-claude-code)

---

## SDK & Programmatic Usage

Claude Code can be used programmatically via its SDK. The `sdk-tools.d.ts` file (2,724 lines of TypeScript definitions) defines every tool's input/output schema.

### Import & Use as SDK

```typescript
import Anthropic from '@anthropic-ai/sdk';

const client = new Anthropic({
  apiKey: process.env.ANTHROPIC_API_KEY,
});

// Create a message with tool use
const message = await client.messages.create({
  model: 'claude-sonnet-4-6',
  max_tokens: 4096,
  messages: [
    { role: 'user', content: 'Read the file at /path/to/file.ts' }
  ],
  tools: [
    {
      name: 'FileRead',
      description: 'Read a file from the filesystem',
      input_schema: {
        type: 'object',
        properties: {
          file_path: { type: 'string', description: 'Absolute path to read' },
          offset: { type: 'number', description: 'Starting line number' },
          limit: { type: 'number', description: 'Number of lines to read' }
        },
        required: ['file_path']
      }
    }
  ]
});
```

### Streaming Responses

```typescript
const stream = client.messages.stream({
  model: 'claude-sonnet-4-6',
  max_tokens: 4096,
  messages: [{ role: 'user', content: 'Hello' }],
});

stream.on('text', (text) => process.stdout.write(text));
stream.on('message', (message) => console.log('Done:', message));

await stream.finalMessage();
```

---

## Complete Tool Schema Reference

### Input Schemas

#### `AgentInput` — Spawn Sub-Agents
```typescript
interface AgentInput {
  description: string;        // 3-5 word task description
  prompt: string;             // Full task instructions
  subagent_type?: string;     // Specialized agent type
  model?: "sonnet" | "opus" | "haiku";  // Model override
  run_in_background?: boolean; // Async execution
  name?: string;              // Addressable agent name
  team_name?: string;         // Team context
  mode?: "acceptEdits" | "bypassPermissions" | "default" | "dontAsk" | "plan";
  isolation?: "worktree";     // Git worktree isolation
}
```

#### `BashInput` — Shell Commands
```typescript
interface BashInput {
  command: string;              // The command to execute
  timeout?: number;             // Max 600,000ms (10 min)
  description?: string;         // Human-readable description
  run_in_background?: boolean;  // Background execution
  dangerouslyDisableSandbox?: boolean;  // Override sandbox
}
```

#### `FileEditInput` — Code Editing
```typescript
interface FileEditInput {
  file_path: string;    // Absolute path (required)
  old_string: string;   // Exact text to find
  new_string: string;   // Replacement text (must differ)
  replace_all?: boolean; // Replace all occurrences (default: false)
}
```

#### `FileReadInput` — File Reading
```typescript
interface FileReadInput {
  file_path: string;    // Absolute path (required)
  offset?: number;      // Start line (for large files)
  limit?: number;       // Number of lines
  pages?: string;       // PDF page range (e.g., "1-5")
}
```

#### `FileWriteInput` — File Creation/Writing
```typescript
interface FileWriteInput {
  file_path: string;    // Absolute path (required)
  content: string;      // Full file contents
}
```

#### `GlobInput` — File Pattern Matching
```typescript
interface GlobInput {
  pattern: string;      // Glob pattern (e.g., "**/*.ts")
  path?: string;        // Directory to search (default: cwd)
}
```

#### `GrepInput` — Code Search (ripgrep)
```typescript
interface GrepInput {
  pattern: string;               // Regex pattern
  path?: string;                 // Search directory
  glob?: string;                 // File filter (e.g., "*.{ts,tsx}")
  output_mode?: "content" | "files_with_matches" | "count";
  "-B"?: number;                 // Lines before match
  "-A"?: number;                 // Lines after match
  "-C"?: number;                 // Context lines
  "-n"?: boolean;                // Show line numbers
  "-i"?: boolean;                // Case insensitive
  type?: string;                 // File type (js, py, etc.)
  head_limit?: number;           // Limit results (default: 250)
  offset?: number;               // Skip N results
  multiline?: boolean;           // Multi-line mode
}
```

#### `WebSearchInput` — Web Search
```typescript
interface WebSearchInput {
  query: string;                 // Search query
  allowed_domains?: string[];    // Only these domains
  blocked_domains?: string[];    // Exclude these domains
}
```

#### `WebFetchInput` — URL Fetching
```typescript
interface WebFetchInput {
  url: string;          // URL to fetch
  prompt: string;       // What to extract from the content
}
```

#### `TodoWriteInput` — Task Management
```typescript
interface TodoWriteInput {
  todos: {
    content: string;                              // Task description
    status: "pending" | "in_progress" | "completed";
    activeForm: string;                           // Active phrasing
  }[];
}
```

#### `NotebookEditInput` — Jupyter Notebooks
```typescript
interface NotebookEditInput {
  notebook_path: string;                    // Absolute path
  cell_id?: string;                         // Cell to edit
  new_source: string;                       // New cell content
  cell_type?: "code" | "markdown";          // Cell type
  edit_mode?: "replace" | "insert" | "delete";
}
```

#### `AskUserQuestionInput` — Interactive Questions
```typescript
interface AskUserQuestionInput {
  questions: {
    question: string;       // The question text
    header: string;         // Short label (max 12 chars)
    options: {              // 2-4 options
      label: string;        // Option text (1-5 words)
      description: string;  // What this option means
      preview?: string;     // Preview content
    }[];
    multiSelect: boolean;   // Allow multiple selections
  }[];  // 1-4 questions
}
```

#### `ConfigInput` — Configuration
```typescript
interface ConfigInput {
  // Get or set Claude Code configuration values
  [k: string]: unknown;
}
```

#### `EnterWorktreeInput` / `ExitWorktreeInput` — Git Worktree
```typescript
// Enter: creates an isolated copy of the repo
// Exit: cleans up or keeps the worktree
interface ExitWorktreeOutput {
  action: "keep" | "remove";
  originalCwd: string;
  worktreePath: string;
  worktreeBranch?: string;
  discardedFiles?: number;
  discardedCommits?: number;
  message: string;
}
```

---

### Output Schemas

#### `AgentOutput` — Sub-Agent Results
```typescript
type AgentOutput =
  | {
      status: "completed";
      agentId: string;
      content: { type: "text"; text: string }[];
      totalToolUseCount: number;
      totalDurationMs: number;
      totalTokens: number;
      usage: {
        input_tokens: number;
        output_tokens: number;
        cache_creation_input_tokens: number | null;
        cache_read_input_tokens: number | null;
        server_tool_use: {
          web_search_requests: number;
          web_fetch_requests: number;
        } | null;
        service_tier: "standard" | "priority" | "batch" | null;
      };
      prompt: string;
    }
  | {
      status: "async_launched";
      agentId: string;
      description: string;
      prompt: string;
      outputFile: string;     // File to check progress
    };
```

#### `FileReadOutput` — Supports Multiple Formats
```typescript
type FileReadOutput =
  | { type: "text"; file: { filePath: string; content: string; numLines: number; startLine: number; totalLines: number } }
  | { type: "image"; file: { base64: string; type: "image/jpeg" | "image/png" | "image/gif" | "image/webp"; dimensions?: {...} } }
  | { type: "notebook"; file: { filePath: string; cells: unknown[] } }
  | { type: "pdf"; file: { filePath: string; base64: string; originalSize: number } }
  | { type: "parts"; file: { filePath: string; count: number; outputDir: string } }
  | { type: "file_unchanged"; file: { filePath: string } };
```

---

## Anthropic API Integration

### Embedded SDK

Claude Code bundles the full **Anthropic TypeScript SDK** (v0.74.0) directly into `cli.js`. Key classes:

```
Anthropic (client)
├── messages.create()      — Create chat completions
├── messages.stream()      — Stream responses
├── messages.countTokens() — Count tokens before sending
├── beta.messages.*        — Beta features
├── beta.files.*           — File API
├── beta.models.*          — Model listing
└── completions.create()   — Legacy completions
```

### Authentication Methods

```typescript
// 1. API Key (most common)
const client = new Anthropic({ apiKey: "sk-ant-..." });

// 2. Auth Token (OAuth/bearer)
const client = new Anthropic({ authToken: "Bearer ..." });

// 3. Environment Variables (auto-detected)
// Set ANTHROPIC_API_KEY or ANTHROPIC_AUTH_TOKEN

// 4. File Descriptor (for secure passing)
// --api-key-fd <fd> or --oauth-token-fd <fd>
```

### API Versioning
- API Version: `2023-06-01` (sent as `anthropic-version` header)
- Beta features use `anthropic-beta` header
- Supported betas: `files-api-2025-04-14`, `message-batches-2024-09-24`, `token-counting-2024-11-01`, `structured-outputs-2025-12-15`, `skills-2025-10-02`

### Rate Limiting & Retries
- Default max retries: **2**
- Exponential backoff with jitter
- Respects `retry-after` and `retry-after-ms` headers
- Auto-retries on: 408, 409, 429, 5xx errors

---

## Internal State Machine

Claude Code maintains a rich internal state tracked in a global singleton:

```typescript
// Key state properties (from source analysis)
{
  sessionId: string,              // UUID per session
  parentSessionId?: string,       // For sub-agent sessions
  originalCwd: string,            // Where claude was launched
  projectRoot: string,            // Detected project root
  cwd: string,                    // Current working directory
  
  // Cost tracking
  totalCostUSD: number,
  totalAPIDuration: number,
  totalToolDuration: number,
  totalLinesAdded: number,
  totalLinesRemoved: number,
  
  // Model configuration
  modelUsage: Record<string, ModelUsage>,
  mainLoopModelOverride?: string,
  initialMainLoopModel: string | null,
  modelStrings: ModelStrings | null,
  
  // Session settings
  isInteractive: boolean,
  clientType: "cli" | "claude-vscode" | "sdk" | ...,
  sessionSource?: string,
  sessionBypassPermissionsMode: boolean,
  
  // Context management
  cachedClaudeMdContent: string | null,
  systemPromptSectionCache: Map,
  invokedSkills: Map,
  registeredHooks: HookCallbacks | null,
  
  // Telemetry
  meter: OTel Meter,
  tracerProvider: OTel TracerProvider,
  eventLogger: EventLogger,
  
  // Multi-agent
  agentColorMap: Map<string, Color>,
  mainThreadAgentType?: string,
  sessionCreatedTeams: Set<string>,
}
```

### Context Compaction

When the context window approaches its limit:

1. **Token counting** — tracks input/output tokens per turn
2. **Threshold check** — default 100,000 tokens triggers compaction
3. **Summary generation** — Claude summarizes the conversation
4. **Message replacement** — old messages are replaced with the summary
5. **Continuation** — the conversation continues with compressed context

---

## Bundled Dependencies

### Runtime Dependencies (in cli.js)
- **Anthropic SDK** v0.74.0 — API client
- **Ink** — React-based terminal UI framework
- **React** — Component rendering for TUI
- **Lodash** (partial) — Utility functions (cloneDeep, get, set, etc.)
- **Ripgrep** — Fast code search (platform-specific binaries in `/vendor/`)
- **Sharp** — Image processing (optional, per-platform)

### Platform Binaries

```
vendor/ripgrep/
├── arm64-darwin/rg    # macOS Apple Silicon
├── x64-darwin/rg      # macOS Intel
├── x64-linux/rg       # Linux x64
└── x64-win32/rg.exe   # Windows x64
```

---

## Security & Sandboxing

### Bash Sandbox
Claude Code runs shell commands in a **sandboxed environment**:

- Commands are analyzed before execution
- Destructive commands require user permission
- Timeout enforcement (max 10 minutes)
- Background execution support with output capture
- Option to disable sandbox (`dangerouslyDisableSandbox`) — not recommended

### File System Protection
- All file paths must be **absolute** (prevents path traversal)
- `FileEdit` requires exact string matching (prevents accidental edits)
- Symlink resolution and loop detection
- FIFO/socket/device files are handled specially

### API Security
- API keys never logged (masked in debug output as `***`)
- Supports secure key passing via file descriptors
- Browser usage blocked by default (`dangerouslyAllowBrowser` required)
- CORS headers sent for browser contexts

---

## Extending Claude Code

### Via MCP Servers

The most supported extension mechanism. Create an MCP server:

```typescript
// my-mcp-server.ts
import { Server } from "@modelcontextprotocol/sdk/server/index.js";

const server = new Server({
  name: "my-tool",
  version: "1.0.0",
});

server.setRequestHandler("tools/list", async () => ({
  tools: [{
    name: "query_database",
    description: "Query the production database",
    inputSchema: {
      type: "object",
      properties: {
        sql: { type: "string", description: "SQL query" }
      }
    }
  }]
}));

server.setRequestHandler("tools/call", async (request) => {
  // Handle tool execution
});
```

Register in `.claude/mcp.json`:
```json
{
  "servers": {
    "my-tool": {
      "command": "node",
      "args": ["./my-mcp-server.js"]
    }
  }
}
```

### Via Hooks

Claude Code supports **hooks** — callbacks that run at specific lifecycle points:

- Pre-tool execution
- Post-tool execution
- On message
- On error

Hooks can be registered via plugins or CLAUDE.md configuration.

### Via CLAUDE.md

The simplest extension — add instructions in `CLAUDE.md`:

```markdown
## Custom Commands
When I say "deploy", run:
1. `npm run build`
2. `npm run test`
3. `vercel deploy --prod`

## Code Style
- Always use named exports
- Prefer `async/await` over `.then()`
- Maximum function length: 50 lines
```

---

## Key Source Code Patterns

### Error Handling Hierarchy
```
AnthropicError (base)
├── APIError (HTTP errors)
│   ├── BadRequestError (400)
│   ├── AuthenticationError (401)
│   ├── PermissionDeniedError (403)
│   ├── NotFoundError (404)
│   ├── ConflictError (409)
│   ├── UnprocessableEntityError (422)
│   ├── RateLimitError (429)
│   └── InternalServerError (5xx)
├── APIConnectionError
│   └── APIConnectionTimeoutError
└── APIUserAbortError
```

### Event System
Claude Code uses a custom pub/sub event system:
```typescript
// Internal event emitter pattern
function createEventEmitter() {
  const subscribers = new Set();
  return {
    subscribe(callback) { ... },
    emit(...args) { ... },
    clear() { ... }
  };
}
```

### File I/O Abstraction
All file operations go through a unified `M8()` interface that wraps Node.js `fs` with:
- Automatic error handling
- Performance tracking (slow operation detection)
- NFC unicode normalization for paths
- Symlink resolution

---

## Telemetry & Observability

Claude Code integrates with **OpenTelemetry**:

```typescript
// Tracked metrics
claude_code.session.count          // Sessions started
claude_code.lines_of_code.count    // Lines added/removed
claude_code.pull_request.count     // PRs created
claude_code.commit.count           // Commits created
claude_code.cost.usage             // Cost in USD
claude_code.token.usage            // Tokens consumed
claude_code.code_edit_tool.decision // Edit accept/reject
claude_code.active_time.total      // Active time in seconds
```

---

## Version History Notes

| Version | Notable Changes |
|---------|----------------|
| 2.1.89 | Current version (as of this analysis) |
| 2.1.88 | Source maps accidentally included in npm package |

### Deprecated Models (from source)
- `claude-3-sonnet-20240229` → EOL July 21, 2025
- `claude-3-opus-20240229` → EOL January 5, 2026
- `claude-2.0`, `claude-2.1` → EOL July 21, 2025
- `claude-3-7-sonnet-*` → EOL February 19, 2026
- `claude-3-5-haiku-*` → EOL February 19, 2026

---

*This technical reference is based on analysis of `@anthropic-ai/claude-code` v2.1.89 source code.*
