# Personal blog on kielsucks.github.io — design

## Purpose

Turn the placeholder `kielsucks.github.io` (a 22-byte "Welcome to this world" `index.html` from 2023) into a personal blog/site covering: homebrewing, 3D printing / maker projects, personal software projects, and ADHD / mental health / parenting.

## Structure

Single tagged feed, not separate top-level sections. One reverse-chronological home page listing all posts; each post carries one or more tags (`beer`, `maker`, `software`, `adhd-parenting`). A `/tags.html` page groups posts by tag for browsing by topic. This keeps authoring low-friction — write and tag, no need to decide which "section" a post belongs to.

## Authoring workflow

Posts are plain markdown files in `_posts/`, named `YYYY-MM-DD-title.md` with YAML front matter (`title`, `date`, `tags`). Publish = commit + push to `main`; GitHub Pages rebuilds and deploys automatically (no CI config needed — Jekyll is natively supported). No CMS, no extra service.

## Visual direction: "Field Notes"

A maker's lab-notebook feel:
- Warm paper background (`#f2f0e9`) with a faint graph-paper grid
- Ink-dark text (`#1b1d24`), safety-orange accent (`#e8541e`) used sparingly (left border on post cards, tag labels)
- Display/body text in a warm literary serif (Iowan Old Style / Palatino / Georgia stack); post metadata (date, tag) in a monospace stack (ui-monospace / SF Mono / Cascadia Code / Menlo / Consolas) to read like lab-log annotations
- Post entries styled as index cards: monospace date+tag line, serif title, monospace excerpt line

Approved via a 3-way mockup comparison (Field Notes vs. Brew Copper vs. Quiet Signal); Field Notes was the clear pick.

## Architecture

Jekyll, built directly by GitHub Pages from the `main` branch (using the `github-pages` gem so the local/remote build environment matches). A small custom theme is hand-written to match the Field Notes look rather than reskinning an existing Jekyll theme.

```
kielsucks.github.io/
  _config.yml
  Gemfile
  index.html          # home feed: all posts, tag chips
  tags.html           # posts grouped by tag
  about.md            # about page
  _layouts/
    default.html
    post.html
  _includes/
    header.html
    footer.html
  _posts/
    2026-07-31-welcome.md   # starter post so the feed isn't empty at launch
  assets/
    css/style.scss     # Field Notes styles (graph paper bg, monospace metadata, index-card posts)
  .gitignore
```

Tag pages use plain Liquid (`site.tags`) rather than the `jekyll-archives` plugin, since GitHub Pages' plugin whitelist can be unreliable — this avoids that dependency entirely.

## Data flow

Static site, no backend. Build-time only: Jekyll reads `_posts/` + front matter, renders `_layouts/`, GitHub Pages serves the output. No forms, no user data, no server-side state.

## Error handling

Not much applies to a static site. The main failure mode is a broken Jekyll build (bad YAML front matter, Liquid syntax errors). Mitigated by shipping one working starter post as a template to copy from, and verifying the build (see below) before considering the launch done.

## Testing / verification

1. `bundle exec jekyll build` (or `jekyll serve` for local preview) if Ruby/Bundler is in workable shape locally — nice-to-have, not required to publish.
2. Push to `main`, then check the GitHub Pages build status (`gh api` on the pages/deployments endpoint, or the repo's Actions tab) to confirm a clean deploy.
3. Curl/load the live `https://kielsucks.github.io` URL and confirm the home feed, a post page, and the tags page all render.

## Out of scope (for this pass)

- CMS / phone-based publishing (explicitly deferred — markdown + git is the chosen workflow)
- Comments, search, RSS styling beyond Jekyll's default feed
- Custom domain
- Separate top-level section pages (explicitly rejected in favor of tagged single feed)
