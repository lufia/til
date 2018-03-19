Plan 9のインストール関連
========================

macOSでQEMU
-----------

kvmが無いのでとても遅いが、Hypervisor.framework対応版が存在した。
これは最終的に本家へマージされるのが目標らしい。

* `Qemu on MacOSX with Hypervisor Framework <http://www.breakintheweb.com/2017/10/14/Qemu-on-MacOSX-with-Hypervisor-Framework/>`_

QEMUで ``-nographic`` 起動すると止まる
--------------------------------------

通常のグラフィックスで起動すると問題ないのに、
``-nographic`` オプションをつけると *9load* がカーネルを
読み込み終わった後、処理がカーネルに移った直後から進まなくなる。

通常のコンソールでは::

	pcirouting: BIOS workaround: PCI.0.1.3 at pin 1 link 96 irq 10 -> 9
	 disk loader

	cpu0:  2201MHz GenuineIntel Celeron (cpuid: AX 0x0663 DX 0x781ABFD)
	ELCR: 0C00
	497M memory: 497M kernel data, 0M user, 18M swap
	found partition #S/sdC0/data 0 4,194,304
	disks: sdC0 sdD0
	trying sdC0....found 9pcf
	.1212060...................................................................................................................................................+2066268.............................................................................................................................................................................................................................................................+458996=3737324
	entry: 0xf0100020

	Plan 9
	E820: 00000000 0009fc00 memory

のように進むが、``entry: 0xf0100020`` で止まってしまう。

**Plan 9** 以降のログはカーネルが出力している。
原因は、*plan9.ini* に *console=* が定義されていないので、
ログが流れないだけだった。
なので先に通常起動して、*plan9.ini* にコンソール設定を入れれば良い。

インストーラの場合はあらかじめ *console=* を設定できないので、
``-nographic`` の代わりに ``-curses`` を使って起動する。
インストール中もグラフィックスを使いたくないなら、
例えば ``vgasize [640x480x8]:`` のプロンプトに適当な値を入れると
テキストモードでインストーラが起動する。

QEMUでhangしたので終了させたい
------------------------------

Ctl+Aを叩いてからCでメニューに入れる。
``(qemu)`` プロンプトが表示されたら、``quit`` で抜けられる。

GCEでのインストール
-------------------

GCEでPlan 9を使うときは通常環境と異なり以下の制限がある。

* ディスクイメージはGNUの *tar* で、``tar -Scz`` する
* ファイル名はなんでもいいが内容は **disk.raw** が含まれる
* カーネルにはVirtioドライバが必要
* 起動ディスクは */dev/sd01* にアタッチされる
* *plan9.ini* にコンソールの設定が必要

Gitを使う
---------

.. highlight:: console

* `Git wrapper <http://www.9legacy.org/9legacy/tools/git>`_
* `dgit <https://github.com/driusan/dgit>`_

Git wrapperはそのままダウンロードして/に *bind* すればいい::

	% hget http://www.9legacy.org/9legacy/tools/git >$home/bin/rc/git
	% bind -a $home/bin/rc /bin    # 必要なら

簡単だけれど履歴などが取れないので、*dgit* の方が便利::

	% GOPATH=$home
	% mkdir -p $GOPATH/src/github.com/driusan
	% cd $GOPATH/src/github.com/driusan
	% git clone https://github.com/driusan/dgit
	% go get https://github.com/driusan/dgit

*dgit* を使うためには最低でも以下の設定が必要だった。
まずは *$home/.gitconfig* に *user.name* と *user.email* を追加::

	% git config user.name lufia
	% git config user.email lufia@lufia.org
	% cat $home/.gitconfig
	[user]
		name = lufia
		email = lufia@lufia.org

次に、GitHubへ ``go get`` でアクセスするために証明書を更新::

	% hget http://www.9legacy.org/9legacy/patch/ca.diff >/tmp/ca.diff
	% cd /
	% ape/patch -p1 </tmp/ca.diff

これで ``go get`` できる::

	% GOPATH=$home
	% go get github.com/lufia/qsh

標準実装と異なり、途中でエラーになってもファイルは残るので、
自分で不要なファイルを削除する必要がある。
