---
title: "SSH MITM: interposing to capture credentials and hijack the session"
description: "With a network position and a defeated host-key check, an attacker runs an SSH machine-in-the-middle proxy that terminates the client's connection with a substitute host key and relays to the real server. This captures the password or key material the client submits, records the plaintext session, and allows injecting commands into it."
keywords:
  - ssh mitm
  - host key substitution
  - credential capture
  - session hijack
  - ssh-mitm
---

# SSH MITM

An SSH machine-in-the-middle combines a network position with a defeated host-key check to sit between client and server. The attacker's proxy accepts the client's connection presenting its own host key (trusted because of a [known-hosts bypass](known-hosts-bypass.md)), authenticates the client against itself, and relays to the real server. Because the proxy decrypts and re-encrypts, it sees everything the client sends: the password in a password login, or, for key auth, the ability to use the client's agent-forwarded key or capture the session. It records the full plaintext session and can inject commands the user never typed.

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
- For public-key auth the private key is not transmitted, but if the client enables agent forwarding, the MITM can use the forwarded agent to authenticate onward as the user while the session is live; without forwarding, the proxy still records and can inject into the session.
- The whole attack depends on the host-key check failing (first use, strict-checking off, or a pre-poisoned `known_hosts`); against a client that pins and enforces the key, the substitute key triggers a hard failure.
- Weak negotiated crypto ([weak ciphers](../weak-cryptography/weak-ciphers.md)) is a separate route to attacking a captured session, but the host-key MITM is the direct credential-capture path.

## Tools

- [ssh-mitm](https://github.com/ssh-mitm/ssh-mitm)
- [dsniff/arpspoof (positioning)](https://www.monkey.org/~dugsong/dsniff/)

## References

- [ssh-mitm documentation](https://docs.ssh-mitm.at/)
- [HackTricks: SSH MITM](https://book.hacktricks.xyz/network-services-pentesting/pentesting-ssh)
