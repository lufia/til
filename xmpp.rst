============
XMPP(Jabber)
============

.. highlight: xml

手元でのデバッグ方法
====================

.. code-block: console

FCM-XMPP版をテストする場合は、``openssl s_client`` で接続する::

	$ openssl s_client -connect fcm-xmpp.googleapis.com:5235

接続できたら、以下のテキストを順番に書き込んで応答を待つ::

	<stream:stream to="fcm-xmpp.googleapis.com"
		version="1.0" xmlns="jabber:client"
		xmlns:stream="http://etherx.jabber.org/streams">

認証が必要なので資格情報を送る::

	<auth mechanism="PLAIN"
		xmlns="urn:ietf:params:xml:ns:xmpp-sasl">(認証コード)</auth>

認証が成功したらまた開始::

	<stream:stream to="fcm-xmpp.googleapis.com"
		version="1.0" xmlns="jabber:client"
		xmlns:stream="http://etherx.jabber.org/streams">

メッセージ送信::

	<message id="m-1"><gcm xmlns="google:mobile:data">テスト</gcm></message>

送るものが終わればタグを閉じる::

	</stream:stream>

.. code-block: go

認証コードは以下のコードで作れる::

	package main

	import (
		"encoding/base64"
		"flag"
		"fmt"
	)

	var (
		flagSender = flag.String("sender", "", "sender ID")
		flagKey    = flag.String("key", "", "server key")
	)

	func main() {
		flag.Parse()
		raw := "\x00" + *flagSender + "@gcm.googleapis.com\x00" + *flagKey
		s := base64.StdEncoding.EncodeToString([]byte(raw))
		fmt.Println(s)
	}
