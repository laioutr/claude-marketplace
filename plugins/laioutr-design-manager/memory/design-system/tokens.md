# Design tokens — single source of truth

> Template. Project copy lives at `.claude/design-manager/design-system/tokens.md`; fill it for your project.

> Token architecture of the project's design-system file (see the project copy of `memory/figma/files.md`). Read it from the variable collections via the Plugin API (`use_figma`), not from memory. UI kit / Storybook names lead (see `migration.md`).

> ## ⚠️ Primitive rule (non-negotiable)
> **Only the `colors` collection contains color primitives (raw hex).** `color schemes` and `theme` are **alias layers** — never hex. Every new color value is first created as a primitive in `colors`, then aliased. Details: `color-scheme.md`.

---

## Variable collections

| Collection | Modes | Content |
|---|---|---|
| `colors` | light, dark | **Primitives** (raw hex): `bright colors/<hue>/1…12` (+`…alpha`), `default/white`, `default/black`, `overlays/*` — _TBD_ |
| `theme` | laioutr, classic, tech, sunny | Brand-theme layer (alias) — incl. `color scheme/<scheme>/…` and component tokens — _TBD_ |
| `color schemes` | default, pale, solid, muted | Semantic surface tokens (alias) + STRING variant selectors — _TBD_ |
| `sizing & spacing` | mobile, desktop | Responsive spacing — _TBD_ |
| `typography` | mobile, desktop | Responsive typography (composite tokens) — _TBD_ |
| `radius` | single mode | Radius scale — _TBD_ |
| `effects` | single mode | Shadows / effects — _TBD_ |
| `breakpoints` | single mode | Breakpoint values — _TBD_ |
| `motion` | single mode | Motion tokens (duration FLOAT, easing as cubic-bezier STRING), if used — _TBD_ |

Theming axes: **theme × color scheme × appearance (light/dark) × viewport (mobile/desktop)**. Full logic: `color-scheme.md`.

Fill in: which brand theme(s) the project uses, and any project-specific modes added.

---

## Color (semantic — alias, never hex)

Generic text/surface/icon colors bind the `color schemes` tokens; component-specific colors bind `theme › <component>/<variant>/<prop>`. Resolved reference values — **for contrast checks only**:

| Token | default light | default dark | Notes |
|---|---|---|---|
| `text/heading` · `text/body` | _TBD_ | _TBD_ | |
| `text/subline` | _TBD_ | _TBD_ | |
| `text/caption` | _TBD_ | _TBD_ | |
| `background/section` | _TBD_ | _TBD_ | |

Variant selectors in `color schemes` (scheme → variant):

| Selector | default | pale | solid | muted | Rationale |
|---|---|---|---|---|---|
| `button/default` | _TBD_ | _TBD_ | _TBD_ | _TBD_ | _TBD_ |

## Typography (collection `typography`, modes mobile/desktop)

Composite token `<role>/<size>` (e.g. `heading/M`) = alias of `font/family/<role>` + `font/weight/<role>` + `font size/<size>`.

| Example | Composition |
|---|---|
| `heading/M` | family _TBD_, weight _TBD_, `font size/M` = _TBD_ |
| `body/S` | family _TBD_, weight _TBD_, `font size/S` = _TBD_ |
| Primitives | `font/family/*`, `font/weight/*`, `font size/XS\|S\|SM\|M\|…`, `paragraph spacing/*` |

## Spacing / radius (often component-namespaced)

Fill in: the spacing scale (`Spacing/*`), the `radius` scale, and component-namespaced tokens (e.g. `buttons/size/<s>/padding-*`, `buttons/size/<s>/min-height`, `<Component>/border-radius`) — _TBD_.

## Component tokens

| Component | Namespace (`theme`) | Scheme-aware? | Notes |
|---|---|---|---|
| _TBD_ | _TBD_ | _TBD_ | _TBD_ |

## Motion (collection `motion`, if used)

| Variable | Type | Value | Meaning |
|---|---|---|---|
| _TBD_ | FLOAT / STRING | _TBD_ | _TBD_ |

---

## Naming vocabulary (canonical)

Component-property axes: `Variant`, `Size`, `State`, `color`, `icon-set`. **Forbidden bare field names** (section/block convention): `style`, `class`, `key`, `ref`, `is`, `slot`, `slots`, `refFor`, `refKey`.
Token patterns: `color schemes` → `text/*`, `background/*`, `icon/*` (+ `<comp>/default` STRING selectors); `theme` → `color scheme/<scheme>/…` and `<component>/<variant>/<prop>`; `colors` → `<palette>/<step>` (primitives).

> Detailed rules: `laioutr-developer` plugin, skill `writing-section-block-definitions` (authoritative for reserved field names). Reading: `use_figma` → `getLocalVariableCollectionsAsync()` / `getLocalVariablesAsync()`.
