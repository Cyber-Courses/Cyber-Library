---
title: "Abus multipart MIME : faire passer des parties en fraude devant un composeur de mail"
description: "Contrôler la frontière MIME ou les en-têtes de partie pour ajouter des pièces jointes, faire passer en fraude un corps alternatif qu'un filtre n'inspecte jamais, ou désynchroniser la façon dont le MTA et le client de messagerie analysent le message."
keywords:
  - injection MIME
  - frontière multipart
  - smuggling de pièce jointe
  - corps alternatif
  - injection de Content-Type
  - confusion de parseur de mail
---

# Abus multipart MIME

Un email multipart est un conteneur structuré : un en-tête de premier niveau `Content-Type: multipart/...; boundary="X"` déclare une chaîne de délimiteur, et chaque partie est introduite par `--X` suivi de ses propres en-têtes et contenu, avec `--X--` fermant l'ensemble. Lorsqu'une application laisse de l'entrée non fiable atteindre le token de frontière ou les en-têtes d'une partie, l'attaquant peut forger des séparateurs de partie, ajouter ses propres parties, ou faire diverger deux parseurs sur l'endroit où les parties commencent et finissent.

## Où l'entrée atteint la structure

Deux sinks dominent. Premièrement, l'injection d'en-tête (CRLF dans un champ d'en-tête) qui atteint la ligne `Content-Type` laisse l'attaquant déclarer ou redéfinir la frontière. Deuxièmement, les templates qui interpolent des données utilisateur dans un corps multipart construit à la main placent les octets de l'attaquant directement entre les séparateurs de partie. Dans les deux cas, l'attaquant a besoin de la chaîne de frontière, qui est souvent prévisible : statique dans le source, dérivée d'un timestamp, ou renvoyée dans une réponse antérieure.

## Ajouter une partie ou une pièce jointe

Une fois la frontière connue, injecter une séquence `CRLF` plus un nouveau bloc `--boundary` ajoute une partie que l'application n'a jamais prévue. Livrer une partie HTML :

```
x@evil.test%0d%0aContent-Type: multipart/mixed; boundary="AaB03x"%0d%0a%0d%0a--AaB03x%0d%0aContent-Type: text/html%0d%0a%0d%0a<h1>Phish</h1>%0d%0a--AaB03x--
```

La même forme porte une pièce jointe en posant `Content-Disposition: attachment; filename="invoice.html"`, en déclarant `Content-Transfer-Encoding: base64`, et un corps base64 (l'en-tête `Content-Transfer-Encoding` est requis, sans quoi le client traite le texte base64 comme du contenu littéral et ne le décode jamais en fichier). Cela transforme un simple mailer de notification en canal de livraison de malware ou de document de phishing envoyé depuis le domaine de confiance.

## Faire passer un corps alternatif en fraude devant un filtre

`multipart/alternative` dit au client de rendre la dernière partie qu'il comprend, généralement celle en HTML, tandis que les scanners de contenu n'inspectent souvent que la première partie `text/plain`. L'attaquant expédie une partie en texte propre et une partie HTML malveillante sous la même frontière :

```
--AaB03x
Content-Type: text/plain

Your account summary is attached.
--AaB03x
Content-Type: text/html

<a href="https://evil.test">Review your account</a>
--AaB03x--
```

Le filtre valide le texte inoffensif ; le client de la victime montre le HTML. Le lecteur et le scanner ne voient jamais le même message.

## Désynchronisation de parseur

Les MTA, les passerelles et les clients implémentent chacun l'appariement de frontière légèrement différemment : gestion des espaces de tête/de fin sur le délimiteur, d'une frontière quasi identique (`--AaB03x ` avec un espace de fin), d'un en-tête `Content-Type` dupliqué, ou d'une frontière qui apparaît aussi à l'intérieur du contenu d'une partie. Injecter une frontière qu'un hop traite comme un séparateur et que le suivant traite comme du texte littéral fait analyser des structures différentes aux mêmes octets par la passerelle et le client, de sorte qu'une partie visible pour l'un est avalée par l'autre. Faire entrer en collision la frontière déclarée avec une chaîne qui apparaît naturellement dans le corps est une façon fiable de scinder le message à un point inattendu.

## Faire que ça passe

- La chaîne de frontière doit être connue ou contrôlable ; récoltez-la depuis le source, des en-têtes fuités, ou un générateur prévisible avant de construire des parties.
- Les séparateurs de partie et les en-têtes de partie ont tous deux besoin de vrais `CRLF` ; sur le fil depuis une requête web, c'est `%0d%0a`, et la ligne vide avant le contenu d'une partie est un double `%0d%0a%0d%0a`.
- Une partie doit se terminer par `CRLF` immédiatement avant le `--boundary` suivant, et le délimiteur final a besoin de son `--` de fin.

## Outils

- **swaks** : couteau suisse pour SMTP, scriptant des messages multipart forgés.
- **Burp Repeater** : injecter des payloads de frontière et d'en-tête de partie dans les champs de mail.
- **curl** : envoyer des payloads CRLF et de frontière forgés dans les champs de composition de mail.
- Test manuel avec Burp Repeater et des payloads forgés.

## Références

- [OWASP: Testing for IMAP SMTP Injection (WSTG)](https://owasp.org/www-project-web-security-testing-guide/latest/4-Web_Application_Security_Testing/07-Input_Validation_Testing/05-Testing_for_IMAP_SMTP_Injection)
- [PayloadsAllTheThings: CRLF Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/CRLF%20Injection)
