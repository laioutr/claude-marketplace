# Vue `defineProps` and Cross-Package Types

A Nuxt module build fails with these errors when `defineProps<T>()` resolves through a type imported from another package:

```
[vue-sfc-transformer] Failed to resolve extends base type.
[vue-sfc-transformer] Unresolvable type reference or unsupported built-in utility type
```

`vue-sfc-transformer` can't walk the type graph across package boundaries (published `node_modules` entries, workspace symlinks, etc.). The error names no file. `typecheck` passes; only the actual Nuxt-module build reveals the problem — run `nuxi build` or `pnpm build` (or `turbo run <pkg>#build` if you're in a monorepo).

## Rule

The type passed to `defineProps<...>()` must be resolvable inside the file's own package. Don't pass it a type that is — directly, via `extends`, or wrapped in `Omit<>` / `Pick<>` / `Partial<>` / intersection — sourced from another package.

## Fix

Inline the field-level types you need. Field-level imports across packages are fine; aggregate-interface imports as a `defineProps` base are not.

```ts
// ❌
import type { Base } from '#ui-kit/components/Foo/Foo.vue';
export interface Props extends Base { heading: string }
// also broken: Omit<Base, 'x'> & {...}, Base & {...}, Pick<Base, 'y'>

// ✅
import type { ImageContrastOverlayProps } from '#ui-kit/components/ImageContrastOverlay/types';
import type { Media } from '@laioutr-core/core-types/common';
export type Props = {
  backgroundImage?: Media;        // fields previously inherited from Base,
  overlay?: ImageContrastOverlayProps;  // inlined here
  heading: string;
};
```

## Don't

`/* @vue-ignore */` (the error's own suggestion) silently turns the base props into runtime fallthrough attrs — Studio schemas, Storybook controls, and TypeScript consumers all stop seeing them. Never use it.

## Verify

Run the actual Nuxt-module build for the affected package (not just `typecheck`). Ecosystem issue still open upstream: nuxt/nuxt [#30213](https://github.com/nuxt/nuxt/issues/30213), nuxt/ui [#2291](https://github.com/nuxt/ui/issues/2291).
