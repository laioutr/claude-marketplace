# Laioutr Claude Code Marketplace

Public Claude Code / Cowork plugin marketplace for the Laioutr platform.
Plugins in this repo target **external** developers, designers, and
content authors building on Laioutr (Vue 3 / Nuxt 3 storefronts, UI
components, Orchestr data integrations).

## Install

Type these as slash commands inside a Claude Code session:

```text
/plugin marketplace add laioutr/claude-marketplace
/plugin install laioutr-developer@laioutr
/plugin install laioutr-design-manager@laioutr
```

## Plugins in this marketplace

| Plugin | Audience | What it ships |
| --- | --- | --- |
| [`laioutr-developer`](plugins/laioutr-developer/) | External frontend / full-stack developers building Vue / Nuxt apps and Orchestr integrations on Laioutr | 11 skills (platform overview, sections/blocks, entity and action tokens, Storybook, the Figma pipeline) with 67 read-on-demand reference files, plus the MCP server for `docs.laioutr.com` |
| [`laioutr-design-manager`](plugins/laioutr-design-manager/) | Designers and UX / content teams working on Laioutr projects | 7 skills, 2 agents (`ui-designer` builds components and the design system in Figma; `ux-designer` composes sections and blocks into pages in Studio), page-type recipes, the Storybook section/block catalog, design-system templates, plus MCP configs for Figma, `docs.laioutr.com` and the Laioutr Cockpit |

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