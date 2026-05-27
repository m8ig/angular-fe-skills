---
name: add css framework to angular project
description: Use when adding Bootstrap (via ng-bootstrap) as the CSS framework to an Angular project. Covers integrating from source SCSS files instead of the minified bundle for full customization control.
---

Use Bootstrap (via ng-bootstrap) as the CSS framework. Always integrate from source SCSS files — not the minified bundle — for full customization control.

## Install

```
ng add @ng-bootstrap/ng-bootstrap
```

## SCSS folder structure

Create `/src/styles/` with the following layout:

```
/src
  /styles
    /_               ← auto-generated files (icon fonts, design tokens)
    /common          ← typography, fonts, icons, container, helpers
    /components      ← app-specific component styles
    /forms           ← form element overrides (when variables aren't enough)
    /mixins          ← reusable SCSS mixins (optional)
    /variables       ← all variable files
      ├── _palette.scss
      ├── _bootstrap.scss
      └── variables.scss
    /vendor          ← third-party component overrides (non-form)
    ├── app.scss
    └── vendor.scss
  styles.scss
```

## vendor.scss — correct import order (critical)

```scss
@import 'bootstrap/scss/mixins/banner';
@include bsBanner('');

// 1. Bootstrap functions (required by everything)
@import 'bootstrap/scss/functions';
// 2. Bootstrap defaults
@import 'bootstrap/scss/variables';

// 3. Our overrides — MUST come after Bootstrap defaults
@import 'variables/variables';

// 4. Bootstrap builds its system using our colors
@import 'bootstrap/scss/maps';
@import 'bootstrap/scss/mixins';
@import 'bootstrap/scss/utilities';

// Core
@import 'bootstrap/scss/root';
@import 'bootstrap/scss/reboot';
@import 'bootstrap/scss/type';
@import 'bootstrap/scss/containers';
@import 'bootstrap/scss/grid';

// Forms
@import 'bootstrap/scss/forms';
@import 'bootstrap/scss/buttons';
@import 'bootstrap/scss/transitions';

// Components (comment out unused ones to reduce bundle size)
@import 'bootstrap/scss/navbar';
@import 'bootstrap/scss/card';
@import 'bootstrap/scss/pagination';
@import 'bootstrap/scss/badge';
@import 'bootstrap/scss/alert';
@import 'bootstrap/scss/modal';
@import 'bootstrap/scss/placeholders';

// Helpers + utilities
@import 'bootstrap/scss/helpers';
@import 'bootstrap/scss/utilities/api';
```

## styles.scss

```scss
@import 'vendor.scss';
@import 'app.scss';
```

## stylePreprocessorOptions — for Angular project

In `angular.json`:

```json
"build": {
  "options": {
    "stylePreprocessorOptions": {
      "includePaths": ["src/styles/"]
    }
  }
}
```

## stylePreprocessorOptions — for NX monorepo

In `project.json` of each app:

```json
"build": {
  "options": {
    "stylePreprocessorOptions": {
      "includePaths": ["libs/ui-styles/src/lib"]
    }
  }
}
```

With this setup, any Angular component can import variables without relative paths:

```scss
@import 'variables/variables';

.hero {
  background-color: $primary;
}
```
