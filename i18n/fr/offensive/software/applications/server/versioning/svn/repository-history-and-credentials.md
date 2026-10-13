---
title: "Historique et identifiants du dépôt : atteindre les révisions et les secrets Subversion"
description: "Atteindre un serveur de dépôt Subversion pour récupérer code source et secrets : lecture anonyme via svnserve et HTTP/WebDAV, parcourir l'historique des révisions pour récupérer les fichiers supprimés et les identifiants commités avec svn log et svn cat, et récolter les identifiants Subversion en cache sur le disque."
keywords:
  - subversion history
  - svnserve
  - anonymous read
  - svn cat
  - cached credentials
---

# Historique et identifiants du dépôt

Au-delà d'une copie de travail fuitée, le serveur de dépôt Subversion lui-même détient l'historique complet des révisions. Là où il autorise la lecture anonyme, cet historique est une divulgation de code source et de secrets : chaque révision de chaque fichier, y compris les fichiers supprimés plus tard, est récupérable, et Subversion met en cache les identifiants sur les clients qui s'y connectent.

## Atteindre le serveur

Subversion est servi via `svn://` (le démon svnserve sur le port TCP 3690) ou via HTTP/HTTPS avec le module Apache `mod_dav_svn`. La lecture anonyme est un paramètre par défaut courant sur svnserve : un `svnserve.conf` avec `anon-access = read` laisse quiconque lister et faire un checkout.

```bash
nmap -p3690,80,443 -sV <target>
svn info  svn://<target>/repo        # or an http(s)://<target>/svn/repo URL
svn ls -R svn://<target>/repo         # anonymous listing => anonymous read is on
```

Un `svn ls`/`svn info` réussi sans identifiants confirme la lecture anonyme.

## Parcourir l'historique

`svn log` énumère les révisions et `svn cat` récupère un fichier à n'importe quelle révision. Pour un chemin qui existe toujours à HEAD, `-r` seul suffit ; pour un chemin qui a été supprimé, fixez la peg revision avec `@<rev>` pour que Subversion le localise dans la révision où il existait plutôt qu'à HEAD (le peg implicite pour une URL) :

```bash
svn log -v svn://<target>/repo                      # revisions, with the paths each one changed
svn cat 'svn://<target>/repo/config/db.ini@42'      # peg at r42: recovers the file even if deleted at HEAD
svn export svn://<target>/repo ./repo-head          # the full current tree in one step
```

Lisez le log `-v` à la recherche des chemins marqués `D` (supprimés) : le fichier a disparu de HEAD, mais `svn cat <url>@<rev>` fixé sur une révision avant la suppression le ramène. Un commit « remove the password » se récupère exactement de cette façon.

## Identifiants en cache

Subversion stocke les identifiants utilisés pour s'authentifier auprès d'un dépôt sous le répertoire de config de l'utilisateur, souvent non chiffrés. Sur un hôte où vous avez déjà un point d'appui, ceux-ci livrent des identifiants de dépôt (et parfois de domaine) fonctionnels :

```bash
ls ~/.subversion/auth/svn.simple/          # one file per realm
cat ~/.subversion/auth/svn.simple/*        # username and, on many platforms, the password in cleartext
```

## Suite

Le code source et les fichiers de config récupérés portent des identifiants de base de données, de service et de déploiement ; les entrées `svn.simple` en cache sont directement réutilisables contre le dépôt et partout où le mot de passe a été réutilisé. Alimentez les deux dans la réutilisation d'identifiants à travers le parc.

## Références

- [svnserve and anonymous access (SVN book)](https://svnbook.red-bean.com/en/1.7/svn.serverconfig.svnserve.html)
- [Subversion client credential caching](https://svnbook.red-bean.com/en/1.7/svn.serverconfig.netmodel.html)
