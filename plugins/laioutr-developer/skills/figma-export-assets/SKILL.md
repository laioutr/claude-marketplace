---
name: figma-export-assets
description: Use when implementing a component from a Figma frame that contains image content (illustrations, photos, logos, markers, screenshots, custom icons) that needs to live as files in the laioutr repo. Excludes themable defaults registered via themeMedia/themeResponsiveMedia — that workflow belongs to figma-design-analysis.
---

# Figma → Repo Asset Export

## What this is for

Getting visual assets out of a Figma frame and into the right place in the laioutr monorepo, so a Vue/TS component can reference them via a root-relative URL string (`<img src="/cta/cta-banner-bg.png" />`).

In scope:
- **Custom icons** that are not already in the `IconName` union (`packages/ui-kit/src/runtime/app/types/theme.ts`)
- **One-off raster images** used by a single section/block/composition (decorative photo, screenshot mock, marker, illustration)
- **Brand/partner logos** (payment methods, integrations, third-party badges)

Out of scope (use a different workflow):
- **Themable defaults** registered in `ImageSet` via `themeMedia()` / `themeResponsiveMedia()` — those are theme-system assets, see the `figma-design-analysis` skill.
- **Existing theme icons** in the `IconName` union — reuse the existing key; do not export.

## The MCP can't write asset bytes (read this first)

The `mcp__figma__*` tools (`get_design_context`, `get_screenshot`, `get_metadata`, `get_variable_defs`) return **inline data the agent can view, not files the agent can save**. There is no `mcp__figma__export_asset` tool.

Three real paths to get bytes onto disk:

1. **Manual export by the user** (default, most reliable). The agent identifies node IDs + format + scale + target paths; the user runs File → Export Selection in Figma. The skill's job is to produce that spec precisely enough that "export" is a one-click operation.
2. **Figma REST API**. `https://api.figma.com/v1/images/{fileKey}?ids={nodeId}&format={svg|png|jpg}&scale={1|2|3}` returns a temporary CDN URL → `curl` to download → write to the path. Requires a `FIGMA_ACCESS_TOKEN` env var. Only use this when the token is already configured; do not try to obtain one.
3. **Verification-only via screenshot**. When neither (1) nor (2) is available, `mcp__figma__get_screenshot` with `contentsOnly: true` produces an inline image the agent can visually verify is the right asset, and the agent hands the export spec to the user as a deliverable.

**Never substitute** a base64 data URI in code for missing bytes, and **never** write a placeholder file with arbitrary content "to be replaced later" — a missing file is a louder signal than a wrong file.

## Step 0 — Verify MCP access to the requested file

The `mcp__figma__*` tools only operate against the browser tab the user currently has focused, and only when the user's Figma seat has Dev Mode access to that file. Both conditions can fail silently. Before any export work, probe both:

```
get_metadata()              # no nodeId — returns the currently selected node or the top-level page list
get_metadata(nodeId)        # the specific node you were asked about
```

Three distinct error modes to recognize and handle separately:

1. **"No node could be found"** — active tab is on a different file than the URL you were given. Stop and tell the user *"the active Figma tab is on file X / node Y; please switch to file Z for this task."* Wait for them to switch tabs. Do NOT silently work against whatever node *is* selected.

2. **"This resource couldn't be accessed" / Dev Mode messaging** — active tab is correct, but the seat/file permissions block the MCP. Stop and tell the user the file needs Dev Mode access from their seat. Tab switching will not help; only an access change will. If the URL contains `Community` in the slug (e.g. `…--Community-?node-id=…`), it's a Figma Community file and the most common fix is *"duplicate this file into your own workspace, then switch the tab to the duplicate"* — Community files often expose Dev Mode only on the duplicate, not the original.

3. **Empty / no response** — same as (2) operationally; stop and report.

In all three cases, the correct terminal state is **stop, produce a written report, do not fabricate**. Never invent node IDs, frame names, or export specs based on a guess about what the asked-for file might contain. A missing export is a louder signal than a wrong one.

This step exists because the failure modes are silent: the MCP doesn't error loudly when the wrong tab is active or the file is inaccessible, it just returns "no node" or a generic access error that's easy to misread as "the agent should try harder."

## Step 1 — Inventory

Before touching the export tool, list what the frame contains and decide per-element whether it ships as a file.

```
get_metadata(nodeId)        — structure of the frame
get_screenshot(nodeId)      — what each candidate node looks like
```

For each visual element, classify:

| Element | Action |
|---|---|
| Text node (heading, body copy, label, caption, button text) | **Skip.** Component renders this via `t('key')` — never bake into an exported asset. |
| Icon that matches a key in `IconName` (`packages/ui-kit/src/runtime/app/types/theme.ts`) | **Skip.** Reuse the existing key. |
| Existing public asset (search `packages/{ui-kit,ui,ui-app}/src/runtime/**/public/`) | **Skip.** Reference the existing path. |
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
| Animated decorative element that must move | **WebM** | Only when animation is essential and SVG/CSS can't do it. See `packages/ui/src/runtime/public/icons/animated/party-popper.webm`. |
| Default | **PNG** | If unsure, PNG. |

**Hard rules:**
- **Never WebP.** Zero WebP files exist in the repo. The toolchain and themes are calibrated for SVG/PNG/JPG only.
- **Never base64 / data URIs in code.** Always a file in `runtime/public/` referenced by a root-relative path string. Base64 defeats browser caching, inflates bundle size, and prevents the asset being themed.
- **Prefer SVG over raster** whenever the source is vector. If you screenshot the node and it shows hard-edged geometric shapes or text-shaped paths, try SVG first.

## Step 4 — Choose the destination package

The asset lives in the same package as its consumer.

| Consumer | Package | Public dir |
|---|---|---|
| Atomic ui-kit primitive | `packages/ui-kit/` | `src/runtime/app/public/` |
| Commerce composition (ui) | `packages/ui/` | `src/runtime/public/` ← no `app/` segment |
| Section / Block (ui-app) | `packages/ui-app/` | `src/runtime/app/public/` |
| First-party app integration (shopify, ambiendo, …) | `packages/app-<name>/` | `src/runtime/app/public/` |

**Verify the segment.** `ui` is `runtime/public/`, the others are `runtime/app/public/`. Run `ls packages/<pkg>/src/runtime/` if unsure — whichever segment exists is the one to use.

If the asset has more than one likely consumer across packages, place it in the lowest-level package they share (usually `ui-kit`) and reference it from there.

**Production assets vs. Storybook-only fixtures.** Assets that ship with the component (default content, theme images, component previews, real consumer-facing artwork) go in the module's `runtime/public/` (or `runtime/app/public/`) so they're served by the module at runtime. Assets that exist *only* as Storybook story fixtures (e.g. demo payment-logo placeholders that show up only inside `.stories.ts`, never used by the component itself) can live in `packages/<pkg>/playground/.storybook/public/<feature>/` instead — that path is served only inside the Storybook dev server and is not published with the package. When in doubt, prefer `runtime/public/` — if the component ever needs a default value pointing at the asset, it must be runtime-served.

## Step 5 — Choose the path inside `public/`

Group by feature/component, not by format. The leading `/` in the runtime URL maps to the package's public root.

| Pattern | Used for | Example |
|---|---|---|
| `/<feature>/<filename>` (ui) | Per-component or per-feature directory | `/cta/cta-banner-bg.png`, `/brand-hero/brand-header-light-desktop-laioutr.svg` |
| `/img/<feature>/<filename>` (ui-kit) | Atomic-package assets grouped under `img/` | `/img/banner/showcase-laioutr-light-desktop.svg`, `/img/placeholder/1x1-product.png` |
| `/app-ui/component-previews/<ComponentName>.png` (ui-app) | Cockpit component-picker preview screenshots | `/app-ui/component-previews/SectionFooter.png` |
| `/<feature>/<filename>` (ui-app) | Per-section / per-block feature directory for assets that are not previews | `/awards/holidaycheck-2026-award.png`, `/hero/background-desktop.jpg` |
| `/placeholders/<aspect>.{jpg,png,svg}` (ui) | Fixture placeholders for stories | `/placeholders/16x9.jpg`, `/placeholders/1x1.svg` |
| `/logo/<filename>` (ui) | Brand/partner logos | `/logo/logo-light.svg` |

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

Decide which of the three paths in "The MCP can't write asset bytes" applies. Produce an export spec like:

```
File:   <Figma file URL>
Nodes:  1:5177 → /Users/sl/src/laioutr/packages/ui/src/runtime/public/location-finder/marker-pinlet.svg (SVG, 1×)
        1:5421 → /Users/sl/src/laioutr/packages/ui/src/runtime/public/location-finder/marker-current-location.svg (SVG, 1×)
        1:5176 → /Users/sl/src/laioutr/packages/ui/src/runtime/public/location-finder/map-placeholder.png (PNG, 2×)
```

Then either:
- (A) Hand the spec to the user with explicit "select these nodes, File → Export" instructions, OR
- (B) Run a Figma REST API script if `FIGMA_ACCESS_TOKEN` is set, OR
- (C) Stop and report the spec as deliverable.

Verify after the bytes land:
```bash
file packages/ui/src/runtime/public/location-finder/marker-pinlet.svg
# should report "SVG Scalable Vector Graphics image"
```

For SVGs, also `grep -l '<text' packages/.../marker-*.svg` — non-empty output means text leaked into the export, go back to Step 2.

## Step 8 — Wire up the component

Outside this skill's scope, but the immediate next step: the component references the asset via a root-relative URL string (`<img src="/location-finder/marker-pinlet.svg" />` or via `<Media :media="toMedia('/...')" />` in stories). Don't import the file path as a module; the runtime/public mechanism serves it as a static asset.

## Common mistakes

| Mistake | Fix |
|---|---|
| Exporting a wrapper frame with text + image | Drill to the leaf image node. Text is component-rendered, not asset-baked. |
| Choosing WebP for "better compression" | The repo has zero WebP. Use SVG / PNG / JPG. |
| Inlining as base64 in a `.vue` file | Always write to `runtime/public/` and reference via a `/...` path. |
| Re-exporting an icon that's already in `IconName` | Search the union first (`grep '|' packages/ui-kit/src/runtime/app/types/theme.ts`). Reuse the key. |
| Adding a new SVG to `ui-kit/.../assets/icons/` to make a story work | Theme icons are design-owned. Raise the gap with design instead. |
| Saving 1× on a non-vector asset | PNG/JPG should be exported at 2× and let the browser downscale. SVG is scale-independent. |
| Filename like `Bildschirmfoto 2026-03-12.png` or `1-5176.png` | Describe the content: `map-placeholder.png`, `cta-banner-bg.png`. |
| Putting the file in `runtime/public/` for a package that uses `runtime/app/public/` (or vice versa) | Check `ls packages/<pkg>/src/runtime/` before deciding. |
| Placing a *production* asset in `playground/.storybook/public/` instead of the module's `runtime/public/` | Storybook serves the module's `runtime/public/` already; assets the component itself references must live there or they won't be served at runtime. Storybook-only story fixtures (never referenced by the component) can stay in `playground/.storybook/public/`. |
| Writing a placeholder file because the MCP can't export bytes | Don't. Stop with a clear export spec and let the user / a REST script land the real bytes. |

## Pre-export checklist

- [ ] Verified the active Figma tab matches the requested file (Step 0, error mode 1)
- [ ] Verified the MCP can actually read the file (Step 0, error modes 2-3) — if Dev Mode access is blocked, the only correct output is a written report; stop here
- [ ] Used `get_metadata` to see the frame structure
- [ ] Identified candidate leaf nodes (not wrapper frames)
- [ ] Verified with `get_screenshot(contentsOnly: true)` that each candidate is text-free and chrome-free
- [ ] Cross-checked against `IconName` union and existing `public/` dirs to eliminate duplicates
- [ ] Picked format per Step 3 (no WebP, no base64)
- [ ] Picked package per Step 4 (verified `runtime/public/` vs `runtime/app/public/` segment)
- [ ] Picked path per Step 5 (feature-grouped, kebab-case)
- [ ] Picked filename per Step 6 (no node IDs, no timestamps, no generic names)
- [ ] Produced an export spec the user can act on, OR ran a REST API export if a token exists

## Related skills

- `figma-design-analysis` — read this skill's "Themable Default Images" section if the asset is a backdrop, placeholder, or empty-state illustration that should be registered in `ImageSet` instead.
- `figma-to-component` — wiring up the component that consumes the exported asset.
