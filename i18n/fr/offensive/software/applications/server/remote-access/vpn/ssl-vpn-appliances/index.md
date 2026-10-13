---
order: 6
title: "Appliances SSL-VPN : exploits pré-authentification sur les gateways d'accès distant"
description: "Les appliances SSL-VPN et gateways d'accès distant sont exposées sur Internet, détiennent une route vers le réseau interne et exécutent un firmware propriétaire complexe, ce qui fait de leurs vulnérabilités pré-authentification le principal vecteur d'accès initial. Les principaux produits, Fortinet FortiOS, Ivanti Connect Secure, Palo Alto GlobalProtect, Citrix Gateway et Cisco ASA, ont chacun connu des chaînes d'exploitation non authentifiées utilisées à grande échelle."
keywords:
  - ssl-vpn
  - appliance
  - pre-auth
  - initial access
  - gateway
---

# Appliances SSL-VPN

Les appliances SSL-VPN et gateways d'accès distant concentrent le risque : elles sont exposées sur Internet par conception, elles détiennent une route vers le réseau interne et elles exécutent de vastes piles de firmware propriétaires qui se sont révélées riches en vulnérabilités. Résultat, leurs failles pré-authentification, path traversal, contournement d'authentification, injection de commandes et corruption mémoire atteignables sur le portail web, constituent le vecteur d'accès initial le plus lourd de conséquences de ces dernières années, exploité à grande échelle par les ransomwares et les acteurs étatiques. Chaque produit majeur a connu des chaînes non authentifiées : Fortinet FortiOS, Ivanti Connect Secure, Palo Alto GlobalProtect, Citrix Gateway et Cisco ASA. La méthode consiste toujours à identifier le produit et sa version, puis à les faire correspondre à son exploit pré-authentification connu.

```bash
curl -skI https://<gateway>/                   # product/headers
curl -sk https://<gateway>/ | grep -iE 'fortigate|ivanti|pulse|globalprotect|netscaler|citrix|anyconnect'
```

## Sous-thèmes

- **[Fortinet FortiOS](fortinet-fortios.md)** : path traversal du SSL-VPN FortiGate et RCE heap.
- **[Ivanti Connect Secure](ivanti-connect-secure.md)** : chaînes de contournement d'authentification et d'injection de commandes.
- **[Palo Alto GlobalProtect](palo-alto-globalprotect.md)** : RCE non authentifiée GlobalProtect/PAN-OS.
- **[Citrix Gateway](citrix-gateway.md)** : chaîne traversée-vers-RCE et la divulgation de jeton Citrix Bleed.
- **[Cisco ASA et AnyConnect](cisco-asa-and-anyconnect.md)** : traversal WebVPN et attaques d'identifiants sur le portail.

## Références

- [CISA Known Exploited Vulnerabilities catalog](https://www.cisa.gov/known-exploited-vulnerabilities-catalog)
- [CISA: edge-device exploitation advisories](https://www.cisa.gov/news-events/cybersecurity-advisories)
