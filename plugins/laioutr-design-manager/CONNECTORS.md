# Connectors

## How tool references work

Plugin files use `~~category` as a placeholder for whatever tool the user connects in that category. For example, `~~design` means Figma here, but any design tool with an MCP server would work.

Plugins are **tool-agnostic** — they describe workflows in terms of categories (design, docs, etc.) rather than specific products. The `.mcp.json` pre-configures specific MCP servers, but any MCP server in that category works.

## Connectors for this plugin

| Category | Placeholder | Included servers | Other options |
|----------|-------------|-----------------|---------------|
| **Design (hosted)** | `~~design` | **Figma hosted** (`mcp.figma.com`) — design context, screenshots, metadata by fileKey | Penpot |
| **Design (local / live selection)** | `~~design-local` | **Figma Dev Mode MCP** (`http://127.0.0.1:3845/mcp`) — variables/selection from the desktop app | |
| **Platform docs** | `~~docs` | Laioutr Docs MCP (`docs.laioutr.com`) | |
| **Platform API** | `~~platform` | Laioutr Cockpit MCP (`mcp.laioutr.com`) — Studio tools (`studio_get_page_tree`, `studio_add_section`, `studio_list_global_sections`, `studio_add_global_section_reference`, …) | |

> **`~~platform` requires sign-in.** The Laioutr Cockpit MCP works on the user's own Laioutr project: the user must be signed in to Laioutr and have access to the project. Without it, the `ux-designer` can still plan a composition, but cannot write it into Studio.

## Important: two Figma servers, two purposes

| Server | Can | Cannot |
|---|---|---|
| **Figma hosted** (`mcp.figma.com`) | `get_design_context`, `get_screenshot`, `get_metadata` by `fileKey`/`nodeId` | **no live selection / `get_variable_defs`** → reports "nothing selected" |
| **Figma Dev Mode** (`127.0.0.1:3845/mcp`) | live selection, **`get_variable_defs`**, Code Connect | only while the desktop app is running with the Dev Mode server enabled |

### Enable the Dev Mode server (one-time)
1. Open the Figma desktop app and open a design file.
2. Turn on **Dev Mode** (toolbar at the bottom, or `Shift+D`).
3. In the Inspect panel, click **"Enable desktop MCP server"** — the confirmation "server enabled and running" appears. It runs at `http://127.0.0.1:3845/mcp` (available on Dev and Full seats).
4. Then, for every variable probe: **select a layer/component** in the file that uses the variables → `get_variable_defs` reads the library.

## Two ways tools are addressed

| Mode | When | Examples |
|---|---|---|
| **MCP connector** (configured by organization/user) | Tools are registered as MCP. **No token management** in the plugin — tools such as `get_design_context` / `use_figma` are called directly. | Figma, Laioutr Docs, Laioutr Cockpit MCP |
| **Direct integration (REST/GraphQL + token)** | When no MCP connector is available. | — (none currently) |

→ **Figma** needs no memory file for auth — the MCP is self-documenting.
→ **Figma file keys and library structure** of the project are kept in the project copy of `memory/figma/files.md` (`.claude/design-manager/figma/files.md`): which file is the design-system master, which page holds which library.

## Figma MCP — the central axis of this plugin

**Read (design → context/code):** `get_design_context`, `get_screenshot`, `get_metadata`, `get_variable_defs`, `get_figjam`, `search_design_system`
**Write (intent/code → design):** `use_figma`, `create_new_file`, `upload_assets`
**Code Connect bridge:** `get_code_connect_map`, `add_code_connect_map`, `send_code_connect_mappings`

> **Mandatory:** Before every `use_figma` write, load the matching Figma skill instructions (`/figma-use`), and before `generate_diagram` the `/figma-generate-diagram` instructions. The `ui-designer` agent does this automatically.

## Boundary with the `laioutr-developer` plugin

The `laioutr-developer` plugin owns the **design → code pipeline for developers** (`figma-design-analysis` → `figma-component-architecture` → `figma-to-component` → `figma-export-assets`). This plugin owns the **design side** (building in Figma, design-system governance) and deliberately reaches **end-to-end** into implementation. Where the two overlap, the `laioutr-developer` pipeline is the canonical code reference; the `ui-designer` loads those skills for implementation instead of inventing its own code conventions.
