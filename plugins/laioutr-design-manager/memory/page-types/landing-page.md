# Recipe: Landing Page (marketing)

> **kind: static** — multiple instances, each with its own URL + content, freely composed. The core use case of the `ux-designer`: marketing-driven campaign pages.

## Purpose + funnel role

A focused page for **one campaign / one offer** with **exactly one primary conversion action** (book a demo, newsletter, download, purchase). No full-navigation baggage — everything pays into that one action.

## Section order (default composition)

> Check section names against `../sections-blocks/catalog.md` (Storybook). Example composition:

| # | Section | Purpose (funnel) | Slots → Blocks | Data |
|---|---|---|---|---|
| 1 | BannerBasic / BrandHero / BannerShowcase | **Attention + value** (above the fold) | (hero content / HeroSlide) | `text` heading, `richtext` sub, `media` bg, `link`+`text` CTA |
| 2 | ContentGrid / EditorialGrid | **Benefit** (3–4 benefits) | default → BenefitsBox / QuoteCard | `array` benefits / `richtext` |
| 3 | ContentSlider / BannerShowcase | **Proof** (social proof) | default → PersonaQuote / QuoteCard / ReviewItem | testimonials (`text`/`richtext`/`media`) |
| 4 | CategoryCardGrid / BannerShowcase | **Offer** (products/plans) | default → ProductTileBasic / PlanCard | `query` Product (components: Base, Prices, Media; `limit`) |
| 5 | BannerIntegrated / Container | **Action** (primary CTA, maybe newsletter) | default → (CTA / PopUpNewsletter trigger) | `text`+`link` CTA |
| (6) | Footer | Close/legal | FooterMenu | menu links |

> Not every landing page needs all 6. **One** action, clear logic, no section sprawl.

## Conversion goal

Define exactly **one** primary action (e.g. "Book a demo"). All CTAs point to it. At most one secondary action, visually subordinate.

## Data binding

- **Static:** campaign copy/images via `text`/`richtext`/`media`/`link`. Keep the voice on-brand per the project's brand guidelines (see `memory/design-system/corporate-design.md`; if the project has its own brand/voice guide or plugin, use it).
- **Dynamic:** product/plan data via `query` (Orchestr) — only the entity components you need + `limit`.
- **Multilingual:** fill translatable fields per language.

## Campaign context

Get the campaign goal, audience and brand voice from the user / the project's brand guidelines before building. The landing page is part of a campaign, not a standalone.

## Notes

- `static` page type → in Studio a new page instance with its own slug (`pathConstraints`).
- Choose the hero variant via `propsWizard` (centered/split or similar), if available.
- WCAG 2.1 AA applies here too → run `design-critique` before going live.
