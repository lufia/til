macOS High Sierraインストール関連
=================================

FileVault有効な場合インストールに時間がかかる
---------------------------------------------

暗号化を一旦解除してからAPFSへ変換しているからなのか、
100GBほど使用済みのHFS+ディスクを持ったMacBook Airは、
High Sierraインストールに2時間30分ほど必要だった。

クリーンインストール用ブータブルUSBを作る
-----------------------------------------

macOS High SierraのインストーラをApp Storeからダウンロードして、
インストーラが起動した(/Applications以下に **Install macOS High Sierra.app** が存在する)状態で、
USBメモリをマウントして、以下のコマンドを実行する。

.. code-block:: console

	$ cd /Applications/Install\ macOS\ High\ Sierra.app/Contents/Resources
	$ sudo createinstallmedia \
		--volume /Volumes/disk \
		--applicationpath /Applications/Install\ macOS\ High\ Sierra.app

途中で、USBメモリ(上記では/Volumes/disk)を消去して良いか確認されるので、
問題なければ **Y** と答えて続行する。
