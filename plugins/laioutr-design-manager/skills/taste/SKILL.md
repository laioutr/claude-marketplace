---
name: taste
description: "Design taste gate for Laioutr projects — corrects the statistical LLM biases towards generic AI-slop UI. Enforces a reasoned taste direction via three dials (design variance, motion, visual density), anti-AI tells (no AI purple, no centered hero, no row of three identical cards, no generic names/fake numbers/filler words), type/color discipline and complete interaction states (loading/empty/error) — all mapped to the Laioutr stack (Vue 3/Nuxt, @laioutr-core/ui-kit, defineSection/defineBlock, Figma variables ONLY, WCAG 2.1 AA). MUST be loaded by both ui-designer AND ux-designer as soon as new components or layouts/pages are created. Triggers: 'new component', 'new layout', 'build a hero', 'landing page', 'compose a page', 'looks generic', 'anti-slop', 'looks AI-made', 'more taste', 'premium look'. (DE: 'neue Component', 'neues Layout', 'Seite komponieren', 'sieht generisch aus', 'wirkt nach AI', 'mehr Geschmack')"
---

# taste — taste gate against AI slop (Laioutr adaptation)

Adapted from the `design-taste-frontend` skill in Leonxlnx/taste-skill (MIT, see THIRD_PARTY_LICENSE). The original skill is specific to React/Next/Tailwind/Framer Motion — **this content is fully rewritten for the Laioutr stack**: Vue 3 / Nuxt, `@laioutr-core/ui-kit` / `ui` / `ui-tokens`, `defineSection`/`defineBlock`, **Figma variables only (no styles)**, WCAG 2.1 AA. The source's React code is **not** used — only its taste and anti-slop **principles**.

## Purpose

LLMs have a statistical tendency towards generic UI: AI purple/neon glows, centered hero sections, three identical cards side by side, "John Doe" + "Acme" + "99.99%", filler words like "Elevate"/"Seamless". That is **AI slop**. This skill is the **gate** that actively steers the ui-designer and ux-designer away from these defaults — complementary to `design-principles.md` (quality bar), `corporate-design.md` (hard UI defaults), `ux-laws.md` (composition) and `content-ux.md` (copy).

**Who & when:** Both agents load this skill **as soon as new components, layouts or pages are created** — the ui-designer when building in Figma, the ux-designer when composing on the platform. The skill **does not replace** any existing rule — it sharpens the taste decision.

## 1. Taste dials (baseline)

Three dials control the taste direction. **Default baseline for Laioutr** (functional-minimalist, Stripe/Linear school):

| Dial | Default | 1 | 10 |
|---|---|---|---|
| `DESIGN_VARIANCE` | **5** | perfect symmetry | artsy/asymmetric |
| `MOTION_INTENSITY` | **3** | static | cinematic |
| `VISUAL_DENSITY` | **4** | art gallery/airy | cockpit/data-dense |

> The Laioutr default is deliberately calmer than the generic taste baseline (8/6/4) — commerce storefronts and B2B interfaces are not a Dribbble showcase. **Adapt the dials dynamically** if the user asks for something else (e.g. "more playful", "more motion", "dense dashboard"). Don't ask — pick a default and name it in the rationale.

## 2. Stack conventions (mandatory — override the React source)

- **Compose, don't import.** There is **no** `npm install` and **no** third-party libs (Tailwind, framer-motion, shadcn, Phosphor) in this context. **REUSE before NEW**: every region maps to an existing component from `@laioutr-core/ui-kit` / `ui`. If one is missing → the ui-designer builds it (with rationale), the ux-designer flags it + hands off. **Never** invent components/sections/blocks that don't exist.
- **Variables only, no styles.** Color, type, spacing, radius, shadow, z-index → exclusively **Figma variables** of the design-system library, semantically aliased to primitives (`memory/figma/variable-workflow.md`). No hardcoded hex/pixels, no color/text/effect styles.
- **Buildable.** Every region is a `defineSection`/`defineBlock` candidate + Vue component. "Looks nice but not buildable" = not done.
- **Theming.** `theme × color scheme (default/pale/solid/muted) × light/dark` (`memory/design-system/color-scheme.md`). Taste must not bypass the theming model.
- **WCAG 2.1 AA is a hard gate** — every taste decision must survive contrast (4.5:1 / 3:1), focus visibility, keyboard, touch target ≥ 44px.
- **Icons** come from the design system's icon set (design-system-bound) — not from React icon packages. **No emojis** in UI, markup, alt text or copy.

## 3. Design-engineering directives (bias correction)

**Rule 1 — Typography with intent.**
Hierarchy through weight + color + spacing, not only size. **No oversized H1 that shouts.** All type values from the design system's type-scale variables (`text/display`, `text/heading`, `text/body` …), never typed in. Keep body copy readable (line length ~60–75 characters via the design-system measure). Font choice follows the design system — **do not** introduce arbitrary "premium fonts"; if the type scale is `_TBD_`, flag it instead of inventing one.

**Rule 2 — Color calibration.**
- **One accent.** At most one accent color per view, saturation < 80%. Neutral base (no mixing warm and cool greys on the same page).
- **AI purple ban.** The "AI purple/blue" aesthetic (purple button glows, neon gradients) is **forbidden**. The accent comes from the design system's color-scheme logic, not from an invented gradient.
- **Consistent palette** across the whole page — one color-scheme family, not reinvented per section.

**Rule 3 — Layout diversification (anti-center bias).**
A centered hero/H1 is **forbidden** at `DESIGN_VARIANCE > 4` (matches `corporate-design.md`: no centered body text). Instead: split screen (50/50), text left / asset right, or asymmetric whitespace. **Mobile override:** asymmetric layouts collapse cleanly to one column below the mobile breakpoint (no horizontal scroll).

**Rule 4 — Materiality & anti-card-overuse.**
Cards **only** when elevation communicates real hierarchy. At high density, group via `border-t`/dividers/negative space instead of locking every datum in a box (matches `corporate-design.md`: compact cards, card background set apart). Shadow, if any, tinted in the background hue — **no** neon/outer glows.

**Rule 5 — Interaction states (mandatory).**
LLMs only deliver the "happy static state". You deliver the full cycle:
- **Loading:** a skeleton that mirrors the layout dimensions — no generic spinner.
- **Empty:** a well-composed empty state that shows how to fill in data.
- **Error:** a clear error message placed inline (e.g. below the input).
- **Tactile feedback:** a subtle press effect on `:active` (`scale`/`translate` via a design-system motion variable), never linear/abrupt.
For the `ux-designer`: check whether the chosen section/block supports these states (empty `query` result, loading/error behaviour) — otherwise flag it.

**Rule 6 — Form patterns.**
Label **above** the input. Helper text optional, but provided for in the markup. Error text **below** the input. Consistent gap block (design-system spacing variable).

## 4. Motion (only when `MOTION_INTENSITY > 3`)

- Animate **only** cheap properties (transform/opacity equivalents) — never layout dimensions. Performance before effect.
- **Spring/soft easing** via design-system motion variables, no hard linear.
- **Keep it subtle.** Perpetual micro-motion (pulse/float/shimmer) only when it carries a function (status is "alive", loading hint) — not as decoration. At the Laioutr default (3), motion stays limited to `:hover`/`:focus`/`:active`.
- No scroll-driven spectacles on B2B conversion pages, unless the user explicitly asks for it.

## 5. AI tells (forbidden patterns)

Avoid these slop signatures unless explicitly requested:

**Visual**
- ❌ Neon/outer glows · ❌ pure `#000000` (off-black/zinc-950 via a design-system variable) · ❌ oversaturated accents · ❌ gradient text on large headlines · ❌ custom mouse cursors.

**Typography**
- ❌ Oversized H1 · ❌ hierarchy through size only · ❌ introducing an arbitrary "premium font" instead of the design-system type scale.

**Layout & spacing**
- ❌ **Row of 3 identical cards** as a feature row (matches `corporate-design.md`) → 2-column zigzag, asymmetric grid or horizontal scroll · ❌ floating elements with uneven gaps — spacing from a consistent design-system scale.

**Content & data (the "Jane Doe" effect)**
- ❌ Generic names ("John Doe", "Sarah Chan") → realistic, context-appropriate names · ❌ egg/default avatars → believable placeholders · ❌ fake numbers (`99.99%`, `50%`, `1234567`) → organic values (`47.2%`) · ❌ startup-slop brands ("Acme", "Nexus", "SmartFlow") → premium, contextual · ❌ filler words ("Elevate", "Seamless", "Unleash", "Next-Gen") → concrete verbs (matches `content-ux.md`).

**Resources**
- ❌ Unsplash hotlinks → image suggestions according to `imagery.md`, media `fill` instead of `fit` · ❌ leaving components in their generic default — set radius/color/shadow deliberately via the design system.

## 6. Dial definitions

**`DESIGN_VARIANCE`** — 1–3 symmetric 12-column grid, equal paddings · 4–7 offset alignment, mixed aspect ratios, left-aligned headers over centered data · 8–10 masonry/bento, fractional grid columns (`2fr 1fr 1fr`), large empty zones. **Mobile override:** 4–10 strictly collapse to one column below the mobile breakpoint.

**`MOTION_INTENSITY`** — 1–3 only `:hover`/`:focus`/`:active` · 4–7 soft transitions + staggered load-ins via transform/opacity · 8–10 scroll-driven reveals/parallax (only on explicit request).

**`VISUAL_DENSITY`** — 1–3 lots of whitespace, large section gaps · 4–7 normal app density · 8–10 cockpit: small paddings, dividers instead of boxes, mono for numbers.

## 7. Pre-flight check (last gate before output)

- [ ] Dial values chosen deliberately + named in the rationale?
- [ ] No AI purple / neon glow / pure black — accent from the design-system color scheme?
- [ ] Hero/H1 **not** centered (at variance > 4)? No centered body text?
- [ ] **No** row of 3 identical cards?
- [ ] Mobile collapse to one column guaranteed?
- [ ] Loading, empty, error state provided for (or flagged if the section/block doesn't support them)?
- [ ] Cards only where elevation carries hierarchy — otherwise whitespace/dividers?
- [ ] Realistic names/data/brands, no filler words, no emojis?
- [ ] Everything bound to design-system variables, REUSE before NEW, buildable?
- [ ] WCAG 2.1 AA holds (contrast, focus, keyboard, touch target)?

On ⚠️ in any point: fix it **before** the design/page goes out. For the deep QA run afterwards: `design-critique`.

## Scope boundaries

- **Quality bar / heuristics** → `memory/principles/design-principles.md`
- **Hard UI defaults (buttons s/m, header + footer, no centered text)** → `memory/design-system/corporate-design.md` (project copy)
- **Composition (Laws of UX, section order)** → `memory/principles/ux-laws.md`
- **Copy / claim resolution** → `memory/principles/content-ux.md`
- **Pre-handoff QA (accessibility + design system + buildability)** → `design-critique` skill

This skill delivers the **taste direction + anti-slop correction** — it is input before building/composing, not a replacement for the QA gate afterwards.
