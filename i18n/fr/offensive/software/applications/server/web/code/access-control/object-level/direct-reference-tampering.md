---
title: "IDOR et BOLA : manipulation de référence directe quand les serveurs sautent l'autorisation par objet"
description: Manipuler les identifiants d'objets dans les chemins, les chaînes de requête ou les corps JSON pour accéder aux enregistrements d'autres utilisateurs lorsque le serveur omet un contrôle d'autorisation par objet.
keywords:
  - IDOR
  - BOLA
  - autorisation au niveau objet
  - référence d'objet directe non sécurisée
---

# Référence directe (IDOR)

## Contexte

Le serveur charge une ligne par un identifiant, un slug ou une clé composite issu de la requête. Le sujet est authentifié, mais le code n'affirme pas que le sujet peut accéder à cette ligne précise. Échanger un identifiant contre un autre en labo avec deux comptes de test est la preuve contrôlée standard. Ce n'est pas « deviner des UUID aléatoires » comme explication complète : le bug est un contrôle manquant, non une question de longueur de clé.

## Théorie

Un `SELECT` sur une table unique par clé primaire sans `owner_id` ni `tenant_id` dans la clause `WHERE`, des interfaces de liste qui filtrent la liste mais pas la route de détail, et des resolvers GraphQL `node(id:)` qui résolvent n'importe quel identifiant global relèvent de la même classe. Les variantes par lot et à ressource imbriquée étendent l'ensemble d'identifiants dans une seule requête ou un seul chemin.

## Pratique

### Test horizontal avec deux comptes

- En tant qu'utilisateur A, créez ou lisez un objet et notez son identifiant. En tant qu'utilisateur B, appelez la même route de lecture ou d'écriture avec l'identifiant de A dans le chemin ou le corps. Un `200` avec les données de A alors que vous êtes authentifié comme B est un signal BOLA direct en labo.

### Fuzzer le type et l'encodage d'identifiant

- Essayez les voisins entiers `+1`, différents encodages de chaîne du même identifiant, et des champs de tableau parallèles quand l'API accepte `ids[]` ou une liste JSON de références.

## Outils

- **Burp Suite**
- **curl**
- **Autorize** (extension Burp)
