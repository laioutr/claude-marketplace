# Recipe: Home

> **kind: dynamic** — one instance per project, path `/`. More constrained than a landing page (routing/queries), but the section composition is designed by the `ux-designer`.

## Purpose + funnel role

The brand and navigation entry point. Broader than a landing page: several paths (browse, categories, news, trust), not just one action. Clear hierarchy, fast value.

## Section order (default composition)

> Check section names against `../sections-blocks/catalog.md` (Storybook).

| # | Section | Purpose | Slots → Blocks | Data |
|---|---|---|---|---|
| 1 | BannerShowcase / BrandHero | Brand hero + main message | HeroSlide(s) | `media`, `text`, `link` |
| 2 | CategoryCardSlider / CategoryCardGrid | Entry into the assortment | ProductTileBasic / category cards | `query` Category/Product |
| 3 | BannerShowcase | Promotion/product highlight | ProductTileBasic | `query` Product (Base, Prices, Media) |
| 4 | ContentSlider / EditorialGrid | Storytelling / editorial | QuoteCard / content blocks | `richtext`, `media` |
| 5 | ContentGrid | Trust / benefits / brands | BenefitsBox / PersonaQuote | `array`, `richtext` |
| 6 | BannerIntegrated / Container | Newsletter / secondary CTA | PopUpNewsletter / CTA | `text`, `link` |
| 7 | Footer | Footer | FooterMenu | menu links |

## Data binding

Bind product/category sections to real entities via `query` (Orchestr) (Base + Prices + Media, a sensible `limit`). Editorial content static via `richtext`/`media`.

## Notes

- `dynamic`, path `/` — one instance. Section choice is free, but header/navigation/footer are often global (not part of the page composition).
- Several entry points are allowed (unlike a landing page) — still keep a clear hierarchy.
- WCAG 2.1 AA + `design-critique` before going live.
