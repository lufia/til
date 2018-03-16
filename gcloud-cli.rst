================
Google Cloud SDK
================

.. highlight:: console

一般
====

リリース情報
------------

* `Go Release Notes <https://cloud.google.com/appengine/docs/standard/go/release-notes>`_

アカウント関連
--------------

ログインする::

	$ gcloud auth login

プロジェクト関連
----------------

リスト取得::

	$ gcloud projects list

カレントプロジェクト確認::

	$ gcloud config get-value project

カレントプロジェクト設定::

	$ gcloud config set project $PROJECT_ID

App Engine
==========

バックアップ
------------

* `エンティティのエクスポートとインポート <https://cloud.google.com/datastore/docs/export-import-entities>`_
* `エクスポートのスケジューリング <https://cloud.google.com/datastore/docs/schedule-export>`_

バケットの確認::

	$ gsutil ls
	gs://bucket/
	$ gsutil sl gs://bucket/
	gs://bucket/sample.txt

Datastoreのバックアップ::

	$ gcloud datastore export --namespaces='(default)' gs://bucket/

リストア::

	$ gcloud datastore import gs://bucket/**/*.overall_export_metadata
