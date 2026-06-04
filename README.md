# Angular Frontend Skills

A collection of 12 skills for Angular frontend development. Covers project setup, component patterns, state management, design system, and tooling.

Compatible with **Claude Code** and **OpenAI Codex**.

## Skills

| Skill | Description |
|---|---|
| `add-analytics` | Use when adding web analytics to a project. Covers GTM setup via PUBLIC_GTM_ID env variable, disabled in local dev. All other tools (Yandex Metrika, Google Analytics) are configured through GTM tags. |
| `add-css-framework` | Use when adding Bootstrap (via ng-bootstrap) as the CSS framework to an Angular project. Covers integrating from source SCSS files instead of the minified bundle for full customization control. |
| `add-favicon` | Use when adding favicons to a web project. Covers generating all required formats (svg, ico, png) and the full set of HTML link tags for cross-browser and PWA support. |
| `add-meta-tags` | Use when adding SEO meta tags to a web project. Covers the full set of Open Graph, Twitter Card, and standard SEO meta tags with examples for Next.js and Angular. |
| `create-angular-component` | Use when creating any Angular component. Covers Angular 21+ patterns: standalone components, signals, input(), inject(), async/await with run() helper, ngModel forms, @if/@for template syntax. No OnPush, no NgModules, no UntilDestroy. |
| `customize-bootstrap` | Use when customizing Bootstrap components in an Angular project. Covers the four levels of customization — variables first, then overrides. |
| `fetch-feature-data-ngxs` | Use when deciding how to fetch and store data in an Angular app. Covers when to use NGXS store vs plain service, store folder structure, actions/state/selectors patterns, and lazy-load-triggered data fetching. |
| `fetch-global-data-ngxs` | Use when data must be loaded once on app start and available across all features. Covers the Bootstrap store + Resolver pattern: global state, cache-first resolver, attached to the root route. |
| `handle-deeplinks` | Use when a project has native iOS/Android apps and needs URL-based deep linking. Covers Universal Links/App Links with web fallback and AASA file setup. |
| `setup-color-palette` | Use when setting up SCSS color variables in a project. Covers the _palette.scss + _bootstrap.scss + variables.scss structure for organizing base colors, shades, and functional colors. |
| `setup-design-tokens` | Use when connecting Figma design variables to the codebase. Covers Tokens Studio plugin + Style Dictionary workflow for a single source of truth without manual syncing. |
| `setup-icon-system` | Use when setting up an icon system in a web project. Covers two approaches: icon font for monochrome icons (recommended first choice) and SVG sprites for multicolor icons. |

## Installation

### Claude Code

```
claude plugin add m8ig/angular-fe-skills
```

### OpenAI Codex

```bash
git clone https://github.com/m8ig/angular-fe-skills /tmp/angular-fe-skills
cp -r /tmp/angular-fe-skills/skills/* ~/.codex/skills/
```

## Usage

### Claude Code

```
/create-angular-component
/setup-color-palette
```

### OpenAI Codex

```
$create-angular-component
$setup-color-palette
```

Skills also activate automatically when your task matches the skill description.
