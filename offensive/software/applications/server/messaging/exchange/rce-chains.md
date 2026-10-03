---
title: "Exchange RCE chains: ProxyLogon, ProxyShell, ProxyNotShell"
description: "The pre- and post-authentication remote code execution chains against on-premises Exchange: ProxyLogon and ProxyShell SSRF-to-RCE, and the ProxyNotShell PowerShell path, each yielding SYSTEM on the Exchange server."
keywords:
  - ProxyLogon
  - ProxyShell
  - ProxyNotShell
  - Exchange RCE
  - SSRF
---

# Exchange RCE chains

A recurring class of Exchange flaws turns the front-end proxy into a path to **SYSTEM on the server**. The front end (Client Access) proxies requests to the back end and, in these chains, can be tricked into forwarding **attacker-controlled, implicitly-trusted** requests to privileged back-end endpoints (PowerShell, EWS), which then run code. They share a shape: a server-side request forgery or auth confusion, chained to a write or deserialization primitive.

## The chains

- **ProxyLogon**: a pre-auth **SSRF** in the front end plus an arbitrary file write, chained to drop a web shell. Pre-authentication, so it needs only network access to OWA.
- **ProxyShell**: a path-confusion that reaches the PowerShell back end **pre-auth**, abused to create a mailbox export or new mailbox and write a shell. Three bugs chained; version-specific.
- **ProxyNotShell**: an authenticated SSRF plus a remote PowerShell abuse; needs **one valid credential** (from [spraying](password-spraying.md)) and reaches RCE through the PowerShell endpoint. Later bypasses (for example the ApprovedApplicationCollection path) revived it after partial fixes.

```text
# These are version-specific chains; use a maintained exploit matched to the build you fingerprinted
# e.g. metasploit exchange_proxylogon_rce / exchange_proxyshell_rce, or standalone PoCs
# confirm the exact Exchange build from enumeration before firing
```

## Exploitation notes

- **Match the build**: these chains are tightly version-bound, so the [version fingerprint](enumeration.md) decides which one (if any) applies; firing the wrong one just alerts the defender.
- The payoff is **SYSTEM on the Exchange server**, and because Exchange is privileged in AD, that is usually a short step from Domain Admin (dump LSASS/machine account, or pivot via the Exchange machine account).
- Pre-auth chains (ProxyLogon, ProxyShell) work from the **internet** with no credential; ProxyNotShell needs one sprayed credential.
- These are heavily signatured and AMSI-instrumented on current Exchange, so expect detection; they remain devastating on unpatched or legacy builds.

## Tools

- **Metasploit** (`exchange_proxylogon_rce`, `exchange_proxyshell_rce`): maintained modules.
- **Standalone PoCs** matched to the specific build and chain.

## References

- [Google Cloud / Mandiant: PST, want a shell? ProxyShell exploiting Exchange](https://cloud.google.com/blog/topics/threat-intelligence/pst-want-shell-proxyshell-exploiting-microsoft-exchange-servers)
- [Zero Day Initiative: exploiting Exchange PowerShell after ProxyNotShell](https://www.thezdi.com/blog/2024/9/11/exploiting-exchange-powershell-after-proxynotshell-part-2-approvedapplicationcollection)
- [Microsoft: securing Exchange Server with AMSI](https://www.microsoft.com/en-us/security/blog/2025/04/09/stopping-attacks-against-on-premises-exchange-server-and-sharepoint-server-with-amsi/)
