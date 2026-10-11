---
title: "Énumération des GPO et des OU : objets de stratégie de groupe, liaisons et portée"
order: 1
description: "Énumération des objets de stratégie de groupe et des unités organisationnelles Active Directory : quelles GPO existent, où elles sont liées, à qui elles s'appliquent et qui peut les éditer, pour trouver des chemins d'élévation de privilèges basés sur les stratégies."
keywords:
  - énumération des GPO
  - unité organisationnelle
  - gPLink
  - stratégie de groupe
  - permissions des GPO
---

# Énumération des GPO et des OU

Les objets de stratégie de groupe (GPO) poussent de la configuration (et, utilement pour un attaquant, des tâches planifiées, des scripts et des appartenances à des groupes) vers les utilisateurs et les ordinateurs auxquels ils sont liés. Un principal qui peut **éditer** une GPO peut exécuter du code sur tout ce à quoi cette GPO s'applique. L'énumération répond ici à trois questions : quelles GPO existent, à quoi elles sont liées (pour que vous connaissiez le rayon d'impact) et qui peut les modifier.

## Cartographier les GPO vers leurs cibles

Les GPO sont liées aux sites, domaines et unités organisationnelles via l'attribut `gPLink` du conteneur. Pour savoir qui une GPO affecte, cartographiez GPO vers liaison vers les objets sous ce conteneur :

```
# PowerView
Get-DomainGPO -Properties displayname,gpcfilesyspath
Get-DomainOU -Properties name,gplink
Get-DomainGPO -Identity '{GUID}' | Get-DomainOU   # resolve a GPO to its linked OUs
Get-DomainComputer -SearchBase 'OU=Workstations,DC=example,DC=local'   # who is under that OU
```

```bash
# From Linux
nxc ldap <dc> -u user -p pass --gpo
ldapsearch ... '(objectClass=groupPolicyContainer)' displayName gPCFileSysPath
```

Le `gPCFileSysPath` pointe vers les fichiers de la GPO dans `SYSVOL` (`\\domain\SYSVOL\domain\Policies\{GUID}`), lisibles par n'importe quel utilisateur du domaine, là où les mots de passe des Group Policy Preferences ont historiquement fuité.

## Trouver les GPO éditables

Le chemin d'élévation est une GPO dont la DACL accorde à un principal que vous contrôlez un accès en écriture (`WriteProperty`/`GenericWrite`/`GenericAll`). Énumérez les permissions des GPO de la même façon que l'ACL de n'importe quel objet :

```
Get-DomainGPO | Get-DomainObjectAcl -ResolveGUIDs |
  ? { $_.ActiveDirectoryRights -match 'WriteProperty|GenericWrite|GenericAll' }
```

BloodHound dessine cela comme une arête `GenericWrite`/`GPOAbuse` du principal vers la GPO, puis montre chaque ordinateur et utilisateur que la GPO atteint.

## Notes d'exploitation

- Priorisez les GPO liées aux OU qui contiennent des ordinateurs **privilégiés ou nombreux** ; éditer une GPO liée à une OU Domain Controllers est critique pour le domaine.
- Un arbre de stratégies `SYSVOL` lisible vaut la peine d'y chercher `cpassword` et des secrets d'autologon ([Group Policy Preferences](group-policy-preferences.md)) ainsi que des scripts référençant des identifiants.
- L'énumération identifie la GPO éditable à forte portée ; la transformer en exécution est traité par [éditer une GPO](editing-a-gpo.md), et atteindre des objets via une OU accessible en écriture par [lier une GPO](linking-a-gpo.md).

## Outils

- **PowerView `Get-DomainGPO` / `Get-DomainOU`** : cartographie des GPO et des OU avec les ACL.
- **NetExec (`nxc`) `ldap --gpo`** : listage des GPO depuis Linux (plus `-M gpp_password` sur SYSVOL).
- **Group3r / Grouper2** : auditer le contenu et les ACE des GPO pour y trouver des paramètres exploitables.
- **BloodHound** : portée GPO-vers-cible, arêtes `GenericWrite`-sur-GPO et `WriteGPLink`.

## Références

- The Hacker Recipes: Group policies
- Microsoft: Group Policy architecture and gPLink
