---
order: 1
title: "Authentification : attaquer le contrôle d'accès VPN"
description: "Les VPN s'authentifient avec des clés pré-partagées, un nom d'utilisateur et un mot de passe, des certificats, ou une combinaison, souvent avec de la MFA superposée. Chacun est attaqué : les PSK faibles ou par défaut sont cassées, les mots de passe font l'objet de spraying et de stuffing contre le portail, et les certificats sont volés ou leur validation contournée. Une authentification VPN valide accorde un tunnel vers le réseau interne."
keywords:
  - authentification vpn
  - clé pré-partagée
  - mot de passe
  - certificat
  - mfa
---

# Authentification

Un VPN authentifie les utilisateurs distants avant d'accorder un tunnel, et les méthodes, clés pré-partagées, mots de passe, certificats et MFA superposée, sont chacune attaquables. Les clés pré-partagées faibles ou par défaut (courantes dans les configurations IPsec et L2TP) sont capturées et cassées hors ligne. Les mots de passe sont brute forcés, sprayés et sujets au credential stuffing contre le portail SSL-VPN ou l'IKE XAUTH, et les portails VPN sont une cible de choix pour le stuffing parce qu'ils acceptent les informations d'identification d'entreprise. Les certificats sont volés sur les clients ou leur validation contournée. Parce que le succès donne un tunnel vers le réseau interne, l'authentification VPN est une cible de grande valeur, et la MFA, lorsqu'elle est présente, est l'obstacle suivant (parfois lui-même contournable).

```bash
ike-scan -A -M <target>                         # aggressive mode leaks PSK material (see IKE)
nxc <sslvpn-portal> ...                          # product-specific portal spraying
```

## Sous-thèmes

- **[Clés pré-partagées faibles](weak-pre-shared-keys.md)** : casser les PSK par défaut et faibles.
- **[Brute force de mot de passe](password-brute-force.md)** : spraying et stuffing du portail.
- **[Abus de certificat](certificate-abuse.md)** : certificats volés et contournement de validation.

## Références

- [HackTricks: IPsec/IKE auth](https://book.hacktricks.xyz/network-services-pentesting/ipsec-ike-vpn-pentesting)
- [CISA: VPN credential attacks](https://www.cisa.gov/news-events/cybersecurity-advisories)
