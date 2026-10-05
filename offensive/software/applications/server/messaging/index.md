---
title: "Messaging: attacking mail and chat servers"
description: "Offensive techniques against messaging infrastructure: the email protocols (SMTP, IMAP, POP3) and sender authentication, webmail applications, Microsoft Exchange, and real-time chat protocols and team-chat platforms."
keywords:
  - messaging
  - mail server
  - email
  - Exchange
  - instant messaging
  - chat server
---

# Messaging

Messaging servers are high-value targets: they authenticate every user, they are usually reachable from the internet, they are frequently privileged in the surrounding directory, and the mailboxes and chat histories they hold are full of credentials and sensitive intel. Compromising one often yields both a foothold and the ability to speak as trusted employees.

The area splits by the kind of system. Email covers the transport and access protocols and the sender-authentication records that decide whether a spoof lands. Exchange is broken out on its own because its perimeter-plus-Active-Directory position makes it the single most productive mail target in a Windows estate. Instant messaging covers the open chat protocols and the self-hosted team-chat products.

## Triage

```bash
# Mail surface
nmap -sV -p25,110,143,465,587,993,995 <target>      # SMTP / POP3 / IMAP (+TLS)
# Webmail / Exchange front ends
curl -sI https://<target>/owa/ ; curl -sI https://<target>/roundcube/
# Chat surface
nmap -sV -p6667,6697,5222,5269,8448,8065,3000 <target>  # IRC / XMPP / Matrix / Mattermost / Rocket.Chat
```

Open mail ports with a Postfix/Exim/Sendmail banner route to [Email](email/index.md); an OWA/ECP or Autodiscover front end routes to [Exchange](exchange/index.md); a chat port or a self-hosted chat web app routes to [Instant messaging](instant-messaging/index.md).

## Subtopics

- **[Email](email/index.md)**: the mail protocols (SMTP, IMAP, POP3), the SPF/DKIM/DMARC sender-authentication gaps that let a domain be spoofed, and the webmail applications that front a mailbox over HTTP.
- **[Exchange](exchange/index.md)**: on-premises Microsoft Exchange, from enumeration and spraying through the pre- and post-auth RCE chains, directory relay, and mailbox and client abuse.
- **[Instant messaging](instant-messaging/index.md)**: the open chat protocols (IRC, XMPP, Matrix) and the self-hosted team-chat platforms (Mattermost, Rocket.Chat).

## References

- [MITRE ATT&CK: Email Collection (T1114)](https://attack.mitre.org/techniques/T1114/)
- [HackTricks: pentesting mail services](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-smtp/index.html)
- [RFC 5321: Simple Mail Transfer Protocol](https://www.rfc-editor.org/rfc/rfc5321)
