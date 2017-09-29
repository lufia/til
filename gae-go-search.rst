Google App Engine/GoのSearch API
================================

* `Documents and Indexes <https://cloud.google.com/appengine/docs/standard/go/search/>`_
* `Query and Sorting Options <https://cloud.google.com/appengine/docs/standard/python/search/options>`_

snippet関数のqueryにユーザ入力を使うのはよくない
------------------------------------------------

.. highlight:: go

以下のように書くとマッチした部分を抽出できるが::

	options := search.SearchOptions{
		Expressions: []search.FieldExpression{
			{
				Name: "Snippet",
				Expr: fmt.Sprintf(`snippet("%s", Body)`, expr),
			},
		},
	}
	t := x.Search(c, "Body="+expr, &options)

この ``expr`` に ``"`` を含んでしまった場合、Search APIは

	| search: INVALID_REQUEST: Failed to parse field expression 'snippet("テスト"", Body)': parse error at line 0 position -1

というエラーで動作しなくなってしまう。
``unicode.IsControl(c) || unicode.IsPunct(c)`` を削除すれば
多くの場合よくなるが、常に正しく扱えるわけではない。
`Escaping search queries for Google's full text search service <https://stackoverflow.com/questions/10741011/escaping-search-queries-for-googles-full-text-search-service>`_ に同様の議論があるけれど、解決しているようには見えない。
