# イベント告知アプリケーション
> Ubuntu 22.04.5 LTS on WSL2  
> Ruby 3.3.8   
> Rails 7.1.5.1

## 1: 仕様
- 1-1: 実装の仕様
- 1-2: ルーティング仕様
- 1-3: ER図

## 2: アプリケーション作成と下準備
- 2-1: rails new アプリケーション作成
- 2-2: Haml導入
- 2-3: トップページ表示
- 2-4: Bootstrap導入

## 3: GiHubログイン機能
- 3-1: OAuthとは
- 3-2: GitHubアプリの登録
- 3-3: GitHubアカウントでログイン機能
- 3-4: ユーザのモデル作成
- 3-5: ログイン処理
- 3-6: ログアウト処理

## 4: イベント登録機能
- 4-1: タイムゾーン設定
- 4-2: モデル作成（イベント用）
- 4-3: ログイン状態管理の処理
- 4-4: フォーム作成：イベント登録用
- 4-5: バリデーションエラー時の設定
- 4-6: i18n 設定  

## 5: イベント閲覧 機能
- 5-1: イベント詳細ページ
- 5-2: イベント一覧ページ

## 6: イベント編集・削除 機能
- 6-1: イベント編集機能
- 6-2: イベント削除機能

## 7: 参加・キャンセル機能
- 7-1: イベント参加機能
- column: どんな時にヘルパーを使うか
- column: N+1問題をどのように察知するか

## 8: 退会機能
- 8-1: 退会用のコントローラ・ビュー・ルーティング
- column: コントローラの粒度と名前付け
- 8-2: 退会処理

## 9: おわりに




## Bootstrap導入
importmapを使ってBootstrap導入  
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
**UJS**ライブラリをテストするリンク
```haml
= link_to "UJS確認（確認ダイアログ）", "#", data: { confirm: "UJSが動いていますか？" }
-# ダイアログが出ない場合、Rails UJS（Unobtrusive JavaScript）が正しく読み込まれていない可能性
```

これでdata-methodやdata-confirmが動作。（postもできましたgetになってた）

まとめ  
- Importmapで rails-ujs を import は非推奨。エラーになる
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
<a target="_blank" href="https://amzn.to/44foMlT" style="
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
  <img 
    src="https://m.media-amazon.com/images/I/71LvP9jh7SL._SY342_.jpg" 
    alt="OAuth徹底入門 セキュアな認可システムを適用するための原則と実践" 
    style="border:none;width:110px;" /><script>window.dataLayer = window.dataLayer || [];function gtag() { dataLayer.push(arguments); }gtag('js', new Date());gtag('config', 'G-CJF3K99R7P');</script><br />
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

暗号化のためコマンドから複合化してYAMLファイルを開きClient ID/Client Secretsを保存。暗号化されて保存される(GitHubに登録したアプリのClient IDとClient secrets)
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
- コントローラ：セッション削除
- ログアウトリンク作成
- コントローラ：ログアウト機能作成
- ルーティング
  

6-4 イベント登録機能：
- ヘッダーに「イベントを作る」リンク  を作成
- ログイン状態 で「イベントを作る」をクリック → イベント登録フォームに遷移
- 未ログイン状態 で「イベントを作る」をクリック → トップページにリダイレクト
- イベント登録フォームで 間違った入力をする → エラーメッセージが表示
- イベント登録フォームで 正しい入力をする → イベント登録される







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