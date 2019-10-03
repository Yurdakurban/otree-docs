.. _wait_pages:

Wait pages
==========

Wait pages are necessary when one player needs to wait for
others to take some action before they can proceed. For example,
in an ultimatum game, player 2 cannot accept or reject before they have
seen player 1's offer.

If you have a ``WaitPage`` in your sequence of pages,
then oTree waits until all players in the group have
arrived at that point in the sequence, and then all players are allowed
to proceed.

If your subsession has multiple groups playing simultaneously, and you
would like a wait page that waits for all groups (i.e. all players in
the subsession), you can set the attribute
``wait_for_all_groups = True`` on the wait page.

For more information on groups, see :ref:`groups`.

Wait pages can define the following methods:

.. _after_all_players_arrive:

after_all_players_arrive()
--------------------------

``after_all_players_arrive`` will be executed once all players have arrived at the wait
page. This is a good place to set the players' payoffs
or determine the winner. You should first define a method on the group,
then call it here:

.. code-block:: python

    def after_all_players_arrive(self):
        self.group.set_payoffs()

If you set ``wait_for_all_groups = True``,
then you should define a method on the *subsession* and call that.

is_displayed()
--------------

Works the same way as with regular pages.

If some players in the group skip the wait page,
then ``after_all_players_arrive()`` may not be run.

.. _group_by_arrival_time:

group_by_arrival_time
---------------------

If you set ``group_by_arrival_time = True`` on a WaitPage,
players will be grouped in the order they arrive at that wait page:

.. code-block:: python

    class MyWaitPage(WaitPage):
        group_by_arrival_time = True

For example, if ``players_per_group = 2``, the first 2 players to arrive
at the wait page will be grouped together, then the next 2 players, and so on.

This is useful in sessions where some participants
might drop out (e.g. online experiments,
or experiments with consent pages that let the participant quit early), or
sessions where some participants take much longer than others.

A typical way to use ``group_by_arrival_time`` is to put it after an app
that filters out participants. For example, if your session has a consent page
that gives participants the chance to opt out of the study, you can make a "consent" app
that just contains the consent pages, and
then have an ``app_sequence`` like ``['consent', 'my_game']``,
where ``my_game`` uses ``group_by_arrival_time``.
This means that if someone opts out in ``consent``,
they will be excluded from the grouping in ``my_game``.

If a game has multiple rounds,
you may want to only group by arrival time in round 1:

.. code-block:: python

    class MyWaitPage(WaitPage):
        group_by_arrival_time = True

        def is_displayed(self):
            return self.round_number == 1

If you do this, then subsequent rounds will keep the same group structure as
round 1. Otherwise, players will be re-grouped by their arrival time
in each round.
(``group_by_arrival_time`` copies the group structure to future rounds.)

Notes:

-   ``id_in_group`` is not necessarily assigned in the order players arrived at the page.
-   ``group_by_arrival_time`` can only be used if the wait page is the first page in ``page_sequence``
-   If you use ``is_displayed`` on a page with ``group_by_arrival_time``,
    it should only be based on the round number. Don't use ``is_displayed``
    to show the page to some players but not others.
-   If ``group_by_arrival_time = True``, then in ``creating_session``,
    all players will initially be in the same group. Groups are only created
    "on the fly" as players arrive at the wait page.

If you need further control on arranging players into groups,
use :ref:`get_players_for_group`.

.. _get_players_for_group:

get_players_for_group()
-----------------------

If you're using ``group_by_arrival_time`` and want more control over
which players are assigned together, you can use ``get_players_for_group()``.

Let's say that in addition to grouping by arrival time, you need each group
group to consist of 1 man and 1 woman (or 2 "A" players and 2 "B" players, etc).

If you define a method called ``get_players_for_group``,
it will get called whenever a new player reaches the wait page.
The method's argument is the list of players who are waiting to be grouped
(minus those who have disconnected or closed the page).
If you select some of these players and return them as a list,
those players will be assigned to a group, and move forward.
If you don't return anything, then no grouping occurs.

Here's an example where each group has 2 A players, 2 B players.

.. code-block:: python

    class GroupingWaitPage(WaitPage):
        group_by_arrival_time = True

        def get_players_for_group(self, waiting_players):
            print('in get_players_for_group')
            a_players = [p for p in waiting_players if p.participant.vars['type'] == 'A']
            b_players = [p for p in waiting_players if p.participant.vars['type'] == 'B']

            if len(a_players) >= 2 and len(b_players) >= 2:
                print('about to create a group')
                return [a_players[0], a_players[1], b_players[0], b_players[1]]
            print('not enough players to create a group')

        def is_displayed(self):
            return self.round_number == 1

.. _wait-page-stuck:

Preventing players from getting stuck on wait pages
---------------------------------------------------

A common problem especially with online experiments is players getting stuck
waiting for another player in their group who dropped out or is too slow.

Here are some things you can do to reduce this problem:

Use ``group_by_arrival_time``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

As described above, you can use ``group_by_arrival_time`` so that only
players who are actively playing around the same time get grouped together.

``group_by_arrival_time`` works well if used after a "lock-in" task.
In other words, before your multiplayer game, you can have a
single-player effort task. The idea is that a
participant takes the effort to complete this initial task, they are
less likely to drop out after that point.

Use page timeouts
~~~~~~~~~~~~~~~~~

Use :ref:`timeout_seconds` on each page, so that if a player is slow or inactive,
their page will automatically advance. Or, you can manually force a timeout by clicking
the "Advance slowest participants" button in the admin interface.

Customizing the wait page's appearance
--------------------------------------

You can customize the text that appears on a wait page
by setting the ``title_text`` and ``body_text`` attributes, e.g.:

.. code-block:: python

    class MyWaitPage(WaitPage):
        title_text = "Custom title text"
        body_text = "Custom body text"

See also: :ref:`customize_wait_page`.