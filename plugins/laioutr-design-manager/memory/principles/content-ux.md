# Content UX — copy, microcopy, navigation, claim resolution

> Binding for every page the `ux-designer` builds. A page does not convince through more text but through **clear, concise, resolved** content and good paths to more information. Final voice polish follows the project's brand guidelines (see `memory/design-system/corporate-design.md`; if the project has its own brand/voice guide or plugin, use it).

## Copy rules (less is more)

- **Short, clear sentences.** One thought per sentence. Active, concrete, no stacked marketing-speak.
- **Less is more.** Keep every section as short as possible. Cut long rich-text blocks; when in doubt, delete. A headline + 1–2 sentences beats a paragraph.
- **Use captions** where they take load off the body copy (image/card captions, eyebrows/kickers above headlines, short labels). Hierarchy: eyebrow → headline → 1 sentence → optional caption.
- **Scannable, not reading text.** Technical topic = visitors scan. Bullets/cards/USP items instead of walls of prose.

## Button microcopy

- **Describes the goal/benefit,** never "Click here"/"More". Examples: "Book a demo", "See pricing", "Explore the product", "View for B2B".
- 1–3 words, verb-led, concrete. Secondary actions may name their destination so visitors know where the click leads (information scent).

## Primary vs. secondary actions (two levels)

- **Primary = conversion** (e.g. "Book a demo"): disciplined, 2–3× at decision points, one `primary` CTA per view (see `corporate-design.md`).
- **Secondary = information** (`secondary`/`ghost`/`link`/card link): **use deliberately** to lead visitors to **sub-pages with more information** (product detail, individual offers, industries, features, case studies). This is mandatory for good informability — not every visitor is ready to buy, many want to understand first.
- Secondary actions do **not** compete visually with the primary CTA (no second `primary` button), but they clearly lead onward. They are not CTA spam but information paths.

## Claim resolution (narrative integrity)

- **Every claim made is resolved.** If a **problem** is named, the **solution** follows visibly. If a promise is made ("in weeks, not months"), the proof / the path to it follows.
- Before "done": read the page as a chain of argument — problem → solution → proof → action. Close or cut open loops (problem without answer, claim without proof).

## Hero + intro pick-up (AIDA)

- **Hero = attention.** Keep headline + subline **drastically short** — one punchy marketing message, one thought. The hero does not sell the details, it **catches attention** and names ONE clear action (primary CTA). Rule of thumb: headline ≤ ~6 words, subline 1 sentence.
- **No content gets lost:** whatever falls out of the short hero message (explanation, feature list, proof) moves into an **intro section directly below the hero** — a `SectionContainer` + `BlockText`, named **"Intro"** in the layer tree. Never cut content for brevity without a replacement — **relocate** it.
- The intro **picks up the hero's topic + claim** and "pulls the user into the page": it delivers interest + desire before the feature/proof sections follow.
- **AIDA as page dramaturgy:** hero (attention, short) → intro (interest/desire, picks up the claim) → feature/proof sections (desire + proof) → CTA (action).

## Self-check

Sentences short · captions/eyebrows used · no walls of text · every claim resolved (especially problem→solution) · button labels name their goal · secondary information paths to sub-pages exist without diluting the primary CTA · hero short (attention), relocated content in the intro pick-up instead of the hero (AIDA).
