# Andre's Agent Brain — Genesis Hub

> Paste this file into any AI session (Perplexity, or any LLM) to load full context.
> Then paste the relevant `projects/[name].md` and any `skills/[name]/SKILL.md` files below.

---

## Identity

Full-stack developer / project manager. JS/HTML/CSS primary. Supabase + GitHub + GitHub Pages stack.
- **Company:** 207 Analytix / Data Solutions for Me
- **Location:** Westbrook, Maine
- **GitHub:** [andredavisme](https://github.com/andredavisme)

---

## Active Stack

| Layer | Technology |
|-------|------------|
| Frontend | Vanilla JS / HTML / CSS (no build step preferred) |
| Backend/DB | Supabase — PostgreSQL, RLS, Edge Functions, Realtime, pg_cron |
| Auth | Supabase Auth — magic links, OAuth, sessionStorage |
| Hosting | GitHub Pages (static frontend) |
| CDN | unpkg.com/@supabase/supabase-js@2 (no-build projects) |
| Version Control | GitHub — issues, projects, kanban, milestones |
| AI | Perplexity (primary) — LLM-agnostic skill format |
| Game Dev | Godot / GDScript (select projects) |
| PDF Gen | Python (select projects) |

---

## Standing Rules

### Code
- Spec before code — always
- One slice at a time: implement → verify → commit
- No build step unless the project warrants it (Vite only if justified)
- CDN for Supabase client in no-build projects

### Database
- Every schema change gets a numbered migration file: `NNN_description.sql`
- RLS is non-negotiable on every Supabase table
- `service_role` for INSERT/UPDATE/DELETE; `anon` for SELECT — never mix
- Schema namespacing required: one prefix per project domain (e.g. `csdp`, `indB2B`, `cid_`)
- Flag migrations: `-- COMMIT` or `-- ROLLBACK` — never apply ROLLBACK files to production
- Set `REPLICA IDENTITY FULL` on tables used with Realtime

### GitHub / Deploy
- Never edit `gh-pages` branch or `docs/` directly — source is always `frontend/src/`
- Commit messages: imperative, present tense ("add auth flow" not "added auth flow")
- GitHub Issues track all work — no undocumented decisions
- `PROGRESS.md` is the single source of truth for every project

### Documentation
- Every project has a `PROGRESS.md` (dev continuity) and `SESSION_HANDOFF.md` (agent pickup)
- Every milestone entry includes: date, status, migration ref, edge function version, commit SHA
- Quick Reference table at the bottom of every `PROGRESS.md`

---

## Active Project This Session

> Replace this block with the contents of `projects/[project-name].md`

```
[PASTE PROJECT CARD HERE]
```

---

## Skills Active This Session

> Paste the SKILL.md content for any skill relevant to today's work.

```
[PASTE SKILL(S) HERE]
```
