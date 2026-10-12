---
title: "Énumération : identification d'un service RDP"
description: "L'énumération RDP collecte l'identité et la posture de sécurité de l'hôte avant une attaque : la version et, via le handshake NTLM, le nom d'hôte et le domaine, ainsi que les paramètres de sécurité, à savoir si l'authentification au niveau du réseau (NLA) est requise, le niveau de chiffrement et le certificat TLS. Ces éléments déterminent quels chemins d'authentification, d'exposition et de pré-authentification s'appliquent."
keywords:
  - rdp enumeration
  - rdp-ntlm-info
  - nla
  - encryption level
  - certificate
---

# Énumération

Énumérer RDP permet d'établir ce que vous attaquez et comment c'est configuré. Le service divulgue des données d'identité utiles, notamment le nom d'hôte, le domaine et la version de l'OS via le fournisseur de sécurité NTLM pendant le handshake de connexion, et ses paramètres de sécurité décident de l'attaque : si l'authentification au niveau du réseau (NLA) est requise (ce qui force les identifiants avant une session), le niveau de sécurité et de chiffrement négocié, et le certificat TLS (qui nomme l'hôte et date le système). Cela oriente le choix entre force brute, analyse de l'exposition et exploitation en pré-authentification.

```bash
nmap -p3389 --script rdp-ntlm-info <target>    # hostname, domain, OS build via NTLM
nmap -p3389 --script rdp-enum-encryption <target>   # security layer and encryption level
```

## Sous-thèmes

- **[Banner grabbing](banner-grabbing.md)** : divulgation de la version et de l'identité.
- **[Paramètres de sécurité](security-settings.md)** : NLA, niveau de chiffrement et posture du certificat.

## Références

- [nmap rdp-ntlm-info](https://nmap.org/nsedoc/scripts/rdp-ntlm-info.html)
- [HackTricks: RDP enumeration](https://book.hacktricks.xyz/network-services-pentesting/pentesting-rdp)
