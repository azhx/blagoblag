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

Forward secrecy is a cryptographic property best known about in the context of
TLS. TLS's cipher suites can be divided into two groups: ones that use RSA key
exchange, and ones that use Diffie Hellman key exchange.

TLS connections are divided into two phases, a handshake phase, where the
server and client agree on a key and perform some authentication, and the
record phase where the contents of the connection are transferred (e.g. your
HTTPS requests and responses).

When the RSA key exchange is used, the way the client and server agree on a key
is that the client randomly generates a new key, and then encrypts it to the
server's public key. The server uses its corresponding private key to decrypt
the message. Now both sides have the same key, which they use to encrypt
messages in the record phase. The server will also sign the contents of the
handshake, to prove it's the same person as its certificate says.

In the Diffie Hellman key exchange, things work a little bit differently. Here,
both the client and server will generate a new Diffie Hellman public/private
keypair. Then both sides will send their public keys to the other, and each
side performs the Diffie Hellman algorithm to compute a shared key. The server
still has an RSA public key, but now it's only used for the signature.

Why do we prefer this second one, you might ask? It sounds a bit more
complicated, we've got two kinds of keys instead of just one, and besides,
encrypting things with RSA public keys and then decrypting them with private
keys is solid crypto. The algorithm is solid, if we use modern key size, it's
strong against any attacker, no one can decrypt these messages or crack the
key.

The answer is in forward secrecy. In the Diffie Hellman version, for each
connection the server and client generate new keys, so as soon as the
connection is done, they can throw them away. In the RSA key exchange, we're
encrypting things for the servers *long term* public key. The RSA key is a part
of a website's certificate, so it can't just be thrown away. And if somewhere
down the line, maybe even years, the key gets stolen, the attack can go back in
time and decrypt all the traffic.

Wait what!? Imagine you make a TLS connection to a server with the RSA key
exchange, you encrypt your record key to the server, you send some data, you
get some data, you go on your merry way. While you're doing that, an attacker
records all your TCP traffic. They can't do anything with it of course, it's
all encrypted and the crypto is solid. A year later, `HeartBleed`_ happens. The
same attacker uses HeartBleed to steal the server's private key, and uses that
to go back in time and decrypt your traffic from a year ago.

If the server was using Diffie Hellman, you'd have been fine, the RSA key is
used for signatures only with Diffie Hellman, and a year later your
connection's keys are nowhere to be found in memory. But RSA key exchange put
you at risk. Forward secrecy means using your long term keys only for
authentication, encryption should always use short term keys.

As a result, the web has been working hard to get servers to migrate from  RSA
key exchange to Diffie Hellman. The upcoming TLS 1.3 standard doesn't even
support RSA key exchanges anymore.

So, it's a clear win for TLS. What's this got to do with GPG encrypted emails?
GPG uses the same model as the RSA key exchange, except worse! In GPG, you get
your peer's public key from a keyserver, then you generate a random AES key,
encrypt the AES key with the RSA public key, and then encrypt my email with the
AES key. It has the exact same problem, if your RSA key is compromised, even
years later, if you have the ciphertext, you can decrypt the message. What
makes this even worse than TLS? People tend to rotate their GPG keys incredibly
infrequently, a person's GPG key might be in use for a decade or more.

Luckily, modern message protocols solve this problem. The most famous (and
widely deployed) of these protocols is the Signal Protocol, which is used by
Signal and WhatsApp. The Signal protocol uses the Diffie Hellman key exchange
on *every single message* you send. This means if I steal the key used to
encrypt one of your messages, I can't decrypt anything that came before it (I
can decrypt things that come after it though). There's also *no* long term key
you can steal and decrypt *everything* with.

However, I still think I've undersold the importance of forward secrecy.
Because forward secrecy means being able to delete your message. Right now, if
I delete an email that's been GPG encrypted to mean, I haven't *really* deleted
anything. The attacker who steals my email still has it (there's always an
attacker, that's why I encrypt things). The only way to make sure that email is
gone for good is for me to delete my GPG private key permanently. With a
forward secure messaging system, I can delete any messages I want and as long
as my chatting partner deletes them as well, they're gone for good, no keys
lying around for years, putting my privacy at risk.

It's popular to make fun of the security guarnatees of SnapChat, since they're
in no way resistant to a malicious chatting partner. But the ability to
securely, and meaningfully, delete messages is a huge win for many threat
models. Whether you're a dissident whose phone is being searched by a
repressive regime, or a White House employee whose phone is being searchd by
your boss, you want to be able to delete your messages.

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
