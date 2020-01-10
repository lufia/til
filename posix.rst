========
POSIX
========

雑なメモ。

printfはスレッドセーフ
----------------------

*FILE* はロックするのでスレッドセーフ。ロックを扱う関数がいくつかある。

* flockfile
* ftrylockfile
* funlockfile

リンク
------

* `Cストリーム入出力関数とスレッド安全性 <https://yohhoy.hatenadiary.jp/entry/20130128/p1>`_
* `stdio.h <https://pubs.opengroup.org/onlinepubs/9699919799/basedefs/stdio.h.html>`_
* `flockfile(3) <https://linuxjm.osdn.jp/html/LDP_man-pages/man3/flockfile.3.html>`_
