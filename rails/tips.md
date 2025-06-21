# Rails

## tips

### デバッガー: debugger で使える主要なコマンド
```rb
def show
  debugger  # デバッグ用のブレークポイントを追加
```
- `q` または `quit` - デバッガーを終了してプログラムの実行も停止
- `exit` - デバッガーを終了してプログラムを続行
- `n` または `next` - 次の行に進む（メソッドの中には入らない）
- `s` または `step` - 次の行に進む（メソッドの中に入る）
- `c` または `continue` - デバッガーを終了してプログラムを続行
- `l` または `list` - 現在の周辺のコードを表示
- `p 変数名` - 変数の値を表示
- `pp 変数名` - 変数の値を整形して表示
  
    
### `config/initializers/*` 配下のファイル
- `config/initializers/*` 配下のファイルは、Railsアプリ起動時に一度だけ読み込まれ実行される。
- そのため、initializersディレクトリ内を変更した場合`rails s`（サーバー）を再起動しないと変更が反映されない。

**まとめ：**  
initializersディレクトリを編集したら、`rails s` 再起動

### Webpacker of Rails.v7 is
- Rails 7 では Webpackerはデフォルトでは有効になっていない。
- Rails 7 のデフォルトは Importma。
- Webpackerやesbuild、Viteなどは自分で追加しない限り使われません。


## post指定したのに get でリクエストされる
```html
<a class="nav-link" rel="nofollow" data-method="post" href="/auth/github">GitHubでログイン</a>
```
- **data-method="post"** は Rails の **UJS機能**（ライブラリ: Unobtrusive JavaScript）。
- JavaScriptが無効な場合や、UJSが正しく読み込まれていない場合は、通常のリンク（GETリクエスト）として動作します。
- UJS（Unobtrusive JavaScript） は、Railsが提供するJavaScriptライブラリで、data-method などの属性を解釈してPOSTやDELETEリクエストを送る仕組みです。  
  JavaScriptが動いていても、UJSが正しく読み込まれていなければ、data-method は機能しません。

まとめ  
data-method="post" はJavaScript（rails-ujs）が動作して初めて有効。
それが無効だと、リンクはGETリクエストになる。

---

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
  Rackから受け取ったenv Hashをもとに、`ActionDispatch::Request`（= `request`）のインスタンスを生成します。  
  この `request.env` が、Rackから受け取った元の環境変数（Hash）です。  
  ActionDispatch::Requestは、RailsでHTTPリクエストの詳細情報（ヘッダー、パラメータ、セッションなど）を扱うためのクラス。

つまり、
- `request` はコントローラ内で利用できるActionDispatch::RequestのインスタンスでRailsが生成したリクエストオブジェクト
- `request.env` はRackから受け取ったリクエスト情報（Hash）が代入されてる

`params`はリクエストパラメータ（GET/POSTデータなど）だけですが、`request`は以下のような多くの情報を持っています：
- params（リクエストパラメータ）
- headers（リクエストヘッダー）
- cookies
- session
- path, method, ip など

例：  

```rb
# コントローラ内
def show
  request.class # => ActionDispatch::Request
  request.params # => paramsと同じ内容
  request.headers["User-Agent"] # => ユーザーエージェント
  request.remote_ip # => クライアントIP
end
```

```bash
echo "hello"
```

```php
function getAdder(int $x): int 
{
    return 123;
}
```
```html
<p>This is a paragraph</p>
<a href="//docsify.js.org/">Docsify</a>
```



まとめ:  
- request は ActionDispatch::Requestインスタンス です。(params よりも多くのリクエスト情報を持つ)  
- params は ActionController::Parametersクラスのインスタンス  
  - 通常のHashのように使えますが、許可されたパラメータのみを扱うためのメソッド（permit, requireなど）が追加されています。
```rb
params.class # => ActionController::Parameters
params[:id]  # => パラメータの値を取得
params.permit(:name, :email) # => 許可したパラメータのみ取得
```

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
  <img src="https://m.media-amazon.com/images/I/81yskupyNhL._SY342_.jpg" 
    alt="パーフェクトRubyonRails" style="border:none;width:110px;" /><script>window.dataLayer = window.dataLayer || [];function gtag() { dataLayer.push(arguments); }gtag('js', new Date());gtag('config', 'G-CJF3K99R7P');</script><br />
  パーフェクトRubyonRails | Amazon
</a>
---


 ## link

> https://rubyonrails.org/docs
