==================
JavaScriptとかWeb
==================

.. highlight:: js

最近の書き方についていけてないので少しずつ。

開発ツール
==========

Gulp
	タスクランナー

webpack
	jsファイルをバンドルする

Babel
	ES6トランスパイラ

	webpackから使う

PostCSS
	CSS生成

	webpackから使うけど、gulpからも使えるらしい

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
