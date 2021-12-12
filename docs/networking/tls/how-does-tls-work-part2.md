---
title: "Part 2 - How Does TLS Work?"
summary: "This article deals with what happens when you make a TLS connection"
date: 2021-12-10
---

# Introduction

Now that we know [how to get a TLS certificate](how-does-tls-work-part1.md) from a trusted
certificate authority, we'll look at TLS from the perspective of a user who connects to a web server
and uses a website encrypted using TLS. This will give us a complete overview of how TLS works.

Here's what happened when we got our certificate — we created a private key, generated a CSR signed
using our private key which contains information about the domain for which we want a TLS
certificate, sent that CSR to a CA, placed the TLS certificate received from the CA on our web
server.

However, we didn't really learn about how the TLS protocol actually works. How is TLS 1.3 different
from 1.2? Can we encrypt the SNI and get better privacy with TLS 1.3? Is there some sort of
transparency about how CAs issue certificates? We're also not sure what intermediate certificates
are and what a "full chain" means.

# What Happens When You Visit `google.com`?

Although we could tackle this question from a lot of different perspectives[^1], we'll focus on how TLS
works when a user connects to a HTTPS website like `google.com`.

```
$ openssl s_client -showcerts -connect google.com:443 < /dev/null
depth=2 C = US, O = Google Trust Services LLC, CN = GTS Root R1
verify return:1
depth=1 C = US, O = Google Trust Services LLC, CN = GTS CA 1C3
verify return:1
depth=0 CN = *.google.com
verify return:1
CONNECTED(00000003)
---
Certificate chain
 0 s:CN = *.google.com
   i:C = US, O = Google Trust Services LLC, CN = GTS CA 1C3
-----BEGIN CERTIFICATE-----
...........................
<X.509 Certificate>........
<BASE 64 Encoded Data>.....
...........................
-----END CERTIFICATE-----
 1 s:C = US, O = Google Trust Services LLC, CN = GTS CA 1C3
   i:C = US, O = Google Trust Services LLC, CN = GTS Root R1
-----BEGIN CERTIFICATE-----
...........................
<X.509 Certificate>........
<BASE 64 Encoded Data>.....
...........................
-----END CERTIFICATE-----
 2 s:C = US, O = Google Trust Services LLC, CN = GTS Root R1
   i:C = BE, O = GlobalSign nv-sa, OU = Root CA, CN = GlobalSign Root CA
-----BEGIN CERTIFICATE-----
...........................
<X.509 Certificate>........
<BASE 64 Encoded Data>.....
...........................
-----END CERTIFICATE-----
Server certificate
subject=CN = *.google.com
issuer=C = US, O = Google Trust Services LLC, CN = GTS CA 1C3
---
No client certificate CA names sent
Peer signing digest: SHA256
Peer signature type: ECDSA
Server Temp Key: X25519, 253 bits
---
New, TLSv1.3, Cipher is TLS_AES_256_GCM_SHA384
Server public key is 256 bit
Secure Renegotiation IS NOT supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
Early data was not sent
Verify return code: 0 (ok)
---
DONE
```

## Certificate Chain

The first thing that stands out in this incredibly verbose output is the **certificate chain**. The
`-showcerts` flag gives us the X.509 base64 encoded TLS certificate of not just `google.com` but
also that of:

- `GTS CA 1C3`, signed and issued by `GTS Root R1`, and
- `GTS Root R1`, signed and issued by `GlobalSign Root CA`

`GTS CA 1C3` is an **intermediate CA**, a CA which has been authorized to sign certificates on
behalf of the **root CA** `GTS Root R1`. From what I can gather, it's mandatory for root CAs to have
at least one intermediate CA which should then be used to sign certificates. It's also interesting
to note that `GTS Root R1`, while being a root CA, has its certificate cross signed by `GlobalSign
Root CA`.

This is what we have.

![Certificate Chain - Dark Mode](/images/certificate_chain_dark.webp){ .dark-mode-img }
![Certificate Chain - Light Mode](/images/certificate_chain_light.webp){ .light-mode-img }

For all the scary warnings that web browsers present when we use self-signed certificates, it's
kinda ironic that root CA certificates can be self-signed certificates. It really makes you think
about the inherent trust that we place on these CAs and PKI of TLS.

In this case, the root CA certificate of `GTS Root R1` is cross-signed by `GlobalSign Root CA`.
However, you'll notice the certificate chain depth terminates at `GTS Root R1` and doesn't check for
the validity of `GlobalSign Root CA`. This makes us think that TLS clients need at least 3
certificates — server certificate of the domain they're visiting, the intermediate certificate which
signed the server certificate, and the root certificate which signed the intermediate certificate.

Interestingly, I noticed something strange while playing around with `-showcerts` on my own domain:

```
$ openssl s_client -connect router.ayushnix.com:443 < /dev/null 2>&1
depth=0 CN = router.ayushnix.com
verify error:num=20:unable to get local issuer certificate
verify return:1
depth=0 CN = router.ayushnix.com
verify error:num=21:unable to verify the first certificate
verify return:1
depth=0 CN = router.ayushnix.com
verify return:1
CONNECTED(00000003)
---
Certificate chain
 0 s:CN = router.ayushnix.com
   i:C = US, O = Let's Encrypt, CN = R3
---
```

Although Firefox doesn't complain about this and is able to fill in the blanks, `openssl s_client`
isn't able to find my intermediate CA and root CA certificate and starts crying. Apparently, my
mistake was to write this

```
ssl_certificate router.ayushnix.com.cer;
```

instead of

```
ssl_certificate fullchain.cer;
```

in my nginx web server. Providing the complete certificate chain instead of just the server
certificate gives us the expected output:

```
$ openssl s_client -connect router.ayushnix.com:443 < /dev/null
CONNECTED(00000003)
depth=2 C = US, O = Internet Security Research Group, CN = ISRG Root X1
verify return:1
depth=1 C = US, O = Let's Encrypt, CN = R3
verify return:1
depth=0 CN = router.ayushnix.com
verify return:1
---
Certificate chain
 0 s:CN = router.ayushnix.com
   i:C = US, O = Let's Encrypt, CN = R3
 1 s:C = US, O = Let's Encrypt, CN = R3
   i:C = US, O = Internet Security Research Group, CN = ISRG Root X1
 2 s:C = US, O = Internet Security Research Group, CN = ISRG Root X1
   i:O = Digital Signature Trust Co., CN = DST Root CA X3
---
```

[Here's](https://pki.goog/repository/) the web page that lists the root and intermediate CAs
operated by "Google Trust Services LLC". [Here's](https://letsencrypt.org/certificates/) the "chain
of trust" structure of Let's Encrypt.

A computing device with a reasonably modern operating system should provide a set of root
certificates out of the box to enable client programs like `curl` and `openssl` to make TLS
connections. If you're using Linux, you're most probably using the [set of root certificates
provided by Mozilla](https://ccadb-public.secure.force.com/mozilla/IncludedCACertificateReport).

Some clients like the Firefox web browser can automatically fetch intermediate certificates if a web
server doesn't provide one but it is a mistake on the part of the web server administrator if he
doesn't provide them considering intermediate certificates aren't installed by default in computing
devices.

If you're still wondering how the certificate offered by `google.com` is verified to be valid,
here's what happens — the digital signature at the end of the X.509 certificate offered by
`google.com` can be verified using the public key of `GTS CA 1C3`, which is present in intermediate
certificate belonging to `GTS CA 1C3`. Each digital signature in a certificate in the chain is
validated against the public key present in the parent certificate until we reach the root
certificate, `GTS Root R1`. At this point, we can accept that the connection is trustworthy or we
can go further down the chain[^2] and verify the root certificate of `GTS Root R1` which has been
cross signed using `GlobalSign Root CA` as well.

One of the reasons why intermediate certificates aren't provided by default is because they can be
revoked and new intermediate certificates can be issued by root CAs. However, *root certificates
can't be revoked*. If root certificates are needed to be distrusted, they have to be removed from
potentially billions of computing devices, which is obviously not a trivial or a pleasant task. It
would ultimately depend on the user to keep his web browser and operating system updated[^3].
Remember, a root CA holds a lot of power and trust because it can issue certificates for any domain
on the Internet. To put it rather blatantly, a root CA is an entity which creates self-signed
certificates that the world chooses to trust to offer certificates for our servers because
cryptography and PKI is hard.

To imagine the scale of the impact if a root CA gets compromised, take a look at what happened when
[DigiNotar was
compromised](https://slate.com/technology/2016/12/how-the-2011-hack-of-diginotar-changed-the-internets-infrastructure.html)
back in 2011.
[WoSign](https://blog.mozilla.org/security/2016/10/24/distrusting-new-wosign-and-startcom-certificates/)
and [CNNIC](https://blog.mozilla.org/security/2015/04/02/distrusting-new-cnnic-certificates/) had
their certificates revoked or distrusted as well.

## Ciphers Suites

You're probably aware about different versions of TLS going all the way back to SSL 1.0, which is
how the term 'SSL' still survives to this day, and the latest version, TLS 1.3. Each SSL/TLS version
supports a range of cipher suites that a client and a web server have to agree upon before an
encrypted and authenticated session can be established.

I'm not going to discuss anything besides [TLS 1.2](https://tls.ulfheim.net/) and [TLS
1.3](https://tls13.ulfheim.net/). If your application or operating system doesn't support TLS 1.3
yet, you should consider moving it to `/dev/null`. If you're running a web server, consider
supporting TLS 1.3 if you haven't already done it and, if possible, disable everything else[^4]
including TLS 1.2. There's no need to support Internet Explorer or ancient versions of modern web
browsers.

The most apparent difference between TLS 1.2 and TLS 1.3 in the `s_client` output is how the name of
the cipher suite is written.

```
$ openssl s_client -connect google.com:443 < /dev/null 2> /dev/null | rg 'TLS'
New, TLSv1.3, Cipher is TLS_AES_256_GCM_SHA384
$ openssl s_client -no_tls1_3 -connect google.com:443 < /dev/null 2> /dev/null | rg 'TLS'
New, TLSv1.2, Cipher is ECDHE-ECDSA-CHACHA20-POLY1305
```

The [RFC 8446](https://datatracker.ietf.org/doc/html/rfc8446#appendix-B.4) specification for TLS 1.3
has defined only 5 cipher suites, two of which seem to be unsupported by OpenSSL on my system.

Don't mind the `awk` filter. I've omitted some columns which aren't useful.

```
$ openssl ciphers -v -s -tls1_3 | awk '{$2=""; print $0}' | column -t
TLS_AES_256_GCM_SHA384        Kx=any  Au=any  Enc=AESGCM(256)             Mac=AEAD
TLS_CHACHA20_POLY1305_SHA256  Kx=any  Au=any  Enc=CHACHA20/POLY1305(256)  Mac=AEAD
TLS_AES_128_GCM_SHA256        Kx=any  Au=any  Enc=AESGCM(128)             Mac=AEAD
```

Yup, that's it. TLS 1.3 seems to support only 3 cipher suites on my system. For a stark contrast,
here's the supported list of cipher suites in TLS 1.2. The `-stdname` flag tells OpenSSL to use the
official IANA names, not its own made up names.

```
$ openssl ciphers -v -s -stdname -tls1_2 | awk '{$2=$3=$4=""; print $0}' | column -t
TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384        Kx=ECDH  Au=ECDSA  Enc=AESGCM(256)             Mac=AEAD
TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384          Kx=ECDH  Au=RSA    Enc=AESGCM(256)             Mac=AEAD
TLS_DHE_RSA_WITH_AES_256_GCM_SHA384            Kx=DH    Au=RSA    Enc=AESGCM(256)             Mac=AEAD
TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305_SHA256  Kx=ECDH  Au=ECDSA  Enc=CHACHA20/POLY1305(256)  Mac=AEAD
TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256    Kx=ECDH  Au=RSA    Enc=CHACHA20/POLY1305(256)  Mac=AEAD
TLS_DHE_RSA_WITH_CHACHA20_POLY1305_SHA256      Kx=DH    Au=RSA    Enc=CHACHA20/POLY1305(256)  Mac=AEAD
TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256        Kx=ECDH  Au=ECDSA  Enc=AESGCM(128)             Mac=AEAD
TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256          Kx=ECDH  Au=RSA    Enc=AESGCM(128)             Mac=AEAD
TLS_DHE_RSA_WITH_AES_128_GCM_SHA256            Kx=DH    Au=RSA    Enc=AESGCM(128)             Mac=AEAD
TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA384        Kx=ECDH  Au=ECDSA  Enc=AES(256)                Mac=SHA384
TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA384          Kx=ECDH  Au=RSA    Enc=AES(256)                Mac=SHA384
TLS_DHE_RSA_WITH_AES_256_CBC_SHA256            Kx=DH    Au=RSA    Enc=AES(256)                Mac=SHA256
TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256        Kx=ECDH  Au=ECDSA  Enc=AES(128)                Mac=SHA256
TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256          Kx=ECDH  Au=RSA    Enc=AES(128)                Mac=SHA256
TLS_DHE_RSA_WITH_AES_128_CBC_SHA256            Kx=DH    Au=RSA    Enc=AES(128)                Mac=SHA256
TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA           Kx=ECDH  Au=ECDSA  Enc=AES(256)                Mac=SHA1
TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA             Kx=ECDH  Au=RSA    Enc=AES(256)                Mac=SHA1
TLS_DHE_RSA_WITH_AES_256_CBC_SHA               Kx=DH    Au=RSA    Enc=AES(256)                Mac=SHA1
TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA           Kx=ECDH  Au=ECDSA  Enc=AES(128)                Mac=SHA1
TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA             Kx=ECDH  Au=RSA    Enc=AES(128)                Mac=SHA1
TLS_DHE_RSA_WITH_AES_128_CBC_SHA               Kx=DH    Au=RSA    Enc=AES(128)                Mac=SHA1
TLS_RSA_WITH_AES_256_GCM_SHA384                Kx=RSA   Au=RSA    Enc=AESGCM(256)             Mac=AEAD
TLS_RSA_WITH_AES_128_GCM_SHA256                Kx=RSA   Au=RSA    Enc=AESGCM(128)             Mac=AEAD
TLS_RSA_WITH_AES_256_CBC_SHA256                Kx=RSA   Au=RSA    Enc=AES(256)                Mac=SHA256
TLS_RSA_WITH_AES_128_CBC_SHA256                Kx=RSA   Au=RSA    Enc=AES(128)                Mac=SHA256
TLS_RSA_WITH_AES_256_CBC_SHA                   Kx=RSA   Au=RSA    Enc=AES(256)                Mac=SHA1
TLS_RSA_WITH_AES_128_CBC_SHA                   Kx=RSA   Au=RSA    Enc=AES(128)                Mac=SHA1
```

Yes, TLS 1.2 also seems to support some cipher suites which were introduced decades ago in SSL 3.0.

The format in which cipher suites are named in TLS 1.3 is `TLS_Enc_Mac` whereas TLS 1.2 uses
`TLS_Kx_Au_WITH_Enc_MAC`. If `Kx` and `Au` are the same, it's mentioned only once in TLS 1.2.
However, TLS 1.3 omits `Kx` and `Au` from the cipher suite names.

If we want to make TLS 1.2 better than what we have above, disable cipher suites with RSA key
exchange because it doesn't support [Perfect Forward
Secrecy](https://en.wikipedia.org/wiki/Forward_secrecy) (PFS). By using ephemeral keys that are
destroyed after every single connection, there's nothing to leak and nothing to decrypt after the
connection has been successfully completed. TLS 1.3 decided to limit key exchange to ECDHE.

Next, go ahead and [disable all cipher suites with
CBC](https://blog.cloudflare.com/padding-oracles-and-the-decline-of-cbc-mode-ciphersuites/). This is
something that TLS 1.3 has already done out of the box.

This is what we're left with in TLS 1.2. If you do have TLS 1.2 enabled in your web server, make
sure that these are the only supported cipher suites.

```
TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384        Kx=ECDH  Au=ECDSA  Enc=AESGCM(256)             Mac=AEAD
TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384          Kx=ECDH  Au=RSA    Enc=AESGCM(256)             Mac=AEAD
TLS_DHE_RSA_WITH_AES_256_GCM_SHA384            Kx=DH    Au=RSA    Enc=AESGCM(256)             Mac=AEAD
TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305_SHA256  Kx=ECDH  Au=ECDSA  Enc=CHACHA20/POLY1305(256)  Mac=AEAD
TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305_SHA256    Kx=ECDH  Au=RSA    Enc=CHACHA20/POLY1305(256)  Mac=AEAD
TLS_DHE_RSA_WITH_CHACHA20_POLY1305_SHA256      Kx=DH    Au=RSA    Enc=CHACHA20/POLY1305(256)  Mac=AEAD
TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256        Kx=ECDH  Au=ECDSA  Enc=AESGCM(128)             Mac=AEAD
TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256          Kx=ECDH  Au=RSA    Enc=AESGCM(128)             Mac=AEAD
TLS_DHE_RSA_WITH_AES_128_GCM_SHA256            Kx=DH    Au=RSA    Enc=AESGCM(128)             Mac=AEAD
```

You'll notice that we're left with cipher suites which only support
[AEAD](https://blog.cloudflare.com/it-takes-two-to-chacha-poly/), an approach towards authenticated
encryption that combines encryption and authentication schemes in a secure manner. TLS 1.3 defaults
to AEAD in its cipher suites.

## TLS Handshake

Although [this](https://tls.ulfheim.net/) page is an excellent reference to understand the entire
TLS 1.2 handshake process in detail, there's nothing better than using
[wireshark](https://gitlab.com/wireshark/wireshark) and looking at the TLS handshake as it happens
in real time.

Here's how the first part of a TLS 1.2 handshake, `Client Hello`, looks like in
[termshark](https://github.com/gcla/termshark):

![Wireshark Client Hello TLS 1.2 - Dark Mode](/images/client_hello_dark.webp){ .dark-mode-img }
![Wireshark Client Hello TLS 1.2 - Light Mode](/images/client_hello_light.webp){ .light-mode-img }

That being said, I think a brief summary of the handshake process would be beneficial
for future reference.

The arrows shown below indicate the direction in which a message is sent. A right arrow indicates
that a message was sent from a client to a server and vice versa.

### TLS 1.2

Here's what happens in the TLS 1.2 handshake:

:fontawesome-solid-long-arrow-alt-right: `Client Hello`

:   The client initiates a TLS connection by sending information to a server which includes the TLS
protocol version, random data, and a list of cipher suites that the client supports. It also
contains data for a lot of extensions including SNI and *TLS Renegotiation* information.

:fontawesome-solid-long-arrow-alt-left: `Server Hello`

:   The server, upon receiving the `Client Hello` message, sends back information of its own which
includes the TLS protcol version, the cipher suite, random data, and extension data that both the
server and the client support.

:fontawesome-solid-long-arrow-alt-left: `Server Key Exchange`

:   This is where the ephemeral DH keys are generated. The server generates and sends its ephemeral
public key (this is the `Kx` part in the cipher suite), its DER certificate, and a signature made
using the private key of the DER certificate (this is the `Au` part in the cipher suite) for the
combined message containing the ephemeral public key, random data sent by the client and the server,
and the curve information. The server then indicates that it has completed the initial phase of the
handshake from its end.

This was the first part of the handshake.

:fontawesome-solid-long-arrow-alt-right: `Client Key Exchange`

:   At this point, the client has everything it needs to generate a "pre-master secret", which will
in turn be used to generate a "master secret", which will be used for symmetric encryption of the
entire session. The client sends the ephemeral public key to the server so that it can come up with
this master key independently as well.

:fontawesome-solid-long-arrow-alt-right: `Client Change Cipher Spec`

:   The client tells the server that it has the master secret and that it will encrypt all further
communications using that secret/ s key.

:fontawesome-solid-long-arrow-alt-right: `Client Handshake Finished`

:   The client tells the server that it's finished with the client TLS handshake. It sends a hash of
all the handshake messages made until now. This can be called the `client_verify_data` value.

:fontawesome-solid-long-arrow-alt-left: `Server Change Cipher Spec`

:   Using the ephemeral public key sent by the client in the `Client Key Exchange` message, the
server has everything it needs to generate the same pre-master secret and the master secret that the
client has. The server informs the client that all further communications will be encrypted using
this key.

:fontawesome-solid-long-arrow-alt-left: `Server Handshake Finished`

:   The server tells the client that it's finished with the server TLS handshake. It sends a hash of
all the handshake messages made until now. This can be called the `server_verify_data` value.

This marks the end of the TLS 1.2 handshake process. After this, the client and server can exchange
data and the TLS session is closed.

One of the things we can notice is that the entire TLS handshake process isn't encrypted. The first
time encrypted traffic is sent over the wire is during `Client Change Cipher Spec`. We can also
notice that the TLS 1.2 handshake is a 2-RTT process. A 2-RTT might be acceptable for an initial
connection but we'd ideally want to reduce the RTT in subsequent connections to save time and costs.

Let's see what `s_client` tells us when we connect to `google.com` using TLS 1.2.

```
$ openssl s_client -no_tls1_3 -connect google.com:443 < /dev/null
...
...
New, TLSv1.2, Cipher is ECDHE-ECDSA-CHACHA20-POLY1305
Server public key is 256 bit
Secure Renegotiation IS supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
SSL-Session:
    Protocol  : TLSv1.2
    Cipher    : ECDHE-ECDSA-CHACHA20-POLY1305
    Session-ID: EDAA4E0FF38A0BFCDD85BB43513CE1930BE5DFE83A36F729BACFC3B9634817D6
    Session-ID-ctx:
    Master-Key: 76CD03B3CF4439B10FCF8F32FA2C92F75F40A97BAD19A1CEC04AEA36F8199E51...
    PSK identity: None
    PSK identity hint: None
    SRP username: None
    TLS session ticket lifetime hint: 100800 (seconds)
    TLS session ticket:
    0000 - 01 61 36 00 d8 38 02 a9-6c c0 ce 9e 61 4b b6 73   .a6..8..l...aK.s
    0010 - 69 7f 76 3c d7 d5 c2 8c-64 cd 83 be 14 92 26 86   i.v<....d.....&.
    xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx   xxxxxxxxxxxxxxxx

    Start Time: 1639318368
    Timeout   : 7200 (sec)
    Verify return code: 0 (ok)
    Extended master secret: yes
---
DONE
```

TLS 1.2 supports **Secure Renegotiation**, a method for either the client or the server to
*renegotiate* a TLS handshake while being inside the "same session" of the initial handshake. This
possibility could arise for a variety of reasons which include upgrading cipher suites for a
specific part of a web page (considering the actual URL the client wants to visit isn't known until
the initial TLS handshake is completed) or asking for client certificates for authentication. The
3rd section of [RFC 5746](https://datatracker.ietf.org/doc/html/rfc5746#section-3) goes into more
detail about how this happens. Essentially, it relies on the Secure Renegotiation TLS extension
parameter being present in the `Client Hello` and `Server Hello` messages. In addition, the client
and the server expect the `verify_data` values which were saved before during the initial handshake.

The introduction of secure renegotiation in TLS 1.2 was in response to an [injection
vulnerability](https://scribe.rip/8fc08f1d58ea) found in the legacy renegotiation capability present
in TLS 1.2 and earlier versions. A MitM was able to inject arbitrary/malicious data by intercepting
the initial handshake. After this was done, the MitM renegotiated the TLS handshake on behalf of the
client although the client thought that the second `Server Hello` response was actually the first
and the client sends its data as well. The combination of the MitM data and the client data could
end up harming the client.
[This](https://web.archive.org/web/20100406064650/http://www.educatedguesswork.org/2009/11/understanding_the_tls_renegoti.html)
post goes into more detail.

If there's something you want to take away from all this, it's that "Secure" Renegotiation isn't a
part of TLS 1.3.

TLS 1.2 also supports **Session IDs** and **Session Tickets**, two methods which can be used to
improve the performance of subsequent TLS 1.2 connections at the risk of compromising PFS. Both
methods cache the master key of the session and re-use it to reduce the TLS handshake from a 2-RTT
process to a 1-RTT process.

If session IDs are used, the server sends a unique string to the client, as shown above, and keeps
the cryptographic parameters and the master key of the session in question. If the client presents
that session ID in the subsequent connection, the server retrieves the needed cryptographic
parameters from its own storage, re-uses it, and finishes the TLS 1.2 handshake. It's important to
note that the initial transmission of the Session ID is encrypted but subsequent transmissions
aren't. Session IDs present three problems:

- you'll need to figure out a way to share the cached session information[^5] if you have multiple
  servers

- it poses resource scalability issues considering a server may need to store information for
  potentially millions of clients

- session IDs pose a single point of failure risk that can compromise PFS if the cached session
  information from the server is leaked

If session tickets are used, the server outsources the storage requirements to the client. If a
client sends the Session Ticket TLS extension in `Client Hello` during the initial handshake, the
server encrypts the cryptographic session parameters using a key known only to the server, and
sends the session ticket to the client at the end of the initial handshake. If the client sends a
valid session ticket in subsequent sessions, the handshake becomes a 1-RTT process. Session Tickets
pose similar problems:

- for some reason, the encrypted session ticket (which contains the master key) is sent in [clear
  text](https://datatracker.ietf.org/doc/html/rfc5077#section-3.1) just before `Server Change Cipher
  Spec`, which essentially makes PFS moot

- if running at scale, the session ticket still needs to be distributed among multiple servers in a
  secure manner[^6]

- although it doesn't look like a mandatory requirement, the
  [recommended](https://datatracker.ietf.org/doc/html/rfc5077#section-4) algorithm to encrypt the
  session key is `AES-128-CBC`, a cipher suite which we've already eliminated before

The
[recommendations](https://timtaubert.de/blog/2014/11/the-sad-state-of-server-side-tls-session-resumption-implementations/)
regarding the usage of TLS 1.2 session tickets usually involve focusing on regular key rotation at
short intervals or disabling them if not needed. However, TLS 1.2 session tickets, as they are, are
not included in TLS 1.3.

PSK and SRP are alternative methods for TLS session authentication. In the former case, a PSK is
shared between the client and the server before the TLS handshake is initiated. As the name
indicates, in the case of SRP, the client submits a username as well. I don't expect to see this
form of TLS authentication outside some niche corporate environments.

The **extended master secret** is yet another extension on top of the TLS 1.2 protocol to prevent a
MitM attack when an attacker can sit between a client and a server, act as a proxy, and be able to
generate the same master secret that a client and server would've. It looks like this attack is
possible when client certificates and session resumption and renegotiation are active.
[This](https://www.mitls.org/pages/attacks/3SHAKE) page goes into more detail about this attack.

### TLS 1.3

As indicated before, TLS 1.3 sheds a lot of legacy baggage and focuses on minimalism and security.
Here's how a TLS 1.3 connection looks like:

```
$ openssl s_client -connect google.com:443 < /dev/null
...
...
New, TLSv1.3, Cipher is TLS_AES_256_GCM_SHA384
Server public key is 256 bit
Secure Renegotiation IS NOT supported
Compression: NONE
Expansion: NONE
No ALPN negotiated
Early data was not sent
Verify return code: 0 (ok)
---
DONE
```

Yup, that's it. TLS 1.3 doesn't have much in the name of patchwork (for now). Secure Renegotiation
isn't supported, session IDs and session tickets are gone, RSA key exchange is gone as well.

This is a simplified version of how the TLS 1.3 handshake works:

:fontawesome-solid-long-arrow-alt-right: `Client Hello`

:   This message is mostly the same as it is in TLS 1.2 except that it includes the ephemeral public
key generated using (EC)DHE `Kx` method in this message. This is wrapped inside a new extension
called the `Key Share`. At this stage, the client tries to guess that the public key generated using
the curve of its choice will be acceptable to the client.

:fontawesome-solid-long-arrow-alt-left: `Server Hello`

:   This message is mostly the same as it is in TLS 1.2 except that it includes the ephemeral public
key generated using the `Kx` algorithm chosen by the client. This is wrapped inside a new extension
called the `Key Share`. If the curve used by the client isn't accepted by the server, it would send
a `Hello Retry Request` instead with the list of curves it supports.

At this point, both the server and the client have enough data to create a "handshake secret" and
the connection will be encrypted from this point.

:fontawesome-solid-long-arrow-alt-left: `Server Certificate`, `Server Certificate Verify`

:   The server sends its DER certificate, a signature on the hash of the handshake messages exchange
at this point, and verification data built using the hash of all previous messages and the handshake
secret generated earlier.

:fontawesome-solid-long-arrow-alt-left: `Server Handshake Finished`

:   The server sends a verification message using the handshake secret and the hash of all messages
so far.

At this point, the server and the client can generate the "master secret" of the session using the
handshake secret and the hash of all the messages so far. This also concludes the first RTT.
Interestingly, the server can send the application data that the client requested at this stage even
though the client hasn't sent a finished message of its own.

:fontawesome-solid-long-arrow-alt-right: `Client Handshake Finished`

:   The client sends a verification message using the handshake secret and the hash of all the
messages so far. This is proceeded by the client sending application data, if any, from its end.

Yes, TLS 1.3 managed to reduce the default handshake to just 1-RTT. The connection does fall back to
2-RTT if a server has to send the `Hello Retry Request` message.

Although session IDs and session tickets aren't a part of TLS 1.3, it does include a method for
session resumption using PSK. The server can send a session ID or a session ticket in an encrypted
form after the `Client Handshake Finished` message is received by the server. The client can send
this encrypted blob in the `Client Hello` message in subsequent sessions. This encrypted blob is
what we're calling the PSK.

Besides (EC)DHE, this PSK is another `Kx` method that TLS 1.3 supports. As you may have guessed, the
PSK `Kx` doesn't provide any sort of forward secrecy so the [usual
caveats](https://timtaubert.de/blog/2017/02/the-future-of-session-resumption/) about short lived
PSKs apply. However, this PSK can be coupled with (EC)DHE as another form of `Kx` method.

In an effort to provide even more speed, TLS 1.3 provides a **0-RTT** mode which, as expected,
trades security for [latency](https://blog.cloudflare.com/introducing-0-rtt/). In case you were
wondering, this is what the "early data" meant in the output from `s_client`. 0-RTT simulates
unencrypted HTTP requests when it comes to RTT.

You can think of 0-RTT as a client using the PSK sent by a server in subsequent messages to send an
"early data" extension in the `Client Hello` message and the application data encrypted using the
PSK. Since the client resorts to using only the PSK for the `Kx` and for sending application data,
0-RTT doesn't provide forward secrecy. It is also susceptible to [replay
attacks](https://timtaubert.de/blog/2015/11/more-privacy-less-latency-improved-handshakes-in-tls-13/)
where the attacker and intercept and send the application data in addition to the client sending
that data. To prevent that from happening, a server needs to limit the type of requests which are
eligible for 0-RTT, such as idempotent GET requests without query parameters, and maintain a cache
or record of a value that uniquely identifies the client.

[RFC 8446](https://datatracker.ietf.org/doc/html/rfc8446) is the canonical source of reference for
TLS 1.3. The [major differences](https://datatracker.ietf.org/doc/html/rfc8446#section-1.2) and
[updates](https://datatracker.ietf.org/doc/html/rfc8446#section-1.3) present a good overview of why
TLS 1.3 should be preferred over TLS 1.3.

For reference, the `Kx` algorithms supported by TLS 1.3 are (EC)DHE. The supported finite field
groups and curves are mentioned [here](https://datatracker.ietf.org/doc/html/rfc8446#section-4.2.7).
The `Au` algorithms supported are mentioned
[here](https://datatracker.ietf.org/doc/html/rfc8446#section-4.2.3). RSA-PSS, ECDSA, and EdDSA are
supported.

## Fixing SNI

Although we've managed to encrypt application data using TLS, an important piece of metadata has
always been bereft of encryption and this has enabled ISPs and governments around the world to
propagate surveillance and censorship.

The SNI extension in TLS is needed when a website is hosted on an IP address which is shared by
other websites with different domain names. For the web server to be able to provide the appropriate
certificate needed in a TLS handshake, it needs to know the domain for which the certificate should
be provided to the client for verification. Since this extension is part of the unencrypted `Client
Hello` message, it is open for exploitation.

![SNI - Dark Mode](/images/sni-dark.webp){ .dark-mode-img }
![SNI - Light Mode](/images/sni-light.webp){ .light-mode-img }

Typically, the domain to which a user wants to connect to appears in 3 different places — the DNS
query (which is often unencrypted and easy to exploit), the TLS SNI extension, and the HTTP host
header, which is encrypted using TLS. Even if we encrypt the DNS request using DoH/DoT/DNSCrypt, the
TLS SNI message is still visible to anyone who wants to look at it. Yes, this means that anyone who
sits between you and the websites your're connecting to can log the list of domain names you vist
and create a profile on you. Sure, you could use a VPN or Tor but that isn't really a solution to
this prevailing problem.

For a while, [domain fronting](https://en.wikipedia.org/wiki/Domain_fronting) was a popular method
to obfuscate the SNI value to popular CDN provider domain names while keeping the actual domain
intact in the HTTP host header but most big CDN providers have prohibited domain fronting. The
example on Wikipedia is quite good for explaining what domain fronting is:

```
$ curl -sSL -H 'Host: www.youtube.com' https://www.google.com | hq title data
<title>YouTube</title>
```

Although we're pretending to connect to Google Search (which is what the SNI will see), the HTTP
host header contains the domain name for YouTube and that's what we get[^7].

[ESNI](https://blog.cloudflare.com/encrypted-sni/) was introduced as an extension for TLS 1.3 to
encrypt the SNI in a TLS handshake by using a base64 encoded ESNI public key of the server. This
public key would be published as a DNS TXT record. Yes, sounds insane considering DNS is insecure
and unauthenticated by default but Cloudflare provides both DoH and DNSSEC for its domains. ESNI was
a draft and never really became standard and it has been replaced by **Encrypted Client Hello**,
abbreviated as [ECH](https://blog.cloudflare.com/encrypted-client-hello/), which also seems to be a
[draft](https://datatracker.ietf.org/doc/html/draft-ietf-tls-esni-13) as of December 2021. It's safe
to say, however, that ECH will need DoH/DoT/DNSCrypt to work as expected. We also may get a new DNS
record for distributing the ECH public keys called [HTTPS Resource
Record](https://datatracker.ietf.org/doc/html/draft-ietf-dnsop-svcb-https-08).

I'll revisit ECH when it's a formal specification rather than a draft. Hopefully, it becomes a
standard [soon](https://blog.cloudflare.com/handshake-encryption-endgame-an-ech-update/).

--8<-- "include/abbreviations.md"

[^1]:
If you're looking for a perspective in terms of DNS, [this](../dns/how-does-dns-work.md) article
might be helpful.
[^2]:
[Here's](https://scribe.rip/168a5bafaa14) an interesting blog post which goes into detail about
this.
[^3]:
In the age of smartphones, which are built on the principles of planned obsolescence, and
[IoT](https://twitter.com/internetofshit), it isn't unusual that people use devices that are
horrendously outdated.
[^4]:
[This](https://www.youtube.com/watch?v=0opakLwtPWk) video might convince you to do so.
[This](https://blog.cloudflare.com/rfc-8446-aka-tls-1-3/) is the corresponding article.
[^5]:
[This](https://blog.cloudflare.com/tls-session-resumption-full-speed-and-secure/) blog post
highlights how CloudFlare use a storage cluster to provide access to session IDs across multiple
servers.
[^6]:
[This](https://blog.twitter.com/engineering/en_us/a/2013/forward-secrecy-at-twitter) blog post shows
how Twitter dealt with this issue.
[^7]:
Not sure how that succeeded assuming the TLS connection was valid. Although the certificate for
`google.com` seems to be valid for `youtube.com` as well but that of `www.google.com` isn't.
