The worst HTTP offenders
========================

For a year or so I've been running a `Chrome extension I wrote`_, which tracks
which origins I make the most ``http://`` requests to, in an effort to be more
data driven. Websites that top this list are flagrantly disrespectful of my
privacy and show no regard for whether bytes make their way to me unmodified.

Over the past year, I've seen several websites that used to top this list
migrate to ``https://``, for example Amazon, Netflix, and the Washington Post.

Here's the top 25 worst HTTP offenders (grouped by brand)! If you work for one
of these sites, offering HTTPS should be a top priority. If you're not sure
why, watch `this talk on MOAR TLS`_ from earlier this year.

ESPN
----

ESPN has a lot of domains.

* ``a.espncdn.com``
* ``a1.espncdn.com``
* ``cdn.espn.go.com``
* ``a2.espncdn.com``
* ``cdn.espn.com``
* ``site.api.espn.com``
* ``fcast.us-east-1.espncdn.com``
* ``fastcast.espn.com``
* ``a4.espncdn.com``
* ``www.espn.com``

MLB
---

Sports seems to be a theme.

* ``mlb.mlb.com``
* ``statsapi.mlb.com``
* ``m.mlb.com``
* ``an.mlb.com``


24/7 Sudoku
-----------

* ``www.247sudoku.com``
* ``www.fortythievessolitaire.info``

New York Times
--------------

While most of the New York Times site has gone ``https://``, the `daily
KenKen`_ has not yet.

* ``www.nytimes.com``

Comedy Central
--------------

* ``mb.mtvnservices.com``

IMDB
----

* ``ia.media-imdb.com``

Basketball Reference
--------------------

* ``www.basketball-reference.com``

Facebook
--------

Facebook itself is all ``https://``, but this domain isn't

* ``static.ak.fbcdn.net``

CNN
---

* ``data.cnn.com``
* ``i.cdn.turner.com``
* ``i2.cdn.turner.com``
* ``www.i.cdn.cnn.com``


.. _`Chrome extension I wrote`: https://github.com/alex/tls-stats
.. _`this talk on MOAR TLS`: https://www.youtube.com/watch?v=jplIY1GXBHM
.. _`daily KenKen`: http://www.nytimes.com/ref/crosswords/kenken.html
