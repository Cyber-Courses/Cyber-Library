---
order: 2
title: "Collecte d'identifiants SCCM : NAA, PXE et secrets de stratégie"
description: "Extraction des identifiants que Configuration Manager transmet aux clients : le Network Access Account issu de la stratégie machine, les secrets des supports de démarrage PXE non authentifiés et les variables de séquence de tâches, transformant n'importe quel client ou n'importe quel compte d'ordinateur en identifiants de domaine."
keywords:
  - Network Access Account
  - NAA
  - PXE
  - stratégie SCCM
  - SharpSCCM
---

# Collecte d'identifiants

Configuration Manager doit fournir des identifiants aux clients pour qu'ils effectuent leurs tâches, et il les a historiquement stockés à un endroit où un client, ou toute personne capable de s'enregistrer comme tel, peut les récupérer et les déchiffrer. Ce fut la première classe d'attaque SCCM à être exploitée et elle fonctionne toujours partout où les paramètres hérités subsistent.

## Le Network Access Account

Le **Network Access Account (NAA)** est un compte de domaine que SCCM fournit aux clients afin qu'ils puissent accéder au contenu lorsque leur propre compte machine ne le peut pas. Il est transmis dans la **stratégie machine** et chiffré avec une clé que le client détient lui-même, de sorte que n'importe quelle machine (ou n'importe quel compte d'ordinateur enregistré) peut le déchiffrer pour retrouver un identifiant de domaine en clair :

```bash
# SharpSCCM: request machine policy and recover NAA credentials as SYSTEM on a client
SharpSCCM.exe get naa

# sccmhunter: register a computer, pull policy from the management point, decrypt the NAA
sccmhunter.py http -u user -p password -d example.local -dc-ip <dc> -auto
```

C'était le « gain facile » originel de SCCM. L'**Enhanced HTTP** de Microsoft et l'abandon du NAA le réduisent, donc son absence vous pousse vers la [prise de contrôle du site](site-takeover.md) ; sa présence constitue souvent un identifiant de domaine instantané.

## Supports de démarrage PXE

Un point de distribution configuré pour le **PXE** (déploiement de système d'exploitation) distribuera des supports de démarrage dont le paquet de variables peut contenir le NAA et les identifiants de séquence de tâches. Si le PXE n'est pas protégé par mot de passe, ou si le mot de passe est faible, ceci est **non authentifié** :

```bash
# PXEThief: pull and decrypt secrets from PXE boot media
python3 pxethief.py 2 <distribution-point-ip>
```

## Variables de séquence de tâches et de collection

Les séquences de tâches et les variables de collection intègrent fréquemment des identifiants de compte de service ou d'administrateur local pour l'imagerie et les installations ; dès que vous pouvez lire la stratégie ou que vous détenez le rôle Full Administrator, elles constituent un second trésor d'identifiants.

## Notes d'exploitation

- Les chemins NAA et PXE transforment **un seul client, ou simplement un compte d'ordinateur, en identifiant de domaine**, sans accès privilégié préalable.
- Même un NAA **désactivé** reste souvent récupérable à partir d'anciens blobs de stratégie, donc extrayez-le indépendamment de la configuration actuelle.
- Le PXE sans mot de passe robuste constitue un véritable point d'entrée non authentifié depuis le réseau, donc testez-le toujours lors de la [reconnaissance](reconnaissance.md).

## Outils

- **SharpSCCM** (`get naa`) : récupération des identifiants de stratégie.
- **sccmhunter** (module `http`) : enregistrement d'un appareil et extraction des secrets de stratégie.
- **PXEthief** : extraction et déchiffrement des secrets des supports PXE.

## Références

- [SpecterOps: Misconfiguration Manager, CRED](https://github.com/subat0mik/Misconfiguration-Manager/tree/main/attack-techniques/CRED)
- [GuidePoint: SCCM exploitation, compromising Network Access Accounts](https://www.guidepointsecurity.com/blog/sccm-exploitation-compromising-network-access-accounts/)
- [PXEthief (MWR / Christopher Panayi)](https://github.com/MWR-CyberSec/PXEThief)
