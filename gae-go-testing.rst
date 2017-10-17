Google App Engine/Goのテスト
============================

.. highlight:: go

GAE/Goのテストに関する情報。

GAE/Goでechoを使ったハンドラのテスト
------------------------------------

**echo** の `Testing <https://echo.labstack.com/guide/testing>`_  ガイドでは
``net/http/httptest.NewRequest`` を使ってリクエストを生成しているが、
``appengine.NewContext`` は ``appengine/aetest.Instance.NewRequest`` によって
作成したリクエストでなければ:

	| appengine: NewContext passed an unknown http.Request

というエラーで ``context.Context`` の取得に失敗する。

テストをするための具体的なコードは以下のようになる::

	instance, err := aetest.NewInstance(nil)
	if err != nil {
		t.Fatal(err)
	}
	defer instance.Close()

	r, err := instance.NewRequest(echo.POST, "/?q=1", bytes.NewReader(...))
	if err != nil {
		t.Fatal(err)
	}
	rec := httptest.NewRecorder()
	e := echo.New()
	c := e.NewContext(r, rec)
	if err := handler(c); err != nil {
		t.Errorf("handler() = %v", err)
	}
	if rec.Code != http.StatusOK {
		t.Errorf("handler() = %v; want %v", rec.Code, http.StatusOK)
	}

このテストコードは **go test** で実行してもエラーになるだけなので、
ビルド制約によって制限した方が望ましい::

	// +build appengine

	package app
