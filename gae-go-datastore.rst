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

クエリカーソルはLimitまで
-------------------------

例えば以下のようなクエリ::

	q := datastore.NewQuery("User").Limit(10)

このクエリから10件取得終わると、``Iterator.Next`` は ``datastore.Done`` を返す。
全てのデータでページングしたい場合は ``Limit`` を設定しない方が良い。

新しいフィールドはクエリできない
---------------------------------

構造体にフィールドを追加した場合::

	type User struct {
		ID   int
		Name string
		Paid bool // 追加
	}

データストアに以前から存在していたエンティティは、
追加したフィールドをインデックスしていないので、
クエリの ``Filter`` で検索することができない。

Goのコードではゼロ値が入っているので、

1. Go側でフィルタする
2. 過去のデータを新しいモデルで上書きして使う

* `GAE Datastore (Golang): Filter Query When Adding New DB Field <https://stackoverflow.com/questions/39104283/gae-datastore-golang-filter-query-when-adding-new-db-field>`_

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
