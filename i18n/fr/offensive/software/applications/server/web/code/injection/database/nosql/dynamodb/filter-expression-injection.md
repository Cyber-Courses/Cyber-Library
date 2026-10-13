---
order: 2
title: "Injection d'expression de filtre DynamoDB : élargir les ensembles de résultats de Query et Scan"
description: "Lorsqu'une entrée utilisateur façonne une chaîne FilterExpression ou KeyConditionExpression (ou ses ExpressionAttributeNames/Values), un attaquant peut élargir le prédicat et renvoyer des items hors de la portée prévue."
keywords:
  - injection d'expression de filtre
  - DynamoDB
  - FilterExpression
  - KeyConditionExpression
  - ExpressionAttributeValues
  - Query Scan
  - injection NoSQL
---

# Injection d'expression de filtre

`Query` et `Scan` restreignent leurs résultats avec une `FilterExpression`, et `Query` sélectionne la plage de partition/tri avec une `KeyConditionExpression`. Les deux sont des chaînes analysées par DynamoDB, et les deux référencent les noms d'attributs via des placeholders préfixés par `#` résolus depuis `ExpressionAttributeNames` et les valeurs via des placeholders préfixés par `:` résolus depuis `ExpressionAttributeValues`. L'injection survient lorsqu'une entrée d'attaquant est concaténée dans le texte de l'expression lui-même, ou lorsque l'attaquant contrôle les **clés** des maps nom/valeur d'attributs de sorte que les placeholders se résolvent en attributs ou opérateurs que le développeur n'avait pas prévus.

## Schémas vulnérables

Expression construite par chaînes, le point d'injection évident. Les chaînes d'expression ne prennent pas de littéraux en ligne comme le fait PartiQL : les valeurs doivent être des `:`-placeholders, donc le schéma injectable concatène une **clause ou un opérateur** (pas une valeur) dans le texte de l'expression :

```python
# account_id is bound safely with :aid, but extra_filter is attacker text joined in
expr = f"account_id = :aid AND {extra_filter}"
resp = table.scan(
    FilterExpression=expr,
    ExpressionAttributeValues={":aid": {"S": account_id}},
)
```

Tout ce que `extra_filter` contient devient partie intégrante du prédicat analysé.

Maps nom/valeur d'attributs pilotées par la structure de la requête, le point d'injection plus subtil :

```python
# filters is a dict taken straight from JSON query params
names = {f"#{k}": k for k in filters}
values = {f":{k}": {"S": v} for k, v in filters.items()}
expr = " AND ".join(f"#{k} = :{k}" for k in filters)
table.scan(FilterExpression=expr,
           ExpressionAttributeNames=names,
           ExpressionAttributeValues=values)
```

Ici le client choisit quels attributs sont comparés et peut abandonner entièrement le garde de locataire, ou soumettre des clés forgées.

## Exploitation

**Injecter `OR` pour déjouer le garde.** Il n'y a pas d'évasion de littéral entre guillemets (les valeurs sont des placeholders), donc la primitive consiste à ajouter une clause jointe par `OR` à une tautologie de fonction. Injecté dans `extra_filter` ci-dessus :

```
attribute_exists(account_id) OR attribute_exists(account_id)
```

rend l'expression `account_id = :aid AND attribute_exists(account_id) OR attribute_exists(account_id)`. Comme `AND` lie plus fortement que `OR`, cela s'évalue comme `(account_id = :aid AND ...) OR attribute_exists(account_id)`, vrai pour chaque item portant cet attribut, donc le `Scan` renvoie toute la table.

**`attribute_exists` / `attribute_not_exists` comme tautologies.** Ces fonctions sont l'équivalent de `1=1` dans le langage d'expression. `attribute_exists(<clé de partition>)` est vrai pour chaque item ; `attribute_not_exists(<tout attribut toujours présent>)` est fiablement faux et utile pour les tests négatifs et l'inférence booléenne.

**Élargir une condition de clé.** Sur `Query`, la `KeyConditionExpression` contrôle quelle partition est lue. Injecter une comparaison plus large ou un `begins_with` supplémentaire sur la clé de tri étend la plage renvoyée :

```
pk = :pk AND begins_with(sk, :empty)
```

avec `:empty` fixé à `""` correspond à chaque clé de tri sous la partition.

**Permutation d'opérateur via les maps.** Lorsque seules les valeurs sont paramétrées mais que l'**opérateur** est choisi à partir d'une entrée, retournez une égalité en inégalité ou un `>` pour élargir les correspondances :

```
#attr <> :v     # instead of  #attr = :v  → returns everything except one value
```

**Abandonner le filtre de locataire.** Dans le point d'injection piloté par map, omettez simplement la clé `account_id`/locataire de l'objet de filtre soumis. Si le backend construit toute l'expression à partir du dict du client, rien ne restreint le scan aux données de l'appelant, et la réponse contient des items inter-locataires.

**Inférence en aveugle.** Lorsque la réponse ne signale que correspondance-ou-pas-de-correspondance, enchaînez `attribute_exists`/`contains`/`begins_with` contre des noms d'attributs devinés et des préfixes de valeur pour confirmer des champs cachés et récupérer leur contenu un caractère à la fois.

Notez que la `FilterExpression` est appliquée **après** que les items sont lus et comptés, donc un filtre élargi consomme et renvoie quand même tout ce que la condition de clé a sélectionné, ce qui en fait un chemin efficace vers la divulgation en masse via des appels `Scan`/`Query` paginés répétés.

## Outils

- **AWS CLI** : exécuter Scan et Query avec des payloads FilterExpression élargis contre la table.
- **Burp Repeater** : injecter des clauses OR et des permutations d'opérateur via la requête.

## Références

- [AWS DynamoDB: Filter expressions for Scan](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Scan.html#Scan.FilterExpression)
- [AWS DynamoDB: Condition and filter expression reference](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Expressions.OperatorsAndFunctions.html)
- [AWS DynamoDB: Expression attribute names and values](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/Expressions.ExpressionAttributeNames.html)
