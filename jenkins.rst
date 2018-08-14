========
Jenkins
========

構築関係
=========

ワークスペースと成果物ディレクトリを移動
-----------------------------------------

Jenkinsシステム設定に「ワークスペースとビルドのルートディレクトリ」項目があり、
変更できるような記事があるけれど、
Dockerイメージ(jenkins/jenkins:lts)の2.121.2にはそのメニューが存在しなかった。

これらの設定は *$JENKINS_HOME/config.xml* に

.. code-block:: xml

	<?xml version='1.1' encoding='UTF-8'?>
	<hudson>
	  <workspaceDir>${JENKINS_HOME}/workspace/${ITEM_FULLNAME}</workspaceDir>
	  <buildsDir>${ITEM_ROOTDIR}/builds</buildsDir>
	</hudson>

として記述されているので、これを書き換えてもいいが、
javaのコマンドラインオプションで指定することができる。

.. code-block:: console

	$ java -Djenkins.model.Jenkins.buildsDir='${ITEM_ROOTDIR}/builds' \
		-Djenkins.model.Jenkins.workspaceDir='${JENKINS_HOME}/workspace/${ITEM_FULL_NAME}' -jar jenkins.war ...

ここで使えるパラメータと環境変数は以下の資料にまとめられている。

* `<https://wiki.jenkins.io/display/JENKINS/Features+controlled+by+system+properties>`_

Dockerイメージの場合は、環境変数 *JAVA_OPTS* と *JENKINS_OPTS* に
引数となるパラメータを設定しておくと、コンテナ実行時に以下の形で展開される。

.. code-block:: bash

	java -Duser.home=/var/jenkins_home $JAVA_OPTS -jar jenkins.war $JENKINS_OPTS

Dockerを使ってビルド
====================

マルチスレーブ
--------------

ビルドの時にDockerコンテナを使う。
プラグインは標準で入っているものを使えば良い。

1台で構成された環境の場合は、JenkinsマスターにDockerが入っていれば良いが、
マルチスレーブの場合は

1. スレーブを使う
2. スレーブ上でDockerコマンドを実行する

のような動きをするので、まずスレーブを作らなければならない。

Dockerコンテナでスレーブを実行
------------------------------

いくつか方法はあるが、公式に提供されているイメージを使う方法が簡単。

1. JNLPでスレーブを作成
  ノード名
    docker-slave1 (任意)
  リモートFSルート
    /home/jenkins/agent
  ラベル
    docker (任意)
2. 作成したスレーブのLaunchボタンから *.jnlp* ファイルを取得
3. *.jnlp* ファイルはXMLなのでエディタで開く
4. XMLの */jnlp/application-desc/argument* 要素からシークレットを取り出す
5. 次の ``docker`` コマンドでスレーブを実行する

.. code-block:: console

	docker run -d --name jenkins-slave jenkins/jnlp-slave -url (JENKINS_URL) \
		-workDir /home/jenkins/agent (シークレット) (ノード名)

リモートFSルートも任意だが、*jenkins/jnlp-slave* イメージによって
ボリュームとして定義されているので、変更しないほうが良いと思う。

* `jenkins/jnlp-slave <https://hub.docker.com/r/jenkins/jnlp-slave/>`_
* `Jenkins nodes on Docker containers <https://piotrminkowski.wordpress.com/2017/03/13/jenkins-nodes-on-docker-containers/>`_
