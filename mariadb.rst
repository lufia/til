MySQL・MariaDB・PostgreSQLの違い
================================

|書き方の違い    |MySQL           |MariaDB         |PostgreSQL   |
|----------------|----------------|----------------|-------------|
|プレースホルダ  |?, ?            |?, ?            |$1, $2       |
|最後に挿入したID|LAST_INSERT_ID()|LAST_INSERT_ID()|LASTVAL()    |
|文字列実行      |mysql -e '...'  |mysql -e '...'  |psql -c '...'|
