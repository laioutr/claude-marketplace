---
name: design-critique
description: "Pre-handoff QA gate for designs on a Laioutr project. Validates a finished Figma design (or an export) against three gates: WCAG 2.1 AA (contrast, focus, keyboard, touch target, semantics), design-system consistency (only canonical tokens/components, no detached instances/hardcodes) and buildability (every region maps to defineSection/defineBlock + Vue). NOT for building — that is the ui-designer / create-layout. Delivers severity-separated findings (Critical/Warning/Nit) with before/after. Triggers: '/design-critique', 'review the design', 'is this AA-compliant', 'design review', 'can this go to code', 'accessibility check', 'design-system consistency check'. (DE: 'prüf das Design', 'ist das AA-konform', 'kann das in Code')"
---

# design-critique — pre-handoff QA gate

Validates a finished design **before** it goes live or is handed over to code. Does not write, does not build — it checks and reports.

## Sources (always load first)

Project-copy files live under `.claude/design-manager/` in the user's project; read the project copy first, never write into the plugin directory.

- `memory/patterns/accessibility-baseline.md` — WCAG 2.1 AA checklist + token contrast matrix
- `memory/design-system/tokens.md` + `components.md` + `atomic-structure.md` (project copies) + `color-scheme.md` — what is canonical (tokens, pages, theming modes)
- `memory/principles/design-principles.md` — quality bar
- Figma: `get_design_context`, `get_screenshot`, `get_variable_defs` of the node under review

## The three gates

### Gate 1 — WCAG 2.1 AA
- **Contrast:** text 4.5:1, large text/UI components 3:1. Check against the token contrast matrix.
- **Focus:** visible focus indicator on all interactive elements.
- **Keyboard:** logical tab order, no keyboard traps.
- **Touch target:** ≥ 44×44px for interactive elements.
- **Semantics:** clean heading hierarchy (one H1, no skipped levels), labels for inputs, meaningful alt concepts.

### Gate 2 — Design-system consistency
- **Only variable-bound** properties — no hardcoded hex/pixels, **no design styles** (color/text/effect).
- Semantic variables **alias to primitives** (no raw value in a semantic variable), modes correct (theme × color scheme × light/dark).
- New/changed values live as a **variable in the library** (the project's design-system file), not on the layer.
- **Component instances**, no detached copies. Sorted onto the **right atomic page** (alphabetical + side by side, clean on canvas + in the layer list).
- Naming vocabulary followed, no duplicates of existing UI-kit components.
- **Color architecture (mandatory, frequently missed):** `color schemes` = **purely semantic** (generic `text/*` / `background/*` / `icon/*` + STRING selectors). **All color assignments in `theme`** (component-owned `theme › <comp>/<variant>/*`). A component-specific color token in `color schemes` = **Critical**. Scheme awareness only via generic semantic tokens and/or a STRING variant selector.
- **Component-owned padding/radius (mandatory):** padding (4 sides) + radius (4 corners) bound to the component's own `<comp>/*` tokens, **not** generic `Spacing/*` / radius reused on the node. Missing → Warning.
- **0 orphans:** retired/renamed tokens have 0 consumers.

### Gate 3 — Buildability
- Every region maps to an existing or deliberately new component (`defineSection`/`defineBlock`).
- Responsive behaviour defined (mobile + desktop).
- No "painterly" elements that cannot be reproduced in code.

## Output

```
## Design critique: <design/node>
**Verdict:** ✅ Ready / ⚠️ Fix before handoff / ⛔ Blocked

### Critical (stops handoff)
- [Gate] <finding> → Fix: <before/after>

### Warning (fix before handoff)
- …

### Nit (optional)
- …

### Gate status
- WCAG 2.1 AA: ✅/⚠️/⛔
- Design-system consistency: ✅/⚠️/⛔
- Buildability: ✅/⚠️/⛔
```

> **If the report is saved as a file:** `${DESIGN_MANAGER_OUTPUT_DIR:-./design-manager-output}/<YYYY-MM-DD>/design-critique-<node>.md` — never into the plugin directory and never into `.claude/design-manager/` (that folder holds curated project knowledge only).

## Rule

- **Critical findings stop the handoff.** No design with a failed AA gate or an unmapped region goes to code.
- For fixes, go back to `create-layout` / `component-library-build` / `ui-designer` — the critique does not build itself.
