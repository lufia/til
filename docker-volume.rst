Dockerのボリューム関連
======================

.. highlight:: console

Docker for macのストレージ
--------------------------

以下の場所に、ディスクのフォーマットによってどちらかが存在する。

* *~/Library/Containers/com.docker.docker/Data/com.docker.driver.amd64-linux/Docker.qcow2*
* *~/Library/Containers/com.docker.docker/Data/com.docker.driver.amd64-linux/Docker.raw*

サイズを調べる場合は ``qemu-img`` で行う::

	$ qemu-img info Docker.*

サイズを変更するにはDockerのPreferencesを開き、
Diskタブの **Resize disk image** を使う。
