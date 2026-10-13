---
order: 1
title: "Microsoft SQL Server"
description: "La surface offensive de Microsoft SQL Server : s'authentifier et énumérer, exécuter des commandes OS avec xp_cmdshell et d'autres sinks, se déplacer latéralement via les serveurs liés, escalader via l'usurpation et les chaînes de propriété TRUSTWORTHY, et forcer l'authentification du compte de service pour le relais NTLM."
keywords:
  - MSSQL
  - SQL Server
  - xp_cmdshell
  - serveurs liés
  - relais NTLM
---

# Microsoft SQL Server

Microsoft SQL Server (MSSQL, port 1433) est la cible de base de données la plus productive dans un parc Windows. Il authentifie à la fois des connexions SQL et des **comptes Windows/de domaine**, son service s'exécute sous un compte privilégié qui parle **NTLM**, et il est livré avec des procédures stockées qui **exécutent des commandes du système d'exploitation**, **atteignent d'autres serveurs** et **touchent au système de fichiers**. Une seule connexion peu privilégiée s'enchaîne souvent jusqu'à SYSTEM sur l'hôte ou Domain Admin.

## Pourquoi MSSQL compte

- Il est **intégré à AD** : les utilisateurs de domaine peuvent être des principaux de base, et l'authentification du compte de service peut être forcée et [relayée](coercion-and-relay.md).
- Il exécute du code : `xp_cmdshell` et plusieurs autres sinks donnent une exécution de commandes OS sous le compte de service.
- Il est **mis en réseau avec d'autres serveurs** via les serveurs liés, donc une instance pivote vers plusieurs.
- Son modèle de privilèges (usurpation, propriété de base, TRUSTWORTHY) comporte des chaînes d'**escalade** bien rodées de `public` vers `sysadmin`.

## Pages

- **[Énumération](enumeration.md)** : trouver les instances, versions, connexions, bases et les privilèges que vous détenez.
- **[Accès](access.md)** : s'authentifier en tant que connexion SQL ou de domaine et les rôles qui comptent.
- **[Exécution de commandes](command-execution.md)** : xp_cmdshell, automatisation OLE et CLR pour les commandes OS.
- **[Serveurs liés](linked-servers.md)** : interroger et exécuter à travers les liens de confiance entre serveurs.
- **[Usurpation](impersonation.md)** : chaînes de propriété EXECUTE AS et TRUSTWORTHY vers sysadmin.
- **[Coercition et relais](coercion-and-relay.md)** : xp_dirtree/xp_fileexist pour capturer ou relayer le compte de service.

## Références

- [HackTricks: pentesting MSSQL (1433)](https://hacktricks.wiki/en/network-services-pentesting/pentesting-mssql-microsoft-sql-server/index.html)
- [NetExec: MSSQL protocol and modules](https://www.netexec.wiki/mssql-protocol)
- [Impacket mssqlclient](https://github.com/fortra/impacket/blob/master/examples/mssqlclient.py)
