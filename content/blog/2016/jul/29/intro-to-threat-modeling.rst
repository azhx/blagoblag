Intro to threat modeling
========================

What is threat modeling? Threat modeling is a computer security technique to
help defenders (that's you, I assume) understand their own systems and drive
the process of building better defenses.

Core to the idea of a threat model is the idea that the things you need to do
to protect yourself vary depending on what you're defending against. Therefore,
threat modeling forces you to be explicit about *who* you're going to defend
*against*. This may sound obvious, however it's an extremely common trap to
fall into trying to defend against everything, or to try defend against `"sexy"
attackers`_, and ignore your most `immediate concerns`_. This is problematic
because it will lead you astray of solving real problems to protect yourself.

To begin, you want to start with enumerating what assets you're trying to
protect. This is something like "all the personal information in our database"
or "our user's ability to visit our website and get the right content" (i.e.
availability and integrity).

Next, we want to enumerate vectors by which someone could compromise one of
assets. For example, SQLi in our application code could compromise our
database. Our private key could be comprised via RCE vulnabilities in a process
or arbitrary file disclosure.

Vectors will also be recursive; "compromise my laptop" is both a vector to
getting my credentials, and also something which needs specific technical
vectors to accomplish. Your list of vectors will probably be long, really long.
That's ok. You are *not* making a list of everything you're going to need to
fix, you're making of list of potential ways your assets could be compromised.

Finally, and most importantly, we want to enumerate who our attackers are. We
want to do this in terms of two things, their capabilities and their
motivations. For example, if an attacker might be a hacktivists whose
motivation is defacing my website. Their capabilities might include phishing
and "metasploit"-style attacks, but not novel research or significant
computational resources. Another attacker might be a criminal organization,
motivated by access to PII that can be turned into money (social security
numbers, credit cards, etc.). Their capabilities might include computational
resources and previously breached password databases.

Thus far this has been pretty theoretical. Let me show a basic example for this
blog.

**Assets:** The integrity of the contents of this site. Also
accounts/credentials necessary to operate this site (e.g. account with the
hosting provider, HTTPS certificates, etc.)

**Vectors:** Integrity can be attacked at any of: my local machine with the
site's repository, the hosted repository with the static contents (Github), the
host the HTTP server runs on (Heroku), the CDN that terminates end user
connections (CloudFlare).

Any sort of local compromise of my laptop could modify the local repo in any
sort of way (and either push to Github or deploy to Heroku). My credentials to
any of these could be compromised by phishing (except Github, where I use a
Yubikey; and in practice I use a password manager for all which dramatically
reduces the risk of phishing). Similarly, password reuse attacks don't exist
because they're mitigated by 2fa on all service providers and lack of password
reuse.

A direct compromise of the service provider's infrastructure could be
used to compromise the integrity of the contents.

Finally, an attacker able to MITM connections between a visitor and the CDN or
the CDN and the origin could compromise the integrity of the contents.

**Attackers:**

(It's not clear that *any* attacker is interested in my site specifically, but
we're listing the attackers I would want to be able to resist.)

* Script kiddies, motivated by "looking cool" and defacing my site.
  Capabilities include metasploit-style attacks.
* Hacktivists, motivated by defacing my site for political purposes.
  Capabilities include phishing and metasploit-style attack.

So you've made some lists? Now what? Now you ruthlessly prioritize your
investments in your security, based on what your threat model says about your
attackers. Every minute or dime you spend on security should be addressing
something *in your threat model*. Use your model of attackers and assets to
cull things from the vectors list that cannot be attacked within those.

Prioritize your vectors by "cost to exploit", where cost is function of effort,
raw dollars, and necessary capabilities, and opportunity cost. For example,
using a 0-day you personally researched might be medium effort, and "free", but
it would still be a very high cost because the it requires advanced capability
to achieve and the opportunity cost of using an nginx 0-day on my site is high
because there are more valuable targets.

A prioritized list of vectors for my blog (ignoring some mitigations I've
already implemented) might look like:

* Phishing my credentials to Github, CloudFlare, or Heroku
* Attacking a vulnerability in a software component of the blog
* Owning my laptop (breaking this down into constituent vectors is an exercise
  for the reader)

Attacks such as compromising Github, CloudFlare, or Heroku are not included
because the capabilities required to do so are outside my threat model. Now
that I have a prioritized list of vectors within my threat model, I can start
planning further mitigation activities.

An extremely common experience I've had trying to secure services is working
with people who do not think through what they're trying to defend against, and
end up trying to craft defenses against attackers who can already execute
arbitrary code on a server, or who can break best-in-class cryptography,
instead of for their real problem, which is that none of the developers on
their team know what XSS is. A threat model is a crucial tool in avoiding this
type of mistake.

Threat modeling can be as high-level or as detailed as you want it to be. It is
an invaluable tool in securing yourself, your company, and your code, and
there's no time like the present to start being explicit about what you're
trying to protect, and who you're trying to protect it against.

.. _`"sexy" attackers`: https://en.wikipedia.org/wiki/Advanced_persistent_threat
.. _`immediate concerns`: https://en.wikipedia.org/wiki/Script_kiddie
