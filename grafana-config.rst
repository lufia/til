Grafanaの設定
=============

.. highlight:: toml

LDAPのキーは大文字小文字を区別する
----------------------------------

LDAPの属性名は、デフォルトでは大文字小文字を区別しないが、
Grafanaの *ldap.toml* においては、異なる文字を使うとマッチしない。

例えば *ldap.toml* は::

	[servers.attributes]
	name = "givenName"
	surname = "sn"
	username = "cn"
	member_of = "memberOf"
	email =  "mail"

と記述されている場合、LDAPのDNが ``CN=`` の場合にはマッチしない。

この場合、2回目のログイン時に:

	| error="UNIQUE constraint failed: user.email"

というエラーが出力される。
これは、最初のログイン時に ``username`` がLDAPのエントリにマッチせず、
結果、GrafanaのDBには ``login = ''`` として登録される。
次に同じユーザでログインすると、DBには該当するユーザが存在しないので
登録を行おうとするが、メールアドレスが重複しているためエラーになる。
