# TODO FIGMA Marker

Convention for marking provisional design decisions in code so they can be reconciled when Figma delivers the dedicated tokens or specs.

## When to use

Use `TODO FIGMA` when you ship a component before its Figma source of truth exists. Typical cases:

- Reaching for a generic token (`--grey-7`, `--spacing-xs`) because no component-specific token has been defined yet
- Picking a placeholder layout/sizing that Figma will later override
- Leaving `parameters.design.url` empty in a Storybook story because no Figma frame exists

Do **not** use `TODO FIGMA` for code-side TODOs (use plain `TODO:` for those). The marker exists specifically to signal "design owes us an answer".

## Format

In CSS:

```css
.toggle--size-m {
  /* TODO FIGMA: toggle size tokens */
  height: var(--sizing-l);
  padding: 0 var(--spacing-m);
}
```

In Vue templates (rare — prefer CSS):

```vue
<!-- TODO FIGMA: hover-card width spec -->
<RekaHoverCardContent :side-offset="8" />
```

In Storybook stories with no Figma frame yet:

```ts
parameters: {
  design: {
    type: 'figma',
    url: '', // TODO FIGMA
  },
},
```

The marker must be inline above (or next to) the provisional value, not in a file-level comment block. That keeps it grep-discoverable per occurrence.

## Resolution workflow

When Figma delivers:

1. `grep -rn "TODO FIGMA" src` (or wherever your module's source root lives) — list all open markers
2. For each match, replace the generic token / empty URL / placeholder with the Figma-defined value
3. Remove the marker comment
4. Commit per component (don't bundle 30 unrelated component updates in one commit)

## Why not a tracking ticket?

Tickets rot. A `TODO FIGMA` lives next to the affected line, survives every refactor, and can be batch-resolved with one grep. Use a ticket for the broader design dependency, but the inline marker is what guarantees nothing slips.
