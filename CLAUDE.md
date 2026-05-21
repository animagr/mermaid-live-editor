# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Mermaid Live Editor — a SvelteKit app for editing, previewing, and sharing Mermaid diagrams in real time. Live at https://mermaid.live/. Built with Svelte 5, SvelteKit 2, Vite 7, and Tailwind CSS 4.

## Commands

```bash
pnpm install            # Install dependencies (requires pnpm 10.10+, Node >= 20.19)
pnpm dev                # Dev server on http://localhost:3000
pnpm build              # Production build (outputs to docs/)
pnpm lint               # Check formatting (Prettier) + linting (ESLint)
pnpm lint:fix           # Auto-fix lint/format issues
pnpm test:unit          # Run Vitest unit tests (watch mode)
pnpm test:unit --run    # Run unit tests once
pnpm test:e2e           # Run Playwright E2E tests (needs dev server running)
vitest run src/lib/util/serde.ts  # Run a single unit test file
```
  ## Install setup


cd \mermaid-live-editor
Node.js
npm install -g pnpm@10.30.3
pnpm install
pnpm dev -- --host localhost

Open http://localhost:3000. That's it — it uses the published mermaid package from npm out of the box.

## Architecture

**Static SPA**: SSR is disabled (`ssr: false` in `src/routes/+layout.ts`). The app prerenders to static files via `@sveltejs/adapter-static` into `docs/`. All routing is client-side.

**Two routes**: `/edit` (main editor with Monaco, config panel, actions, history) and `/view` (read-only diagram viewer). Root `/` redirects to one of these.

**State management** (`src/lib/util/state.ts`): `inputStateStore` is a writable Svelte store persisted to localStorage. `stateStore` is a derived store that validates/parses the diagram. Diagram state is serialized into the URL hash for sharing — the hash is the single source of truth for shared diagrams.

**Path alias**: `$/*` maps to `src/lib/*` (configured in `svelte.config.js`). All internal imports use `$/components/...`, `$/util/...`, etc.

**Diagram rendering**: Mermaid 11 with layout engines (ELK, tidy-tree) and ZenUML plugin. Rendering happens client-side; the `View` component renders SVG with optional pan/zoom and rough-sketch mode.

**Editors**: Monaco Editor for code editing, CodeMirror 6 for config/syntax highlighting.

**UI components**: `src/lib/components/ui/` contains shadcn-svelte primitives (via bits-ui). These have relaxed ESLint rules — don't add sort-keys or unicorn rules there.

**Icons**: Custom icons in `static/icons/` loaded via `unplugin-icons`. Import as `~icons/custom/<name>` or from icon collections like `~icons/material-symbols/<name>`.

**Environment variables**: Prefixed with `MERMAID_` (Vite envPrefix). Configured in `.env`, override locally via `.env.local`. Key vars: `MERMAID_RENDERER_URL`, `MERMAID_KROKI_RENDERER_URL`, `MERMAID_ANALYTICS_URL`, `MERMAID_DOMAIN`, `MERMAID_BASE_PATH`.

## Code Conventions

- **Prettier**: 100 char width, single quotes, no trailing commas, `bracketSameLine: true`. Svelte file order: options → scripts → markup → styles.
- **ESLint**: Flat config with typescript-eslint strict/stylistic, svelte, unicorn. Object keys in `src/` must be sorted (ascending, objects with 5+ keys) via `sort-keys/sort-keys-fix`.
- **HMR is disabled**: Vite is configured for full page reload on every change (state inconsistencies with HMR).
- **Pre-commit hook**: Husky + lint-staged runs Prettier and ESLint on staged `.ts`, `.svelte`, `.js`, `.css`, `.md`, `.json` files.

## Testing

- **Unit tests**: Vitest with jsdom environment. Uses in-source testing (`includeSource: ['src/**/*.{js,ts,svelte}']`) — tests can live alongside source code. Setup file at `src/tests/setup.ts`.
- **E2E tests**: Playwright (Chromium) in `tests/` directory. Tests run against `http://localhost:3000` with 1920×1080 viewport.
