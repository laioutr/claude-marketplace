# App Settings Come From Your Manifest: Names and Types, Never Values

Declare the settings a customer must enter for your app (an API host, a token, a store id) in `laioutr.manifest.json` at your package root. The Cockpit turns the manifest into a form under each environment's Configuration, stores secrets encrypted, and offers a **Test connection** button. Saved values reach your Nuxt module as its options on the environment's next deployment, so your module code does not change. The manifest says **which** settings exist and how to show them. It never carries a value.

## How a manifest reaches a customer

1. **The manifest is in your tarball.** List `laioutr.manifest.json` in `package.json → files`, next to `dist`. npm packs only what `files` names. Check with `npm pack --dry-run`.
2. **You publish to the Laioutr registry** (`npm.laioutr.cloud`). The registry reads the manifest once, at publish, and keeps it with that version (max 256 KB). A package published only to npmjs.org gets no form. A manifest that is invalid JSON or breaks a validation rule does **not** stop the publish; that version simply has no form. A published version cannot be replaced, so a fixed manifest needs a new version.
3. **You release the version** with `laioutr app release`. The Cockpit records the version, its peer dependencies, its channel and its settings form.
4. **The customer saves values per environment.** They take effect with that environment's next deployment, as the options of your module.

## A minimal manifest

```json
{
  "configSettings": {
    "menu": { "label": "Acme Search" },
    "sections": [
      {
        "label": "Account",
        "description": "Acme dashboard > Settings > API keys. Use a read-only key.",
        "fields": [
          { "name": "apiHost", "type": "text", "label": "API host", "placeholder": "eu.api.acme.example.com", "required": true },
          { "name": "apiToken", "type": "secret", "label": "API token", "required": true }
        ],
        "test": {
          "hosts": ["*.api.acme.example.com"],
          "method": "GET",
          "url": "https://{{apiHost}}/v1/account",
          "headers": { "Authorization": "Bearer {{apiToken}}" },
          "expect": "account.id"
        }
      }
    ]
  }
}
```

```ts
// src/module.ts
export interface ModuleOptions {
  apiHost: string;
  apiToken: string;
}
```

## Fields are module options

- **Each field's `name` is a key of your `ModuleOptions`.** Your module receives one flat options object, so a `name` is unique across **all** sections. `__proto__`, `constructor` and `prototype` are refused. The one exception is an `info` field: it shows a notice and holds no value.
- **Field types:** `text`, `textarea`, `number`, `select`, `radio`, `checkbox`, `secret`, `info`, `json`. `richtext`, `array` and `object` validate but the Cockpit cannot edit them yet. The validator also accepts `toggle_button`; use `checkbox` or `select` / `radio` instead.
- **`default` pre-fills the form only.** It is not a fallback your module receives. An empty optional field is removed from the stored settings, so keep a real default in your module's `defaults`.
- **Mark a field `required` only when your module cannot start without it.** A required field that is empty blocks saving **every** other setting of your app in that environment. That includes customers whose value still comes from an environment variable on their own host.
- **Set `configSettings.menu.label`.** Without it the form is headed by the catalogue title, which for a first-listed third-party app is the last segment of the package name (`search` for `@laioutr-org/acme__search`). `menu.icon` is accepted but not shown. `placement` accepts `environment | project | organization | studio`, but only `environment` is rendered; leave it out.

## Secrets and groups

- **A credential is `"type": "secret"` with no `default`.** The validator refuses a default on a secret. The Cockpit stores it encrypted and afterwards shows only bullets plus the last four characters (for values of 20 characters or more).
- **A placeholder is a shape, never a real value** (`acme-shop.example.com`).
- **A group your module reads as one object is one `json` field.** If your module reads `search?: { appId: string; adminApiKey: string }`, declare `{ "name": "search", "type": "json" }`. The customer pastes the object as JSON, and your module receives the same object it always read.
- **A group that holds any credential adds `"secret": true`.** It is then sealed as one value and shown only as set, never any part of it. The validator refuses a default only on `type: "secret"`, not on a sealed `json` group, so check that one yourself.
- **Prefer flat fields when you can choose.** A flat field gets its own label, validation and help text, and a connection test can use it. A test never sees a value inside a `json` group.
- **Keep secrets out of public runtime config.** A `secret` option belongs in `RuntimeConfigModulePrivate` only, never merged into `runtimeConfig.public[name]`.

## Connection tests

A section may declare one `test`: a request the Cockpit sends from its server, with the values on screen or the stored ones, secrets included. The validator enforces:

- **`hosts`** lists exact names (`api.acme.example.com`) or subdomain wildcards (`*.acme.example.com`). IP addresses, ports, a bare `*` and internal names (`localhost`, or names ending in `.local`, `.internal`, `.intranet`, `.lan`, `.home`, `.corp`) are refused.
- **`url`** starts with `https://`.
- **No secret in the URL.** A `{{secret}}` placeholder goes in `headers` or `body`, because an address ends up in logs and error messages.
- **`method`** is `GET` or `POST`, and `body` is allowed with `POST` only.
- **Every `{{name}}` names a field.** It may be a field from another section. Only text and number values fill a placeholder.
- **A section with a `test` has a `label`**, unique among the sections that have tests.

One rule the validator cannot check: **the request must read and change nothing** in the customer's account. At run time the Cockpit encodes each value where it lands, refuses hosts that resolve to private addresses, does not follow redirects, and times out after 10 seconds. The browser learns only pass or the reason for failure, never the address, the answer, or any part of a credential. A test runs only when the customer clicks it, never on save or before a deployment.

## Test the manifest in your package

`validateAppManifest` ships in `@laioutr-core/core-types` 0.64.0 and later. The app starter pins an older range, so raise it.

```ts
// src/laioutrManifest.test.ts
import { describe, expect, it } from 'vitest';
import { validateAppManifest } from '@laioutr-core/core-types/app';
import manifest from '../laioutr.manifest.json';
import type { ModuleOptions } from './module';

/** Every option the module reads. The type forces the list to be complete and to name nothing else. */
const OPTIONS: Record<keyof ModuleOptions, true> = {
  apiHost: true,
  apiToken: true,
};

const fields = manifest.configSettings.sections.flatMap((section) => section.fields);

describe('laioutr.manifest.json', () => {
  it('is a sound manifest', () => {
    expect(validateAppManifest(manifest)).toEqual([]);
  });

  it('names only options the module reads', () => {
    expect(fields.map((field) => field.name).filter((name) => !(name in OPTIONS))).toEqual([]);
  });

  it('marks every credential as a secret', () => {
    expect(fields.filter((field) => field.type === 'secret').map((field) => field.name)).toEqual(['apiToken']);
  });
});
```

Renaming a `ModuleOptions` key makes the `OPTIONS` literal fail type-checking (`test:types`). Once you fix the literal, the second test catches the stale manifest name before a customer does. Leave `info` fields out of that comparison. If a test URL embeds an API version, assert it against the constant your client calls, so the test and the module cannot drift apart.

## Releasing so the form appears

```bash
LAIOUTR_API_KEY=orgKey_xxx npx @laioutr/cli app release
```

- **Run it in the package folder, once for every version you publish.** It reads `name` and `version` from `package.json` and refuses (`409`) a version the registry does not serve. Running it again for the same version is safe. It installs nothing and changes no environment.
- **Use an organization API key with the `app:publish` scope, from the organization that owns the package.** That is the organization whose slug `@laioutr-org/<organization-slug>__<package-name>` carries, or one the owner granted publishing to. A key from any other organization gets `404`, the same answer as for an unknown name. This is easy to hit when you belong to both your own organization and a customer's.
- **The release does not report manifest problems.** Run `validateAppManifest` before you publish (the test above does).
- **The first release of a third-party package lists it as a private app** of your organization. It appears in no project's app catalogue, and its form shows only in environments that already include the package. Contact Laioutr to make it public.
- **Only versions that carry the manifest get a form.** An environment pinned to an older version shows nothing until it moves to a version with the manifest. A missing field needs a new version, published and released.

## Hard rules

1. A manifest names settings and never carries a customer's value: no token, key or default credential, not even as a placeholder.
2. Every credential is `type: "secret"` (or a `json` group with `secret: true`) and has no `default`.
3. Every field `name` (except `info`) is a `ModuleOptions` key; prove it with a `Record<keyof ModuleOptions, true>` test.
4. `laioutr.manifest.json` is listed in `package.json → files`.
5. `required` only when the module cannot start without the value.
6. Connection tests name their hosts, use `https`, never put a secret in the URL, and change nothing.
7. Set `configSettings.menu.label`.
8. Release every published version with an `app:publish` key from the organization that owns the package.

## Why

1. **A value in the manifest ships to everyone.** The manifest is inside your published tarball and kept by the registry. A default token there is a leaked token.
2. **The field name is the contract.** The saved value reaches your module under exactly that name. A renamed option silently turns the form field into a value nothing reads.
3. **Mistakes fail quietly.** Neither publish nor release reports an invalid manifest. The version just has no form, and the customer sees nothing, so your own test is the only place the error shows up.
4. **A required field that is not truly required blocks the customer.** It stops them from saving anything else, including settings that would work.

Full reference: [App Manifest](https://docs.laioutr.com/apps/app-development/app-manifest) and [Releasing an App](https://docs.laioutr.com/apps/app-development/releasing-an-app).

## Related

- [`fail-soft-resilience.md`](./fail-soft-resilience.md): a missing required credential is a fail-hard case; name the option in the error.
- [`testing-with-nuxt-context.md`](./testing-with-nuxt-context.md): where the manifest test runs in your Vitest setup.
