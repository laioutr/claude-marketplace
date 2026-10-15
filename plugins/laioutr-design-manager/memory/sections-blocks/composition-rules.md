# Composition rules — how sections, blocks & data work together

> The rule set the `ux-designer` uses to build pages. Source: Laioutr docs (`/apps/app-development/section-definitions`, `/block-definitions`, `/apps/app-development/schema-fields`, `/frontend/features/pagetypes`) + the Storybook inventory (`catalog.md`).

---

## 1) Sections, blocks, slots — the nesting model

- A **section** is a top-level page building block. It has **slots** (named insertion points) and a **schema** (sidebar fields in Studio).
- A **block** is a smaller building block that goes **into a section slot** (product card, slide, FAQ item, quote …).
- **Slots** control which blocks are allowed:

| Slot property | Effect |
|---|---|
| `restrictTo: [...]` | **Only** these blocks may go into the slot. |
| `allow: [...]` | Allows `isStandalone:false` blocks in this slot. |
| `prefer: [...]` | These blocks appear **first** in the Studio block picker. |
| (no slot / `slots: []`) | The section takes **no** child blocks. |

- **`isStandalone`** on the block:
  - `true` (default): the block can be placed anywhere, as long as a slot doesn't exclude it via `restrictTo`.
  - `false`: the block can only be used in slots that explicitly list it in `allow` (e.g. `BlockTestimonial` only in `SectionTestimonialCarousel`).

**Rule for the UX designer:** never put a block into a slot that doesn't allow it. When in doubt, check Storybook / the definition.

## 2) Well-known tags (Studio categories)

Sections/blocks carry tags for the Studio picker. Canonical tags:

`Banner`, `Brands`, `Blog`, `Checkout & Cart`, `Content`, `Customer Relations`, `Featuring Products`, `Footer`, `Grids`, `Header`, `Heroes`, `Navigation`, `Products`, `Product Detail Page`, `Product Listing Page`, `Testimonials`, `Sliders`, `Blank Containers`

→ Tags let the UX designer quickly find the right section for a purpose (e.g. "Heroes" for the page entry, "Featuring Products" for product highlights).

## 3) Filling data — schema field types

Sections/blocks are filled through their **schema**. Field types (from `schema-fields`):

| Field | Data | Translatable |
|---|---|---|
| `text`, `textarea` | String | ✅ |
| `richtext` | HTML string (render with `RichContent`) | ✅ |
| `link` | Link object (reference/url/anchor/page/pageType) | ✅ |
| `media` | Media object (image/video, responsive, alt) | ✅ |
| `number` | Number | ❌ |
| `checkbox` | Boolean | ❌ |
| `select`/`radio`/`toggle_button` | Option value | ❌ |
| `icon` | Icon name | ❌ |
| `color` | Theme colour/custom | ❌ |
| `object` | Nested fields | ❌ |
| `array` | List (max, labelSingular) | ❌ |
| `query` | **Entity data from Orchestr** | ❌ |
| `json` | Raw JSON | ❌ |

**Most important field for real data: `query`.** Binds a section/block to entities (e.g. `entityType: 'Product'`):
- `singleEntity: true` → one entity (e.g. PDP). Otherwise → `ClientEntitySet` with `entities[]` (e.g. product slider).
- `components: [...]` → which entity data slices (ProductBase, ProductPrices, ProductMedia …).
- `links: { ... }` → linked entities (e.g. product variants), with `limit`.
- **Fetch sparingly** (only the components/limit you need) — every link costs render time.

**Rule:** real product/category/blog data comes via `query`, never hardcoded. Static marketing copy via `text`/`richtext`/`media`/`link`.

## 4) Defaults & fallbacks

- `default` is applied **once** on creation. After that, an empty field falls back to the **type-specific fallback** (`text`→`''`, `select`→first option, `media/link/number/color/icon/query`→`undefined`, `array`→`[]`).
- Mind the consequence: an empty `media`/`link` is `undefined` → needs a guard in the template. For the UX designer this means: **actively fill** relevant fields, don't rely on the fallback.

## 5) Multilingual

Fill translatable fields (`text`, `textarea`, `richtext`, `link`, `media`) **per language** when the page is multilingual. Colour and query settings are shared across languages. **Caution:** layout props (`alignment`, `margin`, `padding`, `columnsDesktop`, …) are `translatable: false` in the schema but are evaluated per rendered locale — write them for every active locale (see `../design-system/corporate-design.md`).

## 6) Props wizard (section variants)

Some sections have a `propsWizard` (a Studio step before placing) that picks a **preconfigured variant** (e.g. hero "Centered" vs. "Split"). The UX designer picks the variant that fits the purpose.

---

## Composition heuristic (funnel logic)

A good page follows a reading/conversion logic, not "every section that fits":

1. **Attention** — hero/banner (tags: Heroes, Banner) with a clear value statement.
2. **Benefit** — image-and-content / BenefitsBox: what's in it for me?
3. **Proof** — testimonials/QuoteCard/reviews/brands: why believe it?
4. **Offer** — featuring products / plan cards / category grids.
5. **Action** — one clear CTA (newsletter, contact, purchase). On landing pages: **exactly one** primary conversion action.
