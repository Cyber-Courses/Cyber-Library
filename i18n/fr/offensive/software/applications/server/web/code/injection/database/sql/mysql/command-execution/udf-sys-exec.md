---
title: "Exécution de commandes MySQL via l'UDF lib_mysqludf_sys"
description: "Charger la bibliothèque lib_mysqludf_sys et enregistrer sys_exec/sys_eval pour exécuter des commandes système directement depuis une injection MySQL privilégiée."
keywords:
  - lib_mysqludf_sys
  - sys_exec
  - sys_eval
  - fonction définie par l'utilisateur
  - RCE UDF MySQL
---

# UDF sys_exec

Une fonction définie par l'utilisateur étend MySQL avec du code natif chargé depuis une bibliothèque partagée dans le répertoire de plugins. La bibliothèque `lib_mysqludf_sys` expose `sys_exec` (exécuter une commande) et `sys_eval` (exécuter une commande et retourner sa sortie), qui donnent l'exécution de commandes sous le compte de service MySQL directement depuis SQL.

La chaîne comporte trois étapes, chacune nécessitant des privilèges :

1. Écrire la bibliothèque dans le répertoire de plugins avec `INTO DUMPFILE` (nécessite `FILE`, un `secure_file_priv` permissif et un `@@plugin_dir` accessible en écriture).
2. Enregistrer la fonction, ce qui nécessite `CREATE FUNCTION` (fait partie des privilèges d'administration) :

```sql
CREATE FUNCTION sys_exec RETURNS INT SONAME 'lib_mysqludf_sys.so';
CREATE FUNCTION sys_eval RETURNS STRING SONAME 'lib_mysqludf_sys.so';
```

3. L'appeler :

```sql
SELECT sys_eval('id');
```

`sys_eval('id')` retourne la sortie de la commande sous forme de chaîne, donc elle se relit in-band ; `sys_exec` ne retourne qu'un code de sortie et est utilisé pour les actions sans retour comme ajouter un utilisateur ou démarrer un reverse shell.

Cette voie dépend des requêtes empilées ou d'un contexte de requête qui autorise `CREATE FUNCTION`, ce que les pilotes PHP courants ne fournissent pas, donc en pratique elle s'atteint via une console d'administration, un client multi-instructions ou une injection dans un contexte qui permet plusieurs instructions. Le gain est l'exécution sous le compte de service de la base plutôt que sous l'utilisateur web moins privilégié sous lequel s'exécute un web shell.

## Outils

- sqlmap (`--os-shell` automatise les voies OUTFILE et UDF)
- lib_mysqludf_sys

## Références

- MySQL Reference Manual: CREATE FUNCTION (UDF), `plugin_dir`
- OWASP Testing Guide: Testing for SQL Injection
