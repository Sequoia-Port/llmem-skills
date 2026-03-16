# LLMem Skills

Persistent, encrypted memory for AI agents. Works across every model, every conversation, every agent.

```bash
npx skills add Sequoia-Port/llmem-skills
```

## What is LLMem?

LLMem gives your AI agent a memory that persists. When you tell Claude your name, Cursor remembers it tomorrow. When you tell Grok your coding preferences, Copilot already knows them.

Every memory is encrypted end-to-end with a key derived from your API key. Not even the LLMem service can read your data.

**Get your free API key at [llmem.io](https://www.llmem.io)**

## Skills

| Skill | Description |
|---|---|
| **llmem-memory** | Core memory skill. Teaches any agent the Search, Respond, Save loop. Install this one. |
| **llmem-setup** | One-time setup. Walks you through connecting your API key. |
| **llmem-mcp** | MCP server integration for agents that support it (Claude Code, Cursor, etc). |

## Quick Start

1. Sign up at [llmem.io](https://www.llmem.io) and grab your API key
2. Install the skills:
   ```bash
   npx skills add Sequoia-Port/llmem-skills
   ```
3. Set your key:
   ```bash
   export LLMEM_API_KEY="llm_ak_your_key_here"
   ```
4. Start talking. Your agent will search your memories before responding and save new facts automatically.

## How It Works

On every turn, your agent:

1. **Searches** your memory store for context relevant to the conversation
2. **Responds** using what it found — referencing your preferences, history, and projects naturally
3. **Saves** anything new you shared — preferences, decisions, corrections, context

Memories are organized by namespace (identity, preferences, projects, etc.) and tier (core, long-term, short-term). The system tracks how you change over time and surfaces contradictions when your thinking evolves.

## Privacy

LLMem uses client-side key derivation. Your API key generates an encryption key (KEK) that encrypts all memory values before they're stored. The server never sees plaintext memories and never has access to your KEK. If you delete your key, your memories become unrecoverable ciphertext.

This isn't a feature bolted on after launch. Encryption is the architecture.

## Supported Agents

Works with every agent on [skills.sh](https://skills.sh) including Claude Code, Cursor, Codex, Copilot, Windsurf, Gemini CLI, Goose, Roo, Cline, and 30+ more.

## License

MIT
