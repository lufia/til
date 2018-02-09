================================
MySQL・MariaDB・PostgreSQLの違い
================================

|書き方の違い    |MySQL           |MariaDB         |PostgreSQL   |
|----------------|----------------|----------------|-------------|
|プレースホルダ  |?, ?            |?, ?            |$1, $2       |
|最後に挿入したID|LAST_INSERT_ID()|LAST_INSERT_ID()|LASTVAL()    |
|文字列実行      |mysql -e '...'  |mysql -e '...'  |psql -c '...'|

Go関連
======

タイムゾーン
------------

MySQLドライバ(github.com/go-sql-driver/mysql)の場合、
接続文字列に ``loc`` パラメータを含める。
例えばUTCにする場合は ``loc=UTC`` と書く。

PostgreSQLドライバ(github.com/lib/pq)は、*libpq* のものがそのまま使える。

* `環境変数 <https://www.postgresql.org/docs/current/static/libpq-envars.html>`_
* `接続パラメータ <https://www.postgresql.org/docs/current/static/libpq-connect.html#LIBPQ-PARAMKEYWORDS>`_

接続文字列(DSN)の、*host*, *port*, *sslmode* などの
DBドライバ固有のパラメータは ``sql.Open`` 接続時に使われる。
それ以外は、接続が終わった後にスタートアップパラメータとしてDBへ送られる。
明確な記述は無いけど ``PGTZ`` 環境変数の値は ``timezone`` パラメータに
内部的にマップされているので、``timezone=Etc/UTC`` が使えるのではないかと思う。

* `Date/Timeキーワード <https://www.postgresql.org/docs/current/static/datetime-keywords.html>`_
