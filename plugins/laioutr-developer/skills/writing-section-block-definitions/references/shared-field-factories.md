# Shared Field Factory Functions

When a `shared-fields/*.ts` export is a **function** that returns a field (as opposed to a static object), the return type must narrow to a literal — especially the `name` property — even when the call site lives inside `defineSection`/`defineBlock`'s `fields` array.

Canonical reference: [Shared field factories — Laioutr docs](https://docs.laioutr.com/apps/app-development/shared-field-factories).

## The contextual-widening trap

Schema field arrays in `defineSection` are contextually typed as `StudioFieldDefinition[]` (a union). Union members carry `name: string`. When TypeScript resolves a generic factory call inside that array, the contextual type can **widen** a generic type parameter back to `string`, even though `const` modifiers and template-literal defaults are declared.

Concretely, this pattern is broken:

```ts
export const visibilityField = <
  const ForKey extends string,
  const Name extends string = `${ForKey}Visible`,
>(opts: { for: ForKey; name?: Name; ... }) => {
  const field: { ...; name: Name } = { ..., name: opts.name ?? `${opts.for}Visible` };
  return field;
};

defineSection({
  schema: [{ fields: [
    { name: 'heading', type: 'text' },
    { name: 'subline', type: 'text' },
    visibilityField({ for: 'subline' }),  // ← Name widens to `string` here
  ]}],
});
```

Standalone (`const f = visibilityField({ for: 'subline' })`), `Name` infers to `"sublineVisible"`. Inside the array, contextual typing from the union's `name: string` widens `Name` to `string`. The field's `name` property becomes `string`, and `FindFieldWithName<Fields, 'heading'>` then matches **both** the text field and the checkbox, producing `string | boolean` for `props.heading` and breaking every downstream consumer.

## The fix: single Opts generic + conditional resolution

Collapse the multiple type parameters into a single `const Opts extends {...}` generic and compute the resolved `name` at the **return type** via a conditional on `Opts`. The resolved name is derived structurally from `Opts`, not inferred as an independent generic with a default — so contextual typing can't widen it.

```ts
export const visibilityField = <
  const Opts extends {
    for: string;
    name?: string;
    label?: string;
    default?: boolean;
  },
>(
  opts: Opts
) => {
  type ResolvedName = Opts extends { name: string } ? Opts['name'] : `${Opts['for']}Visible`;

  const field: {
    type: 'checkbox';
    as: 'visibility';
    for: Opts['for'];
    name: ResolvedName;
    label?: string;
    default?: boolean;
  } = {
    type: 'checkbox',
    as: 'visibility',
    for: opts.for,
    name: (opts.name ?? `${opts.for}Visible`) as ResolvedName,
  };
  if (opts.label !== undefined) field.label = opts.label;
  if (opts.default !== undefined) field.default = opts.default;
  return field;
};
```

## Rules of thumb

Use this pattern whenever you add a `shared-fields/*.ts` factory that:

- Takes a sub-property typed with `const X extends string` to drive a literal.
- Has a second type parameter with a template-literal default depending on the first (e.g. `Name = \`${ForKey}Visible\``).
- Returns a field that will be placed inside a schema array constrained to a union (`StudioFieldDefinition[]`, `BaseFieldDefinition[]`, …).

Prefer a **single `Opts` generic** and derive everything from `Opts[...]`. Never rely on a template-literal generic default to compute a literal name — it breaks under contextual typing.

Static factories that accept the whole object via `const T extends StudioFieldDefinition` (like `defineField`/`defineFieldset`) are fine — they don't have this problem because the entire argument is captured as `T`, not a sub-property.

## How to detect the regression

If `FindFieldWithName`-driven prop types suddenly become `string | boolean` (or `string | number`, etc.) instead of a single primitive, and the schema contains a factory-generated checkbox/number alongside a text field of a different name, suspect name-widening. Add a one-off `type Check = typeof def['schema'][number]['fields'][number];` in the component and inspect it via a deliberate type error — if the checkbox's `name` shows as `string`, this rule applies.
