# Canonical Stack Reference

> This is the authoritative record of all technologies in use across Andre's projects.
> Update this file when a new tool is adopted or retired.

---

## Core Stack

| Layer | Tool | Notes |
|-------|------|-------|
| Frontend language | Vanilla JS (ES6+) | No framework preference |
| Markup | HTML5 | Semantic, accessible |
| Styling | CSS (custom properties) | tokens.css → base → layout → components pattern |
| Build tool | None (preferred) / Vite | No-build-step unless justified |
| CDN | unpkg.com/@supabase/supabase-js@2 | No-build projects |
| Database | Supabase / PostgreSQL | Hosted, managed |
| Auth | Supabase Auth | Magic links, OAuth |
| Auth storage | sessionStorage | Client-side |
| Realtime | Supabase Realtime | REPLICA IDENTITY FULL required |
| Serverless | Supabase Edge Functions | TypeScript, Deno runtime |
| Scheduled jobs | pg_cron (via Supabase) | World tick, automation |
| Hosting | GitHub Pages | Static frontend |
| Version control | GitHub | Issues, Projects, Actions |
| AI interface | Perplexity | Primary LLM |

## Secondary / Project-Specific

| Tool | Used In | Notes |
|------|---------|-------|
| Godot / GDScript | virtual-pinball-wall | Game engine |
| Python | the-thread-pdf | PDF generation |
| PLpgSQL | chronicle-worlds, RFQ-solutions, reddit-mirror | Complex DB logic |
| Vite | chronicle-worlds | Justified by complexity |

## Supabase Projects Registry

| Project ID | Region | Used By |
|------------|--------|---------|
| hhyhulqngdkwsxhymmcd | us-west-2 | chronicle-worlds, virtual-pinball-wall |
| nmemmfblpzrkwyljpmvp | us-east-2 | rfq-course |
| [add as needed] | | |

## Deploy Patterns

### Pattern A — Simple (no build step)
- Source: `main` branch, `index.html` at root
- Pages config: Deploy from branch → `main` / `/ (root)`
- Used by: rfq-course, most static sites

### Pattern B — Frontend subdirectory + Actions
- Source: `frontend/src/` on `main`
- Deploy: GitHub Actions triggers on push to `frontend/**`, builds to `gh-pages` branch
- Never edit `gh-pages` or `docs/` directly
- Used by: chronicle-worlds

---

*Last updated: 2026-07-06*
