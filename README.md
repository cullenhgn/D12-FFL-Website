# Fantasy League Site — Starter (Jekyll + GitHub Pages)

A starter site for your keeper-league to track draft-pick trades ESPN can't,
show historical records, and post news/updates.

## What's in here

```
_config.yml           Site settings — edit title, description, league name
_data/teams.yml        Your league's teams/owners (used everywhere else)
_data/trades.yml        The trade log — this is the "spreadsheet" for trades
_data/records.yml       The record book — same idea, for stats/records
_posts/                 Blog posts / news updates
_layouts/                Page templates (you shouldn't need to touch these)
trades/index.md         Renders _data/trades.yml as a trade tracker page
records/index.md        Renders _data/records.yml as a records page
index.md                Homepage — lists blog posts
assets/css/style.css    All the styling
.github/workflows/pages.yml   Auto-builds & deploys on every push to main
```

## 1. Get this on GitHub

1. Create a new repo on GitHub. If you want it at `https://<you>.github.io`,
   name the repo exactly `<you>.github.io`. Otherwise any name works and
   your site will be at `https://<you>.github.io/<reponame>`.
2. Push these files to the `main` branch of that repo.
3. In the repo, go to **Settings → Pages**, and under "Build and deployment"
   set **Source** to **GitHub Actions**. The included workflow
   (`.github/workflows/pages.yml`) will build and deploy automatically.
4. If your repo is *not* named `<you>.github.io`, set `baseurl` in
   `_config.yml` to `/<reponame>` and `url` to your GitHub Pages URL.

Push to `main` again and check the **Actions** tab — once the workflow run
is green, your site is live.

## 2. Try it locally (optional but recommended)

```bash
gem install bundler
bundle install
bundle exec jekyll serve
```

Visit `http://localhost:4000`. `jekyll serve` auto-rebuilds as you edit files.

## 3. Add your league's data

- **Teams**: edit `_data/teams.yml` — add one entry per owner/team. Every
  other file references teams by their `id`, so set these once.
- **Trades**: edit `_data/trades.yml` — copy the example block for each new
  trade. This is your source of truth for pick trades since ESPN won't
  track them.
- **Records**: edit `_data/records.yml` — add categories and entries as
  history happens (champions, single-week highs, etc.).

These are plain YAML files — structured like rows in a spreadsheet, just in
a text format. GitHub's own web editor lets you edit them directly in the
browser (click the file → pencil icon) without installing anything.

## 4. Post news/updates

Add a file to `_posts/` named `YYYY-MM-DD-a-short-title.md`:

```markdown
---
title: "Draft Recap: The Chaos of Round 6"
---

Your update text here, in Markdown.
```

## 5. Want an actual form instead of editing YAML?

If clicking into YAML files feels too manual, the standard next step is
**Decap CMS** (free, git-based) — it gives you a real web form/admin panel
at `/admin` that commits changes to these same YAML files for you, so
non-technical league members can log a trade or update a record without
touching code. It needs a small one-time OAuth setup since GitHub Pages is
static hosting. I can walk you through adding this once the base site is
live — just ask.

## Notes

- The trades and records pages loop over the data files — you generally
  never need to edit the page templates, just the data.
- `_config.yml` changes require the site to rebuild (the Actions workflow
  handles this automatically on push; locally you need to restart
  `jekyll serve`).
