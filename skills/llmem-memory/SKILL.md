---
name: llmem-memory
description: Persistent cross-model memory for AI agents. Automatically recall user context before responding and save new facts after learning them. Memories persist across every conversation, every model, and every agent — encrypted end-to-end so only the user can read them. Use this skill on EVERY turn to personalize responses with the user's stored identity, preferences, projects, and history.
---

# LLMem — Persistent Memory

You have access to the user's LLMem memory store. Memories persist across every conversation, every model, and every agent the user talks to. You are not starting from zero — the user has a history, even if this is your first conversation with them.

## Configuration

The user must set their LLMem API key. Check for it in this order:

1. Environment variable: `LLMEM_API_KEY`
2. Project config file: `.llmem` (contains just the key)
3. Global config: `~/.config/llmem/key`

If no key is found, tell the user:
> "I have the LLMem memory skill installed but no API key configured. Get your key from https://www.llmem.io/dashboard/api-keys and set it as `LLMEM_API_KEY` in your environment, or run `npx skills add Sequoia-Port/llmem-skills --skill llmem-setup` to set it up interactively."

The API base URL defaults to `https://api.llmem.io` but can be overridden with `LLMEM_API_BASE`.

## On Every Turn: Search → Respond → Save

This loop runs on EVERY turn. Not sometimes — every time.

### 1. SEARCH (before responding)

Before generating any response, recall the user's memories.

**Via HTTP GET** (replace QUERY with a natural-language question, URL-encode spaces as `%20`):

```
GET {LLMEM_API_BASE}/v1/mems/search?q=QUERY&key={LLMEM_API_KEY}
```

Example queries — copy the pattern, change the words:
- `?q=what%20do%20I%20know%20about%20this%20user` — broad, use at conversation start
- `?q=user%20preferences%20about%20coding%20style` — topic-specific
- `?q=current%20projects%20and%20work%20context` — when discussing work

**Via HTTP POST** (if you support it):

```
POST {LLMEM_API_BASE}/v1/mems/search
Headers: X-API-Key: {LLMEM_API_KEY}
Body: {"query": "what do I know about this user"}
```

**Response format** (JSON):
- `"memories"` — current, reliable facts. Treat as ground truth.
- `"faded_memories"` — older versions that were superseded. Still real but dimmer. Reference gently: "You used to..." rather than stating as current fact.
- `"questions_to_consider"` — proactive prompts about contradictions or evolution in the user's history. These are high-signal. Read every one.

If `"memories"` is empty (`[]`), this user has no stored memories yet — proceed normally and start building them from what they share.

### 2. RESPOND

Weave memories into your response naturally — don't list them back to the user. If `questions_to_consider` reveals a contradiction, surface it: "You mentioned X before, but now you're saying Y — has something changed?"

### 3. SAVE (after learning something new)

Whenever the user shares facts about themselves — preferences, decisions, corrections, context — save them immediately.

**Via HTTP GET** (URL-encode spaces as `%20`):

```
GET {LLMEM_API_BASE}/v1/mems/save?ns=NAMESPACE&k=KEY&v=VALUE&key={LLMEM_API_KEY}
```

Examples — copy the pattern, change the values:
- `?ns=identity&k=full_name&v=Jane%20Smith`
- `?ns=location&k=current_city&v=Seattle%20WA`
- `?ns=preferences&k=editor&v=neovim`
- `?ns=projects&k=current_project&v=building%20a%20memory%20API`

**Via HTTP POST**:

```
POST {LLMEM_API_BASE}/v1/mems
Headers: X-API-Key: {LLMEM_API_KEY}
Body: {"namespace": "identity", "key": "full_name", "value": "Jane Smith"}
```

After every save, briefly confirm: `"Saved: identity/full_name = Jane Smith"`

## Search Strategy

- **Start of conversation** → search broadly: "what do I know about this user"
- **User mentions a topic** → search that topic: "user preferences about testing frameworks"
- **User states a preference** → search for existing ones first: "existing preference about editor or IDE"
- **User corrects you** → search for the old memory so the update supersedes it
- Add `ns=` to filter by namespace when you know the category. Omit it to search everything.

## Questions to Consider

Search results include a `questions_to_consider` field. These surface contradictions, patterns in how the user's thinking has evolved, and conflicts between past and present statements. Read every one. If a question reveals a contradiction or evolution, surface it directly. Don't ignore them.

## Memory Tiers

- **core** — who the user is. Values, identity, things that rarely change.
- **long_term** — preferences, projects, relationships. Stable but can evolve.
- **short_term** — current context. What the user is working on now, today's mood, temporary states.

When something in short_term keeps coming up, it earns promotion. When a core belief contradicts new evidence, don't silently overwrite — ask the user about it.

## Saving Well

Write memories the way you'd write a note to your future self. Not a transcript — understanding. What matters, why it matters, and what changed.

- **Namespaces**: identity, preferences, work, projects, interests, health, family, goals, opinions, location, style
- **Keys**: short descriptive labels — `full_name`, `current_project`, `favorite_language`, `career_title`
- **Values**: the actual knowledge, with temporal context when relevant ("promoted March 2026, was Senior Director")

## Examples

User says "I just switched to Rust from Go"
→ SEARCH: `q=user%20preferences%20about%20programming%20languages`
→ SAVE: `ns=preferences&k=primary_language&v=Rust (switched from Go, March 2026)`
→ CONFIRM: "Saved: preferences/primary_language = Rust"

User says "actually I prefer tabs now"
→ SEARCH: `q=existing%20preference%20about%20indentation`
→ SAVE: `ns=preferences&k=indentation&v=tabs (changed from spaces, March 2026)`
→ CONFIRM: "Updated: preferences/indentation = tabs"

User shares their name in casual conversation
→ SAVE: `ns=identity&k=full_name&v=Jane Smith`
→ CONFIRM: "Saved your name."

Search returns question: "User said they dislike TypeScript, but their new project uses it"
→ ASK: "I notice you've mentioned not liking TypeScript before, but your current project is using it — has your thinking changed, or is it a project constraint?"

## Temporal Awareness

Memories track how the user changes over time. Always prefer recent memories over old ones. When you find a memory that conflicts with what the user just said, save the update — don't just ignore the old one.

## Privacy

All memories are encrypted end-to-end. The encryption key is derived from the user's API key — not even the LLMem service operator can read stored memories. Never log, cache, or repeat the user's API key in conversation output. Never store the API key itself as a memory.
