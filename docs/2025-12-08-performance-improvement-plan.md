# Performance Improvement Plan

This plan addresses the performance issues identified by PageSpeed Insights for juliotrigo.com (mobile score: 88).

## Progress

| Step | Description | Status |
|------|-------------|--------|
| 1 | Fix render-blocking Google Fonts | ✅ Completed (2025-12-08) |
| 2 | Optimize Google Analytics loading | ⏳ Pending |
| 3 | Add explicit dimensions to images | ⏳ Pending |
| 4 | Fix `<html lang>` attribute | ⏳ Pending |
| 5 | Enable CSS minification | ⏳ Pending |
| 6 | Optimize images (optional) | ⏳ Pending |

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

**File:** Create `_includes/google-analytics.html` (override Minima's default)

**Problem:** Google Analytics (gtag.js) loads synchronously, contributing 54 KiB of unused JavaScript on initial load.

**Solution:** Ensure the script loads with `async` attribute and consider deferring until after page load.

**Changes:**

```html
<!-- Async loading of gtag.js -->
<script async src="https://www.googletagmanager.com/gtag/js?id={{ site.google_analytics }}"></script>
<script>
  window.dataLayer = window.dataLayer || [];
  function gtag(){dataLayer.push(arguments);}
  gtag('js', new Date());
  gtag('config', '{{ site.google_analytics }}');
</script>
```

**Note:** The current `google_analytics` value (`UA-104164449-1`) is a Universal Analytics ID (deprecated). The PageSpeed results show `G-EPMRDFVSSX` being loaded, which is a GA4 ID. Verify which analytics setup is active and consider consolidating.

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

**Changes:**

```yaml
# Before
lang: en_GB

# After
lang: en-GB
```

---

### 5. Enable CSS Minification

**File:** `_config.yml`

**Problem:** SASS outputs expanded CSS (17KB). Minification can reduce this by ~40-50%.

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

## Files to Modify

1. `_includes/head.html` - Preload Google Fonts
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
