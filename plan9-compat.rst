=============================
Plan 9にUnixツールを移植する
=============================

Plan 9メモ

POSIX関連
==========

poll/epoll
-----------

非同期I/Oの参考に ``select`` が良さそう。

* `select.h <https://github.com/0intro/plan9-contrib/blob/master/sys/include/ape/select.h>`_
* `_buf.c <https://github.com/0intro/plan9-contrib/blob/master/sys/src/ape/lib/ap/plan9/_buf.c>`_
