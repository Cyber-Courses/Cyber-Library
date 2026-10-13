---
order: 3
title: "Fichiers de sauvegarde et temporaires : swaps d'éditeur, .bak et archives oubliées"
description: "Récupération de code source et de secrets dans les fichiers de sauvegarde et temporaires laissés dans la racine web : fichiers swap d'éditeur, copies .bak/~/.old et archives de déploiement que le serveur sert comme contenu statique."
keywords:
  - fichiers de sauvegarde
  - .bak
  - vim swap
  - divulgation de source
  - fichiers temporaires
---

# Fichiers de sauvegarde et temporaires

Éditer des fichiers sur le serveur, enregistrer des copies « au cas où » ou laisser une archive de déploiement dans la racine web produit des fichiers que le serveur traite comme statiques (il en sert donc les octets bruts au lieu de les exécuter). Cela transforme un `.php` en code source lisible et une archive en application complète.

## Ce qu'il faut chercher

Pour un script connu `app.php`, sondez les variantes courantes de sauvegarde et de swap :

```
app.php.bak      app.php~        app.php.old      app.php.save
app.php.orig     app.php.swp     app.php.swo      app.php.1
.app.php.swp     app.php.txt     app.php.copy     app.php.dist
#app.php#        app.phps
```

Les artefacts d'éditeur sont particulièrement fiables : Vim écrit `.<name>.swp`/`.swo`, Emacs écrit `name~` et `#name#`, et une édition interrompue laisse un swap récupérable. `.phps` est un handler de coloration de source qui renvoie le PHP comme source. N'importe lequel de ceux-ci servi avec un type de contenu `text/plain` est de la source brute.

Balayez aussi les restes de déploiement dans la racine et les chemins courants :

```
backup.zip   site.tar.gz   www.zip   backup.sql   db.sql   dump.sql
release.tar  app.tar.gz    .env.bak  config.php.bak
```

## Comment les trouver

Le content-discovery avec une wordlist est l'approche pratique, en mutant les noms de fichiers connus avec des suffixes de sauvegarde :

```bash
ffuf -w paths.txt -u https://target/FUZZ -mc 200
# backup-suffix mutation of discovered files:
for f in app.php index.php config.php; do
  for s in .bak '~' .old .swp .save .orig; do echo "$f$s"; done
done > bak.txt
ffuf -w bak.txt -u https://target/FUZZ -mc 200
```

Récupérez un swap Vim avec `vim -r` hors ligne :

```bash
curl -s https://target/.app.php.swp -o app.swp && vim -r app.swp
```

## Exploitation

- La source brute issue d'un `.bak`/swap offre le même levier qu'un dump de VCS : sinks, clés secrètes, identifiants.
- Un dump `.sql` ou une archive peut contenir toute la base de données (hashs de mots de passe, tokens) ou l'application entière, configuration comprise.
- Combinez avec le listing de répertoire et l'exposition de VCS ; l'un pointe souvent vers les autres (un listing révèle le nom de la sauvegarde, un historique `.git` révèle quels fichiers existent et pour lesquels sonder des sauvegardes).

## Outils

- **ffuf** / **feroxbuster** avec des wordlists de suffixes de sauvegarde ; **vim -r** pour la récupération de swap.

## Références

- OWASP WSTG : Review old backup and unreferenced files
- SecLists : backup-file wordlists
