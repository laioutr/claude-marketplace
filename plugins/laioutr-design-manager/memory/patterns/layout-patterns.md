# Layout patterns — canonical solutions

> Reusable layout solutions per type. Saves the `ui-designer` from reinventing and keeps designs consistent + buildable.

**Status:** _TBD_ — patterns are filled in once the design system and first designs exist.

---

| Pattern | When | Section/block split | UI-kit components | Note |
|---|---|---|---|---|
| Hero | Landing page entry | `defineSection` heroX | _TBD_ | Heading + subline + CTA + media |
| Feature grid | Feature overview | `defineSection` + `defineBlock` (cards) | _TBD_ | responsive grid, equal card height |
| PDP (product detail) | E-commerce detail page | several sections | _TBD_ | Gallery, info, CTA, cross-sell |
| Listing / collection | Product/content list | Section + block card | _TBD_ | Filters, pagination, empty state |
| Pricing | Plan comparison | `defineSection` pricing | _TBD_ | Tier cards, highlighted tier |
| Forms | Lead/checkout/input | Section + field blocks | _TBD_ | Label convention, error states, AA focus |

---

## Pattern rules

- Every pattern has an **empty / error / loading state** (not just the happy path).
- Every pattern is documented as **responsive** (mobile + desktop) and **AA-compliant**.
- Add new patterns here as soon as they repeat (used ≥ 2×).

---

## Spacing rule (applies to every section, every pattern)

- Choose margin + padding **deliberately** per section — never leave everything on default/none. Goal: a calm vertical rhythm, sections that breathe.
- **Consistent spacing scale** across the whole page: same role → same value (no randomly mixed s/m/l).
- Larger gaps at thematic breaks, tighter ones within related blocks → grouping through whitespace.
- **Alternate the background rhythm** deliberately (none/pale/solid) so sections stand apart instead of merging into one block.
- Always check the page **as a whole**: visually appealing and consistent with the brand / overall presence — including how header and footer fit.

---

## Studio composition — toolbox (NOT a default template)

> **Meta-principle (most important lesson):** there is **no standard layout**. **Always** derive the composition from **content + goal** and rethink it for each case. The techniques below are **options** to pick from situationally — not a recipe to apply everywhere. Laying the same pattern over everything ("always a USP banner", "always band + card") makes for bad design.
>
> **Grouping is mandatory:** things that belong together thematically sit **next to each other** and are **visibly grouped as one zone** (through whitespace, a shared background or bracketing).

### `SectionContainer` — the two background layers (use deliberately)
- `background` = colour of the **content box** (with `boxed`, the centred box; with `full-width`, the full width).
- `sectionBackground` = **full-width band** behind the boxed content box (**only with `boxed`**).
- Together they create **contrast + bracketing**: e.g. a near-white content box (`background`) inside a tinted full-width band (`sectionBackground`) → the content looks framed / "floating". A tool, not a must.

### Technique A — "band + card" bracketing (example solution, situational)
Fits **parallel info blocks** (feature / eligibility / benefit cards) on a marketing landing page:
- The heading section **and** the card-grid section share the **same** `sectionBackground` (brand tint) → one continuous zone, **no seam** between heading and cards.
- Card grid: `boxed`, `background` = near-white (the lightest step of the brand ramp, e.g. `<brand>-1`) → the card box stands out from the tint band (figure-ground without a border).
- `padding: s` on the card box (the band provides the outer whitespace), card grid often **3 columns**.
- **Not** a default — other content (storytelling, comparison, process, editorial intro) calls for entirely different splits (media & text / zigzag, split screen, table, timeline …).

### Technique B — two-column text (split container, no media)
An alternative to `SectionMediaText` when there is **no media** but the text needs structure:
- `SectionContainer` with `columnsDesktop: 2` (`columnsMobile: 1`), slot `default` with **two `BlockText`**.
- **Left:** heading + body copy (context/explanation). **Right:** bullet points / key facts (compact list).
- Scannable rhythm (narrative left, facts right), tidier than one long single-column block, F-pattern friendly.
- Control column widths via `sizing` (`fixed` = equal weight, `greedy-first`/`greedy-last` = one column dominant).
- Situational — **not** a replacement for media & text where an image/video carries the message. Use it when the text would otherwise read as a wall of text.

### Technique C — solid content box in a tint band (panel with its own weight)
Deliberately lifts **one** block out as a standalone panel without breaking it out of the page flow:
- `SectionContainer` `boxed`: `sectionBackground` = **the same tint as the preceding section** (the band visually continues, no seam) + `background` (content box) = **`solid`** (filled brand surface).
- Result: the content box gets **its own weight** / reads as a highlighted panel inside the continuous band — figure-ground through fill instead of a border.
- Good for a **focal block** (API / integration / secondary-CTA story) that should stand out but stay part of the zone. An empty tinted `SectionContainer` right after it as a **spacing / connecting band** keeps the rhythm.
- **A11y must:** `solid` = dark/strong brand fill → text + CTA must be **light/adaptive** (adaptive variants) and hold AA contrast in **both** modes.
- Situational, not a default — too many solid panels cancel each other out.

### Other tools (combine freely)
- **The comparison/pricing table (`SectionPlanComparisonTable`) has NO heading/subline field of its own** → always put a **heading section before it** (`SectionContainer` + `BlockText` with caption/heading/subline). UX copy should set expectations + reduce anxiety (e.g. "All plans include … — no hidden costs").
- **Empty coloured `SectionContainer` as a spacer / connecting band** between two zones — controls rhythm + separation without an extra heading.
- **Break the pattern deliberately:** leave one zone on `default`/white so a saturated `solid` CTA section lands as the climax (tension through contrast, not uniformity).
- **Cards / icon chips — check contrast against the surface first:** **always choose `iconBackground` against the surface the card sits on**, not by default. On a **white/neutral** card surface, `pale` looks calm and is enough. But if the card sits on a **tinted band** (`sectionBackground: pale`/tint), `pale` = **pale-on-pale** → weak, muddy contrast → use `solid` instead (filled brand chip, light icon). Rule: **chip tone ≠ surface tone.** **Omit** the icon when a number (1/2/3) is already the anchor — no redundant decoration.
- **Heading hierarchy:** the hero stays the largest level; section headings smaller (control `BlockText.size`, H2, no level skips H1→H3).

### Colour discipline (hard rule)
- For tints use the brand ramp (e.g. `<brand>-1…12`), not the semantic `pale` scheme next to the brand colour — they can clash.
- One accent family per page, applied consistently.

### Dark-mode caveat for custom two-tone colours
- Set **real** values per mode — not the same light hex for `light` **and** `dark`. Otherwise a near-white content box (`<brand>-1`) glares in dark mode.
- Check text **directly on a tint band** against **both** modes (a light brand tint needs near-black text; a dark step like `<brand>-12` needs near-white).
