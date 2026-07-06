# GitHub Pages Checklist

> Quick-reference for deploying and maintaining GitHub Pages sites.

---

## New Site Setup
- [ ] Decide deploy pattern (A or B — see STACK.md)
- [ ] **Pattern A:** Settings → Pages → Deploy from branch → `main` / `/ (root)`
- [ ] **Pattern B:** GitHub Actions workflow targeting `frontend/**`, output to `gh-pages` branch
- [ ] `index.html` at correct root location
- [ ] Supabase anon key in JS only — never in a server context
- [ ] CORS configured in Supabase Edge Functions for the Pages URL
- [ ] Live URL confirmed in PROGRESS.md Quick Reference

## Deploy Rules
- [ ] Never edit `gh-pages` branch directly
- [ ] Never edit `docs/` folder directly (if used as Pages source)
- [ ] Source is always `frontend/src/` for Pattern B projects
- [ ] Test locally (open `index.html` in browser) before pushing

## Pre-Launch Gate
- [ ] Live URL loads correctly
- [ ] Supabase queries return data (check browser console)
- [ ] No 404s on assets (CSS, JS, images)
- [ ] Auth flow tested end-to-end if auth is used
- [ ] CORS errors absent from browser console

---

*Last updated: 2026-07-06*
