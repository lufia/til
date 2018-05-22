========
環境構築
========

.. highlight:: console

ルール
======

* カスタマイズは環境を移った場合に困るのでなるべく少なく
* 基本的にはApple標準ツールを使う

自分で追加するしかないツールは、
*GOPATH* に合わせて以下のディレクトリに入れる。
なるべくホームディレクトリを汚したくないので、
設定可能な場合はなるべく以下のどこかに置く。

*~/bin*
	``drawterm`` など実行コマンド類

*~/pkg*
	``google-cloud-sdk`` などのパッケージ

	設定が可能な場合は、``npm`` や ``cocoapods`` もこの場所に入れる

*~/lib*
	``gitconfig`` や ``bash_profile`` など設定ファイル

*~/src*
	各種ソースコード

基本
====

トラックパッド関連
------------------

右手の人差し指を使ってタップしているけど、
トラックパッドを押し込んだ時に、ポイント位置が少しずれてしまうことがある。
左右の親指を使えば安定して押し込めるけれど、煩わしいので、
タップするだけでクリックとして扱うように変更する。

1. 環境設定を開く
2. トラックパッドを開く
3. *ポイントとクリック* タブを選択
4. **タップでクリック** にチェックを入れる

また、トラックパッドを押し込んだままドラッグするのは、
広い範囲を移動する時に何度も押し込み直す必要があって面倒なので、
ダブルタップでドラッグを維持するようにオプションを変更する。

1. 環境設定を開く
2. アクセシビリティを開く
3. マウスとトラックパッドを選択
4. トラックパッドオプションを開く
5. **ドラッグを有効にする** にチェックを入れて、**ドラッグロックなし** に変更

キーボード
----------

個人的には *Caps Lock* を使わないので *Control* に置き換える。

1. 環境設定を開く
2. キーボードを開く
3. *キーボード* タブを選んで *装飾キー* ボタンを押す
4. *Apple内蔵キーボード* の設定で **Caps Lock** を **Control** に変更

コンピュータ名の変更
--------------------

1. 環境設定を開く
2. 共有を開く
3. コンピュータ名を変更

iCloud
-------

あまりいい思い出がないけれど、以下の項目を同期させる。

* Safari
* キーチェーン

メールもメモも安定しないのでGoogleを使う。

開発ツール
----------

今後、``git`` や ``cc`` が必要なので *Command Line Tools* を入れる::

	$ xcode-select --install

SSH鍵
------

バックアップから復元しない場合は、鍵を作り直す::

	$ ssh-keygen -t ecdsa -b 384

これをGitHub等に登録しておく。

``ssh`` のデフォルトではKeychainを使わないので、*~/.ssh/config* に以下を追加::

	Host github.com
		Hostname github.com
		User git
		IdentityFile ~/.ssh/id_ecdsa
		AddKeysToAgent yes
		UseKeychain yes

dotfiles
--------

GitHubからdotfilesリポジトリをcloneして、必要な場所にリンクを張る::

	$ git clone git@github.com:lufia/dotfiles
	$ cd dotfiles
	$ ln -s $(relpath)/dot.inputrc ~/.inputrc

ハードリンクを使うと ``git pull`` などでリンクが切れる。

Plan 9環境
==========

Plan 9関連ツールを入れる。

Drawterm
--------

`drawterm-cocoa <https://bitbucket.org/jas/drawterm-cocoa>`_ を使う。

*Command Line Tools* を使うために *Make.osx-cocoa* を一部修正する。

.. code-block:: diff

	--- Make.osx-cocoa.orig	2017-12-03 04:24:00.000000000 +0900
	+++ Make.osx-cocoa	2018-04-24 22:11:45.000000000 +0900
	@@ -11,6 +11,8 @@
	 
	 SDK1011=-isysroot /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX10.11.sdk -mmacosx-version-min=10.7
	 
	+SDK1013=-isysroot /Library/Developer/CommandLineTools/SDKs/MacOSX10.13.sdk -mmacosx-version-min=10.7
	+
	 SDK10=-isysroot /Applications/Xcode.app/Contents/Developer/Platforms/MacOSX.platform/Developer/SDKs/MacOSX.sdk -mmacosx-version-min=10.7
	 
	 ARCHFLAGS64=-arch x86_64 -m64
	@@ -22,7 +24,7 @@
	 # any prior versions, just keep the SDK as defined.
	 ASFLAGS=
	 ARCHFLAGS=
	-SDK=$(SDK10)
	+SDK=$(SDK1013)
	 
	 PTHREAD=-lpthread	# for Mac

これでビルドする::

	$ make 'CONF=osx-cocoa'

Plan 9 from User Space
----------------------

.. todo:: 入れる

開発環境
========

Go
-----

ソースからコンパイルするためにバイナリパッケージを展開する::

	$ curl https://dl.google.com/go/go1.9.6.darwin-amd64.tar.gz | tar x
	$ mv go ~/go1.9

Goのソースを取得してビルド::

	$ git clone https://go.googlesource.com/go
	$ cd go/src
	$ GOROOT_BOOTSTRAP=~/go1.9 ./all.bash

Google Cloud SDK
-----------------

パッケージ管理ツールを使わなくても ``gcloud components`` で十分だった。
なのでzip版をダウンロードして、*~/pkg/google-cloud-sdk* に展開::

	$ mv google-cloud-sdk ~/pkg
	$ xattr -rd com.apple.quarantine ~/pkg/google-cloud-sdk

その後で必要なものを入れる::

	$ gcloud components update
	$ gcloud components install app-engine-go

Node.js
-------

最近のツールはNode.js製のものが増えたので `nvm <https://github.com/creationix/nvm>`_ を使って入れる::

	$ cd ~/pkg
	$ git clone https://github.com/creationix/nvm
	$ cd nvm
	$ git checkout $version # git tagで新しいバージョンを選ぶ

``nvm`` でNode.jsを入れる::

	$ nvm install node # 最新版; --ltsでLTS版が入る
	$ nvm alias default node # 最新を常に使う

これで、nodeは *$NVM_DIR/versions/node* 以下に入る。
以後 ``npm`` でインストールしたものは、
*$NVM_DIR/versions/node/$VERSION/lib/node_modules* や
*$NVM_DIR/versions/node/$VERSION/bin* 以下に置かれる。

nvm環境では、*~/.npmrc* で ``prefix`` を設定するとエラーになる。
*NPM_PACKAGES* や *NODE_PATH* 環境変数も不要。

``nvm`` の他に、``nodebrew`` や ``n`` というツールがあるが、
その中では ``nvm`` が良さそうだった。

アプリケーション
================

* AppCleaner
* Google Drive File Stream

Things 3
--------

デフォルトでは、Ctl+Spaceがクイック入力に割り当てられていて、
US配列のキーボードで日本語入力の切り替えキーと競合する。

* Things Cloud設定
* クイック入力ショートカット設定
