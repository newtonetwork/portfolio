# Portfolio Site Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build and publish portfolio.wwoab.dev — an Astro static portfolio site served from Cloudflare Workers static assets, with project writeups, an HTML resume + PDF, and auto-deploy on push.

**Architecture:** Fully static Astro build (`dist/`) served by a Cloudflare Worker that has **no script** — only an `assets` block in `wrangler.jsonc`. Project writeups live in a zod-validated content collection (`src/content/projects/*.md`); the landing page and writeup pages render from it. Adaptive light/dark theming via CSS custom properties + `prefers-color-scheme`, with a manual toggle persisted in `localStorage`.

**Tech Stack:** Astro 6 (static output, no adapter), Cloudflare Workers static assets, Wrangler 4, plain CSS (no Tailwind, no UI frameworks).

## Global Constraints

- Node >= 22.12.0 (Astro 6 floor).
- Live URL is exactly `https://portfolio.wwoab.dev`; repo is `newtonetwork/portfolio` (public).
- No Worker script, no SSR adapter, no Tailwind, no client-side framework. The only client JS is the inline theme snippet and the toggle listener.
- Working directory for every command: `/Users/enderx/Github/portfolio`.
- Repo already exists locally with the design spec committed and `public/resume.pdf` present (untracked — committed in Task 6).
- Every commit message ends with `Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>`.

---

### Task 1: Scaffold Astro project + Wrangler config

**Files:**
- Create: `package.json`
- Create: `.gitignore`
- Create: `astro.config.mjs`
- Create: `tsconfig.json`
- Create: `wrangler.jsonc`
- Create: `src/pages/index.astro` (placeholder, replaced in Task 4)

**Interfaces:**
- Produces: `npm run build` → static site in `dist/`; `npm run check` → `astro check`; `npm run deploy` → build + `wrangler deploy`. Later tasks assume these scripts exist.

- [ ] **Step 1: Write `package.json`**

```json
{
  "name": "portfolio",
  "type": "module",
  "version": "1.0.0",
  "private": true,
  "engines": {
    "node": ">=22.12.0"
  },
  "scripts": {
    "dev": "astro dev",
    "build": "astro build",
    "check": "astro check",
    "preview": "wrangler dev",
    "deploy": "astro build && wrangler deploy"
  },
  "dependencies": {
    "astro": "^6.3.1"
  },
  "devDependencies": {
    "@astrojs/check": "^0.9.4",
    "typescript": "^5.7.2",
    "wrangler": "^4.90.0"
  }
}
```

- [ ] **Step 2: Write `.gitignore`**

```
node_modules/
dist/
.astro/
.wrangler/
.DS_Store
```

- [ ] **Step 3: Write `astro.config.mjs`**

```js
import { defineConfig } from 'astro/config';

export default defineConfig({
  site: 'https://portfolio.wwoab.dev',
});
```

- [ ] **Step 4: Write `tsconfig.json`**

```json
{
  "extends": "astro/tsconfigs/strict",
  "include": [".astro/types.d.ts", "**/*"],
  "exclude": ["dist"]
}
```

- [ ] **Step 5: Write `wrangler.jsonc`**

```jsonc
{
  "name": "portfolio",
  "compatibility_date": "2026-07-15",
  "assets": {
    "directory": "./dist",
    "not_found_handling": "404-page"
  },
  "routes": [
    { "pattern": "portfolio.wwoab.dev", "custom_domain": true }
  ]
}
```

- [ ] **Step 6: Write placeholder `src/pages/index.astro`**

```astro
---
---
<html lang="en">
  <body>
    <h1>portfolio scaffold</h1>
  </body>
</html>
```

- [ ] **Step 7: Install and build to verify the toolchain**

Run: `npm install && npm run build`
Expected: build completes; `ls dist/index.html` shows the file exists.

- [ ] **Step 8: Commit**

```bash
git add package.json package-lock.json .gitignore astro.config.mjs tsconfig.json wrangler.jsonc src/pages/index.astro
git commit -m "feat: scaffold Astro project with Workers static assets config

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

---

### Task 2: Base layout, global CSS, adaptive theme

**Files:**
- Create: `src/consts.ts`
- Create: `src/styles/global.css`
- Create: `src/layouts/Base.astro`
- Modify: `src/pages/index.astro` (use the layout; still placeholder content)

**Interfaces:**
- Produces: `SITE` const (`name`, `title`, `description`, `email`, `github`, `linkedin` — all strings). `Base.astro` accepts optional props `title?: string`, `description?: string` and provides a default `<slot />`. All later pages wrap content in `<Base>`.

- [ ] **Step 1: Write `src/consts.ts`**

```ts
export const SITE = {
  name: 'Nicholas A. Garcia',
  title: 'Nick Garcia — SRE & AI Tooling',
  description:
    'Senior Staff Site Reliability Engineer — Kubernetes at scale in FedRAMP environments, custom MCP servers, and LLM-assisted SRE workflows.',
  email: 'nick.garci28@gmail.com',
  github: 'https://github.com/newtonetwork',
  linkedin: '', // add URL when ready; footer hides the link while empty
};
```

- [ ] **Step 2: Write `src/styles/global.css`**

```css
:root {
  --bg: #fafaf8;
  --fg: #1a1a1a;
  --muted: #57534e;
  --border: #e7e5e4;
  --card: #ffffff;
  --accent: #0f766e;
  --accent-fg: #ffffff;
  --mono: ui-monospace, "SF Mono", "JetBrains Mono", Menlo, monospace;
  --sans: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif;
}

@media (prefers-color-scheme: dark) {
  :root:not([data-theme="light"]) {
    --bg: #121210;
    --fg: #e7e5e4;
    --muted: #a8a29e;
    --border: #292524;
    --card: #1c1917;
    --accent: #2dd4bf;
    --accent-fg: #121210;
  }
}

:root[data-theme="dark"] {
  --bg: #121210;
  --fg: #e7e5e4;
  --muted: #a8a29e;
  --border: #292524;
  --card: #1c1917;
  --accent: #2dd4bf;
  --accent-fg: #121210;
}

* {
  box-sizing: border-box;
  margin: 0;
}

html {
  color-scheme: light dark;
}

body {
  background: var(--bg);
  color: var(--fg);
  font-family: var(--sans);
  line-height: 1.6;
  font-size: 1.0625rem;
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

main {
  width: min(44rem, 100% - 2.5rem);
  margin-inline: auto;
  flex: 1;
  padding-block: 2.5rem 4rem;
}

h1, h2, h3 {
  line-height: 1.2;
  letter-spacing: -0.015em;
}

h2 {
  margin-block: 2.5rem 1rem;
  font-size: 1.4rem;
}

p + p {
  margin-top: 0.75rem;
}

a {
  color: var(--accent);
  text-decoration-thickness: 1px;
  text-underline-offset: 3px;
}

code {
  font-family: var(--mono);
  font-size: 0.9em;
}

.site-header nav {
  width: min(44rem, 100% - 2.5rem);
  margin-inline: auto;
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding-block: 1.25rem;
}

.brand {
  font-family: var(--mono);
  font-weight: 600;
  color: var(--fg);
  text-decoration: none;
}

.nav-links {
  display: flex;
  align-items: center;
  gap: 1.25rem;
}

.nav-links a {
  color: var(--muted);
  text-decoration: none;
}

.nav-links a:hover {
  color: var(--fg);
}

#theme-toggle {
  background: none;
  border: 1px solid var(--border);
  border-radius: 999px;
  color: var(--fg);
  cursor: pointer;
  font-size: 1rem;
  line-height: 1;
  padding: 0.35rem 0.55rem;
}

.site-footer {
  border-top: 1px solid var(--border);
  padding-block: 1.5rem;
}

.site-footer .inner {
  width: min(44rem, 100% - 2.5rem);
  margin-inline: auto;
  display: flex;
  flex-wrap: wrap;
  gap: 0.5rem 1.5rem;
  color: var(--muted);
  font-size: 0.9rem;
}

.hero .tagline {
  font-size: 1.15rem;
  color: var(--muted);
  margin-top: 0.75rem;
}

.highlights {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(11rem, 1fr));
  gap: 1rem;
  margin-top: 2rem;
}

.highlight {
  border: 1px solid var(--border);
  border-radius: 0.75rem;
  background: var(--card);
  padding: 1rem 1.15rem;
}

.highlight strong {
  display: block;
  font-size: 1.35rem;
}

.highlight span {
  color: var(--muted);
  font-size: 0.9rem;
}

.project-grid {
  display: grid;
  gap: 1rem;
}

.card {
  display: block;
  border: 1px solid var(--border);
  border-radius: 0.75rem;
  background: var(--card);
  padding: 1.25rem 1.4rem;
  color: var(--fg);
  text-decoration: none;
}

.card:hover {
  border-color: var(--accent);
}

.card p {
  color: var(--muted);
  margin-top: 0.35rem;
}

.stack {
  list-style: none;
  padding: 0;
  display: flex;
  flex-wrap: wrap;
  gap: 0.4rem;
  margin-top: 0.9rem;
}

.stack li {
  font-family: var(--mono);
  font-size: 0.75rem;
  border: 1px solid var(--border);
  border-radius: 999px;
  padding: 0.15rem 0.6rem;
  color: var(--muted);
}

.writeup .back {
  font-size: 0.9rem;
}

.writeup h1 {
  margin-top: 1rem;
}

.writeup .summary {
  color: var(--muted);
  font-size: 1.1rem;
  margin-top: 0.5rem;
}

.writeup .links {
  margin-top: 0.75rem;
  display: flex;
  gap: 1.25rem;
}

.writeup :is(h2, h3) {
  margin-block: 2rem 0.75rem;
}

.writeup ul {
  padding-left: 1.25rem;
  margin-block: 0.75rem;
}

.resume-actions {
  margin-block: 1rem 2rem;
}

.resume-actions a {
  display: inline-block;
  background: var(--accent);
  color: var(--accent-fg);
  border-radius: 0.5rem;
  padding: 0.5rem 1rem;
  text-decoration: none;
  font-weight: 600;
}

.resume h2 {
  border-bottom: 1px solid var(--border);
  padding-bottom: 0.35rem;
}

.resume .role {
  margin-top: 1.5rem;
}

.resume .role h3 {
  font-size: 1.05rem;
}

.resume .meta {
  color: var(--muted);
  font-size: 0.9rem;
}

.resume ul {
  padding-left: 1.25rem;
  margin-top: 0.5rem;
}

.resume ul li + li {
  margin-top: 0.3rem;
}
```

- [ ] **Step 3: Write `src/layouts/Base.astro`**

```astro
---
import '../styles/global.css';
import { SITE } from '../consts';

interface Props {
  title?: string;
  description?: string;
}

const { title = SITE.title, description = SITE.description } = Astro.props;
---

<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>{title}</title>
    <meta name="description" content={description} />
    <script is:inline>
      const saved = localStorage.getItem('theme');
      if (saved) document.documentElement.dataset.theme = saved;
    </script>
  </head>
  <body>
    <header class="site-header">
      <nav>
        <a href="/" class="brand">nick.garcia</a>
        <div class="nav-links">
          <a href="/#projects">Projects</a>
          <a href="/resume/">Resume</a>
          <button id="theme-toggle" aria-label="Toggle color theme">◐</button>
        </div>
      </nav>
    </header>
    <main>
      <slot />
    </main>
    <footer class="site-footer">
      <div class="inner">
        <a href={`mailto:${SITE.email}`}>{SITE.email}</a>
        <a href={SITE.github}>GitHub</a>
        {SITE.linkedin && <a href={SITE.linkedin}>LinkedIn</a>}
        <span>© 2026 {SITE.name}</span>
      </div>
    </footer>
    <script>
      document.getElementById('theme-toggle')?.addEventListener('click', () => {
        const root = document.documentElement;
        const current =
          root.dataset.theme ??
          (matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light');
        const next = current === 'dark' ? 'light' : 'dark';
        root.dataset.theme = next;
        localStorage.setItem('theme', next);
      });
    </script>
  </body>
</html>
```

- [ ] **Step 4: Update `src/pages/index.astro` to use the layout**

```astro
---
import Base from '../layouts/Base.astro';
---

<Base>
  <h1>portfolio scaffold</h1>
</Base>
```

- [ ] **Step 5: Verify build and theme wiring**

Run: `npm run build && grep -c 'data-theme\|theme-toggle' dist/index.html`
Expected: build passes; grep prints a count >= 2 (inline theme script + toggle button present).

- [ ] **Step 6: Commit**

```bash
git add src/consts.ts src/styles/global.css src/layouts/Base.astro src/pages/index.astro
git commit -m "feat: base layout with adaptive light/dark theme

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

---

### Task 3: Projects content collection + seed writeups

**Files:**
- Create: `src/content.config.ts`
- Create: `src/content/projects/cf-board.md`
- Create: `src/content/projects/configdrift.md`

**Interfaces:**
- Produces: collection `projects` with entry frontmatter `{ title: string; summary: string; stack: string[]; repo?: string; demo?: string; order: number; draft: boolean }`. Entry ids are the filenames without extension (`cf-board`, `configdrift`). Tasks 4 and 5 consume it via `getCollection('projects', ({ data }) => !data.draft)`.

- [ ] **Step 1: Write `src/content.config.ts`**

```ts
import { defineCollection, z } from 'astro:content';
import { glob } from 'astro/loaders';

const projects = defineCollection({
  loader: glob({ pattern: '**/*.md', base: './src/content/projects' }),
  schema: z.object({
    title: z.string(),
    summary: z.string(),
    stack: z.array(z.string()).default([]),
    repo: z.string().url().optional(),
    demo: z.string().url().optional(),
    order: z.number(),
    draft: z.boolean().default(false),
  }),
});

export const collections = { projects };
```

- [ ] **Step 2: Write `src/content/projects/cf-board.md`**

```markdown
---
title: "cf-board — shared trip board on Cloudflare Workers"
summary: "A no-login, real-time-enough family trip board. One Worker serves a React app and a KV-backed API; everyone with the URL sees the same live board. Runs for ~$0/month with hourly automated backups."
stack: ["Cloudflare Workers", "Workers KV", "React", "Wrangler"]
order: 1
---

## Problem

Coordinating a multi-family trip meant itinerary details scattered across
group texts. Shared docs require accounts, and nobody installs an app for
one trip. The requirement: one URL anyone can open, edit, and trust to be
current — with zero onboarding.

## Architecture

A single Cloudflare Worker does everything:

- Serves the static React app via Workers static assets.
- Exposes a tiny JSON API (`/api/kv/:key`) backed by Workers KV for board
  state — lists, schedules, assignments.
- A scheduled trigger snapshots KV hourly to versioned backup keys, so a
  bad edit is recoverable.

No accounts, no database server, no build pipeline beyond `wrangler deploy`.

## Decisions

- **KV over D1:** board state is a handful of JSON documents with
  last-write-wins semantics — a key-value store fits; SQL would be
  ceremony. KV's eventual consistency is acceptable for this use.
- **Capability URL instead of auth:** the unguessable URL is the access
  control. Right-sized for a family trip board; documented as an explicit
  trade-off, not an oversight.
- **Backups before features:** the hourly KV snapshot shipped before nice-
  to-haves, because "someone deleted the schedule" is the actual risk.

## Outcome

In active family use. Total infrastructure cost: $0 on the Workers free
tier. Deploy-to-live is under a minute.
```

- [ ] **Step 3: Write `src/content/projects/configdrift.md`**

```markdown
---
title: "ConfigDrift — self-hosted configuration drift monitor"
summary: "A Go agent snapshots host state into SQLite and diffs it against a promoted baseline; a FastAPI + HTMX UI reviews, acknowledges, and promotes drift. Built with NIST 800-53 configuration-management controls in mind."
stack: ["Go", "SQLite", "FastAPI", "HTMX", "Docker"]
order: 2
---

## Problem

Configuration drift is how compliant systems quietly stop being compliant.
NIST 800-53's CM family (CM-2 baseline configuration, CM-3 change control,
CM-6 configuration settings) assumes you can answer: *what changed on this
host since the approved baseline, and who acknowledged it?* Most homelab
and small-team setups can't answer that at all.

## Architecture

Two components, both self-hosted in Docker:

- **Agent (Go):** snapshots watched state into SQLite — file and directory
  metadata/content, installed packages (`dpkg-query` with `rpm -qa`
  fallback), systemd units, per-user crontabs and timers, and running
  Docker containers via the Engine API. Each run diffs against the
  promoted baseline and records drift events.
- **UI (FastAPI + HTMX):** syntax-highlighted diff viewer to inspect each
  drift event, acknowledge it, or promote the new snapshot as the
  baseline. Immediate and digest alerts go out through an optional ntfy
  webhook — no external services otherwise.

## Decisions

- **Baseline + promote, not just alerting:** drift tools that only alert
  train you to ignore them. The acknowledge/promote workflow mirrors how
  change control actually works: drift is either approved (promote) or a
  finding (investigate).
- **Ignore rules and content redaction** for noisy or secret-bearing paths,
  so the drift log stays reviewable and safe to retain as evidence.
- **SQLite over a server database:** one host, one file, trivial backup —
  the operational cost of Postgres buys nothing here.

## Compliance angle

The drift event log — what changed, when, and its acknowledge/promote
disposition — is exactly the shape of evidence a CM-family control
assessment asks for. The roadmap pairs it with LLM-generated evidence
narratives per drift event.
```

- [ ] **Step 4: Verify the schema gate actually fails a bad entry**

Run:
```bash
sed -i '' 's/^order: 2$/order: "two"/' src/content/projects/configdrift.md
npm run build
```
Expected: build FAILS with a zod error naming `order` in `configdrift.md`.

- [ ] **Step 5: Revert the intentional break and verify build passes**

Run:
```bash
sed -i '' 's/^order: "two"$/order: 2/' src/content/projects/configdrift.md
npm run build
```
Expected: build PASSES.

- [ ] **Step 6: Commit**

```bash
git add src/content.config.ts src/content/projects/
git commit -m "feat: projects content collection with cf-board and configdrift writeups

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

---

### Task 4: Landing page (bio, highlights, project cards, contact)

**Files:**
- Modify: `src/pages/index.astro` (replace placeholder entirely)

**Interfaces:**
- Consumes: `Base.astro` layout, `SITE` const, `projects` collection from Task 3.
- Produces: `/` with a `#projects` anchor that nav links target; cards link to `/projects/<id>/` (built in Task 5).

- [ ] **Step 1: Replace `src/pages/index.astro`**

```astro
---
import { getCollection } from 'astro:content';
import Base from '../layouts/Base.astro';
import { SITE } from '../consts';

const projects = (await getCollection('projects', ({ data }) => !data.draft)).sort(
  (a, b) => a.data.order - b.data.order,
);
---

<Base>
  <section class="hero">
    <h1>{SITE.name}</h1>
    <p class="tagline">
      Senior Staff Site Reliability Engineer — Kubernetes at scale in FedRAMP
      environments, and hands-on AI/agent tooling.
    </p>
    <p>
      I run the reliability side of a FedRAMP-authorized platform — EKS,
      Terraform, and continuous NIST 800-53 compliance — and build AI tooling
      that makes that work faster: custom MCP servers, Claude-integrated
      agents, and LLM-assisted SRE workflows. Before this: privileged access
      management consulting, and six years operating airborne mission systems
      in the US Air Force.
    </p>
  </section>

  <section class="highlights" aria-label="Highlights">
    <div class="highlight"><strong>11 yrs</strong><span>IT &amp; mission-critical ops</span></div>
    <div class="highlight"><strong>FedRAMP</strong><span>core contributor, first authorized deployment</span></div>
    <div class="highlight"><strong>5 EKS</strong><span>clusters owned, prod + non-prod</span></div>
  </section>

  <section id="projects">
    <h2>Projects</h2>
    <div class="project-grid">
      {
        projects.map((p) => (
          <a class="card" href={`/projects/${p.id}/`}>
            <h3>{p.data.title}</h3>
            <p>{p.data.summary}</p>
            <ul class="stack">
              {p.data.stack.map((s) => (
                <li>{s}</li>
              ))}
            </ul>
          </a>
        ))
      }
    </div>
  </section>

  <section id="contact">
    <h2>Contact</h2>
    <p>
      <a href={`mailto:${SITE.email}`}>{SITE.email}</a> · <a href={SITE.github}>github.com/newtonetwork</a>
    </p>
  </section>
</Base>
```

- [ ] **Step 2: Verify cards render from the collection**

Run: `npm run build && grep -o 'cf-board[^"]*\|ConfigDrift[^<]*' dist/index.html | sort -u`
Expected: build passes; output includes both project titles.

- [ ] **Step 3: Commit**

```bash
git add src/pages/index.astro
git commit -m "feat: landing page with bio, highlights, and project cards

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

---

### Task 5: Project writeup pages

**Files:**
- Create: `src/pages/projects/[slug].astro`

**Interfaces:**
- Consumes: `projects` collection (entry ids as slugs), `Base.astro`.
- Produces: `/projects/cf-board/` and `/projects/configdrift/` static pages.

- [ ] **Step 1: Write `src/pages/projects/[slug].astro`**

```astro
---
import { getCollection, render } from 'astro:content';
import Base from '../../layouts/Base.astro';

export async function getStaticPaths() {
  const projects = await getCollection('projects', ({ data }) => !data.draft);
  return projects.map((p) => ({ params: { slug: p.id }, props: { project: p } }));
}

const { project } = Astro.props;
const { Content } = await render(project);
---

<Base title={`${project.data.title} — Nick Garcia`} description={project.data.summary}>
  <article class="writeup">
    <a class="back" href="/#projects">← All projects</a>
    <h1>{project.data.title}</h1>
    <p class="summary">{project.data.summary}</p>
    <ul class="stack">
      {project.data.stack.map((s) => <li>{s}</li>)}
    </ul>
    {
      (project.data.repo || project.data.demo) && (
        <p class="links">
          {project.data.repo && <a href={project.data.repo}>Source</a>}
          {project.data.demo && <a href={project.data.demo}>Live demo</a>}
        </p>
      )
    }
    <Content />
  </article>
</Base>
```

- [ ] **Step 2: Verify both writeup pages are emitted**

Run: `npm run build && ls dist/projects/cf-board/index.html dist/projects/configdrift/index.html`
Expected: both files listed.

- [ ] **Step 3: Commit**

```bash
git add src/pages/projects/
git commit -m "feat: project writeup pages from content collection

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

---

### Task 6: Resume page + PDF

**Files:**
- Create: `src/pages/resume.astro`
- Add (already on disk, untracked): `public/resume.pdf`

**Interfaces:**
- Consumes: `Base.astro`, `SITE`.
- Produces: `/resume/` page and `/resume.pdf` download (no path conflict — `dist/resume/index.html` and `dist/resume.pdf` coexist).

- [ ] **Step 1: Write `src/pages/resume.astro`**

```astro
---
import Base from '../layouts/Base.astro';
import { SITE } from '../consts';
---

<Base title="Resume — Nick Garcia">
  <article class="resume">
    <h1>{SITE.name}</h1>
    <p class="meta">Pasco, WA · <a href={`mailto:${SITE.email}`}>{SITE.email}</a></p>
    <p class="resume-actions"><a href="/resume.pdf" download>Download PDF</a></p>

    <h2>Profile</h2>
    <p>
      Senior Staff SRE with 11 years of IT experience operating Kubernetes at
      scale in FedRAMP environments under continuous NIST 800-53 compliance.
      Core contributor to CyberArk's first FedRAMP-authorized deployment —
      architecture, control implementation, and audit readiness across initial
      and annual assessments. Led the migration from a legacy Windows/IIS +
      MSSQL stack to Kubernetes microservices. Builds AI/agent tooling
      hands-on: custom MCP servers, Claude-integrated agents, and LLM-assisted
      SRE workflows. Military background in mission-critical systems + M.S.
      Cybersecurity.
    </p>

    <h2>AI &amp; Agent Engineering</h2>
    <div class="role">
      <h3>AI Infrastructure &amp; Agent Workflows (Independent Projects)</h3>
      <ul>
        <li>Built custom MCP (Model Context Protocol) servers from scratch, wiring Claude to external tools and APIs to create functional AI agents with real-world integrations.</li>
        <li>Architected a multi-project persistent memory system (Basic Memory MCP) enabling stateful context across agent sessions, structured across work, infrastructure, and personal knowledge domains.</li>
        <li>Designed agent automation workflows combining LLM reasoning with scripted tool calls — accelerating documentation generation, IT assessments, and operational runbooks.</li>
      </ul>
    </div>
    <div class="role">
      <h3>Applied AI for SRE (Active Prototyping)</h3>
      <ul>
        <li>Prototyping LLM-assisted incident triage, runbook generation, and compliance drift detection — pairing agent reasoning with Steampipe and Terraform state diffing for NIST 800-53 evidence generation.</li>
      </ul>
    </div>

    <h2>Experience</h2>
    <div class="role">
      <h3>Senior Staff Site Reliability Engineer — Palo Alto Networks (formerly CyberArk)</h3>
      <p class="meta">Jan 2024 – Present · CyberArk acquired by Palo Alto Networks, Feb 2026</p>
      <ul>
        <li>Own reliability and operational excellence of the Endpoint Privilege Management (EPM) backend in AWS — 5 EKS clusters spanning production and non-production across commercial and FedRAMP boundaries.</li>
        <li>Led container security scan vendor migration to JFrog Xray and Snyk, satisfying third-party requirements for FedRAMP continuous monitoring.</li>
        <li>Built a FedRAMP-compliant database restore process to demonstrate disaster recovery for annual assessment.</li>
        <li>Automated CorePAS component install/upgrade with Ansible, cutting deployment time by ~2 hours vs. the prior manual process.</li>
        <li>Executed STIG hardening and upgrades to satisfy NIST 800-53 Rev5 controls; eliminated wildcard certificates and configured automatic access key rotation.</li>
        <li>Led modernization from legacy Windows/IIS + MSSQL to Kubernetes microservices on RDS Postgres, OpenSearch, and RabbitMQ.</li>
        <li>Architect and maintain cloud-native infrastructure (EKS, EC2, RDS/Postgres, S3, VPC, IAM) fully codified in Terraform, managed with Helm, Ansible, and Bamboo CI/CD.</li>
        <li>Own the vulnerability management loop — Tenable/Nessus and Invicti scanning, CVE triage, remediation coordination, and audit-ready reporting — plus certificate lifecycle operations across ACM, OpenSSL, and internal CAs.</li>
      </ul>
    </div>
    <div class="role">
      <h3>Site Reliability Engineer — CyberArk</h3>
      <p class="meta">Apr 2023 – Jan 2024</p>
      <ul>
        <li>Implemented high availability and disaster recovery for the CorePAS environment, making privileged access infrastructure fully redundant.</li>
        <li>Operated AWS/EKS platform services in FedRAMP context; on-call response, triage, and escalation for customer-impacting incidents.</li>
        <li>Tuned CloudWatch metrics, alarms, and dashboards to reduce alert noise; automated recurring operational work via runbooks and IaC.</li>
      </ul>
    </div>
    <div class="role">
      <h3>Technical Product Engineer — CyberArk</h3>
      <p class="meta">Oct 2022 – Apr 2023</p>
      <ul>
        <li>Technical SME for Endpoint Privilege Manager — policy enforcement, agent deployment, performance tuning, and endpoint integration across Windows and macOS.</li>
        <li>Led deep-dive investigations into low-level Windows behavior and process elevation flow; escalated edge-case defects to R&amp;D with debugging artifacts.</li>
      </ul>
    </div>
    <div class="role">
      <h3>CyberArk Consultant — Focal Point Data Risk LLC (CDW)</h3>
      <p class="meta">Jun 2020 – Oct 2022</p>
      <ul>
        <li>Designed and deployed CyberArk PAM infrastructure across on-premise, hybrid, and cloud environments for Federal customers.</li>
      </ul>
    </div>
    <div class="role">
      <h3>Airborne Mission Systems Operator — USAF</h3>
      <p class="meta">Dec 2014 – 2020 · Staff Sergeant (E-5)</p>
      <ul>
        <li>Led a flight of ~40 personnel across multiple operational theaters — readiness, scheduling, and mission execution for globally distributed operations.</li>
        <li>Developed and delivered aircrew training programs — standardized qualification processes and improved troubleshooting proficiency.</li>
      </ul>
    </div>

    <h2>Education &amp; Certifications</h2>
    <ul>
      <li>M.S., Cybersecurity — Bellevue University, 2020</li>
      <li>B.A.S., Computer Information Systems — Bellevue University, 2018</li>
      <li>AWS Generative AI and AI Agents with Amazon Bedrock Specialization — 2026</li>
      <li>Certified CyberArk Delivery Engineer — 2022</li>
    </ul>

    <h2>Skills</h2>
    <ul>
      <li><strong>AI &amp; Agent Tooling:</strong> MCP server development, Claude integrations, LLM-assisted SRE workflows, agent automation, persistent memory architecture</li>
      <li><strong>FedRAMP / Compliance:</strong> NIST 800-53 (Rev5), continuous monitoring, STIG hardening, vulnerability remediation, CVE triage</li>
      <li><strong>Cloud / Platform:</strong> AWS (EKS, EC2, VPC, IAM, RDS/Postgres, S3, CloudWatch, ACM), Kubernetes, Docker, Helm</li>
      <li><strong>IaC / Automation:</strong> Terraform, CloudFormation, Ansible, Bamboo CI/CD, Bash, PowerShell, Python</li>
      <li><strong>Observability / IR:</strong> CloudWatch, PagerDuty, Splunk, Steampipe, runbooks, RCA/corrective actions</li>
    </ul>
  </article>
</Base>
```

- [ ] **Step 2: Verify resume page and PDF land in dist**

Run: `npm run build && ls dist/resume/index.html dist/resume.pdf && grep -c 'Palo Alto Networks' dist/resume/index.html`
Expected: both files listed; grep count >= 1.

- [ ] **Step 3: Commit (includes the PDF)**

```bash
git add src/pages/resume.astro public/resume.pdf
git commit -m "feat: resume page with PDF download

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

---

### Task 7: 404 page

**Files:**
- Create: `src/pages/404.astro`

**Interfaces:**
- Consumes: `Base.astro`. `wrangler.jsonc` already sets `"not_found_handling": "404-page"`, which serves `dist/404.html` for unknown paths.

- [ ] **Step 1: Write `src/pages/404.astro`**

```astro
---
import Base from '../layouts/Base.astro';
---

<Base title="404 — Nick Garcia">
  <h1>404</h1>
  <p>That page doesn't exist. <a href="/">Back to the front page</a>.</p>
</Base>
```

- [ ] **Step 2: Verify 404 output + run full check**

Run: `npm run build && ls dist/404.html && npm run check`
Expected: `dist/404.html` exists; `astro check` reports 0 errors.

- [ ] **Step 3: Commit**

```bash
git add src/pages/404.astro
git commit -m "feat: 404 page

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
```

---

### Task 8: Publish repo + first deploy + custom domain

**Files:**
- No file changes. GitHub repo creation, Worker deploy, DNS.

**Interfaces:**
- Consumes: everything above; `wrangler.jsonc` route block creates the custom domain on first deploy (zone `wwoab.dev` is already in the Cloudflare account).
- Produces: public repo `newtonetwork/portfolio`; live site at `https://portfolio.wwoab.dev`.

- [ ] **Step 1: Create the public GitHub repo and push**

```bash
gh repo create newtonetwork/portfolio --public --source . --push \
  --description "Portfolio — Astro on Cloudflare Workers static assets (portfolio.wwoab.dev)"
```
Expected: repo created; `main` pushed.

- [ ] **Step 2: Deploy**

Run: `npm run deploy`
Expected: wrangler uploads assets and prints the deployed Worker with the
`portfolio.wwoab.dev` custom domain attached. If wrangler is not authenticated,
stop and ask Nick to run `npx wrangler login` (interactive browser auth).

- [ ] **Step 3: Verify live**

Run: `curl -sI https://portfolio.wwoab.dev | head -1 && curl -s https://portfolio.wwoab.dev/resume.pdf -o /dev/null -w '%{http_code}\n'`
Expected: `HTTP/2 200` and `200`. Certificate provisioning for a new custom
domain can take a few minutes — retry before treating a 5xx/526 as failure.

- [ ] **Step 4: Manual spot-check (report to Nick, don't skip)**

Open `https://portfolio.wwoab.dev` and verify: both themes via the toggle,
mobile-width layout, both project writeups reachable from cards, resume page
renders, PDF downloads, 404 page on a garbage URL.

---

### Task 9: Auto-deploy on push (Workers Builds)

**Files:** none — Cloudflare dashboard configuration (requires Nick).

- [ ] **Step 1: Connect the repo (Nick, in the dashboard)**

Cloudflare dashboard → Workers & Pages → `portfolio` → Settings → Build →
Connect the `newtonetwork/portfolio` GitHub repo.
Build command: `npm run check && npm run build` · Deploy command:
`npx wrangler deploy` · Branch: `main`. (The check step is the spec's CI
gate — a broken writeup or type error fails the build instead of deploying.)

- [ ] **Step 2: Verify the pipeline**

```bash
git commit --allow-empty -m "chore: trigger Workers Builds

Co-Authored-By: Claude Fable 5 <noreply@anthropic.com>"
git push
```
Expected: a build appears in the dashboard and finishes green; site still
serves 200 afterward. From here on, publishing a new project writeup is:
add one `.md` to `src/content/projects/`, push.
