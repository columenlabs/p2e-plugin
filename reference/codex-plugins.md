---
title: Codex plugin reference (distilled)
hash: ref-codex
status: living
date: 2026-05-10
owner: bchoor
source: https://developers.openai.com/codex/plugins/build
---

# Codex plugin reference

Distilled from the upstream Codex build-plugins guide. Refresh when upstream changes.

## Plugin layout

```
my-plugin/
├── .codex-plugin/
│   └── plugin.json         # required
├── skills/
│   └── <skill-name>/
│       └── SKILL.md
├── .app.json               # optional, app integrations
├── .mcp.json               # optional, MCP servers
├── hooks/hooks.json        # optional
└── assets/                 # icons, images
```

Codex does NOT support Claude-style slash commands, named-agent manifests, or `PreToolUse`-style hooks. The Codex-equivalent surface is **skills** (invoked by name in plain language or via explicit alias).

## `.codex-plugin/plugin.json`

Required:

```json
{
  "name": "kebab-case",
  "version": "0.1.0",
  "description": "One-line summary"
}
```

Component pointers (paths relative to plugin root, must start with `./`):

- `skills`: e.g. `"./skills/"`
- `mcpServers`: e.g. `"./.mcp.json"`
- `apps`: e.g. `"./.app.json"`
- `hooks`: defaults to `./hooks/hooks.json`

Publisher metadata (optional): `author`, `homepage`, `repository`, `license`, `keywords`.

Interface metadata (optional, controls install surface):

- `displayName`, `shortDescription`, `longDescription`
- `developerName`, `category` (e.g. `"Coding"`)
- `capabilities`: subset of `["Interactive", "Read", "Write"]`
- `websiteURL`, `privacyPolicyURL`, `termsOfServiceURL`
- `brandColor`, `composerIcon`, `logo`, `screenshots`
- `defaultPrompt`: array of starter prompts shown to users

## Skills

Each skill is `skills/<name>/SKILL.md` with YAML frontmatter:

```yaml
---
name: skill-identifier
description: When and what this skill does.
---
```

Body is the prompt Codex executes. Since v0.12, this repo ships a single **`p2e-mode`** skill as the entry point.

## MCP servers

Same `mcpServers` schema as Claude Code. This repo keeps it at `.codex-plugin/mcp.json` (not the root `.mcp.json`) so Claude Code does not auto-load a second copy; Claude Code uses the claude.ai p2e connector.

## Discovery and invocation

- Read **`p2e-mode`** at session start for entity model, MCP surface, story lifecycle, and review pipeline.

## What this repo uses (v0.12+)

- `.codex-plugin/plugin.json` with `skills`, `mcpServers`, `interface`
- `skills/p2e-mode/SKILL.md` (sole entry point)
- `.codex-plugin/mcp.json` (Codex-only MCP config)
