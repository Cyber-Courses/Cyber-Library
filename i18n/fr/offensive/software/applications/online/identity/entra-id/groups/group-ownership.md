---
title: "Propriété de groupe : un propriétaire s'ajoute comme membre"
description: "Abuser de la propriété d'un groupe pour vous ajouter, vous ou un principal contrôlé, comme membre, en héritant de tout accès ou rôle que le groupe confère."
keywords:
  - propriété de groupe
  - propriétaire
  - ajouter un membre
  - appartenance
  - élévation de privilèges
---

# Propriété de groupe

Un **propriétaire** de groupe peut gérer son appartenance, de sorte que posséder un groupe signifie que vous pouvez vous y ajouter et hériter de tout ce qu'il accorde : accès à une application, rôles de ressources, ou un rôle d'annuaire lié. La propriété est déléguée avec désinvolture et rarement auditée, ce qui en fait un accès discret courant vers un groupe privilégié.

## S'ajouter comme membre

```bash
# list groups you own, then add a member
az rest --method GET --url "https://graph.microsoft.com/v1.0/me/ownedObjects"
az rest --method POST --url "https://graph.microsoft.com/v1.0/groups/<group>/members/\$ref" \
  --body '{"@odata.id":"https://graph.microsoft.com/v1.0/directoryObjects/<you>"}'
```

## Notes d'exploitation

- Visez les propriétaires de groupes qui soutiennent des app roles privilégiés ou qui sont [role-assignable](role-assignable-groups.md) ; l'appartenance devient alors ces permissions.
- Les propriétaires peuvent aussi ajouter un service principal que vous contrôlez, donnant un point d'ancrage non interactif dans le groupe.
- Les changements d'appartenance de groupe sont visibles dans les listes d'appartenance, c'est donc plus durable que furtif.

## Outils

- **az cli** / **Graph** (`groups/{id}/members/$ref`).
- **AzureHound** / **BARK** : arêtes propriétaire-vers-groupe.

## Références

- [SpecterOps : AzureHound owner edges](https://github.com/BloodHoundAD/AzureHound)
- [HackTricks Cloud : group ownership](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Microsoft : manage group membership](https://learn.microsoft.com/entra/fundamentals/how-to-manage-groups)
