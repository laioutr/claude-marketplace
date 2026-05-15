# Laioutr Developer Plugin

Claude Code / Cowork plugin for **external developers** building on the
Laioutr platform — Vue 3 / Nuxt 3 storefronts, UI components, and Orchestr
data integrations.

The plugin ships the same skills and rules the Laioutr core team uses
internally, with monorepo-internal pieces stripped out. See `MAPPING.md`
for what was kept and what was excluded.

## Install

Locally during development:

```bash
claude plugin install /path/to/claude-plugin-developer
```

Or as a packaged `.plugin` file:

```bash
cd /path/to/claude-plugin-developer \
  && zip -r /tmp/laioutr-developer.plugin . -x "*.DS_Store" "*.git/*"
```

## What's inside

### Skills (4)

Skills load automatically when Claude detects a matching task.

| Skill | When it triggers |
| --- | --- |
| `adr` | Making architectural decisions, capturing design rationale |
| `component-architecture` | Producing component API specs from a Figma analysis |
| `figma-design-analysis` | Decomposing complex Figma layouts into component hierarchies |
| `figma-to-component` | Implementing Vue components from Figma designs |

### Rules (37)

Read-on-demand rules organized in two tiers:

- **Top-level** (13 rules) — cross-cutting conventions that apply across
  all three component layers: naming, CSS, TypeScript, Vue SFC
  structure, Storybook stories, etc.
- **Layer-specific** (24 rules) in `rules/ui-kit/`, `rules/ui/`,
  `rules/ui-app/` — fine-grained patterns scoped to each layer:

| Layer | Count | Examples |
| --- | --- | --- |
| `ui-kit/` | 13 + CLAUDE.md | Component structure, reka-ui patterns and type-leakage, no-outer-chrome, theming, Storybook Atomic Design categorization |
| `ui/` | 5 + CLAUDE.md | Package role, ui-kit-first workflow, viewport stories, z-ordering, Storybook role-based categorization |
| `ui-app/` | 7 + CLAUDE.md | Section/Block config standard, schema `if` conditions, block naming, shared-field factories & options, forbidden field names |

Each layer folder includes a `CLAUDE.md` summarizing the layer's role
and pointing at its rules.

### Top-level guidance

`CLAUDE.md` gives Claude project-level context (platform overview,
Orchestr handler types, commit conventions, refactor discipline).

### MCP server — Laioutr developer docs

The plugin registers a `laioutr-docs` MCP server (configured in
`.mcp.json`) that points at `https://docs.laioutr.com/mcp`. Once Claude
connects, it can search the official Laioutr developer documentation
(Orchestr framework, data model, CLI commands, UI components, sections
and blocks, TypeScript API reference) directly during a session.

If your team self-hosts the docs at a different URL or behind auth,
adjust `.mcp.json` accordingly — the file is a standard MCP server
config with `type: "http"` and `url`.

## Structure

```
claude-plugin-developer/
├── .claude-plugin/
│   └── plugin.json
├── CLAUDE.md                 # Platform-level guidance for Claude
├── MAPPING.md                # What was carried over from the monorepo, and why
├── README.md
├── rules/                    # 13 cross-cutting rules + 3 layer subfolders
│   ├── ui-kit/               # 13 rules + CLAUDE.md (atomic primitives)
│   ├── ui/                   # 5 rules + CLAUDE.md (commerce organisms)
│   └── ui-app/               # 7 rules + CLAUDE.md (sections / blocks)
├── skills/                   # 4 skills (each in its own folder)
└── .mcp.json                 # MCP server config (Laioutr developer docs)
```

## Validation

```bash
claude plugin validate ./.claude-plugin/plugin.json
```

## Updating from the monorepo

This plugin is a curated mirror of `laioutr/.claude/`. When the core team
updates a skill or rule, re-run the copy step (see `MAPPING.md` for the
exact include/exclude list).
