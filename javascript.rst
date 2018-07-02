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
	将来の標準CSSを使えるようにトランスパイルする

	* automatic vendor prefixes
	* custom prpoerties, var
	* custom media queries
	* custom selectors

	webpackから使うけど、gulpからも使えるらしい

npm-scripts
	Gulpの代わりに使える

* `cssnextでみる次世代CSSとPostCSS <http://blog.yucchiy.com/2015/04/22/cssnext-postcss-for-nextgeneration-of-css/>`_

CSS命名規則
===========

過去にいくつか存在したらしいけど、今はこの2つを覚えておけば良さそう。

BEM
	Block-Element-Modifierを繋げて書く

FLOCSS
	Foundation, Layout, Objectでディレクトリも分けて書く

* `hiloki/flocss: CSS organization methodology <https://github.com/hiloki/flocss>`_
* `OOCSS, BEM, SMACSS, FLOCSS, RSCSSを比較して自分にあった設計思想をみつける <https://kuroeveryday.blogspot.com/2017/03/css-structure-and-rules.html>`_

Web Components
==============

CSSの影響をShadow DOMに隔離できるから良さそうなんだけど。

* `Web Componentsで近未来のフロントエンド開発 <https://nulab-inc.com/ja/blog/cacoo/web-components/>`_

最近はほとんどネイティブで動作するけど、
Custom ElementsとShadow DOMは、FirefoxとEdgeでPolyfillが必要。

* `webcomponents.org <https://www.webcomponents.org>`_
* `webcomponentsjs <https://github.com/webcomponents/webcomponentsjs>`_

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
2018年6月現在、Babel 7対応の ``babel-loader`` は8.0.0-beta.4なので、
なるべく新しいベータ版を探して入れる。

.. code-block:: console

	$ npm info babel-loader versions
	[ '4.0.0',
	  ...
	  '8.0.0-beta.4' ]
	$ npm install -D @babel/core @babel/preset-env babel-loader@8.0.0-beta.4

また、*webpack.config.js* をES6で書くために ``@babel/register`` も入れておく。
入れておくだけで *\*.babel.js* にマッチしたファイルをBabel経由で扱うため、
*webpack.config.babel.js* で ``import`` などの新しい構文が使えるようになる。

.. code-block:: console

	$ npm install -D @babel/register

*package.json* でBabelのプリセットを指定する。
本当は *.babelrc* に書くものだが、隠しファイルが増えると
見通し悪くなるので、*package.json* に書く方が好み。

.. code-block:: json

	{
	  "babel": {
	    "presets": ['@babel/preset-env']
	  }
	}

これを書いていない場合、*\*.babel.js* ファイルで ``import`` を使った時に、
以下のようなエラーになる。

	import xxx from 'xxx'
	       ^^^

	SyntaxError: Unexpected identifier

一通り準備ができたら、*webpack.config.babel.js* を作成::

	export default {
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
	              presets: ['@babel/preset-env']
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
* `Webpack with Babel 7 <https://medium.com/oredi/b61f7caa9565>`_

*webpack.config.js* では ``entry`` で1つだけファイルを選択しているけど、
複数のファイルがある場合はどうするんだろう。

PostCSS
--------

これも *webpack* から使う方が良さそう。
*postcss-cssnext* は *postcss-preset-env* に置き換えられた。
ES6の ``import`` 文が使えた方が便利なので ``babel-register`` も入れると良い。

.. code-block:: console

	$ npm install -D style-loader css-loader postcss-loader \
		postcss-preset-env postcss-import

*webpack.config.babel.js* にもルールを追加する。
``@babel/register`` を入れていない場合は、
コメントアウトしている方の書き方(ES5)しか使えない。

.. code-block:: js

	//const postcssPresetEnv = require('postcss-preset-env')
	import postcssPresetEnv from 'postcss-preset-env'

	//module.exports = {
	export default {
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
	                postcssPresetEnv({
	                  browsers: 'last 2 versions'
	                })
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

PostCSS(コマンドライン)
------------------------

*npm-scripts* から直接使う場合はコマンドラインをインストールする。

.. code-block:: console

	$ npm install -D postcss-cli

設定したい場合は、*postcss.config.js* を書けばいいらしい。

* `スタイルシート(CSSやSass)を取り込む方法 <https://ics.media/entry/17376>`_

HTML
----

HTMLも *src* 以下で管理し、webpackの対象にする。
以下どちらもwebpackのプラグイン。

html-webpack-plugin
	webpackで生成したJavaScriptをロードするための<script>タグを自動挿入する

script-ext-html-webpack-plugin
	<script>タグの属性(deferなど)をカスタマイズする

npmでインストールする。

.. code-block:: console

	$ npm install -D html-webpack-plugin script-ext-html-webpack-plugin

*webpack.config.babel.js* にプラグインを設定する::

	import HtmlWebpackPlugin from 'html-webpack-plugin'
	import ScriptExtHtmlWebpackPlugin from 'script-ext-html-webpack-plugin'

	export default {
	  module: {
	    ..
	  },
	  plugins: [
	    new HtmlWebpackPlugin({
	      template: 'src/index.html'
	    }),
	    new ScriptExtHtmlWebpackPlugin({
	      defaultAttribute: 'defer'
	    })
	  ]
	}

React
======

Reactのモジュールを追加。Babelを使っている場合はローダも追加。

.. code-block:: console

	$ npm install -D react react-dom
	$ npm install -D @babel/preset-react

*webpack.config.babel.js* の ``presets`` にReactの設定を追加::

	export default {
	  module: {
	    rules: [
	      {
	        test: /\.jsx?$/,
	        use: [
	          {
	            loader: 'babel-loader',
	            options: {
	              presets: ['@babel/preset-env', '@babel/preset-react']
	            }
	          }
	        ]
	      }
	    ]
	  },
	  resolve: {
	    extensions: ['.js', '.jsx']
	  }
	}

これであとは普通に書けばビルドできる::

	function hello() {
		let f = () => (<div>hello</div>)
		console.log(f())
	}

Context API
------------

* `Reactの新Context APIとRedux is deadはどう関係するのか？ <https://medium.com/@terrierscript/6d12a32f2f0c>`_

Redux
=======

.. code-block:: console

インストール。

	$ npm install -D redux react-redux

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

Thread
=======

Node.js 10.5.0から、worker_threadsが入ったらしい。

* `Node.jsにworkerが入った <http://blog.hiroppy.me/entry/worker_threads>`_
