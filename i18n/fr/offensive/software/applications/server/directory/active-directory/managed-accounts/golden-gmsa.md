---
order: 4
title: "Golden gMSA : forger hors ligne les mots de passe des comptes de service gérés"
description: "Extraire la clé racine KDS pour calculer hors ligne et pour toujours le mot de passe de n'importe quel group Managed Service Account, l'attaque Golden gMSA, un équivalent gMSA du golden ticket sans aucun moyen de renouveler le secret sous-jacent."
keywords:
  - golden gMSA
  - clé racine KDS
  - GoldenGMSA
  - msDS-ManagedPasswordID
  - persistance de domaine
---

# Golden gMSA

Un mot de passe [gMSA](gmsa.md) n'est pas stocké ; il est **calculé** à partir de la **clé racine KDS** de la forêt plus des attributs du compte (`msDS-ManagedPasswordID`). Normalement, le DC effectue ce calcul et seuls les hôtes autorisés récupèrent le résultat. L'attaque **Golden gMSA** s'empare de la clé racine KDS elle-même : extrayez-la une fois, et vous pouvez calculer les mots de passe actuels et futurs de **chaque** gMSA lié à cette clé, **hors ligne**, sans jamais contacter un DC. C'est l'analogue gMSA du [golden ticket](../authentication/kerberos/forged-tickets.md), et il est pire sur un point : contrairement à `krbtgt`, la **clé racine KDS ne peut pas être renouvelée**, de sorte qu'une clé compromise constitue une exposition permanente.

## L'attaque

Extraire la clé racine KDS nécessite un privilège élevé (Domain Admin, ou SYSTEM sur un DC) une fois ; après quoi le calcul est non privilégié et hors ligne :

```bash
# Semperis GoldenGMSA: read the KDS root key (privileged, once)
GoldenGMSA.exe kdsinfo
# Enumerate gMSAs and their managed-password IDs
GoldenGMSA.exe gmsainfo
# Compute a gMSA's password fully offline: SID + its managed-password ID (from gmsainfo) + the root key
GoldenGMSA.exe compute --sid <gMSA-SID> --pwdid <managed-password-id> --kdskey <base64-kds-key>
```

La sortie est le mot de passe du gMSA (et donc son hash NT / ses clés Kerberos), utilisé avec [pass-the-hash](../authentication/ntlm/pass-the-hash.md) ou [overpass-the-hash](../authentication/kerberos/pass-the-key-and-overpass-the-hash.md).

## Pourquoi c'est durable

- La **clé racine KDS ne tourne jamais** : il n'y a pas d'équivalent d'un double reset de `krbtgt`, donc une fois la clé fuitée, chaque gMSA associé reste forgeable jusqu'à ce que chaque gMSA soit migré vers une nouvelle clé.
- Le calcul est **hors ligne**, il ne produit donc **aucun log d'authentification sur le DC**, contrairement à la récupération légitime de `msDS-ManagedPassword`.
- Il couvre **tous** les gMSA sous la clé à la fois, y compris les comptes de service privilégiés, faisant d'une seule extraction de clé une skeleton key de comptes de service à l'échelle de la forêt.

## Notes d'exploitation

- Il s'agit de persistance et de portée latérale, pas d'accès initial : vous avez besoin de la lecture privilégiée ponctuelle de la clé racine KDS (depuis le DC, ou depuis une sauvegarde/`ntds.dit`).
- L'équivalent delegated-MSA, **Golden dMSA**, applique la même idée aux dMSA de Server 2025 ; associez cela à la voie dMSA [BadSuccessor](badsuccessor.md) lorsque des dMSA sont présents.
- Visez les gMSA qui exécutent des services privilégiés ; un mot de passe gMSA forgé vaut autant que les droits du compte de service.

## Outils

- **GoldenGMSA** (Semperis) : extraire la clé racine KDS et calculer les mots de passe gMSA hors ligne.
- **GoldenDMSA** (Semperis) : la variante delegated-MSA pour Server 2025.

## Références

- [Semperis: introducing the Golden gMSA attack](https://www.semperis.com/blog/golden-gmsa-attack/)
- [GoldenGMSA (Semperis)](https://github.com/Semperis/GoldenGMSA)
- [GoldenDMSA (Semperis)](https://github.com/Semperis/GoldenDMSA)
