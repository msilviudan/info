# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Personal website/blog built with **Astro 5** and deployed to **Cloudflare Workers** via the `@astrojs/cloudflare` adapter (server-side rendering at the edge, not static-only). Started from the Astro Blog starter template; some content is in Romanian (e.g. the `sibiubus` privacy-policy page).

## Commands

- `npm run dev` — local dev server (Astro, http://localhost:4321)
- `npm run build` — build to `./dist/` (produces the Cloudflare `_worker.js`)
- `npm run preview` — build, then serve via `wrangler dev` (runs the actual Worker locally; use this over `dev` to reproduce edge behavior)
- `npm run check` — full validation: `astro build && tsc && wrangler deploy --dry-run`. Run before deploying.
- `npm run deploy` — `wrangler deploy` (expects `./dist` to already be built)
- `npm run build && npm run deploy` — build and deploy in one step
- `npm run cf-typegen` — regenerate `worker-configuration.d.ts` from Wrangler bindings; run after changing bindings in `wrangler.json`
- `npx wrangler tail` — stream live Worker logs (observability is enabled)

There is no test suite and no separate lint step; `tsc` (via `check`) is the type gate.

## Architecture

- **Routing**: file-based under [src/pages/](src/pages/). `.astro`/`.md` files become routes by filename. `blog/[...slug].astro` renders one page per blog entry via `getStaticPaths()`.
- **Content collections**: blog posts live in [src/content/blog/](src/content/blog/) as `.md`/`.mdx`. The `blog` collection is defined in [src/content.config.ts](src/content.config.ts) with a Zod frontmatter schema (`title`, `description`, `pubDate`, optional `updatedDate`, `heroImage`). Adding a frontmatter field means updating that schema. Retrieve posts with `getCollection('blog')`.
- **Layouts & components**: [src/layouts/BlogPost.astro](src/layouts/BlogPost.astro) is the shared page shell (BaseHead + Header + Footer + prose slot). Standalone pages that should look like posts (e.g. `sibiubus.astro`) import this layout directly and pass the same props the schema expects. Shared pieces live in [src/components/](src/components/) — `BaseHead.astro` (SEO/OpenGraph head), `Header`, `Footer`, `FormattedDate`, `HeaderLink`.
- **Site metadata**: `SITE_TITLE` / `SITE_DESCRIPTION` in [src/consts.ts](src/consts.ts) feed the RSS feed and page heads. The public site URL is `site` in [astro.config.mjs](astro.config.mjs) (currently the placeholder `https://example.com` — used for canonical URLs, sitemap, and RSS `link`s, so update it when deploying for real).
- **RSS**: [src/pages/rss.xml.js](src/pages/rss.xml.js) generates the feed from the blog collection.

## Cloudflare specifics

- [wrangler.json](wrangler.json) configures the Worker: entry is `./dist/_worker.js/index.js`, static assets served from `./dist` via the `ASSETS` binding, `nodejs_compat` flag enabled, source-map upload and observability on.
- `astro.config.mjs` enables `platformProxy` so Cloudflare bindings are available during `astro dev`.
- Cloudflare binding types are generated into `worker-configuration.d.ts` (do not edit by hand — regenerate with `npm run cf-typegen`).
