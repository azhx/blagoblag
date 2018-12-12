Optimize for Auditability
=========================

When we write code, we optimize for many different things. We optimize for
writability: how easy it is to write the code in the first place? We optimize
for maintainability: how easy it is to make ongoing changes? We optimize for
readability: how easy it is to understand what the code does?

However, we rarely optimize for auditability: how easy it is to tell if the code
has a security vulnerability? By ignoring this aspect of software design we
increase the burden on people reviewing code for vulnerabilities, which reduces
the overall security of our software.

Some might ask, why optimize for auditability? After all, isn't it the same as
readability? And if we know how to make bugs obvious, why not just fix them
instead? Both of these have the same answer: it's very easy to write code whose
intended purpose is clear to any engineer, but which has vulnerabilities that
only a security expert will recognize. Auditability means designing languages,
APIs, and patterns such that places in the code which are deserving of more
stringent review are clearly delineated. This allows an auditor to focus their
time as much as possible.

I have explored auditability as a goal in two different security contexts:
cryptography and memory unsafety. In both contexts, I've found that code which
was written with auditability in mind has allowed me to perform audits
significantly more quickly and to have much higher confidence that the audit
found everything it should have.

Cryptography
------------

When we built the `pyca/cryptography`_ Python library, auditability was a core
design criteria for our API. We were responding to very negative experiences we
had with other libraries, where low-level APIs had defaults which were often
dangerous and always hindered review. An example of this is a symmetric block
cipher API with a default mode, or providing a default initialization vector
when using CBC mode. While the danger of insecure defaults (such as ECB mode, or
an all-zero IV) is clear, even places with acceptable defaults stymied reviews
because they made it more difficult to answer questions like "Which
cryptographic algorithms does this project use?"

As a result, we decided that we'd have two styles of APIs: low-level ones, where
users were required to explicitly specify every parameter and algorithm, and
high-level ones which should have no choices at all, and which clearly
documented how they were built. The goal was that auditors could easily do
things like:

* Find uses of high-level recipes and know they were implemented securely, and
  not requiring significant review.
* Search for known insecure algorithms such as MD5, ECB, or PKCS1v15 encryption
  very quickly.
* Assess whether an ``encrypt()`` function you were auditing was secure easily,
  by putting the cryptographic algorithms front and center.

Our API design strategy works. In auditing numerous applications and libraries
using `pyca/cryptography`_, I've found that I've been able to very easily
identify uses of poor algorithms, as well as limit the scope of code I needed to
consider when trying to answer higher level questions about protocols and
algorithm compositions.

Memory unsafety
---------------

Frequent readers of me will know I'm a big fan of Rust, and more broadly of
moving away from memory unsafe languages like C and C++ to memory safe
languages, be they Swift, Rust, or Go. One of the most common reactions I get
when I state my belief that Rust could produce an order of magnitude fewer
vulnerabilities is that Rust has ``unsafe``. And since ``unsafe`` allows memory
corruption vulnerabilities, that breaks the security of the entire system,
therefore Rust is really no better than C or C++. Leaving aside the somewhat
tortured logic here, ensuring that ``unsafe`` does not provide an unending
stream of vulnerabilities is an important task.

I've recently had the opportunity to audit a few medium-sized Rust code bases in
the thousands of lines of code range. They made significant use of ``unsafe``,
primarily for interacting with C code and unsafe system APIs. In each codebase I
found one memory corruption vulnerability, one of which was unexploitable and
the other was probably exploitable. In my experience this is fewer
vulnerabilities than I would have identified in similar codebases written in
C/C++, but the far more interesting element was how easy it was to perform the
audit.

For C/C++ codebases like this, I'd start my audit by identifying all the
entrypoints where untrusted data is introduced into the system, for example
socket reads, public API functions, or RPC handlers. Then for each of these,
depending on size, I'd try to fuzz it with something like `libFuzzer`_ and
manually review the code to look for vulnerabilities. For these Rust audits, I
took a dramatically different approach. I was only interested in memory
corruption vulnerabilities, so I simply grepped for ``"unsafe"`` and reviewed
each instance for vulnerabilities. In some cases this required reviewing
callers, callees, or other code within the module, but many sites could be
resolved just by examining the ``unsafe`` block itself. As a result, I was able
to perform these audits to a high level of confidence in just a few hours.

Requiring code which uses dangerous features that can cause memory corruption to
be within an ``unsafe`` block is a powerful form of optimizing for auditability.
With C/C++, code is guilty until proven innocent; any code could have memory
unsafety vulnerabilities until you've given it a look. Rust provides a sharp
contrast by making code innocent until proven guilty, unless it's within an
``unsafe`` block (or in the same module and responsible for maintaining
invariants that an ``unsafe`` block depends on), Rust code can be trusted to be
memory safe. This dramatically reduces the scope of code you need to audit.

Conclusion
----------

In both of these domains, optimizing for auditability lived up to my hopes. Code
which was optimized for auditability took less time to review, and when I
completed the reviews I was more confident I'd found everything there was to
Find. This dramatic improvement in the ability to identify security issues means
less bugs to become critical incidents.

Optimizing for auditability pairs well with APIs which are designed to be easy
to use securely. It should be a component of more teams' strategies for building
software that’s secure.

.. _`pyca/cryptography`: https://cryptography.io
.. _`libFuzzer`: https://llvm.org/docs/LibFuzzer.html
