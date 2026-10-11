---
title: "Protocoles de messagerie : attaquer les services SMTP, IMAP et POP3"
order: 2
description: "Attaquer les protocoles filaires de la messagerie au niveau du service : SMTP pour le transport et la soumission du courrier, et IMAP et POP3 pour l'accès aux boîtes aux lettres, à travers l'énumération, le relayage, l'authentification, la sécurité du transport et l'exploitation du logiciel serveur."
keywords:
  - SMTP
  - IMAP
  - POP3
  - mail protocol
  - mail server
---

# Protocoles

Les protocoles de messagerie s'attaquent au niveau du service en cours d'exécution. SMTP déplace le courrier entre serveurs (25) et accepte la soumission authentifiée (587, 465) ; IMAP (143, 993) et POP3 (110, 995) remettent une boîte aux lettres à un client. Chacun expose les mêmes questions récurrentes : pouvez-vous énumérer les comptes, relayer ou pulvériser contre lui, dépouiller sa sécurité de transport, et le logiciel serveur lui-même est-il exploitable.

## Triage

```bash
nmap -sV -p25,110,143,465,587,993,995 --script \
  smtp-commands,smtp-open-relay,imap-capabilities,pop3-capabilities <target>
# read the banner to identify the daemon, then route to the matching page
printf 'EHLO x\r\nQUIT\r\n' | nc <target> 25     # advertised verbs: AUTH, STARTTLS, VRFY, SIZE
```

Un service 25/587/465 renvoie vers [SMTP](smtp/index.md) ; un service 143/993 vers [IMAP](imap/index.md) ; un service 110/995 vers [POP3](pop3/index.md). La bannière (Postfix, Exim, Sendmail, Dovecot, Cyrus) décide aussi du chemin d'exploitation du serveur à l'intérieur de chacun.

## Sous-rubriques

- **[SMTP](smtp/index.md)** : énumération des utilisateurs, open relay, authentification, downgrade STARTTLS, SMTP smuggling et exploitation de l'agent de transfert de courrier.
- **[IMAP](imap/index.md)** : énumération, authentification, accès à la boîte aux lettres et exploitation du serveur.
- **[POP3](pop3/index.md)** : énumération, authentification, accès à la boîte aux lettres et exploitation du serveur.

## Références

- [RFC 5321 : SMTP](https://www.rfc-editor.org/rfc/rfc5321)
- [RFC 3501 : IMAP4rev1](https://www.rfc-editor.org/rfc/rfc3501)
- [RFC 1939 : POP3](https://www.rfc-editor.org/rfc/rfc1939)
