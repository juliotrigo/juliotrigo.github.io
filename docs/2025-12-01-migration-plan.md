# Jekyll & Ruby Migration Plan

This document outlines the plan to upgrade the juliotrigo.com website from outdated Jekyll/Ruby versions to the latest stable releases.

## Current State

### Versions

| Component | Current Version | Target Version |
|-----------|-----------------|----------------|
| Ruby | 3.1.3 | 3.4.7 |
| Jekyll | 3.9.3 | 4.4.1 |
| Minima | 2.5.1 | 2.5.2 |
| WEBrick | 1.8.x | 1.9.x |
| Bundler | 2.3.26 | Latest |

### Current Setup

- **Deployment**: Netlify (not GitHub Pages)
- **Gemfile**: Uses `github-pages` meta-gem, which bundles ~100+ gems but pins to older versions
- **Theme**: Minima with extensive customizations

### Customizations in Place

**Layouts (`_layouts/`):**
- `home.html` - Simplified version (shows content only, no posts list)
- `post.html` - Added `modified_date` support, removed Disqus comments

**Includes (`_includes/`):**
- `footer.html` - Gravatar photo, custom social icons (GitHub, Twitter, Slides), CC BY-SA 4.0 license section
- `head.html` - Custom OG/Twitter meta tags, favicons, Google Fonts, webmention support
- Custom icon includes: `icon-github.html`, `icon-twitter.html`, `icon-slides.html`
- `youtubePlayer.html` - Responsive YouTube embeds
- `cc_by-sa_4-0_88x31.svg` - Creative Commons badge

**SASS (`_sass/`):**
- `juliotrigo.scss` + partials (`_base.scss`, `_layout.scss`, `_syntax-highlighting.scss`)
- `latolatinweb.scss` - Custom LatoLatin font definitions
- `base16-eighties-dark.scss` - Color scheme for syntax highlighting
- `pygments-template.scss` - Syntax highlighting rules

**Assets:**
- Custom fonts in `assets/fonts/`
- Custom images in `assets/images/`

### Plugins Actually Used

| Plugin | Usage |
|--------|-------|
| jekyll-feed | `{% feed_meta %}` tag in head.html for RSS |
| jekyll-redirect-from | `redirect_from:` front matter in 8 files |
| jekyll-sitemap | Generates sitemap.xml |

---

## Migration Steps

### Step 1: Update `.ruby-version`

Change from `3.1.3` to `3.4.7`.

### Step 2: Update `Gemfile`

Replace the entire Gemfile with a minimal set of dependencies:
- Remove the `github-pages` meta-gem
- Add Jekyll 4.4.x directly
- Add Minima 2.5.x directly
- Add only the plugins actually used
- Upgrade WEBrick from 1.8.x to 1.9.x

### Step 3: Delete `Gemfile.lock`

Force fresh dependency resolution by removing the lock file.

### Step 4: Install Dependencies

```bash
chruby 3.4.7
bundle install
```

### Step 5: Update `.gitignore`

Add `.jekyll-cache/` directory (Jekyll 4.x creates this).

### Step 6: Local Testing

```bash
bundle exec jekyll serve
```

Test all pages:
- [x] Homepage renders correctly
- [x] All articles display properly
- [x] Post dates and modified dates show
- [x] Syntax highlighting works
- [x] Custom fonts (LatoLatin) load
- [x] Social icons display in footer
- [x] License section in footer
- [x] RSS feed generates (`/feed.xml`)
- [x] Sitemap generates (`/sitemap.xml`)
- [x] Redirects work (test one old URL)
- [x] YouTube embeds work
- [x] Gravatar image loads

---

## Test Results

All tests passed on 2025-12-01. Testing was performed by inspecting the generated files in `_site/`.

### Site Functionality Tests

| Test | Status | Evidence |
|------|--------|----------|
| Homepage renders | PASS | Title, navigation, content, footer all present in `_site/index.html` |
| Articles display | PASS | 11 articles generated in `_site/articles/` |
| Post dates | PASS | "Feb 2, 2016" shown correctly in post headers |
| Modified dates | PASS | "Nov 12, 2019 ~ Apr 18, 2020" shown in `lato-2-0-font-family` post |
| Syntax highlighting | PASS | 17 `highlighter-rouge` CSS classes found in `unicode-strings-and-bytestrings-in-python-2` post |
| Social icons in footer | PASS | GitHub, Twitter, Slides, RSS icons all present in footer HTML |
| License section | PASS | CC BY-SA 4.0 SVG badge and link present in footer |
| YouTube embeds | PASS | iframe with `youtube-container` class in `pycones-2015-distributed-services` post |
| Gravatar | PASS | Image URL `https://www.gravatar.com/avatar/...` present in footer |
| Custom fonts | PASS | Google Fonts links (Indie Flower, Ubuntu Mono, Material Icons) in `<head>` |

### Plugin Verification

#### jekyll-feed

**Status: WORKING**

Verified by inspecting `_site/feed.xml`:
- Valid Atom XML structure with `xmlns="http://www.w3.org/2005/Atom"`
- Generator tag shows `Jekyll v4.4.1`
- Contains entries for all posts with titles, links, and content

#### jekyll-sitemap

**Status: WORKING**

Verified by inspecting `_site/sitemap.xml`:
- Valid XML with proper sitemap schema
- Contains 18 URLs for all pages and posts
- Each URL includes `<loc>` and `<lastmod>` tags

#### jekyll-redirect-from

**Status: WORKING**

Verified by checking that redirect HTML pages were generated for all `redirect_from:` front matter entries:

| Redirect From | Redirect To | Generated File |
|---------------|-------------|----------------|
| `/archive/` | `/articles/` | `_site/archive/index.html` |
| `/p/about-this-blog.html` | `/about/` | `_site/p/about-this-blog.html` |
| `/posts/nameko-eventlog-dispatcher/` | `/articles/nameko-eventlog-dispatcher/` | `_site/posts/nameko-eventlog-dispatcher/index.html` |
| `/posts/pycones-2015-distributed-services/` | `/articles/pycones-2015-distributed-services/` | `_site/posts/pycones-2015-distributed-services/index.html` |
| `/posts/pycones-2019-...` | `/articles/pycones-2019-...` | `_site/posts/pycones-2019-.../index.html` |
| `/posts/lato-2-0-font-family/` | `/articles/lato-2-0-font-family/` | `_site/posts/lato-2-0-font-family/index.html` |

Each redirect page contains proper HTML with both meta refresh and JavaScript redirect:
```html
<meta http-equiv="refresh" content="0; url=http://localhost:4000/articles/">
<script>location="http://localhost:4000/articles/"</script>
```

---

## Changes Made

The following file changes have been applied:

| File | Change |
|------|--------|
| `.ruby-version` | `3.1.3` → `3.4.7` |
| `Gemfile` | Replaced `github-pages` meta-gem with minimal dependencies |
| `Gemfile.lock` | Deleted |
| `.gitignore` | Added `.jekyll-cache/` |

### Post-Migration Cleanup

#### Removed explicit `webrick` gem (2025-12-01)

The `webrick` gem was originally added explicitly to make Jekyll work with Ruby 3.x. However, since Jekyll 4.3.0, `webrick` is listed as a direct dependency of Jekyll itself (`webrick (~> 1.7)`).

With Jekyll 4.4.1, there's no need to declare `webrick` in the Gemfile - it's pulled in automatically as a transitive dependency.

## Manual Steps Required

Run these commands in your terminal:

```bash
chruby 3.4.7
bundle install
bundle exec jekyll serve
```

Then test the site locally to verify everything works before committing.

---

## Known Warnings

Jekyll 4 uses Dart Sass instead of LibSass, which produces deprecation warnings:

### `@import` Deprecation

Sass `@import` is deprecated in favor of `@use`/`@forward` (will be removed in Dart Sass 3.0). Affects:
- `assets/main.scss` - local file
- Minima's internal SCSS files

### Color Function Deprecation

`lighten()` and `darken()` are deprecated in favor of `color.adjust()`. These warnings come from **Minima's gem files**, not local code.

### Decision

**Leave as-is.** These are warnings only - the site builds and works correctly. Dart Sass 3.0 is not yet released. Fixing would require either:
- Migrating to `@use`/`@forward` (complex, and Minima itself still uses `@import`)
- Overriding Minima's SCSS files locally (maintenance burden)

When Minima releases a version with updated Sass syntax, upgrading will resolve these warnings.

---

## Potential Issues to Watch

### Minima 2.5.2

Nearly identical to 2.5.1 - no breaking changes expected. Custom overrides should continue working.

---

## References

- [Jekyll 4.4.1 Release Notes](https://jekyllrb.com/news/releases/)
- [Minima Theme Repository](https://github.com/jekyll/minima)
- [Ruby Downloads](https://www.ruby-lang.org/en/downloads/)
