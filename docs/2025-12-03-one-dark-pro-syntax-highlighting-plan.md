# Plan: Implement One Dark Pro Syntax Highlighting

## Current Setup

- **Markdown processor**: kramdown
- **Syntax highlighter**: Rouge (Jekyll's default, Pygments-compatible)
- **Current theme**: Base16 Eighties Dark
- **Key files**:
  - `_sass/base16-eighties-dark.scss` - Color variables
  - `_sass/pygments-template.scss` - Token-to-color mappings
  - `_sass/juliotrigo/_syntax-highlighting.scss` - Additional syntax styles
  - `assets/main.scss` - Main stylesheet that imports all SCSS files

## Approach

Create a new SCSS file with One Dark Pro color variables, replacing the Base16 Eighties Dark import.

## One Dark Pro Color Palette

Based on the official VS Code theme:

| Variable | Hex | Usage |
|----------|-----|-------|
| $base00 | #282c34 | Background |
| $base01 | #2c313c | Line highlight |
| $base02 | #3e4451 | Selection |
| $base03 | #7f848e | Comments |
| $base04 | #5c6370 | Dark foreground |
| $base05 | #abb2bf | Foreground |
| $base06 | #c8ccd4 | Light foreground |
| $base07 | #d7dae0 | Lightest foreground |
| $base08 | #e06c75 | Red (variables, tags) |
| $base09 | #d19a66 | Orange (numbers, attributes) |
| $base0a | #e5c07b | Yellow (classes, types) |
| $base0b | #98c379 | Green (strings) |
| $base0c | #56b6c2 | Cyan (operators, regex) |
| $base0d | #61afef | Blue (functions) |
| $base0e | #c678dd | Purple (keywords) |
| $base0f | #be5046 | Dark red (deprecated) |

## Implementation Steps

1. Create `_sass/one-dark-pro.scss` with the color variables above
2. Update `assets/main.scss` to import `one-dark-pro` instead of `base16-eighties-dark`
3. Update `$code-background` variable if needed in `_sass/juliotrigo.scss` or related file
4. Test locally with `bundle exec jekyll serve`
5. Verify syntax highlighting on pages with code blocks

## Files to Modify

- `_sass/one-dark-pro.scss` (new file)
- `assets/main.scss` (change import)
- Possibly `_sass/juliotrigo.scss` or `_sass/juliotrigo/_base.scss` (if $code-background is defined there)

## Rollback

If needed, revert `assets/main.scss` to import `base16-eighties-dark` instead of `one-dark-pro`.

## Sources

- [One Dark Pro - VS Code Marketplace](https://marketplace.visualstudio.com/items?itemName=zhuangtongfa.Material-theme)
- [One Dark Pro Theme JSON - GitHub](https://github.com/Binaryify/OneDark-Pro/blob/master/themes/OneDark-Pro.json)
- [OneDarkJekyll - GitHub](https://github.com/mgyongyosi/OneDarkJekyll)

---

## Implementation Summary

**Completed on:** 2025-12-03

### Files Created

- `_sass/one-dark-pro.scss` - One Dark Pro color variables mapped to Base16 variable names

### Files Modified

- `assets/main.scss` - Changed import from `base16-eighties-dark` to `one-dark-pro`
- `_sass/juliotrigo.scss` - Updated `$code-background` from `#2E3440` (Nord) to `#282c34` (One Dark Pro)

### Testing

- Ran `bundle exec jekyll serve` successfully
- Build completed without errors
- Site served at http://127.0.0.1:4000/

### Notes

- The existing `pygments-template.scss` was reused without modification. This file maps Rouge token classes (e.g., `.c` for comments, `.s` for strings) to Base16 variable names (`$base00`-`$base0f`), not actual color values. By defining the same variable names in `one-dark-pro.scss`, the template automatically picks up the new colors.
- Sass `@import` deprecation warnings appeared during build (unrelated to this change; will require migration to `@use` in the future)

### Color Comparison: Base16 Eighties Dark vs One Dark Pro

The two themes have similar palettes (both are Atom-era dark themes):

| Token | Base16 Eighties Dark | One Dark Pro |
|-------|---------------------|--------------|
| Background | `#2d2d2d` | `#282c34` |
| Comments | `#747369` | `#7f848e` |
| Red | `#f2777a` | `#e06c75` |
| Orange | `#f99157` | `#d19a66` |
| Yellow | `#ffcc66` | `#e5c07b` |
| Green | `#99cc99` | `#98c379` |
| Cyan | `#66cccc` | `#56b6c2` |
| Blue | `#6699cc` | `#61afef` |
| Purple | `#cc99cc` | `#c678dd` |

Key differences:
- **Background**: One Dark Pro is slightly bluer (`#282c34` vs `#2d2d2d`)
- **Orange/Yellow**: One Dark Pro is more muted/desaturated
- **Red**: One Dark Pro is slightly darker and less pink

### Verification

Colors confirmed from the latest One Dark Pro master branch:
https://raw.githubusercontent.com/Binaryify/OneDark-Pro/master/themes/OneDark-Pro.json
