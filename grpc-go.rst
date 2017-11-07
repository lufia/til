gRPC/Go
=======

.. highlight: go

ロードバランス
--------------

DNSリゾルバと ``grpc.WithBalancer`` を使うと良いらしい。
`Go+Microservices at Mercari <https://talks.godoc.org/github.com/tcnksm/talks/2017/11/gocon2017/gocon2017.slide>`_ によると::

	resolver, _ := naming.NewDNSResolverWithFreq(1 * time.Second)
	balancer := grpc.RoundRobin(resolver)
	conn, _ := grpc.DialContext(context.Background(), host, grpc.WithBalancer(balancer))

Middleware
----------

gRPCでは、HTTPのミドルウェアに該当するものはInterceptorという。
StackdriverクライアントライブラリはInterceptorを提供している。

Headers
-------

HTTPヘッダのようなものを扱う場合はMetadataを使う。
例えばメソッドのエラーでは大した情報を載せられないので、
詳細なエラー情報をクライアントに返したい、など。
