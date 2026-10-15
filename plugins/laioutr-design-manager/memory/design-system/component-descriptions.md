# Component-description convention (Figma `description` field)

> Pattern for the **Figma component description field** (Assets panel / Dev Mode) of components in the project's design-system file. Platform knowledge, read-only.

## Hard rules

- **Language: English.** Always, regardless of the chat language — the design-system layer (component and variable names) is English throughout, so the description stays consistent.
- **Inspect before writing:** run `get_metadata` + `get_screenshot` + `get_variable_defs` on the node to know its type (COMPONENT_SET vs. COMPONENT), its variants/states and the variables actually bound. **Never invent values or behaviour** — if information is missing, ask the user.
- **Set it on the component set** (`node.description` via `use_figma`; load the `/figma-use` skill first). Then **read it back and verify**.
- **Reference tokens by variable path** (e.g. `Organism/<Family>/IconButton/*`), not by hex/px. Mark typography and sizes as **theme-dependent**. Make color-scheme awareness explicit (e.g. "fixed white-on-media, not colour-scheme-aware"; adaptive vs. `always-*` tokens, see `color-scheme.md`).

## Schema (sections in this order, only what applies)

1. **Lead** — one sentence: what it is + where it lives (context / parent organism).
2. **Variants / Responsive** — variant axes or breakpoints and what changes per variant.
3. **Behaviour** — interaction/trigger logic. Say whether the whole component or only a part is the hit area; what a click triggers (e.g. an off-canvas sheet, not an inline expand).
4. **Content / format logic** — if needed, e.g. number formatting, label on/off, icon swap, media sizing.
5. **Scroll & navigation** — if scrollable: direction, snap behaviour, mobile vs. desktop gestures, additional button navigation.
6. **States** — Default · Hover · Pressed · Focus (focus-ring values); note "interactive → Focus required for keyboard".
7. **Layering / footprint** — only for overlays/full-screen: area (100 % w/h), what it overlays, what may overlay it.
8. **Use for / Avoid for** — short usage limits (optional for small atoms).
9. **Tokens** — bound variable group(s), theme dependence, legibility means (drop shadow / overlay gradient), scheme awareness.

## Generic lessons (examples)

- **Overlays and full-screen components:** state the footprint (e.g. "occupies 100 % w/h of the screen, above header/footer") and name exactly what may overlay it (e.g. "only the detail sheet may overlay it").
- **Foreground on media:** if the foreground is fixed white-on-media and not scheme-aware, say so, name the legibility means (drop shadow, overlay gradient) and always add "ensure contrast in the media behind it".
- **Device-dependent behaviour:** when position or gesture changes per device, describe both (e.g. "mobile: bottom sheet from below; desktop: side panel from the right — same timing, only the axis changes") and reference the motion tokens by path instead of hard-coding durations.
- **Format logic:** spell out thresholds and notation precisely (e.g. "raw number up to 9,999, abbreviated from 10,000"), so developers can implement it without guessing.

## Example skeleton

```
<Lead: what it is + where it lives>

Variants: • A — … • B — …
Behaviour: <whole component is the toggle / opens off-canvas sheet / …>
States: Default · Hover · Pressed · Focus (2px ring, 8px radius). Interactive → Focus required for keyboard.
Layering: occupies 100% w/h, above header/footer; only the … sheet may overlay it.
Tokens: bound to Organism/<Family>/* (typography theme-dependent). Fixed white-on-media, drop shadow for legibility — ensure contrast behind it.
```
