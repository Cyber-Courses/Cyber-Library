---
title: "LAPS : lire les mots de passe d'administrateur local depuis l'annuaire"
order: 2
description: "Récupérer les mots de passe d'administrateur local des machines que LAPS stocke dans Active Directory : l'attribut hérité en clair ms-Mcs-AdmPwd et les attributs Windows LAPS msLAPS-Password / msLAPS-EncryptedPassword, lus par LDAP avec une ACE suffisante."
keywords:
  - LAPS
  - ms-Mcs-AdmPwd
  - msLAPS-Password
  - ReadLAPSPassword
  - local administrator
---

# LAPS

Le Local Administrator Password Solution (LAPS) rend aléatoire le mot de passe d'administrateur local de chaque machine et le stocke **dans l'objet d'annuaire de l'ordinateur**. Pratique, et directement exploitable : un droit de lecture sur le bon attribut vous remet le mot de passe d'administrateur local de cet hôte. BloodHound le représente par l'arête `ReadLAPSPassword`. Il existe deux générations, stockées dans des attributs différents.

## LAPS hérité ou Windows LAPS

- **LAPS hérité** : le mot de passe est **en clair** dans **`ms-Mcs-AdmPwd`** (expiration dans `ms-Mcs-AdmPwdExpirationTime`). Tout principal disposant d'un accès en lecture à cet attribut obtient le mot de passe tel quel.
- **Windows LAPS** (intégré aux Windows actuels) : utilise **`msLAPS-Password`** (JSON en clair lorsque le chiffrement est désactivé) et **`msLAPS-EncryptedPassword`** (chiffré en DPAPI-NG pour un principal/groupe choisi), plus des attributs d'**historique** de mots de passe. La forme chiffrée exige que vous soyez (ou que vous atteigniez) le déchiffreur autorisé. BloodHound suit aussi **`SyncLAPSPassword`**, une arête au niveau du domaine permettant de récupérer des attributs confidentiels et filtrés RODC (y compris le `ms-Mcs-AdmPwd` hérité) via la synchronisation de l'annuaire, à distinguer de **`ReadLAPSPassword`**, qui est un accès en lecture directe sur un seul objet ordinateur.

## Le lire

```bash
# NetExec: pull LAPS passwords over LDAP (handles legacy and Windows LAPS attributes)
nxc ldap <dc> -u user -p pass -M laps
nxc ldap <dc> -u user -p pass -M laps -o COMPUTER='WKSTN-*'   # filter to hosts

# pyLAPS: get (and set) the LAPS attribute from Linux
pyLAPS.py --action get -u user -p pass -d example.local --dc-ip <dc>

# bloodyAD: raw attribute read
bloodyAD --host <dc> -d example.local -u user -p pass get object 'WKSTN01$' --attr ms-Mcs-AdmPwd
```

## Notes d'exploitation

- Un mot de passe LAPS récupéré donne **administrateur local sur ce seul hôte** (LAPS attribue à chaque machine un mot de passe unique), il s'agit donc de mouvement latéral une machine à la fois, pas d'une clé maîtresse : extrayez-en les secrets et pivotez.
- Qui peut lire LAPS est défini par la délégation de l'OU ; cherchez des droits de lecture trop larges (un groupe de support, ou `Authenticated Users` par mauvaise configuration) avec l'[énumération des ACL](../dacl/acl-enumeration.md).
- Les mots de passe **chiffrés** de Windows LAPS nécessitent le déchiffreur autorisé ; si vous n'avez que la lecture et non les droits de déchiffrement, le blob est inutile tant que vous n'atteignez pas ce principal.
- L'**historique** des mots de passe LAPS (Windows LAPS) peut exposer un mot de passe plus ancien encore valide ailleurs ou utile contre du matériel mis en cache.

## Outils

- **NetExec (`nxc`) `-M laps`** : lire par LDAP les attributs LAPS hérités et Windows LAPS.
- **pyLAPS / LAPSDumper** : get/set des attributs LAPS depuis Linux.
- **bloodyAD** (`get object --attr`) : lecture brute de `ms-Mcs-AdmPwd` / `msLAPS-Password`.

## Références

- [SpecterOps: BloodHound ReadLAPSPassword edge](https://bloodhound.specterops.io/resources/edges/read-laps-password)
- [Microsoft: Windows LAPS overview](https://learn.microsoft.com/en-us/windows-server/identity/laps/laps-overview)
- [Microsoft (MS-ADA2): ms-LAPS-Password attribute](https://learn.microsoft.com/en-us/openspecs/windows_protocols/ms-ada2/b2e01af2-3ff5-4c64-8ef3-d0d8a545945b)
- [NetExec LAPS module source](https://github.com/Pennyw0rth/NetExec/blob/main/nxc/modules/laps.py)
