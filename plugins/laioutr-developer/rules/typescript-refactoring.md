# TypeScript Refactoring

## Reference lookups (do this first)

**Prefer LSP `findReferences` over Grep / ripgrep for finding symbol references.** Grep misses template usages, dynamic references, and type-only imports — making it unreliable for refactor planning.

Before deleting a symbol, file, or export — or when planning a migration — call LSP `findReferences` on the symbol to find all usages. Fall back to grep only if the LSP returns no results (e.g. cross-package auto-imports through `#imports` in Nuxt, which the LSP may not follow).

This applies to:

- Checking if a symbol, file, or export is used before deleting it
- Finding all references to plan a migration or rename
- Verifying that a removed export has no consumers

If your editor / tooling does not expose `findReferences` as a callable tool, use whatever your IDE's "Find All References" command surfaces — the principle is to use a typed reference index, not a text grep.

## Code transformations

When performing bulk TypeScript / JavaScript refactoring operations, prefer typed AST-based tooling over manual edits or `sed` / `awk` style scripts. In rough order of preference (use whatever is available in your setup):

1. **A TypeScript-aware LSP refactor tool** (e.g. a `typescript-lsp` plugin in Claude / your IDE) — fastest path when available, since it reuses the editor's project graph.
2. **An AST refactor MCP / library** such as `mcp-refactor-typescript` if you have it wired in.
3. **`ast-grep`** for pattern-driven transforms across many files.
4. **`ts-morph`** for full programmatic rewrites with type awareness — the most powerful but heaviest option.

You don't need all four — pick the lightest one that handles the task.

When to reach for AST tooling:

- Renaming imports across multiple files
- Updating import paths (e.g. after moving files)
- Deleting files that are no longer used (after a `findReferences` check)
- Adding or removing imports programmatically
- Refactoring function signatures
- Renaming variables, functions, or classes across files
- Any AST-based bulk transformation

## ts-morph: temporary script pattern

For one-off transformations using `ts-morph`, isolate the script so it doesn't pollute your module's dependency tree:

```
your-project/scripts/refactor-task/
├── package.json   (ts-morph is the only dependency)
└── index.ts       (run with `pnpm install && npx tsx index.ts`,
                    or `npm install && npx tsx index.ts`)
```

Remove the temporary script folder after the refactor is complete and committed.

## Why AST tooling beats manual editing

- Consistent transformations across every match
- No risk of missing occurrences hidden in templates or dynamic imports
- Proper AST manipulation (won't break syntax)
- Much faster than file-by-file edits for bulk operations
- Handles scope-aware renames (e.g. won't touch a same-named local in another scope)
