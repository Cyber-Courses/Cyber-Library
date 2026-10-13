---
title: "Windows : attaquer la plateforme hôte Windows"
description: "Techniques offensives contre un hôte Windows : élévation de privilèges locale via la mauvaise configuration des services, du registre et des jetons, accès aux identifiants depuis LSASS, SAM et les magasins DPAPI, contournement de l'UAC et des niveaux d'intégrité, persistance via les mécanismes de démarrage et de planification de Windows, et exploitation du noyau et des pilotes."
keywords:
  - élévation de privilèges Windows
  - LSASS
  - DPAPI
  - contournement de l'UAC
  - usurpation de jeton
---

# Windows

Windows est la plateforme de postes de travail et de serveurs d'entreprise dominante, et sa surface d'attaque locale est la plus riche des trois systèmes d'exploitation. Une fois que du code s'exécute en tant qu'utilisateur ordinaire, le travail consiste à s'élever jusqu'à `SYSTEM`, à récolter les identifiants et secrets que la machine met en cache, et à établir des points d'ancrage qui survivent à un redémarrage, le tout avant, ou à la place, d'atteindre le domaine Active Directory environnant.

## La surface locale

- **Élévation de privilèges** : chemins de services non entre guillemets ou à permissions faibles, mauvaise configuration des services et des tâches, privilèges exploitables (`SeImpersonate`, `SeBackup`, `SeDebug` et les attaques de jetons de la famille potato), détournement de DLL et abus de l'ordre de recherche, Always-Install-Elevated, et pilotes tiers et signés vulnérables.
- **Accès aux identifiants** : extraction de LSASS pour le clair et les hashes, les ruches SAM et SECURITY, les identifiants de domaine en cache, les secrets protégés par DPAPI (navigateur, gestionnaire d'identifiants), et les jetons pour l'usurpation.
- **Intégrité et UAC** : passer d'un niveau d'intégrité à un autre et les nombreux contournements du contrôle de compte d'utilisateur qui élèvent sans invite.
- **Persistance** : clés Run, services, tâches planifiées, abonnements aux événements WMI, détournement COM, et dossiers de démarrage.
- **Noyau** : exploitation locale du noyau et des pilotes pour l'étape finale vers `SYSTEM` ou pour déjouer les protections.

## Jonctions

Active Directory (authentification de domaine, Kerberos, NTLM, AD CS) est un domaine distinct dans [Directory](../../applications/server/directory/index.md) ; il est atteint depuis un point d'ancrage Windows mais attaqué en tant qu'annuaire. Ce domaine porte sur l'hôte Windows autonome.

## Références

- [MITRE ATT&CK: Windows privilege escalation](https://attack.mitre.org/tactics/TA0004/)
- [The Hacker Recipes: Windows](https://www.thehacker.recipes/)
