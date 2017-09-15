OpenSSLのオプション
===================

秘密鍵に設定しているパスフレーズを外す場合、
インターネットで調べると以下のコマンドを見かけるが:: console

	$ openssl rsa -in key.pem -out nopass-key.pem

標準出力に出力したいなら、
単純に ``-out`` オプションを付けなければいい:: console

	$ openssl rsa -in key.pem
