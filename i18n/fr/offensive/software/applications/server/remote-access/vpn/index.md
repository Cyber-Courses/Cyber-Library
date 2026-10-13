---
order: 7
title: "VPN : attaquer les passerelles et clients VPN d'accès distant"
description: "Les VPN d'accès distant se terminent sur des passerelles exposées sur Internet et authentifient les utilisateurs distants vers le réseau interne, ce qui en fait une cible d'accès initial de premier plan. La surface comprend l'authentification (PSK, mots de passe, certificats), la négociation et le downgrade de cryptographie, l'énumération du service et des protocoles, les failles spécifiques aux protocoles (IPsec/IKE, OpenVPN, PPTP), les fuites de gestion du trafic, et les vulnérabilités d'appliance SSL-VPN massivement exploitées."
keywords:
  - vpn
  - ssl-vpn
  - ipsec
  - ike
  - accès distant
---

# VPN

Une passerelle VPN d'accès distant se trouve sur Internet et authentifie les utilisateurs distants vers le réseau interne, de sorte que compromettre l'une d'elles est une voie directe de l'extérieur vers l'intérieur, et c'est pourquoi les appliances VPN comptent parmi les cibles d'accès initial les plus exploitées. La surface couvre l'authentification (clés pré-partagées, mots de passe, certificats, et la MFA souvent ajoutée), la négociation et le downgrade de cryptographie, l'énumération du service et de ses protocoles, les faiblesses spécifiques aux protocoles (mode agressif IPsec/IKE, failles d'OpenVPN et de PPTP), les fuites de gestion du trafic (DNS, routes, split tunneling), et, de la plus grande portée, les vulnérabilités pré-authentification des principales appliances SSL-VPN.

```bash
# identify VPN services
nmap -sU -p500,4500 <target>                   # IKE/IPsec
nmap -p1723,443,1194 -sV <target>              # PPTP, SSL-VPN/portal, OpenVPN
ike-scan <target>                              # IKE handshake fingerprint
```

## Sous-thèmes

- **[Énumération](enumeration/index.md)** : détection de protocole et de version.
- **[Authentification](authentication/index.md)** : PSK, mots de passe et certificats.
- **[Cryptographie faible](weak-cryptography/index.md)** : faiblesses de chiffrement et d'échange de clés.
- **[Protocoles](protocols/index.md)** : IPsec/IKE, OpenVPN, PPTP, WireGuard, L2TP/SSTP.
- **[Gestion du trafic](traffic-handling/index.md)** : fuites DNS, de routes et de split tunnel.
- **[Appliances SSL-VPN](ssl-vpn-appliances/index.md)** : les exploits d'appliance pré-authentification.

## Références

- [NIST SP 800-77 (VPN IPsec)](https://csrc.nist.gov/pubs/sp/800/77/r1/final)
- [CISA: VPN exploitation advisories](https://www.cisa.gov/news-events/cybersecurity-advisories)
- [HackTricks: IPsec/IKE VPN](https://book.hacktricks.xyz/network-services-pentesting/ipsec-ike-vpn-pentesting)
