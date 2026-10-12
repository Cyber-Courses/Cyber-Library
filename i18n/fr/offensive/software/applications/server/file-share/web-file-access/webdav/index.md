---
title: "WebDAV : attaquer l'extension HTTP en lecture-écriture"
description: "WebDAV étend HTTP avec des verbes pour lister, créer, déplacer et écrire des fichiers, transformant un serveur web en système de fichiers montable et accessible en écriture. La surface offensive comprend la découverte des verbes activés, le contournement d'une authentification faible, la traversée hors de la collection prévue, et l'utilisation de PUT ou MOVE pour placer un fichier exécutable dans un chemin interprété par le web en vue d'une exécution de code."
keywords:
  - webdav
  - propfind
  - put
  - dav verbs
  - move
---

# WebDAV

WebDAV (Web Distributed Authoring and Versioning) ajoute des verbes à HTTP, PROPFIND (lister et lire des propriétés), PUT (écrire un fichier), MKCOL (créer une collection), MOVE, COPY, DELETE, LOCK, de sorte qu'un serveur web devient un système de fichiers montable et accessible en écriture. Cette capacité de lecture-écriture est la surface d'attaque. Les manœuvres consistent à découvrir quels verbes sont activés et ce qu'ils exposent, à contourner une authentification souvent faible ou appliquée de manière incohérente, à traverser hors de la collection prévue et, avec le plus d'impact, à utiliser PUT (parfois combiné avec MOVE pour esquiver un filtre d'extension) pour déposer un fichier exécutable par le serveur en vue d'une exécution de code.

```bash
# detect WebDAV and its verbs
curl -s -X OPTIONS http://<target>/ -i | grep -iE 'allow|dav'   # DAV header + verbs
davtest -url http://<target>/                 # tests PUT and which extensions execute
```

## Sous-thèmes

- **[Découverte et méthodes](discovery-and-methods.md)** : trouver WebDAV et ses verbes activés.
- **[Contournement d'authentification](authentication-bypass.md)** : atteindre les verbes DAV malgré une authentification faible.
- **[Traversée de répertoires](directory-traversal.md)** : s'échapper de la collection prévue.
- **[Téléversement PUT vers RCE](put-upload-to-rce.md)** : déposer un fichier exécutable via PUT/MOVE.

## Références

- [RFC 4918 (WebDAV)](https://datatracker.ietf.org/doc/html/rfc4918)
- [HackTricks : WebDAV](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/put-method-webdav)
