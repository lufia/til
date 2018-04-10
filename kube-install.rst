===========
kubernetes
===========

.. highlight:: yaml

概要
====

Container Linuxで動作させる場合は、以下の用意が必要。

* etcd
* flanneld
* マスターノード
	* kube-apiserver
	* kube-controller-manager
	* kube-scheduler
* ワーカーノード
	* kube-proxy
	* kubelet
* クライアント
	* kubectl

マスターノードはワーカーと同じでも良いが通常は分ける。

``kube-xxx`` はHyperkubeを使ってコンテナ内で組み立てるほうが良いらしい。

ネットワーク
------------

Kubernetes service IP
	``kube-apiserver`` の公開アドレス？

	この記事では **192.168.1.10** を使う

Pod network
	Podに割り当てられるネットワークのCIDR

	``kube-proxy`` の ``clusterCIDR`` と ``flanneld`` の *Network* はこれ

	この記事では **10.254.0.0/16** を使う

Service IP range
	KubernetesクラスタのネットワークCIDR

	これは既存ネットワークや、Pod networkの範囲と重なってはいけない

	``kube-apiserver`` の ``--service-cluster-ip-range`` はこれ

	この記事では **10.253.0.0/16** を使う

準備
======

Container LinuxはIgnition Configで初期化する。

ct
------

Container Linux Config trainspilerでYAMLをJSONに変換するツール。

.. code-block:: console

Nixpkgsでインストールするには::

	$ nix-env -i ct

.. code-block:: console

``ct`` でContainer Linux ConfigをIgnition Configに変換する::

	$ ct -in-file config.yaml -out-file ignition.json

.. code-block:: makefile

*Makefile* を作っておくと良い::

	TARG=config.iso
	DIR=img
	SSL=ssl
	KEYS=$(SSL)/sa.key $(SSL)/sa.pub
	DEPS=$(wildcard */*.yaml)
	RES=.

	.PHONY: all keys clean nuke

	all: keys $(TARG)

	%.iso: %.json
		rm -rf $(DIR)/*
		mkdir -p $(DIR)
		cp $< $(DIR)/$<
		rm -f $@
		hdiutil makehybrid -iso -joliet -default-volume-name $* -o $@ $(DIR)

	%.json: %.yaml $(KEYS) $(DEPS)
		ct -files-dir $(RES) -in-file $< -out-file $@

	keys: $(KEYS)

	%.pub: %.key
		mkdir -p $(dir $@)
		openssl ec -in $< -pubout -out $@

	%.key:
		mkdir -p $(dir $@)
		openssl ecparam -name prime256v1 -genkey -noout -out $@

	clean:
		rm -rf $(DIR)

	nuke:
		rm -rf $(DIR) $(SSL) $(TARG)

ここではISOイメージを作成している。
ネットワーク等でインストール前のContainer Linuxとファイルコピーが可能なら
ISOを経由せず直接JSONを作成しても良いが、手元では無かったのでISOにした。

Ignition Configを使ってインストール
-----------------------------------

2番目のCD-ROMは、インストーラから以下のコマンドでマウントできる::

	$ sudo mount -o ro -t iso9660 /dev/sr1 /media

``coreos-install`` に ``-i`` オプションでIgnition Configのパスを渡す::

	$ sudo coreos-install -d /dev/sda -i /media/config.json

ログイン可能にする
------------------

なくても良いけど、ログインできた方が便利なのでContainer Linux Configに書く::

	passwd:
	  users:
	    - name: core
	      ssh_authorized_keys:
	        - "ssh-rsa xxxx"
	      password_hash: $6$xxxx

``password_hash`` は無くても良いが、設定を間違った時の確認に便利::

	$ mkpasswd -m sha-512
	Password:

ネットワーク関連設定
--------------------

ホスト名を設定する::

	storage:
	  files:
	    - path: /etc/hostname
	      filesystem: root
	      mode: 0644
	      contents:
	        inline: (ホスト名)

固定IPアドレスと静的ルートを設定する::

	networkd:
	  units:
	    - name: 10-static.network
	      contents: |
	        [Match]
	        Name=eth0

	        [Network]
	        Address=192.168.1.10/24
	        Gateway=192.168.1.1
	        DNS=192.168.1.1
	        DNS=192.168.1.2

	        [Route]
	        Gateway=192.168.1.124
	        Destination=10.45.0.0/16

静的ルートがなければ ``[Route]`` は無くてもよい。

etcdのインストール
==================

Container Linuxなら設定を書くだけで有効になる。

etcdのインストール
------------------

Container Linux Configに専用のエントリがある::

	etcd:
	  name: app-etcd-1
	  listen_client_urls: http://0.0.0.0:2379
	  advertise_client_urls: http://192.168.1.10:2379
	  listen_peer_urls: http://0.0.0.0:2380
	  initial_advertise_peer_urls: http://192.168.1.10:2380
	  initial_cluster: app-etcd-1=http://192.168.1.10:2380
	  initial_cluster_token: xxxx
	  initial_cluster_state: new

``api-server`` などコンテナの中からetcdにアクセスするため、
ここでは全てのインターフェイスでlistenしているが、
*lo* と *flannel0* に制限しても良いかもしれない。

.. code-block:: console

動作確認
--------

正しく構築できれば、以下のコマンドで操作できる::

	$ etcdctl ls /
	$ etcdctl mkdir /test
	$ etcdctl set /test/key 'aaaa'
	$ etcdctl get /test/key
	$ etcdctl rm /test/key
	$ etcdctl rmdir /test

flanneld
========

flanneldのインストール
----------------------

ホスト起動時に、flanneldに必要な設定を行う::

	systemd:
	  units:
	    - name: flanneld.service
	      dropins:
	        - name: 50-network-config.conf
	          contents: |
	            [Service]
	            ExecStartPre=/usr/bin/etcdctl set /coreos.com/network/config '{"Network": "10.254.0.0/16"}'

このとき、``Network`` の範囲がホストのネットワークと重なってしまうと、
*sshd* なども全て ``Network`` 側に流れてしまって管理ができなくなるので注意。

flanneldを有効にする::

	flannel: ~

マスターノードの構築
====================

マスターノードは以下のプロセスが必要。

* kube-apiserver
* kube-scheduler
* kube-controller-manager

これらは ``kubelet`` を使って、Podとして動作させる。

接続コンテキスト
----------------

接続先のホストと認証情報をまとめてコンテキストとして扱うファイルを作成する。
上記の他にも、``certificate-authority`` などのパラメータが存在する。

まずContainer Linux configにエントリを追加::

	storage:
	  files:
	    - path: /etc/kubernetes/kubeconfig/master-config.yaml
	      filesystem: root
	      mode: 0644
	      contents:
	        local: kubeconfig/master-config.yaml

*kubeconfig/master-config.yaml* の内容::

	apiVersion: v1
	kind: Config
	clusters:
	  - name: local
	    cluster:
	      api-version: v1
	      server: http://127.0.0.1:8080
	contexts:
	  - context:
	      cluster: local
	    name: kubelet-context
	current-context: kubelet-context

このファイルは、マスターノードで動作するコンポーネントから共通して利用する。
上記の他にも、``certificate-authority`` などのパラメータが存在する。

kube-apiserver
---------------

``kube-apiserver`` は、スケジューラや ``kubectl`` などのリクエストを処理するプロセス。
リクエストを受けて、``etcd`` を読み書きして結果を返す。

*/etc/kubernetes/manifests/kube-apiserver.yaml* を作成する::

	files:
	  - path: /etc/kubernetes/manifests/kube-apiserver.yaml
	    filesystem: root
	    mode: 0644
	    contents:
	      local: manifests/kube-apiserver.yaml

*kube-apiserver.yaml* の内容::

	apiVersion: v1
	kind: Pod
	metadata:
	  name: kube-apiserver
	  namespace: kube-system
	spec:
	  hostNetwork: true
	  containers:
	    - name: kube-apiserver
	      image: quay.io/coreos/hyperkube:v1.9.6_coreos.0
	      command:
	        - /hyperkube
	        - apiserver
	        - --insecure-bind-address=0.0.0.0
	        - --insecure-port=8080
	        - --etcd-servers=http://192.168.1.10:2379
	        - --allow-privileged=true
	        - --service-cluster-ip-range=10.253.0.0/16
	        - --advertise-address=192.168.1.10
	        - --admission-control=NamespaceLifecycle,LimitRanger,ServiceAccount,DefaultStorageClass,ResourceQuota
	        - --anonymous-auth=true
	      livenessProbe:
	        httpGet:
	          host: 127.0.0.1
	          port: 8080
	          path: /healthz
	        initialDelaySeconds: 15
	        timeoutSeconds: 15
	      ports:
	        - containerPort: 8080
	          hostPort: 8080
	          name: http
	      volumeMounts:
	        - mountPath: /etc/kubernetes/ssl
	          name: ssl-certs-kubernetes
	          readOnly: true
	        - mountPath: /etc/ssl/certs
	          name: ssl-certs-host
	          readOnly: true
	  volumes:
	    - hostPath:
	        path: /etc/kubernetes/ssl
	      name: ssl-certs-kubernetes
	    - hostPath:
	        path: /usr/share/ca-certificates
	      name: ssl-certs-host

``--admission-control`` オプションは、``kube-apiserver`` の機能を有効にする。

* `Using Admission Controllers <https://kubernetes.io/docs/admin/admission-controllers/>`_

また、ここでは暗号化していないが、認証を行うためにはTLSが必要らしい。
TLSを有効にする場合は ``--bind-address`` と ``--secure-port`` で調整する。
認証には ``--service-account-key-file`` で秘密鍵の指定も必要。

* `kubernetesの認証とアクセス制御を動かしてみる <https://ishiis.net/2017/01/21/kubernetes-authentication-authorization/>`_

kube-controller-manager
-----------------------

ワーカーノードの状態などを取得して ``kube-apiserver`` に渡すプロセス。

*/etc/kubernetes/manifests/kube-controller-manager.yaml* を作成::

	apiVersion: v1
	kind: Pod
	metadata:
	  name: kube-controller-manager
	  namespace: kube-system
	spec:
	  hostNetwork: true
	  containers:
	    - name: kube-controller-manager
	      image: quay.io/coreos/hyperkube:v1.9.6_coreos.0
	      command:
	        - /hyperkube
	        - controller-manager
	        - --master=http://127.0.0.1:8080
	        - --leader-elect=true
	      livenessProbe:
	        httpGet:
	          host: 127.0.0.1
	          path: /healthz
	          port: 10252
	        initialDelaySeconds: 15
	        timeoutSeconds: 1
	      volumeMounts:
	        - mountPath: /etc/kubernetes/ssl
	          name: ssl-certs-kubernetes
	          readOnly: true
	        - mountPath: /etc/ssl/certs
	          name: ssl-certs-host
	          readOnly: true
	  volumes:
	    - hostPath:
	        path: /etc/kubernetes/ssl
	      name: ssl-certs-kubernetes
	    - hostPath:
	        path: /usr/share/ca-certificates
	      name: ssl-certs-host

``--master`` は ``kube-apiserver`` の待ち受けるアドレス。
``kubelet`` で起動する場合、*127.0.0.1* は別のPodで生成されたコンテナに届く。
上記のマニフェストにおいては、TCP/8080は ``kube-apiserver`` のサービスが待ち受ける。

``kube-apiserver`` で認証を有効にした場合は、
``--service-account-private-key-file`` で秘密鍵の指定も必要。
他にも、``--root-ca-file`` などいろいろなオプションがある。

kube-scheduler
---------------

必要なPodの作成、削除を行うプロセス。

*/etc/kubernetes/manifests/kube-scheduler.yaml* を作成::

	apiVersion: v1
	kind: Pod
	metadata:
	  name: kube-scheduler
	  namespace: kube-system
	spec:
	  hostNetwork: true
	  containers:
	    - name: kube-scheduler
	      image: quay.io/coreos/hyperkube:v1.9.6_coreos.0
	      command:
	        - /hyperkube
	        - scheduler
	        - --config=/etc/kubernetes/kubeconfig/kube-scheduler-config.yaml
	      volumeMounts:
	        - mountPath: /etc/kubernetes/kubeconfig
	          name: kubeconfig
	          readOnly: true
	      livenessProbe:
	        httpGet:
	          host: 127.0.0.1
	          path: /healthz
	          port: 10251
	        initialDelaySeconds: 15
	        timeoutSeconds: 1
	  volumes:
	    - hostPath:
	        path: /etc/kubernetes/kubeconfig
	      name: kubeconfig

kube-scheduler-config.yaml::

	apiVersion: componentconfig/v1alpha1
	kind: KubeSchedulerConfiguration
	clientConnection:
	  kubeconfig: /etc/kubernetes/kubeconfig/master-config.yaml
	leaderElection:
	  leaderElect: true

各パラメータは `type KubeSchedulerConfiguration <https://github.com/kubernetes/kubernetes/blob/master/pkg/apis/componentconfig/types.go>`_ を読んで書く。
``apiVersion`` の値は、どこから拾ってくるのが正解なのかわからない。

ワーカーノードの構築
====================

kube-proxy
-----------

``kube-proxy`` は色々なコマンドラインオプションが廃止されて、
代わりにKubeProxyConfigurationが使われるようになった。
Kubernetes 1.9現在、オプションはまだ利用可能だが、

	WARNING: all flags other than --config, --write-config-to, and --cleanup are deprecated. Please begin using a config file ASAP.

のような警告をログに出力するようになった。
``--config`` を使うように修正した方が良いので、このファイルを作成する::

	storage:
	  files:
	    - path: /etc/kubernetes/kubeconfig/kube-proxy-config.yaml
	      filesystem: root
	      mode: 0644
	      contents:
	        local: kubeconfig/kube-proxy-config.yaml

*kube-proxy-config.yaml* の内容::

	apiVersion: kubeproxy.config.k8s.io/v1alpha1
	kind: KubeProxyConfiguration
	bindAddress: 0.0.0.0
	clusterCIDR: 10.254.0.0/16
	#hostnameOverride: app-kube1
	clientConnection:
	  kubeconfig: /etc/kubernetes/kubeconfig/master-config.yaml
	mode: iptables

このファイルは、ドキュメントが見つからなかったので、
`proxy/apis/kubeproxyconfig/v1alpha1/types.go <https://github.com/kubernetes/kubernetes/blob/master/pkg/proxy/apis/kubeproxyconfig/v1alpha1/types.go>`_ のコードを読むしかなかった。

用意ができたら、``kube-proxy`` のマニフェストを用意する::

	storage:
	  files:
	    - path: /etc/kubernetes/manifests/kube-proxy.yaml
	      filesystem: root
	      mode: 0644
	      contents:
	        local: manifests/kube-proxy.yaml

*kube-proxy* のマニフェスト::

	apiVersion: v1
	kind: Pod
	metadata:
	  name: kube-proxy
	  namespace: kube-system
	spec:
	  hostNetwork: true
	  containers:
	  - name: kube-proxy
	    image: quay.io/coreos/hyperkube:v1.9.6_coreos.0
	    command:
	      - /hyperkube
	      - proxy
	      - --config=/etc/kubernetes/kubeconfig/kube-proxy-config.yaml
	    securityContext:
	      privileged: true
	    volumeMounts:
	      - mountPath: /etc/ssl/certs
	        name: ssl-certs-host
	        readOnly: true
	      - mountPath: /etc/kubernetes/kubeconfig
	        name: kubeconfig
	        readOnly: true
	  volumes:
	    - hostPath:
	        path: /usr/share/ca-certificates
	      name: ssl-certs-host
	    - hostPath:
	        path: /etc/kubernetes/kubeconfig
	      name: kubeconfig

IPVSの有効化
------------

試験的に、Kubernetes 1.9以降で、ルーティングにIPVSを使えるようになった。
iptablesでは、数千エントリ以上になった場合に遅くなる問題があるらしい。
これは *kube-proxy-config.yaml* で ``mode: ipvs`` を設定すれば良い。

IPVSを使う場合、*ip_vs* モジュールを有効にする必要がある。
Container Linuxにはモジュールは入っているので、これを有効にする::

	storage:
	  files:
	    - path: /etc/modules-load.d/ip_vs.conf
	      filesystem: root
	      mode: 0644
	      contents:
	        inline: ip_vs

また、試験導入の機能を使うためには、FeatureGateを通して有効にしなければならない。
FeatureGateは *kube-proxy-config.yaml* で設定する(一部抜粋)::

	kind: KubeProxyConfiguration
	featureGates: "SupportIPVSProxyMode=true"
	bindAddress: 0.0.0.0
	mode: ipvs

* `IPVS <https://github.com/kubernetes/kubernetes/blob/master/pkg/proxy/ipvs/README.md>`_
* `Feature Gates <https://kubernetes.io/docs/reference/feature-gates/>`_

ノードの立ち上げ
================

これまでに作ったマニフェストを、``kubelet`` から起動する必要がある。
Container Linuxには、``kubelet-wrapper`` コマンドが用意されていて、
必要に応じてダウンロードと実行を行ってくれるので、これを使う。

kubeletサービスの作成
---------------------

systemdにサービスを作成する::

	systemd:
	  units:
	    - name: kubelet.service
	      enabled: true
	      contents: |
	        [Unit]
	        Description=Kubernetes Kubelet
	        Documentation=https://github.com/kubernetes/kubernetes

	        [Service]
	        Environment=KUBELET_IMAGE_TAG=v1.9.6_coreos.0
	        Environment="RKT_RUN_ARGS=--uuid-file-save=/var/run/kubelet-pod.uuid \
	            --volume var-log,kind=host,source=/var/log \
	            --mount volume=var-log,target=/var/log \
	            --volume dns,kind=host,source=/etc/resolv.conf \
	            --mount volume=dns,target=/etc/resolv.conf"
	        ExecStartPre=/usr/bin/mkdir -p /var/log/containers
	        ExecStartPre=-/usr/bin/rkt rm --uuid-file=/var/run/kubelet-pod.uuid
	        ExecStart=/usr/lib/coreos/kubelet-wrapper \
	            --kubeconfig=/etc/kubernetes/kubeconfig/master-config.yaml \
	            --register-schedulable=true \
	            --allow-privileged=true \
	            --pod-manifest-path=/etc/kubernetes/manifests
	        ExecStop=-/usr/bin/rkt stop --uuid-file=/var/run/kubelet-pod.uuid
	        Restart=always
	        RestartSec=10

	        [Install]
	        WantedBy=multi-user.target

``--register-schedulable`` は、ここでは ``true`` に設定した。
``true`` の場合、自分のホスト情報を、定期的にマスターノードへ登録する。
例えば、マスターノードはワーカーとして動作させたくない場合、
このオプションを ``false`` にするとよい。

``--hostname-override`` は ``os.Hostname()`` の代わりに、
指定したホスト名を使うように指示するオプション。無くても動く。

マスターノードの動作確認
----------------------

.. code-block:: console

コンソールから確認::

	$ kubectl config set-cluster kubetest --server=http://192.168.1.10:8080
	$ kubectl config set-context kubetest --cluster=kubetest
	$ kubectl config use-context kubetest
	$ kubectl cluster-info
	Kubernetes master is running at http://192.168.1.10:8080

.. code-block:: console

ワーカーノードも1つだけ存在する::

	$ kubectl get nodes
	NAME        STATUS    ROLES     AGE       VERSION
	app-kube1   Ready     <none>    2h        v1.9.6+coreos.0

その他情報
==========

マスターノードのログ
--------------------

``kube-apiserver`` のログに、

	etcdserver: mvcc: required revision has been compacted.

というメッセージが流れるけど、これはエラーではないらしい。

``kubelet`` のスタンドアロンモード
-----------------------------------

クラスタの一部ではなく完全に単体で動作するモードらしい。
`Standalone Kubelet Tutorial <https://github.com/kelseyhightower/standalone-kubelet-tutorial>`_ によると、

	There are many options for managing containers on
	a single compute instance including docker compose,
	or some configuration management tool like ansible or chef,
	however the Kubernetes Kubelet running in standalone mode
	may be the better option.

モードの切り替わりは、``--kubeconfig`` オプションがあればクラスタとして、
なければスタンドアロンとして動作する。

ワーカーノードの構築
====================

kube-proxy
-----------

.. todo:: 複数ノードの場合について書く

サービスアカウント設定
======================

このままでは、``kube-apiserver`` と ``kube-controller-manager`` 間で、
トークンがないため ``kubectl create`` が

	No API token found for service account

というエラーになってしまう。
Podの作成など書き込み操作が行えるように、サービスアカウントを作成する。

.. code-block:: console

サービスアカウント鍵の作成::

	$ sudo openssl ecparam -name prime256v1 -genkey -noout -out sa.key
	$ sudo openssl ec -in sa.key -pubout -out sa.pub

ここではECDSAで鍵ペアを作ったが、RSA鍵ペアでも良いらしい。

作った公開鍵を ``kube-apiserver`` のマニフェストに追加::

	spec:
	  containers:
	    - name: kube-apiserver
	      command:
	        - /hyperkube
	        - apiserver
	        - (snip)
	        - --service-account-key-file=/etc/kubernetes/ssl/sa.pub

秘密鍵は ``kube-controller-manager`` のマニフェストに追加::

	spec:
	  containers:
	    - name: kube-apiserver
	      command:
	        - /hyperkube
	        - controller-manager
	        - (snip)
	        - --service-account-private-key-file=/etc/kubernetes/ssl/sa.key

.. code-block:: console

これで起動しなおせば ``kubectl create`` が通る::

	$ kubectl create -f nginx.yaml
	$ kubectl get pods
	NAME      READY     STATUS    RESTARTS   AGE
	nginx     1/1       Running   0          14m

Anonymous auth
---------------

リクエストにトークンが存在しない場合、匿名ユーザとして扱うらしい。
不正なトークンが含まれている場合はエラーになる。

これは ``--anonymous-auth=true`` オプションで有効になるが、
``--authorization-mode`` に ``AlwaysAuth`` が含まれている場合、
危険なので強制的に無効化される。

うまく動かない場合
==================

いくつかのログを調査すると解決するかもしれません。

OSのログ
	*/var/log/messages* のようなログファイル

``kubelet.service`` のログ
	``journalctl -u kubelet.service`` で読めます

``kube-apiserver`` などのログ
	``docker logs`` コマンドで読めます

参考情報
========

クラスタについて。

* `Kubernetes: 構成コンポーネント一覧 <https://qiita.com/tkusumi/items/c2a92cd52bfdb9edd613>`_
* `Getting started with etcd <https://coreos.com/etcd/docs/latest/getting-started-with-etcd.html>`_
* `Configuring flannel for container networking <https://coreos.com/flannel/docs/latest/flannel-config.html>`_
* `How to Deploy Kubernetes on CoreOS Cluster <https://www.upcloud.com/support/deploy-kubernetes-coreos/>`_
* `Deploy Kubernetes Master Node(s) <https://github.com/coreos/coreos-kubernetes/blob/master/Documentation/deploy-master.md>`_
* `Kubernetesでクラスタ環境構築手順 <https://qiita.com/Esfahan/items/db7a79816731e6aa5cf5>`
* `Kubernetesにまつわるエトセトラ <https://www.slideshare.net/WorksApplications/kubernetes-65070472>`_
* `Creating a Custom Cluster from Scratch <https://kubernetes.io/docs/getting-started-guides/scratch/>`_

認証、認可の話。

* `Managing Service Accounts <https://kubernetes.io/docs/admin/service-accounts-admin/>`_
* `Authenticating <https://kubernetes.io/docs/admin/authentication/>`_

Kubernetes以外の話。

* `CoreOSでLVSを有効にする <https://qiita.com/monamour555/items/16581ec18f85a637320e>`_
