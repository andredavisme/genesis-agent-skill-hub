# How to Use This Hub

> Step-by-step guide for getting value from the Genesis Agent Skill Hub.

---

## Starting an AI Session

1. Open `AGENTS.md` — copy everything
2. Paste into Perplexity (or any LLM) as your first message
3. Open `projects/[your-project].md` — copy and paste it into the same message
4. Identify which skills apply to today's work — paste each `SKILL.md`
5. State your session goal in one sentence
6. Proceed

**Minimal session start (quick work):**
Paste just `AGENTS.md` + project card. Skip skills if it's a quick fix.

**Full session start (new feature or architecture):**
Paste `AGENTS.md` + project card + all relevant skills + the relevant persona from `agents/`.

---

## Ending a Session

1. Update `PROGRESS.md` in the project repo with a new milestone entry
2. Update `SESSION_HANDOFF.md` in the project repo
3. Update `projects/[name].md` in THIS hub (last milestone + next 3 tasks)
4. Push all commits
5. Close resolved GitHub Issues

---

## Adding a New Project

1. Copy `projects/_template.md`
2. Rename to `projects/[repo-name].md`
3. Fill in all fields
4. Commit to this hub

---

## Adding a New Skill

1. Read `docs/skill-anatomy.md` first
2. Create `skills/[skill-name]/SKILL.md`
3. Follow the anatomy format exactly
4. Update the skills inventory in `GENESIS-OUTLINE.md`
5. Commit

---

## Updating an Existing Skill

1. Edit `skills/[skill-name]/SKILL.md`
2. Increment the version number at the bottom
3. Note what changed in the commit message
4. If the change is significant, note it in `GENESIS-OUTLINE.md`

---

## Keeping the Hub Healthy

- **Weekly (if active):** Check that project cards reflect current state
- **Per project:** Update the project card after every session
- **Per new pattern:** If you solve something new, encode it as a skill or reference file
- **Per retired project:** Mark project card status as SHIPPED or PAUSED

---

*Last updated: 2026-07-06*
