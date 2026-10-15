---
name: design-system-governance
description: "Maintains the project's design system with full governance — creates tokens/components in the project's design-system file in Figma, structures the variable library logically (primitive + semantic/theme collection with theme × color scheme × light/dark), diffs against @laioutr-core/ui-tokens + Storybook, migrates the legacy Figma state to the current UI kit and records every delta in the project copy of memory/design-system/migration.md. The source of truth is ALWAYS the Laioutr UI kit/Storybook; Figma follows. Loads memory/design-system/* and memory/figma/files.md. Triggers: '/ds-sync', 'maintain the design system', 'migrate a token', 'UI kit sync', 'structure the variable library', 'set up a color scheme', 'the Figma button still has old tokens', 'add a new component to the design system', 'bring Figma up to date', 'design token audit'. (DE: 'Design-System pflegen', 'Token migrieren', 'Figma auf aktuellen Stand ziehen')"
---

# design-system-governance — maintain, structure & migrate the design system

Full governance over the design system. **The names + values in the Laioutr UI kit (`@laioutr-core/ui-kit`, `@laioutr-core/ui-tokens`) and its Storybook (`https://storybook.laioutr.cloud`) are leading; on any mismatch, UI kit/Storybook wins and Figma is pulled up to date.** Building and structuring happens in the project's design-system file.

> **Mandatory mode: variables only.** All work in Figma goes through the **variable library** (no color/text/effect styles). Variables reference variables (aliasing). Theming modes: `theme × color scheme (default/pale/solid/muted) × light/dark` (`memory/design-system/color-scheme.md`). Build target + variable library: the project's design-system file, see the project copy of `memory/figma/files.md`.

## Sources (always load first)

Project-copy files live under `.claude/design-manager/` in the user's project. Read the project copy first; if it does not exist, copy the template from the plugin's `memory/` there on first use and fill it with the user. Write all updates to the project copy — never into the plugin directory.

- `memory/design-system/tokens.md` (project copy) — canonical scales + naming vocabulary
- `memory/design-system/components.md` (project copy) — component inventory + status
- `memory/design-system/atomic-structure.md` (project copy) — atoms/molecules/organisms page structure (for sorting)
- `memory/design-system/color-scheme.md` — theming model + semantic→primitive aliasing
- `memory/design-system/migration.md` (project copy) — previous deltas legacy Figma → design-system file
- `memory/figma/files.md` (project copy) — design-system file, variable collections, Storybook, UI-kit packages
- When needed, the Laioutr docs (`~~docs`, `https://docs.laioutr.com`) for the canonical token/component definitions

> **Connector:** Reading variables completely needs the **Figma Dev Mode (desktop) MCP connector (`~~design-local`)** with the file open. The hosted MCP only returns the active page + resolved values (no mode/alias details). See `CONNECTORS.md`.

## Workflow

### A) Token/component diff & migration
1. **Probe the current Figma state:** `get_variable_defs` (+ `search_design_system`) → current Figma variables/components.
2. **Target state from UI kit/Storybook:** canonical values from the project copy of `tokens.md` (or Laioutr docs / `@laioutr-core/ui-tokens` + Storybook).
3. **Build the diff:** per token/component — do value, name, scale, mode resolution match? List the deviations.
4. **Migrate:** pull deviating Figma variables/components to the canonical values (`use_figma`, load `/figma-use` first). Component instances instead of detaching.
5. **Document:** every delta in the project copy of `memory/design-system/migration.md` (what it was, what it is now, why, date). Update the project copies of `tokens.md`/`components.md`.

### B) Structure the variable library (project design-system file)
1. **Primitive collection** (raw colors, `font size/*`, `font/family/*`, `font/weight/*`, spacing, radius) — no semantic meaning.
2. **Semantic/theme collection** with mode axes `theme × color scheme (default/pale/solid/muted) × light/dark`. Semantic variables (`text/*`, `background/*`, `buttons/<variant>/<state>/<role>-color`, type composites `<role>/<size>`) **alias to primitives**.
3. Bind **component-namespaced** spacing/radius (`buttons/size/s/*`, `TextGroup/size/s/gap`, `Blocks/Text/s/gap`) to the component sets.
4. Record the structure in the project copy of `tokens.md` (architecture rules: `color-scheme.md`).

### C) New token / new component
1. **Check the scale:** does the new value fit the existing scale (`space`, `radius`, `color/surface`, …)? If not: discuss the scale first, no one-off values.
2. **Follow the naming vocabulary:** `size`, `variant`, `state`, `tone`, … — avoid forbidden bare names (`style`, `class`, `key`, `ref`, `is`, `slot`, `slots`, `refFor`, `refKey`).
3. **Create it in the project's design-system file** (bound variable, component with properties, right atomic page), **record it in the project memory.**

## Output

```
## Design-system governance: <context>
### Diff
| Token/component | Figma (was) | UI kit/Storybook (target) | Action |
|---|---|---|---|
### Migration / structuring done
<list + Figma links>
### Memory update
<which project copies were updated: tokens.md / components.md / atomic-structure.md / migration.md>
### Open _TBD_
<values not yet mirrored / missing UI-kit reference / pale-solid-muted differences>
```

> **Where outputs go (mandatory):** token/component changes are made in the Figma design-system file, migration/delta knowledge goes into the project copy under `.claude/design-manager/` (`migration.md` etc.). File artifacts (drift/diff reports, exports, screenshots) go to `${DESIGN_MANAGER_OUTPUT_DIR:-./design-manager-output}/<YYYY-MM-DD>/` — never into the plugin directory and never into `.claude/design-manager/`.

## Guardrails

- ❌ Never put Figma above the UI kit/Storybook as source of truth.
- ❌ **No design styles** — variables only, semantically aliased to primitives.
- ❌ **Raw hex in `color schemes` or `theme`** — the only primitives live in `colors` (light/dark). `color schemes`/`theme` are alias-only; create a new color value as a `colors` primitive first, then alias it.
- ❌ No invented token values — if the UI-kit reference is missing, set `_TBD_` and flag it.
- ❌ No one-off values outside the scale.
- ✅ Every change is recorded in the project memory (curated, shared knowledge) and created/changed as a **variable in the library**.
