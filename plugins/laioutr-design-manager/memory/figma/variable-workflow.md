# Figma variable workflow — hard operating rule

> **Non-negotiable.** This plugin works in Figma **exclusively with variables** — never with design styles (color styles, text styles, effect styles). Platform knowledge, read-only.

---

## The rules

0. **Only `colors` holds primitives (raw hex).** In the project's design-system file the only color primitives live in the `colors` collection (modes light/dark). `color schemes` (default/pale/solid/muted) and `theme` (laioutr/classic/tech/sunny) are **pure alias layers — never hex**. A new color value is **first created as a primitive in `colors`**, then aliased. Details: `../design-system/color-scheme.md`.
1. **Only variables, no styles.** Colors, typography, spacing, radius, shadow, z-index — everything as a **Figma variable**. Never create or use color/text/effect styles.
2. **The outcome is always a variable in the library.** Every design result changes or creates variables in the **variable library of the project's design-system file** (see the project copy of `files.md`). Values are never hard-coded ad hoc on a layer.
3. **Variables reference variables (aliasing).** Semantic variables (`color schemes`) point through the `theme` layer to `colors` primitives — e.g. `color schemes › text/heading → theme › color scheme/default/text/heading → colors › <primitive>`. No raw value in a semantic variable. This keeps the library consistent and themeable (modes).
4. **Bound variables on the layer — EVERYTHING, without exception.** **Every** property of every built element is **bound** to a variable, not typed — **nothing** stays hard-coded. This explicitly includes:
   - **Fill / stroke / radius / spacing / size** → tokens (generic text/surface/icon colors via `color schemes`; component-specific colors via the component's own `theme › <component>/<variant>/<prop>` tokens; never `colors` primitives or hex directly — see `../design-system/color-scheme.md`).
   - **Typography:** `fontFamily`, `fontStyle` (= weight) **and** `fontSize` of every text node → variables (`font/family/<role>`, `font/weight/<role>` or `font weight/<400…800>`, `font size/<scale>` or component-namespaced such as `Avatar/font-size/<size>`).
   - **Icon instances:** `icon-set` property → the icon-set theming variable (`global/icon-set`); `size` property → the matching size variable in the `theme` collection (if none exists → create one with proper naming, e.g. `badge/icon-size`, `Avatar/icon-size`).
   - If no suitable variable exists, it is **created** (correct collection/scale, correct naming) — never hard-coded.
5. **No new raw values outside the scale.** If a design needs a value that does not exist yet: first check whether an existing variable fits; otherwise create a new variable **in the correct collection/scale** (color primitives in `colors`, then alias semantically) — via `design-system-governance`, never hard-coded.
6. **Component-set layout convention (mandatory):** every new COMPONENT_SET immediately gets the uniform frame look (solid fill `#E9E9E9`, stroke set to `visible = false`, `dashPattern = [10,5]`, `cornerRadius = 5`, `layoutMode = NONE`, no padding/spacing) and stacks its variants **vertically** (no overlap at x=0/y=0). Take a screenshot before committing — all variants must be visible. Deviation = build error.

## Why

- **Themeability & modes:** aliasing enables theme × color scheme × light/dark × mobile/desktop without touching layers.
- **Sync with the UI kit:** variables map 1:1 to `@laioutr-core/ui-tokens` (CSS custom properties). Styles do not map cleanly.
- **Migration:** when restructuring, a pure variable library is the only reliable source.

## In practice (MCP)

- **Reliable reading = Plugin API via `use_figma` (read-only):** `figma.variables.getLocalVariableCollectionsAsync()` (collections + modes) and `getLocalVariablesAsync()` (`valuesByMode` + aliases). Works through the hosted connector. `get_variable_defs`, in contrast, only resolves the appearance axis (blind to the scheme axis) — see `files.md`.
- Before building: probe the current state (understand collections, modes, aliasing structure).
- While building (`use_figma`, load `/figma-use` first): bind variables, create new primitives in `colors`, alias semantically through `theme` to `colors`.
- After building: record changed/created variables in the project copies of `../design-system/tokens.md` (and of `migration.md` where relevant) — never in the plugin directory.

## Icon instance — mandatory bindings (pattern)

Icon instances in **every** component that embeds an icon atom (Button, IconButton, Badge, BadgePromotion, Avatar, …) **MUST** keep their properties bound to variables — a theming requirement, never hard-coded:

- **`icon-set` property** → bound to the icon-set theming variable **`global/icon-set`** (external icon library; resolves e.g. to `"solar"`). Bind via `instance.setProperties({ 'icon-set': figma.variables.createVariableAlias(variable) })`. Without the binding, theming breaks.
- **`size` property** → bound to the icon-size variable matching the button/component size (local `theme` collection), e.g. `buttons/size/<xs|s|m|l>/icon-size`. A size may intentionally map to a smaller icon (e.g. an `l` button → `m` icon) — keep that in the variable, not on the instance.
- The binding sits at the **`componentProperties` level** of the icon INSTANCE (not on the surrounding frame).
- Verify after building: 0 hard-coded values, 0 unbound icon instances across all variants.
- Applies to every new component that embeds an icon atom.

## Typography — mandatory bindings (pattern)

Every text node of a component binds **all three** axes to variables (otherwise no theming + a hard-code violation):

- **`fontFamily`** → `font/family/<role>` (theme-aware; e.g. each theme maps `font/family/<role>` to its own family).
- **`fontStyle`** (= weight) → the theme-aware role variable `font/weight/<role>` **if** its style fits, otherwise the exact numeric variable `font weight/<400=Regular|500=Medium|600=SemiBold|700=Bold|800=ExtraBold>`. When binding a family/style variable, **`loadFontAsync` all theme font combinations up front** (otherwise you get an "unloaded font" error).
- **`fontSize`** → `font size/<XS…4XL>` (typography) or a component-namespaced FLOAT (`Avatar/font-size/<size>`) if the standard scale does not fit → create it.
- **Mechanics:** `textNode.setBoundVariable('fontFamily' | 'fontStyle' | 'fontSize', variable)`. Verify afterwards that every text node has family + style + size bound.

## Anti-patterns

- ❌ Creating or applying a color style / text style.
- ❌ Typing hex/pixels directly on a layer.
- ❌ **Raw hex in `color schemes` or `theme`** (alias layers) — primitives belong in `colors`.
- ❌ Filling a semantic variable with a raw value when a matching primitive exists (no alias = error).
- ❌ A one-off value outside the scale.
- ❌ Hard-coding the `icon-set` or `size` property of an icon instance instead of binding it (breaks theming).
- ❌ **Leaving `fontFamily`, `fontStyle` (weight) or `fontSize` of a text node unbound** — all three must be bound to variables.
- ❌ Leaving any property unbound/typed — **always bind everything to variables**.
