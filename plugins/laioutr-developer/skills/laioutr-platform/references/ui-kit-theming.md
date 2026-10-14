# Theming, Icons & Translations

## Themes

**Available upstream:** classic, laioutr, sunny, tech

Define your own with `defineTheme()` / `extendTheme()` (ui-kit auto-imports) in your module, typically `src/runtime/app/theme/*.ts`. Themes can inherit.

**Dark mode** is automatic via tokens—no explicit dark styles needed.

## Icons

```vue
<LIcon name="category/icon-name" size="m" />
```

**Props:** `name: IconName | string`, `size?: 's' | 'sm' | 'm' | 'l'`

ui-kit components are auto-imported with the `L` prefix (`LIcon`, `LMedia`, `LButton`). A bare `<Icon>` only resolves with an explicit import from `#ui-kit/components/Icon/Icon.vue`.

## Images

Never import raw paths. Register themable images with:

- `themeMedia(path, width, height, fallbackColor, extension = 'png', focalPoint?)` — same image on mobile and desktop, served from `/img/<path>.<extension>`
- `themeResponsiveMedia(path, mobileWidth, mobileHeight, desktopWidth, desktopHeight, fallbackColor, extension = 'png', mobileFocalPoint?, desktopFocalPoint?)` — separate `/img/<path>-mobile.<ext>` and `/img/<path>-desktop.<ext>`

`fallbackColor` is the solid placeholder shown while the image loads. Read a registered image with `useTheme().image('<key>')`.

Images are always rendered using the `LMedia` component.

## Surface Tone

Use `OnSurface` + `useSurfaceTone()` for surface-aware components.

Values: `'dark'` | `'light'` | `'bright'`

Useful if e.g. a banner has a dark background, so texts are white.

The `OnSurface` component provides tone context and applies the appropriate CSS classes (`on-dark`, `on-light`, `on-bright`).

See [`surface-tone.md`](./surface-tone.md) for the full propagation model, the **paint → declare** rule, multi-region components, and forbidden patterns.

## Theme resources

The `IconName`, `ImageName` and `ImageSet` types come from `#ui-kit/types` (`import type { IconName } from '#ui-kit/types'`). Import the type to let TypeScript narrow valid icon and image keys at the call site.
