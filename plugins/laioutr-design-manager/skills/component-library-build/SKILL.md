---
name: component-library-build
description: "Builds the component library of the project's design-system file in Figma — atoms/molecules/organisms modelled on the Laioutr Storybook (https://storybook.laioutr.cloud). Compares the Laioutr UI kit (`@laioutr-core/ui-kit`, names/props) and Storybook with the project's legacy Figma files, transfers the legacy components into the design-system file, renames them to the UI-kit/Storybook names where needed and sorts them onto the right atomic page. Structures the variable library logically and applies the color-scheme logic along the way. Delegates the heavy generation to the ui-designer agent. Triggers: '/build-components', 'create atoms', 'build molecules/organisms', 'transfer a component from the old UI', 'create a Storybook component in Figma', 'build the component library', 'migrate the old UI into the design system', 'structure the variable library'. (DE: 'Atoms anlegen', 'Component aus dem alten UI übertragen', 'Component-Library aufbauen', 'Variablen-Library strukturieren')"
---

# component-library-build — build the component library of the project's design-system file

Transfers the component knowledge from **the project's legacy Figma files + the Laioutr UI kit + Storybook** into **the project's design-system file** (listed in the project copy of `memory/figma/files.md`) — as clean atoms/molecules/organisms, with a logically structured variable library and the color-scheme logic. Senior UI designer quality, **variables only** (no styles), strictly buildable.

## Sources (always load first)

Project-copy files live under `.claude/design-manager/` in the user's project. Read the project copy first; if it does not exist, copy the template from the plugin's `memory/` there on first use and fill it with the user. Write all updates to the project copy — never into the plugin directory.

- `memory/figma/files.md` (project copy) — build target (the project's design-system file), references, legacy files, Storybook, UI-kit packages
- `memory/design-system/atomic-structure.md` (project copy) — page structure (atoms/molecules/organisms) + sorting/naming rules + probed inventory
- `memory/design-system/color-scheme.md` — theming model (theme × color scheme × light/dark), semantic→primitive aliasing, naming
- `memory/design-system/components.md` (project copy) — component inventory + status (REUSE before NEW)
- `memory/design-system/tokens.md` (project copy) — token scales + naming vocabulary
- `memory/design-system/migration.md` (project copy) — transfer/rename log (written back here)
- `memory/figma/variable-workflow.md` — **mandatory**: variables only, aliasing, no styles
- **`taste` skill — mandatory input for NEW components:** anti-AI-slop taste gate (dials, color calibration, anti-card-overuse, type discipline, interaction states), mapped to the Laioutr stack. Applies as soon as a new component is designed (not for pure REUSE/TRANSFER).

## Three comparison sources (truth hierarchy)

1. **Laioutr UI kit** (`@laioutr-core/ui-kit`, plus `@laioutr-core/ui` / `@laioutr-core/ui-tokens`) — **names + props/values are leading.** Read from the installed npm packages or via the `laioutr-docs` MCP (`https://docs.laioutr.com`).
2. **Storybook** (https://storybook.laioutr.cloud) — rendered truth for **structure + categorisation** (atoms/molecules/organisms) and naming. `index.json` lists all stories.
3. **The project's legacy Figma files** (listed in the project copy of `memory/figma/files.md`) — the **existing stock that gets transferred** (visuals/components), but with legacy names → **rename** to UI-kit/Storybook names.

> On conflict: UI kit/Storybook wins over legacy Figma. No value/name from the legacy UI is adopted unchecked.

## Workflow

1. **Sharpen the scope (one round max).** Which level/components? (e.g. "all atoms", "form-field molecule", "header organism"). Otherwise make an assumption and name it.
2. **Probe the current state of the design-system file.** `get_metadata` (page list), `get_variable_defs`/`search_design_system` → what already exists, no duplicates. **For complete pages/variables use the Figma Dev Mode (desktop) MCP connector (`~~design-local`) with the file open** (the hosted MCP only shows one page).
3. **Three-source diff.** Per target component: UI-kit name/props ↔ Storybook category/variants ↔ legacy Figma stock. Result: REUSE (exists in the design-system file) / TRANSFER (transfer from legacy UI + rename) / NEW (with rationale).
4. **Structure the variable library (before/in parallel with building components).** Primitive collection + semantic/theme collection with modes `theme × color scheme (default/pale/solid/muted) × light/dark`. Semantics alias to primitives (`color-scheme.md`). **Mandatory per component:**
   - **Color assignments ONLY in `theme`; `color schemes` stays purely semantic.** Component colors as component-owned `theme › <comp>/<variant>/<prop>` (alias → `colors`, e.g. `theme › badge/<variant>/background`) — **never** a component-specific color token in `color schemes`. Scheme awareness reaches the component via (a) generic semantic `color schemes › text/*` / `background/*` / `icon/*` (which switch per scheme + light/dark) and/or (b) a STRING variant selector `color schemes › <comp>/default` → `theme › color scheme/<scheme>/<comp>/default`. No re-coloring per prop, no per-scheme color tokens per component.
   - **Component-owned padding + radius:** create and bind `theme › <comp>/padding-{left,right,top,bottom}` + `<comp>/border-radius` (alias → spacing/radius primitives) — do **not** reuse generic `Spacing/*` / radius directly on the node (e.g. `theme › badge/padding-left`).
5. **Delegate to the `ui-designer` agent (build).** Create the component set per level on the **right page** (atoms/molecules/organisms), axes with the canonical vocabulary (`Variant`, `Size`, `State`, `icon-set`, …), properties **bound to semantic variables**, component instances instead of detaching, at least the variants shown in Storybook. **For NEW components, apply the `taste` gate** (color calibration, anti-card-overuse, states loading/empty/error/active).
6. **Rename + sort.** Rename transferred legacy components to UI-kit/Storybook names and place them on the matching atomic page. Log every rename delta in `migration.md` (was → is, page, why, date).
7. **Gate: `design-critique`.** WCAG 2.1 AA + design-system consistency (variables only, aliasing, instances) + buildability. On ⚠️, back to the agent.
8. **Memory update (project copies).** `components.md` (inventory/status), `atomic-structure.md` (new pages/components), `tokens.md` (new variables), `migration.md` (transfer log).

## Mandatory self-check before the gate (read-only, brief — numbers only, no prose)

Before step 7, probe read-only and confirm exactly these points — if one is missing, back to the agent:

- [ ] **Color architecture:** `color schemes` contains **0** new component color tokens (only generic semantics + STRING selectors); all component colors bind `theme › <comp>/*` OR generic `color schemes › text/*` etc. All chains resolve for the relevant schemes × light/dark (0 `unresolved`).
- [ ] **Own padding/radius:** padding (4 sides) + radius (4 corners) bound to the component's own `<comp>/*` tokens, no generic `Spacing/*` / radius reuse on the node.
- [ ] **Type:** `fontFamily`/`fontStyle`/`fontSize` of every text node bound; **icon instance:** `icon-set` + `size` bound.
- [ ] **0 hardcodes, 0 unbound props, 0 orphans** (retired tokens have 0 consumers), alphabetical + side-by-side arrangement correct (`atomic-structure.md`).
- [ ] **WCAG 2.1 AA** per relevant scheme/appearance on the frame (otherwise flag `_TBD_`).

## Output

```
## Component library build: <level/scope>
**Figma:** <link to the page / the built component sets>
**Source diff:** UI kit ↔ Storybook ↔ legacy UI

### Action map
| Target component | Atomic page | Source (legacy UI / UI kit / Storybook) | REUSE/TRANSFER/NEW | Rename (was → is) | Tokens/variables |
|---|---|---|---|---|---|

### Variable library
<created/structured collections + modes + aliases>

### Gates
- WCAG 2.1 AA: ✅/⚠️
- Design-system consistency (variables only, aliasing, instances): ✅/⚠️
- Buildability: ✅/⚠️

### Memory update
<components.md / atomic-structure.md / tokens.md / migration.md (project copies)>

### Open _TBD_
<unreachable pages, missing UI-kit/Storybook values, pale/solid/muted differences>
```

> **Where outputs go (mandatory):** components/variables are created in the Figma design-system file, inventory knowledge goes into the project copy under `.claude/design-manager/`. File artifacts (exports, screenshots, diff reports) go to `${DESIGN_MANAGER_OUTPUT_DIR:-./design-manager-output}/<YYYY-MM-DD>/` — never into the plugin directory and never into `.claude/design-manager/`.

## Guardrails

- ❌ Raw hex in `color schemes` or `theme` — **the only primitives live in `colors`** (light/dark); `color schemes`/`theme` are alias-only. New color value: first a primitive in `colors`, then alias it. (`memory/design-system/color-scheme.md`)
- ❌ Do not keep legacy Figma names where the UI kit/Storybook has a canonical name — **rename**.
- ❌ No design styles, no hardcoded hex/pixels — only variable-bound properties, semantically aliased to primitives.
- ❌ Component on the wrong atomic page (atom on the organisms page, etc.).
- ❌ Inventing values/names when the UI kit/Storybook is unreachable — set `_TBD_`, flag it, close it via `design-system-governance` + the Figma Dev Mode (desktop) connector.
- ✅ REUSE before TRANSFER before NEW. Every action is recorded in the project memory.

## When not to use this skill

- New **layout/screen** (no library build) → `create-layout`.
- Pure **token/variable diff & migration** without component transfer → `design-system-governance`.
- Pure **audit** → `design-critique`.
- **Code implementation** → `laioutr-developer` plugin (`figma-design-analysis` → `figma-component-architecture` → `figma-to-component`).
