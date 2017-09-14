Nginxの設定関連
===============

リクエストサイズの制限
----------------------

* `公式ドキュメント <http://nginx.org/en/docs/http/ngx_http_core_module.html>`_

クライアントから送ってくるリクエストの ``Content-Length`` は、
``client_max_body_size`` で上限が決まっている。
デフォルトは1MBになっているため、ファイルをアップロードするようなサービスへ
nginxを通してアクセスする場合はおそらくサイズを大きくする必要がある。

``client_max_body_size`` は以下のコンテキストで設定可能。

* http
* server
* location

リバースプロキシでは/をURLエンコードしない
------------------------------------------

* `Alphabetical index of variables <http://nginx.org/en/docs/varindex.html>`_

Nginxはクライアントから届いたURLをデコードして、
``proxy_pass`` で設定したURLへ転送するときにエンコードし直す。
このとき、リクエストURLの途中に含まれる **%2F** はエンコードしない。

例えば以下の設定を行っていた場合::

	location / {
		proxy_pass http://example.com/;
	}

**http://localhost/wiki/2017%2F11%2F20** というリクエストを受け取ると、
Nginxは **http://example.com/wiki/2017/11/20** へプロキシする。
ほとんどは問題ないと思うが、一部のサービスではうまく扱えないかもしれない。

*$request_uri* を使うと、リクエストされたURLをそのまま使える::

	location / {
		proxy_pass http://example.com$request_uri;
	}
