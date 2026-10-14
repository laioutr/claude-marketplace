---
name: designing-entity-components
description: Use when designing entity data in a Laioutr app module. That covers defining a new entity type, or new `defineEntityComponentToken` components (on your own entity, or extra components on a canonical one such as `Product` or `Location`), for connector-fed data such as events, job postings, stores, recipes or brands rendered in listing and detail views. Also use when deciding whether a relationship between entities (a store's events, a brand's products) needs a link token (`defineLinkToken`), when shaping how a component's fields behave as a Laioutr CMS form (`.meta({ cms })`, `localized`, `wellKnownString`, a raw-JSON fallback, `cmsOffers`, `requiredComponents`), or before asking Laioutr to change a canonical entity.
---

# Designing Entity Components

## Overview

An entity component model is **researched before it is written**: internal prior art first, then the outside world. The structure (one entity, many small `defineEntityComponentToken` components) is easy to copy. The value comes from four things:

1. adopting an idiom a canonical sibling already implements;
2. adopting the vocabulary schema.org and the source systems already use;
3. reusing shared value types and standard codes;
4. splitting components by the two forces that matter, **render-view cost** and **connector heterogeneity**.

The two most common failures are reinventing a shape a canonical component already ships (designing a `{ key, name, value }` row from scratch when `ProductVariantOptions` already has `wellKnownName` + `name` + `value`), and inventing a field vocabulary that silently diverges from schema.org and the systems your connector maps from.

Every component may also become a **form in the Laioutr CMS**, which Cockpit builds from the schema alone. A schema that suits connectors can still give editors a raw JSON box or store a SKU once per language.

## When to use

- Defining a new entity type your app's connector feeds (`StoreLocation`, `Event`, `Recipe`).
- Adding your own components to a canonical entity (loyalty points on `Product`, a customer price on `ProductVariant`).
- Reshaping components, deciding links between entities, or deciding CMS behavior.
- **Not** for supplying data for a component that already exists. Import the canonical token and write a resolver for it ([Component Resolvers](https://docs.laioutr.com/frontend/orchestr/component-resolvers)).

## Where this lives in your app

Token files go in `src/runtime/shared/tokens/<feature>.ts`, so both your server handlers and your Vue components can import them ([Runtime layout](https://docs.laioutr.com/apps/app-development/coding-standards#runtime-layout)).

```ts
import { z } from 'zod/v4';
import { defineEntityComponentToken, defineLinkToken } from '@laioutr-core/core-types/orchestr';
// shared value types: Money, Media, HtmlFragment, Link, DateTime, wellKnownString, …
import { HtmlFragment, MediaImage, wellKnownString } from '@laioutr-core/core-types/common';
// canonical value types: SeoMeta, PhoneNumber, GeoCoordinates, MailingAddress, …
import { GeoCoordinates, SeoMeta } from '@laioutr-core/canonical-types';

export const StoreLocationBase = defineEntityComponentToken('base', {
  entityType: 'StoreLocation',
  schema: z.object({ slug: z.string(), name: z.string() }),
});
```

The first argument is the **component name**, the key sections read (`entity.components.base`). Use the canonical conventions: `base`, `info`, `content`, `media`, `seo`, `status`, …. The token registry keys a component by `<entityType>/<name>`, and a second definition of the same pair silently replaces the first. On a canonical entity, therefore, never reuse a canonical component name, and pick a name specific to your feature (`loyalty`, not `info`).

**Changing a canonical component** (adding a field to `ProductInfo`, or a code to a canonical WellKnown set) is not yours to do. Do not redefine the token. Model the data as your own component on that entity, and propose the change upstream to Laioutr, the "proper channels" in [Coding Standards](https://docs.laioutr.com/apps/app-development/coding-standards#handler-implementation).

## Workflow

Do these in order. Don't jump to writing schemas.

1. **Explore what already exists, and search for the pattern, not only the entity.** Read the closest canonical analogue in the [Entities reference](https://docs.laioutr.com/frontend/api-reference/entities) (`Product`, `Location`, `BlogPost`, `Category`) and its installed source. For example, `node_modules/@laioutr-core/canonical-types/dist/server/entity/Location/` holds `.d.ts` files with JSDoc and `.js` files with the zod calls. Then grep for the *shape* you are about to build: `grep -rE "wellKnownString|z.union|cms:" node_modules/@laioutr-core/canonical-types/dist/server/entity`. If a sibling implements it, copy its idiom verbatim.
2. **Research how others model this data.** Dispatch web-research subagents if you can. Cover the **schema.org** type, Google structured-data requirements (Google Jobs, LocalBusiness, …), and the **SaaS / shop-system APIs** your connector maps from. For each field, capture the vocabulary, whether it is **coded** or free text, whether values are **heterogeneous** (a typed value union), whether they are **localized** (resolved per locale by the connector, so a plain `z.string()`, never an inline locale map), and whether rows **group into sections**. Adopt established names. Do not invent parallel ones.
3. **Map to existing types and codes.** Check [`value-types-and-code-systems.md`](./references/value-types-and-code-systems.md) before typing any field `z.string()` or `z.number()`.
4. **Decide each field's CMS treatment:** localized or not, which editor it gets, whether it falls back to raw JSON, and which components an entry needs before it publishes ([`cms-form.md`](./references/cms-form.md)).
5. **Propose the components**, split by the rules below, and confirm the split with the user before writing files.
6. **Map the links the frontend will load, and only those** ([`links-between-entities.md`](./references/links-between-entities.md)). Confirm the link set together with the component split.
7. **Wire it.** Export the tokens from your shared token file. Write the component resolver (`src/runtime/server/orchestr/<Entity>/*.resolver.ts`) and any query and link handlers. If projects should store the type in the CMS, add a `cmsOffers` entry to `registerLaioutrApp` ([`cms-form.md`](./references/cms-form.md#offering-your-types-cmsoffers)).

## Splitting rules

- **Split by render view (performance).** Lightweight listing fields (identity, status, filter facets, a plain-text teaser, a thumbnail) go in small components the listing resolves. Heavy HTML and galleries go in **detail-only** components, so a listing never resolves what it does not render. Canonical precedent: `ProductInfo.shortDescription` (tile) vs `ProductDescription` (product page), and `LocationInfo.cover` vs `LocationContent` / `LocationMedia`.
- **Split by connector heterogeneity.** Give each concern its own component with **all-optional** fields, so a connector emits only what it has and an absent component degrades gracefully. The exception is a field the CMS's `requiredComponents` relies on (an event's start): put it in its own small component and make it required there. Document what "absent" means (canonical `LocationStatus`: absent means operational). One fat component full of nulls is the anti-pattern.
- **Object-wrapped schemas, always.** `schema` is `z.object({ named: Field })`, never a bare value type. See [`entity-component-object-wrapped.md`](./references/entity-component-object-wrapped.md).
- **Single source of truth.** Never carry the same fact in two places. Geo lives only on `LocationCoordinates`, not also on the address. The one allowed exception is a cheap value denormalized onto the listing `info` component so a card need not resolve a detail component (`LocationInfo.cover`). Flag that duplication to the user.
- **Reuse before invent.** A phone is a `PhoneNumber`, a country an ISO 3166-1 code, money a `Money`, a calendar date a `CalendarDate`, an instant (date and time) a `DateTime`. Store codes, not display strings or icon names.
- **No foreign keys.** A relationship is a link, never an `xxxId` field.

## Common mistakes

| Mistake | Fix |
|---|---|
| Reinventing a shape a canonical sibling ships | Step 1: grep the installed entities for the pattern and copy the idiom |
| Skipping external research | Step 2: adopt schema.org and source-system vocabulary |
| `z.string()` for a standardized field, or for an open vocabulary | The shared type or code, or `wellKnownString(WELL_KNOWN_X)` |
| A code, id, SKU or time zone as a bare `z.string()` | `.meta({ cms: { localized: false } })` |
| A record, tuple or discriminator-less union added without a decision | It is a raw JSON box in the CMS. Prefer an object or `z.discriminatedUnion`. |
| WellKnown key as the only carrier of a label | Pair it with a display-label fallback (`wellKnownName` + `name`) |
| Pre-formatted string for a locale-sensitive value | A typed value union |
| One fat component with nullable fields | One component per concern, all-optional |
| Heavy HTML or a gallery in a listing-resolved component | A detail-only component |
| Bare value type as `schema` | `z.object({ named: ... })` |
| Redefining or reusing a canonical component name | Your own component name, plus an upstream proposal |
| Linking every backend relationship | Link only what a real page loads |

## References

Open the one whose trigger matches:

- [`references/value-types-and-code-systems.md`](./references/value-types-and-code-systems.md): typing a field. Covers `wellKnownString` and open vocabularies, display-label pairing, typed value unions, section grouping, the shared value type table with import paths, temporal types, and standard codes.
- [`references/links-between-entities.md`](./references/links-between-entities.md): one entity relates to another. Covers the frontend-load test, non-links, direction and depth (`WrongLinkSourceError`, `LinkRecursionError`), the `defineLinkToken` idiom and metadata.
- [`references/cms-form.md`](./references/cms-form.md): the type is or may become a CMS collection. Covers `.meta({ cms })` keys, localization defaults, editors, raw-JSON fallbacks, labels, `requiredComponents`, `cmsOffers` and their queries.
- [`references/entity-component-object-wrapped.md`](./references/entity-component-object-wrapped.md): writing any `schema:`. Includes the legacy canonical shapes not to copy.

Related skill: `designing-action-tokens`, for mutations and on-demand reads that are not entity data.
