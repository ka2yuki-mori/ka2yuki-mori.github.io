> WordPress には 2種類の REST API が存在していて、それぞれ仕様やエンドポイントが異なります：
> 

# 1. WordPress.org 系（コア REST API）
- エンドポイント例：https://example.com/wp-json/wp/v2/posts
- ドキュメント：WordPress REST API Handbook
- 特徴：
  - WordPress 本体に組み込まれている REST API
  - 自分でホスティングしている WordPress（WordPress.org）で使われる
  - カスタム投稿タイプや認証の拡張がしやすい

# 2. WordPress.com / Jetpack 系（Public API）
- エンドポイント例：https://public-api.wordpress.com/rest/v1.1/sites/{site}/posts/
- ドキュメント：Jetpack Developer API Reference
- 特徴：
  - WordPress.com 上のサイトや Jetpack 連携サイト向け
  - サイト情報や投稿一覧などを取得可能
  - 認証なしでも読み取り可能な範囲が広い  
  


|項目| WordPress.org REST API  | WordPress.com Public API |
|---|--------|--------|
|URL形式|  `/wp-json/wp/v2/`  |  `https://public-api.wordpress.com/rest/v1.1/`  |
|カスタマイズ性 |  高い（プラグインやテーマで拡張可能） |  低い（API仕様に従う） |
| 認証 |  Cookie認証、Application Passwords、OAuth等  |  OAuth2（認証なしでも一部取得可能） |
| 対象 |  自分で運用するWordPress（WordPress.org） |  WordPress.comサイト、Jetpack連携サイト  |
  

WordPress.com 上にある場合、Jetpack Public API の方。  
ただ、将来的に WordPress.org に移行したり、より細かい制御をしたい場合はコア REST API の方が自由度が高い。

> tips:  
> Public API も REST API の一種  
> 「WordPress本体のREST API」と「WordPress.com（Jetpack）のPublic API」では 設計思想や提供者、使える範囲が異なる ため、目的に応じて使い分ける必要があります。

## 🛠 そもそも REST API とは？
> REST（Representational State Transfer）に基づいた設計思想で作られた Web API のこと。  
> `GET /posts`, `POST /comments` など、**HTTPメソッド**＋**URL構造でリソース（データ）を操作**するのが特徴です。  
> WordPress の API もこの原則に則って作られているため、両方とも “REST API” に分類されます。

## ✅ Public API も REST API だが…
Public API は **WordPress.com の環境に特化した RESTful API**
- APIの形（構造）は REST に準拠 ✅
- WordPressコアの /wp-json/ とは別物 👀
- WordPress.com のデータに対してしか動作しない 

## 💡 たとえるなら…
- WordPress コア REST API → 「自分のキッチンで自由に料理できる」
- WordPress.com Public API → 「フードコートで決められたメニューから注文する」
どちらも「食事（REST API）」だけど、使い方や自由度がちょっと違う、というイメージ

