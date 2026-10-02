---
title: "NTLM relay: forwarding authentication to act as the victim"
description: "Forwarding captured or coerced NTLM authentication in real time to a third service (SMB, LDAP, HTTP, AD CS) to act as the victim without knowing the password, and the signing and channel-binding protections that stop it."
keywords:
  - NTLM relay
  - ntlmrelayx
  - RBCD
  - ESC8
  - SMB signing
---

# NTLM relay

NTLM has no binding between the authentication and the service it was meant for. So an authentication produced for server A can be **forwarded** to server B, where you complete it and act as the victim, all without ever knowing their password. Relay turns a [captured](net-ntlm-capture-and-poisoning.md) or [coerced](coercion.md) authentication directly into access on another system. What you can relay to is decided entirely by the target's signing posture.

## What stops a relay

Signing and channel binding bind the authentication to the session or the TLS channel, breaking the forward:

- **SMB signing**: if the SMB target requires signing, SMB relay to it fails. DCs require it; member servers often do not.
- **[LDAP signing and channel binding](../../../ldap/signing-and-channel-binding.md)**: LDAP relay needs signing **not** enforced; LDAPS relay additionally needs channel binding (EPA) off. DCs with both enforced cannot be relayed to over LDAP.
- **HTTP (EPA)**: AD CS web enrollment and other HTTP endpoints are relayable when Extended Protection for Authentication is not enforced.

The job is to find a path where the *source* can be coerced and the *target* does not enforce the relevant protection.

## High-value relay targets

```bash
# Relay to LDAP: set RBCD on the victim computer object, or dump the domain
ntlmrelayx.py -t ldap://<dc> --delegate-access --no-dump

# Relay coerced DC auth to AD CS web enrollment (ESC8) -> DC certificate
ntlmrelayx.py -t http://<ca>/certsrv/certfnsh.asp --adcs --template DomainController

# Relay to SMB on a non-signing host and execute
ntlmrelayx.py -t smb://<host> -c 'whoami'
```

- **Relay to LDAP + RBCD**: configure resource-based constrained delegation on a computer object you can then impersonate any user to, a common path from coerced machine authentication to local admin on that machine.
- **Relay to AD CS (ESC8)**: relay a coerced DC or user HTTP authentication to the CA web enrollment and obtain a certificate as that principal; a DC certificate is domain compromise.
- **Relay to SMB**: execute or dump SAM on member servers that do not require signing.

## The full chain

Coercion and relay combine into the canonical unauthenticated-to-domain path: **coerce a DC over HTTP (WebClient) -> relay to AD CS ESC8 -> DC certificate -> authenticate as the DC -> DCSync.** Each link depends on one missing protection (HTTP coercion possible, EPA off on the CA).

## Exploitation notes

- You cannot relay an authentication back to the host that produced it in modern Windows (the reflection fix), so relay is always to a *different* target.
- Machine-account authentications are fully relayable even though their passwords are uncrackable, which is why coercing `DC$` and relaying beats trying to crack it.
- Pair relay with `mitm6` (IPv6 DNS takeover) to source authentications broadly and relay to LDAP for domain-wide RBCD.

## Tools

- **ntlmrelayx.py** (Impacket): the relay engine, with `--delegate-access` (RBCD), `--adcs` (ESC8), SMB exec, and LDAP dump.
- **Responder / mitm6**: source the authentications to relay.
- **krbrelayx**: related Kerberos/unconstrained-delegation relay paths.

## References

- The Hacker Recipes: NTLM relay
- Microsoft: SMB signing, LDAP channel binding and signing
