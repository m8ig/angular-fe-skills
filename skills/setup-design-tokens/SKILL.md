---
name: set up design tokens (figma to scss)
description: Use when connecting Figma design variables to the codebase. Covers Tokens Studio plugin + Style Dictionary workflow for a single source of truth without manual syncing.
---

Connect Figma variables to the codebase via Tokens Studio plugin + Style Dictionary. One source of truth — no manual syncing.

## How it works

```
Figma Variables (Tokens Studio plugin)
  ↓ push to GitHub
design-tokens/tokens.json
  ↓ npm run tokens:build (Style Dictionary)
src/styles/_/tokens.generated.scss  ← SCSS vars + CSS custom props
src/design-tokens.generated.ts      ← TypeScript constants
  ↓ @import in _palette.scss
Bootstrap theme ($primary, $success, etc.)
```

## 1. Token file

Create `/design-tokens/tokens.json`:

```json
{
    "$description": "Design tokens — source of truth for brand colors.",
    "token": {
        "primary":   { "$type": "color", "$value": "#0d6efd", "$description": "Figma: Brand/Primary" },
        "secondary": { "$type": "color", "$value": "#6c757d", "$description": "Figma: Brand/Secondary" },
        "success":   { "$type": "color", "$value": "#198754", "$description": "Figma: Brand/Success" },
        "info":      { "$type": "color", "$value": "#0dcaf0", "$description": "Figma: Brand/Info" },
        "warning":   { "$type": "color", "$value": "#ffc107", "$description": "Figma: Brand/Warning" },
        "danger":    { "$type": "color", "$value": "#dc3545", "$description": "Figma: Brand/Danger" }
    }
}
```

## 2. Figma setup

In Figma → Local variables → create collection named `Brand` with the same six variables:
```
Brand/Primary, Brand/Secondary, Brand/Success, Brand/Info, Brand/Warning, Brand/Danger
```

Naming rule: the sync script matches by the **last path segment** (case-insensitive). `Brand/Primary` → matches `primary` token.

## 3. Tokens Studio plugin (Figma)

Install "Tokens Studio for Figma" from Figma Community.

Settings → Sync → GitHub:
```
Repository:  your-org/your-repo
Branch:      main
File path:   design-tokens/tokens.json
Secret:      GitHub Personal Access Token (repo scope)
```

Enable **"Use Figma Variables"** to import the Brand collection directly as tokens.

Designer updates colors in Figma → pushes from the plugin → PR appears in GitHub.

## 4. Style Dictionary build

```
yarn add style-dictionary @tokens-studio/sd-transforms --dev
```

Create `/design-tokens/build-tokens.mjs`:

```js
import StyleDictionary from 'style-dictionary';
import { register }    from '@tokens-studio/sd-transforms';
import { dirname, join } from 'path';
import { fileURLToPath } from 'url';

register(StyleDictionary);

const __dirname = dirname(fileURLToPath(import.meta.url));
process.chdir(join(__dirname, '..'));

const sd = new StyleDictionary({
    usesDtcg: true,
    preprocessors: ['tokens-studio'],
    source: ['design-tokens/tokens.json'],
    platforms: {
        scss: {
            transforms: ['ts/color/css/hexrgba', 'ts/opacity', 'ts/resolveMath', 'name/kebab'],
            buildPath: 'src/styles/_/',
            files: [{ destination: 'tokens.generated.scss', format: 'scss/tokens-with-custom-props' }],
        },
        js: {
            transforms: ['ts/color/css/hexrgba', 'ts/opacity', 'ts/resolveMath', 'name/camel'],
            buildPath: 'src/',
            files: [{ destination: 'design-tokens.generated.ts', format: 'javascript/es6' }],
        },
    },
    hooks: {
        formats: {
            'scss/tokens-with-custom-props': ({ dictionary }) => {
                const tokens = dictionary.allTokens;
                const now    = new Date().toISOString().slice(0, 10);
                const pad    = Math.max(...tokens.map(t => t.name.length)) + 2;
                const val    = t => t.$value ?? t.value;
                const scssVars = tokens.map(t => `$${t.name.padEnd(pad)}: ${val(t)};`).join('\n');
                const cssProps = tokens.map(t => `  --${t.name.padEnd(pad)}: ${val(t)};`).join('\n');
                return [
                    `// AUTO-GENERATED — DO NOT EDIT DIRECTLY`,
                    `// Source: design-tokens/tokens.json`,
                    `// Rebuild: npm run tokens:build`,
                    ``, scssVars, ``,
                    `:root {`, cssProps, `}`, ``,
                ].join('\n');
            },
        },
    },
});

await sd.buildAllPlatforms();
```

In `package.json`:
```json
"tokens:build": "node design-tokens/build-tokens.mjs"
```

Run once to generate files:
```
npm run tokens:build
```

## 5. Connect tokens to Bootstrap

In `src/styles/variables/_palette.scss`, replace hardcoded colors with token imports:

```scss
// ── Design Tokens ──────────────────────────────────────────────────────────
// Source of truth: design-tokens/tokens.json
// To update: edit tokens.json → npm run tokens:build
@import '../_/tokens.generated';

// Map token variables → Bootstrap theme colors
$primary:   $token-primary;
$secondary: $token-secondary;
$success:   $token-success;
$info:      $token-info;
$warning:   $token-warning;
$danger:    $token-danger;
```

The generated files (`tokens.generated.scss`, `design-tokens.generated.ts`) must never be edited manually — they live in `src/styles/_/` and `src/` respectively.

## .gitignore

Do NOT ignore the generated files — they need to be committed so CI and other developers don't need to run the build step to get a working project. Only ignore them if you have CI that runs `tokens:build` on every build.
