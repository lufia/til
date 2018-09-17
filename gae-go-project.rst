Google App Engine/Goのプロジェクト
==================================

.. highlight:: yaml

GAE/Goのコーディング以外に関する情報。

パッケージ
-----------

* google.golang.org/appengine
* cloud.google.com/go

などあるが、前者はApp Engine standardで使うもの。後者はそれ以外。

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

ファイル数多すぎエラーの回避
----------------------------

``dev_appserver.py`` を実行した時、以下のエラーが発生する場合がある。

	UserWarning: There are too many files in your application for changes in all of them to be monitored. You may have to restart the development server to see some changes to your files.
	There are too many files in your application for

ファイル数が多すぎることが問題なので、vendorディレクトリなどを無視する::

	runtime: go

	skip_files:
	- .*node_modules

* `GAEの開発サーバで監視対象ファイルが多すぎてオートリロードが無効になる場合の対応方法 <https://qiita.com/nirasan/items/547c142f8676015c2d95>`_
