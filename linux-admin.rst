=========
Linux管理
=========

.. highlight:: console

よく忘れるコマンドメモ。

一般
=====

ランダム文字列生成::

	$ openssl rand -hex 16

tcpdump
---------

loopbackは通常、明記しないとキャプチャされない::

	tcpdump [-i lo0] -s0 -A src net 10.1.2.0/24 and src port 443
	tcpdump [-i lo0] -s0 -A host 10.1.2.202 or host 10.1.2.203 and port 443

システム
========

* `プロセスグループとセッショングループ <https://blog.a-know.me/entry/2016/10/27/082350>`_
* `Ctrl+Cとkill -SIGINTの違いからLinuxプロセスグループを理解する <http://equj65.net/tech/linuxprocessgroup/>`_

プロセスグループ
-----------------

複数のプロセスがまとまったもの。
基本的に最初のプロセスがグループのリーダープロセスになる。
リーダーから ``fork`` したプロセスは、同じグループに属する。
シグナルは同じグループの全プロセスに送られる。

セッショングループ
------------------

プロセスグループの集合。
パイプで繋がれたプロセスは、一般的には同じセッショングループに属する。
``ps`` コマンドの *STAT* が ``Ss`` の場合はセッションリーダーで、
``S+`` の場合はフォアグラウンド(端末にアタッチされている)という意味らしい。

SystemTap
---------

SolarisのDTraceみたいなもの。

* `SystemTapメモ <http://myokota.hatenablog.jp/entry/2015/01/03/235944>`_

管理
======

ファイル階層
------------

* `Linuxのディレクトリ構成(FHS) <http://www.7key.jp/computer/linux/directory.html>`_

ディスクエラー時の対応
----------------------

I/Oエラーが発生した時にマウントオプションで挙動を変更できる。

* `errors=remount-ro <https://www.shigemk2.com/entry/errors%3Dremount-ro>`_

ファイルシステムのデフォルトは *tune2fs* でスーパーブロックを読めば参照できる::

	$ sudo tune2fs -l /dev/sda1

ネットワークデバイス
--------------------

昔は ``eth0`` などの名前が使われたが、MACアドレスを編集するのが難しい環境や、
USBイーサネットなどの抜き差しするデバイスのために、
どこに刺さっているネットワークデバイスなのかで
``p0p1`` のような変わらない名前をつけるルールがあるらしい。

* `ネットワークインターフェースの名前 <http://blog.keshi.org/hogememo/2014/12/28/debian-vs-ubuntu-network-interface-names>`_

ifconfig/netstat vs ip/ss
-------------------------

* `There's real reasons for Linux to replace ifconfig, netstat, et al <https://utcc.utoronto.ca/~cks/space/blog/linux/ReplacingNetstatNotBad>`_

systemd
--------

* `systemd.index <https://www.freedesktop.org/software/systemd/man/index.html>`_

BPF
----

* `Why is the kernel community replacing iptables with BPF <https://cilium.io/blog/2018/04/17/why-is-the-kernel-community-replacing-iptables/>`_

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

apt関連
=======

Python
-------

Ubuntuでは、Python2と3が同時に入っているが、``pip`` は入っていない。
必要なら以下のコマンドでインストールする::

	$ apt update
	$ apt install python-pip python3-pip
	$ pip -V
	$ pip3 -V

キャッシュ
----------

``apt`` のキャッシュは */var/lib/apt/lists* 以下にある。
