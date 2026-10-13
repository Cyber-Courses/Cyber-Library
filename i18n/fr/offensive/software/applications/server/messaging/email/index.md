---
title: "Email : attaquer les protocoles de messagerie, l'authentification des expéditeurs et le webmail"
order: 1
description: "La surface d'attaque de l'email : les protocoles de transport et d'accès (SMTP, IMAP, POP3), les enregistrements d'authentification d'expéditeur SPF, DKIM et DMARC qui décident si une usurpation est délivrée, et les applications de webmail qui exposent une boîte aux lettres sur HTTP."
keywords:
  - email
  - SMTP
  - IMAP
  - POP3
  - SPF DKIM DMARC
  - webmail
---

# Email

L'email se décompose en trois surfaces d'attaque atteintes très différemment. Les protocoles filaires (SMTP pour le transport, IMAP et POP3 pour l'accès) sont attaqués au niveau du service : énumération, relais, authentification et failles du logiciel serveur. L'authentification d'expéditeur (SPF, DKIM, DMARC) est attaquée au niveau du DNS et du destinataire : des enregistrements faibles ou absents permettent à un attaquant d'envoyer du courrier qui usurpe le domaine tout en passant les contrôles. Le webmail est attaqué comme une application web : le frontal HTTP au-dessus de la boîte aux lettres, où les bogues de XSS, de SSRF et d'écriture de fichiers se transforment en prise de contrôle de la boîte aux lettres et en exécution de code.

## Triage

```bash
nmap -sV -p25,110,143,465,587,993,995 <target>     # which mail protocols are live
dig +short TXT <domain> | grep -i spf              # SPF present and how strict
dig +short TXT _dmarc.<domain>                     # DMARC policy: none / quarantine / reject
curl -skI https://<target>/ | grep -i -E 'roundcube|horde|zimbra'   # webmail fingerprint
```

Un service SMTP/IMAP/POP3 actif oriente vers [Protocoles](protocols/index.md) ; un enregistrement SPF/DMARC permissif ou absent oriente vers [Authentification d'expéditeur](sender-authentication/index.md) ; une page de connexion webmail oriente vers [Webmail](webmail/index.md).

## Sous-thèmes

- **[Protocoles](protocols/index.md)** : SMTP, IMAP et POP3, attaqués au niveau du service pour l'énumération, le relais, l'authentification et l'exploitation serveur.
- **[Authentification d'expéditeur](sender-authentication/index.md)** : lacunes des enregistrements SPF, DKIM et DMARC, et l'usurpation qui en abuse.
- **[Webmail](webmail/index.md)** : Roundcube, Horde et Zimbra, attaqués comme des applications web placées devant le magasin de courrier.

## Références

- [RFC 5321 : SMTP](https://www.rfc-editor.org/rfc/rfc5321)
- [RFC 7489 : DMARC](https://www.rfc-editor.org/rfc/rfc7489)
- [HackTricks : pentesting SMTP, IMAP, POP3](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-smtp/index.html)
