---
name: Self-Management
description: "Enter self-management mode — configure providers, agents, channels, skills, generation, and other system settings"
scope: system
invocation:
  user_invocable: true
  disable_model_invocation: true
---

# Aleph Self-Management Mode

You are now in self-management mode. You have full access to read, modify,
and manage your own configuration and workspace.

## ⚠️ Critical Rules (read before doing ANYTHING)

1. **Be efficient with tool calls** — you have limited iterations. Do NOT waste calls on web_fetch, unnecessary reads, or exploratory actions. Go directly to the task.
2. **API keys are PII-filtered** — the user's message may have the API key replaced with `[REDACTED]`. If so, ask the user to send it separately or use vault_store directly.
3. **file_ops CANNOT access ~/.aleph/config.toml** — it's in denied_paths. Always use `bash` tool to read/write config.
4. **Store API key FIRST, then edit config** — vault_store, then bash edit. Never write keys to config files.
5. **Copy the exact TOML templates below** — do NOT invent field names. Wrong field names silently fail.
6. **TOML section ordering** — `[generation.providers.X]` must be BEFORE `[group_chat]`, `[tools]`, etc. Insert at the correct position.

## Workspace Structure: ~/.aleph/

```
~/.aleph/
├── config.toml              # Main config (hot-reload via fswatch)
├── soul.md                  # Global persona definition
├── user_profile.md          # User profile (loaded per session)
├── mcp_config.json          # MCP server definitions (mcp_manage to reload)
│
├── agents/{id}/             # Agent data directory
│   ├── SOUL.md              # Persona (SoulManifest, YAML frontmatter)
│   ├── IDENTITY.md          # Name, role, description
│   ├── MEMORY.md            # Long-term memory (≤20K chars)
│   ├── AGENTS.md            # Operating manual
│   ├── TOOLS.md             # Tool configuration
│   └── sessions/            # Session history
│
├── workspaces/{id}/         # Agent workspace (file output)
│   ├── output/              # Generated files
│   └── .tool_output/        # Tool temporary output
│
├── guides/                  # Config guides (read by read_config_guide tool)
├── skills/                  # User custom skills
├── skills-official/         # Official skills repo (auto-updated, read-only)
├── plugins/                 # Plugin system
├── data/                    # Persistent data (LanceDB, vault, sessions DB)
├── logs/                    # Log files
├── backups/                 # Config backups (timestamped)
└── output/                  # Global default output directory
```

## Operation Protocol

1. **Store secrets first**: vault_store for any API keys
2. **Read config**: `bash(cat ~/.aleph/config.toml)` to understand current structure
3. **Edit config**: `bash` with sed or python to insert/modify TOML sections
4. **Verify**: `bash(cat ~/.aleph/config.toml | grep -A10 'section_name')` to check
5. Config auto-reloads (fswatch 500ms) — no restart needed

## Secret Management

API keys MUST be stored in the encrypted vault. NEVER write secrets to config files.

```
vault_store(action="store", key="<convention>", secret="<api_key>")
vault_store(action="delete", key="<convention>")
vault_store(action="list")
```

| Type | Key Convention | Example |
|------|---------------|---------|
| LLM providers | `provider:{name}` | `provider:openai` |
| Generation providers | `gen:{name}` | `gen:T8StarVideo` |
| Channels | `channel:{type}:{id}` | `channel:telegram:bot1` |
| Embedding | `embedding:{name}` | `embedding:openai` |

**PII warning**: If the user sends an API key in their message, Aleph's PII filter may
replace it with `[REDACTED]` before you see it. If you see `[REDACTED]`, tell the user:
"API key was filtered by security. Please use the vault_store tool directly or send the
key in a separate message."

## Detailed Guides

For domain-specific details, call `read_config_guide(topic)`:

| Topic | Covers |
|-------|--------|
| overview | File map, operation model, all sections |
| providers | LLM provider config + vault |
| generation | Image/speech/video/audio providers |
| channels | Telegram, Discord config |
| agents | Agent workspace, SOUL.md |
| skills | Skill install, format |
| mcp | MCP server config |
| general | Default provider, language, policies |
| cron | Scheduled tasks |

## Generation Provider Configuration

### Supported provider_type values (EXACT — do not invent others)

| Category | provider_type values |
|----------|---------------------|
| Image | `openai_image`, `dalle`, `stability`, `stability_image`, `sdxl`, `midjourney`, `mj`, `google_imagen`, `imagen`, `replicate` |
| Video | `t8star_veo`, `google_veo`, `veo` |
| Speech | `openai_tts`, `tts`, `elevenlabs` |
| Audio | `openai_compat` |

### TOML field reference (EXACT field names — do not use alternatives)

| Correct field | WRONG alternatives (do NOT use) |
|---------------|--------------------------------|
| `provider_type = "t8star_veo"` | ~~provider = "..."~~, ~~type = "..."~~ |
| `models = ["model-name"]` | ~~model = "..."~~ |
| `capabilities = ["video"]` | ~~type = "video"~~ |
| `base_url = "https://..."` | ~~url = "..."~~ |

### Complete video provider template

```toml
[generation.providers.T8StarVideo]
provider_type = "t8star_veo"
base_url = "https://ai.t8star.cn/v2/videos/generations"
models = ["veo3.1-pro-4k"]
capabilities = ["video"]
enabled = true
timeout_seconds = 300
color = "#808080"
```

### Complete image provider template

```toml
[generation.providers.openai-dalle]
provider_type = "openai_image"
base_url = "https://api.openai.com/v1"
models = ["dall-e-3"]
capabilities = ["image"]
enabled = true
timeout_seconds = 60
color = "#808080"
```

### Complete speech provider template

```toml
[generation.providers.openai-tts]
provider_type = "openai_tts"
base_url = "https://api.openai.com/v1"
models = ["tts-1-hd"]
capabilities = ["speech"]
enabled = true
timeout_seconds = 60
color = "#808080"
```

### URL rules
- **Standard URL** (base only, e.g., `https://api.openai.com/v1`): system auto-appends path
- **Full URL** (e.g., `https://ai.t8star.cn/v2/videos/generations`): used as-is

### Step-by-step: Add a generation provider

Exact tool call sequence (minimize tool calls):

**Step 1**: Store API key
```
vault_store(action="store", key="gen:<ProviderName>", secret="<api_key>")
```

**Step 2**: Read current config to find insertion point
```
bash(grep -n '^\[generation\]\|^\[group_chat\]\|^\[tools\]\|^\[mcp\]' ~/.aleph/config.toml)
```

**Step 3**: Insert provider config at correct position (before next top-level section)
```
bash(python3 -c "
import re
with open('$HOME/.aleph/config.toml', 'r') as f:
    content = f.read()

new_section = '''
[generation.providers.<NAME>]
provider_type = \"<TYPE>\"
base_url = \"<URL>\"
models = [\"<MODEL>\"]
capabilities = [\"<CAPABILITY>\"]
enabled = true
timeout_seconds = 300
color = \"#808080\"
'''

# Insert before [group_chat] or [tools] or [mcp] — whichever comes first after [generation]
import re
pattern = r'(\[(?:group_chat|tools|mcp)\])'
match = re.search(pattern, content)
if match:
    pos = match.start()
    content = content[:pos] + new_section + '\n' + content[pos:]
else:
    content += new_section

with open('$HOME/.aleph/config.toml', 'w') as f:
    f.write(content)
print('Done')
")
```

**Step 4**: Set default and verify
```
bash(grep -A8 '<NAME>' ~/.aleph/config.toml)
```

Total: 4 tool calls. Do NOT add unnecessary steps.

## LLM Provider Configuration

### Complete LLM provider template

```toml
[providers.MyProvider]
protocol = "openai"
base_url = "https://api.example.com/v1"
models = ["model-name"]
enabled = true
verified = false
timeout_seconds = 300
color = "#808080"
```

- `protocol`: `"openai"` (for any OpenAI-compatible API including OpenRouter, DMX), `"anthropic"`, `"gemini"`, `"ollama"`
- Store key: `vault_store(action="store", key="provider:<name>", secret="<key>")`

## Config File Editing Rules

1. **Always use `bash` tool** — `file_ops` cannot access `~/.aleph/config.toml` (protected path)
2. **TOML section ordering matters**: subsections like `[generation.providers.X]` must appear
   after their parent `[generation]` and before the next top-level section
3. **Use python3 for complex edits** — safer than sed for TOML manipulation
4. **Verify after edit**: grep the modified section to confirm
5. **Config auto-reloads**: no restart needed (fswatch 500ms debounce)
