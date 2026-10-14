# Reuse Value Types, Codes and Open Vocabularies Before Typing a Field

Before you type any field as `z.string()` or `z.number()`, check this file. A field that has a standard representation uses it. Store **codes**, never display strings or icon names, and let the frontend map each code to a localized label and icon. When no shared type fits, define one once in your app instead of inlining the shape across components.

## Open vocabularies: `wellKnownString`

A field whose values vary across connectors but share a known core (services, payment methods, employment types, property or section names) is an **open vocabulary**. Build it with `wellKnownString` from `@laioutr-core/core-types/common`. Do not write a bare `z.string()` (which loses the union in inference), and do not cast `z.string() as z.ZodType<…>`.

```ts
import { z } from 'zod/v4';
import { wellKnownString, type WellKnownString } from '@laioutr-core/core-types/common';
import { defineEntityComponentToken } from '@laioutr-core/core-types/orchestr';

export const WELL_KNOWN_EMPLOYMENT_TYPES = ['full-time', 'part-time', 'contract'] as const;
export type WellKnownEmploymentType = WellKnownString<typeof WELL_KNOWN_EMPLOYMENT_TYPES>;

export const JobPostingInfo = defineEntityComponentToken('info', {
  entityType: 'JobPosting',
  schema: z.object({
    employmentType: wellKnownString(WELL_KNOWN_EMPLOYMENT_TYPES).optional(),
  }),
});
// The runtime accepts any string. The type autocompletes the known ones.
```

`wellKnownString` keeps the union in inference, gives the CMS the known values as `suggestions` (an editor picks one or types another), and marks the field `localized: false`, because the key is the same in every language. Export the values as **one** `WELL_KNOWN_…` constant, so the schema and your frontend's label map share a single source. Canonical precedents: `ProductVariantOptions`, `ProductSpecifications` and `Measurement.unit`.

Use a **closed `z.enum`** instead only when the set is fixed and owned by your app or the platform (a lifecycle `status`). Whichever you choose, apply it consistently across sibling fields and say why.

**Pair a WellKnown key with a display-label fallback.** The key is locale-independent identity: the frontend maps it to a label and icon, and it enables grouping and comparison across entities. The frontend cannot know every connector's key, though. Also carry a plain, connector-resolved label as the fallback for an unmapped key: `wellKnownName?` + `name` (required), and for grouping `wellKnownSectionName?` + `sectionName?`. Never make the key the only carrier of a human-readable value. For a single coded field on an entity whose `name` already means something else, nest the pair: `category: z.object({ wellKnownName: wellKnownString(WELL_KNOWN_CATEGORIES).optional(), name: z.string() })`. Copy canonical `ProductVariantOptions` (`selected[].wellKnownName` + `name` + `value`) verbatim when you need this row shape.

**Check entity-side vocabulary first.** If a canonical entity already defines the concept (`WellKnownQuoteStatus` on `Quote`, `WellKnownBudgetPeriod` on `Budget`, `WellKnownOptionName` on `ProductVariant`), import and reuse it. Two diverging code sets for one concept are worse than a cross-package `import type`.

## Typed value unions

When one field's value is heterogeneous across rows (text, a count, a quantity, a price), do **not** pre-format it as a string. A pre-formatted string is frozen in one locale, so a German market cannot reformat it. Model it as a union of bare primitives and reusable value types, so the frontend formats it per locale. schema.org's `PropertyValue.value` is exactly `Text | Number | Boolean | StructuredValue`.

```ts
value: z.union([z.string(), z.number(), z.boolean(), Measurement, Money]).meta({ cms: { localized: true } }),
```

Keep primitives bare (`value: 'Organic cotton'`, `value: 400`). `Measurement` (`{ value, unit }`) and `Money` (`{ amount, currency }`) are objects with disjoint keys, so each identifies itself. A type that would be ambiguous as a bare string (an ISO date) gets a single-key wrapper, as `Duration` (`{ duration }`) and `Time` (`{ time }`) already are. Add such members only when a connector needs them.

**The CMS cost:** a union without a discriminator is a raw JSON box in the CMS and is not localized unless it says so. Use it only where connector fidelity outweighs the editor, flag it, and add `localized: true` when it can hold text, exactly as canonical `ProductSpecifications.value` does. See [`cms-form.md`](./cms-form.md#raw-json-is-a-fallback-not-a-field-type).

**Group without forcing structure.** For rows that sometimes belong to sections, use an **optional per-row section key and label** (Google Merchant's `product_detail` repeats `section_name` per row), not a nested `groups: [{ label, rows }]` array. Nesting forces a section onto connectors that have none. Absent section fields mean the row is ungrouped, and the frontend groups by key.

## Reusable value types

| Need | Type | Import from |
|---|---|---|
| Price / amount | `Money` (integer minor units + ISO 4217; see [`money-currency-code.md`](../../laioutr-platform/references/money-currency-code.md)) | `@laioutr-core/core-types/common` |
| Per-unit price | `UnitPrice` | `@laioutr-core/core-types/common` |
| Image / video / audio | `Media`, `MediaImage` | `@laioutr-core/core-types/common` |
| Rich text | `HtmlFragment` (`{ html }`) | `@laioutr-core/core-types/common` |
| Internal link / reference | `Link` | `@laioutr-core/core-types/common` |
| Physical quantity | `Measurement` (WellKnown unit) | `@laioutr-core/core-types/common` |
| Opening hours | `OpeningHours` (IANA `timeZone` + exceptions) | `@laioutr-core/core-types/common` |
| Connector passthrough | `CustomFields` (a record, so raw JSON in the CMS) | `@laioutr-core/core-types/common` |
| SEO meta | `SeoMeta` | `@laioutr-core/canonical-types` |
| Rating | `Rating` (1–5) | `@laioutr-core/canonical-types` |
| Phone / fax | `PhoneNumber` (E.164) | `@laioutr-core/canonical-types` |
| Person name | `PersonName` | `@laioutr-core/canonical-types` |
| Postal address | `LocationAddress` / `MailingAddress` | `@laioutr-core/canonical-types` |
| Geo point | `GeoCoordinates` (WGS84) | `@laioutr-core/canonical-types` |

The common types are documented under [Common Types](https://docs.laioutr.com/frontend/api-reference/common-types). The installed source, with JSDoc, is in `node_modules/@laioutr-core/core-types/dist/common/` and `node_modules/@laioutr-core/canonical-types/dist/server/common/`.

## Temporal types: pick by precision

Never default a temporal field to `z.string()`. All of these come from `@laioutr-core/core-types/common`:

| Concept | Type | Format |
|---|---|---|
| Calendar date, no time | `CalendarDate` | `YYYY-MM-DD` |
| Time of day | `Time` | `{ time: 'HH:MM:SS' }` (+ optional offset) |
| Full instant | `DateTime` | ISO 8601 with offset or `Z` |
| Duration | `Duration` | `{ duration: <ISO 8601 duration> }` |
| Range of dates or instants | `Timespan` | `min` / `max`. It is raw JSON in the CMS, whereas two `DateTime` fields get pickers. |

## Standard code systems

| Standard | Codifies | Example |
|---|---|---|
| ISO 3166-1 alpha-2 | Country | `DE` |
| ISO 3166-2 | Region / province (without the country prefix, as in `LocationAddress`) | `BY` |
| ISO 4217 | Currency (in `Money`) | `EUR` |
| ISO 8601 | Date / time / duration | `2026-01-31` |
| E.164 | Phone (`PhoneNumber`) | `+49301234567` |
| BCP-47 | Language tag | `de-DE` |
| IANA tz | Time zone (in `OpeningHours`) | `Europe/Berlin` |
| WGS84 | Coordinates (`GeoCoordinates`) | `52.52, 13.40` |

A plain-string code field (a country, a language tag, a zone) needs `.meta({ cms: { localized: false } })`. See [`cms-form.md`](./cms-form.md#localization-decide-it-for-every-string).

## When the type you need is missing

Do not fall back to `z.string()` / `z.date()`, and do not copy an inline shape across sibling components. Define the type **once in your app** (for example `src/runtime/shared/types/<Type>.ts`, exported for your tokens and your frontend) and reference it under a key. If it is generic (a primitive every app would want), also propose it upstream for `@laioutr-core/core-types` or `@laioutr-core/canonical-types`.

## Forbidden patterns

| Pattern | Instead |
|---|---|
| `z.string()` for a field with a standard (phone, country, money, date) | The value type or code system from the tables above |
| `z.string()` or an `as z.ZodType<WellKnownX>` cast for an open vocabulary | `wellKnownString(WELL_KNOWN_X).optional()` |
| A display label or icon name stored as the value | Store the code. The frontend maps it to a label and icon. |
| A WellKnown key as the only human-readable carrier | Pair it with a display-label fallback (`wellKnownName` + `name`) |
| A pre-formatted string for a value the frontend must localize | A typed value union |
| A nested `groups[]` structure for optional grouping | An optional per-row section key + label |
| A second code set for a concept a canonical entity already defines | Import the canonical type |
