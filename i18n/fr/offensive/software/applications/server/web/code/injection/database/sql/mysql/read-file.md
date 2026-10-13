---
order: 7
title: "Lecture des fichiers du serveur via une injection MySQL avec LOAD_FILE"
description: "Utiliser LOAD_FILE pour lire les fichiers de l'hôte de la base de données via une injection MySQL, et les conditions liées au privilège FILE et à secure_file_priv qui la conditionnent."
keywords:
  - LOAD_FILE
  - lecture de fichier
  - secure_file_priv
  - privilège FILE
  - divulgation de fichiers MySQL
---

# Lecture de fichier

`LOAD_FILE(path)` renvoie le contenu d'un fichier du serveur de base de données sous forme de chaîne, ce qui en fait une primitive de lecture directe sur tout ce que le compte MySQL peut ouvrir. On l'utilise couramment pour récupérer des fichiers de configuration, du code source et des identifiants depuis l'hôte.

Placez-la dans une colonne reflétée :

```sql
' UNION SELECT NULL,LOAD_FILE('/etc/passwd'),NULL-- 
```

Trois conditions doivent toutes être réunies, sinon `LOAD_FILE` renvoie silencieusement `NULL` :

- Le compte possède le privilège `FILE`.
- `secure_file_priv` autorise le chemin. Vérifiez-le avec `SELECT @@secure_file_priv` : une valeur vide signifie aucune restriction, un répertoire signifie que les lectures sont confinées à ce répertoire, et `NULL` signifie que les opérations sur les fichiers sont entièrement désactivées.
- Le fichier du système d'exploitation est lisible par l'utilisateur du processus MySQL et sa taille est inférieure à `max_allowed_packet`.

Comme une lecture bloquée et un fichier manquant produisent tous deux `NULL`, confirmez d'abord le privilège en lisant un fichier connu pour exister et être lisible par tous. Les fichiers binaires ou non UTF-8 ressortent déformés à travers une colonne de texte, enveloppez-les donc dans `HEX(LOAD_FILE(...))` et décodez côté client.

## Outils

- **sqlmap** : lit les fichiers du serveur via l'injection avec `--file-read` en utilisant `LOAD_FILE`.
- **mysql** : client officiel pour confirmer `secure_file_priv` et tester directement `LOAD_FILE`.

## Références

- MySQL Reference Manual : LOAD_FILE, `secure_file_priv`, FILE privilege
- OWASP Testing Guide : Testing for SQL Injection
