=======================
Linuxパフォーマンス関連
=======================

.. highlight: console

vmstatでざっくり調べる
======================

2秒ごとに ``vmstat`` を取得する(単位はMB)::

	$ vmstat -w -S m 2

procs
	r
		実行キューに入っているプロセス数

	b
		実行可能だがブロックされているプロセス数

procsのr数値が高い
------------------

実行中のプロセスは ``ps`` コマンドの *STAT* フィールドで **R** が付いている。
**R** フラグは、実行中またはrun queueに入っているものに付くので、
run queueのみを調べる方法はなさそう。

	$ ps -ef

``top`` でも調べられるらしいがよくわからなかった。

* `vmstatの見方と考え方 <http://piro791.blog.so-net.ne.jp/2008-10-02>`_
* `LinuxのI/OやCPUの負荷とロードアベレージの関係を詳しく見てみる <https://qiita.com/kunihirotanaka/items/21194f77713aa0663e3b>`_
* `I/O負荷の正確な状況はiowaitでは分かりません <https://qiita.com/kunihirotanaka/items/a536ee35d589027e4a5a>`_

メモリ
------

VSS(VSZ)
	プロセスのサイズ

	``malloc`` で割り当てられたけど、
	まだ書き込まれていない(ページが割り当てられていない)アドレスも含む

RSS
	プロセスが使用するアドレスサイズ

	共有ライブラリのアドレスを含む

PSS
	RSSと似ているが、共有ライブラリは共有するプロセス数で割る

USS
	プロセスが使用する固有のメモリサイズ

* `VSS RSS PSS USS の説明 <http://gntm-mdk.hatenadiary.com/entry/2015/01/21/231258>`_

スケジューラ
============

プロセスの状態
---------------

TASK_RUNNING
	実行中または実行待ち

TASK_INTERRUPTIBLE
	割り込み、シグナルなどでウェイトキューにある

TASK_UNINTERRUPTIBLE
	TASK_INTERRUPTIBLEと同じだがシグナルで起きない

TASK_STOPPED
	停止状態、ランキューまたはウェイトキューのどちらにもない

* `Linuxのしくみを学ぶ - プロセス管理とスケジューリング <https://syuu1228.github.io/process_management_and_process_schedule/process_management_and_process_schedule.html>`_
