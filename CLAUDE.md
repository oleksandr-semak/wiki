# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Jekyll-based personal wiki/blog for DevOps content: AWS certifications, homelab experiments, and DevOps cases. Uses the Minima theme with dark skin and custom sidebar navigation.

## Build Commands

```bash
# Requires Ruby 3.0+ (Homebrew Ruby recommended)
# Add to PATH if needed: export PATH="/opt/homebrew/opt/ruby/bin:$PATH"

bundle install              # Install dependencies
bundle exec jekyll serve    # Local dev server at localhost:4000
bundle exec jekyll build    # Production build to _site/
```

## Architecture

- **Theme:** Minima with dark skin
- **Content:** `_wiki/` collection with nested structure (aws/exam/...)
- **Navigation:** Data-driven collapsible sidebar via `_data/navigation.yml`
- **Layouts:** `wiki.html` (base with sidebar), `home.html` (extends wiki)
- **Styles:** Custom sidebar styles in `_sass/minima/custom-styles.scss`

## Adding Content

1. Add navigation entry to `_data/navigation.yml`
2. Create page in `_wiki/` with `layout: wiki` and matching `permalink`
