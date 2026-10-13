---
order: 2
title: "Répertoires de gestion de version exposés : reconstruire le code source depuis .git, .svn et .hg"
description: "Trouver et dumper les répertoires de gestion de version servis par le web pour récupérer l'intégralité du code source, de l'historique et des secrets, puis reconstruire l'arbre de travail hors ligne."
keywords:
  - exposition git
  - répertoire .git
  - git-dumper
  - exposition svn
  - divulgation de source
---

# Répertoires de gestion de version

Déployer par `git clone`/`git pull` dans la racine web, ou copier un répertoire de travail, laisse le dossier `.git` (ou `.svn`, `.hg`) sous la racine du document. Quand le serveur sert ces fichiers comme contenu statique, un attaquant télécharge les métadonnées du dépôt et reconstruit l'**intégralité de l'arbre source et de son historique**, y compris les secrets commités puis « retirés » plus tard.

## Détecter l'exposition

Sondez les fichiers de métadonnées qui doivent exister dans un dépôt :

```
GET /.git/HEAD            HTTP/1.1     # "ref: refs/heads/main"
GET /.git/config          HTTP/1.1     # remotes, maybe creds in URLs
GET /.svn/wc.db           HTTP/1.1
GET /.hg/requires         HTTP/1.1
```

Un 200 avec `ref: refs/heads/...` sur `/.git/HEAD` confirme un répertoire Git servi. Le listing de répertoire désactivé ne vous protège pas : les noms de fichiers à l'intérieur de `.git` sont prévisibles, l'arbre peut donc être parcouru sans lui.

## Dumper et reconstruire

Même sans listing de répertoire, le modèle objet de Git permet de parcourir depuis `HEAD` vers chaque objet. Des outils automatisés le font :

```bash
git-dumper https://target/.git/ ./loot
# or
python3 gitdumper.py https://target/.git/ ./loot && cd loot && git checkout -- .
```

`git-dumper` récupère `HEAD`, les refs, les objets packés et les objets loose, puis extrait l'arbre de travail. À partir de là :

```bash
git log --all --oneline          # full history, including deleted content
git show <commit>:config.php      # recover a file from any revision
git log -p | grep -iE 'password|secret|api[_-]?key|token'
```

Les secrets qui ont été commités puis supprimés restent dans l'historique, donc greppez chaque révision, pas seulement la pointe.

## Exploitation

- Le code source récupéré révèle les sinks exacts d'injection/désérialisation, les clés secrètes du framework (forge de cookies signés et de ViewState), ainsi que les identifiants de base de données et de tiers.
- `.git/config` et les packed-refs contiennent parfois des URL de remote avec des tokens embarqués ; les logs (`.git/logs/HEAD`) révèlent les e-mails des committers et l'activité des branches.
- Pour `.svn`, parsez `wc.db` (SQLite) pour la liste des fichiers pristine et récupérez les copies pristine mises en cache ; pour `.hg`, récupérez le store et reconstruisez.

## Outils

- **git-dumper**, **gitdumper.py** (GitTools), **svn-extractor**.

## Références

- OWASP WSTG : Review webserver metafiles / source code disclosure
- Projet GitTools
