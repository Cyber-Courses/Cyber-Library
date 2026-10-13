---
order: 2
title: "Exécution de commandes via injection IBM Db2"
description: "Atteindre l'exécution de commandes OS depuis une injection IBM Db2 privilégiée via des routines externes C ou Java, et pourquoi ADMIN_CMD et QCMDEXC ne sont pas des shells OS sur Db2 LUW."
keywords:
  - exécution de commandes Db2
  - routine externe
  - CREATE PROCEDURE LANGUAGE C
  - ADMIN_CMD
  - QCMDEXC
---

# Exécution de commandes

Db2 LUW n'a pas de fonction d'exécution de commandes OS intégrée, et deux noms qui en ont l'air sont souvent mal employés. `SYSPROC.ADMIN_CMD` exécute des commandes administratives du CLP Db2 (telles que `EXPORT`, `IMPORT`, `RUNSTATS`, `REORG`), pas un shell OS arbitraire. `QSYS2.QCMDEXC` exécute bien des commandes OS, mais appartient à Db2 for i (IBM i), pas à Db2 LUW, et ne s'applique donc pas ici. Ne présentez ni l'un ni l'autre comme un shell sur LUW.

Sur Db2 LUW, la vraie voie est une routine externe. Avec `DBADM` ou `CREATE_EXTERNAL_ROUTINE`, définissez une procédure ou une fonction implémentée en C ou Java qui appelle le système d'exploitation, puis invoquez-la :

```sql
CREATE PROCEDURE shell(IN cmd VARCHAR(500)) EXTERNAL NAME 'mylib!run' LANGUAGE C PARAMETER STYLE GENERAL;
CALL shell('id');
```

La bibliothèque externe doit exister sur le serveur (dans le répertoire `function` de l'instance), donc cela se combine généralement avec une primitive d'écriture de fichier pour déposer d'abord la bibliothèque, ou réutilise une bibliothèque déjà présente. La routine s'exécute dans le compte du processus fenced de Db2, dont les privilèges déterminent le pied d'accès. Les routines Java (`LANGUAGE JAVA`) qui encapsulent `Runtime.exec()` sont l'équivalent là où Java est configuré.

Parce que définir et exécuter des routines externes requiert une autorité élevée et généralement un contexte d'instruction/composé (Db2 n'empile pas les instructions simples via les pilotes standard), l'exécution de commandes est l'étape finale après que l'énumération des privilèges confirme la présence des autorités. Là où elles sont absentes, l'impact atteignable reste à l'extraction de données et aux actions administratives que `ADMIN_CMD` autorise réellement.

## Outils

- **db2** (ou clpplus) : le client Db2 pour définir et appeler la routine externe C ou Java.
- Test manuel avec Burp Repeater et le client Db2.

## Références

- IBM Db2 SQL Reference : CREATE PROCEDURE (external), SYSPROC.ADMIN_CMD
- OWASP Testing Guide : Testing for SQL Injection
