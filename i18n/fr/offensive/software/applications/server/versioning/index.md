---
title: "Gestion de versions : attaquer le contrôle de version auto-hébergé"
description: "Attaquer le contrôle de version auto-hébergé : récupérer les répertoires de métadonnées .git et .svn exposés sur le web pour reconstruire le code source et les secrets, atteindre les protocoles serveur non authentifiés de git et Subversion, et obtenir l'exécution de code via les hooks Git et la configuration au moment du clone."
keywords:
  - version control
  - Git
  - Subversion
  - source code disclosure
  - secrets in history
---

# Gestion de versions

Un checkout de contrôle de version transporte bien plus que les fichiers actuels : un historique complet de chaque changement, la configuration qui pilote l'outillage et, très souvent, des identifiants qui ont été commités une fois puis « retirés » plus tard mais qui restent toujours dans le store d'objets. La valeur offensive vient de trois directions : reconstruire le code source et les secrets depuis un répertoire de métadonnées qu'un serveur web a exposé par accident, atteindre un serveur de dépôt qui autorise l'accès anonyme, et transformer un clone ou un push en exécution de code via les hooks et la config que le VCS exécute automatiquement.

Ce domaine couvre les outils de contrôle de version eux-mêmes. Les forges d'hébergement de code en SaaS (GitHub, GitLab, Bitbucket) sont attaquées comme des services hébergés par un fournisseur sous [Online > Forges](../../online/forges/index.md).

## Triage

```bash
# A web root that leaked its VCS metadata directory
curl -s -o /dev/null -w '%{http_code}\n' https://<target>/.git/HEAD      # 200 => exposed .git
curl -s -o /dev/null -w '%{http_code}\n' https://<target>/.svn/wc.db     # 200 => exposed .svn (>=1.7)
# A reachable repository server
nmap -p9418,3690 -sV <target>                                            # 9418 git://  3690 svn://
```

Un `.git`/`.svn` lisible via HTTP mène aux pages de récupération propres à chaque outil ; un service `git://` ou `svn://` ouvert mène aux pages de protocole serveur ; un dépôt que vous pouvez cloner ou vers lequel vous pouvez pousser mène aux pages de hooks et de config.

## Sous-thèmes

- **[Git](git/index.md)** : récupérer un `.git` exposé, miner les secrets de l'historique, le démon `git://` non authentifié et l'exécution de code via les hooks et la config au moment du clone.
- **[SVN](svn/index.md)** : récupérer une copie de travail `.svn` exposée, et atteindre l'historique et les identifiants Subversion.

## Références

- [Git internals (Pro Git)](https://git-scm.com/book/en/v2/Git-Internals-Plumbing-and-Porcelain)
- [Subversion documentation](https://svnbook.red-bean.com/)
- [HackTricks: git and source code disclosure](https://book.hacktricks.wiki/en/network-services-pentesting/pentesting-web/git.html)
