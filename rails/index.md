# Rails

## tips
- `config/initializers/*` 配下のファイルは、Railsアプリ起動時に一度だけ読み込まれ実行されます。
- そのため、initializersディレクトリ内を変更した場合`rails s`（サーバー）を再起動しないと変更が反映されない。

**まとめ：**  
initializersディレクトリを編集したら、`rails s` 再起動

### Railsv7のWebpackerは..
- Rails 7 では Webpackerはデフォルトでは有効になっていません。
- Rails 7 のデフォルトは Importmap です。
- Webpackerやesbuild、Viteなどは自分で追加しない限り使われません。

```html
<a class="nav-link" rel="nofollow" data-method="post" href="/auth/github">GitHubでログイン</a>
```

## post指定したのに get でリクエストされる

- data-method="post" は Rails の UJS（Unobtrusive JavaScript） 機能によって、JavaScriptが有効な場合のみ動作します。
- JavaScriptが無効な場合や、UJSが正しく読み込まれていない場合は、通常のリンク（GETリクエスト）として動作します。
- UJS（Unobtrusive JavaScript） は、Railsが提供するJavaScriptライブラリで、data-method などの属性を解釈してPOSTやDELETEリクエストを送る仕組みです。  
  JavaScriptが動いていても、UJSが正しく読み込まれていなければ、data-method は機能しません。

まとめ  
data-method="post" はJavaScript（rails-ujs）が動作して初めて有効。
それが無効だと、リンクはGETリクエストになる。



# リクエストが来たときの流れ（RackとRailsの関係）
- Rackが最初に受け取り、次にRailsに渡す という流れ
- Railsは「Rackアプリケーション」として動作している
- Rackは「RubyのWebアプリの共通インターフェース」
  

RailsはRackから受け取ったリクエストを解析し、ルーティングしてコントローラなどで処理する  

イメージ：  
[Webサーバー] → [Rack] → [Rails] → [コントローラ/アクション]  
  
まとめ  
- 最初に処理するのは「Rack」
- その後、「Rails」が本格的なアプリ処理を行う

ex)  
- **サーバー（例: Puma, Unicorn）**  
  クライアントからのリクエストを受け取ります。
- **Rack**  
  サーバーとRailsアプリの間でリクエスト情報を「環境変数（env）」というHashにまとめて渡します。
- **Rails**  
  Rackから受け取ったenv Hashをもとに、`ActionDispatch::Request`（= `request`）インスタンスを生成します。  
  この `request.env` が、Rackから受け取った元の環境変数（Hash）です。

つまり、
- `request` はRailsが生成したリクエストオブジェクト
- `request.env` はRackから受け取ったリクエスト情報（Hash）


1. **request.env とは何か？**
    - `request.env` は、Railsコントローラ内で使える「Rack環境変数（Rack Environment）」のハッシュ。
    - これは、HTTPリクエストに関するさまざまな情報（ヘッダー、サーバー情報、ミドルウェアが追加した値など）を格納。
    - `request` は Railsコントローラのインスタンスメソッドで、現在のリクエストオブジェクト（`ActionDispatch::Request`）を返す。
    - `request.env` は、そのリクエストに紐づくRack環境変数（Hash）。

2. **Rails 定数や params との違い**
    - Rails 定数は、Railsアプリ全体の設定や情報を持つ「グローバル」な定数。
        - 例: `Rails.env`, `Rails.root` など
    - `params` は、リクエストで送信されたパラメータ（GET/POSTデータなど）を格納したハッシュです。
    - `request.env` は、リクエストごとに生成される「リクエストスコープ」のハッシュで、HTTPヘッダーやミドルウェアが追加した情報（例: OmniAuthの認証情報）などが入ります。

3. **どこからでも request.env できる？**
    - コントローラやビューで `request.env` を使うことはできます（ただしビューで使うことは稀）。
    - `rails c`（コンソール）で `request.env` は使えません。
        - なぜなら、コンソールはリクエストが存在しないためです。
    - モデルやlibなど、リクエストの文脈がない場所では使えません。
    - コントローラや一部のヘルパーでのみ利用可能です。

- `request.env` は**サーバー側の情報**なので、<u>ブラウザの開発者ツール（F12）では直接見られません</u>。
- Railsでエラーが発生したときに下に表示されるコンソール画面は、**RailsのWebコンソール（Web Console）**で `reauest.env`を確認できる。

### request.env 確認方法
1. コントローラで logger を使う
```ruby
def create
  logger.info request.env.inspect  # すべて出力（量が多いので注意）
  logger.info request.env["omniauth.auth"].inspect  # 必要な部分だけ
  # ...残りの処理
end
```

- サーバーを起動しているターミナル（またはlog/development.log）で内容を確認できます。
2. byebug や pry でブレークポイントを置く
Gem byebug や pry を使って、リクエスト時に一時停止し、コンソールで中身を確認できます。
```ruby
def create
  byebug
  # または
  # binding.pry
  # ...残りの処理
end
```
- ブラウザでそのアクションにアクセスすると、サーバー側でデバッガが起動し、request.env などを直接確認できます。

3. 必要な値だけビューに表示する（簡易的な方法）
一時的にビューやコントローラで値を表示することもできます。
```ruby
render plain: request.env["omniauth.auth"].inspect
```


request とは
リクエスト全体の情報を持つオブジェクト（ActionDispatch::Request）。
HTTPメソッド（GET/POSTなど）、ヘッダー、IPアドレス、パス、クッキー、env など、
リクエストに関するあらゆる情報を取得できる。
例：

```rb
request.method         # "GET" や "POST"
request.path           # "/users/1"
request.remote_ip      # アクセス元IP
request.env            # Rack環境変数（ハッシュ）
request.headers        # HTTPヘッダー
```

params とは
リクエストで送信されたパラメータ（値）だけをまとめたハッシュ。
URLのクエリパラメータ、フォーム送信値、ルーティングで抽出された値などが入る。
例：
```ruby
params[:id]            # /users/1 なら 1
params[:name]          # フォームで送信された name
params[:search]        # ?search=xxx の値
```

まとめ表    

|            | request                                 | params                       |
|------------|-----------------------------------------|------------------------------|
| **役割**   | リクエスト全体の情報                    | パラメータ（値）だけ         |
| **型**     | オブジェクト                            | ハッシュ                     |
| **例**     | method, path, env, headers など         | id, name, search など        |
| **用途**   | 詳細なリクエスト情報が必要な時           | 送信された値を使いたい時     |

> **イメージ**  
> request = 「封筒そのもの」  
> params = 「封筒の中に入っている申込書の内容（値）」  

paramsの内容も広い意味でrequestの中に含まれています。  
詳しく説明  
params は、request オブジェクトが持つ情報の一部（リクエストから抽出されたパラメータ）です。
ただし、request の中に params というメソッドやプロパティが直接あるわけではありません（Railsが便利にまとめてくれている）。

### 仕組み

params は、以下の値を合体させたものです。

- `request.query_parameters`（GETパラメータ）
- `request.request_parameters`（POSTパラメータ）
- `request.path_parameters`（ルーティングで抽出された値）

つまり、params の元データは request オブジェクトの中にあります。

```ruby
params[:id]
# 実際は
request.query_parameters[:id]
request.request_parameters[:id]
request.path_parameters[:id]
# などを合成したもの
```

> **まとめ**  
> params の内容は request から作られている。  
> ただし、`request.params` という形で直接アクセスすることは少ない（Railsがコントローラで params を用意してくれる）。

参考
- params は「リクエストの中の“値”だけをまとめたもの」
- request は「リクエスト全体の情報（値以外も含む）」


# railsとbin/railsコマンドの違い

- `rails`コマンド は システム全体にインストールされた Rails 実行ファイル を使います。
- `bin/rails`コマンド は そのプロジェクト専用の Bundler環境 で実行されます。

| コマンド | 実行環境 | 特徴 |
|--------|--------|--------|
| `rails` | グローバル環境 | システムにインストールされた Rails を使う。複数プロジェクトで gem のバージョンが違うと不整合が起きる可能性あり。 |
| `bin/rails` | プロジェクトローカル | `bundle install` で生成された `bin/rails` を使い、Gemfile に定義されたバージョンで Rails を実行。安全・確実。|  

 bin/rails は実質的に bundle exec rails と同じ意味を持ちます。




---

## importmapを使ってBootstrap導入
importmap の基本的な流れ  

1. CDNなどのURLを importmap で pin する  
importmap.rb で、必要なライブラリやモジュールをCDN（またはローカル）からpinします。
```rb
pin "lodash", to: "https://cdn.jsdelivr.net/npm/lodash-es@4.17.21/lodash.js"
pin "bootstrap", to: "https://cdn.jsdelivr.net/npm/bootstrap@5.3.3/dist/js/bootstrap.esm.min.js"
```

2. application.js で import する  
importmapでpinした名前を使って、ESM形式でimportします。
```js
import { debounce } from "lodash"
import * as bootstrap from "bootstrap"
```

3. 必要な関数やクラスだけ import して使う  
例えば、import { hoge, foo } from "mylib" のように、必要な部分だけimportできます。

まとめ  
- importmapでpinするのは基本的にCDNのURL（またはローカルのパス）
- application.jsなどでESM形式（ECMAScript Modules）でimportして使う
- 必要な関数やクラスだけをimportして利用できる





> 注意：追記  
> コンソールにエラー出てないか確認  
import "bootstrap" で Cannot use import statement outside a module というエラーが出る場合、  
Importmap環境ではESM（ECMAScript Modules）として解釈されていないことが原因。
```haml
%head
  = javascript_importmap_tags
```
- `javascript_include_tag, javascript_pack_tag` ではなく javascript_importmap_tags を使う
  
> 重要：


まとめ  
- Importmap環境**では** rails-ujs を直接利用することはサポートされていません（グローバルスコープの違いによりエラーになります）。
- Rails 7 + Importmap では、UJS の代わりに Turbo や Stimulus の利用が推奨されています。


```sh
rails-ujs.js:12 Uncaught TypeError: Cannot set properties of undefined (setting 'Rails')
```
このエラーは、rails-ujs がグローバルな window.Rails をセットしようとしているが、window が未定義（ESM環境でのimport時によく発生）なため。
 
原因  
- rails-ujs **は** 本来SprocketsやWebpackerなどの「古い」アセットパイプライン用で、ESM(importmap)環境ではうまく動作しません。
- Importmapで直接rails-ujsを読み込むと、グローバル変数の扱いでエラーになります。

解決策  
> 1. UJSをImportmapで使うのは非推奨    
Rails 7 + Importmapでは、UJSは公式にはサポートされていません。
（Turbo/Stimulusへの移行が推奨されています）
2. どうしてもUJSを使いたい場合  
CDNで直接scriptタグで読み込む  

application.html.haml の</head>直前などに以下を追加：  
```haml
%script{ src: "https://cdn.jsdelivr.net/npm/rails-ujs@5.2.4/lib/assets/compiled/rails-ujs.js" }
:javascript
  Rails.start();
```
これでdata-methodやdata-confirmが動作しました。（postできましたgetになってた)

まとめ  
- Importmapでrails-ujsをimportするとエラーになる
- CDNでscriptタグで直接読み込むことで一応動作します
- 公式にはTurbo/Stimulusへの移行が推奨されています
  - Hotwire (Turbo & Stimulus) 公式サイト: https://hotwired.dev/
  - Rails Guides: Working with JavaScript in Rails (Turbo & Stimulus): https://guides.rubyonrails.org/working_with_javascript_in_rails.html

**Rails 7以降は、従来のUJSではなくTurboとStimulusによるモダンなJavaScript開発が推奨されています。**


### Importmapで Bootstrap / jQuery / Popper.js 導入方法

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
- **CSS**（Bootstrapのスタイル）は &lt;link&gt;タグ 読み込み

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
  <img src="https://m.media-amazon.com/images/I/81yskupyNhL._SY342_.jpg" 
    alt="パーフェクトRubyonRails" style="border:none;width:110px;" /><script>window.dataLayer = window.dataLayer || [];function gtag() { dataLayer.push(arguments); }gtag('js', new Date());gtag('config', 'G-CJF3K99R7P');</script><br />
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
- method: :post はJavaScript/UJSが動作して初めてPOSTリクエストになります。JSが無効・読み込まれていない場合はGETになります。
- Rails UJSの読み込みと、javascript_importmap_tags の記述を再確認

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
