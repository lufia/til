==================
Kubernetes関連メモ
==================

用語的なもの
============

Pod
	コンテナインスタンス

	IPアドレスを持つ

Service
	Podをまとめたもの

	ロードバランサ?

	IPアドレスを持つ

Sidecar
	メインのコンテナとは別に、同じPodで動作するコンテナ

	``spec.containers`` に配列として複数書く。

	メインとストレージを共有する

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
