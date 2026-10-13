---
title: "Écrire des octets exacts avec MySQL INTO DUMPFILE"
description: "Utiliser SELECT INTO DUMPFILE pour écrire une seule ligne verbatim depuis une injection MySQL, l'écriture binaire sûre utilisée pour déposer une bibliothèque partagée UDF."
keywords:
  - INTO DUMPFILE
  - écriture de fichier binaire
  - bibliothèque UDF
  - écriture de fichier MySQL
  - répertoire de plugins
---

# INTO DUMPFILE

`SELECT ... INTO DUMPFILE '/path/file'` écrit une seule ligne dans un fichier sans aucun formatage de ligne ou de colonne, donc les octets atterrissent exactement tels que sélectionnés. C'est l'équivalent binaire sûr d'`INTO OUTFILE` : `OUTFILE` ajoute des terminateurs et de l'échappement qui corromperaient un binaire, alors que `DUMPFILE` ne le fait pas. Son usage principal est de déposer un artefact compilé, le plus souvent la bibliothèque partagée pour un UDF d'exécution de commandes.

Le contenu est fourni sous forme de littéral hexadécimal afin que des octets arbitraires survivent au transport, et la cible est le répertoire de plugins du serveur :

```sql
' UNION SELECT 0x7f454c46...  INTO DUMPFILE '/usr/lib/mysql/plugin/lib_mysqludf_sys.so'-- 
```

Trouvez d'abord le répertoire de plugins pour que la bibliothèque atterrisse là où `CREATE FUNCTION` la cherchera :

```sql
' UNION SELECT @@plugin_dir,NULL,NULL-- 
```

`DUMPFILE` n'écrit qu'une seule ligne, donc il ne peut pas vider un résultat multi-lignes, et comme `OUTFILE`, il nécessite le privilège `FILE`, un `secure_file_priv` permissif, une destination accessible en écriture et un fichier cible inexistant. Une fois la bibliothèque en place, l'UDF est enregistré et appelé comme indiqué dans la page `sys_exec`.

## Outils

- **sqlmap** : écrit des octets exacts sur l'hôte avec `--file-write` en utilisant `INTO DUMPFILE`.
- **mysql** : client officiel pour confirmer `@@plugin_dir` et le chemin d'écriture.

## Références

- MySQL Reference Manual: SELECT INTO DUMPFILE, `plugin_dir`, `secure_file_priv`
- OWASP Testing Guide: Testing for SQL Injection
