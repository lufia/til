=========
Linux管理
=========

.. highlight:: console

よく忘れるコマンドメモ。

Yum関連
=======

アップデートが必要
------------------

アップデートが必要なサービスをリスト::

	$ sudo needs-restarting -s

システム再起動が必要かどうか調べる::

	$ sudo needs-restarting -r

古いカーネルを削除
------------------

2つだけ残してあとは削除::

	$ sudo package-cleanup --oldkernels --count=2

プログラムをトレース
--------------------

``ltrace`` でライブラリの動作をトレースする。
``strace`` でシステムコールの動作をトレースする。

* `straceとltraceでトレース <http://szarny.hatenablog.com/entry/2017/08/27/153048>`_
