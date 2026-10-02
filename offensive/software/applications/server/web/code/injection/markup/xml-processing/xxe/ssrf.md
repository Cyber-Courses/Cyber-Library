---
title: "SSRF through XXE external entities"
description: "Pointing a SYSTEM entity at internal hosts or cloud metadata turns an XML parser into a server-side request forgery primitive."
keywords:
  - XXE SSRF
  - cloud metadata
  - 169.254.169.254
  - internal port probing
  - server-side request forgery
---

# SSRF

An external entity does not have to name a `file://` path. Point it at an `http://` URL and the XML parser becomes a server-side request forgery engine: it issues the request from inside the target network, with the service's own source address and whatever internal reachability that implies. This turns XXE into a way to reach hosts, ports, and metadata endpoints the attacker cannot touch directly.

## Reaching internal services

Declare the entity against an internal URL and reference it in a reflected field:

```xml
<?xml version="1.0"?>
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM "http://192.168.0.10:8080/admin">
]>
<stockCheck><productId>&xxe;</productId></stockCheck>
```

The parser fetches the URL and, in the in-band case, splices the response body into the document where `&xxe;` sits. Admin panels, actuator endpoints, message queues, and other services bound to `localhost` or an internal subnet become readable through the parser even when a firewall blocks them from outside.

## Cloud metadata

The highest-value target in cloud environments is the instance metadata service at the link-local address `169.254.169.254`. On AWS it exposes temporary IAM credentials:

```xml
<!DOCTYPE foo [
  <!ENTITY xxe SYSTEM
    "http://169.254.169.254/latest/meta-data/iam/security-credentials/">
]>
<stockCheck><productId>&xxe;</productId></stockCheck>
```

The first request returns the role name, and a second entity appends it to the path to fetch the `AccessKeyId`, `SecretAccessKey`, and `Token`:

```
http://169.254.169.254/latest/meta-data/iam/security-credentials/<role-name>
```

Those credentials are usable against the account's APIs until they expire. Equivalent paths exist on other providers: GCP metadata under `http://metadata.google.internal/computeMetadata/v1/` (though it requires a header many parsers cannot set), and Azure under `http://169.254.169.254/metadata/instance`. Where the entity cannot carry the required header, the metadata read may only succeed on providers that do not enforce one or on older IMDSv1-style endpoints.

## Port and host probing

Even when the response body is not reflected, the parser's behavior leaks information. Point entities at a range of internal addresses and ports and distinguish the outcomes:

```xml
<!ENTITY xxe SYSTEM "http://10.0.0.5:22/">
```

A connection that is accepted, refused, or times out produces a different error message, a different response time, or a different status in the application. An open port that speaks an unexpected protocol often returns a parser error that differs from the connection-refused case, so comparing responses maps which hosts are alive and which ports are open across the internal range. Timing alone separates a filtered port (long timeout) from a closed one (fast reset), giving a blind internal port scan driven entirely through the XML parser.

For fully blind targets, combine this with the out-of-band pattern so the fetched internal response is exfiltrated to an attacker server rather than inferred from errors.

## Tools

- **Burp Suite**: pointing SYSTEM entities at internal hosts and metadata endpoints.
- **Burp Collaborator**: confirming blind SSRF reach out-of-band.
- **XXEinjector**: automating entity-driven internal requests.

## References

- [OWASP: Server Side Request Forgery](https://owasp.org/www-community/attacks/Server_Side_Request_Forgery)
- [PayloadsAllTheThings: XXE Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/XXE%20Injection)
