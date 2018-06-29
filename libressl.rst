=========
LibreSSL
=========

ポータブル版のビルド
====================

準備
-------

1. 依存しているOpenBSDのソースコードを *openbsd* 以下に取得
2. *openbsd* 以下のヘッダファイルをコピー
	* *include*
	* *include/openssl*
	* *crypto*
3. *openbsd/src/lib/libcrypto/\*/asm* 以下のperlコードを実行
	* perlコードは *flavor(elf, macosx)* と出力ファイル名を受け取る
	* *crypto/\*/xxx-$flavor-$arch.S に実コード生成
	* x86_64だけの様子
4. *openbsd/src/usr.bin/xx* のコードを *apps/xx* などにコピー
	* nc
	* openssl
5. テストコードを *tests* にコピー
6. *patches/\*.patch* を ``patch -p0``
7. ``autoreconf``
8. *scripts/config.xx* をリポジトリルートにコピー
9. ``./configure``

Plan 9に移植するには、

* ``autogen.sh`` を *rc* に移植
* ``update.sh`` を *rc* に移植
* 生成したMakefileをmkfileに移植(configureが動くとは思えない)
* 各種ソースコードの修正

あたりが必要か。
