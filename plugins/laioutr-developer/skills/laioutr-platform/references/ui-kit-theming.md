# Theming, Icons & Translations

## Themes

**Available:** classic, laioutr, strawberry-field, sunny, tech

Define via `defineTheme()` / `extendTheme()` in `src/runtime/app/theme/*.ts`. Themes can inherit.

**Dark mode** is automatic via tokens—no explicit dark styles needed.

## Icons

```vue
<Icon name="category/icon-name" size="m" />
```

**Props:** `name: IconName | string`, `size?: 's' | 'sm' | 'm' | 'l'`

## Images

Never import raw paths. Use:

- `themeMedia(path)` - Same image mobile/desktop
- `themeResponsiveMedia(path)` - Separate `<path>-mobile.<ext>` and `<path>-desktop.<ext>`

Images are always rendered using the `Media.vue` component.

## Surface Tone

Use `OnSurface` + `useSurfaceTone()` for surface-aware components.

Values: `'dark'` | `'light'` | `'bright'`

Useful if e.g. a banner has a dark background, so texts are white.

The `OnSurface` component provides tone context and applies the appropriate CSS classes (`on-dark`, `on-light`, `on-bright`).

See [`../surface-tone.md`](../surface-tone.md) for the full propagation model, the **paint → declare** rule, multi-region components, and forbidden patterns.

## Theme resources

The current `IconName` / `ImageSet` types and the per-theme maps are exported from `@laioutr-core/ui-kit`. Import the type to let TypeScript narrow valid icon and image keys at the call site.
