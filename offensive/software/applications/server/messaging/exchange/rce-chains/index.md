---
title: "RCE chains: front-end proxy to privileged back end"
order: 3
description: "The on-premises Exchange remote code execution chains that abuse the Client Access front end to reach a privileged back end (PowerShell, ECP, EWS) running as SYSTEM: ProxyLogon, ProxyShell, ProxyNotShell and OWASSRF, and ViewState deserialization. Triage by build to pick the chain in range."
keywords:
  - Exchange RCE
  - ProxyLogon
  - ProxyShell
  - ProxyNotShell
  - ViewState
---

# RCE chains

A recurring class of Exchange flaws shares one shape. The **Client Access (front-end)** service proxies requests to the **back end** and, in these chains, can be tricked into forwarding **attacker-controlled, implicitly-trusted** requests to a privileged back-end endpoint (the remote PowerShell service, ECP, or EWS). The back end trusts the front end and runs the request as `SYSTEM` or as the Exchange application pool identity. The variations are in how the front end is fooled (a cookie-steered SSRF, an Autodiscover URL path confusion) and in the primitive the back end then exposes (a mailbox export that writes a web shell, a PowerShell command, a signed `__VIEWSTATE` that deserializes). Chain any front-end trick to any back-end write and you have code execution.

## Triage: match the build to the chain

The build you read during [enumeration](../enumeration.md) decides which chain is in range. Firing the wrong one against a patched build only generates alerts.

```bash
# read the build, then match it to the in-range chain below
curl -sk -I https://<target>/owa/ | grep -i 'X-OWA-Version'   # e.g. 15.2.1118.x
# 15.0 = 2013, 15.1 = 2016, 15.2 = 2019; compare the build/rollup to the dates below
```

- **[ProxyLogon](proxylogon.md)**, pre-authentication. SSRF via the `X-BEResource` cookie reaches the back-end `/ecp`, authenticates as any mailbox, then an OAB virtual-directory write drops a web shell. In range on builds at or below the early-2021 rollup. No credential.
- **[ProxyShell](proxyshell.md)**, pre-authentication. Autodiscover path confusion reaches the PowerShell back end, which runs `New-MailboxExportRequest` to write a `.aspx` shell. In range on builds below the mid-2021 rollup. No credential.
- **[ProxyNotShell](proxynotshell.md)**, authenticated with one valid mailbox credential. Autodiscover SSRF to the PowerShell back end. In range below the late-2022 rollup; the **OWASSRF** variant on the same page reaches the back end through the `/owa/.../ecp` path after the first URL-rewrite mitigation.
- **[ViewState deserialization](viewstate-deserialization.md)**, needs the ECP `machineKey` (static, leaked, or recovered). A forged `__VIEWSTATE` signed with those keys deserializes to code execution as the ECP app pool. Build-independent; depends on key exposure.

## Why the payoff is so large

The back end runs as `SYSTEM` or the Exchange app-pool identity, so the shell is immediately high-privilege on the box. Because Exchange is privileged in AD (see [PrivExchange](../privexchange.md)), `SYSTEM` on an Exchange server is usually a short step from Domain Admin: dump LSASS and the machine account, then pivot through the Exchange machine account's rights or DCSync.

## Pages

- **[ProxyLogon](proxylogon.md)**: `X-BEResource` SSRF to the back-end ECP, then an OAB write to a web shell.
- **[ProxyShell](proxyshell.md)**: Autodiscover path confusion to the PowerShell back end, then a mailbox export to a shell.
- **[ProxyNotShell](proxynotshell.md)**: authenticated Autodiscover SSRF to PowerShell, with the OWASSRF and ApprovedApplicationCollection variants.
- **[ViewState deserialization](viewstate-deserialization.md)**: forging a signed `__VIEWSTATE` from a leaked `machineKey` to deserialize into the ECP app pool.

## References

- [Google Cloud / Mandiant: ProxyShell exploiting Microsoft Exchange servers](https://cloud.google.com/blog/topics/threat-intelligence/pst-want-shell-proxyshell-exploiting-microsoft-exchange-servers)
- [Orange Tsai: ProxyLogon and the Exchange attack surface (Black Hat / DEVCORE)](https://devco.re/blog/2021/08/06/a-new-attack-surface-on-MS-exchange-part-1-ProxyLogon/)
- [Zero Day Initiative: exploiting Exchange PowerShell after ProxyNotShell](https://www.thezdi.com/blog/2024/9/11/exploiting-exchange-powershell-after-proxynotshell-part-2-approvedapplicationcollection)
