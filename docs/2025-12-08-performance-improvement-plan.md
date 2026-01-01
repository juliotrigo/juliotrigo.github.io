# Performance Improvement Plan

This plan addresses the performance issues identified by PageSpeed Insights for juliotrigo.com (mobile score: 88).

## Progress

| Step | Description | Status |
|------|-------------|--------|
| 1 | Fix render-blocking Google Fonts | ✅ Completed (2025-12-08) |
| 2 | Optimize Google Analytics loading | ✅ Completed (2025-12-08) |
| 3 | Add explicit dimensions to images | ⏳ Pending |
| 4 | Fix `<html lang>` attribute | ✅ Completed (2025-12-08) |
| 5 | Enable CSS minification | ✅ Completed (2025-12-08) |
| 6 | Optimize images (optional) | ⏳ Pending |
| 7 | Preload Lato fonts to reduce critical chain | ✅ Completed (2025-12-10) |

---

## Summary of Issues

| Priority | Issue | Impact | Savings |
|----------|-------|--------|---------|
| High | Render-blocking Google Fonts | 540ms | Significant |
| High | Unused JavaScript (gtag.js) | 54 KiB | Medium |
| Medium | Unsized images (Gravatar, Sohonet logo) | CLS | Layout stability |
| Medium | Image delivery (no optimization) | 7 KiB | Minor |
| Low | Inefficient cache policy | 19 KiB | Minor |
| Low | `<html lang>` invalid value | Accessibility | None |
| Low | CSS not minified | ~8 KiB | Minor |

---

## Implementation Steps

### 1. Fix Render-Blocking Google Fonts

**File:** `_includes/head.html`

**Problem:** Three Google Font stylesheets load synchronously in `<head>`, blocking rendering for ~540ms:
- Indie Flower (title font)
- IBM Plex Mono (code font)
- Material Icons (icon font)

**Solution:** Use `preconnect` hints and `preload` with async pattern.

**How this works:**

1. **`<link rel="preconnect">`** - Establishes early connections to Google's font servers. This performs DNS lookup, TCP handshake, and TLS negotiation *before* the browser discovers it needs fonts. Saves ~100-300ms per origin.

2. **`<link rel="preload" as="style">`** - Tells the browser to start downloading the font CSS file immediately with high priority, but without blocking rendering. The `as="style"` attribute ensures correct prioritization and caching.

3. **`onload="this.onload=null;this.rel='stylesheet'"`** - This JavaScript snippet runs when the preload completes. It:
   - Sets `onload=null` to prevent the handler running twice
   - Changes `rel` from `preload` to `stylesheet`, which activates the CSS

4. **`<noscript>` fallback** - For users with JavaScript disabled, loads fonts the traditional way.

5. **`display=swap`** (already in URLs) - Tells the browser to show fallback text immediately, then swap to the custom font when loaded. This prevents invisible text (FOIT - Flash of Invisible Text) at the cost of a brief flash when fonts swap in (FOUT - Flash of Unstyled Text).

**Changes:**

```html
<!-- Before -->
{% if site.google_font %}
  {% for link in site.google_font %}
    <link rel="stylesheet" href="{{ link.url }}" type="text/css" />
  {% endfor %}
{% endif %}

<!-- After -->
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
{% if site.google_font %}
  {% for link in site.google_font %}
    <link rel="preload" href="{{ link.url }}" as="style" onload="this.onload=null;this.rel='stylesheet'">
    <noscript><link rel="stylesheet" href="{{ link.url }}" type="text/css"></noscript>
  {% endfor %}
{% endif %}
```

**Testing (2025-12-08):**

1. Built site with `bundle exec jekyll build` - ✅ Success
2. Verified generated HTML in `_site/index.html` contains:
   - ✅ Preconnect links for `fonts.googleapis.com` and `fonts.gstatic.com`
   - ✅ Preload links for all 3 fonts (Indie Flower, IBM Plex Mono, Material Icons)
   - ✅ Noscript fallbacks with `type="text/css"`
3. Started local server with `bundle exec jekyll serve`
4. Verified via curl that served page includes correct font links - ✅ All present
5. Verified Google Fonts URL returns HTTP 200 - ✅ Valid
6. Page renders correctly with title "Hi there 👋 | Julio Trigo" - ✅ Working

---

### 2. Optimize Google Analytics Loading

**Files:**
- Create `_includes/google-analytics.html` (override Minima's default)
- Update `_config.yml` (change analytics ID)

**Problem:**
- Google Analytics (gtag.js) contributes 54 KiB of unused JavaScript on initial load
- Minima's default template uses the deprecated Universal Analytics (`analytics.js`)
- The config used an old Universal Analytics ID (`UA-104164449-1`) which was sunset by Google on July 1, 2023

**Solution:**
- Migrate to Google Analytics 4 (GA4) using the modern `gtag.js` library
- Use the `async` attribute for non-blocking script loading
- Preserve Do Not Track (DNT) privacy check from Minima

**Code source:** The gtag.js snippet is from [Google's official documentation](https://developers.google.com/tag-platform/gtagjs), adapted with Jekyll templating and Minima's privacy wrapper.

**Changes to `_includes/google-analytics.html`:**

```html
<!-- Google Analytics 4 (gtag.js) -->
<script async src="https://www.googletagmanager.com/gtag/js?id={{ site.google_analytics }}"></script>
<script>
if (!(window.doNotTrack === "1" || navigator.doNotTrack === "1" || navigator.doNotTrack === "yes" || navigator.msDoNotTrack === "1")) {
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());
  gtag('config', '{{ site.google_analytics }}');
}
</script>
```

**Changes to `_config.yml`:**

```yaml
# Before
google_analytics: UA-104164449-1

# After
google_analytics: G-EPMRDFVSSX
```

**Do Not Track (DNT) privacy feature:**

The code checks if the user has enabled "Do Not Track" in their browser settings. If DNT is enabled, the analytics code doesn't run, respecting the user's privacy preference. This was present in Minima's original template and is preserved here as good privacy practice.

**Testing (2025-12-08):**

1. Verified `_includes/google-analytics.html` created with GA4 code - ✅ Correct
2. Verified `_config.yml` updated with GA4 ID (`G-EPMRDFVSSX`) - ✅ Correct
3. Built with `JEKYLL_ENV=production` and checked `_site/index.html` - ✅ Contains:
   - `async` attribute on script tag
   - Correct GA4 measurement ID
   - Do Not Track privacy check wrapper

**Note:** Analytics script only appears in production builds (controlled by `{% if jekyll.environment == 'production' %}` in `head.html`).

---

### 3. Add Explicit Dimensions to Images

**Problem:** Images without `width` and `height` attributes cause layout shifts (CLS) as they load.

**Affected images:**
1. Gravatar photo in footer (`_includes/footer.html`)
2. Sohonet logo in homepage (`index.md`)

#### 3a. Footer Gravatar Image

**File:** `_includes/footer.html`

**Changes:**

```html
<!-- Before -->
<img class="u-photo" src="{{- site.gravatar.photo_url | escape -}}" alt="{{- site.gravatar.photo_title | escape -}}" />

<!-- After -->
<img class="u-photo" src="{{- site.gravatar.photo_url | escape -}}" alt="{{- site.gravatar.photo_title | escape -}}" width="80" height="80" loading="lazy">
```

**Note:** Gravatar default size is 80x80. Added `loading="lazy"` since footer is below the fold.

#### 3b. Sohonet Logo in Homepage

**File:** `index.md`

**Changes:**

```markdown
<!-- Before -->
<img src="/assets/images/sohonet-logo.png" alt="Sohonet's logo" />

<!-- After -->
<img src="/assets/images/sohonet-logo.png" alt="Sohonet's logo" width="25" height="23">
```

**Note:** Actual image dimensions are 25x23 pixels (verified from file).

---

### 4. Fix `<html lang>` Attribute

**File:** `_config.yml`

**Problem:** The `lang` value uses underscore (`en_GB`) but HTML spec requires hyphen (`en-GB`).

**Background:** The underscore format (`en_GB`) is the Unix locale format. The HTML spec uses the hyphen format (`en-GB`) based on BCP 47 language tags. The original value was added in May 2020.

**Impact:** Minimal. This is primarily an accessibility/validation fix. Screen readers use the `lang` attribute to select correct pronunciation, and the hyphen format is the standard they expect.

**Changes:**

```yaml
# Before
lang: en_GB

# After
lang: en-GB
```

**Testing (2025-12-08):**

1. Verified local server output: `<html lang="en-GB">` - ✅ Correct

---

### 5. Enable CSS Minification

**File:** `_config.yml`

**Problem:** SASS outputs expanded CSS (17KB). Minification can reduce this by ~40-50%.

**What this does:** The `style: compressed` setting tells Jekyll's SASS compiler to minify the CSS output, removing all whitespace and newlines. The CSS works exactly the same, just smaller.

**Changes:**

```yaml
# Before
sass:
  quiet_deps: true

# After
sass:
  quiet_deps: true
  style: compressed
```

**Testing (2025-12-08):**

1. Measured CSS size before: 17,910 bytes
2. Built site with `bundle exec jekyll build`
3. Measured CSS size after: 11,781 bytes
4. **Result:** 34% reduction (~6 KB saved) ✅

---

### 6. Optimize Images (Optional)

**Problem:** PNG images could be smaller with optimization or modern formats.

**Files:**
- `/assets/images/sohonet-logo.png` (6.7 KB)
- `/assets/images/juliotrigo-logo-23.png` (13 KB)
- `/logo32.png` (3.6 KB)

**Options:**
1. Run through an optimizer like `optipng` or `pngquant`
2. Convert to WebP format (requires `<picture>` element for fallback)
3. Convert logos to SVG (scalable, often smaller)

**Recommendation:** For small logos like these, PNG optimization alone should suffice. WebP conversion is optional given the minimal savings (~7 KiB total).

---

### 7. Preload Lato Fonts to Reduce Critical Chain

**File:** `_includes/head.html`

**Problem:** PageSpeed flags a critical request chain with ~1,085ms latency. For the Ruby blog post, all 4 Lato font variants are in the critical path:

```
1. HTML page (~209ms)
   └── 2. main.css (~482ms) - CSS must wait for HTML
       ├── LatoLatin-Regular.woff2 (~1,004ms)
       ├── LatoLatin-Bold.woff2 (~1,085ms) ← LCP bottleneck
       ├── LatoLatin-Italic.woff2 (~901ms)
       └── LatoLatin-BoldItalic.woff2 (~967ms)
```

The browser doesn't know about the Lato fonts until it parses the CSS `@font-face` declarations, creating a waterfall delay.

**Analysis of font usage across the site:**

| Font Variant | Size | Used on |
|--------------|------|---------|
| LatoLatin-Regular | 43 KB | All pages (body text) |
| LatoLatin-Bold | 44 KB | Blog posts with `**bold**` text (9 posts + 1 draft) |
| LatoLatin-Italic | 45 KB | Blog posts with `*italic*` text (4 posts) |
| LatoLatin-BoldItalic | 45 KB | Blog posts with `***bold italic***` text (7 posts) |

Pages using only Regular (no bold/italic): `index.md`, `articles.md`, `bookmarks.md`, `books.md`, `presentations.md`, `projects.md`, `about.md`.

**Solution:** Add `<link rel="preload">` for all 4 Lato font variants. This tells the browser to start downloading the fonts immediately, in parallel with CSS parsing.

**Changes:**

```html
<!-- Add to head.html, before the main.css link -->
<link rel="preload" href="/assets/fonts/LatoLatin-Regular.woff2" as="font" type="font/woff2" crossorigin>
<link rel="preload" href="/assets/fonts/LatoLatin-Bold.woff2" as="font" type="font/woff2" crossorigin>
<link rel="preload" href="/assets/fonts/LatoLatin-Italic.woff2" as="font" type="font/woff2" crossorigin>
<link rel="preload" href="/assets/fonts/LatoLatin-BoldItalic.woff2" as="font" type="font/woff2" crossorigin>
```

**Why `crossorigin`:** Required for fonts even when self-hosted, due to how browsers handle font requests.

**Trade-off:** Pages that only use Regular (homepage, etc.) will download ~134 KB of fonts they don't use. This is acceptable because:
1. Most content pages (blog posts) use multiple font variants
2. The homepage is lightweight and fast anyway
3. 134 KB is relatively small on modern connections
4. Preloading only Regular would still leave Bold as the critical path bottleneck on blog posts

**Testing (2025-12-10):**

1. Built site with `bundle exec jekyll build` - ✅ Success
2. Verified generated HTML contains all 4 preload links - ✅ Present
3. Tested locally with `bundle exec jekyll serve` - ✅ Working
4. PageSpeed results for Ruby blog post:
   - **Before:** Maximum critical path latency 1,085ms (fonts in chain)
   - **After:** Maximum critical path latency 897ms (fonts removed from chain)
   - **Improvement:** ~188ms reduction in critical path latency

**Note:** The remaining HTML → CSS chain (897ms) is unavoidable and cannot be optimised with preloading. CSS is already discovered immediately in the `<head>`, so preloading it would be redundant. This chain represents the minimum critical path for any web page.

---

## Files to Modify

1. `_includes/head.html` - Preload Google Fonts, preload all 4 Lato font variants
2. `_includes/google-analytics.html` - Create/override with async loading
3. `_includes/footer.html` - Add image dimensions
4. `index.md` - Add image dimensions to Sohonet logo
5. `_config.yml` - Fix lang attribute, enable CSS minification

## Expected Results

After implementing these changes:
- **Render-blocking time:** Reduced by ~540ms
- **Unused JavaScript:** Reduced impact (async loading)
- **CLS:** Improved (explicit image dimensions)
- **CSS size:** Reduced by ~40-50% (minification)
- **Accessibility:** Improved (valid lang attribute)

Estimated new Performance score: **92-96** (mobile)
