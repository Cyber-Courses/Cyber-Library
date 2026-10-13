---
order: 4
title: "Mauvaise configuration d'IIS : astuces de noms de fichiers NTFS, décodage, handlers et web.config"
description: "Mauvaises configurations de plateforme propres à IIS : flux de données alternatifs NTFS et énumération de noms courts, double décodage et traversée Unicode, mappages de handlers, et comportement et divulgation de web.config."
keywords:
  - mauvaise configuration iis
  - ::$DATA
  - nom court
  - double décodage
  - web.config
---

# IIS

IIS sur Windows/NTFS hérite des particularités du système de fichiers et d'un modèle de handlers-et-config qui produisent un ensemble de mauvaises configurations distinctif. Identifiez-le via `Server: Microsoft-IIS`, le cookie `ASP.NET_SessionId`, `X-AspNet-Version`/`X-Powered-By: ASP.NET` et les pages d'erreur par défaut, puis déroulez cette checklist.

## Ce qu'il faut vérifier

- **Astuces de noms de fichiers NTFS** : flux de données alternatifs `::$DATA`, points/espaces finaux et énumération des noms courts 8.3 qui divulguent la source ou révèlent des fichiers cachés.
- **Décodage** : double décodage et traversée Unicode à encodage trop long sur les IIS plus anciens, où le chemin est décodé plus d'une fois.
- **Handlers et web.config** : mappages de handlers qui exécutent les uploads ou divulguent la source, et comportement de `web.config` (config par répertoire, exposition de source et abus d'upload-en-tant-que-config).

## Pages

- **[Astuces de noms de fichiers NTFS](ntfs-filename-tricks.md)** : `::$DATA`, point/espace final et énumération de noms courts.
- **[Double décodage et traversée Unicode](double-decode-and-unicode-traversal.md)** : décodage multi-étapes et trop long qui fait passer en douce la traversée.
- **[Handlers et web.config](handlers-and-web-config.md)** : mappages de handlers, divulgation de source et abus de `web.config`.

## Références

- Documentation Microsoft IIS ; flux de données alternatifs NTFS et nommage 8.3
- Soroush Dalili : recherche IIS short-name et ::$DATA
