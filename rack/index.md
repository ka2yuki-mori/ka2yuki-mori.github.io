# Rack memo

## Ruckとは：
  - インターフェイスを共通化した仕様で実装となっているライブラリ：WebAppサーバとWebAppフレームワーク間のインターフェイス。：パーフェクトRubyonRails: p129
  - PythonのWGSIという規格を元に提案されたものが Ruck。｜パーフェクトRubyonRails: p130
  - https://github.com/rack/rack

Rcukインターフェイス：
> Ruck に対応する アプリケーションやフレームワークは Ruck規約 に則ったインターフェイスを定義する必要がある。

- callメソッドを定義する
- callメソッドは慣習的に env あるいは enviroment と命名する引数を１つ受け取る
- callメソッドは次の値を配列型で戻り値として返す必要がある
  - HTTPステータスコードを表す数値オブジェクト
  - HTTPヘッダーを表すハッシュオブジェクト
  - レスポンスボディとなる文字列を含んだ配列風オブジェクト

```rb
def call(env)
  [status, headres, body]
end
```

> **どのような**長大なアプリケーションでも..**最終的**に収束する..　Ruck対応のアプリケーション。パーフェクトRubyonRails:p131

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

ソースコード：https://github.com/ka2yuki-mori/rack_app