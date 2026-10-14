# Plan Output Template

The analysis must produce a structured plan document. This is the primary deliverable of `figma-design-analysis`. This reference is the full template, the per-section rules, and the definition of "non-obvious behavior" worth annotating.

## Template

```markdown
# Design Analysis: [Component/Page Name]

Figma: [URL]
Analyzed: [date]

## Design Intent

[Brief description of what the design achieves as a whole — 2-3 sentences on UX goals]

### Jira Context

| Ticket | Summary | Key Acceptance Criteria |
|---|---|---|
| PROJECT-123 | [summary from Jira] | [key criteria] |
| ... | ... | ... |

[If no Jira tickets: omit this subsection]

## Component Hierarchy

- **ComponentA** -- NEW — [Desktop Layout](https://www.figma.com/design/fileKey/fileName?node-id=75-37105), [Mobile Layout](https://www.figma.com/design/fileKey/fileName?node-id=75-38200)
  - SubComponentB -- EXISTS at `#ui/components/SubComponentB/`
  - SubComponentC -- EXISTS at `#ui-kit/components/SubComponentC/`
  - _Designer note (I75:37105;13:4105 "Details" button, development): "Go to Location Detail Page — 1) default location detail page type, 2) or link to custom page / pagetype" — routing contract; reflected as `to: RouteLocationRaw` prop_
  - _Non-obvious: hover on SubComponentB reveals tooltip with delivery estimate; collapsed → expanded transition on selection_
- **ComponentD** -- NEW — [Form Variants](https://www.figma.com/design/fileKey/fileName?node-id=80-42000)
  - Field, Input, Select -- EXISTS in `@laioutr-core/ui-kit`
- ExistingComponent -- REUSE at `#ui/components/ExistingComponent/` — [Current Design](https://www.figma.com/design/fileKey/fileName?node-id=90-55000)
  - _Modification intent: needs to support expandable content on selection and a custom icon area_

Legend: **bold** = needs implementation, normal = exists (the existing import path follows `EXISTS at` / `REUSE at`), _Designer note (node-id name, category[, variant scope]): "verbatim"_ = a `data-*-annotations` value extracted from the Figma MCP (any annotation category), _Non-obvious: ..._ = behavioral note inferred from the design, _Modification intent: ..._ = REUSE component change rationale, [Link Title](url) = Figma design for this component

## Variant Matrix

| Axis | Figma Values | Maps to Prop | Prop Type |
|---|---|---|---|
| breakpoint | xs, s, sm, md, lg | N/A (CSS media queries) | -- |
| size | full-width, boxed | `containerStyle` | `'full-width' \| 'boxed'` |
| ... | ... | ... | ... |

## Token Mapping

| Figma Variable | CSS Custom Property | Usage |
|---|---|---|
| `Spacing/SM` | `var(--spacing-sm)` | Card padding |
| ... | ... | ... |

## Existing Component Matches

| Figma Element | Existing Component | Import | Notes |
|---|---|---|---|
| CTA Button | `BackgroundAwareButton` | `#ui-kit/...` | Adapts to backdrop |
| ... | ... | ... | ... |

## Suggested Modifications

| Component | Current Behavior | Modification Intent |
|---|---|---|
| `RadioSelectItem` | Static content, no expand | Support expandable content on selection; custom icon area for shipping/payment icons |
| ... | ... | ... |

[If no modifications needed: omit this section]

## Issues Found

### In components you own (actionable)

- [ ] Double `colorValueToCss` processing in your `SectionFoo` wrapper + inner `Foo`
- [ ] Hardcoded `border-radius: 8px` in your `BlockBar` — should be `var(--forms-border-radius)`
- [ ] ...

### In upstream components (advisory — file on `laioutr-ui/ui-kit` or `laioutr-ui/ui`)

- [ ] [Upstream `@laioutr-core/ui` `BrandHero`] hardcoded theme map missing `tech` — silent fallback
- [ ] ...

[If no issues in either bucket: omit the subsection.]

## Implementation Order

1. **SubComponentB** (leaf component) -- [rationale]
2. **ComponentA** (composes SubComponentB + upstream primitives) -- [rationale]
3. **ComponentD** (independent, composes upstream primitives) -- [rationale]
4. **SectionWrapper** (defineSection wrapping ComponentA) -- [rationale]

## Notes for Implementation

- [any architectural decisions, data flow notes, slot composition patterns]

### Open Questions

- [Open-ended gap Phase 5 could not close interactively — needs design input] — _Suggested default:_ [what the implementer uses until design responds]
- ...

[If no open questions: omit this subsection]
```

## Key Rules for the Plan

1. **Every component gets a status**: NEW, EXISTS, or REUSE (exists but needs modification)
2. **Every NEW and REUSE component gets a Figma link**: link to the specific Figma node so the implementation skill can find the design without re-exploring the file
3. **REUSE components include modification intent**: high-level description of what changes and why, not props/slots
4. **Implementation order is leaf-first**: innermost components before their parents
5. **Issues distinguish owner**: list issues in components your module owns separately from issues observed in upstream — own-module issues carry a concrete fix; upstream issues are advisory and name the repo to file them on (`laioutr-ui/ui-kit` or `laioutr-ui/ui`)
6. **Token mapping is complete**: every Figma variable used in the design appears in the table
7. **Variant matrix distinguishes props from CSS**: breakpoints are never props
8. **Non-obvious behaviors are annotated inline**: only behaviors an implementer might miss from a static mockup
9. **Open questions are explicit and few**: Phase 5 closes every gap it can via `AskUserQuestion`; only truly open-ended gaps land here, each with a suggested default — never silently ignored
10. **Designer annotations are surfaced verbatim and shape decisions**: every `data-*-annotations` value found in the Figma MCP output — across *all* annotation categories, not just `development` — appears in the plan as a `_Designer note (<node-id> <name>, <category>): "<verbatim>"_` line under the relevant component. Where coverage is uneven, record which nodes were recipe-verified and which were not. Annotations also drive decisions — `-> UI Kit` overrides default placement, `Go to {Page}` is a routing contract reflected in props/events, `optional` means conditional content. If an annotation contradicts your visual analysis, the annotation wins.

## What Counts as Non-Obvious Behavior

Document only behaviors an implementer might miss from looking at a static mockup:

- State transitions: collapsed → expanded, idle → loading → success
- Hover/focus effects that reveal additional UI (tooltips, action buttons)
- Scroll-triggered behaviors (sticky headers, lazy loading)
- Viewport resize causing layout reflow (not just responsive breakpoints)
- Animation/transition expectations
- Conditional visibility (elements that appear/disappear based on state)

Do NOT document:
- "Button triggers navigation" — obvious from context
- "Input accepts text" — self-evident
- Standard form validation behavior
