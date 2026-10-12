---
title: "Accès aux fichiers par le web : attaquer les services de fichiers sur HTTP"
description: "Les fichiers sont de plus en plus servis et reçus sur HTTP : simples serveurs de fichiers HTTP et listages de répertoire, WebDAV pour l'accès en lecture-écriture, et produits de transfert de fichiers géré pour les échanges inter-entreprises. Chacun expose sa propre surface, listage et traversée de répertoire, téléversement vers exécution, contournement d'authentification, et les vulnérabilités fortement ciblées des appliances MFT."
keywords:
  - http file server
  - webdav
  - managed file transfer
  - upload
  - path traversal
---

# Accès aux fichiers par le web

Le partage de fichiers est passé sur HTTP, et trois formes dominent. Les simples serveurs de fichiers HTTP (du serveur intégré d'un langage aux applications dédiées) exposent des répertoires pour le téléchargement et parfois le téléversement. WebDAV étend HTTP avec des verbes de lecture-écriture, transformant un serveur web en système de fichiers montable et accessible en écriture. Et les produits de transfert de fichiers géré (MFT) fournissent des échanges de fichiers inter-entreprises gouvernés via des portails web et des API. Chacun a une surface distincte : listage et traversée sur les serveurs de fichiers simples, téléversement vers exécution et abus de verbes sur WebDAV, et sur le MFT les contournements d'authentification et les failles d'injection qui ont motivé certaines des plus grandes campagnes d'exploitation de masse.

```bash
# fingerprint what is serving files over HTTP
curl -sI http://<target>/                      # Server header, allowed methods
curl -s -X OPTIONS http://<target>/ -i | grep -i allow   # WebDAV verbs?
```

## Sous-thèmes

- **[Serveurs de fichiers HTTP](http-file-servers/index.md)** : listage de répertoire, traversée et téléversement vers RCE.
- **[WebDAV](webdav/index.md)** : l'extension HTTP en lecture-écriture.
- **[Transfert de fichiers géré](managed-file-transfer/index.md)** : les appliances MFT et leurs exploits.

## Références

- [RFC 4918 (WebDAV)](https://datatracker.ietf.org/doc/html/rfc4918)
- [OWASP : téléversement de fichier sans restriction](https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload)
