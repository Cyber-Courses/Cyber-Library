---
title: "Messagerie : attaquer les serveurs de mail et de chat"
order: 5
description: "Techniques offensives contre l'infrastructure de messagerie : les protocoles de courriel (SMTP, IMAP, POP3) et l'authentification de l'expéditeur, les applications de webmail, Microsoft Exchange, et les protocoles de chat en temps réel ainsi que les plateformes de chat d'équipe."
keywords:
  - messaging
  - mail server
  - email
  - Exchange
  - instant messaging
  - chat server
---

# Messagerie

Les serveurs de messagerie sont des cibles de grande valeur : ils authentifient chaque utilisateur, ils sont généralement accessibles depuis internet, ils sont fréquemment privilégiés dans l'annuaire environnant, et les boîtes aux lettres et historiques de chat qu'ils hébergent regorgent d'identifiants et de renseignements sensibles. Compromettre l'un d'eux offre souvent à la fois un point d'ancrage et la capacité de parler au nom d'employés de confiance.

Le domaine se divise selon le type de système. Le courriel couvre les protocoles de transport et d'accès ainsi que les enregistrements d'authentification de l'expéditeur qui déterminent si une usurpation réussit. Exchange est traité à part parce que sa position à la fois en périmètre et dans Active Directory en fait la cible de mail la plus productive dans un parc Windows. La messagerie instantanée couvre les protocoles de chat ouverts et les produits de chat d'équipe auto-hébergés.

## Triage

```bash
# Mail surface
nmap -sV -p25,110,143,465,587,993,995 <target>      # SMTP / POP3 / IMAP (+TLS)
# Webmail / Exchange front ends
curl -sI https://<target>/owa/ ; curl -sI https://<target>/roundcube/
# Chat surface
nmap -sV -p6667,6697,5222,5269,8448,8065,3000 <target>  # IRC / XMPP / Matrix / Mattermost / Rocket.Chat
```

Les ports de mail ouverts avec une bannière Postfix/Exim/Sendmail mènent à [Courriel](email/index.md) ; un front end OWA/ECP ou Autodiscover mène à [Exchange](exchange/index.md) ; un port de chat ou une application web de chat auto-hébergée mène à [Messagerie instantanée](instant-messaging/index.md).

## Sous-thèmes

- **[Courriel](email/index.md)** : les protocoles de mail (SMTP, IMAP, POP3), les failles d'authentification d'expéditeur SPF/DKIM/DMARC qui permettent d'usurper un domaine, et les applications de webmail qui exposent une boîte aux lettres via HTTP.
- **[Exchange](exchange/index.md)** : Microsoft Exchange sur site, de l'énumération et du spraying jusqu'aux chaînes de RCE pré- et post-authentification, au relais d'annuaire, et aux abus de boîtes aux lettres et de clients.
- **[Chat](instant-messaging/index.md)** : les protocoles de chat ouverts (IRC, XMPP, Matrix) et les plateformes de chat d'équipe auto-hébergées (Mattermost, Rocket.Chat).

## Références

- [MITRE ATT&CK: Email Collection (T1114)](https://attack.mitre.org/techniques/T1114/)
- [HackTricks: pentesting mail services](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-smtp/index.html)
- [RFC 5321: Simple Mail Transfer Protocol](https://www.rfc-editor.org/rfc/rfc5321)
