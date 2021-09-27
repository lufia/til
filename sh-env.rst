=============
シェルのモード
=============

.. highlight:: console

ルール
======

*bash* や *zsh* には以下の区別がある。

ログインシェル::

	ログインした時のシェル。

	`-l` オプションを与えてもログインシェルとなる。

インタラクティブシェル::

	対話的に利用するシェル。

	`-i` オプションを与えてもインタラクティブシェルとなる。

現在有効になっているフラグは `$-` から読み取れる::

	% echo $-
	569XZilms  # ログインシェルでありインタラクティブ(lとiがある)

	% zsh
	% echo $-
	569XZims   # インタラクティブシェル(iだけ)

	% zsh -c 'echo $-'
	569X       # どちらもない

設定ファイル
===========

zshの `Startup/Shutdown Files <http://zsh.sourceforge.net/Doc/Release/Files.html>`_ によると、

/etc/zshenvと$ZDOTDIR/.zshenv::

	常に読み込む

/etc/zprofileと$ZDOTDIR/.zprofile::

	ログインシェルの場合に読み込む

/etc/zshrcと$ZDOTDIR/.zshrc::

	インタラクティブシェルの場合に読み込む

/etc/zloginと$ZDOTDIR/.zlogin::

	ログインシェルの場合に読み込む

	zprofileとの違いはzshrcを読んだ後に実行される

/etc/zlogoutと$ZDOTDIR/.zlogout::

	ログインシェルの場合、exitした時に読み込む

*$ZDOTDIR* が未設定の場合は *$HOME* が使われるらしい。

メモ
=====

上記の通りなんだけど、9termでzsh起動するとzshrcが呼ばれない。
呼ばれているけど `unset zle_bracketed_paste` が効かない...
