---
name: write css less scss styles
description: "Use when writing or editing CSS/LESS/SCSS styles for a web feature. Covers where styles live (global ui-styles lib, not component-local), BEM naming, no nested selectors, comment convention, and a no-JS slide/expand animation technique."
---

Use when writing or modifying styles for a web feature — CSS, LESS, or SCSS — in the Baza NX monorepo.

## Where styles live

Component libraries (`ui-components`, feature libs) generally have **no local `.scss`/`.less` stylesheet and no inline `styles` array in `@Component`**. Global visual styling lives in a dedicated styles library, e.g. `libs/sandbox-web/ui-styles/src/lib/components/`, one file per component/page:

```
libs/sandbox-web/ui-styles/src/lib/components/_invest.less
libs/sandbox-web/ui-styles/src/lib/components/_header-mini.less
```

Before adding styles for a new or existing component:
1. Check whether a matching `_<component-name>.less` already exists in the relevant `ui-styles` lib — add to it rather than creating component-local styles.
2. If genuinely new, create `_<component-name>.less` following the existing naming pattern (leading underscore, kebab-case) and import it wherever the other partials are imported.
3. Do not reach for inline `@Component({ styles: [...] })` or a co-located `.scss` file unless the styling is truly one-off and has no home in the shared styles lib — check with the user first.

## Naming: BEM

Use BEM (`block__element--modifier`) for class names:

```less
.invest__note-expanded { ... }
.invest__note-expanded--open { ... }
.invest__note-expanded-content { ... }
```

- Block = the component/feature root (`invest`).
- Element = `block__element` (double underscore).
- Modifier = `block__element--modifier` (double dash), always written as its **own flat selector**, never nested under the element.

## No nested selectors

Even though LESS supports `&` nesting, **do not use it**. Every selector — including modifiers — is written as its own top-level flat rule:

```less
// wrong — nested modifier
.invest__note-expanded {
    display: grid;

    &--open {
        grid-template-rows: 1fr;
    }
}

// correct — flat, separate rule
.invest__note-expanded {
    display: grid;
    grid-template-rows: 0fr;
    transition: grid-template-rows 450ms ease;
}

.invest__note-expanded--open {
    grid-template-rows: 1fr;
}
```

`@media` blocks are the one exception — they're an at-rule, not selector nesting, and are written nested as usual (see `.invest__chart` in `_invest.less`).

## Comment convention

Every rule is preceded by a one-line comment that mirrors the selector's block/element/modifier name in plain words:

```less
// invest note-expanded
.invest__note-expanded {
    ...
}

// invest note-expanded--open
.invest__note-expanded--open {
    ...
}
```

## Slide/expand animation without JS height calc

To animate a block from hidden to its natural (auto) height — e.g. expanding a "show more" note — use the CSS grid `0fr` → `1fr` trick instead of `max-height` hacks or Angular animations. Keep the element always in the DOM (don't gate it with `*ngIf`); toggle a modifier class instead:

```less
.invest__note-expanded {
    display: grid;
    grid-template-rows: 0fr;
    transition: grid-template-rows 450ms ease;
}

.invest__note-expanded--open {
    grid-template-rows: 1fr;
}

.invest__note-expanded-content {
    overflow: hidden;
}
```

```html
<div class="invest__note-expanded" [class.invest__note-expanded--open]="isNoteExpanded">
    <div class="invest__note-expanded-content" [innerHTML]="..."></div>
</div>
```

The outer element is the grid row that animates; the inner content element needs `overflow: hidden` so it doesn't visually escape while the row is collapsing/expanding.
