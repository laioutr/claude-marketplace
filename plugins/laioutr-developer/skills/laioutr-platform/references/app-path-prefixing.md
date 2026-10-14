# Every Path Your App Owns Lives Under `/app-<name>/` or `/api/app-<name>/`

Your app shares one origin with the project's editor-created content pages, the platform's own routes, and every other installed app. Put every route, page and public asset your app registers under one of two prefixes. **The policy the path needs decides which one.** Who calls it and what it answers with do not.

| Prefix | Response | Crawlers | Unmatched path | Holds |
| --- | --- | --- | --- | --- |
| `/app-<name>/` | same for every visitor | allowed | rendered 404 page | public assets, custom Vue pages |
| `/api/app-<name>/` | may be per-visitor or single-use | disallowed | static 404 page, no app boot | fetch endpoints, webhooks, redirect endpoints |

A path belongs under `/api/` whenever a cached hit or a crawler hit would be wrong. So a browser-facing **redirect endpoint**, such as an OAuth callback or a checkout handoff, goes under `/api/` even though a browser navigates to it. Shopify's Customer Account API callback lives at `/api/app-shopify/auth/callback`.

```ts
// src/module.ts of @laioutr-org/acme__checkout, so <name> is `checkout`

// ✘ a bare path collides with an editor's /checkout page; a callback under /app-checkout/ is crawlable and cacheable
addServerHandler({ route: '/checkout', handler: resolveRuntimeModule('./server/routes/checkout') });
addServerHandler({ route: '/app-checkout/auth/callback', method: 'get', handler: resolveRuntimeModule('./server/routes/authCallback') });

// ✅
addServerHandler({ route: '/api/app-checkout/handoff', handler: resolveRuntimeModule('./server/routes/handoff') });
addServerHandler({ route: '/api/app-checkout/auth/callback', method: 'get', handler: resolveRuntimeModule('./server/routes/authCallback') });
addServerHandler({ route: '/api/app-checkout/webhooks/:topic', method: 'post', handler: resolveRuntimeModule('./server/routes/webhook') });
```

`addServerHandler` comes from `@nuxt/kit`. `resolveRuntimeModule` is the module's own `(path) => resolve('./runtime', path)` helper (see the docs' module structure).

## `<name>`

`<name>` is always the app segment of your package name. A third-party package's provider segment never appears in the path.

| Package | `<name>` |
| --- | --- |
| `@laioutr-app/shopify` | `shopify` |
| `@laioutr/app-hygraph` | `hygraph` |
| `@laioutr-org/acme__checkout` | `checkout` |

Use the same segment for assets, routes and pages, and name your public-assets folder after it, so the app occupies one predictable namespace. Two apps that resolve to the same `<name>` claim the same paths, and nothing warns you.

## Where each kind of path goes

- **Public assets.** Put files in `src/runtime/app/public/app-<name>/`. They are served at `/app-<name>/…`. That is where `defineOrchestr.meta({ logoUrl })` points (`/app-shopify/shopify-logo.svg`), along with any image, font or icon your runtime references. Register the folder once in `module.ts` with `nuxt.options.nitro.publicAssets.push({ dir: resolveRuntimeModule('./app/public') })`. The app starter does not do this for you.
- **Custom Vue pages** your app adds to the router go under `/app-<name>/…`.
- **Fetch endpoints** your own JS calls go under `/api/app-<name>/…`.
- **Redirect endpoints** that an external service or a click sends the browser to go under `/api/app-<name>/…`.
- **Inbound webhooks** go under `/api/app-<name>/webhooks/…`. No platform gate protects them, so verify the vendor's signature yourself.

## Caching and crawlers are your handler's job

- **The platform sets no cache header on either prefix.** Two shapes under `/api/app-<name>/` must never reach a shared cache. The first is a redirect endpoint, because its authorization code is single-use. The second is a per-visitor `GET` that sets no cookie. HTTP already keeps a response with `Set-Cookie`, and every `POST`, out of shared caches.
- **Spell your prefix out in a Nitro route rule.** `/api/app-checkout/**` matches. `/api/app-*/**` matches nothing, because radix3 matches `*` against a whole segment, never a prefix inside one.
- **Write no robots rule.** One `Disallow: /api/` line in the project's robots.txt excludes every installed app at once.
- **Unmatched `/api/**` paths answer a static 404 without booting the app.** Nitro prefers the deeper match, so your registered route still wins over that catch-all.

## Paths a standard fixes

Register a path under `/.well-known/` only when a published standard (an RFC or a protocol spec) fixes that exact path. Such a path cannot carry your app's name, because the client that fetches it never learns it. The Shopify app serves the Universal Commerce Protocol profile at `/.well-known/ucp`.

You own the collision. Nitro keeps one handler per route and gives no warning, so a second app that registers the same path silently replaces yours. Register one only when no other app that can be installed beside yours makes the same claim. Every other `/.well-known/` path answers the platform's plain 404.

## Reserved by the platform

Never register a route under any of these:

- **`/api/laioutr/*`** is gated behind the project secret (`Authorization: Bearer <projectSecretKey>`). A link click or redirect sends no such header and gets `401`.
- **`/api/orchestr/*`** is Orchestr's own query and action transport.
- **`/api/frontend/*`** holds Frontend Core's and the platform runtime's endpoints.
- **`/api/public/*`** is the platform's class for callers outside the platform. It is gated by a project-scoped restricted key and the project's allowed-origin list, and it answers 404 in a project that has not enabled it. An app route there would inherit a credential check your app does not control.

## Why

1. **Collisions are silent.** Editors create content pages at arbitrary slugs, and Nitro keeps one handler per route without warning. An un-namespaced `/cart` either shadows a real page or is shadowed by one, depending on registration order.
2. **The prefix carries the policy.** Crawlers, the 404 behaviour and the robots rule all key off `/api/`. Choose the prefix by the policy and the platform applies the right behaviour without any configuration from you.
3. **A wrongly cached redirect breaks sign-in.** An OAuth callback served from a shared cache replays a single-use code to the next visitor. Nothing upstream stops it, because your handler owns the cache header.
4. **Reserved namespaces carry gates you don't control.** A browser navigation through `/api/laioutr/*` always gets `401`. A route under `/api/public/*` disappears in every project that has not enabled the public API.

## Hard rules

1. **Never use a bare top-level path** (`/checkout`, `/cart`, `/account`).
2. **Never route a browser navigation through `/api/laioutr/*`.** It answers `401` to every link click and every redirect.
3. **Never use a wildcard inside a segment in a route rule** (`/api/app-*/**`). Spell out your own prefix.

Full write-up: [Coding standards → Paths and routes](https://docs.laioutr.com/apps/app-development/coding-standards#paths-and-routes).

## Related

- [`fail-soft-resilience.md`](./fail-soft-resilience.md): a webhook signature check is a security boundary and fails hard; it never falls back.
