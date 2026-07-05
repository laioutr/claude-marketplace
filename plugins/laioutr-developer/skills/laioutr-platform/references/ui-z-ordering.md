# Z-Ordering

Reference: [docs.laioutr.com/laioutr-ui/getting-started/z-ordering](https://docs.laioutr.com/laioutr-ui/getting-started/z-ordering).

Use tokens, never raw z-index values, for anything that paints across sections:

| Token               | Value  | Use case                                       |
| ------------------- | ------ | ---------------------------------------------- |
| `--z-index-sticky`  | `100`  | Sticky headers, fixed filter bars              |
| `--z-index-modal`   | `1400` | Sheet, dialog, alert dialog (overlay + body)   |
| `--z-index-popover` | `1500` | Dropdowns, select menus, popovers              |
| `--z-index-tooltip` | `1600` | Tooltips                                       |
| `--z-index-toast`   | `1700` | Toast notifications                            |

Raw integers are only OK for **local** stacking inside a section (e.g. slider arrows above a slide) — every section already has `isolation: isolate`, so those values can't leak out.

Portaled content (reka-ui `Dialog`, `Sheet`, `DropdownMenu`, `AlertDialog`) mounts at `<body>` and must use the token scale.

Sticky/fixed sections that must stay visible over later sections need `rendering: { isolate: false }` in their `defineSection` config — otherwise the stacking context traps them. Don't disable isolation for any other reason.

`--z-index-modal` covers both the backdrop and the body; DOM/portal order handles stacking between open modals. Don't invent a separate overlay tier.
