# Story Icons Must Exist in the Theme Icon List

Every icon name used in a `.stories.ts` for a ui-kit-layer or ui-layer component must be a key from the `IconName` union exported by `@laioutr-core/ui-kit` (the union is mapped per theme — `laioutr`, `classic`, `shared`, etc.).

Applies to every `icon`, `iconLeft`, `iconRight`, `iconName`, `iconOnly` prop, `<Icon name="…">` binding, and every `.icon` field in data fixtures (`subMenuItems`, `icons`, etc.).

## Check before writing

Import the type and let TypeScript narrow the candidate:

```ts
import type { IconName } from '@laioutr-core/ui-kit';

const candidate: IconName = 'arrows/arrow-right'; // type-checks only if the key exists
```

Or grep the resolved package types under your project's `node_modules` if you need a quick listing of all valid keys:

```bash
rg -oN "'[a-z][a-z-]*/[a-z][a-z0-9-]*'" \
  node_modules/@laioutr-core/ui-kit/dist/runtime/app/types/theme.d.ts | sort -u
```

If the candidate name is not in that list, pick one that is. Do not invent a new key in a story.

## Do not

- Add SVGs to your local theme map or override `@laioutr-core/ui-kit`'s icon map to make a story work — icon additions are design-owned upstream. Raise the gap with the platform team instead.
- Swap to a direct Iconify name (`noto:*`, `solar:*`, `lhuge:*`, etc.) to bypass the theme map. Use direct names only where the surrounding story file already uses that pattern.
