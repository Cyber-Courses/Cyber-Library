---
order: 1
title: "Abus de l'upload de fichiers"
description: "Transformer une fonctionnalité d'upload en exécution de code ou en écrasement de fichier en venant à bout des contrôles sur le nom, le type, le contenu et la destination des fichiers fournis par l'attaquant."
keywords:
  - upload de fichiers
  - web shell
  - upload sans restriction
  - zip slip
  - écriture de fichier arbitraire
---

# Upload

L'abus d'upload part de toute fonctionnalité qui écrit sur disque des octets fournis par l'attaquant : avatars, pièces jointes de documents, assistants d'import, arrière-plans de profil.

Le but est le contrôle de trois propriétés du fichier stocké : son **nom et son extension** (pour que le serveur l'exécute), son **contenu déclaré et réel** (pour qu'il passe les contrôles de type tout en s'exécutant), et son **chemin de destination** (pour qu'il atterrisse là où il est servi ou là où il écrase quelque chose d'important). L'upload sans restriction chaîne les deux premières pour déposer un web shell dans un répertoire servi par le web puis le requête. L'extraction d'archive (Zip Slip) attaque la troisième : les noms d'entrées dans une archive forgée s'échappent du répertoire de décompression pour écrire partout où le processus le peut, écrasant des ressources, des binaires ou de la configuration. Les deux routes se terminent couramment par une exécution de code à distance.

## Pages

- **[Unrestricted upload](unrestricted-upload.md)** : venir à bout des contrôles d'extension, de Content-Type et de magic bytes pour stocker un script exécutable dans un chemin servi par le web, puis le requêter pour une exécution de code à distance.
- **[Zip Slip](zip-slip.md)** : des entrées d'archive avec des séquences de traversée ou des chemins absolus écrivent hors du répertoire de décompression pendant l'extraction, écrasant des ressources web ou des binaires et implant...

## Outils

- **[Burp Suite](https://portswigger.net/burp)** : Intruder pour fuzzer les extensions, le Content-Type et les magic bytes.
- **[exiftool](https://exiftool.org/)** : intégrer des payloads dans les métadonnées d'image pour construire des polyglottes.

## Références

- [OWASP: Unrestricted File Upload](https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload)
- [PayloadsAllTheThings: Upload Insecure Files](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Upload%20Insecure%20Files)
