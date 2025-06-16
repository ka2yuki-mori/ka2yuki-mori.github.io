mise と rbenv の違いとカテゴリ

## 違い

**mise**  
- 複数言語（Ruby, Node.js, Python など）のバージョン管理ができるツール  
- asdf 互換で、1つのコマンドで多言語を管理できる  
- `.tool-versions` ファイルで管理  

**rbenv**  
- Ruby 専用のバージョン管理ツール  
- `.ruby-version` ファイルで管理  

## カテゴリ

- mise や rbenv は「バージョン管理ツール（バージョンマネージャー）」のカテゴリです。
- パッケージマネージャー（例: gem, npm, yarn）は「ライブラリやパッケージの依存管理ツール」です。

## まとめ

- mise/rbenv は「バージョンマネージャー」
- gem/npm/yarn は「パッケージマネージャー」