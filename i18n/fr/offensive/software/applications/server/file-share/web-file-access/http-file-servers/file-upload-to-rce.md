---
order: 2
title: "Envoi de fichier vers RCE : transformer un envoi en exécution de code"
description: "Un serveur de fichiers HTTP qui accepte les envois devient de l'exécution de code lorsqu'un fichier envoyé est placé là où le serveur l'interprète : un script dans un répertoire exécutable côté web, un fichier dont l'extension ou le content-type est mal géré, ou un envoi combiné à une traversée pour contrôler le chemin de destination. Le fichier déposé s'exécute alors côté serveur à la requête."
keywords:
  - envoi de fichier
  - web shell
  - contournement d'extension
  - content-type
  - envoi non restreint
---

# Envoi de fichier vers RCE

Une fonctionnalité d'envoi est dangereuse dès que le fichier envoyé peut se retrouver quelque part où le serveur l'exécutera. Le cas direct est l'envoi d'un script côté serveur (`.php`, `.jsp`, `.aspx`) dans un répertoire que le serveur web interprète, ce qui donne un web shell. Lorsque l'application tente de restreindre les envois, les contournements abondent : extensions interdites éludées par des variantes ou la casse, vérifications de content-type ou de magic bytes trompées, doubles extensions, octets nuls et caractères en fin de nom. Et un envoi combiné à une traversée de chemin permet à l'attaquant de choisir la destination, plaçant un fichier exécutable dans un chemin servi par le web même si les envois sont censés être confinés.

```bash
# plain upload of a web shell where the dir is executable
curl -s -F 'file=@shell.php' http://<target>/upload
curl -s 'http://<target>/uploads/shell.php?cmd=id'          # trigger it
# extension / content-type bypasses when filtering exists
#   shell.php.jpg | shell.pHp | shell.phtml/.php5 | trailing dot/space (Windows)
curl -s -F 'file=@shell.phtml;type=image/jpeg' http://<target>/upload
# upload + traversal to control the write path (escape the uploads dir)
curl -s -F 'file=@shell.php;filename=../../shell.php' http://<target>/upload
```

## Notes d'exploitation

- Deux conditions doivent être réunies : le fichier doit être stocké et l'emplacement de stockage doit être interprété. Confirmez où atterrissent les envois (le listing ou la réponse révèle souvent le chemin) et si ce chemin exécute des scripts.
- Les contournements de filtre sont basés sur l'extension (`.phtml`, `.php5`, la casse, la double extension), sur le contenu (préfixer des magic bytes valides, définir un content-type d'image), ou spécifiques à l'environnement (point/espace de fin sous Windows, `::$DATA`) ; essayez-les systématiquement quand un envoi naïf est rejeté.
- Si le répertoire d'envoi n'est pas exécutable, utilisez la traversée dans le nom de fichier pour placer le fichier dans un répertoire qui l'est, ou chaînez avec une réécriture de configuration serveur (`.htaccess` pour faire exécuter à un répertoire une extension choisie).
- Les payloads non-script comptent aussi : écraser une config, un cron ou un `authorized_keys` via envoi+traversée atteint l'exécution sans web shell.

## Références

- [OWASP: unrestricted file upload](https://owasp.org/www-community/vulnerabilities/Unrestricted_File_Upload)
- [PortSwigger: file upload vulnerabilities](https://portswigger.net/web-security/file-upload)
