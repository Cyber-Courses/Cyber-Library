---
order: 2
title: "Abus du transfert d'arguments de resolver GraphQL"
description: "Un resolver qui transmet ses arguments GraphQL directement dans du SQL, une commande, une URL ou une API interne transforme une requête validée par le schéma en injection ordinaire, atteinte via le resolver et masquée par le système de types."
keywords:
  - injection d'argument de resolver
  - injection SQL GraphQL
  - injection via resolver
  - sink de resolver
  - transfert d'arguments
---

# Abus du transfert d'arguments de resolver

Le schéma vérifie le type des arguments d'un resolver, ce qui prouve qu'ils sont bien formés, pas qu'ils sont sûrs à utiliser. Lorsqu'un resolver transfère un argument vers un backend, une requête SQL, une commande système, une URL sortante ou un appel de service interne, sans le valider ni le paramétrer, le graphe devient un vecteur de livraison pour les mêmes classes d'injection que l'on trouve partout ailleurs, atteintes via une requête bien typée qui passe chaque vérification de schéma.

## Argument vers une requête

Un argument `String` que le schéma accepte circule vers une requête concaténée dans le resolver :

```graphql
query {
  products(filter: "name LIKE '%a%'") { id name }
}
```

Si le resolver construit le SQL en concaténant `filter`, l'argument porte un payload que le schéma n'a jamais inspecté, car pour GraphQL il s'agit simplement d'un `String` valide. Les mécanismes de l'injection qui en résulte relèvent du backend (voir le sous-arbre [base de données](../../database/index.md)) ; le point propre à GraphQL est que le système de types donne une fausse assurance, de sorte que les arguments doivent être traités comme non fiables au niveau du resolver même lorsqu'ils sont fortement typés.

## Argument vers une requête sortante

Les arguments qui nomment une ressource, un hôte, un chemin ou un identifiant pour un appel en aval constituent le même danger visant un sink différent. Un resolver qui récupère une URL construite à partir d'un argument est un sink de server-side request forgery atteint via le graphe :

```graphql
query {
  linkPreview(url: "http://169.254.169.254/latest/meta-data/") { title }
}
```

et un resolver qui transmet un argument dans un chemin de service interne atteint ce service avec les privilèges du serveur de graphe.

## Pourquoi cela se cache

L'abus du transfert est facile à manquer car le schéma ressemble à de la validation. Un relecteur voit des arguments typés et suppose la sécurité, tandis que le resolver les interpole discrètement. Les resolvers imbriqués et de relation aggravent le problème : un argument sur un champ imbriqué peut atteindre un backend différent de celui que suggère la requête de premier niveau, de sorte que le sink n'est pas là où la requête semble pointer. Le test consiste à tracer chaque argument depuis la définition du schéma jusqu'au code qui le consomme, et à envoyer des valeurs valides par type porteuses de métacaractères propres au backend, en guettant le comportement en aval plutôt qu'une erreur de schéma.

## Outils

- **InQL** : cartographie des arguments de resolver dignes d'être tracés jusqu'à un sink backend.
- **Burp Repeater** : envoi d'arguments valides par type porteurs de métacaractères backend.
- **sqlmap** : confirmation d'une injection SQL lorsqu'un resolver transfère un argument dans une requête.

## Références

- [OWASP: GraphQL Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/GraphQL_Cheat_Sheet.html)
- [OWASP API Security Top 10](https://owasp.org/API-Security/editions/2023/en/0x11-t10/)
