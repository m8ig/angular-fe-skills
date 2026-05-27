---
name: customize bootstrap components
description: Use when customizing Bootstrap components in an Angular project. Covers the four levels of customization — variables first, then overrides — with Ng-Zorro or Bootstrap.
---

There are four levels of customization for any Bootstrap component. Always start from the top — use variables first, only drop to overrides when variables aren't enough.

## Levels of customization

| What you want to change | Where to put it |
|---|---|
| All instances of a component (e.g. all buttons) | `variables/_bootstrap.scss` |
| A specific variant (e.g. primary button only) | `variables/_palette.scss` |
| A component inside a specific third-party context | `styles/vendor/_component.scss` |
| A component inside your own component | Component's own `.scss` file |

## _bootstrap.scss — variable overrides

Always check Bootstrap's `_variables.scss` first. Most sizing, spacing, font, and radius properties have a variable:

```scss
// Buttons — applied to ALL button sizes/variants
$btn-padding-y:      7px;
$btn-padding-x:      30px;
$btn-font-family:    Satoshi, sans-serif;
$btn-font-size:      14px;
$btn-line-height:    1.43;
$btn-font-weight:    700;
$btn-border-radius:  100px;

// Alerts
$alert-padding-y:    16px;
$alert-padding-x:    16px;
$alert-border-radius: 4px;
$alert-border-width: 0px;

// Typography (applies globally)
$font-family-sans-serif: Satoshi, sans-serif;
```

## forms/ folder — form element overrides

Use for form element styles that can't be done via variables. One file per element.

`/src/styles/forms/_btn.scss`:
```scss
.btn {
    text-transform: uppercase;
}
```

`/src/styles/forms/_input.scss`:
```scss
.form-control {
    // overrides here
}
```

Import all form overrides in `app.scss`:
```scss
// forms styles
@import './forms/_btn';
@import './forms/_input';
```

## vendor/ folder — third-party component overrides

Use for non-form Bootstrap components (modals, alerts, nav, etc.) or any other third-party library styles. One file per component.

`/src/styles/vendor/_alert.scss`:
```scss
.alert {
    font-weight: 500;
}
```

Import all vendor overrides in `app.scss` after forms:
```scss
// forms styles
@import './forms/_btn';

// vendor styles
@import './vendor/_alert';
```

## Rule

**forms/** = form elements only (`input`, `button`, `select`, `checkbox`, etc.)  
**vendor/** = everything else, including form elements when they appear inside a specific component context
