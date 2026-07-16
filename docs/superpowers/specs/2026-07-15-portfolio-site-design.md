# Portfolio Site — Design

**Date:** 2026-07-15
**Status:** Approved
**Live URL:** https://portfolio.wwoab.dev
**Repo:** newtonetwork/portfolio (public)

## Purpose

A personal portfolio site for Nick that substantiates the SRE + AI story on his
resume. It hosts a bio, an HTML resume with PDF download, and per-project
writeups that are added incrementally as repos are polished and made public.
Success: a hiring manager lands on one URL and can read the resume, scan
project cards, and click through to writeups and GitHub repos.

## Decisions

- **Hostname:** `portfolio.wwoab.dev` (existing Cloudflare zone; custom domain
  attached to the Worker).
- **Stack:** Astro, fully static output, served via Cloudflare Workers static
  assets. No Worker script, no SSR adapter, no Tailwind — plain Astro-scoped
  CSS with custom properties.
- **Theme:** adaptive light/dark via `prefers-color-scheme`, plus a manual
  toggle persisted in `localStorage`.
- **V1 scope:** landing page (bio + highlights + project cards), per-project
  writeup pages, resume page + downloadable PDF, contact/links section.
  Explicitly out of scope for v1: remote MCP demo, contact form, analytics,
  blog.
- **Seed content:** two project writeups at launch — cf-board and configdrift —
  so the site never looks empty. homelab-mcp / homelab-agent writeups follow
  after their repos pass a secrets scan.

## Repo layout

```
portfolio/
  astro.config.mjs        static output, site: https://portfolio.wwoab.dev
  wrangler.jsonc          name: "portfolio", assets: { directory: "./dist" }
  src/
    content.config.ts     "projects" collection schema (zod)
    content/projects/     one .md per project
    layouts/Base.astro    head, nav, footer, theme handling
    pages/
      index.astro         bio + highlights + project cards
      resume.astro        HTML resume
      404.astro           custom not-found page
      projects/[slug].astro   writeup pages (getStaticPaths over collection)
  public/
    resume.pdf            downloadable PDF
  docs/superpowers/specs/ design docs (this file)
```

## Content model

`projects` content collection. Frontmatter schema (zod, build fails on
mismatch):

| Field     | Type          | Notes                                          |
|-----------|---------------|------------------------------------------------|
| `title`   | string        | required                                       |
| `summary` | string        | card text on landing page, required            |
| `stack`   | string[]      | rendered as tags                               |
| `repo`    | URL, optional | omitted while a repo is still private          |
| `demo`    | URL, optional | live demo link                                 |
| `order`   | number        | card sort order                                |
| `draft`   | boolean       | default false; drafts excluded from build      |

Body: problem → architecture → decisions → links. Publishing a new project is:
add one `.md`, push.

## Deploy flow

1. First deploy manual: `astro build && wrangler deploy` → live on
   `portfolio.<account>.workers.dev`.
2. Attach `portfolio.wwoab.dev` as a custom domain on the Worker (zone already
   in the account; DNS record is created automatically).
3. Connect the GitHub repo to Workers Builds so pushes to `main` auto-build
   and deploy. No deploy tokens stored in GitHub.

## Error handling & testing

- `404.astro` served automatically by Workers assets for unknown paths.
- Zod schema on the collection: malformed project frontmatter fails the build
  rather than rendering broken.
- CI gate: `astro check` + `astro build` must pass.
- Pre-ship manual verification: both themes, mobile layout, resume PDF
  download, all card → writeup → repo links.
