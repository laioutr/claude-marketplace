# Three-Layer Architecture (upstream, consumed)

Laioutr ships its design system as three layered npm packages, maintained
upstream by the Laioutr core team. Your module installs them as
dependencies — you do not author files inside them.

| Package | Role | Contains |
| --- | --- | --- |
| `@laioutr-core/ui-kit` | Atoms / primitives | `Button`, `Input`, `Icon`, `OnSurface`, … |
| `@laioutr-core/ui` | Commerce organisms | `ProductTile`, `Header`, `HeroBanner`, … |
| `@laioutr-core/ui-app` | Section / Block infrastructure | `defineSection`, `defineBlock`, `definitionToProps` |

`ui` consumes `ui-kit`. `ui-app` consumes both. Never the other direction.

## Where your code goes

You ship one Nuxt module (typically `@laioutr-org/<provider>__<app>`) with
the three packages installed as dependencies. The majority of what you
author plays the **ui-app role**: Sections, Blocks, and shared-field
presets under `src/runtime/app/`. These wrap upstream organisms and bind
them to Studio schemas via `defineSection` / `defineBlock`.

Less often you author components that play other roles in the same single
module:

- **ui-layer role** — a custom organism when no upstream `@laioutr-core/ui`
  component fits and the gap is specific to your app's domain.
- **ui-kit-layer role** — a custom primitive when you need an atom that
  doesn't belong upstream.

These typically live under `src/runtime/components/`. They are still your
own components in your own module — there is no separate ui-kit or ui
package for you to publish them to.

## When upstream is missing something

When a Section needs a layout that `@laioutr-core/ui` doesn't provide,
choose in order:

1. **Prop or minor CSS override on the existing component** — preferred
   when a single prop or style change is enough.
2. **Build a local custom component in your module** — for gaps that are
   specific to your app's domain.
3. **Fork from `https://github.com/laioutr/ui-source`** — when the upstream
   component is almost right and you need to adjust internals. Copy the
   relevant file into your module and modify locally. Acceptable, but
   not the first reach: you take on the maintenance.
4. **Open an upstream issue** — when the gap is generic and would benefit
   other apps. Improving upstream is preferred over a long-lived fork.

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

## How rule files refer to "layers"

Throughout `rules/`, *"ui-kit-layer / ui-layer / ui-app-layer component"*
means **a component in your module that plays that role**. It is not a
file inside the upstream `@laioutr-core/*` package — those are off-limits
to your edits (forking copies code into your module; the upstream package
itself stays untouched).
