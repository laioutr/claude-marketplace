# Color-scheme logic — the theming model

> How color and theming work in a Laioutr design-system Figma file (the project's design-system file, see the project copy of `memory/figma/files.md`). Platform knowledge, read-only. Project-specific values (resolved hex, variable inventories) belong in the project copy of `memory/design-system/tokens.md`, never here.

> ## ⚠️ Hard rule (non-negotiable)
> **The only primitives (raw hex values) live in the `colors` collection.** `color schemes` and `theme` are **pure alias layers** — they **never** contain hex values; they only point (through the chain) to `colors` primitives. When creating or restructuring: not a single raw hex in `color schemes` or `theme`. Any resolved hex you note down (for contrast checks) is only the end point of the alias chain in `colors`, not the token's content.

---

## The variable collections

| Collection | Modes | Role |
|---|---|---|
| `colors` | **light**, dark | **Primitives** — raw palettes (e.g. `bright colors/<hue>/1…12` + `…alpha`, `default/white`, `default/black`, `overlays/*`). Holds the light/dark values. |
| `theme` | **laioutr**, classic, tech, sunny | **Brand-theme layer** — switchable themes, alias-only. Contains, among others, the `color scheme/<scheme>/…` tokens that the `color schemes` collection aliases to, plus all component color tokens. |
| `color schemes` | **default**, pale, solid, muted | **Purely semantic layer** — generic semantics (`background/section`, `background/content`, `text/*`, `icon/default`) + **STRING** variant selectors (`button/default`, …). References `theme`, never holds component color tokens. |
| `sizing & spacing` | **mobile**, desktop | Responsive spacing scale (two breakpoint modes). |
| `typography` | **mobile**, desktop | Responsive typography (composite tokens). |
| `radius` | single mode | Radius scale. |
| `effects` | single mode | Shadows / effects. |
| `breakpoints` | single mode | Breakpoint values. |
| `motion` | single mode | Motion tokens (durations, easing as cubic-bezier STRING), where used. |

### Theming = orthogonal mode axes
**theme** (laioutr/classic/tech/sunny) × **color scheme** (default/pale/solid/muted) × **appearance** (`colors` collection: light/dark) × **viewport** (sizing/typography: mobile/desktop). A component binds semantic tokens; its rendered look results from the active modes of each collection.

---

## The `color schemes` tokens + alias chain

Global semantic tokens: `background/section`, `background/content`, `text/default`, `text/heading`, `text/body`, `text/subline`, `text/caption`, `icon/default`, plus STRING selectors such as `button/default`. Each one aliases, per scheme mode, into the `theme` layer:

```
color schemes › text/heading [default] → theme › color scheme/default/text/heading → colors › <primitive> (light|dark)
                            [pale]    → theme › color scheme/pale/text/heading    → …
                            [solid]   → …  [muted] → …
```

→ Generic text, surface and icon colors are always bound to the `color schemes` tokens (`text/heading`, `background/section`, …). Scheme and appearance are set via modes, not by swapping to other tokens.

## Pattern: how a component follows the color-scheme logic

**The `color schemes` collection is purely semantic and lean.** It does exactly two things — never more:
1. **References** the `theme › color scheme/<scheme>/…` variables (the generic semantic tokens: `background/section`, `background/content`, `text/*`, `icon/default`).
2. **Decides per scheme which component variant** is used — as a **STRING selector** (model: `button/default`).

**Component colors live in `theme`, never in `color schemes`.** A component binds its own `theme › <component>/<variant>/<prop>` tokens directly (brand-theme-aware, and light/dark-aware through `colors`). It gets **no** color tokens of its own in `color schemes`.

> Anti-pattern: creating per-component color tokens in `color schemes` (e.g. `badge/<variant>/{bg,text,icon,border}` plus a `theme › color scheme/*/badge/*` copy per scheme). This bloats the library by a multiple and breaks the semantic layer. Do not do it — use a selector instead.

### The variant selector (canonical pattern `button/default`)

```
color schemes › button/default   [mode = scheme]   (STRING)
   → theme › color scheme/<scheme>/button/default   [mode = brand theme]   (STRING = variant name)
```
Example: the `default` scheme resolves to `primary`; the `solid` scheme resolves to `secondary-white` (or `secondary`, depending on the brand theme). Blocks/sections read the selector and render the matching **variant** of the component. The component is therefore **not** re-colored per surface — the scheme picks the variant.

To make another component scheme-dependent, create **one** such selector (`<comp>/default` STRING in `color schemes` + one `theme › color scheme/<scheme>/<comp>/default` entry per scheme), **not** per-prop color tokens. Document the mapping (scheme → variant) and its rationale, e.g. "on `solid` (brand-saturated) surfaces use the boxed variant for legibility; plain everywhere else".

**Clarification `pale`/`solid`/`muted`:** these are **color schemes** (the surface axis), not component variants.

### Scheme-awareness — the two legitimate mechanisms (NO component color tokens in `color schemes`)
1. **Variant selector** (`<comp>/default` STRING in `color schemes`, model `button/default`) — the surface picks the variant. The selector is semantic (a variant name), not a color assignment.
2. **Generic semantic tokens** — the component binds `color schemes › text/*` / `background/*` / `icon/*` (these already switch per scheme mode and light/dark). A component-namespaced `theme` token may also alias to one of these generic tokens to become scheme-aware. That way scheme-awareness reaches a component **without** component-specific color tokens in `color schemes`.

Components that are intentionally **not** scheme-aware (e.g. buttons, where blocks/sections choose the variant) bind their `theme › <component>/*` tokens directly and say so in their description.

---

## Adaptive vs. constant foreground tokens

There are **adaptive** and **constant** foreground tokens — essential for theming and for any contrast statement:

| Token pattern | Behaviour |
|---|---|
| `text/black` (no `always-`) | **adaptive** — light mode = the theme's black, dark mode = white |
| `text/always-black` | **constant** — the theme's black in light and dark |
| `text/always-white` | **constant** — white in light and dark |
| `overlays/white alpha/*` | adaptive alpha ladder |
| `overlays/always white alpha/*`, `overlays/always black alpha/*` | constant alpha ladders |

> Rule of thumb: `always-*` = constant (context-bound, e.g. only valid on a dark background); without `always-` = adaptive (flips with light/dark). Do not confuse them — mixing them up leads to wrong statements in component descriptions.

**Bridge mechanics:** `theme` has no light/dark axis — that axis lives in `colors`. When a theme-aware token needs light = theme-specific value **and** dark = a different value (e.g. white), create a **bridge primitive** in `colors` that encapsulates the (light, dark) pair, and let the `theme` token alias to the matching bridge per brand theme.

**Consequence for glass variants (e.g. Button / IconButton / Input):**
- `glass-dark` (or `glass-black`): text/icon = `text/always-white` → **constant white** → only works on a **dark** background (otherwise < 3:1). Document it as "dark background only".
- `glass-white`: text/icon = `text/black` → **adaptive** (black in light mode, white in dark mode) → **not** a blanket "dark background only"; it is theme-adaptive. In dark mode on very dark surfaces, check contrast on the real frame because of the semi-transparent white overlay.

---

## How to read variables reliably (Plugin API, works through the hosted connector)

Use `use_figma` (read-only; load `/figma-use` first) instead of `get_variable_defs`:
```js
const cols = await figma.variables.getLocalVariableCollectionsAsync(); // collections + modes
const vars = await figma.variables.getLocalVariablesAsync();           // valuesByMode + aliases
```
For concrete hex values per mode combination: follow the alias chain and choose the target mode per collection (theme = `<brand theme>`, color schemes = `<scheme>`, colors = light/dark). Aliases into an external library resolve via `figma.variables.getVariableByIdAsync(id)` — a purely local resolver will wrongly report them as unresolved. The hosted `get_variable_defs`, in contrast, only resolves the appearance axis — it is blind to the scheme axis. Text tokens resolved via forced modes can differ from the real render; verify contrast on the real frame before relying on it.

## Target rules for building

1. **Only `colors` holds primitives (raw hex).** `color schemes` and `theme` are alias-only — never hex (hard rule, see above).
2. Components never bind `colors` primitives or hex directly. Generic text/surface/icon colors → `color schemes` tokens (`text/*`, `background/*`, `icon/*`); component-specific colors → the component's own `theme › <component>/<variant>/<prop>` tokens.
3. Component colors are **never** added to `color schemes`. Scheme-dependence goes through a STRING variant selector or the generic semantic tokens.
4. A new brand theme = a new mode in `theme`; a new surface tone = a new mode in `color schemes`. Both keep aliasing until they end in `colors`.
5. Responsive spacing/typography via the mobile/desktop modes of the `sizing & spacing` / `typography` collections.
6. Strict aliasing: `color schemes` → `theme` → `colors`. A new color value is **first created as a primitive in `colors`** and then aliased — never written directly into a semantic token.
