# Laioutr Design Manager Plugin

A Claude Code plugin for designers and teams building storefronts on the
**Laioutr** platform. It is the **design side** of a Laioutr project, with
**two clearly separated senior agents**:

- **`ui-designer`** — builds **building blocks** (components, tokens, the
  design system) **directly in Figma**, owns the design system (full
  governance) and reaches end-to-end into buildable Vue sections and blocks.
- **`ux-designer`** — **composes** existing sections and blocks into
  **pages and marketing-driven landing pages** and fills them with data in
  **Laioutr Studio**.

**The split is deliberate:** the `ui-designer` designs *building blocks*,
the `ux-designer` *builds pages from them*. If the `ux-designer` is missing
a building block, it does not build it — it hands off to the `ui-designer`.

With this plugin installed, when you ask Claude to …

- *"build a hero section in Figma"* → Claude **builds actively in Figma**
  (auto layout, bound variables, component instances), delivers one
  reasoned direction, gates on WCAG 2.1 AA and treats the Laioutr UI kit as
  the source of truth;
- *"build a landing page for our spring campaign"* → Claude **composes
  existing sections and blocks**, nests them into slots by the rules, fills
  them with data (including `query` fields) and **builds the page directly
  in Laioutr Studio**;
- … it always knows the building blocks from **Storybook**
  (`storybook.laioutr.cloud`) — no invented sections or blocks.

## Install

This plugin ships through the public Laioutr marketplace at
[`laioutr/claude-marketplace`](https://github.com/laioutr/claude-marketplace).

Open Claude Code and type these as slash commands inside the session:

```text
/plugin marketplace add laioutr/claude-marketplace
/plugin install laioutr-design-manager@laioutr
```

The first line is needed once per machine. After installing, run
`/reload-plugins` (or restart Claude Code) so the agents, skills, commands
and MCP servers pick up.

### Updating

```text
/plugin marketplace update laioutr
/plugin install laioutr-design-manager@laioutr
```

Updates replace the plugin directory. Your project knowledge is not
affected — it lives in your project under `.claude/design-manager/` (see
[Memory](#memory)).

## Requirements

- Claude Code 2.x with plugin support (`/plugin` command available).
- **Figma MCP** for the `ui-designer` — the hosted server
  (`mcp.figma.com`) and, for reading variables, the Figma **Dev Mode** MCP
  server of the desktop app (`http://127.0.0.1:3845/mcp`). Setup steps:
  [CONNECTORS.md](CONNECTORS.md).
- **Laioutr Cockpit MCP** (`mcp.laioutr.com`) for the `ux-designer` — you
  must be signed in to your Laioutr project. Without it the `ux-designer`
  can plan a composition but cannot write it into Studio.
- Outbound HTTPS to `https://docs.laioutr.com` (docs MCP) and
  `https://storybook.laioutr.cloud` (section/block inventory).
- Recommended: the
  [`laioutr-developer`](https://github.com/laioutr/claude-marketplace/tree/main/plugins/laioutr-developer)
  plugin — the canonical Figma → code pipeline the `ui-designer` hands off
  to.

## Architecture

| Layer | What | When to use |
|---|---|---|
| **Orchestrator** | `skills/design-manager` | Composite design/layout/page questions, delegation |
| **Agents** | `agents/ui-designer` (Figma / design system), `agents/ux-designer` (page composition) | Build or composition jobs |
| **Skills** | `create-layout`, `component-library-build`, `design-system-governance`, `design-critique`, `compose-page`, `taste` | Tactical: build, library, design system, QA, pages, taste gate |
| **Memory** | `design-system`, `figma`, `sections-blocks` (Storybook), `page-types`, `principles`, `patterns`, `personas` | Platform knowledge + project templates |

## Skills

| Skill | Description |
|---|---|
| `design-manager` | Orchestrator — plans, delegates, measures |
| `create-layout` | Brief → Figma build (delegates to `ui-designer`) |
| `component-library-build` | Builds atoms/molecules/organisms + transfers legacy UI (→ `ui-designer`) |
| `design-system-governance` | Token/component upkeep, migration, variable library |
| `design-critique` | Pre-handoff gate: WCAG 2.1 AA + design-system consistency + buildability |
| `compose-page` | Composes a page/landing page from sections/blocks and builds it in Laioutr Studio (→ `ux-designer`) |
| `taste` | Taste gate against generic AI-slop UI — mandatory input for new components, layouts and pages |

## Agents

| Agent | Persona | When |
|---|---|---|
| `ui-designer` | Senior UI designer — functional-minimalist, builds building blocks and the design system in Figma, full governance, end-to-end | Component / token / design-system / Figma builds |
| `ux-designer` | Senior UX designer — composes existing sections/blocks into pages and landing pages, fills them with data in Studio | Page and landing-page composition |

> **Tools:** Both agents deliberately run without a `tools` restriction in
> their frontmatter. The `ui-designer` needs the Figma MCP write tools
> (`use_figma`, …) and the Figma → code skills; the `ux-designer` needs the
> Laioutr Cockpit MCP (`~~platform`) to **write** pages. Both are active
> builders, not read-only analysts.

## Commands

| Command | Description |
|---|---|
| `/design <what to build>` | Build a layout/screen/component in Figma (`ui-designer`) |
| `/landing-page <campaign / goal>` | Compose a landing page from sections/blocks and build it in Laioutr Studio (`ux-designer`) |

## Memory

The plugin directory is replaced on every update, so nothing is ever
written into it. Memory comes in two kinds:

**Platform knowledge** — ships with the plugin, read-only:

| Memory | Content |
|---|---|
| `memory/sections-blocks/` | `catalog.md` (sections/blocks/features from **Storybook**) + `composition-rules.md` |
| `memory/page-types/` | Page-type recipes (landing page, home, product listing, B2B product listing, …) |
| `memory/principles/` | Senior quality bar, Laws of UX, content UX |
| `memory/patterns/` | Layout patterns + WCAG 2.1 AA baseline |
| `memory/figma/variable-workflow.md` | Variables only, aliasing, outcome in the library |
| `memory/design-system/color-scheme.md` | Theming architecture: theme × color scheme × light/dark |
| `memory/design-system/component-descriptions.md` | Convention for Figma component description fields |

**Project knowledge** — lives in your project at `.claude/design-manager/`
(same relative paths). The plugin ships **templates** for it:

| Template | Project copy |
|---|---|
| `memory/design-system/{tokens,components,atomic-structure,migration}.md` | `.claude/design-manager/design-system/…` — your design system's tokens, component inventory, atomic structure, migration deltas |
| `memory/design-system/{corporate-design,imagery}.md` | `.claude/design-manager/design-system/…` — your brand's hard UI defaults and image concept |
| `memory/figma/files.md` | `.claude/design-manager/figma/files.md` — your design-system file and legacy files |
| `memory/personas/README.md` | `.claude/design-manager/personas/README.md` — who your project designs for |

The agents read the project copy first. If it does not exist, they copy
the template there on first use and fill it with you. All updates go to
the project copy. Where a template is still `_TBD_`, the agents flag the
gap instead of inventing values.

## Output files

File artifacts (exports, screenshots, preview HTML, critique reports) go to
`${DESIGN_MANAGER_OUTPUT_DIR:-./design-manager-output}/<YYYY-MM-DD>/` —
never into the plugin directory and never into `.claude/design-manager/`
(that folder holds curated project knowledge only).

## MCP integrations

See [CONNECTORS.md](CONNECTORS.md) for details. `.mcp.json` registers:

- **Figma** (`~~design`) — the `ui-designer` axis: read
  (`get_design_context`, `get_variable_defs`, …) and write (`use_figma`,
  `create_new_file`). Plus `figma-desktop` (Dev Mode) for variables.
- **Laioutr Cockpit MCP** (`~~platform`, `mcp.laioutr.com`) — the
  `ux-designer` axis: build and fill pages in Studio.
- **Laioutr Docs** (`~~docs`, `docs.laioutr.com`) — platform, UI kit and
  section/block conventions.

Storybook (`storybook.laioutr.cloud`, `index.json`) is read over HTTPS as
the inventory source for sections, blocks and features.

## Layout

```
laioutr-design-manager/
├── .claude-plugin/plugin.json
├── .mcp.json                    # figma, figma-desktop, laioutr-docs, laioutr
├── CLAUDE.md                    # working memory (hot cache)
├── CONNECTORS.md
├── LICENSE                      # MIT
├── README.md
├── agents/                      # ui-designer, ux-designer
├── commands/                    # /design, /landing-page
├── memory/                      # platform knowledge + project templates
└── skills/                      # design-manager, create-layout, component-library-build,
                                 # design-system-governance, design-critique, compose-page, taste
```

## License

MIT — see [`LICENSE`](./LICENSE).

The `taste` skill is adapted from
[Leonxlnx/taste-skill](https://github.com/Leonxlnx/taste-skill) (MIT) — see
[`skills/taste/THIRD_PARTY_LICENSE`](./skills/taste/THIRD_PARTY_LICENSE).
