Dockerコマンド
==============

.. highlight:: console

CPUとメモリの状況を調べる
-------------------------

``docker stats`` で調べられる。
引数でハッシュを与えると、そのコンテナだけ絞り込む。
``-a`` オプションで全てのコンテナ。

依存サービスとの通信
--------------------

dockerでlinkしたMySQLがまだ立ち上がっていない場合、
connection refusedになるが、restart: alwaysを入れておくと良い

他のコンテナと名前で通信したい場合、

.. code-block:: yaml

	version: '2'
	services:
	  back:
	    image: mysql
	  front:
	    links:
	      - back:alias
	    restart: always

のようにすれば *front* コンテナから *alias* という名前で *back* と通信できる。

また、上記の例では *back* が立ち上がる前に *front* が実行されることが起きるけど、
そういう場合にエラー終了してしまうため、``restart: always`` を入れておくといい。

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

権限追加
--------

dockerコマンドで--cap-add=NET_ADMINのように実行すると権限が付与される。
docker-composeでの書き方:

.. code-block:: yaml

	services:
	  (container-name):
	    cap_add:
	      - NET_ADMIN

ホスト名解決
------------

*docker-compose.yml* に ``extra_hosts`` を追加する。
同じホスト名が複数あった場合、最後のものしか有効にならない。

.. code-block:: yaml

	services:
	  (container-name):
	    extra_hosts:
	      - "hostname:192.168.1.3"

Docker-composeで複数インスタンス
--------------------------------

``scale`` オプションを使う::

	$ docker-compose scale (イメージ名)=(インスタンス数)
