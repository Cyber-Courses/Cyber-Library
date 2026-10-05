---
title: "Enumeration: service discovery and user probing over an XML stream"
description: "An open XMPP stream answers service discovery (disco#info and disco#items) without credentials, listing the server's supported features and its components such as multi-user chat, proxies, and admin interfaces. Hosted vhosts, and the existence of specific users, are probed through component queries, vCard and roster requests, and in-band registration checks."
keywords:
  - XMPP enumeration
  - service discovery
  - disco#info
  - disco#items
  - XMPP components
---

# Enumeration

XMPP service discovery (XEP-0030) is designed to let clients learn what a server can do, and it answers on a stream that has not authenticated. Two queries carry it: `disco#info` returns a server's or component's supported features (identities and feature namespaces), and `disco#items` returns the child items, which on a server are its components, the hosted multi-user-chat service, file-transfer proxies, and often an admin component. From that you learn the server software, the hosted vhosts, and whether capabilities like in-band registration are enabled, all before you have an account. User existence is then probed with targeted stanzas.

## Preconditions

An opened stream (the server answered your `<stream:stream>` with `<stream:features>`). Discovery works before TLS and before SASL on most deployments; if the server returns `<not-authorized/>` to disco, it requires an authenticated session, in which case self-provision one first via [authentication](authentication.md). Know the server's domain (the `to=` of the stream), since components are named as subdomains of it.

## Worked service discovery

Send an `<iq>` of type `get` carrying the disco#items query, then drill into each returned component with disco#info:

```xml
<!-- list the server's components and services -->
<iq type='get' id='d1' to='target.lan'>
  <query xmlns='http://jabber.org/protocol/disco#items'/>
</iq>

<!-- server response: each item is a component you can target next -->
<iq type='result' id='d1' from='target.lan'>
  <query xmlns='http://jabber.org/protocol/disco#items'>
    <item jid='conference.target.lan' name='Chatrooms'/>
    <item jid='proxy.target.lan' name='SOCKS5 Bytestreams'/>
    <item jid='pubsub.target.lan' name='Publish-Subscribe'/>
  </query>
</iq>

<!-- ask a component what it supports and what it is -->
<iq type='get' id='d2' to='target.lan'>
  <query xmlns='http://jabber.org/protocol/disco#info'/>
</iq>
<iq type='result' id='d2' from='target.lan'>
  <query xmlns='http://jabber.org/protocol/disco#info'>
    <identity category='server' type='im' name='Prosody'/>
    <feature var='jabber:iq:register'/>
    <feature var='http://jabber.org/protocol/muc'/>
  </query>
</iq>
```

Interpretation: each `<item>` is a live component, naming the multi-user-chat service (`conference.`), file proxy, and pubsub node to enumerate further. The `<identity ... name='Prosody'/>` fingerprints the daemon for [server exploitation](server-exploitation.md), and the advertised `<feature var='jabber:iq:register'/>` confirms in-band registration is on, so you can self-provision an account without any credential (see [authentication](authentication.md)).

Enumerate the chat rooms on the MUC component the same way:

```xml
<iq type='get' id='r1' to='conference.target.lan'>
  <query xmlns='http://jabber.org/protocol/disco#items'/>
</iq>
<!-- each returned <item jid='ops@conference.target.lan' name='ops'/> is a room to join -->
```

## Probing user existence and vhosts

```xml
<!-- does this user exist? a vCard request distinguishes a real JID from an empty one -->
<iq type='get' id='v1' to='jdoe@target.lan'><vCard xmlns='vcard-temp'/></iq>
<!-- type='result' with content => user exists; item-not-found => no such account -->

<!-- is in-band registration open, and what fields does it want? -->
<iq type='get' id='reg1' to='target.lan'><query xmlns='jabber:iq:register'/></iq>
<!-- a returned <username/><password/> form => self-provisioning available -->
```

A `type='result'` vCard with populated fields (or presence/last-activity data) confirms the JID is real, while `<item-not-found/>` or an empty result means it is not; iterate a name list to build the user inventory. Additional vhosts appear as extra server domains in disco or in the stream features; repeat discovery against each.

## Follow-on

- The component list names exactly what to attack next: the MUC service for rooms, the admin component for privileged operations, and any HTTP-upload or proxy component for file abuse.
- A confirmed `jabber:iq:register` form means you can mint your own account immediately; go to [authentication](authentication.md).
- Enumerated JIDs feed SASL brute force and the roster of real users to impersonate once you hold an account.

## References

- [XEP-0030 Service Discovery](https://xmpp.org/extensions/xep-0030.html)
- [XEP-0077 In-Band Registration](https://xmpp.org/extensions/xep-0077.html)
- [RFC 6121 (XMPP IM and Presence)](https://datatracker.ietf.org/doc/html/rfc6121)
