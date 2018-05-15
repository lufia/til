======================
Plan 9関連プロジェクト
======================

公式
====

Plan 9
-------

* `Plan 9 from Bell Labs <http://plan9.bell-labs.com/plan9/>`_

本家。比較的よく落ちる。2015年1月版で更新は停止中。
他のフォークと並べる時に、labsやvanillaと表記されることもある。

Plan 9 from User Space
-----------------------

* `Plan 9 from User Space <https://9fans.github.io/plan9port/>`_

Plan 9ツールをUnixで動作させるもの。

Inferno
-------

* `Inferno <http://www.vitanuova.com/inferno/>`_

Plan 9をベースとした別のシステム。
VMとして動作し、Inferno専用のLimbo言語でプログラムを書く。

派生プロジェクト
================

9p.io
------

* `9p.io <http://9p.io/plan9/>`_

公式サイトのミラー。plan9.bell-labs.comはよく落ちるので、
最近はこちらを参照することが多い気がする。

9legacy
--------

* `9legacy <http://9legacy.org>`_

ベル研のPlan 9最新版に対するパッチ集。
Goがサポートしているのはこれ。

9front
------

* `9FRONT.ORG <http://9front.org>`_

Plan 9のフォーク。機能としては最も進んでいるが、
拡張版の ``sam`` や、``venti+fossil`` の代わりに ``cwfs`` がデフォルト。
公式サイトは個性的。

9atom
------

* `9atom <http://www.9atom.org>`_

Erikさんが開発しているフォーク。
サポートされるハードウェアが増えているけど最近は停止中？

Harvey OS
---------

* `Harvey OS <https://harvey-os.org>`_

Plan 9のフォークで、Plan 9をgccでコンパイルできるようにしたもの。
2018年現在、おそらくフォークの中では最も活発に開発されている。

Octopus
--------

* `The Octopus <http://lsub.org/ls/octopus.html>`_

Infernoベースの分散リソース。

NIX
------

* `nix-os <https://code.google.com/archive/p/nix-os/>`_

純関数型OSとは別のもの。

関連ペーパー
=============

* `/sys/doc <http://9p.io/sys/doc/>`_
* `Papers (Plan 9 wiki) <https://9p.io/wiki/plan9/papers/>`_
* `Fast, Inexpensive Content-Addressed Storage in Foundation <https://swtch.com/~rsc/papers/fndn/>`_
