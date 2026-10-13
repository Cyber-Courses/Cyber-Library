---
title: "Injection HQL : Hibernate Query Language construit à partir d'une entrée non fiable"
description: Concaténer l'entrée utilisateur dans le HQL/JPQL de createQuery() contourne le binding de paramètres d'Hibernate, exposant les données d'entités via la logique booléenne et des sous-requêtes corrélées sur les entités mappées.
keywords:
  - Hibernate
  - injection HQL
  - injection JPQL
  - createQuery()
  - binding de paramètres
  - injection SQL
---

# Injection HQL

Le Hibernate Query Language (HQL), et son équivalent JPA le JPQL, est un langage de requête orienté objet qu'Hibernate traduit en SQL. Il prend en charge les **paramètres** nommés/positionnels, mais quand le code applicatif concatène l'entrée utilisateur dans la chaîne de requête passée à `createQuery()`, ces protections sont ignorées et la requête devient injectable.

## Schéma vulnérable

```java
// username from the request
String hql = "FROM User WHERE username = '" + username + "'";
Query<User> q = session.createQuery(hql, User.class);
```

La forme sûre lie un paramètre (`WHERE username = :u` avec `setParameter("u", username)`) ; la forme injectable interpole la valeur dans la chaîne.

## Exploitation

Le HQL opère sur des **entités et leurs champs**, non sur des tables brutes, ce qui façonne les charges utiles. L'évasion booléenne fonctionne comme en SQL :

```
' OR '1'='1
' OR 1=1 --
```

La récupération par ensembles dépend de la version d'Hibernate. Hibernate ORM 6.0 et ultérieur ajoutent `union`/`union all` (et `intersect`/`except`) au HQL (introduits avec le Semantic Query Model), donc sur une pile actuelle un pivot de style UNION est une surface d'attaque valide. Les opérations d'ensemble HQL exigent des éléments de sélection **compatibles en type** sur les deux branches, donc la requête hôte doit projeter un scalaire correspondant à celui injecté (ici la requête est supposée sélectionner `u.username`, unie à `u.password`, toutes deux des chaînes) :

```
xyz' UNION SELECT u.password FROM User u WHERE '1'='1
```

Les anciennes versions d'Hibernate et le JPQL portable n'ont pas d'opérateurs d'ensemble, donc là la récupération passe par des **sous-requêtes et de la navigation d'entités** :

```
' OR username='admin' --
' OR (SELECT u2.password FROM User u2 WHERE u2.username='admin') LIKE 'a%' --
```

Comme le HQL résout l'accès aux champs d'après le mapping, une sous-requête corrélée ou un chemin d'association lit des propriétés sensibles d'entités liées que la requête n'avait jamais prévu d'exposer (par exemple `user.credentials.passwordHash`). Lorsque le HQL ne peut toujours pas exprimer une construction nécessaire, pivotez vers le point d'injection de requête native (voir [Injection SQL native](native-sql-injection.md)) si l'application en expose aussi un.

L'extraction en aveugle utilise la même inférence booléenne/par sous-chaîne que l'injection SQL, élaborez des conditions sur `SUBSTRING(u.password,1,1)='a'` et observez les différences de résultats. Hibernate s'exécute par-dessous sur n'importe quel backend JDBC (PostgreSQL, MySQL, Oracle, SQL Server), donc le dialecte SQL généré suit la base de données configurée.

## Outils

- **Burp Repeater** : élaborer des évasions HQL booléennes et par sous-requête et lire les réponses.
- **Burp Intruder** : automatiser l'inférence en aveugle par sous-chaîne sur les champs d'entités.
- **sqlmap** : automatiser l'extraction après un pivot vers un point d'injection SQL natif.

## Références

- [Hibernate ORM : HQL/JPQL](https://docs.jboss.org/hibernate/orm/current/userguide/html_single/Hibernate_User_Guide.html#hql)
- [OWASP : Testing for ORM Injection](https://owasp.org/www-project-web-security-testing-guide/)
- [PayloadsAllTheThings : SQL Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/SQL%20Injection)
