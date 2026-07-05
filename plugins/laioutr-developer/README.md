# Laioutr Developer Plugin

A Claude Code plugin for developers building on the **Laioutr** e-commerce
platform — Nuxt 3 / Vue 3 storefronts, custom UI components, and Orchestr
data integrations.

With this plugin installed, when you ask Claude to "build a hero section
from this Figma frame" or "wire up an Orchestr query handler for product
reviews", Claude will:

- Search the official Laioutr docs through an MCP server before guessing
  at prop names, package surfaces, or schema fields.
- Apply Laioutr conventions automatically — the `defineSection` schema
  shape, the `@laioutr-core/ui-kit` vs `ui` vs `ui-app` layering,
  the `--z-index-*` token discipline.
- Follow a structured Figma-to-component workflow that decomposes a
  design into a component hierarchy and an API spec before writing any
  Vue code.
- Pull in role-specific guardrails on demand (atom-style primitive,
  organism, section/block) instead of one bloated context dump per turn.

## Install

This plugin ships through the public Laioutr marketplace at
[`laioutr/claude-marketplace`](https://github.com/laioutr/claude-marketplace).

```bash
# Add the marketplace (once per machine)
claude /plugin marketplace add laioutr/claude-marketplace

# Install the plugin
claude /plugin install laioutr-developer@laioutr
```

After installing, run `/reload-plugins` (or restart Claude Code) so the
skills, rules, and MCP server pick up.

### Try it out

Drop a Figma URL or describe a Laioutr task in plain language — no need
to invoke skills explicitly. A few starting prompts:

- *"Implement the hero in this Figma frame: <figma-url>"* → triggers
  `figma-design-analysis` → `figma-component-architecture` →
  `figma-to-component`.
- *"Add a featured-products section with a configurable product limit."*
  → triggers `writing-section-block-definitions`.
- *"How do I render currency in this block?"* → triggers
  `laioutr-platform`, which then opens `money-currency-code.md`
  on demand.

To force a specific skill regardless of phrasing, use
`/laioutr-developer:<skill-name>` — e.g.
`/laioutr-developer:figma-to-component`.

### Other install paths

```bash
# From a local checkout, for development
claude /plugin install /path/to/claude-marketplace/plugins/laioutr-developer
```

## Requirements

- Claude Code 2.x with plugin support (`/plugin` command available).
- Node 20+ and pnpm or npm for the storefronts you're working on.
- Outbound HTTPS to `https://docs.laioutr.com` so the docs MCP server can
  connect.

You do not need a Laioutr account to install the plugin, but most of the
linked documentation expects a developer account on
<https://docs.laioutr.com>.

## What you'll see Claude doing differently

**Before:** "Add a price field to this block."
Claude writes `{ name: 'price', type: 'number' }` and a `formatCurrency`
helper that hard-codes `€`.

**After:** Claude reads
`skills/laioutr-platform/references/money-currency-code.md`, picks the
canonical `Money` type from `@laioutr-core/canonical-types`, and uses the
storefront's locale-aware formatter — no homegrown currency math.

---

**Before:** "Implement this hero from Figma."
Claude eyeballs the frame and writes one 400-line `.vue` file with
inline CSS and ad-hoc breakpoint logic.

**After:** Claude runs the `figma-design-analysis` skill to decompose
the frame, drafts a component-API spec with `figma-component-architecture`,
checks `@laioutr-core/ui-kit` for upstream primitives before writing
markup (per `ui-ui-kit-first.md`), and uses CSS-first responsive
patterns rather than `useBreakpoints` (per
`ui-kit-css-first-responsive.md`).

---

**Before:** "What's the resolution order if `UiKitButton` is missing a
variant I need?"
Claude guesses or suggests forking from npm.

**After:** Claude opens
`skills/laioutr-platform/references/three-layer-architecture.md` and
walks the ladder: prop override → local override component in your
module → fork from `laioutr/ui-source` → upstream issue. Forking is
explicit and documented, not a last resort whispered between developers.

## What this plugin ships

- **`skills/laioutr-platform/`** — the platform skill. Loads when Claude
  detects Laioutr work in your prompt (mentions of Laioutr,
  `@laioutr-core/*`, `defineSection`, `defineBlock`, Orchestr handlers,
  etc.). Carries the platform overview, the discovery-surface table
  (docs MCP / Storybook / ui-source), the Orchestr handler reference,
  commit conventions, and the trigger index for the cross-cutting
  conventions in `references/`.
- **`skills/laioutr-platform/references/`** — 24 read-on-demand
  convention files (includes `three-layer-architecture.md`). The
  filename prefix encodes the role the convention applies to
  (`ui-kit-*` for atom-style primitives, `ui-*` for organisms, no
  prefix for cross-cutting concerns like Vue SFC mechanics, naming,
  public CSS contracts, TypeScript discipline). The platform skill
  indexes them; Claude opens individual files on demand when their
  trigger matches the task.
- **Task-triggered skills** — auto-load when the prompt matches their
  description. Laioutr-specific:

  | Skill | Triggers on | What it carries |
  | --- | --- | --- |
  | `writing-section-block-definitions` | `defineSection` / `defineBlock`, schema fields, `Block*.vue` naming | Sidebar group order, canonical field names, forbidden-name list, `if` operators, factory literal-type discipline, `defineSelectOptions` rules — plus 6 reference files |
  | `writing-storybook-stories` | `.stories.ts` authoring, Storybook variants | Meta titles, variant naming, no per-viewport exports, theming checks, refactor workflow |
  | `figma-design-analysis` | Decomposing a multi-component Figma frame (PDP, checkout, full layouts) into a component hierarchy | Frame exploration, component matching, token mapping, themable defaults, subagent orchestration — plus 6 reference files |
  | `figma-component-architecture` | Producing component API specs (props / slots / events / state) from a design plan | Spec format, core architectural constraints, output template — plus 3 reference files |
  | `figma-to-component` | Implementing Vue components from a Figma design or analysis plan | Vue component conventions, breakpoint handling, verification pass |
  | `figma-export-assets` | Exporting illustrations, photos, logos, markers, screenshots, custom icons from a Figma frame into the repo | MCP probe → inventory → leaf isolation → format selection → destination → bytes → wire-up |

  General-purpose authoring helpers (bundled for convenience):

  | Skill | Triggers on | What it carries |
  | --- | --- | --- |
  | `writing-claude-rules` | Authoring a rule file in a project's `rules/` or `.claude/rules/` | Position-claim shape, verification before encoding, voice discipline, examples format |
  | `adr` | Capturing an architectural decision | ADR section template, clarification-first workflow |
- **`.mcp.json`** — registers the `laioutr-docs` HTTP MCP server at
  `https://docs.laioutr.com/mcp`. This is how Claude searches the
  official Laioutr developer documentation (Orchestr framework, data
  model, CLI, UI components, sections and blocks, TypeScript API
  reference) during a session.

The full reference index, with one-line triggers, lives at the bottom
of `skills/laioutr-platform/SKILL.md` — that's the file Claude reads
when the platform skill triggers.

## How the rules are loaded

Two surfaces, both triggered by your prompt:

1. **Skills auto-load** when their `description` matches the task.
   `laioutr-platform` triggers on any Laioutr work (broad); the
   task-specific skills (`writing-section-block-definitions`,
   `writing-storybook-stories`, the Figma skills) trigger on narrower
   prompts. The matching skill's `SKILL.md` body loads in full.

   To override automatic matching, force a specific skill with
   `/laioutr-developer:<skill-name>` (e.g.
   `/laioutr-developer:figma-to-component`).
2. **References load on demand.** Each skill's body indexes the
   reference files in its `references/` directory with one-line
   triggers. When a task plausibly matches a reference's trigger, Claude
   opens that file before writing or changing code. For example, the
   `laioutr-platform` index includes entries like:

   ```
   - money-currency-code.md — handling Money objects, prices, or
     currency in code, stories, or fixtures
   - no-wrapper-css-overrides.md — before writing CSS in a parent
     that targets a child component's class names
   - parent-prefix-naming.md — naming a Vue component, especially
     before reaching for <Parent><Part> compound naming
   ```

If Claude misses a reference that obviously applies, name it explicitly
("check `section-config-standard.md` before you change the schema").
The trigger catalogs at the bottom of each `SKILL.md` keep filenames
and triggers paired.

## The docs MCP server

When connected, `laioutr-docs` lets Claude search the official Laioutr
developer documentation in-session. Use it for:

- Props, emits, slots, and usage examples for any upstream
  `@laioutr-core/*` component.
- Orchestr handler signatures and entity resolver patterns.
- Section / block schema field types and constraints.
- CLI commands and the canonical TypeScript types.

Setup details: <https://docs.laioutr.com/getting-started/mcp-server>.

The same content is also indexed by Context7 as a fallback when the
official MCP is unreachable, but prefer the official server when both
are available — it is closer to source of truth.

## Troubleshooting

**The docs MCP server isn't connecting.** Check that outbound HTTPS to
`docs.laioutr.com` is allowed. Run `/mcp` inside Claude Code to see the
connection status for `laioutr-docs`. If the server is failing, Claude
will still work — it will just lean on Context7 (if installed) and its
training data instead of authoritative docs.

**Claude isn't picking up a rule I expected.** Convention references
don't apply automatically per file — they're indexed by a skill and
read on demand. If the relevant skill itself didn't trigger, your
prompt may not have matched its description (try mentioning "Laioutr",
"`defineSection`", "`.stories.ts`", "`@laioutr-core/ui-kit`", etc.). If
the skill did trigger but a specific reference didn't fire, name it
explicitly in your prompt, or check the reference index at the bottom
of the relevant `SKILL.md` — the trigger may be narrower than you
expect.

**A skill isn't activating.** Skills load when Claude detects a matching
task from your message. If `figma-to-component` doesn't fire, mention
"implement this Figma component" or pass a Figma frame link. You can
also force a skill with `/laioutr-developer:<skill-name>` once the
plugin is installed.

## Relationship to the internal monorepo

This plugin is curated from the Laioutr core team's internal
`.claude/` configuration, with monorepo-only paths, private namespaces,
and internal release tooling stripped out. See
[`MAPPING.md`](./MAPPING.md) for the carry-over list, the rewrite
notes, and known internal references that still need cleanup.

## Layout

```
laioutr-developer/
├── .claude-plugin/
│   └── plugin.json                          # Plugin manifest
├── .mcp.json                                # MCP server config (laioutr-docs)
├── LICENSE                                  # MIT
├── MAPPING.md                               # What was carried over from the monorepo, and why
├── README.md
└── skills/
    ├── laioutr-platform/                    # Auto-loads on Laioutr work
    │   ├── SKILL.md                         # Platform overview + reference index
    │   └── references/                      # 24 cross-cutting conventions (includes three-layer-architecture)
    ├── writing-section-block-definitions/   # Auto-loads on defineSection / defineBlock work
    │   ├── SKILL.md                         # Overview + quick reference
    │   └── references/                      # 6 deep-detail files (schema standard, if, factories, naming, …)
    ├── writing-storybook-stories/           # Auto-loads on .stories.ts work
    │   └── SKILL.md                         # Inline (no extra references)
    ├── figma-design-analysis/               # Figma frame → component hierarchy
    │   ├── SKILL.md
    │   └── references/                      # 6 deep-detail files (exploration, token mapping, themable defaults, …)
    ├── figma-component-architecture/        # Figma analysis → component API spec
    │   ├── SKILL.md
    │   └── references/                      # 3 deep-detail files (core constraints, spec format, output template)
    ├── figma-to-component/                  # Implement Vue from Figma
    │   └── SKILL.md                         # Inline (no extra references)
    ├── figma-export-assets/                 # Export illustrations / photos / logos / icons from Figma into the repo
    │   └── SKILL.md                         # Inline (no extra references)
    ├── writing-claude-rules/                # Author a new rule file (general-purpose)
    │   └── SKILL.md
    └── adr/                                 # Capture an ADR (general-purpose)
        └── SKILL.md
```

## License

MIT — see [`LICENSE`](./LICENSE).
