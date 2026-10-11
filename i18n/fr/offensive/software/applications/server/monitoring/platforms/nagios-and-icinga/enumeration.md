---
title: "Énumération : identification de Nagios et Icinga"
order: 5
description: "Déterminer si une cible exécute Nagios Core, Nagios XI ou Icinga, ainsi que sa version, à partir des chemins de l'interface web, du contenu des pages et de la réponse de version de l'agent NRPE. Le produit et la version déterminent l'attaque : les exploits applicatifs web de Nagios XI contre la configuration par fichiers de Core contre Icinga Web 2, et le numéro de build exact correspond à des vulnérabilités connues."
keywords:
  - nagios enumeration
  - nagios xi
  - icinga web
  - nrpe version
  - fingerprint
---

# Énumération

La première étape consiste à identifier quel produit et quelle version sont présents, car les trois divergent fortement. Nagios Core expose une interface CGI (couramment sous `/nagios/`) avec une interface reconnaissable et une chaîne de version ; Nagios XI est une application PHP complète (sous `/nagiosxi/`) avec sa propre page de connexion et de nombreux points de terminaison supplémentaires ; Icinga Web 2 (sous `/icingaweb2/`) est l'interface d'Icinga. Les chemins web, les titres de pages, les ressources statiques et les chaînes de version en pied de page permettent d'identifier le produit et le build. L'agent NRPE sur le port 5666 renvoie également sa version à une requête `check_nrpe`. Le produit et la version déterminent toute l'approche : la riche surface d'attaque applicative web de Nagios XI et ses chaînes connues, contre l'injection dans les définitions de commandes de Core, contre les problèmes propres à Icinga Web 2.

```bash
# web product/version
curl -sk https://<target>/nagios/ | grep -ioE 'Nagios Core [0-9.]+'
curl -sk https://<target>/nagiosxi/ | grep -i 'nagios xi'
curl -sk https://<target>/icingaweb2/ | grep -i icinga
# NRPE agent version
check_nrpe -H <target>                            # returns "NRPE vX.Y.Z"
```

## Notes d'exploitation

- La distinction entre produits est décisive : Nagios XI est une application web avec un long historique de vulnérabilités ([exploits Nagios XI](nagios-xi-exploits.md)), Nagios Core se configure par fichiers (les attaques se concentrent donc sur l'[injection de commandes](command-and-plugin-injection.md) et [NRPE](nrpe-abuse.md)), et Icinga Web 2 possède sa propre surface.
- La chaîne de version correspond à des avis de sécurité connus, en particulier pour Nagios XI où les chaînes de RCE propres à chaque version sont fréquentes ; procédez à une identification précise.
- La version de NRPE (obtenue avec `check_nrpe` sans commande) confirme l'agent et sa génération, ce qui détermine si le passage d'arguments (et donc l'injection) est possible, voir [abus de NRPE](nrpe-abuse.md).
- Dirigez-vous vers l'[authentification](authentication.md) pour l'interface web et vers les surfaces agent/commande pour l'exécution.

## Références

- [Nagios XI](https://www.nagios.com/products/nagios-xi/)
- [HackTricks : Nagios](https://book.hacktricks.xyz/network-services-pentesting/5666-pentesting-nrpe)
