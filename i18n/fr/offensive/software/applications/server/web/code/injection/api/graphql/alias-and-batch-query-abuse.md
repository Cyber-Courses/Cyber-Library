---
order: 3
title: "Abus des alias et des requêtes groupées GraphQL"
description: "Les alias, les opérations groupées et les champs dupliqués imbriqués permettent à une seule requête d'exécuter le même travail de nombreuses fois, amplifiant le coût et forçant des valeurs par force brute au-delà de limites qui n'étaient appliquées que par requête HTTP."
keywords:
  - alias GraphQL
  - requêtes groupées
  - coût de requête
  - contournement de limitation de débit
  - profondeur de requête
---

# Abus des alias et des requêtes groupées

Une requête GraphQL peut demander le même champ plusieurs fois sous différents alias, envoyer plusieurs opérations dans une seule requête HTTP et imbriquer des champs en profondeur. Chaque répétition réexécute le resolver. Lorsqu'une limite, un plafond de débit, un compteur de tentatives de connexion ou un budget de coût est appliqué par requête HTTP plutôt que par exécution de resolver, ces fonctionnalités regroupent de nombreuses opérations logiques en une seule requête et passent en dessous.

## Les alias pour multiplier une requête

Les alias permettent au même champ d'apparaître à plusieurs reprises dans une seule requête, chacun avec des arguments différents. Une limitation de débit par requête ne voit qu'une seule requête ; le serveur exécute le resolver une fois par alias :

```graphql
mutation {
  a: login(user: "admin", password: "guess1") { token }
  b: login(user: "admin", password: "guess2") { token }
  c: login(user: "admin", password: "guess3") { token }
  # ... hundreds more aliases in one request
}
```

Un compteur qui s'incrémente une fois par requête HTTP ne se déclenche jamais, de sorte qu'un credential stuffing ou une attaque par force brute sur un code à usage unique qui serait limité sur de nombreuses requêtes s'exécute en une poignée. La même astuce lit de nombreux objets à la fois en utilisant des alias sur un champ par identifiant sur une plage d'identifiants.

## Grouper les opérations

Lorsque le serveur accepte un tableau d'opérations dans une seule requête, le tableau est l'amplificateur à la place des alias, avec le même effet sur n'importe quel contrôle par requête :

```json
[
  {"query": "mutation { redeem(code: \"0001\") { ok } }"},
  {"query": "mutation { redeem(code: \"0002\") { ok } }"},
  {"query": "mutation { redeem(code: \"0003\") { ok } }"}
]
```

## Amplification du coût et de la profondeur

L'imbrication et la répétition font aussi grimper le coût brut. Une requête qui suit une relation cyclique, ou répète un champ coûteux sur de nombreux alias, force un travail disproportionné de la base de données et du CPU à partir d'une petite requête, ce qui constitue un levier de déni de service là où aucune analyse de coût de requête ne borne l'exécution :

```graphql
query {
  users {
    posts { author { posts { author { posts { id } } } } }
  }
}
```

## Confirmer l'amplification

Envoyez un champ bénin sous un nombre croissant d'alias et observez le temps de réponse ainsi que tout compteur de limite : si le serveur les traite tous et que le contrôle par requête ne s'enregistre qu'une fois, l'amplification est réelle. Pour la force brute, la preuve est un flux limité (connexion, coupon, code à usage unique) qui réussit en bien moins de requêtes HTTP que la limite ne devrait l'autoriser. Les défenses qui arrêteraient cela, comptabilité du coût par resolver, plafonds d'alias et de profondeur, résident dans le code et sont souvent absentes, ce qui explique pourquoi l'hypothèse du par requête vaut la peine d'être testée en premier.

## Outils

- **graphql-cop** : audit d'un endpoint pour l'amplification par groupage et par alias.
- **InQL** : cartographie des champs interrogeables et génération d'opérations dans Burp.
- **Burp Repeater et Intruder** : élaboration de requêtes avec alias et groupées, et mesure de l'amplification.

## Références

- [OWASP: GraphQL Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/GraphQL_Cheat_Sheet.html)
- [OWASP API Security Top 10: Unrestricted Resource Consumption](https://owasp.org/API-Security/editions/2023/en/0xa4-unrestricted-resource-consumption/)
