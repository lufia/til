Scoold関連
==========

Paraとの関係
------------

Scooldはフロントエンドサービスで、Paraはバックエンド全般を受け持つサービス。
ストレージ、認証などはすべてParaが持っている機能を呼び出して使う。
ParaとScooldで同じ設定キーを使うため、どのキーがフロントで設定するべきものか
分かりづらいけれど、とりあえずはScooldのREADMEに書いている項目が、
Scoold側で定義するべき設定だと判断すれば良さそう。

LDAP認証情報のやり取り
----------------------

LDAPの設定は、Scoold側の設定ファイルで行う。
最初にScooldが起動した時、バックエンドのParaへ接続するが、
この時に ``PUT /v1/_settings`` を使ってLDAPの設定内容をParaに連携する。

* scoold/src/main/java/com/erudika/scoold/ScooldServer.java
	* ``paraClientBean()``
* para/para-client/src/main/java/com/erudika/para/client/ParaClient.java
	* ``setAppSettings()``
	* ``invokePut()`` でバックエンドにPUT
* para/para-server/src/main/java/com/erudika/para/rest/Api1.java
	* ``appSettingsHandler(null)``
	* ``app.setSettings()`` または ``app.addSettings()``

ここの ``paraClientBean()`` は、``@Bean`` が付いているので、
必要になった時にインジェクトされる。

次に、ログインが発生した時、ユーザ名とパスワードを ``POST /jwt_auth`` する。
コードとしては以下の順で処理が動く。

* scoold/src/main/java/com/erudika/scoold/controllers/SigninController.java
	* ブラウザから */signin* へリクエスト
	* ``signinPost()``
	* ``getAuth()``
* para/para-client/src/main/java/com/erudika/para/client/ParaClient.java
	* ``signIn()``
	* ``invokePost()`` でバックエンドにPOST
* para/para-server/src/main/java/com/erudika/para/security/SecurityModule.java
	* ``getJWTAuthFilter()`` (これは事前に呼ばれている気がする)
* para/para-server/src/main/java/com/erudika/para/security/JWTRestfulAuthFilter.java
	* ``setLdapAuth()`` (これも事前に呼ばれている気がする)
	* ``doFilter()``
	* ``newTokenHandler()``
	* ``getOrCreateUser()``
* para/para-server/src/main/java/com/erudika/para/security/filters/LdapAuthFilter.java
	* ``getOrCreateUser(App, String)``
* para/para-server/src/main/java/com/erudika/para/security/LDAPAuthentication.java
	* ``withApp()``
* para/para-server/src/main/java/com/erudika/para/security/LDAPAuthenticationProvider.java
	* ``authenticate()``
* para/para-server/src/main/java/com/erudika/para/security/LDAPAuthentication.java
	* ``getLdapSettings()``
* para/para-server/src/main/java/com/erudika/para/security/SecurityUtils.java
	* ``getLdapSettingsForApp()``
	* ``app.getSettings()`` で接続時に ``app.setSettings()`` した値が取れる？
* para/para-server/src/main/java/com/erudika/para/security/LDAPAuthenticator.java
	* ``[spring]LdapAuthenticationProvider(LDAPAuthenticator(ldapSettings))``
	* ``authenticate()``
