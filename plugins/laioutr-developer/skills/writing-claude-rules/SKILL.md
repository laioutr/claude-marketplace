---
name: writing-claude-rules
description: Use when authoring a new rule file in a project's `rules/` or `.claude/rules/` directory, when a user asks to "write a rule for X" / "add a rule about Y" / "encode this convention", or when a recurring code-review correction needs to be turned into project memory.
---

# Writing Claude Rules

## Overview

A rule is a **position** (one prescriptive stance), with **evidence** (forbidden vs required examples), and a **reason** (so Claude can judge edge cases). That's the whole shape. Everything below is in service of those three.

This skill is platform-agnostic. It applies whether the project uses:

- Anthropic-native `.claude/rules/*.md` — auto-loaded, optionally path-scoped via a `paths:` YAML frontmatter glob.
- An on-demand `rules/` directory — markdown files read only when a trigger sentence in CLAUDE.md matches the task.

## Before You Write — Is a Rule the Right Vehicle?

Most "we should write a rule for that" instincts should land somewhere else. Sort first:

| Symptom | Vehicle |
|---|---|
| Purely mechanical check (formatting, unused imports, naming regex) | **Lint rule / formatter** — Claude is expensive and slow vs. ESLint |
| Must run at a fixed lifecycle moment (pre-commit, post-edit, on-stop) | **Hook** in `.claude/settings.json` |
| Multi-step workflow with branching | **Skill** (this is one) |
| Only relevant in the current PR / sprint | **Conversation**, not encoded |
| Already discoverable from the code (file paths, layer names) | **CLAUDE.md** map, or trust Claude to read |
| Judgment call where Claude routinely picks the wrong default | **Rule** ✓ |

**Don't write a rule that:**

- A linter could enforce instead.
- Restates conventions Claude follows correctly without being told (dead instructions burn the instruction budget).
- Encodes session-only state ("we're freezing X this week").

If you can't articulate the failure case the rule prevents, don't write it yet.

## Verify Before Encoding

Agents fabricate when they don't know. Specific things to **grep / read** before writing:

- **Alias names and import paths** — check `tsconfig.json` `paths`, `nuxt.config.ts` `alias`, `package.json` `imports`. Don't invent.
- **Lint rule names** — confirm `vue/no-async-in-setup` actually exists before claiming it does.
- **File and directory paths** — `src/test/factories/` only exists if you saw it.
- **Exceptions** — if you can't name a real case from the codebase or a real prior incident, omit the Exceptions section. Manufactured exceptions make the rule weaker, not stronger.

If you can't verify a specific, write the generic form ("the configured package alias for this directory") instead of inventing the specific.

### Push back when verification contradicts the framing

The user's brain-dump is the starting point, not the spec. When grepping turns up evidence that the framing is too broad, too narrow, or wrong, **correct the rule's position before encoding it** — and tell the user what you found.

Examples:

- User says "ban all top-level `await` in `<script setup>`", but the canonical Nuxt pattern *is* `await useFetch(...)`. The right rule is "no raw `await` of fetch helpers" — not "no top-level await".
- User says "deep relative imports are bad", but intra-package `../../` is the established pattern in the repo. The right rule is "no deep relatives **across package boundaries**" — not blanket.

A rule that contradicts its own codebase will be ignored or, worse, applied wrongly. The skill's verification step is the moment to catch this.

## When the Rule Isn't Crisp Enough Yet

Some rules aren't ready to encode. Detect this before writing anything, then either narrow the scope or send it back.

**Signals the rule isn't ready:**

- You can't write a real Forbidden example and a real Required example that are clearly different shapes.
- The "do X" half is vague ("write cleaner components") while only the "don't Y" half is concrete — or vice versa.
- The brief is **one incident, not a pattern** — you can't find a second instance of the same shape in the codebase or in the user's mental model.
- You'd have to invent specifics — an alias name, an exception, a lint rule — that the user can't confirm.
- The brief is actually two or three rules colliding, and you can't pick a single position without arbitrating.
- You can't fill in the **Why** section without speculating about the user's reasoning.

**What to do — in this order:**

1. **Don't write the rule yet.** A vague rule that ships is worse than no rule — it gets applied wrongly or ignored, and it teaches the team to stop trusting the `rules/` directory.
2. **Ask narrow, specific questions — not "tell me more".** Good shapes:
   - "Can you point me to one file in the repo that gets this wrong today, and one that gets it right?"
   - "When two people on the team disagree about this, which side wins — and what's the tiebreaker?"
   - "Is this a rule about *what* the code looks like, or about *when* you reach for this pattern?"
   - "If we ship this rule and someone follows it, what bug or smell does that prevent?"
   - "What's the closest existing rule? Should this extend it, or contradict it?"
3. **Research before asking — when the answer might already exist.** Pick the right source for the question:
   - **Codebase** — when the question is "does this pattern exist here?". Grep / read for the good shape and the bad shape. If you find both, the rule is encodable. If only one (or neither), say so — that's the answer the user needs.
   - **Framework / library docs** — when the question is "what does Vue / Nuxt / Reka UI / Pinia consider idiomatic?". Check upstream docs first (via `context7` MCP if available, then the project's official site). Cite what you find. The framework's position is **informative, not authoritative** — the team's rule still has to fit their codebase, but you save a round-trip if the framework has already taken a clear stance.
   - **Web search** — only when the question is genuinely community-level ("how do teams typically handle X?"), not project-specific. Treat individual blogs as opinions; require multiple authoritative sources to call something "consensus." If sources disagree, surface the disagreement to the user rather than picking a side.
   - **Don't research instead of asking.** External research narrows the question and gives the user something to react to ("Nuxt docs say X — does that match how we want to do it here?"); it doesn't replace asking.
4. **Propose a narrower sub-rule that you *can* write,** and ask whether that's the right scope to start with. "I can write a rule for X today — does that cover what you have in mind, or do you want me to wait until the broader scope is clearer?"
5. **If nothing crisps after one round of clarification, recommend leaving the rule unwritten.** "Not yet" is a legitimate outcome — note what's missing (e.g. "revisit when there's a second incident with the same shape") and move on.

The goal is not to ship a rule per request — it's to ship rules that hold up. A skipped rule with a clear "revisit when X" note is better than an authored rule that quietly contradicts the codebase.

## Required Structure

```markdown
# <Prescriptive H1 — claims a position>

<1–3 sentence rule statement. Direct imperative. State the rule before any subheader.>

<Forbidden / Required code examples — with ✘ and ✅ markers — when the rule is about code shape. Either inline under "## Forbidden / ## Required" headers, or in a single block under "## The rule".>

## Why

1. **<Reason 1 as a bold lead>.** <One paragraph: mechanism, consequence, or prior incident.>
2. **<Reason 2>.** <…>

## When to apply  /  When this doesn't apply

<Use one or both. Lists, not prose, when the boundary is non-obvious.>

## How to fix  (or: What to do when you hit this)

<Numbered remediation steps.>

## Exceptions  (optional — omit if you can't name real cases)

- **<Concrete case>.** <Why it's exempt.>

## Related

- [`other-rule.md`](./other-rule.md) — <one-line connection>
```

**The H1 claims a position.** "Parent-Prefix Naming Is Not the Default" claims a position. "About Parent Prefixes" doesn't. "No Wrapper CSS Overrides" is fine; "CSS Style Guide" isn't.

**Rule statement comes first, before any subheader.** That's the line a Claude scanning the file in 200ms needs to find.

**Voice: direct imperatives.** "Use X." "Never Y." Strike "we generally prefer", "in most cases", "try to avoid".

**Why is load-bearing.** Without it, Claude either over-applies the rule or abandons it under pressure. The reasons are what let it judge an edge case in front of it. Numbered, bolded lead clause, one paragraph per reason. Three to five is usually right.

**Forbidden + Required examples** for any code-shape rule. Real code, valid in the codebase, with `// ❌` and `// ✅` (or `✘ / ✅`) markers. No pseudo-code.

## Length Is Not a Metric — Fit Is

A 3-line rule (e.g. `nuxt-hooks.md`) is correct when the rule is mechanical and uncontroversial. A 200-line rule (`ui-app-section-config-standard.md`) is correct when the topic genuinely has fields, sub-cases, or a design-tool source-of-truth.

Three paragraphs for one idea = correct. Six sections for the same idea = bloat — cut.

If you find yourself writing a second rule inside the first, split.

## Naming and Placement

- **Filename:** `kebab-case`, descriptive of the position. `no-wrapper-css-overrides.md` over `css.md`. `parent-prefix-naming.md` over `naming.md`. Prefix with a layer / domain when the index groups by layer (`ui-app-block-naming.md`).
- **Location:**
  - Anthropic-native, auto-loaded by file glob → `.claude/rules/<slug>.md` with `paths:` frontmatter.
  - On-demand, indexed from CLAUDE.md → `rules/<slug>.md`, plus a trigger entry in the index (see below).

## Update the Index

For on-demand rule systems, the **trigger sentence** in CLAUDE.md is the discovery surface. Claude only opens the rule if the trigger matches the task.

After writing a rule:

1. Open the project's CLAUDE.md rule index.
2. Add `- rules/<slug>.md — <trigger phrase>` in the right section.
3. **Trigger phrase = the situation in which the rule should fire, in the reader's words.** "Before writing CSS in a parent that targets a child component's class names" beats "CSS encapsulation". Be specific about the moment, not the topic.

For Anthropic-native rules with `paths:` frontmatter, no index update needed — the glob is the trigger.

## Common Mistakes

| Mistake | Fix |
|---|---|
| Title is a topic, not a position | "No Deep Relative Imports" or "Use Aliases for Cross-Package Imports" — not "Imports". |
| WHY omitted | Add it. WHY is what lets Claude judge edge cases. |
| Invented exceptions to "sound thorough" | Delete the section. Exceptions you can't name aren't exceptions. |
| Speculated alias path, lint rule name, file path | Verify with grep, or write the generic form. |
| Two unrelated topics in one rule | Split, cross-reference. |
| Vague voice ("we generally prefer") | Direct imperative. |
| Restates a linter rule | Delete the rule; let the linter enforce. |
| Trigger phrase too abstract ("CSS encapsulation") | Describe the *moment* the rule applies, in the author's words. |
| Pseudo-code examples | Write real code that would compile in the codebase. |
| One-time situation encoded as a rule | Leave it in conversation; don't encode session state. |

## Red Flags — Stop and Reconsider

- "I'll add the exceptions later" → if you don't know them now, don't promise them.
- "Reviewers will figure it out" → if it's that fuzzy, it's not encodeable yet.
- "Claude would do this anyway" → then don't write the rule.
- "This is just a quick note" → quick notes are session state, not rules.
- You're writing example code without checking that the imports / types / calls exist → stop, verify, then write.
- The user's framing and the codebase disagree, and you're encoding the framing → push back, restate the rule's true position, *then* write.
