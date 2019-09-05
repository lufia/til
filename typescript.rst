===========
TypeScript
===========

.. highlight: typescript

型
=====

Mapped Types
--------------

こんな雰囲気::

	type A = 'aaa' | 'bbb' | 'ccc';
	type B = { [kind in A]: string }

これは以下のコードと同じ::

	type B = {
		aaa: string
		bbb: string
		ccc: string
	}

Union TYpes
------------

雰囲気::

	type A = X | Y;

Intersection Types
--------------------

雰囲気::

	type A = X & Y;

* `Advanced Types <https://www.typescriptlang.org/docs/handbook/advanced-types.html#mapped-types>`_
