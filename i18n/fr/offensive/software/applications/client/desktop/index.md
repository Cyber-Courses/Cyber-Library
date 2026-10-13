---
title: "Desktop : attaquer les applications de bureau installées"
description: "Techniques offensives contre les applications de bureau : communication interprocessus locale et gestionnaires de protocoles personnalisés, canaux de mise à jour automatique non sécurisés, faiblesses des navigateurs embarqués et d'Electron qui atteignent le système d'exploitation, failles de désérialisation et d'analyse syntaxique, et secrets que l'application stocke sur le poste."
keywords:
  - desktop application security
  - Electron
  - protocol handler
  - insecure update
  - local IPC
---

# Desktop

Une application de bureau s'exécute avec les propres privilèges de l'utilisateur et dispose d'une portée locale directe : le système de fichiers, les autres processus et le système d'exploitation. Sa surface d'attaque est constituée de ce qu'elle expose localement aux autres logiciels, de la manière dont elle récupère les mises à jour et le contenu et de la confiance qu'elle leur accorde, et des secrets qu'elle conserve sur le disque. L'attaquant est souvent un autre processus local ou une page web capable d'atteindre l'application via un gestionnaire enregistré.

## La surface

- **IPC locale et gestionnaires de protocoles** : canaux nommés (named pipes), sockets locaux et gestionnaires de schémas d'URL personnalisés que d'autres logiciels locaux (ou une page web) peuvent invoquer pour piloter l'application.
- **Mises à jour non sécurisées** : canaux de mise à jour automatique qui récupèrent le contenu via un transport faible ou sans vérification de signature, transformant le composant de mise à jour en exécution de code.
- **Navigateurs embarqués et Electron** : applications construites sur des moteurs web où une isolation désactivée, une intégration Node exposée ou une navigation vers un contenu contrôlé par l'attaquant atteint le système d'exploitation.
- **Analyse syntaxique et désérialisation** : formats de fichiers et de messages que l'application ouvre, et les failles de sécurité mémoire ou de désérialisation lors de leur analyse.
- **Secrets locaux** : identifiants, jetons et clés que l'application stocke sur le poste.

## Références

- [Electron security guidelines](https://www.electronjs.org/docs/latest/tutorial/security)
- [OWASP Desktop App Security Testing](https://owasp.org/www-project-desktop-app-security-testing/)
