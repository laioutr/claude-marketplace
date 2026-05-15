# No reka-ui Types on Public Component Surfaces

`reka-ui` is an **implementation detail** of `ui-kit`. Its types must not appear in any `ui-kit` component's public API. Consumers of a `ui-kit` component should never need `reka-ui` in scope to satisfy a prop, slot, or emit signature.

## The rule

Inside any ui-kit-layer component in your Nuxt module, types imported from `reka-ui` must not appear in any of the following positions:

1. **`defineProps<T>()`** — directly, via `extends`, intersection, `Omit<>` / `Pick<>` / `Partial<>`, or indexed access (`Foo['bar']`).
2. **`defineEmits<T>()`** — same as above.
3. **An exported `interface` or `type`** that the `<script setup>` macro consumes (e.g. `export interface FooProps { x: RekaThing }` then `defineProps<FooProps>()`).
4. **A typed `defineSlots<T>()`** signature where slot props expose reka-ui types to the consumer's template.
5. **A barrel re-export** — `export ... from 'reka-ui'`, `export type * from 'reka-ui'`, or any re-export of a reka-ui symbol from `index.ts` / package entry.

Runtime imports of reka-ui (`import { Primitive, ConfigProvider, TooltipArrow } from 'reka-ui'`) are fine — that's the whole point of the dependency. The rule is strictly about the **type surface**.

## How to fix a leak

Inline the fields the wrapper actually exposes. Mirror reka-ui's shape locally rather than borrowing its type.

```ts
// ❌ Leaks PrimitiveProps onto the public surface
import type { PrimitiveProps } from 'reka-ui';

defineProps<
  PrimitiveProps & {
    brightness?: BackgroundBrightnessInput;
  }
>();
```

```ts
// ✅ Local interface, identical runtime behavior
import type { Component } from 'vue';

export interface OnBackgroundProps {
  as?: string | Component;
  asChild?: boolean;
  brightness?: BackgroundBrightnessInput;
}

defineProps<OnBackgroundProps>();
```

For wrappers that pass props through with `v-bind`, narrow the public type to the fields that are _actually meant to be configured by consumers_. Anything the wrapper hardcodes or controls itself should not appear in the local interface, even if reka-ui's type lists it.

```ts
// ❌ AppProps advertises ConfigProviderProps, but App.vue
//    hardcodes useId/locale/dir — those fields would be silently ignored.
import type { ConfigProviderProps } from 'reka-ui';
export interface AppProps {
  config?: ConfigProviderProps;
}
```

```ts
// ✅ Only the field consumers can actually influence
export interface AppConfigProps {
  scrollBody?: boolean | { padding?: boolean | string; background?: boolean | string };
}
export interface AppProps {
  config?: AppConfigProps;
}
```

For indexed accesses like `TooltipArrowProps['width']`, replace with the resolved primitive (`number`, `string`, etc.). The indirection is never worth keeping.

## Why

1. **Layer boundary.** `ui-kit` is the foundation every layer above depends on. A reka-ui type on a public prop transitively makes every consumer (`ui`, `ui-app`, app code) take a structural dependency on reka-ui's exports. A reka-ui minor version that renames a field or tightens a union becomes a breaking change for the whole monorepo without the wrapper file changing.
2. **Honest API surface.** `v-bind="props.config"` will silently pass any field the consumer provides — but our type is a contract about _what consumers should configure_. Advertising the full reka-ui type when the wrapper hardcodes half of it is a lie. Local interfaces document the real configuration surface.
3. **No surprise re-exports.** When the prop type is borrowed, the type's transitive references (`AsTag`, `AcceptableValue`, etc.) leak into our `.d.ts` output. Consumers' IDEs auto-import from `reka-ui` on completion. Local types keep our `.d.ts` self-contained.
4. **Replaceability.** If we ever swap reka-ui for a different headless library — or drop it for a small set of components — local prop types mean the wrapper changes, the consumers don't.

## How to spot a leak

Before opening a PR that touches a `ui-kit` component, grep its file:

```sh
rg "from 'reka-ui'" src/runtime/components/<Name>/<Name>.vue
```

For each match, check:

- Is it `import type` or a `type` modifier inside a value import?
- Does the imported name appear in `defineProps`, `defineEmits`, `defineSlots`, or in an exported `interface` / `type`?

If yes to both, it's a leak. Inline the shape locally.

A repo-wide audit is one command:

```sh
rg -l "import type .* from 'reka-ui'" src
rg -l ", type [A-Z][A-Za-z]+(Props|Emits|Context).*from 'reka-ui'" src
```

## Exceptions

- **Intra-`ui-kit` shared types.** If two `ui-kit` components legitimately need to share a reka-ui-derived type internally (e.g. a context shape consumed by sibling sub-components), import it inside the implementation but **don't re-export it from a public barrel**. Internal coupling is fine; public coupling is not.
- **`createContext` from reka-ui at runtime.** This is a runtime utility, not a type leak. Use it freely.

## Related rules

- `reka-ui.md` — when to use reka-ui, styling via data attributes
- `reka-ui-wrapper-patterns.md` — the typed `Props` export pattern, which this rule constrains
- `package-role.md` — `ui-kit` as the bottom layer of the three-layer architecture
- `../vue-sfc-cross-package-props.md` — separate trap: cross-package types in `defineProps` break the build
