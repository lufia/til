systemdユニットの設定方法
=========================

.. highlight:: ini

ユニットの設定を検証する
------------------------

以下のコマンドで記述間違いがあるかを確認する。

.. code-block:: console

	$ systemd-analyze verify docker.service

IPルーティング設定
------------------

ネットワークの設定は */etc/systemd/network/* で行う。
以下は固定IPアドレスとルーティングをセットした例::

	[Match]
	Name=eth0

	[Network]
	Address=192.168.1.3/24
	Gateway=192.168.1.1
	DNS=192.168.1.1
	DNS=192.168.1.2

	[Route]
	Gateway=192.168.1.123
	Destination=10.1.2.0/24

設定が終わったら再起動するか、ネットワークを再スタートする。

.. code-block:: console

	# systemctl daemon-reload
	# systemctl restart systemd-networkd

空の変数を空文字列として扱う
----------------------------

例えば以下の設定があった場合::

	[Service]
	Environment=USER=
	ExecStart=command -u $USER

実際に実行されるのは ``command -u`` となってしまって、
``-u`` オプションの引数がないので実行エラーになってしまう。
また、``ExecStart`` はシェルではないため、``"$USER"`` とした場合は
コマンドには ``command -u '"$USER"'`` のように、そのままの文字が渡される。

空文字列を表現するためには、``${USER}`` のように書く必要があった。
最終的には以下のように::

	[Service]
	Environment=USER=
	ExecStart=command -u ${USER}

スケジュールされた実行
----------------------

タイマーで実行させることができる::

	[Timer]
	OnCalendar=weekly
	Persistent=true

	[Install]
	WantedBy=timers.target

``Type=oneshot`` と組み合わせると良さそう::

	[Service]
	Type=oneshot
	ExecStart=/bin/certbot renew
	ExecStartPost=/bin/systemctl reload httpd.service

* `cronの代わりにsystemdのtimerを使う <http://blog.n-z.jp/blog/2017-06-04-cron-systemd-timer.html>`_
