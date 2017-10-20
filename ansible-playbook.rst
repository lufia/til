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
