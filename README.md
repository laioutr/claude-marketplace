# Laioutr Claude Code Marketplace

Public Claude Code / Cowork plugin marketplace for the Laioutr platform.
Plugins in this repo target **external** developers, designers, and
content authors building on Laioutr (Vue 3 / Nuxt 3 storefronts, UI
components, Orchestr data integrations).

> **Sister marketplace.** Laioutr also maintains a private marketplace
> at `laioutr/claude-marketplace-private` for internal teams. The naming
> mirrors the npm scopes: `@laioutr/*` (public) and `@laioutr-private/*`
> (internal). Internal plugins are installed via
> `claude /plugin install <name>@laioutr-private`.

## Install

```bash
# Add the marketplace
claude /plugin marketplace add laioutr/claude-marketplace

# Install a plugin
claude /plugin install laioutr-developer@laioutr
```

## Plugins in this marketplace

| Plugin | Audience | What it ships |
| --- | --- | --- |
| [`laioutr-developer`](plugins/laioutr-developer/) | External frontend / full-stack developers building Vue / Nuxt apps and Orchestr integrations on Laioutr | 4 skills, 37 rules across `ui-kit` / `ui` / `ui-app` layers, MCP server for `docs.laioutr.com` |

Each plugin has its own `README.md` and `MAPPING.md` documenting source
material, rewrites, and exclusions.

## Repo structure

```
claude-marketplace/
├── .claude-plugin/
│   └── marketplace.json          # Marketplace manifest — lists all plugins below
├── plugins/
│   └── laioutr-developer/        # One folder per plugin
│       ├── .claude-plugin/plugin.json
│       ├── CLAUDE.md
│       ├── README.md
│       ├── MAPPING.md
│       ├── .mcp.json
│       ├── rules/
│       └── skills/
└── README.md                     # This file
```

To add a new plugin: create a new subfolder under `plugins/`, give it a
`.claude-plugin/plugin.json`, and add an entry to
`.claude-plugin/marketplace.json`.

## Versioning and releases

Each plugin owns its own version (in `plugins/<name>/.claude-plugin/plugin.json`).
The marketplace itself is unversioned — `claude /plugin marketplace add`
always pulls the latest manifest on `main`.

## Sister marketplace (internal)

The private marketplace lives at `laioutr/claude-marketplace-private`
(GitHub Org access required). Same layout as this repo. Plugins there
target internal Laioutr teams (Cockpit engineers, CLI maintainers,
platform engineering) and may reference internal-only tooling and paths.
