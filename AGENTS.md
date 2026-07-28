# AGENTS.md

## What this repo is

A personal academic website built with Jekyll (Minimal Mistakes theme), deployed to GitHub Pages at `ZhouLinli.github.io`. The site has two main content columns: **Research Skill Column** and **Essayistic Reflection Column**, each with numbered subtabs containing articles.

## Key commands

```bash
# Install dependencies (run once or after Gemfile changes)
bundle install

# Local preview with auto-rebuild
bundle exec jekyll serve          # http://localhost:4000
bundle exec jekyll liveserve      # same, with live-reload (uses hawkins gem)

# JS build (rarely needed — only if you change vendor JS)
npm run build:js
```

Restart `jekyll serve` (Ctrl+C then re-run) after editing `_config.yml`. All other file edits auto-rebuild.

## `_config.yml` changes require a server restart

`_config.yml` is **not** hot-reloaded. Every other file type (Markdown, HTML, SCSS, data files) auto-rebuilds.

## File naming conventions (ACTUAL — not what TABS-GUIDE.md describes)

The `TABS-GUIDE.md` shows a generic `index.md` convention. **The real files use different names:**

| Level | Convention | Example |
|---|---|---|
| Tab landing | `_pages/<tab>/<tab>Nmain_<name>.md` | `_pages/tab1_research/tab1main_research.md` |
| Subtab listing | `_pages/<tab>/<subtab>/subtab<N>.Nmain_<name>.md` | `_pages/tab1_research/1.1collect/subtab1.1main_collect.md` |
| Article | `_pages/<tab>/<subtab>/article-<N>.md` | `_pages/tab1_research/1.1collect/article-1.md` |

Folder names use abbreviated numbered prefixes: `1.1collect`, `1.2viz`, `1.3analytics`, `2.1econ`, `2.2ppdev`.

**Permalink convention** (must match `_data/navigation.yml` URLs):
- Tab: `/Research/`, `/Essay/`
- Subtab: `/Research/Data-Collection/`, `/Research/Data-Visualization/`, etc.
- Article: `/Research/Data-Collection/article-1/`

## Navigation: the two sources of truth

Adding/removing/renaming tabs and subtabs requires editing **both** files:

1. **`_data/navigation.yml`** — controls the nav bar. The `url:` value here must match the page's `permalink:`.
2. **The page's front matter** — `permalink:` must exactly match the `url:` in `navigation.yml`.

These must stay in sync or you get 404s.

## Content structure: three-file pattern for articles

To add an article to a subtab:

1. **Create** `_pages/<tab>/<subtab>/article-<N>.md` with `title`, `permalink`, `date`, `excerpt`
2. **Register** it in the subtab's listing page (`subtab<N>.Nmain_<name>.md`) — add a `### [Title](/url/)` + abstract block before the `<!-- ADD MORE ARTICLES HERE -->` comment

## Collections

Defined in `_config.yml` under `collections:`:
- `teaching`, `publications`, `portfolio`, `talks` — standard Minimal Mistakes collections
- `posts` (default) — `_posts/` directory, date-prefixed filenames

## Style/SCSS changes

SCSS lives in `_sass/`. The `sass.style` is `compressed` in production, `expanded` in dev (`_config.dev.yml`). Avatar shape is controlled in `_sass/_sidebar.scss` (`border-radius` on `.author__avatar img`).

## Common gotchas

- **`_config.yml` not reloaded**: must restart `jekyll serve` after changes.
- **Permalink mismatch**: if `navigation.yml` url and page permalink don't match, the link 404s.
- **TABS-GUIDE.md is outdated**: it describes `index.md` naming that doesn't match the actual files. Follow the real naming conventions above.
- **`Gemfile.lock` security warning**: if GitHub flags a vulnerability, delete `Gemfile.lock` and re-run `bundle install`.
- **`future: true`** in `_config.yml` means posts with future dates (like `2199-01-01-future-post.md`) render on the site.

## References

- `TABS-GUIDE.md` — detailed guide for adding tabs/subtabs/articles (useful for workflow, but file naming examples are generic placeholders)
- `_data/navigation.yml` — master nav bar definition (has inline docs)
- `_config.yml` — all site settings (has inline docs)
