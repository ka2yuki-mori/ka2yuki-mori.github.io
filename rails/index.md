# Rails

## importmapを使ってBootstrap導入
Importmapで Bootstrap / jQuery / Popper.js 導入方法

1. **config/importmap.rb**: **パッケージ追加**

```rb
pin "bootstrap", to: "https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.bundle.min.js"
pin "jquery",    to: "https://cdn.jsdelivr.net/npm/jquery@3.7.1/dist/jquery.min.js"
pin "popper",    to: "https://cdn.jsdelivr.net/npm/@popperjs/core@2.11.8/dist/umd/popper.min.js"
```

**`config/importmap.rb`** で **pin** できるのは JavaScriptファイル だけ。  
**CSS**（Bootstrapのスタイルシート）は Importmap では管理できません。  
そのため、Bootstrap の CSS は <link> タグで HTML（例: application.html.erb の <head> 内）に記述する必要があります。

2. app/javascript/application.js でインポート

```js
import "bootstrap"
import "jquery"
import "popper"
```

3. BootstrapのCSS は CDN で読み込む（例）
```js
<!-- app/views/layouts/application.html.erb の <head> 内など -->
<link href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/css/bootstrap.min.css" rel="stylesheet">
```

まとめ
- **Importmap**で **pin**できるのは **JavaScriptだけ**
- **CSS**（Bootstrapのスタイル）は &lt;link&gt;タグ 読み込adfasdfd

---
<a target="_blank" href="https://amzn.to/43QRVDL" style="
    border: 2px solid;
    width: 220px;
    display: block;
    overflow: hidden;
    border-radius: 10px;
    border-color: #90e790;
    text-align: center;
    padding: 9px 0;
    animation: flashShadow 2.5s infinite alternate;
    ">
  <img src="https://m.media-amazon.com/images/W/MEDIAX_1215821-T1/images/I/81yskupyNhL._AC_AIweblab1006854,T3_SF700,700_PQ60_.jpg?aicid=detailPage-mediaBlock" alt="パーフェクトRubyonRails" style="border:none;width:110px;" /><script>window.dataLayer = window.dataLayer || [];function gtag() { dataLayer.push(arguments); }gtag('js', new Date());gtag('config', 'G-CJF3K99R7P');</script><br />
  パーフェクトRubyonRails | Amazon
</a>
---


# GitHubログイン | OAuth
## OAuthとは
> **プロトコル**: 安全にアクセス権限を提供するための

- **OAuthなし**：GithubのユーザIDとパスワード :arrow_right: アプリA に預ける必要がある
  - パスワード漏洩
  - アプリA は ユーザと同じ権限を持つことになる。（Githubにログイン出来てしまう）
  - Githubから退会させることも
- OAuth**あり**: パスワード :arrow_right: アプリA に渡さなくてよくなる
  - Googleログイン や Facebookログインなど
---
<a target="_blank" 
  href="https://www.amazon.co.jp/OAuth%E5%BE%B9%E5%BA%95%E5%85%A5%E9%96%80-%E3%82%BB%E3%82%AD%E3%83%A5%E3%82%A2%E3%81%AA%E8%AA%8D%E5%8F%AF%E3%82%B7%E3%82%B9%E3%83%86%E3%83%A0%E3%82%92%E9%81%A9%E7%94%A8%E3%81%99%E3%82%8B%E3%81%9F%E3%82%81%E3%81%AE%E5%8E%9F%E5%89%87%E3%81%A8%E5%AE%9F%E8%B7%B5-Justin-Richer/dp/4798159298?&linkCode=sl1&tag=ka2yuki-22&linkId=21dd5bb0da23988e6305cbe032797e5e&language=ja_JP&ref_=as_li_ss_tl" style="
    border: 2px solid;
    width: 220px;
    display: block;
    overflow: hidden;
    border-radius: 10px;
    border-color: #90e790;
    text-align: center;
    padding: 9px 5px;
    animation: flashShadow 2.5s infinite alternate;
    ">
  <img src="https://m.media-amazon.com/images/I/71LvP9jh7SL._SY342_.jpg" 
    alt="OAuth徹底入門 セキュアな認可システムを適用するための原則と実践" 
    style="border:none;width:110px;" />
  <script>window.dataLayer = window.dataLayer || [];function gtag() { dataLayer.push(arguments); }gtag('js', new Date());gtag('config', 'G-CJF3K99R7P');</script><br />
  OAuth徹底入門 セキュアな認可システムを適用するための原則と実践 | amazon.co.jp
</a>
---


## GitHub登録
Settings > Developer settings > OAuth Apps

| 項目 | 内容 | 意味 |
|--------|--------|--------|
| Application name | アプリの名前 | アプリケーション名 |
| Homepage URL | `http://localhost:3000/` | アプリのURL |
| Appllication description | アプリの概要 | アプリの概要 |
| Authorization callback URL | `http://localhost:3000/auth/github/callback | OAuth認証**後** に遷移するURL |


## ログイン機能の作成

機能一覧: 
- 「GitHubでログイン」というリンク追加 on ヘッダー
- Github認証画面に遷移 when Click
- Github認証完了 ⇒ アプリにログイン
-「GitHubでログイン」が→「GitHubでログアウト」on ヘッダ
- ログアウトできる

使用gem: 
- omniauth 
- omniauth-github
- omniauth-rails_csrf_protection

> OmniAuth:  
ユーザ認証機能の**ベースgem**。  
「どのような形式で認証するか」は OmniAuthとは別に実装。OmniOAuthと組み合わせることで様々な(google/facebookログインなど)認証形式を提供できる。  

> OmniAuth Github:  
GithubのOAuth認証を使用できる

注意点：
- CSRF（クロスサイトリクエストフォージェリ）の脆弱性対策。
  - ソーシャルアカウント追加が実行の可能性 → アカウント乗っ取りの可能性。
- 「GitHubログイン」**だけ**の場合この脆弱性の悪用はできない
- 将来的に複数のサービスプロバイダー対応を備えてあらかじめ対策。:heavy_check_mark:  
  - :heavy_check_mark:gemが存在する：この脆弱性を対策するライブラリ(gem)

[パーフェクトRubyonRails](https://amzn.to/45aUlzo):p293

### Gemインストール
Gemfileに追加
```Gemfile
gem 'omniauth'
gem 'omniauth-github'
gem 'omniauth-rails_csrf_protection'
```
追加したgemをインストール
```sh
# ターミナル
>$ bundle install
```

ファイル作成（GitHubに登録したアプリのClient ID/Client secretsを取り扱う：`config/initializers/omniauth.rb`
```rb
Rails.application.config.middleware.use OmniAuth::Builder do
  provider :github,
            Rails.application.credentials.github[:client_id],
            Rails.application.credentials.github[:client_secret]
end
```
> OmniAuth は Rackミドルウェア として動作：  
「/auth/:provider」URLにマッチするアクセスを受け取ると認証処理を開始。  
:provider として渡された文字列。を どのサービスプロバイダーで認証したらよいか判別 しています。[パーフェクトRubyonRails](https://amzn.to/45aUlzo):p296

暗号化のためコマンドから複合化してYAMLファイルを開きClient ID/Client secretsを保存。暗号化されて保存される(GitHubに登録したアプリのClient IDとClient secrets)
```bash
bin/rails credentials:edit
```
```yml
secret_key_base: 英数字が羅列されている
github:
  client_id: "クライアントID"
  client_secret: "クライアント＿シークレット"
```
- [omniauth](https://github.com/omniauth/omniauth)
- [omniauth-github](https://github.com/omniauth/omniauth-github)
- [omniauth-rails_csrf_protection](https://github.com/cookpad/omniauth-rails_csrf_protection)

### Userモデル作成
```sh
bin/rails g model user provider uid name image_url
```

| カラム名 | 意味 |
|--------|--------|
| provider | OmniAuthの認証で使用するプロバイダ名（例："github" |
| uid | provider毎に与えられるユーザ識別用の文字列 |
| name | GitHubユーザ名 |
| image_url | Githubアイコン画像のURL|

仕様：  
NULLになるケースがないカラムの存在  
想定外のデータ不整合を防ぐ
- NOT NULL制約
- ユニークインデックス  
```rb
# db/migrate/20250618112709_create_user.rb
class CreateUsers < ActiveRecord::Migration[7.1]
  def change
    create_table :users do |t|
      t.string :provider,  null: false 
      t.string :uid,       null: false
      t.string :name,      null: false
      t.string :image_url, null: false
      t.timestamps
    end
    # null: falseの部分を追加しました
    
    add_index :users, %i[provider uid], unique: true
    # add_index の行を追加しました
  end
end
```
マイグレファイルの定義をDataBaseに反映
```sh
bin/rails db:migrate
```

### ログイン処理を作成
```rb
# 略
%body
  %header.navbar.navbar-expand-sm.navbar-light.bg-light
    .container
      = link_to('AwesomeEvents', root_path, class: 'navbar-brand')
      %ul.navbar-nav
        %li.nav-ite
          = link_to "GitHubでログイン", "/auth/github", class: 'nav-link', method: :post
          # この部分
  .container
    = yield
```
- CSRF対策のため：リンクは postメソッド

ログインをクリック → 認証ボタンをクリック → エラー(認証後の処理を書いていないため)
<img src="/assets/img/auth-err.png" />

### 認証後の処理：



---
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
    border-color: #90e790;
    text-align: center;
    padding: 9px 0;
    animation: flashShadow 2.5s infinite alternate;
    ">
  <img src="https://m.media-amazon.com/images/W/MEDIAX_1215821-T1/images/I/81yskupyNhL._AC_AIweblab1006854,T3_SF700,700_PQ60_.jpg?aicid=detailPage-mediaBlock" alt="パーフェクトRubyonRails" style="border:none;width:110px;" /><script>window.dataLayer = window.dataLayer || [];function gtag() { dataLayer.push(arguments); }gtag('js', new Date());gtag('config', 'G-CJF3K99R7P');</script><br />
  パーフェクトRubyonRails | Amazon
</a>


 ## link

> https://rubyonrails.org/docs
