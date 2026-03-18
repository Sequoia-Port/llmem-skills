---
name: llmem-mcp
description: MCP server integration for LLMem persistent memory. Provides search_memories and save_memory as native MCP tools instead of HTTP calls. Use this for agents that support the Model Context Protocol (Claude Code, Cursor, etc.) for tighter integration than the URL-based llmem-memory skill.
---

# LLMem MCP Server

For agents that support the Model Context Protocol, LLMem can be registered as an MCP server. This gives you native `search_memories` and `save_memory` tools instead of making HTTP calls.

## Installation

### Option 1: npx (no install)

Add to your agent's MCP configuration:

**Claude Code** (`~/.claude/settings.json`):
```json
{
  "mcpServers": {
    "llmem": {
      "command": "npx",
      "args": ["-y", "@llmem/mcp-server"],
      "env": {
        "LLMEM_API_KEY": "your-key-here"
      }
    }
  }
}
```

**Cursor** (`.cursor/mcp.json`):
```json
{
  "mcpServers": {
    "llmem": {
      "command": "npx",
      "args": ["-y", "@llmem/mcp-server"],
      "env": {
        "LLMEM_API_KEY": "your-key-here"
      }
    }
  }
}
```

### Option 2: Global install

```bash
npm install -g @llmem/mcp-server
```

Then reference `llmem-mcp-server` as the command instead of `npx -y @llmem/mcp-server`.

## Available Tools

Once registered, the MCP server exposes these tools:

### `remember` (recommended)

Save information using natural language. Haiku automatically parses raw text into structured memories.

**Parameters:**
- `text` (string, required): Natural language description of what to remember. Example: "User switched from Neovim to VS Code"

**Returns:** JSON with `status` (saved/updated), `namespace`, `key`, and `value` echo fields.

### `recall_memories`

Search the user's memory store. Call this at the start of every conversation and whenever the topic changes.

**Parameters:**
- `query` (string, required): Natural language question about the user. Example: "what do I know about this user's coding preferences"
- `namespace` (string, optional): Filter to a specific namespace like "preferences" or "projects"
- `limit` (number, optional): Max results to return. Default: 20

**Returns:** JSON with `mems` array, `_hint` (contextual notes), and `questions_to_consider` (contradictions/patterns).

### `save_memory` (structured saves)

Save a new fact with explicit structure. Use when you need fine-grained control over namespace, key, and value.

**Parameters:**
- `namespace` (string, required): Category — identity, preferences, work, projects, interests, health, family, goals, opinions, location, style
- `key` (string, required): Short descriptive label — full_name, current_project, favorite_editor
- `value` (string, required): The actual information, with temporal context when relevant

**Returns:** Confirmation with the saved memory's ID.

## Behavior Guidelines

The same rules from the `llmem-memory` skill apply:

1. **Search before every response** — recall the user's context first using `recall_memories`.
2. **Weave memories naturally** — don't list them back to the user.
3. **Save proactively** — when the user shares something about themselves, use `remember` with natural language.
4. **Confirm saves** — briefly tell the user what was stored using the response: "Saved: preferences/editor = neovim"
5. **Surface contradictions** — if `questions_to_consider` highlights an evolution, ask about it.
6. **Prefer recent memories** — when old and new conflict, the new one wins.
7. **Never store the API key** — it's in the environment, not in memory.
8. **Use `remember` for most saves** — it handles parsing automatically. Use `save_memory` only when you need explicit control.

## Environment Variables

| Variable | Default | Description |
|---|---|---|
| `LLMEM_API_KEY` | (required) | User's LLMem API key |
| `LLMEM_API_BASE` | `https://api.llmem.io` | API base URL |
| `LLMEM_MCP_PORT` | `3100` | Port for stdio transport (if using HTTP mode) |
