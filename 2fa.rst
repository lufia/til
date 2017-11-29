=========
2段階認証
=========

HMAC-Based One-Time Password(HOTP)
==================================

`RFC4226 <https://tools.ietf.org/html/rfc4226>`_ で規定。

Time-Based One-Time Password(TOTP)
==================================

`RFC6238 <https://tools.ietf.org/html/rfc6238>`_ で規定。

実装
====

* https://github.com/rsc/2fa
* http://buty4649.hatenablog.com/archive/2015/05/06

使い方
======

.. code-block: console

``go get`` でインストール::

	$ go get github.com/rsc/2fa

GitHubの場合は **text code** のリンクを開いて表示された文字列を、
GitLabの場合はQRコードの右にあるテキストから **Key:** の内容をコピーして、
``2fa -add`` コマンドの *2fa key for name:* プロンプトに貼り付ける::

	$ 2fa -add gitlab
	2fa key for gitlab: xxxx xxxx xxx xx (スペースを含んでも良い)

``2fa name`` を実行して乱数を生成、それを各サービスの管理画面にポスト::

	$ 2fa gitlab
	012345

あとは *~/.2fa* をどこかにバックアップしておけば良い。
