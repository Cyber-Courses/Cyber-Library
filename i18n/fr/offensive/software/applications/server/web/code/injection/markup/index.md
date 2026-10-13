---
order: 5
title: "Injection de balisage"
description: "Injection dans les langages de balisage et de traitement XML côté serveur : Server-Side Includes, parsing XML, requêtes XPath/XQuery/XSLT et abus d'entités XXE."
keywords:
  - injection de balisage
  - injection SSI
  - injection XML
  - injection XPath
  - XXE
---

# Balisage

Les langages de traitement de balisage se placent entre la requête et la réponse, et beaucoup d'entre eux exécutent des directives qui résident à l'intérieur d'un texte par ailleurs inerte. Lorsque l'entrée utilisateur atteint un document que le serveur interprète comme du balisage plutôt que comme des données, le parseur agit sur des directives fournies par l'attaquant. Ce sous-arbre regroupe les familles d'injection qui partagent cette cause racine.

Les **Server-Side Includes (SSI)** permettent à un serveur web d'étendre les directives `<!--#...-->` à l'intérieur d'une page avant de l'envoyer, de sorte que des directives injectées peuvent exécuter des commandes, renvoyer des variables serveur et inclure des fichiers. Le **traitement XML** couvre les étapes de parsing, d'interrogation et de transformation d'un pipeline XML : un parseur qui résout les entités externes expose le XXE, tandis qu'une requête construite par concaténation de chaînes expose l'injection dans **XPath**, **XQuery** et **XSLT**. Dans chaque cas la primitive est la même, une frontière entre donnée et instruction que l'entrée franchit, et les charges utiles ne diffèrent que par la syntaxe de directive que chaque processeur comprend.

Cette section couvre l'injection de directives SSI et la famille du traitement XML, y compris l'injection de requêtes XPath et les attaques au niveau des entités qui découlent de parseurs permissifs.

## Sous-thèmes

- **[SSI](ssi/index.md)** : les Server-Side Includes étendent les directives <!--#...--> avant qu'une page ne soit servie ; l'entrée utilisateur qui atteint une page interprétée en SSI exécute des directives permettant l'exécution de commandes,...
- **[Traitement XML](xml-processing/index.md)** : injection à travers tout le pipeline XML : parsing, interrogation avec XPath/XQuery et transformation avec XSLT, ainsi que l'abus d'entités externes que les parseurs permissifs autorisent...

## Outils

- **Burp Suite** : injection et itération de charges utiles SSI et XML via Repeater et Intruder.
- **XXEinjector** : automatisation de l'exploitation XXE dans la branche du traitement XML.
- **xcat** : automatisation de l'extraction de données XPath.
- **Burp Collaborator** : confirmation des injections aveugles et hors bande.

## Références

- OWASP : Server-Side Includes (SSI) Injection
- PortSwigger Web Security Academy : XML external entity (XXE) injection
