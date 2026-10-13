---
title: "Atteindre le schéma et la version MySQL malgré des noms bloqués"
description: "Alternatives quand information_schema, version() et GROUP_CONCAT sont filtrés dans MySQL, via les tables de statistiques innodb et des fonctions équivalentes."
keywords:
  - alternative à information_schema
  - innodb_table_stats
  - alternative à la fonction version
  - alternative à GROUP_CONCAT
  - évasion de filtre MySQL
---

# Catalogue et fonctions alternatifs

Les filtres mettent souvent en liste noire les chaînes exactes dont un attaquant a besoin : `information_schema`, `version()`, `group_concat`. MySQL expose les mêmes informations sous d'autres noms, de sorte qu'un seul jeton bloqué arrête rarement l'énumération.

Lorsque `information_schema` est bloqué, les tables de statistiques InnoDB listent les tables utilisateur (sur des comptes privilégiés) et sont rarement filtrées :

```sql
' UNION SELECT NULL,GROUP_CONCAT(table_name),NULL FROM mysql.innodb_table_stats WHERE database_name=database()-- 
```

Lorsque `version()` est bloqué, la version du serveur se trouve dans plusieurs variables système :

```sql
' UNION SELECT @@version,NULL,NULL-- 
' UNION SELECT @@global.version,NULL,NULL--  
```

Lorsque `GROUP_CONCAT` est bloqué, `JSON_ARRAYAGG` (MySQL 5.7.22+) agrège les lignes en une seule valeur :

```sql
' UNION SELECT NULL,JSON_ARRAYAGG(table_name),NULL FROM mysql.innodb_table_stats WHERE database_name=database()-- 
```

Les littéraux numériques ont eux aussi des formes compatibles avec les filtres : la notation scientifique (`1e0` pour `1`) et l'hexadécimal (`0x...`) passent devant les filtres qui ne font correspondre que des chiffres simples ou des chaînes entre guillemets. Le principe est que MySQL propose généralement un synonyme, de sorte que l'énumération s'adapte à ce que le filtre bloque plutôt que d'être arrêtée par lui.

## Outils

- **sqlmap** : des scripts tamper atteignent le schéma et la version malgré des noms bloqués.
- **Burp Repeater** : testez à la main les payloads de catalogue et de fonctions alternatifs.

## Références

- MySQL Reference Manual : system variables, `mysql.innodb_table_stats`, JSON_ARRAYAGG
- OWASP Testing Guide : Testing for SQL Injection
