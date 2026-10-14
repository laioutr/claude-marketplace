# Hooks: Write to `result.value` Synchronously, and Expose Your Own Only Where Nothing Else Reaches

An app meets hooks from two sides. It **registers handlers** on Frontend Core and orchestr hooks, and it may **define hooks of its own** so a project can plug into the app without forking it. Both sides share one naming scheme and one result contract. The catalogue of platform hooks, with payloads, mechanics and dispatch, is at <https://docs.laioutr.com/frontend/features/hooks>.

## Hard rules

1. Name hooks `namespace:entity:action` in colon-separated kebab-case.
2. On a synchronously dispatched hook, write `result.value` (or mutate the payload) before any `await`. The caller does not wait for a returned promise.
3. Handler return values are ignored. Write to `result.value`, or mutate the payload of a modify hook.
4. Define a hook only where your app runs an effect the project can't otherwise reach.
5. Fire a result-returning client hook with `useHookResult` / `getHookResult`, never `callHook` followed by a read.

## Naming

Use your app's name as the namespace, then the entity, then the action: `frontend-core:link-resolver:resolve`, `orchestr:client-env:modify`, `gtm:data-layer:resolve` (from `@laioutr-app/gtm`). Use present tense for a hook called before or during an action, and past tense for one called after (`frontend-core:content-preview:changed`, `orchestr:query-params:parsed`). Lifecycle families append the phase: `orchestr:action:mutation:before | success | error | finally`.

## Registering handlers on platform hooks

Register client hooks from a Nuxt plugin with `nuxtApp.hook(...)`. Register server hooks (`orchestr:action:handler:*`, `frontend-core:redirect:send`) from a `defineNitroPlugin` with `nitroApp.hooks.hook(...)`. Every hook has a mechanic:

- **filter** — runs after the default with `result.value` pre-seeded; each handler sees the previous one's output;
- **override** — runs before the default with `result.value` empty; the first handler to set it wins;
- **modify** — mutate the payload in place; there is no `result`;
- **lifecycle** — `:before` / `:success` / `:error` / `:finally` around an operation.

The docs also list a dispatch for each hook, sync or async. On a **sync** hook the caller reads `result.value` on the very next line and ignores a returned promise, so anything written after an `await` is lost without an error:

```ts
// ✘ async handler on a sync filter — the title is applied before the await resolves
nuxtApp.hook('frontend-core:page-head:resolve', async ({ result }) => {
  const site = await $fetch<{ name: string }>('/api/app-acme/site');
  result.value.seo.title = `${result.value.seo.title} — ${site.name}`;
});

// ✅ load first, then read synchronously inside the handler
export default defineNuxtPlugin(async (nuxtApp) => {
  const site = await $fetch<{ name: string }>('/api/app-acme/site');
  nuxtApp.hook('frontend-core:page-head:resolve', ({ result }) => {
    result.value.seo.title = `${result.value.seo.title} — ${site.name}`;
  });
});
```

On a result-returning hook, a handler that throws is skipped with a warning and the remaining handlers still run. Because dispatch is synchronous, a reactive read inside a handler is tracked by the caller's `computed`. For example, `useResolvedLink()` re-evaluates when a source your `link-resolver:resolve` handler reads changes.

## Defining your own hook

**Where a hook earns its keep.** Add a hook where your app runs an effect the project can't otherwise reach. That is either an automatic emission site (your plugin calls `useHead`, pushes to a data layer, sets a cookie) or a value your runtime consumes internally. The test is whether the project can reach the effect, not whether the code is a composable. The one anti-pattern is a filter hook inside a **pure producer** — a composable that computes a value and *returns* it. The caller already owns that value and can transform it inline, so put the hook where the value is *applied*: one emission site, one hook.

**Type it** in `src/globalExtensions.ts`. Client hooks augment `RuntimeNuxtHooks` in `declare module '#app'`; server hooks augment `NitroRuntimeHooks` in `declare module 'nitropack'`.

**Fire a client result hook** with `useHookResult(name, args, seed?)`, or `getHookResult(name, args, nuxtApp, seed?)` when you already hold the Nuxt app. Both are auto-imported from `#imports`. They add the `result: { value }` slot and run handlers synchronously, so `result.value` is readable on the next line. `nuxtApp.callHook` defers handlers to a microtask, so a value read right after it is still the seed.

```ts
// src/globalExtensions.ts
import type { HookResult } from 'nuxt/schema';

declare module '#app' {
  interface RuntimeNuxtHooks {
    'acme:badge:label': (args: { productId: string; result: { value: string } }) => HookResult;
  }
}

// in your runtime code
const label = useHookResult('acme:badge:label', { productId }, defaultLabel) ?? defaultLabel;
```

- **Seed → filter.** Handlers run after the default and transform it. Use this to post-process a produced value. Seed the same slot you read back; a separate input field breaks threading across handlers.
- **No seed → override.** `result.value` starts `undefined`, and the caller falls back to the default if no handler sets it. Use this to select or replace a value, or for an escape hatch that must run before the default's error handling. Start with override when no consumer exists yet. Don't convert to filter speculatively: a filter runs after the default, so it loses the escape hatch.

**Fire a server hook** with `await useNitroApp().hooks.callHook(name, payload)`, and read `result.value` only after the `await`. Awaiting runs every handler in turn, so async handlers are safe there. For an operation projects should observe, fire a lifecycle family around it: `:before`, then `:success` or `:error`, then `:finally`.

## Related

- [`reactive-value-args.md`](./reactive-value-args.md) — the same reachability test applied to the arguments of imperative stores
