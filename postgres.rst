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
