---
title: "Injection de l'API Criteria d'Hibernate : sqlRestriction et noms de propriété dynamiques"
description: L'API Criteria est sûre côté paramètre pour les valeurs, mais Restrictions.sqlRestriction() incorpore du SQL brut et les noms de propriété contrôlés par l'utilisateur atteignent des colonnes imprévues.
keywords:
  - Hibernate
  - API Criteria
  - sqlRestriction
  - Restrictions
  - injection de propriété
  - injection SQL
---

# Injection de l'API Criteria

L'API Criteria d'Hibernate construit les requêtes de manière programmatique et lie les **valeurs** sous forme de paramètres, donc l'appel habituel `Restrictions.eq("name", userValue)` n'est pas injectable par la valeur. Deux autres voies en font un point d'injection : l'échappatoire SQL brut `Restrictions.sqlRestriction()`, et les **noms de propriété ou d'alias** contrôlés par l'utilisateur passés dans la construction des critères.

## Schémas vulnérables

**SQL brut via `sqlRestriction`**, le fragment est inséré dans le SQL généré :

```java
criteria.add(Restrictions.sqlRestriction("name = '" + name + "'"));
```

**Nom de propriété/colonne contrôlé par l'utilisateur**, non paramétrable, donc souvent concaténé :

```java
String sortCol = request.getParameter("sort");
criteria.addOrder(Order.asc(sortCol));              // column context
criteria.add(Restrictions.eq(userField, value));    // field chosen by user
```

## Exploitation

`sqlRestriction` est du SQL brut, injectez donc comme dans un fragment `WHERE` :

```
x' OR '1'='1
x') UNION SELECT password FROM users --
x' OR (SELECT SUBSTRING(password,1,1) FROM users WHERE username='admin')='a
```

`sqlRestriction` prend en charge les espaces réservés `?` pour les valeurs ; lorsqu'ils sont ignorés au profit de la concaténation, le fragment est entièrement contrôlé par l'attaquant.

Pour les contextes de **propriété/colonne** (`addOrder`, `eq` dynamique), il n'y a pas de chaîne d'où s'échapper, abusez-en comme d'une injection de nom de colonne : orientez le tri pour révéler des oracles d'ordonnancement, ou atteignez des chemins d'association/colonnes que la requête n'avait jamais prévu d'exposer. Le code moderne utilisant le **Criteria** JPA (`CriteriaBuilder`) est plus sûr pour les valeurs mais peut toujours concaténer des noms de colonne ou se rabattre sur des fragments bruts de style `sqlRestriction`.

## Outils

- **sqlmap** : exploiter le SQL brut inséré via Restrictions.sqlRestriction.
- **Burp Repeater et Intruder** : tester les fragments sqlRestriction et la sélection de nom de propriété.

## Références

- [Hibernate ORM : Legacy Criteria / Restrictions](https://docs.jboss.org/hibernate/orm/current/userguide/html_single/Hibernate_User_Guide.html)
- [PayloadsAllTheThings : SQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection)
