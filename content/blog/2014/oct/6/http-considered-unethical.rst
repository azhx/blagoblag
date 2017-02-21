HTTP Considered Unethical
=========================

Returning visitors: notice anything different? No, not the design (although
thanks `Kenneth Love`_!). I'm talking about the lock icon! Accessing this
website now requires TLS (and a fairly modern client to boot).

But Alex, it's just your blog! There's nothing confidential on here, why does
it need TLS?

First, TLS doesn't just guarantee confidentiality, it also provides
authentication and guarantees the integrity of this page. That prevents an
`attacker on the network`_ from serving you bogus content, and pretending like
it's from me.

But we could work around those, I could include a PGP signature for every post,
and you'd know they're from me. Confidentiality is required because **I
shouldn't get to have an opinion about what's important to my readers**.

Everything on my blog may be public, but that doesn't mean any given reader is
ok with it being known by the prying eyes between them and me.

A student should be able to browse for information on depression without their
principal knowing it, a dissident should be able to research how to avoid
national firewalls without their government knowing it, and anyone who damn
well pleases should be able to read whatever they like without their ISP
knowing it.

A key issue in designing API and user interfaces is making sure that actions
reflect the intent and agency of their user. No users' mental model for how
their browser works includes the possibility that attackers are reading or
manipulating the contents of pages (if you don't believe, try explaining to
someone that the ``From`` field of an email doesn't prove who it's really
from).

As operators of websites, we should not be empowered to give away our users'
privacy.

Right now on the web, the default is unauthenticated and unencrypted, and
that's got to change. HTTP must cease to exist as we know it. Typing
``example.com`` *must* access it over HTTPS.

If you operate a website, please add HTTPS [#]_, and redirect all traffic to it. If
you're in a position to define web standards going forward, or browser UI:
require HTTPS for anything and everything, and where you can't, penalize sites
using HTTP.

.. [#] Cost is no longer an excuse, both `StartSSL`_ and `CloudFlare`_ provide
       free certificates.

.. _`Kenneth Love`: http://gigantuan.net/
.. _`attacker on the network`: https://arstechnica.com/tech-policy/2014/09/why-comcasts-javascript-ad-injections-threaten-security-net-neutrality/
.. _`StartSSL`: https://www.startssl.com/
.. _`CloudFlare`: https://www.cloudflare.com
