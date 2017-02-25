SHA1 and Richard Feynman
========================

In Richard Feynman's appendix to the Roger's Commission report on the Space
Shuttle Challenger disaster, one of the issues he describes is a lack of
understanding of the term "safety factor" by NASA managers:

    This is a strange use of the engineer's term, "safety factor." If a bridge
    is built to withstand a certain load without the beams permanently
    deforming, cracking, or breaking, it may be designed for the materials used
    to actually stand up under three times the load. This "safety factor" is to
    allow for uncertain excesses of load, or unknown extra loads, or weaknesses
    in the material that might have unexpected flaws, etc. If now the expected
    load comes on to the new bridge and a crack appears in a beam, this is a
    failure of the design. There was no safety factor at all; even though the
    bridge did not actually collapse because the crack went only one-third of
    the way through the beam.

This lack of understanding seems to mirror the same misunderstanding people
have with respect to SHA1, and other cryptographic breaks.

Lots of people see a practical collision attack and say, "well actually, the
thing you really care about is second pre-image resistance". I see a practical
collision attack and say, "the claimed properties of this algorithm no longer
hold, it is mathematically broken for all cryptographic purposes".

I come to this conclusion for two reasons. One is the age old adage amongst
cryptographers: attacks only get stronger.

The other is that security is really hard. This isn't a particularly novel
observation. Specifically, I believe that security is sufficiently hard that we
should avail ourselves of every opportunity available to us to make our
analysis easier, and to make our systems more resilient to partial failures.

Sometimes competing design considerations mean that we use imperfect
cryptographic constructions. However, whenever "compromise-free" choices are
available to us, we should be using them.

Allow me to illustrate with an example. When it comes to TLS, the usage of SHA1
that gets the most attention is for certificate signatures: for several years
now, the browser ecosystem (led by Microsoft and Google) has been working to
remove SHA1 signed certificates from the ecosystem, with good success. However,
there's another use of hashing: in a TLS connection, the server uses its
private key to sign the handshake, proving the relationship between the
certificate and the server.

In TLS 1.0 and 1.1, the hash used here is a concatenation of MD5 and SHA1. In
TLS 1.2 and above, the client and the server negotiate which hashing algorithm
to use. About 93% of the Alexa Top Million websites will successfully negotiate
SHA256 or better, about 7% of these servers will still chose SHA1, even if the
client supports newer and better things. [#]_

For a long time, using MD5 here was very common. It wasn't until 2015 with the
`SLOTH attack`_ that we fixed up the MD5 usage.

Do we have a practical attack against using SHA1 here today? No. But attacks
only get better, and if we want to fix the entire web ecosystem, we should
start before the attacks are perfect, not once it's already an emergency;
particularly since there's no downsides.

To make this plea actionable: check to make sure your servers aren't using
SHA1! You can check which signature algorithm your server is using with the
OpenSSL command line:

.. code-block:: console

    $ HOST=google.com
    $ PORT=443
    $ echo | \
        openssl s_client -connect $HOST:$PORT -servername $HOST 2>&1 | \
        grep "Peer signing"
    Peer signing digest: SHA256

(This doesn't work with the OpenSSL included with macOS, use the one from
`Homebrew`_).

You can also use Censys to take a look at a `list of the servers that still use
SHA1`_.

The fix for this is to upgrade whatever hardware or software you using for
terminating TLS.

Remember, attacks only get better, and there's too much interesting analysis to
spend your days figuring out whether it's safe to use known broken primitives,
and a cracked bridge has no safety factor.


.. [#] Numbers found using `censys.io`_

.. _`SLOTH attack`: https://www.mitls.org/pages/attacks/SLOTH
.. _`Homebrew`: https://brew.sh/
.. _`list of the servers that still use SHA1`: https://censys.io/domain?q=443.https.tls.signature.hash_algorithm%3Asha1
.. _`censys.io`: https://censys.io
