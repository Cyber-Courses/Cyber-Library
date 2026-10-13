---
title: "Démon Git : cloner des dépôts privés via git:// non authentifié"
description: "Abuser d'un démon Git non authentifié : comment le protocole git:// sur le port 9418 sert des dépôts sans authentification, énumérer et cloner les dépôts exposés, les verrous export-ok et exportAll, et le danger d'un service receive-pack ou upload-archive activé."
keywords:
  - git daemon
  - git protocol
  - port 9418
  - anonymous clone
  - export-ok
---

# Démon

`git daemon` expose des dépôts via le protocole `git://` sur le port TCP 9418 sans authentification ni chiffrement de transport, par conception : il est destiné à la lecture anonyme de miroirs publics. Lorsqu'il est pointé vers une arborescence de dépôts internes ou privés, quiconque peut atteindre le port peut cloner le code et son historique complet.

## Identifier et énumérer

```bash
nmap -p9418 -sV <target>
# the daemon only serves a repo that is explicitly exported, so test specific paths
git ls-remote git://<target>/project.git        # refs listed => the repo is served
```

Un dépôt n'est servi que lorsqu'il est marqué exportable : un fichier `git-daemon-export-ok` dans le dépôt bare, ou le démon démarré avec `--export-all` (qui exporte chaque dépôt sous son chemin de base indépendamment du marqueur). Un démon `--export-all` est la mauvaise configuration courante, il transforme tout le répertoire de base en dépôts lisibles anonymement.

## Cloner et récupérer le butin

```bash
git clone git://<target>/project.git
cd project && git log -p --all          # then mine it as in secrets-in-history
```

Il n'y a pas d'étape d'authentification, donc un clone réussi est une divulgation immédiate du code source et de l'historique. Si vous ne connaissez pas les noms de dépôts, essayez les plus courants (`.git`, `project.git`, le nom de l'application) et épluchez toute configuration CI ou de déploiement dont vous disposez déjà à la recherche d'URL `git://`.

## Services inscriptibles et étendus

L'ensemble de services du démon a son importance. Par défaut, seul `upload-pack` (clone/fetch) est activé, mais un démon démarré avec `--enable=receive-pack` accepte les push anonymes, vous permettant d'écrire dans le dépôt, ce qui, combiné à des hooks côté serveur, est de l'exécution de code (voir [hooks et exécution par la config](hooks-and-config-execution.md)). Un service `upload-archive` activé a historiquement permis d'atteindre des objets non prévus. Vérifiez ce que le démon répond :

```bash
# anonymous push is accepted only if receive-pack is enabled for the repo
git push git://<target>/project.git HEAD:refs/heads/test
```

Un push accepté plutôt que refusé est un dépôt anonyme inscriptible.

## Références

- [git-daemon documentation](https://git-scm.com/docs/git-daemon)
- [Git protocol (Pro Git): the dumb and smart protocols](https://git-scm.com/book/en/v2/Git-Internals-Transfer-Protocols)
