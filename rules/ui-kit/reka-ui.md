# reka-ui & Accessibility

## When to Use reka-ui

Prefer reka-ui as a basis when available. It handles keyboard navigation, ARIA, focus management, and screen reader support.

**Primitives to use:** RovingFocus, FocusScope, VisuallyHidden, etc.

## Styling Pattern

1. Apply your own BEM classes to reka-ui components
2. Use data attributes for state: `[data-state="open"]`, `[data-highlighted]`, `[data-disabled]`
3. Use `asChild` prop for full control over rendered element
4. Avoid styling reka-ui's internal class names

```css
.dropdown-menu__item[data-highlighted] {
  /* highlighted */
}
```

## Context Access

Use `injectContext` functions (e.g., `injectAccordionRootContext`) for advanced composition only. Check if context exists before using—may be `undefined` outside scope. Prefer props/events when possible.

## Accessibility Requirements

- Follow **WCAG 2.1** guidelines
- Keyboard navigation required for all interactive elements
- Focus states must be visible
- Disabled states: `cursor: not-allowed` + appropriate token colors
