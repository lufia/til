macOS High Sierraインストール関連
=================================

クリーンインストール用ブータブルUSBを作る
-----------------------------------------

macOS High SierraのインストーラをApp Storeからダウンロードして、
インストーラが起動した(/Applications以下に **Install macOS High Sierra.app** が存在する)状態で、
USBメモリをマウントして、以下のコマンドを実行する。
インストール済みであっても、最新バージョンならApp Storeでダウンロードは可能。

.. code-block:: console

	$ cd /Applications/Install\ macOS\ High\ Sierra.app/Contents/Resources
	$ sudo createinstallmedia \
		--volume /Volumes/disk \
		--applicationpath /Applications/Install\ macOS\ High\ Sierra.app

途中で、USBメモリ(上記では/Volumes/disk)を消去して良いか確認されるので、
問題なければ **Y** と答えて続行する。

* `macOS の起動可能なインストーラを作成する方法 <https://support.apple.com/ja-jp/HT201372>`_
