---
order: 2
title: "Corruption mémoire : exécution de code à distance dans telnetd"
description: "Les implémentations de serveur Telnet ont porté de sérieuses vulnérabilités de corruption mémoire atteignables avant ou pendant l'authentification : débordements de buffer sur la pile et failles de chaîne de format dans la gestion des options et de l'environnement. Sur les anciens telnetd Unix et les builds embarqués, elles donnent une exécution de code à distance, souvent en tant que root, contre un service antérieur aux protections modernes."
keywords:
  - telnetd
  - buffer overflow
  - format string
  - rce pré-auth
  - corruption mémoire
---

# Corruption mémoire

Au-delà de ses faiblesses de protocole, le code du serveur Telnet lui-même a un long historique de bugs de corruption mémoire, et parce que `telnetd` s'exécute souvent en tant que root et est antérieur aux protections modernes contre l'exploitation (en particulier sur les builds Unix hérités et embarqués), ceux-ci donnent une exécution de code à distance avec un privilège élevé. Les classes classiques sont les débordements de buffer sur la pile dans la gestion des options et de l'environnement (les chemins de code `telrcv`/option de chiffrement) et les failles de chaîne de format, plusieurs atteignables avant authentification. Un ancien `telnetd` exposé n'est donc pas seulement un login faible mais une cible de RCE directe une fois son empreinte prise.

```bash
# fingerprint the telnetd implementation/version (banner, option negotiation)
nc <target> 23; nmap -p23 -sV <target>
# match the build to the applicable advisory: stack overflow vs format string
```

## Sous-thèmes

- **[Stack overflow](stack-overflow.md)** : débordements de buffer dans la gestion des options/de l'environnement.
- **[Format string](format-string.md)** : failles de chaîne de format dans telnetd.

## Références

- [HackTricks: Telnet](https://book.hacktricks.xyz/network-services-pentesting/pentesting-telnet)
- [Avis de sécurité historiques sur telnetd](https://www.cve.org/)
