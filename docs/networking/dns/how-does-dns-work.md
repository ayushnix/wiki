---
title: "How Does DNS Work?"
summary: "If you're hosting websites, you should know how DNS works."
date: 2021-03-14
---

# Introduction

Here's what I did for hosting my website on GitHub Pages using a custom domain. I'll probably self
host a web server on a VPS as well.

Go ahead and purchase a domain from a registrar which offers decent purchase as well as renewal
prices on domains. Choose a top-level domain (TLD) which doesn't restrict WHOIS privacy and has
DNSSEC. The `.com` TLD fulfills these criteria but the `.in` TLD doesn't. The registrar you choose
should also offer WHOIS privacy by default for free.

As on 14th April 2021, CloudFlare doesn't allow you to buy domain names but does allow you to
transfer domain names to them but this is a time consuming process. After your domain has been
transferred, you'll be locked into CloudFlare's services which might be a bummer depending on your
use case and preferences. However, buying and renewing domains from CloudFlare would be
[cheaper](https://www.cloudflare.com/products/registrar/).

The domain name registrar you use would typically offer their own authoritative name servers. You
can keep using them if you wish. They might also offer free TLS certificates. You can choose to use
them or create your own. For now, we're hosting our website on GitHub Pages and that takes care of
the TLS part. If we use a VPS in the future and host our own content on our own web server, that'd
need messing around with our TLS certs.

# How Does DNS Work?

## DNS Query and Nameservers

I've tried to envision how DNS works, from a bird's eye view, in the following diagram. This might
need some changes though.

![How DNS Works - Dark Mode](/wiki/images/dns-2x-dark.webp){ .dark-mode-img }
![How DNS Works - Light Mode](/wiki/images/dns-2x-light.webp){ .light-mode-img }

Here's what we need to know.

The domain, `ayushnix.com`, is known as the **apex domain** or the **root domain**. You might've
come across the `www` variant on many websites. That's a subdomain, although a special one since it
has been in use since the world wide web came along back in 1989.

You'll enter `ayushnix.com` on your web browser and it'll route your query to your local DNS client
(also sometimes called a stub resolver). In my case, that's
[dnscrypt-proxy](https://github.com/DNSCrypt/dnscrypt-proxy). People call
[Unbound](https://github.com/NLnetLabs/unbound) a recursive DNS resolver but that's probably only
applicable when using Unbound in an actual DNS server setup.

At this point, realistically, our query can often be found in the DNS cache of the client we're
using and gets resolved almost instantaneously. However, if it isn't, it's forwarded to a
**recursive** DNS server. This can be `1.1.1.1` or `8.8.8.8` or something similar. These recursive
DNS servers probably have our query in their cache and if that's the case, we'll get a response
shortly. If not, depending on the situation, the recursive DNS you've queried can either query
(recursion!) the

- authoritative nameservers
- top level domain (TLD) nameservers of `.com`
- root nameservers

These nameservers (NS) are mentioned in decreasing order of the probability of being visited.

If the recursive DNS we're using doesn't have the `A` records of `ayushnix.com`, it'll go to the
authoritative NS that `ayushnix.com` is using. For now, I'm using the NS provided by ~~my domain
name registrar~~ but one can also use authoritative nameservers provided by Cloudflare or other
providers for faster DNS resolution. These authoritative nameservers will provide the required `A`
records and DNS resolution for my website will finish successfully.

If the recursive DNS we're using has neither `A` records nor the authoritative NS records for
`ayushnix.com`, it'll go to the TLD NS of `com` since I'm using the `com` top level domain. It'll
ask for the authoritative NS records of `ayushnix.com` and upon receiving them, will visit the
authoritative NS and get the needed `A` records.

In rare cases, when the recursive DNS we're using doesn't have the address of TLD NS of `com`, it'll
need to go the root nameservers and get the NS records for `com`. As on April 2021, there are 13
root nameservers maintained by various organizations around the world. Obviously, there are lots of
anycast servers behind the scenes. If you want, you can check the list of root nameservers
[here](https://www.internic.net/domain/named.root).

It's interesting to note that most queries to root nameservers aren't legitimate but NXDOMAIN
queries or other queries due to caching issues. [An interesting
case](https://arstechnica.com/gadgets/2020/08/a-chrome-feature-is-creating-enormous-load-on-global-root-dns-servers/)
was published on ArsTechnica which mentioned how a feature in Chrome was generating large amounts of
traffic on root nameservers. A detailed [blog
post](https://blog.apnic.net/2020/08/21/chromiums-impact-on-root-dns-traffic/) was published by
Verisign.

Basically, ISPs often hijack user DNS and show ads for their products on NXDOMAIN pages.
Interestingly, Verisign did this themselves with
[SiteFinder](https://en.m.wikipedia.org/wiki/Site_Finder) making a wildcard DNS record for all
NXDOMAINs in `.com` and `.net` TLD and showing ads. Pretty ironic considering this issue with Chrome
has been highlighted by Verisign themselves. To stop this bullshit, Chrome started generating random
domains to check whether NXDOMAINs were being hijacked and if yes, stopped suggesting the user to
visit `http://myserver` in case the user types `myserver` in the search plus address bar. However,
in case NXDOMAINs weren't being hijacked, Chrome kept checking this and increased load on root
nameservers such that more than half all traffic on the root nameserver run by Verisign came from
Chrome.

## DNS Resource Records

DNS resource records are, apparently, often a source of confusion among website owners. I won't
document each and every resource record in this section. The `A/AAAA` record, `ALIAS/ANAME` record,
and the `CNAME` record are the ones we need to focus on. If you need more information about DNS
resource records, go to [Cloudflare's Learning
Center](https://www.cloudflare.com/learning/dns/dns-records/).

### A Record

Understanding the `A/AAAA` record is simple — it holds the IPv4/IPv6 address for a domain. You can
use multiple `A` records for a single domain, kinda like how [GitHub
Pages](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site#configuring-an-apex-domain)
tells you to point your apex domain to the following IP addresses and create `A` records.

```
185.199.108.153
185.199.109.153
185.199.110.153
185.199.111.153
```

Some providers, like CloudFlare, might use `@` to denote the apex domain.

### CNAME Record

`A` records are straightforward to use. However, when you're dealing with multiple subdomains,
writing `A` records for each of them isn't the best thing we can do. If the IP addresses that all
those `A` records would point to change, we're gonna have a hard time changing all the `A` records
to reflect those changes.

To resolve this issue, we can use `CNAME` records. A `CNAME` record must always point to another
domain, never to an IP address. The most obvious usage of `CNAME` records is pointing the `www`
subdomain variant to the apex domain. In my case, I have a `CNAME` record pointing
`www.ayushnix.com` to `ayushnix.com`.  Here's a truncated output showcasing what I'm talking about.

```
➜ drill ayushnix.com

;; ANSWER SECTION:
ayushnix.com.   3599    IN      A       185.199.109.153
ayushnix.com.   3599    IN      A       185.199.111.153
ayushnix.com.   3599    IN      A       185.199.110.153
ayushnix.com.   3599    IN      A       185.199.108.153

➜ drill www.ayushnix.com

;; ANSWER SECTION:
www.ayushnix.com.       2399    IN      CNAME   ayushnix.github.io.
ayushnix.github.io.     2399    IN      A       185.199.108.153
ayushnix.github.io.     2399    IN      A       185.199.109.153
ayushnix.github.io.     2399    IN      A       185.199.110.153
ayushnix.github.io.     2399    IN      A       185.199.111.153
```

A `CNAME` record can't be used to point the apex domain to another domain. This is a limitation
imposed by IETF specifications although some DNS providers, like Cloudflare, don't care about this
and allow you to do so.
[Here's](https://blog.cloudflare.com/introducing-cname-flattening-rfc-compliant-cnames-at-a-domains-root/)
a good article by Cloudflare about this.

> Technically, the root could be a CNAME but the RFCs state that once a record has a CNAME it can't
> have any other entries associated with it: that's a problem for a root record like example.com
> because it will often have an MX record (so email gets delivered), an NS record (to find out which
> nameserver handles the zone) and an SOA record.

> Because they follow this specification, most authoritative DNS servers won't allow you to include
> CNAME records at the root. At CloudFlare, we decided to let our users include a CNAME at the root
> even though we knew it violated the DNS specification. And that worked, most of the time.
> Unfortunately, there were a handful of edge cases that caused all sorts of problems.

It'd be better if we err on the side of caution here and assume that `CNAME` records can't be used
for anything else (except DNSSEC).

It might not be obvious but there's no restriction regarding a `CNAME` record pointing to another
`CNAME` record or a `CNAME` record using a subdomain pointing to another different domain. Here's an
example.

```
$ drill www.skyscanner.net

;; ->>HEADER<<- opcode: QUERY, rcode: NOERROR, id: 15722
;; flags: qr rd ra ; QUERY: 1, ANSWER: 3, AUTHORITY: 0, ADDITIONAL: 0
;; QUESTION SECTION:
;; www.skyscanner.net.	IN	A

;; ANSWER SECTION:
www.skyscanner.net.	2399	IN	CNAME	www.skyscanner.net.edgekey.net.
www.skyscanner.net.edgekey.net.	2399	IN	CNAME	e11001.a.akamaiedge.net.
e11001.a.akamaiedge.net.	2399	IN	A	23.213.57.166
```

Just so we're clear, `www.skyscanner.net` is a label and `www.skyscanner.net.edgekey.net` is the
CNAME.

As shown, `www.skyscanner.net` points to the `CNAME` `www.skyscanner.net.edgekey.net` which in turn
points to the `CNAME` `e11001.a.akamaiedge.net` which then gives us an IP address.

Here's one of the advantages of using this setup. What if `www.skyscanner.net` suddenly starts
experiencing too much traffic or wants to use a CDN? In this case, the CNAME of
`www.skyscanner.net.edgekey.net` can be changed to something else like `e11316.a.akamaiedge.net`
when needed and the actual website wouldn't experience any downtime or wouldn't need to change
anything.

The case of websites using `CNAME`s for tracking is interesting to note. Here's an example from
[AdGuard's cname-trackers repository](https://github.com/AdguardTeam/cname-trackers).

```
"tybfxw.puma.com": "gum.am5.vip.prod.criteo.com"
```

Yes, a web browser without CNAME tracking protection would consider `tybfxw.puma.com` a first party
domain of `puma.com` but it isn't. It redirects to a subdomain of `criteo.com` which is a well known
advertising company.

[Here's](https://blog.apnic.net/2020/08/04/characterizing-cname-cloaking-based-tracking/) a good
blog post about this issue. Basically, use uBlock Origin on Firefox if you want protection against
CNAME tracking.

### ALIAS/ANAME Record

An `ALIAS/ANAME` record is basically a `CNAME` record without the limitations and inefficiency of
`CNAME` records. Cloudflare calls this `CNAME` flattening.

If we use an `ALIAS` record, we can point `ayushnix.com` to `ayushnix.github.io` without any
worries. This takes care of the primary limitation of `CNAME` records.

To understand how inefficient `CNAME` records can be, let's go back to our `www.skyscanner.net`
example mentioned above.

```
www.skyscanner.net.             🠊 www.skyscanner.net.edgekey.net.
www.skyscanner.net.edgekey.net. 🠊 e11001.a.akamaiedge.net.
e11001.a.akamaiedge.net.        🠊 23.213.57.166
```

In this case, our recursive NS will start with resolving `www.skyscanner.net`. After traveling all
the way to an authoritative NS and back, it has to visit an authoritative server again to resolve
`www.skyscanner.net.edgekey.net`. This process will be repeated for the resolution of
`e11001.a.akamaiedge.net`. We've essentially ended up making three DNS queries to resolve a single
domain.  Obviously, this introduces latency.

In contrast, when using an `ALIAS` record, the authoritative NS of `www.skyscanner.net` should
resolve the underlying domains and return an IP address as the answer to the recursive resolver
we're using. As you can imagine, this is relatively faster and the final answer looks like an `A`
record. This can be confirmed by using `drill` on `ayushnix.com`.

One of the disadvantages of `ALIAS` mentioned on [this blog
post](https://ns1.com/blog/the-difference-between-alias-and-cname) tells us that `ALIAS` records may
make CDN efforts worthless because the CDN will respond to the location of the authoritative
nameserver, not yours.
