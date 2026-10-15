# Corporate design — hard UI defaults for composed pages

> Template. Project copy lives at `.claude/design-manager/design-system/corporate-design.md`; fill it for your project.

> Binding constraints for every page the `ux-designer` (and `ui-designer`) builds. The generic rules below apply **automatically**, without the user naming them. "Looks professional" is no accident — it comes from consistently set design options, not from defaults. If the project has its own brand/voice guide or plugin, use it in addition.

## Project brand (fill in)

| Topic | Project value |
|---|---|
| **Brand colors** (as design-system variables, never hex) | _TBD_ |
| **Brand theme / color schemes in use** | _TBD_ |
| **Voice & tone** (formal/informal address, register, words to use/avoid) | _TBD_ |
| **Imagery style** (details in `imagery.md`) | _TBD_ |
| **Do** | _TBD_ |
| **Don't** | _TBD_ |
| **Reference site(s)** for the look | _TBD_ |

---

## Mandatory: actively use design options (do not leave defaults)

Every section/block has design props (`background`, `padding`/`margin`, `alignment`, `variant`, `layout`, columns/count, …). **These are the tool for good looks.** A page where everything sits on default/none is not finished.

- **Background rhythm:** alternate backgrounds deliberately (e.g. `none` → `pale`/`muted` → `none`) so sections stand apart instead of merging into one grey block.
- **Spacing scale:** consistent margin/padding values for the same roles across the whole page. Sections breathe (see `../principles/design-principles.md`).
- **Variant/layout per section** chosen for its purpose (e.g. card variant, column count), not the first default.

## Buttons — size & weight IN CONTEXT (not mechanical)

A button's size/style depends on its **context**, not on a fixed rule. Principle: **the more buttons close together, the lighter and smaller they are.**

- **Never `l`/`xl`.** Large buttons look aggressive and cheap.
- **Lone primary CTA** (hero, closing banner): `primary`, size `m` — it may be the anchor. In heroes lean towards smaller (`s`) rather than larger.
- **Buttons in groups / near other buttons, cards, repetitions** (e.g. one CTA per card across a grid): **smaller + lighter** — `ghost`/`link`, size `s` (or `xs`, if spacing still keeps the tap target ≥ 44 px). **No row of heavy, filled `m` buttons** — that overwhelms the layout (many filled secondary `m` buttons side by side draw far too much attention through size + weight; ghost buttons in `s`/`xs` are right).
- **One visible weight hierarchy per view:** one anchor (primary), everything else clearly lighter. Secondary/repeated actions never compete in weight with the anchor (Von Restorff).
- Touch target ≥ 44 px (AA) remains mandatory — for `xs`, secure it via enough padding/spacing.

## Text & alignment (WCAG default, automatic)

- **No centered body text.** The default is **left-aligned** (reading direction, better legibility/AA). Center at most a short single-line headline/eyebrow — never paragraphs, never lists, never whole hero/CTA blocks by default.
- Consistent hierarchy through size/weight/spacing, not through centering.

## WCAG 2.1 AA — baseline the `ux-designer` applies without asking

- Contrast 4.5:1 (text) / 3:1 (large text, UI). For text on images: choose an overlay/darkened background.
- Visible focus, full keyboard operability, meaningful semantics/order.
- Every `media` field needs alt text. Touch ≥ 44 px.
- Details: `memory/patterns/accessibility-baseline.md`.

## Header & footer (mandatory on every page)

- **Every page references the global header and footer sections.** Without them the page is incomplete (navigation, login, footer menus, legal).
- Studio: `studio_list_global_sections` → attach the existing global header/footer section via `studio_add_global_section_reference` to the page's `header`/`footer` slot.
- If the project has no global header/footer: **flag the gap** (creating the container does not run through MCP → Studio / `ui-designer`), do not just leave it out.

## CTA discipline (against CTA inflation)

- **One** primary conversion goal per page — but do **not** put a primary button into every section. That dilutes (Von Restorff) and feels pushy.
- The primary CTA is **earned by the story:** first explain, then ask. Place it at **decision points**: hero, after established value/proof, closing banner. Usually **2–3 primary CTAs in total**.
- **Secondary info paths are welcome** (not forbidden): `secondary`/`ghost`/`link`/card links leading to **subpages with more information** (product details, categories, use cases, references). They help visitors inform themselves, do not compete visually with the primary CTA, and need good microcopy (name the destination). Details: `../principles/content-ux.md`.

## Story before conversion (content completeness)

- A page must **first explain what the brand/product is** (and what makes it different), before it pushes for the action.
- Do not forget the brand's core themes: a typical arc is headline promise → problem → solution/offer → how it works → breadth (range, categories, use cases) → proof (cases/testimonials) → closing CTA. Fill in the project's own core themes in "Project brand" above.

## Visual consistency & tidy layout (mandatory)

A page must look **consistent and tidy** as a whole — otherwise it looks unprofessional.

- **Cards compact + consistent.** Same card size/variant for the same role. **Short headlines** (1–3 words, no sentence headlines), **left-aligned** card text (never centered), concise body. No huge cards with long headlines.
- **Consistent scale** across the page: same spacing, card, column and variant values for the same roles. Do not mix randomly.
- **Keep alignment consistent** (left-aligned throughout), clean grid.
- **Figure-ground / contrast to the page background:** cards and sections must **stand out** from what is behind them, otherwise they look "lost"/floating. Cards on a light section background need a **distinct card background** (or border/shadow); a section that is not separated in color from its neighbour blurs into it (e.g. a slider whose cards have no distinct background makes the whole section look lost — set the card background apart).
- **Clean backgrounds — no random stripes.** Think background + padding together: a banner section must read as **one clear block**, not as thin solid stripes at top/bottom. Check how your section background interacts with adjacent sections (full bleed where intended, no accidental band). Stripes like that are a composition decision, not a component bug.

## Reduce text density (especially for technical topics)

- **Not text-heavy.** Long body copy is tiring — shorten, break it into bullets/USP items/cards/captions, use whitespace and visuals. One clear idea per section, not three paragraphs.
- Visual relief: icons, images, short labels instead of walls of text (see `../principles/content-ux.md`, `imagery.md`).

## Media defaults

- Background/hero media set to **`fill`/cover**, not `fit` — set the option correctly even when the image URL is still missing.
- Empty `media` fields: make a **short image suggestion** (see `imagery.md`), do not just flag them.

## Judge the overall impression (senior duty, not rule ticking)

Following rules is not enough — **step back and judge the page as a whole**, like a senior with taste looking at the finished picture:

- **Balance & weight:** Is the visual load distributed? Does something (a button row, a card block) overwhelm the rest? Is there exactly **one** clear anchor per view?
- **Figure-ground:** Does every section/card stand out cleanly, or does something get "lost" in the background?
- **Rhythm & tidiness:** Do spacing/backgrounds form a calm alternation — or do stripes, breaks, restlessness appear?
- **Consistency:** Do equal roles look equal (cards, buttons, alignment)?
- Where the impression is not "tidy + professional": **improve deliberately** (size, weight, background, spacing), do not wave it through. Make conscious design decisions with a rationale.

## Hard rules from reviews

1. **Locale gotcha (critical, most frequent error):** layout/alignment props (`alignment`, `buttonAlignment`, `margin`, `padding`, `textAlignmentDesktop/Mobile`, `sizing`, `columnsDesktop/Mobile`, …) are `translatable: false` in the schema, but the frontend evaluates them **per rendered locale**. `studio_set_static_prop` **without** `locales` writes to the default locale only; any other rendered locale falls back to the component default (e.g. centered card text). → **Always write layout props for every active locale** (pass `locales` explicitly). Otherwise e.g. `alignment: left` does not take effect and text stays centered.
2. **Sections always need spacing:** set `margin` (m/l), never `none` everywhere, otherwise sections stick together. Padding does NOT replace margin.
3. **No centered body text:** `BlockCard` appearance → align = **left**; `SectionContentGrid` `alignment` + `buttonAlignment` = **left** — in every active locale. The default is center.
4. **Button size in context:** hero buttons NOT `m`/`l` (they look huge), rather `s`. Keep repeated card CTAs light (ghost variant), not filled.
5. **Hero/slide background image ALWAYS `sizing: "fill"`** (not fit/keep-ratio).
6. **Not everything in cards:** deliberately use **`BlockText`** for body copy/intros instead of cards everywhere. For mixed or one-/multi-column passages use the **empty `SectionContainer`** (`columnsDesktop`/`columnsMobile`, `sizing` greedy-first/last) and combine different block types.
7. **Do not forget breadcrumbs:** `SectionBreadcrumbs` as the topmost body section.
8. **Verification:** a live domain may serve a cached publish state and does NOT show Studio drafts immediately. Check drafts in the Studio editor (or after publishing), not on the live domain.

## Short checklist before "done"

Header + footer referenced · buttons in context (anchor `m`, groups lighter ghost `s`/`xs`) · no centered body text · cards compact + short headlines + left-aligned + **distinct card background** · consistent scale/alignment · background rhythm **without stripes** · not text-heavy (visuals/captions) · primary CTA at 2–3 points + secondary info paths · every claim resolved (problem → solution) · media `fill` + image suggestions · brand/product story told before the ask · layout props written for every active locale · AA · **overall impression judged + improved**.
