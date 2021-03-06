.. _views:

Views
=====

Each page that your players see is defined by a ``Page`` class in
``views.py`` ("views" is basically a synonym for "pages").

Your ``views.py`` must have a ``page_sequence``
variable that gives the order of the pages. For example:

.. code-block:: python

    page_sequence = [Start, Offer, Accept, Results]

If your game has multiple rounds, this sequence will be repeated.
See :ref:`rounds` for more info.

A ``Page`` class can have any of the following optional methods and attributes:

.. _is_displayed:

is_displayed()
~~~~~~~~~~~~~~

You can define this function to return ``True`` if the page should be shown,
and False if the page should be skipped.
If omitted, the page will be shown.

For example, to only show the page to P2 in each group:

.. code-block:: python

    class Page1(Page):
        def is_displayed(self):
            return self.player.id_in_group == 2

Or only show the page in round 1:

.. code-block:: python

    class Page1(Page):
        def is_displayed(self):
            return self.round_number == 1

If you need to repeat the same rule for many pages, see :ref:`here <skip_many>`.

The code you put in ``is_displayed()`` should just return ``True`` or ``False``;
it should not have any side effects, such as modifying a model field.

.. _vars_for_template:

vars_for_template()
~~~~~~~~~~~~~~~~~~~

Use this to pass variables to the template. Example:

.. code-block:: python

    class Page1(Page):
        def vars_for_template(self):
            return {'a': 1 + 1, 'b': self.player.foo * 10}

Then in the template you can access ``a`` and ``b`` like this:

.. code-block:: html+django

    Variables {{ a }} and {{ b }} ...

oTree automatically passes the following objects to the template:
``player``, ``group``, ``subsession``, ``participant``, ``session``, and ``Constants``.
You can access them in the template like this: ``{{ Constants.blah }}`` or ``{{ player.blah }}``.

If you need to pass the same variables to many pages,
see :ref:`here <vars_for_many_templates>`.

.. note::

    You shouldn't generate random values in ``vars_for_template``,
    because if the user refreshes their page, ``vars_for_template`` will be executed again,
    and the random calculation might return a different value.
    Instead, you should calculate random values in either ``creating_session``,
    ``before_next_page``, or ``after_all_players_arrive``, each of which
    only executes once.

.. _before_next_page:

before_next_page()
~~~~~~~~~~~~~~~~~~

Here you define any code that should be executed
after form validation, before the player proceeds to the next page.

If the page is skipped with ``is_displayed``,
then ``before_next_page`` will be skipped as well.

Example:

.. code-block:: python

    class Page1(Page):
        def before_next_page(self):
            self.player.tripled_payoff = self.player.bonus * 3

template_name
~~~~~~~~~~~~~

Each Page should have a file in ``templates/`` with the same name.
For example, if your app has this page in ``my_app/views.py``:

.. code-block:: python

    class Page1(Page):
        pass

Then you should create a file ``my_app/templates/my_app/Page1.html``,
(note that my_app is repeated).
See :ref:`templates` for info on how to write an HTML template.

If the template needs to have a different name from your
view class (e.g. you are sharing the same template for multiple views),
set ``template_name``. Example:

.. code-block:: python

    class Page1(Page):
        template_name = 'app_name/MyView.html'

timeout_seconds, timeout_submission, etc
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

See :ref:`timeouts`


Randomizing page sequence
~~~~~~~~~~~~~~~~~~~~~~~~~

You can randomize the order of pages using rounds.
An example is `here <https://github.com/oTree-org/otree-snippets/tree/master/random_page_order>`__.