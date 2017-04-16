Certificate Transparency for Server Operators
=============================================

Say you've got a website, and because you care about protecting the privacy of
your visitors and the integrity of the content you serve to them, your website
is served over HTTPS. If your website is particularly high impact, you might be
concerned about some other CA misissuing a certificate for your domain; or
perhaps you have a very distributed engineering organization and it's hard to
keep track of who is issuing what certs.

In cases like this, it'd be helpful to be able to see a list of all
certificates ever issued. That's where Certificate Transparency comes in.

This post won't go into the details of how Certificate Transparency works, but
the short version is that people (can be CAs, server operators, or anyone else)
can submit certificates to a log, and the log provides cryptographic guarnatees
that once something is submitted, it can't be removed.

Certificate Transparency provides a huge boon for the ecosystem, because it
makes it easier to discover misissuance and other challenges. Chrome began
requiring Certificate Transparency for EV certificates in January 2015.
Following misissuance by Symantec, `Chrome began requiring Ceritifcate
Transparency for Symantec certificates`_ in 2016. And Google has announced that
Certificate Transparency will be required for all certificates starting in
October 2017.

This post will review the tooling around Certificate Transparency that exists
for server operators.

The first thing you probably want to do is check if a certificate you have has
been submitted to a log. The easiest way to do that is with a tool called
`crt.sh`_. ``crt.sh`` is a database compiled from all major Certificate
Transparency logs. You can query it to see if your certificate is listed there,
it will show details about the certificate, including which logs it was
submitted to. If it's not listed, you can submit your certificate to a log
yourself, though doing so is outside the scope of this article.

When a certificate is submmitted to a Certificate Transparecy log, the
submitted receives a Signed Certificate Timestamp (SCT). SCTs are cryptographic
assertions that a certificate has been submitted to a log, and will be publicly
visible in the log shortly.

Now that your certificate is included in a log, you'll want to serve your SCTs
to people who access your website. Just being included in a log is not
sufficient to meet Chrome's Certificate Transparency requirement, you must be
serving the SCTs. There are three ways of doing this:

1) Embed the SCTs in your certificate (your CA has to do this)
2) Embed the SCTs in your TLS handshake
3) Embed the SCTs in a stapled OCSP response

You can see if you are serving SCTs from the Google Chrome Developer Tools,
Security Tab:

.. image::

There exist modules for both `Apache`_ and `Nginx`_ to enable option (2).
Unfortunately the tooling here is still a little raw. I'm hopeful that popular
webservers will get support for handling SCTs automatically. It should be
possible for servers to do this without requiring any additional configuration
from operators.

Now that you've got your server serving SCTs, the final thing you'll want to do
is use a CT monitor to get notified when a CA issues a certificate for your
domain. Two choices for CT monitor are `Facebook`_ and `Cert Spotter`_. Both
are pretty simple, enter a domain name and you'll be notified when a
certificate is issued for that domain (including subdomains).


.. _`Chrome began requiring Ceritifcate Transparency for Symantec certificates`: https://security.googleblog.com/2015/10/sustaining-digital-certificate-security.html
.. _`crt.sh`: https://crt.sh/
.. _`Apache`: https://httpd.apache.org/docs/trunk/mod/mod_ssl_ct.html
.. _`Nginx`: https://github.com/grahamedgecombe/nginx-ct
.. _`Facebook`: https://developers.facebook.com/tools/ct/
.. _`Cert Spotter`: https://sslmate.com/certspotter/
