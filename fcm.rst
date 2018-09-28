========================
Firebase Cloud Messaging
========================

.. highlight:: http

FCM HTTP v1 API
===============

認証
-------

通知送信のための認証は、サーバキーでの認証からOAuth2へ変更になった。

* `送信リクエストを承認する <https://firebase.google.com/docs/cloud-messaging/auth-server>`_

スコープに ``https://www.googleapis.com/auth/firebase.messaging`` が必要。

エラー
-------

エラーが発生した場合のステータスコードやレスポンスがよくわからないので調べた。
``error.details[]`` の中に ``@type`` が *type.googleapis.com/google.firebase.fcm.v1.FcmError* のオブジェクトがあれば、その ``error.details[].errorCode``がエラーコードで、見つからない場合は ``error.status`` がエラーコード。

実際に試してみた結果を貼る。

正常な場合::

	HTTP/2.0 200 OK
	Date: Fri, 28 Sep 2018 05:36:37 GMT
	Content-Type: application/json; charset=UTF-8

	{
	  "name": "projects/fenrir-boltzmessenger-dev/messages/0:123456%abcdef123456"
	}

スコープを指定しない場合(``error`` として返却)::

	Post https://fcm.googleapis.com/v1/projects/(project_id)/messages:send: oauth2: cannot fetch token: 400 Bad Request
	Response: {
	  "error": "invalid_scope",
	  "error_description": "Empty or missing scope not allowed."
	}

スコープ間違い::

	HTTP/2.0 403 Forbidden
	Content-Type: application/json; charset=UTF-8
	Date: Fri, 28 Sep 2018 06:41:41 GMT

	{
	  "error": {
	    "code": 403,
	    "message": "Request had insufficient authentication scopes.",
	    "status": "PERMISSION_DENIED"
	  }
	}

認証エラーの場合::

	HTTP/2.0 403 Forbidden
	Content-Type: application/json; charset=UTF-8
	Date: Fri, 28 Sep 2018 06:21:37 GMT

	{
	  "error": {
	    "code": 403,
	    "message": "Firebase Cloud Messaging API has not been used in project 00000 before or it is disabled. Enable it by visiting https://console.developers.google.com/apis/api/fcm.googleapis.com/overview?project=00000 then retry. If you enabled this API recently, wait a few minutes for the action to propagate to our systems and retry.",
	    "status": "PERMISSION_DENIED",
	    "details": [
	      {
	        "@type": "type.googleapis.com/google.rpc.Help",
	        "links": [
	          {
	            "description": "Google developers console API activation",
	            "url": "https://console.developers.google.com/apis/api/fcm.googleapis.com/overview?project=00000"
	          }
	        ]
	      }
	    ]
	  }
	}


リクエストエラーの場合::

	HTTP/2.0 400 Bad Request
	Content-Type: application/json; charset=UTF-8
	Date: Fri, 28 Sep 2018 01:13:46 GMT

	{
	  "error": {
	    "code": 400,
	    "message": "Request contains an invalid argument.",
	    "status": "INVALID_ARGUMENT",
	    "details": [
	      {
	        "@type": "type.googleapis.com/google.rpc.BadRequest",
	        "fieldViolations": [
	          {
	            "field": "message",
	            "description": "Target of the message is not set."
	          }
	        ]
	      },
	      {
	        "@type": "type.googleapis.com/google.firebase.fcm.v1.FcmError",
	        "errorCode": "INVALID_ARGUMENT"
	      }
	    ]
	  }
	}

JSONが不正な場合::

	HTTP/2.0 400 Bad Request
	Content-Type: application/json; charset=UTF-8
	Date: Fri, 28 Sep 2018 05:30:09 GMT

	{
	  "error": {
	    "code": 400,
	    "message": "Invalid JSON payload received. Unknown name \"android\": Cannot find field.",
	    "status": "INVALID_ARGUMENT",
	    "details": [
	      {
	        "@type": "type.googleapis.com/google.rpc.BadRequest",
	        "fieldViolations": [
	          {
	            "description": "Invalid JSON payload received. Unknown name \"android\": Cannot find field."
	          }
	        ]
	      }
	    ]
	  }
	}

無効なトークン::

	HTTP/2.0 404 Not Found
	Content-Type: application/json; charset=UTF-8
	Date: Fri, 28 Sep 2018 07:27:19 GMT

	{
	  "error": {
	    "code": 404,
	    "message": "Requested entity was not found.",
	    "status": "NOT_FOUND",
	    "details": [
	      {
	        "@type": "type.googleapis.com/google.firebase.fcm.v1.FcmError",
	        "errorCode": "UNREGISTERED"
	      }
	    ]
	  }
	}

* `ErrorCode <https://firebase.google.com/docs/reference/fcm/rest/v1/ErrorCode>`_

Canonical Registration ID
-------------------------

FCMサーバからCanonical registration IDが返却されることはなくなった。
古い登録IDが更新される場合に限り、アプリ側で ``onTokenRefresh`` を受けてサーバの値を更新する。

* `Firebase Cloud Messaging - Are GCM canonical IDs still necessary? <https://stackoverflow.com/questions/41687344/firebase-cloud-messaging-are-gcm-canonical-ids-still-necessary/>`_
* `Retrieve FCM canonical_id in v1 API <https://stackoverflow.com/questions/48542261/retrieve-fcm-canonical-id-in-v1-api>`_
* `Instance ID <https://developers.google.com/instance-id/>`_

Message
-----------

汎用データとプラットフォーム固有のデータに別れている。
実際に届くものは、両方をマージした結果。

* `What's new with FCM? Customizing messages across platforms! <https://firebase.googleblog.com/2017/11/whats-new-with-fcm-customizing-messages.html>`_
