==================
Kubernetes関連メモ
==================

基本的なところ
==============

なぜKubernetesが必要なのか、という話。実際の運用においては、
コンテナホストの負荷をみてデプロイしたり、
複数ホストを管理したりといった機能が必要だから。
オーケストレーション。

* `なぜKubernetesが必要なのか？ <https://thinkit.co.jp/article/13289>`_

用語的なもの
============

Pod
	コンテナインスタンス

	IPアドレスを持つ

Service
	ロードバランサ?

	IPアドレスを持つ

ReplicaSet
	Podをまとめたもの

	ラベルを使ってクラスタ内で設定したPodの数を維持する

Deployment
	複数のReplicaSetを管理するもの

	ローリングアップデート、ロールバックで使う

	`KubernetesのWorkloadsリソース(その1) <https://thinkit.co.jp/article/13610/page/1/1>`_

DaemonSet
	ReplicaSetに似ている

	全てのノードでコンテナを動かしたい場合に利用する

Sidecar
	メインのコンテナとは別に、同じPodで動作するコンテナ

	``spec.containers`` に配列として複数書く

	メインとストレージを共有する

Minikube
	ローカルでKubernetesを動作させるツール

	冗長性は全く無い

Admission Controller
	``kube-apiserver`` のプラグイン

	コマンドラインオプションで有効・無効を切り替える

	`Using Admission Controllers <https://kubernetes.io/docs/admin/admission-controllers/>`_

Service Account
	APIを利用するためのアカウント

	``controller-manager`` と ``apiserver`` の間で使う

	ユーザアカウントとサービスアカウントの2種類あるが、
	ユーザアカウントはグローバルに一意のユーザで、
	サービスアカウントは名前空間に閉じられる

Ingress
-------

HTTPロードバランサのルール。Serviceとインターネットの間に位置する。
``kind: Ingress`` として定義する。

``type: LoadBalancer`` との違いは、LoadBalancerはTCPロードバランサなので、
リクエストを認識して扱うことができない。

Ingressは ``kind: Config`` と同じようなリソースで、これ自体は動作しない。
動作の実態はIngress Controllerとして実装される。
たとえばIstio Ingress Controller(a Kubernetes Ingress Controller based on Envoy)など。

* `Ingress <https://kubernetes.io/docs/concepts/services-networking/ingress/>`_

Ingress Controller
------------------

Ingressのリソース通りに動作するコントローラ。色々存在する。

* `Ingress Controller Catalog <https://github.com/kubernetes/ingress-nginx/blob/master/docs/ingress-controller-catalog.md>`_
* `NGINX Ingress Controller <https://github.com/kubernetes/ingress-nginx>`_
* Istio Ingress Controller
* `nghttpx Ingress controller <https://github.com/zlabjp/nghttpx-ingress-lb>`_

Istio
------

Envoyをまとめて操作するためのコンポーネント？Service Mesh?

Envoy
	プロキシ

	サイドカーとしてPodに挿入される

Pilot
	Envoyを操作するコンポーネント

Mixer
	Envoy経由で情報収拾してポリシーを管理するコンポーネント

Istio-Auth
	Certified Authority相当

Envoy
------

Nginxのようなもの。
マイクロサービスに便利な、分散トレーシングなどが用意されている。

Helm
------

Kubernetesのパッケージマネージャ。Podをパッケージとして運用する。

Hyperkube
---------

Kubernetesを構成するコンポーネントが全部まとまったもの。
``hyperkube apiserver`` のようにサブコマンドでコンポーネントを実行する。

Kubeadm
--------

良くわからない。構築するための便利ツール？

kube-dns
---------

Podが起動や移動した時に追従してくれるDNS。

Kubectl
=======

``kubectl`` はユーザとクラスタをコンテキストにまとめて、
コンテキストを切り替えてKubernetesにアクセスする。

* `Kubernetesの基礎 <https://thinkit.co.jp/article/13542>`_

ユーザ
	接続ユーザ名とキー

クラスタ
	Kubernetesクラスタのホスト名とポート

コンテキスト
	ユーザ、クラスタ、名前空間をセットにしたもの

	複数のコンテキストを作成可能

この設定は *~/.kube/config* に置かれている。
ファイルの場所は環境変数 *KUBECONFIG* で変更可能。

リンク
======

Kubernetes
----------

* `Kubernetesのネットワーク <http://tech.uzabase.com/entry/2017/09/12/164756>`_
* `Kubernetes サイドカーの作り方とファイル共有 <https://qiita.com/MahoTakara/items/c6db540a5a121cc7c2c2>`_
* `Kubernetesはクラスタで障害があったとき、どういう動きをするのか <http://dr-asa.hatenablog.com/entry/2018/04/02/174006>`_
* `Kubernetes Security - Best Practice Guide <https://github.com/freach/kubernetes-security-best-practice>`_
* `Deploymentの仕組み <https://qiita.com/tkusumi/items/01cd18c59b742eebdc6a>`_

ロギング
--------

* `k8sでロギングってどんなやり方があるんかな？ <http://bufferings.hatenablog.com/entry/2018/01/25/222500>`_
* `Kubernetesのログ分析環境を作る <http://tech.uzabase.com/entry/2018/02/01/161447>`_

Istio
------

* `Envoy、Istioとは <https://qiita.com/seikoudoku2000/items/9d54f910d6f05cbd556d>`_
* `Kubernetesをサービスメッシュ化するIstioとは？ <https://thinkit.co.jp/article/13471>`_
* `Istio入門 その1 <https://qiita.com/Ladicle/items/979d59ef0303425752c8>`_

その他
------

* `RPCに特化したGoogleのセキュリティ通信ALTSとは何か <https://jovi0608.hatenablog.com/entry/2018/01/16/085647>`_
* `Google Borgとコンテナベース分散システムデザインパターン <https://www.slideshare.net/ktateish/google-borg>`_
* `クラウドの設計パターン <https://docs.microsoft.com/ja-jp/azure/architecture/patterns/>`_
