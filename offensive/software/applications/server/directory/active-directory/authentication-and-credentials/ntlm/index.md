---
title: "NTLM: capturing, relaying, and replaying Windows authentication"
description: "Abusing NTLM authentication in Active Directory: capturing NetNTLM challenge-responses through name-resolution poisoning, coercing authentication from privileged machines, relaying it to other services, and replaying a hash directly with pass-the-hash."
keywords:
  - NTLM
  - NetNTLMv2
  - NTLM relay
  - pass the hash
  - coercion
---

# NTLM

NTLM is the legacy Windows challenge-response authentication protocol, still enabled almost everywhere alongside Kerberos. Its weaknesses are structural: the response is derived directly from the NT hash (so the hash alone authenticates, no password needed), and the protocol has no inherent binding to the service it is sent to (so a captured authentication can be forwarded elsewhere). Those two facts produce the whole family of NTLM attacks.

## The four moves

- **Capture**: force a victim to authenticate to you (by poisoning name resolution), and record the NetNTLMv2 response to crack offline.
- **Coerce**: make a privileged machine, especially a domain controller, authenticate to you on demand through an RPC or protocol trigger.
- **Relay**: forward a captured or coerced authentication to a third service in real time and act as the victim there, without ever knowing the password.
- **Replay (pass-the-hash)**: authenticate directly using a stolen NT hash, since the protocol never needs the plaintext.

Capture and coercion produce the authentication; relay and pass-the-hash consume it. The relay target's protections (SMB signing, [LDAP signing and channel binding](../../../ldap/signing-and-channel-binding.md)) decide whether relay is possible.

## Pages

- **[Net-NTLM capture and poisoning](net-ntlm-capture-and-poisoning.md)**: harvesting authentication with LLMNR/NBT-NS/mDNS poisoning.
- **[Coercion](coercion.md)**: forcing privileged machines to authenticate (PetitPotam, PrinterBug, and others).
- **[Relay](relay.md)**: forwarding authentication to SMB, LDAP, and HTTP targets.
- **[Pass-the-hash](pass-the-hash.md)**: authenticating and executing with a stolen NT hash.

## References

- The Hacker Recipes: abusing NTLM
- Microsoft: NTLM authentication
