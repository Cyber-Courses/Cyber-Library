---
order: 1
title: "Fortinet FortiOS : path traversal du SSL-VPN FortiGate et RCE heap"
description: "Le SSL-VPN de Fortinet FortiGate (FortiOS) a connu des failles pré-authentification atteignables sur le portail web : un path traversal qui lit des fichiers arbitraires, y compris les données de session et d'identifiants, et des heap buffer overflows donnant l'exécution de code à distance. Les deux sont non authentifiées et ont été exploitées à grande échelle pour l'accès initial au réseau interne."
keywords:
  - fortinet
  - fortios
  - fortigate
  - ssl-vpn
  - path traversal
---

# Fortinet FortiOS

Le SSL-VPN de FortiGate, qui fait partie de FortiOS, a été une cible récurrente d'accès initial via des failles pré-authentification sur le portail web SSL-VPN. Deux classes dominent. Une vulnérabilité de path traversal a permis à un attaquant non authentifié de lire des fichiers arbitraires depuis l'appliance, y compris les fichiers de session et le matériel d'identifiants, directement depuis le portail. Et des heap buffer overflows dans le démon web SSL-VPN ont donné l'exécution de code à distance non authentifiée sur l'équipement. Parce que FortiGate est exposé sur Internet et fait le pont vers le réseau interne, les deux ont été exploitées à grande échelle ; l'approche consiste à identifier la version de FortiOS et à la faire correspondre à la faille applicable.

```bash
# fingerprint FortiGate/FortiOS and the SSL-VPN portal
curl -skI https://<gateway>/
curl -sk 'https://<gateway>/remote/login' | grep -i fortinet
# path-traversal class: unauthenticated arbitrary file read via a crafted portal request
#   target session/credential files on the appliance to recover access
# heap-overflow class: a crafted SSL-VPN request overflows the web daemon -> RCE
# match the exact FortiOS build to the advisory for the specific flaw + request.
```

## Notes d'exploitation

- Le path traversal donne une lecture de fichiers arbitraires non authentifiée ; les cibles de grande valeur sont les fichiers de session et les identifiants stockés, qui transforment une primitive de lecture en accès VPN authentifié ou en accès admin.
- La classe heap overflow est une RCE directe non authentifiée sur l'appliance, le résultat le plus fort, donnant le contrôle de l'équipement et sa route vers l'intérieur.
- Les deux dépendent de la version ; identifiez précisément la version de FortiOS (ressources du portail, en-têtes) et faites-la correspondre à l'avis, car l'endpoint vulnérable et le payload diffèrent selon la faille.
- La post-exploitation sur l'appliance livre la configuration, les identifiants en cache et le pivot vers le réseau interne ; le compromis de FortiGate a été un point d'entrée classique des ransomwares.

## Références

- [Fortinet PSIRT advisories](https://www.fortiguard.com/psirt)
- [CISA: FortiOS exploitation](https://www.cisa.gov/news-events/cybersecurity-advisories)
