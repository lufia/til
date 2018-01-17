Dockerのボリューム関連
======================

.. highlight:: console

``docker volume`` コマンド
--------------------------

Docker 1.9より、named volumeが使えるようになった。
基本的にdata-only containerパターンを置き換えるもの。

作成方法::

	$ docker volume create xxx

確認方法::

	$ docker volume ls

作成したボリュームは ``dockr run -v volume:/path`` で使える。
*volume* が/からのパスならホストパス、それ以外ならvolume nameとして扱う。

デフォルトは *local* ドライバだけど、*local-persisitent* とか色々ある。

* `Docker Registry storage driver <https://docs.docker.com/registry/storage-drivers/>`_

ボリューム内のユーザID
----------------------

``docker volume create`` で作成したボリュームは、
*local* ドライバの場合、owner=root, group=rootのディレクトリになる。
(/var/lib/docker/volumes/以下に実態がある)

このオーナーを変更したい場合は、別のコンテナでボリュームをマウントして、
マウントした場所を ``chown`` すればいい。

複数コンテナから同じボリュームをマウントする場合、
コンテナ間でユーザIDを合わせておかないと、異なるユーザとして扱われる。
Dockerで作成する一般ユーザは、uid=1000にするのが一般的らしい。

Docker for macのストレージ
--------------------------

以下の場所に、ディスクのフォーマットによってどちらかが存在する。

* *~/Library/Containers/com.docker.docker/Data/com.docker.driver.amd64-linux/Docker.qcow2*
* *~/Library/Containers/com.docker.docker/Data/com.docker.driver.amd64-linux/Docker.raw*

サイズを調べる場合は ``qemu-img`` で行う::

	$ qemu-img info Docker.*

サイズを変更するにはDockerのPreferencesを開き、
Diskタブの **Resize disk image** を使う。
