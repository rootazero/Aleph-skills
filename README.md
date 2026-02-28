# AlephSkills

Official skill library for the [Aleph](https://github.com/rootazero/Aleph) project — 30 standalone skills that extend Claude Code with structured methodologies and CLI automation capabilities.

## Quick Start

Clone into your Claude Code plugins directory or reference individual skills:

```bash
git clone https://github.com/rootazero/AlephSkills.git
```

Each skill is a self-contained `SKILL.md` file that can be invoked independently.

## Skill Library

### Foundation (Core Methodology)

| Skill | Description |
|-------|-------------|
| **debug** | 7-step systematic debugging protocol with POE-aligned success contracts |
| **test** | TDD cycles, framework detection, test patterns across languages |
| **code-review** | Multi-dimensional review with confidence-based filtering (P0-P4) |
| **git** | Daily operations, advanced techniques (bisect, reflog, worktree), commit conventions |
| **refactor** | Smell detection, technique selection, test-protected incremental changes |
| **search** | Code archaeology, documentation retrieval, web research strategies |
| **shell** | System diagnostics, Docker operations, process and network management |
| **doc** | 3-stage documentation co-authoring with templates (RFC, ADR, README, runbook) |

### Workflow (Development Lifecycle)

| Skill | Description |
|-------|-------------|
| **plan** | From ambiguous requirements to clear decisions with ADR documentation |
| **api-design** | Contract-first REST/GraphQL design, error handling, versioning |
| **database** | Selection decision tree, schema design, query optimization, migrations |
| **deploy** | Containerization, release strategies, monitoring, rollback procedures |
| **security** | OWASP checklist, credential leak response, dependency scanning |
| **performance** | Measure-first profiling, optimization patterns by language |
| **ci-cd** | GitHub Actions templates, caching strategies, pipeline patterns |

### Specialist (Domain Expertise)

| Skill | Description |
|-------|-------------|
| **architecture** | Pattern selection, dependency analysis, the Aleph 1-2-3-4 model |
| **emergency** | Git disasters, disk full, DB locks, credential leaks, deployment failures |
| **regex** | Validation, parsing, replacement patterns with language-specific usage |
| **mcp-dev** | Agent-centric MCP server development, 4-phase workflow |
| **knowledge** | Insight extraction using the Rephrasing Test, N-count confidence system |

### Automation (CLI Wrappers)

| Skill | Tooling | Description |
|-------|---------|-------------|
| **github** | `gh` | PR lifecycle, issue management, CI debugging, API queries |
| **playwright** | `npx playwright` | E2E testing, screenshots, form filling, data extraction |
| **http-client** | `curl` | API debugging, authentication patterns, response analysis |
| **data-pipeline** | `jq` / `python3` | JSON/CSV transformation, filtering, aggregation |
| **email** | `curl` (SMTP/IMAP) | Send and receive emails, templates for notifications |
| **ssh** | `ssh` / `rsync` | Connections, port forwarding, file transfer, key management |
| **media-tools** | `ffmpeg` | Video/audio/image processing, format conversion |
| **notification** | `osascript` / `curl` | macOS native, ntfy.sh, Telegram, webhooks |
| **typeset** | `typst` | Document rendering to PDF, math, templates |
| **web-scraper** | `curl` / `python3` | Structured data extraction, search, PDF download |

## Skill File Format

```yaml
---
name: skill-name
description: Topic — key capabilities
scope: standalone
---

# Skill Title

## When to Use
...

## Core Content
...

## Anti-Patterns
...
```

Automation skills additionally declare `requirements.binaries`, `triggers`, and `allowed-tools` in frontmatter.

## Conventions

- **POE Framework** — Principle (define success), Operation (execute), Evaluation (verify) — maps across debug, test, plan, and other skills
- **Cross-references** — Skills reference each other by tier code: F1-F8, W1-W7, S1-S5, A1-A10
- **Git commits** — `<scope>: <description>` in imperative mood, lowercase, no period

## License

MIT
