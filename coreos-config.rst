Container Linux (CoreOS)の設定
==============================

.. highlight:: console

以前はCloud Configを使って設定を行なっていたが、
読み込むタイミングが遅いとかYAMLパーサが誤解するとかあったらしく、
今はIgnitionというJSONファイルが使われるようになった。

* https://coreos.com/os/docs/latest/migrating-to-clcs.html
* https://coreos.com/os/docs/latest/customizing-docker.html
* https://coreos.com/os/docs/latest/configuration.html
* https://blog.lorentzca.me/install-container-linux-to-disk/

Ignitionを与える方法はいくつかある。

ディスクに新規インストールする場合
----------------------------------

``coreos-install`` でインストールする時に、
``-i`` オプションを使ってIgnitionファイルのパスを渡す。

こうすると、引数で渡したファイルが **LABEL=OEM** なパーティションに、
*coreos-install.json* という名前で保存され、
さらにカーネルのコマンドラインに ``coreos.config.url=oem:///coreos-install.json`` が追加される。

	set linux_append="$linux_append coreos.config.url=oem:///coreos-install.json"

**LABEL=OEM** なパーティションは ``sudo blkid -t "LABEL=OEM"`` で調べられる。

既存のディスクブート環境を更新する場合
--------------------------------------

Ignitionはディスクのフォーマット等も行うので、最初のブート時のみ動作する。
Cloud Configと違って起動時に上書きされないので、必要なファイルは直接編集すればいいらしい。

* `What is Ignition? <https://coreos.com/ignition/docs/latest/what-is-ignition.html>`_

Container Linuxは、*/dev/sda3* と */dev/sda4* のどちらかを */usr* にマウントする。
アップデートする際には、使われていない方のパーティションを更新し、*/usr* を切り替えて再起動する。
パーティションの切り替えは */usr* だけなので、*/etc* や */var* などは残る。
そのため、一般的なLinuxサーバと同じように手で更新してもいいし、
Ansibleなどの構成管理ツールを使って管理してもいい。

* `Performing manual CoreOS Container Linux rollbacks <https://coreos.com/os/docs/latest/manual-rollbacks.html>`_
* `Managing CoreOS with Ansible <https://coreos.com/blog/managing-coreos-with-ansible.html>`_

DockerのログをGCPへ送る
-----------------------

とりあえず以下の設定で動作した。

/etc/docker/daemon.json:

.. code-block:: json

	{
		"log-driver": "gcplogs",
		"log-opts": {
			"gcp-project": "(プロジェクトID)",
			"gcp-meta-name": "azuki"
		}
	}

/etc/systemd/system/docker.service.d/20-gcp-credentials.conf::

	[Service]
	Environment="GOOGLE_APPLICATION_CREDENTIALS=/etc/docker/credential.json"

* `GCP外のホストからDockerコンテナのログをStackdriver Loggingに送る <https://www.xmisao.com/2017/04/23/send-docker-container-logs-to-stackdriver-logging-from-the-outside-of-gcp.html>`_

ディスク容量を増やす
--------------------

途中でディスク容量を拡張したい場合、
``qemu-img resize`` などでディスクを拡張してOSを再起動すれば良い。

* `Adding disk space <https://coreos.com/os/docs/latest/adding-disk-space.html>`_

ディスクを追加する
------------------

systemdでマウントすれば良いらしい。以下は */dev/sdb* の場合::

	$ sudo mkfs.ext4
	$ sudo mkdir /mnt/data
	$ cat /etc/systemd/system/mnt-data.mount
	[Unit]
	Before=local-fs.target
	
	[Mount]
	What=/dev/sdb
	Where=/mnt/data
	Type=ext4
	
	[Install]
	WantedBy=local-fs.target
	$ sudo systemctl daemon-reload
	$ sudo systemctl start mnt-data.mount
	$ sudo systemctl enable mnt-data.mount

これでContainer Linuxから認識できたので、あとはコンテナから参照して使う。
おそらく `local-persistドライバ <https://github.com/CWSpear/local-persist>`_ を使うと便利。

* `Mounting storage <https://coreos.com/os/docs/latest/mounting-storage.html>`_
