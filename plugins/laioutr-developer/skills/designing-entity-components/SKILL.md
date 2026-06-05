---
name: designing-entity-components
description: Use when adding a new entity, or new components for an existing entity, to packages/canonical-types (defineEntityComponentToken schemas) — a connector-fed data type (event, job posting, location, recipe, brand, …) rendered across listing and detail views.
---

# Designing Entity Components

## Overview

An entity component model is **researched before it is written**. The structural patterns (one entity → many small `defineEntityComponentToken` components) are easy to copy from an existing entity; the value is in (1) adopting the vocabulary the rest of the world already uses, (2) reusing types and standards instead of reinventing them, and (3) splitting components by the two forces that actually matter: **render-view cost** and **connector heterogeneity**.

The most common failure is skipping research and reinventing a field vocabulary that silently diverges from schema.org / the source systems connectors map from.

## When to use

- Adding a new entity to `packages/canonical-types/src/entity/<Entity>/`.
- Adding or reshaping components on an existing entity.
- Any `defineEntityComponentToken` design decision.

## Workflow

Do these in order. Don't jump to writing schemas.

1. **Explore what's already here.** Read the closest existing analogue in `packages/canonical-types/src/entity/` (`Product`, `Location`, `BlogPost`, `Category`) and the shared value types in `packages/core-types/src/common/` and `packages/canonical-types/src/common/`. Learn the `defineEntityComponentToken` shape and the token-name conventions (`base`, `info`, `content`, `media`, `seo`, `status`, …).

2. **Research how others model this data.** Dispatch subagent(s) (web research) covering, as relevant: **schema.org** type for this concept (and Google structured-data requirements — e.g. Google Jobs for a posting, LocalBusiness for a place), the relevant **SaaS / APIs / shop systems** connectors will map from (Shopify, Yext, Uberall, Greenhouse/Lever, Google Business Profile, …), and what fields each exposes. Capture the *field vocabulary* and which values are standardized. Adopt the established tokens; do not invent parallel ones.

3. **Map to existing types + standard formats.** Before typing any field as `z.string()`/`z.number()`, check the tables below. Reuse a shared value type and a standard code system if one fits. If a *generic* primitive is missing (e.g. a date-without-time), add it to `core-types/common` rather than inlining.

4. **Propose components**, split by the two forces (below), applying every rule. Confirm the split with the user before writing files; schema changes to published packages need a changeset (`.claude/rules/changeset.md`).

## Splitting rules

**Split by render view (performance).** Lightweight listing/finder fields (identity, status, filter facets, a plain-text teaser, a thumbnail) go in small components the listing resolves. Heavy HTML and galleries go in their own **detail-only** components so the listing never resolves (or ships) what it won't render. Precedent: `ProductInfo.shortDescription` (tile) vs `ProductDescription` (HTML, PDP); `LocationInfo.cover` vs `LocationContent`/`LocationMedia`.

**Split by connector heterogeneity.** Each concern is its own component with **all-optional** fields, so a connector emits only what it has and absent components degrade gracefully (document "absent ⇒ default", e.g. on `status`). One fat component full of nulls is the anti-pattern.

**Object-wrapped schemas, always.** A component `schema` is always `z.object({ named: Field })`, never a bare value type — even when the value type is itself an object. See `.claude/rules/entity-component-object-wrapped.md`.

**Single source of truth.** Never carry the same fact in two places (the coordinates lesson: geo lives only on `LocationCoordinates`, not also on the address). Pick one carrier. *Allowed exception:* a cheap value may be denormalized onto the listing `info` component so a card needn't resolve a detail component (as `LocationInfo.cover` does) — but the canonical carrier is still the detail component, and you must flag the duplication for sign-off.

**Reuse before invent.** A field with a standard representation must use it (phone → `PhoneNumber`/E.164, country → ISO 3166-1, money → `Money`, date → `CalendarDate`). Don't put display strings or icon names in the data layer — store codes; the frontend maps code → localized label + icon.

## WellKnown types (open vocabularies)

For a field whose values vary across connectors but have a known core (services, payment methods, employment types, categories), use the **WellKnown union + escape hatch** pattern (precedent: `Measurement`, `LocationAttributes`):

```ts
export type WellKnownEmploymentType =
  | 'full-time'   // Full time
  | 'part-time'   // Part time
  | 'contract';   // Contract
export type EmploymentType = WellKnownEmploymentType | (string & {});
// runtime stays loose; the frontend maps codes → localized label/icon
schema: z.object({ employmentTypes: z.array(z.string()).optional() })
```

Use a **closed `z.enum`** instead only when the set is fixed and platform-owned (e.g. a lifecycle `status`). When you reach for either, be consistent across sibling fields and say why.

## Reusable value types

| Need | Type | From |
|---|---|---|
| Price / amount | `Money` (minor units + ISO 4217) | `core-types/common` |
| Image / video / audio | `Media`, `MediaImage` | `core-types/common` |
| Rich text | `HtmlFragment` (`{ html }`) | `core-types/common` |
| SEO meta | `SeoMeta` | `canonical-types/common` |
| Rating | `Rating` (1–5) | `canonical-types/common` |
| Phone / fax | `PhoneNumber` (E.164) | `canonical-types/common` |
| Person name | `PersonName` | `canonical-types/common` |
| Postal address | `LocationAddress` / `MailingAddress` | `canonical-types/common` |
| Geo point | `GeoCoordinates` (WGS84) | `canonical-types/common` |
| Opening hours | `OpeningHours` (IANA tz + exceptions) | `core-types/common` |
| Internal link / reference | `Link` | `core-types/common` |
| Physical quantity | `Measurement` (WellKnown unit) | `core-types/common` |

## Temporal primitives — pick by precision

Check the current set in `core-types/common` (it is being consolidated). Never default a temporal field to `z.string()`.

| Concept | Type | Format |
|---|---|---|
| Calendar date, no time | `CalendarDate` | `YYYY-MM-DD` |
| Time of day | `Time` | `HH:MM:SS` (+ optional offset) |
| Full instant | `DateTime` | ISO 8601 with offset/`Z` |
| Duration | `Duration` | ISO 8601 duration |
| Range of dates/datetimes | `Timespan` | min/max |

## Standard code systems

| Standard | Codifies | Example |
|---|---|---|
| ISO 3166-1 alpha-2 | Country | `DE` |
| ISO 3166-2 | Region/province | `BY` |
| ISO 4217 | Currency (in `Money`) | `EUR` |
| ISO 8601 | Date / time / duration | `2026-01-31` |
| E.164 | Phone (`PhoneNumber`) | `+49301234567` |
| BCP-47 | Language tag | `de-DE` |
| IANA tz | Timezone (in `OpeningHours`) | `Europe/Berlin` |
| WGS84 | Coordinates (`GeoCoordinates`) | `52.52, 13.40` |

## Common mistakes

| Mistake | Fix |
|---|---|
| Skipping external research; reinventing the field vocabulary | Phase 2 — adopt schema.org / source-system tokens |
| `z.string()` for a field with a standard | Reuse the type / code system in the tables |
| Display labels or icon names in the schema | Store a code; frontend maps to label/icon |
| Inconsistent `z.enum` vs WellKnown across sibling fields | Pick per the WellKnown rule; document why |
| One fat component with nullable fields | One component per concern, all-optional |
| Same fact in two components/fields | Single source of truth — one carrier |
| Bare value type as a component `schema` | Wrap in `z.object({ named: ... })` |
| Heavy HTML/gallery in a listing-resolved component | Isolate in a detail-only component |

## Related

- `.claude/rules/entity-component-object-wrapped.md` — schemas are always object-wrapped
- `.claude/rules/zod-schemas.md` — `z.object({ ...base.shape })` over `.extend()`
- `.claude/rules/changeset.md` — schema changes to `packages/*` need a changeset
