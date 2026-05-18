# Story Icons Must Exist in the Theme Icon List

Every icon name used in a `.stories.ts` must be a key from the `IconName` union exported by `@laioutr-core/ui-kit` (the union is mapped per theme — `laioutr`, `classic`, `shared`, etc.).

Applies to every `icon`, `iconLeft`, `iconRight`, `iconName`, `iconOnly` prop, `<Icon name="…">` binding, and every `.icon` field in data fixtures (`subMenuItems`, `icons`, etc.).

## Check before writing

Import the type and let TypeScript narrow the candidate:

```ts
import type { IconName } from '@laioutr-core/ui-kit';

const candidate: IconName = 'arrows/arrow-right'; // type-checks only if the key exists
```

For a quick browse of all available icons, query the official docs MCP server (`laioutr-docs`) — it indexes the icon catalogue per theme. The upstream Storybook (<https://storybook.laioutr.cloud/>) also lists the icons visually.

If the candidate name is not in `IconName`, pick one that is. Do not invent a new key in a story.

## Do not

- Add SVGs to your local theme map or override `@laioutr-core/ui-kit`'s icon map to make a story work — icon additions are design-owned upstream. File an issue on `laioutr/ui-source` (or contact the Laioutr team) to add the icon canonically.
- Swap to a direct Iconify name (`noto:*`, `solar:*`, `lhuge:*`, etc.) to bypass the theme map. Use direct names only where the surrounding story file already uses that pattern.
