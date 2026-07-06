# Genesis Materials — Agent Skill Hub
## Master Outline & Reference Document

> **Purpose:** This file is the canonical reference for building out the Genesis Agent Skill Hub — a portable, LLM-agnostic "brain" for coding sessions across Perplexity and other AI platforms.
> **Status:** SKILLS INVENTORY COMPLETE — Ready to author SKILL.md files

---

## 1. What This Hub Is

A single GitHub repo that stores reusable, plain-Markdown skill definitions, project context cards, and standing conventions. At the start of any AI coding session, relevant files from this hub are pasted or referenced to give the AI full context — eliminating the "starting from scratch" problem across projects.

**Primary AI Environment:** Perplexity (LLM-agnostic format — works with any platform accepting system prompts or pasted context)
**Stack:** Vanilla JS / HTML / CSS → GitHub Pages (frontend) + Supabase (PostgreSQL, RLS, Edge Functions, Realtime) + GitHub Projects (management)

---

## 2. Repo Structure (Target)

```
genesis-agent-skill-hub/
├── GENESIS-OUTLINE.md              ← this file
├── AGENTS.md                       ← master context file (paste into any AI session)
├── README.md
├── STACK.md
├── CONVENTIONS.md
├── GLOSSARY.md
│
├── skills/
│   ├── session-handoff/SKILL.md            ← START HERE
│   ├── supabase-schema-migration/SKILL.md
│   ├── supabase-rls-policy/SKILL.md
│   ├── github-pages-deploy/SKILL.md
│   ├── supabase-edge-functions/SKILL.md
│   ├── js-vanilla-frontend/SKILL.md
│   ├── db-driven-content/SKILL.md
│   ├── github-project-management/SKILL.md
│   ├── spec-driven-development/SKILL.md
│   ├── living-documentation/SKILL.md
│   ├── multi-project-context/SKILL.md
│   └── game-supabase-integration/SKILL.md
│
├── references/
│   ├── supabase-checklist.md
│   ├── github-pages-checklist.md
│   ├── security-checklist.md
│   ├── definition-of-done.md
│   └── session-log-template.md
│
├── agents/
│   ├── fullstack-engineer.md
│   ├── db-architect.md
│   └── project-manager.md
│
└── projects/
    ├── _template.md
    └── [project-name].md
```

---

## 3. Confirmed Skills — From Repo Analysis

> Scanned: `chronicle-worlds`, `rfq-course`, `industrial-maintenance-b2b`, `virtual-pinball-wall`, `community-infrastructure-development`, `customer-sales-data-practical` + original proposed list.

### CONFIRMED — Build These

| Priority | Skill | Source Evidence | What It Encodes |
|----------|-------|-----------------|-----------------|
| 1 | `session-handoff` | `industrial-maintenance-b2b/docs/SESSION_HANDOFF.md`, `chronicle-worlds/PROGRESS.md` | Session start/end ritual, living-docs pattern, project state continuity |
| 2 | `supabase-schema-migration` | `chronicle-worlds` (20 migrations), `rfq-course` (8 tables), `customer-sales-data-practical` (csdp schema), `community-infrastructure-development` (schema/) | Migration naming, version sequencing, rollback notes, never-apply-to-prod flags |
| 3 | `supabase-rls-policy` | `chronicle-worlds` (migrations 002, 008, 011–012, 014), `rfq-course` (public read pattern) | RLS on every table, service_role INSERT policies, public SELECT patterns, anon key read |
| 4 | `github-pages-deploy` | `rfq-course` (main branch root deploy), `chronicle-worlds` (gh-pages branch, frontend/** trigger), `community-infrastructure-development` (multi-page static site) | Deploy source config, branch strategy, never edit docs/ directly rule, CORS config |
| 5 | `supabase-edge-functions` | `chronicle-worlds` (`resolve-turn` v9, `discover-cell` v6, versioned ACTIVE/INACTIVE pattern) | Edge Function versioning, ACTIVE tag convention, TypeScript index.ts structure |
| 6 | `js-vanilla-frontend` | `rfq-course` (no build step, hash router, CSS tokens, supabase.js helper), `chronicle-worlds` (Vite + JS), `customer-sales-data-practical` (Phase III CRM) | No-build-step pattern, module structure, createClient() placement, CDN vs npm |
| 7 | `db-driven-content` | `rfq-course` (content in DB, frontend renders statically), `customer-sales-data-practical` (progressive schema + views), `community-infrastructure-development` (Supabase as data layer for Pages site) | Content-in-database architecture, versioned markdown in DB, public read via REST API |
| 8 | `living-documentation` | `chronicle-worlds/DOCUMENTATION-PHILOSOPHY.md`, `chronicle-worlds/PROGRESS.md`, `industrial-maintenance-b2b/docs/SESSION_HANDOFF.md` | PROGRESS.md as single source of truth, milestone log format, quick reference table |
| 9 | `github-project-management` | `industrial-maintenance-b2b` (canonical reference repo structure), `chronicle-worlds` (.github/), `virtual-pinball-wall` (5 open issues, milestone structure) | Repo as canonical reference, SESSION_HANDOFF.md pattern, issue-driven development |
| 10 | `spec-driven-development` | `customer-sales-data-practical` (3-phase project structure), `community-infrastructure-development` (fictitious scenarios with real benchmarks), `virtual-pinball-wall` (objective priority + MVP milestones table) | Spec before code, phased delivery, objective priority order, MVP milestone table format |
| 11 | `living-documentation` | `chronicle-worlds/DOCUMENTATION-PHILOSOPHY.md` | Single source of truth file, milestone log with decisions + next steps, quick reference table |
| 12 | `multi-project-context` | 60+ repos, multiple active Supabase projects (`hhyhulqngdkwsxhymmcd`, `nmemmfblpzrkwyljpmvp`) | Project card format, Supabase project ID tracking, cross-repo references, context switching |
| 13 | `game-supabase-integration` | `chronicle-worlds` (pg_cron, Realtime, leaderboard pattern), `virtual-pinball-wall` (scoreboard + session tracking), `ff-futbol` (physics game + Supabase leaderboard) | pg_cron world tick pattern, Realtime subscription, leaderboard schema, session auth in games |

---

### FROM addyosmani/agent-skills — CONFIRMED ADOPTIONS

These map directly to patterns already in use across your repos.

| Skill | Adopt? | Reason |
|-------|--------|--------|
| `context-engineering` | ✅ ADOPT | Directly solves the insularity problem — this hub IS the solution |
| `incremental-implementation` | ✅ ADOPT | Chronicle-worlds 23-milestone pattern is exactly this |
| `planning-and-task-breakdown` | ✅ ADOPT | MVP milestone tables used in virtual-pinball-wall, customer-sales-data-practical |
| `debugging-and-error-recovery` | ✅ ADOPT | Universal — used across all projects |
| `git-workflow-and-versioning` | ✅ ADOPT | Atomic commits, trunk-based, gh-pages branch pattern in use |
| `security-and-hardening` | ✅ ADOPT | RLS non-negotiable, CORS config, anon key discipline |
| `api-and-interface-design` | ✅ ADOPT | Supabase REST API, Edge Function contracts |
| `documentation-and-adrs` | ✅ ADOPT | You already do ADRs informally (PROGRESS.md decisions) — formalize it |
| `shipping-and-launch` | ✅ ADOPT | GitHub Pages deploy ritual, pre-launch checklist |
| `code-simplification` | ✅ ADOPT | Solo dev — readability and maintainability over cleverness |
| `source-driven-development` | ✅ ADOPT | Supabase docs, MDN, official sources — community-infrastructure-development already cites sources |
| `frontend-ui-engineering` | ✅ ADOPT (lite) | Vanilla JS only — adopt the component + CSS token patterns (rfq-course already uses tokens.css) |
| `performance-optimization` | ⚡ PARTIAL | Static sites on GitHub Pages — adopt Core Web Vitals targets, skip backend profiling |
| `ci-cd-and-automation` | ⚡ PARTIAL | GitHub Actions for Pages deploy — adopt deploy trigger patterns only |
| `deprecation-and-migration` | ⚡ PARTIAL | Multiple active projects — adopt the "code as liability" framing + migration cleanup |
| `browser-testing-with-devtools` | ⚡ PARTIAL | Browser-based apps — adopt console/network patterns, skip DevTools MCP dependency |
| `test-driven-development` | ❌ TRIM | No formal test framework in use across repos — add when a project warrants it |
| `observability-and-instrumentation` | ❌ TRIM | No telemetry stack — revisit if a project adds monitoring |
| `code-review-and-quality` | ❌ TRIM | Solo dev primarily — embedded in `session-handoff` review step instead |
| `doubt-driven-development` | ❌ TRIM | Useful concept but too heavyweight for solo workflow |
| `idea-refine` | ❌ TRIM | Strong ideation already — not needed |
| `interview-me` | ❌ TRIM | Solo context — not applicable |

---

## 4. Key Patterns Identified Across Repos

These are the concrete, repeated patterns to encode directly in SKILL.md files:

### Supabase
- Migration files named `NNN_description.sql` — sequential, never skip numbers
- `ROLLBACK` vs `COMMIT` flags on migration files (chronicle-worlds migration 004 pattern)
- `service_role` for INSERT, `anon` for SELECT — separation is standing rule
- pg_cron for world/data tick jobs (`* * * * *` pattern)
- Supabase project ID + region tracked in every PROGRESS.md
- `REPLICA IDENTITY` set explicitly on tables used with Realtime
- Schema namespacing: `csdp`, `indB2B`, `cid_` prefixes — one schema per project domain

### GitHub Pages
- Two deploy patterns in use: (1) `main` branch root `index.html`, (2) `gh-pages` branch with GitHub Actions trigger on `frontend/**`
- Never edit `docs/` or `gh-pages` directly — source is always `frontend/src/`
- No build step preferred unless Vite is warranted (chronicle-worlds uses Vite; rfq-course does not)
- CDN: `unpkg.com/@supabase/supabase-js@2` for no-build projects

### JS Frontend
- `supabase.js` as dedicated client file with all query helpers
- `tokens.css` → `base.css` → `layout.css` → `components.css` CSS layer pattern (rfq-course)
- Hash-based router (`router.js`) for SPA navigation without a framework
- Auth stored in `sessionStorage` (chronicle-worlds)

### Documentation
- `PROGRESS.md` = single source of truth for dev continuity
- `SESSION_HANDOFF.md` = current work state for agent pickup
- Quick Reference table at bottom of every PROGRESS.md (supabase ID, live URL, branch, keys)
- Milestone format: date, status, migration ref, edge function version, commit SHA

---

## 5. AGENTS.md Template (Master Context File)

```markdown
# Andre's Agent Brain — Genesis Hub

## Identity
Full-stack developer / project manager. JS/HTML/CSS primary. Supabase + GitHub + GitHub Pages stack.
Company: 207 Analytix / Data Solutions for Me — Westbrook, Maine.

## Active Stack
- Frontend: Vanilla JS / HTML / CSS → GitHub Pages (no build step preferred)
- Backend/DB: Supabase (PostgreSQL, RLS, Edge Functions, Realtime, pg_cron)
- Auth: Supabase Auth (magic links, OAuth, sessionStorage)
- Hosting: GitHub Pages (static), Supabase (data/auth/functions)
- AI: Perplexity (primary), LLM-agnostic skill format
- Version Control: GitHub — issues, projects, kanban, milestones
- CDN: unpkg.com/@supabase/supabase-js@2 (no-build projects)

## Standing Rules
- Spec before code
- One slice at a time — implement, verify, commit
- Every schema change gets a numbered migration file (NNN_description.sql)
- RLS is non-negotiable on every Supabase table
- service_role for INSERT, anon for SELECT — never mix
- PROGRESS.md is the single source of truth for every project
- Commit messages: imperative, present tense
- GitHub Issues track all work — no undocumented decisions
- Never edit gh-pages or docs/ directly

## Active Project This Session
→ [paste relevant projects/[project-name].md here]

## Skills Active This Session
→ [paste relevant skills/[skill-name]/SKILL.md here]
```

---

## 6. Project Context Card Template

```markdown
# Project: [Name]

## Identity
- Repo: https://github.com/andredavisme/[repo-name]
- Live URL: https://andredavisme.github.io/[repo-name]/
- Supabase Project: [name] — [project-id] (region: [region])
- Status: [ACTIVE / PAUSED / SHIPPED]

## Stack Notes
- Deploy pattern: [main root / gh-pages branch + Actions]
- Auth method: [magic link / OAuth / none]
- Schema prefix: [e.g. csdp, indB2B, public]
- Key tables: [list]
- RLS: [enabled / partial / none]
- Edge Functions: [list + version or none]
- pg_cron jobs: [list or none]

## Current State
- Last milestone: [description + date]
- Active branch: [branch name]
- Open issues: [count + link]

## Open Decisions
- [ ] [decision needed]

## Next 3 Tasks
1.
2.
3.

## Quick Reference
| Item | Value |
|------|-------|
| Supabase Project ID | |
| Live URL | |
| Deploy trigger | |
| Key migration | |

## Session Log
| Date | What happened |
|------|---------------|
| YYYY-MM-DD | [summary] |
```

---

## 7. Next Steps (In Order)

1. ✅ Create hub repo
2. ✅ Draft GENESIS-OUTLINE.md
3. ✅ Scan repos and confirm skills inventory
4. **[NEXT]** Author `skills/session-handoff/SKILL.md` — highest priority
5. **[NEXT]** Author `AGENTS.md` master context file
6. **[NEXT]** Create `projects/_template.md` and cards for top 5 active projects
7. **[THEN]** Author remaining CONFIRMED skills in priority order
8. **[THEN]** Add `STACK.md`, `CONVENTIONS.md`, `GLOSSARY.md`
9. **[THEN]** Add CONFIRMED addyosmani adoptions (copy + adapt SKILL.md files)

---

*Last updated: 2026-07-06 | Author: Andre Davis | Hub: andredavisme/genesis-agent-skill-hub*
