# Accept `MaybeRefOrGetter` When a Function Stores a Value That Renders Later

When a function **stores a caller's value in reactive state that outlives the call**, and something else **renders it later**, type the parameter `MaybeRefOrGetter<T>`. Resolve it with `toValue` where it is read, not at call time. A plain `T` freezes the value when the call happens; a getter keeps it tracking the caller's state (active locale, props, entity data) after the call returns.

This applies to imperative composables and stores your module ships. The ui-kit stores already work this way, so when you **call** them with user-facing text, pass a getter:

```ts
import { useDialogStore, useLocale, useToasterStore } from '#imports';

const { openDialog, closeDialog } = useDialogStore();
const { addToast } = useToasterStore();
const { t } = useLocale();

// ✘ snapshot — the title stays in the old language after a locale switch
addToast({ title: t('addToCart.successToast'), variant: 'success', orientation: 'horizontal' });

// ✅ getter — re-resolved every time the toast renders
addToast(() => ({ title: t('addToCart.successToast'), variant: 'success', orientation: 'horizontal' }));

// ✅ the returned id is usable inside the getter
const dialogId = openDialog(() => ({
  title: t('withdrawal.heading'),
  props: { onClose: () => closeDialog(dialogId) },
}));
```

`openDialog` and `addToast` take `MaybeRefOrGetter<…>` and return the entry's id. `useLightboxStore().openLightbox` deliberately stays plain (see below).

## When to widen — all three must hold

1. **Imperative capture into persistent reactive state.** The function pushes the value into a store `ref` or array, a global, or a queue that outlives the call (typically `createGlobalState`, a module-level `ref`, or a Pinia store).
2. **Rendered later.** A separate component reads that state and renders it. The call does not consume the value itself.
3. **Not an identity key.** The value is not used for dedup or lookup.

The payoff is largest for **user-facing text** (title, message, label). That is what must follow `$tl` / `useLocale` after a locale switch.

## When not to widen — keep the plain value

- **Compute-and-return functions.** The caller owns the return value and can wrap it in `computed` themselves.
- **Identity-keyed stores.** If the stored value is looked up via `.includes(x)`, `.indexOf(x)` or a `Map`/`Set` key, a re-resolved getter yields a new object each read and **breaks the lookup**. `useLightboxStore().registerMedia` / `openLightbox` match `Media` by identity, so they stay plain. Check for this explicitly.
- **Already reactive.** The parameter is already `Ref`, `MaybeRef` or `MaybeRefOrGetter`.
- **One-time config or setup.** Flags, limits, registration — reactivity means nothing there.
- **Synchronous consumption.** The value is used inside the call and not kept.

## How to implement

1. **Signature:** `fn(x: MaybeRefOrGetter<T>): …`.
2. **Resolve inside the `computed` that renders it.** Calling `toValue(x)` once at call time re-freezes the snapshot.
3. **Keep store-owned state separate.** Fields the store controls (`id`, `open`) are not read from the caller's value — spread the resolved content first, then override them.
4. **Store a normalized getter `() => toValue(x)`, not the raw `MaybeRefOrGetter`.** A `Ref` stored inside `ref([])` is deep-unwrapped by Vue's `UnwrapRef`, which drops the `Ref` arm from the type and severs the reactive link. A plain function is not unwrapped.
5. **Return an id** when the caller may need to reference the entry from inside its own getter (`onClose: () => closeDialog(id)`).

```ts
import { computed, type MaybeRefOrGetter, ref, toValue } from 'vue';

interface BannerContent { title: string; message?: string }

interface Entry {
  id: string;
  open: boolean;                // store-owned lifecycle
  source: () => BannerContent;  // normalized getter — NOT MaybeRefOrGetter<BannerContent>
}

const entries = ref<Entry[]>([]);

// source() runs INSIDE the computed, so it re-resolves when the getter's dependencies change
const activeBanners = computed(() =>
  entries.value.filter((e) => e.open).map((e) => ({ ...e.source(), id: e.id, open: e.open }))
);

function showBanner(input: MaybeRefOrGetter<BannerContent>): string {
  const id = crypto.randomUUID();
  entries.value.push({ id, open: true, source: () => toValue(input) });
  return id;
}
```

Callers get every style for free: plain object, `ref`, `computed`, or getter.

## Why

1. **The frozen-snapshot bug.** A plain-value capture bakes the text in at call time. A dialog or toast opened before a locale switch, or built from props that later change, renders stale text. The only fix is an imperative `update*` call.
2. **One signature, every caller style.** `toValue` accepts a value, `ref`, `computed` and getter. Widening costs callers nothing and removes the need for a parallel `update*` method.
3. **The getter reads better** at the call site than wrapping individual fields in `computed()`.

## Related

- [`translations-tl-vs-useLocale.md`](./translations-tl-vs-useLocale.md) — getting `t` in script for the getter
- [`component-state-contract.md`](./component-state-contract.md) — the `open` / `update:open` channel that store-owned lifecycle fields usually mirror
- [`nuxt-hooks.md`](./nuxt-hooks.md) — "where a hook earns its keep" applies the same reachability test to interception points
