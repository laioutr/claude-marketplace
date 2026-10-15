# Recipe: Product Listing Page B2B (PLP B2B)

> **kind: dynamic** — one page per project, route-driven (category slug), loads the category + product set via Orchestr. The section composition is designed by the `ux-designer`; routing and queries are given by the page type.

## Purpose + funnel role

The working page for **buyers with a goal**: whoever opens a B2B listing page usually knows what is needed (article number, standard, dimension, pack size). The page helps with **comparing and narrowing down** and leads to the right item. Ordering happens on the PDP.

Difference from the B2C PLP:

| Aspect | B2C PLP | **B2B PLP** |
|---|---|---|
| View | Image tiles, emotional | **Dense display**: tile + list/table, article number visible |
| Price | Gross, one price | **Net** (VAT incl./excl. stated), customer-specific, **tiered prices** |
| Action | Click on the tile → PDP | Click on the tile → PDP; once per page **request a quote** |
| Filters | Colour, size, price | **Technical attributes** (dimension, material, standard), availability, delivery time |
| Marketing | Promo banners, inspiration | Minimal: at most one notice banner (framework contract, delivery terms) |
| Access | Public | Often **login-gated** (prices/availability only after sign-in) |

## Section order (default composition)

> Check section/block names against `../sections-blocks/catalog.md` (Storybook). There is **no** dedicated PLP section — the listing blocks (`ProductListingGrid`, `FilterBar`, `SortModes`) go into a Container section.

| # | Section | Purpose | Slots → Blocks | Data |
|---|---|---|---|---|
| (0) | TopBar | B2B USPs: delivery terms, contact person, hotline | (USP content) | `text`, `link` — usually **global** |
| (0) | HeaderShop | Navigation, **search by article / customer article number**, cart, account/company switch | MegaMenu / MenuBasic | menu links — usually **global** |
| 1 | Breadcrumbs | Orientation in the (deep) B2B assortment tree | — | from the category path |
| 2 | BannerBasic (slim) / SearchResultHero | Category title + short technical description, **no** image hero | — | `text` heading, `richtext` (category description) |
| 3 | CategoryCardSlider | Subcategories as a quick entry (only if the category has children) | category cards | `query` Category (children) |
| 4 | **Container** — listing | Core of the page: filter, sort, list | **FilterBar** (incl. sorting + result count) → **ProductListingGrid** (→ ProductTileBasic with PriceInfo, QuantityDiscount, **without buttons**) | page query `products` (ProductBase, ProductPrices, ProductMedia + availability); filter/sort/pagination from Orchestr |
| (5) | BannerIntegrated | **Request a quote** / contact person, for needs outside the assortment or large quantities | (CTA) | `text`, `link` → quote request |
| 6 | Footer | Legal, terms, B2B T&Cs | FooterMenu | menu links — usually **global** |

Mobile: `FilterOffCanvas` (feature) instead of expanded filters. Handle empty results with the listing empty state from the UI kit.

> Deliberately **not** on the B2B PLP: PromotionBanner between products, PersonaQuoteSlider, newsletter pop-ups. They disrupt the workflow.

## The product tile in B2B

`BlockProductTileBasic` is the base. What a B2B tile/row must show:

1. **Article number** (plus customer article number, if stored), easy to read, copyable.
2. **Net price** with clear VAT labelling, unit price (`UnitPrice`), pack size / packaging unit.
3. **Tiered prices** → `BlockQuantityDiscount` (data: `{ quantity, savingsPercent }[]`).
4. **Availability + delivery time**, not just "in stock".

**No buttons in the tile.** No `BlockAddToCart`, no quantity field, no order-list trigger. The whole tile is a link to the PDP. Cart, quantity and order list belong on the PDP.

If the UI kit's base tile does not cover points 1 or 4: use a project-wide override of the product tile via Nuxt global component-name shadowing of `LProductTileBasic`. That is the job of the `ui-designer` and `laioutr-developer`, not the `ux-designer`.

## Data binding

- **Page query** (given by the page type): category by slug + the category's product set. Always read pagination totals from `products.pagination.total` (not `products.total`).
- **Components sparingly:** ProductBase, ProductPrices, ProductMedia, plus availability and tiered prices only if the tile shows them. Variant `links` only with a `limit`.
- **Customer-specific prices:** come from the connector in the context of the signed-in organization, not from Studio. Never maintain prices statically.
- **B2B actions (Orchestr, `b2b/*`):** `b2b/quote/request` (request a quote), `b2b/organization/product-number-get-all` (customer article numbers), `b2b/organization/switch` (company switch in the header). Check per project whether the connector provides them (e.g. B2B Sellers for Shopware).
- **Filters:** facets come from Orchestr (`/frontend/orchestr/filters`). Mark technical attributes as filterable in the backend, otherwise they do not appear in the FilterBar.

## Conversion goal

Primary: **find the right item and move to the PDP** (quantity, cart, order list happen there). Secondary, once per page: **request a quote**.

## Page type: variant or own token?

Two options, decided per project:

1. **Page variant of the canonical PLP** (`@laioutr-core/canonical-types/ecommerce`), served to B2B customers. Preferred when B2C and B2B run in the same shop: same route, same queries, own composition.
2. **Own page type** via `definePageTypeToken` (e.g. `b2b/product-listing-page`, `kind: 'dynamic'`, `pathConstraints.default` e.g. `/b2b/c/:slug+`, `resolveFor: [{ referenceType: 'Category' }]`). Needed when B2B requires its own route or its own queries (pure B2B shop, separate assortment). Implementation is done by `laioutr-developer`.

The Cockpit MCP cannot create new pages or variants; a person does that in Studio, then the ux-designer fills the sections.

## Gaps

The B2B package of Laioutr UI is in development per the docs (`/laioutr-ui/b2b`). Not yet in Storybook:

- **List/table view** of the listing (toggle tile ↔ list) → _TBD_
- **Quick order** (SKU input, bulk quantities, CSV import) → _TBD_, today a separate page rather than part of the PLP
- **Net/gross toggle** → _TBD_

Until then, build with the existing blocks and name the gaps in the brief to the `ui-designer` instead of inventing substitute sections.

## Notes

- `dynamic`: one instance per project or per variant. Header, TopBar and Footer are usually global sections.
- Density is welcome, readability is still mandatory: article number and price not below 14 px, contrast WCAG 2.1 AA.
- Desktop-first in usage, responsive is still mandatory. The tile is clickable as a whole surface (touch target ≥ 44 px).
- Before going live: `design-critique` + accessibility check.
