---
name: compose-page
description: "Composes a complete page from existing Laioutr sections/blocks, fills it with data directly in Laioutr Studio (via the Cockpit MCP). Delegates to the ux-designer agent. Loads memory/sections-blocks/ (catalog + composition rules) and memory/page-types/ (recipes). Focus: marketing-driven landing pages, but also home/PDP/PLP/content/blog. Triggers: '/compose-page', '/landing-page', 'build me a landing page', 'compose a page', 'which sections for the homepage', 'create a page in Laioutr', 'set up page type X'. (DE: 'bau mir eine Landing Page', 'komponier eine Seite', 'welche Sections für die Startseite', 'leg eine Seite in Laioutr an')"
---

# compose-page — compose a page from sections/blocks + fill it in Studio

Builds a page **on the Laioutr platform** from the **existing** sections/blocks, and fills it with data in Studio. Does not design new building blocks (that is `ui-designer` / `component-library-build`).

## Sources (always load first)

- `memory/sections-blocks/catalog.md` — available sections/blocks/features (source: **Storybook** `https://storybook.laioutr.cloud`)
- `memory/sections-blocks/composition-rules.md` — slots/`restrictTo`/`allow`/`prefer`/`isStandalone`, well-known tags, schema field types, `query` data binding, funnel logic
- `memory/page-types/` — recipe for the page type (landing page, home, …)
- **`taste` skill — mandatory input:** anti-AI-slop taste lens for the composition (no centered hero, no row of three identical cards, one accent/color-scheme family, realistic data instead of "Acme"/"99.99%", no filler words, think through the sections' loading/empty/error states).
- Marketing context (for landing pages): the project's brand guidelines (see `memory/design-system/corporate-design.md`, project copy under `.claude/design-manager/`; if the project has its own brand/voice guide or plugin, use it) plus the campaign brief from the user.

## Workflow

1. **Sharpen the brief (briefly).** Page type (static/dynamic?), goal/funnel stage, persona, campaign context. For landing pages: the **one** conversion action.
2. **Load the recipe** from `memory/page-types/` as the starting point.
3. **Delegate to the `ux-designer` agent:** choose + order sections (funnel logic), put blocks into slots following the rules, fill schema fields + `query` fields — **under the `taste` lens** (anti-slop in the section cut and in the data).
4. **Fill it in Laioutr Studio** (`~~platform`): the Cockpit MCP cannot create new pages or page variants (and does not publish). If the target page/variant does not exist yet, ask the user to create it in Studio (static page: slug according to `pathConstraints`), then add sections/blocks and write fields/data via the MCP. Translatable fields per language. Publishing stays with the user.
5. **Gate:** self-check (every section has a purpose, slots follow the rules, data is valid, one conversion action) **+ `taste` pre-flight** (no AI slop, realistic data, no filler words). Before going live, optionally `design-critique` (WCAG/consistency of the rendered page).
6. **Output:** composition table + link/page ID + open `_TBD_` + next step.

> **Where outputs go (mandatory):** the page itself lives in Studio. File artifacts (composition reports, screenshots, exports) go to `${DESIGN_MANAGER_OUTPUT_DIR:-./design-manager-output}/<YYYY-MM-DD>/` — never into the plugin directory and never into `.claude/design-manager/` (that folder holds curated project knowledge only).

## When not to use this skill

- **Missing section/block** (does not exist in Storybook/on the platform) → `ui-designer` / `component-library-build`.
- **New visual design / token** → `ui-designer` / `design-system-governance`.
- **Pure accessibility/consistency audit** → `design-critique`.

## Sync note

If a needed section/block is not in the catalog: re-pull Storybook `index.json` (`https://storybook.laioutr.cloud/index.json`) and check against it **before** composing — never invent building blocks. The plugin's `catalog.md` is read-only platform knowledge: if it is out of date, work from the fresh `index.json` and tell the user which entries were missing.
