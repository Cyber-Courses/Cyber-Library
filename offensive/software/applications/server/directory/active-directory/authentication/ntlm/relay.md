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

**Lineage.** Relay began as SMBRelay in 2001, reflecting authentication straight back to the origin host. Microsoft blocked the self-relay and then raised SMB and LDAP signing, the MIC, and channel binding over successive releases, each closing a path. Modern relay answers by crossing protocols, forwarding to LDAP, to AD CS web enrollment, and even back into Kerberos, wherever signing or channel binding is still unenforced.

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

# Relay to LDAP to add Shadow Credentials on a victim (no RBCD computer needed)
ntlmrelayx.py -t ldap://<dc> --shadow-credentials --shadow-target 'victim$'
```

`ntlmrelayx` also relays to **MSSQL** and Exchange HTTP endpoints, and supports **multi-relay**: it answers the victim with an HTTP **307 redirect** so the client re-authenticates for each additional target. A single captured challenge-response cannot satisfy several server challenges, so this is one **coercion trigger** driving repeated fresh authentications, not one authentication reused many times, and it only works when the coerced client follows the redirects.

- **Relay to LDAP + RBCD**: configure resource-based constrained delegation on a computer object you can then impersonate any user to, a common path from coerced machine authentication to local admin on that machine.
- **Relay to AD CS (ESC8)**: relay a coerced DC or user HTTP authentication to the CA web enrollment and obtain a certificate as that principal; a DC certificate is domain compromise.
- **Relay to SMB**: execute or dump SAM on member servers that do not require signing.

## The full chain

Coercion and relay combine into the canonical unauthenticated-to-domain path: **coerce a DC over HTTP (WebClient) -> relay to AD CS ESC8 -> DC certificate -> authenticate as the DC -> DCSync.** Each link depends on one missing protection (HTTP coercion possible, EPA off on the CA).

## Exploitation notes

- You cannot relay an authentication back to the host that produced it in modern Windows (the reflection fix), so relay is always to a *different* target.
- Machine-account authentications are fully relayable even though their passwords are uncrackable, which is why coercing `DC$` and relaying beats trying to crack it.
- Pair relay with `mitm6` (IPv6 DNS takeover) to source authentications broadly and relay to LDAP for domain-wide RBCD.
- **Windows Server 2025 narrows the classics, but per host and per service**: a 2025 **DC** enables LDAP channel binding and SMB signing more widely, so relay-to-LDAP(S) against it dies. **ESC8 depends on the AD CS web-enrolment host, not the DC**: it closes only if the CA's host runs a build/configuration with EPA and HTTPS enforced, so in a mixed estate (2025 DC, older CA with EPA still off) ESC8 remains viable. Check the CA host's version and EPA state separately from the DC's. New reflective techniques (abusing the SMB sign/seal negotiation flags) keep relay alive regardless, so test the actual posture rather than assuming.

## Tools

- **ntlmrelayx.py** (Impacket): the relay engine, with `--delegate-access` (RBCD), `--shadow-credentials`, `--adcs` (ESC8), SMB exec, LDAP dump, and multi-relay.
- **Responder / mitm6**: source the authentications to relay.
- **krbrelayx**: related Kerberos / unconstrained-delegation relay paths.

## References

- [SpecterOps: the renaissance of NTLM relay attacks](https://posts.specterops.io/the-renaissance-of-ntlm-relay-attacks-everything-you-need-to-know-abfc3677c34e)
- [SecureAuth: a technical guide to relaying credentials everywhere](https://secureauth.com/resources/blog/we-love-relaying-credentials-technical-guide)
- [Impacket ntlmrelayx](https://github.com/fortra/impacket)
- [The Hacker Recipes: NTLM relay](https://www.thehacker.recipes/ad/movement/ntlm/relay)
