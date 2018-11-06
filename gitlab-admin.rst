==========
GitLab管理
==========

トラブル集
==========

BANされた場合の対応
--------------------

GitLabでは、``rack-attack`` というGemがBANを行なっていて、
デフォルトでは、ログインに10回連続失敗するとIPアドレスを1時間BANする。

*config.yml* でホワイトリストや回数などを設定できる。

.. code-block:: yaml

	production: &base
	  rack_attack:
	    git_basic_auth:
	      enabled: true
	      ip_whitelist: [127.0.0.1]
	      maxretry: 10
	      findtime: 60
	      bantime: 3600

ブロックされてしまったけれど1時間も待てない場合は、
Redisに保存されている値を直接削除すれば良い。

.. code-block:: console

	$ docker exec -ti xxx bash
	redis# redis-cli
	127.0.0.1:6379> keys *rack*
	...
	3) "cache:gitlab:rack::attack:allow2ban:ban:42.125.229.42"
	...
	127.0.0.1:6379> del "cache:gitlab:rack::attack:allow2ban:ban:42.125.229.42"

* `Remove banned IP from GitLab <https://medium.com/@Nomadic.UA/13067bf91707>`_

運用・管理
===========

シークレットの管理
-------------------

sameersbn/docker-gitlab の場合は、**gitlab-secrets** という名前の
シークレットがあれば起動時に ``source /run/secrets/gitlab-secrets`` で読み込む。
なので ``docker secret create gitlab-secrets xxx`` で環境変数を追加すれば良い。
