# Component Matching and Placement

Used in Phase 3 of `figma-design-analysis`. Search the existing inventory before planning anything NEW, describe modification intent for REUSE components, run cross-file verification on existing components, place components in the right package, decompose complex designs into hierarchies, and use the Early Exit path when nothing new is needed.

## Component Matching

Before planning ANY new component, search the existing inventory.

**Prefer modifications to existing components over creating new ones.** Before proposing a NEW component, evaluate whether an existing component can be extended. Mark such components as REUSE.

### Search Process

```bash
# 1. Check @laioutr-core/ui-kit components (73 atomic components)
ls node_modules/@laioutr-core/ui-kit/src/runtime/app/components/

# 2. Check @laioutr-core/ui components (140 commerce organisms)
ls node_modules/@laioutr-core/ui/src/runtime/components/
ls node_modules/@laioutr-core/ui/src/runtime/components/organism/

# 4. Search by keyword from the Figma design
grep -r "ComponentNameFromFigma" \
  node_modules/@laioutr-core/ui-kit/src/ \
  node_modules/@laioutr-core/ui/src/
```

(Path layout assumes npm/yarn; pnpm users adjust to `node_modules/.pnpm/...`.)

### Common Matches

| Figma Element | Existing Component | Import Path |
|---|---|---|
| Text / Heading / Body / Caption | CSS classes (deprecated `Text`) | Use `heading-m`, `body-s`, `caption-xs` etc. on semantic HTML elements |
| Button (any variant) | `Button` | `#ui-kit/components/Button/Button.vue` |
| Button adapting to background | `BackgroundAwareButton` | `#ui-kit/components/BackgroundAwareButton/BackgroundAwareButton.vue` |
| Icon | `Icon` | `#ui-kit/components/Icon/Icon.vue` |
| Image / Media | `Media` | `#ui-kit/components/Media/Media.vue` |
| Input / Text Field | `Input` | `#ui-kit/components/Input/Input.vue` |
| Form Field (label + input + error) | `Field` | `#ui-kit/components/Field/Field.vue` |
| Checkbox | `InputCheckbox` | `#ui-kit/components/InputCheckbox/InputCheckbox.vue` |
| Dropdown / Select | `Select` | `#ui-kit/components/Select/Select.vue` |
| Card container | `Card` | `#ui/components/Card/Card.vue` |
| Dialog / Modal | `Dialog` | `#ui-kit/components/Dialog/Dialog.vue` |
| Accordion | `Accordion` + `AccordionItem` | `#ui-kit/components/Accordion/` |
| Star rating display | `StarsRating` | `#ui-kit/components/StarsRating/StarsRating.vue` |
| Color swatch | `ColorSwatch` | `#ui-kit/components/ColorSwatch/ColorSwatch.vue` |
| Product tile | `ProductTileBasic` | `#ui/components/organism/ProductTileBasic/` |
| CTA Banner base | `CtaBannerBase` | `#ui-kit/components/CtaBanner/CtaBannerBase.vue` |
| Product flags (sale/new/promo) | `ProductTileFlag` | `#ui-kit/components/ProductTileFlag/` |
| Progress bar | `ProgressBar` | `#ui-kit/components/ProgressBar/ProgressBar.vue` |
| Loading spinner | `LoadingSpinner` | `#ui-kit/components/LoadingSpinner/` |
| Navigation menu | `NavigationMenu` + subcomponents | `#ui-kit/components/NavigationMenu/` |

**Always compose from existing components. Never recreate primitives.**

### Describing Modification Intent (REUSE Components)

For each existing component that needs modification, describe **what** needs to change and **why** at a high level. Do NOT specify exact props/slots — that's for a future `component-architecture` skill.

**Good:** "RadioSelectItem needs to support expandable content when selected and a custom icon area — the checkout shipping selector shows a radio that reveals delivery date options on selection."

**Bad:** "Add an `expandable` prop of type boolean and a `#icon` slot to RadioSelectItem."

### Cross-File Verification

When analyzing an existing component with a **section wrapper** (e.g., `SectionBrandHero` wraps `BrandHero`), trace props from the wrapper into the presentation component:

1. **Check for double processing** -- If the wrapper calls a utility (e.g., `colorValueToCss`) on a value and then the inner component calls it again, flag it. Classify severity: if the function is idempotent on its own output, it's a code smell / confused responsibility rather than a functional bug.
2. **Check for prop/schema mismatches** -- If the inner component accepts a prop value but the section schema doesn't expose it as an option, flag the discrepancy.
3. **Check for dead injection keys** -- If `types.ts` defines an `InjectionKey` / `Symbol`, verify that matching `provide()` and `inject()` calls exist in the component tree. Unused injection keys are dead code.
4. **Check for duplicate CVA classes** -- If the same CSS class is added by multiple CVA definitions, flag the redundancy.
5. **Check for overlapping CVA compound variants** -- When multiple compound variants can match simultaneously, trace which utility classes apply and whether they conflict. Dense padding/margin compound variants with breakpoint overrides are especially error-prone.
6. **Check for fragile CSS coupling** -- If CSS selectors reference another component's internal class names (e.g., MediaText targeting `.cards-container__inner` from Backdrop), flag as fragile coupling.
7. **Check for hardcoded config maps** -- If a component has a hardcoded map keyed by theme ID, verify completeness:

```bash
# List all registered themes
ls node_modules/@laioutr-core/ui-kit/src/runtime/app/theme/

# Compare against hardcoded keys in the component
grep -n "laioutr\|classic\|tech\|sunny\|strawberry" path/to/ComponentName.vue
```

Missing entries in hardcoded maps are silent bugs -- they fall back to defaults without error.

## Package Placement

**This skill only plans components for `@laioutr-core/ui-kit` and `@laioutr-core/ui`.** Components in these packages are design system building blocks — they receive data via props and emit events, with no direct connection to external systems (APIs, Orchestr handlers, Pinia stores, payment SDKs). Data integration happens exclusively at the `@laioutr-core/ui-app` layer, which is out of scope for this skill.

```dot
digraph placement {
    "What kind of component?" [shape=diamond];
    "Atomic primitive (Button, Input, Card)?" [shape=diamond];
    "Commerce-specific organism?" [shape=diamond];
    "Needs data integration?" [shape=diamond];

    "@laioutr-core/ui-kit" [shape=box];
    "@laioutr-core/ui" [shape=box];
    "Out of scope\n(ui-app / integration layer)" [shape=box, style=dashed];

    "What kind of component?" -> "Atomic primitive (Button, Input, Card)?";
    "Atomic primitive (Button, Input, Card)?" -> "@laioutr-core/ui-kit" [label="yes"];
    "Atomic primitive (Button, Input, Card)?" -> "Commerce-specific organism?" [label="no"];
    "Commerce-specific organism?" -> "Needs data integration?" [label="yes"];
    "Commerce-specific organism?" -> "Atomic primitive (Button, Input, Card)?" [label="no, re-evaluate"];
    "Needs data integration?" -> "@laioutr-core/ui" [label="no — props only"];
    "Needs data integration?" -> "Out of scope\n(ui-app / integration layer)" [label="yes — fetches/mutations"];
}
```

| Package | Component Type | Examples | Directory | In scope? |
|---|---|---|---|---|
| `@laioutr-core/ui-kit` | Atomic, reusable across contexts | Button, Card, Icon, Input, Dialog | `src/runtime/app/components/` | **Yes** |
| `@laioutr-core/ui` component | Mid-level commerce compositions | DarkModeSwitch, RatingInput, SwatchItem | `src/runtime/components/<Name>/` | **Yes** |
| `@laioutr-core/ui` organism | Complex page sections | Header, Footer, CartSheet, ProductGrid | `src/runtime/components/organism/<Name>/` | **Yes** |
| `@laioutr-core/ui-app` section | `defineSection()` page sections with data binding | CmsContainer, BrandHero | `src/runtime/app/section/` | No |
| `@laioutr-core/ui-app` block | `defineSection()` blocks with data binding | — | `src/runtime/app/block/` | No |

## Decomposing Complex Designs

For multi-component designs (checkout flows, full pages):

1. **Identify visual boundaries** -- Each distinct section with its own background, padding, or card container is a candidate component
2. **Check Figma layer names** -- Designers name layers semantically (e.g., "Order Summary", "Payment Form", "Cart Items")
3. **Map to component hierarchy** -- every component is either composed into another component or will eventually be wrapped by a Section/Block in `@laioutr-core/ui-app`:
   - Top-level visual groups -> organisms in `@laioutr-core/ui` (`components/organism/`)
   - Repeated patterns -> components in `@laioutr-core/ui` (`components/<Name>/`)
   - Atomic elements -> already exist in `@laioutr-core/ui-kit`
4. **Start from leaves** -- Plan implementation of innermost components first, then compose outward
5. **One component per Figma logical group** -- Don't create a component for every Figma frame; group by semantic meaning

**Every region of the design must be accounted for.** If a designer created a named component, instance, or visually distinct region in the Figma design, it MUST appear in the Component Hierarchy — as NEW, EXISTS, or REUSE. You do not get to decide that something is "too simple to be a component" or "barely warrants its own component." A checkout header with just a logo and a back-link is still a component. A footer with just legal links is still a component. Simplicity is not a reason to omit — it just means the component is easy to implement.

**The only things excluded from the hierarchy are:**
- Pure text content that maps to CSS typography classes (headings, body text, captions)
- Standard HTML elements that don't need a Vue component wrapper (a `<hr>` divider, a plain `<a>` link)
- Layout primitives that are just CSS (a flex container, a grid wrapper) — unless the designer gave them distinct visual treatment (background, border, padding that makes them a "card" or "section")

**Hidden elements:** Include elements marked `hidden="true"` in the hierarchy but annotate them as `[hidden]`. Hidden elements represent alternate states (e.g., an expanded form hidden in the collapsed view) or optional features (e.g., a focus ring). They are important for understanding the full interaction model.

**Layout frames vs semantic components:** When `get_metadata` shows plain `<frame>` nodes (not `<instance>` or `<symbol>`), distinguish between:
- **Semantic frames** that represent a distinct UI concept (e.g., "express checkout method", "account", "shipping") — these need their own Vue component
- **Structural frames** that are just CSS layout containers (e.g., "main", "content", "left", "right", "row 30/70") — these become `<div>` elements with CSS, not Vue components

Mark structural frames as `(layout)` in the hierarchy so the implementation skill knows they are CSS-only. If in doubt, it's a semantic component — err on the side of creating components.

## Early Exit: All Components Already Exist

After decomposing and matching, if the hierarchy contains **zero NEW or REUSE components** (everything is EXISTS or external), the design does not require new ui-kit/ui work. Instead of producing an empty plan:

1. **Tell the user:** "All components in this design already exist. No new ui-kit or ui work is needed."
2. **Present the hierarchy anyway** as a confirmation/audit — it maps Figma regions to existing components, which is still valuable for verification.
3. **Suggest next steps:** The work may be at the ui-app layer (creating a Section/Block that assembles these existing components) or at the page layout level — both are outside this skill's scope.
4. **Skip Phases 4-5** (section-by-section presentation and challenge) — they add no value when there's nothing to plan.

This commonly happens when analyzing page compositions from "component examples" or showcase files where all UI elements are already implemented.

**Do NOT plan page components, page layouts, or Nuxt routing.** Those are Nuxt-layer concerns handled outside this skill's scope. The top-level output of this skill is always a self-contained, composable component (organism) that can be placed anywhere via a ui-app Section/Block wrapper.
