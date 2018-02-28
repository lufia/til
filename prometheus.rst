==============
Prometheus関連
==============

.. highlight:: yaml

Prometheus 2.0へのアップグレード
--------------------------------

Prometheus 2.0からアラートの設定ファイルやオプションが変わった。

* `Prometheus 2.0 migration guide <https://github.com/prometheus/prometheus/blob/master/docs/migration.md>`_

.. code-block:: console

*alert.rules* は以下のコマンドで変換できる::

	$ promtool update rules alert.rules

ExporterのURLを変更する
-----------------------

デフォルトのNode Exporterは */metrics* がエンドポイントだが、
ものによっては異なる場合がある。
だけど *prometheus.yml* の ``targets`` にはホスト名とポートしか書けない。

この場合、``metrics_path`` を使ってURLのパス部分を変更する::

	- job_name: 'jenkins'
	  metrics_path: /prometheus
	  static_configs:
	    - targets:
	        - 192.168.1.11:8080

ジョブ単位で収集間隔を変更する
------------------------------

全体はグローバルで設定する::

	global:
	  scrape_interval: 15s
	  evaluation_interval: 15s

データ取得が遅いExporterは個別に延ばす::

	scrape_configs:
	  - job_name: jenkins
	    scrape_interval: 60s
	    scrape_timeout: 50s
	    static_configs:
	      - targets: [...]
