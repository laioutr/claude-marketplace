# Prop Naming Vocabulary

Use one canonical word per concept. Applies to any component your module ships, and to the upstream `@laioutr-core/ui-kit`, `@laioutr-core/ui`, and `@laioutr-core/ui-app` patterns you mirror.

## Sizes — one scale

```
… '4xs' | '3xs' | '2xs' | 'xs' | 's' | 'sm' | 'm' | 'ml' | 'l' | 'xl' | '2xl' | '3xl' | '4xl' …
```

- Default is `'m'`.
- A component picks the **contiguous subset** relevant to it (most use `'xs' | 's' | 'm' | 'l'`, adding `'xl'` only where Figma demands it).
- Long-form (`'small'` / `'medium'` / `'large'` / `'default'`) is **outlawed**.
- Exception: `Text` keeps its full typographic scale by design.

## Icons

`icon` (single), `iconLeft` / `iconRight` (paired). Never `iconName`, `leftIcon`, or `rightIcon`.

## Links and sources

- `href` everywhere for navigation targets. No `url`, no `link: { href, text }` object.
- `src` for embeds and media sources (`<iframe>`, `<img>`).

## `variant`, not `style` / `type`

A render-time visual variant prop is named `variant`.

```ts
// ✅
defineProps<{ variant?: 'plain' | 'boxed' }>();
// ✘ — `type` collides with the native HTML attribute (Naive UI had to invent attr-type, Element Plus native-type)
defineProps<{ type?: 'plain' | 'boxed'; style?: 'light' | 'dark' }>();
```

A pre-existing color-tier axis that was misnamed `variant` is `colorScheme` (e.g. `CaptionFlag`, `LoadingSpinner`).

## Heading / title / description — do **not** mass-rename

House style for a primary heading is `heading` and for body text is `description`, but the existing surface is mostly accepted as-is:

- **Keep existing `title` props.** Do not blanket-rename `title` → `heading`. Only surgical renames where there is a clear win, decided per component (only a few upstream components moved, e.g. `EmptyStateCart`, `StatusMessage`, `FilterOffCanvasSwitchItem`).
- **Keep existing `description` props.** No migration of `description` → `subline`.
- **Keep existing `heading` props.** No rename.
- Native/ARIA-driven `title` (e.g. `Iframe.title`, Reka `<DialogTitle>`) stays.

When a rename *is* warranted, read every affected component first (never rename from a grep hit list), then touch every template, type import, and schema string in one coordinated commit.

## Related

- [`boolean-prop-naming.md`](./boolean-prop-naming.md) — the `is*` rule
- [`component-state-contract.md`](./component-state-contract.md) — v-model, validation, field context
- [`sub-part-tag-prop-naming.md`](./sub-part-tag-prop-naming.md) — `<part>As` for tag-controlling props
