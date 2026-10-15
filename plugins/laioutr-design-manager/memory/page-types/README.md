# Page-type recipes — canonical structure per page type

> How a page of a given type is composed: which sections in which order, which blocks go into the slots, which data. The `ux-designer` starts every job from here.

**Source:** the page-type system from the Laioutr docs (`/frontend/features/pagetypes`, `/frontend/api-reference/page-types`) + the section inventory from Storybook (`../sections-blocks/catalog.md`).

---

## Page types: static vs. dynamic

| Kind | Meaning | Examples | Path |
|---|---|---|---|
| **dynamic** | **one** page per project, route/param-driven, loads entity data | Home (`/`), Product Detail (`/products/:slug`), Product Listing, Blog Post, Blog Collection, Location Detail/Finder | from `pathConstraints` |
| **static** | **multiple** instances, each with its own URL + content, composed by an editor | **Landing Page**, Content Page, Promotion Page | per instance |

→ **Landing pages + content pages are `static`** — exactly the free composition the UX designer does. Dynamic types (Home/PDP/PLP) are more constrained by queries/routing.

## Built-in page-type domains (API reference)

- **Core** — general pages (Home, Content/Landing).
- **Ecommerce** — Product Detail Page, Product Listing Page, Search.
- **Blog** — Blog Post, Blog Collection.
- **Locations** — Location Finder, Location Detail.

> Import from the domain package (e.g. `@laioutr-core/canonical-types/ecommerce`). Custom page types via `definePageTypeToken` (a job for `laioutr-developer`).

---

## Recipe schema (how every type is documented)

Each recipe file (`<page-type>.md`) contains:
1. **Purpose + kind** (static/dynamic) + funnel role.
2. **Section order** (table: # · Section · Purpose · Slots→Blocks · Data).
3. **Data binding** (which `query` fields, which entities).
4. **Conversion goal** (especially for landing pages).
5. **Variants / notes**.

## Available recipes

| Page type | File | kind | Status |
|---|---|---|---|
| **Landing Page** (marketing) | `landing-page.md` | static | v0.1 |
| **Home** | `homepage.md` | dynamic | v0.1 |
| Product Detail (PDP) | _TBD_ | dynamic | open |
| **Product Listing** (PLP, B2C) | `product-listing-page.md` | dynamic | v0.1 |
| **Product Listing B2B** (PLP B2B) | `product-listing-page-b2b.md` | dynamic | v0.1 |
| Content Page | _TBD_ | static | open |
| Blog Post / Collection | _TBD_ | dynamic | open |
| Location Finder / Detail | _TBD_ | dynamic | open |

> **Rule for all product listing pages (B2C + B2B):** product tiles carry **no buttons** (no add-to-cart, no quantity field, no wishlist/order-list trigger). The whole tile links to the PDP.

> Add new recipes here as soon as a page type is built for the second time. Always check section names against `../sections-blocks/catalog.md` (Storybook).
