---
title: "Traversée de chemin : lire des fichiers hors de la racine servie"
order: 3
description: "Un serveur de fichiers HTTP qui construit un chemin de système de fichiers à partir de la requête sans le canonicaliser correctement permet la traversée de chemin : les séquences ../, les variantes encodées ou les chemins absolus dans l'URL ou un paramètre de nom de fichier atteignent des fichiers hors de la racine servie. Cela produit une lecture arbitraire de fichiers, récupérant configurations, identifiants et code source au privilège du processus serveur."
keywords:
  - path traversal
  - directory traversal
  - ../
  - url encoding
  - arbitrary file read
---

# Traversée de chemin

Un serveur de fichiers associe une requête à un fichier sur le disque, et s'il concatène la racine servie avec le chemin demandé sans canonicaliser ni borner le résultat, un attaquant s'échappe de la racine avec des séquences de traversée. Le classique `../` remonte les répertoires ; les formes encodées (`%2e%2e%2f`, double encodage, UTF-8 surlong), les séparateurs mixtes sous Windows (`..\`) et les chemins absolus contournent les filtres naïfs. Le résultat est une lecture arbitraire de fichiers limitée uniquement par les privilèges du processus serveur, récupérant fichiers système, configuration d'application, identifiants et code source.

```bash
# basic and encoded traversal in the path or a download parameter
curl -s --path-as-is http://<target>/../../../../etc/passwd
curl -s 'http://<target>/download?file=../../../../etc/passwd'
curl -s 'http://<target>/download?file=%2e%2e%2f%2e%2e%2f%2e%2e%2fetc%2fpasswd'  # url-encoded
curl -s 'http://<target>/download?file=....//....//etc/passwd'                   # filter bypass
# Windows targets
curl -s --path-as-is 'http://<target>/..\..\..\windows\win.ini'
```

## Notes d'exploitation

- Utilisez `--path-as-is` pour que curl ne réduise pas le `../` avant l'envoi ; de nombreux filtres sont contournés par l'encodage (`%2e%2e%2f`), le double encodage, ou le motif `....//` qui survit à une suppression naïve unique.
- Visez les fichiers de grande valeur pour la plateforme : `/etc/passwd` et `/etc/shadow`, la configuration d'application (`web.config`, `.env`, paramètres de framework avec identifiants de BD), les clés SSH et le code source de l'application lui-même pour d'autres failles.
- Une traversée dans un paramètre de téléchargement/`file=` est aussi utile qu'une traversée dans le chemin ; testez les deux.
- La lecture est bornée par le privilège du processus serveur ; un serveur de fichiers s'exécutant en tant que root ou en tant que compte de service atteint davantage. Associez à tout téléversement pour un impact côté écriture ([Téléversement de fichier vers RCE](file-upload-to-rce.md)).

## Références

- [OWASP : traversée de chemin](https://owasp.org/www-community/attacks/Path_Traversal)
- [PortSwigger : traversée de répertoires](https://portswigger.net/web-security/file-path-traversal)
