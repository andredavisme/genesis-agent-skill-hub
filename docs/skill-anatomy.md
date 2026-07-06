# Skill Anatomy Guide

> How to write a SKILL.md file for this hub.
> Follow this format for every skill — custom and adopted.

---

## File Location

```
skills/[skill-name]/SKILL.md
```

One directory per skill. The `SKILL.md` is the only required file. Supporting references go in `references/`.

---

## SKILL.md Format

```markdown
---
name: lowercase-hyphen-name
description: One sentence. Guides [what]. Use when [trigger condition].
---

# [Skill Title]

## Overview
[2-3 sentences. What this skill does and why it exists.]

## When to Use
- [Condition 1]
- [Condition 2]

## Process
[Numbered steps. Be specific. Each step should be actionable.]
1.
2.
3.

## Verification Gates
- [ ] [Checkable condition that proves the step is done]
- [ ] ...

## Anti-Rationalizations

| Excuse | Rebuttal |
|--------|----------|
| [Common excuse to skip this skill] | [Why that's wrong] |

---

*Skill version: 1.0 — YYYY-MM-DD*
```

---

## Rules for Writing Skills

1. **Process, not prose.** Steps to follow, not concepts to understand.
2. **Verification gates are mandatory.** Every skill must end with checkable evidence.
3. **Anti-rationalizations are mandatory.** Include at least 2 common excuses and rebuttals.
4. **Minimal.** Only what's needed to guide the session. If it's reference material, put it in `references/`.
5. **Version it.** Increment the version number when the process changes significantly.

---

## Updating the Inventory

After creating a new skill, update the skills table in `GENESIS-OUTLINE.md`:
- Change status from `PENDING` to `CONFIRMED`
- Add the file link

---

*Last updated: 2026-07-06*
