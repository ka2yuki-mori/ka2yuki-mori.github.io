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