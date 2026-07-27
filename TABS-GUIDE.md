# Navigation Tabs, Subtabs & Articles — Setup Guide

This document explains how the navigation tabs and hover dropdowns work, which files to edit for each change, and how to rename, add, remove, or hide tabs, subtabs, and articles. It also covers how to preview the site locally before pushing to GitHub.

---

## Current Structure at a Glance

```
Nav bar
├── Tab One                         _pages/tab-one.md
│   ├── Tab One — Subtab A          _pages/tab-one-subtab-a.md
│   │   ├── Article 1A-1            _pages/tab-one-subtab-a-article-1.md
│   │   ├── Article 1A-2            _pages/tab-one-subtab-a-article-2.md
│   │   └── Article 1A-3            _pages/tab-one-subtab-a-article-3.md
│   └── Tab One — Subtab B          _pages/tab-one-subtab-b.md
│       ├── Article 1B-1            _pages/tab-one-subtab-b-article-1.md
│       ├── Article 1B-2            _pages/tab-one-subtab-b-article-2.md
│       └── Article 1B-3            _pages/tab-one-subtab-b-article-3.md
│
├── Tab Two                         _pages/tab-two.md
│   ├── Tab Two — Subtab A          _pages/tab-two-subtab-a.md
│   │   ├── Article 2A-1            _pages/tab-two-subtab-a-article-1.md
│   │   ├── Article 2A-2            _pages/tab-two-subtab-a-article-2.md
│   │   └── Article 2A-3            _pages/tab-two-subtab-a-article-3.md
│   ├── Tab Two — Subtab B          _pages/tab-two-subtab-b.md
│   │   ├── Article 2B-1            _pages/tab-two-subtab-b-article-1.md
│   │   ├── Article 2B-2            _pages/tab-two-subtab-b-article-2.md
│   │   └── Article 2B-3            _pages/tab-two-subtab-b-article-3.md
│   └── Tab Two — Subtab C          _pages/tab-two-subtab-c.md
│       ├── Article 2C-1            _pages/tab-two-subtab-c-article-1.md
│       ├── Article 2C-2            _pages/tab-two-subtab-c-article-2.md
│       └── Article 2C-3            _pages/tab-two-subtab-c-article-3.md
│
├── Tab Three  (no subtabs)         _pages/tab-three.md
│   ├── Article 3-1                 _pages/tab-three-article-1.md
│   ├── Article 3-2                 _pages/tab-three-article-2.md
│   └── Article 3-3                 _pages/tab-three-article-3.md
│
└── Tab Four                        _pages/tab-four.md
    ├── Tab Four — Subtab A         _pages/tab-four-subtab-a.md
    │   ├── Article 4A-1            _pages/tab-four-subtab-a-article-1.md
    │   ├── Article 4A-2            _pages/tab-four-subtab-a-article-2.md
    │   └── Article 4A-3            _pages/tab-four-subtab-a-article-3.md
    └── Tab Four — Subtab B         _pages/tab-four-subtab-b.md
        ├── Article 4B-1            _pages/tab-four-subtab-b-article-1.md
        ├── Article 4B-2            _pages/tab-four-subtab-b-article-2.md
        └── Article 4B-3            _pages/tab-four-subtab-b-article-3.md
```

---

## Files Overview

| File                                   | Purpose                                                                                                                                                                     |
| -------------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `_data/navigation.yml`               | **Master list** of all tabs and subtabs. Controls what appears in the nav bar. Articles are NOT listed here — they are linked only from their subtab's listing page. |
| `_includes/masthead.html`            | Renders the nav bar HTML. Reads from`navigation.yml`. Rarely needs editing.                                                                                               |
| `_sass/_masthead.scss`               | CSS styles for the nav bar and hover dropdown. Rarely needs editing.                                                                                                        |
| `_pages/tab-*.md`                    | Tab landing pages — show subtab cards (linked name + description).                                                                                                         |
| `_pages/tab-*-subtab-*.md`           | Subtab listing pages — show article cards (linked title + abstract).                                                                                                       |
| `_pages/tab-*-subtab-*-article-*.md` | Individual article pages — full article content.                                                                                                                           |

---

## How to Add a New Article

This is the most common task. Follow these three steps every time.

### Step 1 — Create the article file

Copy any existing article file for the same subtab, e.g.:

```
_pages/tab-one-subtab-a-article-3.md  →  copy to  →  _pages/tab-one-subtab-a-article-4.md
```

Naming convention: `<tab>-<subtab>-article-<number>.md`

- Use the same tab and subtab prefix as the other articles in that section.
- Increment the number.

Update these four fields in the front matter at the top of the new file:

```yaml
title: "Your Real Article Title"           # ← full title shown on the article page
permalink: /tab-one/subtab-a/article-4/   # ← unique URL; follow the pattern of existing articles
date: 2024-06-01                           # ← publication date YYYY-MM-DD
excerpt: "One sentence summary."           # ← shown in search results / meta tags
```

Then replace all placeholder body text below the front matter with your real article content.

### Step 2 — Add the article to its subtab listing page

Open the subtab's listing page, e.g. `_pages/tab-one-subtab-a.md`.

Find the `<!-- ADD MORE ARTICLES HERE -->` comment at the bottom and paste a new article block just above it:

```markdown
### [Your Real Article Title](/tab-one/subtab-a/article-4/)

Your 2–3 sentence abstract. Tell the reader what this article covers and why it matters.

---
```

- The `###` heading becomes the clickable title.
- The URL in `(...)` must exactly match the `permalink` you set in Step 1.
- The paragraph below is the abstract shown on the listing page.

### Step 3 — Preview locally to verify (see Local Preview section below)

Run the local server and visit the subtab page to confirm the new article card appears and the link works.

---

## How to Edit an Existing Article

Open the article's `_pages/` file directly and edit the body content. To rename the article title:

- Update `title:` in the front matter.
- Update the `[Title]` text in the subtab listing page where the article is linked.
- The `permalink:` does not need to change (URLs are independent of display titles).

---

## How to Remove an Article

1. Delete (or comment out) the article's block in the subtab listing page (`### [Title](url/)` + abstract + `---`).
2. Optionally delete the article's `_pages/` file. If you leave the file, the page still exists at its URL but is just no longer linked from the listing.

---

## How to Rename a Tab or Subtab

You need to change the name in **two places**:

**1. `_data/navigation.yml`** — update `title:` for the tab or subtab entry.

**2. The page file** — update `title:` in the front matter of the corresponding `_pages/` file.

The `url` / `permalink` values do **not** need to change — they are internal routing slugs and are independent of display names.

If you also want to update how the subtab name appears as a card on its parent tab page, open the parent tab's `_pages/tab-*.md` and update the `[Subtab Name]` text in the `## [Subtab Name](/url/)` heading.

---

## How to Hide the Dropdown (Remove Subtabs from a Tab)

Delete or comment out the `children:` block for that tab in `_data/navigation.yml`:

```yaml
# Dropdown removed — tab goes directly to its page on click
- title: "Tab One"
  url: /tab-one/
  # children:
  #   - title: "Tab One — Subtab A"
  #     url: /tab-one/subtab-a/
```

The subtab page files stay in `_pages/` and remain accessible by URL — they just won't appear in the nav dropdown.

---

## How to Add a New Subtab to an Existing Tab

**1.** Add the subtab entry under `children:` in `_data/navigation.yml`:

```yaml
children:
  - title: "Tab One — Subtab C"    # ← new subtab label
    url: /tab-one/subtab-c/        # ← new URL
```

**2.** Create the subtab listing page `_pages/tab-one-subtab-c.md` — copy an existing subtab file and update `title:`, `permalink:`, and the article list.

**3.** Add a subtab card to the parent tab's landing page `_pages/tab-one.md`:

```markdown
## [Tab One — Subtab C](/tab-one/subtab-c/)

A brief description of what this subtab covers.

---
```

**4.** Create article files for the new subtab following the "How to Add a New Article" steps above.

---

## How to Add a Completely New Tab

**1.** Add the tab block to `_data/navigation.yml`:

```yaml
- title: "New Tab"
  url: /new-tab/
  children:                        # remove this block if no subtabs needed
    - title: "New Tab — Subtab A"
      url: /new-tab/subtab-a/
```

**2.** Create the tab landing page `_pages/new-tab.md` — copy an existing tab file and update `title:`, `permalink:`, and the subtab cards.

**3.** Create subtab listing pages and article files following the steps above.

---

## How to Remove a Tab Entirely

1. Delete or comment out the tab block in `_data/navigation.yml`.
2. Optionally delete the tab's `_pages/` files (tab page, subtab pages, article pages).

---

## File Naming Convention

```
_pages/
  tab-one.md                          ← Tab One landing page
  tab-one-subtab-a.md                 ← Tab One › Subtab A listing
  tab-one-subtab-a-article-1.md       ← Tab One › Subtab A › Article 1
  tab-one-subtab-a-article-2.md       ← Tab One › Subtab A › Article 2
  tab-one-subtab-b.md                 ← Tab One › Subtab B listing
  tab-one-subtab-b-article-1.md       ← Tab One › Subtab B › Article 1
  ...
  tab-three.md                        ← Tab Three (no subtabs) — articles listed directly
  tab-three-article-1.md              ← Tab Three › Article 1
  ...
```

Permalink convention to match:

```
/tab-one/                             ← tab landing
/tab-one/subtab-a/                    ← subtab listing
/tab-one/subtab-a/article-1/          ← article page
/tab-three/article-1/                 ← article under a tab with no subtabs
```

---

## Quick-Reference Checklist

**Add a new article to a subtab:**

- [ ] Create `_pages/<tab>-<subtab>-article-<N>.md` — set `title`, `permalink`, `date`, `excerpt`, write body
- [ ] Open `_pages/<tab>-<subtab>.md` — add `### [Title](/url/)` + abstract block before the ADD MORE comment

**Rename a tab:**

- [ ] `_data/navigation.yml` → update `title:`
- [ ] `_pages/<tab>.md` → update `title:` in front matter

**Rename a subtab:**

- [ ] `_data/navigation.yml` → update `title:` under `children:`
- [ ] `_pages/<tab>-<subtab>.md` → update `title:` in front matter
- [ ] `_pages/<tab>.md` → update the `[Subtab Name]` text in the subtab card heading

**Add a subtab:**

- [ ] `_data/navigation.yml` → add entry under `children:`
- [ ] Create `_pages/<tab>-<subtab>.md`
- [ ] `_pages/<tab>.md` → add subtab card block
- [ ] Create article files for the new subtab

**Remove subtabs from a tab (no dropdown):**

- [ ] `_data/navigation.yml` → comment out or delete the `children:` block

**Add a new tab:**

- [ ] `_data/navigation.yml` → add tab block
- [ ] Create `_pages/<tab>.md`
- [ ] Create subtab and article files as needed

---

## How to Preview the Site Locally

Preview lets you see all changes in your browser **before** pushing to GitHub. The live site at `ZhouLinli.github.io` only updates after a `git push` — the local preview is your safe sandbox to test everything first.

### Prerequisites (one-time setup)

You need Ruby and Bundler installed. Check by running:

```bash
ruby --version
bundle --version
```

If either command is not found, install them:

- **Ruby**: install via [rbenv](https://github.com/rbenv/rbenv) or the [official installer](https://www.ruby-lang.org/en/documentation/installation/)
- **Bundler**: once Ruby is installed, run `gem install bundler`

Then install the site's dependencies (run once, or again after changing `Gemfile`):

```bash
cd /Users/linlizhou/Documents/Columnist/ZhouLinli.github.io
bundle install
```

### Start the local preview server

```bash
cd /Users/linlizhou/Documents/Columnist/ZhouLinli.github.io
bundle exec jekyll serve
```

Open your browser and go to: **http://localhost:4000**

The server watches for file changes and rebuilds automatically — you only need to refresh your browser after saving a file. You do **not** need to restart the server for most edits (exception: changes to `_config.yml` require a restart).

### Stop the server

Press `Ctrl + C` in the terminal where the server is running.

### Live reload (optional)

Your `Gemfile` already includes the `hawkins` gem, which adds live-reload support. Use this command instead for automatic browser refresh on save:

```bash
bundle exec jekyll liveserve
```

### Troubleshooting common issues

| Problem                            | Fix                                                                                                                        |
| ---------------------------------- | -------------------------------------------------------------------------------------------------------------------------- |
| `bundle: command not found`      | Run`gem install bundler` then retry                                                                                      |
| `Could not find gem '...'`       | Run`bundle install` to install missing gems                                                                              |
| Port 4000 already in use           | Run`bundle exec jekyll serve --port 4001` and visit http://localhost:4001                                                |
| Changes not appearing              | Hard-refresh the browser (`Cmd + Shift + R` on Mac); if still missing, check the terminal for build errors               |
| `_config.yml` change not applied | Stop the server (`Ctrl + C`) and restart it                                                                              |
| A new page returns 404             | Check that the`permalink:` in the page's front matter exactly matches the URL you are visiting, including trailing slash |

### Workflow: edit → preview → push

```
1. Make changes to any _pages/ file or _data/navigation.yml
2. Save the file
3. Refresh browser at http://localhost:4000 — verify the change looks correct
4. Repeat steps 1–3 until satisfied
5. git add, git commit, git push → live site updates within ~1 minute
```

---

*Last updated: structure has 4 tabs, 7 subtabs, 24 article pages (21 under subtabs + 3 directly under Tab Three).*
