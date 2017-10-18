Dockerコマンド
==============

.. highlight:: console

``docker cp`` コマンドの展開ルール
----------------------------------

* `docker cp <https://docs.docker.com/engine/reference/commandline/cp/>`_ 

``docker cp`` はコピー元とコピー先がディレクトリかどうかによって、
展開する際の動作が異なる。

コンテナ内ディレクトリの内容をホストの既存ディレクトリへ展開する
(コピー元パスの最後に **/.** が必要)::

	$ mkdir /backup
	$ docker cp a93c:/app/data/. /backup/

コンテナ内のファイルをホストの特定ディレクトリへ展開する::

	$ mkdir /backup
	$ docker cp a93c:/app/config.yml /backup/

コンテナ内のファイルでホストのファイルを上書きする::

	$ touch /backup/config.yml
	$ docker cp a93c:/app/config.yml /backup/config.yml

上記の例では、コピー元をコンテナにしているが、逆でも同じ。
