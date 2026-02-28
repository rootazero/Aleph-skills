# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What This Is

AlephSkills is the official skill library for the Aleph project. Each skill is a standalone `SKILL.md` file with YAML frontmatter and structured markdown content, designed to be invoked by Claude Code as a specialized capability.

## Repository Structure

Skills are organized into 4 tiers:

- **foundation/** — Core development methodology (debug, test, code-review, git, refactor, search, shell, doc)
- **workflow/** — Development lifecycle processes (plan, api-design, database, deploy, security, performance, ci-cd)
- **specialist/** — Domain-specific expertise (architecture, emergency, regex, mcp-dev, knowledge)
- **automation/** — CLI tool wrappers (github, playwright, http-client, data-pipeline, email, ssh, media-tools, notification, typeset, web-scraper)

Each skill lives at `<tier>/<skill-name>/SKILL.md`.

## Skill File Format

Every SKILL.md follows this structure:

```yaml
---
name: skill-name
description: One-line description — concise summary
scope: standalone
# Automation skills additionally have:
emoji: "..."
category: automation
cli-wrapper: true
allowed-tools: [Bash]
requirements:
  binaries: [tool-name]
  platforms: [macos, linux]
  install: [{manager: brew, package: tool-name}]
triggers: [keyword1, keyword2]
---
```

Followed by markdown with sections: "When to Use", core content, anti-patterns, and optionally "Aleph Integration" (cross-references to other skills).

## Conventions

- Skills cross-reference each other using codes: F1-F8 (foundation), W1-W7 (workflow), S1-S5 (specialist), A1-A10 (automation)
- The "POE" framework (Principle, Operation, Evaluation) maps to many skill workflows (e.g., TDD Red/Green/Refactor, Debug Define/Fix/Verify)
- Git commit format: `<scope>: <description>` — imperative mood, lowercase, no period, English only
- Aleph architectural rule: "1-2-3-4 Model" — 1 Core, 2 Faces, 3 Limbs, 4 Nerves (referenced in architecture skill)

## When Editing Skills

- Keep skills self-contained — each must work independently without requiring other skills
- Frontmatter `description` should be a dash-separated format: `Topic — key capabilities listed`
- Automation skills must declare `requirements.binaries` for any CLI tools they depend on
- Include concrete code examples, not abstract descriptions
- Every skill should have an anti-patterns section documenting common mistakes
