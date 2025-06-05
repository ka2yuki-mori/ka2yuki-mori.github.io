# Security
# 脆弱性診断（Vulnerability Assessment）
アプリやWebサイト、サーバーなどに既知の脆弱性がないかをツールなどでチェックする作業

## npm audit : 
  - Node.jsの依存パッケージの脆弱性**のみ**をチェック
  - HTML/CSSや独自のJavaScriptコードの脆弱性（XSS等）はチェック**しません**。
  - これには ESLint + セキュリティルール や、手動のレビュー・専用ツールが必要

上記を忘れないように GitHub Actions のワークフロー作成で自動化: sample  
:arrow_right:「set up this workflow」
```yml
name: Run npm audit

on:
  push:
    branches: [ main ]

jobs:
  audit:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v4
        with:
          node-version: '18'
      - run: npm install
      - run: npm audit
```

:book: [Content Security Policy Level 3におけるXSS対策](https://inside.pixiv.blog/kobo/5137) | pixiv.blog