# laioutr-developer — Plugin Maintenance Guide

For plugin **maintainers** working inside `plugins/laioutr-developer/`. End users install the plugin and read [`README.md`](./README.md); this file is for the person editing the skills, references, and MCP config.

## The two-tier load model

- **Skills** (`skills/<name>/SKILL.md`) auto-load when their `description` matches the user's prompt. SKILL.md bodies are eagerly loaded into the conversation — token budget matters per-conversation.
- **References** (`skills/<name>/references/*.md`) are NOT auto-loaded. The parent SKILL.md indexes them with one-line triggers; Claude opens them on demand. References can be longer because they only load when asked.

Implication: keep SKILL.md bodies orientation-shaped (route, don't duplicate). Put depth in references.

## Disciplines for editing

Invoke the matching skill BEFORE editing the corresponding file shape:

- **`writing-claude-rules`** (plugin-shipped) for any reference (rule) file. H1 claims a position, rule statement before any subheader, `## Why` is load-bearing, `✘ / ✅` examples for code-shape rules, no invented exceptions.
- **`superpowers:writing-skills`** (external — Anthropic Superpowers marketplace) for any SKILL.md edit. RED-GREEN-REFACTOR via subagent pressure tests. No skill edit without a failing test first.

If `superpowers:writing-skills` isn't installed, apply the same discipline manually: write a pressure scenario, run it against a subagent with the current SKILL.md, observe failures, edit, re-run.

## CSO trigger tests (description-only)

To test whether the right skill auto-loads for a prompt (separately from whether the body teaches the right thing), dispatch a subagent with **only the skill descriptions** (no bodies, no paths) and a battery of task prompts. The subagent matches each prompt to a skill purely from the description text. This isolates the discovery surface from the implementation surface.

Useful when you suspect a description is too narrow (real triggers don't match) or too eager (unrelated prompts match).

## The three reference shapes

References fall into three shapes — pick the shape before writing:

1. **Single-position rule.** Examples: `parent-prefix-naming.md`, `nuxt-hooks.md`, `no-wrapper-css-overrides.md`. Apply `writing-claude-rules` strictly.
2. **Multi-rule bundle.** Example: `ui-kit-styling.md` ships tokens + CSS approach + responsive media queries in one file. This violates writing-claude-rules' *"if you find yourself writing a second rule inside the first, split."* Refactor into multiple single-position files when touching.
3. **System explainer.** Examples: `surface-tone.md`, `schema-field-if.md`. Too complex for a single position — describes a system with channels, tiers, operators. Apply a looser discipline: clarity, completeness, navigable section structure, a "Hard rules" / "Forbidden patterns" section to compress the prescriptive layer. Don't force the writing-claude-rules shape onto these.

## Umbrella vs specialist skills

`laioutr-platform` is the **umbrella** — it triggers on any Laioutr work and co-loads on almost every prompt. The other Laioutr skills (`writing-section-block-definitions`, `writing-storybook-stories`, the figma family) are **specialists** that load alongside the umbrella when their narrower trigger matches.

This is by design (hierarchical partition, not disjoint). When you find yourself wondering "shouldn't only one of these load?", remember the umbrella is intentional context, not a duplicate.

## Monorepo leakage to watch for

This plugin is curated from Laioutr's internal `.claude/` config. When re-syncing or adding new content, watch for:

- Hardcoded paths like `packages/ui-kit/` — rewrite as "the ui-kit layer in your project" or use the public `@laioutr-core/*` package names.
- Internal namespaces (`@laioutr-private/*`) — strip; not externally visible.
- Tooling references that assume the monorepo's `playground/` directory or internal release pipelines — generalize or remove.
- Workflows that only make sense for the core team (e.g. forking `laioutr/ui-source` to land upstream patches) need framing as "the rare external-dev case."

[`MAPPING.md`](./MAPPING.md) is the carry-over registry. Update it when adding, splitting, or removing files. It also tracks deferred cleanup (oversize SKILL bodies, pending skill promotions, refs still containing monorepo paths).

## Public Nuxt aliases vs monorepo internals

`#frontend-core`, `#ui-kit/...`, `#ui/...` are **public Nuxt aliases** exposed by `@laioutr-core/frontend-core` — external devs use these directly. Don't rewrite them as monorepo-relative paths; they ARE the public surface.

## Cross-plugin dependencies

Plugin-bundled skills don't depend on `superpowers:*`. If you reference a superpowers skill in plugin content, frame it as optional ("if you have `superpowers:test-driven-development` installed, use it; otherwise apply the same discipline manually").

## Don't put session state here

Durable orientation only. Not:
- "We just split X" — that's git history.
- "Current todo: rewrite Y" — use `MAPPING.md` or a task tracker.
- "Sebastian prefers Z" — that's auto-memory if it's behavioral, not a project convention.

Conventions and gotchas, nothing that decays.
