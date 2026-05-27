---
name: add favicon to the project
description: Use when adding favicons to a web project. Covers generating all required formats (svg, ico, png) and the full set of HTML link tags for cross-browser and PWA support.
---

Check if it's possible to generate all kinds of favicons for the project, including `svg`, `ico`, `png`, etc. This is an example of the result you need to get:

```html
<link rel="icon" type="image/png" href="/favicon-96x96.png" sizes="96x96" />
<link rel="icon" type="image/svg+xml" href="/favicon.svg" />
<link rel="shortcut icon" href="/favicon.ico" />
<link rel="apple-touch-icon" sizes="180x180" href="/apple-touch-icon.png" />
<link rel="manifest" href="/site.webmanifest" />
```

If you don't have the right icon to generate from or you can't generate it by yourself, suggest doing it for a user and remind them about https://realfavicongenerator.net.
