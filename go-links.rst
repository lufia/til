==============
Goの参考リンク
==============

AA
==

Gopher::

	ʕ ◔ϖ◔ʔ

* `Gopherくん入門 <http://write.kogus.org/articles/S78LHt>`_

言語仕様
========

* `[翻訳]Go言語の構文がC言語から大胆に変わった理由 <https://qiita.com/hachi8833/items/7c43a93130fcce3e308f>`_
* `Goにおける文字列、バイト、ルーンと文字 <https://www.ymotongpoo.com/works/goblog-ja/post/strings/>`_
* `配列、スライス（と文字列）：’append’の動作原理 <https://www.ymotongpoo.com/works/goblog-ja/post/slices/>`_
* `Goのスライスの内部実装 <http://jxck.hatenablog.com/entry/golang-slice-internals>`_
* `How the Go runtime implements maps efficiently (without generics) <https://dave.cheney.net/2018/05/29/how-the-go-runtime-implements-maps-efficiently-without-generics>`_
* `Go Range Loop Internals <https://garbagecollected.org/2017/02/22/go-range-loop-internals/>`_
* `Issue 9 <https://github.com/golang/go/issues/9>`_

コンパイラ
==========

* `GoのSSA最適化制御オプション <https://qiita.com/tooru/items/a55bcdac0500d9a93f39>`_
* `GoのInterfaceとは何者なのか <http://niconegoto.hatenadiary.jp/entry/2017/12/03/222922>`_
* `Assembly <https://goroutines.com/asm>`_
* `Go functions in assembly language <https://github.com/golang/go/files/447163/GoFunctionsInAssembly.pdf>`_
* `The Plan 9 Assembler Handbook <https://taimen.jp/f/324>`_
* `Introduction to the Compiler <https://github.com/golang/go/blob/master/src/cmd/compile/README.md>`_
* `Go 2にむけて <https://www.ymotongpoo.com/works/goblog-ja/post/toward-go2/>`_

ツール類
========

* `go tool trace <https://making.pusher.com/go-tool-trace/>`_
* `go tool traceでgoroutineの実行状況を可視化する <http://yuroyoro.hatenablog.com/entry/2017/12/11/192341>`_
* `Go execution tracer <https://blog.gopheracademy.com/advent-2017/go-execution-tracer/>`_
* `x/tools <https://godoc.org/golang.org/x/tools/cmd/>`_
* `dep <https://godoc.org/github.com/golang/dep/cmd/dep>`_
* `golint <https://github.com/golang/lint>`_
* `goversion <https://godoc.org/rsc.io/goversion>`_
* delve デバッガ
* `Go fonts <https://blog.golang.org/go-fonts>`_
* `vgo <https://godoc.org/golang.org/x/vgo>`_
	* `Go & Versioning <https://research.swtch.com/vgo>`_

パッケージ
==========

ランタイム
----------

* `意外と知らないgoroutineのスケジューラーの挙動 <https://qiita.com/niconegoto/items/3952d3c53d00fccc363b>`_
* `【翻訳】goroutine の仕組み <http://sairoutine.hatenablog.com/entry/2017/12/02/182827>`_
* `GoConで発表してきたのでついでにruntime以下の知識をまとめていく <http://niconegoto.hatenadiary.jp/entry/2017/04/11/092810>`_
* `Goのスケジューラー実装とハマりポイント <https://talks.godoc.org/github.com/niconegoto/talks/concurrency.slide>`_
* `Golangのスケジューラあたりの話 <https://qiita.com/takc923/items/de68671ea889d8df6904>`_

GC
-----

* `GolangのGCを追う <https://deeeet.com/writing/2016/05/08/gogc-2016/>`_
* `Golangの新しいGCアルゴリズム Transaction Oriented Collector <https://deeeet.com/writing/2016/06/29/toc/>`_
* `mgc.go <https://golang.org/src/runtime/mgc.go>`_
* `Go言語のリアルタイムGC 理論と実践 <https://postd.cc/golangs-real-time-gc-in-theory-and-practice/>`_

コンテナ
--------

* `Go1.9から追加されたsync.Mapのパフォーマンス <https://tanksuzuki.com/entries/golang-syncmap/>`_

HTTP
----

* `The complete guide to Go net/http timeouts <https://blog.cloudflare.com/the-complete-guide-to-golang-net-http-timeouts/>`_
* `Goでnet/httpを使う時のこまごまとした注意 <https://qiita.com/ono_matope/items/60e96c01b43c64ed1d18>`_
* `Goのリバースプロキシーでレスポンスを書き換える <https://qiita.com/shibukawa/items/55f64d81ea6ac802dd15>`_

context
--------

* `なぜContextを構造体に含めてはいけないのか、またそれが許される状況について <https://qiita.com/sonatard/items/d97279086b24e588a82d>`_

golang.org/x
-------------

* `Goにおける言語とロケールのマッチング <https://www.ymotongpoo.com/works/goblog-ja/post/matchlang/>`_
* `Goでの文字列の正規化 <https://www.ymotongpoo.com/works/goblog-ja/post/normalization/>`_

コードの書き方
==============

良いコード
----------

* `パッケージ名(Package names) <https://www.ymotongpoo.com/works/goblog-ja/post/package-names/>`_

* `Go Proverbsを勉強がてら和訳して少し解説した <http://nametake-1009.hatenablog.com/entry/2016/12/11/203328>`_
* `understanding nil <https://speakerdeck.com/campoy/understanding-nil>`_
* `understanding the interface <https://speakerdeck.com/campoy/understanding-the-interface>`_
* `なぜContextを構造体に含めてはいけないのか、またそれが許される状況について <https://qiita.com/sonatard/items/d97279086b24e588a82d>`_
* `プロダクション環境でのベストプラクティス <https://qiita.com/umisama/items/c2a8db6c23db18dd5437>`_
* `6年間におけるGoのベストプラクティス <http://postd.cc/go-best-practices-2016/>`_
* `標準的なパッケージのレイアウト <http://allishackedoff.hatenablog.com/entry/2016/08/23/015016>`_
* `Building and using coverage-instrumented programs with Go <http://damien.lespiau.name/2017/05/building-and-using-coverage.html>`_
* `Don't just check errors, handle them gracefully <https://dave.cheney.net/2016/04/27/dont-just-check-errors-handle-them-gracefully>`_

コマンドの終了コードは、

0
	正常終了

1
	実行エラー

2
	間違った使い方や-helpなど、コマンド未実行でのエラー

128以上
	シグナルでの終了とか

* `Exit Codes With Special Meanings <http://tldp.org/LDP/abs/html/exitcodes.html>`_

パターン
--------

エラー関連

* `Errors are values <https://blog.golang.org/errors-are-values>`_
* `Error handling in Upspin <https://commandcenter.blogspot.jp/2017/12/error-handling-in-upspin.html>`_
* `Failure is your Domain <https://middlemost.com/failure-is-your-domain/>`_

オプション

* `Go言語のFunctional Option Pattern <https://qiita.com/weloan/items/56f1c7792088b5ede136>`_
* `Self-referential functions and the design of options <https://commandcenter.blogspot.jp/2014/01/self-referential-functions-and-design.html>`_

考え方
------

* `loggingについて話そう <https://qiita.com/methane/items/cedbf546ae2db8a63c3d>`_
* `C言語プログラミングの覚え書き(改訳) <http://d.hatena.ne.jp/takeda25/20141012/1413116114>`_
	* `Notes on Programming in C <http://doc.cat-v.org/bell_labs/pikestyle>`_

プロジェクトレイアウト
----------------------

* `Go Project Layout <https://medium.com/golang-learn/e5213cdcfaa2>`_
* `Goのパッケージ構成の失敗遍歴と現状確認 <https://medium.com/@timakin/fc6a4369337>`_
* `golang のレイヤ構造において、他のコードに影響なくインフラレイヤのデータソース実装を差し替えることは可能か? <http://pospome.hatenablog.com/entry/2017/11/24/163149>`_

テスト・デバッグ
----------------

* `Advanced Testing in Go <https://about.sourcegraph.com/go/advanced-testing-in-go/>`_
* `Diagnostics <https://golang.org/doc/diagnostics.html>`_
* `go tool trace <https://making.pusher.com/go-tool-trace/>`_
* `golangでパフォーマンスチューニングする際に気をつけるべきこと <https://mattn.kaoriya.net/software/lang/go/20161019124907.htm>`_
* `Building and using coverage-instrumented programs with Go <http://damien.lespiau.name/2017/05/building-and-using-coverage.html>`_
* `Go Fridayこぼれ話:非公開(unexported)な機能を使ったテスト <https://tech.mercari.com/entry/2018/08/08/080000>`_
* `Goのtestを理解する in 2018 <https://budougumi0617.github.io/2018/08/19/go-testing2018/>`_
* `Goでテストを書く(テストの実装パターン集) <https://qiita.com/atotto/items/f6b8c773264a3183a53c>`_

オプション
----------

* `Go のオプション引数で -v -v -v みたいに複数指定する方法 <http://tyru.hatenablog.com/entry/2017/12/09/013948>`_
* `Re: Goでコマンドライン引数と環境変数の両方からflagを設定したい <https://mattn.kaoriya.net/software/lang/go/20170609110526.htm>`_

埋め込み
--------

* `Choosing A Library to Embed Static Assets in Go <http://tech.townsourced.com/post/embedding-static-files-in-go/>`_

GAE/Go
======

プロジェクト
------------

* `実践的なGAE/Goの構成について <https://qiita.com/koki_cheese/items/216fe73caf958db34aa2>`_
* `再考 GAE/Goのプロジェクト構成 <https://qiita.com/ryutah/items/eff6a044c81c5ba109d0>`_

データストア
------------

* `GAE/Goで本番のDatastoreをローカル環境に持ってくる 2016 <https://qiita.com/aql/items/9754b23a7d23544b1c10>`_

タスクキュー
------------

* `GAEのTaskQueue(PushQueue)で、delayパッケージとHTTPの受け口(handler)を定義するのは何が違うのか? <http://pospome.hatenablog.com/entry/2017/12/17/182509>`_

その他
------

* `Automatic Stackdriver Tracing for gRPC <https://rakyll.org/grpc-trace/>`_
	* cloud.google.com/go/traceで送れるらしい
* `GAE/Goのurlfetchのタイムアウトを設定する <http://pospome.hatenablog.com/entry/2017/12/17/112144>`_

情報収集
========

* `The Go Blog <https://blog.glang.org/>`_
* `GopherAcademy <https://blog.gopheracademy.com/>`_
* `goz Go's News <http://goz.hexacosa.net/>`_
