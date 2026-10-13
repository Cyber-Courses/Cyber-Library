---
title: "XMPP: attacking Jabber servers and their XML streams"
order: 2
description: "XMPP on TCP 5222 (client, STARTTLS), 5223 (legacy TLS), and 5269 (server-to-server federation), served by ejabberd, Prosody, or Openfire. The surface is unauthenticated service discovery over an open XML stream, SASL and in-band registration abuse for account access, mechanism downgrade to cleartext, and server flaws such as the Openfire admin-console path-traversal auth bypass."
keywords:
  - XMPP
  - Jabber
  - port 5222
  - ejabberd
  - Openfire
---

# XMPP

XMPP (Jabber) is an XML-streaming messaging protocol. Clients connect on TCP 5222 with STARTTLS, legacy clients on 5223 with implicit TLS, and servers federate to each other on 5269. The common server implementations are ejabberd, Prosody, and Openfire; the protocol is identical across them, but the server flaws are not, so fingerprinting the daemon is the first step. The protocol is conversational XML: you send an opening `<stream:stream>` and exchange `<iq>`, `<message>`, and `<presence>` stanzas. Much of the offensive value needs no account at all, because service discovery, feature advertisement, and in-band registration all answer on an unauthenticated stream.

Open a stream to fingerprint the server:

```bash
nmap -p5222,5269 -sV --script xmpp-info <target>

# open a client stream by hand (the server echoes its features and often its software)
printf "<?xml version='1.0'?><stream:stream to='target.lan' xmlns='jabber:client' xmlns:stream='http://etherx.jabber.org/streams' version='1.0'>" | nc <target> 5222
```

The returned `<stream:features>` lists the SASL mechanisms and whether STARTTLS is required or merely offered; the `<stream:stream from='target.lan' ...>` and any software banner distinguish ejabberd, Prosody, and Openfire.

## Triage

- Does the stream open and advertise features without TLS being enforced? That enables mechanism downgrade and cleartext capture, covered in [authentication](authentication.md).
- Does service discovery answer on an unauthenticated stream, and is in-band registration enabled? That drives [enumeration](enumeration.md) and self-provisioning.
- Which daemon is it, and is it an Openfire version with the admin-console path-traversal auth bypass? That drives [server exploitation](server-exploitation.md).

## Subtopics

- **[Enumeration](enumeration.md)**: open an XML stream and use disco#info/disco#items to list server features, components, and vhosts, and probe user existence via roster, vCard, and registration checks.
- **[Authentication](authentication.md)**: SASL (PLAIN/SCRAM/DIGEST-MD5), in-band registration to self-provision accounts, mechanism downgrade to cleartext, and SASL brute force.
- **[Server exploitation](server-exploitation.md)**: the Openfire admin-console path-traversal auth bypass chained to malicious plugin upload for RCE, plus ejabberd and Prosody module and TLS issues.

## References

- [RFC 6120 (XMPP Core)](https://datatracker.ietf.org/doc/html/rfc6120)
- [RFC 6121 (XMPP Instant Messaging and Presence)](https://datatracker.ietf.org/doc/html/rfc6121)
- [XEP-0030 Service Discovery](https://xmpp.org/extensions/xep-0030.html)
