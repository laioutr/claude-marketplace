---
description: "Compose and build a marketing-driven landing page in Laioutr — from existing sections/blocks, filled with data, one clear conversion action. Loads skills/compose-page and delegates to the ux-designer agent."
argument-hint: "<campaign / goal> (e.g. 'spring sale, newsletter sign-up', 'new sneaker collection, shop now')"
---

# `/landing-page` — compose and build a landing page

Thin wrapper around `skills/compose-page` with the landing-page recipe.

## Trigger

`/landing-page {campaign / goal, optional persona/topic}`

## Workflow

1. Load `skills/compose-page` + `memory/page-types/landing-page.md`.
2. Sharpen: the **one** conversion action, persona, campaign/topic context (from the user's brief).
3. Delegate to `ux-designer`: choose/order sections, blocks into slots, fill data + `query`.
4. Fill the page in Laioutr Studio (`~~platform`). It is a **static** page type; the Cockpit MCP cannot create pages, so if it does not exist yet, ask the user to create it in Studio first.
5. Deliver composition table + link + next step (`design-critique` before going live).

> **Output rule (mandatory):** The page lives on the platform. File artifacts (exports, screenshots, preview HTML, critique reports) go to `${DESIGN_MANAGER_OUTPUT_DIR:-./design-manager-output}/<YYYY-MM-DD>/` — never into the plugin directory and never into `.claude/design-manager/` (that folder holds curated project knowledge only).

## When not to use this command

- Home/PDP/PLP/Content/Blog → `compose-page` skill (general).
- Missing building block → `ui-designer` / `component-library-build`.
