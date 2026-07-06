---
name: session-handoff
description: Guides the start and end of every AI coding session. Loads project context, confirms standing rules, and closes with a handoff update. Use at the beginning and end of every session.
---

# Session Handoff Skill

## Overview
Every AI coding session starts cold. This skill ensures the AI and developer are fully aligned on project state before any code is written, and that the project is left in a resumable state when the session ends.

## When to Use
- **Session start:** Before writing any code or making any decisions
- **Session end:** Before closing the conversation
- **Context switch:** When switching between projects mid-session

## Process

### SESSION START
1. Paste `AGENTS.md` into the conversation
2. Paste the relevant `projects/[name].md` context card
3. Paste any `skills/[name]/SKILL.md` files relevant to today's work
4. State the session goal in one sentence
5. Confirm: does the AI acknowledge the standing rules and current project state?
6. If not — clarify before proceeding

### DURING SESSION
- Reference `PROGRESS.md` before making any architectural decision
- Every decision gets logged — no undocumented choices
- Commit after each verified slice (not at end of session)
- Flag open questions explicitly rather than assuming

### SESSION END
1. Update `PROGRESS.md` with a new milestone entry (date, what was done, commit SHA)
2. Update `SESSION_HANDOFF.md` with current work state and next steps
3. Update the project card in `projects/[name].md` (last milestone, next 3 tasks)
4. Confirm all commits are pushed
5. Close any resolved GitHub Issues
6. Note any open decisions or blockers in the handoff doc

## Verification Gates
- [ ] AGENTS.md was pasted at session start
- [ ] Project card was loaded
- [ ] Session goal was stated
- [ ] PROGRESS.md was updated with milestone entry
- [ ] SESSION_HANDOFF.md reflects current state
- [ ] All commits pushed
- [ ] Next 3 tasks are documented

## Anti-Rationalizations

| Excuse | Rebuttal |
|--------|----------|
| "I'll update PROGRESS.md next session" | Next session starts cold. If it's not written, it's lost. |
| "It was a small change, no need to log it" | Small undocumented changes cause the biggest debugging sessions. |
| "I know what I was doing, I don't need a handoff" | You are not the only agent who will work on this. |

---

*Skill version: 1.0 — 2026-07-06*
