---
title: "Secrets dans l'historique : miner les commits, objets et le reflog Git"
description: "Récupérer des identifiants depuis l'historique d'un dépôt Git : pourquoi supprimer un secret de l'arbre de travail le laisse dans le store d'objets, chercher dans tous les commits avec le pickaxe, récupérer les objets orphelins et force-pushés avec fsck et le reflog, et automatiser avec TruffleHog et Gitleaks."
keywords:
  - secrets in git history
  - git pickaxe
  - dangling objects
  - trufflehog
  - gitleaks
---

# Secrets dans l'historique

Retirer un secret dans un commit ultérieur ne le retire pas du dépôt. Git conserve chaque version de chaque fichier comme un objet immuable indexé par le hash de son contenu, de sorte qu'une clé, un jeton ou un mot de passe qui a été commité un jour reste accessible via l'historique, via des objets vers lesquels aucune branche ne pointe plus, et via le reflog, tant que vous détenez le dépôt. Un `.git` que vous avez copié ou dumpé emporte ce store d'objets complet avec lui ; un clone anonyme récent n'emporte que les objets accessibles depuis les refs publiées, pas les objets non référencés ni les reflogs de l'origine.

## Chercher dans l'historique commité

Le pickaxe (`-S`/`-G`) trouve le commit exact qui a introduit ou retiré une chaîne, à travers chaque ref, et c'est là que tombent les nettoyages du type « j'ai supprimé le mot de passe dans le commit suivant » :

```bash
git log -p --all -S 'password'                 # commits that add/remove the literal "password"
git log -p --all -G 'AKIA[0-9A-Z]{16}'         # regex: an AWS access-key id anywhere in a diff
git log --all --oneline -- config/secrets.yml  # every revision of a sensitive path, then git show each
```

`--all` parcourt toutes les refs plutôt que la seule branche courante, et `--source` nomme la ref sur laquelle vit chaque résultat.

## Récupérer les objets orphelins et force-pushés

Les commits retirés par `git rebase`, `git commit --amend` ou un force-push deviennent non référencés mais restent dans la base de données d'objets d'un dépôt qui les détenait avant la réécriture : votre propre clone de travail de longue durée, ou un `.git` que vous avez dumpé pendant que les objets étaient encore présents. `git fsck` les fait remonter, et le reflog local enregistre où pointaient vos branches avant un reset ou un amend :

```bash
git fsck --lost-found --dangling 2>/dev/null   # dangling commit/blob SHAs present in THIS repo
git cat-file -p <dangling-sha>                 # read the content directly
git reflog --all                               # your local ref history (not the server's)
```

Un blob orphelin que `git log` ne peut atteindre est là où un commit réécrit « fix: remove secret » a laissé la valeur d'origine. Notez la limite : un clone pris après le force-push du serveur ne reçoit pas les objets abandonnés, et son reflog ne contient que vos propres mises à jour, donc récupérez ceux-ci depuis un dépôt que vous déteniez plus tôt ou depuis un `.git` dumpé, pas depuis un clone frais.

## L'automatiser

Lancez un scanner dédié sur l'historique complet plutôt que de grep à l'aveugle ; ils portent des centaines de motifs d'identifiants et peuvent vérifier les clés en direct :

```bash
trufflehog git file://./loot --only-verified    # scan a local clone/dump, confirm which creds still work
gitleaks detect --source . --log-opts '--all'   # regex + entropy across all refs
```

## Suite

Un identifiant récupéré est un matériau de réutilisation immédiat : clés de base de données et de cloud, jetons CI, clés de déploiement SSH et secrets d'API. Validez-le, puis pivotez vers tout ce qu'il déverrouille. Les clés « rotées » après la fuite valent la peine d'être testées quand même, la rotation est fréquemment incomplète.

## Outils

- [TruffleHog](https://github.com/trufflesecurity/trufflehog)
- [Gitleaks](https://github.com/gitleaks/gitleaks)

## Références

- [git log pickaxe (-S/-G)](https://git-scm.com/docs/git-log)
- [git fsck](https://git-scm.com/docs/git-fsck)
