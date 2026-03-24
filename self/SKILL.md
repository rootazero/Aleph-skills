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
│   ├── HEARTBEAT.md         # Heartbeat state
│   └── sessions/            # Session history
│
├── workspaces/{id}/         # Agent workspace (file output)
│   ├── output/              # Generated files
│   └── .tool_output/        # Tool temporary output
│
├── guides/                  # Config guides (read by read_config_guide tool)
│   ├── overview.md          # File map, operation model, all sections
│   ├── providers.md         # LLM providers (OpenAI, Claude, Gemini, Ollama)
│   ├── generation.md        # Media generation (image/speech/video/audio)
│   ├── channels.md          # Telegram, Discord, etc.
│   ├── agents.md            # Agent workspace, SOUL.md, identity
│   ├── skills.md            # Skill install, format, discovery
│   ├── mcp.md               # MCP server configuration
│   ├── general.md           # Default provider, language, policies
│   └── cron.md              # Scheduled tasks
│
├── skills/                  # User custom skills (not git-managed by Aleph)
│   └── {name}/SKILL.md
│
├── skills-official/         # Official skills repo (git clone, auto-updated)
│   └── {name}/SKILL.md
│
├── plugins/                 # Plugin system
│   ├── installed/{name}/    # Installed plugins
│   └── cache/               # Marketplace cache
│
├── data/                    # Persistent data (LanceDB, vault, sessions DB)
├── logs/                    # Log files (aleph-server.log.YYYY-MM-DD)
├── backups/                 # Config backups (timestamped)
├── browser/                 # Headless browser profile
├── templates/               # Team templates
├── output/                  # Global default output directory
└── .venv/                   # Python virtual environment (for skills)
```

## Operation Protocol

1. **Backup first**: `bash(cp ~/.aleph/config.toml ~/.aleph/config.toml.bak)`
2. **Read current state**: Read the target file before any modification
3. **Show plan**: Tell the user what you intend to change and why
4. **Confirm**: Wait for user confirmation before writing
5. **Write**: Make the change
6. **Verify**: Read back and validate format
7. **Reload**: config.toml auto-reloads; MCP needs `mcp_manage`; channels need restart

## Secret Management (CRITICAL)

API keys and credentials MUST be stored in the encrypted vault.
NEVER write secrets to config files — the vault encrypts them at rest.

### How to store API keys

```
vault_store(action="store", key="<convention>", secret="<api_key>")
```

### How to delete API keys

```
vault_store(action="delete", key="<convention>")
```

### How to list stored keys (names only, never values)

```
vault_store(action="list")
```

### Key naming conventions

| Type | Convention | Example |
|------|-----------|---------|
| LLM providers | `provider:{name}` | `provider:openai`, `provider:claude` |
| Generation providers | `gen:{name}` | `gen:stability`, `gen:t8star-video` |
| Channels | `channel:{type}:{id}` | `channel:telegram:bot1` |
| Embedding | `embedding:{name}` | `embedding:openai` |

### Example: Add a new API key

User says: "Add my OpenAI key sk-abc123"

Steps:
1. `vault_store(action="store", key="provider:openai", secret="sk-abc123")`
2. Verify: `vault_store(action="list")` — should show `provider:openai`
3. **NEVER** write `api_key = "sk-abc123"` in config.toml

## Detailed Guides

For specific configuration domains, call read_config_guide(topic):

| Topic | Covers |
|-------|--------|
| overview | File map, operation model, all sections |
| providers | LLM providers (OpenAI, Claude, Gemini, Ollama) |
| generation | Image/speech/video/audio generation providers |
| channels | Telegram, Discord, channel-agent bindings |
| agents | Agent workspace, SOUL.md, identity, model override |
| skills | Skill install, format, discovery |
| mcp | MCP server configuration |
| general | Default provider, language, memory, policies |
| cron | Scheduled tasks |

**Always call the relevant guide before making changes** — guides contain
structure templates, field definitions, and caveats you need.

## Common Workflows

### Add a generation provider (image/video/speech/audio)

**CRITICAL**: Use the correct `provider_type` value. Supported types:
- Image: `openai_image`, `dalle`, `stability`, `stability_image`, `sdxl`, `midjourney`, `mj`, `google_imagen`, `imagen`, `replicate`
- Video: `t8star_veo`, `google_veo`, `veo`
- Speech: `openai_tts`, `tts`, `elevenlabs`
- Audio: `openai_compat`

**URL rules**:
- Standard URL (e.g., `https://api.openai.com` or `https://api.openai.com/v1`): system auto-appends path
- Full URL (e.g., `https://ai.t8star.cn/v2/videos/generations`): used as-is, no auto-completion

**Steps**:
1. read_config_guide(topic="generation") for full structure reference
2. Store API key FIRST: `vault_store(action="store", key="gen:<name>", secret="<api_key>")`
3. Add provider config to `~/.aleph/config.toml` under `[generation]` section using bash:

```bash
# IMPORTANT: Insert INSIDE the [generation] section, BEFORE the next top-level section
# Use cat with heredoc to append at correct position
```

**Video provider example** (T8Star Veo):
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

**Image provider example** (OpenAI DALL-E):
```toml
[generation.providers.openai-dalle]
provider_type = "openai_image"
base_url = "https://api.openai.com/v1"
models = ["dall-e-3"]
capabilities = ["image"]
enabled = true
```

4. Set as default: add `default_video = "T8StarVideo"` (or `default_image`, `default_speech`) in `[generation]`
5. Verify config is valid: `cat ~/.aleph/config.toml | head -5` (config auto-reloads)

**Common mistakes to avoid**:
- Do NOT use `provider = "..."` — the correct field is `provider_type = "..."`
- Do NOT use `type = "video"` — the correct field is `capabilities = ["video"]`
- Do NOT put `[generation.providers.X]` after `[tools]` or other top-level sections
- Do NOT write API keys in config — always use vault_store
- The `provider_type` must be one of the supported values listed above

### Add an LLM provider
1. read_config_guide(topic="providers") for structure
2. Add [providers.<name>] with protocol, models, enabled=true
3. vault_store(action="store", key="provider:<name>", secret="<api_key>")

### Modify agent personality
1. Read ~/.aleph/agents/{id}/SOUL.md
2. Edit content, preserve YAML frontmatter structure
3. Write back — takes effect on next agent resolution

### Install a plugin
1. aleph plugin marketplace update
2. aleph plugin install <name>

## Custom Provider Notes

Many users use third-party API proxies (DMX, OpenRouter, custom relay servers) that are
OpenAI-compatible but host various models (Claude, Gemini, Llama, etc.).

### LLM Custom Provider
```toml
[providers.MyProxy]
protocol = "openai"                    # Always "openai" for OpenAI-compatible proxies
base_url = "https://my-proxy.com/v1"   # Include /v1 if the proxy expects it
models = ["claude-sonnet-4-6", "gpt-4o"]  # Models available through the proxy
enabled = true
verified = false                       # Will be set to true after successful test
timeout_seconds = 300
```
- `protocol` is always `"openai"` for any OpenAI-compatible endpoint (including OpenRouter, DMX, etc.)
- `base_url` should be the base URL the proxy expects (some need `/v1`, some don't)
- Store API key: `vault_store(action="store", key="provider:MyProxy", secret="sk-xxx")`

### Generation Custom Provider
```toml
[generation.providers.MyVideoProvider]
provider_type = "t8star_veo"           # Must be a supported type (see list above)
base_url = "https://my-api.com/v2/videos/generations"  # Full URL = no auto-completion
models = ["model-name"]
capabilities = ["video"]               # One of: image, video, speech, audio
enabled = true
timeout_seconds = 300
```
- `provider_type` MUST be one of the supported values — there is no generic "custom" type
- For video via OpenAI-compatible proxy, use `t8star_veo` or `google_veo`
- For image via OpenAI-compatible proxy, use `openai_image`
- For speech via OpenAI-compatible proxy, use `openai_tts`
- `capabilities` determines what generation type this provider handles
- Store API key: `vault_store(action="store", key="gen:MyVideoProvider", secret="sk-xxx")`

## Config File Editing Rules

When editing `~/.aleph/config.toml`:

1. **Read the file first** to understand the current structure
2. **TOML section ordering matters**: `[generation.providers.X]` must be inside the `[generation]` block, before any other top-level section like `[tools]`, `[group_chat]`, etc.
3. **Use bash to edit**: `file_ops` cannot access `~/.aleph/config.toml` (protected). Use bash with careful sed/heredoc commands.
4. **Verify after edit**: read the file back and check TOML validity
5. **Config auto-reloads**: no restart needed for config.toml changes (fswatch 500ms debounce)
