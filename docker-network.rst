Dockerのネットワーク関連
========================

.. highlight:: console

ネットワークアドレスの変更
--------------------------

Dockerは起動時に、docker0ブリッジへネットワークアドレスを割り当てる。
例えば以下のような範囲から利用可能なものを選ぶ。

* 172.17.0.0/16

``/16`` という広い範囲が割り当てられてしまうため、
ローカルのネットワークと競合する場合があるので、
その場合は ``--bip`` と ``--fixed-cidr`` オプションで切り替えると良い。

.. code-block:: ini

	[Service]
	Environment='DOCKER_OPTS=--fixed-cidr="192.168.100.0/24" --bip="192.168.100.1/24"'

* `Customize the docker0 bridge <https://docs.docker.com/engine/userguide/networking/default_network/custom-docker0/>`_
* `Network configuration <http://docs.docker.com/v1.7/articles/networking/>`_

ただし、``docker-compose`` を使っている場合は、
独自のネットワークが作られる？ので、docker0とは別のネットワークが利用されるらしい。
subnetキーを使って指定する方法もあるみたい。

.. code-block:: yaml

	version: '2'
	services:
	  app:
	    networks:
	      net
	networks:
	  net:
	    driver: bridge
	    ipam:
	      driver: default
	      config:
	        - subnet: xxx.xxx.xxx.xxx/xx
	          gateway: xxx.xxx.xxx.xxx

毎回指定するのは面倒なのでなんとかならないだろうか。

* `Docker network周りでハマった話 <http://junchang1031.hatenablog.com/entry/2016/06/15/020545>`_
* `Compose file reference <https://docs.docker.com/compose/compose-file/>`_
