# Output Template

The final document produced by `figma-component-architecture`. Write to `docs/plans/YYYY-MM-DD-<topic>-component-architecture.md` (the filename suffix names the document type and is stable across the skill rename).

## Template

```markdown
# Component Architecture: [Topic]

Analysis source: [path to design analysis document]
Date: [date]

## Conventions

- Props + events for single-level composition (default)
- createContext (reka-ui) for compound components sharing UI config with deep children
- All data via props. No data fetching, no store access.
- Internal state is UI-only: open/close, selected index, animation flags

## Architecture Decisions

| # | Decision | Choice | Rationale |
|---|---|---|---|
| 1 | [question from Phase 2] | [chosen approach] | [why] |
| ... | ... | ... | ... |

## Component Specs

### Phase 1: [leaf components]

#### ComponentA
[trivial or full spec]

#### ComponentB
[trivial or full spec]

### Phase 2: [compositions]

#### ComponentC
[full spec — composes Phase 1 components]

### Phase 3: [top-level organisms]

#### ComponentD
[full spec — composes Phase 2 components]

## Resolved Challenges

| # | Challenge | Resolution |
|---|---|---|
| 1 | [from Phase 4] | [user's answer] |

## Integration Requirements (deferred to Section/Block wrapper)

- [Consumer] needs [data] from [source], mediated by [page/Section/Block wrapper]
- [If no cross-organism data dependencies: omit this section]

## Open Questions

- [Unresolved items, if any]
```
