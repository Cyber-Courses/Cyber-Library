---
title: "Prise de contrôle du runner"
order: 5
description: "Obtenir une exécution de code sur les runners GitLab via un job CI contrôlé, s'échapper de l'exécuteur Docker lorsqu'il tourne en mode privilégié ou monte le socket de l'hôte, atteindre les données de projets colocataires sur un runner partagé, et enregistrer un runner pirate avec un jeton d'enregistrement divulgué pour capturer les futurs jobs."
keywords:
  - runner GitLab
  - runner partagé
  - exécuteur Docker
  - mode privilégié
  - jeton d'enregistrement de runner
  - runner pirate
---

# Prise de contrôle du runner

Un runner GitLab exécute le `script:` de n'importe quel job qu'il prend en charge, de sorte qu'un job que vous contrôlez est, par conception, une exécution de code sur le runner. Sur un runner **partagé** qui sert de nombreux projets, cette exécution se trouve à côté des environnements de build, caches et artefacts d'autres locataires. Les cibles d'élévation sont l'hôte sous un runner à exécuteur Docker, les données des projets colocataires, et la capacité d'enregistrer votre propre runner pour qu'il récolte les futurs jobs.

## Prérequis

- Un projet dont vous pouvez exécuter le pipeline (voir [injection de pipeline CI/CD](ci-cd-pipeline-injection.md)) auquel est assigné un runner **partagé** ou de groupe.
- Pour une évasion d'hôte : l'exécuteur Docker configuré avec `privileged = true`, ou le socket Docker de l'hôte monté dans les conteneurs de job. Faites l'empreinte depuis l'intérieur d'un job avant toute autre chose :

```yaml
recon:
  script:
    - id; cat /proc/1/cgroup              # am I in a container, as which user
    - ls -la /var/run/docker.sock || true # is the host socket mounted into the job
    - grep CapEff /proc/self/status        # capability set -> privileged container
    - mount | grep -E 'docker|host'        # host paths bind-mounted in
```

Un `docker.sock` présent dans le job, ou un masque de capacités `CapEff` complet, signale un exécuteur privilégié et un chemin direct vers l'hôte. Un ensemble de capacités minimal et aucun socket signifie que vous êtes confiné au conteneur et devriez plutôt pivoter vers les données des locataires et l'enregistrement pirate.

## Pourquoi l'exécuteur Docker s'échappe

Le `config.toml` du runner définit le comportement de l'exécuteur. Deux mauvaises configurations effondrent la frontière du conteneur :

- `privileged = true` donne aux conteneurs de job les capacités et l'accès aux périphériques pour monter les périphériques de bloc de l'hôte, de sorte que l'« isolation » est cosmétique.
- Un montage `volumes = ["/var/run/docker.sock:/var/run/docker.sock"]` remet au job le contrôle du démon Docker de l'hôte, qui peut démarrer de nouveaux conteneurs montant en bind le système de fichiers de l'hôte.

L'un comme l'autre signifie qu'un job peut atteindre le système de fichiers racine de l'hôte. L'évasion concrète est le breakout standard de conteneur privilégié / socket monté documenté dans les références d'évasion de conteneur ; depuis l'hôte, vous atteignez le `config.toml` du runner lui-même, qui contient les jetons du runner, et tout identifiant mis en cache sur la machine. Traitez la lecture de ce `config.toml` comme l'objectif, car il livre le jeton d'authentification du runner utilisé dans l'étape du runner pirate ci-dessous.

## Données colocataires sur un runner partagé

Même confinées au conteneur, les arborescences de travail et de cache d'un runner partagé persistent entre les jobs et contiennent les checkouts et archives de cache d'autres projets :

```yaml
survey:
  script:
    - find / -type d \( -name builds -o -name cache \) 2>/dev/null
    - ls -la /builds 2>/dev/null; ls -la /cache 2>/dev/null
```

Un `/builds/<other-group>` ou `/cache` peuplé par des projets qui ne sont pas les vôtres est une fuite de colocataire : leur code source, et tout secret qu'un job antérieur a écrit sur le disque, sont lisibles depuis votre job. Les archives de cache sont aussi un vecteur d'empoisonnement, car un job ultérieur qui restaure une clé de cache que vous pouvez écrire fait confiance à son contenu.

## Enregistrer un runner pirate

Si vous récupérez un **jeton d'enregistrement de runner** (depuis le `config.toml` de l'hôte, un paramètre de groupe/projet divulgué, ou un point d'ancrage admin), vous enregistrez votre propre runner contre l'instance. Il prend alors en charge de vrais jobs de cette portée et les exécute dans un environnement que vous observez entièrement, ce qui capture les variables CI/CD et les jetons de job de ces jobs à mesure qu'ils s'exécutent.

```bash
# Register an attacker-controlled runner to the group/instance scope
gitlab-runner register \
  --non-interactive \
  --url "https://gitlab.com/" \
  --registration-token "$LEAKED_REG_TOKEN" \
  --executor shell \
  --description "build-cache-node-7"   # blend in with existing runner names

# Then run it; jobs it accepts execute on your host with their secrets in the env
gitlab-runner run
```

Interprétation : un enregistrement réussi renvoie un jeton d'authentification de runner et le runner apparaît dans la liste des runners du projet/groupe. Comme vous avez choisi l'exécuteur `shell`, chaque job qu'il accepte s'exécute directement sur votre machine, de sorte que vous lisez ses variables et son `CI_JOB_TOKEN` directement dans l'environnement. Une chaîne de nommage qui correspond aux runners existants l'empêche de ressortir dans la liste des runners.

## Suite

L'accès à l'hôte livre le magasin de jetons du runner et les identifiants cloud mis en cache, qui pivotent hors de GitLab vers l'infrastructure connectée. Les variables de job et les jetons de job capturés alimentent l'[abus de jetons](token-abuse.md). Un jeton d'enregistrement atteignable ici est le gain des chemins admin dans [exploits admin et API connus](known-admin-and-api-exploits.md). Les mécanismes d'évasion de conteneur sont couverts dans le matériel d'évasion de conteneur Server.

## Outils

- **gitlab-runner** (le binaire officiel) pour enregistrer et exécuter un runner pirate.
- Un collecteur de requêtes pour les rappels issus de l'exécution de jobs.

## Références

- [GitLab Runner executors](https://docs.gitlab.com/runner/executors/)
- [GitLab Runner registration](https://docs.gitlab.com/runner/register/)
- [GitLab : security implications of Docker executor privileged mode](https://docs.gitlab.com/runner/executors/docker.html#use-docker-in-docker-with-privileged-mode)
- [GitLab : runner security](https://docs.gitlab.com/runner/security/)
