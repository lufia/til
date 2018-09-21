======
OAuth2
======

JSON Web Token
==============

署名のアルゴリズム
------------------

RS256, RS384, RS512
	RSA using SHA-xx hash algorithm

	非対称アルゴリズムで公開鍵・秘密鍵ペアを使う

HS256, HS384, HS512
	HMAC using SHA-xx hash algorithm

	対称アルゴリズムで共有鍵を使う

ES256, ES384, ES512
	ECDSA using P-xx curve and SHA-xx hash algorithm

	非対称アルゴリズムで公開鍵・秘密鍵ペアを使う

PS256, PS384, PS512
	RSASSA-PSS using SHA-xx and MFG1 with SHA-xx hash algorithm

	非対称アルゴリズムで公開鍵・秘密鍵ペアを使う

など色々ある。

* `RS256 と HS256 ってなにが違うの <https://qiita.com/satton_maroyaka/items/e68afe3de6267cebcfea>`_
* `JSON Web Algorithm <https://tools.ietf.org/html/rfc7518>`_

JWT, JWS, JWE
-------------

JWT (RFC 7519)
	JSON Web Token

	JSON形式でクレームを表すための仕様

JWS (RFC 7515)
	JSON Web Signature

	JWTを使って署名を行う仕様

JWE (RFC 7516)
	JSON Web Encryption

	JWTを使って暗号化を行う仕様

JWK (RFC 7517)
	JSON Web Key

	JWTを使って暗号化鍵を表現する仕様？

JWA (RFC 7518)
	JSON Web Algorithm

	JWS, JWE, JWKで使われるアルゴリズム

ログイン処理でよく使われるものは、おそらくJWSだと思う。

* `どうして JWT をセッションに使っちゃうわけ？ <https://co3k.org/blog/why-do-you-use-jwt-for-session>`_

JWT関連Goパッケージ
-------------------

* `golang.org/x/oauth2/jws <https://godoc.org/golang.org/x/oauth2/jws>`_
* `golang.org/x/oauth2/jwt <https://godoc.org/golang.org/x/oauth2/jwt>`_

利用用途
========

用語
-----

クライアント
	リソースサーバや認可サーバへリクエストを行うアプリケーション

リソースサーバ
	保護リソースを所有するサーバ

認可サーバ
	認可コードや各種トークンを発行するサーバ

	例: Google, Facebook

認可コード
	アクセストークンとリフレッシュトークン取得で使う文字列

アクセストークン
	リソースサーバへリクエストするためのトークン

リフレッシュトークン
	アクセストークン更新の際に使われるトークン

認可エンドポイント
	認可を行うためのエンドポイント

	だいたい認可コードの取得に使う

	例: */authorization*, */oauth*

トークンエンドポイント
	トークン発行のためのエンドポイント

	例: */token*, */access_token*

認可フロー
----------

OAuth2には

Authorization Code Grant
	よくあるリダイレクトを使うやつ

	3-legged OAuth 2.0

Implicit Grant
	Authorization Codeからアクセストークン取得を除いたもの

	認可コードの代わりにアクセストークンが直接渡される

Resource Owner Password Credentials Grant
	普通のIDパスワード認証

	Basic認証やLDAPからの移行などで使う想定

	基本的に使うべきではない

Client Credentials Grant
	クライアントIDとシークレットを使うやつ

	ユーザの同意を必要としないリソースに使う想定

	2-legged OAuth 2.0

* `RFCとなった「OAuth 2.0」 <http://www.atmarkit.co.jp/ait/articles/1209/10/news105.html>`_
* `色々な OAuth のフローと doorkeeper gem での実装 <https://qiita.com/tyamagu2/items/5aafff7f6ae0a9ec94aa>`_
* `Why the Resource Owner Password Credentials Grant Type is not Authentication nor Suitable for Modern Applications <https://www.scottbrady91.com/OAuth/Why-the-Resource-Owner-Password-Credentials-Grant-Type-is-not-Authentication-nor-Suitable-for-Modern-Applications>`_

クライアントタイプ
------------------

Confidential Client
	クライアントシークレットなどを秘密にできるクライアント

	サーバで動作するWebアプリなど

	Authorization Code Grantでアクセストークンを取得するべき

Public Client
	クライアントシークレットを秘密にできないクライアント

	モバイルアプリなど

	Authorization Code Grant+PKCEでアクセストークンを取得するべき

以前は、Public Clientはクライアントシークレット漏洩の懸念があるため、
Implicit Grantを使うように書かれていたが、なりすましの危険性があった。
2018年現在、モバイルアプリでもAuthorization Code Grantを推奨するメモがあった。

* `BCP 212 - Oauth 2.0 for Native Apps <https://tools.ietf.org/html/bcp212>`_

このメモでは、

* 外部ブラウザを使ってAuthorization Code Grantを行うこと
* ブラウザとアプリの連携はカスタムURLスキーマなどでリダイレクトすること
* カスタムURLスキーマ横取り防止のためPKCEを使うこと

などが書かれている。

PKCE
-----

同一のカスタムURLスキーマを異なるアプリが受信可能な場合、
どちらか片方がリダイレクトを受信してしまう。
このため、悪意のあるアプリがたまたまインストールされてしまうと、
トークンを横取りされてしまうことになって困る。

このような漏洩を防ぐために、認可サーバとクライアントで協力して、
*code_verifier* と *code_challenge* を使ってトークン取得時にも検証する仕様。

* `PKCEで防げる「認可コード横取り攻撃」とはどのような攻撃か <https://qiita.com/SAM-l/items/9574d1e237228c718cd6>`_

認可フローGoパッケージ
-----------------------

* `golang.org/x/oauth2 <https://godoc.org/golang.org/x/oauth2>`_
* `golang.org/x/oauth2/clientcredentials <https://godoc.org/golang.org/x/oauth2/clientcredentials>`_

他
====

OpenSSLコマンドメモ
-------------------

.. code-block:: bash

PKCS8でECDSA P-256秘密鍵を生成::

	openssl ecparam -genkey -name prime256v1 -noout |
	openssl pkcs8 -topk8 -nocrypt -out key.p8

.. code-block:: bash

秘密鍵から公開鍵を生成::

	openssl ec -in key.p8 -pubout
