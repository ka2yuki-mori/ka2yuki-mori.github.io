curl と wget の主な違い。

## wget

- 主にファイルやWebページのダウンロードに特化
- 再帰的ダウンロードやミラーリングが得意
- シンプルなGETリクエストが中心
- スクリプトや自動化でよく使われる

```sh
wget http://localhost:3000/auth/github
```

## curl

- HTTPだけでなく、FTPやSFTPなど多くのプロトコルに対応
- GETだけでなく、POSTやPUTなど様々なHTTPメソッドをサポート
- APIのテストやデータ送信、ヘッダー操作などが得意
- レスポンスの内容を細かく制御・表示できる

```sh
curl -v http://localhost:3000/auth/github
curl -v -X POST http://localhost:3000/auth/github
```

## httpie（おすすめ）
より人間に読みやすい出力。インストールが必要
```sh
# install
sudo apt install httpie
# usage
http http://localhost:3000/auth/github
http POST http://localhost:3000/auth/github
```

## jq
jq は、JSONデータを扱うためのコマンドラインツールです。いわば「JSON専用の grep や sed のような存在」で、APIレスポンスやログなどのJSONを整形・抽出・加工するのにとても便利

- 整形表示（インデント付きで見やすく）
```sh
cat data.json | jq .
```
- 特定のキーだけ抽出
```sh
cat data.json | jq '.title'
```
- 配列の中の要素をループ処理

```sh
cat data.json | jq '.items[].name'
```
- ダブルクォートを外してプレーンテキストで出力

```sh
jq -r '.title' data.json
```

- macOS: brew install jq
- Ubuntu: sudo apt install jq
