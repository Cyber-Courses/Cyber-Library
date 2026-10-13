---
order: 3
title: "Contournement d'expression de condition DynamoDB : forcer la réussite des écritures conditionnelles"
description: "Manipuler les opérandes d'une ConditionExpression pour qu'un garde d'écriture conditionnelle réussisse là où il devrait échouer, permettant la création, la mise à jour, la suppression et l'écrasement non autorisés d'items."
keywords:
  - contournement d'expression de condition
  - DynamoDB
  - ConditionExpression
  - attribute_not_exists
  - écriture conditionnelle
  - verrouillage optimiste
  - injection NoSQL
---

# Contournement d'expression de condition

Les écritures conditionnelles de DynamoDB (`PutItem`, `UpdateItem`, `DeleteItem` et les formes d'écriture d'`ExecuteStatement`) prennent une `ConditionExpression` qui doit s'évaluer à vrai pour que l'écriture soit validée. Les applications s'appuient sur ces gardes pour la justesse et l'autorisation : `attribute_not_exists(pk)` pour empêcher l'écrasement d'un item existant, un contrôle d'égalité pour imposer la propriété, ou une comparaison de version pour le verrouillage optimiste. Lorsqu'une entrée d'attaquant façonne la chaîne de condition ou les `ExpressionAttributeNames`/`ExpressionAttributeValues` qui l'alimentent, le garde peut être amené à réussir là où il devrait échouer, transformant une écriture protégée en création, mise à jour, suppression ou écrasement non autorisés.

## Schémas vulnérables

Chaîne de garde construite à partir d'une entrée. Comme pour les expressions de filtre, les valeurs doivent être des `:`-placeholders, donc le schéma injectable concatène une **clause ou un opérateur**, pas une valeur entre guillemets :

```python
# guard is attacker text joined into the condition; :o holds the real owner value
cond = f"owner = :o AND {guard}"
table.update_item(
    Key={"id": {"S": item_id}},
    UpdateExpression="SET balance = :b",
    ConditionExpression=cond,
    ExpressionAttributeValues={":o": {"S": owner}, ":b": {"N": new_balance}},
)
```

Opérande de garde choisi à partir d'une entrée (le cas plus subtil) :

```python
# expected_version supplied by the client
table.update_item(
    Key=key,
    UpdateExpression="SET #d = :d",
    ConditionExpression="version = :v",
    ExpressionAttributeValues={":v": {"N": client["expected_version"]},
                               ":d": {"S": client["data"]}},
)
```

## Exploitation

**Neutraliser le garde de propriété avec `OR`.** Avec une clause concaténée dans la condition, ajoutez un `OR` à une tautologie de fonction pour que l'écriture soit validée quel que soit le propriétaire. Injecté dans `guard` ci-dessus :

```
attribute_exists(id) OR attribute_exists(id)
```

rend la condition `owner = :o AND attribute_exists(id) OR attribute_exists(id)`. Comme `AND` lie plus fortement que `OR`, cela est vrai pour chaque item existant, permettant à l'attaquant de mettre à jour des enregistrements dont il n'est pas propriétaire.

**Forcer un garde « créer-si-absent » à réussir et écraser.** Un `PutItem` protégé par `attribute_not_exists(pk)` est censé refuser d'écraser un item existant. Si le texte de la condition est influencé par l'attaquant, remplacez le garde par une tautologie pour que le put écrase l'item de la victime :

```
attribute_not_exists(pk) OR attribute_exists(pk)
```

L'une ou l'autre branche couvre désormais tous les cas, donc l'écriture atterrit toujours et écrase silencieusement.

**Ce qui n'est pas un contournement : l'opérande de version attendue.** Fournir la `version` que vous avez lue précédemment est le protocole de verrouillage optimiste *prévu*, pas une injection. La clé identifie déjà un seul item, DynamoDB ne convertit pas de façon lâche la valeur typée, et une écriture concurrente change la version de sorte que la condition échoue alors. Cela ne devient un problème que si l'application confond séparément la version avec une autorisation. Le vrai contournement dans le schéma contrôlé par l'opérande est l'injection d'opérateur, ci-après.

**Inverser la comparaison via l'injection d'opérateur.** Lorsque seul l'opérande est paramétré mais que le jeton de comparaison provient d'une entrée, retournez le sens du garde :

```
version <> :v      # was  version = :v  → passes for every item except the real one
```

combiné à une clé que l'attaquant contrôle, cela valide de façon fiable des écritures que le garde d'égalité était censé bloquer.

**Supprimer en passant outre le garde.** Les mêmes primitives s'appliquent à `DeleteItem` avec une `ConditionExpression` : une condition tautologique retire l'autorisation d'une suppression, de sorte qu'une clé choisie par l'attaquant peut être détruite même si le garde était destiné à restreindre la suppression au propriétaire. (C'est un contournement d'autorisation sur l'écriture, pas une opération de vidage de corbeille ou de purge en masse.)

**Gardes de transaction.** `TransactWriteItems` attache un `ConditionCheck` et des `ConditionExpression` par item ; les mêmes techniques d'évasion et de tautologie déjouent un contrôle individuel et laissent toute la transaction se valider, ce qui est utile lorsqu'un invariant de solde ou d'inventaire n'est imposé qu'au travers de ce contrôle.

Faites correspondre les noms et types d'attributs à la table cible ; lorsque le résultat de la condition n'est observable qu'à travers la réussite/l'échec de l'écriture, utilisez-le comme oracle booléen pour déduire la présence et les valeurs d'attributs avant de valider l'écriture de contournement finale.

## Outils

- **AWS CLI** : émettre un PutItem, UpdateItem ou DeleteItem avec une ConditionExpression forgée pour confirmer le contournement.
- **Burp Repeater** : altérer la clause de garde ou l'opérande dans la requête de l'application.

## Références

- [AWS DynamoDB: Condition expressions](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Expressions.ConditionExpressions.html)
- [AWS DynamoDB: Comparison operators and functions](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Expressions.OperatorsAndFunctions.html)
- [AWS DynamoDB: Working with items, conditional writes](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/WorkingWithItems.html#WorkingWithItems.ConditionalUpdate)
