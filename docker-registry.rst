================
Dockerレジストリ
================

.. highlight:: console

管理
======

Dockerイメージを削除可能にする
-------------------------------

通常はイメージ削除を行えず、削除APIを実行しても *UNSUPPORTED* エラーになる。

APIからのイメージ削除を許可するには、
環境変数 *REGISTRY_STORAGE_DELETE_ENABLED* を *true* に設定して起動する::

	$ docker run -e REGISTRY_STORAGE_DELETE_ENABLED=true registry:latest

API
=======

イメージのリスト取得
---------------------

*_catalog* を参照する::

	$ curl https://example.com/v2/_catalog
	{
		"repositories": [
			"test/builder",
			"sdk/rpmbuild"
		]
	}

特定イメージのタグリスト取得
----------------------------

*(image)/tags/list* を参照する::

	$ curl https://example.com/v2/test/builder/tags/list
	{
		"name": "test/builder",
		"tags": ["latest"]
	}

* `Docker Registry HTTP API v2 <https://github.com/docker/distribution/blob/master/docs/spec/api.md>`_
