Security wants for 2019
=======================

About 3 years ago I wrote about `five projects I thought were very important for
advancing the state of computer security`_. Looking back at that old post, I was
reasonably pleased to find that all are having real positive impacts and none
turned out to be busts. So I wanted to take a stab at writing down the things I
want to see happen in 2019, in the hopes that the universe will provide a few of
them. I've attempted to make each of these be something that could realistically
be accomplished in a year, and I've tried to include some success criteria for
each one. They are in no particular order.

Rust breakthrough
-----------------

I'm very bullish on Rust as a major component of our collective strategy for
dealing with the scourge of memory corruption, as `I've written about before`_.
I've been very encouraged by Rust's growth to date. My hope is that 2019 is a
break out year for Rust, with significant adoption in a domain I'm passionate
about, systems software traditionally written in C/C++: browsers, OS kernels,
hypervisors, and other similar software that's responsible for implementing a
strong security boundary.

Success criteria:

* Adoption of Rust as an official development language by another major OS and
  browser.
* Public talks/writing from teams that adopted Rust in these domains describing
  the value it added from a security perspective.

Security key breakthrough
-------------------------

Like the last point, I'm a big believer in the power of security keys (U2F) to
address the scourge of phishing. I want more consumer awareness of security
keys, more large deployments at companies, and more products to support them.

Success criteria:

* Google Accounts, Facebook, and Twitter switching from the legacy u2f.js API to
  webauthn.
* Every major browser ships webauthn in a stable release. NFC security key
  support on iOS.
* 25% drop in cost for keys (currently $10 for USB only, $17 for NFC, $25 for
  BLE+NFC). This would mostly be accomplished by removing the need for BLE keys,
  via iOS support for NFC keys, but cost reduction in other variants would also
  be valuable.
* My bank to offer U2F support.
* Just once when talking to someone about security, I'd like them to have heard
  of security keys before I spoke to them.

TLS 1.3 to the moon
-------------------

The IETF shipped the final `TLS 1.3 spec`_ in 2018. In 2019 I want adoption to
take off. TLS 1.3 makes a number of security improvements over previous
versions: better key handling for session resumption and removal of poor
ciphersuites (such as CBC mode ciphers and RSA key exchange). It also makes
performance improvements. Further, in 2018 all the major browsers announced they
were planning on retiring TLS 1.0 and 1.1 in 2019, I hope that continues
according to plan.

Success criteria:

* TLS 1.3 jumps from 5.7% of TLS connections in Firefox to 15%.
* TLS 1.0+1.1 drop from 1.2% of TLS connections in Firefox to less than 0.1%.

Who builds secure software?
---------------------------

The most secure email provider (Gmail) and consumer operating system (ChromeOS)
are built by an incredibly powerful surveillance apparatus. Commercial
surveillance is every bit as much a part of security as government surveillance.
Our industry, and society more broadly, should have a conversation about how it
came to be that some of our most secure tools have this potentially dangerous
provenance. Is it possible to fund secure software development without a
mostly-unrelated spigot of money?

Urgency around exploitation
---------------------------

Right now vulnerabilities (almost certainly memory corruption!) in cell phones
are being used to surveil `human rights activists in the Middle East`_ and a
`nutritional epidemiologist investigating the impact of soda consumption on
obesity`_. Yes, you read that correctly. These are just two of what are many
many stories of horrific abuses of software.

We should feel the same urgency around this exploitation as we do around an
attacker who has breached our network and is exfiltrating data. We need to make
exploitation so much more expensive that it is impractical as a tool of petty
despots.

User agency first
-----------------

Software is getting more complex at a rapid clip. If you're not responsible for
building it (and take a healthy interest in things you don't work on), it's fair
to say that it's literally impossible to know how anything on your computer
works. Worse though, it's rapidly becoming impossible to know *what* it is
doing, much less *how*. I've lost track of the number of times I've had to
answer "how did Facebook know to show me an ad for that?"

The ultimate purpose of security is to build systems that are worthy of our
trust. How can users trust mean anything if they can't `understand what software
is doing on their behalf?`_.

.. _`five projects I thought were very important for advancing the state of computer security`: https://alexgaynor.net/2015/nov/28/5-critical-security-projects/
.. _`I've written about before`: https://alexgaynor.net/2017/nov/20/a-vulnerability-by-any-other-name/
.. _`TLS 1.3 spec`: https://tools.ietf.org/html/rfc8446
.. _`human rights activists in the Middle East`: https://citizenlab.ca/2016/08/million-dollar-dissident-iphone-zero-day-nso-group-uae/
.. _`nutritional epidemiologist investigating the impact of soda consumption on obesity`: https://citizenlab.ca/2017/02/bittersweet-nso-mexico-spyware/
.. _`understand what software is doing on their behalf?`: https://glyph.twistedmatrix.com/2005/11/ethics-for-programmers-primum-non.html
