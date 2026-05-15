# Styling Rules

## Tokens

**Never hardcode colors, sizes, or spacing.** Use CSS variables from `@laioutr-core/ui-tokens`.

**Exception:** Animation/transition timings can be hardcoded.

```css
/* Allowed */
transition: all 0.2s ease-in-out;

/* NOT allowed */
padding: 8px; /* Use var(--spacing-sm) */
color: #333; /* Use var(--text-primary) */
border-radius: 4px; /* Use var(--buttons-border-radius) */
```

**Token categories:** `--spacing-*`, `--font-size-*`, `--buttons-*`, `--text-*`, `--sizing-icon-size-*`

**State coverage:** Interactive components must style all states: default, hovered, pressed, focus, disabled.

## CSS Approach

- Use unscoped `<style>` blocks (no `scoped` attribute)
- Follow BEM: `.block`, `.block__element`, `.block--modifier`
- Wrap all component styles in `@layer lui-components`
- Use `@layer lui-overridable` for consumer-overridable styles (lower specificity than lui-components)

**Layers order:** `lui-tokens` → `lui-reset` → `lui-externals` → `lui-global` → `lui-overridable` → `lui-components` → `unocss` → unlayered

## Responsive

Use custom media queries: `@media (--lg)`, `@media (--md)`, etc.

| Name | Min-width |
| ---- | --------- |
| xs   | 360px     |
| s    | 414px     |
| sm   | 600px     |
| md   | 800px     |
| lg   | 1280px    |
| xl   | 1920px    |
| xxl  | 2048px    |
