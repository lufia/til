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

Sidecar
	メインのコンテナとは別に、同じPodで動作するコンテナ

	``spec.containers`` に配列として複数書く

	メインとストレージを共有する

Minikube
	ローカルでKubernetesを動作させるツール

	冗長性は全く無い

Ingress
-------

HTTPロードバランサ。Serviceとインターネットの間に位置する。
``kind: Ingress`` として定義する。

``type: LoadBalancer`` との違いは、LoadBalancerはTCPロードバランサなので、
リクエストを認識して扱うことができない。

実態はIngress Controllerとして実装されるらしい。
たとえばIstio Ingress Controller(a Kubernetes Ingress Controller based on Envoy)など。

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

Istio
------

* `Envoy、Istioとは <https://qiita.com/seikoudoku2000/items/9d54f910d6f05cbd556d>`_
* `Kubernetesをサービスメッシュ化するIstioとは？ <https://thinkit.co.jp/article/13471>`_
* `Istio入門 その1 <https://qiita.com/Ladicle/items/979d59ef0303425752c8>`_
