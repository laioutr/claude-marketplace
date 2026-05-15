# Storybook Atomic Design Categorization

ui-kit Storybook stories are organized by Atomic Design. Every new story must pick the right category up-front — moving stories later breaks Chromatic baselines.

## Title format

```
title: 'UI Kit/<Category>/<ComponentName>'
```

Categories: `Atoms`, `Molecules`, `Organisms`, `Foundations`.

## How to categorize

### Atoms

Single-purpose, indivisible primitives. One tag, one prop-driven control — the consumer never composes sub-parts. Internal composition of utility wrappers (Icon, VisuallyHidden) is fine.

Examples: `Button`, `Input`, `Checkbox`, `Slider`, `Badge`, `Label`, `Avatar`, `Switch`, `Separator`, `Progress`, `Pagination`.

### Molecules

Small composed units made of 2+ atoms working together as a single functional element. Either the consumer composes exposed sub-components, or the wrapper internally wires several atoms together as one unit.

Examples: `Field` (Label + Input + helper text), `InputGroup`, `SearchInput` (Input + Icon + Button), `PasswordInput` (Input + visibility toggle), `Combobox` (Input + Listbox), `ColorSwatchPicker`, `InputPin` (a coordinated row of input cells), `Tabs` (triggers + panels), `Popover` (trigger + portaled content), `Tooltip` (trigger + content), `ContextMenu`, `DropdownMenu`, `HoverCard`, `Accordion`, `Card` (media + text + CTA wired as one unit).

### Organisms

Larger components with their own internal structure and multiple atoms/molecules. Often have their own slots and can host blocks of content.

Examples: `Header`, `Footer`, `NavigationMenu`, `Dialog`, `Sheet`, `LightboxGallery`, `EmptyState`, `Toaster`.

### Foundations

Not components — design system primitives: typography scales, color palettes, spacing tokens, icon catalog, theme reference. Stories here are reference docs, not interactive components.

Examples: `Text` (typography reference), `Icon` (icon catalog), `Tokens` (color/spacing showcase).

## Edge cases

- **Wrapping a Reka primitive is not automatically an Atom.** Decide by the consumer-facing API, not the wrapper's implementation:

  - One tag, one prop-driven control → Atom. E.g. `Switch`, `Slider`, `Checkbox`, `Pagination`. The consumer never composes sub-parts.
  - Multiple exposed sub-components the consumer composes (`<Tabs>` + `<TabsTrigger>` + `<TabsContent>`, `<Popover>` + `<PopoverTrigger>` + `<PopoverContent>`) → Molecule. The composition is the API.
  - Coordinated cells wired together as one unit (`InputPin` iterating `PinInputInput`) → Molecule. Internal composition is still composition.
  - Flat wrappers that hide reka's internal structure behind a single prop surface (e.g. `Slider` renders Track + Range + Thumb internally) → Atom. The consumer still writes one tag.

  The test is _what the consumer writes_, not what's in the wrapper's template.

- **Domain-specific compositions** (price displays, product chrome) → don't belong in ui-kit at all. Push to `ui`.
- **Layout helpers** (`GridFill`, `GridMasonry`, `OnSurface`) → Atoms — they have a single layout job and no further composition.
- **Compound components** with sub-parts (`Tabs` + `TabsTrigger` + `TabsContent`) → one story under the parent's category. Don't create separate stories per sub-part.

## Why this matters

- Storybook navigation stays predictable as the kit grows
- Chromatic baselines stay stable (the title is part of the snapshot key)
- Cross-team discovery: designers and product expect Atomic Design hierarchy
- Forces the right architectural question up-front: if you can't decide between Atom and Organism, the component is probably doing too much
