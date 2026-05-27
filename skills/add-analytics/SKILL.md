---
name: add analytics to project
description: Use when adding web analytics to a project. Covers GTM setup via PUBLIC_GTM_ID env variable, disabled in local dev. All other tools (Yandex Metrika, Google Analytics) are configured through GTM tags.
---

1) Add the GTM script to all pages
2) GTM_ID should be inserted as an env variable from the file with the PUBLIC_GTM_ID name
3) This feature should be turned off during local development
4) All other tools, like Y.Metrika and Google Analytics, will be set up via the GTM tag.
