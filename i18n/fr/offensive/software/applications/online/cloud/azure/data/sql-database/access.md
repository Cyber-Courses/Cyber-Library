---
title: "Accès : atteindre Azure SQL via l'authentification, le pare-feu et l'identité du serveur"
description: "Atteindre Azure SQL via l'authentification Entra, les connexions SQL, les utilisateurs de base de données contenue ou l'identité managée du serveur."
keywords:
  - accès Azure SQL
  - authentification Entra
  - utilisateur contenu
  - connexion SQL
  - identité managée
---

# Accès

Une base de données Azure SQL est accessible quand le pare-feu du serveur logique admet votre source et que vous détenez un identifiant ou un jeton qu'il accepte. Les connexions SQL et les utilisateurs de base de données contenue s'authentifient avec un mot de passe ; l'authentification Microsoft Entra s'authentifie avec un jeton généré pour la ressource `https://database.windows.net/`, qu'un utilisateur compromis ou l'identité managée d'une ressource peut obtenir.

## Ouvrir le pare-feu et l'atteindre

```bash
# add a server firewall rule for your source (needs Microsoft.Sql/servers/firewallRules/write)
az sql server firewall-rule create -g <rg> -s <server> -n open \
  --start-ip-address 0.0.0.0 --end-ip-address 255.255.255.255
az sql db list --server <server> -g <rg> -o table
```

## S'authentifier

```bash
# Entra token as the current principal or a managed identity, used as the SQL password
TOKEN=$(az account get-access-token --resource https://database.windows.net/ --query accessToken -o tsv)
sqlcmd -S <server>.database.windows.net -d <db> -G -P "$TOKEN" -Q "SELECT USER_NAME();"

# or a recovered SQL login / contained user
sqlcmd -S <server>.database.windows.net -d <db> -U app -P '<pw>' -Q "SELECT name FROM sys.database_principals;"
```

## Notes d'exploitation

- Un utilisateur de base de données contenue n'existe que dans la base de données, de sorte qu'il survit même quand les connexions du serveur sont renouvelées, et il est souvent oublié dans l'hygiène des identifiants.
- Depuis une VM ou un App Service doté d'une identité managée, générez le jeton `database.windows.net` depuis les [métadonnées d'instance](../../credentials/instance-metadata/index.md) et authentifiez-vous sans mot de passe ; le serveur doit avoir un administrateur Entra et l'identité mappée comme utilisateur.
- Une règle de pare-feu couvrant `0.0.0.0` à `255.255.255.255` est une ouverture bruyante mais efficace ; préférez votre propre IP de sortie quand vous la connaissez.

## Outils

- **sqlcmd / mssql-cli** : clients natifs prenant en charge l'authentification par jeton Entra (`-G`).
- **az cli** (`az sql server firewall-rule`, `az account get-access-token`).
- **MicroBurst** : utilitaires d'énumération Azure SQL.

## Références

- [Microsoft : authentification Entra Azure SQL](https://learn.microsoft.com/azure/azure-sql/database/authentication-aad-overview)
- [HackTricks Cloud : Azure SQL](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [NetSPI : MicroBurst](https://github.com/NetSPI/MicroBurst)
