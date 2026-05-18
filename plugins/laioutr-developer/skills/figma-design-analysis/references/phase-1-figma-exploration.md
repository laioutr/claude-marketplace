# Phase 1: Figma Exploration

Detailed mechanics for the first phase of `figma-design-analysis`. Covers MCP failure handling, dev-mode annotations and the jq recovery recipe, discovering component definitions across the three file shapes, capturing per-component Figma links, recognizing variant matrices, and scanning for Jira ticket references.

## MCP Failure Handling

The Figma MCP requires the **Figma desktop app** to be open with the target document as the active tab.

- **"No node could be found"** → Ask the user to open the document in Figma desktop and make it the active tab
- **Intermittent failures** (some calls succeed, then later calls fail) → The Figma app likely lost focus or switched tabs. Ask the user to re-focus the document. Do NOT discard data from earlier successful calls — work with partial data and note which nodes could not be explored
- **Timeout or large output saved to file** → Use `grep` on the saved file to extract specific elements (e.g., `<canvas` tags, `<instance` nodes) rather than reading the full file

## Extracting the Node ID

From URL `https://www.figma.com/design/:fileKey/:fileName?node-id=75-37105`, the node ID is `75:37105` (replace `-` with `:`).

## Exploration Strategy

**For any design, start with `get_design_context`** -- it returns structured code-ready output with component hierarchy, styling, and variable references. Use `clientLanguages: "typescript,vue"` and `clientFrameworks: "vue,nuxt"`.

**For complex multi-component designs** (checkout pages, full layouts):
1. `get_metadata` on the root node to see the full tree structure (names, positions, sizes)
2. Identify logical groups (header, form section, summary, etc.) by node names
3. `get_design_context` on each logical group separately (avoids output size limits)
4. `get_variable_defs` on the root to get all token definitions at once
5. `get_screenshot` on key nodes to visually verify your understanding

**For focused single-component designs** (a button, a card, a banner):
1. `get_design_context` on the node directly (returns code-ready output)
2. `get_variable_defs` on the same node for token definitions
3. `get_screenshot` if the visual layout isn't clear from context alone

## Extracting Dev-Mode Annotations

`get_design_context` embeds designer annotations in the returned code as `data-development-annotations="..."` attributes on the JSX elements. The MCP appends an explicit warning at the end of every response that contains them: *"Some elements have annotation data attributes... IMPORTANT: Do not ignore these annotation attributes."*

These annotations are the only place a designer can record information that is not visible in the layer tree, the screenshot, or the variable list:

| Annotation pattern | Typical meaning |
|---|---|
| `optional section` | The element/region is conditional content; the parent must accept it as a slot or boolean prop |
| `simple accordeon`, `on hover reveals tooltip` | Interaction model that is invisible in a static mockup |
| `Go to {Page}` | Routing contract on a button or link — reflect as part of the component's events/props |
| `New Component -> UI Kit` (or `-> Order Summary`) | Package placement or composition target — overrides the default placement heuristic |
| `... is optional, but at least one is required` | Data-shape contract for a prop |
| `First Image is always the Location Image also used on the location cards` | Cross-component contract — the same data flows into another component |

**Section-level `get_design_context` calls return sparse metadata WITHOUT annotations.** Annotations only appear in leaf-level (or near-leaf) calls that return full code. Drill into each meaningful subgroup; do not stop at the section.

**Annotation values can be multi-line** — newlines inside the attribute value are preserved literally.

**Prefer a dedicated annotation tool when available.** Before using the JSX scan recipe below, check whether your MCP exposes a tool whose name contains `annotation` (e.g. `mcp__figma__get_annotations` — Figma's roadmap mentions a dedicated annotations API in development). A structured tool response is more reliable than regex-parsing JSX strings; fall back to the recipe only when no such tool exists.

**Recovery recipe.** Use jq alone — its `scan` regex matches across the JSON envelope and across embedded newlines in one pass, returning structured `[node-id] name: annotation` rows with no surrounding noise:

```bash
jq -r '
  .[].text
  | [scan("data-node-id=\"([^\"]+)\"[^>]*?data-development-annotations=\"([^\"]+)\"(?:[^>]*?data-name=\"([^\"]+)\")?"; "g")][]
  | "[\(.[0])] \(.[2] // "(no name)"):\n  \(.[1] | gsub("\n"; "\n  "))\n"
' /path/to/saved-output.txt
```

Works for both MCP response forms: the file form is a JSON array `[{type, text}]` where the JSX is a single JSON-encoded string with `\n` escaped; the inline form returned directly into the conversation has the same shape. The pattern relies on `data-node-id` always preceding `data-development-annotations` in the MCP's emitted JSX (verified). `data-name` is captured opportunistically when present.

For quick ad-hoc inspection, `jq -r '.[].text' file | grep -B 2 -A 10 'data-development-annotations='` also works but returns ~10 lines of unrelated attributes per match.

**Brittleness & sanity check.** The recipe assumes (a) the attribute is named `data-development-annotations`, (b) `data-node-id` precedes it on the same element, (c) values are double-quoted with no escaped `"`. If Figma changes any of those, the recipe silently returns zero rows. After running it, sanity-check with:

```bash
jq -r '.[].text' /path/to/saved-output.txt | grep -ci 'annotat'
```

If this finds matches but the recipe returned zero, the output format has changed — visually scan the response for any `data-*annotation*` attribute name (or comment-block / sibling field carrying the notes) and adapt the regex. Do NOT conclude "no annotations exist" unless both the recipe AND this sanity grep return zero.

**For multi-component designs**, do the extraction inside the per-group exploration subagent (see [subagent-orchestration.md](./subagent-orchestration.md)). The 70K+ JSX dump stays in the subagent's context; only the structured annotation list bubbles up.

**Use annotations as inputs to your decisions, not just text to surface:**

- A `-> UI Kit` or `-> {Component}` annotation **overrides** the default package-placement heuristic. Place the component there even if the visual signals (commerce-specific, complex composition) suggest `ui` or `organism`. If an annotation contradicts your visual analysis, the annotation wins.
- A `Go to {Page}` annotation on a button or link is a routing contract — reflect it in the component's planned events or props (e.g., a `to: RouteLocationRaw` prop, or a `@click` event the wrapper Section/Block routes).
- An `optional` annotation means the parent must accept the element as conditional content (slot or boolean prop) — flag it in the modification intent or in the proposed props for the new component.
- An annotation that names another component (e.g., `also used on the location cards`) implies a shared data contract — verify both components have compatible prop shapes, or surface the gap as an Open Question.

**Surface every annotation verbatim in the plan** under a structured `Designer note:` line in the Component Hierarchy (see [plan-output-template.md](./plan-output-template.md)). Do NOT fold designer annotations into `Non-obvious:` italic lines — reviewers must be able to distinguish verbatim designer intent from agent inference.

## Discovering Component Definitions

Figma files typically separate **page compositions** (full pages with component instances assembled into layouts) from **component definitions** (detailed designs with all variants and states). When given a page composition, you must find the component definitions.

**Why this matters:** A page composition shows component *instances* — a single checkout header placed in context. The component *definition* shows all variants (Device × Logo position), all states (default, hover, focus), and detailed styling. The implementation skill needs the definition, not just the instance.

**How to find component definitions:**

1. **Explore the file's page structure.** Call `get_metadata` on `0:0` (the document root) to see all top-level pages/canvases. The result can be very large (100K+ characters for files with many pages) — if it exceeds the output limit and is saved to a file, grep for `<canvas` tags to extract page names and IDs only. Classify the file into one of three patterns:

   - **Dedicated component library page** — Look for pages named "💎 Components", "Component Library", "Design System", or similar. This is the ideal case (see step 2a).
   - **Per-component canvas pages** — Each canvas is a component category (e.g., "Footer", "Navigation", "Newsletter", "404"). There is no single library page; definitions are distributed across named canvases. This is common in "Component-Examples" or showcase files (see step 2b).
   - **No component definitions at all** — The file contains only page compositions assembled from external instances. All component definitions live in other Figma files (see step 2c).

2a. **Dedicated library page.** Call `get_metadata` on the library canvas to see its sections and definition frames. This gives you the full inventory of component definitions in the file.

2b. **Per-component canvas pages.** Match instances on the composition to canvas pages by name. For instance `headers/ basic shop header / desktop` on a file with a "Navigation" canvas — explore that canvas for the definition. For `basic / desktop` with a "Footer" canvas — explore that canvas. Each canvas may contain variant definitions (desktop/mobile, expanded/collapsed) for that component category.

2c. **No local definitions.** The file is purely compositional — all components are instantiated from external Figma libraries. Link to the composition nodes directly and mark all instances as external. Consider whether the user should analyze the source file instead.

3. **Match instances to definitions.** When `get_metadata` on a page composition shows `<instance>` nodes (e.g., `<instance id="75:81197" name="Checkout Header" />`), look for matching frames on the component library page or the relevant canvas page. Match by name — but instance names may differ from definition names (e.g., instance "totals" → definition "totals box"). When uncertain, use `get_design_context` on the instance to find the linked source node ID and cross-reference with definition IDs.

4. **Parse path-structured instance names.** Figma instances often use `/`-separated paths: `headers/ basic shop header / desktop`. Parse these as `category / component-name / variant`. The component name is the middle segment (`basic shop header`); the last segment is typically a breakpoint or variant (`desktop`, `mobile`); the first segment is a category grouping (`headers`). Use the component name for matching — search for "basic shop header" or "shop header" on the library/canvas pages, not the full path string.

5. **Prefer definition links over instance links.** In the Component Hierarchy, link to the component **definition frame** (where all variants are shown), not to the instance on the page composition. The definition frame gives the implementation skill access to all variant combinations.

6. **Handle external library components.** Compositions frequently use instances from *other* Figma files (shared design system libraries). If an instance name has no matching definition on any library page or canvas page within the file, it is an external component. Mark it in the hierarchy with its source context (e.g., "EXISTS — external, from main design system"). Do NOT force-match it to an unrelated local definition.

**Example (dedicated library):** The checkout page at `75:37105` contains an instance `75:81197` named "Checkout Header". The same file has a "💎 Checkout Components" page at `1:47` with a "Checkout Header" definition frame at `66:20116` containing 4 variants (Device × Logo position). Link to `66:20116`, not `75:81197`.

**Example (per-component canvases):** The "Component-Examples" file has canvases named "Footer", "Navigation", "Newsletter", "404", etc. An instance `basic / desktop` on the 404 composition maps to the "Footer" canvas. An instance `headers/ basic shop header / desktop` maps to the "Navigation" canvas. Explore each canvas to find the definition frames with all variants.

## Capturing Per-Component Figma Links

During exploration, **record Figma links for each component you identify**. Every NEW or REUSE component in the Component Hierarchy must include links to its relevant Figma nodes. This enables the `figma-to-component` implementation skill to locate the exact design without re-exploring the full file.

**How to capture node IDs:**
- `get_metadata` returns node IDs for every element in the tree — map logical groups to their IDs
- `get_design_context` responses reference specific nodes — note which node corresponds to which planned component
- **Prefer component definition nodes** over instance nodes — if a component library page exists, link to the definition frame (with all variants) rather than the instance on the page composition

**Node ID → URL:** Given file key from the root URL and node ID `75:37105`, the component link is:
`https://www.figma.com/design/{fileKey}/{fileName}?node-id=75-37105` (replace `:` with `-` in the node ID). For Figma instance nodes (IDs like `I8:64336;11:32401`), link to the parent frame or component definition instead — instance IDs are unstable and don't produce reliable URLs.

**Link title format:** `{ComponentName} {Qualifier}` where Qualifier is the breakpoint (`Mobile`, `Desktop`, `Tablet`) or state (`Expanded`, `Collapsed`, `Error`). Use developer-friendly names, not raw Figma layer names. Examples: `[Accordion Desktop](url)`, `[Order Summary Expanded](url)`, `[CTA Banner All Variants](url)`.

**What to link and how many:**
- **Separate responsive layouts** — If the design has distinct desktop and mobile frames in different nodes, link both: `[ContactSection Desktop](url)`, `[ContactSection Mobile](url)`
- **State variants** — If a component has visually distinct states in separate frames (collapsed/expanded, empty/filled, error/success), link those too: `[Order Summary Collapsed](url)`, `[Order Summary Expanded](url)`. State links count toward the per-component budget.
- **Variant matrices** — When a single parent frame contains many variant combinations (breakpoint × size × fill × alignment — potentially hundreds of children), link to the **parent frame only**. The implementation skill can explore individual variants from there.
- **Composition frames vs variant frames** — A frame containing *different* child components assembled together (a checkout section with header, form, and summary) is a composition — link the individual children. A frame containing *variants of the same component* (breakpoint × size × color) is a variant matrix — link the parent.
- **Don't over-link** — 1-3 links per component is typical. If you find yourself adding more, you're probably linking individual variants instead of their parent frame. Link each unique component *type* once, not each instance (if `InputField` appears 8 times, one representative link is enough).
- **Missing counterparts** — If a frame is named "... / mobile" or "... / desktop" and the counterpart is not in scope, note it as a gap in Open Questions so the implementation skill knows to look for it.

**Use subagents for large Figma files** — see [subagent-orchestration.md](./subagent-orchestration.md) for the parallel pipeline.

## Recognizing Figma Variant Matrices

Figma designers often enumerate all combinations of component properties in a grid. A node with hundreds of children that look identical except for small differences is a **variant matrix**. Example: 384 children = 4 breakpoints x 3 sizes x 2 fills x 8 alignments x 2 color modes x 3 ratios.

In code, this reduces to a **single component with props**. Each Figma axis becomes a prop. The breakpoint axis is NOT a prop -- it's handled by CSS media queries.

**Figma breakpoints != implementation breakpoints.** Figma may show variants at 7 widths (initial, xs, s, sm, md, lg, xl), but the implementation typically collapses several into fewer `@media` queries. Check which breakpoints actually cause layout or spacing changes -- adjacent breakpoints with identical styling should be merged. Document which Figma breakpoints are collapsed.

## Jira Ticket Context

During exploration, scan Figma component descriptions and layer names for Jira ticket references (e.g., `PROJECT-123`, `TEAM-456` — match whatever project keys your team uses). Also accept ticket references provided directly by the user.

For each ticket found, fetch context using the Jira MCP (`getJiraIssue`) to retrieve summary, description, and acceptance criteria. Include gathered ticket context in the Design Intent section of the output document.

If the Jira MCP is unavailable, note the ticket references and ask the user to provide the relevant context manually.
