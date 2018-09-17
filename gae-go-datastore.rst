=====================
GAE/Goのデータストア
=====================

雑多
=====

トランザクション中でPutMultiするにはオプションが必要
----------------------------------------------------

親キーを持たないエンティティに対して ``PutMulti`` すると、

	API error 1 (datastore_v3: BAD_REQUEST): cross-groups transaction need to be explicitly specified (xg=True)

というエラーになる。親キーを持たないエンティティの場合、
それぞれ独立したグループになるため、オプションが必要だった。

PutMultiで書き込むグループ数が多いとエラーになる
------------------------------------------------

独立した25個以上のエンティティグループを、
一度の ``PutMulti`` で書き込もうとした場合に以下のエラーが出力される

	API error 1 (datastore_v3: BAD_REQUEST): operating on too many entity groups in a single transaction.

ドキュメントによると、

	トランザクションがアクセスするすべてのデータは、最大 25 のエンティティ グループに制限されます。

とあるので、親キーを付けるように設計するか、小分けにするしかなさそう。

* `トランザクション <https://cloud.google.com/datastore/docs/concepts/transactions>`_
