---
description: "Build a UI layout/screen/component directly in Figma — functional-minimalist, one reasoned direction, buildable (defineSection/defineBlock + Vue), WCAG 2.1 AA. Loads skills/create-layout and delegates to the ui-designer agent."
argument-hint: "<what to build> (e.g. 'hero section for a spring campaign landing page', 'PDP layout', 'pricing card')"
---

# `/design` — build a layout in Figma

Thin wrapper around `skills/create-layout`.

## Trigger

`/design {what to build, optional persona/breakpoint/constraints}`

## Workflow

1. Load `skills/create-layout` and run its workflow.
2. Sharpen the brief in at most one round (only if needed).
3. Delegate the build to the `ui-designer` agent.
4. Run `design-critique` as the gate.
5. Deliver Figma link + rationale + buildable map + gate status + next step.

> **Output rule (mandatory):** File artifacts (exports, screenshots, preview HTML, critique reports) go to `${DESIGN_MANAGER_OUTPUT_DIR:-./design-manager-output}/<YYYY-MM-DD>/` — never into the plugin directory and never into `.claude/design-manager/` (that folder holds curated project knowledge only).

## When not to use this command

- Build the component library / transfer legacy UI → `component-library-build`.
- Token/component migration → `design-system-governance`.
- Audit only → `design-critique`.
- Code implementation → `laioutr-developer` plugin.
