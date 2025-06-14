# GitHub Pages セキュリティ対策チェックリスト

## 1. 外部ライブラリ・プラグインの管理
- [ ] 使用しているライブラリ（例：jQuery, Bootstrap）は最新版か？
- [ ] CDNで読み込む場合は SRI（Subresource Integrity）を使用しているか？
- [ ] メンテナンスされていないプラグインやGitHubの過去リポジトリは使っていないか？

## 2. クロスサイト・スクリプティング（XSS）対策
- [ ] ユーザー入力やURLクエリをそのままHTMLに出力していないか？
- [ ] DOM操作（innerHTMLなど）を使うとき、値のサニタイズをしているか？
- [ ] Markdownなどを動的にHTML化する場合、安全なパーサー（例：marked + DOMPurify）を使っているか？

## 3. 外部API・フォームの安全性
- [ ] Googleフォームや他社のフォームサービスを使う場合、HTTPSで安全に送信されているか？
- [ ] API連携時、信頼できるエンドポイントのみを使用しているか？
- [ ] CORSやRefererチェックで不要な外部呼び出しを制限しているか？

## 4. コンテンツの改ざん防止
- [ ] GitHubリポジトリの管理者アクセスが適切に制限されているか？
- [ ] GitHub Actionsを使っている場合、セキュリティに配慮されたworkflowになっているか？

## 5. ソースコードと構成管理
- [ ] `.env`ファイルや機密情報がコミットされていないか？
- [ ] `_config.yml` や `CNAME` に意図しない設定がされていないか？
- [ ] GitHub Pages のビルド環境（Jekyllなど）のバージョンを明示的に指定しているか？

## 6. テスト・診断
- [ ] Security HeadersやMozilla ObservatoryでHTTPヘッダーのチェックを行ったか？
- [ ] ページ上のスクリプトやリンクに混在コンテンツ（http）が含まれていないか？
