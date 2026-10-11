---
title: "Serveurs de fichiers HTTP : attaquer les applications web servant des répertoires"
description: "Les serveurs de fichiers HTTP exposent une arborescence de répertoires pour le téléchargement et parfois le téléversement, du serveur intégré d'un langage à des produits comme HFS et diverses interfaces web de NAS. La surface comprend le listage de répertoire qui révèle l'arborescence, la traversée de chemin pour lire des fichiers hors de la racine servie, et le téléversement de fichier qui devient exécution de code lorsque le chemin de téléversement est servi ou interprété."
keywords:
  - http file server
  - directory listing
  - path traversal
  - file upload
  - hfs
---

# Serveurs de fichiers HTTP

Un serveur de fichiers HTTP expose une arborescence de répertoires via HTTP pour la navigation et le téléchargement, et souvent pour le téléversement. Ils vont du serveur intégré en une ligne (le `http.server` de Python, le serveur intégré de PHP) à des applications dédiées (HFS HTTP File Server, diverses interfaces web de NAS et d'appliances). La surface est constante : listage de répertoire qui cartographie l'arborescence servie et révèle les fichiers, traversée de chemin qui s'échappe de la racine servie pour lire des fichiers arbitraires, et fonctionnalité de téléversement qui devient exécution de code lorsque le fichier téléversé atterrit dans un chemin que le serveur sert ou interprète.

```bash
# fingerprint the server and look for listing/upload
curl -sI http://<target>:<port>/              # Server header identifies the product
curl -s http://<target>:<port>/               # directory listing present?
curl -s -X OPTIONS http://<target>:<port>/ -i # methods (PUT => upload)
```

## Sous-thèmes

- **[Listage de répertoire](directory-listing.md)** : cartographier l'arborescence servie.
- **[Traversée de chemin](path-traversal.md)** : lire des fichiers hors de la racine servie.
- **[Téléversement de fichier vers RCE](file-upload-to-rce.md)** : transformer le téléversement en exécution de code.
- **[Exploits serveur connus](known-server-exploits.md)** : vulnérabilités de serveurs de fichiers propres à un produit.

## Références

- [OWASP : traversée de chemin](https://owasp.org/www-community/attacks/Path_Traversal)
- [OWASP : téléversement de fichier sans restriction](https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload)
