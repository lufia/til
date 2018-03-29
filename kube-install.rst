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
	RES=.

	.PHONY: all clean nuke

	all: $(TARG)

	%.iso: %.json
		rm -rf $(DIR)/*
		mkdir -p $(DIR)
		cp $< $(DIR)/$<
		hdiutil makehybrid -iso -joliet -default-volume-name $* -o $@ $(DIR)

	%.json: %.yaml
		ct -files-dir $(RES) -in-file $< -out-file $@

	clean:
		rm -rf $(DIR)

	nuke:
		rm -rf $(DIR) $(TARG)

ここではISOイメージを作成している。
ネットワーク等でインストール前のContainer Linuxとファイルコピーが可能なら
ISOを経由せず直接JSONを作成しても良いが、手元では無かったのでISOにした。

Ignition Configを使ってインストール
-----------------------------------

2番目のCD-ROMは、インストーラから以下のコマンドでマウントできる::

	$ sudo mount -o ro -t iso9660 /dev/sr1 /media

``coreos-install`` に ``-i`` オプションでIgnition Configのパスを渡す::

	$ sudo coreos-install -d /dev/sda /media/config.json

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
	            ExecStartPre=/usr/bin/etcdctl set /coreos.com/network/config '{"Network": "10.1.0.0/16"}'

このとき、``Network`` の範囲がホストのネットワークと重なってしまうと、
*sshd* なども全て ``Network`` 側に流れてしまって管理ができなくなるので注意。

flanneldを有効にする::

	flannel: ~

Masterノードの構築
==================

Masterノードは以下のプロセスが必要。

* kube-apiserver
* kube-scheduler
* kube-controller-manager

これらは ``kubelet`` を使って、Podとして動作させる。

kube-apiserver
---------------

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
	        - --service-cluster-ip-range=10.1.0.0/24
	        - --advertise-address=192.168.1.10
	        - --admission-control=NamespaceLifecycle,LimitRanger,ServiceAccount,DefaultStorageClass,ResourceQuota
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

ここでは暗号化していないが、認証を行うためにはTLSが必要らしい。
TLSを有効にする場合は ``--bind-address`` と ``--secure-port`` で調整する。
認証には ``--service-account-key-file`` で秘密鍵の指定も必要。

* `kubernetesの認証とアクセス制御を動かしてみる <https://ishiis.net/2017/01/21/kubernetes-authentication-authorization/>`_

kube-controller-manager
-----------------------

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
	        - --root-ca-file=/etc/kubernetes/ssl/ca.pem
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
``kube-apiserver`` で認証を有効にした場合は、
``--service-account-private-key-file`` で秘密鍵の指定も必要。

kube-scheduler
---------------

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
	         - --master=http://127.0.0.1:8080
	         - --leader-elect=true
	       livenessProbe:
	         httpGet:
	           host: 127.0.0.1
	           path: /healthz
	           port: 10251
	         initialDelaySeconds: 15
	         timeoutSeconds: 1

kubeletサービスの作成
---------------------

Container Linuxには ``kubelet`` のラッパーコマンドがあるので使う::

	systemd:
	  units:
	    - name: kubelet.service
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
	            --kubeconfig=/etc/kubernetes/master-config.yaml \
	            --register-schedulable=false \
	            --allow-privileged=true \
	            --pod-manifest-path=/etc/kubernetes/manifests \
	            --hostname-override=127.0.0.1
	        ExecStop=-/usr/bin/rkt stop --uuid-file=/var/run/kubelet-pod.uuid
	        Restart=always
	        RestartSec=10

	        [Install]
	        WantedBy=multi-user.target

``kubelet`` の ``--api-servers`` オプションは無くなったので、
代わりに ``--kubeconfig`` で設定を渡す必要がある。

Masterノードのkubeconfig例::

	storage:
	  files:
	    - path: /etc/kubernetes/master-config.yaml
	      filesystem: root
	      mode: 0644
	      contents:
	        local: master-config.yaml

*master-config.yaml* の内容::

	apiVersion: v1
	kind: Config
	clusters:
	  - name: local
	    cluster:
	      api-version: v1
	      server: http://127.0.0.1:8080
	      certificate-authority: /etc/kubernetes/ssl/ca.pem

.. code-block:: console

Masterノードの動作確認
----------------------

kubeletを起動する::

	$ sudo systemctl start kubelet.service
	$ sudo systemctl enable kubelet.service

.. code-block:: console

コンソールから確認::

	$ kubectl config set-cluster kubetest --server=http://192.168.1.10:8080
	$ kubectl config set-context kubetest --cluster=kubetest
	$ kubectl config use-context kubetest
	$ kubectl cluster-info

.. code-block:: console

Workerノードはまだ無い::

	$ kubectl get nodes
	No resources found.

Workerノードの構築
==================

kube-proxy
-----------

TODO

参考情報
========

* `Kubernetes: 構成コンポーネント一覧 <https://qiita.com/tkusumi/items/c2a92cd52bfdb9edd613>`_
* `Getting started with etcd <https://coreos.com/etcd/docs/latest/getting-started-with-etcd.html>`_
* `Configuring flannel for container networking <https://coreos.com/flannel/docs/latest/flannel-config.html>`_
* `How to Deploy Kubernetes on CoreOS Cluster <https://www.upcloud.com/support/deploy-kubernetes-coreos/>`_
* `Deploy Kubernetes Master Node(s) <https://github.com/coreos/coreos-kubernetes/blob/master/Documentation/deploy-master.md>`_
* `Kubernetesにまつわるエトセトラ <https://www.slideshare.net/WorksApplications/kubernetes-65070472>`_
