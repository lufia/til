========================
WiXでインストーラを作る
========================

アップグレード
==============

WiX 3からは、``<MajorUpgrade>`` が使えるのでこれで良い。

.. code-block:: xml

	<?xml version="1.0" encoding="utf-8"?>
	<?define CurrentVersion="1.1.0"?>
	<Wix xmlns="http://schemas.microsoft.com/wix/2006/wi" xmlns:util="http://schemas.microsoft.com/wix/UtilExtension">
	  <Product Id="*"
						 Name="サーバ"
						 Version="$(var.CurrentVersion)"
						 Manufacturer="Lufia.org"
						 Language="1041"
						 Codepage="932"
						 UpgradeCode="E6A4DF6E-EC69-436b-917A-E875AC8F15F8">
	    <Package Id="*"
							 Description="便利サーバ"
							 InstallerVersion="200"
							 Compressed="yes"
							 Manufacturer="Lufia.org"
							 Languages="1041"
							 SummaryCodepage="932"/>
	    <MajorUpgrade
	      AllowDowngrades="no"
	      AllowSameVersionUpgrades="no"
	      IgnoreRemoveFailure="no"
	      DowngradeErrorMessage="新しいバージョンの[ProductName]がインストールされています。"
	      Schedule="afterInstallInitialize"/>
	...

``<RemoveExistingProducts>`` を設定しないと

	unresolved reference to symbol WixAction:InstallExecuteSequence/RemoveExistingProducts

のようなエラーになる、と書かれている記事もあるが、
``Schedule`` 属性と競合するので逆に書いてはいけない。

また、アップグレード可能にするためには、以下の条件に一致しておく必要がある。

* ``<Product>`` の ``Id`` 属性が異なる
* ``<Product>`` の ``Version`` 属性が異なる
* ``<Product>`` の ``UpgradeCode`` が同じ
