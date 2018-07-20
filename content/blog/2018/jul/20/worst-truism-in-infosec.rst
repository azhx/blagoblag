The worst truism in information security
========================================

    Attackers just need one vulnerability, defenders need to be perfect

This may be the single most repeated truism in information security. Just this
week, a colleague invoked this, with the quip that those of us who've chosen
defense must be pretty dumb, given the challenge of that task, and the
possibility of an easier career in offense. There's just one problem: it's not
actually true, and it's harmful to reasoning about information security,
particularly for non-practitioners.

This presentation of information security suggests a simple dichotomy, systems
are either secure or insecure. And if any single vulnerability exists, a
system is insecure. Since no system of any significant size is completely free
of vulnerabilities, it must be that no secure systems exist, cue nihilism.

Skilled practitioners don't actually adopt this black and white, binary
attitude. Instead, their approach could be more likened to economics. They ask
questions like, "What are the costs to exploit this system?", "What skills are
required to discover and exploit vulnerabilities?", "How valuable are the
assets the systems protects?", "How can we use monitoring to give ourselves
more opportunities to stop attackers?"

These questions orient us towards the idea that defenders win when they make
compromising a system more expensive than it is profitable, not when they make
it invulnerable. And this framing invites us to consider an array of valuable
security activities that don't fit into a binary secure/not-secure narrative.
For example:

* privilege separation (such as sandboxing), which either reduces the assets
  available to an attacker or forces them to chain additional vulnerabilities
  together to achieve their goals
* asset depreciation (such as hashing passwords), which reduces the value of
  the assets an attacker can access
* exploit mitigations (such as Control Flow Integrity or Content Security
  Policies), which make exploiting vulnerabilities harder, or impossible
* detection and incident response, which allow identifying successful
  attackers, limiting the window of compromise

None of these tactics remove or prevent vulnerabilities, and would therefore
by rejected by a "defense's job is to make sure there are no vulnerabilities
for the attackers to find" approach. However, these are all incredibly
valuable activities for security teams, and lower the expected value of trying
to attack a system.

Monitoring and incident response in particular defy this binary
secure/insecure paradigm. They presuppose that exploitation is possible, and
instead raise costs by forcing attackers to either be stealthier, which
requires more time and greater skills, or be caught and stopped before the
attack actually accomplishes their goals. Investments here flip the "defenders
need to be perfect" truism on it's head, instead forcing attackers to be
perfect, lest one mistake alert the defense of their presence.

Approaching security as economics highlights the importance of accurate threat
modeling. If you overestimate your detection capability, or underestimate your
attackers' resources or how they value your assets, you cede an advantage to
them. It is always the case that a miscalibrated threat model adds risk, but
in an economics approach to security, there's the possibility that
misestimates will lead you to the conclusion that you have rigorously
determined your system to be appropriately secure, while your attackers would
describe it as anything but. There's no escaping the reality that if you plan
for script kiddies and the GRU shows up you're going to have a bad time, so
conservatism is a virtue [#]_.

Finding, fixing, and preventing bugs are not the only jobs of a security team.
The "attackers just need one vulnerability, defenders need to be perfect"
"truism" of information security misleads people, by framing them as the only
responsibility of a security team, and suggesting that anything short of
perfection is worthless. A more accurate summary of information security might
be: systems can have vulnerabilities, but as long as identifying
vulnerabilities and exploiting them without detection is more expensive than
the value of the assets the system protects, defenders are winning.

.. [#] This is particularly true when estimating the efficacy of exploit mitigations, where developers have a tendency to overestimate the challenges to bypassing them. The difficulty of exploiting a vulnerability should be empirically assessed with red-teaming, not guessed at.
