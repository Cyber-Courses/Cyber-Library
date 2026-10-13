---
title: "Exposition sur internet : trouver et évaluer un RDP exposé"
description: "Exposer directement RDP sur le port 3389 vers internet est un vecteur d'entrée majeur pour les rançongiciels. Les attaquants trouvent ces points d'accès par scan de masse et moteurs de recherche, identifient l'hôte et sa posture NLA/version, puis appliquent des attaques sur les identifiants et en pré-authentification. Les ports non standards et les frontaux RD Gateway déplacent l'exposition mais ne la suppriment pas."
keywords:
  - internet exposure
  - port 3389
  - shodan
  - rd gateway
  - ransomware
---

# Exposition sur internet

Publier RDP directement sur internet est l'une des mauvaises configurations les plus lourdes de conséquences en pratique : les points d'accès 3389 exposés sont continuellement découverts et attaqués, et ils constituent une voie d'accès initial dominante pour les rançongiciels. Les trouver est trivial, le scan de ports de masse et les moteurs de recherche les indexent, et une fois trouvé, l'hôte est identifié puis attaqué via les identifiants ou des bugs de pré-authentification. Déplacer RDP vers un port non standard ou le faire précéder d'un RD Gateway (RDP sur HTTPS/443) change la méthode de découverte mais ne supprime pas l'exposition sous-jacente.

```bash
# direct discovery
nmap -p3389 --open <range>
masscan -p3389 <range> --rate 1000
# search-engine discovery (external): Shodan/Censys "port:3389" or RDP product tags
# RD Gateway front-end (RDP tunnelled over 443)
nmap -p443 --script http-title <target> | grep -i 'RD Web\|Remote Desktop'
# fingerprint discovered hosts for posture
nmap -p3389 --script rdp-ntlm-info,rdp-enum-encryption <discovered>
```

## Notes d'exploitation

- La découverte est facile et continue ; l'objet de cette étape est d'inventorier les hôtes exposés et d'identifier immédiatement leur statut NLA, leur version et leur domaine afin que la suite (identifiants versus pré-authentification) soit choisie hôte par hôte.
- Les ports non standards ne font qu'obscurcir : un scan `-sV` sur toute la plage identifie quand même RDP, et les moteurs de recherche l'indexent quel que soit le port.
- RD Gateway et RD Web Access déplacent RDP sur le port 443, ce qui est une surface différente (HTTPS, authentification web, parfois MFA) mais qui, au bout du compte, relaie toujours RDP ; énumérez ces frontaux web séparément.
- Un RDP exposé sur internet combine toutes les autres faiblesses de RDP : c'est là que le [credential stuffing](../authentication/credential-stuffing.md), le spraying et la [RCE en pré-authentification](../pre-authentication-flaws/index.md) sont appliqués à grande échelle.

## Références

- [CISA: RDP exposure guidance](https://www.cisa.gov/news-events/cybersecurity-advisories)
- [Microsoft: RD Gateway](https://learn.microsoft.com/en-us/windows-server/remote/remote-desktop-services/)
