# AGENTS.md

This file is for coding agents working in this repository.

## Project summary

- This repo is a VitePress-based personal blog.
- Deployment target is Cloudflare Pages.
- GitHub Actions deploys automatically on pushes to `main`.
- The built site output is `.vitepress/dist`.

## Stack

- Node.js + npm
- VitePress `^1.6.4`
- Vue components inside the custom VitePress theme
- `gray-matter` for frontmatter parsing
- `globby` + `fs-extra` for post discovery and generated pagination pages
- Giscus comments via `vitepress-plugin-comment-with-giscus`
- Mermaid via `vitepress-plugin-mermaid`

## Key files and directories

- `package.json` — scripts and dependencies
- `.github/workflows/deploy-cloudflare.yml` — CI build and Cloudflare Pages deploy
- `.vitepress/config.ts` — main site config
- `.vitepress/theme/index.ts` — custom theme entry
- `.vitepress/theme/serverUtils.ts` — scans posts, parses frontmatter, sorts posts, generates pagination pages
- `.vitepress/theme/functions.ts` — tag/category/archive grouping helpers
- `.vitepress/theme/components/` — custom UI pieces such as layout, list pages, tags, archives, category, comments
- `posts/` — blog posts in Markdown
- `pages/` — standalone pages like tags, category, archives, about
- `public/` — static assets
- `index.md` — generated home page source used by VitePress

## Local commands

Run these from the repo root:

```bash
npm ci
npm run dev
npm run build
```

Notes:

- `npm run dev` maps to `vitepress dev --host 0.0.0.0`
- `npm run build` maps to `vitepress build`
- CI currently uses Node 20 in GitHub Actions
- Local builds were verified to succeed in this repo

## Content model

Posts live under `posts/**/*.md` and are expected to use frontmatter like:

```yaml
---
date: 2026-03-17
title: Example title
category: AI
tags:
  - tag1
  - tag2
description: Short summary
order: 0 # optional
---
```

Observed conventions:

- `date`, `title`, `category`, `tags`, and `description` are expected by the theme
- `order` is optional and is converted to a number; higher `order` sorts first and is shown as pinned
- Content is primarily Chinese, but some navigation labels are English; preserve the existing style unless asked to rework it globally

## Build-time behavior you must know

`getPosts()` in `.vitepress/theme/serverUtils.ts` does more than just read content:

- scans `posts/**/**.md`
- parses frontmatter with `gray-matter`
- sorts posts by `order` descending, then by `date` descending
- generates pagination source files at the repo root
- rewrites `index.md` from generated pagination content

Important consequences:

- Do not hand-edit root `index.md` as a source of truth; it is generated
- Additional `page_N.md` files may be generated at the repo root when the number of posts exceeds the page size
- Production builds exclude:
  - `posts/draft/**/*.md`
  - `posts/private-notes/**/*.md`
  - `posts/trash/**/*.md`
- `.vitepress/config.ts` also excludes `README.md` in source processing

## Theme behavior

- `.vitepress/theme/components/NewLayout.vue` injects post metadata above docs and comments below docs
- `CommentGiscus.vue` wires comments from `themeConfig.comment`
- Tags/category/archive pages are generated from `theme.value.posts`
- `base` is currently set to `/`; change it only if deployment path changes
- `vite.server.allowedHosts` currently includes `bbit.fun`

## Deployment behavior

GitHub Actions workflow:

- file: `.github/workflows/deploy-cloudflare.yml`
- triggers:
  - push to `main`
  - manual dispatch
- steps:
  - checkout
  - setup Node 20 with npm cache
  - `npm ci`
  - `npm run build`
  - deploy `.vitepress/dist` via `cloudflare/pages-action@v1`

Required GitHub secrets:

- `CLOUDFLARE_API_TOKEN`
- `CLOUDFLARE_ACCOUNT_ID`
- `CLOUDFLARE_PAGES_PROJECT_NAME`

## Guardrails for future edits

- After changing theme logic, config, workflow, or post-processing code, run `npm run build`
- Do not commit `node_modules/` or `.vitepress/dist/`
- Be careful editing `themeConfig.comment` values; wrong repo IDs or category IDs break comments
- Be careful editing `serverUtils.ts`; it mutates root-level Markdown files during build
- Keep changes minimal and aligned with the current architecture; this is a small custom VitePress site, not a generic Vue app

## Git and ignore rules

- Root-level Markdown files are ignored by default through `/*.md` in `.gitignore`
- `AGENTS.md` has been explicitly unignored so it can be tracked
- If you add other root-level Markdown files that must be committed, update `.gitignore` deliberately

## Recommended agent workflow

1. Read `package.json`, `.vitepress/config.ts`, and the relevant theme files before editing
2. If changing content behavior, inspect `.vitepress/theme/serverUtils.ts` first
3. Make the smallest possible change
4. Run `npm run build`
5. Check git status and diff before finishing
