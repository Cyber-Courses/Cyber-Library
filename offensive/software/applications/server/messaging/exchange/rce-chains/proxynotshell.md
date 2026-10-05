---
title: "ProxyNotShell: authenticated Autodiscover SSRF to PowerShell"
description: "The Exchange chain that needs one valid mailbox credential: an authenticated Autodiscover SSRF reaches the remote PowerShell back end for code execution, with the OWASSRF variant pivoting to the /owa/.../ecp path after the URL-rewrite mitigation and a later ApprovedApplicationCollection bypass to reach PowerShell again."
keywords:
  - ProxyNotShell
  - OWASSRF
  - authenticated SSRF
  - Autodiscover
  - PowerShell backend
---

# ProxyNotShell

ProxyNotShell is the single-credential successor to ProxyShell. Where ProxyShell forged its way in pre-auth, ProxyNotShell requires **one valid mailbox credential** (from [spraying](../password-spraying.md)) and uses it to drive an **authenticated Autodiscover SSRF** that reaches the remote **PowerShell** back end, which then runs code as the Exchange identity. The first mitigation Microsoft shipped was a URL-rewrite rule that blocked the Autodiscover SSRF pattern; the **OWASSRF** variant sidestepped that rule by reaching the back end through the `/owa/.../ecp` path instead, and a later bypass reached PowerShell again by abusing the `ApprovedApplicationCollection` handling. The credential is the only gate that never went away.

## Preconditions

- One valid mailbox credential with a mailbox and remote-PowerShell access enabled for that user.
- The build is below the late-2022 security rollup for the full ProxyNotShell path, or a build where the URL-rewrite mitigation was applied but the SSRF fix was not (the OWASSRF window), confirmed against the [version fingerprint](../enumeration.md).
- `/autodiscover` (ProxyNotShell) or `/owa` and the ECP back end (OWASSRF) reachable.

## Step 1: authenticate, then SSRF to PowerShell

Unlike ProxyShell, you supply real credentials. The authenticated front end proxies an Autodiscover request whose embedded path steers to the back-end PowerShell endpoint:

```http
POST /autodiscover/autodiscover.json?@foo.com/powershell/?&Email=autodiscover/autodiscover.json%3F@foo.com HTTP/1.1
Host: mail.example.com
Authorization: Basic <base64 EXAMPLE\john.doe:Password1>
Content-Type: text/xml

<request body selecting the PowerShell endpoint and the WSMan action>
```

A `200` carrying a WSMan/SOAP response (rather than a `401` or a login redirect) means the authenticated SSRF reached the PowerShell back end and you have a runspace. A `401` means the credential is wrong or lacks remote-PowerShell access; a redirect means the Autodiscover pattern is blocked, so try the OWASSRF path below.

## Step 2 (OWASSRF variant): reach the back end through /owa/.../ecp

When the Autodiscover URL-rewrite mitigation is present but the server is otherwise unpatched, the same back-end PowerShell is reachable by routing through the OWA front end into the ECP back end, which the rewrite rule did not cover:

```http
POST /owa/mastermailbox%2f..%2fecp/... HTTP/1.1
Host: mail.example.com
Authorization: Basic <base64 EXAMPLE\john.doe:Password1>
Cookie: <authenticated OWA session cookies>
```

The `mastermailbox/..` segment is the path normalization that lands the request on the ECP/PowerShell back end despite the Autodiscover rewrite. A `200` with a PowerShell runspace response confirms OWASSRF works on this build.

## Step 3: run code in the runspace

With either path open, you hold a remote-PowerShell runspace running as the Exchange identity. From here the write primitives are the same as ProxyShell: drop a shell with a mailbox export, or (quieter) grant impersonation. A single tool chains authentication, SSRF, and the command:

```bash
# Supply the sprayed credential; the tool opens the runspace and runs the command
python3 proxynotshell.py -u 'EXAMPLE\john.doe' -p 'Password1' \
  -t https://mail.example.com -c 'whoami'
# -> nt authority\system  (confirms code execution)
```

If the tool authenticates but reports "runspace denied," the mailbox lacks remote-PowerShell access; pick another sprayed credential whose user is PowerShell-enabled.

## Follow-on

The runspace is `SYSTEM`/Exchange app-pool on the server. Drop a web shell for persistence or grant `ApplicationImpersonation` for stealthy [mailbox access](../mailbox-access.md), then dump credentials and pivot to the domain through Exchange's AD rights, as in the [RCE chains overview](index.md).

## Exploitation notes

- The credential requirement is the defining constraint: pair ProxyNotShell with a [spraying](../password-spraying.md) hit, and make sure that user has a mailbox and remote PowerShell enabled, since not every valid domain account does.
- Try ProxyNotShell's Autodiscover path first; if it redirects, the URL-rewrite mitigation is in place and OWASSRF's `/owa/.../ecp` path is the one to use.
- The quieter exit is to use the runspace to grant impersonation and read mail over EWS, avoiding the export-request object and shell file that a web shell leaves behind.

## Tools

- **ProxyNotShell / OWASSRF PoCs**: authenticated SSRF drivers for both path variants.
- **Metasploit** (Exchange authenticated PowerShell modules): maintained end-to-end options.
- **Exchange remote PowerShell cmdlets**: the back-end write and role-grant primitives once the runspace is open.

## References

- [CrowdStrike: OWASSRF, exploiting Exchange after the ProxyNotShell mitigation](https://www.crowdstrike.com/en-us/blog/owassrf-exploit-analysis-and-recommendations/)
- [Zero Day Initiative: exploiting Exchange PowerShell after ProxyNotShell (ApprovedApplicationCollection)](https://www.thezdi.com/blog/2024/9/11/exploiting-exchange-powershell-after-proxynotshell-part-2-approvedapplicationcollection)
- [Microsoft Security: stopping attacks against on-premises Exchange Server with AMSI](https://www.microsoft.com/en-us/security/blog/2025/04/09/stopping-attacks-against-on-premises-exchange-server-and-sharepoint-server-with-amsi/)
