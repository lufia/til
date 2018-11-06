==============
Docker関連メモ
==============

.. highlight:: console

イメージ作成
============

キャッシュの削除
----------------

``apt`` は */var/lib/opt/lists* 以下にキャッシュを持っているので削除する::

	# rm -rf /var/lib/apt/lists/*

``pip`` は実行したユーザの *~/.cache/pip* に残っているので削除する::

	# rm -rf ~/.cache/pip

Compose
=========

構文確認
----------

configサブコマンドで確認できる::

	$ docker-compose [-f docker-compose.yml] config

v3系への移行
--------------

**docker-compose.yml** をv3系に移行すると、``volumes_from`` が使えなくなる。
代わりに、``volumes`` 属性を使う必要がある。

新規で作る場合は、

.. code-block:: yaml

	services:
	  mysql:
	    image: mysql:latest
	    volumes:
	      - data:/var/lib/mysql
	volumes:
	  data:

とすれば良いが、既存のデータがある場合は、
ボリュームコンテナを ``volumes`` へ移行しなければならない。

例えばv1でこのような記述をした場合、

.. code-block:: yaml

	data:
	  build: ./data	# VOLUME ["/var/lib/mysql", "/var/lib/redis"]
	mysql:
	  image: mysql:latest
	  volumes_from:
	    - data
	redis:
	  image: redis:latest
	  volumes_from:
	    - data

v3に移行すると以下のようになる。

.. code-block:: yaml

	version: '3.1'
	services:
	  mysql:
	    image: mysql:latest
	    volumes:
	      - db:/var/lib/mysql
	  redis:
	    image: redis:latest
	    volumes:
	      - cache:/var/lib/redis
	volumes:
	  db:
	    external:
	      name: (dataコンテナの/var/lib/mysqlボリュームID)
	  cache:
	    external:
	      name: (dataコンテナの/var/lib/redisボリュームID)

ボリュームIDは、``docker volume ls`` を実行すると
``VOLUME`` 単位で列挙されるので、``docker volume inspect`` や
**/var/lib/docker/volumes/** 以下の内容から頑張って探す。

Dockerコマンド
==============

``-v(--volume)`` と ``--mount`` の違い
--------------------------------------

どちらもマウントするためのオプションだけど、
``--mount`` の方が、ファイルをマウント可能など色々拡張されている。
今なら ``--mount`` を使ったほうがいい。

* `Use bind mounts <https://docs.docker.com/engine/admin/volumes/bind-mounts/>`_

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

Docker-composeでコンテナ名を指定する
------------------------------------

通常はディレクトリ名がプリフィックスに付くが、
異なる名前を付けたい場合は、環境変数 ``COMPOSE_PROJECT_NAME`` を設定する。

.. code-block:: bash

	export COMPOSE_PROJECT_NAME=xxx
	docker-compose build
	docker-compose up -d

Docker service
--------------

使いどころはよくわからないけど、``docker run`` 相当のことができそう。

``docker-compose`` は ``docker stack deploy`` に
------------------------------------------------

* `Docker Compose入門～今日から始めるComposeの初歩からswarm mode対応まで <https://www.slideshare.net/zembutsu/docker-compose-and-swarm-mode-orchestration>`_

不要なオブジェクトを削除する
----------------------------

未使用のイメージを削除::

	$ docker image prune

未使用のボリュームを削除::

	$ docker volume prune

未使用のネットワークを削除::

	$ docker network prune

未使用のコンテナを削除::

	$ docker container prune

上記全てを一括で::

	$ docker system prune

Swarmモード
============

Swarmモードでなければ使えないサブコマンドがいくつかある。
特に ``docker secret`` が欲しい。

docker swarm init
-------------------

Swarmクラスタを初期化してマネージャに昇格させる。
初期化時にトークンが出力されるが、
この値は後から取り出すことが可能なので忘れても良い。

destroyやpurgeのような、Swarmクラスタを破棄するコマンドは用意されていない。
なので自分でネットワークなどを破棄する::

	$ docker swarm leave -f
	$ docker network ls --filter label=com.docker.compose.project
	$ docker network rm ...

* `What is opposite of docker swarm init <https://stackoverflow.com/questions/48345602/what-is-opposite-of-docker-swarm-init>`_

docker secret create
---------------------

シークレット等をファイルとして与えることができる::

	$ docker secret create (name) (file)

このファイルは、コンテナの **/run/secrets/(name)** にマウントされる。
環境変数として渡すことはできなさそう。

* `Docker ComposeのSecretsを試す <https://blue1st-tech.hateblo.jp/entry/2017/08/27/230546>`_
* `Manage sensitive data with Docker secrets <https://docs.docker.com/engine/swarm/secrets/>`_

docker-compose
---------------

composeはSwarmモードに対応していない。代わりに ``stack deploy`` を使う::

	$ docker stack deploy --compose-file docker-compose.yml (name)
