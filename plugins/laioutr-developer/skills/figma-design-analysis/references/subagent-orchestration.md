# Subagent Orchestration

For complex multi-component designs, `figma-design-analysis` parallelizes exploration using subagents. This reference describes the pipeline, the per-stage subagent prompts, and the constraints.

## Pipeline

```dot
digraph subagent_pipeline {
    rankdir=TB;

    "get_metadata on root" [shape=box];
    "Identify logical groups" [shape=box];
    "Subagent: Group A\nget_design_context + get_variable_defs" [shape=box];
    "Subagent: Group B\nget_design_context + get_variable_defs" [shape=box];
    "Subagent: Group N\n..." [shape=box];
    "Merge exploration results" [shape=box];
    "Subagent: Token mapping" [shape=box];
    "Subagent: Component matching" [shape=box];
    "Cross-file verification\n+ placement + plan assembly" [shape=box];

    "get_metadata on root" -> "Identify logical groups";
    "Identify logical groups" -> "Subagent: Group A\nget_design_context + get_variable_defs";
    "Identify logical groups" -> "Subagent: Group B\nget_design_context + get_variable_defs";
    "Identify logical groups" -> "Subagent: Group N\n...";
    "Subagent: Group A\nget_design_context + get_variable_defs" -> "Merge exploration results";
    "Subagent: Group B\nget_design_context + get_variable_defs" -> "Merge exploration results";
    "Subagent: Group N\n..." -> "Merge exploration results";
    "Merge exploration results" -> "Subagent: Token mapping";
    "Merge exploration results" -> "Subagent: Component matching";
    "Subagent: Token mapping" -> "Cross-file verification\n+ placement + plan assembly";
    "Subagent: Component matching" -> "Cross-file verification\n+ placement + plan assembly";
}
```

## Pipeline Stages

| Stage | Parallel? | Subagent prompt includes | Returns |
|---|---|---|---|
| Root exploration | No (sequential) | Figma URL, root node ID | Tree structure, group names + node IDs |
| Per-group exploration | **Yes** (1 subagent per group) | Group node ID, `clientLanguages: "typescript,vue"` | Component hierarchy, tokens used, visual structure |
| Token mapping | **Yes** (parallel with matching) | All collected Figma variables from merge | Token mapping table (Figma var → CSS var) |
| Component matching | **Yes** (parallel with tokens) | All component names from merge | Existing matches table, cross-file issues |
| Plan assembly | No (sequential) | All above results | Final plan document |

## Subagent Prompts

**Per-group exploration subagent:**
> Explore Figma node `{nodeId}` using `get_design_context` with `clientLanguages: "typescript,vue"`, `clientFrameworks: "vue,nuxt"`. Also call `get_variable_defs` on the same node. Return: (1) component hierarchy with element names, (2) all Figma variables referenced, (3) variant axes if this is a variant set, (4) **every `data-*-annotations="..."` value found in the response, verbatim, paired with the `data-node-id`, `data-name`, and the annotation category** — these are designer-authored notes about behavior, optionality, routing, and package placement. Match `data-[a-z-]*annotations`, not just `data-development-annotations`; multiple categories exist. Use the jq scan recipe in [`phase-1-figma-exploration.md`](./phase-1-figma-exploration.md) under "Extracting Dev-Mode Annotations", and run it before your response is discarded — inline responses cannot be re-scanned later. If no annotations exist, return "no annotations" **and** the list of `data-*` attribute names you did see, as evidence the scan ran. Use `get_screenshot` if layout isn't clear from context.

**Component matching subagent:**
> Search for existing components matching these names: `{componentNames}`. Search paths: `node_modules/@laioutr-core/ui-kit/dist/runtime/app/components/` (atomic primitives), `node_modules/@laioutr-core/ui/dist/runtime/components/` (commerce organisms). For each match, return: component path, props interface, and import alias. If the design might also exist as a Section/Block in your own module's `src/runtime/app/sections/` or `src/runtime/app/blocks/`, search there too (this is rare — most matches are upstream).

**Token mapping subagent:**
> Map these Figma variables to CSS custom properties using the conversion rules in [`token-mapping.md`](./token-mapping.md): `{variables}`. Validate any hex colors against installed token files: `grep -i "{hex}" node_modules/@laioutr-core/ui-tokens/dist/*.css`. Return the complete mapping table.

## Constraints

- **Figma MCP bottleneck**: All subagents call the same Figma desktop app via MCP. Parallel reads work, but if the app throttles, dispatching 5+ simultaneous `get_design_context` calls may queue. Monitor for timeouts.
- **Merge step is critical**: The orchestrating agent must merge per-group results before dispatching Phase 2 subagents. Don't skip the merge -- token mapping needs the full variable list, component matching needs all component names.
- **Plan assembly stays in the main agent**: Only the orchestrating agent has full context to make placement decisions and assemble the final plan document.
