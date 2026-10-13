---
title: "Délégation : abuser de l'impersonation Kerberos"
order: 5
description: "Abus de la délégation Kerberos dans Active Directory : capture de TGT par délégation non contrainte, délégation contrainte avec transition de protocole (S4U2self/S4U2proxy) et délégation contrainte basée sur les ressources configurée depuis un objet ordinateur inscriptible."
keywords:
  - kerberos delegation
  - unconstrained delegation
  - constrained delegation
  - RBCD
  - S4U
---

# Délégation

La délégation Kerberos permet à un service d'agir **pour le compte** d'un utilisateur auprès d'un autre service, de sorte qu'un front-end (une application web, par exemple) puisse atteindre un back-end (une base de données) en tant que l'utilisateur. Cette fonctionnalité permet nécessairement à un compte d'obtenir des tickets pour une autre identité, et chaque type de délégation constitue donc une primitive d'impersonation lorsque le compte de délégation est compromis ou que sa configuration est inscriptible.

**Filiation.** La délégation non contrainte était le mécanisme d'origine, grossier, et reste le plus dangereux. La délégation contrainte et la transition de protocole (S4U) ont été ajoutées pour le restreindre, et la délégation contrainte basée sur les ressources, formalisée offensivement en 2018, a déplacé le contrôle sur l'objet cible, ce qui fait aujourd'hui qu'un unique objet ordinateur inscriptible devient une prise de contrôle.

## Les trois types

- **Non contrainte** : l'hôte de délégation reçoit et stocke le **TGT** de l'utilisateur, de sorte qu'il peut agir en tant que cet utilisateur *partout*. Compromettez un tel hôte (ou forcez un DC à s'authentifier auprès de lui) et vous capturez des TGT, y compris celui d'un DC.
- **Contrainte** : le compte ne peut demander des tickets pour un utilisateur que vers un **ensemble listé de SPN**, à l'aide des extensions S4U, souvent avec la **transition de protocole** qui lui permet de le faire sans que l'utilisateur ne s'authentifie jamais.
- **Contrainte basée sur les ressources (RBCD)** : la permission réside sur l'objet **cible** (`msDS-AllowedToActOnBehalfOfOtherIdentity`), de sorte qu'un accès en écriture à un objet ordinateur vous permet de désigner un compte autorisé à usurper l'identité de n'importe qui auprès de lui.

## Le mécanisme S4U

La délégation contrainte et la délégation basée sur les ressources reposent toutes deux sur les extensions **Service-for-User** :

- **S4U2self** : le service demande au KDC un ticket vers *lui-même* en tant qu'utilisateur arbitraire (sans intervention de l'utilisateur). Cela produit un ticket d'impersonation.
- **S4U2proxy** : le service utilise ce ticket pour demander un ticket vers le service *back-end* en tant que l'utilisateur.

Le fait que le ticket obtenu soit **transférable** (forwardable, et donc utilisable ensuite) dépend de la transition de protocole et des flags de compte, ce qui constitue le cœur de ce que chaque abus peut atteindre.

## Pages

- **[Délégation non contrainte](unconstrained.md)** : capture de TGT, y compris par coercition.
- **[Délégation contrainte](constrained.md)** : abus de S4U2self/S4U2proxy et de la transition de protocole.
- **[Délégation contrainte basée sur les ressources](resource-based-constrained.md)** : écriture de l'attribut de délégation pour usurper une identité.

## Références

- [Elad Shamir: Wagging the Dog (delegation primitives)](https://shenaniganslabs.io/2019/01/28/Wagging-the-Dog.html)
- [The Hacker Recipes: Kerberos delegations](https://www.thehacker.recipes/ad/movement/kerberos/delegations)
- [GhostPack Rubeus (s4u)](https://github.com/GhostPack/Rubeus)
