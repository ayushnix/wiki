---
title: "SSH Certificates"
summary: "How do SSH certificates work and why should I use them?"
date: 2021-10-19
---

# Introduction

After [reading](https://news.ycombinator.com/item?id=28193942) that I could use subdomains of my
apex domain as FQDN hostnames of my local machines, I thought about getting a wildcard certificate
from Let's Encrypt so that I could have HTTPS for all of my self hosted services without resorting
to using self-signed certificates or [mkcert](https://github.com/FiloSottile/mkcert). At this point,
I realized that I didn't know how TLS certificates work. I thought about understading SSH, GPG, and
then coming back to TLS and this is when I discovered SSH certifcates.

Let's see how deep the rabbit hole goes!

## Password Authentication

If you want to SSH to a machine you've just installed, you might perform password based SSH
authentication for the first time, even if it's just to execute `ssh-copy-id`. How does it work?

``` linenums="1" hl_lines="15-19"
[user@client ~]$ ssh -v remote@server
debug1: Connecting to server port 22.
debug1: Connection established.
debug1: Local version string SSH-2.0-OpenSSH_7.4
debug1: Remote protocol version 2.0, remote software version OpenSSH_7.4
debug1: match: OpenSSH_7.4 pat OpenSSH* compat 0x04000000
debug1: Authenticating to server:22 as 'remote'
debug1: Server host key: ssh-ed25519 SHA256:JQN3c34sPuf2QMGpgxnNYwYipaeaVxuhQUPV4sGmVGZ
debug1: SSH2_MSG_KEXINIT sent
debug1: SSH2_MSG_KEXINIT received
debug1: kex: algorithm: curve25519-sha256
debug1: kex: host key algorithm: ecdsa-sha2-nistp256
debug1: kex: server->client cipher: chacha20-poly1305@openssh.com MAC: <implicit> compression: none
debug1: kex: client->server cipher: chacha20-poly1305@openssh.com MAC: <implicit> compression: none
The authenticity of host 'server' can't be established.
ED25519 key fingerprint is SHA256:JQN3c34sPuf2QMGpgxnNYwYipaeaVxuhQUPV4sGmVGZ
This key is not known by any other names
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'server' (ED25519) to the list of known hosts.
debug1: Authentications that can continue: publickey,password
debug1: Next authentication method: publickey
debug1: Offering public key: /home/user/.ssh/id_ed25519 ED25519 SHA256:xg8HUsaVq2HNOh3W728UBRhwnE3KSQtSLsSWWkYZIMH explicit agent
debug1: Authentications that can continue: publickey,password
debug1: Next authentication method: password
remote@server's password:
Authenticated to server:22 using "password".
[remote@server ~]$
```

The gist of what happens here is

- the client and server exchange information about the OpenSSH versions installed on their systems
  and check for compatibility
- a key exchange method is used to generate a *session key*, a shared secret that is used to encrypt
  the entire session using symmetric encryption[^1]
- the server presents the fingerprint of its public key, the *host key*, for the client to confirm
  that it is actually connecting to the server that the client thinks its connecting to

The *host key* is the ED25519 key fingprint in line 16. This security model, asking a client to
trust that server in question is really the server he wants to connect to by presenting a
fingerprint and using that trust for subsequent sessions, is called
[TOFU](https://developer.mozilla.org/en-US/docs/Glossary/TOFU). When you enter `yes` to the question
posed in line 18, the host key of the server gets added to the clients' `~/.ssh/known_hosts` file.

The host key fingerprint can be confirmed manually by using the following command

```
$ ssh-keygen -l -f /etc/ssh/ssh_host_rsa_key
```

From what I understand, TOFU sounds more like BTOFU (Blind TOFU) because you don't know who you're
connecting to unless you already know who you're connecting to. Also, if the host you're connecting
to ever changes its pair of SSH keys and you connect to it again, you're presented with a scary
warning that might look like this

```
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
Someone could be eavesdropping on you right now (man-in-the-middle attack)!
It is also possible that a host key has just been changed.
The fingerprint for the ED25519 key sent by the remote host is
SHA256:KZ5RxlkdjfasXhcCVsnNHlH6pSCe3pijvE19oGXcB8ruA.
Please contact your system administrator.
Add correct host key in /home/user/.ssh/known_hosts to get rid of this message.
Offending ED25519 key in /home/user/.ssh/known_hosts:8
ED25519 host key for server.ayushnix.com has changed and you have requested strict checking.
Host key verification failed.
```

Obviously, this isn't really a great experience for anyone dealing with SSH, although people often
choose to ignore this warning.

## Key Authentication

It may not be obvious but password based authentication for SSH is the last thing you want. It is
neither scalable nor secure and goes against the basic principles of automation and IaC. The most
widely known alternative is using asymmetric encryption, public and private key pairs[^2], for
authentication.

```
$ ssh-keygen -t ed25519 -f ./sample
Generating public/private ed25519 key pair.
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in ./sample
Your public key has been saved in ./sample.pub
The key fingerprint is:
SHA256:Os72NAl35ndxN4yxWExTTtyyt443TsD948iaStrnbwM

$ cat sample.pub
ssh-ed25519 AAAAQfCueYncamnLiGfZxULxvgrAyQZAPIKaigYv7WzjyYfGm329LF1O6qGtJjZNBuUu
```

The typical workflow now includes appending the contents of `sample.pub`, generated on the client's
machine, to `~/.ssh/authorized_keys` on the server. Once you've done that,

```
[user@client] $ ssh -v remote@server
...
debug1: Authenticating to server:22 as 'remote'
debug1: Host 'server' is known and matches the ED25519 host key.
debug1: Found key in /home/user/.ssh/known_hosts:6
debug1: Authentications that can continue: publickey
debug1: Next authentication method: publickey
debug1: Offering public key: /home/user/.ssh/sample ED25519 SHA256:vVQzNhWmI6HYaBMSTbtUvfxQ69Z0jA3GXcE0MbS1rC8
debug1: Server accepts key: /home/user/.ssh/sample ED25519 SHA256:vVQzNhWmI6HYaBMSTbtUvfxQ69Z0jA3GXcE0MbS1rC8
Authenticated to server:22 using "publickey".
...
[remote@server ~] $
```

Instead of entering passwords on the wire, the client sends

- the public key
- a signature generated using the private key

If the public key sent by the client is present in the server's `authorized_key` file and the
signature sent by the client is cryptographically valid considering the sent public key, the
authentication succeeds. For more information, there isn't a better resource than the [section 7 of
RFC 4252](https://www.rfc-editor.org/rfc/rfc4252#section-7).

### Issues with Key Authentication

Even though we can eliminate security concerns when using password authentication by switching to
key pair based authentication, we're still left with a lot of issues

- management of `known_hosts` and `authorized_keys` isn't scalable

    Does the idea of managing entries in `authorized_keys` and `known_hosts` file sound good when we
    have potentially hundreds or thousands of users and servers in an organization? Even if we were
    to automate this process, we're still left with
    [bad](https://cat.pdx.edu/2021/07/27/changes-to-ssh-host-keys/)
    [UX](https://www.cs.uky.edu/docs/users/ssh.html) when it comes to host key fingerprint
    verification due to TOFU. Sure, you could advertize the host keys at another secure location,
    such as a website with HTTPS, but the clients would still need to make changes to, and maintain,
    their `known_hosts` file which is, again, bad UX. If your security measures end up making a
    client impatient or frustrated or interfere with their workflow, your security measures are
    worthless and wrong.

- `known_hosts` and `authorized_keys` are a source of sensitive information[^3]

    As we saw in the ShapeShift hack mentioned above, `known_hosts` and `authorized_keys` are a
    source of sensitive information. A single client can potentially have a list of hundreds of
    servers in his `known_hosts` which gives a bad actor a lot of potential. The same goes for
    `authorized_keys` file on a server.

- clients might be, intentionally or unintentionally, malicious

    A serious issue when relying only on key based authentication is ensuring that the client
    connecting to your server isn't malicious. If the private keys of the client aren't encrypted,
    it's trivial to impersonate the client if his private keys are stolen. The [hack at
    ShapeShift](https://archive.md/brTMA) is a great example of just how costly it can be to have
    clients connecting to your servers without an encrypted private key[^4].

An obvious solution might be to switch to the IAM solution offered by your cloud provider. However,
not everyone is using AWS and GCP or wants to setup LDAP/FreeIPA.

We'll look at **SSH Certificates**, an authentication solution provided by OpenSSH itself.

# Using SSH Certificates

When using OpenSSH, a certificate authority is just another public and private key pair. Let's
create one for ourselves.

```
$ ssh-keygen -t ed25519 -a 64 -C "ssh-ca@ayushnix.com" -f ./ssh-ca
```

For SSH certificates to be worthwhile, we need to eliminate the management of `authorized_keys` on
the server and `known_hosts` on the client.

We'll start with eliminating the need to rely on TOFU and `known_hosts`.

## Host Certificate

We'll use the certificate authority we created earlier to generate a *host certificate* using the
host key. The host will use this signed host certificate instead of its host key to present its
identity to clients.

The host key of a server is usually found at `/etc/ssh/ssh_host_ed25519_key.pub`.

```
$ ssh-keygen -s .ssh/ssh-ca -h -I server.ayushnix.com -n server,server.ayushnix.com ssh_host_ed25519_key.pub
Signed host key ssh_host_ed25519_key-cert.pub: id "server.ayushnix.com" serial 0 for server valid forever

$ ssh-keygen -L -f ssh_host_ed25519_key-cert.pub
ssh_host_ed25519_key-cert.pub:
        Type: ssh-ed25519-cert-v01@openssh.com host certificate
        Public key: ED25519-CERT SHA256:cg9yimV7rbuuoD57ElUxkd9iCQye4uykD3FpKHjBxSV
        Signing CA: ED25519 SHA256:fxkiT3H3SAPefoUtVm2DfshWkHGkObeI3Caf7z5fCpA (using ssh-ed25519)
        Key ID: "server.ayushnix.com"
        Serial: 0
        Valid: forever
        Principals:
                server
                server.ayushnix.com
        Critical Options: (none)
        Extensions: (none)
```

The `-h` option is needed to mention that we want a host certificate rather than a client
certificate. The `-I` option just mentions a unique identity for the certificate. The `-n` option
specifies *principles*, a list of hostnames, for which the host certificate is valid. We could've
used `-V` to mention the time for which the certificate will be valid and `-z` to mention a serial
number for this certificate. According to the man page of `ssh-keygen (1)`, host certificates don't
have any valid options.

Now that we have a host certificate, we'll make the server use that instead of host keys.

```
[remote@server ~]$ doas nvim /etc/ssh/sshd_config
...
...
HostCertificate /etc/ssh/ssh_host_ed25519_key-cert.pub
```

To make sure that a client doesn't resort to TOFU and accepts host certificates signed by the
certificate authority we created earlier,

```
[user@client ~]$ nvim .ssh/known_hosts
...
...
@cert-authority *.ayushnix.com ssh-ed25519 AAAAtEHSaGBHqhr6GSQNFdEQcBm9pItZWekkoUUIbigCreh1YAF30DSWH03GzGk2ilH4 ssh-ca@ayushnix.com
```

Now, remove entries from `known_hosts` for servers which are now presenting their host certificates
signed by the certificate authority we created and try to SSH again.

```
[user@client ~]$ ssh -v server
...
debug1: Host 'server.ayushnix.com' is known and matches the ED25519-CERT host certificate.
debug1: Found CA key in /home/user/.ssh/known_hosts:2
...
[remote@server ~]$
```

We've successfully eliminated the need to rely on TOFU and managing `known_hosts`. In case it wasn't
clear, here's what happened

- we created a certificate authority, a public and private key pair
- we signed the host key of the server, created a host certificate, and configured SSH to use that
  for verification
- we modified the client's `~/.ssh/known_hosts` file to trust host key certificates signed by the
  certificate authority that we created

As long as the server presents a signed host certificate, the client won't see any TOFU messages.
Even if the server host keys change, all that the server needs to do is get its host keys signed by
the certificate authority and present the new certificate. The client won't see scary TOFU warning
messages.

The client's `known_hosts` file no longer has a list of servers which can be harvested for
potentially malicious activity. The client just needs to maintain a single line in `known_hosts`,
the public key of the certificate authority.

## Client Certificate

Now, we'll eliminate the need for the server to maintain its `authorized_keys` file. We'll use the
certificate authority to create a client certificate.

```
$ ssh-keygen -s .ssh/ssh-ca -I user.ayushnix.com -n remote id_user.pub
Signed user key id_user-cert.pub: id "user.ayushnix.com" serial 0 for remote valid forever

$ ssh-keygen -L -f id_user-cert.pub
id_user-cert.pub:
        Type: ssh-ed25519-cert-v01@openssh.com user certificate
        Public key: ED25519-CERT SHA256:5QpRKY09iaHCIE0peEtFHyr5xFrV2i9wEJk2PFQ4092
        Signing CA: ED25519 SHA256:fxkiT3H3SAPefoUtVm2DfshWkHGkObeI3Caf7z5fCpA (using ssh-ed25519)
        Key ID: "user.ayushnix.com"
        Serial: 0
        Valid: forever
        Principals:
                remote
        Critical Options: (none)
        Extensions:
                permit-X11-forwarding
                permit-agent-forwarding
                permit-port-forwarding
                permit-pty
                permit-user-rc
```

All that's left is to tell the server to trust clients which can present a valid certificate signed
by our certificate authority. The server needs the public key of the certificate authority.

```
[remote@server ~]$ doas nvim /etc/ssh/sshd_config
...
TrustedUserCAKeys /etc/ssh/ssh-ca.pub
```

The client certificate **must** contain at least one principal, a user name that the client will use
on the server, for the client certificate to be valid. If no principal is specified while creating
client certificates, they won't be used for authentication. The server can also whitelist the user
names clients can connect as using the `AuthorizedPrincipalsFile` option.

Go ahead and remove the public key for the client in the `authorized_keys` file on the server and
restart the sshd service.

That's it! We're done.

```
...
debug1: Connecting to server.ayushnix.com port 22.
debug1: identity file /home/user/.ssh/id_user type 3
debug1: identity file /home/user/.ssh/id_user-cert type 7
debug1: Authenticating to server.ayushnix.com:22 as 'remote'
debug1: Host 'server.ayushnix.com' is known and matches the ED25519-CERT host certificate.
debug1: Found CA key in /home/user/.ssh/known_hosts:2
debug1: Offering public key: /home/user/.ssh/id_user ED25519 SHA256:MSiX5FpgZDnLW5qjw7ry59EDN3ZhG7aHSB8InaTiBvJ
debug1: Authentications that can continue: publickey
debug1: Offering public key: /home/user/.ssh/id_user ED25519-CERT SHA256:MSiX5FpgZDnLW5qjw7ry59EDN3ZhG7aHSB8InaTiBvJ
debug1: Server accepts key: /home/user/.ssh/id_user ED25519-CERT SHA256:MSiX5FpgZDnLW5qjw7ry59EDN3ZhG7aHSB8InaTiBvJ
Authenticated to server.ayushnix.com using "publickey".
...
[remote@server ~]$
```

The client can now login to the server just as it did before when it was using key based
authentication but now,

- the client doesn't need to maintain `known_hosts` except a single line
- the server doesn't need to have `authorized_keys`
- SSH doesn't use TOFU to print scary warnings

## Certificate Options

You may have noticed that SSH certificates have `Critical Options:` and `Extensions:`. Instead of
specifiying options inside `authorized_keys`, this is how we grant granular access to clients (in
addition to using principals). The most interesting options are

`force-command`

:   which can force a client to execute a command as soon as it successfully authenticates to a
    server

`source-address`

:   which can limit the IP address from which a client certificate can connect from

You can disable `permit-agent-forwarding` and use `ProxyJump` in your `ssh_config` instead.

Other options that'll probably be of interest are

- `CertificateFile` and `IdentityFile` in `ssh_config`
- `AuthorizedPrincipalsFile` and `AuthorizedPrincipalsCommand` in `sshd_config`

## Difference in Workflow

If you want to grasp the difference in workflow between key based authentication and SSH
certificates, imagine a small organization with 20 servers and 40 clients. We'll showcase what
happens when a new server and a new client comes in to the organization how each workflow works.

### Key Authentication

When a new server is installed,

- create a `authorized_keys` file and enter the list of clients that should be allowed

    This may not be as easy as it sounds though. Do we simply allow all the 40 clients to access this
    server? What if we want only 15 people to access it? Sure, we can write 15 names but then, how do
    you manage the different configurations for `authorized_keys` across 20 servers? What if we want
    granular permissions for each and every client? We could mention the options for granular
    permissions in the `authorized_keys` file itself but can you image the mess of maintaining
    potentially different options for the same user across different servers?

- advertize the its host key fingerprint in a secure manner (or not, and let clients assume
  everything's ok and they'll enter `yes` in the TOFU warning and move on)

When a new client is comes in,

- the client populates his `known_hosts` file manually, one by one for 20 servers, while ignoring
  TOFU warnings in each case

    The `known_hosts` file can be advertized in a secure manner and the client can add it but someone
    needs to keep the list updated and advertize it whenever a new server comes in or a server is
    destroyed (we want to treat our servers as cattle, not pets). The client needs to keep it updated
    or face (and ignore) TOFU warnings.

- the 20 (or maybe less) servers in question need to add the public of the new client in their
  `authorized_keys` file

    The same problems mentioned earlier apply again.

### SSH Certificate Authentication

When a new server is installed,

- get the public key of the certificate authority and trust it in `sshd_config`
- get a host key certificate from the certificate authority and set `sshd_config` to advertize that

When a new client comes in,

- get the public key of the certificate authority and trust it in `known_hosts`
- get a client certificate from the certificate authority

    The certificate authority has to ensure to assign appropriate permissions to the client while
    generating the certificate. As we saw earlier, this can be done using principals and options.
    Embedding the permissions that clients should have inside their certificates is much better than
    maintaining them inside `authorized_keys`.

Yes, that's it.

## Issues with SSH Certificates

SSH Certificates aren't a panacea of user friendliness and security. The primary issues at this
point are

- we need to figure out how to issue certificates in a painless and secure manner

    We can't regress to manual certificate generation and don't want to rely on a ticketing system
    to generate SSH certificates. We want the transition from public key authentication to be as
    smooth and secure as possible.

- certificates shouldn't be valid forever (like we just did before)

    We'd prefer not to rely on key revocation lists. They offer similar maintenance disadvantages
    offered by public key authentication.

- unrelated to SSH certificates but we have to ensure that the user a client wants to connect as
  exists on the server, assuming we want the process to be painless

# Beyond SSH Certificates



# References

Here are some links which I've used as a reference for this post.

- [If you’re not using SSH certificates you’re doing SSH wrong](https://smallstep.com/blog/use-ssh-certificates/)

    [Blog posts from Smallstep about SSH](https://smallstep.com/tags/ssh/) were really helpful in
    understanding how SSH access is supposed to work at scale in large organizations who know what
    they're doing.

- [How to create an SSH certificate authority](https://jameshfisher.com/2018/03/16/how-to-create-an-ssh-certificate-authority/)
- [SSH host identity certification](https://blog.liw.fi/posts/2021/09/28/sshca/) and [SSH CA host and user certificates](https://liw.fi/sshca/)
- [Creating a PKI for OpenSSH](https://hosakacorp.net/p/ssh-pki.html)

--8<-- "include/abbreviations.md"

[^1]:
If you're interested in how this works in more detail, you might find
[this](https://goteleport.com/blog/ssh-handshake-explained/) blog post helpful.
[^2]:
Please stop using RSA, DSA, and ECDSA key pairs. ED25519 key pairs are enough. No, your 4096 bit RSA
key pairs aren't helping. Seriously, let's stop supporting obsolete and outdated protocols and stick
to well tested and modern protocols. WireGuard is a great example. It only supports Curve25519 as
the Diffie Helman function. Nothing else.
[^3]:
We can set the `HashKnownHosts` in `~/.ssh/config` to hash the identification strings and not reveal
sensitive information at the cost of not being able to manage `known_hosts` ourselves if we want to.
Debian 'bullseye' 11 seems to have enabled this option by default. All things considered, if you are
using key based authentication, this option makes sense.
[^4]:
The guy, "Bob", also had RDP access to the laptop in question so even if the SSH key was encrypted,
he could've sniffed the passphrase.
