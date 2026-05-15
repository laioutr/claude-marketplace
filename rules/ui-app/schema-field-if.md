# Schema Field `if` — Conditional Visibility

`StudioFieldDefinition` and `StudioFieldsetDefinition` accept an optional
`if?: SchemaCondition` property. When the expression evaluates to **falsy**,
Studio hides the field (or fieldset) from the sidebar. The data is **not**
deleted from storage — only the control disappears.

Use `if` to remove controls that aren't meaningful for the current
configuration (e.g. hide `customBackground` when `background !== 'custom'`).

## Where the expression sees its data

The expression evaluates against an **unwrapped** view of the section/block
values, shaped to match what the component receives as props for most field
types:

- `text` / `textarea` / `richtext` → active locale's string.
- `checkbox` → boolean.
- `select` / `radio` / `toggle_button` → option's string value.
- `number` → number, or `undefined` when unset.
- `icon` → icon name string, or `undefined`.
- `color` / `link` / `media` → the plain object the component receives
  (`ColorFieldValue` / `Link` / `MediaImage | MediaVideo`). The storage
  `{ subtype, value }` wrapper is stripped, same as `rcPageToRender` does
  on the frontend.
- `json` → parsed JSON value, or `null`.
- `object` → plain object with nested fields unwrapped recursively.
- `array` → array of unwrapped item objects (without the `.id` the
  component receives on each item).
- `query` → **raw query reference** `{ type: 'entity-set', queryId, link?,
  limit? }`, NOT the resolved Orchestr data. `if` can test "is a query
  configured" but not inspect entity contents.

Missing entries fall back to the per-type runtime fallback (see
`CLAUDE.md` → "Default values vs runtime fallbacks").

## Path syntax

Inside a nested `object` / `array` field, scope is the **innermost
enclosing object** (or array item). Reach further out with prefixes:

| Path                     | Resolves from               |
| ------------------------ | --------------------------- |
| `['get', 'foo']`         | Current scope               |
| `['get', '^foo']`        | Parent scope (one up)       |
| `['get', '^^foo']`       | Grandparent scope           |
| `['get', '/foo']`        | Section/block root          |
| `['get', 'obj.nested']`  | Nested key inside scope     |
| `['get', 'arr[0].name']` | Array indexing inside scope |

Example from the codebase:

- `SectionQuoteCardSlider.vue` — `cardStyle.starColor` field:
  `['get', '/showStarRating']` (root-level checkbox from inside `cardStyle`).

## Common patterns

```ts
// Mode-switch dependent — show only when `mode === 'grid'`
if: ['==', ['get', 'mode'], 'grid']

// Set-membership — the inner `['basic', 'compact']` is wrapped in
// an extra `[…]` so the engine reads it as an array literal instead of
// an operator call (`arr-of` works too — see the operator note below)
if: ['in', [['basic', 'compact']], ['get', 'variant']]

// Truthy check. At `if`-toplevel `['bool', x]` is equivalent to `x`
// (the engine coerces the result via `Boolean(result)`). Use `bool`
// to make the intent explicit, or drop it for terseness.
if: ['bool', ['get', 'icon']]

// Negation — `!=` is shorter than wrapping `==` in `!`
if: ['!=', ['get', 'background'], 'none']

// Boolean checkbox — bare get is enough (returns the boolean directly)
if: ['get', 'showProductAmount']

// Empty array check — `eq` uses deep equality, so the literal escape
// `[[]]` (which evaluates to `[]`) works. Raw `[]` evaluates to
// `undefined`, so it must be wrapped.
if: ['==', ['get', 'items'], [[]]]
```

## Available operators

The Studio engine registers `defaultOperators` plus the array and type
bundles from `@laioutr/expression`. Aliases in parentheses.

| Category   | Operators                                                                  |
| ---------- | -------------------------------------------------------------------------- |
| Input      | `get` (`$`), `get?` (`$?`), `?get` (`?$`)                                  |
| Logical    | `and` (`&&`), `or` (`\|\|`), `not` (`!`), `coalesce` (`??`)                |
| Comparison | `eq` (`==`), `ne` (`!=`), `gt` (`>`), `ge` (`>=`), `lt` (`<`), `le` (`<=`) |
| Branching  | `if` (`?`), `cond`                                                         |
| Container  | `len`, `member` (`[]`)                                                     |
| Array      | `in`, `arr-of`, `slice`, `join`, `map`                                     |
| Type       | `bool`, `num`, `str`, `type`                                               |

`in` signature: `['in', haystack, needle]` — array first, value second.
For a static haystack you can wrap it in an extra `[…]` to pass it as a
literal (`[['a', 'b', 'c']]`); you can also use `arr-of`
(`['arr-of', 'a', 'b', 'c']`). `arr-of` is required when haystack
elements are themselves expressions needing evaluation
(`['arr-of', ['get', 'x'], 'b']`).

`bool` (`!!x`) is plain JavaScript truthy coercion. At `if`-toplevel it
is redundant — the engine wraps the result in `Boolean(result)` already
(`evaluateCondition.ts`), so `if: ['bool', x]` ≡ `if: x` and
`['!', ['bool', x]]` ≡ `['!', x]` (because `not` is implemented as
`!evalExpr(arg)` in the expression engine's logical-ops module). The same
goes for sub-args of `and`/`or` — those also do JS truthy coercion
internally.

`bool` is meaningful only when you need a literal `true`/`false` value
**inside a larger expression** (`== false`, `arr-of`, etc.). For
"is this field set?" the bare `['get', 'x']` (or `['!', ['get', 'x']]`)
is equivalent and shorter. Keep `bool` when it makes the intent more
readable; drop it for terseness.

When you need a specific match (`== ''`, `== false`, `== '<select
option>'`) reach for `eq`/`ne` directly.

Operators and arities are validated **at prepare time** — typos or wrong
arity throw synchronously, not at first invocation.

## Forbidden / pitfalls

1. **No `undefined` literal, and no array literal.** JSON expressions
   have no `undefined`. Raw arrays inside an expression are interpreted by
   the engine, not used as data: `[]` resolves to `undefined`, `[x]`
   resolves to `x` (the literal escape), `[op, …]` is an operator call.
   To pass a literal array as data, wrap it in `[[…]]` or build it with
   `arr-of`. For "is unset", use `['!', ['get', 'x']]` (relies on `!`'s
   truthy coercion) or compare against the type's fallback (`''`, `false`,
   `null`, `[]` via `[[]]`, first-option string).

2. **Fail-open by design.** Any thrown error (typo, missing path with no
   `get?` / `?get` guard, malformed shape) returns `true` and always logs
   a warning to the Studio browser console (`console.warn` is
   unconditional in `evaluateCondition.ts`). A _hidden_ control is
   invisible; a _shown-but-broken_ control is loud and gets reported.
   Don't rely on `if` for security — it's UX.

3. **Don't read `props.surfaceTone`-style data through `if`.** `if`
   resolves against the **schema's own values** (the unwrapped storage).
   It cannot see Vue computed properties, runtime context, or values
   derived after rendering. Keep `if` to declarative dependencies on
   sibling/ancestor schema values.

## Per-type fallbacks the unwrap injects

`unwrapForCondition` walks the **schema** (not storage), so every field
declared today has a known value in the unwrapped output — even on
sections created before the field was added to the schema. Reliable
"is unset" checks per type:

| Field type                                                               | Fallback when unset                           | Canonical "unset" check                           |
| ------------------------------------------------------------------------ | --------------------------------------------- | ------------------------------------------------- |
| `text`, `textarea`, `richtext`                                           | `''`                                          | `['!', ['get', 'x']]`                             |
| `checkbox`                                                               | `false` (or `true` for visibility decorators) | `['!', ['get', 'x']]`                             |
| `select`, `radio`, `toggle_button`                                       | First option's value                          | `['==', ['get', 'x'], '<first option>']`          |
| `object`                                                                 | Object of recursive fallbacks                 | Drill into a leaf                                 |
| `array`                                                                  | `[]`                                          | `['==', ['get', 'x'], [[]]]` or `['len', …] == 0` |
| `json`                                                                   | `null`                                        | `['==', ['get', 'x'], null]`                      |
| `number`, `icon`, `media`, `link`, `query`, `color`, `content_alignment` | `undefined`                                   | `['!', ['get', 'x']]`                             |

For every field type except `select`/`radio`/`toggle_button` and `array`
(whose fallbacks are truthy), `['get', 'x']` is itself a truthy "is set"
test at `if`-toplevel and `['!', ['get', 'x']]` is the "is unset" test;
the `bool`-wrapped forms are equivalent (see `bool` note above).

For nested object fields, the unwrap recursively fallbacks the entire
subtree, so `['get', 'cta.text']` returns `''` reliably even if `cta`
has never been stored.

## Relationship to visibility decorators

`visibility` decorators (`visibilityField({ for, name? })`) and `if` are
**separate patterns** that answer different questions. They are not
alternatives.

| Pattern                         | What it does                                                                                                                                                                                                                                                                                                                         |
| ------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `visibilityField({ for: 'x' })` | Renders a checkbox in the Studio sidebar grouped with the target field. **Does not** affect Studio sidebar rendering of the target — the component template reads the checkbox value (`xVisible`) to decide whether to render the target's value on the frontend. A frontend render-time toggle expressed through a sidebar control. |
| `if`                            | Hides a field/fieldset's **Studio sidebar control** based on an expression over other schema values. No effect on stored data; data is preserved across hide/show. No effect on frontend rendering — that's whatever the component template does with the stored value.                                                              |

The visibility decorator is a render-time toggle wired through component
code. `if` is a Studio-authoring conditional. A field can have both:
the editor sees the control (when `if` is true), and the editor's visibility
checkbox separately controls whether the frontend renders the value.

### Hard rule: never migrate one to the other

Do **not** convert a `visibilityField` into an `if`, and do **not**
convert an `if` into a `visibilityField`. They aren't alternatives — they
do different things. The choice is per field by UX or dev based on which
question the field is actually answering:

- "Should the frontend render this content right now?" → visibility
  decorator (the checkbox controls component template logic).
- "Should the editor see this control in Studio right now?" → `if`
  (declarative, based on other field values).

Removing a `visibilityField` in favor of a content `if` deletes the
editor's render-time toggle and changes the component contract (the
`xVisible` prop the template reads disappears). Replacing an `if` with a
`visibilityField` adds a frontend checkbox no one asked for and doesn't
hide the Studio control the way `if` does.

## Lives where

- Type: `SchemaCondition` in `@laioutr-core/core-types/fields`. Derives
  the operator union from `@laioutr/expression`'s `defaultOperators` +
  array + type bundles via `type` imports — no runtime dependency.
- Runtime engine: `apps/cockpit/src/features/project/studio/schema-conditions/engine.ts`
  instantiates `JsonExpression` with the matching operator bundles. The
  type and runtime are configured separately; keep them in sync if you
  add another operator bundle.
- Studio integration: `apps/cockpit/src/features/project/studio/schema-conditions/`
- Section/block authoring: anywhere in your module's ui-app-layer (typically `src/runtime/app/section/`, `src/runtime/app/block/`, `src/runtime/app/shared-fields/`)

## Related

- `section-config-standard.md` — Rules panel sits in Studio's standard
  layout; visibility conditions are part of the broader Rules concept.
- `forbidden-field-names.md` — Field naming caveats `if` doesn't change.
