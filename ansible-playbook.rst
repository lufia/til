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
