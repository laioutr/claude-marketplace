# UX laws — composition heuristics of the `ux-designer`

> Source: [Laws of UX](https://lawsofux.com/) (Jon Yablonski). Translated into **composition decisions** for Laioutr pages: which sections/blocks, which order, which slot assignment, how much data. Per job the `ux-designer` **deliberately picks the relevant laws** (not all at once) and implements them with existing sections/blocks.

## Application (mandatory protocol, short)

1. **Goal first.** Conversion? Fewer bounces? Better navigation? Trust? → determines which laws apply.
2. **Pick 3–5 laws** that fit the goal + the page type. Don't cite the whole set.
3. **Map them onto sections/blocks.** Every chosen law must lead to a concrete composition decision (section choice, order, number of items, slot, spacing, CTA placement).
4. **Name them in the rationale** — max. 1 law tag in brackets per decision, e.g. "CTA top + bottom (Serial Position)". No law essays.

## Cheat sheet: goal → laws

| Goal | Reach for |
|---|---|
| **Conversion** | Goal-Gradient, Von Restorff, Serial Position, Hick's, Fitts's, Peak-End |
| **Fewer bounces** | Jakob's, Aesthetic-Usability, Cognitive Load, Doherty, Flow |
| **Better navigation / orientation** | Jakob's, Mental Model, Proximity, Common Region, Chunking, Miller's |
| **Trust / credibility** | Aesthetic-Usability, Social Proof (Peak-End), Mental Model |
| **Readability / comprehension** | Chunking, Miller's, Prägnanz, Occam's Razor, Similarity |

---

## 1 · Lower decision cost (cognitive load)

- **Hick's Law** — decision time grows with the number/complexity of options. → **One** primary action per section; limit nav items (cluster in `SectionHeaderShop`/`BlockMegaMenu` instead of 20 top-level links); avoid competing CTAs on landing pages.
- **Choice Overload** — too many equal options paralyse. → `SectionPlanCardSlider`/`SectionPlanComparisonTable` at 3–4 plans, mark one as "recommended"; curated product sliders instead of "everything".
- **Miller's Law / working memory** — ~7±2 units at once. → USP lists, nav groups, `BlockBenefitsBox` items in bundles of 3–5, not 12 in a row.
- **Chunking** — break information into meaningful groups. → Spread long content across several sections (`SectionMediaText`, `SectionContentGrid`) instead of one wall of text; footer menus in topic columns.
- **Cognitive Load** — keep mental load minimal. → Only show fields/content that serve the current step; keep forms (newsletter, review) to the essentials.
- **Occam's Razor / Law of Prägnanz** — the simplest form wins. → When in doubt, drop the section; simple, regular layouts before nested special cases.
- **Tesler's Law** — residual complexity is unavoidable; someone carries it. → Shift complexity into the system/defaults (query defaults, schema defaults), don't load it onto the user.
- **Paradox of the Active User / Selective Attention** — users don't read instructions, scan with a goal, ignore anything "banner-like". → Value visible immediately (hero), action self-explanatory; don't style important CTAs like ad banners.

## 2 · Direct attention

- **Von Restorff (isolation)** — the odd one out gets remembered/clicked. → **Exactly one** visually emphasised primary CTA per view; the "recommended" plan stands out. Emphasising everything = emphasising nothing. **Avoid CTA inflation:** the same CTA in every section dilutes the signal — only **2–3 CTAs** at decision points, narratively earned (story/proof first, then ask). See `corporate-design.md`.
- **Serial Position** — beginning & end are remembered best. → Most important message in the hero (top) + conversion section (bottom, e.g. `SectionNewsletterRegistration`/CTA banner). Weaker content in the middle. Do **not** plaster every in-between section with a CTA.
- **Aesthetic-Usability** — beautiful design feels more usable & trustworthy. → Spacing rhythm, background changes, on-brand images are conversion factors, not nice-to-haves (see `design-principles.md` #7).

## 3 · Visual structure (Gestalt)

- **Law of Proximity** — things close together read as related. → Label+field, image+caption, CTA close to its value proposition; intra-section spacing < section-to-section spacing.
- **Law of Common Region** — a shared frame groups. → Card blocks (`BlockPlanCard`, `BlockProductTileBasic`, `BlockBenefitsBox`) bundle what belongs together; `SectionContainer`/background as group boundary.
- **Law of Similarity** — similar things are read as a group. → Same role = same block pattern (all product cards alike); style CTAs consistently so they stay recognisably clickable.
- **Law of Uniform Connectedness** — visually connected things feel connected. → Slider/grid sections for sets of connected items; dividers/background changes only at real topic breaks.

## 4 · Motivation & conversion

- **Goal-Gradient Effect** — proximity to the goal increases drive. → Make progress visible (`BlockFreeDeliveryProgress`, checkout steps); short funnels with a clear end; show remaining steps.
- **Zeigarnik Effect** — unfinished things stay present. → Keep started actions visible (cart badge, "profile X % complete"), gently lead to completion.
- **Peak-End Rule** — an experience is judged by its peak + end. → Create one strong moment (hero/showcase) + make the ending positive (confirmation, `SectionReview`, thank-you state). Social proof close to the decision.
- **Pareto Principle (80/20)** — a little content carries most of the value. → Above the fold show the 20 % that drive 80 % of conversions; secondary content further down / in the footer.
- **Flow** — uninterrupted focus. → No unnecessary pop-ups/interruptions in the funnel (`BlockPopUp*` sparingly, triggered, not on landing); consistent paths without dead ends.

## 5 · Expectation & familiarity

- **Jakob's Law** — users expect your site to work like others. → Keep conventions: logo left → home, cart top right, footer with legal/contact, search/filters where expected (`BlockFilterBar`/`BlockSortModes`). Don't break a convention without reason.
- **Mental Model** — users bring a prior model. → Page-type recipes (`memory/page-types/`) match the expected structure (PDP has gallery+price+add-to-cart; PLP has filters+grid). Terms/IA in the users' language.
- **Postel's Law** — be liberal in what you accept, robust in what you output. → Generous input handling (search/form tolerance), always provide empty/error states (`BlockEmptyStateCart`, `SectionPageNotFound`).

## 6 · Speed & reachability

- **Doherty Threshold** — interaction < 400 ms keeps people productive. → Perceived performance: light above-the-fold sections, loading/skeleton states, moderate images; no heavy hero that kills LCP.
- **Fitts's Law** — time to target ~ distance & size. → CTAs large enough + touch ≥ 44 px (see AA gate); primary action easy to reach (on mobile in the thumb zone), not hidden in tiny links.
- **Parkinson's Law** — a task expands to fill the available time. → Tighten funnels, clear defaults, optional steps skippable — build the fastest path.

## 7 · Meta

- **Cognitive Bias** — systematic distortions of perception. → Use deliberately & honestly (real social proof, honest scarcity). No dark patterns — they break trust (Aesthetic-Usability, brand).

---

## Anti-patterns (short)

| Don't | Law against it |
|---|---|
| Several equally strong CTAs in one view | Von Restorff, Hick's |
| 10+ plans/options unfiltered | Choice Overload, Miller's |
| The most important content only mid-page | Serial Position |
| Pop-up immediately on landing | Flow, Selective Attention |
| Idiosyncratic nav/icons "because it's prettier" | Jakob's, Mental Model |
| Wall of text in one section | Chunking, Cognitive Load |
| Mini CTA / touch < 44 px | Fitts's, AA gate |
| No empty/error state | Postel's |
