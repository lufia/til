=======
gRPC/Go
=======

.. highlight: go

ロードバランス
==============

DNSリゾルバと ``grpc.WithBalancer`` を使うと良いらしい。
`Go+Microservices at Mercari <https://talks.godoc.org/github.com/tcnksm/talks/2017/11/gocon2017/gocon2017.slide>`_ によると::

	resolver, _ := naming.NewDNSResolverWithFreq(1 * time.Second)
	balancer := grpc.RoundRobin(resolver)
	conn, _ := grpc.DialContext(context.Background(), host, grpc.WithBalancer(balancer))

Middleware
==========

gRPCでは、HTTPのミドルウェアに該当するものはInterceptorという。
StackdriverクライアントライブラリはInterceptorを提供している。

Headers
=======

HTTPヘッダのようなものを扱う場合はMetadataを使う。
例えばメソッドのエラーでは大した情報を載せられないので、
詳細なエラー情報をクライアントに返したい、など。

Channelz
========

接続状態の診断ができたりする。
gRPCの接続状態は以下のどれかに該当する。

CONNECTING
	TCP接続確率やTLSハンドシェイクなどを行なっている状態

READY
	gRPCとして確立した状態

TRANSIENT_FAILURE
	エラーやタイムアウトなどでリトライする状態

IDLE
	何も接続していない状態

SHUTDOWN
	シャットダウンを行なっている状態

* `gRPCのChannelzを使ってみた <https://qiita.com/kazegusuri/items/0945f9d805edfb6958ad>`_
* `gRPC Connectivity Semantics and API <https://github.com/grpc/grpc/blob/master/doc/connectivity-semantics-and-api.md>`_

リンク
======

* `goのgRPCで便利ツールを使う <https://qiita.com/h3_poteto/items/3a39c41743b4fd87c134>`_
* `Protocol Buffers v3でフィールドのrequiredという仕様が削除された話 <https://qiita.com/qsona/items/22c5c987d431552bbfe0>`_
* `HTTP/2における双方向通信とgRPCとこれから <https://qiita.com/namusyaka/items/71cf27fd3242adbf348c>`_
* `gRPC in Production <https://about.sourcegraph.com/go/grpc-in-production-alan-shreve/>`_
* `gRPCからREST API Serverをつくる <https://fisproject.jp/2018/09/translates-grpc-into-rest-json-api-with-go/>`_
