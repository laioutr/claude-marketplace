# Unique Component Names Across ui Layers

Every Vue component file you author in your Nuxt module that maps to one of the three Laioutr UI layers — ui-kit (atoms), ui (commerce organisms), ui-app (sections / blocks) — must have a **globally unique basename** across all three layers in your module.

No two components may share the same filename stem — not within one layer, not across layers.

## The rule

For any new or renamed component `<Name>.vue` in your module that plays one of:

- a ui-kit-layer role (atomic primitive, e.g. `src/runtime/components/`)
- a ui-layer role (commerce organism)
- a ui-app-layer role (typically `src/runtime/app/{block,section,connected-components,components}/`)

the stem `<Name>` must not already exist in any of the three layer scopes.

The `Block*` / `Section*` / `Connected*` / `UiApp*` prefixes used in ui-app filenames count as part of the name — `BlockCategoryCard` does not collide with `CategoryCard`. The rule is about **bare filename stems** only.

## Why

1. **Auto-import hygiene.** Nuxt auto-import and template-based component resolution hit the first matching component name. Two components named `Text` will silently resolve to whichever module registered last, producing non-deterministic renders.
2. **Grep / LSP correctness.** Refactors that search for a component by name return both results and force the author to disambiguate every call site manually. Unique names make `findReferences` and grep authoritative.
3. **Cognitive load.** A reviewer reading `<CategoryCard />` in a template should not have to trace imports to know whether it's the ui-kit primitive or the ui composition. One name, one component.
4. **Storybook titles.** Duplicate stems end up under different Storybook categories with the same display name, breaking Chromatic snapshot keys and confusing designers.

## What to do when a collision would be introduced

Before creating or renaming a component:

1. Search for the stem across all UI-layer directories in your module:

   ```
   Glob src/runtime/**/<Name>.vue
   ```

   If you consume `@laioutr-core/ui-kit` and `@laioutr-core/ui`, also check whether the name collides with an upstream component you would shadow via your module's auto-import order — a colliding name silently overrides the upstream one.

2. If any match exists, pick a different name that reflects the component's role:
   - **ui-kit-layer** is the design-system primitive. Its name should be the generic, Figma-style noun (`Button`, `Pagination`, `Text`).
   - **ui-layer** compositions should carry domain intent (`ListingPagination`, `CategoryTile`) when the ui-kit name is taken.
   - **ui-app-layer** Block/Section/Connected files already carry their own prefix and rarely collide — but verify the suffix stem too (`BlockCategoryCard` → watch for `CategoryCard` elsewhere).

3. Do not resolve a collision by adding a `*Block` or `*Section` suffix in the ui or ui-kit layers — those suffixes belong to the ui-app filename convention and using them elsewhere confuses readers.

## Resolving an existing collision

If an existing duplicate is discovered in your module:

1. Use LSP `findReferences` (or grep fallback for `.vue` files) to count consumers of each side.
2. If one side has **zero consumers**, delete it.
3. If one side has **one consumer**, consider inlining it into that consumer rather than renaming.
4. Otherwise, rename the more domain-specific side (typically the ui-layer composition) to a name that reflects its role, and update every consumer in one coordinated pass — folder name, file name, `<script setup name="...">`, barrel exports, every consuming import, and any string registries.

If your module ships a versioned release flow, bundle the rename into a single release entry — it's one conceptual change.

## Examples

Allowed:

- A ui-kit-layer `CategoryCard.vue` + a ui-app-layer `BlockCategoryCard.vue` — different stems (`BlockCategoryCard` ≠ `CategoryCard`).
- A ui-kit-layer `Pagination.vue` + a ui-layer `ListingPagination.vue` — the ui-layer composition uses a domain-specific name.

Not allowed:

- A ui-kit-layer `Text.vue` + a ui-layer `Text.vue` — same stem across layers.
- A ui-layer `CategoryCard.vue` + a ui-kit-layer `CategoryCard.vue` — same stem across layers.
