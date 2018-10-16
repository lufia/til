===========
macOS調べ物
===========

.. highlight: console

Time Machine
============

コマンドライン
--------------

``tmutil`` でコマンドラインから操作できる。

* `Control Time Machine from the command line <https://www.macworld.com/article/2033804/control-time-machine-from-the-command-line.html>`_

古いバックアップを削除する::

	$ tmutil listbackups | grep 2017- | tr "\n" "\0" | sudo xargs -0 tmutil delete

途中で止まった場合
------------------

``mdutil`` でインデックスを削除すると動く場合がある::

	$ sudo mdutil -i off -a
	$ sudo mdutil -E -a
	$ sudo rm -rf /Volumes/xxx/.Spotlight-V100
	$ sudo mdutil -i on -a

* `Time Machineの不具合 <http://goroneko.la.coocan.jp/wordpress/?p=770>`_

雑多メモ
=========

TCC
------

権限関連のデータベースらしい

* tccutil

ファイルの実態はSQLiteのデータベース

* */Library/Application Support/com.apple.TCC/TCC.db*

ディレクトリ
------------

* dscl

ファイル属性
------------

* xattr
* `パーミッションの末尾+文字とは <https://tkamada.blogspot.com/2012/06/macos-xls-l.html>`_

脆弱性(VCE-ID)
==============

脆弱性対応された場合、`Appleセキュリティアップデート <https://support.apple.com/ja-jp/HT201222>`_ に詳細ページへのリンクが追加される。
詳細ページに、CVE-IDが付与されている脆弱性なら言及が行われる。
例えばHigh Sierraの *root* ログイン問題は、`macOS High Sierra 10.13.2 <https://support.apple.com/ja-jp/HT208331>`_ で言及が行われている。

.. code-block: text

このページのURLは以下の形式になっている::

	https://support.apple.com/en-us/HT208331
	https://support.apple.com/ja-jp/HT208331

``softwareupdate`` と ``plutil`` コマンドを使うと、
recommendedとマークされたアップデートのプロダクトキーを調べることができるが、
上記ページのID (HTxxxx) とプロダクトキーを繋げる方法がわからない。

``softwareupate`` の結果::

	$ softwareupdate --list
	Software Update Tool

	Finding available software
	Software Update found the following new or updated software:
	   * macOS 10.13.2 Update-10.13.2
	        macOS 10.13.2アップデート (10.13.2), 1624732K [recommended] [restart]

PlistBuddyの結果::

	$ /usr/libexec/PlistBuddy -c Print /Library/Preferences/com.apple.SoftwareUpdate.plist 
	Dict {
	    LastSuccessfulDate = Thu Dec 14 10:04:42 JST 2017
	    LastBackgroundCCDSuccessfulDate = Wed Sep 21 10:37:18 JST 2016
	    LastAttemptSystemVersion = 10.13.1 (17B1003)
	    SkipLocalCDN = false
	    LastUpdatesAvailable = 1
	    LastRecommendedUpdatesAvailable = 1
	    LastAttemptBuildVersion = 10.13.1 (17B1003)
	    RecommendedUpdates = Array {
	        Dict {
	            Identifier = macOS 10.13.2 Update
	            Product Key = 091-52053
	            Display Name = macOS 10.13.2アップデート
	            Display Version = 10.13.2
	        }
	    }
	    LastFullSuccessfulDate = Thu Dec 14 10:04:42 JST 2017
	    PrimaryLanguages = Array {
	        ja
	        ja-JP
	    }
	    LastSessionSuccessful = true
	    LastBackgroundSuccessfulDate = Thu Dec 14 09:28:27 JST 2017
	    LastResultCode = 0
	}
