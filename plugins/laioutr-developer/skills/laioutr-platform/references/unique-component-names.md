# Unique Component Names

Every Vue component file you author in your Nuxt module must have a **globally unique basename** within your module.

No two components may share the same filename stem.

## The rule

For any new or renamed component `<Name>.vue` in your module, the stem `<Name>` must not already exist anywhere else in your module.

The `Block*` / `Section*` / `Connected*` prefixes used in Section/Block filenames count as part of the name — `BlockCategoryCard` does not collide with `CategoryCard`. The rule is about **bare filename stems** only.

**Sections and Blocks must be unique across the whole project, not just your module.** Laioutr registers them as global Vue components in one registry shared by every installed app — your module, `@laioutr-app/ui`, and any other app. If two apps register the same name, whichever loads last silently overwrites the other. Keep the `Section` / `Block` prefix, keep the definition's `component` value identical to the filename, and make the rest of the name specific to your app's domain (`SectionAcmeStoreFinder`, not `SectionStoreFinder`) when a generic name could plausibly exist elsewhere.

Upstream `@laioutr-core/ui-kit` / `@laioutr-core/ui` components are auto-imported with an `L` prefix (`LButton`, `LProductTileBasic`), so a component in your module's `src/runtime/app/components/` does not shadow them. Still avoid reusing their bare names — readers can't tell which one a template means.

## Why

1. **Registry hygiene.** Global component registration keeps one entry per name. Two Sections named `SectionHero` from two apps silently resolve to whichever app registered last, so a page renders a different component than the one its editor placed.
2. **Grep / LSP correctness.** Refactors that search for a component by name return both results and force the author to disambiguate every call site manually. Unique names make `findReferences` and grep authoritative.
3. **Cognitive load.** A reviewer reading `<CategoryCard />` in a template should not have to trace imports to know whether it's the upstream primitive or your module's composition. One name, one component.
4. **Storybook titles.** Duplicate stems end up under different Storybook categories with the same display name, breaking Chromatic snapshot keys and confusing designers.

## What to do when a collision would be introduced

Before creating or renaming a component:

1. Search for the stem across your module:

   ```
   Glob src/runtime/**/<Name>.vue
   ```

   For a Section or Block, also check the names already registered in the project — `@laioutr-app/ui`'s Sections and Blocks (`ls node_modules/@laioutr-app/ui/dist/runtime/app/section/ node_modules/@laioutr-app/ui/dist/runtime/app/block/`) and those of every other app the project installs. For other components, check the upstream Storybook (<https://storybook.laioutr.cloud/>) so you don't reuse an upstream name.

2. If any match exists, pick a different name:
   - **Generic, Figma-style noun** (`Button`, `Pagination`, `Text`) when nothing upstream conflicts.
   - **Domain-specific composition name** (`ListingPagination`, `CategoryTile`) when a generic name is taken.
   - **`Block*` / `Section*` / `Connected*`** files already carry their own prefix and rarely collide — but verify the suffix stem too (`BlockCategoryCard` → watch for `CategoryCard` elsewhere).

3. Do not resolve a collision by adding a `*Block` or `*Section` suffix to a non-block / non-section file — those suffixes are the Section/Block filename convention and using them elsewhere confuses readers.

## Resolving an existing collision

If an existing duplicate is discovered in your module:

1. Use LSP `findReferences` (or grep fallback for `.vue` files) to count consumers of each side.
2. If one side has **zero consumers**, delete it.
3. If one side has **one consumer**, consider inlining it into that consumer rather than renaming.
4. Otherwise, rename the more domain-specific side (typically the composition rather than the primitive it composes) to a more specific name, and update every consumer in one coordinated pass — folder name, file name, `<script setup name="...">`, barrel exports, every consuming import, and any string registries.

If your module ships a versioned release flow, bundle the rename into a single release entry — it's one conceptual change.

## Examples

Allowed:

- A custom atom `CategoryCard.vue` + a block `BlockCategoryCard.vue` — different stems (`BlockCategoryCard` ≠ `CategoryCard`).
- A custom atom `Pagination.vue` + an organism `ListingPagination.vue` — the organism uses a domain-specific name.

Not allowed:

- Two components named `Text.vue` anywhere in your module — same stem.
- A `CategoryCard.vue` in two different directories — same stem.
