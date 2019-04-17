## 実行時に影響するもの

これらはほとんど [runtime](https://golang.org/pkg/runtime/)でドキュメント化されている。

### GOGC

次のGCを発生させる条件をパーセンテージで指定する。GC完了後から確保したメモリサイズが、GC直後に残ったサイズと比べて、比率が `GOGC` で設定した割合に達したらGCを行う。デフォルトは `GOGC=100` がセットされている。

例えば、最後のGCで処理した後のサイズが10MBだとすると、次のGCはメモリ確保量が直前のGC時点のサイズの100%になった場合に発生する。

`GOGC=off` の場合はGCしない。

* [GolangのGCを追う](https://deeeet.com/writing/2016/05/08/gogc-2016/)

### GODEBUG

これはサブパラメータが色々ある。複数設定する場合は `GODEBUG=gctrace=1,cgocheck=1` のようにカンマで区切る。

#### allocfreetrace

`GODEBUG=allocfreetrace=1` すると、メモリのアロケーションが発生した時点の
スタックトレースが見られる。

* [Goでアロケーションに気をつけたコードを書く方法](http://dsas.blog.klab.org/archives/52191778.html)

#### cgocheck

cgoでCの関数へGoのポインタを渡した時のチェックを行うかどうか。デフォルトは `GODEBUG=cgocheck=1` で、Goのポインタを渡すとエラーになる。`GODEBUG=cgocheck=0` を設定するとチェックを抑制する。

`GODEBUG=cgocheck=2` とすると厳密にチェックするようになるが、代わりに遅くなる。

* [Go1.6でポインタをcgoの関数へ渡す際に発生するcgoCheckPointerを回避する方法](https://qiita.com/mattn/items/90c8558d5fff05a2ba0c)

#### cpu.extension=off

1.12より。

#### efence

`GODEBUG=efence=1` とすると、アロケータの動作を、全てのオブジェクトが個別のページに確保されて、かつリサイクルされないようなモードに切り替える。

#### gccheckmark

`GODEBUG=gccheckmark=1` に設定すると、GCの事後検証を有効にする。検証時は全てのスレッドが停止する。この検証では、到達可能であるはずなのに、GCでマークされなかったオブジェクトを探す。もし該当するオブジェクトが発見されたら、GCはpanicする。

#### gcpacertrace

`GODEBUG=gcpacertrace=1` で、コンカレントペーサーの内部状態をプリントする。

*コンカレントペーサー(concurrent pacer)とは何だろう。*

#### gcshrinkstackoff

`GODEBUG=gcshrinkstackoff=1` とすると、ゴルーチンの動作するスタックは現在のスタックよりも小さくならない。

おそらく、ランタイムの問題切り分けのために存在するオプションのように思う。普段は使わないのでは。

#### gcrescanstacks

`GODEBUG=gcrescanstacks=1` (enable)

#### gcstoptheworld

`GODEBUG=gcstoptheworld=1`に設定すると、GCはコンカレントなマークを無効化し、必ずスレッドを停止して(stop the world)オブジェクトのマークを行う。また、`GODEBUG=gcstoptheworld=2`にすると、スイープもコンカレントではなくなる。

#### gctrace

`GODEBUG=gctrace=1`に設定すると、GCが発生するたびに、標準エラー出力にGCの要約とメモリ解放の要約をプリントする。`GODEBUG=gctrace=2`もプリントされる内容は同じだが、2以上の場合は、通常バックグラウンドで行われるスイープを、GCのmarkterminationフェーズ(markフェーズの後)でも行うようになる。

GCのフェーズは

* off
* mark
* marktermination

の3種類存在する。スイープは通常、バックグラウンドで行われる。

#### madvdontneed

Linuxにメモリ管理のヒントを与える。デフォルトではMADV_FREEだけど `GODEBUG=madvdontneed=1` とするとMADV_DONTNEED

#### memprofilerate

`GODEBUG=memprofilerate=0` の場合はメモリのプリファイリングを行わない。1以上の値をセットすると、`runtime.MemProfileRate`にそのままセットされる。

#### invalidptr

`GODEBUG=invalidptr=1`は、不正なポインタ値が見つかったらクラッシュさせる。`GODEBUG=invalidptr=0`にするとクラッシュしなくなる。

通常はデフォルトの`1`で良い。

#### sbrk

`GODEBUG=sbrk=1`とすると、メモリアロケータとGCを変更する。

古い実装？使わない気がする。

#### scavenge

`GODEBUG=scavenge=1`とすると`runtime`のScavengerをデバッグモードにする。

#### scheddetail, schedtrace

`GODEBUG=scheddetail=1,schedtrace=X`に設定するとXミリ秒ごとに、スケジューラ、プロセッサ、スレッド、ゴルーチンの状態を出力する。

#### tracebackancestors

TODO

#### tls13=1

Go 1.12では、`GODEBUG=tls13=1` にするとTLS 1.3が有効になる。

#### x509ignoreCN

このパラメータは[crypto/x509](https://golang.org/pkg/crypto/x509/)パッケージで提供される。

#### netdns

このパラメータは[net](https://golang.org/pkg/net/)パッケージで提供される。

`GODEBUG=netdns=go` またはデフォルトは、Goで実装されたDNSリゾルバを使う。`GODEBUG=netdns=cgo` の場合はCの実装を使う。

数値の場合はデバッグ出力が有効になる。両方設定する場合は`netdns=cgo+1`と書く。

* [Goの通信経路選択](http://blog.restartr.com/2016/08/29/golang-networking/)

#### http2client, http2server

このパラメータは[net/http](https://golang.org/pkg/net/http/)パッケージで提供される。

クライアント側でHTTP/2を無効にする場合は `GODEBUG=http2client=0` とする。また、サーバ側は`GODEBUG=http2server=0`とする。

#### http2debug

このパラメータは[net/http](https://golang.org/pkg/net/http/)パッケージで提供される。

`GODEBUG=http2debug=1`とするとデバッグ出力が有効になる。`http2debug=2`の場合は、より詳細な出力がされる。

### GOMAXPROCS

プロセッサの数。

* [Golangのスケジューラあたりの話](https://qiita.com/takc923/items/de68671ea889d8df6904)

### GOTRACEBACK

Goのプログラムがpanicした場合やエラー終了した場合の出力形式を変更する。

|名前  |数値|意味                                              |
|------|----|--------------------------------------------------|
|none  |0   |ゴルーチンのスタックトレースを出力しない          |
|single|    |現在のゴルーチンスタックを簡易に出力する          |
|all   |1   |全てのゴルーチンスタックと詳細を出力する          |
|system|2   |ランタイムも含めて全てのスタックトレースを出力する|
|crash |    |OS固有の方法で終了？                              |

* [A whirlwind tour of Go’s runtime environment variables](https://dave.cheney.net/2015/11/29/a-whirlwind-tour-of-gos-runtime-environment-variables)
* [GOTRACEBACKのメモ](https://qiita.com/maaaato/items/a3e502a40c7d2b3a1e28)


## ビルド時に影響するもの

https://golang.org/cmd/go/

`go help environment` でも良い

### GOARCH

ビルド対象のアーキテクチャ名。

* amd64
* 386
* arm64
* arm

### GOOS

ビルド対象のOS名。

* linux
* windows
* darwin
* plan9

### GOPATH

Goワークスペースのルートになるディレクトリ。`:`で区切って複数書くこともできる。

### GOROOT

Goコンパイラなどツールチェインの場所。今は`go`のパスから自動的に決まるので、環境変数は設定しない方が良い。

### GCCGO

### GOBIN

### GORACE
### GOTMPDIR

### GOCACHE

ビルドキャッシュの保存先ディレクトリを設定する。
macOSの場合、デフォルトは *~/Library/Caches/go-build/* が使われる。

offの場合はキャッシュしない。

* [Command go](https://golang.org/cmd/go/)

### GODEBUG

#### gocacheverify
#### gocachehash
#### gocachetest

GOHOSTARCH
GOHOSTOS

GO386
GOARM
GOMIPS

GOROOT_FINAL
GO_EXTLINK_ENABLED

### GOSSAFUNC

SSA変換過程を表示したい関数名をセットするとHTMLで過程を出力する。
例えば`main`の過程を表示したい場合は`GOSSAFUNC=main go build`とする。

* [GoのSSA最適化制御オプション](https://qiita.com/tooru/items/a55bcdac0500d9a93f39)
* [Looking at your program’s structure in Go 1.7](https://pauladamsmith.com/blog/2016/08/go-1.7-ssa.html)
* [Debugging code generation in Go](https://rakyll.org/codegen/)

GOSSAHASH
	cmd/compile/internal/ssa/func.go
	DebugTest  bool
		default true unless $GOSSAHASH != ""
		as a debugging aid, make new code conditional on this and
		use GOSSAHASH to binary search for failing cases

GO_SSA_PHI_LOC_CUTOFF
	0: enables sparse for all >=0
	-1: disables sparse completely

## cgo

`cgo`関連の環境変数。

CC

### CGO_ENABLED

cgoを有効にする場合は`CGO_ENABLED=1`とする。
通常は有効だが、クロスコンパイル時のデフォルトは`0`なので環境変数が必要。

* [GoでCのライブラリを使ったプログラムをクロスコンパイルする](https://hatappi.hateblo.jp/entry/2017/11/23/204211)

CGO_LDFLAGS
CGO_xxxxx_ALLOW
CGO_xxxxx_DISALLOW

### Goのコンパイル時に使う

GOROOT_BOOTSTRAP
GO_GCFLAGS
GO_LDFLAGS
	Go自体のコンパイルフラグ
GOBOOTSTRAP_TOOLEXEC
	$GOROOT_BOOTSTRAP/bin/go install \
		-gcflags=-l ... -toolexec=$GOBOOTSTRAP_TOOLEXEC

### expiremental

GO19CONCURRENTCOMPILATION
	1: enabled(default)
	0: disabled

GO15VENDOREXPERIMENT
	1: enabled
	0: disabled

### 未分類(不明)

GCC
GO
GOBUILDTIMELOGFILE
GOBULDTIMELOGFILE
GOCLOBBERDEADHASH
GOEXPERIMENT
GOROOT_FINAL_OLD
GOTESTONLY
	plan9
	https://go.googlesource.com/build/+/20d848347d0f5efd48953b70cf0fc67eb1b0ada7/cmd/buildlet/buildlet.go
GO_CGOCHECKBYTES_TRY

### xxx

osusergoビルドタグ(os/user)
netgoビルドタグ(net)

### 環境変数ではないけどビルドフラグ

* go build -gcflags='-m -l'
* 下位レイヤーのコマンド(go tool compileなど)に渡すフラグ
* go testでも使える
* asmflagsなどもある

https://qiita.com/ryskiwt/items/574a07c6235735afa5d7
