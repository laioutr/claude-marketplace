---
name: laioutr-platform
description: Use when developing on the Laioutr e-commerce platform — building Nuxt 3 / Vue 3 storefronts, custom UI components, sections and blocks via `defineSection` / `defineBlock`, Orchestr query / link / component-resolver / action handlers, or any work involving the `@laioutr-core/*` packages (`frontend-core`, `orchestr`, `kit`, `core-types`, `canonical-types`, `ui-kit`, `ui`, `ui-app`, `ui-tokens`), `@laioutr-app/*` first-party integrations, or `@laioutr-org/<provider>__<app>` third-party apps. Skip for generic Node / pnpm / CI plumbing that doesn't touch Laioutr packages (lockfile resolution, dependency upgrades, build-tool config). Loads the platform overview, discovery-surface routing (docs MCP, Storybook, ui-source), the Orchestr handler reference, commit conventions, and the trigger index for ~30 read-on-demand convention references in `references/`.
---

# Laioutr Developer — Skill Guidance

This skill gives Claude context for working alongside developers building on
the Laioutr platform — Vue 3 / Nuxt 3 frontends, UI components, and Orchestr
data integrations.

> **Audience:** External developers building Laioutr Apps, custom UI
> components, or Orchestr handlers. This is **not** the internal monorepo
> guidance — internal-only paths, build setup, and namespaces have been
> intentionally removed.

## Working Style

Be a partner, not a sycophant. Be direct and constructive, offering solutions
alongside criticism. If a rule is missing or ambiguous, ask before guessing.

If the user provides a rule that's stable enough to encode, write it to a new
file in the `rules/` directory of their project (not into this plugin —
the plugin ships read-only rules).

For Laioutr-specific questions (Orchestr, sections/blocks, schema fields,
ui-kit / ui / ui-app components), prefer the official Laioutr docs MCP
server (`laioutr-docs`, registered in this plugin's `.mcp.json` at
`https://docs.laioutr.io/mcp`) over guessing. For general Vue, Nuxt,
TypeScript, Pinia, UnoCSS, Storybook, and Playwright questions, prefer
Context7 MCP if available.

## Laioutr Platform Overview

Laioutr is a Nuxt 3 + Vue 3 platform for building scalable e-commerce
storefronts. External developers typically interact with it through:

- **`@laioutr/*`** — Public packages on npmjs.org
- **`@laioutr-core/*`** — Customer-facing packages (npm.laioutr.cloud)
- **`@laioutr-app/*`** — First-party integration apps (Shopify, Shopware,
  Adobe Commerce, Ambiendo)
- **`@laioutr-org/<provider>__<app>`** — Third-party licensed integration
  apps (your packages live here)

Internal namespaces (`@laioutr-private/*`) are not visible externally.

### Core packages you'll consume

- `@laioutr-core/frontend-core` — Required Nuxt module for all storefronts
  (routing, state, data fetching)
- `@laioutr-core/orchestr` — Data orchestration layer using Pinia Colada
- `@laioutr-core/kit` — Utility functions and composables for building
  Laioutr Apps
- `@laioutr-core/core-types` — Shared TypeScript types for the platform
- `@laioutr-core/canonical-types` — Shared TypeScript types for apps built
  on top of the platform

### UI layer

- `@laioutr-core/ui-kit` — Atomic Vue 3 components (Button, Input, Icon,
  etc.)
- `@laioutr-core/ui` — Commerce-specific organisms built on `ui-kit`
- `@laioutr-core/ui-app` — Section/block components via `defineSection()`
- `@laioutr-core/ui-tokens` — Design tokens (Figma → Style Dictionary → CSS
  variables)

## Discovery surfaces

Different questions go to different surfaces:

| You need | Use |
| --- | --- |
| Props, emits, slots, usage, code samples for any upstream component | The official docs MCP (`laioutr-docs` → `https://docs.laioutr.io/mcp`). Falls back to context7 when unreachable. |
| What visual states / variants a component supports | <https://storybook.laioutr.cloud/> |
| The actual implementation (e.g. before forking) | <https://github.com/laioutr/ui-source> |

See [`references/three-layer-architecture.md`](./references/three-layer-architecture.md) for the resolution ladder when an upstream component is missing or almost-right.

### Topic deep dives (entry points for the docs MCP)

When the task hits one of these areas, the docs MCP is the right place
to dig in — these URLs are the canonical entry pages, not summaries to
duplicate inline:

| Area | Entry point |
| --- | --- |
| Multi-language storefronts | <https://docs.laioutr.io/frontend/features/multi-language-support> |
| Multi-market storefronts | <https://docs.laioutr.io/frontend/features/multi-market> |
| Custom page types and routing | <https://docs.laioutr.io/frontend/features/pagetypes> |
| Consent management / adapters | <https://docs.laioutr.io/frontend/features/consent-management> |
| Checkout & payment integrations | <https://docs.laioutr.io/checkout> |
| Hosting / deployment / BYO server | <https://docs.laioutr.io/hosting> |

## Section Component Pattern

Used in `ui-app` for configurable page sections:

```ts
import { defineSection, definitionToProps } from '#frontend-core';

const definition = defineSection({
  // schema definition
});

const props = definitionToProps(definition);
```

## Orchestr — Data Integrations

Orchestr is the data orchestration framework you'll extend when building
data integrations for Laioutr Apps. Handler types and the canonical
reference for each:

| Handler                 | Responsibility                                   | Docs |
| ----------------------- | ------------------------------------------------ | ---- |
| **Query / Link Handlers** | Fetch entity IDs and define relationships      | <https://docs.laioutr.io/frontend/orchestr/queries> |
| **Component Resolvers** | Resolve data components for entities             | <https://docs.laioutr.io/frontend/orchestr/component-resolvers> |
| **Action Handlers**     | Handle mutations and side effects                | <https://docs.laioutr.io/frontend/orchestr/actions> |

Adjacent mechanisms:

- **Middleware** — request/response interception: <https://docs.laioutr.io/frontend/orchestr/middleware>
- **Filters** — request/response filtering contract: <https://docs.laioutr.io/frontend/orchestr/filters>
- **Caching** — three cache layers + strategies: <https://docs.laioutr.io/frontend/orchestr/caching>

The official docs MCP server (`laioutr-docs`) indexes all of the above —
query it for handler signatures, registration patterns, and recipes.

## Bulk Refactoring Discipline

- **Preserve all comments.** When rewriting files, carry over every JSDoc
  block, TODO, and inline comment from the original.
- **Audit after refactors** — exported names → helper types → properties →
  runtime functions → comments. "Typecheck passes" is not sufficient.
- **TypeScript diamond inheritance:** When splitting an interface into
  layers where a child extends two parents that both carry the same
  property at different widths, don't make the mixin extend the base. Let
  base properties flow through one chain only.

## Rule Index

Convention references in `references/` are not auto-loaded — read the
file listed below when its trigger matches the task at hand. The trigger
after each `—` is the only signal you have; if a task plausibly matches,
open the file before writing code.

Group headings reflect the **role** the component your code touches plays
(atom-style primitive, organism, section/block), not separate packages —
your module is one package; see [`references/three-layer-architecture.md`](./references/three-layer-architecture.md).

### Architecture & discovery

- [`references/three-layer-architecture.md`](./references/three-layer-architecture.md) — what `@laioutr-core/ui-kit` / `ui` / `ui-app` are, where your code goes, and the ladder when upstream is missing something (prop override → local component → fork from `laioutr/ui-source` → upstream issue)

### Cross-cutting (apply to any component your module ships)

- [`references/money-currency-code.md`](./references/money-currency-code.md) — handling Money objects, prices, or currency in code, stories, or fixtures
- [`references/no-wrapper-css-overrides.md`](./references/no-wrapper-css-overrides.md) — before writing CSS in a parent that targets a child component's class names
- [`references/nuxt-hooks.md`](./references/nuxt-hooks.md) — naming or wiring a Nuxt hook
- [`references/parent-prefix-naming.md`](./references/parent-prefix-naming.md) — naming a Vue component, especially before reaching for `<Parent><Part>` compound naming
- [`references/public-css-api.md`](./references/public-css-api.md) — adding, renaming, or removing CSS class names exposed by any component you ship
- [`references/story-icons-must-exist.md`](./references/story-icons-must-exist.md) — referencing an icon name in a `.stories.ts`
- **`writing-storybook-stories` skill (sibling)** — writing or updating any `.stories.ts` file: meta titles, variant naming, no per-viewport exports, theming checks, refactor workflow. Auto-loads when the prompt mentions Storybook / stories / `.stories.ts`.
- [`references/surface-tone.md`](./references/surface-tone.md) — implementing a container that paints a background (light/dark/bright), or text/icons whose contrast must adapt
- [`references/typescript-refactoring.md`](./references/typescript-refactoring.md) — deleting or renaming a symbol, planning a migration, or doing reference lookups (use LSP, not grep)
- [`references/unique-component-names.md`](./references/unique-component-names.md) — picking a basename for a new Vue component file in your module
- [`references/vue-component-file-structure.md`](./references/vue-component-file-structure.md) — creating a new Vue component file; deciding where types live; sub-component naming
- [`references/vue-script-imports-single-block.md`](./references/vue-script-imports-single-block.md) — adding `import` statements to a Vue SFC
- [`references/vue-sfc-cross-package-props.md`](./references/vue-sfc-cross-package-props.md) — using `defineProps<T>()` with a type imported from another package, or hitting cross-package SFC build errors

### Atom-style components (ui-kit role)

Apply when your module ships a custom primitive — or a fork of an upstream `@laioutr-core/ui-kit` component.

- [`references/ui-kit-styling.md`](./references/ui-kit-styling.md) — writing CSS or picking tokens
- [`references/ui-kit-theming.md`](./references/ui-kit-theming.md) — themes, icons, translations, background-brightness
- [`references/ui-kit-no-outer-chrome-in-primitives.md`](./references/ui-kit-no-outer-chrome-in-primitives.md) — adding margin/padding around a primitive, or a consumer asking to override outer spacing
- [`references/ui-kit-css-first-responsive.md`](./references/ui-kit-css-first-responsive.md) — picking between CSS media queries and `useBreakpoints` / `useIsMobile`
- [`references/ui-kit-vue-template-type-casts.md`](./references/ui-kit-vue-template-type-casts.md) — using `as T | U` inside a template binding (resolves `vue/no-deprecated-filter`)
- [`references/ui-kit-todo-figma-marker.md`](./references/ui-kit-todo-figma-marker.md) — shipping a provisional design value before its Figma source-of-truth exists
- [`references/ui-kit-reka-ui.md`](./references/ui-kit-reka-ui.md) — wrapping a reka-ui primitive; a11y or state-styling questions
- [`references/reka-ui-wrapper-patterns.md`](./references/reka-ui-wrapper-patterns.md) — picking a wrapper pattern (split vs flat, as-child, Portal, typed props/emits) for a reka-ui primitive
- [`references/reka-ui-no-type-leakage.md`](./references/reka-ui-no-type-leakage.md) — declaring public props/emits/slots on a reka-ui wrapper, or barrel-exporting types

### Organism-style components (ui role)

Apply when your module ships a custom organism — or a fork of an upstream `@laioutr-core/ui` component.

- [`references/ui-ui-kit-first.md`](./references/ui-ui-kit-first.md) — before writing more than ~3 lines of structural markup; search upstream `@laioutr-core/ui-kit` first
- [`references/ui-z-ordering.md`](./references/ui-z-ordering.md) — picking a z-index — use `--z-index-*` tokens, never raw values

### Sections & Blocks (ui-app role)

The full Sections & Blocks ruleset now lives in the sibling **`writing-section-block-definitions` skill** — it auto-loads when the prompt mentions `defineSection`, `defineBlock`, `shared-fields/`, schema field types, or `Block*.vue` naming. The skill carries the sidebar-group order, canonical field names, forbidden-name list, `if` operators, factory literal-type discipline, and `defineSelectOptions` rules, plus the six detailed reference files behind them.

If you reach this section without that skill having loaded, name it explicitly in your prompt.
