=============
MySQL関連メモ
=============

チューニング
============

* `MySQLのクエリの良し悪しはrows_examinedで判断する <http://blog.kamipo.net/entry/2018/03/22/084126>`_

パラメータ
==========

見ておくべきパラメータ

* @@character_set_database
* @@character_set_server
* @@innodb_lock_wait_timeout
* @@innodb_max_dirty_pages_pct
* @@max_heap_table_size
* @@skip_name_resolve
* @@slow_launch_time
* @@thread_cache_size
* @@tmp_table_size

雑多
====

パスワード
----------

* 環境変数 *MYSQL_PWD* に入れておくと自動で使われる
