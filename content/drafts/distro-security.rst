Your Distro's Definition of a Security Issue is Wrong
=====================================================

Linux distributions are interesting pieces of software. Unlike traditional
commercial operating systems (Windows or OS X), they don't build the kernel or
the userspace applications [#]_. Instead, most of what actually gets built as a
part of a Linux distribution is tooling to bring everything together. Most
notably, this includes a package manager and a (usually curated) set of
packages that can be installed.

The repository of packages is central to a Linux distribution's value
proposition. Basically every distro makes a promise like: If you install
packages from our repository, we'll take responsibility to making sure that all
the security patches are applied and that things work nicely with each other.

Security packages as a service! It's a great promise; the burden on individual
developers and sysadmins to keep track of vulnerabilities in the myriad of
software packages they use would be enormous, no one would ever manage to ship
any software (or secure it)!

The distros appear to use a definition of security issues that is:

    A specific bug in a specific piece of software that results in a specific
    vulnerability (that someone, under some threat model, could exploit)

This is not unreasonable. Unfortunately, it's also not sufficient. In the era
of nearly ubiquotsly networked computing, there is a second set of security
issues: emergent ones which do not occur because any piece of software has a
vulnerability, but rather because a set of incentives and *features* have
resulted in a less secure ecosystem.

That's a bit abstract, let's look at some specific examples.

1024-bit root certificates
--------------------------

* Old OpenSSL's which don't handle chain certs preventing the removal of 1024-bit roots


SHA-256 certificates
--------------------

* Old NSS which don't handle something about the SHA256 migration preventing the removal of SHA1 certs

pip RCE
-------

* Debian's pip has a trivial RCE

PBKDF2 Performance
------------------

Conclusion
----------

Because of the narrow way in which distros define security issues, users of
their "stable" releases are generally running very old versions of things.
These versions do not have any specific vulnerabilities, nevertheless have
non-security bugs and missing features which block eco-system wide efforts to
improve security.

Linux distributors need to more aggressively upgrade packages, particularly
security sensitive ones, to newer versions; similarly to what the Python
community has done with `PEP 466`_.

In order to enable this, library authors need to be more rigorous in maintaing
backwards compatibility, including ABI compatibility. Distributors should not
be forced to choose between shipping something insecure and breaking the world
to secure it. The commmunity at large should be more acceptable of `tooling
which will enable better adherence to backwards compatibility`_.

Finally, Linux distributions should move towards deploying individual packages
as isolated containers, with their own copy of dependencies. It should be
possible for Debian developers to push out new versions of OpenSSL for Nginx
specifically, rather than for the entire OS. Debian developers are able to test
Nginx with a new version of OpenSSL, while they are not able to test it with
arbitrary software, this lets them be less conservative and defensive in
upgrades.

Most internet security standards and key open source security software was
developed in a time where security was less prioritized than it is today, and
with different threat models. As the software engineering community attempts to
roll out security improvements, it's important that we not burden these
migrations by continuing to use antiquated software.

.. [#] Though many companies which produce Linux distributions *also* pay people to work on the kernel and various components of one of the Linux userspace stacks.

.. _`PEP 466`: https://www.python.org/dev/peps/pep-0466/
.. _`tooling which will enable better adherence to backwards compatibility`: https://alexgaynor.net/2015/sep/03/telemetry-for-open-source/
