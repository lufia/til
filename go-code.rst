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
