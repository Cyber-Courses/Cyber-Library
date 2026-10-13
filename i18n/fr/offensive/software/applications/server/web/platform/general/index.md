---
order: 1
title: "Exposition générale du serveur : source, secrets et détails de déploiement sur n'importe quel serveur"
description: "Expositions de plateforme indépendantes du produit qui s'appliquent quel que soit le serveur web : endpoints de statut, répertoires de gestion de version, fichiers de sauvegarde et temporaires, listings de répertoire et fichiers de configuration divulgués."
keywords:
  - divulgation d'information
  - server-status
  - exposition git
  - fichiers de sauvegarde
  - listing de répertoire
---

# Général

Avant même que le fingerprinting n'ait de l'importance, tout serveur peut être amené à fuiter. Cette section rassemble les expositions **indépendantes du produit** : elles dépendent de l'accessibilité de fichiers et d'endpoints, pas du serveur web utilisé. Vérifier celles-ci en premier est peu coûteux et à fort rendement, et évite les répétitions dans les sections par produit.

## Pourquoi c'est important

C'est de la reconnaissance qui paie immédiatement. Le code source récupéré révèle les sinks exacts d'injection et de désérialisation ; un `.env` ou un fichier de config divulgué livre les identifiants de base de données et d'API ainsi que les clés secrètes du framework (qui déverrouillent cookies signés et ViewState) ; une page `server-status` expose les requêtes d'autres utilisateurs et des chemins internes. Rien de tout cela ne nécessite de toucher au code applicatif.

## Pages

- **[Endpoints de statut et d'info](status-and-info-endpoints.md)** : `server-status`, `server-info`, `stub_status` de nginx et pages de statut PHP-FPM.
- **[Répertoires de gestion de version](version-control-directories.md)** : `.git`, `.svn` et `.hg` exposés, et reconstruction du code source à partir d'eux.
- **[Fichiers de sauvegarde et temporaires](backup-and-temporary-files.md)** : fichiers swap d'éditeur, `.bak`/`~`/`.old` et archives oubliées.
- **[Listing de répertoire](directory-listing.md)** : autoindex et navigation de répertoire comme source d'énumération.
- **[Config et dotfiles exposés](exposed-config-and-dotfiles.md)** : `.env`, fichiers de config, `.htaccess` et dotfiles de métadonnées.

## Références

- PortSwigger Web Security Academy : Information disclosure
- OWASP WSTG : Testing for information leakage
