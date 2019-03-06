==========
SSH鍵管理
==========

**ssh_config** は ``include`` 可能になったので、プロジェクト単位で
ファイルを分けておくと便利。

ただし最終的には1つのコンフィグに合成されるだけなので、
ホスト名などは重複しないように自分で気をつけないといけない。

.ssh/config
-----------

一般的な設定を行う::

	Include conf.d/*.conf

	host *
		ServerAliveInterval 15
		#AddKeysToAgent yes
		UseKeychain yes
		#ForwardAgent yes
		#IdentitiesOnly yes

*Include* はブロック内で使うとブロックに取り込まれるので先頭に書いた方が良い。

* `ssh_configのIncludeでハマった話 <https://tech.innovator.jp.net/entry/2018/05/24/143654>`_

.ssh/conf.d/\*.conf
-------------------

プロジェクト毎の設定を行う::

	Host project1-*
		UserKnownHostFile ~/.ssh/conf.d/project1/known_hosts
		IdentityFile ~/.ssh/id_rsa

	Host project1-host1
		HostName 192.168.1.xx

	Match OriginalHost project1-*, Host 192.168.1.*
		ProxyCommand ssh -W %h:%p project1-bastion

秘密鍵など関連するファイルは別のディレクトリに入れておくと扱いやすい。
