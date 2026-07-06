# Genesis Materials — Agent Skill Hub

> The portable AI brain for coding through Perplexity and other LLM platforms.

**Owner:** [André Maurice Davis](https://github.com/andredavisme) — 207 Analytix 
**Stack:** Vanilla JS / HTML / CSS → GitHub Pages + Supabase (PostgreSQL, RLS, Edge Functions, Realtime)

---

## How to Use This Hub

### Starting an AI Session
1. Open `AGENTS.md` — copy the full contents
2. Paste into your Perplexity (or any LLM) conversation as your first message
3. Find the relevant project card in `projects/` — paste it in too
4. Paste any skill files from `skills/` that apply to today's work
5. Code with full context loaded

### Adding a New Project
1. Copy `projects/_template.md`
2. Rename to `projects/[repo-name].md`
3. Fill in the identity, stack notes, and current state
4. Update at the end of every session

### Adding or Updating a Skill
1. Create `skills/[skill-name]/SKILL.md`
2. Follow the anatomy in `docs/skill-anatomy.md`
3. Update the inventory table in `GENESIS-OUTLINE.md`

---

## Structure at a Glance

```
genesis-agent-skill-hub/
├── AGENTS.md                   ← paste this into every AI session
├── GENESIS-OUTLINE.md          ← master plan and skills inventory
├── STACK.md                    ← canonical stack reference
├── CONVENTIONS.md              ← naming, commits, structure rules
├── GLOSSARY.md                 ← terms and abbreviations
├── skills/                     ← reusable workflow skill files
├── references/                 ← checklists pulled in by skills
├── agents/                     ← specialist AI personas
├── projects/                   ← per-project context cards
└── docs/                       ← hub maintenance guides
```

---

*Last updated: 2026-07-06*
