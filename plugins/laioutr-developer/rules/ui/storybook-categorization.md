# Storybook Categorization

`ui` package Storybook stories are organized by the **role the component plays in ui-app**, not by Atomic Design. Every new story must pick the right category up-front — moving stories later breaks Chromatic baselines.

## Title format

```
title: 'UI/<Category>/<ComponentName>'
```

Categories: `Blocks`, `Sections`, `Features`.

## How to categorize

### Blocks

Small, composable units that live inside a Section slot. Wrapped by a `Block*.vue` in `ui-app` and placed by editors in Studio.

Examples: `HeroSlide`, `AddToCart`, `PriceInfo`, `Rating`, `QuoteCard`, `Text`, `BenefitsBox`, `ProductTileBasic`.

### Sections

Full-width page-level layouts. Wrapped by a `Section*.vue` in `ui-app` and placed by editors as a page section in Studio.

Examples: `BannerBasic`, `BannerHero`, `BrandHero`, `BrandList`, `Breadcrumbs`, `ContentGrid`, `CategoryCardSlider`, `Footer`, `ProductGrid`.

### Features

Functional UI that is **not configurable via Studio** — it exists because the storefront needs it, not because an editor places it. Usually mounted once per shop (chrome, overlays, global surfaces) and wired from a `ConnectedX` component in `ui-app`.

Examples: `CartSheet`, `LightboxGallery`, `Header` (if introduced), `AutoSuggest`, `MiniWishlist`.

Signals that a component is a Feature:

- It has no corresponding `Section*.vue` or `Block*.vue` in `ui-app`
- It is mounted once, globally — not placed by editors per page
- Its Studio config surface is **absent**, not "empty"
- It binds to runtime data via a `Connected*` wrapper in `ui-app`

## Edge cases

- **A Feature that re-uses a Section visually** (e.g. a sheet that embeds a CategoryCardGrid) → still a Feature. Categorization follows the consumer-facing role, not the internal composition.
- **A Block that is only used by one Section** → still a Block. Don't promote it to Feature just because it's narrowly used.
- **If you can't decide** between Feature and Section: ask "does an editor place this from the component picker?". If yes → Section. If no → Feature.

## Why this matters

- Storybook navigation mirrors the three real roles Studio exposes (or explicitly does not).
- Chromatic baselines stay stable (the title is part of the snapshot key).
- Forces the architectural question up-front: if a component can't find a category, it is likely doing two jobs and should be split.
