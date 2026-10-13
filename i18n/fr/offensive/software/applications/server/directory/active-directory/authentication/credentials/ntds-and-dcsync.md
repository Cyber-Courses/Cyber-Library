---
order: 4
title: "NTDS et DCSync : extraire le hash de chaque compte du domaine"
description: "Obtenir les hashs NTLM de tous les comptes du domaine, y compris krbtgt, en extrayant la base NTDS.dit sur un contrôleur de domaine ou en abusant de la réplication d'annuaire (DCSync) depuis n'importe quel hôte détenant le droit de réplication."
keywords:
  - NTDS.dit
  - DCSync
  - DRSUAPI
  - krbtgt
  - hashs de domaine
---

# NTDS et DCSync

La base `NTDS.dit` présente sur chaque contrôleur de domaine contient le hash NTLM de **chaque** compte du domaine, y compris le compte `krbtgt` dont le hash signe tous les tickets Kerberos. L'obtenir équivaut concrètement à une compromission complète du domaine : cela permet le pass-the-hash en tant que n'importe quel utilisateur et la forge de golden tickets. Il existe deux façons de l'obtenir, l'une nécessitant un accès à un DC et l'autre uniquement une permission de réplication spécifique.

**Filiation.** Extraire NTDS consistait autrefois à récupérer le fichier de la base de données via un cliché instantané de volume (volume shadow copy) ou un média d'installation IFM sur le DC. DCSync, publié en 2015, a remplacé cela par la réplication d'annuaire via DRSUAPI, ne nécessitant que les droits de réplication et ne laissant aucun artefact de fichier ; il reste la méthode standard de collecte après l'obtention des droits domain admin, et le parent de la variante DCShadow sur le chemin de réplication. (ZeroLogon est une faille Netlogon distincte qui réinitialise le compte du DC pour ensuite alimenter DCSync, ce n'est pas une variante de réplication de celui-ci.)

## DCSync (abus de réplication)

Un contrôleur de domaine réplique les modifications de l'annuaire, secrets compris, via le protocole **DRSUAPI**. Tout principal auquel sont accordés les droits étendus **`DS-Replication-Get-Changes`** et **`DS-Replication-Get-Changes-All`** peut *demander* à un DC de lui répliquer les secrets des comptes, ce qui ressemble à du trafic DC-à-DC normal et ne nécessite aucun code sur le DC. C'est **DCSync** :

```bash
# Impacket: pull one account, or the whole domain
secretsdump.py -just-dc-user krbtgt 'EXAMPLE/admin:password@<dc>'
secretsdump.py -just-dc 'EXAMPLE/admin:password@<dc>'

# NetExec
nxc smb <dc> -u admin -p pass --ntds

# Mimikatz, on-host
lsadump::dcsync /domain:example.local /user:krbtgt
```

Les Domain Admins, Enterprise Admins et contrôleurs de domaine détiennent ces droits par défaut, mais la vraie puissance de l'attaque réside dans le cas où un principal **non-administrateur** se les est vu accorder (directement ou via une DACL modifiable), transformant un compte ordinaire en extracteur de secrets du domaine.

## Extraire NTDS.dit directement

Avec un accès à un DC (ou à une sauvegarde), extrayez la base de données et la ruche SYSTEM qui contient sa clé de chiffrement :

```
# Create a shadow copy to read the locked database, then parse offline
vssadmin create shadow /for=C:
copy \\?\GLOBALROOT\...\Windows\NTDS\NTDS.dit  &  reg save HKLM\SYSTEM system.save
secretsdump.py -ntds NTDS.dit -system system.save LOCAL

# ntdsutil "IFM" media creation produces the same files
```

## Notes d'exploitation

- Le hash `krbtgt` est le joyau de la couronne : il signe chaque ticket, donc le posséder permet la forge de golden tickets et une persistance à l'échelle du domaine. Extrayez-le spécifiquement même lorsqu'un dump complet est bruyant.
- Préférez DCSync à l'accès au système de fichiers du DC ; c'est plus discret et cela ne nécessite que le droit de réplication, ce qui est précisément pourquoi la recherche de principaux non-DA disposant de ce droit (voir [énumération des ACL](../../dacl/acl-enumeration.md)) en vaut la peine.
- Le dump inclut les hashs historiques et, avec `-just-dc`, les clés AES Kerberos ; conservez les clés AES pour le pass-the-key et la forge de tickets.

## Outils

- **secretsdump.py** (Impacket) : DCSync (`-just-dc`) et analyse hors ligne de NTDS.
- **NetExec (nxc) --ntds** : dump des hashs du domaine.
- **Mimikatz** (`lsadump::dcsync`) : pull de réplication exécuté sur l'hôte.

## Références

- [adsecurity: Mimikatz DCSync usage, exploitation, detection](https://adsecurity.org/?p=1729)
- [The Hacker Recipes: DCSync](https://www.thehacker.recipes/ad/movement/credentials/dumping/dcsync)
- [Impacket secretsdump (fortra)](https://github.com/fortra/impacket)
- [NetExec: domain NTDS dump](https://github.com/Pennyw0rth/NetExec)
