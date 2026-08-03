# kielsucks.github.io

Personal blog — beer, maker/3D printing projects, personal software builds, and ADHD/parenting. Jekyll, built and deployed automatically by GitHub Pages.

## Add a post

Create a new file in `_posts/` named `YYYY-MM-DD-title.md`:

```
---
layout: post
title: "Your title here"
tags: [beer]
---

Your post content, in markdown.
```

Then:

```bash
git add _posts/YYYY-MM-DD-title.md
git commit -m "Add post: your title"
git push
```

GitHub Pages rebuilds and deploys automatically — live in under a minute.

- **Date** comes from the filename, not a front-matter field — don't add a `date:` key.
- **Tags** are fully dynamic. Use an existing tag (`beer`, `maker`, `software`, `adhd-parenting`) to add the post to that section on `/tags.html`, or make up a brand-new tag — a new section appears automatically on the next build. Nothing else to register.
- A post can have more than one tag: `tags: [beer, maker]`.

## Preview locally (optional)

Not required to publish — GitHub does the real build — but useful to check before pushing:

```bash
export PATH="/opt/homebrew/opt/ruby@3.2/bin:$PATH"
bundle exec jekyll serve
```

Then open `http://localhost:4000`. Must use `ruby@3.2` specifically — the system Ruby (2.6) is too old and plain Homebrew `ruby` (4.0+) is too new for the `github-pages` gem's pinned dependencies.

## Structure

- `_posts/` — all posts
- `_layouts/`, `_includes/`, `assets/css/style.scss` — the "Field Notes" theme
- `index.html` — home feed (all posts, reverse-chron)
- `tags.html` — posts grouped by tag
- `about.md` — about page
- `docs/superpowers/` — build history (design spec + implementation plan), not part of the published site
