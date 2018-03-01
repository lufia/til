============
acme環境構築
============

一般ツール
==========

Watch
-----

``syscall.Kqueue()`` が使われているのでPlan 9では動かなそう。

* https://github.com/rsc/rsc/blob/master/cmd/Watch/main.go

Go関連
======

A
-----

.. code-block:: console

*acme* からGoのソースを扱うコマンド::

	$ go get github.com/davidrjenni/A

マウスカーソルを関数名などに移動させて ``A def`` で定義にジャンプする。

aw
-----

.. code-block:: console

*acme* でGoを書きやすくする::

	$ go get github.com/mjibson/aw
