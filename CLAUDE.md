# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Jekyll static site blog ("abotk.in") hosted on GitHub Pages, built and deployed by GitHub Actions (`.github/workflows/pages.yml`). Personal blog focused on iOS/macOS development topics.

## Build & Development Commands

```bash
bundle install              # Install dependencies
bundle exec jekyll serve    # Local dev server at http://localhost:4000
bundle exec jekyll build    # Build static site to _site/
```

## Architecture

Standard Jekyll 4 site using the `minima` theme. It does not use the `github-pages` gem, so dependencies are whatever `Gemfile.lock` pins:

- **`_config.yml`** — Site config: kramdown markdown, `/:title/` permalinks, minima theme, compressed Sass output
- **`.github/workflows/pages.yml`** — Builds on PRs; builds and deploys to Pages on push to `main` (actions pinned by SHA)
- **`.github/dependabot.yml`** — Weekly bundler and GitHub Actions updates
- **`_posts/`** — Blog posts as `YYYY-MM-DD-slug.markdown` with YAML front matter
- **`_includes/head.html`** — Custom HEAD partial (overrides minima's default for favicon support)
- **`index.md`**, **`about.md`**, **`tags.html`**, **`404.html`** — Top-level pages
- **`_site/`** — Generated output (git-ignored)
- **`CNAME`** — Custom domain (`abotk.in`)

## Writing Posts

Posts go in `_posts/` with filename format `YYYY-MM-DD-title-slug.markdown`. Front matter:

```yaml
---
layout: post
title: "Post Title"
date: YYYY-MM-DD HH:MM:SS -0500
categories: [Category1, Category2]
---
```

Categories appear on the `/tags/` page. Syntax highlighting uses Jekyll's `{% highlight lang %}` blocks.

## Plugins

- `jekyll-feed` — RSS/Atom feed generation
- `jekyll-seo-tag` — SEO meta tags
