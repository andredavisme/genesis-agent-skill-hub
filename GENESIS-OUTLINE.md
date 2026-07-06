# Genesis Materials — Agent Skill Hub
## Master Outline & Reference Document

> **Purpose:** This file is the canonical reference for building out the Genesis Agent Skill Hub — a portable, LLM-agnostic "brain" for coding sessions across Perplexity and other AI platforms.
> **Status:** DRAFT — Skills inventory pending repo analysis

---

## 1. What This Hub Is

A single GitHub repo that stores reusable, plain-Markdown skill definitions, project context cards, and standing conventions. At the start of any AI coding session, relevant files from this hub are pasted or referenced to give the AI full context — eliminating the "starting from scratch" problem across projects.

**Primary AI Environment:** Perplexity (LLM-agnostic format — works with any platform accepting system prompts or pasted context)
**Stack:** Vanilla JS / HTML / CSS → GitHub Pages (frontend) + Supabase (PostgreSQL, RLS, Edge Functions) + GitHub Projects (management)

---

## 2. Repo Structure (Target)

```
genesis-agent-skill-hub/
├── GENESIS-OUTLINE.md          ← this file
├── AGENTS.md                   ← master context file (paste into any AI session)
├── README.md                   ← human-readable hub overview
├── STACK.md                    ← canonical stack declaration
├── CONVENTIONS.md              ← naming, commits, branching, file structure
├── GLOSSARY.md                 ← project-specific terms and abbreviations
│
├── skills/
│   ├── session-handoff/
│   │   └── SKILL.md            ← START HERE: session start/end ritual
│   ├── supabase-schema-design/
│   │   └── SKILL.md
│   ├── supabase-rls-policy/
│   │   └── SKILL.md
│   ├── github-pages-deploy/
│   │   └── SKILL.md
│   ├── js-fullstack-patterns/
│   │   └── SKILL.md
│   ├── auth-flows/
│   │   └── SKILL.md            ← OAuth, magic links, JWT
│   ├── github-project-management/
│   │   └── SKILL.md
│   └── spec-driven-development/
│       └── SKILL.md
│
├── references/
│   ├── supabase-checklist.md
│   ├── github-pages-checklist.md
│   ├── security-checklist.md   ← RLS, CORS, auth edge cases
│   ├── definition-of-done.md
│   └── session-log-template.md
│
├── agents/
│   ├── fullstack-engineer.md   ← primary persona
│   ├── db-architect.md         ← Supabase/PostgreSQL focus
│   └── project-manager.md      ← GitHub Projects / kanban
│
└── projects/
    ├── _template.md            ← blank project context card
    └── [project-name].md       ← one card per active project
```

---

## 3. Skills — Inventory Status

> **Next Step:** Analyze existing repos to identify all patterns and techniques currently in use, then map them to skills. Each row below will be marked CONFIRMED, TRIMMED, or PENDING after that review.

### 3a. Proposed Skills (Pre-Review)

| Priority | Skill | Rationale | Status |
|----------|-------|-----------|--------|
| 1 | `session-handoff` | Encodes existing living-docs pattern — used every session | PENDING |
| 2 | `supabase-schema-design` | DB architecture conventions, migration workflow | PENDING |
| 3 | `supabase-rls-policy` | Auth patterns, row-level security rules | PENDING |
| 4 | `github-pages-deploy` | Static build → gh-pages branch → CORS config | PENDING |
| 5 | `spec-driven-development` | Project kickoff ritual, adapted from addyosmani/agent-skills | PENDING |
| 6 | `auth-flows` | OAuth, magic links, JWT — reused across every app | PENDING |
| 7 | `js-fullstack-patterns` | JS conventions, module patterns, fetch/async patterns | PENDING |
| 8 | `github-project-management` | Issues, milestones, kanban, labels, project cards | PENDING |

### 3b. Skills from addyosmani/agent-skills to Evaluate

Reference: https://github.com/addyosmani/agent-skills/tree/main/skills

| Skill | Relevant To Your Stack? | Status |
|-------|------------------------|--------|
| `context-engineering` | HIGH — directly solves insularity problem | PENDING |
| `incremental-implementation` | HIGH — thin vertical slices match your build style | PENDING |
| `spec-driven-development` | HIGH — project kickoff discipline | PENDING |
| `planning-and-task-breakdown` | HIGH — GitHub Projects workflow | PENDING |
| `debugging-and-error-recovery` | HIGH — universal | PENDING |
| `git-workflow-and-versioning` | HIGH — trunk-based dev, atomic commits | PENDING |
| `test-driven-development` | MEDIUM — depends on project type | PENDING |
| `security-and-hardening` | HIGH — Supabase RLS, CORS, auth | PENDING |
| `api-and-interface-design` | HIGH — Supabase API / Edge Functions | PENDING |
| `frontend-ui-engineering` | MEDIUM — vanilla JS, no framework | PENDING |
| `documentation-and-adrs` | HIGH — you already do living docs | PENDING |
| `ci-cd-and-automation` | MEDIUM — GitHub Actions usage varies | PENDING |
| `code-review-and-quality` | LOW-MEDIUM — solo dev primarily | PENDING |
| `performance-optimization` | MEDIUM — GitHub Pages static sites | PENDING |
| `shipping-and-launch` | HIGH — GitHub Pages deploy ritual | PENDING |
| `observability-and-instrumentation` | LOW — no current telemetry stack | PENDING |
| `deprecation-and-migration` | MEDIUM — multiple active projects | PENDING |
| `browser-testing-with-devtools` | MEDIUM — browser-based apps | PENDING |
| `code-simplification` | HIGH — solo dev, readability matters | PENDING |
| `source-driven-development` | HIGH — authoritative Supabase/JS docs | PENDING |
| `doubt-driven-development` | MEDIUM — high-stakes production decisions | PENDING |
| `idea-refine` | LOW — you already have strong ideation | PENDING |
| `interview-me` | LOW — solo context | PENDING |

---

## 4. AGENTS.md Template (Master Context File)

This is what gets pasted into Perplexity (or any LLM) at the start of a session.

```markdown
# Andre's Agent Brain — Genesis Hub

## Identity
Full-stack developer / project manager. JS/HTML/CSS primary. Supabase + GitHub + GitHub Pages stack.
Company: 207 Analytix / Data Solutions for Me — York Harbor / Westbrook, Maine.

## Active Stack
- Frontend: Vanilla JS / HTML / CSS → GitHub Pages
- Backend/DB: Supabase (PostgreSQL, RLS, Edge Functions, Realtime)
- Auth: Supabase Auth (magic links, OAuth)
- Hosting: GitHub Pages (static), Supabase (data/auth/functions)
- AI: Perplexity (primary), LLM-agnostic skill format
- Version Control: GitHub — issues, projects, kanban, milestones
- Dev Environment: VS Code

## Standing Rules
- Spec before code
- One slice at a time — implement, verify, commit
- Every schema change gets a migration file
- RLS is non-negotiable on every Supabase table
- Commit messages: imperative, present tense ("add auth flow" not "added")
- GitHub Issues track all work — no undocumented decisions

## Active Project This Session
→ [paste relevant projects/[project-name].md here]

## Skills Active This Session
→ [paste relevant skills/[skill-name]/SKILL.md here]
```

---

## 5. Project Context Card Template

Location: `projects/_template.md` — copy and rename per project.

```markdown
# Project: [Name]

## Identity
- Repo: https://github.com/andredavisme/[repo-name]
- Live URL: https://andredavisme.github.io/[repo-name]/
- Supabase Project: [project name / URL]
- Status: [ACTIVE / PAUSED / SHIPPED]

## Stack Notes
- Auth method: [magic link / OAuth / none]
- Tables: [list key tables]
- RLS: [enabled / partial / none]
- Edge Functions: [list or none]

## Current State
- Last milestone: [description]
- Active branch: [branch name]
- Open issues: [count + link]

## Open Decisions
- [ ] [decision needed]

## Next 3 Tasks
1. 
2. 
3. 

## Session Log
| Date | What happened |
|------|---------------|
| YYYY-MM-DD | [summary] |
```

---

## 6. Repo Inventory for Skills Analysis

Repos to scan for patterns and techniques currently in use:

| Repo | Type | Key Patterns Expected |
|------|------|----------------------|
| [chronicle-worlds](https://github.com/andredavisme/chronicle-worlds) | Game / Supabase | PLpgSQL, RLS, realtime |
| [parts-spec-matcher](https://github.com/andredavisme/parts-spec-matcher) | Tool / Supabase | Schema design, fetch patterns |
| [resume-portal](https://github.com/andredavisme/resume-portal) | Pages + Supabase | Auth, GitHub Pages deploy |
| [rfq-course](https://github.com/andredavisme/rfq-course) | Educational / Supabase | DB-driven content, learner state |
| [industrial-network-admin](https://github.com/andredavisme/industrial-network-admin) | Admin / Supabase | CRUD, RLS, admin patterns |
| [personal-ledger-public-display](https://github.com/andredavisme/personal-ledger-public-display) | Finance / Supabase | Public data display, fetch |
| [reddit-mirror](https://github.com/andredavisme/reddit-mirror) | Automation / Supabase | PLpgSQL, automation patterns |
| [RFQ-solutions](https://github.com/andredavisme/RFQ-solutions) | Knowledge base / DB | Schema design, PLpgSQL |
| [pipe-puzzle-builder](https://github.com/andredavisme/pipe-puzzle-builder) | Game / Supabase | Canvas/drag-drop, catalog backend |
| [ff-futbol](https://github.com/andredavisme/ff-futbol) | Game / Supabase | Physics game, leaderboard, sessions |
| [virtual-pinball-wall](https://github.com/andredavisme/virtual-pinball-wall) | Game / GDScript | Godot, physics, modular input |
| [data-solutions-for-me](https://github.com/andredavisme/data-solutions-for-me) | Brand site / Pages | Static site, CSS architecture |
| [the-thread-pdf](https://github.com/andredavisme/the-thread-pdf) | PDF gen / Python | PDF generation workflow |
| [customer-sales-data-practical](https://github.com/andredavisme/customer-sales-data-practical) | Textbook / Pages | Educational content, schema workbook |
| [andremauricedavis.com](https://github.com/andredavisme/andremauricedavis.com) | Personal site / Pages | GitHub Pages, static platform |

---

## 7. Next Steps (In Order)

1. **[IN PROGRESS]** Create this outline file in the hub repo ✓
2. **[NEXT]** Scan top repos to identify all skills/patterns in active use
3. **[NEXT]** Review addyosmani/agent-skills list — CONFIRM or TRIM each entry
4. **[THEN]** Write `session-handoff/SKILL.md` first (highest value)
5. **[THEN]** Write `AGENTS.md` master context file
6. **[THEN]** Create `projects/_template.md` and cards for active projects
7. **[THEN]** Write remaining CONFIRMED skills one by one
8. **[THEN]** Add `STACK.md`, `CONVENTIONS.md`, `GLOSSARY.md`

---

*Last updated: 2026-07-06 | Author: Andre Davis | Hub: genesis-agent-skill-hub*
