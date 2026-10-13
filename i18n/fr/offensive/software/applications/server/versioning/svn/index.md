---
title: "SVN : attaquer le contrôle de version Subversion"
description: "Attaquer Subversion : récupérer une copie de travail .svn exposée sur le web pour reconstruire le code source et les métadonnées depuis la base de données wc.db et le pristine store, et atteindre l'historique et les identifiants du dépôt Subversion via svn et HTTP."
keywords:
  - SVN
  - Subversion
  - .svn exposure
  - svnserve
  - source code disclosure
---

# SVN

Subversion garde les métadonnées d'une copie de travail dans un répertoire `.svn`. Depuis la version 1.7, il s'agit d'une unique base de données SQLite (`.svn/wc.db`) plus un store `pristine` du contenu de fichier non modifié, tandis que les dispositions plus anciennes dispersaient un dossier `.svn` avec des copies `entries` et `text-base` dans chaque répertoire. Dans tous les cas, un répertoire `.svn` qu'un serveur web sert livre le code source complet et ses métadonnées, et le serveur de dépôt derrière lui autorise souvent un accès anonyme à l'historique qui récupère les fichiers supprimés et les identifiants commités.

## Triage

```bash
curl -s -o /dev/null -w '%{http_code}\n' https://<target>/.svn/wc.db          # 200 => exposed (>=1.7)
curl -s -o /dev/null -w '%{http_code}\n' https://<target>/.svn/entries        # 200 => exposed (legacy)
nmap -p3690 -sV <target>; svn ls svn://<target>/ 2>/dev/null                  # svnserve, anonymous read
```

## Pages

- **[Répertoire .svn exposé](exposed-svn-directory.md)** : reconstruire le code source et les métadonnées depuis une copie de travail servie par le web.
- **[Historique et identifiants du dépôt](repository-history-and-credentials.md)** : atteindre l'historique Subversion pour récupérer les fichiers supprimés et les identifiants.

## Références

- [Version Control with Subversion (the SVN book)](https://svnbook.red-bean.com/)
- [HackTricks: pentesting SVN / source disclosure](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-web/index.html)
