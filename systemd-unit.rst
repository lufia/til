systemdユニットの設定方法
=========================

.. highlight:: ini

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
