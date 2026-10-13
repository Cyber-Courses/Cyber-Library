---
order: 7
title: "Injection DynamoDB"
description: "Amazon DynamoDB est schemaless et clé-valeur, mais les API qui l'interrogent (instructions PartiQL et expressions de filtre/condition) restent injectables lorsqu'elles sont construites à partir d'une entrée non fiable."
keywords:
  - injection DynamoDB
  - injection PartiQL
  - injection d'expression de filtre
  - expression de condition
  - injection NoSQL
  - AWS
---

# DynamoDB

Amazon DynamoDB n'a pas de moteur SQL au sens classique, donc on le suppose souvent immunisé contre l'injection. Il ne l'est pas. DynamoDB expose deux surfaces de requête que le code applicatif construit couramment à partir d'une entrée utilisateur, et les deux sont injectables lorsque cette entrée est concaténée dans une chaîne d'instruction ou d'expression plutôt que passée en paramètres.

La première surface est **PartiQL**, un langage de requête compatible SQL exécuté via `ExecuteStatement`, `BatchExecuteStatement` et les API de transaction. Le code qui construit une instruction PartiQL par chaînes rouvre toute la classe de l'injection d'instruction : sortir d'une valeur, élargir un `WHERE` avec `OR` et lire des items auxquels l'appelant n'était jamais restreint.

La seconde surface est la famille des **expressions** : `FilterExpression`, `KeyConditionExpression` et `ConditionExpression`, ainsi que les maps `ExpressionAttributeNames`/`ExpressionAttributeValues` qui les alimentent. Lorsqu'une entrée d'attaquant façonne le texte de l'expression ou ces maps, un filtre peut être élargi pour renvoyer plus d'items, ou un garde d'écriture conditionnelle peut être amené à réussir là où il devrait échouer.

## Pages

- **[Contournement d'expression de condition](condition-expression-bypass.md)** : Manipuler les opérandes d'une ConditionExpression pour qu'un garde d'écriture conditionnelle réussisse là où il devrait échouer, permettant la création, la mise à jour, la suppression et l'écrasement non autorisés...
- **[Injection d'expression de filtre](filter-expression-injection.md)** : Lorsqu'une entrée utilisateur façonne une chaîne FilterExpression ou KeyConditionExpression (ou ses ExpressionAttributeNames/Values), un attaquant peut élargir le prédicat et...
- **[Injection PartiQL](partiql-injection.md)** : Les instructions PartiQL exécutées via ExecuteStatement et construites par concaténation de chaînes sont injectables comme du SQL : sortir de la valeur du WHERE, élargir avec OR, et lire...

## Outils

- **AWS CLI** : piloter les API ExecuteStatement, Scan et Query avec une entrée forgée.
- **Burp Suite** : intercepter et altérer les requêtes web adossées à DynamoDB.

## Références

- [AWS DynamoDB: Developer guide](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Introduction.html)
- [PayloadsAllTheThings: NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
