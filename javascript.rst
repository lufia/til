==================
JavaScriptとかWeb
==================

.. highlight:: js

最近の書き方についていけてないので少しずつ。

Fetch API
==========

* `window.fetch polyfill <https://github.com/github/fetch>`_

Cookieを送る
-------------

デフォルトではCookieを含めないリクエストを行う。
含めるためには ``credentials`` で指定する。

omit
	送らない

same-origin
	同一オリジンの場合は含める

include
	常に含める

コード例::

	fetch('/details', {
		credentials: 'same-origin'
	})
