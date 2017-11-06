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