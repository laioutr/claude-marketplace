# Upstream Packages and the Resolution Ladder

Laioutr ships its design system as three layers, maintained upstream by
the Laioutr core team. Your module consumes them — you do not author
files inside them.

| Package | Contains | When you reach for it |
| --- | --- | --- |
| `@laioutr-core/ui-kit` | Atomic primitives — `Button`, `Input`, `Icon`, `OnSurface`, … Auto-imported with an `L` prefix (`LButton`, `LMedia`); aliased as `#ui-kit/...` | Compose these directly inside any component you write. |
| `@laioutr-core/ui` | Commerce organisms — `ProductTile`, `Header`, `HeroBanner`, … Auto-imported with an `L` prefix; aliased as `#ui/...` | Compose, override via props, or fork (see ladder below) when the design needs an organism. |
| `@laioutr-app/ui` | The ready-made Sections and Blocks editors place in Studio. It is an app, installed like any other | Use its Sections as-is, or read them as the reference for writing your own. |

`ui` consumes `ui-kit`. `@laioutr-app/ui` consumes both. Never the other
direction. The Section / Block helpers — `defineSection`, `defineBlock`,
`definitionToProps` — come from none of these: they are
`@laioutr-core/frontend-core` auto-imports (`#imports`).

## Where your code goes

You ship one Nuxt module (typically `@laioutr-org/<provider>__<app>`).
There is no separate ui-kit or ui package for you to publish to — and no
need to sub-categorize your own components by upstream layer. The
client-side tree is:

```text
src/runtime/app/
  components/   presentational components: props in, events out
  sections/     SectionXxx.vue — defineSection, registered with registerLaioutrApp
  blocks/       BlockXxx.vue — defineBlock, registered with registerLaioutrApp
src/runtime/shared/tokens/   Orchestr token declarations, imported by server and app
src/runtime/server/          Orchestr handlers, API clients, mappers
```

- `components/` holds custom primitives, custom organisms and layout
  helpers. It is not auto-discovered and not exposed to Studio — import
  from it directly.
- `sections/` and `blocks/` are the only client-side directories you pass
  to `registerLaioutrApp({ sections: [...], blocks: [...] })`; Orchestr
  handlers under `src/runtime/server/orchestr/` go in via
  `orchestrDirs: [...]`. See the `writing-section-block-definitions`
  skill.

The ui-kit / ui split is about how the upstream packages are layered for
*consumption* — it tells you where to find an atomic primitive vs. a
commerce organism. It is not a categorization you apply to components
you write yourself.

## When upstream is missing something

When a design needs a layout or primitive that upstream doesn't
provide, choose in order:

1. **Prop or minor CSS override on the existing upstream component** —
   preferred when a single prop or style change is enough. See
   [`public-css-api.md`](./public-css-api.md) for the override surfaces
   (CSS custom properties, BEM block names, data-attributes,
   slot-class props).
2. **Build a local component in your module** — for gaps that are
   specific to your app's domain. Drop the new component into
   `src/runtime/app/components/` and compose upstream primitives
   inside it.
3. **Fork the upstream component** — when it is almost right and you
   need to adjust internals. ui-kit primitives come from
   <https://github.com/laioutr-ui/ui-kit> (public); `ui` organisms from
   <https://github.com/laioutr-ui/ui> (licensed users only — without
   access, start from the installed source in
   `node_modules/@laioutr-core/ui/dist/runtime/components/`). Copy the
   relevant file into your module and modify locally. Acceptable, but
   not the first reach: you take on the maintenance.
4. **Open an upstream issue** on the matching repo (`laioutr-ui/ui-kit`
   or `laioutr-ui/ui`) — when the gap is generic and would benefit other
   apps. Improving upstream is preferred over a
   long-lived fork.

Never `:deep()` into upstream class names — they are not part of the
public CSS API and change between releases (see
[`public-css-api.md`](./public-css-api.md)).

For the discovery surfaces (docs MCP, Storybook, the source repos) that answer
"what props / states / source does an upstream component have," see the
**Discovery surfaces** table in the parent `laioutr-platform` skill.
