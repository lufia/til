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

npm-scripts
	Gulpの代わりに使える

npm-scriptsでの設定例
=====================

*npm-scripts* はシェルスクリプトを実行するだけだが、
``npm-run-all`` 等で小さいタスクをまとめて実行できるようなので、
Gulpと比べて見通しの良いこちらを使うことにする。

ファイル構造は、自分で書いたコードは *src* に入れて、
変換したものを *dist* に出力する習慣らしい。

* src
	* app.js
* webpack.config.js
* package.json
* dist
	* main.js

* `Gulpの代わりにnpm-scriptsをタスクランナーとして使う <http://glatchdesign.com/blog/web/tools/1265>`_

package.json
------------

``npm init [-y]`` で作成する。*npmrc* で以下を設定しておくと便利::

	init.author.email = xxx@example.com
	init.author.name = xxx

``main`` はモジュールを ``require('xx')`` した時に参照されるもの。
``npm init`` を実行すると必ず設定させられるが、
作っているものがモジュールでなければ気にしなくてもいい。

``license`` はIDが色々ある。新しいBSDライセンスは ``BSD-3-Clause`` を使う。

* `npm package.json日本語版取扱説明書 <http://liberty-technology.biz/PublicItems/npm/package.json.html>`_

webpack
-------

*npm-scripts* から実行するため以下をインストールする。

.. code-block:: console

	$ npm install -D webpack webpack-cli

webpack自体は、JavaScriptファイルをバンドルするためのもので、
普通は ``loader`` を使って他のトランスパイラを呼び出すことが多い。

Babel
-----

``async`` やアロー関数など、新しめのJavaScript構文を使えるようにするもの。

.. code-block:: console

	$ npm install -D babel-core babel-loader babel-preset-env

*webpack.config.js* を作成::

	//export default {
	module.exports = {
	  mode: 'development',
	  entry: './src/app.js',
	  module: {
	    rules: [
	      {
	        test: /\.js$/,
	        use: [
	          {
	            loader: 'babel-loader',
	            options: {
	              presets: [
	                ['env', {'modules': false}]
	              ]
	            }
	          }
	        ]
	      }
	    ]
	  }
	}

*npm-scripts* から使えるようにする。

.. code-block:: json

	{
		"scripts": {
			"build": "webpack"
		}
	}

これで ``npm run build`` が使える。
*src/app.js* を適当に作ってビルドすると、*dist/main.js* が生成できる。

.. code-block:: console

	$ npm run build

* `BabelでES2018環境の構築(React, Vue, Three.js, jQueryのサンプル付き) <https://ics.media/entry/16028>`_

*webpack.config.js* では ``entry`` で1つだけファイルを選択しているけど、
複数のファイルがある場合はどうするんだろう。

PostCSS
--------

これも *webpack* から使う方が良さそう。
*postcss-cssnext* は *postcss-preset-env* に置き換えられた。

.. code-block:: console

	$ npm install -D style-loader css-loader postcss-loader \
		postcss-preset-env postcss-import

*webpack.config.js* にもルールを追加する。

.. code-block:: js

	//import postcssPresetEnv from 'postcss-preset-env'
	const postcssPresetEnv = require('postcss-preset-env')

	//export default {
	module.exports = {
	  devtool: 'source-map',
	  module: {
	    rules: [
	      {
	        test: /\.css$/,
	        use: [
	          'style-loader',
	          {
	            loader: 'css-loader',
	            options: {
	              sourceMap: true,
	              minimize: true,
	              importLoaders: 1
	            }
	          },
	          {
	            loader: 'postcss-loader',
	            options: {
	              ident: 'postcss',
	              sourceMap: true,
	              plugins: () => [
	                postcssPresetEnv()
	              ]
	            }
	          }
	        ]
	      }
	    ]
	  }
	}

*app.js* からCSSをロードする。

	import './app.css'

これでCSSも *dist/main.js* にバンドルされる。
``postcssPresetEnv()`` はサポートするブラウザバージョンなど、
色々なオプションが設定できる。オプションは公式のREADMEでOptionsを読めばいい。

* `postcss-preset-env <https://github.com/csstools/postcss-preset-env>`_

*npm-scripts* から直接使う場合はコマンドラインをインストールする。

.. code-block:: console

	$ npm install -D postcss-cli

* `スタイルシート(CSSやSass)を取り込む方法 <https://ics.media/entry/17376>`_

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
