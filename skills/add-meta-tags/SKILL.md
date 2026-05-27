---
name: add meta-tags
description: Use when adding SEO meta tags to a web project. Covers the full set of Open Graph, Twitter Card, and standard SEO meta tags with examples for Next.js and Angular.
---

You need to generate and paste all possible meta-tags for SEO optimization.
This is an example from a Next.js project. Please use it as a reference:

```
{
	title: content.layout.title,
	description: content.layout.description,
	alternates: {
		canonical: content.siteUrl,
	},
	openGraph: {
		type: 'website',
		url: content.siteUrl,
		title: content.layout.title,
		description: content.layout.description,
		images: [{ url: '/og-image.png' }],
	},
	twitter: {
		card: 'summary_large_image',
		title: content.layout.title,
		description: content.layout.description,
		images: ['/og-image.png'],
	},
```

For `og-image` is enough to make a screenshot of the home page with the right size.
If you can't do it, please notify the user.
