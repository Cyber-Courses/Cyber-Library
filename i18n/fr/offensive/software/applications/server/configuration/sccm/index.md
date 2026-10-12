---
title: "SCCM / MECM : attaquer Configuration Manager"
description: "La surface offensive de Microsoft Configuration Manager (SCCM/MECM) : repérer les systèmes de site dans AD, récolter les identifiants qu'il stocke, prendre le contrôle du site via le relais NTLM et détourner le déploiement d'applications et de scripts pour exécuter du code en tant que SYSTEM sur chaque appareil géré."
keywords:
  - SCCM
  - MECM
  - Configuration Manager
  - site takeover
  - SharpSCCM
---

# SCCM / MECM

Microsoft Configuration Manager (SCCM, renommé MECM) gère les logiciels, les mises à jour et les paramètres de la plupart des grands parcs Windows. Un agent s'exécute en tant que **SYSTEM sur chaque client géré**, un **serveur de site** central l'orchestre, et l'ensemble est enregistré dans Active Directory. Cela en fait, selon les mots des chercheurs qui l'ont cartographié, un **framework de C2 d'entreprise** déjà installé dans l'environnement : prenez-en le contrôle et vous pouvez exécuter du code en tant que SYSTEM partout, ou extraire les identifiants qu'il détient.

## Comment la surface d'attaque s'est développée

La compréhension offensive de SCCM a évolué par étapes, et chaque étape reste utile selon la configuration de l'environnement :

- **2022, les identifiants d'abord** : le « gain facile » d'origine consistait à extraire le **Network Access Account** depuis la politique client, un identifiant de domaine remis à toute machine qui le demandait. Le démarrage PXE offrait une autre voie non authentifiée vers les mêmes secrets.
- **La réponse de Microsoft** : Enhanced HTTP et l'abandon du NAA ont réduit cette voie, donc l'attention s'est déplacée vers le plan de contrôle.
- **À partir de 2023, la prise de contrôle** : le **relais** NTLM contre le serveur de site et le SMS Provider, aboutissant à la base de données du site, accorde le rôle **Full Administrator**, ce qui équivaut à une exécution au niveau SYSTEM sur chaque client. C'est le centre de gravité actuel, catalogué par Misconfiguration Manager de SpecterOps (RECON, CRED, TAKEOVER, EXEC, ELEVATE).

## Pages

- **[Reconnaissance](reconnaissance.md)** : localiser les serveurs de site, les management points et les distribution points depuis AD.
- **[Récolte d'identifiants](credential-harvesting.md)** : Network Access Account, média PXE et secrets de politique.
- **[Prise de contrôle du site](site-takeover.md)** : relais NTLM vers la base de données du site ou le SMS Provider pour obtenir Full Administrator.
- **[Déploiement d'applications](application-deployment.md)** : exécuter du code en tant que SYSTEM sur l'ensemble des collections d'appareils.

## Références

- [SpecterOps: Misconfiguration Manager](https://github.com/subat0mik/Misconfiguration-Manager)
- [SharpSCCM (Mayyhem)](https://github.com/Mayyhem/SharpSCCM)
- [DEF CON 32: offensive SCCM, abusing Microsoft's C2 framework](https://infocondb.org/con/def-con/def-con-32/offensive-sccm-abusing-microsofts-c2-framework)
