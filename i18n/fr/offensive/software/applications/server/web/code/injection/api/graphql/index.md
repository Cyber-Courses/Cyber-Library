---
order: 1
title: "Abus de GraphQL : confiance des resolvers et forme de requête"
description: "Un endpoint GraphQL exécute une requête façonnée par le client contre des resolvers par champ. La surface d'attaque se situe là où réside l'autorisation, dans la façon dont les arguments des resolvers atteignent les backends, et dans la façon dont les alias et le groupage amplifient une seule requête."
keywords:
  - injection GraphQL
  - autorisation de resolver
  - arguments GraphQL
  - abus d'alias
  - autorisation au niveau des champs
---

# GraphQL

GraphQL expose un seul endpoint qui exécute un arbre de requête arbitraire, façonné par le client. Le serveur parcourt la requête champ par champ, appelant un resolver pour chacun, et le client décide quels champs demander, dans quelle imbrication, sous quels alias et combien à la fois. Cette flexibilité constitue la surface d'attaque : l'autorisation et la validation qui présupposent un ensemble fixe de routes REST ne se transposent pas proprement sur un graphe où l'appelant compose l'opération.

## Où cela tourne mal

Les pages ici suivent les trois endroits où un serveur GraphQL place mal sa confiance :

- **[Contournement de l'autorisation au niveau des champs](field-authorization-bypass.md)** : contrôles appliqués au niveau de l'opération ou à la frontière HTTP mais pas sur les resolvers de champs et d'arêtes individuels, de sorte qu'un appelant lit ou modifie des objets hors de son périmètre de tenancy à travers le graphe.
- **[Abus du transfert d'arguments de resolver](resolver-argument-forwarding-abuse.md)** : un resolver qui transmet ses arguments directement dans du SQL, une commande, une URL ou une API interne, une injection de second ordre cachée derrière les vérifications de type du schéma.
- **[Abus des alias et des requêtes groupées](alias-and-batch-query-abuse.md)** : alias, opérations groupées et champs dupliqués imbriqués utilisés pour amplifier le coût ou forcer des valeurs par force brute au-delà de limites qui n'étaient appliquées que par requête HTTP.

## Pourquoi le graphe change le modèle

En REST, une route correspond à un handler avec un seul endroit où vérifier l'accès. En GraphQL, une seule requête peut toucher des dizaines de resolvers sur de nombreux types, et chaque resolver constitue sa propre frontière de confiance. Un schéma qui vérifie le type des arguments prouve qu'ils sont bien formés, pas qu'ils sont autorisés ou sûrs à transmettre. Le test récurrent consiste à exécuter la même requête sous deux identités différentes, et à tracer chaque argument depuis le schéma jusqu'au backend qu'il atteint.

## Outils

- **graphw00f** : empreinte du moteur GraphQL derrière un endpoint.
- **InQL** : introspection de schéma et génération de requêtes dans Burp.
- **clairvoyance** : reconstruction d'un schéma lorsque l'introspection est désactivée.
- **graphql-cop** : audit de sécurité rapide pour le groupage, l'introspection et les suggestions de champs.

## Références

- [OWASP: GraphQL Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/GraphQL_Cheat_Sheet.html)
- [OWASP API Security Top 10](https://owasp.org/API-Security/editions/2023/en/0x11-t10/)
