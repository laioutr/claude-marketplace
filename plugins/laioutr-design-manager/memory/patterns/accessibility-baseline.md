# Accessibility baseline — WCAG 2.1 AA

> The hard gate for every design. `design-critique` checks against it. Standard: **WCAG 2.1 AA**.

---

## Checklist (AA)

| Criterion | Requirement | Quick check |
|---|---|---|
| Text contrast | ≥ 4.5:1 | normal text on surface |
| Large text / UI | ≥ 3:1 | ≥ 24px or ≥ 18.66px bold; UI component borders |
| Visible focus | visible indicator | all interactive elements, ≥ 3:1 against the background |
| Keyboard | fully operable | logical tab order, no trap |
| Touch target | ≥ 44×44px | buttons, links, inputs |
| Heading hierarchy | one H1, no skips | clean H1→H2→H3 |
| Input labels | every field has a label | not just a placeholder |
| Non-colour signals | status not conveyed by colour alone | add icon/text |
| Motion | reduce-motion respected | no forced animation |

## Token contrast matrix (default scheme, project theme)

> Fill in for your project: resolve the text/background token pairs from the project copy of `../design-system/color-scheme.md` / `tokens.md` and record the ratios here (or in the project copy under `.claude/design-manager/patterns/accessibility-baseline.md`).

| Foreground | Background | Ratio (approx.) | AA status |
|---|---|---|---|
| `text/heading` / `text/body` light `_TBD_` | `background/default` `_TBD_` | _TBD_ | _TBD_ |
| `text/subline` light `_TBD_` | `background/default` `_TBD_` | _TBD_ | _TBD_ |
| `text/caption` light `_TBD_` | `background/default` `_TBD_` | _TBD_ | _TBD_ |
| `text/heading` / `text/body` dark `_TBD_` | `background/default` dark `_TBD_` | _TBD_ | _TBD_ |
| `text/subline` dark `_TBD_` | `background/default` dark `_TBD_` | _TBD_ | _TBD_ |
| `text/caption` dark `_TBD_` | `background/default` dark `_TBD_` | _TBD_ | _TBD_ |

> Accent caption colours often sit just under 4.5:1 — reserve them for large text/UI, not body copy. Also check the `pale` / `solid` / `muted` scheme pairs once their text values are confirmed.

## Note: EU / EAA

WCAG 2.1 AA is the chosen baseline. The European Accessibility Act (since mid-2025) makes digital accessibility mandatory for many B2B/B2C offerings — raise to WCAG 2.2 AA later if needed (project decision).
