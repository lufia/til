GKE(Kubernetes Engine)の管理
============================

最小限の権限
------------

`Kubernetes Engine 1.8でのコンテナセキュリティの強化 <https://cloudplatform-jp.googleblog.com/2017/12/precious-cargo-securing-containers-with-Kubernetes-Engine-18.html>`_ によると、
サービスアカウントに少なくとも以下3つのロールが必要

* monitoring.viewer
* monitoring.metricWriter
* logging.logWriter

デフォルトのアカウントはいっぱいロールを持っているので、
サービスアカウントを作ってそれを使ったほうが良い。
