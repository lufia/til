=====================
GAE/Goのデータストア
=====================

.. highlight:: go

雑多
=====

Filterするキーは **プロパティ名**
----------------------------------

以下のような構造体をデータストアに書き込む場合::

	type Data struct {
		A string `datastore:"a_field"`
		B string
	}

``Filter`` や ``Order`` で参照する名前はこのようになる::

	datastore.Query("Data").Filter("a_field").Order("B")

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

memcache
=========

.. code-block:: go

JSONとGobは標準で用意されている::

	var a []string
	_, err := memcache.JSON.Get(c, "key", &a)

	err := memcache.JSON.Set(c, &memcache.Item{
		Key: "key",
		Object: a,
	})
