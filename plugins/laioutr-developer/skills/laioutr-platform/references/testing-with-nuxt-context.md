# Don't Invent a Seam to Avoid a Nuxt Context

Your app's runtime code (composables, stores, plugins, components under `src/runtime/app/`) can run inside a real Nuxt app under `@nuxt/test-utils`. "The test needs it without Nuxt" is therefore not a reason for a parameter, a `Deps` object, or a narrowed structural interface. Call `useNuxtApp()`, `useConsentStore()`, `useCookie()` and `nuxtApp.hook(...)` in the test. Add a seam only when you can name what the real collaborator cannot do.

```ts
// ✘ an interface whose members wrap composables, globals and the clock
export interface VisitorTokenDeps {
  getCookie: (key: string) => string | undefined;
  setCookie: (key: string, value: string, maxAgeSeconds: number) => void;
  uuid: () => string;
  now: () => number;
  hasAnalyticsConsent: () => boolean;
}
export const createVisitorTokenManager = (deps: VisitorTokenDeps) => { /* … */ };
```

```ts
// ✅ the values the caller genuinely owns, as plain parameters; everything else called directly
export const createVisitorTokenManager = (cookieName: string, maxAgeSeconds: number) => {
  const cookie = useCookie<string | undefined>(cookieName, { maxAge: maxAgeSeconds });
  const consent = useConsentStore();
  const token = crypto.randomUUID();
  // …
};
```

```ts
// ✘ a fake that reimplements the mechanism under test
const fakeNuxt = (handlers) => ({
  hooks: { callHookWith: (caller, _hook, payload) => caller(handlers, [payload]) },
});

// ✅ the real app, the real hook, the real calling convention
const nuxtApp = useNuxtApp();
const unhook = nuxtApp.hook('frontend-core:analytics:project', handler);
```

The fake above ignores the hook *name*, so renaming the hook in production leaves it green. Any fake that restates how a framework mechanism works can drift from it silently. That is the whole class.

## Getting the context in an app package

The app starter ships a `playground/` (your module plus Frontend Core) and an SSR smoke test in `test/basic.test.ts` that uses `@nuxt/test-utils/e2e`. It has no `vitest.config.ts`. To run runtime tests inside Nuxt **and** keep the e2e test, give each its own Vitest project:

```ts
// vitest.config.ts
import { defineConfig } from 'vitest/config';
import { defineVitestProject } from '@nuxt/test-utils/config';

export default defineConfig({
  test: {
    // `workspace` on the starter's Vitest 3.1; Vitest 3.2+ calls it `projects`.
    workspace: [
      { test: { name: 'e2e', include: ['test/**/*.test.ts'], environment: 'node' } },
      await defineVitestProject({
        test: {
          name: 'nuxt',
          include: ['src/**/*.test.ts'],
          environment: 'nuxt',
          environmentOptions: { nuxt: { rootDir: './playground' } },
        },
      }),
    ],
  },
});
```

- **`rootDir: './playground'` is what loads Frontend Core.** Without it the Nuxt environment boots your package root, which has no Frontend Core module, so its auto-imports are missing. The symptom names a composable, not the cause: `[unimport] failed to find "useConsentStore" imported from "#imports"`. Check `rootDir` before you debug that message.
- **Add `happy-dom` to your devDependencies.** The Nuxt environment loads it, and the starter does not install it.
- **Keep e2e tests out of the `nuxt` project.** `@nuxt/test-utils/e2e` needs the node environment.
- **Two shapes skip tests silently** (checked with `@nuxt/test-utils` 3.19, the starter's version). `defineVitestConfig` from `@nuxt/test-utils/config` with no `include` runs only `*.nuxt.test.ts` and `test/nuxt/**`. With `include: ['src/**/*.test.ts']`, it drops the starter's `test/basic.test.ts`. Neither reports a skipped file.

## Mocking inside the Nuxt environment

- **`vi.mock('#imports', …)` is silently ignored.** Nuxt's import transform rewrites every `#imports` specifier to the real module path before Vitest sees it, so the factory is registered against a module nothing imports. You get no error and no warning, and the test runs the real composable while reading as though it were mocked.
- **To replace an auto-import, use `mockNuxtImport`** from `@nuxt/test-utils/runtime`:

  ```ts
  // ✘ ignored: the real useRuntimeConfig runs
  vi.mock('#imports', () => ({ useRuntimeConfig: () => ({ public: { flag: true } }) }));

  // ✅
  import { mockNuxtImport } from '@nuxt/test-utils/runtime';
  mockNuxtImport('useRuntimeConfig', () => () => ({ public: { flag: true } }));
  ```

- **`vi.mock('#app', …)` reaches only code that imports from `'#app'` literally.** Code that uses the auto-import, or imports from `#imports`, is not affected.
- **A collaborator module** you wrote: `vi.mock('<path relative to the test file>')`.
- **A clock or an id source:** `vi.useFakeTimers()` (with `vi.setSystemTime`) and `vi.spyOn(crypto, 'randomUUID')`. Neither needs a parameter.
- **Consent:** install an adapter with `useConsentStore().setAdapter(...)` and push state through the `report` function its `setup` receives, then `await nextTick()`. That keeps the store's own watcher in the path. Hold the stop handle `setAdapter` returns and call it in `afterEach`.

  ```ts
  import { nextTick } from 'vue';
  import type { ConsentReport } from '#frontend/consent';

  let report!: ConsentReport;
  const stop = useConsentStore().setAdapter({ name: 'test', setup: (r) => { report = r; }, openConsentUi: () => {} });
  report({ analytics: true });
  await nextTick();
  expect(useConsentStore().hasPurposeConsent('analytics')).toBe(true);
  ```

## Tells that a seam answers a question that already has an answer

- **A comment naming tests as the motive.** `// so it stays testable without Nuxt` next to an interface.
- **One clock injected, another called directly.** A store that routes `Date.now()` through a `now` option while calling `performance.now()` three lines away injected the clock its tests wanted, not one the code needs. An injected clock is also frozen, so it cannot express "the same instance, an hour later". The seam removes a test case rather than adding one.
- **A narrowed structural interface.** `{ context: Record<string, unknown> }` standing in for `H3Event` accepts anything that has a `context`, an Orchestr handler's arguments included, so the wrong argument compiles. Take the real type.
- **A bound the seam made observable.** That is no reason to keep the seam. Extract the computation as a pure function (`maxAgeSeconds(createdAt, now)`) and test it directly.

The rationalization to expect, written by an agent that had not read this rule:

> The real `useCookie` is a Nuxt auto-import that only exists inside a Nuxt app context. Stub it with an in-memory ref-like store so the module under test can run under plain vitest, fast and without booting Nuxt.

Each clause is false here. The context exists. The `nuxt` project already runs the file inside it. And a stub aimed at `#imports` silently does nothing.

## What a synchronous fake hides

A cookie fake that stores values in a `Map` writes immediately. Nuxt's `useCookie` writes through a watcher with Vue's default `pre` flush, so `document.cookie` lags a tick. A test named *"deletes synchronously on revoke"* against such a fake asserts a property the shipped code never had, and keeps passing however far the deletion is deferred. Any fake that is more immediate, more ordered or more total than the real collaborator moves the test off the behaviour it names. "The fake is simpler" is an argument against it.

## When a seam is still right

Not "is it DI": dependency injection that earns its keep looks identical. Ask **what the seam lets the test reach that the real collaborator cannot**:

- **A network or a server that isn't there.** For example, a transport that takes a `send` function. Budget for the cost: everything the default implementation does is then untested and needs its own test.
- **Two instances of a per-app singleton.** One test process has one `nuxtApp`, so proving that two concurrent SSR requests do not share state needs two.
- **A bound whose production value makes the test absurd**, such as evicting at a real 50-event or 256 KB limit.
- **A second real implementation.** `ConsentAdapter` has one implementation per CMP. That is an interface, not a seam.
- **A library that captured a global at import time.** `lru-cache` binds `performance` when it loads, and fake timers install a new global it never sees, so its own `perf` option is the only way in. Measure before you accept this one, because it is also the shape a rationalization takes.
- **A genuine layer boundary.** A delivery state machine that imports nothing from Nuxt, plus a few lines that wire the real store and hooks into it. The split survives deleting every test.

One real implementation is not, on its own, a verdict. Before concluding that a seam is needed, check the candidate against fake timers, a `crypto.randomUUID` spy and `vi.mock` of the collaborator module. Those three cover most cases.

## When this doesn't apply

- **Orchestr handlers and other Nitro server code.** Nitro server auto-imports such as `defineOrchestr` are `undefined` in this environment. Keep handlers thin, test the pure mapping in `orchestr-helper/` and `mappers/` directly, and exercise handlers end to end through the playground and the Orchestr DevTools Request Editor.
- **Pure functions** need no Nuxt context. They run fine in the `nuxt` project too; don't add one for them.

## Removing a seam

Converting a test to the Nuxt context and finding it still green proves nothing on its own. A test that no longer gates anything is worse than the seam you removed. Sabotage the behaviour it covers, confirm that the test fails, then restore.

## Why

1. **Fakes of framework mechanisms drift silently.** They restate how Nuxt works, and when Nuxt or your hook name changes, they stay green.
2. **The seam costs production code for a test convenience that already exists.** `@nuxt/test-utils` provides the real context, so the extra parameter or interface buys nothing.
3. **The mock you think you have may be a no-op.** `vi.mock('#imports')` runs the real code without complaint, so a test can read as isolated while asserting nothing.
4. **A wrongly configured runner hides tests, not just failures.** A missing `rootDir` fails loudly, but a wrong `include` makes tests disappear.

## Related

- [`app-settings-manifest.md`](./app-settings-manifest.md): the manifest test lives in `src/` and runs in the `nuxt` project.
- [`nuxt-hooks.md`](./nuxt-hooks.md): naming the hooks you test against.
