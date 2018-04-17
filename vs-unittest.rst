=========================
Visual Studioの単体テスト
=========================

.. highlight:: C#

LocalDBを使う
=============

1. 新規追加で、*サービスベースのデータベース(mdf)* を追加する
2. Visual Studioのサーバエクスプローラでmdfファイルに接続する
3. mdfファイルとldfファイルの *ビルドアクション* を *コンテンツ* に変更
4. 同じように *出力ディレクトリにコピー* を *常にコピーする* に変更

テストプロジェクトの場合、このままではコピーされないので、``DeploymentItemAttribute`` を付けておく必要がある::

	[TestMethod]
	[DeploymentItem("TestDatabase.mdf")]
	[DeploymentItem("TestDatabase_log.ldf")]
	public void TestMethod1()
	{
		var repository = new GatewayRepository(0, TimeSpan.FromSeconds(100.0));
		var messages = repository.FindActiveMessages();
		Assert.AreEqual(1, messages.Count);
	}

.. code-block:: xml

接続文字列を *App.config* または *Web.config* で定義している場合は、
テストプロジェクトの *App.config* に接続文字列を用意しておく必要がある::

	<configuration>
		<connectionStrings>
			<add name="TestDBEntities"
				connectionString="metadata=res://*/Models.TestModel.csdl|res://*/Models.TestModel.ssdl|res://*/Models.TestModel.msl;provider=System.Data.SqlClient;provider connection string=&quot;data source=(LocalDB)\MSSQLLocalDB;AttachDBFilename=|DataDirectory|\TestDatabase.mdf;integrated security=True;App=EntityFramework&quot;"
				providerName="System.Data.EntityClient"/>
		</connectionStrings>
	</configuration>

接続文字列の最初にある ``metadata=`` は、Entity Data Model(edmx)ファイルの名前らしい。
通常は、テスト対象プロジェクトに含まれているはず。

また、テストプロジェクトでは ``|DataDirectory|`` が定義されていないので、
テストの実行前に自分で設定が必要::

	[ClassInitialize]
	public static void ClassInitialize(TestContext context)
	{
		AppDomain.CurrentDomain.SetData("DataDirectory", context.TestDeploymentDir);
	}
