===========
kubernetes
===========

.. highlight:: yaml

概要
====

Container Linuxで動作させる場合は、以下の用意が必要。

* etcd
* flanneld
* マスターノード
* ワーカーノード
* kubectl (クライアント)

マスターノードはワーカーと同じでも良いが分ける。

準備
======

Container LinuxはIgnition Configで初期化する。

ct
------

Container Linux Config trainspilerでYAMLをJSONに変換するツール。

.. code-block:: console

Nixpkgsでインストールするには::

	$ nix-env -i ct

.. code-block:: console

``ct`` でContainer Linux ConfigをIgnition Configに変換する::

	$ ct -in-file config.yaml -out-file ignition.json

.. code-block:: makefile

*Makefile* を作っておくと良い::

	TARG=config.iso
	DIR=img

	.PHONY: all clean nuke

	all: $(TARG)

	%.iso: %.json
		rm -rf $(DIR)/*
		mkdir -p $(DIR)
		cp $< $(DIR)/$<
		hdiutil makehybrid -iso -joliet -default-volume-name $* -o $@ $(DIR)

	%.json: %.yaml
		ct -in-file $< -out-file $@

	clean:
		rm -rf $(DIR)

	nuke:
		rm -rf $(DIR) $(TARG)

ここではISOイメージを作成している。
ネットワーク等でインストール前のContainer Linuxとファイルコピーが可能なら
ISOを経由せず直接JSONを作成しても良いが、手元では無かったのでISOにした。

Ignition Configを使ってインストール
-----------------------------------

2番目のCD-ROMは、インストーラから以下のコマンドでマウントできる::

	$ sudo mount -o ro -t iso9660 /dev/sr1 /media

``coreos-install`` に ``-i`` オプションでIgnition Configのパスを渡す::

	$ sudo coreos-install -d /dev/sda /media/config.json

ログイン可能にする
------------------

なくても良いけど、ログインできた方が便利なのでContainer Linux Configに書く::

	passwd:
	  users:
	    - name: core
	      ssh_authorized_keys:
	        - "ssh-rsa xxxx"
	      password_hash: $6$xxxx

``password_hash`` は無くても良いが、設定を間違った時の確認に便利::

	$ mkpasswd -m sha-512
	Password:

ネットワーク関連設定
--------------------

ホスト名を設定する::

	storage:
	  files:
	    - path: /etc/hostname
	      filesystem: root
	      mode: 0644
	      contents:
	        inline: (ホスト名)

固定IPアドレスと静的ルートを設定する::

	networkd:
	  units:
	    - name: 10-static.network
	      contents: |
	      [Match]
	      Name=eth0

	      [Network]
	      Address=192.168.1.10/24
	      Gateway=192.168.1.1
	      DNS=192.168.1.1
	      DNS=192.168.1.2

	      [Route]
	      Gateway=192.168.1.124
	      Destination=10.45.0.0/16

静的ルートがなければ ``[Route]`` は無くてもよい。

etcdのインストール
==================

Container Linuxなら設定を書くだけで有効になる。

etcdのインストール
------------------

Container Linux Configに専用のエントリがある::

	etcd:
	  name: app-etcd-1
	  listen_client_urls: http://127.0.0.1:2379
	  advertise_client_urls: http://localhost:2379
	  listen_peer_urls: http://127.0.0.1:2380
	  initial_advertise_peer_urls: http://127.0.0.1:2380
	  initial_cluster: app-etcd-1=http://127.0.0.1:2380
	  initial_cluster_token: xxxx
	  initial_cluster_state: new

.. code-block:: console

動作確認
--------

正しく構築できれば、以下のコマンドで操作できる::

	$ etcdctl ls /
	$ etcdctl mkdir /test
	$ etcdctl set /test/key 'aaaa'
	$ etcdctl get /test/key
	$ etcdctl rm /test/key
	$ etcdctl rmdir /test

flanneld
========

flanneldのインストール
----------------------

ホスト起動時に、flanneldに必要な設定を行う::

	systemd:
	  units:
	    - name: flanneld.service
	      dropins:
	        - name: 50-network-config.conf
	          contents: |
	            [Service]
	            ExecStartPre=/usr/bin/etcdctl set /coreos.com/network/config '{"Network": "10.1.0.0/16"}'

このとき、``Network`` の範囲がホストのネットワークと重なってしまうと、
*sshd* なども全て ``Network`` 側に流れてしまって管理ができなくなるので注意。

flanneldを有効にする::

	flannel: ~

参考情報
========

* `Getting started with etcd <https://coreos.com/etcd/docs/latest/getting-started-with-etcd.html>`_
* `Configuring flannel for container networking <https://coreos.com/flannel/docs/latest/flannel-config.html>`_
* `Kubernetesにまつわるエトセトラ <https://www.slideshare.net/WorksApplications/kubernetes-65070472>`_
