---
title: "Récupération de bannière : divulgation de version et de produit VPN"
description: "Les passerelles VPN divulguent des détails identifiants : les implémentations IKE laissent fuiter une empreinte d'éditeur et des ensembles de transformations, PPTP renvoie une chaîne d'éditeur et de firmware, et les portails SSL-VPN révèlent le produit et le build via la page de connexion, les en-têtes HTTP et les ressources statiques. Cette version est la clé pour relier une appliance à son exploit pré-authentification."
keywords:
  - bannière vpn
  - empreinte ike
  - portail sslvpn
  - version
  - fingerprint
---

# Récupération de bannière

Une fois le protocole connu, les détails de version et de produit proviennent du service. Les implémentations IKE répondent à `ike-scan` avec un ID de fournisseur et les ensembles de transformations proposés, identifiant l'appareil et sa cryptographie prise en charge. La connexion de contrôle de PPTP renvoie un nom d'éditeur et une version de firmware. Les portails SSL-VPN sur 443 sont les plus riches : le branding de la page de connexion, les en-têtes HTTP, les noms de cookies et les chemins de ressources statiques (noms et versions de fichiers CSS/JS) identifient le produit (FortiGate, Ivanti/Pulse, GlobalProtect, Citrix/NetScaler, Cisco ASA) et souvent le build exact. Ce build est décisif, car les exploits d'appliance sont spécifiques à la version.

```bash
ike-scan -M <target>                           # vendor ID + transforms (IKE fingerprint)
nmap -p1723 --script pptp-version <target>     # PPTP vendor/firmware
curl -skI https://<target>/                    # Server header, cookies
curl -sk https://<target>/ | grep -iE 'fortigate|pulse|ivanti|globalprotect|netscaler|citrix|anyconnect|cisco'
# product-specific paths/resources reveal the build (e.g. versioned JS/CSS, login paths)
```

## Notes d'exploitation

- Le produit et le build SSL-VPN sont les faits clés : ils se relient directement aux [exploits pré-authentification d'appliance](../ssl-vpn-appliances/index.md), identifiez donc soigneusement le branding, les en-têtes, les cookies et les ressources versionnées du portail de connexion.
- Les ID de fournisseur IKE et les ensembles de transformations identifient l'appareil IPsec et révèlent la prise en charge de cryptographie faible et la disponibilité du mode agressif, alimentant les [attaques IKE](../protocols/ipsec-ike.md).
- Les portails exposent fréquemment le build dans un chemin de ressource ou une chaîne de version même lorsque la page de connexion est générique ; comparez les hashs/noms de ressources statiques aux versions connues au besoin.
- Combinez avec la [détection de protocole](protocol-detection.md) pour un fingerprint complet avant d'attaquer.

## Références

- [Fingerprinting d'éditeur ike-scan](https://github.com/royhills/ike-scan)
- [HackTricks: VPN enumeration](https://book.hacktricks.xyz/network-services-pentesting/ipsec-ike-vpn-pentesting)
