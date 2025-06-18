# Rack memo

## Ruckとは：
  - インターフェイスを共通化した仕様で実装となっている**ライブラリ**：WebAppサーバとWebAppフレームワーク間の**インターフェイス**。：[[パーフェクトRubyonRails](https://amzn.to/43QRVDL)](https://amzn.to/45aUlzo): p129
  - PythonのWGSIという規格を元に提案されたものが Ruck。｜[[パーフェクトRubyonRails](https://amzn.to/43QRVDL)](https://amzn.to/45aUlzo): p130
  - https://github.com/rack/rack

**Rcukインターフェイス**：
> Ruck に対応するアプリケーションやフレームワーク は Ruck規約 に則ったインターフェイス を定義する必要がある。

- <u>callメソッドを定義</u>する
- callメソッドは慣習的に env あるいは enviroment と命名する<u>引数を１つ</u>受け取る
- callメソッドは次の値を<u>配列型で戻り値</u>として返す必要がある
  - HTTPステータスコードを表す数値オブジェクト
  - HTTPヘッダーを表すハッシュオブジェクト
  - レスポンスボディとなる文字列を含んだ配列風オブジェクト

```rb
def call(env)
  [status, headres, body]
end
```

> **どのような**長大なアプリケーションでも、**最終的**に**これらの値を返すことができれば** Ruckの規約に則ったアプリケーションとして成立します。  
**Ruck対応のフレームワークは** 最終的にこのシンプルな形のインターフェイスに**収束**する..　[[パーフェクトRubyonRails](https://amzn.to/43QRVDL)](https://amzn.to/45aUlzo):p131

## かんたんなRuckアプリケーション
```sh
$ gem install rack # global install
$ mkdir rack_app
$ touch rack_app/app.rb
```
```app.rb
class App
  def call(env)
    pp env # check
    status = 200
    headers = {"content-type" => "text/plain"}
    body = ["sample"]
    [status, headers, body]
  end
end
```
```config.ru
require rack
require_relative "app"
run App.new
```
```sh
$ rackup # rack server start
Puma starting in single mode...
..
* Listening on http://127.0.0.1:9292
```

## Rackミドルウェアを使ってみる
```config.ru
require rack
require_relative "app"
use Rack::Runtime # + added

run App.new
```
ResponseHeadersに 処理時間 が追記される。

> Rack の**便利なミドルウェア**を集めた別リポジトリ：
https://github.com/rack/rack-contrib


## Rackミドルウェアを作ってみる
- inittialize メソッドに引数を１つ取る
- Rackアプリケーションと同じインターフェイスのcallメソッドを用意する

```SimpleMiddleware.rb
class SimpleMiddleware
  def initialize(app)
    puts "*" * 50
    puts "* #{self.class} initialize(app = #{app.class})"
    puts "*" * 50
    @app = app
  end

  def call(env)
    status, headers, body = @app.call(env)
    puts "*" * 50
    puts "* #{self.class} call(body = #{body})"
    puts "*" * 50
    return [status, headers, body]
  end
end
```
```config.ru
require "rack"
require_relative "app"
require_relative "simple_middleware.rb" # + add

use Rack::Runtime # Response Headers: be added time to catch from request to response.
use SimpleMiddleware # + add

run App.new
```
> リクエストデータに基づく処理を行い..中心のRackアプリに到達するとcallメソッドの戻り値のレスポンスデータを返しながら、各層でレスポンスデータに対する処理を行い　クライアントに返すレスポンスデータを決定します。

Rackのミドルウェアとアプリケーションの概念図：
![](https://docs.pylonsproject.org/projects/pylons-webframework/en/v1.0.1rc1/_images/pylons_as_onion.png)  
[引用:Pylons:WSGIミドルウェア](https://docs.pylonsproject.org/projects/pylons-webframework/en/v1.0.1rc1/concepts.html#wsgi-middleware)


## RailsとRack の関係
> `config.ru`.. これこそ Rackアプリケーション の証左

config.ru内にも `This file is used by Rack-based severs to start the application.`

> Rails では いくつかの Rackミドルウェアも利用しています。  
`$ rails middleware`
```sh
$ rails middleware
use ActionDispatch::HostAuthorization
use Rack::Sendfile       # <- Rack
use ActionDispatch::Static
use ActionDispatch::Executor
use ActionDispatch::ServerTiming
use Rack::Runtime        # <- Rack
use Rack::MethodOverride # <- Rack
use ActionDispatch::RequestId
use ActionDispatch::RemoteIp
use Rails::Rack::Logger  # <- Rack
use ActionDispatch::ShowExceptions
use WebConsole::Middleware
use ActionDispatch::DebugExceptions
use ActionDispatch::ActionableExceptions
use ActionDispatch::Reloader
use ActionDispatch::Callbacks
use ActiveRecord::Migration::CheckPending
use ActionDispatch::Cookies
use ActionDispatch::Session::CookieStore
use ActionDispatch::Flash
use ActionDispatch::ContentSecurityPolicy::Middleware
use ActionDispatch::PermissionsPolicy::Middleware
use Rack::Head           # <- Rack
use Rack::ConditionalGet # <- Rack
use Rack::ETag           # <- Rack
use Rack::TempfileReaper # <- Rack
run RailsRackSample::Application.routes
```
> 上から順にミドルウェアが積まれた順番 | [[パーフェクトRubyonRails](https://amzn.to/43QRVDL)](https://amzn.to/45aUlzo):p139


## 自作 Rackミドルウェア を Rails に追加
```sh
mkdir lib/middlewares
touch lib/middlewares/upcase_middleware.rb
```
```upcase_middleware.rb
class UpcaseMiddleware
  def initialize(app)
    @app = app
  end

  def call(env)
    status, headers, body = @app.call(env)
    body.each {|s| s.gsub!(/ruby/i, "RUBY")}
    [status, headers, body]
  end
end
```
- development環境のみ： `config/environments/developmens.rb`  
- 常に使用する：`config/application.rb`
> configオブジェクト を**通じて** Rackミドルウェア管理

```development.rb
require "middlewares/upcase_middleware"

Rails.application.configure do
  # 通常のミドルウェア追加
  # config.middleware.use UpcaseMiddleware
  # 特定ミドルウェアの次に追加する指定
  config.middleware.insert_after Rack::ETag, UpcaseMiddleware
```
```sh
$ rails middleware # ミドルウェアの確認
...
use Rack::ETag
use UpcaseMiddleware # <-
use Rack::TempfileReaper
run RailsRackSample::Application.routes

$ rails s # ruby が RUBY に変換されてるか確認
```

more info: 
- https://guides.rubyonrails.org/rails_on_rack.html#introduction-to-rack


