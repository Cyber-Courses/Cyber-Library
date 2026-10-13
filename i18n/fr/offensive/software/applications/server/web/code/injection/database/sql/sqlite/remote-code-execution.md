---
order: 6
title: "Exécution de code à distance via l'injection SQLite"
description: "Atteindre l'exécution de code depuis une injection SQLite via load_extension chargeant une bibliothèque partagée, et le web shell ATTACH écrit dans un répertoire servi."
keywords:
  - load_extension
  - RCE SQLite
  - bibliothèque partagée
  - webshell ATTACH
  - enable_load_extension
---

# Exécution de code à distance

SQLite offre deux voies vers l'exécution de code depuis une injection, toutes deux avec des préconditions claires.

`load_extension(path)` charge une bibliothèque partagée et appelle son point d'entrée, qui exécute du code natif. C'est la voie directe, mais elle est désactivée par défaut à deux niveaux : de nombreux builds compilent avec les extensions chargeables entièrement désactivées, et même quand elles sont compilées, le chargement d'extensions doit être activé par l'application hôte via l'API C (`sqlite3_enable_load_extension()` ou `sqlite3_db_config()`). Il n'existe pas de pragma SQL qui l'active, donc l'injection ne peut pas déverrouiller cette voie elle-même ; elle ne fonctionne que là où l'application a déjà activé le chargement d'extensions. Là où c'est le cas, une bibliothèque malveillante atteinte via un chemin inscriptible ou fourni par l'attaquant donne l'exécution :

```sql
' ; SELECT load_extension('/tmp/evil')-- 
```

Le point d'entrée (nommé d'après le fichier, par exemple `sqlite3_evil_init`) s'exécute au chargement de la bibliothèque, donc le simple fait de la charger exécute le code de l'attaquant en tant que processus de l'application.

La seconde voie n'a pas besoin de `load_extension` du tout : elle chaîne la primitive d'écriture de fichier avec la couche web. Écrire un payload PHP (ou autre côté serveur) dans la racine du document avec `ATTACH DATABASE`, comme montré sur la page de manipulation de fichiers, produit un script que le serveur web exécute à la requête, donnant l'exécution de commandes via HTTP plutôt que via SQLite lui-même. C'est souvent la voie la plus pratique, puisque `load_extension` est généralement désactivée, tandis qu'une racine web inscriptible plus une exécution empilée est une configuration courante dans les petites applications qui embarquent SQLite.

## Outils

- **sqlmap** : chaîne les écritures empilées vers un shell en racine web là où l'API l'autorise.
- **sqlite3** : la CLI du moteur pour construire la bibliothèque `load_extension` et tester les écritures `ATTACH`.
- Tests manuels avec Burp Repeater et le client sqlite3.

## Références

- Documentation SQLite : load_extension, enable_load_extension, extensions chargeables au runtime
- OWASP Testing Guide: Testing for SQL Injection
