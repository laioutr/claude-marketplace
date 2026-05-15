# Laioutr Developer — Claude Guidance

This file gives Claude context for working alongside developers building on
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

For Vue, Nuxt, TypeScript, Pinia, UnoCSS, Storybook, and Playwright, prefer
official docs (via Context7 MCP if available) over guessing.

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

## Section Component Pattern

Used in `ui-app` for configurable page sections:

```ts
import { defineSection, definitionToProps } from '#frontend-core';

const definition = defineSection({
  // schema definition
});

const props = definitionToProps(definition);
```

When migrating stored configurations across breaking changes, see the
`writing-section-block-migration-manifest` skill.

## Orchestr — Data Integrations

Orchestr is the data orchestration framework you'll extend when building
data integrations for Laioutr Apps. There are four handler types:

| Handler                | Responsibility                                    |
| ---------------------- | ------------------------------------------------- |
| **Query Handlers**     | Fetch and return entity IDs                       |
| **Link Handlers**      | Define entity relationships                       |
| **Component Resolvers** | Resolve data components for entities             |
| **Action Handlers**    | Handle mutations and side effects                 |

Detailed platform documentation is in the Laioutr developer docs site —
the user should have access via their developer account.

## Commit Conventions

Conventional commits (Angular-style):

```
feat(package-name): add new feature
fix(my-app): [TICKET-123] resolve bug in component
chore: release my-app v1.0.0
```

See `rules/commit-scope-ui.md` for scope rules.

## Bulk Refactoring Discipline

- **Preserve all comments.** When rewriting files, carry over every JSDoc
  block, TODO, and inline comment from the original.
- **Audit after refactors** — exported names → helper types → properties →
  runtime functions → comments. "Typecheck passes" is not sufficient.
- **TypeScript diamond inheritance:** When splitting an interface into
  layers where a child extends two parents that both carry the same
  property at different widths, don't make the mixin extend the base. Let
  base properties flow through one chain only.

## How to use this plugin

This plugin ships skills, agents, and rules that encode Laioutr conventions.
Claude loads them automatically when triggered by the right context:

- **Skills** activate when you describe a matching task (e.g. "implement
  this Figma component" → `figma-to-component`)
- **Agents** are specialists you can delegate work to (e.g. `vue-expert`)
- **Rules** are read whenever Claude touches related code

See `README.md` for the full inventory and `MAPPING.md` for what was
intentionally excluded from the public plugin.
