---
title: "Appartenance à un groupe : rejoindre un groupe privilégié"
order: 3
description: "Utilisation de AddMember, AddSelf ou GenericWrite sur l'attribut member d'un groupe Active Directory pour y ajouter un principal contrôlé, héritant des droits du groupe, y compris les considérations sur les groupes imbriqués et primaryGroupID et la voie Account Operators."
keywords:
  - AddMember
  - AddSelf
  - appartenance à un groupe
  - primaryGroupID
  - Account Operators
---

# Appartenance à un groupe

Écrire l'attribut **`member`** d'un groupe (le droit `AddMember`, le `AddSelf` limité à soi-même, ou un `GenericWrite`/`GenericAll` plus large) vous permet d'y ajouter un principal contrôlé et d'hériter de chaque droit que détient le groupe. Contre un groupe privilégié, c'est une élévation immédiate, et c'est souvent l'arête la plus simple que BloodHound fait ressortir.

## S'ajouter soi-même

```bash
# bloodyAD
bloodyAD --host <dc> -d example.local -u user -p pass add groupMember 'Domain Admins' 'user'

# Samba net
net rpc group addmem 'Domain Admins' 'user' -U 'example.local/user%pass' -S <dc>
```

```powershell
# PowerView
Add-DomainGroupMember -Identity 'Domain Admins' -Members 'user'
```

L'appartenance à un groupe à forte valeur prend généralement effet à la prochaine ouverture de session/au prochain ticket (le nouveau SID est dans le PAC), demandez donc un nouveau TGT après vous être ajouté.

## Les détails qui décident si ça fonctionne

- **Groupes imbriqués** : l'appartenance aux groupes AD est transitive, vous n'avez donc pas besoin d'une arête vers le groupe cible lui-même. Une écriture sur n'importe quel groupe qui est (même indirectement) membre d'un groupe privilégié suffit. BloodHound résout déjà cela.
- **`primaryGroupID` (furtivité, pas un moyen d'entrée)** : cela n'accorde **pas** une nouvelle appartenance. AD n'autorise à définir le `primaryGroupID` d'un utilisateur que vers un groupe dont l'utilisateur est **déjà** membre, et la mise à jour supprime alors l'entrée `member` explicite (l'appartenance au groupe primaire est implicite). Cela ne peut donc que **masquer** une appartenance que vous avez déjà, en gardant une appartenance privilégiée hors de la liste `member` du groupe après coup, pas rejoindre un groupe dont vous ne faisiez pas partie.
- **Account Operators** : les membres peuvent modifier la plupart des groupes et comptes non protégés mais **pas** les groupes protégés (ceux gardés par [AdminSDHolder](adminsdholder.md) comme Domain Admins). C'est un point d'ancrage « presque admin » courant : utilisez-le pour atteindre n'importe quel compte non protégé, pas directement les comptes protégés.
- **Groupes builtin vs groupes de domaine** : ajouter à `Administrators` (builtin, de domaine local) accorde l'admin local du contrôleur de domaine ; ajouter à `Domain Admins` (global) est plus large. Choisissez le groupe qui correspond à l'accès dont vous avez besoin.

## Notes d'exploitation

- Les changements d'appartenance sont visibles dans la liste `member` du groupe et dans les événements d'audit du service d'annuaire, ils sont donc faciles à repérer ; la variante `primaryGroupID` échappe à un examen superficiel de `member`.
- Retirez-vous ensuite pour réduire l'empreinte, sauf si vous voulez l'appartenance comme persistance.
- Une écriture sur un groupe dont vous ne pensiez pas qu'il importait peut quand même l'emporter grâce à l'imbrication, fiez-vous donc au chemin BloodHound plutôt qu'à un coup d'œil sur les groupes évidents.

## Outils

- **bloodyAD** (`add groupMember`), **Samba `net rpc group addmem`** : ajouter depuis Linux.
- **PowerView** (`Add-DomainGroupMember`, `Set-DomainObject` pour `primaryGroupID`) : en local.
- **BloodHound** : résout l'appartenance imbriquée et le chemin le plus court vers un groupe privilégié.

## Références

- The Hacker Recipes: AddMember / group abuse
- SpecterOps: BloodHound AddMember edge
