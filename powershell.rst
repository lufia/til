PowerShell
==========

.. highlight:: powershell

引数を受け取る
--------------

``Param()`` で定義すれば受け取れるし、名前の補完もできるようになる::

	Param(
		[parameter(mandatory=$True)][string]$Path,
		[int]$Days = 60
	)

上記の *$Path* は必須パラメータで、*$Days* は省略されると60になる。

配列とオブジェクト
------------------

配列は以下のように ``@()`` を使って書く::

	$a = @(0, 1, 2)

オブジェクトは ``@{name=value}`` を使う::

	$p = @{key="Feb"; value=2}

PowerShellオブジェクトをJSONに変換する
--------------------------------------

``ConvertTo-Json`` を使う::

	$p = @{type=1; account="user1"}
	$json = ConvertTo-Json $p

デフォルトでは2階層までしか変換しない。
以下のオブジェクトがあった場合::

	$p = @{
		users=@(
			@{
				address=@{
					postal="000-1111";
					street="street";
				};
				name="user1";
			}
		)
	}

オプションを付けずに変換すると、``address`` の値が型の名前になる::

	ConvertTo-Json $p
	...snip...
		{
			"name":  "user1",
			"address":  "System.Collections.Hashtable"
		}

``-Depth`` を使うと任意の深さまで変換される::

	ConvertTo-Json -Depth 3 $p

JSONをPOSTすると文字化けする
----------------------------

以下のコードでは、日本語文字を含む場合に文字化けする::

	$body = @{name="テスト"}
	$json = ConvertTo-Json $body
	Invoke-RestMethod -Method POST -Uri http://localhost:8080/ -Body $json
	# {"name":"???"}のようになる

正しくはこちら::

	$body = @{name="テスト"}
	$json = ConvertTo-Json $body
	$bytes = [System.Text.Encoding]::UTF8.GetBytes($json)
	Invoke-RestMethod -Method POST -Uri http://localhost:8080/ -Body $bytes
