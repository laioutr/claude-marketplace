# Recipe: Product Listing Page (PLP, B2C)

> **kind: dynamic** — one page per project, route-driven (category slug), loads the category + product set via Orchestr. Canonical page type from `@laioutr-core/canonical-types/ecommerce`. The section composition is designed by the `ux-designer`; routing and queries are given by the page type.
>
> B2B counterpart: `product-listing-page-b2b.md`.

## Purpose + funnel role

The page between entry (home, navigation, campaign) and purchase decision. Whoever lands here roughly knows what they are looking for (category), but not yet which product. The page helps with **browsing, narrowing down and comparing**, and leads to the **PDP** in one click. Buying happens on the PDP, not in the list.

## Section order (default composition)

> Check section/block names against `../sections-blocks/catalog.md` (Storybook). There is **no** dedicated PLP section — the listing blocks (`FilterBar`, `ProductListingGrid`) go into a Container section.

| # | Section | Purpose | Slots → Blocks | Data |
|---|---|---|---|---|
| (0) | TopBar / UspBanner | USPs: shipping, returns, service | BenefitsBox | `text`, `icon` — usually **global** |
| (0) | HeaderShop | Navigation, search, cart | MegaMenu / MenuBasic | menu links — usually **global** |
| 1 | Breadcrumbs | Orientation + the way back to the parent category | — | from the category path |
| 2 | BannerBasic / BrandHero | Category entry: title, short text, one mood image | — | `text` heading, `richtext` (category description), `media` |
| 3 | CategoryCardSlider | Subcategories as a quick entry (only if the category has children) | category cards | `query` Category (children) |
| 4 | **Container** — listing | Core of the page: filter, sort, browse | **FilterBar** (sticky, incl. sorting + result count) → **ProductListingGrid** (→ ProductTileBasic **without buttons**) | page query `products` (ProductBase, ProductPrices, ProductMedia); filter/sort/pagination from Orchestr |
| (5) | PromotionBanner / BannerIntegrated | At most **one** promotion or inspiration banner, below the grid or after the first rows | (CTA) | `text`, `media`, `link` |
| (6) | ProductSliderShowcase / ProductSlider | "Recently viewed" / "Popular in this category" | ProductTileBasic (**without buttons**) | `query` Product, `limit` 8–12 |
| (7) | MediaText | SEO/guide text for the category, at the very bottom | — | `richtext`, `media` |
| 8 | Footer | Legal, service, navigation | FooterMenu | menu links — usually **global** |

Mobile: `FilterOffCanvas` (feature) for deep filtering; the FilterBar opens it. Use `SortModes` on its own only when no FilterBar is used (the FilterBar already includes sorting). Handle empty results with the listing empty state from the UI kit.

> Not every PLP needs 5–7. The grid must start **without scrolling**: keep the hero slim, show subcategories only when there are real children.

## The product tile

`BlockProductTileBasic` in the grid and in sliders. The tile shows:

1. **Image** (optionally a second image on hover/swipe), consistent aspect ratio across the whole grid.
2. **Brand + title** (max. 2 lines).
3. **Price** via `PriceInfo` — strike-through/sale price clearly marked, unit price where legally required (`UnitPrice`).
4. **Flags** (new, sale, sustainable) — at most two per tile.
5. **Colour swatches** if the product has variants.
6. **Rating** (stars + count), if available.

**No buttons in the tile.** No `BlockAddToCart`, no size/quantity selection, no wishlist button. Leave the add-to-cart slot of `ProductTileBasic` empty. The whole tile is a link to the PDP; variant selection, cart and wishlist belong on the PDP. This also applies to product sliders on this page.

If the project needs the tile to look different (e.g. second image, energy label): use a project-wide override of the product tile via Nuxt global component-name shadowing of `LProductTileBasic`. That is the job of the `ui-designer` and `laioutr-developer`, not the `ux-designer`.

## Data binding

- **Page query** (given by the page type): category by slug + the category's product set. Always read pagination totals from `products.pagination.total` (not `products.total`).
- **Components sparingly:** ProductBase, ProductPrices, ProductMedia; swatches/ratings only if the tile shows them. Variant `links` only with a `limit`.
- **Filters:** facets come from Orchestr (`/frontend/orchestr/filters`). Anything not marked as filterable in the backend does not appear in the FilterBar.
- **Editorial** (hero text, banner, SEO text): static via `text`/`richtext`/`media`/`link`, filled per language.
- **Pagination:** "Load more" or page numbers, per project. Infinite scroll only if the footer stays reachable.

## Conversion goal

Primary: **click on a product tile → PDP**. Everything on the page pays into it: narrow down fast, compare well, click clearly. Secondary (optional, once): the one promotion banner.

## Notes

- `dynamic`: one instance per project. Header, TopBar and Footer are usually global sections.
- The same composition often also serves brand and search-result pages; for search, use `SearchResultHero` instead of the category hero.
- `ProductListingGrid` controls its own grid columns — no per-page layout overrides.
- The whole tile is the click target (touch target ≥ 44 px); check keyboard focus order in the grid. Contrast WCAG 2.1 AA.
- Before going live: `design-critique` + accessibility check.
