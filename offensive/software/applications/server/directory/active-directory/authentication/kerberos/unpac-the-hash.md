---
title: "UnPAC-the-hash: recovering the NT hash from PKINIT"
description: "Recovering an account's NT hash from a PKINIT (certificate) Kerberos authentication by reading the NTLM_SUPPLEMENTAL_CREDENTIAL the KDC returns in the PAC, turning certificate-based access into a reusable hash."
keywords:
  - UnPAC the hash
  - PKINIT
  - NTLM_SUPPLEMENTAL_CREDENTIAL
  - certificate
  - NT hash
---

# UnPAC-the-hash

When an account authenticates with a **certificate** through PKINIT, the KDC still needs to support legacy NTLM for that user, so it returns the account's **NT hash** inside the PAC, in an encrypted `NTLM_SUPPLEMENTAL_CREDENTIAL` structure, so that NTLM can work after a certificate logon. **UnPAC-the-hash** simply reads that hash out of the TGT exchange. It converts any certificate-based access into the account's reusable NT hash, no cracking involved.

## The technique

```bash
# PKINITtools: PKINIT with the certificate, then decrypt the PAC credential
gettgtpkinit.py -cert-pfx cred.pfx example.local/victim victim.ccache
export KRB5CCNAME=victim.ccache
getnthash.py -key <AS-REP-key-from-step-1> example.local/victim

# Certipy does both in one step
certipy auth -pfx victim.pfx -username victim -domain example.local
```

`gettgtpkinit.py` prints the AS-REP encryption key; `getnthash.py` uses it to decrypt the supplemental credential and recover the NT hash.

## Where the certificate comes from

Any path that yields a certificate for the account feeds UnPAC:

- **[Shadow credentials](shadow-credentials.md)**: add a key credential, get a certificate, UnPAC the hash.
- **AD CS enrollment / ESC abuses**: a certificate obtained from a vulnerable template.
- **Relay to AD CS (ESC8)**: a certificate for a coerced machine or user.

## Exploitation notes

- The payoff is a **persistent, reusable NT hash** from what might otherwise be a time-limited certificate, enabling [pass-the-hash](../ntlm/pass-the-hash.md) and [overpass-the-hash](pass-the-key-and-overpass-the-hash.md) long after the certificate work is done.
- It is the standard follow-on to shadow credentials and to any AD CS certificate: get the cert, UnPAC to a hash, move laterally with the hash.
- Works because the KDC includes NTLM material for interoperability; it needs no additional rights beyond holding the certificate.

## Tools

- **Certipy** (`auth`): PKINIT plus hash recovery in one command.
- **PKINITtools** (`gettgtpkinit.py`, `getnthash.py`): the explicit two-step.

## References

- The Hacker Recipes: UnPAC the hash
- Microsoft: PKINIT and the NTLM_SUPPLEMENTAL_CREDENTIAL in the PAC
