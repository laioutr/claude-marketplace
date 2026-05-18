# Vue SFC: Keep All Imports in One Script Block

All `import` statements in a Vue SFC must live in a **single** `<script>` block. Prefer `<script setup>`.

Having imports split across `<script lang="ts">` (for `export interface`) and `<script setup lang="ts">` breaks `import-x/order` in a way `eslint --fix` cannot repair.

## The trap

```vue
<!-- ❌ Don't do this -->
<script lang="ts">
import { $money } from '#imports';
import type { QuantityPickerProps } from '#ui-kit/components/QuantityPicker/types';
import Badge from '#ui-kit/components/Badge/Badge.vue';

export interface FooProps { /* ... */ }
</script>

<script setup lang="ts">
import Text from '#ui-kit/components/Text/Text.vue';
import Icon from '#ui-kit/components/Icon/Icon.vue';
// ...
</script>
```

`eslint-plugin-import-x` flattens imports from both blocks into one list and sorts them against the configured groups (`builtin`, `external`, `sibling`, `internal`, `object`, `type`, `parent`, `index`). When block 1's `#imports` (internal) appears before block 2's `#ui-kit/.../Text.vue` (also internal but alphabetically earlier in the configured order), the rule reports:

> `#imports` import should occur after import of `#ui-kit/components/Text/Text.vue`

`--fix` cannot solve this — reordering would require moving imports **across script blocks**, which the autofixer will not do. Your pre-commit hook (or CI lint step) then rejects the commit.

## The rule

- **All imports live in `<script setup>`** — including type imports (`import type { ... }`).
- **`export interface` / `export type` / `export const` declarations** also live in `<script setup>` (Vue 3.3+ hoists them correctly — see `ProductSlider.vue` for a canonical example).
- **No `<script lang="ts">` block** unless you have a very specific reason (e.g. `defineOptions` equivalent that setup can't express). If you do keep one, it must contain **zero imports**.

## The fix

Consolidate into a single `<script setup>` block. Types and components imported in the merged block are available for both `export interface` declarations and setup logic.

```vue
<!-- ✅ Single setup block -->
<script setup lang="ts">
import Badge from '#ui-kit/components/Badge/Badge.vue';
import Icon from '#ui-kit/components/Icon/Icon.vue';
import QuantityPicker from '#ui-kit/components/QuantityPicker/QuantityPicker.vue';
import Text from '#ui-kit/components/Text/Text.vue';
import { $money } from '#imports';
import type { QuantityPickerProps } from '#ui-kit/components/QuantityPicker/types';

export interface FooProps {
  quantityPicker?: QuantityPickerProps;
  // ...
}

defineProps<FooProps>();
</script>
```

## Why not keep two blocks with imports only in setup?

It would compile. But it preserves a two-block structure that no longer earns its keep — the non-setup block becomes a bare `export` container. Simpler to delete it. Reviewers don't have to ask "why two blocks?" and future editors won't accidentally re-introduce an import in the wrong block.

## Exception

Page-level `definePageMeta` callers or rare cases where the non-setup block carries compiler directives that `<script setup>` genuinely cannot express. These are extremely rare in this repo — if you think you need one, check first.

## Do not consolidate opportunistically

When you read or edit a Vue SFC that already has two script blocks **but isn't the file you were sent in to change**, leave the structure alone. Do not consolidate it into a single `<script setup>` block as a drive-by cleanup.

Why:

1. **Scope creep.** A two-block consolidation rewrites every import line and every `defineProps` / `defineEmits` location. Bundling it into an unrelated change makes the diff unreviewable and the commit hard to revert.
2. **Risk without coverage.** Consolidation looks mechanical but isn't — `export interface` hoisting, `<script setup>`-only macros, and `<script lang="ts">`-only directives behave subtly differently. A change that "should be a no-op" can flip prop typing or break consumer imports of the exported interface. Doing this on a file you don't own and aren't testing is a way to ship silent regressions.
3. **The rule only binds at the moment of authoring or modifying imports.** A pre-existing two-block file that compiles, lints, and runs is not in violation of this rule today; it only becomes a problem if someone adds an import to the wrong block. Leave it until that happens, or until the file is the explicit target of a refactor.

When *is* consolidation appropriate?

- The file is the explicit target of the current task and you are already touching its imports/props.
- A linter or commit hook is failing on this file because of the split-block trap above.
- A rename or move workflow already mandates rewriting the script block (e.g. moving a component between packages).

In all other cases — including "I noticed this file has two blocks while reading it" — do not refactor. The rule exists to prevent introducing the trap, not to retroactively rewrite every file in the repo.
