---
title: "ADIDNS spoofing: adding DNS records to poison name resolution"
order: 5
description: "Abusing Active Directory-integrated DNS, where authenticated users can create records, to add a wildcard or targeted record (WPAD, SCCM) that turns the domain's DNS server into an enterprise-wide responder for capturing or relaying authentication."
keywords:
  - ADIDNS
  - wildcard record
  - WPAD
  - Powermad
  - dnstool
---

# ADIDNS spoofing

When AD DNS zones are **Active Directory-integrated (ADIDNS)**, the zone is stored in the directory, and by default **Authenticated Users** can **create** child objects (`dnsNode`) in it. So any domain account can **add DNS records**. That is a powerful poisoning primitive: unlike LLMNR/NBT-NS, which only answer failed lookups on the local segment, a record in ADIDNS answers for the **whole domain**, authoritatively, for as long as it exists.

## The wildcard trick

Adding a **wildcard** record (`*`) makes the domain DNS server resolve **every otherwise-unknown name** to your host, turning it into a domain-wide responder, like LLMNR poisoning but enterprise-scale and persistent:

```bash
# dnstool.py (from the krbrelayx repo): add a wildcard A record to your IP
dnstool.py -u 'EXAMPLE\user' -p 'password' --record '*' --action add --data <attacker-ip> <dc>
```

```powershell
# Powermad (on Windows): add a record via secure dynamic update
Invoke-DNSUpdate -DNSType A -DNSName evil -DNSData <attacker-ip>
```

Then run [Responder](net-ntlm-capture-and-poisoning.md) or [ntlmrelayx](relay.md) to capture or relay the authentication that now flows to you.

## Targeted records

Where a wildcard is too noisy, add a single high-value name:

- **WPAD**: a `wpad` record (or an NS record bypassing the Global Query Block List) makes every browser auto-discover your proxy, capturing HTTP auth broadly. The GQBL blocks a direct `wpad` A record, so the NS-record bypass is used.
- **SCCM / WSUS / WDS** names: impersonate update or deployment infrastructure to coerce authentication or push content.
- A specific host whose name you want to hijack for a relay chain.

## Exploitation notes

- The record is **authoritative and domain-wide**, so it reaches clients a link-local poisoner never would, and it persists until removed; clean it up afterwards.
- It needs only a **single domain account**, making it a strong early move to source authentications for [relay](relay.md) (to LDAP for RBCD, or AD CS ESC8).
- DNS caching means effects are not instant and linger after removal; account for TTLs when timing a capture.
- Pair with `mitm6` where IPv6 is available, or use ADIDNS where IPv6 is disabled, to source the same authentications.

## Tools

- **dnstool.py** (krbrelayx repo): add/modify/remove ADIDNS records from Linux.
- **Powermad** (`Invoke-DNSUpdate`): add records from Windows via secure dynamic update.
- **Responder / ntlmrelayx**: capture or relay the redirected authentication.

## References

- [The Hacker Recipes: ADIDNS spoofing](https://www.thehacker.recipes/ad/movement/mitm-and-coerced-authentications/adidns-spoofing)
- [krbrelayx / dnstool (dirkjanm)](https://github.com/dirkjanm/krbrelayx)
- [HackTricks: AD DNS records](https://hacktricks.wiki/en/windows-hardening/active-directory-methodology/ad-dns-records.html)
