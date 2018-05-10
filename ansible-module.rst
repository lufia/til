=================
Ansibleモジュール
=================


.. highlight:: python

* `Ansibleのモジュール開発(基礎編) <https://dev.classmethod.jp/server-side/ansible/ansible-develop-module-basis/>`_
* `Ansibleのモジュール開発(Python実装編) <https://dev.classmethod.jp/server-side/ansible/ansible-develop-module-python/>`_
* `初めてのAnsible(10章:カスタムモジュール) <https://qiita.com/c0tt0n-candy/items/33ef98d3c42f04ce7450>`_

モジュールのパラメータは辞書で定義する::

    module_args = dict(
		user = dict(type='str', required=True),
		repo = dict(type='str', required=True),
		token = dict(type='str', required=False, default=None, no_log=True),
		flatten = dict(type='bool', required=False, default=False)
	)
	module = AnsibleModule(
		argument_spec=module_args,
		supports_check_mode=True
	)

``type`` は、

* ``str``
* ``int``
* ``bool``

などがある。

パスワードなどログに残したくないものは ``no_log=True`` とする。
これらは、``module.params['user']`` のように参照できる。

モジュールの実行結果はいくつかのメソッドで行う。

``module.exit_json()``
	正常終了

``module.fail_json()``
	エラー終了

コマンド実行
------------

``module`` に ``run_command`` があるのでそれを使う::

	argv0 = module.get_bin_path('command', required=True)
	argv = [argv0, '-f', '/mnt/font/GoMono/14a/font']
	module.run_command(argv)

``get_bin_path()`` はデフォルトでは */sbin*, */usr/sbin*, */usr/local/sbin* しか検索しない。
サーチするPATHを追加したい場合、``opt_dirs`` でリストを与える::

	argv0 = module.get_bin_path('command', required=True, ['/usr/local/plan9/bin'])

``run_command`` で追加する場合は、``path_prefix=`` で文字列を与える::

	module.run_command(argv, path_prefix='/usr/local/plan9/bin')
