# Component inventory — REUSE before NEW

> Template. Project copy lives at `.claude/design-manager/design-system/components.md`; fill it for your project.

> Inventory of the project's design-system file, read via the Plugin API (`use_figma`). UI kit / Storybook names lead (`https://storybook.laioutr.cloud`, npm `@laioutr-core/ui-kit`). Page logic: `atomic-structure.md`. Color architecture: `color-scheme.md`.

---

## REUSE/NEW decision (heuristic)

1. **Does the component exist in the inventory below / Storybook / the UI kit (≥ 80 % fit)?** → REUSE.
2. **Only a visual deviation?** → REUSE + token override (via tokens, never hex), no new component.
3. **A genuinely new pattern?** → NEW, with rationale; record it here and in `atomic-structure.md`.

> Transfer workflow legacy UI → project design-system file: `../../skills/component-library-build/SKILL.md`. Inventory at any time via `use_figma`: `page.findAllWithCriteria({types:['COMPONENT_SET','COMPONENT']})`.

---

## Atoms

> Order = alphabetical (mirrors the canvas order on the Atoms page).

| Component | Type | Variant axes | Status | Notes |
|---|---|---|---|---|
| `<Button>` _(example)_ | COMPONENT_SET | `Variant` × `Size` × `State` | _TBD_ | binds `theme › buttons/*`; not scheme-aware (blocks pick the variant) |
| _TBD_ | _TBD_ | _TBD_ | _TBD_ | _TBD_ |

## Molecules

| Component | Type | Variant axes | Status | Notes |
|---|---|---|---|---|
| _TBD_ | _TBD_ | _TBD_ | _TBD_ | _TBD_ |

## Organisms

| Component | Type | Variant axes | Status | Notes |
|---|---|---|---|---|
| _TBD_ | _TBD_ | _TBD_ | _TBD_ | _TBD_ |

Fill in: Storybook gives the target list (Atoms → Molecules → Organisms). Status values: `present`, `transferred`, `new`, `deviates from UI kit` (explain in Notes and log it in `migration.md`).
