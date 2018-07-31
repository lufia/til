Goの書き方関係
==============

.. highlight:: go

一般的な情報
=============

モジュールの作成
-----------------

1. 環境変数 *GO111MODULE* を *on* に設定
2. パッケージを ``go build`` すると *go.mod* が無ければ作られる
3. *go.mod* をコミット
4. セマンティックバージョニングでタグを付ける

*go.mod* と同時に *go.sum* が生成される場合もある。
これはモジュール(zipファイル)のハッシュ(SHA1?)と *go.mod* のハッシュを持つ。
*go.mod* を更新するために依存するモジュールの全ソースコードが必要になるのは無駄なので、
最適化を行うために *go.sum* は存在するらしい。

* `x/vgo: possible no-determinism in updating go.sum <https://github.com/golang/go/issues/26310>`_

GitHub では *.gitignore* に含んでいるリポジトリがいくつかあった。

モジュールの使い方
-------------------

Go 1.10 までと同じように、``import`` して ``go get`` を実行するだけ。

モジュールのアップデートは、``go get -u`` でマイナーバージョンのアップデート、
``go get -u=patch`` でパッチバージョンのアップデートを行う。

モジュールファイルはバージョンごとにzipでダウンロードされるため、
今までのパッケージと別になっていて、*$GOPATH/src/mod* 以下に展開される。
*$GOPATH/src/mod* の理由は、*$GOPATH* が *$HOME* の人もいるし、
*$GOPATH/pkg* を無くしたいから、らしい。

* `$GOPATH/src/mod <https://groups.google.com/d/topic/golang-dev/RjSj4bGSmsw/discussion>`_
* `cmd/go: drop $GOPATH/src <https://github.com/golang/go/issues/4719>`_

並行処理パターン
----------------

* for-select loop
* or-channel
* or-done-channel
* tee-channel
* fan-in
* fan-out

* `Go Concurrency Patterns: Pipelines and cancellation <https://blog.golang.org/pipelines>`_
* `or-done-channelでコードの可読性を上げる <http://ymotongpoo.hatenablog.com/entry/2017/12/04/091403>`_

具体的な書き方
===============

自分のIPアドレスを調べる
------------------------

以下のコードでループバックを除いて取得できる::

	func LocalIP() ([]net.IP, error) {
		ifaces, err := net.Interfaces()
		if err != nil {
			return nil, err
		}
		var ipaddrs []net.IP
		for _, iface := range ifaces {
			addrs, err := iface.Addrs()
			if err != nil {
				return nil, err
			}
			for _, addr := range addrs {
				ipnet, ok := addr.(*net.IPNet)
				if !ok || ipnet.IP.IsLoopback() {
					continue
				}
				if ipnet.IP.To4() == nil {
					continue
				}
				ipaddrs = append(ipaddrs, ipnet.IP)
			}
		}
		return ipaddrs, nil
	}

``net.IPNet`` はIPアドレスとネットマスクの構造体。

* `GoのnetパッケージにおけるIPアドレスの内部表現 <https://qiita.com/cubicdaiya/items/6441551467b91a160695>`_

ライセンス表記
--------------

Goの標準ライブラリや *golang.org/x* パッケージなどは修正BSDライセンス。

あまりよくわかっていないが、

* 標準ライブラリのコードをコピーする場合、そのファイルのヘッダにCopyright表記を含める
* 同一パッケージのうち、自分で書いたコードには不要
* Goのコードに手を加えた場合のCopyrightはGo Authorsとは別に追加する？

.. todo:: バイナリ配布の場合はどうする？

エクスポートしない型を返す
--------------------------

パッケージAで::

	package sample

	type result struct {
	}

	func New() *result {
		return &result{}
	}

別のパッケージから呼び出す::

	package main

	import (
		"fmt"

		"path/to/sample"
	)

	func main() {
		r := sample.New()
		fmt.Printf("%T\n", r)
	}

これは動作する。メンバー変数を参照させたいけど、
初期化は必ず特定の関数を通して行いたい場合に使える。
だけど ``golint`` では警告される。

	sample.go:7:12: exported func New returns unexported type *sample.result, which can be annoying to use

ゼロ値をうまく使うように努力した方が良い。

値のコピーを抑制する
--------------------

コピーを防ぎたい型に ``Lock()`` メソッドを実装すると、
値をコピーするコードが ``go vet`` で警告される。

* `Goの構造体のコピーを防止する方法 <https://shogo82148.github.io/blog/2018/05/16/macopy-is-struct/>`_

必ずメンバー名を使って初期化させる
----------------------------------

構造体の先頭にエクスポートしない型を置けば良い::

	package sample

	type unexported struct{}

	type Request struct {
		unexported
		URL  string
		Body string
	}

これを、名前を使わず初期化しようとすると::

	r := sample.Request{"http://example.com", "テスト"}

以下のようなエラーでビルドできない。

	implicit assignment of unexported field 'unexported' in sample.Request literal

名前付きで初期化すれば通る::

	r := sample.Request{URL: "http://example.org", Body: "テスト"}

``fmt.Println(r)`` すると最初の構造体が見えてしまって不恰好だけどたまに便利。

この記事をどこかで読んだ気がするけれど見失った。

関数オプション
--------------

エクスポートしないフィールドを明示的に初期化させたい場合のパターン。

* `Go言語のFunctional Option Pattern <https://qiita.com/weloan/items/56f1c7792088b5ede136>`_

flag
=====

カンマを配列にするオプション::

	type stringSlice []string
	
	func newStringSlice(val []string, p *[]string) *stringSlice {
		*p = val
		return (*stringSlice)(p)
	}
	
	func (a *stringSlice) Set(s string) error {
		v := strings.Split(s, ",")
		*a = stringSlice(v)
		return nil
	}
	
	func (a *stringSlice) Get() interface{} {
		return []string(*a)
	}
	
	func (a *stringSlice) String() string {
		return strings.Join([]string(*a), ",")
	}

	var slice []string
	func init() {
		flag.Var(newStringSlice([]string{"default"}, &slice), "a", "sample")
	}

net
=====

DNSサーバを指定する
-------------------

標準では、DNSサーバは */etc/resolv.conf* などから読み込み、それが使われる。
プログラムの中で *resolv.conf* ではないDNSサーバを参照したい場合は、
サーバを直接変更する方法は用意されていないので、
``net.Resolver`` の ``Dial`` を設定して無理やり向きを変える::

	func dial(ctx context.Context, network, address string) (net.Conn, error) {
		var d net.Dialer
		return d.DialContext(ctx, network, "8.8.8.8:53")
	}

	func main() {
		var resolver net.Resolver
		resolver.PreferGo = true
		resolver.Dial = dial
		addrs, err := resolver.LookupHost("www.google.com")
	}

``net.LookupXxx`` は ``net.DefaultResolver`` を参照するので、
他の動作に影響がなければ、``DefaultResolver.Dial`` を置き換えても良い。

net/http
========

MaxIdleConns, MaxIdleConnsPerHost
---------------------------------

MaxIdleConnsは、デフォルトでは100で、0にすると無制限となる。
これはホストにかかわらず全体で使い回されるコネクションの数。

MaxIdleConnsPerHostはホスト単位で使い回すコネクションの数。

これらの値を超える場合、
例えばMaxIdleConnsPerHost=1で2つ以上のリクエストを実行すると、
ブロックするのではなく新しく接続を行い、そのままリクエストを終える。
ただし、処理が終わった後は、MaxIdleConsPerHost=1なので、
1本だけ残してあとの接続は閉じる。

httptrace.GotConnで値を出力すると使い回されたかどうかがわかる。

net/http/httputil
------------------

新しい ``ReverseProxy`` は ``NewSingleHostReverseProxy`` を提供しているが、
これは以下のような *X-Forwarded-xx* に対応していない。

X-Forwarded-Proto
	クライアントがアクセスしてきたオリジナルのプロトコル(httpsなど)

X-Forwarded-Host
	クライアントがアクセスしてきたオリジナルのホスト名

X-Forwarded-Port
	クライアントがアクセスしてきたオリジナルのポート番号(あれば)

なので自分で ``ReverseProxy.Director`` を書くと良い::

	p := &httputil.ReverseProxy{
		Director: func(req *http.Request) {
			host, port, err := net.SplitHostPort(req.Host)
			if err != nil {
				// can't parse hostname: no port or many colons
				host = req.Host
			}
			req.Header.Set("X-Forwarded-Host", host)
			if port != "" {
				req.Header.Set("X-Forwarded-Port", "port")
			}
			if req.TLS != nil {
				req.Header.Set("X-Forwarded-Proto", "https")
			} else {
				req.Header.Set("X-Forwarded-Proto", "http")
			}

			// 以後はNewSingleHostReverseProxyそのまま
		},
	}

クラウドサービス
================

クラウドサービスAPIへの抽象化層が入ったので、なるべくこれを使う方が良さそう。
2018年7月現在はAWSとGCPのみ。

* `Go Cloud <https://godoc.org/github.com/google/go-cloud>`_

たまに使うと便利な関数
======================

runtime
-------

* SetFinalizer

reflect
-------

* DeepEqual
* Select

net/http/httptrace
------------------

net/httpの動作を調べるときに便利。
