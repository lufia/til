Nginxの設定関連
===============

* `公式ドキュメント <http://nginx.org/en/docs/http/ngx_http_core_module.html>`_

クライアントから送ってくるリクエストの ``Content-Length`` は、
``client_max_body_size`` で上限が決まっている。
デフォルトは1MBになっているため、ファイルをアップロードするようなサービスへ
nginxを通してアクセスする場合はおそらくサイズを大きくする必要がある。

``client_max_body_size`` は以下のコンテキストで設定可能。

* http
* server
* location
