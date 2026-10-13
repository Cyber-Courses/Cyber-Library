---
order: 5
title: "Lecture et écriture de fichiers via une injection Oracle"
description: "Accès aux fichiers Oracle depuis une injection : UTL_FILE avec un objet directory, DBMS_LOB pour les lectures, et Java pour des opérations de fichier arbitraires, avec les privilèges requis par chacun."
keywords:
  - UTL_FILE
  - objet directory
  - DBMS_LOB
  - lecture de fichier
  - écriture de fichier Oracle
---

# Manipulation de fichiers

Une session Oracle privilégiée peut lire et écrire des fichiers de l'hôte, ce qui mène à la divulgation de configuration et d'identifiants et, via l'écriture, à l'exécution de code. Chaque voie nécessite un privilège execute et, pour `UTL_FILE`, un objet directory.

`UTL_FILE` lit et écrit des fichiers à l'intérieur d'un répertoire vers lequel pointe un objet `DIRECTORY`. Avec `CREATE ANY DIRECTORY` (ou un répertoire lisible existant), créez-en un ou réutilisez-le et lisez-le. `CREATE DIRECTORY` est du DDL, qui ne peut pas apparaître directement dans un bloc PL/SQL, il s'exécute donc via `EXECUTE IMMEDIATE` à l'intérieur du bloc injecté (et Oracle n'empile pas les instructions simples, ce doit donc être un contexte PL/SQL) :

```sql
DECLARE f UTL_FILE.FILE_TYPE; s VARCHAR2(4000);
BEGIN
  EXECUTE IMMEDIATE 'CREATE OR REPLACE DIRECTORY d AS ''/etc''';
  f:=UTL_FILE.FOPEN('D','passwd','R'); UTL_FILE.GET_LINE(f,s); -- s now holds a line of /etc/passwd
END;
```

`UTL_FILE.FOPEN(..., 'W')` écrit, ce qui dépose un script ou une entrée de tâche planifiée dans un répertoire accessible en écriture. La lecture via `UTL_FILE` est orientée ligne ; `DBMS_LOB` avec un `BFILE` lit des octets pour les fichiers binaires.

La voie la plus générale est Java. Un schéma à qui `java.io.FilePermission` est accordée peut charger une petite classe Java qui lit ou écrit n'importe quel chemin avec les privilèges OS du serveur de base de données, contournant entièrement le modèle d'objet directory.

Parce que `UTL_FILE` et `CREATE DIRECTORY` sont des instructions et du PL/SQL, cette voie nécessite généralement un contexte PL/SQL injectable ou une exécution empilée (qu'Oracle n'accorde qu'à l'intérieur du PL/SQL), ainsi que les privilèges directory et execute. Énumérez d'abord les répertoires existants avec `SELECT directory_name, directory_path FROM all_directories` pour trouver un répertoire pointant déjà vers un emplacement utile.

## Outils

- **ODAT** (Oracle Database Attacking Tool) : lit et écrit des fichiers du serveur via `UTL_FILE`, `DBMS_LOB` et Java.
- **sqlplus** (ou SQLcl) : le client Oracle pour créer des objets directory et appeler `UTL_FILE`.
- Tests manuels avec Burp Repeater et le client Oracle.

## Références

- Oracle Database PL/SQL Packages and Types Reference : UTL_FILE, DBMS_LOB ; CREATE DIRECTORY
- OWASP Testing Guide : Testing for SQL Injection
