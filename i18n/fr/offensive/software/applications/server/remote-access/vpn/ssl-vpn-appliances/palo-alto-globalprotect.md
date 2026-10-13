---
order: 3
title: "Palo Alto GlobalProtect : RCE non authentifiée sur la gateway"
description: "Palo Alto GlobalProtect et l'interface de gestion PAN-OS ont connu des failles d'exécution de code à distance non authentifiée, dont une injection de commandes atteignable sur le portail et la gateway GlobalProtect. Exploitées pour l'accès initial, elles donnent l'exécution de code en tant que compte privilégié sur le pare-feu, qui sert de façade et route vers le réseau interne."
keywords:
  - palo alto
  - globalprotect
  - pan-os
  - command injection
  - unauthenticated rce
---

# Palo Alto GlobalProtect

GlobalProtect est le SSL-VPN de Palo Alto, servi par le pare-feu PAN-OS, et le portail/gateway GlobalProtect comme l'interface de gestion PAN-OS ont porté des failles d'exécution de code à distance non authentifiée. La classe récurrente est l'injection de commandes, où une entrée contrôlée par l'attaquant atteint une commande OS sur l'équipement sans authentification, donnant l'exécution de code en tant que compte privilégié. Parce que le portail GlobalProtect est exposé sur Internet et que le pare-feu route vers le réseau interne, elles ont été utilisées pour l'accès initial à grande échelle ; certaines chaînes combinent une injection avec une faille de chemin ou de validation pour atteindre le sink vulnérable sans authentification.

```bash
# fingerprint GlobalProtect/PAN-OS
curl -skI https://<gateway>/
curl -sk 'https://<gateway>/global-protect/login.esp' | grep -i 'globalprotect\|pan-os'
# command-injection class: a crafted value in a GlobalProtect/management request reaches
# an OS command unauthenticated -> code execution as a privileged device account.
# match the PAN-OS/GlobalProtect version to the advisory for the specific injection + path.
```

## Notes d'exploitation

- L'injection de commandes sur le portail GlobalProtect ou la gestion PAN-OS est la classe dominante et donne l'exécution de code directe non authentifiée sur le pare-feu en tant que compte privilégié.
- Certaines chaînes associent l'injection à un path traversal ou à une faille de validation pour atteindre le sink sans authentification ; identifiez les deux moitiés pour la version cible.
- Le pare-feu est le périmètre : l'exécution de code sur l'appliance livre la configuration, les identifiants et le pivot interne, et la persistance sur l'équipement survit au-delà d'une session.
- Dépend de la version ; identifiez PAN-OS/GlobalProtect (les ressources du portail `global-protect`) et faites-les correspondre à l'avis pour la faille exacte.

## Références

- [Palo Alto Networks Security Advisories](https://security.paloaltonetworks.com/)
- [CISA: PAN-OS/GlobalProtect exploitation](https://www.cisa.gov/news-events/cybersecurity-advisories)
