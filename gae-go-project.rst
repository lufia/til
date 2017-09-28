Google App Engine/Goのプロジェクト
==================================

GAE/Goのコーディング以外に関する情報。

1つのプロジェクトで複数のApp Engineを使う
-----------------------------------------

GCPは、以下の階層で構成されている。

1. 組織
2. プロジェクト
3. サービス
4. バージョン
5. インスタンス

App Engineはサービスに該当し、よく見る **app.yaml** は::

	application: app-name
	version: 1
	runtime: go
	
	handlers:
	  - url: /.*
	    script: _go_app

のような例なので、1つしか作れないように見えるが、
実際はこれは **service** が省略されて **default** になっているだけで、
自分で **service** キーを設定すれば複数のApp Engineサービスを作ることができる。

app.yamlのhandlersマッチング
----------------------------

**app.yaml** の ``handlers`` は、書いた順番に上からマッチしていく。
なので例えば::

	handlers:
	  - url: /.*
	    script: _go_app
	    login: required
	  - url: /_ah/queue/.*
	    script: _go_app
	    login: admin

のように書くと、先に ``/.*`` が評価されて、``/_ah/queue/.*`` に届かない。
広い範囲にマッチするルールは、下に書くように注意する。
