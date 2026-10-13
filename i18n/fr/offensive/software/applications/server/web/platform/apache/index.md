---
order: 2
title: "Mauvaise configuration d'Apache httpd : handlers, alias et négociation de contenu"
description: "Mauvaises configurations de plateforme propres à Apache : mappages de handlers et de types trop larges, abus de surcharge .htaccess, traversée via Alias et mod_rewrite, et divulgation par négociation de contenu MultiViews."
keywords:
  - mauvaise configuration apache
  - AddHandler
  - htaccess
  - mod_rewrite
  - MultiViews
---

# Apache

La flexibilité d'Apache httpd (surcharges `.htaccess` par répertoire, handlers basés sur l'extension et jeu de modules riche) est aussi sa surface de mauvaise configuration. Identifiez-le via l'en-tête `Server: Apache`, les pages d'erreur par défaut et le comportement `.htaccess`/`mod_*`, puis déroulez cette checklist.

## Ce qu'il faut vérifier

- **Mappage des handlers et des types** : un handler correspond-il à une extension interne (`shell.php.jpg`), le handler d'un répertoire est-il réglé pour exécuter les uploads, ou le handler PHP manque-t-il de sorte que les scripts reviennent comme source ?
- **Surcharges .htaccess** : lorsque `AllowOverride` est permissif et qu'un répertoire est accessible en écriture (uploads), un `.htaccess` fourni par l'attaquant modifie les handlers, les réécritures et l'authentification.
- **Alias et rewrite** : les règles `Alias`/`AliasMatch`/`mod_rewrite` construisent-elles des chemins du système de fichiers à partir de captures d'URL non contraintes, permettant une traversée vers le parent du répertoire mappé ?
- **MultiViews** : la négociation de contenu est-elle activée, permettant à un attaquant de demander `page` et à Apache de choisir `page.php`/`page.bak`, énumérant et divulguant des fichiers ?

## Pages

- **[Mappage des handlers et des types](handler-and-type-mapping.md)** : erreurs d'exécution et de divulgation de source avec `AddHandler`/`AddType`/`SetHandler`, abus de `.htaccess` et exécution d'upload à double extension.
- **[Traversée via Alias et rewrite](alias-and-rewrite-traversal.md)** : mappage de chemin `Alias`/`AliasMatch`/`mod_rewrite` qui s'échappe du répertoire prévu.
- **[MultiViews et négociation de contenu](multiviews-and-negotiation.md)** : abus de `mod_negotiation` pour énumérer et divulguer des fichiers.

## Références

- Documentation Apache httpd : mod_mime, mod_alias, mod_rewrite, mod_negotiation, AllowOverride
- OWASP WSTG : Testing for application platform configuration
