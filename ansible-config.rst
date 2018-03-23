Ansibleベストプラクティス
=========================

.. highlight:: yaml

* `Best Practices <http://docs.ansible.com/ansible/latest/playbooks_best_practices.html>`_

本当に小規模でなければ、ベストプラクティスの通りに構成したほうがよい。

Ansibleの検索パス
=================

Ansibleは以下の順でファイルを探す。(ベストプラクティスに沿った場合)

1. 実行しているロールの階層(./roles/<ロール名>/files/)
2. Playbookと同じ階層(./)

アクションによって、*files/* だったり *templates/* だったりするが、基本は同じルールが適用される。
*ansible.cfg* や *hosts* 等の、ロールのないものは同じ階層にあればそれを使う。

対象ホスト関連
==============

変数の適用
----------

.. code-block:: ini

*hosts* に、``[xxx:children]`` を定義しておくと、グループ変数を適用できる::

    [node1]
    127.0.0.1

    [docker-servers:children]
    node1

    [servers:children]
    docker-servers

この場合、

1. *./group_vars/servers*
2. *./group_vars/docker-servers*
3. *./host_vars/node1*

が適用される。

* `Ansibleを使い出す前に押さえておきたかったディレクトリ構成のベストプラクティス <http://sechiro.hatenablog.com/entries/2015/01/06>`_

複数ノードの書き方
------------------

複数のグループを指定したいなら、``group1:group2`` のように:で区切る。

* `Patterns <http://docs.ansible.com/ansible/latest/intro_patterns.html>`_

対象ノードのデバッグ
--------------------

.. code-block:: console

以下コマンドで対象となる一覧が取得できる::

    $ ansible --list-hosts {target}

*target* はPlaybookのインベントリに書くものと同じ。

SSH接続関連
===========

SSHユーザ名と秘密鍵を変更する
-----------------------------

.. code-block:: ini

*hosts* ファイルに以下を書く::

    [all:vars]
    ansible_ssh_user=ec2-user
    ansible_ssh_private_key_file=~/.ssh/aws_rsa
    ansible_ssh_common_args=-o 'StrictHostKeyChecking=no'

.. code-block:: ini

*ansible.cfg* を使っても同じことができる::

    [defaults]
    remote_user=ec2-user
    private_key_file=~/.ssh/aws_rsa
    ssh_args=-o 'StrictHostKeyChecking=no'

踏み台経由でアクセスする
------------------------

.. code-block:: ini

通常は、*~/.ssh/config* に書くしかないが、
*ansible.cfg* に ``ssh -F`` コマンドを設定すれば *config* を別に切り出せる::

    [ssh_connection]
    ssh_args = -F ssh.config

.. code-block:: plain

*ssh.config* はこのように::

    host ssh-proxy
        hostname ssh.example.com
    host ssh-node1
        hostname node1.example.local
        proxycommand ssh -F ssh.config ssh-proxy nc %h %p

sftpで接続できないという警告が出る
----------------------------------

``ansible-playbook`` を実行すると以下の警告が出る場合がある。

	sftp transfer mechanism failed on [0.0.0.0]. Use ANSIBLE_DEBUG=1 to see detailed information.

.. code-block:: ini

このエラーが出た場合は、*ansible.cfg* に以下の設定を追加する::

    [defaults]
    transport=ssh

原因は `paramikoがsftpを使う <http://tagomoris.hatenablog.com/entry/20140318/1395118495>`_ から、らしい。

hostsファイル名を変更する
=========================

.. code-block:: ini

*ansible.cfg* で変更する::

    [defaults]
    inventory=./hosts

.. code-block:: console

または、コマンドオプションで変更する::

    $ ansible-playbook -i hosts ...

Playbookの書き方
================

長い行を複数行にする
--------------------

task等でよく書く::

    name: be sure file is exist
    file: path=/path/to/file user=user1 group=group1 mode=0644

これは、長くなる場合、複数行でも書ける::

    name: be sure file is exist
    file:
      path: /path/to/file
      user: user1
      group: group1
      mode: 0644

変数を使う
----------

``{{var}}`` で使える::

    name: be sure file is exist
    file: path="{{ dir_name }}/file" user=user1 group=group1 mode=0644

hostsを変更する
---------------

変数を使うパターン::

    hosts: "{{ hosts | default('development') }}"

こうしたうえで、``--extra-vars 'hosts=xxxx'`` を加えると変更できる。

または、絞り込むパターン。
``ansible-playbook`` コマンドの--limitオプションを使うと対象を絞り込める。

タスクを絞り込む
----------------

tagsでタスクごとにタグを入れておく::

    tasks:
      copy: src=file dest=file mode=0644
      tags:
        - setup

.. code-block:: console

タグを使って絞り込む::

    $ ansible-playbook main.yml --tags setup

複数タグがある場合はカンマで区切る、タグを除く場合は ``--skip-tags`` を使う。

OS単位でタスクを切り替える
--------------------------

``when`` 属性で ``ansible_distribution`` などを使って絞り込む。

====== ==================== ================= ==============
OS     ansible_distribution ansible_os_family ansible_system
====== ==================== ================= ==============
CentOS CentOS               RedHat            Linux
macOS  MacOSX               Darwin            Darwin
====== ==================== ================= ==============

これらは ``ansible all -m setup | egrep 'ansible_(dist|os|sys)'`` で調べる。
調べたら、タスクの ``when`` で条件として上記の変数を条件に書く::

	name: macOSのみ実行するタスク
	command: ...
	when: ansible_distribution == 'MacOSX'

これでも良さそう::

	name: macOSのみ実行するタスク
	command: ...
	when:
	  - ansible_os_family == 'Darwin'

未分類
======

Makefile
--------

.. code-block:: ini

Ansibleは(ssh接続的に)同じホスト名の場合、同じものとして扱われるため::

    [host1]
    127.0.0.1

    [host2]
    127.0.0.1

のようなことはできない。必ずhost1またはhost2のどちらかにのみ属する。

* `separate group_vars being overwritten when deploying to same host <https://github.com/ansible/ansible/issues/9065>`_

.. code-block:: makefile

なので、いくつか方法はあるけどhostsファイルを分ける案が良いと思う。
以下は *hosts* を分けた場合の *Makefile* 例::

    hosts=./hosts

    TARG=$(shell ls -1 $(hosts))
    INSTALL=$(TARG:%=%.install)

    AFLAGS=

    .PHONY : all install
    all: install

    install: $(INSTALL)

    %.install:
    	ansible-playbook $(AFLAGS) -i $(hosts)/$* site.yml

* `ansibleで実行対象を切り替える方法 <http://tdoc.info/blog/2014/05/30/ansible_target_switching.html>`_

retryファイルを作らない
-----------------------

.. code-block:: ini

*ansible.cfg* に以下を書く。``true`` なら作成する::

    [defaults]
    retry_files_enabled = false
    #retry_files_save_path = "~/"
