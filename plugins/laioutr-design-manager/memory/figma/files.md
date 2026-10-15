# Figma project index

> Template. Project copy lives at `.claude/design-manager/figma/files.md`; fill it for your project.

> Curated project knowledge: which file is the **project's design-system file** (build target + variable library), which files are **legacy** (transfer sources), and which files serve as **references**. File keys + URLs + page references.

---

## Project home

- **Team:** _TBD_ · **Project:** _TBD_
- **URL:** _TBD_
- All current files of the project live here. Files from older projects (section "Legacy" below) are **transfer sources** — they do not lead.

---

## Source-of-truth governance

- **The UI kit and Storybook lead:** names and values in `@laioutr-core/ui-kit` / `@laioutr-core/ui-tokens` and the rendered Storybook. Figma follows the UI kit.
- **The project's design-system file is the build target.** Atoms/Molecules/Organisms and the variable library are created and structured **there** — not in legacy files.
- **Legacy files are transfer sources only.** On conflict the UI kit / Storybook wins; `design-system-governance` brings Figma in line and the delta is logged in the project copy of `../design-system/migration.md`.

---

## Files

| Role | File name | File key | Pages | URL |
|---|---|---|---|---|
| **Design system (build target)** — Atoms/Molecules/Organisms + variable collections | _TBD_ | `<your-file-key>` | _TBD_ | _TBD_ |
| Reference (read, do not build) | _TBD_ | _TBD_ | _TBD_ | _TBD_ |

### Legacy — transfer sources (transfer → rename → sort into the design-system file)

| Role | File name | File key | Pages | URL |
|---|---|---|---|---|
| _TBD_ | _TBD_ | _TBD_ | _TBD_ | _TBD_ |

### Non-Figma sources (comparison)

| Source | Used for | Address |
|---|---|---|
| **Storybook** (rendered truth) | Atoms/Molecules/Organisms structure + naming model | `https://storybook.laioutr.cloud` (component index: `https://storybook.laioutr.cloud/index.json`) |
| **UI kit package** (code, names lead) | canonical component/token names + values | npm `@laioutr-core/ui-kit` (tokens: `@laioutr-core/ui-tokens`) |
| **Docs** | platform concepts, sections/blocks | `https://docs.laioutr.com` (via the `laioutr-docs` MCP) |

Detail inventory: `../design-system/components.md`. Atomic page structure: `../design-system/atomic-structure.md`. Color-scheme logic: `../design-system/color-scheme.md`.

---

## How to probe variables reliably (Plugin API)

> **Reliable reading = Plugin API via `use_figma` (read-only)** — works through the hosted connector and returns **all pages, collections, modes and variable values/aliases**, regardless of the active mode. The hosted `get_metadata` only shows the active page, and `get_variable_defs` only resolves the appearance axis (light/dark) — it is blind to the scheme axis. So for in-depth work use `use_figma` read scripts, **not** `get_variable_defs`.

```js
// in use_figma (read-only), fileKey = <your-file-key>
const cols = await figma.variables.getLocalVariableCollectionsAsync(); // collections + modes
const vars = await figma.variables.getLocalVariablesAsync();           // valuesByMode + aliases
const pages = figma.root.children.map(p => p.name);                    // all pages
```
For concrete hex values: follow the alias chain, choose the target mode per collection (theme = `<brand theme>`, color schemes = `<scheme>`, colors = light/dark). Record results in the project copies of `../design-system/tokens.md`.

---

> **URL parsing:** `figma.com/design/<fileKey>/<name>?node-id=<nodeId>` → convert `-` to `:` in the node id before making MCP calls.
