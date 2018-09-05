.. title: Fractions
.. slug: fractions
.. date: 2017-07-19 06:30:52 UTC+03:00
.. tags: python, snippet
.. category:
.. link:
.. description:
.. type: text

Занятно, но иногда (очень-очень редко) бывают нужны рациональные чиселки. И вот, что пришло в голову:

.. code:: python

	import itertools as it
	from fractions import Fraction


	def take(n, iterable):
		"""Return first n items of the iterable"""
		return it.islice(iterable, n)

	start = Fraction(0)
	step = Fraction(1, 3)

	for i, fr in enumerate(take(11, it.count(start, step))):
		print(f'{i}: {fr}')

	print(f'Result: {i + Fraction(17, 19)}')

В консоле получим:

.. code:: console

	0: 0
	1: 1/3
	2: 2/3
	3: 1
	4: 4/3
	5: 5/3
	6: 2
	7: 7/3
	8: 8/3
	9: 3
	10: 10/3
	Result: 207/19

Собственно, когда нам нужно оперировать долями целого, то разумно накапливать результат прямо в рациональных числах, и лишь в конце перейти в `Decimal <https://docs.python.org/3/library/decimal.html#decimal-objects>`__.
