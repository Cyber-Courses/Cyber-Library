---
order: 2
title: "Injection de mail"
description: "Abus d'email au niveau applicatif où de l'entrée non fiable atteint les en-têtes du message, le corps, ou la remise SMTP et est interprétée comme partie du message plutôt que traitée comme des données."
keywords:
  - injection de mail
  - injection d'en-tête d'email
  - injection CRLF
  - injection SMTP
  - injection MIME
---

# Mail

L'injection de mail couvre l'abus au niveau applicatif de la composition d'email : un attaquant fournit de l'entrée qu'une application concatène dans un message et que la pile mail interprète ensuite comme de la structure plutôt que du contenu.

La primitive partagée est une frontière de confiance qu'un formulaire web ou une API franchit lorsqu'il construit un message à partir de données utilisateur. Formulaires de contact, fonctionnalités « inviter un ami », mailers de réinitialisation de mot de passe, ticketing de support et inscriptions à une newsletter prennent tous des champs tels que le nom de l'expéditeur, le sujet et le destinataire et les remettent à un appel `mail()`, à une bibliothèque de mailer, ou à une conversation SMTP brute. Lorsque ces champs passent sans séparation, le saut de ligne et les tokens de protocole qui délimitent les en-têtes, le corps, les parties MIME et les commandes SMTP deviennent contrôlés par l'attaquant. Le sous-arbre se divise selon l'endroit où l'interprétation se produit : injection d'en-tête au niveau du bloc d'en-têtes, abus multipart MIME au niveau de la structure du message, et injection de segment SMTP au niveau de la remise du protocole.

## Pages

- **[Injection d'en-tête](header-injection.md)** : injecter du CRLF (%0d%0a) dans Subject, From ou To pour que l'entrée concaténée ajoute des destinataires Bcc/Cc supplémentaires, forge des en-têtes, ou scinde dans le corps du message, le c...
- **[Abus multipart MIME](mime-multipart-abuse.md)** : contrôler la frontière MIME ou les en-têtes de partie pour ajouter des pièces jointes, faire passer en fraude un corps alternatif qu'un filtre n'inspecte jamais, ou désynchroniser la façon dont le MTA et le m...
- **[Injection de segment SMTP](smtp-segment-injection.md)** : là où une application parle du SMTP brut, injecter dans MAIL FROM, RCPT TO ou DATA pour ajouter des destinataires, réécrire l'enveloppe, ou terminer DATA prématurément avec un point isolé et...

## Outils

- **swaks** : couteau suisse pour SMTP, scriptant des enveloppes de mail forgées.
- **Burp Repeater** : injecter des séquences CRLF dans les champs de formulaire de mail.
- **curl** : envoyer des payloads CRLF forgés dans les champs de composition de mail.
- Test manuel avec Burp Repeater et des payloads forgés.

## Références

- [OWASP: Testing for IMAP SMTP Injection (WSTG)](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/07-Input_Validation_Testing/05-Testing_for_IMAP_SMTP_Injection)
- [PayloadsAllTheThings: CRLF Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/CRLF%20Injection)
