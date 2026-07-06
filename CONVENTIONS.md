# Conventions

> Standing rules for naming, file structure, commits, and workflow across all projects.
> These apply everywhere unless a project card explicitly overrides them.

---

## File & Directory Naming

- Directories: `lowercase-hyphen`
- Markdown files: `UPPERCASE.md` for root-level docs (README, PROGRESS, AGENTS, etc.)
- SQL migrations: `NNN_description_of_change.sql` (zero-padded 3 digits)
- JS files: `camelCase.js`
- CSS files: `lowercase-hyphen.css`
- HTML files: `lowercase-hyphen.html` (except `index.html`)

## Commit Messages

Format: `type: short imperative description`

| Type | When to use |
|------|-------------|
| `init` | First commit / scaffolding |
| `add` | New feature, file, or content |
| `update` | Change to existing content |
| `fix` | Bug fix |
| `migrate` | Database migration applied |
| `deploy` | Deploy config change |
| `docs` | Documentation only |
| `refactor` | Code restructure, no behavior change |
| `remove` | Delete file or feature |

Examples:
- `add: session-handoff skill`
- `migrate: 021_add_player_inventory`
- `fix: rls policy on materials table`

## Branch Strategy

- `main` — always deployable, source of truth
- `gh-pages` — compiled output only, never edit directly
- Feature branches: `feature/short-description` (optional for solo work)
- Never commit secrets, keys, or credentials to any branch

## SQL Migration Rules

1. Number sequentially: `001`, `002`, `003` — never skip or reuse
2. Every migration file begins with a comment block:
   ```sql
   -- Migration: NNN_description
   -- Date: YYYY-MM-DD
   -- Status: COMMIT | ROLLBACK
   -- Notes: [what this does and why]
   ```
3. `-- ROLLBACK` files are reference-only — never apply to production
4. One logical change per migration file
5. Schema prefix required on all objects: `schema_name.table_name`

## GitHub Issues

- Every piece of work gets an issue before coding starts
- Issue title: imperative ("Add RLS to materials table")
- Close issues with a commit reference
- Labels: `bug`, `feature`, `docs`, `migration`, `deploy`, `blocked`

## PROGRESS.md Format

Every project PROGRESS.md follows this structure:
1. Project overview (stack, Supabase ID, live URL)
2. Architecture / design canon (key decisions)
3. Milestone log (✅ complete, 🔼 next candidates)
4. Quick Reference table (all IDs, URLs, keys, branches)

---

*Last updated: 2026-07-06*
