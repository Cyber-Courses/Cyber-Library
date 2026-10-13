---
title: "Déploiement d'applications SCCM : exécuter du code en tant que SYSTEM sur tout le parc"
order: 4
description: "Utiliser les fonctionnalités de déploiement propres à Configuration Manager, une fois que vous détenez le rôle Full Administrator ou des droits équivalents, pour exécuter des applications, des scripts et des requêtes CMPivot en tant que SYSTEM sur des collections d'appareils ciblées, transformant SCCM en une plateforme d'exécution de masse intégrée."
keywords:
  - SCCM application deployment
  - CMPivot
  - run scripts
  - device collection
  - SYSTEM execution
---

# Déploiement d'applications

C'est l'aboutissement de la [prise de contrôle du site](site-takeover.md) : SCCM existe pour distribuer des logiciels aux clients, donc un Full Administrator (ou toute personne disposant des bons droits de déploiement) peut pointer cette machinerie vers des cibles choisies et exécuter du code **en tant que SYSTEM** sur celles-ci. Aucun exploit n'est en jeu ; vous utilisez le produit tel qu'il a été conçu, contre le parc.

## Déployer vers une collection

```bash
# SharpSCCM: deploy to a device or a named collection; -s runs as SYSTEM (default is the logged-on user)
SharpSCCM.exe exec -d <device> -s -p "C:\Windows\System32\cmd.exe /c <payload>"
SharpSCCM.exe exec -n <collection-name> -s -p "<command>"
```

Une **device collection** peut être un seul hôte ou le parc tout entier, donc la même action passe à l'échelle d'une cible unique à chaque machine gérée.

## Run Scripts et CMPivot

Au-delà des déploiements d'applications complets, la fonctionnalité **Run Scripts** pousse du PowerShell vers les clients, et **CMPivot** exécute des requêtes en direct sur une collection. Les deux s'exécutent sur le client et sont plus rapides et plus discrets que l'empaquetage d'une application :

```text
# CMPivot runs a live query across a collection (recon or execution primitive)
# Run Scripts pushes approved PowerShell to selected devices as SYSTEM
```

## Notes d'exploitation

- Le déploiement s'exécute dans le contexte de l'agent client, qui est **SYSTEM**, donc cela équivaut à un accès administrateur local sur chaque cible sans toucher à leurs identifiants.
- **SharpSCCM `exec`** peut aussi forcer un client à s'authentifier auprès de vous (une primitive de coercition) plutôt que d'exécuter une charge utile, ce qui se rebranche sur le [relais](../../directory/active-directory/authentication/ntlm/relay.md).
- La portée de la collection constitue le rayon d'impact : déployer vers une collection large revient à exécuter du code à l'échelle du parc, donc définissez la portée de façon délibérée lors d'une mission.
- CMPivot est un outil de **reconnaissance** utile même avant une prise de contrôle complète si vous détenez des droits de lecture, inventoriant chaque client en direct.

## Outils

- **SharpSCCM** (`exec`) : déploiement d'applications et de commandes, et coercition de client.
- **Console native / Run Scripts / CMPivot** : déploiement et requêtes en direct une fois que vous détenez des droits administratifs.

## Références

- [SpecterOps: Misconfiguration Manager, EXEC](https://github.com/subat0mik/Misconfiguration-Manager/tree/main/attack-techniques/EXEC)
- [SharpSCCM (Mayyhem)](https://github.com/Mayyhem/SharpSCCM)
- [HTTP418InfoSec: offensive SCCM summary](https://http418infosec.com/offensive-sccm-summary)
