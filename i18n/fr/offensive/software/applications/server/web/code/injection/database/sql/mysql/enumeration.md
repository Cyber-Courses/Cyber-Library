---
order: 12
title: "Fingerprinting et énumération dans l'injection MySQL"
description: "S'orienter dans une injection MySQL : confirmer le moteur, lire la version et le contexte courant, et vérifier les privilèges du compte avant de choisir une technique."
keywords:
  - fingerprinting MySQL
  - détection de version
  - current_user database
  - énumération des privilèges
  - secure_file_priv
---

# Énumération

Avant de choisir une technique, confirmez que le moteur est MySQL et lisez le contexte dans lequel l'injection s'exécute. Le choix entre les voies union, error, file et command dépend de la version et de ce que le compte de base de données est autorisé à faire, donc l'énumération vient en premier.

La version et l'identité proviennent de fonctions et variables système, lues via le canal disponible (une colonne reflétée ici) :

```sql
' UNION SELECT CONCAT_WS(0x3a,@@version,@@version_comment,@@hostname),current_user(),database()-- 
```

`@@version` distingue MySQL de MariaDB (dont la chaîne de version contient `MariaDB`) et donne la version exacte, ce qui décide des comportements conditionnés par la version. `current_user()` est le compte que le serveur a authentifié, `user()` est le login fourni par le client, et `database()` est le schéma courant.

Les privilèges décident jusqu'où va l'injection. Les droits accordés au compte sont dans `information_schema.user_privileges`, et le paramètre d'accès aux fichiers est dans `secure_file_priv` :

```sql
' UNION SELECT GROUP_CONCAT(privilege_type),NULL,NULL FROM information_schema.user_privileges WHERE grantee=CONCAT(0x27,REPLACE(current_user(),0x40,0x274027),0x27)-- 
' UNION SELECT @@secure_file_priv,NULL,NULL-- 
```

Voir `FILE` dans la liste des privilèges, et un `secure_file_priv` vide plutôt qu'un chemin restrictif ou `NULL`, est ce qui vous indique que les voies de lecture de fichier, out-of-band et d'exécution de commandes sont accessibles. Un privilège `SUPER` ou admin signale de même que la voie UDF et les changements de variables serveur sont possibles. Le moteur, la version et les privilèges connus, le reste des techniques s'applique avec les bonnes attentes.

## Outils

- **sqlmap** : identifie le moteur et lit la version, l'utilisateur courant et les privilèges (`--fingerprint`, `--current-user`, `--privileges`).
- **mysql** : client officiel pour confirmer directement `@@version`, `current_user()` et `secure_file_priv`.

## Références

- MySQL Reference Manual: information functions, `information_schema.user_privileges`, `secure_file_priv`
- OWASP Testing Guide: Testing for SQL Injection
