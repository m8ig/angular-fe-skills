---
name: set up icon system
description: "Use when setting up an icon system in a web project. Covers two approaches: icon font for monochrome icons (recommended first choice) and SVG sprites for multicolor icons."
---

Two approaches depending on whether icons are monochrome or multicolor.

## Option A — Icon font (monochrome icons, recommended first choice)

Best for: single-color icons, easy CSS control (`font-size`, `color`).  
Limitation: no multi-color support.

### Folder structure

```
/src
  /assets
    /fonts      ← generated font files go here
    /icons      ← source SVG files go here
  /styles
    /_           ← auto-generated SCSS files go here
```

### SVG preparation rules

1. All icons must be square (equal width and height)
2. Same pixel dimensions for all icons (e.g. 50×50px)
3. Convert all fills/strokes/effects to discrete objects
4. Remove all editor comments from SVG code (e.g. Adobe Illustrator generator comment)

### Install Fantasticon

```
yarn add fantasticon --dev
```

### .fantasticonrc (project root)

```js
module.exports = {
    name: 'Your-Icons',
    inputDir: './src/assets/icons',
    outputDir: './src/assets/fonts',
    fontTypes: ['eot', 'woff2', 'woff', 'ttf'],
    assetTypes: ['css'],
    fontsUrl: '/assets/fonts',
    pathOptions: {
        css: './src/styles/_/icon-font.scss',
    },
};
```

### package.json script

```json
"icon": "fantasticon -c .fantasticonrc"
```

### Run and import

```
yarn icon
```

In `src/styles/app.scss`:
```scss
// auto-generated files
@import './_/icon-font';
```

In `angular.json` — add fonts to assets:
```json
"assets": [
    "src/assets/fonts/",
    { "glob": "**/*", "input": "public" }
]
```

### Usage in HTML

```html
<i class="icon icon-angle-left"></i>
```

---

## Option B — SVG sprites (multi-color icons)

Best for: icons with 2+ colors. No color or size limitation.

### Additional folders

```
/src/assets/icons-sprite    ← source SVG files (exact final size/color)
/src/assets/sprites         ← generated sprite goes here
```

No SVG preparation needed — icons go in as-is.

### Install svg2sprite-cli globally

```
npm i -g svg2sprite-cli
```

### package.json script

```json
"icon:sprite": "svg2sprite ./src/assets/icons-sprite ./src/assets/sprites/icons.svg"
```

### Install ng-svg-icon-sprite in project

```
yarn add ng-svg-icon-sprite --dev
```

In `app.config.ts`:
```ts
import { IconSpriteModule } from 'ng-svg-icon-sprite';

providers: [
    importProvidersFrom(
        IconSpriteModule.forRoot({ path: 'assets/sprites/icons.svg' })
    ),
]
```

In component:
```ts
import { IconSpriteModule } from 'ng-svg-icon-sprite';

@Component({
    imports: [IconSpriteModule],
    ...
})
```

```html
<svg-icon-sprite [src]="'icon-name'"></svg-icon-sprite>
```

### Regenerating

Add new SVGs to `/icons-sprite` and run:
```
yarn icon:sprite
```
