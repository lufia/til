======================
Go関係の便利ツールなど
======================

* `Go Tools <https://gotools.org/>`_
* `Go Search <http://go-search.org/>`_
* `Go Issues <https://goissues.org/>`_
* `Go Changes <https://gochanges.org/>`_

JSON生成

* `quicktype <https://quicktype.io/>`_
* `JSON-to-Go <https://mholt.github.io/json-to-go/>`_

JavaScript

* `jsgo <https://play.jsgo.io/>`_
	* `DOMを扱うサンプル <https://play.jsgo.io/github.com/dave/jstest>`_
	* `画像圧縮サンプル <https://play.jsgo.io/github.com/dave/img>`_

SSHで ``go get`` する
---------------------

.. code-block:: console

以下のコマンドで、*https* アクセスした時に *ssh* へ切り替える::

	$ git config --global url.git@github.com:.insteadOf https://github.com/

.. code-block:: ini

これで、*~/.gitconfig* には以下のようなエントリが追加される::

	[url "git@github.com:"]
		insteadOf = https://github.com/

プライベートリポジトリを ``go get`` する
----------------------------------------

色々やってみたけどできなかった::

	$ git config --global http.extraheader 'PRIVATE-TOKEN: xxxxxxx'
	$ git config --global git@github.com:.insteadOf https://github.com/
	$ go get xxx # error
	$ go get xxx.git # ok

リポジトリ名に *.git* つければ動くけど、なんだろう。

スライドを作成する
------------------

* `Presentでスライドを作ろう <https://www.slideshare.net/YutakaKato/present-75952579>`_
