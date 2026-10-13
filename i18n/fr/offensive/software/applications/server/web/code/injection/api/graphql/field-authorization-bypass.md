---
order: 1
title: "Contournement de l'autorisation au niveau des champs GraphQL"
description: "Lorsque l'autorisation est appliquée au niveau de l'opération ou à la frontière HTTP plutôt que sur les resolvers de champs et d'arêtes individuels, un appelant lit ou modifie des objets hors de son périmètre de tenancy en naviguant dans le graphe jusqu'à eux."
keywords:
  - BOLA GraphQL
  - autorisation au niveau des champs
  - autorisation de resolver
  - IDOR GraphQL
  - broken object level authorization
---

# Contournement de l'autorisation au niveau des champs

GraphQL résout une requête champ par champ, et chaque resolver de champ constitue sa propre frontière de confiance. Lorsque le contrôle d'accès est appliqué une seule fois, à la couche HTTP ou sur l'opération de premier niveau, mais pas sur les resolvers qui récupèrent les objets liés, un appelant atteint des données hors de son périmètre de tenancy en parcourant le graphe jusqu'à elles. Il s'agit d'une broken object-level authorization (BOLA/IDOR) sous forme de graphe, et c'est la faille GraphQL la plus courante et à plus fort impact.

## Naviguer vers des objets non autorisés

Une requête est autorisée à récupérer le propre objet de l'appelant, mais un champ imbriqué expose un voisin. Si le resolver de ce champ imbriqué récupère par ID sans revérifier la propriété, l'appelant le lit :

```graphql
query {
  me {
    id
    organization {
      members { id email role }   # are these resolvers checking the caller's membership?
    }
  }
}
```

La forme directe fournit un identifiant que l'appelant ne devrait pas posséder. Lorsque les ID d'objets sont globaux et devinables ou énumérables (séquentiels, ou exposés ailleurs dans le graphe), demander le nœud d'un autre tenant teste si le resolver autorise par identité ou simplement par la forme de la requête :

```graphql
query {
  node(id: "VXNlcjoxMDQ=") {       # another tenant's user, base64 of User:104
    ... on User { email role }
  }
}
```

## Les mutations sont la même faille, avec des effets de bord

Les failles au niveau des champs sur les mutations sont pires, car le contrôle manquant écrit désormais. Une mutation autorisée à mettre à jour le profil de l'appelant peut accepter un `id` ou une entrée imbriquée ciblant un autre objet :

```graphql
mutation {
  updateUser(id: "104", input: { role: ADMIN }) { id role }
}
```

Si le resolver charge la cible par `id` et applique la modification sans confirmer que l'appelant peut modifier cet utilisateur, la requête élève des privilèges ou altère les données d'un autre tenant.

## Confirmer la faille

Le test fiable utilise deux identités. Capturez une requête qui renvoie le graphe d'objets d'un utilisateur, puis rejouez-la en tant que second utilisateur sans lien, avec les ID du premier utilisateur substitués. Une réponse qui renvoie les champs du premier utilisateur, plutôt qu'une erreur d'autorisation ou null, prouve que le resolver fait confiance à la forme de la requête plutôt qu'à l'identité de l'appelant. L'introspection, là où elle est activée, cartographie les champs et les arêtes qui valent la peine d'être sondés ; là où elle est désactivée, le schéma reste récupérable à partir des messages d'erreur et des bundles clients. Chaque resolver qui récupère par un ID fourni par l'appelant ou navigue sur une arête est un candidat, alors énumérez-les plutôt que de tester uniquement l'opération de premier niveau.

## Outils

- **InQL** : énumération des champs et des arêtes pour sonder les autorisations manquantes dans Burp.
- **clairvoyance** : récupération du schéma lorsque l'introspection est désactivée.
- **Burp Repeater** : rejeu d'une requête capturée sous une seconde identité avec des ID substitués.

## Références

- [OWASP API Security Top 10: Broken Object Level Authorization](https://owasp.org/API-Security/editions/2023/en/0xa1-broken-object-level-authorization/)
- [OWASP: GraphQL Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/GraphQL_Cheat_Sheet.html)
