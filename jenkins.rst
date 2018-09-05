========
Jenkins
========

.. highlight:: groovy

構築関係
=========

Dockerで作る記事
-----------------

過去に書いた記事。

* `iOSビルド環境をJenkinsとDockerとAnsibleでコード化する(実際のコード付き) <https://blog.fenrir-inc.com/jp/2017/07/jenkins-build-app.html>`_

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
* `How to find JNLP Node's secret key remotely? <https://support.cloudbees.com/hc/en-us/articles/222520647>`_

手動でスレーブ追加がつらい
--------------------------

せっかくDocker使うならスレーブもDockerで構築したい。
その場合、Docker pluginを使うと実現できる。

Docker関連でよく名前を聞くプラグインは2つあり、

* Docker pipeline plugin (ID: docker-workflow)
* Docker plugin (ID: docker-plugin)

Docker pipeline pluginは、*Jenkinsfile* で ``docker { ... }`` ディレクティブを
記述するためのプラグイン。Jenkins 2.xなら標準でインストールされているはず。
これは、特定のラベルが付いた(dockerコマンドを実行可能な)ノードで、
指定したイメージを使ってビルドを行うもの。
例えば以下のように書くと、指定したイメージ上で後続の処理を実行することができる。

.. code-block:: groovy

	pipeline {
		agent {
			 docker {
				image 'golang:1.11'
			}
		}
	}

ただし、Docker pipeline pluginはDockerコンテナの中で実行させることはできるが、
スレーブを動的に追加することはできない。

Docker pluginは、設定しておいたラベルのノードが要求された時に、
事前に設定しておいたノードのテンプレートを使ってコンテナを実行するもの。
Docker plugin自体は特にスレーブの実行を想定しているわけではない。
本来はビルドに必要なラベルとイメージを複数用意しておき、
ビルドの際にラベルを使ってDockerイメージを選ぶものだと思うが、
管理者でなければラベル(=イメージ)を用意できないのは嬉しくない。

それよりもDocker pipeline pluginの自由度は好ましいので、
Docker pluginを使ってスレーブを作成し、スレーブが *Jenkinsfile* の実行を行い、
必要な場合は *Jenkinsfile* で ``docker { ... }`` ディレクティブを使って
コンテナを構築する方法が良いと思う。

Docker pipe
-----------

Dockerを使えるSlaveイメージ
----------------------------

スレーブはJenkinsコミュニティが `jenkins/jnlp-slave <https://hub.docker.com/r/jenkins/jnlp-slave/>`_ を用意してくれているので、
このイメージをベースに ``docker`` コマンドを追加したイメージを作る。

ここではJNLP版を使うが、SSHスレーブもある。

Dockerfile::

	FROM	jenkins/jnlp-slave:latest

	ARG	url=https://download.docker.com/linux/debian

	USER	root
	RUN	apt-get update && \
		apt-get install -y \
			apt-transport-https \
			ca-certificates \
			curl \
			gnupg2 \
			software-properties-common && \
		curl -fsSL $url/gpg | apt-key add - && \
		add-apt-repository "deb [arch=amd64] $url $(lsb_release -cs) stable" && \
		apt-get update && \
		apt-get install -y docker-ce && \
		rm -rf /var/lib/apt/lists/*

	USER	${user}

これでイメージを作ってプライベートリポジトリなどへpushしておく。

Docker pluginの設定
--------------------

上記のイメージでコンテナを実行するように設定する。
Docker pluginの構成は大きく2つに分けられる。

* Dockerホストと繋ぐための情報
* 構築するコンテナのテンプレート

Docker pluginをインストールすると、Jenkinsの管理/システムの設定、に
クラウドカテゴリが作られるので、Dockerを追加する。
設定が必要な項目を以下にまとめる。

まずはDockerホストとの接続設定。

Name
	なんでも良い

Docker Host URI
	Dockerデーモンが動作しているURL (例: *tcp://192.168.1.20:2375*)

Server credentials
	Dockerホストの認証情報

Enabled
	通常はチェックを入れる

Expose DOCKER_HOST
	Docker Host URIをコンテナの中でも設定するかどうか

	Docker pluginだけで使う場合はチェック不要だが、
	今回はスレーブからコンテナを起動するため必要。

Container Cap
	このホストで同時にいくつのコンテナを実行できるか

次にテンプレートの設定。

Labels
	このラベルでノードを要求された時にコンテナ作成される

Enabled
	通常はチェックを入れる

Name
	スレーブの名前(プリフィックス)

Docker Image
	上記で作成したdockerコマンド入りのスレーブイメージ名

Registry authentication
	レジストリアクセスの時に認証が必要なら設定する

Container settings
	テンプレートで生成するコンテナの設定を必要なら行う

Instance Capacity (Deprecated)
	必要なら

Remote File System Root
	/home/jenkins/agent

	*jenkins/jnlp-slave* により ``VOLUME`` 設定されている

Connect method
	今回はJNLPイメージを使うため **Connect with JNLP** を選択

JNLPの設定。基本的には初期値のまま。

User
	空でよい

Jenkins URL
	Jenkinsマスターが動作しているURL

	起動したスレーブが接続するために必要

Internal data directory
	remoting

これで完成。*docker* ラベルでノードが要求されたら、
Docker pluginがテンプレートからスレーブを作成してビルドが行われる。

Pipeline Model Definition
--------------------------

無くても問題ないが、**Jenkinsの管理/システムの設定** で、
Docker pipeline pluginのラベルとプライベートレジストリURLを
デフォルトとして設定しておくと *Jenkinsfile* の記述を省略できて親切。

Docker Label
	docker (Docker pluginのラベルと揃えておく)

Docker registry URL
	必要なら設定

Registry credentials
	必要なら設定

最終的なGroovyファイル
-----------------------

Docker pipeline pluginの設定::

	import jenkins.model.Jenkins
	import org.jenkinsci.plugins.pipeline.modeldefinition.config.GlobalConfig
	import org.jenkinsci.plugins.docker.commons.credentials.DockerRegistryEndpoint

	def dockerLabel = 'docker'
	def registryUrl = 'https://registry.a.fnrr.biz/v2/'
	def credentialId = null

	def p = Jenkins.instance.getDescriptorByType(GlobalConfig.class)
	p?.dockerLabel = dockerLabel
	p?.registry = new DockerRegistryEndpoint(registryUrl, credentialId)

Docker pluginの設定::

	import com.nirima.jenkins.plugins.docker.DockerCloud
	import com.nirima.jenkins.plugins.docker.DockerTemplate
	import com.nirima.jenkins.plugins.docker.DockerTemplateBase
	import com.nirima.jenkins.plugins.docker.launcher.AttachedDockerComputerLauncher
	import io.jenkins.docker.connector.DockerComputerAttachConnector
	import io.jenkins.docker.connector.DockerComputerJNLPConnector
	import jenkins.model.Jenkins
	import hudson.slaves.JNLPLauncher
	import jenkins.slaves.RemotingWorkDirSettings

	def nodeProperties = [
	  'docker-node1': [
	    serverUrl: 'tcp://192.168.1.30:2375',
	    instanceCapStr: '5',
	    containerCapStr: '5',
	    exposeDockerHost: true,
	  ],
	]

	def defaultProperties = [
	  // Template base parameters
	  bindAllPorts: false,
	  bindPorts: '',
	  cpuShares: null,
	  dnsString: '',
	  dockerCommand: '',
	  environmentsString: '',
	  extraHostsString: '',
	  hostname: '',
	  image: 'my.repo.example.org/jenkins-slave:latest',
	  macAddress: '',
	  memoryLimit: null,
	  memorySwap: null,
	  shmSize: null,
	  network: '',
	  privileged: false,
	  pullCredentialsId: '',
	  tty: false,
	  volumesFromString: '',
	  volumesString: '',

	  // Template parameters
	  instanceCapStr: '5',
	  labelString: 'docker',
	  remoteFs: '/home/jenkins/agent',
	  removeVolumes: true,
	  //disabled: false,

	  // Cloud parameters
	  connectTimeout: 60,
	  containerCapStr: '5',
	  credentialsId: '',
	  dockerHostname: '',
	  //nameはnodePropertiesで必須とする
	  readTimeout: 60,
	  serverUrl: 'unix:///var/run/docker.sock',
	  version: '',
	  exposeDockerHost: false,
	  //disabled: false,

	  // Connector parameters
	  tunnel: '',
	  jvmArgs: '',
	  user: '',
	  jenkinsUrl: 'https://jenkins.example.org/',
	  entryPointArguments: [],
	]

	nodeProperties.each { name, p ->
	  def existentCloud = Jenkins.instance.clouds.getByName(name)
	  if (existentCloud != null)
	    Jenkins.instance.clouds.remove(existentCloud)
	  def templateBase = new DockerTemplateBase(
	    p.image ?: defaultProperties.image,
	    p.pullCredentialsId ?: defaultProperties.pullCredentialsId,
	    p.dnsString ?: defaultProperties.dnsString,
	    p.network ?: defaultProperties.network,
	    p.dockerCommand ?: defaultProperties.dockerCommand,
	    p.volumesString ?: defaultProperties.volumesString,
	    p.volumesFromString ?: defaultProperties.volumesFromString,
	    p.environmentsString ?: defaultProperties.environmentsString,
	    p.hostname ?: defaultProperties.hostname,
	    p.memoryLimit ?: defaultProperties.memoryLimit,
	    p.memorySwap ?: defaultProperties.memorySwap,
	    p.cpuShares ?: defaultProperties.cpuShares,
	    p.shmSize ?: defaultProperties.shmSize,
	    p.bindPorts ?: defaultProperties.bindPorts,
	    p.bindAllPorts ?: defaultProperties.bindAllPorts,
	    p.privileged ?: defaultProperties.privileged,
	    p.tty ?: defaultProperties.tty,
	    p.macAddress ?: defaultProperties.macAddress,
	    p.extraHostsString ?: defaultProperties.extraHostsString
	  )

	  def connector = new DockerComputerJNLPConnector(
	    new JNLPLauncher(
	      p.tunnel ?: defaultProperties.tunnel,
	      p.jvmArgs ?: defaultProperties.jvmArgs,
	      new RemotingWorkDirSettings()
	    )
	  )
	  connector.user = p.user ?: defaultProperties.user
	  connector.jenkinsUrl = p.jenkinsUrl ?: defaultProperties.jenkinsUrl
	  connector.entryPointArguments = p.entryPointArguments ?: defaultProperties.entryPointArguments
	  def template = new DockerTemplate(
	    templateBase,
	    connector,
	    p.labelString ?: defaultProperties.labelString,
	    p.remoteFs ?: defaultProperties.remoteFs,
	    p.instanceCapStr ?: defaultProperties.instanceCapStr
	  )

	  def cloud = new DockerCloud(
	    name,
	    [template],
	    p.serverUrl,
	    p.containerCapStr ?: defaultProperties.containerCapStr,
	    p.connectTimeout ?: defaultProperties.connectTimeout,
	    p.readTimeout ?: defaultProperties.readTimeout,
	    p.credentialsId ?: defaultProperties.credentialsId,
	    p.version ?: defaultProperties.version,
	    p.dockerHostname ?: defaultProperties.dockerHostname
	  )
	  cloud.exposeDockerHost = p.exposeDockerHost
	  Jenkins.instance.clouds.add(cloud)
	}
	Jenkins.instance.save()

AWS S3にアップロード
====================

``aws s3`` コマンドはプロファイルを使う方法など複数の認証方法に対応しているが、
JenkinsからS3にアップロードする場合は環境変数を使う方法が一番簡単。

* aws-credentials プラグインをインストールしておく
* Jenkinsの管理画面から資格情報を登録する(例: aws_user)

これでJenkinsfileから参照できるようになる::

	withCredentials([[
		$class: 'AmazonWebServicesCredentialsBinding',
		credentialsId: 'aws_user',
		accessKeyVariable: 'AWS_ACCESS_KEY_ID',
		secretKeyVariable: 'AWS_SECRET_ACCESS_KEY'
	]]){
		...
	}

トラブル対応
=============

ログインできなくなった場合
--------------------------

*$JENKINS_HOME/config.xml* の ``useSecurity`` を ``false`` に変更すればよい。

.. code-block:: bash

	# Linux(GNU sed)の場合
	sed -i '/useSecurity/s/true/false/' config.xml
