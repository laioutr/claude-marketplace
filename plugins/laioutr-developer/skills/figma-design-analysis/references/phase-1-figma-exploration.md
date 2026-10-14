# Phase 1: Figma Exploration

Detailed mechanics for the first phase of `figma-design-analysis`. Covers MCP failure handling, enumerating visible-state variant frames, dev-mode annotations (every category, both emitted forms) and the jq recovery recipe, discovering component definitions across the three file shapes, capturing per-component Figma links, recognizing variant matrices, and scanning for Jira ticket references.

## MCP Failure Handling

The Figma MCP requires the **Figma desktop app** to be open with the target document as the active tab.

- **"No node could be found"** → Ask the user to open the document in Figma desktop and make it the active tab
- **Intermittent failures** (some calls succeed, then later calls fail) → The Figma app likely lost focus or switched tabs. Ask the user to re-focus the document. Do NOT discard data from earlier successful calls — work with partial data and note which nodes could not be explored
- **Timeout or large output saved to file** → Use `rg` (or `jq` for the annotation recipe) on the saved file to extract specific elements (e.g., `<canvas` tags, `<instance` nodes) rather than reading the full file

## Extracting the Node ID

From URL `https://www.figma.com/design/:fileKey/:fileName?node-id=75-37105`, the node ID is `75:37105` (replace `-` with `:`).

**Validate the resolved node before deep-exploring it.** Call `get_metadata` on the node first; if the returned frame's name doesn't match the URL filename (e.g., URL says `Location-Finder` but the node is named "payment methods"), or if its dimensions are suspiciously small (<200px on both axes for what should be a page-level frame), the URL anchor points at a sub-instance rather than the design root. Call `get_metadata(0:0)` to list the file's top-level frames and pick the one that matches the URL's `fileName` slug. Don't waste exploration budget drilling into the wrong subtree.

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

## Enumerating Visible-State Variant Frames

Designs often have multiple frames at the root level representing different visible states of the same screen — collapsed/expanded, default/active, mobile/desktop, with-results/empty. Each frame renders a different subtree, and **annotations only appear in `get_design_context` output for the frame where the annotated element is actually visible**. Annotations attached to an element that is hidden in one variant never surface in that variant's call.

Before extracting annotations:

1. Enumerate root-level frames whose names share a stem and differ only by a state qualifier (`location detail` vs `location detail - opening times open`, `mobile / search active` vs `mobile / search results`, `LG - laptops - location detail` vs `LG - laptops - location detail - opening times open`).
2. Call `get_design_context` on **each** variant frame, not just the default. Do not assume "expanded" is "default with an accordion toggled" — Figma represents states as full sibling frames, and the expanded subtree only exists in the expanded frame.
3. Run the annotation recipe on each saved response and merge the rows.

This is breadth (calling across sibling variant frames). "Drill into each meaningful subgroup" below is depth (calling on leaves within one frame). Both are required — depth alone misses every annotation on hidden subtrees.

## Extracting Dev-Mode Annotations

`get_design_context` embeds designer annotations in the returned code as `data-*-annotations="..."` attributes on the JSX elements.

**There is more than one annotation attribute name.** Figma annotation *categories* each produce their own attribute. Two are confirmed in the wild:

| Attribute | Figma category |
|---|---|
| `data-development-annotations` | the default "Development" category |
| `data-information-for-designers-annotations` | a custom category named "Information for designers" |

A team can create any category, so the set is open-ended: the attribute name is the category name slugified, wrapped as `data-<slug>-annotations`. **Always match `data-[a-z-]*annotations`, never a single literal name** — a recipe hardcoded to `data-development-annotations` silently returns zero rows for every other category and you will conclude "no annotations" while staring at one.

**The MCP's trailing warning is not a per-node signal.** Responses that contain annotations usually end with *"Some elements have annotation data attributes... IMPORTANT: Do not ignore these annotation attributes."* — but this warning has been observed firing for a node whose returned JSX contains **no** annotation attribute at all (it appears to be scoped to the component set or file, not the requested node). So: its presence does not prove there is something to extract, and you still have to run the scan. Treat it as a hint, never as evidence either way.

These annotations are the only place a designer can record information that is not visible in the layer tree, the screenshot, or the variable list:

| Annotation pattern | Typical meaning |
|---|---|
| `optional section` | The element/region is conditional content; the parent must accept it as a slot or boolean prop |
| `simple accordeon`, `on hover reveals tooltip` | Interaction model that is invisible in a static mockup |
| `Go to {Page}` | Routing contract on a button or link — reflect as part of the component's events/props |
| `New Component -> UI Kit` (or `-> Order Summary`) | Package placement or composition target — overrides the default placement heuristic |
| `... is optional, but at least one is required` | Data-shape contract for a prop |
| `First Image is always the Location Image also used on the location cards` | Cross-component contract — the same data flows into another component |
| `Display X in a basic table [https://www.figma.com/design/<other-fileKey>/...]` | URL pointing at a node in a different Figma file — supplementary context the designer recorded. Could be a canonical component definition, a usage example, a related pattern, or background reference. The contents aren't determined until you fetch them — but it always needs investigation, not skipping |

**Section-level `get_design_context` calls return sparse metadata WITHOUT annotations.** Annotations only appear in leaf-level (or near-leaf) calls that return full code. Drill into each meaningful subgroup; do not stop at the section.

**Annotation values can be multi-line.** In the quoted form the newlines come through as real line breaks; in the JSX-expression form they come through as escaped `\n`. Render both as line breaks when you quote the note in the plan.

**Prefer a dedicated annotation tool when available.** Before using the JSX scan recipe below, check whether your MCP exposes a tool whose name contains `annotation` (e.g. `mcp__figma__get_annotations` — Figma's roadmap mentions a dedicated annotations API in development). A structured tool response is more reliable than regex-parsing JSX strings; fall back to the recipe only when no such tool exists.

**Recovery recipe.** Use jq alone. It splits the JSX at each `data-node-id`, so every element is scanned on its own and an element carrying **several** annotation categories yields one row per category. It returns structured `[node-id] <category-annotations>:` rows (one note value per indented line) with no surrounding noise:

```bash
jq -r '
  .[].text
  | split("data-node-id=\"")[1:][]
  | (capture("^(?<id>[^\"]+)").id) as $id
  | [scan("data-([a-z-]*)annotations=(\"[^\"]+\"|\\{[^}]*\\})")][]
  | . as $m
  | ($m[1] | [scan("\"((?:[^\"\\\\]|\\\\.)+)\"")] | map(.[0]) | map(select(. != "undefined"))) as $vals
  | "[\($id)] <\($m[0])annotations>:\n  " + ($vals | join("\n  ")) + "\n"
' /path/to/saved-output.txt
```

**Two emitted forms — the recipe must handle both.** This is the single most likely way to silently miss an annotation:

| Form | When | Example |
|---|---|---|
| Quoted attribute | plain frames | `data-development-annotations="is displayed when the video is paused"` |
| **JSX expression** | **variant-set components** | `data-development-annotations={size === "s" && state === "start" ? "Default\n\nthe beginning of the animation" : undefined}` |

A variant set (`State=Default \| hover \| pressed`, `breakpoint=xs \| s \| …`) renders every variant as conditionals, and its annotations become ternaries. A pattern anchored to `="…"` matches **zero** of them. The alternation `("[^"]+"|\{[^}]*\})` above covers both.

Trade-off to expect: for the expression form the inner scan also returns the ternary's *condition* literals (`"s"`, `"start"`) alongside the real note. Noise is the right failure direction here — the annotation text is normally the longest value in the list. Do not tighten the pattern to suppress it. The condition is not only noise: it says which variant the note applies to (`size === "s" && state === "start"`), so record it as the note's variant scope in the plan.

The `data-([a-z-]*)annotations` group captures **which** category produced each note, so a `development` note and an `information-for-designers` note stay distinguishable in the output. Works for both MCP response forms: the file form is a JSON array `[{type, text}]` where the JSX is a single JSON-encoded string with `\n` escaped; the inline form returned directly into the conversation has the same shape. The pattern relies on `data-node-id` always preceding the annotation attribute in the MCP's emitted JSX (verified). The recipe does not capture `data-name` — take the layer name for each node id from the same response or the `get_metadata` tree when writing the `Designer note` line.

**Inline responses are not persisted.** Only oversized results get written to a tool-results file; anything returned inline exists solely in your context and cannot be re-scanned with jq later. So either run the scan while the response is still fresh, or accept that verifying it later means re-fetching the node. Never claim recipe-level coverage for a response you only read by eye — say which nodes were recipe-verified and which were not.

For quick ad-hoc inspection, `jq -r '.[].text' file | rg -o 'data-[a-z-]*annotations'` lists the category names present. Avoid `grep -o` with large context windows here — some `grep` builds (ugrep) reject wide `.{0,N}` context patterns with "exceeds complexity limits".

**Brittleness & sanity check.** The recipe assumes (a) the attribute matches `data-*-annotations`, (b) `data-node-id` precedes it on the same element, (c) the value is either a double-quoted string or a `{…}` expression containing no nested braces. If Figma changes any of those, the recipe silently returns zero rows.

**The `rg -c 'annotat'` check below is the real safety net** — it is form-agnostic, so it catches quoted values, JSX expressions, and any future syntax alike. A zero there is trustworthy in a way a zero from the structured recipe is not. Always run it. After running the recipe, sanity-check with:

```bash
jq -r '.[].text' /path/to/saved-output.txt | rg -c 'annotat'
jq -r '.[].text' /path/to/saved-output.txt | rg -o 'data-[a-z-]*annotations=' | wc -l
jq -r '.[].text' /path/to/saved-output.txt | rg -o 'data-[a-z-]+=' | sort -u
```

Reconcile them:

- **Attribute count = recipe rows.** The second command counts annotation attributes; it must equal the number of `[node-id]` rows the recipe printed. A difference means the recipe missed one — find it before going on.
- **The line count is a presence check, not a total.** `rg -c` counts *lines*, and the MCP's trailing warning ("Some elements have annotation data attributes…") is one of them, so expect it to exceed the attribute count by one when that warning is present. To see what any other surplus is: `jq -r '.[].text' file | rg 'annotat' | rg -v 'data-[a-z-]*annotations='`.
- **Format change.** If the first command finds matches but the recipe and the second command return zero, Figma changed the format — inspect the third list for an unfamiliar attribute name and adapt the regex.

A clean zero looks like the third command returning only `data-name=` and `data-node-id=`. Do NOT conclude "no annotations exist" unless the recipe AND all three checks agree.

**For multi-component designs**, do the extraction inside the per-group exploration subagent (see [subagent-orchestration.md](./subagent-orchestration.md)). The 70K+ JSX dump stays in the subagent's context; only the structured annotation list bubbles up.

**Use annotations as inputs to your decisions, not just text to surface:**

- A `-> UI Kit` or `-> {Component}` annotation **overrides** the default package-placement heuristic. Place the component there even if the visual signals (commerce-specific, complex composition) suggest `ui` or `organism`. If an annotation contradicts your visual analysis, the annotation wins.
- A `Go to {Page}` annotation on a button or link is a routing contract — reflect it in the component's planned events or props (e.g., a `to: RouteLocationRaw` prop, or a `@click` event the wrapper Section/Block routes).
- An `optional` annotation means the parent must accept the element as conditional content (slot or boolean prop) — flag it in the modification intent or in the proposed props for the new component.
- An annotation that names another component (e.g., `also used on the location cards`) implies a shared data contract — verify both components have compatible prop shapes, or surface the gap as an Open Question.
- An annotation value that contains a URL to a **different Figma file** (different `fileKey`) is supplementary context the designer chose to record. It might be a canonical component definition, a usage example, a related pattern, or background reference — you cannot tell which from the URL alone. Treat it as required reading: the linked node likely carries information (variants, states, intended composition, design rationale) that isn't in the inline render. **When trying to read a Figma reference in a file that's not the active tab, do not defer or proceed without it. Stop, ask the user to open the linked file in Figma desktop, and wait for confirmation before continuing.** Then call `get_design_context` on the linked node and fold what you learn into the analysis.

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
