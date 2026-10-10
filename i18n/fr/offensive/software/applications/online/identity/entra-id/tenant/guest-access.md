---
title: "Accès invité : les invités B2B à l'intérieur du tenant"
order: 1
description: "Abus des comptes invités B2B : invitation à faible friction, énumération de l'annuaire par un invité, et chemins d'escalade d'invité à membre."
keywords:
  - accès invité
  - B2B
  - invitation
  - énumération d'invités
  - utilisateur externe
---

# Accès invité

Un invité B2B est une identité externe à laquelle on attribue un objet à l'intérieur du tenant cible. Les invités sont souvent trop peu restreints : ils peuvent énumérer les utilisateurs, les groupes et les applications, et dans les tenants permissifs atteindre des données et même escalader vers un accès équivalent à celui d'un membre. Un point d'ancrage invité est un moyen peu coûteux de pénétrer un tenant que vous ne possédez pas.

## Énumérer en tant qu'invité

```bash
# authenticated as a guest, enumerate the directory (default guest permissions often allow this)
az rest --method GET --url "https://graph.microsoft.com/v1.0/users?\$top=999"
az rest --method GET --url "https://graph.microsoft.com/v1.0/groups"
# roadrecon runs a full guest-scoped dump
roadrecon gather
```

## Notes d'exploitation

- Les permissions d'invité par défaut dans de nombreux tenants autorisent une lecture large des utilisateurs, groupes et applications, ce qui cartographie toute la surface d'attaque depuis l'extérieur.
- Les invités peuvent posséder des groupes et des applications ou y être ajoutés, ce qui ouvre des arêtes vers les [groupes](../groups/index.md) et les [applications](../applications/index.md) permettant d'escalader vers un accès de membre.
- L'invitation est à faible friction : une invitation acceptée plante l'objet invité ; la validation (redemption) peut parfois être devancée pour prendre le compte.

## Outils

- **ROADtools** (`roadrecon`) : énumération de l'annuaire limitée au périmètre invité.
- **az cli** / **Graph**.

## Références

- [dirkjanm.io : abus des invités B2B](https://dirkjanm.io/)
- [HackTricks Cloud : accès invité](https://cloud.hacktricks.wiki/en/pentesting-cloud/azure-security/index.html)
- [Microsoft : collaboration B2B](https://learn.microsoft.com/entra/external-id/what-is-b2b)
