---
title: "gMSA : lire les mots de passe des comptes de service gérés"
order: 1
description: "Récupérer le mot de passe d'un group Managed Service Account depuis Active Directory en lisant le blob msDS-ManagedPassword, autorisé aux principaux listés dans msDS-GroupMSAMembership, et dériver hors ligne l'empreinte NT et les clés AES du compte."
keywords:
  - gMSA
  - msDS-ManagedPassword
  - ReadGMSAPassword
  - gMSADumper
  - PrincipalsAllowedToRetrieveManagedPassword
---

# gMSA

Un group Managed Service Account a son mot de passe généré et renouvelé par le domaine et stocké dans l'annuaire, dans l'attribut **`msDS-ManagedPassword`** (un blob contenant le mot de passe actuel et le précédent). AD ne renvoie cet attribut qu'aux principaux listés dans le **`msDS-GroupMSAMembership`** du compte (`PrincipalsAllowedToRetrieveManagedPassword`). Donc si vous contrôlez un principal de cet ensemble, vous lisez le blob et dérivez l'empreinte NT et les clés Kerberos du compte directement depuis l'annuaire, sans aucun code sur aucun hôte. BloodHound le marque comme l'arête `ReadGMSAPassword`.

## Lire le mot de passe

```bash
# NetExec: dump all gMSA passwords you are allowed to read (uses LDAPS automatically)
nxc ldap <dc> -u user -p pass --gmsa
# authenticate with a machine account's hash where that host may read the gMSA
nxc ldap <dc> -u 'WEB01$' -H <nthash> -d example.local --gmsa

# gMSADumper: read msDS-ManagedPassword and print NTLM / Kerberos keys
gMSADumper.py -u user -p pass -d example.local -l <dc>

# bloodyAD: read the raw attribute
bloodyAD --host <dc> -d example.local -u user -p pass get object 'svc_gmsa$' --attr msDS-ManagedPassword
```

La sortie est l'empreinte NT du gMSA (et les clés AES), que vous utilisez avec [pass-the-hash](../authentication/ntlm/pass-the-hash.md) ou [overpass-the-hash](../authentication/kerberos/pass-the-key-and-overpass-the-hash.md).

## Entrer dans l'ensemble de lecture

L'arête dont vous avez besoin est l'accès en écriture au `msDS-GroupMSAMembership` du gMSA, ou l'appartenance à un groupe qui y figure déjà :

- Un `GenericWrite`/`GenericAll` ([DACL](../dacl/index.md)) sur le gMSA vous permet de vous ajouter à `PrincipalsAllowedToRetrieveManagedPassword`, puis de lire.
- La compromission d'un **hôte** déjà autorisé à exécuter le gMSA (son compte machine est dans l'ensemble) vous permet de lire avec ce compte machine.

## Notes d'exploitation

- Les mots de passe gMSA sont longs et aléatoires, ils ne se casseront donc pas ; la valeur réside dans l'**empreinte/les clés que vous lisez directement**, utilisées telles quelles.
- Un gMSA est fréquemment **privilégié** (il exécute un service doté de droits réels), donc son empreinte octroie souvent un accès significatif ou de nouvelles arêtes.
- Avec la **clé racine KDS** (lisible par les Domain Admins / sur un DC), vous pouvez calculer le mot de passe de n'importe quel gMSA **hors ligne** pour n'importe quel instant, ce qui constitue à la fois un raccourci d'escalade et une persistance gMSA.
- Le blob contient aussi le mot de passe précédent, utile si une rotation est survenue entre la reconnaissance et l'utilisation.

## Outils

- **NetExec (`nxc`) `--gmsa`** : lire depuis Linux tous les mots de passe gMSA autorisés.
- **gMSADumper.py** : lire et décoder vers les clés NTLM/Kerberos.
- **bloodyAD** (`get object --attr msDS-ManagedPassword`) : lecture brute de l'attribut, et `set object` sur `msDS-GroupMSAMembership` pour vous y ajouter.

## Références

- [SpecterOps: BloodHound ReadGMSAPassword edge](https://bloodhound.specterops.io/resources/edges/read-gmsa-password)
- [Microsoft (MS-ADTS): msDS-ManagedPassword](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-adts/9cd2fc5e-7305-4fb8-b233-2a60bc3eec68)
- [gMSADumper (micahvandeusen)](https://github.com/micahvandeusen/gMSADumper)
- [NetExec: dumping gMSA over LDAP](https://github.com/Pennyw0rth/NetExec-Wiki/blob/main/ldap-protocol/dump-gmsa.md)
