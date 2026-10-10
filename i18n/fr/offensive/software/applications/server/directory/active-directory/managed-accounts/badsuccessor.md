---
title: "BadSuccessor : abus de la migration dMSA sur Server 2025"
order: 3
description: "Escalader vers n'importe quel compte, y compris Domain Admin, sur un domaine Windows Server 2025 en créant ou en modifiant un Managed Service Account délégué et en définissant ses attributs de migration pour que le KDC construise son PAC à partir des SID d'un compte remplacé."
keywords:
  - BadSuccessor
  - dMSA
  - msDS-ManagedAccountPrecededByLink
  - msDS-DelegatedMSAState
  - Server 2025
---

# BadSuccessor

Windows Server 2025 ajoute les **delegated Managed Service Accounts (dMSA)** et une méthode prise en charge pour **migrer** un compte de service hérité vers un dMSA. La migration s'exprime dans deux attributs, et le KDC leur fait confiance sans vérifier qu'une véritable migration a eu lieu : si le **`msDS-ManagedAccountPrecededByLink`** d'un dMSA pointe vers un compte et que **`msDS-DelegatedMSAState`** vaut `2` (« migration terminée »), le KDC construit le PAC du dMSA à partir des **SID du compte remplacé**. Ainsi, quiconque peut définir ces deux attributs sur un dMSA qu'il contrôle peut s'authentifier en tant que ce dMSA et recevoir tous les privilèges de **n'importe quel** compte qu'il nomme, jusqu'à Domain Admin. C'est **BadSuccessor** (Akamai, 2025).

## Pourquoi c'est si dangereux

- Cela fonctionne avec la **configuration par défaut** d'un domaine Server 2025 et ne **requiert pas** que le domaine utilise de dMSA du tout ; la simple présence de la fonctionnalité (un seul DC Server 2025) suffit.
- Dans sa forme originale (avant correctif), le seul prérequis est la capacité de **créer un dMSA** (`CreateChild` pour la classe `msDS-DelegatedManagedServiceAccount` sur une OU). Les droits de création sur une OU sont bien plus courants et bien moins surveillés que Domain Admin, et un tel point d'ancrage peut cibler **n'importe quel** compte du domaine.
- Il n'y a aucune exigence d'être « déjà privilégié », il s'agit donc d'une vraie escalade du bas vers le haut, et pas seulement de persistance.

**Le niveau de correctif compte.** Le déroulé ci-dessous est la technique originale à sens unique et fonctionne sur les DC Server 2025 **non corrigés** (builds antérieurs à août 2025). Sur les DC corrigés, le KDC rejette un `PrecededByLink` à sens unique, et la variante post-correctif **BetterSuccessor** requiert en plus un **contrôle en écriture sur le compte cible** pour établir l'appariement mutuel, de sorte que le simple chemin « création seule » ne suffit plus là. Confirmez le build du DC avant de vous y fier.

## L'attaque

```bash
# Linux: bloodyAD automates create + weaponize (name takes NO trailing '$'; it is appended)
bloodyAD --host <dc> -d example.local -u user -p pass add badSuccessor 'evil_dmsa'

# or the attributes by hand on a dMSA you created
bloodyAD ... set object 'evil_dmsa$' msDS-ManagedAccountPrecededByLink -v 'CN=Administrator,CN=Users,DC=example,DC=local'
bloodyAD ... set object 'evil_dmsa$' msDS-DelegatedMSAState -v 2
```

```text
# Windows: SharpSuccessor uses a user's CreateChild right to create and weaponize the dMSA
SharpSuccessor.exe add /path:"OU=..." /account:evil_dmsa /name:evil_dmsa /impersonate:Administrator
```

Ensuite, **authentifiez-vous en tant que dMSA** : son mot de passe géré se récupère de la même façon que celui d'un gMSA, et l'outillage demande un TGT pour lui. Le PAC du ticket renvoyé porte les SID du compte remplacé, vous agissez donc en tant que ce compte immédiatement (demande de tickets de service, DCSync en tant que DA, etc.).

## Notes d'exploitation

- Nommez un compte remplacé **à forte valeur** (un Domain Admin, ou l'Administrator intégré) pour sauter directement à la compromission du domaine ; le KDC en copie les SID en bloc.
- Cherchez les OU où votre principal possède `CreateChild` (ou où un `msDS-DelegatedManagedServiceAccount` peut être créé) lors de l'[énumération des ACL](../dacl/acl-enumeration.md) ; c'est le verrou, et il est fréquemment ouvert via la délégation d'OU.
- Sur un DC corrigé, basculez vers la variante **BetterSuccessor** : elle escalade toujours, mais nécessite un contrôle en écriture sur le compte cible (pour poser le lien réciproque) plutôt que de simples droits de création ; considérez donc un domaine Server 2025 comme exposé et testez le comportement actuel par rapport à son niveau de correctif.
- Un dMSA créé et ses liens sont durables jusqu'à suppression, nettoyez donc le dMSA ensuite, sauf si vous souhaitez le conserver comme point d'ancrage.

## Outils

- **bloodyAD** (`add badSuccessor`) : créer et armer un dMSA depuis Linux.
- **SharpSuccessor** : PoC Windows qui abuse d'un droit `CreateChild` de bout en bout.
- **Rubeus / Impacket** : s'authentifier en tant que dMSA et obtenir le TGT dont le PAC porte les SID remplacés.

## Références

- [Akamai: BadSuccessor, abusing dMSA for privilege escalation](https://www.akamai.com/blog/security-research/abusing-dmsa-for-privilege-escalation-in-active-directory)
- [Akamai: analyzing the BadSuccessor patch](https://www.akamai.com/blog/security-research/badsuccessor-is-dead-analyzing-badsuccessor-patch)
- [AlteredSecurity: BetterSuccessor (dMSA abuse after the fix)](https://www.alteredsecurity.com/post/bettersuccessor-still-abusing-dmsa-for-privilege-escalation-badsuccessor-after-patch)
- [Microsoft: Delegated Managed Service Accounts overview](https://learn.microsoft.com/en-us/windows-server/identity/ad-ds/manage/delegated-managed-service-accounts/delegated-managed-service-accounts-overview)
