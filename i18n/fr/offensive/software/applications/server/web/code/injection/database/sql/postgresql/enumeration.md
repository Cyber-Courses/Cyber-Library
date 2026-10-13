---
order: 11
title: "Prise d'empreinte et énumération dans l'injection PostgreSQL"
description: "S'orienter dans une injection PostgreSQL : confirmer le moteur, lire la version et le contexte courant, et vérifier si le rôle est superutilisateur."
keywords:
  - prise d'empreinte PostgreSQL
  - fonction version
  - current_user current_database
  - is_superuser
  - énumération de rôle
---

# Énumération

Confirmez que le moteur est PostgreSQL et lisez le contexte du rôle avant de choisir une technique, car les chemins fichier, hors bande et commande dépendent tous de la question de savoir si le rôle est privilégié.

`version()` identifie PostgreSQL sans ambiguïté (sa sortie commence par le littéral `PostgreSQL`, ce qui le distingue aussi de MySQL dont la version commence par un chiffre), et les fonctions d'identité donnent le rôle et la base de données courants :

```sql
' UNION SELECT version(),current_user,current_database()-- 
```

`current_user` est le rôle utilisé pour les contrôles de permission, `session_user` est le rôle qui s'est connecté, et `current_database()` nomme la base de données. La version numérique pour les comparaisons est `current_setting('server_version_num')` (par exemple `160001`), ce qui évite de parser la bannière textuelle.

Le fait le plus important est de savoir si le rôle est superutilisateur, puisque cela décide de la lecture de fichiers, de l'écriture de fichiers et de l'exécution de commandes :

```sql
' UNION SELECT current_setting('is_superuser'),NULL,NULL-- 
```

Une valeur `on` signifie que l'ensemble des primitives fichier et programme est disponible. Sur la version 11 et ultérieure, un non-superutilisateur peut tout de même en atteindre certaines s'il appartient aux rôles `pg_read_server_files`, `pg_write_server_files` ou `pg_execute_server_program`, que la page des privilèges énumère. Une fois la version et le rôle connus, les techniques restantes s'appliquent avec les bonnes attentes.

## Outils

- **sqlmap** : prend l'empreinte du moteur et lit la version, le rôle et le statut superutilisateur.
- **psql** : client officiel pour confirmer directement `version()`, `current_user` et `is_superuser`.

## Références

- PostgreSQL Documentation: system information functions, `current_setting`
- OWASP Testing Guide: Testing for SQL Injection
