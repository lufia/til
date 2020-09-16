Dockerのネットワーク関連
========================

.. highlight:: console

Dockerネットワーク
-------------------

* `Dockerネットワークの基礎 <http://tech.uzabase.com/entry/2017/08/07/172411>`_

ホストとの疎通
--------------

* **host.docker.internal**

ネットワークアドレスの変更
--------------------------

Dockerは起動時に、docker0ブリッジへネットワークアドレスを割り当てる。
例えば以下のような範囲から利用可能なものを選ぶ。
コードは `docker/libnetwork:ipamutils/utils.go <https://github.com/docker/libnetwork/blob/master/ipamutils/utils.go>`_ の辺り。

* 172.17.0.0/16
* 172.18.0.0/16
* 172.19.0.0/16

``/16`` という広い範囲が割り当てられてしまうため、
ローカルのネットワークと競合する場合があるので、
その場合は ``--bip`` と ``--fixed-cidr`` オプションで切り替えると良い。

.. code-block:: ini

	[Service]
	Environment='DOCKER_OPTS=--fixed-cidr="192.168.100.0/24" --bip="192.168.100.1/24"'

* `Customize the docker0 bridge <https://docs.docker.com/engine/userguide/networking/default_network/custom-docker0/>`_
* `Network configuration <http://docs.docker.com/v1.7/articles/networking/>`_

ただし、``docker-compose`` を使っている場合は、``docker-compose up`` した時に
独自のネットワークが作られてしまうため、docker0とは別のネットワークになる。
``subnet`` キーを使って指定する方法や、
``external`` で事前に作成しておいたネットワークを使う方法はあるが、
どちらもネットワークアドレスが重複しないように気をつける必要があって微妙。

以下は ``subnet`` を使った例。

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

どうやら事前に、重複するルーティング設定をしておくと良いらしい。
systemdを使う場合は、*/etc/systemd/network/10-static.network* あたりに書く。

.. code-block:: ini

	[Match]
	Name=eth0

	[Route]
	Gateway=10.1.2.254
	Destination=172.17.0.0/16

同等のコマンドライン。

.. code-block:: bash

	ip route add 172.17.0.0/16 via 10.1.2.254 dev eth0

* `Docker network周りでハマった話 <http://junchang1031.hatenablog.com/entry/2016/06/15/020545>`_
* `Change Docker network bridge IP range <https://mogutan.wordpress.com/2016/12/28/change-docker-network-bridge-ip-range/>`_
* `Compose file reference <https://docs.docker.com/compose/compose-file/>`_
* `How does compose chooses subnet for default network? #4336 <https://github.com/docker/compose/issues/4336>`_
