Forward secrecy is the most important thing
===========================================

An alternate title for this post would be "Why GPG isn't ok in 2017".

Imagine you were designing a new encrypted messaging system, what kinds of
things would you worry about? You'd want to make sure you were using good
encryption algorithms, authentication for senders, a high quality random number
generator, maybe you'd spend some time thinking about side channels for things
like message length. Unfortunately, if you're thinking about your protocol in
the context of something like email, it's very likely that you're not thinking
about forward secrecy.

GPG is a textbook example of this. GPG's cryptographic model is centered around
long-term RSA keys. I get your public key from a keyserver, I generate a random
AES key and encrypt that key with your public key, and then I encrypt my email
with the AES key. When you receive my email, you decrypt the AES key with your
RSA private key, and then you decrypt the contents with the AES key. So what's
the problem? RSA is a solid algorithm, AES is a solid algorithm, if we use
modern key size and padding it's strong against any attacker, no one can
decrypt these messages or crack the key.

The problem is that if anything happens to our private key, even years later,
it can ruin the confidentiality of our messages. Imagine you send someone a GPG
encrypted email. Someone's recording all your TCP traffic, but it's not a big
deal because the message is encrypted. Then a year later, your friend's hard
drive gets stolen, and with it their GPG key. Now our attacker can use that to
go back in time a year and decrypt the email in the recorded traffic from a
year ago. People tend to rotate their GPG keys incredible infrequently
(arguably, this behavior is *incentivized* by the web of trust model, which
demands a persistent key), so these keys live for a decade or more, putting all
the data encrypted with them at risk.

Luckily, modern message protocols solve this problem. The most famous (and
widely deployed) of these protocols is the Signal Protocol, which is used by
Signal and WhatsApp. The Signal protocol uses the Diffie Hellman key exchange
to achieve Forward Secrecy. This means that every message is encrypted with a
fresh key, and then that key is thrown away. An ephemeral key exchange is used
to get the key to your peer, but all of its parameters can also be thrown away
as soon as it's finished. Morever, the Signal protocol does one of these on
*every single message* you send. This means if I steal the key used to encrypt
one of your messages, I can't decrypt anything that came before it (I can
decrypt things that come after it though). There's also *no* long term key you
can steal and decrypt *everything* with. Because of this, forward secrecy is
often defined as using long term keys for authentication, and short term keys
for encryption.

However, I think this undersells the importance of forward secrecy. Because
forward secrecy means being able to delete your message. Right now, if I delete
an email that's been GPG encrypted to mean, I haven't *really* deleted
anything. The attacker who steals my email still has it (there's always an
attacker, that's why I encrypt things). The only way to make sure that email is
gone for good is for me to delete my GPG private key permanently. With a
forward secure messaging system, I can delete any messages I want and as long
as my chatting partner deletes them as well, they're gone for good, no keys
lying around for years, putting my privacy at risk.

It's popular to make fun of the security guarantees of SnapChat, since they're
in no way resistant to a malicious chatting partner. But the ability to
securely, and meaningfully, delete messages is a huge win for many threat
models. Whether you're a dissident whose phone is being searched by a
repressive regime, or a White House employee whose phone is being searchd by
your boss, you want to be able to delete your messages.

The TLS ecosystem has been making this exact migration to protect user privacy.
Current versions of TLS offer two forms of key exchange, RSA key exchanges with
this major drawback, and Diffie Hellman key exchanges which offer forward
secrecy. The upcoming TLS 1.3 specification completely drops the RSA key
exchange, making TLS always forward secure.

And this is why I say GPG encryption isn't ok in 2017, there's no ability to
delete messages without deleting my key, and deleting things is critical to
protecting my privacy. This criticism doesn't just apply to GPG of course,
almost all "secure email systems" are built on the same cryptographic choices,
which offer no forward secrecy. If you're designing a new secure messaging
system, use forward secure protocols to protect your users' privacy. If you're
choosing a messaging system to use, stick to ones with a forward secure
protocol (such as Signal) to protect *your* privacy.

PS: You can `donate to support Signal's development`_.

.. _`HeartBleed`: http://heartbleed.com/
.. _`donate to support Signal's development`: https://freedom.press/crowdfunding/signal/
