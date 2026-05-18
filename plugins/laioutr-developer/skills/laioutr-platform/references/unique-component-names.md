# Unique Component Names

Every Vue component file you author in your Nuxt module must have a **globally unique basename** within your module — regardless of which role it plays (atom-style primitive, organism, section, block).

No two components may share the same filename stem.

## The rule

For any new or renamed component `<Name>.vue` in your module — whether it plays an atom-style, organism, or section/block role — the stem `<Name>` must not already exist anywhere else in your module.

The `Block*` / `Section*` / `Connected*` prefixes used in ui-app filenames count as part of the name — `BlockCategoryCard` does not collide with `CategoryCard`. The rule is about **bare filename stems** only.

Also check for collisions against upstream `@laioutr-core/ui-kit` / `@laioutr-core/ui` component names: a colliding name in your module silently overrides the upstream one via auto-import order.

## Why

1. **Auto-import hygiene.** Nuxt auto-import and template-based component resolution hit the first matching component name. Two components named `Text` will silently resolve to whichever module registered last, producing non-deterministic renders.
2. **Grep / LSP correctness.** Refactors that search for a component by name return both results and force the author to disambiguate every call site manually. Unique names make `findReferences` and grep authoritative.
3. **Cognitive load.** A reviewer reading `<CategoryCard />` in a template should not have to trace imports to know whether it's the ui-kit primitive or the ui composition. One name, one component.
4. **Storybook titles.** Duplicate stems end up under different Storybook categories with the same display name, breaking Chromatic snapshot keys and confusing designers.

## What to do when a collision would be introduced

Before creating or renaming a component:

1. Search for the stem across your module:

   ```
   Glob src/runtime/**/<Name>.vue
   ```

   Then check against the upstream Storybook (<https://storybook.laioutr.cloud/>) and `@laioutr-core/ui-kit` / `@laioutr-core/ui` exports for any collisions you'd shadow via auto-import.

2. If any match exists, pick a different name that reflects the component's role:
   - **Atom-style** components should use the generic, Figma-style noun (`Button`, `Pagination`, `Text`) when nothing upstream conflicts.
   - **Organism-style** compositions should carry domain intent (`ListingPagination`, `CategoryTile`) when a generic name is taken.
   - **Block/Section/Connected** files already carry their own prefix and rarely collide — but verify the suffix stem too (`BlockCategoryCard` → watch for `CategoryCard` elsewhere).

3. Do not resolve a collision by adding a `*Block` or `*Section` suffix to a non-block / non-section file — those suffixes belong to the ui-app filename convention and using them elsewhere confuses readers.

## Resolving an existing collision

If an existing duplicate is discovered in your module:

1. Use LSP `findReferences` (or grep fallback for `.vue` files) to count consumers of each side.
2. If one side has **zero consumers**, delete it.
3. If one side has **one consumer**, consider inlining it into that consumer rather than renaming.
4. Otherwise, rename the more domain-specific side (typically the ui-layer composition) to a name that reflects its role, and update every consumer in one coordinated pass — folder name, file name, `<script setup name="...">`, barrel exports, every consuming import, and any string registries.

If your module ships a versioned release flow, bundle the rename into a single release entry — it's one conceptual change.

## Examples

Allowed:

- A custom atom `CategoryCard.vue` + a block `BlockCategoryCard.vue` — different stems (`BlockCategoryCard` ≠ `CategoryCard`).
- A custom atom `Pagination.vue` + an organism `ListingPagination.vue` — the organism uses a domain-specific name.

Not allowed:

- Two components named `Text.vue` anywhere in your module — same stem.
- A `CategoryCard.vue` in two different directories — same stem.
