---
title: "eslint + typescript 配置参考"
date: 2021-04-18
tags: [TypeScript] 
categories: [前端笔记]
---

```bash
npm i eslint eslint-plugin-react @typescript-eslint/parser @typescript-eslint/eslint-plugin

```

.eslintrc.js

```css
module.exports = {
  parser: '@typescript-eslint/parser',
  parserOptions: {
    project: './tsconfig.json',
  },
  plugins: ['@typescript-eslint'],
  extends: [
    'plugin:react/recommended',
    'plugin:@typescript-eslint/recommended',
  ],
  rules:  {
    // Overwrite rules specified from the extended configs e.g. 
    // "@typescript-eslint/explicit-function-return-type": "off",
  }
}
```

vscode: settings.json

```json
"eslint.validate":  [
  "javascript",
  "javascriptreact",
  {"language":  "typescript",  "autoFix":  true  },
  {"language":  "typescriptreact",  "autoFix":  true  }
],
```