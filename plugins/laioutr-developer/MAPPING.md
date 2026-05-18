# Source Mapping

Tracks what was carried over from `laioutr/.claude/` and the package-local
`packages/*/.claude/` folders into this plugin, what was rewritten, and
what was intentionally excluded.

Final composition: 4 skills, 0 agents, 37 rules (13 top-level + 24
layer-specific across `ui-kit`, `ui`, `ui-app`), 1 MCP server config.

## Skills

| Source path | Plugin path | Action | Notes |
| --- | --- | --- | --- |
| `skills/adr/` | `skills/adr/` | copied | Generic ADR writing |
| `skills/component-architecture/` | `skills/component-architecture/` | copied | Still contains `packages/ui-kit/...` paths in grep examples (see Open work below) |
| `skills/figma-design-analysis/` | `skills/figma-design-analysis/` | copied | `COCKPIT-99` / `LUI-123` Jira keys generalized to `PROJECT-123` / `TEAM-456`. Still contains heavy `packages/ui-kit/`, `packages/ui/`, `packages/ui-app/` path usage. A `## Related skills` footer points at `figma-export-assets` and `figma-to-component`; the "Themable Default Images" section gained a one-line callout naming `figma-export-assets` as the non-themable image lane |
| `skills/figma-export-assets/` | `skills/figma-export-assets/` | copied + rewritten | Sourced from `~/.claude/skills/figma-export-assets/`. Monorepo paths rewritten for external devs: Step 4 destination table collapsed to "Consumer role / Public dir" (no `packages/<x>/` column), Step 5 path-pattern table re-labeled by role, Step 7 export-spec example uses `<module-root>/src/runtime/public/...` instead of `/Users/sl/src/laioutr/packages/ui/...`, verify commands use `src/runtime/public/...`, `IconName` references point at `@laioutr-core/ui-kit` instead of `packages/ui-kit/src/runtime/app/types/theme.ts`, "Cockpit component-picker" → "Studio component-picker", and the "new SVG to ui-kit" mistake fix gained an explicit reference to the four-step resolution ladder. Cross-references `figma-design-analysis` and `figma-to-component`, both present in the plugin; backlinks added in both directions |
| `skills/figma-to-component/` | `skills/figma-to-component/` | copied | Line 172 reka-ui internal cross-ref replaced with an inline summary of the data-attribute pattern and a link to reka-ui's public docs. A `## Related skills` footer points at `figma-export-assets`, `figma-design-analysis`, and `component-architecture` |
| `skills/changeset/` | — | **excluded** | Removed in trimming pass (monorepo-internal release workflow) |
| `skills/discoveryjs/` | — | **excluded** | Removed in trimming pass |
| `skills/json-yaml-tools.md` | — | **excluded** | Removed in trimming pass |
| `skills/writing-section-block-migration-manifest/` | — | **excluded** | Removed in trimming pass — also resolved the most critical internal-tool leak (`@laioutr-private/component-schema-diff`) |
| `skills/plan-housekeeping/` | — | **excluded** | Internal workflow ("/plan-housekeeping" cleanup of repo plans) |
| `skills/supabase-table-schemas.md` | — | **excluded** | Cockpit-internal Supabase project, not relevant externally |

## Agents

No agents are bundled in this plugin. The directory and all previously
copied agents (`api-documenter`, `architect-reviewer`, `typescript-pro`,
`vue-expert`, `vue-nuxt-expert`) were dropped — the plugin focuses on
skills, rules, and the docs MCP server. External developers can still
delegate to general-purpose subagents in their own Claude setup.

For reference, these `.claude/agents/*.md` files exist in the source
repo but are **not** carried over:

| Source path | Action | Notes |
| --- | --- | --- |
| `agents/api-documenter.md` | **excluded** | Not bundled |
| `agents/architect-reviewer.md` | **excluded** | Not bundled |
| `agents/typescript-pro.md` | **excluded** | Not bundled |
| `agents/vue-expert.md` | **excluded** | Not bundled |
| `agents/vue-nuxt-expert.md` | **excluded** | Not bundled |
| `agents/cli-developer.md` | **excluded** | Not needed for the external Frontend/Orchestr workflow |
| `agents/dx-optimizer.md` | **excluded** | Not needed for the external Frontend/Orchestr workflow |
| `agents/nextjs-developer.md` | **excluded** | Cockpit-internal stack |
| `agents/react-specialist.md` | **excluded** | Cockpit-internal stack |

## Rules

13 of 23 rules from `.claude/rules/` were carried over and rewritten for an external-developer audience. Excluded:

| Source path | Action | Notes |
| --- | --- | --- |
| `rules/worktree-setup.md` | **excluded** | Laioutr-internal git worktree setup |
| `rules/changeset.md` | **excluded** | Monorepo-internal release workflow |
| `rules/git-reset.md` | **excluded** | Internal git-safety guidance, not specific to the platform |
| `rules/lint-after-ui-changes.md` | **excluded** | Tied to `turbo run @laioutr-core/...#lint` monorepo commands |
| `rules/no-intra-package-aliases.md` | **excluded** | Monorepo intra-package import rules |
| `rules/rename-workflow.md` | **excluded** | References internal monorepo touchpoints (Cockpit, registry strings, monorepo-wide LSP search) |
| `rules/commit-scope-ui.md` | **excluded** | Pure monorepo commitlint scope convention; external repos have their own scopes |
| `rules/husky-lint-autofix.md` | **excluded** | Removed in trimming pass — tied to a specific ESLint+Husky setup |
| `rules/swiper-breakpoints.md` | **excluded** | Removed in trimming pass — only relevant when matching Laioutr's canonical Swiper spacing |
| `rules/zod-schemas.md` | **excluded** | Removed in trimming pass |

### Layer-specific rules

In a second pass we pulled in the package-local `.claude/rules/` rules from
`packages/ui-kit/`, `packages/ui/`, and `packages/ui-app/`, plus each
package's `CLAUDE.md` summary. They were placed in `rules/ui-kit/`,
`rules/ui/`, and `rules/ui-app/` subfolders to avoid name collisions
(`package-role.md`, `no-viewport-stories.md`, `css-first-responsive.md`
each existed in multiple packages).

| Source path | Plugin path | Action |
| --- | --- | --- |
| `packages/ui-kit/CLAUDE.md` | `rules/ui-kit/CLAUDE.md` | copied + cross-ref rewritten |
| `packages/ui-kit/.claude/rules/*.md` (13 files) | `rules/ui-kit/*.md` | copied + `packages/...` paths and cross-refs rewritten |
| `packages/ui/CLAUDE.md` | `rules/ui/CLAUDE.md` | copied + cross-ref rewritten + `/Users/sl/src/docs` reference removed |
| `packages/ui/.claude/rules/no-viewport-stories.md` | `rules/ui/no-viewport-stories.md` | copied |
| `packages/ui/.claude/rules/package-role.md` | `rules/ui/package-role.md` | copied |
| `packages/ui/.claude/rules/storybook-categorization.md` | `rules/ui/storybook-categorization.md` | copied |
| `packages/ui/.claude/rules/ui-kit-first.md` | `rules/ui/ui-kit-first.md` | copied + Glob example rewritten to `node_modules/@laioutr-core/ui-kit/` |
| `packages/ui/.claude/rules/z-ordering.md` | `rules/ui/z-ordering.md` | copied — points at `docs.laioutr.io` which the docs MCP can also reach |
| `packages/ui/.claude/rules/css-first-responsive.md` | — | **excluded** — only 7 lines, a pointer to the ui-kit version |
| `packages/ui-app/CLAUDE.md` | `rules/ui-app/CLAUDE.md` | copied + cross-ref rewritten |
| `packages/ui-app/.claude/rules/*.md` (7 files) | `rules/ui-app/*.md` | copied + `packages/...` paths rewritten + Cockpit Studio references softened to "Studio" |

### Layer-rule rewrites (audit pass)

| Rule | Changes |
| --- | --- |
| `ui-kit/no-reka-ui-type-leakage.md` | `packages/ui-kit/...` paths → `src/runtime/...` / `src`; cross-ref `../../../../.claude/rules/vue-sfc-cross-package-props.md` → `../vue-sfc-cross-package-props.md` |
| `ui-kit/no-outer-chrome-in-primitives.md` | Three cross-refs rewritten to relative `../`/`./` paths |
| `ui-kit/theming.md` | Cross-ref to `surface-tone.md` → relative `../`; concrete type-file path replaced with `@laioutr-core/ui-kit` package surface |
| `ui-kit/todo-figma-marker.md` | `grep -rn "TODO FIGMA" packages/ui-kit/src` → `grep -rn "TODO FIGMA" src` |
| `ui-kit/CLAUDE.md` | "Detailed patterns in `.claude/rules/`" → "in this folder (`rules/ui-kit/`)" |
| `ui/ui-kit-first.md` | Glob path `packages/ui-kit/src/runtime/**/components/*/` → `node_modules/@laioutr-core/ui-kit/**/components/*/` |
| `ui/CLAUDE.md` | `/Users/sl/src/docs` reference removed; `packages/ui-kit/CLAUDE.md` → relative `../ui-kit/CLAUDE.md`; docs MCP mentioned |
| `ui-app/CLAUDE.md` | "`.claude/rules/`" → "this folder" |
| `ui-app/schema-field-if.md` | `packages/expression/...` source path softened; `packages/ui-app/src/runtime/app/` → "your module's ui-app-layer" |
| `ui-app/block-naming.md` | `packages/ui-app/src/runtime/app/block/` → "your module's ui-app-layer" |
| `ui-app/section-config-standard.md` | `packages/ui-app/` and `packages/ui/` → "your module's ui-app and ui layers"; `packages/ui-app/CLAUDE.md` ref → relative `CLAUDE.md`; `packages/ui-app/src/runtime/app/shared-fields/` → "your module's `src/runtime/app/shared-fields/`"; `packages/core-types/src/fields/` → "`@laioutr-core/core-types` under the `fields` namespace"; "Cockpit Studio" → "Studio" |

### Rule rewrites (after audit pass)

These rules were edited to remove monorepo-internal references and to reframe path-based guidance ("`packages/ui-kit/...`") as references to the three UI layers (ui-kit / ui / ui-app) that external devs can implement in their own Nuxt modules.

| Rule | Changes |
| --- | --- |
| `parent-prefix-naming.md` | Removed broken cross-ref to internal `rename-workflow.md`; replaced with inline summary of the rename touchpoints |
| `public-css-api.md` | "`packages/ui-kit/`, `packages/ui/`, `packages/ui-app/`" → "three Laioutr UI layers"; removed broken ADR 0013 link |
| `story-icons-must-exist.md` | Rewrote `packages/ui-kit/src/runtime/app/types/theme.ts` path → `IconName` import from `@laioutr-core/ui-kit` |
| `storybook-stories.md` | Restructured: story conventions first, refactor workflow conditional; theme list framed as Laioutr-provided not as enforced; broken cross-refs removed |
| `surface-tone.md` | Layer references reframed; removed broken cross-refs to internal `.claude/rules/theming.md`, `section-config-standard.md`; removed `docs/plans/2026-05-07-...` internal path |
| `translations-tl-vs-useLocale.md` | Plugin source path → `@laioutr-core/ui-kit` reference |
| `typescript-refactoring.md` | "Hard requirement" softened to "prefer"; internal LSP-plugin chain reframed as a preference order with "use whatever is available in your setup" |
| `unique-component-names.md` | Full rewrite — "packages/ui-kit, ui, ui-app" → "three Laioutr UI layers in your Nuxt module"; collision-search example uses `src/runtime/**` |
| `vue-sfc-cross-package-props.md` | "monorepo package" → "another package"; `turbo run <pkg>#build` kept as one example alongside `pnpm build` / `nuxi build` / "or equivalent" |

If a copied rule contains references to internal paths or namespaces
(`@laioutr-private/*`, `/Users/sl/src/docs/`, Infisical), surface them in
review — they should be updated before the plugin is shipped publicly.

## Top-level files

| Source | Plugin | Action |
| --- | --- | --- |
| `.claude/CLAUDE.md` | `CLAUDE.md` | **rewritten** for external audience — internal build setup, Infisical, monorepo paths, and namespace internals removed; Orchestr handler types and section/block patterns kept |
| `.claude/settings.json` | — | excluded (project-local Claude config) |
| `.claude/settings.local.json` | — | excluded (machine-local) |
| `.claude/review/` | — | excluded (audit/review work-in-progress notes) |
| — | `.mcp.json` | **new** — registers an HTTP MCP server pointing at `https://docs.laioutr.io/mcp` so the plugin can search the Laioutr developer docs at runtime |
| — | `.claude-plugin/marketplace.json` | **new** — makes this repo a single-plugin Claude Code marketplace so it can be installed via `claude /plugin marketplace add laioutr/claude-plugin-developer` |

## Re-syncing from the monorepo

```bash
SRC=/path/to/laioutr/.claude
DST=/path/to/claude-plugin-developer

# Skills
for s in adr changeset component-architecture figma-design-analysis \
         figma-to-component writing-section-block-migration-manifest \
         discoveryjs; do
  rm -rf "$DST/skills/$s" && cp -r "$SRC/skills/$s" "$DST/skills/"
done
cp "$SRC/skills/json-yaml-tools.md" "$DST/skills/json-yaml-tools/SKILL.md"

# Agents
for a in api-documenter architect-reviewer typescript-pro vue-expert \
         vue-nuxt-expert; do
  cp "$SRC/agents/$a.md" "$DST/agents/"
done

# Rules (curated subset — see exclusions in the table above)
EXCLUDE='worktree-setup.md changeset.md git-reset.md lint-after-ui-changes.md no-intra-package-aliases.md rename-workflow.md'
for r in "$SRC/rules"/*.md; do
  name=$(basename "$r")
  case " $EXCLUDE " in *" $name "*) continue ;; esac
  cp "$r" "$DST/rules/"
done
```

Re-review `CLAUDE.md` manually after each sync — it's a curated rewrite,
not a copy.

## Known internal references in copied content

Full audit after a second-pass scan of all carried-over skills and rules.
Grouped by severity and category.

### A. Broken cross-references to internal monorepo-local rule files

These point at `.claude/rules/*` files that live **inside individual
packages** of the Laioutr monorepo and were not (and cannot be) carried
into this plugin. They render as dead links for external devs.

| File | Line | Broken reference |
| --- | --- | --- |
| `skills/figma-to-component/SKILL.md` | 172 | `packages/ui-kit/.claude/rules/reka-ui.md` |
| `rules/surface-tone.md` | 202 | `packages/ui-kit/.claude/rules/theming.md` |
| `rules/surface-tone.md` | 203 | `packages/ui-app/.claude/rules/section-config-standard.md` |
| `rules/storybook-stories.md` | 36 | `packages/ui-kit/.claude/rules/no-viewport-stories.md`, `packages/ui/.claude/rules/no-viewport-stories.md` |
| `rules/husky-lint-autofix.md` | 24 | `packages/ui-kit/.claude/rules/vue-template-type-casts.md` |
| `rules/parent-prefix-naming.md` | 50 | `packages/ui-kit/.claude/rules/component-structure.md` |

**Severity: medium.** Each is a hyperlink the external dev cannot follow.
Either remove the line, summarize the linked content inline, or rewrite
the cross-ref to point at the official Laioutr docs site.

### B. Internal tool / private package references

External devs do not have these packages or tools installed.

| File | Line(s) | Reference | Severity |
| --- | --- | --- | --- |
| `skills/writing-section-block-migration-manifest/SKILL.md` | 574, 878 | `pnpm --filter @laioutr-private/component-schema-diff start` + `docs/plans/2026-05-08-component-schema-diff-*.md` | **high** — external devs cannot run this tool or access these paths |
| `skills/changeset/SKILL.md` | 111 | `@laioutr-private/*` listed as ignored | low — informational only |
| `skills/changeset/references/packages.md` | 49 | `@laioutr-private/*` documented as excluded | low — informational only |

**Recommendation for migration-manifest:** ship a public equivalent of
`component-schema-diff`, or rewrite those sections to describe inputs and
outputs abstractly and drop the concrete commands.

### C. Internal Jira project keys — RESOLVED

`figma-design-analysis` previously used `COCKPIT-99` / `LUI-123` as
Jira-key examples. **Now generalized** to `PROJECT-123` / `TEAM-456`
placeholders. Lines 205 and 620 updated.

### D. Internal monorepo file paths used as concrete locations

The Laioutr monorepo writes plans under `docs/plans/` and ADRs under
`docs/adr/records/`. External devs may or may not follow the same
convention.

| File | Line(s) | Reference | Severity |
| --- | --- | --- | --- |
| `skills/adr/SKILL.md` | 15, 110, 112, 132 | `docs/adr/records/NNNN-slug.md`, `docs/plans/YYYY-MM-DD-*.md` | low — a documented convention, useful as a default |
| `skills/component-architecture/SKILL.md` | 528 | `docs/plans/YYYY-MM-DD-<topic>-component-architecture.md` | low — convention |
| `rules/surface-tone.md` | 13, 205 | `docs/plans/2026-05-07-surface-tone-refactor/` (a specific internal plan folder) | medium — this is a literal internal path |

**Recommendation:** D-low items can stay as suggested conventions. The
`2026-05-07-surface-tone-refactor/` reference is an internal artifact
and should be removed (the surrounding paragraph still makes sense
without it).

### E. Concrete `packages/ui-kit/`, `packages/ui/`, `packages/ui-app/` paths

Skills and rules use literal monorepo paths in `grep` examples and
file-location guidance.

| File | Sample line | Notes |
| --- | --- | --- |
| `skills/figma-design-analysis/SKILL.md` | 302, 305, 317, 320, 373, 399, 413–425, 477, 480, 496–505, 627–635, 773–793 | Heavy use of `packages/ui-kit/`, `packages/ui/`, `packages/ui-app/`, `packages/ui-tokens/` paths |
| `skills/figma-to-component/SKILL.md` | 59, 426 | Uses `packages/...` paths and `#ui-kit/components/...` aliases |
| `skills/component-architecture/SKILL.md` | 283, 286, 290 | `grep packages/ui-kit/...` examples |
| `rules/surface-tone.md` | 3, 24, 166 | References `packages/ui-kit/`, `packages/ui/`, `packages/ui-app/` |
| `rules/unique-component-names.md` | 3, 11 | Same |

**Severity: ambiguous.** Two readings:

1. *External devs reuse the same three-layer architecture in their own
   repos* → the path names are conventional shorthand, fine to keep.
2. *External devs consume Laioutr as packages (`@laioutr-core/ui-kit`,
   etc.) and do not have a `packages/ui-kit/` directory of their own*
   → the paths are misleading and should be rewritten as package
   imports.

**Recommendation:** add a clarifying paragraph in `CLAUDE.md` (or in the
README) stating which reading applies, and decide once whether to
rewrite or to leave the paths as architectural conventions.

### F. Internal-stack mentions

Some references are not broken, but identify Laioutr-internal authoring
surfaces. They are likely fine to keep.

| File | Line(s) | Reference | Action |
| --- | --- | --- | --- |
| `rules/surface-tone.md` | 192 | `Studio` (Laioutr CMS authoring environment) | keep — Studio is the customer-facing authoring surface |
| `rules/vue-sfc-cross-package-props.md` | 38 | `Studio schemas` | keep |
| `skills/component-architecture/*` | various | `orchestr`, `frontend-core`, `@laioutr-app/*`, `@laioutr-core/canonical-types`, `@laioutr-core/core-types` | keep — these are public packages and core platform concepts |
| `skills/figma-design-analysis/SKILL.md` | 246, 487 | Orchestr handlers as data source | keep — relevant for external devs |

No Cockpit-specific code references were found in the kept skills or
rules (no Chakra UI, no tRPC, no Liveblocks/Yjs/Valtio/Zustand, no
oclif, no `apps/cockpit/`, no `apps/cli/`). The only `COCKPIT-*`
mentions are example Jira keys (see Section C).

## Status of recommended actions

| # | Action | Status |
| --- | --- | --- |
| 1 | **Section E** — rewrite `packages/...` paths to layer references in **rules** | ✅ done in rules; **still pending** in `skills/figma-design-analysis/SKILL.md` (~25 lines) and `skills/component-architecture/SKILL.md` (lines 283, 286, 290) — these are the most invasive skills and were left untouched pending decision |
| 2 | **Section A** — six broken cross-references to monorepo-local `.claude/rules/*` files | ✅ all six resolved (5 in rules, 1 in `figma-to-component/SKILL.md` line 172) |
| 3 | **Section C** — Jira-key examples | ✅ done (`COCKPIT-99` → `PROJECT-123`, `LUI-123` → `TEAM-456`) |
| 4 | **Section B-high** — `@laioutr-private/component-schema-diff` refs in `writing-section-block-migration-manifest` | ❌ not yet — skill needs either a public tool equivalent, or the concrete commands rewritten as abstract input/output descriptions |
| 5 | **Section D-medium** — `docs/plans/2026-05-07-surface-tone-refactor/` literal | ✅ done — reference removed from `rules/surface-tone.md` |

## Open work

The skill bodies (`figma-design-analysis`, `figma-to-component`,
`component-architecture`, `writing-section-block-migration-manifest`)
still contain monorepo-internal `packages/...` paths used as concrete
file locations in `grep` examples and architectural placement guides.
Rewriting these requires the same rephrasing pattern applied to the
rules — "your module's `src/runtime/...`" or import-from-package
references. Pending a decision on whether to invest the same level of
effort in skill rewrites.

## Cleanup pass (post-initial-publish)

After publishing the initial mirror, the rules tree was audited a second
time against the actual external-developer mental model: **one Nuxt
module, multiple component roles** (atom-style, organism, section/block)
— not three separate packages. The internal monorepo's three-package
split lives only in `@laioutr-core/*`, which the external dev consumes.

Structural changes:

- **Subfolders flattened.** `rules/ui-kit/`, `rules/ui/`, `rules/ui-app/`
  collapsed into a flat `rules/` directory. Role is encoded by filename
  prefix (`ui-kit-*`, `ui-*`, `ui-app-*`); cross-cutting rules have no
  prefix. The folder structure was reinforcing the wrong "you have three
  packages" mental model.
- **`three-layer-architecture.md` added** to replace the three deleted
  `package-role.md` files. Frames the layers as upstream packages the
  dev consumes, and documents the resolution ladder when upstream is
  missing something: (1) prop / minor CSS override, (2) local custom
  component in the dev's module, (3) fork from
  <https://github.com/laioutr/ui-source>, (4) upstream issue. The
  forking option is explicit — not the default, but valid.
- **Three discovery surfaces documented:** docs MCP (`laioutr-docs` →
  `https://docs.laioutr.io/mcp`) for props/emits/slots/usage,
  Storybook (<https://storybook.laioutr.cloud/>) for visual states,
  and `laioutr/ui-source` for implementation/forking.

Files dropped (10):

| File | Reason |
| --- | --- |
| `rules/ui-kit/CLAUDE.md`, `rules/ui/CLAUDE.md`, `rules/ui-app/CLAUDE.md` | Folder-level summaries; superseded by the flat Rule Index in top-level `CLAUDE.md` |
| `rules/ui-kit/package-role.md`, `rules/ui/package-role.md`, `rules/ui-app/package-role.md` | Merged into `rules/three-layer-architecture.md` |
| `rules/ui-kit/storybook-atoms-categorization.md`, `rules/ui/storybook-categorization.md` | Prescribe Laioutr-internal Storybook taxonomy that external devs pick on their own |
| `rules/ui-kit/no-viewport-stories.md`, `rules/ui/no-viewport-stories.md` | Absorbed as a section into top-level `rules/storybook-stories.md` |

Files renamed (18) — all moved to top-level `rules/` with role prefix:

| Old path | New path |
| --- | --- |
| `rules/ui-kit/component-structure.md` | `rules/vue-component-file-structure.md` (reframed as general Vue SFC structure) |
| `rules/ui-kit/reka-ui-wrapper-patterns.md` | `rules/reka-ui-wrapper-patterns.md` (applies anywhere you wrap reka-ui) |
| `rules/ui-kit/no-reka-ui-type-leakage.md` | `rules/reka-ui-no-type-leakage.md` (same) |
| `rules/ui-kit/css-first-responsive.md` | `rules/ui-kit-css-first-responsive.md` |
| `rules/ui-kit/no-outer-chrome-in-primitives.md` | `rules/ui-kit-no-outer-chrome-in-primitives.md` |
| `rules/ui-kit/reka-ui.md` | `rules/ui-kit-reka-ui.md` |
| `rules/ui-kit/styling.md` | `rules/ui-kit-styling.md` |
| `rules/ui-kit/theming.md` | `rules/ui-kit-theming.md` |
| `rules/ui-kit/todo-figma-marker.md` | `rules/ui-kit-todo-figma-marker.md` |
| `rules/ui-kit/vue-template-type-casts.md` | `rules/ui-kit-vue-template-type-casts.md` |
| `rules/ui/ui-kit-first.md` | `rules/ui-ui-kit-first.md` |
| `rules/ui/z-ordering.md` | `rules/ui-z-ordering.md` |
| `rules/ui-app/block-naming.md` | `rules/ui-app-block-naming.md` |
| `rules/ui-app/forbidden-field-names.md` | `rules/ui-app-forbidden-field-names.md` |
| `rules/ui-app/schema-field-if.md` | `rules/ui-app-schema-field-if.md` |
| `rules/ui-app/section-config-standard.md` | `rules/ui-app-section-config-standard.md` |
| `rules/ui-app/shared-field-factories.md` | `rules/ui-app-shared-field-factories.md` |
| `rules/ui-app/shared-field-options.md` | `rules/ui-app-shared-field-options.md` |

Content edits (residual leaks + reframing):

- **Reframed for consumer + author** (no longer assume the reader edits
  upstream packages): `vue-component-file-structure.md`,
  `ui-kit-no-outer-chrome-in-primitives.md`, `reka-ui-wrapper-patterns.md`,
  `reka-ui-no-type-leakage.md`.
- **Resolution-ladder rewrite** with forking option:
  `ui-ui-kit-first.md`, `ui-app-block-naming.md` (line 39).
- **Phase / target-state framing stripped**: `surface-tone.md` (status
  callout, Phase 2/3/4 section headers, upstream-component example
  list, narrowing-verdict block, portal-handling target-state),
  `public-css-api.md` (status column in override-surfaces table,
  forward-looking migration section, ADR link, broken
  `lint-after-ui-changes.md` cross-ref), `ui-app-section-config-standard.md`
  (Figma file ID `QgRgNtTxBTCAxpTe1rriHM`, broken `CLAUDE.md`
  cross-ref, "Renames enforced by this standard" migration table,
  Phase 2 preset migration, "Gaps and follow-ups" section).
- **Residual monorepo leaks fixed**: `no-wrapper-css-overrides.md`
  (broken ADR 0013 link at lines 5 + 98, three-layer phrasing at line
  68), `vue-script-imports-single-block.md` (husky reference at line
  30 → generic "your linter"), `unique-component-names.md` (line 3
  "across all three layers" → "within your module"),
  `story-icons-must-exist.md` (rg dist path → docs MCP / Storybook),
  `translations-tl-vs-useLocale.md` (line 50 relative
  `../../composables/useLocale` → `#ui-kit/composables/useLocale`),
  `ui-app-schema-field-if.md` (Cockpit-internal
  `apps/cockpit/src/features/project/studio/...` paths → generic
  "Studio's schema-conditions evaluator").
- **Stale references**: top-level `CLAUDE.md`'s dead pointer to the
  excluded `rules/commit-scope-ui.md` removed; pointer to the excluded
  `writing-section-block-migration-manifest` skill removed; Context7-vs-laioutr-docs preference clarified
  (Laioutr docs MCP first, Context7 fallback).
- **`.mcp.json` URL fix**: `https://docs.laioutr.com/mcp` →
  `https://docs.laioutr.io/mcp` (the canonical URL per
  <https://docs.laioutr.io/getting-started/mcp-server>).

Net rule count: 31 rules + 1 architecture page = 32 files in
`skills/laioutr-platform/rules/` (down from 38 + 3 layer `CLAUDE.md`
files).

Note: `#ui-kit/...`, `#ui/...`, and `#frontend-core` Nuxt aliases are
public API surface of the upstream packages, not monorepo-internal
paths — they may appear in rules and examples freely.

## Architecture migration (skill-first)

Earlier ship assumed a plugin-root `CLAUDE.md` would be auto-loaded into
Claude's context. This is **not** how Claude Code plugins work — the
official plugins reference is explicit (*"A `CLAUDE.md` file at the
plugin root is not loaded as project context. Plugins contribute context
through skills, agents, and hooks rather than `CLAUDE.md`. To ship
instructions that load into Claude's context, put them in a skill."*),
and `claude plugin validate` emits the same warning verbatim. With the
old layout, the platform overview, discovery surfaces, Orchestr handler
table, commit conventions, and the rule index never reached the model.

Migration applied:

- **New skill: `skills/laioutr-platform/SKILL.md`** carries everything
  the dead `CLAUDE.md` used to carry. Frontmatter description matches on
  any Laioutr task — mentions of "Laioutr", `@laioutr-core/*`,
  `@laioutr-app/*`, `@laioutr-org/*`, `defineSection`, `defineBlock`,
  `ui-kit`, `ui-app`, Orchestr, storefront, etc. — so it auto-loads
  whenever the plugin's audience is at the keyboard.
- **`rules/` moved under the skill** (`git mv rules
  skills/laioutr-platform/rules`). Relative paths in the rule index
  (`./rules/<file>.md`) and in cross-rule references (`./<file>.md`)
  remain valid.
- **Plugin-root `CLAUDE.md` deleted.** README.md is now the only
  developer-facing doc at the plugin root.
- **`plugin.json` cleanup**: dropped the `version` field (the
  `marketplace.json` entry owns version per the docs anti-pattern
  guidance), translated the German `description` to English.
- **README rewrites**: removed the "auto-loaded `CLAUDE.md`" claim,
  documented the new two-step on-demand loading flow, fixed rule and
  skill counts, refreshed the Layout tree.

Re-sync impact: anything previously written to the plugin-root
`CLAUDE.md` (Working Style, Platform Overview, Orchestr table, Rule
Index updates) now belongs in `skills/laioutr-platform/SKILL.md`. Do not
re-introduce a plugin-root `CLAUDE.md` — the validator will warn and
the content will be invisible to Claude.

Open items deliberately deferred:

- **Oversize `SKILL.md` bodies**: `skills/figma-design-analysis/SKILL.md`
  (~824 lines) and `skills/component-architecture/SKILL.md` (~640
  lines) are over the 500-line working ceiling cited by Anthropic's
  skill-authoring guidance. Body should be split into sibling
  `reference/*.md` files at one level of depth. Not yet done.
- **Further skill promotions**: research surfaced `using-reka-ui` (3
  refs: `ui-kit-reka-ui`, `reka-ui-wrapper-patterns`,
  `reka-ui-no-type-leakage`) and `using-surface-tone` (single dense
  rule) as next promotion candidates. Deferred for now to keep the
  skill-listing budget headroom.

## Rules-as-references rename + first promotions

After research on how other Claude Code plugins expose rule libraries
(Anthropic's `plugin-dev/*` skills, `frontend-design`,
`security-guidance`, hashicorp/agent-skills, obra/superpowers), we
aligned with the Anthropic convention and promoted two high-volume
task surfaces to their own skills.

- **`rules/` renamed to `references/`** to match the Anthropic
  convention (used by every first-party skill that ships sibling
  reference material). The `skill-reviewer` agent in `plugin-dev`
  explicitly checks for this folder name. Same content, same on-demand
  loading model.

- **New skill: `writing-storybook-stories`.** The full content of
  `storybook-stories.md` now lives in the skill body
  (`skills/writing-storybook-stories/SKILL.md`). Reference file deleted
  from `laioutr-platform/references/`. Trigger: any `.stories.ts`
  authoring or refactor task.

- **New skill: `writing-section-block-definitions`.** Bundles 6 ui-app
  references (the highest-volume external-developer task surface):
  `section-config-standard.md`, `forbidden-field-names.md`,
  `schema-field-if.md`, `block-naming.md`, `shared-field-factories.md`,
  `shared-field-options.md`. All six were `git mv`'d from
  `laioutr-platform/references/` to
  `writing-section-block-definitions/references/` with the `ui-app-`
  prefix dropped (the skill folder makes the role explicit). Internal
  cross-references between the six updated; one external cross-ref
  (`block-naming.md` → `three-layer-architecture.md`) rewritten to
  `../../laioutr-platform/references/three-layer-architecture.md`. The
  platform skill's "Sections & Blocks" rule-index section replaced with
  a pointer to the new skill.

Decision rationale (per [research §2 / §3](/tmp/claude-plugin-best-practices.md
and the survey of `plugin-dev`, `frontend-design`, `superpowers`,
`hashicorp/agent-skills`)): we ruled out promoting every rule into its
own skill — at 31 rules × ~150 description tokens ≈ 4.7 K, the
skill-listing budget (~1 % of context, ~2 K tokens) would overflow and
descriptions would silently truncate. We also flagged the [active
`paths:`-glob discoverability bug](https://github.com/anthropics/claude-code/issues/49835)
as a reason to avoid per-rule promotion in the near term. The chosen
pattern — one always-on platform skill plus task-triggered sibling
skills for the highest-volume authoring surfaces — keeps the budget
headroom while still surfacing the two most frequently-needed
rulesets through their own descriptions.

Final shape:

- 1 always-on platform skill (`laioutr-platform`) with 25 cross-cutting
  references + 1 architecture page in `references/`.
- 2 task-triggered convention skills (`writing-storybook-stories`,
  `writing-section-block-definitions`) — the second carries its own
  `references/` with 6 deep-detail files.
- 5 workflow skills carried over from the monorepo (`adr`,
  `component-architecture`, `figma-design-analysis`,
  `figma-to-component`, `writing-claude-rules`).

Net `references/` count: **25 + 6 = 31 deep-detail files + 1
architecture page**. The reduction from "32 rule files under one
skill" to a split layout is the deliberate result of the
storybook-content-into-skill-body absorption.

Re-sync impact: when re-syncing from the monorepo, the
section/block-definition rules go under
`writing-section-block-definitions/references/` (no `ui-app-` prefix);
the storybook conventions get folded into `writing-storybook-stories/SKILL.md`
directly (no separate reference file).
