# Design Every Component as a CMS Form

Any entity type can become a collection in the Laioutr CMS. That happens when your app offers it through `cmsOffers` and a project names it, or when a project names a token module for it. Cockpit then builds the editor's form from the component's zod schema and reads nothing else. A schema that suits connectors can still give editors a raw JSON box, store a SKU once per language, or publish an entry your sections cannot render. Decide each field's CMS behavior when you design the component, not afterwards.

Primary source: [Content Collections](https://docs.laioutr.com/apps/app-development/content-collections). The behavior below is verified against it and against the CMS form compiler.

## Hints: `.meta({ cms: { … } })`

Hints are typed as `CmsFieldMeta` (`@laioutr-core/core-types/common`). They change only how Cockpit edits a field, never how your storefront parses a value.

| Key | Effect |
|---|---|
| `localized` | `true` / `false`: whether the value differs per locale. It reaches every field below the one that carries it. |
| `label` | The field label. Without it, Cockpit uses the schema's `title`, then the property name split into words (`googlePlaceId` reads "Google Place Id"). |
| `description` | Help text. Without it, Cockpit uses the schema's `description`. `''` or `null` shows none. |
| `placeholder` | Shown in an empty input. |
| `default` | Written when the value is created (new entry, new list item, new variant), never to an existing value. |
| `suggestions` | Values the editor offers for a string field, which still accepts any other. `wellKnownString` sets this for you. |
| `kind` | The editor to use instead of the one the shape gives (`core.*`, `laioutr.richtext`, `laioutr.money`, `laioutr.media`, `laioutr.link`, `core.json`). |

Each key is read on its own, and **the outermost one wins, key by key**. `price: Money.meta({ cms: { label: 'Price' } })` keeps `Money`'s own `kind` and `localized` and adds the label.

## Localization: decide it for every string

The CMS stores a localized field once per locale and any other field once for all locales. The default follows the shape:

| Shape | Localized by default |
|---|---|
| `z.string()` with no format, enum or literal | **yes** |
| `HtmlFragment` (rich text), `Media` / `MediaImage`, `Link` | yes |
| number, integer, boolean, `z.enum`, literal | no |
| a string with a format: `z.email()`, `z.url()`, `DateTime`, `CalendarDate`, the `time` in `Time` | no |
| `Money`, `wellKnownString(…)`, `PhoneNumber` | no, because they carry their own `localized: false` |
| raw-JSON fallback (below) | no |

An object, a list or a discriminated union holds no value itself, so each field below it decides for itself. A `localized` set on the container reaches every field below it.

**A plain `z.string()` that holds no words a translator would change gets `.meta({ cms: { localized: false } })`.** That covers an internal code, an external or source-system id, a SKU or GTIN, a Google Place ID, a BCP-47 tag, an IANA time zone, a country or postal code, and any key. Canonical precedents: `LocationBase.code`, `.externalId` and `.googlePlaceId`. Words stay localized: `name`, a teaser, a display-label fallback (`sectionName`, `statusLabel`). `slug` stays localized too, as it is on every canonical `base` component.

## Take the editor from the shared type

`Money`, `Media` / `MediaImage`, `Link` and `HtmlFragment` declare their own `kind`, so reusing them brings the money, media, link and rich-text editors for free. `DateTime`, `CalendarDate` and the `time` in `Time` reflect with a `format` and get date and time pickers. A `z.date()` field is stored as an ISO 8601 timestamp in UTC and reaches your component as a `Date`. Write `kind` only to choose one of Cockpit's shipped editors that the shape would not otherwise get. A `kind` Cockpit has no editor for is kept, and the field is edited as JSON or shown read-only.

## Raw JSON is a fallback, not a field type

Cockpit cannot build a form for these shapes. It shows a JSON box instead, and does **not** localize the value:

- a record (`z.record`) or an object without declared properties;
- a tuple;
- a recursive schema;
- a list of lists;
- a union whose branches share no required literal property, i.e. no discriminator.

Canonical types that already fall back include `Timespan` (a union without a discriminator), `Swatch` (tuples), `CustomFields` (a record) and the `OpeningHours` schedule (records).

- **Prefer a shape with an editor:** an object, a list of objects, or `z.discriminatedUnion('type', [...])` over objects with a literal `type`. The CMS gives the last one a variant picker.
- **If a fallback is still the right model** (connector fidelity beats the editor), choose it deliberately and flag it to the user. Mark it `.meta({ cms: { localized: true } })` when it can hold text, as canonical `ProductSpecifications.value` does.

## Labels and help text

The default label is the property name split into words. Where that would mislead an editor, set `cms.label`. Help text comes only from `cms.description` or the schema's `description`. A JSDoc comment on a field is a TypeScript comment, not schema data. Laioutr's own packages copy field JSDoc into `.meta({ description })` at build time (visible in the published `.js`), but a normal module build does not. Write `cms.description` (or `.meta({ description })`) yourself where an editor needs help.

## Required components

Every component of an entry may be absent, and Cockpit publishes it anyway unless the type marks components as mandatory. For a type you offer, set `requiredComponents` on the offer. **If you omit it, nothing is required.** For a type a project defines itself, the project sets it on its collection entry.

Name a component when your sections read it **without checking that it is there**. Decide by those reads, not by what connectors usually return. Canonical `ProductVariant` requires `availability` but not `prices`, because every price read is guarded. For a new type with no sections yet, require what a listing cannot render without (an event's start time), and confirm the list together with the component split. A name that matches no component is not enforced. The build or the server start logs it.

## Offering your types: `cmsOffers`

```ts
// src/module.ts, inside setup()
await registerLaioutrApp({
  name,
  version,
  orchestrDirs: [resolve('./runtime/server/orchestr')],
  cmsOffers: [
    {
      entityType: 'Recipe',
      tokens: resolve('./runtime/shared/tokens/recipe'), // exports the type's components, queries, links, page types
      slug: 'base.slug',
      requiredComponents: ['base'],
      queries: { 'acme/recipe/all': 'all', 'acme/recipe/by-slug': 'by-slug' },
    },
  ],
});
```

- **Register offers in module setup.** An offer registered later is not seen.
- **The token module is imported twice,** into the server bundle and into the Vue app, so it must load in both. A token file in `src/runtime/shared/` already satisfies that.
- **An offer serves nothing by itself.** A project must name the type in `collections`.
- **`queries`** sets the queries the CMS answers by default: `'all'`, `'by-slug'` (the slug in input key `slug`), `{ serve: 'by-slug', input: '<key>' }`, or `{ serve: 'through-link', … }`. A query not served by default, and not listed by the project, is not answered.
- **Page types:** the CMS indexes a page type for the type only when `slug` is its one route parameter and one of its `requiredQueries` is served by slug. To copy a working pair, use canonical `BlogPost`: the `blog/post/all` and `blog/post/by-slug` queries, plus the `blog/post-single` page type.
- **Links** are served by the source type's collection. See [`links-between-entities.md`](./links-between-entities.md#in-the-laioutr-cms).

## Changing a schema after entries exist

Published entries passed the schema that was current when they were published. If you reshape a component afterwards, the resolver skips any entry that no longer passes, with a warning naming the entry and the component. The entry's id stays in query results, so sections receive an entity without data. Treat a component reshape as a breaking change of your app.

## Hard rules

- Decide `localized` for every string field. Codes, ids, keys, tags and zones get `localized: false`.
- Reuse `Money`, `Media`, `Link`, `HtmlFragment` and the temporal types for their editors. Do not hand-roll their shapes.
- Never add a record, a tuple or a discriminator-less union without deciding to accept the JSON box.
- Never rely on field JSDoc as help text. Use `cms.label` and `cms.description`.
- Decide `requiredComponents` for every type you offer.
