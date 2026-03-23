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
1. read_config_guide(topic="generation") for structure and URL rules
2. Add [generation.providers.<name>] to config.toml
   - base_url: use full URL for non-standard APIs (no auto-completion),
     or standard base URL for OpenAI-compatible APIs (system auto-appends path)
3. vault_store(action="store", key="gen:<name>", secret="<api_key>")
4. Optionally set default_<type> = "<name>" in [generation]

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
