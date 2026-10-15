# Laioutr Design Manager Plugin — Working Memory

Hot cache for the `laioutr-design-manager` plugin. Load this first in every layout, Figma or design-system conversation. Anything not covered here lives in `memory/`.

## Context

Laioutr is a frontend management platform for composable/headless commerce on Nuxt 3 / Vue 3. Designs are implemented as Vue components; the design system lives in the Laioutr UI kit (`@laioutr-core/ui-tokens` / `@laioutr-core/ui-kit` / `@laioutr-core/ui`, ready-made sections and blocks in `@laioutr-app/ui`), documented in Storybook (`https://storybook.laioutr.cloud`). Each project mirrors it in its own Figma design-system file.

## What this plugin is

The **design side** of building a Laioutr project. Two clearly separated senior agents:
- **`ui-designer`** — builds **building blocks** (components, tokens, design system) **directly in Figma**, full design-system governance, end-to-end to buildable.
- **`ux-designer`** — **composes** existing sections/blocks into **pages and landing pages** and fills them with data **in Laioutr Studio**. Does not design new building blocks.

**The split is deliberate:** if the `ux-designer` is missing a building block, it does not build it — it hands off to the `ui-designer`.

## Operating profile: `ui-designer`

| Dimension | Decision |
|---|---|
| Way of working | **Builds directly in Figma** (Figma MCP `use_figma`), not just specs |
| Design philosophy | **Functional-minimalist** (clarity, whitespace, system over decoration) |
| Brief workflow | **One reasoned direction** + rationale (no spread of variants) |
| Buildability | **Strictly buildable** — `defineSection`/`defineBlock` + Vue, REUSE before NEW |
| Accessibility | **WCAG 2.1 AA** as a hard gate |
| Design-system authority | **Full governance** — the UI kit is the truth, Figma is brought in line |
| Scope | **End-to-end** — deliberately overlaps with `laioutr-developer` (Figma → code) |

## Operating profile: `ux-designer`

| Dimension | Decision |
|---|---|
| Work surface | **Laioutr Studio** (`~~platform`), not Figma |
| Does | **composes** existing sections/blocks into pages + fills them with data |
| Output | **builds the page directly in Studio and fills it** (sections, blocks, schema fields + `query`) |
| Focus | **marketing-driven landing pages** + page types (Home/PDP/PLP/Content/Blog) |
| Inventory source | **Storybook** (`storybook.laioutr.cloud`) → `memory/sections-blocks/catalog.md` |
| Boundary | designs **no** new building blocks — if something is missing, hand off to `ui-designer` |

## Single source of truth

**The UI kit leads** — names and values in `@laioutr-core/ui-kit` / `@laioutr-core/ui-tokens` and in Storybook (`https://storybook.laioutr.cloud`; docs via the `laioutr-docs` MCP at `https://docs.laioutr.com`). The project's Figma design-system file is brought in line with it. Project-specific state (tokens, components, migration deltas) is tracked in the project copy of `memory/design-system/`.

## Hard rule: variables only, no styles

In Figma use **variables only** (no color/text/effect styles). Every outcome creates or changes variables in the project's variable library; **variables reference variables** (aliasing). Layer properties are bound, never typed. Details: `memory/figma/variable-workflow.md`.

## Project memory model

The plugin directory is replaced on every plugin update — **never write into it.**

- **Platform knowledge** ships with the plugin and is read-only: `memory/sections-blocks/`, `memory/page-types/`, `memory/principles/`, `memory/patterns/`, `memory/figma/variable-workflow.md`, `memory/design-system/color-scheme.md` (architecture rules), `memory/design-system/component-descriptions.md`.
- **Project knowledge** lives in the user's project at `.claude/design-manager/`, with the same relative paths (e.g. `.claude/design-manager/design-system/tokens.md`, `.claude/design-manager/figma/files.md`). The plugin's copies are **templates**: `memory/design-system/{tokens,components,atomic-structure,migration,corporate-design,imagery}.md`, `memory/figma/files.md`, `memory/personas/README.md`.
- Read the project copy under `.claude/design-manager/` first. If it does not exist, copy the template from the plugin's `memory/` there on first use and fill it with the user. Write all updates to the project copy — never into the plugin directory.
- Where a template is still `_TBD_`, the agents flag the gap instead of inventing values.

## First run in a project

1. Create `.claude/design-manager/` in the project root.
2. Copy `memory/figma/files.md` there (`.claude/design-manager/figma/files.md`) and fill in the project's design-system file and any legacy files.
3. Copy the design-system templates (`memory/design-system/{tokens,components,atomic-structure,migration,corporate-design,imagery}.md`) to `.claude/design-manager/design-system/` and fill them — from the UI kit / Storybook and a `get_variable_defs` run against the project's design-system file.
4. Optionally copy `memory/personas/README.md` to `.claude/design-manager/personas/` and describe the project's audience.

## Directory

```
agents/ui-designer.md                    ← Senior UI designer (builds building blocks / design system in Figma)
agents/ux-designer.md                    ← Senior UX designer (composes pages in Studio)
skills/design-manager/                   ← Orchestrator
skills/create-layout/                    ← Brief → Figma build (→ ui-designer)
skills/component-library-build/          ← Atoms/molecules/organisms + legacy transfer (→ ui-designer)
skills/design-system-governance/         ← Token/component upkeep + migration + variable library
skills/design-critique/                  ← Pre-handoff QA gate (WCAG AA + design system + buildability)
skills/taste/                            ← Taste gate against AI slop (mandatory input for new components/layouts/pages — ui- AND ux-designer; adapted from Leonxlnx/taste-skill, see THIRD_PARTY_LICENSE)
skills/compose-page/                     ← Page / landing page from sections/blocks (→ ux-designer)
commands/design.md                       ← /design (Figma build)
commands/landing-page.md                 ← /landing-page (page composition)
memory/design-system/                    ← templates: tokens, components, atomic-structure, migration, corporate-design (hard UI defaults: buttons s/m, no centered body text, header + footer, use design options, CTA discipline), imagery (image concept, image suggestions, fill not fit)
                                           platform: color-scheme (theming architecture), component-descriptions (Figma description-field convention)
memory/figma/                            ← files.md (template: build target + legacy files) + variable-workflow.md (platform)
memory/sections-blocks/                  ← catalog.md (from Storybook) + composition-rules.md
memory/page-types/                       ← page-type recipes (landing-page, homepage, product-listing-page, product-listing-page-b2b, …)
memory/principles/design-principles.md   ← senior quality bar (ui-designer)
memory/principles/ux-laws.md             ← Laws of UX as composition heuristics (ux-designer: goal → laws → sections)
memory/principles/content-ux.md          ← copy/microcopy, resolving claims, secondary paths to subpages
memory/patterns/                         ← layout patterns + accessibility baseline
memory/personas/                         ← template: who the project designs for
```

## Connectors

Figma (`~~design`) + Figma Dev Mode (`figma-desktop`, variables), Laioutr Docs (`~~docs`), Laioutr Cockpit MCP (`~~platform`, Studio: add and fill sections), **Storybook** (`storybook.laioutr.cloud`, section/block inventory). See `CONNECTORS.md`.

## Cross-plugin

- **`laioutr-developer`** — Figma → code pipeline (`figma-design-analysis` → `figma-component-architecture` → `figma-to-component` → `figma-export-assets`) and `writing-section-block-definitions`. Canonical code reference during implementation.
- **Brand:** the project's brand guidelines live in the project copy of `memory/design-system/corporate-design.md`; if the project has its own brand/voice guide or plugin, use it.

## Output rule (mandatory — also for manual command runs)

File artifacts (exports, screenshots, preview HTML, critique reports) go to `${DESIGN_MANAGER_OUTPUT_DIR:-./design-manager-output}/<YYYY-MM-DD>/` — never into the plugin directory and never into `.claude/design-manager/` (that folder holds curated project knowledge only).

```bash
OUTPUT_DIR="${DESIGN_MANAGER_OUTPUT_DIR:-./design-manager-output}/$(date +%Y-%m-%d)"
mkdir -p "${OUTPUT_DIR}"
```
