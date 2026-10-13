---
order: 2
title: "Ivanti Connect Secure : chaînes de contournement d'authentification et d'injection de commandes"
description: "Ivanti Connect Secure (anciennement Pulse Connect Secure) a connu des chaînes pré-authentification combinant une faille de contournement d'authentification avec une faille d'injection de commandes pour atteindre l'exécution de code à distance non authentifiée sur la gateway. Exploitées à grande échelle, elles donnent le contrôle de l'appliance et une route vers le réseau interne auquel elle sert de façade."
keywords:
  - ivanti
  - pulse connect secure
  - connect secure
  - auth bypass
  - command injection
---

# Ivanti Connect Secure

Ivanti Connect Secure (auparavant Pulse Connect Secure) est une gateway d'accès distant largement déployée, et elle a été exploitée via des chaînes pré-authentification qui combinent deux failles : un contournement d'authentification qui atteint des endpoints ou des fonctionnalités censés exiger une connexion, et une injection de commandes qui exécute des commandes OS sur l'appliance. Chaînées, elles donnent l'exécution de code à distance non authentifiée sur la gateway. Elles ont été exploitées à grande échelle, y compris par des acteurs étatiques, parce que l'appliance est exposée sur Internet et détient la route vers l'intérieur ; le propre vérificateur d'intégrité de l'appliance a aussi été ciblé pour échapper à la détection après compromission.

```bash
# fingerprint Ivanti/Pulse Connect Secure
curl -skI https://<gateway>/
curl -sk 'https://<gateway>/dana-na/' | grep -iE 'ivanti|pulse'
# the chain: reach a protected endpoint via the auth-bypass, then invoke the
# command-injection sink to run OS commands on the appliance (unauthenticated RCE).
# match the exact Connect Secure build to the advisory for the specific chain.
```

## Notes d'exploitation

- La puissance réside dans la chaîne : le contournement d'authentification seul atteint des fonctionnalités restreintes, et y acheminer l'injection de commandes donne l'exécution de code non authentifiée, donc identifiez les deux composants pour la version cible.
- Après compromission, des attaquants ont altéré l'Integrity Checker Tool de l'appliance pour se dissimuler, de sorte que l'équipement ne peut pas être cru lorsqu'il rend compte de sa propre compromission ; cela a son importance pour les considérations de persistance et d'évasion de détection.
- Le résultat est le contrôle de l'appliance plus le pivot vers le réseau interne et tout identifiant en cache ou renvoyé ; le compromis de Connect Secure a alimenté des intrusions majeures.
- Dépend de la version ; identifiez la version de Connect Secure (les chemins et ressources du portail `dana-na`) et faites-la correspondre à l'avis pour la chaîne exacte.

## Références

- [Ivanti security advisories](https://www.ivanti.com/blog)
- [CISA: Ivanti Connect Secure exploitation](https://www.cisa.gov/news-events/cybersecurity-advisories)
