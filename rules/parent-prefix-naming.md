# Parent-Prefix Naming Is Not the Default

A component named `<Parent><Part>` (e.g. `DropdownMenuItem`, `MenuSideBySideNode`) is a **claim** that the component belongs to a specific parent's compound API. Don't reach for that pattern just because a component happens to render inside something. Use it only when the constraint actually holds.

## When `<Parent><Part>` is right

All three must be true:

1. **The component is used by exactly one parent**, and the parent owns its lifecycle (slot composition, context, prop coupling).
2. **It is not useful outside that parent.** Its API references types, contexts, or assumptions only the parent provides.
3. **The compound is the documented API.** Consumers of the parent are expected to compose with the part (`<Tabs><TabsTrigger /></Tabs>`).

Examples that pass: `DropdownMenuItem`, `AccordionTrigger`, `DialogTitle`, `MenuSideBySideNode`, `CartCouponCodeAccordionInput`.

## When it's wrong

Any of these is enough to reject the parent prefix:

- **Two or more unrelated consumers** — even within one package. If `Footer` and `FooterMenu` both render the part, the part isn't owned by either; pick a name that describes what the *thing* is.
- **Generic API with no parent-specific types or context.** A `LoginReviewPanel` whose props are `title` / `description` / `isSuccess` is not a sub-component of `Review` — it's a generic info panel that `Review` happens to use.
- **The parent isn't an obvious composition surface.** `NavigationNodeButton` named after `NavigationMenu` looks compound, but the consumer wires its own data and the button is a standalone clickable list-row primitive.

## How to choose a name instead

Describe the part by what it **is**, not by who renders it:

- `MenuLinkItem` (used by `Footer` + `FooterMenu`) → `NavLinkItem` — it's a navigation list link primitive.
- `MenuSectionTitle` → `NavSectionHeading` — it's a heading inside a navigation list, not a piece of `Menu`.
- `LoginReviewPanel` (one of two `<Panel>`s inside `Review`) → `ReviewLoginPanel` if it stays Review-scoped, or rename to a generic `InfoPanel`/`CtaPanel` if the API is fully generic.
- `NavigationNodeButton` (already moved) → `MenuSideBySideNode` because it *is* tied to one parent's data shape.

When in doubt, the name should answer "what kind of thing is this?" before "where does it appear?".

## Why

1. **Auto-import collisions.** Nuxt auto-import registers components by basename. A misleading parent prefix increases the chance of a name clash with a future genuine sub-component, and produces silent first-match resolutions.
2. **False ownership signal.** A reader sees `MenuFooLinkItem` and assumes there is a `MenuFoo` parent that controls it. When there isn't, every refactor of the supposed parent has to detour through this component to confirm it's actually unrelated.
3. **Search precision.** `findReferences` on a parent name turns up its real sub-components plus every parent-prefixed sibling that just happens to share the prefix. Honest names keep the dependency graph honest.
4. **Resists premature abstraction.** Naming a thing after a parent freezes it into that parent's mental model. The part-name should survive the parent being renamed, deleted, or split. If the component would still make sense after the parent disappears, it shouldn't carry the parent's name.

## What to do when you spot a misnomer

1. Use LSP `findReferences` on the basename to count consumers and check API coupling.
2. If it has ≥2 unrelated consumers, or a generic API, rename it — touching folder name, file name, `<script setup name="...">`, all barrel exports, every consuming import, and any string references (registries, schemas) in a single coordinated pass.
3. Pick the new name so it answers "what is this?" without leaning on the parent. `Nav*`, `Link*`, `Info*`, `List*`, `Field*` are usually closer to the truth than `<ParentName>*`.
4. Bundle into the same commit as the move/refactor that exposed the misnomer — don't leave the wrong name as a follow-up.

## Relation to other rules

- Globally unique basenames across the ui-kit / ui / ui-app component layers: `rules/unique-component-names.md`.

This rule sits in front of name-mechanics rules: decide *whether* to use the compound naming pattern at all before applying the mechanics.
