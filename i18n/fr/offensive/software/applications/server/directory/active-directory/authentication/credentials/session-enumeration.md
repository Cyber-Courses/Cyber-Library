---
order: 19
title: "Énumération des sessions : trouver où les utilisateurs privilégiés sont connectés"
description: "Énumérer les sessions ouvertes et les administrateurs locaux à travers un domaine Active Directory pour trouver où les identifiants d'une cible sont disponibles pour un vol, les données qui transforment une arête de graphe en plan de déplacement latéral."
keywords:
  - énumération des sessions
  - NetSessionEnum
  - NetWkstaUserEnum
  - utilisateurs connectés
  - administrateur local
---

# Énumération des sessions

Savoir *qui est connecté où* est ce qui rend possible un déplacement latéral ciblé : si un Domain Admin a une session active sur un hôte que vous pouvez atteindre, ses identifiants (ou un ticket Kerberos exploitable) se trouvent dans la mémoire de cet hôte, qui devient donc un tremplin. L'énumération des sessions est aussi la reconnaissance la plus bruyante et la plus dépendante des accès, car elle interroge de nombreux hôtes plutôt que seulement le DC.

## Les sources de données

Deux appels RPC fournissent l'essentiel :

- **`NetSessionEnum`** liste les sessions vers un hôte (quels utilisateurs ont une connexion, typiquement vers des partages de fichiers), historiquement interrogeable à distance par tout utilisateur authentifié, et sur de nombreux hôtes toujours le cas. Il révèle d'où les utilisateurs se connectent.
- **`NetWkstaUserEnum`** liste les utilisateurs connectés de façon interactive à un hôte, mais nécessite généralement un accès administrateur local sur la cible.
- **L'appartenance aux groupes locaux** (`NetLocalGroupGetMembers`) révèle qui est administrateur local sur chaque hôte, l'autre moitié de la cartographie latérale.

```bash
# NetExec gathers sessions, logged-on users, and local admins across a range
nxc smb <range> -u user -p pass --sessions
nxc smb <range> -u user -p pass --loggedon-users
nxc smb <range> -u user -p pass --local-auth --groups Administrators
```

Depuis un hôte du domaine, PowerView encapsule les mêmes appels :

```
Get-NetSession -ComputerName <host>
Get-NetLoggedOn -ComputerName <host>
Find-DomainUserLocation -UserGroupIdentity 'Domain Admins'   # hunt a group's sessions
```

## Les sessions BloodHound

La collecte des sessions et des administrateurs locaux par BloodHound correspond exactement à ces données à grande échelle : les arêtes `HasSession` proviennent de `NetSessionEnum`, et les arêtes `AdminTo` de l'appartenance aux groupes locaux. La collecter (voir [BloodHound](../../dacl/bloodhound.md)) permet au graphe de calculer automatiquement « du principal possédé au Domain Admin via une session sur l'hôte X ».

## Notes d'exploitation

- Les données de session sont **volatiles** : la session d'un utilisateur prend fin lorsqu'il se déconnecte, recollectez donc juste avant d'agir plutôt que de vous fier à un ancien instantané.
- Microsoft a durci les permissions de `NetSessionEnum` à distance dans les versions ultérieures de Windows, un domaine moderne et à jour peut donc renvoyer peu de choses sans accès privilégié ; là où cela fonctionne encore, c'est de grande valeur.
- La cible de la chasse aux sessions est presque toujours les hôtes où des comptes privilégiés (Domain Admins, comptes de service, helpdesk aux droits étendus) sont connectés, car c'est là que leurs identifiants peuvent être dumpés.

## Outils

- **NetExec (nxc)** : `--sessions`, `--loggedon-users`, énumération des administrateurs locaux sur des plages.
- **PowerView** : `Get-NetSession`, `Find-DomainUserLocation`.
- **BloodHound** : arêtes de sessions et d'administrateurs locaux à grande échelle.

## Références

- [NetExec: sessions and logged-on users](https://github.com/Pennyw0rth/NetExec)
- [Impacket (fortra): netview and NetSessionEnum](https://github.com/fortra/impacket)
- [SpecterOps: BloodHound session collection](https://bloodhound.specterops.io/)
- [The Hacker Recipes: Active Directory recon](https://www.thehacker.recipes/ad/recon/)
