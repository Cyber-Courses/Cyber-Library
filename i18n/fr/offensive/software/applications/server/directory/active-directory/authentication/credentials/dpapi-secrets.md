---
order: 3
title: "Secrets DPAPI : identifiants de navigateur, de coffre-fort et d'applications"
description: "Récupérer les identifiants protégés par la Windows Data Protection API : mots de passe enregistrés dans les navigateurs, coffres du Gestionnaire d'identification Windows, identifiants RDP et de tâches planifiées, en déchiffrant les blobs DPAPI avec la clé maîtresse de l'utilisateur ou du domaine."
keywords:
  - DPAPI
  - clé maîtresse
  - credential manager
  - mots de passe de navigateur
  - dpapi backup key
---

# Secrets DPAPI

La Data Protection API (DPAPI) est la manière dont Windows chiffre les secrets par utilisateur au repos : mots de passe enregistrés dans les navigateurs, entrées du Gestionnaire d'identification Windows, identifiants RDP enregistrés, mots de passe de tâches planifiées, et secrets d'applications. Chacun est un blob DPAPI déchiffrable uniquement avec la **clé maîtresse** de l'utilisateur, elle-même dérivée du mot de passe de l'utilisateur (ou récupérable par le domaine). Atteindre ces secrets convertit un point d'ancrage en un trésor d'autres mots de passe de l'utilisateur.

## La chaîne de déchiffrement

Un blob DPAPI est déchiffré avec une clé maîtresse ; la clé maîtresse est déverrouillée de l'une des trois manières suivantes :

- **Le mot de passe ou le hash NTLM de l'utilisateur** (vous l'avez, ou vous vous exécutez en tant que lui).
- **La clé maîtresse déjà mise en cache en mémoire** (Mimikatz `sekurlsa::dpapi` extrait les clés maîtresses déchiffrées de LSASS).
- **La clé de sauvegarde DPAPI du domaine**, une clé privée à l'échelle de la forêt détenue sur les contrôleurs de domaine. Avec elle, un administrateur de domaine peut déchiffrer les secrets DPAPI de **n'importe quel** utilisateur du domaine, hors ligne et pour toujours.

```
# Extract the domain backup key once (Domain Admin), then decrypt anyone's blobs
mimikatz: lsadump::backupkeys /system:<dc> /export
dpapi.py backupkeys -t EXAMPLE/admin:password@<dc>
```

## Récupérer les secrets

```
# Mimikatz on-host: master keys then the vaulted/credential blobs
sekurlsa::dpapi
dpapi::cred /in:"%APPDATA%\Microsoft\Credentials\<guid>"
vault::cred /patch

# Impacket, with a recovered master key
dpapi.py masterkey -file <masterkey> -sid <user-sid> -password <user-pass>
dpapi.py credential -file <cred-blob> -key <decrypted-masterkey>
```

Les magasins d'identifiants des navigateurs (Chrome/Edge `Login Data`, cookies) sont protégés par DPAPI sur les versions plus anciennes ; les Chromium récents ajoutent un chiffrement lié à l'application (app-bound), un navigateur actuel nécessite donc une étape supplémentaire au-delà de la clé maîtresse DPAPI.

## Notes d'exploitation

- La clé de sauvegarde du domaine est la cible de grande valeur : une seule extraction (nécessite un administrateur de domaine ou le DC) permet en permanence de déchiffrer les secrets DPAPI de chaque utilisateur du domaine, c'est donc à la fois une primitive de collecte d'identifiants et de persistance.
- DPAPI fournit fréquemment des identifiants pour *d'autres* systèmes (RDP enregistré, VPN, connexions d'applications) qui ne sont pas autrement récupérables à partir de hash, élargissant le déplacement latéral au-delà de l'ensemble des comptes AD.
- Les identifiants de tâches planifiées et de services stockés via le Gestionnaire d'identification appartiennent souvent à des comptes d'automatisation privilégiés.

## Outils

- **Mimikatz** (`sekurlsa::dpapi`, `dpapi::cred`, `lsadump::backupkeys`) : déchiffrement sur l'hôte et export de la clé de sauvegarde.
- **Impacket dpapi.py** : déchiffrement hors ligne de clés maîtresses et de blobs, extraction de la clé de sauvegarde.
- **SharpDPAPI / DonPAPI** : collecte et déchiffrement automatisés sur plusieurs hôtes.

## Références

- [DonPAPI (login-securite): automated DPAPI looting](https://github.com/login-securite/DonPAPI)
- [SharpDPAPI (GhostPack)](https://github.com/GhostPack/SharpDPAPI)
- [Impacket dpapi.py (fortra)](https://github.com/fortra/impacket)
- [Microsoft: Windows Data Protection (DPAPI)](https://learn.microsoft.com/en-us/previous-versions/ms995355%28v=msdn.10%29)
