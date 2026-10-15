---
name: create-layout
description: "Creates a new UI layout/screen/component directly in Figma from a brief. Delegates the heavy generation to the ui-designer agent — functional-minimalist, one reasoned direction, strictly buildable (defineSection/defineBlock + Vue), WCAG 2.1 AA. Loads memory/design-system/, memory/figma/, memory/patterns/ and the project's brand guidelines. Runs design-critique after the build as the pre-handoff gate. Triggers: '/design', 'build me a hero section', 'make a layout for X', 'Figma screen for Y', 'landing page design', 'component design in Figma'. (DE: 'bau mir eine Hero-Section', 'mach ein Layout für X', 'Figma-Screen für Y')"
---

# create-layout — build a layout/screen/component in Figma

Creates a new design **directly in Figma**, at senior UI designer quality, buildable from the start.

## Sources (always load first)

Project-copy files live under `.claude/design-manager/` in the user's project. Read the project copy first; if it does not exist, copy the template from the plugin's `memory/` there on first use and fill it with the user. Never write into the plugin directory.

- `memory/design-system/tokens.md` + `components.md` (project copies) — canonical tokens + component inventory (REUSE before NEW)
- `memory/design-system/atomic-structure.md` (project copy) + `color-scheme.md` — where components live + how the theming variables resolve
- `memory/figma/files.md` (project copy) — which file/page to build into (build target: the project's design-system file), design-system reference
- `memory/patterns/layout-patterns.md` — canonical solution for the layout type
- `memory/patterns/accessibility-baseline.md` — WCAG 2.1 AA checklist + contrast matrix
- `memory/principles/design-principles.md` — decision heuristics
- **`taste` skill — mandatory input:** anti-AI-slop taste gate (dials, no AI purple / centered hero / row of three cards, interaction states), mapped to the Laioutr stack. Load before building.
- The project's brand guidelines — see `memory/design-system/corporate-design.md` (project copy); if the project has its own brand/voice guide or plugin, use it

## Workflow

1. **Sharpen the brief (one round max).** Only ask if necessary: purpose/funnel stage, persona, breakpoint focus, hard constraints. Otherwise make assumptions and name them.
2. **Probe the current Figma state.** `get_variable_defs` + `search_design_system` → build on canonical tokens/components, no duplicates.
3. **Delegate to the `ui-designer` agent.** The agent composes buildably (region → UI-kit component → `defineSection`/`defineBlock`), builds in Figma (auto layout, bound variables, component instances, at least mobile + desktop), **applies the `taste` gate** (dials, anti-slop, states) and delivers **one reasoned direction**.
4. **Gate: `design-critique`.** WCAG 2.1 AA + design-system consistency + buildability. On ⚠️, back to the agent.
5. **Output + next step.** Figma link, rationale, buildable map, gate status, open `_TBD_`. Optional end-to-end handoff to `laioutr-developer`.

## Output

The structured format of the `ui-designer` agent (Design / Rationale / Buildable map / Gates / Open points / Next step). File artifacts (exports, screenshots, preview HTML, critique reports) go to `${DESIGN_MANAGER_OUTPUT_DIR:-./design-manager-output}/<YYYY-MM-DD>/` — never into the plugin directory and never into `.claude/design-manager/` (that folder holds curated project knowledge only).

## When not to use this skill

- **Building the component library / transferring legacy UI** (atoms/molecules/organisms) → `component-library-build`.
- Pure **token/component migration** without a new layout → `design-system-governance`.
- Pure **audit** of an existing design → `design-critique`.
- **Code implementation** → `laioutr-developer` plugin.
