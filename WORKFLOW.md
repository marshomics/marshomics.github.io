# Workflow: forking, customizing, and pulling upstream updates

This site is a customized fork of [academicpages/academicpages.github.io](https://github.com/academicpages/academicpages.github.io). The customizations are deliberately confined to a small number of files so that you can keep pulling updates from the upstream template without losing your changes.

## One-time setup

### 1. Fork on GitHub

Go to <https://github.com/academicpages/academicpages.github.io> and click **Use this template → Create a new repository** (this is preferred over a regular fork because it gives you a clean history; the upstream remote will still let you merge updates).

Name the new repo **`marshomics.github.io`**. The exact name matters — GitHub Pages serves user repos at `<username>.github.io` only when the repo is named that way.

### 2. Push these customizations into your new repo

On your local machine:

```bash
# Clone your empty new repo
git clone https://github.com/marshomics/marshomics.github.io.git
cd marshomics.github.io

# Copy the customized files from this Cowork workspace folder into the clone.
# Easiest: drag-drop in Finder, or rsync:
rsync -av --exclude='.git' --exclude='WORKFLOW.md' \
  /Users/jmarsh/Documents/cowork/archaea_defense_antidefense/website/ ./

# Add the original template as an "upstream" remote so you can pull updates later
git remote add upstream https://github.com/academicpages/academicpages.github.io.git
git remote -v   # should show origin (your fork) and upstream (the template)

# Commit and push
git add -A
git commit -m "Initial customization from academicpages template"
git push origin master   # or main, depending on your default branch
```

GitHub Pages will start building automatically. Within a few minutes the site will be live at `https://marshomics.github.io`.

### 3. Enable GitHub Pages (if not already on)

Settings → Pages → Source: **Deploy from a branch** → Branch: `master` (or `main`), folder: `/ (root)` → Save.

## Pulling upstream updates later

When the academicpages maintainers push improvements you want:

```bash
git fetch upstream
git checkout master
git merge upstream/master
```

If there are conflicts, they'll be in a small, predictable set of files. Resolve, commit, push.

### Files where conflicts may occur (and how to resolve)

| File | Likely conflict | Resolution |
|---|---|---|
| `_config.yml` | Upstream adds new options or renames fields | Keep your values; merge in any new keys |
| `_data/navigation.yml` | Upstream adds new nav items | Decide whether you want them; usually keep yours |
| `_pages/about.md` | Upstream rewrites the demo home page | **Always keep yours.** This is your lab front page. |
| `_pages/people.html` | New file (yours) | No upstream conflict — this page exists only here. |
| `_pages/news.html`, `_pages/join.md` | New files (yours) | No upstream conflict. |
| `_pages/cv.md`, `publications.html`, etc. | Minor structural updates | Usually keep yours; cherry-pick structural improvements if useful |
| `assets/css/main.scss` | Upstream adds new SCSS partials to the import list | Merge: keep upstream's new imports AND your `"overrides"` line at the end |
| `_includes/head/custom.html` | Upstream adds new head snippets (favicons etc.) | Merge: keep upstream's snippets AND your font imports |
| `_includes/sidebar.html` | Upstream restructures the sidebar | Merge: keep your two added lines (the `bsky_sidebar` flag check and the `{% include bsky-feed-compact.html %}` line) |
| `_sass/_overrides.scss` (the `#main { display: grid; ... }` block) | Upstream changes the layout of `#main` | Merge: keep the grid override block; it's required for the column-bg differentiation. The override sets `display: grid`, fixed column widths, and applies `cream-soft` / `paper` backgrounds to `.sidebar` and `.page`/`.archive`. Without it, those backgrounds wouldn't fill the column heights. |

### Files that should rarely conflict

These are touched only by you, not by upstream:

- `_sass/_overrides.scss` — the entire custom theme layer
- `_people/` — entire new collection (lab members), unique to this site
- `_pages/people.html`, `_pages/news.html`, `_pages/join.md` — new pages, no upstream equivalents
- Your real content files in `_publications/`, `_portfolio/`, `_posts/` (once you replace the placeholders)
- `images/lab-logo.png`, `images/people/*.jpg`, the PI's CV PDF in `files/`, etc.

### Files where you should generally take upstream's version

These are theme internals — accept upstream changes wholesale:

- `_sass/layout/*`, `_sass/include/*`, `_sass/vendor/*`, `_sass/theme/*`
- `_layouts/*` (unless you've overridden a specific one — currently none)
- `_includes/*` (except `head/custom.html`, see above)
- `assets/js/*`, `assets/css/academicons.css`, `assets/css/collapse.css`
- `Gemfile`, `Dockerfile`, `_config_docker.yml`

## The customization architecture

This is a **lab site** (Marsh Lab), not a personal academic page. The structure differs from stock academicpages in three ways:

1. **A new `_people` collection** for lab members. Each person is a markdown file in `_people/` with `role`, `status` (`current` or `alumni`), `order`, `photo`, `links`. Rendered by `_pages/people.html` as a grid for current members + a list for alumni.
2. **`_posts/` is repurposed as lab news.** Rendered by `_pages/news.html`.
3. **A new `_pages/join.md`** for prospective members.

Heavy theming is contained in **one file**: `_sass/_overrides.scss`. It's wired in by appending `"overrides"` to the import list in `assets/css/main.scss` — that's the only file in the SCSS chain you've modified.

The `_overrides.scss` layer:
- Defines design tokens as CSS custom properties (`--jm-forest`, `--jm-ochre`, etc.)
- Loads Fraunces (serif headings) and Inter (body) via Google Fonts in `_includes/head/custom.html`
- Restyles masthead, sidebar, archive lists, buttons, footer
- Provides `.jm-hero` and `.jm-highlight` for the front page
- Provides `.jm-people` / `.jm-person` for the People page grid
- Provides `.jm-alumni` for the alumni list
- Provides `.jm-news` for the News list

Front page motif: the phylogenetic-tree SVG is **inlined** in `_pages/about.md`. No external SVG file to track, no extra HTTP request.

## Bluesky feed

The home page includes a live Bluesky feed (`_includes/bsky-feed.html`) that fetches recent posts client-side from `public.api.bsky.app`. No auth, no API key, no build-time step — fresh on every page load.

To enable it: set `author.bluesky` in `_config.yml` to your handle without the `@`, e.g.

```yaml
author:
  ...
  bluesky: "marshlab.bsky.social"
```

The handle can be either the default `*.bsky.social` form or a custom domain (`marsh-lab.org`, etc.). Until you set a handle, the section shows a quiet "configure your handle" message rather than rendering broken.

The feed shows the 5 most recent original posts (replies are filtered out by the API call's `filter=posts_no_replies` parameter). Reposts are kept and labelled. URLs in post text are auto-linked. To change the count or filter, edit the `data-limit` attribute and the `endpoint` URL in `_includes/bsky-feed.html`.

If the API is down or rate-limits you, the feed shows a graceful "temporarily unavailable" message with a direct link to your Bluesky profile — the page never breaks.

### Compact sidebar version

A second, smaller variant lives at `_includes/bsky-feed-compact.html` and shows the 3 most recent posts in the left sidebar. It's enabled per-page by adding `bsky_sidebar: true` to the front matter — currently set on the home page only (`_pages/about.md`). To put it on other pages, add the flag there too. To remove it from the home page, delete that one front-matter line.

The conditional include lives in `_includes/sidebar.html`, which is the only upstream `_includes/` file you've modified. The change is two lines (a flag check in the outer `if` and the `{% include bsky-feed-compact.html %}` line). On future upstream merges, if `sidebar.html` conflicts, keep your two added lines and accept everything else from upstream.

## Cleaning up the example content

The template ships with example publications, talks, teaching, portfolio, and posts. They've been replaced with `EXAMPLE` placeholder content. After cloning to your machine, run:

```bash
# Delete all example placeholders
git rm _publications/*.md _talks/*.md _teaching/*.md _portfolio/*
git rm _posts/2012-*.md _posts/2013-*.md _posts/2014-*.md _posts/2015-*.md _posts/2199-*.md
git rm _people/example-*.md      # keep _people/marsh-james.md, fill in real members
```

Then add real content one file at a time using the placeholders as templates. Patterns:

- **Lab members** → `_people/firstname-lastname.md` with `status: current` or `status: alumni`, integer `order:` controlling display order
- **News items** → `_posts/YYYY-MM-DD-short-slug.md`
- **Publications** → `_publications/YYYY-MM-DD-short-slug.md` with `category: manuscripts | preprints | conferences`
- **Research projects** → `_portfolio/short-slug.md`

## Local preview

To preview the site on your machine before pushing:

```bash
bundle install              # one-time, installs Jekyll + plugins
bundle exec jekyll serve -l # serves at http://localhost:4000 with live reload
```

Or use the Docker setup:

```bash
docker compose up
```

## Getting the front page to look right

The hero section depends on:
- `_sass/_overrides.scss` being imported (check `assets/css/main.scss` last line is `"overrides"`)
- Google Fonts loading (check `_includes/head/custom.html` has the `<link>` to fonts.googleapis.com)
- The inline SVG in `_pages/about.md` rendering correctly (no curly-brace conflicts with Liquid; the SVG uses no `{{ }}` syntax so it's safe)

If headings look like Times New Roman after deployment, the Google Fonts link is being blocked or hasn't loaded — check the browser network tab.

## Adding a lab logo and member photos

- **Lab logo / sidebar image** → drop a square image at `images/lab-logo.png` (the `avatar` field in `_config.yml` points at this filename). For a lab site this is usually a wordmark, group photo, or visual identity rather than a face.
- **Member photos** → drop square JPGs at `images/people/firstname-lastname.jpg` and reference them in each `_people/*.md` file's `photo:` field. The grid card uses `aspect-ratio: 1/1`, so square crops look best.

## Filling in the rest

The `_config.yml` has placeholder fields for: lab `email`, `location` (department + institution), `employer`, `googlescholar`, `orcid`, `linkedin`, `bluesky`, etc. Search for `# fill in` or empty values. The sidebar hides any field you leave blank.

The `_pages/join.md` has placeholders like *DATE*, *NUMBER OF POSITIONS*, *PROGRAM NAME* — search for ALL-CAPS terms wrapped in asterisks and fill them in.
