# Trust the Schema in Components

When you add a schema field with a `default` value, frontend-core guarantees the value reaches the consuming component already populated, and picker-shaped fields additionally guarantee the value is one of the declared options. The component must trust that contract — no `?? 'fallback'`, no `withDefaults`, no optional `?` on the prop, no `includes()`-narrowing helper.

## The rule

A schema field defined with `default: X` is **always present** at the component. **Picker-shaped fields** — `select`, `toggle_button`, `content_alignment`, `select_radio`, `checkbox`, and any other Studio control that emits from a closed value set — are additionally **always one of the declared values**.

Type the prop as the exact literal union and use it directly. Defensive code that re-checks these guarantees encodes states that cannot reach runtime.

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

## Why

A defaulted picker-shaped field carries two guarantees and one type benefit:

1. **Always present.** Frontend-core fills in the default whenever the author didn't pick one in Studio.
2. **Always one of the options.** Every Studio picker — `select`, `toggle_button`, `content_alignment`, `select_radio`, `checkbox` — emits only values from the declared option set. There is no "free text" path that could produce something off-list. `checkbox` is the degenerate case: the option set is `true | false`.
3. **Exact literal type.** Encoding the union as `'none' | 'pale' | 'solid'` (or `'top-left' | 'top-center' | …` for `content_alignment`) lets the TypeScript compiler enforce exhaustive `switch` and complete record lookups. The moment you widen the union with `?` or a fallback, exhaustiveness checks silently degrade.

Defensive fallbacks pretend the picker can emit garbage. They bloat the component with computed-property indirection, hide the schema contract from anyone reading the component, and make exhaustive-switch checking impossible.

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
```

## When the value really can be absent

The rule applies to fields **with a `default` set** (and, for picker-shaped fields, an `options` list). Other shapes legitimately can be empty:

| Field | Empty case |
| --- | --- |
| `text` / `textarea` / `richtext` with no `default` | Author left the input blank → empty string |
| `media` | Author hasn't picked an asset → empty media object |
| `link` | Author hasn't picked a target |
| `query` | No data source bound |
| `object` with no inner defaults | Each child handles its own emptiness |

For these, normal guarding (`v-if="heading"`, `v-if="media?.url"`) is correct. The rule is about fields the schema **defaults**, not about whether emptiness is meaningful for the concept.

## Red flags — STOP and trust the schema

Each of these means: delete the defensive code.

- About to write `?? 'something'` on a prop whose schema field has a `default`
- About to add `withDefaults` for a field that already has a schema `default`
- About to type a schema-defaulted prop with `?`
- About to write `if (props.x === 'unexpected')` for a value not in `options`
- About to write a `safeX` computed wrapping a schema-defaulted prop
- About to add a `try/catch` or `console.warn` for an "unknown variant" branch

If the answer is "but what if the value is missing/invalid?" — re-read §Why. It cannot be. The schema contract is the type contract.

## How this surfaces if you ignore it

- **Unreachable branches** in coverage reports and linter output.
- **Degraded exhaustive checks**: `case 'none'` / `case 'pale'` / `case 'solid'` / `default:` becomes the only place an off-list value could land, but that off-list value cannot exist, so the `default` branch is dead code that prevents the compiler from telling you when a new option was added but a handler was forgotten.
- **Drift between schema and component**: someone renames an option in `section.ts`; the fallback in the component still references the old name, no error.
- **Reader confusion**: the next developer assumes the picker can emit garbage and copies the pattern to other components.
