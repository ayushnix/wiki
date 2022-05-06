---
title: "Certificate Transparency"
summary: "Certificate Transparency became a thing after DigiNotar and Revocation is broken"
date: 2021-12-25
---

# Certificate Transparency

What would you do if a root CA issues a certificate for your domain without your permission? They
certainly have the power to do so. CAs like DigiNotar, WoSign, CNNIC, and Symantec have done this
before, intentionally and unintentionally. Yes, unknown to most Internet users, the mighty root CAs
[mess](https://wiki.mozilla.org/CA/Incident_Dashboard) [up](https://crt.sh/?cablint=issues) all the
time[^1].

After the DigiNotar
[incident](https://slate.com/technology/2016/12/how-the-2011-hack-of-diginotar-changed-the-internets-infrastructure.html)
, people apparently realized that the state of affairs of PKI on the Internet isn't as trustworthy
as it should be and this spurred the introduction, and eventual enforcement, of [Certificate
Transparency](https://certificate.transparency.dev/) for all publicly issued certificates. This has
been made formal in [RFC 6962](https://datatracker.ietf.org/doc/html/rfc6962). However, a new
version of CT, [RFC 9162](https://datatracker.ietf.org/doc/html/rfc9162), has been released in
December 2021, which makes 6962 obsolete. For now, we'll focus on 6962.

## How Does CT Work?

Before a CA can issue a certificate for your domain, it will generate a **pre-certificate**, a
certificate that is meant to be submitted to append-only, publicly auditable, and cryptographically
verifiable ledgers, called as *Certificate Transparency Logs*. The logs append the pre-certificate
into their records, send a signed certificate timestamp to the CA for inclusion in the final
certificate that the domain owner will use, and expect that certificate to be appended to the log as
well within 24 hours. This time period is called the *maximum merge delay*.

You may have noticed a section called `CT Precertificate SCTs` in the output of `openssl x509`.

```
$ openssl s_client -connect github.com:443 < /dev/null \
  | openssl x509 -text -noout -fingerprint -sha256
...
...
CT Precertificate SCTs:
    Signed Certificate Timestamp:
        Version   : v1 (0x0)
        Log ID    : 29:79:BE:F0:9E:39:39:21:F0:56:73:9F:63:A5:77:E5:
                    BE:57:7D:9C:60:0A:F8:F9:4D:5D:26:5C:25:5D:C7:84
        Timestamp : Mar 25 18:57:33.978 2021 GMT
        Extensions: none
        Signature : ecdsa-with-SHA256
                    30:45:02:21:00:9E:E6:88:44:7F:FC:34:45:9C:32:4D:
                    9F:AB:94:86:06:AE:DD:63:2D:E2:F5:5F:63:97:46:8A:
                    0B:A5:39:D8:D7:02:20:48:54:27:D1:C6:32:B5:BF:81:
                    77:D7:EB:15:68:AC:F2:C8:EE:C9:01:AD:1F:CC:34:0C:
                    EE:C9:10:72:44:98:59
    Signed Certificate Timestamp:
        Version   : v1 (0x0)
        Log ID    : 22:45:45:07:59:55:24:56:96:3F:A1:2F:F1:F7:6D:86:
                    E0:23:26:63:AD:C0:4B:7F:5D:C6:83:5C:6E:E2:0F:02
        Timestamp : Mar 25 18:57:34.009 2021 GMT
        Extensions: none
        Signature : ecdsa-with-SHA256
                    30:46:02:21:00:98:00:12:4A:09:41:18:AF:06:5C:28:
                    EF:1E:BB:DE:85:6C:7F:58:A9:D3:DE:96:B2:16:6A:99:
                    10:AE:2F:F2:69:02:21:00:DD:C5:F8:AD:BD:F0:68:B0:
                    CB:AB:80:B8:F0:D4:A8:52:67:30:E7:A3:F0:3B:F9:B6:
                    BB:09:D0:A6:B6:FE:CA:1D
...
...
SHA256 Fingerprint=0A:E3:84:BF:D4:DD:E9:D1:3E:50:C5:85:7C:05:A4:42:
C9:3F:8E:01:44:5E:E4:B3:45:40:D2:2B:D1:E3:7F:1B
```

Go ahead and use the SHA256 certificate fingerprint on [crt.sh](https://crt.sh) to get certificate
that GitHub is using. There should be a hyperlink on "Precertificate" and you can click on that to
get [this](https://crt.sh/?id=4273320465) pre-certificate created by DigiCert. The only significant
difference between the certificate that you'll receive and the pre-certificate is that the
pre-certificate has a critical X.509v3 extension called `CT Precertificate Poison` which indicates
that it is indeed a pre-certificate and shouldn't be accepted by TLS clients[^2]. This
pre-certificate was sent to Google's Argon 2022 log operation service and Cloudflare's Nimbus 2022
service. This can be verified on the leaf certificate page.

In case you're wondering why we need pre-certificates at all, it's there to resolve a chicken and
egg problem — what comes first? The SCTs that a certificate should have or the certificate itself? A
CA sends a preliminary form of a certificate to a log service which will create SCTs and send it
back to the CA. The CA adds those SCTs to the certificate and, finally, adds his signature to the
certificate and sends it back to us.

It's interesting to note that Google Chrome's [CT
Policy](https://github.com/GoogleChrome/CertificateTransparency/blob/4fa79451e0bf3740204f01d6bd6f6dfe6d2754dd/ct_policy.md)
requires all valid certificates to have a SCT from at least one Google CT log service. It wouldn't
be unreasonable to say that this essentially mandates all publicly issued certificates in the world
to be logged to a Google CT log. [This](https://certificate.transparency.dev/logs/) page lists all
the usable log services in existence. A [JSON](https://www.gstatic.com/ct/log_list/v2/log_list.json)
is also available.

A pre-certificate only seems to be needed when SCTs are delivered using TLS certificates. A SCT can
also be delivered in the TLS handshake using an extension[^3] or using OCSP stapling, something
which we'll talk about later in this article. However, Let's Encrypt embeds SCTs in their
certificates by default and the reason is explained
[here](https://community.letsencrypt.org/t/duplicates-in-ct-logs-precertificate-vs-leaf-certificate/69190/4).

## We Have Logs, So What?

Although these logs do provide us with a method to confirm if any unintentional certificates have
been issued for our domains, there's no way to prevent a root CA from mis-issuing a certificate. The
DNS [CAA](../dns/how-does-dns-work.md#caa) record is also an instruction, not an imposing
requirement which a root CA cannot defy. Hell, a root CA doesn't even need to log anything before
and after creating a certificate. It doesn't need to create a certificate with SCTs either and such
a certificate would be valid.

Fortunately, there are countermeasures in place to respond appropriately if either situation arises.
If a CA doesn't create a log when a certificate is issued, it can't get SCTs and those are needed
from at least two independent log services for web browsers[^4] to accept a TLS connection as valid.

In case a root CA fulfills all the certificate transparency critera (while ignoring the DNS CAA
record) and issues a certificate for a domain that it shouldn't have, there are
[monitors](https://certificate.transparency.dev/monitors/) in place that can be used to send alerts
about certificates that have been issued for your domain. There's also the fact that once a CA is
caught engaging in potentially malicious activities, either intentionally or unintentionally, the
consequences are usually severe and it'll end up being removed from trust stores of all major web
browsers.

## Issues With CT

Even though CT is useful, it isn't without its flaws. There's no real financial incentive to run a
public CT log that consumes millions of certificates being issued by Let's Encrypt or run a CT
monitor service unless you're charging money for it like [SSLMate
is](https://sslmate.com/certspotter/)[^5]. Cloudflare does provide a free monitoring service but
only for domains that are managed using their services. Facebook also provides a monitoring
[service](https://developers.facebook.com/tools/ct/search/) but they want you to have a Facebook
account for it and I, for one, won't create one. If you're an enterprise or a user who knows what
he's doing, it might make sense to [run your own](https://github.com/SSLMate/certspotter) monitoring
service for the specific domains that you own.

The specification for interacting with [these](https://www.gstatic.com/ct/log_list/v2/log_list.json)
logs is mentioned [here](https://www.rfc-editor.org/rfc/rfc6962#section-4).

```
$ curl -s 'https://ct.googleapis.com/logs/argon2022/ct/v1/get-sth' | jq -r .tree_size
448916655
```

This number grows pretty fast and this should give you an idea about just how many TLS certificates
are being issued every second.

```
$ curl -s 'https://ct.googleapis.com/logs/argon2022/ct/v1/get-entries?start=0&end=0' | jq -r
{
  "entries": [
    {
      "leaf_input": "AAAAAAFkp6WtAwAAAATzMIIE7zCCAtegAwIBAgIHBXEu3yVaIDANBgkqhk....
      "extra_data": "AAujAAXMMIIFyDCCA7CgAwIBAgICEAEwDQYJKoZIhvcNAQEFBQAwfTELMA....
    }
  ]
}
```

Unfortunately, it isn't trivial to interact with this binary data (encoded using base64). You'll
need a [client](https://github.com/google/certificate-transparency-go) which can parse this data,
write code to parse this data
[yourself](https://valeurbit.com/blog/googles-certificate-transparency-as-a-data-source-for-attack-prevention/)
or use services like [CertStream](https://certstream.calidog.io/) and
[ct_search_api](https://sslmate.com/ct_search_api/).

Another aspect that you should be aware of is that CT logs are public and anyone can inspect them
and use them to gain information that you may not want to disclose. For example, if you got a
certificate for your router web interface like I did, anyone who goes through CT logs for your
domain will know the FQDN of your router web interface. If you have exposed a web interface on the
Internet and haven't secured your web server, as soon as you get a TLS certificate, your domain will
be made public for anyone to try and exploit the service running behind it. This was highlighted
back in 2015 in [DEFCON 25](https://youtu.be/TMNeSnjZfCI?t=519) and in 2017 in [BSides Las
Vegas](https://youtu.be/7KIk2uA7_Cw?t=2566).
[Here](https://media.defcon.org/DEF%20CON%2025/DEF%20CON%2025%20presentations/DEF%20CON%2025%20-%20Hanno-Boeck-Abusing-Certificate-Transparency-Logs-UPDATED.pdf)
[are](https://raw.githubusercontent.com/hdm/2017-BSidesLV-Modern-Recon/main/Modern%20Internet%20Scale%20Reconnaisance.pdf)
the presentation slides in case YouTube deletes those videos because of DMCA or some other
unfathomable reason.

Yup, this might sound scary but you can

- get a wildcard certificate for your apex domain at the cost of creating a single point of failure
- deploy a private CA
- don't use TLS (which would be ... inadvisable)
- get good and deploy your application in a secure manner

Now that you're aware about this, free and automatically issued TLS certificates from hosting
providers and domain name registrars may not seem like such a good idea. I don't like the idea of
not controlling my private key anyways. I'll try and move this wiki from GitHub Pages to my own VPS.
It's a good excuse to learn what's needed to run a public web server.

# Certificate Revocation

How would you respond if you know that your private key has been compromised[^6]? To make things
worse, what if a root CA [issues a
certificate](https://security.googleblog.com/2015/09/improved-digital-certificate-security.html) for
your domain even though it wasn't supposed to, like DigiNotar did? Just like GPG PKI, TLS PKI
supports certificate revocation. As expected, however, key revocation is a mess and may not work as
expected. Your ACME client should provide an option to revoke a certificate and the process will
mostly be similar to how the certificate was issued. The interesting question is that how is
revocation enforced across billions of client devices?

The somewhat obvious solution to this problem is distributing a list of revoked certificates to web
browsers. This is what **Certificate Revocation Lists** do. However, at the rate with which HTTPS
adoption is happening, this solution isn't really scalable. Not to mention that not everyone is
privileged enough to have access to inexpensive and reliable bandwidth to download a list with an
[ever increasing size](https://isc.sans.edu/crls.html).

Yes, as you might have been able to guess, when clients can't handle data in a scalable manner, make
the servers handle them which is what **Online Certificate Status Protocol** is meant to do. The
client will contact an OCSP responder and ask if a particular certificate is valid. The responder
will send an OCSP response signed by the CA which signed the certificate and the client will then
behave appropriately. This is how a signed OCSP response looks like:

```
$ openssl s_client -connect ayushnix.com:443 -status < /dev/null
...
OCSP response:
======================================
OCSP Response Data:
    OCSP Response Status: successful (0x0)
    Response Type: Basic OCSP Response
    Version: 1 (0x0)
    Responder Id: C = US, O = Let's Encrypt, CN = R3
    Produced At: Dec 30 15:18:00 2021 GMT
    Responses:
    Certificate ID:
      Hash Algorithm: sha1
      Issuer Name Hash: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
      Issuer Key Hash: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
      Serial Number: xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
    Cert Status: good
    This Update: Dec 30 15:00:00 2021 GMT
    Next Update: Jan  6 14:59:58 2022 GMT

    Signature Algorithm: sha256WithRSAEncryption
         xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:
======================================
```

However, just like you lose ownership and privacy when you choose to use Google Stadia over a
digital offline copy, you end up leaking your browsing history to yet another party, the OCSP
responder. Then again, until ECH becomes ubiquitous, your browsing history is already up for grabs,
unless you're using WireGuard or Tor.

There's another problem with both CRL and OCSP. What happens if the OCSP responder is offline? What
if your ISP simply blocks the OCSP server like my ISP [blocks
DuckDuckGo](https://torguard.net/blog/duckduckgo-was-recently-blocked-in-india/)? After all, OCSP
connections aren't encrypted by default. Should your browser fail to load the website you want
(hard-fail) to browse just because an OCSP responder is down even though the website may be up and
running? Should your browser go on about its business as usual and ignore the risk (soft-fail) that
the website it is connecting to might be fake? As you may have guessed, hard-fail isn't sensible and
soft-fail is useless.

Adam Langley used an analogy and [compared](https://www.imperialviolet.org/2012/02/05/crlsets.html)
revocation with a shitty seat belt that snaps during a car crash, right at the moment when you need
that seat belt the most.

Nonetheless, certificate revocation is still in use and this is what we have right now.

## OCSP Stapling

Instead of shifting the burden of hosting the list of revoked certificates to the client or leaking
our browsing history to an OCSP responder, we can ask the web server in question to fetch and serve
valid OCSP responses. Yes, I know, it sounds strange asking a web server for a valid OCSP response
considering we're trying to verify the status of the certificate offered by the web server itself.

The web server in question will contact the OCSP responder and get a short lived OCSP response which
will be signed by the trusted CA which signed the certificate being served by the web server. The
OCSP URL should be embedded inside your certificate:

```
$ openssl s_client -connect google.com:443 < /dev/null 2> /dev/null \
  | openssl x509 -text -noout | rg -i ocsp
                OCSP - URI:http://ocsp.pki.goog/gts1c3
```

The DER encoded and signed OCSP response shown before is included in the *Certificate Status
Request* TLS extension[^7]. The clients will be presented with this signed OCSP response and they
don't have to maintain an updated list of revoked certificates or query an OCSP responder each and
every time they visit a website. Although a web server will need to query the OCSP responder, it
will only need to do so after specific intervals to get fresh responses, not for each and every
query.

The [initial version](https://datatracker.ietf.org/doc/html/rfc6066#section-8) of the status request
extension supported only one OCSP response, the one needed to ascertain the status of the
certificate for the web server. However, we also need to know the revocation status of intermediate
certificates offered by CAs and this is what [RFC
6961](https://datatracker.ietf.org/doc/html/rfc6961) offers by defining the next version of the
certificate status request extension.

Unfortunately, it looks like OCSP status request messages in TLS 1.2 aren't encrypted.

![OCSP Response - Dark Mode](/images/ocsp-response-dark.webp#only-dark)
![OCSP Response - Light Mode](/images/ocsp-response-light.webp#only-light)

Fortunately, OCSP responses using the status request extension are encrypted in TLS 1.3. There, we
now have another reason to use TLS 1.3 in our servers.

### Implementing OCSP Stapling

Mozilla's [SSL Configuration Generator](https://ssl-config.mozilla.org/) is an excellent tool for
this job. In nginx, you'll have to add

```
server {
    ....
    ....

    # OCSP stapling
    ssl_stapling on;
    ssl_stapling_verify on;

    # verify chain of trust of OCSP response using Root CA and Intermediate certs
    ssl_trusted_certificate /path/to/root_CA_cert_plus_intermediates;

    # replace with the IP address of your resolver
    resolver 127.0.0.1;
}
```

### OCSP Must-Staple

OCSP stapling sounds good so far but the client doesn't really know that the server supports OCSP
stapling and may not ask for an OCSP response. Although I'm not sure about TLS 1.3, the certificate
status request extension in TLS 1.2 is unencrypted and may be tampered with by an attacker. This
sounds like the useless soft-fail behavior all over again. Fortunately, since we're using OCSP
stapling, hard-fail doesn't sound as absurd as it did before considering OCSP responses are cached
and are valid for a few days. This is where **OCSP Must Staple** comes into the picture.

If your CA supports OCSP stapling and if you specify

```
[v3_req]
tlsfeature = status_request
```

in the CSR config file before you get a certificate, the certificate generated by the CA should have
an additional section that mentions that the certificate must be accompanied by an OCSP response.
Some ACME clients might use `1.3.6.1.5.5.7.1.24 = DER:30:03:02:01:05` instead, which is older, but
equivalent, method of doing the same thing.

This is how a CSR with the OCSP must staple extension looks like

``` hl_lines="19-20"
root@router:~# openssl req -in testing.ayushnix.com.csr -noout -text
Certificate Request:
    Data:
        Version: 1 (0x0)
        Subject: CN = testing.ayushnix.com
        Subject Public Key Info:
            Public Key Algorithm: id-ecPublicKey
                Public-Key: (256 bit)
                pub:
                    xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:
                ASN1 OID: prime256v1
                NIST CURVE: P-256
        Attributes:
        Requested Extensions:
            X509v3 Subject Alternative Name:
                DNS:testing.ayushnix.com
            X509v3 Basic Constraints:
                CA:FALSE
            TLS Feature:
                status_request
    Signature Algorithm: ecdsa-with-SHA256
         xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:
```

and this is how the certificate will look like

``` hl_lines="18-19"
...
...
           CT Precertificate SCTs:
                Signed Certificate Timestamp:
                    Version   : v1 (0x0)
                    Log ID    : xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:
                    Timestamp : Nov  7 00:07:05.239 2021 GMT
                    Extensions: none
                    Signature : ecdsa-with-SHA256
                                xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:
                Signed Certificate Timestamp:
                    Version   : v1 (0x0)
                    Log ID    : xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:
                    Timestamp : Nov  7 00:07:05.226 2021 GMT
                    Extensions: none
                    Signature : ecdsa-with-SHA256
                                xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:
            TLS Feature:
                status_request
    Signature Algorithm: sha256WithRSAEncryption
         xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:xx:
```

Before you go ahead and get a certificate with this extension, you have to make sure that your web
server supports, and has enabled, OCSP stapling. As you may expect, if a client does not receive a
valid OCSP response from the server, the website in question will be inaccessible.

Even though OCSP responses are valid for several days, this is still hard-fail and can be
devastating if there's an
[issue](https://community.letsencrypt.org/t/may-19-2017-ocsp-and-issuance-outage-postmortem/34922/1)
with the OCSP responder. What if the cached OCSP response is set to expire at a time when the OCSP
responder is down? Scott Helme seems to have proposed an HTTP header called
[expect-staple](https://scotthelme.co.uk/designing-a-new-security-header-expect-staple/) which
would've been used to monitor OCSP responses and send reports to a specified URL but I don't think
that ever became an [official
specification](https://twitter.com/estark37/status/949336829349707776). We're essentially back to
square one with revocation still being broken and useless.

At this moment, Google Chrome and Mozilla Firefox use
[CRLSets](https://www.imperialviolet.org/2012/02/05/crlsets.html) and
[OneCRL](https://blog.mozilla.org/security/2015/03/03/revoking-intermediate-certificates-introducing-onecrl/)
respectively. However, they only include a [tiny](https://crt.sh/mozilla-onecrl) fraction of the
revoked certificates published by CAs because of reasons explained before. In short, they include
revoked certificates to prevent catastrophic issues like the DigiNotar fiasco.

Mozilla has implemented a new solution for certificate revocation called
[CRLite](https://blog.mozilla.org/security/2020/01/09/crlite-part-1-all-web-pki-revocations-compressed/)
which might end up becoming the de-facto standard for TLS certificate revocation. However, it
doesn't seem to be the default method for certificate revocation checking in Firefox yet.
[Here's](https://wiki.mozilla.org/CA/Revocation_Checking_in_Firefox) how Firefox is doing
certificate revocation. [Here's](https://www.youtube.com/watch?v=Ult8JPc3rPY) a nice video
explaining what CRLite is.

--8<-- "include/abbreviations.md"

[^1]:
[This](https://robertheaton.com/2018/11/28/https-in-the-real-world/) blog post presents a good
overview of the issues surrounding TLS.
[^2]:
The format of pre-certificates has
[changed](https://datatracker.ietf.org/doc/html/rfc9162#section-1.3) in CT 2.0 from a X.509
certificate to a PKCS#7 CMS object, just like a private key or signed data.
[^3]:
The `signed_certificate_timestamp` extension used before has been replaced by the
`transparency_info` TLS extension.
[^4]:
For reasons I can't fathom, Firefox [doesn't](https://bugzilla.mozilla.org/show_bug.cgi?id=1281469)
seem to support CT. Oh, and if I'm not mistaken, web browsers that check for valid SCTs are also
called auditors.
[^5]:
[CertSpotter](https://github.com/SSLMate/certspotter/) is available as an open source project
though, so you can run it locally and possibly set it up to monitor your domains.
[^6]:
It's better to know that your private key has been compromised than not knowing.
[^7]:
I have no idea why everyone uses the term OCSP "stapling". Scott Helme even goes as far as to say
that the OCSP response is "stapled" to the certificate and approved by the CA. This mislead me into
believing that the OCSP response is somehow added to the certificate itself like CT signed
certificate timestamps are but that's not possible because a CA has to prepare a precertificate to
get around this issue of when to embed its signature in the final certificate and it can't just add
more data to the certificate after it has already signed the certificate. OCSP responses are
delivered using a TLS extension, they're NOT "stapled" to the certificate.
