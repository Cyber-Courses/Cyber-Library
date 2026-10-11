---
title: "Découverte et méthodes : trouver WebDAV et ses verbes activés"
order: 4
description: "WebDAV se détecte à partir de l'en-tête de réponse DAV et des verbes qu'une requête OPTIONS rapporte. PROPFIND liste ensuite les collections et les fichiers, et tester PUT révèle si les écritures sont autorisées et quelles extensions téléversées le serveur exécutera. Cette énumération définit exactement quelle attaque WebDAV s'applique à une cible."
keywords:
  - options
  - propfind
  - dav header
  - davtest
  - allowed methods
---

# Découverte et méthodes

L'énumération WebDAV établit si DAV est présent, quels verbes sont activés et ce que les écritures permettent d'obtenir. L'en-tête `DAV` dans une réponse OPTIONS confirme WebDAV et sa classe de conformité ; l'en-tête `Allow` liste les verbes. PROPFIND énumère les collections et les fichiers (le listage de WebDAV), et un test PUT contrôlé montre si les téléversements sont acceptés et, surtout, quelles extensions téléversées le serveur exécute par rapport à celles qu'il sert de façon inerte. Ce dernier point décide si un téléversement est une exécution de code ou juste un fichier.

```bash
# verbs and DAV support
curl -s -X OPTIONS http://<target>/ -i | grep -iE 'allow:|dav:'
# PROPFIND listing (Depth: 1 lists the immediate collection)
curl -s -X PROPFIND http://<target>/ -H 'Depth: 1' --data '' | grep -oE '<D:href>[^<]+'
# automated: which extensions can be uploaded and which execute
davtest -url http://<target>/
cadaver http://<target>/                       # interactive DAV client (ls, put, move)
```

Lisez la sortie de `davtest` spécifiquement pour savoir quelles extensions ont à la fois été téléversées avec succès et exécutées ; c'est la liste des types de charges utiles utilisables pour [Téléversement PUT vers RCE](put-upload-to-rce.md).

## Notes d'exploitation

- La présence de l'en-tête `DAV` et une liste de verbes incluant `PUT`/`MKCOL`/`MOVE` signalent un WebDAV accessible en écriture qui mérite d'être attaqué ; un DAV en lecture seule (seulement PROPFIND/GET) permet tout de même le listage et la traversée.
- `davtest` est le moyen le plus rapide de connaître l'ensemble des extensions exécutables ; là où le PUT de `.php`/`.jsp`/`.aspx` est bloqué mais qu'un autre type exécutable (ou un renommage par MOVE) fonctionne, c'est la voie.
- Le listage PROPFIND révèle des fichiers et des collections qui ne sont pas autrement liés, comme un listage de répertoire, alimentant à la fois le pillage et les cibles de traversée/téléversement.
- Énumérez sous chaque identité dont vous disposez (anonyme et tout identifiant), car la disponibilité des verbes diffère souvent selon l'authentification, ce qui motive l'étape de [contournement d'authentification](authentication-bypass.md).

## Outils

- [davtest](https://github.com/cldrn/davtest)
- [cadaver](http://www.webdav.org/cadaver/)

## Références

- [RFC 4918 : OPTIONS et PROPFIND](https://datatracker.ietf.org/doc/html/rfc4918)
- [HackTricks : WebDAV](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/put-method-webdav)
