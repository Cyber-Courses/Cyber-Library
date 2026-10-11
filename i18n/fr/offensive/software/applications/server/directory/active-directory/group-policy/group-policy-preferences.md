---
title: "Group Policy Preferences : identifiants dans SYSVOL"
order: 4
description: "Récupération d'identifiants depuis les XML de Group Policy Preferences laissés dans SYSVOL : le champ cpassword chiffré avec la clé AES publiée par Microsoft, et les identifiants d'autologon, tous deux lisibles par n'importe quel utilisateur du domaine et déchiffrables hors ligne."
keywords:
  - group policy preferences
  - cpassword
  - SYSVOL
  - gpp_password
  - MS14-025
---

# Preferences

Les Group Policy Preferences (GPP) permettent aux administrateurs de pousser des utilisateurs locaux, des tâches planifiées, des lecteurs mappés et des services vers les machines, et les plus anciennes stockent un mot de passe dans le XML de stratégie sous forme d'un champ **`cpassword`**. Ce champ est chiffré en AES avec une clé que Microsoft a **publiée**, donc quiconque peut lire le XML peut le déchiffrer. Comme les fichiers de stratégie résident dans **SYSVOL**, que tout utilisateur authentifié du domaine peut lire, un seul `cpassword` est un identifiant utilisateur-de-domaine-vers-admin-local (et souvent réutilisé partout) offert gratuitement.

## Trouver et déchiffrer

```bash
# NetExec: locate and decrypt cpassword across SYSVOL in one step
nxc smb <dc> -u user -p pass -M gpp_password
# Autologon credentials stored in Registry.xml preferences
nxc smb <dc> -u user -p pass -M gpp_autologin

# Manual: grep SYSVOL for cpassword, then decrypt with the published key
grep -rl 'cpassword' /mnt/sysvol/   # Groups.xml, Services.xml, Drives.xml, ScheduledTasks.xml, Datasources.xml
gpp-decrypt '<cpassword-blob>'
```

```powershell
# On a domain host
Get-GPPPassword
```

Les fichiers XML qui portent `cpassword` sont `Groups.xml` (utilisateurs locaux), `Services.xml`, `ScheduledTasks.xml`, `Drives.xml`, `Datasources.xml` et `Printers.xml` sous le dossier de préférences `Machine` ou `User` de chaque stratégie.

## Pourquoi c'est toujours d'actualité

- Microsoft a supprimé la capacité de **créer** de nouveaux mots de passe GPP en 2014 (MS14-025), mais il n'a **pas** effacé ceux existants, de sorte que les anciennes entrées `cpassword` persistent dans SYSVOL pendant des années.
- Le compte récupéré est fréquemment un **administrateur local** poussé vers de nombreuses machines avec le même mot de passe, de sorte qu'une seule trouvaille permet du [pass-the-hash](../authentication/ntlm/pass-the-hash.md)/une réutilisation de mot de passe à travers le parc.
- Il ne nécessite qu'un seul compte de domaine peu privilégié (lecture SYSVOL), ce qui en fait une recherche d'identifiants de premier coup aux côtés de la [reconnaissance](../authentication/credentials/index.md).

## Notes d'exploitation

- Faites un grep sur **tout** l'arbre de stratégies SYSVOL, pas seulement `Groups.xml` ; les identifiants se cachent aussi dans les services, les tâches planifiées et les préférences de source de données.
- Les identifiants d'autologon (`gpp_autologin`, depuis `Registry.xml`) sont en clair, pas chiffrés par `cpassword`, et exposent souvent un compte privilégié configuré pour connecter automatiquement une borne ou une machine de build.
- Validez et diffusez l'identifiant récupéré avec `nxc` pour cartographier où ce compte local est réutilisé.

## Outils

- **NetExec (`nxc`) `-M gpp_password` / `-M gpp_autologin`** : trouver et déchiffrer depuis Linux.
- **gpp-decrypt** : déchiffrer un blob `cpassword` avec la clé AES publiée.
- **PowerSploit `Get-GPPPassword`** : récupération en local.

## Références

- The Hacker Recipes: Group Policy Preferences passwords
- Microsoft: MS14-025 (Group Policy Preferences password removal)
