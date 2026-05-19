---
name: figma-export-assets
description: Use when implementing a component from a Figma frame that contains image content (illustrations, photos, logos, markers, screenshots, custom icons) that needs to live as files in your Laioutr-based Nuxt module. Excludes themable defaults registered via themeMedia/themeResponsiveMedia — that workflow belongs to figma-design-analysis.
---

# Figma → Repo Asset Export

## What this is for

Getting visual assets out of a Figma frame and into the right place in your Laioutr-based Nuxt module, so a Vue/TS component can reference them via a root-relative URL string (`<img src="/cta/cta-banner-bg.png" />`).

In scope:
- **Custom icons** that are not already in the `IconName` union (exported by `@laioutr-core/ui-kit`)
- **One-off raster images** used by a single section/block/composition (decorative photo, screenshot mock, marker, illustration)
- **Brand/partner logos** (payment methods, integrations, third-party badges)

Out of scope (use a different workflow):
- **Themable defaults** registered in `ImageSet` via `themeMedia()` / `themeResponsiveMedia()` — those are theme-system assets, see the `figma-design-analysis` skill.
- **Existing theme icons** in the `IconName` union — reuse the existing key; do not export.

## The MCP can't write asset bytes (read this first)

The `mcp__figma__*` tools (`get_design_context`, `get_screenshot`, `get_metadata`, `get_variable_defs`) return **inline data the agent can view, not files the agent can save**. There is no `mcp__figma__export_asset` tool.

Three real paths to get bytes onto disk:

1. **Figma REST API** (default — try this first regardless of asset count). `https://api.figma.com/v1/images/{fileKey}?ids={nodeId,…}&format={svg|png|jpg}&scale={1|2|3}` returns temporary CDN URLs for every requested node in a single call → `curl` to download → write to the path. Requires a `FIGMA_ACCESS_TOKEN` env var. If one isn't already set, the skill's Step 7 path B prescribes the exact user-facing instructions for creating one. PAT creation is available on every Figma plan including the free Starter tier and takes about 30 seconds. Don't fall back to manual export without offering this option first. Starter-tier caveat: 6 file-content requests per month per file; one export run is 1 request regardless of how many node IDs you batch, so a single pass is fine but repeated dev iterations against the same file will exhaust the limit.
2. **Manual export by the user** (fallback only when the user declines to create a token, or the file is on a non-cooperative network where REST isn't reachable). The agent identifies node IDs + format + scale + target paths and produces a per-node spec the user clicks through in Figma. Tedious; see Step 7 path A for the exact deliverable shape that makes it tolerable.
3. **Verification-only via screenshot**. When neither (1) nor (2) is available, `mcp__figma__get_screenshot` with `contentsOnly: true` produces an inline image the agent can visually verify is the right asset, and the agent hands the export spec to the user as a deliverable.

**Never substitute** a base64 data URI in code for missing bytes, and **never** write a placeholder file with arbitrary content "to be replaced later" — a missing file is a louder signal than a wrong file.

## Step 0 — Verify MCP access to the requested file

The `mcp__figma__*` tools only operate against the browser tab the user currently has focused, and only when the user's Figma seat has Dev Mode access to that file. Both conditions can fail silently. Before any export work, probe both:

```
get_metadata()              # no nodeId — returns the currently selected node or the top-level page list
get_metadata(nodeId)        # the specific node you were asked about
```

**The probes don't verify file identity.** Both calls return information about a *node*, never about the file. If the user's Figma selection happens to match the requested nodeId, the unscoped probe returns identical XML to the scoped one — and there's no `fileKey` in either response to cross-check. So passing both probes only proves *some* node exists at that ID in *some* accessible file; it does not prove the file is the one you were asked about. Close the gap with one of:

1. **Visual content check.** Call `get_screenshot(nodeId)` and confirm the returned image matches what the task implies the node should be (a hero section looks like a hero, an awards strip shows badges, a checkout page shows a form). If the task gives no visual basis for confirmation, fall to (2).
2. **Title-bar confirmation by the user.** The MCP exposes no `fileKey` and no top-level file name. The only ground truth is the Figma desktop title bar. Ask the user explicitly, using the URL's slug as the anchor: *"your active Figma tab — what does the title bar show? The task references file `<fileKey-from-URL>` whose slug is '<filename-slug-from-URL>'. They should match."* Don't paraphrase based on page names; ask for the literal title-bar string.

**Topical fit is not identity proof.** When `get_metadata` returns a generic-looking page list (e.g., `Screens / Sections / Blocks / Components`) that *plausibly matches* the project's vocabulary, do NOT conclude "looks right, must be right." Two different Laioutr-shaped files will return similarly Laioutr-shaped page lists. Identity requires either a visual content match on the requested nodeId or an explicit title-bar confirmation from the user. The cost of skipping is fabricating exports from a wrong-file node tree.

Three distinct error modes to recognize and handle separately:

1. **"No node could be found"** — active tab is on a different file than the URL you were given. Stop and tell the user *"the active Figma tab is on file X / node Y; please switch to file Z for this task."* Wait for them to switch tabs. Do NOT silently work against whatever node *is* selected.

2. **"This resource couldn't be accessed" / Dev Mode messaging** — active tab is correct, but the seat/file permissions block the MCP. Stop and tell the user the file needs Dev Mode access from their seat. Tab switching will not help; only an access change will. If the URL contains `Community` in the slug (e.g. `…--Community-?node-id=…`), it's a Figma Community file and the most common fix is *"duplicate this file into your own workspace, then switch the tab to the duplicate"* — Community files often expose Dev Mode only on the duplicate, not the original.

3. **Empty / no response** — same as (2) operationally; stop and report.

In all three cases, the correct terminal state is **stop, produce a written report, do not fabricate**. Never invent node IDs, frame names, or export specs based on a guess about what the asked-for file might contain. A missing export is a louder signal than a wrong one.

This step exists because the failure modes are silent: the MCP doesn't error loudly when the wrong tab is active or the file is inaccessible, it just returns "no node" or a generic access error that's easy to misread as "the agent should try harder."

## Step 1 — Inventory

Before touching the export tool, list what the frame contains and decide per-element whether it ships as a file.

**Always start `get_metadata` from the requested nodeId, never from a top-level page.** `get_metadata("0:1")` or `get_metadata("<page-id>")` on a large design page can return multiple megabytes of XML — easily 2M+ tokens — and truncate or blow context before you've found the requested node. The requested nodeId narrows the response to just that subtree. If you need to traverse upward (e.g., to confirm the parent frame name), do it from the leaf, not from the page.

**Leaf nodes can share generic names; the `nodeId` is the only stable handle.** It's common for many leaves to share a `name` attribute like `image 22`, `Frame 1234`, or `Rectangle 5` — Figma auto-numbers freely and designers rarely rename. Never use `name` to identify, dedupe, or skip "already-handled" leaves: eight distinct partner badges with `name="image 22"` will collapse to one if you key on the name. Always key on `nodeId`. When producing a Path A spec, surface the generic name in the "Figma exports as" line so the user knows the default filename is ambiguous and will need the rename script to disambiguate.

```
get_metadata(nodeId)        — structure of the frame
get_screenshot(nodeId)      — what each candidate node looks like
```

For each visual element, classify:

| Element | Action |
|---|---|
| Text node (heading, body copy, label, caption, button text) | **Skip.** Component renders this via `t('key')` — never bake into an exported asset. |
| Icon that matches a key in `IconName` (exported by `@laioutr-core/ui-kit`) | **Skip.** Reuse the existing key. |
| Existing public asset (search your module's `src/runtime/**/public/` and the upstream packages' equivalents under `@laioutr-core/{ui-kit,ui,ui-app}`) | **Skip.** Reference the existing path. |
| Decorative shape achievable in CSS (gradient, rounded rectangle, border) | **Skip.** Implement in the component's `<style>`. |
| SDK-rendered chrome (map tiles, autocomplete popovers, browser UI) | **Skip.** Not part of the design. |
| New leaf image / illustration / logo / marker / photo | **Export.** Goes through the rest of the steps. |

## Step 2 — Isolate the image (the #1 rule)

**Export the leaf image node, never a wrapper frame that includes adjacent text or component-rendered chrome.**

A Figma frame called "HeroCard" might visually contain: a background photo + a headline + a subline + a CTA button. The asset that lives in the repo is **only the photo**. Headline, subline, and button are rendered by the Vue component and translated via `useLocale()`.

Verification recipe:
1. `mcp__figma__get_screenshot(nodeId, contentsOnly: true)` on the candidate node.
2. Inspect the screenshot. If you see text, buttons, padding around the image, or any other chrome — you picked the wrong node. Drill into the metadata, find the actual image rectangle (usually a node with `type=RECTANGLE` and an image fill, or `type=VECTOR` for SVG-shaped graphics), and re-screenshot.
3. For SVG candidates after export: open the file as text; flag any `<text>` element unless the asset is intentionally a wordmark logo.

Why this matters: text baked into an asset breaks i18n (no translation), breaks surface-tone adaptation (foreground colour can't follow the container), inflates the file, and breaks the design-system contract that text lives in `<Text>`-family components.

**Exception — text that IS the asset.** Wordmark logos, partner badges, award certificates, and certification stamps contain text *as part of the artwork itself* — the text is the brand identity (e.g. a "HolidayCheck 2026 Award" badge, a "TOP 100 Germany Travel" stamp). Removing it would destroy the asset. This text is kept as-is in the export. The distinction:

- **Component-rendered text** (headlines, sublines, captions, CTAs, body copy) → never bake in. Belongs in the .vue template with `t()`.
- **Brand/artwork text** (wordmarks, badge text, certification stamps, watermarks) → keep in the asset. It cannot be translated or surface-tone-adapted; it's a fixed visual identity.

When in doubt: if the text could plausibly be translated to another language for the same product, it's component-rendered. If translating it would corrupt the brand identity, it's brand/artwork text.

## Step 3 — Choose the format

| Asset type | Format | Notes |
|---|---|---|
| Vector illustration, logo, custom icon, custom marker | **SVG** | First choice for anything that started life as vector in Figma. Tiny, sharp at any size, CSS-recolorable via `currentColor` or CSS variables. |
| Photograph (real-world subject, gradient-rich, no hard text edges) | **JPG** | Use for natural photography only. Export at 2× the largest displayed size. |
| Raster with sharp edges, screenshots of UI, pixel-art, or anything with text/lines that JPG would smudge | **PNG** | Lossless. Use for newsletter teasers, CTA backgrounds, component previews, app screenshots. Export at 2×. |
| Third-party logo, award badge, or partner certificate supplied as bitmap (no vector source available) | **PNG** | Lossless, preserves text crispness and alpha channel for sticker/cutout shapes. Export at 2×. SVG would be preferable if the source is vector — but for partner-supplied raster artwork, PNG is the realistic choice. |
| Animated decorative element that must move | **WebM** | Only when animation is essential and SVG/CSS can't do it. See the animated icons shipped under `@laioutr-core/ui`'s public dir (e.g. `/icons/animated/party-popper.webm`). |
| Default | **PNG** | If unsure, PNG. |

**Hard rules:**
- **Never WebP.** Laioutr's toolchain and themes are calibrated for SVG/PNG/JPG only. Don't introduce WebP in your module.
- **Never base64 / data URIs in code.** Always a file in `runtime/public/` referenced by a root-relative path string. Base64 defeats browser caching, inflates bundle size, and prevents the asset being themed.
- **Detect raster vs vector from metadata, not screenshots.** A high-res raster and the original vector look identical in `get_screenshot`. Read the leaf's element tag in the `get_metadata` XML dump (the MCP renders node types as XML elements, not as a `type=` attribute):
  - `<rectangle>`, `<rounded-rectangle>`, `<ellipse>` → raster source. PNG or JPG only; SVG is not an option (the bytes are already pixels).
  - `<vector>`, `<boolean-operation>`, `<star>`, `<polygon>`, or a `<frame>`/`<group>` composed of these → vector source. Try SVG first.
  - `<text>` → not an asset; component-rendered.

  `get_metadata` doesn't expose paint fills, so you can't distinguish a rectangle-as-image from a rectangle-as-decorative-shape from the XML alone. Use `get_screenshot(nodeId, contentsOnly: true)` to confirm there's actual image content before exporting; an empty rectangle with a solid colour is a styling artefact, not an asset. When a leaf is a group mixing vector children with a `<rectangle>` carrying image content (e.g., a logo lockup laid over a photo), the asset is mixed-source — export raster (PNG @ 2×) since SVG can't faithfully embed the bitmap part.

**Scale selection isn't free.** "Export at 2×" is the right default for raster web assets — Retina/high-DPI displays render at 2× device pixels per CSS pixel, and 1× exports look fuzzy when sized for Retina. But scale=2 is wrong in three specific cases:

- **Source bitmap is smaller than the canvas displays it.** A partner supplied a 200×200 PNG that Figma renders at 400×400 on the canvas; asking for `scale=2` returns an 800×800 file where Figma upscaled an already-fuzzy 200×200 source twice. Bigger file, no extra sharpness. The skill can't detect this from `get_metadata` alone, but `get_design_context` may surface the source bitmap's intrinsic width/height. When in doubt, fetch both `scale=1` and `scale=2` and compare file sizes — if they're suspiciously close, Figma was upscaling and `scale=1` is the honest answer.
- **Pixel-art or decorative assets that won't display large.** A 24×24 status icon doesn't need 96 device pixels; `scale=1` saves bandwidth without quality cost.
- **Hero-size assets on the latest 3× phones** (iPhone Pro line). Above ~600 CSS pixels of displayed width, the gap between 2× and 3× becomes visible. Use `scale=3` knowingly, not as a default — it triples bytes.

For everything else (badges, logos, photos, decorative imagery sized in the typical 50–500 CSS-pixel range): `scale=2`.

## Step 4 — Choose the destination directory

The asset lives in your module's public directory. Pick the segment that matches where the consuming component lives in your module:

| Consumer file location | Public dir |
|---|---|
| `src/runtime/components/...` (presentational component) | `src/runtime/public/` |
| `src/runtime/app/section/...` or `src/runtime/app/block/...` (Section/Block wrapper) | `src/runtime/app/public/` |

Run `ls src/runtime/` and `ls src/runtime/app/` in your module if unsure — whichever segment exists with a `public/` subdirectory is the one to use. If the asset is consumed by both presentational components and Section/Block wrappers, place it under the higher-level path (`src/runtime/public/`) where both can reference it.

**Production assets vs. Storybook-only fixtures.** Assets that ship with the component (default content, theme images, component previews, real consumer-facing artwork) go in your module's `runtime/public/` (or `runtime/app/public/`) so they're served at runtime. Assets that exist *only* as Storybook story fixtures (e.g. demo payment-logo placeholders that show up only inside `.stories.ts`, never used by the component itself) can live in your local Storybook's public dir instead (`.storybook/public/<feature>/`) — that path is served only inside the Storybook dev server and is not published with the module. When in doubt, prefer `runtime/public/` — if the component ever needs a default value pointing at the asset, it must be runtime-served.

## Step 5 — Choose the path inside `public/`

Group by feature/component, not by format. The leading `/` in the runtime URL maps to the module's public root.

| Pattern | Used for | Example |
|---|---|---|
| `/<feature>/<filename>` | Per-component or per-feature directory under a presentational component's `public/` | `/cta/cta-banner-bg.png`, `/brand-hero/brand-header-light-desktop-laioutr.svg` |
| `/img/<feature>/<filename>` | Generic image library grouped under `img/` (mirrors upstream `@laioutr-core/ui-kit`'s convention) | `/img/banner/showcase-laioutr-light-desktop.svg`, `/img/placeholder/1x1-product.png` |
| `/app-ui/component-previews/<ComponentName>.png` | Studio component-picker preview screenshots under `src/runtime/app/public/` | `/app-ui/component-previews/SectionFooter.png` |
| `/<feature>/<filename>` (under `src/runtime/app/public/`) | Per-section / per-block feature directory for assets that are not previews | `/awards/holidaycheck-2026-award.png`, `/hero/background-desktop.jpg` |
| `/placeholders/<aspect>.{jpg,png,svg}` | Fixture placeholders for stories | `/placeholders/16x9.jpg`, `/placeholders/1x1.svg` |
| `/logo/<filename>` | Brand/partner logos | `/logo/logo-light.svg` |

## Step 6 — Choose the filename

**kebab-case, lowercase, content-describing.** No camelCase, no timestamps, no node IDs.

Suffix order for variant axes — **only when the asset actually varies along one or more axes**, in this order:
```
<prefix>-<theme>-<light|dark>-<mobile|desktop>.<ext>
```

If the asset has zero variant axes (a single artifact with no theme/mode/breakpoint variations — a partner logo, a badge, a one-off illustration), use only the content-describing slug. No filler suffixes.

Examples:
- `showcase-laioutr-light-desktop.svg` (theme × colour-mode × breakpoint)
- `logo-dark.svg` (colour-mode only)
- `desktop.png` inside a `newsletter/` folder (breakpoint only)
- `1x1-product.png` (aspect-ratio + content tag)
- `holidaycheck-2026-award.png` (no axes — brand-named single artifact)
- `vdfu.png` (no axes — partner wordmark)

For component-preview screenshots specifically: PascalCase matching the section/block component name — `SectionFooter.png`, `BlockButton.png`. This is the one place where kebab-case is *not* used; it matches the Vue component's class name.

Never use:
- `Bildschirmfoto 2026-…`, `Screenshot at …`, `image-1`, `image_copy_2`
- The Figma node ID as the filename (`1-5177.svg`)
- `Logo.svg`, `Icon.svg`, `Background.svg` — too generic, will collide

## Step 7 — Export the bytes

Decide which path applies. The deliverable in each case is something the developer can act on without having to translate node IDs back into Figma UI actions on their own.

### Path B — Figma REST API (preferred, regardless of asset count)

If `FIGMA_ACCESS_TOKEN` is set in the environment, skip to the script below.

If it isn't set, **do not silently fall through to path A** — guide the user through creating one first. Hand them this exact set of instructions:

> To export the assets via the REST API, the agent needs a Figma Personal Access Token. Creation takes ~30 seconds and works on every Figma plan including the free Starter tier. The token stays on your machine; nothing is shared.
>
> 1. Open <https://www.figma.com/settings> in a browser (or click your avatar in the Figma app → **Settings**).
> 2. In the left sidebar, click **Security** (older Figma versions: **Account**).
> 3. Scroll to **Personal access tokens** → **Generate new token**.
> 4. Name it something like `claude-code-export` and set the **File content** scope to **Read-only**. Other scopes can stay default. Click **Generate**.
> 5. **Copy the token immediately** — Figma only shows it once.
> 6. Then either:
>    - Export it for the current shell: `export FIGMA_ACCESS_TOKEN="figd_…"`. Re-run the agent.
>    - Or paste it inline in the chat: the agent uses it for this session only and does not persist it.

Wait for the user to confirm before continuing. Don't proceed without a token; producing a script that silently fails on auth is worse than stopping.

Once the token is available, generate a Node script like this for the batch. One `/v1/images` call per format group (the endpoint takes a single `format=` query param). Node ≥ 18 is required for built-in `fetch`; the Laioutr toolchain already mandates a newer Node, so this is safe to assume.

On the Figma free Starter plan, this counts as one of your six monthly file-content requests per file — comfortable for a single export pass, tight if you iterate against the same file repeatedly.

```js
#!/usr/bin/env node
import fs from 'node:fs';
import path from 'node:path';

const TOKEN = process.env.FIGMA_ACCESS_TOKEN;
if (!TOKEN) { console.error('Set FIGMA_ACCESS_TOKEN before running.'); process.exit(1); }

const FILE_KEY = '<fileKey-from-URL>';
const TARGET   = '<module-root>/src/runtime/public/location-finder';

const GROUPS = {
  svg: { scale: 1, nodes: { '1:5177': 'marker-pinlet.svg', '1:5421': 'marker-current-location.svg' } },
  png: { scale: 2, nodes: { '1:5176': 'map-placeholder.png' } },
};

fs.mkdirSync(TARGET, { recursive: true });
for (const [format, { scale, nodes }] of Object.entries(GROUPS)) {
  const ids = Object.keys(nodes).join(',');
  const res = await fetch(
    `https://api.figma.com/v1/images/${FILE_KEY}?ids=${ids}&format=${format}&scale=${scale}`,
    { headers: { 'X-Figma-Token': TOKEN } },
  );
  const json = await res.json();
  if (json.err) { console.error(`Figma API error for ${format}: ${json.err}`); process.exit(1); }
  for (const [id, url] of Object.entries(json.images || {})) {
    if (!url) { console.error(`No URL returned for node ${id} (format=${format})`); continue; }
    const buf = Buffer.from(await (await fetch(url)).arrayBuffer());
    fs.writeFileSync(path.join(TARGET, nodes[id]), buf);
    console.log(`wrote ${path.join(TARGET, nodes[id])}`);
  }
}
```

**Save with the `.mjs` extension** so Node treats the file as an ES module — the `import` statements and top-level `await` require ESM. (`.js` works too if the surrounding package has `"type": "module"` in its `package.json`, but `.mjs` is unambiguous and survives outside any project. Don't use the CommonJS `require` form here; `.mjs` files reject it.) Then run `FIGMA_ACCESS_TOKEN=figd_… node path/to/script.mjs`. Verify with the block below. Done.

**Surface the plan before running.** The script bakes in decisions the developer can't audit by reading code at a glance — scale, target directory, filename map, asset count. Before invoking the script, present the plan to the user in this compact shape:

```
I'll export to /<target dir>/:
  N <format>s at scale=<K> (<one-line rationale, e.g. "Retina default for award badges">)
  <node-id> → <filename>   [<one-line visible content from screenshot>]
  <node-id> → <filename>   [<one-line visible content from screenshot>]
  …
Proceed, or override scale / drop assets / rename anything?
```

**The content hint is load-bearing, not decorative.** Filenames are agent inventions made by reading the screenshot — for badges, logos, partner artwork, the agent has to interpret visible wordmarks into a kebab-case slug. The dev can't audit that interpretation from the filename alone (`tourismuspreis.png` vs `holidaycheck-gold-award.png` look equally plausible until you see which badge was actually on screen). The hint must contain enough of the visible text/content to let the dev spot a misread in one glance — the badge's wordmark, the partner name, the certification year, whatever is most distinctive. Skipping the hint defeats the whole point of pre-flight surfacing.

Also flag two state items inline when relevant:
- *"target dir doesn't exist yet — will be created"* if `mkdir -p` would create a new tree (catches typos in the target path before bytes silently land in a junk directory)
- *"this run uses N of your remaining ~~6~~ monthly file-content requests"* if the dev is on the Figma Starter plan and the count is non-trivial (≥ 3 of 6)

Accept silent confirmation as proceed. The dev usually doesn't have a strong opinion on scale or filenames, but they're the only one with display-context info — give them the chance to override before bytes land. Don't ask each decision separately; surface them as one plan with sensible defaults.

### Path A — Manual export by the user (≤2 assets, or user declined the token)

The spec must do three things the original "list of node IDs and target paths" didn't: give the user a way to navigate to each node (Figma's UI has no ID search), tell them what filename Figma will write by default (Figma names exports by the node's `name` attribute, not by the kebab-case target name), and give them a copy-paste rename script for after.

```
File:   <Figma file URL>
Target: <module-root>/src/runtime/public/<feature>/

Nodes to export (click each link → node selects in the active Figma tab → File → Export Selection):

  1:5177 → https://www.figma.com/design/<fileKey>/<filename>?node-id=1-5177
           Figma exports as:  "marker variant 1.svg"
           Rename to:         marker-pinlet.svg
           Settings:          SVG, 1×

  1:5176 → https://www.figma.com/design/<fileKey>/<filename>?node-id=1-5176
           Figma exports as:  "image 19.png"
           Rename to:         map-placeholder.png
           Settings:          PNG, 2×

Post-export rename (run in the directory Figma saved into):
  mv "marker variant 1.svg" <module-root>/src/runtime/public/location-finder/marker-pinlet.svg
  mv "image 19.png"         <module-root>/src/runtime/public/location-finder/map-placeholder.png
```

Building the spec correctly:

- The deep link reuses the source URL's `<fileKey>` and replaces the colon in the node ID with a hyphen (`1:5177` → `node-id=1-5177`). Clicking the link in any browser focuses the active Figma desktop tab on that node, selected.
- "Figma exports as" is the leaf's `name` attribute from `get_metadata` — that's what Figma writes to disk by default. If `get_metadata` shows the leaf as unnamed or with a generic name like `<rectangle>`, surface that in the spec so the user knows there's no useful default and they should rename in Figma first.
- The `mv` script's source paths assume the user picked a single export directory for the batch (Figma's default workflow when multiple nodes are selected and exported together). If the user exports nodes one at a time, they pick a folder per export; the script still works as long as they `cd` to wherever they saved each file.

### Path C — Stop and report

If Step 0 failed (wrong tab, no Dev Mode, Community file requiring duplication), the deliverable is a written report: what was asked, what's blocking, what the spec *would* be once access is restored. No fabricated node IDs, no placeholder files.

### Verification (after bytes land, paths A or B)

```bash
file src/runtime/public/location-finder/marker-pinlet.svg
# should report "SVG Scalable Vector Graphics image"
```

For SVGs, also `grep -l '<text' src/runtime/public/**/marker-*.svg` — non-empty output means text leaked into the export, go back to Step 2.

## Step 8 — Wire up the component

Outside this skill's scope, but the immediate next step: the component references the asset via a root-relative URL string (`<img src="/location-finder/marker-pinlet.svg" />` or via `<Media :media="toMedia('/...')" />` in stories). Don't import the file path as a module; the runtime/public mechanism serves it as a static asset.

## Common mistakes

| Mistake | Fix |
|---|---|
| Exporting a wrapper frame with text + image | Drill to the leaf image node. Text is component-rendered, not asset-baked. |
| Choosing WebP for "better compression" | Laioutr's toolchain is calibrated for SVG / PNG / JPG only — don't introduce WebP. |
| Inlining as base64 in a `.vue` file | Always write to `runtime/public/` and reference via a `/...` path. |
| Re-exporting an icon that's already in `IconName` | Look up the `IconName` union in `@laioutr-core/ui-kit` (editor go-to-definition, or query the docs MCP) and reuse the key. |
| Adding a new SVG to the upstream ui-kit theme icon set to make a story work | Theme icons are design-owned. Raise the gap with design instead — or follow the resolution ladder (prop override → local custom component → fork `laioutr/ui-source` → upstream issue) if you need to land it locally. |
| Saving 1× on a non-vector asset | PNG/JPG should be exported at 2× and let the browser downscale. SVG is scale-independent. |
| Filename like `Bildschirmfoto 2026-03-12.png` or `1-5176.png` | Describe the content: `map-placeholder.png`, `cta-banner-bg.png`. |
| Putting the file in `runtime/public/` when the role uses `runtime/app/public/` (or vice versa) | Check `ls src/runtime/` in your module before deciding. |
| Placing a *production* asset in your Storybook's public dir instead of the module's `runtime/public/` | Storybook serves the module's `runtime/public/` already; assets the component itself references must live there or they won't be served at runtime. Storybook-only story fixtures (never referenced by the component) can stay in the Storybook public dir. |
| Writing a placeholder file because the MCP can't export bytes | Don't. Stop with a clear export spec and let the user / a REST script land the real bytes. |

## Pre-export checklist

- [ ] Verified the active Figma tab matches the requested file (Step 0, error mode 1)
- [ ] Verified the MCP can actually read the file (Step 0, error modes 2-3) — if Dev Mode access is blocked, the only correct output is a written report; stop here
- [ ] Used `get_metadata` to see the frame structure
- [ ] Identified candidate leaf nodes (not wrapper frames)
- [ ] Verified with `get_screenshot(contentsOnly: true)` that each candidate is text-free and chrome-free
- [ ] Cross-checked against `IconName` union and existing `public/` dirs to eliminate duplicates
- [ ] Picked format per Step 3 (no WebP, no base64)
- [ ] Picked destination per Step 4 (verified `runtime/public/` vs `runtime/app/public/` segment)
- [ ] Picked path per Step 5 (feature-grouped, kebab-case)
- [ ] Picked filename per Step 6 (no node IDs, no timestamps, no generic names)
- [ ] Produced an export spec the user can act on, OR ran a REST API export if a token exists

## Related skills

- `figma-design-analysis` — read this skill's "Themable Default Images" section if the asset is a backdrop, placeholder, or empty-state illustration that should be registered in `ImageSet` instead.
- `figma-to-component` — wiring up the component that consumes the exported asset.
