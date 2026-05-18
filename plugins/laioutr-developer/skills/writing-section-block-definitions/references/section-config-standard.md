# Section & Block Config Standard

Single source of truth for the configuration schema of every section and block
your module contributes to the ui-app and ui layers. Governs **structure**
only — sidebar group order, group names, option order within a group,
canonical `name` + field type per concept. **Default values are explicitly
out of scope** and remain as currently configured per section/block.

> Conflict rule: when Figma Sections/Blocks pages disagree with the Widgets
> page, the **Sections/Blocks pages are authoritative**. The Widgets page
> describes how a control is rendered; the Sections/Blocks pages describe
> which controls are used and how they are labelled/ordered.

## Sources of truth

This standard covers **structure, naming, and ordering**. It does not restate
field-type mechanics (prop types, fallbacks, allowed properties per type) —
those live in the public docs:

- [Section Definitions](https://docs.laioutr.io/apps/app-development/section-definitions)
- [Block Definitions](https://docs.laioutr.io/apps/app-development/block-definitions)
- [Schema Fields](https://docs.laioutr.io/apps/app-development/schema-fields)

The official docs MCP server (`laioutr-docs`) indexes these pages — query it directly when you need field-type mechanics.

## 1. Sidebar group order (top to bottom)

Every section and block uses this order. Groups are optional — omit a group
if the component has no options in it — but never reorder them.

1. **Section info & settings** — rendered by Studio from `studio.`\* meta
   (`label`, `description`, `previewSrc`, tags). Not part of `schema`.
2. **Content** — editorial content authored per instance: texts, media,
   CTAs, queries, slots-specific options.
3. **Design** — visual appearance. Contains two sub-groups in this order:
4. **Styling** — Style, Background, Color, Overlay, Caption Style.
5. **Layout** — Margin, Padding, Columns, Rows, Mobile, Desktop,
   Alignment.
6. **Rules** — data-binding and visibility conditions. Modelled today via
   `visibility` decorators and `checkbox` toggles; consolidate under a
   single top-level `Rules` fieldset when your section/block has more
   than one such control.

No other top-level panels. "Behaviour" / "Advanced" / "Settings" panels
seen in legacy code are not part of the standard; their fields belong in
Content (if editorial) or Design (if visual).

## 2. Option order within each group

### 2.1 Rules

Order follows the Figma Sections/Blocks spec:

1. `visibilityCondition` — visibility toggle / conditional rule
2. `dataBinding` — dynamic data source binding (see `query` below)

### 2.2 Content

Follows the natural reading order of the rendered component. Canonical
sequence when multiple apply:

1. `caption`
2. `heading`
3. `subline`
4. `description`
5. `media`
6. `cta`
7. `query` (dynamic data source)
8. `items` / `slots`-specific arrays

### 2.3 Design → Styling

1. `variant`
2. `background`
3. `customBackground` (only when `background` supports a custom variant)
4. `color`
5. `overlay`
6. `captionStyle` (attached `as: 'style'` to caption fields)

### 2.4 Design → Layout

1. `margin`
2. `padding`
3. `columns`
4. `rows`
5. `mobile`
6. `desktop`
7. `alignment`

## 3. Canonical option names & field types

Every section and block that uses these concepts **must** use exactly the
`name` and `type` below. Options-enums are binding; defaults are not.

> **Note:** schema fields have no `required` property. The only base
> properties are `type`, `name`, `label`, `default`, `description` (plus
> type-specific properties like `options`, `placeholder`, `schema`). Do
> **not** add `required: true` — fields without a value use a runtime
> fallback (see [Schema Fields docs](https://docs.laioutr.io/apps/app-development/schema-fields)).

### Content

| Concept             | `name`        | `type`                   | Notes                                       |
| ------------------- | ------------- | ------------------------ | ------------------------------------------- |
| Caption             | `caption`     | `text`                   | —                                           |
| Heading             | `heading`     | `text`                   | —                                           |
| Subline             | `subline`     | `text`                   | —                                           |
| Description         | `description` | `textarea` or `richtext` | Pick per component; name is identical       |
| Media               | `media`       | `media`                  | —                                           |
| CTA button          | `cta`         | `object`                 | Use shared preset `shared-fields/button.ts` |
| Dynamic data source | `query`       | `query`                  | —                                           |
| Link                | `link`        | `link`                   | —                                           |

### Design → Styling

| Concept           | `name`             | `type`                      | Options                                                                                                                                                |
| ----------------- | ------------------ | --------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------ |
| Style variant     | `variant`          | `toggle_button` or `select` | Component-specific enum. The bare `style` is forbidden (see `forbidden-field-names.md`).                                                               |
| Background preset | `background`       | `select`                    | `none`, `pale`, `solid`, `default`                                                                                                                     |
| Custom background | `customBackground` | `color`                     | Only when `background` has a `custom` variant                                                                                                          |
| Foreground color  | `color`            | `color`                     | `allowCustom: true`                                                                                                                                    |
| Overlay           | `overlay`          | `object`                    | Use shared preset `shared-fields/overlay.ts`                                                                                                           |
| Caption styling   | `captionStyle`     | `object` (`as: 'style'`)    | Attached to a caption field; shared preset TBD. Compound `*Style` names like this are allowed; see `forbidden-field-names.md` for the bare-name rules. |

### Design → Layout

| Concept            | `name`      | `type`               | Options                              |
| ------------------ | ----------- | -------------------- | ------------------------------------ |
| Margin             | `margin`    | `select`             | `none`, `s`, `m`, `l`                |
| Padding            | `padding`   | `select`             | `none`, `s`, `m`, `l`                |
| Columns            | `columns`   | `number` or `select` | —                                    |
| Rows               | `rows`      | `number` or `select` | —                                    |
| Mobile breakpoint  | `mobile`    | `object`             | Responsive overrides                 |
| Desktop breakpoint | `desktop`   | `object`             | Responsive overrides                 |
| Alignment (1D)     | `alignment` | `toggle_button`      | `left`, `center`, `right`            |
| Alignment (2D)     | `alignment` | `content_alignment`  | 3×3 grid `top-left` … `bottom-right` |

When refactoring an existing section/block whose field names drift from the canonical list above, normalize them in the same change. Common drifts to fix on contact:

- `backgroundColor`, `customBackground` (used as a variant selector) → `background`
- `blockMargin` → `margin`
- `innerBlockPadding` → `padding`
- `containerStyle`, `accordionStyle`, or any other `<component>Style` used as the top-level variant selector → `variant`. (The bare `style` is forbidden — see [`forbidden-field-names.md`](./forbidden-field-names.md).)
- Per-field `imageSizeMode`, `aspectRatio` live in Design → Layout with their own names; do not re-alias them.

## 4. Shared-field presets

Presets live in your module's `src/runtime/app/shared-fields/` directory. The standard mandates use of the preset wherever the concept is configurable.

Recommended presets in your module's `shared-fields/`:

- **`button.ts`** — canonical button fieldset (`buttonFields`) for every `cta` / button `object` field. Imports `buttonVariantOptions` from `buttonVariant.ts` to expose the full `ButtonVariant` set.
- **`overlay.ts`** — canonical overlay object.
- **`background.ts`** — `background` (select) + `customBackground` (color). Belongs in Design → Styling.
- **`margin.ts`** — `margin` (`select`, `none|s|m|l`). Belongs in Design → Layout.
- **`padding.ts`** — `padding` (`select`, `none|s|m|l`). Belongs in Design → Layout.
- **`size.ts`** — `sizeField` (`select`, `s|m|l`) + `sizeOptions` (shared options array) for non-spacing size selects (text size, rating size, banner size, …). Labels: `S`, `M`, `L`.
- **`visibility.ts`** — `visibilityField({ for, name?, label?, default? })` factory for `type: 'checkbox'` + `as: 'visibility'` decorators. `name` defaults to `${for}Visible`.
- **`buttonVariant.ts`** — `buttonVariantOptions`: canonical list of `ButtonVariant` values as select options.
- **`colorSchemes.ts`** — canonical option list for themed background-color selects (`grey`, `pale`, `dark-background-color`, `bright-background-color`, `custom`). Filter or extend per section/block.
- **Domain-specific query presets** — e.g. `productTileQuery.ts`, `productDetailQuery.ts`, `blogPostCollectionWithPostsQuery.ts`, `nodeFlag.ts`.

## 5. Naming rules

- `name` props are `camelCase`.
- One concept → one canonical `name` across your module (see §3).
- Qualify with a prefix only when a component legitimately has multiple
  instances of the same concept (e.g. `headingStyle`, `sublineStyle` when
  both `heading` and `subline` have their own style object).
- Visibility decorators: `<fieldName>Visible`, `as: 'visibility'`.
- Style decorators: `<fieldName>Style`, `as: 'style'`.

## 6. Out of scope

- Unified default values across sections/blocks.
- Studio UI rendering (e.g. how a 3×3 alignment grid is rendered, how the Rules panel UI looks) — owned by the Laioutr platform, not by app authors.
