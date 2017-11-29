Goにコントリビュートする
========================

基本的には `Contribution Guide <https://golang.org/doc/contribute.html>`_ の通りに進めていけばできる。

用語
----

Gerritの用語らしい。

CLs
	change lists

	https://godoc.org/golang.org/x/build/cmd/cl
CR
	code review

DNR
	do not review

RT
	run trybot

TR
	trybot result

ソースコードはgo getで取得する
------------------------------

Go公式のソースコードはgo.googlesource.comで扱うが、
リポジトリのURLを調べるのが面倒なので ``go get -d xxx`` で取得すると良い。
``go get`` するとリポジトリのURLはgo.googlesource.comを参照している。

コミットの名前は間違わないように
--------------------------------

*CONTRIBUTORS* ファイルはコミットの氏名とメールアドレスから生成される。
なので、コミットする時の名前は注意して設定する必要がある。

Forkする必要はない
------------------

直接 *master* に対して変更を加えて、一通り終わったら変更をステージに入れる。
その状態で(コミットせずに) ``git codereview change <branch-name>`` すると、
ブランチが作成されて、ステージの内容がブランチにコミットされる。

もし不備があるなら、その状態で変更を加えてステージに入れ、
再度 ``git codereview change`` すればよい。この時はブランチ名を省略する。

一通りテストも終わったら、``git codereview mail`` でサーバへ送る。
これはmailというコマンドだが、実際は ``git push`` と同じことをしているだけ。
