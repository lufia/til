PostgreSQLメモ
==============

.. highlight:: sql

データベースが存在するか調べる
------------------------------

*database_name* の存在を調べる::

	SELECT *
	FROM pg_database
	WHERE datname = 'database_name'

テーブルが存在するか調べる
--------------------------

*public.table_name* の存在を調べる::

	SELECT *
	FROM information_schema.tables
	WHERE table_schema = 'public'
	AND table_name = 'table_name';

``psql`` コマンドのパスワード入力を省略する
-------------------------------------------

自動テストやDockerコンテナのビルドなどで、パスワードを省略したい場合がある。
その場合は *~/.pgpass* に書くか、*PGPASSWORD* 環境変数にセットすると良い。

*~/.pgpass* ファイルは、以下の項目を **:** で区切った書式。

1. ホスト名
2. ポート番号
3. データベース名
4. ユーザ名
5. パスワード

また、*~/.pgpass* のパーミッションは600のみ許可。

* `psqlにてパスワードを省略する方法 <https://kaede.jp/2015/10/27002723.html>`_
