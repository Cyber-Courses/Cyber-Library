---
title: "Injection SQL native Hibernate : createNativeQuery() / createSQLQuery() avec concaténation"
description: Les API de requête native d'Hibernate exécutent du SQL brut du backend ; concaténer l'entrée utilisateur dans celles-ci donne une injection SQL complète, y compris les requêtes empilées et les fonctions spécifiques au SGBD.
keywords:
  - Hibernate
  - injection SQL native
  - createNativeQuery()
  - createSQLQuery()
  - JDBC
  - injection SQL
---

# Injection SQL native

Hibernate permet aux applications de sortir du HQL et d'exécuter du **SQL natif du backend** via `createNativeQuery()` (JPA) ou l'ancien `createSQLQuery()`. Ceux-ci exécutent la chaîne brute contre la connexion JDBC sous-jacente, donc concaténer l'entrée utilisateur produit une injection SQL complète dans le dialecte propre à la base, plus puissante que l'[injection HQL](hql-injection.md) parce que chaque construction native est disponible.

## Schéma vulnérable

```java
// id from the request
String sql = "SELECT * FROM users WHERE id = " + id;
Query q = session.createNativeQuery(sql);
```

La forme sûre utilise des espaces réservés (`WHERE id = ?1` / `:id` avec `setParameter`) ; la forme injectable interpole directement.

## Exploitation

Les requêtes natives exposent les tables réelles et le jeu de fonctionnalités du SGBD. Contexte numérique :

```
1 OR 1=1
0 UNION SELECT username, password, NULL FROM users
```

Contexte de chaîne :

```
' UNION SELECT username, password FROM users --
```

Selon le backend et les réglages JDBC, les **requêtes empilées** peuvent être disponibles (`; UPDATE ...`), et les primitives spécifiques au SGBD s'appliquent, `pg_sleep()` / `BENCHMARK()` pour l'aveugle temporisé, `CONVERT()`/`CAST()` pour l'extraction par erreur, et des primitives de fichier ou de commande lorsque la base et les privilèges le permettent. Confirmez d'abord le backend (via des chaînes d'erreur, des fonctions de version telles que `@@version` / `version()`), puis choisissez les charges utiles du dialecte correspondant.

Lorsque la requête native mappe les résultats vers une entité via `addEntity()`, l'ordre des colonnes sélectionnées doit correspondre au mapping de l'entité ; avec `createNativeQuery(sql)` renvoyant un `Object[]`, un `UNION` peut projeter des colonnes arbitraires pour une lecture directe.

## Outils

- **sqlmap** : automatiser l'extraction contre le point d'injection SQL brut createNativeQuery.
- **Burp Repeater et Intruder** : délivrer des charges utiles UNION et en aveugle spécifiques au dialecte.

## Références

- [Hibernate ORM : Native SQL queries](https://docs.jboss.org/hibernate/orm/current/userguide/html_single/Hibernate_User_Guide.html#sql)
- [PayloadsAllTheThings : SQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection)
