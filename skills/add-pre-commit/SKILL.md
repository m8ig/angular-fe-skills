---
name: add pre-commit checks
description: Use when setting up pre-commit hooks for a project. Covers Husky + Commitlint + lint-staged configuration to enforce code quality and commit message format before every commit.
---

Install Husky & Commitlint with the following config:

1) Add Husky to run
	1) pre-commit: yarn lint-staged
	
```
"lint-staged": {
	"*.{js,mjs,ts,jsx,tsx}": [
		"eslint --fix",
		"prettier --write"
	],
	"*.{md,html,less}": [
		"prettier --write"
	]
}
```

	2) commit-msg: yarn commitlint --edit $1

```
module.exports = {
	extends: ['@commitlint/config-conventional'],
	parserPreset: {
		parserOpts: {
			issuePrefixes: ['SHORT-NAME-OF-PROJECT-'], // prefix should be optional
		},
	},
};
```
