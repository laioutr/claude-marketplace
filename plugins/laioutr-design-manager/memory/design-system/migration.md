# Design-system migration — legacy Figma → project design-system file

> Template. Project copy lives at `.claude/design-manager/design-system/migration.md`; fill it for your project.

## Purpose

Legacy Figma files often deviate from the current UI kit / Storybook. This log records every transferred, renamed or migrated delta between the legacy files and the project's design-system file (see the project copy of `memory/figma/files.md`), so the history of each component and token stays traceable.

## Governance direction

- **UI kit / Storybook wins.** Names and values in `@laioutr-core/ui-kit` / `@laioutr-core/ui-tokens` and the rendered Storybook (`https://storybook.laioutr.cloud`) lead. Figma follows.
- Legacy components are transferred into the project's design-system file, **renamed to UI-kit names** and sorted onto the correct atomic page (`atomic-structure.md`).
- **Always:** work only with **variables** (no styles), variables reference variables — `../figma/variable-workflow.md`. Theming modes: `color-scheme.md`.

## Procedure

Per run: `component-library-build` (transfer/rename/sort) or `design-system-governance` (token/variable diff) → read legacy vs. UI kit / Storybook (`use_figma`, `get_metadata`) → delta list → transfer → record here.

---

## Migration / transfer log

| Date | Component/token | Was (legacy name) | Is (UI-kit name) | Page | Why |
|---|---|---|---|---|---|
| _TBD_ | _TBD_ | _TBD_ | _TBD_ | _TBD_ | _TBD_ |

## Open deltas

Fill in: deviations that are known but not yet resolved (e.g. a legacy variant not in the UI kit's variant axis, a component deliberately not transferred and why, missing Code Connect mappings).

- _TBD_
