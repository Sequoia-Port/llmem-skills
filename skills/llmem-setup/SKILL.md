---
name: llmem-setup
description: Interactive setup for LLMem persistent memory. Creates an account, generates an API key, and configures the agent environment so the llmem-memory skill can connect automatically. Run this once before using LLMem.
---

# LLMem Setup

This skill walks the user through connecting their LLMem account to this agent.

## Prerequisites

The user needs a free LLMem account at https://www.llmem.io

## Setup Steps

### 1. Check for Existing Configuration

Look for an existing API key in this order:
1. Environment variable `LLMEM_API_KEY`
2. Project file `.llmem`
3. Global config `~/.config/llmem/key`

If a key already exists, confirm with the user: "You already have a LLMem key configured. Want to replace it or keep using this one?"

### 2. Get the API Key

Tell the user:
> "To connect LLMem, I need your API key. You can find it (or create one) at:
> https://www.llmem.io/dashboard/api-keys
>
> Copy the key that starts with `llm_ak_` and paste it here."

### 3. Store the Key

Once the user provides the key, store it based on their preference:

**Option A — Global (recommended, works across all projects):**
```bash
mkdir -p ~/.config/llmem
echo "LLMEM_API_KEY={key}" > ~/.config/llmem/key
chmod 600 ~/.config/llmem/key
```

**Option B — Project-local (for team setups where each dev has their own key):**
```bash
echo "{key}" > .llmem
echo ".llmem" >> .gitignore
```

**Option C — Environment variable (for CI or advanced setups):**
Tell the user to add `export LLMEM_API_KEY={key}` to their shell profile.

### 4. Verify Connection

Test the key by running a search:
```
GET https://api.llmem.io/v1/mems/search?q=test&key={key}
```

If the response returns valid JSON (even if `memories` is empty), the connection works. Tell the user:
> "LLMem is connected. I'll automatically recall your memories at the start of every conversation and save new things you share. Your memories are encrypted end-to-end — only your key can read them."

If the response returns a 401 or 403, the key is invalid. Ask the user to double-check it.

### 5. Install the Memory Skill

If the `llmem-memory` skill isn't already installed, suggest:
```bash
npx skills add Sequoia-Port/llmem-skills --skill llmem-memory
```

## Security Notes

- Never commit API keys to git. Always add `.llmem` to `.gitignore`.
- The key file should have `600` permissions (owner-only read/write).
- Never display the full API key back to the user after storage — show only the first 10 characters followed by `...`.
