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

Goパッケージ
------------

* `golang.org/x/oauth2/jws <https://godoc.org/golang.org/x/oauth2/jws>`_
* `golang.org/x/oauth2/jwt <https://godoc.org/golang.org/x/oauth2/jwt>`_

他
==

OAuth2には

Authorization Code
	よくあるリダイレクトを使うやつ

	3-legged OAuth 2.0

Implicit Grant
	よくわからない

Resource Owner Password Credentials
	よくわからない

Client Credentials
	クライアントIDとシークレットを使うやつ

	2-legged OAuth 2.0

* `色々な OAuth のフローと doorkeeper gem での実装 <https://qiita.com/tyamagu2/items/5aafff7f6ae0a9ec94aa>`_

Goパッケージ
------------

* `golang.org/x/oauth2 <https://godoc.org/golang.org/x/oauth2>`_
* `golang.org/x/oauth2/clientcredentials <https://godoc.org/golang.org/x/oauth2/clientcredentials>`_
