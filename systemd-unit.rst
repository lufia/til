systemdユニットの設定方法
=========================

.. highlight:: ini

ユニットの設定を検証する
------------------------

以下のコマンドで記述間違いがあるかを確認する。

.. code-block:: console

	$ systemd-analyze verify docker.service

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
