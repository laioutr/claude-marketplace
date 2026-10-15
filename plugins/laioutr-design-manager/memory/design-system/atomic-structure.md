# Atomic design structure — Atoms / Molecules / Organisms

> Template. Project copy lives at `.claude/design-manager/design-system/atomic-structure.md`; fill it for your project.

> Page structure of the project's design-system file, modelled on the **Storybook** (`https://storybook.laioutr.cloud`). Read it via the Plugin API (`use_figma`).

---

## Pages

| Page | Content |
|---|---|
| **Atoms** | _TBD_ (alphabetical list of top-level components) |
| **Molecules** | _TBD_ |
| **Organisms** | _TBD_ |

> Storybook hierarchy (Introduction → Atoms → Molecules → Organisms) is the sorting reference. Component details live in `components.md`.

---

## Sorting & naming rules

0. **Order per page = alphabetical (mandatory).** Top-level components are sorted alphabetically by component name on every atomic page (canvas left → right). Insert every new/transferred component at the correct alphabetical position and re-order the page — the order is **always** alphabetical.
1. **Page = Storybook category.** Atom → Atoms, Molecule → Molecules, Organism → Organisms.
2. **Naming follows the UI kit / Storybook.** Axis vocabulary: `Variant`, `Size`, `State`, `color`, `icon-set` (component-property names; Title-Case such as `Variant=primary` is fine). Avoid forbidden bare field names (`style`, `class`, `key`, `ref`, `is`, `slot`, `slots`, `refFor`, `refKey`).
3. **Properties bound to tokens** (`color-scheme.md`): generic colors via `color schemes`, component colors via `theme › <component>/*`; spacing/typography via the responsive collections.
4. **REUSE before NEW** — check against the inventory (`components.md`) before creating anything new.
5. **Component-set layout + frame convention (mandatory):** the variants of a COMPONENT_SET are stacked **vertically** (never side by side, never overlapping at x=0/y=0). The COMPONENT_SET frame follows the uniform page pattern: solid fill `#E9E9E9`, stroke `visible = false`, `dashPattern = [10,5]`, `cornerRadius = 5`, `layoutMode = NONE`, `padding = 0`, `itemSpacing = 0`. Every newly created/migrated set adopts the pattern — take a screenshot before committing (all variants visible). Deviation = build error.

> Read at any time via `use_figma`: `figma.root.children` (pages) + `page.findAllWithCriteria({types:['COMPONENT_SET','COMPONENT']})`. Transfer workflow legacy UI → project design-system file: `../../skills/component-library-build/SKILL.md`.
