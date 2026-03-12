# AlephSkills

Official skill library for the [Aleph](https://github.com/rootazero/Aleph) project — 30 standalone skills that extend Claude Code with structured methodologies and CLI automation capabilities.

## Quick Start

Clone into your Claude Code plugins directory or reference individual skills:

```bash
git clone https://github.com/rootazero/AlephSkills.git
```

Each skill is a self-contained `SKILL.md` file that can be invoked independently.

## Skill Library

| Skill | Description |
|-------|-------------|
| **api-design** | Contract-first REST/GraphQL design, error handling, versioning |
| **architecture** | Pattern selection, dependency analysis, the Aleph 1-2-3-4 model |
| **ci-cd** | GitHub Actions templates, caching strategies, pipeline patterns |
| **code-review** | Multi-dimensional review with confidence-based filtering (P0-P4) |
| **data-pipeline** | JSON/CSV transformation, filtering, aggregation (`jq` / `python3`) |
| **database** | Selection decision tree, schema design, query optimization, migrations |
| **debug** | 7-step systematic debugging protocol with POE-aligned success contracts |
| **deploy** | Containerization, release strategies, monitoring, rollback procedures |
| **doc** | 3-stage documentation co-authoring with templates (RFC, ADR, README, runbook) |
| **email** | Send and receive emails, templates for notifications (`curl` SMTP/IMAP) |
| **emergency** | Git disasters, disk full, DB locks, credential leaks, deployment failures |
| **git** | Daily operations, advanced techniques (bisect, reflog, worktree), commit conventions |
| **github** | PR lifecycle, issue management, CI debugging, API queries (`gh`) |
| **http-client** | API debugging, authentication patterns, response analysis (`curl`) |
| **knowledge** | Insight extraction using the Rephrasing Test, N-count confidence system |
| **mcp-dev** | Agent-centric MCP server development, 4-phase workflow |
| **media-tools** | Video/audio/image processing, format conversion (`ffmpeg`) |
| **notification** | macOS native, ntfy.sh, Telegram, webhooks (`osascript` / `curl`) |
| **performance** | Measure-first profiling, optimization patterns by language |
| **plan** | From ambiguous requirements to clear decisions with ADR documentation |
| **playwright** | E2E testing, screenshots, form filling, data extraction (`npx playwright`) |
| **refactor** | Smell detection, technique selection, test-protected incremental changes |
| **regex** | Validation, parsing, replacement patterns with language-specific usage |
| **search** | Code archaeology, documentation retrieval, web research strategies |
| **security** | OWASP checklist, credential leak response, dependency scanning |
| **shell** | System diagnostics, Docker operations, process and network management |
| **ssh** | Connections, port forwarding, file transfer, key management (`ssh` / `rsync`) |
| **test** | TDD cycles, framework detection, test patterns across languages |
| **typeset** | Document rendering to PDF, math, templates (`typst`) |
| **web-scraper** | Structured data extraction, search, PDF download (`curl` / `python3`) |

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
