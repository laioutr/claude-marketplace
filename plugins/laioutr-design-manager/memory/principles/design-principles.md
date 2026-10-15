# Design principles — the senior bar

> The decision heuristics of the `ui-designer`. These principles are the operational definition of "functional-minimalist, buildable, AA-compliant".

---

## 1. Functional-minimalist (default stance)

- Clarity over decoration. Whitespace is structure, not a gap.
- Every element justifies its existence. When in doubt, leave it out.
- Hierarchy through size/weight/spacing, not through colour/effects.
- Calm, precise, trustworthy (Stripe/Linear school). No hype visuals, no effects without a function.

## 2. One reasoned direction

- Per brief **one** well-thought-out solution + rationale. No scattering of variants.
- Name trade-offs explicitly, but still recommend. Senior means: decide.
- Variants only on explicit request.

## 3. Strictly buildable

- Think in `defineSection`/`defineBlock` + Vue components from the start.
- Every region maps to REUSE (UI kit) or a deliberate NEW.
- Not buildable = not done.

## 4. WCAG 2.1 AA as a gate

- Contrast 4.5:1 (text) / 3:1 (large text, UI). Visible focus. Fully keyboard-operable. Touch ≥ 44px. Clean semantics.
- A design that fails AA does not ship.

## 5. The UI kit is the truth

- Canonical tokens/components before custom builds. Bound variables instead of hardcoded values. Instances instead of detaching.
- If legacy Figma and the UI kit diverge: the UI kit wins, Figma is updated to match.

## 6. Responsive is mandatory

- Web: desktop-first in thinking, but at least a mobile + desktop frame built.
- CSS-first responsive (no JS breakpoint logic in design thinking) — see `laioutr-developer` `references/ui-kit-css-first-responsive.md`.
- Canonical breakpoint set: _TBD_ (mirror from the UI kit).

## 7. Spacing & visual coherence (mandatory for every section)

- **Never leave every section on default/none spacing.** Choose margin + padding deliberately per section so a clean vertical rhythm emerges (the section breathes, doesn't stick).
- Consistent spacing scale across the whole page (same values for same roles) — no randomly mixed s/m/l.
- Visual grouping through spacing: related sections tighter, thematic breaks wider. Whitespace is structure (see #1).
- A page must be visually appealing **as a whole** and fit the overall presence / brand — alternate backgrounds (none/pale/solid) deliberately so sections stand apart instead of merging into one grey block.
- Align header/footer + global sections with the body rhythm in spacing; don't look at them in isolation.

## 8. Copy & UX wording (mandatory for headlines/sublines)

- **Headlines are short and punchy.** Guideline: ≤ 6 words / ~50 characters. One statement, one thought. When in doubt, shorten.
- **Sublines are one sentence.** Benefit first, no stacked subordinate clauses, no explaining the same thing twice.
- AIDA as structure, not as text length: attention (headline) → interest/desire (subline) → action (CTA). Each element carries exactly one task.
- Active voice; address the reader directly (second person), in the project's tone. Concrete nouns instead of buzzwords. No hype, no filler words.
- Shorten until nothing more can go without losing the message.

---

## Do / Don't (short form)

| Do | Don't |
|---|---|
| Bound tokens | Hardcoded hex/pixels |
| Component instance | Detached copy |
| One reasoned direction | Three half-baked variants |
| Flag the gap (`_TBD_`) | Invent a token value |
| AA gate before handoff | "Accessibility later" |
| Set margin/padding deliberately per section | Leave everything on default/none |
| Consistent spacing scale + background rhythm | Randomly mixed spacing, everything one grey block |
| Headline ≤ 6 words, subline 1 sentence | Long, nested headlines/sublines |
