# Source Mapping

Tracks what was carried over from Laioutr's internal `design-manager` plugin
into this public plugin, what was rewritten, and what was intentionally
excluded.

Current composition (2026-10-15): 7 skills, 2 agents, 2 commands, 23 memory
files (14 platform knowledge, 9 project templates), 1 MCP server config.

## General rewrites (all files)

- Translated from German to English. Agents reply in the user's language;
  skill descriptions carry a few German trigger phrases for DACH users.
- Removed all references to Laioutr's own Figma files (file keys, team and
  project IDs, node and variable IDs), to people, and to dated decision history.
  The resulting rules were kept.
- Removed examples from Laioutr's own website projects; the generic lessons were kept.
- Replaced references to the private monorepo with the public sources: the
  `@laioutr-core/ui-kit` / `ui` / `ui-tokens` packages, Storybook
  (`storybook.laioutr.cloud`) and the docs MCP (`docs.laioutr.com`).
- Replaced references to other internal plugins (marketing, sales) with "the
  project's brand guidelines".
- **New: project memory model.** The plugin directory is replaced on update, so
  project-specific knowledge now lives in the user's project under
  `.claude/design-manager/`. The plugin's project files are templates.
- **New: output rule.** File artifacts go to
  `${DESIGN_MANAGER_OUTPUT_DIR:-./design-manager-output}/<date>/` instead of an
  internal workdesk path.
- `mcp.json` → `.mcp.json`; docs MCP moved to `docs.laioutr.com`.

## Skills

| Source path | Plugin path | Action | Notes |
| --- | --- | --- | --- |
| `skills/design-manager/` | `skills/design-manager/` | translated + rewritten | Orchestrator; now also lists `taste` |
| `skills/create-layout/` | `skills/create-layout/` | translated | |
| `skills/component-library-build/` | `skills/component-library-build/` | translated + generalized | Builds the project's design-system file instead of Laioutr's own |
| `skills/design-system-governance/` | `skills/design-system-governance/` | translated + generalized | Same as above |
| `skills/design-critique/` | `skills/design-critique/` | translated | |
| `skills/compose-page/` | `skills/compose-page/` | translated | States the Cockpit MCP limits (no page creation or publishing) |
| `skills/taste/` | `skills/taste/` | translated | Adapted from Leonxlnx/taste-skill (MIT); license in `THIRD_PARTY_LICENSE` |
| `skills/ai-component-descriptions/` | — | **excluded** | Depends on an unreleased Cockpit feature |

## Agents and commands

| Source path | Action | Notes |
| --- | --- | --- |
| `agents/ui-designer.md` | translated + generalized | Works on the project's design-system file |
| `agents/ux-designer.md` | translated + generalized | Laioutr-website specifics turned into generic composition rules |
| `commands/design.md`, `commands/landing-page.md` | translated | |

## Memory

| Source path | Action | Notes |
| --- | --- | --- |
| `memory/sections-blocks/*`, `memory/page-types/*`, `memory/principles/*` | translated | Platform knowledge |
| `memory/patterns/accessibility-baseline.md` | translated | Contrast matrix is now a template table |
| `memory/patterns/layout-patterns.md` | translated | Project examples removed |
| `memory/figma/variable-workflow.md`, `memory/design-system/color-scheme.md`, `memory/design-system/component-descriptions.md` | translated + stripped | Architecture rules kept, Laioutr file data removed |
| `memory/design-system/{tokens,components,atomic-structure,migration,corporate-design,imagery}.md`, `memory/figma/files.md`, `memory/personas/README.md` | **converted to templates** | Structure and generic rules kept; content is `_TBD_` for the project to fill |
| `memory/patterns/hero-illustrations.md` | **excluded** | Laioutr's own website illustrations |
| `SCHEDULED-TASKS.md` | **excluded** | Internal routine |
