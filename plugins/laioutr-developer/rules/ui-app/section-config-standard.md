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
those live in the public docs.

**Structure / sidebar layout — Figma file `QgRgNtTxBTCAxpTe1rriHM`:**

- Widget specs — page "Side bar fields" (`887:101495`)
- Section specs — page "Sections" (`865:101215`)
- Block specs — page "Blocks" (`865:101216`)

**Field-type mechanics — public docs:**

- [Section Definitions](https://docs.laioutr.io/apps/app-development/section-definitions)
- [Block Definitions](https://docs.laioutr.io/apps/app-development/block-definitions)
- [Schema Fields](https://docs.laioutr.io/apps/app-development/schema-fields)

See also [`CLAUDE.md`](CLAUDE.md) in this folder for a layer-level summary of the same material.

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
6. **Rules** — data-binding and visibility conditions. First-class panel per
   UX spec. Currently no shared preset or field type in code; treat as a
   documented gap (see §6). Where Rules exist today, they are modelled
   ad-hoc via `visibility` decorators and `checkbox` toggles — these
   should be consolidated under a single top-level `Rules` fieldset.

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

**Renames enforced by this standard** (existing components drift from the
canonical name — Phase 2 will normalize):

- `backgroundColor`, `customBackground` (as variant selector) → `background`
- `blockMargin` → `margin`
- `innerBlockPadding` → `padding`
- `containerStyle`, `accordionStyle`, and any other `<component>Style` used as
  the component's top-level variant selector → `variant`. (The bare `style`
  is forbidden outright — see `forbidden-field-names.md`.)
- Per-field ad-hoc `imageSizeMode`, `aspectRatio` live in Design → Layout
  with their own names; they are not re-aliased.

## 4. Shared-field presets

Presets live in your module's `src/runtime/app/shared-fields/` directory
(re-using factories and option lists from `@laioutr-app/ui` where
appropriate). The standard mandates use of the preset wherever the concept
is configurable.

### Existing presets — keep

- `button.ts` — canonical button fieldset (`buttonFields`). Use for every
  `cta` / button `object` field. Imports `buttonVariantOptions` from
  `buttonVariant.ts` to expose the full `ButtonVariant` set.
- `overlay.ts` — canonical overlay object.
- `productTileQuery.ts`, `productDetailQuery.ts`,
  `blogPostCollectionWithPostsQuery.ts`, `nodeFlag.ts` — domain-specific
  query presets. Use as-is.

### Existing presets — replace in Phase 2

- `backdrop.ts` — bundles `background`, `customBackground`, `blockMargin`,
  `innerBlockPadding` in a single "Backdrop" fieldset. This violates the
  group split (Styling vs. Layout) and uses non-canonical names
  (`blockMargin`, `innerBlockPadding`). Replace with two new presets (see
  below).

### New presets to be added in Phase 2

- `shared-fields/background.ts` — `background` (select) + `customBackground`
  (color). Belongs in Design → Styling.
- `shared-fields/margin.ts` — `margin` (`select`, `none|s|m|l`). Belongs
  in Design → Layout.
- `shared-fields/padding.ts` — `padding` (`select`, `none|s|m|l`). Belongs
  in Design → Layout.
- `shared-fields/size.ts` — `sizeField` (`select`, `s|m|l`) + `sizeOptions`
  (shared options array) for non-spacing size selects (text size, rating
  size, banner size, …). Labels are `S`, `M`, `L`.
- `shared-fields/visibility.ts` — `visibilityField({ for, name?, label?, default? })`
  factory for `type: 'checkbox'` + `as: 'visibility'` decorators. `name`
  defaults to `${for}Visible`. Always use this for visibility toggles.
- `shared-fields/buttonVariant.ts` — `buttonVariantOptions`: canonical list
  of all `ButtonVariant` values as select options. Consumed by
  `shared-fields/button.ts` and directly by component schemas that model
  button-variant selects.
- `shared-fields/colorSchemes.ts` — `colorSchemes`:
  canonical option list for themed background-color selects (`grey`,
  `pale`, `dark-background-color`, `bright-background-color`, `custom`).
  Use whenever a section/block exposes a select (or `toggle_button`)
  for the platform's themed background palette. Consumers may use the
  list as-is, filter to a subset, or extend it (e.g. prefix with
  `{ label: 'Default', value: 'default' }`).
- `shared-fields/responsiveBackground.ts` — background with
  `mobile` / `desktop` overrides (Widget `925:105539`).
- `shared-fields/captionStyle.ts` — caption styling preset for
  `as: 'style'` attachment (Widget `917:103219`).
- `shared-fields/rules.ts` — Rules panel (see §6 for gap discussion).

## 5. Naming rules

- `name` props are `camelCase`.
- One concept → one canonical `name` across the entire codebase (see §3).
- Qualify with a prefix only when a component legitimately has multiple
  instances of the same concept (e.g. `headingStyle`, `sublineStyle` when
  both `heading` and `subline` have their own style object).
- Visibility decorators: `<fieldName>Visible`, `as: 'visibility'`.
- Style decorators: `<fieldName>Style`, `as: 'style'`.

## 6. Gaps and follow-ups

The following concepts are required by the standard but not yet fully
supported in code. Phase 2 will address each with a concrete ticket.

- **Rules panel** — no shared preset, no dedicated field type. Visibility
  rules exist ad-hoc via the `visibility` decorator. Decision required
  (UX + Engineering): introduce `shared-fields/rules.ts` aggregator vs.
  new `rules` field type. Standard already pins the panel position
  (directly after "Section info & settings").
- **2D alignment (3×3 grid)** — `content_alignment` field type exists
  (exported from `@laioutr-core/core-types` under the `fields` namespace).
  Verify Studio renders it as a 3×3 grid (Widget `892:102556`); if not,
  this is a platform-side follow-up.
- **Responsive background** — Widget `925:105539` shows mobile/desktop
  breakpoint overrides. Not modelled today; requires
  `shared-fields/responsiveBackground.ts`.
- **Caption styling preset** — used inline today (e.g. `BlockTestimonial`
  via ad-hoc `as: 'style'`). Consolidate into
  `shared-fields/captionStyle.ts`.
- **Block header chrome** — Figma block specs show a "block header" panel
  with layer handle, hide, delete icons. This is Studio UI chrome, not
  part of the definition schema. **Out of scope** for this standard.

## 7. Out of scope

- Unified default values across sections/blocks.
- Implementation of the 22 Checkout/Cart/Form blocks in Figma "Blocks" that
  do not yet exist in code.
- Studio UI changes (e.g. 3×3 alignment grid renderer, dedicated Rules
  panel renderer, responsive background UI) — owned by the platform team,
  not by app authors.
- Mapping individual Figma sections/blocks to specific Vue components. The
  standard is global; per-component reconciliation happens in Phase 2.
