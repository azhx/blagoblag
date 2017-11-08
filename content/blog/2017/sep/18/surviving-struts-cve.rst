Surviving Apache Struts CVE-2017-5638
=====================================

If you're a software engineer or work in tech, there's a decent chance that
your first thought after hearing about the Equifax breach was "oh my god, how
incompetent do you have to be to get owned like that?" Don't worry, I had the
same reaction. After a few days of introspection and reviewing the evidence,
I've come to the conclusion that Equifax made one uncommonly disastrous
mistake: not upgrading Struts immediately after a remote-code-execution
vulnerability was disclosed in it; everything else about the situation was
exceptionally common. My best bet is 497 of the Fortune 500 couldn't survive
that mistake either. "Just upgrade" is both valuable advice, and not
particularly interesting to explore. What if we wanted to design a system that
could withstand this mistake?

Or better yet, what if we wanted to survive *unknown mistakes*? Equifax screwed
up huge because there was a known RCE and they didn't patch, however for many
years before that there was an unknown RCE. But they, and many other companies,
also screw up by not having an architecture which is resilient to such
mistakes. If the person who discovered the Apache Struts vulnerability had been
motivated differently companies could have been exploited without warning. What
would we have to do to survive that situation?

I have no particular insider information on what Equifax's environment looks
like (or any of the Fortune 500's for that matter), but let's imagine they look
like what a reasonably savvy startup using AWS has:

They've got a VPC in us-east-1, and some EC2 instances in it, perhaps even in a
multiple availability zones. Each AZ's EC2 instances are in a security group,
and the only things with ingress to the SG is an ELB, in HTTP/HTTPS mode and a
bastion server. EC2 instances have private IPs only, but can access the
internet through a NAT gateway. The application uses an RDS PostgreSQL
database, it has full disk encryption enabled, requires TLS for connections,
and is accessible exclusively via our security groups.

This is a pretty well put together infrastructure for a startup. And if the EC2
instances were running a vulnerable copy of Struts, it would be game over for
our startup. Once an attacker had RCE, they'd grab the DB credentials from
disk, from an environment variable, or right out of memory if they had to. Pump
the DB for data, and then ship it off network, they could even upload it to S3
if you want the outbound traffic to be clandestine. Pwned.

Now, perhaps there are things on the detection front that could be done to
allow us to notice slightly more quickly than the amount of time it takes to
ship 143 million people's data off the network, but I'm not going to focus on
monitoring or incident response. I want to explore just what it would take to
make this foot hold of arbitrary code execution on our application web server
useless.

There are two routes I see to making our system resilient. One involves some
crypto, the other involves a distributed system (and a tiny bit of crypto). The
ground rules we'll use when analyzing our proposed solutions: 1) No degrading
the functionality of our application, 2) We assume our attacker has an RCE
against Struts but no other exploits, anything else they accomplish should be
inherent in our design, 3) We're trying to prevent stealing 143 million
records, not stealing 14 records.

With those rules, let's forge ahead!

The one with crypto
-------------------

Immediately following a breach the battle cry of the uninformed is always the
same, "just encrypt it!". A little knowledge can be dangerous, serious security
practitioners know that even the strongest encryption algorithm means nothing
without a key management scheme that matches our threat model.

This fact should be evident from our problem description: we already had full
disk encryption enabled on our RDS database and TLS for data in transit! And of
course they do nothing in this attack scenario, full disk encryption protects
against someone with access to the physical disk or the raw block device, not
someone able to interact with the database; at that level the data has already
been decrypted. Any encryption scheme where the keys for every single row are
accessible from our web application will meet the same fate.

In short, we want to encrypt records under a key that is specific to that
record, and which the application server does not have ambient access to, the
application server must only have this key when the user is trying to access
their own records. How about a key derived from the user's password?

When the user logs in, we check their password against the stored hash in the
database, and if it's correct we compute a derived encryption key and store it
in the session. Now, whenever we need to read or write a record from the
database, we decrypt or encrypt it using this key. Now an attacker can steal
records that they see in the application server itself, but can't make off with
our entire database.

However, as described, this puts serious constraints on the rest of our system;
we can no longer have any backend systems which read or write the user's data.
Considering Equifax's entire business is storing data about people without
their direct involvement, this constraint is probably a no go, we need a scheme
that permits backends, disconnected from the web application, to access users'
data.

To accomplish this, we can introduce a second key. Now, instead of deriving a
key from the password and encrypting our data with that, we'll generate a
random key for each record, and encrypt that key with a key derived from the
password, and then store a second copy of that key, encrypted with Amazon's
KMS. That was probably hard to follow, let's try some pseudo-code:

.. code-block:: python

    raw_key = os.urandom(32)
    password_key = derive_key(password)

    password_encrypted_key = encrypt_with_key(key=password_key, data=raw_key)
    kms_encrypted_key = encrypt_with_kms(raw_key)

    encrypted_data = encrypt_with_key(key=raw_key, data=data)

    sql.execute(
        """
        INSERT INTO users (password_encrypted_key, kms_encrypted_key, encrypted_data)
        VALUES (?, ?, ?)
        """,
        (password_encrypted_key, kms_encrypted_key, encrypted_data)
    )

Now our web application can read and write data for the logged in user by
decrypting ``password_encrypted_key`` with the user's password when they log
in. Our backends can read and write the data for any user with
``kms_encrypted_key``. To prevent the web application from reading arbitrary
users' data with ``kms_encrypted_key`` we give it IAM permissions to *encrypt*
with our KMS key, but not *decrypt*. Our backend services run under a different
IAM user, which does have decrypt permissions.

This scheme works, but there are definitely disadvantages and caveats:

* There's crypto involved, so very few teams will have the expertise to build
  it without destroying all the security via a simple mistake.
* Bulk queries are no longer possible, if we need to search for records by a
  value which is encrypted, it needs to happen via a batch job that scans every
  single row.
* Adding additional permissions rules, such as "I would like to give access to
  my data to this other user" become significantly more complex.
* If an attacker is able to move laterally from our web application server to a
  backend processing sever, this blows away our defenses, so that becomes an
  important security boundary.

The one with a distributed system
---------------------------------

Our first approach was based on addressing the problem that with access to the
DB, you could read all the records. This approach is going to be based on
removing the ability to read arbitrary records from the DB from the web server.
To do that, we need to sever our application's access to the SQL database.

We'll introduce a service oriented architecture. Instead of our application
directly executing SQL queries against the DB, we'll have a service, on its own
isolated machine, that exposes APIs like ``get_user_for_ssn`` and queries the
DB for us. Instead of our web server having credentials for and a connection to
the database, it now has a connection to this backend RPC server. This means
the web server has no ability to execute ``SELECT * FROM users`` and walk off
with the data.

Ooops, except the space of SSNs is small enough that given our
``get_user_for_ssn`` method, one can just enumerate all possible SSNs and query
for them. We need to somehow bind a request to the user on whos behalf it's
being made (we'll call this the "principal"). Now our ``get_user_for_ssn`` RPC
method takes ``(principal, ssn)``, and the backend can perform authorization
checks that the ``principal`` is allowed to request that ``ssn``.

What is a principal? It's an assertion of the identity of the user who we're
making requests for. The simplest possible implementation would be just the
user's ID, except those are trivial to forge, so we need something that can't
just be ginned up out of thin air.

A more sophisticated implementation is
``principal = HMAC(K, "user-id=...") + "user-id=..."``, where ``K`` is a key
that both the login page and our RPC server share. The login page generates a
principal when a user logs in, and the RPC server validates the HMAC on
requests, and then performs the authorization checks. These principals can't
just be generated out of thin air, you need ``K``. If these look a lot like
signed cookies to you, that's because they are (and just like with a web
application's session, we could have also implemented this with a database
shared between our login page and the RPC server).

One small snag, right now our login page is part of our main application
server, so the box that our attacker is on has ``K``. We can solve this by
moving the login process -- validating a user's password and generating a
principal -- into its own service. Now the web application server has no
ability to generate principals to authorize requests to the backend service.
Problem solved!

Our attacker can, as always, steal principals for sessions that happen while
they are watching, but this affects a small portion of users out of our 143
million, and is basically an unavoidable problem. We can timebox the impact by
including a TTL in our principal that limits how long it can be used for, now
our attacker can only steal data for the lifetime of the principal, not for as
long as they're in our network.

If our backend services are built on Struts, we're still screwed. The same
exploit which got onto our service could be used to get into the login or
backend service, so we need to use a different technology stack. This is
reasonable. Building applications for the public web involves a lot of
complexity (HTML templates, ``Content-Type`` negotiation, localization, etc.).
Internal services don't require any of this functionality and therefore can
makes a lot of simplifying assumptions, so an RPC framework like `GRPC`_ or
`Apache Thrift`_ makes more sense. Even if we don't use a different technology
stack, this intermediate service gives us a valuable vantage point for
additional monitoring; for example, while a public server can expect to receive
many invalid requests everyday, an internal server is not, so aggressive
logging of malformed requests gives us an opportunity to catch our attacker
exploring the attack surface.

Conclusion
----------

We've just designed two alternate architectures that make us resilient to RCE
in our web application. A vulnerability like the one in Apache Struts which was
Equifax's downfall can no longer be used to steal all of our data. We've also
seen that it's difficult; both of these designs are objectively more complex
than the one we started with, and require expertise in distributed systems and
cryptography. That sort of talent is unfortunately rare. While this post
focused on prevention, it's important to recognize that detection and incident
response are critical components of a complete security strategy.

If you want to explore more into these topics, I recommend reading up on
`Kerberos`_, `Macaroons`_, and `BeyondCorp`_. I hope that eventually we
grow mature open source frameworks for building systems like these, in the same
way Django and other web frameworks provided defenses against XSS, SQL
injection, and CSRF out of the box. In the meantime, the next time you go to
mock Equifax, ask yourself: could your systems survive an RCE on your web
server? And if not, do you at least know when your dependencies have critical
security vulnerabilities?

.. _`GRPC`: https://grpc.io/
.. _`Apache Thrift`: https://thrift.apache.org/
.. _`Kerberos`: https://web.mit.edu/kerberos/dialogue.html
.. _`Macaroons`: https://air.mozilla.org/macaroons-cookies-with-contextual-caveats-for-decentralized-authorization-in-the-cloud/
.. _`BeyondCorp`: https://cloud.google.com/beyondcorp/#researchPapers
