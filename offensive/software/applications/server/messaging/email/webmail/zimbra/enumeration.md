---
title: "Enumeration: accounts and version from the Zimbra SOAP endpoint"
order: 1
description: "Enumerating Zimbra before you hold a token: validating accounts through AuthRequest and login differentials on the public /service/soap endpoint, reading build and configuration from GetInfoRequest and GetVersionInfoRequest, probing autodiscover and /home/ paths, and detecting the admin console on 7071. Worked SOAP envelopes with response interpretation."
keywords:
  - zimbra enumeration
  - zimbra soap authrequest
  - zimbra account enumeration
  - getversioninforequest
  - zimbra admin console detection
---

# Enumeration

Zimbra answers a great deal before it authenticates you. The public SOAP endpoint at `/service/soap` accepts unauthenticated `AuthRequest` envelopes, and the way it responds to a known versus an unknown account validates usernames. Version and configuration leak through SOAP info requests and through client asset paths. The admin console on 7071 is independently detectable. Each result selects the next move: a validated account list to spray or to feed `DelegateAuthRequest`, and an exact build to match against the [RCE chains](rce-chains.md).

## Fingerprint the build

```bash
curl -sk https://<target>/zimbra/ | grep -oiE '/js/[0-9]+\.[0-9]+\.[0-9]+[^"]*' | head
curl -sk https://<target>:7071/zimbraAdmin/ -i | head        # admin console presence
```

The client asset path embeds the ZCS version; a reachable 7071 `/zimbraAdmin/` confirms the admin console is network-exposed, which matters for the admin-token chains. A public, unauthenticated `GetVersionInfoRequest` returns the build directly where it is not restricted:

```http
POST /service/soap HTTP/1.1
Host: target
Content-Type: application/soap+xml

<soap:Envelope xmlns:soap="http://www.w3.org/2003/05/soap-envelope">
  <soap:Body>
    <GetVersionInfoRequest xmlns="urn:zimbraAccount"/>
  </soap:Body>
</soap:Envelope>
```

A populated `<version>` element in the response body pins the patch level. Read it against the chain matrix before firing anything version-specific.

## Account validation

`AuthRequest` is the username oracle. Send a login attempt for a candidate account with a junk password and read how the SOAP fault differs between an account that exists and one that does not. Known accounts return an authentication-failure fault (`account.AUTH_FAILED`); unknown accounts frequently return a distinct fault or a different code/timing, and some configurations return `account.NO_SUCH_ACCOUNT`.

```http
POST /service/soap HTTP/1.1
Host: target
Content-Type: application/soap+xml

<soap:Envelope xmlns:soap="http://www.w3.org/2003/05/soap-envelope">
  <soap:Body>
    <AuthRequest xmlns="urn:zimbraAccount">
      <account by="name">alice@target</account>
      <password>x</password>
    </AuthRequest>
  </soap:Body>
</soap:Envelope>
```

Interpret the `<soap:Fault>`:

```xml
<soap:Fault>
  <soap:Code><soap:Value>soap:Sender</soap:Value></soap:Code>
  <soap:Reason><soap:Text>authentication failed for [alice@target]</soap:Text></soap:Reason>
  <soap:Detail><Error><Code>account.AUTH_FAILED</Code></Error></soap:Detail>
</soap:Fault>
```

An `account.AUTH_FAILED` naming the address back means the account exists (the server reached the password check). A fault that does not echo the address, or a different code, flags a non-existent account. Script the candidate list and classify on the `Code` value and the presence of the echoed address; where codes are uniform, fall back to response-timing deltas as with any login oracle.

## Post-token info (and why enumeration feeds it)

Once any user token is held (sprayed, phished, or minted via the [authentication bypass](authentication-bypass.md)), `GetInfoRequest` dumps the account's configuration, including the server name, data source credentials, and feature set, which maps the backend for the RCE stage:

```http
POST /service/soap HTTP/1.1
Host: target
Content-Type: application/soap+xml
Cookie: ZM_AUTH_TOKEN=<token>

<soap:Envelope xmlns:soap="http://www.w3.org/2003/05/soap-envelope">
  <soap:Header>
    <context xmlns="urn:zimbra"><authToken><![CDATA[<token>]]></authToken></context>
  </soap:Header>
  <soap:Body><GetInfoRequest xmlns="urn:zimbraAccount"/></soap:Body>
</soap:Envelope>
```

The response enumerates the host's `<soapURL>`, mailbox server, and configured data sources, telling you which internal mailbox node the ProxyServlet and `mboximport` chains must target.

## Other surfaces

- **Autodiscover / `/Autodiscover/`** accepts an email address and returns per-account server settings, validating addresses and mapping users to backend hosts.
- **`/home/<user>/`** and the REST interface respond differently for existing versus missing mailboxes, a secondary account oracle where SOAP faults are normalized.
- **`/service/admin/soap` on 443 and the console on 7071** confirm the admin plane is reachable, a precondition for the admin-token chains.

## Follow-on

Feed validated accounts to [authentication bypass](authentication-bypass.md) (spraying, or as the `<account>` target of `DelegateAuthRequest`) and carry the exact build to [RCE chains](rce-chains.md).

## Tools

- [nmap, curl for raw SOAP envelopes](https://curl.se/)
- Burp Suite repeater for iterating `AuthRequest` across a username list.

## References

- [Zimbra SOAP API reference (AuthRequest, GetInfoRequest)](https://wiki.zimbra.com/wiki/SOAP_API_Reference_Material_Beginning_with_ZCS_8.0)
- [Zimbra: account provisioning SOAP (zimbraAdmin namespace)](https://wiki.zimbra.com/wiki/Account_Provisioning)
- [HackTricks: pentesting web services](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-web/index.html)
