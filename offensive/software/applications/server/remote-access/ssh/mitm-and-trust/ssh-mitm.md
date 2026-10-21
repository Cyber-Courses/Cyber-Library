---
title: "SSH MITM: interposing to capture credentials and hijack the session"
description: "With a network position and a defeated host-key check, an attacker runs an SSH machine-in-the-middle proxy that terminates the client's connection with a substitute host key and relays to the real server. This can capture passwords, record or alter the plaintext session, and abuse an explicitly forwarded agent, but it does not expose the client's private key."
keywords:
  - ssh mitm
  - host key substitution
  - credential capture
  - session hijack
  - ssh-mitm
---

# SSH MITM

An SSH machine-in-the-middle combines a network position with a defeated host-key check to sit between client and server. The attacker's proxy accepts the client's connection using its own host key (trusted because of a [known-hosts bypass](known-hosts-bypass.md)) and establishes a separate connection to the real server. Because the proxy terminates both encrypted connections, it can capture a password and reuse it for the onward login, then record or alter the plaintext session. Public-key authentication instead sends a public key and a signature bound to the client's SSH session, not the private key or a signature reusable on the proxy's separate server connection. Onward authentication with that identity therefore requires another capability, such as access to an agent that the client explicitly forwards after authenticating to the proxy.

```bash
# 1. gain an on-path position to the target SSH (ARP spoof, rogue DNS, route)
arpspoof -i eth0 -t <client> <gateway>; sysctl -w net.ipv4.ip_forward=1
# 2. run an SSH MITM proxy that presents a substitute host key and relays
ssh-mitm server --remote-host <real-server>    # captures creds + logs session
# 3. the client connects (to the attacker), is prompted/accepts the host key,
#    submits its password (captured) or forwards its agent (abusable)
```

## Exploitation notes

- Password logins are fully captured: the proxy sees the cleartext password the client sends, directly yielding the credential.
- For public-key auth the private key is not transmitted and the session-bound authentication signature cannot be reused on the proxy's separate connection. If the client enables agent forwarding, the MITM can request new signatures from the forwarded agent while it remains available; otherwise the proxy needs separate credentials or another authentication path before it can relay a real server session.
- The whole attack depends on the host-key check failing (first use, strict-checking off, or a pre-poisoned `known_hosts`); against a client that pins and enforces the key, the substitute key triggers a hard failure.
- Legacy negotiated crypto ([weak ciphers](../weak-cryptography/weak-ciphers.md)) requires its own cipher-specific attack and preconditions; advertising an old option does not provide the direct credential-capture path that failed host-key verification does.

## Tools

- [ssh-mitm](https://github.com/ssh-mitm/ssh-mitm)
- [dsniff/arpspoof (positioning)](https://www.monkey.org/~dugsong/dsniff/)

## References

- [ssh-mitm documentation](https://docs.ssh-mitm.at/)
- [HackTricks: SSH MITM](https://book.hacktricks.xyz/network-services-pentesting/pentesting-ssh)
