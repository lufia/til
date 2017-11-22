Ansible playbookの書き方
========================

.. highlight:: yaml

ローカルでコマンドを実行したい
------------------------------

Playbookの途中で、ローカルの実行結果を使いたい場合、
``local_action`` を使えばコマンドを走らせることができる。
これは ``action`` (通常は省略する)の代わりなので、モジュールが続く::

	tasks:
	  - name: update users
	    local_action: command updateuserscommand "{{ inventory_hostname }}"

特定のホストでコマンドを実行させたい場合は、``delegate_to`` を使う::

	tasks::
	  - name: sync files
	    command: runcommand "{{ inventory_hostname }}"
	    delegate_to: server

ループで複数ユーザにモジュールを実行したい
------------------------------------------

Ansibleはタスクに対して ``become`` できるので、ループを使おうとすると::

	- git_config: name=user.name scope=global value={{ item }}
	  become: yes
	  become_user: "{{ item }}"
	  with_items:
	    - user1
	    - user2

デフォルトでは以下のエラーになる::

	| Failed to set permissions on the temporary files
	| Ansible needs to create when becoming an unprivileged user

この場合、**ansible.cfg** に ``pipelining=True`` すると良い。

.. code-block:: ini

	[ssh_connection]
	pipelining=True

他にも同じリストに対して処理を行いたいなら、
``include:`` と ``with_items:`` を使うほうが綺麗に書ける場合がある。

main.yml::

	- include: git.yml
	  with_items:
	    - user1
	    - user2
	  loop_control:
	    loop_var: user

git.yml::

	- git_config: name=user.name scope=global value={{ user }}
	  become: yes
	  become_user: "{{ user }}"

踏み台経由で実行したい
----------------------

踏み台を経由して、プライベートIPアドレスなホストを構成する場合、
``ssh -F`` と *ssh_config* で実行する方法はよく見かける。
だけどAnsibleのインベントリと *ssh_config* の両方をメンテするのは面倒なので、
使わないような方法を調べた。

.. code-block:: ini

ansible.cfg::

	[defaults]
	inventory=./hosts
	remote_user=lufia
	private_key_file=~/.ssh/id_rsa

	[ssh_connection]
	ssh_args=-o 'ProxyCommand=ssh -W %h:%p -i ~/.ssh/id_rsa -l lufia proxy.example.net'

.. code-block:: ini

hosts::

	[servers]
	192.168.1.100

.. code-block:: console

これで、以下のようにすると確認ができた::

	$ ansible servers -m ping -v
