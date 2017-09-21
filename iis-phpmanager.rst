PHP Manager
===========

IISでPHPのバージョン等、設定を行えるツールらしい。
WebPIでインストールできる。

- `PHP Manager for IIS 7を試用 <http://blogs.gine2.jp/kusa/archives/14>`_

PHP Managerを開くとphp.iniが見つからないエラー
----------------------------------------------

PHP Managerを開いた直後に、*php.iniが見つかりません*
とエラーダイアログが表示され、全てのメニューが無効になっている。

この場合、IISコンソールから **FastCGI** を開いて、
PHPと名前のついているエントリを全て削除する。
削除後にPHP Managerを開くと、PHPが何も登録されていない状態になっているので、
**新規のPHPバージョンを登録する** メニューから、
例えば ``C:\Program Files (x86)\PHP\5.6\php-cgi.exe`` 等を選択すれば元に戻る。
