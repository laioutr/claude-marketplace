# Mutations Go Through Orchestr Action Tokens

When one of your Sections, Blocks or data-connected components changes state or calls a backend imperatively (add to cart, submit a form, subscribe, register, save, delete, redeem a code), call it through an **Orchestr action token** with `useMutationAction(Token)`, or with `fetchAction(Token, input)` when you need no mutation state. Never inline the call: no `$fetch` / `useFetch`, no direct backend SDK or API call, no hand-built `POST` to `/api/orchestr/action/...`.

Split the work across the layers. The presentational component in `src/runtime/app/components/` emits the intent and takes a `loading` prop. The Section or Block wires that event to `useMutationAction`. The token itself lives in `src/runtime/shared/tokens/`, so your server handler and your frontend import the same value.

## Forbidden

```ts
// ✘ BlockCatalogRequest.vue: the browser talks to the CRM directly
const onSubmit = async (payload: { email: string; postalCode: string }) => {
  await $fetch('https://crm.example.com/api/catalog-requests', { method: 'POST', body: payload });
};
```

## Required

Use a canonical token when one fits. Check the [Canonical Actions](https://docs.laioutr.com/frontend/api-reference/ecommerce/actions) reference first (`@laioutr-core/canonical-types/ecommerce`, `/newsletter`, `/b2b`). Define your own only when nothing fits, and implement an action handler for it on the server:

```ts
// src/runtime/shared/tokens/catalog.ts
import { z } from 'zod/v4';
import { defineActionToken } from '@laioutr-core/core-types/orchestr';

export const CatalogRequestAction = defineActionToken('my-laioutr-app/catalog/request', {
  input: z.object({
    email: z.email(),
    postalCode: z.string(),
  }),
});
```

```vue
<!-- src/runtime/app/blocks/BlockCatalogRequest.vue -->
<script lang="ts">
import { defineBlock, definitionToProps, useMutationAction } from '#imports';
import CatalogRequestForm from '../components/CatalogRequestForm.vue';
import { CatalogRequestAction } from '../../shared/tokens/catalog';

export const definition = defineBlock({
  component: 'BlockCatalogRequest',
  studio: { label: 'Catalog Request' },
  schema: [],
});
</script>

<script setup lang="ts">
defineProps(definitionToProps(definition));

const { mutate: requestCatalog, isLoading } = useMutationAction(CatalogRequestAction);
</script>

<template>
  <!-- CatalogRequestForm emits `submit` with { email, postalCode } and renders `loading`; it never calls an action -->
  <CatalogRequestForm :loading="isLoading" @submit="requestCatalog" />
</template>
```

`useMutationAction` and `fetchAction` are auto-imports from `#imports`. `useMutationAction` wraps Pinia Colada's `useMutation`, so it returns `mutate`, `mutateAsync`, `isLoading`, `error` and `data`. Use `mutateAsync` inside a `try` when the Block needs to react to success or failure (a toast, a redirect).

Namespace a custom token with your app's name, as `<your-app>/<feature>/<action>`. The name becomes a URL path, so `defineActionToken` accepts only lower-case letters, digits, `-` and `/`: drop the `@` of a scoped package name.

## Why

1. **The token is the contract, the handler is swappable.** The Block depends on a token's typed input and output, not on one backend. A project can serve the same token from a different connector without touching your component.
2. **Inline calls skip the pipeline.** `fetchAction` attaches the client environment (market, language, preview) and fires the `orchestr:action:*` runtime hooks. `useMutationAction` adds the `orchestr:action:mutation:*` lifecycle hooks on top. Action middleware and typed errors run on the server. A `$fetch` gets none of this, so other apps listening to those hooks never see your mutation.
3. **Backend credentials stay on the server.** An action handler runs as a server route. A direct SDK or API call from a component runs in the visitor's browser, along with whatever key it needs.
4. **The component stays reusable and testable.** A presentational component that only emits can be rendered in Storybook and reused by another Block. One that calls an action is welded to that action.

## When the token doesn't exist yet

Don't inline a fetch "for now". Without a token there is nothing for a handler to implement. Define the token (input and output as `zod/v4` schemas), write its handler under `src/runtime/server/orchestr/`, then wire it here. See [Orchestr Actions](https://docs.laioutr.com/frontend/orchestr/actions) for tokens, handlers, middleware and error classes, and the [runtime layout](https://docs.laioutr.com/apps/app-development/coding-standards#runtime-layout) for where token files go.

## Related

- [`components-render-without-blocking.md`](./components-render-without-blocking.md): an `await` inside a mutation handler is fine. A top-level `await` in setup is not.
- [`tracking-never-degrades-the-page.md`](./tracking-never-degrades-the-page.md): how to report a mutation without letting the report fail it.
- [`three-layer-architecture.md`](../../laioutr-platform/references/three-layer-architecture.md): where presentational components, Sections and Blocks, and tokens live.
- The `designing-action-tokens` skill: designing the token itself — canonical first, naming, input/output schemas, errors.
