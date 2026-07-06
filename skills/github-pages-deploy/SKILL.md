---
name: github-pages-deploy
description: Guides setting up, structuring, and deploying static sites to GitHub Pages. Covers both the root (main branch) and docs/ folder deploy patterns, Supabase anon key handling, multi-page routing, favicon, and the pre-launch checklist.
---

# GitHub Pages Deploy Skill

## Overview
GitHub Pages serves a static site (HTML/CSS/JS) directly from a repository. No build step, no bundler — just files. All projects currently use vanilla HTML, inline styles, and ES module imports from CDNs. Deploy is automatic: push to the configured branch/folder and the site updates in ~30 seconds.

## When to Use
- Creating a new project frontend
- Adding a new page to an existing site
- Debugging a deploy that isn't updating or showing errors
- Setting up Pages for the first time on a repo

---

## Two Deploy Configurations in Use

| Project | Source | Branch | Notes |
|---------|--------|--------|-------|
| `maine-civic-tracker` | Root (`/`) | `main` | All HTML files at root; `app.js` + `styles.css` co-located |
| `chronicle-worlds` | `docs/` folder | `main` | Frontend isolated in `docs/`; functions and DB code separate |

Set in: **Repo → Settings → Pages → Source**

> Rule of thumb: use `docs/` when the repo also has backend code (migrations, Edge Functions). Use root when the repo is frontend-only.

---

## Required `<head>` Structure

Every HTML file must include these in `<head>`, in this order:

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Page Title — Site Name</title>
  <link rel="icon" href="data:image/svg+xml,..." />
  <!-- styles: external link OR inline <style> -->
</head>
```

### Favicon
Use an inline SVG data URI — no separate file to manage, no 404:
```html
<link rel="icon" href="data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 32 32'><rect width='32' height='32' fill='%230a0a0f'/><text x='50%25' y='60%25' dominant-baseline='middle' text-anchor='middle' font-size='20'>🌐</text></svg>" />
```
Replace the emoji with whatever fits the project. Use `%23` for `#` and `%25` for `%` inside the data URI.

Real example from `chronicle-worlds` — layered SVG shapes as favicon: [cite:52]
```html
<link rel="icon" href="data:image/svg+xml,<svg xmlns='http://www.w3.org/2000/svg' viewBox='0 0 32 32'><rect width='32' height='32' fill='%230a0a0f'/><ellipse cx='16' cy='20' rx='10' ry='5' fill='%23224422'/><ellipse cx='16' cy='16' rx='6' ry='3' fill='%236688ff'/><ellipse cx='16' cy='12' rx='3' ry='1.5' fill='%238888cc'/></svg>" />
```

> `maine-civic-tracker` currently has no favicon — it's a known gap. Add one when next editing `index.html`.

---

## Supabase Client on GitHub Pages

Two patterns in use. Both work. Choose based on project complexity.

### Pattern A — ES Module (external JS file)
Used by: `maine-civic-tracker` (`app.js`) [cite:51]

```html
<!-- In <head> or before </body> -->
<script type="module" src="app.js"></script>
```

```js
// app.js
import { createClient } from 'https://cdn.jsdelivr.net/npm/@supabase/supabase-js@2/+esm';

const SUPABASE_URL = 'https://yourproject.supabase.co';
const SUPABASE_KEY = 'your-anon-key';
const db = createClient(SUPABASE_URL, SUPABASE_KEY);
```

### Pattern B — UMD Script Tag (inline JS in HTML)
Used by: `chronicle-worlds` (`docs/index.html`) [cite:52]

```html
<script src="https://unpkg.com/@supabase/supabase-js@2/dist/umd/supabase.js"></script>
<script>
  const SUPABASE_URL = 'https://yourproject.supabase.co';
  const SUPABASE_KEY = 'your-anon-key';
  const sb = supabase.createClient(SUPABASE_URL, SUPABASE_KEY, {
    auth: { storage: window.sessionStorage, persistSession: true, autoRefreshToken: true }
  });
</script>
```

> Pattern B allows `auth` options — use it when the frontend needs Supabase Auth (`signInWithPassword`, session handling). Pattern A is cleaner for read-only public data.

### Anon Key in Public Repos
The anon key is intentionally public. It is designed to be embedded in frontend code. It is safe because:
- RLS policies control what it can read/write
- It cannot access service_role operations
- Supabase's URL structure + RLS is the security boundary — not the key itself

Do not treat the anon key as a secret. Do not use `.env` files or GitHub Secrets for it. It belongs in the JS.

---

## Multi-Page Routing

GitHub Pages serves static files. There is no server-side routing. Rules:

- Every page is its own `.html` file: `budget.html`, `overview.html`, etc.
- Links between pages use relative file paths: `href="budget.html"`, not `/budget`
- `index.html` is the default root page
- A shared nav is duplicated across pages (no include system) — keep it in sync manually
- JavaScript routing is done by reading `location.pathname` directly:

```js
// maine-civic-tracker pattern:
const page = () => location.pathname.split('/').pop() || 'index.html';
const p = page();
if (p === 'index.html' || p === '') initHome();
else if (p === 'budget.html') initBudget();
```

---

## Deploy Process

### First-Time Setup
1. Push at least one HTML file to the repo
2. Go to **Repo → Settings → Pages**
3. Set Source: `Deploy from a branch`
4. Set Branch: `main`, Folder: `/` (root) or `/docs`
5. Click Save
6. First deploy takes ~1-2 minutes; subsequent deploys take ~30 seconds

### Every Subsequent Deploy
```bash
git add .
git commit -m "update: description of change"
git push
```
That's it. GitHub Actions handles the rest automatically.

### Verify the Deploy
- Go to **Repo → Actions** tab → confirm the Pages workflow succeeded (green ✓)
- Visit `https://[username].github.io/[repo-name]/`
- If using `docs/`: visit the same URL — GitHub strips the `/docs` prefix automatically

---

## CORS and External Fetch

GitHub Pages itself doesn't add CORS headers — you control the HTML/JS. But when calling Supabase Edge Functions from the frontend:

- The browser sends a preflight OPTIONS request
- The Edge Function must respond with the correct CORS headers (see `supabase-edge-functions` skill)
- If the function returns a non-200 without CORS headers, the browser hides the error — you'll see a network error, not the actual status code
- Test by opening DevTools → Network tab and looking at the OPTIONS preflight response

---

## 404 Handling

GitHub Pages returns its own generic 404 page for missing files. To customize:
- Create a `404.html` at the root (or in `docs/` for docs-source repos)
- GitHub Pages automatically serves it for any unmatched path

Currently no custom 404 pages in active projects — worth adding to any project that has multiple pages.

---

## Watch-Outs From Real Projects

| Issue | Source | What Happened | Fix |
|-------|--------|--------------|-----|
| Pages not updating after push | Any | GitHub Actions workflow failed silently | Check Repo → Actions tab for the Pages deploy job |
| `app.js` not found | maine-civic-tracker | Script tag missing `type="module"` for ESM import | Add `type="module"` to the script tag |
| Supabase calls return empty, no error | Any | Wrong anon key or wrong project URL | Double-check URL and key match the Supabase project |
| Auth session lost on page nav | chronicle-worlds | Default `localStorage` not working in some browsers | Use `sessionStorage` option in `createClient` for auth |
| Page works locally, blank on Pages | Any | Relative path issue — script src or link href uses `/` prefix | Use relative paths (`app.js`, `styles.css`) not absolute (`/app.js`) |
| `gameInitialised` guard missing | chronicle-worlds | `onAuthStateChange` fires twice (INITIAL_SESSION + SIGNED_IN) — channels subscribed twice | Add a `let gameInitialised = false` guard before the heavy init block |
| No favicon | maine-civic-tracker | Browser tab shows blank icon | Add inline SVG data URI favicon to every HTML file's `<head>` |

---

## Pre-Launch Checklist

- [ ] `<!DOCTYPE html>` on line 1
- [ ] `<html lang="en">`
- [ ] `<meta charset="UTF-8" />`
- [ ] `<meta name="viewport" content="width=device-width, initial-scale=1.0" />`
- [ ] `<title>` set to something meaningful (not "Untitled")
- [ ] Favicon present as inline SVG data URI
- [ ] Supabase URL and anon key match the correct project
- [ ] All internal links use relative paths (`page.html`, not `/page`)
- [ ] Every page has the shared nav (if multi-page site)
- [ ] Pages source configured correctly in Repo → Settings → Pages
- [ ] First deploy verified in Repo → Actions tab (green ✓)
- [ ] Site visited in browser with DevTools open — no console errors
- [ ] Network tab checked — no failed Supabase calls on load
- [ ] CORS verified: Edge Function calls return actual errors (not silent network failures)
- [ ] `404.html` created if site has multiple pages
- [ ] PROGRESS.md updated with the live URL

---

## Anti-Rationalizations

| Excuse | Rebuttal |
|--------|----------|
| "The anon key is sensitive — I'll put it in a GitHub Secret" | The anon key is designed to be public. It is in the browser JS by design. RLS is the security layer. Using Secrets for the anon key adds complexity with no security benefit. |
| "I'll add the favicon later" | The favicon is a 1-line addition to `<head>`. "Later" means never. Add it when the file is first created. |
| "Relative paths work on localhost so they'll work on Pages" | Paths with a leading `/` resolve to the GitHub Pages root domain, not the repo subdirectory. Use relative paths everywhere. |
| "I'll just check the site, not the Actions tab" | The site may still show the previous version for minutes after a failed deploy. Always confirm the Actions workflow succeeded. |

---

*Skill version: 1.0 — 2026-07-06 | Source evidence: maine-civic-tracker (index.html, app.js), chronicle-worlds (docs/index.html)*
