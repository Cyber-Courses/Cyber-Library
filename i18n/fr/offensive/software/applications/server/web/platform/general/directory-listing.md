---
order: 4
title: "Listing de répertoire : autoindex et navigation de répertoire comme source d'énumération"
description: "Utiliser le listing de répertoire du serveur (autoindex Apache, autoindex nginx, navigation de répertoire IIS) pour énumérer des fichiers, découvrir du contenu non référencé et localiser sauvegardes, uploads et configuration."
keywords:
  - listing de répertoire
  - autoindex
  - navigation de répertoire
  - indexes
  - énumération
---

# Listing de répertoire

Quand un répertoire n'a pas de fichier d'index et que le serveur est configuré pour lister son contenu, demander le répertoire renvoie un index navigable de tous les fichiers qu'il contient. Cela remet à l'attaquant les noms exacts de fichiers qu'il devrait sinon deviner : sauvegardes, uploads, journaux, configuration et scripts non référencés.

## Le reconnaître

Une URL de répertoire renvoyant un index HTML intitulé « Index of /... » (Apache/nginx) ou un tableau de fichiers simple (IIS) est de la navigation de répertoire :

```
GET /uploads/ HTTP/1.1
GET /backup/  HTTP/1.1
GET /.git/    HTTP/1.1     # listing also makes VCS dumping trivial
```

`Options +Indexes` d'Apache, `autoindex on;` de nginx et la navigation de répertoire d'IIS produisent chacun ce résultat. C'est souvent activé sur un sous-répertoire (uploads, assets, exports) même quand la racine a un fichier d'index.

## Pourquoi c'est utile

- **Des noms impossibles à deviner** : un listing de `/uploads/` révèle chaque nom de fichier téléversé, exposant les documents d'autres utilisateurs et le chemin de tout webshell déposé.
- **Sauvegardes et dumps** : un listing transforme la devinette des [fichiers de sauvegarde](backup-and-temporary-files.md) en une simple lecture de répertoire.
- **Horodatages et tailles** : l'index révèle les dates de modification et les tailles, utiles pour repérer les fichiers récemment modifiés et les grosses archives/dumps.
- **Cartographie récursive** : parcourez chaque sous-répertoire listé pour bâtir une carte complète des fichiers sans wordlist.

## Exploitation

- Énumérez directement les répertoires susceptibles d'être listés (`/uploads`, `/files`, `/backup`, `/export`, `/static`, `/tmp`, `/logs`, `/.git`).
- Miroitez récursivement une arborescence listée :

  ```bash
  wget -r -np -nH --reject "index.html*" https://target/uploads/
  ```

- Triez l'index par date/taille (Apache prend en charge `?C=M;O=D`) pour faire remonter en premier les fichiers les plus récents ou les plus volumineux, puis récupérez source, dumps et secrets.

## Outils

- **wget -r** / **curl** ; navigateur pour un triage rapide.

## Références

- Apache httpd : mod_autoindex, Options Indexes
- nginx : ngx_http_autoindex_module
