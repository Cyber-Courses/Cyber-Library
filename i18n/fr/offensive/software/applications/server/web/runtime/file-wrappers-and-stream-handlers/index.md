---
order: 2
title: "Wrappers de fichiers et gestionnaires de flux : escalader les opérations de fichiers via les pseudo-protocoles du moteur"
description: "Comment les wrappers de flux du runtime (surtout les php://, phar://, data:// de PHP) transforment une lecture ou une inclusion de fichier en divulgation de source, désérialisation et exécution de code à distance."
keywords:
  - wrappers php
  - php://filter
  - phar
  - wrapper data
  - gestionnaire de flux
  - LFI vers RCE
---

# Wrappers de fichiers et gestionnaires de flux

Les runtimes de langage exposent des **wrappers de flux** : des pseudo-protocoles qui permettent aux fonctions de fichier ordinaires d'opérer sur des choses qui ne sont pas de simples fichiers. PHP est de loin la cible la plus riche, avec `php://`, `phar://`, `data://`, `zip://`, et d'autres. Lorsqu'une application passe une entrée influencée par l'attaquant à une fonction de fichier (`include`, `fopen`, `file_get_contents`, `getimagesize`, `copy`), ces wrappers transforment ce qui ressemble à une lecture de fichier en divulgation de source, désérialisation ou exécution de code. C'est un problème de couche runtime car la capacité provient du moteur, non de la logique propre à l'application.

## Pourquoi c'est important

Un bug de traitement de chemin qui ne serait autrement qu'une lecture de fichier local limitée devient bien plus lorsque les wrappers sont disponibles :

- `php://filter` lit et transforme le contenu des fichiers, en extrayant le code source, et des filtres chaînés peuvent synthétiser du PHP exécutable à partir d'aucun fichier.
- `phar://` désérialise les métadonnées d'archive dès qu'une fonction de fichier touche le chemin, atteignant l'injection d'objet sans appel à `unserialize()`.
- `data://` et `php://input` fournissent du contenu contrôlé par l'attaquant directement à un `include` (les deux requièrent `allow_url_include=On`) ; lorsque ce réglage est désactivé, les chaînes `php://filter` atteignent toujours l'exécution de code sans dépendance de configuration.

## Pages

- **[Wrapper php filter](php-filter-wrapper.md)** : lire la source et des fichiers arbitraires avec `php://filter` et les filtres de conversion.
- **[Chaînes de filtres vers RCE](filter-chains-to-rce.md)** : synthétiser un payload PHP purement par des filtres de conversion chaînés.
- **[Désérialisation phar](phar-deserialization.md)** : déclencher une injection d'objet via `phar://` lors des opérations de fichier.
- **[Wrappers data et input](data-and-input-wrappers.md)** : `data://`, `php://input` et `expect://` pour l'inclusion et l'exécution directes.

## Références

- PHP manual: supported protocols and wrappers
- PortSwigger Web Security Academy: File path traversal, LFI
