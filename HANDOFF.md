# Handoff: Hugo + PaperMod on GitHub Pages

This repo is a working template for a Hugo site using the PaperMod theme, deployed via GitHub Actions to GitHub Pages. No Netlify, no server access needed.

Live site: https://datwins.github.io/owl_wisdom/

---

## How it works

Every push to `main` triggers `.github/workflows/hugo.yml`, which:
1. Installs Hugo (version pinned in the workflow)
2. Checks out the repo including the PaperMod theme submodule
3. Builds the site with `hugo --gc --minify`
4. Deploys the output to GitHub Pages

The site is live at `https://DatWins.github.io/REPO-NAME/` within a minute or two of each push.

---

## Use this as a template for a new site

### Step 1 — Create a new repo on GitHub
- Click **+ → New repository**
- Leave README, .gitignore, and license **unchecked**
- Copy the repo URL

### Step 2 — Clone this repo and point it at the new one
```
git clone --recurse-submodules https://github.com/DatWins/owl_wisdom NEW-SITE-NAME
cd NEW-SITE-NAME
git remote set-url origin https://github.com/DatWins/NEW-SITE-NAME.git
git push -u origin main
```

### Step 3 — Update config.yml
Edit these lines for the new site:
```yaml
baseURL: "https://datwins.github.io/NEW-SITE-NAME/"
title: Your Site Title
```
Also update `description`, `author`, `googleAnalytics`, and the `profileMode` section.

### Step 4 — Enable GitHub Pages
In the new repo: **Settings → Pages → Build and deployment → Source → GitHub Actions**

### Step 5 — Push and watch it build
```
git add config.yml
git commit -m "Set site title and baseURL"
git push
```
Go to the **Actions** tab. Green check = site is live.

---

## Daily workflow

```
hugo server -D          # preview at http://localhost:1313
git add .
git commit -m "Describe the change"
git push
```

---

## Key files

| File | Purpose |
|---|---|
| `config.yml` | Site title, URL, theme settings, menu |
| `.github/workflows/hugo.yml` | Builds and deploys on every push to main |
| `.gitmodules` | Points to the PaperMod theme on GitHub |
| `themes/PaperMod/` | Theme submodule (do not edit files here) |
| `layouts/partials/comments.html` | Custom Remark42 comment widget |
| `content/` | Your posts and pages (Markdown files) |
| `static/` | Images and other static assets |

---

## Lessons learned migrating from Netlify

These are the issues that came up and how they were fixed, so you don't hit them again.

**1. Workflow folder named `workflow` instead of `workflows`**
GitHub only reads `.github/workflows/` (with the s). Nothing runs if the folder is spelled wrong.
Fix: rename the folder.

**2. `hugo.yml` file had backtick fences at top and bottom**
The file was created by copy-pasting from instructions that included the ` ```yaml ` code-block markers. Those are documentation formatting, not part of the YAML. The file must start with `name:`, not ` ```yaml `.
Fix: delete the first and last line of the file.

**3. Hugo version too old for the theme**
PaperMod requires a newer Hugo than the original `0.140.0` in the workflow. The error was `no such template "_internal/google_news.html"`.
Fix: raise `HUGO_VERSION` in `.github/workflows/hugo.yml` to `0.159.2` (or the latest at https://github.com/gohugoio/hugo/releases).

**4. Deprecated pagination config**
Hugo 0.128+ replaced `paginate` and `paginatePath` with a `pagination:` block.
Fix in `config.yml`:
```yaml
# Old (broken):
paginate: 10
paginatePath: blog

# New (correct):
pagination:
  pagerSize: 10
  path: blog
```

**5. PaperMod theme submodule was outdated**
The old theme version called `_internal/google_news.html` which no longer exists.
Fix: update the submodule: `git submodule update --init --remote themes/PaperMod`, then commit.

**6. Custom `layouts/partials/footer.html` used deprecated Hugo syntax**
The custom footer used `slice (resources.Get ...) | resources.Concat` which broke in Hugo 0.128+. The updated PaperMod theme already has a modern footer, so the custom file was not needed.
Fix: delete `layouts/partials/footer.html` entirely.

**7. `baseURL` missing trailing slash**
Without the trailing slash, Hugo builds asset paths incorrectly on a subpath site like `/owl_wisdom/`.
Fix: `baseURL: "https://datwins.github.io/owl_wisdom/"` (note the `/` at the end).

**8. `relativeURLs: true` broke CSS on GitHub Pages subpath**
Relative URLs cause CSS and JS links to break when the site is served from a subdirectory.
Fix: set `relativeURLs: false` in `config.yml`.

---

## Updating the PaperMod theme

When PaperMod releases fixes, update it:
```
git submodule update --remote themes/PaperMod
git add themes/PaperMod
git commit -m "Update PaperMod theme"
git push
```

---

## Checking the Hugo version

To see the latest Hugo release: https://github.com/gohugoio/hugo/releases

To update the version used in builds, edit this line in `.github/workflows/hugo.yml`:
```yaml
HUGO_VERSION: 0.159.2
```
