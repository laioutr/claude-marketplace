# No Viewport Stories

Storybook stories in `ui-kit` document **component states and variants via props**. They do **not** document viewport behaviour.

## The rule

Do **not** create stories whose only purpose is to show the component at a different viewport width. Responsive behaviour is verified in Storybook's viewport toolbar, not by duplicating stories.

Forbidden pattern:

```ts
// ❌ Don't do this
export const ViewportXS: Story = {
  name: 'Viewport / xs (360)',
  parameters: { viewport: { defaultViewport: 'xs' } },
};

export const ViewportS: Story = {
  /* ... */
};
export const ViewportSM: Story = {
  /* ... */
};
export const ViewportLG: Story = {
  /* ... */
};
```

Allowed pattern — stories differ in **props / state**:

```ts
// ✅ State and variant stories only
export const Default: Story = {};
export const Disabled: Story = { args: { disabled: true } };
export const WithIcon: Story = { args: { icon: 'essentials/search' } };
export const Loading: Story = { args: { isLoading: true } };
```

## What about responsive behaviour?

- Use Storybook's built-in **viewport addon toolbar** to resize any story on demand.
- If a component truly renders a structurally different layout per breakpoint that cannot be shown by resizing (e.g. different DOM on mobile vs desktop), write **one** dedicated story for that variant and name it after the layout (`MobileDrawer`, `DesktopInline`) — **not** after the viewport width.
- Chromatic snapshots configure viewports at the project level, not per-story.

## Why

- Stories are a **state matrix**, not a **viewport matrix**. State is a prop concern; viewport is an environment concern.
- Per-viewport exports bloat the sidebar and slow Chromatic runs with near-duplicate snapshots.
- Drag-to-resize in the Storybook toolbar already covers exploratory viewport checks.
- Layout bugs that depend on content width are revealed by changing **content**, not the viewport — and content lives in props.

## Retroactive cleanup

When touching a component whose stories file still contains `ViewportXS` / `ViewportS` / `ViewportSM` / `ViewportLG` exports, remove them as part of the change. Do not leave them behind "for now".
