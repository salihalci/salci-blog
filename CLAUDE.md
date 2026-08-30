# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A personal blog built on the **AstroPaper v6** theme (Astro + TypeScript + TailwindCSS v4). Static site, deployed to Cloudflare Pages (`wrangler.jsonc`); a `Dockerfile` / `compose.yaml` also exist for container builds.

Package manager is **pnpm** (`pnpm-lock.yaml`; CI uses pnpm). Node `>=22.12`.

## Commands

```
pnpm dev            # dev server at localhost:4321
pnpm build          # astro check (typecheck) + astro build + pagefind index + copy index to public/pagefind/
pnpm preview         # preview the production build
pnpm sync           # regenerate astro:content / astro:env types after schema or config changes
pnpm lint           # eslint
pnpm format:check   # prettier check (format:check, not format, is what CI runs)
pnpm format         # prettier write
```

CI (`.github/workflows/ci.yml`) runs `lint`, `format:check`, and `build` on every PR — all three must pass. There is **no test suite**.

Dev server in background mode: `astro dev --background`, managed with `astro dev stop` / `astro dev status` / `astro dev logs`.

## Configuration layering

- **`astro-paper.config.ts`** — the only file you should edit for site settings (title, socials, feature flags, posts-per-page, etc.). Typed by `defineAstroPaperConfig`.
- **`src/types/config.ts`** — the config schema and per-field documentation. Read this to understand what a setting does.
- **`src/config.ts`** — applies defaults and exports the resolved config consumed everywhere as `@/config`. **Do not edit**; change `astro-paper.config.ts` instead.
- `astro.config.ts` — Astro integrations, the markdown/Shiki pipeline, and font config.

## Content

Content collections are defined in `src/content.config.ts` (zod-validated frontmatter):

- **`posts`** — `src/content/posts/**/*.{md,mdx}`. Files or directories whose name starts with `_` are **excluded** from the collection (glob `**/[^_]*`), used for draft/scratch material. A post's parent subdirectory name becomes part of its URL.
- **`pages`** — `src/content/pages/` (currently just `about.md`).

Post visibility (`src/utils/postFilter.ts`): drafts (`draft: true`) are always hidden; scheduled posts (future `pubDatetime`) are hidden in production until within `posts.scheduledPostMargin` of publish time, but **always shown in dev**. Listings are sorted by `modDatetime ?? pubDatetime` descending (`src/utils/getSortedPosts.ts`).

## Architecture notes

- **Path alias**: `@/*` → `src/*` (plus `@/astro-paper.config`). Used throughout.
- **Routing**: file-based in `src/pages/`. Dynamic post routes under `src/pages/posts/[...slug]/`, tag pages under `src/pages/tags/`, archives under `src/pages/archives/`. Directories/files prefixed `_` (e.g. `_components`, `_utils`) are route-excluded helpers colocated with their route.
- **Dynamic OG images**: generated with Satori + Sharp. Site-level `src/pages/og.png.ts`, per-post `src/pages/posts/[...slug]/index.png.ts`. Controlled by `features.dynamicOgImage`; when disabled, `public/{site.ogImage}` must exist or the build fails.
- **Search**: Pagefind. The index is produced during `pnpm build` only — search does not work against a plain `pnpm dev` unless you build first.
- **Markdown pipeline** (`astro.config.ts`): custom `unified` processor with `remark-toc`, `remark-collapse` (collapses a section titled "Table of contents"), and `rehype-callouts`. Code blocks use Shiki (`min-light` / `night-owl`), `@shikijs/transformers` notation (`// [!code highlight]`, `// [!code ++]`, etc.), and a custom filename transformer (`src/utils/transformers/fileName.js`).
- **i18n**: UI strings live in `src/i18n/lang/*.ts` (only `en.ts`); access via `useTranslations(locale)` from `src/i18n`. `tplStr` handles interpolation. Site is currently single-locale (`en`).
- **Theming**: light/dark handled by `src/scripts/theme.ts`; styles split across `src/styles/{global,theme,typography}.css` with Tailwind v4 (configured via the Vite plugin, no `tailwind.config`).

## Conventions

- Commits follow **Conventional Commits** (commitizen, `cz.yaml`); version bumps update the changelog.
- Prettier (`.prettierrc`): no semicolons setting is `semi: true`, double quotes, `arrowParens: avoid`, 80 col; plugins for Astro and Tailwind class sorting.
- ESLint: `no-console` is an **error** — no stray `console.log`.
