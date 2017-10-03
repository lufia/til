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
