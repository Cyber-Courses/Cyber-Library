---
title: "Approbations intra-forêt : traverser les domaines à l'intérieur d'une forêt"
order: 2
description: "Se déplacer d'un domaine Active Directory enfant ou frère vers la racine de la forêt en injectant un SID privilégié dans un ticket forgé, parce que les approbations intra-forêt n'appliquent pas de filtrage de SID et que la frontière de sécurité est la forêt, pas le domaine."
keywords:
  - intra-forest trust
  - SID history
  - extra SIDs
  - Enterprise Admins
  - forest root
---

# Approbations intra-forêt

La frontière de sécurité dans Active Directory est la **forêt**, pas le domaine. Chaque domaine d'une forêt fait confiance aux autres via des approbations automatiques, bidirectionnelles et transitives, et ces approbations n'appliquent **pas** de filtrage de SID. Ainsi, le contrôle d'un seul domaine équivaut au contrôle de toute la forêt : un ticket forgé dans un domaine enfant peut revendiquer l'appartenance à un groupe de la racine de forêt, et la racine l'honore.

## Pourquoi un domaine possède la forêt

- Un ticket Kerberos porte les SID de son détenteur dans son PAC, y compris un champ **`extraSids`** (le mécanisme derrière le SID history) pour les SID d'autres domaines.
- Les approbations intra-forêt laissent passer ces SID sans filtrage **par défaut**, parce qu'au sein d'une forêt ils sont présumés légitimes. (L'exception est une approbation explicitement marquée `QuarantinedWithinForest`, où seuls le SID enterprise-DC et les SID listés sur l'objet du domaine approuvé passent, filtrant un `-519` forgé ; vérifiez ce point lors de l'énumération.)
- Le groupe **Enterprise Admins** (RID 519, dans la racine de forêt) et les **Domain Admins** du domaine racine portent à l'échelle de la forêt. La compromission d'un domaine enfant capable de forger des tickets peut donc ajouter ces SID et agir en tant qu'administrateur de forêt.

## La technique

Avec le domaine enfant entièrement compromis (sa clé `krbtgt`, obtenue par [DCSync](../authentication/credentials/ntds-and-dcsync.md)), forgez un golden ticket qui porte le SID Enterprise Admins dans `extraSids` :

```bash
# Child domain krbtgt + child domain SID, plus the forest-root Enterprise Admins SID
ticketer.py -nthash <child-krbtgt-hash> -domain-sid <CHILD-domain-SID> \
  -domain child.example.local \
  -extra-sid <ROOT-domain-SID>-519 Administrator
export KRB5CCNAME=Administrator.ccache
secretsdump.py -k -no-pass -just-dc root-dc.example.local   # DCSync the forest root
```

Le ticket forgé est un TGT de domaine enfant, mais le SID `-519` dans `extraSids` fait traiter le détenteur par la racine de forêt comme un Enterprise Admin, de sorte que vous pouvez immédiatement faire un DCSync ou agir contre le domaine racine.

## Le SID history comme persistance

La même idée d'`extraSids`/SID history persiste lorsqu'elle est écrite dans l'attribut **`sIDHistory`** d'un compte : un compte portant un SID privilégié dans `sIDHistory` conserve cet accès au fil des connexions sans appartenance de groupe qui ressorte. Écrire `sIDHistory` nécessite normalement un accès de niveau DC (ou `DsAddSidHistory`), il s'agit donc d'une manœuvre de persistance post-compromission plutôt que d'une escalade.

## Notes d'exploitation

- C'est la raison du « posséder un domaine, posséder la forêt » : une seule clé `krbtgt` enfant plus le SID racine atteint Enterprise Admin, les domaines ne sont donc pas une frontière de confinement face à un adversaire à l'échelle de la forêt.
- Nécessite la clé `krbtgt` du domaine enfant et le **SID du domaine racine de forêt** (lu lors de l'[énumération des approbations](trust-enumeration.md)) ; le RID `-519` cible Enterprise Admins.
- Une variante diamond/sapphire (modifier un vrai ticket) est plus furtive qu'un golden ticket créé de toutes pièces là où les anomalies de PAC sont surveillées (voir [tickets forgés](../authentication/kerberos/forged-tickets.md)).

## Outils

- **Impacket** (`ticketer.py -extra-sid`, `secretsdump.py`, `raiseChild.py` qui automatise tout le chemin enfant-vers-racine).
- **Mimikatz** (`kerberos::golden /sids:`) : golden ticket avec des SID supplémentaires sous Windows.

## Références

- The Hacker Recipes: intra-forest trusts and SID history
- Microsoft: the forest as a security boundary, SID filtering
