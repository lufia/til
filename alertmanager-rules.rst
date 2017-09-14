Prometheus Alertmanagerのルール
===============================

* `公式ドキュメント <https://prometheus.io/docs/alerting/configuration/>`_

ルーティング
------------

*route.receiver* でデフォルトの宛先を指定する。
レシーバは *receivers* で複数用意できるが、デフォルトは1つだけしか設定できない。
複数のサービスへ同時にアラートを送りたい場合は、*receivers* 側に複数定義する。

以下はメールとWebhookへ同時にアラートを飛ばす例::

	route:
	  receiver: 'default-receiver'
	  group_by: ['alertname', 'instance', 'severity']
	  group_wait: 30m
	  group_interval: 10m
	  repeat_interval: 3h
	
	receivers:
	  - name: 'default-receiver'
	    email_configs:
	      - to: 'user1@example.com'
	        send_resolved: true
	    webhook_configs:
	      - url: 'https://example.com/alerts'
