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
| `skills/figma-design-analysis/` | `skills/figma-design-analysis/` | copied | `COCKPIT-99` / `LUI-123` Jira keys generalized to `PROJECT-123` / `TEAM-456`. Still contains heavy `packages/ui-kit/`, `packages/ui/`, `packages/ui-app/` path usage |
| `skills/figma-to-component/` | `skills/figma-to-component/` | copied | Line 172 reka-ui internal cross-ref replaced with an inline summary of the data-attribute pattern and a link to reka-ui's public docs |
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
| — | `.mcp.json` | **new** — registers an HTTP MCP server pointing at `https://docs.laioutr.com/mcp` so the plugin can search the Laioutr developer docs at runtime |
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
