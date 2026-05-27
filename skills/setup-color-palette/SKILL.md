---
name: set up color palette (scss)
description: Use when setting up SCSS color variables in a project. Covers the _palette.scss + _bootstrap.scss + variables.scss structure for organizing base colors, shades, and functional colors.
---

Color variables live in two separate files inside `/src/styles/variables/`:

- `_palette.scss` — all color definitions (base colors, shades, functional colors)
- `_bootstrap.scss` — everything else (typography, spacing, components)
- `variables.scss` — the single entry point that imports both

## _palette.scss

Must import Bootstrap functions at the top (needed for `tint-color()` / `shade-color()`):

```scss
@import 'bootstrap/scss/functions';

// ── Base colors ───────────────────────────────────────
$white:    #fff;
$gray-100: #f8f9fa;
$gray-200: #e9ecef;
$gray-300: #dee2e6;
$gray-400: #ced4da;
$gray-500: #adb5bd;
$gray-600: #6c757d;
$gray-700: #495057;
$gray-800: #343a40;
$gray-900: #212529;
$black:    #000;

$blue:   #2175f2;
$green:  #198754;
$red:    #dc3545;
$yellow: #ffc107;
$cyan:   #0dcaf0;

// ── Functional colors (map these to Bootstrap theme) ──
$primary:   $blue;
$secondary: $gray-600;
$success:   $green;
$info:      $cyan;
$warning:   $yellow;
$danger:    $red;
$light:     $gray-100;
$dark:      $gray-900;

// ── Shade scale for each functional color ─────────────
// Hover / active states should always derive from these, not hardcoded hex values.
$primary-1: tint-color($primary, 80%);
$primary-2: tint-color($primary, 60%);
$primary-3: tint-color($primary, 40%);
$primary-4: tint-color($primary, 20%);
$primary-5: $primary;
$primary-6: shade-color($primary, 20%);
$primary-7: shade-color($primary, 40%);
$primary-8: shade-color($primary, 60%);

// Repeat the same pattern for $success, $danger, $warning if needed.

// ── Subtle variants (Bootstrap Alert, Badge backgrounds) ──
$success-text-emphasis: #141414;
$info-text-emphasis:    #141414;
$warning-text-emphasis: #141414;
$danger-text-emphasis:  #141414;

$success-bg-subtle: #69d68f;
$info-bg-subtle:    #91d5ff;
$warning-bg-subtle: #ffe1b8;
$danger-bg-subtle:  #ffaf85;
```

## variables.scss — entry point

```scss
@import './_palette.scss';
@import './_bootstrap.scss';
```

Import `variables/variables` everywhere you need access to variables — in component SCSS files and in `vendor.scss`.

## Usage rule

Always use functional color variables and their shades (`$primary`, `$primary-7`) instead of base color names (`$blue`, `$blue-700`). This way a full color scheme change requires editing only the functional variable assignments at the top of `_palette.scss`.
