.. _currency:

Money & Payoffs
===============

In many experiments, participants play for currency:
either real money, or points. oTree supports both;
you can switch from points to real money by setting ``USE_POINTS = False``
in ``settings.py``.

If you have a value that represents an amount of currency
(either points or dollars, etc),
you should mark it with ``c()``, e.g. ``c(10)`` or ``c(0)``.
It will still work just like a number
(e.g. ``c(1) + c(0.2) == c(1.2)``).
The advantage is that when it's displayed to users, it will automatically
formatted as ``$1.20`` or ``1,20 €``, etc., depending on your
``REAL_WORLD_CURRENCY_CODE`` and ``LANGUAGE_CODE`` settings.
Money amounts are displayed with 2 decimal places by default;
you can change this with the setting ``REAL_WORLD_CURRENCY_DECIMAL_PLACES``.

If a model field is a currency amount,
you should define it as a ``CurrencyField``.
For example:

.. code-block:: python

    class Player(BasePlayer):
        random_bonus = models.CurrencyField()

        def some_method(self):
            self.random_bonus = c(random.randint(1, 10))

Note: instead of using Python's built-in ``range`` function,
you should use oTree's ``currency_range`` with currency values,
e.g.:

.. code-block:: python

    class Player(BasePlayer):
        contribution = models.CurrencyField(
            choices=currency_range(c(0), c(0.10), c(0.02))
            # this gives:
            # [$0.00, $0.02, $0.04, $0.06, $0.08, $0.10]
        )


``currency_range`` takes 3 arguments (start, stop, step), just like range.
However, unlike ``range()``, the returned list includes the ``stop`` value
as shown above.

In templates, instead of using the ``c()`` function, you should use the
``|c`` filter.
For example, ``{{ 20|c }}`` displays as ``20 points``.

.. _payoff:

payoffs
-------

Each player has a ``payoff`` field,
which is a ``CurrencyField``.
If your player makes money, you should store it in this field.
``self.participant.payoff`` automatically stores the sum of payoffs
from all subsessions. You can modify ``self.participant.payoff`` directly,
e.g. to round the final payoff to a whole number.

At the end of the experiment, a participant's
total profit can be accessed by ``self.participant.payoff_plus_participation_fee()``;
it is calculated by converting ``self.participant.payoff`` to real-world currency
(if ``USE_POINTS`` is ``True``), and then adding
``self.session.config['participation_fee']``.

.. _points:

Points (i.e. "experimental currency")
-------------------------------------

If you set ``USE_POINTS = True``, then currency amounts will be points instead of dollars/euros/etc.
For example, ``c(10)`` is displayed as ``10 points`` (or ``10 puntos``, etc.)

The oTree's admin's payment tab will automatically convert points to money
using your session config's ``real_world_currency_per_point``
(you can add a ``real_world_currency_per_point`` entry to each session config).

Points are integers. This means amounts will get rounded to whole numbers,
like ``10`` divided by ``3`` is ``3``.
So, we recommend using point magnitudes high enough that you don't care about rounding error.
For example, set the endowment of a game to 1000 points, rather than 100.
However, if you really want decimal places in your points, you can set
``POINTS_DECIMAL_PLACES = 2``, etc.

To customize the name "points" to something else like "tokens" or "credits",
set ``POINTS_CUSTOM_NAME``, e.g. ``POINTS_CUSTOM_NAME = 'tokens'``.

Converting points to real world currency
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can convert a points amount to money using the method
``.to_real_world_currency(self.session)``. In the above example, that would be:

.. code-block:: python

    >>> c(10).to_real_world_currency(self.session)
    $0.20

It requires ``self.session``, because
different sessions can have different conversion rates).

Misc
~~~~

If you want numbers to be formatted like ``1,000,000`` or ``1 000 000``,
see `USE_THOUSAND_SEPARATOR <https://docs.djangoproject.com/en/1.11/ref/settings/#std:setting-USE_THOUSAND_SEPARATOR>`__.
