# CSS-first Responsive Rendering

Do not use JS-derived viewport size to drive rendering when CSS can accomplish the same thing. Use `@media` queries.

## The rule

`useBreakpoints()` and `useIsMobile()` (exposed by `@laioutr-core/ui-kit` via `#ui-kit/composables`) must not decide:

- which DOM is rendered (`v-if="isMobile"` vs `v-else`, `<component :is="isMobile ? X : Y" />`)
- a prop value that has a direct CSS equivalent (layout, spacing, sizing, visibility)

CSS media queries are always preferred for any component that participates in SSR. Custom media queries available: `@media (--xs | --s | --sm | --md | --lg | --xl | --xxl)`. See [`ui-kit-styling.md`](./ui-kit-styling.md) for the min-width scale.

## The exception: post-interaction content

JS-driven breakpoints are allowed when the affected DOM **only mounts after a user interaction** — i.e. it is not present during SSR and hydration.

Typical cases: dropdown / popover / hover-card / dialog / sheet content (mounted on open, usually portaled), any subtree behind `v-if="isOpen"` on a user-driven toggle.

For these there is no server render to mismatch against, so a client-side computed on `useIsMobile()` is fine. Example: `DropdownMenu.vue` computes `side="bottom"` on mobile, but its content is only portaled when the user opens the menu.

**Self-check:** does this element exist in the initial SSR HTML? If yes → CSS. If no → either works.

## Why this matters

1. **Hydration mismatch.** `useBreakpoints` has no viewport on the server (the upstream composable doesn't yet derive `ssrWidth` from the request — see "What if the value can't be expressed in CSS?" below). The SSR render uses a default guess; the client measures the real viewport; hydration replaces the wrong tree. Users see a flash, and Vue warns in dev. CSS media queries resolve at paint time with the real viewport — no mismatch possible.
2. **Cacheability.** SSR HTML should be identical for every device. A JS viewport branch ties the rendered markup to a guess and defeats shared HTML caches, CDNs, and service workers.
3. **Performance.** A media query toggles a property. A JS computed re-runs on every resize and re-renders a subtree. For anything visible on first paint, CSS is strictly cheaper.

## What if the value can't be expressed in CSS?

Before reaching for the composable, ask whether the child can be restructured to accept a CSS-driven layout (display toggles, grid-template switches, container queries, `order`, etc. usually cover more than you'd expect). If it genuinely can't and the component is part of the initial SSR tree, prefer in this order:

1. **Server-aware breakpoint via upstream.** The proper long-term fix is for `useBreakpoints` to derive a real `ssrWidth` from UA parsing so server and client agree on the initial render. This isn't yet implemented upstream — if you need it, file an issue against `laioutr/ui-source` or fork the composable into your module. Don't try to monkey-patch the imported composable.
2. **Accept a hydration mismatch.** Ship the SSR default, let hydration swap the subtree on the client. The user sees visible content immediately and a brief flicker — tolerable, and crawlable.
3. **`<ClientOnly>` as a last resort only.** Wrapping in `<ClientOnly>` ships an empty subtree on the server: the content is invisible to crawlers, delays first paint, and causes CLS when it mounts. Prefer a hydration mismatch over `<ClientOnly>`.

Don't silently rely on the default without understanding which of the three you're opting into.
