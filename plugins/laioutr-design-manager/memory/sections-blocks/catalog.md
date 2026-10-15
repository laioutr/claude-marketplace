# Component catalog — available sections, blocks, features

> **Inventory source = Storybook** (`https://storybook.laioutr.cloud`), **not** the docs. The docs only describe the system (`defineSection`/`defineBlock`, schema fields, page types). Which sections/blocks **actually exist** is in Storybook.

Synced from Storybook `index.json`; re-sync when Storybook changes. At last sync: **36 sections, 35 blocks, 14 features**. Additionally in the UI kit: **atoms 35 · molecules 60 · organisms 8** (primitive level → `../design-system/atomic-structure.md`, the `ui-designer`'s domain).

> **Naming convention:** Storybook title `UI/Sections/BannerBasic` ↔ Vue component `SectionBannerBasic` ↔ Studio label "Banner Basic". Blocks likewise `UI/Blocks/X` ↔ `BlockX`. The `Section`/`Block` prefix is mandatory (see `laioutr-developer` coding standards).

---

## Sync (how this catalog is updated)

1. Fetch the Storybook index: `https://storybook.laioutr.cloud/index.json` (JSON, all stories; if the response is large, read it in chunks).
2. Extract the titles `UI/Sections/*`, `UI/Blocks/*`, `UI/Features/*`.
3. Record them here: name, **purpose**, typical slots/data. For detailed props, open a story (`?path=/docs/ui-sections-<name>--docs`) or the section definition in code (`laioutr-developer`).
4. Maintain the page-type mapping in `../page-types/`.

> Storybook is the **rendered truth**. When in doubt whether a section exists or what it can do: open Storybook, don't guess.

---

## Sections (page building blocks — top-level composition)

> _All 36 sections at last sync._

| Section (Storybook) | Component | Purpose (composition) | Typical tags |
|---|---|---|---|
| AlphabeticalIndex | SectionAlphabeticalIndex | A–Z index (e.g. brands/terms) | Navigation, Content |
| BannerBasic | SectionBannerBasic | Simple banner (heading + media + CTA) | Banner, Heroes |
| BannerIntegrated | SectionBannerIntegrated | Banner integrated into content | Banner |
| BannerShowcase | SectionBannerShowcase | Showcase banner (product/feature highlight) | Banner, Featuring Products |
| BlogPostDetail | SectionBlogPostDetail | Blog article detail view | Blog |
| BlogPostListing | SectionBlogPostListing | Blog overview/list | Blog, Grids |
| BrandHero | SectionBrandHero | Brand hero (brand landing) | Heroes, Brands |
| Breadcrumbs | SectionBreadcrumbs | Breadcrumb navigation | Navigation |
| CategoryCardGrid | SectionCategoryCardGrid | Category cards as a grid | Grids, Products |
| CategoryCardSlider | SectionCategoryCardSlider | Category cards as a slider | Sliders, Products |
| Container | SectionContainer | Generic layout container (blank container) | Blank Containers |
| ContentGrid | SectionContentGrid | Content grid (free blocks) | Grids, Content |
| ContentSlider | SectionContentSlider | Content slider (free blocks) | Sliders, Content |
| EditorialGrid | SectionEditorialGrid | Editorial grid (magazine layout) | Grids, Content |
| Footer | SectionFooter | Page footer | Footer |
| HeaderBasic | SectionHeaderBasic | Simple page header | Header, Navigation |
| HeaderShop | SectionHeaderShop | Shop header (nav + search + cart) | Header, Navigation, Checkout & Cart |
| HeroSlider | SectionHeroSlider | Hero with several slides (→ BlockHeroSlide) | Heroes, Sliders |
| LogoGrid | SectionLogoGrid | Logo wall (brands/partners) as a grid | Brands, Grids |
| LogoSlider | SectionLogoSlider | Logo wall as a slider | Brands, Sliders |
| MediaText | SectionMediaText | Media + text side by side | Banner, Content |
| NewsletterRegistration | SectionNewsletterRegistration | Newsletter sign-up | Customer Relations |
| PageNotFound | SectionPageNotFound | 404 page | (Utility) |
| PersonaQuoteSlider | SectionPersonaQuoteSlider | Persona/testimonial quotes as a slider (→ BlockPersonaQuote) | Testimonials, Sliders |
| PlanCardSlider | SectionPlanCardSlider | Plan cards as a slider (→ BlockPlanCard) | Sliders |
| PlanComparisonTable | SectionPlanComparisonTable | Plan comparison table | (Pricing) |
| ProductDetail | SectionProductDetail | PDP main section (query Product, singleEntity) | Product Detail Page |
| ProductImageGallery | SectionProductImageGallery | Product image gallery (PDP) | Product Detail Page |
| ProductSlider | SectionProductSlider | Product slider (→ BlockProductTileBasic, query) | Featuring Products, Sliders |
| ProductSliderShowcase | SectionProductSliderShowcase | Product slider, showcase variant | Featuring Products, Sliders |
| PromotionBanner | SectionPromotionBanner | Promotion banner | Banner |
| QuoteCardSlider | SectionQuoteCardSlider | Quote card slider (→ BlockQuoteCard) | Testimonials, Sliders |
| Review | SectionReview | Reviews (→ BlockReviewItem/ReviewForm) | Customer Relations |
| SearchResultHero | SectionSearchResultHero | Search result hero | Header |
| TopBar | SectionTopBar | Top bar (USP/promo bar) | Header |
| UspBanner | SectionUspBanner | USP bar (→ BlockBenefitsBox) | Banner, Content |

## Blocks (slot content — live in section slots)

> _All blocks at last sync._

| Block (Storybook) | Component | Purpose |
|---|---|---|
| AddToCart | BlockAddToCart | "Add to cart" action |
| BenefitsBox | BlockBenefitsBox | USP/benefit box |
| CartCouponCodeAccordion | BlockCartCouponCodeAccordion | Coupon input (cart, accordion) |
| CartListItem | BlockCartListItem | Cart line |
| CartSummaryBox | BlockCartSummaryBox | Cart totals box |
| CouponBox | BlockCouponBox | Coupon box |
| EmptyStateCart | BlockEmptyStateCart | Empty-cart state |
| EnergyLabel | BlockEnergyLabel | EU energy label |
| FilterBar | BlockFilterBar | Filter bar (listing) |
| FooterMenu | BlockFooterMenu | Footer menu column |
| FreeDeliveryProgress | BlockFreeDeliveryProgress | "Free delivery" progress |
| HeroSlide | BlockHeroSlide | Single slide for the hero slider |
| ImageGallery | BlockImageGallery | Image gallery |
| LocationCard | BlockLocationCard | Location card |
| MegaMenu | BlockMegaMenu | Mega menu |
| MenuBasic | BlockMenuBasic | Simple menu |
| MenuSideBySide | BlockMenuSideBySide | Two-column menu |
| PersonaQuote | BlockPersonaQuote | Persona/testimonial quote |
| PlanCard | BlockPlanCard | Plan/pricing card |
| PopUpInfo | BlockPopUpInfo | Info pop-up |
| PopUpNewsletter | BlockPopUpNewsletter | Newsletter pop-up |
| PopUpPromotion | BlockPopUpPromotion | Promotion pop-up |
| PriceInfo | BlockPriceInfo | Price display |
| ProductListingGrid | BlockProductListingGrid | Product listing grid |
| ProductTileBasic | BlockProductTileBasic | Product tile |
| ProductTitle | BlockProductTitle | Product title |
| QuantityDiscount | BlockQuantityDiscount | Quantity discount display |
| QuoteCard | BlockQuoteCard | Quote card |
| ReviewForm | BlockReviewForm | Review form |
| ReviewItem | BlockReviewItem | Single review |
| SocialShare | BlockSocialShare | Social share buttons |
| SortModes | BlockSortModes | Sort selection (listing) |
| VariantSelectorConfigurator | BlockVariantSelectorConfigurator | Variant configurator |
| VariantSelectorMultiple | BlockVariantSelectorMultiple | Multiple variant selection |
| VariantSelectorOptions | BlockVariantSelectorOptions | Variant options |

## Features (composite functional components)

> _At last sync. Features = more complex, interactive UI units (often app/storefront functions)._

| Feature (Storybook) | Purpose |
|---|---|
| CartSheet | Cart sheet/drawer |
| FilterOffCanvas | Filter off-canvas (mobile) |
| LocationDetail (+ Header, InfoList) | Location detail |
| LocationFinder (+ List, Map) | Location finder with map/list |
| MediaGallery | Media gallery |
| MobileMenuBasic | Basic mobile menu |
| MobileMenuShop | Shop mobile menu |
| PopUp | Pop-up container |
| SearchAutoSuggest | Search auto-suggest |
| VariantOffCanvas | Variant off-canvas |

---

> **What the UX designer uses this for:** sections = top-level composition of a page; blocks = what goes into the section slots; features = interactive functional units. Which blocks a slot allows is governed by `restrictTo`/`allow`/`prefer`/`isStandalone` — see `composition-rules.md`. Data comes in via schema fields (`text`, `media`, `link`, `query`, …) — also described there.
