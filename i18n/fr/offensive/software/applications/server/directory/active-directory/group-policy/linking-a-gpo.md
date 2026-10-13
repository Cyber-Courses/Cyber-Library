---
title: "Lier une GPO : appliquer une stratégie via gPLink"
order: 3
description: "Exploitation de l'accès en écriture à l'attribut gPLink d'une unité organisationnelle (le droit WriteGPLink) pour lier un objet de stratégie de groupe contrôlé ou éditable à l'OU, amenant chaque utilisateur et ordinateur qu'elle contient dans la portée de la stratégie."
keywords:
  - WriteGPLink
  - gPLink
  - unité organisationnelle
  - stratégie de groupe
  - New-GPLink
---

# Lier une GPO

Éditer une GPO nécessite un accès en écriture à cette GPO. **Lier** ne nécessite qu'un accès en écriture à l'attribut **`gPLink` d'une unité organisationnelle** (l'arête `WriteGPLink`). Avec lui, vous attachez une GPO que vous contrôlez à l'OU, et chaque utilisateur et ordinateur sous cette OU entre dans la portée de la GPO et la traite. C'est la seconde voie vers l'exécution de code par GPO, atteinte depuis le contrôle d'un conteneur plutôt que d'une stratégie.

## L'attaque

Vous avez besoin d'une écriture sur le `gPLink` de l'OU cible et d'une GPO dont vous contrôlez le contenu, et la GPO doit réellement **s'appliquer** aux cibles : son filtrage de sécurité doit accorder aux utilisateurs/ordinateurs cibles **Read + Apply Group Policy** (le `Authenticated Users` par défaut le fait, un filtre personnalisé peut ne pas le faire), et tout **filtre WMI** dessus doit s'évaluer à vrai pour eux. Vérifiez les deux avant de vous fier à la liaison.

`gPLink` est une **chaîne unique** d'entrées `[DN;flags]` concaténées, vous devez donc **ajouter** votre entrée, pas écraser l'attribut : une écriture brute remplace la valeur et dissocie silencieusement chaque GPO déjà présente sur l'OU (cassant probablement la stratégie de production). `New-GPLink` ajoute en toute sécurité ; les écritures LDAP brutes doivent faire du read-modify-write.

```powershell
# RSAT / Group Policy module: safely ADDS a link, preserving existing ones
New-GPLink -Name "Vulnerable GPO" -Target "OU=Workstations,DC=example,DC=local" -LinkEnabled Yes

# PowerView: -Set REPLACES, so read the current gPLink and append your entry
$ou  = Get-DomainOU -Identity 'OU=Workstations,DC=example,DC=local' -Properties gplink
$new = $ou.gplink + '[LDAP://cn={GPO-GUID},cn=policies,cn=system,DC=example,DC=local;0]'
Set-DomainObject -Identity 'OU=Workstations,DC=example,DC=local' -Set @{'gplink'=$new}
```

```bash
# From Linux: read the existing gPLink, then write back existing + new (never overwrite blind)
bloodyAD --host <dc> -d example.local -u user -p pass get object \
  'OU=Workstations,DC=example,DC=local' --attr gPLink      # capture current value
bloodyAD --host <dc> -d example.local -u user -p pass set object \
  'OU=Workstations,DC=example,DC=local' gPLink \
  -v '<existing-gplink>[LDAP://cn={GPO-GUID},cn=policies,cn=system,DC=example,DC=local;0]'
```

Poussez ensuite une [tâche planifiée immédiate dans cette GPO](editing-a-gpo.md) (ou utilisez une GPO qui en porte déjà une), et les machines nouvellement dans la portée l'exécutent.

## Obtenir une GPO à lier

- **Une GPO que vous pouvez déjà éditer** : liez-la à la nouvelle OU, élargissant sa portée aux objets de l'OU.
- **Une GPO que vous créez** : les membres de **Group Policy Creator Owners** (ou avec des droits de création dans le conteneur Policies) peuvent créer une nouvelle GPO, la remplir et la lier. Par défaut, les utilisateurs ordinaires ne peuvent pas créer de GPO, cette voie nécessite donc cette appartenance.
- Combinés, `WriteGPLink` sur une OU à forte valeur plus n'importe quelle GPO éditable constitue une exécution de code sur le contenu de cette OU.

## Notes d'exploitation

- Visez une OU qui contient des ordinateurs **privilégiés ou nombreux** ; `WriteGPLink` sur l'OU Domain Controllers est critique pour le domaine.
- La liaison prend effet au prochain rafraîchissement de la stratégie de groupe pour les objets de l'OU, comme toute GPO appliquée.
- **Retirez la liaison** ensuite (rétablissez la valeur `gPLink` d'origine) ; une liaison inattendue sur une OU sensible est un artefact visible. `gPLink` est une chaîne unique d'entrées `[DN;flags]`, préservez donc les autres lors de l'édition.
- BloodHound modélise cela comme l'arête `WriteGPLink`/`GPLink` d'un principal vers une OU et résout les objets affectés.

## Outils

- **bloodyAD** / **ldapmodify** : définir le `gPLink` de l'OU depuis Linux.
- **PowerView `Set-DomainObject`** / **RSAT `New-GPLink`** : lier depuis un shell Windows.
- **BloodHound** : arêtes `WriteGPLink` et l'ensemble des objets affectés.

## Références

- The Hacker Recipes: Group policies
- SpecterOps: BloodHound WriteGPLink edge
