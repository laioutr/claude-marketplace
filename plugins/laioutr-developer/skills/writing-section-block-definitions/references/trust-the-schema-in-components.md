# Trust the Schema in Components

Frontend-core normalizes every schema field before it reaches the component. A field with `default: X` arrives already populated; picker-shaped fields additionally arrive as one of the declared options; and `object` / `array` fields are *always* materialized — `{}` with sub-fields recursively populated, or `[]` — never `null` or `undefined`, with or without a `default`. The component must trust that contract — no `?? 'fallback'`, no `?? {}`, no `?? []`, no `withDefaults`, no optional `?` on the prop, no `includes()`-narrowing helper, no `v-if` to guard the prop's existence.

## The rule

A schema field defined with `default: X` is **always present** at the component. **Picker-shaped fields** — `select`, `toggle_button`, `content_alignment`, `select_radio`, `checkbox`, and any other Studio control that emits from a closed value set — are additionally **always one of the declared values**.

**`object` fields** are always materialized as a plain object whose declared sub-fields are each populated by their own field-type contract — picker children at their default, `text` children at `''`, nested `object` children recursed, nested `array` children at `[]`. The contract is structural: it holds whether or not the `object` field itself declares a `default`. Emptiness lives one level down, at the leaves.

**`array` fields** are always materialized as an array — possibly `[]`, but never `null` or `undefined`. Same structural guarantee: it does not depend on a `default`. Type the prop as `T[]` and iterate directly; if you need an empty-state branch, check `.length`.

Type the prop as the exact literal union (pickers), exact object shape (objects), or `T[]` (arrays). Use it directly. Defensive code that re-checks these guarantees encodes states that cannot reach runtime.

```ts
// ✅ Schema
{ name: 'background', type: 'select',
  options: defineSelectOptions(['none', 'pale', 'solid']),
  default: 'none' }

// ✅ Component
interface Props {
  background: 'none' | 'pale' | 'solid'  // required, exact union
}
const props = defineProps<Props>()       // no withDefaults
const bgClass = {
  none:  '',
  pale:  'hero--bg-pale',
  solid: 'hero--bg-solid',
}[props.background]                      // exhaustive record lookup, no fallback
```

```ts
// ✅ Schema — object + array
{ type: 'object', name: 'headingStyle', schema: [{ fields: [
  { type: 'select', name: 'headingAs',
    options: defineSelectOptions(['h1', 'h2', 'h3']), default: 'h2' },
] }] },
{ type: 'array',  name: 'informationLinks', schema: [{ fields: [
  { type: 'text', name: 'label' },
  { type: 'link', name: 'href' },
] }] }

// ✅ Component
interface Props {
  headingStyle:     { headingAs: 'h1' | 'h2' | 'h3' }            // never optional
  informationLinks: { label: string; href: LinkValue }[]         // T[], not T[] | undefined
}
const props = defineProps<Props>()
// Direct access on the object, direct iteration on the array.
// Inner `label` text may be '' — guard at the leaf, not at the prop.
const items = computed(() =>
  props.informationLinks.map(l => ({ label: l.label, href: resolveLink(l.href) })),
)
```

## Why

A defaulted picker-shaped field carries two guarantees and one type benefit:

1. **Always present.** Frontend-core fills in the default whenever the author didn't pick one in Studio.
2. **Always one of the options.** Every Studio picker — `select`, `toggle_button`, `content_alignment`, `select_radio`, `checkbox` — emits only values from the declared option set. There is no "free text" path that could produce something off-list. `checkbox` is the degenerate case: the option set is `true | false`.
3. **Exact literal type.** Encoding the union as `'none' | 'pale' | 'solid'` (or `'top-left' | 'top-center' | …` for `content_alignment`) lets the TypeScript compiler enforce exhaustive `switch` and complete record lookups. The moment you widen the union with `?` or a fallback, exhaustiveness checks silently degrade.
4. **Structural materialization for `object` and `array`.** Frontend-core's prop-normalization step fills every `object` field with a fresh `{}` whose sub-fields are recursively populated by the same `getFieldFallback` machinery, and every `array` field with `[]`. There is no code path that delivers `undefined` or `null` for these shapes. The component receives the full structural skeleton; only leaf values inside can be "empty" by their own field type's rules.

Defensive fallbacks pretend the picker can emit garbage, or that the runtime forgot to materialize an object or array. They bloat the component with computed-property indirection, hide the schema contract from anyone reading the component, and make exhaustive-switch checking impossible.

## Forbidden patterns

```ts
// ✘ Runtime narrowing for impossible values
const safeVariant = computed<Variant>(() =>
  (['center', 'split'] as readonly string[]).includes(props.variant)
    ? props.variant
    : 'center',
)

// ✘ withDefaults re-asserting the schema default
const props = withDefaults(defineProps<Props>(), {
  background: 'none',  // schema already defaults to 'none'
  variant: 'center',   // schema already defaults to 'center'
})

// ✘ Optional prop typing on a schema-defaulted field
interface Props {
  variant?: 'center' | 'split'  // schema guarantees presence — drop the `?`
}

// ✘ Nullish coalescing on a defaulted field
const size = props.size ?? 'm'        // ?? branch is unreachable
class={`hero--bg-${props.background ?? 'none'}`}  // same — unreachable

// ✘ "Just in case" branch for an off-list value
if (props.background !== 'none' && props.background !== 'pale'
    && props.background !== 'solid') {
  return  // can't happen — value is always one of the three
}

// ✘ Nullish coalesce on an array prop
const items = (props.informationLinks ?? []).map(...)   // ?? branch unreachable

// ✘ Optional chaining on an array prop
props.informationLinks?.map(...)                        // the prop is always an array

// ✘ Optional prop typing on an object or array field
interface Props {
  headingStyle?:     { headingAs: 'h1' | 'h2' | 'h3' }   // drop the `?`
  informationLinks?: { label: string; href: LinkValue }[] // drop the `?`
}

// ✘ withDefaults re-asserting the structural fallback
const props = withDefaults(defineProps<Props>(), {
  headingStyle:     () => ({ headingAs: 'h2' }),         // runtime already does this
  informationLinks: () => [],                            // runtime already does this
})

// ✘ Nullish coalesce on an object prop
const style = props.headingStyle ?? { headingAs: 'h2' } // ?? branch unreachable

// ✘ v-if guarding existence of an object or array prop
<MyChild v-if="props.headingStyle" ... />               // it is always present
<ul v-if="props.informationLinks">…</ul>                // use v-if="links.length" if you need an empty-state branch
```

## When the value really can be absent

The "always materialized" guarantee for `object` / `array` and the "always present + on-list" guarantee for defaulted picker fields don't extend to every field type. **Leaf fields** without a `default` arrive empty-but-materialized:

| Field | Empty case |
| --- | --- |
| `text` / `textarea` / `richtext` with no `default` | Author left the input blank → empty string |
| `media` | Author hasn't picked an asset → empty media object |
| `link` | Author hasn't picked a target |
| `query` | No data source bound |

For these, normal guarding (`v-if="heading"`, `v-if="media?.url"`, `v-if="link.targetId"`) is correct. Note that even `media` and `link` are themselves materialized as objects — only their *contents* are empty.

**`object` and `array` fields are never absent.** Emptiness lives one level down: guard the inner leaves, never the object or array prop itself. For an array, `v-if="props.items.length"` is the right shape if you need an empty-state branch; `v-if="props.items"` is wrong because the array is always there.

## Red flags — STOP and trust the schema

Each of these means: delete the defensive code.

- About to write `?? 'something'` on a prop whose schema field has a `default`
- About to write `?? []` or `?? {}` on an `array` or `object` prop
- About to write `props.someArray?.map(...)` or `props.someObject?.foo` to "be safe"
- About to add `withDefaults` for a field that already has a schema `default`, or for any `object` / `array` field
- About to type a schema-defaulted prop with `?`, or to type an `object` / `array` prop as optional
- About to write `v-if="props.someArray"` or `v-if="props.someObject"` to guard the prop's existence (use `.length` if you need an empty-state branch)
- About to write `if (props.x === 'unexpected')` for a value not in `options`
- About to write a `safeX` computed wrapping a schema-defaulted prop
- About to add a `try/catch` or `console.warn` for an "unknown variant" branch

If the answer is "but what if the value is missing/invalid?" — re-read §Why. It cannot be. The schema contract is the type contract.

## How this surfaces if you ignore it

- **Unreachable branches** in coverage reports and linter output.
- **Degraded exhaustive checks**: `case 'none'` / `case 'pale'` / `case 'solid'` / `default:` becomes the only place an off-list value could land, but that off-list value cannot exist, so the `default` branch is dead code that prevents the compiler from telling you when a new option was added but a handler was forgotten.
- **Drift between schema and component**: someone renames an option in `section.ts`; the fallback in the component still references the old name, no error.
- **Reader confusion**: the next developer assumes the picker can emit garbage and copies the pattern to other components.
