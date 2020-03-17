================
Google Cloud SDK
================

.. highlight:: console

一般
====

設定ファイルの場所
-----------------

``gcloud`` の設定
	*~/.config/gcloud/*

``gsutil`` の設定
	*~/.gsutil/*

リリース情報
------------

* `Go Release Notes <https://cloud.google.com/appengine/docs/standard/go/release-notes>`_

アカウント関連
--------------

ログインする::

	$ gcloud auth login

コンフィグ関連
--------------

ログインやプロジェクトなどをコンフィグとして切り替えることができる。

リスト取得::

	$ gcloud config configurations list

追加::

	$ gcloud config configurations create $NAME
	$ gcloud config configurations set project xx
	$ gcloud config configurations set account xx@gmail.com

切り替え::

	$ gcloud config configurations activate $NAME

削除::

	$ gcloud config configurations delete $NAME

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

Compute Engine
==============

シリアルポートに接続
--------------------

gcloudを使う::

	gcloud compute --project=<project-id> \
		connect-to-serial-port <instance-name> \
		--zone=us-central1-a

初回の場合、**~/.ssh/google_compute_engine** の生成が行われる。
これで生成した公開鍵はGCEのメタデータに自動的に追加される。

メタデータは *project-info* サブコマンドで確認できる::

	gcloud compute project-info describe

ストレージ
===========

* `perfdiag - Run performance diagnostic <https://cloud.google.com/storage/docs/gsutil/commands/perfdiag>`_
