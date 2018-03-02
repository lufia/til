=======================
Visual Studioでデプロイ
=======================

環境構築
========

Windows Server 2016でCAPI2 513エラーが記録される
------------------------------------------------

Windows Serverバックアップを実行すると記録される場合の対応。

* https://support.microsoft.com/ja-jp/help/3193007

Web Deploy環境の構築
--------------------

1. サーバの役割で以下をインストール
    * Webサーバ(IIS)
        * 管理ツール
            * 管理サービス
2. IISマネージャの **管理サービス** でユーザを追加
3. IISマネージャの **管理サービスの委任** で以下を有効化
    * setAd
    * contentPath, iisApp
    * createApp
4. IISマネージャで **Web配置による発行の有効化** を設定
5. WindowsファイアウォールでTCP/8172を開く

参考リンク

* [Web DeployでIISに発行するためのWindows Serverの設定](http://katsuyuzu.hatenablog.jp/entry/2016/01/26/001735)
