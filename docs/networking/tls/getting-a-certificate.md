---
title: "Getting A Certificate"
summary: "Before I get a certificate for my domain, I want to know how TLS actually works."
date: 2021-11-18
---

# Introduction

The intent to learn how TLS works began when I realized that I could get a wildcard certificate for
my locally hosted services. I realized I didn't understand how TLS works so I started with
[SSH](../../sysadmin/ssh/ssh-certificates.md) and [GPG](../../sysadmin/gpg/gpg-101.md) before I got
here. If you're interested, you might wanna check out those articles before reading this.

This article will focus on TLS from a system administrator's point of view, not that of a
cryptographer or a mathematician. However, I'll focus more on understanding how things work rather
than focusing on operational aspects. This will involve a traditional bottom up approach.

As a beginner, there seem to be two primary aspects when it comes to understanding how TLS works:

- getting a certificate for your own website
- understanding how TLS works when you visit a website

Before we begin, there are a few things I'd like to mention.

## Issues with OpenSSL and TLS

`The openssl command is incredibly complex`

:  I'm kinda surprised by how much flak `gnupg` receives but `openssl` seems to be relatively
unscathed. Sure, some distributions switched to LibreSSL after
[HeartBleed](https://en.wikipedia.org/wiki/Heartbleed) but almost all of them [switched
back](https://voidlinux.org/news/2021/02/OpenSSL.html) to OpenSSL because maintaining patches for
applications that didn't support LibreSSL turned out to be cumbersome.

`The existing documentation on the Internet is deprecated and obsolete`

:  I couldn't easily find any quality blog posts or articles about how to use `openssl`[^1]. Most of
them are still using deprecated commands like `genrsa` and don't encrypt private keys. It doesn't
really help that `openssl` has support for a wide range of algorithms and protocols that belong in a
muesuem. There's also a fair bit of redundancy with multiple methods of doing the same thing.

`TLS didn't support ED25519 keys at the end of 2021`

:  One would think that the Snowden disclosures back in 2013 would've made an impact. To be fair, it
definitely has but I was disappointed to learn that I can't use ED25519 key pairs when using TLS. I
can either use RSA or ECDSA with NIST curves. Apparently, an organization called the [CA/Browser
Forum](https://cabforum.org/) is responsible for deciding what keys sizes and algorithms certificate
authorities should recognize. This is defined in their [Baseline Requirements
Documents](https://cabforum.org/baseline-requirements-documents/). The latest version available at
the time this article was written was
[1.8.0](https://cabforum.org/wp-content/uploads/CA-Browser-Forum-BR-1.8.0.pdf). You want to look at
Section 6.1.5.

Anyways, let's not resign ourselves to cynicism and start with getting our own certificate. I
recommend using either [OpenSSL](https://www.openssl.org/) or
[LibreSSL](https://www.libressl.org/)[^2]. I'm going to use OpenSSL in this article but it would be
better to use LibreSSL on an OpenBSD machine, either physical or virtual.

# Getting a Certificate

As we've seen while using `ssh` and `gpg`, asymmetric cryptography begins with a key pair. If you
went through my article on [gpg](../../sysadmin/gpg/gpg-101.md), you may recall how I criticized the
user experience while generating an ECC key pair. Well, `openssl` is worse. I'll try to keep it
simple.

We can use the `genpkey` sub-command to generate an elliptic curve parameter file and an encrypted
private key file. The generation of a parameter file seems to be unique to ECDSA because neither RSA
nor ED25519 needs it[^3]. The `genpkey` sub-command also lets us encrypt the private key using
`aes256`[^4].

```
$ openssl genpkey -genparam -algorithm EC -out ayushnix-param.pem -pkeyopt ec_paramgen_curve:P-256
$ openssl genpkey -paramfile ayushnix-param.pem -aes256 -out ayushnix-priv.key
$ ll
.rw-r-----  75 user user 22 Nov 23:29 ayushnix-param.pem
.rw------- 399 user user 22 Nov 23:50 ayushnix-priv.key
$ cat ayushnix-param.pem
-----BEGIN EC PARAMETERS-----
BggqhkjOPQMBBw==
-----END EC PARAMETERS-----
$ cat ayushnix-priv.key
-----BEGIN ENCRYPTED PRIVATE KEY-----
OqoJIF+zGqjBZcxfd7A6BquirndqMr3NNYPceYuQT1RFJPPVLu/f11zsEu4yRIsE
8rgba1T7ZK8NO/vQYsySU8ugoeXQSrARjzpwSkooCT786/LcGlFzlRXuL3rQ+YRN
PJjswrfKX4mNgL7Nf6UHQxkTUJH2jlt7eH2zCz6WSfYxyL0etR7jangLtREHdXPx
Et8bNCDmJTHdovXm+SOuXeTUU2QMnHxNKFA2ZSVAwTE6Q58h9aoXDJXQugOyrrd0
8/Jshjxmp/hfWTDumiA/RJ7WQm5IfaMzkqmPeJJTZ2Ghyc8yuzCmQdkvAWySuWny
-----END ENCRYPTED PRIVATE KEY-----
```

What about the public key?

```
$ openssl pkey -in ayushnix-priv.key -text_pub -noout
Public-Key: (256 bit)
pub:
    d2:c8:3d:ae:76:49:f1:01:68:0b:14:47:f1:9a:3d
    ad:5d:da:42:c9:db:7f:2a:2c:2a:80:18:d8:33:86
    40:99:d7:af:80:c6:4b:1a:e0:64:71:fa:c1:b9:f2
    96:ba:09:65:ce:53:6b:a2:b1:33:d9:f1:7b:e5:25
    cf:41:9f:ce:0c
ASN1 OID: prime256v1
NIST CURVE: P-256
```

Yes, `openssl genpkey` doesn't generate a public key file.

I've chosen to use the curve P-256 developed by NIST. You can go ahead and list the number of curves
that openssl supports using `openssl ecparam -list_curves` but it probably won't be a pleasant
experience. You can use

```
$ openssl ecparam -list_curves | rg -i '256|384'
```

to get the list of curves you probably need to care about when generating ECDSA key pairs. I'm not
sure if there's a difference between `secp256k1` and `prime256v1` but
[this](https://tools.ietf.org/search/rfc4492#appendix-A) indicates that they're "equivalent". You
can let openssl decide which curve to use by specifying `P-256`. I couldn't find any compelling
reason to use `P-384`. Go ahead and use `P-384` if you want.

There's a few things that we should know before we try to get a certificate.

## PEM, ASN, OID ... what?

**Abstract Syntax Notation One (ASN.1)** is a method for defining data structures in a language
independent manner. It seems to be used for defining specifications for how data should be stored
for usage in protocols like SS7, 4G, 5G. You can also think of ASN.1 data as a complex tree with
lots of nodes. A specific node in this complex tree can be identified using a specific path carved
all the way from the root node and this path is the **object identifier (OID)**[^5].

A preview of ASN.1 data and OIDs should make things easier. Let's parse the private key that we
generated earlier.

```
$ openssl asn1parse -in ayushnix-priv.key -i
    0:d=0  hl=3 l= 236 cons: SEQUENCE
    3:d=1  hl=2 l=  87 cons:  SEQUENCE
    5:d=2  hl=2 l=   9 prim:   OBJECT            :PBES2
   16:d=2  hl=2 l=  74 cons:   SEQUENCE
   18:d=3  hl=2 l=  41 cons:    SEQUENCE
   20:d=4  hl=2 l=   9 prim:     OBJECT            :PBKDF2
   31:d=4  hl=2 l=  28 cons:     SEQUENCE
   33:d=5  hl=2 l=   8 prim:      OCTET STRING      [HEX DUMP]:6861DBBCAF87B6A8
   43:d=5  hl=2 l=   2 prim:      INTEGER           :0800
   47:d=5  hl=2 l=  12 cons:      SEQUENCE
   49:d=6  hl=2 l=   8 prim:       OBJECT            :hmacWithSHA256
   59:d=6  hl=2 l=   0 prim:       NULL
   61:d=3  hl=2 l=  29 cons:    SEQUENCE
   63:d=4  hl=2 l=   9 prim:     OBJECT            :aes-256-cbc
   74:d=4  hl=2 l=  16 prim:     OCTET STRING      [HEX DUMP]:<hex data>
   92:d=1  hl=3 l= 144 prim:  OCTET STRING      [HEX DUMP]:<hex data>
```

All of the "nodes" in our tree which are named `OBJECT` have values which are human-readable forms
of OIDs. For example, `hmacWithSHA256` has an OID of
[1.2.840.113549.2.9](http://oid-info.com/get/1.2.840.113549.2.9).

All of this data is encoded using specific encoding rules. One of them is **Distinguished Encoding
Rules (DER)** which encodes this data in binary format. We'll prefer encoding data in **Privacy
Enhanced Mail (PEM)** format, which is just data encoded in base64.

## PKCS

The private key that we parsed earlier was actually stored in a syntax of its own known. RSA
Security LLC came up with **Public Key Cryptography Standards** to define how cryptographic data
like private keys will be stored. The EC private key that we generated was written according to
[PKCS #8](https://en.wikipedia.org/wiki/PKCS_8). If we had generated a private key using RSA, it
would've used PKCS #1 instead.

[Here's](https://en.wikipedia.org/wiki/PKCS) a complete list of PKCS formats which are widely used.

Although OpenSSH uses [its own format](https://datatracker.ietf.org/doc/html/rfc4716) for storing
keys, we can tell `ssh-keygen` to output keys in PEM and PKCS #8 format as well.

To avoid confusion at this point, we could think of ASN.1 as the syntax of a programming language
and standards like PKCS and PEM as programs written using that syntax.

## Certificate Signing Request

When we use OpenSSH, a certificate authority signs our public key to generate a certificate. The
certificate authority also decides the key ID, principals, and extensions that should be granted to
the certificate but this information must also somehow come from the source of the public key in an
authenticated manner. This authentication is often provided by methods like OAuth and tools like
HashiCorp Vault.

Similarly, when using TLS, a certificate authority needs to know certain details, such as your
domain name, before it can grant a certificate to you. These details are provided by the client
using a **Certificate Signing Request (CSR)**, a signed message using the private key. This signed
message is stored in the PKCS #10 format. If you're looking for a similar operation in SSH, the [Key
Authentication](../../sysadmin/ssh/ssh-certificates.md#key-authentication) section should help.

Although we can use the OpenSSL command line to mention all the required details, the command would
look like an abomination[^6]. We'll use a configuration file to mention these details. You can find
a default configuration file called `openssl.cnf` using

```
$ openssl version -d
OPENSSLDIR: "/etc/ssl"
```

You'll find several sections in the default configuration file. Right now, we're interested in the
`[ req ]` section. Here's a simple file to get started:

```
# create an ECDSA P-256 CSR for router.ayushnix.com

[ req ]
prompt                        = no
input_password                = the-password-used-to-encrypt-the-private-key
distinguished_name            = req_distinguished_name
req_extensions                = v3_req

[ req_distinguished_name ]

[ v3_req ]
subjectAltName                = @alt_names

[ alt_names ]
DNS.1 = router.ayushnix.com
```

### Certificate Validation Levels and X.509

Yes, we have yet another standard for CSRs and signed certificates known as **X.509**. It seems to
come from a series of standards known as X.500 which seem to function similarly to DNS in that they
maintain key-value pairs. The keys are called *Distinguished Names*.

- `Common Name (CN)`
- `Organization (O)`
- `Organizational Unit (OU)`
- `Locality (L)`
- `State (S)`
- `Country (C)`

If you want a certificate for a personal website or a blog, you want a **domain validation (DV)**
certificate. You don't need to mention any distinguished names in your CSR if you want a DV
certificate. For reference, Let's Encrypt offers only DV certificates.

If you want a certificate for a website of an organization, you may want an **organization
validation (OV)** certificate. This is where you do need to mention these distinguished names. This
might involve some background check by the CA but an OV is functionally equivalent to a DV. If your
organization has a lot of dumb money and they don't understand technology, they might wanna get an
**extended validation (EV)** certificate.

In case you need to get an OV, you should be familiar with the quirks of `C` and `CN`. The value of
`C` should be a two letter [ISO 3166](https://en.wikipedia.org/wiki/List_of_ISO_3166_country_codes)
code of the country you're in.  `CN`, which was meant to include a canonical domain name but could
also have a human friendly string instead, has been deprecated for more than [two
decades](https://datatracker.ietf.org/doc/html/rfc2818#section-3.1). The Google Chrome web browser
simply [ignores
CN](https://developers.google.com/web/updates/2017/03/chrome-58-deprecations#remove_support_for_commonname_matching_in_certificates)
since version 58 came out back in April 2017. If your application or CA doesn't work without a `CN`
in a CSR, it belongs in the nearest trash bin.

Instead of using a `CN`, you should use **Subject Alternative Names (SAN)**, which is what we've
done by specifying the `req_extensions` field in the sample configuration file mentioned above. You
can write the list of domain names you want a certificate for, including wildcard domains such as
`*.router.ayushnix.com`, in the `alt_names` section.

At this point, we can make two unique choices:

- create a self-signed certificate
- create a CSR and submit it to a certificate authority, such as Let's Encrypt, to get a DV

You could also create your own personal CA but I'm not going to explore that in this article.

### Self-Signed Certificate

A self-signed certificate is something where you issue a certificate to yourself using your own
private key. Although this certificate can be used to secure HTTP communications, it won't be
recognized by a web browser unless it has been manually made to trust a certificate authority
generated using your private key.

As you might have imagined, nothing stops you from generating a self-signed certificate for any
domain you like, even `google.com`. Of course, your browser won't trust it and will warn you if you
try to use it. This is what man-in-the-middle attack looks like. Some organizations deploy their own
private CAs and self-signed certificates and MITM all of their machines for controlling traffic on
them[^7].

You can create self-signed certificates using `openssl req -x509 ...` or use a tool like
[mkcert](https://github.com/FiloSottile/mkcert) to makes things easier. For now, I'm going to
replace the self signed certificate on the uHTTPd web server in my router. This is how a self signed
certificate generated by OpenWRT looks like

```
$ openssl s_client -brief -connect router.ayushnix.com:443
depth=0 C = ZZ, ST = Somewhere, L = Unknown, O = OpenWrtda170f87, CN = OpenWrt
verify error:num=18:self signed certificate
CONNECTION ESTABLISHED
Protocol version: TLSv1.3
Ciphersuite: TLS_CHACHA20_POLY1305_SHA256
Peer certificate: C = ZZ, ST = Somewhere, L = Unknown, O = OpenWrtda170f87, CN = OpenWrt
Hash used: SHA256
Signature type: ECDSA
Verification error: self signed certificate
Server Temp Key: ECDH, P-256, 256 bits
```

To generate a self-signed certificate using the sample configuration file mentioned above

```
$ openssl req -x509 -config router.ayushnix.com.cnf -key ayushnix-priv.key \
    -subj / -out self-signed-router.ayushnix.com.crt
```

The `-subj /` portion is needed to tell OpenWRT to not ask for any distinguished names and from what
I tested, you can't just skip writing the `distinguished_name` field in the configuration file
either. If you do, OpenSSL will complain that it's still stuck in the 1990s.

This is how the self signed certificate looks like

```
$ openssl x509 -in self-signed-router.ayushnix.com.crt -noout -text
Certificate:
    Data:
        Version: 1 (0x0)
        Serial Number:
	    d3:60:05:f8:c4:e6:ea:d0:fd:42:84:f9:88:2e:ab:6e:1c:29:d5:12
        Signature Algorithm: ecdsa-with-SHA256
        Issuer:
        Validity
            Not Before: Nov 25 16:36:48 2021 GMT
            Not After : Dec  2 16:36:48 2021 GMT
        Subject:
        Subject Public Key Info:
            Public Key Algorithm: id-ecPublicKey
                Public-Key: (256 bit)
                pub:
                    80:3e:75:f7:7f:73:91:a8:67:24:15:40:1e:c0:cb:
                    eb:ce:bd:17:a7:17:b3:5d:06:23:99:00:6b:af:e9:
                    5e:cd:0c:a9:46:c8:9a:e7:84:c1:03:f8:44:94:6c:
                    05:33:e9:a4:78:f3:94:73:2a:58:12:16:de:fd:17:
                    3a:ff:8f:41:8d
                ASN1 OID: prime256v1
                NIST CURVE: P-256
    Signature Algorithm: ecdsa-with-SHA256
         30:45:02:20:62:a5:14:95:5e:07:0e:ec:54:d2:3b:2d:e9:60:
         92:cc:b5:4f:51:49:e3:96:35:3b:8b:97:1e:61:47:12:db:ae:
         02:21:00:ab:47:70:58:f9:21:36:63:b0:9f:06:5d:81:c1:3f:
         37:ab:96:38:33:fd:d2:96:9d:ce:49:70:a6:94:c6:d5:ed:4f:
$ openssl s_client -brief -connect router.ayushnix.com:443
depth=0
verify error:num=18:self signed certificate
CONNECTION ESTABLISHED
Protocol version: TLSv1.3
Ciphersuite: TLS_CHACHA20_POLY1305_SHA256
Peer certificate:
Hash used: SHA256
Signature type: ECDSA
Verification error: self signed certificate
Server Temp Key: ECDH, P-256, 256 bits
```

The absensce of distinguished names should be obvious but more importantly, the certificate works on
all modern web browsers. You'll still get a warning about the certificate being untrusted and you
can go ahead and make certificates signed using your private key trusted by your system if you're
just testing things out. I won't go into details about this in this article because I'd rather get a
trusted certificate.

As mentioned earlier in the 4th footnote[^4], we could've used `openssl req` to generate the private
key while generating the CSR.

```
$ openssl req -newkey param:ayushnix-param.pem -keyout ayushnix-req-priv.key \
    -config router.ayushnix.com.cnf -subj / -out router.ayushnix.com.csr
```

This is how the generated CSR file looks like

```
$ openssl req -in router.ayushnix.com.csr -noout -text
Certificate Request:
    Data:
        Version: 1 (0x0)
        Subject:
        Subject Public Key Info:
            Public Key Algorithm: id-ecPublicKey
                Public-Key: (256 bit)
                pub:
                    2c:ef:fb:b8:1a:5b:5d:3b:a8:41:83:21:d7:8e:5f:
                    38:b9:93:b9:5e:b8:e1:9a:b8:ba:3b:48:9a:1e:06:
                    0d:67:2e:f1:26:df:fc:3f:17:8d:34:f6:f8:6f:51:
                    7f:c2:9b:6d:f2:f8:58:48:45:13:b5:c5:09:7e:ee:
                    5d:9c:fa:b0:73
                ASN1 OID: prime256v1
                NIST CURVE: P-256
        Attributes:
        Requested Extensions:
            X509v3 Subject Alternative Name:
                DNS:router.ayushnix.com
    Signature Algorithm: ecdsa-with-SHA256
         30:46:02:21:00:f5:d4:ab:44:0f:cd:f6:3e:f4:41:d2:b2:d4:
         d1:16:5b:df:b8:69:33:11:1f:22:5e:cb:ca:8f:b1:60:63:f1:
         2c:d7:e9:43:8f:a2:5f:13:8d:56:bd:74:50:75:71:b4:2a:1f:
         c4:08:59:d4:fb:c4:34:84:da:57:45:7f:78:04:e3:df:cd:99
```

However, this is how the generated private key looks like:

``` hl_lines="15"
$ openssl asn1parse -in ayushnix-req-priv.key -i
 0:d=0  hl=3 l= 227 cons: SEQUENCE
 3:d=1  hl=2 l=  78 cons:  SEQUENCE
 5:d=2  hl=2 l=   9 prim:   OBJECT            :PBES2
16:d=2  hl=2 l=  65 cons:   SEQUENCE
18:d=3  hl=2 l=  41 cons:    SEQUENCE
20:d=4  hl=2 l=   9 prim:     OBJECT            :PBKDF2
31:d=4  hl=2 l=  28 cons:     SEQUENCE
33:d=5  hl=2 l=   8 prim:      OCTET STRING      [HEX DUMP]:E0845A12305F93A5
43:d=5  hl=2 l=   2 prim:      INTEGER           :0800
47:d=5  hl=2 l=  12 cons:      SEQUENCE
49:d=6  hl=2 l=   8 prim:       OBJECT            :hmacWithSHA256
59:d=6  hl=2 l=   0 prim:       NULL
61:d=3  hl=2 l=  20 cons:    SEQUENCE
63:d=4  hl=2 l=   8 prim:     OBJECT            :des-ede3-cbc
73:d=4  hl=2 l=   8 prim:     OCTET STRING      [HEX DUMP]:77574371A6421F9A
83:d=1  hl=3 l= 144 prim:  OCTET STRING      [HEX DUMP]:<hex dump>
```

I'll let you decide which method to use.

## ACME

The ACME protocol allows a client to send JSON messages over HTTPS to an ACME server for automating
the certificate creation process.

Before ACME was a thing, the typical user experience for getting a DV certificate would involve

- sending a CSR file to a CA
- proving control over the domain(s) mentioned in the CSR
- downloading the issued certificate and placing it on a web server

We can thank the people behind [RFC 8555](https://datatracker.ietf.org/doc/html/rfc8555) for making
ACME a standard and automating this tedious process.

To begin using ACME, we'll need to pick an [ACME
client](https://letsencrypt.org/docs/client-options/). I'll use
[acme.sh](https://github.com/acmesh-official/acme.sh)[^8] because it's present in the package
repositories of OpenWRT and seems to have support for using the DNS API of my authoritative
nameserver provider.

### ACME Challenge

There are three types of challenges you can complete to prove that you control the domains that
you've mentioned in your CSR. Obviously, a recognized certificate authority won't just hand over
certificates for any domain you mention in your CSR.

The **HTTP-01** challenge asks you to place a file given by the CA in a location on your web server.
This location is `http://<YOUR_DOMAIN>/.well-known/acme-challenge/<TOKEN>`. This makes sense for
anyone who's hosting a website on a web server. This probably doesn't make sense if your web server
isn't exposed on the Internet on port 80 or if you need wildcard domain certificates.

The **DNS-01** challenge is what we want to use in our case. It asks you to place a TXT record with
the label `_acme-challenge.<YOUR_DOMAIN>`. This works well for wildcard domain certificates and when
your web server isn't exposed on the Internet. However, you do take on the risk of keeping your DNS
API credentials on your server although this might be mitigated to an extent by [using an alias
domain name](https://dan.langille.org/2019/02/01/acme-domain-alias-mode/) made specifically for
DNS-01.

The **TLS-ALPN-01** challenge makes sense if, for some reason, port 80 isn't available on a web
server but 443 is. It uses ALPN to check if your server can present a self-signed certificate with a
specific ALPN extension value along with the SAN and the `acmeIdentifier` extension. This challenge
is suitable if you're running a reverse proxy which can validate and implement TLS on behalf of
several servers.

As I said, I'll need to use the DNS-01 challenge to get a certificate for `router.ayushnix.com`
because the nginx[^9] web server running inside my OpenWRT router isn't publicly available at either
port 80 or 443.

### Staging Environment

Let's Encrypt provides a [staging environment](https://letsencrypt.org/docs/staging-environment/)
where you can mess around and not worry about being rate limited, to a certain extent, if you don't
do something right. Here's what I did[^10] to get a test certificate from the staging environment of
Let's Encrypt:

```
root@router:~# acme.sh --staging --create-account-key -ak ec-256
root@router:~# acme.sh --staging --register-account
root@router:~# acme.sh --staging --update-account -m me@email.com
```

This is a one-time activity. You create an account and register yourself on the Let's Encrypt
servers using a private key called the *account key*.

acme.sh allows us to create a private key for our domain, which it calls the *domain key*, although
it doesn't encrypt it. You can also create a CSR based on that private key.

```
root@router:~# acme.sh --staging --create-domain-key -d router.ayushnix.com \
                 -k ec-256
root@router:~# acme.sh --staging --create-csr --ecc -d router.ayushnix.com

root@router:~# tree /etc/acme/
/etc/acme/
├── ca
│   └── acme-staging-v02.api.letsencrypt.org
│       └── directory
│           ├── account.json
│           ├── account.key
│           ├── account_thumbprint.txt
│           └── ca.conf
├── http.header
└── router.ayushnix.com_ecc
    ├── router.ayushnix.com.conf
    ├── router.ayushnix.com.csr
    ├── router.ayushnix.com.csr.conf
    └── router.ayushnix.com.key
```

Since we're going to use the DNS-01 challenge, we're going to need credentials to modify TXT records
in our DNS. [This](https://github.com/acmesh-official/acme.sh/wiki/dnsapi) should be helpful.

```
root@router:~# export PROVIDER_KEYS=""
root@router:~# acme.sh --staging --issue --dns dns_provider \
                 -d router.ayushnix.com -k ec-256
```

The creation of the private key and the CSR in the beginning was unnecessary because the last
command does that. It should generate three things:

- a certificate for your domain
- an intermediate certificate
- a full chain certificate

We'll not go into details about the intermediate and the full chain certificate in this article. It
belongs in the 2nd aspect of TLS that we talked about in the beginning and it will get its own
article.

This is how a certificate from the staging environment of Let's Encrypt looks like:

```
$ openssl s_client -connect router.ayushnix.com:443 < /dev/null 2> /dev/null \
    | openssl x509 -noout -text
Certificate:
    Data:
        Version: 3 (0x2)
        Serial Number:
            ....................
        Signature Algorithm: ecdsa-with-SHA384
        Issuer: C = US, O = (STAGING) Let's Encrypt, CN = (STAGING) Ersatz Edamame E1
        Validity
            Not Before: Dec  8 15:28:16 2021 GMT
            Not After : Mar  8 15:28:15 2022 GMT
        Subject: CN = router.ayushnix.com
        Subject Public Key Info:
            Public Key Algorithm: id-ecPublicKey
                Public-Key: (256 bit)
                pub:
                    ....................
                ASN1 OID: prime256v1
                NIST CURVE: P-256
        X509v3 extensions:
            X509v3 Key Usage: critical
                Digital Signature
            X509v3 Extended Key Usage:
                TLS Web Server Authentication, TLS Web Client Authentication
            X509v3 Basic Constraints: critical
                CA:FALSE
            X509v3 Subject Key Identifier:
                ...........................
            X509v3 Authority Key Identifier:
                keyid:....................

            Authority Information Access:
                OCSP - URI:http://stg-e1.o.lencr.org
                CA Issuers - URI:http://stg-e1.i.lencr.org/

            X509v3 Subject Alternative Name:
                DNS:router.ayushnix.com
            X509v3 Certificate Policies:
                Policy: 2.23.140.1.2.1
                Policy: 1.3.6.1.4.1.44947.1.1.1
                  CPS: http://cps.letsencrypt.org

            CT Precertificate SCTs:
                Signed Certificate Timestamp:
                    Version   : v1 (0x0)
                    Log ID    : .................
                    Timestamp : Dec  8 16:28:16.535 2021 GMT
                    Extensions: none
                    Signature : ecdsa-with-SHA256
                                .................
                Signed Certificate Timestamp:
                    Version   : v1 (0x0)
                    Log ID    : .................
                    Timestamp : Dec  8 16:28:16.557 2021 GMT
                    Extensions: none
                    Signature : ecdsa-with-SHA256
                                .................
    Signature Algorithm: ecdsa-with-SHA384
         .........
```

Of course, certificates issued by the staging environment aren't trusted and we'll still get errors
in our web browser but this is a good start.

We can also renew certificates

```
root@router:~# acme.sh --staging -r -d router.ayushnix.com --ecc --always-force-new-domain-key
```

Let's go ahead and get a real certificate (finally!). Yes, I moved the `/etc/acme` directory to
`/etc/acme_staging` to make way for the trusted certificate.

```
root@router:~# acme.sh --issue --dns dns_provider -d router.ayushnix.com -k ec-256
```

[Here's](https://crt.sh/?id=5761815225) the certificate that was issued[^11].

Here's an image that shows what we learned in this article.

![Getting a Certificate - Dark Mode](/images/getting_a_cert_dark.webp#only-dark)
![Getting a Certificate - Light Mode](/images/getting_a_cert_light.webp#only-light)

--8<-- "include/abbreviations.md"

[^1]:
Fortunately, I did find the book [TLS Mastery](https://mwl.io/nonfiction/networking#tls) by Michael
Lucas. Honestly speaking, I didn't really like the writing style but this book is certainly better
than most documentation you'll find on the Internet about OpenSSL and TLS.
[^2]:
You're using LibreSSL if you're using OpenBSD. You're almost certainly using OpenSSL if you're using
Linux. I guess I now have another reason to run OpenBSD on at least one of my machines.
[^3]:
The complexity of generating a private key should be reduced whenever the CA/B decides to let the
Internet use ED25519 key pairs. In case you're thinking about RSA, no, I'm not gonna use that.
[^4]:
The encryption algorithm used is `aes256 => AES-256-CBC`. I'm not sure if that's fine. However, the
alternative is to use `openssl req` and generate an encrypted private key encrypted using `des`.
[^5]:
[This](https://letsencrypt.org/docs/a-warm-welcome-to-asn1-and-der/#object-identifier) post from
Let's Encrypt seems to go into more details.
[^6]:
If your program has more than a few options to work with, please provide a configuration file rather
than [just environment variables](https://github.com/Cloudef/bemenu/issues/15) or command line
options. In fact, it'd be fine if environment variables and command line flags weren't provided but
a configuration file would be read by the program.
[^7]:
I've seen colleagues log in to websites with their personal information on such devices. I hope you
don't do the same thing.
[^8]:
There's [acme.sh](https://github.com/acmesh-official/acme.sh) and
[dehydrated](https://github.com/dehydrated-io/dehydrated) but they make me uneasy. I don't think
shell scripts should be used for parsing JSON and if you need to write pipelines like
[this](https://github.com/acmesh-official/acme.sh/blob/e384df30faa015481437875c64b59ddf4f62214c/acme.sh#L1064)
or
[this](https://github.com/dehydrated-io/dehydrated/blob/784fb806c891979262eef9c8f38e3c10b825aefd/dehydrated#L68),
it should probably be rewritten in another language which can handle data better than shell scripts.
[^9]:
I replaced uhttpd with nginx in my OpenWRT router because uhttpd can't handle encrypted private
keys. If you don't have any issues leaving your private key unencrypted on your router, stick with
uhttpd.
[^10]:
OpenWRT tries to wrap `acme.sh` with their own script which I didn't like because there's no
documentation on how to use their script. Anyways, I created a symlink in `~/.local/bin` to make
`acme.sh` available in `$PATH` and also used `--home /etc/acme_staging` for all the commands
mentioned to make sure the staging doesn't end up getting mixed with the production environment.
[^11]:
Yes, I changed the name of the domain throughout the article. A bit of an irrational paranoia I
guess?
