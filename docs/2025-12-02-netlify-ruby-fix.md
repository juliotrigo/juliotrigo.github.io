# Netlify Ruby Version Fix

This document describes the fix for the Netlify deployment timeout caused by Ruby 3.4.7.

## Problem

Netlify deployment times out during the Ruby installation step:

```
8:16:02 AM: mise ruby@3.4.7      ==> Installing ruby-3.4.7...
8:16:03 AM: mise ruby@3.4.7      -> ./configure "--prefix=..." --enable-shared
...
8:34:01 AM: Execution timed out after 17m59.74787331s
```

**Root cause**: Netlify's Noble build image (Ubuntu 24.04) doesn't have a pre-built binary for Ruby 3.4.7. When this happens, `mise` falls back to compiling Ruby from source, which exceeds Netlify's 18-minute timeout.

## Solution

Downgrade to Ruby 3.3.6, which is pre-installed on Netlify Noble.

### Gem Compatibility Verification

| Gem | Compatibility with Ruby 3.3.6 |
|-----|-------------------------------|
| Jekyll 4.4.1 | Yes (requires Ruby >= 2.7.0) |
| sass-embedded 1.94.2 | Yes (has pre-built binaries) |
| google-protobuf 4.33.1 | Yes (has pre-built binaries) |
| All other gems | Yes |

## Steps

### Step 1: Update `.ruby-version`

Change from `3.4.7` to `3.3.6`.

### Step 2: Delete `Gemfile.lock`

Force fresh dependency resolution for the new Ruby version.

### Step 3: Install Ruby 3.3.6 (Apple Silicon Macs)

On Apple Silicon Macs with newer Clang versions (17.x), compiling Ruby 3.3.x fails with:

```
./vm_callinfo.h:179:9: error: use of undeclared identifier 'RUBY_FUNCTION_NAME_STRING'
```

**Root cause**: Ruby's autoconf script fails to detect that the compiler supports `__func__`.

**Fix**: Pass the variable explicitly to ruby-install:

```bash
ruby-install 3.3.6 -- rb_cv_function_name_string=__func__
```

### Step 4: Install Dependencies

```bash
chruby 3.3.6
bundle install
```

### Step 5: Local Testing

```bash
bundle exec jekyll serve
```

---

## Test Results

All tests passed on 2025-12-02. Testing was performed by inspecting the generated files in `_site/`.

### Site Functionality Tests

| Test | Status | Evidence |
|------|--------|----------|
| Homepage renders | PASS | Title, navigation, content, footer all present in `_site/index.html` |
| Articles display | PASS | 11 articles generated in `_site/articles/` |
| Post dates | PASS | Dates showing correctly in posts |
| Modified dates | PASS | "Nov 12, 2019 ~ Apr 18, 2020" shown in `lato-2-0-font-family` post |
| Syntax highlighting | PASS | 17 `highlighter-rouge` CSS classes found in `unicode-strings-and-bytestrings-in-python-2` post |
| Social icons in footer | PASS | GitHub, Twitter, Slides, RSS icons all present in footer HTML |
| License section | PASS | CC BY-SA 4.0 SVG badge and link present in footer |
| YouTube embeds | PASS | iframe with `youtube-container` class in `pycones-2015-distributed-services` post |
| Gravatar | PASS | Image URL `https://www.gravatar.com/avatar/...` present in footer |
| Custom fonts | PASS | Google Fonts links (Indie Flower, Ubuntu Mono, Material Icons, IBM Plex Mono) in `<head>` |

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

Each redirect page contains proper HTML with both meta refresh and JavaScript redirect:
```html
<meta http-equiv="refresh" content="0; url=http://localhost:4000/articles/">
<script>location="http://localhost:4000/articles/"</script>
```

---

## References

- [Netlify Noble Build Image Announcement](https://answers.netlify.com/t/new-ubuntu-24-04-noble-numbat-build-image/130497)
- [Netlify Available Software Docs](https://docs.netlify.com/build/configure-builds/available-software-at-build-time/)
- [Jekyll Releases](https://github.com/jekyll/jekyll/releases)
- [Error of RUBY_FUNCTION_NAME_STRING when compiling Ruby](https://dev.to/franklinyu/error-of-rubyfunctionnamestring-when-compiling-ruby-32b8)
