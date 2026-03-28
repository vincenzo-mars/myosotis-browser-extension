# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev            # dev server with HMR via CRXJS
npm run build          # production build → dist/
npm run typecheck      # tsc --noEmit
npm run lint           # biome check
npm run lint:fix       # biome check --write
```

## Architecture

Manifest V3 Chrome extension. All entry points are declared in `manifest.json` and resolved automatically by CRXJS — no manual Vite input configuration needed.

| Entry point | Type | Notes |
|---|---|---|
| `src/popup/` | React SPA | Toolbar popup |
| `src/options/` | React SPA | Options page |
| `src/background/index.ts` | Service worker | No DOM access |
| `src/content/index.ts` | Content script | Injected into all URLs |

`popup` and `options` are independent React SPAs — each has its own `index.html`, `index.tsx` (React root + StrictMode), and `App.tsx`. Both import `src/styles/globals.css`, which is a single `@import "tailwindcss"` line; Tailwind is configured entirely via the `@tailwindcss/vite` plugin.

To add a new entry point (e.g. `newtab`), declare it in `manifest.json`. CRXJS picks it up automatically.

## Critical constraints

**Tailwind does not apply in the content script** — `globals.css` is imported only by the popup and options entry points. To inject styled UI into pages, use Shadow DOM and declare resources under `web_accessible_resources` in `manifest.json`.

**`export {}`** in `src/background/index.ts` and `src/content/index.ts` is required — it makes those files ES modules so `@types/chrome` globals resolve correctly. Do not remove it.

## Tooling

- **Biome** replaces ESLint and Prettier. Style: double quotes, semicolons, trailing commas, 2-space indent, 100-char line width.
- **Commitlint + Commitizen** enforce Conventional Commits (`feat:`, `fix:`, `chore:`, etc.). Use `cz commit` for the interactive prompt; raw `git commit` is also enforced via the `commit-msg` Husky hook.
- **lint-staged** runs `biome check --write` on staged `*.ts`, `*.tsx`, `*.js`, `*.json` files before each commit.
- **semantic-release** is configured on `main` — merging to `main` triggers automated versioning and GitHub releases based on commit messages.
