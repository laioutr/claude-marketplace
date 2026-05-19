# Reference Lookups for Laioutr Refactors

**Before deleting or renaming an exported symbol, file, or `#imports`-exposed helper, use LSP `findReferences` — not grep — to find usages.** Grep misses template usages, dynamic references, and type-only imports.

## Why

Laioutr code relies heavily on Nuxt auto-imports through `#imports`, `#frontend-core`, `#ui-kit/...`, and `#ui/...` — these resolve at build time and don't appear in source as literal import statements. Grep for the symbol name finds the *definition* and explicit imports, but misses every auto-import call site.

Fall back to grep only when the LSP returns no results (some `#imports` chains the LSP doesn't follow) — and treat that fallback as "double-check," not "primary."
