---
order: 1
title: "Astuces de noms de fichiers NTFS IIS : ::$DATA, points finaux et énumération de noms courts"
description: "Abus de la gestion des noms de fichiers NTFS sur IIS pour divulguer la source et énumérer des fichiers cachés : flux de données alternatifs (::$DATA), points et espaces finaux, et énumération différentielle des noms courts 8.3."
keywords:
  - ::$DATA
  - NTFS ADS
  - nom court 8.3
  - point final
  - divulgation de source IIS
---

# Astuces de noms de fichiers NTFS

NTFS accepte plusieurs formes de noms de fichiers qui résolvent vers le même fichier mais paraissent différentes à un handler ou à un filtre. Sur IIS, celles-ci divulguent le code source des scripts et énumèrent des fichiers qui ne sont liés nulle part.

## Flux de données alternatifs (::$DATA)

Chaque fichier NTFS possède un flux de données par défaut nommé `$DATA`. Ajouter `::$DATA` référence les mêmes octets mais change l'extension que voit le handler, si bien qu'un handler de script peut ne pas se déclencher et qu'IIS renvoie la **source brute** :

```
GET /default.aspx::$DATA HTTP/1.1      # ASPX source, not executed
GET /web.config::$DATA HTTP/1.1
GET /login.asp::$DATA HTTP/1.1
```

Une forme de flux apparentée est `:$I30:$INDEX_ALLOCATION` ajoutée à un nom de répertoire.

## Points et espaces finaux

Windows retire les points et espaces finaux à l'ouverture d'un fichier, mais un handler ou un filtre peut se baser sur la requête littérale. `app.aspx.` (point final) ou `app.aspx%20` ouvre `app.aspx` sur le disque tout en présentant un suffixe différent au handler, produisant une divulgation de source ou un contournement de handler :

```
GET /secret.aspx. HTTP/1.1
GET /secret.aspx%20 HTTP/1.1
```

## Énumération de noms courts 8.3

NTFS conserve les noms courts 8.3 hérités (`PROGRA~1`). IIS révèle leur existence via des réponses différentielles à un motif de tilde, permettant à un attaquant de récupérer caractère par caractère des noms de fichiers autrement inconnus :

```
GET /secr~1.asp*~ HTTP/1.1     # tilde-probe pattern used by short-name scanners
```

Un scanner de noms courts automatise cela pour reconstruire des noms partiels de fichiers de config, de sauvegarde et d'administration qui ne sont pas liés, que vous récupérez ensuite en entier (et via `::$DATA` pour la source).

## Exploitation

- Ajoutez `::$DATA` et un `.`/`%20` final aux scripts exécutables connus (`.aspx`, `.asp`, `.ashx`, `.asmx`, `.config`) pour récupérer source et secrets.
- Lancez un scanner de noms courts IIS pour énumérer les noms de fichiers cachés, puis demandez les noms complets et leur source `::$DATA`.
- Envoyez avec `curl --path-as-is` / Burp pour que les octets exacts atteignent IIS.

## Outils

- **IIS-ShortName-Scanner** ; **curl --path-as-is** / Burp.

## Références

- Microsoft : flux de données alternatifs NTFS, nommage 8.3
- Soroush Dalili : recherche IIS short-name et ::$DATA
