# Personal Blog (kielsucks.github.io) Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Turn the placeholder `kielsucks.github.io` repo into a Jekyll-powered personal blog with a "Field Notes" visual identity, a single tagged post feed, and a git-based publishing workflow.

**Architecture:** Jekyll site built natively by GitHub Pages from the `main` branch (no CI config). Custom hand-written layout/CSS instead of a stock theme. Tag browsing via plain Liquid (`site.tags`), no plugins beyond what `github-pages`/`jekyll-feed` already whitelist.

**Tech Stack:** Jekyll (via the `github-pages` gem), Ruby/Bundler, Liquid templating, Sass (`.scss`), markdown (kramdown).

## Global Constraints

- Repo: `kielsucks/kielsucks.github.io`, local clone at `~/code/kielsucks.github.io`, work happens on `main` (GitHub Pages serves `main` directly — no separate `gh-pages` branch). Implementation happens in the git worktree at `/Users/kiel/code/kielsucks.github.io/.claude/worktrees/personal-blog` (branch `worktree-personal-blog`) — every `Run:` command below executes there, and merges back to `main` at the end.
- Environment: this machine's system Ruby (2.6, at `/usr/bin/ruby`) is too old for the current `github-pages` gem (its `nokogiri` dependency needs Ruby >= 3.0). Use the Homebrew Ruby installed at `/opt/homebrew/opt/ruby/bin` instead — prefix every `bundle`/`jekyll` command with `export PATH="/opt/homebrew/opt/ruby/bin:$PATH"` (or run it in the same shell invocation). The classic `source "https://pages.github.com/"` Gemfile source is dead (404s) — use `source "https://rubygems.org"` instead.
- Build must work with GitHub Pages' native Jekyll build (`github-pages` gem) — no plugins outside its whitelist. Tag pages use plain Liquid loops over `site.tags`, never `jekyll-archives`.
- Posts live in `_posts/` named `YYYY-MM-DD-title.md`; date comes from the filename, not a front-matter `date:` field.
- Site structure is a single tagged feed (home page = all posts reverse-chron) plus a `/tags.html` grouping page — no separate top-level section pages per topic.
- Visual identity "Field Notes": paper background `#f2f0e9` with a faint graph-paper grid, ink text `#1b1d24`, accent `#e8541e` used sparingly, body/display type in a warm serif stack (`"Iowan Old Style", "Palatino Linotype", Georgia, serif`), metadata (dates/tags) in a monospace stack (`ui-monospace, "SF Mono", "Cascadia Code", Menlo, Consolas, monospace`).
- No CMS, no comments, no custom domain, no RSS styling beyond Jekyll's default `jekyll-feed` output — out of scope for this pass.
- Publishing workflow going forward: add a file to `_posts/`, commit, push to `main`; GitHub Pages rebuilds automatically.

---

### Task 1: Jekyll scaffolding

**Files:**
- Create: `Gemfile`
- Create: `_config.yml`
- Create: `.gitignore`

**Interfaces:**
- Produces: `_config.yml` keys `title`, `description`, `url`, `permalink: /:year/:month/:day/:title/`, `plugins: [jekyll-feed]` — every later task's generated URLs follow this permalink pattern.

- [ ] **Step 1: Write the Gemfile**

```ruby
source "https://rubygems.org"

gem "github-pages", group: :jekyll_plugins
```

- [ ] **Step 2: Write `_config.yml`**

```yaml
title: kielsucks
description: >-
  Beer, maker projects, software, and the occasional word about parenting with ADHD.
url: "https://kielsucks.github.io"
permalink: /:year/:month/:day/:title/
markdown: kramdown
plugins:
  - jekyll-feed
```

- [ ] **Step 3: Write `.gitignore`**

```
_site/
.sass-cache/
.jekyll-cache/
.jekyll-metadata
vendor/
.bundle/
```

- [ ] **Step 4: Install dependencies**

Run: `cd /Users/kiel/code/kielsucks.github.io/.claude/worktrees/personal-blog && export PATH="/opt/homebrew/opt/ruby/bin:$PATH" && bundle config set path 'vendor/bundle' && bundle install`
Expected: gems resolve and install cleanly (~90 gems, takes a minute).

- [ ] **Step 5: Verify the build works**

Run: `cd /Users/kiel/code/kielsucks.github.io/.claude/worktrees/personal-blog && export PATH="/opt/homebrew/opt/ruby/bin:$PATH" && bundle exec jekyll build`
Expected: exits 0, creates a `_site/` directory, and `_site/index.html` exists containing the untouched placeholder text `Welcome to this world` (no layout applied yet since the existing `index.html` has no front matter).

- [ ] **Step 6: Commit**

```bash
git add Gemfile Gemfile.lock _config.yml .gitignore
git commit -m "Add Jekyll scaffolding for GitHub Pages build"
```

---

### Task 2: Base layout and Field Notes stylesheet

**Files:**
- Create: `_layouts/default.html`
- Create: `_includes/header.html`
- Create: `_includes/footer.html`
- Create: `assets/css/style.scss`

**Interfaces:**
- Consumes: `_config.yml` `title` from Task 1.
- Produces: layout name `default` (used by `layout: default` front matter in later tasks), CSS classes `.site-header`, `.site-nav`, `.site-main`, `.post-list`, `.post-card`, `.post-meta`, `.post-title`, `.tags-section` — later tasks' markup must use these exact class names to pick up the styling.

- [ ] **Step 1: Write `_includes/header.html`**

```html
<header class="site-header">
  <a class="site-title" href="{{ '/' | relative_url }}">kielsucks</a>
  <nav class="site-nav">
    <a href="{{ '/' | relative_url }}">feed</a>
    <a href="{{ '/tags.html' | relative_url }}">tags</a>
    <a href="{{ '/about.html' | relative_url }}">about</a>
  </nav>
</header>
```

- [ ] **Step 2: Write `_includes/footer.html`**

```html
<footer class="site-footer">
  <span>&copy; {{ site.time | date: '%Y' }} kielsucks</span>
  <a href="{{ '/feed.xml' | relative_url }}">rss</a>
</footer>
```

- [ ] **Step 3: Write `_layouts/default.html`**

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>{% if page.title %}{{ page.title }} &middot; {% endif %}{{ site.title }}</title>
  <link rel="stylesheet" href="{{ '/assets/css/style.css' | relative_url }}">
</head>
<body>
  {% include header.html %}
  <main class="site-main">
    {{ content }}
  </main>
  {% include footer.html %}
</body>
</html>
```

- [ ] **Step 4: Write `assets/css/style.scss`**

```scss
---
---
:root {
  --paper: #f2f0e9;
  --ink: #1b1d24;
  --accent: #e8541e;
  --grid-line: #c9c6b855;
  --muted: #6b6f76;
  --card-bg: #fbfaf6;
  --card-border: #d8d5c6;
}

* { box-sizing: border-box; }

body {
  margin: 0;
  min-height: 100vh;
  background:
    linear-gradient(var(--grid-line) 1px, transparent 1px) 0 0 / 100% 22px,
    linear-gradient(90deg, var(--grid-line) 1px, transparent 1px) 0 0 / 22px 100%,
    var(--paper);
  color: var(--ink);
  font-family: "Iowan Old Style", "Palatino Linotype", Georgia, serif;
  line-height: 1.6;
}

a { color: var(--ink); }

.site-header, .site-footer {
  display: flex;
  justify-content: space-between;
  align-items: baseline;
  max-width: 720px;
  margin: 0 auto;
  padding: 1.5rem;
  font-family: ui-monospace, "SF Mono", "Cascadia Code", Menlo, Consolas, monospace;
  font-size: 0.8rem;
  text-transform: uppercase;
  letter-spacing: 0.04em;
}

.site-title { font-weight: 700; text-decoration: none; }

.site-nav a {
  margin-left: 1rem;
  text-decoration: none;
  color: var(--muted);
}
.site-nav a:hover { color: var(--accent); }

.site-main {
  max-width: 720px;
  margin: 0 auto;
  padding: 0 1.5rem 3rem;
}

.post-list {
  list-style: none;
  margin: 0;
  padding: 0;
  display: flex;
  flex-direction: column;
  gap: 1rem;
}

.post-card {
  background: var(--card-bg);
  border: 1px solid var(--card-border);
  border-left: 3px solid var(--accent);
  padding: 0.8rem 1rem;
}

.post-meta {
  font-family: ui-monospace, "SF Mono", "Cascadia Code", Menlo, Consolas, monospace;
  font-size: 0.7rem;
  letter-spacing: 0.02em;
  color: var(--accent);
  text-transform: uppercase;
  display: flex;
  gap: 0.75rem;
  margin: 0 0 0.3rem;
}

.post-title { margin: 0; font-size: 1.05rem; }
.post-title a { text-decoration: none; }
.post-title a:hover { text-decoration: underline; }

article.post .post-title { font-size: 1.6rem; }

.tags-section h2 {
  font-family: ui-monospace, "SF Mono", "Cascadia Code", Menlo, Consolas, monospace;
  font-size: 0.85rem;
  text-transform: uppercase;
  letter-spacing: 0.04em;
  color: var(--accent);
  border-bottom: 1px solid var(--card-border);
  padding-bottom: 0.3rem;
}
```

- [ ] **Step 5: Build and verify the stylesheet compiles**

Run: `cd /Users/kiel/code/kielsucks.github.io/.claude/worktrees/personal-blog && export PATH="/opt/homebrew/opt/ruby/bin:$PATH" && bundle exec jekyll build`
Expected: exits 0, and `_site/assets/css/style.css` exists and contains the string `--accent: #e8541e`.

Run: `grep -c '^---$' _site/assets/css/style.css`
Expected: `0` (no literal `---` lines in the compiled output — confirms Jekyll recognized the empty front matter and processed the file through Sass instead of copying it as raw text).

- [ ] **Step 6: Commit**

```bash
git add _layouts/default.html _includes/header.html _includes/footer.html assets/css/style.scss
git commit -m "Add Field Notes base layout and stylesheet"
```

---

### Task 3: Post layout and first post

**Files:**
- Create: `_layouts/post.html`
- Create: `_posts/2026-07-31-welcome.md`

**Interfaces:**
- Consumes: layout `default` from Task 2, CSS classes `.post-meta`, `.post-title` from Task 2.
- Produces: layout name `post` (used by `layout: post` front matter in Task 7's posts), front-matter convention `title`, `tags` (array) for all posts.

- [ ] **Step 1: Write `_layouts/post.html`**

```html
---
layout: default
---
<article class="post">
  <p class="post-meta">
    <span>{{ page.date | date: '%Y.%m.%d' }}</span>
    {% for tag in page.tags %}<span>{{ tag }}</span>{% endfor %}
  </p>
  <h1 class="post-title">{{ page.title }}</h1>
  <div class="post-content">
    {{ content }}
  </div>
</article>
```

- [ ] **Step 2: Write `_posts/2026-07-31-welcome.md`**

```markdown
---
layout: post
title: "Welcome"
tags: [software]
---

This is the first post on what's going to be a running log of the stuff I actually spend my free time on: homebrewing, 3D printing and other maker projects, personal software builds, and the occasional word about parenting with ADHD. No schedule, no pretense — just notes as I go.
```

- [ ] **Step 3: Build and verify the post page renders**

Run: `cd /Users/kiel/code/kielsucks.github.io/.claude/worktrees/personal-blog && export PATH="/opt/homebrew/opt/ruby/bin:$PATH" && bundle exec jekyll build`
Expected: exits 0, and `_site/2026/07/31/welcome/index.html` exists.

Run: `grep -E "Welcome|2026.07.31|software" _site/2026/07/31/welcome/index.html`
Expected: all three strings found — confirms the post layout, date formatting, and tag are rendering.

- [ ] **Step 4: Commit**

```bash
git add _layouts/post.html _posts/2026-07-31-welcome.md
git commit -m "Add post layout and welcome post"
```

---

### Task 4: Home feed

**Files:**
- Modify: `index.html` (replace the 2023 placeholder entirely)

**Interfaces:**
- Consumes: layout `default` from Task 2, CSS classes `.post-list`, `.post-card`, `.post-meta`, `.post-title` from Task 2, `site.posts` populated by Task 3's post.

- [ ] **Step 1: Overwrite `index.html`**

```html
---
layout: default
title: Home
---
<ul class="post-list">
  {% for post in site.posts %}
  <li class="post-card">
    <p class="post-meta">
      <span>{{ post.date | date: '%Y.%m.%d' }}</span>
      {% for tag in post.tags %}<span>{{ tag }}</span>{% endfor %}
    </p>
    <h2 class="post-title"><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h2>
  </li>
  {% endfor %}
</ul>
```

- [ ] **Step 2: Build and verify the feed lists the post**

Run: `cd /Users/kiel/code/kielsucks.github.io/.claude/worktrees/personal-blog && export PATH="/opt/homebrew/opt/ruby/bin:$PATH" && bundle exec jekyll build`
Expected: exits 0.

Run: `grep -c "post-card" _site/index.html`
Expected: `1` (one post so far — the welcome post from Task 3).

Run: `grep "welcome" _site/index.html`
Expected: a match — confirms the home feed links to the welcome post's URL.

- [ ] **Step 3: Commit**

```bash
git add index.html
git commit -m "Replace placeholder index.html with the home feed"
```

---

### Task 5: Tags page

**Files:**
- Create: `tags.html`

**Interfaces:**
- Consumes: layout `default`, CSS classes `.tags-section`, `.post-list`, `.post-card`, `.post-meta`, `.post-title` from Task 2; `site.tags` populated by post front matter.

- [ ] **Step 1: Write `tags.html`**

```html
---
layout: default
title: Tags
permalink: /tags.html
---
{% assign sorted_tags = site.tags | sort %}
{% for tag in sorted_tags %}
<section class="tags-section">
  <h2>{{ tag[0] }}</h2>
  <ul class="post-list">
    {% for post in tag[1] %}
    <li class="post-card">
      <p class="post-meta"><span>{{ post.date | date: '%Y.%m.%d' }}</span></p>
      <h2 class="post-title"><a href="{{ post.url | relative_url }}">{{ post.title }}</a></h2>
    </li>
    {% endfor %}
  </ul>
</section>
{% endfor %}
```

- [ ] **Step 2: Build and verify tag grouping**

Run: `cd /Users/kiel/code/kielsucks.github.io/.claude/worktrees/personal-blog && export PATH="/opt/homebrew/opt/ruby/bin:$PATH" && bundle exec jekyll build`
Expected: exits 0, `_site/tags.html` exists.

Run: `grep -E "software|welcome" _site/tags.html`
Expected: both strings found — confirms the `software` tag section lists the welcome post.

- [ ] **Step 3: Commit**

```bash
git add tags.html
git commit -m "Add tags page grouping posts by tag"
```

---

### Task 6: About page

**Files:**
- Create: `about.md`

**Interfaces:**
- Consumes: layout `default` from Task 2.

- [ ] **Step 1: Write `about.md`**

```markdown
---
layout: default
title: About
permalink: /about.html
---

I'm Kiel — security engineer by day, and this is where I write about the stuff outside of that: homebrewing, 3D printing and other maker projects, personal software projects, and life with ADHD as a parent.
```

- [ ] **Step 2: Build and verify**

Run: `cd /Users/kiel/code/kielsucks.github.io/.claude/worktrees/personal-blog && export PATH="/opt/homebrew/opt/ruby/bin:$PATH" && bundle exec jekyll build`
Expected: exits 0, `_site/about.html` exists.

Run: `grep "security engineer" _site/about.html`
Expected: a match.

- [ ] **Step 3: Commit**

```bash
git add about.md
git commit -m "Add about page"
```

---

### Task 7: Seed posts across all topics

**Files:**
- Create: `_posts/2026-07-20-ender-bed-adhesion.md`
- Create: `_posts/2026-07-24-batch-14-dry-hop.md`
- Create: `_posts/2026-07-27-loud-house.md`

**Interfaces:**
- Consumes: layout `post` from Task 3, front-matter convention (`layout`, `title`, `tags`) from Task 3.
- Produces: four total posts across tags `software`, `maker`, `beer`, `adhd-parenting` for Task 8's deploy check.

- [ ] **Step 1: Write `_posts/2026-07-20-ender-bed-adhesion.md`**

```markdown
---
layout: post
title: "Fixing warping on the Ender's PLA bed adhesion"
tags: [maker]
---

Bed at 60°C, glue stick, and it's still lifting at the corners on anything over 100mm. Next: dialing in first-layer squish before I reach for a brim.
```

- [ ] **Step 2: Write `_posts/2026-07-24-batch-14-dry-hop.md`**

```markdown
---
layout: post
title: "Batch 14: dry-hopping the hazy IPA at day 3"
tags: [beer]
---

Citra and mosaic at 60/40, added on day 3 of fermentation instead of my usual day 5 — gravity was already down to 1.014 and I wanted to catch more of the biotransformation window. Two weeks until I know if that was the right call.
```

- [ ] **Step 3: Write `_posts/2026-07-27-loud-house.md`**

```markdown
---
layout: post
title: "What actually works when the whole house is loud"
tags: [adhd-parenting]
---

Noise-canceling headphones for me, not the kids — that was the surprising part. Writing up what's actually helped versus what just felt like it should.
```

- [ ] **Step 4: Build and verify all four posts show up**

Run: `cd /Users/kiel/code/kielsucks.github.io/.claude/worktrees/personal-blog && export PATH="/opt/homebrew/opt/ruby/bin:$PATH" && bundle exec jekyll build`
Expected: exits 0.

Run: `grep -c "post-card" _site/index.html`
Expected: `4`.

Run: `grep -c "tags-section" _site/tags.html`
Expected: `4` (one section each for `adhd-parenting`, `beer`, `maker`, `software`).

- [ ] **Step 5: Commit**

```bash
git add _posts/2026-07-20-ender-bed-adhesion.md _posts/2026-07-24-batch-14-dry-hop.md _posts/2026-07-27-loud-house.md
git commit -m "Seed starter posts for beer, maker, and adhd-parenting tags"
```

---

### Task 8: Deploy and verify live

**Files:**
- None (push existing commits; no new files).

**Interfaces:**
- Consumes: all committed work from Tasks 1–7.

- [ ] **Step 1: Push to GitHub**

Run: from `/Users/kiel/code/kielsucks.github.io` (the original clone, not the worktree): merge the `worktree-personal-blog` branch into `main` via superpowers:finishing-a-development-branch, then `git push origin main`
Expected: pushes cleanly (no conflicts — the repo only had the placeholder before Task 1).

- [ ] **Step 2: Confirm the Pages build succeeded**

Run: `gh api repos/kielsucks/kielsucks.github.io/pages/builds/latest`
Expected: JSON response with `"status": "built"`. If `"status": "errored"`, read the `"error"` field for the Jekyll build error and fix it before continuing.

- [ ] **Step 3: Verify the live site**

Run: `curl -s https://kielsucks.github.io/ | grep -c "post-card"`
Expected: `4` (matches the local build from Task 7 — confirms the deployed site matches what was pushed).

Run: `curl -s https://kielsucks.github.io/tags.html | grep -c "tags-section"`
Expected: `4`.

- [ ] **Step 4: No commit needed — this task only verifies the deploy.**
