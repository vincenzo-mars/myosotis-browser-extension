# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev            # dev server with HMR via CRXJS
npm run build          # production build → dist/
npm run test           # vitest watch mode
npm run test:run       # vitest single run
npm run test:coverage  # vitest with v8 coverage
npm run typecheck      # tsc --noEmit
npm run lint           # biome check
npm run lint:fix       # biome check --write
```

To run a single test file: `npx vitest run src/path/to/file.test.tsx`

## Architecture

Manifest V3 Chrome extension. All entry points are declared in `manifest.json` and resolved automatically by CRXJS — no manual Vite input configuration needed.

| Entry point | Type | Notes |
|---|---|---|
| `src/popup/` | React SPA | Toolbar popup |
| `src/options/` | React SPA | Options page |
| `src/background/index.ts` | Service worker | No DOM access |
| `src/content/index.ts` | Content script | Injected into all URLs |

To add a new entry point (e.g. `newtab`), declare it in `manifest.json`. CRXJS picks it up automatically.

## Critical constraints

**Vitest cannot use `vite.config.ts`** — CRXJS is incompatible with jsdom. Tests run via `vitest.config.ts`, which uses only `@vitejs/plugin-react` and jsdom. Never merge the two configs.

**Tailwind does not apply in the content script** — `globals.css` is imported only by the popup and options entry points. To inject styled UI into pages, use Shadow DOM and declare resources under `web_accessible_resources` in `manifest.json`.

**`export {}`** in `src/background/index.ts` and `src/content/index.ts` is required — it makes those files ES modules so `@types/chrome` globals resolve correctly. Do not remove it.

## Tooling

- **Biome** replaces ESLint and Prettier. Style: double quotes, semicolons, trailing commas, 2-space indent, 100-char line width.
- **Commitlint** enforces Conventional Commits (`feat:`, `fix:`, `chore:`, etc.).
- **Tailwind v4** is configured via the `@tailwindcss/vite` plugin — no `tailwind.config.js` or PostCSS config exists or is needed. CSS entry is a single `@import "tailwindcss"` in `src/styles/globals.css`.
