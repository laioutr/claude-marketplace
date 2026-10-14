---
name: laioutr-platform
description: Use when developing on the Laioutr e-commerce platform — building Nuxt 3 / Vue 3 storefronts, custom UI components, sections and blocks via `defineSection` / `defineBlock`, Orchestr query / link / component-resolver / action handlers, or any work involving the `@laioutr-core/*` packages (`frontend-core`, `orchestr`, `kit`, `core-types`, `canonical-types`, `ui-kit`, `ui`, `ui-tokens`), `@laioutr-app/*` first-party apps (including `@laioutr-app/ui`, the ready-made Sections and Blocks), or `@laioutr-org/<provider>__<app>` third-party apps. Skip for generic Node / pnpm / CI plumbing that doesn't touch Laioutr packages (lockfile resolution, dependency upgrades, build-tool config). Loads the platform overview, discovery-surface routing (docs MCP, Storybook, the `laioutr-ui` source repos), the Orchestr handler reference, and the trigger index for ~30 read-on-demand convention references in `references/`.
---

# Laioutr Developer — Skill Guidance

This skill gives Claude context for working alongside developers building on
the Laioutr platform — Vue 3 / Nuxt 3 frontends, UI components, and Orchestr
data integrations.

> **Audience:** External developers building Laioutr Apps, custom UI
> components, or Orchestr handlers. This is **not** the internal monorepo
> guidance — internal-only paths, build setup, and namespaces have been
> intentionally removed.

If the user provides a rule that's stable enough to encode, write it to a new
file in the `rules/` directory of their project (not into this plugin —
the plugin ships read-only rules).

For Laioutr-specific questions (Orchestr, sections/blocks, schema fields,
upstream `@laioutr-core/ui-kit` / `ui` components, `@laioutr-app/ui`
sections), prefer the
official Laioutr docs MCP server (`laioutr-docs`, registered in this
plugin's `.mcp.json` at `https://docs.laioutr.com/mcp`) over guessing.
For general Vue, Nuxt, TypeScript, Pinia, UnoCSS, Storybook, and
Playwright questions, prefer Context7 MCP if available.

## Laioutr Platform Overview

Laioutr is a Nuxt 3 + Vue 3 platform for building scalable e-commerce
storefronts. External developers typically interact with it through:

- **`@laioutr/*`** — Public packages on npmjs.org
- **`@laioutr-core/*`** — Customer-facing packages (npm.laioutr.cloud)
- **`@laioutr-app/*`** — First-party apps: commerce connectors (Shopify,
  Shopware, Adobe Commerce), consent and tracking (GTM, Cookiebot, CCM19),
  the CMS (`@laioutr-app/cms`) and the ready-made Sections/Blocks
  (`@laioutr-app/ui`)
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
  etc.), auto-imported as `LButton`, `LInput`, … and aliased as `#ui-kit/...`
- `@laioutr-core/ui` — Commerce-specific organisms built on `ui-kit`,
  also `L`-prefixed, aliased as `#ui/...`
- `@laioutr-app/ui` — The ready-made Sections and Blocks (an app, not a
  core package)
- `@laioutr-core/ui-tokens` — Design tokens (Figma → Style Dictionary → CSS
  variables)

## Discovery surfaces

Different questions go to different surfaces:

| You need | Use |
| --- | --- |
| Props, emits, slots, usage, code samples for any upstream component | The official docs MCP (`laioutr-docs` → `https://docs.laioutr.com/mcp`). Falls back to context7 when unreachable. |
| What visual states / variants a component supports | <https://storybook.laioutr.cloud/> |
| The actual implementation (e.g. before forking), or where to file an upstream issue | `@laioutr-core/ui-kit`: <https://github.com/laioutr-ui/ui-kit> (public). `@laioutr-core/ui`: <https://github.com/laioutr-ui/ui> (licensed users only). Without access, read the installed `.vue` sources under `node_modules/@laioutr-core/<pkg>/dist/runtime/`. |

See [`references/three-layer-architecture.md`](./references/three-layer-architecture.md) for the resolution ladder when an upstream component is missing or almost-right.

### Topic deep dives (entry points for the docs MCP)

When the task hits one of these areas, the docs MCP is the right place
to dig in — these URLs are the canonical entry pages, not summaries to
duplicate inline:

| Area | Entry point |
| --- | --- |
| Multi-language storefronts | <https://docs.laioutr.com/frontend/features/multi-language-support> |
| Multi-market storefronts | <https://docs.laioutr.com/frontend/features/multi-market> |
| Custom page types and routing | <https://docs.laioutr.com/frontend/features/pagetypes> |
| Consent management / adapters | <https://docs.laioutr.com/frontend/features/consent-management> |
| Tracking / analytics events (`useAnalytics().track(...)`) | <https://docs.laioutr.com/frontend/features/tracking> |
| Bot protection | <https://docs.laioutr.com/frontend/features/bot-protection> |
| Studio content preview | <https://docs.laioutr.com/frontend/features/content-preview> |
| Hooks (filter / bail / lifecycle) | <https://docs.laioutr.com/frontend/features/hooks> |
| Where to start on a new app or connector | <https://docs.laioutr.com/apps/app-development/implementation-overview> |
| App conventions (layout, paths and routes, naming) | <https://docs.laioutr.com/apps/app-development/coding-standards> |
| App manifest (`laioutr.manifest.json`, settings form) | <https://docs.laioutr.com/apps/app-development/app-manifest> |
| Releasing an app | <https://docs.laioutr.com/apps/app-development/releasing-an-app> |
| CMS content collections | <https://docs.laioutr.com/apps/app-development/content-collections> |
| Media library connectors | <https://docs.laioutr.com/apps/app-development/media-libraries> |
| Checkout & payment integrations | <https://docs.laioutr.com/checkout> |
| Hosting / deployment / BYO server | <https://docs.laioutr.com/hosting> |

## Section Component Pattern

A Section is one `.vue` file in `src/runtime/app/sections/`. The
definition is exported from a plain `<script>` block; the props come
from it. `defineSection`, `defineBlock` and `definitionToProps` are
`@laioutr-core/frontend-core` auto-imports:

```vue
<script lang="ts">
import { defineSection, definitionToProps } from '#imports';

export const definition = defineSection({
  component: 'SectionHeroBanner', // must match the file name
  studio: { label: 'Hero Banner' },
  slots: [], // required — list the slots blocks can be placed in, or []
  schema: [
    {
      label: 'Content',
      fields: [{ type: 'text', name: 'heading', label: 'Heading' }],
    },
  ],
});
</script>

<script setup lang="ts">
const props = defineProps(definitionToProps(definition));
</script>
```

Register the directory with `registerLaioutrApp({ sections: [resolve('./runtime/app/sections')] })`.
Blocks work the same way with `defineBlock` in `src/runtime/app/blocks/`.
Schema conventions live in the `writing-section-block-definitions` skill.

## Orchestr — Data Integrations

Orchestr is the data orchestration framework you'll extend when building
data integrations for Laioutr Apps. Each handler implements a **token** —
the contract between frontend, Orchestr and apps.

| Handler | Responsibility | Builder method | Docs |
| --- | --- | --- | --- |
| **Query handler** | Return entity IDs for a query token | `.queryHandler` | <https://docs.laioutr.com/frontend/orchestr/queries> |
| **Query template provider** | Offer sample inputs so Studio can preview a query (provide them for id queries, not slug queries) | `.queryTemplateProvider` | <https://docs.laioutr.com/frontend/orchestr/queries#query-template-providers> |
| **Link handler** | Return related entity IDs for a link token | `.linkHandler` | <https://docs.laioutr.com/frontend/orchestr/queries#links> |
| **Component resolver** | Resolve entity components for a set of IDs | `.componentResolver` | <https://docs.laioutr.com/frontend/orchestr/component-resolvers> |
| **Action handler** | Mutations and side effects | `.actionHandler` | <https://docs.laioutr.com/frontend/orchestr/actions> |
| **Page index** | Enumerate the concrete pages behind a dynamic page type | `.pageIndex` | <https://docs.laioutr.com/frontend/orchestr/page-index> |

### Building blocks

- **Tokens** — `defineQueryToken`, `defineLinkToken`, `defineActionToken`
  and `defineEntityComponentToken` from `@laioutr-core/core-types/orchestr`;
  `definePageTypeToken` from `@laioutr-core/core-types/frontend`. Canonical
  tokens and entity components come from `@laioutr-core/canonical-types`.
  Your own tokens go in `src/runtime/shared/tokens/<feature>.ts` — both
  server handlers and frontend components import them. Schemas use
  `import { z } from 'zod/v4'`. **Namespace every token name you define**
  with your app (`store-locator/store/search`, never `store/search`):
  names are global across all installed apps and only canonical tokens
  own the bare `ecommerce/…` / `newsletter/…` space. Names are lowercase
  `a-z0-9-/` — for a scoped package drop the `@` and join with `-`
  (`@acme/store-locator` → `acme-store-locator/…`).
- **Base builder** — in `src/runtime/server/middleware/`, create
  `defineOrchestr.meta({ app, label, logoUrl })` (`app` = your package
  name, `logoUrl` = a path under your public assets) and chain
  `.extendRequest(async ({ event, clientEnv }) => ({ context: { … } }))`
  to build the per-request API client. Re-export its methods
  (`defineMyAppQuery = base.queryHandler`, …) so every handler receives
  that `context`. Without one, the
  same builders are auto-imported as `defineQueryHandler`,
  `defineLinkHandler`, `defineComponentResolver`, `defineActionHandler`,
  `defineQueryTemplateProvider`, `definePageIndex`.
- **Registration** — `registerLaioutrApp` from `@laioutr-core/kit` in
  your `module.ts`: handlers are discovered from
  `orchestrDirs: [resolve('./runtime/server/orchestr')]`, alongside
  `sections` / `blocks`. Handler files there follow
  `<entity>/<name>.query.ts`, `.link.ts`, `.action.ts`, `base.resolver.ts`.

### Gotchas

- **Batch.** A link handler or component resolver gets many `entityIds`
  — make one backend call for all of them, never one per ID.
- **Client environment.** Read `clientEnv.language.code` and
  `clientEnv.market.currency`; both are always populated, so write no
  fallbacks. The flat `clientEnv.locale` / `clientEnv.currency` are
  deprecated. See <https://docs.laioutr.com/frontend/orchestr/client-env>.
- **Cookies and headers** can be set only in `extendRequest` or in an
  action handler — queries, links and resolvers stream, so their headers
  are already sent. Use the auto-imported
  `setManagedCookie(event, name, value, options?)` /
  `deleteManagedCookie`, never h3's `setCookie`.
- **Caching** — per-handler `cache` blocks:
  `cache: { ttl: '1h' }` or `cache: { ttl: '1h', swr: true, staleMaxAge: '1d' }`
  (a number means seconds; a string is parsed by `ms`); a handler with no
  `cache` block is not cached. There is no `strategy`
  field any more (`'ttl' | 'swr' | 'live'` was removed) — ignore it if a
  docs page still shows it. See <https://docs.laioutr.com/frontend/orchestr/caching>.
- **Several apps, one token.** When two installed apps implement the same
  token, the handler's `order` decides (highest wins), unless the project
  picks a provider in `laioutrrc.json` → `config.orchestr.providers`. See
  <https://docs.laioutr.com/frontend/orchestr/provider-selection>.

Adjacent mechanisms: **Middleware** (`extendRequest`, request context —
<https://docs.laioutr.com/frontend/orchestr/middleware>) and **Filters**
(the faceted-search `filter` request and `availableFilters` response
shapes — <https://docs.laioutr.com/frontend/orchestr/filters>).

The official docs MCP server (`laioutr-docs`) indexes all of the above —
query it for handler signatures, registration patterns, and recipes.

## Rule Index

Convention references in `references/` are not auto-loaded — read the
file listed below when its trigger matches the task at hand. The trigger
after each `—` is the only signal you have; if a task plausibly matches,
open the file before writing code.

### Architecture & discovery

- [`references/three-layer-architecture.md`](./references/three-layer-architecture.md) — what `@laioutr-core/ui-kit` / `ui` and `@laioutr-app/ui` contain, where your code goes (`app/components`, `app/sections`, `app/blocks`, `shared/tokens`), and the ladder when upstream is missing something (prop override → local component → fork from `laioutr-ui/ui-kit` / `laioutr-ui/ui` → upstream issue)

### App packaging, data integrations, and testing

- [`references/app-path-prefixing.md`](./references/app-path-prefixing.md) — registering a server route, page, webhook, OAuth callback or public asset in your app's module, or setting a Nitro route rule
- [`references/app-settings-manifest.md`](./references/app-settings-manifest.md) — your app needs per-environment settings or credentials from the customer: writing or changing `laioutr.manifest.json`, adding a connection test, or a settings form that doesn't appear after publishing
- [`references/fail-soft-resilience.md`](./references/fail-soft-resilience.md) — mapping backend data in a query, link, resolver, webhook or `module.ts` setup, or before adding a try/catch, fallback value or `throw` there
- [`references/testing-with-nuxt-context.md`](./references/testing-with-nuxt-context.md) — writing a unit test for a composable, store, plugin or component in your app, or before adding a parameter or interface to make code testable without Nuxt

### Component conventions (apply to any component your module ships)

- [`references/boolean-prop-naming.md`](./references/boolean-prop-naming.md) — naming a boolean prop; deciding whether it takes an `is` prefix (state the component is in → bare token; fact about an external entity → `is*`)
- [`references/component-state-contract.md`](./references/component-state-contract.md) — adding v-model, validation (`invalid` / `errorMessage`), or `Field` context to a form input or overlay
- [`references/money-currency-code.md`](./references/money-currency-code.md) — handling Money objects, prices, or currency in code, stories, or fixtures; converting a decimal price into `Money`
- [`references/no-hardcoded-ui-strings.md`](./references/no-hardcoded-ui-strings.md) — adding user-visible text to a component; before shipping an English/German string default instead of a locale key
- [`references/no-wrapper-css-overrides.md`](./references/no-wrapper-css-overrides.md) — before writing CSS in a parent that targets a child component's class names
- [`references/nuxt-hooks.md`](./references/nuxt-hooks.md) — registering a handler on a platform hook (`frontend-core:*`, `orchestr:*`), or exposing a hook from your app
- [`references/parent-prefix-naming.md`](./references/parent-prefix-naming.md) — naming a Vue component, especially before reaching for `<Parent><Part>` compound naming
- [`references/prop-naming-vocabulary.md`](./references/prop-naming-vocabulary.md) — naming a prop for size, icon, link/`src`, or visual `variant`; deciding whether to rename a `title` / `description` prop
- [`references/public-css-api.md`](./references/public-css-api.md) — adding, renaming, or removing CSS class names exposed by any component you ship
- [`references/story-icons-must-exist.md`](./references/story-icons-must-exist.md) — referencing an icon name in a `.stories.ts`
- [`references/sub-part-tag-prop-naming.md`](./references/sub-part-tag-prop-naming.md) — exposing a prop or schema field that controls the HTML tag of a named sub-part (heading, subline, caption); name it `<part>As`
- [`references/surface-tone.md`](./references/surface-tone.md) — implementing a container that paints a background (light/dark/bright), or text/icons whose contrast must adapt
- [`references/typescript-refactoring.md`](./references/typescript-refactoring.md) — deleting or renaming a symbol, planning a migration, or doing reference lookups (use LSP, not grep)
- [`references/unique-component-names.md`](./references/unique-component-names.md) — picking a basename for a new Vue component file in your module
- [`references/vue-component-file-structure.md`](./references/vue-component-file-structure.md) — creating a new Vue component file; deciding where types live; sub-component naming
- [`references/vue-script-imports-single-block.md`](./references/vue-script-imports-single-block.md) — adding `import` statements to a Vue SFC
- [`references/vue-sfc-cross-package-props.md`](./references/vue-sfc-cross-package-props.md) — using `defineProps<T>()` with a type imported from another package, or hitting cross-package SFC build errors

### Styling, theming, and CSS

- [`references/ui-kit-styling.md`](./references/ui-kit-styling.md) — writing CSS or picking tokens
- [`references/ui-kit-theming.md`](./references/ui-kit-theming.md) — themes, icons, themable images (`themeMedia`), surface tone
- [`references/ui-kit-no-outer-chrome-in-primitives.md`](./references/ui-kit-no-outer-chrome-in-primitives.md) — adding margin/padding around a primitive, or a consumer asking to override outer spacing
- [`references/ui-kit-css-first-responsive.md`](./references/ui-kit-css-first-responsive.md) — picking between CSS media queries and `useBreakpoints` / `useIsMobile`
- [`references/ui-z-ordering.md`](./references/ui-z-ordering.md) — picking a z-index — use `--z-index-*` tokens, never raw values
- [`references/component-state-attributes.md`](./references/component-state-attributes.md) — styling a component's runtime state (open, disabled, selected) or choosing a `data-*` attribute name for it

### Vue template and import patterns

- [`references/ui-kit-vue-template-type-casts.md`](./references/ui-kit-vue-template-type-casts.md) — using `as T | U` inside a template binding (resolves `vue/no-deprecated-filter`)
- [`references/ui-kit-todo-figma-marker.md`](./references/ui-kit-todo-figma-marker.md) — shipping a provisional design value before its Figma source-of-truth exists
- [`references/translations-tl-vs-useLocale.md`](./references/translations-tl-vs-useLocale.md) — translating text in a component — choosing between `$tl` in the template and `useLocale()` in script
- [`references/reactive-value-args.md`](./references/reactive-value-args.md) — writing a composable or store that keeps a caller's value to render later, or passing translated text to `openDialog` / `addToast`
- [`references/ui-ui-kit-first.md`](./references/ui-ui-kit-first.md) — before writing more than ~3 lines of structural markup; search upstream `@laioutr-core/ui-kit` first

### Reka-UI integration

- [`references/ui-kit-reka-ui.md`](./references/ui-kit-reka-ui.md) — wrapping a reka-ui primitive; a11y or state-styling questions
- [`references/reka-ui-wrapper-patterns.md`](./references/reka-ui-wrapper-patterns.md) — picking a wrapper pattern (split vs flat, as-child, Portal, typed props/emits) for a reka-ui primitive
- [`references/reka-ui-no-type-leakage.md`](./references/reka-ui-no-type-leakage.md) — declaring public props/emits/slots on a reka-ui wrapper, or barrel-exporting types

### Sibling skills

- **`writing-storybook-stories`** — auto-loads when the prompt mentions Storybook / stories / `.stories.ts`. Carries meta titles, variant naming, no per-viewport exports, theming checks, and the refactor workflow.
- **`writing-section-block-definitions`** — auto-loads when the prompt mentions `defineSection`, `defineBlock`, `shared-fields/`, schema field types, or `Block*.vue` naming. Carries the sidebar-group order, canonical field names, forbidden-name list, `if` operators, the per-field-type runtime contract, factory literal-type discipline, and `defineSelectOptions` rules, plus detailed reference files (runtime behavior, mutations, tracking, AI metadata, sizing, text styling, query fields).
- **`designing-entity-components`** — defining entity types or `defineEntityComponentToken` components in your app, link tokens between entities, and how component fields behave as a Laioutr CMS form.
- **`designing-action-tokens`** — adding or reshaping an action token (`defineActionToken`) and its input, output and errors.
- **`figma-design-analysis` → `figma-component-architecture` → `figma-to-component`**, plus **`figma-export-assets`** — the Figma-to-code pipeline.

If a task clearly matches one of these and it hasn't loaded, name it explicitly in your prompt.
