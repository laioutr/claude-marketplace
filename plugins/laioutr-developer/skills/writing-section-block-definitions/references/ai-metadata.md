# Leave `ai` Off; When You Use It, State Facts, Never Guidance

Leave the `ai` key off your `defineSection` / `defineBlock` definitions by default. Put any fact an agent needs into `studio.description` first. Use `ai.description` only for an agent-relevant fact that would clutter the picker text, and write it as a declarative fact, never as "use when", "avoid", "never with", "pairs well with" or "at most N" guidance. Use `ai.examples` only on a section that hosts several distinct block types whose intended assembly can't be guessed from the block names.

`ai` (`AiComponentMetadata` from `@laioutr-core/core-types/frontend`) has exactly two optional string fields, `description` and `examples`. Agents composing pages through the [Laioutr MCP server](https://docs.laioutr.com/agent-api/laioutr-mcp/supported-tools) read it. Studio's picker never shows it, and an agent receives it only when it fetches a component's full schema, never in the component list.

## Forbidden

```ts
// ✘ prescriptive guidance: use / avoid / max-N / pairing
ai: {
  description:
    'Use this on the homepage to grow the newsletter list. Avoid placing it more than once per page. Pairs well with SectionHeroBanner.',
},

// ✘ a paraphrase of what the label, description and schema already say
studio: { label: 'Store Finder', description: 'Find a store near you.' },
ai: { description: 'A section that lets visitors search for nearby stores.' },

// ✘ a placement rule written as prose instead of schema
ai: { description: 'Only BlockFaqItem blocks belong in the items slot.' },
```

## Required

```ts
// ✅ the default: the fact lives in studio.description, no `ai` key at all
studio: {
  label: 'Store Finder',
  description:
    "A map and list of your stores, searchable by postal code. Renders the stores from the page's Location query.",
},

// ✅ overflow: a non-obvious behaviour that would read oddly in the picker
ai: {
  description:
    'There is no trigger or scheduling engine: the pop-up opens on every page load, and closing it lasts only for the current session.',
},

// ✅ examples, on a section whose slots take several block types in a specific order
ai: {
  examples:
    "Child BlockConfigurator* blocks read this section's queried product, so they need no query of their own. A typical setup fills the `options` slot top to bottom with BlockConfiguratorMaterial, BlockConfiguratorSize and BlockConfiguratorSummary; the slot order is the on-page order.",
},

// ✅ a placement rule as schema, where Studio and the agent both enforce it
slots: [{ name: 'items', studio: { label: 'Questions' }, restrictTo: ['BlockFaqItem'] }],
```

## Writing `studio.description`

Keep it to two natural sentences at most. Lead with what the component renders, then add any non-obvious capability or data-binding fact ("plays video despite the 'Image' label", "renders the product from the page's Product query"). The same text is what an agent sees while choosing components, so it is the primary agent signal as well as the picker text.

## What earns a place in `ai.description`

Only facts an agent cannot infer from the label, `studio.description` and the field and slot schema:

- **Non-obvious automatic behaviour.** "The monthly/annual toggle appears once any plan sets an annual price."
- **Capability corrections.** "Plays video despite the 'Image' label."
- **Data binding.** "Renders the product from the page's Product query; there is no manual-entry fallback."
- **Current limitations.** "Shows progress only; it does not fetch the next batch."

Check each against `studio.description` first. If it would help an editor in the picker too, and it usually would, it belongs there instead.

## Why

1. **Instructions hurt composition.** Laioutr's evaluation rounds found that use/avoid/never/pairing guidance measurably degrades how agents compose pages. Cautions make agents leave out components the brief needed; capability brochures make them add ones it didn't. A declarative fact lets the agent apply its own judgment to the brief in front of it.
2. **Duplicates cost context and buy nothing.** The agent already has the label, `studio.description` and the full schema. A paraphrase of them is tokens with no new information.
3. **The agent chooses from the list, where `ai` isn't.** The component list carries `studio.description`; `ai` arrives only after a component's full schema is fetched. A fact that should influence the choice has to be in `studio.description`.
4. **Schema constraints are enforced; prose isn't.** Slot `restrictTo` and `allow`, and `isStandalone`, are checked by Studio for editors and agents alike. The same rule as text can drift from the schema and still be obeyed.
5. **Page-level knowledge is not a fact about your component.** Funnel position, how many per page, and which section usually follows are exactly the pairing rules point 1 warns about.

## When to reach for `examples`

- **Yes:** a section with several slots, or one slot that takes several distinct block types, where the intended order or combination is not evident from the block names (a product-detail family, a configurator, a header with separate desktop and mobile menu slots).
- **No:** a section with one obvious kind of child, any section whose blocks' names already say how they stack, and blocks. A block has no slots, so it has no assembly of its own to describe.

Write one concise worked composition: which blocks, in which slots, in what order, for what scenario.

## Related

- [Section definitions: AI metadata](https://docs.laioutr.com/apps/app-development/section-definitions#ai-metadata): the docs' version of these rules.
- [`section-config-standard.md`](./section-config-standard.md): `studio.*` fields sit in the "Section info & settings" panel, outside `schema`.
