==========
Kubernetes
==========

.. highlight: console

* `kubectl Overview <https://kubernetes-v1-4.github.io/docs/user-guide/kubectl-overview/>`_

構築関連
========

* https://cloud.google.com/kubernetes-engine/docs/quickstart
* https://cloud.google.com/kubernetes-engine/docs/tutorials/persistent-disk/

GKEの場合、クラスタ関連の項目はGKEのメニューに入っているが、
GCE側にもディスクやインスタンスの情報が含まれる。

永続ディスクの作成
------------------

``gcloud compute disks create`` を使う::

	$ gcloud compute disks create --size 200GB venti-disk

作成したディスクは ``gcloud compute disks list`` で確認する::

	$ gcloud compute disks list
	NAME           ZONE               SIZE_GB  TYPE         STATUS
	venti-disk     asia-northeast1-a  200      pd-standard  READY

永続ディスクの利用
------------------

.. code-block: yaml

PodのYAMLファイルでボリュームとしてマウントする::

	spec:
	  template:
	    spec:
	      containers:
	        - image: lufia/venti:latest
	          volumeMounts:
	            - name: venti-persistent-storage
	              mountPath: /mnt/venti
	      volumes:
	        - name: venti-persistent-storage
	          gcePersistentDisk:
	            pdName: venti-disk
	            fsType: ext4

上の設定は、ボリュームに必要な部分だけ抜粋した。

ログの表示
----------

Podのログを表示するには、まずPodの名前を調べる::

	$ kubectl get pod
	NAME                READY     STATUS    RESTARTS   AGE
	venti-xxxx-xx       1/1       Running   0          1m

名前を使ってログを読む::

	$ kubectl logs -f venti-xxxx-xx

コンテナにログイン
------------------

Dockerと同じ::

	$ kubectl exec -ti venti-xxxx-xx /bin/bash

コンテキスト
------------

現在のコンテキストを取得::

	$ kubectl config current-context

コンテキスト取得::

	$ kubectl config get-contexts

コンテキスト削除::

	$ kubectl config delete-context xxxx

クラスタ取得::

	$ kubectl config get-clusters

クラスタ削除::

	$ kubectl config delete-cluster xxxx
