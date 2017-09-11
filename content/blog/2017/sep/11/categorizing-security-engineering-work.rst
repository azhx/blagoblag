Categorizing Security Engineering Work
======================================

There's a lot of different types of work that tend to get put into the bucket
"security engineering". This goal of this post is to describe how I categorize
different kinds of work, and why this is useful.

At the highest level, security work goes into one of four buckets:

1. Work that prevents us from getting owned. In this bucket are things like
   fixing bugs as well fixing root causes so bugs don't appear. For example,
   adding an ``html_escape`` to prevent XSS goes in this bucket, as does
   changing template languages to one that HTML escapes variables by default.
2. Work that lets us know if we've been owned. This bucket contains logging,
   monitoring, and alerting. For example, logging all outgoing TCP connections
   from your servers and alerting on unexpected hosts, or enabling HTTP
   Public-Key-Pinning in Report-Only mode.
3. Work that reduces the damage if we get owned. This bucket contains defensive
   mitigations and things that reduce the value of assets you own. For example,
   storing passwords in your database with ``scrypt``, and making sure the
   OAuth2 tokens you store have as limited scopes as possible both go in this
   bucket. This bucket sometimes bleeds into (1). For example, I work on
   Firefox's Sandboxing team, our work reduces the damage that can be done with
   a remote code execution vulnerability in Firefox's renderer process, but we
   also make it more difficult to own our user's systems completely, by
   requiring an attack have a second vulnerability.
4. Work that creates more work. In this bucket we have audits, and bug
   bounties, red team exercises, and threat modeling . None of these activities
   improve your security directly, instead they generate work in one of the
   other buckets for your team to work on. For example, a red teaming exercise
   might discover that your monitoring does a poor job of caching lateral
   movement between servers, or a bug bounty might turn up a pile of XSS
   vulnerabilities that need fixing.

Those are the buckets, but why are they useful? First, because like breakfast,
security requires balance. Over-investing in one bucket to the detriment of the
others produces suboptimal results. Second, because otherwise you might
implicitly mix them up. Several times I have seen teams that were overworked
and unable to keep up with the bugs they know about think a bug bounty would
solve their security problems. It won't, it will just generate more bugs that
they'll be unable to keep to speed with. The flip side is that if you aren't
sure of the weaknesses of your monitoring, a red teaming exercise can give you
critical insights into how attackers view your systems, and let you better plan
out the next round of work you need to do.

These buckets are neither universal nor perfect -- I suspect you can already
think of work that fits into multiple buckets or which doesn't fit cleanly into
any one of them -- nonetheless I've found them a useful system for quickly
categorizing how a security team is spending its time, and where it ought to be
spending its time.
