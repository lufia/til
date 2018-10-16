CentOS関連
==========

SCL(Software Collections)
-------------------------

SCLは、RHELSC語幹の *centos-release-scl-rh* と、CentOS SCLo SIGの *centos-release-scl* の2つある。
基本はRHELSC互換のものを使うが、足りないものをCentOS SCLo SIGパッケージで補う。

* http://www.tooyama.org/el7/scl.html
* https://access.redhat.com/ja/solutions/2112231

``scl enable [package ...] [command]`` は、*package* (複数可能) に含まれるコマンドを使える状態にして *command* を実行する。
ほとんどは、*command* で ``bash`` を起動することが多い。

リカバリモードでネットワークを使う
----------------------------------

リカバリモードはネットワークが停止している状態で起動する。
例えば誤って ``lvm2`` パッケージを削除してしまった場合、
``yum`` から入れなおさなければいけないのでネットワークが必要になる。

.. code-block:: console

	$ ifup eth0
	$ dhclient eth0

以後、ネットワークが通常通り使えるようになる。

RPMビルド
-----------

``%setup`` マクロを使わない場合、``Source`` タグも不要になる。
単純にファイルを展開したい場合などに使える。
または ``NoSource`` タグを使ってもいい。

*%{_topdir}/SOURCES* はspecファイルの中で
*$RPM_SOURCE_DIR* または *%{_sourcedir}* として参照できる。

* `RPMパッケージ作成メモ <http://www.02.246.ne.jp/~torutk/linux/centos5/rpmpackagebuild.html>`_
* `%setupマクロの詳細 <https://vinelinux.org/docs/vine6/making-rpm/setup-macro.html>`_

``%setup`` デフォルトではソースコード展開のルールが決まっているけれど、
展開後のディレクトリ名などは、オプションでいくつかは変更可能。
