---
title: "Git : attaquer l'outil de gestion de versions Git"
description: "Attaquer Git lui-même : récupérer un répertoire .git exposé sur le web pour en tirer le code source complet, miner l'historique des commits et les objets orphelins à la recherche de secrets, cloner depuis un démon git non authentifié, et obtenir l'exécution de code via les hooks et la configuration au moment du clone."
keywords:
  - Git
  - .git exposure
  - git daemon
  - git hooks
  - secrets in history
---

# Git

Git stocke un dépôt sous forme de base de données d'objets adressée par le contenu sous `.git/` : commits, trees et blobs indexés par SHA-1, plus des refs, un store d'objets packés et un `config`. Tout ce qu'un attaquant veut découle de cette disposition. Si un serveur web sert `.git/`, l'ensemble de l'arborescence et son historique peuvent être reconstruits hors ligne. Si l'historique contient un identifiant supprimé par la suite, il reste dans le store d'objets. Si un serveur de dépôt est anonyme, il peut être cloné. Et parce que Git exécute des hooks et honore la config de dépôt automatiquement, un clone ou un push peut devenir une exécution de code.

## Triage

```bash
curl -s -o /dev/null -w '%{http_code}\n' https://<target>/.git/HEAD     # 200 + "ref: refs/heads/..." => exposed
nmap -p9418 -sV <target>                                                # git:// daemon
git ls-remote git://<target>/<repo>                                     # anonymous clone test
```

## Pages

- **[Répertoire .git exposé](exposed-git-directory.md)** : reconstruire l'arborescence source complète et la config depuis un `.git` servi par le web.
- **[Secrets dans l'historique](secrets-in-history.md)** : récupérer des identifiants depuis l'historique des commits, les objets orphelins et le reflog.
- **[Démon](git-daemon.md)** : cloner des dépôts privés depuis un service `git://` non authentifié.
- **[Hooks et exécution par la config](hooks-and-config-execution.md)** : exécution de code via les hooks et la configuration au moment du clone.

## Références

- [Git internals: plumbing and porcelain](https://git-scm.com/book/en/v2/Git-Internals-Plumbing-and-Porcelain)
- [git-dumper](https://github.com/arthaud/git-dumper)
