===========
fluentd関連
===========

基本
=====

実装の詳細と、ログ消失の可能性について。

* `fluentdの基礎知識 <https://abicky.net/2017/10/23/110103/>`_

Go言語
------

* `fluent-logger-golangの実践的な使い方まとめ <https://sfujiwara.hatenablog.com/entry/2017/12/14/135529>`_

GCPログ
-------

fluentdでGCPのログに送る場合は ``GOOGLE_APPLICATION_CREDENTIALS`` 環境変数にJSONファイルのパスを与えておくと良い。

* `Google Cloud Logging driver <https://docs.docker.com/config/containers/logging/gcplogs/>`_
* `GCP外のホストからDockerコンテナのログをStackdriver Loggingに送る <https://www.xmisao.com/2017/04/23/send-docker-container-logs-to-stackdriver-logging-from-the-outside-of-gcp.html>`_
* `GCPのStackdriver Loggingを使ってエラーログを集める <http://sekaie.hatenablog.com/entry/2016/07/06/090737>`_
