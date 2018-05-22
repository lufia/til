Goの書き方関係
==============

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
