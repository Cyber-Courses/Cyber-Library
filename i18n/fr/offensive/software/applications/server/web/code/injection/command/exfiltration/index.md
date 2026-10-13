---
title: "Exfiltration en injection de commande aveugle"
description: "Récupérer la sortie quand aucune n'est renvoyée : oracles temporels et canaux hors bande DNS/HTTP."
keywords:
  - injection de commande aveugle
  - hors bande
  - exfiltration OOB
  - temporel
  - exfiltration DNS
---

# Exfiltration

Lorsqu'une commande s'exécute mais que sa sortie n'apparaît jamais dans la réponse, les données sont récupérées par des canaux auxiliaires. Un oracle **temporel** rend l'exécution conditionnelle à une supposition et mesure un délai délibéré ; un canal **hors bande** fait atteindre par la cible un hôte que vous contrôlez (DNS ou HTTP) et transporte la sortie de la commande dans le nom d'hôte ou le chemin de requête. Ce sont les mêmes techniques d'inférence utilisées en injection SQL aveugle, appliquées à la sortie du shell.

## Pages

- **[Canal hors bande DNS](dns.md)** : Encoder la sortie d'une commande dans un label de sous-domaine DNS pour qu'elle quitte une cible d'injection de commande OS aveugle via la résolution de noms, capturée par interactsh ou un dnsbin s...
- **[Extraction aveugle temporelle](time-based.md)** : Utiliser les délais de sleep et de ping comme oracle booléen en injection de commande OS aveugle pour confirmer l'exécution et extraire des données caractère par caractère quand aucune sortie et...

## Outils

- **[commix](https://github.com/commixproject/commix)** : automatise l'extraction aveugle temporelle et hors bande.
- **[interactsh](https://github.com/projectdiscovery/interactsh)** : capture de callback DNS et HTTP auto-hébergeable.
- **Burp Collaborator** : catcher d'interactions hors bande intégré.

## Références

- [PortSwigger Web Security Academy: Blind OS command injection](https://portswigger.net/web-security/os-command-injection)
- [OWASP: OS Command Injection](https://owasp.org/www-community/attacks/Command_Injection)
