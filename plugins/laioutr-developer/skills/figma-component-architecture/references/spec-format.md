# Component Spec Format

How to write a single component spec produced by `figma-component-architecture`. Covers when to introduce components not in the analysis, the trivial vs full spec templates, prop-boundary rules (parent-owned state, unions over flags, state transitions, i18n boundary, list item types), and the Vue API conventions every spec must follow.

## Introducing components not in the analysis

The analysis document from `figma-design-analysis` captures what's visible in Figma. Architecture sometimes requires components that aren't in any design — data-grouping wrappers, intermediate composition layers, or structural abstractions that emerge from data flow analysis.

**When to introduce:** During Phase 3, if the prop boundary check reveals that two or more sibling components always receive the same scoped data (e.g., `DeliveryEstimate` and `ProductItemList` both need data for a single delivery group), introduce a composition wrapper to mediate the data flow. Also introduce components when the user proposes them during Phase 2/3 review.

**How to document:** Mark introduced components with `(introduced — not in analysis)` in the spec. Include a one-sentence justification explaining why the component is architecturally necessary. Classify as reusable or sub-component using the standard removal test.

## Trivial component spec (one-liner)

For pure presentational components with no internal state, no events beyond click, no slots, and whose prop interface is self-explanatory from types alone:

```markdown
**PriceDisplay** _(reusable)_ — Renders formatted price with optional strikethrough for original price.
Props: `{ price: Money; originalPrice?: Money; size?: 's' | 'm' | 'l' }`
```

## Complex component spec (full)

For components with internal state, slots, events, or compound composition:

```markdown
### ComponentName _(reusable)_ or _(sub-component of ParentName)_

**Purpose:** One sentence describing what this component does.

**Props:**
\```typescript
interface ComponentNameProps {
  // Data props — the component's data contract
  items: ItemType[];
  selectedId?: string;

  // UI configuration props
  size?: 's' | 'm' | 'l';
  variant?: 'default' | 'compact';

  // v-model props (Vue 3.4+ defineModel)
  // modelValue is implicit via defineModel
}
\```

**Slots:**
| Name | Scoped Data | Purpose |
|---|---|---|
| `default` | — | Main content area |
| `item` | `{ item: ItemType; index: number; isSelected: boolean }` | Custom rendering per item |
| `empty` | — | Shown when items array is empty |

**Events:**
| Event | Payload | When |
|---|---|---|
| `select` | `[id: string]` | User selects an item |
| `remove` | `[id: string]` | User clicks remove on an item |

**Internal State:** _(mutable state only — values that change in response to user interaction)_
- `expandedId: Ref<string | null>` — tracks which item is expanded (UI-only)
- `isAnimating: Ref<boolean>` — prevents interaction during transition
- _(Computed/derived values from props, like overflow counts or formatted labels, are implementation details — don't list them here.)_

**Context Sharing:** _(only if compound component)_
- Provides: `{ size: ComputedRef<'s' | 'm' | 'l'>; expandedId: Ref<string | null> }` via createContext
- Children consume via `injectComponentNameContext()`

**Composition:**
- Imports `ChildA`, `ChildB` directly
- `item` slot allows parent to override default item rendering
- Uses reka-ui `AccordionRoot` for keyboard navigation
- Default slot with structural wrapping — each child wrapped in reka-ui `AccordionItem` for coordination _(when applicable)_
```

## Parent-owned state as props

States that depend on operations or decisions owned by the parent (async mutations, form validation timing, submission attempts) are not internal — they must be received as props. The test: does the component itself decide *when* to show this state? If the parent triggers or controls when the state appears, it's a prop.

**Async operation status:** For components that trigger async operations via events (e.g., add-to-cart, form submit), accept a `status` prop from the parent. The parent owns the mutation and controls the status. The component may use internal state for timed visual transitions (e.g., reverting a success indicator after a delay).

```typescript
status?: 'idle' | 'loading' | 'success' | 'error';
```

**Validation errors** are almost always props because the parent controls when validation runs (on form submit, on blur, on field change). A `ShippingMethodSelector` knows whether nothing is selected, but the parent decides when to show the error — the component receives `error?: string` as a prop.

## Prefer unions over boolean flags

When a prop has mutually exclusive states, use a discriminated or literal union instead of separate boolean flags. Unions make invalid states unrepresentable and produce cleaner APIs.

```typescript
// ❌ BAD: boolean flag + separate value
value: Money;
isFree: boolean;  // if true, value is ignored — confusing

// ✅ GOOD: literal union
value: Money | 'free';

// ❌ BAD: flat flags for grouped metadata
icon?: string;
code?: string;
badgeText?: string;

// ✅ GOOD: composed type
badge?: { icon?: string; text: string };
```

Apply the same principle to event payloads and slot scoped data. If two fields are always used together, group them. If a boolean exists only to disable/ignore another prop, merge them into a union.

## State transitions for multi-state components

When a component's state prop has 3+ values with non-obvious transitions, document valid transitions in the spec. Include which event triggers each transition and which transitions are invalid. Without this, the implementer and the Section/Block wrapper author may disagree on valid transitions.

```markdown
**State Transitions:**
`closed` --(@toggle)--> `open` --(@redeem)--> `redeemed` --(@undo)--> `open`
Invalid: `closed` -> `redeemed` (must open first)
```

Binary toggles (open/closed) don't need this — the transitions are obvious.

## Static text vs data props (i18n boundary)

Static user-facing text (headings like "Thank you for your order!", section titles like "Order summary", button labels like "Continue shopping") is an i18n implementation detail — it lives inside the component as a locale key (`$t('thankYou.heading')`), NOT as a prop. Do not spec `heading?: string` for text that is the same across all usages.

Dynamic data embedded in user-facing text (order numbers, names, counts, dates) should be individual props. The text template containing the dynamic data is handled by i18n interpolation inside the component (`$t('thankYou.subline', { orderNumber })`).

| Text type | Spec as | Example |
|---|---|---|
| Fully static | Not a prop — i18n key inside component | "Order summary" heading |
| Static with dynamic data | Individual props for each dynamic value | `orderNumber: string` (template: "Order #{orderNumber}") |
| Fully dynamic (varies per usage) | `string` prop | `title: string` on a generic card |

When unsure, check Phase 1 findings for how existing components handle similar text. If a component has a mandatory `title` prop that callers always set to the same i18n string, it should probably be internal.

## List item types

When a parent component renders a list of children, the item type should be the child component's props interface (or a subset via `Pick<>`), not a domain entity type. If the item shape starts resembling a canonical entity rather than a component contract, that is a sign the mapping responsibility has leaked from the Section/Block wrapper into the component.

**When the child component's interface is unknown (cross-plan dependency):** If the child is planned in another analysis with no architecture spec yet, prefer a **slot** over an array prop. A slot defers the interface decision to the parent (Section/Block wrapper), which will know the child's final props. Mark the slot with a note: *"slot defers to parent because [ChildComponent] props are not yet specced — switch to array prop once [reference] architecture is complete."* If the parent needs to control layout around the list items (e.g., gap, separator), spec the slot with structural wrapping guidance. Only use a guessed array prop if the child's behavioral description in the referenced analysis makes the props obvious and stable.

**Reuse child props types for composition:** When a parent composes a child 1:1 (e.g., `DeliveryGroup` contains one `DeliveryEstimate`), accept the child's props interface as a single prop rather than duplicating its fields. This keeps the parent's API in sync with the child and avoids field drift.

```typescript
// ❌ BAD: duplicating child fields
interface DeliveryGroupProps {
  title: string;
  estimatedDelivery: Timespan;  // duplicated from DeliveryEstimate
  providerLogo?: Media;         // duplicated from DeliveryEstimate
  items: CheckoutProductItemProps[];
}

// ✅ GOOD: reference child's props type
interface DeliveryGroupProps {
  title: string;
  deliveryEstimate: DeliveryEstimateProps;
  items: CheckoutProductItemProps[];
}
```

This also applies to list items: `items: ChildComponentProps[]` is better than defining an inline type that mirrors the child's interface. The exception is when the parent uses only a subset — use `Pick<ChildProps, ...>[]` to narrow.

**Interface narrowing for reduced-mode usage:** When a parent uses a child in a reduced mode (e.g., only 1 of 9 line item types, read-only without interactive controls), spec the parent's props to match only its actual usage — not the child's full interface. Note the child's broader capability for future extension. This prevents the parent's API from exposing unused complexity.

## Vue API conventions (from figma-to-component)

| API | Usage |
|---|---|
| `defineProps<T>()` + `withDefaults()` | All props with defaults |
| `defineEmits<{ event: [payload] }>()` | Vue 3.3+ tuple syntax, never legacy call-signature |
| `defineModel<T>('name')` | Two-way `v-model` binding (Vue 3.4+) |
| `defineSlots<{ name(props: T): any }>()` | Typed slot declarations |
| `createContext` from reka-ui | Compound component state sharing |

## Presentation order

Group specs by implementation phase from the analysis document. After presenting each group, **stop and wait for explicit approval** before presenting the next group — do not interpret "looks good" on a previous group as blanket approval for all remaining groups. Ask: *"Does this group look right, or should I adjust anything before moving on?"*
