# Upstream Packages and the Resolution Ladder

Laioutr ships its design system as three layered npm packages, maintained
upstream by the Laioutr core team. Your module installs them as
dependencies — you do not author files inside them.

| Package | Contains | When you reach for it |
| --- | --- | --- |
| `@laioutr-core/ui-kit` | Atomic primitives — `Button`, `Input`, `Icon`, `OnSurface`, … | Compose these directly inside any component you write. |
| `@laioutr-core/ui` | Commerce organisms — `ProductTile`, `Header`, `HeroBanner`, … | Compose, override via props, or fork (see ladder below) when the design needs an organism. |
| `@laioutr-core/ui-app` | Section / Block infrastructure — `defineSection`, `defineBlock`, `definitionToProps` | Import the helpers when authoring your own Sections and Blocks. |

`ui` consumes `ui-kit`. `ui-app` consumes both. Never the other direction.

## Where your code goes

You ship one Nuxt module (typically `@laioutr-org/<provider>__<app>`)
with the three packages installed as dependencies. There is no separate
ui-kit, ui, or ui-app package for you to publish to — and no need to
sub-categorize your own components by upstream layer. Two directories
matter:

- `src/runtime/components/` — presentational components: anything that
  receives data via props and emits events. Custom primitives, custom
  organisms, layout helpers — all of them just live here.
- `src/runtime/app/section/` and `src/runtime/app/block/` —
  Studio-bound wrappers built with `defineSection` / `defineBlock` (see
  the `writing-section-block-definitions` skill).

The ui-kit / ui / ui-app distinction is about how the upstream packages
are layered for *consumption* — it tells you where to find an atomic
primitive vs. a commerce organism vs. the section/block helpers. It is
not a categorization you apply to components you write yourself.

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
   `src/runtime/components/<Name>/` and compose upstream primitives
   inside it.
3. **Fork from `https://github.com/laioutr/ui-source`** — when the
   upstream component is almost right and you need to adjust internals.
   Copy the relevant file into your module and modify locally.
   Acceptable, but not the first reach: you take on the maintenance.
4. **Open an upstream issue** — when the gap is generic and would
   benefit other apps. Improving upstream is preferred over a
   long-lived fork.

Never `:deep()` into upstream class names — they are not part of the
public CSS API and change between releases (see
[`public-css-api.md`](./public-css-api.md)).

## Discovery surfaces

Different questions, different surfaces:

| You need | Use |
| --- | --- |
| Props, emits, slots, usage, code samples | The official docs MCP server (registered in this plugin's `.mcp.json` as `laioutr-docs`, pointing at `https://docs.laioutr.io/mcp`). Falls back to context7 if the MCP is unreachable. |
| What visual states / variants a component supports | The official Storybook: <https://storybook.laioutr.cloud/> |
| The actual implementation (e.g. before forking) | The component source repo: <https://github.com/laioutr/ui-source> |
