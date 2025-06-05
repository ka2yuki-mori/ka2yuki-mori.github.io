# Railsめも

# Rails DBコマンド
| コマンド | 概要 |
|---------|------|
| rails db:create | データベースを作成する（SQLite3の場合は不要） |
| rails db:drop | データベースを削除する（SQLite3の場合は「.sqlite3」ファイル自体を削除） |
| rails db:migrate | マイグレーションファイルの内容をDBに反映させる |
| rails db:migrate:status | 現在のマイグレーション状態を表示する |
| rails db:prepare | DBが存在すればマイグレーションを、存在しなければデータベースのセットアップを実行する |
| rails db:reset | データベースをdropし、db:setupを実行する |
| rails db:rollback | 実行したマイグレーションを取り消す |
| rails db:schema:dump | DB構成(スキーマファイル)をdb/schema.rbへ出力 |
| rails db:schema:load | db/schema.rb(スキーマファイル)内容をDBへ反映 |
| rails db:seed | db/seeds.rbの内容を実行する |
| rails db:replant | DBの内容をTRUNCATEし、db:seedを実行する |
| rails db:setup | データベースを作成し、スキーマの読み込みとシードデータの読み込みを行う |
| rails db:version | 現在適用しているマイグレーションのバージョンを表示する |

マイグレを戻す
- １ステップ前に戻りたい場合：`bin/rails db:rollback`
- 任意の状態まで戻したい場合：`bin/rails db:rollback STEP=2` 

あらかじめ用意した(テスト)データ読み込み
- `db:seed`で `db/seeds.rb`からデータ投入
- 「！」付きメソッドでエラー発生にすぐ気がつける

```db/seeds.rb
# db/seeds.rb
Blog.create!(name: 'cool blog')
```
```sh
rails db:seed # テストデータ反映
rails db:seed # 実行回数分反映される
rails runner "p Blog.count" # 確認
> 2
rails db:seed:replant # 削除 ⇒ 再実行
```

```sh
rails db:drop # データベース削除
rails db:setup # データベース作成、テーブル構築、seed読み込み
rails runner "p Blog.count"
```

# 複数DBを扱う

> アプリケーション規模が大きくなると応答速度の低下が問題。  
この時ボトルネックとなりやすいのはDBアクセスに関する部分。
  
  
# 秘密情報を管理
:memo: 未読：[12factor app](https://12factor.net/ja/)の考え方: 秘密の文字列は環境変数経由で利用する | [[パーフェクトRubyonRails](https://amzn.to/43QRVDL)](https://amzn.to/45aUlzo):p153

:heavy_check_mark: ステージング環境用に credentials を分ける | [[パーフェクトRubyonRails](https://amzn.to/43QRVDL)](https://amzn.to/45aUlzo):p157

```sh
bin/rails credentials:edit --environment staging # 作成
bin/rails credentials:show --environment staging # 確認
```
- config/credentials/staging.key : 作成される
- config/credentials/staging.yml.enc : 作成される
- .gitignore へ staging.key が自動的に登録される（staging.keyをコミットしないように）


# 参考書籍
<a target="_blank" href="https://amzn.to/43QRVDL" style="
    border: 2px solid;
    width: 220px;
    display: block;
    overflow: hidden;
    border-radius: 10px;
    border-color: #c1cbc1;
    text-align: center;
    padding: 9px 0;
    animation: flashShadow 2.5s infinite alternate;
    ">
  <img src="https://m.media-amazon.com/images/W/MEDIAX_1215821-T1/images/I/81yskupyNhL._AC_AIweblab1006854,T3_SF700,700_PQ60_.jpg?aicid=detailPage-mediaBlock" alt="パーフェクトRubyonRails" style="border:none;width:110px;" /><script>window.dataLayer = window.dataLayer || [];function gtag() { dataLayer.push(arguments); }gtag('js', new Date());gtag('config', 'G-CJF3K99R7P');</script><br />
  パーフェクトRubyonRails | Amazon
</a>


 ## link

> https://rubyonrails.org/docs